# 🐳 Manual Maestro de Docker y Contenedores

> **Guía de referencia profesional para la administración de entornos de software.**  
> Este manual consolida la infraestructura dockerizada de desarrollo rápido de bases de datos y la orquestación avanzada de entornos de **Desarrollo (Dev)** y **Producción (Prod)** para aplicaciones modernas.

---

## 📑 Índice de Contenidos

1. [📂 Parte 1: Contenedores Genéricos de Base de Datos](#1-contenedores-genéricos-de-base-de-datos)
   - [MySQL General / Personal](#mysql-general--personal)
   - [PostgreSQL Neuromedic](#postgresql-neuromedic)
   - [SQLite Server (MCP)](#sqlite-server-mcp)
2. [💻 Parte 2: Entorno de Desarrollo Local (Docker Dev)](#2-entorno-de-desarrollo-local-docker-dev)
   - [Estructura del docker-compose.yml de Dev](#estructura-del-docker-composeyml-de-dev)
   - [Configuración de Variables de Entorno (`.env`)](#configuración-de-variables-de-entorno-env)
   - [Script SQL de Inicialización (`01-init-database.sql`)](#script-sql-de-inicialización-01-init-database-sql)
   - [Comandos de Control Local](#comandos-de-control-local)
3. [🌐 Parte 3: Entorno de Producción VPS (Docker Prod)](#3-entorno-de-producción-vps-docker-prod)
   - [Dockerfile Multi-Stage Optimizado para SvelteKit](#dockerfile-multi-stage-optimizado-para-sveltekit)
   - [Estructura del docker-compose.yml de Producción](#estructura-del-docker-composeyml-de-producción)
   - [Configuración de Producción (`.env`)](#configuración-de-producción-env)
   - [Ciclo de Control y Monitoreo de Logs en Servidor](#ciclo-de-control-y-monitoreo-de-logs-en-servidor)

---

## 📂 1. Contenedores Genéricos de Base de Datos

Estas plantillas permiten levantar motores de base de datos aislados y persistentes en tu máquina local de forma ágil mediante Docker Compose.

### MySQL General / Personal

Disponemos de dos configuraciones según el puerto requerido para evitar colisiones:

#### 1. MySQL General (Puerto `3306`)
```yaml
# docker-compose.yml
services:
  db:
    container_name: mysql-general
    image: mysql:latest
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: prueba123
      MYSQL_USER: prueba
      MYSQL_PASSWORD: prueba
      MYSQL_DATABASE: prueba
    ports:
      - "3306:3306"
    volumes:
      - mysql_general_data:/var/lib/mysql

volumes:
  mysql_general_data:
```

#### 2. MySQL Personal (Puerto `3307`)
```yaml
# docker-compose.yml
services:
  db:
    container_name: mysql-personal
    image: mysql:latest
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: prueba123
      MYSQL_USER: prueba
      MYSQL_PASSWORD: prueba
      MYSQL_DATABASE: prueba
    ports:
      - "3307:3306"
    volumes:
      - mysql_personal_data:/var/lib/mysql

volumes:
  mysql_personal_data:
```

---

### PostgreSQL Neuromedic

Contenedor de PostgreSQL persistente optimizado para la base de datos de Neuromedic:

```yaml
# docker-compose.yml
services:
  db:
    container_name: postgresql-neuromedic
    image: postgres:latest
    restart: always
    environment:
      POSTGRES_USER: neuromedic
      POSTGRES_PASSWORD: neuromedic
      POSTGRES_DB: neuromedicdb
    ports:
      - "5432:5432"
    volumes:
      - postgresql_neuromedic_data:/var/lib/postgresql/data

volumes:
  postgresql_neuromedic_data:
```

---

### SQLite Server (MCP)

Levantar un motor de SQLite expuesto como servicio utilizando volúmenes de desarrollo local:

```yaml
# docker-compose.yml
services:
  sqlite:
    container_name: sqlite-server
    image: mcp/sqlite
    volumes:
      - ./sqlite_data:/mcp
    command: ["--db-path", "/mcp/db.sqlite"]
```

> [!TIP]
> **Ejecución y Consultas Rápidas en SQLite:**
> * Levantar el servicio: `docker compose up -d`
> * Ejecutar una consulta rápida desde consola:
>   ```bash
>   docker exec -i sqlite-server write_query --query "CREATE TABLE usuarios (id INTEGER PRIMARY KEY, nombre TEXT, email TEXT);"
>   ```

---

## 💻 2. Entorno de Desarrollo Local (Docker Dev)

Esta infraestructura levanta la pila completa de servicios de desarrollo local para aplicaciones modernas de forma automatizada y con volúmenes persistentes.

### Estructura del docker-compose.yml de Dev

Ubicado en `docker/dev/docker-compose.yml` en el proyecto:

```yaml
name: template-dev

services:
  db:
    image: mariadb:12.2.2-noble
    container_name: template-dev-db
    restart: always
    environment:
      MARIADB_DATABASE: ${MARIADB_DATABASE:?MARIADB_DATABASE is required}
      MARIADB_USER: ${MARIADB_USER:?MARIADB_USER is required}
      MARIADB_PASSWORD: ${MARIADB_PASSWORD:?MARIADB_PASSWORD is required}
      MARIADB_ROOT_PASSWORD: ${MARIADB_ROOT_PASSWORD:?MARIADB_ROOT_PASSWORD is required}
    ports:
      - '${MARIADB_PORT:?MARIADB_PORT is required}:3306'
    volumes:
      - template_db_data_dev:/var/lib/mysql
      - ./init-db:/docker-entrypoint-initdb.d:ro
    networks:
      - template_net
    healthcheck:
      test: ['CMD', 'healthcheck.sh', '--connect', '--innodb_initialized']
      interval: 10s
      timeout: 5s
      retries: 5

  mailpit:
    image: axllent/mailpit
    container_name: template-dev-mailpit
    restart: always
    ports:
      - '${MAILPIT_PORT_WEB:?MAILPIT_PORT_WEB is required}:8025'
      - '${MAILPIT_PORT_SMTP:?MAILPIT_PORT_SMTP is required}:1025'
    networks:
      - template_net

  valkey:
    image: valkey/valkey
    container_name: template-dev-valkey
    restart: always
    ports:
      - '${VALKEY_PORT:?VALKEY_PORT is required}:6379'
    networks:
      - template_net

volumes:
  template_db_data_dev:

networks:
  template_net:
    driver: bridge
```

---

### Configuración de Variables de Entorno (`.env`)

Ubicado en `docker/dev/.env`. Configura los puertos de desarrollo locales para evitar colisiones:

```env
# ==============================================================================
# DATABASE (MariaDB / MySQL Dev)
# ==============================================================================
MARIADB_PORT=3309
MARIADB_DATABASE=template_db
MARIADB_USER=template_user
MARIADB_PASSWORD=template_pass
MARIADB_ROOT_PASSWORD=template_root_pass

# ==============================================================================
# MAILPIT (SMTP local testing)
# ==============================================================================
MAILPIT_PORT_WEB=8104
MAILPIT_PORT_SMTP=1024

# ==============================================================================
# VALKEY (Cache)
# ==============================================================================
VALKEY_PORT=6379
```

---

### Script SQL de Inicialización (`01-init-database.sql`)

Ubicado en `docker/dev/init-db/01-init-database.sql`. MariaDB lo ejecuta en el primer arranque para asegurar Unicode y permisos adecuados:

```sql
-- Asegurar soporte completo de Unicode (Emojis y caracteres complejos)
ALTER DATABASE `template_db`
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_unicode_ci;

-- Otorgar privilegios exclusivos sobre la base de datos
GRANT ALL PRIVILEGES ON `template_db`.* TO 'template_user'@'%';

-- Aplicar cambios
FLUSH PRIVILEGES;
```

---

### Comandos de Control Local

*   **Levantar en segundo plano**: `docker compose -f docker/dev/docker-compose.yml up -d`
*   **Detener contenedores**: `docker compose -f docker/dev/docker-compose.yml down`
*   **Ver logs en tiempo real**: `docker compose -f docker/dev/docker-compose.yml logs -f`
*   **Detener y eliminar volúmenes (Limpieza profunda)**: `docker compose -f docker/dev/docker-compose.yml down -v`

---

## 🌐 3. Entorno de Producción VPS (Docker Prod)

Esta sección describe el flujo de despliegue atómico en producción, utilizando un pipeline de compilación multi-etapa y Compose para aislar los servicios.

### Dockerfile Multi-Stage Optimizado para SvelteKit

Ubicado en la raíz del proyecto SvelteKit (`./Dockerfile`). Permite compilar la aplicación, desechar las dependencias de desarrollo pesadas y construir una imagen ultra-ligera y segura para producción:

```dockerfile
# ==========================================
# ETEPA 1: Compilación de la Aplicación
# ==========================================
FROM node:24-alpine AS builder

WORKDIR /app

# Habilitar soporte nativo de pnpm
RUN corepack enable && corepack prepare pnpm@11.2.1 --activate

# Copiar archivos de empaquetado para instalar dependencias de desarrollo
COPY pnpm-lock.yaml package.json ./

# Instalar dependencias completas de desarrollo
RUN pnpm install --frozen-lockfile

# Copiar el código fuente del proyecto
COPY . .

# Compilar SvelteKit (Genera la carpeta ./build usando adapter-node)
RUN pnpm build

# ==========================================
# ETAPA 2: Entorno de Ejecución Producción
# ==========================================
FROM node:24-alpine AS runner

WORKDIR /app

# Habilitar soporte nativo de pnpm
RUN corepack enable && corepack prepare pnpm@11.2.1 --activate

# Copiar archivos de empaquetado
COPY pnpm-lock.yaml package.json ./

# Instalar únicamente dependencias requeridas en ejecución (--prod)
RUN pnpm install --prod --frozen-lockfile

# Copiar la aplicación compilada del constructor (builder)
COPY --from=builder /app/build ./build

# Exponer el puerto de Node y definir variables de entorno
EXPOSE 8110
ENV NODE_ENV=production
ENV PORT=8110

# Comando de ejecución seguro
CMD ["node", "build/index.js"]
```

---

### Estructura del docker-compose.yml de Producción

Ubicado en `docker/prod/docker-compose.yml`. Orquesta la aplicación web y la base de datos de producción:

```yaml
name: template-prod

services:
  # Init-container para corregir permisos de volúmenes en Linux VPS
  init-permissions:
    image: alpine:3.23
    container_name: template-prod-init-perms
    user: root
    volumes:
      - ../../logs:/app/logs
      - ../../documents:/app/documents
    command: ['sh', '-c', 'chown -R 1000:1000 /app/logs /app/documents']

  api:
    build:
      context: ../..
      dockerfile: Dockerfile
    container_name: template-prod-api
    restart: always
    init: true
    env_file:
      - .env
    ports:
      - '${SERVER_PORT:?SERVER_PORT is required}:8110'
    volumes:
      - ../../logs:/app/logs
      - ../../documents:/app/documents
    networks:
      - template_net
    depends_on:
      init-permissions:
        condition: service_completed_successfully
      db:
        condition: service_healthy

  db:
    image: mariadb:12.2.2-noble
    container_name: template-prod-db
    restart: always
    environment:
      MARIADB_DATABASE: ${MARIADB_DATABASE:?MARIADB_DATABASE is required}
      MARIADB_USER: ${MARIADB_USER:?MARIADB_USER is required}
      MARIADB_PASSWORD: ${MARIADB_PASSWORD:?MARIADB_PASSWORD is required}
      MARIADB_ROOT_PASSWORD: ${MARIADB_ROOT_PASSWORD:?MARIADB_ROOT_PASSWORD is required}
    volumes:
      - template_db_data_prod:/var/lib/mysql
    networks:
      - template_net
    healthcheck:
      test:
        [
          'CMD-SHELL',
          'mariadb-admin ping -h localhost -u root -p$${MARIADB_ROOT_PASSWORD} || exit 1'
        ]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s

volumes:
  template_db_data_prod:

networks:
  template_net:
    driver: bridge
```

---

### Configuración de Producción (`.env`)

Ubicado en `docker/prod/.env`. Contiene los mapeos de puertos de producción (`8410` asignado para esta plantilla) y credenciales seguras de servidor:

```env
SERVER_PORT=8410
DATABASE_URL=mysql://template_db_prod:credenciales_seguras@vps-internal-ip:3306/template_db
JWT_SECRET=tu_firma_segura_para_produccion
```

---

### Ciclo de Control y Monitoreo de Logs en Servidor

Utiliza estos comandos desde la carpeta `docker/prod/` en el VPS para gestionar tu aplicación:

*   **Compilar y levantar el stack completo**:
    ```bash
    docker compose up -d --build
    ```
*   **Monitorear logs en tiempo real**:
    ```bash
    # logs del contenedor web principal
    docker logs -f template-prod-api
    
    # logs del contenedor de base de datos
    docker logs -f template-prod-db
    ```
*   **Detener la infraestructura**:
    ```bash
    docker compose down
    ```

> [!WARNING]
> **Seguridad en Producción:**
> Nunca dejes las contraseñas por defecto (`template_pass`, `prueba123`) en tu archivo `docker/prod/.env`. Utiliza claves robustas y deshabilita la exposición pública de puertos de base de datos en tu Firewall local.
