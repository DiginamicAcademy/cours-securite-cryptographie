[← Chapitre précédent](11-chiffrement-data.md) · [Sommaire](Readme.md) · [Chapitre suivant →](A-Glossaire-et-ressources.md)

# Compromis et réponse

> La cryptographie a un coût : cycles de processeur, latence, complexité
> opérationnelle. Trop de chiffrement, et l'application ralentit au point que les
> équipes le désactivent ; trop peu, et la fuite est là. La sécurité n'est pas un
> réglage que l'on pousse à fond : c'est un arbitrage permanent entre protection,
> performance et sobriété.

**Durée de lecture :** 40 min hors approfondissements facultatifs · **Prérequis :** les chapitres précédents de la journée

---

## Objectifs

À l'issue de ce chapitre, vous serez capable de :

- **Mesurer** le coût d'un chiffrement sur un traitement réel et le comparer à son
  bénéfice de protection.
- **Arbitrer** entre protection, performance et coût opérationnel, en justifiant le
  choix devant un interlocuteur non technique.
- **Conduire** la réponse à une fuite de clé : révoquer, évaluer, ré-chiffrer,
  notifier.

---

## I. Coûts et performances

### Ce que le chiffrement coûte réellement

Chiffrer demande des opérations mathématiques ; les faire tourner consomme du
processeur, de la mémoire, du temps. Mais le coût se répartit rarement là où on
l'imagine :

| Poste de coût | Ce qu'il représente | Ordre de grandeur |
| --- | --- | --- |
| Chiffrement / déchiffrement des données | Calcul pur | Faible : les processeurs récents chiffrent plusieurs gigaoctets par seconde, grâce à l'accélération matérielle |
| Poignée TLS | Négociation réseau + calcul | Modéré, surtout à la connexion ; amorti par la reprise de session |
| Appel à un coffre de clés (KMS) | Aller-retour réseau | Élevé s'il est fait par ligne lue — à mettre en cache |
| Déchiffrement applicatif par ligne | Calcul + aller-retour | Cumulatif sur de gros volumes |
| Stockage | Le chiffré est légèrement plus volumineux | Négligeable dans la plupart des cas |

La conclusion pratique : **le chiffrement n'est presque jamais le goulot
d'étranglement**. Ce qui coûte, c'est l'architecture qui l'entoure — un appel réseau
par enregistrement, une clé relue à chaque requête, une poignée de main refaite pour
chaque requête HTTP. Mesurer avant d'optimiser : un benchmark sur le traitement réel
vaut mieux que dix intuitions.

```python
import os
import timeit

from cryptography.fernet import Fernet

f = Fernet(Fernet.generate_key())
donnee = os.urandom(1024)

chiffrer = lambda: f.encrypt(donnee)
print("chiffrement :", timeit.timeit(chiffrer, number=1000), "s pour 1000 appels")
```

La mesure répond à « ce chiffrement ralentit-il mon traitement ? », jamais à « la
sécurité, ça coûte cher ? ». La bonne question reste : **protège-t-on proportionnellement
à la valeur des données ?**

> [!TIP]
> Avant de chiffrer une colonne, demandez quelle décision métier s'appuie dessus. Une
> donnée que personne ne lit ne mérite ni index ni déchiffrement systématique ; une
> donnée critique mérite le chiffrement, même si elle coûte quelques millisecondes.

---

## II. Sobriété

Chiffrer, c'est calculer, et calculer se paie. Deux leviers intéressent directement
l'arbitrage sécurité / performance : **réutiliser** — maintenir les connexions ouvertes
et la reprise de session TLS plutôt que renégocier à chaque requête — et **ne pas
stocker** ce qui ne sert pas, la minimisation du chapitre précédent étant aussi un gain
de calcul. Les leviers plus techniques (choix de l'algorithme selon le matériel,
rechiffrement évité par enveloppe) sont en fin de chapitre.

> [!WARNING]
> Une protection que les utilisateurs contournent ne protège rien. Un chiffrement qui
> rend un service inutilisable finit désactivé, avec l'approbation de tout le monde.
> La sobriété, ici, est un garde-fou : elle force à chiffrer **là où cela compte**, et
> à l'expliquer.

---

## III. Fuite de clé

### Le scénario

