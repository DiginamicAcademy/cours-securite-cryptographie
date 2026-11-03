# Grille de vérification — Concevoir un plan de gestion des clés

> Grille d'auto-correction du [défi « Concevoir un plan de gestion des clés »](../06-defi-projet-cles.md).
> Un défi se juge sur la **complétude et la cohérence** du plan, pas sur une solution
> unique : confrontez votre document aux critères ci-dessous, section par section.

---

## Comment utiliser cette grille

Pour chaque section, cochez ce que votre plan contient réellement. Une case non cochée
signale un trou à combler — pas une erreur de formulation. Un plan dont toutes les
cases sont cochées est recevable en l'état.

---

## 1. Inventaire

- [ ] Les quatre familles de secrets sont recensées : chiffrement des données au repos,
      jetons d'API (Application Programming Interface), comptes de service, certificats.
- [ ] Chaque entrée nomme un **propriétaire** (une personne ou une équipe, jamais
      « l'informatique »).
- [ ] Chaque entrée indique l'**emplacement** de stockage et l'**usage**.
- [ ] Les secrets encore en clair dans les scripts sont listés explicitement, avec leur
      remédiation.

**Ce qui fait la différence :** un inventaire qui distingue ce qui existe de ce qui
traîne encore. Le script signalé dans l'énoncé doit y figurer — sinon le plan part d'un
constat incomplet.

---

## 2. Création et stockage

- [ ] La création se fait avec un générateur adapté à la cryptographie (et non un
      générateur pseudo-aléatoire ordinaire).
- [ ] Le stockage est **hors du code source**, avec un emplacement unique par
      environnement.
- [ ] Test et production sont **séparés** : jamais la même clé des deux côtés.
- [ ] Les accès au stockage de clés sont eux-mêmes protégés et journalisés.

---

## 3. Distribution et accès

- [ ] Le plan dit **qui** a accès à **quelle** clé, et selon quel rôle.
- [ ] La distribution se fait par un canal chiffré ou par enveloppement avec la clé
      publique du destinataire.
- [ ] Les accès sont revus à chaque changement d'équipe (arrivée, départ, changement de
      poste).
- [ ] Aucune clé n'est transmise par courriel, ticket ou messagerie instantanée.

---

## 4. Rotation

- [ ] Une **périodicité** est fixée pour chaque famille de clés.
- [ ] Les **événements déclencheurs** sont listés (départ, fuite suspectée, changement
      d'usage, dépréciation d'algorithme).
- [ ] La procédure **versionne** les clés : les données déjà chiffrées restent lisibles.
- [ ] La migration des données au repos est prévue, avec une étape de destruction de
      l'ancienne clé.

**Ce qui fait la différence :** une rotation qui préserve la lecture de l'existant. Un
plan qui remplace la clé sans versionner casse la plateforme — et sera contourné par les
équipes.

---

## 5. Révocation et incident

- [ ] Un **décideur** est nommé, avec un **délai** pour révoquer.
- [ ] La révocation est **immédiate** dès qu'une clé est suspecte, avant l'analyse de la
      fuite.
- [ ] La communication interne est décrite : qui est prévenu, dans quel ordre.
- [ ] La remédiation couvre ce qui a déjà été chiffré avec la clé compromise (rotation
      des données, notification si nécessaire).

---

## 6. Destruction

- [ ] Les critères de destruction sont posés (fin d'usage, fin de conservation, fin de
      migration).
- [ ] La destruction est **prouvable** (journal, attestation) et pas seulement déclarée.
- [ ] Les copies (sauvegardes, réplicas, exports) sont couvertes par la même exigence.

---

## Ce qu'un bon plan apporte de plus

- **Des responsables nommés**, pas des équipes anonymes.
- **Des délais**, pas « dès que possible ».
- **Une traçabilité** : on peut démontrer après coup ce qui a été fait.
- **Une proportionnalité** : la lourdeur de la procédure est calibrée sur la sensibilité
  des données — un plan applicable vaut mieux qu'un plan parfait qu'on ignore.

---

## Points à retenir

- Un plan de gestion des clés commence par un **inventaire** : sans lui, la rotation est
  impossible.
- Chaque clé a un **propriétaire**, un **emplacement**, un **usage** — et une date de
  fin.
- La rotation se conçoit pour **ne pas casser** la lecture des données existantes :
  versionner, migrer, détruire.
- La révocation est **immédiate** et **décidée** : un plan sans décideur ni délai ne
  sert pas le jour de l'incident.
- La destruction se **prouve** — y compris dans les sauvegardes.
