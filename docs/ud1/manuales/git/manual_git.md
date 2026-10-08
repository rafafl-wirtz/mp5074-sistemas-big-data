# Manual de Git

**Módulo:** MP5074 – Sistemas de Big Data · Curso 2026-27

> Git es un sistema de **control de versiones**: guarda la historia de todos los cambios de un proyecto, permite volver a cualquier versión anterior y facilita que varias personas trabajen a la vez sobre el mismo código. En este módulo lo usaremos desde **WSL (Ubuntu)** para gestionar scripts, notebooks y configuraciones, y **GitHub** para compartirlos y entregar prácticas. Ver también los manuales de [WSL](../wsl/manual_wsl.md) y [JupyterLab](../jupyterlab/manual_jupyterlab.md).

---

## Índice

1. [¿Qué es Git?](#1-qué-es-git)
2. [Conceptos clave](#2-conceptos-clave)
3. [Instalación y configuración inicial](#3-instalación-y-configuración-inicial)
4. [Crear o clonar un repositorio](#4-crear-o-clonar-un-repositorio)
5. [El flujo básico: `add` y `commit`](#5-el-flujo-básico-add-y-commit)
6. [Ignorar archivos: `.gitignore`](#6-ignorar-archivos-gitignore)
7. [Consultar la historia y las diferencias](#7-consultar-la-historia-y-las-diferencias)
8. [Deshacer cambios](#8-deshacer-cambios)
9. [Ramas](#9-ramas)
10. [Fusionar ramas y resolver conflictos](#10-fusionar-ramas-y-resolver-conflictos)
11. [Repositorios remotos y GitHub](#11-repositorios-remotos-y-github)
12. [Autenticación con GitHub desde WSL](#12-autenticación-con-github-desde-wsl)
13. [Trabajo en equipo: *pull requests* y *forks*](#13-trabajo-en-equipo-pull-requests-y-forks)
14. [Otras herramientas útiles: `stash`, etiquetas y `rebase`](#14-otras-herramientas-útiles-stash-etiquetas-y-rebase)
15. [Git en proyectos de datos](#15-git-en-proyectos-de-datos)
16. [Git en VS Code y JupyterLab](#16-git-en-vs-code-y-jupyterlab)
17. [Buenas prácticas](#17-buenas-prácticas)
18. [Solución de problemas frecuentes](#18-solución-de-problemas-frecuentes)
19. [Chuleta rápida](#19-chuleta-rápida)
20. [Referencias](#20-referencias)

---

## 1. ¿Qué es Git?

Git es un sistema de control de versiones **distribuido** creado en 2005 por Linus Torvalds para desarrollar el kernel de Linux. Hoy es el estándar en la industria.

**Distribuido** significa que cada persona tiene en su equipo **una copia completa** del repositorio, con toda su historia. Se puede trabajar sin conexión y sincronizar después.

**¿Para qué sirve?**

- Guardar **versiones** del proyecto con una descripción de qué cambió y por qué.
- **Volver atrás** si algo se rompe.
- Probar ideas en **ramas** sin estropear la versión que funciona.
- **Colaborar** sin pisarse el trabajo y con registro de quién hizo cada cambio.
- Tener una **copia de seguridad** en un servidor remoto (GitHub, GitLab…).

**Git no es GitHub:**

| | Git | GitHub / GitLab / Bitbucket |
|---|---|---|
| Qué es | Programa que se instala en el equipo | Servicio web que aloja repositorios Git |
| Funciona sin Internet | Sí | No |
| Aporta | Control de versiones | Copia remota, colaboración, *pull requests*, *issues*, CI/CD… |

---

## 2. Conceptos clave

- **Repositorio (repo):** carpeta del proyecto cuya historia controla Git. La información interna está en la subcarpeta oculta `.git/`.
- **Commit:** una "foto" del proyecto en un momento dado, con autor, fecha, mensaje y un identificador único (*hash*, p. ej., `a1b2c3d`).
- **Rama (*branch*):** línea de desarrollo independiente. La principal suele llamarse `main`.
- **`HEAD`:** puntero al commit en el que estás trabajando.
- **Remoto (*remote*):** copia del repositorio en otro sitio, normalmente GitHub. Por convención el principal se llama `origin`.

### Las tres áreas de Git

```
 Directorio de trabajo        Área de preparación          Repositorio
  (tus archivos)               (staging / índice)           (historia, .git)
┌─────────────────┐  git add  ┌─────────────────┐ git commit ┌─────────────────┐
│ archivos        │──────────▶│ cambios que     │───────────▶│ commits         │
│ modificados     │           │ irán en el      │            │ guardados       │
│                 │◀──────────│ próximo commit  │            │                 │
└─────────────────┘ git restore└─────────────────┘            └─────────────────┘
        ▲                                                            │
        └─────────────────── git switch / git restore ◀──────────────┘
```

### Estados de un archivo

| Estado | Significado |
|---|---|
| *Untracked* (sin seguimiento) | Git lo ve, pero nunca se ha añadido |
| *Modified* (modificado) | Ha cambiado desde el último commit |
| *Staged* (preparado) | Marcado con `git add` para el próximo commit |
| *Committed* (confirmado) | Guardado en la historia |

---

## 3. Instalación y configuración inicial

### 3.1 Instalar Git en WSL (Ubuntu)

```bash
sudo apt update
sudo apt install -y git
git --version
```

> En Windows existe **Git for Windows** (<https://git-scm.com/downloads/win>). No es obligatorio para trabajar en WSL, pero sí lo necesitas si quieres usar el gestor de credenciales de Windows (sección 12.3).

### 3.2 Identidad (obligatorio)

Cada commit lleva nombre y correo. Usa el **mismo correo que en GitHub**:

```bash
git config --global user.name "Nombre Apellido"
git config --global user.email "correo@ejemplo.com"
```

### 3.3 Configuración recomendada

```bash
git config --global init.defaultBranch main     # Rama principal "main" en los repos nuevos
git config --global core.autocrlf input         # Guarda finales de línea LF (ver 15.3)
git config --global core.editor "nano"          # Editor para mensajes (o "code --wait" para VS Code)
git config --global pull.rebase false           # "git pull" fusiona (comportamiento clásico)
git config --global color.ui auto
```

Consultar la configuración:

```bash
git config --global --list
git config user.email          # Un valor concreto
```

Niveles de configuración: `--system` (todo el equipo), `--global` (tu usuario, en `~/.gitconfig`) y `--local` (solo ese repositorio, en `.git/config`). Manda el más específico.

### 3.4 Ayuda

```bash
git help commit
git commit -h          # Resumen breve de opciones
```

---

## 4. Crear o clonar un repositorio

### 4.1 Crear un repositorio nuevo

```bash
mkdir -p ~/bigdata/proyecto-ventas
cd ~/bigdata/proyecto-ventas
git init
```

Se crea la carpeta oculta `.git/`. **No la borres ni la modifiques a mano**: es todo el historial.

### 4.2 Clonar un repositorio existente

```bash
cd ~/bigdata
git clone https://github.com/usuario/repositorio.git
git clone git@github.com:usuario/repositorio.git mi-carpeta   # Por SSH y con otro nombre de carpeta
```

`git clone` descarga el proyecto con toda su historia y configura el remoto `origin` automáticamente.

> **En WSL:** crea los repositorios dentro de Linux (`~/...`), **no** en `/mnt/c/...`: allí Git es mucho más lento y hay problemas de permisos y finales de línea.

---

## 5. El flujo básico: `add` y `commit`

### 5.1 Ver el estado

```bash
git status
git status -s        # Versión corta
```

Es el comando más importante: **úsalo antes y después de cada operación**.

### 5.2 Preparar cambios: `git add`

```bash
git add limpieza.py              # Un archivo
git add scripts/                 # Una carpeta
git add .                        # Todo lo modificado y nuevo del directorio actual
git add -p                       # Elegir los fragmentos de cada archivo de forma interactiva
```

### 5.3 Confirmar: `git commit`

```bash
git commit -m "Añade script de limpieza de ventas"
git commit                       # Abre el editor para escribir un mensaje largo
git commit -am "Corrige ruta del CSV"   # add + commit de archivos YA seguidos (no incluye nuevos)
```

### 5.4 Ejemplo completo

```bash
cd ~/bigdata/proyecto-ventas
echo "# Proyecto ventas" > README.md
git status                       # README.md aparece como "Untracked"
git add README.md
git status                       # Aparece en "Changes to be committed"
git commit -m "Primer commit: añade README"
git log --oneline                # a1b2c3d (HEAD -> main) Primer commit: añade README
```

### 5.5 Mover y borrar archivos

```bash
git mv antiguo.py nuevo.py       # Renombrar
git rm fichero.py                # Borrar del disco y de Git
git rm --cached datos.csv        # Dejar de seguirlo en Git, pero conservarlo en el disco
```

---

## 6. Ignorar archivos: `.gitignore`

El archivo `.gitignore`, en la raíz del repositorio, indica qué **no** debe entrar nunca en Git.

### 6.1 Sintaxis

| Patrón | Qué ignora |
|---|---|
| `datos.csv` | Ese archivo en cualquier carpeta |
| `*.log` | Todos los `.log` |
| `datos/` | La carpeta `datos` y todo su contenido |
| `/config.local` | Solo el de la raíz |
| `!datos/ejemplo.csv` | **Excepción**: no ignorar este archivo |
| `**/tmp/` | Carpetas `tmp` a cualquier profundidad |

### 6.2 `.gitignore` recomendado para el módulo

```gitignore
# ---------- Python ----------
__pycache__/
*.py[cod]
.venv/
venv/
env/

# ---------- Jupyter ----------
.ipynb_checkpoints/

# ---------- Datos (no se suben a Git) ----------
datos/
*.csv
*.parquet
*.json.gz
!datos/muestra/

# ---------- Spark / Hadoop ----------
spark-warehouse/
metastore_db/
derby.log
*.crc

# ---------- Credenciales y configuración local ----------
.env
*.pem
*.key
credenciales*.json

# ---------- Sistema y editores ----------
.DS_Store
Thumbs.db
.vscode/
.idea/
```

> **`.gitignore` solo afecta a archivos que aún no están seguidos.** Si ya hiciste commit de un archivo, añádelo al `.gitignore` y además ejecuta `git rm --cached <archivo>`.

Plantillas por lenguaje: <https://github.com/github/gitignore>.

---

## 7. Consultar la historia y las diferencias

### 7.1 `git log`

```bash
git log                                  # Historia completa
git log --oneline                        # Una línea por commit
git log --oneline --graph --all          # Con gráfico de ramas ✅
git log -5                               # Últimos 5 commits
git log -- limpieza.py                   # Commits que tocaron ese archivo
git log --author="Ana"                   # Por autor
git log --since="2 weeks ago"            # Por fecha
git log -S "groupBy"                     # Commits que añadieron o quitaron ese texto
```

Crear un alias para el log con gráfico:

```bash
git config --global alias.lg "log --oneline --graph --all --decorate"
git lg
```

### 7.2 `git diff`

```bash
git diff                         # Cambios del directorio de trabajo aún NO preparados
git diff --staged                # Cambios preparados (lo que irá en el commit)
git diff main rama-nueva         # Diferencias entre dos ramas
git diff a1b2c3d e4f5g6h         # Entre dos commits
git diff --stat                  # Resumen: archivos y nº de líneas cambiadas
```

### 7.3 Ver un commit o una versión antigua

```bash
git show a1b2c3d                         # Qué cambió en ese commit
git show a1b2c3d:scripts/limpieza.py     # El archivo tal como estaba en ese commit
git blame scripts/limpieza.py            # Quién cambió cada línea y cuándo
```

### 7.4 Referencias a commits

| Referencia | Significado |
|---|---|
| `a1b2c3d` | Commit por su *hash* (basta con los primeros 7 caracteres) |
| `HEAD` | Commit actual |
| `HEAD~1` | El anterior al actual |
| `HEAD~3` | Tres commits antes |
| `main` | Último commit de la rama `main` |

---

## 8. Deshacer cambios

> ⚠️ Antes de deshacer, haz `git status`. Los comandos marcados con ⚠️ **pierden trabajo** sin posibilidad de recuperarlo fácilmente.

| Situación | Comando |
|---|---|
| Descartar cambios de un archivo **no preparados** | ⚠️ `git restore archivo.py` |
| Quitar un archivo del área de preparación (sin perder cambios) | `git restore --staged archivo.py` |
| Corregir el mensaje del **último** commit o añadirle algo que faltaba | `git add olvidado.py` y `git commit --amend` |
| Deshacer un commit **ya subido** creando otro que lo invierte (seguro) | `git revert a1b2c3d` |
| Deshacer los últimos commits **locales** conservando los cambios en los archivos | `git reset HEAD~1` |
| Deshacer commits locales **y borrar** los cambios | ⚠️ `git reset --hard HEAD~1` |
| Recuperar un archivo tal como estaba en un commit | `git restore --source a1b2c3d archivo.py` |
| Borrar archivos no seguidos (ver antes con `-n`) | `git clean -n` y después ⚠️ `git clean -f` |

### `revert` frente a `reset`

- **`git revert`** crea un commit nuevo que deshace otro. **No reescribe la historia**: úsalo con commits que ya están en GitHub.
- **`git reset`** mueve la rama hacia atrás y **reescribe la historia**: úsalo solo con commits que **no has subido**.

### Red de seguridad: `git reflog`

Git registra durante un tiempo todos los movimientos de `HEAD`, incluso los commits "perdidos" tras un `reset`:

```bash
git reflog                       # Lista: a1b2c3d HEAD@{2}: commit: Añade limpieza...
git reset --hard a1b2c3d         # Vuelve a ese punto
```

---

## 9. Ramas

Una rama permite desarrollar una funcionalidad o probar algo **sin afectar a `main`**.

```
main:          A───B───────────E  (merge)
                    \         /
rama-limpieza:       C───D───┘
```

| Comando | Descripción |
|---|---|
| `git branch` | Lista ramas locales (`*` = actual) |
| `git branch -a` | Incluye las ramas remotas |
| `git branch limpieza` | Crea una rama (sin cambiarse a ella) |
| `git switch limpieza` | Cambia a una rama |
| `git switch -c limpieza` | Crea la rama y se cambia a ella ✅ |
| `git branch -m viejo nuevo` | Renombra una rama |
| `git branch -d limpieza` | Borra una rama ya fusionada |
| `git branch -D limpieza` | ⚠️ Borra una rama aunque no esté fusionada |

> `git checkout` hace lo mismo que `switch` y `restore` juntos (sintaxis antigua). Verás mucho `git checkout -b rama`, que equivale a `git switch -c rama`.

**Antes de cambiar de rama**, haz commit de tus cambios (o guárdalos con `git stash`, sección 14).

---

## 10. Fusionar ramas y resolver conflictos

### 10.1 Fusionar (*merge*)

Desde la rama **que recibe** los cambios:

```bash
git switch main
git merge limpieza
git branch -d limpieza          # La rama ya no hace falta
```

Tipos de fusión:

- **Fast-forward:** si `main` no ha cambiado desde que se creó la rama, Git solo avanza el puntero; no crea commit de fusión.
- **Fusión de tres vías:** si ambas ramas tienen commits nuevos, Git crea un **commit de fusión**.

### 10.2 Conflictos

Aparecen cuando dos ramas cambian **las mismas líneas** de un archivo:

```
Auto-merging limpieza.py
CONFLICT (content): Merge conflict in limpieza.py
Automatic merge failed; fix conflicts and then commit the result.
```

El archivo queda marcado así:

```python
<<<<<<< HEAD
df = df.dropna(subset=["importe"])
=======
df = df.fillna({"importe": 0})
>>>>>>> limpieza
```

- Entre `<<<<<<< HEAD` y `=======` → la versión de tu rama actual.
- Entre `=======` y `>>>>>>> limpieza` → la versión de la rama que fusionas.

**Pasos para resolverlo:**

1. `git status` para ver los archivos en conflicto (*both modified*).
2. Edita cada archivo: deja el código correcto y **borra las marcas** `<<<<<<<`, `=======` y `>>>>>>>`.
3. `git add limpieza.py`
4. `git commit` (Git propone un mensaje de fusión).

Si prefieres abandonar la fusión: `git merge --abort`.

> VS Code muestra los conflictos con botones *Accept Current / Accept Incoming / Accept Both*, lo que facilita mucho la resolución.

---

## 11. Repositorios remotos y GitHub

### 11.1 Crear el repositorio en GitHub y conectarlo

1. En GitHub: **New repository**. Ponle nombre y **no** marques *Add a README* si ya tienes un repositorio local.
2. En la terminal:

```bash
git remote add origin git@github.com:usuario/proyecto-ventas.git
git push -u origin main
```

`-u` vincula tu rama local con la remota; después bastará con `git push`.

### 11.2 Comandos de remotos

| Comando | Descripción |
|---|---|
| `git remote -v` | Lista los remotos y sus URLs |
| `git remote add <nombre> <url>` | Añade un remoto |
| `git remote set-url origin <url>` | Cambia la URL (p. ej., de HTTPS a SSH) |
| `git fetch` | Descarga cambios del remoto **sin** aplicarlos |
| `git pull` | Descarga **y** fusiona en tu rama (`fetch` + `merge`) |
| `git push` | Sube tus commits |
| `git push -u origin <rama>` | Sube una rama nueva y la vincula |
| `git push origin --delete <rama>` | Borra una rama remota |

### 11.3 Ciclo de trabajo diario

```bash
git pull                          # 1. Trae lo último antes de empezar
# ... trabajar ...
git status                        # 2. Revisa qué has cambiado
git add .                         # 3. Prepara
git commit -m "Mensaje claro"     # 4. Confirma
git pull                          # 5. Integra cambios de otros (resuelve conflictos si hay)
git push                          # 6. Sube
```

> **Nunca uses `git push --force`** en ramas compartidas (como `main`): sobrescribe el trabajo de otras personas. Si de verdad es necesario en tu propia rama, usa `git push --force-with-lease`, que es más seguro.

---

## 12. Autenticación con GitHub desde WSL

GitHub **no acepta la contraseña de la cuenta** para `git push` por HTTPS. Elige **una** de estas opciones.

### 12.1 Opción 1: claves SSH (recomendada)

```bash
ssh-keygen -t ed25519 -C "correo@ejemplo.com"     # Pulsa Enter para aceptar la ruta; pon una frase de paso
cat ~/.ssh/id_ed25519.pub                          # Copia la clave PÚBLICA que se muestra
```

1. En GitHub: **Settings → SSH and GPG keys → New SSH key** y pega la clave.
2. Comprueba la conexión:

   ```bash
   ssh -T git@github.com
   # Hi usuario! You've successfully authenticated...
   ```

3. Usa URLs SSH: `git@github.com:usuario/repositorio.git`.

> Nunca compartas el archivo `~/.ssh/id_ed25519` (sin `.pub`): es tu clave **privada**.

### 12.2 Opción 2: GitHub CLI (`gh`)

```bash
sudo apt install -y gh
gh auth login          # Elige GitHub.com → HTTPS → "Login with a web browser"
```

`gh` configura las credenciales de Git automáticamente y además permite crear repositorios y *pull requests* desde la terminal (`gh repo create`, `gh pr create`).

### 12.3 Opción 3: Git Credential Manager de Windows

Si tienes **Git for Windows** instalado, WSL puede usar su gestor de credenciales (inicio de sesión en el navegador):

```bash
git config --global credential.helper "/mnt/c/Program\ Files/Git/mingw64/bin/git-credential-manager.exe"
```

La primera vez que hagas `git push` por HTTPS se abrirá una ventana de inicio de sesión de GitHub.

---

## 13. Trabajo en equipo: *pull requests* y *forks*

### 13.1 Flujo con ramas y *pull requests* (*GitHub Flow*)

1. Actualiza `main`: `git switch main && git pull`.
2. Crea una rama para la tarea: `git switch -c analisis-ventas`.
3. Haz commits pequeños y súbelos: `git push -u origin analisis-ventas`.
4. En GitHub, abre un **Pull Request (PR)** de `analisis-ventas` hacia `main`.
5. El equipo (o el profesorado) **revisa** el código y comenta.
6. Cuando está aprobado, se hace **Merge** en GitHub.
7. En local: `git switch main && git pull && git branch -d analisis-ventas`.

### 13.2 *Forks*

Un **fork** es una copia de un repositorio ajeno en tu cuenta de GitHub. Se usa para contribuir a proyectos en los que no tienes permiso de escritura (o para partir de una plantilla de prácticas):

```bash
git clone git@github.com:MI_USUARIO/proyecto.git
cd proyecto
git remote add upstream https://github.com/ORIGINAL/proyecto.git   # Repositorio original
git fetch upstream
git merge upstream/main          # Traer novedades del original
```

### 13.3 Colaboradores

En un repositorio propio: **Settings → Collaborators → Add people** para dar permiso de escritura a los compañeros o al profesorado.

---

## 14. Otras herramientas útiles: `stash`, etiquetas y `rebase`

### 14.1 `git stash`: guardar cambios temporalmente

Útil para cambiar de rama sin hacer commit de un trabajo a medias:

```bash
git stash push -m "limpieza a medias"   # Guarda y deja el directorio limpio
git switch otra-rama
# ...
git switch rama-original
git stash list                          # Ver lo guardado
git stash pop                           # Recupera el último y lo borra de la lista
```

### 14.2 Etiquetas (*tags*): marcar versiones

```bash
git tag -a v1.0 -m "Entrega práctica 1"
git tag                                 # Listar
git push origin v1.0                    # Subir la etiqueta (o: git push --tags)
git switch --detach v1.0                # Ver el proyecto en esa versión
```

### 14.3 `rebase`: reescribir la base de una rama

```bash
git switch mi-rama
git rebase main          # Reaplica tus commits encima del último main (historia lineal)
```

> ⚠️ **Regla de oro:** no hagas `rebase` de commits que ya has compartido (subido a una rama en la que trabajan otros).

---

## 15. Git en proyectos de datos

### 15.1 Qué subir y qué no

| ✅ Sí a Git | ❌ No a Git |
|---|---|
| Código (`.py`, `.sql`, `.sh`) | Datasets grandes (`.csv`, `.parquet` de MB o GB) |
| Notebooks (preferiblemente sin salidas) | Credenciales, contraseñas, claves, `.env` |
| `environment.yml`, `requirements.txt`, `Dockerfile`, `compose.yaml` | Entornos virtuales (`venv/`, `envs/`) |
| Documentación (`README.md`) | Resultados que se pueden regenerar |
| Una **muestra pequeña** de datos para pruebas | `spark-warehouse/`, `metastore_db/`, *logs* |

GitHub **rechaza archivos de más de 100 MB** y avisa a partir de 50 MB. Los datos se guardan fuera de Git (almacenamiento compartido, HDFS, S3/MinIO…) y en el `README.md` se explica cómo obtenerlos.

Para archivos binarios grandes que **deban** versionarse existe **Git LFS** (*Large File Storage*):

```bash
sudo apt install -y git-lfs
git lfs install
git lfs track "*.parquet"
git add .gitattributes
```

### 15.2 Notebooks

Los `.ipynb` guardan las salidas (tablas, imágenes), lo que genera diferencias enormes y conflictos. Soluciones (detalladas en el [manual de JupyterLab](../jupyterlab/manual_jupyterlab.md), sección 15):

```bash
pip install nbstripout
nbstripout --install          # Limpia las salidas automáticamente en cada commit
```

### 15.3 Finales de línea (Windows ↔ Linux)

Windows usa `CRLF` y Linux `LF`. Si se mezclan, Git muestra archivos "modificados" sin cambios reales y los scripts `.sh` fallan en Linux. Añade un `.gitattributes` en la raíz del repositorio:

```gitattributes
* text=auto eol=lf
*.bat text eol=crlf
*.png binary
*.jpg binary
*.parquet binary
```

### 15.4 Si subes una contraseña por error

1. **Cámbiala inmediatamente** (revoca la clave o el *token*): borrarla del repositorio no basta, porque sigue en la historia.
2. Elimínala de la historia con herramientas como `git filter-repo` y avisa al resto del equipo.

---

## 16. Git en VS Code y JupyterLab

### VS Code

Con un repositorio abierto (`code .` desde WSL), el panel **Source Control** (`Ctrl + Shift + G`) permite:

- Ver los archivos modificados y sus diferencias (clic en el archivo).
- Preparar cambios (**+**), escribir el mensaje y hacer commit (✔).
- Cambiar de rama desde la barra de estado (abajo a la izquierda).
- *Sync Changes* (pull + push).
- Resolver conflictos con un editor visual.

### JupyterLab

Con la extensión `jupyterlab-git` (`pip install jupyterlab-git`) aparece un panel de Git en la barra lateral con las mismas operaciones básicas y diferencias visuales de notebooks.

> Las interfaces gráficas ayudan, pero **conviene dominar los comandos**: son iguales en cualquier equipo o servidor y permiten entender lo que pasa cuando algo falla.

---

## 17. Buenas prácticas

1. **Commits pequeños y frecuentes**, cada uno con **un solo cambio lógico**.
2. **Mensajes claros**, en imperativo, que expliquen *qué* y *por qué*:
   - ✅ `Añade filtro de ventas nulas en limpieza.py`
   - ✅ `Corrige ruta relativa del dataset de clientes`
   - ❌ `cambios`, `arreglo`, `asdf`, `versión final definitiva 2`
3. Primera línea del mensaje de **menos de 50–70 caracteres**; si hace falta, una línea en blanco y después la explicación detallada.
4. **`git status` y `git diff --staged` antes de cada commit.**
5. **`git pull` antes de empezar** a trabajar y **antes de `git push`**.
6. **Una rama por tarea**; `main` siempre debe funcionar.
7. **Nunca** subas credenciales ni datasets grandes; crea el `.gitignore` **al principio** del proyecto.
8. Incluye un **`README.md`** que explique qué hace el proyecto, cómo instalar el entorno y cómo ejecutarlo.
9. No reescribas historia compartida (`reset`, `rebase`, `push --force`).

Estructura recomendada:

```
proyecto-ventas/
├── .gitignore
├── .gitattributes
├── README.md
├── environment.yml
├── datos/              ← ignorada (salvo datos/muestra/)
├── notebooks/
├── src/
└── tests/
```

---

## 18. Solución de problemas frecuentes

| Problema | Causa probable | Solución |
|---|---|---|
| `Please tell me who you are` | Identidad no configurada | `git config --global user.name ...` y `user.email ...` |
| `Support for password authentication was removed` | GitHub no acepta contraseñas por HTTPS | Usar SSH, `gh auth login` o el gestor de credenciales (sección 12) |
| `Permission denied (publickey)` | Clave SSH no creada o no añadida a GitHub | Repetir la sección 12.1; comprobar con `ssh -T git@github.com` |
| `rejected ... (fetch first)` / `non-fast-forward` al hacer push | El remoto tiene commits que tú no tienes | `git pull`, resolver conflictos si los hay, y `git push` |
| `fatal: not a git repository` | No estás en la carpeta del repositorio | `cd` a la carpeta correcta, o `git init` |
| `src refspec main does not match any` | Aún no hay commits, o la rama se llama `master` | Hacer un primer commit; `git branch` para ver el nombre de la rama |
| `fatal: refusing to merge unrelated histories` | Repo local y remoto creados por separado (p. ej., con README en GitHub) | `git pull origin main --allow-unrelated-histories` |
| `Your local changes would be overwritten` | Cambios sin guardar al cambiar de rama o hacer pull | `git stash`, hacer la operación y `git stash pop` (o hacer commit) |
| Mensaje en un editor extraño (Vim) tras `git commit` o `merge` | Editor por defecto | En Vim: `Esc`, `:wq` e `Intro`. Cambiar el editor: `git config --global core.editor nano` |
| Todos los archivos aparecen modificados sin cambios | Finales de línea o permisos (repo en `/mnt/c`) | `.gitattributes` (15.3), `git config core.fileMode false`, trabajar en `~/` |
| Archivo en `.gitignore` que sigue apareciendo | Ya estaba seguido | `git rm --cached <archivo>` y commit |
| `File ... exceeds GitHub's file size limit of 100 MB` | Archivo grande en algún commit | Quitarlo con `git rm --cached`, `git commit --amend` si es el último commit; añadirlo al `.gitignore` |
| He perdido commits tras un `reset` | Historia movida | `git reflog` y `git reset --hard <hash>` |
| Estado *detached HEAD* | Hiciste `switch --detach` o `checkout <hash>` | `git switch main`; si hiciste commits, antes `git switch -c rama-nueva` |

Diagnóstico general:

```bash
git status
git log --oneline --graph --all -10
git remote -v
git config --list --show-origin
```

---

## 19. Chuleta rápida

```bash
# ---------- Configuración (una vez) ----------
git config --global user.name "Nombre Apellido"
git config --global user.email "correo@ejemplo.com"
git config --global init.defaultBranch main

# ---------- Empezar ----------
git init                              # Nuevo repositorio
git clone <url>                       # Copiar uno existente

# ---------- Día a día ----------
git status                            # ¿Qué ha cambiado?
git diff                              # Ver cambios
git add <archivo> | git add .         # Preparar
git commit -m "Mensaje"               # Confirmar
git log --oneline --graph --all       # Historia

# ---------- Ramas ----------
git switch -c <rama>                  # Crear y cambiar
git switch <rama>                     # Cambiar
git merge <rama>                      # Fusionar en la actual
git branch -d <rama>                  # Borrar

# ---------- Remoto ----------
git remote add origin <url>           # Conectar
git push -u origin main               # Primera subida
git pull                              # Traer y fusionar
git push                              # Subir

# ---------- Deshacer ----------
git restore <archivo>                 # Descartar cambios (⚠️)
git restore --staged <archivo>        # Quitar de preparación
git commit --amend                    # Corregir último commit (no subido)
git revert <hash>                     # Deshacer commit ya subido
git stash / git stash pop             # Guardar y recuperar trabajo a medias
git reflog                            # Red de seguridad
```

---

## 20. Referencias

- Documentación oficial de Git: <https://git-scm.com/doc>
- Libro *Pro Git* (gratuito, en español): <https://git-scm.com/book/es/v2>
- Git en WSL (Microsoft Learn): <https://learn.microsoft.com/es-es/windows/wsl/tutorials/wsl-git>
- GitHub Docs – Primeros pasos: <https://docs.github.com/es/get-started>
- GitHub Docs – Conectar con SSH: <https://docs.github.com/es/authentication/connecting-to-github-with-ssh>
- GitHub CLI: <https://cli.github.com/>
- Plantillas de `.gitignore`: <https://github.com/github/gitignore>
- Git LFS: <https://git-lfs.com/>
- Práctica interactiva de ramas: <https://learngitbranching.js.org/?locale=es_ES>
