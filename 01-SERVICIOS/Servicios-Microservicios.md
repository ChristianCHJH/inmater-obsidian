# Microservicios Backend

## 📡 Servicios disponibles

### 1. API Gateway (Puerto 8080)
- **Archivo:** `services/EMR.Api-Gateway.Service`
- **Responsabilidad:** Entrada única, enrutamiento, autenticación
- **Rutas:** `Router.ts` con patrones wildcard
- **Ver:** [[EMR.Api-Gateway]]

### 2. Auth Service (Puerto 3003)
- **Archivo:** `services/EMR.Auth.Service`
- **Responsabilidad:** JWT tokens, multi-sede login, sesiones
- **Ver:** [[EMR.Auth]]

### 3. Patient Service (Puerto 3001)
- **Archivo:** `services/EMR.Patient.Service`
- **Responsabilidad:** Gestión de pacientes (Mujer, Hombre, Pacientes endpoints)
- **Ver:** [[EMR.Patient]]

### 4. Calendar Service (Puerto 3002)
- **Archivo:** `services/EMR.Calendar.Service`
- **Responsabilidad:** Turnos, agendamientos, salas
- **Ver:** [[EMR.Calendar]]

## 🛠️ Comandos comunes

```bash
cd services/[NOMBRE_SERVICIO]

# Instalar dependencias
npm install

# Desarrollo con hot reload
npm run dev

# Build TypeScript
npm run build

# Producción
npm start

# Tests
npm test
npm run test:watch
npm run test:coverage
```

## 📋 Respuesta estándar API

Todos los endpoints retornan:

```json
{
  "success": true|false,
  "message": "Descripción",
  "data": { ... }  // opcional
}
```

## 🗄️ Tabla de rutas (API Gateway)

| Patrón | Destino |
|--------|---------|
| `/microservicesPacientes/*` | Patient Service |
| `/microservicesCalendario/*` | Calendar Service |
| `/microservicesFacturacion/*` | Facturacion Service |
| `/microservicesWitness/*` | Witness Service |

## ⚙️ Variables de entorno requeridas

```env
# Gateway
JWT_SECRET=xxx
PACIENTES_SERVICE_URL=http://localhost:3001
CALENDARIO_SERVICE_URL=http://localhost:3002
AUTH_SERVICE_URL=http://localhost:3003
RATE_LIMIT=100
REQUEST_TIMEOUT=30000

# Servicios
PORT=3001
DB_HOST=localhost
DB_PORT=5432
DB_USER=usuario
DB_PASSWORD=contraseña
DB_NAME=inmater_db
DB_SCHEMA_APP_MODULO=appinmater_modulo
URL_API_GATEWAY=http://localhost:8080
LIMIT_SIZE_PAYLOAD=10mb
```

## 📍 Ver también

- [[Arquitectura/Arquitectura-Stack-Tecnologico]]
- [[Base de Datos/Base de Datos-PostgreSQL]]
