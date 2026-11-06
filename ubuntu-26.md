# 🟠 Comandos de Ubuntu 26 en WSL2

> Referencia de comandos para la actualización inicial, configuración del entorno con Systemd y comandos esenciales de administración del sistema.

---

## 📑 Índice

- [1. Instalación y Actualización](#1-instalación-y-actualización)
- [2. Configuración de Entorno y Systemd](#2-configuración-de-entorno-y-systemd)
- [3. Comandos del Sistema](#3-comandos-del-sistema)

---

## 1. Instalación y Actualización

```bash
# Actualizar el índice de repositorios disponibles
sudo apt update

# Actualizar todos los paquetes instalados a su última versión
sudo apt upgrade -y
```

---

## 2. Configuración de Entorno y Systemd

### Variables de sesión en bashrc

```bash
# Abrir el archivo de configuración de sesión del usuario
nano ~/.bashrc
```

**Agregar al final de `~/.bashrc`:**

```bash
# Abrir navegador de Windows desde herramientas de WSL
export BROWSER="cmd.exe /c start"

# Definir directorio de runtime del usuario para sockets y servicios
export XDG_RUNTIME_DIR=/run/user/$(id -u)

# Configurar socket del bus de sesión D-Bus del usuario
export DBUS_SESSION_BUS_ADDRESS="unix:path=$XDG_RUNTIME_DIR/bus"
```

```bash
# Recargar la configuración de sesión en la terminal actual
source ~/.bashrc
```

### Configuración de arranque con Systemd (wsl.conf)

```bash
# Abrir el archivo de configuración global de la distribución
sudo nano /etc/wsl.conf
```

**Contenido para `/etc/wsl.conf`:**

```ini
[boot]
systemd=true

[user]
default=kyoz
```

> [!IMPORTANT]
> Sal de la distribución con `exit` y reinicia WSL desde PowerShell (`wsl --shutdown` o `wsl -t <distro>`) para aplicar los cambios antes de continuar.

### Persistencia de sesión y optimización de arranque

```bash
# Salir de la sesión actual de WSL
exit

# Habilitar persistencia (linger) permanente para el usuario
loginctl enable-linger $USER

# Verificar que la persistencia quedó aplicada
loginctl show-user $USER | grep Linger
# Salida: Linger=yes

# Deshabilitar el servicio de espera de red para acelerar el inicio en WSL
sudo systemctl disable systemd-networkd-wait-online.service 2>/dev/null

# Enmascarar el servicio para impedir que se vuelva a activar automáticamente
sudo systemctl mask systemd-networkd-wait-online.service 2>/dev/null
```

---

## 3. Comandos del Sistema

### Inspección y control de puertos

```bash
# Comprobar todos los puertos TCP y UDP en escucha con su PID correspondiente
ss -tulpn

# Encontrar el proceso que está escuchando en un puerto específico
lsof -i :3000

# Matar inmediatamente el proceso que ocupa un puerto TCP
sudo fuser -k 3000/tcp
```

### Mantenimiento y limpieza de paquetes

```bash
# Actualizar paquetes resolviendo cambios de dependencias y eliminando obsoletos
sudo apt full-upgrade -y

# Eliminar automáticamente paquetes huérfanos y dependencias residuales
sudo apt autoremove --purge -y

# Limpiar la caché local de paquetes descargados para recuperar espacio
sudo apt clean
```

### Servidor OpenSSH y claves

```bash
# Instalar el servidor OpenSSH
sudo apt install -y openssh-server

# Habilitar e iniciar el servicio SSH inmediatamente
sudo systemctl enable --now ssh

# Comprobar el estado del servicio SSH
sudo systemctl status ssh

# Generar un nuevo par de llaves seguras ED25519 con comentario identificador
ssh-keygen -t ed25519 -C "tu-correo@ejemplo.com"

# Asignar permisos estrictos requeridos a la carpeta .ssh
chmod 700 ~/.ssh

# Asignar permisos estrictos al archivo de claves autorizadas
chmod 600 ~/.ssh/authorized_keys

# Asignar permisos estrictos a la clave privada en tu máquina cliente
chmod 600 ~/.ssh/id_ed25519
```

---

[← Volver al README](README.md)
