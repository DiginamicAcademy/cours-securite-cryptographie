[← Chapitre précédent](04-hachage-et-signature.md) · [Sommaire](Readme.md) · [Chapitre suivant →](06-alea-et-cles.md)

# Les erreurs classiques de cryptographie

> Bruce Schneier, qui a passé sa carrière à concevoir et à casser des algorithmes,
> résume une expérience commune : « toute personne peut créer un algorithme qu'elle ne
> peut pas casser elle-même ». Un chiffrement qui résiste à son auteur ne résiste pas
> pour autant à qui que ce soit d'autre.

**Durée de lecture :** 40 min hors approfondissements facultatifs · **Prérequis :** [Du chiffre de César à AES](02-chiffrement-symetrique.md), [Le hachage et la signature numérique](04-hachage-et-signature.md)

---

## Objectifs

À l'issue de ce chapitre, vous serez capable de :

- **Repérer** un chiffrement inventé sur mesure et **proposer** une bibliothèque standard.
- **Identifier** les algorithmes et modes obsolètes dans un code ou une configuration.
- **Détecter** les fuites indirectes : comparaison non constante, nonce réutilisé, sel absent.
- **Hiérarchiser** les corrections à apporter à un module qui traite des données sensibles.

---

## I. Inventer son propre chiffrement

Le réflexe est universel : il faut cacher une donnée, on écrit une petite fonction —
décalage, mélange de caractères, XOR (ou exclusif) maison — et cela « marche » : le
résultat est illisible. Le problème n'apparaît jamais pendant le développement, mais le
jour où quelqu'un d'autre s'y intéresse.

Trois raisons font échouer systématiquement les chiffrements inventés :

- **On ne voit pas ses propres faiblesses.** L'auteur teste contre ses propres idées
  d'attaque — précisément celles que son algorithme résiste déjà.
- **Les motifs survivent.** Un chiffrement maison laisse presque toujours filtrer la
  longueur, les répétitions ou les fréquences — exactement ce que l'analyse de fréquence
  exploite.
- **L'implémentation trahit l'algorithme.** Même un bon algorithme tombe s'il est
  codé avec un générateur prévisible, une clé en dur, ou une comparaison qui fuite.

