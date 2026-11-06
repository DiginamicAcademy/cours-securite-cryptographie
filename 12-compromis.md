[← Chapitre précédent](11-chiffrement-data.md) · [Sommaire](Readme.md) · [Chapitre suivant →](A-Glossaire-et-ressources.md)

# Compromis et réponse

> La cryptographie a un coût : cycles de processeur, latence, complexité
> opérationnelle. Trop de chiffrement, et l'application ralentit au point que les
> équipes le désactivent ; trop peu, et la fuite est là. La sécurité n'est pas un
> réglage que l'on pousse à fond : c'est un arbitrage permanent entre protection,
> performance et sobriété.

**Durée de lecture :** 40 min hors approfondissements facultatifs · **Prérequis :** les chapitres précédents de la journée

---

## Objectifs

À l'issue de ce chapitre, vous serez capable de :

- **Mesurer** le coût d'un chiffrement sur un traitement réel et le comparer à son
  bénéfice de protection.
- **Arbitrer** entre protection, performance et coût opérationnel, en justifiant le
  choix devant un interlocuteur non technique.
- **Conduire** la réponse à une fuite de clé : révoquer, évaluer, ré-chiffrer,
  notifier.

---

## I. Coûts et performances

### Ce que le chiffrement coûte réellement

Chiffrer demande des opérations mathématiques ; les faire tourner consomme du
processeur, de la mémoire, du temps. Mais le coût se répartit rarement là où on
l'imagine :

| Poste de coût | Ce qu'il représente | Ordre de grandeur |
| --- | --- | --- |
| Chiffrement / déchiffrement des données | Calcul pur | Faible : les processeurs récents chiffrent plusieurs gigaoctets par seconde, grâce à l'accélération matérielle |
| Poignée TLS | Négociation réseau + calcul | Modéré, surtout à la connexion ; amorti par la reprise de session |
| Appel à un coffre de clés (KMS) | Aller-retour réseau | Élevé s'il est fait par ligne lue — à mettre en cache |
| Déchiffrement applicatif par ligne | Calcul + aller-retour | Cumulatif sur de gros volumes |
| Stockage | Le chiffré est légèrement plus volumineux | Négligeable dans la plupart des cas |

La conclusion pratique : **le chiffrement n'est presque jamais le goulot
d'étranglement**. Ce qui coûte, c'est l'architecture qui l'entoure — un appel réseau
par enregistrement, une clé relue à chaque requête, une poignée de main refaite pour
chaque requête HTTP. Mesurer avant d'optimiser : un benchmark sur le traitement réel
vaut mieux que dix intuitions.

```python
import os
import timeit

from cryptography.fernet import Fernet

f = Fernet(Fernet.generate_key())
donnee = os.urandom(1024)

chiffrer = lambda: f.encrypt(donnee)
print("chiffrement :", timeit.timeit(chiffrer, number=1000), "s pour 1000 appels")
```

La mesure répond à « ce chiffrement ralentit-il mon traitement ? », jamais à « la
sécurité, ça coûte cher ? ». La bonne question reste : **protège-t-on proportionnellement
à la valeur des données ?**

> [!TIP]
> Avant de chiffrer une colonne, demandez quelle décision métier s'appuie dessus. Une
> donnée que personne ne lit ne mérite ni index ni déchiffrement systématique ; une
> donnée critique mérite le chiffrement, même si elle coûte quelques millisecondes.

---

## II. Sobriété

Chiffrer, c'est calculer, et calculer se paie. Deux leviers intéressent directement
l'arbitrage sécurité / performance : **réutiliser** — maintenir les connexions ouvertes
et la reprise de session TLS plutôt que renégocier à chaque requête — et **ne pas
stocker** ce qui ne sert pas, la minimisation du chapitre précédent étant aussi un gain
de calcul. Les leviers plus techniques (choix de l'algorithme selon le matériel,
rechiffrement évité par enveloppe) sont en fin de chapitre.

> [!WARNING]
> Une protection que les utilisateurs contournent ne protège rien. Un chiffrement qui
> rend un service inutilisable finit désactivé, avec l'approbation de tout le monde.
> La sobriété, ici, est un garde-fou : elle force à chiffrer **là où cela compte**, et
> à l'expliquer.

---

## III. Fuite de clé

### Le scénario

Une clé de chiffrement apparaît dans un dépôt public, un journal d'application, une
sauvegarde oubliée — ou elle est emportée par une personne qui quitte l'équipe. Peu
importe la voie : à partir de maintenant, **on suppose qu'elle est entre de mauvaises
mains**. Les quatre actions prioritaires s'enchaînent dans un ordre précis.

