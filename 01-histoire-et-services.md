[Sommaire](Readme.md)

# Histoire et services de la cryptographie

> 400 av. J.-C., Sparte : un général enroule une bande de cuir autour d'un bâton de
> taille précise, écrit son message, puis déroule la bande. L'ennemi qui intercepte
> le courrier ne trouve qu'un gribouillis ; le destinataire, qui possède le même
> bâton, retrouve le texte en réenroulant. Toute la cryptographie est née de cette
> scène : **un secret partagé**, et l'art de le garder.

**Durée de lecture :** 45 min · **Prérequis :** aucun

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
chiffrement connus. L'émetteur enroule une bande de parchemin autour d'un bâton de
diamètre donné, écrit son message le long de la bande, puis la déroule : les lettres
sont désormais dispersées. Seul qui possède un bâton du même diamètre reconstitue le
message en réenroulant la bande.

Le dispositif est rudimentaire, mais il contient déjà tout ce qui nous occupera :

- il y a un **secret** — la taille du bâton, connue de l'émetteur et du destinataire ;
- il y a un **texte clair** (le message lisible) et un **texte chiffré** (le gribouillis) ;
- il y a une **opération réversible** : enrouler pour cacher, dérouler pour retrouver.

C'est le principe même de la **cryptographie** (du grec *kruptós*, caché, et
*gráphein*, écrire) : rendre un message inintelligible à quiconque ne possède pas le
secret.

### Le secret partagé

Toute la cryptographie repose sur une idée simple : **sans secret, pas de
confidentialité**. La scytale ne protège que si la taille du bâton reste inconnue de
l'ennemi. Le jour où l'adversaire la devine, le chiffrement ne vaut plus rien —
même si les mathématiques derrière sont parfaites.

L'image classique est celle de la **boîte à deux cadenas**. Alice veut envoyer un
objet à Bob sans que le transporteur y touche :

1. Alice pose son cadenas sur la boîte fermée et l'envoie.
2. Bob ajoute son propre cadenas, puis renvoie la boîte.
3. Alice retire son cadenas : il ne reste que celui de Bob.
4. Bob ouvre son cadenas et récupère l'objet.

À aucun moment un cadenas n'a circulé ouvert, et pourtant l'échange a eu lieu. Le
point décisif : **chacun garde sa clé pour lui**, et personne d'autre ne la possède.
Vous retrouverez ce principe plus loin dans la journée, avec la cryptographie
asymétrique.

> 🧪 **Atelier rapide (5 min)**
>
> La fromagerie **DIGICHEESE** (la fromagerie du fil rouge) doit prévenir son
> transporteur de l'heure de passage.
> Écrivez « Rendez-vous 22h », puis décalez chaque lettre de trois rangs dans
> l'alphabet (A devient D, B devient E, et ainsi de suite). Que donne le message ?
> Montrez-le à quelqu'un sans explication : peut-il le lire ? Notez le résultat
> quelque part — vous allez le retrouver dans la journée.

---

## II. Ce que la cryptographie ne fait pas

La cryptographie protège **l'information**. Elle ne protège ni les personnes, ni les
locaux, ni les usages. Trois limites à garder en tête :

- **Elle ne remplace pas un contrôle d'accès.** Un serveur dont le disque est chiffré
  reste ouvert à quiconque connaît le mot de passe de la session ouverte. Chiffrer ne
  dispense pas de verrouiller.
- **Elle ne protège pas contre l'utilisateur négligent.** Une clé écrite sur un
  post-it collé sur l'écran, un mot de passe transmis par courriel : la meilleure
  algorithmique du monde ne résiste pas à une clé qu'on donne.
- **Elle ne garantit pas qu'un message est *vrai*.** Elle peut prouver qu'un message
  n'a pas été modifié et d'où il vient ; elle ne dit pas que son contenu est juste.
  Une commande signée reste une commande erronée si elle a été signée par erreur.

Autrement dit : la cryptographie est un **outil**, pas une politique de sécurité. Elle
entre dans une chaîne — contrôle d'accès, sauvegardes, procédures, formation des
équipes — et c'est la chaîne qui tient, pas un maillon isolé.

---

## III. Les quatre services

La cryptographie rend quatre services distincts. On les confond souvent ; savoir les
distinguer est le premier réflexe à prendre, car chacun répond à une question
différente.

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

Le service de non-répudiation repose sur la **signature numérique**. Quand la
comptable de DIGICHEESE signe un bon de commande, elle produit une valeur que seul
son secret permet de créer — mais que tout le monde peut vérifier. Résultat : le
fournisseur est sûr que la commande vient d'elle, et elle ne peut plus prétendre le
contraire. C'est l'équivalent moderne du sceau de cire apposé sur une lettre : il
prouve l'origine et scelle le contenu.

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

[Sommaire](Readme.md)
