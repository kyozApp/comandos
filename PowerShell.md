# Comandos para configurar y crear mi power shell

**Ir a:**

- [Inicio](./README.md)
- [Node.js](./NodeJS.md)
- [GitHub](./GitHub.md)
- [PostgreSQL](./PostgreSQL.md)
- [PM2](./PM2.md)
- [Caddy](./Caddy.md)


### Para crear un profile:

```bash
New-Item -Path $PROFILE -Type File -Force
```

### Para ingresar en mi profile creado:

```bash
notepad $PROFILE
```

### Para poder usar node, npm, python, etc sin profile:

```bash
docker run --rm -it -v ${PWD}:/app -w /app node:22-alpine sh
```

```bash
docker run --rm -it -v ${PWD}:/app -w /app python:latest sh
```

<br>

---

### Configuración básica dentro el profile:

- Para tener tu usuario personalizado:

    ```bash
    function prompt { "Pk > " }
    ```

- Para poder usar node y npm con docker:

    ```bash
    function npm {
        docker run --rm -v ${PWD}:/app -w /app node:22-alpine npm @args
    }
    
    function node {
        docker run --rm -v ${PWD}:/app -w /app node:22-alpine node @args
    }

    function docker-node {
        docker run --rm -it -v ${PWD}:/app -w /app node:22-alpine sh
    }
    ```

- Para poder usar python y pip con docker:

    ```bash
    function python {
        docker run --rm -v ${PWD}:/app -w /app python:latest python @args
    }

    function pip {
        docker run --rm -v ${PWD}:/app -w /app python:latest pip @args
    }

    function docker-python {
        docker run --rm -it -v ${PWD}:/app -w /app python:latest sh
    }
    ```

<br>

---
