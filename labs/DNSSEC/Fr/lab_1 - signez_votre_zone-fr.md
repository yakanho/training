<img src="https://github.com/yakanho/training/assets/54844453/321060e5-fc84-40f7-8caa-846d0a68494b" alt="ICANN" style="zoom:25%;" />

------

# Lab: Signez votre zone

```
Créé par: Yazid AKANHO
Modifié par: -
version courante: 2026051700
Version antérieure:-
```

------

Pour ce lab, vous utiliserez le conteneur "SOA" (autoritatif primaire) [grpX-soa].

## Créez le répertoire qui abritera vos clés DNSSEC

```
$ sudo mkdir -p /var/lib/bind/keys
$ sudo chown -R bind:bind /var/lib/bind/keys
```



## Configurez BIND pour signer la zone (in-line signing)

Le fichier de configuration  `/etc/bind/named.conf.local` comporte déjà des instructions ayant permis de charger votre zone. A présent, vous devez mettre à jour cette configuration en ajoutant les instructions de signature automatique de zone par BIND. La nouvelle configuration pour votre zone devrait ressembler à celle-ci : 

```
zone "grpX.<lab_domain>.te-labs.training" {
	type primary;
	file "/var/lib/bind/zones/db.grpX";
	allow-transfer { any; };
	also-notify {100.100.X.130; 100.100.X.131; };
	key-directory "/var/lib/bind/keys"; 				# nouvelle instruction
	inline-signing yes;									# nouvelle instruction
	dnssec-policy NotForProduction;						# nouvelle instruction
	checkds ( no );
};

////// Nouvelle DNSSEC POLICY pour la gestion DNSSEC //////

dnssec-policy NotForProduction {
    inline-signing yes;
    dnskey-ttl 60;
    max-zone-ttl 60;
    offline-ksk false;
    parent-ds-ttl 60s;
    parent-propagation-delay 1s;
    zone-propagation-delay 1s;
    publish-safety 0s;
    purge-keys 1h;
    retire-safety 1m;
    signatures-jitter 31s;
    signatures-refresh 15m;
    signatures-validity 1h;
    signatures-validity-dnskey 1h;
    keys {
        ksk key-directory lifetime 4h algorithm ecdsa256;
        zsk key-directory lifetime 1h algorithm ecdsa256;
    };
    cdnskey no;
    cds-digest-types { };
};
```

Une fois le fichier précédent mis à jour, vous devez instruire BIND pour appliquer la nouvelle configuration avec `rndc reconfig` ou simplement redémarrer le service avec  `systemctl restart bind9`.

> [!IMPORTANT]
>
> N'oubliez pas de toujours vérifier si le service fonctionne bien après un redémarrage. 



#### Vérifiez si BIND a bien généré les clés conformément aux instructions de la "DNSSEC policy" utilisée

```
$ ll /var/lib/bind/keys
```

> [!TIP]
>
> Je vous suggère de lire le contenu de ces fichiers pour vous faire une meilleure idée de leurs rôles respectifs et de cette étape du processus de signature DNSSEC.



#### Vérifiez si BIND a signé la zone.

Utilisez la commande  `rndc dnssec -status ZONE ` pour confirmer si BIND a effectivement signé votre zone.



```
$ sudo rndc dnssec -status grpX.<lab_domain>.te-labs.training
dnssec-policy: NotForProduction
current time:  Sun May 17 17:52:38 2026

key: 8731 (ECDSAP256SHA256), ZSK
  published:      yes - since Sun May 17 17:50:48 2026
  zone signing:   yes - since Sun May 17 17:50:48 2026

  Next rollover scheduled on Sun May 17 18:49:47 2026
  - goal:           omnipresent
  - dnskey:         omnipresent
  - zone rrsig:     omnipresent

key: 42330 (ECDSAP256SHA256), KSK
  published:      yes - since Sun May 17 17:50:48 2026
  key signing:    yes - since Sun May 17 17:50:48 2026

  Next rollover scheduled on Sun May 17 21:49:47 2026
  - goal:           omnipresent
  - dnskey:         omnipresent
  - ds:             rumoured
  - key rrsig:      omnipresent

```



Auparavant, nous utilisions la commande  `rndc signing -list ZONE ` pour confirmer la signature de zone par BIND.

```
$ sudo rndc signing -list grpX.<lab_domain>.te-labs.training
Done signing with key 8731/ECDSAP256SHA256
Done signing with key 42330/ECDSAP256SHA256
```



Vérifier les logs systèmes générés par ces differentes actions et opérations survenues sur votre zone vous permettra de mieux comprendre et confirmer la signature de zone ainsi que les autres opérations/tâches impliquées dans la signature DNSSEC de zone. Vous pourriez également en profiter pour vérifier le statut du transfert de la nouvelle zone vers les serveurs secondaires NS1 and NS2.

```
$ sudo journalctl -u named
```



#### Interrogez à présent votre zone et familiarisez vous avec les nouveaux enregistrements issus du processus de signature DNSSEC.

Utilisez votre compagnon  ***dig***  pour interroger la zone et obtenir les enregistrements, y compris ceux liés à DNSSEC (DNSKEY, RRSIG, NSEC/NSEC3, etc.).



> [!WARNING]
>
> Pour rappel, à ce stade, votre zone est seulement signée avec DNSSEC. Une étape importante du processus de signature de zone DNSSEC est l'**établissement de la chaine de confiance**. Le lab suivant est dédié à cette tâche.



**QUESTION**: examinez les commandes ci-dessous et dites si vous pensez obtenir le drapeau (flag) "**ad**" dans les réponses qu'elles devraient engendrer? Pourquoi ?

1. dig SOA *grpX*.<*lab_domain*>.te-labs.training @100.100.X.66
2. dig SOA *grpX*.<*lab_domain*>.te-labs.training @100.100.X.66 +dnssec
3. dig DNSKEY *grpX*.<*lab_domain*>.te-labs.training @100.100.X.66
4. dig DNSKEY *grpX*.<*lab_domain*>.te-labs.training @100.100.X.66 +dnssec +multi
5. dig DNSKEY *grpX*.<*lab_domain*>.te-labs.training @100.100.X.130
6. dig DNSKEY *grpX*.<*lab_domain*>.te-labs.training @100.100.X.130 +dnssec +multi
7. dig SOA *grpX*.<*lab_domain*>.te-labs.training @100.100.X.130 +dnssec +multi



**QUESTIONS**: Avez-vous obtenu les réponses à toutes les requêtes ? Avez-vous observé les signatures ? Avez-vous obtenu le drapeau "**ad**" ? Pourquoi ?
