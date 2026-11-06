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

### Exercice 1 — Auditer un code qui utilise mal la crypto (50 min)

#### Contexte

**Cinéma Le Palace**, un réseau de trois salles de projection, fait relire son
application de réservation avant une nouvelle mise en production. L'application gère
des comptes clients, des jetons de session et un export de données envoyé à un
partenaire publicitaire. Vous recevez l'extrait ci-dessous, extrait du module
`booking_api.py` :

```python
# booking_api.py — application de réservation
import hashlib
import random

from Crypto.Cipher import AES

CLE = "0123456789abcdef"  # même clé pour toute l'application

def empreinte_mot_de_passe(mot_de_passe):
    """Stocke l'empreinte du mot de passe en base."""
    return hashlib.md5(mot_de_passe.encode()).hexdigest()

def chiffrer_export(donnees):
    """Chiffre l'export envoyé au partenaire publicitaire."""
    chiffreur = AES.new(CLE.encode(), AES.MODE_ECB)
    return chiffreur.encrypt(donnees)

def generer_jeton():
    """Génère un jeton de session."""
    random.seed()
    return "".join(random.choice("0123456789") for _ in range(12))

def verifier_jeton(jeton, attendu):
    """Vérifie le jeton présenté par le client."""
    return jeton == attendu
```

La politique interne de l'entreprise impose :

- mots de passe stockés en **empreinte salée** (sel aléatoire de **16 octets**), calculée par une fonction lente, avec au
  moins **100 000** itérations ;
- jetons de session de **32 octets**, tirés par un générateur adapté à la
  cryptographie ;
- export chiffré en mode **authentifiant**, avec une valeur de départ unique par
  message ;
- vérifications faites à **temps constant**, quel que soit le résultat.

#### Consignes

##### Partie 1 — Recenser les usages

Pour chacune des quatre fonctions, dites en une phrase **ce qu'elle protège** (un mot
de passe, un fichier, une session) et **ce qu'elle garantit** — ou ne garantit pas.

##### Partie 2 — Auditer

Remplissez le tableau ci-dessous. Une ligne par problème trouvé.

| Fonction | Problème | Risque concret | Correction proposée |
| --- | --- | --- | --- |
| | | | |

Visez au moins **cinq** problèmes distincts — il y en a dans chaque fonction, et un
dans les constantes du module.

##### Partie 3 — Corriger

Réécrivez deux fonctions au choix, en respectant la politique interne. Vous pouvez
utiliser la bibliothèque standard de Python (`hashlib`, `hmac`, `secrets`) : aucune
installation supplémentaire n'est nécessaire.

##### Partie 4 — Hiérarchiser

L'équipe n'a pas le temps de tout corriger avant la mise en production. Classez vos
constats du plus urgent au moins urgent, et justifiez votre premier choix en deux
phrases.

#### Attendu

- Un tableau d'audit d'au moins cinq lignes, rempli.
- Deux fonctions corrigées, exécutables.
- Un classement justifié des corrections.

#### Vérification

Vous avez réussi quand :

- chaque fonction du module apparaît au moins une fois dans votre tableau ;
- vos corrections respectent les quatre points de la politique interne ;
- votre classement distingue ce qui expose des données **déjà stockées** de ce qui
  protège des données **à venir**.

#### ✅ Auto-évaluation

Avant de passer au corrigé, vérifiez que :

- [ ] J'ai relié chaque fonction à ce qu'elle protège réellement.
- [ ] Mon tableau compte au moins cinq problèmes, avec un risque concret pour chacun.
- [ ] Mes deux corrections utilisent la bibliothèque standard et respectent la politique interne.
- [ ] J'ai justifié mon premier choix de correction par un risque, pas par une préférence.
- [ ] Je peux expliquer pourquoi « ça marche » ne veut pas dire « c'est sûr ».

#### Pour aller plus loin (exercice)

*Facultatif.* Ajoutez au module une fonction `verifier_signature` qui compare un code
d'authentification reçu à celui calculé, **sans fuite de chronométrage**, puis mesurez
la différence de comportement entre `==` et `hmac.compare_digest` sur une comparaison
qui échoue au premier caractère.

<details>
<summary>Voir la correction</summary>

#### Partie 1 — Recenser les usages

**Éléments attendus :**

| Fonction | Ce qu'elle protège | Ce qu'elle garantit |
| --- | --- | --- |
| `empreinte_mot_de_passe` | Le mot de passe d'un compte client | Rien de fiable : l'empreinte est calculée sans sel, avec une fonction cassée |
| `chiffrer_export` | Les données envoyées au partenaire | Une confidentialité partielle : les motifs du fichier restent visibles |
| `generer_jeton` | La session d'un client connecté | Rien : le jeton est devinable |
| `verifier_jeton` | L'accès à une session | L'exactitude du jeton, mais en fuyant par chronométrage |

