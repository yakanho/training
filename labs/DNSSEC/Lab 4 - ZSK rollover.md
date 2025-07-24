<img src="https://github.com/yakanho/training/assets/54844453/321060e5-fc84-40f7-8caa-846d0a68494b" alt="ICANN" style="zoom:25%;" />

------

# Lab ZSK rollover

```
Created by: Yazid AKANHO
Modified by: -
Current version: 2024020800
Previous version:-
```
------

### Prerequisites

We assume that:

1. your authoritative servers ns1.grpX.<*lab_domain*>.te-labs.training. and ns2.grpX.<*lab_domain*>.te-labs.training. are serving your domain.
2. Your zone is signed using BIND9, from an earlier lab.
3. You are using two key pairs, a KSK and a ZSK.
4. A DS RRSet has been uploaded to the class domain authoritative nameserver, and is published in the workshop <**lab_domain**> zone.

> [!WARNING]
>
> If any of these things are not true, finish off the previous labs before you start this one. If you need help getting up-to-date, one of your friendly workshop lab staff will be happy to assist.



Before you continue, check that:

```
$ dig grpX.<*lab_domain*>.te-labs.training dnskey +dnssec +multiline
```
shows you one ZSK and one KSK. Remember that the KSK has flags 257.

```
$ dig grpX.<*lab_domain*>.te-labs.training SOA +dnssec +multiline
```
gives you the SOA record for your domain, and that it is signed.


## Introduction

In this lab, we will carry out a manual key rollover of the ZSK. Since we are not changing the KSK, we do not need to generate a new DS record to the parent zone (this will be done in the case of a KSK rollover). Our approach will be to:

* Set some timers on the existing ZSK to specify when this key will become inactive (no longer used!) and retire from the zone.
* Create a new ZSK in addition to the existing one (a “**successor** ZSK”).
* Publish the new ZSK in the DNSKEY RRSet (so it contains two ZSKs).
* Sign the zone with both the old and the new ZSK (in this lab, we are using the default ZSK rollover methodology which is pre-publication. Double-signature and double-RRSIG are other methodologies for ZSK rollover).
* When it is safe to do so, stop signing with the old ZSK.
* When it is safe to do so, remove the old ZSK.

BIND9 will handle most of these steps, but you will still have to generate a new ZSK and provide timing instructions for the transition.

## ZSK Rollover

### Set some timers on the current ZSK (the “retiring” ZSK)

1. Find your current ZSK in the keys folder. The ZSK should have the value 256. That's the one we want. Note down its number and have a look in the content of the file. In our example, this is the ZSK. 

   > [!CAUTION]
   >
   > Remember that your filename will be different! Do not simply copy and paste Kmytld.+008+26734 later in the lab!

   ```
   root@soa:/etc/bind/keys# cat KgrpX.<lab_domain>.te-labs.training.+008+26734.key 
   ; This is a zone-signing key, keyid 26734, for grpX.<lab_domain>.te-labs.training.
   ; Created: 20221103220010 (Thu Nov  3 22:00:10 2022)
   ; Publish: 20221103220010 (Thu Nov  3 22:00:10 2022)
   ; Activate: 20221103220010 (Thu Nov  3 22:00:10 2022)
   grpX.<lab_domain>.te-labs.training. IN DNSKEY 256 3 8 AwEAAadehqG2E23DsA4MnHcaeTH/bKTHlLftvUKR9i8lVbvWNTydacdQ MsZJPTTFZXHeXFdSmxAxImc/FEGNnk9VRr3FfzfJKbc+s6r17PLWn1bO sUxawKZogOvISPytMcWnhbj8Trs8KOoAekB1PRaiPGsCP/nj68ufvrzl x2AcfDJAWPynNDjgHxeFygifVlM6iYuzmPlpcMAY5LCIS/B1MrfashJh wtj0dldgqJSp6yZHaP8vcrMa6+s5McQcqRpyoR2rpNpl6PiOUBtjE0Ho nwg1XYzSaBAbhLdmQhC4MWL/aNiXp1ybwXSVb8uZqL5k26QlKRNH2eB8YRRtq+B9rIs=
   root@soa:/etc/bind/keys# 
   ```

   

