# Manual de JupyterLab

**Módulo:** MP5074 – Sistemas de Big Data · Curso 2026-27

> JupyterLab es un entorno de trabajo web para crear **notebooks**: documentos que mezclan código ejecutable, resultados, gráficos y texto explicativo. Es la herramienta estándar para explorar y analizar datos con Python, pandas y PySpark. En este módulo lo ejecutaremos dentro de **WSL**, en un entorno de **Miniconda** o en un contenedor **Docker** (ver los manuales de [WSL](../wsl/manual_wsl.md), [Miniconda](../miniconda/manual_miniconda.md) y [Docker](../docker/manual_docker.md)).

---

## Índice

1. [¿Qué es JupyterLab?](#1-qué-es-jupyterlab)
2. [Conceptos clave](#2-conceptos-clave)
3. [Instalación](#3-instalación)
4. [Arrancar y detener JupyterLab](#4-arrancar-y-detener-jupyterlab)
5. [La interfaz](#5-la-interfaz)
6. [Trabajar con notebooks: celdas y modos](#6-trabajar-con-notebooks-celdas-y-modos)
7. [Atajos de teclado](#7-atajos-de-teclado)
8. [Kernels: elegir el entorno de ejecución](#8-kernels-elegir-el-entorno-de-ejecución)
9. [Celdas Markdown: documentar el análisis](#9-celdas-markdown-documentar-el-análisis)
10. [Comandos mágicos y comandos del sistema](#10-comandos-mágicos-y-comandos-del-sistema)
11. [Datos y gráficos en el notebook](#11-datos-y-gráficos-en-el-notebook)
12. [PySpark en JupyterLab](#12-pyspark-en-jupyterlab)
13. [Herramientas integradas y extensiones](#13-herramientas-integradas-y-extensiones)
14. [Exportar y compartir notebooks](#14-exportar-y-compartir-notebooks)
15. [Notebooks y Git](#15-notebooks-y-git)
16. [Configuración y seguridad](#16-configuración-y-seguridad)
17. [Buenas prácticas](#17-buenas-prácticas)
18. [Solución de problemas frecuentes](#18-solución-de-problemas-frecuentes)
19. [Chuleta rápida](#19-chuleta-rápida)
20. [Referencias](#20-referencias)

---

## 1. ¿Qué es JupyterLab?

**Proyecto Jupyter** es un proyecto de código abierto cuyo nombre viene de **Ju**lia, **Pyt**hon y **R**, los primeros lenguajes que soportó. Incluye varias herramientas:

| Herramienta | Descripción |
|---|---|
| **JupyterLab** (la que usaremos) | Entorno completo: notebooks, terminal, editor de archivos, explorador, paneles organizables |
| Jupyter Notebook | Interfaz clásica, más sencilla, un notebook por pestaña del navegador |
| JupyterHub | Servidor multiusuario (un JupyterLab por alumno en un servidor del centro) |
| nbconvert | Conversión de notebooks a HTML, PDF, script… |

**¿Por qué notebooks en Big Data?**

- Permiten **explorar datos paso a paso**: ejecutar un trozo, ver el resultado y seguir.
- El código, los resultados y las explicaciones quedan en **un único documento**, ideal para entregar prácticas.
- Muestran tablas y gráficos directamente.
- Se integran con pandas, PySpark, SQL, bases de datos…

La versión actual es la serie **JupyterLab 4.x**.

---

## 2. Conceptos clave

- **Notebook (`.ipynb`):** archivo con una lista de celdas y sus resultados. Internamente es un JSON.
- **Celda:** bloque del notebook. Puede ser de **código**, **Markdown** (texto) o **Raw** (texto sin procesar).
- **Kernel:** el proceso que **ejecuta el código** del notebook (por ejemplo, un intérprete de Python de un entorno conda). Cada notebook abierto tiene su propio kernel.
- **Servidor de Jupyter:** el programa que se lanza con `jupyter lab`; sirve la interfaz web y gestiona los kernels.
- **Estado del kernel:** las variables viven en memoria **mientras el kernel esté activo**. Si se reinicia el kernel, hay que volver a ejecutar las celdas.
- **Orden de ejecución:** el número entre corchetes `[5]` junto a una celda indica en qué orden se ejecutó. **No tiene por qué coincidir con el orden visual**, y esto es una fuente habitual de errores.

```
Navegador (Windows)          Servidor Jupyter (WSL)           Kernels
┌──────────────────┐  HTTP  ┌─────────────────────┐   ┌───────────────────┐
│   JupyterLab     │◀──────▶│  jupyter lab        │──▶│ Python (bigdata)  │
│ localhost:8888   │        │  (gestiona archivos │──▶│ Python (pruebas)  │
└──────────────────┘        │   y kernels)        │   └───────────────────┘
                            └─────────────────────┘
```

---

## 3. Instalación

Elige **una** de estas opciones.

### 3.1 Con Miniconda (recomendada en el módulo)

En la terminal de Ubuntu (WSL):

```bash
conda activate bigdata
conda install -c conda-forge jupyterlab ipykernel
jupyter lab --version
```

(Si has creado el entorno con el `environment.yml` del manual de Miniconda, JupyterLab ya está instalado.)

### 3.2 Con `pip` y un entorno virtual

```bash
python3 -m venv ~/bigdata/venv
source ~/bigdata/venv/bin/activate
pip install jupyterlab
```

### 3.3 Con Docker (sin instalar nada)

```bash
mkdir -p ~/bigdata/notebooks
docker run -d --name jupyter -p 8888:8888 \
  -v ~/bigdata/notebooks:/home/jovyan/work \
  quay.io/jupyter/pyspark-notebook
docker logs jupyter 2>&1 | grep "127.0.0.1:8888"
```

Hay varias imágenes oficiales según lo que necesites: `base-notebook`, `scipy-notebook` (pandas, matplotlib…), `pyspark-notebook` (añade Spark), etc.

### 3.4 JupyterLab Desktop (alternativa)

Existe una aplicación de escritorio para Windows (<https://github.com/jupyterlab/jupyterlab-desktop>). Es cómoda para empezar, pero en el módulo trabajaremos desde WSL.

---

## 4. Arrancar y detener JupyterLab

### 4.1 Arrancar

```bash
conda activate bigdata
cd ~/bigdata                 # La carpeta desde la que arrancas será la raíz del explorador
jupyter lab --no-browser
```

En la terminal aparecerá algo como:

```
    To access the server, open this file in a browser:
        ...
    Or copy and paste one of these URLs:
        http://localhost:8888/lab?token=3f9a1c...
```

Copia la URL completa (con el *token*) en el navegador de Windows. En WSL, `localhost` funciona directamente.

> Deja **abierta** la terminal donde se ejecuta JupyterLab: si la cierras, se detiene el servidor.

Opciones útiles:

| Opción | Uso |
|---|---|
| `--no-browser` | No intenta abrir el navegador (recomendable en WSL) |
| `--port 8890` | Usa otro puerto (si el 8888 está ocupado) |
| `--notebook-dir ~/bigdata` | Carpeta raíz, sin tener que hacer `cd` antes |

### 4.2 Consultar servidores en marcha

```bash
jupyter server list
```

Muestra las URLs con su *token* (útil si has perdido la URL).

### 4.3 Detener

- En la terminal del servidor: `Ctrl + C` y confirma con `y`.
- O desde la interfaz: **File → Shut Down**.

---

## 5. La interfaz

```
┌───────────────────────────────────────────────────────────────────┐
│ File  Edit  View  Run  Kernel  Tabs  Settings  Help   ← Menú      │
├────┬──────────────────┬───────────────────────────────────────────┤
│ 📁 │                  │ [analisis.ipynb] [datos.csv] [Terminal 1] │
│ ⏹  │  Barra lateral   │                                           │
│ 📑 │  (explorador de  │           Área principal                  │
│ 🧩 │   archivos,      │    (notebooks, editores, terminales,      │
│    │   kernels…)      │     visores; se pueden dividir en         │
│    │                  │     paneles arrastrando las pestañas)     │
├────┴──────────────────┴───────────────────────────────────────────┤
│ Barra de estado: kernel, estado (Idle/Busy), línea/columna…       │
└───────────────────────────────────────────────────────────────────┘
```

### 5.1 Barra lateral izquierda

| Icono | Panel | Para qué sirve |
|---|---|---|
| 📁 | **Explorador de archivos** | Crear, abrir, renombrar, subir (arrastrando) y descargar archivos |
| ⏹ | **Kernels y terminales en ejecución** | Ver y cerrar kernels abiertos (libera memoria) |
| 📑 | **Índice (*Table of Contents*)** | Navegar por los títulos Markdown del notebook |
| 🧩 | **Extensiones** | Gestor de extensiones |

`Ctrl + B` muestra u oculta la barra lateral.

La **barra lateral derecha** incluye el **inspector de propiedades** (metadatos de celdas) y el **depurador**.

### 5.2 El *Launcher*

Se abre con el botón **+** o `Ctrl + Shift + L`. Desde él se crea:

- Un **Notebook** con el kernel elegido.
- Una **Console** (intérprete interactivo).
- Una **Terminal** (shell de Linux dentro del navegador).
- Un archivo de texto, Markdown o Python.

### 5.3 Organizar el espacio de trabajo

- Arrastra una pestaña hacia un borde del área principal para **dividir la pantalla** (p. ej., notebook a la izquierda y CSV a la derecha).
- Clic derecho sobre un notebook → **New View for Notebook**: dos vistas del mismo notebook.
- Clic derecho en una celda de salida → **Create New View for Cell Output**: deja un gráfico siempre visible.
- **Paleta de comandos:** `Ctrl + Shift + C`. Permite buscar cualquier acción por su nombre.

---

## 6. Trabajar con notebooks: celdas y modos

### 6.1 Tipos de celda

| Tipo | Contenido | Tecla (modo comando) |
|---|---|---|
| **Code** | Código que ejecuta el kernel | `Y` |
| **Markdown** | Texto con formato, títulos, tablas, fórmulas | `M` |
| **Raw** | Texto que no se procesa | `R` |

### 6.2 Modos de trabajo

| Modo | Cómo se reconoce | Cómo se entra | Para qué |
|---|---|---|---|
| **Edición** | Cursor dentro de la celda | `Enter` o clic dentro de la celda | Escribir en la celda |
| **Comando** | Barra azul a la izquierda, sin cursor | `Esc` o clic fuera del texto | Manejar celdas con atajos (crear, borrar, mover…) |

### 6.3 Ejecutar celdas

- `Shift + Enter` → ejecuta y pasa a la siguiente.
- `Ctrl + Enter` → ejecuta y se queda en la celda.
- `Alt + Enter` → ejecuta e inserta una celda nueva debajo.
- Menú **Run → Run All Cells** → ejecuta el notebook completo.

El indicador junto a la celda muestra su estado: `[ ]` sin ejecutar, `[*]` ejecutándose, `[7]` ejecutada (séptima ejecución).

La última expresión de una celda de código **se muestra automáticamente** como resultado:

```python
x = 10
y = 32
x + y        # Se muestra: 42
```

### 6.4 Gestionar el kernel (menú *Kernel*)

| Acción | Cuándo usarla |
|---|---|
| **Interrupt** (`I`, `I`) | Una celda tarda demasiado o está en bucle infinito |
| **Restart** (`0`, `0`) | Empezar de cero (borra todas las variables) |
| **Restart and Run All** | Comprobar que el notebook funciona **de principio a fin**. ✅ Hazlo siempre antes de entregar |
| **Change Kernel** | Usar otro entorno (ver sección 8) |
| **Shut Down** | Cerrar el kernel y liberar memoria |

---

## 7. Atajos de teclado

### Modo comando (`Esc`)

| Atajo | Acción |
|---|---|
| `A` / `B` | Insertar celda **encima** / **debajo** |
| `D`, `D` | Borrar celda |
| `Z` | Deshacer operación de celda (p. ej., un borrado) |
| `C` / `X` / `V` | Copiar / cortar / pegar celda |
| `M` / `Y` / `R` | Convertir a Markdown / código / raw |
| `Shift + M` | Unir celdas seleccionadas |
| `↑` `↓` o `K` `J` | Moverse entre celdas |
| `Shift + ↑/↓` | Seleccionar varias celdas |
| `I`, `I` | Interrumpir el kernel |
| `0`, `0` | Reiniciar el kernel |
| `Shift + L` | Mostrar/ocultar números de línea |
| `Enter` | Pasar a modo edición |

### Modo edición (`Enter`)

| Atajo | Acción |
|---|---|
| `Tab` | Autocompletar |
| `Shift + Tab` | Ver la documentación de la función bajo el cursor |
| `Ctrl + /` | Comentar/descomentar líneas |
| `Ctrl + Shift + -` | Dividir la celda en el cursor |
| `Esc` | Pasar a modo comando |

### Globales

| Atajo | Acción |
|---|---|
| `Shift + Enter` | Ejecutar celda y avanzar |
| `Ctrl + Enter` | Ejecutar celda |
| `Ctrl + S` | Guardar |
| `Ctrl + B` | Mostrar/ocultar barra lateral |
| `Ctrl + Shift + C` | Paleta de comandos |
| `Ctrl + Shift + L` | Nuevo *Launcher* |

> Todos los atajos se pueden consultar y cambiar en **Settings → Settings Editor → Keyboard Shortcuts**.

---

## 8. Kernels: elegir el entorno de ejecución

JupyterLab puede estar instalado en un entorno y **ejecutar código con otro**. Para que un entorno conda aparezca como kernel:

```bash
conda activate bigdata
conda install ipykernel
python -m ipykernel install --user --name bigdata --display-name "Python (bigdata)"
```

Desde ese momento, "Python (bigdata)" aparece en el *Launcher* y en **Kernel → Change Kernel**.

| Comando | Descripción |
|---|---|
| `jupyter kernelspec list` | Lista los kernels registrados |
| `jupyter kernelspec uninstall bigdata` | Elimina un kernel |

**Comprobar qué Python usa el notebook:**

```python
import sys
print(sys.executable)   # Debe ser .../miniconda3/envs/bigdata/bin/python
```

---

## 9. Celdas Markdown: documentar el análisis

Un buen notebook **explica** lo que hace. Sintaxis básica:

```markdown
# Título 1
## Título 2
### Título 3

Texto normal, **negrita**, *cursiva* y `código`.

- Elemento de lista
- Otro elemento

1. Paso uno
2. Paso dos

[Enlace](https://spark.apache.org)

![Imagen](imagenes/arquitectura.png)

| Columna A | Columna B |
|-----------|-----------|
| 1         | 2         |

> Nota o cita destacada
```

**Fórmulas matemáticas (LaTeX):**

```markdown
La media es $\bar{x} = \frac{1}{n}\sum_{i=1}^{n} x_i$

$$
\sigma = \sqrt{\frac{1}{n}\sum_{i=1}^{n}(x_i - \bar{x})^2}
$$
```

**Bloques de código con resaltado:**

````markdown
```python
df.groupBy("ciudad").count().show()
```
````

Los títulos (`#`, `##`…) generan automáticamente el **índice** en la barra lateral y permiten **plegar secciones** con la flecha que aparece a su izquierda.

---

## 10. Comandos mágicos y comandos del sistema

Los **comandos mágicos** son órdenes especiales de IPython (el kernel de Python):

- `%comando` → actúa sobre **una línea**.
- `%%comando` → actúa sobre **toda la celda** (debe ir en la primera línea).

| Comando | Qué hace |
|---|---|
| `%timeit sum(range(1000))` | Mide el tiempo medio de una línea (la ejecuta muchas veces) |
| `%%time` | Mide el tiempo de ejecución de la celda completa |
| `%pwd` / `%cd ruta` | Directorio actual / cambiar de directorio |
| `%ls` | Lista archivos |
| `%run script.py` | Ejecuta un script de Python |
| `%load script.py` | Carga el contenido de un script en la celda |
| `%who` / `%whos` | Variables definidas en el kernel |
| `%env` | Variables de entorno |
| `%%writefile fichero.py` | Guarda el contenido de la celda en un archivo |
| `%pip install paquete` | Instala un paquete **en el kernel actual** |
| `%conda install paquete` | Igual, con conda |
| `%load_ext autoreload` + `%autoreload 2` | Recarga automáticamente módulos propios al modificarlos |
| `%lsmagic` | Lista todos los comandos mágicos |

**Comandos del sistema** con `!`:

```python
!ls -lh datos/
!head -n 5 datos/ventas.csv
!wc -l datos/ventas.csv
```

> Usa `%pip install` y **no** `!pip install`: `%pip` instala siempre en el entorno del kernel, mientras que `!pip` puede instalar en otro Python.

---

## 11. Datos y gráficos en el notebook

### 11.1 Tablas con pandas

```python
import pandas as pd

pd.set_option("display.max_columns", 50)   # Columnas visibles
pd.set_option("display.max_rows", 100)     # Filas visibles

df = pd.read_csv("datos/ventas.csv")
df.head()          # Se muestra como tabla HTML
```

Otras funciones útiles para explorar: `df.info()`, `df.describe()`, `df.shape`, `df.dtypes`, `df.isna().sum()`.

### 11.2 Gráficos con matplotlib

```python
import matplotlib.pyplot as plt

ventas = df.groupby("ciudad")["importe"].sum().sort_values()
ventas.plot(kind="barh", title="Ventas por ciudad")
plt.xlabel("Importe (€)")
plt.tight_layout()
plt.show()
```

Los gráficos aparecen justo debajo de la celda. Para gráficos interactivos (zoom, desplazamiento) se puede instalar `ipympl` y usar `%matplotlib widget`.

### 11.3 Mostrar varias salidas en una celda

```python
from IPython.display import display, Markdown

display(df.head(3))
display(Markdown(f"**Total de filas:** {len(df)}"))
```

### 11.4 Abrir archivos de datos

Haciendo doble clic en el explorador:

- Los **CSV** se abren en un visor de tablas.
- Los **JSON** se abren en un visor en árbol.
- Las **imágenes** y los **PDF** se muestran directamente.

> ⚠️ No abras con doble clic CSV de cientos de MB: el navegador puede bloquearse. Explóralos con `!head` o con pandas/Spark.

---

## 12. PySpark en JupyterLab

Con el entorno `bigdata` (que incluye `pyspark` y Java) o con la imagen Docker `pyspark-notebook`:

```python
from pyspark.sql import SparkSession
from pyspark.sql import functions as F

spark = (
    SparkSession.builder
    .appName("NotebookBigData")
    .master("local[*]")                        # Usa todos los núcleos del equipo
    .config("spark.driver.memory", "2g")
    .getOrCreate()
)

spark    # Muestra la versión y un enlace a la interfaz web de Spark
```

Consejos específicos para notebooks con Spark:

- Crea **una sola `SparkSession`** al principio del notebook.
- `df.show()` imprime texto; para ver una tabla bonita de **pocas filas**, usa `df.limit(20).toPandas()`.
- **Nunca** hagas `toPandas()` de un DataFrame grande: trae todos los datos a la memoria del notebook.
- Activa la visualización automática de DataFrames como tabla:

  ```python
  spark.conf.set("spark.sql.repl.eagerEval.enabled", True)
  df   # Se muestra como tabla sin necesidad de .show()
  ```

- Sigue la ejecución de los trabajos en la interfaz web de Spark: <http://localhost:4040>.
- Al terminar: `spark.stop()`, o reinicia el kernel.

---

## 13. Herramientas integradas y extensiones

### 13.1 Incluidas en JupyterLab 4

- **Terminal** integrada (desde el *Launcher*).
- **Consola** vinculada a un notebook: clic derecho en el notebook → **New Console for Notebook**; comparte las variables del kernel, ideal para pruebas rápidas.
- **Depurador:** activa el icono del insecto 🐞 en la barra del notebook, pon *breakpoints* haciendo clic junto a los números de línea y ejecuta la celda.
- **Inspector de variables** (panel del depurador).
- **Índice** automático a partir de los títulos Markdown.
- **Búsqueda y reemplazo** en todo el notebook: `Ctrl + F`.
- **Tema oscuro:** **Settings → Theme → JupyterLab Dark**.
- **Idioma español:** instala el paquete de idioma y selecciónalo en **Settings → Language**:

  ```bash
  pip install jupyterlab-language-pack-es-ES
  ```

### 13.2 Extensiones útiles

Las extensiones de JupyterLab 4 se instalan como paquetes de Python (con `pip` o `conda`) y **después se reinicia JupyterLab**:

| Extensión | Paquete | Para qué |
|---|---|---|
| Git | `jupyterlab-git` | Panel de Git (commits, diferencias, ramas) |
| Autocompletado avanzado | `jupyterlab-lsp` + `python-lsp-server` | Diagnóstico de errores, ir a la definición… |
| Formateo de código | `jupyterlab-code-formatter` + `black` | Formatea el código de las celdas |
| Colaboración en tiempo real | `jupyter-collaboration` | Varias personas editando el mismo notebook |
| Gráficos interactivos | `ipympl` | `%matplotlib widget` |

```bash
conda activate bigdata
pip install jupyterlab-git
jupyter labextension list     # Comprueba las extensiones instaladas
```

---

## 14. Exportar y compartir notebooks

### 14.1 Desde la interfaz

**File → Save and Export Notebook As…** → HTML, PDF, Markdown, script de Python (`.py`), LaTeX…

### 14.2 Desde la terminal con `nbconvert`

```bash
jupyter nbconvert --to html analisis.ipynb
jupyter nbconvert --to script analisis.ipynb               # Genera analisis.py
jupyter nbconvert --to html --no-input analisis.ipynb      # Solo resultados, sin código (informes)
jupyter nbconvert --to notebook --execute --inplace analisis.ipynb   # Ejecuta el notebook completo
```

> Para exportar a **PDF** hace falta LaTeX (`sudo apt install texlive-xetex texlive-fonts-recommended pandoc`). Una alternativa más sencilla es exportar a **HTML** e imprimirlo a PDF desde el navegador, o usar `--to webpdf` (requiere `pip install "nbconvert[webpdf]"` y `playwright install chromium`).

### 14.3 Entregar prácticas

Antes de entregar un notebook:

1. **Kernel → Restart Kernel and Run All Cells**.
2. Comprueba que **no hay errores** y que los contadores de ejecución van en orden (1, 2, 3…).
3. Guarda (`Ctrl + S`).
4. Entrega el `.ipynb` (y, si se pide, también el HTML exportado).

---

## 15. Notebooks y Git

Un `.ipynb` es un JSON que guarda también las **salidas** (tablas, imágenes en base64, contadores). Esto provoca diferencias enormes y conflictos en Git.

**Opción 1 – Limpiar las salidas antes de hacer *commit*:** **Edit → Clear Outputs of All Cells**, o desde la terminal:

```bash
jupyter nbconvert --clear-output --inplace analisis.ipynb
```

**Opción 2 – `nbstripout`: limpia las salidas automáticamente en cada *commit*:**

```bash
pip install nbstripout
cd mi-repositorio
nbstripout --install
```

**Opción 3 – Ver diferencias legibles con `nbdime`:**

```bash
pip install nbdime
nbdime config-git --enable
nbdiff-web analisis.ipynb     # Comparación visual en el navegador
```

Añade al `.gitignore`:

```
.ipynb_checkpoints/
```

(Esa carpeta contiene las copias de seguridad automáticas de JupyterLab: **File → Revert Notebook to Checkpoint**.)

---

## 16. Configuración y seguridad

### 16.1 Archivo de configuración

```bash
jupyter lab --generate-config
# Crea ~/.jupyter/jupyter_lab_config.py
```

Ejemplos de opciones (quita el `#` de la línea correspondiente):

```python
c.ServerApp.open_browser = False
c.ServerApp.port = 8888
c.ServerApp.root_dir = "/home/alumno/bigdata"
```

Las preferencias de la interfaz (tema, atajos, tamaño de fuente…) se cambian en **Settings → Settings Editor**.

### 16.2 Contraseña en lugar de *token*

```bash
jupyter server password
```

A partir de entonces, JupyterLab pide esa contraseña en lugar de usar el *token* de la URL.

### 16.3 Seguridad

- Un notebook **ejecuta código con tus permisos**: no ejecutes notebooks descargados sin revisarlos.
- **No expongas JupyterLab a la red** (`--ip 0.0.0.0`) sin contraseña: cualquiera que acceda podría ejecutar comandos en tu equipo.
- **No escribas contraseñas ni claves en las celdas.** Léelas de variables de entorno o pídelas al ejecutar:

  ```python
  import os
  from getpass import getpass

  usuario = os.environ.get("DB_USER")
  clave = getpass("Contraseña de la base de datos: ")
  ```

---

## 17. Buenas prácticas

1. **Estructura clara:** título, descripción del objetivo, importaciones y configuración al principio, y conclusiones al final.
2. **Una idea por celda**, con celdas cortas.
3. **Explica** con celdas Markdown qué haces y por qué.
4. **Ejecuta de arriba abajo:** el notebook debe funcionar con *Restart and Run All*.
5. **Rutas relativas** (`datos/ventas.csv`), no absolutas (`/home/ana/...`), para que funcione en otros equipos.
6. **No guardes datasets grandes** dentro del notebook ni en el repositorio; guárdalos aparte en `datos/`.
7. Cuando un código se repite, **llévalo a un `.py`** e impórtalo.
8. **Cierra los kernels** que no uses (panel ⏹): cada uno ocupa memoria, y Spark mucha.
9. Pon **nombres descriptivos** a los notebooks: `01_carga_datos.ipynb`, `02_limpieza.ipynb`, `03_analisis.ipynb`.

Estructura recomendada de un proyecto:

```
proyecto/
├── datos/
│   ├── crudos/
│   └── procesados/
├── notebooks/
│   ├── 01_carga_datos.ipynb
│   ├── 02_limpieza.ipynb
│   └── 03_analisis.ipynb
├── src/
│   └── utilidades.py
├── environment.yml
└── README.md
```

---

## 18. Solución de problemas frecuentes

| Problema | Causa probable | Solución |
|---|---|---|
| `jupyter: command not found` | Entorno no activado o JupyterLab no instalado | `conda activate bigdata` o instalar `jupyterlab` |
| El navegador pide un *token* | Se abrió la URL sin el *token* | Copia la URL completa de la terminal, o `jupyter server list` |
| `ModuleNotFoundError` en el notebook, aunque el paquete está instalado | El kernel usa otro entorno | `sys.executable` para comprobarlo; cambia de kernel o instala con `%pip install` |
| El entorno conda no aparece como kernel | Kernel no registrado | `python -m ipykernel install --user --name <entorno>` |
| La celda se queda en `[*]` para siempre | Operación larga, bucle infinito o kernel bloqueado | `I`, `I` (interrumpir); si no responde, reiniciar el kernel |
| `Kernel died` / el kernel se reinicia solo | Falta de memoria (datos demasiado grandes, `toPandas()`…) | Trabajar con muestras, usar Spark, aumentar la memoria de WSL en `.wslconfig` |
| `Port 8888 is already in use` | Hay otro JupyterLab (o contenedor) abierto | `jupyter server list`, cerrarlo, o `--port 8890` |
| Variable "no definida" que sí está en el notebook | Las celdas se ejecutaron en otro orden, o se reinició el kernel | **Run → Run All Cells** |
| Los cambios en un `.py` propio no se aplican | Python ya importó el módulo | `%load_ext autoreload` y `%autoreload 2`, o reiniciar el kernel |
| La extensión instalada no aparece | JupyterLab no se ha reiniciado | Detener y volver a arrancar `jupyter lab` |
| Todo va lento al abrir archivos | Proyecto en `/mnt/c` | Mover el proyecto a `~/` dentro de Linux |

Diagnóstico general:

```bash
jupyter --version
jupyter kernelspec list
jupyter labextension list
jupyter server list
```

---

## 19. Chuleta rápida

```bash
# ---------- Terminal ----------
conda activate bigdata                    # Activar el entorno
jupyter lab --no-browser                  # Arrancar JupyterLab
jupyter server list                       # Ver servidores y tokens
python -m ipykernel install --user --name <env>   # Registrar kernel
jupyter kernelspec list                   # Ver kernels
jupyter nbconvert --to html <nb>.ipynb    # Exportar a HTML
```

```text
---------- Notebook ----------
Shift+Enter   Ejecutar y avanzar       Esc      Modo comando
Ctrl+Enter    Ejecutar                 Enter    Modo edición
A / B         Celda arriba / abajo     D, D     Borrar celda
M / Y         Markdown / Código        Z        Deshacer (celdas)
I, I          Interrumpir kernel       0, 0     Reiniciar kernel
Tab           Autocompletar            Shift+Tab  Ayuda de la función
Ctrl+S        Guardar                  Ctrl+Shift+C  Paleta de comandos
```

```python
# ---------- Comandos mágicos ----------
%%time              # Tiempo de la celda
%timeit expr        # Tiempo medio de una expresión
%pip install pkg    # Instalar en el kernel actual
!ls datos/          # Comando del sistema
%whos               # Variables definidas
```

---

## 20. Referencias

- Documentación oficial de JupyterLab: <https://jupyterlab.readthedocs.io/>
- Proyecto Jupyter: <https://jupyter.org/>
- Atajos y comandos (en la propia interfaz): **Help → Show Keyboard Shortcuts**
- IPython – comandos mágicos: <https://ipython.readthedocs.io/en/stable/interactive/magics.html>
- nbconvert: <https://nbconvert.readthedocs.io/>
- Imágenes Docker de Jupyter (Docker Stacks): <https://jupyter-docker-stacks.readthedocs.io/>
- nbstripout: <https://github.com/kynan/nbstripout>
- nbdime: <https://nbdime.readthedocs.io/>
- JupyterLab Desktop: <https://github.com/jupyterlab/jupyterlab-desktop>
