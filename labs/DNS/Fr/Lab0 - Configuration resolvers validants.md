<img src="https://github.com/yakanho/training/assets/54844453/321060e5-fc84-40f7-8caa-846d0a68494b" alt="ICANN" style="zoom:25%;" />
------

# Lab : Mise en place de résolveurs validants

------

```
Créé par : Yazid AKANHO
Modifié par : -
Version actuelle : 2026112500
Version précédente : -
```

------


Au cours de cet exercice pratique, nous allons accéder aux équipements suivants :

* **grpX-cli** : client
* **grpX-resolv1** & **grpX-resolv2** : serveurs récursifs (résolveurs)


### Mettre en place un serveur récursif validant BIND.

Nous utilisons le conteneur "Resolv 1" (serveur récursif) [**grpX-resolv1**]. Ce conteneur dispose déjà des paquets BIND9 téléchargés et installés.

Placez-vous dans le répertoire /etc/bind :

```
$ cd /etc/bind
```

À ce stade, nous devons configurer quelques options de BIND9.
Pour ce faire, éditez le fichier `/etc/bind/named.conf.options` :

```
$ sudo nano named.conf.options
```

Ajoutez ensuite les options permettant d'indiquer (lors de la résolution) les adresses IP autorisées à interroger ce résolveur, ainsi que les adresses IP sur lesquelles notre serveur écoutera (port 53). Le fichier doit se présenter comme suit :

```
options {
	directory "/var/cache/bind";

	// If there is a firewall between you and nameservers you want
	// to talk to, you may need to fix the firewall to allow multiple
	// ports to talk. See http://www.kb.cert.org/vuls/id/800113

	// If your ISP provided one or more IP addresses for stable 
	// nameservers, you probably want to use them as forwarders.  
	// Uncomment the following block, and insert the addresses replacing 
	// the all-0's placeholder.

	// forwarders {
	// 	0.0.0.0;
	// };

	//========================================================================
	// If BIND logs error messages about the root key being expired,
	// you will need to update your keys. See https://www.isc.org/bind-keys
	//========================================================================
	dnssec-validation auto;
	listen-on port 53 { localhost; 100.100.0.0/16; };
	listen-on-v6 port 53 { localhost; fd89:59e0::/32; };
	allow-query { localhost; 100.100.0.0/16; fd89:59e0::/32; };
	recursion yes;
};
```

Une fois la modification du fichier de configuration terminée, vérifiez la syntaxe de la configuration :

```
$ sudo named-checkconf
```

Redémarrez ensuite le service afin qu'il prenne en compte les changements de configuration :

```
$ sudo rndc reload
```

Vérifiez l'état du processus bind9 :

```
$ sudo systemctl status bind9
```

Vous devriez obtenir un résultat similaire à celui ci-dessous :

```
● named.service - BIND Domain Name Server
   Loaded: loaded (/lib/systemd/system/named.service; enabled; vendor preset: enabled)
  Drop-In: /etc/systemd/system/service.d
       └─lxc.conf
   Active: **active (running)** since Thu 2021-05-13 01:38:27 UTC; 4s ago
    Docs: man:named(8)
  Main PID: 849 (named)
   Tasks: 50 (limit: 152822)
   Memory: 103.2M
   CGroup: /system.slice/named.service
       └─849 /usr/sbin/named -f -u bind

May 13 01:38:27 resolv1.grpX.<lab_domain>.te-labs.training named[849]: **command channel listening on ::1#953**
May 13 01:38:27 resolv1.grpX.<lab_domain>.te-labs.training named[849]: managed-keys-zone: loaded serial 6
May 13 01:38:27 resolv1.grpX.<lab_domain>.te-labs.training named[849]: zone 0.in-addr.arpa/IN: loaded serial 1
May 13 01:38:27 resolv1.grpX.<lab_domain>.te-labs.training named[849]: zone 127.in-addr.arpa/IN: loaded serial 1
May 13 01:38:27 resolv1.grpX.<lab_domain>.te-labs.training named[849]: zone localhost/IN: loaded serial 2
May 13 01:38:27 resolv1.grpX.<lab_domain>.te-labs.training named[849]: zone 255.in-addr.arpa/IN: loaded serial 1
May 13 01:38:27 resolv1.grpX.<lab_domain>.te-labs.training named[849]: **all zones loaded**
May 13 01:38:27 resolv1.grpX.<lab_domain>.te-labs.training named[849]: **running**
May 13 01:38:27 resolv1.grpX.<lab_domain>.te-labs.training named[849]: managed-keys-zone: Key 20326 for zone . is now trusted (acceptance timer>
May 13 01:38:27 resolv1.grpX.<lab_domain>.te-labs.training named[849]: resolver priming query complete
```

