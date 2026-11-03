[← Chapitre précédent](09-donnees-au-repos.md) · [Sommaire](Readme.md) · [Chapitre suivant →](11-chiffrement-data.md)

# Gestion des secrets

> En septembre 2023, les chercheurs de la société Wiz découvrent qu'un jeton d'accès
> à un stockage Azure traîne dans un dépôt GitHub public tenu par une équipe de
> recherche de Microsoft. Le jeton, trop puissant et daté de 2051, ouvre **38
> téraoctets** de données privées : des sauvegardes de postes de travail, des clés, des
> mots de passe, plus de 30 000 messages internes. Le jeton était là depuis presque
> trois ans. Un fichier `.env` commité par erreur, c'est exactement la même chose — à
> l'échelle de votre projet.

**Durée de lecture :** 55 min · **Prérequis :** les notions de clé et de secret (journée
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

C'est la leçon de l'incident de 2023 : la faille n'était ni un algorithme cassé ni une
injection sophistiquée. C'était **une chaîne de caractères laissée au mauvais
endroit**, et un jeton dont personne n'avait réduit la durée de vie ni la portée.

### La clé à côté des données

La journée tourne autour d'un constat plus ancien. En 2011, la PlayStation Network est
compromise : 77 millions de comptes concernés. Les mots de passe étaient chiffrés —
mais, selon les analyses publiées après l'incident, la clé de chiffrement était
stockée sur le même périmètre que les données protégées. L'attaquant qui prend l'un
prend l'autre.

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
> Ouvrez le fichier `lab/docker-compose.yml` de l'environnement de démonstration et
> cherchez `APP_SECRET_KEY`. Répondez, en notant vos réponses :
>
> 1. Où la valeur est-elle écrite, et qui peut la lire ?
> 2. Que se passe-t-il si cette valeur est celle du dépôt de production ?
> 3. Regardez ensuite comment l'application la lit (`lab/secure/app.py`) : que
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

## IV. Aller plus loin

*Ces approfondissements sont facultatifs : ils ne sont jamais exigés pour terminer la
journée.*

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

[← Chapitre précédent](09-donnees-au-repos.md) · [Sommaire](Readme.md) · [Chapitre suivant →](11-chiffrement-data.md)