2. Use the `dnssec-settime` command, to set an “*inactive*” time and a “*delete*” time for this key. These parameters are required by BIND9 to manage the ZSK rollover. In the below example, we use "**+10mi**” to mean “ten minutes from now” and "**+30mi**” to mean “thirty minutes from now”. The change on the timers will be recorded in the key file itself. In the real world you would probably use much longer timers.

```
root@soa:/etc/bind/keys# sudo dnssec-settime -I +10mi -D +30mi KgrpX.<lab_domain>.te-labs.training.+008+26734
./KgrpX.<lab_domain>.te-labs.training.+008+26734.key
./KgrpX.<lab_domain>.te-labs.training.+008+26734.private
root@soa:/etc/bind/keys#
```

You will see two new lines ("Inactive" and "Delete") that appear in the key file.

```
root@soa:/etc/bind/keys# cat KgrpX.<lab_domain>.te-labs.training.+008+26734.key 
; This is a zone-signing key, keyid 26734, for grpX.<lab_domain>.te-labs.training.
; Created: 20221103220010 (Thu Nov  3 22:00:10 2022)
; Publish: 20221103220010 (Thu Nov  3 22:00:10 2022)
; Activate: 20221103220010 (Thu Nov  3 22:00:10 2022)
; Inactive: 20221104163455 (Fri Nov  4 16:34:55 2022)
; Delete: 20221104165455 (Fri Nov  4 16:54:55 2022)
grpX.<lab_domain>.te-labs.training. IN DNSKEY 256 3 8 AwEAAadehqG2E23DsA4MnHcaeTH/bKTHlLftvUKR9i8lVbvWNTydacdQ MsZJPTTFZXHeXFdSmxAxImc/FEGNnk9VRr3FfzfJKbc+s6r17PLWn1bO sUxawKZogOvISPytMcWnhbj8Trs8KOoAekB1PRaiPGsCP/nj68ufvrzl x2AcfDJAWPynNDjgHxeFygifVlM6iYuzmPlpcMAY5LCIS/B1MrfashJh wtj0dldgqJSp6yZHaP8vcrMa6+s5McQcqRpyoR2rpNpl6PiOUBtjE0Ho nwg1XYzSaBAbhLdmQhC4MWL/aNiXp1ybwXSVb8uZqL5k26QlKRNH2eB8 YRRtq+B9rIs=
root@soa:/etc/bind/keys# 
```



### Create a new ZSK (a “successor” ZSK)

We do not need to specify the full set of parameters (algorithm name, key size, etc.) when we generate a replacement ZSK  because we will tell the dnssec-signzone command that we are creating a successor to the old ZSK, and the software will make sure the new key it generates matches.

Use the **-i** option to specify a much shorter pre-publication interval than normal, to be compatible with the very short timers used in previous one.

```
root@soa:/etc/bind/keys# dnssec-keygen -S KgrpX.<lab_domain>.te-labs.training.+008+26734 -i5mi
Generating key pair............+++++ ..............................+++++ 
KgrpX.<lab_domain>.te-labs.training.+008+12969
root@soa:/etc/bind/keys# 
root@soa:/etc/bind/keys# chown -R bind:bind /etc/bind/keys
root@soa:/etc/bind/keys# rndc loadkeys grpX.<lab_domain>.te-labs.training
root@soa:/etc/bind/keys#
```

Then check the logs and watch the DNS! 

Here we summarize the serie of events.

