# ⚡ Comandos de Prisma 8 CLI y Data Contracts

> Referencia de comandos CLI para la instalación con PNPM, inicialización interactiva, contratos de datos en TypeScript, flujo profesional de migraciones en entornos de Desarrollo y Producción (`migration plan` + `db migrate`), y administración visual con Prisma 8.

---

## 📑 Índice

- [1. Instalación y Configuración Inicial](#1-instalación-y-configuración-inicial)
- [2. Entorno de Desarrollo (Dev)](#2-entorno-de-desarrollo-dev)
  - [2.1. Primera vez (Setup Inicial y Migración Base)](#21-primera-vez-setup-inicial-y-migración-base)
  - [2.2. Actualizar (Flujo Diario de Modificaciones)](#22-actualizar-flujo-diario-de-modificaciones)
- [3. Entorno de Producción (Prod)](#3-entorno-de-producción-prod)
  - [3.1. Primera vez (Setup Inicial en Servidor)](#31-primera-vez-setup-inicial-en-servidor)
  - [3.2. Actualizar (Despliegue de Nuevas Migraciones)](#32-actualizar-despliegue-de-nuevas-migraciones)
- [4. Panel Visual y Semillas (Seed)](#4-panel-visual-y-semillas-seed)

---

## 1. Instalación y Configuración Inicial

### Instalación de dependencias

```bash
# Instalar Prisma CLI como dependencia de desarrollo
pnpm add -D prisma
```

### Aprobación de binarios nativos en PNPM

> [!IMPORTANT]
> En versiones modernas de PNPM (v10+, incluido v12), los scripts de compilación de dependencias nativas se bloquean por defecto por seguridad arrojando la advertencia `[ERR_PNPM_IGNORED_BUILDS]`.

```bash
# Autorizar scripts de compilación requeridos por Prisma (msgpackr-extract, workerd)
pnpm approve-builds
```

### Inicialización del ORM con Target PostgreSQL

> [!NOTE]
> Durante el asistente interactivo de la CLI, selecciona estas respuestas:
>
> - **How do you want to write your schema?** → `TypeScript (.ts)`
> - **Where should the schema file go?** → `src/prisma/contract.ts`
> - **Also write a .env file from .env.example?** → `No`

```bash
# Inicializar el contrato de Prisma 8 especificando PostgreSQL como motor
pnpm prisma orm init --target postgres
```

> **Estructura generada automáticamente:**
>
> - `src/prisma/contract.ts` (esquema del contrato tipado)
> - `prisma.config.ts` (configuración del runtime)
> - `src/prisma/db.ts` (cliente de base de datos tipado)

---

## 2. Entorno de Desarrollo (Dev)

### 2.1. Primera vez (Setup Inicial y Migración Base)

Para generar la primera migración oficial (`init`) desde un estado limpio y dejar la base de datos local lista y verificada:

```bash
# 1. Compilar src/prisma/contract.ts y generar los artefactos tipados (contract.json y .d.ts)
pnpm prisma contract emit

# 2. Crear el paquete de migración inicial desde un estado vacío (@empty)
pnpm prisma migration plan --from @empty --name "init"

# 3. Aplicar la migración a la base de datos local
pnpm prisma db migrate

# 4. Verificar que la base de datos física cumpla exactamente con el contrato emitido
pnpm prisma db verify

# 5. Comprobar que el historial de migraciones esté al día y sin pendientes
pnpm prisma migration status
```

> Esto creará dentro de `migrations/app/` el paquete auditable con todas las instrucciones `CREATE TABLE`.

---

### 2.2. Actualizar (Flujo Diario de Modificaciones)

Cada vez que agregues tablas, relaciones o campos en `src/prisma/contract.ts`:

```bash
# 1. Recompilar el contrato tras editar el esquema
pnpm prisma contract emit

# 2. Planificar la migración generando el paquete en disco
pnpm prisma migration plan --name "agregar_campo_x"

# Si renombraste un campo o modelo, usa --rename para evitar pérdida de datos:
# pnpm prisma migration plan --rename Modelo.campo_viejo:Modelo.campo_nuevo --name "renombrar_campo"

# 3. Aplicar la migración a tu base de datos local
pnpm prisma db migrate

# 4. Comprobar que no exista ninguna desalineación
pnpm prisma db verify

# 5. Comprobar que el historial de migraciones esté al día y sin pendientes
pnpm prisma migration status
```

---

## 3. Entorno de Producción (Prod)

### 3.1. Primera vez (Setup Inicial en Servidor)

#### Opción A: Base de datos en Producción limpia / vacía

Si estás levantando la base de datos de producción desde cero:

```bash
# 1. Ejecutar todo el grafo de migraciones en orden determinista
pnpm prisma db migrate

# 2. Verificar que la estructura física coincida con el contrato
pnpm prisma db verify

# 3. Comprobar que el historial de migraciones esté al día
pnpm prisma migration status
```

#### Opción B: Base de datos en Producción que ya tiene las tablas creadas

Si la base de datos ya contiene las tablas correspondientes al contrato actual:

```bash
# 1. Firmar la base de datos con el hash del contrato actual (sin reaplicar migraciones)
pnpm prisma db sign

# 2. Verificar que todo coincida al 100% sin discrepancias
pnpm prisma db verify

# 3. Consultar el estado del historial de migraciones
pnpm prisma migration status
```

---

### 3.2. Actualizar (Despliegue de Nuevas Migraciones)

Una vez probados los cambios en desarrollo y tras realizar commit/push de la carpeta `migrations/`:

```bash
# 1. Aplicar las migraciones pendientes en el servidor de producción
pnpm prisma db migrate

# 2. Verificar que la base de datos física cumpla exactamente con el contrato emitido
pnpm prisma db verify

# 3. Consultar el estado y orden de las migraciones ejecutadas
pnpm prisma migration status
```

---

## 4. Panel Visual y Semillas (Seed)

```bash
# Abrir el panel interactivo visual de administración de datos en el navegador (Prisma Studio)
pnpm prisma studio

# Ejecutar el script de siembra (seed) de datos con tsx
pnpm tsx src/prisma/seed.ts
```

---

[← Volver al README](README.md)