Une clé de chiffrement apparaît dans un dépôt public, un journal d'application, une
sauvegarde oubliée — ou elle est emportée par une personne qui quitte l'équipe. Peu
importe la voie : à partir de maintenant, **on suppose qu'elle est entre de mauvaises
mains**. Les quatre actions prioritaires s'enchaînent dans un ordre précis.

| Ordre | Action | Pourquoi maintenant |
| --- | --- | --- |
| 1 | **Révoquer** la clé : la rendre inutilisable, isoler les systèmes qui l'emploient | Chaque minute qui passe est une minute d'exploitation possible |
| 2 | **Évaluer** l'exposition : quels données, quels systèmes, sur quelle période, avec quels accès | On ne répare que ce que l'on sait touché — et on documente pour la suite |
| 3 | **Ré-chiffrer** les données avec une clé neuve, et remplacer la clé partout où elle sert | Les données déjà chiffrées avec la clé fuitée restent lisibles pour l'attaquant tant qu'elles ne sont pas rechiffrées |
| 4 | **Notifier** : en interne d'abord, puis les personnes concernées si des données personnelles sont en jeu | L'obligation légale ne dépend pas de la gravité ressentie, mais de la donnée touchée |

À ces quatre actions s'ajoute le travail de fond : comprendre **comment** la clé a
fuité, et fermer la voie — sinon la clé suivante suivra le même chemin.

> 🧪 **Atelier rapide (8 min)**
>
> Scénario : la clé de chiffrement de la base de réservations d'une plateforme de
> location de vélos est retrouvée dans l'historique d'un dépôt public, où elle traîne
> depuis plusieurs semaines. En huit minutes, écrivez vos **quatre actions
> prioritaires**, dans l'ordre, avec pour chacune : qui la conduit, et ce qui se passe
> si elle est faite dans le mauvais ordre. Comparez votre ordre au tableau ci-dessus.

### Ré-chiffrer sans tout casser

Le ré-chiffrement d'un gros volume est un chantier : il prend du temps, mobilise les
clés anciennes et nouvelles, et ne doit pas interrompre le service. La méthode est
celle des migrations de clés : **période à deux clés**, migration progressive des
enregistrements, vérification de la relecture, puis retrait de l'ancienne clé. Pendant
tout le chantier, la clé fuitée reste nécessaire pour déchiffrer l'ancien — d'où
l'importance de l'étape 1 : l'isoler, même si on ne peut pas encore s'en passer.

---

### Exercice bonus — Planifier une rotation de clés (1 h 15, facultatif)

*Cet exercice est **facultatif** : il n'est jamais noté, jamais chronométré, et ne bloque pas la progression de la journée.*

#### Contexte

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

#### Consignes

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

#### Attendu

- un plan structuré en phases, avec responsables, points de contrôle et critères de
  réussite ;
- une procédure de repli explicite ;
- une note de communication (quelques lignes suffisent) destinée aux équipes.

#### ✅ Auto-évaluation

Un défi n'a pas de corrigé pas-à-pas : c'est votre plan qui est l'objet. Vérifiez-le
vous-même :

- [ ] Mon plan décrit une **période à deux clés** — jamais un remplacement brutal.
- [ ] Chaque phase a un point de contrôle observable, et un critère pour passer à la suivante.
- [ ] Le traitement de nuit et le partenaire sont traités explicitement, avec un responsable nommé.
- [ ] Ma preuve de complétude ne repose pas sur « on a regardé le code ».
- [ ] Le repli est possible **pendant** toute la rotation, y compris pour les écritures déjà migrées.
- [ ] La communication est prévue avant le début de l'opération, pas après l'incident.

#### Pour aller plus loin (exercice)

*Facultatif.* Reprenez votre plan pour le cas d'une rotation **d'urgence**, déclenchée
par la suspicion d'une fuite : quelles phases raccourcit-on, lesquelles ne sautent-on
jamais, et comment gère-t-on le fait que l'ancienne clé est peut-être déjà entre de
mauvaises mains ? Proposez ensuite le déclenchement **automatique** de la rotation
périodique, et ce qu'il faut tracer pour qu'un auditeur puisse la vérifier sans vous
croire sur parole.

<details>
<summary>Voir la correction</summary>

#### Grille de vérification

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

#### Un exemple de plan, commenté

##### Phase 0 — Préparer (sans toucher à la production)

