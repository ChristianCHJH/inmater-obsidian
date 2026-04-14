# Estructura del Vault - Inmater

**Última actualización:** 2026-04-13

## 📋 Visión General

Este vault documenta **múltiples proyectos con lógica de negocio compartida** usando una estructura **híbrida de 4 niveles**. Permite búsqueda global, evita duplicación, y mantiene contexto completo entre servicios.

---

## 🏗️ Los 4 Niveles

### Nivel 1: `01-SERVICIOS/` - Repositorios individuales

Carpeta por cada repo/servicio del ecosistema inmater-core.

```
01-SERVICIOS/
├── Servicios-Microservicios.md         # Catálogo general
├── emr-api-gateway/
│   ├── README.md                       # Entrada al servicio
│   ├── architecture/
│   ├── endpoints/                      # Catálogo de endpoints
│   ├── routing-pattern.md
│   └── deployment/
├── emr-auth-service/
├── emr-patient-service/
├── emr-calendar-service/
├── Frontend-Vue3.md                    # SPA app
└── apps-web/                           # PHP legacy
```

**Cuándo usarla:**
- Documentación específica de UN servicio
- Endpoints particulares
- Deployment/CI-CD de ese service
- Cambios schema locales

**Links:**
- Enlaza a `[[autenticacion]]` (dominio), `[[laboratorio]]` (dominio)
- Referencia patrones en `[[03-ARQUITECTURA]]`

---

### Nivel 2: `02-DOMINIOS/` - Lógica compartida (EJE CENTRAL)

Donde vive la **lógica de negocio** que afecta múltiples servicios.

```
02-DOMINIOS/
├── Procesos-Negocio.md                 # Flujos de negocio generales
│
├── autenticacion/
│   ├── flow.md                         # Cómo funciona auth
│   ├── multi-sede-logic.md             # Regla compartida
│   └── jwt-patterns.md                 # Implementación reutilizable
│   └── [[EMR.Auth.Service]], [[EMR.Api-Gateway.Service]]
│
├── pacientes/
│   ├── domain-model.md                 # DDD: agregados, entities
│   ├── validaciones.md                 # DNI, etc (reutilizable)
│   └── [[EMR.Patient.Service]]
│
├── calendario/
│   ├── turnos-logic.md
│   └── [[EMR.Calendar.Service]]
│
├── facturacion/
│   └── ...
│
├── laboratorio/                        # ⭐ Tu nuevo descubrimiento
│   ├── Laboratorio-Procesos.md         # Procesos generales
│   ├── Columna OUT - Estado del Embrión.md  # TRACE: lógica OUT
│   └── ciclos-evaluacion.md
│
└── _shared-patterns/                   # Reutilizable en TODOS
    ├── sequelize-setup.md              # Cómo usar ORM
    ├── error-responses.md              # Formato estándar
    ├── caching-strategy.md             # Estrategia cache
    ├── jwt-validation.md               # Validación JWT
    ├── testing-patterns.md             # Jest patterns
    └── repository-pattern.md           # Repository interface
```

**Cuándo usarla:**
- Reglas de negocio que afectan varios servicios
- Patrones implementados en múltiples repos
- Lógica que se reutiliza
- Decisiones de diseño compartido

**Ejemplo:**
```markdown
# [[autenticacion#Multi-sede Logic]]

Implementado en:
- [[EMR.Auth.Service]] - genera tokens multi-sede
- [[EMR.Api-Gateway.Service]] - valida tokens multi-sede
- [[apps-frontend]] - detecta sede activa

Ver patrón: [[_shared-patterns/jwt-validation.md]]
```

---

### Nivel 3: `03-ARQUITECTURA/` - Decisiones y patrones

Contenedor de decisiones arquitectónicas, patrones técnicos, y stack setup.

