# 🦭 Comandos de Podman y Podman Compose

> Referencia de comandos para la instalación, configuración rootless y linger, orquestación con Podman Compose y administración de contenedores, imágenes y volúmenes.

---

## 📑 Índice

- [1. Instalación](#1-instalación)
- [2. Configuración (Socket, Linger y containers.conf)](#2-configuración-socket-linger-y-containersconf)
- [3. Verificación de Funcionamiento](#3-verificación-de-funcionamiento)
- [4. Orquestación con Podman Compose](#4-orquestación-con-podman-compose)
- [5. Gestión de Contenedores](#5-gestión-de-contenedores)
- [6. Gestión de Imágenes](#6-gestión-de-imágenes)
- [7. Gestión de Volúmenes](#7-gestión-de-volúmenes)
- [8. Mantenimiento y Limpieza del Sistema](#8-mantenimiento-y-limpieza-del-sistema)

---

## 1. Instalación

```bash
# Instalar Podman y el plugin de Compose en Ubuntu / Debian
sudo apt install -y podman podman-compose
```

---

## 2. Configuración (Socket, Linger y containers.conf)

### Habilitar socket, reinicio automático y persistencia de sesión

```bash
# Habilitar e iniciar el socket de usuario de Podman
systemctl --user enable --now podman.socket

# Habilitar el servicio de reinicio automático de contenedores con reinicios del sistema
systemctl --user enable --now podman-restart.service

# Habilitar persistencia (linger) para que los contenedores sigan corriendo al cerrar la terminal
loginctl enable-linger $USER

# Verificar que la persistencia de sesión quedó aplicada (Salida esperada: Linger=yes)
loginctl show-user $USER | grep Linger
```

### Configuración del motor de Podman (containers.conf)

> [!TIP]
> Esta configuración previene errores de cgroups y registros de logs al ejecutar Podman en WSL2 o entornos con múltiples distribuciones.

```bash
# Crear la carpeta de configuración de usuario si no existe
mkdir -p ~/.config/containers

# Abrir el archivo de configuración de contenedores para editar
nano ~/.config/containers/containers.conf
```

**Contenido para `~/.config/containers/containers.conf`:**

```toml
[containers]
log_driver = "k8s-file"

[engine]
cgroup_manager = "cgroupfs"
events_logger = "file"
```

---

## 3. Verificación de Funcionamiento

```bash
# Ejecutar un contenedor de prueba que se autoelimina al terminar
podman run --rm docker.io/library/hello-world

# Comprobar la información del motor Podman y confirmar funcionamiento sin root
podman info
```

---

## 4. Orquestación con Podman Compose

```bash
# Levantar el stack multicontenedor en segundo plano
podman-compose up -d

# Levantar servicios forzando la reconstrucción previa de imágenes
podman-compose up -d --build

# Ver y seguir en tiempo real los registros de todo el stack
podman-compose logs -f

# Listar el estado de todos los servicios definidos en el compose
podman-compose ps

# Reiniciar únicamente un servicio específico (ejemplo: postgres)
podman-compose restart postgres

# Detener el stack manteniendo los datos persistentes de los volúmenes
podman-compose down

# Detener el stack y eliminar todo incluyendo los volúmenes de datos (-v)
podman-compose down -v
```

---

## 5. Gestión de Contenedores

```bash
# Listar todos los contenedores activos en ejecución
podman ps

# Listar todos los contenedores tanto activos como detenidos
podman ps -a

# Ver logs en tiempo real de un contenedor específico
podman logs -f <nombre_o_id>

# Entrar a una consola interactiva dentro de un contenedor en ejecución
podman exec -it <nombre_contenedor> sh

# Acceder a la consola de base de datos dentro de un contenedor PostgreSQL (ejemplo)
podman exec -it dev_pg psql -U usuario -d app_db

# Reiniciar un contenedor rápidamente si se traba
podman restart <nombre_contenedor>

# Iniciar un contenedor existente que estaba detenido
podman start <nombre_contenedor>

# Detener un contenedor en ejecución sin eliminarlo
podman stop <nombre_contenedor>

# Forzar la detención y eliminación de un contenedor individual
podman rm -f <nombre_contenedor>

# Forzar la eliminación de todos los contenedores existentes (activos o detenidos)
podman rm -a -f

# Ver consumo de CPU y RAM de los contenedores en tiempo real
podman stats --no-stream
```

---

## 6. Gestión de Imágenes

```bash
# Listar todas las imágenes descargadas localmente
podman images

# Descargar una imagen desde Docker Hub
podman pull docker.io/library/postgres:18-alpine

# Eliminar una imagen local específica
podman rmi <nombre_o_id_imagen>

# Eliminar únicamente las imágenes huérfanas sin etiqueta (<none> o dangling)
podman image prune

# Eliminar todas las imágenes que no estén asociadas a al menos un contenedor
podman image prune -a

# Forzar la eliminación de todas las imágenes huérfanas mediante consulta de IDs
podman rmi -f $(podman images -f "dangling=true" -q)
```

---

## 7. Gestión de Volúmenes

```bash
# Listar todos los volúmenes existentes
podman volume ls

# Listar únicamente los volúmenes huérfanos sin uso (dangling)
podman volume ls --filter dangling=true

# Crear un volumen nuevo con nombre
podman volume create datos_pg

# Ver detalles e inspeccionar la ruta física en disco de un volumen
podman volume inspect <nombre_volumen>

# Eliminar un volumen específico por nombre
podman volume rm <nombre_volumen>

# Eliminar todos los volúmenes que no estén asociados a ningún contenedor
podman volume prune
```

---

## 8. Mantenimiento y Limpieza del Sistema

```bash
# Limpiar imágenes no utilizadas, contenedores detenidos y caché
podman system prune -a

# Limpieza total profunda: borra contenedores detenidos, imágenes sin uso y volúmenes huérfanos
podman system prune -a --volumes
```
