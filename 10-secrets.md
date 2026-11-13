[← Chapitre précédent](09-donnees-au-repos.md) · [Sommaire](Readme.md) · [Chapitre suivant →](11-chiffrement-data.md)

# Gestion des secrets

> En septembre 2023, les chercheurs de la société Wiz découvrent qu'un jeton d'accès
> à un stockage Azure traîne dans un dépôt GitHub public tenu par une équipe de
> recherche de Microsoft. Le jeton, trop puissant et daté de 2051, ouvre **38
> téraoctets** de données privées : des sauvegardes de postes de travail, des clés, des
> mots de passe, plus de 30 000 messages internes. Le jeton était là depuis presque
> trois ans. Un fichier `.env` commité par erreur, c'est exactement la même chose — à
> l'échelle de votre projet.

**Durée de lecture :** 50 min hors approfondissements facultatifs · **Prérequis :** les notions de clé et de secret (journée
précédente)

---

## Objectifs

À l'issue de ce chapitre, vous serez capable de :

- **Identifier** les endroits où un secret fuit dans un projet : code source, image
  conteneur, historique Git, fichiers de CI, journaux.
- **Décrire** ce qu'un attaquant peut faire d'un secret découvert, et pourquoi la
  révocation prime sur le nettoyage.
- **Mettre en œuvre** une gestion des secrets par variables d'environnement et coffre
  de secrets.
- **Planifier** la rotation d'un secret sans interruption de service.

---

## I. Pourquoi c'est grave

### Un secret commité n'est plus un secret

Le scénario est toujours le même : un développeur écrit une clé dans un fichier de
configuration « pour faire vite », le fichier part dans le dépôt, et l'affaire est
classée. Or un dépôt Git n'est pas un tiroir :

- **l'historique garde tout** : supprimer la ligne au commit suivant ne l'efface pas,
  elle reste dans les versions précédentes ;
- **le dépôt se copie partout** : clones locaux, forks, miroirs, caches de CI,
  sauvegardes du serveur Git ;
- **les outils indexent** : les robots des plateformes de code parcourent les dépôts
  publics en continu, à la recherche exactement de ces motifs — une clé exposée est
  souvent exploitée en quelques minutes.

C'est la leçon de l'incident de 2023 : ni algorithme cassé, ni injection sophistiquée —
juste **une chaîne de caractères laissée au mauvais endroit**, et un jeton dont
personne n'avait borné la durée de vie ni la portée.

### La clé à côté des données

Le constat est plus ancien que l'incident ci-dessus, et il a un nom : la **PlayStation
Network**, en 2011 — 77 millions de comptes, raconté la journée précédente. Les mots de
passe y étaient chiffrés, mais la clé dormait sur le même périmètre que les données
protégées. L'attaquant qui prend l'un prend l'autre.

> [!DANGER]
> Chiffrer avec une clé rangée à côté du chiffré, c'est enfermer la clé dans la
> boîte. La valeur d'un chiffrement est **exactement** la valeur de la séparation
> entre la donnée et sa clé.

### Où les secrets se cachent

| Endroit | Ce que l'attaquant y trouve | Remède |
| --- | --- | --- |
| Code source, fichier `.env` commité | La clé en clair, dans l'historique | Variables d'environnement + coffre |
| `Dockerfile`, image conteneur | La clé cuite dans l'image, distribuée avec | Injecter au démarrage, jamais à la construction |
| Fichier de CI/CD | Jeton de déploiement à hauts privilèges | Coffre de secrets de la plateforme de CI |
| Journaux et traces de débogage | La clé affichée par une requête mal filtrée | Masquer les valeurs sensibles à l'affichage |
| Sauvegarde de la base | Clés et données au même endroit | Séparer la clé de la sauvegarde |

> 🧪 **Atelier rapide (8 min)**
>
> Ouvrez le fichier `lab-securite/docker-compose.yml` de l'environnement de démonstration et
> cherchez `APP_SECRET_KEY`. Répondez, en notant vos réponses :
>
> 1. Où la valeur est-elle écrite, et qui peut la lire ?
> 2. Que se passe-t-il si cette valeur est celle du dépôt de production ?
> 3. Regardez ensuite comment l'application la lit (`lab-securite/secure/app.py`) : que
>    constatez-vous sur le comportement par défaut quand la variable est absente ?
>
> Notez les trois réponses : l'exercice du bloc les reprend dans un projet plus large.

