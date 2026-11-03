<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/Logo-diginamic-color-blk.png">
    <img src="assets/Logo-diginamic-color.svg" alt="Diginamic Formation" width="358">
  </picture>
</p>

# Template de cours public

Ce dépôt est le modèle des cours Diginamic. On n'y rédige pas de cours : on
s'en sert pour créer le dépôt de chaque cours, qui hérite de tout l'outillage :

- le **support de cours** en PDF, assemblé à partir du Readme et des chapitres ;
- les **diapositives**, en PDF et PPTX, sur fond sombre et sur fond clair ;
- une **release** GitHub par version, qui porte ces documents ;
- la **partie pédagogique** du cours : un dépôt privé `<cours>-Peda`, où sont
  produits la déclaration RNCP et le syllabus.

Les fichiers du template servent à la fois de documentation et d'exemple : ce
Readme montre comment ouvrir un cours, les chapitres comment les écrire.

## 1. Créer le dépôt d'un cours

1. **Créer le dépôt** : sur GitHub, *Use this template* → *Create a new
   repository*, propriétaire `DiginamicAcademy`, nom du cours sans espace ni
   accent (`Angular-22`), visibilité *Public*.
2. **Déclarer le cours** dans `config.yaml` : ce qu'il vise (`course`) et qui
   le signe (`credits`).
3. **Faire place nette** : réécrire ce Readme, remplacer les chapitres `00-` à
   `05-` et `slides.md` par ceux du cours.
4. **Rédiger**, en poussant au fil de l'eau.
5. **Publier une version** : `git tag v1.0 && git push origin v1.0`.

Chaque étape est détaillée au [chapitre 00](00-Creer-un-depot-de-cours.md).

## 2. Comment rédiger le cours ?

### 2.1 Parties conseillées

Commencer par une section "Objectifs pédagogiques" rappelant / dévoilant les objectifs du cours.

Continuer avec un phasage : quelles sont les étapes du cours ?

## 3. Le parcours

Chaque étape est un fichier. On les suit **dans l'ordre** : chacune s'appuie sur la base laissée par la précédente.

| # | Étape | Durée | Ce qu'on y fait |
|---|---|---|---|
| [00](00-Creer-un-depot-de-cours.md) | Créer le dépôt d'un cours | 15 min | le bouton *Use this template*, ce qu'on garde et ce qu'on remplace, la première version |
| [01](01-Exemple%20de%20cours%20en%20un%20seul%20fichier.md) | Un cours en un seul fichier | 20 min | la structure type d'un chapitre : objectifs, schémas Mermaid, images, code, parties escamotables |
| [02](02-Exemple%20de%20cours%20en%20multi-fichier.md) | Organiser un cours en plusieurs fichiers | 20 min | nommage et ordre des chapitres, intercalaires, liens entre fichiers, pièges de génération |
| [03](03-Slides.md) | Faire les diapositives avec Marp | 20 min | en-tête du diaporama, séparation des diapositives, images, duo sombre / clair, prévisualisation |
| [04](04-Auteurs-et-contributeurs.md) | Créditer l'auteur et les contributeurs | 10 min | la section `credits` de `config.yaml`, la liste tirée de l'historique git, la section « Crédits » |
| [05](05-Partie-pedagogique.md) | Déclarer le cours et sa partie pédagogique | 15 min | la section `course` de `config.yaml`, sa vérification, le dépôt privé `-Peda` créé et régénéré à chaque version |

Ce tableau est le sommaire du cours : il tient lieu de plan et donne au lecteur une idée du temps à prévoir. Les liens pointent vers les fichiers du dépôt ; à la génération du support, ils deviennent des liens internes vers le chapitre correspondant.
