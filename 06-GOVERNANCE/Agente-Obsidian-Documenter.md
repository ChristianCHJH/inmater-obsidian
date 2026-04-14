# Agente Obsidian Documenter

**Especificación:** `.claude/commands/obsidian-documenter.md`

## 🤖 ¿Qué es?

Agente global especializado en **documentar automáticamente el proyecto inmater-core en Obsidian**. Analiza código, determina qué documentar, y genera documentos siguiendo la estructura del vault.

## 🎯 Capacidades

El agente puede:

1. ✅ **Documentar endpoints**
   - Analiza controller + route
   - Genera en `01-SERVICIOS/[service]/endpoints/`
   - Crea backlinks automáticamente

2. ✅ **Crear traces (investigaciones)**
   - Flujo completo: UI → API → Controller → DB
   - Organiza por layer (frontend/backend/database)
   - Resuelve cómo funciona algo

3. ✅ **Documentar lógica de negocio**
   - Identifica si es compartida (→ `02-DOMINIOS/`) o específica (→ `01-SERVICIOS/`)
   - Linkea servicios afectados
   - Maneja dependencias

4. ✅ **Documentar patrones**
   - Va a `02-DOMINIOS/_shared-patterns/` o `03-ARQUITECTURA/patterns/`
   - Incluye ejemplos de código
   - Linkea usos

5. ✅ **Crear ADRs (decisiones arquitectónicas)**
   - Genera en `03-ARQUITECTURA/decisions/`
   - Linkea servicios afectados
   - Usa template

6. ✅ **Actualizar catálogos**
   - `04-REFERENCIAS/Endpoints-Catalog.md`
   - `04-REFERENCIAS/Tables-Catalog.md`
   - `04-REFERENCIAS/Services-Dependency-Map.md`

## 📋 Cómo decidir dónde documentar

El agente usa esta **matriz de decisión**:

| Pregunta | Destino | Ejemplo |
|----------|---------|---------|
| ¿Afecta múltiples servicios? | `02-DOMINIOS/[dominio]/` | JWT validation (Auth + Gateway) |
| ¿Es de 1 servicio solo? | `01-SERVICIOS/[service]/` | POST /pacientes endpoint |
| ¿Es un patrón reutilizable? | `02-DOMINIOS/_shared-patterns/` o `03-ARQUITECTURA/patterns/` | Repository pattern |
| ¿Es una decisión arquitectónica? | `03-ARQUITECTURA/decisions/` | JWT over sessions |
| ¿Es una investigación? | `05-TRACES/[layer]/` | "Cómo funciona el login" |
| ¿Necesita referencia rápida? | `04-REFERENCIAS/` | Listado de todos los endpoints |

## 🔥 Ejemplos de uso

### Ejemplo 1: Documentar un endpoint

```
Usuario: "Documenta el endpoint POST /pacientes del EMR.Patient.Service"

Agente:
1. Analiza: src/presentation/controllers/PatientController.ts
2. Decide: Type=endpoint, Destino=01-SERVICIOS/emr-patient-service/endpoints/
3. Genera: POST-create-patient.md
4. Incluye:
   - Descripción
   - Request/Response schema
   - Validaciones (DNI, email)
   - Tablas afectadas (hc_paciente, hc_paciente_contact)
   - Links a: [[02-DOMINIOS/pacientes]], [[04-REFERENCIAS/Tables-Catalog]]
5. Actualiza: 04-REFERENCIAS/Endpoints-Catalog.md
```

**Resultado:**
```
01-SERVICIOS/emr-patient-service/endpoints/
└── POST-create-patient.md

---
title: POST /microservicesPacientes/pacientes
projects: [[EMR.Patient.Service]], [[EMR.Api-Gateway.Service]]
domain: [[02-DOMINIOS/pacientes]]
layer: backend
type: endpoint
status: documented
---

## Crear nuevo paciente
...
```

### Ejemplo 2: Crear una trace

```
Usuario: "Crea un trace del flujo de login completo"

Agente:
1. Analiza: frontend button → axios call → backend auth
2. Decide: Type=trace, Destino=05-TRACES/
3. Genera: login-flow.md
4. Documenta:
   - Frontend: Button click → useAuth composable → POST /auth/login
   - Backend: AuthController.login → JWT generation → DB lookup
   - Database: SELECT FROM login WHERE usuario = ?
5. Linkea capas: frontend → backend → database
```

**Resultado:**
```
05-TRACES/frontend/
└── login-flow.md

05-TRACES/backend/
└── jwt-validation-flow.md

05-TRACES/database/
└── login-credentials-query.md
```

### Ejemplo 3: Documentar patrón compartido

```
Usuario: "Documenta el patrón Repository que usan todos los servicios"

Agente:
1. Analiza: src/infrastructure/UseRepository/
2. Decide: Type=pattern, Destino=02-DOMINIOS/_shared-patterns/
3. Genera: repository-pattern.md
4. Linkea servicios que lo usan:
   - [[EMR.Patient.Service]]
   - [[EMR.Auth.Service]]
   - [[EMR.Calendar.Service]]
   - [[EMR.Patient.Service]]
5. Incluye: Ejemplos de código, interfaces, implementación
```