### Tester votre nouveau résolveur validant

Exécutez les commandes suivantes et vérifiez si vous recevez le flag « ad » :

1. dig SOA com. @100.100.X.67 
2. dig SOA com. @100.100.X.67 +dnssec
3. dig A www.icann.org @100.100.X.67
4. dig NS icann.org @100.100.X.67
5. dig DNSKEY *grpX*.<*lab_domain*>.te-labs.training @100.100.X.67
6. dig NS *grpX*.<*lab_domain*>.te-labs.training @100.100.X.67 +dnssec
7. dig SOA *grpX*.<*lab_domain*>.te-labs.training @100.100.X.67
8. dig DNSKEY *grpX*.<*lab_domain*>.te-labs.training @100.100.X.130
9. dig DNSKEY *grpX*.<*lab_domain*>.te-labs.training @100.100.X.130 +multi
10. dig SOA *grpX*.<*lab_domain*>.te-labs.training @100.100.X.131 +dnssec +multi

Avez-vous reçu le flag « ad » pour les trois dernières requêtes dig ? Pourquoi ?

Essayez maintenant les commandes ci-dessous :

1. dig www.dnssec-failed.org @9.9.9.9

2. dig www.dnssec-failed.org @100.100.X.67 

3. dig www.dnssec-failed.org +dnssec @100.100.X.67

4. dig www.dnssec-failed.org +dnssec +cd @100.100.X.67

   

### Indiquez au système d'exploitation d'utiliser votre nouveau résolveur récursif pour ses requêtes DNS.

Toujours sur le serveur **grpX-resolv1**, éditez le fichier de configuration **/etc/resolv.conf** et remplacez tout son contenu par ceci uniquement :

```
nameserver 100.100.X.67
```

Enregistrez et quittez. Essayez ensuite les requêtes suivantes :

1. dig SOA com. 
2. dig SOA com. +dnssec
3. dig A www.icann.org
4. dig NS icann.org
5. dig NS *grpX*.<*lab_domain*>.te-labs.training
6. dig NS *grpX*.<*lab_domain*>.te-labs.training +dnssec
7. dig SOA *grpX*.<*lab_domain*>.te-labs.training
8. dig SOA *grpX*.<*lab_domain*>.te-labs.training +dnssec +multi
9. dig DNSKEY *grpX*.<*lab_domain*>.te-labs.training
10. dig DNSKEY *grpX*.<*lab_domain*>.te-labs.training +multi

Avez-vous obtenu le flag « ad » dans tous les cas ?



## Mettre en place un serveur récursif Unbound

Utilisez le conteneur "Resolv 2" (serveur récursif) [**grpX-resolv2**]. Ce conteneur dispose déjà des paquets UNBOUND téléchargés et installés.

Commencez par configurer UNBOUND. Pour ce faire, passez à l'utilisateur root et éditez le fichier de configuration **/etc/unbound/unbound.conf** :

```
$ sudo nano /etc/unbound/unbound.conf
```

Ajoutez les options indiquant (lors de la résolution) l'interface IP et le port d'écoute des requêtes, les adresses IP autorisées à interroger ce résolveur, ainsi que quelques autres paramètres. Le fichier doit se présenter comme suit :

```
# Unbound configuration file for Debian.
#
# See the unbound.conf(5) man page.
#
# See /usr/share/doc/unbound/examples/unbound.conf for a commented
# reference config file.
#
# The following line includes additional configuration files from the
# /etc/unbound/unbound.conf.d directory.

server:
        interface: 0.0.0.0
        interface: ::0

        access-control: 127.0.0.0/8 allow
        access-control: 100.100.0.0/16 allow
        access-control: fd89:59e0::/32 allow

        port: 53

        do-udp: yes
        do-tcp: yes
        do-ip4: yes
        do-ip6: yes

include: "/etc/unbound/unbound.conf.d/*.conf"
```

Vérifiez ensuite la syntaxe du fichier de configuration :

```
$ sudo unbound-checkconf
```

Si tout est correct, vous devriez obtenir un résultat similaire à celui-ci :

```
unbound-checkconf: no errors in /etc/unbound/unbound.conf
```

Redémarrez ensuite le service afin qu'il prenne en compte les changements de configuration :

```
$ sudo systemctl restart unbound
```

Vérifiez l'état du processus UNBOUND :

```
$ sudo systemctl status unbound
```

Vous devriez obtenir un résultat similaire à celui ci-dessous :

