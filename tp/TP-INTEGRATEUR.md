# TP intégrateur — Sécuriser un système de bout en bout

> **Nature :** TP intégrateur **facultatif**, noté au bulletin s'il est rendu · **Durée :** 3 h 30 · **Notion :** mobilise les douze chapitres du cours, de la primitive à la réponse à incident

Ce TP est **facultatif** ; si vous le rendez, il est **noté** et compte pour la totalité de la note pratique du cours. Comptez 3 h 30 de travail, à répartir entre les deux journées comme vous l'entendez : tous les supports sont remis d'un coup, avancez à votre rythme.

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

## Ce qui est fourni

Aucun système existant ne vous est livré : tout est à écrire. Deux éléments de départ
sont toutefois fournis ci-dessous, pour que les étapes 2 et 4 partent d'un état de base
réel et non d'une table vide. **Toutes ces données sont fictives**, fabriquées pour
l'exercice : toute ressemblance avec une personne existante serait fortuite.

### La base `patients`, dans son état de départ

C'est l'état **avant** votre travail : le numéro de sécurité sociale y est encore en
clair, et c'est précisément ce que l'étape 2 doit corriger.

```sql
-- Échantillon de travail. Une ligne = une fiche patient avec son dernier résultat validé.
CREATE DATABASE IF NOT EXISTS valmont CHARACTER SET utf8mb4;
USE valmont;

CREATE TABLE patients (
    id             INT PRIMARY KEY,
    nom            VARCHAR(64)  NOT NULL,
    prenom         VARCHAR(64)  NOT NULL,
    date_naissance DATE         NOT NULL,
    adresse        VARCHAR(128),
    telephone      VARCHAR(20),
    email          VARCHAR(128),
    nir            CHAR(15)     NOT NULL,      -- numéro de sécurité sociale, en clair
    analyse        VARCHAR(32)  NOT NULL,
    valeur         VARCHAR(16)  NOT NULL,
    unite          VARCHAR(16),
    rendu_le       DATETIME     NOT NULL,
    INDEX idx_nir (nir)
);

INSERT INTO patients
    (id, nom, prenom, date_naissance, adresse, telephone, email,
     nir, analyse, valeur, unite, rendu_le) VALUES
  (1,  'Marchand',  'Alicie',  '1985-07-14', '18 rue des Lilas, 59210 Coudekerque-Village', '03 28 00 11 22', 'a.marchand@exemple.fr',  '285075921504281', 'Glycémie à jeun',   '1.08',  'g/L',    '2026-11-05 08:12:00'),
  (2,  'Ferrand',   'Bruno',   '1972-03-02', '7 chemin des Ares, 34200 Sète',               '04 67 12 34 56', 'b.ferrand@exemple.fr',  '172033421805614', 'Créatinine',        '88',    'µmol/L', '2026-11-05 07:41:00'),
  (3,  'Bensaïd',   'Chloé',   '1990-11-27', '24 rue de Vaugirard, 75006 Paris',            '01 45 44 07 18', 'c.bensaid@exemple.fr',  '290117510603133', 'TSH',               '2.1',   'mUI/L',  '2026-11-04 16:05:00'),
  (4,  'Oliveira',  'Damien',  '1964-05-09', '61 boulevard Baille, 13005 Marseille',        '04 91 33 72 05', 'd.oliveira@exemple.fr', '164051309502867', 'Cholestérol LDL',   '1.22',  'g/L',    '2026-11-05 09:26:00'),
  (5,  'Vasseur',   'Élodie',  '1978-09-21', '9 rue de Sully, 38100 Grenoble',              '04 76 51 20 63', 'e.vasseur@exemple.fr',  '278093818504775', 'HbA1c',             '5.4',   '%',      '2026-11-04 11:48:00'),
  (6,  'Benali',    'Farid',   '1953-01-30', '3 allée des Tilleuls, 91080 Évry-Courcouronnes', '01 60 87 41 92', 'f.benali@exemple.fr', '153019124401986', 'CRP',               '3',     'mg/L',   '2026-11-05 10:02:00'),
  (7,  'Poncet',    'Gaëlle',  '1996-06-11', '142 rue Paul Bert, 69003 Lyon',               '04 78 62 15 30', 'g.poncet@exemple.fr',   '296066926607323', 'Hémoglobine',       '13.9',  'g/dL',   '2026-11-04 14:19:00'),
  (8,  'Schneider', 'Hugo',    '1947-12-05', '27 route du Polygone, 67100 Strasbourg',      '03 88 30 76 14', 'h.schneider@exemple.fr','147126721405234', 'D-dimères',         '210',   'ng/mL',  '2026-11-05 06:55:00'),
  (9,  'Da Costa',  'Inès',    '2001-04-18', '53 avenue de Reynerie, 31400 Toulouse',       '05 61 47 88 21', 'i.dacosta@exemple.fr',  '201043115506190', 'Sodium',            '139',   'mmol/L', '2026-11-05 12:33:00'),
  (10, 'Rivemale',  'Jules',   '1988-08-23', '12 rue des Martyrs, 07200 Aubenas',           '04 75 36 09 47', 'j.rivemale@exemple.fr', '188080714603487', 'Kalium',            '4.2',   'mmol/L', '2026-11-04 09:07:00'),
  (11, 'Delaunay',  'Karine',  '1969-02-14', '48 avenue de Paris, 78000 Versailles',        '01 39 20 65 08', 'k.delaunay@exemple.fr', '269027807908838', 'ALAT',              '26',    'U/L',    '2026-11-05 15:24:00'),
  (12, 'Fabre',     'Loïc',    '1993-10-31', '90 chemin de la Croix, 84000 Avignon',        '04 90 85 32 76', 'l.fabre@exemple.fr',    '193108410101537', 'Vitamine 25-OH-D',  '28',    'ng/mL',  '2026-11-04 17:52:00'),
  (13, 'Aubry',     'Marlène', '1959-07-08', '3 rue d''Aguesseau, 92100 Boulogne-Billancourt', '01 46 03 51 39', 'm.aubry@exemple.fr',  '259079232804677', 'Ferritine',         '145',   'µg/L',   '2026-11-05 11:16:00'),
  (14, 'Kessaï',    'Nadir',   '1981-03-19', '88 rue Saint-Maur, 75011 Paris',              '01 43 57 24 90', 'n.kessai@exemple.fr',   '181037534009703', 'Acide urique',      '52',    'mg/L',   '2026-11-05 13:47:00'),
  (15, 'Renaud',    'Olivia',  '1975-11-26', '21 rue de la Porte, 29200 Brest',             '02 98 44 61 05', 'o.renaud@exemple.fr',   '275112909602387', 'Bilirubine totale', '12',    'µmol/L', '2026-11-04 10:38:00');
```

