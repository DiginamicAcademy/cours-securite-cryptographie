# TLS en pratique

> Avant HTTPS, le Wi-Fi du café laissait lire toutes vos requêtes en clair : le mot
> de passe tapé sur la boutique en ligne, la fiche produit consultée, l'adresse de
> livraison. **TLS** est le sceau de cire moderne : il chiffre le canal **et**
> authentifie le site avec qui vous parlez. Sans lui, le cadenas affiché par le
> navigateur n'est qu'un dessin.

**Durée de lecture :** 75 min · **Prérequis :** les notions de cryptographie de la
journée précédente (chiffrement symétrique et asymétrique, signature numérique)

---

## Objectifs

À l'issue de ce chapitre, vous serez capable de :

- **Décrire** les trois temps de la poignée TLS (*handshake*) et ce que chacun apporte.
- **Lire** les en-têtes de sécurité d'une réponse HTTPS et en tirer les conclusions.
- **Repérer** les faiblesses de configuration TLS les plus courantes : versions
  obsolètes, suites de chiffrement faibles, certificat expiré, HSTS absent.
- **Corriger** une configuration HTTPS faible en choisissant des versions et des
  suites de chiffrement encore sûres.

---

## I. La poignée de main

### Ce que TLS protège

**TLS** (*Transport Layer Security*) est le protocole qui sécurise les échanges entre
un client — typiquement un navigateur — et un serveur. Quand vous voyez `https://`
dans la barre d'adresse, TLS travaille en coulisses. Il apporte deux garanties
complémentaires :

- la **confidentialité** du canal : ce qui circule est chiffré, un observateur ne
  lit que du bruit ;
- l'**authentification** du serveur : le site prouve son identité par un certificat
  signé par une autorité que votre navigateur reconnaît.

La première garantie sans la seconde ne vaut rien : chiffrer un canal vers
l'imposteur qui se fait passer pour votre banque, c'est protéger le secret pendant
qu'on vous le vole. Les deux vont ensemble, et c'est toute l'élégance du protocole.

### Les trois temps de la poignée

Avant d'échanger la moindre donnée utile, client et serveur doivent s'entendre sur
**comment** ils vont se parler, se **vérifier**, puis obtenir une **clé commune**.
C'est la poignée de main TLS (*handshake*), une danse en trois temps :

```text
Navigateur                                     Serveur
   |                                              |
   | 1. « Bonjour » : versions de TLS             |
   |    acceptées, suites de chiffrement          |
   |    proposées, nombre aléatoire               |
   | -------------------------------------------> |
   |                                              |
   | 2. « Voici mon identité » : choix de la       |
   |    suite, certificat du serveur              |
   | <------------------------------------------- |
   |     le navigateur vérifie la chaîne          |
   |     de certificats jusqu'à une racine        |
   |     qu'il fait confiance                     |
   |                                              |
   | 3. Échange de clé → clé de session           |
   | <=========================================> |
   |     tout le reste circule chiffré            |
```

1. **Se présenter.** Le client envoie ses préférences : les versions de TLS qu'il
   accepte, les algorithmes qu'il sait utiliser, et un nombre aléatoire. Le serveur
   répond par son choix et son propre nombre aléatoire. Ces deux nombres aléatoires
   serviront à fabriquer la clé de session — personne d'autre ne peut la calculer.
2. **Vérifier les certificats.** Le serveur présente son certificat, un document qui
   lie son nom de domaine à une clé publique, signé par une autorité de certification
   (AC). Le navigateur remonte la chaîne des signatures jusqu'à une autorité racine
   qu'il fait confiance par défaut. Si la chaîne est rompue, le nom de domaine ne
   correspond pas ou le certificat est expiré, il affiche un avertissement.
3. **Échanger une clé.** Grâce aux informations précédentes, client et serveur
   calculent la même **clé de session**. Cette clé est symétrique : elle servira à
   chiffrer tout le reste de la conversation, bien plus rapide qu'un chiffrement
   asymétrique.

> [!TIP]
> Retenez l'image du sceau de cire : le certificat **prouve à qui vous parlez**, la
> clé de session **scelle le contenu**. Un HTTPS sans certificat valide, c'est une
> lettre scellée adressée à n'importe qui.

