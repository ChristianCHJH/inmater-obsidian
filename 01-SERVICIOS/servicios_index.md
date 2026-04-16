# Índice: Servicios y Repositorios

## 📋 Descripción

Catálogo de todos los servicios (microservicios backend, frontend, legacy) que componen el ecosistema inmater.

## 📂 Servicios disponibles

### Backend - Microservicios

| Servicio | Puerto | Descripción | Repo |
| --- | --- | --- | --- |
| **EMR.Api-Gateway.Service** | 8080 | Enrutador central | inmater-core |
| **EMR.Auth.Service** | 3003 | Autenticación y JWT | inmater-core |
| **EMR.Patient.Service** | 3001 | Datos de pacientes | inmater-core |
| **EMR.Calendar.Service** | 3002 | Calendario y turnos | inmater-core |

### Frontend

| Aplicación | URL | Descripción | Repo | Tech |
| --- | --- | --- | --- | --- |
| **portal.inmater.pe** (nueva) | [portal.inmater.pe](https://portal.inmater.pe) | SPA Vue 3 | GitHub | Vue 3 + Vite + PrimeVue |
| **app.inmater.pe** (legacy) | [app.inmater.pe](https://app.inmater.pe) | PHP legacy | Bitbucket | PHP + jQuery Mobile |

### PHP Legacy — Módulos documentados

| Módulo | Archivo | Doc |
| --- | --- | --- |
| Emisión de comprobantes | `apps/web/pago.php` | [[01-SERVICIOS/php-legacy/pago-emision-comprobantes]] |

## 📝 Cómo agregar un nuevo servicio

1. Crea carpeta: `01-SERVICIOS/[nombre-servicio]/`
2. Crea archivos:
   - `[nombre-servicio]_index.md` - Índice del servicio
   - `README.md` - Información del servicio
   - `architecture/` - Decisiones de diseño
   - `endpoints/` - Documentación de endpoints
   - `deployment/` - CI/CD, deployment notes
3. Actualiza este índice

## 🔗 Relaciones

- **Dominios:** [[02-DOMINIOS/dominios_index.md]] (qué lógica implementan)
- **Arquitectura:** [[03-ARQUITECTURA/arquitectura_index.md]] (qué patrones usan)
- **Referencias:** [[04-REFERENCIAS/referencias_index.md]] (catálogos de endpoints)
- **Governance:** [[06-GOVERNANCE/governance_index.md]]

---

**Tags:** #services #repository
**Última actualización:** 2026-04-16
