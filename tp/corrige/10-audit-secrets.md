# Corrigé — Auditer la gestion des secrets

> Corrigé de [l'exercice « Auditer la gestion des secrets »](../10-audit-secrets.md).
> Comparez d'abord votre travail aux **étapes** ci-dessous : la méthode compte
> autant que le résultat.

---

## Partie 1 — Inventorier

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

---

## Partie 2 — Évaluer

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

---

## Partie 3 — Plan de remédiation

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

---

## Partie 4 — Prévenir la récidive

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

---

## Pour aller plus loin

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

---

## Points à retenir

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
