# Columna OUT - Estado final del Embrión

**Última actualización:** 2026-04-13

## ¿Qué es OUT?

La columna **OUT** en la tabla "EVALUACIÓN DEL DESARROLLO" muestra el **estado final del embrión** después de ser evaluado durante los días del ciclo.

```
┌─────────────────────────────────────────────┐
│ ID Embrión │ DIA 2 │ DIA 3 │ DIA 5 │ OUT   │
├─────────────────────────────────────────────┤
│ 4          │ ...   │ ...   │ ...   │ Trans │  ← Estado final
│ 11         │ ...   │ ...   │ ...   │       │  ← Sin determinar
└─────────────────────────────────────────────┘
```

## Posibles valores

| Valor | Significado | Código BD | Línea código |
|-------|-------------|-----------|--------------|
| **Transferido** | Seleccionado para transferencia fresca | `d*f_cic = 'T'` | e_repro_info.php:426,442,459,474,496,517 |
| **CRIO** | Criopreservado (congelado) | `d*f_cic = 'C'` | e_repro_info.php:422,438,455,470,492,513 |
| **NV** | No viable | `d*f_cic = 'N'` | e_repro_info.php:428,444,461,476,498,519 |
| **(vacío)** | Sin evaluar / Sin determinar | `d*f_cic` vacío | e_repro_info.php:409 |

## Cómo se asigna

### Lógica de código

```php
// Inicialización (línea 409)
$fin = '';  // comienza vacío

// Se evalúa por cada día según el ciclo:
if ($asp['d2f_cic'] == 'T') {
    $fin = 'Transferido';  // Línea 426
}
if ($asp['d2f_cic'] == 'C') {
    $fin = 'CRIO';         // Línea 422
}
if ($asp['d2f_cic'] == 'N') {
    $fin = 'NV';           // Línea 428
}

// Lo mismo para d3, d4, d5, d6, d7...
```

### Campos evaluados por día

- **Day 2:** `d2f_cic` (evaluado si dias >= 3)
- **Day 3:** `d3f_cic` (evaluado si dias >= 4)
- **Day 4:** `d4f_cic` (evaluado si dias >= 5)
- **Day 5:** `d5f_cic` (evaluado si dias >= 6)
- **Day 6:** `d6f_cic` (evaluado si dias >= 7)
- **Day 7:** `d7f_cic` (evaluado si dias >= 8)

### Importante: Último valor gana

El valor mostrado en OUT es el **último asignado** durante la evaluación:

**Ejemplo:**
- Si `d2f_cic = 'T'` y `d3f_cic = 'C'` → Muestra **CRIO** (no Transferido)
- Si `d2f_cic = 'T'` y `d3f_cic = ''` → Muestra **Transferido**

## Base de datos

### Tabla: `hc_aspirado`

Almacena datos de evaluación embrionaria:

```sql
SELECT d2f_cic, d3f_cic, d4f_cic, d5f_cic, d6f_cic, d7f_cic
FROM hc_aspirado
WHERE pro = ? AND estado = true
```

### Archivo: `e_repro_info.php`

- Línea 111: Query a `lab_aspira` (obtiene días del ciclo)
- Línea 380: Query a `hc_aspirado` (obtiene evaluaciones)
- Línea 409: Inicializa `$fin = ''`
- Línea 420-519: Evalúa y asigna valores según `d*f_cic`
- Línea 538: Renderiza en HTML

## Contadores asociados

La misma lógica se usa para contar:

- **Total Transferidos:** `$c_T++` (cuando `d*f_cic = 'T'`)
- **Total Criopreservados:** `$c_C++` (cuando `d*f_cic = 'C'`)

## Casos de uso

### Embrión evaluado para transferencia
```
d2f_cic = 'T' → OUT = "Transferido"
                Incrementa c_T
```

### Embrión congelado
```
d3f_cic = 'C' → OUT = "CRIO"
                Incrementa c_C
```

### Embrión descartado
```
d2f_cic = 'N' → OUT = "NV" (No Viable)
```

### Embrión sin evaluar completamente
```
d2f_cic = '' y d3f_cic = '' → OUT = "" (vacío)
```

## Flujo de datos

```
Laboratorio evalúa embrión
        ↓
Registra estado en d*f_cic
        ↓
e_repro_info.php lee hc_aspirado
        ↓
Asigna $fin según valores
        ↓
Renderiza en columna OUT
        ↓
Cuenta transferidos/criopreservados
```

## 📍 Ver también

- [[02-DOMINIOS/laboratorio/Laboratorio-Procesos]]

## 🔍 Investigación detallada

Para entender exactamente qué define que aparezca "Transferido":
- 👉 [[05-TRACES/backend/transferido-status-determination.md]] - TRACE completo con análisis línea por línea
