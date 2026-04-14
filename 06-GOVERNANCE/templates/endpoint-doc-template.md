# Endpoint: [MÉTODO] [PATH]

**Servicio:** [[01-SERVICIOS/[service-name]]]
**Dominio:** [[02-DOMINIOS/[dominio]]]
**Última actualización:** 2026-04-13
**Status:** 🟢 Documentado

## 📌 Información general

| Propiedad | Valor |
|-----------|-------|
| **Método HTTP** | [GET/POST/PUT/DELETE/PATCH] |
| **Path completo** | `/microservices[Servicio]/[path]` |
| **Path interno** | `[path]` |
| **Autenticación** | JWT Bearer token (requerido/opcional) |
| **Rate limit** | [Si aplica] |

---

## 📥 Request

### Parámetros de ruta (path)

```
GET /microservicesPacientes/pacientes/:id
```

| Parámetro | Tipo | Requerido | Descripción |
|-----------|------|----------|-------------|
| `id` | UUID | ✅ Sí | ID del paciente |

### Query parameters

| Parámetro | Tipo | Requerido | Descripción | Ejemplo |
|-----------|------|----------|-------------|---------|
| `limit` | integer | ❌ No | Cantidad de registros | `10` |
| `offset` | integer | ❌ No | Desplazamiento | `0` |

### Body (si aplica)

```json
{
  "nombre": "Juan Pérez",
  "email": "juan@example.com",
  "dni": "12345678"
}
```

**Validaciones:**
- `nombre` (string) - Requerido, máx 255 caracteres
- `email` (string) - Requerido, formato válido
- `dni` (string) - Requerido, formato DNI válido

---

## 📤 Response

### Success (200 OK / 201 Created)

```json
{
  "success": true,
  "message": "Paciente creado exitosamente",
  "data": {
    "id": "uuid-1234-5678",
    "nombre": "Juan Pérez",
    "email": "juan@example.com",
    "dni": "12345678",
    "createdAt": "2026-04-13T10:30:00Z"
  }
}
```

### Error (4xx/5xx)

```json
{
  "success": false,
  "message": "Email ya existe en el sistema"
}
```

**Códigos de error comunes:**
- `400 Bad Request` - Validación fallida (campo requerido, formato inválido)
- `401 Unauthorized` - Token faltante o inválido
- `403 Forbidden` - Permiso insuficiente
- `404 Not Found` - Recurso no encontrado
- `409 Conflict` - Email/DNI duplicado
- `500 Internal Server Error` - Error del servidor

---

## 🔧 Implementación

### Controller

**Archivo:** `src/presentation/controllers/[Controller].ts`
**Método:** `[methodName]`

```typescript
// Describir qué hace el controller
// Qué validaciones
// Qué llama al servicio
```

### Service

**Archivo:** `src/application/services/[Service].ts`

```typescript
// Descripción de la lógica de negocio
```

### Repository

**Archivo:** `src/infrastructure/UseRepository/[Repository].ts`

```typescript
// Descripción de las queries a base de datos
```

---

## 💾 Tablas afectadas

| Tabla | Operación | Descripción |
|-------|-----------|-------------|
| `appinmater_modulo.hc_paciente` | SELECT/INSERT/UPDATE | Datos del paciente |
| `appinmater_modulo.hc_paciente_contact` | INSERT/UPDATE | Información de contacto |

---

## 🔍 Flujo completo

Ver trace completo: [[05-TRACES/backend/[flujo-nombre]]]

1. Usuario hace request al endpoint
2. Gateway valida JWT
3. Gateway rutea a [Servicio]
4. Controller recibe request
5. Service ejecuta lógica
6. Repository accede a BD
7. Response se envía al cliente

---

## 🔗 Relaciones

| Tipo | Relacionado | Notas |
|------|-------------|-------|
| **Dominio** | [[02-DOMINIOS/...]] | Lógica de negocio |
| **Patrón** | [[03-ARQUITECTURA/patterns/controller-service-repo]] | Estructura |
| **ADR** | [[03-ARQUITECTURA/decisions/...]] | Decisión |
| **Catálogo** | [[04-REFERENCIAS/Endpoints-Catalog#...]] | Listado global |

---

## 📋 Ejemplos de uso

### Curl

```bash
curl -X POST http://localhost:8080/microservicesPacientes/pacientes \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  -d '{
    "nombre": "Juan Pérez",
    "email": "juan@example.com",
    "dni": "12345678"
  }'
```

### JavaScript/Axios

```javascript
const response = await axios.post(
  '/microservicesPacientes/pacientes',
  {
    nombre: 'Juan Pérez',
    email: 'juan@example.com',
    dni: '12345678'
  },
  {
    headers: {
      'Authorization': `Bearer ${token}`
    }
  }
);
```

---

## ⚠️ Consideraciones

- **Autenticación:** Se requiere JWT válido
- **Rate limiting:** [X requests por minuto]
- **Caducidad de datos:** [Si aplica]
- **Auditoría:** Se registra en tabla de logs

---

**Tags:** #endpoint #[servicio] #[dominio]
**Última revisión:** 2026-04-13
