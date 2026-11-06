# Corrigé — Diagnostiquer une configuration TLS

> Corrigé de [l'exercice « Diagnostiquer une configuration TLS »](../07-diagnostiquer-tls.md).
> Comparez d'abord votre travail aux **étapes** ci-dessous : la méthode compte
> autant que le résultat.

---

## Partie 1 — Repérer les faiblesses et les classer

**Éléments attendus :** sept constats, du plus grave au plus accessoire.

| N° | Constat | Où dans la configuration | Gravité | Pourquoi ce degré |
| --- | --- | --- | --- | --- |
| 1 | Le certificat est **auto-signé** (`boutique-selfsigned.crt`) : il chiffre, mais n'authentifie personne. | `ssl_certificate` | **Bloquant** | Le navigateur ne peut pas authentifier le serveur : la garantie d'identité de TLS est absente. |
| 2 | Le serveur `80` **sert le contenu en clair** au lieu de rediriger vers `https://`. | premier bloc `server` | **Bloquant** | Tant qu'une version en clair répond, un attaquant peut s'y tenir — le chiffrement devient optionnel. |
| 3 | Les protocoles **TLSv1** et **TLSv1.1** sont acceptés alors que la politique n'autorise que 1.2 et 1.3. | `ssl_protocols` | **Important** | Un client malveillant peut forcer ces versions, dont les attaques sont connues (BEAST) et le statut déprécié. |
| 4 | Les suites **RC4, DES, 3DES, MD5, EXPORT** sont toutes faibles ou cassées ; aucune suite AEAD n'est proposée. | `ssl_ciphers` | **Important** | La confidentialité de la session dépend de la suite négociée ; avec RC4 ou 3DES, elle est théorique. |
| 5 | Aucun en-tête **HSTS** n'est renvoyé : rien n'empêche un retour en clair. | bloc `443` (absent) | **Important** | Le site est protégé aujourd'hui, mais rien ne garantit la prochaine connexion contre un *downgrade*. |
| 6 | Aucune **chaîne de certification** Let's Encrypt n'est utilisée, alors que la politique l'impose — et donc aucun renouvellement automatisé. | `ssl_certificate` | **Important** | C'est la cause la plus fréquente de panne réelle : le certificat finit par expirer. |
| 7 | `ssl_prefer_server_ciphers off` laisse le client choisir la suite. | `ssl_prefer_server_ciphers` | **Confort** | Le réglage n'est pas une faille ; il complète une liste de suites déjà restreinte. |

**Ce qu'il faut retenir :** le constat 2 est celui que l'on rate le plus souvent. On
regarde le bloc `443` avec soin, et on oublie que le bloc `80` répond encore — le
site est donc accessible en clair, tout simplement, pour qui tape l'adresse sans le
`s`.

**Erreurs fréquentes :** classer le HSTS en « confort ». L'en-tête est court, mais il
ferme la dernière porte du *downgrade* : c'est lui qui transforme « le site sait
faire du HTTPS » en « le site est en HTTPS, point ». Inversement, traiter
`ssl_prefer_server_ciphers` comme une urgence dilue le diagnostic : les correctifs
bloquants doivent être traités en premier.

---

## Partie 2 — Corriger la configuration

**Éléments attendus :**

```nginx
# Redirection obligatoire : aucune ressource n'est servie en clair.
server {
    listen 80;
    server_name www.maison-berthier.fr;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name www.maison-berthier.fr;

    # Certificat Let's Encrypt : chaîne complète, renouvelée automatiquement.
    ssl_certificate     /etc/letsencrypt/live/www.maison-berthier.fr/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/www.maison-berthier.fr/privkey.pem;

    # Versions conformes à la politique interne.
    ssl_protocols TLSv1.2 TLSv1.3;

    # Uniquement des suites AEAD (AES-GCM ou ChaCha20-Poly1305).
    ssl_ciphers ECDHE+AESGCM:ECDHE+CHACHA20;

    # HSTS : deux ans, sous-domaines inclus.
    add_header Strict-Transport-Security "max-age=63072000; includeSubDomains" always;

    root /var/www/boutique;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

**Points de méthode :**

- La redirection se fait par `return 301` : le serveur `80` ne sert plus rien, il
  renvoie vers `https://`. C'est la correction la plus importante du lot.
- `ssl_protocols` liste explicitement les versions acceptées : ne jamais s'appuyer
  sur les valeurs par défaut de la version de Nginx installée.
- `ssl_ciphers` restreint la négociation aux suites AEAD. Les suites de TLS 1.3 ne
  se configurent pas ici : le protocole les impose toutes fortes.
- L'en-tête HSTS porte `always` pour être renvoyé aussi sur les réponses d'erreur.
- `ssl_prefer_server_ciphers` peut être supprimé : avec une liste de suites déjà
  réduite aux algorithmes forts, son réglage change peu de chose.

---

## Partie 3 — Vérifier le résultat

**Éléments attendus :**

| Vérification | Commande | Résultat qui prouve la réussite |
| --- | --- | --- |
| Les anciennes versions sont refusées | `openssl s_client -connect www.maison-berthier.fr:443 -tls1_1` | La connexion échoue (pas de version commune). |
| La version forte fonctionne | `openssl s_client -connect www.maison-berthier.fr:443 -tls1_2` | La poignée de main aboutit et affiche la suite négociée. |
| Le certificat est valide | `openssl s_client -connect www.maison-berthier.fr:443 -showcerts` | La chaîne remonte jusqu'à une autorité reconnue, sans erreur de nom de domaine. |
| La redirection est active | `curl -I http://www.maison-berthier.fr/` | Le serveur répond par un `301` vers `https://`. |
| HSTS est présent | `curl -I https://www.maison-berthier.fr/` | L'en-tête `Strict-Transport-Security` apparaît avec la valeur prévue. |

**Ce qu'il faut retenir :** une configuration ne se juge pas à sa lecture, mais à ce
que le serveur répond réellement. Chaque ligne corrigée a sa contrepartie observable
— si la commande ne montre rien, la correction n'est pas en production.

---

## Pour aller plus loin

**Chaîne de renouvellement attendue :** un client ACME (par exemple *certbot*)
déclenche le renouvellement avant expiration, place les fichiers dans
`/etc/letsencrypt/live/`, puis recharge Nginx via un crochet post-renouvellement. La
surveillance compte : une alerte est envoyée si un certificat expire dans moins de
quinze jours ou si le dernier renouvellement a échoué — c'est exactement le défaut
qui a causé les deux pannes racontées dans l'approfondissement du chapitre TLS
(décembre 2018, février 2020).

**OCSP stapling :** le serveur interroge lui-même l'autorité sur la validité du
certificat et joint la réponse à la poignée de main (`ssl_stapling on;
ssl_stapling_verify on;`). Le navigateur n'a plus à contacter l'autorité à chaque
visite : la connexion démarre plus vite, et elle ne dépend plus de la disponibilité
du service OCSP de l'autorité.

---

## Points à retenir

- Un site « en HTTPS » peut être faible sur **six** points à la fois : le cadenas
  n'est pas un audit.
- Les deux constats bloquants — certificat non authentifiant, contenu encore servi en
  clair — annulent la garantie de TLS ; ils se traitent avant tout le reste.
- La politique de sécurité interne est le référentiel du diagnostic : comparer la
  configuration à une règle écrite évite les jugements de goût.
- Chaque correction se **vérifie par une commande**, côté serveur et côté client.
- Le renouvellement automatisé n'est pas un détail de confort : c'est la première
  cause réelle de rupture de HTTPS.
