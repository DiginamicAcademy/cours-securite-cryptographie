# TP intégrateur — Sécuriser un système de bout en bout

> **Nature :** TP intégrateur noté · **Durée :** 3 h 30 · **Notion :** mobilise les douze chapitres du cours, de la primitive à la réponse à incident

Ce TP est **noté** : il compte pour la totalité de la note pratique du cours (grille d'évaluation en fin de document). Comptez 3 h 30 de travail, à répartir entre les deux journées comme vous l'entendez : tous les supports sont remis d'un coup, avancez à votre rythme.

---

## Contexte

Le **Laboratoire Valmont** réalise des analyses médicales pour trois hôpitaux et un cabinet de médecine générale. Son système d'information se résume ainsi :

- un **portail patient** (site web hébergé sur Nginx) : chaque patient y télécharge ses comptes rendus et y dépose ses ordonnances ;
- une **base de données** `patients` (MySQL) : identité, coordonnées, numéro de sécurité sociale, résultats d'analyses ;
- des **sauvegardes** quotidiennes de la base, copiées vers un stockage objet (MinIO) ;
- un **export nocturne** envoyé au centre hospitalier partenaire : les comptes rendus validés de la journée, au format CSV (Comma-Separated Values) ;
- des **comptes rendus** validés par le médecin biologiste, que le partenaire doit pouvoir authentifier ;
- une **équipe** de deux développeurs et d'un administrateur système, qui déploient l'application par une chaîne d'intégration continue.

La direction vous demande de **sécuriser ce système de bout en bout** : les données au repos comme en transit, les clés comme les secrets, jusqu'à la réponse le jour où une clé fuit.

---

## Objectifs

À la fin de ce TP, vous savez :

- **Recenser** les besoins de sécurité d'un système réel et **choisir** la primitive qui les couvre.
- **Chiffrer** et **déchiffrer** des données au repos — un fichier et une colonne de base de données — et **migrer** des données existantes.
- **Transmettre** une clé de session à un partenaire distant sans jamais la livrer en clair.
- **Hacher**, **signer** et **vérifier** l'intégrité et l'origine de documents.
- **Recenser** les clés et les secrets du système, et **sortir** les secrets du code source.
- **Configurer** TLS (Transport Layer Security) sur le portail et **vérifier** que la configuration tient ses promesses.
- **Cartographier** le trajet d'une donnée et **justifier** ce qui ne doit pas être stocké.
- **Planifier** une rotation de clés sans interruption de service, face à une fuite.

---

## Étapes

### Étape 1 — Recenser les besoins et choisir les primitives

*Chapitres 01 et 05.*

Dans `01-besoins.md`, recensez les six éléments à protéger du contexte (portail, base, sauvegardes, export partenaire, comptes rendus, secrets de l'équipe). Pour chacun, indiquez la garantie recherchée — confidentialité, intégrité, authentification, non-répudiation — et la primitive retenue : hachage, chiffrement symétrique, chiffrement asymétrique ou signature. Justifiez en une phrase les cas où deux primitives se combinent.

### Étape 2 — Chiffrer les données au repos

*Chapitres 02 et 09.*

Dans `02-au-repos/` :

1. `chiffrer_fichier.py` et `dechiffrer_fichier.py` : chiffrer l'export CSV de la veille avec AES-GCM (Advanced Encryption Standard en mode Galois/Counter Mode) — clé de 32 octets tirée par un CSPRNG (Cryptographically Secure Pseudo-Random Number Generator), nonce de 12 octets unique, préfixé au chiffré. Vérifiez qu'un octet altéré fait échouer le déchiffrement au lieu de produire un fichier corrompu.
2. `migrer_colonne.py` : rendre le numéro de sécurité sociale illisible dans la table `patients` — chiffrement au niveau applicatif, migration des lignes existantes, colonne d'index pour retrouver une fiche. Après migration, aucune requête ne doit montrer de numéro lisible.
3. La clé de chiffrement vient d'une variable d'environnement : elle ne figure ni dans le code, ni dans la base.

### Étape 3 — Transmettre une clé sans la livrer

*Chapitre 03.*

Le partenaire est distant : la clé de session de l'export nocturne ne peut pas traverser le réseau en clair. Dans `03-echange-de-cle/`, implémentez l'échange hybride — la clé de session est chiffrée avec la clé publique du partenaire, les données avec la clé de session. Livrez `echanger_cle.py` et, en trois phrases, ce que l'asymétrique résout là où le symétrique seul échoue.

### Étape 4 — Hacher et signer les comptes rendus

*Chapitre 04.*

Dans `04-signature/` : chaque compte rendu sortant porte son empreinte SHA-256 (Secure Hash Algorithm 2) et la signature du médecin biologiste ; le partenaire vérifie les deux avec la clé publique du signataire. Côté portail, les mots de passe des comptes patients sont conservés sous forme d'empreinte salée, calculée par une fonction lente. Livrez `signer.py`, `verifier.py`, et la preuve qu'un document modifié ne passe pas la vérification.

### Étape 5 — Recenser les clés et les secrets

*Chapitres 06 et 10.*

Dans `05-cles-et-secrets/` : la **carte des clés** du laboratoire (un tableau : clé, usage, propriétaire, emplacement, périodicité de rotation) et le plan de sortie des secrets du code — variables d'environnement, coffre de secrets, jetons de la chaîne d'intégration continue — avec les deux garde-fous à mettre en place avant le prochain déploiement.

### Étape 6 — Sécuriser le transit : TLS et certificats

*Chapitres 07 et 08.*

Dans `06-tls/`, produisez la configuration du serveur du portail, conforme à la politique interne du laboratoire :

- versions TLS 1.2 et 1.3 uniquement ;
- suites de chiffrement AEAD (Authenticated Encryption with Associated Data) uniquement ;
- en-tête HSTS (HTTP Strict Transport Security) avec `includeSubDomains` ;
- redirection de toute requête HTTP (Hypertext Transfer Protocol) vers HTTPS (Hypertext Transfer Protocol Secure) ;
- certificat Let's Encrypt, renouvellement automatisé.

Joignez les commandes de vérification — côté serveur (`openssl`) et côté client (`curl`) — avec le résultat attendu pour chacune, et dites en deux phrases ce que le client vérifie dans la chaîne de confiance.

### Étape 7 — Cartographier le trajet de la donnée

*Chapitre 11.*

Dans `07-flux/` : le trajet complet d'un résultat d'analyse (base → environnement de préparation → entrepôt → tableaux de bord), la mesure de protection retenue pour chaque tronçon, et ce qui ne doit **pas** être stocké ni exporté — minimisation au titre du RGPD (Règlement général sur la protection des données). Justifiez chaque choix par un risque, pas par une habitude.

### Étape 8 — Répondre à l'incident : fuite de clé

*Chapitres 06 et 12.*

**Scénario final** : la clé de l'export nocturne vient d'être retrouvée dans un dépôt public. Dans `08-incident/`, produisez le plan qui suit la crise de bout en bout :

1. la rotation sans interruption — période à deux clés, ré-chiffrement progressif, consommateurs couverts (application, traitement de nuit, partenaire, entrepôt) ;
2. la preuve de complétude : comment savoir que plus rien n'utilise l'ancienne clé ;
3. le repli : ce qui se passe si la rotation échoue en cours de route ;
4. une note de communication de quelques lignes, destinée aux équipes ;
5. une mesure du coût du chiffrement (temps de lecture avec et sans chiffrement) pour situer le compromis performance / protection.

---

## Contrôle transversal — les erreurs classiques

*Chapitre 05.*

Avant de rendre le dossier, relisez votre propre chaîne et pointez **trois** points de vigilance dans `RAPPORT.md` : la ligne de code concernée, le risque, la correction. Visez les erreurs classiques du cours — nonce réutilisé, mode obsolète, comparaison qui fuite par chronométrage, secret dans le code, message d'erreur trop bavard.

---

## Livrable final unique

Le livrable est **le dossier `valmont-securite/` complet**, remis en une seule fois (tel quel, ou compressé en une archive) :

```text
valmont-securite/
├── RAPPORT.md              note de synthèse (3 pages maximum) : sommaire du dossier,
│                           contrôle des erreurs classiques, compromis assumés
├── 01-besoins.md           étape 1
├── 02-au-repos/            étape 2 — scripts et preuves
├── 03-echange-de-cle/      étape 3 — script et enveloppe de clé
├── 04-signature/           étape 4 — signature et vérification
├── 05-cles-et-secrets/     étape 5 — carte des clés et plan de secrets
├── 06-tls/                 étape 6 — configuration et vérifications
├── 07-flux/                étape 7 — trajet de la donnée et minimisation
└── 08-incident/            étape 8 — rotation, repli, communication
```

Tout ce qui est scripté doit être **exécutable de bout en bout** sur une machine de cours, sans autre dépendance que Python et la bibliothèque `cryptography`.

---

## ✅ Auto-évaluation

Avant de rendre le dossier, vérifiez que :

- [ ] Chaque besoin du contexte est relié à une garantie et à une primitive, en une phrase justificative.
- [ ] Mes scripts s'exécutent de bout en bout, et une altération d'un seul octet est bien refusée.
- [ ] Après migration, aucune requête sur la base ne montre de numéro de sécurité sociale lisible.
- [ ] La clé de session n'a jamais transité en clair, et ma clé de chiffrement ne figure nulle part dans le code.
- [ ] Un compte rendu modifié ne passe pas la vérification de signature.
- [ ] Ma configuration TLS respecte les cinq points de la politique interne, chacun prouvé par une commande.
- [ ] Ma carte des clés nomme un propriétaire et un emplacement pour chaque clé.
- [ ] Mon plan de rotation tient le service ouvert pendant toute l'opération, et prévoit le repli.
- [ ] J'ai relevé trois points de vigilance sur mon propre code, avec leur correction.
