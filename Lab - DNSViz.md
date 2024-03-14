<img src="./icann.jpg" alt="ICANN" style="zoom:25%;" />

# Lab: DNSViz
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
