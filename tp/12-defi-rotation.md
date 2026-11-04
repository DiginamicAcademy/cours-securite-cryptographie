# Exercice — Défi : planifier une rotation de clés

> **Nature :** élément intégrable dans le TP (défi de conception, facultatif) · **Durée :** 1 h 15 (indicatif) · **Notion :** rotation de clés — bascule sans interruption de service
> **Lien :** chapitre [Compromis et réponse](../12-compromis.md)
> **Objectif :** concevoir un plan de rotation de clés qui ne coupe jamais le service.
> Cet exercice est **facultatif** : il n'est jamais noté, jamais chronométré, et ne
> bloque pas la progression de la journée.

---

## Contexte

**Cadence** est une plateforme de billetterie en ligne. Les données des acheteurs
(courriel, téléphone, coordonnées de remboursement) sont chiffrées au niveau
applicatif avec une clé de données, `CLE_DONNEES`, stockée dans le coffre de secrets.
La politique de sécurité impose une rotation **au moins trimestrielle**, et
immédiate en cas de suspicion d'exposition.

Trois contraintes rendent l'opération délicate :

1. le service est ouvert en continu — aucune interruption de service n'est acceptable,
   y compris pendant les pics de billetterie ;
2. un **traitement de nuit** réconcilie les paiements et déchiffre les coordonnées de
   remboursement avec la même clé ;
3. un **partenaire** reçoit chaque semaine un export chiffré, et déchiffre de son côté
   avec la clé qui lui a été transmise.

À cela s'ajoute l'historique : trois années de données chiffrées avec la clé
actuelle, dans l'entrepôt.

---

## Consignes

Produisez un **document de planification** — pas du code. Il couvre les points
suivants, dans l'ordre que vous jugez bon :

1. **Les phases de la rotation** : ce qui se passe, dans quel ordre, avec pour chaque
   phase son point de contrôle et son critère de passage à la suivante.
2. **Les trois contraintes** : comment le traitement de nuit et le partenaire
   traversent la rotation sans interruption, et qui porte la responsabilité de chaque
   bascule.
3. **La preuve de complétude** : comment savoir, à la fin, que plus rien n'utilise
   l'ancienne clé — et que faire si un lecteur oublié réapparaît.
4. **Le repli** : ce qui se passe si la rotation échoue en cours de route, et comment
   revenir en arrière sans perdre les écritures faites pendant la tentative.
5. **La communication** : ce que vous dites aux équipes, au moment où vous le dites, et
   ce que vous documentez pour la rotation suivante.

---

## Attendu

- un plan structuré en phases, avec responsables, points de contrôle et critères de
  réussite ;
- une procédure de repli explicite ;
- une note de communication (quelques lignes suffisent) destinée aux équipes.

---

## ✅ Auto-évaluation

Un défi n'a pas de corrigé pas-à-pas : c'est votre plan qui est l'objet. Vérifiez-le
vous-même :

- [ ] Mon plan décrit une **période à deux clés** — jamais un remplacement brutal.
- [ ] Chaque phase a un point de contrôle observable, et un critère pour passer à la suivante.
- [ ] Le traitement de nuit et le partenaire sont traités explicitement, avec un responsable nommé.
- [ ] Ma preuve de complétude ne repose pas sur « on a regardé le code ».
- [ ] Le repli est possible **pendant** toute la rotation, y compris pour les écritures déjà migrées.
- [ ] La communication est prévue avant le début de l'opération, pas après l'incident.

---

## Pour aller plus loin

*Facultatif.* Reprenez votre plan pour le cas d'une rotation **d'urgence**, déclenchée
par la suspicion d'une fuite : quelles phases raccourcit-on, lesquelles ne sautent-on
jamais, et comment gère-t-on le fait que l'ancienne clé est peut-être déjà entre de
mauvaises mains ? Proposez ensuite le déclenchement **automatique** de la rotation
périodique, et ce qu'il faut tracer pour qu'un auditeur puisse la vérifier sans vous
croire sur parole.
