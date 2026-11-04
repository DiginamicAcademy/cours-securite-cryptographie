# Exercice — Auditer la gestion des secrets

> **Nature :** exercice d'audit · **Durée :** 1 h · **Notion :** gestion des secrets — inventorier, évaluer les risques, remédier
> **Lien :** chapitre [Gestion des secrets](../10-secrets.md)
> **Objectif :** auditer la gestion des secrets d'un projet réel, hiérarchiser les
> risques et produire un plan de remédiation exécutable.

---

## Contexte

**Lumen Analytics** édite un tableau de bord de consommation énergétique pour des
collectivités. Le dépôt Git de l'application vient d'être transféré vers une équipe
externe de maintenance, qui vous demande un audit avant de reprendre la main. Voici
l'arborescence et les extraits pertinents.

> Les valeurs sensibles sont **masquées** dans cet énoncé : à l'audit, ce sont les
> emplacements, les formats et les portées qui comptent — pas les caractères exacts.

```text
lumen-analytics/
├── .env
├── .gitignore
├── Dockerfile
├── README.md
├── app/
│   ├── config.py
│   └── main.py
├── scripts/
│   └── backup.sh
└── .github/
    └── workflows/
        └── deploy.yml
```

**`.env`** (versionné dans le dépôt) :

```bash
DATABASE_URL=postgres://admin:[mot de passe en clair]@db.lumen.internal:5432/analytics
OPENWEATHER_API_KEY=[clé d'API - 32 caractères hexadécimaux]
```

**`.gitignore`** :

```text
__pycache__/
*.pyc
.venv/
```

**`Dockerfile`** (extrait) :

```dockerfile
FROM python:3.12-slim
WORKDIR /srv
COPY . .
ENV STRIPE_SECRET_KEY=sk_live_[masqué]
RUN pip install -r requirements.txt
CMD ["python", "app/main.py"]
```

**`app/config.py`** (extrait) :

```python
import os

# Clé de session : valeur par défaut pour le développement local.
SECRET_KEY = os.environ.get("APP_SECRET_KEY", "lumen-dev-secret-2024")
```

**`.github/workflows/deploy.yml`** (extrait) :

```yaml
steps:
  - name: Deploy to production
    run: ./scripts/deploy.sh
    env:
      DEPLOY_TOKEN: ghp_[masqué]
```

**`scripts/backup.sh`** (extrait) :

```bash
#!/usr/bin/env bash
# Sauvegarde quotidienne de la base vers le stockage objet.
export AWS_ACCESS_KEY_ID=AKIA[masqué]
export AWS_SECRET_ACCESS_KEY=[clé secrète AWS - masquée]
aws s3 sync /var/backups s3://lumen-analytics-backups
```

**`README.md`** (extrait) :

```text
## Démarrage rapide
mysql -u root -p[mot de passe en clair] lumen < schema.sql
```

Enfin, l'équipe vous apprend que le fichier `.env` est présent dans l'historique Git
depuis **huit mois**, et que le dépôt était public pendant les deux premiers mois du
projet.

---

## Consignes

### Partie 1 — Inventorier

Produisez un tableau qui recense **chaque secret** présent dans ces extraits : fichier,
motif ou valeur concernée, type de secret (clé de base de données, jeton de
déploiement, clé d'API tierce…), et ce qu'il donne accès. Visez l'exhaustivité : un
secret caché dans un commentaire ou un script compte autant que les autres.

### Partie 2 — Évaluer

Pour chaque secret trouvé :

1. attribuez une gravité — **critique**, **majeur**, **mineur** — en tenant compte de
   la portée de l'accès et de l'exposition (dépôt public, historique, image
   distribuée) ;
2. décrivez en une phrase ce qu'un attaquant pourrait en faire concrètement ;
3. signalez les secrets qui, en plus, **ne sont pas révocables** tels quels (une donnée
   personnelle diffusée ne se « révoque » pas).

### Partie 3 — Plan de remédiation

Rédigez un plan d'action **ordonné**, avec pour chaque étape son responsable type
(développeur, administrateur, sécurité) et son prérequis. L'ordre compte : justifiez
pourquoi la première étape ne peut pas être « supprimer le fichier `.env` ».

### Partie 4 — Prévenir la récidive

Proposez les garde-fous à mettre en place pour que le prochain audit ne trouve rien :
configuration Git, contrôle automatique avant commit, contrôle en intégration
continue, stockage des secrets, politique de rotation. Pour chacun, indiquez où il se
place dans le flux de travail quotidien.

---

## Attendu

- un tableau d'inventaire complet (partie 1) ;
- une évaluation graduée avec justification (partie 2) ;
- un plan de remédiation ordonné (partie 3) ;
- une liste de garde-fous situés dans le flux de travail (partie 4).

---

## ✅ Auto-évaluation

Avant de passer au corrigé, vérifiez que :

- [ ] Mon inventaire recense tous les secrets des extraits, y compris ceux des scripts et du README.
- [ ] Chaque secret porte une gravité justifiée par sa portée **et** par son exposition.
- [ ] J'ai identifié au moins un secret dont la fuite ne se répare pas par une simple rotation.
- [ ] Mon plan commence par la révocation / le remplacement, pas par le nettoyage du code.
- [ ] Mes garde-fous couvrent le commit, l'intégration continue et le stockage des secrets.
- [ ] Je peux expliquer pourquoi effacer le fichier de l'historique ne suffit pas.

---

## Pour aller plus loin

*Facultatif.* Le dépôt contient l'historique complet, avec le `.env` depuis huit mois.
Étudiez la réécriture de l'historique (outils dédiés, migration des clones) : quels
sont ses risques sur un dépôt partagé, et quelles conditions doit-on réunir avant de
la lancer ? Proposez enfin la politique de rotation des secrets de ce projet :
fréquence, déclencheurs, et procédure de bascule sans interruption.