```
root@soa:/etc/bind/keys# grep named /var/log/syslog
Nov  4 15:15:51 soa named[1186]: zone grpX.<lab_domain>.te-labs.training/IN (signed): next key event: 04-Nov-2022 16:15:51.577
Nov  4 16:15:51 soa named[1186]: zone grpX.<lab_domain>.te-labs.training/IN (signed): reconfiguring zone keys
Nov  4 16:15:51 soa named[1186]: zone grpX.<lab_domain>.te-labs.training/IN (signed): next key event: 04-Nov-2022 17:15:51.580
Nov  4 16:26:23 soa named[1186]: received control channel command 'loadkeys grpX.<lab_domain>.te-labs.training'
Nov  4 16:26:23 soa named[1186]: zone grpX.<lab_domain>.te-labs.training/IN (signed): reconfiguring zone keys
Nov  4 16:26:23 soa named[1186]: zone grpX.<lab_domain>.te-labs.training/IN (signed): next key event: 04-Nov-2022 16:29:55.905
Nov  4 16:29:55 soa named[1186]: zone grpX.<lab_domain>.te-labs.training/IN (signed): reconfiguring zone keys
Nov  4 16:29:55 soa named[1186]: Fetching grpX.<lab_domain>.te-labs.training/RSASHA256/12969 (ZSK) from key repository.
Nov  4 16:29:55 soa named[1186]: DNSKEY grpX.<lab_domain>.te-labs.training/RSASHA256/12969 (ZSK) is now published
Nov  4 16:29:55 soa named[1186]: zone grpX.<lab_domain>.te-labs.training/IN (signed): next key event: 04-Nov-2022 16:34:55.906
Nov  4 16:29:55 soa named[1186]: zone grpX.<lab_domain>.te-labs.training/IN (signed): sending notifies (serial 5)
Nov  4 16:29:55 soa named[1186]: client @0x7fed1c04b930 100.100.1.130#54173 (grpX.<lab_domain>.te-labs.training): transfer of 'grpX.<lab_domain>.te-labs.training/IN': IXFR started (serial 4 -> 5)
Nov  4 16:29:55 soa named[1186]: client @0x7fed1c04b930 100.100.1.130#54173 (grpX.<lab_domain>.te-labs.training): transfer of 'grpX.<lab_domain>.te-labs.training/IN': IXFR ended: 1 messages, 11 records, 2343 bytes, 0.001 secs (2343000 bytes/sec)
Nov  4 16:29:56 soa named[1186]: client @0x7fed1c0af4c0 100.100.1.131#41174 (grpX.<lab_domain>.te-labs.training): transfer of 'grpX.<lab_domain>.te-labs.training/IN': AXFR started (serial 5)
Nov  4 16:29:56 soa named[1186]: client @0x7fed1c0af4c0 100.100.1.131#41174 (grpX.<lab_domain>.te-labs.training): transfer of 'grpX.<lab_domain>.te-labs.training/IN': AXFR ended: 1 messages, 28 records, 5030 bytes, 0.001 secs (5030000 bytes/sec)
Nov  4 16:34:55 soa named[1186]: zone grpX.<lab_domain>.te-labs.training/IN (signed): reconfiguring zone keys
Nov  4 16:34:55 soa named[1186]: DNSKEY grpX.<lab_domain>.te-labs.training/RSASHA256/12969 (ZSK) is now active
Nov  4 16:34:55 soa named[1186]: DNSKEY grpX.<lab_domain>.te-labs.training/RSASHA256/26734 (ZSK) is now inactive
Nov  4 16:34:55 soa named[1186]: zone grpX.<lab_domain>.te-labs.training/IN (signed): next key event: 04-Nov-2022 16:54:55.907
Nov  4 16:34:55 soa named[1186]: zone grpX.<lab_domain>.te-labs.training/IN (signed): sending notifies (serial 6)
Nov  4 16:34:55 soa named[1186]: client @0x7fed2c05dc00 100.100.1.130#44965 (grpX.<lab_domain>.te-labs.training): transfer of 'grpX.<lab_domain>.te-labs.training/IN': IXFR started (serial 5 -> 6)
Nov  4 16:34:55 soa named[1186]: client @0x7fed2c05dc00 100.100.1.130#44965 (grpX.<lab_domain>.te-labs.training): transfer of 'grpX.<lab_domain>.te-labs.training/IN': IXFR ended: 1 messages, 10 records, 2067 bytes, 0.001 secs (2067000 bytes/sec)
Nov  4 16:34:56 soa named[1186]: client @0x7fed1c049bc0 100.100.1.131#44942 (grpX.<lab_domain>.te-labs.training): transfer of 'grpX.<lab_domain>.te-labs.training/IN': AXFR started (serial 6)
Nov  4 16:34:56 soa named[1186]: client @0x7fed1c049bc0 100.100.1.131#44942 (grpX.<lab_domain>.te-labs.training): transfer of 'grpX.<lab_domain>.te-labs.training/IN': AXFR ended: 1 messages, 28 records, 5030 bytes, 0.001 secs (5030000 bytes/sec)
Nov  4 16:54:55 soa named[1186]: zone grpX.<lab_domain>.te-labs.training/IN (signed): reconfiguring zone keys
Nov  4 16:54:55 soa named[1186]: Removing expired key 26734/RSASHA256 from DNSKEY RRset.
Nov  4 16:54:55 soa named[1186]: DNSKEY grpX.<lab_domain>.te-labs.training/RSASHA256/26734 (ZSK) is now deleted
Nov  4 16:54:55 soa named[1186]: zone grpX.<lab_domain>.te-labs.training/IN (signed): next key event: 04-Nov-2022 17:54:55.908
Nov  4 16:54:55 soa named[1186]: zone grpX.<lab_domain>.te-labs.training/IN (signed): sending notifies (serial 7)
Nov  4 16:54:55 soa named[1186]: client @0x7fed1c0a7660 100.100.1.130#42265 (grpX.<lab_domain>.te-labs.training): transfer of 'grpX.<lab_domain>.te-labs.training/IN': IXFR started (serial 6 -> 8)
Nov  4 16:54:55 soa named[1186]: client @0x7fed1c0a7660 100.100.1.130#42265 (grpX.<lab_domain>.te-labs.training): transfer of 'grpX.<lab_domain>.te-labs.training/IN': IXFR ended: 1 messages, 38 records, 9338 bytes, 0.001 secs (9338000 bytes/sec)
Nov  4 16:54:56 soa named[1186]: client @0x7fed2405ad30 100.100.1.131#60700 (grpX.<lab_domain>.te-labs.training): transfer of 'grpX.<lab_domain>.te-labs.training/IN': AXFR started (serial 8)
Nov  4 16:54:56 soa named[1186]: client @0x7fed2405ad30 100.100.1.131#60700 (grpX.<lab_domain>.te-labs.training): transfer of 'grpX.<lab_domain>.te-labs.training/IN': AXFR ended: 1 messages, 26 records, 4737 bytes, 0.001 secs (4737000 bytes/sec)
Nov  4 16:55:00 soa named[1186]: zone grpX.<lab_domain>.te-labs.training/IN (signed): sending notifies (serial 8)
```


From the logs, we can see that several events happened:

* At 16:29:55, the new ZSK was fetched and published into the zone DNSKEY.
* This generated a new zone file to be transfered secondaries `IXFR started (serial 4 -> 5)`. If you query the zone for its keys, you will notice that there are two ZSKs there. But only one is active.
* At 16:34:55, the zone keys were reconfigured, the key 12969 became active while 26734 became inactive. This also created a zone update and zone tranfer between the primary and secondaries. At this step, there are still two ZSK in the DNSKEY but they have now changed roles.
* At 16:54:55, the key 26734/RSASHA256 was deleted (removed) from the zone.
* And again, a new zone file was available to transfer toward each secondary NS. At this stage, only the active ZSK remains in the zone.

> [!IMPORTANT]
>
> In real life, you should anticipate all these changes and set enough time between events. This will allow you to react properly in case of any unexpected behavior.
>
> It is also critical to have a proper monitoring system and maintenance routines in place to watch these events in details and take necessary actions when and where required.
