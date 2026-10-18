# Manual de conocimientos informáticos previos

**Módulo:** MP5074 – Sistemas de Big Data · Curso 2026-27

> Casi todos los errores de las primeras semanas no son de Python ni de pandas: son del tipo «no encuentra el fichero» o «no reconoce el comando». Los dos tienen la misma raíz: no saber **dónde** está algo ni **cómo** lo busca el sistema. Este manual repasa el sistema de ficheros, las rutas, las extensiones y las variables de entorno para que las herramientas que vienen después (WSL, bash, conda, Jupyter) no parezcan magia.

---

## Índice

1. [¿Por qué empezar por aquí?](#1-por-qué-empezar-por-aquí)
2. [Conceptos clave](#2-conceptos-clave)
3. [El sistema de ficheros es un árbol](#3-el-sistema-de-ficheros-es-un-árbol)
4. [Rutas absolutas y relativas](#4-rutas-absolutas-y-relativas)
5. [Moverse por el árbol desde la terminal](#5-moverse-por-el-árbol-desde-la-terminal)
6. [Rutas en Python y en los notebooks](#6-rutas-en-python-y-en-los-notebooks)
7. [Extensiones de fichero y codificación](#7-extensiones-de-fichero-y-codificación)
8. [Variables de entorno](#8-variables-de-entorno)
9. [`PATH`: cómo encuentra el sistema los programas](#9-path-cómo-encuentra-el-sistema-los-programas)
10. [Solución de problemas frecuentes](#10-solución-de-problemas-frecuentes)
11. [Chuleta rápida](#11-chuleta-rápida)
12. [Referencias](#12-referencias)
13. [Ejemplo resuelto: el notebook que no encuentra el CSV](#13-ejemplo-resuelto-el-notebook-que-no-encuentra-el-csv)

---

## 1. ¿Por qué empezar por aquí?

Dos mensajes de error aparecen una y otra vez al empezar:

| Error | Qué significa en realidad |
|---|---|
| `FileNotFoundError` / *No such file or directory* | La **ruta** no lleva al fichero desde la carpeta en la que estás |
| *command not found* / *«python» no se reconoce como un comando…* | El sistema no encuentra el programa en ninguna carpeta del **`PATH`** |

Entender las ideas de este manual resuelve la mayoría de esos problemas sin tener que buscar en Internet.

---

## 2. Conceptos clave

| Concepto | Explicación breve |
|---|---|
| Sistema de ficheros | Organización en árbol de carpetas y ficheros dentro de un disco. |
| Raíz | Carpeta de la que cuelga todo el árbol (`/` en Linux, `C:\` en Windows). |
| Directorio de trabajo | Carpeta desde la que se interpretan las rutas relativas. |
| Ruta absoluta | Ruta que empieza en la raíz; vale desde cualquier sitio. |
| Ruta relativa | Ruta que parte de la carpeta actual; usa `.` y `..`. |
| Extensión | Sufijo del nombre que indica el tipo esperado del fichero. |
| Codificación | Regla que traduce bytes a caracteres (UTF-8, Latin-1). |
| Variable de entorno | Par nombre=valor que el sistema pasa a los programas. |
| `PATH` | Lista ordenada de carpetas donde se buscan los ejecutables. |

---

## 3. El sistema de ficheros es un árbol

Un **sistema de ficheros** organiza los datos del disco en **carpetas** (directorios) que contienen ficheros y otras carpetas. El resultado es un árbol invertido: arriba está la **raíz** y de ella cuelga todo lo demás.

| | Windows | Linux |
|---|---|---|
| Raíz | Una por unidad: `C:\`, `D:\` | Una sola: `/` |
| Separador | Barra invertida `\` | Barra `/` |
| Carpeta personal | `C:\Users\ana` | `/home/ana` (abreviada `~`) |
| Mayúsculas | No las distingue (`Datos.csv` = `datos.csv`) | Sí las distingue: son dos ficheros distintos |
| Otros discos | Otra letra (`D:\`) | Se «montan» dentro del árbol (`/mnt/d`) |

> **Con WSL:** el disco `C:` de Windows aparece dentro de Linux como `/mnt/c`, y el disco de Linux se ve desde Windows como `\\wsl$\Ubuntu\...`. Consulta el [manual de WSL](../manuales/wsl/manual_wsl.md) para más detalle.

---

## 4. Rutas absolutas y relativas

Una **ruta** es la dirección de un fichero dentro del árbol.

| Tipo | Empieza en… | Ejemplo | ¿Vale desde cualquier sitio? |
|---|---|---|---|
| **Absoluta** | La raíz | `/home/ana/mp5074/datos/ventas.csv` · `C:\Users\ana\mp5074\datos\ventas.csv` | Sí |
| **Relativa** | El **directorio de trabajo actual** | `datos/ventas.csv` | Solo si estás dentro de `mp5074` |

### 4.1 Atajos en las rutas

| Símbolo | Significa |
|---|---|
| `.` | La carpeta actual |
| `..` | La carpeta padre (un nivel arriba) |
| `~` | La carpeta personal del usuario (en Linux y en PowerShell) |

**Analogía:** una ruta absoluta es la dirección postal completa («calle Mayor 3, A Coruña»); una relativa es una indicación desde donde estás («la segunda puerta a la izquierda»). La segunda es más corta, pero solo sirve si sabes dónde estás.

### 4.2 En los proyectos, rutas relativas

> En un notebook o un script, usa rutas **relativas a la carpeta del proyecto** (`data/ventas.csv`). Si escribes `C:\Users\ana\...`, el código no funcionará en el ordenador de nadie más.

---

## 5. Moverse por el árbol desde la terminal

| Acción | Linux (bash) | Windows (PowerShell) |
|---|---|---|
| ¿Dónde estoy? | `pwd` | `Get-Location` (o `pwd`) |
| Listar el contenido | `ls -la` | `Get-ChildItem` (o `dir`, `ls`) |
| Entrar en una carpeta | `cd datos` | `cd datos` |
| Subir un nivel | `cd ..` | `cd ..` |
| Ir a la carpeta personal | `cd ~` (o solo `cd`) | `cd ~` |
| Ver el árbol | `tree` (instalar con `sudo apt install tree`) | `tree /F` |

> Truco: pulsa `Tab` mientras escribes una ruta y la terminal la completa. Si no completa, es que esa ruta no existe desde donde estás.

El manejo de bash se trata a fondo más adelante.

---

## 6. Rutas en Python y en los notebooks

En Python, las rutas relativas se interpretan desde el **directorio de trabajo del proceso**, que en un notebook de Jupyter suele ser la carpeta donde está el `.ipynb`.

```python
import os
os.getcwd()          # Directorio de trabajo actual
os.listdir(".")      # Contenido de la carpeta actual
```

Con `pathlib` (recomendado) las rutas funcionan igual en Windows y en Linux:

```python
from pathlib import Path

Path.cwd()                                  # Directorio de trabajo actual
ruta = Path("..") / "datos" / "ventas.csv"  # Construir rutas sin preocuparse del separador
ruta.exists()                               # ¿Existe?
ruta.resolve()                              # Ruta absoluta equivalente
```

> Comprueba siempre `Path.cwd()` antes de culpar al fichero: casi siempre el problema es desde dónde se ejecuta el código.

---

## 7. Extensiones de fichero y codificación

La **extensión** es la parte del nombre que va detrás del último punto (`.csv`, `.ipynb`, `.py`). El sistema la usa para decidir con qué programa abrir el fichero, pero **no cambia su contenido**: renombrar `ventas.xlsx` a `ventas.csv` no lo convierte en CSV.

| Tipo | Características | Ejemplos |
|---|---|---|
| **Ficheros de texto** | Se pueden abrir con cualquier editor y leer. Importa la **codificación** (UTF-8, Latin-1), que dice cómo se guardan las letras con tilde | `.csv`, `.json`, `.py`, `.md`, `.txt`, `.ipynb` |
| **Ficheros binarios** | Solo los entiende su programa | `.xlsx`, `.parquet`, `.png`, `.zip` |

> **Activa las extensiones en Windows.** Windows las oculta por defecto. En el Explorador: *Vista → Mostrar → Extensiones de nombre de archivo*. Así evitarás ficheros llamados `datos.csv.txt`.

Si al leer un CSV aparecen caracteres raros (`GalicÃ­a`) o un `UnicodeDecodeError`, el problema es la codificación:

```python
pd.read_csv("datos/ventas.csv", encoding="latin-1")
```

---

## 8. Variables de entorno

Una **variable de entorno** es un par nombre=valor que el sistema operativo pasa a todos los programas que se ejecutan. Sirven para configurar sin tocar el código: dónde está la carpeta personal, qué idioma usar o dónde buscar programas.

| Variable | Contenido | Ver en Linux | Ver en PowerShell |
|---|---|---|---|
| `HOME` / `USERPROFILE` | Carpeta personal | `echo $HOME` | `$env:USERPROFILE` |
| `USER` / `USERNAME` | Usuario actual | `echo $USER` | `$env:USERNAME` |
| `PATH` | Carpetas donde buscar programas | `echo $PATH` | `$env:Path` |

### 8.1 Crear y hacer permanente una variable (Linux)

```bash
export NOMBRE=valor                      # Solo para la sesión actual
echo $NOMBRE
echo 'export NOMBRE=valor' >> ~/.bashrc  # Permanente: ~/.bashrc se ejecuta al abrir cada terminal
source ~/.bashrc                         # Aplicar el cambio sin cerrar la terminal
```

### 8.2 Listar todas las variables

```bash
env                 # Linux
```

```powershell
Get-ChildItem Env:  # PowerShell
```

---

## 9. `PATH`: cómo encuentra el sistema los programas

Cuando escribes `python` en la terminal, el sistema no busca en todo el disco. Recorre, **en orden**, las carpetas de la variable `PATH` y ejecuta el **primer** `python` que encuentra. Si no lo encuentra en ninguna, aparece el error *command not found* (o *«python» no se reconoce como un comando…*).

| | Linux | Windows |
|---|---|---|
| Separador entre carpetas | `:` | `;` |
| ¿Qué programa se ejecutará? | `which python` | `Get-Command python` |

**Consecuencias prácticas**

1. Instalar un programa no basta: su carpeta tiene que estar en `PATH`.
2. Si hay dos versiones instaladas, gana la que esté **antes** en `PATH`. Eso es exactamente lo que hace `conda activate`: pone delante la carpeta del entorno (ver el [manual de Miniconda](../manuales/miniconda/manual_miniconda.md)).

Añadir una carpeta al principio del `PATH` en Linux:

```bash
export PATH="$HOME/bin:$PATH"
```

> ⚠️ Escribe siempre `:$PATH` al final. Si haces `export PATH="$HOME/bin"`, borras el resto de carpetas y dejarán de funcionar `ls`, `cd`… hasta que abras una terminal nueva.

---

## 10. Solución de problemas frecuentes

| Problema | Causa probable | Solución |
|---|---|---|
| `FileNotFoundError` al leer un fichero | Ruta relativa escrita desde otra carpeta | Comprobar `os.getcwd()` y ajustar la ruta con `..` |
| El código funciona en mi equipo y no en el de un compañero | Ruta absoluta (`C:\Users\...`) en el código | Usar rutas relativas a la carpeta del proyecto |
| `Datos.csv` no se encuentra en Linux pero sí en Windows | Linux distingue mayúsculas y minúsculas | Escribir el nombre exacto |
| Fichero llamado `datos.csv.txt` | Extensiones ocultas en Windows | Activar *Mostrar extensiones* en el Explorador |
| Caracteres raros (`Ã³`) o `UnicodeDecodeError` | Codificación distinta de UTF-8 | `encoding="latin-1"` (o la que corresponda) |
| *command not found* tras instalar algo | La carpeta del programa no está en `PATH` (o el entorno no está activado) | `which <programa>`, `echo $PATH`, `conda activate <entorno>` |
| Se ejecuta otra versión del programa | Otra carpeta aparece antes en `PATH` | `which -a <programa>` para ver todas y ajustar el orden |
| Una variable creada con `export` desaparece | Solo dura la sesión actual | Añadirla a `~/.bashrc` |

---

## 11. Chuleta rápida

```bash
# ---------- Dónde estoy y qué hay (Linux) ----------
pwd                       # Directorio de trabajo
ls -la                    # Contenido (incluidos ocultos)
cd ..                     # Subir un nivel
cd ~                      # Carpeta personal
/mnt/c/Users/<usuario>    # Disco C: de Windows desde WSL

# ---------- Variables de entorno ----------
echo $HOME                # Ver una variable
export NOMBRE=valor       # Crear (sesión actual)
echo 'export NOMBRE=valor' >> ~/.bashrc   # Permanente

# ---------- PATH ----------
echo $PATH                # Carpetas donde se buscan programas
which python              # Qué python se ejecutará
export PATH="$HOME/bin:$PATH"   # Añadir carpeta al principio
```

```powershell
# ---------- PowerShell (Windows) ----------
Get-Location              # Directorio de trabajo
$env:USERPROFILE          # Carpeta personal
$env:Path                 # PATH
Get-Command python        # Qué python se ejecutará
```

```python
# ---------- Python ----------
from pathlib import Path
Path.cwd()                                # Directorio de trabajo
Path("..") / "datos" / "ventas.csv"       # Construir rutas
```

---

## 12. Referencias

- Formatos de ruta de acceso de archivo en Windows (Microsoft Learn): <https://learn.microsoft.com/es-es/dotnet/standard/io/file-path-formats>
- Trabajar con sistemas de archivos de Windows y Linux en WSL: <https://learn.microsoft.com/es-es/windows/wsl/filesystems>
- Variables de entorno en PowerShell (`about_Environment_Variables`): <https://learn.microsoft.com/es-es/powershell/module/microsoft.powershell.core/about/about_environment_variables>
- Manual de GNU Bash: <https://www.gnu.org/software/bash/manual/>
- Python, módulo `pathlib`: <https://docs.python.org/es/3/library/pathlib.html>
- Python, módulo `os`: <https://docs.python.org/es/3/library/os.html>

---

## 13. Ejemplo resuelto: el notebook que no encuentra el CSV

**Situación.** Tienes este proyecto en WSL:

```text
/home/ana/mp5074/
├── datos/
│   └── ventas.csv
└── notebooks/
    └── analisis.ipynb
```

El notebook `analisis.ipynb` se ejecuta con la carpeta `notebooks` como directorio de trabajo, y la celda `pd.read_csv("ventas.csv")` falla con `FileNotFoundError`.

**Paso 1. Averiguar dónde estás.** En una celda del notebook:

```python
import os
os.getcwd()
```

```text
'/home/ana/mp5074/notebooks'
```

**Paso 2. Escribir la ruta relativa correcta.** Desde `notebooks` hay que subir un nivel (`..`) y entrar en `datos`:

```python
import pandas as pd
df = pd.read_csv("../datos/ventas.csv")
```

**Paso 3. Comprobar la ruta absoluta equivalente.** `/home/ana/mp5074/notebooks/../datos/ventas.csv` se simplifica a `/home/ana/mp5074/datos/ventas.csv`, porque `notebooks/..` se anula. Funcionaría, pero solo en el ordenador de Ana.

**Paso 4. El mismo fichero visto desde Windows.** WSL guarda su disco aparte; Windows lo ve a través de la red como `\\wsl$\Ubuntu\home\ana\mp5074\datos\ventas.csv`. Y al revés: un fichero de `C:\Users\ana\Descargas\ventas.csv` se ve desde Linux como `/mnt/c/Users/ana/Descargas/ventas.csv`.

**Paso 5. Un problema de `PATH`.** En la terminal escribes `jupyter lab` y aparece `jupyter: command not found`. Comprobamos:

```bash
which jupyter        # no devuelve nada → no está en ninguna carpeta del PATH
echo $PATH           # /usr/local/bin:/usr/bin:/bin...
conda activate bigdata
which jupyter        # /home/ana/miniconda3/envs/bigdata/bin/jupyter
```

Al activar el entorno, conda añade `/home/ana/miniconda3/envs/bigdata/bin` al principio de `PATH`, y el sistema ya encuentra `jupyter`. El error no era que Jupyter no estuviera instalado: era que el sistema no sabía dónde buscarlo.
