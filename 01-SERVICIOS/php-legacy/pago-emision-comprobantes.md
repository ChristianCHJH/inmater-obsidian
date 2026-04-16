---
title: "PHP Legacy — Módulo de Emisión de Comprobantes (pago.php)"
projects: [[apps/web]]
domain: [[02-DOMINIOS/facturacion/gestor-pagos]]
layer: legacy-frontend
type: domain
status: documented
created: 2026-04-16
last-reviewed: 2026-04-16
sessions:
  - 2026-04-16: fixes servicio gratuito, bancos, monto sugerido modal
  - 2026-04-16: vinculación POS completa (codigo_autorizacion, bloqueo campos, regla monto fijo)
tech-stack:
  - PHP
  - jQuery Mobile
  - jQuery
backlog: false
---

# PHP Legacy — Módulo de Emisión de Comprobantes

**URL:** `http://app.inmater.pe/pago.php?id=&t=&s={tipo_servicio}`
**Repositorio:** `apps/web/` (Bitbucket)

---

## Descripción

Módulo PHP (legacy) que maneja la emisión de comprobantes de pago (boletas/facturas) para la clínica. Permite seleccionar servicios, configurar formas de pago y generar el comprobante electrónico vía facturación electrónica.

No es un microservicio NestJS ni un componente Vue 3 — es PHP puro con jQuery Mobile como framework de UI.

---

## Archivos principales

| Archivo | Rol |
|---------|-----|
| `pago.php` | Vista principal: HTML + PHP + includes de componentes |
| `_database/pago.php` | Backend: función `recibo()`, INSERT/UPDATE en `appinmater_modulo.recibos` |
| `_componentes/pago.php` | JS: eventos de búsqueda de paciente, cálculos de totales, validaciones |
| `_componentes/pago_ini.php` | JS: inicialización del formulario, POS, modal "Cobrar POS", bridge PinPad |
| `_componentes/pago_form.php` | JS: validación del submit del formulario `#form2` |
| `_componentes/pago_total.php` | JS: recarga de servicios, selección de procedimiento/tarifario |

---

## Flujo principal

```
1. Usuario abre pago.php?s={tipo_servicio}
2. Selecciona empresa → sede → tipo de comprobante (boleta/factura)
3. Selecciona paciente (búsqueda AJAX en le_tanque.php)
4. Selecciona sede contabilidad → tarifario → procedimiento → servicios
5. Configura condición de pago (Al Contado / Al Crédito)
6. Configura formas de pago (hasta 3: Forma de Pago 1, 2, 3)
7. Submit → POST a sí mismo → función recibo() → INSERT en BD + facturación electrónica
```

---

## Sección "Condiciones de Pago"

### Checkbox: Servicio Gratuito

- **ID:** `#check_serviciogratuito`
- **Nombre POST:** `check_serviciogratuito[]`
- **Campo DB:** `recibos.gratuito` (int, 0 o 1)

**Comportamiento al marcar:**
- `#total_descuento` → `"0"` (Total a pagar = 0)
- `#p1` → `"0"` (monto Forma de Pago 1 = 0)
- `#porcentaje_descuento, #descuento, #total_cancelar, #vuelto` → vacíos
- `#check_descuento` → deshabilitado y desmarcado
- Alerta de medios de pago → oculta

**Comportamiento al desmarcar:**
- `#total_descuento` y `#p1` → restaurados al valor de `#tot`
- `#check_descuento` → habilitado de nuevo

> **Bug corregido (2026-04-16):** Antes, al marcar "Servicio gratuito", `#total_descuento` NO se ponía en 0. La fórmula incorrecta era `(tot - descuento)` con `descuento` ya vaciado, lo que mantenía el total original.

---

## Sección "Medios de Pago"

Tres formas de pago paralelas (`Forma de Pago 1`, `2`, `3`). Cada una tiene:

| Campo HTML | ID | Descripción |
|------------|-----|-------------|
| Tipo de pago | `#t1`, `#t2`, `#t3` | Forma de pago (efectivo, tarjeta, etc.) |
| Banco | `#banco1`, `#banco2`, `#banco3` | Banco emisor |
| Tipo tarjeta | `#tipotarjeta1`, `#tipotarjeta2`, `#tipotarjeta3` | Débito / Crédito |
| Cuotas | `#numerocuotas1`, `#numerocuotas2`, `#numerocuotas3` | Número de cuotas |
| Moneda | `#m1`, `#m2`, `#m3` | 1=USD, 0=PEN |
| Monto | `#p1`, `#p2`, `#p3` | Monto del pago |
| POS | `#poss1`, `#poss2`, `#poss3` | Terminal POS seleccionado |
| Transacción POS | `#txn_pos_1`, `#txn_pos_2`, `#txn_pos_3` | ID de transacción vinculada |

