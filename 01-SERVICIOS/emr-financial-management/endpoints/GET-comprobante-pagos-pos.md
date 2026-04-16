---
title: "GET /comprobantes/legacy/:id_recibo/:tip_recibo/pagos-pos"
projects: [[EMR.Financial-Management.Service]]
domain: [[facturación]], [[POS/pinpad]]
layer: backend
type: endpoint
status: documented
created: 2026-04-16
last-reviewed: 2026-04-16
tech-stack:
  - Express
  - PostgreSQL
  - Sequelize (raw queries)
backlog: false
---

# GET `/microservicesFinancialManagement/comprobantes/legacy/:id_recibo/:tip_recibo/pagos-pos`

---

## Descripción

Retorna el detalle de las transacciones POS asociadas a cada forma de pago (1, 2, 3) de un recibo legacy. Permite saber exactamente qué cobro POS Niubiz corresponde a cada forma de pago del comprobante.

**Contexto:** El formulario `pago.php` permite hasta 3 formas de pago con opción de cobro vía POS. Con la implementación de 2026-04-16, cada forma de pago puede tener un `txn_pos_N` que apunta a `transacciones_pagos.id`.

---

## Parámetros

| Parámetro | Tipo | Descripción |
|-----------|------|-------------|
| `id_recibo` | `number` | ID del recibo en `appinmater_modulo.recibos` |
| `tip_recibo` | `number` | Tipo del recibo (1=boleta, 2=factura, etc.) |

> Ambos parámetros son la PK compuesta de la tabla `recibos`.

---

## Respuesta exitosa

```json
{
  "success": true,
  "message": "Pagos POS obtenidos",
  "data": {
    "forma_pago_1": {
      "medio_pago_id": 5,
      "banco_id": 2,
      "tipo_tarjeta_id": 1,
      "monto": 500.00,
      "transaccion_pos": {
        "id": 44,
        "marca_tarjeta": "MASTERCARD",
        "banco_emisor": "BANCO RIPLEY",
        "tipo_tarjeta": "CREDITO",
        "fecha_transaccion": "2026-03-19",
        "hora_transaccion": "15:39",
        "moneda": "PEN",
        "monto": 500.00,
        "numero_cuotas": 0,
        "numero_tarjeta_mask": "****4433",
        "cancelado": false,
        "voucher_comercio": "...",
        "voucher_cliente": "..."
      }
    },
    "forma_pago_2": {
      "medio_pago_id": null,
      "banco_id": null,
      "tipo_tarjeta_id": null,
      "monto": null,
      "transaccion_pos": null
    },
    "forma_pago_3": {
      "medio_pago_id": null,
      "banco_id": null,
      "tipo_tarjeta_id": null,
      "monto": null,
      "transaccion_pos": null
    }
  }
}
```

## Respuesta de error

```json
{ "success": false, "message": "Recibo no encontrado" }
{ "success": false, "message": "id_recibo y tip_recibo deben ser números válidos" }
```

---

## Implementación

### Capas involucradas

| Capa | Archivo |
|------|---------|
| Entity | `src/modules/comprobantes/domain/entities/Comprobante.ts` — interfaces `TransaccionPosDetalle`, `FormaPagoConPos`, `PagoPosResult` |
| Repository interface | `src/modules/comprobantes/domain/repositories/Comprobante.repository.ts` — `obtenerPagosPos()` |
| Repository impl | `src/modules/comprobantes/infrastructure/repositories/Comprobante.sequelize.ts` |
| Service | `src/modules/comprobantes/application/services/Comprobante.service.ts` |
| Controller | `src/modules/comprobantes/presentation/controllers/Comprobante.controller.ts` |
| Routes | `src/modules/comprobantes/presentation/routes/Comprobante.routes.ts` |

### Queries SQL ejecutadas

**Query 1 — Recibo con IDs de transacciones:**
```sql
SELECT
  m1, p1, banco1, tipotarjeta1, txn_pos_1,
  m2, p2, banco2, tipotarjeta2, txn_pos_2,
  m3, p3, banco3, tipotarjeta3, txn_pos_3
FROM appinmater_modulo.recibos
WHERE id = :id_recibo AND tip = :tip_recibo
LIMIT 1
```

**Query 2 — Detalle de transacciones POS (solo si hay txn_pos no nulos):**
```sql
SELECT
  tp.id, tp.marca_tarjeta, tp.banco_emisor, tp.tipo_tarjeta,
  tp.fecha_transaccion, tp.hora_transaccion, tp.monto,
  tp.numero_cuotas, tp.numero_tarjeta_mask, tp.cancelado,
  tp.voucher_comercio, tp.voucher_cliente,
  (SELECT t.moneda FROM {schema_fm}.terminales t
   WHERE t.productnumber = tp.id_terminal LIMIT 1) AS moneda
FROM {schema_fm}.transacciones_pagos tp
WHERE tp.id IN (:ids)
```

> **Nota sobre `moneda`:** No está almacenada en `transacciones_pagos`. Se deriva del terminal que procesó la transacción via subquery a `terminales.moneda`. Mismo patrón usado en `TransaccionPago.sequelize.ts` para el endpoint `findRecientes`.

---

## Consideraciones

- **Cross-schema:** Query 1 usa `appinmater_modulo`, Query 2 usa `DB_SCHEMA_FINANCIAL_MANAGEMENT`. El servicio ya tiene este patrón en otros endpoints de comprobantes.
- **Históricos:** Recibos anteriores a 2026-04-16 tendrán `txn_pos_1/2/3 = NULL` → `transaccion_pos: null` en respuesta. Comportamiento correcto y esperado.
- **Sin FK formal:** No hay FOREIGN KEY entre schemas. La integridad es responsabilidad de la aplicación (PHP en el momento del INSERT/UPDATE).
- **Modelo Sede:** Se creó `src/shared/models/Sede.model.ts` en esta misma sesión para reutilización futura. Usa `DB_SCHEMA_APP_MODULO` y timestamps `createdate`/`updatex`.

---

## Relacionado

- [[01-SERVICIOS/php-legacy/pago-emision-comprobantes]] — Módulo PHP que escribe `txn_pos_1/2/3`
- [[05-TRACES/backend/pos-recibo-vinculacion]] — Trace completo del flujo
- Migración BD: `ALTER TABLE appinmater_modulo.recibos ADD COLUMN txn_pos_1 BIGINT NULL, txn_pos_2 BIGINT NULL, txn_pos_3 BIGINT NULL`
