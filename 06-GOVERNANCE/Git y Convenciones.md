# Git y Convenciones de Código

## 📝 Convenciones de Commits

Seguimos **Conventional Commits** format:

```
type(scope): description

[optional body]
[optional footer]
```

### Tipos válidos

| Tipo | Uso |
|------|-----|
| `feat` | Nueva funcionalidad |
| `fix` | Corrección de bug |
| `docs` | Cambios en documentación |
| `style` | Formato, linting (no afecta lógica) |
| `refactor` | Refactorización sin cambiar comportamiento |
| `perf` | Mejoras de performance |
| `test` | Agregando/actualizando tests |
| `chore` | Tareas de mantenimiento, deps |

### Ejemplos válidos

```bash
# BUENOS
feat(laboratory): add embryo viability scoring
fix(auth): correct JWT token expiration
chore(deps): update frontend submodule
docs(api): add endpoint documentation

# MALOS (evitar)
feat: agregar cosas
Update stuff
WIP
Fixed the thing
```

### Reglas

- ❌ NO agregar "Generated with Claude Code"
- ❌ NO agregar "Co-Authored-By" footer
- ✅ Mantener commits CORTOS
- ✅ Una línea para cambios simples
- ✅ Body solo para: breaking changes, múltiples cambios relacionados
- ✅ Description específica (qué se hizo, no generic)

## 🌳 Git Flow

```
main (producción)
  ↑
staging (pre-producción)
  ↑
feature/* (desarrollo)
```

### Flujo típico

```bash
# 1. Crear feature branch
git checkout -b feature/add-embryo-scoring

# 2. Realizar cambios y commits
git commit -m "feat(lab): add viability scoring logic"

# 3. Push y crear PR
git push -u origin feature/add-embryo-scoring
# Crear PR a staging

# 4. Merge a staging después de review
# Crear PR a main para production

# 5. Delete branch
git branch -d feature/add-embryo-scoring
```

## 📋 Branch naming

```
feature/[descripcion-corta]      # Nueva funcionalidad
fix/[bug-name]                   # Corrección de bug
docs/[capitulo]                  # Documentación
chore/[tarea]                    # Mantenimiento
refactor/[area]                  # Refactorización
```

## 🔄 Submodules

El repositorio usa git submodules para servicios y apps:

```bash
# Clonar con submodules
git clone --recursive [repo-url]

# Actualizar submodules existentes
git submodule update --init --recursive

# Actualizar un submodule específico
cd services/EMR.Api-Gateway.Service
git pull origin main
cd ../..
git commit -am "chore(deps): update API Gateway submodule"
```

## 💾 Commits importantes

```bash
# Ver historial formateado
git log --oneline -n 20

# Ver cambios en un commit
git show <commit-hash>

# Diff entre branches
git diff main..feature/my-feature
```

## 📍 Ver también

- `docs/governance/` - Estructura de governance
- `CLAUDE.md` - Instrucciones completas del proyecto