### Opciones de banco (igual en las 3 formas)

| value | Banco |
|-------|-------|
| 1 | BBVA |
| 2 | BCP |
| 3 | Dinners Club |
| 4 | Interbank |
| 6 | OH |
| 7 | Scotiabank |
| 5 | Otros |

> **Bug corregido (2026-04-16):** `#banco2` y `#banco3` no tenían las opciones `OH` (value=6) y `Scotiabank` (value=7). Solo `#banco1` las tenía. Se igualaron las tres.

### Tipos de pago POS

Los tipos `[4, 5, 6, 8, 9]` son pagos via POS/tarjeta. Cuando se selecciona uno de estos:
- Se muestra el bloque `#vincular-txn-pos-{n}` (buscar transacción + "Cobrar POS")
- Se habilita la selección de banco, cuotas y terminal POS

Para tipos que NO son POS (ej: efectivo), el bloque POS se oculta (`verificarMostrarBuscadorPOS()`).

---

## Modal "Nuevo Cobro con POS"

Permite iniciar un cobro directamente desde la UI hacia el bridge PinPad.

### Función: `_calcularMontoSugerido(slot)`

Calcula el monto a pre-rellenar en el modal.

**Lógica actual (post fix 2026-04-16):**
```javascript
function _calcularMontoSugerido(slot) {
    var total = parseFloat(document.getElementById('total_descuento').value) || 0;
    var pagado = 0;
    for (var i = 1; i <= 3; i++) {
        if (i === slot) continue;
        pagado += parseFloat(document.getElementById('p' + i).value) || 0;
    }
    return Math.max(total - pagado, 0);
}
```

Para cualquier slot: `total_descuento - (suma de los otros slots)`.

> **Bug corregido (2026-04-16):** Para el slot 1, la función devolvía el valor de `#p1` si era > 0, ignorando `total_descuento`. Esto causaba que el modal mostrara el monto ingresado en `#p1` (ej: 3720) en lugar del "Total a pagar" real (ej: 3725).

### Flujo del modal

```
abrirModalCobro(slot)
  → _calcularMontoSugerido(slot) → monto pre-rellenado
  → Usuario selecciona terminal PinPad
  → iniciarCobroPOS()
    → POST a CobroPOS.service.php (endpoint: pclSale)
      → Éxito: _registrarTransaccionPOS() → seleccionarTxnPOS()
      → Fallo: _activarEstadoPendiente() (reintento automático x3)
```

---

## Campos de la tabla `recibos` (resumen relevante)

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `id` | int | ID del comprobante |
| `tip` | int | Tipo (1=boleta, 2=factura) |
| `gratuito` | int | 1 si es servicio gratuito |
| `descuento` | decimal | Monto de descuento |
| `tot` | decimal | Total bruto |
| `total_cancelar` | decimal | Lo que el paciente paga en efectivo |
| `t1/t2/t3` | int | Tipo de pago 1/2/3 |
| `m1/m2/m3` | int | Moneda 1/2/3 |
| `p1/p2/p3` | decimal | Monto 1/2/3 |
| `banco1/banco2/banco3` | int | Banco 1/2/3 |
| `pos1_id/pos2_id/pos3_id` | int | Terminal POS 1/2/3 |
| `txn_pos_1/txn_pos_2/txn_pos_3` | int | ID transacción POS vinculada |

---

## Log de auditoría

Cada INSERT/UPDATE en `recibos` también inserta en `appinmater_log.recibos` con `action='I'` (insert) o `action='U'` (update).

---

## Vinculación POS → Forma de pago (2026-04-16)

Las columnas `txn_pos_1`, `txn_pos_2`, `txn_pos_3` (BIGINT NULL) fueron agregadas a `appinmater_modulo.recibos` para registrar qué transacción POS corresponde a cada forma de pago.

**SQL de migración:**
```sql
ALTER TABLE appinmater_modulo.recibos
  ADD COLUMN txn_pos_1 BIGINT NULL,
  ADD COLUMN txn_pos_2 BIGINT NULL,
  ADD COLUMN txn_pos_3 BIGINT NULL;
```

