---
title: "PHP Legacy — Módulo de Emisión de Comprobantes (pago.php)"
projects: [[apps/web]]
domain: [[02-DOMINIOS/facturacion/gestor-pagos]]
layer: legacy-frontend
type: domain
status: documented
created: 2026-04-16
last-reviewed: 2026-04-16
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

## Relacionado

- Dominio POS/pagos: [[02-DOMINIOS/facturacion/gestor-pagos]]
- Traces frontend: [[05-TRACES/frontend/voucher-anulacion-gestor-pagos]]
- Servicio financiero: `EMR.Financial-Management.Service` (vinculación de transacciones POS)
- Bridge PinPad: `/_Microservices/EMR/Services/CobroPOS.service.php`
