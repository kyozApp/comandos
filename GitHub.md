# 🚀 Guía Profesional de GitHub (Standard Moderno)

> [!NOTE]
> **Navegación:**
> [🏠 **Inicio**](./README.md) • [📦 **Node.js**](./NodeJS.md) • [🐚 **PowerShell**](./PowerShell.md) • [🚀 **GitHub**](./GitHub.md) • [🐘 **Postgres**](./PostgreSQL.md) • [⚙️ **PM2**](./PM2.md) • [🔒 **Caddy**](./Caddy.md)

---

Esta guía define el estándar de excelencia para el manejo de repositorios. No es solo una lista de comandos, es un **manifiesto de trabajo** diseñado para la colaboración, la escalabilidad y la paz mental.

---

## 📑 Índice

1.  [🚀 Instalación y Setup](#instalacion)
2.  [📜 El Manifiesto Git (La Main es Sagrada)](#manifiesto)
3.  [🏗️ Ramas: Tu Espacio de Trabajo Real](#ramas-espacio)
4.  [🏷️ Nomenclatura Semántica (Ramas y Commits)](#nomenclatura)
5.  [🔄 Flujo de Feature Branching (Alto Rendimiento)](#feature-branching)
6.  [🧪 Pruebas de Concepto (Flujo PoC v1 vs v2)](#poc)
7.  [👥 Trabajo en Equipo (Pull Requests con `gh`)](#equipo)
8.  [📅 El Ritmo Diario (¿Dónde me quedo al terminar?)](#ritmo-diario)
9.  [👨‍💻 El Flujo del Desarrollador (Manual de Supervivencia)](#flujo-desarrollador)
10. [🔑 Seguridad en Servidores (Deploy Keys)](#deploy-keys)
11. [📁 Crear un Nuevo Repositorio (Público/Privado)](#crear-repo)
12. [🛡️ Supervivencia y Recuperación](#supervivencia)

---

## <a id="instalacion"></a>🚀 1. Instalación y Setup Moderno

Un profesional no descarga instaladores manualmente si puede evitarlo.

### Windows (Vía Winget)
Usa la terminal (PowerShell) para instalar las herramientas oficiales de Microsoft.
```powershell
# Instala Git
winget install --id Git.Git -e --source winget

# Instala GitHub CLI (Indispensable)
winget install --id GitHub.cli -e --source winget
```

### Linux (Ubuntu/Debian)
```bash
sudo apt update
sudo apt install git gh -y
```

### Autenticación Premium (CLI)
Olvida las contraseñas. Usa la GitHub CLI para gestionar todo.
```powershell
# Inicia sesión de forma interactiva
gh auth login

# Verifica que todo esté correcto
gh auth status
```

### Configuración de Identidad
Git necesita saber quién eres para que tus commits tengan validez legal y técnica.
```powershell
git config --global user.name "Tu Nombre"
git config --global user.email "tu@email.com"

# Recomendado: Final de línea automático (evita errores entre Windows/Linux)
git config --global core.autocrlf input
```

---

## <a id="manifiesto"></a>📜 2. El Manifiesto Git

### "La Main es Sagrada"
La rama `main` es el código que está en producción. **Nunca** se sube código directamente a `main` sin que haya pasado por una rama de tarea y una revisión.

> [!IMPORTANT]
> Si rompes `main`, rompes el negocio. Todo cambio nace en una rama secundaria.

---

## <a id="ramas-espacio"></a>🏗️ 3. Ramas: Tu Espacio de Trabajo Real

### ¿Las ramas son solo para tareas?
**No. Las ramas son tu oficina.** 

Muchos desarrolladores cometen el error de pensar que las ramas son "trámites" para subir código. **Falso.** La rama es tu espacio de seguridad.

1.  **Aislamiento Total**: En una rama puedes romper el código, borrar archivos y experimentar sin miedo. Si algo sale mal, simplemente borras la rama y `main` sigue intacta.
2.  **Multitarea Real**: Si estás trabajando en una funcionalidad (`feat/nueva-ui`) y de pronto surge un error crítico en producción, haces un `git switch main` y luego `git switch -c fix/error-critico`. Arreglas el error, lo subes, y regresas a tu UI exactamente donde la dejaste.
3.  **Orden Mental**: Trabajar en ramas te obliga a enfocarte en **una sola cosa a la vez**. Si estás en `feat/login`, solo tocas cosas de login.

> [!CAUTION]
> Trabajar en `main` es como intentar limpiar tu casa mientras hay una fiesta de 100 personas. Nunca terminarás y probablemente rompas algo valioso.

---

## <a id="nomenclatura"></a>🏷️ 4. Nomenclatura Semántica

### Ramas Profesionales
No usamos nombres propios (`erick/`, `dev/`). Usamos el **propósito**.

| Prefijo | Uso | Ejemplo |
| :--- | :--- | :--- |
| `feat/` | Nueva funcionalidad | `feat/login-google` |
| `fix/` | Corrección de un error | `fix/css-navbar` |
| `chore/` | Tareas de mantenimiento o config | `chore/update-deps` |
| `poc/` | Prueba de concepto / Experimento | `poc/new-design-v1` |
| `docs/` | Cambios solo en documentación | `docs/api-ref` |
| `refactor/` | Mejora de código sin cambiar lógica | `refactor/auth-logic` |

### Commits Semánticos (Conventional Commits)
Tus mensajes de commit deben contar una historia clara.
**Formato:** `tipo: descripción corta`

```powershell
git commit -m "feat: implementar autenticación con JWT"
git commit -m "fix: corregir salto de línea en el pie de página"
git commit -m "chore: configurar eslint y prettier"
```

---

## <a id="feature-branching"></a>🔄 5. Flujo de Feature Branching (Alto Rendimiento)

Este es el ciclo de vida de cualquier tarea profesional.

### Paso 1: Sincronizar y Crear
Siempre partimos de una base limpia y actualizada.
```powershell
# Asegúrate de estar en main y actualizado
git switch main
git pull origin main

# Crea y salta a la nueva rama (Sintaxis Moderna)
git switch -c feat/nombre-de-la-tarea
```

### Paso 2: El Ciclo de Desarrollo
```powershell
# Agrega solo lo necesario (o todo si eres ordenado)
git add .

# Haz commits pequeños y frecuentes
git commit -m "feat: estructura básica del componente"
```

### Paso 3: El Primer Push (Upstream)
La primera vez que subes la rama, debes presentarla al servidor.
```powershell
git push -u origin feat/nombre-de-la-tarea
```
*Las siguientes veces solo necesitarás un simple `git push`.*

---

## <a id="poc"></a>🧪 6. Pruebas de Concepto (Flujo PoC v1 vs v2)

¿Tienes dos ideas y no sabes cuál es mejor? No las mezcles.

### Opción 1: Crear la V1
```powershell
git switch main
git switch -c poc/propuesta-visual-v1
# ... haces tus cambios ...
git add .
git commit -m "poc: implementación de la propuesta v1"
git push -u origin poc/propuesta-visual-v1
```

### Opción 2: Crear la V2 (Limpieza Total)
```powershell
# Crucial: Regresar a main para no heredar nada de la V1
git switch main
git switch -c poc/propuesta-visual-v2
# ... haces tus cambios ...
git add .
git commit -m "poc: implementación de la propuesta v2"
git push -u origin poc/propuesta-visual-v2
```

---

## <a id="equipo"></a>👥 7. Trabajo en Equipo (Pull Requests con `gh`)

Integrar cambios de forma segura es la marca de un desarrollador senior.

### El "Safe Merge" (Sincronización Pre-Integración)
Antes de pedir un Pull Request, asegúrate de que tu rama está al día con `main` para evitar conflictos.
```powershell
git switch main
git pull origin main
git switch feat/tu-tarea
git merge main
```

### Crear un Pull Request (PR) desde Terminal
No pierdas tiempo en la web de GitHub. Usa la CLI.
```powershell
# Crea la PR y rellena el título/cuerpo automáticamente desde los commits
gh pr create --fill

# Si quieres elegir la base manualmente
gh pr create --base main --head feat/tu-tarea --title "feat: nueva funcionalidad"
```

### Revisión y Fusión (Merge)
```powershell
# Listar PRs abiertas
gh pr list

# Ver qué cambió en una PR específica
gh pr diff [numero_pr]

# Integrar la PR (Merge) de forma profesional
gh pr merge [numero_pr] --squash --delete-branch
```
> [!TIP]
> El flag `--squash` limpia el historial de commits antes de entrar a `main`. El flag `--delete-branch` borra la rama automáticamente del servidor tras el merge.

---

## <a id="ritmo-diario"></a>📅 8. El Ritmo Diario (¿Dónde me quedo al terminar?)

Esta es la duda que separa a los novatos de los pros: **"¿Qué hago con la rama cuando termino?"**

### Escenario A: Una tarea que dura varios días
No creas una rama cada día. La rama vive lo que viva la tarea.
*   **Lunes**: Creas `feat/nombre-de-la-tarea`. Haces commits y `git push`. Te quedas en esa rama al apagar la PC.
*   **Martes**: Enciendes la PC, sigues en `feat/nombre-de-la-tarea`. Sigues trabajando. `git push`.
*   **Miércoles**: Terminas la tarea.

### Escenario B: Finalización y Limpieza (Merge & Destroy)
Una vez que tu código llega a `main` (vía PR o Merge), la rama ya no tiene propósito. **Es basura.**
1.  **Regresa a casa**: `git switch main`
2.  **Actualiza tu base**: `git pull origin main` (Ahora tienes tu código nuevo dentro de main).
3.  **Destruye la evidencia**: `git branch -d feat/nombre-de-la-tarea`. 

> [!CRITICAL]
> **No seas un "acumulador digital"**. Si tienes 20 ramas locales, estás trabajando mal. Un repositorio profesional solo tiene ramas de tareas **activas**. Si la tarea terminó, la rama muere.

### ¿En qué rama me quedo después de subir?
*   Si la tarea **sigue pendiente**: Quédate en la rama de la tarea.
*   Si la tarea **terminó**: Regresa a `main`, borra la rama vieja y crea una nueva para el siguiente reto.

---

## <a id="flujo-desarrollador"></a>👨‍💻 9. El Flujo del Desarrollador (Manual de Supervivencia)

Si eres nuevo en un proyecto, este es el camino que debes seguir. No improvises.

### 1. El Inicio: Clonar y Ubicarse
```powershell
gh repo clone usuario/nombre-del-repo
# Por defecto, estarás en 'main'. Es la zona de "solo lectura" para ti.
```

### 2. La Creación: Tu zona segura
```powershell
git switch -c feat/nombre-de-la-tarea
# Ahora estás en tu propia burbuja. Aquí trabajas hoy, mañana y lo que haga falta.
```

### 3. El Final del Día: Guardar en la Nube
**¿Se termina el día y no has acabado la tarea?** No importa. Sube tus cambios a TU rama.
```powershell
git add .
git commit -m "feat: avance parcial de la tarea"
git push -u origin feat/nombre-de-la-tarea
```
*Esto no afecta a nadie más. Tu código está seguro en GitHub, pero no en la rama principal.*

### 4. ¿Cómo se prueba si no está en Main?
**Crítica técnica:** Nunca, jamás, se prueba en Producción.
*   **Prueba Local:** Tú lo pruebas en tu propia computadora. Si no funciona en tu PC, no se sube.
*   **Entornos de Desarrollo (Dev/Staging):** En equipos profesionales, al subir tu rama, el servidor crea una versión "especial" (un link de prueba) solo para tu rama. Así el líder puede ver si funciona sin romper la web oficial.

### 5. La Integración: El Pull Request (PR)
Cuando estés 100% seguro de que funciona:
```powershell
gh pr create --fill
```
Aquí es donde ocurre la magia:
1.  **Code Review:** Un programador senior leerá tu código. Te dirá: "Esto está mal, cámbialo".
2.  **Aprobación:** Solo cuando te den el "Check", tu código se une a `main`.

### 6. El Cierre: Limpieza
Una vez que el PR se acepta:
1.  Vuelves a `main`: `git switch main`
2.  Bajas lo nuevo (incluyendo lo tuyo): `git pull origin main`
3.  **Borras tu rama**: `git branch -d feat/nombre-de-la-tarea`

> [!CAUTION]
> **¿Si borro la rama pierdo el código?** No. Si la borras **después** del merge, tu código ya vive en `main`. Borrar la rama es como tirar el andamio después de terminar el edificio. El edificio (tu código) se queda en `main`.

---

## <a id="deploy-keys"></a>🔑 10. Seguridad en Servidores (Deploy Keys)

Cuando despliegas en un VPS (como en `nombre-del-repo`), no usas tu cuenta personal de GitHub. Usas **Deploy Keys**.

### ¿Por qué es mejor? (Crítica)
Si usas tu cuenta personal en el servidor y alguien hackea el servidor, tiene acceso a **TODOS** tus repositorios. Con una Deploy Key, solo tienen acceso (y normalmente solo de lectura) a **UN** repositorio. Es aislamiento profesional.

### Paso 1: Generar la llave en el Servidor
```bash
# Crea una llave específica para el proyecto
ssh-keygen -t ed25519 -C "deploy-nombre-del-repo" -f ~/.ssh/id_ed25519_nombre_del_repo
```

### Paso 2: Configurar el Alias (SSH Config)
Para que Git sepa qué llave usar, editamos `~/.ssh/config`:
```bash
Host github.com-nombre-del-repo
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_nombre_del_repo
    IdentitiesOnly yes
```

### Paso 3: Registrar en GitHub
1. Copia la llave pública: `cat ~/.ssh/id_ed25519_nombre_del_repo.pub`
2. Ve a tu repo en GitHub -> **Settings > Deploy keys**.
3. Pégala y **NO marques** "Allow write access". El servidor solo debe leer código, no subirlo.

### Paso 4: Clonar con el Alias
```bash
git clone git@github.com-nombre-del-repo:usuario/nombre-del-repo.git
```

> [!TIP]
> Al usar este flujo, proteges tu cuenta personal y aseguras que el servidor sea una "isla" aislada. Si el servidor se ve comprometido, solo borras esa Deploy Key y el resto de tus proyectos están a salvo.

---

## <a id="crear-repo"></a>📁 11. Crear un Nuevo Repositorio (Público/Privado)

Si tienes un proyecto en tu PC y quieres subirlo a GitHub por primera vez, usa la CLI. Es más rápido que la web.

### Paso 1: Inicializar Git Localmente
```powershell
# 1. Crea el repositorio (Nace por defecto como 'master')
git init

# 2. Prepara y guarda tus archivos
git add .
git commit -m "chore: initial commit"

# 3. Renombra la rama a 'main' (Solo puedes renombrar algo que YA existe)
git branch -M main
```

### Paso 2: Crear el Repo en la Nube
Elige si quieres que el mundo lo vea (`--public`) o si es secreto (`--private`).

#### Opción A: Repositorio Público
```powershell
gh repo create nombre-del-repo --public --source=. --remote=origin --push
```

#### Opción B: Repositorio Privado
```powershell
gh repo create nombre-del-repo --private --source=. --remote=origin --push
```

> [!NOTE]
> El flag `--source=.` le dice a GitHub que use la carpeta actual. El flag `--push` sube el código inmediatamente después de crear el repositorio.

---

## <a id="supervivencia"></a>🛡️ 12. Supervivencia y Recuperación

Todos cometemos errores. Aquí cómo arreglarlos sin pánico.

### Deshacer Cambios (Sintaxis Moderna)
```powershell
# Descartar cambios en un archivo específico (volver al estado del último commit)
git restore src/db/db.ts

# Sacar un archivo del área de preparación (unstage)
git restore --staged src/db/db.ts
```

### Limpieza de Ramas
```powershell
# Borrar rama local (solo si ya se integró)
git branch -d feat/nombre-de-la-tarea

# Forzar borrado (cuidado: borra aunque no esté integrada)
git branch -D feat/experimento-fallido

# Borrar rama remota
git push origin --delete feat/nombre-de-la-tarea
```

### Actualización de VPS (Producción)
En el servidor, no queremos "desarrollar", solo queremos los cambios exactos de `main`.
```powershell
# La forma segura: Bajar cambios y forzar que el estado local sea igual al remoto
git fetch origin
git reset --hard origin/main
```

---

## 🚀 Resumen de Comandos Rápidos

| Acción | Comando |
| :--- | :--- |
| Nueva Tarea | `git switch -c feat/nombre-de-la-tarea` |
| Guardar | `git add .` + `git commit -m "tipo: mensaje"` |
| Subir | `git push` (o `git push -u origin feat/nombre-de-la-tarea` la primera vez) |
| Integrar | `gh pr create --fill` |
| Pánico | `git restore .` |

---
*Documentación generada con estándares de excelencia técnica.*
