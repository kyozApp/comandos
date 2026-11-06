# 🟠 Comandos de Ubuntu 26 en WSL2

> Referencia de comandos para la actualización y limpieza del sistema, configuración del entorno con Systemd y administración esencial de recursos, puertos y SSH.

---

## 📑 Índice

- [1. Actualización y Limpieza del Sistema](#1-actualización-y-limpieza-del-sistema)
- [2. Configuración de Entorno y Systemd](#2-configuración-de-entorno-y-systemd)
- [3. Comandos del Sistema](#3-comandos-del-sistema)

---

## 1. Actualización y Limpieza del Sistema

```bash
# Actualizar el índice de repositorios disponibles
sudo apt update

# Actualizar paquetes instalados a su última versión
sudo apt upgrade -y

# Actualizar resolviendo cambios de dependencias, transiciones y kernel
sudo apt full-upgrade -y

# Eliminar automáticamente paquetes huérfanos y sus configuraciones residuales
sudo apt autoremove --purge -y

# Limpiar la caché local de paquetes descargados para recuperar espacio
sudo apt clean
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
```

```bash
# Recargar la configuración de sesión en la terminal actual
source ~/.bashrc
```

### Configuración de arranque con Systemd (wsl.conf)

> [!NOTE]
> Ubuntu 26.04 en WSL2 genera `/etc/wsl.conf` automáticamente al crear la instancia. Si lo editas, conserva siempre la sección `[user]` para no iniciar sesión como `root`.

```bash
# Abrir el archivo de configuración global de la distribución
sudo nano /etc/wsl.conf
```

**Contenido de `/etc/wsl.conf`:**

```ini
[boot]
systemd=true

[user]
default=<tu_usuario>
```

> [!IMPORTANT]
> Sal de la distribución con `exit` y reinicia WSL desde PowerShell (`wsl --shutdown` o `wsl -t <distro>`) para aplicar cualquier cambio en `/etc/wsl.conf`.

### Persistencia de sesión y optimización de arranque

```bash
# Habilitar persistencia (linger) permanente para el usuario
loginctl enable-linger $USER

# Verificar que la persistencia quedó aplicada (salida esperada: Linger=yes)
loginctl show-user $USER | grep Linger

# Enmascarar la espera de red para acelerar el inicio en WSL
sudo systemctl mask systemd-networkd-wait-online.service
```

---

## 3. Comandos del Sistema

### Monitoreo de recursos y red

```bash
# Ver uso de espacio en disco de las particiones montadas de forma legible
df -h

# Ver uso de memoria RAM y SWAP
free -h

# Ver las interfaces de red y direcciones IP asignadas
ip a
```

### Inspección y control de puertos

```bash
# Comprobar todos los puertos TCP y UDP en escucha con su PID correspondiente
ss -tulpn

# Encontrar el proceso que está escuchando en un puerto específico
lsof -i :3000

# Matar inmediatamente el proceso que ocupa un puerto TCP
sudo fuser -k 3000/tcp
```

### Servidor OpenSSH y claves

> [!TIP]
> Para exponer la instancia por SSH en la red local (puerto dedicado y firewall de Windows) consulta [Acceso SSH a WSL2 desde la Red Local](ssh-wsl-red-local.md).

```bash
# Instalar el servidor OpenSSH
sudo apt update
sudo apt install -y openssh-server

# Habilitar e iniciar el socket SSH
sudo systemctl enable --now ssh.socket

# Comprobar el estado del socket SSH
sudo systemctl status ssh.socket

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