```
03-ARQUITECTURA/
│
├── Arquitectura-Stack-Tecnologico.md   # Overview del stack
│
├── decisions/                          # ADRs (Architecture Decision Records)
│   ├── 0003-backend-cicd-cloud-build.md
│   └── 0008-lab-service-adr.md
│
├── patterns/                           # Patrones reutilizables
│   ├── express-ddd.md                  # Estructura de servicios
│   ├── vue3-composition.md             # Frontend patterns
│   ├── controller-service-repo.md      # Layers en Express
│   └── testing-patterns.md             # Jest + TDD
│
└── infra/                              # Infraestructura
    ├── DevOps-Infraestructura.md       # GCP, Cloud Run, Firebase
    ├── Base de Datos-PostgreSQL.md     # Sequelize, schemas, setup
    ├── Docker-compose.md               # Local dev
    └── environment-vars.md             # Variables globales
```

**Cuándo usarla:**
- Decisiones arquitectónicas que afectan múltiples servicios
- Patrones técnicos reutilizables
- Setup de infraestructura
- Standards y convenciones

---

### Nivel 4: `04-REFERENCIAS/` - Catálogos globales

Índices y catálogos de referencia rápida.

```
04-REFERENCIAS/
├── Endpoints-Catalog.md                # Todos los endpoints de 6 servicios
├── Tables-Catalog.md                   # Todas las tablas PostgreSQL
├── Services-Dependency-Map.md          # Qué servicio depende de qué
├── Tech-Stack.md                       # Stack por servicio
└── Glossary-Medical-Terms.md          # Términos clínicos + técnicos
```

**Cuándo usarla:**
- Búsqueda rápida: ¿qué endpoints tiene Patient Service?
- ¿Qué tablas afecta esta query?
- ¿Qué servicios se comunican?

---

## 📊 Niveles complementarios (no jerárquicos)

### `05-TRACES/` - Investigaciones documentadas

Cada investigación que haces sobre "cómo funciona esto" se documenta como TRACE.

```
05-TRACES/
├── frontend/
│   ├── patient-registration-button.md
│   └── login-flow.md
├── backend/
│   ├── jwt-validation-flow.md
│   └── appointment-creation.md
└── database/
    ├── patient-query-optimization.md
    └── legacy-schema-mapping.md
```

**Cuándo crear:**
- Investigas "cómo funciona X"
- Descubres un patrón (como [[Columna OUT]])
- Necesitas documentar flujo completo

---

### `06-GOVERNANCE/` - Cómo mantener el vault

```
06-GOVERNANCE/
├── Estructura-Vault-Inmater.md         # TÚ ESTÁS AQUÍ
├── Git-Convenciones.md                 # Commits, branches, workflow
│
├── roles/
│   ├── architect.md                    # Diseña arquitectura
│   ├── builder.md                      # Implementa código
│   ├── reviewer.md                     # Revisa PRs
│   └── documentor.md                   # Actualiza documentación
│
└── templates/
    ├── trace-template.md               # Template para traces
    ├── adr-template.md                 # Template para ADRs
    ├── domain-model-template.md        # Template DDD
    └── endpoint-doc-template.md        # Template endpoints
```

---

## 🔗 Cómo navegar

### Si necesitas entender un SERVICIO
```
1. Ve a: 01-SERVICIOS/[nombre-servicio]/
2. Lee: README.md
3. Links te llevan a: 02-DOMINIOS/ (lógica compartida)
4. Links te llevan a: 03-ARQUITECTURA/ (patrones que usa)
```

### Si necesitas entender una REGLA DE NEGOCIO
```
1. Ve a: 02-DOMINIOS/[dominio]/
2. Lee: flow.md o logic.md
3. Verás qué servicios la implementan
4. Links te llevan a cada servicio en 01-SERVICIOS/
5. Links a patrones reutilizables en _shared-patterns/
```

### Si necesitas entender una DECISIÓN ARQUITECTÓNICA
```
1. Ve a: 03-ARQUITECTURA/decisions/
2. Lee el ADR correspondiente
3. Ve qué servicios la implementan
4. Links te llevan a 01-SERVICIOS/ o 02-DOMINIOS/
```

### Si necesitas un CATÁLOGO RÁPIDO
```
1. Ve a: 04-REFERENCIAS/
2. Busca lo que necesitas
3. Encontrarás índices cruzados
```