| Ordre | Action | Pourquoi maintenant |
| --- | --- | --- |
| 1 | **Révoquer** la clé : la rendre inutilisable, isoler les systèmes qui l'emploient | Chaque minute qui passe est une minute d'exploitation possible |
| 2 | **Évaluer** l'exposition : quels données, quels systèmes, sur quelle période, avec quels accès | On ne répare que ce que l'on sait touché — et on documente pour la suite |
| 3 | **Ré-chiffrer** les données avec une clé neuve, et remplacer la clé partout où elle sert | Les données déjà chiffrées avec la clé fuitée restent lisibles pour l'attaquant tant qu'elles ne sont pas rechiffrées |
| 4 | **Notifier** : en interne d'abord, puis les personnes concernées si des données personnelles sont en jeu | L'obligation légale ne dépend pas de la gravité ressentie, mais de la donnée touchée |

À ces quatre actions s'ajoute le travail de fond : comprendre **comment** la clé a
fuité, et fermer la voie — sinon la clé suivante suivra le même chemin.

> 🧪 **Atelier rapide (8 min)**
>
> Scénario : la clé de chiffrement de la base de réservations d'une plateforme de
> location de vélos est retrouvée dans l'historique d'un dépôt public, où elle traîne
> depuis plusieurs semaines. En huit minutes, écrivez vos **quatre actions
> prioritaires**, dans l'ordre, avec pour chacune : qui la conduit, et ce qui se passe
> si elle est faite dans le mauvais ordre. Comparez votre ordre au tableau ci-dessus.

### Ré-chiffrer sans tout casser

Le ré-chiffrement d'un gros volume est un chantier : il prend du temps, mobilise les
clés anciennes et nouvelles, et ne doit pas interrompre le service. La méthode est
celle des migrations de clés : **période à deux clés**, migration progressive des
enregistrements, vérification de la relecture, puis retrait de l'ancienne clé. Pendant
tout le chantier, la clé fuitée reste nécessaire pour déchiffrer l'ancien — d'où
l'importance de l'étape 1 : l'isoler, même si on ne peut pas encore s'en passer.

---

## IV. Aller plus loin

🔁 **Pour aller plus loin (facultatif)** — à lire si le créneau le permet, rien
d'obligatoire n'en dépend.

### Les quatre leviers d'éco-conception

À l'échelle d'une machine, chiffrer ne consomme rien ; à celle d'une plateforme qui
établit des millions de connexions par jour, ou d'un cluster dont chaque disque est
chiffré, cela compte. Sobriété et sécurité se rejoignent sur un principe : ne faire que
les calculs utiles. Du plus simple au plus structurant :

- **réutiliser** : maintenir les connexions ouvertes, activer la reprise de session
  TLS, plutôt que renégocier à chaque requête ;
- **choisir l'algorithme adapté au matériel** : AES tirant parti de l'accélération
  matérielle là où elle existe, ChaCha20 sur les appareils qui en sont dépourvus —
  même résultat de protection, moins de cycles perdus ;
- **éviter le rechiffrement inutile** : le chiffrement par enveloppe permet de tourner
  une clé maître sans rechiffrer des téraoctets de données ;
- **ne pas stocker** ce qui ne sert pas — la minimisation est aussi une mesure
  d'éco-conception.

### Et le quantique, dans tout ça ?

- **La cryptographie post-quantique** : les algorithmes d'échange de clés et de
  signature actuels (RSA, courbes elliptiques) reposent sur des problèmes qu'un
  ordinateur quantique suffisamment puissant résoudrait. Le NIST a publié en 2024 ses
  premiers standards post-quantiques (ML-KEM pour l'échange de clés, ML-DSA pour la
  signature), et les grands acteurs du web déploient déjà des échanges de clés
  **hybrides** : classique et post-quantique en parallèle, pour être sûr des deux.
- **« Récolter maintenant, déchiffrer plus tard »** : un attaquant qui enregistre
  aujourd'hui du trafic chiffré peut espérer le déchiffrer le jour où un ordinateur
  quantique existera. C'est pourquoi les données à longue durée de vie — santé,
  état civil, secrets industriels — méritent déjà des algorithmes résistants, ou une
  protection par confidentialité permanente.
- **Le coût de la transition** : remplacer les algorithmes partout prend des années —
  dans les cartes, les protocoles, les bases, les objets connectés. Inventorier ce qui
  utilise la cryptographie est le premier chantier, bien avant le remplacement lui-même.

---

## Points clés

- Le chiffrement est rarement le goulot d'étranglement : ce qui coûte, c'est
  l'architecture autour — appels réseau, poignées de main répétées, clés relues.
- **Mesurer** sur le traitement réel, puis arbitrer : la protection suit la valeur des
  données, pas l'inverse.
- La sobriété rejoint la sécurité : réutiliser les connexions, choisir l'algorithme
  adapté au matériel, éviter le rechiffrement inutile, ne pas stocker le superflu.
- Face à une fuite de clé : **révoquer, évaluer, ré-chiffrer, notifier** — dans cet
  ordre, puis fermer la voie par laquelle elle est partie.
- Un chiffrement se décide **au moment du choix**, pas après : la durée de vie attendue
  de la donnée fait partie du cahier des charges, au même titre que sa sensibilité.

---

[← Chapitre précédent](11-chiffrement-data.md) · [Sommaire](Readme.md) · [Chapitre suivant →](A-Glossaire-et-ressources.md)
