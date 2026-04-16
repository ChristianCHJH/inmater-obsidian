---
title: "Trace: Flujo Inicializar PinPad"
projects: [[Frontend-Vue3]]
domain: gestor-pagos
layer: frontend
type: trace
status: documented
created: 2026-04-15
last-reviewed: 2026-04-15
tech-stack:
  - Vue 3
  - PrimeVue 4
  - TailwindCSS 4
  - TypeScript
backlog: false
---

# Trace: Flujo Inicializar PinPad

**Última actualización:** 2026-04-15
**Estado:** 🟢 Documentado

## 📍 Origen (dónde comienza)

- **Componente:** `PinpadStatusCard.vue`
- **Archivo:** `apps/frontend/src/modules/gestor-pagos/components/PinpadStatusCard.vue`
- **Evento:** Click en botón "Inicializar" → emite `inicializar: [idBridge: string]`

## 🎯 Objetivo

Documentar el flujo completo de: **inicialización de un terminal PinPad (Niubiz Bridge) desde el Panel de Control**, incluyendo estados del modal, manejo de respuesta exitosa con reporte, y manejo de errores con descripción humanizada del statusCode.

---

## 📊 Flujo por capas

### Capa Frontend (UI)

#### 1. `PinpadStatusCard.vue` — botón disparador

```typescript
// Emite evento al padre con el ID del bridge
emit('inicializar', bridge.id)
```

Props relevantes:
- `inicializando: boolean` — deshabilita "Validar conexión" y muestra loading en el botón
- `validando: boolean` — deshabilita "Inicializar" mientras se valida

#### 2. `GestorPagosPage.vue` — orquestador

```typescript
async function onInicializar(idBridge: string) {
  idBridgeInit.value = idBridge
  await inicializarTerminal(idBridge)   // delega al composable
}
```

El template pasa el estado de carga por terminal:
```html
<PinpadStatusCard
  :inicializando="inicializandoTerminal[bridge.id] ?? false"
  @inicializar="onInicializar"
/>
```

#### 3. `usePinpadInit.ts` — composable de estado

**Archivo:** `apps/frontend/src/modules/gestor-pagos/composables/usePinpadInit.ts`

Estados manejados:
```typescript
const mostrarModal = ref(false)
const estado = ref<'loading' | 'success' | 'error'>('loading')
const terminalNombre = ref<string | null>(null)
const reporte = ref<string | null>(null)
const errorInfo = reactive<{
  statusCode?: string
  responseCode?: string
  mensaje?: string
  descripcion?: string
}>({})
const inicializandoTerminal = reactive<Record<string, boolean>>({})
```

Flujo de `inicializar(idBridge)`:
```
1. Mapea idBridge → nombre legible (sede · moneda)
2. Resetea errorInfo y reporte
3. inicializandoTerminal[idBridge] = true
4. estado = 'loading', mostrarModal = true
5. Llama a inicializarPinpad(idBridge) desde terminal.api.ts
6a. ÉXITO: response === 'TRANSACCION EXITOSA'
    → reporte = resultado.report
    → estado = 'success'
6b. ERROR: response === 'TRANSACCION FALLIDA'
    → errorInfo.statusCode = resultado.statusCode
    → errorInfo.descripcion = getPinpadErrorInfo(statusCode)?.message
    → estado = 'error'
6c. EXCEPCIÓN (red caída, bridge no disponible)
    → errorInfo.mensaje = e.message
    → estado = 'error'
7. inicializandoTerminal[idBridge] = false
```

#### 4. `terminal.api.ts` — llamada al Bridge

**Archivo:** `apps/frontend/src/modules/gestor-pagos/api/terminal.api.ts`

```typescript
export async function inicializarPinpad(idBridge: string): Promise<RespuestaInicializacion> {
  const bridgeUrl = getBridgeUrl(idBridge)
  const response = await fetchConLog(idBridge, '/pcl/init', `${bridgeUrl}/pcl/init`)
  const data = await response.json()
  return data as RespuestaInicializacion
}
```

- Método HTTP: `GET`
- Endpoint Bridge: `{bridgeUrl}/pcl/init`
- `fetchConLog()` registra automáticamente la petición en DB vía `registrarLogPeticion()`

URLs de bridge por terminal:

