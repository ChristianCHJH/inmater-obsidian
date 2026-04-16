---
title: "Trace — Vinculación POS en módulo legacy pago.php"
projects: [[apps/web]], [[EMR.Financial-Management.Service]]
domain: [[02-DOMINIOS/facturacion/gestor-pagos]]
layer: legacy-frontend
type: trace
status: documented
created: 2026-04-16
last-reviewed: 2026-04-16
tech-stack:
  - PHP
  - jQuery Mobile
  - jQuery
  - NestJS
  - Sequelize
backlog: false
---

# Trace — Vinculación POS en módulo legacy pago.php

Flujo completo de vinculación de transacciones POS (Niubiz/PinPad) a las formas de pago del módulo de emisión de comprobantes (`apps/web/pago.php`).

---

## Contexto

El formulario de pago tiene 3 slots de forma de pago. Cada slot puede vincularse a una transacción POS real (cobrada con tarjeta) de dos maneras:

1. **Búsqueda manual**: el cajero ingresa el ID de una transacción existente y hace click en "Vincular"
2. **Cobro directo**: el cajero abre el modal "Cobrar POS", selecciona terminal y el sistema cobra en tiempo real

---

## Flujo A — Búsqueda + Vincular

```
[Cajero] ingresa ID en #buscar_txn_pos_{n}
    → click "Buscar" → buscarTxnPOS(slot)
        → POST a TransaccionesPago.service.php
            { endpoint: 'recientes', data: { search: id } }
            → requestToRouting('/microservicesFinancialManagement/pinpad/transaccionesPagos/recientes?search={id}')
                → EMR.Financial-Management.Service
                    → TransaccionPagoController.recientes()
                    → TransaccionPagoService.obtenerRecientes(search)
                    → TransaccionPagoSequelize.findRecientes(search)
                        → SELECT id, monto, marca_tarjeta, banco_emisor, tipo_tarjeta,
                                 numero_tarjeta_mask, numero_cuotas, fecha_transaccion,
                                 hora_transaccion, codigo_autorizacion,
                                 (SELECT moneda FROM terminales...) AS moneda
                           FROM transacciones_pagos
                           WHERE cancelado = false AND id = {search}
                           ORDER BY fecha_creacion DESC LIMIT 10
                    ← { success: true, data: [ { id, marca_tarjeta, banco_emisor,
                                                  tipo_tarjeta, monto, moneda,
                                                  numero_tarjeta_mask, numero_cuotas,
                                                  fecha_transaccion, hora_transaccion,
                                                  codigo_autorizacion } ] }
        ← lista de resultados renderizada en #resultado_txn_pos_{n}

[Cajero] click "Vincular" en un resultado
    → onclick: seleccionarTxnPOS(slot, id, monto, marca, ultimos4,
                                  tipoTarjeta, issuingBank, numeroCuotas,
                                  moneda, codigoAutorizacion)
```

---

## `seleccionarTxnPOS` — efectos al vincular

```javascript
function seleccionarTxnPOS(slot, id, monto, marca, ultimos4,
                            tipoTarjeta, issuingBank, numeroCuotas,
                            moneda, codigoAutorizacion)
```

Al ejecutarse, produce los siguientes efectos en la UI:

| Acción | Detalle |
|--------|---------|
| `#txn_pos_{slot}` = id | Hidden field que va en el POST del formulario |
| Badge visible | `#txn_seleccionada_{slot}` muestra "PAY-NNN \| S/monto \| MARCA *ultimos4 ×" |
| Tipo de pago `#t{slot}` | Se selecciona la opción cuyo texto contiene la marca (ej: "Mastercard") |
| Banco `#banco{slot}` | Se selecciona por nombre (fuzzy match); fallback a "Otros" si no coincide |
| Tipo tarjeta `#tipotarjeta{slot}` | Se selecciona "CREDITO" o "DEBITO" según `tipoTarjeta` |
| Cuotas `#numerocuotas{slot}` | `0` si DEBITO; `numeroCuotas` si CREDITO |
| Moneda `#m{slot}` | `1` si USD, `0` si PEN |
| Monto `#p{slot}` | = monto de la transacción (inmutable mientras esté vinculada) |
| `#comprobante_referencia` | Se agrega `codigoAutorizacion` al sistema `_auth_codes` |
| Sección buscar | `.parent().hide()` — oculta input + botón Buscar |
| Botón Cobrar POS | `#btn_cobrar_pos_{slot}.hide()` |
| Selector terminal | `#poss{slot}` → `.closest('.ui-select').hide()` |
| Campos bloqueados | `.campo-txn-vinculada` (selects) y `.campo-txn-vinculada-input` (inputs) |

### CSS de bloqueo

```css
.campo-txn-vinculada .ui-btn {
    pointer-events: none !important;
    opacity: 0.6 !important;
}
input.campo-txn-vinculada-input {
    pointer-events: none !important;
    opacity: 0.6 !important;
    cursor: default !important;
}
```

Los selects JQM se bloquean visualmente pero el `<select>` subyacente sigue habilitado → el valor se incluye en el POST.

---

## Flujo B — Cobrar POS (cobro directo desde la UI)

