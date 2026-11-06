# 🪟 Comandos de WSL2 en Windows

> Referencia de comandos para la instalación del motor de WSL2, gestión de instancias, arranque, apagado y control desde PowerShell.

---

## 📑 Índice

- [1. Motor de WSL](#1-motor-de-wsl)
- [2. Listado de Distribuciones](#2-listado-de-distribuciones)
- [3. Gestión de Instancias](#3-gestión-de-instancias)

---

## 1. Motor de WSL

```powershell
# Instalar el motor de WSL2 sin instalar ninguna distribución por defecto
wsl --install --no-distribution

# Actualizar el kernel de WSL y sus componentes
wsl --update

# Ver el estado general del motor de WSL2
wsl --status

# Apagar inmediatamente todas las distribuciones y liberar la memoria RAM
wsl --shutdown
```

---

## 2. Listado de Distribuciones

```powershell
# Listar todas las distribuciones instaladas con su estado y versión de WSL
wsl -l -v

# Listar únicamente las distribuciones que se encuentran encendidas actualmente
wsl --list --running

# Listar las distribuciones oficiales disponibles para instalación desde Microsoft
wsl --list --online
```

---

## 3. Gestión de Instancias

```powershell
# Instalar una distribución asignándole un nombre de instancia personalizado
wsl --install -d <nombre-distro> --name <nombre-instancia>

# Establecer una distribución específica como la predeterminada del sistema
wsl --set-default <nombre-instancia>

# Entrar o iniciar una distribución específica por su nombre
wsl -d <nombre-instancia>

# Detener o apagar forzadamente una única distribución específica
wsl --terminate <nombre-instancia>

# Eliminar definitivamente una distribución y borrar todo su disco virtual (.vhdx)
wsl --unregister <nombre-instancia>
```
