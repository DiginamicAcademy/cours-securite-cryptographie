# Exercice — Casser le chiffre de César

> **Nature :** exercice · **Durée :** 30 min · **Notion :** chiffrement symétrique — décalage, espace des clés, force brute
> **Lien :** chapitre [Du chiffre de César à AES](../02-chiffrement-symetrique.md)
> **Objectif :** implémenter le chiffre de César en Python, le casser par force brute,
> puis expliquer pourquoi un chiffrement moderne résiste là où César cède.

---

## Contexte

**Transports Delmas**, une PME (petite et moyenne entreprise) de transport routier, organise ses tournées par de
courts messages entre la réception et les chauffeurs : « le camion passe à midi »,
« quai nord », etc. Pour qu'un message lu par-dessus l'épaule ne livre rien, les
équipes utilisent un chiffrement par décalage — le chiffre de César — avec une clé
changée chaque semaine.

Cette semaine, la clé est **7**. Un message a été intercepté sur un parking :

```text
sl jhtpvu whzzl h tpkp
```

Vous disposez du code de départ ci-dessous, qui ne fait encore rien d'utile.

```python
def chiffrer(texte, cle):
    """Chiffre un message par décalage (lettres minuscules, reste inchangé)."""
    return texte  # à compléter
```

Quelques repères pour ne pas partir dans le décor : l'alphabet en compte **26**
lettres, et le décalage **26** ramène au texte clair — il n'y a donc que **25**
décalages utiles à essayer pour casser un message.

---

## Consignes

### Partie 1 — Chiffrer et déchiffrer

1. Complétez `chiffrer(texte, cle)` : chaque lettre minuscule est décalée de `cle`
   rangs dans l'alphabet, en bouclant en fin d'alphabet (x devient a, y devient b, z
   devient c). Les espaces et la ponctuation sont conservés tels quels.
2. Écrivez `dechiffrer(texte, cle)`, qui retrouve le texte clair. Faites en sorte de
   **réutiliser** `chiffrer` plutôt que de recopier la logique.
3. Chiffrez le message `le camion passe a midi` avec la clé 7, puis vérifiez que
   `dechiffrer` le restitue exactement.

### Partie 2 — Casser par force brute

1. Écrivez `casser(texte_chiffre)` : la fonction affiche les **25** décalages
   possibles (clés de **1** à **25**), chacun précédé de la clé testée.
2. Lancez-la sur le message intercepté `sl jhtpvu whzzl h tpkp`.
3. Identifiez la clé qui donne un français lisible, et notez le message retrouvé.

### Partie 3 — Expliquer la limite de César

En trois phrases au maximum, expliquez pourquoi ce chiffrement ne protège rien dès
que l'adversaire a le message chiffré. Votre réponse doit mentionner **la taille de
l'espace des clés** et **ce que le texte chiffré laisse encore deviner**.

### Partie 4 — Relier à la cryptographie moderne

Un collègue affirme : « il suffit d'augmenter le décalage pour rendre le chiffre
solide ». Répondez-lui en deux phrases, puis nommez les deux propriétés d'un
chiffrement moderne comme AES (Advanced Encryption Standard) qui répondent
précisément à ce que César ne protège pas.

---

## Attendu

- Un script Python contenant `chiffrer`, `dechiffrer` et `casser`, exécutable de bout
  en bout.
- Le message clair retrouvé pour le message intercepté.
- Deux réponses courtes écrites (parties 3 et 4).

---

## Vérification

Vous avez réussi quand :

- `dechiffrer(chiffrer(texte, 7), 7)` renvoie `texte` pour n'importe quelle phrase en
  minuscules ;
- `casser` affiche bien 25 lignes, dont une seule est lisible ;
- votre explication de la partie 3 cite le nombre de clés possibles ;
- votre réponse de la partie 4 nomme une propriété liée à la **clé** et une liée au
  **texte chiffré**.

---

## ✅ Auto-évaluation

Avant de passer au corrigé, vérifiez que :

- [ ] J'ai implémenté `chiffrer` et `dechiffrer` sans recopier deux fois la même logique.
- [ ] Mon aller-retour chiffrement / déchiffrement restitue le texte d'origine.
- [ ] J'ai affiché les 25 décalages et repéré la bonne clé à l'œil.
- [ ] J'ai expliqué pourquoi la force brute suffit à casser César.
- [ ] J'ai relié cette faiblesse à une propriété d'un chiffrement moderne.

---

## Pour aller plus loin

*Facultatif.* Modifiez `chiffrer` pour qu'elle conserve aussi les majuscules, puis
pour qu'elle accepte un décalage négatif sans changer de signature. Vérifiez que
`dechiffrer` fonctionne toujours.
