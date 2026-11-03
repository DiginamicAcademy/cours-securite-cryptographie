# Corrigé — Auditer un code qui utilise mal la crypto

> Corrigé de [l'exercice « Auditer un code qui utilise mal la crypto »](../05-auditer-crypto.md).
> Comparez d'abord votre travail aux **étapes** ci-dessous : la méthode compte autant
> que le résultat.

---

## Partie 1 — Recenser les usages

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

---

## Partie 2 — Auditer

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

---

## Partie 3 — Corriger

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

---

## Partie 4 — Hiérarchiser

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

---

## Points à retenir

- Un audit crypto commence par **nommer ce que chaque fonction protège** — sinon on
  corrige des détails.
- Les erreurs graves sont rarement spectaculaires : **clé en dur**, **sel absent**,
  **générateur prévisible**, **comparaison non constante**.
- Choisir un bon algorithme ne suffit pas : le **mode** (ECB contre GCM), le **nonce**
  et la **gestion de la clé** décident de la sécurité réelle.
- Les mots de passe se stockent en **empreinte salée**, avec une fonction lente — jamais
  chiffrés, jamais avec une fonction de hachage rapide.
- On corrige d'abord ce qui expose des données **déjà en circulation**.
