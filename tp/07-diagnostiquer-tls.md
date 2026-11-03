# Exercice — Diagnostiquer une configuration TLS

> 🟢 **Échauffement** · **Durée :** 25 min · **Lien :** chapitre [TLS en pratique](../07-tls-en-pratique.md)
> **Objectif :** lire une configuration HTTPS, repérer ses faiblesses et proposer la
> correction qui va avec.

---

## Contexte

**Maison Berthier**, un négociant en vins, héberge sa boutique en ligne sur un serveur
Nginx. Le site est « en HTTPS », ce qui rassure tout le monde — jusqu'au jour où un
client signale un avertissement de sécurité dans son navigateur. L'administrateur
système vous transmet la configuration en production et la politique de sécurité
interne de l'entreprise.

**Politique de sécurité interne (extrait) :**

- versions TLS acceptées : **1.2 et 1.3 uniquement** ;
- suites de chiffrement : **AEAD uniquement** (AES-GCM ou ChaCha20-Poly1305) ;
- en-tête HSTS : `max-age=63072000` avec `includeSubDomains` ;
- toute requête `http://` est redirigée vers `https://` ;
- certificat Let's Encrypt, renouvellement automatisé.

**Configuration en production :**

```nginx
server {
    listen 80;
    server_name www.maison-berthier.fr;
    root /var/www/boutique;

    location / {
        try_files $uri $uri/ =404;
    }
}

server {
    listen 443 ssl;
    server_name www.maison-berthier.fr;

    ssl_certificate     /etc/ssl/certs/boutique-selfsigned.crt;
    ssl_certificate_key /etc/ssl/private/boutique.key;

    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers RC4:DES:3DES:MD5:EXPORT;
    ssl_prefer_server_ciphers off;

    root /var/www/boutique;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

---

## Consignes

### Partie 1 — Repérer les faiblesses

Listez les écarts entre cette configuration et la politique de sécurité interne.
Visez au moins **six** constats : deux concernent les certificats ou les versions,
deux les algorithmes, deux le comportement du serveur (ce qu'il accepte, ce qu'il
renvoie).

### Partie 2 — Classer par gravité

Classez chaque constat dans l'une des trois catégories suivantes, et justifiez en
une phrase :

| Catégorie | Signification |
| --- | --- |
| **Bloquant** | La garantie de TLS est cassée : le site ne devrait pas être ouvert en l'état. |
| **Important** | La garantie tient, mais un scénario réaliste la contourne. |
| **Confort** | Bonne pratique à mettre en place, sans urgence. |

### Partie 3 — Corriger la configuration

Réécrivez les directives Nginx concernées pour respecter la politique interne. On
attend un bloc de configuration complet et commenté, prêt à être déployé — pas une
liste de conseils.

### Partie 4 — Vérifier le résultat

Indiquez les commandes à exécuter sur le serveur pour confirmer que chaque correction
est effective, et le résultat qui prouve la réussite. Prévoyez au moins une
vérification côté serveur (`openssl`) et une côté client (`curl` ou le navigateur).

---

## ✅ Auto-évaluation

Avant de passer au corrigé, vérifiez que :

- [ ] J'ai listé au moins six constats, chacun relié à une ligne précise de la configuration.
- [ ] Chaque constat porte une gravité justifiée en une phrase.
- [ ] Ma configuration corrigée respecte **tous** les points de la politique interne (versions, suites, HSTS, redirection, certificat).
- [ ] J'ai écrit une commande de vérification par correction, avec le résultat attendu.
- [ ] Je peux expliquer pourquoi un site « en HTTPS » peut malgré tout être faible.

---

## Pour aller plus loin

*Facultatif.* La politique interne ne dit rien du renouvellement du certificat.
Proposez la chaîne complète : qui demande le certificat, quand il est renouvelé, qui
recharge Nginx, et comment être prévenu si le renouvellement échoue. Ajoutez
l'activation de l'*OCSP stapling* et expliquez en deux phrases ce qu'elle apporte.