| ID Bridge | Bridge URL (proxy Vite) | Puerto |
|-----------|------------------------|--------|
| MR-PEN | `/pinpad/MR-PEN/pcl/init` | 8679 |
| MR-USD | `/pinpad/MR-USD/pcl/init` | 8676 |
| FEBA-PEN | `/pinpad/FEBA-PEN/pcl/init` | 8689 |
| FEBA-USD | `/pinpad/FEBA-USD/pcl/init` | 8686 |

#### 5. `PinpadInitDialog.vue` — modal de resultado

**Archivo:** `apps/frontend/src/modules/gestor-pagos/components/PinpadInitDialog.vue`

Estados del modal:

| Estado | UI |
|--------|----|
| `loading` | Spinner animado + "Inicializando terminal..." |
| `success` | Checkmark verde + reporte completo en `<pre>` scrollable |
| `error` | Ícono rojo + statusCode + descripción humanizada + responseCode (si existe) |

El botón "Reintentar" emite `reintentar: [idBridge]` → llama `reintentarInit(idBridge)` en el composable, que re-ejecuta el flujo completo.

---

### Respuesta del Bridge Niubiz

**Éxito:**
```json
{
  "response": "TRANSACCION EXITOSA",
  "report": "\nFECHA: HORA:\n13/06/2023 16:43:12\n CONFIGURACION\n..."
}
```

**Error:**
```json
{
  "response": "TRANSACCION FALLIDA",
  "statusCode": "XX",
  "responseCode": "YY"
}
```

---

### Tabla de statusCode (Niubiz)

Definida en `apps/frontend/src/modules/gestor-pagos/types/pinpad.ts`:

| Código | Descripción | Tipo |
|--------|-------------|------|
| `02` | Se venció el tiempo de espera durante la operación | error |
| `03` | El terminal no permite realizar la operación o el usuario canceló presionando la tecla roja | rejected |
| `05` | La operación no se puede realizar porque el lote está vacío | error |
| `08` | Error de comunicación con el HOST | error |
| `09` | Tarjeta no permitida para la operación seleccionada | rejected |
| `0B` | Error interno durante el procesamiento de la transacción | error |
| `FF` | Error generando un ticket o reporte. Contactar a soporte | error |

---

## 🔗 Relaciones

| Aspecto | Relacionado | Notas |
|---------|-------------|-------|
| **Módulo** | `apps/frontend/src/modules/gestor-pagos/` | Feature module |
| **Composable** | `composables/usePinpadInit.ts` | Estado y lógica del flujo |
| **API** | `api/terminal.api.ts` | Llamada al bridge Niubiz |
| **Tipos error** | `types/pinpad.ts` | PINPAD_STATUS_CODES |
| **Dialog** | `components/PinpadInitDialog.vue` | UI del resultado |
| **Card** | `components/PinpadStatusCard.vue` | Origen del evento |
| **Vista** | `views/GestorPagosPage.vue` | Orquestador principal |
| **Patrón similar** | `composables/useCobro.ts` | Mismo patrón loading/success/error |

---

## 🔍 Descubrimientos

- **`fetchConLog`** es un wrapper interno que auto-registra toda petición al bridge en la tabla de logs del backend (`/microservicesFinancialManagement/pinpad/logPeticiones/create`). No requiere código adicional.
- **`inicializandoTerminal`** es un `Record<string, boolean>` reactivo que permite deshabilitar el botón del terminal específico sin bloquear los demás.
- El bridge Niubiz puede tardar hasta **90 segundos** en responder a `/pcl/init` (timeout de respuesta configurado en el bridge).
- El `report` de la respuesta exitosa es un string de texto plano con formato de ticket, renderizado en `<pre>` con fuente monospace.
- El modal no se puede cerrar mientras `estado === 'loading'` (`:closable="estado !== 'loading'"`).

---

## 📝 Archivos creados/modificados

| Archivo | Cambio |
|---------|--------|
| `api/terminal.api.ts` | +`inicializarPinpad()` + interface `RespuestaInicializacion` |
| `composables/usePinpadInit.ts` | Nuevo composable |
| `components/PinpadInitDialog.vue` | Nuevo componente modal |
| `components/PinpadStatusCard.vue` | +prop `inicializando`, +emit `inicializar`, +botón |
| `views/GestorPagosPage.vue` | Conecta composable + dialog |
| `types/pinpad.ts` | Mensajes de statusCode actualizados a documentación oficial Niubiz |

---

**Tags:** #layer/frontend #domain/gestor-pagos #pinpad #niubiz
**Última revisión:** 2026-04-15
