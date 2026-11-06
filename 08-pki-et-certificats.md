[← Chapitre précédent](07-tls-en-pratique.md) · [Sommaire](Readme.md) · [Chapitre suivant →](09-donnees-au-repos.md)

# PKI et certificats

> Le sceau du roi est reconnu de tous parce que tout le monde connaît le sceau. Une
> lettre qui le porte emprunte la confiance accordée au souverain, sans que le
> destinataire ait jamais rencontré l'émetteur. La **PKI** moderne fonctionne sur le
> même principe : des **autorités de certification** dont les navigateurs font
> confiance par défaut, et une chaîne de signatures qui relie votre site à l'une
> d'entre elles.

**Durée de lecture :** 50 min hors approfondissements facultatifs · **Prérequis :** le chapitre [TLS en pratique](07-tls-en-pratique.md)

---

## Objectifs

À l'issue de ce chapitre, vous serez capable de :

- **Décrire** la chaîne de confiance d'un certificat, de la feuille serveur jusqu'à la
  racine de confiance.
- **Distinguer** les rôles d'une autorité de certification et d'un mécanisme de
  révocation (CRL, OCSP).
- **Vérifier** la chaîne de certificats d'un site avec `openssl`.
- **Planifier** le renouvellement et la rotation d'un certificat sans interruption de
  service.

---

## I. Chaîne de confiance

### Une infrastructure, pas un fichier

Une **PKI** (*Public Key Infrastructure*, infrastructure à clé publique) est
l'ensemble des rôles, des documents et des procédures qui permet de dire « cette clé
publique appartient bien à ce nom ». Son produit visible est le **certificat** ; son
produit réel est la **confiance**. Trois acteurs la composent :

- l'**autorité de certification** (AC, *certificate authority*) : elle signe les
  certificats. Sa signature est ce que le navigateur reconnaît ;
- le **demandeur** : le propriétaire d'un domaine, qui prouve qu'il le contrôle et
  reçoit un certificat à son nom ;
- le **client** : navigateur ou application, qui détient une liste d'autorités racines
  de confiance et vérifie chaque certificat qui lui est présenté.

### De la feuille à la racine

En production, un certificat n'est presque jamais signé directement par une racine.
La chaîne compte trois niveaux :

```text
Racine (root CA)                — auto-signée, rangée hors ligne,
   |                              présente dans le magasin de confiance
   | signe                        du navigateur et du système
   v
Autorité intermédiaire          — signe les certificats du quotidien,
   |                              reste révocable sans toucher la racine
   | signe
   v
Certificat feuille (serveur)    — celui que présente votre site,
                                  à son nom de domaine
```

La contrainte est concrète : on ne sort la racine du coffre que pour signer une
intermédiaire, et une intermédiaire compromise se remplace sans toucher à la racine.
C'est pourquoi votre site présente souvent **deux** certificats : le sien et celui de
l'intermédiaire.

### Ce que le client vérifie

À la réception du certificat, le navigateur contrôle quatre choses :

1. **La signature** : chaque maillon est bien signé par le suivant, jusqu'à une
   racine qu'il fait confiance.
2. **Le nom de domaine** : le champ *Subject Alternative Name* du certificat couvre
   bien le domaine demandé.
3. **Les dates** : la date d'émission est passée, la date d'expiration n'est pas
   atteinte.
4. **La révocation** : le certificat n'a pas été annulé avant son terme (voir
   partie III).

Un seul contrôle qui échoue, et le navigateur affiche un avertissement plein écran.

> 🧪 **Atelier rapide (5 min)**
>
> Affichez la chaîne telle que la reçoit un client, en une seule connexion :
>
> ```bash
> openssl s_client -connect tutor.diginamic.fr:443 -showcerts
> ```
>
> Repérez la section `Certificate chain` : elle liste les certificats du plus proche
> (votre serveur) au plus lointain (l'autorité intermédiaire). La racine, elle, n'est
> presque jamais envoyée — le client la possède déjà. Recherchez ensuite `CN =` (le nom
> commun) et `depth` pour identifier le rôle de chacun, puis notez le nombre de
> maillons : c'est la longueur réelle de la chaîne de confiance de ce site.

---

## II. Let's Encrypt et la rotation

### Renouveler sans y penser

Un certificat expire. Tant que le renouvellement est manuel, l'expiration est une
question de temps — et les pannes les plus spectaculaires de l'histoire du web viennent
de certificats oubliés, pas d'algorithmes cassés. **Let's Encrypt**, autorité lancée
en 2015 par l'ISRG (*Internet Security Research Group*), a changé l'équation en
rendant les certificats gratuits, courts (90 jours) et **renouvelés par script**, via
le protocole **ACME** (*Automated Certificate Management Environment*).

Le client ACME dialogue avec l'autorité, prouve le contrôle du domaine (en déposant
un jeton sur le site ou un enregistrement DNS), reçoit son certificat, l'installe et
recharge le serveur. Le cycle complet se déclenche tout seul, typiquement deux fois
par mois.

### La rotation des clés

