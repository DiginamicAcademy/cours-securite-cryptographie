[Sommaire](Readme.md) · [Chapitre suivant →](02-chiffrement-symetrique.md)

# Histoire et services de la cryptographie

> 400 av. J.-C., Sparte : un général enroule une bande de cuir autour d'un bâton de
> taille précise, écrit son message, puis déroule la bande. L'ennemi qui intercepte
> le courrier ne trouve qu'un gribouillis ; le destinataire, qui possède le même
> bâton, retrouve le texte en réenroulant. Toute la cryptographie est née de cette
> scène : **un secret partagé**, et l'art de le garder.

**Durée de lecture :** 35 min hors approfondissements facultatifs · **Prérequis :** aucun

---

## Objectifs

À l'issue de ce chapitre, vous serez capable de :

- **Décrire** le principe du secret partagé, de la scytale de Sparte aux clés d'aujourd'hui.
- **Nommer** les quatre services rendus par la cryptographie.
- **Distinguer** ce que la cryptographie garantit de ce qu'elle laisse hors de son périmètre.
- **Associer** une situation concrète au service de cryptographie qui la couvre.

---

## I. Pourquoi chiffrer ?

La **scytale** (du grec *skutálē*, le bâton) est l'un des plus anciens outils de
chiffrement connus : déroulé, le parchemin ne livre qu'un gribouillis ; il n'est
lisible qu'enroulé autour d'un bâton du même diamètre. Le dispositif est rudimentaire,
mais il contient déjà tout ce qui nous occupera :

- un **secret** — la taille du bâton, connue de l'émetteur et du destinataire ;
- un **texte clair** (le message lisible) et un **texte chiffré** (le gribouillis) ;
- une **opération réversible** : enrouler pour cacher, dérouler pour retrouver.

C'est le principe même de la **cryptographie** (du grec *kruptós*, caché, et
*gráphein*, écrire) : rendre un message inintelligible à quiconque ne possède pas le
secret.

### Le secret partagé

**Sans secret, pas de confidentialité** : la scytale ne protège que si la taille du
bâton reste inconnue ; le jour où l'ennemi la devine, le chiffrement ne vaut plus rien —
même avec des mathématiques parfaites.

L'image classique est celle de la **boîte à deux cadenas**. Alice veut envoyer un objet
à Bob sans que le transporteur y touche :

1. Alice pose son cadenas sur la boîte fermée et l'envoie.
2. Bob ajoute son propre cadenas, puis renvoie la boîte.
3. Alice retire son cadenas : il ne reste que celui de Bob.
4. Bob ouvre son cadenas et récupère l'objet.

À aucun moment un cadenas n'a circulé ouvert : **chacun garde sa clé pour lui**, et
personne d'autre ne la possède. Vous retrouverez ce principe au chapitre sur le
chiffrement asymétrique.

> 🧪 **Atelier rapide (5 min)**
>
> La fromagerie **DIGICHEESE** (l'entreprise d'étude du parcours) doit prévenir son
> transporteur de l'heure de passage.
> Écrivez « Rendez-vous 22h », puis décalez chaque lettre de trois rangs dans
> l'alphabet (A devient D, B devient E, et ainsi de suite). Que donne le message ?
> Montrez-le à quelqu'un sans explication : peut-il le lire ? Notez le résultat
> quelque part — vous allez le retrouver dans la journée.

---

## II. Ce que la cryptographie ne fait pas

La cryptographie protège **l'information** : ni les personnes, ni les locaux, ni les
usages. Trois limites :

- **Elle ne remplace pas un contrôle d'accès.** Un serveur dont le disque est chiffré
  reste ouvert à quiconque connaît le mot de passe de la session ouverte.
- **Elle ne protège pas contre l'utilisateur négligent.** Une clé écrite sur un
  post-it, un mot de passe transmis par courriel : la meilleure algorithmique du monde
  ne résiste pas à une clé qu'on donne.
- **Elle ne garantit pas qu'un message est *vrai*.** Elle peut prouver qu'un message
  n'a pas été modifié et d'où il vient ; elle ne dit pas que son contenu est juste.
  Une commande signée reste une commande erronée si elle a été signée par erreur.

Autrement dit : la cryptographie est un **outil**, pas une politique de sécurité. Elle
entre dans une chaîne — contrôle d'accès, sauvegardes, procédures, formation des
équipes — et c'est la chaîne qui tient, pas un maillon isolé.

---

## III. Les quatre services

La cryptographie rend quatre services distincts, souvent confondus. Les distinguer est
le premier réflexe à prendre : chacun répond à une question différente.

| Service | La question qu'il résout | Exemple chez DIGICHEESE |
| --- | --- | --- |
| **Confidentialité** | Seul le destinataire peut lire. | Les prix négociés avec les fournisseurs ne doivent pas fuiter vers un concurrent. |
| **Intégrité** | Le message n'a pas été modifié en route. | La fiche technique reçue par un client est exactement celle qui a été envoyée. |
| **Authentification** | L'émetteur est bien celui qu'il prétend être. | Le client parle bien au serveur de la boutique, et pas à un imposteur. |
| **Non-répudiation** | L'émetteur ne peut pas nier avoir envoyé. | La comptable ne peut pas nier avoir signé le bon de commande. |

