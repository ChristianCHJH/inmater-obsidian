# Índice: Traces - Capa Frontend

## 📋 Archivos en esta carpeta

Investigaciones que comienzan en la interfaz de usuario (componentes, botones, eventos).

| Archivo | Origen | Descripción |
|---------|--------|-------------|
| (vacío) | - | Aquí irán las trazas de frontend cuando se investiguen flujos |

## 📝 Cómo crear una trace de frontend

1. Identifica el componente/botón/evento que dispara el flujo
2. Documenta: Componente Vue → Composable → Axios call → API
3. Usa template: [[06-GOVERNANCE/templates/trace-template.md]]
4. Nombra: `{accion-descripliva}.md` (ej: `login-flow.md`, `patient-registration-button.md`)

**Ejemplo:**
```
Componente: LoginForm
Evento: click en botón "Ingresar"
Llamada API: POST /auth/login
Composable usado: useAuth()
Estado actualizado: Pinia authStore.user
```

## 🔗 Relaciones

- **Padre:** [[traces_index.md]]
- **Backend relacionado:** [[../backend/backend-traces_index.md]]
- **Database relacionada:** [[../database/database-traces_index.md]]
- **Template:** [[../../06-GOVERNANCE/templates/trace-template.md]]

---

**Tags:** #traces #frontend
**Última actualización:** 2026-04-13
**Estado:** 🟡 Vacío - Pendiente llenar
