<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/Logo-diginamic-color-blk.png">
    <img src="assets/Logo-diginamic-color.svg" alt="Diginamic Formation" width="358">
  </picture>
</p>

# La cryptographie

> Cours d'**autoformation** · **RNCP40573** (Répertoire national des certifications professionnelles) — Expert en informatique et systèmes d'information · **brique transversale de sécurité** adossée au bloc **BC03** (conception et développement applicatif)
> 2 journées · supports **rédigés par IA** (intelligence artificielle), relus et validés.

---

## Pourquoi ce cours

Un mot de passe mal stocké, une sauvegarde non chiffrée, une clé laissée dans le code source : la cryptographie ne se règle pas à la fin d'un projet, elle se décide au début. Mal employée, elle donne un sentiment de protection ; bien employée, elle couvre des besoins très concrets — cacher un prix fournisseur, vérifier qu'une fiche technique n'a pas été modifiée, prouver l'origine d'un bon de commande.

Les incidents le rappellent régulièrement : en 2011, l'attaque de la PlayStation Network a exposé les données de 77 millions de comptes. La leçon tient en une ligne : chiffrer ne sert à rien si la clé est à portée de l'attaquant.

Ce cours donne les repères pour choisir et mettre en œuvre les bons outils : distinguer hachage, chiffrement et signature ; comprendre ce que garantissent TLS (Transport Layer Security) et les certificats ; chiffrer les données au repos ; gérer les secrets sans les laisser traîner. Tout au long des deux journées, les exemples suivent la fromagerie **DIGICHEESE** (la fromagerie du fil rouge) : ses prix fournisseurs, ses bons de commande, sa boutique en ligne et ses bases de données.

## À l'issue de ce cours

- **Décrire** les quatre services rendus par la cryptographie : confidentialité, intégrité, authentification et non-répudiation.
- **Expliquer** le principe du chiffrement symétrique et celui du chiffrement asymétrique.
- **Distinguer** hachage, chiffrement et signature, et choisir la bonne primitive pour un besoin donné.
- **Utiliser** une bibliothèque de cryptographie standard pour chiffrer, déchiffrer et hacher.
- **Repérer** les erreurs d'implémentation courantes : mode ECB (Electronic Codebook), hachage MD5 (Message-Digest 5), sel absent, vecteur d'initialisation réutilisé.
- **Choisir** un algorithme et une taille de clé adaptés à un usage donné.
- **Décrire** le fonctionnement de TLS et de la confiance par certificats.
- **Expliquer** le rôle d'une autorité de certification dans une infrastructure à clés publiques.
- **Appliquer** le chiffrement des données au repos : bases de données, fichiers, sauvegardes.
- **Mettre en œuvre** la gestion des secrets : coffre de secrets, variables d'environnement, rotation.
- **Analyser** les failles de configuration courantes de HTTPS (Hypertext Transfer Protocol Secure) et de TLS.
- **Évaluer** le compromis entre protection, performance et coût des choix de cryptographie.

## Sommaire

