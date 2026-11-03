# Corrigé — Classer les primitives

> Corrigé de [l'exercice « Classer les primitives »](../01-classer-primitives.md).
> Comparez d'abord votre travail aux **étapes** ci-dessous : la méthode compte
> autant que le résultat.

---

## Partie 1 — Reformuler le besoin

**Éléments attendus :**

| Situation | Ce qu'il faut garantir |
| --- | --- |
| 1 | **Cacher** — les prix doivent rester secrets. |
| 2 | **Vérifier** — la fiche doit être intacte à l'arrivée. |
| 3 | **Prouver** — l'origine du bon de commande doit être incontestable. |
| 4 | **Vérifier** — et surtout pas cacher : on doit pouvoir *reconnaître* le mot de passe, pas le relire. |
| 5 | **Cacher** — le document doit être illisible pour quiconque l'intercepte. |
| 6 | **Prouver** — l'identité du serveur doit être établie. |

**Ce qu'il faut retenir :** la situation 4 est le piège classique. On ne « lit » pas
un mot de passe stocké : on le **compare**. C'est un besoin de vérification, pas de
confidentialité — même si ce qui est rangé en base ressemble à du texte chiffré.

---

## Partie 2 — Associer usage et primitive

**Éléments attendus :**

| Situation | Besoin | Primitive principale | Pourquoi celle-ci |
| --- | --- | --- | --- |
| 1 | Cacher | **Chiffrement symétrique** | Une seule équipe manipule ces prix : une clé partagée entre eux suffit, c'est le plus simple et le plus rapide. |
| 2 | Vérifier | **Hachage** | Il faut détecter toute modification, sans secret à distribuer ni à protéger. |
| 3 | Prouver | **Signature** | Seule la comptable peut produire cette signature ; le fournisseur la vérifie sans détenir son secret. |
| 4 | Vérifier | **Hachage salé** | On range une empreinte, jamais le mot de passe lui-même. |
| 5 | Cacher | **Chiffrement asymétrique** | Les deux parties ne se sont pas rencontrées : la clé publique du destinataire permet de chiffrer sans avoir échangé de secret au préalable. |
| 6 | Prouver | **Signature** | Le serveur prouve son identité en signant ; le navigateur vérifie avec la clé publique correspondante. |

**Erreurs fréquentes :** attribuer le **chiffrement symétrique** à la situation 5,
parce que « chiffrer, c'est chiffrer ». Le critère qui tranche n'est pas le résultat
(lu ou non lu) mais la **manière dont la clé circule** : si les parties ne peuvent
pas se rencontrer pour se mettre d'accord, seul le chiffrement asymétrique
fonctionne. Inversement, choisir le chiffrement asymétrique partout « parce que c'est
plus sûr » est un contresens : c'est plus lourd, et inutile quand une clé partagée
existe déjà.

---

## Partie 3 — Justifier un choix

**Situation 2 — Hachage :**

La fiche technique est un document public : il n'y a rien à cacher, seulement à
vérifier. Une fonction de hachage produit une empreinte qui change au moindre
changement du document, et cette empreinte se publie sans risque. Le chiffrement
conviendrait mal : il exigerait de distribuer une clé à chaque client, pour un
secret qui n'existe pas.

**Situation 4 — Hachage salé :**

Le serveur doit reconnaître un mot de passe, pas le restituer. On range donc une
empreinte, que l'on compare à celle calculée lors de la connexion. Le chiffrement
serait dangereux : il rendrait le mot de passe lisible par quiconque détient la clé,
y compris en cas de fuite de la base. Le **sel** — une valeur aléatoire ajoutée avant
le hachage — évite que deux mots de passe identiques donnent la même empreinte.

**Ce qu'il faut retenir :** le hachage **vérifie**, il ne **cache** pas. C'est
exactement la confusion de la partie suivante.

---

## Partie 4 — Détecter le contresens

**Éléments attendus :**

L'affirmation est fausse parce qu'elle confond deux opérations. Un hachage est
**à sens unique** : il produit une empreinte dont on ne revient pas au fichier
d'origine, mais il ne masque rien du tout — et surtout, il n'est pas conçu pour
rendre un contenu illisible. Pire : si l'on peut « hacher » un fichier, on peut aussi
le hacher soi-même pour vérifier une hypothèse ; l'opération ne demande aucune clé.

Ce qu'il fallait faire : **chiffrer** le fichier des prix, avec une clé tenue secrète
et distribuée uniquement aux personnes autorisées à connaître ces prix.

**Sur la formulation :** l'erreur vient souvent du vocabulaire. « Hacher » sonne
comme « mettre sous scellé », alors que c'est « réduire à une empreinte ». Quand un
besoin se formule par « personne ne doit pouvoir **lire** », la réponse est un
chiffrement ; quand il se formule par « personne ne doit pouvoir **modifier sans que
je le voie** », la réponse est un hachage.

---

## Points à retenir

- Un besoin se formule avant de choisir un outil : **cacher**, **vérifier** ou
  **prouver** — la primitive découle de la question, jamais l'inverse.
- Le hachage **vérifie** l'intégrité, il ne cache rien ; le chiffrement **cache**, il
  ne vérifie rien.
- Le choix entre chiffrement symétrique et asymétrique se joue sur **la circulation de
  la clé**, pas sur le niveau de sécurité ressenti.
- La signature répond à deux questions à la fois : **qui a envoyé**, et **peut-il le
  nier**.
