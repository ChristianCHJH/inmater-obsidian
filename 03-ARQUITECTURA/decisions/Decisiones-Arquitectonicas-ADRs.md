# Decisiones Arquitectónicas (ADRs)

Registro de decisiones arquitectónicas importantes del proyecto.

**Ubicación oficial:** `adr/` (raíz del repositorio)

## 📋 ADRs principales

| ADR | Título | Estado | Descripción |
|-----|--------|--------|-------------|
| [ADR-0003](../../adr/0003-backend-cicd-cloud-build-gcp.md) | CI/CD con Cloud Build y Cloud Run | ✅ Implementado | Backend despliega a Google Cloud Run |
| [ADR-0008](../../adr/0008-dia7-ciclo-laboratorio-lab-aspira.md) | Ciclo de laboratorio día 7 | 📝 Nuevo | Evaluación de laboratorio hasta día 7 |

## 🔑 Decisiones clave

### 1. Microservicios
Arquitectura basada en microservicios independientes comunicados vía API Gateway.

**Ventajas:**
- Escalabilidad independiente
- Equipos autónomos
- Despliegue independiente

**Desventajas:**
- Complejidad operacional
- Consistencia de datos

### 2. API Gateway
Gateway central que enruta a servicios downstream.

**Beneficios:**
- Punto único de autenticación
- Rate limiting centralizado
- Logs unificados

### 3. Microservicios en TypeScript/Node.js
Stack moderno y consistente.

**Decisión:** TypeScript + Express + Sequelize

### 4. Frontend Vue 3
SPA moderna junto a PHP legacy.

**Migración gradual:** Vue 3 en portal.inmater.pe, PHP en app.inmater.pe

### 5. PostgreSQL
Base de datos relacional con schemas múltiples.

**Schemas:**
- `appinmater_modulo` - Datos de producción
- `appinmater_log` - Auditoría

### 6. Deployment en GCP

| Componente | Plataforma |
|-----------|-----------|
| Backend services | Cloud Run |
| Frontend | Firebase Hosting |
| Database | Cloud SQL PostgreSQL |

## 📋 Patrón de decisiones

Cada ADR debe incluir:

1. **Contexto** - Por qué se toma la decisión
2. **Opción A, B, C** - Alternativas consideradas
3. **Decisión** - Qué se eligió y por qué
4. **Consecuencias** - Implicaciones
5. **Alternativas futuras** - Cambios posibles

## 📍 Ver también

- [[Arquitectura/Arquitectura-Stack-Tecnologico]]
- `adr/README.md` en el repositorio
