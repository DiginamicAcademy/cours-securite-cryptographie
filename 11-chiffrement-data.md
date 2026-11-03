[← Chapitre précédent](10-secrets.md) · [Sommaire](Readme.md) · [Chapitre suivant →](12-compromis.md)

# Chiffrement des données data

> Une chaîne de traitement extrait chaque nuit les données clients d'une base de
> production, les transforme dans un orchestrateur, puis les charge dans un entrepôt
> analytique. Sur ce trajet, tout passe par le réseau. Si un seul tronçon n'est pas
> chiffré, c'est du clair qui circule — et personne ne s'en aperçoit, car rien ne casse.

**Durée de lecture :** 55 min · **Prérequis :** les chapitres [Données au repos](09-donnees-au-repos.md)
et [Gestion des secrets](10-secrets.md)

---

## Objectifs

À l'issue de ce chapitre, vous serez capable de :

- **Cartographier** les flux d'une chaîne data et repérer les tronçons qui transitent
  en clair.
- **Appliquer** le chiffrement des flux entre les briques d'une chaîne data :
  orchestrateur, stockage objet, base, entrepôt.
- **Chiffrer** les colonnes sensibles d'un entrepôt et séparer les clés par domaine de
  données.
- **Réduire** la surface d'exposition par la minimisation des données, conformément au
  RGPD.

---

## I. Chiffrer les flux

### Le trajet d'une donnée

Une chaîne data moderne est une suite de sauts : base de production → zone de
dépôt temporaire (*staging*) → orchestrateur (Airflow, par exemple) → stockage objet
(MinIO ou un équivalent S3) → entrepôt → outils de restitution. Chaque flèche de ce
schéma est un **fil**, et chaque fil peut être écouté :

```text
Base de prod ──> Staging ──> Orchestrateur ──> Stockage objet ──> Entrepôt ──> BI
     │              │              │                 │              │
   TLS ?          TLS ?          TLS ?             TLS ?          TLS ?
```

L'objection classique est « tout cela reste dans notre réseau ». Elle ne tient plus :
les clusters sont partagés, les services sont managés par des tiers, les accès de
support traversent le périmètre, et une machine compromise dans le réseau lit tout ce
qui passe en clair. Le chiffrement des flux ne coûte presque rien — son absence non
plus, jusqu'au jour où elle coûte tout.

### Les mesures, par tronçon

| Flux | Risque si en clair | Mesure |
| --- | --- | --- |
| Application → base | Identifiants et données lus par un observateur réseau | Exiger TLS côté serveur de base, refuser les connexions non chiffrées |
| Orchestrateur → stockage objet | Fichiers de staging interceptés ou modifiés | Point d'entrée HTTPS uniquement, certificats vérifiés |
| Orchestrateur → base / entrepôt | Identifiants de connexion et requêtes | Connexions chiffrées + secrets injectés (chapitre précédent) |
| Entrepôt → outil de restitution | Agrégats et données détaillées | HTTPS sur les API, TLS sur les connecteurs BI |
| Livraison à un tiers | Tout le fichier | Chiffrement du fichier lui-même, clé transmise par un autre canal |

Deux réflexes complètent le tableau :

- **Refuser le clair**, plutôt que le déconseiller : une base qui accepte encore les
  connexions non chiffrées les acceptera, tôt ou tard, pour quelqu'un.
- **Chiffrer au-delà du réseau** quand la donnée quitte la zone de confiance : un
  fichier livré à un partenaire se chiffre pour lui seul, même si le transport est
  déjà chiffré — le transport protège le trajet, le chiffrement du fichier protège le
  contenu chez le destinataire.

> [!TIP]
> La cartographie des flux se fait en une heure : lister les flèches de la chaîne,
> puis pour chacune noter « chiffré / en clair / inconnu ». La case « inconnu » est
> la plus instructive — c'est celle que personne ne surveille.

---

## II. Chiffrer l'entrepôt

### Pourquoi l'entrepôt est un sujet à part

Un entrepôt agrège **tout, longtemps, pour beaucoup de monde** : des années de
données, des dizaines d'analystes, des outils de restitution, des exports CSV qui
repartent sur des portables. La surface d'exposition y est plus large que dans la base
de production, et la durée de rétention bien plus longue. C'est là que le chiffrement
au repos et le chiffrement de colonne prennent tout leur sens.

### Chiffrer par colonne, séparer par domaine

Le principe est celui du chapitre « Données au repos » : les attributs sensibles —
identifiants, coordonnées, données de santé, consommations rattachées à une personne —
sont chiffrés par l'application avant l'écriture. Ce qui change en contexte data, c'est
la **gestion des clés** :

- une clé par **domaine** de données (par table, par client, par filière) plutôt
  qu'une clé unique : la fuite d'une clé n'expose qu'un domaine ;
- les clés rangées dans un coffre, avec une **clé maître** qui les chiffre à leur tour
  (chiffrement par enveloppe) : tourner la clé maître ne force pas à rechiffrer
  l'entrepôt ;
- chaque accès aux clés **tracé** : qui a déchiffré quoi, quand.

### Le compromis analytique

Une colonne chiffrée ne s'agrège plus : pas de `SUM`, pas de `GROUP BY`, pas de
jointure. Les équipes data contournent le problème de trois façons, à choisir selon
l'usage :

