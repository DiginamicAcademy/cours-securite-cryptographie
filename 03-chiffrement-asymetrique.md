[← Chapitre précédent](02-chiffrement-symetrique.md) · [Sommaire](Readme.md) · [Chapitre suivant →](04-hachage-et-signature.md)

# Le chiffrement asymétrique

> Au XVe siècle, deux rois qui veulent s'entendre échangent leurs codes par des
> messagers — et chaque messager capturé compromet tout l'édifice : la sécurité repose
> sur une clé qui a voyagé. En 1977, trois chercheurs inventent un procédé qui
> supprime ce voyage : une clé que l'on **publie** sans compromettre le secret. C'est
> la naissance du chiffrement asymétrique.

**Durée de lecture :** 70 min · **Prérequis :** [Du chiffre de César à AES](02-chiffrement-symetrique.md)

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

Le chiffrement symétrique est rapide, simple, et parfaitement sûr — à une condition :
que l'émetteur et le destinataire possèdent **déjà** la même clé. Or cette clé, il
faut bien la faire parvenir. Par un messager, un courriel, une clé de stockage : autant
d'occasions de la perdre.

Le calcul est sans pitié. Dans une équipe de 10 personnes qui doivent toutes
communiquer deux à deux, il faut distribuer 45 clés distinctes ; à 100 personnes, ce
sont 4 950 clés à créer, transmettre, stocker et révoquer. Et chaque nouvelle personne
rejointe oblige à renouveler une partie du stock.

> [!WARNING]
> La distribution de clés est le talon d'Achille du chiffrement symétrique. Un
> algorithme inviolable ne protège rien si la clé a transité par un canal qu'on ne
> maîtrise pas.

L'image de la **boîte à deux cadenas**, déjà rencontrée plus tôt, donne la sortie :
pour échanger un objet sans qu'un cadenas ouvert ne circule, chacun pose le sien. Il
manque encore une pièce — un cadenas dont la **clé d'ouverture ne quitte jamais son
propriétaire**, alors que le cadenas, lui, circule librement.

---

## II. La paire de clés : publique et privée

C'est l'invention de Whitfield Diffie et Martin Hellman en 1976, concrétisée en 1977
par Ronald Rivest, Adi Shamir et Leonard Adleman au MIT (Massachusetts Institute of
Technology) : **RSA (Rivest-Shamir-Adleman)**. Chaque utilisateur génère une **paire de
clés** :

- la **clé publique** est diffusée sans risque — c'est le cadenas ouvert que tout le
  monde peut poser ;
- la **clé privée** reste secrète — c'est la seule clé qui ouvre le cadenas.

Les deux sont mathématiquement liées, mais on ne déduit pas l'une de l'autre : la
sécurité de RSA repose sur la difficulté de décomposer un très grand nombre en produit
de deux nombres premiers. Un ordinateur peut multiplier deux grands nombres premiers en
un instant ; revenir en arrière prendrait, avec les moyens actuels, des milliards
d'années.

### Ce que chaque clé autorise

| Opération | Avec quelle clé | Qui peut le faire | Service rendu |
| --- | --- | --- | --- |
| Chiffrer un message | Clé publique du destinataire | N'importe qui | Confidentialité |
| Déchiffrer | Clé privée du destinataire | Le destinataire seul | Confidentialité |
| Signer | Clé privée de l'émetteur | L'émetteur seul | Authentification, non-répudiation |
| Vérifier une signature | Clé publique de l'émetteur | N'importe qui | Intégrité, preuve d'origine |

Le point décisif : **chiffrer avec la clé publique ne permet pas de revenir au clair
sans la clé privée**. On peut donc publier la clé publique sur un site, dans un
annuaire, au dos d'une carte de visite — sans rien compromettre.

