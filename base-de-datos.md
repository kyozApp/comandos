# 🗄️ Comandos de Base de Datos (PostgreSQL)

> Referencia de comandos para consola interactiva (psql), respaldos planos y binarios para Disaster Recovery, sentencias SQL de administración y mantenimiento de ENUMs.

---

## 📑 Índice

- [1. Conexión a la Consola Interactiva (psql)](#1-conexión-a-la-consola-interactiva-psql)
- [2. Meta-Comandos Esenciales dentro de psql](#2-meta-comandos-esenciales-dentro-de-psql)
- [3. Sentencias SQL de Administración de Usuarios y DB](#3-sentencias-sql-de-administración-de-usuarios-y-db)
- [4. Respaldos y Restauración Planos (.sql)](#4-respaldos-y-restauración-planos-sql)
- [5. Respaldos Binarios y Disaster Recovery (.dump)](#5-respaldos-binarios-y-disaster-recovery-dump)
- [6. Mantenimiento Avanzado: Alterar ENUM Nativo sin Pérdida](#6-mantenimiento-avanzado-alterar-enum-nativo-sin-pérdida)
- [7. Flujos de Trabajo Agrupados](#7-flujos-de-trabajo-agrupados)

---

## 1. Conexión a la Consola Interactiva (psql)

```bash
# Conectarse a psql dentro del contenedor Podman local (ej. dev_pg)
podman exec -it dev_pg psql -U dev_user -d <nombre_bd>

# Conectarse a psql dentro del contenedor Podman en producción (VPS)
podman exec -it <nombre_contenedor_db> psql -U <usuario_db> -d <nombre_bd>

# Conectarse localmente como superusuario postgres en instalación nativa
sudo -u postgres psql
```

---

## 2. Meta-Comandos Esenciales dentro de psql

```sql
-- Listar todas las bases de datos existentes
\l

-- Conectarse a otra base de datos específica
\c nombre_base_datos

-- Listar todas las tablas y vistas del esquema actual
\dt

-- Ver la estructura y tipos de columnas de una tabla específica
\d nombre_tabla

-- Listar todos los usuarios y sus roles asignados
\du

-- Activar el modo expandido para ver registros largos verticalmente
\x

-- Salir de la consola interactiva psql
\q
```

---

## 3. Sentencias SQL de Administración de Usuarios y DB

```sql
-- Crear un nuevo usuario con contraseña cifrada
CREATE USER app_user WITH PASSWORD '<tu_clave_segura>';

-- Crear una nueva base de datos asignando el usuario como dueño
CREATE DATABASE app_db OWNER app_user;

-- Otorgar todos los privilegios sobre la base de datos al usuario
GRANT ALL PRIVILEGES ON DATABASE app_db TO app_user;

-- Otorgar privilegios sobre el esquema public (Requerido en PostgreSQL 15+)
\c app_db
GRANT ALL ON SCHEMA public TO app_user;

-- Cambiar la contraseña de un usuario existente
ALTER USER app_user WITH PASSWORD '<nueva_clave_segura>';
```

---

## 4. Respaldos y Restauración Planos (.sql)

```bash
# Exportar base de datos a archivo .sql plano desde Podman (incluye DROP previo)
podman exec -i dev_pg pg_dump -U dev_user --clean -d <nombre_bd> > backup_dev.sql

# Restaurar base de datos desde archivo .sql plano hacia Podman
podman exec -i dev_pg psql -U dev_user -d <nombre_bd> < backup_dev.sql

# Exportar base de datos en servidor nativo
pg_dump -U <usuario_db> -h localhost -d <nombre_bd> > backup.sql

# Restaurar base de datos en servidor nativo
psql -U <usuario_db> -h localhost -d <nombre_bd> < backup.sql
```

---

## 5. Respaldos Binarios y Disaster Recovery (.dump)

```bash
# Crear directorio de respaldos en el servidor
mkdir -p ~/backups/postgresql

# Exportar copia de seguridad binaria comprimida (-Fc) para Disaster Recovery
podman exec -i <nombre_contenedor_db> pg_dump -U <usuario_db> -Fc -d <nombre_bd> > ~/backups/postgresql/backup-$(date +%F-%H%M%S).dump

# Restaurar respaldo binario limpiando objetos existentes (--clean --if-exists --no-owner)
podman exec -i <nombre_contenedor_db> pg_restore -U <usuario_db> -d <nombre_bd> --clean --if-exists --no-owner < ~/backups/postgresql/backup_a_restaurar.dump

# Limpiar archivos temporales dentro del contenedor
podman exec <nombre_contenedor_db> sh -c "rm -f /tmp/*.sql /tmp/*.dump"
```

---

## 6. Mantenimiento Avanzado: Alterar ENUM Nativo sin Pérdida

```sql
-- ==============================================================================
-- Procedimiento SQL para renombrar o alterar valores de un ENUM sin perder datos
-- ==============================================================================

-- 1. Crear el nuevo enum con la lista actualizada de valores
CREATE TYPE "Role_new" AS ENUM ('SUPERADMIN', 'ADMIN', 'MANAGER', 'USER');

-- 2. Quitar temporalmente el valor por defecto de la columna afectada
ALTER TABLE "User" ALTER COLUMN "role" DROP DEFAULT;

-- 3. Mapear los valores viejos a los nuevos aplicando casting explícito
ALTER TABLE "User"
  ALTER COLUMN "role" TYPE "Role_new"
    USING (
      CASE "role"::text
        WHEN 'VIEWER' THEN 'USER'::"Role_new"
        ELSE "role"::text::"Role_new"
      END
    );

-- 4. Reasignar el valor por defecto en la tabla
ALTER TABLE "User" ALTER COLUMN "role" SET DEFAULT 'USER'::"Role_new";

-- 5. Eliminar el enum antiguo y renombrar el nuevo con el nombre original
DROP TYPE "Role";
ALTER TYPE "Role_new" RENAME TO "Role";
```

---

## 7. Flujos de Trabajo Agrupados

```bash
# ==============================================================================
# Flujo: Creación inicial completa de usuario y base de datos en PostgreSQL
# ==============================================================================

# 1. Crear usuario con contraseña
sudo -u postgres psql -c "CREATE USER app_user WITH PASSWORD '<tu_clave_segura>';"

# 2. Crear base de datos asignando dueño
sudo -u postgres psql -c "CREATE DATABASE app_db OWNER app_user;"

# 3. Asignar todos los privilegios sobre la base de datos
sudo -u postgres psql -c "GRANT ALL PRIVILEGES ON DATABASE app_db TO app_user;"

# 4. Asignar privilegios sobre el esquema public (PostgreSQL 15+)
sudo -u postgres psql -d app_db -c "GRANT ALL ON SCHEMA public TO app_user;"
```

---

[← Volver al README](README.md)
