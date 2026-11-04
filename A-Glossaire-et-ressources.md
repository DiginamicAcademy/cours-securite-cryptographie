[← Chapitre précédent](12-compromis.md) · [Sommaire](Readme.md)

# Glossaire et ressources

Ce document rassemble les acronymes employés dans le cours, les termes techniques clés et une sélection de ressources pour aller plus loin. Chaque acronyme y est donné avec sa signification complète, comme à sa première occurrence dans le cours, suivi des chapitres où il est employé. Les adresses des ressources ont été vérifiées le 4 novembre 2026.

---

## Acronymes

- **3DES** — *Triple DES* : DES appliqué trois fois pour compenser la brièveté de sa clé ; aujourd'hui trop lent et trop faible. (chapitres 02, 07)
- **ACME** — *Automatic Certificate Management Environment* : protocole qui automatise la délivrance et le renouvellement des certificats, utilisé par Let's Encrypt. (chapitres 07, 08)
- **AEAD** — *Authenticated Encryption with Associated Data* : chiffrement authentifié, qui garantit confidentialité et intégrité en une seule passe (AES-GCM, ChaCha20-Poly1305). (chapitres 02, 07)
- **AES** — *Advanced Encryption Standard* : standard de chiffrement symétrique par blocs de 128 bits, avec des clés de 128, 192 ou 256 bits. (chapitres 02, 03)
- **ANSSI** — Agence nationale de la sécurité des systèmes d'information : autorité française de cybersécurité, référence pour les recommandations cryptographiques. (chapitres 01, 02)
- **API** — *Application Programming Interface* : interface de programmation, par laquelle un service est appelé par un autre programme. (chapitres 06, 11)
- **AWS** — *Amazon Web Services* : fournisseur de cloud, dont les accès sont protégés par une paire de clés à ne jamais commiter. (chapitre 10)
- **BC03** — bloc de compétences 3 du RNCP40573 (conception et développement applicatif), auquel ce cours est adossé. (Readme)
- **BEAST** — *Browser Exploit Against SSL/TLS* : attaque de 2011 contre le mode CBC de TLS 1.0. (chapitre 07)
- **BI** — *Business Intelligence* : outils de restitution et tableaux de bord alimentés par l'entrepôt. (chapitre 11)
- **CA** — *Certificate Authority* : autorité de certification, qui émet et signe les certificats. (chapitre 08)
- **CAA** — *Certification Authority Authorization* : enregistrement DNS qui indique quelles autorités sont habilitées à émettre pour un domaine. (chapitre 08)
- **CBC** — *Cipher Block Chaining* : mode de chiffrement par blocs où chaque bloc dépend du précédent. (chapitres 02, 07)
- **CFB** — *Cipher Feedback* : mode de chiffrement par blocs qui produit un flot. (chapitre 02)
- **CI/CD** — intégration continue / déploiement continu : chaîne qui construit, teste et déploie le code à chaque changement. (chapitre 10)
- **CID** — Confidentialité, Intégrité, Disponibilité : les trois piliers de la sécurité — à ne pas confondre avec les quatre services de la cryptographie. (chapitre 01)
- **CN** — *Common Name* : champ d'un certificat qui porte le nom de l'entité certifiée. (chapitre 08)
- **CRL** — *Certificate Revocation List* : liste de révocation publiée par une autorité de certification. (chapitre 08)
- **CSPRNG** — *Cryptographically Secure Pseudo-Random Number Generator* : générateur pseudo-aléatoire conçu pour la cryptographie (`secrets`, `os.urandom` en Python). (chapitre 02)
- **CTR** — *Counter* : mode de chiffrement par blocs transformé en flot. (chapitre 02)
- **CT** — *Certificate Transparency* : journaux publics dans lesquels les certificats émis doivent être inscrits. (chapitre 08)
- **CSV** — *Comma-Separated Values* : format de fichier tabulaire en texte brut. (chapitres 09, 11)
- **CVE** — *Common Vulnerabilities and Exposures* : base publique de référence des vulnérabilités logicielles. (chapitre 05)
- **DES** — *Data Encryption Standard* : ancien standard de chiffrement par blocs, à clé de 56 bits, obsolète. (chapitres 02, 07)
- **DNS** — *Domain Name System* : annuaire qui associe noms de domaine et adresses. (chapitres 07, 08)
- **DSA** — *Digital Signature Algorithm* : algorithme de signature numérique, supplanté par ses variantes à courbes elliptiques. (chapitre 12)
- **ECC** — *Elliptic Curve Cryptography* : cryptographie à courbes elliptiques, même sécurité que RSA avec des clés plus courtes. (chapitre 03)
- **ECH** — *Encrypted Client Hello* : extension de TLS 1.3 qui chiffre le nom du site demandé. (chapitre 07)
- **ECB** — *Electronic Codebook* : mode de chiffrement qui réutilise le même bloc chiffré pour le même bloc clair — à proscrire. (chapitres 02, 05)
- **Ed25519** — schéma de signature numérique à courbes elliptiques, rapide et à clés courtes. (chapitre 03)
- **EISI** — Expert en informatique et systèmes d'information : titre professionnel visé par ce cours. (Readme)
- **GCM** — *Galois/Counter Mode* : mode de chiffrement authentifié d'AES. (chapitres 02, 03)
- **HMAC** — *Hash-based Message Authentication Code* : code d'authentification de message qui combine une fonction de hachage et une clé partagée. (chapitre 04)
- **HSM** — *Hardware Security Module* : module matériel qui génère et garde des clés sans jamais les livrer. (chapitre 06)
- **HSTS** — *HTTP Strict Transport Security* : en-tête HTTP qui interdit au navigateur de revenir en clair. (chapitres 07, 08)
- **HTTP** — *Hypertext Transfer Protocol* : protocole du web, en clair par défaut. (chapitre 07)
- **HTTPS** — *Hypertext Transfer Protocol Secure* : HTTP transporté par TLS. (chapitre 07)
- **IA** — intelligence artificielle. (Readme)
- **IBAN** — *International Bank Account Number* : numéro de compte bancaire normalisé. (chapitre 09)
- **ISRG** — *Internet Security Research Group* : association à l'origine de Let's Encrypt. (chapitres 07, 08)
- **IV** — *Initialization Vector* : vecteur d'initialisation, valeur de départ qui doit être unique sans être secrète. (chapitre 02)
- **JSON** — *JavaScript Object Notation* : format d'échange de données textuel. (chapitres 04, 05)
- **KDF** — *Key Derivation Function* : fonction qui dérive une clé à partir d'une phrase secrète ou d'une autre clé. (chapitre 06)
- **KEM** — *Key Encapsulation Mechanism* : mécanisme d'encapsulation de clé, base des échanges post-quantiques. (chapitre 12)
- **KMS** — *Key Management Service* : service de gestion de clés — génération, stockage, rotation. (chapitre 06)
- **LUKS** — *Linux Unified Key Setup* : standard de chiffrement de volume sous Linux. (chapitre 09)
- **MD5** — *Message-Digest 5* : fonction de hachage cassée, à ne plus employer. (chapitres 04, 05)
- **ML-DSA** — *Module-Lattice Digital Signature Algorithm* : standard de signature post-quantique. (chapitre 12)
- **ML-KEM** — *Module-Lattice Key Encapsulation Mechanism* : standard d'échange de clés post-quantique. (chapitre 12)
- **MySQL** — système de gestion de bases de données relationnelles. (chapitre 09)
- **NIST** — *National Institute of Standards and Technology* : organisme américain de normalisation, référence en cryptographie. (chapitres 05, 12)
- **OCSP** — *Online Certificate Status Protocol* : protocole qui interroge l'état de révocation d'un certificat. (chapitres 07, 08)
- **OFB** — *Output Feedback* : mode de chiffrement par blocs qui produit un flot. (chapitre 02)
- **PBKDF2** — *Password-Based Key Derivation Function 2* : fonction de dérivation de clé depuis une phrase secrète, à itérations nombreuses. (exercice 03)
- **PGP** — *Pretty Good Privacy* : logiciel de chiffrement et de signature de fichiers et de courriels. (chapitre 04)
- **PKI** — *Public Key Infrastructure* : infrastructure à clés publiques — autorités de certification, certificats, révocation. (chapitre 08)
- **PME** — petite et moyenne entreprise. (exercice 02)
- **POODLE** — *Padding Oracle On Downgraded Legacy Encryption* : attaque de 2014 contre SSL 3.0. (chapitre 07)
- **PRNG** — *Pseudo-Random Number Generator* : générateur pseudo-aléatoire, insuffisant pour la cryptographie. (chapitre 02)
- **RC4** — ancien chiffrement par flot, cassé. (chapitre 07)
- **RFC** — *Request For Comments* : document de normalisation de l'Internet — les versions de TLS sont décrites dans des RFC. (chapitre 07)
- **RGPD** — Règlement général sur la protection des données : cadre européen de protection des données personnelles. (chapitres 11, 12)
- **RNCP** — Répertoire national des certifications professionnelles. (Readme)
- **RSA** — Rivest-Shamir-Adleman : algorithme de chiffrement et de signature asymétriques, fondé sur la difficulté de la factorisation. (chapitre 03)
- **S3** — *Simple Storage Service* : service de stockage objet d'AWS. (chapitre 11)
- **SCT** — *Signed Certificate Timestamp* : preuve signée qu'un certificat a été inscrit dans un journal de transparence. (chapitre 08)
- **SHA-1 / SHA-2 / SHA-3** — *Secure Hash Algorithm* : familles de fonctions de hachage ; SHA-1 est cassée, SHA-2 et SHA-3 sont recommandées. (chapitre 04)
- **SQL** — *Structured Query Language* : langage d'interrogation des bases de données relationnelles. (chapitre 09)
- **SSH** — *Secure Shell* : protocole d'administration à distance chiffré. (chapitre 03)
- **SSL** — *Secure Sockets Layer* : ancêtre de TLS, abandonné. (chapitre 07)
- **SWEET32** — attaque de 2016 contre les chiffrements à blocs de 64 bits, dont 3DES. (chapitre 07)
- **TLS** — *Transport Layer Security* : protocole qui chiffre et authentifie les communications réseau ; HTTPS en est l'usage le plus courant. (chapitres 03, 07)
- **TP** — travaux pratiques. (Readme)
- **TPM** — *Trusted Platform Module* : puce matérielle qui garde des clés liées à une machine. (chapitre 09)
- **WEP / WPA2 / WPA3** — *Wired Equivalent Privacy / Wi-Fi Protected Access* : protocoles de sécurité du Wi-Fi ; WEP est cassé, WPA3 est l'état de l'art. (chapitre 05)
- **X.509** — standard des certificats à clés publiques. (chapitre 08)
- **XOR** — ou exclusif : opération bit à bit, base de nombreux chiffrements. (chapitre 05)

