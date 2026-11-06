# 🗄️ Comandos de Base de Datos (PostgreSQL)

> Referencia de comandos para consola interactiva (psql), respaldos planos y binarios para Disaster Recovery, sentencias SQL de administración y mantenimiento de ENUMs.

---

## 📑 Índice

- [1. Conexión a la Consola Interactiva (psql)](#1-conexión-a-la-consola-interactiva-psql)
- [2. Meta-Comandos Esenciales dentro de psql](#2-meta-comandos-esenciales-dentro-de-psql)
- [3. Respaldos y Restauración Planos (.sql)](#3-respaldos-y-restauración-planos-sql)
- [4. Respaldos Binarios y Disaster Recovery (.dump)](#4-respaldos-binarios-y-disaster-recovery-dump)
- [5. Mantenimiento Avanzado: Alterar ENUM Nativo sin Pérdida](#5-mantenimiento-avanzado-alterar-enum-nativo-sin-pérdida)
- [6. Flujos de Trabajo Agrupados](#6-flujos-de-trabajo-agrupados)

---

## 1. Conexión a la Consola Interactiva (psql)

```bash
# Conectarse a psql dentro del contenedor Podman
podman exec -it dev_pg psql -U dev_user -d nombre_base_datos

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

## 3. Respaldos y Restauración Planos (.sql)

```bash
# Exportar base de datos a archivo .sql plano desde Podman (incluye DROP previo)
podman exec -i dev_pg pg_dump -U dev_user --clean -d gol_comercial_db > backup_dev.sql

# Restaurar base de datos desde archivo .sql plano hacia Podman
podman exec -i dev_pg psql -U dev_user -d gol_comercial_db < backup_dev.sql

# Exportar base de datos en servidor nativo
pg_dump -U usuario -h localhost -d mi_base_datos > backup.sql

# Restaurar base de datos en servidor nativo
psql -U usuario -h localhost -d mi_base_datos < backup.sql
```

---

## 4. Respaldos Binarios y Disaster Recovery (.dump)

```bash
# Crear directorio de respaldos en el servidor
mkdir -p ~/backups/postgresql

# Exportar copia de seguridad binaria comprimida (-Fc) para Disaster Recovery en producción
podman exec -i gol_comercial_prod_db pg_dump -U prod_user -Fc -d gol_comercial_db > ~/backups/postgresql/backup-$(date +%F-%H%M%S).dump

# Restaurar respaldo binario limpiando objetos existentes (--clean --if-exists --no-owner)
podman exec -i gol_comercial_prod_db pg_restore -U prod_user -d gol_comercial_db --clean --if-exists --no-owner < ~/backups/postgresql/backup_a_restaurar.dump

# Limpiar archivos temporales dentro del contenedor
podman exec dev_pg sh -c "rm -f /tmp/*.sql /tmp/*.dump"
```

---

## 5. Mantenimiento Avanzado: Alterar ENUM Nativo sin Pérdida

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

## 6. Flujos de Trabajo Agrupados

```bash
# ==============================================================================
# Flujo: Creación inicial completa de usuario y base de datos en PostgreSQL
# ==============================================================================

# 1. Crear usuario con contraseña
sudo -u postgres psql -c "CREATE USER kyoz_app WITH PASSWORD 'ClaveSegura2026!';"

# 2. Crear base de datos asignando dueño
sudo -u postgres psql -c "CREATE DATABASE kyoz_db OWNER kyoz_app;"

# 3. Asignar todos los privilegios sobre la base de datos
sudo -u postgres psql -c "GRANT ALL PRIVILEGES ON DATABASE kyoz_db TO kyoz_app;"

# 4. Asignar privilegios sobre el esquema public (PostgreSQL 15+)
sudo -u postgres psql -d kyoz_db -c "GRANT ALL ON SCHEMA public TO kyoz_app;"
```

---

[← Volver al README](README.md)
