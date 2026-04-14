# Índice: Traces - Capa Database

## 📋 Archivos en esta carpeta

Investigaciones de queries, schemas, optimizaciones, y análisis de bases de datos.

| Archivo | Tabla(s) | Descripción |
|---------|----------|-------------|
| (vacío) | - | Aquí irán las trazas de database cuando se investiguen queries |

## 📝 Cómo crear una trace de database

1. Identifica la/las tabla(s) afectadas
2. Documenta: Query SQL → Schema → Índices → Optimizaciones
3. Usa template: [[06-GOVERNANCE/templates/trace-template.md]]
4. Nombra: `{tabla-o-operacion}-query.md` (ej: `patient-table-schema.md`, `login-credentials-query.md`)

**Ejemplo:**
```
Tabla: hc_paciente
Schema: appinmater_modulo
Query: SELECT id, nombre, email FROM hc_paciente WHERE email = ?
Índices: email (índice simple)
Optimización: Usar índice en email para búsquedas
Relaciones: hc_paciente.id → hc_paciente_contact.paciente_id
```

## 🔗 Relaciones

- **Padre:** [[traces_index.md]]
- **Frontend relacionado:** [[../frontend/frontend-traces_index.md]]
- **Backend relacionado:** [[../backend/backend-traces_index.md]]
- **Template:** [[../../06-GOVERNANCE/templates/trace-template.md]]
- **Catálogo de tablas:** [[../../04-REFERENCIAS/referencias_index.md#Tables-Catalog]]

---

**Tags:** #traces #database
**Última actualización:** 2026-04-13
**Estado:** 🟡 Vacío - Pendiente llenar
