# Índice: Arquitectura (Decisiones, Patrones, Infraestructura)

## 📋 Descripción

Contiene decisiones arquitectónicas, patrones técnicos reutilizables, e infraestructura del sistema.

## 📂 Subcarpetas

| Carpeta | Contenido | Índice |
|---------|----------|--------|
| **decisions/** | ADRs (Architecture Decision Records) | [decisions_index.md](decisions/decisions_index.md) |
| **patterns/** | Patrones técnicos reutilizables | [patterns_index.md](patterns/patterns_index.md) |
| **infra/** | Infraestructura, DevOps, Base de datos | [infra_index.md](infra/infra_index.md) |

## 📋 Archivos principales

| Archivo | Descripción |
|---------|-------------|
| [Arquitectura-Stack-Tecnologico.md](Arquitectura-Stack-Tecnologico.md) | Overview del stack tecnológico (Express, Vue3, PostgreSQL, etc.) |

## 🎯 Responsabilidades

Este nivel define:
- ✅ **Decisiones:** Por qué elegimos ciertas tecnologías/patrones
- ✅ **Patrones:** Cómo implementamos soluciones recurrentes
- ✅ **Infraestructura:** DevOps, CI/CD, bases de datos, deployment

## 🔗 Relaciones

- **Nivel 1 (Servicios):** [[01-SERVICIOS/servicios_index.md]] (implementan estas decisiones)
- **Nivel 2 (Dominios):** [[02-DOMINIOS/dominios_index.md]] (usan estos patrones)
- **Nivel 4 (Referencias):** [[04-REFERENCIAS/referencias_index.md]] (catálogos globales)
- **Governance:** [[06-GOVERNANCE/governance_index.md]]

---

**Tags:** #architecture
**Última actualización:** 2026-04-13
