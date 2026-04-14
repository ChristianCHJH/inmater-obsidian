# Índice: Traces (Investigaciones documentadas)

## 📋 Descripción

Las **traces** documentan investigaciones sobre cómo funcionan los flujos del sistema. Cada trace sigue un flujo completo desde la UI → API → Base de datos.

## 📂 Subcarpetas

| Carpeta | Contenido | Índice |
|---------|----------|--------|
| **frontend/** | Investigaciones iniciadas desde UI (componentes, botones, flows) | [frontend-traces_index.md](frontend/frontend-traces_index.md) |
| **backend/** | Investigaciones de APIs, controllers, servicios, lógica | [backend-traces_index.md](backend/backend-traces_index.md) |
| **database/** | Investigaciones de queries, schemas, optimizaciones | [database-traces_index.md](database/database-traces_index.md) |

## 🔍 Cómo crear una trace

1. Usa template: [[06-GOVERNANCE/templates/trace-template.md]]
2. Documenta el flujo: Frontend → Backend → Database
3. Guarda en la carpeta correcta (frontend/, backend/, database/)
4. Actualiza el índice de la subcarpeta

**Ejemplo:** Si investigas "cómo funciona el login":
- `05-TRACES/frontend/login-flow.md` - inicioFrontend: componente Login → composable useAuth
- `05-TRACES/backend/jwt-validation-flow.md` - Backend: AuthController → JWT generation
- `05-TRACES/database/login-credentials-query.md` - Database: SELECT FROM usuarios

## 🔗 Relaciones

- **Gobernanza:** [[06-GOVERNANCE/governance_index.md]]
- **Dominios:** [[02-DOMINIOS/dominios_index.md]] (qué dominio afecta)
- **Servicios:** [[01-SERVICIOS/servicios_index.md]] (qué servicio implementa)
- **Patrones:** [[03-ARQUITECTURA/patterns/patterns_index.md]] (qué patrones usa)

---

**Tags:** #traces #investigation
**Última actualización:** 2026-04-13