```
● unbound.service - Unbound DNS server     
Loaded: loaded (/lib/systemd/system/unbound.service; enabled; vendor preset: enabled)    
Drop-In: /etc/systemd/system/service.d
	└─lxc.conf     Active: active (running) since Thu 2021-05-13 03:49:11 UTC; 13s ago       Docs: man:unbound(8)
	Process: 571 ExecStartPre=/usr/lib/unbound/package-helper chroot_setup (code=exited, status=0/SUCCESS)
	Process: 574 ExecStartPre=/usr/lib/unbound/package-helper root_trust_anchor_update (code=exited, status=0/SUCCESS)   Main PID: 578 (unbound)      Tasks: 1 (limit: 152822)     Memory: 7.8M     
	CGroup: /system.slice/unbound.service             		└─578 /usr/sbin/unbound -d
May 13 03:49:10 resolv2.grpX.<lab_domain>.te-labs.training unbound[178]: [178:0] info: [25%]=0 median[50%]=0 [75%]=0
May 13 03:49:10 resolv2.grpX.<lab_domain>.te-labs.training unbound[178]: [178:0] info: lower(secs) upper(secs) recursions
May 13 03:49:10 resolv2.grpX.<lab_domain>.te-labs.training unbound[178]: [178:0] info:    0.000000    0.000001 1
May 13 03:49:11 resolv2.grpX.<lab_domain>.te-labs.training package-helper[577]: /var/lib/unbound/root.key has content
May 13 03:49:11 resolv2.grpX.<lab_domain>.te-labs.training package-helper[577]: success: the anchor is ok
May 13 03:49:11 resolv2.grpX.<lab_domain>.te-labs.training unbound[578]: [578:0] notice: init module 0: subnet
May 13 03:49:11 resolv2.grpX.<lab_domain>.te-labs.training unbound[578]: [578:0] notice: init module 1: validator
May 13 03:49:11 resolv2.grpX.<lab_domain>.te-labs.training unbound[578]: [578:0] notice: init module 2: iterator
May 13 03:49:11 resolv2.grpX.<lab_domain>.te-labs.training unbound[578]: [578:0] info: start of service (unbound 1.9.4).
May 13 03:49:11 resolv2.grpX.<lab_domain>.te-labs.training systemd[1]: Started Unbound DNS server.
```

### Tester votre nouveau résolveur validant

Exécutez les commandes suivantes et vérifiez si vous recevez le flag « ad » :

1. dig SOA com. @100.100.X.68 
2. dig SOA com. @100.100.X.68 +dnssec
3. dig A www.icann.org @100.100.X.68
4. dig NS icann.org @100.100.X.68
5. dig DNSKEY *grpX*.<*lab_domain*>.te-labs.training @100.100.X.68
6. dig NS *grpX*.<*lab_domain*>.te-labs.training @100.100.X.68 +dnssec
7. dig SOA *grpX*.<*lab_domain*>.te-labs.training @100.100.X.68
8. dig DNSKEY *grpX*.<*lab_domain*>.te-labs.training @100.100.X.130
9. dig DNSKEY *grpX*.<*lab_domain*>.te-labs.training @100.100.X.130 +multi
10. dig SOA *grpX*.<*lab_domain*>.te-labs.training @100.100.X.131 +dnssec +multi

Avez-vous reçu le flag « ad » pour les trois dernières requêtes dig ? Pourquoi ?

Essayez maintenant les commandes ci-dessous :

1. dig www.dnssec-failed.org @9.9.9.9
2. dig www.dnssec-failed.org @100.100.X.68 
3. dig www.dnssec-failed.org +dnssec @100.100.X.68
4. dig www.dnssec-failed.org +dnssec +cd @100.100.X.68



### Indiquez au système d'exploitation d'utiliser votre (second) nouveau résolveur récursif pour ses requêtes DNS.

Toujours sur le serveur **grpX-resolv2**, éditez le fichier de configuration **/etc/resolv.conf** et remplacez tout son contenu par ceci uniquement :

```
nameserver 100.100.X.68
```

Enregistrez et quittez. Essayez ensuite les requêtes suivantes :

1. dig SOA com. 
2. dig SOA com. +dnssec
3. dig A www.icann.org
4. dig NS icann.org
5. dig NS *grpX*.<*lab_domain*>.te-labs.training
6. dig NS *grpX*.<*lab_domain*>.te-labs.training +dnssec
7. dig SOA *grpX*.<*lab_domain*>.te-labs.training
8. dig SOA *grpX*.<*lab_domain*>.te-labs.training +dnssec +multi
9. dig DNSKEY *grpX*.<*lab_domain*>.te-labs.training
10. dig DNSKEY *grpX*.<*lab_domain*>.te-labs.training +multi

Avez-vous obtenu le flag « ad » dans tous les cas ?
