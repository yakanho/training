<img src="https://github.com/yakanho/training/assets/54844453/321060e5-fc84-40f7-8caa-846d0a68494b" alt="ICANN" style="zoom:25%;" />

------

# Lab : Configurer votre zone DNS

```
Créé par : Yazid AKANHO
Modifié par : -
Version actuelle : 2026051000
Version précédente : 2025032400
```

------

<img src="https://github.com/yakanho/training/assets/54844453/d794aab6-720a-4802-86b5-6afea3032957" alt="lab_topology" style="zoom:50%;" />

**<u>Topologie du lab (Groupe X) </u>**

------



```
  DEVICE NAME        IPv4 ADDRESS              IPv6 ADDRESS
+--------------+-----------------------+-----------------------------+
| grpX-cli     | 100.100.X.2 (eth0)    | fd89:59e0:X::2 (eth0)       |
+--------------+-----------------------+-----------------------------+
| grpX-ns1     | 100.100.X.130 (eth0)  | fd89:59e0:X:128::130 (eth0) |
+--------------+-----------------------+-----------------------------+
| grpX-ns2     | 100.100.X.131 (eth0)  | fd89:59e0:X:128::131 (eth0) |
+--------------+-----------------------+-----------------------------+
| grpX-resolv1 | 100.100.X.67 (eth0)   | fd89:59e0:X:64::67 (eth0)   |
+--------------+-----------------------+-----------------------------+
| grpX-resolv2 | 100.100.X.68 (eth0)   | fd89:59e0:X:64::68 (eth0)   |
+--------------+-----------------------+-----------------------------+
| grpX-rtr     | 100.64.1.X (eth0)     | fd89:59e0:X::1 (eth1)       |
|              | 100.100.X.65 (eth2)   | fd89:59e0:X:64::1 (eth2)    |
|              | 100.100.X.193 (eth4)  | fd89:59e0:X:192::1 (eth4)   |
|              | 100.100.X.129 (eth3)  | fd89:59e0:X:128::1 (eth3)   |
|              | 100.100.X.1 (eth1)    | fd89:59e0:0:1::X (eth0)     |
+--------------+-----------------------+-----------------------------+
| grpX-soa     | 100.100.X.66 (eth0)   | fd89:59e0:X:64::66 (eth0)   |
+--------------+-----------------------+-----------------------------+
```

Au cours de cet exercice pratique, nous n'accéderons qu'aux équipements suivants :

* **grpX-cli** : client
* **grpX-soa** : serveur faisant autorité caché (primaire)
* **grpX-ns1** & **grpX-ns2** : serveurs faisant autorité secondaires

> [!IMPORTANT]
>
> Tout au long de ce lab, veillez à toujours remplacer ***X*** par votre numéro de groupe dans les adresses IP, les noms de serveurs et partout ailleurs où c'est nécessaire. Il en va de même pour <*lab_domain*>, à remplacer par le nom de domaine enregistré pour la session.



## Configurer le serveur faisant autorité primaire (BIND)

### Introduction

Nous allons configurer un serveur faisant autorité caché et créer la zone faisant autorité pour votre nom de domaine *grpX*.<*lab_domain*>.te-labs.training.

### Ce que nous savons déjà

Notre « parent » (<*lab_domain*>.te-labs.training) a déjà créé les éléments suivants dans sa propre zone :

```shell
; grpX
grpX             NS          <lab_domain>.te-labs.training.
; ---Placeholder for grpX DS record (DO NOT MANUALLY EDIT THIS LINE)---

```

La configuration de notre zone doit être compatible avec cela.

### Définir la zone faisant autorité

Nous utilisons le conteneur « SOA » (primaire faisant autorité caché) [**grpX-soa**]

Nous nous plaçons dans le répertoire `/var/lib`, créons un nouveau dossier pour nos fichiers de zone. Dans ce nouveau dossier, nous créons ensuite un nouveau fichier pour les données de zone de notre domaine.

```
$ sudo mkdir -p /var/lib/bind/zones
$ cd /var/lib/bind/zones
$ sudo touch db.grpX
$ sudo chown -R bind:bind /var/lib/bind
```

Ensuite, modifiez la zone db.grpX pour qu'elle ressemble à ce qui suit :

