# Corrigé — Chiffrer une colonne sensible

> Corrigé de [l'exercice « Chiffrer une colonne sensible »](../09-chiffrer-bdd.md).
> Comparez d'abord votre travail aux **étapes** ci-dessous : la méthode compte
> autant que le résultat.

---

## Partie 1 — Choisir la stratégie

**Éléments attendus :**

**1. Pourquoi le chiffrement de disque ne suffit pas.** Il protège le volume de la
machine, pas les copies qui en sortent : la sauvegarde du soir, l'export de support,
la réplication, la session d'un administrateur qui ouvre un client SQL. Toutes ces
voies livrent la colonne en clair, quel que soit l'état du disque. Le chiffrement
applicatif protège la **valeur** elle-même, où qu'elle aille.

**2. Pourquoi chiffrer plutôt que hacher.** La paie doit **relire** l'IBAN pour
ordonner le virement : le hachage est à sens unique, il rendrait la donnée
inutilisable. Le hachage convient quand on veut seulement **reconnaître** une valeur
(un mot de passe, par exemple) ; ici, le besoin est de la restituer. « Plus sûr » n'a
de sens que rapporté à un usage : un hachage appliqué à une donnée à relire est une
perte de données, pas une sécurité.

**3. Nom et email.** Ce sont des données personnelles, mais leur sensibilité n'est pas
la même que celle d'un IBAN. Position défendable : les garder lisibles pour le
fonctionnement courant, limiter strictement qui les consulte, tracer les accès, et
**minimiser** — ne stocker que ce qui sert réellement. Chiffrer l'email, par exemple,
est possible, mais cela casse les notifications et les recherches : l'arbitrage se
fait au cas par cas, en documentant ce que l'on choisit de ne pas protéger et pour
quelle raison.

---

## Partie 2 — Chiffrer et déchiffrer

**Éléments attendus :** un module qui refuse de démarrer sans clé, chiffre à
l'écriture, déchiffre à la lecture, et migre les lignes existantes.

```python
"""Chiffrement applicatif de la colonne iban — Novéo RH."""
import hashlib
import hmac
import os

from cryptography.fernet import Fernet


def charger_cle() -> Fernet:
    """Lit la clé dans l'environnement ; refuse de démarrer sans elle."""
    cle = os.environ.get("CLE_CHIFFREMENT")
    if not cle:
        raise RuntimeError("Variable CLE_CHIFFREMENT absente : refus de démarrer.")
    return Fernet(cle.encode("utf-8"))


def normaliser(iban: str) -> str:
    """Retire les espaces et passe en majuscules : la forme stockée est unique."""
    return iban.replace(" ", "").upper()


def chiffrer(f: Fernet, valeur: str) -> bytes:
    """Chiffre une valeur sensible avant écriture en base."""
    return f.encrypt(normaliser(valeur).encode("utf-8"))


def dechiffrer(f: Fernet, stocke: bytes) -> str:
    """Déchiffre une valeur lue en base."""
    return f.decrypt(stocke).decode("utf-8")
```

La migration s'écrit en trois temps — ajouter, remplir, puis retirer :

```python
def migrer(conn) -> None:
    f = charger_cle()
    conn.execute("ALTER TABLE employes ADD COLUMN iban_chiffre BLOB")
    conn.execute("ALTER TABLE employes ADD COLUMN iban_index TEXT")

    for id_, iban in conn.execute("SELECT id, iban FROM employes").fetchall():
        conn.execute(
            "UPDATE employes SET iban_chiffre = ?, iban_index = ? WHERE id = ?",
            (chiffrer(f, iban), index_recherche(iban), id_),
        )
    conn.commit()

    # Vérifier la relecture de chaque ligne AVANT de supprimer l'ancienne colonne.
    for id_, iban in conn.execute("SELECT id, iban FROM employes").fetchall():
        lu = conn.execute(
            "SELECT iban_chiffre FROM employes WHERE id = ?", (id_,)
        ).fetchone()[0]
        assert dechiffrer(f, lu) == normaliser(iban)

    conn.execute("ALTER TABLE employes DROP COLUMN iban")
    conn.commit()
```

**Ce qu'il faut retenir :** la normalisation (`normaliser`) n'est pas un détail. Les
IBAN sont saisis avec ou sans espaces ; si l'on chiffre la forme brute, deux écritures
du même compte donnent deux valeurs différentes, et la recherche de la partie 4
échouera sur une ligne sur deux. On fixe **une** forme, avant de chiffrer comme avant
d'indexer.

**Erreurs fréquentes :** supprimer l'ancienne colonne avant d'avoir vérifié la
relecture — en cas d'erreur de format, les données sont perdues avec elle. La bonne
ordre est : ajouter, remplir, **vérifier**, retirer. Et pendant toute la migration, la
sauvegarde de secours reste chiffrée et séparée de la clé.

---

## Partie 3 — Traiter la clé

**Éléments attendus :**

1. **Génération et stockage.** La clé se génère une fois, dans un environnement de
   confiance (`Fernet.generate_key()` ou, mieux, un coffre de secrets). Elle se stocke
   dans un gestionnaire de secrets ou une variable d'environnement injectée au
   démarrage — jamais dans le dépôt, jamais dans la base, jamais dans une image
   Docker. L'accès est limité à l'identité d'exécution de l'application.
2. **Perte de la clé.** Les données chiffrées deviennent illisibles : c'est une perte
   sèche, pas une indisponibilité temporaire. D'où l'obligation d'une **clé de
   secours** rangée ailleurs que la base (coffre, enveloppe scellée, responsable
   désigné) — et d'un exercice de restauration qui prouve qu'elle fonctionne.
3. **Remplacement sans interruption.** On procède par période à deux clés : la
   nouvelle clé chiffre les écritures, l'ancienne reste active pour le déchiffrement ;
   on migre les lignes progressivement, puis on désactive l'ancienne. Pour distinguer
   les deux, on versionne le chiffré — une colonne `cle_id` ou un préfixe dans la
   valeur stockée — afin que la lecture sache quelle clé employer.

---

## Partie 4 — Retrouver une fiche

**Éléments attendus :**

```python
def index_recherche(valeur: str) -> str:
    """Index aveugle : empreinte de recherche, calculée avec une clé distincte."""
    cle_index = os.environ["CLE_INDEX"].encode("utf-8")
    return hmac.new(cle_index, normaliser(valeur).encode("utf-8"), hashlib.sha256).hexdigest()


def trouver_par_iban(conn, iban: str):
    """Retrouve une fiche à partir d'un IBAN, sans jamais lire la colonne chiffrée."""
    f = charger_cle()
    ligne = conn.execute(
        "SELECT id, nom, email, iban_chiffre FROM employes WHERE iban_index = ?",
        (index_recherche(iban),),
    ).fetchone()
    if ligne is None:
        return None
    id_, nom, email, iban_chiffre = ligne
    return {"id": id_, "nom": nom, "email": email, "iban": dechiffrer(f, iban_chiffre)}
```

**Ce que révèle l'index :** deux salariés qui partagent le même IBAN produisent le
même `iban_index` — un lecteur de la base voit donc les **égalités** entre lignes,
même sans la clé. Et si la clé `CLE_INDEX` vient à fuiter, un attaquant peut tester
des IBAN candidats et confirmer lesquels sont présents. C'est un compromis assumé :
l'index rend la recherche possible, au prix d'une fuite limitée sur les
répétitions. La mitigation simple : une clé d'index distincte de la clé de
chiffrement, et le même niveau de protection pour les deux.

**Erreurs fréquentes :** réutiliser la même clé pour chiffrer et pour indexer. Les
deux usages sont différents — l'une protège la confidentialité, l'autre sert de
signature de recherche — et une fuite de l'index ne doit pas livrer le déchiffrement.

---

## Points à retenir

- Le chiffrement applicatif protège la donnée **là où elle va** : sauvegarde, export,
  réplication — pas seulement sur le disque.
- **Chiffrer pour relire, hacher pour reconnaître** : l'usage décide, jamais la
  réputation de la technique.
- La clé vit hors du code et hors de la base ; sa **perte** est aussi grave qu'une
  fuite — d'où la clé de secours et l'exercice de restauration.
- La migration suit un ordre strict : ajouter, remplir, **vérifier**, retirer.
- La recherche sur colonne chiffrée passe par un **index aveugle** qui révèle les
  égalités : compromis documenté, clé distincte, jamais improvisé.
