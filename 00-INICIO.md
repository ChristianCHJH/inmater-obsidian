# EMR - Sistema de Historias Clínicas

**Fecha actualización:** 2026-04-16

## 📋 Descripción General

EMR (Electronic Medical Record) es un sistema de historias clínicas para una clínica de medicina reproductiva. Full-stack monorepo con microservicios backend (TypeScript/Node.js) y frontend Vue 3, gestionado mediante git submodules.

## 🗺️ ¿Por dónde empezar?

👉 Lee primero: [[06-GOVERNANCE/Estructura-Vault-Inmater]] para entender cómo está organizado este vault

## 🏢 Repositorios principales

- **inmater-core**: Repositorio orquestador con submodules
- **EMR.Api-Gateway.Service**: Microservicio puerta de entrada
- **EMR.Auth.Service**: Autenticación y sesiones
- **EMR.Patient.Service**: Gestión de pacientes
- **EMR.Calendar.Service**: Agenda y turnos
- **EMR.Financial-Management.Service**: POS Niubiz, transacciones, comisiones → [[01-SERVICIOS/emr-financial-management/_index]]
- **apps/frontend**: Vue 3 SPA
- **apps/web**: PHP legacy

## 📁 Navegación por niveles

### Nivel 1️⃣: SERVICIOS (por proyecto/repo)

- [[01-SERVICIOS/Servicios-Microservicios]] - Catálogo de servicios
- API Gateway, Auth Service, Patient Service, Calendar Service
- Vue3 frontend, PHP legacy → [[01-SERVICIOS/php-legacy/pago-emision-comprobantes]]

### Nivel 2️⃣: DOMINIOS (lógica compartida - EJE CENTRAL)

- [[02-DOMINIOS/Procesos-Negocio]] - Flujos de negocio generales
- [[02-DOMINIOS/laboratorio/Laboratorio-Procesos]] - Laboratorio
- [[02-DOMINIOS/facturacion/gestor-pagos]] - Pagos POS, cobros, comprobantes
- [[02-DOMINIOS/_shared-patterns/]] - Patrones reutilizables

### Nivel 3️⃣: ARQUITECTURA (decisiones y patrones)

- [[03-ARQUITECTURA/Arquitectura-Stack-Tecnologico]] - Stack general
- [[03-ARQUITECTURA/decisions/]] - ADRs (Architecture Decision Records)
- [[03-ARQUITECTURA/patterns/]] - Patrones técnicos
- [[03-ARQUITECTURA/infra/]] - DevOps, Docker, GCP, PostgreSQL

### Nivel 4️⃣: REFERENCIAS (catálogos globales)

- [[04-REFERENCIAS/]] - Endpoints, tables, servicios, tech stack

### Complementarios

- [[05-TRACES/]] - Investigaciones documentadas
- [[06-GOVERNANCE/]] - Cómo mantener el vault
  - [[06-GOVERNANCE/Estructura-Vault-Inmater]] ← Lee esto primero
  - [[06-GOVERNANCE/Git-Convenciones]]

## 🚀 Quick Start

### Backend

```bash
cd services/EMR.Api-Gateway.Service
npm install
npm run dev
```

### Frontend

```bash
cd apps/frontend
npm install
npm run dev # http://localhost:5173
```

## 🔗 Enlaces útiles

- Documentación en `/docs`
- CLAUDE.md - Instrucciones para Claude Code
- Governance - Estructura de governance en `docs/governance/`
