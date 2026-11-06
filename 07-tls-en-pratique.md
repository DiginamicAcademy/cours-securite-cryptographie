[← Chapitre précédent](06-alea-et-cles.md) · [Sommaire](Readme.md) · [Chapitre suivant →](08-pki-et-certificats.md)

# TLS en pratique

> Avant HTTPS, le Wi-Fi du café laissait lire toutes vos requêtes en clair : le mot
> de passe tapé sur la boutique en ligne, la fiche produit consultée, l'adresse de
> livraison. **TLS** est le sceau de cire moderne : il chiffre le canal **et**
> authentifie le site avec qui vous parlez. Sans lui, le cadenas affiché par le
> navigateur n'est qu'un dessin.

**Durée de lecture :** 55 min hors approfondissements facultatifs · **Prérequis :** les notions de cryptographie de la
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

1. **Se présenter.** Le client envoie ses préférences — versions de TLS acceptées,
   algorithmes qu'il connaît, un nombre aléatoire — et le serveur répond par son choix
   et son propre nombre aléatoire. Ces deux nombres serviront à fabriquer la clé de
   session : personne d'autre ne peut la calculer.
2. **Vérifier les certificats.** Le serveur présente son certificat, qui lie son nom de
   domaine à une clé publique sous la signature d'une **autorité de certification**
   (AC), et le navigateur remonte la chaîne jusqu'à une racine qu'il fait confiance par
   défaut. Chaîne rompue, nom de domaine qui ne correspond pas, certificat expiré : il
   affiche un avertissement. Détail en partie II.
3. **Échanger une clé.** Client et serveur calculent alors la même **clé de session**,
   symétrique : elle chiffre tout le reste de la conversation, bien plus vite qu'un
   chiffrement asymétrique.

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

Un certificat X.509 — le format standard des certificats TLS — lie un nom de domaine
à une clé publique : il porte une date d'expiration et la signature d'une **autorité
de certification** (AC, *certificate authority*). Le navigateur détient une liste
d'autorités racines de confiance et valide chaque certificat en remontant de signature
en signature jusqu'à l'une d'entre elles.

Le chapitre [PKI et certificats](08-pki-et-certificats.md) détaille cette chaîne, ses
fragilités et le renouvellement automatisé — **Let's Encrypt** (2015), dont les
certificats gratuits valent 90 jours et se renouvellent par script via le protocole
**ACME** (*Automated Certificate Management Environment*). Retenez dès ici la seule
conséquence pratique : la durée courte est volontaire, elle **force
l'automatisation**, et un site moderne ne devrait plus jamais afficher « certificat
expiré ».

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
domaine en clair : il réécrit `http://` en `https://` avant même d'envoyer la requête.

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
avertissement plein écran — et la plupart font demi-tour. La cause est rarement
cryptographique, presque toujours opérationnelle : **personne n'a automatisé le
renouvellement**. Deux pannes massives et documentées, en fin de chapitre.

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
pour la session : chiffrement de masse, échange de clé, signature, hachage. Le réflexe
tient en une ligne : en 2024, une configuration raisonnable se limite à TLS 1.2 et 1.3
avec des suites **AEAD** (AES-GCM, ChaCha20-Poly1305) et des clés d'au moins 128 bits
de sécurité. Tout ce qui n'est pas AEAD, ou qui traîne d'un autre âge, se refuse sans
discuter — le catalogue des familles cassées est en fin de chapitre.

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

🔁 **Pour aller plus loin (facultatif)** — à lire si le créneau le permet, rien
d'obligatoire n'en dépend.

### Deux pannes de certificats (historique)

L'incident du certificat expiré n'a rien de théorique. En décembre 2018, un certificat
logiciel expiré chez l'équipementier Ericsson a privé de réseau des millions d'abonnés
du britannique O2 pendant une journée ; en février 2020, un certificat
d'authentification expiré a rendu Microsoft Teams indisponible pendant plusieurs
heures. Dans les deux cas, la cause est bureaucratique plus que cryptographique :
personne n'avait automatisé le renouvellement.

### Les familles de suites à écarter (historique des attaques)

- **RC4** et **DES/3DES** : cassés ou trop courts — l'attaque SWEET32 (2016) exploite
  la taille de bloc de 3DES ;
- les suites d'**export** et à clés faibles, vestiges des années 1990 ;
- les modes **CBC** mal implémentés, source de fuites par padding (Lucky Thirteen).

Ces noms reviennent dans les rapports d'audit : les connaître aide à lire une
configuration, mais le réflexe à appliquer ne demande pas de les retenir — une suite
qui n'est pas AEAD se refuse.