Chargez cet échantillon dans votre instance MySQL (`mysql -u root -p valmont < patients.sql`),
ou reproduisez la même structure dans le serveur dont vous disposez.

### Les comptes du portail, dans leur état de départ

L'étape 4 porte sur la façon dont ces comptes sont stockés. Le format attendu y est
donné : votre fonction de hachage doit reproduire les empreintes ci-dessous.

```sql
CREATE TABLE comptes_portail (
    id          INT PRIMARY KEY,
    patient_id  INT         NOT NULL,   -- référence patients(id)
    identifiant VARCHAR(32) NOT NULL,
    sel         CHAR(32)    NOT NULL,   -- sel hexadécimal de 16 octets, unique par compte
    empreinte   CHAR(64)    NOT NULL,   -- PBKDF2-HMAC-SHA-256, 100 000 itérations
    UNIQUE KEY uq_identifiant (identifiant)
);

INSERT INTO comptes_portail (id, patient_id, identifiant, sel, empreinte) VALUES
  (1, 1,  'marchand.a',  '9f2c41b0d7e38a5c614f0b9d2e77a3c4', '452eacfe75a54c3d374ad9fa037eb8d59e13e3a7f9690e995df588b66d9c99a6'),
  (2, 2,  'ferrand.b',   '31b8ae04c9d25f7e0a6c18bd45f2e970', '56df327891eebd7b5d235a7ff12ed34f8eb5e755ecffa3a8fbf9b97b2b8e8b21'),
  (3, 3,  'bensaid.c',   'c47d9e12f308a5b6de9174c0825b3af1', 'be04bae8b804547c31689121719fe028221db6ca00913c61158eaed7124593b9'),
  (4, 4,  'oliveira.d',  '7a05c3e8b46192fd30e97ba1c46d8502', '6aed73eaea617cf35ab31ed7d5cf9a0689bf4df48f5f98e619694c7dd1f3ed58'),
  (5, 5,  'vasseur.e',   'e28f10b7d5a34c960b82f17ac35e6d94', '15da2ee67c048211b4af751eb0d60f07bd8e68c32fb06dc497f44347baffe9ad'),
  (6, 6,  'benali.f',    '0b6d94a1cf73e285d90a63b17f42c8e5', '8a4ccca287710f7a8cfef3fa91e6b014d737bce25bd85abab3f85c054650633c');
```

