# Índice: Patrones Compartidos (Reutilizables)

## 📋 Descripción

Patrones y soluciones reutilizables en **todos los servicios**. Estos no son específicos de un dominio, sino generales.

## 📋 Archivos en esta carpeta

| Patrón | Descripción | Usado por | Status |
| ------ | ----------- | --------- | ------ |
| [[user-audit-pattern]] | Cómo registrar `id_usuario_creacion` en tablas (body vs header) | EMR.Financial-Management | documented |

### Patrones pendientes de documentar

| Patrón | Descripción | Usado por |
| ------ | ----------- | --------- |
| `sequelize-setup.md` | Cómo configurar Sequelize | Todos los servicios |
| `error-responses.md` | Formato estándar de respuestas de error | Todos los servicios |
| `caching-strategy.md` | Estrategia de caché con node-cache | Todos los servicios |
| `jwt-validation.md` | Validación de JWT tokens | Gateway + Auth Service |
| `repository-pattern.md` | Implementación del Repository pattern | Todos los servicios |
| `testing-patterns.md` | Jest patterns, mocks, fixtures | Todos los servicios |

## 📝 Cómo agregar un patrón

1. Identifica que es reutilizable en **2+ servicios**
2. Documenta: **Problema** → **Solución** → **Ejemplo de código** → **Dónde se usa**
3. Nombra: `{patrón-nombre}.md`
4. Guarda en esta carpeta
5. Actualiza este índice

## 🔗 Relaciones

- **Padre:** [[dominios_index.md]]
- **Servicios que usan:** [[../../01-SERVICIOS/servicios_index.md]] (todos)
- **Patrones técnicos:** [[../../03-ARQUITECTURA/patterns/patterns_index.md]] (patrones específicos de arquitectura)

---

**Tags:** #patterns #shared #reusable
**Última actualización:** 2026-04-16
**Estado:** 🟢 1 patrón documentado
