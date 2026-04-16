---
title: "Mifact — Integración de Facturación Electrónica (OSE SUNAT)"
projects: [[apps/web]]
domain: [[02-DOMINIOS/facturacion/facturacion_index]]
layer: backend
type: domain
status: documented
created: 2026-04-16
last-reviewed: 2026-04-16
tech-stack:
  - PHP
  - mifact API (REST/JSON)
  - SUNAT UBL 2.1
backlog: false
---

# Mifact — Integración de Facturación Electrónica

**Proveedor:** [mifact.net.pe](https://mifact.net.pe) — OSE (Operador de Servicios Económicos) certificado SUNAT  
**Implementación PHP:** `apps/web/_database/db_facturacion_electronica.php`  
**Documentación fuente:** `docs/christian/mifact.documentacion.xlsx` + `docs/christian/apijson-master/`

---

## Descripción

Mifact es el OSE que intermedia entre el sistema inmater y SUNAT para la emisión, anulación y consulta de comprobantes electrónicos (facturas, boletas, notas de crédito/débito). El sistema envía un JSON estructurado a mifact, que genera el XML UBL 2.1, lo firma digitalmente y lo envía a SUNAT.

---

## URLs de conexión

| Ambiente | URL base | Uso |
|----------|----------|-----|
| **Producción** | `https://mifact.net.pe/bmifactapi/invoiceService.svc` | Activo en `$_ENV["FACTURACION_ELECTRONICA_SERVICE"]` |
| **Demo/Testing** | `https://demo.mifact.net.pe/api/invoiceService.svc` | Pruebas con RUC `20100100100` |
| **Guía Remisión** | `https://demo.mifact.net.pe/api/GuiaRemision.svc` | Solo demo (no usado actualmente) |
| **Retenciones** | `https://demo.mifact.net.pe/api/RetencionService.svc` | Solo demo (no usado actualmente) |

> **Importante:** La URL por empresa está en `man_empresas.service_mifact`. El `$_ENV` es un fallback global; en producción cada empresa tiene su propia URL.

**Credenciales de demo (solo pruebas):**
- Token: `gN8zNRBV+/FVxTLwdaZx0w==`
- RUC: `20100100100`

---

## Configuración por empresa (`man_empresas`)

Cada empresa tiene su propia configuración de mifact almacenada en `man_empresas`:

| Campo DB | Descripción |
|----------|-------------|
| `service_mifact` | URL base del servicio mifact para esta empresa |
| `token` | Token de autenticación (rotativo cada ~6 meses) |
| `enviar_a_sunat` | `true` = envío inmediato a SUNAT; `false` = diferido (lo envía mifact en horario programado) |
| `cod_tip_nif_emis` | Tipo de documento del emisor (siempre `6` = RUC) |
| `num_nif_emis` | RUC del emisor |
| `nom_rzn_soc_emis` | Razón social del emisor (exacta como SUNAT) |
| `nom_comer_emis` | Nombre comercial (si tiene en SUNAT) |
| `cod_ubi_emis` | Ubigeo INEI de la dirección fiscal |
| `txt_dmcl_fisc_emis` | Dirección fiscal del emisor |

Se administra desde `empresas_man_edit.php`.

---

## Métodos API utilizados por inmater

### `POST /SendInvoice` — Emitir comprobante

**Función PHP:** `cargar_facturacion_electronica()` + `enviar_facturacion_electronica($data, "/SendInvoice", $empresa)`  
**Invocado desde:** `_database/pago.php` al crear un recibo

**Request (campos principales):**

```json
{
  "TOKEN": "...",
  "COD_TIP_NIF_EMIS": "6",
  "NUM_NIF_EMIS": "RUC_EMISOR",
  "NOM_RZN_SOC_EMIS": "Razón Social",
  "NOM_COMER_EMIS": "Nombre comercial",
  "COD_UBI_EMIS": "150101",
  "TXT_DMCL_FISC_EMIS": "Dirección fiscal",
  "COD_TIP_NIF_RECP": "6",
  "NUM_NIF_RECP": "RUC_RECEPTOR",
  "NOM_RZN_SOC_RECP": "Nombre receptor",
  "TXT_DMCL_FISC_RECEP": "Dirección receptor",
  "FEC_EMIS": "2026-04-16",
  "FEC_VENCIMIENTO": "2026-04-16",
  "COD_TIP_CPE": "01",
  "NUM_SERIE_CPE": "F001",
  "NUM_CORRE_CPE": "00000001",
  "COD_MND": "PEN",
  "MNT_TOT_GRAVADO": "847.46",
  "MNT_TOT_TRIB_IGV": "152.54",
  "MNT_TOT": "1000.00",
  "ENVIAR_A_SUNAT": "true",
  "RETORNA_XML_ENVIO": "false",
  "RETORNA_XML_CDR": "true",
  "RETORNA_PDF": "false",
  "COD_FORM_IMPR": "003",
  "TXT_VERS_UBL": "2.1",
  "TXT_VERS_ESTRUCT_UBL": "2.0",
  "COD_ANEXO_EMIS": "0000",
  "COD_TIP_OPE_SUNAT": "0101",
  "items": [...]
}
```

**Tipo de documento (COD_TIP_CPE) — mapeo con campo `tip` de `recibos`:**

| `recibos.tip` | COD_TIP_CPE | Tipo comprobante | Serie |
|--------------|-------------|-----------------|-------|
| `1` | `03` | Boleta | `B...` |
| `2` | `01` | Factura | `F...` |
| — | `07` | Nota de Crédito | `F...` o `B...` |
| — | `08` | Nota de Débito | `F...` o `B...` |
| — | `09` | Guía Remisión Remitente | — |

**Estructura de un ítem:**

```json
{
  "COD_ITEM": "ID_SERVICIO_INTERNO",
  "COD_UNID_ITEM": "NIU",
  "CANT_UNID_ITEM": "1",
  "VAL_UNIT_ITEM": "847.46",
  "PRC_VTA_UNIT_ITEM": "1000.00",
  "VAL_VTA_ITEM": "847.46",
  "MNT_PV_ITEM": "1000.00",
  "COD_TIP_PRC_VTA": "01",
  "COD_TIP_AFECT_IGV_ITEM": "10",
  "COD_TRIB_IGV_ITEM": "1000",
  "POR_IGV_ITEM": "18",
  "MNT_IGV_ITEM": "152.54",
  "TXT_DESC_ITEM": "Descripción del servicio"
}
```

**Códigos de afectación IGV (COD_TIP_AFECT_IGV_ITEM):**

| Código | Significado | Cuándo aplica en inmater |
|--------|-------------|--------------------------|
| `10` | Afecto a IGV (gravado) | Servicio normal |
| `13` | Gratuito gravado | `recibos.gratuito = 1` |
| `20` | Exonerado | `exonerado_igv = 1` (no domiciliados) |

**Códigos de tributo (COD_TRIB_IGV_ITEM):**

| Código | Significado |
|--------|-------------|
| `1000` | IGV |
| `9996` | GRA (gratuito) |
| `9997` | EXO (exonerado) |

---

### `POST /LowInvoice` — Anular comprobante

**Función PHP:** `anular_facturacion_electronica()`

**Request:**
```json
{
  "TOKEN": "...",
  "COD_TIP_NIF_EMIS": "6",
  "NUM_NIF_EMIS": "RUC_EMISOR",
  "FEC_EMIS": "2026-04-16",
  "COD_TIP_CPE": "01",
  "NUM_SERIE_CPE": "F001",
  "NUM_CORRE_CPE": "00000001",
  "TXT_DESC_MTVO": "ANULACION POR ERROR"
}
```

> SUNAT por normativa solo permite anular desde la clave SOL en producción para guías de remisión. Para facturas y boletas sí funciona vía API dentro del mismo día.

---

### `POST /GetEstatusInvoice` — Consultar estado en SUNAT

**Función PHP:** `consultar_facturacion_electronica()`

**Request:**
```json
{
  "TOKEN": "...",
  "COD_TIP_NIF_EMIS": "6",
  "NUM_NIF_EMIS": "RUC_EMISOR",
  "COD_TIP_CPE": "01",
  "NUM_SERIE_CPE": "F001",
  "NUM_CORRE_CPE": "00000001"
}
```

---

### `POST /GetInvoice` — Obtener PDF/XML/CDR

**Función PHP:** `impresion_facturacion_electronica()`

**Request:**
```json
{
  "TOKEN": "...",
  "NUM_NIF_EMIS": "RUC_EMISOR",
  "COD_TIP_CPE": "01",
  "NUM_SERIE_CPE": "F001",
  "NUM_CORRE_CPE": "00000001",
  "FEC_EMIS": "2026-04-16",
  "RETORNA_XML_ENVIO": false,
  "RETORNA_XML_CDR": false,
  "RETORNA_PDF": true,
  "COD_FORM_IMPR": "003"
}
```

**Formatos de impresión (COD_FORM_IMPR):**

| Código | Formato |
|--------|---------|
| `001` | A4 |
| `002` | A5 |
| `003` | Ticket matricial (usado por inmater) |
| `004` | Ticket térmico 80mm |

---

## Estados de documento (`estado_documento`)

| Código | Estado | Descripción |
|--------|--------|-------------|
| `101` | En proceso | Enviado a mifact, pendiente de SUNAT |
| `102` | Aceptado | SUNAT aceptó el comprobante |
| `103` | Aceptado con observación | SUNAT aceptó pero con advertencias |
| `104` | Rechazado | SUNAT rechazó el comprobante |
| `105` | Anulado | Baja aplicada |
| `108` | Solicitud de baja pendiente | Baja no enviada a SUNAT aún |

Los estados `101`, `102`, `103` se consideran "válidos" en las queries de inmater (documentos imprimibles).

---

## Tablas de respuesta en BD

### `facturacion_recibo_mifact_response` (principal)

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `id` | int | PK |
| `id_recibo` | int | FK → `recibos.id` |
| `tip_recibo` | int | FK → `recibos.tip` (1=boleta, 2=factura) |
| `estado_documento` | varchar | Código de estado (101–108) |
| `serie_cpe` | varchar | Serie del comprobante (ej: `F001`) |
| `correlativo_cpe` | varchar | Correlativo (ej: `00000001`) |
| `tipo_cpe` | varchar | Tipo (01, 03, etc.) |
| `cadena_para_codigo_qr` | text | Cadena para generar el QR del comprobante |
| `codigo_hash` | varchar | Hash del XML (firma) |
| `cdr_sunat` | text | CDR (Constancia de Recepción) de SUNAT en base64 |
| `xml_enviado` | text | XML UBL 2.1 enviado a SUNAT en base64 |
| `pdf_bytes` | longtext | PDF del comprobante en base64 (si se solicitó) |
| `ticket_sunat` | varchar | Ticket de SUNAT para consulta asíncrona |
| `sunat_responsecode` | varchar | Código de respuesta SUNAT (0=sin error) |
| `sunat_description` | text | Descripción de la respuesta SUNAT |
| `sunat_note` | text | Observaciones de SUNAT |
| `errors` | text | Errores de mifact (vacío si OK) |
| `url` | varchar | URL del comprobante en mifact |
| `estado` | tinyint | 1=activo |
| `createdate` | datetime | Fecha de creación |
| `idusercreate` | varchar | Usuario que emitió |

### `factu_mifact_response` (legacy)

Tabla anterior para log de request/response. Solo almacena el JSON crudo y el error. Está en proceso de deprecación implícita (se inserta en ambas tablas).

---

## Lógica de cálculo de totales (PHP)

```php
// Servicio gravado normal
$mnt_tot_gravado = tot / 1.18         // base imponible sin IGV
$mnt_tot_trib_igv = tot * 0.18 / 1.18 // IGV
$mnt_tot = tot                         // total con IGV

// Servicio gratuito (gratuito=1)
$mnt_tot_gratuito = tot / 1.18
// → COD_TIP_AFECT_IGV_ITEM = "13", COD_TRIB_IGV_ITEM = "9996"

// No domiciliado (exonerado_igv=1)
$mnt_tot_exonerado = tot
$mnt_tot_trib_igv = 0
$mnt_tot_gravado = 0
// → COD_TIP_OPE_SUNAT = "0401" (en lugar de "0101")
// → COD_TIP_AFECT_IGV_ITEM = "20", COD_TRIB_IGV_ITEM = "9997"

// Pago a crédito (condicion_pago_id=2)
$mifact["MNT_PENDIENTE"] = tot * 0.88  // 88% del total (neto de detracción 12%)
```

---

## Tipos de operación SUNAT (COD_TIP_OPE_SUNAT — Catálogo 51)

| Código | Descripción | Cuándo aplica |
|--------|-------------|---------------|
| `0101` | Venta interna | Caso general |
| `0401` | Exportación | No domiciliados (exonerado_igv=1) |

---

## Casuísticas soportadas (según ejemplos en apijson-master)

Documentados en `apijson-master/integracionConJson_FV_BV_NC_ND/Ejemplos Archivos JSON UBL 2_1/`:

| Casuística | Ejemplo disponible |
|------------|--------------------|
| Factura con IGV tradicional | ✅ `Factura_con_IGV_TRADICIONAL.txt` |
| Boleta tradicional | ✅ `boleta_tradicional.txt` |
| Factura exonerada | ✅ `factura_exonerada.txt` |
| Factura exportación bien/servicio | ✅ |
| Nota de crédito | ✅ `nota de credito.txt` |
| Nota de débito | ✅ `nota de debito.txt` |
| Factura con detracción | ✅ (varios tipos: 1001–1004) |
| Factura con anticipo parcial/total | ✅ |
| Factura con ISC | ✅ |
| Factura con percepción | ✅ |
| Factura con descuento global/por ítem | ✅ |
| Factura con recargo (propina) | ✅ |
| Anulación de documento | ✅ `ANULACION_DOCUMENTO.txt` |
| Consulta estado | ✅ `CONSULTAR ESTADO_DOCUMENTO.txt` |
| Obtener PDF/XML/CDR | ✅ `CONSULTAR_PDF_XML_CDR.txt` |
| Retención | ✅ (`RetencionService.svc`) |
| Percepción | ✅ (`RetencionService.svc`) |
| Guía remisión remitente | ✅ (`GuiaRemision.svc`) |
| Guía remisión transportista | ✅ (`GuiaRemision.svc`) |

> **Nota:** Inmater actualmente solo usa `SendInvoice`, `LowInvoice`, `GetEstatusInvoice` y `GetInvoice`. Guías de remisión y retenciones/percepciones no están implementadas.

---

## Flujo completo de emisión

```
pago.php (submit)
  ↓
_database/pago.php → recibo()
  ↓
cargar_facturacion_electronica($data)
  → Calcula totales (gravado/exonerado/gratuito)
  → Arma JSON con datos de empresa, receptor, items
  → cargar_facturacion_electronica_detalle() parsea el HTML de servicios seleccionados
  ↓
enviar_facturacion_electronica($request, "/SendInvoice", $empresa)
  → POST a $empresa["service_mifact"] + "/SendInvoice"
  → file_get_contents() con stream_context (HTTP POST)
  → Retorna JSON decodificado
  ↓
INSERT INTO factu_mifact_response (log legacy)
INSERT INTO facturacion_recibo_mifact_response (response estructurada)
```

---

## Integración con guía remisión y retenciones (no implementado en inmater)

Los endpoints existen en mifact y están documentados en `apijson-master/`:

| Servicio | Métodos disponibles |
|----------|---------------------|
| `GuiaRemision.svc` | `SendGuia`, `LowGuia`, `GetEstatusGuia`, `GetGuia`, `SendMailGuia` |
| `RetencionService.svc` | `SendRetencion`, `LowRetencion`, `GetEstatusRetencion`, `GetRetencion` |

> Pendiente: Si inmater empieza a emitir guías de remisión o retenciones, consultar los ejemplos en `apijson-master/` para la estructura del JSON requerida.

---

## Archivos de referencia

| Archivo | Descripción |
|---------|-------------|
| `apps/web/_database/db_facturacion_electronica.php` | Toda la integración PHP con mifact |
| `apps/web/_database/pago.php` (línea ~305) | Invocación de `enviar_facturacion_electronica` |
| `apps/web/config/environment.php` (línea 13) | URL global de producción |
| `apps/web/empresas_man_edit.php` | UI para configurar mifact por empresa |
| `docs/christian/mifact.documentacion.xlsx` | Documentación interna (Excel binario — abrir con Excel) |
| `docs/christian/apijson-master/` | Documentación oficial mifact + ejemplos JSON |
| `docs/christian/apijson-master/integracionConJson_FV_BV_NC_ND/DocumentacionFV_BV_NC_ND_Json.xlsx` | Catálogo de campos FV/BV/NC/ND |
| `docs/christian/apijson-master/integracionConJson_GuiaRemision/DocumentacionGuiaRemisionRemitenteJson.xlsx` | Catálogo guía remisión |
| `docs/christian/apijson-master/integracionConJson_RetencionesPercepciones/DocumentacionTecnica_Valores.xlsx` | Valores técnicos retenciones |

---

## Relacionado

- Módulo PHP: [[01-SERVICIOS/php-legacy/pago-emision-comprobantes]]
- Dominio facturación: [[02-DOMINIOS/facturacion/facturacion_index]]
- Pagos POS: [[02-DOMINIOS/facturacion/gestor-pagos]]
- Trace emisión: [[05-TRACES/backend/pos-recibo-vinculacion]]
