[← Chapitre précédent](02-chiffrement-symetrique.md) · [Sommaire](Readme.md) · [Chapitre suivant →](04-hachage-et-signature.md)

# Le chiffrement asymétrique

> Deux rois veulent s'entendre et échangent leurs codes par messagers ; chaque
> messager capturé compromet tout l'édifice : la sécurité repose sur une clé qui a
> voyagé. En 1977, un procédé supprime ce voyage — une clé que l'on **publie** sans
> compromettre le secret.

**Durée de lecture :** 45 min hors approfondissements facultatifs · **Prérequis :** [Du chiffre de César à AES](02-chiffrement-symetrique.md)

---

## Objectifs

À l'issue de ce chapitre, vous serez capable de :

- **Expliquer** pourquoi la distribution des clés est le point faible du chiffrement symétrique.
- **Découpler** clé publique et clé privée, et **décrire** ce que chacune autorise.
- **Comparer** RSA et les courbes elliptiques sur la taille de clé et l'usage.
- **Reproduire** un échange de clés entre deux parties qui ne se sont jamais parlé.
- **Assembler** chiffrement symétrique et asymétrique dans un scénario hybride.

---

## I. Le problème de l'échange de clés

Le chiffrement symétrique est rapide, simple, parfaitement sûr — à une condition : que
l'émetteur et le destinataire possèdent **déjà** la même clé. Or il faut bien la faire
parvenir, et un messager, un courriel ou une clé de stockage sont autant d'occasions de
la perdre.

Le calcul est sans pitié : dix personnes qui communiquent deux à deux doivent distribuer
45 clés distinctes ; à cent, ce sont 4 950 clés à créer, transmettre, stocker et
révoquer — et chaque arrivée renouvelle une partie du stock.

> [!WARNING]
> La distribution de clés est le talon d'Achille du chiffrement symétrique. Un
> algorithme inviolable ne protège rien si la clé a transité par un canal qu'on ne
> maîtrise pas.

La **boîte à deux cadenas** du chapitre précédent donne la sortie : chacun pose le
sien, aucun cadenas ne circule ouvert. Il manque une pièce — un cadenas dont la **clé
d'ouverture ne quitte jamais son propriétaire**, alors que le cadenas circule librement.

---

## II. La paire de clés : publique et privée

C'est l'invention de Whitfield Diffie et Martin Hellman (1976), concrétisée en 1977 par
Ronald Rivest, Adi Shamir et Leonard Adleman — **RSA (Rivest-Shamir-Adleman)**. Chaque
utilisateur génère une **paire de clés** :

- la **clé publique** se diffuse sans risque — c'est le cadenas ouvert que tout le monde
  peut poser ;
- la **clé privée** reste secrète — c'est la seule qui l'ouvre.

Liées mathématiquement, mais l'une ne se déduit pas de l'autre : la sécurité de RSA
repose sur la difficulté de décomposer un très grand nombre en produit de deux nombres
premiers. Multiplier deux grands premiers prend un instant ; revenir en arrière prendrait
des milliards d'années.

### Ce que chaque clé autorise

| Opération | Avec quelle clé | Qui peut le faire | Service rendu |
| --- | --- | --- | --- |
| Chiffrer un message | Clé publique du destinataire | N'importe qui | Confidentialité |
| Déchiffrer | Clé privée du destinataire | Le destinataire seul | Confidentialité |
| Signer | Clé privée de l'émetteur | L'émetteur seul | Authentification, non-répudiation |
| Vérifier une signature | Clé publique de l'émetteur | N'importe qui | Intégrité, preuve d'origine |

Le point décisif : **sans la clé privée, on ne revient pas au clair**. On peut donc
publier la clé publique sur un site ou dans un annuaire sans rien compromettre.

> 🧪 **Atelier rapide (10 min)**
>
> 1. Générez une paire de clés sur votre machine :
>
> ```bash
> ssh-keygen -t ed25519 -f /tmp/cle-demo -N ""
> ```
>
> 2. Affichez la clé publique (`cat /tmp/cle-demo.pub`) puis la clé privée
> (`cat /tmp/cle-demo`). **SSH (Secure Shell)**, le protocole d'administration à
> distance, utilise exactement ce mécanisme.
> 3. Comparez : la publique tient sur une ligne et se partage sans risque ; la privée
> s'annonce comme telle et ne doit jamais quitter la machine.
> 4. Demandez-vous ce qu'un attaquant pourrait faire de l'une ou de l'autre.

