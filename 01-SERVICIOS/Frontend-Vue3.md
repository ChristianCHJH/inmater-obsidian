# Frontend - Aplicación Vue 3

## 📱 Aplicación principal

**Ubicación:** `apps/frontend`
**URL de desarrollo:** http://localhost:5173
**Framework:** Vue 3 + TypeScript
**Build tool:** Vite 7.2

## 🏗️ Estructura de carpetas

```
apps/frontend/src/
├── app/
│   ├── config/              # Configuración (API, rutas)
│   ├── plugins/             # PrimeVue, i18n, etc.
│   └── router/              # Vue Router setup
├── modules/                 # Feature modules
│   ├── andrology/
│   ├── auth/
│   ├── billing/
│   ├── dashboard/
│   ├── laboratory/
│   ├── reports/
│   └── settings/
└── shared/
    ├── components/          # Componentes reutilizables
    ├── composables/         # Vue composables
    ├── services/            # Servicios API
    ├── styles/              # Estilos globales
    ├── types/               # Interfaces TypeScript
    └── utils/               # Funciones auxiliares
```

## 🎨 Stack UI/UX

| Librería | Propósito |
|----------|-----------|
| **PrimeVue 4** | Componentes UI |
| **TailwindCSS 4** | Framework CSS |
| **Vue Router 4** | Enrutamiento (modo hash) |
| **Pinia 3** | State management |
| **Axios** | HTTP client con interceptores JWT |

## 🔌 Servicios API

Los servicios en `shared/services/` manejan comunicación con el API Gateway:

```typescript
// shared/services/[modulo].api.ts
const response = await api.get('/microservices[Modulo]/*');
```

## 🔐 Autenticación

- JWT token almacenado en Pinia/localStorage
- Interceptores de Axios añaden `Authorization: Bearer <token>`
- API Gateway valida JWT en cada request

## 🌐 Modos de enrutamiento

- **Hash mode:** Compatible con PHP legacy (#/dashboard)
- **Router 4:** Vue Router 4 para navegación SPA

## 📋 Módulos principales

### Laboratory
Gestión de evaluación de embriones y laboratorio

### Billing
Facturación y transacciones

### Dashboard
Panel de control principal

### Settings
Configuración de usuario y sistema

## 🛠️ Comandos

```bash
cd apps/frontend

npm install              # Instalar dependencias
npm run dev              # Servidor de desarrollo
npm run build            # Build para producción
npm run preview          # Vista previa de producción
```

## 📍 Ver también

- [[Arquitectura/Arquitectura-Stack-Tecnologico]]
- [[Frontend/Proyecto]]