**Ce qu'il faut retenir :** l'audit commence par nommer ce que chaque fonction est
censée protéger. Sans cette liste, on corrige des détails et on rate la fonction qui
expose tout.

#### Partie 2 — Auditer

**Éléments attendus :**

| Fonction | Problème | Risque concret | Correction proposée |
| --- | --- | --- | --- |
| Constantes du module | Clé en dur dans le code source | Quiconque lit le dépôt déchiffre tous les exports | Clé tirée au hasard, stockée hors du code, injectée à l'exécution |
| `empreinte_mot_de_passe` | MD5, sans sel | Collisions connues, et deux mots de passe identiques donnent la même empreinte | Fonction de hachage lente, avec sel unique par utilisateur |
| `chiffrer_export` | Mode ECB (Electronic Codebook) | Deux blocs identiques donnent le même chiffré : les motifs du fichier réapparaissent | AES (Advanced Encryption Standard) en mode GCM (Galois/Counter Mode) |
| `chiffrer_export` | Aucune valeur de départ unique | Deux exports comparables, et pas de détection d'altération | Nonce aléatoire de 12 octets, jamais réutilisé avec la même clé |
| `generer_jeton` | `random`, générateur prévisible | Le jeton se reconstitue à partir de la graine : session détournée | `secrets.token_bytes(32)` |
| `generer_jeton` | Jeton de 12 chiffres | Espace de recherche réduit, énumérable | Jeton de 32 octets, encodé pour le transport |
| `verifier_jeton` | Comparaison avec `==` | Fuite par chronométrage : le jeton se reconstruit caractère par caractère | `hmac.compare_digest` |

**Erreurs fréquentes :**

- Ne relever que les algorithmes (MD5, ECB) et oublier ce qui les entoure : la clé en
  dur et la comparaison non constante sont au moins aussi graves.
- Proposer « AES partout » sans préciser le mode : en ECB, AES ne corrige rien du tout.
- Confondre gravité et exotisme : une clé dans le dépôt est banale et catastrophique ;
  une attaque par canal auxiliaire est spectaculaire et rare.

#### Partie 3 — Corriger

**Éléments attendus :**

```python
import hashlib
import hmac
import secrets

def empreinte_mot_de_passe(mot_de_passe, sel):
    """Empreinte salée, calculée par une fonction lente (100 000 itérations)."""
    return hashlib.pbkdf2_hmac(
        "sha256", mot_de_passe.encode(), sel, 100000
    ).hex()

def generer_jeton():
    """Jeton de session de 32 octets, tiré par un générateur sûr."""
    return secrets.token_bytes(32).hex()
```

Le sel se tire lui aussi au hasard, une fois par utilisateur, et se stocke à côté de
l'empreinte — il n'a pas besoin d'être secret :

```python
sel = secrets.token_bytes(16)
empreinte = empreinte_mot_de_passe("mot-de-passe-saisi", sel)
```

**Ce qu'il faut retenir :** la correction de `empreinte_mot_de_passe` change deux
choses à la fois — la fonction (lente, récente) et l'ajout d'un sel. L'une sans l'autre
reste insuffisante : une fonction de hachage rapide, même la plus répandue, reste trop rapide pour un mot de passe, et un sel
ajouté à MD5 ne répare pas les collisions.

**Erreurs fréquentes :**

- Passer à une fonction de hachage classique « parce que MD5 est cassé » : pour un mot de passe, il faut une
  fonction **lente**, conçue pour cela.
- Oublier de stocker le sel : sans lui, impossible de vérifier l'empreinte à la
  connexion.
- Générer le sel avec `random` : on retombe sur un générateur prévisible.

#### Partie 4 — Hiérarchiser

**Éléments attendus :**

Ordre raisonnable, du plus urgent au moins urgent :

1. **La clé en dur** — elle expose les données **déjà envoyées** au partenaire, et
   quiconque a accès au code les déchiffre rétroactivement. C'est le seul constat dont
   l'impact porte sur le passé.
2. **Les mots de passe en MD5 sans sel** — la base est déjà constituée ainsi : une fuite
   expose des comptes actuels, et la migration prend du temps.
3. **Le jeton devinable** — l'attaque est facile mais elle ne porte que sur les sessions
   **à venir**, et une rotation de clé de session en limite la portée.
4. **Le mode ECB et l'absence de nonce** — les exports sont faits pour être régénérés :
   corriger avant le prochain envoi suffit.
5. **La comparaison non constante** — réelle, mais elle exige une position d'attaque et
   un volume de mesures.

**Sur la formulation :** hiérarchiser, c'est arbitrer entre **ce qui est déjà exposé** et
ce qui le sera. Un défaut qui compromet des données stockées passe avant un défaut qui
compromet des données futures, même si ce dernier est plus élégant à exploiter.

#### Points à retenir

- Un audit crypto commence par **nommer ce que chaque fonction protège** — sinon on
  corrige des détails.
- Les erreurs graves sont rarement spectaculaires : **clé en dur**, **sel absent**,
  **générateur prévisible**, **comparaison non constante**.
