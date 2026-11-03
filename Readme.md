<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/Logo-diginamic-color-blk.png">
    <img src="assets/Logo-diginamic-color.svg" alt="Diginamic Formation" width="358">
  </picture>
</p>

# La cryptographie

> Cours d'**autoformation** · **RNCP40573** Expert en informatique et systèmes d'information · spécialisation **4B Big Data & IA**
> 2 journée(s) · supports **rédigés par IA**, relus et validés.

---

## Pourquoi ce cours

Chiffrer ne consiste pas seulement à rendre un message illisible : la cryptographie
répond à quatre questions précises — qui peut lire, si le contenu a été modifié en
route, qui l'a envoyé, et qui ne peut pas le nier. Ce cours donne les réflexes pour
poser ces questions avant de choisir un outil, du secret partagé de la scytale aux
clés d'aujourd'hui.

Le fil rouge est **DIGICHEESE** (la fromagerie du fil rouge) : ses prix fournisseurs,
ses bons de commande et sa boutique en ligne circulent sur des canaux qu'elle ne
maîtrise pas. Chaque notion s'y applique, puis s'étend à un environnement data :
TLS (Transport Layer Security), chaînes de certificats, chiffrement des données au
repos, gestion des secrets.

Les enjeux sont concrets : une implémentation négligée annule les garanties de
l'algorithme le plus solide, et une clé mal gérée fuit plus sûrement qu'aucun
chiffrement ne protège.

## À l'issue de ce cours

Vous serez capable de :

- **Nommer** les quatre services rendus par la cryptographie — confidentialité,
  intégrité, authentification, non-répudiation — et les distinguer de la
  disponibilité.
- **Distinguer** chiffrement symétrique, chiffrement asymétrique, hachage et signature
  numérique.
- **Chiffrer** et **déchiffrer** des données avec une bibliothèque cryptographique
  standard en Python.
- **Hacher** une donnée et contrôler son intégrité avec les fonctions SHA-2 (Secure
  Hash Algorithm 2) et SHA-3.
- **Identifier** les erreurs d'implémentation classiques : mode ECB (Electronic
  Codebook), MD5 (Message Digest 5), sel absent, vecteur d'initialisation réutilisé.
- **Choisir** un algorithme et une longueur de clé adaptés à un usage donné.
- **Décrire** le déroulement d'une poignée de main TLS et la confiance établie par les
  certificats.
- **Mettre en œuvre** le chiffrement des données au repos et la gestion des secrets
  (coffre de secrets, rotation des clés).
- **Analyser** les faiblesses d'une configuration TLS et de gestion des secrets dans un
  projet type.
- **Évaluer** le compromis entre sécurité, performance et coût d'un choix
  cryptographique.

## Sommaire

| # | Chapitre | Durée de lecture |
| --- | --- | --- |
| 01 | [Histoire et services de la cryptographie](01-histoire-et-services.md) | 45 min |

---

## Aller plus loin

*Ces approfondissements sont facultatifs : ils ne sont jamais exigés pour terminer
le cours.*

- **ANSSI (Agence nationale de la sécurité des systèmes d'information) — Guide de
  sélection d'algorithmes cryptographiques** : la référence française pour choisir les
  mécanismes et les tailles de clés à l'état de l'art.
  <https://messervices.cyber.gouv.fr/guides/mecanismes-cryptographiques>
- **OWASP (Open Worldwide Application Security Project) — Cryptographic Storage
  Cheat Sheet** : les bonnes pratiques de stockage cryptographique côté applicatif.
  <https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html>
- **France Compétences — fiche RNCP40573** : le référentiel officiel du titre visé,
  dont ce cours couvre la brique sécurité.
  <https://www.francecompetences.fr/recherche/rncp/40573>
- **pyca/cryptography — la bibliothèque cryptographique de référence en Python** :
  recettes et primitives pour chiffrer, hacher et signer.
  <https://cryptography.io/>
- **Let's Encrypt — documentation officielle** : la délivrance et le renouvellement
  automatisés des certificats, en pratique.
  <https://letsencrypt.org/docs/>