### Le préchargement HSTS

Un domaine peut être inscrit sur une liste de préchargement (*preload*) intégrée aux
navigateurs : il est alors protégé dès la toute première visite, avant même d'avoir
renvoyé un en-tête.

### Trois prolongements techniques

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

---

## 🧠 Vérifiez vos acquis

*Formatif, jamais noté. Répondez sans relire le chapitre, puis cochez. Les réponses sont juste après le séparateur : ce qui compte est de repérer où vous vacillez, pas de faire 5 sur 5.*

**1.** Un site chiffre parfaitement le canal, mais son certificat n'est pas valide pour son domaine. Que dit le cours sur cette situation ?

- A. Rien de grave : la confidentialité est assurée
- B. La première garantie sans la seconde ne vaut rien : on chiffre vers l'imposteur pendant qu'on vous vole le secret
- C. Le navigateur bascule automatiquement en HTTP pour continuer
- D. Le certificat n'intervient qu'au renouvellement, pas à la connexion

- [ ] ✓ je savais

**2.** Au troisième temps de la poignée de main, que obtiennent le client et le serveur ?

- A. La clé privée du serveur
- B. Une clé de session symétrique, calculée de part et d'autre, qui chiffre tout le reste de la conversation
- C. La liste complète des suites de chiffrement acceptées
- D. L'empreinte du certificat racine

- [ ] ✓ je savais

**3.** À quoi sert l'en-tête `Strict-Transport-Security` (HSTS) ?

- A. À interdire le retour en clair : pendant `max-age`, le navigateur réécrit `http://` en `https://` avant même d'envoyer la requête
- B. À bloquer le chargement des scripts externes
- C. À renouveler automatiquement le certificat avant son expiration
- D. À forcer le passage à TLS 1.3 sur les clients anciens

- [ ] ✓ je savais

**4.** Que garantit exactement le cadenas affiché par le navigateur ?

- A. Que le site est honnête et qu'on peut lui confier ses données
- B. Que le canal est chiffré et que le certificat est valide pour ce domaine
- C. Que l'éditeur du site est l'exploitant réel du serveur
- D. Que les données stockées sur le serveur sont chiffrées

- [ ] ✓ je savais

**5.** *Question ouverte (3 lignes).* Un audit de la boutique en ligne relève trois lignes : « TLS 1.0 accepté pour compatibilité », « certificat auto-signé », « HSTS absent ». Pour chacune, dites ce qui casse et la correction.

- [ ] ✓ je savais

---

### Réponses

**1. B.** Les deux garanties de TLS sont indissociables : confidentialité du canal **et** authentification du serveur.
**Pourquoi :** chiffrer un canal vers l'imposteur qui se fait passer pour votre banque, c'est protéger le secret pendant qu'on vous le vole.

**2. B.** Client et serveur calculent la même clé de session, symétrique.
**Pourquoi :** elle chiffre tout le reste de la conversation, bien plus vite qu'un chiffrement asymétrique — l'asymétrique sert à se rencontrer, le symétrique à discuter.

**3. A.** C'est la réponse à l'attaque du *downgrade*, qui force la version en clair avant même que TLS ne démarre.
**Pourquoi :** attention à l'ordre posé par le cours — HSTS se déclare **après** avoir vérifié que tout fonctionne en HTTPS, y compris les sous-domaines, sinon on verrouille les utilisateurs hors du site.

**4. B.** Le cadenas dit « canal chiffré et certificat valide pour ce domaine », rien de plus.
**Pourquoi :** un site d'hameçonnage peut obtenir un certificat valide pour son propre domaine — il chiffre très bien ce qu'il vous vole.

**5.** Attendu : **TLS 1.0 accepté** → version dépréciée et attaquable, les attaquants choisissent toujours le chemin le plus faible ; on limite à TLS 1.2 et 1.3 avec des suites AEAD (AES-GCM, ChaCha20-Poly1305). **Certificat auto-signé** → il chiffre mais n'authentifie rien, n'importe qui peut en fabriquer un ; on passe par une autorité de certification (Let's Encrypt convient). **HSTS absent** → rien n'empêche le downgrade au premier aller-retour ; on ajoute l'en-tête une fois le HTTPS maîtrisé partout.
**Pourquoi :** une configuration TLS se juge à ses maillons les plus faibles, et les erreurs les plus coûteuses sont opérationnelles, pas mathématiques.

---

[← Chapitre précédent](06-alea-et-cles.md) · [Sommaire](Readme.md) · [Chapitre suivant →](08-pki-et-certificats.md)