```
[Cajero] click "Cobrar POS" (btn_cobrar_pos_{slot})
    → abrirModalCobro(slot)
        → _calcularMontoSugerido(slot)
            = total_descuento - (suma de otros slots)
        → modal pre-rellenado con monto sugerido
        → [Cajero selecciona terminal PinPad]
        → click "Iniciar cobro" → iniciarCobroPOS()
            → POST a CobroPOS.service.php
                { endpoint: 'pclSale', monto, terminal }
                → bridge PinPad local (localhost:8679 o similar)
                    → terminal física Niubiz
            ← Éxito: respuesta Niubiz con authNumber, cardType, maskedPAN, etc.
                → _registrarTransaccionPOS(monto, respuestaPinpad)
                    → POST a TransaccionesPago.service.php
                        { endpoint: 'recientes' } (para obtener el ID creado)
                    → seleccionarTxnPOS(_cobro_slot, txnId, monto, marca,
                                        ultimos4, tipoTarjeta, issuingBank)
                        → mismos efectos que Flujo A
                    → _actualizarReferenciaAutorizacion(_cobro_slot, authNumber)
            ← Fallo: _activarEstadoPendiente()
                → guarda en localStorage (key pos_pending_*)
                → reintento automático x3 con timer
                → banner "Pendiente" visible al cajero
```

---

## Sistema `_auth_codes` — Número de comprobante de referencia

El campo `#comprobante_referencia` se construye dinámicamente desde:

```javascript
var _auth_codes = { 1: null, 2: null, 3: null };

function _actualizarReferenciaAutorizacion(slot, codigo) {
    _auth_codes[slot] = codigo || null;
    var partes = [];
    [1, 2, 3].forEach(function(s) {
        if (_auth_codes[s]) partes.push(_auth_codes[s]);
    });
    document.getElementById('comprobante_referencia').value = partes.join(' / ');
}
```

**Ejemplos:**

| Estado | Valor del campo |
|--------|----------------|
| Slot 1 vinculado (260438) | `"260438"` |
| Slots 1 y 2 vinculados (260438, 141125) | `"260438 / 141125"` |
| Se desvincula slot 1 | `"141125"` |
| Se reemplaza slot 2 por otro código | Reconstruye sin duplicar |

**Fuentes del código de autorización:**
- Flujo A: campo `codigo_autorizacion` del endpoint `/recientes`
- Flujo B: campo `authNumber` de la respuesta del bridge PinPad

---

## Flujo de desvinculación (`limpiarTxnPOS`)

```
[Cajero] click × en badge de transacción vinculada
    → limpiarTxnPOS(slot)
        → #txn_pos_{slot} = ''
        → #resultado_txn_pos_{slot} = ''
        → #txn_seleccionada_{slot} = ''
        → #buscar_txn_pos_{slot} = ''
        → _actualizarReferenciaAutorizacion(slot, null)
        → Resetea selects: #t{slot}, #banco{slot}, #tipotarjeta{slot}, #m{slot}
            → selectedIndex = 0 + .trigger('change') para refrescar JQM
        → #numerocuotas{slot} = ''
        → #poss{slot} → .closest('.ui-select').show()
        → _desbloquearCamposPago(slot):
            → remueve .campo-txn-vinculada de selects
            → remueve readonly y .campo-txn-vinculada-input de inputs
            → #buscar_txn_pos_{slot}.parent().show()
            → #btn_cobrar_pos_{slot}.show()
        → [monto #p{slot} NO se limpia]
```

---

## Regla de negocio: monto de slot vinculado es inmutable ante cambios de servicio

Si `#txn_pos_1` tiene valor y el usuario cambia los servicios:

- `#tot` se actualiza con el nuevo total de servicios
- `#p1` **NO** se actualiza — mantiene el monto real cobrado

**Implementación:** funciones guard que reemplazan `$("#tot, #p1").val(X)`:

```javascript
function _setTotYP1(valor) {
    $("#tot").val(valor);
    if (!$("#txn_pos_1").val()) $("#p1").val(valor);
}
```

Todos los lugares en `pago_total.php` (9 ocurrencias) y `pago.php` (4 ocurrencias) que actualizaban `#p1` fueron refactorizados para respetar esta regla.

> **Nota:** esta protección solo aplica a slot 1 (`#p1`) porque es el único que hereda el total automáticamente al cargar el formulario. Los slots 2 y 3 empiezan vacíos.

---

## Guardado en BD

Al submit del formulario `#form2`:

```
POST pago.php (self)
    $_POST['txn_pos_1'] → intval() o null si vacío
    $_POST['txn_pos_2'] → ídem
    $_POST['txn_pos_3'] → ídem
    → recibo() en _database/pago.php
        → INSERT/UPDATE appinmater_modulo.recibos
            txn_pos_1, txn_pos_2, txn_pos_3 (BIGINT NULL)
        → INSERT appinmater_log.recibos (action='I' o 'U')
```

---

## Relacionado

- Doc módulo: [[01-SERVICIOS/php-legacy/pago-emision-comprobantes]]
- Trace backend lectura: [[../backend/pos-recibo-vinculacion]]
- Endpoint recientes: `GET /microservicesFinancialManagement/pinpad/transaccionesPagos/recientes`
- Bridge PinPad: `/_Microservices/EMR/Services/CobroPOS.service.php`
- Dominio pagos: [[../../02-DOMINIOS/facturacion/gestor-pagos]]
