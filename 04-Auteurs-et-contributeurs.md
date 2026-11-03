[← Chapitre précédent](03-Slides.md) · [Sommaire](Readme.md) · [Chapitre suivant →](05-Partie-pedagogique.md)

# Créditer l'auteur et les contributeurs

Un support de cours circule détaché de son dépôt : téléchargé, transféré,
imprimé. Il doit donc porter en lui-même le nom de qui l'a écrit. Le workflow
s'en charge, à partir de deux sources bien distinctes.

| Source | Nature | Où elle apparaît |
|---|---|---|
| `credits.authors`, dans `config.yaml` | une décision éditoriale, écrite à la main | page de garde et ouverture des crédits |
| L'historique git | un relevé, calculé | liste des contributeurs, en fin de document |

La distinction est le cœur du sujet : **signer n'est pas contribuer**. Corriger
trois coquilles inscrit au relevé, pas au générique.

## La section `credits` de `config.yaml`

Les signataires se déclarent dans `config.yaml`, sous la déclaration du cours,
dans la liste `authors` :

```yaml
credits:
  authors:
    - name: Jean-François Vial
      github:
        - JeffDiginamic
```

| Champ | Rôle |
|---|---|
| `name` | le nom affiché sur la page de garde et dans les crédits |
| `github` | les comptes GitHub avec lesquels il commite, pour reconnaître ses commits ; facultatifs |

La liste est facultative. Le workflow `Vérifier` en contrôle la forme à chaque
push qui modifie le fichier, et s'assure que chaque compte existe. La section
reste dans le cours : elle n'est pas recopiée dans sa partie pédagogique.

Aucune adresse électronique n'y figure : `config.yaml` est public, comme tout le
dépôt. Voir « Garder son adresse privée », plus bas.

## Ce que les signataires deviennent

Un signataire par élément, dans l'ordre où il doit apparaître : c'est un choix
éditorial, pas un classement.

Les noms sont ensuite assemblés en une phrase française — « A », « A et B »,
« A, B et C » — qui sert à deux endroits :

- sur la **page de garde**, en regard du libellé « Par » ;
- en tête de la section **Crédits**, sous la forme « Cours de A et B. ».

Sans `authors`, ces deux mentions disparaissent : la page de garde ne porte plus
que le dépôt, la source et la version, et les crédits s'ouvrent sur « Ont
contribué à ce cours ».

## Les contributeurs

La liste est tirée de l'historique git, les plus prolifiques d'abord. Quatre
étapes :

1. **L'historique** fournit un nom et une adresse par commit.
2. **GitHub** relie chaque adresse à un compte, quand elle en a un.
3. **Le tri** écarte les robots, les doublons, et les signataires — reconnus à
   leur nom ou à leur compte —, déjà crédités plus haut.
4. Pour les autres, **le compte** l'emporte sur le commit : son nom sur celui du
   commit, souvent un pseudonyme, et son adresse publique sur l'adresse de
   commit. Une adresse `noreply` est masquée.

Le recours à GitHub est du « au mieux » : sans réseau ni droits suffisants, le
workflow émet un avertissement et retombe sur les identités des commits. Le
document se construit dans tous les cas.

## Reconnaître un signataire dans l'historique

Un poste au bureau, un autre à la maison, un pseudonyme dans la configuration
git de l'un d'eux : les commits d'une même personne portent rarement un seul
nom. Un signataire n'est écarté des contributeurs que si ses commits sont
reconnus comme les siens — sinon, il est crédité deux fois, comme signataire et
comme contributeur.

C'est le rôle de `github` : tous les commits que GitHub rattache à l'un de ces
comptes lui sont attribués, quels que soient le nom et l'adresse saisis sur le
poste. GitHub rattache un commit à un compte quand l'adresse du commit est
vérifiée sur ce compte — elle peut y rester privée.

Sans `github`, un signataire n'est reconnu que si ses commits portent
exactement son `name`, ou si son compte GitHub porte ce nom.

## Garder son adresse privée

`config.yaml` ne contient aucune adresse. Mais chaque commit porte celle de son
auteur, lisible par quiconque parcourt l'historique d'un dépôt public. Pour
qu'une adresse ne soit pas exposée :

1. dans GitHub → *Settings* → *Emails*, cocher *Keep my email addresses
   private* et *Block command line pushes that expose my email* ;
2. commiter avec l'adresse `noreply` que GitHub affiche alors :

```bash
git config --global user.email "<id>+<compte>@users.noreply.github.com"
```

Cette adresse est rattachée au compte : les commits restent reconnus, et le
support n'affiche que le nom. Une adresse déjà publiée dans l'historique y
reste, en revanche : le changement ne vaut que pour les commits suivants.

## Ce que le document affiche

La section « Crédits » ferme le support. Elle n'est ajoutée que s'il y a quelque
chose à dire : sans signataire ni contributeur, elle n'existe pas.

```markdown
## Crédits

Cours de Jean-François Vial.

Ont également contribué :

- Une Contributrice <elle@exemple.fr>
- Un Contributeur
```

Une personne sans adresse publique n'apparaît que par son nom : une adresse
`noreply` n'apprendrait rien à personne.

## Les robots ne sont pas crédités

Les commits signés par un assistant — Claude, un `[bot]` quelconque — sont
écartés de la liste. Les lignes `Co-Authored-By` ne sont de toute façon pas
lues : seul l'auteur du commit compte.

## Les anciens fichiers `AUTHORS` et `.mailmap`

Les cours créés avant `config.yaml` déclaraient leurs signataires dans un
fichier `AUTHORS`, et leurs adresses dans un `.mailmap`. Le workflow ne lit
plus `AUTHORS`, et le signale par un avertissement s'il en trouve un : ses
lignes sont à reporter dans `authors`, avec les comptes GitHub des signataires
sous `github`. Un `.mailmap` resté en place est encore appliqué par git, mais
il expose les adresses qu'il cite : mieux vaut le supprimer.

---

[← Chapitre précédent](03-Slides.md) · [Sommaire](Readme.md) · [Chapitre suivant →](05-Partie-pedagogique.md)
