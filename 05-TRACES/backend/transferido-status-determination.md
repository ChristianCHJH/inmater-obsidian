# Trace: ¿Qué define que aparezca "Transferido"?

**Última actualización:** 2026-04-13

## Origen

| Campo | Valor |
|-------|-------|
| Pantalla | Laboratorio de Reproducción Asistida - Evaluación del Desarrollo |
| Componente | `apps/web/e_repro_info.php` |
| Sección | Tabla HTML de "EVALUACIÓN DEL DESARROLLO" (línea 598-613) |
| Columna visual | OUT (salida/destino final del embrión) |

## Pregunta clave

**¿Qué determina que un embrión muestre "Transferido" en la columna OUT del reporte?**

## Respuesta rápida

Un embrión muestra **"Transferido"** en la columna OUT cuando **al menos uno de sus campos `d*f_cic` (Day X Final Ciclo) en la tabla `hc_aspirado` tiene el valor `'T'`**.

---

## Lógica de asignación

### Inicialización (Línea 409)
```php
$fin = '';  // Comienza vacío, sin valor asignado
```

### Evaluación por días (Líneas 420-517)

Para **cada día del ciclo**, se verifica el campo correspondiente:

```php
// Día 2 (si dias >= 3)
if ($asp['d2f_cic'] == 'T') {
    $c_T++;                    // Incrementa contador de transferidos
    $fin = 'Transferido';      // Asigna valor
}

// Día 3 (si dias >= 4)
if ($asp['d3f_cic'] == 'T') {
    $c_T++;
    $fin = 'Transferido';
}

// Día 4, 5, 6, 7... (mismo patrón)
if ($asp['d4f_cic'] == 'T') { ... }
if ($asp['d5f_cic'] == 'T') { ... }
if ($asp['d6f_cic'] == 'T') { ... }
if ($asp['d7f_cic'] == 'T') { ... }
```

### El valor final que se muestra

El valor mostrado en OUT es el **último asignado** durante la evaluación completa.

---

## Tabla: Valores posibles de `d*f_cic`

| Valor | Significado | Qué aparece en OUT | Línea código | Acción |
|-------|-------------|-------------------|--------------|--------|
| **'T'** | Transferencia fresca | "Transferido" | 426, 442, 459, 474, 496, 517 | `$c_T++` |
| **'C'** | Criopreservación | "CRIO" | 422, 438, 455, 470, 492, 513 | `$c_C++` |
| **'N'** | No viable | "NV" | 428, 444, 461, 476, 498, 519 | (sin contador) |
| **(vacío)** | Sin evaluar | (vacío) | 409 | (sin acción) |

---

## Campos evaluados

### Por día de ciclo

| Día | Campo | Línea | Condición |
|-----|-------|-------|-----------|
| 2 | `$asp['d2f_cic']` | 420 | `if ($pop['dias'] >= 3)` |
| 3 | `$asp['d3f_cic']` | 436 | `if ($pop['dias'] >= 4)` |
| 4 | `$asp['d4f_cic']` | 453 | `if ($pop['dias'] >= 5)` |
| 5 | `$asp['d5f_cic']` | 468 | `if ($pop['dias'] >= 6)` |
| 6 | `$asp['d6f_cic']` | 490 | `if ($pop['dias'] >= 7)` |
| 7 | `$asp['d7f_cic']` | 511 | `if ($pop['dias'] >= 8)` |

---

## Base de datos

### Tabla: `hc_aspirado`

```sql
SELECT 
    d2f_cic,   -- Day 2 Final Ciclo
    d3f_cic,   -- Day 3 Final Ciclo
    d4f_cic,   -- Day 4 Final Ciclo
    d5f_cic,   -- Day 5 Final Ciclo
    d6f_cic,   -- Day 6 Final Ciclo
    d7f_cic    -- Day 7 Final Ciclo
FROM hc_aspirado
WHERE pro = ? AND estado = true
```

### Schema

- **Schema:** `appinmater_modulo`
- **Tabla:** `hc_aspirado`
- **Campos:** Contienen códigos: 'T', 'C', 'N', o vacío

---

## Flujo de datos (Backend)

```
1. USER ACTION
   └─ Laboratorio evalúa embrión y registra en BD

2. PHP LEE DATOS (línea 111-117)
   └─ SELECT * FROM hc_aspirado WHERE pro = ?

3. LOOP SOBRE ASPIRADOS (línea 392)
   └─ for each $asp in aspirados:

4. INICIALIZA (línea 409)
   └─ $fin = '' (comienza vacío)

5. EVALÚA DÍAS (líneas 420-517)
   ├─ IF d2f_cic == 'T' → $fin = 'Transferido'
   ├─ IF d3f_cic == 'T' → $fin = 'Transferido' (overwrite)
   ├─ IF d4f_cic == 'T' → $fin = 'Transferido' (overwrite)
   ├─ IF d5f_cic == 'T' → $fin = 'Transferido' (overwrite)
   ├─ IF d6f_cic == 'T' → $fin = 'Transferido' (overwrite)
   └─ IF d7f_cic == 'T' → $fin = 'Transferido' (overwrite)

6. RENDERIZA EN HTML (línea 538)
   └─ <td>{{ $fin }}</td>  → Muestra en columna OUT

7. ACTUALIZA CONTADORES (línea 426, 442, etc)
   └─ $c_T++ (cuando $fin = 'Transferido')

8. PÁGINA SE CARGA
   └─ Usuario ve reporte con OUT = "Transferido"
```