---

## II. Les bonnes pratiques

### La règle, en une phrase

Un secret ne vit **ni dans le code, ni dans l'image, ni dans les journaux**. Il est
injecté au démarrage par l'environnement d'exécution, et il est **rotatif** : conçu
pour être remplacé.

### Les variables d'environnement

Le minimum viable : la configuration sensible arrive par variable d'environnement,
fournie par le déploiement. Le code lit `os.environ[...]` et refuse de démarrer si la
variable manque — un défaut silencieux (une valeur de repli codée en dur) est une
faille déguisée en confort.

Les limites sont réelles : une variable d'environnement est lisible par les processus
enfants, visible dans certaines inspections système, et parfois recrachée par un
point de débogage mal placé. Elle suffit pour un petit projet ; elle ne suffit pas
quand les secrets sont nombreux, partagés, ou soumis à audit.

### Les coffres de secrets

Un **gestionnaire de secrets** (*secrets manager*) centralise la distribution :

- le secret est chiffré au repos dans un coffre, avec un contrôle d'accès par identité ;
- l'application le récupère au démarrage, ou le renouvelle automatiquement ;
- chaque accès est **tracé** : qui a lu quoi, quand ;
- les secrets peuvent être **dynamiques** : une base émet un compte à durée de vie
  courte pour une application, plutôt qu'un mot de passe éternel partagé.

### La CI/CD et les jetons

Les pipelines de livraison sont des lieux de fuite classiques : un jeton de
déploiement écrit en clair dans le fichier de pipeline, ou affiché dans les journaux
de construction. Trois réflexes :

1. stocker les jetons dans le **coffre de secrets de la plateforme de CI**, jamais
   dans le fichier de pipeline versionné ;
2. **masquer** les valeurs sensibles dans les journaux, et ne jamais les afficher
   dans un message de débogage ;
3. donner aux jetons la **portée la plus petite** possible — un jeton de déploiement
   qui peut aussi supprimer le dépôt est un risque, pas un gain de temps.

### Si un secret fuit

L'ordre des opérations est immuable :

1. **Révoquer** le secret — le rendre inutilisable, immédiatement.
2. **Remplacer** — émettre un nouveau secret, le déployer.
3. **Nettoyer** — retirer le secret du code, de l'historique, des images.
4. **Prévenir** — informer les personnes concernées, documenter l'incident.

Le nettoyage vient **après** la révocation, jamais avant : un secret retiré du code
mais encore valide est un secret toujours exploitable, quelque part dans un clone.

> [!TIP]
> Automatisez la détection : des outils de *secret scanning* (gitleaks, ou les
> analyses intégrées aux plateformes de code) passent en crochet avant chaque commit
> et en contrôle continu. Un secret détecté en local coûte une minute ; détecté en
> production, il coûte un incident.

---

## III. Rotation

### Pourquoi tourner

Un secret qui ne change jamais finit par traîner : dans un ancien portable, une
sauvegarde, une capture d'écran de documentation. La **rotation** — le remplacer
régulièrement — borne la durée d'exposition : même fuité, il ne vaut que jusqu'à la
prochaine rotation. Elle se déclenche sur un calendrier (par exemple à chaque
trimestre) et **à chaque suspicion** d'exposition.

### Changer sans interruption de service

Remplacer un secret utilisé par un service en production suit le même schéma que la
migration d'une clé de chiffrement : une **période de recouvrement**.

1. Émettre le nouveau secret, l'ajouter au coffre **à côté** de l'ancien.
2. Faire accepter les deux par le service (les deux valeurs sont valides).
3. Basculer progressivement les consommateurs sur le nouveau.
4. Vérifier que plus rien n'utilise l'ancien — les journaux du coffre le disent.
5. Révoquer l'ancien.

Le service ne s'arrête jamais, et l'étape de vérification évite le scénario classique
du secret révoqué trop tôt, utilisé par une tâche de nuit que personne n'avait en
tête.

> [!WARNING]
> La rotation sans traçage est un saut dans le noir : si vous ne savez pas **qui**
> utilise un secret, vous ne pouvez pas savoir quand il est sûr de le retirer. Le
> registre des secrets — quoi, où, qui, quand remplacé — précède la rotation.

