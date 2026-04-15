---
title: Ciclo de vida del campo ANU en día 5 y día 6
projects: [[apps-web]]
domain: [[laboratorio]]
layer: backend
type: trace
status: documented
created: 2026-04-15
last-reviewed: 2026-04-15
tech-stack:
  - PHP
  - PostgreSQL
source-file: apps/web/le_aspi5.php, apps/web/le_aspi6.php
source-lines: le_aspi5.php:275-320, le_aspi6.php:268-310
complexity: medium
---

# Trace: Ciclo de vida del campo `anu` en día 5 y día 6

**Última actualización:** 2026-04-15
**Estado:** 🟢 Documentado

## 📍 Origen

| Campo | Valor |
|-------|-------|
| Pantalla | Laboratorio — Evaluación día 5 / día 6 |
| Archivos | [le_aspi5.php](apps/web/le_aspi5.php), [le_aspi6.php](apps/web/le_aspi6.php) |
| Evento | Submit del formulario "GUARDAR DATOS" |
| Campo analizado | `anu` (tabla `lab_aspi`) |

## 🎯 Pregunta de investigación

**¿Qué condiciones diferencian el procesamiento de óvulos entre día 5 y día 6, y qué determina el valor final de `$anu` en cada caso?**

## Respuesta rápida

- En **día 5**, sólo se procesan óvulos con `anu = 0` o `anu >= 6`, y al cerrarse quedan con `anu = 6`.
- En **día 6**, sólo se procesan óvulos con `anu = 0` o `anu >= 7`, y al cerrarse quedan con `anu = 7`.
- El cierre se dispara cuando `f_cic != 'O'` (es decir, T, C o N).
- Día 5 permite *reactivar* (bajar `anu` de 6 a 0); día 6 no tiene esa lógica.

Ver [[02-DOMINIOS/laboratorio/Campo ANU - Bloqueo por Día del Óvulo]] para la semántica completa.

---

## 📊 Flujo lado a lado

### Día 5 — [le_aspi5.php:275-320](apps/web/le_aspi5.php#L275-L320)

```php
if (isset($_POST['n_ovo']) and isset($_POST['guardar']) and $_POST['guardar'] == "GUARDAR DATOS") {
    $cancela = 0;
    $c = $_POST['c'];
    $c2 = 0;
    $fin = 0;

    if ($c > 0) {
        if ($_POST['don'] == 'D') {$don = 1;} else {$don = 0;}

        for ($i = 1; $i <= $c; $i++) {
            if ($_POST['anu' . $i] == 0 or $_POST['anu' . $i] >= 6) {
                $c2++;
                if ($_POST['f_cic' . $i] == "O") {
                    if ($_POST['anu' . $i] == 6) {$anu = 0;}
                    else {$anu = $_POST['anu' . $i];}
                } else {
                    $anu = 6;
                    $cancela++;
                }
                lab_updateAspi_d5($_POST['pro'], $i, $anu, ...);
            }
        }
    }

    if ($_POST['dias'] <= 7) {
        lab_updateAspi_sta($_POST['pro'], 'Dia 6', 6, ...);
    }
    if ($_POST['Tra'] == 1) {
        lab_updateAspi_sta_T($_POST['Tra_id'], $_POST['pro'], 5, ...);
    }
    if ($cancela == $c2) {
        if ($_POST['dias'] > 6) {
            lab_updateAspi_sta($_POST['pro'], 'Dia 6', 6, ...);
        }
        lab_updateAspi_fin($_POST['pro']);
        $fin = 1;
    }
}
```

### Día 6 — [le_aspi6.php:268-310](apps/web/le_aspi6.php#L268-L310)

```php
if (isset($_POST['n_ovo']) and $_POST['guardar'] == "GUARDAR DATOS") {
    $c = $_POST['c'];
    $c2 = 0;
    $crio_d7 = 0;

    if ($c > 0) {
        if ($_POST['don'] == 'D') {$don = 1;} else {$don = 0;}
        for ($i = 1; $i <= $c; $i++) {
            if ($_POST['anu' . $i] == 0 or $_POST['anu' . $i] >= 7) {
                $c2++;
                if ($_POST['f_cic' . $i] == "O") {$anu = 0;}
                else {$anu = 7;}
                if ($_POST['f_cic' . $i] == 'C') $crio_d7++;
                lab_updateAspi_d6($_POST['pro'], $i, $anu, ...);
            }
        }
    }

    lab_updateAspi_sta($_POST['pro'], 'Dia 7', 7, ...);
    if ($_POST['Tra'] == 1) {
        lab_updateAspi_sta_T($_POST['Tra_id'], $_POST['pro'], 6, ...);
    }
    if ($_POST['pro'] and isset($_FILES) and (...)) {
        lab_insertEmbry(...);
        embryoscope_video(...);
    }
    if ($crio_d7 == 0) {
        lab_updateAspi_fin($_POST['pro']);
    }
    lab_updateRepro2([...]);
}
```

---

## 🔬 Comparación de condiciones

