[← Chapitre précédent](04-Auteurs-et-contributeurs.md) · [Sommaire](Readme.md)

# Déclarer le cours et sa partie pédagogique

Un cours a deux faces. Celle-ci, publique, est remise aux apprenant·e·s : le
support, les diapositives, les TP. L'autre, la **partie pédagogique**, reste
interne : la déclaration du module (titres RNCP visés, compétences couvertes),
son syllabus et, à terme, les évaluations et leurs corrigés. Elle vit dans un
dépôt privé distinct, que le cours crée et alimente lui-même à chaque version.

| Dépôt | Visibilité | Contenu |
|---|---|---|
| `Angular-22` | public | le cours, et `config.yaml` qui le déclare |
| `Angular-22-Peda` | privé | `RNCP.yaml`, `syllabus.md`, évaluations… |

Le nom du dépôt privé se déduit de celui du cours : on ajoute `-Peda`.

## La section `course` de `config.yaml`

À la racine du cours, `config.yaml` déclare ce que le cours vise, dans sa
section `course`. Tout ce que produit la partie pédagogique en découle. L'autre
section du fichier, `credits`, ne sert qu'au support de cours : elle est décrite
au [chapitre 04](04-Auteurs-et-contributeurs.md).

```yaml
course:
  name: "Angular — les fondamentaux"
  RNCP:
    - RNCP40573
  RNCPBloc:
    - RNCP40573BC06
  timing: 2
```

| Champ | Rôle |
|---|---|
| `name` | intitulé du module, repris tel quel dans le syllabus |
| `RNCP` | titres RNCP visés, par leur code de fiche France Compétences |
| `RNCPBloc` | blocs optionnels visés, au format `<titre><bloc>` ; la liste peut être vide |
| `timing` | durée prévue, en jours |

**Un bloc s'écrit d'un seul tenant**, le titre suivi du bloc : `RNCP40573BC06`,
tel que la fiche France Compétences le présente. Le titre doit aussi figurer
dans `RNCP`.

Le dépôt du cours, lui, n'a pas à être déclaré : c'est celui-ci.

## La vérification à chaque push

Dès qu'un push modifie `config.yaml`, le workflow `Vérifier` le contrôle : le
fichier se lit, les champs sont présents et bien formés, chaque bloc relève d'un
titre déclaré, chaque fiche et chaque bloc existent sur France Compétences — et
la section `credits`, si elle est là, a la forme attendue et cite des comptes
GitHub qui existent. Les erreurs
s'affichent en annotations, sur la ligne en cause.

Si France Compétences ne répond pas, ce n'est qu'un avertissement : une panne du
site ne bloque pas le travail sur le cours.

Le template n'est pas vérifié, ses valeurs n'étant que des exemples. Un cours
fraîchement créé, lui, échoue tant que `config.yaml` n'a pas été rempli : c'est
voulu.

Le même contrôle se lance en local, PyYAML installé :

```bash
python3 .github/scripts/verifier.py
```

## Ce qui se passe à chaque version

Une version se publie en poussant un tag :

```bash
git tag v1.0
git push origin v1.0
```

Le workflow `Documents` produit les supports et les attache à une release. Une
fois la release publiée, il passe à la partie pédagogique :

1. `config.yaml` est vérifié à nouveau ;
2. le dépôt `<cours>-Peda` est créé à partir du template
   [Template-Cours-Private](https://github.com/DiginamicAcademy/Template-Cours-Private),
   s'il n'existe pas encore ;
3. la section `course` de `config.yaml` y est recopiée, complétée de l'adresse
   du cours ;
4. le workflow « Régénérer les documents » y est lancé **sur cette version** :
   Claude lit le cours tel qu'il est au tag, écrit `RNCP.yaml` et
   `syllabus.md`, et une pull request est ouverte dans le dépôt privé.

Rien n'arrive sur la branche principale du dépôt privé sans relecture : le
syllabus est remis aux apprenant·e·s. Le résumé du run donne le lien vers la
régénération.

Deux conséquences pratiques :

- **`config.yaml` se modifie ici, jamais dans le dépôt privé** : la copie y est
  remplacée à chaque version.
- **Une modification ne part qu'avec une version.** Corriger `config.yaml` sur
  `main` ne change rien côté privé tant qu'aucun tag n'est poussé.

L'essai à blanc du workflow `Documents`, lancé à la main sans release, ne
touche pas à la partie pédagogique. Le template non plus n'en a pas.

## Configuration

Le jeton d'un workflow n'ouvre que le dépôt où il tourne : il ne peut ni créer
un dépôt dans l'organisation, ni écrire dans un autre. D'où un secret,
`PEDA_TOKEN`, à poser au niveau de l'organisation pour que tous les cours en
héritent :

| Réglage | Valeur |
|---|---|
| Type | *fine-grained personal access token* |
| Propriétaire de ressource | l'organisation `DiginamicAcademy` |
| Dépôts | *All repositories* : le dépôt privé n'existe pas encore quand le jeton sert la première fois |
| Administration | *Read and write* : créer le dépôt, l'autoriser à ouvrir des pull requests |
| Contents | *Read and write* : lire le template, écrire `config.yaml` |
| Actions | *Read and write* : lancer la régénération |

Ce jeton peut administrer tous les dépôts de l'organisation. Mieux vaut le
rattacher à un compte de service qu'à une personne, et lui donner une date
d'expiration.

Deux réglages complètent le tableau :

- le template privé doit être marqué *Template repository* (Settings →
  General) ;
- la régénération utilise, côté privé, ses propres secrets d'organisation —
  `ANTHROPIC_API_KEY` notamment. Leur visibilité doit couvrir les dépôts
  privés : un dépôt `-Peda` tout juste créé doit en hériter d'emblée.

## En cas d'échec

Les annotations du run disent ce qui manque :

- **`PEDA_TOKEN` n'est pas configuré** : le secret manque, ou n'est pas visible
  de ce dépôt ;
- **création refusée** : le jeton n'a pas la permission Administration, le
  template privé n'est pas marqué comme tel, ou un dépôt `-Peda` existe déjà
  mais hors de portée du jeton ;
- **régénération non lancée** : les Actions sont désactivées dans le dépôt
  privé, ou le jeton n'a pas la permission Actions.

Une fois la cause corrigée, inutile de publier une nouvelle version : relancer
le job en échec (*Re-run failed jobs*), ou lancer le workflow `Pédagogie` à la
main en choisissant le tag dans « Use workflow from ».

---

[← Chapitre précédent](04-Auteurs-et-contributeurs.md) · [Sommaire](Readme.md)
