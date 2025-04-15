<img src="https://github.com/yakanho/training/assets/54844453/321060e5-fc84-40f7-8caa-846d0a68494b" alt="ICANN" style="zoom:25%;" />

# Lab Enabling logging in BIND

```
Created by: Yazid AKANHO
Modified by: -
Current version: 2025032500
Previous version:-
```

------

## Intro
Logs are important for monitoring, maintenance, troubleshooting, etc.
Logs from *named* are, by default, sent to `/var/log/syslog` via syslog. However, it may be useful to make these logs more useful by configuring the logging.


## Config
On your authoritative nameservers SOA and NS1:

Create the log directory:

```
$ sudo mkdir -p /var/log/bind
$ sudo chown bind:bind /var/log/bind
```

Paste the below at the end of the `/etc/bind/named.conf.options` config file:

```
logging {
        channel transfers {
            file "/var/log/bind/transfers" versions 3 size 20M;
            print-time yes;
            severity info;
        };
        channel notify {
            file "/var/log/bind/notify" versions 3 size 20M;
            print-time yes;
            severity info;
        };
        channel dnssec {
            file "/var/log/bind/dnssec" versions 3 size 20M;
            print-time yes;
            severity info;
        };
        channel query {
            file "/var/log/bind/query" versions 5 size 20M;
            print-time yes;
            severity info;
        };
        channel general {
            file "/var/log/bind/general" versions 3 size 20M;
        print-time yes;
        severity info;
        };
    channel slog {
        syslog security;
        severity info;
    };
        category xfer-out { transfers; slog; };
        category xfer-in { transfers; slog; };
        category notify { notify; };

        category lame-servers { general; };
        category config { general; };
        category default { general; };
        category security { general; slog; };
        category dnssec { dnssec; };
        // category queries { query; };
};
```

Save and exit.

Test to confirm it works. If not, you should troubleshoot and fix your config.

```
$ sudo named-checkconf
```

**==Note:==** Be careful to the “queries” category. In production environment with hundreds or millions of queries per second, if this log is activated, the corresponding file could quickly become **heavy**.

### Update AppArmor rules
Skip this step if AppArmor is not installed on your system.

By default, AppArmor (security system) will not allow writing files to `/var/log/bind` folder. So, we need to update the Ubuntu AppArmor profile for named (bind9) to allow it:

```
$ sudo nano /etc/apparmor.d/usr.sbin.named
```

Find the following section:

```
  # /etc/bind should be read-only for bind
  # /var/lib/bind is for dynamically updated zone (and journal) files.
  # /var/cache/bind is for slave/stub data, since we're not the origin of it.
  # See /usr/share/doc/bind9/README.Debian.gz
  /etc/bind/** r,
  /var/lib/bind/** rw,
  /var/lib/bind/ rw,
  /var/cache/bind/** lrw,
  /var/cache/bind/ rw,
```

And, immediately after the last line in that block (`/var/cache/bind/ rw,`), add the following two lines :

```
  /var/log/bind/** rw,
  /var/log/bind/ rw,
```

Save the file and exit, then reload AppArmor:

```
$ sudo systemctl restart apparmor
```

Then reconfig or restart bind

```
$ sudo rndc reconfig
```

Look into `/var/log/bind/`, and see if the log files are created.

If it doesn’t work, check permissions for `/var/log/bind` and restart `named`.

Try a manual zone transfer of your own domain:

```
$ dig @100.100.X.66 AXFR MY_DOMAIN
```

Check the appropriate log file `/var/log/bind/transfers`:

```
17-Feb-2023 11:18:15.331 client 100.100.X.66#61235: transfer of 'MY_DOMAIN/IN': AXFR started
17-Feb-2023 11:18:15.331 client 100.100.X.66#61235: transfer of 'MY_DOMAIN/IN': AXFR ended
```

Try a zone transfer for a non-existent domain and check the same log file again.  What do you notice ?

Update the serial number on your primary zone file, then reload your zone and read the *notify* log file.

```
$ tail -100 /var/log/bind/notify
```

You should see something like this:

```
17-Feb-2023 13:43:48.647 zone MY_DOMAIN/IN: sending notifies (serial 2023021206)
```

### Optional - view queries

In your logging statement in `named.conf.options`, remove the `//` from the front of `category queries { query; }` and restart the nameserver. 

Then start monitoring the query log file.

```
$ sudo tail -F /var/log/bind/query
```

While that is running, in another terminal window or on someone else’s machine, execute a dig query toward your nameserver. You should see the query in the logfile.

Your should comment back the `category queries { query; };` and restart bind9 to keep the logs from filling up.





