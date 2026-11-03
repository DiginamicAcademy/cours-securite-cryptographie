---
marp: true
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
header: "Template de cours public"
footer: "![height:20px](https://raw.githubusercontent.com/DiginamicInternal/PublicAssets/refs/heads/main/Logo-diginamic-color-blk.png)"
---

# Template de cours public

### Rédiger, découper et projeter un cours

<!-- Diapositive d'ouverture. Ce commentaire est une note de présentateur : il
     n'apparaît pas à la projection, mais reste visible dans le PPTX. -->

---

## 🎯 Objectifs pédagogiques

À la fin de ce cours, vous saurez :

1. Créer le dépôt d'un cours à partir du template
2. Écrire un cours mono fichier efficient
3. Comprendre les parties importantes à y mettre
4. Découper un cours long en chapitres
5. Produire les diapositives à partir du même dépôt
6. Déclarer le cours, ses auteurs, et le publier

---

## Le parcours

| #  | Étape                                      | Durée  |
|----|--------------------------------------------|--------|
| 00 | Créer le dépôt d'un cours                  | 15 min |
| 01 | Un cours en un seul fichier                | 20 min |
| 02 | Organiser un cours en plusieurs fichiers   | 20 min |
| 03 | Faire les diapositives avec Marp           | 20 min |
| 04 | Créditer l'auteur et les contributeurs     | 10 min |
| 05 | Déclarer le cours et sa partie pédagogique | 15 min |

Chaque étape est un fichier. On les suit **dans l'ordre**.

---

## Deux documents, une source

- Le **support de cours** se lit : Readme + chapitres, assemblés en un PDF relié
- Les **diapositives** se projettent : un seul `slides.md`
- Tout part du même dépôt, tout est produit par le même workflow

> On n'écrit pas deux fois la même chose : on écrit deux choses différentes.

---

# 0. Créer le dépôt d'un cours

---

## Un dépôt par cours

1. **Use this template** → *Create a new repository* : `DiginamicAcademy`, nom du cours sans espace ni accent (`Angular-22`), *Public*
2. Remplir **`config.yaml`** : sections `course` et `credits`
3. Réécrire le **Readme** — son titre est le nom du cours
4. Remplacer les **chapitres** d'exemple et **`slides.md`**
5. Rédiger, pousser, puis **publier** : `git tag v1.0`

> Le template ne s'édite pas : il se copie.

---

## `config.yaml`

```yaml
course:            # ce que vise le cours → partie pédagogique
  name: "Angular — les fondamentaux"
  RNCP: [RNCP40573]
  RNCPBloc: [RNCP40573BC06]
  timing: 2
credits:           # qui le signe → page de garde, crédits du PDF
  authors:
    - name: Prénom Nom
      github: [compte-github]
```

Vérifié à chaque push qui le modifie.

---

# 1. Un cours en un seul fichier

---

## Les parties conseillées

1. **Objectifs pédagogiques** — ce que le lecteur saura faire à la fin
2. **Pourquoi ce cours ?** — l'ancrage dans un corpus plus large *(optionnel)*
3. **Sommaire** — utile dès que le document s'allonge *(optionnel)*
4. Les **parties** numérotées, séparées par un filet
5. Les **annexes** — ressources, références, liens

---

## Les schémas

Autant que faire se peut, les schémas se font en **Mermaid** : du texte,
versionnable, relisible en revue.

```markdown
flowchart TD
    A[Lorem ipsum] --> B{Dolor sit amet ?}
    B -->|Consectetur| C[Adipiscing elit]
    B -->|Sed do| D[Eiusmod tempor]
```

> ⚠️ **Attention :** Mermaid se rend assez mal dans les slides, préférez une copie d'écran du rendu Mermaid insérée comme une image dans les slides.

---

## Les images

Deux sources acceptables, dans l'ordre de préférence :

- un dossier `assets` du dépôt, référencé par son **chemin relatif**
- une image déjà publique sur un CDN pérenne, référencée par son URL

```markdown
![texte de remplacement](assets/mon-image.png)
```

Pour centrer, encadrer l'image d'une balise `<center>` et de lignes vides.

---

## Le code

Toujours préciser le langage : la coloration syntaxique se joue là.

```javascript
// exemple de javascript
console.log('Je suis coloré !');
```

```yaml
Services:
    Lorem:
        ipsum: dolor
```

---

## Les parties escamotables

