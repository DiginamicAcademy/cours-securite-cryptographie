<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/Logo-diginamic-color-blk.png">
    <img src="assets/Logo-diginamic-color.svg" alt="Diginamic Formation" width="358">
  </picture>
</p>

# La cryptographie

> Cours d'**autoformation** · **RNCP40573** (Répertoire national des certifications professionnelles) — Expert en informatique et systèmes d'information · **brique transversale de sécurité** adossée au bloc **BC03** (conception et développement applicatif)
> 2 journée(s) · supports **rédigés par IA** (intelligence artificielle), relus et validés.

---

## Pourquoi ce cours

Un mot de passe mal stocké, une sauvegarde non chiffrée, une clé laissée dans le code source : la cryptographie ne se règle pas à la fin d'un projet, elle se décide au début. Mal employée, elle donne un sentiment de protection ; bien employée, elle couvre des besoins très concrets — cacher un prix fournisseur, vérifier qu'une fiche technique n'a pas été modifiée, prouver l'origine d'un bon de commande.

Les incidents le rappellent régulièrement : en 2011, l'attaque de la PlayStation Network a exposé les données de 77 millions de comptes. La leçon tient en une ligne : chiffrer ne sert à rien si la clé est à portée de l'attaquant.

Ce cours donne les repères pour choisir et mettre en œuvre les bons outils : distinguer hachage, chiffrement et signature ; comprendre ce que garantissent TLS (Transport Layer Security) et les certificats ; chiffrer les données au repos ; gérer les secrets sans les laisser traîner. Tout au long des deux journées, les exemples suivent la fromagerie **DIGICHEESE** (la fromagerie du fil rouge) : ses prix fournisseurs, ses bons de commande, sa boutique en ligne et ses bases de données.

## À l'issue de ce cours

- **Décrire** les quatre services rendus par la cryptographie : confidentialité, intégrité, authentification et non-répudiation.
- **Expliquer** le principe du chiffrement symétrique et celui du chiffrement asymétrique.
- **Distinguer** hachage, chiffrement et signature, et choisir la bonne primitive pour un besoin donné.
- **Utiliser** une bibliothèque de cryptographie standard pour chiffrer, déchiffrer et hacher.
- **Repérer** les erreurs d'implémentation courantes : mode ECB (Electronic Codebook), hachage MD5 (Message-Digest 5), sel absent, vecteur d'initialisation réutilisé.
- **Choisir** un algorithme et une taille de clé adaptés à un usage donné.
- **Décrire** le fonctionnement de TLS et de la confiance par certificats.
- **Expliquer** le rôle d'une autorité de certification dans une infrastructure à clés publiques.
- **Appliquer** le chiffrement des données au repos : bases de données, fichiers, sauvegardes.
- **Mettre en œuvre** la gestion des secrets : coffre de secrets, variables d'environnement, rotation.
- **Analyser** les failles de configuration courantes de HTTPS (Hypertext Transfer Protocol Secure) et de TLS.
- **Évaluer** le compromis entre protection, performance et coût des choix de cryptographie.

## Sommaire

1. [Histoire et services de la cryptographie](01-histoire-et-services.md) — ce que fait (et ne fait pas) la cryptographie, et ses quatre services. *45 min de lecture.*

**Travaux pratiques**

- [Classer les primitives](tp/01-classer-primitives.md) — associer chaque besoin de sécurité à la primitive qui le couvre. *20 min · [corrigé](tp/corrige/01-classer-primitives.md).*

---

## Aller plus loin

*Ces approfondissements sont facultatifs : ils ne sont jamais exigés pour terminer le cours.*

- **Les recommandations de l'ANSSI** (Agence nationale de la sécurité des systèmes d'information) : l'autorité française en matière de cybersécurité publie les algorithmes et les tailles de clés jugés fiables pour un usage professionnel.
- **Deux ouvrages de référence** : *Applied Cryptography* (Bruce Schneier) pour les fondements, *Serious Cryptography* (Jean-Philippe Aumasson) pour une vision moderne et pratique.
- **La cryptographie post-quantique** : les algorithmes conçus pour résister aux ordinateurs quantiques commencent à être standardisés — un sujet à suivre pour tout choix de long terme.
