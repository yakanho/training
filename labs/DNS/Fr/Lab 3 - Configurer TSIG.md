<img src="https://github.com/yakanho/training/assets/54844453/321060e5-fc84-40f7-8caa-846d0a68494b" alt="ICANN" style="zoom:25%;" />

------

# Lab : Configurer TSIG

```
Créé par : Yazid AKANHO
Modifié par : Yazid AKANHO
Version actuelle : 2024102900
Version précédente : 2024020800
```

------

# Introduction

Sécurité basée sur une clé TSIG

## Objectifs

Au lieu d'utiliser des adresses IP, nous allons maintenant utiliser des clés cryptographiques pour authentifier les transferts de zone. Cela repose sur TSIG, un mécanisme par lequel la communication entre le serveur primaire et le serveur secondaire est authentifiée à l'aide de cette clé.

**==Remarque :==**

> Les commandes précédées de $ impliquent que vous devez exécuter la commande en tant qu'utilisateur normal, et non en tant que root.
> Les commandes précédées de « # » impliquent que vous devez travailler en tant que root.
> Les commandes avec une invite plus spécifique (par exemple « rtrX> » ou « mysql> ») impliquent que vous exécutez des commandes sur un équipement distant.


## Sur votre serveur primaire (serveur SOA)

Générez la clé TSIG 

```
$ sudo tsig-keygen -a hmac-sha256 grpX-key > /tmp/grpX-key.txt
```

Vérifiez le contenu du fichier. Il devrait ressembler à ceci :

```
key "grpX-key" {
	algorithm hmac-sha256;
	secret "THIS_IS_MY_KEY";
}; 
```

Ajoutez la clé TSIG à la fin du fichier de configuration **named.conf.options**.

```
key "grpX-key" {
algorithm hmac-sha256;
        secret "THIS_IS_MY_KEY";
};
server 100.100.X.130 {
     keys {grpX-key; };
};
server 100.100.X.131 {
     keys {grpX-key; };
};
```

N'oubliez pas de remplacer X dans « **grpX-key** » ....!

Ensuite, dans votre zone, **modifiez la ligne allow-transfer** comme indiqué ci-dessous

```
zone "grpX.<lab_domain>.te-labs.training" {                                                                               
        type primary;                                                                                                  
        file "/var/lib/bind/zones/db.grpX";                                                                                     
        allow-transfer { key grpX-key; };
        also-notify { 100.100.X.130; 100.100.X.131; };                                                                                      
};
```

Comme vous pouvez le constater ci-dessus, nous avons modifié la déclaration « allow-transfer » afin d'autoriser le transfert de la zone aux détenteurs de la « tsig-key ».

Redémarrez le service *named*

```
$ sudo named-checkconf
$ sudo rndc reconfig
$ sudo systemctl status bind9
```

## Sur le serveur NS1

Testez que le transfert de zone ne fonctionne plus.

```
$ dig @100.100.X.66 axfr grpX.<lab_domain>.te-labs.training

...
; Transfer failed.
```

Un coup d'œil aux journaux du serveur SOA devrait montrer quelque chose comme :

```
$ tail /var/log/syslog

24-May-2022 10:03:29.433 client @0x7f185c006920 100.100.1.130#38993 (grp1.<lab_domain>.te-labs.training): zone transfer 'grp1.<lab_domain>.te-labs.training/AXFR/IN' denied
```

**Nous avons besoin de la clé !**

Vous pouvez également tester manuellement comme suit :

```
$ dig @100.100.X.66 -y hmac-sha256:grpX-key:THIS_IS_MY_KEY axfr grpX.<lab_domain>.te-labs.training
```


### Ajouter la clé TSIG à la configuration de votre NS1

Dans **/etc/bind/named.conf.options**, ajoutez la clé TSIG, ainsi qu'une déclaration indiquant quelle clé utiliser pour communiquer avec « 100.100.X.66; » (le serveur SOA) :

```
key "grpX-key" {
        algorithm hmac-sha256;
        secret "THIS_IS_MY_KEY";
};

server 100.100.X.66 {		// here you put the IP of YOUR primary server (SOA)
        keys { grpX-key; };
};
```

Enregistrez, quittez et **redémarrez bind9**.

### Tester la configuration

Sur le serveur SOA, **incrémentez le numéro de série** et **rechargez le service BIND9**. 

Sur ns1, consultez les journaux et validez que le transfert a réussi.

```
$ tail /var/log/syslog

zone grp2.<lab_domain>.te-labs.training/IN: Transfer started.
transfer of 'grp2.<lab_domain>.te-labs.training/IN' from 100.100.2.66#53: connected using 100.100.2.13>
zone grp2.<lab_domain>.te-labs.training/IN: transferred serial 2022052401: TSIG 'grp2-key'
transfer of 'grp2.<lab_domain>.te-labs.training/IN' from 100.100.2.66#53: Transfer status: success
transfer of 'grp2.<lab_domain>.te-labs.training/IN' from 100.100.2.66#53: Transfer completed: 1 messag>
zone grp2.<lab_domain>.te-labs.training/IN: sending notifies (serial 2022052401)
managed-keys-zone: Key 20326 for zone . is now trusted (acceptance timer complete)
resolver priming query complete
```

## Sur le serveur NS2

Éditez le fichier **/etc/nsd/nsd.conf**, créez une section key et ajoutez-y la clé TSIG grpX-key

```
key:
        name: "grpX-key"
        algorithm: hmac-sha256
        secret: "THIS_IS_MY_KEY"
```

puis corrigez le pattern :

remplacez ces lignes

```
allow-notify: 100.100.X.66 NOKEY
request-xfr: AXFR 100.100.X.66 NOKEY
```

par celles-ci :

```
allow-notify: 100.100.X.66 grpX-key
request-xfr: AXFR 100.100.X.66 grpX-key
```


Enregistrez, quittez, vérifiez et **redémarrez le service NSD**.

```
$ nsd-checkconf /etc/nsd/nsd.conf
$ sudo nsd-control reconfig
$ sudo nsd-control reload grpX.<lab_domain>.te-labs.training
```

Consultez les journaux sur NS2 et sur le serveur SOA.
