# Comandos GitHub

**Ir a:**

- [Inicio](./README.md)
- [Node.js](./NodeJS.md)
- [PostgreSQL](./PostgreSQL.md)
- [PM2](./PM2.md)
- [Caddy](./Caddy.md)

## Indice

- [Comandos básicos](#comandos-basicos)
- [Instalación en Windows](#instalación-en-windows)
- [Instalación en Linux](#instalación-en-linux)
- [Crear una rama](#crear-una-rama)
- [Crear y subir un proyecto](#crear-y-subir-un-proyecto)
- [Ver cambios antes de actualizar un proyecto](#ver-cambios-antes-de-actualizar-un-proyecto)
- [Subir cambios al repositorio individual o en equipo](#subir-cambios-al-repositorio)

---

## <a id="comandos-basicos"></a>**Comandos básicos**

<details>
<summary>Haz clic para ver</summary>

### Clonar un repositorio:

```powershell
gh repo clone kyozApp/comandos
```

### Inicializar un nuevo repositorio Git:

```powershell
git init
```

### Cambiar el nombre de la rama principal a main:

```powershell
git branch -m master main
```

### Actualizar el repositorio local con los cambios de la rama main:

```powershell
git pull origin main
```

### Subir cambios de una rama local a main:

```powershell
git push origin Erick:main
```

### Eliminar un repositorio desde GitHub:

```powershell
gh repo delete kyozApp/comandos --confirm
```

### Si no tienes permisos para eliminar, ejecuta el siguiente comando:

```powershell
gh auth refresh -h github.com -s delete_repo
```

</details>

<br>

---

## <a id="instalación-en-windows"></a>**Instalación en Windows**

<details>
<summary>Haz clic para ver</summary>

### Descarga e instala GitHub:

```https
https://cli.github.com/
```

### Descarga e instala CLI :

```https
https://git-scm.com/
```

<br>

## Pasos para configurar mi GitHub

### Autenticación de GitHub:

```powershell
gh auth login
```

### Configura nombre de usuario:

```powershell
git config --global user.name "kyozApp"
```

### Configura correo electrónico:

```powershell
git config --global user.email "kyoz.dev@proton.me"
```

<br>

## Opcional

### Configura el formato de línea de finalización:

```powershell
git config --global core.autocrlf input
```

</details>

<br>

---

## <a id="instalación-en-linux"></a>**Instalación en Linux**

<details>
<summary>Haz clic para ver</summary>

### Ingresa como administrador:

```powershell
sudo su
```

###  Instala Git

```powershell
apt install git
```

### Verifica la instalación

```powershell
git --version
```

<br>

## Pasos para configurar mi GitHub

### Autenticación de GitHub:

```powershell
gh auth login
```

### Configura nombre de usuario:

```powershell
git config --global user.name "kyozApp"
```

### Configura correo electrónico:

```powershell
git config --global user.email "kyoz.dev@proton.me"
```


</details>

<br>

---

## <a id="crear-una-rama"></a>**Crear una rama**

<details>
<summary>Haz clic para ver</summary>

##  Cuando no hay ramas remotas aún (primer push)

### Crear y cambiarse de rama:

```powershell
git switch -c Erick
```

### Subir rama al remoto:

```powershell
git push -u origin Erick
```

<br>

## Cuando hay ramas remotas y no locales

### Actualizar información del repositorio remoto:

```powershell
git fetch origin
```

### Crear una rama local basada en la rama remota:

```powershell
git switch -c Erick origin/Erick
```

<br>

## Opcional

### Ver ramas locales:

```powershell
git branch
```

### Ver todas las ramas:

```powershell
git branch -a
```

### Ver ramas locales y remotas:

```powershell
git branch -vv
```

### Crear rama local:

```powershell
git branch Erick
```

### Cambiarse a una rama existente:

```powershell
git switch Erick
```

### Eliminar rama local:

```powershell
git branch -D Erick
```

### Eliminar rama remota:

```powershell
git push origin --delete Erick
```

</details>

<br>

---

## <a id="crear-y-subir-un-proyecto"></a>**Crear y subir un proyecto**

<details>
<summary>Haz clic para ver</summary>

## Crear un repositorio publico:

### Inicializar un nuevo repositorio Git:

```powershell
git init
```

### Agrega los cambios:

```powershell
git add .
```

### Comenta los cambios:

```powershell
git commit -m "Mensaje de commit"
```

### Crea el repositorio:

```powershell
gh repo create comandos --public --source=.
```

### Sube los cambios:

```powershell
git push
```

<br>

## Crear un repositorio privado:

### Inicializar un nuevo repositorio Git:

```powershell
git init
```

### Agrega los cambios:

```powershell
git add .
```

### Comenta los cambios:

```powershell
git commit -m "Mensaje de commit"
```

### Crea el repositorio:

```powershell
gh repo create comandos --private --source=.
```

### Sube los cambios:

```powershell
git push
```

</details>

<br>

---

## <a id="ver-cambios-antes-de-actualizar-un-proyecto"></a>**Ver cambios antes de actualizar un proyecto**

<details>
<summary>Haz clic para ver</summary>

### Ver los cambios específicos en un archivo importante:

```powershell
git diff origin/main src/db/db.ts
```

### Ver qué archivos han cambiado:

```powershell
git diff --name-only origin/main
```

### Ver qué archivos fueron modificados y cómo:

```powershell
git diff --name-status origin/main
```

### Ver todos los cambios:

```powershell
git diff origin/main
```

</details>

<br>

---

## <a id="subir-cambios-al-repositorio"></a>**Subir cambios al repositorio**

<details>
<summary>Haz clic para ver</summary>

## Trabajo individual

### Agrega los cambios:

```powershell
git add .
```

### Comenta los cambios:

```powershell
git commit -m "Mensaje de commit"
```

### Sube los cambios:

```powershell
git push
```

<br>

## Trabajo en equipo

### Agrega los cambios:

```powershell
git add .
```

### Comenta los cambios:

```powershell
git commit -m "Mensaje de commit"
```

### Sube los cambios:

```powershell
git push
```

### Crea una solicitud de cambios:

```powershell
gh pr create --base main --head Erick
```

</details>

<br>

---

