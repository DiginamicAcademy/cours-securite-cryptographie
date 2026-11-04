# Exercice — Chiffrer un fichier avec AES-GCM

> **Nature :** exercice · **Durée :** 45 min · **Notion :** chiffrement symétrique authentifié (AES-GCM) — chiffrer un fichier, détecter une altération, échanger la clé
> **Lien :** chapitre [Le chiffrement asymétrique](../03-chiffrement-asymetrique.md)
> **Objectif :** chiffrer puis déchiffrer un fichier de données avec une bibliothèque
> crypto standard, et vérifier que toute altération du chiffré est détectée.

---

## Contexte

**VéloCité**, un opérateur de vélos en libre-service, confie à un laboratoire
universitaire un extrait de ses données de location pour une étude de mobilité. Le
fichier contient des données personnelles (dates, durées, habitudes de déplacement) :
il doit être chiffré **avant** de quitter l'infrastructure de l'opérateur, et seul le
laboratoire doit pouvoir le lire.

L'équipe data a retenu le mode GCM (Galois/Counter Mode) du standard AES (Advanced
Encryption Standard) : il chiffre et authentifie en une seule passe. La bibliothèque
retenue est `cryptography`, la référence en Python — elle s'installe avec
`pip install cryptography`.

Voici le fichier à protéger, `export-locations.csv` :

```csv
date,station,duree_min,abonne
2026-10-05,Gare Centrale,12,1
2026-10-05,Place du Marche,7,0
2026-10-06,Universite,23,1
```

Les repères de l'équipe, à respecter à la lettre :

- la clé de chiffrement fait **32 octets** (soit **256 bits**), tirés au hasard ;
- le **nonce** (valeur de départ unique) fait **12 octets** (soit **96 bits**), tirés
  au hasard, et il est **préfixé** au fichier chiffré pour que le déchiffrement
  retrouve tout seul ;
- le nonce ne doit **jamais** être réutilisé avec la même clé.

---

## Consignes

### Partie 1 — Générer la clé et le nonce

1. Écrivez un script `chiffrer.py` qui génère la clé et le nonce avec le module
   `secrets` (et non `random`, qui est prévisible).
2. Affichez leur longueur en octets pour vérifier qu'elle correspond aux repères
   ci-dessus.

### Partie 2 — Chiffrer le fichier

1. Lisez `export-locations.csv` **en binaire** (`read_bytes`).
2. Chiffrez-le avec la classe AESGCM (AES en mode GCM), sans donnée additionnelle.
3. Écrivez le résultat dans `export-locations.csv.chiffre`, sous la forme
   `nonce + chiffré` (le nonce d'abord, puis le corps chiffré).
4. Constatez que le fichier produit est illisible : ouvrez-le, il ne doit ressembler à
   rien de lisible.

### Partie 3 — Déchiffrer et vérifier

1. Écrivez `dechiffrer.py` : il lit le fichier chiffré, sépare le nonce du corps,
   déchiffre, et réécrit `export-locations-relu.csv`.
2. Vérifiez que `export-locations-relu.csv` est **identique octet à octet** au fichier
   d'origine.

### Partie 4 — Détecter une altération

1. Modifiez **un seul octet** du corps chiffré dans `export-locations.csv.chiffre`, par
   exemple en lui ajoutant 1.
2. Relancez `dechiffrer.py` : que se passe-t-il ?
3. Notez le nom de l'exception levée, et expliquez en deux phrases pourquoi ce
   comportement est un avantage et non un défaut.

### Partie 5 — Poser la question de la clé

Le laboratoire est distant : comment lui transmettre la clé de chiffrement sans la
livrer à quiconque écouterait le canal ? Répondez en trois phrases maximum, en
utilisant ce que le chapitre sur le chiffrement asymétrique décrit.

---

## Attendu

- `chiffrer.py` et `dechiffrer.py`, exécutables de bout en bout.
- Le fichier chiffré produit, et le fichier relu identique à l'original.
- Le compte rendu de la partie 4 (exception observée + explication).
- La réponse courte de la partie 5.

---

## Vérification

Vous avez réussi quand :

- `export-locations.csv.chiffre` commence bien par le nonce, et son contenu est
  illisible ;
- `export-locations-relu.csv` est identique au fichier d'origine ;
- une altération d'un seul octet fait échouer le déchiffrement au lieu de produire un
  fichier corrompu ;
- votre réponse de la partie 5 ne propose **pas** d'envoyer la clé en clair.

---

## ✅ Auto-évaluation

Avant de passer au corrigé, vérifiez que :

- [ ] Ma clé et mon nonce sont tirés avec `secrets`, et leur longueur est vérifiée.
- [ ] Mon fichier chiffré contient le nonce en tête, puis le corps chiffré.
- [ ] Le déchiffrement restitue un fichier identique octet à octet.
- [ ] J'ai observé le refus de déchiffrer un contenu altéré et je sais l'expliquer.
- [ ] J'ai décrit une transmission de clé qui ne passe pas par un canal en clair.

---

## Pour aller plus loin

*Facultatif.* Plutôt que de tirer la clé au hasard, dérivez-la d'une phrase secrète
avec PBKDF2, puis vérifiez que deux phrases secrètes différentes produisent deux clés
différentes. Notez ce que cela change si la phrase secrète est faible.