### Si necesitas investigar CÓMO FUNCIONA ALGO
```
1. Busca en: 05-TRACES/
2. O crea uno nuevo usando template en 06-GOVERNANCE/templates/
```

---

## 🏷️ Sistema de Tags (Metadatos)

Cada archivo tiene tags para clasificación cruzada:

```yaml
tags:
  - #project/emr-api-gateway        # Qué proyecto
  - #project/emr-auth-service
  - #domain/autenticacion           # Qué dominio
  - #domain/laboratorio
  - #layer/backend                  # Qué capa
  - #layer/frontend
  - #layer/database
  - #pattern/repository             # Qué patrón
  - #pattern/controller
  - #status/documented              # Estado
  - #status/needs-trace
  - #status/needs-adр
```

---

## 📐 Flujo de documentación (Governance)

Cuando **implementas algo nuevo**:

```
1. Código → Creas feature en repo

2. Si afecta LÓGICA COMPARTIDA:
   → Documenta en 02-DOMINIOS/[dominio]/

3. Si es PATRÓN REUTILIZABLE:
   → Documenta en 03-ARQUITECTURA/patterns/

4. SIEMPRE crea TRACE:
   → 05-TRACES/[layer]/[accion].md
   → Documenta: button → API call → controller → DB

5. Si hay DECISIÓN ARQUITECTÓNICA:
   → Crea ADR en 03-ARQUITECTURA/decisions/

6. Actualiza CATÁLOGOS:
   → 04-REFERENCIAS/[catalog] si es necesario
```

---

## 🔍 Búsqueda inteligente

### Usar Obsidian search para:
```
- Buscar #project/emr-api-gateway → todos los archivos de ese servicio
- Buscar #domain/laboratorio → toda lógica de laboratorio
- Buscar #pattern/repository → dónde se usa patrón Repository
- Buscar [[EMR.Auth.Service]] → qué lo referencia
```

### Usar Dataview para queries dinámicas:
```dataview
TABLE projects, domain, status
WHERE layer = "backend" AND status = "documented"
SORT last-reviewed DESC
```

---

## 📋 Principios de Organización

1. **Dos ejes principales:**
   - VERTICAL: `01-SERVICIOS/` (por proyecto)
   - HORIZONTAL: `02-DOMINIOS/` (lógica compartida)

2. **Evita duplicación:** Lógica compartida vive en `02-DOMINIOS/`

3. **Links son la navegación:** No necesitas carpetas profundas, usa `[[links]]`

4. **Nombres descriptivos:** No uses "INDEX", usa nombres que digan qué es

5. **Frontmatter + tags:** Metadata para búsqueda cruzada

6. **Una fuente de verdad:** Un archivo por concepto (no lo repitas)

---

## 📍 Ubicación actual vs recomendada

| Archivo | Dónde está | Dónde debería | Cambio |
|---------|-----------|--------------|--------|
| Columna OUT | `02-DOMINIOS/laboratorio/` | ✅ Correcto | ✓ |
| JWT patterns | `02-DOMINIOS/_shared-patterns/` | ✅ Correcto | ✓ |
| ADRs | `03-ARQUITECTURA/decisions/` | ✅ Correcto | ✓ |
| Endpoints | `04-REFERENCIAS/` | ✅ Correcto | ✓ |

---

## 📑 Convención de Índices (`{nombre}_index.md`)

**Regla:** Cada carpeta del vault tiene un archivo índice llamado `{nombre}_index.md` que lista y relaciona todos los archivos dentro.

**Propósito:** Navegar el vault sin depender del graph de Obsidian. Un archivo auto-documentado que responde: "¿Qué hay en esta carpeta?"

**Nombre único:** Usamos `{nombre}_index.md` en lugar de `INDEX.md` para evitar ambigüedad cuando hay muchas carpetas.

### Índices por nivel

