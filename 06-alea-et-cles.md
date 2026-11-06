[← Chapitre précédent](05-erreurs-classiques.md) · [Sommaire](Readme.md) · [Chapitre suivant →](07-tls-en-pratique.md)

# L'aléa et la gestion des clés

> Les armées changent de code après chaque bataille : non que l'ancien chiffrement fût
> mauvais, mais une clé qui a trop servi finit par fuiter — capture, indiscrétion,
> usure des habitudes. La qualité d'un système se juge donc autant à la façon dont il
> **génère** ses clés qu'à celle dont il les **remplace**.

**Durée de lecture :** 35 min hors approfondissements facultatifs · **Prérequis :** [Du chiffre de César à AES](02-chiffrement-symetrique.md)

---

## Objectifs

À l'issue de ce chapitre, vous serez capable de :

- **Distinguer** un générateur pseudo-aléatoire d'un générateur adapté à la cryptographie.
- **Générer** une clé avec un outil sûr et **la stocker** hors du code source.
- **Décrire** le cycle de vie d'une clé : création, usage, rotation, destruction.
- **Déclencher** une rotation et **révoquer** une clé compromise.
- **Rédiger** un plan de gestion des clés pour une application qui traite des données sensibles.

---

## I. L'aléa, matière première de toute clé

Une clé n'est sûre que si elle est **imprévisible**. Le chapitre
[Du chiffre de César à AES](02-chiffrement-symetrique.md) a posé le mécanisme : le hasard d'un ordinateur est
simulé à partir d'une **graine** (*seed*), et `random` (PRNG) se reconstitue dès qu'on
connaît cette graine — parfait pour un jeu, impropre à la cryptographie. Le générateur
adapté est donc un CSPRNG : `secrets` en Python, `os.urandom`, ou les appels système
dédiés.

Ce que ces appels ajoutent, c'est l'**entropie** : ils puisent dans l'activité
imprévisible de la machine — horloge à haute résolution, bruit matériel, interruptions —
et sont conçus pour qu'une partie de leur sortie ne permette pas de prédire la suite.

> [!DANGER]
> Une graine faible ou prévisible, c'est une clé devinable — et un chiffrement qui ne
> protège rien, si parfait soit l'algorithme. En révision de code, toute apparition de
> `random` dans un module qui touche aux clés, aux jetons ou aux mots de passe est un
> défaut.

---

## II. Générer, stocker, distribuer

Le cycle de vie d'une clé compte plusieurs étapes, et chacune a ses exigences.

| Étape | Ce qu'on fait | Ce qu'on ne fait jamais |
| --- | --- | --- |
| Création | Tirer la clé avec un CSPRNG, à la longueur recommandée | La dériver d'un mot de passe faible, ou la coder en dur |
| Stockage | Coffre de secrets, gestionnaire de clés, ou KMS (*Key Management System*), ou fichier protégé hors dépôt | La commiter dans le code, la coller dans un ticket, l'envoyer par courriel |
| Distribution | Canal chiffré, ou enveloppement avec la clé publique du destinataire | La transmettre à côté du message qu'elle protège |
| Usage | Une clé, un usage : ne pas mélanger chiffrement de données et signature | Réutiliser la même clé partout « par simplicité » |
| Rotation | Remplacer la clé à intervalle ou sur événement | Attendre qu'elle ait fuité |
| Destruction | Invalider, puis effacer les exemplaires | Garder l'ancienne clé « au cas où » |

### La clé n'a pas sa place dans le code

Un secret écrit dans le code source se retrouve dans l'historique du dépôt, les copies
locales et les sauvegardes du serveur de code — et il y reste même après suppression. La
règle est simple : **le code se partage, les clés non**. On les injecte à l'exécution :
variable d'environnement, configuration hors dépôt, ou service de gestion de secrets.

> [!TIP]
> Le bon réflexe en cas de doute : demandez-vous « si ce dépôt devenait public
> demain, qu'est-ce qui serait compromis ? ». Si la réponse inclut une donnée réelle,
> un secret y traîne.

### Une clé, un usage

Mélanger les usages multiplie les dégâts en cas de fuite : la clé qui chiffre les
exports ne doit pas signer des jetons, ni celle qui authentifie un service chiffrer les
sauvegardes. Quand une clé tombe, on sait exactement ce qui est touché — et ce qui ne
l'est pas.

Pour les données au repos, on peut **dériver** des clés spécifiques d'une clé maîtresse
avec une fonction de dérivation de clé, ou KDF (*Key Derivation Function*) : chaque usage
reçoit sa propre clé, sans en gérer des dizaines à la main.

---

## III. Rotation et révocation

La rotation n'est pas une formalité : c'est la limite de ce qu'un adversaire peut
obtenir en volant une clé.