---

### Exercice 1 — Auditer la gestion des secrets (55 min)

#### Contexte

**Lumen Analytics** édite un tableau de bord de consommation énergétique pour des
collectivités. Le dépôt Git de l'application vient d'être transféré vers une équipe
externe de maintenance, qui vous demande un audit avant de reprendre la main. Voici
l'arborescence et les extraits pertinents.

> Les valeurs sensibles sont **masquées** dans cet énoncé : à l'audit, ce sont les
> emplacements, les formats et les portées qui comptent — pas les caractères exacts.

```text
lumen-analytics/
├── .env
├── .gitignore
├── Dockerfile
├── README.md
├── app/
│   ├── config.py
│   └── main.py
├── scripts/
│   └── backup.sh
└── .github/
    └── workflows/
        └── deploy.yml
```

**`.env`** (versionné dans le dépôt) :

```bash
DATABASE_URL=postgres://admin:[mot de passe en clair]@db.lumen.internal:5432/analytics
OPENWEATHER_API_KEY=[clé d'API - 32 caractères hexadécimaux]
```

**`.gitignore`** :

```text
__pycache__/
*.pyc
.venv/
```

**`Dockerfile`** (extrait) :

```dockerfile
FROM python:3.12-slim
WORKDIR /srv
COPY . .
ENV STRIPE_SECRET_KEY=sk_live_[masqué]
RUN pip install -r requirements.txt
CMD ["python", "app/main.py"]
```

**`app/config.py`** (extrait) :

```python
import os

# Clé de session : valeur par défaut pour le développement local.
SECRET_KEY = os.environ.get("APP_SECRET_KEY", "lumen-dev-secret-2024")
```

**`.github/workflows/deploy.yml`** (extrait) :

```yaml
steps:
  - name: Deploy to production
    run: ./scripts/deploy.sh
    env:
      DEPLOY_TOKEN: ghp_[masqué]
```

**`scripts/backup.sh`** (extrait) :

```bash
#!/usr/bin/env bash
# Sauvegarde quotidienne de la base vers le stockage objet.
export AWS_ACCESS_KEY_ID=AKIA[masqué]
export AWS_SECRET_ACCESS_KEY=[clé secrète AWS - masquée]
aws s3 sync /var/backups s3://lumen-analytics-backups
```

**`README.md`** (extrait) :

```text
#### Démarrage rapide
mysql -u root -p[mot de passe en clair] lumen < schema.sql
```

Enfin, l'équipe vous apprend que le fichier `.env` est présent dans l'historique Git
depuis **huit mois**, et que le dépôt était public pendant les deux premiers mois du
projet.

#### Consignes

##### Partie 1 — Inventorier

