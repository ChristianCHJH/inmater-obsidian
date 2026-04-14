# Índice: Patrones Técnicos

## 📋 Descripción

Patrones reutilizables en la arquitectura: cómo estructuramos los servicios, componentes, y patrones de implementación.

## 📋 Archivos en esta carpeta

Actualmente esta carpeta está **vacía**. Aquí irán patrones como:

- `express-ddd.md` - Estructura DDD en servicios Express
- `controller-service-repo.md` - Patrón Controller → Service → Repository
- `repository-pattern.md` - Implementación del patrón Repository
- `testing-patterns.md` - Patrones de testing con Jest
- `vue3-composition.md` - Patrones de composición en Vue 3
- `error-handling.md` - Manejo consistente de errores

## 📝 Cómo crear un patrón

1. Identifica un patrón reutilizable (lo usan 2+ servicios o componentes)
2. Documenta: **Problema** → **Solución** → **Ejemplo** → **Ventajas/Desventajas**
3. Nombra: `{nombre-patrón}.md` (ej: `repository-pattern.md`)
4. Guarda en esta carpeta
5. Actualiza este índice

**Ejemplo de patrón:**
```
Patrón: Controller-Service-Repository

Problema: Mezclar lógica de negocio con queries en controllers

Solución:
- Controller: recibe request, valida, llama a Service
- Service: ejecuta lógica de negocio, llama a Repository
- Repository: accede a datos via ORM

Implementación: Todos los servicios usan esta estructura
```

## 🔗 Relaciones

- **Padre:** [[../arquitectura_index.md]]
- **ADRs relacionados:** [[../decisions/decisions_index.md]]
- **Dominios que usan:** [[../../02-DOMINIOS/dominios_index.md]]
- **Shared patterns (reutilizables):** [[../../02-DOMINIOS/_shared-patterns/shared-patterns_index.md]]

---

**Tags:** #architecture #patterns
**Última actualización:** 2026-04-13
**Estado:** 🟡 Vacío - Pendiente documentar patrones