### Ejemplo 4: Actualizar catálogo de endpoints

```
Usuario: "Actualiza el catálogo de endpoints con los nuevos"

Agente:
1. Escanea: todos los servicios
2. Extrae: GET /pacientes, POST /pacientes, etc.
3. Actualiza: 04-REFERENCIAS/Endpoints-Catalog.md
4. Formato:
   | Método | Path | Servicio | Descripción |
   |--------|------|----------|-------------|
   | GET | /microservicesPacientes/pacientes | EMR.Patient.Service | Listar pacientes |
```

## 🏷️ Metadatos generados automáticamente

Cada documento incluye frontmatter con:

```yaml
---
title: [Título]
projects: [[Servicio1]], [[Servicio2]]          # Servicios involucrados
domain: [[dominio]]                             # Qué dominio afecta
layer: backend|frontend|database|infra         # Qué capa
type: endpoint|trace|pattern|adr|domain        # Qué tipo de doc
status: documented|needs-review|deprecated     # Estado
created: 2026-04-13                            # Fecha creación
last-reviewed: 2026-04-13                      # Última revisión
tech-stack:
  - Express                                     # Stack técnico
  - TypeScript
  - PostgreSQL
backlog: false                                  # ¿Pendiente?
---
```

## 🔗 Linkeo automático

El agente crea backlinks:

```markdown
## Relacionado
- **Dominio:** [[02-DOMINIOS/pacientes]]
- **Servicio:** [[01-SERVICIOS/emr-patient-service]]
- **Patrón:** [[03-ARQUITECTURA/patterns/controller-service-repo]]
- **ADR:** [[03-ARQUITECTURA/decisions/0005-error-response-format]]
- **Tablas:** [[04-REFERENCIAS/Tables-Catalog#hc_paciente]]
- **Trace:** [[05-TRACES/backend/create-patient-flow]]

## Implementado por
- [[EMR.Patient.Service#PatientController]]
- [[EMR.Api-Gateway.Service#Routing#Pacientes]]
```

## 🎓 Reglas del agente

1. ✅ **No duplica:** Si existe, actualiza
2. ✅ **Links siempre:** Cada doc linkea a relacionados
3. ✅ **Nombres claros:** Descriptivos, no genéricos
4. ✅ **Metadatos completos:** Frontmatter siempre
5. ✅ **Catálogos sincronizados:** Actualiza 04-REFERENCIAS/
6. ✅ **Status tracking:** Mantiene control de documentación
7. ✅ **Índices actualizados:** Cuando crea, mueve o elimina un documento en una carpeta, actualiza el `{carpeta}_index.md` de esa carpeta con el nuevo listado

## 📍 Localización del agente

- **Definición:** `inmater-core/.claude/commands/obsidian-documenter.md`
- **Uso:** Desde Claude Code o CLI
- **Datos:** Lee del proyecto inmater-core
- **Escribe:** En el vault en `c:/Users/Christian/Documents/inmater/`

## 🚀 Cómo invocar

Desde Claude Code, puedes invocar:

```bash
# Documentar un endpoint específico
/obsidian-documenter endpoint \
  --service emr-patient-service \
  --method POST \
  --path "/pacientes"

# Crear una trace
/obsidian-documenter trace \
  --origin "patient-registration-button"

# Documentar un patrón
/obsidian-documenter pattern \
  --name "Repository Pattern"

# Crear ADR
/obsidian-documenter adr \
  --title "JWT token rotation strategy"

# Actualizar catálogos
/obsidian-documenter catalog --type endpoints
```

## 🔄 Flujo típico de documentación

```
Implementas una feature
    ↓
Invocas: /obsidian-documenter [tipo] [args]
    ↓
Agente analiza código
    ↓
Agente determina:
  - Qué documentar
  - Dónde ponerlo
  - Qué linkear
    ↓
Agente genera documento
    ↓
Actualiza catálogos
    ↓
Actualiza {carpeta}_index.md
    ↓
Documento está listo en Obsidian
    ↓
Links sincronizados
```

## 📊 Integración con governance

El agente respeta:
- ✅ Estructura en `Estructura-Vault-Inmater.md`
- ✅ Templates en `06-GOVERNANCE/templates/`
- ✅ Roles en `06-GOVERNANCE/roles.md`
- ✅ Convenciones en `Git-Convenciones.md`

## ⚙️ Tecnología detrás

- **Analiza:** TypeScript/JavaScript files via Grep
- **Lee:** Controladores, routes, servicios
- **Genera:** Markdown con structure completa
- **Linkea:** Via Obsidian `[[wiki-link]]`
- **Metadatos:** YAML frontmatter

---

**Status:** ✅ Disponible y listo para usar
**Última actualización:** 2026-04-13
**Ubicación especificación:** `inmater-core/.claude/commands/obsidian-documenter.md`
