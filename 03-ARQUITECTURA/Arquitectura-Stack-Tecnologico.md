# Arquitectura del Sistema

## 📐 Vista General

Sistema de microservicios con Clean Architecture en cada servicio.

```
inmater-core (orquestador)
├── services/ (backend microservicios)
│   ├── EMR.Api-Gateway.Service (Puerto 8080)
│   ├── EMR.Auth.Service (Puerto 3003)
│   ├── EMR.Calendar.Service (Puerto 3002)
│   └── EMR.Patient.Service (Puerto 3001)
├── apps/ (frontend)
│   ├── web/ (PHP legacy - app.inmater.pe)
│   └── frontend/ (Vue 3 - portal.inmater.pe)
└── docs/ (documentación)
```

## 🏗️ Estructura interna de cada servicio

```
src/
├── app.ts                    # Bootstrap
├── domain/
│   ├── interfaces/           # Contratos de negocio
│   └── RepositoriosModel/    # Abstracciones (repositorios)
├── infrastructure/
│   ├── config/               # Bases de datos, seguridad
│   ├── models/               # Sequelize ORM
│   └── UseRepository/        # Implementaciones
└── presentation/
    ├── controllers/          # Express handlers
    └── routes/               # Rutas
```

## 🔀 Patrón de Routing

El API Gateway enruta requests usando patrones wildcard:

- `/microservicesPacientes/*` → Patient Service
- `/microservicesCalendario/*` → Calendar Service
- `/microservicesFacturacion/*` → Facturacion Service

## 📊 Stack Tecnológico

| Capa | Tecnología |
|------|-----------|
| Backend | TypeScript, Node.js, Express |
| ORM | Sequelize |
| BD | PostgreSQL |
| Frontend | Vue 3, Vite, Pinia |
| UI | PrimeVue, TailwindCSS |
| Auth | JWT |
| Deployment | Google Cloud Run + Firebase Hosting |

## 🔐 Seguridad

- JWT tokens en API Gateway
- CORS configurado para ambos dominios
- Helmet para headers seguros
- Rate limiting por IP y path
- Validación en capas (gateway + servicios)

## 📍 Ver también

- [[Servicios/Servicios-Microservicios]]
- [[Base de Datos/Base de Datos-PostgreSQL]]