- Choisir un bon algorithme ne suffit pas : le **mode** (ECB contre GCM), le **nonce**
  et la **gestion de la clé** décident de la sécurité réelle.
- Les mots de passe se stockent en **empreinte salée**, avec une fonction lente — jamais
  chiffrés, jamais avec une fonction de hachage rapide.
- On corrige d'abord ce qui expose des données **déjà en circulation**.

</details>

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

## 🧠 Vérifiez vos acquis

*Formatif, jamais noté. Répondez sans relire le chapitre, puis dépliez « Vérifier ses réponses ». Ce qui compte est de repérer où vous vacillez, pas de faire 5 sur 5.*

**1.** Un développeur a écrit sa propre fonction de mélange pour rendre un champ « illisible », et l'illibilité semble fonctionner. Quelle est la bonne attitude ?

- A. Accepter : si le résultat est illisible pour un lecteur humain, le chiffrement tient, c'est le critère
- B. Écarter : on utilise une bibliothèque éprouvée, et l'effort se porte sur la gestion des clés
- C. Accepter en attendant, puis faire relire l'algorithme par un collègue plus expérimenté en crypto
- D. Améliorer l'algorithme maison en ajoutant un XOR supplémentaire et une seconde passe de mélange

**2.** Un fichier image chiffré laisse encore reconnaître le contour du sujet, en pixels chiffrés. Quelle est la cause ?

- A. Le mode ECB : chaque bloc est chiffré indépendamment avec la même clé
- B. Le padding : le dernier bloc est complété, et la fin du fichier trahit la structure de l'image
- C. La clé trop courte : l'énumération des clés devient possible, et l'image finit par revenir en clair
- D. L'absence de sel : sans valeur aléatoire, deux images comparables produisent le même chiffré

**3.** Comment stocker les mots de passe des clients de DIGICHEESE dans la base ?

- A. Chiffrés avec la clé de l'application, pour pouvoir les restituer
- B. En empreinte salée, calculée par une fonction lente dédiée
- C. En clair, avec un accès restreint au compte de service
- D. En MD5, puisque l'empreinte est à sens unique

**4.** Le service qui déchiffre les bordereaux renvoie « Clé incorrecte » ou « Format du chiffré invalide » selon le cas. Quel défaut cela constitue-t-il ?

- A. Aucun : une erreur précise aide le support
- B. Un message trop bavard : la différence d'information renseigne l'attaquant
- C. Une fuite par chronométrage : la comparaison s'arrête trop tôt
- D. Un algorithme obsolète : il faut passer à AES

**5.** *Question ouverte (2-3 lignes).* La revue de code crypto du chapitre tient en six questions. Citez-en au moins trois, et dites pour chacune ce qu'elle attrape comme erreur réelle.

---

<details>
<summary>Vérifier ses réponses</summary>

### Réponses

**1. B.** « Ne roule jamais ta propre crypto » est l'une des rares règles absolues du domaine.
**Pourquoi :** on ne voit pas ses propres faiblesses, les motifs du clair survivent (longueur, répétitions, fréquences), et l'implémentation trahit l'algorithme. La gestion des clés, elle, ne se délègue pas — c'est là que porte l'effort.

**2. A.** C'est l'image du pingouin : ECB fuit la structure complète des données répétées, et la révélation est immédiate sur un CSV, un JSON ou une image.
**Pourquoi :** deux blocs identiques donnent le même chiffré. Les remplacements sont CBC ou GCM.

**3. B.** Chiffrer les mots de passe, c'est garder une clé qui rend tout lisible en cas de fuite de la base.
**Pourquoi :** on ne « retrouve » jamais le mot de passe, on le vérifie — d'où une fonction lente dédiée et un sel propre à chaque utilisateur, sans quoi deux mêmes mots de passe donnent la même empreinte et s'attaquent en une passe.

**4. B.** Un service qui déchiffre doit répondre de façon uniforme, sans distinguer ce qui a échoué.
**Pourquoi :** chaque différence de message — comme chaque différence de temps de réponse — est une information gratuite pour l'attaquant.

**5.** Attendu (trois parmi les six) : **bibliothèque standard ou pas ?** → attrape le chiffrement maison ; **la clé, d'où vient-elle ?** → clé en dur dans le dépôt ; **nonce unique ?** → IV ou nonce réutilisé, qui ruine GCM ; **comparaison constante ?** → `==` à la place de `hmac.compare_digest` ; **sel présent ?** → empreintes identiques pour mots de passe identiques ; **réponse uniforme ?** → messages d'erreur trop précis.
**Pourquoi :** le plus difficile à repérer n'est pas l'algorithme mais ce qui l'entoure — un chiffrement impeccable se casse par le comportement du programme qui l'utilise.

</details>

---

[← Chapitre précédent](04-hachage-et-signature.md) · [Sommaire](Readme.md) · [Chapitre suivant →](06-alea-et-cles.md)
