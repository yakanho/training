

<img src="https://github.com/yakanho/training/assets/54844453/321060e5-fc84-40f7-8caa-846d0a68494b" alt="ICANN" style="zoom:25%;" />

------

# Lab: Manual Zone Signing

```
Created by: Yazid AKANHO
Modified by: -
Current version: 2026020400
Previous version:-
```

------

This lab should be executed in your Primary authoritative nameserver (***SOA***).



To sign the zone manually, we first need two pairs of keys: a ZSK and a KSK. 

> [!NOTE]
>
> It is possible to sign the zone with a single key pair called CSK: Combined Signing Key. We won't use this option in this lab.



Let's start with making a backup of our zone file:

```
$ sudo cp /var/lib/bind/zones/db.grpX /var/lib/bind/zones/db.grpX.backup
```

Create a directory for your DNSSEC keys

```
$ mkdir -p /var/lib/bind/keys
$ cd /var/lib/bind/keys
```

Generate **ZSK**

```
$ dnssec-keygen -a RSASHA256 -b 2048 -n ZONE grpX.<lab_domain>.te-labs.training
```

Generate **KSK**

```
$ dnssec-keygen -f KSK -a RSASHA256 -b 2048 -n ZONE grpX.<lab_domain>.te-labs.training
```

Change ownership to the bind folder content and then sign your zone.

```
$ sudo chown -R bind:bind /var/lib/bind/
$ sudo dnssec-signzone -S -K /var/lib/bind/keys/ -o grpX.<lab_domain>.te-labs.training /var/lib/bind/zones/db.grpX
```

You should get an output similar to the below:

```
Fetching grpX.<lab_domain>.te-labs.training/RSASHA256/22521 (ZSK) from key repository.
Fetching grpX.<lab_domain>.te-labs.training/RSASHA256/16214 (KSK) from key repository.
- RSASHA256
Zone fully signed:
Algorithm: RSASHA256: KSKs: 1 active, 0 stand-by, 0 revoked
                      ZSKs: 1 active, 0 stand-by, 0 revoked
/var/lib/bind/zones/db.grpX.signed
```


Then, replace the *db.grpX* file in `named.conf.local` with the *db.grpX.signed* and restart the server using ```rndc reload```.


#### Use command line tools to query the signed zone and verify if the signing is effective.

We can now use *dig* utility to confirm that the zone is signed and play with the new DNSSEC RRs.

> [!WARNING]
>
> remember that at this stage, you have only signed the zone and have not yet established the chain of trust.

**QUESTION**: Will you get the "ad" flag ? Why ?

```
root@soa:/etc/bind# dig @localhost soa grpX.<lab_domain>.te-labs.training. +dnssec 

; <<>> DiG 9.20.27-1+ubuntu24.04.1+deb.sury.org+1-Ubuntu <<>> @localhost soa grpX.<lab_domain>.te-labs.training +dnssec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 11463
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags: do; udp: 1232
; COOKIE: f435e3a4f89133f4010000006a97595e534de7fc2af78306 (good)
;; QUESTION SECTION:
;grpX.<lab_domain>.te-labs.training.   IN      SOA

;; ANSWER SECTION:
grpX.<lab_domain>.te-labs.training. 120 IN     SOA     soa.grpX.<lab_domain>.te-labs.training. dnsadmin.grpX.<lab_domain>.te-labs.training. 5 604800 86400 2419200 86400
grpX.<lab_domain>.te-labs.training. 120 IN     RRSIG   SOA 8 4 120 20261001220017 20260901220017 22521 grpX.<lab_domain>.te-labs.training. L3az6N6luNLohAvpRdR2qYyc5/MV/E6HnHdopNdvWdHaunJdK1hYI9Mv 7oChN+fkpkyqxZ2mcebehVcHnN2HVF6tka6DlP9/XVpkjdzClNEdyy2e yrPNXh84sYZLJUaxI0icys8S7Syj6fjL3oEn0mVkpvB87PzMoUW8Yyzl AI07DR3k7uQQjWn3DWflO8Az3WtanQCbdtbDT2kl4qcNj9sknHcOvgyJ 3feS4RyjV85zLb2daWnlnlEd2hZOdwB4WgIf5oZ9p2KHyvyLAvChYeRD fWeHDASTZPUMwUE5Yl4XS76NWguC7OGdYBul8XciiJvITy0EsGZRcCt6 6kRsiA==

;; Query time: 0 msec
;; SERVER: 127.0.0.1#53(localhost) (UDP)
;; WHEN: Tue Sep 01 23:01:50 UTC 2026
;; MSG SIZE  rcvd: 448
```

More tests: 
1. dig DNSKEY *grpX*.<*lab_domain*>.te-labs.training @100.100.X.130
2. dig DNSKEY *grpX*.<*lab_domain*>.te-labs.training @100.100.X.130 +dnssec +multi
3. dig SOA *grpX*.<*lab_domain*>.te-labs.training @100.100.X.130 +dnssec +multi

**QUESTIONS**: did you get the answers ? Did you receive the  signatures ? Did you get the "ad" flag ? Why ?



> [!IMPORTANT]
>
> When you are done with the manual signing and confirm that your public (secondary) nameservers are serving the signed zone, you should:

1. revert back `named.conf.local` to its previous configuration, i.e. configure BIND to serve the unsigned zone file as before the manual signing configuration : `file "/etc/bind/zones/db.grpX";` 
2. backup the ***db.grpX.signed*** zone file and delete it. BIND will create its own signed zone file in the next step of the lab.
3. increase the serial in the unsigned zone file () and reload BIND9.