| Aspecto | Día 5 | Día 6 |
|---------|-------|-------|
| **Filtro de entrada** | `anu == 0 OR anu >= 6` | `anu == 0 OR anu >= 7` |
| **Valor si `f_cic = 'O'`** | `$anu = 0` (si venía en 6) o mantiene | `$anu = 0` (siempre) |
| **Valor si `f_cic != 'O'`** | `$anu = 6` | `$anu = 7` |
| **Lógica de reactivación** | ✅ Sí (de 6 → 0) | ❌ No |
| **Contador de cierres** | `$cancela` (todos T/C/N) | `$crio_d7` (sólo C) |
| **Función de persistencia** | `lab_updateAspi_d5` | `lab_updateAspi_d6` |
| **Cabecero siguiente día** | Condicional (`dias <= 7`) crea Día 6 | Incondicional crea Día 7 |
| **Transferencia** | `lab_updateAspi_sta_T(..., 5, ...)` | `lab_updateAspi_sta_T(..., 6, ...)` |
| **Cierre del ciclo** | Si `cancela == c2` → `lab_updateAspi_fin` + `$fin=1` | Si `crio_d7 == 0` → `lab_updateAspi_fin` |
| **Embryoscope (video/PDF)** | Sí, mismo patrón | Sí, con validaciones extra (`isset`, `!empty`) |
| **`lab_updateRepro2`** | No se llama en este bloque | ✅ Siempre al final |

---

## 🔍 Valores de `f_cic` (fin de ciclo)

Select definido en [le_aspi5.php:871-882](apps/web/le_aspi5.php#L871-L882):

| Valor | Label UI | Significado clínico | Efecto sobre `$anu` |
|-------|----------|--------------------|--------------------|
| `O` | Obs | Observación — sigue en cultivo | No cierra: `$anu = 0` |
| `T` | T | Transferencia fresca al útero | Cierra: `$anu = día+1` |
| `C` | Crio | Criopreservación (congelado) | Cierra: `$anu = día+1` |
| `N` | NV | No Viable (descartado) | Cierra: `$anu = día+1` |

En [le_aspi5.php:405-406](apps/web/le_aspi5.php#L405-L406) se remueven dinámicamente las opciones `T` y `O` bajo ciertas condiciones de UI.

---

## 🧭 Diagrama de transición por óvulo

```
                  ┌──────────────┐
                  │ anu = 0      │  ← activo, en cultivo
                  │ (cualquier   │
                  │  día previo) │
                  └──────┬───────┘
                         │
                  ┌──────┴───────┐
                  │  DÍA 5       │
                  │  formulario  │
                  └──────┬───────┘
                         │
           ┌─────────────┼─────────────┐
           │             │             │
      f_cic = O     f_cic = T/C/N
           │             │
           ▼             ▼
      anu = 0        anu = 6   ← cerrado en día 5
           │         (bloqueado desde día 6)
           │             │
           ▼             │ (ya no entra al loop de día 6)
      ┌──────────┐       │
      │  DÍA 6   │       │
      │formulario│       │
      └────┬─────┘       │
           │             │
   ┌───────┼───────┐     │
   │               │     │
 f_cic = O   f_cic = T/C/N
   │               │     │
   ▼               ▼     ▼
 anu = 0       anu = 7   (registro inmutable)
   │        cerrado en día 6
   │        (bloqueado desde día 7)
   ▼
 DÍA 7...
```

---

## 🔎 Descubrimientos

- **`anu` NO indica qué pasó, sino cuándo terminó.** El "qué" se guarda en `d*f_cic`.
- **`anu = día_actual + 1`** es el invariante al cerrar un óvulo.
- **Sólo día 5 permite "deshacer"** el cierre (pasar de 6 a 0). Los demás días asumen que una vez marcado T/C/N, la decisión es inmutable en esa sesión.
- **El criterio de "ciclo terminado" cambia entre días:**
  - Día 5: todos los óvulos procesados cerraron (`cancela == c2`)
  - Día 6: ninguno quedó en criopreservación pendiente para día 7 (`crio_d7 == 0`)
- **Día 6 siempre crea el cabecero del día 7** (`lab_updateAspi_sta('Dia 7', 7, ...)`), incluso si no hay óvulos activos. Día 5 sólo crea el del día 6 bajo la condición `dias <= 7`.
- **`lab_updateRepro2`** se llama únicamente en día 6 dentro del bloque analizado (sincroniza datos reproductivos al cerrar el flujo de día 6).

---

## ⚠️ Casos edge a validar

- [ ] ¿Qué pasa si el embriólogo marca `O` en día 5 pero no envía nada del día 6? El `anu` queda en 0 pero no avanza en evaluación.
- [ ] ¿Puede un óvulo con `anu = 7` reabrirse en día 7? (Requiere revisar `le_aspi7.php` si existe.)
- [ ] Comportamiento cuando `$_POST['dias']` viene con un valor raro (ej. 10) frente a las condiciones `<= 7` y `> 6`.

---

## 🔗 Relacionado

- Dominio: [[02-DOMINIOS/laboratorio/Campo ANU - Bloqueo por Día del Óvulo]]
- Columna OUT: [[02-DOMINIOS/laboratorio/Columna OUT - Estado del Embrión]]
- Trace hermano: [[transferido-status-determination]]
- Procesos: [[02-DOMINIOS/laboratorio/Laboratorio-Procesos]]
- Servicio: PHP Legacy (`apps/web/`)
- Tabla: `lab_aspi` (pendiente en [[04-REFERENCIAS/Tables-Catalog]])

---

**Tags:** #layer/backend #laboratorio #php-legacy #ciclo-embrion #anu
**Investigación realizada:** 2026-04-15
**Por:** Obsidian Documenter Agent
**Fuente:** Análisis de `le_aspi5.php` y `le_aspi6.php`
