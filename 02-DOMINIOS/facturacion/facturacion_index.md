# Índice: Dominio Facturación

**Última actualización:** 2026-04-15
**Estado:** 🟡 En construcción

## Archivos en esta carpeta

| Archivo | Descripción | Servicios |
|---------|-------------|-----------|
| [[gestor-pagos]] | Lógica del módulo Gestor de Pagos (POS Niubiz): transacciones, anulaciones, vouchers | `EMR.Financial-Management.Service`, `apps/frontend` |

## Relaciones con otras carpetas

- **Implementado por:** `EMR.Financial-Management.Service`
- **Frontend:** `apps/frontend/src/modules/gestor-pagos/`
- **Traces relacionadas:** [[05-TRACES/frontend/voucher-anulacion-gestor-pagos]], [[05-TRACES/frontend/gestor-pagos-dev-mode-401-cascade]]
- **Patrones compartidos:** [[02-DOMINIOS/_shared-patterns/shared-patterns_index]]

## Pendiente documentar

- [ ] Flujo completo de emisión de comprobante (boleta/factura)
- [ ] Integración con el módulo de citas (¿pagos vinculados a citas?)
- [ ] Lógica de cierre de lote (`lotesActivos`)
- [ ] Puentes (bridges) EMR-PEN, EMR-USD, FEVA-PEN, FEBA-USD y su rol

---

**Tags:** #domain/facturacion
**Última actualización:** 2026-04-15
