# Domain Model: [Nombre del dominio]

**Última actualización:** 2026-04-13
**Dominio:** [[02-DOMINIOS/[dominio-nombre]]]

## 📝 Descripción

[Explicar qué es este dominio y qué conceptos contiene en el lenguaje del negocio - no técnico]

---

## 🎯 Conceptos clave (Ubiquitous Language)

| Término | Significado | Ejemplo |
|---------|-----------|---------|
| [Concepto 1] | [Definición en lenguaje de negocio] | [Ejemplo real] |
| [Concepto 2] | [Definición en lenguaje de negocio] | [Ejemplo real] |
| [Concepto 3] | [Definición en lenguaje de negocio] | [Ejemplo real] |

---

## 🏗️ Agregados (DDD)

### Agregado 1: [Nombre]

**Entidad raíz:** [Qué es el raíz]

**Entidades:**
- [Entidad 1] - [Descripción]
- [Entidad 2] - [Descripción]

**Value Objects:**
- [Value Object 1] - [Descripción]
- [Value Object 2] - [Descripción]

**Invariantes:**
- [Regla de negocio 1 que nunca se puede violar]
- [Regla de negocio 2 que nunca se puede violar]

**Métodos principales:**
- `crear()` - [Qué hace]
- `actualizar()` - [Qué hace]
- `cambiarEstado()` - [Qué hace]

---

## 📊 State Machine (si aplica)

```
[Estado A] --[evento]--> [Estado B]
   |                        |
   +----[evento]---> [Estado C]
```

**Estados:**
- **[Estado 1]:** [Descripción - cuándo ocurre]
- **[Estado 2]:** [Descripción - cuándo ocurre]
- **[Estado 3]:** [Descripción - cuándo ocurre]

**Transiciones permitidas:**
- [Estado A] → [Estado B] (condición: [X])
- [Estado B] → [Estado C] (condición: [Y])

---

## 🔄 Procesos de negocio

### Proceso 1: [Nombre]

**Flujo:**
1. [Paso 1 en lenguaje de negocio]
2. [Paso 2 en lenguaje de negocio]
3. [Paso 3 en lenguaje de negocio]

**Actores:**
- [Rol 1] - [Qué hace]
- [Rol 2] - [Qué hace]

**Reglas:**
- [Regla 1]
- [Regla 2]

---

## 💾 Persistencia

### Tablas principales

| Tabla | Agregado | Propósito |
|-------|----------|-----------|
| `[tabla_1]` | [Agregado] | [Qué persiste] |
| `[tabla_2]` | [Agregado] | [Qué persiste] |

### Relaciones

```sql
[tabla_1] --< [tabla_2]   -- [Descripción de relación]
```

---

## 🔗 Servicios relacionados

| Servicio | Rol | Implementación |
|----------|-----|-----------------|
| [[01-SERVICIOS/...]] | Consulta | [Qué implementa] |
| [[01-SERVICIOS/...]] | Escritura | [Qué implementa] |

---

## 📋 Reglas de negocio

- **Regla 1:** [Descripción clara y concisa]
  - Validación: [Cómo se valida]
  - Excepción: [Cuándo no aplica]

- **Regla 2:** [Descripción clara y concisa]
  - Validación: [Cómo se valida]
  - Excepción: [Cuándo no aplica]

---

## 🔐 Restricciones

- [Restricción 1]
- [Restricción 2]

---

## 📚 Documentación relacionada

- Flujo completo: [[05-TRACES/...]]
- Decisión arquitectónica: [[03-ARQUITECTURA/decisions/...]]
- Patrón usado: [[03-ARQUITECTURA/patterns/...]]

---

**Tags:** #domain #ddd #[dominio]
**Última revisión:** 2026-04-13
