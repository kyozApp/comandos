# 🚀 Despliegue en VPS con Systemd y Rsync

> Referencia de comandos para el despliegue de aplicaciones en servidores VPS: sincronización con exclusiones mediante Rsync, definición de servicios Systemd con Cgroups v2 y verificación de salud de puertos.

---

## 📑 Índice

- [1. Sincronización de Código con Exclusiones (Rsync)](#1-sincronización-de-código-con-exclusiones-rsync)
- [2. Configuración y Control del Servicio Systemd](#2-configuración-y-control-del-servicio-systemd)
- [3. Análisis y Filtrado de Logs con Journalctl](#3-análisis-y-filtrado-de-logs-con-journalctl)
- [4. Diagnóstico de Salud y Liberación de Puertos](#4-diagnóstico-de-salud-y-liberación-de-puertos)
- [5. Flujos de Trabajo Agrupados](#5-flujos-de-trabajo-agrupados)

---

## 1. Sincronización de Código con Exclusiones (Rsync)

```bash
# Sincronizar proyecto API/Backend hacia el VPS por SSH con exclusiones
rsync -avz -e 'ssh -p <puerto_ssh>' --exclude 'node_modules/' --exclude '.git/' --exclude 'dist/' --exclude 'build/' --exclude '.env' --exclude '.agents' --exclude 'skills' ~/proyectos/<mi_proyecto>/ <usuario>@<ip_servidor>:~/proyectos/<mi_proyecto>/

# Simular la sincronización sin realizar cambios reales en disco (--dry-run)
rsync -avz --dry-run -e 'ssh -p <puerto_ssh>' --exclude 'node_modules/' --exclude '.git/' ~/proyectos/<mi_proyecto>/ <usuario>@<ip_servidor>:~/proyectos/<mi_proyecto>/
```

---

## 2. Configuración y Control del Servicio Systemd

```bash
# Crear o editar el archivo de configuración del servicio en Systemd
sudo nano /etc/systemd/system/<mi_servicio>.service

# Recargar el daemon para registrar cambios en los archivos de servicio
sudo systemctl daemon-reload

# Habilitar el inicio automático al reiniciar el servidor y arrancar inmediatamente
sudo systemctl enable --now <mi_servicio>

# Reiniciar el servicio tras compilar una nueva versión
sudo systemctl restart <mi_servicio>

# Ver el estado y consumo de recursos en tiempo real del servicio
sudo systemctl status <mi_servicio>

# Detener el servicio inmediatamente
sudo systemctl stop <mi_servicio>

# Deshabilitar el arranque automático
sudo systemctl disable <mi_servicio>

# Limpiar cualquier estado residual de fallos en memoria
sudo systemctl reset-failed
```

### Configuración de Referencia para el archivo `.service`

> [!IMPORTANT]
> **Campos que debes actualizar según tu proyecto:**
>
> - **`Description`**: Nombre legible del servicio (ej. `mi-api REST Service`).
> - **`User`**: Usuario de Linux sin privilegios root que ejecutará el proceso (`<usuario>`).
> - **`WorkingDirectory`**: Ruta absoluta hacia la raíz de tu proyecto en el servidor (ej. `/home/<usuario>/proyectos/mi-api`).
> - **`ExecStart`**: Ruta absoluta hacia el binario estable de Node (verifícalo con `which node`, requiere Node >= 20.6.0 para `--env-file`) junto al archivo compilado y `--env-file=.env`.
> - **`SyslogIdentifier`**: Identificador único que se usará para consultar los logs en Journalctl (`journalctl -u <mi_servicio> -f`).

```ini
[Unit]
Description=mi-api REST Service
After=network.target

[Service]
Type=simple
User=<usuario>
WorkingDirectory=/home/<usuario>/proyectos/mi-api
ExecStart=/home/<usuario>/.local/share/pnpm/bin/node --env-file=.env dist/index.js
Restart=always
RestartSec=5
Environment=NODE_ENV=production

# Parada limpia y gestión de subprocesos
KillMode=control-group
TimeoutStopSec=15

# Protección de memoria (Cgroups v2)
MemoryMax=1G
MemoryHigh=800M

# Logs centralizados con Journald
StandardOutput=journal
StandardError=journal
SyslogIdentifier=mi-api

[Install]
WantedBy=multi-user.target
```

---

## 3. Análisis y Filtrado de Logs con Journalctl

```bash
# Seguir los logs en vivo y en tiempo real de la API
journalctl -u <mi_servicio> -f

# Ver las últimas 100 líneas combinadas sin paginador
journalctl -u <mi_servicio> -n 100 --no-pager

# Filtrar exclusivamente los mensajes de error (sin ruido informativo)
journalctl -u <mi_servicio> -p err --no-pager

# Ver logs de una ventana de tiempo específica (ejemplo: últimos 15 minutos)
journalctl -u <mi_servicio> --since "15 minutes ago"

# Ver únicamente los logs generados desde el último reinicio del servidor (-b)
journalctl -u <mi_servicio> -b

# Verificar el uso total de espacio en disco de los logs del sistema
journalctl --disk-usage
```

---

## 4. Diagnóstico de Salud y Liberación de Puertos

```bash
# Probar respuesta HTTP real del endpoint de salud
curl -I http://localhost:3000/health

# Comprobar que el puerto esté activo y en escucha en el sistema
ss -tulpn | grep 3000

# Listar todos los servicios personalizados creados en Systemd
find /etc/systemd/system/ -maxdepth 1 -type f -name "*.service" -exec basename {} \;

# Matar el proceso y liberar el puerto TCP inmediatamente (si corre bajo Systemd, detén primero el servicio)
sudo fuser -k 3000/tcp
```

---

## 5. Flujos de Trabajo Agrupados

```bash
# ==============================================================================
# Flujo: Despliegue de actualización en el servidor VPS
# ==============================================================================

# 1. En el VPS, entrar al directorio del proyecto y compilar la nueva versión
cd ~/proyectos/<mi_proyecto>
pnpm build

# 2. Reiniciar el servicio para aplicar los cambios del nuevo build
sudo systemctl restart <mi_servicio>

# 3. Comprobar que el servicio haya arrancado con éxito
sudo systemctl status <mi_servicio>

# 4. Probar la respuesta HTTP del endpoint de salud
curl -I http://localhost:3000/health
```

---

[← Volver al README](README.md)
