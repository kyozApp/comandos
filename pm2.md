# ⚙️ Comandos de PM2 (Gestor de Procesos en Segundo Plano)

> Referencia de comandos para la instalación, persistencia con Systemd/fnm, prueba rápida, ejecución de aplicaciones con Ecosystem File y gestión con PM2.

---

## 📑 Índice

- [1. Instalación](#1-instalación)
- [2. Configuración y Persistencia (Systemd y fnm)](#2-configuración-y-persistencia-systemd-y-fnm)
- [3. Prueba Rápida](#3-prueba-rápida)
- [4. Ejecución de Aplicaciones con Ecosystem File](#4-ejecución-de-aplicaciones-con-ecosystem-file)
- [5. Monitoreo y Logs](#5-monitoreo-y-logs)
- [6. Comandos de Gestión y Ciclo de Vida](#6-comandos-de-gestión-y-ciclo-de-vida)

---

## 1. Instalación

```bash
# Instalar PM2 de forma global con PNPM
pnpm add -g pm2

# Verificar la versión instalada de PM2
pm2 -v

# Instalar el módulo oficial para rotación automática de logs
pm2 install pm2-logrotate
```

---

## 2. Configuración y Persistencia (Systemd y fnm)

### Configurar rotación de logs (pm2-logrotate)

> [!TIP]
> Los logs rotarán automáticamente al alcanzar **50 MB** y se conservarán las últimas **10** rotaciones comprimidas en gzip para no saturar el almacenamiento.

```bash
# Definir límite de tamaño por archivo de log antes de rotar
pm2 set pm2-logrotate:max_size 50M

# Definir la cantidad de archivos rotados que se conservan
pm2 set pm2-logrotate:retain 10

# Comprimir automáticamente los logs antiguos en gzip
pm2 set pm2-logrotate:compress true
```

### Configurar inicio automático con el sistema (Systemd)

> [!CAUTION]
> **fnm crea rutas temporales de sesión** (`/run/user/1000/fnm_multishells/...`) que desaparecen al cerrar la terminal o reiniciar.
> Si ejecutas `pm2 startup` directamente, el servicio fallará con `status=127` (_"node: No such file or directory"_).
> Debes anteponer la **ruta estable** de fnm antes de generar el servicio.

```bash
# 1. Ver la versión exacta de Node instalada por fnm (ejemplo: v24.20.0)
ls ~/.local/share/fnm/node-versions/

# 2. Anteponer la ruta estable de Node al PATH de esta sesión (reemplaza vX.Y.Z)
export PATH="$HOME/.local/share/fnm/node-versions/vX.Y.Z/installation/bin:$PATH"

# 3. Confirmar que apunta a la ruta estable y NO a fnm_multishells
which node

# 4. Generar el servicio Systemd con la ruta estable activa en el PATH
pm2 startup
```

> [!IMPORTANT]
> `pm2 startup` imprime en pantalla un comando con `sudo` que **debes copiar y ejecutar manualmente**.

```bash
# 5. Confirmar que el servicio quedó con la ruta estable
cat /etc/systemd/system/pm2-kyoz.service | grep -i "node"

# 6. Congelar la lista base vacía y generar el archivo de persistencia
pm2 save --force
```

---

## 3. Prueba Rápida

```bash
# Crear script mínimo de prueba en Node.js
echo "setInterval(() => console.log('hello world'), 2000)" > hello.js

# Iniciar el proceso de prueba con PM2
pm2 start hello.js

# Ver los logs en tiempo real (presiona Ctrl + C para salir)
pm2 logs hello

# Detener y eliminar el proceso de prueba
pm2 delete hello

# Eliminar el archivo temporal de prueba
rm hello.js
```

---

## 4. Ejecución de Aplicaciones con Ecosystem File

> [!TIP]
> Si tu proyecto usa `"type": "module"` en `package.json`, nombra el archivo como **`ecosystem.config.cjs`** para evitar fallos de sintaxis CommonJS en Node moderno.

### Generar la plantilla base

```bash
# Crear el archivo de configuración en la raíz del proyecto
pm2 ecosystem
```

### Plantilla recomendada para producción (`ecosystem.config.cjs`)

```javascript
module.exports = {
  apps: [
    {
      name: "mi-api",
      script: "./dist/index.js",
      exec_mode: "cluster",
      instances: "max",
      autorestart: true,
      max_memory_restart: "1G",
      exp_backoff_restart_delay: 100,
      watch: false,
      merge_logs: true,
      log_date_format: "YYYY-MM-DD HH:mm:ss",
      env_production: {
        NODE_ENV: "production",
        PORT: 3000,
      },
    },
  ],
};
```

### Comandos de control con Ecosystem

```bash
# Iniciar todas las aplicaciones declaradas en el archivo para producción
pm2 start ecosystem.config.cjs --env production

# Recargar en caliente aplicando cambios de configuración o variables de entorno
pm2 reload ecosystem.config.cjs --env production --update-env

# Reiniciar todas las aplicaciones del ecosystem
pm2 restart ecosystem.config.cjs

# Detener todas las aplicaciones del ecosystem
pm2 stop ecosystem.config.cjs
```

---

## 5. Monitoreo y Logs

```bash
# Ver los logs en vivo de todos los procesos en tiempo real
pm2 logs

# Ver los logs de un proceso específico mostrando las últimas 100 líneas
pm2 logs mi-api --lines 100

# Abrir el panel interactivo visual en terminal (CPU, RAM y Event Loop)
pm2 monit

# Vaciar y limpiar todos los archivos de log acumulados
pm2 flush
```

---

## 6. Comandos de Gestión y Ciclo de Vida

```bash
# Listar todos los procesos activos con su PID, memoria, CPU y estado
pm2 list

# Detener un proceso sin eliminarlo de la lista
pm2 stop mi-api

# Detener todos los procesos activos
pm2 stop all

# Reiniciar un proceso
pm2 restart mi-api

# Recargar proceso en clúster sin caída de servicio (Zero-Downtime Reload)
pm2 reload mi-api

# Eliminar un proceso de la lista de PM2
pm2 delete mi-api

# Eliminar todos los procesos de la lista
pm2 delete all

# Guardar la lista actual de procesos para que se restauren tras reiniciar
pm2 save
```
