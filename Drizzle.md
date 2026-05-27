# 🐘 Guía de Persistencia Relacional con Drizzle ORM

> **Estándar de desarrollo para el modelado y consumo de bases de datos relacionales.**  
> Este manual documenta la persistencia de datos relacionales estructurada mediante **Drizzle ORM**, detallando la arquitectura de conexiones, esquemas declarativos en TypeScript, carga de relaciones automáticas y directivas de migración para producción.

---

## 📑 Índice de Contenidos

1. [📂 Parte 1: Arquitectura y Pool de Conexiones](#1-arquitectura-y-pool-de-conexiones)
2. [🗄️ Parte 2: Modelos y Esquemas de Base de Datos](#2-modelos-y-esquemas-de-base-de-datos)
3. [🧩 Parte 3: Modelado de Relaciones Avanzadas (1 a Muchos)](#3-modelado-de-relaciones-avanzadas-1-a-muchos)
4. [🔍 Parte 4: Consumo de Datos (Relational API vs. Query Builder)](#4-consumo-de-datos-relational-api-vs-query-builder)
5. [🔄 Parte 5: Filosofía de Sincronización (db:push vs. db:migrate)](#5-filosofía-de-sincronización-dbpush-vs-dbmigrate)

---

## 📂 1. Arquitectura y Pool de Conexiones

SvelteKit cuenta con una división estricta entre el cliente (frontend) y el servidor (backend). Para resguardar la seguridad y las credenciales de base de datos, toda interacción física con bases de datos **debe ubicarse obligatoriamente del lado del servidor** (`.server`).

### Conexión e Instanciación (`src/lib/server/db/index.ts`)

Establece la conexión a MariaDB/MySQL utilizando un pool de conexiones optimizado de `mysql2`:

```typescript
import { env } from '$env/dynamic/private';
import { drizzle } from 'drizzle-orm/mysql2';
import mysql from 'mysql2/promise';

import { users } from './schemas/user';

if (!env.DATABASE_URL) {
	throw new Error('La variable de entorno DATABASE_URL no está configurada');
}

// Inicializar el Pool de Conexiones físico
const client = mysql.createPool({
	uri: env.DATABASE_URL,
	timezone: 'Z' // Garantizar consistencia horaria UTC
});

// Instancia global cliente db
export const db = drizzle(client, { schema: { users }, mode: 'default' });
```

> [!WARNING]
> **Aislamiento del Servidor:**
> Nunca importes ningún módulo o archivo ubicado en `src/lib/server/` dentro de archivos cargados por el cliente (como componentes de interfaz sin extensión `.server`). SvelteKit bloqueará la compilación en producción por medidas esenciales de seguridad.

---

## 🗄️ 2. Modelos y Esquemas de Base de Datos

Definimos los modelos de forma modular. Cada entidad del sistema se define en un archivo TypeScript independiente dentro de la carpeta `src/lib/server/db/schemas/`.

### Definición del Esquema Inicial (`src/lib/server/db/schemas/user.ts`)

```typescript
import { int, mysqlTable, varchar } from 'drizzle-orm/mysql-core';

// Tabla Principal de Usuarios
export const users = mysqlTable('users', {
	id: int().primaryKey().autoincrement(),
	name: varchar({ length: 255 }).notNull(),
	email: varchar({ length: 255 }).notNull().unique(),
	password: varchar({ length: 255 }).notNull()
});
```

### Configuración del Drizzle Kit (`drizzle.config.ts`)

El archivo de configuración lee los esquemas dinámicamente utilizando patrones glob:

```typescript
import { defineConfig } from 'drizzle-kit';

if (!process.env.DATABASE_URL) {
	throw new Error('La variable de entorno DATABASE_URL no está configurada');
}

export default defineConfig({
	schema: './src/lib/server/db/schemas/*.ts',
	dialect: 'mysql',
	dbCredentials: {
		url: process.env.DATABASE_URL
	},
	verbose: true,
	strict: true
});
```

---

## 🧩 3. Modelado de Relaciones Avanzadas (1 a Muchos)

Para modelar relaciones complejas (un usuario puede tener múltiples dispositivos activos de sesión), combinamos la llave foránea física (`references`) con la definición conceptual de relaciones (`relations`):

```typescript
import { relations } from 'drizzle-orm';
import { char, mysqlTable, timestamp, varchar } from 'drizzle-orm/mysql-core';

// 1. Tabla de Usuarios
export const users = mysqlTable('users', {
	id: char({ length: 36 }).primaryKey(),
	name: varchar({ length: 255 }).notNull(),
	email: varchar({ length: 255 }).notNull().unique(),
	password: varchar({ length: 255 }).notNull()
});

// 2. Tabla de Dispositivos (Llave Foránea Física)
export const userDevices = mysqlTable('user_devices', {
	id: char({ length: 36 }).primaryKey(),
	deviceId: varchar('device_id', { length: 255 }).notNull(),
	userId: char('user_id', { length: 36 })
		.notNull()
		.references(() => users.id, { onDelete: 'cascade' }),
	createdAt: timestamp('created_at').defaultNow().notNull()
});

// ==============================================================================
// 3. DEFINICIÓN CONCEPTUAL DE RELACIONES (Drizzle Relational API)
// ==============================================================================

// Relación desde el lado del Usuario (Un usuario tiene muchos dispositivos)
export const usersRelations = relations(users, ({ many }) => ({
	devices: many(userDevices)
}));

// Relación desde el lado del Dispositivo (Un dispositivo pertenece a un usuario)
export const userDevicesRelations = relations(userDevices, ({ one }) => ({
	user: one(users, {
		fields: [userDevices.userId],
		references: [users.id]
	})
}));
```

---

## 🔍 4. Consumo de Datos (Relational API vs. Query Builder)

Drizzle ORM provee dos formas de operar con base de datos, cada una optimizada para flujos específicos:

### A. Lecturas Fluidas (`db.query`) - Relational Query API
Es la herramienta recomendada para consultar datos porque automatiza los JOINs y retorna objetos anidados limpios, completamente tipados y filtrados.

#### Ejemplo: Obtener Perfil de Usuario con todos sus Dispositivos
```typescript
import { eq } from 'drizzle-orm';
import { db } from '$lib/server/db';
import { users } from '$lib/server/db/schemas/user';

export async function getUserProfile(userId: string) {
	return await db.query.users.findFirst({
		where: eq(users.id, userId),
		with: {
			devices: {
				columns: {
					deviceId: true,
					createdAt: true
				},
				orderBy: (devices, { desc }) => [desc(devices.createdAt)]
			}
		}
	});
}
```

---

### B. Escrituras Seguras (`db.insert`, `db.update`, `db.delete`) - Traditional Query Builder
Las mutaciones de base de datos **no soportan la API Relacional**. Deben realizarse estrictamente mediante el Query Builder tradicional operando de forma directa sobre las tablas físicas.

#### Ejemplo: Insertar un Nuevo Registro
```typescript
import { db } from '$lib/server/db';
import { users } from '$lib/server/db/schemas/user';

export async function createUser(data: typeof users.$inferInsert) {
	await db.insert(users).values({
		name: data.name,
		email: data.email,
		password: data.password
	});
}
```

---

## 🔄 5. Filosofía de Sincronización (db:push vs. db:migrate)

Drizzle ORM provee dos metodologías para sincronizar los esquemas declarados en TypeScript con la base de datos física:

| Criterio | Sincronización Directa (`pnpm db:push`) | Migraciones SQL (`pnpm db:generate` + `db:migrate`) |
| :--- | :--- | :--- |
| **Entorno recomendado** | Exclusivo para **Desarrollo Local** | Mandatorio en **Staging y Producción** |
| **Generación de Archivos**| No genera ningún archivo intermedio | Genera archivos `.sql` cronológicos e inmutables en Git |
| **Velocidad** | Instantánea (Ajuste rápido en Docker) | Requiere confirmación y revisión del `.sql` generado |
| **Seguridad de Datos** | Puede causar pérdida si hay alteraciones críticas | Alerta de forma estricta ante cambios destructivos |
| **Automatización** | No apto para pipelines de CI/CD | Integración atómica ideal en despliegues automatizados |

### Flujo de Trabajo Semántico Diario en Producción:
1. Modificas un archivo de esquema TypeScript (ej: `user.ts`).
2. Generas la migración correspondiente:
   ```bash
   pnpm db:generate
   ```
3. Ejecutas y empujas el cambio en producción VPS de forma atómica:
   ```bash
   pnpm db:migrate
   ```
