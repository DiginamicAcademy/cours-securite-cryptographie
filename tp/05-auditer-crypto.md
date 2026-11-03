# Exercice — Auditer un code qui utilise mal la crypto

> 🔵 **Intégration** · **Durée :** 55 min · **Lien :** chapitre [Les erreurs classiques de cryptographie](../05-erreurs-classiques.md)
> **Objectif :** repérer les mauvais usages de la cryptographie dans un extrait de code,
> hiérarchiser les corrections et en proposer deux.

---

## Contexte

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

---

## Consignes

### Partie 1 — Recenser les usages

Pour chacune des quatre fonctions, dites en une phrase **ce qu'elle protège** (un mot
de passe, un fichier, une session) et **ce qu'elle garantit** — ou ne garantit pas.

### Partie 2 — Auditer

Remplissez le tableau ci-dessous. Une ligne par problème trouvé.

| Fonction | Problème | Risque concret | Correction proposée |
| --- | --- | --- | --- |
| | | | |

Visez au moins **cinq** problèmes distincts — il y en a dans chaque fonction, et un
dans les constantes du module.

### Partie 3 — Corriger

Réécrivez deux fonctions au choix, en respectant la politique interne. Vous pouvez
utiliser la bibliothèque standard de Python (`hashlib`, `hmac`, `secrets`) : aucune
installation supplémentaire n'est nécessaire.

### Partie 4 — Hiérarchiser

L'équipe n'a pas le temps de tout corriger avant la mise en production. Classez vos
constats du plus urgent au moins urgent, et justifiez votre premier choix en deux
phrases.

---

## Attendu

- Un tableau d'audit d'au moins cinq lignes, rempli.
- Deux fonctions corrigées, exécutables.
- Un classement justifié des corrections.

---

## Vérification

Vous avez réussi quand :

- chaque fonction du module apparaît au moins une fois dans votre tableau ;
- vos corrections respectent les quatre points de la politique interne ;
- votre classement distingue ce qui expose des données **déjà stockées** de ce qui
  protège des données **à venir**.

---

## ✅ Auto-évaluation

Avant de passer au corrigé, vérifiez que :

- [ ] J'ai relié chaque fonction à ce qu'elle protège réellement.
- [ ] Mon tableau compte au moins cinq problèmes, avec un risque concret pour chacun.
- [ ] Mes deux corrections utilisent la bibliothèque standard et respectent la politique interne.
- [ ] J'ai justifié mon premier choix de correction par un risque, pas par une préférence.
- [ ] Je peux expliquer pourquoi « ça marche » ne veut pas dire « c'est sûr ».

---

## Pour aller plus loin

*Facultatif.* Ajoutez au module une fonction `verifier_signature` qui compare un code
d'authentification reçu à celui calculé, **sans fuite de chronométrage**, puis mesurez
la différence de comportement entre `==` et `hmac.compare_digest` sur une comparaison
qui échoue au premier caractère.