- **déchiffrer dans une vue contrôlée** : l'entrepôt stocke le chiffré, une vue
  réservée déchiffre pour les traitements qui l'exigent ;
- **pseudonymiser** : remplacer l'identifiant par un jeton stable, la clé de
  ré-identification étant conservée **ailleurs** — les analyses restent possibles, la
  donnée brute reste hors de portée ;
- **agréger en amont** : ne charger dans l'entrepôt que des totaux, des comptages, des
  profils — jamais la ligne individuelle.

> [!WARNING]
> Un export CSV est une fuite qui ne dit pas son nom. Chiffrer l'entrepôt, puis
> laisser un analyste copier la table entière sur son portable, annule l'effort :
> la donnée a quitté la zone protégée. Les exports se limitent, se tracent, et se
> chiffrent quand ils sortent.

---

## III. RGPD et minimisation

### La meilleure donnée protégée est celle qu'on ne stocke pas

Le **RGPD** (Règlement général sur la protection des données, adopté en 2016,
applicable depuis 2018) pose des principes qui rejoignent la sécurité par la boutade
du métier : **minimisation** (ne collecter que le nécessaire), **limitation de la
finalité** (ne pas réutiliser au-delà de ce qui a été annoncé), et **limitation de la
conservation** (ne pas garder indéfiniment). Chaque donnée non stockée est une donnée
qui ne peut ni fuir, ni être détournée, ni être réclamée en cas d'incident.

Concrètement, dans une chaîne data :

- supprimer les colonnes qui ne servent à aucun traitement réel ;
- **anonymiser** ce qui doit rester analysable : une donnée réellement anonymisée —
  irréversiblement, avec vérification — sortit du champ du RGPD ;
- **pseudonymiser** ce qui doit rester rattachable, en gardant la table de
  ré-identification dans un autre système, sous un autre contrôle ;
- définir une **durée de conservation** par table, et l'appliquer réellement (une
  règle écrite sans suppression automatique est un vœu).

> [!TIP]
> Anonymiser n'est pas masquer. Supprimer le nom tout en gardant l'adresse exacte, la
> date de naissance et le compteur individuel permet de ré-identifier la personne en
> quelques croisements. La question n'est pas « ai-je retiré un champ ? » mais «
> peut-on, avec ce qui reste, remonter à une personne ? »

> 🧪 **Atelier rapide (5 min)**
>
> Voici un extrait de table tel qu'il arrive dans l'entrepôt d'une collectivité :
>
> | id | nom | email | adresse | date_naissance | compteur | horodatage | kwh |
> | --- | --- | --- | --- | --- | --- | --- | --- |
> | 1 | A. Martin | a.martin@exemple.fr | 12 rue des Lilas, 34000 Montpellier | 1978-04-12 | FR7X-001 | 2026-03-01 00:00 | 4,2 |
> | 2 | B. Petit | b.petit@exemple.fr | 5 avenue de la Mer, 34000 Montpellier | 1991-11-03 | FR7X-002 | 2026-03-01 00:00 | 6,8 |
>
> L'objectif métier de l'entrepôt est un **reporting de consommation agrégée par
> quartier et par mois**. En une minute : quelles colonnes peut-on supprimer, lesquelles
> agréger, lesquelles faut-il garder — et lesquelles pseudonymiser ? Notez votre
> découpage : il sert de point de départ à la réflexion sur la minimisation.

---

## IV. Aller plus loin

*Ces approfondissements sont facultatifs : ils ne sont jamais exigés pour terminer la
journée.*

- **Chiffrement homomorphe** : il permet de calculer **sur** des données chiffrées —
  additionner, agréger, sans jamais déchiffrer. Le résultat, une fois déchiffré, est
  celui qu'on aurait obtenu en clair. La théorie est mature, les performances restent
  l'obstacle : quelques opérations sur des jeux réduits plutôt que des tableaux
  entiers. C'est la direction de recherche la plus active du domaine.
- **Les enclaves de calcul** (*confidential computing*) : le traitement s'exécute dans
  une zone mémoire protégée par le matériel, illisible même pour l'administrateur de
  la machine. Utile quand le calcul doit se faire chez un hébergeur que l'on ne fait
  pas entièrement confiance.
- **La tokenisation** : remplacer la donnée sensible par un jeton sans rapport
  mathématique avec elle, la correspondance vivant dans un coffre dédié. Plus simple
  qu'un chiffrement à gérer soi-même, à condition d'accepter la dépendance au coffre.

---

## Points clés

- Chaque flèche d'une chaîne data est un fil : **cartographier**, puis chiffrer —
  refuser le clair plutôt que le déconseiller.
- Quand la donnée quitte la zone de confiance, on chiffre **le contenu**, pas
  seulement le transport.
- L'entrepôt cumule volume, durée et multiplicité des accès : chiffrement de colonne,
  **clés par domaine**, chiffrement par enveloppe.
- Une colonne chiffrée ne s'analyse plus : vue contrôlée, pseudonymisation ou
  agrégation en amont — trois réponses à trois usages.
- Le RGPD rejoint la sécurité par la **minimisation** : la donnée non stockée ne fuit
  pas. Anonymiser réellement, ou pseudonymiser en séparant la ré-identification.

---

[← Chapitre précédent](10-secrets.md) · [Sommaire](Readme.md) · [Chapitre suivant →](12-compromis.md)