```
; grpX 

$TTL    120
@       IN      SOA     soa.grpX.<lab_domain>.te-labs.training. dnsadmin.grpX.<lab_domain>.te-labs.training. (                                            
                              1         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                          86400 )       ; Negative Cache TTL
;

; grpX 
@             NS           <lab_domain>.te-labs.training.
@	     TXT	   "I AM LEARNING DNS AND IT IS FUN"

ns1         A           100.100.X.130
ns1         AAAA        fd89:59e0:X:128::130
ns2         A           100.100.X.131
ns2         AAAA        fd89:59e0:X:128::131
soa			A			100.100.X.66
soa 		AAAA		fd89:59e0:X:64::66
resolv1		A			100.100.X.67
resolv1		AAAA		fd89:59e0:X:64::67
resolv2		A			100.100.X.68
resolv2		AAAA		fd89:59e0:X:64::68
www         A           100.100.X.130
```

Vous pouvez ajouter autant d'enregistrements que vous le souhaitez.

> [!TIP]
>
> Une fois cela fait, il est important de vérifier le format du fichier de zone.
> Utilisez pour cela la commande ***named-checkzone DOMAIN_NAME ZONE_FILE*** (pensez à remplacer DOMAIN_NAME et ZONE_FILE en conséquence).



Dans le fichier de configuration ***/etc/bind/named.conf.local***, créez une nouvelle déclaration « zone » comme ci-dessous :

```
zone "grpX.<lab_domain>.te-labs.training" {
		type primary;
		file "/var/lib/bind/zones/db.grpX";
		allow-transfer { any; };
		also-notify {
                100.100.X.130; 
                100.100.X.131; 
                fd89:59e0:X:128::130; 
                fd89:59e0:X:128::131; 
        };
}; 
```


> [!TIP]
>
> Une fois cela fait, utilisez ***named-checkconf*** pour vérifier que votre configuration BIND est correcte.



Rechargez le service DNS pour prendre en compte la dernière configuration et vérifiez son état. Vous devriez obtenir une sortie semblable à celle ci-dessous

```
root@soa:/var/lib$ sudo rndc reload
root@soa:/var/lib$ sudo systemctl status bind9
● named.service - BIND Domain Name Server
     Loaded: loaded (/lib/systemd/system/named.service; enabled; vendor preset: enabled)
    Drop-In: /run/systemd/system/service.d
             └─zzz-lxc-service.conf
     Active: active (running) since Tue 2024-10-29 11:37:52 UTC; 7s ago
       Docs: man:named(8)
   Main PID: 2944 (named)
      Tasks: 26 (limit: 38066)
     Memory: 52.8M
     CGroup: /system.slice/named.service
             └─2944 /usr/sbin/named -f -u bind

Oct 29 11:37:52 soa.grpX.<lab_domain>.te-labs.training named[2944]: zone grpX.<lab_domain>.te-labs.training/I>
Oct 29 11:37:52 soa.grpX.<lab_domain>.te-labs.training named[2944]: all zones loaded
Oct 29 11:37:52 soa.grpX.<lab_domain>.te-labs.training named[2944]: running
Oct 29 11:37:52 soa.grpX.<lab_domain>.te-labs.training named[2944]: zone grpX.<lab_domain>.te-labs.training/I>
Oct 29 11:37:53 soa.grpX.<lab_domain>.te-labs.training named[2944]: managed-keys-zone: Key 20326 fo>
Oct 29 11:37:53 soa.grpX.<lab_domain>.te-labs.training named[2944]: resolver priming query complete
Oct 29 11:37:53 soa.grpX.<lab_domain>.te-labs.training named[2944]: checkhints: b.root-servers.net/>
Oct 29 11:37:53 soa.grpX.<lab_domain>.te-labs.training named[2944]: checkhints: b.root-servers.net/>
Oct 29 11:37:53 soa.grpX.<lab_domain>.te-labs.training named[2944]: checkhints: b.root-servers.net/>
Oct 29 11:37:53 soa.grpX.<lab_domain>.te-labs.training named[29
...
```



Ensuite, interrogez votre zone sur le serveur local :

