# Kiality

**Plateforme de qualité et conformité des données** pour les environnements industriels et tertiaires.

Kiality connecte vos outils métier (ERP, CRM, ITSM…), évalue vos règles de conformité en continu et pilote la résolution des anomalies détectées - de la détection à la clôture.

---

## Fonctionnement

```
Source de données (ERP · CRM · ITSM · SQL · SFTP · S3…)
        │
        ▼  Connecteur configuré
  Kiality Worker  ──▶  Évalue les règles  ──▶  Anomalie détectée ?
                                                       │ Oui
                                                       ▼
                                             Alerte email + assignation
                                                       │
                                                       ▼
                                             Portail : traitement + clôture
```

1. **Connectez** vos sources via des connecteurs configurables (REST, OAuth2, SQL, SFTP, S3, Azure Blob, MongoDB)
2. **Définissez** vos règles : conditions JSONPath, requêtes DuckDB SQL, ou description en langage naturel convertie en SQL par IA
3. **Planifiez** l'évaluation via des déclencheurs cron ou des appels API externes (n8n, Zapier…)
4. **Détectez** les anomalies automatiquement et pilotez leur résolution
5. **Alertez** les bons interlocuteurs avec escalades automatiques si vos SLA ne sont pas respectés

---

## Images Docker

Les images sont publiées sur GitHub Container Registry :

| Image | Description |
|-------|-------------|
| `ghcr.io/kiality/api` | API REST backend |
| `ghcr.io/kiality/frontend` | Portail client Blazor Server |
| `ghcr.io/kiality/worker` | Worker de traitement asynchrone |

```bash
docker pull ghcr.io/kiality/api:latest
docker pull ghcr.io/kiality/frontend:latest
docker pull ghcr.io/kiality/worker:latest
```

> Privilégiez un tag de version fixe en production (`1.2.3`) plutôt que `latest`.

---

## Démarrage rapide

### Prérequis

- Docker 24+
- Docker Compose v2
- Une clé de licence Kiality - [demander un accès](https://www.kiality.com/Demande)

### 1. Créer le fichier `.env`

```env
# Base de données
POSTGRES_PASSWORD=changeme

# JWT — générer avec : openssl rand -base64 64
JWT_KEY=votre-clé-jwt-très-longue-et-secrète

# Licence (fournie par Kiality)
LICENSE_PUBLIC_KEY="-----BEGIN PUBLIC KEY-----\n...\n-----END PUBLIC KEY-----"
HUB_API_KEY=votre-clé-hub

# SMTP (optionnel)
SMTP_HOST=
SMTP_PORT=587
SMTP_USER=
SMTP_PASSWORD=

# LLM pour le mode Langage Naturel (optionnel)
AI_PROVIDER=anthropic
AI_MODEL=claude-sonnet-4-6
AI_API_KEY=
```

### 2. Créer le fichier `docker-compose.yml`

```yaml
services:
  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: kiality
      POSTGRES_USER: kiality
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U kiality"]
      interval: 10s
      retries: 5

  api:
    image: ghcr.io/kiality/api:latest
    depends_on:
      postgres:
        condition: service_healthy
    environment:
      ConnectionStrings__DefaultConnection: "Host=postgres;Database=kiality;Username=kiality;Password=${POSTGRES_PASSWORD}"
      Jwt__Key: ${JWT_KEY}
      Jwt__Issuer: KialityApi
      Jwt__Audience: KialityFrontend
      License__PublicKeyPem: ${LICENSE_PUBLIC_KEY}
      License__HubApiKey: ${HUB_API_KEY}
      License__HubValidationUrl: https://hub.kiality.com/api/license/heartbeat
      Hub__CommunityUrl: https://hub.kiality.com
      AppBaseUrl: http://localhost:8080
      AppFrontendUrl: http://localhost:3000
    ports:
      - "8080:8080"

  frontend:
    image: ghcr.io/kiality/frontend:latest
    depends_on:
      - api
    environment:
      ApiBaseUrl: http://api:8080/
    ports:
      - "3000:8080"

  worker:
    image: ghcr.io/kiality/worker:latest
    depends_on:
      - api
    environment:
      ConnectionStrings__DefaultConnection: "Host=postgres;Database=kiality;Username=kiality;Password=${POSTGRES_PASSWORD}"

volumes:
  postgres_data:
```

### 3. Lancer

```bash
docker compose up -d

# Vérifier que tous les services sont healthy
docker compose ps
```

### 4. Premier accès

| Service | URL locale | Description |
|---------|-----------|-------------|
| Portail | http://localhost:3000 | Interface utilisateur Blazor |
| API | http://localhost:8080 | REST API |
| Swagger | http://localhost:8080/swagger | Documentation API interactive |

Identifiants par défaut :

```
Email         : account@domaine.com
Mot de passe  : changeme
```

> ⚠️ Changez le mot de passe dès la première connexion.

Activez ensuite votre licence dans **Paramètres → Licence**.

---

## Tiers et fonctionnalités

| Fonctionnalité | Free | Starter | Business | Enterprise |
|----------------|:----:|:-------:|:--------:|:----------:|
| Gestion des anomalies | ✅ | ✅ | ✅ | ✅ |
| Normes DuckDB / Langage naturel | ✅ | ✅ | ✅ | ✅ |
| Récapitulatifs email | ❌ | ✅ | ✅ | ✅ |
| Escalades automatiques | ❌ | ❌ | ✅ | ✅ |
| Journal d'audit | ❌ | ❌ | ✅ | ✅ |
| Export CSV / PDF | ❌ | ❌ | ✅ | ✅ |
| Pièces jointes sur anomalies | ❌ | ❌ | ✅ | ✅ |
| Catalogue Hub communautaire | ❌ | ❌ | ✅ | ✅ |
| Webhooks outbound | ❌ | ❌ | ❌ | ✅ |
| Dashboard SLA & tendances | ❌ | ❌ | ❌ | ✅ |
| SSO Microsoft / Google | ❌ | ❌ | ❌ | ✅ |
| Simulation dry-run des normes | ❌ | ❌ | ❌ | ✅ |
| **Normes max** | 1 | 10 | 50 | Illimité |
| **Utilisateurs max** | 1 | 10 | 50 | Illimité |

---

## Documentation

La documentation complète est disponible sur **[docs.kiality.com](https://docs.kiality.com)** :

- [Installation](https://docs.kiality.com/docs/demarrage/installation)
- [Concepts clés](https://docs.kiality.com/docs/concepts)
- [Guide utilisateur](https://docs.kiality.com/docs/guide-utilisateur)
- [Guide administrateur](https://docs.kiality.com/docs/guide-admin)
- [Référence API REST](https://docs.kiality.com/docs/api)
- [Déploiement Kubernetes](https://docs.kiality.com/docs/deploiement/kubernetes)

---

## Obtenir une licence

👉 [www.kiality.com/Demande](https://www.kiality.com/Demande)

📧 [contact@kiality.com](mailto:contact@kiality.com)

---

## Mentions légales

Les images Docker distribuées dans ce registre sont un logiciel propriétaire édité par **Devapy**.  
Leur utilisation est soumise à l'acceptation des [conditions générales d'utilisation](https://www.kiality.com/cgu).  
Toute redistribution, modification ou utilisation sans licence valide est interdite.
