[← Chapitre précédent](01-histoire-et-services.md) · [Sommaire](Readme.md) · [Chapitre suivant →](03-chiffrement-asymetrique.md)

# Du chiffre de César à AES

> En 58 av. J.-C., César envoie ses ordres à ses légions par des messagers qu'il ne
> choisit pas toujours. Pour qu'un message intercepté ne livre rien, il décale chaque
> lettre de son texte d'un nombre fixe de rangs. Le procédé tient tant que l'ennemi
> ignore le décalage — et il ne tient pas longtemps. De cette idée simple est né tout
> le chiffrement symétrique, jusqu'à AES.

**Durée de lecture :** 75 min · **Prérequis :** [Histoire et services de la cryptographie](01-histoire-et-services.md)

---

## Objectifs

À l'issue de ce chapitre, vous serez capable de :

- **Chiffrer** un message court avec le chiffre de César, puis **le casser** par force brute.
- **Expliquer** pourquoi la fréquence des lettres ruine les chiffrements par substitution.
- **Distinguer** un chiffrement par flot d'un chiffrement par blocs, et **nommer** les modes ECB, CBC et GCM.
- **Justifier** le rôle de la clé, du vecteur d'initialisation et du padding dans un chiffrement moderne.
- **Écarter** un générateur pseudo-aléatoire non adapté à la production de clés.

---

## I. Le chiffre de César

César (100-44 av. J.-C.) protège ses dépêches militaires par un procédé d'une
simplicité désarmante : **décaler chaque lettre** d'un nombre fixe de rangs dans
l'alphabet. Avec un décalage de 3, A devient D, B devient E, et ainsi de suite ; en
fin d'alphabet, on boucle — X devient A.

L'usage est militaire : si un messager est capturé, l'ennemi ne lit rien. Le contexte
explique aussi la limite du procédé : la clé est un simple nombre, et il n'y en a pas
beaucoup.

Le vocabulaire de cette époque est déjà celui d'aujourd'hui :

| Terme | Signification | Dans l'exemple de César |
| --- | --- | --- |
| Texte clair | Le message lisible | L'ordre transmis à la légion |
| Texte chiffré | Le message transformé | Le message décalé |
| Clé | Le secret partagé | Le nombre de rangs de décalage |
| Algorithme | La règle de transformation | Décaler chaque lettre de N rangs |

En Python, le décalage tient en quelques lignes :

```python
def cesar(texte, decalage):
    """Chiffre un message par décalage (lettres minuscules, reste inchangé)."""
    resultat = ""
    for lettre in texte:
        if "a" <= lettre <= "z":
            base = ord("a")
           # % 26 : on boucle en fin d'alphabet (x -> a, y -> b, z -> c)
            resultat += chr(base + (ord(lettre) - base + decalage) % 26)
        else:
            resultat += lettre # espaces et ponctuation sont conservés
    return resultat

print(cesar("veni vidi vici", 3))
```

L'exemple historique se lit ainsi :

```text
Texte clair   : VENI VIDI VICI
Décalage (clé): 3
Texte chiffré : YHQL YLGL YLFL
```

Le texte chiffré est parfaitement inintelligible — pour peu que le destinataire
connaisse la clé, ici le nombre 3.

> [!TIP]
> Un chiffrement ne cache rien tant que la clé est connue de l'adversaire. La
> question n'est jamais « mon texte est-il transformé ? » mais « ma clé est-elle
> restée secrète ? ».

### Casser César

Une clé qui ne peut prendre que 25 valeurs ne résiste pas à l'énumération : on essaie
les 25 décalages, on lit les 25 résultats, on garde celui qui est compréhensible. C'est
une attaque par **force brute** — tester toutes les clés possibles — et elle prend une
fraction de seconde sur n'importe quel ordinateur.

```python
def casser_cesar(texte_chiffre):
    """Affiche les 25 décalages possibles : la bonne clé se repère à l'œil."""
    for cle in range(1, 26):
        print(cle, cesar(texte_chiffre, -cle))
```

> 🧪 **Atelier rapide (10 min)**
>
> 1. Reprenez la fonction `cesar` et chiffrez une phrase de votre choix avec le
> décalage 3.
> 2. Lancez `casser_cesar` sur votre message chiffré.
> 3. Repérez la bonne clé à l'œil, puis notez le nombre d'essais nécessaires.

La leçon est brutale : **la robustesse se mesure au nombre de clés possibles**. Avec
25 clés, l'attaquant les essaie toutes. Il faudra donc des algorithmes dont l'espace
des clés est tellement grand que l'énumération devient impossible.

---

## II. La substitution mono-alphabétique

