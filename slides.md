---
marp: true
RNCPBloc: [RNCP40573BC03]
theme: default
style: |
  section {background-color: #121114}
  h1,h2,h3 {color: #8393f0}
  p,ul,li,td,th {color: #ccc}
  section table {font-size: 0.85em}
  section table td {background-color: #121114}
  section table th {background-color: #272133; font-weight: bolder}
  pre {background-color: #17151a; color: #ccc}
  blockquote {border: 2px green solid; border-left-width: 15px; padding: 0.5em 15px;font-style: italic }
paginate: true
header: "La cryptographie"
footer: "![height:20px](https://raw.githubusercontent.com/DiginamicInternal/PublicAssets/refs/heads/main/Logo-diginamic-color-blk.png)"
---

# La cryptographie

## Cacher, vérifier, prouver

<!-- Diapositive d'ouverture. Deux journées, les fondamentaux puis la cryptographie appliquée. -->

---

## 🎯 Objectifs pédagogiques

À la fin de ce cours, vous serez capable de :

1. Nommer les quatre services de la cryptographie, et ce qu'elle ne couvre pas
2. Distinguer hachage, chiffrement symétrique, chiffrement asymétrique et signature
3. Choisir un algorithme et une taille de clé adaptés à un usage donné
4. Repérer les erreurs d'implémentation courantes dans un code ou une configuration
5. Décrire le fonctionnement de TLS (Transport Layer Security) et de la confiance par certificats
6. Chiffrer les données au repos et gérer les secrets d'une application

---

## Le parcours — première journée : les fondamentaux

*Six blocs à suivre dans l'ordre, du plus simple au plus technique.*

| # | Bloc | Thème |
| --- | --- | --- |
| 1 | Services de la cryptographie | Ce que fait (et ne fait pas) la cryptographie, ses quatre services |
| 2 | Chiffrement symétrique | Du chiffre de César au standard AES (Advanced Encryption Standard) |
| 3 | Chiffrement asymétrique | Clé publique, clé privée, échange de clés |
| 4 | Hachage et signature | Empreintes numériques et signatures |
| 5 | Erreurs classiques | Faire soi-même sa cryptographie, algorithmes obsolètes, fuites par chronométrage |
| 6 | Aléa et gestion des clés | Générer des clés imprévisibles, les renouveler |

---

## Le parcours — deuxième journée : la cryptographie appliquée

*Les briques de la première journée, posées sur du vrai.*

| # | Bloc | Thème |
| --- | --- | --- |
| 1 | TLS en pratique | La poignée de main, les certificats, les en-têtes de sécurité |
| 2 | Infrastructures à clés publiques | Chaîne de confiance et autorités de certification |
| 3 | Données au repos | Disques, bases de données, sauvegardes |
| 4 | Gestion des secrets | Variables d'environnement, coffres de secrets, rotation |
| 5 | Chiffrement des données | Flux d'intégration, entrepôts, RGPD (Règlement général sur la protection des données) |
| 6 | Compromis et réponse | Performance, éco-conception, fuite de clé |

---

## 1. Histoire et services de la cryptographie

<!-- Intercalaire du premier chapitre. -->

---

### La scytale de Sparte

> 400 av. J.-C. : un général enroule une bande de cuir autour d'un bâton de taille précise, écrit son message, puis déroule la bande. L'ennemi qui intercepte le courrier ne trouve qu'un gribouillis.

- Un **secret partagé** : la taille du bâton, connue de l'émetteur et du destinataire
- Un **texte clair** et un **texte chiffré**
- Une **opération réversible** : enrouler pour cacher, dérouler pour retrouver

<!-- Raconter la scène avant de montrer les trois points. Toute la cryptographie est née de là. -->

---

### Le secret partagé

**Sans secret, pas de confidentialité** : la scytale ne protège que si la taille du bâton reste inconnue de l'ennemi.

La boîte à deux cadenas — Alice veut envoyer un objet à Bob sans que le transporteur y touche :

1. Alice pose son cadenas sur la boîte fermée et l'envoie
2. Bob ajoute le sien, puis renvoie la boîte
3. Alice retire son cadenas : il ne reste que celui de Bob
4. Bob ouvre son cadenas et récupère l'objet

> À aucun moment un cadenas n'a circulé ouvert. Chacun garde sa clé pour lui.

---

### 🧪 Atelier rapide (5 min)

La fromagerie **DIGICHEESE** (la fromagerie du fil rouge) doit prévenir son transporteur de l'heure de passage.

1. Écrivez « Rendez-vous 22h »
2. Décalez chaque lettre de trois rangs dans l'alphabet : A devient D, B devient E, et ainsi de suite
3. Montrez le résultat à quelqu'un sans explication : peut-il le lire ?

> Notez votre message chiffré : vous allez le retrouver dans la journée.

---

### Ce que la cryptographie ne fait pas

Elle protège **l'information** — ni les personnes, ni les locaux, ni les usages.

- **Elle ne remplace pas un contrôle d'accès.** Un disque chiffré reste ouvert à quiconque connaît le mot de passe de la session ouverte.
- **Elle ne protège pas contre l'utilisateur négligent.** Une clé écrite sur un post-it, un mot de passe transmis par courriel.
- **Elle ne garantit pas qu'un message est vrai.** Une commande signée reste une commande erronée si elle a été signée par erreur.

> Un outil, pas une politique de sécurité : elle est un maillon de la chaîne, pas la chaîne.

---

### Les quatre services

| Service | La question qu'il résout | Exemple chez DIGICHEESE |
| --- | --- | --- |
| **Confidentialité** | Seul le destinataire peut lire | Les prix négociés avec les fournisseurs ne doivent pas fuiter vers un concurrent |
| **Intégrité** | Le message n'a pas été modifié en route | La fiche technique reçue par un client est exactement celle qui a été envoyée |
| **Authentification** | L'émetteur est bien celui qu'il prétend être | Le client parle bien au serveur de la boutique, et pas à un imposteur |
| **Non-répudiation** | L'émetteur ne peut pas nier avoir envoyé | La comptable ne peut pas nier avoir signé le bon de commande |

---

### À ne pas confondre avec les piliers de la sécurité

Les piliers **CID** (Confidentialité, Intégrité, Disponibilité) viennent d'un autre cadre : la **disponibilité** n'est pas un service de la cryptographie.

- Un système chiffré peut être parfaitement indisponible — un serveur démonté l'est tout autant
- La cryptographie travaille sur le **contenu** des messages, pas sur leur **présence**

---

### La signature, en une phrase

Le service de non-répudiation repose sur la **signature numérique**.

- La comptable de DIGICHEESE signe un bon de commande : seule son secret permet de produire cette valeur
- Tout le monde peut la vérifier : le fournisseur est sûr que la commande vient d'elle
- Elle ne peut plus prétendre le contraire

> L'équivalent moderne du sceau de cire apposé sur une lettre : il prouve l'origine et scelle le contenu.

---

### 🧪 Travaux pratiques : classer les primitives

La fromagerie DIGICHEESE finalise sa boutique en ligne. Au tableau blanc : « on chiffre tout », « on hache tout », « une signature, c'est pareil qu'un chiffrement »…

À vous de trancher, sans écrire une ligne de code :

1. Reformuler chaque besoin : **cacher**, **vérifier** ou **prouver**
2. Associer une primitive principale à chaque situation : hachage, chiffrement symétrique, chiffrement asymétrique, signature
3. Justifier deux choix en écartant une autre primitive
4. Détecter le contresens : « j'ai haché le fichier des prix pour que personne ne puisse le lire »

> 20 min · corrigé et auto-évaluation dans le support de cours.

---

### Points clés

- La cryptographie repose sur un **secret partagé** : sans lui, aucune garantie ne tient, si bon soit l'algorithme
- Elle rend **quatre services** : confidentialité, intégrité, authentification, non-répudiation — quatre questions différentes
- La **disponibilité** n'en fait pas partie : elle agit sur le contenu des messages, pas sur leur présence
- Elle ne remplace ni le contrôle d'accès, ni la vigilance des utilisateurs, ni une politique de sécurité

---

### Aller plus loin

*Ces approfondissements sont facultatifs : ils ne sont jamais exigés pour terminer la journée.*

- **Enigma et Purple** : deux machines de chiffrement réputées incassables, cassées pendant la Seconde Guerre mondiale — aucun secret ne tient indéfiniment
- **ANSSI** (Agence nationale de la sécurité des systèmes d'information) : ses recommandations détaillent les algorithmes et les tailles de clés jugés fiables pour un usage professionnel
- **La cryptographie avant l'informatique** : les codes de la Renaissance, le chiffre de Vigenère, les dictionnaires de codes diplomatiques

---

## Des questions ?

### Le dépôt reste la référence

Pour toute question [adressez-vous au service pédagogie](mailto:pedagogie@diginamic.fr)