> 🧪 **Atelier rapide (8 min)**
>
> Observez les en-têtes de sécurité d'un service réel, en une seule requête de
> lecture :
>
> ```bash
> curl -I https://tutor.diginamic.fr/docs
> ```
>
> Repérez les en-têtes `strict-transport-security`, `content-security-policy`,
> `x-content-type-options`. Leur forme générale est la suivante (les valeurs varient
> d'un serveur à l'autre) :
>
> ```http
> HTTP/2 200
> strict-transport-security: max-age=31536000
> x-content-type-options: nosniff
> ```
>
> Notez lesquels sont présents et lesquels manquent : vous allez en reparler dans la
> partie suivante. Restez à une ou deux requêtes de lecture — c'est un service de
> production, on n'y teste ni la charge ni les failles.

---

## II. Certificats et HSTS

### La chaîne de confiance, en bref

Un certificat X.509 — le format standard des certificats TLS — est un document qui
contient notamment le nom de domaine couvert, la clé publique du serveur, une date
d'expiration, et la signature d'une **autorité de certification** (AC, *certificate
authority*). Votre navigateur détient une liste d'autorités racines de confiance ;
chaque certificat se valide en remontant de signature en signature jusqu'à l'une
d'entre elles. Le chapitre suivant détaille cette chaîne et ses fragilités.

### Let's Encrypt : des certificats gratuits et renouvelés tout seuls

Jusqu'en 2015, un certificat se demandait à une autorité commerciale, se payait, et
se renouvelait à la main tous les un à deux ans — d'où les certificats expirés que
tout le monde a déjà croisés. **Let's Encrypt**, autorité lancée en 2015 par l'ISRG
(*Internet Security Research Group*), délivre gratuitement des certificats valables
90 jours, via le protocole **ACME** (*Automated Certificate Management Environment*).

La durée courte est volontaire : elle **force l'automatisation**. Un certificat
renouvelé par un script ne devient jamais obsolète sans que personne ne s'en aperçoive.
Des outils comme *certbot* ou le module ACME de votre serveur web gèrent la demande,
le renouvellement et le rechargement du service. En pratique, un site moderne ne
devrait plus jamais afficher « certificat expiré ».

### HSTS : interdire le retour en clair

Le chiffrement du canal ne suffit pas si le navigateur accepte, une fois, de
retourner en `http://`. C'est exactement l'attaque dite du *downgrade* : un
intercepteur force la version en clair de la connexion, avant même que TLS ne
démarre. La réponse s'appelle **HSTS** (*HTTP Strict Transport Security*), un
en-tête par lequel le serveur déclare :

```http
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

Dès lors, pendant `max-age` secondes (ici un an), le navigateur refuse d'ouvrir ce
domaine en clair — il réécrit `http://` en `https://` avant même d'envoyer la
requête. Le domaine peut même être inscrit sur une liste de préchargement
(*preload*) intégrée aux navigateurs, pour être protégé dès la toute première visite.

> [!WARNING]
> HSTS se déclare **après** avoir vérifié que tout fonctionne en HTTPS, y compris
> les sous-domaines. Un `max-age` long sur un domaine dont un sous-domaine ne sait
> pas faire de HTTPS verrouille les utilisateurs hors du site — et le déverrouillage
> est lent, car il dépend de chaque navigateur.

---

## III. Erreurs courantes

Une configuration TLS se juge à ses maillons les plus faibles. Voici les erreurs que
l'on rencontre le plus souvent, du plus grave au plus anecdotique.

### Le certificat expiré

Un certificat a une date de fin. Passée celle-ci, chaque visiteur reçoit un
avertissement plein écran — et la plupart font demi-tour. L'incident n'a rien de
théorique : en décembre 2018, un certificat logiciel expiré chez l'équipementier
Ericsson a privé de réseau des millions d'abonnés du britannique O2 pendant une
journée ; en février 2020, un certificat authentification expiré a rendu Microsoft
Teams indisponible pendant plusieurs heures. Dans les deux cas, la cause est
bureaucratique plus que cryptographique : **personne n'avait automatisé le
renouvellement**.

### Les versions obsolètes

