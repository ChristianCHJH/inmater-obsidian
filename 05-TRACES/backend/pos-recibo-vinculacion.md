---
title: "Trace: Vinculación transacción POS con forma de pago del recibo"
projects: [[apps/web]], [[EMR.Financial-Management.Service]], [[nd-be-pacientes]]
domain: [[facturación]], [[POS/pinpad]]
layer: backend
type: trace
status: documented
created: 2026-04-16
last-reviewed: 2026-04-16
tech-stack:
  - PHP
  - Node.js
  - PostgreSQL
  - Sequelize
  - jQuery
backlog: false
---

# Trace: Vinculación transacción POS → forma de pago del recibo

---

## Problema que resuelve

El formulario `pago.php` permite hasta 3 formas de pago. Cada una puede cobrarse vía POS Niubiz. Sin este mecanismo, si un recibo tiene 2 cobros POS, es imposible saber cuál corresponde a la Forma de Pago 1 y cuál a la 2.

**Antes (sin vincular):** recibo tiene `id_transaccion_pago1 = NULL`, `id_transaccion_pago2 = NULL`, `id_transaccion_pago3 = NULL` aunque haya transacciones POS con `id_comprobante = id_recibo`.

**Después (vinculado):** recibo tiene `id_transaccion_pago1 = 44`, `id_transaccion_pago2 = 51`, `id_transaccion_pago3 = NULL` — se sabe exactamente qué transacción corresponde a cada forma de pago.

---

## Solución implementada

Agregar columnas `id_transaccion_pago1`, `id_transaccion_pago2`, `id_transaccion_pago3` (BIGINT NULL) en `appinmater_modulo.recibos`. Simétrico con la estructura existente (`m1/p1/banco1`, `m2/p2/banco2`, `m3/p3/banco3`).

**SQL de migración:**
```sql
ALTER TABLE appinmater_modulo.recibos
  ADD COLUMN id_transaccion_pago1 BIGINT NULL,
  ADD COLUMN id_transaccion_pago2 BIGINT NULL,
  ADD COLUMN id_transaccion_pago3 BIGINT NULL;
```

---

## Flujo completo

### Paso 1 — Usuario cobra con POS en `pago.php`

Dos caminos para vincular una transacción a una forma de pago:

#### Camino A: "Cobrar POS" (modal)
```
Usuario hace clic en "Cobrar POS" (slot N)
  → abrirModalCobro(N)
    → iniciarCobroPOS()
      → POST a CobroPOS.service.php {endpoint: 'pclSale'}
        → Éxito (statusCode='00')
          → _registrarTransaccionPOS(monto, respuestaPinpad)
            → POST a CobroPOS.service.php {endpoint: 'registrarTransaccion'}
              → Registra en transacciones_pagos
              → Retorna { id, marca_tarjeta, monto, ... }
            → seleccionarTxnPOS(slot, txnId, ...)
              → $("#txn_pos_" + slot).val(txnId)  ← SET hidden input
```

#### Camino B: "Buscar" por ID
```
Usuario ingresa ID en buscar_txn_pos_{N}
  → buscarTxnPOS(N)
    → POST a TransaccionesPago.service.php {endpoint: 'recientes'}
      → Muestra lista de transacciones recientes
    → Usuario hace clic "Vincular" en una transacción
      → seleccionarTxnPOS(slot, txnId, ...)
        → $("#txn_pos_" + slot).val(txnId)  ← SET hidden input
```

### Paso 2 — Usuario presiona GUARDAR

```
Submit del #form2
  → POST a pago.php
    → $data['id_transaccion_pago1'] = intval($_POST['id_transaccion_pago1']) o null
    → $data['id_transaccion_pago2'] = intval($_POST['id_transaccion_pago2']) o null
    → $data['id_transaccion_pago3'] = intval($_POST['id_transaccion_pago3']) o null
    → recibo($id, ..., $datos = $data)
```

### Paso 3 — `_database/pago.php`: función `recibo()`

#### Si es INSERT (recibo nuevo):
```sql
INSERT INTO recibos (..., id_transaccion_pago1, id_transaccion_pago2, id_transaccion_pago3)
VALUES (..., $txnPos1, $txnPos2, $txnPos3)
```