> 🧪 **Atelier rapide (10 min)**
>
> 1. Générez une paire de clés sur votre machine :
>
> ```bash
> ssh-keygen -t ed25519 -f /tmp/cle-demo -N ""
> ```
>
> 2. Affichez la clé publique avec `cat /tmp/cle-demo.pub`, puis la clé privée avec
> `cat /tmp/cle-demo`. **SSH (Secure Shell)**, le protocole d'administration à
> distance, utilise exactement ce mécanisme.
> 3. Comparez : la clé publique tient sur une ligne et se partage sans risque ; la
> clé privée s'ouvre sur un en-tête qui annonce une clé privée, et ne doit jamais
> quitter la machine.
> 4. Notez la longueur de chaque fichier. Demandez-vous ce qu'un attaquant pourrait
> faire s'il obtenait l'un ou l'autre.

### La lenteur, contrepartie du confort

RSA est **lent** : chiffrer un fichier de plusieurs mégaoctets prendrait un temps
prohibitif, et la taille du message est bornée par celle de la clé. On ne chiffre donc
jamais les données directement avec RSA. On chiffre **une clé** :

1. l'émetteur tire au hasard une clé de session symétrique ;
2. il chiffre les données avec cette clé, en AES (Advanced Encryption Standard) mode
   GCM (Galois/Counter Mode) — rapide et authentifiant ;
3. il chiffre la seule clé de session avec la clé publique du destinataire ;
4. il envoie le tout : données chiffrées + clé de session chiffrée.

C'est le principe **hybride** : l'asymétrique résout la distribution de la clé, le
symétrique assure le volume. La quasi-totalité des échanges protégés d'aujourd'hui —
TLS (Transport Layer Security), messageries chiffrées, sauvegardes — fonctionne ainsi.

---

## III. Les courbes elliptiques et l'échange de clés

RSA n'est pas la seule famille asymétrique. Depuis les années 2000, les **courbes
elliptiques**, ou ECC (*Elliptic Curve Cryptography*) offrent la même sécurité avec des
clés bien plus courtes : une clé de 256 bits en courbes elliptiques équivaut à une clé
RSA de plusieurs milliers de bits. Conséquences concrètes : calculs plus rapides,
certificats plus légers, échanges plus courts — d'où leur adoption massive dans les
protocoles récents.

### Se mettre d'accord sans se parler

Reste une question : deux machines qui ne se connaissent pas doivent-elles pour autant
échanger une clé publique ? Non. L'**échange de clés de Diffie-Hellman** permet à deux
parties de calculer un secret commun sur un canal public, sans que l'observateur qui
écoute la conversation ne puisse le reconstituer. Chaque partie combine sa propre valeur
secrète avec la valeur publique de l'autre ; les deux calculs aboutissent au même
résultat, que seul un participant peut obtenir.

C'est le cœur de la poignée de main TLS : deux serveurs qui ne se sont jamais vus
conviennent d'une clé de session en quelques messages, puis basculent en chiffrement
symétrique. L'asymétrique sert à **se rencontrer**, le symétrique à **discuter**.

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

- Le chiffrement symétrique exige une clé partagée : **la distribuer est son point
  faible**, surtout quand le nombre d'interlocuteurs augmente.
- Le chiffrement asymétrique sépare une **clé publique** (que l'on diffuse) d'une
  **clé privée** (que l'on ne quitte jamais) : la distribution de clés devient un
  non-problème.
- Chiffrer avec la clé publique du destinataire garantit que **lui seul** déchiffre ;
  signer avec sa clé privée prouve **qu'il est bien l'auteur**.
- RSA repose sur la difficulté de factoriser ; les **courbes elliptiques** offrent la
  même sécurité avec des clés plus courtes et des calculs plus rapides.
- En pratique, on chiffre **une clé de session** avec l'asymétrique, et les **données**
  avec le symétrique : c'est le principe hybride de TLS et des messageries chiffrées.
- La clé privée est le seul secret du système : sa protection et sa rotation priment
  sur le choix de l'algorithme.

---

[← Chapitre précédent](02-chiffrement-symetrique.md) · [Sommaire](Readme.md) · [Chapitre suivant →](04-hachage-et-signature.md)
