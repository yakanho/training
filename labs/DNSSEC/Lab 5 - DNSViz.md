
<img src="https://github.com/yakanho/training/assets/54844453/321060e5-fc84-40f7-8caa-846d0a68494b" alt="ICANN" style="zoom:25%;" />

------

# Lab DNSViz

```
Created by: Yazid AKANHO
Modified by: -
Current version: 2024040100
Previous version:-
```
------
### Prerequisites

We assume that:

1. your zone grpX.<*lab_domain*>.te-labs.training. is DNSSEC signed 
2. A DS RRSet has been uploaded to the class domain authoritative nameserver, and is published in the workshop <**lab_domain**> zone.
3. Your recursive resolvers resolv1 and resolv2 are configured and DNSSEC validating (lab DNSSEC validation).
4. Your recursive resolvers (resolv1 and resolv2) software are configured to use themselves as resolvers (in /etc/resolv.conf)

> [!WARNING]
>
> If any of these things is not true, finish off the previous labs before you start this one. If you need help getting up-to-date, one of your friendly workshop lab staff will be happy to assist.



Before you continue, check the following in any of your two recursive resolvers:

```
$ dig grpX.<*lab_domain*>.te-labs.training SOA
```
returns:

- a correct "ANSWER"
- "AD" flag present 
- and SERVER should be the localhost (::1 or 100.100.X.67 or 100.100.X.68).



## Introduction

In this lab, we will check the chain of trust of DNSSEC for your lab domain using a tool called dnsviz. For your live domains, you can use its online version available at https://dnsviz.net/.



## Steps

Go to any of your recursive resolvers.

Install dnsviz required packages

```
root@resolv1:~# apt install python3-dns python3-pygraphviz python3-m2crypto
root@resolv1:~# apt-get install dnsviz
```

Run dnsviz against your domain

```
root@resolv1:~# dnsviz probe grpX.<lab-domain>.te-labs.training | dnsviz graph -Tpng -O
```

Inform your trainer who will download the file and open it in a graphical environment.

Alternatively, you may want to download and open this file on your own personal laptop. To do so, you can do the following.

Upload the file to the cloud 

```
root@resolv1:~# curl -F "file=@grp11.sa.te-labs.training.png" https://file.io
```

The expected output is 

```
root@resolv1:~# curl -F "file=@grp11.sa.te-labs.training.png" https://file.io
{"success":true,"status":200,"id":"d3e8b360-6a9c-11ef-84e6-698f82efe096",
"key":"OpFcoD21ewSG",
"path":"/",
"nodeType":"file",
"name":"grp11.sa.te-labs.training.png",
"title":null,"description":null,"size":144179,"link":"https://file.io/OpFcoD21ewSG",
"private":false,"expires":"2024-09-18T09:05:27.433Z",
"downloads":0,"maxDownloads":1,"autoDelete":true,"planId":0,"screeningStatus":"pending",
"mimeType":"image/png",
"created":"2024-09-04T09:05:27.433Z",
"modified":"2024-09-04T09:05:27.433Z"}
```

You can now open the generated URL in your browser to download the file on file.io and open it into your local machine.