**Flujo de escritura (pago.php + _database/pago.php):**
- `seleccionarTxnPOS(slot, id, monto, marca, ultimos4, tipoTarjeta, issuingBank, numeroCuotas, moneda, codigoAutorizacion)` setea `#txn_pos_{slot}` y todos los campos del slot al vincular o cobrar
- El form envía `txn_pos_1/2/3` por POST
- El INSERT y el UPDATE en `_database/pago.php` guardan los valores en BD
- Si no hay transacción POS → se guarda `NULL`

**Flujo de lectura (EMR.Financial-Management.Service):**
```
GET /microservicesFinancialManagement/comprobantes/legacy/{id}/{tip}/pagos-pos
```
Retorna `forma_pago_1/2/3` con detalle de `transaccion_pos` o `null` si no aplica.

Ver trace completo: [[05-TRACES/backend/pos-recibo-vinculacion]]
Ver flujo UI completo: [[05-TRACES/frontend/pago-legacy-vinculacion-pos]]

---

## Sistema de bloqueo de campos al vincular POS (2026-04-16)

Cuando una transacción queda vinculada a un slot, la UI aplica bloqueo visual y funcional:

### Campos bloqueados (no editables)
| Campo | Mecanismo | Razón |
|-------|-----------|-------|
| Tipo de pago (`#t{n}`) | CSS `pointer-events: none; opacity: 0.6` vía clase `.campo-txn-vinculada` | Select JQM no se puede hacer `disabled` sin bloquear el POST |
| Banco (`#banco{n}`) | ídem | ídem |
| Tipo tarjeta (`#tipotarjeta{n}`) | ídem | ídem |
| Moneda (`#m{n}`) | ídem | ídem |
| Cuotas (`#numerocuotas{n}`) | `readonly` + clase `.campo-txn-vinculada-input` | Preserva valor en POST |
| Monto (`#p{n}`) | `readonly` + clase `.campo-txn-vinculada-input` | ídem |

### Sección oculta al vincular
- Row del buscador (input "ID transaccion..." + botón "Buscar"): `$('#buscar_txn_pos_' + slot).parent().hide()`
- Botón "Cobrar POS" (`#btn_cobrar_pos_{n}`): `.hide()`
- Selector de terminal POS (`#poss{n}`): `.closest('.ui-select').hide()`

Todo se restaura al hacer × (`limpiarTxnPOS`).

### CSS aplicado
```css
/* pago_ini.php <style> block */
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

---

## Sistema de códigos de autorización (`_auth_codes`) (2026-04-16)

El campo `#comprobante_referencia` ("Número de comprobante de referencia") se pobla automáticamente con los códigos de autorización POS de cada slot vinculado.

```javascript
var _auth_codes = { 1: null, 2: null, 3: null };

function _actualizarReferenciaAutorizacion(slot, codigo) {
    _auth_codes[slot] = codigo || null;
    var partes = [];
    [1, 2, 3].forEach(function(s) {
        if (_auth_codes[s]) partes.push(_auth_codes[s]);
    });
    var inputRef = document.getElementById('comprobante_referencia');
    if (inputRef) inputRef.value = partes.join(' / ');
}
```

**Comportamiento:**
- Vincular slot 1 (código "260438") → campo: `"260438"`
- Vincular slot 2 (código "141125") → campo: `"260438 / 141125"`
- Desvincular slot 1 → campo: `"141125"`
- Reemplazar slot 2 con nuevo código → campo se reconstruye sin duplicar

**Bug corregido:** antes se concatenaba siempre (`valorActual + ' / ' + authNumber`), causando duplicados al reemplazar transacciones.

**Fuente del código:** 
- Flujo "Buscar + Vincular": `codigo_autorizacion` viene del endpoint `/recientes` de `EMR.Financial-Management.Service`
- Flujo "Cobrar POS": `authNumber` viene directamente de la respuesta del bridge PinPad Niubiz

---

## Regla de negocio: monto de slot vinculado es inmutable (2026-04-16)

**Regla:** Si `#txn_pos_{n}` tiene valor (transacción vinculada), el campo `#p{n}` no se actualiza cuando el usuario cambia los servicios o el total.

**Razonamiento:** La transacción POS representa dinero real debitado de la tarjeta. Ese monto es un hecho consumado. Cambiar los servicios no deshace el cobro.