### La lenteur, contrepartie du confort

RSA est **lent**, et la taille du message est bornée par celle de la clé : on ne
chiffre donc jamais les données directement avec RSA. On chiffre **une clé** :

1. l'émetteur tire au hasard une clé de session symétrique ;
2. il chiffre les données avec cette clé, en AES (Advanced Encryption Standard) mode
   GCM (Galois/Counter Mode) — rapide et authentifiant ;
3. il chiffre la seule clé de session avec la clé publique du destinataire ;
4. il envoie le tout : données chiffrées + clé de session chiffrée.

C'est le principe **hybride** : l'asymétrique résout la distribution, le symétrique
assure le volume. TLS (Transport Layer Security), les messageries chiffrées et les
sauvegardes fonctionnent ainsi.

---

## III. Les courbes elliptiques et l'échange de clés

RSA n'est pas la seule famille asymétrique. Depuis les années 2000, les **courbes
elliptiques**, ou ECC (*Elliptic Curve Cryptography*), offrent la même sécurité avec des
clés bien plus courtes : 256 bits en ECC valent plusieurs milliers de bits RSA. D'où,
dans les protocoles récents, des calculs plus rapides et des certificats plus légers.

### Se mettre d'accord sans se parler

Deux machines qui ne se connaissent doivent-elles au moins échanger une clé publique ?
Non. L'**échange de clés de Diffie-Hellman** fait calculer à deux parties un secret
commun sur un canal public, sans que l'observateur qui écoute puisse le reconstituer :
chacune combine sa valeur secrète avec la valeur publique de l'autre, et les deux
calculs aboutissent au même résultat.

C'est le cœur de la poignée de main TLS : on convient d'une clé de session en quelques
messages, puis on bascule en symétrique. L'asymétrique sert à **se rencontrer**, le
symétrique à **discuter**.

> [!TIP]
> En pratique, on retient trois temps : **authentifier** l'interlocuteur (certificat,
> signature), **convenir** d'une clé de session (échange de clés), puis **chiffrer** les
> données (symétrique). Chaque temps répond à une question différente.

### Ce qu'il faut surveiller

- **La clé privée est le seul secret.** Elle se stocke avec des droits restrictifs, se
  sauvegarde chiffrée, et ne transite jamais — ni par courriel, ni par dépôt de code.
- **Une paire de clés se renouvelle.** Une clé privée fuite ou devient suspecte : on la
  révoque et on en génère une nouvelle, plutôt que de négocier avec le doute.
- **La taille de clé suit les recommandations à jour.** Les tailles acceptables
  augmentent avec la puissance de calcul : ce qui suffisait il y a dix ans peut être
  court aujourd'hui.

---

## IV. Aller plus loin

*Ces approfondissements sont facultatifs : ils ne sont jamais exigés pour terminer la
journée.*

- **Pourquoi RSA résiste** : décomposer un nombre de plusieurs centaines de chiffres en
  produit de deux nombres premiers est, avec les algorithmes connus, hors de portée des
  machines actuelles. La recherche en théorie des nombres dicte directement la durée de
  vie des tailles de clés.
- **L'horizon quantique** : un ordinateur quantique suffisamment grand casserait RSA et
  l'échange de clés classique avec l'algorithme de Shor. La cryptographie post-quantique
  — déjà standardisée en partie — prépare des algorithmes résistants ; les systèmes
  d'aujourd'hui sont évalués sur leur capacité à migrer.
- **Signer avec une courbe elliptique** : les signatures Ed25519 sont rapides, de taille
  fixe, et utilisées par défaut dans les outils récents. Comparer une signature RSA et
  une signature Ed25519 sur le même fichier est un bon exercice de sensibilisation.
- **Le chiffrement de bout en bout** : dans une messagerie chiffrée, les clés vivent sur
  les appareils des utilisateurs, pas sur le serveur. Le serveur transporte des messages
  qu'il ne peut pas lire — ce qui pose aussi la question de la perte de l'appareil.

---

## Points clés

- Le symétrique exige une clé partagée : **la distribuer est son point faible**,
  surtout quand les interlocuteurs se multiplient.
- L'asymétrique sépare **clé publique** (diffusée) et **clé privée** (jamais quittée) :
  la distribution devient un non-problème.
