---
title: Función lab_updateAspi_sta — propósito y condiciones de ejecución por día
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
source-file: apps/web/_database/db_tools.php, apps/web/le_aspi0-7.php
source-lines: db_tools.php:3189-3197
complexity: medium
---

# Función `lab_updateAspi_sta` — propósito y condiciones de ejecución

**Última actualización:** 2026-04-15
**Estado:** 🟢 Documentado

## 📍 Origen

| Campo | Valor |
|-------|-------|
| Función | `lab_updateAspi_sta` |
| Definición | [db_tools.php:3189](apps/web/_database/db_tools.php#L3189) |
| Tabla afectada | `lab_aspira` |
| Tipo de operación | `UPDATE` por `pro + sede_id` |

## 🎯 Qué hace

Estampa en la cabecera del ciclo (`lab_aspira`) el **avance al siguiente día** y guarda los datos de cierre del día actual. En la práctica marca: "este ciclo llegó hasta el día N".

### Definición

```php
function lab_updateAspi_sta($pro, $sta, $dias, $hra, $emb, $hra_c, $emb_c, $embtubing='')
{
    global $db;
    $emb_c     = ($emb_c !== '')     ? intval($emb_c)     : 0;
    $embtubing = ($embtubing !== '') ? intval($embtubing) : 0;

    $stmt = $db->prepare(
      "UPDATE lab_aspira
          SET sta = ?,
              dias = ?,
              hra"      . ($dias - 1) . " = ?,
              emb"      . ($dias - 1) . " = ?,
              hra"      . ($dias - 1) . "c = ?,
              emb"      . ($dias - 1) . "c = ?,
              emb"      . ($dias - 1) . "tubing = ?
        WHERE pro = ? AND estado IS TRUE AND sede_id = ?"
    );
    $stmt->execute([$sta, $dias, $hra, $emb, $hra_c, $emb_c, $embtubing, $pro, $_SESSION['sede']]);
}
```

### Parámetros

| Param | Descripción |
|-------|-------------|
| `$pro` | ID del ciclo (`lab_aspira.pro`) |
| `$sta` | Label del siguiente día (ej. `'Dia 6'`, `'Dia 7'`) |
| `$dias` | Número entero del siguiente día alcanzado |
| `$hra`, `$emb` | Hora y cantidad de embriones al cierre del día actual |
| `$hra_c`, `$emb_c` | Hora/cantidad para la parte "c" (criopreservación) del día actual |
| `$embtubing` | Embriones en tubing (sólo día 5, 6, 7; opcional) |

### Trampa clave: `dias - 1`

Las columnas donde escribe se construyen con `dias - 1`. Es decir, **los datos del día que se está cerrando se guardan en las columnas de ese día, no del siguiente.** El parámetro `$dias` indica el siguiente día al que avanza el ciclo.

**Ejemplo:** desde `le_aspi5.php` (día 5) se llama con `$dias = 6`. Internamente escribe en `hra5`, `emb5`, `hra5c`, `emb5c`, `emb5tubing` (el día que se cierra), y setea `sta = 'Dia 6'`, `dias = 6` (el siguiente).

---

## 📊 Llamadas en el sistema

| Archivo | Línea | `$sta` | `$dias` | Condición | Columnas escritas |
|---------|-------|--------|---------|-----------|---------------------|
| [le_aspi0.php:333](apps/web/le_aspi0.php#L333) | — | `'Dia 0'` | `1` | dentro de flujo de cierre | `hra0`, `emb0`, `hra0c`, `emb0c` |
| [le_aspi1.php:215](apps/web/le_aspi1.php#L215) | 1ª | `'Dia 2'` | `2` | tras loop de día 1 | `hra1`, `emb1`, `hra1c`, `emb1c` |
| [le_aspi1.php:219](apps/web/le_aspi1.php#L219) | 2ª | `'Dia 2'` | `2` | dentro de cierre de ciclo | idem |
| [le_aspi2.php:229](apps/web/le_aspi2.php#L229) | 1ª | `'Dia 3'` | `3` | `$_POST['dias'] <= 3` | `hra2`, `emb2`, `hra2c`, `emb2c` |
| [le_aspi2.php:235](apps/web/le_aspi2.php#L235) | 2ª | `'Dia 3'` | `3` | `cancela==c2` **y** `dias > 3` | idem |
| [le_aspi3.php:230](apps/web/le_aspi3.php#L230) | 1ª | `'Dia 4'` | `4` | `$_POST['dias'] <= 4` | `hra3`, `emb3`, `hra3c`, `emb3c` |
| [le_aspi3.php:236](apps/web/le_aspi3.php#L236) | 2ª | `'Dia 4'` | `4` | `cancela==c2` **y** `dias > 4` | idem |
| [le_aspi4.php:242](apps/web/le_aspi4.php#L242) | 1ª | `'Dia 5'` | `5` | `$_POST['dias'] <= 5` | `hra4`, `emb4`, `hra4c`, `emb4c` |
| [le_aspi4.php:249](apps/web/le_aspi4.php#L249) | 2ª | `'Dia 5'` | `5` | `cancela==c2` **y** `dias > 5` | idem |
| **[le_aspi5.php:301](apps/web/le_aspi5.php#L301)** | **1ª** | `'Dia 6'` | `6` | `$_POST['dias'] <= 7` | `hra5`, `emb5`, `hra5c`, `emb5c`, `emb5tubing` |
| **[le_aspi5.php:310](apps/web/le_aspi5.php#L310)** | **2ª** | `'Dia 6'` | `6` | `cancela==c2` **y** `dias > 6` | idem |
| **[le_aspi6.php:289](apps/web/le_aspi6.php#L289)** | única | `'Dia 7'` | `7` | **sin condición** (siempre se ejecuta en submit) | `hra6`, `emb6`, `hra6c`, `emb6c`, `emb6tubing` |
| [le_aspi7.php:275](apps/web/le_aspi7.php#L275) | única | `'Dia 7'` | `8` | **sin condición** | `hra7`, `emb7`, `hra7c`, `emb7c`, `emb7tubing` |

---

## 🔬 Patrón general (aspi2 – aspi5)

Cada día del 2 al 5 ejecuta `lab_updateAspi_sta` bajo **dos posibles disparadores**:

### Disparador 1 — Avance normal
```php
if ($_POST['dias'] <= N+1) {
    lab_updateAspi_sta($_POST['pro'], 'Dia N+1', N+1, ...);
}
```
> **"Si el ciclo estaba planeado para llegar al menos hasta día N+1, avanza el estado."**

Lógica: no se sobrescribe el estado si el ciclo ya estaba planeado para terminar en un día anterior (ej. un ciclo corto de 3 días no debería pasar por estado "Dia 4").

### Disparador 2 — Avance forzado por cierre anticipado
```php
if ($cancela == $c2) {
    if ($_POST['dias'] > N+1) {
        lab_updateAspi_sta($_POST['pro'], 'Dia N+1', N+1, ...);
    }
    lab_updateAspi_fin($_POST['pro']);
}
```
> **"Si todos los óvulos cerraron en este día pero el ciclo estaba planeado para más días, fuerza el avance al día N+1 antes de cerrar."**

Esto mantiene consistencia en el `sta`/`dias` de `lab_aspira` incluso cuando el ciclo termina antes de lo previsto (ej. todos T/C/N en día 3 aunque el ciclo estaba planeado hasta día 5).

### Cobertura combinada (semántica unificada)

Para cualquier valor de `$_POST['dias']`, al menos uno de los dos disparadores se ejecutará si corresponde:

| `$_POST['dias']` | Disparador 1 (`<= N+1`) | Disparador 2 (`> N+1` + todos cerraron) |
|-------------------|--------------------------|------------------------------------------|
| `< N+1` | ✅ | ❌ |
| `= N+1` | ✅ | ❌ |
| `> N+1` | ❌ | ✅ (sólo si `cancela == c2`) |

Resultado: el estado siempre avanza a `'Dia N+1'` **salvo** que `dias > N+1` y no todos hayan cerrado (el ciclo continúa normalmente al siguiente día, el avance se hará en el submit de ese día).

---

## 🔬 Día 5 (caso especial)

```php
// Línea 300
if ($_POST['dias'] <= 7) {
    lab_updateAspi_sta($_POST['pro'], 'Dia 6', 6, ...);
}
// ...
if ($cancela == $c2) {
    // Línea 309
    if ($_POST['dias'] > 6) {
        lab_updateAspi_sta($_POST['pro'], 'Dia 6', 6, ...);
    }
    lab_updateAspi_fin($_POST['pro']);
}
```

**Asimetría respecto al patrón general:** el disparador 1 usa `<= 7` (no `<= 6`). Esto introduce un solapamiento intencional con el disparador 2 cuando `dias == 7`:
- Si `dias == 7` y no todos cerraron → sólo disparador 1 (avance normal a día 6).
- Si `dias == 7` y todos cerraron → **ambos se ejecutan** (doble llamada, efecto idéntico — segunda es redundante pero inocua).
- Si `dias > 7` y todos cerraron → sólo disparador 2.

Probablemente heredado de un refactor incompleto. No causa bug (el UPDATE es idempotente con los mismos valores), pero sí trabajo redundante contra BD.

---

## 🔬 Día 6 (único, incondicional)

```php
// Línea 289
lab_updateAspi_sta($_POST['pro'], 'Dia 7', 7, ...);
```

### Condiciones (las del bloque padre)

La llamada sólo requiere que:
1. `isset($_POST['n_ovo'])` — el formulario del día 6 fue enviado con lista de óvulos.
2. `$_POST['guardar'] == "GUARDAR DATOS"` — el botón correcto.

Una vez dentro del bloque, **siempre se ejecuta**. No hay condicional `dias <= X` ni fallback por `cancela == c2`.

### Por qué no tiene condicional

Día 6 es el último día con cabecera propia (día 7 comparte label `'Dia 7'` con día 6 pero con `dias = 8`). Cualquier ciclo que pase por día 6 necesariamente alcanzó ese estado, independientemente del plan original: ya no hay escenario donde "no avanzar al día 7" tenga sentido.

---

## 🧭 Resumen visual

```
┌──────────────────┐
│  submit día N    │
└────────┬─────────┘
         │
  ┌──────┴──────┐
  │ Loop óvulos │  ← anu + f_cic por cada uno
  └──────┬──────┘
         │
         ├─── Disparador 1: avance normal (dias <= N+1)
         │     └─ lab_updateAspi_sta('Dia N+1', N+1, ...)
         │
         ├─── (si Tra == 1) lab_updateAspi_sta_T(...)
         │
         └─── Disparador 2: si cancela == c2
               ├─ si dias > N+1 → lab_updateAspi_sta('Dia N+1', N+1, ...)
               └─ lab_updateAspi_fin($pro)
```

---

## 🔎 Descubrimientos

- **Función única para todos los días:** la misma `lab_updateAspi_sta` sirve para cabecera de día 0 al 7; cambian parámetros.
- **Columnas dinámicas:** usa concatenación de strings para construir el nombre de columna (`hra5`, `emb5`, etc.). Sensible a typos, pero funcional.
- **Día 5 rompe el patrón** con la condición `<= 7` en lugar de `<= 6`. Inocuo pero inconsistente.
- **Día 6 es incondicional** — consistente con el hecho de que es el último día con semántica propia.
- **`$_POST['dias']` representa el plan original del ciclo**, no el día actual. Es clave entender esto para leer las condiciones.
- El UPDATE depende de `$_SESSION['sede']`; si la sesión pierde el contexto de sede, el update no impacta ninguna fila.

---

## 🧩 Efecto observado: día 7 visible en el sidebar aunque el ciclo cerró en día 6

### Síntoma reportado

Al cerrar un ciclo en día 6 (todos los embriones en T/C/N, fix nuevo `$obs_d7 == 0` → `lab_updateAspi_fin`), el menú lateral "Lista de Procedimientos" sigue mostrando el enlace a **Día 7**.

### Causa raíz

Dos piezas concurrentes:

1. **En día 6 esta función se llama SIN condición:**
   ```php
   // le_aspi6.php:289
   lab_updateAspi_sta($_POST['pro'], 'Dia 7', 7, ...);
   ```
   → `lab_aspira.dias = 7` siempre que haya submit de día 6, exista o no cierre posterior.

2. **El menú renderiza día 7 con un único criterio:**
   ```php
   // _includes/menu_laboratorio.php:22-24
   <?php if (isset($paci['dias']) and $paci['dias'] >= 7) { ?>
       <li><a href="le_aspi7.php?id=<?php echo $paci['pro']; ?>">Dia 7</a>
   ```
   → Muestra día 7 si `dias >= 7`, sin consultar `f_fin`.

### Resultado

| Estado del ciclo | `dias` en BD | `f_fin` en BD | Sidebar muestra día 7 |
|-------------------|---------------|----------------|-------------------------|
| Activo, pasó día 6 con observados | 7 | NULL | ✅ Correcto |
| Cerrado en día 6 (todos T/C/N) | 7 | HOY | ⚠️ Aparece aunque ya no aplica |
| Cerrado en día 5 | 6 | HOY | ✅ No aparece |

### Efecto colateral potencial

Si el usuario entra a día 7 de un ciclo cerrado y da submit, [le_aspi7.php:295](apps/web/le_aspi7.php#L295) vuelve a ejecutar `lab_updateAspi_fin`, **reescribiendo `f_fin` con la fecha actual**. No es destructivo pero:
- Pierde la fecha real de cierre original.
- Puede ejecutar `lab_insertEmbry` / `embryoscope_video` sobre un ciclo ya finalizado.

### Fix sugerido (pendiente de acordar)

Opción A — **menú filtra por cierre** (menos riesgo):
```php
<?php if (isset($paci['dias']) and $paci['dias'] >= 7 and empty($paci['f_fin'])) { ?>
```
El enlace desaparece cuando el ciclo cierra. Consultas históricas se hacen por otra vía.

Opción B — **día 6 condiciona el avance** (más simétrico con día 5):
```php
// le_aspi6.php - antes de lab_updateAspi_sta
if ($obs_d7 > 0 || tiene_crio_activo(...)) {
    lab_updateAspi_sta(..., 'Dia 7', 7, ...);
}
```
Evita setear `dias = 7` si el ciclo va a cerrarse en día 6.

**Recomendación:** Opción A. Es menos invasiva y no rompe la lógica interna del flujo de día 6.

---

## ⚠️ Casos edge a validar

- [ ] ¿Qué pasa si `$_POST['dias']` llega con un valor inesperado (ej. string, null, 10)? El comparador numérico podría fallar silenciosamente.
- [ ] Revisar si la primera llamada en día 5 (línea 301) con `<= 7` y no `<= 6` es un bug histórico o intencional.
- [ ] Doble ejecución en día 5 cuando `dias == 7` y `cancela == c2`: confirmar que no genera dobles entradas en auditoría (si la hay).
- [ ] En días 0 y 1 la función se llama también — revisar semántica completa (no analizada en este trace).

---

## 📖 Glosario de variables usadas en los flujos

| Variable | Dónde aparece | Qué cuenta / significa |
|----------|----------------|--------------------------|
| `$c` | Todos los días (`$_POST['c']`) | Cantidad total de óvulos enviados por el formulario |
| `$c2` | aspi2–aspi6 | Contador de óvulos **procesados** (pasaron el filtro `anu == 0 OR anu >= siguiente_dia`). Su valor depende de cuántos óvulos activos haya. No es fijo. |
| `$cancela` | aspi2–aspi5 | Contador de óvulos que **cerraron** en este submit (`f_cic` = T, C o N). |
| `$obs_d7` | aspi6 (post-fix) | Contador de óvulos que **siguen en observación** al final del día 6. Reemplazó a `$crio_d7`. |
| `$_POST['dias']` | todos | **Plan original** del ciclo: cuántos días estaba proyectado durar. No es el día actual. |
| `$fin` | aspi3–aspi5 | Bandera local que indica que el ciclo se cerró en este submit. Se usa para lógica condicional posterior (embrioscope, etc.). |
| `$anu` | todos | Valor a guardar en `lab_aspira_dias.anu` para el óvulo actual del loop. |
| `$don` | todos | 1 si el paciente es donante (`$_POST['don'] == 'D'`), 0 en caso contrario. |

**Invariante:** `$c2 = $cancela + $observados` (si no hay otros estados). En día 5 se cierra cuando `$cancela == $c2` (→ observados = 0); en día 6 cuando `$obs_d7 == 0` (equivalente).

## 🔗 Relacionado

- Dominio: [[02-DOMINIOS/laboratorio/Campo ANU - Bloqueo por Día del Óvulo]]
- Función relacionada: `lab_updateAspi_fin` (cierre de ciclo) — ver [[anu-dia6-bug-cierre-prematuro]]
- Función hermana: `lab_updateAspi_sta_T` (actualiza `lab_aspira_t` en transferencia fresca) — [db_tools.php:3199](apps/web/_database/db_tools.php#L3199)
- Sidebar del laboratorio: [_includes/menu_laboratorio.php](apps/web/_includes/menu_laboratorio.php)
- Trace relacionado: [[anu-lifecycle-dia5-dia6]]
- Servicio: PHP Legacy (`apps/web/`)

---

**Tags:** #layer/backend #laboratorio #php-legacy #cabecera-ciclo #lab_aspira
**Investigación realizada:** 2026-04-15
**Fuente:** Análisis de `le_aspi0.php` – `le_aspi7.php` + `_database/db_tools.php`
