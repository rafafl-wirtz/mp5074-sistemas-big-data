# Manual de SSH: claves para GitHub en Windows y Linux

**Módulo:** MP5074 – Sistemas de Big Data · Curso 2026-27

> SSH (*Secure Shell*) es el protocolo que usamos para **conectarnos de forma segura** a otros equipos y servicios. En este módulo lo necesitamos sobre todo para **autenticarnos en GitHub sin contraseña**: con un par de claves SSH, `git clone`, `git pull` y `git push` funcionan sin pedir usuario ni *token*. Este manual explica cómo crear las claves, añadirlas a GitHub y dejarlas listas tanto en **Windows** (PowerShell, Git for Windows, VS Code) como en **Linux / WSL**. Ver también los manuales de [Git](../git/manual_git.md) y [WSL](../wsl/manual_wsl.md).

---

## Índice

1. [¿Qué es SSH y para qué lo usamos?](#1-qué-es-ssh-y-para-qué-lo-usamos)
2. [Conceptos clave](#2-conceptos-clave)
3. [Comprobar el cliente SSH y las claves existentes](#3-comprobar-el-cliente-ssh-y-las-claves-existentes)
4. [Generar un par de claves](#4-generar-un-par-de-claves)
5. [Añadir la clave pública a GitHub](#5-añadir-la-clave-pública-a-github)
6. [Probar la conexión](#6-probar-la-conexión)
7. [Usar SSH con Git](#7-usar-ssh-con-git)
8. [El archivo de configuración `~/.ssh/config`](#8-el-archivo-de-configuración-sshconfig)
9. [Redes que bloquean el puerto 22](#9-redes-que-bloquean-el-puerto-22)
10. [Windows y WSL en el mismo equipo](#10-windows-y-wsl-en-el-mismo-equipo)
11. [Firmar commits con la clave SSH (opcional)](#11-firmar-commits-con-la-clave-ssh-opcional)
12. [Seguridad y buenas prácticas](#12-seguridad-y-buenas-prácticas)
13. [Solución de problemas frecuentes](#13-solución-de-problemas-frecuentes)
14. [Chuleta rápida](#14-chuleta-rápida)
15. [Referencias](#15-referencias)

---

## 1. ¿Qué es SSH y para qué lo usamos?

SSH es un protocolo que **cifra** toda la comunicación entre dos equipos y permite **autenticarse** sin enviar contraseñas por la red. Se usa para:

- Abrir una **terminal remota** en un servidor (`ssh usuario@servidor`), por ejemplo, el nodo maestro de un clúster Hadoop o Spark.
- Copiar archivos de forma segura (`scp`, `sftp`).
- **Autenticarse en GitHub, GitLab o Bitbucket** para trabajar con repositorios Git. ← *lo que nos ocupa*

**¿Por qué SSH y no HTTPS con GitHub?**

| | HTTPS | SSH |
|---|---|---|
| URL del repositorio | `https://github.com/usuario/repo.git` | `git@github.com:usuario/repo.git` |
| Credencial | *Personal Access Token* o gestor de credenciales | Par de claves |
| ¿Caduca? | Los *tokens* suelen caducar | No, hasta que la borres de GitHub |
| Configuración | Hay que guardar el *token* en algún sitio | Se configura **una vez por equipo** |
| Puerto | 443 | 22 (o 443, ver [sección 9](#9-redes-que-bloquean-el-puerto-22)) |

> GitHub **no acepta la contraseña de la cuenta** en `git push` desde 2021. Hay que usar un *token*, el gestor de credenciales o, lo más cómodo, **SSH**.

---

## 2. Conceptos clave

La autenticación SSH se basa en **criptografía de clave pública**: se genera un **par de claves** relacionadas matemáticamente.

```
                  ssh-keygen
                      │
         ┌────────────┴────────────┐
         ▼                         ▼
  id_ed25519  (PRIVADA)      id_ed25519.pub  (PÚBLICA)
  ─────────────────────      ─────────────────────────
  Se queda en TU equipo      Se copia a GitHub
  NUNCA se comparte          Se puede compartir sin riesgo
```

Al conectar, GitHub envía un "reto" que solo puede resolver quien tenga la clave **privada** correspondiente a una clave **pública** registrada en tu cuenta. La clave privada nunca sale de tu equipo.

| Término | Significado |
|---|---|
| **Clave privada** | Archivo `id_ed25519`. Es tu identidad: quien la tenga puede actuar como tú |
| **Clave pública** | Archivo `id_ed25519.pub`. Una línea de texto que empieza por `ssh-ed25519 AAAA...` |
| **`known_hosts`** | Archivo con las "huellas" de los servidores a los que ya te has conectado; detecta suplantaciones |
| **Huella** (*fingerprint*) | Resumen corto de una clave (`SHA256:...`) para identificarla y compararla |
| **Ed25519** | Algoritmo de clave recomendado hoy: claves cortas, rápidas y seguras. Alternativa compatible: RSA de 4096 bits |

### Dónde se guardan las claves

| Sistema | Carpeta | Ejemplo |
|---|---|---|
| Linux / WSL | `~/.ssh/` | `/home/ana/.ssh/id_ed25519` |
| Windows | `%USERPROFILE%\.ssh\` | `C:\Users\Ana\.ssh\id_ed25519` |

En ambos sistemas la carpeta `.ssh` contiene normalmente:

```
.ssh/
├── id_ed25519          ← clave privada (¡no compartir!)
├── id_ed25519.pub      ← clave pública (esta va a GitHub)
├── known_hosts         ← huellas de servidores conocidos
└── config              ← configuración opcional (sección 8)
```

---

## 3. Comprobar el cliente SSH y las claves existentes

### 3.1 ¿Está instalado el cliente SSH?

=== "Linux / WSL"

    ```bash
    ssh -V
    # OpenSSH_9.6p1 Ubuntu-3ubuntu13, OpenSSL 3.0.13 ...
    ```

    Si no aparece, instálalo:

    ```bash
    sudo apt update
    sudo apt install -y openssh-client
    ```

=== "Windows (PowerShell)"

    Windows 10 (1809 o posterior) y Windows 11 traen **OpenSSH** de serie:

    ```powershell
    ssh -V
    # OpenSSH_for_Windows_9.5p1, LibreSSL 3.8.2
    ```

    Si responde *"ssh no se reconoce como comando"*, actívalo en **Configuración → Sistema → Características opcionales → Agregar una característica → Cliente OpenSSH**, o desde PowerShell **como administrador**:

    ```powershell
    Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0
    ```

> **Git Bash** (incluido en Git for Windows) trae su propio `ssh`. Funciona igual que en Linux, pero usa la misma carpeta `C:\Users\<usuario>\.ssh` que PowerShell.

### 3.2 ¿Ya tienes claves?

=== "Linux / WSL"

    ```bash
    ls -la ~/.ssh
    ```

=== "Windows (PowerShell)"

    ```powershell
    Get-ChildItem $env:USERPROFILE\.ssh
    ```

- Si ves `id_ed25519` e `id_ed25519.pub` (o `id_rsa` e `id_rsa.pub`), **ya tienes un par de claves**: puedes saltar a la [sección 5](#5-añadir-la-clave-pública-a-github).
- Si la carpeta no existe o está vacía, crea las claves en la sección 4.

> ⚠️ **No sobrescribas una clave existente** sin saber para qué se usa: podrías perder el acceso a otros servidores. Si quieres una clave nueva, ponle otro nombre (sección 4.2).

---

## 4. Generar un par de claves

### 4.1 Crear la clave

El comando es **el mismo** en Linux, WSL, PowerShell y Git Bash. Usa el **correo de tu cuenta de GitHub** como comentario (solo sirve para identificar la clave):

```bash
ssh-keygen -t ed25519 -C "correo@ejemplo.com"
```

Pulsa **Intro** en todas las preguntas para aceptar los valores por defecto:

```
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/ana/.ssh/id_ed25519):    ← Intro
Enter passphrase (empty for no passphrase):                         ← Intro
Enter same passphrase again:                                        ← Intro
Your identification has been saved in /home/ana/.ssh/id_ed25519
Your public key has been saved in /home/ana/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:Xb3k9...Qz7 correo@ejemplo.com
```

En Windows la ruta propuesta será `C:\Users\Ana/.ssh/id_ed25519`; acéptala igualmente con **Intro**.

### 4.2 Opciones útiles de `ssh-keygen`

```bash
# Elegir otro nombre de archivo (p. ej., una clave solo para el centro)
ssh-keygen -t ed25519 -C "correo@ejemplo.com" -f ~/.ssh/id_ed25519_centro

# Si un sistema antiguo no admite Ed25519, usa RSA de 4096 bits
ssh-keygen -t rsa -b 4096 -C "correo@ejemplo.com"

# Ver la huella de una clave (para compararla con la que muestra GitHub)
ssh-keygen -lf ~/.ssh/id_ed25519.pub
```

En PowerShell, `~` también funciona en estos comandos; si da problemas, usa `$env:USERPROFILE\.ssh\...`.

---

## 5. Añadir la clave pública a GitHub

### 5.1 Copiar la clave **pública**

⚠️ Copia siempre el archivo que acaba en **`.pub`**.

=== "Linux / WSL"

    ```bash
    cat ~/.ssh/id_ed25519.pub
    # ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAI...  correo@ejemplo.com
    ```

    Selecciona y copia la línea **completa**. En WSL puedes mandarla directamente al portapapeles de Windows:

    ```bash
    cat ~/.ssh/id_ed25519.pub | clip.exe
    ```

    En Ubuntu de escritorio: `xclip -selection clipboard < ~/.ssh/id_ed25519.pub` (instala antes `xclip`).

=== "Windows (PowerShell)"

    ```powershell
    Get-Content $env:USERPROFILE\.ssh\id_ed25519.pub | Set-Clipboard
    ```

    O muéstrala y cópiala a mano:

    ```powershell
    type $env:USERPROFILE\.ssh\id_ed25519.pub
    ```

### 5.2 Registrarla en GitHub (web)

1. Inicia sesión en <https://github.com>.
2. Arriba a la derecha, tu foto → **Settings**.
3. Menú lateral, sección *Access* → **SSH and GPG keys**.
4. Botón **New SSH key**.
5. Rellena:
    - **Title**: un nombre que identifique el **equipo**, p. ej., `Portátil casa - WSL` o `Aula 2.05 - Windows`.
    - **Key type**: `Authentication Key`.
    - **Key**: pega la clave (debe empezar por `ssh-ed25519` o `ssh-rsa`).
6. **Add SSH key** (GitHub puede pedir confirmación con tu contraseña o 2FA).

Enlace directo: <https://github.com/settings/keys>.

### 5.3 Registrarla con GitHub CLI (alternativa)

Si ya usas `gh` (ver [manual de Git](../git/manual_git.md), sección 12.2):

```bash
gh auth refresh -h github.com -s admin:public_key     # Da permiso a gh para gestionar claves (una vez)
gh ssh-key add ~/.ssh/id_ed25519.pub --title "Portátil casa - WSL"
gh ssh-key list
```

> Al ejecutar `gh auth login` y elegir **SSH** como protocolo, `gh` también ofrece generar y subir la clave automáticamente.

---

## 6. Probar la conexión

```bash
ssh -T git@github.com
```

La **primera vez** aparece un aviso sobre la autenticidad del servidor:

```
The authenticity of host 'github.com (140.82.121.4)' can't be established.
ED25519 key fingerprint is SHA256:+DiY3wvvV6TuJJhbpZisF/zLDA0zPMSvHdkr4UvCOqU.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

Comprueba que la huella coincide con la **publicada por GitHub** y escribe `yes`:

| Tipo | Huella oficial de GitHub |
|---|---|
| Ed25519 | `SHA256:+DiY3wvvV6TuJJhbpZisF/zLDA0zPMSvHdkr4UvCOqU` |
| ECDSA | `SHA256:p2QAMXNIC1TJYWeIOttrVc98/R1BUFWu3/LiyKgUfQM` |
| RSA | `SHA256:uNiVztksCsDhcc0u9e8BujQXVUpKZIDTMczCvj3tD2s` |

> Si la huella **no coincide**, responde `no`: alguien podría estar interceptando la conexión. Consulta siempre la lista actualizada en la [documentación de GitHub](https://docs.github.com/es/authentication/keeping-your-account-and-data-secure/githubs-ssh-key-fingerprints).

La huella se guarda en `~/.ssh/known_hosts` y ya no se vuelve a preguntar. Si todo va bien:

```
Hi ana-garcia! You've successfully authenticated, but GitHub does not provide shell access.
```

✅ El mensaje *"does not provide shell access"* es **correcto**: GitHub no da terminal, solo acepta operaciones de Git.

Para ver qué está pasando cuando algo falla, usa el modo detallado:

```bash
ssh -vT git@github.com        # -vvv para aún más detalle
```

Busca líneas como `Offering public key: .../id_ed25519` y `Authenticated to github.com`.

---

## 7. Usar SSH con Git

### 7.1 Clonar por SSH

En GitHub, botón verde **Code → pestaña SSH** → copia la URL:

```bash
git clone git@github.com:usuario/repositorio.git
```

### 7.2 Pasar un repositorio existente de HTTPS a SSH

```bash
git remote -v
# origin  https://github.com/usuario/repositorio.git (fetch)

git remote set-url origin git@github.com:usuario/repositorio.git

git remote -v
# origin  git@github.com:usuario/repositorio.git (fetch)
git push                    # Ya no pide usuario ni token
```

### 7.3 Forzar SSH aunque copies URLs HTTPS (opcional)

```bash
git config --global url."git@github.com:".insteadOf "https://github.com/"
```

Con esto, Git convierte automáticamente cualquier URL `https://github.com/...` en su equivalente SSH.

### 7.4 VS Code

VS Code usa el Git del sistema, así que **hereda la configuración SSH**:

- **VS Code conectado a WSL** (`code .` desde Ubuntu): usa las claves de `~/.ssh` de WSL.
- **VS Code en Windows**: usa las claves de `C:\Users\<usuario>\.ssh`.

---

## 8. El archivo de configuración `~/.ssh/config`

Este archivo de texto (sin extensión) permite definir **alias y opciones por servidor**. Ruta:

- Linux / WSL: `~/.ssh/config`
- Windows: `C:\Users\<usuario>\.ssh\config`

> ⚠️ En Windows, si lo creas con el Bloc de notas, asegúrate de que **no** se guarda como `config.txt` (elige *Tipo: Todos los archivos*). Mejor: `code $env:USERPROFILE\.ssh\config`.

### 8.1 Configuración básica para GitHub

```sshconfig
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519
    IdentitiesOnly yes
```

| Opción | Para qué sirve |
|---|---|
| `Host` | Nombre (alias) que escribirás en los comandos |
| `HostName` | Servidor real al que se conecta |
| `User` | Usuario remoto (en GitHub siempre `git`) |
| `IdentityFile` | Qué clave privada usar |
| `IdentitiesOnly yes` | Probar **solo** esa clave (evita *"Too many authentication failures"*) |
| `Port` | Puerto (por defecto 22) |

### 8.2 Dos cuentas de GitHub en el mismo equipo

Por ejemplo, una cuenta personal y otra del centro, cada una con su clave:

```sshconfig
# Cuenta personal
Host github-personal
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519
    IdentitiesOnly yes

# Cuenta del centro
Host github-centro
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_centro
    IdentitiesOnly yes
```

Al clonar, sustituye `github.com` por el **alias**:

```bash
git clone git@github-centro:rafafl-wirtz/mp5074-sistemas-big-data.git
ssh -T git@github-centro                # "Hi <cuenta-del-centro>!"
```

> Recuerda configurar también el nombre y el correo **de ese repositorio** con `git config user.email "..."` (sin `--global`).

### 8.3 Alias para servidores del módulo

El mismo archivo sirve para los servidores a los que te conectes con `ssh` (máquinas virtuales, nodos de un clúster):

```sshconfig
Host spark-master
    HostName 192.168.56.10
    User alumno
    IdentityFile ~/.ssh/id_ed25519
```

Ahora basta con `ssh spark-master`. Para entrar sin contraseña en un servidor Linux, copia tu clave pública a él:

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub alumno@192.168.56.10
```

---

## 9. Redes que bloquean el puerto 22

Algunas redes (centros educativos, empresas, wifi públicas) bloquean el puerto 22. El síntoma es:

```
ssh: connect to host github.com port 22: Connection timed out
```

GitHub ofrece SSH también por el **puerto 443** en el servidor `ssh.github.com`. Compruébalo:

```bash
ssh -T -p 443 git@ssh.github.com
# Hi ana-garcia! You've successfully authenticated...
```

Si funciona, añade a `~/.ssh/config`:

```sshconfig
Host github.com
    HostName ssh.github.com
    Port 443
    User git
    IdentityFile ~/.ssh/id_ed25519
```

No hace falta cambiar las URLs de los repositorios: `git@github.com:...` seguirá funcionando y saldrá por el 443.

> La primera conexión por este servidor vuelve a pedir confirmar la huella: es la misma que la de `github.com`.

---

## 10. Windows y WSL en el mismo equipo

Windows y WSL tienen **carpetas `.ssh` distintas**:

```
Windows:  C:\Users\Ana\.ssh\        ← la usan PowerShell, Git for Windows, VS Code (Windows)
WSL:      /home/ana/.ssh/           ← la usan Ubuntu y VS Code conectado a WSL
```

Lo recomendado es tener **una clave en cada sistema**: repite las secciones 4–6 en Windows y en WSL, y añade **las dos** claves públicas a GitHub con títulos distintos (`Portátil - Windows`, `Portátil - WSL`). Si una se compromete, se revoca solo esa.

> **No** apuntes `IdentityFile` en WSL a la clave de Windows (`/mnt/c/...`): SSH la rechaza.

---

## 11. Firmar commits con la clave SSH (opcional)

GitHub puede mostrar tus commits como **Verified** si los firmas. Desde Git 2.34 se puede firmar con la misma clave SSH:

```bash
git config --global gpg.format ssh
git config --global user.signingkey ~/.ssh/id_ed25519.pub
git config --global commit.gpgsign true          # Firmar todos los commits
```

En GitHub, **Settings → SSH and GPG keys → New SSH key**, pega **la misma clave pública** y en *Key type* elige **Signing Key** (la clave queda registrada dos veces: una para autenticar y otra para firmar).

> En Windows, usa la ruta completa: `git config --global user.signingkey "C:/Users/Ana/.ssh/id_ed25519.pub"`.

---

## 12. Seguridad y buenas prácticas

1. **La clave privada nunca sale del equipo**: no se envía por correo, no se sube a Git, no se pega en un chat ni en un notebook.
2. **Una clave por equipo** (y por sistema: Windows y WSL), con un **título claro** en GitHub. Así se puede revocar solo la del equipo perdido.
3. **Revisa periódicamente** <https://github.com/settings/keys>: GitHub muestra la fecha del último uso de cada clave. Borra las que no reconozcas o ya no uses.
4. **En equipos compartidos** del aula: al terminar el curso, borra la clave de GitHub **y** del equipo (`~/.ssh/id_ed25519*`).
5. Si sospechas que te han robado la clave: **bórrala de GitHub inmediatamente** y genera otra.
6. Añade a tu `.gitignore` global patrones como `*.pem`, `*.key`, `id_*` para no subir claves por error (ver [manual de Git](../git/manual_git.md), sección 6).
7. Usa **Ed25519**; evita RSA de menos de 2048 bits y DSA (GitHub ya no los admite).

---

## 13. Solución de problemas frecuentes

| Problema | Causa probable | Solución |
|---|---|---|
| `Permission denied (publickey)` | La clave no está en GitHub, no se está ofreciendo o es otra | `ssh -vT git@github.com` para ver qué clave ofrece; comprueba que la `.pub` está en GitHub |
| `Host key verification failed` | Cambió la huella guardada de GitHub o `known_hosts` está corrupto | `ssh-keygen -R github.com` y vuelve a conectar comprobando la huella (sección 6) |
| `Connection timed out` en el puerto 22 | Red que bloquea el puerto 22 | Usar el puerto 443 (sección 9) |
| Sigue pidiendo **usuario y contraseña** | El remoto usa una URL HTTPS | `git remote set-url origin git@github.com:usuario/repo.git` (sección 7.2) |
| `Too many authentication failures` | Se ofrecen demasiadas claves | `IdentitiesOnly yes` e `IdentityFile` en `~/.ssh/config` (sección 8) |
| `Key is already in use` al añadirla a GitHub | Esa clave ya está en otra cuenta (o en la misma) | Usa una clave distinta para cada cuenta (sección 8.2) |
| `ERROR: Repository not found` | Sin permiso en ese repo con la cuenta asociada a la clave | `ssh -T git@github.com` para ver con qué cuenta entras; pide acceso o usa el alias correcto |
| `Load key ...: invalid format` | Se ha pegado la clave mal, o el archivo tiene finales de línea de Windows | Genera la clave de nuevo en ese sistema o convierte con `dos2unix` |
| El archivo `config` no hace efecto (Windows) | Se guardó como `config.txt` | Renómbralo: `Rename-Item $env:USERPROFILE\.ssh\config.txt config` |

Diagnóstico general:

```bash
ssh -V                          # Versión del cliente
ls -la ~/.ssh                   # Claves existentes
ssh -vT git@github.com          # Conexión con detalle
git remote -v                   # ¿URL SSH o HTTPS?
```

---

## 14. Chuleta rápida

```bash
# ---------- Crear la clave (igual en Linux, WSL, PowerShell y Git Bash) ----------
ssh-keygen -t ed25519 -C "correo@ejemplo.com"          # Intro en todas las preguntas

# ---------- Copiar la clave PÚBLICA ----------
cat ~/.ssh/id_ed25519.pub                                             # Linux
cat ~/.ssh/id_ed25519.pub | clip.exe                                  # WSL → portapapeles
Get-Content $env:USERPROFILE\.ssh\id_ed25519.pub | Set-Clipboard      # PowerShell
# → GitHub: Settings → SSH and GPG keys → New SSH key

# ---------- Probar ----------
ssh -T git@github.com                     # "Hi usuario! You've successfully authenticated"
ssh -T -p 443 git@ssh.github.com          # Si el puerto 22 está bloqueado
ssh -vT git@github.com                    # Diagnóstico

# ---------- Usar con Git ----------
git clone git@github.com:usuario/repo.git
git remote set-url origin git@github.com:usuario/repo.git
```

---

## 15. Referencias

- GitHub Docs – Conectar con GitHub mediante SSH: <https://docs.github.com/es/authentication/connecting-to-github-with-ssh>
- GitHub Docs – Generar una clave SSH: <https://docs.github.com/es/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent>
- GitHub Docs – Huellas de las claves SSH de GitHub: <https://docs.github.com/es/authentication/keeping-your-account-and-data-secure/githubs-ssh-key-fingerprints>
- GitHub Docs – Usar SSH a través del puerto HTTPS: <https://docs.github.com/es/authentication/troubleshooting-ssh/using-ssh-over-the-https-port>
- GitHub Docs – Solución de problemas de SSH: <https://docs.github.com/es/authentication/troubleshooting-ssh>
- GitHub Docs – Firmar commits con SSH: <https://docs.github.com/es/authentication/managing-commit-signature-verification/telling-git-about-your-signing-key>
- Microsoft Learn – OpenSSH para Windows: <https://learn.microsoft.com/es-es/windows-server/administration/openssh/openssh_overview>
- Manual de OpenSSH (`man ssh_config`): <https://man.openbsd.org/ssh_config>