> **À ne pas confondre** avec les piliers **CID** (Confidentialité, Intégrité,
> Disponibilité) : la **disponibilité** n'est pas un service de la cryptographie. Un
> système chiffré peut être parfaitement indisponible — un serveur démonté l'est tout
> autant. La cryptographie travaille sur le **contenu** des messages, pas sur leur
> **présence**.

### La signature, en une phrase

Le service de non-répudiation repose sur la **signature numérique** : en signant un bon
de commande, la comptable de DIGICHEESE produit une valeur que seul son secret permet de
créer, mais que tout le monde peut vérifier. Le fournisseur est sûr que la commande
vient d'elle et elle ne peut plus le nier — comme un sceau de cire prouve l'origine et
scelle le contenu.

---

### Exercice 1 — Classer les primitives (20 min)

#### Contexte

La fromagerie **DIGICHEESE** (l'entreprise d'étude du parcours) finalise sa boutique en
ligne. L'équipe a noté ses besoins
de sécurité au tableau blanc, mais personne n'est d'accord sur la technique à
employer : « on chiffre tout », « on hache tout », « une signature, c'est pareil
qu'un chiffrement »…

Vous êtes appelé à trancher. Aucun code n'est nécessaire : il s'agit de **choisir la
bonne primitive pour le bon problème**.

#### Consignes

##### Partie 1 — Reformuler le besoin

Pour chaque situation ci-dessous, dites en une phrase ce qu'il faut garantir :
**cacher** l'information, **vérifier** qu'elle n'a pas changé, ou **prouver** son
origine.

1. Les prix négociés avec les fournisseurs de la fromagerie ne doivent pas être connus
   des concurrents.
2. Un client qui télécharge la fiche technique d'un fromage doit pouvoir vérifier
   qu'elle n'a pas été modifiée en route.
3. La comptable envoie un bon de commande au fournisseur ; celui-ci doit pouvoir
   prouver, plus tard, que c'est bien elle qui l'a envoyé.
4. Les mots de passe des comptes clients sont conservés dans une base de données.
5. Un partenaire extérieur veut transmettre un document confidentiel à la fromagerie,
   sans que les deux parties aient pu se rencontrer pour se mettre d'accord au
   préalable sur un secret commun.
6. Le navigateur du client affiche un cadenas : le client veut s'assurer qu'il parle
   bien au serveur de la boutique, et pas à un site qui se fait passer pour elle.

##### Partie 2 — Associer usage et primitive

Complétez le tableau suivant. Pour chaque situation, retenez **une** primitive
principale parmi : **hachage**, **chiffrement symétrique**, **chiffrement
asymétrique**, **signature**.

| Situation | Besoin (cacher / vérifier / prouver) | Primitive principale |
| --- | --- | --- |
| 1 | | |
| 2 | | |
| 3 | | |
| 4 | | |
| 5 | | |
| 6 | | |

##### Partie 3 — Justifier un choix

Pour les situations 2 et 4, expliquez en deux phrases pourquoi la primitive retenue
convient — et pourquoi une autre ne conviendrait pas.

##### Partie 4 — Détecter le contresens

Un développeur de l'équipe déclare :

> « J'ai haché le fichier des prix pour que personne ne puisse le lire. »

Expliquez en deux ou trois phrases pourquoi cette affirmation est fausse, et ce
qu'il aurait fallu faire à la place.

#### ✅ Auto-évaluation

Avant de passer au corrigé, vérifiez que :

- [ ] J'ai reformulé les six besoins avec mes propres mots (cacher / vérifier / prouver).
- [ ] Mon tableau attribue **une** primitive principale à chaque situation.
- [ ] J'ai justifié les situations 2 et 4, y compris en écartant une autre primitive.
- [ ] Ma réponse à la partie 4 distingue clairement **hachage** et **chiffrement**.
- [ ] Je peux expliquer à voix haute pourquoi « chiffrer » et « hacher » ne sont pas interchangeables.

#### Prolongement de l'exercice

*Facultatif.* Parmi les six situations, lesquelles gagneraient à combiner **deux**
primitives ? Proposez la combinaison et dites ce qu'elle apporte.

<details>
<summary>Voir la correction</summary>

#### Partie 1 — Reformuler le besoin

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

#### Partie 2 — Associer usage et primitive

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

#### Partie 3 — Justifier un choix

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

#### Partie 4 — Détecter le contresens

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

#### Points à retenir

- Un besoin se formule avant de choisir un outil : **cacher**, **vérifier** ou
  **prouver** — la primitive découle de la question, jamais l'inverse.
- Le hachage **vérifie** l'intégrité, il ne cache rien ; le chiffrement **cache**, il
  ne vérifie rien.
- Le choix entre chiffrement symétrique et asymétrique se joue sur **la circulation de
  la clé**, pas sur le niveau de sécurité ressenti.
- La signature répond à deux questions à la fois : **qui a envoyé**, et **peut-il le
  nier**.

