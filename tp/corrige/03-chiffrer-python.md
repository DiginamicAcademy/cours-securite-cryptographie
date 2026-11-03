# Corrigé — Chiffrer un fichier avec AES-GCM

> Corrigé de [l'exercice « Chiffrer un fichier avec AES-GCM »](../03-chiffrer-python.md).
> Comparez d'abord votre travail aux **étapes** ci-dessous : la méthode compte autant
> que le résultat.
>
> Le chiffrement employé est AES (Advanced Encryption Standard) en mode
> GCM (Galois/Counter Mode).

---

## Partie 1 — Générer la clé et le nonce

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

---

## Partie 2 — Chiffrer le fichier

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

---

## Partie 3 — Déchiffrer et vérifier

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

---

## Partie 4 — Détecter une altération

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

---

## Partie 5 — Poser la question de la clé

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

---

## Points à retenir

- Une bibliothèque standard (`cryptography`) fait le travail difficile : on gère la
  **clé**, le **nonce** et les **fichiers**, pas les mathématiques.
- La clé se tire avec `secrets` ; le nonce est aléatoire, unique, et **préfixé** au
  message chiffré.
- GCM chiffre **et** authentifie : toute altération fait échouer le déchiffrement au
  lieu de produire un fichier corrompu.
- On travaille en **octets** de bout en bout, et on vérifie par comparaison binaire.
- La clé de session se transmet elle-même **chiffrée** avec la clé publique du
  destinataire : c'est le principe hybride.