> [!DANGER]
> « Ne roule jamais ta propre crypto » (*don't roll your own crypto*) est l'une des
> rares règles absolues du domaine. On utilise des bibliothèques éprouvées
> (`cryptography`, OpenSSL, les API système), et on se concentre sur la gestion des
> clés — qui, elle, ne se délègue pas.

> 🧪 **Atelier rapide (8 min)**
>
> Ce fragment est extrait d'une application de fidélité qui génère des jetons de
> connexion. Repérez ce qui ne va pas, puis corrigez-le :
>
```python
import random


def generer_jeton():
    random.seed()  # graine tirée de l'horloge
    return "".join(
        random.choice("0123456789abcdef") for _ in range(16)
    )
```
>
> 1. Listez les problèmes de sécurité de ce code (il y en a deux majeurs).
> 2. Remplacez-le par une version qui utilise `secrets`.
> 3. Expliquez en une phrase pourquoi un jeton généré ainsi ne peut pas être deviné.

---

## II. Les algorithmes et modes obsolètes

Un code ancien n'est pas un code sûr. Les fonctions et les modes ci-dessous traînent
encore dans des applications en production : leur présence est un signal d'alarme.

| Ce qu'on trouve | Pourquoi c'est un problème | Remplacement |
| --- | --- | --- |
| MD5 ou SHA-1 pour hacher un mot de passe ou vérifier un contenu | Collisions fabriquées depuis des années | SHA-256, SHA-3, ou une fonction de hachage de mot de passe |
| Mode ECB (Electronic Codebook) | Deux blocs identiques donnent le même chiffré : les motifs réapparaissent | CBC (Cipher Block Chaining) ou GCM (Galois/Counter Mode) |
| DES (Data Encryption Standard) | Clé de 56 bits : cassée par énumération | AES (Advanced Encryption Standard) |
| RC4, WEP (Wired Equivalent Privacy) | Flot de clé faible, attaques pratiques | WPA2/WPA3, ou TLS (Transport Layer Security) à jour |
| Clé ou mot de passe en dur dans le code | Le secret est dans le dépôt, donc chez quiconque y accède | Coffre de secrets, variable d'environnement injectée |

### Le mode ECB, en une image

Le mode ECB chiffre chaque bloc indépendamment avec la même clé. Conséquence
spectaculaire : sur une image comportant de grandes zones de même couleur, les blocs
identiques donnent des chiffrés identiques, et l'image reste **reconnaissable dans le
chiffré** — le contour du sujet apparaît en pixels chiffrés. Chiffré en ECB, le pingouin
célèbre reste un pingouin.

> [!WARNING]
> ECB ne fuit pas « un peu » : il fuit la structure complète des données répétées. Sur
> des données structurées — un fichier CSV (comma-separated values), un export JSON
> (JavaScript Object Notation), une image — la révélation est immédiate.

### Les mots de passe ne se chiffrent pas

Une confusion coûteuse : **chiffrer** les mots de passe pour les stocker. C'est garder
une clé qui rend tout lisible en cas de fuite de la base. On stocke une **empreinte
salée** calculée par une fonction lente dédiée — on ne « retrouve » jamais le mot de
passe, on le vérifie.

---

## III. Les fuites indirectes

Le plus difficile à repérer n'est pas l'algorithme mais ce qui l'entoure : un
chiffrement impeccable se casse par le comportement du programme qui l'utilise.

### La comparaison qui fuite par chronométrage

Comparer deux empreintes avec `==` s'arrête au premier caractère différent : en mesurant
le temps de réponse, un attaquant reconstruit la valeur attendue caractère par
caractère. La parade est `hmac.compare_digest`, qui met toujours le même temps.

### Le nonce réutilisé

Un IV ou un nonce réutilisé avec la même clé ruine le chiffrement : en GCM, cela révèle
la relation entre deux messages et invalide l'authentification. Le nonce se tire au
hasard, ou se compte sans jamais se répéter.

### Le sel absent

Deux utilisateurs qui choisissent le même mot de passe produisent la même empreinte :
on repère les mots de passe fréquents dans la base et on attaque tout le monde en une
passe. Le **sel** — valeur aléatoire propre à chaque utilisateur, ajoutée avant le
hachage — rend chaque empreinte unique.

### Les messages d'erreur trop bavards

« Clé incorrecte », « format du chiffré invalide », « signature non valide » : chaque
différence de message ou de temps de réponse renseigne l'attaquant. Un service qui
déchiffre doit répondre de façon uniforme, sans distinguer ce qui a échoué.

> [!TIP]
> En revue de code, passez la liste courte : bibliothèque standard ou pas ? clé
> d'où vient-elle ? nonce unique ? comparaison constante ? sel présent ? réponse
> uniforme ? Six questions couvrent l'essentiel des erreurs réelles.

---

## IV. Aller plus loin

*Ces approfondissements sont facultatifs : ils ne sont jamais exigés pour terminer la
journée.*

- **Les attaques par canal auxiliaire** : le chronométrage n'est qu'un cas. La
  consommation électrique, les émissions électromagnétiques, ou même le bruit d'un
  ventilateur ont été exploités pour extraire des clés. C'est un domaine à part
  entière, mais son principe est celui de la section III : le comportement trahit le
  secret.
- **Les failles de protocole** : une implémentation parfaite d'un protocole mal conçu
  reste vulnérable. Les publications de failles CVE (*Common Vulnerabilities and
  Exposures*) sur les bibliothèques crypto montrent que la vigilance porte autant sur la
  version utilisée que sur le code écrit.
- **La dette crypto** : remplacer MD5 dans un système ancien oblige parfois à migrer des
  données existantes. Anticiper — en versionnant les empreintes, en acceptant plusieurs
  formats à la lecture — évite de se retrouver prisonnier d'un algorithme obsolète.
- **Évaluer un choix d'algorithme** : les recommandations de l'ANSSI (Agence nationale
  de la sécurité des systèmes d'information) et du NIST (National Institute of Standards
  and Technology) sont publiques, datées, et révisées. S'y référer vaut mieux qu'un avis
  de forum.

---

## Points clés

- **Ne jamais inventer son propre chiffrement** : utiliser une bibliothèque éprouvée, et
  concentrer l'effort sur la gestion des clés.
- ECB, MD5, SHA-1, DES, RC4 sont **obsolètes** : leur présence dans un code est un
  défaut, pas une question de goût.
- Les mots de passe se stockent en **empreinte salée**, jamais chiffrés.
- Les fuites les plus fréquentes sont **indirectes** : comparaison non constante, nonce
  réutilisé, sel absent, messages d'erreur trop précis.
- Une revue de code crypto tient en six questions : bibliothèque, origine de la clé,
  unicité du nonce, comparaison constante, sel, uniformité des réponses.

---

[← Chapitre précédent](04-hachage-et-signature.md) · [Sommaire](Readme.md) · [Chapitre suivant →](06-alea-et-cles.md)