- Chiffrer avec la publique du destinataire garantit que **lui seul** déchiffre ; signer
  avec sa privée prouve **qu'il est l'auteur**.
- RSA repose sur la factorisation ; les **courbes elliptiques** donnent la même sécurité
  avec des clés plus courtes.
- On chiffre **une clé de session** en asymétrique et les **données** en symétrique :
  c'est le principe hybride de TLS.
- La clé privée est le seul secret : sa protection prime sur le choix de l'algorithme.

---

## 🧠 Vérifiez vos acquis

*Formatif, jamais noté. Répondez sans relire le chapitre, puis cochez. Les réponses sont juste après le séparateur : ce qui compte est de repérer où vous vacillez, pas de faire 5 sur 5.*

**1.** Cent personnes doivent communiquer deux à deux en chiffrement symétrique. Combien de clés distinctes faut-il créer, transmettre, stocker et révoquer ?

- A. 100
- B. 200
- C. 4 950
- D. Autant que nécessaire, mais une seule suffit si elles sont longues

- [ ] ✓ je savais

**2.** Alice veut envoyer un document que Bob est le seul à pouvoir lire, en asymétrique. Quelle clé utilise-t-elle pour chiffrer ?

- A. Sa propre clé publique
- B. La clé publique de Bob
- C. La clé privée de Bob
- D. La clé privée d'Alice

- [ ] ✓ je savais

**3.** Pourquoi ne chiffre-t-on jamais les données directement avec RSA ?

- A. Parce que RSA est lent et que la taille du message est bornée par celle de la clé
- B. Parce que RSA n'est pas un algorithme public
- C. Parce que RSA ne chiffre que des nombres premiers
- D. Parce que RSA ne chiffre que des clés de 128 bits

- [ ] ✓ je savais

**4.** Que faut-il retenir des courbes elliptiques (ECC) par rapport à RSA ?

- A. Elles sont plus sûres à taille de clé égale, mais incompatibles avec TLS
- B. Elles offrent la même sécurité avec des clés bien plus courtes
- C. Elles suppriment le besoin de clé privée
- D. Elles remplacent l'échange de clés par la signature

- [ ] ✓ je savais

**5.** *Question ouverte (2-3 lignes).* Deux machines qui ne se sont jamais parlé conviennent d'une clé de session sur un canal public, que l'observateur ne peut pas reconstituer. Nommez ce mécanisme, puis replacez-le dans les trois temps du cours : que fait chacun des trois ?

- [ ] ✓ je savais

---

### Réponses

**1. C.** 4 950 clés : le nombre de paires croît bien plus vite que le nombre d'interlocuteurs (dix personnes en demandent déjà 45).
**Pourquoi :** c'est le talon d'Achille du symétrique — la distribution. Un algorithme inviolable ne protège rien si la clé a transité par un canal non maîtrisé.

**2. B.** La clé publique du destinataire : n'importe qui peut chiffrer, mais **lui seul** déchiffre, avec sa clé privée.
**Pourquoi :** sans la clé privée, on ne revient pas au clair — c'est ce qui permet de publier la clé publique sans rien compromettre.

**3. A.** RSA est lent et borné en taille : on chiffre **une clé de session** avec la clé publique du destinataire, et les données avec cette clé en AES-GCM.
**Pourquoi :** c'est le principe **hybride** — l'asymétrique résout la distribution, le symétrique assure le volume.

**4. B.** 256 bits en ECC valent plusieurs milliers de bits RSA, d'où des calculs plus rapides et des certificats plus légers.
**Pourquoi :** la différence est la taille de clé pour une sécurité équivalente, pas un changement de rôle — la clé privée reste le seul secret.

**5.** Attendu : l'échange de clés de **Diffie-Hellman** — chacune combine sa valeur secrète avec la valeur publique de l'autre et les deux calculs aboutissent au même résultat. Les trois temps : **authentifier** l'interlocuteur (certificat, signature), **convenir** d'une clé de session (échange de clés), puis **chiffrer** les données en symétrique.
**Pourquoi :** l'asymétrique sert à se rencontrer, le symétrique à discuter — chaque temps répond à une question différente.

---

[← Chapitre précédent](02-chiffrement-symetrique.md) · [Sommaire](Readme.md) · [Chapitre suivant →](04-hachage-et-signature.md)
