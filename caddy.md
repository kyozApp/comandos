# 🔒 Comandos de Caddy (Servidor Web y Proxy Inverso)

> Referencia de comandos para la instalación oficial, control del servicio Systemd, pasos para desplegar proyectos y plantillas para proyectos estáticos y dinámicos (Node.js, SvelteKit, Hono).

---

## 📑 Índice

- [1. Instalación Oficial](#1-instalación-oficial)
- [2. Configuración Inicial del Servicio](#2-configuración-inicial-del-servicio)
- [3. Pasos para Desplegar un Proyecto](#3-pasos-para-desplegar-un-proyecto)
- [4. Comandos de Gestión y Diagnóstico](#4-comandos-de-gestión-y-diagnóstico)

---

## 1. Instalación Oficial

```bash
# Instalar dependencias de transporte seguro, llaveros y curl
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https curl

# Descargar y registrar la clave GPG oficial de Caddy
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg

# Agregar el repositorio oficial de Caddy a la lista de fuentes de APT
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list

# Asegurar permisos de lectura para que APT pueda leer las llaves
sudo chmod o+r /usr/share/keyrings/caddy-stable-archive-keyring.gpg
sudo chmod o+r /etc/apt/sources.list.d/caddy-stable.list

# Actualizar repositorios e instalar Caddy
sudo apt update && sudo apt install -y caddy
```

---

## 2. Configuración Inicial del Servicio

```bash
# Habilitar Caddy para que arranque automáticamente con el sistema y encenderlo
sudo systemctl enable --now caddy

# Ver el estado inicial del servicio para confirmar que está activo
sudo systemctl status caddy

# Ver la versión instalada de Caddy
caddy version
```

---

## 3. Pasos para Desplegar un Proyecto

### Paso 1: Comprobar DNS (Solo si usas dominio)

```bash
# Verificar que el dominio apunte a la IP pública de tu VPS antes de pedir SSL
nslookup api.tudominio.com
```

### Paso 2: Editar el Caddyfile

```bash
# Abrir el archivo principal de configuración
sudo nano /etc/caddy/Caddyfile
```

#### Modalidad A - Configuración por IP pública (sin dominio - HTTP)

```caddyfile
# 1. Proyecto Dinámico (API Node / PM2 en puerto interno)
http://200.58.103.96 {
    reverse_proxy localhost:3000
}

# 2. Proyecto Estático (Archivos web servidos desde carpeta en puerto alternativo)
http://200.58.103.96:8080 {
    root * /var/www/mi-sitio-estatico
    encode gzip zstd
    file_server
}
```

#### Modalidad B - Configuración con dominio público (con dominio - HTTPS automático)

```caddyfile
# 1. Proyecto Dinámico (API Hono / SvelteKit SSR con certificado SSL automático)
api.tudominio.com {
    encode gzip zstd
    reverse_proxy localhost:4000
}

# 2. Proyecto Estático (Frontend SPA Vite con enrutamiento y SSL automático)
app.tudominio.com {
    root * /var/www/dashboard/dist
    encode gzip zstd
    try_files {path} /index.html
    file_server
}
```

### Paso 3: Formatear el Caddyfile

```bash
# Formatear el archivo para mantener la indentación oficial limpia
sudo caddy fmt --overwrite /etc/caddy/Caddyfile
```

### Paso 4: Validar la sintaxis

```bash
# Validar que la sintaxis y directivas del Caddyfile sean 100% correctas
caddy validate --config /etc/caddy/Caddyfile
```

### Paso 5: Recargar en caliente

```bash
# Recargar Caddy en caliente sin interrumpir conexiones activas (Zero Downtime)
sudo systemctl reload caddy
```

### Paso 6: Verificar emisión de certificados y logs

```bash
# Inspeccionar las últimas líneas de logs para confirmar que el SSL o proxy arrancó bien
journalctl -u caddy -n 50 --no-pager
```

---

## 4. Comandos de Gestión y Diagnóstico

```bash
# Ver el estado en vivo del servicio Caddy
sudo systemctl status caddy

# Recargar la configuración sin interrumpir conexiones activas (Zero Downtime)
sudo systemctl reload caddy

# Reiniciar el servicio Caddy
sudo systemctl restart caddy

# Detener el servicio Caddy
sudo systemctl stop caddy

# Ver los logs en vivo de Caddy en tiempo real (seguimiento activo)
journalctl -u caddy -f --no-pager

# Ver las últimas 100 líneas de logs de Caddy sin paginador
journalctl -u caddy -n 100 --no-pager
```
