# Manual de Miniconda

**Módulo:** MP5074 – Sistemas de Big Data · Curso 2026-27

> Miniconda es un instalador mínimo de **conda**, un gestor de paquetes y de entornos. Permite tener varias versiones de Python y de librerías (pandas, PySpark, Jupyter…) aisladas por proyecto, sin que choquen entre sí ni con el Python del sistema. En este módulo lo instalaremos **dentro de WSL (Ubuntu)**, siguiendo el [manual de WSL](../wsl/manual_wsl.md).

---

## Índice

1. [¿Qué es Miniconda?](#1-qué-es-miniconda)
2. [Conceptos clave](#2-conceptos-clave)
3. [Instalación en WSL / Linux](#3-instalación-en-wsl--linux)
4. [Instalación en Windows (alternativa)](#4-instalación-en-windows-alternativa)
5. [Configuración inicial recomendada](#5-configuración-inicial-recomendada)
6. [Gestión de entornos](#6-gestión-de-entornos)
7. [Gestión de paquetes](#7-gestión-de-paquetes)
8. [Canales: `defaults` y `conda-forge`](#8-canales-defaults-y-conda-forge)
9. [Archivos `environment.yml`: entornos reproducibles](#9-archivos-environmentyml-entornos-reproducibles)
10. [conda y pip juntos](#10-conda-y-pip-juntos)
11. [Jupyter y VS Code con conda](#11-jupyter-y-vs-code-con-conda)
12. [Entorno de Big Data para el módulo](#12-entorno-de-big-data-para-el-módulo)
13. [Mantenimiento: actualizar, limpiar y desinstalar](#13-mantenimiento-actualizar-limpiar-y-desinstalar)
14. [Solución de problemas frecuentes](#14-solución-de-problemas-frecuentes)
15. [Chuleta rápida](#15-chuleta-rápida)
16. [Referencias](#16-referencias)

---

## 1. ¿Qué es Miniconda?

**conda** es un gestor de paquetes y de entornos multiplataforma (Linux, Windows, macOS). A diferencia de `pip`, no se limita a Python: también instala dependencias del sistema, como bibliotecas en C/C++, compiladores o incluso **Java**.

Hay varias distribuciones que incluyen conda:

| Distribución | Qué incluye | Tamaño aprox. | Cuándo usarla |
|---|---|---|---|
| **Anaconda** | conda + Python + más de 300 paquetes científicos + Anaconda Navigator (interfaz gráfica) | Varios GB | Quien quiere todo instalado de golpe |
| **Miniconda** (la que usaremos) | conda + Python + lo mínimo | ~100 MB (instalador) | Instalar solo lo necesario, entornos ligeros |
| **Miniforge** | Como Miniconda, pero usando por defecto el canal comunitario `conda-forge` | ~100 MB | Alternativa 100 % comunitaria (ver [sección 8](#8-canales-defaults-y-conda-forge)) |

**¿Por qué Miniconda y no solo `pip` + `venv`?**

- Cada entorno puede tener **su propia versión de Python** (3.10, 3.11, 3.12…).
- Instala paquetes **precompilados** con sus dependencias nativas: menos errores al compilar.
- Los entornos se pueden **exportar y reproducir** en otro equipo con un solo archivo.
- Es el estándar de facto en ciencia de datos.

---

## 2. Conceptos clave

- **Paquete:** software que se instala (p. ej., `pandas`, `pyspark`, `openjdk`).
- **Entorno (*environment*):** carpeta aislada con su propio Python y sus paquetes. Lo que se instala en un entorno no afecta a los demás.
- **Entorno `base`:** el entorno que crea el instalador. **No se debe usar para trabajar**; es solo para conda.
- **Canal (*channel*):** repositorio desde el que se descargan los paquetes (`defaults`, `conda-forge`…).
- **Solver:** el componente que calcula qué versiones de cada paquete son compatibles. Desde conda 23.10 se usa por defecto `libmamba`, que es mucho más rápido que el antiguo.
- **`.condarc`:** archivo de configuración de conda del usuario (`~/.condarc`).

```
~/miniconda3/
├── bin/conda           ← ejecutable de conda
├── pkgs/               ← caché de paquetes descargados
└── envs/
    ├── bigdata/        ← un entorno (su propio Python y librerías)
    └── pruebas/        ← otro entorno, totalmente independiente
```

---

## 3. Instalación en WSL / Linux

> Todos los comandos de esta sección se ejecutan **en la terminal de Ubuntu (WSL)**, no en PowerShell.

### 3.1 Descargar el instalador

```bash
cd ~
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
```

(También vale `curl -O <misma URL>`.)

### 3.2 Ejecutar el instalador

**Instalación interactiva:**

```bash
bash ~/Miniconda3-latest-Linux-x86_64.sh
```

Durante la instalación:

1. Pulsa `Intro` y lee la licencia (`q` para salir del texto).
2. Escribe `yes` para aceptar.
3. Acepta la ruta propuesta: `~/miniconda3` (pulsa `Intro`).
4. Cuando pregunte si quieres inicializar conda (*"initialize Miniconda3 by running conda init?"*), responde **`yes`**.

**Instalación desatendida** (útil para scripts o para preparar muchos equipos):

```bash
bash ~/Miniconda3-latest-Linux-x86_64.sh -b -p ~/miniconda3
~/miniconda3/bin/conda init bash
```

- `-b` → modo *batch*: sin preguntas, acepta la licencia.
- `-p` → ruta de instalación.

### 3.3 Activar conda en la terminal

```bash
source ~/.bashrc
```

(O cierra y vuelve a abrir la terminal.) El *prompt* debe mostrar `(base)` al principio:

```
(base) alumno@PC:~$
```

### 3.4 Verificar la instalación

```bash
conda --version
conda info
which python      # Debe apuntar a ~/miniconda3/bin/python
```

Puedes borrar el instalador:

```bash
rm ~/Miniconda3-latest-Linux-x86_64.sh
```

---

## 4. Instalación en Windows (alternativa)

Solo si no se va a usar WSL:

1. Descarga `Miniconda3-latest-Windows-x86_64.exe` desde <https://www.anaconda.com/download> (sección *Miniconda Installers*).
2. Instala con la opción **"Just Me"** (recomendada).
3. **No** marques *"Add Miniconda3 to my PATH environment variable"*; puede dar conflictos con otros Python.
4. Abre **Anaconda Prompt (miniconda3)** desde el menú Inicio para usar conda.
5. Para usarlo también desde PowerShell, ejecuta una vez en Anaconda Prompt `conda init powershell` y vuelve a abrir PowerShell.

Instalación desatendida en Windows (desde CMD):

```bat
Miniconda3-latest-Windows-x86_64.exe /InstallationType=JustMe /RegisterPython=0 /S /D=%UserProfile%\Miniconda3
```

> ⚠️ Los comandos de conda son los mismos en Windows y Linux, pero **las herramientas de Big Data (Hadoop, Spark…) funcionan mejor en Linux**. En el módulo usaremos la instalación de WSL.

---

## 5. Configuración inicial recomendada

### 5.1 Que no se active `base` automáticamente

Así cada terminal arranca "limpia" y activamos solo el entorno que necesitemos:

```bash
conda config --set auto_activate false
```

> En versiones antiguas de conda la opción se llamaba `auto_activate_base`. Si la anterior da error, usa `conda config --set auto_activate_base false`.

### 5.2 Añadir `conda-forge` como canal prioritario

```bash
conda config --add channels conda-forge
conda config --set channel_priority strict
```

### 5.3 Consultar la configuración

```bash
conda config --show channels
cat ~/.condarc
```

Contenido resultante de `~/.condarc`:

```yaml
auto_activate: false
channels:
  - conda-forge
  - defaults
channel_priority: strict
```

### 5.4 Aceptar las condiciones de uso de los canales de Anaconda

Las versiones recientes de conda piden aceptar las condiciones de servicio (*Terms of Service*) antes de usar los canales de Anaconda (`defaults`). Si al crear un entorno aparece un error `CondaToSNonInteractiveError` o un mensaje sobre *Terms of Service*, acéptalas con:

```bash
conda tos accept --override-channels --channel https://repo.anaconda.com/pkgs/main
conda tos accept --override-channels --channel https://repo.anaconda.com/pkgs/r
```

Otra opción es trabajar **solo con `conda-forge`** (ver [sección 8](#8-canales-defaults-y-conda-forge)).

---

## 6. Gestión de entornos

### 6.1 Crear un entorno

```bash
conda create -n bigdata python=3.12
```

Crear el entorno e instalar paquetes a la vez (recomendado, así el *solver* resuelve todo junto):

```bash
conda create -n analisis python=3.12 pandas numpy matplotlib
```

### 6.2 Activar y desactivar

```bash
conda activate bigdata      # El prompt pasa a mostrar (bigdata)
python --version
conda deactivate            # Vuelve al entorno anterior
```

### 6.3 Listar, clonar, renombrar y eliminar

| Comando | Descripción |
|---|---|
| `conda env list` (o `conda info --envs`) | Lista los entornos; el `*` marca el activo |
| `conda create -n copia --clone bigdata` | Clona un entorno |
| `conda rename -n viejo nuevo` | Renombra un entorno |
| `conda remove -n pruebas --all` | ⚠️ Elimina un entorno por completo |
| `conda env remove -n pruebas` | Igual que el anterior |

### 6.4 Entornos en una carpeta del proyecto

En lugar de por nombre, un entorno puede vivir dentro de la carpeta del proyecto:

```bash
conda create -p ./env python=3.12
conda activate ./env
```

### 6.5 Buenas prácticas

- **Un entorno por proyecto** o por tipo de práctica.
- **No instales nada en `base`**; solo se actualiza conda.
- Fija la versión de Python al crear el entorno (`python=3.12`).
- Guarda la definición del entorno en un `environment.yml` (ver [sección 9](#9-archivos-environmentyml-entornos-reproducibles)).

---

## 7. Gestión de paquetes

> Los comandos actúan sobre el **entorno activo**. Para actuar sobre otro sin activarlo, añade `-n <entorno>`.

| Comando | Descripción |
|---|---|
| `conda install pandas` | Instala un paquete |
| `conda install pandas=2.2 numpy` | Instala una versión concreta y varios paquetes |
| `conda install -c conda-forge pyspark` | Instala desde un canal concreto |
| `conda install -n bigdata pandas` | Instala en otro entorno |
| `conda list` | Paquetes instalados en el entorno activo |
| `conda list pandas` | Busca un paquete concreto entre los instalados |
| `conda search pyspark` | Versiones disponibles en los canales |
| `conda update pandas` | Actualiza un paquete |
| `conda update --all` | Actualiza todos los paquetes del entorno |
| `conda remove pandas` | Desinstala un paquete |

Simular antes de instalar (muestra qué cambiaría sin hacer nada):

```bash
conda install pyspark --dry-run
```

---

## 8. Canales: `defaults` y `conda-forge`

| Canal | Quién lo mantiene | Características |
|---|---|---|
| `defaults` (`repo.anaconda.com`) | Anaconda, Inc. | Paquetes revisados por la empresa. Sujeto a las [condiciones de servicio de Anaconda](https://www.anaconda.com/legal) |
| `conda-forge` | Comunidad (código abierto) | El catálogo más grande y actualizado; gratuito para cualquier uso |

> **Licencias:** el uso de los canales de Anaconda (`defaults`) es gratuito para particulares y en determinados supuestos (organizaciones pequeñas, docencia), pero las organizaciones grandes pueden necesitar licencia comercial. Las condiciones cambian con el tiempo: consúltalas en <https://www.anaconda.com/legal>. `conda-forge` **no tiene esa restricción**.

**Trabajar solo con `conda-forge`:**

```bash
conda config --remove channels defaults
conda config --add channels conda-forge
conda config --set channel_priority strict
```

(También se puede instalar directamente **Miniforge**, que ya viene configurado así: <https://github.com/conda-forge/miniforge>. Todos los comandos de este manual funcionan igual.)

**Regla importante:** no mezcles canales dentro de un entorno sin `channel_priority: strict`; es la causa más habitual de conflictos y errores raros.

---

## 9. Archivos `environment.yml`: entornos reproducibles

Un `environment.yml` describe un entorno completo. Sirve para que **todo el alumnado tenga exactamente el mismo entorno**.

### 9.1 Ejemplo

```yaml
name: bigdata
channels:
  - conda-forge
dependencies:
  - python=3.12
  - pandas
  - numpy
  - jupyterlab
  - pip
  - pip:
      - un-paquete-solo-en-pypi
```

### 9.2 Crear, actualizar y exportar

| Comando | Descripción |
|---|---|
| `conda env create -f environment.yml` | Crea el entorno a partir del archivo |
| `conda env update -f environment.yml --prune` | Actualiza el entorno si se cambia el archivo (`--prune` quita lo que sobra) |
| `conda env export --from-history > environment.yml` | Exporta **solo lo que instalaste explícitamente** (portable entre sistemas) ✅ |
| `conda env export > environment.lock.yml` | Exporta **todas** las versiones exactas (reproducible, pero solo en el mismo SO) |

> Recomendación: usa `--from-history` para compartir entornos entre Windows y Linux.

---

## 10. conda y pip juntos

A veces un paquete solo está en PyPI. Se puede usar `pip` **dentro de un entorno conda**, siguiendo estas normas:

1. Instala **primero todo lo posible con conda** y después lo que falte con pip.
2. Instala `pip` en el propio entorno (`conda install pip`) y **usa siempre el pip del entorno activo**.
3. **Nunca** uses `pip install` en el entorno `base`.
4. Si después necesitas más paquetes de conda, es más seguro **recrear el entorno** desde el `environment.yml`.

```bash
conda activate bigdata
conda install pip
python -m pip install paquete-de-pypi
which pip     # Debe apuntar a ~/miniconda3/envs/bigdata/bin/pip
```

---

## 11. Jupyter y VS Code con conda

### 11.1 JupyterLab

```bash
conda activate bigdata
conda install jupyterlab
jupyter lab --no-browser
```

Copia la URL `http://localhost:8888/lab?token=...` en el navegador de Windows.

### 11.2 Usar varios entornos como *kernels* de Jupyter

Para elegir en Jupyter el entorno con el que ejecutar cada *notebook*:

```bash
conda activate bigdata
conda install ipykernel
python -m ipykernel install --user --name bigdata --display-name "Python (bigdata)"
```

Listar o borrar kernels:

```bash
jupyter kernelspec list
jupyter kernelspec uninstall bigdata
```

### 11.3 VS Code

1. Abre la carpeta del proyecto desde WSL: `code .`
2. Instala en WSL las extensiones **Python** y **Jupyter**.
3. `Ctrl + Shift + P` → **Python: Select Interpreter** → elige `~/miniconda3/envs/bigdata/bin/python`.
4. En los *notebooks*, arriba a la derecha: **Select Kernel** → el entorno conda.

---

## 12. Entorno de Big Data para el módulo

conda permite instalar **Java y PySpark en el mismo entorno**, sin tocar el sistema.

### 12.1 Archivo `environment.yml` del módulo

Guarda este archivo como `~/bigdata/environment.yml`:

```yaml
name: bigdata
channels:
  - conda-forge
dependencies:
  - python=3.12
  - openjdk=17          # Java necesario para Spark
  - pyspark
  - pandas
  - numpy
  - pyarrow             # Formato Parquet y transferencia Spark <-> pandas
  - matplotlib
  - seaborn
  - jupyterlab
  - ipykernel
  - findspark
  - pip
```

### 12.2 Crear y registrar el entorno

```bash
cd ~/bigdata
conda env create -f environment.yml
conda activate bigdata
python -m ipykernel install --user --name bigdata --display-name "Python (bigdata)"
```

### 12.3 Comprobar que todo funciona

```bash
java -version
python -c "import pyspark; print(pyspark.__version__)"
```

Prueba rápida de Spark (`prueba_spark.py`):

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("PruebaConda").master("local[*]").getOrCreate()

df = spark.createDataFrame(
    [("Ana", 34), ("Luis", 28), ("Marta", 41)],
    ["nombre", "edad"],
)
df.filter(df.edad > 30).show()

spark.stop()
```

```bash
python prueba_spark.py
```

Salida esperada:

```
+------+----+
|nombre|edad|
+------+----+
|   Ana|  34|
| Marta|  41|
+------+----+
```

Mientras se ejecuta, la interfaz web de Spark está en <http://localhost:4040>.

> Antes de fijar versiones, comprueba en la documentación de Spark qué versiones de Java y Python admite la que vayáis a usar.

---

## 13. Mantenimiento: actualizar, limpiar y desinstalar

### 13.1 Actualizar conda

```bash
conda update -n base conda
```

### 13.2 Liberar espacio

La caché de paquetes crece rápido:

```bash
conda clean --all        # Borra la caché de paquetes y tarballs no usados
du -sh ~/miniconda3      # Espacio ocupado
```

### 13.3 Desinstalar Miniconda (Linux / WSL)

```bash
conda init --reverse --all        # Quita las líneas de conda de ~/.bashrc
rm -rf ~/miniconda3               # Borra la instalación y todos los entornos
rm -rf ~/.condarc ~/.conda ~/.continuum
```

Cierra y vuelve a abrir la terminal.

**En Windows:** *Configuración → Aplicaciones → Miniconda3 → Desinstalar*.

---

## 14. Solución de problemas frecuentes

| Problema | Causa probable | Solución |
|---|---|---|
| `conda: command not found` | La terminal no ha cargado la configuración | `source ~/.bashrc` o `~/miniconda3/bin/conda init bash` y reabrir la terminal |
| `CondaError: Run 'conda init' before 'conda activate'` | Shell no inicializada | `conda init bash` y reabrir la terminal |
| Error sobre *Terms of Service* (`CondaToSNonInteractiveError`) | No se han aceptado las condiciones del canal `defaults` | `conda tos accept ...` ([5.4](#54-aceptar-las-condiciones-de-uso-de-los-canales-de-anaconda)) o usar solo `conda-forge` |
| `(base)` aparece siempre | Autoactivación de `base` | `conda config --set auto_activate false` |
| La instalación de paquetes tarda muchísimo o da conflictos | Canales mezclados o entorno muy cargado | `channel_priority strict`, crear un entorno nuevo con todo en un solo `conda create` |
| `ModuleNotFoundError` a pesar de haber instalado el paquete | Estás en otro entorno o usas otro Python | `conda env list`, `which python`, activar el entorno correcto |
| Jupyter no ve el entorno | Kernel no registrado | `python -m ipykernel install --user --name <entorno>` |
| Spark: `JAVA_HOME is not set` | Java no está en el entorno | `conda install -c conda-forge openjdk=17` y reactivar el entorno |
| El disco se llena | Caché de paquetes | `conda clean --all` |

Diagnóstico general:

```bash
conda info
conda config --show
conda env list
which python
```

---

## 15. Chuleta rápida

```bash
# ---------- Entornos ----------
conda create -n <env> python=3.12       # Crear
conda activate <env>                    # Activar
conda deactivate                        # Desactivar
conda env list                          # Listar
conda remove -n <env> --all             # Eliminar

# ---------- Paquetes ----------
conda install <paquete>                 # Instalar
conda install -c conda-forge <paquete>  # Instalar desde conda-forge
conda list                              # Ver instalados
conda update --all                      # Actualizar todo
conda remove <paquete>                  # Desinstalar

# ---------- Reproducibilidad ----------
conda env create -f environment.yml             # Crear desde archivo
conda env export --from-history > environment.yml  # Exportar

# ---------- Mantenimiento ----------
conda update -n base conda              # Actualizar conda
conda clean --all                       # Liberar espacio
conda config --set auto_activate false  # No activar base al abrir la terminal
```

---

## 16. Referencias

- Instalación de Miniconda (Anaconda Docs): <https://www.anaconda.com/docs/getting-started/miniconda/install>
- Instalación en Linux: <https://www.anaconda.com/docs/getting-started/miniconda/install/linux-install>
- Documentación de conda – Gestión de entornos: <https://docs.conda.io/projects/conda/en/stable/user-guide/tasks/manage-environments.html>
- Documentación de conda – Notas de versión: <https://docs.conda.io/projects/conda/en/stable/release-notes.html>
- conda-forge: <https://conda-forge.org/>
- Miniforge: <https://github.com/conda-forge/miniforge>
- Condiciones de servicio de Anaconda: <https://www.anaconda.com/legal>