</details>

---

## IV. Aller plus loin

*Ces approfondissements sont facultatifs : ils ne sont jamais exigés pour terminer
la journée.*

- **Enigma et Purple** : pendant la Seconde Guerre mondiale, deux machines de
  chiffrement réputées incassables ont été pourtant cassées — la première par
  l'équipe d'Alan Turing à Bletchley Park, la seconde par les cryptanalystes
  américains. La leçon tient en une ligne : un chiffrement ne vaut que tant que son
  secret tient, et l'histoire montre qu'aucun secret ne tient indéfiniment.
- **ANSSI** (Agence nationale de la sécurité des systèmes d'information) : l'autorité
  française en matière de cybersécurité. Ses recommandations détaillent les
  algorithmes et les tailles de clés jugés fiables pour un usage professionnel.
- **La cryptographie avant l'informatique** : les codes de la Renaissance, le
  chiffre de Vigenère, les dictionnaires de codes diplomatiques. Un terrain de jeu
  idéal pour comprendre qu'un chiffrement « inviolable » l'est rarement longtemps.

---

## Points clés

- La cryptographie repose sur un **secret partagé** : sans lui, aucune garantie ne
  tient, si bon soit l'algorithme.
- Elle rend **quatre services** : confidentialité, intégrité, authentification,
  non-répudiation — quatre questions différentes, quatre outils différents.
- La **disponibilité** n'en fait pas partie : la cryptographie agit sur le contenu
  des messages, pas sur leur présence.
- Elle ne remplace ni le contrôle d'accès, ni la vigilance des utilisateurs, ni une
  politique de sécurité : elle est un maillon de la chaîne, pas la chaîne.

---

## 🧠 Vérifiez vos acquis

*Formatif, jamais noté. Répondez sans relire le chapitre, puis dépliez « Vérifier ses réponses ». Ce qui compte est de repérer où vous vacillez, pas de faire 5 sur 5.*

**1.** Les prix négociés par DIGICHEESE avec ses fournisseurs ne doivent pas tomber chez un concurrent. Quel service de la cryptographie répond à cette exigence ?

- A. L'intégrité
- B. La confidentialité
- C. L'authentification
- D. La non-répudiation

**2.** Le cours distingue les quatre services de la cryptographie des piliers CID. Lequel de ces éléments n'est **pas** un service rendu par la cryptographie ?

- A. La confidentialité
- B. L'intégrité
- C. La disponibilité
- D. La non-répudiation

**3.** Le jour où l'ennemi devine le diamètre du bâton, que devient la scytale ?

- A. Elle reste solide : le parchemin reste illisible en l'état
- B. Elle ne protège plus rien : sans secret partagé, le chiffrement ne vaut plus rien
- C. Elle ne protège plus la confidentialité, mais garde l'intégrité
- D. Elle devient plus lente à casser, ce qui suffit

**4.** Une commande est correctement signée, mais la comptable s'est trompée de quantité. Que garantit la signature ?

- A. Que la commande vient bien d'elle et n'a pas été modifiée, pas que son contenu est juste
- B. Que la commande est exacte : une signature valide prouve un contenu correct
- C. Que personne ne pourra lire la commande
- D. Rien : une commande erronée n'est pas signable

**5.** *Question ouverte (2-3 lignes).* Dans l'histoire de la boîte à deux cadenas, à aucun moment un cadenas ne circule ouvert. Quel principe du cours cette scène illustre-t-elle, et pourquoi est-ce important pour la suite du cours ?

---

<details>
<summary>Vérifier ses réponses</summary>

### Réponses

**1. B.** La confidentialité répond à « seul le destinataire peut lire » — c'est l'exemple même du tableau du chapitre.
**Pourquoi :** les trois autres services portent sur une autre question : modification en route, identité de l'émetteur, impossibilité de nier.

**2. C.** La disponibilité fait partie des piliers CID, mais pas des services de la cryptographie.
**Pourquoi :** la cryptographie travaille sur le **contenu** des messages, pas sur leur **présence** : un serveur démonté reste indisponible, chiffré ou non.

**3. B.** Tout repose sur le secret partagé — la taille du bâton, connue de l'émetteur et du destinataire seulement.
**Pourquoi :** le cours est explicite : sans secret, pas de confidentialité, même avec des mathématiques parfaites.

**4. A.** La cryptographie prouve qu'un message n'a pas été modifié et d'où il vient ; elle ne dit pas que son contenu est juste.
**Pourquoi :** une commande signée reste une commande erronée si elle a été signée par erreur — la cryptographie est un outil, pas une politique de sécurité.

**5.** Attendu : chacun garde sa clé pour soi et personne d'autre ne la possède — le secret partagé, sans jamais transmettre le secret lui-même. C'est important parce que c'est le principe retrouvé au chapitre sur le chiffrement asymétrique.
**Pourquoi :** la difficulté n'est pas de chiffrer, elle est de partager le secret sans l'exposer ; c'est tout l'enjeu de la journée.

</details>

---

[Sommaire](Readme.md) · [Chapitre suivant →](02-chiffrement-symetrique.md)
