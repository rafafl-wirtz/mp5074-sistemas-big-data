# Manual de WSL (Windows Subsystem for Linux)

**Módulo:** MP5074 – Sistemas de Big Data · Curso 2026-27

> WSL permite ejecutar un sistema Linux real (Ubuntu, Debian…) dentro de Windows, sin máquina virtual tradicional ni arranque dual. En este módulo lo usaremos como entorno de trabajo para las herramientas de Big Data (Python, Java, Docker, Hadoop, Spark…), que están pensadas para Linux.

---

## Índice

1. [¿Qué es WSL?](#1-qué-es-wsl)
2. [Requisitos previos](#2-requisitos-previos)
3. [Instalación](#3-instalación)
4. [Primer arranque: usuario y contraseña](#4-primer-arranque-usuario-y-contraseña)
5. [Actualizar el sistema](#5-actualizar-el-sistema)
6. [Comandos básicos de `wsl` (desde PowerShell)](#6-comandos-básicos-de-wsl-desde-powershell)
7. [Sistema de archivos: Windows ↔ Linux](#7-sistema-de-archivos-windows--linux)
8. [Interoperabilidad entre Windows y Linux](#8-interoperabilidad-entre-windows-y-linux)
9. [Herramientas recomendadas: Windows Terminal y VS Code](#9-herramientas-recomendadas-windows-terminal-y-vs-code)
10. [Configuración avanzada: `.wslconfig` y `wsl.conf`](#10-configuración-avanzada-wslconfig-y-wslconf)
11. [Preparar el entorno para Big Data](#11-preparar-el-entorno-para-big-data)
12. [Copias de seguridad: exportar e importar](#12-copias-de-seguridad-exportar-e-importar)
13. [Solución de problemas frecuentes](#13-solución-de-problemas-frecuentes)
14. [Chuleta rápida](#14-chuleta-rápida)
15. [Referencias](#15-referencias)

---

## 1. ¿Qué es WSL?

**WSL** (*Windows Subsystem for Linux*) es una característica de Windows que permite ejecutar distribuciones GNU/Linux directamente en Windows.

Hay dos versiones:

| Característica | WSL 1 | WSL 2 (recomendada) |
|---|---|---|
| Arquitectura | Traduce llamadas de Linux a Windows | Kernel Linux real en una VM ligera (Hyper-V) |
| Compatibilidad con llamadas al sistema | Parcial | Completa |
| Docker, systemd | No | Sí |
| Rendimiento con archivos dentro de Linux | Medio | Alto |
| Rendimiento con archivos en `C:\` | Alto | Más bajo |

`wsl --install` instala **WSL 2 por defecto**, que es la que usaremos en el módulo.

**Conceptos clave**

- **Distribución (distro):** el Linux concreto que se instala (Ubuntu, Debian, openSUSE…). Se pueden tener varias a la vez.
- **Kernel de WSL:** kernel Linux mantenido por Microsoft que se actualiza con `wsl --update`.
- **Disco virtual (`ext4.vhdx`):** archivo de Windows donde se guarda todo el sistema de archivos de cada distro.

---

## 2. Requisitos previos

- **Windows 11**, o **Windows 10 versión 2004 (compilación 19041) o superior**.
  - Comprobarlo: `Win + R` → `winver`.
- **Virtualización activada en la BIOS/UEFI** (Intel VT-x / AMD-V). Se puede comprobar en *Administrador de tareas → Rendimiento → CPU → Virtualización: Habilitado*.
- Permisos de **administrador** para la instalación.
- Recomendable: **16 GB de RAM** o más para trabajar con herramientas de Big Data (8 GB es el mínimo práctico).

---

## 3. Instalación

### 3.1 Instalación estándar (recomendada)

1. Abre **PowerShell como administrador** (clic derecho sobre el menú Inicio → *Terminal (administrador)*).
2. Ejecuta:

   ```powershell
   wsl --install
   ```

   Esto activa las características necesarias, descarga el kernel Linux, establece WSL 2 como versión por defecto e instala **Ubuntu**.
3. **Reinicia** el equipo.
4. Tras el reinicio, se abrirá Ubuntu automáticamente (o ábrelo desde el menú Inicio) para terminar la instalación.

### 3.2 Instalar otra distribución

```powershell
wsl --list --online          # Ver las distribuciones disponibles (abreviado: wsl -l -o)
wsl --install -d Debian      # Instalar una distribución concreta
wsl --install -d Ubuntu-24.04
```

### 3.3 Opciones útiles de instalación

| Comando | Para qué sirve |
|---|---|
| `wsl --install --no-distribution` | Instala WSL sin ninguna distribución |
| `wsl --install --web-download -d <Distro>` | Descarga desde GitHub en vez de la Microsoft Store (útil si la instalación se queda en 0 %) |
| `wsl --set-default-version 2` | Fija WSL 2 para las nuevas distribuciones |
| `wsl --set-version <Distro> 2` | Convierte una distribución existente a WSL 2 |

### 3.4 Instalación sin conexión

Si no hay acceso a la Microsoft Store: descarga el instalador `.msi` desde las [versiones de WSL en GitHub](https://github.com/microsoft/wsl/releases), activa la plataforma de máquina virtual y reinicia:

```powershell
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

### 3.5 Verificar la instalación

```powershell
wsl --version       # Versión de WSL, kernel, Windows…
wsl --status        # Distribución y versión por defecto
wsl -l -v           # Distribuciones instaladas, estado y versión (1 o 2)
```

Salida esperada de `wsl -l -v`:

```
  NAME      STATE           VERSION
* Ubuntu    Running         2
```

El asterisco (`*`) marca la distribución por defecto.

---

## 4. Primer arranque: usuario y contraseña

La primera vez que se abre la distribución se pide:

- **Nombre de usuario de Linux:** en minúsculas, sin espacios (p. ej., `alumno`).
- **Contraseña:** mientras se escribe **no aparece nada en pantalla**; es normal.

Ten en cuenta que:

- Este usuario y contraseña son **independientes** del usuario de Windows.
- Son **propios de cada distribución**: cada distro que instales tendrá los suyos.
- El usuario creado es administrador y puede usar `sudo`.

**Si olvidas la contraseña** (desde PowerShell):

```powershell
wsl -u root              # Entra como root sin contraseña
```
```bash
passwd <usuario>         # Define la nueva contraseña
exit
```

**Cambiar el usuario por defecto de una distro:**

```powershell
ubuntu config --default-user <usuario>
```

---

## 5. Actualizar el sistema

Las distribuciones **no se actualizan solas**. Hazlo nada más instalar y de forma periódica:

```bash
sudo apt update && sudo apt upgrade -y
```

Para actualizar el propio WSL (kernel y componentes), desde PowerShell:

```powershell
wsl --update
```

---

## 6. Comandos básicos de `wsl` (desde PowerShell)

> Estos comandos se ejecutan desde **PowerShell o CMD**. Si estás dentro de Linux y quieres llamarlos, usa `wsl.exe` en lugar de `wsl`.

### Arrancar y entrar

| Comando | Descripción |
|---|---|
| `wsl` | Abre la distribución por defecto |
| `wsl ~` | Abre la distribución directamente en el directorio *home* del usuario |
| `wsl -d Debian` | Abre una distribución concreta |
| `wsl -u root` | Entra como otro usuario (aquí, root) |
| `wsl -d Ubuntu -u alumno` | Distribución y usuario concretos |

### Listar y gestionar

| Comando | Descripción |
|---|---|
| `wsl -l -v` | Lista las distros instaladas con estado y versión |
| `wsl -l --running` | Solo las que están en ejecución |
| `wsl --set-default Debian` | Cambia la distribución por defecto |
| `wsl --terminate Ubuntu` | Detiene una distribución |
| `wsl --shutdown` | Detiene **todas** las distros y la VM de WSL 2 (libera la RAM) |
| `wsl --unregister Ubuntu` | ⚠️ **Elimina** la distribución y todos sus datos |

### Información y ayuda

| Comando | Descripción |
|---|---|
| `wsl --version` | Versión de WSL y del kernel |
| `wsl --status` | Configuración general |
| `wsl --update` | Actualiza WSL |
| `wsl --help` | Ayuda con todos los comandos |

### Direcciones IP

```bash
hostname -I                                          # IP de la distro (dentro de Linux)
ip route show | grep -i default | awk '{print $3}'   # IP del host Windows vista desde Linux
```

---

## 7. Sistema de archivos: Windows ↔ Linux

### 7.1 Acceder a Windows desde Linux

Las unidades de Windows están montadas en `/mnt`:

```bash
cd /mnt/c/Users/<UsuarioWindows>/Documents
ls /mnt/d
```

### 7.2 Acceder a Linux desde Windows

En el Explorador de archivos, escribe en la barra de direcciones:

```
\\wsl$\Ubuntu\home\<usuario>
```

También aparece la entrada **Linux** en el panel lateral del Explorador. Desde Linux puedes abrir la carpeta actual en el Explorador con:

```bash
explorer.exe .
```

### 7.3 ¿Dónde guardar los proyectos? ⚠️ Importante

> **Regla de oro:** guarda los archivos en el mismo sistema operativo que las herramientas que los usan.

- Si trabajas con herramientas de Linux (Python, Spark, Hadoop, Docker…) → guarda el proyecto **dentro de Linux**: `~/proyectos/...`
- Evita trabajar desde Linux sobre `/mnt/c/...`: el acceso entre sistemas en WSL 2 es **mucho más lento**, algo que se nota mucho con datasets grandes.

Estructura recomendada para el módulo:

```bash
mkdir -p ~/bigdata/{datos,notebooks,scripts,proyectos}
```

### 7.4 Permisos y finales de línea

- Los archivos creados desde Windows pueden perder los permisos de ejecución. Si un script no se ejecuta: `chmod +x script.sh`.
- Windows usa finales de línea `CRLF` y Linux `LF`. Un script `.sh` editado en Windows puede dar el error `/bin/bash^M: bad interpreter`. Solución:

  ```bash
  sudo apt install dos2unix
  dos2unix script.sh
  ```

---

## 8. Interoperabilidad entre Windows y Linux

Desde **PowerShell** se pueden lanzar comandos de Linux:

```powershell
wsl ls -la
wsl ls -la | findstr "git"      # Combinar Linux y Windows
```

Desde **Linux** se pueden lanzar programas de Windows (añadiendo `.exe`):

```bash
notepad.exe ~/.bashrc
ipconfig.exe | grep IPv4
explorer.exe .
```

**Redes:** un servicio que se ejecuta en WSL (por ejemplo, Jupyter en el puerto 8888 o la interfaz web de Spark en el 4040) es accesible desde el navegador de Windows en `http://localhost:<puerto>`.

---

## 9. Herramientas recomendadas: Windows Terminal y VS Code

### Windows Terminal

Viene incluido en Windows 11 (en Windows 10 se instala desde la Microsoft Store). Permite pestañas y paneles con PowerShell, CMD y cada distribución de Linux. Recomendación: en *Configuración → Inicio → Perfil predeterminado* elegir **Ubuntu**.

### Visual Studio Code

1. Instala VS Code **en Windows**.
2. Instala la extensión **WSL** (Microsoft).
3. Desde la terminal de Linux, dentro de la carpeta del proyecto:

   ```bash
   code .
   ```

VS Code se abrirá en Windows, pero trabajando directamente sobre los archivos y el intérprete de Linux (en la esquina inferior izquierda aparecerá `WSL: Ubuntu`). Las extensiones de Python, Jupyter, etc. se instalan "en WSL".

### Git

Git se instala y configura dentro de Linux:

```bash
sudo apt install git
git config --global user.name "Nombre Apellido"
git config --global user.email "correo@ejemplo.com"
git config --global core.autocrlf input
```

---

## 10. Configuración avanzada: `.wslconfig` y `wsl.conf`

Hay dos archivos de configuración:

| Archivo | Ubicación | Ámbito |
|---|---|---|
| `.wslconfig` | `C:\Users\<UsuarioWindows>\.wslconfig` (Windows) | Global: afecta a la VM de WSL 2 (RAM, CPU, red…) |
| `wsl.conf` | `/etc/wsl.conf` (dentro de cada distro) | Por distribución (systemd, usuario por defecto, montaje…) |

Después de modificar cualquiera de ellos hay que reiniciar WSL:

```powershell
wsl --shutdown
```

### 10.1 `.wslconfig`: limitar recursos (muy útil en Big Data)

Por defecto WSL 2 puede usar hasta el 50 % de la RAM del equipo. Con Spark o Hadoop conviene ajustarlo. Ejemplo para un equipo de 16 GB:

```ini
[wsl2]
# RAM máxima para WSL
memory=10GB
# Núcleos de CPU
processors=4
# Tamaño del archivo de intercambio
swap=4GB

[experimental]
# Devuelve a Windows la memoria que no se usa
autoMemoryReclaim=gradual
# El disco virtual se reduce automáticamente
sparseVhd=true
```

### 10.2 `wsl.conf`: activar systemd y otras opciones

```bash
sudo nano /etc/wsl.conf
```

```ini
[boot]
# Permite gestionar servicios con systemctl
systemd=true

[user]
# Usuario por defecto
default=alumno

[interop]
appendWindowsPath=true
```

Las instalaciones recientes de Ubuntu ya traen `systemd=true` activado.

---

## 11. Preparar el entorno para Big Data

Con Ubuntu recién instalado y actualizado:

### 11.1 Utilidades básicas

```bash
sudo apt install -y build-essential curl wget unzip zip git htop tree ca-certificates
```

### 11.2 Java (necesario para Hadoop, Spark y Kafka)

```bash
sudo apt install -y openjdk-17-jdk
java -version
echo 'export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64' >> ~/.bashrc
source ~/.bashrc
```

> Comprueba en la documentación de cada herramienta qué versión de Java admite (Hadoop 3.4 y Spark 3.5 funcionan con Java 17).

### 11.3 Python y entornos virtuales

```bash
sudo apt install -y python3 python3-pip python3-venv
python3 -m venv ~/bigdata/venv
source ~/bigdata/venv/bin/activate
pip install --upgrade pip
pip install pandas numpy jupyterlab pyspark
```

Lanzar Jupyter y abrirlo desde el navegador de Windows:

```bash
jupyter lab --no-browser
# Copia la URL http://localhost:8888/lab?token=... en el navegador
```

### 11.4 SSH (lo necesita Hadoop en modo pseudodistribuido)

```bash
sudo apt install -y openssh-server
sudo systemctl enable --now ssh
ssh-keygen -t ed25519 -N "" -f ~/.ssh/id_ed25519
cat ~/.ssh/id_ed25519.pub >> ~/.ssh/authorized_keys
ssh localhost   # Debe conectar sin pedir contraseña
```

### 11.5 Docker

Hay dos opciones:

- **Docker Desktop para Windows** con la opción *Use the WSL 2 based engine* activada y la integración con Ubuntu habilitada (*Settings → Resources → WSL integration*). Es la más sencilla.
- **Docker Engine instalado directamente en Ubuntu** (requiere systemd activado). Sigue la guía oficial de Docker para Ubuntu y después:

  ```bash
  sudo usermod -aG docker $USER   # Usar docker sin sudo (cierra y vuelve a abrir la sesión)
  docker run hello-world
  ```

### 11.6 Comprobación final

```bash
java -version && python3 --version && git --version && docker --version
```

---

## 12. Copias de seguridad: exportar e importar

Exportar una distribución completa a un archivo (desde PowerShell):

```powershell
wsl --shutdown
wsl --export Ubuntu D:\backups\ubuntu-bigdata.tar
```

Importarla (por ejemplo, en otro equipo o con otro nombre):

```powershell
wsl --import Ubuntu-BigData D:\WSL\Ubuntu-BigData D:\backups\ubuntu-bigdata.tar
wsl -d Ubuntu-BigData
```

> Tras importar, la distro entra como `root`. Define el usuario por defecto en `/etc/wsl.conf` (sección `[user]`) y reinicia con `wsl --shutdown`.

Así el profesorado puede preparar una imagen con todo el entorno instalado y repartirla al alumnado.

---

## 13. Solución de problemas frecuentes

| Problema | Causa probable | Solución |
|---|---|---|
| `0x80370102` o "La máquina virtual no se pudo iniciar" | Virtualización desactivada | Activar VT-x/AMD-V en la BIOS/UEFI y la característica *Plataforma de máquina virtual* |
| La instalación se queda en 0 % | Problema con la Microsoft Store | `wsl --install --web-download -d Ubuntu` |
| `wsl` no se reconoce como comando | Windows demasiado antiguo | Actualizar Windows a 10 v2004 o superior |
| WSL consume mucha RAM (`VmmemWSL`) | Sin límite de memoria | Configurar `.wslconfig` y ejecutar `wsl --shutdown` |
| Todo va lento al leer archivos | Proyecto guardado en `/mnt/c` | Mover el proyecto a `~/` dentro de Linux |
| `bad interpreter: ^M` | Finales de línea CRLF | `dos2unix script.sh` |
| Sin Internet dentro de WSL (VPN, DNS) | Configuración de red | Probar `networkingMode=mirrored` en la sección `[wsl2]` de `.wslconfig` y reiniciar WSL |
| Contraseña de Linux olvidada | — | `wsl -u root` y `passwd <usuario>` |
| El disco virtual ocupa mucho | El `.vhdx` no se reduce solo | Activar `sparseVhd=true` o `wsl --manage <Distro> --set-sparse true` |

Diagnóstico general:

```powershell
wsl --status
wsl --version
wsl -l -v
```

---

## 14. Chuleta rápida

```powershell
# ---------- PowerShell (Windows) ----------
wsl --install                  # Instalar WSL + Ubuntu
wsl -l -o                      # Distros disponibles
wsl -l -v                      # Distros instaladas
wsl                            # Entrar en la distro por defecto
wsl -d <Distro>                # Entrar en una distro concreta
wsl --shutdown                 # Apagar todo WSL
wsl --update                   # Actualizar WSL
wsl --export <Distro> f.tar    # Copia de seguridad
wsl --import <Nombre> <Dir> f.tar
```

```bash
# ---------- Bash (Linux) ----------
sudo apt update && sudo apt upgrade -y   # Actualizar
cd ~                                     # Home de Linux (guardar proyectos aquí)
cd /mnt/c/Users/<UsuarioWindows>         # Disco C: de Windows
explorer.exe .                           # Abrir la carpeta en el Explorador
code .                                   # Abrir la carpeta en VS Code
```

---

## 15. Referencias

- Documentación oficial de WSL (Microsoft Learn): <https://learn.microsoft.com/es-es/windows/wsl/>
- Configurar un entorno de desarrollo con WSL: <https://learn.microsoft.com/es-es/windows/wsl/setup/environment>
- Instalar WSL: <https://learn.microsoft.com/es-es/windows/wsl/install>
- Comandos básicos de WSL: <https://learn.microsoft.com/es-es/windows/wsl/basic-commands>
- Configuración avanzada (`.wslconfig` y `wsl.conf`): <https://learn.microsoft.com/es-es/windows/wsl/wsl-config>
- Versiones de WSL en GitHub: <https://github.com/microsoft/wsl/releases>
