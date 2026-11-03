# Corrigé — Défi : planifier une rotation de clés

> Corrigé de [l'exercice « Défi : planifier une rotation de clés »](../12-defi-rotation.md).
> Un défi n'a pas de solution unique : ce document propose une **grille de
> vérification**, puis **un exemple de plan commenté**. Servez-vous-en pour juger le
> vôtre, pas pour le recopier.

---

## Grille de vérification

Un plan de rotation se juge sur ces critères — chacun doit pouvoir être vérifié par
quelqu'un qui n'a pas participé à l'opération :

| Critère | Ce qui le vérifie concrètement |
| --- | --- |
| Aucune interruption de service | Le service répond pendant toute l'opération ; aucune phase n'exige d'arrêt |
| Période à deux clés | Les deux clés sont valides simultanément pendant une fenêtre explicite |
| Écritures tracées | Chaque valeur chiffrée porte la marque de la clé qui l'a produite |
| Consommateurs couverts | Application, traitement de nuit, partenaire, entrepôt — chacun a un responsable nommé |
| Preuve de complétude | Une trace exploitable (journal du coffre, inventaire des chiffrés) montre que l'ancienne clé n'est plus lue |
| Repli possible | À chaque phase, une procédure de retour existe et a été décrite **avant** le début |
| Communication | Les équipes sont prévenues avant l'opération, informées de son résultat |

---

## Un exemple de plan, commenté

### Phase 0 — Préparer (sans toucher à la production)

On recense **qui lit la clé** : l'application, le traitement de nuit, le partenaire,
l'entrepôt, et — toujours — quelques consommateurs oubliés (un carnet de
recherche, un export automatisé, une sauvegarde de configuration). Ce recensement est
le livrable le plus précieux de toute l'opération : sans lui, la preuve de complétude
est impossible.

*Point de contrôle :* le registre des consommateurs est complet et chaque ligne a un
responsable.

### Phase 1 — Émettre la nouvelle clé, à côté de l'ancienne

La nouvelle clé rejoint le coffre **sans remplacer** l'ancienne. Les deux sont
valides. Pour distinguer ce qui a été chiffré avec l'une ou l'autre, chaque valeur
stockée porte une marque — une colonne `cle_id`, ou un préfixe dans la valeur. Sans
cette marque, la lecture devine, et la migration devient un casse-tête.

*Point de contrôle :* une valeur chiffrée par la nouvelle clé est lue correctement par
l'application, et la marque de clé est bien présente.

### Phase 2 — Rendre les lecteurs ambidextres

Avant toute bascule d'écriture, **tous** les lecteurs savent déchiffrer avec les deux
clés : l'application, le traitement de nuit, le partenaire. C'est l'étape que l'on
voudrait sauter — et celle qui fait échouer les rotations précipitées. Le traitement
de nuit, en particulier, tourne quand personne ne surveille : on le fait tourner une
fois en mode de vérification avant de le laisser partir seul.

*Point de contrôle :* chaque consommateur a lu au moins une valeur de chaque clé, en
conditions réelles.

### Phase 3 — Basculer les écritures

Les nouvelles écritures utilisent la nouvelle clé. L'ancienne reste active pour la
lecture. Rien ne change pour l'utilisateur ; la proportion de données à l'ancienne
clé décroît à chaque écriture.

*Point de contrôle :* les journaux montrent que plus aucune écriture ne produit de
valeur marquée par l'ancienne clé.

### Phase 4 — Migrer l'historique, par tranches

Le ré-chiffrement des données existantes se fait en arrière-plan, par tranches, avec
un point de reprise : une migration interrompue reprend là où elle s'est arrêtée, sans
repartir de zéro. Pendant cette phase, l'ancienne clé est encore nécessaire — c'est
normal, et cela ne contredit pas la phase 1 : elle est **isolée**, pas encore
détruite.

*Point de contrôle :* le compteur des valeurs restant à l'ancienne clé baisse à chaque
tranche, et la relecture d'un échantillon migré est vérifiée.

### Phase 5 — Traiter le partenaire à part

Le partenaire déchiffre ses exports **chez lui**, avec la clé qui lui a été transmise.
Deux situations coexistent : les anciens exports, chiffrés avec l'ancienne clé, et les
nouveaux. On lui transmet la nouvelle clé par un canal distinct de celui des exports,
avec une date de bascule convenue — et on décide explicitement du sort des anciens
exports : soit ils sont re-chiffrés pour lui, soit l'ancienne clé reste conservée le
temps de leur durée de vie. Ce point se tranche **avant** la phase suivante.

*Point de contrôle :* le partenaire confirme par écrit qu'il lit les deux formats.

### Phase 6 — Prouver, puis retirer

La preuve de complétude ne repose jamais sur « on a regardé le code » :

- le **journal du coffre** ne montre plus de lecture de l'ancienne clé depuis la date
  de bascule ;
- un **inventaire des valeurs chiffrées** ne trouve plus aucune marque de l'ancienne
  clé ;
- une **alerte** est posée sur toute tentative d'utilisation de l'ancienne clé, pour
  attraper le lecteur oublié qui réapparaîtrait.

Alors seulement, l'ancienne clé est révoquée — et sa copie de secours traitée selon la
décision prise pour le partenaire.

*Point de contrôle :* l'alerte est active avant la révocation, pas après.

### Repli

À toute phase jusqu'à la révocation, le retour en arrière est possible : les
écritures portent leur marque de clé, un lecteur qui accepte les deux formats relit
donc l'ensemble, quelle que soit la clé utilisée entre-temps. Le repli se décrit
**avant** l'opération — pendant un incident, personne n'écrit de procédure.

---

## Les points que l'on rate souvent

- **Les exports du partenaire.** L'ancienne clé reste nécessaire tant qu'il existe un
  fichier chiffré avec elle, quelque part, chez quelqu'un. Détruire la clé trop tôt
  transforme une rotation en perte de données.
- **Le lecteur oublié.** Un carnet de recherche, un export automatisé, une sauvegarde
  de configuration : le registre de la phase 0 les couvre, l'alerte de la phase 6 les
  attrape.
- **Le traitement de nuit.** Il tourne sans surveillance : le faire passer en mode de
  vérification une fois coûte une soirée, l'oublier coûte une nuit de paiements non
  réconciliés.
- **La communication après coup.** Les équipes prévenues avant l'opération signalent
  les consommateurs que vous n'aviez pas vus ; prévenues après, elles découvrent les
  pannes.

---

## Points à retenir

- Une rotation sans interruption est une **période à deux clés**, jamais un
  remplacement brutal.
- Chaque valeur chiffrée **porte la marque de sa clé** : c'est ce qui rend la lecture,
  la migration et le repli possibles.
- La preuve de complétude se lit dans des **traces** — journal du coffre, inventaire
  des chiffrés, alerte sur l'ancienne clé — pas dans le code.
- L'ancienne clé survit à la rotation aussi longtemps qu'un fichier chiffré avec elle
  existe quelque part.
- Le plan, le repli et la communication s'écrivent **avant** l'opération.
