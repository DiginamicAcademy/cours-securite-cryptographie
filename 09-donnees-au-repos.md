[← Chapitre précédent](08-pki-et-certificats.md) · [Sommaire](Readme.md) · [Chapitre suivant →](10-secrets.md)

# Données au repos

> Un ordinateur portable est volé dans un train. Il contient le dossier de patients
> d'un cabinet médical : noms, coordonnées, comptes rendus. Si le disque est chiffré,
> le voleur repart avec un morceau de métal ; s'il ne l'est pas, il repart avec la vie
> privée de centaines de personnes. Le vol est un fait **physique** — hors du périmètre
> de ce cours ; la réponse **logicielle**, elle, tient en un mot : chiffrer ce qui
> repose.

**Durée de lecture :** 50 min hors approfondissements facultatifs · **Prérequis :** le chiffrement symétrique et la
gestion des clés (journée précédente)

---

## Objectifs

À l'issue de ce chapitre, vous serez capable de :

- **Distinguer** ce que protège le chiffrement de disque de ce que protège le
  chiffrement applicatif d'une colonne.
- **Choisir** le niveau de chiffrement adapté à chaque catégorie de données au repos.
- **Chiffrer** une colonne sensible d'une base de données depuis l'application.
- **Sécuriser** une sauvegarde et vérifier qu'elle est réellement restaurable.

---

## I. Chiffrement de disque

### Le premier niveau

Le **chiffrement de disque** (ou chiffrement au repos, *encryption at rest*) chiffre
l'intégralité d'un volume : le système d'exploitation le déchiffre à la volée quand il
lit un bloc, et le rechiffre à l'écriture. Pour l'utilisateur, rien ne change ; pour
quiconque démonte le disque, tout est illisible.

Trois implémentations courantes :

| Outil | Plateforme | Principe |
| --- | --- | --- |
| **BitLocker** | Windows | Chiffrement de volume, clé liée à la puce TPM de la machine |
| **LUKS** (*Linux Unified Key Setup*) | Linux | Volume chiffré géré par `cryptsetup`, ouvert par une phrase de passe |
| **FileVault** | macOS | Chiffrement de disque intégré au système |

Le chiffrement de disque répond à un scénario précis : **le support sort de son
environnement** — vol, revente de matériel, machine envoyée en réparation, disque
d'un serveur mis au rebut. C'est un rempart efficace, et souvent gratuit à activer.

### Ce qu'il ne protège pas

Le volume est déchiffré **tant que la machine tourne** : un attaquant qui arrive sur
une session ouverte, un malware exécuté avec les droits de l'utilisateur ou un
administrateur légitime lisent exactement la même chose qu'avant.

> [!WARNING]
> Le chiffrement de disque protège le **support**, pas la **donnée**. Toute copie qui
> en sort — dump SQL, CSV envoyé par courriel, sauvegarde, réplication — repart sans
> protection. C'est pour cette raison que la suite du chapitre traite la donnée
> elle-même.

Un point pratique qui décide de tout : **la clé de récupération**. Une phrase de passe
perdue ou une clé de récupération rangée dans le même tiroir que le portable, et le
chiffrement se retourne contre vous — les données sont perdues pour tout le monde, y
compris pour vous. La clé de récupération se range ailleurs : coffre, gestionnaire de
secrets, enveloppe scellée chez un responsable.

---

## II. Chiffrement de colonne

### Quand le disque ne suffit plus

Une base de données contient des informations dont la sensibilité dépasse celle du
serveur qui les héberge : coordonnées bancaires (IBAN, *International Bank Account
Number*), numéros d'identification, données de santé, historiques d'achats.

Le **chiffrement de colonne** déplace la protection au plus près de la donnée :
l'application chiffre la valeur avant de l'écrire, et la déchiffre à la lecture. La
base ne stocke que du chiffré — un dump de la table ne livre rien.

### Chiffrer, ou hacher : la question à poser avant

Toutes les données sensibles ne se traitent pas de la même façon :

- **Chiffrer** quand la donnée doit être **relue** : un IBAN pour payer un
  fournisseur, une adresse pour livrer. Le chiffrement est réversible, avec une clé.
- **Hacher** quand la donnée ne doit être que **reconnue** : un mot de passe se
  compare, il ne se relit jamais. On stocke une empreinte salée, jamais la valeur.

Confondre les deux est coûteux : chiffrer un mot de passe rend sa fuite exploitable
par quiconque détient la clé, et hacher un IBAN le rend inutilisable.

### En pratique, côté application

Le principe tient en quelques lignes — la bibliothèque `cryptography` de Python
fournit des primitives prêtes à l'emploi :

```python
from cryptography.fernet import Fernet
import os

# La clé vient de l'environnement, jamais du code source.
CLE = os.environ["CLE_CHIFFREMENT"]
f = Fernet(CLE)

def chiffrer(valeur: str) -> bytes:
    """Chiffre une valeur sensible avant écriture en base."""
    return f.encrypt(valeur.encode("utf-8"))

def dechiffrer(stocke: bytes) -> str:
    """Déchiffre une valeur lue en base."""
    return f.decrypt(stocke).decode("utf-8")
```