```
root@soa:/var/lib$ dig @localhost soa grpX.<lab_domain>.te-labs.training.
; <<>> DiG 9.16.1-Ubuntu <<>> @localhost soa grpX.<lab_domain>.te-labs.training.
; (2 servers found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 64339
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 270e2c46ed443c1c01000000609c59f04ba85015ff71998d (good)
;; QUESTION SECTION:
;grpX.<lab_domain>.te-labs.training.        IN      SOA

;; ANSWER SECTION:
grpX.<lab_domain>.te-labs.training. 120 IN   SOA     soa.grpX.<lab_domain>.te-labs.training. dnsadmin.grpX.<lab_domain>.te-labs.training. 1 604800 86400 2419200 86400

;; Query time: 0 msec
;; SERVER: ::1#53(::1)
;; WHEN: Wed May 12 22:42:56 UTC 2024
;; MSG SIZE  rcvd: 170
```



## Configurer les serveurs faisant autorité secondaires

Ces serveurs sont ceux qui exposent publiquement notre zone (ils seront donc ouverts à tous).

#### Configurer le serveur ns1 [**ns1.grpX**]

**Le serveur ns1 exécute BIND** (d'ISC)

Placez-vous dans le répertoire `/var/lib` et créez un fichier qui contiendra notre fichier de zone sur le serveur de noms :

```
$ sudo mkdir -p /var/lib/bind/zones
$ sudo touch /var/lib/bind/zones/db.grpX.secondary
$ sudo chown -R bind:bind /var/lib/bind
```

Pour cela, dans le fichier ***/etc/bind/named.conf.local***, configurez les paramètres suivants :

```
//
// Do any local configuration here
//

zone "grpX.<lab_domain>.te-labs.training" {
        type secondary;
        file "/var/lib/bind/zones/db.grpX.secondary";
        masters { 
        100.100.X.66; 
        fd89:59e0:X:64::66;
    };
};
```


Vérifiez la configuration et, s'il n'y a pas d'erreur, rechargez la configuration :

```
$ sudo named-checkconf
$ sudo rndc reload
```


Vérifiez qu'il a redémarré correctement :

```
$ sudo systemctl status bind9
```

```
● named.service - BIND Domain Name Server
     Loaded: loaded (/lib/systemd/system/named.service; enabled; vendor preset: enabled)
    Drop-In: /etc/systemd/system/service.d
             └─lxc.conf
     Active: active (running) since Thu 2024-05-13 04:25:43 UTC; 9s ago
       Docs: man:named(8)
   Main PID: 739 (named)
      Tasks: 50 (limit: 152822)
     Memory: 103.9M
     CGroup: /system.slice/named.service
             └─739 /usr/sbin/named -f -u bind

May 13 04:25:43 ns1.grpX.<lab_domain>.te-labs.training named[739]: all zones loaded
May 13 04:25:43 ns1.grpX.<lab_domain>.te-labs.training named[739]: running
May 13 04:25:43 ns1.grpX.<lab_domain>.te-labs.training named[739]: zone grpX.<lab_domain>.te-labs.training/IN: Transfer started.
May 13 04:25:43 ns1.grpX.<lab_domain>.te-labs.training named[739]: transfer of 'grpX.<lab_domain>.te-labs.training/IN' from 100.100.X.66#53: connec>
May 13 04:25:43 ns1.grpX.<lab_domain>.te-labs.training named[739]: zone grpX.<lab_domain>.te-labs.training/IN: transferred serial 1
May 13 04:25:43 ns1.grpX.<lab_domain>.te-labs.training named[739]: transfer of 'grpX.<lab_domain>.te-labs.training/IN' from 100.100.X.66#53: Transf>
May 13 04:25:43 ns1.grpX.<lab_domain>.te-labs.training named[739]: transfer of 'grpX.<lab_domain>.te-labs.training/IN' from 100.100.X.66#53: Transf>
May 13 04:25:43 ns1.grpX.<lab_domain>.te-labs.training named[739]: zone grpX.<lab_domain>.te-labs.training/IN: sending notifies (serial 1)
May 13 04:25:43 ns1.grpX.<lab_domain>.te-labs.training named[739]: managed-keys-zone: Key 20326 for zone . is now trusted (acceptance timer com>
May 13 04:25:43 ns1.grpX.<lab_domain>.te-labs.training named[739]: resolver priming query complete
```



#### Configurer le serveur ns2 [**ns2.grpX**]

**Le serveur ns2 exécute NSD** (de NLnet Labs)

Commencez par créer le répertoire destiné à vos fichiers de zone.

```
$ sudo mkdir -p /var/lib/nsd
$ sudo chown -R nsd:nsd /var/lib/nsd
```

Ensuite, dans le fichier ***/etc/nsd/nsd.conf***, configurez les paramètres suivants :

```
# NSD configuration file for Debian.
#
# See the nsd.conf(5) man page.
#
# See /usr/share/doc/nsd/examples/nsd.conf for a commented
# reference config file.
#
# The following line includes additional configuration files from the
# /etc/nsd/nsd.conf.d directory.

include: "/etc/nsd/nsd.conf.d/*.conf"

server:
    log-only-syslog: yes
    zonesdir: "/var/lib/nsd"
    nsid: "ascii_grpX NSD nsid"
    hide-version: no
    hide-identity: no
    cookie-secret: "71ff147d946b942ed66e608b64dc54c9"
    answer-cookie: yes

pattern:
    name: "fromprimary"
    allow-notify: 100.100.X.66 NOKEY
    allow-notify: fd89:59e0:X:64::66 NOKEY
    request-xfr: AXFR 100.100.X.66 NOKEY
    request-xfr: AXFR fd89:59e0:X:64::66 NOKEY
    request-xfr: AXFR fd89:59e0:X::2 NOKEY

zone:
    name: "grpX.<lab_domain>.te-labs.training."
    zonefile: "db.grpX.secondary"
    include-pattern: "fromprimary"
```

Vérifiez la configuration et, s'il n'y a pas d'erreur, rechargez la configuration :

```
$ sudo nsd-checkconf /etc/nsd/nsd.conf
$ sudo nsd-control reconfig
```

Vérifiez qu'il a redémarré correctement :

```
$ sudo systemctl status nsd
```

```
● nsd.service - Name Server Daemon
     Loaded: loaded (/lib/systemd/system/nsd.service; enabled; vendor preset: enabled)
    Drop-In: /etc/systemd/system/service.d
             └─lxc.conf
     Active: active (running) since Thu 2024-05-13 05:02:35 UTC; 1min 22s ago
       Docs: man:nsd(8)
   Main PID: 638 (nsd)
      Tasks: 3 (limit: 152822)
     Memory: 114.5M
     CGroup: /system.slice/nsd.service
             ├─638 /usr/sbin/nsd -d
             ├─639 /usr/sbin/nsd -d
             └─640 /usr/sbin/nsd -d

May 13 05:02:35 ns2.grpX.<lab_domain>.te-labs.training systemd[1]: Starting Name Server Daemon...
May 13 05:02:35 ns2.grpX.<lab_domain>.te-labs.training nsd[638]: nsd starting (NSD 4.1.26)
May 13 05:02:35 ns2.grpX.<lab_domain>.te-labs.training nsd[638]: [2024-05-13 05:02:35.865] nsd[638]: notice: nsd starting (NSD 4.1.26)
May 13 05:02:35 ns2.grpX.<lab_domain>.te-labs.training nsd[639]: nsd started (NSD 4.1.26), pid 638
May 13 05:02:35 ns2.grpX.<lab_domain>.te-labs.training nsd[639]: [2024-05-13 05:02:35.922] nsd[639]: notice: nsd started (NSD 4.1.26), pid 638
May 13 05:02:35 ns2.grpX.<lab_domain>.te-labs.training systemd[1]: Started Name Server Daemon.
```



## Tester la configuration et la propagation de votre zone.

Nous allons maintenant utiliser l'outil *dig* pour vérifier la configuration et la propagation de la zone, puis faire de même pour un ou deux autres groupes de la classe et partager vos observations. Depuis votre client, exécutez les requêtes dig suivantes. Toutes doivent retourner une **section réponse (answer section)**, sinon vous devez revoir vos configurations avant de continuer :

1. dig soa *grpX*.<*lab_domain*>.te-labs.training. @100.100.X.66
2. dig soa *grpX*.<*lab_domain*>.te-labs.training. @100.100.X.130
3. dig soa *grpX*.<*lab_domain*>.te-labs.training. @100.100.X.131
4. dig soa *grpX*.<*lab_domain*>.te-labs.training. @100.100.X.131 +short
5. dig soa *grpX*.<*lab_domain*>.te-labs.training. @100.100.X.131 +multi
6. dig NS *grpX*.<*lab_domain*>.te-labs.training. @100.100.X.130
7. dig NS *grpX*.<*lab_domain*>.te-labs.training. @100.100.X.130 +multi
