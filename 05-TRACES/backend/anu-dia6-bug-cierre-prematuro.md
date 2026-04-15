---
title: BUG — Cierre prematuro del ciclo en día 6 con óvulos observados
projects: [[apps-web]]
domain: [[laboratorio]]
layer: backend
type: trace
status: fix-applied
severity: high
created: 2026-04-15
last-reviewed: 2026-04-15
tech-stack:
  - PHP
  - PostgreSQL
source-file: apps/web/le_aspi6.php
source-lines: le_aspi6.php:271, le_aspi6.php:278-281, le_aspi6.php:298
complexity: low
branch: fix/aspi6-cierre-ciclo-con-observados
---

# BUG: Cierre prematuro del ciclo en día 6

**Última actualización:** 2026-04-15
**Estado:** 🟢 Fix aplicado — pendiente verificación en staging
**Reportado por:** Christian Jara (validación funcional)

## 📍 Origen

| Campo | Valor |
|-------|-------|
| Pantalla | Laboratorio — Evaluación día 6 |
| Archivo | [le_aspi6.php:298-300](apps/web/le_aspi6.php#L298-L300) |
| Evento | Submit del formulario "GUARDAR DATOS" |
| Función afectada | `lab_updateAspi_fin` |

## 🎯 Resumen del bug

El día 6 cierra el ciclo usando el criterio incorrecto: mira si hay **criopreservaciones** (`$crio_d7 == 0`) en lugar de mirar si hay **observados** (`f_cic = 'O'`). Esto rompe la regla invariante del sistema y puede dejar embriones vivos (`anu = 0`) dentro de un ciclo marcado como finalizado.

## 📏 Regla invariante del sistema

> **Un ciclo se cierra cuando ya ningún óvulo tiene que pasar al siguiente día.**

"Pasar al siguiente día" equivale a quedar en estado **Observado** (`f_cic = 'O'`, `anu = 0`). Los estados T/C/N son terminales para ese óvulo: quedan bloqueados con `anu = día_siguiente` y no avanzan.

**Regla de negocio confirmada (2026-04-15):**
- En día 6, un óvulo criopreservado **NO necesita pasar a día 7** (no requiere embrioscope ni biopsia como criterio de bloqueo del cierre).
- El ciclo **no puede cerrar con ningún embrión en `O`**.

## 🔬 Comparación con día 5 (correcto)

### Día 5 — [le_aspi5.php:308-315](apps/web/le_aspi5.php#L308-L315)
```php
if ($cancela == $c2) {
    // cancela = contador de T/C/N   (nadie quedó en O)
    // c2      = total procesados
    lab_updateAspi_fin($_POST['pro']);
    $fin = 1;
}
```
✅ **Correcto:** cierra cuando ningún óvulo quedó en `O`.

### Día 6 — [le_aspi6.php:298-300](apps/web/le_aspi6.php#L298-L300)
```php
if ($crio_d7 == 0) {
    // crio_d7 = contador de f_cic == 'C' solamente
    lab_updateAspi_fin($_POST['pro']);
}
```
❌ **Incorrecto:** cierra si no hay crio, sin mirar si hay observados.

## 🧪 Escenario reproducible

Día 6 con 3 óvulos activos (`anu = 0` al entrar al loop):

| Óvulo | `f_cic` | `anu` después | Debería pasar a día 7 |
|-------|---------|----------------|------------------------|
| 1 | `O` (Observado) | `0` | ✅ Sí |
| 2 | `T` (Transferido) | `7` | ❌ No |
| 3 | `N` (No viable) | `7` | ❌ No |

**Resultado con código actual:**
- `$crio_d7 = 0` (nadie marcó `C`)
- Se ejecuta `lab_updateAspi_fin` → ciclo cerrado con `f_fin = HOY`
- Óvulo 1 queda con `anu = 0` → estado **"vivo dentro de ciclo cerrado"**
- Inconsistencia funcional: el óvulo 1 nunca se procesa en día 7.

## ✅ Lógica correcta propuesta

Opción A — Seguir exactamente el patrón de día 5:
```php
// Durante el loop:
if ($_POST['f_cic' . $i] != "O") $cancela++;

// Al final:
if ($cancela == $c2) {
    lab_updateAspi_fin($_POST['pro']);
}
```

Opción B — Contador explícito de observados:
```php
// Durante el loop:
if ($_POST['f_cic' . $i] == "O") $obs_d7++;

// Al final:
if ($obs_d7 == 0) {
    lab_updateAspi_fin($_POST['pro']);
}
```

Ambas son equivalentes. La Opción A mantiene la simetría textual con día 5 (`$cancela`, `$c2`), lo que facilita el mantenimiento.

## 🔎 Análisis de causa raíz

La condición `$crio_d7 == 0` mezcla dos preocupaciones que deberían estar separadas:

1. **Bloqueo del óvulo:** decidido por `anu` (basado en si está en `O` o no).
2. **Procesamiento adicional:** embrioscope / biopsia / video, que en otro punto del código se ejecuta condicionalmente por archivos subidos.

El autor probablemente asumió que la presencia de crio implicaba "este ciclo va a día 7", pero:
- Un crio en día 6 puede cerrarse igual que un T o un N (no requiere día 7).
- Lo que va a día 7 son los **observados**, y esos no se están contando.

## ✅ Fix aplicado (2026-04-15)

**Rama:** `fix/aspi6-cierre-ciclo-con-observados`
**Commit:** `fix(laboratorio): cerrar ciclo en día 6 solo si no quedan embriones observados`

### Cambios en [le_aspi6.php](apps/web/le_aspi6.php)

| # | Línea | Antes | Después |
|---|-------|-------|---------|
| 1 | 271 | `$crio_d7 = 0;` | `$obs_d7 = 0;` |
| 2 | 278-281 | Contaba `$crio_d7++` cuando `f_cic == 'C'` | Cuenta `$obs_d7++` cuando `f_cic == 'O'`, dentro del mismo `if` del observado |
| 3 | 298 | `if ($crio_d7 == 0)` | `if ($obs_d7 == 0)` |

### Bloque nuevo

```php
for ($i = 1; $i <= $c; $i++) {
    if ($_POST['anu' . $i] == 0 or $_POST['anu' . $i] >= 7) {
        $c2++;
        if ($_POST['f_cic' . $i] == "O") {
            $anu = 0;
            $obs_d7++;
        } else {
            $anu = 7;
        }
        lab_updateAspi_d6(...);
    }
}
// ...
if ($obs_d7 == 0) {
    lab_updateAspi_fin($_POST['pro']);
}
```

### Concepto final

> **Un ciclo se cierra en día 6 si y sólo si ningún embrión quedó en `Observado`.**

- Simetría con día 5 (que usa `$cancela == $c2`, semánticamente equivalente).
- Eliminado el falso criterio "hay crio → no cerrar".
- Los embriones en `O` ahora siempre pasan a día 7; los demás (T/C/N) son terminales con `anu = 7`.

### Por qué se descartó el criterio `$crio_d7`

Confirmado con la regla de negocio: en día 6, un crio **no necesita pasar a día 7** (no hay procesamiento obligatorio de embrioscope/biopsia que obligue a abrir día 7). Lo único que obliga a día 7 son los óvulos en observación.

### Nombre de la variable: `$obs_d7`

Se eligió por:
- **Simetría con `$crio_d7`** (el nombre anterior): mismo patrón `{estado}_d{día_destino}`.
- **Expresa la intención:** "óvulos observados que pasarían al día 7".
- **Generalizable:** si el mismo fix se aplica a días 2-5, queda `$obs_d3`, `$obs_d4`, etc.

## 🗃️ Impacto

- **Datos:** Posible existencia de filas en `lab_aspira_dias` con `anu = 0` cuyo `lab_aspira.f_fin` no es NULL.
- **Clínico:** El embriólogo pierde la vista de día 7 para óvulos que debían seguir en cultivo.
- **Reportes:** Columna OUT muestra estado inconsistente (sigue en cultivo aparentemente, pero ciclo cerrado).

## 📋 Validaciones pendientes

- [ ] Probar en staging los 4 escenarios:
  1. Todos T/C/N (nadie en O) → debe cerrar.
  2. Al menos uno en O → NO debe cerrar.
  3. Mezcla con crio, ninguno en O → debe cerrar (antes quedaba abierto sin necesidad).
  4. Observado sin crio → NO debe cerrar (antes cerraba incorrectamente).
- [ ] Query de verificación de ciclos ya afectados en prod (conteo + detalle):
  ```sql
  SELECT COUNT(DISTINCT la.pro) AS ciclos_afectados,
         COUNT(lad.ovo)         AS ovulos_huerfanos
  FROM lab_aspira la
  JOIN lab_aspira_dias lad
    ON lad.pro = la.pro
   AND lad.idsede_lab = la.sede_id
  WHERE la.f_fin IS NOT NULL
    AND la.estado IS TRUE
    AND lad.estado IS TRUE
    AND lad.anu = 0;
  ```
- [ ] Revisar si existe el mismo patrón defectuoso en días 2, 3, 4 ([le_aspi2.php:237](apps/web/le_aspi2.php#L237), [le_aspi3.php:237](apps/web/le_aspi3.php#L237), [le_aspi4.php:250](apps/web/le_aspi4.php#L250)). *Nota: estos días usan `$cancela == $c2` (patrón correcto), probablemente no aplique.*
- [ ] Definir estrategia de remediación para ciclos ya cerrados con `anu = 0` (si la query confirma el caso).

## 🔗 Relacionado

- Dominio: [[02-DOMINIOS/laboratorio/Campo ANU - Bloqueo por Día del Óvulo]]
- Trace base: [[anu-lifecycle-dia5-dia6]]
- Columna OUT: [[02-DOMINIOS/laboratorio/Columna OUT - Estado del Embrión]]
- Servicio: PHP Legacy (`apps/web/`)

---

**Tags:** #bug #layer/backend #laboratorio #php-legacy #ciclo-embrion #anu #cierre-ciclo
**Investigación realizada:** 2026-04-15
**Fuente:** Análisis conjunto + validación funcional con Christian Jara
