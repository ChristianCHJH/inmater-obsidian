# Índice: Traces (Investigaciones Documentadas)

**Última actualización:** 2026-04-13
**Total de traces:** 1
**Estado:** 🟢 Actualizado

## 📊 Traces por capa

### Frontend Traces
```
05-TRACES/frontend/
(vacío - próximas investigaciones)
```

### Backend Traces
```
05-TRACES/backend/
├── backend-traces_index.md - Índice de esta capa
└── [[transferido-status-determination.md]] ⭐ Qué define "Transferido"
```

### Database Traces
```
05-TRACES/database/
(vacío - próximas investigaciones)
```

---

## 📋 Listado completo

| Archivo | Capa | Dominio | Descripción | Estado |
|---------|------|---------|-------------|--------|
| [[05-TRACES/backend/transferido-status-determination.md]] | Backend | laboratorio | Lógica que determina "Transferido" en OUT | 🟢 Documentado |

---

## 🔗 Convención de Traces

Un **TRACE** documenta:
1. **Origen:** Dónde inicia (button, evento, decisión)
2. **Capas:** Frontend → Backend → Database
3. **Flujo completo:** Paso a paso qué ocurre
4. **Decisiones:** Qué determina cada resultado
5. **Impacto:** Qué cambia, qué se actualiza, qué se muestra

### Template: `06-GOVERNANCE/templates/trace-template.md`

---

## 📍 Próximas investigaciones

### Frontend
- [ ] Patient registration button flow
- [ ] Login/logout flow
- [ ] Appointment scheduling flow
- [ ] Laboratory evaluation form

### Backend
- [ ] Create patient endpoint
- [ ] JWT validation in Gateway
- [ ] Multi-sede authentication
- [ ] Appointment creation flow

### Database
- [ ] Patient schema relationships
- [ ] Audit log structure
- [ ] Legacy vs new schema mapping

---

## 🎯 Cómo crear una nueva trace

```
1. Identifica qué quieres investigar
2. Copia template: 06-GOVERNANCE/templates/trace-template.md
3. Guarda en: 05-TRACES/[layer]/[nombre].md
4. Documenta: origen → capas → decisiones
5. Linkea: servicios, dominios, tablas
6. Actualiza: este índice
```

---

**Última actualización:** 2026-04-13
**Mantenido por:** Obsidian Documenter Agent