### Quand tourner une clé ?

- **À intervalle régulier**, défini par la politique de l'organisation — par exemple à
  chaque trimestre pour des données sensibles.
- **À chaque événement suspect** : départ d'une personne, fuite suspectée, compromission
  d'un système voisin, découverte d'un secret dans un dépôt.
- **Quand l'usage change** : une clé qui passe d'un environnement de test à un
  environnement de production se remplace, elle ne se déplace pas.
- **Quand l'algorithme ou la taille de clé est dépréciée** : migrer avant d'être pris par
  la fin de vie.

### Comment tourner sans casser le service ?

La rotation se prépare : les messages déjà chiffrés avec l'ancienne clé doivent rester
lisibles. On versionne donc les clés — chaque message porte la référence de celle qui
l'a produit — et on conserve l'ancienne en lecture seule le temps de la migration :

1. générer la nouvelle clé ;
2. chiffrer les **nouveaux** messages avec elle ;
3. conserver l'ancienne pour **déchiffrer** l'existant ;
4. migrer les données au repos ;
5. détruire l'ancienne clé, et le constater.

### Révoquer, c'est décider vite

Une clé compromise se révoque **immédiatement**, avant même de comprendre comment la
fuite s'est produite. La révocation rend la clé inutilisable pour les opérations
futures mais ne protège pas ce qui a déjà été chiffré avec — c'est là que la rotation
régulière prend son sens : elle borne la fenêtre d'exposition.

> [!WARNING]
> Une clé dont on a perdu la trace est une clé compromise. L'inventaire — où sont mes
> clés, qui y a accès, quand elles ont tourné — est la première pièce d'une gestion des
> clés, avant tout choix d'algorithme.

---

### Exercice bonus — Concevoir un plan de gestion des clés (1 h 30, facultatif)

*Cet exercice est facultatif : il ne bloque jamais la progression de la journée. Il n'est ni noté ni chronométré : on y va quand les autres blocs sont solides.*

#### Contexte

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

#### Consigne

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

#### Attendu

- Un document d'une à trois pages, structuré en six sections.
- Un tableau d'inventaire initial, même partiel.
- Une procédure de rotation applicable sans interruption de service.

#### Vérification

Un plan est jugé recevable quand :

- chaque clé identifiée a un **propriétaire** et un **emplacement** ;
- la rotation décrite **ne casse pas** la lecture des données déjà chiffrées ;
- la section incident dit **qui décide** et **dans quel délai** ;
- le plan distingue explicitement test et production.

#### ✅ Auto-évaluation

Avant de confronter votre travail à la grille de vérification, vérifiez que :

- [ ] Mon inventaire liste au moins les quatre familles de secrets de la plateforme.
- [ ] Chaque ligne de l'inventaire nomme un responsable et un emplacement de stockage.
- [ ] Ma procédure de rotation préserve la lisibilité des données existantes.
- [ ] Ma section incident fixe un décideur et un délai.
- [ ] Mon plan distingue les environnements de test et de production.
- [ ] Je peux défendre chaque choix devant une équipe qui ne fait pas de sécurité.

#### Pour aller plus loin (exercice)

*Facultatif.* Ajoutez une section sur la **preuve** : comment démontrer à un auditeur
que la rotation a bien eu lieu, et que les anciennes clés ont été détruites.

<details>
<summary>Voir la correction</summary>

#### Comment utiliser cette grille

Pour chaque section, cochez ce que votre plan contient réellement. Une case non cochée
signale un trou à combler — pas une erreur de formulation. Un plan dont toutes les
cases sont cochées est recevable en l'état.

#### 1. Inventaire

- [ ] Les quatre familles de secrets sont recensées : chiffrement des données au repos,
      jetons d'API (Application Programming Interface), comptes de service, certificats.
- [ ] Chaque entrée nomme un **propriétaire** (une personne ou une équipe, jamais
      « l'informatique »).
- [ ] Chaque entrée indique l'**emplacement** de stockage et l'**usage**.
- [ ] Les secrets encore en clair dans les scripts sont listés explicitement, avec leur
      remédiation.

**Ce qui fait la différence :** un inventaire qui distingue ce qui existe de ce qui
traîne encore. Le script signalé dans l'énoncé doit y figurer — sinon le plan part d'un
constat incomplet.

#### 2. Création et stockage

- [ ] La création se fait avec un générateur adapté à la cryptographie (et non un
      générateur pseudo-aléatoire ordinaire).
- [ ] Le stockage est **hors du code source**, avec un emplacement unique par
      environnement.
- [ ] Test et production sont **séparés** : jamais la même clé des deux côtés.
- [ ] Les accès au stockage de clés sont eux-mêmes protégés et journalisés.

#### 3. Distribution et accès

- [ ] Le plan dit **qui** a accès à **quelle** clé, et selon quel rôle.
- [ ] La distribution se fait par un canal chiffré ou par enveloppement avec la clé
      publique du destinataire.
- [ ] Les accès sont revus à chaque changement d'équipe (arrivée, départ, changement de
      poste).