| # | Chapitre | Durée de lecture |
| --- | --- | --- |
| 01 | [Histoire et services de la cryptographie](01-histoire-et-services.md) | 45 min |
| 02 | [Du chiffre de César à AES](02-chiffrement-symetrique.md) | 75 min |
| 03 | [Le chiffrement asymétrique](03-chiffrement-asymetrique.md) | 70 min |
| 04 | [Le hachage et la signature numérique](04-hachage-et-signature.md) | 60 min |
| 05 | [Les erreurs classiques de cryptographie](05-erreurs-classiques.md) | 45 min |
| 06 | [L'aléa et la gestion des clés](06-alea-et-cles.md) | 45 min |
| 07 | [TLS en pratique](07-tls-en-pratique.md) | 75 min |
| 08 | [PKI et certificats](08-pki-et-certificats.md) | 60 min |
| 09 | [Données au repos](09-donnees-au-repos.md) | 65 min |
| 10 | [Gestion des secrets](10-secrets.md) | 55 min |
| 11 | [Chiffrement des données data](11-chiffrement-data.md) | 55 min |
| 12 | [Compromis et réponse](12-compromis.md) | 45 min |

**Travaux pratiques**

| | Sujet | Durée |
| --- | --- | --- |
| 🟢 | [Exercice — Classer les primitives](tp/01-classer-primitives.md) · [corrigé](tp/corrige/01-classer-primitives.md) | 20 min |
| 🟢 | [Exercice — Casser le chiffre de César](tp/02-crypter-cesar.md) · [corrigé](tp/corrige/02-crypter-cesar.md) | 30 min |
| 🟡 | [Exercice — Chiffrer un fichier avec AES-GCM](tp/03-chiffrer-python.md) · [corrigé](tp/corrige/03-chiffrer-python.md) | 45 min |
| 🔵 | [Exercice — Auditer un code qui utilise mal la crypto](tp/05-auditer-crypto.md) · [corrigé](tp/corrige/05-auditer-crypto.md) | 55 min |
| 🔴 | [Défi — Concevoir un plan de gestion des clés](tp/06-defi-projet-cles.md) · [corrigé](tp/corrige/06-defi-projet-cles.md) | — |
| 🟢 | [Exercice — Diagnostiquer une configuration TLS](tp/07-diagnostiquer-tls.md) · [corrigé](tp/corrige/07-diagnostiquer-tls.md) | 25 min |
| 🟡 | [Exercice — Chiffrer une colonne sensible](tp/09-chiffrer-bdd.md) · [corrigé](tp/corrige/09-chiffrer-bdd.md) | 45 min |
| 🔵 | [Exercice — Auditer la gestion des secrets](tp/10-audit-secrets.md) · [corrigé](tp/corrige/10-audit-secrets.md) | 55 min |
| 🔴 | [Exercice — Défi : planifier une rotation de clés](tp/12-defi-rotation.md) · [corrigé](tp/corrige/12-defi-rotation.md) | — |
| 🎯 | **TP intégrateur — seul élément noté du cours** · [Sécuriser un système de bout en bout](tp/TP-INTEGRATEUR.md) | 3 h 30 |

---

## ✅ Checkpoints de fin de journée

À cocher avant de passer à la journée suivante. En cas de doute : ne pas
avancer, revenir au concept concerné, refaire l'exercice.

### Jour 5 — chapitres 01 à 06

- [ ] J'ai lu les chapitres 01 à 06 et les points clés sont clairs.
- [ ] J'ai fait au moins les exercices 🟢 et 🟡 (partie principale), et vérifié mes réponses dans `tp/corrige/`. Les extensions « Pour aller plus loin » sont facultatives.
- [ ] J'ai répondu au QCM formatif, lorsqu'il est disponible, et identifié mes notions à revoir.
- [ ] Je sais expliquer, avec mes mots, les 2-3 idées principales de la journée.
- [ ] Je peux appliquer ces idées à un cas simple (un extrait de code, une config, un flux).

### Jour 6 — chapitres 07 à 12

- [ ] J'ai lu les chapitres 07 à 12 et les points clés sont clairs.
- [ ] J'ai fait au moins les exercices 🟢 et 🟡 (partie principale), et vérifié mes réponses dans `tp/corrige/`. Les extensions « Pour aller plus loin » sont facultatives.
- [ ] J'ai répondu au QCM formatif, lorsqu'il est disponible, et identifié mes notions à revoir.
- [ ] Je sais expliquer, avec mes mots, les 2-3 idées principales de la journée.
- [ ] Je peux appliquer ces idées à un cas simple (un extrait de code, une config, un flux).

---

## Aller plus loin

*Ces approfondissements sont facultatifs : ils ne sont jamais exigés pour terminer le cours.*

- **Les recommandations de l'ANSSI** (Agence nationale de la sécurité des systèmes d'information) : l'autorité française en matière de cybersécurité publie les algorithmes et les tailles de clés jugés fiables pour un usage professionnel.
- **Deux ouvrages de référence** : *Applied Cryptography* (Bruce Schneier) pour les fondements, *Serious Cryptography* (Jean-Philippe Aumasson) pour une vision moderne et pratique.
- **La cryptographie post-quantique** : les algorithmes conçus pour résister aux ordinateurs quantiques commencent à être standardisés — un sujet à suivre pour tout choix de long terme.