Renouveler un certificat n'est pas tout à fait la même chose que **tourner la clé**.
Le renouvellement prolonge la validité ; la rotation remplace la paire de clés
elle-même. Les deux se font ensemble quand on le peut : à chaque renouvellement,
générer une nouvelle paire de clés limite la durée d'exposition d'une clé donnée —
si elle fuit, la fenêtre de risque est bornée par la date du prochain renouvellement.

Trois réflexes opérationnels :

- **Automatiser** : un minuteur système, pas un rappel dans un agenda.
- **Surveiller** : une alerte si un certificat expire dans moins de quinze jours, ou
  si le dernier renouvellement a échoué. L'automatisation sans surveillance échoue
  en silence.
- **Tester la reprise** : vérifier après déploiement que la chaîne complète est bien
  servie (certificat + intermédiaire). Un certificat valide servi sans son
  intermédiaire casse la chaîne pour une partie des clients.

> [!WARNING]
> Un certificat renouvelé automatiquement n'est pas un certificat surveillé
> automatiquement. Le scénario classique : le script de renouvellement échoue sur un
> changement de configuration, personne n'est prévenu, et le site affiche un
> avertissement trois mois plus tard. La notification fait partie du dispositif.

---

## III. Quand la chaîne casse

La confiance est **transitive** : vous faites confiance à la racine, donc à ce
qu'elle signe, donc au certificat de votre banque. Cette transitivité est une force
et une fragilité — elle se rompt à tous les maillons.

### Révoquer un certificat

Un certificat compromis — clé privée volée, domaine changé de propriétaire, erreur
d'émission — doit être **annulé avant son expiration**. Deux mécanismes existent :

- la **CRL** (*Certificate Revocation List*, liste de révocation) : l'autorité publie
  la liste des certificats annulés, que le client télécharge ;
- l'**OCSP** (*Online Certificate Status Protocol*) : le client interroge l'autorité
  sur un certificat précis.

Les deux ont la même faiblesse : en cas de doute ou d'indisponibilité, les clients
**continuent souvent** (comportement dit *soft-fail*). Révoquer n'est donc pas
toujours suffisant — dans les cas graves, on remplace le certificat et on retire
l'autorité des magasins de confiance.

### Le cas DigiNotar (2011)

L'autorité néerlandaise **DigiNotar** est compromise en 2011 : un attaquant y émet des
certificats valides pour des domaines qu'il ne contrôle pas. En quelques semaines les
navigateurs retirent sa racine des magasins de confiance, et l'entreprise fait
faillite. La leçon structure tout le paysage actuel : **une autorité corrompue casse la
chaîne entière**, pour tous ses clients, d'un seul coup — d'où la transparence des
certificats (partie IV) et la séparation stricte entre racines hors ligne et autorités
intermédiaires. Le détail de l'affaire est en fin de chapitre.

> [!DANGER]
> « Faire confiance à une autorité » n'est pas un sentiment, c'est une décision
> inscrite dans un magasin de confiance — et qui se retire. Quand une autorité est
> compromise, la bonne réaction n'est pas de « mieux vérifier les certificats » :
> c'est de retirer l'autorité et de remplacer tout ce qu'elle a signé.

---

## IV. Aller plus loin

🔁 **Pour aller plus loin (facultatif)** — à lire si le créneau le permet, rien
d'obligatoire n'en dépend.

### DigiNotar en détail (2011)

L'attaquant qui prend l'autorité néerlandaise **DigiNotar** émet des certificats
valides pour des domaines qu'il ne contrôle pas, dont `*.google.com`, et s'en sert pour
intercepter des connexions ; l'affaire n'est découverte qu'en août. Les navigateurs
retirent la racine de leurs magasins de confiance en quelques semaines, l'entreprise
est mise en faillite. Les contraintes DNS sur les autorités habilitées (enregistrements
CAA, juste en dessous) viennent de là.

- **Certificate Transparency (CT)** : depuis 2018, les certificats émis pour les
  domaines publics sont inscrits dans des **journaux publics** infalsifiables, et le
  serveur remet les preuves d'inscription (*SCT*, *Signed Certificate Timestamp*) avec
  le certificat. On peut surveiller ces journaux pour son propre domaine et lever une
  alerte dès qu'une émission inconnue y apparaît — c'est le garde-fou qui aurait trahi
  DigiNotar en quelques heures.
- **Les enregistrements CAA** (*Certification Authority Authorization*) : une entrée
  DNS qui déclare quelles autorités sont habilitées à émettre pour votre domaine. Une
  autorité conforme refuse d'émettre si elle n'est pas listée — une barrière de plus
  contre une émission par erreur ou par une autorité tierce.
- **La durée de vie des certificats** : le secteur réduit continuellement la durée
  maximale — 90 jours chez Let's Encrypt, et des discussions actuelles pour des
  certificats de quelques jours. Plus un certificat vit peu, moins une clé volée
  sert longtemps.

---

## Points clés

- Une PKI repose sur une **chaîne de signatures** : feuille, autorité intermédiaire,
  racine de confiance — la racine reste hors ligne, l'intermédiaire travaille.
