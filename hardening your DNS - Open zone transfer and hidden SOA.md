![ICANN](file:///Users/yazid.akanho/Documents/Markdown%20documents/Lab%20scripts/icann.jpg?lastModify=1716534751)

# Lab Hardening your DNS config (Intro)

```
Created by: Yazid AKANHO
version: 2024042400
Previous revision : 2024030100
Modified by: -
```



Now that your zone configuration is working fine (if not, please do not continue here and go back to fix the config before you continue), we need to do some security enhancements to protect our DNS against some types of attacks.

##Open Zone transfer
Try to do a manual zone transfer using respectively ns1, ns2 and SOA server IP address from our client machine. What do you notice ? 

Tip: the command to use is ***dig axfr <*domain*> @nameserver***. 


Do some Internet research and talk with your classmates to find a solution to avoid open zone transfer.

####Fix the open zone transfer
If your answer to the above question is ***allow-transfer***, then you are right.
Go ahead and apply the appropriate configuration update to fix the issue discussed above.
Then, test again the zone transfer. Is the issue resolved ?
> Note: allow-transfer is not the single way to achieve this. Other methods exist.


## Hidden primary
Is your hidden primary server really "hidden" ? Try to query your zone on it from any other VM and confirm wheather it replies or not. After this test, can you confirm if it is really "hidden" ?

Discuss with other participants to find a solution to make it a real "hidden" primary nameserver; i.e. it should not respond to DNS queries as it is the primary and not supposed to serve the zone publicly. This is a best practice in DNS. However, do not forget that for troubleshooting purpose, you may always need to query it from some specific (or known) IP addresses.

Apply the solution after you have agreed on a consensus with the training facilitators.