| Nivel | Carpeta | Archivo índice | Contenido |
|-------|---------|---|-----------|
| **0** | Raíz | (no aplica) | Ver 00-INICIO.md |
| **1** | `01-SERVICIOS/` | `servicios_index.md` | Catálogo de servicios + enlaces |
| **1** | `02-DOMINIOS/` | `dominios_index.md` | Catálogo de dominios + enlaces |
| **2** | `02-DOMINIOS/laboratorio/` | `laboratorio_index.md` | Archivos del dominio laboratorio |
| **2** | `02-DOMINIOS/_shared-patterns/` | `shared-patterns_index.md` | Patrones reutilizables |
| **1** | `03-ARQUITECTURA/` | `arquitectura_index.md` | Decisiones, patrones, infra |
| **2** | `03-ARQUITECTURA/decisions/` | `decisions_index.md` | ADRs y decisiones arquitectónicas |
| **2** | `03-ARQUITECTURA/patterns/` | `patterns_index.md` | Patrones técnicos |
| **2** | `03-ARQUITECTURA/infra/` | `infra_index.md` | Infraestructura, DevOps, BD |
| **1** | `04-REFERENCIAS/` | `referencias_index.md` | Catálogos globales |
| **1** | `05-TRACES/` | `traces_index.md` | Investigaciones documentadas |
| **2** | `05-TRACES/frontend/` | `frontend-traces_index.md` | Trazas de capas frontend |
| **2** | `05-TRACES/backend/` | `backend-traces_index.md` | Trazas de capas backend |
| **2** | `05-TRACES/database/` | `database-traces_index.md` | Trazas de capas database |
| **1** | `06-GOVERNANCE/` | `governance_index.md` | Roles, templates, convenciones |
| **2** | `06-GOVERNANCE/roles/` | `roles_index.md` | Definiciones de roles |
| **2** | `06-GOVERNANCE/templates/` | `templates_index.md` | Templates para documentación |

### Estructura de un `*_index.md`

```markdown
# Índice: [Nombre de la carpeta]

## Archivos en esta carpeta

| Archivo | Descripción | Tags | Relacionado |
|---------|-------------|------|-----------|
| archivo-1.md | Qué es | #domain | [[02-DOMINIOS/...]] |
| archivo-2.md | Qué es | #pattern | [[01-SERVICIOS/...]] |

## Relaciones con otras carpetas

- **Depende de:** [[03-ARQUITECTURA/patterns/...]]
- **Es referenciado por:** [[01-SERVICIOS/...]]
- **Relacionado con:** [[02-DOMINIOS/...]]

## Metadatos

- **Última actualización:** YYYY-MM-DD
- **Archivos:** N
- **Estado:** 🟢 Actualizado | 🟡 Pendiente | 🔴 Obsoleto
```

### Cuándo actualizar el índice

El agente documentador actualiza automáticamente el `*_index.md` cuando:
1. ✅ Se crea un nuevo documento en una carpeta
2. ✅ Se mueve un documento a una carpeta diferente
3. ✅ Se elimina un documento
4. ✅ Se actualiza significativamente un documento existente

### Ejemplo: `02-DOMINIOS/laboratorio_index.md`

```markdown
# Índice: Dominio Laboratorio

## Archivos en esta carpeta

| Archivo | Descripción | Servicios |
|---------|-------------|----------|
| Laboratorio-Procesos.md | Flujos generales del dominio | EMR.Patient.Service |
| Columna OUT - Estado del Embrión.md | TRACE: lógica de estado embrionario | EMR.Patient.Service |
| ciclos-evaluacion.md | Reglas de evaluación de ciclos | EMR.Calendar.Service |

## Relaciones

- **Implementado por:** [[01-SERVICIOS/emr-patient-service]]
- **Usa patrones:** [[02-DOMINIOS/_shared-patterns/repository-pattern.md]]
```

---

## 🚀 Next steps

Cuando agregues más repositorios:

1. ✅ Crear carpeta en `01-SERVICIOS/[nombre]`
2. ✅ Documentar lógica compartida en `02-DOMINIOS/`
3. ✅ Linkear entre servicios y dominios
4. ✅ Mantener `02-DOMINIOS/` como eje central

---

## 📖 Ver también

- [[00-INICIO]] - Página de inicio
- [[06-GOVERNANCE/Git-Convenciones]] - Cómo hacer commits
- [[02-DOMINIOS/laboratorio/Columna OUT - Estado del Embrión]] - Ejemplo de trace
