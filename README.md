# 🛠️ Comandos y manuales técnicos

Estos son los comandos que uso habitualmente:

**Ir a:**

- [Node.js](./NodeJS.md)
- [GitHub](./GitHub.md)
- [PostgreSQL](./PostgreSQL.md)
- [PM2](./PM2.md)
- [Caddy](./Caddy.md)

---

## Índice

- [Instalación en Windows](#instalación-en-windows)
- [Instalación en Linux](#instalación-en-linux)


<br>

---

## <a id="instalación-en-windows"></a>**Instalación en Windows**

<details>
<summary>Haz clic para ver</summary>

### Si tenías una clave SSH guardada y has formateado el VPS, elimina la clave antigua para evitar problemas de conexión. Ejecuta el siguiente comando:

```powershell
Remove-Item C:\Users\kyozd\.ssh\known_hosts # (rm C:\Users\kyozd\.ssh\known_hosts)
```

### Para limpiar mi terminal:

```powershell
cls
```

### Para abrir el explorador de archivos:

```powershell
explorer .
```

### Para crear una carpeta:

```powershell
mkdir nombre_carpeta
```

### Para borrar una carpeta:

```powershell
rmdir nombre_carpeta
```

### Para forzar la eliminación de una carpeta:

```powershell
rmdir nombre_carpeta -recurse
```

</details>

<br>

---

## <a id="instalación-en-linux"></a>**Instalación en Linux**

<details>
<summary>Haz clic para ver</summary>

## Paquetes

### Actualizar la lista de paquetes disponibles:

```powershell
sudo apt update
```

### Actualizar los paquetes instalados:

```powershell
sudo apt upgrade
```

### Instalar un paquete:

```powershell
sudo apt install nombre_paquete
```

### Eliminar un paquete:

```powershell
sudo apt remove nombre_paquete
```

### Limpiar la caché de paquetes:

```powershell
sudo apt clean
```

### Limpiar la caché de paquetes y eliminar archivos no utilizados:

```powershell
sudo apt autoclean
```

<br>

## Carpetas

### Crear la carpeta neuromedic-api:

```powershell
mkdir comandos
```

### Crear la ruta de la carpeta:

```powershell
mkdir -p /var/www/html/
```

### Navegar hasta la ruta:

```powershell
cd /var/www/html/
```

### Eliminar la carpeta neuromedic-api y todo su contenido:

```powershell
rm -rf /var/www/html/comandos
```

<br>

## Archivos

### Copiar el archivo .env.plantilla como .env:

```powershell
cp .env.plantilla .env
```

### Editar el archivo .env con nano:

```powershell
nano .env
```

### Atajos dentro de nano:
- `Ctrl + W`: Buscar texto  
- `Ctrl + O`: Guardar cambios (WriteOut)  
- `Enter`: Confirmar nombre del archivo al guardar  
- `Ctrl + X`: Salir del editor  

<br>

</details>

<br>

---
