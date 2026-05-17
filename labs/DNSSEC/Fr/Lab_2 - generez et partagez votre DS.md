<img src="https://github.com/yakanho/training/assets/54844453/321060e5-fc84-40f7-8caa-846d0a68494b" alt="ICANN" style="zoom:25%;" />

------

# Lab - Générez la délégation de signature (DS) de votre domaine et soumettre à votre parent

```
Créé par: Yazid AKANHO
Modifié par: -
version courante: 2026051700
Version antérieure:-
```

------



Pour rappel, votre nom de domaine est *grpX.<lab_domain>.te-labs.training* et celui de votre parent est  *<lab_domain>.te-labs.training*.

La première tâche de ce lab consistera à créer l'enregistrement DS de votre zone et l'enregistrer dans un fichier. Pour cela, créez un répertoire dédié pour vos DS

```
$ sudo mkdir -p /var/lib/bind/ds
$ sudo chown -R bind:bind /var/lib/bind/ds
```



### Générez l'enregistrement DS

> [!IMPORTANT]
> Pour générer l'enregistrement DS, vous devrez remplacer les lettres "XYZ" et "YOUR-KSK-key_ID" dans la commande ci-dessous par leurs valeurs correctes en fonction de votre algorithme de signature et l'ID de votre KSK.
>
> Par exemple, vous obtiendrez `/var/lib/bind/keys/KgrpX.<lab_domain>.te-labs.training.+013+21554.key |sudo tee /var/lib/bind/ds/DS_21554.grpX` si vous avez utilisé l'algorithme de signature DNSSEC 13 et si la KSK générée par votre signer est 21554.

Exécutez donc la commande suivante pour obtenir l'enregistrement DS et le sauvegarder dans un fichier dans votre nouveau répertoire des DS:

```
$ dnssec-dsfromkey /var/lib/bind/keys/KgrpX.<lab_domain>.te-labs.training.+XYZ+YOUR-KSK-key-tag.key |sudo tee /var/lib/bind/ds/DS_YOUR-KSK-key-tag.grpX
```



Une autre option serait de générer l'enregistrement directement depuis l'enregistrement DNSKEY en interrogeant votre domaine avec l'option ""-f". 

```
$ dig @localhost dnskey grpX.<lab_domain>.te-labs.training | dnssec-dsfromkey -f - grpX.<lab_domain>.te-labs.training |sudo tee /var/lib/bind/ds/DS_YOUR-KSK-key-tag.grpX
```



Vérifiez le contenu de votre fichier afin de confirmer l'enregistrement DS :

```
$ cat /var/lib/bind/ds/DS_YOUR-KSK-key-tag.grpX
grpX.<lab_domain>.te-labs.training. IN DS 42330 13 2 6376D6757675DC67535CD3546D9CE6B531E542417BA8B80DC550F793C2DC61AB
```

Which should contain something similar to the following line:

```
grpX.<lab_domain>.te-labs.training. IN DS YOUR-KSK-key-tag 8 2 018A86C0139BA5500AC87A5BAD8FB5D8D4F9672C319B34DB5A7F3BC10A424D6E
```



### Envoyez l'enregistrement DS au registre

Afin d'établir la chaine de confiance DNSSEC et rendre opérationelle la signature de votre zone, il est nécessaire de partager votre enregistrement DS à votre registre (parent) afin qu'il le publie dans sa zone (comme vos NS et glue records par exemple). 

Pour cela, copiez la ligne de votre enregistrement DS et collez la dans le champ dédié dans la page d'accueil de votre lab `https://<*lab_domain*>.te-labs.training/grpX` et patientez 5 minutes avant de continuer les prochaines étapes.



### Vérifiez la publication de votre DS par votre parent

Utilisez la commande "dig" pour vérifier si votre enregistrement DS est publié par votre parent.

```
$ dig DS grpX.<*lab_domain*>.te-labs.training 

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
grpX.<*lab_domain*>.te-labs.training. 60    IN    DS      42330 13 2 6376D6757675DC67535CD3546D9CE6B531E542417BA8B80DC550F793 C2DC61AB

;; Query time: 12 msec
;; SERVER: 100.100.35.67#53(100.100.35.67) (UDP)
;; WHEN: Sun May 17 19:02:07 UTC 2026
;; MSG SIZE  rcvd: 133
```



A ce stade, vous venez de dire au monde entier que votre zone est signée DNSSEC et que tout resolver validant (resolver récursif avec la  **validation DNSSEC active**) devra effectuer la validation DNSSEC pour les réponses issues de votre zone. 

Vous pouvez donc à nouveau interroger votre zone en utilisant les mêmes requêtes que dans le lab précédent et voir si vous obtenez cette fois le drapeau "ad". Pourquoi ?