- [ ] Aucune clé n'est transmise par courriel, ticket ou messagerie instantanée.

#### 4. Rotation

- [ ] Une **périodicité** est fixée pour chaque famille de clés.
- [ ] Les **événements déclencheurs** sont listés (départ, fuite suspectée, changement
      d'usage, dépréciation d'algorithme).
- [ ] La procédure **versionne** les clés : les données déjà chiffrées restent lisibles.
- [ ] La migration des données au repos est prévue, avec une étape de destruction de
      l'ancienne clé.

**Ce qui fait la différence :** une rotation qui préserve la lecture de l'existant. Un
plan qui remplace la clé sans versionner casse la plateforme — et sera contourné par les
équipes.

#### 5. Révocation et incident

- [ ] Un **décideur** est nommé, avec un **délai** pour révoquer.
- [ ] La révocation est **immédiate** dès qu'une clé est suspecte, avant l'analyse de la
      fuite.
- [ ] La communication interne est décrite : qui est prévenu, dans quel ordre.
- [ ] La remédiation couvre ce qui a déjà été chiffré avec la clé compromise (rotation
      des données, notification si nécessaire).

#### 6. Destruction

- [ ] Les critères de destruction sont posés (fin d'usage, fin de conservation, fin de
      migration).
- [ ] La destruction est **prouvable** (journal, attestation) et pas seulement déclarée.
- [ ] Les copies (sauvegardes, réplicas, exports) sont couvertes par la même exigence.

#### Ce qu'un bon plan apporte de plus

- **Des responsables nommés**, pas des équipes anonymes.
- **Des délais**, pas « dès que possible ».
- **Une traçabilité** : on peut démontrer après coup ce qui a été fait.
- **Une proportionnalité** : la lourdeur de la procédure est calibrée sur la sensibilité
  des données — un plan applicable vaut mieux qu'un plan parfait qu'on ignore.

#### Points à retenir

- Un plan de gestion des clés commence par un **inventaire** : sans lui, la rotation est
  impossible.
- Chaque clé a un **propriétaire**, un **emplacement**, un **usage** — et une date de
  fin.
- La rotation se conçoit pour **ne pas casser** la lecture des données existantes :
  versionner, migrer, détruire.
- La révocation est **immédiate** et **décidée** : un plan sans décideur ni délai ne
  sert pas le jour de l'incident.
- La destruction se **prouve** — y compris dans les sauvegardes.

</details>

---

## IV. Aller plus loin

*Ces approfondissements sont facultatifs : ils ne sont jamais exigés pour terminer la
journée.*

- **Les modules matériels** : un HSM (*Hardware Security Module*) stocke les clés dans
  un boîtier qui les utilise sans jamais les laisser sortir. Les grands systèmes de
  paiement et les autorités de certification fonctionnent ainsi — le coût se justifie
  quand la clé protège un patrimoine considérable.
- **Les coffres de secrets** : des outils comme Vault centralisent les secrets, en
  journalisent l'accès et en automatisent la rotation. Ils ajoutent une dépendance
  critique — leur propre disponibilité devient un enjeu — et se protègent en conséquence.
- **La rotation automatisée** : dans une plateforme data (Airflow, MinIO, entrepôts), les
  clés de connexion aux sources se renouvellent par script, avec journalisation. Une
  rotation manuelle est une rotation qu'on repousse.
- **L'inventaire comme exercice** : lister les clés d'un projet réel — y compris les
  jetons d'API (Application Programming Interface), les mots de passe de service, les
  certificats — est souvent la meilleure heure de travail de la semaine.

---

## Points clés

- Le hasard se **génère** : `random` est prévisible, `secrets` (CSPRNG) convient. Une
  graine faible, c'est une clé devinable.
- Une clé se **tire** au hasard, se **stocke** hors du code, se **distribue** par canal
  chiffré, se **détruit** quand son usage s'arrête.
- **Une clé, un usage** : les mélanger multiplie les dégâts d'une fuite.
- La **rotation** borne la fenêtre d'exposition ; la **révocation** est immédiate.
- On **versionne** les clés pour que les anciens messages restent lisibles, puis on
  détruit l'ancienne.
- L'**inventaire** des clés précède tout choix d'algorithme : une clé perdue de vue est
  une clé compromise.

---

## 🧠 Vérifiez vos acquis

