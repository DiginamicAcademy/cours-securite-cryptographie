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

### Exercice 1 — Chiffrer un fichier avec AES-GCM (45 min)

#### Contexte

**VéloCité**, un opérateur de vélos en libre-service, confie à un laboratoire
universitaire un extrait de ses données de location pour une étude de mobilité. Le
fichier contient des données personnelles (dates, durées, habitudes de déplacement) :
il doit être chiffré **avant** de quitter l'infrastructure de l'opérateur, et seul le
laboratoire doit pouvoir le lire.

L'équipe data a retenu le mode GCM (Galois/Counter Mode) du standard AES (Advanced
Encryption Standard) : il chiffre et authentifie en une seule passe. La bibliothèque
retenue est `cryptography`, la référence en Python — elle s'installe avec
`pip install cryptography`.

Voici le fichier à protéger, `export-locations.csv` :

```csv
date,station,duree_min,abonne
2026-10-05,Gare Centrale,12,1
2026-10-05,Place du Marche,7,0
2026-10-06,Universite,23,1
```

Les repères de l'équipe, à respecter à la lettre :

- la clé de chiffrement fait **32 octets** (soit **256 bits**), tirés au hasard ;
- le **nonce** (valeur de départ unique) fait **12 octets** (soit **96 bits**), tirés
  au hasard, et il est **préfixé** au fichier chiffré pour que le déchiffrement
  retrouve tout seul ;
- le nonce ne doit **jamais** être réutilisé avec la même clé.

#### Consignes

##### Partie 1 — Générer la clé et le nonce

1. Écrivez un script `chiffrer.py` qui génère la clé et le nonce avec le module
   `secrets` (et non `random`, qui est prévisible).
2. Affichez leur longueur en octets pour vérifier qu'elle correspond aux repères
   ci-dessus.

##### Partie 2 — Chiffrer le fichier

