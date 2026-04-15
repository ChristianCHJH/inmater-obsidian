# Índice: Backend Traces

**Última actualización:** 2026-04-15
**Total de traces:** 5
**Estado:** 🟢 Actualizado

## Traces en esta carpeta

| Archivo                                         | Descripción                                                 | Domain          | Servicio              | Complejidad |
| ----------------------------------------------- | ----------------------------------------------------------- | --------------- | --------------------- | ----------- |
| [[transferido-status-determination.md]]         | Qué define que aparezca "Transferido" en columna OUT        | [[laboratorio]] | apps-web (PHP Legacy) | Baja        |
| [[anu-lifecycle-dia5-dia6.md]]                  | Ciclo de vida del campo `anu` entre día 5 y día 6           | [[laboratorio]] | apps-web (PHP Legacy) | Media       |
| [[anu-dia6-bug-cierre-prematuro.md]]            | 🟢 Fix aplicado — Día 6 cierra ciclo con observados         | [[laboratorio]] | apps-web (PHP Legacy) | Baja        |
| [[lab-updateaspi-sta-call-sites.md]]            | Función `lab_updateAspi_sta` — propósito y condiciones      | [[laboratorio]] | apps-web (PHP Legacy) | Media       |
| [[info-r-cambiar-pareja.md]]                    | Cómo cambiar la pareja en el informe `info_r.php`           | [[laboratorio]] | apps-web (PHP Legacy) | Baja        |

---

## Relaciones con otras carpetas

### Depende de:
- [[02-DOMINIOS/laboratorio/]] - Lógica de negocio de laboratorio
- [[04-REFERENCIAS/Tables-Catalog#hc_aspirado]] - Tabla de datos

### Es referenciado por:
- [[02-DOMINIOS/laboratorio/Columna OUT - Estado del Embrión]]
- [[01-SERVICIOS/apps-web]] - Sistema PHP legacy

---

## Próximas investigaciones

- [ ] Login flow (frontend → backend → auth service)
- [ ] Create patient flow (frontend → backend → database)
- [ ] Appointment creation (frontend → backend → calendar service)
- [ ] JWT validation in Gateway
- [ ] Multi-sede authentication logic
- [ ] `le_aspi7.php` — ¿existe reactivación desde `anu = 7`?
- [ ] Lógica equivalente de `anu` en días 2, 3 y 4 (`le_aspi2.php` — `le_aspi4.php`)

---

**Última actualización:** 2026-04-15
**Mantenido por:** Obsidian Documenter Agent
