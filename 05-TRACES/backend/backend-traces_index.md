# Índice: Traces - Capa Backend

## 📋 Archivos en esta carpeta

Investigaciones de APIs, controllers, servicios, y lógica de negocio backend.

| Archivo | Servicio | Descripción |
|---------|----------|-------------|
| (vacío) | - | Aquí irán las trazas de backend cuando se investiguen flujos |

## 📝 Cómo crear una trace de backend

1. Identifica el endpoint/controller que procesa la request
2. Documenta: Controller → Service → Repository → Query DB
3. Usa template: [[06-GOVERNANCE/templates/trace-template.md]]
4. Nombra: `{accion-descriptiva}-flow.md` (ej: `jwt-validation-flow.md`, `create-patient-flow.md`)

**Ejemplo:**
```
Endpoint: POST /auth/login
Controller: AuthController.login()
Service: AuthService.validateCredentials()
Repository: UserRepository.findByEmail()
Tabla: usuarios
Query: SELECT * FROM usuarios WHERE email = ?
```

## 🔗 Relaciones

- **Padre:** [[traces_index.md]]
- **Frontend relacionado:** [[../frontend/frontend-traces_index.md]]
- **Database relacionada:** [[../database/database-traces_index.md]]
- **Template:** [[../../06-GOVERNANCE/templates/trace-template.md]]

---

**Tags:** #traces #backend
**Última actualización:** 2026-04-13
**Estado:** 🟡 Vacío - Pendiente llenar