1. Lisez `export-locations.csv` **en binaire** (`read_bytes`).
2. Chiffrez-le avec la classe AESGCM (AES en mode GCM), sans donnée additionnelle.
3. Écrivez le résultat dans `export-locations.csv.chiffre`, sous la forme
   `nonce + chiffré` (le nonce d'abord, puis le corps chiffré).
4. Constatez que le fichier produit est illisible : ouvrez-le, il ne doit ressembler à
   rien de lisible.

##### Partie 3 — Déchiffrer et vérifier

1. Écrivez `dechiffrer.py` : il lit le fichier chiffré, sépare le nonce du corps,
   déchiffre, et réécrit `export-locations-relu.csv`.
2. Vérifiez que `export-locations-relu.csv` est **identique octet à octet** au fichier
   d'origine.

##### Partie 4 — Détecter une altération

1. Modifiez **un seul octet** du corps chiffré dans `export-locations.csv.chiffre`, par
   exemple en lui ajoutant 1.
2. Relancez `dechiffrer.py` : que se passe-t-il ?
3. Notez le nom de l'exception levée, et expliquez en deux phrases pourquoi ce
   comportement est un avantage et non un défaut.

##### Partie 5 — Poser la question de la clé

Le laboratoire est distant : comment lui transmettre la clé de chiffrement sans la
livrer à quiconque écouterait le canal ? Répondez en trois phrases maximum, en
utilisant ce que le chapitre sur le chiffrement asymétrique décrit.

#### Attendu

- `chiffrer.py` et `dechiffrer.py`, exécutables de bout en bout.
- Le fichier chiffré produit, et le fichier relu identique à l'original.
- Le compte rendu de la partie 4 (exception observée + explication).
- La réponse courte de la partie 5.

#### Vérification

Vous avez réussi quand :

- `export-locations.csv.chiffre` commence bien par le nonce, et son contenu est
  illisible ;
- `export-locations-relu.csv` est identique au fichier d'origine ;
- une altération d'un seul octet fait échouer le déchiffrement au lieu de produire un
  fichier corrompu ;
- votre réponse de la partie 5 ne propose **pas** d'envoyer la clé en clair.

#### ✅ Auto-évaluation

Avant de passer au corrigé, vérifiez que :

- [ ] Ma clé et mon nonce sont tirés avec `secrets`, et leur longueur est vérifiée.
- [ ] Mon fichier chiffré contient le nonce en tête, puis le corps chiffré.
- [ ] Le déchiffrement restitue un fichier identique octet à octet.
- [ ] J'ai observé le refus de déchiffrer un contenu altéré et je sais l'expliquer.
- [ ] J'ai décrit une transmission de clé qui ne passe pas par un canal en clair.

#### Prolongement de l'exercice

*Facultatif.* Plutôt que de tirer la clé au hasard, dérivez-la d'une phrase secrète
avec PBKDF2, puis vérifiez que deux phrases secrètes différentes produisent deux clés
différentes. Notez ce que cela change si la phrase secrète est faible.

<details>
<summary>Voir la correction</summary>

#### Partie 1 — Générer la clé et le nonce

**Éléments attendus :**

```python
import secrets

cle = secrets.token_bytes(32)    # 256 bits, tirés par un CSPRNG
nonce = secrets.token_bytes(12)  # 96 bits, unique pour cette clé

print(len(cle), len(nonce))      # 32 12
```

**Ce qu'il faut retenir :** `secrets` tire des octets imprévisibles ; `random` produirait
une suite reproductible à partir de sa graine, et donc une clé devinable. La longueur
se vérifie à l'exécution : une clé trop courte ne se rattrape pas après coup.

**Erreurs fréquentes :**

- Générer la clé avec `random.randbytes` ou `os.urandom` mal employé : seul `secrets`
  (ou `os.urandom`) est adapté à la cryptographie.
- Oublier de tirer le nonce au hasard, ou le fixer « pour simplifier » : deux messages
  chiffrés avec le même nonce et la même clé se comparent.

#### Partie 2 — Chiffrer le fichier

**Éléments attendus :**

```python
from pathlib import Path

from cryptography.hazmat.primitives.ciphers.aead import AESGCM

donnees = Path("export-locations.csv").read_bytes()
corps = AESGCM(cle).encrypt(nonce, donnees, None)  # None : pas de donnée additionnelle

# le nonce est préfixé : il voyage avec le message, en clair
Path("export-locations.csv.chiffre").write_bytes(nonce + corps)
```

**Ce qu'il faut retenir :** `encrypt` renvoie le chiffré **accompagné de son tag
d'authentification** — c'est ce qui rend le mode GCM « authentifiant ». Le nonce n'est
pas secret : il est préfixé au fichier pour que le déchiffrement sache par où commencer.

**Erreurs fréquentes :**

- Lire le fichier en texte (`read_text`) : les accents et les fins de ligne se
  transforment, et le déchiffrement ne restituera pas l'original. On travaille en
  **octets**.
- Écrire le nonce dans un fichier séparé, qu'on finit par perdre : sans nonce, le
  message est indéchiffrable.
- Réutiliser le nonce pour chiffrer un second fichier avec la même clé.

#### Partie 3 — Déchiffrer et vérifier

**Éléments attendus :**

```python
from pathlib import Path

from cryptography.hazmat.primitives.ciphers.aead import AESGCM

brut = Path("export-locations.csv.chiffre").read_bytes()
nonce_lu, corps = brut[:12], brut[12:]

clair = AESGCM(cle).decrypt(nonce_lu, corps, None)
Path("export-locations-relu.csv").write_bytes(clair)

assert clair == Path("export-locations.csv").read_bytes()
```

**Ce qu'il faut retenir :** la découpe `[:12]` / `[12:]` est la contrepartie exacte du
préfixage de la partie 2. La comparaison octet à octet est la seule vérification qui
vaille : « le fichier s'ouvre » ne prouve pas qu'il est intact.

**Erreurs fréquentes :**

- Découper à la mauvaise longueur : le nonce lu ne correspond pas à celui utilisé, et
  le déchiffrement échoue alors que le fichier est sain.
- Comparer les fichiers « à l'œil » plutôt que par leur contenu binaire.

#### Partie 4 — Détecter une altération

**Éléments attendus :**

```python
from cryptography.exceptions import InvalidTag

brut = bytearray(Path("export-locations.csv.chiffre").read_bytes())
brut[12] = brut[12] + 1   # on altère un seul octet du corps chiffré

try:
    AESGCM(cle).decrypt(bytes(brut[:12]), bytes(brut[12:]), None)
    print("accepté : c'est un problème")
except InvalidTag:
    print("altération détectée")
```

Le déchiffrement **refuse** de rendre un contenu : l'exception `InvalidTag` signale que
le tag d'authentification ne correspond plus au message.

**Ce qu'il faut retenir :** c'est précisément l'avantage recherché. Un chiffrement qui
restituerait un fichier corrompu laisserait croire à une donnée valide ; GCM préfère
échouer bruyamment. L'intégrité et la confidentialité sont vérifiées en une passe.

**Erreurs fréquentes :**

- Altérer le nonce plutôt que le corps : le résultat est le même (refus), mais on teste
  alors un autre scénario — les deux sont utiles, à mener séparément.
- Interpréter l'exception comme un bug de la bibliothèque et « corriger » en
  ignorant l'erreur : c'est exactement le réflexe qui transforme une protection en
  fausse sécurité.

#### Partie 5 — Poser la question de la clé

**Éléments attendus :**

Envoyer la clé au laboratoire par le même canal que le fichier reviendrait à laisser la
clé voyager en clair : quiconque intercepte les deux obtient tout. La solution est de
chiffrer **la clé** avec la clé publique du laboratoire, et d'envoyer cette clé de
session chiffrée à côté du fichier. Seul le laboratoire, avec sa clé privée, retrouve la
clé de session — c'est le principe hybride décrit dans le chapitre sur le chiffrement
asymétrique.

**Sur la formulation :** la question n'est jamais « comment cacher la clé dans le
fichier ? » mais « qui détient la clé privée qui la déverrouille ? ». Tant que cette
question n'a pas de réponse, le chiffrement des données ne protège rien.

#### Points à retenir

- Une bibliothèque standard (`cryptography`) fait le travail difficile : on gère la
  **clé**, le **nonce** et les **fichiers**, pas les mathématiques.
- La clé se tire avec `secrets` ; le nonce est aléatoire, unique, et **préfixé** au
  message chiffré.
- GCM chiffre **et** authentifie : toute altération fait échouer le déchiffrement au
  lieu de produire un fichier corrompu.
- On travaille en **octets** de bout en bout, et on vérifie par comparaison binaire.
- La clé de session se transmet elle-même **chiffrée** avec la clé publique du
  destinataire : c'est le principe hybride.

</details>

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

*Formatif, jamais noté. Répondez sans relire le chapitre, puis dépliez « Vérifier ses réponses ». Ce qui compte est de repérer où vous vacillez, pas de faire 5 sur 5.*

**1.** Cent personnes doivent communiquer deux à deux en chiffrement symétrique. Combien de clés distinctes faut-il créer, transmettre, stocker et révoquer ?

- A. 100
- B. 200
- C. 4 950
- D. Autant que nécessaire, mais une seule suffit si elles sont longues

**2.** Alice veut envoyer un document que Bob est le seul à pouvoir lire, en asymétrique. Quelle clé utilise-t-elle pour chiffrer ?

- A. Sa propre clé publique
- B. La clé publique de Bob
- C. La clé privée de Bob
- D. La clé privée d'Alice

**3.** Pourquoi ne chiffre-t-on jamais les données directement avec RSA ?

- A. Parce que RSA est lent et que la taille du message est bornée par celle de la clé
- B. Parce que RSA n'est pas un algorithme public : on ne peut pas l'auditer, donc on ne s'y fie pas
- C. Parce que RSA ne chiffre que des nombres premiers : il faudrait convertir les données au préalable
- D. Parce que RSA ne chiffre que des clés de 128 bits, ce qui est aujourd'hui insuffisant

**4.** Que faut-il retenir des courbes elliptiques (ECC) par rapport à RSA ?

- A. Elles sont plus sûres à taille de clé égale, mais incompatibles avec TLS
- B. Elles offrent la même sécurité avec des clés bien plus courtes
- C. Elles suppriment le besoin de clé privée
- D. Elles remplacent l'échange de clés par la signature

**5.** *Question ouverte (2-3 lignes).* Deux machines qui ne se sont jamais parlé conviennent d'une clé de session sur un canal public, que l'observateur ne peut pas reconstituer. Nommez ce mécanisme, puis replacez-le dans les trois temps du cours : que fait chacun des trois ?

---

<details>
<summary>Vérifier ses réponses</summary>

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

</details>

---

[← Chapitre précédent](02-chiffrement-symetrique.md) · [Sommaire](Readme.md) · [Chapitre suivant →](04-hachage-et-signature.md)
