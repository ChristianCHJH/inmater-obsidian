# Índice: Referencias (Catálogos globales)

## 📋 Descripción

Catálogos de referencia rápida para buscar información global del sistema sin navegar múltiples carpetas.

## 📋 Catálogos disponibles

| Catálogo | Archivo | Contenido |
|----------|---------|----------|
| **Endpoints** | `Endpoints-Catalog.md` | Todos los endpoints REST de todos los servicios |
| **Tablas** | `Tables-Catalog.md` | Todas las tablas PostgreSQL con descripción |
| **Dependencias** | `Services-Dependency-Map.md` | Qué servicio depende de cuál |

## 🔗 Cómo usar

### Si necesitas encontrar un endpoint
```
Busca en: Endpoints-Catalog.md
Formato: | GET | /microservicesPacientes/pacientes | EMR.Patient.Service | Listar pacientes |
```

### Si necesitas saber qué tablas toca una operación
```
Busca en: Tables-Catalog.md
Formato: | hc_paciente | appinmater_modulo | Datos del paciente |
```

### Si necesitas entender qué servicios se comunican
```
Busca en: Services-Dependency-Map.md
Formato: EMR.Api-Gateway.Service → EMR.Patient.Service (routing)
```

## 📝 Cuándo actualizar

- **Después de crear un endpoint:** Actualiza Endpoints-Catalog.md
- **Después de crear una tabla/columna:** Actualiza Tables-Catalog.md
- **Después de agregar una dependencia entre servicios:** Actualiza Services-Dependency-Map.md

El agente `obsidian-documenter` actualiza estos catálogos automáticamente.

## 🔗 Relaciones

- **Gobernanza:** [[06-GOVERNANCE/governance_index.md]]
- **Servicios:** [[01-SERVICIOS/servicios_index.md]]
- **Dominios:** [[02-DOMINIOS/dominios_index.md]]
- **Arquitectura:** [[03-ARQUITECTURA/arquitectura_index.md]]
- **Traces:** [[05-TRACES/traces_index.md]]

---

**Tags:** #reference #catalog
**Última actualización:** 2026-04-13