L'étape suivante, dès l'Antiquité, consiste à donner **à chaque lettre sa propre
image** : A devient M, B devient T, C devient R… C'est une substitution
mono-alphabétique — une seule table de correspondance pour tout le message. L'espace
des clés explose : il y a environ 4 × 10^26 façons d'associer 26 lettres entre elles,
bien au-delà de ce qu'on peut énumérer.

Et pourtant, ce chiffrement tombe aussi. L'attaque ne s'intéresse plus aux clés mais
au **langage** : dans un texte français, les lettres n'apparaissent pas au hasard. Le
« E » est de loin la plus fréquente, suivi du « A », du « S », du « I »… En comptant
les lettres du texte chiffré et en comparant la distribution à celle du français, on
retrouve la table de correspondance sans jamais l'avoir connue. C'est l'**analyse de
fréquence**, décrite dès le IXe siècle par le mathématicien Al-Kindi.

> 🧪 **Atelier rapide (8 min)**
>
> 1. Prenez un paragraphe d'une trentaine de lignes (un article, un courriel, un
> chapitre de ce cours).
> 2. Comptez l'apparition de chaque lettre, par exemple avec un dictionnaire Python :
> `compte[lettre] = compte.get(lettre, 0) + 1`.
> 3. Triez les lettres par fréquence décroissante. Le « E » arrive-t-il en tête ?
> 4. Refaites le comptage sur un texte chiffré par substitution : la forme de la
> distribution est la même, seuls les noms des lettres ont changé.

Ce qui a tué la substitution n'est pas un calcul plus rapide, c'est une **propriété du
texte** que le chiffrement laisse passer. Toute la cryptographie moderne est une
réponse à cette leçon : un bon chiffrement ne doit laisser filtrer **aucune** structure
du texte clair — ni les fréquences, ni la longueur, ni les répétitions.

> [!WARNING]
> Chiffrer ne rend pas un message indétectable : sa longueur, son destinataire et son
> horodatage restent visibles. La cryptographie protège le contenu, pas la
> conversation.

---

## III. La cryptographie moderne : AES

Les méthodes précédentes sont des jeux d'enfant comparées aux besoins d'aujourd'hui :
protéger des millions de transactions, des sauvegardes, des flux entre serveurs. Deux
idées ont fait basculer la cryptographie dans l'ère moderne.

**Première idée : des clés qu'on ne peut pas énumérer.** AES (Advanced Encryption
Standard), adopté en 2001, travaille sur des blocs de 128 bits avec des clés de 128,
192 ou 256 bits. Avec une clé de 128 bits, il existe 2^128 clés possibles — un nombre
suivi de 38 zéro. Même en essayant un milliard de clés par seconde, l'énumération
prendrait plus de temps que l'âge de l'univers. La force brute n'est plus une option :
la sécurité repose sur le secret de la clé, pas sur l'ignorance de l'algorithme.

**Deuxième idée : chiffrer par blocs, et donc choisir un mode.** AES transforme des
blocs de 16 octets. Un fichier plus long doit être découpé, et la manière d'assembler
les blocs s'appelle le **mode de chiffrement** :

| Mode | Principe | À retenir |
| --- | --- | --- |
| ECB (Electronic Codebook) | Chaque bloc est chiffré indépendamment, avec la même clé | Deux blocs identiques donnent le même chiffré : les motifs réapparaissent. **À proscrire.** |
| CBC (Cipher Block Chaining) | Chaque bloc est mélangé avec le chiffré du bloc précédent | Les motifs disparaissent ; exige un vecteur d'initialisation et un padding. |
| GCM (Galois/Counter Mode) | Chiffrement par compteur, plus une vérification d'intégrité | Chiffre **et** authentifie en une passe. Le mode à préférer aujourd'hui. |

Le mode CBC a besoin que tous les blocs soient de même taille : on complète le dernier
bloc avec du **padding** (des octets de remplissage dont la valeur encode la longueur
ajoutée). GCM, lui, travaille en flot : il produit une suite de bits qu'on combine avec
le message, bloc par bloc, sans complétion.

### La clé, le vecteur d'initialisation et le hasard

Deux valeurs entrent en jeu, et on les confond souvent :

- **La clé** est le secret. Elle doit être tirée au hasard, gardée secrète, et changée
  quand elle ne l'est plus. Une clé devinable ou réutilisée partout annule tout le
  reste.
- **Le vecteur d'initialisation** (IV, *initialization vector*) est une valeur de
  départ qui fait diverger deux chiffrements faits avec la même clé. Il doit être
  **aléatoire et jamais réutilisé** — mais il n'a pas besoin d'être secret : il voyage
  avec le message, en clair.

