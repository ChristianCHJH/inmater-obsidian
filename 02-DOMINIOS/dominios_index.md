# Índice: Dominios (Lógica de negocio compartida)

## 📋 Descripción

Los dominios contienen la lógica de negocio compartida que afecta múltiples servicios. Este es el **eje central** del vault.

## 📂 Dominios existentes

| Dominio | Carpeta | Índice | Descripción |
|---------|---------|--------|-------------|
| **Autenticación** | `autenticacion/` | (pendiente crear) | Login multi-sede, JWT, sesiones |
| **Pacientes** | `pacientes/` | (pendiente crear) | Datos de pacientes, DNI, validaciones |
| **Calendario** | `calendario/` | (pendiente crear) | Turnos, agendamientos, salas |
| **Facturación** | `facturacion/` | (pendiente crear) | Invoices, transacciones |
| **Laboratorio** | `laboratorio/` | [laboratorio_index.md](laboratorio/laboratorio_index.md) | Estados de embrión, ciclos, evaluaciones |

## 📂 Patrones compartidos

| Carpeta | Contenido | Índice |
|---------|----------|--------|
| **_shared-patterns/** | Patrones reutilizables en TODOS | [shared-patterns_index.md](_shared-patterns/shared-patterns_index.md) |

## 🎯 Estructura de un dominio

Cada dominio contiene:
- `flow.md` - Cómo funciona el flujo general
- `logic.md` - Reglas de negocio específicas
- `validations.md` - Validaciones reutilizables
- Relaciones a servicios que lo implementan

## 🔗 Relaciones

- **Nivel 1 (Servicios):** [[01-SERVICIOS/servicios_index.md]] (implementan estos dominios)
- **Nivel 3 (Arquitectura):** [[03-ARQUITECTURA/arquitectura_index.md]] (patrones técnicos)
- **Nivel 4 (Referencias):** [[04-REFERENCIAS/referencias_index.md]] (catálogos globales)
- **Nivel 5 (Traces):** [[05-TRACES/traces_index.md]] (investigaciones)

---

**Tags:** #domain #business-logic
**Última actualización:** 2026-04-13