Les mots de passe ne sont pas fournis, à l'exception de celui du compte `marchand.a` :
`valmont-novembre`. Il vous sert à prouver que votre vérification fonctionne de bout en
bout — connexion acceptée avec ce mot de passe, refusée avec n'importe quel autre.

### L'export nocturne de la veille

C'est le fichier que l'étape 2 chiffre et que l'étape 3 protège jusqu'au partenaire.
Export du 5 novembre 2026, comptes rendus validés par le biologiste :

```csv
compte_rendu,nir,nom,prenom,analyse,valeur,unite,rendu_le,biologiste
CR-2026-1187,147126721405234,Schneider,Hugo,D-dimères,210,ng/mL,2026-11-05 06:55:00,Dr A. Kessler
CR-2026-1188,172033421805614,Ferrand,Bruno,Créatinine,88,µmol/L,2026-11-05 07:41:00,Dr A. Kessler
CR-2026-1189,285075921504281,Marchand,Alicie,Glycémie à jeun,1.08,g/L,2026-11-05 08:12:00,Dr A. Kessler
CR-2026-1190,164051309502867,Oliveira,Damien,Cholestérol LDL,1.22,g/L,2026-11-05 09:26:00,Dr A. Kessler
CR-2026-1191,153019124401986,Benali,Farid,CRP,3,mg/L,2026-11-05 10:02:00,Dr A. Kessler
CR-2026-1192,259079232804677,Aubry,Marlène,Ferritine,145,µg/L,2026-11-05 11:16:00,Dr A. Kessler
CR-2026-1193,201043115506190,Da Costa,Inès,Sodium,139,mmol/L,2026-11-05 12:33:00,Dr A. Kessler
```

### Ce que le sujet suppose comme environnement

Soyons honnêtes sur le matériel : deux étapes sortent du périmètre « un ordinateur et
Python ». L'**étape 2.2** suppose un serveur **MySQL** (ou MariaDB) joignable, et
l'**étape 6** suppose **Nginx**, `openssl` et `curl`. Sans ces services, simulez-les —
un conteneur MySQL, un certificat auto-signé — et écrivez dans votre rapport **ce que la
simulation ne prouve pas**. La liste complète des dépendances est en fin de sujet.

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

**Aucun prérequis** : le plan de rotation attendu ici se construit dans cette étape. L'exercice [Défi : planifier une rotation de clés](../12-compromis.md) est **facultatif** et peut inspirer ; il n'est pas demandé.

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

Tout ce qui est scripté doit être **exécutable de bout en bout** sur une machine de
cours. Les dépendances réelles sont les suivantes — le reste du sujet ne suppose que
Python et la bibliothèque `cryptography` :

- **dossier entier** : Python 3 et `cryptography` ;
- **étape 2, migration de la colonne** : un serveur **MySQL** (ou MariaDB) et le pilote
  Python associé (`pip install PyMySQL`). À défaut : un conteneur MySQL, ou une base
  **SQLite** locale qui joue le même rôle. Dans ce dernier cas, la migration doit rester
  lisible telle quelle et vous le signalez dans le rapport ;
- **étape 6, TLS** : **Nginx**, `openssl` et `curl`. Un certificat Let's Encrypt réel
  exige un nom de domaine public ; à défaut, un certificat auto-signé (ou `mkcert`)
  suffit pour la configuration et la poignée de main, mais **ne prouve pas la chaîne de
  confiance** — indiquez ce que la simulation laisse de côté ;
- **étapes 3 et 4** : `openssl` en complément de Python. La voie Python seule est
  acceptée, à condition de livrer les commandes de vérification correspondantes.

Les étapes 1, 5, 7 et 8 n'exigent que Python et `cryptography`.

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