Produisez un tableau qui recense **chaque secret** présent dans ces extraits : fichier,
motif ou valeur concernée, type de secret (clé de base de données, jeton de
déploiement, clé d'API tierce…), et ce qu'il donne accès. Visez l'exhaustivité : un
secret caché dans un commentaire ou un script compte autant que les autres.

##### Partie 2 — Évaluer

Pour chaque secret trouvé :

1. attribuez une gravité — **critique**, **majeur**, **mineur** — en tenant compte de
   la portée de l'accès et de l'exposition (dépôt public, historique, image
   distribuée) ;
2. décrivez en une phrase ce qu'un attaquant pourrait en faire concrètement ;
3. signalez les secrets qui, en plus, **ne sont pas révocables** tels quels (une donnée
   personnelle diffusée ne se « révoque » pas).

##### Partie 3 — Plan de remédiation

Rédigez un plan d'action **ordonné**, avec pour chaque étape son responsable type
(développeur, administrateur, sécurité) et son prérequis. L'ordre compte : justifiez
pourquoi la première étape ne peut pas être « supprimer le fichier `.env` ».

##### Partie 4 — Prévenir la récidive

Proposez les garde-fous à mettre en place pour que le prochain audit ne trouve rien :
configuration Git, contrôle automatique avant commit, contrôle en intégration
continue, stockage des secrets, politique de rotation. Pour chacun, indiquez où il se
place dans le flux de travail quotidien.

#### Attendu

Un dossier d'audit en quatre parties : l'inventaire, l'évaluation graduée, le plan
ordonné et les garde-fous — une production par consigne, dans l'ordre.

#### ✅ Auto-évaluation

Avant de passer au corrigé, vérifiez que :

- [ ] Mon inventaire recense tous les secrets des extraits, y compris ceux des scripts et du README.
- [ ] Chaque secret porte une gravité justifiée par sa portée **et** par son exposition.
- [ ] J'ai identifié au moins un secret dont la fuite ne se répare pas par une simple rotation.
- [ ] Mon plan commence par la révocation / le remplacement, pas par le nettoyage du code.
- [ ] Mes garde-fous couvrent le commit, l'intégration continue et le stockage des secrets.
- [ ] Je peux expliquer pourquoi effacer le fichier de l'historique ne suffit pas.

#### Prolongement de l'exercice

*Facultatif.* Le dépôt contient l'historique complet, avec le `.env` depuis huit mois.
Étudiez la réécriture de l'historique (outils dédiés, migration des clones) : quels
sont ses risques sur un dépôt partagé, et quelles conditions doit-on réunir avant de
la lancer ? Proposez enfin la politique de rotation des secrets de ce projet :
fréquence, déclencheurs, et procédure de bascule sans interruption.

<details>
<summary>Voir la correction</summary>

#### Partie 1 — Inventorier

**Éléments attendus :** sept secrets, plus une cause racine qui n'en est pas un.

| N° | Fichier | Secret | Type | Ce qu'il donne accès |
| --- | --- | --- | --- | --- |
| 1 | `.env` | Mot de passe de `DATABASE_URL` | Identifiants de base de données | Lecture et écriture de la base `analytics`, avec le compte `admin` |
| 2 | `.env` | `OPENWEATHER_API_KEY` | Clé d'API tierce | Consommation du quota de l'API météo au nom de Lumen, facturation en conséquence |
| 3 | `Dockerfile` | `STRIPE_SECRET_KEY` (`sk_live_…`) | Clé de paiement | Opérations de paiement réelles sur le compte de la plateforme |
| 4 | `app/config.py` | `SECRET_KEY` de repli | Clé de session | Fabrication de cookies de session valides, donc usurpation de compte |
| 5 | `.github/workflows/deploy.yml` | `DEPLOY_TOKEN` (`ghp_…`) | Jeton de déploiement | Publication de code en production, à la place de l'équipe |
| 6 | `scripts/backup.sh` | `AWS_ACCESS_KEY_ID` et `AWS_SECRET_ACCESS_KEY` | Identifiants de stockage objet | Lecture, modification et **suppression** des sauvegardes |
| 7 | `README.md` | Mot de passe `root` de MySQL | Identifiants d'administration | Contrôle total de la base de démonstration — et de toute base où le même mot de passe a été réutilisé |

**Cause racine :** le `.gitignore` ne couvre pas `.env`. Ce n'est pas un secret, mais
c'est ce qui a permis aux autres de partir dans le dépôt — et de rester dans
l'historique depuis huit mois.

**Ce qu'il faut retenir :** l'exercice 4 est celui que l'on rate presque toujours. Une
valeur de repli codée en dur ressemble à un confort de développement ; en réalité,
elle est **publique** — et si l'application l'accepte en production, tout le monde
peut fabriquer un cookie de session valide.

#### Partie 2 — Évaluer

**Éléments attendus :**

| N° | Gravité | Ce qu'un attaquant en fait | Révocable ? |
| --- | --- | --- | --- |
| 1 | **Critique** | Exfiltre la base clients, la modifie, la chiffre pour rançonner. | Oui, en changeant le mot de passe — mais les données déjà copiées ne reviennent pas. |
| 2 | **Mineur** | Consomme le quota, fait facturer l'entreprise, perturbe le service. | Oui, rotation simple auprès du fournisseur. |
| 3 | **Critique** | Lance des remboursements ou des encaissements, lit les données de paiement. | Oui, mais chaque minute compte : c'est de l'argent réel. |
| 4 | **Critique** | Forge des sessions, se connecte à la place de n'importe quel utilisateur. | Oui — et **immédiatement**, car la valeur est publique depuis le premier commit. |
| 5 | **Critique** | Déploie une version modifiée de l'application en production. | Oui, révocation du jeton sur la plateforme. |
| 6 | **Critique** | Lit les sauvegardes, les supprime, ou les remplace par les siennes. | Oui, mais la suppression des sauvegardes est irréparable si elle a déjà eu lieu. |
| 7 | **Critique** | Contrôle la base, et toute autre base partageant ce mot de passe. | Oui, rotation — et vérification des réutilisations. |

**Ce qui ne se révoque pas :** les **données** déjà copiées. Une rotation rend le
secret inutilisable pour l'avenir ; elle ne retire rien de ce qui a été exfiltré. Si
la base contient des données personnelles — consommation énergétique rattachée à des
personnes, coordonnées de contact — la fuite engage une obligation de notification,
indépendante de toute correction technique. De même, l'image Docker contenant la clé
de paiement a été distribuée : la clé se remplace, les images déjà tirées, elles, ne
se rappellent pas.

**Erreurs fréquentes :** classer le secret de repli (`config.py`) en « mineur » parce
que « c'est juste du développement ». Un secret visible de tous est le plus exploitable
de la liste : il ne demande aucune intrusion, seulement de lire le dépôt.

#### Partie 3 — Plan de remédiation

**Éléments attendus :** l'ordre est dicté par une seule question — *combien de temps
le secret reste-t-il exploitable ?*

| Étape | Action | Responsable type | Prérequis |
| --- | --- | --- | --- |
| 1 | **Révoquer et remplacer** les secrets exposés, du plus critique au moins critique : clé de paiement, identifiants de stockage, jeton de déploiement, clé de session, mots de passe de base, clé d'API tierce. | Sécurité + administrateurs | Avoir la liste complète (partie 1) et les accès d'administration |
| 2 | **Vérifier l'usage** de chaque ancien secret dans les journaux : a-t-il été utilisé depuis l'exposition ? | Sécurité | Journaux d'accès des services concernés |
| 3 | **Retirer les secrets du code** : `.env` hors dépôt, `Dockerfile` sans `ENV` sensible, scripts qui lisent l'environnement, README sans mot de passe. | Développeurs | Étape 1 terminée (sinon le code pointe vers des secrets morts) |
| 4 | **Reconstruire et redistribuer** les images, en retirant les anciennes du registre. | Administrateurs | Étape 3 |
| 5 | **Traiter l'historique Git** et les clones (voir « Pour aller plus loin »). | Administrateurs | Coordination de toute l'équipe |
| 6 | **Notifier** les parties concernées si des données personnelles ou de paiement sont en jeu, et documenter l'incident. | Sécurité + direction | Étape 2 (savoir ce qui a été accédé) |

**Pourquoi « supprimer le `.env` » ne peut pas être la première étape :** le fichier
n'est que la **trace** du secret. Le secret lui-même est déjà copié — dans
l'historique, dans les clones, peut-être dans les mains de quelqu'un. Supprimer le
fichier donne un sentiment de propreté tout en laissant l'accès ouvert, et détruit
les indices dont on a besoin pour savoir ce qui a été touché. On révoque d'abord, on
nettoie ensuite.

#### Partie 4 — Prévenir la récidive

**Éléments attendus :** un garde-fou par étape du flux de travail quotidien.

| Moment du flux | Garde-fou | Mise en œuvre |
| --- | --- | --- |
| Écriture du code | Interdire les fichiers sensibles | `.gitignore` complété : `.env`, `*.env`, `secrets/`, `*.pem` |
| Avant chaque commit | Contrôle automatique des secrets | Crochet pré-commit exécutant un outil de détection (par exemple *gitleaks*) : le commit est refusé si un motif de secret est détecté |
| À chaque poussée | Contrôle en intégration continue | Même analyse sur le dépôt complet, y compris l'historique, en planification régulière |
| Exécution | Injection des secrets | Variables d'environnement fournies par le déploiement, ou lecture depuis un coffre de secrets ; **aucune valeur de repli** dans le code — l'application refuse de démarrer sans sa configuration |
| Construction des images | Aucun secret dans l'image | Pas d'`ENV` ni de `COPY` de fichier sensible ; analyse de l'image avant publication |
| Dans la durée | Registre et rotation | Lister chaque secret (quoi, où, qui, quand remplacé) et le remplacer à fréquence fixe et à chaque suspicion d'exposition |

**Ce qu'il faut retenir :** les garde-fous les plus efficaces sont ceux qui bloquent
**au moment de l'erreur**. Un contrôle en intégration continue arrive après le commit ;
un crochet local arrête le développeur pendant qu'il tape encore. Les deux se
complètent — le premier couvre l'oubli, le second couvre le contournement.

#### Prolongement de l'exercice

**Réécriture de l'historique.** Des outils dédiés (par exemple *git filter-repo*, ou
les utilitaires de nettoyage d'historique) permettent de retirer un fichier de toutes
les révisions. Les risques sont réels : l'historique change d'identifiant, chaque
collaborateur doit **recloner** ou réaligner son dépôt, et une réécriture mal conduite
perd du travail. Les conditions à réunir avant de lancer l'opération : sauvegarde
complète du dépôt, gel des contributions pendant l'opération, coordination de tous les
clones, et — surtout — **révocation préalable de tous les secrets concernés**. Un
historique nettoyé ne protège rien si le secret est encore valide.

**Politique de rotation proposée :** les secrets de paiement et d'infrastructure sont
remplacés à chaque trimestre et **immédiatement** à la moindre suspicion ; les clés
de session sont tournées automatiquement ; chaque rotation suit la procédure à deux
valeurs (ajout de la nouvelle, bascule des consommateurs, vérification, retrait de
l'ancienne) afin de ne jamais interrompre le service.

#### Points à retenir

- Un audit de secrets se fait **par emplacement** : code, configuration, scripts,
  documentation, images, fichiers de CI — pas seulement les fichiers qui s'appellent
  `.env`.
- La gravité se juge sur la **portée** de l'accès et sur l'**exposition** accumulée ;
  une valeur de repli publique est un secret critique.
- L'ordre de remédiation est immuable : **révoquer, remplacer, nettoyer, notifier**.
  Supprimer la trace ne ferme pas l'accès.
- Ce qui ne se révoque pas, ce sont les données déjà copiées : la notification fait
  partie de la réponse, pas de la communication.
- La prévention se place **au moment de l'erreur** : crochet avant commit, contrôle en
  intégration continue, injection au démarrage, rotation planifiée.

</details>

---

## IV. Aller plus loin

🔁 **Pour aller plus loin (facultatif)** — à lire si le créneau le permet, rien
d'obligatoire n'en dépend.

- **HashiCorp Vault** : le coffre de référence en environnement hétérogène. Il va
  au-delà du stockage : secrets dynamiques à durée de vie limitée, renouvellement
  automatique, politique d'accès fine, journal d'audit complet. Le coût est
  opérationnel — un Vault est une infrastructure à administrer, sauvegarder et
  surveiller.
- **Les services cloud (AWS KMS, Azure Key Vault, Google Secret Manager)** : ils
  combinent coffre de secrets et gestion des clés de chiffrement, avec des politiques
  d'accès rattachées aux identités des services. Pratique quand l'infrastructure est
  déjà chez un fournisseur — à condition d'accepter que la clé maître y soit gérée.
- **L'identité à la place du secret** : les approches modernes (par exemple SPIFFE et
  SPIRE) remplacent les secrets statiques par des **identités de charge de travail**
  à courte durée de vie, délivrées automatiquement. Un secret qui n'existe pas ne
  fuit pas — c'est la forme aboutie de la rotation.

---

## Points clés

- Un secret commité n'est plus un secret : l'historique le garde, les clones le
  dispersent, les robots le trouvent.
- Chiffrer avec une clé stockée à côté des données ne protège rien — la séparation
  fait la valeur du chiffrement.
- La règle opérationnelle : **ni dans le code, ni dans l'image, ni dans les journaux** ;
  injecté au démarrage, idéalement depuis un coffre tracé.
- Face à une fuite : **révoquer, remplacer, nettoyer, prévenir** — dans cet ordre.
- La rotation borne la durée d'exposition ; elle suppose un registre et une période de
  recouvrement pour ne jamais interrompre le service.

---

## 🧠 Vérifiez vos acquis

*Formatif, jamais noté. Répondez sans relire le chapitre, puis dépliez « Vérifier ses réponses ». Ce qui compte est de repérer où vous vacillez, pas de faire 5 sur 5.*

**1.** Une clé a été commitée il y a huit mois ; elle a été supprimée au commit suivant. Où se trouve-t-elle encore ?

- A. Nulle part : la suppression l'a effacée
- B. Dans l'historique du dépôt, et dans les clones, forks, miroirs, caches de CI et sauvegardes du serveur Git
- C. Dans l'historique uniquement, inaccessible sans droits d'administration
- D. Dans l'historique, mais les robots des plateformes ne parcourent que la version la plus récente

**2.** La clé d'API est écrite dans le `Dockerfile` de l'application. Quel est le problème, et quel est le remède ?

- A. Aucun : l'image n'est pas publiée sur un registre public, la clé ne sort donc pas de l'entreprise
- B. La clé est cuite dans l'image et distribuée avec ; elle doit être injectée au démarrage, jamais à la construction
- C. La clé est lisible dans les journaux de build ; il suffit de masquer les valeurs sensibles à l'affichage
- D. La clé doit devenir une variable de construction (`ARG`), ce qui la rend anonyme dans l'image

**3.** Le code lit `os.environ["CLE"]` et, si la variable manque, utilise une valeur de repli codée en dur pour ne pas bloquer le démarrage. Que dit le cours ?

- A. C'est une bonne pratique : le service reste disponible, et le repli ne sert qu'en développement
- B. C'est une faille déguisée en confort : le code doit refuser de démarrer si la variable manque
- C. C'est neutre, à condition de documenter la valeur de repli dans le README du projet
- D. C'est nécessaire pour les environnements de test, où aucun coffre de secrets n'est déployé

**4.** Un secret vient d'être découvert dans un dépôt public. Quel est l'ordre des opérations ?

- A. Nettoyer, remplacer, révoquer, prévenir
- B. Révoquer, remplacer, nettoyer, prévenir
- C. Prévenir, nettoyer, révoquer, remplacer
- D. Remplacer, prévenir, nettoyer, révoquer

**5.** *Question ouverte (2-3 lignes).* Le secret de connexion utilisé par un service en production doit tourner, sans interruption. Décrivez le schéma suivi, et expliquez à quoi sert l'avant-dernière étape.

---

<details>
<summary>Vérifier ses réponses</summary>

### Réponses

**1. B.** Un dépôt Git n'est pas un tiroir : l'historique garde tout, le dépôt se copie partout, et les outils indexent — une clé exposée est souvent exploitée en quelques minutes.
**Pourquoi :** la leçon de l'incident de 2023 — ni algorithme cassé ni injection sophistiquée, juste une chaîne de caractères laissée au mauvais endroit, sans durée de vie ni portée bornées.

**2. B.** La règle du chapitre : un secret ne vit **ni dans le code, ni dans l'image, ni dans les journaux**.
**Pourquoi :** l'image se diffuse, et avec elle le secret. Même logique pour les fichiers de CI/CD : les jetons vont dans le coffre de secrets de la plateforme, avec la portée la plus petite possible.

**3. B.** Le minimum viable est une variable d'environnement fournie par le déploiement, et un démarrage qui **refuse** si elle manque.
**Pourquoi :** une variable d'environnement reste lisible par les processus enfants et visible dans certaines inspections — suffisant pour un petit projet, insuffisant quand les secrets sont nombreux, partagés ou soumis à audit : d'où le coffre de secrets, qui trace chaque accès.

**4. B.** Révoquer (rendre inutilisable, immédiatement), remplacer, nettoyer, prévenir — le nettoyage vient **après** la révocation, jamais avant.
**Pourquoi :** un secret retiré du code mais encore valide reste exploitable, quelque part dans un clone.

**5.** Attendu : la **période de recouvrement** — émettre le nouveau secret et l'ajouter au coffre à côté de l'ancien ; faire accepter les deux valeurs par le service ; basculer progressivement les consommateurs ; vérifier que plus rien n'utilise l'ancien ; puis révoquer l'ancien.
**Pourquoi :** l'étape de vérification (les journaux du coffre le disent) évite le scénario classique du secret révoqué trop tôt, utilisé par une tâche de nuit que personne n'avait en tête — la rotation sans traçage est un saut dans le noir, le registre des secrets précède la rotation.

</details>

---

[← Chapitre précédent](09-donnees-au-repos.md) · [Sommaire](Readme.md) · [Chapitre suivant →](11-chiffrement-data.md)
