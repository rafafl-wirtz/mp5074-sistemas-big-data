# Manual de JSON y YAML

**Módulo:** MP5074 – Sistemas de Big Data · Curso 2026-27

> **JSON** y **YAML** son dos formatos de texto para representar **datos estructurados**: objetos con claves, listas y valores anidados. JSON es el idioma de las **APIs**, de MongoDB y de los logs; YAML es el idioma de la **configuración**: el entorno de conda, Docker Compose, GitHub Actions, MkDocs… En este módulo los vas a leer y escribir en todas las unidades didácticas.
>
> Este manual da por hecho que tienes el entorno `bigdata` funcionando (ver los manuales de [Miniconda](../miniconda/manual_miniconda.md), [Python](../python/manual_python.md) y [pandas](../pandas/manual_pandas.md)). Los ejemplos usan dos ficheros del módulo: `datos/catalogo.json` (catálogo de 11 productos con datos anidados) y `datos/visitas_web.jsonl.gz` (≈250 000 eventos de navegación en JSON Lines comprimido).

---

## Índice

1. [¿Qué son JSON y YAML?](#1-qué-son-json-y-yaml)
2. [Instalación de herramientas](#2-instalación-de-herramientas)
3. [Sintaxis de JSON](#3-sintaxis-de-json)
4. [JSON en Python: el módulo `json`](#4-json-en-python-el-módulo-json)
5. [JSON Lines: un registro por línea](#5-json-lines-un-registro-por-línea)
6. [De JSON anidado a tabla con pandas](#6-de-json-anidado-a-tabla-con-pandas)
7. [JSON desde APIs y hacia MongoDB](#7-json-desde-apis-y-hacia-mongodb)
8. [JSON en la terminal: `json.tool` y `jq`](#8-json-en-la-terminal-jsontool-y-jq)
9. [Validar datos con JSON Schema](#9-validar-datos-con-json-schema)
10. [Sintaxis de YAML](#10-sintaxis-de-yaml)
11. [Trampas de YAML](#11-trampas-de-yaml)
12. [YAML en Python: PyYAML](#12-yaml-en-python-pyyaml)
13. [Los YAML del módulo](#13-los-yaml-del-módulo)
14. [Un fichero de configuración para tu pipeline](#14-un-fichero-de-configuración-para-tu-pipeline)
15. [Convertir entre JSON, YAML y otros formatos](#15-convertir-entre-json-yaml-y-otros-formatos)
16. [Validar y revisar: linters y editor](#16-validar-y-revisar-linters-y-editor)
17. [Seguridad](#17-seguridad)
18. [Solución de problemas frecuentes](#18-solución-de-problemas-frecuentes)
19. [Chuleta rápida](#19-chuleta-rápida)
20. [Referencias](#20-referencias)

---

## 1. ¿Qué son JSON y YAML?

**JSON** (*JavaScript Object Notation*) nació a principios de los 2000 como un subconjunto de JavaScript para intercambiar datos entre navegador y servidor. Hoy es un estándar independiente del lenguaje (RFC 8259 / ECMA-404) y el formato por defecto de casi cualquier API web.

**YAML** (*YAML Ain't Markup Language*) se diseñó para ser **cómodo de leer y escribir por personas**: usa la indentación en lugar de llaves y permite comentarios. La versión actual es la 1.2, y un detalle importante: **todo JSON válido es también YAML válido** (YAML 1.2 es un superconjunto de JSON).

El mismo dato en los dos formatos:

```json
{
  "sku": "P001",
  "nombre": "ALTAVOZ",
  "precios": {"coste": 57.55, "pvp": 92.92},
  "etiquetas": ["bluetooth", "portátil"],
  "activo": true
}
```

```yaml
# Producto del catálogo
sku: P001
nombre: ALTAVOZ
precios:
  coste: 57.55
  pvp: 92.92
etiquetas:
  - bluetooth
  - portátil
activo: true
```

**¿Cuál uso para qué?**

| | JSON | YAML |
|---|---|---|
| Pensado para | Máquinas (intercambio de datos) | Personas (configuración) |
| Comentarios | ❌ No | ✅ `# comentario` |
| Estructura | Llaves `{}` y corchetes `[]` | Indentación con espacios |
| Comillas | Obligatorias en claves y textos | Casi nunca necesarias |
| Parseo | Rápido, estricto, sin sorpresas | Más lento, con alguna trampa (sección 11) |
| Lo encontrarás en | APIs, MongoDB, logs, `.ipynb`, GeoJSON | conda, Docker Compose, GitHub Actions, MkDocs, Kubernetes |

**¿Dónde aparecen en el módulo?**

```
 OBTENER                ALMACENAR               ANALIZAR / PUBLICAR          CONFIGURAR (YAML)
┌──────────────────┐   ┌──────────────────┐   ┌────────────────────────┐   ┌──────────────────────┐
│ API REE  → JSON  │   │ data/raw/*.json  │   │ notebooks .ipynb (JSON)│   │ environment.yml      │
│ catalogo.json    │──▶│ MongoDB (BSON)   │──▶│ GeoJSON para mapas     │   │ docker-compose.yml   │
│ visitas .jsonl.gz│   │ JSON Lines       │   │ informes / dashboard   │   │ mkdocs.yml, deploy.yml│
└──────────────────┘   └──────────────────┘   └────────────────────────┘   │ config.yaml pipeline │
                                                                           └──────────────────────┘
```

> 💡 **Curiosidad:** un notebook de Jupyter (`.ipynb`) es un fichero JSON. Ábrelo con un editor de texto y verás celdas, salidas y metadatos como objetos JSON. Por eso los `diff` de Git sobre notebooks son tan ruidosos.

---

## 2. Instalación de herramientas

El módulo `json` viene con Python. Para YAML, validación y la terminal:

```bash
conda activate bigdata

# Librerías de Python
conda install -c conda-forge pyyaml jsonschema
# Opcionales: ruamel.yaml (conserva comentarios) y orjson (JSON muy rápido)
conda install -c conda-forge ruamel.yaml orjson

# Herramientas de terminal
conda install -c conda-forge jq yamllint check-jsonschema
```

| Paquete | Para qué |
|---|---|
| `json` (estándar) | Leer y escribir JSON |
| `pyyaml` | Leer y escribir YAML (`import yaml`) |
| `ruamel.yaml` | Editar YAML conservando comentarios y orden |
| `jsonschema` | Validar datos contra un esquema |
| `orjson` | JSON entre 5 y 10 veces más rápido (ficheros grandes) |
| `jq` | Consultar y transformar JSON en la terminal |
| `yamllint` | Revisar el estilo y los errores de un YAML |
| `check-jsonschema` | Validar ficheros JSON/YAML contra un esquema desde la terminal |

Comprobación:

```python
import json, yaml
from importlib.metadata import version
print(yaml.__version__)          # 6.0.x
print(version("jsonschema"))     # 4.x
```

**En VS Code** instala la extensión **YAML** (Red Hat): marca errores de indentación y autocompleta `docker-compose.yml`, workflows de GitHub y otros ficheros conocidos. JSON lo soporta de serie.

---

## 3. Sintaxis de JSON

### 3.1 Los seis tipos de valor

| Tipo JSON | Ejemplo | En Python |
|---|---|---|
| Objeto | `{"a": 1, "b": 2}` | `dict` |
| Array | `[1, 2, 3]` | `list` |
| Cadena | `"Galicia"` | `str` |
| Número | `42`, `-3.5`, `1.2e6` | `int` / `float` |
| Booleano | `true`, `false` | `True`, `False` |
| Nulo | `null` | `None` |

Con esos seis tipos se construye todo, anidando objetos y arrays a cualquier profundidad.

### 3.2 Reglas estrictas

JSON es estricto: un solo carácter fuera de sitio y el fichero no se puede leer.

```json
{
  "tienda": "Tecnoventas",
  "productos": 11,
  "regiones": ["Galicia", "Asturias"],
  "abierta": true,
  "web": null
}
```

- Las **claves** son siempre **cadenas entre comillas dobles**.
- Las cadenas usan **comillas dobles**, nunca simples.
- **Sin coma final** después del último elemento.
- **Sin comentarios** (ni `//` ni `/* */`).
- `true`, `false` y `null` en **minúsculas**.
- No existen `NaN`, `Infinity` ni fechas: las fechas se guardan como texto, preferiblemente en **ISO 8601** (`"2025-05-23T21:02:15+02:00"`).
- La codificación de intercambio es **UTF-8**.
- Caracteres especiales dentro de cadenas: `\"`, `\\`, `\n`, `\t`, `\u00f1` (ñ).

Errores típicos:

```text
{'nombre': 'ALTAVOZ'}            ❌ comillas simples
{"nombre": "ALTAVOZ",}           ❌ coma final
{nombre: "ALTAVOZ"}              ❌ clave sin comillas
{"activo": True}                 ❌ True es Python, en JSON es true
{"precio": NaN}                  ❌ NaN no existe en JSON
{"ruta": "C:\Users\ana"}         ❌ hay que escapar: "C:\\Users\\ana"
```

### 3.3 Formas típicas de un JSON de datos

Las APIs y los ficheros de datos suelen tener una de estas formas. Saber reconocerlas es la mitad del trabajo para convertirlas en tabla:

```json
// a) Lista de registros: lo más cómodo (una fila por objeto)
[{"region": "Galicia", "ventas": 120}, {"region": "Asturias", "ventas": 80}]

// b) Objeto "sobre" con metadatos y la lista dentro (catalogo.json, muchas APIs)
{"tienda": "Tecnoventas", "actualizado": "2025-12-31", "productos": [ {...}, {...} ]}

// c) Diccionario de columnas
{"region": ["Galicia", "Asturias"], "ventas": [120, 80]}

// d) Diccionario indexado por clave
{"P001": {"nombre": "ALTAVOZ"}, "P002": {"nombre": "AURICULARES"}}
```

*(Los comentarios `//` están solo para explicar: un JSON real no puede llevarlos.)*

---

## 4. JSON en Python: el módulo `json`

### 4.1 Las cuatro funciones

| Función | Qué hace | Mnemotecnia |
|---|---|---|
| `json.load(f)` | Fichero → objeto Python | *load* |
| `json.loads(texto)` | Cadena → objeto Python | *load **s**tring* |
| `json.dump(obj, f)` | Objeto Python → fichero | *dump* |
| `json.dumps(obj)` | Objeto Python → cadena | *dump **s**tring* |

### 4.2 Leer un fichero

```python
import json
from pathlib import Path

with open("datos/catalogo.json", encoding="utf-8") as f:
    catalogo = json.load(f)

type(catalogo)                    # dict
catalogo.keys()                   # dict_keys(['tienda', 'actualizado', 'moneda', 'productos'])
len(catalogo["productos"])        # 11

p = catalogo["productos"][0]
p["nombre"]                       # 'ALTAVOZ'
p["precios"]["pvp"]               # 92.92
p["etiquetas"][0]                 # 'bluetooth'

# Alternativa en una línea con pathlib
catalogo = json.loads(Path("datos/catalogo.json").read_text(encoding="utf-8"))
```

> ⚠️ Abre siempre los ficheros con `encoding="utf-8"`. En Windows el valor por defecto puede ser otro (cp1252) y las tildes salen mal o la lectura falla.

**Recorrer la estructura** es Python normal: diccionarios y listas.

```python
for p in catalogo["productos"]:
    margen = p["precios"]["pvp"] - p["precios"]["coste"]
    stock = sum(p["stock"].values())
    print(f'{p["sku"]} {p["nombre"]:<12} margen {margen:6.2f} €  stock {stock}')

# Comprensiones
audio = [p["nombre"] for p in catalogo["productos"] if p["categoria"] == "Audio"]
pvp_por_sku = {p["sku"]: p["precios"]["pvp"] for p in catalogo["productos"]}

# Acceso seguro a claves que pueden faltar
p.get("descuento", 0)                           # 0 si no existe
p.get("proveedor", {}).get("pais", "desconocido")
```

### 4.3 Escribir un fichero

```python
resumen = {
    "generado": "2026-10-23",
    "productos": len(catalogo["productos"]),
    "categorias": sorted({p["categoria"] for p in catalogo["productos"]}),
}

with open("data/processed/resumen.json", "w", encoding="utf-8") as f:
    json.dump(resumen, f, ensure_ascii=False, indent=2)
```

Parámetros que usarás siempre:

| Parámetro | Efecto |
|---|---|
| `indent=2` | Salida legible con saltos de línea. Sin él, todo en una línea (más compacto) |
| `ensure_ascii=False` | Escribe `"Galicia, Cataluña"` en lugar de `"Catalu\u00f1a"` |
| `sort_keys=True` | Ordena las claves (útil para comparar ficheros o versionarlos en Git) |
| `separators=(",", ":")` | Lo más compacto posible (sin espacios) |
| `default=str` | Convierte a texto lo que JSON no sabe guardar (fechas, `Decimal`, `Path`…) |
| `allow_nan=False` | Lanza error si hay `NaN`/`inf` en lugar de escribir JSON inválido |

```python
json.dumps({"región": "Cataluña"})                       # '{"r\\u00e9gion": "Catalu\\u00f1a"}'
json.dumps({"región": "Cataluña"}, ensure_ascii=False)   # '{"región": "Cataluña"}'
```

### 4.4 Lo que JSON no sabe guardar

JSON solo tiene seis tipos. Al convertir desde Python:

| Python | Resultado al hacer `dumps` |
|---|---|
| `tuple` | Se convierte en array (`list` al volver a leer) |
| `dict` con claves `int` | Las claves pasan a texto: `{1: "a"}` → `{"1": "a"}` |
| `float("nan")` | Escribe `NaN`, que **no es JSON válido** (otras herramientas fallarán) |
| `datetime`, `date` | ❌ `TypeError: Object of type datetime is not JSON serializable` |
| `Decimal`, `set`, `Path`, `numpy.int64` | ❌ `TypeError` |

Soluciones:

```python
from datetime import datetime, date
from decimal import Decimal
import numpy as np

datos = {"cuando": datetime(2025, 5, 23, 21, 2), "importe": Decimal("19.90"),
         "tags": {"a", "b"}, "n": np.int64(7)}

# 1) Rápido: convertir todo lo desconocido a texto
json.dumps(datos, default=str)

# 2) Controlado: una función que decide cómo convertir cada tipo
def a_json(obj):
    if isinstance(obj, (datetime, date)):
        return obj.isoformat()             # '2025-05-23T21:02:00'
    if isinstance(obj, Decimal):
        return float(obj)
    if isinstance(obj, set):
        return sorted(obj)
    if isinstance(obj, np.generic):        # np.int64, np.float64…
        return obj.item()
    raise TypeError(f"No sé convertir {type(obj).__name__}")

json.dumps(datos, default=a_json, ensure_ascii=False)
```

**Al leer** puedes pedir que los decimales lleguen como `Decimal` (importes exactos, sin errores de redondeo de `float`):

```python
from decimal import Decimal
json.loads('{"pvp": 92.92}', parse_float=Decimal)   # {'pvp': Decimal('92.92')}
```

### 4.5 Errores al leer

```python
texto = '{"sku": "P001", "pvp": 92.92,}'      # coma final

try:
    json.loads(texto)
except json.JSONDecodeError as e:
    print(f"JSON inválido: {e.msg} (línea {e.lineno}, columna {e.colno})")
# Python 3.13+: JSON inválido: Illegal trailing comma before end of object (línea 1, columna 29)
# Python 3.11:  JSON inválido: Expecting property name enclosed in double quotes (línea 1, columna 30)
```

El mensaje indica **línea y columna**: ve allí y mira el carácter anterior.

---

## 5. JSON Lines: un registro por línea

Un fichero JSON normal es **un único documento**: para leer el último registro hay que cargarlo entero en memoria. En Big Data se usa mucho **JSON Lines** (`.jsonl` o `.ndjson`): **un objeto JSON completo por línea**.

```text
{"ts": "2025-05-23T21:02:15+02:00", "sesion": "S000001", "usuario_id": 1758, "evento": "vista", "pagina": "/", "dispositivo": "movil", "region": "Cataluña", "duracion_s": 17}
{"ts": "2025-05-23T21:02:32+02:00", "sesion": "S000001", "usuario_id": 1758, "evento": "vista", "pagina": "/producto/tablet", "dispositivo": "movil", "region": "Cataluña", "duracion_s": 93}
```

**Ventajas:** se puede leer **línea a línea** sin cargarlo todo, **añadir** registros al final sin reescribir el fichero, **trocear** y **comprimir** (`.jsonl.gz`), y procesar en paralelo. Es el formato de los logs, de `mongoexport` y de muchas herramientas de streaming (Kafka, Spark).

### 5.1 Leer en streaming con Python

```python
import gzip, json
from collections import Counter

eventos = Counter()
with gzip.open("datos/visitas_web.jsonl.gz", "rt", encoding="utf-8") as f:   # "rt" = texto
    for linea in f:
        registro = json.loads(linea)
        eventos[registro["evento"]] += 1

eventos   # Counter({'vista': 230478, 'carrito': 13453, 'compra': 5731})
```

La memoria usada es la de **una línea**, da igual que el fichero tenga 250 000 o 250 millones de registros.

### 5.2 Escribir JSON Lines

```python
compras = [{"sesion": "S000017", "importe": 61.4}, {"sesion": "S000042", "importe": 154.26}]

with open("data/processed/compras.jsonl", "w", encoding="utf-8") as f:
    for c in compras:
        f.write(json.dumps(c, ensure_ascii=False) + "\n")

# Añadir al final sin tocar lo anterior (modo "a" = append)
with open("data/processed/compras.jsonl", "a", encoding="utf-8") as f:
    f.write(json.dumps({"sesion": "S000099", "importe": 50.9}) + "\n")
```

> ⚠️ En JSON Lines **no** se usa `indent`: cada registro debe ocupar exactamente una línea.

### 5.3 JSON Lines con pandas

```python
import pandas as pd

# Entero (pandas detecta la compresión .gz por la extensión)
visitas = pd.read_json("datos/visitas_web.jsonl.gz", lines=True)
visitas.shape          # (249662, 8)

# Por trozos: para ficheros que no caben en memoria
compras_por_region = Counter()
for trozo in pd.read_json("datos/visitas_web.jsonl.gz", lines=True, chunksize=50_000):
    c = trozo.loc[trozo["evento"] == "compra", "region"].value_counts()
    compras_por_region.update(c.to_dict())

# Escribir
visitas.query("evento == 'compra'").to_json(
    "data/processed/compras.jsonl", orient="records", lines=True, force_ascii=False)
```

> 💡 La columna `ts` lleva zonas horarias distintas (`+02:00` en verano, `+01:00` en invierno). Conviértela con `pd.to_datetime(visitas["ts"], utc=True)` y, si hace falta, `.dt.tz_convert("Europe/Madrid")`.

---

## 6. De JSON anidado a tabla con pandas

El problema habitual: un JSON con **objetos dentro de objetos y listas dentro de objetos** que hay que convertir en una **tabla plana** para analizar.

### 6.1 `pd.read_json`: cuando el JSON ya es casi una tabla

`read_json` funciona bien si el JSON es una lista de registros planos o tiene una de las formas estándar (`orient`):

```python
from io import StringIO

texto = '[{"region": "Galicia", "ventas": 120}, {"region": "Asturias", "ventas": 80}]'
pd.read_json(StringIO(texto))          # orient="records" (lo detecta solo)

texto = '{"region": ["Galicia", "Asturias"], "ventas": [120, 80]}'
pd.read_json(StringIO(texto))          # orient="columns"
```

> ⚠️ Desde pandas 2.1 pasar el JSON como **cadena** directamente está obsoleto (y en pandas 3 da error): envuélvelo en `StringIO`. Con una ruta de fichero o una URL no hace falta.

Valores de `orient` (sirven igual para `read_json` y `to_json`):

| `orient` | Forma del JSON |
|---|---|
| `"records"` | `[{"col": val, ...}, ...]` ← el más útil para intercambiar |
| `"columns"` | `{"col": {"índice": val}}` (por defecto en `to_json` de un DataFrame) |
| `"index"` | `{"índice": {"col": val}}` |
| `"split"` | `{"columns": [...], "index": [...], "data": [[...]]}` |
| `"values"` | Solo `[[...], [...]]` |
| `"table"` | Incluye un esquema con los tipos (reconstruye mejor el DataFrame) |

Si haces `pd.read_json("datos/catalogo.json")` obtendrás una tabla rara: una fila por producto con columnas `tienda`, `moneda`… repetidas y los productos como diccionarios en una columna. Para eso está `json_normalize`.

### 6.2 `pd.json_normalize`: aplanar objetos anidados

```python
import json
import pandas as pd

with open("datos/catalogo.json", encoding="utf-8") as f:
    catalogo = json.load(f)

productos = pd.json_normalize(catalogo["productos"])
productos.columns.tolist()
# ['sku', 'nombre', 'categoria', 'etiquetas', 'proveedor.nombre', 'proveedor.pais',
#  'precios.coste', 'precios.pvp', 'stock.almacen_central', 'stock.tiendas']
```

Cada objeto anidado se convierte en columnas `padre.hijo`. Las **listas** (`etiquetas`) se quedan como listas dentro de la celda.

Parámetros útiles:

```python
# Separador "_" para tener nombres de columna válidos como atributo (df.precios_pvp)
productos = pd.json_normalize(catalogo["productos"], sep="_")

# Aplanar solo hasta cierto nivel
pd.json_normalize(catalogo["productos"], max_level=0)     # no aplana nada

# Añadir metadatos del "sobre" a cada fila
productos["tienda"] = catalogo["tienda"]
productos["actualizado"] = pd.to_datetime(catalogo["actualizado"])
```

### 6.3 `record_path` y `meta`: una fila por elemento de una lista interior

Cuando la lista que quieres convertir en filas **está dentro** de cada registro, usa `record_path` (ruta hasta la lista) y `meta` (campos del padre que quieres conservar):

```python
# Una fila por (producto, etiqueta)
etiquetas = pd.json_normalize(
    catalogo["productos"],
    record_path="etiquetas",                 # la lista a "desenrollar"
    meta=["sku", "nombre", ["proveedor", "nombre"]],   # campos del padre (ruta anidada como lista)
    record_prefix="etiqueta_",
)
etiquetas.head(3)
#   etiqueta_0    sku     nombre  proveedor.nombre
# 0  bluetooth   P001    ALTAVOZ               JBL
# 1   portátil   P001    ALTAVOZ               JBL
# 2  bluetooth   P002 AURICULARES             Sony
```

Un caso real con la API de **REE** (Red Eléctrica): la respuesta trae una lista `included` de tecnologías y, dentro de cada una, la lista `attributes.values` con un valor por fecha.

```python
# Estructura (simplificada) de la respuesta de apidatos.ree.es
respuesta = {
  "included": [
    {"type": "Eólica", "attributes": {"title": "Eólica", "type": "Renovable",
        "values": [{"value": 12345.6, "datetime": "2024-01-01T00:00:00.000+01:00"},
                   {"value": 11000.2, "datetime": "2024-01-02T00:00:00.000+01:00"}]}},
    {"type": "Solar fotovoltaica", "attributes": {"title": "Solar fotovoltaica", "type": "Renovable",
        "values": [{"value": 4500.1, "datetime": "2024-01-01T00:00:00.000+01:00"}]}},
  ]
}

df = pd.json_normalize(
    respuesta["included"],
    record_path=["attributes", "values"],
    meta=["type", ["attributes", "type"]],
)
df.columns = ["valor_mwh", "fecha", "tecnologia", "clase"]
#    valor_mwh                          fecha          tecnologia      clase
# 0    12345.6  2024-01-01T00:00:00.000+01:00              Eólica  Renovable
# 1    11000.2  2024-01-02T00:00:00.000+01:00              Eólica  Renovable
# 2     4500.1  2024-01-01T00:00:00.000+01:00  Solar fotovoltaica  Renovable
```

> 💡 Si algún registro no tiene un campo de `meta`, añade `errors="ignore"` para que ponga `NaN` en lugar de lanzar `KeyError`.

### 6.4 Listas dentro de celdas: `explode`

Alternativa a `record_path` cuando ya tienes la tabla:

```python
productos = pd.json_normalize(catalogo["productos"], sep="_")

# Una fila por etiqueta (el resto de columnas se repite)
por_etiqueta = productos.explode("etiquetas")
por_etiqueta["etiquetas"].value_counts().head()

# O bien una columna booleana por etiqueta ("one-hot")
productos["etiquetas"].str.join("|").str.get_dummies()

# Contar elementos de la lista
productos["n_etiquetas"] = productos["etiquetas"].str.len()
```

### 6.5 Diccionarios dentro de celdas

A veces el JSON ya está en un DataFrame y una columna contiene diccionarios (por ejemplo, tras `read_json` o al leer de MongoDB):

```python
df = pd.DataFrame(catalogo["productos"])        # 'precios' contiene dicts

precios = pd.json_normalize(df["precios"]).add_prefix("precio_")
df = pd.concat([df.drop(columns="precios"), precios], axis=1)
```

### 6.6 Guardar: del DataFrame a JSON

```python
productos.to_json("data/processed/productos.json",
                  orient="records", force_ascii=False, indent=2, date_format="iso")
```

| Parámetro | Para qué |
|---|---|
| `orient="records"` | Lista de objetos, la forma más fácil de reutilizar |
| `lines=True` | JSON Lines (requiere `orient="records"`) |
| `force_ascii=False` | Conserva tildes y eñes |
| `date_format="iso"` | Fechas como `"2025-12-31T00:00:00.000"` en vez de milisegundos desde 1970 |
| `indent=2` | Legible |

> 💡 Para guardar datos **ya limpios** que vas a volver a leer con pandas, Spark o DuckDB, **Parquet** es mejor que JSON: ocupa menos, conserva los tipos y se lee mucho más rápido. JSON es para **intercambiar** o para datos **brutos** tal como llegaron.

---

## 7. JSON desde APIs y hacia MongoDB

### 7.1 APIs con `requests`

```python
import requests, json
from datetime import datetime
from pathlib import Path

url = "https://apidatos.ree.es/es/datos/generacion/estructura-generacion"
params = {"start_date": "2024-01-01T00:00", "end_date": "2024-01-31T23:59",
          "time_trunc": "day", "geo_trunc": "electric_system",
          "geo_limit": "peninsular", "geo_ids": "8741"}

r = requests.get(url, params=params, timeout=30)
r.raise_for_status()              # error si el código HTTP no es 2xx
r.headers["Content-Type"]         # 'application/json; charset=utf-8'
datos = r.json()                  # equivale a json.loads(r.text)

# Guardar la respuesta EN BRUTO, con fecha, antes de transformarla
destino = Path("data/raw") / f"ree_{datetime.now():%Y%m%d_%H%M%S}.json"
destino.write_text(json.dumps(datos, ensure_ascii=False, indent=2), encoding="utf-8")
```

**Para enviar** JSON a una API (POST), usa el parámetro `json=`: `requests` convierte el diccionario y pone la cabecera `Content-Type: application/json`.

```python
requests.post("https://httpbin.org/post", json={"sku": "P001", "unidades": 3}, timeout=10)
```

### 7.2 MongoDB: documentos JSON (BSON)

MongoDB guarda **documentos** con la misma forma que un objeto JSON. Internamente usa **BSON** (JSON binario), que añade tipos que JSON no tiene: fechas, `ObjectId`, enteros de 64 bits, `Decimal128`…

```python
from pymongo import MongoClient

cliente = MongoClient("mongodb://admin:admin@localhost:27017/")
col = cliente["tecnoventas"]["productos"]

col.insert_many(catalogo["productos"])        # cada dict → un documento
col.find_one({"sku": "P001"}, {"_id": 0})     # proyección: sin el _id
col.count_documents({"precios.pvp": {"$gt": 100}})   # notación con punto para campos anidados
```

Al exportar documentos de MongoDB a JSON aparece **Extended JSON**, que representa los tipos BSON con claves que empiezan por `$`:

```json
{"_id": {"$oid": "6718e5c2a1b2c3d4e5f60718"}, "fecha": {"$date": "2025-12-31T00:00:00Z"}}
```

```python
from bson import json_util
texto = json_util.dumps(col.find_one({"sku": "P001"}))   # convierte ObjectId y fechas
```

Desde la terminal (contenedor `mp5074-mongo` del `docker-compose.yml` de la sección 13.2):

```bash
# Exportar (JSON Lines por defecto; --jsonArray para un único array)
docker exec mp5074-mongo mongoexport -u admin -p admin --authenticationDatabase admin \
  -d tecnoventas -c productos --out /tmp/productos.jsonl

# Importar un JSON con array de documentos
mongoimport -u admin -p admin --authenticationDatabase admin \
  -d tecnoventas -c productos --jsonArray --file productos.json
```

---

## 8. JSON en la terminal: `json.tool` y `jq`

### 8.1 Formatear y validar con Python

```bash
# Formatea (y de paso valida: si hay error, dice línea y columna)
python -m json.tool datos/catalogo.json | head -20

# Opciones útiles
python -m json.tool --no-ensure-ascii --indent 2 entrada.json salida.json
python -m json.tool --json-lines compras.jsonl      # valida cada línea
```

### 8.2 `jq`: consultas sobre JSON

`jq` es para JSON lo que `grep`/`awk` son para texto. Se instala con `conda install -c conda-forge jq` (o `sudo apt install jq` en WSL). En PowerShell, escribe el filtro entre **comillas simples** igual que en Bash.

```bash
jq '.' datos/catalogo.json                     # formatear con colores
jq '.tienda' datos/catalogo.json               # "Tecnoventas"
jq '.productos | length' datos/catalogo.json   # 11
jq '.productos[0]' datos/catalogo.json         # primer producto
jq -r '.productos[].nombre' datos/catalogo.json      # -r: texto sin comillas, uno por línea
jq '.productos[] | {sku, pvp: .precios.pvp}' datos/catalogo.json   # nuevo objeto por producto
jq '[.productos[] | select(.categoria == "Audio") | .nombre]' datos/catalogo.json
jq '.productos | map(.precios.pvp) | add / length' datos/catalogo.json   # PVP medio
jq '.productos | sort_by(-.precios.pvp) | .[:3] | map(.nombre)' datos/catalogo.json
jq '[.productos[] | .etiquetas[]] | unique' datos/catalogo.json          # todas las etiquetas
jq '.productos | group_by(.categoria) | map({categoria: .[0].categoria, n: length})' datos/catalogo.json

# Exportar a CSV
jq -r '.productos[] | [.sku, .nombre, .precios.pvp] | @csv' datos/catalogo.json > productos.csv
```

Con JSON Lines, `jq` procesa cada línea por separado:

```bash
# Solo las compras, compactas (-c = una línea por objeto)
zcat datos/visitas_web.jsonl.gz | jq -c 'select(.evento == "compra")' | head -3

# Contar compras por dispositivo (-s junta todas las líneas en un array; ojo con la memoria)
zcat datos/visitas_web.jsonl.gz | jq -s 'map(select(.evento=="compra")) | group_by(.dispositivo) | map({(.[0].dispositivo): length}) | add'
```

| Filtro `jq` | Significado |
|---|---|
| `.campo`, `.a.b` | Acceder a un campo |
| `.[0]`, `.[2:5]`, `.[]` | Elemento, rebanada, todos los elementos |
| `\|` | Encadenar (como la tubería de la shell) |
| `select(cond)` | Filtrar |
| `map(f)`, `length`, `add`, `unique`, `sort_by(f)`, `group_by(f)` | Funciones sobre arrays |
| `{a, b: .x}` | Construir un objeto |
| `@csv`, `@tsv` | Convertir un array a una línea CSV/TSV (con `-r`) |
| `-r`, `-c`, `-s` | Texto crudo, salida compacta, leer todo como un array |

---

## 9. Validar datos con JSON Schema

Un **JSON Schema** es un documento JSON que describe **cómo debe ser** otro JSON: qué campos son obligatorios, qué tipo tiene cada uno, rangos válidos… Es la forma estándar de comprobar automáticamente que lo que llega de una API o de un proveedor tiene la estructura esperada **antes** de meterlo en el pipeline.

### 9.1 Un esquema para un producto del catálogo

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "Producto",
  "type": "object",
  "required": ["sku", "nombre", "categoria", "precios"],
  "properties": {
    "sku":       {"type": "string", "pattern": "^P[0-9]{3}$"},
    "nombre":    {"type": "string", "minLength": 1},
    "categoria": {"enum": ["Audio", "Informática", "Oficina", "Telefonía"]},
    "precios": {
      "type": "object",
      "required": ["coste", "pvp"],
      "properties": {
        "coste": {"type": "number", "exclusiveMinimum": 0},
        "pvp":   {"type": "number", "exclusiveMinimum": 0}
      }
    },
    "etiquetas": {"type": "array", "items": {"type": "string"}, "uniqueItems": true},
    "stock": {
      "type": "object",
      "additionalProperties": {"type": "integer", "minimum": 0}
    }
  }
}
```

Palabras clave más usadas:

| Palabra clave | Para |
|---|---|
| `type` | `string`, `number`, `integer`, `boolean`, `object`, `array`, `null` (o una lista: `["string", "null"]`) |
| `required` | Campos obligatorios de un objeto |
| `properties` / `additionalProperties` | Esquema de cada campo / de los campos no listados (`false` = prohibidos) |
| `enum`, `const` | Lista de valores permitidos / valor fijo |
| `minimum`, `maximum`, `exclusiveMinimum` | Rangos numéricos |
| `minLength`, `maxLength`, `pattern` | Longitud y expresión regular para textos |
| `format` | `date`, `date-time`, `email`, `uri`… (informativo salvo que actives la comprobación) |
| `items`, `minItems`, `uniqueItems` | Elementos de un array |

> 💡 El `enum` recoge las cuatro categorías actuales del catálogo. Si mañana llega un producto de una categoría nueva, el validador avisará: es justo para lo que sirve.

### 9.2 Validar en Python

```python
import json
from jsonschema import Draft202012Validator

with open("esquemas/producto.schema.json", encoding="utf-8") as f:
    esquema = json.load(f)

validador = Draft202012Validator(esquema)

for p in catalogo["productos"]:
    for error in validador.iter_errors(p):
        ruta = "/".join(map(str, error.path)) or "(raíz)"
        print(f'{p.get("sku", "?")}: {ruta} → {error.message}')
```

```python
malo = {"sku": "X1", "nombre": "", "categoria": "Audio", "precios": {"coste": -3}}
for e in validador.iter_errors(malo):
    print(list(e.path), e.message)
# ['sku'] 'X1' does not match '^P[0-9]{3}$'
# ['nombre'] '' should be non-empty
# ['precios'] 'pvp' is a required property
# ['precios', 'coste'] -3 is less than or equal to the minimum of 0
```

- `validate(instancia, esquema)` lanza `ValidationError` al **primer** error (útil para parar el pipeline).
- `iter_errors` los devuelve **todos** (útil para un informe de calidad).

### 9.3 Validar desde la terminal

```bash
check-jsonschema --schemafile esquemas/producto.schema.json productos/*.json

# check-jsonschema trae esquemas conocidos: valida tu workflow de GitHub Actions
check-jsonschema --builtin-schema vendor.github-workflows .github/workflows/deploy.yml
```

> 💡 Un esquema JSON también valida **YAML**: cargas el YAML como diccionario y lo validas igual (lo verás en la sección 14).

---

## 10. Sintaxis de YAML

### 10.1 Lo básico: mapas, listas y escalares

```yaml
# Esto es un comentario
tienda: Tecnoventas            # mapa (clave: valor) — ¡espacio después de los dos puntos!
moneda: EUR
productos: 11                  # entero
iva: 0.21                      # decimal
abierta: true                  # booleano
web: null                      # nulo (también ~ o vacío)

regiones:                      # lista: un guion y un espacio por elemento
  - Galicia
  - Asturias

proveedor:                     # mapa anidado: se indenta (2 espacios es lo habitual)
  nombre: JBL
  pais: Estados Unidos

productos_destacados:          # lista de mapas
  - sku: P001
    nombre: ALTAVOZ
  - sku: P002
    nombre: AURICULARES
```

Reglas clave:

- La **indentación** define la estructura. Usa **siempre espacios**, **nunca tabuladores** (YAML los prohíbe).
- Todos los elementos de un mismo nivel deben tener **exactamente la misma indentación**.
- Entre la clave y el valor: `clave: valor` (dos puntos **y espacio**).
- Los comentarios empiezan por `#` (con un espacio antes si van al final de una línea).
- Las claves no se pueden repetir dentro del mismo mapa.
- Extensiones: `.yaml` (la recomendada) o `.yml` (muy extendida); son equivalentes.

### 10.2 Estilo en línea (*flow style*)

Para estructuras cortas se puede escribir como en JSON:

```yaml
regiones: [Galicia, Asturias, Cantabria]
precios: {coste: 57.55, pvp: 92.92}
branches: [main]
```

Por eso **todo JSON es YAML válido**: `yaml.safe_load('{"a": [1, 2]}')` funciona.

### 10.3 Cadenas y comillas

En YAML casi nunca hace falta poner comillas, pero **a veces es obligatorio**:

```yaml
nombre: Ventas, stock y web        # sin comillas: vale
titulo: "MP5074: Sistemas de Big Data"   # contiene ": " → comillas obligatorias
comentario: "#no es un comentario"       # empieza por #
ruta_win: 'C:\Users\ana\datos'           # comillas simples: \ literal
salto: "línea 1\nlínea 2"                # comillas dobles: interpretan \n, \t, \u00f1
version_python: "3.10"                   # texto, no número (ver sección 11)
puerto: "27017:27017"                    # texto, no número
cita: 'Dijo ''hola'''                    # dentro de simples, la comilla se duplica
```

| Comillas | Escapes `\n`, `\t` | Uso |
|---|---|---|
| Sin comillas | No | La mayoría de textos |
| `'simples'` | No (todo literal) | Rutas de Windows, expresiones regulares |
| `"dobles"` | Sí | Cuando necesitas caracteres especiales |

### 10.4 Texto en varias líneas

```yaml
# | (literal): conserva los saltos de línea tal cual. Ideal para scripts y SQL
consulta: |
  SELECT region, SUM(importe) AS total
  FROM ventas
  GROUP BY region

# > (plegado): une las líneas con espacios (las líneas en blanco sí son saltos)
descripcion: >
  Pipeline del módulo MP5074
  que analiza la generación renovable
  a partir de datos de Red Eléctrica.

# Indicadores de final: |- quita el salto final, |+ conserva todos los finales
mensaje: |-
  Sin salto de línea al final
```

En GitHub Actions verás mucho `|` para ejecutar varios comandos:

```yaml
- run: |
    pip install -r requirements.txt
    mkdocs build --strict
```

### 10.5 Anclas, alias y fusión: no repetirse

```yaml
# & define un ancla, * la reutiliza
defecto: &conexion
  host: localhost
  puerto: 27017
  usuario: admin

desarrollo:
  <<: *conexion              # fusiona (merge) todas las claves del ancla
  base_datos: tecnoventas_dev

produccion:
  <<: *conexion
  host: mongo.ejemplo.local  # sobrescribe una clave
  base_datos: tecnoventas
```

Docker Compose usa mucho este truco con los campos `x-` (extensiones que Compose ignora):

```yaml
x-comun: &comun
  restart: unless-stopped
  environment:
    TZ: Europe/Madrid

services:
  mongo:
    <<: *comun
    image: mongo:7
  mongo-express:
    <<: *comun
    image: mongo-express:1.0.2
```

> ⚠️ La clave de fusión `<<` es de YAML 1.1. La soportan PyYAML, Docker Compose y muchas herramientas, pero **no todas** (por ejemplo, GitHub Actions no admitía anclas hasta 2025). Si un fichero lo va a leer otra herramienta, comprueba que la soporte.

### 10.6 Varios documentos en un fichero

`---` separa documentos dentro de un mismo fichero (muy común en Kubernetes). `...` marca el final opcional.

```yaml
---
nombre: entorno_dev
---
nombre: entorno_prod
```

```python
with open("entornos.yaml", encoding="utf-8") as f:
    docs = list(yaml.safe_load_all(f))     # lista de 2 diccionarios
```

### 10.7 Etiquetas de tipo

Puedes forzar el tipo de un valor con `!!`:

```yaml
codigo_postal: !!str 15009     # texto aunque parezca número
cantidad: !!int "42"
lista_vacia: !!seq []
```

En la práctica es más claro usar comillas. Las etiquetas `!Ref`, `!Sub`… que verás en AWS CloudFormation o `!!python/...` son extensiones que `safe_load` **no** acepta (y es bueno que no lo haga, ver sección 17).

---

## 11. Trampas de YAML

YAML **adivina el tipo** de los valores sin comillas. Casi siempre acierta… y cuando falla, el error es silencioso. PyYAML (la librería más usada en Python) sigue la especificación **YAML 1.1**, que adivina todavía más.

| Escribes | Esperas | PyYAML entiende | Solución |
|---|---|---|---|
| `python: 3.10` | `"3.10"` | `3.1` (float) | `python: "3.10"` |
| `pais: NO` | `"NO"` (Noruega) | `False` | `pais: "NO"` |
| `activo: yes` / `on` / `off` | texto | `True` / `True` / `False` | Usa `true`/`false` o comillas |
| `puerto: 22:22` | `"22:22"` | `1342` (¡base 60!) | `puerto: "22:22"` |
| `permisos: 0755` | `"0755"` | `493` (octal) | `permisos: "0755"` |
| `cp: 01234` | `"01234"` | `668` (octal, se pierde el 0) | `cp: "01234"` |
| `fecha: 2025-12-31` | texto | `datetime.date(2025, 12, 31)` | Bien si quieres fecha; si no, comillas |
| `version: 1e3` | texto | `"1e3"` en PyYAML, `1000.0` en otros | Comillas |
| `clave: ` (vacío) | `""` | `None` | `clave: ""` |
| `sku: P-001 # nota` | `"P-001 # nota"` | `"P-001"` (el resto es comentario) | Comillas si hay `#` |

El famoso caso de **GitHub Actions**: la clave `on:` de un workflow se lee con PyYAML como el booleano `True`.

```python
import yaml
wf = yaml.safe_load(open("deploy.yml", encoding="utf-8"))
wf.keys()            # dict_keys(['name', True, 'permissions', 'concurrency', 'jobs'])
wf[True]             # {'push': {'branches': ['main']}, 'workflow_dispatch': None}
```

GitHub usa su propio lector (YAML 1.2) y no tiene el problema, pero si procesas workflows desde Python, tenlo en cuenta.

**Otros errores clásicos:**

```yaml
# ❌ Tabulador en lugar de espacios → "found character '\t' that cannot start any token"
servicios:
	mongo: ...

# ❌ Indentación distinta en el mismo nivel → "mapping values are not allowed here"
proveedor:
  nombre: JBL
   pais: Estados Unidos

# ❌ Falta el espacio tras los dos puntos → se lee como el texto "nombre:JBL"
nombre:JBL

# ❌ Mezclar lista y mapa al mismo nivel
dependencias:
  - pandas
  version: 2.1

# ❌ Clave duplicada: PyYAML se queda con la última SIN avisar (yamllint sí avisa)
puerto: 27017
puerto: 27018
```

> 🧭 **Regla práctica:** pon entre comillas cualquier valor que deba ser **texto** y empiece por un número, contenga `:` o `#`, o sea `yes/no/on/off/y/n`. Y pasa `yamllint` (sección 16).

---

## 12. YAML en Python: PyYAML

### 12.1 Leer

```python
import yaml

with open("environment.yml", encoding="utf-8") as f:
    entorno = yaml.safe_load(f)

entorno["name"]                    # 'bigdata'
entorno["channels"]                # ['conda-forge']
entorno["dependencies"][:3]        # ['python=3.11', 'pandas>=2.1', 'numpy']

# Las dependencias de pip son un dict dentro de la lista
pip = next(d["pip"] for d in entorno["dependencies"] if isinstance(d, dict))
pip                                # ['streamlit>=1.46', 'mongomock']

# Desde una cadena
yaml.safe_load("a: 1\nb: [x, y]")  # {'a': 1, 'b': ['x', 'y']}
```

> ⚠️ Usa **siempre `yaml.safe_load`**. `yaml.load(f)` sin `Loader` da error en PyYAML 6, y `yaml.load(f, Loader=yaml.Loader)` o `yaml.unsafe_load` pueden **ejecutar código** escondido en el fichero (sección 17).

Fíjate en que los **comentarios desaparecen** al leer: el resultado es un diccionario normal de Python.

### 12.2 Escribir

```python
config = {
    "proyecto": "Ventas, stock y web",
    "periodo": {"inicio": "2023-01-01", "fin": "2024-12-31"},
    "tecnologias": ["eolica", "solar_fotovoltaica", "hidraulica"],
    "python": "3.11",
}

with open("config.yaml", "w", encoding="utf-8") as f:
    yaml.safe_dump(config, f, allow_unicode=True, sort_keys=False, default_flow_style=False)
```

Resultado:

```yaml
proyecto: Ventas, stock y web
periodo:
  inicio: '2023-01-01'
  fin: '2024-12-31'
tecnologias:
- eolica
- solar_fotovoltaica
- hidraulica
python: '3.11'
```

| Parámetro | Efecto |
|---|---|
| `allow_unicode=True` | Escribe `ñ` y tildes tal cual (si no, `"\xF1"`) |
| `sort_keys=False` | Respeta el orden del diccionario (por defecto **ordena** alfabéticamente) |
| `default_flow_style=False` | Estilo en bloque (una clave por línea) |
| `indent=2` | Indentación |
| `width=1000` | Evita que parta líneas largas |

PyYAML añade comillas automáticamente donde hacen falta (`'2023-01-01'`, `'3.11'`): así el fichero se vuelve a leer igual. Las listas salen sin sangrar (`- eolica` al nivel de la clave); es válido, aunque mucha gente prefiere sangrarlas.

### 12.3 Conservar comentarios: `ruamel.yaml`

Si un programa debe **modificar** un YAML escrito por personas (por ejemplo, subir la versión de una dependencia en `environment.yml`), PyYAML perdería los comentarios. `ruamel.yaml` los conserva:

```python
from ruamel.yaml import YAML

ry = YAML()                  # modo "round-trip" por defecto, sigue YAML 1.2
ry.preserve_quotes = True
ry.indent(mapping=2, sequence=4, offset=2)

with open("environment.yml", encoding="utf-8") as f:
    env = ry.load(f)

env["dependencies"][0] = "python=3.12"
with open("environment.yml", "w", encoding="utf-8") as f:
    ry.dump(env, f)          # comentarios y orden intactos
```

---

## 13. Los YAML del módulo

### 13.1 `environment.yml` (conda)

Describe un entorno reproducible: nombre, canales y paquetes (con una sublista para `pip`).

```yaml
# conda env create -f environment.yml   &&   conda activate bigdata
name: bigdata
channels:
  - conda-forge
dependencies:
  - python=3.11
  - pandas>=2.1
  - pyarrow
  - pyyaml
  - pip
  - pip:
      - streamlit>=1.46
```

```bash
conda env create -f environment.yml          # crear
conda env update -f environment.yml --prune  # actualizar (y quitar lo que sobra)
conda env export --from-history > environment.yml   # generar solo con lo que instalaste tú
```

> 💡 Detalle de sintaxis: `- pip:` es un elemento de la lista `dependencies` que a su vez es un **mapa** cuya única clave (`pip`) contiene **otra lista**. Por eso la sublista va más indentada.

### 13.2 `docker-compose.yml`

Describe servicios (contenedores), puertos, variables y volúmenes. Este levanta MongoDB y su interfaz web:

```yaml
services:
  mongo:
    image: mongo:7
    container_name: mp5074-mongo
    restart: unless-stopped
    ports:
      - "27017:27017"            # entre comillas: evita la trampa de base 60
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: admin
    volumes:
      - mongo_datos:/data/db

  mongo-express:
    image: mongo-express:1.0.2
    ports:
      - "8081:8081"
    environment:
      ME_CONFIG_MONGODB_URL: mongodb://admin:admin@mongo:27017/
    depends_on:
      - mongo

volumes:
  mongo_datos:                   # clave sin valor → None: volumen con opciones por defecto
```

```bash
docker compose config        # valida el fichero y muestra cómo lo ha entendido (¡muy útil!)
docker compose up -d
```

- La clave `version:` al principio está **obsoleta**: Compose la ignora y avisa. No la pongas.
- Las variables se pueden sacar a un fichero `.env` y usar como `${MONGO_PASSWORD}`.

### 13.3 GitHub Actions (`.github/workflows/*.yml`)

El `deploy.yml` del sitio del módulo publica la web en cada `push`:

```yaml
name: Publicar en GitHub Pages
on:                                # cuándo se ejecuta
  push:
    branches: [main]
  workflow_dispatch:               # botón manual en la pestaña Actions
permissions:
  contents: read
  pages: write
  id-token: write
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"   # ¡entre comillas! (3.10 sería 3.1)
      - run: pip install -r requirements.txt
      - run: mkdocs build
  deploy:
    needs: build                   # se ejecuta después de build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}   # expresión de GitHub, no de YAML
    steps:
      - id: deployment
        uses: actions/deploy-pages@v4
```

### 13.4 `mkdocs.yml`

Configura el sitio de documentación (tema, navegación, extensiones). La parte más delicada es `nav`, una lista de mapas:

```yaml
nav:
  - Inicio: index.md
  - Flujo de datos: flujo.md
  - UD1:
      - ud1/index.md
      - Retos: ud1/retos.md
```

```bash
mkdocs build --strict    # convierte los avisos (enlaces rotos, páginas fuera de nav) en errores
```

---

## 14. Un fichero de configuración para tu pipeline

Es habitual empezar con la configuración de un pipeline escrita como constantes en un `config.py`. Pasarla a **YAML** tiene ventajas: la puede editar alguien que no programa, no se mezcla con código y se puede validar con un esquema.

**`config.yaml`:**

```yaml
# Configuración del pipeline de generación eléctrica
periodo:
  inicio: "2023-01-01"
  fin: "2024-12-31"          # mínimo 2 años completos

rutas:
  raw: data/raw
  processed: data/processed

ree:
  url: https://apidatos.ree.es/es/datos/generacion/estructura-generacion
  geo:
    geo_trunc: electric_system
    geo_limit: peninsular
    geo_ids: "8741"           # texto: la API lo espera así
  max_dias_por_peticion: 366

tecnologias_renovables:
  - hidraulica
  - eolica
  - solar_fotovoltaica
  - solar_termica

mongo:
  uri: mongodb://localhost:27017/
  base_datos: mp5074
  # la contraseña NO va aquí: se lee de la variable de entorno MONGO_PASSWORD
```

**`src/config.py`** (lee el YAML, valida y permite sobrescribir con variables de entorno):

```python
import os
from pathlib import Path
import yaml
from jsonschema import validate

BASE = Path(__file__).resolve().parent.parent

ESQUEMA = {
    "type": "object",
    "required": ["periodo", "rutas", "ree"],
    "properties": {
        "periodo": {
            "type": "object",
            "required": ["inicio", "fin"],
            "properties": {"inicio": {"type": "string", "format": "date"},
                           "fin": {"type": "string", "format": "date"}},
        },
        "ree": {"type": "object", "required": ["url"],
                "properties": {"max_dias_por_peticion": {"type": "integer", "maximum": 366}}},
    },
}

def cargar_config(ruta: Path = BASE / "config.yaml") -> dict:
    with open(ruta, encoding="utf-8") as f:
        cfg = yaml.safe_load(f)
    validate(cfg, ESQUEMA)                       # para el pipeline si el fichero está mal
    # Variables de entorno con prioridad sobre el fichero
    cfg["periodo"]["inicio"] = os.getenv("MP5074_INICIO", cfg["periodo"]["inicio"])
    cfg["mongo"]["password"] = os.getenv("MONGO_PASSWORD")
    # Rutas relativas → absolutas
    cfg["rutas"] = {k: BASE / v for k, v in cfg["rutas"].items()}
    return cfg

CFG = cargar_config()
```

Uso en cualquier script:

```python
from config import CFG
CFG["periodo"]["inicio"]          # '2023-01-01'
CFG["rutas"]["raw"]               # PosixPath('/…/data/raw')
```

> 💡 **Orden de prioridad habitual:** valores por defecto en el código < fichero `config.yaml` < variables de entorno < argumentos de línea de comandos. Los **secretos** (contraseñas, tokens) nunca en el YAML versionado: en variables de entorno o en un `.env` incluido en `.gitignore`.

---

## 15. Convertir entre JSON, YAML y otros formatos

### 15.1 JSON ↔ YAML con Python

```python
import json, yaml

# JSON → YAML
with open("datos/catalogo.json", encoding="utf-8") as f:
    datos = json.load(f)
with open("catalogo.yaml", "w", encoding="utf-8") as f:
    yaml.safe_dump(datos, f, allow_unicode=True, sort_keys=False)

# YAML → JSON
with open("docker-compose.yml", encoding="utf-8") as f:
    datos = yaml.safe_load(f)
print(json.dumps(datos, indent=2, ensure_ascii=False, default=str))   # default=str por si hay fechas
```

En una línea desde la terminal:

```bash
python -c "import sys,yaml,json; print(json.dumps(yaml.safe_load(sys.stdin), indent=2, ensure_ascii=False, default=str))" < docker-compose.yml
```

También existe **`yq`** (como `jq` pero para YAML). Ojo: hay **dos programas** con ese nombre y sintaxis distinta (el de Mike Farah, en Go, y un envoltorio de `jq` en Python). Con el de Mike Farah (`winget install MikeFarah.yq` en Windows):

```bash
yq '.services.mongo.image' docker-compose.yml        # mongo:7
yq -o=json docker-compose.yml                         # YAML → JSON
yq -P datos/catalogo.json                             # JSON → YAML
```

### 15.2 ¿Y los demás formatos?

| Formato | Estructura | Comentarios | Uso típico en el módulo |
|---|---|---|---|
| **CSV** | Tabla plana | No | Datos tabulares (`ventas.csv`) |
| **JSON** | Árbol | No | APIs, MongoDB, intercambio |
| **JSON Lines** | Registros de árbol | No | Logs, eventos, streaming |
| **YAML** | Árbol | Sí | Configuración |
| **TOML** | Árbol (secciones) | Sí | `pyproject.toml`, `.streamlit/config.toml` |
| **XML** | Árbol con atributos | Sí | Datos públicos antiguos, RSS, SOAP |
| **Parquet** | Tabla columnar binaria | — | Datos procesados para análisis |
| **GeoJSON** | JSON con geometrías | No | Mapas (UD de visualización) |

```python
import tomllib                                  # Python ≥ 3.11, solo lectura
with open("pyproject.toml", "rb") as f:         # ¡en modo binario!
    cfg = tomllib.load(f)
```

---

## 16. Validar y revisar: linters y editor

### 16.1 `yamllint`

Detecta errores de sintaxis **y** problemas que PyYAML deja pasar (claves duplicadas, `yes/no`, indentación inconsistente, espacios al final…).

```bash
yamllint docker-compose.yml environment.yml
# docker-compose.yml
#   3:1   warning  missing document start "---"  (document-start)
#   12:7  error    duplication of key "ports" in mapping  (key-duplicates)
#   20:12 warning  truthy value should be one of [false, true]  (truthy)
```

Configuración del repositorio en `.yamllint.yaml` (relaja reglas demasiado estrictas):

```yaml
extends: default
rules:
  document-start: disable          # no exigir "---" al principio
  line-length:
    max: 120
  truthy:
    allowed-values: ["true", "false"]
    check-keys: false              # permite "on:" de GitHub Actions
```

### 16.2 Comprobar JSON rápidamente

```bash
python -m json.tool fichero.json > /dev/null && echo "JSON válido"
jq empty fichero.json && echo "JSON válido"           # empty: no imprime nada si está bien
```

### 16.3 VS Code

- **Format Document** (`Shift+Alt+F`) formatea JSON y YAML.
- La barra inferior muestra si el fichero usa **espacios o tabuladores**: para YAML, `Spaces: 2`.
- Con la extensión YAML de Red Hat, los ficheros conocidos (`docker-compose.yml`, workflows de GitHub, `mkdocs.yml`) se validan contra su esquema de [SchemaStore](https://www.schemastore.org/) y tienes autocompletado.
- Para tus propios ficheros, asocia un esquema con un comentario en la primera línea:

```yaml
# yaml-language-server: $schema=./esquemas/config.schema.json
periodo:
  inicio: "2023-01-01"
```

### 16.4 Automatizarlo: `pre-commit`

Con [pre-commit](https://pre-commit.com/) los ficheros se revisan antes de cada `git commit`. Su configuración es, cómo no, un YAML (`.pre-commit-config.yaml`):

```yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v5.0.0
    hooks:
      - id: check-json
      - id: check-yaml
      - id: end-of-file-fixer
  - repo: https://github.com/adrienverge/yamllint
    rev: v1.35.1
    hooks:
      - id: yamllint
```

```bash
pip install pre-commit && pre-commit install      # una vez por repositorio
pre-commit run --all-files                        # revisar todo ahora
```

---

## 17. Seguridad

- **Nunca `yaml.load` con `Loader=yaml.Loader` ni `yaml.unsafe_load` sobre ficheros que no controlas.** YAML puede incluir etiquetas que construyen objetos de Python y ejecutan código:

  ```yaml
  !!python/object/apply:os.system ["echo te he ejecutado un comando"]
  ```

  `yaml.safe_load` rechaza esto con `ConstructorError`. Usa siempre `safe_load` / `safe_dump`.

- **JSON es seguro de leer** con `json.loads` (solo crea diccionarios, listas y valores simples). Lo que **nunca** se hace es `eval(texto_json)`.
- **Secretos fuera de los ficheros versionados.** Contraseñas, tokens de API o claves no van en `config.yaml`, `docker-compose.yml` ni en notebooks: usa variables de entorno, un `.env` en el `.gitignore` o los *secrets* de GitHub Actions (`${{ secrets.TOKEN }}`). Las credenciales `admin/admin` del `docker-compose.yml` de la sección 13.2 son **solo para clase**.
- **Respuestas de API enormes:** si una API puede devolver muchos datos, pide paginado o usa JSON Lines y lee en streaming; `r.json()` carga todo en memoria.
- **Datos de terceros:** valida con JSON Schema antes de procesar (sección 9). Un campo que cambia de tipo sin avisar rompe pipelines más que cualquier error de código.

---

## 18. Solución de problemas frecuentes

| Error / síntoma | Causa probable | Solución |
|---|---|---|
| `JSONDecodeError: Expecting property name enclosed in double quotes` / `Illegal trailing comma` | Comillas simples o coma final | Comillas dobles; quita la última coma |
| `JSONDecodeError: Expecting value: line 1 column 1 (char 0)` | El texto está vacío o no es JSON (p. ej., una página HTML de error de la API) | Imprime `r.status_code` y `r.text[:300]` antes de `r.json()` |
| `JSONDecodeError: Extra data: line 2 column 1` | Es JSON Lines, no un único JSON | `lines=True` en pandas o leer línea a línea |
| `TypeError: Object of type datetime is not JSON serializable` | Tipos que JSON no conoce | `default=str` o una función `default` (sección 4.4) |
| `TypeError: Object of type int64 is not JSON serializable` | Valores de NumPy/pandas | `.item()`, `.tolist()` o `df.to_json()` |
| Tildes como `\u00f1` o `Ã±` | Falta `ensure_ascii=False` / el fichero no se abrió como UTF-8 | `ensure_ascii=False` al escribir y `encoding="utf-8"` al abrir |
| `UnicodeDecodeError` en Windows | Lectura con la codificación por defecto (cp1252) | `open(..., encoding="utf-8")` |
| `FutureWarning: Passing literal json to 'read_json' is deprecated` | Cadena JSON pasada directamente | `pd.read_json(StringIO(texto))` |
| Columnas con diccionarios tras `read_json` | JSON anidado | `pd.json_normalize` (sección 6) |
| `KeyError` en `json_normalize(meta=...)` | Falta el campo en algún registro | `errors="ignore"` |
| `ScannerError: mapping values are not allowed here` | Indentación incorrecta o `: ` dentro de un texto sin comillas | Revisa la indentación; pon comillas al texto |
| `found character '\t' that cannot start any token` | Tabulador en un YAML | Sustituye por espacios (en VS Code: *Convert Indentation to Spaces*) |
| `ParserError: expected <block end>, but found '-'` | Lista y mapa mezclados o mal alineados | Alinea los `-` con la misma indentación |
| `TypeError: load() missing 1 required positional argument: 'Loader'` | `yaml.load(f)` en PyYAML ≥ 6 | `yaml.safe_load(f)` |
| `ConstructorError: could not determine a constructor for the tag '!Ref'` | Etiquetas propias de otra herramienta (CloudFormation…) | Usa la herramienta correspondiente o un `Loader` con esas etiquetas |
| Python `3.10` se convierte en `3.1`, `NO` en `False`, un puerto en un número raro | Tipos adivinados por YAML | Comillas (sección 11) |
| Una clave de YAML "desaparece" | Clave duplicada: gana la última | `yamllint` |
| `conda env create` falla con `pip` | `- pip:` mal indentado o falta `- pip` como dependencia | Mira la sección 13.1 |
| `docker compose` dice que `version` está obsoleto | Clave antigua | Bórrala |

---

## 19. Chuleta rápida

```python
# ---------- JSON ----------
import json
with open("f.json", encoding="utf-8") as f: d = json.load(f)
json.loads(texto)
with open("f.json", "w", encoding="utf-8") as f:
    json.dump(d, f, ensure_ascii=False, indent=2, default=str)
json.dumps(d, ensure_ascii=False)
try: json.loads(t)
except json.JSONDecodeError as e: print(e.lineno, e.colno, e.msg)

# ---------- JSON Lines ----------
import gzip
with gzip.open("f.jsonl.gz", "rt", encoding="utf-8") as f:
    for linea in f: r = json.loads(linea)
f.write(json.dumps(r, ensure_ascii=False) + "\n")

# ---------- pandas ----------
import pandas as pd
from io import StringIO
pd.read_json("f.json"); pd.read_json(StringIO(texto))
pd.read_json("f.jsonl.gz", lines=True, chunksize=50_000)
pd.json_normalize(d["productos"], sep="_")
pd.json_normalize(d["included"], record_path=["attributes", "values"], meta=["type"])
df.explode("lista")
df.to_json("f.json", orient="records", force_ascii=False, indent=2, date_format="iso")
df.to_json("f.jsonl", orient="records", lines=True, force_ascii=False)

# ---------- APIs ----------
r = requests.get(url, params=p, timeout=30); r.raise_for_status(); d = r.json()
requests.post(url, json=payload, timeout=10)

# ---------- JSON Schema ----------
from jsonschema import validate, Draft202012Validator
validate(d, esquema)
for e in Draft202012Validator(esquema).iter_errors(d): print(list(e.path), e.message)

# ---------- YAML ----------
import yaml
with open("f.yaml", encoding="utf-8") as f: cfg = yaml.safe_load(f)
list(yaml.safe_load_all(f))                      # varios documentos
with open("f.yaml", "w", encoding="utf-8") as f:
    yaml.safe_dump(cfg, f, allow_unicode=True, sort_keys=False)
```

```yaml
# ---------- Sintaxis YAML ----------
clave: valor              # espacio tras ":"
numero: 42
texto: "3.10"             # comillas si debe ser texto y parece otra cosa
booleano: true            # no uses yes/no/on/off
nulo: null
lista: [a, b]
lista_bloque:
  - a
  - b
mapa: {x: 1, y: 2}
multilinea: |
  línea 1
  línea 2
plegado: >
  se une en
  una sola línea
base: &ancla {host: localhost}
uso:
  <<: *ancla
  puerto: 27017
```

```bash
# ---------- Terminal ----------
python -m json.tool f.json
jq '.productos[] | {sku, pvp: .precios.pvp}' f.json
jq -r '.productos[].nombre' f.json
zcat f.jsonl.gz | jq -c 'select(.evento=="compra")'
yamllint f.yaml
check-jsonschema --schemafile esquema.json f.json
docker compose config
yq -o=json f.yaml
```

---

## 20. Referencias

- Especificación JSON (RFC 8259): <https://www.rfc-editor.org/rfc/rfc8259>
- Introducción a JSON en español: <https://www.json.org/json-es.html>
- Módulo `json` de Python: <https://docs.python.org/es/3/library/json.html>
- JSON Lines: <https://jsonlines.org/>
- `pandas.read_json`: <https://pandas.pydata.org/docs/reference/api/pandas.read_json.html>
- `pandas.json_normalize`: <https://pandas.pydata.org/docs/reference/api/pandas.json_normalize.html>
- Manual de `jq`: <https://jqlang.org/manual/> · Pruebas en línea: <https://play.jqlang.org/>
- JSON Schema (guía *Understanding JSON Schema*): <https://json-schema.org/understanding-json-schema>
- Librería `jsonschema`: <https://python-jsonschema.readthedocs.io/>
- `check-jsonschema`: <https://check-jsonschema.readthedocs.io/>
- Especificación YAML 1.2.2: <https://yaml.org/spec/1.2.2/>
- PyYAML: <https://pyyaml.org/wiki/PyYAMLDocumentation>
- `ruamel.yaml`: <https://yaml.dev/doc/ruamel.yaml/>
- `yamllint`: <https://yamllint.readthedocs.io/>
- `yq` (Mike Farah): <https://mikefarah.gitbook.io/yq/>
- *The YAML document from hell* (las trampas de YAML, con ejemplos): <https://ruudvanasseldonk.com/2023/01/11/the-yaml-document-from-hell>
- Ficheros `environment.yml` de conda: <https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html>
- Referencia de Docker Compose: <https://docs.docker.com/reference/compose-file/>
- Sintaxis de workflows de GitHub Actions: <https://docs.github.com/es/actions/reference/workflows-and-actions/workflow-syntax>
- MongoDB Extended JSON: <https://www.mongodb.com/docs/manual/reference/mongodb-extended-json/>
- SchemaStore (esquemas de ficheros de configuración conocidos): <https://www.schemastore.org/>
