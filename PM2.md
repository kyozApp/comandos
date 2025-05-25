# Instalación de PM2 en VPS

**Ir a:**

- [Inicio](./README.md)
- [Node.js](./NodeJS.md)
- [GitHub](./GitHub.md)
- [PostgreSQL](./PostgreSQL.md)
- [Caddy](./Caddy.md)

### Instalar PM2 globalmente con pnpm:

```bash
pnpm add -g pm2
```

<br>

---

### Iniciar el Frontend (por ejemplo, Astro compilado):

Si tu archivo principal del frontend es `run-server.mjs`, ejecuta:

```bash
pm2 start run-server.mjs --name ui
```

### Iniciar el Backend (por ejemplo, Hono API):

Si ya hiciste `npm run build` o `pnpm build` y tu salida está en `dist/`, inicia así:

```bash
pm2 start dist/index.js --name api
```

<br>

---

### Guardar la configuración actual

Esto asegura que PM2 inicie automáticamente tus procesos al reiniciar el servidor:

```bash
pm2 save
```

<br>

---

### Ver los procesos activos

Lista todas las aplicaciones gestionadas por PM2:

```bash
pm2 list
```

<br>

---

### Ver logs en tiempo real

Muestra los logs del proceso frontend:

```bash
pm2 logs ui
```

Muestra los logs del proceso backend:

```bash
pm2 logs api
```

---

### Limpiar los logs

Limpiar todos los logs:

```bash
pm2 flush
```

Limpiar un log específico:

```bash
pm2 flush api
```

<br>

---

### Reiniciar una aplicación

Reinicia el frontend:

```bash
pm2 restart ui
```

Reinicia el backend:

```bash
pm2 restart api
```

<br>

---

### Detener una aplicación

Detiene temporalmente el frontend:

```bash
pm2 stop ui
```

Detiene temporalmente el backend:

```bash
pm2 stop api
```

<br>

---

### Eliminar un proceso

Elimina el proceso frontend:

```bash
pm2 delete ui
```

Elimina el proceso backend:

```bash
pm2 delete api
```

<br>

---
