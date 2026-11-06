# 🐙 Comandos de Git y GitHub CLI

> Referencia de comandos para control de versiones con Git moderno (ramas, stash, historial, rebase) y gestión remota con GitHub CLI (`gh`).

---

## 📑 Índice

- [1. Instalación y Autenticación en Linux](#1-instalación-y-autenticación-en-linux)
- [2. Configuración Inicial e Identidad](#2-configuración-inicial-e-identidad)
- [3. Crear un Nuevo Repositorio](#3-crear-un-nuevo-repositorio)
- [4. Flujos de Trabajo según el Proyecto](#4-flujos-de-trabajo-según-el-proyecto)

---

## 1. Instalación y Autenticación en Linux

```bash
# Instalar Git y GitHub CLI en Ubuntu/Debian
sudo apt install git gh -y

# Iniciar sesión en GitHub mediante autenticación web en el navegador
gh auth login
```

---

## 2. Configuración Inicial e Identidad

```bash
# Configurar tu nombre de usuario global en Git
git config --global user.name "Tu Nombre"

# Configurar tu correo electrónico global en Git
git config --global user.email "tu-correo@ejemplo.com"

# Establecer "main" como el nombre de rama inicial predeterminado
git config --global init.defaultBranch main

# Configurar saltos de línea LF limpios para Linux y WSL
git config --global core.autocrlf input

# Ver toda la configuración global activa y archivos de origen
git config --list --show-origin
```

---

## 3. Crear un Nuevo Repositorio

```bash
# Inicializar Git en la carpeta del proyecto
git init

# Agregar todos los archivos al área de preparación
git add .

# Crear el primer commit
git commit -m "mensaje"

# Asegurar que la rama principal se llame main
git branch -M main
```

```bash
# Opción A: Crear repositorio remoto público en GitHub y subir el código
gh repo create nombre-del-repo --public --source=. --remote=origin --push
```

```bash
# Opción B: Crear repositorio remoto privado en GitHub y subir el código
gh repo create nombre-del-repo --private --source=. --remote=origin --push
```

---

## 4. Flujos de Trabajo según el Proyecto

### Modalidad 1: Proyecto con solo una rama (`main`)

> [!NOTE]
> En `main` se realizan los cambios directos del día a día: proyectos individuales, dependencias, ajustes menores y documentación.

#### Guardar y subir cambios directos en main

```bash
# Agregar todos los archivos modificados
git add .

# Guardar los cambios con un mensaje
git commit -m "mensaje"
```

```bash
# Opción A: Subir por primera vez en main
git push origin main
```

```bash
# Opción B: Subidas diarias posteriores
git push
```

#### Stash en main (Pausa de emergencia)

> [!TIP]
> **Caso de uso:** Estás programando a medias en `main`, alguien subió cambios a GitHub y quieres hacer `git pull`, pero Git te bloquea por tener cambios locales sin guardar. La bandera `-u` asegura incluir también archivos nuevos sin rastrear (_untracked_).

```bash
# 1. Guardar tus cambios a medias en el stash incluyendo archivos nuevos (-u) para dejar main limpio
git stash push -u -m "ajustes a medias en main"

# 2. Bajar los cambios remotos de GitHub sin bloqueos
git pull origin main

# 3. Recuperar y reaplicar tus cambios de vuelta
git stash pop
```

---

### Modalidad 2: Proyecto con varias ramas

> [!TIP]
> **Regla de oro:** En este modo nunca programas ni subes directamente a `main`. Todo desarrollo se hace en una rama dedicada (`feature/` o `fix/`). Cuando terminas, subes tu rama y envías la solicitud (Pull Request) para que `main` revise y apruebe los cambios.
>
> | Prefijo    | Cuándo usarlo                                     | Ejemplo                |
> | :--------- | :------------------------------------------------ | :--------------------- |
> | `feature/` | Cuando construyes un módulo o funcionalidad nueva | `feature/metodo-pago`  |
> | `fix/`     | Cuando solucionas un error reportado              | `fix/error-login-401`  |
> | `bugfix/`  | Cuando solucionas un bug reportado                | `bugfix/bug-login-401` |
>
> _Todo lo demás va directo a `main` sin crear ramas extras._

#### Guardar, subir y solicitar revisión (Pull Request)

```bash
# Opción A: Crear una nueva rama y cambiarse a ella de inmediato
git switch -c feature/nueva-funcionalidad
```

```bash
# Opción B: Cambiarse a una rama existente que ya fue creada
git switch feature/nueva-funcionalidad
```

```bash
# Traer a tu rama lo último actualizado de main antes de empezar
git pull origin main

# Agregar todos los cambios realizados
git add .

# Guardar los cambios en la rama
git commit -m "feature: mensaje"
```

```bash
# Opción A: Subir la rama por primera vez y vincularla al remoto (-u)
git push -u origin feature/nueva-funcionalidad
```

```bash
# Opción B: Subidas diarias posteriores en esta rama
git push
```

```bash
# Crear la solicitud (Pull Request) para que main revise y apruebe la unión
gh pr create --title "feature: nueva funcionalidad" --body "Implementación lista para revisión"
```

#### Stash en rama y solución de urgencia (Pausa de emergencia)

> [!TIP]
> **Caso de uso:** Estás a mitad de programar tu rama (código incompleto) y surge un error urgente que debes solucionar. En vez de tocar `main` directamente, pausas tu trabajo con stash, creas una rama dedicada `fix/`, resuelves el problema, envías la solicitud (Pull Request) a `main` y retomas tu código de inmediato.

```bash
# 1. Guardar tus cambios a medias en el stash incluyendo archivos nuevos (-u) para dejar tu rama feature limpia
git stash push -u -m "funcionalidad a medias"

# 2. Cambiarte a main para partir desde la versión estable
git switch main

# 3. Traer lo último actualizado del repositorio remoto
git pull origin main

# 4. Crear la rama de corrección y cambiarse a ella de inmediato
git switch -c fix/error-login

# 5. Aquí trabajas en solucionar el error en el código...

# 6. Agregar todos los archivos modificados de la solución
git add .

# 7. Guardar el commit con mensaje descriptivo del arreglo
git commit -m "fix: corregir error urgente de login"

# 8. Subir la rama de corrección a GitHub
git push -u origin fix/error-login

# 9. Mandar la solicitud (Pull Request) a main para su revisión y aprobación
gh pr create --title "fix: corregir error urgente de login" --body "Solución del error para revisión"

# 10. Regresar de inmediato a tu rama de desarrollo original sin esperar la aprobación
git switch feature/nueva-funcionalidad

# 11. Recuperar tus cambios a medias exactamente donde los dejaste
git stash pop
```

---

[← Volver al README](README.md)
