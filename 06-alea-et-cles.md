[← Chapitre précédent](05-erreurs-classiques.md) · [Sommaire](Readme.md) · [Chapitre suivant →](07-tls-en-pratique.md)

# L'aléa et la gestion des clés

> Les armées changent de code après chaque bataille. Non pas parce que l'ancien
> chiffrement était mauvais, mais parce qu'une clé qui a servi trop longtemps finit par
> fuiter — par capture, par indiscrétion, par simple usure des habitudes. La
> cryptographie moderne a gardé cette leçon : la qualité d'un système se juge autant à
> la façon dont il **génère** ses clés qu'à la façon dont il les **remplace**.

**Durée de lecture :** 45 min · **Prérequis :** [Du chiffre de César à AES](02-chiffrement-symetrique.md)

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

Une clé n'est sûre que si elle est **imprévisible**. Or un ordinateur est une machine
déterministe : un programme, avec les mêmes entrées, produit toujours les mêmes
sorties. Le hasard y est donc simulé, à partir d'une **graine** (*seed*) de départ qui
détermine toute la suite.

> 🧪 **Atelier rapide (5 min) — rappel**
>
> Relancez deux fois de suite ce fragment, puis comparez avec la version `secrets` :
>
> ```python
> import random
>
> random.seed(42) # graine fixée : la suite est reproductible
> print([random.randint(0, 99) for _ in range(5)])
> ```
>
> ```python
> import secrets
>
> print(secrets.token_bytes(16)) # 16 octets imprévisibles
> ```
>
> La première suite est **toujours la même** ; la seconde change à chaque exécution.
> Notez laquelle vous confieriez une clé de chiffrement.

Le module `random` est un générateur pseudo-aléatoire, ou PRNG (*Pseudo-Random Number
Generator*) : rapide, commode, reproductible — parfait pour un jeu, **impropre à la
cryptographie**. Quelqu'un qui devine la graine reconstitue toute la suite, donc toutes
les clés qui en sont issues.

Pour la cryptographie, on utilise un générateur pseudo-aléatoire cryptographiquement
sûr, ou CSPRNG (*Cryptographically Secure Pseudo-Random Number Generator*) : `secrets` en
Python, `os.urandom`, ou les appels système dédiés. Ces générateurs puisent leur
entropie dans l'activité imprévisible de la machine — horloge à haute résolution,
bruit matériel, interruptions — et sont conçus pour qu'une partie de leur sortie ne
permette pas de prédire la suite.

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

Un secret écrit dans le code source se retrouve dans l'historique du dépôt, dans les
copies locales, dans les sauvegardes du serveur de code. Même après suppression, il
reste dans l'historique. La règle est simple : **le code se partage, les clés non**. On
les injecte à l'exécution — variable d'environnement, fichier de configuration hors
dépôt, ou service de gestion de secrets.

> [!TIP]
> Le bon réflexe en cas de doute : demandez-vous « si ce dépôt devenait public
> demain, qu'est-ce qui serait compromis ? ». Si la réponse inclut une donnée réelle,
> un secret y traîne.

### Une clé, un usage

Mélanger les usages multiplie les dégâts en cas de fuite. La clé qui chiffre les
exports de données ne doit pas servir à signer des jetons ; celle qui authentifie un
service ne doit pas chiffrer les sauvegardes. Quand une clé tombe, on sait alors
exactement ce qui est touché — et ce qui ne l'est pas.

Pour les données au repos, on peut aussi **dériver** des clés spécifiques à partir d'une
clé maîtresse, avec une fonction de dérivation de clé, ou KDF (*Key Derivation Function*) :
chaque usage reçoit sa propre clé, sans qu'il faille en gérer des dizaines à la main.

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
lisibles. On versionne donc les clés — chaque message porte la référence de la clé qui
l'a produit — et on conserve l'ancienne clé en lecture seule le temps de la migration,
avant de la détruire.

1. générer la nouvelle clé ;
2. chiffrer les **nouveaux** messages avec elle ;
3. conserver l'ancienne pour **déchiffrer** l'existant ;
4. migrer les données au repos ;
5. détruire l'ancienne clé, et le constater.

### Révoquer, c'est décider vite

Une clé compromise se révoque **immédiatement** — avant même de comprendre comment la
fuite s'est produite. La révocation rend la clé inutilisable pour les opérations
futures ; elle ne protège pas ce qui a déjà été chiffré avec, d'où l'intérêt de la
rotation régulière : elle borne la fenêtre d'exposition.

> [!WARNING]
> Une clé dont on a perdu la trace est une clé compromise. L'inventaire — où sont mes
> clés, qui y a accès, quand elles ont tourné — est la première pièce d'une gestion des
> clés, avant tout choix d'algorithme.

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

- Le hasard se **génère** : `random` est prévisible, `secrets` (CSPRNG) est adapté à la
  cryptographie. Une graine faible = une clé devinable.
- Une clé se **tire** au hasard, se **stocke** hors du code, se **distribue** par un
  canal chiffré, et se **détruit** quand son usage s'arrête.
- **Une clé, un usage** : mélanger chiffrement, signature et authentification multiplie
  les dégâts d'une fuite.
- La **rotation** borne la fenêtre d'exposition ; la **révocation** est immédiate dès
  qu'une clé est suspecte.
- On **versionne** les clés pour que les anciens messages restent lisibles pendant la
  migration, puis on détruit l'ancienne clé.
- L'**inventaire** des clés précède tout choix d'algorithme : une clé perdue de vue est
  une clé compromise.

---

[← Chapitre précédent](05-erreurs-classiques.md) · [Sommaire](Readme.md) · [Chapitre suivant →](07-tls-en-pratique.md)
