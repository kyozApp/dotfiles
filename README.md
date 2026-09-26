# dotfiles · Windows + WSL2 Ubuntu 26

> Aprovisionamiento completo de un equipo de desarrollo desde cero.
> Terminal GPU · Linux moderno · Contenedores rootless · Editor de código.

---

## 📑 Índice

|  #  | Paso                                                                        | Dónde           |
| :-: | :-------------------------------------------------------------------------- | :-------------- |
|  0  | [Instalar y configurar Alacritty](#️-paso-0-instalar-y-configurar-alacritty) | Windows         |
|  1  | [Plataforma WSL2 y red](#️-paso-1-plataforma-wsl2-y-red)                     | Windows         |
|  2  | [Ubuntu 26.04](#-paso-2-ubuntu-2604-en-wsl2)                                | Windows → Linux |
|  3  | [GitHub, Git y dotfiles](#-paso-3-github-git-y-dotfiles)                    | Linux           |
|  4  | [pnpm y Node.js](#-paso-4-pnpm-y-nodejs)                                    | Linux           |
|  5  | [Podman rootless](#-paso-5-podman-rootless)                                 | Linux           |
|  6  | [Servicios locales](#️-paso-6-servicios-de-desarrollo-local)                 | Linux           |
|  7  | [Instalar y configurar Zed](#-paso-7-instalar-y-configurar-zed)             | Windows         |

---

## 🖥️ Paso 0: Instalar y Configurar Alacritty

Lo primero en un equipo nuevo: preparar tu terminal antes de tocar WSL.
Todo se hace directamente desde Windows sin abrir ninguna consola todavía.

### Instalar Alacritty

Descarga e instala el ejecutable oficial desde [alacritty.org](https://alacritty.org).

### Instalar la fuente tipográfica

Descarga **0xProto Nerd Font** desde [nerdfonts.com/font-downloads](https://www.nerdfonts.com/font-downloads),
descomprime el `.zip` y selecciona los 3 archivos de la variante estándar:

- `0xProtoNerdFont-Regular.ttf`
- `0xProtoNerdFont-Bold.ttf`
- `0xProtoNerdFont-Italic.ttf`

Haz clic derecho sobre los 3 seleccionados → **Instalar** (o **Instalar para todos los usuarios**).

### Configurar Alacritty

Presiona `Win + R`, escribe `%APPDATA%` y crea la carpeta `alacritty` si no existe.
Dentro crea el archivo `alacritty.toml`, abre [`alacritty/alacritty.toml`](alacritty/alacritty.toml)
desde GitHub (botón **Copy raw file**) y pega el contenido. Guarda y cierra.

Abre **Alacritty**: arrancará con PowerShell, fuente 0xProto Nerd Font y paleta
Catppuccin Mocha lista. Todos los pasos siguientes se ejecutan desde aquí.

---

## ⚙️ Paso 1: Plataforma WSL2 y Red

### Habilitar WSL2

Abre **Alacritty como Administrador** (clic derecho → _Ejecutar como administrador_):

```powershell
wsl --install --no-distribution
```

```powershell
Restart-Computer
```

### Configurar red mirrored

Al volver del reinicio:

1. Presiona `Win + R`, escribe `notepad %USERPROFILE%\.wslconfig` y pulsa `Enter`.
2. Si te pregunta si deseas crear el archivo, selecciona **Sí**.
3. Abre [`windows/.wslconfig`](windows/.wslconfig) de este repositorio, copia su contenido
   y pégalo en el Bloc de notas.
4. Guarda los cambios y cierra el archivo.

> [!NOTE]
> `networkingMode=mirrored` comparte automáticamente todos los puertos de WSL
> con Windows (`localhost:5173`, `localhost:5432`, etc.) sin reglas de firewall.

---

## 🐧 Paso 2: Ubuntu 26.04 en WSL2

### Instalar la distribución

```powershell
wsl --install -d Ubuntu-26.04 --name <nombre-instancia>
```

Al primer inicio configura tu nombre de usuario y contraseña de Linux.

### Verificar instalación

```powershell
wsl -l -v
```

La salida debe mostrar tu distribución en `Version 2` y estado `Running` o `Stopped`.

### Actualizar el sistema e instalar utilidades base

Entra a tu distribución:

```powershell
wsl -d <nombre-instancia> ~
```

Actualiza los paquetes del sistema:

```bash
sudo apt update && sudo apt upgrade -y
```

Instala las utilidades esenciales:

```bash
sudo apt install -y curl unzip build-essential
```

### Configurar navegador para CLI

Para que herramientas como `gh auth login` puedan abrir el navegador de Windows
automáticamente, agrega esta variable al final de `~/.bashrc`:

```bash
nano ~/.bashrc
```

```bash
# Navegador de Windows desde WSL
export BROWSER="cmd.exe /c start"
```

Guarda con `Ctrl + O` → `Enter` → `Ctrl + X`, luego recarga la configuración:

```bash
source ~/.bashrc
```

---

## 🐙 Paso 3: GitHub, Git y dotfiles

### Instalar Git y GitHub CLI

```bash
sudo apt install git gh -y
```

### Autenticar con GitHub

```bash
gh auth login
```

> [!NOTE]
> Selecciona: **GitHub.com** → **HTTPS** → **Login with a web browser**.
> El navegador de Windows se abre automáticamente gracias a la variable `BROWSER`.

### Verificar autenticación

```bash
gh auth status
```

### Configurar identidad global

```bash
git config --global user.name "<tu-usuario>"
git config --global user.email "<tu-correo@ejemplo.com>"
git config --global core.autocrlf input
git config --global init.defaultBranch main
```

> [!TIP]
> En `user.email` puedes usar tu correo habitual o el correo anónimo de GitHub
> (`ID+usuario@users.noreply.github.com` disponible en **Settings** → **Emails**)
> para evitar que tu dirección real aparezca en los commits públicos.

### Clonar este repositorio

```bash
mkdir -p ~/proyectos
cd ~/proyectos
gh repo clone kyozApp/dotfiles
```

---

## ⚡ Paso 4: pnpm y Node.js

### Instalar pnpm

```bash
curl -fsSL https://get.pnpm.io/install.sh | sh -
source ~/.bashrc
```

### Instalar Node.js LTS

```bash
pnpm runtime set node lts -g
```

### Verificar versiones

```bash
node -v
pnpm -v
```

---

## 🐳 Paso 5: Podman Rootless

### Instalar Podman y Podman Compose

```bash
sudo apt install -y podman podman-compose
```

### Habilitar servicios y persistencia

```bash
systemctl --user enable --now podman.socket
systemctl --user enable --now podman-restart.service
loginctl enable-linger $USER
```

Verifica que la persistencia `linger` esté activa:

```bash
loginctl show-user $USER | grep Linger
# Linger=yes
```

### Probar instalación

```bash
podman run --rm docker.io/library/hello-world
```

Elimina la imagen de prueba una vez verificada la instalación:

```bash
podman rmi docker.io/library/hello-world
```

---

## 🗄️ Paso 6: Servicios de Desarrollo Local

Suite de servicios compartidos con Podman Compose para todos tus proyectos locales.

### Servicios incluidos

| Servicio    | Imagen               |    Puerto     | Propósito                          |
| :---------- | :------------------- | :-----------: | :--------------------------------- |
| `dev_pg`    | `postgres:18-alpine` |    `5432`     | PostgreSQL 18                      |
| `dbgate`    | `dbgate:7.3.1`       |    `8080`     | Panel web moderno de base de datos |
| `gotenberg` | `gotenberg:8.36.0`   |    `3000`     | Generador de PDFs                  |
| `mailpit`   | `mailpit:v1.30`      | `1025` `8025` | SMTP local + webmail de pruebas    |

> [!TIP]
> En equipos con recursos muy limitados (< 8 GB RAM), puedes reemplazar `dbgate`
> por `adminer` (`docker.io/library/adminer:5.5.1-standalone`, puerto `8080:8080`)
> en `compose.yaml` para reducir el consumo a menos de 30 MB de RAM.

### Desplegar servicios

Copia la carpeta de servicios desde el repositorio clonado:

```bash
cp -r ~/proyectos/dotfiles/services ~/proyectos/services
cd ~/proyectos/services
```

> [!WARNING]
> Si es la primera vez o un despliegue previo falló, limpia redes residuales antes:
>
> ```bash
> podman network prune -f
> ```

Inicia los servicios en segundo plano:

```bash
podman-compose up -d
podman ps
```

---

## 🟦 Paso 7: Instalar y Configurar Zed

Ahora que el entorno de desarrollo y los servicios están listos, instala el editor.

### Instalar Zed

Descarga e instala el ejecutable oficial desde [zed.dev/download](https://zed.dev/download).

### Configurar Zed

Abre Zed en Windows, ve a **Menu** → **Open Settings File** (o presiona `Ctrl + ,`),
abre [`zed/settings.json`](zed/settings.json) de este repositorio, copia todo su contenido
y pégalo reemplazando el archivo. Guarda con `Ctrl + S`.

---

## ✅ Verificación Final

Abre tu navegador en Windows:

| URL                                                   | Servicio  | Credenciales                                                      |
| :---------------------------------------------------- | :-------- | :---------------------------------------------------------------- |
| [localhost:8080](http://localhost:8080)               | DbGate    | Servidor: `db` · Usuario: `dev_user` · Contraseña: `dev_password` |
| [localhost:8025](http://localhost:8025)               | Mailpit   | —                                                                 |
| [localhost:3000/health](http://localhost:3000/health) | Gotenberg | —                                                                 |

### Conectar Zed a tu entorno WSL

1. Abre **Zed** y presiona `F1`.
2. Escribe `open wsl` y presiona `Enter`.
3. Selecciona tu distribución WSL de la lista.
4. Haz clic en **Open Folder**: verás la ruta `/home/<usuario>`.
5. Agrega `/proyectos` al final para que quede `/home/<usuario>/proyectos` y pulsa `Enter`.

> Tu entorno está 100% aislado en Linux, con red transparente hacia Windows,
> listo para clonar y desarrollar. 🚀
