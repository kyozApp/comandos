# Comandos NodeJS

**Ir a:**

- [Inicio](./README.md)
- [PowerShell](./PowerShell.md)
- [GitHub](./GitHub.md)
- [PostgreSQL](./PostgreSQL.md)
- [PM2](./PM2.md)
- [Caddy](./Caddy.md)

## Indice

- [Instalación en Windows](#instalación-en-windows)
- [Instalación en Linux](#instalación-en-linux)

---

## <a id="instalación-en-windows"></a>**Instalación en Windows**

<details>
<summary>Haz clic para ver</summary>

### Descarga Schniz:

```powershell
winget install Schniz.fnm
```

### Reinicia el terminal y instala Node.js:

```powershell
fnm install 22
```

### Abre el profile:

```powershell
notepad $PROFILE
```

### Si no tienes un profile créalo:

```powershell
New-Item -Path $PROFILE -Type File -Force
```

### Agrega en el profile para la configuración automática:

```powershell
fnm env --use-on-cd | Out-String | Invoke-Expression
```

### Reinicia y verifica q se instalo:

```powershell
node -v
```

```powershell
npm -v
```

```powershell
fnm list
```

### Descarga y instala PNPM:

```powershell
corepack enable pnpm
```

### Reinicia y verifica que se instalo:

```powershell
pnpm -v
```

<br>

## Opcional

### Encuentra la ubicación de `fnm.exe` y agrégalo al path:

Agregar al path la ruta que aparece cuando ejecutas el comando:

```powershell
Get-Command fnm
```

### Verifica si instalo Node:

```powershell
node -v
```

### Verifica si instalo PNPM:

```powershell
pnpm -v
```

</details>

<br>

---

## <a id="instalación-en-linux"></a>**Instalación en Linux**

<details>
<summary>Haz clic para ver</summary>

### Instalar FNM:

```powershell
curl -o- https://fnm.vercel.app/install | bash
```

```powershell
ls /root/.local/share/fnm
```

```powershell
source /root/.bashrc
```

```powershell
fnm --version
```

###  Instalar Node v22

```powershell
fnm install 22
```

### Instalar PNPM:

```powershell
corepack enable pnpm
```

### Configurar entorno para PNPM:

```powershell
pnpm setup
```

### Recargar la configuración de Bash:

```powershell
source ~/.bashrc
```

### Verifica la instalación de Node.js

```powershell
node -v
```

### Verifica la instalación de PNPM

```powershell
pnpm -v
```

</details>

<br>

---