# 🚀 Guía Profesional de GitHub (Standard Moderno)

> [!NOTE]
> **Navegación:**
> [🏠 **Inicio**](./README.md) • [📦 **Node.js**](./NodeJS.md) • [🐚 **PowerShell**](./PowerShell.md) • [🚀 **GitHub**](./GitHub.md) • [🐳 **Docker**](./Docker.md) • [🐘 **Drizzle**](./Drizzle.md) • [⚡ **SvelteKit**](./SvelteKit.md) • [🐘 **Postgres**](./PostgreSQL.md) • [⚙️ **PM2**](./PM2.md) • [🔒 **Caddy**](./Caddy.md)

---

Esta guía define el estándar de excelencia para el manejo de repositorios. No es solo una lista de comandos, es un **manifiesto de trabajo** diseñado para la colaboración, la escalabilidad y la paz mental.

---

## 📑 Índice de Navegación

1.  [🛠️ **Fase 1: Configuración Inicial**](#fase-1) (Tutorial / Solo una vez)
2.  [💻 **Fase 2: Trabajo Diario (Flujo Senior)**](#fase-2) (El corazón de tu día)
3.  [🌐 **Fase 3: Despliegue y Servidores**](#fase-3) (Producción)
4.  [🚑 **Fase 4: Supervivencia (Auxilio)**](#fase-4) (Errores y correcciones)

---

## <a id="fase-1"></a>🛠️ 1. Configuración Inicial y Setup

<details>
<summary><b>Haz clic para ver Instalación y Creación de Repos</b></summary>

### Instalación de Herramientas

Usa la terminal para instalar las herramientas oficiales.

```powershell
# Windows (Winget)
winget install --id Git.Git -e --source winget    # Instala Git
winget install --id GitHub.cli -e --source winget # Instala GitHub CLI

# Linux (Ubuntu) - Desarrollo
sudo apt update && sudo apt install git gh -y # Instala Git y GitHub CLI (Completo)

# Linux (Ubuntu) - Producción / VPS
sudo apt update && sudo apt install git -y    # Solo Git (Mínimo necesario)
```

### Autenticación y Configuración

```powershell
gh auth login           # Inicia sesión
gh auth status          # Verifica cuenta
git config --global user.name "Tu Nombre"   # Nombre global
git config --global user.email "tu@email.com" # Email global
git config --global core.autocrlf input     # Evita errores de fin de línea
```

### Crear un Nuevo Repositorio (Público/Privado)

```powershell
# 1. Preparar localmente
git init                            # Crea el repositorio
git add .                           # Prepara archivos
git commit -m "chore: initial commit" # Primer guardado
git branch -M main                  # Renombra rama a main

# 2. Subir a GitHub (Elige una opción)
gh repo create nombre-del-repo --public --source=. --remote=origin --push  # Público
gh repo create nombre-del-repo --private --source=. --remote=origin --push # Privado
```

</details>

---

## <a id="fase-2"></a>💻 2. Trabajo Diario (Flujo Senior)

Este es el ciclo que repetirás cada vez que abras VS Code.

### 📜 El Manifiesto: "La Main es Sagrada"

La rama `main` es el código que está en producción. **Nunca** se sube código directamente a `main`. Todo cambio nace en una rama secundaria.

### 🏷️ Nomenclatura Profesional (Standard Senior)

Usamos prefijos semánticos tanto para las **Ramas** como para los **Commits**.

| Prefijo     | Uso                 | Ejemplo de Rama     | Ejemplo de Commit                 |
| :---------- | :------------------ | :------------------ | :-------------------------------- |
| `feat/`     | Nueva funcionalidad | `feat/login-google` | `feat(auth): login con google`    |
| `fix/`      | Error corregido     | `fix/error-login`   | `fix(ui): margen en botones`      |
| `refactor/` | Mejora de código    | `refactor/db-logic` | `refactor(db): optimizar queries` |
| `docs/`     | Solo documentación  | `docs/readme-v2`    | `docs(readme): añadir guías`      |
| `chore/`    | Tareas técnicas     | `chore/update-deps` | `chore(deps): update vite`        |
| `poc/`      | Experimento         | `poc/test-ia`       | `poc(api): testear nueva lib`     |

> [!TIP]
> **¿Rama o Commit?**
>
> - **La Rama** define la "misión" completa (ej: arreglar el login).
> - **El Commit** define el "paso" específico que diste en ese momento. Una rama `fix/error-login` puede tener commits de tipo `fix`, `docs` o incluso `refactor` dentro.

### 🔄 El Ciclo de Vida de una Tarea

#### Paso 1: Sincronizar y Ramificar

```powershell
git switch main                     # Regresa a main
git pull origin main                # Baja lo último
git switch -c feat/nombre-de-la-tarea # Crea y salta a la rama
```

#### Paso 2: Desarrollo y Guardado (Push Diario)

Trabaja en tu rama. Si termina el día y no has acabado, sube tus avances a TU rama para que el código esté seguro en la nube.

```powershell
git add .                                   # Prepara todos los cambios
git commit -m "feat(scope): avance parcial" # Guarda localmente (Semántico)
git push -u origin feat/nombre-de-la-tarea # Sube y vincula (Solo la 1ª vez)
```

> [!TIP]
> Una vez vinculada la rama, en los siguientes envíos de la misma rama solo necesitas un simple `git push`.

#### Paso 3: Sincronización Final (Safe Sync)

Antes de pedir la integración, trae lo que otros hayan subido a `main` mientras tú trabajabas. Esto evita conflictos en la nube.

```powershell
git switch main                     # Cambia temporalmente a la base
git pull origin main                # Descarga las actualizaciones de otros
git switch feat/nombre-de-la-tarea  # Regresa a tu oficina de trabajo
git rebase main                     # Pon tus cambios "encima" de lo nuevo
```

#### Paso 4: Integración a Main (PR)

```powershell
gh pr create --fill                     # Crea la petición
gh pr merge --squash --delete-branch    # Fusiona y limpia nube
```

#### Paso 5: Limpieza Local (Reset de Entorno)

Tu tarea ya vive en `main`. No dejes "basura" digital en tu PC.

```powershell
git switch main                      # Regresas definitivamente a main
git pull origin main               # Descarga tu propia tarea ya integrada
git branch -d feat/nombre-de-la-tarea # Borra la rama (ya no es necesaria)
```

---

## <a id="fase-3"></a>🌐 3. Despliegue y Servidores (Producción - Linux VPS)

### 🔑 Seguridad con Deploy Keys

En un servidor de producción (Linux), no usas tu cuenta personal. Usas llaves aisladas por proyecto.

<details>
<summary>Ver configuración de Deploy Keys</summary>

> [!NOTE]
> Estos pasos son idénticos en **Windows** (usando PowerShell) y **Linux**. Ambos usan la carpeta `~/.ssh/`.

1.  **Generar llave**: `ssh-keygen -t ed25519 -C "deploy-repo" -f ~/.ssh/id_ed25519_repo`
2.  **Configurar Alias** (`~/.ssh/config`):
    ```bash
    Host github.com-repo
        HostName github.com
        IdentityFile ~/.ssh/id_ed25519_repo
    ```
3.  **Clonar**: `git clone git@github.com-repo:usuario/repo.git`
</details>

### 🚀 Actualización en Producción

¿Por qué usamos `reset --hard` y no `git pull`?

1.  **Limpieza**: `pull` puede fallar si hay cambios accidentales en el servidor. `reset --hard` los borra y deja el código idéntico a GitHub.
2.  **Seguridad**: No borra archivos que no estén en el repositorio (como tu `.env`).

```powershell
# Sincronización atómica (La forma Senior)
git fetch origin           # Baja info del servidor
git reset --hard origin/main # Fuerza igualdad total con main
```

> [!CAUTION]
> **El archivo .env:** Nunca debe estar en Git. Se crea manualmente en el servidor. `git reset --hard` es seguro porque **ignora** los archivos que no están rastreados, manteniendo tu `.env` intacto.

---

## <a id="fase-4"></a>🚑 4. Supervivencia (Auxilio)

<details>
<summary><b>Haz clic para ver cómo arreglar desastres</b></summary>

### Deshacer Cambios (Restore)

```powershell
# Descartar cambios en un archivo (volver al último commit)
git restore src/archivo.ts         # Restaura un archivo
git restore .                      # Restaura TODO (Pánico)

# Sacar del área de preparación (unstage)
git restore --staged src/archivo.ts # Saca del add
```

### ¿Olvidaste crear la rama y editaste main?

No entres en pánico. Dependiendo de si ya hiciste commit o no, hay dos salidas:

#### Escenario A: Editaste archivos pero NO has hecho commit en main

```powershell
git switch -c feat/nombre-de-la-tarea # 1. Crea y salta con tus cambios
git add .                           # 2. Prepara tus archivos
git commit -m "feat(scope): mensaje" # 3. Guarda en la nueva rama
git push -u origin feat/nombre-de-la-tarea # 4. Sube y respalda en la nube
```

#### Escenario B: ¡Ya hice el commit en main!

```powershell
git branch feat/nombre-de-la-tarea # 1. Captura el commit en nueva rama
git reset --hard HEAD~1            # 2. Borra el commit de main (retrocede)
git switch feat/nombre-de-la-tarea # 3. Salta a tu rama (el commit ya está aquí)
git push -u origin feat/nombre-de-la-tarea # 4. Sube y respalda en la nube
```

_Nota: A partir de aquí, sigues con tu **Fase 2 (Trabajo Diario)** normal._

### Limpieza de Ramas

```powershell
# Borrar rama local (si ya se integró)
git branch -d nombre-rama          # Borrado seguro

# Forzar borrado (si no se integró)
git branch -D nombre-rama          # Borrado forzado

# Borrar rama remota (servidor)
git push origin --delete nombre-rama # Borra en GitHub
```

</details>

---

## 🚀 Resumen de Comandos Rápidos

| Acción          | Comando                                                |
| :-------------- | :----------------------------------------------------- |
| **Nueva Tarea** | `git switch -c feat/nombre`                            |
| **Guardar**     | `git add .` + `git commit -m "tipo(alcance): mensaje"` |
| **Subir**       | `git push`                                             |
| **Integrar**    | `gh pr create --fill`                                  |
| **Pánico**      | `git restore .`                                        |

---

_Documentación generada con estándares de excelencia técnica._
