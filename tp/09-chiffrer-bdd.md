# Exercice — Chiffrer une colonne sensible

> 🟡 **À vous de jouer** · **Durée :** 45 min · **Lien :** chapitre [Données au repos](../09-donnees-au-repos.md)
> **Objectif :** rendre une colonne de coordonnées bancaires illisible dans la base,
> tout en restant capable de payer les salariés et de retrouver une fiche.

---

## Contexte

**Novéo RH** édite un logiciel de paie. Les coordonnées bancaires des salariés sont
stockées en clair dans la table `employes` de la base MySQL de production :

```sql
CREATE TABLE employes (
    id INT PRIMARY KEY,
    nom VARCHAR(64) NOT NULL,
    email VARCHAR(128) NOT NULL,
    iban VARCHAR(34) NOT NULL
);

INSERT INTO employes (id, nom, email, iban) VALUES
    (1, 'Claire Fontaine', 'claire.fontaine@noveo-rh.fr', 'FR76 3000 4008 2800 0123 4567 890'),
    (2, 'Marc Delaunay',   'marc.delaunay@noveo-rh.fr',   'FR14 2004 1010 0505 0001 3M02 606'),
    (3, 'Inès Berthier',   'ines.berthier@noveo-rh.fr',   'FR76 3000 6000 0112 3456 7890 189');
```

L'audit de conformité est formel : les IBAN doivent être **chiffrés au niveau
applicatif**. Le disque du serveur est déjà chiffré, mais la base est copiée tous les
soirs, exportée pour le support, et lue par plusieurs administrateurs. L'application
est écrite en Python ; la bibliothèque `cryptography` est disponible dans
l'environnement.

---

## Consignes

### Partie 1 — Choisir la stratégie

Répondez par écrit, en quelques phrases :

1. Pourquoi le chiffrement de disque ne suffit-il pas dans cette situation ?
2. Pourquoi chiffrer l'IBAN plutôt que le hacher — alors que le hachage est jugé
   « plus sûr » par une partie de l'équipe ?
3. Que faire des colonnes `nom` et `email`, qui sont aussi des données personnelles ?
   Donnez votre position et ses limites.

### Partie 2 — Chiffrer et déchiffrer

Écrivez un module Python qui :

1. lit la clé de chiffrement dans la variable d'environnement `CLE_CHIFFREMENT` —
   jamais dans le code ;
2. expose `chiffrer(valeur: str) -> bytes` et `dechiffrer(stocke: bytes) -> str` ;
3. chiffre les IBAN existants de la table (migration) et les stocke dans une colonne
   adaptée au format binaire ;
4. déchiffre une fiche à la lecture.

Testez votre module sur les trois lignes fournies : après migration, la base ne doit
plus contenir aucun IBAN lisible.

### Partie 3 — Traiter la clé

Décrivez en une dizaine de lignes :

1. où la clé est générée, où elle est stockée, qui y a accès ;
2. ce qui se passe concrètement si la clé est perdue ;
3. comment remplacer la clé par une nouvelle **sans interruption de service**.

### Partie 4 — Retrouver une fiche

La paie doit pouvoir retrouver un salarié à partir de son IBAN. Comme la colonne
chiffrée n'est plus interrogeable, ajoutez une colonne `iban_index` et une fonction de
recherche qui l'utilise. Expliquez en deux phrases ce que cette colonne révèle malgré
tout à qui lit la base.

---

## Attendu

- un module Python exécutable, avec les fonctions de chiffrement, de migration et de
  recherche ;
- un court rapport pour les parties 1 et 3 (une dizaine de lignes suffit) ;
- les réponses de la partie 4, reliées à votre implémentation.

---

## ✅ Auto-évaluation

Avant de passer au corrigé, vérifiez que :

- [ ] J'ai expliqué pourquoi le chiffrement de disque ne couvre pas les copies de la base.
- [ ] J'ai justifié le choix chiffrement / hachage pour l'IBAN, en m'appuyant sur le besoin de le **relire**.
- [ ] Ma clé vient d'une variable d'environnement et n'apparaît nulle part dans le code ni dans la base.
- [ ] Après migration, une requête `SELECT` sur la table ne montre aucun IBAN lisible.
- [ ] Ma fonction de recherche trouve la bonne fiche à partir d'un IBAN complet.
- [ ] J'ai écrit ce que révèle la colonne `iban_index`, et pourquoi c'est un compromis assumé.

---

## Pour aller plus loin

*Facultatif.* Mesurez le coût de votre solution : chronométrez la lecture de dix
mille fiches avec et sans chiffrement, puis la recherche par index. À partir de quel
ordre de grandeur le chiffrement applicatif devient-il un sujet de performance — et
que feriez-vous si ce seuil était atteint en production ?