Trois règles se dégagent de ce schéma :

1. **La clé ne vit pas dans le code.** Elle vient d'une variable d'environnement ou
   d'un gestionnaire de secrets — le chapitre suivant est entièrement consacré à ce
   point.
2. **Chaque écriture produit un chiffré différent.** Les algorithmes modernes
   intègrent un nombre aléatoire (un *nonce*) par message : deux IBAN identiques ne
   donnent pas deux fois la même valeur stockée, ce qui limite ce qu'un observateur
   de la base peut deviner.
3. **La colonne change de nature.** Elle stocke désormais des octets chiffrés :
   plus de `LIKE`, plus de tri, plus de jointure sur cette colonne.

### Le problème de la recherche

Une colonne chiffrée ne se consulte plus. Pour retrouver « la fiche du fournisseur
dont l'IBAN est celui-ci », deux approches :

- **déchiffrer et comparer** : simple, mais linéaire — la base ne peut pas indexer ;
- l'**index aveugle** : on stocke à côté une empreinte de recherche, par exemple un
  HMAC (code d'authentification par hachage) de la valeur calculé avec une **autre
  clé**. La recherche se fait sur l'empreinte, la lecture sur le chiffré.

L'index aveugle révèle les **égalités** (deux lignes qui partagent la même valeur) —
c'est un compromis assumé, à documenter.

> 🧪 **Atelier rapide (8 min)**
>
> Ouvrez la base de démonstration du lab et regardez ce qu'elle stocke réellement :
>
> ```bash
> mysql -h localhost -u root -proot fromagerie
> ```
>
> puis, dans le client MySQL :
>
> ```sql
> SHOW TABLES;
> DESCRIBE users;
> SELECT username, password_hash FROM users LIMIT 5;
> ```
>
> Parmi les colonnes des tables `users`, `products`, `comments` et `orders`, lesquelles
> mériteraient un chiffrement applicatif, lesquelles un simple hachage, et lesquelles
> aucune protection particulière ? Notez votre classement : l'exercice du bloc y
> revient.

---

## III. Sauvegardes

### La cible préférée

Une sauvegarde est, par construction, **tout le système au même endroit**. C'est ce
qui en fait la cible favorite : un seul accès, et l'attaquant repart avec des années
de données — y compris celles que l'on a prises soin de chiffrer ailleurs, si la
sauvegarde contient aussi les clés. Trois principes :

- **Chiffrer la sauvegarde elle-même**, avec un outil qui le fait nativement (les
  outils de sauvegarde modernes chiffrent à la volée) ou un conteneur chiffré.
- **Séparer la clé de la sauvegarde.** Une sauvegarde chiffrée dont la clé est jointe
  au même disque n'est pas chiffrée, elle est décorée. La clé se range dans un autre
  lieu, un autre système, une autre autorité.
- **Rendre la sauvegarde immuable** quand c'est possible : une copie que ni le
  ransomware ni un administrateur pressé ne peut écraser ou supprimer.

### La restauration, seule preuve qui compte

Une sauvegarde non testée est une hypothèse. Le chiffrement ajoute un mode de défaillance
de plus : la clé perdue, le format oublié, le mot de passe de la phrase de passe
consigné nulle part. La seule preuve qu'une sauvegarde protégée est utilisable est un
**exercice de restauration** — périodique, daté, documenté, sur une machine qui n'est
pas celle de production.

> [!TIP]
> Adoptez le rythme simple : restaurer un échantillon à chaque changement majeur de
> la chaîne de sauvegarde, et une restauration complète au moins une fois par an. Notez
> la durée réelle de la restauration — c'est elle qui décide du temps d'indisponibilité
> en cas d'incident, pas la vitesse de la sauvegarde.

---

## IV. Aller plus loin

🔁 **Pour aller plus loin (facultatif)** — à lire si le créneau le permet, rien
d'obligatoire n'en dépend.

- **Confidentialité permanente** (*forward secrecy*) : dans les protocoles modernes
  comme TLS 1.3, la clé de session est **éphémère** — elle est calculée pour la
  session puis détruite. Un attaquant qui enregistre le trafic chiffré aujourd'hui et
  vole la clé privée du serveur demain ne peut pas déchiffrer le passé : les clés de
  session n'existent plus. C'est la différence entre « protégé tant que la clé tient »
  et « protégé pour toujours ».
- **Chiffrement par enveloppe** : la donnée est chiffrée par une clé de données, qui
  est elle-même chiffrée par une clé maître rangée dans un coffre (KMS, *Key
  Management Service*, ou module de sécurité matériel). On peut alors tourner la clé
  maître sans rechiffrer la base entière.
- **Le coût du chiffrement applicatif** : chaque lecture devient une opération
  cryptographique, chaque recherche un compromis. Le chapitre sur les compromis
  sécurité / performance de cette journée chiffre ce que cela représente.

---

## Points clés

- Le chiffrement de disque protège le **support** : vol, revente, mise au rebut. Il
  ne protège rien dès qu'une copie sort du volume.
- Le chiffrement de colonne protège la **donnée** elle-même, y compris dans un dump,
  une réplication ou un export.
- **Chiffrer** pour relire, **hacher** pour reconnaître — jamais l'inverse.
- Une colonne chiffrée ne se recherche plus : l'index aveugle (HMAC) est le compromis
  usuel, à documenter.
- Une sauvegarde se chiffre, se sépare de sa clé, et se **teste** : la restauration est
  la seule preuve qu'elle existe.

---

## 🧠 Vérifiez vos acquis

*Formatif, jamais noté. Répondez sans relire le chapitre, puis cochez. Les réponses sont juste après le séparateur : ce qui compte est de repérer où vous vacillez, pas de faire 5 sur 5.*

**1.** Le portable d'un salarié, dont le disque est chiffré, est compromis alors que sa session est ouverte et qu'un malware s'exécute avec ses droits. Que lit l'attaquant ?

- A. Rien : le disque est chiffré
- B. Exactement la même chose qu'avant : le volume est déchiffré tant que la machine tourne
- C. Uniquement les fichiers ouverts au moment de l'intrusion
- D. Les fichiers, mais sans les métadonnées

- [ ] ✓ je savais

**2.** Dans la base de DIGICHEESE, comment traiter respectivement l'IBAN d'un fournisseur et le mot de passe d'un client ?

- A. Les deux chiffrés, pour pouvoir les relire en cas de besoin
- B. Les deux hachés, pour ne jamais stocker la valeur
- C. L'IBAN chiffré (il doit être relu), le mot de passe haché (il doit être reconnu, jamais relu)
- D. L'IBAN haché, le mot de passe chiffré

- [ ] ✓ je savais

**3.** Une colonne « IBAN » passe en chiffrement applicatif. Que constate-t-on ensuite ?

- A. Les requêtes fonctionnent comme avant, seules les écritures ralentissent
- B. La colonne change de nature : plus de `LIKE`, plus de tri, plus de jointure dessus
- C. Deux IBAN identiques donnent la même valeur stockée, ce qui facilite la recherche
- D. La base peut indexer la colonne chiffrée comme n'importe quelle colonne

- [ ] ✓ je savais

**4.** Quelle affirmation sur les sauvegardes est correcte ?

- A. Une sauvegarde chiffrée dont la clé est sur le même disque est chiffrée
- B. Une sauvegarde non testée est une hypothèse : seule une restauration périodique, datée et documentée prouve qu'elle est utilisable
- C. Le chiffrement de la sauvegarde dispense de la rendre immuable
- D. C'est la vitesse de la sauvegarde qui décide du temps d'indisponibilité en cas d'incident

- [ ] ✓ je savais

**5.** *Question ouverte (2-3 lignes).* Il faut quand même pouvoir retrouver « la fiche du fournisseur dont l'IBAN est celui-ci ». Quel compromis le cours propose-t-il, comment fonctionne-t-il, et que révèle-t-il ?

- [ ] ✓ je savais

---

### Réponses

**1. B.** Le chiffrement de disque protège le **support**, pas la **donnée** : le système déchiffre à la volée à la lecture.
**Pourquoi :** il répond au scénario « le support sort de son environnement » — vol, revente, réparation, mise au rebut. Toute copie qui sort du volume (dump SQL, CSV par courriel, sauvegarde, réplication) repart sans protection.

**2. C.** Chiffrer quand la donnée doit être **relue**, hacher quand elle ne doit être que **reconnue** — jamais l'inverse.
**Pourquoi :** chiffrer un mot de passe rend sa fuite exploitable par quiconque détient la clé ; hacher un IBAN le rend inutilisable.

**3. B.** La colonne stocke désormais des octets chiffrés et ne se consulte plus. En revanche, chaque écriture produit un chiffré différent : les algorithmes modernes intègrent un *nonce* par message, donc deux IBAN identiques ne donnent pas deux fois la même valeur stockée.
**Pourquoi :** c'est ce qui limite ce qu'un observateur de la base peut deviner — et la clé doit venir de l'environnement, jamais du code source.

**4. B.** Une sauvegarde se chiffre, se **sépare** de sa clé (autre lieu, autre système, autre autorité — sinon elle est décorée, pas chiffrée), et se **teste**.
**Pourquoi :** le chiffrement ajoute des modes de défaillance (clé perdue, format oublié) ; et c'est la **durée réelle de la restauration**, pas la vitesse de la sauvegarde, qui décide du temps d'indisponibilité.

**5.** Attendu : l'**index aveugle** — on stocke à côté une empreinte de recherche, par exemple un HMAC de la valeur calculé avec une **autre clé** ; la recherche se fait sur l'empreinte, la lecture sur le chiffré (l'autre approche, déchiffrer et comparer, est linéaire car la base ne peut pas indexer).
**Pourquoi :** l'index aveugle révèle les **égalités** — deux lignes qui partagent la même valeur. C'est un compromis assumé, à documenter.

---

[← Chapitre précédent](08-pki-et-certificats.md) · [Sommaire](Readme.md) · [Chapitre suivant →](10-secrets.md)
