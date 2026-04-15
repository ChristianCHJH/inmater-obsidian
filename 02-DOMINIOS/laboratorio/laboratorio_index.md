# Índice: Dominio Laboratorio

## 📋 Archivos en esta carpeta

| Archivo | Descripción | Relacionado |
|---------|-------------|-----------|
| [Laboratorio-Procesos.md](Laboratorio-Procesos.md) | Flujos generales y procesos del dominio | EMR.Patient.Service |
| [Columna OUT - Estado del Embrión.md](Columna OUT - Estado del Embrión.md) | TRACE: lógica de estado del embrión OUT | EMR.Patient.Service |
| [Campo ANU - Bloqueo por Día del Óvulo.md](Campo%20ANU%20-%20Bloqueo%20por%20Día%20del%20Óvulo.md) | Semántica del campo `anu` y cómo bloquea óvulos por día | apps-web (PHP Legacy) |

## 🎯 Contenido del dominio

Este dominio cubre toda la lógica relacionada con:
- **Estados de embrión:** OUT, ciclo, evaluación, pronóstico
- **Ciclos de laboratorio:** Phases, evaluaciones, pronósticos
- **Procesos:** Cómo fluyen los datos de embrión a través del sistema

## 🔗 Relaciones

- **Padre:** [[dominios_index.md]]
- **Servicio:** [[../../01-SERVICIOS/emr-patient-service]] (implementa esta lógica)
- **Patrones:** [[../../02-DOMINIOS/_shared-patterns/shared-patterns_index.md]] (patrones reutilizables)
- **Decisiones:** [[../../03-ARQUITECTURA/decisions/decisions_index.md]]
- **Traces:** [[../../05-TRACES/traces_index.md]]

---

**Tags:** #domain #laboratorio #healthcare
**Última actualización:** 2026-04-15