**Implementación — funciones guard en `_componentes/pago_ini.php`:**
```javascript
function _setTotYP1(valor) {
    $("#tot").val(valor);                              // siempre se actualiza
    if (!$("#txn_pos_1").val()) $("#p1").val(valor);   // solo si no hay txn vinculada
}

function _setP1SiNoVinculada(valor) {
    if (!$("#txn_pos_1").val()) $("#p1").val(valor);
}
```

**Puntos de aplicación:**
| Archivo | Patrón original | Reemplazado por |
|---------|----------------|-----------------|
| `_componentes/pago_total.php` (9 lugares) | `$("#tot, #p1").val(X)` | `_setTotYP1(X)` |
| `_componentes/pago.php` — handler descuento | `$("#p1").val(totalConDescuento)` | `_setP1SiNoVinculada(totalConDescuento)` |
| `_componentes/pago.php` — servicio gratuito ON | `$("#total_descuento, #p1").val("0")` | `$("#total_descuento").val("0"); _setP1SiNoVinculada("0")` |
| `_componentes/pago.php` — servicio gratuito OFF | `$("#total_descuento, #p1").val(original)` | split similar |

**Nota:** `_setP1SiNoVinculada` solo protege slot 1. Los slots 2 y 3 no tienen este problema porque su valor inicial es vacío (no heredan el total automáticamente).

---

## Comportamiento al desvincular (`limpiarTxnPOS`) — estado completo

Al hacer × en el badge de transacción vinculada:

1. Limpia `#txn_pos_{slot}` (hidden field)
2. Limpia badge (`#txn_seleccionada_{slot}`) y resultados de búsqueda
3. Limpia input buscador (`#buscar_txn_pos_{slot}`)
4. Llama `_actualizarReferenciaAutorizacion(slot, null)` → elimina código del campo referencia
5. Resetea selects a `selectedIndex = 0` con `.trigger('change')` para que JQM refresque la UI:
   - `#t{slot}`, `#banco{slot}`, `#tipotarjeta{slot}`, `#m{slot}`
6. Vacía cuotas (`#numerocuotas{slot}`)
7. **NO** limpia monto (`#p{slot}`) — se mantiene por si el usuario quiere reutilizarlo
8. Muestra selector de terminal POS
9. Muestra sección buscar + botón Cobrar POS
10. Desbloquea todos los campos

---

## Backend: `codigo_autorizacion` en endpoint `/recientes` (2026-04-16)

El endpoint `GET /microservicesFinancialManagement/pinpad/transaccionesPagos/recientes` fue corregido para incluir `codigo_autorizacion` en la respuesta.

**Problema:** el service `obtenerRecientes()` hacía un `.map()` explícito a `TransaccionResumen` que no incluía el campo, descartándolo aunque estuviera en el modelo y en la BD.

**Fix en `TransaccionPago.sequelize.ts`:**
```typescript
attributes: [
  'id', 'monto', 'marca_tarjeta', 'banco_emisor', 'tipo_tarjeta',
  'numero_tarjeta_mask', 'numero_cuotas', 'fecha_transaccion', 'hora_transaccion',
  'codigo_autorizacion',  // ← agregado
  [literal(`...`), 'moneda'],
],
```

**Fix en `TransaccionPago.service.ts`:**
```typescript
export interface TransaccionResumen {
  // ...campos existentes...
  codigo_autorizacion?: string | null;  // ← agregado
}

// en obtenerRecientes().map():
codigo_autorizacion: transaccion.codigo_autorizacion || null,  // ← agregado
```

**Tabla:** `appinmater_financial_management.transacciones_pagos.codigo_autorizacion VARCHAR(20)`

---

## Relacionado

- Dominio POS/pagos: [[02-DOMINIOS/facturacion/gestor-pagos]]
- Traces frontend: [[05-TRACES/frontend/voucher-anulacion-gestor-pagos]]
- Trace vinculación POS UI: [[05-TRACES/frontend/pago-legacy-vinculacion-pos]]
- Trace vinculación backend: [[05-TRACES/backend/pos-recibo-vinculacion]]
- Endpoint de lectura: [[01-SERVICIOS/emr-financial-management/endpoints/GET-comprobante-pagos-pos]]
- Bridge PinPad: `/_Microservices/EMR/Services/CobroPOS.service.php`
