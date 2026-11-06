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
# Sincronizar proyecto API/Backend hacia el VPS por puerto SSH 27 con exclusiones
rsync -avz -e 'ssh -p 27' --exclude 'node_modules/' --exclude '.git/' --exclude 'dist/' --exclude 'build/' --exclude '.env' --exclude '.agents' --exclude 'skills' --exclude '.trae' ~/proyectos/aduamerica/compartidos/ kyoz@192.168.1.211:~/proyectos/compartidos/

# Simular la sincronización sin realizar cambios reales en disco (--dry-run)
rsync -avz --dry-run -e 'ssh -p 27' --exclude 'node_modules/' --exclude '.git/' ~/proyectos/aduamerica/compartidos/ kyoz@192.168.1.211:~/proyectos/compartidos/
```

---

## 2. Configuración y Control del Servicio Systemd

```bash
# Crear o editar el archivo de configuración del servicio en Systemd
sudo nano /etc/systemd/system/compartidos.service

# Recargar el daemon para registrar cambios en los archivos de servicio
sudo systemctl daemon-reload

# Habilitar el inicio automático al reiniciar el servidor y arrancar inmediatamente
sudo systemctl enable --now compartidos

# Reiniciar el servicio tras compilar una nueva versión
sudo systemctl restart compartidos

# Ver el estado y consumo de recursos en tiempo real del servicio
sudo systemctl status compartidos

# Detener el servicio inmediatamente
sudo systemctl stop compartidos

# Deshabilitar el arranque automático
sudo systemctl disable compartidos

# Limpiar cualquier estado residual de fallos en memoria
sudo systemctl reset-failed
```

### Configuración de Referencia para el archivo `.service`

> [!IMPORTANT]
> **Campos que debes actualizar según tu proyecto:**
>
> - **`Description`**: Nombre legible del servicio (ej. `mi-api REST Service`).
> - **`User`**: Usuario de Linux que ejecutará el proceso (ej. `kyoz`).
> - **`WorkingDirectory`**: Ruta absoluta hacia la raíz de tu proyecto en el servidor (ej. `/home/kyoz/proyectos/mi-api`).
> - **`ExecStart`**: Ruta absoluta hacia el binario de Node (verifícalo con `which node`) junto al archivo ejecutable y la bandera `--env-file=.env`.
> - **`SyslogIdentifier`**: Identificador único que se usará para consultar los logs en Journalctl (`journalctl -u tu-identificador -f`).

```ini
[Unit]
Description=compartidos REST Service
After=network.target

[Service]
Type=simple
User=kyoz
WorkingDirectory=/home/kyoz/proyectos/compartidos
ExecStart=/home/kyoz/.local/share/pnpm/bin/node --env-file=.env dist/index.js
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
SyslogIdentifier=compartidos

[Install]
WantedBy=multi-user.target
```

---

## 3. Análisis y Filtrado de Logs con Journalctl

```bash
# Seguir los logs en vivo y en tiempo real de la API
journalctl -u compartidos -f

# Ver las últimas 100 líneas combinadas sin paginador
journalctl -u compartidos -n 100 --no-pager

# Filtrar exclusivamente los mensajes de error (sin ruido informativo)
journalctl -u compartidos -p err --no-pager

# Ver logs de una ventana de tiempo específica (ejemplo: últimos 15 minutos)
journalctl -u compartidos --since "15 minutes ago"

# Ver únicamente los logs generados desde el último reinicio del servidor (-b)
journalctl -u compartidos -b

# Verificar el uso total de espacio en disco de los logs del sistema
journalctl --disk-usage
```

---

## 4. Diagnóstico de Salud y Liberación de Puertos

```bash
# Probar respuesta HTTP real del endpoint de salud
curl -I http://localhost:8220/health

# Comprobar que el puerto esté activo y en escucha en el sistema
ss -tulpn | grep 8220

# Listar todos los servicios creados en el proyecto dentro de Systemd
find /etc/systemd/system/ -maxdepth 1 -type f -name "*.service" -exec basename {} \;

# Matar el proceso y liberar el puerto TCP inmediatamente
sudo fuser -k 8220/tcp
```

---

## 5. Flujos de Trabajo Agrupados

```bash
# ==============================================================================
# Flujo: Despliegue de actualización en el servidor VPS
# ==============================================================================

# 1. En el VPS, entrar al directorio y compilar la nueva versión
cd ~/proyectos/compartidos && pnpm build

# 2. Reiniciar el servicio para aplicar los cambios del nuevo build
sudo systemctl restart compartidos

# 3. Comprobar que el servicio haya arrancado con éxito
sudo systemctl status compartidos

# 4. Probar la respuesta HTTP del endpoint de salud
curl -I http://localhost:8220/health
```
