---
title: Cambiar la pareja en el informe de Laboratorio de Reproducción Asistida
projects: [[apps-web]]
domain: [[laboratorio]], [[pacientes]]
layer: backend
type: trace
status: documented
created: 2026-04-15
last-reviewed: 2026-04-15
tech-stack:
  - PHP
  - PostgreSQL
backlog: false
---

# Trace: ¿Cómo cambiar la pareja en el informe `info_r.php`?

**Última actualización:** 2026-04-15
**Estado:** 🟢 Documentado

## Origen

| Campo | Valor |
|-------|-------|
| Pantalla | LABORATORIO DE REPRODUCCIÓN ASISTIDA (informe PDF) |
| URL ejemplo | `app.inmater.pe/info_r.php?a=M358-26&b=47679551&c=41389131&sede=20` |
| Archivo | `apps/web/info_r.php` |
| Campo visual | Fila **"Pareja"** en sección DATOS DEL PROCEDIMIENTO |

## Pregunta clave

**Si la pareja mostrada en el informe está equivocada (o falta), ¿qué se debe actualizar?**

## Respuesta rápida

La pareja del informe se resuelve en dos pasos:

1. El ciclo reproductivo (`hc_reprod`) apunta a la pareja vía la columna **`hc_reprod.p_dni`** (y `p_dni_het` cuando aplica heterólogo).
2. Los datos mostrados (nombre, apellido, edad) se leen de **`hc_pareja`** filtrando por `p_dni`.

Por lo tanto:

- **Pareja equivocada / reasignar** → cambiar `hc_reprod.p_dni`.
- **Datos de la pareja mal escritos** → cambiar `hc_pareja` donde `p_dni = <dni_actual>`.

---

## Parámetros de la URL