| Version | Statut | Pourquoi l'éviter |
| --- | --- | --- |
| SSL 3.0 | Abandonnée | Cassée par POODLE (2014) |
| TLS 1.0 / 1.1 | Dépréciées par la RFC 8996 (mars 2021) | Attaquables (BEAST, 2011), plus maintenues |
| TLS 1.2 | Toujours sûre si bien configurée | Compatible avec l'ancien matériel |
| TLS 1.3 | Recommandée | Plus rapide, algorithmes modernes uniquement |

Accepter TLS 1.0 « pour les vieux clients » revient à laisser une porte de service
ouverte : les attaquants choisissent toujours le chemin le plus faible.

### Les suites de chiffrement faibles

Une **suite de chiffrement** (*cipher suite*) est l'ensemble des algorithmes utilisés
pour la session : chiffrement de masse, échange de clé, signature, hachage. Quelques
familles à écarter sans hésiter :

- **RC4** et **DES/3DES** : cassés ou trop courts (l'attaque SWEET32, 2016, exploite
  la taille de bloc de 3DES) ;
- les suites d'**export** et à clés faibles, vestiges des années 1990 ;
- les modes **CBC** mal implémentés, source de fuites par padding (Lucky Thirteen).

En 2024, une configuration raisonnable se limite à TLS 1.2 et 1.3, avec des suites
AEAD (AES-GCM, ChaCha20-Poly1305) et des clés d'au moins 128 bits de sécurité.

### Les autres pièges fréquents

- **Le certificat auto-signé** en production : il chiffre, mais n'authentifie rien —
  n'importe qui peut en fabriquer un.
- **Le nom de domaine qui ne correspond pas** au certificat (`boutique.exemple.fr`
  servi par un certificat `www.exemple.fr`).
- **Le contenu mixte** : une page HTTPS qui charge une image ou un script en `http://`
  casse la garantie, et les navigateurs modernes bloquent ces ressources.
- **HSTS absent** : le site est en HTTPS, mais rien n'empêche un downgrade au premier
  aller-retour.

> [!DANGER]
> Le cadenas vert ne dit pas « ce site est honnête ». Il dit : « le canal est
> chiffré et le certificat est valide pour ce domaine ». Un site d'hameçonnage peut
> parfaitement obtenir un certificat valide pour son propre domaine — il chiffre
> très bien ce qu'il vous vole.

---

## IV. Aller plus loin

*Ces approfondissements sont facultatifs : ils ne sont jamais exigés pour terminer la
journée.*

- **TLS 1.3** (RFC 8446, août 2018) : la poignée de main passe à un aller-retour
  (*1-RTT*), les algorithmes obsolètes disparaissent, et la reprise de session en
  *0-RTT* accélère les reconnexions — au prix d'une vulnérabilité au rejeu sur les
  premières requêtes, à réserver aux opérations idempotentes.
- **Encrypted Client Hello (ECH)** : dans TLS 1.3, le nom du site demandé (le champ
  *SNI*) circule encore en clair dans la poignée de main — un observateur sait quel
  site vous visitez, même sans lire le contenu. ECH chiffre cette information, en
  s'appuyant sur une configuration DNS chiffrée.
- **L'épinglage de certificat** (*certificate pinning*) : lier une application à une
  clé précise plutôt qu'à une autorité. Puissant, mais dangereux — un certificat
  perdu ou remplacé casse l'application pour tous les utilisateurs. Les navigateurs
  ont abandonné le mécanisme ; il survit dans quelques applications mobiles.

---

## Points clés

- TLS chiffre le canal **et** authentifie le serveur : les deux garanties sont
  indissociables.
- La poignée de main se déroule en trois temps : **se présenter**, **vérifier les
  certificats**, **échanger une clé de session**.
- Un certificat se valide en remontant une **chaîne de signatures** jusqu'à une
  autorité racine de confiance ; Let's Encrypt rend ce renouvellement automatique.
- **HSTS** interdit le retour en clair et protège contre les attaques de *downgrade*.
- Les erreurs les plus coûteuses ne sont pas mathématiques mais opérationnelles :
  certificat non renouvelé, versions obsolètes laissées ouvertes, contenu mixte.
