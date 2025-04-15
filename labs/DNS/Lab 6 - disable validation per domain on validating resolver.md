<img src="https://github.com/yakanho/training/assets/54844453/321060e5-fc84-40f7-8caa-846d0a68494b" alt="ICANN" style="zoom:25%;" />
------

# Lab disable DNSSEC validation per domain

```
Created by: Yazid AKANHO
Modified by: -
Current version: 2025032500
Previous version:-
```

------


During this practice we are going to access the following equipments:

* **grpX-cli** : client
* **grpX-resolv1** & **grpX-resolv2** : recursive servers (resolvers)

> [!NOTE]
>
> Before doing this lab, you should ensure you have successfully completed the lab that sets up validating resolvers.



## Temporary DNSSEC validation deactivation for broken domains

A zone can become broken due to issues with its DNSSEC configuration. While it is the responsibility of the domain administrator to do their best to avoid such things, it can still happen. And when it happens, users behind a validating recursive resolver usually do not have access to the domain data.

We can configure the recursive resolver to temporarily skip DNSSEC validation for this category of broken domains only. This can also serve when you have internal domains on which DNSSEC validation is not really necessary as those domains are trusted by default.

###BIND resolver (resolv1)

We can edit the file **/etc/bind/named.conf.options** and add the following:

``` 
validate-except
   {
       "dnssec-failed.org";
       "another.example.TLD";
   };
```

Apply the config and test.

###Unbound resolver (resolv2)
To temporarily stop a broken chain of trust, add the following to the **unbound.conf** file:

```
server:
    domain-insecure: "dnssec-failed.org"
```

###Test it !

You can *dig* to confirm if validation is disabled for your excluded domain only.

Now try the below:

1. dig www.dnssec-failed.org @100.100.X.67 

3. dig www.dnssec-failed.org +dnssec @100.100.X.67

4. dig www.dnssec-failed.org +dnssec +cd @100.100.X.67

   

