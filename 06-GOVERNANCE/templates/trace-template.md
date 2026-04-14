# Trace: [Nombre de la investigación]

**Última actualización:** 2026-04-13
**Estado:** 🟢 Documentado

## 📍 Origen (dónde comienza)

- **Componente:** [Nombre del componente/botón/screen]
- **Archivo:** [ruta del archivo]
- **Ubicación:** [L42](enlace relativo)
- **Evento:** [Qué dispara esto - click, submit, etc.]

## 🎯 Objetivo

Documentar el flujo completo de: **[descripción breve del flujo]**

---

## 📊 Flujo por capas

### Capa Frontend (UI)

**Archivo:** `apps/frontend/src/...`

```typescript
// Describir el evento que dispara el flujo
// Qué composable se invoca
// Qué estado se actualiza
```

**Pasos:**
1. Usuario hace [acción]
2. Se invoca [función/método]
3. Estado en Pinia cambia a [estado]
4. Se realiza llamada API a [endpoint]

**Request:**
```json
{
  "campo": "valor"
}
```

### Capa Backend (API)

**Servicio:** [EMR.Patient.Service | EMR.Auth.Service | etc]
**Archivo:** `src/presentation/controllers/[Controller].ts`
**Método:** [método HTTP y path]

```typescript
// Describir el controller que recibe la request
// Qué validaciones hace
// Qué llamadas a servicios/repositorios
```

**Pasos:**
1. Controller valida [qué valida]
2. Servicio [QUÉ HACE]
3. Repository hace query a tabla [tabla]
4. Response enviada: [estructura]

**Response:**
```json
{
  "success": true,
  "message": "Descripción",
  "data": { }
}
```

### Capa Database (BD)

**Base de datos:** PostgreSQL
**Tabla(s) afectada(s):**
- `[schema].[tabla_1]` - [qué se modifica]
- `[schema].[tabla_2]` - [qué se modifica]

**Query:**
```sql
-- Describir la query
SELECT * FROM [tabla] WHERE [condición];
```

---

## 🔗 Relaciones

| Aspecto | Relacionado | Notas |
|---------|-----------|-------|
| **Dominio** | [[02-DOMINIOS/...]] | Lógica compartida |
| **Servicio Frontend** | `apps/frontend/` | Componente que inicia |
| **Servicio Backend** | [[01-SERVICIOS/...]] | Servicio que procesa |
| **Patrón usado** | [[03-ARQUITECTURA/patterns/...]] | Patrón arquitectónico |
| **Tabla(s)** | [[04-REFERENCIAS/Tables-Catalog#...]] | Datos afectados |

---

## 🔍 Descubrimientos

Documentar qué aprendiste al investigar este flujo:

- **Comportamiento esperado:** [X pasa cuando Y]
- **Comportamiento inesperado:** [X no pasa, causó Z]
- **Validaciones importantes:** [Qué valida, por qué]
- **Casos edge:** [Qué pasa en caso especial]

---

## ⚠️ Problemas encontrados

- [ ] Issue 1: [Descripción]
- [ ] Issue 2: [Descripción]

---

## 📝 Notas

[Notas adicionales sobre el flujo, decisiones, alternativas consideradas, etc.]

---

**Tags:** #layer/frontend #layer/backend #layer/database
**Última revisión:** 2026-04-13
