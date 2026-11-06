# 🔑 Acceso SSH a WSL2 desde la Red Local

> Referencia para exponer una o varias instancias WSL2 por SSH en la red local usando modo mirrored, puertos dedicados, firewall de Windows y transferencia de carpetas.

---

## 📑 Índice

- [1. Requisito: Modo Mirrored](#1-requisito-modo-mirrored)
- [2. Configurar OpenSSH en la Instancia](#2-configurar-openssh-en-la-instancia)
- [3. Abrir el Firewall de Windows](#3-abrir-el-firewall-de-windows)
- [4. Conexión Local y Remota](#4-conexión-local-y-remota)
- [5. Transferencia de Carpetas (scp y rsync)](#5-transferencia-de-carpetas-scp-y-rsync)
- [6. Solución de Problemas](#6-solución-de-problemas)
- [7. Desactivar y Revertir](#7-desactivar-y-revertir)

---

## 1. Requisito: Modo Mirrored

El archivo `%USERPROFILE%\.wslconfig` en Windows debe contener:

```ini
[wsl2]
networkingMode=mirrored
dnsTunneling=true
autoProxy=true
```

```powershell
# Reiniciar WSL para aplicar cambios en .wslconfig
wsl --shutdown
```

```bash
# Verificar dentro de la instancia que el modo de red sea mirrored (salida esperada: mirrored)
wslinfo --networking-mode
```

---

## 2. Configurar OpenSSH en la Instancia

> [!IMPORTANT]
> En modo mirrored todas las instancias comparten la IP de Windows, por lo que **cada instancia necesita su propio puerto** (ej. `2221` para la primera, `2222` para la segunda). Evita el puerto `22`, que puede estar ocupado por el OpenSSH de Windows.

```bash
# Actualizar repositorios e instalar el servidor OpenSSH
sudo apt update
sudo apt install -y openssh-server

# Crear archivo de configuración independiente para el puerto dedicado
sudo nano /etc/ssh/sshd_config.d/10-port.conf
```

**Contenido para `/etc/ssh/sshd_config.d/10-port.conf`:**

```text
Port 2221
```

Guarda con `Ctrl + O` → `Enter` → `Ctrl + X`, valida y aplica:

```bash
# Validar la sintaxis de la configuración de sshd
sudo sshd -t

# Aplicar el puerto (Ubuntu 26 usa activación por socket: ssh.socket)
sudo systemctl daemon-reload
sudo systemctl restart ssh.socket

# Habilitar el inicio automático del socket SSH
sudo systemctl enable ssh.socket

# Confirmar que el puerto 2221 está en escucha
ss -tlnp | grep 2221
```

> [!NOTE]
> Desde Ubuntu 24.04, `sshd` se inicia mediante `ssh.socket`. Reiniciar solo `ssh.service` no aplica el cambio de puerto; siempre ejecuta `daemon-reload` y reinicia `ssh.socket`.

---

## 3. Abrir el Firewall de Windows

> [!IMPORTANT]
> Ejecutar en **Windows PowerShell** clásico (ícono azul) como **Administrador**. En PowerShell 7 (`pwsh`) algunos cmdlets de firewall pueden fallar.

```powershell
# Permitir tráfico entrante hacia WSL en el firewall de Hyper-V
Set-NetFirewallHyperVVMSetting -Name '{40E0AC32-46A5-438A-A0B2-2B479E8F2E90}' -DefaultInboundAction Allow

# Abrir el puerto SSH de la instancia en Windows Defender Firewall
New-NetFirewallRule -DisplayName "WSL SSH 2221" -Direction Inbound -LocalPort 2221 -Protocol TCP -Action Allow

# Listar las reglas creadas para WSL SSH
Get-NetFirewallRule -DisplayName "WSL SSH*"
```

---

## 4. Conexión Local y Remota

```bash
# Consultar tu usuario Linux real (es el que se usa para conectarse, no el nombre de la instancia)
whoami
```

```powershell
# Consultar la IPv4 de Windows (adaptador Wi-Fi o Ethernet, ej. 192.168.1.50)
ipconfig
```

```bash
# Conectarse desde el mismo Windows
ssh <usuario>@localhost -p 2221

# Conectarse desde otra PC de la misma red local (Linux, macOS o Windows)
ssh <usuario>@<ip_windows> -p 2221
```

---

## 5. Transferencia de Carpetas (scp y rsync)

```bash
# Copiar una carpeta local hacia la instancia WSL de otra PC
scp -P 2221 -r ./mi-proyecto <usuario>@<ip_windows>:~/proyectos/

# Copiar una carpeta desde la instancia WSL remota hacia tu equipo
scp -P 2221 -r <usuario>@<ip_windows>:~/proyectos/mi-proyecto ./

# Sincronizar solo diferencias, conservando permisos y con progreso (recomendado para carpetas pesadas)
rsync -avP -e 'ssh -p 2221' --exclude 'node_modules/' ./mi-proyecto/ <usuario>@<ip_windows>:~/proyectos/mi-proyecto/
```

---

## 6. Solución de Problemas

| Síntoma                                       | Causa                                                     | Solución                                                                            |
| :-------------------------------------------- | :-------------------------------------------------------- | :---------------------------------------------------------------------------------- |
| `Permission denied (publickey,password)`      | Se usó el nombre de la instancia en vez del usuario Linux | Conectar con el usuario que devuelve `whoami`                                       |
| `Connection refused` en el puerto nuevo       | `ssh.socket` sigue escuchando en el puerto 22             | Ejecutar `sudo systemctl daemon-reload` y luego `sudo systemctl restart ssh.socket` |
| Funciona en `localhost` pero no desde otra PC | Firewall de Hyper-V o de Windows bloqueando               | Repetir los comandos de la sección 3 como Administrador                             |
| `wslinfo` devuelve `nat`                      | `.wslconfig` no aplicado                                  | Revisar el archivo y ejecutar `wsl --shutdown`                                      |

---

## 7. Desactivar y Revertir

```bash
# Detener y deshabilitar SSH temporalmente en la instancia
sudo systemctl disable --now ssh.socket ssh.service

# Desinstalar OpenSSH por completo junto con su configuración
sudo apt remove --purge -y openssh-server

# Limpiar dependencias huérfanas y caché de paquetes
sudo apt autoremove --purge -y
sudo apt clean
```

```powershell
# Eliminar la regla del firewall de Windows (PowerShell como Administrador)
Remove-NetFirewallRule -DisplayName "WSL SSH 2221"
```

---

[← Volver al README](README.md)