---

## Termes techniques clés

- **Analyse fréquentielle** : attaque qui exploite la répétition des lettres ou des blocs dans un texte chiffré pour en deviner le contenu.
- **Attaque par chronométrage** : fuite d'information par la durée d'un calcul — d'où l'obligation de comparer les codes à temps constant.
- **Autorité de certification** : organisme de confiance qui signe les certificats et atteste de l'identité de leur détenteur.
- **Certificat** : document qui associe une identité à une clé publique, signé par une autorité de certification.
- **Chaîne de confiance** : succession de certificats, de la feuille présentée par le serveur jusqu'à une racine connue du client.
- **Chiffrement asymétrique** : chiffrement qui sépare une clé publique (que l'on diffuse) d'une clé privée (que l'on ne quitte jamais).
- **Chiffrement au repos / en transit** : protection des données stockées / des données qui circulent sur un réseau.
- **Chiffrement symétrique** : chiffrement qui utilise la même clé pour chiffrer et déchiffrer.
- **Clé de session** : clé symétrique éphémère, tirée pour une conversation ou un transfert, puis détruite.
- **Coffre de secrets** : service dédié au stockage des clés et secrets, avec journalisation des accès (par exemple Vault).
- **Comparaison à temps constant** : comparaison dont la durée ne dépend pas de l'endroit où les valeurs diffèrent (`hmac.compare_digest`).
- **Condensat (ou empreinte)** : résultat d'une fonction de hachage, de taille fixe, qui détecte toute modification du message.
- **Espace des clés** : ensemble des clés possibles d'un algorithme — sa taille décide de la faisabilité de la force brute.
- **Force brute** : essayer toutes les clés possibles jusqu'à trouver la bonne.
- **Hybride (échange)** : principe qui chiffre une clé de session avec l'asymétrique, et les données avec le symétrique — le fonctionnement de TLS.
- **Minimisation des données** : ne collecter et ne conserver que ce qui est strictement nécessaire (RGPD).
- **Nonce** : valeur de départ unique, jamais réutilisée avec la même clé, qui n'a pas besoin d'être secrète.
- **Non-répudiation** : impossibilité pour l'auteur d'un message de nier l'avoir produite — apportée par la signature.
- **Post-quantique** : algorithmes conçus pour résister aux calculateurs quantiques (ML-KEM, ML-DSA).
- **Rotation** : remplacement périodique d'une clé, en maintenant les anciennes clés lisibles le temps de la migration.
- **Révocation** : retrait de validité d'une clé ou d'un certificat compromis, sans attendre son expiration.
- **Salage (sel)** : valeur aléatoire ajoutée à un mot de passe avant hachage, pour empêcher les tables précalculées.
- **Signature numérique** : preuve d'origine et d'intégrité, produite avec une clé privée et vérifiée avec la clé publique correspondante.

---

## Ressources utiles

Adresses vérifiées le 4 novembre 2026.

| Ressource | Ce qu'elle apporte | Adresse |
| --- | --- | --- |
| ANSSI — recommandations cryptographiques | Les références françaises : algorithmes recommandés, tailles de clés, bonnes pratiques | https://cyber.gouv.fr/ |
| NIST — Cryptographic Standards and Guidelines | Les standards de référence (AES, SHA, recommandations de tailles de clés) | https://csrc.nist.gov/projects/cryptographic-standards-and-guidelines |
| CNIL — le RGPD | Le texte du règlement et ses explications officielles | https://www.cnil.fr/fr/reglement-europeen-protection-donnees |
| Let's Encrypt | Certificats gratuits, renouvellement automatisé, documentation ACME | https://letsencrypt.org/ |
| Mozilla SSL Configuration Generator | Une configuration TLS prête à l'emploi, à jour des bonnes pratiques | https://ssl-config.mozilla.org/ |
| Qualys SSL Labs Server Test | Tester la configuration TLS d'un site public, note et détail des faiblesses | https://www.ssllabs.com/ssltest/ |
| OWASP — Cryptographic Storage Cheat Sheet | La liste de contrôle du stockage chiffré côté application | https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html |
| Bibliothèque Python `cryptography` | La bibliothèque employée dans les exercices : documentation et exemples | https://cryptography.io/ |
| Cryptopals | Exercices pratiques de cryptanalyse, du plus simple au plus corsé | https://cryptopals.com/ |
| RFC 8446 — TLS 1.3 | Le texte de référence du protocole TLS 1.3 | https://www.rfc-editor.org/rfc/rfc8446.html |
| HashiCorp Vault | Un coffre de secrets complet : dépôt, accès tracés, rotation | https://developer.hashicorp.com/vault |
| keylength.com | Comparaison des tailles de clés recommandées selon les organismes | https://www.keylength.com/ |

---

[← Chapitre précédent](12-compromis.md) · [Sommaire](Readme.md)