On recense **qui lit la clé** : l'application, le traitement de nuit, le partenaire,
l'entrepôt, et — toujours — quelques consommateurs oubliés (un carnet de
recherche, un export automatisé, une sauvegarde de configuration). Ce recensement est
le livrable le plus précieux de toute l'opération : sans lui, la preuve de complétude
est impossible.

*Point de contrôle :* le registre des consommateurs est complet et chaque ligne a un
responsable.

##### Phase 1 — Émettre la nouvelle clé, à côté de l'ancienne

La nouvelle clé rejoint le coffre **sans remplacer** l'ancienne. Les deux sont
valides. Pour distinguer ce qui a été chiffré avec l'une ou l'autre, chaque valeur
stockée porte une marque — une colonne `cle_id`, ou un préfixe dans la valeur. Sans
cette marque, la lecture devine, et la migration devient un casse-tête.

*Point de contrôle :* une valeur chiffrée par la nouvelle clé est lue correctement par
l'application, et la marque de clé est bien présente.

##### Phase 2 — Rendre les lecteurs ambidextres

Avant toute bascule d'écriture, **tous** les lecteurs savent déchiffrer avec les deux
clés : l'application, le traitement de nuit, le partenaire. C'est l'étape que l'on
voudrait sauter — et celle qui fait échouer les rotations précipitées. Le traitement
de nuit, en particulier, tourne quand personne ne surveille : on le fait tourner une
fois en mode de vérification avant de le laisser partir seul.

*Point de contrôle :* chaque consommateur a lu au moins une valeur de chaque clé, en
conditions réelles.

##### Phase 3 — Basculer les écritures

Les nouvelles écritures utilisent la nouvelle clé. L'ancienne reste active pour la
lecture. Rien ne change pour l'utilisateur ; la proportion de données à l'ancienne
clé décroît à chaque écriture.

*Point de contrôle :* les journaux montrent que plus aucune écriture ne produit de
valeur marquée par l'ancienne clé.

##### Phase 4 — Migrer l'historique, par tranches

Le ré-chiffrement des données existantes se fait en arrière-plan, par tranches, avec
un point de reprise : une migration interrompue reprend là où elle s'est arrêtée, sans
repartir de zéro. Pendant cette phase, l'ancienne clé est encore nécessaire — c'est
normal, et cela ne contredit pas la phase 1 : elle est **isolée**, pas encore
détruite.

*Point de contrôle :* le compteur des valeurs restant à l'ancienne clé baisse à chaque
tranche, et la relecture d'un échantillon migré est vérifiée.

##### Phase 5 — Traiter le partenaire à part

Le partenaire déchiffre ses exports **chez lui**, avec la clé qui lui a été transmise.
Deux situations coexistent : les anciens exports, chiffrés avec l'ancienne clé, et les
nouveaux. On lui transmet la nouvelle clé par un canal distinct de celui des exports,
avec une date de bascule convenue — et on décide explicitement du sort des anciens
exports : soit ils sont re-chiffrés pour lui, soit l'ancienne clé reste conservée le
temps de leur durée de vie. Ce point se tranche **avant** la phase suivante.

*Point de contrôle :* le partenaire confirme par écrit qu'il lit les deux formats.

##### Phase 6 — Prouver, puis retirer

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

##### Repli

À toute phase jusqu'à la révocation, le retour en arrière est possible : les
écritures portent leur marque de clé, un lecteur qui accepte les deux formats relit
donc l'ensemble, quelle que soit la clé utilisée entre-temps. Le repli se décrit
**avant** l'opération — pendant un incident, personne n'écrit de procédure.

#### Les points que l'on rate souvent

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

#### Points à retenir

- Une rotation sans interruption est une **période à deux clés**, jamais un
  remplacement brutal.
- Chaque valeur chiffrée **porte la marque de sa clé** : c'est ce qui rend la lecture,
  la migration et le repli possibles.
- La preuve de complétude se lit dans des **traces** — journal du coffre, inventaire
  des chiffrés, alerte sur l'ancienne clé — pas dans le code.
- L'ancienne clé survit à la rotation aussi longtemps qu'un fichier chiffré avec elle
  existe quelque part.
- Le plan, le repli et la communication s'écrivent **avant** l'opération.

</details>

---

## IV. Aller plus loin