---

## Observación CRÍTICA: Último valor gana

**Regla:** El valor mostrado es el **último asignado**.

### Ejemplo 1: Transferencia en múltiples días
```
d2f_cic = 'T' → $fin = 'Transferido'
d3f_cic = 'T' → $fin = 'Transferido' (sobreescribe, pero mismo valor)
d4f_cic = ''  → $fin mantiene 'Transferido'

RESULTADO: OUT = "Transferido" ✓
```

### Ejemplo 2: Cambio de decisión
```
d2f_cic = 'T' → $fin = 'Transferido'
d3f_cic = 'C' → $fin = 'CRIO' (sobreescribe)
d4f_cic = ''  → $fin mantiene 'CRIO'

RESULTADO: OUT = "CRIO" (no Transferido) ✗
```

### Ejemplo 3: Sin evaluación
```
d2f_cic = ''
d3f_cic = ''
d4f_cic = ''
... (todos vacíos)

RESULTADO: OUT = "" (vacío) ✗
```

---

## Casos prácticos del reporte

### Caso A: Embrión ID 4 (29-11-2025) → OUT = "Transferido"

```
Laboratorio decidió:
- d2f_cic = 'T' → Se asigna $fin = 'Transferido'
- d3f_cic = '' o no se evalúa
- Resultado: $fin mantiene 'Transferido'
- OUT muestra: "Transferido" ✓
- $c_T incrementa: 1
```

### Caso B: Embrión ID 11 (29-11-2025) → OUT = (vacío)

```
Laboratorio decidió:
- d2f_cic = '' (no tiene 'T')
- d3f_cic = '' o 'C' o 'N' (pero no 'T')
- Resultado: $fin nunca se asigna a 'Transferido'
- OUT muestra: (vacío) o "CRIO" o "NV"
- $c_T NO incrementa
```

---

## Contadores asociados

Cuando `d*f_cic == 'T'`:
- ✅ `$c_T++` → "Total Transferidos" incrementa
- ✅ `$fin = 'Transferido'` → Se muestra en OUT

```php
// Contadores globales
if ($asp['d2f_cic'] == 'T') {
    $c_T++;              // LINE 425
    $fin = 'Transferido'; // LINE 426
}
```

Al final del reporte:
```html
Total Transferidos: {{ $c_T }}
```

---

## Dependencias de negocio

| Elemento | Depende de | Por qué |
|----------|-----------|--------|
| OUT = "Transferido" | `d*f_cic = 'T'` en BD | Decisión del laboratorio |
| `$c_T` (contador) | OUT = "Transferido" | Cuenta embriones para transferencia |
| Total Criopreservados | `d*f_cic = 'C'` en BD | Embriones congelados |
| Visibilidad fila | d1f_cic == 'O' | Solo embriones viables pasan a evaluación |

---

## Contexto clínico

- **Transferido (T):** Embrión seleccionado para transferencia fresca al útero
- **CRIO (C):** Embrión congelado para uso futuro
- **NV (N):** Embrión no viable, descartado
- **(vacío):** Embrión sin evaluación completa

---

## Linaje de decisiones

1. **Laboratorio evalúa** (día 2, 3, 4, 5, 6, 7)
   ↓
2. **Registra en `hc_aspirado.d*f_cic`** (T, C, N, o vacío)
   ↓
3. **PHP lee y asigna `$fin`** (línea 420-517)
   ↓
4. **Renderiza en HTML columna OUT** (línea 538)
   ↓
5. **Usuario ve "Transferido"** en reporte

---

## 🔗 Relacionado

- **Dominio:** [[02-DOMINIOS/laboratorio/Laboratorio-Procesos]]
- **Documentación completa:** [[02-DOMINIOS/laboratorio/Columna OUT - Estado del Embrión]]
- **Servicio:** [[01-SERVICIOS/apps-web]] (PHP Legacy)
- **Tabla:** [[04-REFERENCIAS/Tables-Catalog#hc_aspirado]]
- **Archivo código:** `apps/web/e_repro_info.php` (línea 409-538)

---

## 📊 Metadatos

```yaml
---
title: ¿Qué define que aparezca "Transferido" en columna OUT?
projects: [[apps-web]]
domain: [[laboratorio]]
layer: backend
type: trace
status: documented
created: 2026-04-13
last-reviewed: 2026-04-13
tech-stack:
  - PHP
  - PostgreSQL
  - jQuery
source-file: apps/web/e_repro_info.php
source-lines: 409-538, 598-613
complexity: low
---
```

---

**Investigación realizada:** 2026-04-13
**Por:** Obsidian Documenter Agent
**Fuente:** Análisis de código PHP legacy + reporte de laboratorio
