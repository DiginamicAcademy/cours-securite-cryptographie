[← Chapitre précédent](01-Exemple%20de%20cours%20en%20un%20seul%20fichier.md) · [Sommaire](Readme.md) · [Chapitre suivant →](03-Slides.md)

# Organiser un cours en plusieurs fichiers

Un cours court tient dans un seul fichier. Passé quelques dizaines de pages, il
devient pénible à relire, à corriger à plusieurs et à parcourir sur GitHub. La
solution est de le découper en chapitres : un fichier par chapitre, à la racine
du dépôt. Le workflow `Documents` les recolle en un seul PDF, dans l'ordre, sans
qu'aucun sommaire n'ait à être maintenu à la main.

## Ce qui fait un chapitre

Trois conditions, toutes vérifiées par le workflow :

1. le fichier est **à la racine** du dépôt ;
2. son nom commence par un **numéro** (`01-`, `1-`) ou par une **lettre**
   (`A-`, `B-`), suivi d'un séparateur : tiret, point, souligné ou espace ;
3. ce n'est pas le Readme.

Tout le reste est ignoré. `Readme.md` ouvre le document, `slides.md` alimente
les diapositives, et les fichiers rangés dans des sous-dossiers ne sont jamais
repris — c'est là qu'on met les brouillons, les corrigés et les ressources.

Le séparateur porte à lui seul la distinction entre un chapitre et un fichier
ordinaire. Revers de la médaille : un `a-propos.md` posé à la racine serait pris
pour le chapitre « a ». La liste retenue est journalisée au début de chaque run,
sous « Chapitres : » — c'est le premier endroit à regarder si le PDF contient
autre chose que ce qu'on croyait.

## L'ordre des chapitres

L'ordre est celui des noms de fichiers, trié en version naturelle : `10-` vient
après `9-`, et non avant comme le voudrait l'ordre alphabétique. Les numéros
passent avant les lettres si les deux conventions cohabitent, ce qui permet de
réserver les lettres aux annexes.

```
Readme.md                       ← page d'ouverture du support
01-installation.md              ← chapitre 1
02-premieres-requetes.md        ← chapitre 2
10-agregations.md               ← chapitre 3, et non avant le 02
A-glossaire.md                  ← annexe, après tous les numéros
slides.md                       ← diapositives, hors du support
ressources/                     ← ignoré par le workflow
  exercices.md
  corriges.md
```

Numéroter de dix en dix (`10-`, `20-`, `30-`) laisse la place d'insérer un
chapitre sans renommer les suivants — et sans casser les liens qui pointent
vers eux.

## Le titre de niveau 1 et l'intercalaire

Chaque chapitre doit s'ouvrir sur un titre de niveau 1. Ce titre ne s'imprime
pas tel quel : il est reporté sur une page d'intercalaire pleine page, qui
sépare le chapitre du précédent dans le PDF.

Conséquence directe, **tout ce qui précède ce titre disparaît du support**. La
ligne de navigation en tête de ce fichier — celle qui renvoie au chapitre
précédent et au sommaire — n'existe que pour la lecture sur GitHub. Elle est
utile là où le lecteur navigue de fichier en fichier, elle n'a aucun sens dans
un document relié, et elle est retirée à l'assemblage.

Un chapitre sans titre de niveau 1 reste publié, mais l'intercalaire porte alors
le nom du fichier. C'est un filet de sécurité, pas une manière de faire.

## La navigation entre chapitres

Chaque chapitre porte, en tête et en pied, une ligne de navigation : un lien vers
le chapitre précédent, un vers le sommaire — le Readme —, un vers le chapitre
suivant. Le premier chapitre n'a pas de lien « précédent », le dernier pas de
lien « suivant ».

```text
[← Chapitre précédent](…) · [Sommaire](…) · [Chapitre suivant →](…)
```

Ces lignes ne s'écrivent pas à la main. À chaque push qui touche un `.md` de la
racine, le workflow `Navigation` ajoute celles qui manquent et remet en forme
celles qui existent : un chapitre inséré, renommé ou supprimé, et les liens de
ses voisins suivent. Il commite le résultat sur la branche poussée, sans toucher
au reste des fichiers.

La branche distante prend alors un commit d'avance sur le poste de l'auteur. Le
récupérer avant de continuer évite un push refusé :

```bash
git pull --rebase
```

Dans le support, la ligne du haut disparaît avec tout ce qui précède le titre.
Celle du bas reste : ses liens y deviennent des liens internes, comme tous les
liens entre chapitres.

Le même script se lance en local, `python3 .github/scripts/navigation.py` ;
avec `--verifier`, il signale les chapitres à reprendre sans rien modifier.

## Les liens entre chapitres

Un lien Markdown ordinaire vers un fichier voisin — cible `10-agregations.md`,
libellé libre — fonctionne des deux côtés : GitHub l'ouvre comme un lien de
fichier, et le workflow le réécrit en lien interne vers l'intercalaire du
chapitre visé, cliquable dans le PDF. La ligne de navigation en fin de page en
est un exemple réel.

Quatre points à connaître :

- Le nom de fichier doit être **encodé** s'il contient des espaces (`%20`) :
  un lien dont la cible contient une espace n'est pas reconnu.
- Le **fragment est abandonné** : un lien vers une ancre de section atterrit sur
  l'intercalaire du chapitre, seule ancre garantie dans le PDF.
- Un lien vers un fichier **absent du support** — un corrigé rangé dans un
  sous-dossier, par exemple — est retiré : il ne reste que son libellé, en texte
  simple. Le nombre de liens convertis et retirés est journalisé.
- Après réécriture, plus aucune cible relative en `.md` ne doit subsister, **y
  compris dans un bloc de code** : le garde-fou lit le document entier, et un
  exemple de lien montré dans une clôture ferait échouer la construction. C'est
  la raison pour laquelle cette section décrit la syntaxe au lieu de
  l'illustrer. Une URL absolue vers un `.md` — le fichier d'un autre dépôt, sur
  GitHub — reste un lien web valide et n'est pas concernée.

## Le chemin d'un chapitre jusqu'au PDF

```mermaid
flowchart TD
    A[Fichiers .md à la racine] --> B{"Nom en 01- ou A- ?"}
    B -->|Non| Z[Ignoré]
    B -->|Oui| C[Inventaire, tri naturel]
    C --> D[Titre H1 extrait]
    D --> E[Intercalaire + corps du chapitre]
    E --> F[Assemblage après le Readme]
    F --> G[Liens .md convertis en liens internes]
    G --> H[Images locales recopiées]
    H --> I[Diagrammes mermaid rendus en SVG]
    I --> J[(PDF publié dans la release)]
```

## Deux pièges de fin

**Les images.** Une image du dépôt se désigne par son chemin relatif à la racine
(`assets/schema.png`), comme on l'écrit pour GitHub : le workflow la recopie
dans le dossier de construction avant le rendu. Une image introuvable fait
échouer la construction plutôt que de laisser un cadre vide dans le PDF — le
journal donne le chemin en cause. Une image citée dans un bloc de code ou entre
accents graves est un exemple, pas une illustration : elle n'est pas recherchée.

**Le déclenchement.** Les documents ne sont produits que sur un tag `v*`. Pour
vérifier le rendu d'un découpage sans consommer un numéro de version, lancer le
workflow à la main : il produit l'artefact sans créer de release.

---

[← Chapitre précédent](01-Exemple%20de%20cours%20en%20un%20seul%20fichier.md) · [Sommaire](Readme.md) · [Chapitre suivant →](03-Slides.md)