🔁 **Pour aller plus loin (facultatif)** — à lire si le créneau le permet, rien
d'obligatoire n'en dépend.

### Les quatre leviers d'éco-conception

À l'échelle d'une machine, chiffrer ne consomme rien ; à celle d'une plateforme qui
établit des millions de connexions par jour, ou d'un cluster dont chaque disque est
chiffré, cela compte. Sobriété et sécurité se rejoignent sur un principe : ne faire que
les calculs utiles. Du plus simple au plus structurant :

- **réutiliser** : maintenir les connexions ouvertes, activer la reprise de session
  TLS, plutôt que renégocier à chaque requête ;
- **choisir l'algorithme adapté au matériel** : AES tirant parti de l'accélération
  matérielle là où elle existe, ChaCha20 sur les appareils qui en sont dépourvus —
  même résultat de protection, moins de cycles perdus ;
- **éviter le rechiffrement inutile** : le chiffrement par enveloppe permet de tourner
  une clé maître sans rechiffrer des téraoctets de données ;
- **ne pas stocker** ce qui ne sert pas — la minimisation est aussi une mesure
  d'éco-conception.

### Et le quantique, dans tout ça ?

- **La cryptographie post-quantique** : les algorithmes d'échange de clés et de
  signature actuels (RSA, courbes elliptiques) reposent sur des problèmes qu'un
  ordinateur quantique suffisamment puissant résoudrait. Le NIST a publié en 2024 ses
  premiers standards post-quantiques (ML-KEM pour l'échange de clés, ML-DSA pour la
  signature), et les grands acteurs du web déploient déjà des échanges de clés
  **hybrides** : classique et post-quantique en parallèle, pour être sûr des deux.
- **« Récolter maintenant, déchiffrer plus tard »** : un attaquant qui enregistre
  aujourd'hui du trafic chiffré peut espérer le déchiffrer le jour où un ordinateur
  quantique existera. C'est pourquoi les données à longue durée de vie — santé,
  état civil, secrets industriels — méritent déjà des algorithmes résistants, ou une
  protection par confidentialité permanente.
- **Le coût de la transition** : remplacer les algorithmes partout prend des années —
  dans les cartes, les protocoles, les bases, les objets connectés. Inventorier ce qui
  utilise la cryptographie est le premier chantier, bien avant le remplacement lui-même.

---

## Points clés

- Le chiffrement est rarement le goulot d'étranglement : ce qui coûte, c'est
  l'architecture autour — appels réseau, poignées de main répétées, clés relues.
- **Mesurer** sur le traitement réel, puis arbitrer : la protection suit la valeur des
  données, pas l'inverse.
- La sobriété rejoint la sécurité : réutiliser les connexions, choisir l'algorithme
  adapté au matériel, éviter le rechiffrement inutile, ne pas stocker le superflu.
- Face à une fuite de clé : **révoquer, évaluer, ré-chiffrer, notifier** — dans cet
  ordre, puis fermer la voie par laquelle elle est partie.
- Un chiffrement se décide **au moment du choix**, pas après : la durée de vie attendue
  de la donnée fait partie du cahier des charges, au même titre que sa sensibilité.

---

## 🧠 Vérifiez vos acquis

*Formatif, jamais noté. Répondez sans relire le chapitre, puis dépliez « Vérifier ses réponses ». Ce qui compte est de repérer où vous vacillez, pas de faire 5 sur 5.*

**1.** Un traitement lit un million de lignes et déchiffre chacune après un appel au coffre de clés. Où est le coût, d'après le tableau du chapitre ?

- A. Dans le calcul du chiffrement : les processeurs récents sont lents en la matière
- B. Dans l'aller-retour réseau répété sur le coffre de clés : ce coût-là se met en cache
- C. Dans le stockage : le chiffré est beaucoup plus volumineux
- D. Dans la poignée TLS, refaite à chaque ligne

**2.** Un chiffrement appliqué à un service interne le rend lent au point que les équipes demandent sa désactivation. Que pose le chapitre comme garde-fou ?

- A. Il faut maintenir la contrainte : la sécurité ne se négocie pas, et la lenteur est le prix à payer
- B. Une protection que les utilisateurs contournent ne protège rien : la sobriété fait chiffrer là où cela compte
- C. Il faut chiffrer uniquement les sauvegardes, et laisser les services internes circuler en clair au quotidien
- D. Il faut remplacer le chiffrement par un contrôle d'accès : c'est plus rapide, et cela protège au même point

