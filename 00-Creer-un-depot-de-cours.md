[Sommaire](Readme.md) · [Chapitre suivant →](01-Exemple%20de%20cours%20en%20un%20seul%20fichier.md)

# Créer le dépôt d'un cours

Ce dépôt ne s'édite pas pour écrire un cours : c'est un modèle. Chaque cours a
son propre dépôt, créé à partir de lui, qui en hérite tout l'outillage — les
workflows, les logos, le fichier de configuration — et le garde ensuite pour
lui seul. Les évolutions ultérieures du template ne s'y propagent pas.

## 1. Créer le dépôt

Sur la page GitHub du template, bouton **Use this template** → **Create a new
repository**, puis :

| Champ | Valeur |
|---|---|
| *Owner* | `DiginamicAcademy` |
| *Repository name* | le nom du cours, sans espace ni accent : `Angular-22` |
| *Visibility* | *Public* : le cours est remis aux apprenant·e·s |
| *Include all branches* | non coché |

Le nom compte plus qu'il n'y paraît : il désigne aussi la partie pédagogique du
cours, le dépôt privé `Angular-22-Peda` créé à la première version. Renommer le
cours après coup ferait créer une seconde partie pédagogique, sous le nouveau
nom.

Le dépôt créé part d'un historique neuf, sans version ni release : rien n'y est
publié tant qu'on ne l'a pas décidé. Il reste à le récupérer en local :

```bash
git clone https://github.com/DiginamicAcademy/Angular-22.git
cd Angular-22
```

## 2. Ce que contient le dépôt

```text
Readme.md                    page d'accueil, ouverture du support   → à réécrire
00-… à 05-….md               chapitres d'exemple (dont celui-ci)    → à remplacer
slides.md                    diapositives d'exemple                 → à remplacer ou supprimer
config.yaml                  déclaration du cours et crédits        → à remplir
assets/                      logos, puis les images du cours        → à compléter
.github/                     workflows et scripts                   → à garder tels quels
.gitattributes, .gitignore   conventions du dépôt                   → à garder tels quels
```

Ce qui est sous `.github/` fait tout le travail de production : on n'y touche
pas. Rien n'est à configurer dans le dépôt lui-même non plus : les secrets dont
les workflows ont besoin sont posés une fois pour toutes au niveau de
l'organisation (voir le [chapitre 05](05-Partie-pedagogique.md)).

## 3. Déclarer le cours dans `config.yaml`

Le fichier compte deux sections :

```yaml
course:
  name: "Angular — les fondamentaux"
  RNCP:
    - RNCP40573
  RNCPBloc:
    - RNCP40573BC06
  timing: 2

credits:
  authors:
    - name: Prénom Nom
      github:
        - compte-github
```

- **`course`** dit ce que vise le cours : intitulé, titres et blocs RNCP, durée
  en jours. Il alimente la partie pédagogique — [chapitre 05](05-Partie-pedagogique.md).
- **`credits`** dit qui signe le cours, et avec quels comptes GitHub ses
  auteurs commitent — aucune adresse électronique, le fichier est public. Il
  alimente la page de garde et les crédits du support —
  [chapitre 04](04-Auteurs-et-contributeurs.md).

Chaque push qui modifie `config.yaml` déclenche le workflow `Vérifier`, qui
signale les erreurs en annotations, sur la ligne en cause. Un fichier laissé
tel quel bloque la partie pédagogique : il est vérifié de nouveau à chaque
version, et les valeurs d'exemple du template (le titre `RNCP0000`) n'y passent
pas.

## 4. Faire place nette

**Le Readme.** Il sert de page d'accueil sur GitHub et ouvre le support PDF.
Son titre de niveau 1 est le nom du cours : il figure sur la page de garde et
nomme les fichiers produits — « Angular — les fondamentaux » donne
`Angular-les-fondamentaux-support-de-cours.pdf`. Le reste suit les conseils du
[Readme du template](Readme.md) : les objectifs pédagogiques, puis le parcours
en tableau. Le bloc du logo, en tête, se garde tel quel.

**Les chapitres.** Supprimer les fichiers `00-` à `05-` du template, puis écrire
ceux du cours à la racine, un fichier par chapitre : `01-installation.md`,
`02-composants.md`… Les règles de nommage et d'ordre sont au
[chapitre 02](02-Exemple%20de%20cours%20en%20multi-fichier.md). Les liens de
navigation entre chapitres — précédent, sommaire, suivant — ne s'écrivent pas :
ils sont posés au push. Un cours court peut aussi tenir dans le seul Readme, sur
le modèle du [chapitre 01](01-Exemple%20de%20cours%20en%20un%20seul%20fichier.md).

**Les diapositives.** Remplacer le contenu de `slides.md` en gardant son
en-tête, dont seul le titre `header` change — voir le
[chapitre 03](03-Slides.md). Ou supprimer le fichier : le cours n'aura alors pas
de diapositives, et le workflow saute les étapes correspondantes.

**Les images.** Les logos de `assets/` restent ; les images du cours s'y
ajoutent, désignées par leur chemin relatif (`assets/schema.png`).

## 5. Rédiger

Le cours s'écrit et se pousse au fil de l'eau. Un push ne produit aucun
document, mais deux workflows y réagissent :

- `Vérifier`, quand `config.yaml` change : il signale les erreurs, sans rien
  modifier ;
- `Navigation`, quand un chapitre change : il pose ou corrige les liens entre
  chapitres, et les commite sur la branche.

Ce second commit est à récupérer avant de continuer, sans quoi le push suivant
est refusé :

```bash
git pull --rebase
```

Pour voir le support sans publier de version : *Actions* → *Documents* → *Run
workflow*. Les documents produits sont dans l'artefact du run, et aucune
release n'est créée. Les diapositives, elles, se prévisualisent en local
(chapitre 03).

## 6. Publier une version

```bash
git tag v1.0
git push origin v1.0
```

À chaque tag `v*` :

1. le workflow `Documents` produit le support et les diapositives, et les
   attache à une release `v1.0`, en téléchargement permanent ;
2. il crée ou met à jour la partie pédagogique `Angular-22-Peda`, où une pull
   request propose `RNCP.yaml` et le syllabus, à relire avant fusion.

Les versions suivantes se numérotent de même : `v1.1` pour une correction,
`v2.0` pour une refonte. Un numéro déjà publié ne se réutilise pas.

## En résumé

- [ ] dépôt créé depuis le template, dans `DiginamicAcademy`, public
- [ ] `config.yaml` rempli : `course` et `credits`
- [ ] Readme réécrit, son titre est le nom du cours
- [ ] chapitres d'exemple remplacés, `slides.md` remplacé ou supprimé
- [ ] premier tag `v1.0` poussé

---

[Sommaire](Readme.md) · [Chapitre suivant →](01-Exemple%20de%20cours%20en%20un%20seul%20fichier.md)