*Formatif, jamais noté. Répondez sans relire le chapitre, puis dépliez « Vérifier ses réponses ». Ce qui compte est de repérer où vous vacillez, pas de faire 5 sur 5.*

**1.** Qu'est-ce qu'un CSPRNG comme `secrets` ou `os.urandom` ajoute par rapport à `random` ?

- A. Une sortie plus longue à chaque appel : il produit des clés de 512 bits là où `random` s'arrête à 128
- B. De l'entropie : il puise dans l'activité imprévisible de la machine
- C. Un algorithme tenu secret, contrairement à `random` dont le code est public et donc attaquable
- D. Une graine fixe, qui rend la suite reproductible et donc vérifiable par un auditeur

**2.** Pourquoi « une clé, un usage » ?

- A. Parce qu'une clé ne peut techniquement servir qu'à un seul algorithme : le format du secret l'impose
- B. Parce que mélanger les usages multiplie les dégâts d'une fuite : une clé qui tombe, on sait exactement ce qui est touché
- C. Parce que les clés sont numérotées dans le coffre et qu'un numéro ne peut désigner qu'un seul usage
- D. Parce qu'un usage unique accélère le chiffrement : la clé reste en cache dans le processus

**3.** On fait tourner la clé qui chiffre les exports de DIGICHEESE. Pourquoi versionner les clés (chaque message porte la référence de celle qui l'a produit) ?

- A. Pour que les messages déjà chiffrés avec l'ancienne restent lisibles pendant la migration
- B. Pour accélérer le chiffrement des nouveaux messages : la version évite de tester les clés une par une
- C. Pour pouvoir détruire l'ancienne clé immédiatement après la rotation, sans avoir rien à vérifier
- D. Pour prouver auprès de l'auditeur que la rotation a bien eu lieu, grâce à un horodatage des messages

**4.** Que fait une révocation de clé ?

- A. Elle rend la clé inutilisable pour les opérations futures, mais ne protège pas ce qui a déjà été chiffré avec
- B. Elle rend illisibles toutes les données chiffrées avec cette clé, y compris les archives conservées
- C. Elle remplace la clé par une nouvelle, sans intervention humaine ni reprise des données existantes
- D. Elle n'a d'effet qu'après la publication de l'inventaire des clés auprès des équipes concernées

**5.** *Question ouverte (2-3 lignes).* Une clé de service utilisée depuis six mois est découverte dans un dépôt devenu public. Décrivez la réaction, dans l'ordre, et dites ce que la rotation régulière aurait changé rétrospectivement.

---

<details>
<summary>Vérifier ses réponses</summary>

### Réponses

**1. B.** L'entropie : horloge à haute résolution, bruit matériel, interruptions — et la conception fait qu'une partie de la sortie ne permet pas de prédire la suite.
**Pourquoi :** une graine faible ou prévisible, c'est une clé devinable, donc un chiffrement qui ne protège rien si parfait soit l'algorithme.

**2. B.** La clé qui chiffre les exports ne doit pas signer des jetons, ni celle qui authentifie un service chiffrer les sauvegardes.
**Pourquoi :** quand une clé tombe, le périmètre touché doit être connu ; pour les données au repos, une KDF (*Key Derivation Function*) dérive une clé spécifique par usage depuis une clé maîtresse.

**3. A.** On conserve l'ancienne clé en lecture seule le temps de la migration : générer la nouvelle, chiffrer les nouveaux messages avec elle, déchiffrer l'existant avec l'ancienne, migrer les données au repos, puis détruire l'ancienne et le constater.
**Pourquoi :** une rotation qui rend les archives illisibles n'est pas une rotation, c'est une perte de données.

**4. A.** La révocation est immédiate, avant même de comprendre la fuite — mais elle ne revient pas sur le passé.
**Pourquoi :** c'est précisément là que la rotation régulière prend son sens : elle **borne la fenêtre d'exposition**.

**5.** Attendu : révoquer **immédiatement** la clé exposée, émettre et injecter la nouvelle (hors dépôt : variable d'environnement, coffre), vérifier que le secret ne reste pas dans l'historique du dépôt et dans les copies, inventorier ce qu'elle protégeait et ce qui a pu être lu pendant six mois, puis documenter l'événement et déclencher la rotation des voisines. Rétrospectivement, une rotation régulière aurait **borné la fenêtre d'exposition** : six mois de validité, c'est la surface attaquable.
**Pourquoi :** une clé dont on a perdu la trace est une clé compromise — l'inventaire (où sont mes clés, qui y a accès, quand elles ont tourné) précède tout choix d'algorithme.

</details>

---

[← Chapitre précédent](05-erreurs-classiques.md) · [Sommaire](Readme.md) · [Chapitre suivant →](07-tls-en-pratique.md)
