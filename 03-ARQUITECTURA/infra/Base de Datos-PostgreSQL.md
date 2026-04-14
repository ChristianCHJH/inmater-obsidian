# Base de datos - PostgreSQL

## 🗄️ Información General

- **Motor:** PostgreSQL
- **ORM:** Sequelize (para backend nuevo)
- **Legacy:** PDO (para apps/web PHP)
- **Schemas:** Múltiples schemas por módulo

## 📊 Schemas principales

```sql
appinmater_modulo         -- Datos de aplicación
appinmater_log            -- Logs de auditoría
```

## 🔗 Configuración de conexión

### Sequelize (Backend TypeScript)

```javascript
const db = new Sequelize({
  dialect: 'postgres',
  host: process.env.DB_HOST,
  port: process.env.DB_PORT,
  username: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
  schema: process.env.DB_SCHEMA_APP_MODULO
});
```

### PDO (PHP Legacy)

```php
$db = new PDO(
  "pgsql:host={$host};port={$port};dbname={$db}",
  $user,
  $pass
);
```

## 📋 Tablas principales

### Laboratorio
- `hc_aspirado` - Evaluación de óvulos/embriones
- `lab_aspira` - Procedimiento de aspiración
- `lab_cultivo` - Cultivo de embriones
- `hc_reprod_info` - Información de reproducción

**Ver:** [[Columna OUT - Estado del Embrión]]

### Pacientes
- `hc_paciente` - Datos de pacientes
- `hc_paciente_male` - Datos de parejas
- `hc_paciente_contact` - Contacto

### Autenticación
- `login` - Usuarios
- `login_per` - Permisos
- `hc_sede` - Sedes

### Calendarios
- `agendamientos` - Turnos agendados
- `turnos` - Definición de turnos
- `salas` - Salas de procedimiento

## ⚙️ Variables de entorno

```env
DB_HOST=localhost
DB_PORT=5432
DB_USER=inmater_user
DB_PASSWORD=contraseña
DB_NAME=inmater_db
DB_SCHEMA_APP_MODULO=appinmater_modulo
```

## 📍 Catálogos

- [[Base de Datos/Catálogo de tablas]] - Descripción de todas las tablas
- [[Base de Datos/Catálogo de vistas]] - Vistas SQL

## 📍 Ver también

- [[Arquitectura/Arquitectura-Stack-Tecnologico]]
- [[Laboratorio/Laboratorio-Procesos]]
