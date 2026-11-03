# Exercice — Classer les primitives

> 🟢 **Débutant** · **Durée :** 20 min · **Lien :** chapitre [Histoire et services de la cryptographie](../01-histoire-et-services.md)
> **Objectif :** associer chaque besoin de sécurité à la primitive qui le couvre (hachage, chiffrement, signature).

---

## Contexte

La fromagerie **DIGICHEESE** (la fromagerie du fil rouge) finalise sa boutique en
ligne. L'équipe a noté ses besoins
de sécurité au tableau blanc, mais personne n'est d'accord sur la technique à
employer : « on chiffre tout », « on hache tout », « une signature, c'est pareil
qu'un chiffrement »…

Vous êtes appelé à trancher. Aucun code n'est nécessaire : il s'agit de **choisir la
bonne primitive pour le bon problème**.

---

## Consignes

### Partie 1 — Reformuler le besoin

Pour chaque situation ci-dessous, dites en une phrase ce qu'il faut garantir :
**cacher** l'information, **vérifier** qu'elle n'a pas changé, ou **prouver** son
origine.

1. Les prix négociés avec les fournisseurs de la fromagerie ne doivent pas être connus
   des concurrents.
2. Un client qui télécharge la fiche technique d'un fromage doit pouvoir vérifier
   qu'elle n'a pas été modifiée en route.
3. La comptable envoie un bon de commande au fournisseur ; celui-ci doit pouvoir
   prouver, plus tard, que c'est bien elle qui l'a envoyé.
4. Les mots de passe des comptes clients sont conservés dans une base de données.
5. Un partenaire extérieur veut transmettre un document confidentiel à la fromagerie,
   sans que les deux parties aient pu se rencontrer pour se mettre d'accord au
   préalable sur un secret commun.
6. Le navigateur du client affiche un cadenas : le client veut s'assurer qu'il parle
   bien au serveur de la boutique, et pas à un site qui se fait passer pour elle.

### Partie 2 — Associer usage et primitive

Complétez le tableau suivant. Pour chaque situation, retenez **une** primitive
principale parmi : **hachage**, **chiffrement symétrique**, **chiffrement
asymétrique**, **signature**.

| Situation | Besoin (cacher / vérifier / prouver) | Primitive principale |
| --- | --- | --- |
| 1 | | |
| 2 | | |
| 3 | | |
| 4 | | |
| 5 | | |
| 6 | | |

### Partie 3 — Justifier un choix

Pour les situations 2 et 4, expliquez en deux phrases pourquoi la primitive retenue
convient — et pourquoi une autre ne conviendrait pas.

### Partie 4 — Détecter le contresens

Un développeur de l'équipe déclare :

> « J'ai haché le fichier des prix pour que personne ne puisse le lire. »

Expliquez en deux ou trois phrases pourquoi cette affirmation est fausse, et ce
qu'il aurait fallu faire à la place.

---

## ✅ Auto-évaluation

Avant de passer au corrigé, vérifiez que :

- [ ] J'ai reformulé les six besoins avec mes propres mots (cacher / vérifier / prouver).
- [ ] Mon tableau attribue **une** primitive principale à chaque situation.
- [ ] J'ai justifié les situations 2 et 4, y compris en écartant une autre primitive.
- [ ] Ma réponse à la partie 4 distingue clairement **hachage** et **chiffrement**.
- [ ] Je peux expliquer à voix haute pourquoi « chiffrer » et « hacher » ne sont pas interchangeables.

---

## Pour aller plus loin

*Facultatif.* Parmi les six situations, lesquelles gagneraient à combiner **deux**
primitives ? Proposez la combinaison et dites ce qu'elle apporte.