[info_r.php:18-20](apps/web/info_r.php#L18-L20)

```php
$pro   = $_GET['a']; // protocolo (lab_aspira.pro)
$dni   = $_GET['b']; // DNI de la paciente (hc_paciente.dni)
$p_dni = $_GET['c']; // DNI de la pareja (hc_pareja.p_dni)
```

| Param | Ejemplo    | Tabla / columna resuelta |
| ----- | ---------- | ------------------------ |
| `a`   | `M358-26`  | `lab_aspira.pro` → `la.rep` = `hc_reprod.id` |
| `b`   | `47679551` | `hc_paciente.dni` |
| `c`   | `41389131` | `hc_pareja.p_dni` |

---

## Flujo de resolución

### 1. Obtiene paciente y datos de procedimiento

[info_r.php:21-43](apps/web/info_r.php#L21-L43)

```php
$rPaci = $db->prepare("SELECT nom, ape FROM hc_paciente WHERE dni=?");
$rPaci->execute(array($dni));

$stmt = $db->prepare("SELECT la.fec, COALESCE(sed_agenda.nombre, sed.nombre) as sede_nombre
    FROM hc_reprod hr
    INNER JOIN lab_aspira la ON la.rep = hr.id AND la.estado IS TRUE
    INNER JOIN sedes sed ON sed.id = hr.sedeid
    ...
    WHERE hr.estado = true AND la.pro = ?");
$stmt->execute([$pro]);
```

### 2. Resuelve la pareja (SOLTERA vs. con pareja)

[info_r.php:45-58](apps/web/info_r.php#L45-L58)

```php
if ($p_dni <> "" && $p_dni <> 1) {
    $rPare = $db->prepare("SELECT p_nom, p_ape, p_fnac FROM hc_pareja WHERE p_dni=?");
    $rPare->execute([$p_dni]);
    $pare = $rPare->fetch(PDO::FETCH_ASSOC);

    if ($pare['p_fnac'] == '1899-12-30') {
        $p_edad = ' (Edad: -)';
    } else {
        $p_edad = ' (Edad: '.date_diff(date_create($pare['p_fnac']),
                    date_create($data_procedimiento['fec']))->y.')';
    }
    $pareja = $pare['p_ape'].' '.$pare['p_nom'].$p_edad;
} else {
    $pareja = 'Soltera';
}
```

Reglas:

- `c` vacío o `c=1` → se imprime **"Soltera"**.
- `p_fnac = '1899-12-30'` → edad `-` (fecha centinela de "desconocida").
- La edad se calcula entre `p_fnac` y `lab_aspira.fec` (fecha del procedimiento), NO la fecha actual.

### 3. Render

[info_r.php:124](apps/web/info_r.php#L124)

```php
<th align="left">Pareja</th><td>'.$pareja.'</td>
```

---

## ¿Quién arma la URL con `c = p_dni`?

El parámetro `c` proviene de `hc_reprod.p_dni` (o del alias `$asp['p_dni']` / `$paci['p_dni']`) en todos los call sites. Ejemplos:

| Archivo | Línea | Origen |
|--------|-------|--------|
| [le_tanque.php:217](apps/web/le_tanque.php#L217) | `&c=' . $asp['p_dni']` | lista de aspiraciones |
| [le_aspi1.php:545](apps/web/le_aspi1.php#L545) | `&c=" . $paci['p_dni']` | aspiración día 1 |
| [le_aspi9.php:176](apps/web/le_aspi9.php#L176) | `&c=' . $asp['p_dni']` | aspiración día 9 |
| [labo-betas-lista.php:353](apps/web/labo-betas-lista.php#L353) | `&c=' . $paci['p_dni']` | lista de betas |
| [n_repro.php:592](apps/web/n_repro.php#L592) | `&c='.$repro['p_dni']` | pantalla de reprod. |
| [lista_pro_t.php:142](apps/web/lista_pro_t.php#L142) | `&c=" . $paci['p_dni']` | lista protocolos |
| [lista_pro_f.php:199](apps/web/lista_pro_f.php#L199) | `&c=" . $paci['p_dni']` | lista protocolos |

Conclusión: **cambiar `hc_reprod.p_dni` modifica el informe para TODAS las pantallas que generan este link**.

---

## Escenarios de cambio

### Escenario A — Reasignar el informe a OTRA pareja

Usar cuando el informe vincula a la pareja equivocada (ej. otra persona real, o se creó con DNI vacío/placeholder).

```sql
UPDATE appinmater_modulo.hc_reprod
   SET p_dni       = '<nuevo_dni_pareja>',
       iduserupdate = <login>,
       updatex      = NOW()
 WHERE id = <hc_reprod.id>;
```

Para obtener `hc_reprod.id` desde el protocolo mostrado en el informe:

```sql
SELECT hr.id, hr.p_dni, hr.p_dni_het
  FROM hc_reprod hr
  INNER JOIN lab_aspira la ON la.rep = hr.id AND la.estado IS TRUE
 WHERE la.pro = 'M358-26';   -- parámetro `a` de la URL
```

Precondición: el `<nuevo_dni_pareja>` debe existir en `hc_pareja`. Si no, insertarlo primero:

```sql
INSERT INTO appinmater_modulo.hc_pareja (p_dni, p_nom, p_ape, p_fnac, ...)
VALUES ('<dni>', '<NOMBRES>', '<APELLIDOS>', '<YYYY-MM-DD>', ...);
```

Si el ciclo es heterólogo (muestra de semen de tercero), considerar también `p_dni_het`:

[_database/db_tools.php:2331-2337](apps/web/_database/db_tools.php#L2331-L2337)

```php
function updateReproAndro($id, $p_dni_het) {
    $stmt = $db->prepare("UPDATE hc_reprod SET p_dni_het=?, updatex=? WHERE id=?;");
    ...
}
```

### Escenario B — Corregir nombre / apellido / fecha nacimiento de la pareja actual

Usar cuando el vínculo es correcto pero los datos de la persona están mal escritos.

```sql
UPDATE appinmater_modulo.hc_pareja
   SET p_nom  = '<NOMBRES>',
       p_ape  = '<APELLIDOS>',
       p_fnac = '<YYYY-MM-DD>'
 WHERE p_dni = '41389131';   -- DNI actual (parámetro `c`)
```

Efecto secundario: este UPDATE impacta **todos los informes** que referencien esa pareja. Usar con cuidado si el mismo `p_dni` está asociado a varias pacientes.

### Escenario C — Pareja aparece como "Soltera" pero debería tener pareja

Causa: `hc_reprod.p_dni` está NULL / vacío / `= 1`. Seguir Escenario A.

---

## Vía UI (recomendado sobre SQL directo)

El UPDATE completo de `hc_reprod` — incluyendo `p_dni` y `p_dni_het` — está encapsulado en [_database/db_tools.php:2288-2293](apps/web/_database/db_tools.php#L2288-L2293):

```php
UPDATE hc_reprod
   SET p_dni=?, t_mue=?, eda=?, p_dni_het=?, poseidon=?, ..., iduserupdate=?, updatex=?
 WHERE id=?;
```

Esta función también inserta un registro en `appinmater_log.hc_reprod` (acción `'U'`), dejando trazabilidad del cambio. El formulario que la invoca es la pantalla de edición del ciclo reproductivo ([n_repro.php](apps/web/n_repro.php)). **Preferir editar por UI para garantizar el log de auditoría.**

---

## Tablas involucradas

| Tabla | Columna | Rol |
|-------|---------|-----|
| `appinmater_modulo.hc_reprod` | `id` | PK del ciclo reproductivo |
| `appinmater_modulo.hc_reprod` | `p_dni` | FK lógica → `hc_pareja.p_dni` (pareja biológica) |
| `appinmater_modulo.hc_reprod` | `p_dni_het` | FK lógica → `hc_pareja.p_dni` (pareja heteróloga) |
| `appinmater_modulo.hc_reprod` | `sedeid` | Sede del ciclo |
| `appinmater_modulo.hc_reprod` | `dni` | DNI de la paciente |
| `appinmater_modulo.hc_pareja` | `p_dni` | PK lógica de la pareja |
| `appinmater_modulo.hc_pareja` | `p_nom`, `p_ape`, `p_fnac` | Datos mostrados en el informe |
| `appinmater_modulo.lab_aspira` | `pro` | Código protocolo (parámetro `a`) |
| `appinmater_modulo.lab_aspira` | `rep` | FK → `hc_reprod.id` |
| `appinmater_modulo.lab_aspira` | `fec` | Fecha usada para calcular edad de la pareja |
| `appinmater_log.hc_reprod` | — | Auditoría de cambios (action `'U'`) |

---

## Descubrimientos

- **No hay FK formal paciente↔pareja.** El vínculo se expresa por `hc_reprod.p_dni` a nivel de CICLO, no de paciente. Una paciente puede tener varios ciclos, cada uno con pareja distinta.
- **`c = 1`** es un centinela de "Soltera" (equivalente a vacío). No es un DNI real.
- **`p_fnac = '1899-12-30'`** es el centinela de fecha de nacimiento desconocida.
- **La edad mostrada es histórica**, no actual: usa `lab_aspira.fec` como referencia.

---

## Problemas potenciales

- [ ] Si se actualiza `hc_pareja` (Escenario B) y el mismo `p_dni` está asociado a otras pacientes, cambian todos los informes de esas pacientes.
- [ ] SQL directo NO deja log en `appinmater_log.hc_reprod`. Solo la función [updateRepro](apps/web/_database/db_tools.php#L2288) vía UI lo hace.
- [ ] No existe validación en UI de que `p_dni` exista previamente en `hc_pareja` — hay que verificarlo manualmente antes del UPDATE.

---

## Relaciones

| Aspecto | Relacionado | Notas |
|---------|------------|-------|
| Dominio | [[laboratorio]], [[pacientes]] | Ciclo reproductivo + datos personales |
| Servicio Backend | apps-web (PHP Legacy) | `info_r.php` + `_database/db_tools.php` |
| Tablas | `hc_reprod`, `hc_pareja`, `lab_aspira` | Ver [[04-REFERENCIAS/Tables-Catalog]] |

---

**Tags:** #layer/backend #domain/laboratorio #domain/pacientes #apps-web #php-legacy
