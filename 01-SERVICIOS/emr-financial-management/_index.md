---
title: "EMR.Financial-Management.Service — Índice"
projects: [[EMR.Financial-Management.Service]]
layer: backend
type: index
status: documented
created: 2026-04-16
last-reviewed: 2026-04-16
---

# EMR.Financial-Management.Service — Índice

**Puerto:** variable (Cloud Run)
**Schema DB:** `DB_SCHEMA_FINANCIAL_MANAGEMENT`
**Schemas externos:** `DB_SCHEMA_APP_MODULO` (appinmater_modulo), `DB_SCHEMA_FACTURACION`, `DB_SCHEMA_FARMACIA`
**Ruta base:** `/microservicesFinancialManagement`

---

## 📋 Módulos

| Módulo | Descripción |
|--------|-------------|
| `pinpad` | Integración Niubiz: transacciones, cierres de lote, cancelaciones |
| `terminales` | CRUD de terminales PinPad |
| `comprobantes` | Listado y consulta de comprobantes (cross-schema con legacy) |
| `comisiones` | Comisiones bancarias por tipo de transacción |

---

## 📁 Endpoints documentados

| Archivo | Método | Ruta | Dominio | Status | Última revisión |
|---------|--------|------|---------|--------|-----------------|
| [GET-comprobante-pagos-pos.md](endpoints/GET-comprobante-pagos-pos.md) | GET | `/comprobantes/legacy/:id_recibo/:tip_recibo/pagos-pos` | facturación/POS | documented | 2026-04-16 |

---

## 🗄️ Tablas propias

| Tabla | Schema | Descripción |
|-------|--------|-------------|
| `transacciones_pagos` | `DB_SCHEMA_FINANCIAL_MANAGEMENT` | Transacciones POS Niubiz |
| `terminales` | `DB_SCHEMA_FINANCIAL_MANAGEMENT` | Terminales PinPad registrados |
| `cierres_lote` | `DB_SCHEMA_FINANCIAL_MANAGEMENT` | Cierres de lote por terminal |
| `transacciones_pagos_cancelaciones` | `DB_SCHEMA_FINANCIAL_MANAGEMENT` | Anulaciones de transacciones |
| `comisiones_bancarias` | `DB_SCHEMA_FINANCIAL_MANAGEMENT` | Tarifas de comisión por banco/tipo |
| `comision_defaults` | `DB_SCHEMA_FINANCIAL_MANAGEMENT` | Defaults de comisión |

## 🗄️ Tablas externas que consulta (cross-schema)

| Tabla | Schema | Uso |
|-------|--------|-----|
| `recibos` | `appinmater_modulo` | Listado comprobantes legacy, pagos POS |
| `sedes` | `appinmater_modulo` | JOIN para nombre de sede |
| `comprobantes` | `DB_SCHEMA_FACTURACION` | Listado comprobantes nuevos |
| `comprobante_pagos` | `DB_SCHEMA_FACTURACION` | Detalle de pagos por comprobante |
| `tblpos` | `DB_SCHEMA_FARMACIA` | Nombre y código de terminales POS legacy |

---

## 🔗 Relacionado

- [[01-SERVICIOS/php-legacy/pago-emision-comprobantes]] — Módulo PHP que genera los recibos y vincula txn_pos
- [[05-TRACES/backend/pos-recibo-vinculacion]] — Trace completo del flujo de vinculación
- [[04-REFERENCIAS/referencias_index]] — Catálogo global de endpoints
