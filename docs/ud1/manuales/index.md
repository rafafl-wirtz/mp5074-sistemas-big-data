# <span class="ud ud1">UD1</span> Manuales del entorno de trabajo

Los manuales están agrupados **por para qué sirven**, en el orden en que se usan a lo largo del curso.

## 1. Preparar el equipo

Tener un entorno que funcione igual en todos los ordenadores.

| Manual | Para qué sirve |
|---|---|
| [Conocimientos previos](../conocimientos_previos/manual_conocimientos_previos.md) | Sistema de ficheros, rutas y variables de entorno: lo que hay que saber antes de instalar nada |
| [WSL](wsl/manual_wsl.md) | Ejecutar Ubuntu dentro de Windows: la base de todo el entorno |
| [Miniconda](miniconda/manual_miniconda.md) | Crear entornos de Python aislados y reproducibles (`bigdata`) |
| [Docker](docker/manual_docker.md) | Levantar bases de datos, Spark o Jupyter en contenedores sin instalarlos en el equipo |

## 2. Trabajar y entregar

Guardar versiones, subir el trabajo a GitHub y documentarlo.

| Manual | Para qué sirve |
|---|---|
| [Git](git/manual_git.md) | Control de versiones y entrega de prácticas con GitHub |
| [SSH](ssh/manual_ssh.md) | Claves SSH en Windows y Linux para conectarse a GitHub sin contraseña |
| [Markdown](markdown/manual_markdown.md) | Dar formato a las celdas de los notebooks, los README de GitHub y esta web |

## 3. Programar con datos

Escribir y ejecutar código y leer formatos de datos.

| Manual | Para qué sirve |
|---|---|
| [JupyterLab](jupyterlab/manual_jupyterlab.md) | Notebooks para explorar y analizar datos |
| [Python](python/manual_python.md) | El Python necesario para los notebooks: estructuras de datos, funciones y lectura de ficheros |
| [JSON y YAML](json_yaml/manual_json_yaml.md) | Leer y escribir datos de APIs (JSON, JSON Lines) y ficheros de configuración (conda, Docker Compose, GitHub Actions) |

## 4. Analizar datos

Resumir datos e interpretarlos.

| Manual | Para qué sirve |
|---|---|
| [pandas para estadística](pandas_estadistica/manual_pandas_estadistica.md) | El pandas mínimo para la estadística: `Series`, funciones de resumen, filtros, ausentes y `groupby` |
| [Estadística y agregados](estadistica/manual_estadistica.md) | Medidas de tendencia central y dispersión, funciones de agregación (SQL y pandas) y agregados en sistemas distribuidos |

## Documentación oficial

- WSL: <https://learn.microsoft.com/es-es/windows/wsl/>
- Configurar el entorno de desarrollo en WSL: <https://learn.microsoft.com/es-es/windows/wsl/setup/environment>