> [!DANGER]
> Réutiliser le même IV avec la même clé est une erreur classique et grave : en GCM,
> elle ruine à la fois la confidentialité et l'authentification. En CBC, elle laisse
> apparaître des correspondances entre messages. Quand un IV est tiré au hasard, on le
> stocke à côté du chiffré ; quand il est séquentiel, on garantit qu'il ne se répète
> jamais.

### D'où vient le hasard ?

Un ordinateur est une machine **déterministe** : un programme, avec les mêmes entrées,
produit toujours les mêmes sorties. Alors comment obtenir du hasard ? En simulant le
hasard avec une **graine** (*seed*) de départ, qui détermine toute la suite. Même
graine, même suite — c'est pratique pour les tests, c'est fatal pour la cryptographie.

> 🧪 **Atelier aléatoire et graine (10 min)**
>
> 1. Exécutez ce fragment, puis relancez-le à l'identique :
>
> ```python
> import random
>
> random.seed(42) # graine fixée : la suite est reproductible
> print([random.randint(0, 99) for _ in range(5)])
> ```
>
> 2. Constatez que la suite est **toujours la même**.
> 3. Remplacez par le module `secrets`, conçu pour la cryptographie :
>
> ```python
> import secrets
>
> print(secrets.token_bytes(16)) # 16 octets imprévisibles
> ```
>
> 4. Relancez : cette fois, chaque exécution produit une valeur différente.

Le module `random` de Python est un générateur pseudo-aléatoire, ou PRNG (
*Pseudo-Random Number Generator*) : sa sortie est reproductible à partir de la graine,
et quelqu'un qui devine la graine reconstitue toute la suite. Pour la cryptographie, on
utilise un générateur pseudo-aléatoire cryptographiquement sûr, ou CSPRNG (
*Cryptographically Secure Pseudo-Random Number Generator*), comme `secrets` en Python ou
`os.urandom`. Une graine faible ou prévisible, c'est une clé devinable — et un
chiffrement qui ne protège rien.

> [!TIP]
> En révision de code, le réflexe est simple : toute apparition de `random` dans un
> module qui touche aux clés, aux jetons ou aux mots de passe est un défaut. La bonne
> bibliothèque est `secrets` (ou `os.urandom`), sans exception.

---

## IV. Aller plus loin

*Ces approfondissements sont facultatifs : ils ne sont jamais exigés pour terminer la
journée.*

- **Enigma et la machine de Turing** : pendant la Seconde Guerre mondiale, la machine
  Enigma chiffrait les communications allemandes avec un alphabet de substitution qui
  changeait à chaque frappe. L'équipe d'Alan Turing à Bletchley Park l'a cassée en
  exploitant à la fois des faiblesses de conception et des habitudes des opérateurs. La
  leçon vaut encore : la meilleure algorithmique ne résiste pas à une mauvaise
  implémentation.
- **Les autres modes de bloc** : CTR (Counter) transforme un chiffrement par blocs en
  chiffrement par flot et se parallélise ; CFB (Cipher Feedback) et OFB (Output Feedback) sont des variantes historiques.
  Le choix d'un mode se fait sur les recommandations à jour de l'ANSSI (Agence
  nationale de la sécurité des systèmes d'information), jamais sur une habitude.
- **L'analyse de fréquence, en pratique** : quelques dizaines de lignes de Python
  suffisent pour casser une substitution sur un texte suffisamment long. C'est un
  excellent exercice pour mesurer ce qu'un chiffrement laisse filtrer.
- **Chiffrement par flot contre chiffrement par blocs** : le premier combine le message
  avec une suite de bits générée (comme en GCM), le second transforme des blocs fixes
  (comme en CBC). Les deux coexistent parce qu'ils ne répondent pas aux mêmes contraintes
  de mise en œuvre.

---

## Points clés

- Le chiffre de César illustre le principe de tout chiffrement symétrique : **une
  transformation réversible, gouvernée par une clé partagée**.
- Un chiffrement dont l'espace des clés est petit tombe à la **force brute** ; un
  chiffrement qui laisse filtrer les fréquences tombe à l'**analyse statistique**.
- AES chiffre des blocs de 128 bits avec des clés de 128 bits et plus : l'énumération
  des clés n'est plus une option, la sécurité repose sur le **secret de la clé**.
- Le **mode** (ECB, CBC, GCM) décide comment les blocs sont assemblés ; ECB est à
  proscrire, GCM chiffre et authentifie en une passe.
- La **clé** doit être aléatoire et secrète ; le **vecteur d'initialisation** doit être
  aléatoire et jamais réutilisé, sans être secret.
- Le hasard se génère avec un **CSPRNG** (`secrets`, `os.urandom`) — jamais avec `random`.

---

[← Chapitre précédent](01-histoire-et-services.md) · [Sommaire](Readme.md) · [Chapitre suivant →](03-chiffrement-asymetrique.md)
