# Instalación de Caddy en VPS

**Ir a:**

- [Inicio](./README.md)
- [PowerShell](./PowerShell.md)
- [Node.js](./NodeJS.md)
- [GitHub](./GitHub.md)
- [PostgreSQL](./PostgreSQL.md)
- [PM2](./PM2.md)

### Descargar e instalar Caddy desde repositorio oficial:

```bash
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https
```

```bash
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
```

```bash
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
```

```bash
sudo apt update
```

```bash
sudo apt install caddy
```

<br>

---

### Editar el archivo de configuración de Caddy

La configuración principal está en `/etc/caddy/Caddyfile`, edita el archivo con nano.

```bash
sudo nano /etc/caddy/Caddyfile
```

### Configuración básica por IP o dominio

- Si solo usas IP pública (sin dominio):

    ```caddyfile
    http://200.58.103.96 {
        reverse_proxy localhost:4321
    }
    ```

- Si usas IP pública (con dominio):

    ```caddyfile
    # Backend API
    https://api.kyozdev.site {
        reverse_proxy localhost:3005
    }

    # Dashboard Frontend
    https://dash.kyozdev.site {
        reverse_proxy localhost:4321
    }

    # Sitio Web Principal
    https://neuromedic.kyozdev.site {
        reverse_proxy localhost:4322
    }
    ```

<br>

---

### Si agregas dominio en Caddyfile puedes verificar si apunta bien:

```bash
nslookup api.kyozdev.site
```

### Para iniciar Caddy:

```bash
sudo systemctl start caddy
```

### Para ver el status:

```bash
sudo systemctl status caddy
```

### Para reiniciar:

```bash
sudo systemctl reload caddy
```

<br>

---