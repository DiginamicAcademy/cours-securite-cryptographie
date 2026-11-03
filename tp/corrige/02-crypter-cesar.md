# Corrigé — Casser le chiffre de César

> Corrigé de [l'exercice « Casser le chiffre de César »](../02-crypter-cesar.md).
> Comparez d'abord votre travail aux **étapes** ci-dessous : la méthode compte autant
> que le résultat.

---

## Partie 1 — Chiffrer et déchiffrer

**Éléments attendus :**

```python
def chiffrer(texte, cle):
    """Chiffre un message par décalage (lettres minuscules, reste inchangé)."""
    resultat = ""
    for lettre in texte:
        if "a" <= lettre <= "z":
            base = ord("a")
            # % 26 : on boucle en fin d'alphabet (x -> a, y -> b, z -> c)
            resultat += chr(base + (ord(lettre) - base + cle) % 26)
        else:
            resultat += lettre  # espaces et ponctuation sont conservés
    return resultat


def dechiffrer(texte, cle):
    """Retrouve le texte clair : déchiffrer, c'est chiffrer en sens inverse."""
    return chiffrer(texte, -cle)
```

Vérification demandée :

```python
clair = "le camion passe a midi"
chiffre = chiffrer(clair, 7)
print(chiffre)                  # sl jhtpvu whzzl h tpkp
print(dechiffrer(chiffre, 7))   # le camion passe a midi
```

**Ce qu'il faut retenir :** `dechiffrer` ne contient aucune logique propre. Un
décalage de `-cle` dans un alphabet de 26 lettres est exactement l'opération inverse —
c'est la définition même d'un chiffrement réversible. Recopier la boucle une deuxième
fois, c'est doubler le risque d'erreur.

**Erreurs fréquentes :**

- Oublier le modulo `% 26` : les lettres de fin d'alphabet sortent de l'alphabet et
  donnent des caractères bizarres.
- Écrire `ord(lettre) + cle` sans retirer `base` puis le remettre : le calcul décale
  alors le code du caractère, pas la lettre dans l'alphabet.
- Traiter les espaces comme des lettres : le message se déforme et ne se relit plus.

---

## Partie 2 — Casser par force brute

**Éléments attendus :**

```python
def casser(texte_chiffre):
    """Affiche les 25 décalages possibles : la bonne clé se repère à l'œil."""
    for cle in range(1, 26):
        print(cle, chiffrer(texte_chiffre, -cle))
```

Exécution sur le message intercepté (extrait) :

```text
1  rk igsout vgyyk g sojo
...
7  le camion passe a midi
...
25 tm kiuqwv xiaam i uqlq
```

La clé est **7**, et le message est **le camion passe a midi**.

**Ce qu'il faut retenir :** l'attaque ne demande aucune clé, aucun secret, aucune
hypothèse sur le contenu — seulement la patience d'essayer les 25 possibilités et un
œil pour reconnaître le français. C'est la définition d'une attaque par force brute.

**Erreurs fréquentes :**

- Boucler de 0 à 25 inclus : la clé 26 redonne le texte chiffré lui-même, ce qui fait
  26 lignes au lieu de 25 et noie la bonne réponse.
- Afficher les résultats sans la clé testée : impossible de la rapporter ensuite.
- Chercher la réponse « au hasard » plutôt que de parcourir systématiquement les
  clés — la force brute est justement une énumération complète.

---

## Partie 3 — Expliquer la limite de César

**Éléments attendus :**

Le chiffre de César ne protège pas deux choses. D'abord la clé : il n'existe que 25
décalages utiles, donc un adversaire qui a le message chiffré les essaie tous en une
fraction de seconde. Ensuite le texte lui-même : les mots restent séparés par les
mêmes espaces, les mots de même longueur gardent la même forme, et la fréquence des
lettres du chiffré reproduit celle du français — simplement décalée.

**Ce qu'il faut retenir :** la question n'est pas « le texte est-il transformé ? »
mais « combien de clés faut-il essayer, et qu'est-ce que le chiffré laisse encore
voir ? ». César échoue sur les deux tableaux.

---

## Partie 4 — Relier à la cryptographie moderne

**Éléments attendus :**

Augmenter le décalage ne change rien : il y a toujours 25 décalages à essayer, et
l'analyse de fréquence fonctionne quelle que soit la valeur choisie. Un chiffrement
comme AES (Advanced Encryption Standard) répond aux deux faiblesses : sa clé est
tirée dans un espace si vaste que la force brute ne le parcourt pas, et sa
transformation ne laisse filtrer ni les fréquences, ni les répétitions, ni la longueur
des mots.

**Sur la formulation :** « plus c'est compliqué, plus c'est sûr » est un réflexe
naturel et faux. La sécurité se mesure : taille de l'espace des clés, et quantité
d'information que le chiffré divulgue. Un chiffrement simple et bien conçu bat un
chiffrement compliqué et mal conçu.

---

## Points à retenir

- Un chiffrement symétrique repose sur **une clé partagée** et une transformation
  réversible — déchiffrer, c'est chiffrer en sens inverse.
- La robustesse se juge d'abord à la **taille de l'espace des clés** : 25 clés
  possibles, c'est zéro secret.
- Un chiffré divulgue encore des **structures** (espaces, longueurs, fréquences) :
  les casser ne demande pas de clé.
- AES corrige les deux points : espace des clés immense, et aucune structure du texte
  clair visible dans le texte chiffré.
