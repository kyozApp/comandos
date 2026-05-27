# ⚡ Guía del Ecosistema SvelteKit 5 y Componentes de Interfaz

> **Estándar de desarrollo para el andamiaje, calidad de código y diseño visual.**  
> Este manual documenta el cimiento técnico para construir aplicaciones web ultra-escalables con **SvelteKit 5**, detallando la creación estructurada del proyecto, la configuración estricta del espacio de trabajo y la integración estética del sistema de diseño unificado.

---

## 📑 Índice de Contenidos

1. [🛠️ Parte 1: Creación y Andamiaje del Proyecto](#1-creación-y-andamiaje-del-proyecto)
2. [💻 Parte 2: Calidad de Código y Workspace Config](#2-calidad-de-código-y-workspace-config)
3. [🎨 Parte 3: Librería UI, Tipografía Global y Dark Mode](#3-librería-ui-tipografía-global-y-dark-mode)

---

## 🛠️ 1. Creación y Andamiaje del Proyecto

El andamiaje inicial se genera utilizando el asistente interactivo oficial de **Svelte 5** (Día Cero):

```bash
pnpm dlx sv create template-sveltekit-dashboard-drizzle
```

### ⚙️ Parámetros Seleccionados en el Asistente:

*   **Template**: `SvelteKit minimal` (Para evitar plantillas precargadas y construir un sistema de diseño limpio).
*   **TypeScript**: `Yes, using TypeScript syntax` (Tipado estricto obligatorio).
*   **Addons**: `prettier, eslint, vitest, playwright, tailwindcss, sveltekit-adapter, drizzle`.
*   **vitest**: `unit testing` & `component testing`.
*   **tailwindcss**: `plugins: typography, forms` (Para layouts corporativos).
*   **sveltekit-adapter**: `node` (Para despliegues nativos y optimizados en Linux/Docker VPS).
*   **drizzle**: `MySQL` ➡️ `mysql2` ➡️ `No` (Las conexiones se levantan mediante infraestructura Docker).
*   **Package Manager**: `pnpm`.

Una vez creado, accede al directorio del proyecto:

```bash
cd template-sveltekit-dashboard-drizzle
```

### Fijar Versiones del Motor (Volta Pin)
Para asegurar que todo el equipo trabaje exactamente bajo el mismo entorno inmutable y evitar comportamientos extraños, anclamos las versiones de Node y pnpm mediante Volta:

```bash
volta pin node@24
volta pin pnpm@11
```

Esto inyecta las siguientes propiedades en el `package.json`:

```json
"volta": {
  "node": "24.15.0",
  "pnpm": "11.2.1"
}
```

---

## 💻 2. Calidad de Código y Workspace Config

Para asegurar que todo el equipo formatee e importe de manera unificada y transparente en Windows y Linux, configuramos el espacio de trabajo local:

### A. Configuración de VS Code (`.vscode/settings.json`)

```json
{
	"editor.tabSize": 2,
	"editor.insertSpaces": true,
	"editor.detectIndentation": false,
	"editor.formatOnSave": true,
	"svelte.enable-ts-plugin": true,
	"[svelte]": {
		"editor.defaultFormatter": "svelte.svelte-vscode",
		"editor.formatOnSave": true,
		"editor.codeActionsOnSave": {
			"source.fixAll.eslint": "explicit",
			"source.organizeImports": "explicit"
		}
	},
	"[typescript]": {
		"editor.defaultFormatter": "esbenp.prettier-vscode",
		"editor.formatOnSave": true
	},
	"[javascript]": {
		"editor.defaultFormatter": "esbenp.prettier-vscode",
		"editor.formatOnSave": true
	},
	"js/ts.updateImportsOnFileMove.enabled": "always",
	"tailwindCSS.includeLanguages": {
		"svelte": "html"
	},
	"tailwindCSS.emmetCompletions": true,
	"files.associations": {
		"*.css": "tailwindcss"
	},
	"eslint.validate": ["javascript", "typescript", "svelte"],
	"debug.terminal.clearBeforeReusing": true
}
```

---

### B. Extensiones Sugeridas (`.vscode/extensions.json`)

```json
{
	"recommendations": [
		"svelte.svelte-vscode",
		"bradlc.vscode-tailwindcss",
		"dbaeumer.vscode-eslint",
		"esbenp.prettier-vscode"
	]
}
```

---

### C. Personalización de Prettier (`.prettierrc`)

Instala el plugin para el ordenamiento automático y semántico de sentencias import:

```bash
pnpm add -D @trivago/prettier-plugin-sort-imports
```

Edita el archivo `.prettierrc` existente para incorporar el ordenamiento de imports:

```json
{
	"useTabs": true,
	"singleQuote": true,
	"trailingComma": "none",
	"printWidth": 100,
	"plugins": [
		"prettier-plugin-svelte",
		"@trivago/prettier-plugin-sort-imports",
		"prettier-plugin-tailwindcss"
	],
	"importOrder": [
		"<BUILTIN_MODULES>",
		"^@sveltejs/(.*)$",
		"^\\$app/(.*)$",
		"<THIRD_PARTY_MODULES>",
		"^\\$lib/(.*)$",
		"^[./]",
		"^./\\$types$"
	],
	"importOrderSeparation": true,
	"importOrderSortSpecifiers": true,
	"overrides": [
		{
			"files": "*.svelte",
			"options": {
				"parser": "svelte"
			}
		}
	],
	"tailwindStylesheet": "./src/routes/layout.css"
}
```

---

### D. Ajustes Especiales de ESLint (`eslint.config.js`)

Para evitar conflictos de análisis con SvelteKit, agrega las siguientes excepciones en el bloque de reglas globales del archivo `eslint.config.js`:

```javascript
rules: {
  'no-undef': 'off',
  'svelte/no-navigation-without-resolve': 'off'
}
```

---

## 🎨 3. Librería UI, Tipografía Global y Dark Mode

Incorporamos la estética, componentes unificados y el sistema tipográfico de la aplicación.

### A. Instalar Lucide Icons (Svelte 5 nativo)
```bash
pnpm add @lucide/svelte@next
```

### B. Inicializar shadcn-svelte
```bash
pnpm dlx shadcn-svelte@latest init
```
*   **Preset**: `Vega - Lucide / Inter` (Estética corporativa minimalista).
*   **Global CSS**: `src/routes/layout.css`
*   **Import Aliases**: `$lib`, `$lib/components`, `$lib/components/ui`, `$lib/utils`, `$lib/hooks`
*   **Update CSS**: `Yes`

---

### C. Instalar la Tipografía Corporativa (Fontsource)
Para evitar FOUT (Flash of Unstyled Text) y parpadeos tipográficos, auto-aloja la tipografía corporativa localmente mediante Fontsource:

```bash
pnpm add @fontsource-variable/inter
```

Agrega manualmente la siguiente línea de importación en `src/routes/layout.css` para completar la integración tipográfica:

```css
/* Agregar al principio de layout.css */
@import '@fontsource-variable/inter';
```

El bloque `@theme inline` generado por shadcn ya incluye `--font-sans`. Asegúrate de que su valor sea:

```css
@theme inline {
	--font-sans: 'Inter Variable', sans-serif;
	/* ... resto de tokens generados por shadcn ... */
}
```

Y en el bloque `@layer base`, asegúrate de que `html` aplique la fuente:

```css
@layer base {
	* {
		@apply border-border outline-ring/50;
	}
	body {
		@apply bg-background text-foreground;
	}
	html {
		@apply font-sans;
	}
}
```

---

### D. Configurar Dark Mode (`mode-watcher`)

```bash
pnpm add mode-watcher
```

Edita el componente `src/routes/+layout.svelte` para integrar el watcher de tema:

```svelte
<script lang="ts">
	import { ModeWatcher } from 'mode-watcher';
	import favicon from '$lib/assets/logo.png';
	import './layout.css';

	let { children } = $props();
</script>

<svelte:head>
	<link rel="icon" href={favicon} />
</svelte:head>

<ModeWatcher />

{@render children()}
```

#### Componente del Botón Interactivo de Dark Mode (`ThemeToggle.svelte`)
Primero instala el componente `button` de shadcn-svelte:

```bash
pnpm dlx shadcn-svelte@latest add button
```

Luego crea el componente `src/lib/components/ThemeToggle.svelte`:

```svelte
<script lang="ts">
	import MoonIcon from '@lucide/svelte/icons/moon';
	import SunIcon from '@lucide/svelte/icons/sun';
	import { toggleMode } from 'mode-watcher';

	import { Button } from '$lib/components/ui/button/index.js';
</script>

<Button onclick={toggleMode} variant="outline" size="icon">
	<SunIcon
		class="h-[1.2rem] w-[1.2rem] scale-100 rotate-0 transition-all dark:scale-0 dark:-rotate-90"
	/>
	<MoonIcon
		class="absolute h-[1.2rem] w-[1.2rem] scale-0 rotate-90 transition-all dark:scale-100 dark:rotate-0"
	/>
	<span class="sr-only">Toggle theme</span>
</Button>
```
