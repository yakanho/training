
<img src="https://github.com/yakanho/training/assets/54844453/321060e5-fc84-40f7-8caa-846d0a68494b" alt="ICANN" style="zoom:25%;" />

------

# Lab: Zone signing

```
Created by: Yazid AKANHO
Modified by: -
Current version: 2024020400
Previous version:-
```

------
For this lab, we use the container "SOA" (hidden primary authoritative) [grpX-soa].

## Configure BIND to sign the zone.

#### Edit config file.
Update your zone configuration statement in `/etc/bind/named.conf.local`, to look like the below : 

```
zone "grpX.<lab_domain>.te-labs.training" {
	type primary;
	file "/var/lib/bind/zones/db.grpX";
	allow-transfer { any; };
	also-notify {100.100.X.130; 100.100.X.131; };
	inline-signing yes;
	dnssec-policy te-labs;
};

////// DNSSEC POLICIES //////

dnssec-policy te-labs {
    dnskey-ttl 300;
    keys {
        ksk lifetime 365d algorithm ecdsap256sha256;
        zsk lifetime 60d algorithm ecdsap256sha256;
    };
    max-zone-ttl 300;
    parent-ds-ttl 300;
    parent-propagation-delay 1h;
    publish-safety 7d;
    retire-safety 7d;
    signatures-refresh 5d;
    signatures-validity 15d;
    signatures-validity-dnskey 15d;
    zone-propagation-delay 1h;
};
```


Then, reconfigure or restart BIND: using `rndc reconfig` or `systemctl restart bind9`. Always check status after such operation.

Some new files should appear in the *zones* directory.

#### Verify that your zone is signed.
We use the command `rndc signing -list ` to confirm that the zone is signed. You should get an output like:

```
$ sudo rndc signing -list grpX.<lab_domain>.te-labs.training
Done signing with key 52159/RSASHA256
Done signing with key 51333/RSASHA256
```
Now, time to check your system log files to understand and confirm the signing of your zone as well as the automatic transfer of the signed zone to NS1 and NS2.

```
$ sudo grep "named" /var/log/syslog
```

#### Use command line tools to query the signed zone.
We can now use *dig* utility to confirm that the zone is signed and play with the new DNSSEC RRs.



> [!WARNING]
>
> remember that at this stage, you have only signed the zone and have not yet established the chain of trust.

**QUESTION**: Will you get the "ad" flag ? Why ?

1. dig SOA *grpX*.<*lab_domain*>.te-labs.training @100.100.X.66
2. dig SOA *grpX*.<*lab_domain*>.te-labs.training @100.100.X.66 +dnssec
3. dig DNSKEY *grpX*.<*lab_domain*>.te-labs.training @100.100.X.66
4. dig DNSKEY *grpX*.<*lab_domain*>.te-labs.training @100.100.X.66 +dnssec +multi
5. dig DNSKEY *grpX*.<*lab_domain*>.te-labs.training @100.100.X.130
6. dig DNSKEY *grpX*.<*lab_domain*>.te-labs.training @100.100.X.130 +dnssec +multi
7. dig SOA *grpX*.<*lab_domain*>.te-labs.training @100.100.X.130 +dnssec +multi

**QUESTIONS**: did you get the answers ? Did you receive the  signatures ? Did you get the "ad" flag ? Why ?
