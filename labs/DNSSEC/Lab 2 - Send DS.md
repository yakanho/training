
<img src="https://github.com/yakanho/training/assets/54844453/321060e5-fc84-40f7-8caa-846d0a68494b" alt="ICANN" style="zoom:25%;" />

------

# Lab Generate your domain Delegation Signer (DS) and send it to your parent

```
Created by: Yazid AKANHO
Modified by: -
Current version: 2025032700
Previous version:-
```

------

As a reminder, your domain is grpX.<lab_domain>.te-labs.training, and your parent is <lab_domain>.te-labs.training.
First, create a folder to store your DS key files.

```
$ sudo mkdir -p /var/lib/bind/ds
$ sudo chown -R bind:bind /var/lib/bind/ds
```



### Generate your DS record

Execute the following command to get the DS record and save it in the required file:

```
$ dnssec-dsfromkey /var/lib/bind/keys/KgrpX.<lab_domain>.te-labs.training.+XYZ+YOUR-KSK-key-tag.key > /var/lib/bind/ds/DS_YOUR-KSK-key-tag.grpX
```

or you could extract the DS directly from the DNSKEY by querying your domain.

```
# dig @localhost dnskey grpX.<lab_domain>.te-labs.training | dnssec-dsfromkey -f - grpX.<lab_domain>.te-labs.training > /var/lib/bind/ds/DS_YOUR-KSK-key-tag.grpX
```

Verify the content of the generated file:

```
# cat /var/lib/bind/ds/DS_YOUR-KSK-key-tag.grpX
```

Which should contain something similar to the following line:

```
grpX.<lab_domain>.te-labs.training. IN DS YOUR-KSK-key-tag 8 2 018A86C0139BA5500AC87A5BAD8FB5D8D4F9672C319B34DB5A7F3BC10A424D6E
```



### Push the DS to your parent

You can now send the DS file to the parent. To do so, copy the content of the ds file you generated above and paste it in the text field below the network diagram of your lab in https://<*lab_domain*>.te-labs.training/grpX and wait 5 minutes to continue this lab.


### Verify that your DS is published into your parent's zone

Query your parent zone and confirm that they have published your DS.

```
sysadm@cli:~$ dig DS grpX.<*lab_domain*>.te-labs.training 

; <<>> DiG 9.16.1-Ubuntu <<>> DS grpX.<*lab_domain*>.te-labs.training
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 57805
;; flags: qr rd ra ad; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;grpX.<*lab_domain*>.te-labs.training.      IN      DS

;; ANSWER SECTION:
grpX.<*lab_domain*>.te-labs.training. 60    IN      DS      2404 8 2 8A4D8024E59D115331C8ECAF715E1168A429282646E6861420BEF8D1 7F9676E7

;; Query time: 320 msec
;; SERVER: 9.9.9.9#53(9.9.9.9)
;; WHEN: Fri Nov 04 00:23:17 UTC 2022
;; MSG SIZE  rcvd: 101

sysadm@cli:~$ 
```

At this stage, you are telling the world that you are signed and any validator (recursive resolver with **DNSSEC validation enabled**) will validate DNS answers from your domain. Let's now configure a DNSSEC validation recursive resolver in the next lab to perform that DNSSEC validation.
See you there!

