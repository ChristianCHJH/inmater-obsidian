# DevOps e Infraestructura

## 🚀 Plataformas de Deployment

### Google Cloud Platform (GCP)

**Proyecto:** `telemedicina-477307`
**Región:** `us-central1`

#### Backend Services - Cloud Run

| Servicio | Staging | Production |
|----------|---------|------------|
| API Gateway | emr-api-gateway-staging | emr-api-gateway-service |
| Auth Service | emr-auth-staging | emr-auth-service |
| Patient Service | emr-patient-staging | - |
| Calendar Service | emr-calendar-staging | - |

**Triggger:** 
- `staging` branch → staging
- `main` branch → production

**Artifact Registry:** `us-central1-docker.pkg.dev/telemedicina-477307/cloud-run-source-deploy`

#### Frontend - Firebase Hosting

| Branch | URL |
|--------|-----|
| `staging` | portal-staging.inmater.pe |
| `main` | portal.inmater.pe |

**GitHub Actions:** Deploy automático

#### Base de datos - Cloud SQL

- **Motor:** PostgreSQL
- **Region:** us-central1
- **Backup:** Automático

## 🔧 Variables de entorno Cloud Run

```env
PORT=8080
NODE_ENV=staging|production
DB_HOST=cloudsql-host
DB_PORT=5432
DB_USER=db_user
DB_PASSWORD=db_password
DB_NAME=inmater_db
DB_SCHEMA_APP_MODULO=appinmater_modulo
JWT_SECRET=xxxxx
JWT_ACCESS_SECRET=xxxxx
PACIENTES_SERVICE_URL=http://emr-patient-service:8080
CALENDARIO_SERVICE_URL=http://emr-calendar-service:8080
FACTURACION_SERVICE_URL=http://emr-facturacion-service:8080
WITNESS_SERVICE_URL=http://emr-witness-service:8080
RATE_LIMIT=100
REQUEST_TIMEOUT=30000
```

## 🐳 Docker

**Archivo principal:** `docker-compose.yml`

```bash
# Levantar todo el stack local
docker-compose up -d

# Ver logs
docker-compose logs -f [servicio]

# Detener
docker-compose down
```

## 📋 CI/CD Pipeline

### Backend
```
Git push → Cloud Build trigger → Build image → Cloud Run deploy
```

### Frontend
```
Git push → GitHub Actions → Build Vue.js → Firebase Hosting deploy
```

## ⚡ Optimizaciones

- **Cloud Run:** Escala automática según demanda
- **Firebase:** CDN global, caching automático
- **Cloud SQL:** Backup automático, replicación

## 🔍 Monitoreo

- **Cloud Logging:** Logs centralizados
- **Cloud Monitoring:** Métricas y alertas
- **Cloud Trace:** Tracing distribuido

## 📍 Ver también

- [[Arquitectura/Arquitectura-Stack-Tecnologico]]
- `CLAUDE.md` - Instrucciones de deployment
