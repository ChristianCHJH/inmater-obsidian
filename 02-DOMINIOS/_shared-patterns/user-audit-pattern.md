---
title: "Patrón: Auditoría de Usuario en Tablas"
projects: [[EMR.Financial-Management.Service]], [[EMR.Api-Gateway.Service]]
domain: [[shared]]
layer: backend
type: pattern
status: documented
created: 2026-04-16
last-reviewed: 2026-04-16
tech-stack:
  - Express
  - PostgreSQL
  - Sequelize
  - JWT
backlog: false
---

# Patrón: Auditoría de Usuario en Tablas

## Problema

Toda tabla que registra operaciones de negocio debe saber qué usuario las ejecutó. El sistema usa JWT para autenticación, y el Gateway inyecta headers con los datos del token. La pregunta es: ¿desde dónde debe llegar el `id_usuario_creacion` al servicio — del body o del header?

---

## Dos variantes del patrón

### Variante A — Tablas de negocio (body)

**Cuándo usarla:** la operación es iniciada explícitamente por el usuario desde la UI (cobro, cancelación, creación de registro).

El frontend obtiene el user ID del store de autenticación y lo envía en el body:

```typescript
// Frontend (Vue 3 + Pinia)
await registrarPago(monto, Number(authStore.currentUser!.sub), respuestaPinpad)

// Body enviado
{ monto_cobrado: 150, usuario_caja_id: 42, respuesta_pinpad: {...} }
```

```typescript
// Backend — DTO
interface CreateTransaccionPagoDto {
  monto_cobrado: number;
  usuario_caja_id: number;
  respuesta_pinpad: NiubizPinpadResponse;
}

// Backend — Service
const transaccion = {
  id_usuario_creacion: dto.usuario_caja_id,
  nombre_usuario_creacion: nombreUsuario, // del header x-user-username
}
```

**Columnas resultantes en la tabla:**

| Columna | Tipo | Fuente |
| ------- | ---- | ------ |
| `id_usuario_creacion` | INTEGER NOT NULL | body `usuario_caja_id` |
| `nombre_usuario_creacion` | VARCHAR nullable | header `x-user-username` |
| `id_usuario_actualizacion` | INTEGER nullable | body en operación de update |

---

### Variante B — Tablas de log/auditoría (header)

**Cuándo usarla:** la operación es infraestructura de trazabilidad — no debe depender de que el frontend la invoque correctamente.

El controller extrae el user ID directamente del header `x-user-id` que el Gateway inyecta desde el JWT:

```typescript
// Backend — Controller
public create = async (req: Request, res: Response) => {
  const rawUserId = req.headers['x-user-id'];
  const id_usuario_creacion = rawUserId ? parseInt(rawUserId as string, 10) || null : null;
  await this.service.registrar({ ...req.body, id_usuario_creacion });
}
```

**Columnas resultantes en la tabla:**

| Columna | Tipo | Fuente |
| ------- | ---- | ------ |
| `id_usuario_creacion` | INTEGER nullable | header `x-user-id` (Gateway) |

> `nullable` porque el log puede registrarse desde contextos donde no hay usuario (ej: llamadas internas, health checks).

---

## Headers que inyecta el Gateway

El API Gateway extrae estos valores del JWT y los agrega como headers a todas las peticiones forward:

| Header | Fuente en JWT | Descripción |
| ------ | ------------- | ----------- |
| `x-user-id` | `payload.sub` | ID del usuario autenticado |
| `x-user-username` | `payload.nom` | Nombre del usuario |
| `x-user-sede-id` | `payload.sedeId` | ID de la sede seleccionada |
| `x-user-sede-name` | `payload.sedeName` | Nombre de la sede |
| `x-user-roles` | `payload.roles` | Roles del usuario (JSON array) |

Archivo: `services/EMR.Api-Gateway.Service/src/index.ts` líneas 94-98.

---

## Convención de nombres de columnas

```sql
id_usuario_creacion      INTEGER  -- quien creó el registro
nombre_usuario_creacion  VARCHAR  -- nombre legible (desnormalizado para consultas rápidas)
fecha_creacion           TIMESTAMP DEFAULT NOW()
id_usuario_actualizacion INTEGER  -- quien hizo el último update (nullable)
fecha_actualizacion      TIMESTAMP  -- nullable
```

---

## Dónde se aplica este patrón

| Tabla | Variante | Observaciones |
| ----- | -------- | ------------- |
| `transacciones_pagos` | A (body) | Tiene `id` y `nombre` + campos de actualización |
| `transacciones_pagos_cancelaciones` | A (body) | Solo `id_usuario_creacion` |
| `log_peticiones_pinpad` | B (header) | Agregado 2026-04-16, nullable |

---

## Implementado por

- [[01-SERVICIOS/emr-financial-management/_index]] — módulo Pinpad

## Relacionado

- [[02-DOMINIOS/facturacion/gestor-pagos]] — contexto de negocio del módulo Pinpad
- [[03-ARQUITECTURA/patterns/patterns_index]] — otros patrones técnicos
