---
title: Campo ANU - Día de bloqueo del óvulo en cultivo
projects: [[apps-web]]
domain: [[laboratorio]]
layer: backend
type: domain
status: documented
created: 2026-04-15
last-reviewed: 2026-04-15
tech-stack:
  - PHP
  - PostgreSQL
source-file: apps/web/le_aspi3.php, le_aspi4.php, le_aspi5.php, le_aspi6.php
---

# Campo ANU — Día de bloqueo del óvulo

**Última actualización:** 2026-04-15

## ¿Qué es `anu`?

`anu` es una columna de la tabla `lab_aspi` (óvulos/ovocitos de un ciclo de aspiración) que representa el **primer día a partir del cual ese óvulo YA NO está activo en cultivo**.

Equivalencias válidas para entenderlo:

- "Día en que el óvulo dejó de estar en cultivo"
- "Próximo día bloqueado para este óvulo"
- "Desde qué día este óvulo ya no aparece en las pantallas de evaluación"

> ⚠️ **No confundir con `d*f_cic`** (d5f_cic, d6f_cic, etc.), que almacena *cómo* terminó el ciclo (T/C/N/O). `anu` almacena *cuándo* se cerró.

---

## Posibles valores

| Valor | Significado | Archivo que lo asigna |
|-------|-------------|----------------------|
| `0` | Sigue activo, continúa en cultivo | Cualquier `le_aspi*.php` cuando `f_cic = 'O'` |
| `4` | Cerrado en día 3, bloqueado desde día 4 | [le_aspi3.php:220](apps/web/le_aspi3.php#L220) |
| `5` | Cerrado en día 4, bloqueado desde día 5 | [le_aspi4.php:232](apps/web/le_aspi4.php#L232) |
| `6` | Cerrado en día 5, bloqueado desde día 6 | [le_aspi5.php:291](apps/web/le_aspi5.php#L291) |
| `7` | Cerrado en día 6, bloqueado desde día 7 | [le_aspi6.php:279](apps/web/le_aspi6.php#L279) |

**Regla mnemotécnica:** `anu = día_actual + 1` cuando el óvulo se cierra.

---

## ¿Qué determina el valor?

El campo `f_cic` (Fin de Ciclo) del formulario de ese día:

| `f_cic` | Label UI | Acción sobre `anu` |
|---------|----------|---------------------|
| `O` | Obs (Observación) | **No bloquea:** `anu = 0` (o mantiene el valor posteado) |
| `T` | T (Transferencia fresca) | **Bloquea:** `anu = día_siguiente` |
| `C` | Crio (Criopreservación) | **Bloquea:** `anu = día_siguiente` |
| `N` | NV (No Viable) | **Bloquea:** `anu = día_siguiente` |

**Traducción clínica:** mientras el embriólogo marque `Obs`, el óvulo sigue evolucionando. En el momento que se transfiere, congela o descarta, `anu` queda grabado con el día siguiente al del formulario, evitando que vuelva a aparecer en pantallas posteriores.

---

## Uso del campo en las vistas

En cada pantalla de día `N`, se usa para dos cosas:

### 1. Filtrar qué filas entran al loop de guardado

```php
// le_aspi5.php:286
if ($_POST['anu' . $i] == 0 or $_POST['anu' . $i] >= 6) { ... }

// le_aspi6.php:276
if ($_POST['anu' . $i] == 0 or $_POST['anu' . $i] >= 7) { ... }
```

Sólo se procesan los óvulos activos (`anu=0`) o los que aún pertenecen a ese día.

### 2. Ocultar filas ya cerradas en la UI

```php
// le_aspi5.php:721
if ($aspi['anu'] <> 0 && $aspi['anu'] < 6) {
    // Oculta el <tr> con jQuery: $("#filaN").hide();
}
```

Un óvulo con `anu = 4` (cerrado en día 3) queda oculto en las pantallas de día 4, 5, 6 y 7.

---

## Reactivación (caso especial día 5)

El día 5 tiene una lógica de reactivación que los otros días NO tienen:

```php
// le_aspi5.php:288-289
if ($_POST['f_cic' . $i] == "O") {
    if ($_POST['anu' . $i] == 6) {$anu = 0;}
    else {$anu = $_POST['anu' . $i];}
}
```

**Traducción:** si un óvulo venía con `anu = 6` (cerrado en día 5 en una sesión previa) y el embriólogo vuelve a marcarlo como `Obs`, se devuelve a `anu = 0` para que continúe a día 6.

Este "undo" no existe en día 6: si `f_cic = 'O'` siempre se setea `anu = 0` directamente sin condicional.

---

## Relación con `f_cic` en la salida final (columna OUT)

`anu` determina **cuándo se cerró** el óvulo, pero lo que se muestra en la columna OUT del reporte depende de `d*f_cic`:

- `anu = 6` + `d5f_cic = 'T'` → OUT muestra "Transferido"
- `anu = 6` + `d5f_cic = 'C'` → OUT muestra "CRIO"
- `anu = 6` + `d5f_cic = 'N'` → OUT muestra "NV"
- `anu = 0` → OUT aún vacío (sigue en evolución)

Ver [[Columna OUT - Estado del Embrión]] para la lógica completa de OUT.

---

## ⚠️ Bug conocido (día 6)

El cierre del ciclo en día 6 mira `$crio_d7 == 0` en vez de "no hay observados". Puede cerrar el ciclo dejando óvulos con `anu = 0`. Ver [[anu-dia6-bug-cierre-prematuro]].

## 🔗 Relacionado

- Trace detallado: [[05-TRACES/backend/anu-lifecycle-dia5-dia6]]
- Bug día 6: [[05-TRACES/backend/anu-dia6-bug-cierre-prematuro]]
- Columna OUT: [[Columna OUT - Estado del Embrión]]
- Procesos generales: [[Laboratorio-Procesos]]
- Dominio padre: [[laboratorio_index]]
- Servicio: PHP Legacy (`apps/web/`)

---

**Tags:** #domain #laboratorio #campo-bd #anu #ciclo-embrion