#### Si es UPDATE (recibo existente):
```sql
UPDATE recibos
SET ..., id_transaccion_pago1=?, id_transaccion_pago2=?, id_transaccion_pago3=?
WHERE id=? AND tip=?
```

> **Nota:** El UPDATE de `txn_pos_N` fue el gap que se implementó en esta sesión. El INSERT ya existía.

### Paso 4 — Leer los datos (nuevo endpoint)

```
GET /microservicesFinancialManagement/comprobantes/legacy/{id_recibo}/{tip_recibo}/pagos-pos
  → ComprobanteController.obtenerPagosPos()
    → ComprobanteService.obtenerPagosPos(idRecibo, tipRecibo)
      → ComprobanteSequelize.obtenerPagosPos()
        → Query 1: SELECT id_transaccion_pago1/2/3 FROM appinmater_modulo.recibos WHERE id=? AND tip=?
        → Query 2: SELECT ... FROM {schema_fm}.transacciones_pagos WHERE id IN (ids)
          → Subquery moneda: SELECT t.moneda FROM terminales t WHERE t.productnumber = tp.id_terminal
        → Retorna PagoPosResult { forma_pago_1, forma_pago_2, forma_pago_3 }
```

---

## Tablas afectadas

| Tabla | Schema | Operación | Campo nuevo |
|-------|--------|-----------|-------------|
| `recibos` | `appinmater_modulo` | INSERT + UPDATE | `id_transaccion_pago1`, `id_transaccion_pago2`, `id_transaccion_pago3` |
| `transacciones_pagos` | `DB_SCHEMA_FINANCIAL_MANAGEMENT` | SELECT | — |
| `terminales` | `DB_SCHEMA_FINANCIAL_MANAGEMENT` | SELECT (subquery moneda) | — |

---

## Archivos modificados en la implementación

| Archivo | Cambio |
|---------|--------|
| `services_legacy/nd-be-pacientes/src/models/facturacion.ts` | +3 campos `id_transaccion_pago1/2/3` al modelo `Recibos` |
| `apps/web/_database/pago.php` | UPDATE incluye `id_transaccion_pago1=?, id_transaccion_pago2=?, id_transaccion_pago3=?` |
| `services/EMR.Financial-Management.Service/src/shared/models/Sede.model.ts` | Nuevo modelo Sequelize para tabla `sedes` |
| `src/modules/comprobantes/domain/entities/Comprobante.ts` | +interfaces `TransaccionPosDetalle`, `FormaPagoConPos`, `PagoPosResult` |
| `src/modules/comprobantes/domain/repositories/Comprobante.repository.ts` | +método `obtenerPagosPos()` |
| `src/modules/comprobantes/infrastructure/repositories/Comprobante.sequelize.ts` | +implementación con 2 queries cross-schema |
| `src/modules/comprobantes/application/services/Comprobante.service.ts` | +delegación al repositorio |
| `src/modules/comprobantes/presentation/controllers/Comprobante.controller.ts` | +handler con validación de params |
| `src/modules/comprobantes/presentation/routes/Comprobante.routes.ts` | +ruta `GET /legacy/:id_recibo/:tip_recibo/pagos-pos` |

---

## Dato clave: `moneda` no está en `transacciones_pagos`

La columna `moneda` no existe en la tabla `transacciones_pagos`. Se deriva del terminal que procesó la transacción:

```sql
(SELECT t.moneda FROM {schema_fm}.terminales t
 WHERE t.productnumber = tp.id_terminal LIMIT 1) AS moneda
```

Mismo patrón que ya usaba `findRecientes()` en `TransaccionPago.sequelize.ts`.

---

## Relacionado

- [[01-SERVICIOS/php-legacy/pago-emision-comprobantes]] — Módulo PHP origen
- [[01-SERVICIOS/emr-financial-management/endpoints/GET-comprobante-pagos-pos]] — Endpoint de lectura
- [[05-TRACES/frontend/pinpad-inicializar-flow]] — Flujo de inicialización del terminal