Pour livrer un exercice sans livrer sa correction :

```markdown
<details>
<summary>Voir la correction</summary>

1. réponse

</details>
```

Réservé au support : à la projection, rien à replier.

---

# 2. Plusieurs fichiers

---

## Ce qui fait un chapitre

Trois conditions, toutes vérifiées par le workflow :

1. le fichier est **à la racine** du dépôt
2. son nom commence par un **numéro** ou une **lettre**, suivi d'un séparateur
3. ce n'est pas le Readme

Tout le reste est ignoré : `slides.md`, les sous-dossiers, les brouillons.

---

## L'ordre des chapitres

Tri **naturel** : `10-` vient après `9-`, et non avant.

```text
Readme.md                  ← page d'ouverture
01-installation.md         ← chapitre 1
10-agregations.md          ← chapitre 3, et non avant le 02
A-glossaire.md             ← annexe, après les numéros
ressources/                ← ignoré
```

Numéroter de dix en dix laisse la place d'insérer sans tout renommer.

---

## Le titre et l'intercalaire

- Chaque chapitre s'ouvre sur un **titre de niveau 1**
- Ce titre est reporté sur une **page d'intercalaire** pleine page
- Tout ce qui le précède **disparaît du support**

> Les liens précédent · sommaire · suivant sont posés en tête et en pied
> de chaque chapitre, à chaque push : on ne les écrit pas.

---

## Les liens entre chapitres

Un lien vers un fichier voisin devient un lien interne dans le PDF.

- encoder les espaces du nom de fichier (`%20`)
- le fragment est abandonné : on atterrit sur l'intercalaire
- une cible absente du support est réduite à son libellé
- aucune cible `.md` relative ne doit subsister, **blocs de code compris**

---

# 3. Les diapositives

---

## L'en-tête du diaporama

```yaml
marp: true
theme: default
style: |
  section {background-color: #121114}
paginate: true
header: "Titre du cours"
footer: "![height:20px](URL du logo)"
```

`style` accueille les couleurs des slides. Le générateur produit des slides à fond blanc et fond foncé automatiquement, aux formats PDF et PPTX.

---

## Découper et régler

- Une ligne de **trois tirets** isolée sépare deux diapositives
- Un commentaire devient une note de présentateur dans la conversion PPTX. Exemple : 

```markdown
<!-- Ceci est un commentaire de présentation visible uniquement dans un PPTX coté présentateur -->
```

---

## Sombre et clair

La version claire est dérivée automatiquement :

1. le bloc `style:` est **remplacé** par un style minimal — le thème reprend ses couleurs
2. le logo `…-blk.png` est **remplacé** par sa version couleur `.svg`

Quatre fichiers à l'arrivée : PDF et PPTX, sombre et clair.

> Merci de ne pas modifier le style et le logo 😉

---

## Ce que les diapositives ne font pas

- **Mermaid n'est pas rendu** : le schéma reste dans le support, ou s'exporte
  en image
- **Pas de bloc escamotable** : rien à replier sur un écran

---

## Prévisualiser pendant l'écriture

```bash
npx @marp-team/marp-cli@4.5.1 slides.md --preview --watch
```

Ou utiliser l'extension officielle [Marp for VS Code](https://marketplace.visualstudio.com/items?itemName=marp-team.marp-vscode) — `marp-team.marp-vscode`.

---

## Publier

Les documents ne sont produits que sur un **tag de version** :

```bash
git tag v1.0
git push origin v1.0
```

- artefact du run, conservé 90 jours
- release GitHub, en téléchargement permanent
- partie pédagogique (`<cours>-Peda`) créée ou mise à jour, syllabus régénéré
- `workflow_dispatch` pour un essai à blanc, sans release

---

## Documentation Marp

- [marp.app](https://marp.app/) — le site du projet, point d'entrée
- [marpit.marp.app/markdown](https://marpit.marp.app/markdown) — la syntaxe Markdown de Marp
- [marpit.marp.app/directives](https://marpit.marp.app/directives) — toutes les directives, globales et locales
- [marpit.marp.app/image-syntax](https://marpit.marp.app/image-syntax) — dimensionnement et images de fond
- [github.com/marp-team/marp-cli](https://github.com/marp-team/marp-cli) — les options de la ligne de commande

---

# Des questions ?

### Le dépôt reste la référence

Pour toute question [adressez-vous au service pédagogie](mailto:pedagogie@diginamic.fr)