**3.** Pendant le ré-chiffrement d'un gros volume après fuite de clé, l'ancienne clé (fuitée) reste nécessaire. Pourquoi, et qu'en déduit le cours ?

- A. Rien : on supprime la clé fuitée dès la première minute, et les données anciennes sont alors perdues
- B. Elle sert encore à déchiffrer l'ancien pendant la période à deux clés : d'où l'isoler, quitte à ne pas s'en passer
- C. Elle sert à chiffrer les nouvelles écritures pendant le chantier, la nouvelle clé n'intervenant qu'après
- D. Elle n'est plus nécessaire : le ré-chiffrement se fait en une passe, la nuit, sans période de coexistence

**4.** Avant de chiffrer une colonne, quelle attitude le chapitre recommande-t-il ?

- A. Chiffrer systématiquement : le coût est négligeable avec le matériel récent, la question ne se pose donc pas
- B. Mesurer sur le traitement réel, et demander quelle décision métier s'appuie sur la donnée : protège-t-on selon sa valeur ?
- C. Se fier à l'intuition sur ce que « la sécurité, ça coûte cher » : l'expérience de l'équipe suffit en général
- D. Comparer avec les autres applications du même éditeur : elles ont déjà tranché, on reprend leur choix

**5.** *Question ouverte (3 lignes).* La clé de chiffrement de la base de réservations d'une plateforme de location de vélos est retrouvée dans l'historique d'un dépôt public, où elle traîne depuis plusieurs semaines. Donnez dans l'ordre les quatre actions prioritaires, et dites ce que chaque inversion d'ordre coûterait.

---

<details>
<summary>Vérifier ses réponses</summary>

### Réponses

**1. B.** Le calcul, lui, est **faible** : les processeurs récents chiffrent plusieurs gigaoctets par seconde grâce à l'accélération matérielle, et le surcroît de stockage est négligeable.
**Pourquoi :** le chiffrement n'est presque jamais le goulot d'étranglement — ce qui coûte, c'est l'architecture qui l'entoure : un appel réseau par enregistrement, une clé relue à chaque requête, une poignée de main refaite pour chaque requête HTTP.

**2. B.** Un chiffrement qui rend un service inutilisable finit désactivé, avec l'approbation de tout le monde.
**Pourquoi :** la sobriété est un garde-fou, pas un renoncement — elle concentre le chiffrement là où il protège vraiment quelque chose.

**3. B.** La méthode est celle des migrations de clés : période à deux clés, migration progressive, vérification de la relecture, puis retrait de l'ancienne.
**Pourquoi :** tant que les données ne sont pas rechiffrées, elles restent lisibles pour l'attaquant — mais la clé fuitée sert encore à lire l'ancien, donc on l'**isole** à l'étape 1 même sans pouvoir s'en passer.

**4. B.** Mesurer avant d'optimiser : un benchmark sur le traitement réel vaut mieux que dix intuitions.
**Pourquoi :** la bonne question n'est pas « la sécurité, ça coûte cher ? » mais « protège-t-on proportionnellement à la valeur des données ? » — une donnée que personne ne lit ne mérite ni index ni déchiffrement systématique.

**5.** Attendu, dans l'ordre : **révoquer** (rendre la clé inutilisable, isoler les systèmes qui l'emploient) ; **évaluer** l'exposition (quelles données, quels systèmes, sur quelle période, avec quels accès) ; **ré-chiffrer** avec une clé neuve et remplacer la clé partout où elle sert ; **notifier** (en interne, puis les personnes concernées si des données personnelles sont en jeu). Il faut ensuite comprendre **comment** la clé a fuité et fermer la voie.
**Pourquoi :** chaque minute sans révocation est une minute d'exploitation possible ; ré-chiffrer avant d'évaluer, c'est réparer à l'aveugle ; notifier sans avoir révoqué, c'exposer pendant la communication ; et l'obligation légale de notification ne dépend pas de la gravité ressentie, mais de la donnée touchée.

</details>

---

[← Chapitre précédent](11-chiffrement-data.md) · [Sommaire](Readme.md) · [Chapitre suivant →](A-Glossaire-et-ressources.md)
