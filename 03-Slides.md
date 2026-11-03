[← Chapitre précédent](02-Exemple%20de%20cours%20en%20multi-fichier.md) · [Sommaire](Readme.md) · [Chapitre suivant →](04-Auteurs-et-contributeurs.md)

# Faire les diapositives avec Marp

Le support de cours se lit ; les diapositives se projettent. Ce sont deux
documents différents, et le dépôt les tient séparés : les chapitres d'un côté,
un unique `slides.md` de l'autre. Marp transforme ce Markdown en diaporama.

Les diapositives sont **facultatives**. Sans `slides.md` à la racine, le
workflow saute simplement les étapes correspondantes et ne produit que le
support.

## Quatre fichiers à l'arrivée

À partir du seul `slides.md`, le workflow produit :

| Fichier | Usage |
|---|---|
| `…-slides.pdf` | projection, thème sombre |
| `…-slides.pptx` | reprise dans PowerPoint, thème sombre |
| `…-slides-white.pdf` | impression et photocopie |
| `…-slides-white.pptx` | reprise dans PowerPoint, fond clair |

La version claire n'est pas un second fichier à maintenir : elle est dérivée
automatiquement de la première. Voir « Le duo sombre / clair » plus bas.

## L'en-tête du fichier

Tout commence par un bloc de configuration, entre deux lignes de tirets, en tête
de `slides.md` :

```yaml
marp: true
theme: default
style: |
  section {background-color: #121114}
  h1,h2,h3 {color: #8393f0}
paginate: true
header: "Titre du cours"
footer: "![height:20px](URL du logo)"
```

- `marp: true` active le rendu ; sans lui, rien ne se passe.
- `theme` choisit le thème de base : `default`, `gaia` ou `uncover`.
- `style` ajoute du CSS par-dessus le thème, pour l'essentiel les couleurs du
  fond sombre. **C'est ce bloc que la version claire remplace** — voir « Le duo
  sombre / clair » plus bas.
- `paginate: true` numérote les diapositives.
- `header` et `footer` se répètent sur chaque diapositive.

## Séparer les diapositives

Une ligne de trois tirets isolée termine une diapositive et ouvre la suivante.

C'est le principal écart avec la rédaction d'un chapitre : dans le support, une
telle ligne trace un filet horizontal entre deux sections ; dans `slides.md`,
elle change de page. Une ligne de séparation esthétique copiée depuis un
chapitre coupe donc la diapositive en deux.

## Les images

La syntaxe d'image accepte des mots-clés de dimensionnement et de placement dans
son texte de remplacement :

- `![height:200px](…)` et `![width:60%](…)` contraignent la taille ;
- `![bg right](…)` colle l'image en fond sur la moitié droite, le texte occupant
  l'autre moitié ;
- `![bg](…)` couvre toute la diapositive.

Comme dans le support, une image du dépôt se désigne par son chemin relatif
(`assets/mon-image.png`) : le workflow autorise Marp à lire les fichiers du
dépôt. Une différence toutefois : aucun garde-fou ne vérifie les images des
diapositives, et une image introuvable y laisse un cadre vide sans faire échouer
la construction. La prévisualisation, plus bas, suffit à s'en apercevoir.

## Le duo sombre / clair

Ne pas modifier la partie style de l'en-tête, elle est modifiée à la volée pour produire simultanément une version claire et une version "normale".

La version claire est dérivée de `slides.md` en deux temps :

1. le bloc `style:` de l'en-tête est **remplacé** par un style minimal, défini
   dans le workflow sous le nom `WHITE_STYLE` : le thème reprend ses couleurs
   d'origine, et seules les règles indépendantes du fond — la taille de police
   des tableaux, l'encadré des citations — sont conservées ;
2. le logo `…-blk.png` du pied de page, fait pour le fond sombre, est
   **remplacé** par sa version couleur `.svg`.

Si une règle de mise en page (taille de police, marge…) doit malgré tout être
ajoutée au bloc `style:`, elle est à reporter aussi dans `WHITE_STYLE` : la
version claire l'ignorerait sinon.

## Ce que les diapositives ne savent pas faire

**Les diagrammes Mermaid ne sont pas rendus.** Le rendu Mermaid n'est appliqué
qu'au support de cours ; dans un diaporama, un tel bloc ressortirait en bloc de
code brut. Un schéma destiné à la projection s'insère donc comme une image — une
copie d'écran du rendu Mermaid fait l'affaire — et le diagramme lui-même reste
dans le chapitre.

**Les blocs escamotables non plus.** Une correction masquée n'a pas de sens sur
un écran de projection : elle appartient au support.

## Prévisualiser avant de publier

Les documents ne sont construits que sur un tag. Pour voir le rendu pendant
qu'on écrit, deux options :

- l'extension officielle [**Marp for VS Code**](https://marketplace.visualstudio.com/items?itemName=marp-team.marp-vscode), qui affiche le diaporama à côté du texte — identifiant `marp-team.marp-vscode`, installable aussi en ligne de commande avec `code --install-extension marp-team.marp-vscode` ;
- la ligne de commande, qui ouvre un aperçu se rafraîchissant à chaque
  enregistrement :

```bash
npx @marp-team/marp-cli@4.5.1 slides.md --preview --watch
```

La version est celle qu'emploie le workflow : prévisualiser avec la même évite
les surprises entre l'écran et le fichier publié, même si l'extension VSCode est très fidèle.

## La documentation de Marp

Ce chapitre couvre ce dont on se sert ici ; le reste est dans la documentation
du projet, courte et bien faite :

- [marp.app](https://marp.app/) — le site du projet, point d'entrée ;
- [la syntaxe Markdown](https://marpit.marp.app/markdown) — ce que Marp ajoute
  au Markdown ordinaire ;
- [les directives](https://marpit.marp.app/directives) — la liste complète,
  globales et locales ;
- [la syntaxe d'image](https://marpit.marp.app/image-syntax) — dimensionnement,
  images de fond, filtres ;
- [les thèmes](https://marpit.marp.app/theme-css) — pour aller au-delà du bloc
  `style:` ;
- [marp-cli](https://github.com/marp-team/marp-cli) — les options de la ligne de
  commande, dont celles qu'emploie le workflow.

---

[← Chapitre précédent](02-Exemple%20de%20cours%20en%20multi-fichier.md) · [Sommaire](Readme.md) · [Chapitre suivant →](04-Auteurs-et-contributeurs.md)
