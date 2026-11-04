# Défi — Concevoir un plan de gestion des clés

> **Nature :** élément intégrable dans le TP (défi de conception, facultatif) · **Durée :** 1 h 30 (indicatif) · **Notion :** gestion des clés — plan de vie des clés (inventaire, stockage, rotation, destruction)
> **Lien :** chapitre [L'aléa et la gestion des clés](../06-alea-et-cles.md)
> **Objectif :** rédiger un plan de gestion des clés complet pour une plateforme data
> qui traite des données personnelles.

*Cet exercice est facultatif : il ne bloque jamais la progression de la journée. Il n'est ni noté ni chronométré : on y va quand les autres blocs sont solides.*

---

## Contexte

**AquaVallée**, un opérateur de distribution d'eau, exploite une plateforme data qui
collecte les relevés de compteurs et les coordonnées des abonnés. La chaîne est la
suivante :

- des compteurs connectés envoient leurs relevés vers une file d'attente ;
- un orchestrateur (Airflow) charge les données dans un stockage objet (MinIO) ;
- un entrepôt alimente les tableaux de bord de facturation ;
- un export mensuel part vers un prestataire d'analyse, sous format CSV (comma-separated
  values).

Les données sont personnelles : le RGPD (Règlement général sur la protection des
données) s'applique. L'entreprise vient de découvrir qu'une clé de chiffrement figure
depuis deux ans dans un script d'import — et que personne ne sait dire combien d'autres
clés existent, ni où elles sont.

Le directeur des systèmes d'information vous demande un **plan de gestion des clés**,
qui servira de référence pour les équipes.

---

## Consigne

Rédigez le plan sous forme de document structuré. Il doit couvrir au moins les points
suivants :

1. **Inventaire** — recenser les clés et secrets de la plateforme (chiffrement des
   données au repos, jetons d'API (Application Programming Interface), comptes de
   service, certificats), et indiquer pour chacun : propriétaire, emplacement, usage.
2. **Création et stockage** — fixer la manière dont une clé naît et où elle vit,
   en distinguant les environnements de test et de production.
3. **Distribution et accès** — définir qui a le droit de voir ou d'utiliser chaque clé,
   et par quel canal elle transite.
4. **Rotation** — fixer les périodicités et les événements qui déclenchent une rotation,
   et décrire la procédure pour que les données déjà chiffrées restent lisibles.
5. **Révocation et incident** — décrire ce qu'on fait le jour où une clé est suspecte,
   y compris la communication interne et la remédiation.
6. **Destruction** — préciser quand et comment une clé est détruite, et comment on le
   prouve.

Pour chaque section, le plan doit dire **qui fait quoi** — un plan sans responsable est
une intention, pas une procédure.

---

## Attendu

- Un document d'une à trois pages, structuré en six sections.
- Un tableau d'inventaire initial, même partiel.
- Une procédure de rotation applicable sans interruption de service.

---

## Vérification

Un plan est jugé recevable quand :

- chaque clé identifiée a un **propriétaire** et un **emplacement** ;
- la rotation décrite **ne casse pas** la lecture des données déjà chiffrées ;
- la section incident dit **qui décide** et **dans quel délai** ;
- le plan distingue explicitement test et production.

---

## ✅ Auto-évaluation

Avant de confronter votre travail à la grille de vérification, vérifiez que :

- [ ] Mon inventaire liste au moins les quatre familles de secrets de la plateforme.
- [ ] Chaque ligne de l'inventaire nomme un responsable et un emplacement de stockage.
- [ ] Ma procédure de rotation préserve la lisibilité des données existantes.
- [ ] Ma section incident fixe un décideur et un délai.
- [ ] Mon plan distingue les environnements de test et de production.
- [ ] Je peux défendre chaque choix devant une équipe qui ne fait pas de sécurité.

---

## Pour aller plus loin

*Facultatif.* Ajoutez une section sur la **preuve** : comment démontrer à un auditeur
que la rotation a bien eu lieu, et que les anciennes clés ont été détruites.
