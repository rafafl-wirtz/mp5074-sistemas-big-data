# Manual de pandas para el tratamiento de datos

**Módulo:** MP5074 – Sistemas de Big Data · Curso 2026-27

> **pandas** es la librería de Python para trabajar con **datos tabulares**: leerlos de casi cualquier origen, explorarlos, limpiarlos, transformarlos, agregarlos, combinarlos y guardarlos. Es la herramienta con la que se prepara y se prototipa la mayor parte del trabajo con datos, y su forma de pensar (DataFrames, columnas, agrupaciones) es la misma que usaremos después con **PySpark**. Este manual recoge todo lo necesario para el módulo y dedica una sección a **cómo trabajar con volúmenes grandes** y cuándo dar el salto a herramientas distribuidas. Requiere el entorno `bigdata` del [manual de Miniconda](../miniconda/manual_miniconda.md) y conviene haber repasado el [manual de Python](../python/manual_python.md).

---

## Índice

1. [¿Qué es pandas y qué papel tiene en Big Data?](#1-qué-es-pandas-y-qué-papel-tiene-en-big-data)
2. [Instalación e importación](#2-instalación-e-importación)
3. [Estructuras de datos: `Series` y `DataFrame`](#3-estructuras-de-datos-series-y-dataframe)
4. [Leer datos](#4-leer-datos)
5. [Guardar datos y elegir formato](#5-guardar-datos-y-elegir-formato)
6. [Primera exploración](#6-primera-exploración)
7. [Seleccionar y filtrar](#7-seleccionar-y-filtrar)
8. [Crear y modificar columnas](#8-crear-y-modificar-columnas)
9. [Tipos de datos](#9-tipos-de-datos)
10. [Limpieza de datos](#10-limpieza-de-datos)
11. [Ordenar y clasificar](#11-ordenar-y-clasificar)
12. [Agrupar y agregar](#12-agrupar-y-agregar)
13. [Reestructurar: tablas dinámicas, `melt` y `pivot`](#13-reestructurar-tablas-dinámicas-melt-y-pivot)
14. [Combinar tablas: `concat`, `merge` y `join`](#14-combinar-tablas-concat-merge-y-join)
15. [Fechas y series temporales](#15-fechas-y-series-temporales)
16. [Gráficos rápidos](#16-gráficos-rápidos)
17. [Encadenar operaciones y *Copy-on-Write*](#17-encadenar-operaciones-y-copy-on-write)
18. [Rendimiento y grandes volúmenes](#18-rendimiento-y-grandes-volúmenes)
19. [De pandas a Spark y otras herramientas](#19-de-pandas-a-spark-y-otras-herramientas)
20. [Buenas prácticas](#20-buenas-prácticas)
21. [Solución de problemas frecuentes](#21-solución-de-problemas-frecuentes)
22. [Chuleta rápida](#22-chuleta-rápida)
23. [Referencias](#23-referencias)
24. [Ejemplo resuelto: de CSV en bruto a Parquet analizable](#24-ejemplo-resuelto-de-csv-en-bruto-a-parquet-analizable)

---

## 1. ¿Qué es pandas y qué papel tiene en Big Data?

pandas (de *panel data*) ofrece dos estructuras, `Series` y `DataFrame`, y cientos de operaciones **vectorizadas** sobre ellas: se aplican a columnas enteras de una vez, en código compilado, sin bucles de Python.

**Qué hace bien:**

- Leer y escribir CSV, Excel, JSON, Parquet, SQL, HTML…
- Limpiar y transformar datos con muy poco código.
- Agrupar, agregar, pivotar y combinar tablas.
- Series temporales (remuestreo, ventanas móviles).
- Integrarse con NumPy, matplotlib, scikit-learn, PySpark…

**Su límite:** pandas trabaja **en memoria y en un solo equipo**. Como regla práctica, necesita entre **5 y 10 veces** el tamaño del fichero en RAM para trabajar con comodidad.

| Volumen de datos | Herramienta adecuada |
|---|---|
| Hasta ~1–2 GB | **pandas** sin problemas |
| De 2 a ~20 GB en un equipo | pandas con técnicas de la [sección 18](#18-rendimiento-y-grandes-volúmenes) (columnas, tipos, *chunks*, Parquet), o Polars / DuckDB |
| Más que la memoria de un equipo, o datos ya en un clúster | **PySpark** (u otro motor distribuido) |

> Incluso en proyectos Big Data, pandas se usa constantemente: para **explorar muestras**, prototipar transformaciones que luego se llevan a Spark, y para trabajar con los **resultados agregados** (que suelen ser pequeños). La API de Spark se parece mucho a la de pandas, y existe `pyspark.pandas`, que imita la API de pandas sobre Spark ([sección 19](#19-de-pandas-a-spark-y-otras-herramientas)).

---

## 2. Instalación e importación

pandas ya está en el entorno `bigdata` del módulo. Para leer Excel y Parquet hacen falta dos librerías adicionales:

```bash
conda activate bigdata
conda install -c conda-forge pandas pyarrow openpyxl
```

En el notebook, siempre con los alias habituales:

```python
import numpy as np
import pandas as pd

pd.__version__
```

Opciones de visualización útiles:

```python
pd.set_option("display.max_columns", 50)      # mostrar hasta 50 columnas
pd.set_option("display.max_rows", 100)
pd.set_option("display.float_format", "{:,.2f}".format)   # 2 decimales y separador de miles
pd.reset_option("display.float_format")       # volver al formato por defecto
```

> **Este manual sigue pandas 3.x** (publicado en enero de 2026). Si usas pandas 2.x casi todo funciona igual; las diferencias principales son que pandas 3 usa por defecto el tipo **`str`** para las columnas de texto (en 2.x aparecían como `object`) y que activa siempre el modo ***Copy-on-Write*** ([sección 17](#17-encadenar-operaciones-y-copy-on-write)).

---

## 3. Estructuras de datos: `Series` y `DataFrame`

```
                 columnas (Index)
              ┌─────────┬─────────┬────────┐
              │ tienda  │ unidades│ precio │
       ┌──────┼─────────┼─────────┼────────┤
       │  0   │ Ferrol  │    2    │  25.0  │
índice │  1   │ Vigo    │    1    │  12.5  │   ← una fila
(Index)│  2   │ Lugo    │    3    │   7.9  │
       └──────┴─────────┴─────────┴────────┘
                            ▲
                 una columna = una Series
```

- **`Series`**: una columna de datos **del mismo tipo** con un índice.
- **`DataFrame`**: una tabla; un conjunto de `Series` que comparten el índice.
- **`Index`**: las etiquetas de filas (por defecto 0, 1, 2…) y de columnas.

### 3.1 Crear una `Series`

```python
s = pd.Series([12.5, 7.0, 30.2], index=["a", "b", "c"], name="precio")
s["b"]              # 7.0
s.mean()            # 16.57
s * 1.21            # operación vectorizada sobre todos los elementos
```

### 3.2 Crear un `DataFrame`

```python
# Desde un diccionario de listas (una lista por columna)
df = pd.DataFrame({
    "tienda": ["Ferrol", "Vigo", "Lugo"],
    "unidades": [2, 1, 3],
    "precio": [25.0, 12.5, 7.9],
})

# Desde una lista de diccionarios (un dict por fila: lo que devuelve una API JSON)
filas = [
    {"tienda": "Ferrol", "unidades": 2, "precio": 25.0},
    {"tienda": "Vigo", "unidades": 1},                    # falta precio → NaN
]
pd.DataFrame(filas)
```

### 3.3 Dataset de ejemplo para el manual

Todos los ejemplos siguientes usan estas dos tablas. Ejecuta esta celda al principio para poder seguirlos:

```python
rng = np.random.default_rng(42)
n = 1_000

ventas = pd.DataFrame({
    "id_venta": np.arange(1, n + 1),
    "fecha": pd.date_range("2026-01-01", periods=n, freq="8h"),
    "tienda": rng.choice(["Ferrol", "Coruña", "Vigo", "Lugo"], n),
    "categoria": rng.choice(["Informática", "Hogar", "Libros"], n, p=[0.5, 0.3, 0.2]),
    "canal": rng.choice(["tienda", "online"], n),
    "unidades": rng.integers(1, 6, n),
    "precio": rng.uniform(5, 300, n).round(2),
    "id_cliente": rng.integers(100, 200, n),
})

# Ensuciamos los datos a propósito, como en la vida real
ventas.loc[rng.choice(n, 30, replace=False), "precio"] = np.nan        # nulos
ventas.loc[rng.choice(n, 10, replace=False), "tienda"] = " vigo "      # texto inconsistente
ventas = pd.concat([ventas, ventas.sample(5, random_state=1)], ignore_index=True)   # duplicados

clientes = pd.DataFrame({
    "id_cliente": range(100, 200),
    "segmento": rng.choice(["particular", "empresa"], 100),
    "provincia": rng.choice(["A Coruña", "Lugo", "Ourense", "Pontevedra"], 100),
})

ventas.head()
```

---

## 4. Leer datos

Todas las funciones de lectura empiezan por `pd.read_` y devuelven un `DataFrame`. Aceptan una **ruta local**, una **URL** o un fichero **comprimido** (`.gz`, `.zip`, `.bz2`, `.zst`), que se descomprime automáticamente.

| Formato | Función | Necesita |
|---|---|---|
| CSV / TXT | `pd.read_csv` | — |
| Excel | `pd.read_excel` | `openpyxl` |
| JSON / JSON Lines | `pd.read_json`, `pd.json_normalize` | — |
| Parquet | `pd.read_parquet` | `pyarrow` |
| SQL | `pd.read_sql` | `sqlite3` (incluido) o SQLAlchemy |
| Tablas HTML | `pd.read_html` | `lxml` |
| Portapapeles | `pd.read_clipboard` | — |

Para seguir los ejemplos, guarda primero el dataset en varios formatos:

```python
from pathlib import Path
DATOS = Path("datos")
DATOS.mkdir(exist_ok=True)

ventas.to_csv(DATOS / "ventas.csv", index=False)
ventas.to_csv(DATOS / "ventas_es.csv", index=False, sep=";", decimal=",", encoding="latin-1")
ventas.to_excel(DATOS / "ventas.xlsx", index=False, sheet_name="2026")
ventas.to_parquet(DATOS / "ventas.parquet", index=False)
```

### 4.1 CSV

```python
df = pd.read_csv(DATOS / "ventas.csv")
```

Parámetros que más se usan:

| Parámetro | Para qué | Ejemplo |
|---|---|---|
| `sep` | Separador de columnas | `sep=";"`, `sep="\t"` |
| `decimal` | Separador decimal | `decimal=","` (datos españoles) |
| `thousands` | Separador de miles | `thousands="."` |
| `encoding` | Codificación | `"utf-8"`, `"utf-8-sig"`, `"latin-1"` |
| `header` | Fila de cabecera (o `None` si no hay) | `header=None` |
| `names` | Nombres de columna | `names=["id", "fecha", "importe"]` |
| `usecols` | Leer **solo** algunas columnas | `usecols=["fecha", "precio"]` |
| `dtype` | Forzar tipos | `dtype={"cp": "str", "tienda": "category"}` |
| `parse_dates` | Columnas a convertir en fecha | `parse_dates=["fecha"]` |
| `na_values` | Textos que significan "nulo" | `na_values=["-", "n/d", "NULL"]` |
| `nrows` | Leer solo las primeras N filas | `nrows=1000` |
| `skiprows` | Saltar filas iniciales | `skiprows=3` |
| `index_col` | Columna que será el índice | `index_col="id_venta"` |
| `chunksize` | Leer por bloques ([sección 18.4](#184-procesar-por-bloques-chunksize)) | `chunksize=100_000` |
| `engine` | Motor de lectura | `engine="pyarrow"` (más rápido, multinúcleo) |

**CSV "a la española"** (exportados por Excel o por portales de datos abiertos): separador `;`, coma decimal y codificación Latin-1:

```python
df = pd.read_csv(
    DATOS / "ventas_es.csv",
    sep=";",
    decimal=",",
    encoding="latin-1",
    parse_dates=["fecha"],
)
df.dtypes
```

> ¿No sabes cómo es el fichero? Míralo antes: `!head -n 5 datos/ventas_es.csv` en el notebook, o lee unas pocas filas con `nrows=5`. Ver el [manual de Python](../python/manual_python.md), sección 12.3.

Lectura rápida de solo lo necesario (útil para ficheros grandes):

```python
df = pd.read_csv(
    DATOS / "ventas.csv",
    usecols=["fecha", "tienda", "unidades", "precio"],
    dtype={"tienda": "category", "unidades": "int8"},
    parse_dates=["fecha"],
)
```

### 4.2 Excel

```python
df = pd.read_excel(DATOS / "ventas.xlsx", sheet_name="2026")

hojas = pd.read_excel(DATOS / "ventas.xlsx", sheet_name=None)   # todas: dict {nombre_hoja: DataFrame}
list(hojas)
```

Otros parámetros útiles: `skiprows`, `usecols="A:D"`, `header`, `nrows`. Excel es **lento** y tiene un límite de 1 048 576 filas: no es un formato para Big Data.

### 4.3 JSON

Un JSON con forma de **lista de registros** se lee directamente:

```python
from io import StringIO

texto = '[{"id": 1, "tienda": "Ferrol", "precio": 25.0}, {"id": 2, "tienda": "Vigo", "precio": 12.5}]'
pd.read_json(StringIO(texto))
```

**JSON Lines** (un objeto por línea, típico de logs y exportaciones):

```python
ventas.head(100).to_json(DATOS / "ventas.jsonl", orient="records", lines=True, date_format="iso")
df = pd.read_json(DATOS / "ventas.jsonl", lines=True)
```

**JSON anidado** (respuestas de API): `pd.json_normalize` "aplana" los diccionarios internos en columnas:

```python
pedidos = [
    {"id": "P1", "cliente": {"nombre": "Ana", "ciudad": "Ferrol"},
     "lineas": [{"producto": "teclado", "precio": 25.0}, {"producto": "ratón", "precio": 12.5}]},
    {"id": "P2", "cliente": {"nombre": "Brais", "ciudad": "Vigo"},
     "lineas": [{"producto": "monitor", "precio": 180.0}]},
]

pd.json_normalize(pedidos)                       # columnas: id, lineas, cliente.nombre, cliente.ciudad
pd.json_normalize(pedidos, record_path="lineas", meta=["id", ["cliente", "nombre"]])
```

Resultado del segundo:

```
  producto  precio  id cliente.nombre
0  teclado    25.0  P1            Ana
1    ratón    12.5  P1            Ana
2  monitor   180.0  P2          Brais
```

### 4.4 Parquet

**Parquet** es el formato de referencia en Big Data: columnar, comprimido, conserva los tipos y permite leer solo algunas columnas o filas.

```python
df = pd.read_parquet(DATOS / "ventas.parquet")
df = pd.read_parquet(DATOS / "ventas.parquet", columns=["fecha", "precio"])        # solo 2 columnas
df = pd.read_parquet(DATOS / "ventas.parquet", filters=[("tienda", "==", "Ferrol")])  # filtra al leer
```

También lee **carpetas** con muchos ficheros Parquet, incluidas las particiones `clave=valor` que genera Spark ([sección 5.2](#52-parquet-particionado)).

### 4.5 Bases de datos SQL

```python
import sqlite3

con = sqlite3.connect(DATOS / "tienda.db")
clientes.to_sql("clientes", con, if_exists="replace", index=False)   # escribir una tabla

df = pd.read_sql("SELECT provincia, COUNT(*) AS n FROM clientes GROUP BY provincia", con)
con.close()
```

Para PostgreSQL, MySQL, etc., se usa una conexión de **SQLAlchemy**:

```py
from sqlalchemy import create_engine
motor = create_engine("postgresql+psycopg://usuario:clave@localhost:5432/bigdata")
df = pd.read_sql("SELECT * FROM ventas WHERE fecha >= '2026-01-01'", motor)
```

### 4.6 Desde Internet

```py
url = "https://ejemplo.org/datos/municipios.csv"
df = pd.read_csv(url, sep=";", encoding="latin-1")

tablas = pd.read_html("https://es.wikipedia.org/wiki/Provincia_de_La_Coruña")   # lista de DataFrames
```

> Si vas a usar el fichero varias veces, **descárgalo una vez** y léelo en local (ver [manual de Python](../python/manual_python.md), sección 14.3).

### 4.7 Muchos ficheros a la vez

```python
for mes in [1, 2, 3]:                           # creamos 3 ficheros de ejemplo
    ventas[ventas["fecha"].dt.month == mes].to_csv(DATOS / f"ventas_2026-{mes:02d}.csv", index=False)

ficheros = sorted(DATOS.glob("ventas_2026-*.csv"))
df = pd.concat(
    (pd.read_csv(f).assign(origen=f.name) for f in ficheros),
    ignore_index=True,
)
df["origen"].value_counts()
```

---

## 5. Guardar datos y elegir formato

```python
SALIDA = Path("resultados")
SALIDA.mkdir(exist_ok=True)

ventas.to_csv(SALIDA / "ventas.csv", index=False)                       # index=False casi siempre ✅
ventas.to_csv(SALIDA / "ventas.csv.gz", index=False)                    # comprimido
ventas.to_csv(SALIDA / "ventas_excel.csv", index=False, sep=";", decimal=",", encoding="utf-8-sig")  # para abrir en Excel ES
ventas.to_parquet(SALIDA / "ventas.parquet", index=False)
ventas.to_json(SALIDA / "ventas.jsonl", orient="records", lines=True, date_format="iso", force_ascii=False)
ventas.to_excel(SALIDA / "ventas.xlsx", index=False)
```

Varias hojas en un mismo Excel:

```python
with pd.ExcelWriter(SALIDA / "informe.xlsx") as writer:
    ventas.to_excel(writer, sheet_name="ventas", index=False)
    clientes.to_excel(writer, sheet_name="clientes", index=False)
```

> Sin `index=False`, pandas guarda el índice como una columna más y al volver a leer aparece una columna `Unnamed: 0`.

### 5.1 ¿Qué formato elegir?

| | CSV | JSON / JSON Lines | Excel | **Parquet** |
|---|---|---|---|---|
| Legible por personas | ✅ | ✅ | ✅ (con Excel) | ❌ (binario) |
| Conserva tipos (fechas, enteros, categorías) | ❌ | Parcial | Parcial | ✅ |
| Tamaño en disco | Grande | Muy grande | Medio | **Pequeño** (comprimido por columnas) |
| Velocidad de lectura | Lenta | Lenta | Muy lenta | **Muy rápida** |
| Leer solo algunas columnas | ❌ (lee todo) | ❌ | ❌ | ✅ |
| Datos anidados | ❌ | ✅ | ❌ | ✅ |
| Uso típico | Intercambio, datos abiertos | APIs, logs | Informes para personas | **Almacenamiento y análisis Big Data** (Spark, Hive, data lakes) |

**Regla del módulo:** los datos llegan en CSV/JSON/Excel → se limpian con pandas → se guardan en **Parquet** para trabajar con ellos.

### 5.2 Parquet particionado

Guardar en carpetas por el valor de una o varias columnas, como hace Spark:

```python
(ventas
 .assign(mes=ventas["fecha"].dt.month)
 .to_parquet(SALIDA / "ventas_part", partition_cols=["tienda", "mes"], index=False))
```

```
resultados/ventas_part/
├── tienda=Coruña/mes=1/xxxx.parquet
├── tienda=Coruña/mes=2/xxxx.parquet
└── ...
```

Al leer con filtros por la columna de partición, **solo se abren las carpetas necesarias**:

```python
pd.read_parquet(SALIDA / "ventas_part", filters=[("tienda", "==", "Ferrol")]).shape
```

---

## 6. Primera exploración

Lo primero que se hace con cualquier dataset nuevo:

```python
ventas.shape             # (filas, columnas): (1005, 8)
ventas.head()            # primeras 5 filas (head(10) para 10)
ventas.tail(3)           # últimas 3
ventas.sample(5, random_state=0)   # 5 filas al azar (mejor que head para ver variedad)
ventas.columns           # nombres de columnas
ventas.dtypes            # tipo de cada columna
ventas.info()            # tipos, nº de no nulos y memoria
```

```python
ventas.describe()                    # estadísticas de las columnas numéricas
ventas.describe(include="all")       # todas, incluidas las de texto
ventas.describe(include=["str", "category"])  # solo texto y categorías
```

```python
ventas["tienda"].unique()                  # valores distintos
ventas["tienda"].nunique()                 # cuántos valores distintos
ventas["tienda"].value_counts()            # frecuencia de cada valor
ventas["tienda"].value_counts(normalize=True)   # en proporción
ventas["categoria"].value_counts(dropna=False)  # incluyendo nulos
ventas.nunique()                           # nº de valores distintos por columna
```

```python
ventas.isna().sum()                        # nulos por columna
ventas.isna().mean().mul(100).round(1)     # % de nulos por columna
ventas.duplicated().sum()                  # filas duplicadas
ventas.memory_usage(deep=True).sum() / 1024**2   # memoria en MB
```

Estadísticos sueltos:

```python
ventas["precio"].mean()
ventas["precio"].median()
ventas["precio"].std()
ventas["precio"].min(), ventas["precio"].max()
ventas["precio"].quantile([0.25, 0.5, 0.75, 0.95])
ventas[["unidades", "precio"]].corr()      # correlación entre columnas numéricas
```

> Las operaciones estadísticas **ignoran los nulos** por defecto (`skipna=True`).

---

## 7. Seleccionar y filtrar

### 7.1 Columnas

```python
ventas["precio"]                         # una columna → Series
ventas[["tienda", "precio"]]             # varias columnas → DataFrame (doble corchete)
ventas.precio                            # atajo de lectura (no funciona con espacios ni tildes)
ventas.filter(like="id")                 # columnas cuyo nombre contiene "id"
ventas.select_dtypes("number")           # solo columnas numéricas
ventas.select_dtypes(exclude="number")
```

### 7.2 Filas y celdas: `loc` e `iloc`

| | `loc` | `iloc` |
|---|---|---|
| Selecciona por | **Etiquetas** (nombres de índice y columnas) o condiciones | **Posiciones** enteras (0, 1, 2…) |
| Rango `a:b` | **Incluye** `b` | **Excluye** `b` (como en Python) |

```python
ventas.loc[0]                            # fila con etiqueta 0
ventas.loc[0:4, ["tienda", "precio"]]    # filas 0 a 4 (incluida), dos columnas
ventas.loc[ventas["precio"] > 250, "tienda"]   # condición + columna

ventas.iloc[0]                           # primera fila
ventas.iloc[:5, :3]                      # 5 primeras filas, 3 primeras columnas
ventas.iloc[-1]                          # última fila

ventas.at[0, "precio"]                   # una celda (rápido)
ventas.iat[0, 6]                         # una celda por posición
```

### 7.3 Filtrar con condiciones (*boolean indexing*)

Una comparación sobre una columna devuelve una `Series` de `True`/`False`, que sirve como **máscara**:

```python
ventas["precio"] > 250                   # Series booleana
ventas[ventas["precio"] > 250]           # filas que cumplen
```

Combinar condiciones con `&` (y), `|` (o), `~` (no), **cada condición entre paréntesis**:

```python
caras_ferrol = ventas[(ventas["tienda"] == "Ferrol") & (ventas["precio"] > 250)]
ventas[(ventas["canal"] == "online") | (ventas["unidades"] >= 5)]
ventas[~(ventas["categoria"] == "Libros")]
```

> ⚠️ No uses `and` / `or` con pandas: da el error *"The truth value of a Series is ambiguous"*. Usa `&` y `|`.

Otros filtros muy útiles:

```python
ventas[ventas["tienda"].isin(["Ferrol", "Coruña"])]      # pertenece a una lista
ventas[ventas["precio"].between(50, 100)]                # entre dos valores (incluidos)
ventas[ventas["precio"].isna()]                          # nulos
ventas[ventas["precio"].notna()]
ventas[ventas["tienda"].str.contains("vigo", case=False, na=False)]   # texto que contiene
ventas[ventas["fecha"] >= "2026-06-01"]                  # fechas: comparar con texto ISO
```

### 7.4 `query`: filtros legibles

```python
ventas.query("tienda == 'Ferrol' and precio > 250")
ventas.query("tienda in ['Ferrol', 'Lugo'] and unidades >= 4")

umbral = 200
ventas.query("precio > @umbral")                         # @ para usar variables de Python
```

---

## 8. Crear y modificar columnas

### 8.1 Operaciones vectorizadas

```python
ventas["importe"] = ventas["unidades"] * ventas["precio"]
ventas["importe_iva"] = (ventas["importe"] * 1.21).round(2)
ventas["es_online"] = ventas["canal"] == "online"          # columna booleana
```

`assign` crea columnas devolviendo un DataFrame **nuevo** (ideal para encadenar, [sección 17](#17-encadenar-operaciones-y-copy-on-write)):

```python
ventas.assign(
    importe=lambda d: d["unidades"] * d["precio"],
    mes=lambda d: d["fecha"].dt.month,
).head()
```

### 8.2 Valores condicionales

```python
# Dos opciones
ventas["tipo"] = np.where(ventas["importe"] > 500, "grande", "normal")

# Varias opciones: np.select (la primera condición que se cumpla)
condiciones = [ventas["importe"] >= 1000, ventas["importe"] >= 300]
ventas["tramo"] = np.select(condiciones, ["alto", "medio"], default="bajo")

# Asignar solo en las filas que cumplen una condición
ventas.loc[ventas["canal"] == "online", "gastos_envio"] = 4.95
```

### 8.3 Tramos: `cut` y `qcut`

```python
ventas["rango_precio"] = pd.cut(
    ventas["precio"],
    bins=[0, 50, 150, np.inf],
    labels=["económico", "medio", "premium"],
)
ventas["cuartil_importe"] = pd.qcut(ventas["importe"], q=4, labels=["Q1", "Q2", "Q3", "Q4"])
```

`cut` usa límites fijos; `qcut`, cuantiles (tramos con el mismo número de filas).

### 8.4 Sustituir y mapear valores

```python
ventas["canal"].replace({"tienda": "presencial"})                 # sustituye valores
ventas["categoria"].map({"Informática": "TEC", "Hogar": "HOG", "Libros": "LIB"})   # traduce con un dict
```

> Con `map`, los valores que no están en el diccionario quedan como `NaN`; con `replace`, se quedan como estaban.

### 8.5 `apply`: solo cuando no hay alternativa vectorizada

```python
ventas["precio"].apply(lambda p: round(p * 1.21, 2))     # funciona, pero es LENTO
ventas["precio"].mul(1.21).round(2)                      # ✅ equivalente vectorizado, mucho más rápido

ventas.apply(lambda fila: f"{fila['tienda']}-{fila['id_venta']}", axis=1)   # por filas: aún más lento
ventas["tienda"] + "-" + ventas["id_venta"].astype(str)                    # ✅ vectorizado
```

> `apply` ejecuta una función de Python **fila a fila**: en millones de filas puede ser cientos de veces más lento. Úsalo solo cuando la operación no exista como método de pandas o NumPy.

### 8.6 Renombrar, eliminar y reordenar

```python
ventas = ventas.rename(columns={"id_cliente": "cliente"})
ventas = ventas.rename(columns={"cliente": "id_cliente"})            # la dejamos como estaba

ventas.drop(columns=["importe_iva", "es_online"])                     # devuelve una copia sin esas columnas
ventas.drop(index=[0, 1])                                             # sin esas filas

ventas = ventas[["id_venta", "fecha", "tienda", "categoria", "canal",
                 "unidades", "precio", "importe", "id_cliente"]]     # reordenar (y quedarse con estas)
```

### 8.7 Índice

```python
ventas.set_index("id_venta")             # una columna pasa a ser el índice
ventas.reset_index(drop=True)            # índice 0..n-1 de nuevo (drop=True descarta el anterior)
```

---

## 9. Tipos de datos

Que cada columna tenga el **tipo correcto** es imprescindible: con tipo texto no se puede sumar, con tipo número no se pueden extraer meses…

| Tipo (`dtype`) | Contiene | Ejemplo |
|---|---|---|
| `int64`, `int32`, `int8`… | Enteros | unidades |
| `float64`, `float32` | Decimales (y `NaN`) | precio |
| `bool` | `True` / `False` | es_online |
| `str` (pandas 3) / `object` (pandas 2) | Texto | tienda |
| `category` | Texto con **pocos valores distintos** repetidos | tienda, categoria |
| `datetime64[ns]` / `[us]` | Fechas y horas | fecha |
| `timedelta64` | Duraciones | fecha_fin - fecha_inicio |
| `Int64`, `Float64`, `boolean` (con mayúscula) | Versiones que admiten nulos (`<NA>`) sin perder el tipo | id con huecos |

### 9.1 Convertir tipos

```python
ventas["unidades"] = ventas["unidades"].astype("int16")
ventas["tienda"] = ventas["tienda"].astype("category")
ventas["id_cliente"] = ventas["id_cliente"].astype(str)
ventas["id_cliente"] = ventas["id_cliente"].astype(int)
```

### 9.2 Números guardados como texto

Es el problema más común al leer CSV: una columna numérica aparece como texto porque tiene algún valor "raro".

```python
s = pd.Series(["12,5", "7", "n/d", "1.234,0", None])

pd.to_numeric(s, errors="coerce")       # lo que no se puede convertir → NaN
# 0      NaN   ← "12,5" no es un número válido para Python
# 1      7.0
# ...

limpio = s.str.replace(".", "", regex=False).str.replace(",", ".", regex=False)
pd.to_numeric(limpio, errors="coerce")  # 12.5, 7.0, NaN, 1234.0, NaN
```

> La mejor solución es leer bien desde el principio: `pd.read_csv(..., decimal=",", thousands=".", na_values=["n/d"])`.

### 9.3 Enteros con nulos

Una columna de enteros con algún nulo se convierte en `float64` (`NaN` es un `float`). Para mantenerla entera:

```python
pd.Series([1, 2, None])                 # float64: 1.0, 2.0, NaN
pd.Series([1, 2, None], dtype="Int64")  # Int64:   1, 2, <NA>
ventas.convert_dtypes().dtypes          # convierte cada columna al mejor tipo "nullable"
```

---

## 10. Limpieza de datos

Flujo habitual: **nombres de columnas → tipos → texto → duplicados → nulos → valores imposibles / atípicos**. Documenta cada decisión: cuántas filas se pierden y por qué.

### 10.1 Normalizar los nombres de columnas

```python
sucio = pd.DataFrame(columns=[" Fecha Venta", "Importe (€)", "Código Postal", "TIENDA"])

sucio.columns = (
    sucio.columns
    .str.strip()
    .str.lower()
    .str.normalize("NFKD").str.encode("ascii", "ignore").str.decode("ascii")   # quita tildes
    .str.replace(r"[^\w]+", "_", regex=True)
    .str.strip("_")
)
list(sucio.columns)    # ['fecha_venta', 'importe', 'codigo_postal', 'tienda']
```

### 10.2 Limpiar texto: el accesor `.str`

```python
ventas["tienda"].value_counts()          # aparece " vigo " además de "Vigo"

ventas["tienda"] = ventas["tienda"].str.strip().str.title()
ventas["tienda"].value_counts()          # ya solo 4 tiendas
```

Métodos `.str` más usados:

| Método | Qué hace |
|---|---|
| `.str.strip()`, `.str.lower()`, `.str.upper()`, `.str.title()` | Espacios y mayúsculas |
| `.str.replace("a", "b", regex=False)` | Sustituir texto |
| `.str.replace(r"\s+", " ", regex=True)` | Sustituir con expresión regular |
| `.str.contains("x", case=False, na=False)` | ¿Contiene? |
| `.str.startswith("x")`, `.str.endswith("x")` | Empieza / termina por |
| `.str.len()` | Longitud |
| `.str[:3]` | Subcadena |
| `.str.split("-", expand=True)` | Dividir en varias columnas |
| `.str.extract(r"(\d{5})")` | Extraer con expresión regular |
| `.str.zfill(5)` | Rellenar con ceros (códigos postales: `"1500"` → `"01500"`) |
| `.str.cat(sep=", ")` | Unir todos los valores |

```python
codigos = pd.Series(["ES-15-Ferrol", "ES-36-Vigo", "ES-27-Lugo"])
codigos.str.split("-", expand=True)                    # 3 columnas: ES | 15 | Ferrol
codigos.str.extract(r"ES-(?P<provincia>\d+)-(?P<ciudad>\w+)")   # columnas con nombre
```

### 10.3 Valores inconsistentes

```python
estados = pd.Series(["Sí", "si", "SI", "S", "No", "no", "N", None])
estados.str.lower().str[0].map({"s": True, "n": False})
```

### 10.4 Duplicados

```python
ventas.duplicated().sum()                                 # filas idénticas
ventas[ventas.duplicated(keep=False)]                     # ver todas las copias
ventas.duplicated(subset=["id_venta"]).sum()              # duplicados por clave

ventas = ventas.drop_duplicates()                         # elimina filas idénticas (conserva la primera)
ventas = ventas.drop_duplicates(subset=["id_venta"], keep="last")   # por clave, conserva la última
```

### 10.5 Valores nulos

pandas representa los nulos como `NaN` (números), `None`/`NaN` (texto), `NaT` (fechas) o `<NA>` (tipos *nullable*). Todos se detectan con `isna()`.

```python
ventas.isna().sum()
ventas[ventas["precio"].isna()].head()
```

**Opciones para tratarlos:**

| Estrategia | Código | Cuándo |
|---|---|---|
| Eliminar filas | `df.dropna(subset=["precio"])` | Pocos nulos en una columna imprescindible |
| Eliminar columnas | `df.dropna(axis=1, thresh=int(len(df) * 0.5))` | Columnas con más del 50 % de nulos |
| Valor fijo | `df.fillna({"canal": "desconocido"})` | Categorías: crear la categoría "desconocido" |
| Media / mediana | `df["precio"].fillna(df["precio"].median())` | Numéricas sin estructura temporal (la mediana resiste atípicos) |
| Media **por grupo** | `df.groupby("categoria")["precio"].transform("median")` | Más realista: la mediana de su categoría |
| Valor anterior / siguiente | `df["temp"].ffill()`, `.bfill()` | Series temporales (sensores) |
| Interpolación | `df["temp"].interpolate()` | Series temporales numéricas |

```python
# Imputar el precio con la mediana de su categoría
mediana_cat = ventas.groupby("categoria")["precio"].transform("median")
ventas["precio"] = ventas["precio"].fillna(mediana_cat)
ventas["importe"] = ventas["unidades"] * ventas["precio"]
ventas["precio"].isna().sum()            # 0
```

> Si imputas, es buena práctica crear antes una columna indicadora (`df["precio_imputado"] = df["precio"].isna()`) para no perder la información de qué valores eran reales.

### 10.6 Valores imposibles y atípicos (*outliers*)

Primero, reglas de negocio (**valores imposibles**):

```python
ventas[(ventas["unidades"] <= 0) | (ventas["precio"] < 0)]     # no debería haber ninguna
```

Después, **atípicos** estadísticos con el método del rango intercuartílico (IQR):

```python
q1, q3 = ventas["importe"].quantile([0.25, 0.75])
iqr = q3 - q1
lim_inf, lim_sup = q1 - 1.5 * iqr, q3 + 1.5 * iqr

atipicos = ventas[(ventas["importe"] < lim_inf) | (ventas["importe"] > lim_sup)]
print(f"{len(atipicos)} atípicos fuera de [{lim_inf:.0f}, {lim_sup:.0f}]")

ventas["importe_recortado"] = ventas["importe"].clip(lower=lim_inf, upper=lim_sup)   # o recortarlos
```

> Un atípico **no siempre es un error** (una venta muy grande puede ser real). Investígalo antes de eliminarlo.

---

## 11. Ordenar y clasificar

```python
ventas.sort_values("importe", ascending=False).head()
ventas.sort_values(["tienda", "importe"], ascending=[True, False])
ventas.sort_index()

ventas.nlargest(5, "importe")            # las 5 ventas mayores (más rápido que ordenar todo)
ventas.nsmallest(3, "precio")

ventas["ranking"] = ventas["importe"].rank(ascending=False, method="min")
ventas["ranking_tienda"] = ventas.groupby("tienda")["importe"].rank(ascending=False)
```

---

## 12. Agrupar y agregar

`groupby` sigue el esquema **dividir → aplicar → combinar** (*split-apply-combine*), el mismo que el `GROUP BY` de SQL y el `groupBy` de Spark:

```
  tienda  importe              dividir                 aplicar (sum)      combinar
  Ferrol     100     ┌─ Ferrol: 100, 50 ─────────▶      150         ┐     tienda  importe
  Vigo        80  ──▶├─ Vigo:    80, 20 ─────────▶      100         ├──▶  Ferrol     150
  Ferrol      50     └─ Lugo:    30     ─────────▶       30         ┘     Lugo        30
  Vigo        20                                                          Vigo       100
  Lugo        30
```

### 12.1 Agregaciones simples

```python
ventas.groupby("tienda")["importe"].sum()
ventas.groupby("tienda")["importe"].agg(["sum", "mean", "count"])
ventas.groupby(["tienda", "categoria"])["importe"].sum()          # varias claves → índice múltiple
ventas.groupby("tienda", as_index=False)["importe"].sum()          # tienda como columna, no como índice
ventas.groupby("tienda").size()                                     # nº de filas por grupo
```

Funciones de agregación: `sum`, `mean`, `median`, `min`, `max`, `count` (no nulos), `size` (todas las filas), `nunique`, `std`, `var`, `first`, `last`, `quantile`.

### 12.2 Agregación con nombre (recomendada)

Permite poner nombre a cada resultado y aplicar funciones distintas a cada columna:

```python
resumen = (
    ventas
    .groupby("tienda")
    .agg(
        n_ventas=("id_venta", "count"),
        unidades=("unidades", "sum"),
        facturacion=("importe", "sum"),
        ticket_medio=("importe", "mean"),
        clientes_unicos=("id_cliente", "nunique"),
        ultima_venta=("fecha", "max"),
    )
    .sort_values("facturacion", ascending=False)
    .round({"facturacion": 2, "ticket_medio": 2})
)
resumen
```

### 12.3 `transform`: resultado del grupo en cada fila

`agg` reduce cada grupo a una fila; `transform` devuelve **un valor por fila original**, ideal para comparar cada fila con su grupo:

```python
ventas["media_tienda"] = ventas.groupby("tienda")["importe"].transform("mean")
ventas["pct_sobre_tienda"] = ventas["importe"] / ventas.groupby("tienda")["importe"].transform("sum") * 100
ventas[["tienda", "importe", "media_tienda", "pct_sobre_tienda"]].head()
```

### 12.4 Filtrar grupos

```python
# Solo los clientes con más de 12 compras
ventas.groupby("id_cliente").filter(lambda g: len(g) > 12)["id_cliente"].nunique()

# Alternativa vectorizada (más rápida)
n_compras = ventas.groupby("id_cliente")["id_venta"].transform("count")
ventas[n_compras > 12]
```

### 12.5 Recorrer los grupos

```python
for tienda, grupo in ventas.groupby("tienda"):
    print(tienda, len(grupo), round(grupo["importe"].sum(), 2))
```

---

## 13. Reestructurar: tablas dinámicas, `melt` y `pivot`

### 13.1 Tablas dinámicas: `pivot_table`

Igual que una tabla dinámica de Excel:

```python
pd.pivot_table(
    ventas,
    index="tienda",              # filas
    columns="categoria",         # columnas
    values="importe",            # valores
    aggfunc="sum",               # cómo agregar
    fill_value=0,
    margins=True,                # fila y columna de totales ("All")
).round(0)
```

### 13.2 Tablas de frecuencias: `crosstab`

```python
pd.crosstab(ventas["tienda"], ventas["canal"])                      # recuentos
pd.crosstab(ventas["tienda"], ventas["canal"], normalize="index")   # % por fila
```

### 13.3 Formato ancho ↔ largo

```
 Formato ANCHO                         Formato LARGO
 tienda  ene  feb                      tienda  mes  ventas
 Ferrol  100  120     ── melt ──▶      Ferrol  ene     100
 Vigo     80   90     ◀── pivot ──     Ferrol  feb     120
                                       Vigo    ene      80
                                       Vigo    feb      90
```

El formato **largo** (una observación por fila) es el que necesitan `groupby`, los gráficos de seaborn y Spark. El **ancho** es más cómodo para leer.

```python
ancho = pd.DataFrame({"tienda": ["Ferrol", "Vigo"], "ene": [100, 80], "feb": [120, 90]})

largo = ancho.melt(id_vars="tienda", var_name="mes", value_name="ventas")
largo

largo.pivot(index="tienda", columns="mes", values="ventas").reset_index()
```

> `pivot` falla si hay **combinaciones repetidas** de índice y columna; en ese caso usa `pivot_table` con una función de agregación.

### 13.4 `stack` / `unstack`

Mueven un nivel del índice a las columnas y viceversa:

```python
por_tienda_cat = ventas.groupby(["tienda", "categoria"])["importe"].sum()
por_tienda_cat.unstack()          # categoria pasa a columnas
por_tienda_cat.unstack().stack()  # y vuelve
```

### 13.5 Listas dentro de celdas: `explode`

```python
pedidos = pd.DataFrame({"pedido": ["P1", "P2"], "productos": [["teclado", "ratón"], ["monitor"]]})
pedidos.explode("productos")       # una fila por producto
```

---

## 14. Combinar tablas: `concat`, `merge` y `join`

### 14.1 `concat`: apilar

```python
enero = ventas[ventas["fecha"].dt.month == 1]
febrero = ventas[ventas["fecha"].dt.month == 2]

pd.concat([enero, febrero], ignore_index=True)                 # una debajo de otra (mismas columnas)
pd.concat([enero, febrero], keys=["ene", "feb"])                # con etiqueta de origen
pd.concat([ventas[["tienda"]], ventas[["importe"]]], axis=1)    # una al lado de otra (por índice)
```

### 14.2 `merge`: unir por columnas clave (como un `JOIN` de SQL)

```python
ventas_cli = ventas.merge(clientes, on="id_cliente", how="left")
ventas_cli[["id_venta", "id_cliente", "segmento", "provincia"]].head()
```

| `how` | Resultado | SQL |
|---|---|---|
| `"inner"` (por defecto) | Solo filas con clave en **ambas** tablas | `INNER JOIN` |
| `"left"` | **Todas** las de la izquierda; `NaN` si no hay pareja | `LEFT JOIN` |
| `"right"` | Todas las de la derecha | `RIGHT JOIN` |
| `"outer"` | Todas las de ambas | `FULL OUTER JOIN` |
| `"cross"` | Todas las combinaciones | `CROSS JOIN` |

```
   ventas            clientes                 inner          left            outer
  ┌──────┐          ┌──────┐                ┌──────┐      ┌──────┐         ┌──────┐
  │  A   │──┐    ┌──│  A   │                │  A   │      │  A   │         │  A   │
  │  B   │  └────┘  │  C   │                └──────┘      │  B ∅ │         │  B ∅ │
  └──────┘          └──────┘                              └──────┘         │  ∅ C │
                                                                           └──────┘
```

Opciones útiles:

```python
# Claves con nombre distinto en cada tabla
clientes_ren = clientes.rename(columns={"id_cliente": "codigo"})
ventas.merge(clientes_ren, left_on="id_cliente", right_on="codigo", how="left")

# Varias claves
# a.merge(b, on=["tienda", "fecha"])

# Columnas con el mismo nombre en ambas tablas → sufijos
# a.merge(b, on="id", suffixes=("_ventas", "_clientes"))

# Diagnóstico: ¿de qué tabla viene cada fila?
diag = ventas.merge(clientes, on="id_cliente", how="outer", indicator=True)
diag["_merge"].value_counts()        # both / left_only / right_only

# Comprobar la relación esperada (lanza error si no se cumple)
ventas.merge(clientes, on="id_cliente", how="left", validate="many_to_one")
```

> ⚠️ **Comprueba siempre el número de filas** antes y después de un `merge`. Si la tabla de la derecha tiene **claves duplicadas**, las filas se multiplican sin avisar. `validate="many_to_one"` lo detecta.

### 14.3 `join`: unir por índice

```python
resumen_tienda = ventas.groupby("tienda")[["importe"]].sum()
objetivos = pd.DataFrame({"objetivo": [80_000, 90_000, 70_000, 85_000]},
                         index=["Coruña", "Ferrol", "Lugo", "Vigo"])
resumen_tienda.join(objetivos)
```

---

## 15. Fechas y series temporales

### 15.1 Convertir a fecha

```python
pd.to_datetime("2026-10-20")
pd.to_datetime(pd.Series(["20/10/2026", "21/10/2026"]), format="%d/%m/%Y")   # formato explícito ✅
pd.to_datetime(pd.Series(["20/10/2026", "fecha mala"]), format="%d/%m/%Y", errors="coerce")   # inválidas → NaT
pd.to_datetime(pd.Series([1792483200]), unit="s")       # marca de tiempo Unix (segundos)
```

> Indica siempre `format=` (o `dayfirst=True`) con fechas españolas: sin él, `"01/02/2026"` podría interpretarse como 2 de enero.

### 15.2 El accesor `.dt`

```python
f = ventas["fecha"]
ventas["anio"] = f.dt.year
ventas["mes"] = f.dt.month
ventas["dia_semana"] = f.dt.dayofweek          # 0 = lunes
ventas["nombre_dia"] = f.dt.day_name()         # 'Monday'...
ventas["hora"] = f.dt.hour
ventas["trimestre"] = f.dt.quarter
ventas["semana"] = f.dt.isocalendar().week
ventas["fin_de_semana"] = f.dt.dayofweek >= 5
ventas["solo_fecha"] = f.dt.normalize()        # quita la hora
ventas["periodo_mes"] = f.dt.to_period("M")    # 2026-01, 2026-02...
```

Diferencias entre fechas:

```python
(pd.Timestamp("2026-12-31") - ventas["fecha"]).dt.days.head()
ventas["fecha"] + pd.Timedelta(days=30)
ventas["fecha"] + pd.DateOffset(months=1)
```

### 15.3 Remuestreo: `resample`

Agrega por periodos de tiempo. Requiere que la fecha sea el índice (o usar `on=`):

```python
diario = ventas.resample("D", on="fecha")["importe"].sum()
mensual = ventas.resample("ME", on="fecha")["importe"].agg(["sum", "count"])
semanal_tienda = ventas.groupby("tienda").resample("W", on="fecha")["importe"].sum()
mensual
```

| Alias | Periodo | Alias | Periodo |
|---|---|---|---|
| `"min"` | Minuto | `"W"` | Semana (termina en domingo) |
| `"h"` | Hora | `"ME"` / `"MS"` | Mes (fin / inicio) |
| `"D"` | Día | `"QE"` | Trimestre |
| `"B"` | Día laborable | `"YE"` | Año |

> En pandas ≥ 2.2 se usan `"ME"`, `"QE"`, `"YE"` y `"h"`; los antiguos `"M"`, `"Q"`, `"Y"` y `"H"` dan error en pandas 3.

### 15.4 Ventanas móviles y comparaciones

```python
serie = diario.to_frame("importe")
serie["media_7d"] = serie["importe"].rolling(7).mean()          # media móvil de 7 días
serie["acumulado"] = serie["importe"].cumsum()
serie["dia_anterior"] = serie["importe"].shift(1)
serie["variacion"] = serie["importe"].diff()                     # diferencia con el anterior
serie["variacion_pct"] = serie["importe"].pct_change() * 100
serie.head(10)
```

### 15.5 Rangos de fechas y zonas horarias

```python
pd.date_range("2026-01-01", "2026-01-31", freq="D")            # todos los días de enero
pd.date_range("2026-01-01", periods=12, freq="MS")              # inicio de cada mes

utc = pd.Series(pd.to_datetime(["2026-10-20 10:00"])).dt.tz_localize("UTC")
utc.dt.tz_convert("Europe/Madrid")                              # 12:00+02:00
```

---

## 16. Gráficos rápidos

pandas dibuja directamente con matplotlib a través de `.plot()`:

```python
import matplotlib.pyplot as plt

mensual["sum"].plot(title="Facturación mensual", ylabel="€")
plt.show()

ventas.groupby("tienda")["importe"].sum().sort_values().plot.barh(title="Facturación por tienda")
plt.show()

ventas["precio"].plot.hist(bins=30, title="Distribución de precios")
plt.show()

ventas.plot.scatter(x="precio", y="importe", alpha=0.3)
plt.show()

ventas.boxplot(column="importe", by="tienda")
plt.show()
```

| Método | Gráfico |
|---|---|
| `.plot()` / `.plot.line()` | Líneas (series temporales) |
| `.plot.bar()` / `.plot.barh()` | Barras |
| `.plot.hist(bins=...)` | Histograma |
| `.plot.box()` | Diagrama de caja |
| `.plot.scatter(x=..., y=...)` | Dispersión |
| `.plot.pie()` | Sectores (úsalo poco) |

Más opciones de gráficos en el [manual de JupyterLab](../jupyterlab/manual_jupyterlab.md), sección 11.

---

## 17. Encadenar operaciones y *Copy-on-Write*

### 17.1 Encadenamiento de métodos (*method chaining*)

En lugar de crear muchas variables intermedias (`df1`, `df2`, `df_final`…), se encadenan las operaciones entre paréntesis, una por línea. Se lee de arriba abajo como una receta:

```python
top_tiendas_online = (
    ventas
    .query("canal == 'online'")
    .assign(mes=lambda d: d["fecha"].dt.month)
    .groupby(["tienda", "mes"], as_index=False)
    .agg(facturacion=("importe", "sum"))
    .sort_values("facturacion", ascending=False)
    .head(10)
)
top_tiendas_online
```

Para insertar una función propia en la cadena, `pipe`:

```python
def quitar_atipicos(df, col):
    q1, q3 = df[col].quantile([0.25, 0.75])
    iqr = q3 - q1
    return df[df[col].between(q1 - 1.5 * iqr, q3 + 1.5 * iqr)]

ventas.pipe(quitar_atipicos, "importe").shape
```

### 17.2 *Copy-on-Write*: modificar sin sorpresas

En pandas 3 **cualquier DataFrame obtenido de otro se comporta como una copia independiente**. Consecuencias:

```python
filtrado = ventas[ventas["tienda"] == "Ferrol"]
filtrado["precio"] = 0                     # modifica SOLO 'filtrado', nunca 'ventas' ✅
```

La **asignación encadenada** no funciona (no modifica nada y pandas avisa con `ChainedAssignmentError`):

```py
ventas[ventas["tienda"] == "Ferrol"]["precio"] = 0     # ❌ no cambia 'ventas'
ventas["precio"][0] = 0                                # ❌ tampoco
```

Forma correcta: **una sola operación con `loc`**:

```python
ventas.loc[ventas["tienda"] == "Ferrol", "gastos_envio"] = 0     # ✅
```

> En pandas 2.x estos casos generaban el famoso aviso `SettingWithCopyWarning`. La solución es la misma: `.loc[filas, columna] = valor`, o `.copy()` si quieres trabajar con un subconjunto independiente.

---

## 18. Rendimiento y grandes volúmenes

### 18.1 Medir antes de optimizar

```python
ventas.info(memory_usage="deep")
ventas.memory_usage(deep=True).sort_values(ascending=False) / 1024**2    # MB por columna
```

```py
%time df = pd.read_csv("datos/grande.csv")      # tiempo de una línea (en Jupyter)
!ls -lh datos/                                   # tamaño de los ficheros
!wc -l datos/grande.csv                          # nº de filas sin cargarlo
```

### 18.2 Leer solo lo necesario

```python
pd.read_csv(DATOS / "ventas.csv", nrows=1000)                          # una muestra para explorar
pd.read_csv(DATOS / "ventas.csv", usecols=["fecha", "tienda", "precio"])
pd.read_parquet(DATOS / "ventas.parquet", columns=["tienda", "precio"],
                filters=[("precio", ">", 100)])
```

### 18.3 Tipos que ocupan menos

| Cambio | Ahorro típico |
|---|---|
| Texto repetido → `category` | 90 % o más en esa columna |
| `int64` → `int32` / `int16` / `int8` | 50–87 % |
| `float64` → `float32` | 50 % (si no necesitas tanta precisión) |
| Fechas como texto → `datetime64` | Menos memoria y operaciones de fecha |

```python
def optimizar_tipos(df, max_categorias=0.5):
    """Reduce la memoria: enteros y decimales más pequeños y texto repetido como category."""
    df = df.copy()
    for col in df.select_dtypes("integer"):
        df[col] = pd.to_numeric(df[col], downcast="integer")
    for col in df.select_dtypes("float"):
        df[col] = pd.to_numeric(df[col], downcast="float")
    for col in df.select_dtypes(["object", "str"]):
        if df[col].nunique() / len(df) < max_categorias:
            df[col] = df[col].astype("category")
    return df

antes = ventas.memory_usage(deep=True).sum() / 1024**2
despues = optimizar_tipos(ventas).memory_usage(deep=True).sum() / 1024**2
print(f"{antes:.2f} MB → {despues:.2f} MB")
```

> Los tipos se pueden fijar **al leer** (`dtype={...}` en `read_csv`), lo que evita cargar primero la versión grande.

**Backend de Apache Arrow** (más compacto para texto y compatible con Parquet y Spark):

```python
df = pd.read_csv(DATOS / "ventas.csv", engine="pyarrow", dtype_backend="pyarrow")
df.dtypes.head()        # int64[pyarrow], string[pyarrow], ...
```

### 18.4 Procesar por bloques (`chunksize`)

Si el fichero **no cabe en memoria**, se lee por bloques y se agrega cada bloque. Solo se guarda en memoria el resultado parcial:

```python
parciales = []
for bloque in pd.read_csv(DATOS / "ventas.csv", chunksize=200,
                          usecols=["tienda", "unidades", "precio"]):
    bloque["importe"] = bloque["unidades"] * bloque["precio"]
    parciales.append(bloque.groupby("tienda")["importe"].agg(["sum", "count"]))

total = pd.concat(parciales).groupby(level=0).sum()      # combinar los resultados parciales
total["media"] = total["sum"] / total["count"]           # la media se calcula al final, no se promedian medias
total
```

> En un caso real el `chunksize` sería de 100 000 a 1 000 000 filas. **Ojo:** sumas y recuentos se pueden combinar; **medias y medianas no** (se calculan al final con sumas y recuentos, y la mediana exacta requiere otras técnicas).

Patrón **CSV grande → Parquet por bloques**, para trabajar después con Parquet:

```python
import pyarrow as pa
import pyarrow.parquet as pq

writer = None
for bloque in pd.read_csv(DATOS / "ventas.csv", chunksize=200, parse_dates=["fecha"]):
    tabla = pa.Table.from_pandas(bloque, preserve_index=False)
    if writer is None:
        writer = pq.ParquetWriter(SALIDA / "ventas_convertido.parquet", tabla.schema)
    writer.write_table(tabla)
writer.close()
```

### 18.5 Vectorizar: evitar bucles

| ❌ Lento | ✅ Rápido |
|---|---|
| `for i, fila in df.iterrows(): ...` | Operaciones sobre columnas: `df["a"] * df["b"]` |
| `df.apply(f, axis=1)` | `np.where`, `np.select`, métodos `.str` y `.dt` |
| `df["c"].apply(lambda x: x * 2)` | `df["c"] * 2` |
| Añadir filas una a una con `concat` en un bucle | Acumular en una lista y hacer **un** `concat` al final |
| Bucle con `if` para clasificar | `pd.cut`, `np.select`, `map` |

```python
import time
grande = pd.DataFrame({"a": np.random.rand(200_000), "b": np.random.rand(200_000)})

t = time.perf_counter(); grande.apply(lambda f: f["a"] * f["b"], axis=1); lento = time.perf_counter() - t
t = time.perf_counter(); grande["a"] * grande["b"]; rapido = time.perf_counter() - t
print(f"apply: {lento:.2f} s · vectorizado: {rapido:.4f} s · {lento / rapido:.0f} veces más rápido")
```

### 18.6 Resumen: estrategias por orden

1. **Leer menos**: `usecols`, `nrows`, `columns`/`filters` en Parquet.
2. **Tipos adecuados**: `category`, enteros pequeños, `dtype_backend="pyarrow"`.
3. **Parquet** en lugar de CSV para todo lo que se lea más de una vez.
4. **Vectorizar** en lugar de bucles y `apply`.
5. **Por bloques** (`chunksize`) si no cabe en memoria.
6. **Cambiar de herramienta** cuando nada de lo anterior basta ([sección 19](#19-de-pandas-a-spark-y-otras-herramientas)).

---

## 19. De pandas a Spark y otras herramientas

### 19.1 Alternativas cuando pandas se queda corto

| Herramienta | Qué es | Cuándo |
|---|---|---|
| **PySpark** | Motor distribuido: reparte el trabajo entre muchos núcleos o máquinas | Datos de decenas de GB o más, o ya en un clúster / *data lake*. **Es la herramienta principal del módulo** |
| `pyspark.pandas` | API de pandas que se ejecuta sobre Spark | Llevar código pandas a Spark con pocos cambios |
| **Polars** | DataFrames en Rust, multinúcleo y con ejecución diferida (*lazy*) | Datos grandes en un solo equipo, mucha velocidad |
| **DuckDB** | Base de datos analítica en proceso; SQL directo sobre CSV / Parquet / DataFrames | Consultas SQL rápidas sobre ficheros grandes |
| **Dask** | Paraleliza pandas en bloques | Escalar código pandas existente |

### 19.2 Equivalencias pandas ↔ PySpark

| Operación | pandas | PySpark |
|---|---|---|
| Leer CSV | `pd.read_csv("f.csv")` | `spark.read.csv("f.csv", header=True, inferSchema=True)` |
| Leer Parquet | `pd.read_parquet("f")` | `spark.read.parquet("f")` |
| Ver filas | `df.head()` | `df.show(5)` |
| Esquema | `df.dtypes` / `df.info()` | `df.printSchema()` |
| Seleccionar | `df[["a", "b"]]` | `df.select("a", "b")` |
| Filtrar | `df[df["a"] > 5]` | `df.filter(F.col("a") > 5)` |
| Nueva columna | `df.assign(c=df["a"] * 2)` | `df.withColumn("c", F.col("a") * 2)` |
| Agrupar | `df.groupby("a").agg(total=("b", "sum"))` | `df.groupBy("a").agg(F.sum("b").alias("total"))` |
| Unir | `df.merge(otra, on="id", how="left")` | `df.join(otra, on="id", how="left")` |
| Ordenar | `df.sort_values("a", ascending=False)` | `df.orderBy(F.desc("a"))` |
| Nº de filas | `len(df)` | `df.count()` |
| Guardar | `df.to_parquet("f")` | `df.write.parquet("f")` |

> Diferencia clave: **Spark es perezoso** (*lazy*). `select`, `filter`, `groupBy`… solo construyen un plan; nada se ejecuta hasta una **acción** (`show`, `count`, `collect`, `write`). pandas ejecuta cada línea al momento.

### 19.3 Pasar datos entre pandas y Spark

```py
from pyspark.sql import SparkSession
spark = SparkSession.builder.appName("pandas-spark").getOrCreate()
spark.conf.set("spark.sql.execution.arrow.pyspark.enabled", "true")   # conversión rápida con Arrow

sdf = spark.createDataFrame(ventas)          # pandas → Spark (datos pequeños o medianos)
resumen = (sdf.groupBy("tienda").sum("importe")
              .toPandas())                    # Spark → pandas: SOLO resultados pequeños
```

> ⚠️ `toPandas()` trae **todos** los datos al equipo del *driver*. Úsalo solo con resultados ya filtrados o agregados; con datos grandes, el proceso se queda sin memoria.

Patrón habitual en Big Data:

```
 Datos grandes (Parquet / HDFS / S3)
          │
          ▼
   PySpark: filtrar, limpiar, unir y AGREGAR      ← el trabajo pesado, distribuido
          │  .toPandas()  (resultado pequeño)
          ▼
   pandas: ajustes finales, tablas y gráficos      ← en el notebook
```

---

## 20. Buenas prácticas

1. **Explora antes de transformar**: `shape`, `info()`, `describe()`, `isna().sum()`, `value_counts()`.
2. **Lee bien desde el principio**: `sep`, `decimal`, `encoding`, `dtype`, `parse_dates`, `na_values`. Es más fácil que arreglarlo después.
3. **No modifiques los datos originales**: lee de `datos/brutos/`, guarda en `datos/procesados/` (en Parquet).
4. **Vectoriza**: evita `iterrows` y `apply` con `axis=1`.
5. Usa **`.loc[filas, columnas] = valor`** para asignar; nunca asignación encadenada.
6. **Comprueba el nº de filas** después de cada `merge`, `dropna` o `drop_duplicates`, y anota cuántas se pierden.
7. Prefiere **encadenar métodos** y funciones pequeñas (`pipe`) a muchas variables `df1`, `df2`…
8. Nombres de columnas en **minúsculas, sin espacios ni tildes** (`snake_case`): facilitan el código y el paso a Spark y SQL.
9. Usa `index=False` al guardar.
10. Trabaja con una **muestra** (`nrows`, `sample`) mientras desarrollas y con el dataset completo al final.

---

## 21. Solución de problemas frecuentes

| Problema | Causa probable | Solución |
|---|---|---|
| Todo el CSV aparece en **una sola columna** | Separador incorrecto | `sep=";"` (o `sep="\t"`) |
| `ParserError: Error tokenizing data. Expected 5 fields, saw 6` | Separador dentro de algún valor o filas mal formadas | Revisa `sep` y `quotechar`; para saltarlas: `on_bad_lines="skip"` (y cuenta cuántas) |
| `UnicodeDecodeError` | Codificación distinta de UTF-8 | `encoding="latin-1"` o `"cp1252"` |
| Primera columna `﻿id` o `Unnamed: 0` | BOM de Excel / índice guardado en el CSV | `encoding="utf-8-sig"` / `index_col=0` o guardar con `index=False` |
| Columna numérica con tipo `str`/`object` | Comas decimales, símbolos (`€`, `%`) o textos como `n/d` | `decimal=","`, `thousands="."`, `na_values=[...]`, `pd.to_numeric(..., errors="coerce")` |
| `KeyError: 'precio'` | La columna no existe con ese nombre exacto (espacios, mayúsculas, tildes) | `list(df.columns)`; normaliza nombres (sección 10.1) |
| `ValueError: The truth value of a Series is ambiguous` | `and`/`or` o `if serie:` con Series | Usa `&`, `|`, `~` y paréntesis |
| `ChainedAssignmentError` o `SettingWithCopyWarning` | Asignación encadenada `df[cond]["col"] = x` | `df.loc[cond, "col"] = x` (sección 17.2) |
| Tras un `merge` hay **más filas** que antes | Claves duplicadas en la tabla de la derecha | `otra["id"].duplicated().sum()`; `validate="many_to_one"` |
| Tras un `merge` hay muchos `NaN` | Claves con distinto tipo (`int` vs `str`) o espacios | Iguala tipos con `astype` y limpia con `.str.strip()` |
| Fechas con día y mes intercambiados | Formato ambiguo | `format="%d/%m/%Y"` o `dayfirst=True` |
| `ValueError: Invalid frequency: M` | Alias de frecuencia antiguo | `"ME"` (fin de mes), `"h"` (hora), `"YE"` (año) |
| `ImportError: Missing optional dependency 'openpyxl'` / `'pyarrow'` | Falta una librería opcional | `conda install -c conda-forge openpyxl pyarrow` |
| `MemoryError` o *Kernel died* | El dataset no cabe en memoria | Sección 18: `usecols`, tipos, `chunksize`, Parquet; o Spark |
| `groupby` no muestra algunas categorías | Categorías sin datos | `observed=False` para mostrarlas también |
| El notebook va lentísimo con un `apply` | Bucle de Python sobre millones de filas | Vectorizar (sección 18.5) |

---

## 22. Chuleta rápida

```py
import pandas as pd, numpy as np

# ---------- Leer ----------
pd.read_csv(r, sep=";", decimal=",", encoding="latin-1", usecols=[...], dtype={...}, parse_dates=["fecha"])
pd.read_excel(r, sheet_name="Hoja1");  pd.read_json(r, lines=True);  pd.json_normalize(lista)
pd.read_parquet(r, columns=[...], filters=[("col", "==", "x")]);  pd.read_sql(sql, con)

# ---------- Guardar ----------
df.to_parquet(r, index=False);  df.to_csv(r, index=False);  df.to_excel(r, index=False)

# ---------- Explorar ----------
df.shape; df.head(); df.sample(5); df.info(); df.describe(include="all")
df["c"].value_counts(); df["c"].nunique(); df.isna().sum(); df.duplicated().sum()

# ---------- Seleccionar / filtrar ----------
df["c"]; df[["a", "b"]]; df.loc[filas, cols]; df.iloc[0:5, 0:3]
df[(df["a"] > 5) & (df["b"] == "x")]; df[df["c"].isin([...])]; df.query("a > 5 and b == 'x'")

# ---------- Columnas ----------
df["n"] = df["a"] * df["b"];  df.assign(n=lambda d: d["a"] * 2)
np.where(cond, "si", "no");  np.select([c1, c2], ["A", "B"], default="C")
pd.cut(df["x"], bins=[...], labels=[...]);  df.rename(columns={...});  df.drop(columns=[...])
df["c"].astype("category");  pd.to_numeric(df["c"], errors="coerce")

# ---------- Limpieza ----------
df["t"].str.strip().str.lower();  df.drop_duplicates(subset=[...])
df.dropna(subset=[...]);  df.fillna({"c": 0});  df["x"].fillna(df.groupby("g")["x"].transform("median"))

# ---------- Agrupar ----------
df.groupby("g").agg(total=("x", "sum"), media=("x", "mean"), n=("id", "count"))
df.groupby("g")["x"].transform("mean")
pd.pivot_table(df, index="a", columns="b", values="x", aggfunc="sum", fill_value=0)
pd.crosstab(df["a"], df["b"]);  df.melt(id_vars="id");  df.pivot(index=, columns=, values=)

# ---------- Combinar ----------
pd.concat([df1, df2], ignore_index=True);  df.merge(otra, on="id", how="left", validate="many_to_one")

# ---------- Fechas ----------
pd.to_datetime(s, format="%d/%m/%Y");  df["f"].dt.month;  df["f"].dt.day_name()
df.resample("ME", on="f")["x"].sum();  s.rolling(7).mean();  s.shift(1);  s.pct_change()

# ---------- Grandes volúmenes ----------
df.memory_usage(deep=True).sum();  pd.read_csv(r, chunksize=100_000)
pd.read_csv(r, engine="pyarrow", dtype_backend="pyarrow")
```

---

## 23. Referencias

- Documentación oficial de pandas: <https://pandas.pydata.org/docs/>
- *10 minutes to pandas*: <https://pandas.pydata.org/docs/user_guide/10min.html>
- Guía de usuario (lectura y escritura, `groupby`, `merge`, series temporales…): <https://pandas.pydata.org/docs/user_guide/index.html>
- Novedades de pandas 3.0: <https://pandas.pydata.org/docs/whatsnew/v3.0.0.html>
- *Copy-on-Write* en pandas: <https://pandas.pydata.org/docs/user_guide/copy_on_write.html>
- Escalar a datasets grandes: <https://pandas.pydata.org/docs/user_guide/scale.html>
- Comparación con SQL: <https://pandas.pydata.org/docs/getting_started/comparison/comparison_with_sql.html>
- Hoja de referencia oficial (PDF): <https://pandas.pydata.org/Pandas_Cheat_Sheet.pdf>
- *Python for Data Analysis*, 3.ª ed., Wes McKinney (gratuito en línea): <https://wesmckinney.com/book/>
- Apache Parquet: <https://parquet.apache.org/>
- API de pandas sobre Spark: <https://spark.apache.org/docs/latest/api/python/user_guide/pandas_on_spark/index.html>
- Polars: <https://pola.rs/> · DuckDB: <https://duckdb.org/>

---

## 24. Ejemplo resuelto: de CSV en bruto a Parquet analizable

**Situación:** un portal de datos abiertos publica las lecturas de calidad del aire de varias estaciones en un CSV "a la española" (`;`, coma decimal, Latin-1), con nombres de columna irregulares, fechas en formato `dd/mm/aaaa hh:mm`, textos inconsistentes, valores `n/d`, duplicados y alguna lectura imposible. Queremos un dataset limpio en Parquet y un resumen mensual por estación.

**Paso 1 — Crear el fichero en bruto** (en la realidad, lo descargarías):

```python
from pathlib import Path
import numpy as np
import pandas as pd

BRUTOS = Path("datos/brutos"); BRUTOS.mkdir(parents=True, exist_ok=True)
PROCESADOS = Path("datos/procesados"); PROCESADOS.mkdir(parents=True, exist_ok=True)

rng = np.random.default_rng(7)
n = 2_000
crudo = pd.DataFrame({
    " Estación ": rng.choice(["Ferrol", "A Coruña", " ferrol", "Vigo", "VIGO"], n),
    "Fecha/Hora": pd.date_range("2026-01-01", periods=n, freq="3h").strftime("%d/%m/%Y %H:%M"),
    "NO2 (µg/m³)": rng.normal(30, 10, n).round(1).astype(str),
    "PM10 (µg/m³)": rng.normal(20, 8, n).round(1).astype(str),
})
crudo.loc[rng.choice(n, 40, replace=False), "NO2 (µg/m³)"] = "n/d"
crudo.loc[rng.choice(n, 5, replace=False), "PM10 (µg/m³)"] = "-999"           # código de error del sensor
crudo["NO2 (µg/m³)"] = crudo["NO2 (µg/m³)"].str.replace(".", ",", regex=False)
crudo["PM10 (µg/m³)"] = crudo["PM10 (µg/m³)"].str.replace(".", ",", regex=False)
crudo = pd.concat([crudo, crudo.sample(30, random_state=3)])                   # duplicados

crudo.to_csv(BRUTOS / "calidad_aire_2026.csv", sep=";", index=False, encoding="latin-1",
             errors="replace")
```

**Paso 2 — Inspeccionar y leer correctamente:**

```python
with open(BRUTOS / "calidad_aire_2026.csv", encoding="latin-1") as f:
    for _ in range(3):
        print(repr(f.readline()))

df = pd.read_csv(
    BRUTOS / "calidad_aire_2026.csv",
    sep=";",
    decimal=",",
    encoding="latin-1",
    na_values=["n/d", "-999"],          # -999 es el código de "sin lectura" del sensor
)
print(df.shape)
df.dtypes
```

**Paso 3 — Limpiar con una cadena de transformaciones documentada:**

```python
def normalizar_columnas(d):
    d = d.copy()
    d.columns = (d.columns.str.strip().str.lower()
                 .str.normalize("NFKD").str.encode("ascii", "ignore").str.decode("ascii")
                 .str.replace(r"\(.*\)", "", regex=True)          # quita las unidades entre paréntesis
                 .str.replace(r"[^\w]+", "_", regex=True).str.strip("_"))
    return d

filas_ini = len(df)

limpio = (
    df
    .pipe(normalizar_columnas)                                    # estacion, fecha_hora, no2, pm10
    .assign(
        estacion=lambda d: d["estacion"].str.strip().str.title().replace({"A Coruña": "Coruña"}),
        fecha=lambda d: pd.to_datetime(d["fecha_hora"], format="%d/%m/%Y %H:%M"),
    )
    .drop(columns="fecha_hora")
    .drop_duplicates()
    .query("no2 >= 0 or no2.isna()", engine="python")             # elimina lecturas negativas imposibles
    .sort_values(["estacion", "fecha"])
    .reset_index(drop=True)
)

print(f"Filas: {filas_ini} → {len(limpio)} ({filas_ini - len(limpio)} eliminadas)")
print(limpio["estacion"].value_counts())
limpio.isna().sum()
```

**Paso 4 — Imputar nulos con criterio temporal** (interpolación dentro de cada estación):

```python
limpio[["no2", "pm10"]] = (
    limpio.groupby("estacion")[["no2", "pm10"]]
    .transform(lambda s: s.interpolate(limit=2))      # rellena huecos de hasta 2 lecturas seguidas
)
limpio.isna().sum()
```

**Paso 5 — Tipo `category` y guardar en Parquet particionado:**

```python
limpio = limpio.astype({"estacion": "category"})
limpio["mes"] = limpio["fecha"].dt.month

limpio.to_parquet(PROCESADOS / "calidad_aire", partition_cols=["estacion"], index=False)
sorted(p.name for p in (PROCESADOS / "calidad_aire").iterdir())
```

**Paso 6 — Analizar leyendo solo lo necesario:**

```python
aire = pd.read_parquet(PROCESADOS / "calidad_aire", columns=["estacion", "fecha", "no2", "pm10"])

resumen_mensual = (
    aire
    .groupby(["estacion", pd.Grouper(key="fecha", freq="ME")], observed=True)
    .agg(no2_medio=("no2", "mean"), pm10_max=("pm10", "max"), lecturas=("no2", "count"))
    .round(1)
)
resumen_mensual.head(6)
```

```python
tabla = resumen_mensual["no2_medio"].unstack("estacion")
tabla.index = tabla.index.strftime("%Y-%m")
tabla.plot(title="NO₂ medio mensual por estación", ylabel="µg/m³", marker="o");
```

**Qué hemos aplicado:** lectura con `sep`, `decimal`, `encoding` y `na_values` (secc. 4.1), normalización de columnas y texto (10.1–10.2), fechas con formato explícito (15.1), duplicados y valores imposibles (10.4, 10.6), imputación por grupo y en orden temporal (10.5), tipos optimizados (18.3), Parquet particionado (5.2) y lectura selectiva y agregación temporal (4.4, 15.3). Es el mismo flujo **bruto → limpio → analítico** que se aplica a gran escala con Spark.
