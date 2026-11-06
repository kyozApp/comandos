# ⚡ Comandos de Prisma 8 CLI y Data Contracts

> Referencia de comandos CLI para la instalación con PNPM, inicialización interactiva, contratos de datos en TypeScript, migraciones y panel visual con Prisma 8 (100% Open Source y gratuito).

---

## 📑 Índice

- [1. Instalación y Configuración Inicial](#1-instalación-y-configuración-inicial)
- [2. Inicialización de la Base de Datos (Primera vez)](#2-inicialización-de-la-base-de-datos-primera-vez)
- [3. Modificaciones y Migraciones en Desarrollo](#3-modificaciones-y-migraciones-en-desarrollo)
- [4. Despliegue en Producción e Introspección](#4-despliegue-en-producción-e-introspección)
- [5. Panel Visual y Semillas (Seed)](#5-panel-visual-y-semillas-seed)

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

## 2. Inicialización de la Base de Datos (Primera vez)

```bash
# 1. Compilar src/prisma/contract.ts y generar los artefactos tipados (contract.json y .d.ts)
pnpm prisma contract emit

# 2. Crear las tablas base en PostgreSQL por primera vez de forma aditiva y firmar la BD
pnpm prisma db init

# 3. Comprobar que la base de datos física cumpla exactamente con el contrato emitido
pnpm prisma db verify
```

---

## 3. Modificaciones y Migraciones en Desarrollo

```bash
# 1. Recompilar los contratos tras modificar src/prisma/contract.ts
pnpm prisma contract emit

# 2. Crear y aplicar la migración con nombre descriptivo en desarrollo
pnpm prisma migrate dev --name agregar_usuarios

# 3. Conciliar y sincronizar los cambios en la base de datos
pnpm prisma db update

# 4. Verificar que no exista ninguna desalineación entre el contrato y la BD
pnpm prisma db verify
```

---

## 4. Despliegue en Producción e Introspección

```bash
# Aplicar todas las migraciones pendientes en el servidor de producción
pnpm prisma migrate deploy

# Consultar el estado de las migraciones ejecutadas y pendientes
pnpm prisma migrate status

# Resetear la base de datos completa y reaplicar todas las migraciones (Desarrollo)
pnpm prisma migrate reset --force

# Descargar la estructura de una base de datos existente hacia el esquema local (Introspección)
pnpm prisma db pull
```

---

## 5. Panel Visual y Semillas (Seed)

```bash
# Abrir el panel interactivo visual de administración de datos en el navegador (Prisma Studio)
pnpm prisma studio

# Ejecutar el script de siembra (seed) de datos con tsx
pnpm tsx src/prisma/seed.ts
```

---

[← Volver al README](README.md)
