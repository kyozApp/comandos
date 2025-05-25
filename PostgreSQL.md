# Comandos PostgreSQL en VPS

**Ir a:**

- [Inicio](./README.md)
- [Node.js](./NodeJS.md)
- [GitHub](./GitHub.md)
- [PM2](./PM2.md)
- [Caddy](./Caddy.md)

## Indice

- [Comandos básicos](#comandos)
- [Instalación de PostgreSQL en VPS](#instalación)
- [Acceder al usuario](#acceder-al-usuario)

---

## <a id="comandos"></a>**Comandos básicos**

<details>
<summary>Haz clic para ver</summary>

### Listar tablas:

```postgresql
\dt
```

### Eliminar tablas:

```postgresql
DROP TABLE nombre_tabla;
```

### Exportar una base de datos:

```postgresql
pg_dump -h localhost -U neuromedic -d neuromedicdb -Fc -f neuromedicdb_backup.backup
```


### Importar una base de datos:

```postgresql
pg_restore -h localhost -U neuromedic -d neuromedicdb --verbose --clean --no-owner neuromedicdb_backup_20250503.backup
```

### Exportar una tabla:

```postgresql
pg_dump -h localhost -U usuario -d neuromedicdb -t nombre_tabla -Fc -f tabla_backup.backup
```

### Importar una tabla:

```postgresql
pg_restore -h localhost -U usuario -d neuromedicdb --data-only -t nombre_tabla tabla_backup.backup
```

</details>

<br>

---

## <a id="instalación"></a>**Instalación de PostgreSQL en VPS**

<details>
<summary>Haz clic para ver</summary>

### Instalar PostgreSQL:

```bash
https://www.postgresql.org/download/linux/ubuntu/
```

## Opcional


### Verificar el status:

```bash
sudo systemctl status postgresql
```

### Habilitar para que inicie automáticamente:

```bash
sudo systemctl enable postgresql
```

### Volver a verificar el status:

```bash
sudo systemctl status postgresql
```

### Acceder al directorio de configuración de PostgreSQL:

```bash
ll /etc/postgresql/17/main/
```

### Cambiar los permisos de los archivos de configuración:

```bash
sudo chmod 600 /etc/postgresql/17/main/postgresql.conf
sudo chmod 600 /etc/postgresql/17/main/pg_hba.conf
```

### Editar el archivo de configuración:

```bash
sudo nano /etc/postgresql/17/main/postgresql.conf
```

- Presiona `ctrl + w`, escribe `localhost` y descomenta (quita el `#` de) la línea:

  ```postgresql
  listen_addresses = 'localhost'
  ```

- Luego, presiona `ctrl + w`, escribe `sha-256` y descomenta (quita el `#` de) la línea:
  
  ```postgresql
  password_encryption = scram-sha-256
  ```

- Guarda y sal con `ctrl + x`, luego presiona `y` y `enter`.

### Editar el archivo `pg_hba.conf`:

```bash
sudo nano /etc/postgresql/17/main/pg_hba.conf
```

- Navega hasta el final donde están los accesos locales. Cambia debajo de 'local':
  
  ```
  local   all all     scram-sha-256
  ```

- Agrega debajo de 'ipv6', después del primer `local`:
  
  ```
  local   all all all reject
  ```

- Guarda y sal con `ctrl + x`, luego presiona `y` y `enter`.

### Recargar la configuración de PostgreSQL:

```bash
sudo systemctl reload postgresql
```

</details>

<br>

---

## <a id="acceder-al-usuario"></a>**Acceder al usuario**

<details>
<summary>Haz clic para ver</summary>

### Acceder al usuario postgres:

```bash
sudo -i -u postgres
```

- En el prompt de PostgreSQL, escribe:

  ```sql
  psql
  ```

- Luego, ejecuta los siguientes comandos SQL:

  Cambia el password:

  ```sql
  ALTER USER postgres WITH PASSWORD '5YgS%Xz$4!Z5n$';
  ```

  Crea el usuario:

  ```sql
  CREATE USER prestamo WITH PASSWORD 'nsu9@K5^1e6kjHCM$hN';
  ```

  Crea la base de datos:
  
  ```sql
  CREATE DATABASE prestamodb OWNER prestamo;
  ```

  Muestra las bases de datos:

  ```sql
  \l
  ```

  Sal de la muestra:

  ```sql
  q
  ```

  Muestra los usuarios:

  ```sql
  \du
  ```

  Otorga privilegios al usuario:

  ```sql
  GRANT ALL PRIVILEGES ON DATABASE prestamodb TO prestamo;
  ```

  Sal de la sesión:

  ```sql
  \q
  ```

  Sal de la sesión de `postgres`:

  ```sql
  exit
  ```

### Conectar a la base de datos prestamodb:

```bash
psql -h localhost -U prestamo -d prestamodb
```

  - Ingresa la contraseña: `nsu9@K5^1e6kjHCM$hN`

  - Luego ejecuta:

  Muestra las bases de datos:

  ```sql
  \l
  ```

  Sal de la muestra:

  ```sql
  q
  ```

  Sal de la sesión de `prestamo`:

  ```sql
  \q
  ```

</details>

<br>

---