- Le client vérifie quatre choses : **signature, nom de domaine, dates, révocation**.
- **Let's Encrypt** a rendu le renouvellement gratuit et automatique ; la **rotation**
  remplace en plus la paire de clés, ce qui borne la durée d'exposition.
- Révoquer (CRL, OCSP) est fragile : les clients contournent souvent la vérification.
  Dans les cas graves, on **remplace** plutôt qu'on ne révoque.
- La confiance est transitive et se rompt d'un coup : une autorité compromise se
  **retire**, et tout ce qu'elle a signé est remplacé.

---

## 🧠 Vérifiez vos acquis

*Formatif, jamais noté. Répondez sans relire le chapitre, puis dépliez « Vérifier ses réponses ». Ce qui compte est de repérer où vous vacillez, pas de faire 5 sur 5.*

**1.** Pourquoi la chaîne compte-t-elle une autorité intermédiaire au lieu de signer les certificats de serveur directement depuis la racine ?

- A. Parce que la racine ne sait signer que des noms de domaine, pas des clés : il lui faut un maillon intermédiaire
- B. Parce qu'une intermédiaire compromise se remplace sans toucher à la racine, qu'on ne sort du coffre que pour signer une intermédiaire
- C. Parce que le navigateur refuse tout certificat de serveur signé directement par une racine
- D. Parce que cela raccourcit la poignée de main TLS : un maillon de moins à transmettre au client

**2.** Le navigateur vérifie quatre choses à la réception d'un certificat. Laquelle ne fait **pas** partie de ces contrôles ?

- A. La signature de chaque maillon jusqu'à une racine de confiance
- B. Le nom de domaine couvert par le certificat
- C. Le coût du certificat pour l'exploitant du site
- D. La révocation : le certificat n'a pas été annulé avant son terme

**3.** Quelle distinction le cours pose-t-il entre renouveler un certificat et tourner la clé ?

- A. C'est la même opération, deux noms selon l'outil
- B. Le renouvellement prolonge la validité ; la rotation remplace la paire de clés elle-même
- C. La rotation se fait par ACME, le renouvellement manuellement
- D. Le renouvellement change le nom de domaine, la rotation change l'autorité

**4.** Un certificat dont la clé privée a été volée est révoqué, mais des clients continuent de lui faire confiance. Pourquoi ?

- A. Parce que CRL et OCSP sont contournés par les clients en cas de doute ou d'indisponibilité (*soft-fail*)
- B. Parce que la révocation ne s'applique qu'aux certificats déjà expirés, les autres restant jugés valides
- C. Parce que seule l'autorité racine peut révoquer, et qu'elle ne le fait jamais pour une intermédiaire
- D. Parce que le navigateur ne vérifie la révocation qu'au renouvellement, jamais à chaque connexion

**5.** *Question ouverte (2-3 lignes).* Le script de renouvellement automatique d'un certificat échoue un mardi, personne ne le voit, et le site tombe en avertissement plein écran trois mois plus tard. Quels sont les trois réflexes opérationnels du chapitre, et ce que chacun apporte ?

---

<details>
<summary>Vérifier ses réponses</summary>

### Réponses

**1. B.** La racine est auto-signée et rangée hors ligne ; l'intermédiaire, elle, signe les certificats du quotidien et reste révocable sans toucher à la racine.
**Pourquoi :** c'est aussi pour cela qu'un site présente souvent **deux** certificats — le sien et celui de l'intermédiaire.

**2. C.** Les quatre contrôles sont : signature, nom de domaine (*Subject Alternative Name*), dates, révocation.
**Pourquoi :** un seul contrôle qui échoue suffit à déclencher l'avertissement plein écran — le coût du certificat n'a aucun effet sur la validation.

**3. B.** Les deux se font ensemble quand on le peut : générer une nouvelle paire de clés à chaque renouvellement borne la durée d'exposition d'une clé.
**Pourquoi :** si la clé fuit, la fenêtre de risque est bornée par la date du prochain renouvellement — comme pour toute rotation de clé.

**4. A.** CRL (liste publiée que le client télécharge) et OCSP (interrogation en ligne d'un certificat précis) partagent la même faiblesse : en cas de doute ou d'indisponibilité, les clients continuent souvent.
**Pourquoi :** dans les cas graves, on **remplace** le certificat et on retire l'autorité des magasins de confiance — DigiNotar (2011) l'a montré : une autorité corrompue casse la chaîne entière, pour tous ses clients, d'un seul coup.

**5.** Attendu : **automatiser** (un minuteur système, pas un rappel dans un agenda) ; **surveiller** (alerter si un certificat expire dans moins de quinze jours ou si le dernier renouvellement a échoué) ; **tester la reprise** après déploiement (vérifier que la chaîne complète est servie, certificat + intermédiaire).
**Pourquoi :** un certificat renouvelé automatiquement n'est pas un certificat surveillé automatiquement — l'automatisation sans surveillance échoue en silence, et un certificat valide servi sans son intermédiaire casse la chaîne pour une partie des clients.

</details>

---

[← Chapitre précédent](07-tls-en-pratique.md) · [Sommaire](Readme.md) · [Chapitre suivant →](09-donnees-au-repos.md)
