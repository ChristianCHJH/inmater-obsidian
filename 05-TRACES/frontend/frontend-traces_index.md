# Índice: Traces - Capa Frontend

## 📋 Archivos en esta carpeta

Investigaciones que comienzan en la interfaz de usuario (componentes, botones, eventos).

| Archivo                                  | Origen                                                                    | Descripción                                                                                                    |
|------------------------------------------|---------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------|
| [[pinpad-inicializar-flow.md]]           | `PinpadStatusCard.vue` — botón "Inicializar"                              | Flujo completo de inicialización de terminal PinPad Niubiz: modal loading/success/error, statusCode humanizado |
| [[gestor-pagos-reportes-lote.md]]        | `ControlLoteSection.vue` — botones "Reporte Detallado" / "Totales"        | Flujo completo de reportes de lote Niubiz: API bridge, dialog ticket, impresión 80mm                           |
| [[voucher-anulacion-gestor-pagos.md]]    | `TransaccionesTable.vue` — "Ver Voucher" en transacción cancelada         | Fix: voucher de anulación desde `transacciones_pagos_cancelaciones` en vez del voucher original                |
| [[gestor-pagos-dev-mode-401-cascade.md]] | Carga del módulo gestor-pagos en entorno de desarrollo                    | Fix: cascada de 401 que deslogueaba al usuario; token mock inválido + interceptor axios agresivo               |

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
**Última actualización:** 2026-04-15
