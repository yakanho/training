<img src="https://github.com/yakanho/training/assets/54844453/321060e5-fc84-40f7-8caa-846d0a68494b" alt="ICANN" style="zoom:25%;" />

# Lab : Durcir votre configuration DNS (Introduction)

```
Créé par : Yazid AKANHO
version : 2024042400
Révision précédente : 2024030100
Modifié par : -
```


Maintenant que la configuration de votre zone fonctionne correctement (si ce n'est pas le cas, ne continuez pas ici et retournez corriger la configuration avant de poursuivre), nous devons apporter quelques améliorations de sécurité afin de protéger notre DNS contre certains types d'attaques.

## Transfert de zone ouvert

Essayez d'effectuer un transfert de zone manuel depuis notre machine cliente, en utilisant successivement l'adresse IP de ns1, de ns2 et du serveur SOA. Que remarquez-vous ? 

Astuce : la commande à utiliser est ***dig axfr <*domain*> @nameserver***. 


Faites quelques recherches sur Internet et échangez avec vos camarades pour trouver une solution permettant d'éviter un transfert de zone ouvert.

#### Corriger le transfert de zone ouvert

Si votre réponse à la question ci-dessus est ***allow-transfer***, alors vous avez raison.
Appliquez la modification de configuration appropriée pour corriger le problème évoqué ci-dessus.
Testez ensuite à nouveau le transfert de zone. Le problème est-il résolu ?

> Remarque : allow-transfer n'est pas la seule façon d'y parvenir. D'autres méthodes existent.


## Primaire caché

Votre serveur primaire caché est-il réellement « caché » ? Essayez d'interroger votre zone sur ce serveur depuis n'importe quelle autre VM et confirmez s'il répond ou non. Après ce test, pouvez-vous confirmer qu'il est réellement « caché » ?

Discutez avec les autres participants pour trouver une solution afin d'en faire un véritable serveur de noms primaire « caché » ; c'est-à-dire qu'il ne doit pas répondre aux requêtes DNS, puisqu'il est le primaire et n'est pas censé servir la zone publiquement. Il s'agit d'une bonne pratique en DNS. Cependant, n'oubliez pas que, pour des besoins de dépannage, vous pouvez toujours avoir besoin de l'interroger depuis certaines adresses IP spécifiques (ou connues).

Appliquez la solution une fois qu'un consensus a été trouvé avec les formateurs.
