# Manual de Python para notebooks y ficheros

**Módulo:** MP5074 – Sistemas de Big Data · Curso 2026-27

> Este manual recoge **el Python que hace falta** para trabajar con comodidad en los notebooks del módulo: tipos y estructuras de datos, control de flujo, funciones, errores y, sobre todo, **cómo localizar, leer y escribir ficheros** (texto, CSV, JSON, comprimidos, carpetas con muchos archivos). No pretende ser un curso completo de Python, sino la base sobre la que se apoyan pandas y PySpark. Ver también los manuales de [JupyterLab](../jupyterlab/manual_jupyterlab.md), [Miniconda](../miniconda/manual_miniconda.md), [pandas](../pandas/manual_pandas.md) y el apartado de [Conocimientos previos](../../conocimientos_previos/manual_conocimientos_previos.md) (rutas y sistema de ficheros).

---

## Índice

1. [Python en un notebook](#1-python-en-un-notebook)
2. [Variables y tipos básicos](#2-variables-y-tipos-básicos)
3. [Cadenas de texto](#3-cadenas-de-texto)
4. [Colecciones: listas, tuplas, diccionarios y conjuntos](#4-colecciones-listas-tuplas-diccionarios-y-conjuntos)
5. [Control de flujo](#5-control-de-flujo)
6. [Comprensiones](#6-comprensiones)
7. [Funciones](#7-funciones)
8. [Errores y excepciones](#8-errores-y-excepciones)
9. [Módulos, paquetes e instalación](#9-módulos-paquetes-e-instalación)
10. [Rutas con `pathlib`](#10-rutas-con-pathlib)
11. [Leer y escribir ficheros de texto](#11-leer-y-escribir-ficheros-de-texto)
12. [Ficheros CSV](#12-ficheros-csv)
13. [Ficheros JSON y JSON Lines](#13-ficheros-json-y-json-lines)
14. [Ficheros comprimidos y descargas](#14-ficheros-comprimidos-y-descargas)
15. [Recorrer carpetas con muchos ficheros](#15-recorrer-carpetas-con-muchos-ficheros)
16. [Fechas y horas](#16-fechas-y-horas)
17. [Herramientas propias del notebook](#17-herramientas-propias-del-notebook)
18. [Buenas prácticas en notebooks](#18-buenas-prácticas-en-notebooks)
19. [Solución de problemas frecuentes](#19-solución-de-problemas-frecuentes)
20. [Chuleta rápida](#20-chuleta-rápida)
21. [Referencias](#21-referencias)
22. [Ejemplo resuelto: unificar una carpeta de ficheros](#22-ejemplo-resuelto-unificar-una-carpeta-de-ficheros)

---

## 1. Python en un notebook

Un notebook ejecuta Python **celda a celda** y conserva en memoria todo lo que se va definiendo (variables, funciones, *imports*). Lo que importa es el **orden en que se ejecutan** las celdas, no el orden en que aparecen.

```python
precio = 19.99
unidades = 3
precio * unidades          # La última expresión de la celda se muestra sola: 59.97
```

| En una celda | Qué se ve |
|---|---|
| La **última expresión** | Se muestra automáticamente (`Out[n]`) |
| `print(x)` | Muestra texto en cualquier punto de la celda |
| `display(df)` | Muestra objetos "bonitos" (tablas de pandas) en cualquier punto |
| Terminar la última línea con `;` | Oculta la salida automática (útil con gráficos) |

```python
total = precio * unidades
print("Total:", total)
print(f"Total con IVA: {total * 1.21:.2f} €")
```

> El número entre corchetes `[5]` indica el **orden de ejecución**. Si ves `[12]` encima de `[3]`, el notebook se ha ejecutado desordenado. Antes de entregar: **Kernel → Restart Kernel and Run All Cells** (ver [manual de JupyterLab](../jupyterlab/manual_jupyterlab.md), sección 6).

---

## 2. Variables y tipos básicos

### 2.1 Variables

Una variable es un **nombre** que apunta a un valor. No se declara el tipo: Python lo deduce.

```python
ciudad = "Ferrol"         # str
habitantes = 64_000       # int (el _ solo mejora la lectura)
temperatura = 14.5        # float
es_capital = False        # bool
dato_ausente = None       # NoneType: "no hay valor"
```

Convenciones: `snake_case` en minúsculas (`total_ventas`), constantes en mayúsculas (`RUTA_DATOS`). Los nombres distinguen mayúsculas (`df` ≠ `DF`).

### 2.2 Tipos básicos

| Tipo | Ejemplo | Comentario |
|---|---|---|
| `int` | `42`, `-7`, `1_000_000` | Enteros sin límite de tamaño |
| `float` | `3.14`, `1e6`, `float("nan")` | Decimales (coma flotante). Ojo: `0.1 + 0.2 == 0.30000000000000004` |
| `str` | `"hola"`, `'hola'`, `"""varias líneas"""` | Texto Unicode |
| `bool` | `True`, `False` | Resultado de comparaciones |
| `NoneType` | `None` | Ausencia de valor |

```python
type(temperatura)          # float
isinstance(habitantes, int)  # True
```

### 2.3 Conversiones

```python
int("42")          # 42
float("3,5".replace(",", "."))   # 3.5  ← los CSV españoles usan coma decimal
str(2026)          # "2026"
bool("")           # False (cadenas vacías, 0, None, listas vacías → False)
round(3.14159, 2)  # 3.14
```

> `int("3.5")` da error: convierte primero a `float`. `int("abc")` lanza `ValueError` (ver sección 8).

### 2.4 Operadores

| Tipo | Operadores |
|---|---|
| Aritméticos | `+  -  *  /` (división real) `//` (entera) `%` (resto) `**` (potencia) |
| Comparación | `==  !=  <  <=  >  >=` |
| Lógicos | `and  or  not` |
| Pertenencia | `in`, `not in` |
| Identidad | `is`, `is not` (para comparar con `None`: `x is None`) |
| Asignación compuesta | `+=  -=  *=  /=` |

```python
7 / 2      # 3.5
7 // 2     # 3
7 % 2      # 1
2 ** 10    # 1024
"a" in "casa"          # True
0 < temperatura < 30   # Comparaciones encadenadas: True
```

---

## 3. Cadenas de texto

### 3.1 *f-strings*: formatear resultados

```python
nombre = "Ana"
ventas = 1234567.891
ratio = 0.4567

f"Hola, {nombre}"            # 'Hola, Ana'
f"{ventas:.2f}"              # '1234567.89'      2 decimales
f"{ventas:,.2f}"             # '1,234,567.89'    separador de miles
f"{ratio:.1%}"               # '45.7%'           porcentaje
f"{42:05d}"                  # '00042'           relleno con ceros
f"{nombre:>10}|"             # '       Ana|'     alinear a la derecha
f"{ventas=}"                 # 'ventas=1234567.891'  depuración rápida
```

### 3.2 Métodos más usados

```python
s = "  Ventas_Galicia_2026.CSV  "

s.strip()                    # 'Ventas_Galicia_2026.CSV'  quita espacios de los extremos
s.strip().lower()            # 'ventas_galicia_2026.csv'
s.upper()                    # en mayúsculas
s.replace("_", " ")          # sustituye
s.strip().split("_")         # ['Ventas', 'Galicia', '2026.CSV']
"-".join(["2026", "10", "20"])  # '2026-10-20'
s.strip().startswith("Ventas")  # True
s.strip().lower().endswith(".csv")  # True
"galicia" in s.lower()       # True
s.count("a")                 # nº de apariciones
s.strip().zfill(30)          # rellena con ceros a la izquierda
"123".isdigit()              # True
```

### 3.3 Índices y *slicing*

Las cadenas (y las listas) se indexan desde **0**; los índices negativos cuentan desde el final.

```python
codigo = "ES-15-2026"
codigo[0]        # 'E'
codigo[-1]       # '6'
codigo[3:5]      # '15'      desde 3 hasta 5 (sin incluir)
codigo[:2]       # 'ES'
codigo[-4:]      # '2026'
codigo[::-1]     # '6202-51-SE'  al revés
```

### 3.4 Caracteres especiales y codificación

```python
"Línea 1\nLínea 2"     # \n salto de línea, \t tabulador
r"C:\datos\nuevo"      # cadena "raw": las \ no son especiales (rutas de Windows)
"ñ".encode("utf-8")    # b'\xc3\xb1'  str → bytes
b"\xc3\xb1".decode("utf-8")  # 'ñ'  bytes → str
```

---

## 4. Colecciones: listas, tuplas, diccionarios y conjuntos

| Tipo | Sintaxis | Ordenada | Modificable | Duplicados | Uso típico |
|---|---|---|---|---|---|
| `list` | `[1, 2, 3]` | Sí | Sí | Sí | Secuencia de elementos (ficheros, valores) |
| `tuple` | `(1, 2, 3)` | Sí | **No** | Sí | Registro fijo (lat, lon); devolver varios valores |
| `dict` | `{"a": 1}` | Sí (orden de inserción) | Sí | Claves no | Pares clave → valor (un registro JSON) |
| `set` | `{1, 2, 3}` | No | Sí | **No** | Valores únicos, pertenencia rápida |

### 4.1 Listas

```python
ficheros = ["enero.csv", "febrero.csv"]
ficheros.append("marzo.csv")         # añade al final
ficheros.extend(["abril.csv", "mayo.csv"])  # añade varios
ficheros.insert(0, "diciembre.csv")  # inserta en una posición
ficheros.remove("diciembre.csv")     # borra por valor
ultimo = ficheros.pop()              # saca el último ('mayo.csv')
len(ficheros)                        # 4
ficheros[0], ficheros[-1]            # primero y último
ficheros[1:3]                        # sublista
"marzo.csv" in ficheros              # True

numeros = [5, 2, 9, 1]
sorted(numeros)                      # [1, 2, 5, 9]  (nueva lista)
numeros.sort(reverse=True)           # ordena la propia lista: [9, 5, 2, 1]
sum(numeros), min(numeros), max(numeros)   # (17, 1, 9)
```

> ⚠️ `b = a` **no copia** la lista: ambas variables apuntan a la misma. Para copiar: `b = a.copy()` o `b = list(a)`.

### 4.2 Tuplas y desempaquetado

```python
coordenadas = (43.48, -8.23)
lat, lon = coordenadas               # desempaquetado
x, y = 1, 2
x, y = y, x                          # intercambiar valores: x=2, y=1
primero, *resto = [1, 2, 3, 4]       # primero=1, resto=[2, 3, 4]
```

### 4.3 Diccionarios

La estructura más importante para datos **semiestructurados** (un objeto JSON se lee como un `dict`).

```python
cliente = {"id": 101, "nombre": "Ana", "ciudad": "Ferrol"}

cliente["nombre"]                    # 'Ana'
cliente.get("email")                 # None (sin error si no existe)
cliente.get("email", "sin email")    # valor por defecto
cliente["email"] = "ana@ejemplo.com" # añade o modifica
del cliente["ciudad"]                # borra una clave
"id" in cliente                      # True (busca en las claves)

cliente.keys()                       # claves
cliente.values()                     # valores
cliente.items()                      # pares (clave, valor)

cliente.update({"edad": 34, "nombre": "Ana María"})   # fusiona otro diccionario
```

Diccionarios **anidados** (como en JSON):

```python
pedido = {
    "id": "P-001",
    "cliente": {"id": 101, "nombre": "Ana"},
    "lineas": [
        {"producto": "teclado", "precio": 25.0, "cantidad": 2},
        {"producto": "ratón", "precio": 12.5, "cantidad": 1},
    ],
}
pedido["cliente"]["nombre"]          # 'Ana'
pedido["lineas"][0]["producto"]      # 'teclado'
```

Contar apariciones (muy habitual):

```python
from collections import Counter

ciudades = ["Ferrol", "Vigo", "Ferrol", "Lugo", "Ferrol"]
Counter(ciudades).most_common(2)     # [('Ferrol', 3), ('Vigo', 1)]
```

### 4.4 Conjuntos

```python
a = {"Ferrol", "Vigo", "Lugo"}
b = {"Vigo", "Ourense"}
a | b      # unión
a & b      # intersección: {'Vigo'}
a - b      # diferencia: {'Ferrol', 'Lugo'}
set(["x", "y", "x"])   # quitar duplicados: {'x', 'y'}
```

---

## 5. Control de flujo

En Python los bloques se definen con **sangría de 4 espacios** y dos puntos `:` al final de la línea que abre el bloque.

### 5.1 Condicionales

```python
temperatura = 31

if temperatura >= 30:
    nivel = "alto"
elif temperatura >= 20:
    nivel = "medio"
else:
    nivel = "bajo"

# Expresión condicional en una línea
estado = "calor" if temperatura > 25 else "normal"
```

### 5.2 Bucle `for`

`for` recorre **cualquier iterable**: listas, cadenas, diccionarios, líneas de un fichero, ficheros de una carpeta…

```python
for f in ["enero.csv", "febrero.csv"]:
    print(f)

for i in range(3):                   # 0, 1, 2
    print(i)

for i, f in enumerate(["a.csv", "b.csv"], start=1):   # índice + valor
    print(i, f)

for mes, total in zip(["ene", "feb"], [100, 120]):     # dos listas en paralelo
    print(mes, total)

for clave, valor in cliente.items():                   # diccionario
    print(clave, "→", valor)
```

### 5.3 `while`, `break` y `continue`

```python
pagina = 1
while pagina <= 10:
    if pagina == 3:
        pagina += 1
        continue                     # salta a la siguiente iteración
    if pagina == 6:
        break                        # sale del bucle
    print("Descargando página", pagina)
    pagina += 1
```

> En análisis de datos **evita los bucles sobre filas de un DataFrame**: pandas tiene operaciones vectorizadas mucho más rápidas (ver [manual de pandas](../pandas/manual_pandas.md)). Los bucles sí son adecuados para recorrer **ficheros**, **páginas** de una API o **columnas**.

---

## 6. Comprensiones

Forma compacta de crear listas, diccionarios o conjuntos a partir de otro iterable.

```python
precios = [10, 25, 7, 40]

[p * 1.21 for p in precios]                  # aplicar IVA a todos
[p for p in precios if p > 10]               # filtrar: [25, 40]
["caro" if p > 20 else "barato" for p in precios]

{p: p * 1.21 for p in precios}               # diccionario {10: 12.1, ...}
{f.split(".")[-1] for f in ["a.csv", "b.json", "c.csv"]}   # conjunto {'csv', 'json'}
```

Equivale a:

```python
resultado = []
for p in precios:
    if p > 10:
        resultado.append(p)
```

> Si una comprensión ocupa más de una línea o tiene varios `if`, escríbela como bucle: se lee mejor.

---

## 7. Funciones

### 7.1 Definir y llamar

```python
def precio_con_iva(precio, iva=0.21):
    """Devuelve el precio con IVA redondeado a 2 decimales."""
    return round(precio * (1 + iva), 2)

precio_con_iva(100)            # 121.0
precio_con_iva(100, iva=0.10)  # 110.0  (argumento por nombre)
help(precio_con_iva)           # muestra el docstring
```

- `def` + nombre + parámetros + `:`.
- `return` devuelve el resultado; sin `return`, la función devuelve `None`.
- Los parámetros con valor por defecto van **al final**.
- La primera cadena (*docstring*) documenta la función; en Jupyter se ve con `Shift + Tab`.

### 7.2 Varios valores, anotaciones de tipo

```python
def resumen(valores: list[float]) -> tuple[float, float, float]:
    """Mínimo, máximo y media de una lista."""
    return min(valores), max(valores), sum(valores) / len(valores)

minimo, maximo, media = resumen([3, 7, 5])
```

Las anotaciones (`: list[float]`, `-> tuple`) **no** se comprueban al ejecutar; sirven de documentación y para el autocompletado.

### 7.3 `*args` y `**kwargs`

```python
def registrar(*mensajes, **opciones):
    print(mensajes)     # tupla con los argumentos posicionales
    print(opciones)     # diccionario con los argumentos por nombre

registrar("a", "b", nivel="info", hora="10:00")
```

Lo verás mucho en la documentación de pandas: `pd.read_csv(ruta, **kwargs)`.

### 7.4 Funciones `lambda`

Funciones anónimas de una sola expresión. Se usan como argumento de otras funciones:

```python
ficheros = ["b.csv", "A.csv", "c.csv"]
sorted(ficheros, key=lambda f: f.lower())       # ordenar sin distinguir mayúsculas

ventas = [("Ana", 300), ("Brais", 500), ("Carla", 200)]
sorted(ventas, key=lambda v: v[1], reverse=True)  # por importe

# En pandas: df["importe"].apply(lambda x: x * 1.21)
```

### 7.5 Ámbito de las variables

Las variables creadas dentro de una función son **locales**: desaparecen al terminar. Una función puede **leer** variables globales del notebook, pero es mejor pasarle todo lo que necesite como parámetro: así se puede reutilizar y probar.

---

## 8. Errores y excepciones

### 8.1 Leer un *traceback*

Cuando algo falla, Python muestra un **traceback**. Se lee **de abajo arriba**: la última línea dice el tipo de error y el mensaje; las de encima, dónde ocurrió.

```
---------------------------------------------------------------------------
FileNotFoundError                         Traceback (most recent call last)
Cell In[3], line 1
----> 1 open("datos/ventas.csv")

FileNotFoundError: [Errno 2] No such file or directory: 'datos/ventas.csv'
```

### 8.2 Errores más frecuentes

| Error | Significado | Ejemplo típico |
|---|---|---|
| `NameError` | Variable o función no definida | No has ejecutado la celda que la crea, o está mal escrita |
| `SyntaxError` / `IndentationError` | Código mal escrito | Falta `:`, paréntesis o la sangría no cuadra |
| `TypeError` | Operación con un tipo incorrecto | `"Total: " + 5` (usa f-string) |
| `ValueError` | Tipo correcto, valor inválido | `int("3,5")` |
| `KeyError` | Clave inexistente en un `dict` o columna inexistente en pandas | `cliente["emial"]` |
| `IndexError` | Posición fuera de rango | `lista[10]` con 3 elementos |
| `AttributeError` | El objeto no tiene ese método | `None.split()`; una variable que creías `str` es `None` |
| `FileNotFoundError` | No existe el fichero en esa ruta | Ruta relativa mal calculada (sección 10) |
| `PermissionError` | Sin permiso o fichero abierto en otro programa | El CSV está abierto en Excel (Windows) |
| `UnicodeDecodeError` | Codificación incorrecta al leer | Fichero en Latin-1 leído como UTF-8 (sección 11.3) |
| `ModuleNotFoundError` | Librería no instalada **en el kernel actual** | Sección 9.3 |
| `JSONDecodeError` | JSON mal formado | Fichero JSON Lines leído con `json.load` (sección 13) |

### 8.3 Capturar errores: `try` / `except`

Útil para que un fichero defectuoso **no detenga** el procesamiento de cientos:

```python
def a_numero(texto):
    """Convierte '1.234,5' o '1234.5' a float; devuelve None si no se puede."""
    try:
        return float(texto.replace(".", "").replace(",", ".")) if "," in texto else float(texto)
    except (ValueError, AttributeError):
        return None

a_numero("1.234,5")   # 1234.5
a_numero("n/d")       # None
```

Estructura completa:

```python
try:
    f = open("datos/ventas.csv", encoding="utf-8")
except FileNotFoundError as e:
    print("No encuentro el fichero:", e)
else:
    print("Leído correctamente")       # solo si NO hubo error
    f.close()
finally:
    print("Esto se ejecuta siempre")
```

> No escribas `except:` a secas ni `except Exception: pass`: oculta errores reales y luego no sabrás por qué faltan datos. Captura **el error concreto** y, como mínimo, regístralo.

### 8.4 Lanzar errores y comprobaciones

```python
def leer_mes(mes):
    if not 1 <= mes <= 12:
        raise ValueError(f"Mes no válido: {mes}")
    ...

assert len(ficheros) > 0, "La carpeta de datos está vacía"   # comprobación rápida
```

---

## 9. Módulos, paquetes e instalación

### 9.1 Importar

```python
import math                          # módulo completo
math.sqrt(16)

import pandas as pd                  # con alias (convención)
from pathlib import Path             # solo un nombre
from datetime import datetime, date
```

Alias **convencionales** en ciencia de datos (úsalos siempre, toda la documentación los usa):

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
```

> Nunca uses `from pandas import *`: no se sabe de dónde viene cada nombre y puede sobrescribir otros.

### 9.2 Biblioteca estándar útil para datos

| Módulo | Para qué |
|---|---|
| `pathlib` | Rutas y ficheros (sección 10) |
| `os`, `shutil` | Variables de entorno, copiar / mover / borrar ficheros y carpetas |
| `csv`, `json` | Formatos CSV y JSON (secciones 12 y 13) |
| `gzip`, `zipfile`, `tarfile` | Comprimidos (sección 14) |
| `datetime` | Fechas (sección 16) |
| `collections` | `Counter`, `defaultdict` |
| `re` | Expresiones regulares |
| `urllib.request` | Descargar ficheros |
| `time` | Pausas (`time.sleep`) y medir tiempos |
| `sqlite3` | Base de datos SQL en un fichero |
| `logging` | Registro de mensajes en procesos largos |

### 9.3 Instalar paquetes en el entorno del notebook

El paquete debe instalarse **en el mismo entorno que usa el kernel**. Lo recomendado en el módulo es instalar desde la terminal con conda (ver [manual de Miniconda](../miniconda/manual_miniconda.md)):

```bash
conda activate bigdata
conda install -c conda-forge openpyxl requests
```

Desde una celda del notebook, usa los comandos mágicos, que **siempre** instalan en el kernel activo:

```python
%pip install openpyxl
%conda install -c conda-forge openpyxl
```

> Evita `!pip install ...` (con `!`): puede instalar en otro Python distinto del del kernel. Después de instalar, a veces hace falta **reiniciar el kernel**.

Comprobar qué Python y qué versión de una librería estás usando:

```python
import sys, pandas as pd
sys.executable          # ruta del Python del kernel (debe contener envs/bigdata)
pd.__version__
```

---

## 10. Rutas con `pathlib`

Casi todos los problemas al leer ficheros son **problemas de rutas**. Repasa primero [Conocimientos previos](../../conocimientos_previos/manual_conocimientos_previos.md) (rutas absolutas y relativas).

### 10.1 Lo esencial

```python
from pathlib import Path

Path.cwd()                         # carpeta de trabajo (en Jupyter: la del notebook)
Path.home()                        # carpeta personal (~)

DATOS = Path("..") / "datos"       # el operador / une partes de la ruta
ruta = DATOS / "ventas_2026.csv"

ruta.exists()                      # ¿existe?
ruta.is_file(), ruta.is_dir()
ruta.resolve()                     # ruta absoluta
```

> `pathlib` construye la ruta con el separador correcto en cada sistema (`/` en Linux, `\` en Windows). **No escribas rutas absolutas** del tipo `C:\Users\Ana\...` en un notebook que vayas a entregar: en otro equipo no existirá.

### 10.2 Partes de una ruta

```python
ruta = Path("datos/brutos/ventas_2026-10.csv.gz")

ruta.name        # 'ventas_2026-10.csv.gz'
ruta.stem        # 'ventas_2026-10.csv'
ruta.suffix      # '.gz'
ruta.suffixes    # ['.csv', '.gz']
ruta.parent      # Path('datos/brutos')
ruta.parts       # ('datos', 'brutos', 'ventas_2026-10.csv.gz')
ruta.with_suffix(".parquet")     # cambia la extensión
ruta.parent / "limpios" / ruta.name   # mismo fichero en otra carpeta
```

### 10.3 Crear, listar, mover y borrar

```python
salida = Path("resultados/2026")
salida.mkdir(parents=True, exist_ok=True)     # crea la carpeta (y las intermedias) si no existe

list(Path("datos").iterdir())                 # contenido de una carpeta
sorted(Path("datos").glob("*.csv"))           # CSV de esa carpeta
sorted(Path("datos").rglob("*.json"))         # JSON en esa carpeta y subcarpetas

Path("datos/ventas.csv").stat().st_size / 1024**2   # tamaño en MB
Path("tmp.txt").rename("tmp_viejo.txt")       # renombrar / mover
Path("tmp_viejo.txt").unlink(missing_ok=True) # borrar un fichero

import shutil
shutil.copy("datos/a.csv", "copia/a.csv")     # copiar
shutil.rmtree("tmp/")                         # ⚠️ borrar una carpeta con todo su contenido
```

### 10.4 Patrones de `glob`

| Patrón | Coincide con |
|---|---|
| `*.csv` | Todos los `.csv` de la carpeta |
| `ventas_2026-*.csv` | `ventas_2026-01.csv`, `ventas_2026-02.csv`… |
| `ventas_??.csv` | `ventas_01.csv` (cada `?` = un carácter) |
| `ventas_[0-9]*.csv` | Que empiecen por dígito tras `ventas_` |
| `**/*.parquet` | Todos los `.parquet` a cualquier profundidad (con `glob`) |

### 10.5 Ruta base fija para todo el notebook

Patrón recomendado: definir las rutas **una vez**, en la primera celda:

```python
from pathlib import Path

BASE = Path.cwd().parent           # raíz del proyecto (si el notebook está en notebooks/)
DATOS = BASE / "datos"
SALIDA = BASE / "resultados"
SALIDA.mkdir(exist_ok=True)

print("Datos en:", DATOS.resolve(), "| existe:", DATOS.exists())
```

---

## 11. Leer y escribir ficheros de texto

### 11.1 `with open(...)`

`with` **cierra el fichero automáticamente** aunque haya un error. Indica siempre la codificación.

```python
with open("datos/notas.txt", encoding="utf-8") as f:
    contenido = f.read()             # todo el fichero en una cadena

with open("datos/notas.txt", encoding="utf-8") as f:
    lineas = f.read().splitlines()   # lista de líneas sin '\n'

with open("datos/registro.log", encoding="utf-8") as f:
    for linea in f:                  # línea a línea: NO carga el fichero entero en memoria ✅
        if "ERROR" in linea:
            print(linea.rstrip())
```

> Para ficheros **grandes** (logs de GB), recorre **línea a línea**: `f.read()` intentaría cargarlo todo en memoria.

### 11.2 Modos de apertura

| Modo | Significado |
|---|---|
| `"r"` | Leer (por defecto). Error si no existe |
| `"w"` | Escribir. ⚠️ **Borra** el contenido previo; crea el fichero si no existe |
| `"a"` | Añadir al final |
| `"x"` | Crear; error si ya existe |
| `"rb"`, `"wb"` | Leer / escribir en **binario** (imágenes, comprimidos, Parquet) |

```python
with open("resultados/resumen.txt", "w", encoding="utf-8") as f:
    f.write("Informe de ventas\n")
    f.write(f"Total: {1234.5:.2f} €\n")
    print("Línea escrita con print", file=f)

with open("resultados/resumen.txt", "a", encoding="utf-8") as f:
    f.write("Línea añadida\n")
```

Atajos de `pathlib` para ficheros pequeños:

```python
from pathlib import Path
texto = Path("datos/notas.txt").read_text(encoding="utf-8")
Path("resultados/copia.txt").write_text(texto, encoding="utf-8")
```

### 11.3 Codificaciones

| Codificación | Dónde aparece |
|---|---|
| `utf-8` | Estándar actual: Linux, web, APIs. **Úsala por defecto** |
| `utf-8-sig` | UTF-8 con BOM: CSV exportados por Excel "CSV UTF-8". Evita que la primera columna aparezca como `\ufeffid` |
| `latin-1` / `cp1252` | Ficheros antiguos de Windows y muchos portales de datos españoles |

Si al leer aparece `UnicodeDecodeError` o caracteres como `Ã±` en lugar de `ñ`, prueba otra codificación:

```python
for cod in ["utf-8", "utf-8-sig", "cp1252", "latin-1"]:
    try:
        texto = Path("datos/municipios.csv").read_text(encoding=cod)
        print("Codificación válida:", cod)
        break
    except UnicodeDecodeError:
        pass
```

> En Windows, si no indicas `encoding=`, Python puede usar `cp1252` y fallar con ficheros UTF-8. **Pon siempre `encoding="utf-8"`**.

---

## 12. Ficheros CSV

Para análisis **usarás pandas** (`pd.read_csv`, ver [manual de pandas](../pandas/manual_pandas.md), sección 4). El módulo `csv` de la biblioteca estándar es útil para ficheros enormes que se procesan fila a fila, o para escribir CSV sin pandas.

### 12.1 Leer

```python
import csv

with open("datos/ventas.csv", encoding="utf-8", newline="") as f:
    lector = csv.DictReader(f, delimiter=";")    # cada fila → dict con los nombres de columna
    for fila in lector:
        print(fila["producto"], fila["importe"])  # ¡todos los valores son str!
```

- `csv.reader` devuelve cada fila como **lista**; `csv.DictReader`, como **diccionario**.
- El módulo `csv` **no convierte tipos**: `"12,5"` sigue siendo texto.
- `newline=""` es la forma recomendada de abrir ficheros para el módulo `csv`.

### 12.2 Escribir

```python
filas = [
    {"producto": "teclado", "importe": 25.0},
    {"producto": "ratón", "importe": 12.5},
]

with open("resultados/ventas.csv", "w", encoding="utf-8", newline="") as f:
    escritor = csv.DictWriter(f, fieldnames=["producto", "importe"])
    escritor.writeheader()
    escritor.writerows(filas)
```

### 12.3 Inspeccionar un CSV desconocido

Antes de leer un fichero que no conoces, mira sus primeras líneas para saber el separador, si hay cabecera y la codificación:

```python
with open("datos/desconocido.csv", encoding="utf-8", errors="replace") as f:
    for _ in range(5):
        print(repr(f.readline()))     # repr muestra \t, \r\n y comillas
```

```python
!head -n 5 datos/desconocido.csv      # desde Jupyter en Linux / WSL
```

---

## 13. Ficheros JSON y JSON Lines

### 13.1 JSON ↔ Python

| JSON | Python |
|---|---|
| objeto `{}` | `dict` |
| array `[]` | `list` |
| `"texto"` | `str` |
| `12`, `3.5` | `int`, `float` |
| `true` / `false` | `True` / `False` |
| `null` | `None` |

### 13.2 Leer y escribir

```python
import json

# Fichero → objeto Python
with open("datos/pedido.json", encoding="utf-8") as f:
    pedido = json.load(f)

# Cadena → objeto Python (p. ej., respuesta de una API)
datos = json.loads('{"id": 1, "activo": true, "tags": ["a", "b"]}')

# Objeto Python → fichero
with open("resultados/pedido.json", "w", encoding="utf-8") as f:
    json.dump(pedido, f, ensure_ascii=False, indent=2)

# Objeto Python → cadena
json.dumps(datos, ensure_ascii=False)
```

> `ensure_ascii=False` guarda `ñ` y tildes tal cual en vez de `\u00f1`. `indent=2` lo hace legible.

### 13.3 JSON Lines (`.jsonl`, `.ndjson`)

Formato muy usado en Big Data (logs, exportaciones, Spark, Kafka): **un objeto JSON por línea**. No es un JSON válido completo, así que **no** se lee con `json.load`:

```
{"id": 1, "evento": "login", "ts": "2026-10-20T10:00:00"}
{"id": 2, "evento": "compra", "ts": "2026-10-20T10:05:12"}
```

```python
eventos = []
with open("datos/eventos.jsonl", encoding="utf-8") as f:
    for linea in f:
        if linea.strip():                 # salta líneas vacías
            eventos.append(json.loads(linea))

# Escribir
with open("resultados/eventos.jsonl", "w", encoding="utf-8") as f:
    for e in eventos:
        f.write(json.dumps(e, ensure_ascii=False) + "\n")
```

Con pandas: `pd.read_json("datos/eventos.jsonl", lines=True)`.

### 13.4 Navegar un JSON desconocido

```python
type(pedido)                 # ¿dict o list?
pedido.keys()                # claves del primer nivel
print(json.dumps(pedido, indent=2, ensure_ascii=False)[:1000])   # primeros 1000 caracteres
```

---

## 14. Ficheros comprimidos y descargas

### 14.1 `gzip` (`.gz`)

```python
import gzip, json

with gzip.open("datos/eventos.jsonl.gz", "rt", encoding="utf-8") as f:   # "rt" = leer texto
    for linea in f:
        evento = json.loads(linea)

with gzip.open("resultados/salida.txt.gz", "wt", encoding="utf-8") as f:
    f.write("comprimido al vuelo\n")
```

> pandas descomprime solo: `pd.read_csv("ventas.csv.gz")` o `pd.read_csv("ventas.zip")` (si el ZIP contiene un único fichero).

### 14.2 `zipfile` (`.zip`)

```python
import zipfile

with zipfile.ZipFile("datos/portal_abierto.zip") as z:
    print(z.namelist())                          # ficheros dentro del ZIP
    z.extractall("datos/portal_abierto/")        # extraer todo
    with z.open("municipios.csv") as f:          # leer uno sin extraer
        import pandas as pd
        df = pd.read_csv(f, sep=";", encoding="latin-1")
```

### 14.3 Descargar un fichero

```python
from pathlib import Path
from urllib.request import urlretrieve

url = "https://ejemplo.org/datos/ventas.csv"
destino = Path("datos/ventas.csv")
destino.parent.mkdir(parents=True, exist_ok=True)

if not destino.exists():                 # descarga solo una vez ✅
    urlretrieve(url, destino)
```

Con `requests` (más control; se trata en la UD2):

```py
import requests

r = requests.get(url, timeout=30)
r.raise_for_status()                     # error si la respuesta no es 200
destino.write_bytes(r.content)
```

> Para ficheros grandes, descarga **por bloques** (`requests.get(url, stream=True)` y `r.iter_content(chunk_size=1024*1024)`) en lugar de cargar la respuesta entera en memoria.

---

## 15. Recorrer carpetas con muchos ficheros

En Big Data es habitual que los datos lleguen **particionados**: un fichero por día, por mes o por sensor.

```
datos/ventas/
├── ventas_2026-01.csv
├── ventas_2026-02.csv
├── ...
└── ventas_2026-09.csv
```

### 15.1 Listar y filtrar

```python
from pathlib import Path

carpeta = Path("datos/ventas")
ficheros = sorted(carpeta.glob("ventas_2026-*.csv"))
print(len(ficheros), "ficheros")
print([f.name for f in ficheros[:3]])

total_mb = sum(f.stat().st_size for f in ficheros) / 1024**2
print(f"Tamaño total: {total_mb:.1f} MB")
```

### 15.2 Leer todos y unirlos con pandas

```python
import pandas as pd

partes = []
for f in ficheros:
    df = pd.read_csv(f, sep=";", decimal=",")
    df["fichero_origen"] = f.name          # trazabilidad: de dónde viene cada fila
    partes.append(df)

ventas = pd.concat(partes, ignore_index=True)
```

En una línea con una comprensión:

```python
ventas = pd.concat((pd.read_csv(f) for f in ficheros), ignore_index=True)
```

### 15.3 Procesar sin parar ante un fichero defectuoso

```python
partes, errores = [], []
for f in ficheros:
    try:
        partes.append(pd.read_csv(f))
    except Exception as e:                 # registramos el error y seguimos
        errores.append((f.name, repr(e)))

print(f"Leídos: {len(partes)} · con error: {len(errores)}")
errores[:5]
```

### 15.4 Extraer información del nombre del fichero

```python
import re

f = Path("datos/sensores/sensor_A12_2026-10-20.csv")
m = re.search(r"sensor_(\w+)_(\d{4}-\d{2}-\d{2})", f.stem)
sensor, fecha = m.group(1), m.group(2)      # 'A12', '2026-10-20'
```

### 15.5 Particiones estilo Hive

Spark y muchos sistemas Big Data guardan los datos en carpetas `clave=valor`:

```
datos/ventas_parquet/
├── anio=2026/mes=01/part-0000.parquet
└── anio=2026/mes=02/part-0000.parquet
```

pandas (con `pyarrow`) lee toda la estructura de una vez y crea las columnas `anio` y `mes`:

```python
ventas = pd.read_parquet("datos/ventas_parquet/")
```

---

## 16. Fechas y horas

```python
from datetime import datetime, date, timedelta

ahora = datetime.now()
hoy = date.today()

# Texto → fecha (strptime = "string parse time")
f = datetime.strptime("20/10/2026 14:30", "%d/%m/%Y %H:%M")

# Fecha → texto (strftime = "string format time")
f.strftime("%Y-%m-%d")            # '2026-10-20'  formato ISO, el recomendado
f.isoformat()                     # '2026-10-20T14:30:00'
datetime.fromisoformat("2026-10-20T14:30:00")

# Aritmética
manana = hoy + timedelta(days=1)
(date(2026, 12, 31) - hoy).days   # días que faltan

f.year, f.month, f.day, f.hour, f.weekday()   # weekday: 0 = lunes
```

| Código | Significado | Ejemplo |
|---|---|---|
| `%Y` | Año (4 cifras) | 2026 |
| `%m` | Mes (01–12) | 10 |
| `%d` | Día (01–31) | 20 |
| `%H` / `%M` / `%S` | Hora / minuto / segundo | 14 / 30 / 05 |
| `%j` | Día del año | 293 |
| `%A` / `%B` | Nombre del día / mes (según idioma) | Tuesday / October |

> Para columnas de fechas en tablas usa pandas (`pd.to_datetime`, [manual de pandas](../pandas/manual_pandas.md), sección 15). Guarda las fechas en formato **ISO 8601** (`2026-10-20`): se ordena bien como texto y lo entiende cualquier herramienta.

---

## 17. Herramientas propias del notebook

Resumen de lo más útil (detalle en el [manual de JupyterLab](../jupyterlab/manual_jupyterlab.md), secciones 7 y 10).

### 17.1 Ayuda y autocompletado

| Acción | Cómo |
|---|---|
| Autocompletar nombres, métodos, rutas | `Tab` |
| Ver la firma y la documentación de una función | `Shift + Tab` con el cursor dentro del paréntesis |
| Ayuda completa | `pd.read_csv?` (o `help(pd.read_csv)`) |
| Ver el código fuente | `mi_funcion??` |
| Métodos de un objeto | `dir(objeto)` o escribir `objeto.` + `Tab` |

### 17.2 Comandos mágicos y de sistema

```python
%pwd                     # carpeta de trabajo
%cd ../datos             # cambiar de carpeta (mejor no: usa rutas con pathlib)
%ls                      # listar
!ls -lh datos/           # cualquier comando de la terminal con !
!head -n 3 datos/ventas.csv
!wc -l datos/ventas.csv  # nº de líneas de un fichero (rápido, sin cargarlo)

%who                     # variables definidas
%whos                    # variables con tipo y tamaño
%time resultado = sum(range(10_000_000))     # tiempo de una línea
%%time                   # (primera línea de la celda) tiempo de toda la celda
%timeit sorted(lista)    # repite muchas veces y da el tiempo medio

%load_ext autoreload     # recargar automáticamente tus módulos .py al modificarlos
%autoreload 2
```

### 17.3 Mostrar resultados

```python
from IPython.display import display, Markdown

display(df.head())                       # tabla en mitad de la celda
display(Markdown(f"**Total:** {total:,.2f} €"))

import pandas as pd
pd.set_option("display.max_columns", 50)     # ver más columnas
pd.set_option("display.float_format", "{:,.2f}".format)
```

### 17.4 Barra de progreso para bucles largos

```python
from tqdm.auto import tqdm               # conda install -c conda-forge tqdm

for f in tqdm(ficheros, desc="Leyendo"):
    ...
```

---

## 18. Buenas prácticas en notebooks

1. **Primera celda: todos los `import`**; segunda: **constantes y rutas** (`DATOS`, `SALIDA`).
2. **Rutas relativas** con `pathlib`, nunca `C:\Users\...` (sección 10.5).
3. Indica siempre `encoding="utf-8"` al abrir ficheros de texto.
4. Usa `with` para abrir ficheros.
5. Si repites un bloque de código, conviértelo en **función**. Si la usas en varios notebooks, llévala a un fichero `.py` (`src/utilidades.py`) e impórtala.
6. **Nombres descriptivos**: `ventas_2026`, no `df2`, `df_final_bueno`.
7. **No modifiques los datos originales**: lee de `datos/brutos/`, escribe en `datos/procesados/` o `resultados/`.
8. Celdas **cortas**, cada una con un propósito, y celdas **Markdown** que expliquen qué se hace y por qué.
9. Antes de entregar: **Restart Kernel and Run All Cells** y comprueba que todo se ejecuta de principio a fin.
10. No dejes **contraseñas ni *tokens*** en el notebook: usa variables de entorno (`os.environ["API_KEY"]`) o un fichero `.env` ignorado por Git.

Plantilla de primeras celdas:

```python
# 1. Imports
from pathlib import Path
import json
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

# 2. Configuración
pd.set_option("display.max_columns", 50)

BASE = Path.cwd().parent
DATOS = BASE / "datos"
SALIDA = BASE / "resultados"
SALIDA.mkdir(exist_ok=True)
```

---

## 19. Solución de problemas frecuentes

| Problema | Causa probable | Solución |
|---|---|---|
| `FileNotFoundError` con un fichero que sí existe | La ruta relativa se calcula desde otra carpeta | `Path.cwd()` para ver dónde estás; construye la ruta desde ahí (sección 10) |
| `NameError: name 'df' is not defined` | No se ejecutó la celda que lo crea (o se reinició el kernel) | Ejecuta las celdas anteriores; *Run All Above* |
| `ModuleNotFoundError` pero la librería está instalada | Instalada en otro entorno distinto del kernel | Comprueba el kernel (arriba a la derecha) y `sys.executable`; instala con `%pip` o `%conda` |
| `UnicodeDecodeError: 'utf-8' codec can't decode byte 0xf1` | El fichero está en Latin-1 / cp1252 | `encoding="latin-1"` (sección 11.3) |
| Aparece `Ã±`, `Ã³` en lugar de `ñ`, `ó` | Fichero UTF-8 leído como Latin-1 | `encoding="utf-8"` |
| La primera columna se llama `\ufeffid` o `ï»¿id` | BOM de Excel | `encoding="utf-8-sig"` |
| `json.decoder.JSONDecodeError: Extra data` | Es un fichero JSON Lines | Leer línea a línea (sección 13.3) o `pd.read_json(..., lines=True)` |
| `PermissionError` al escribir (Windows) | El fichero está abierto en Excel | Ciérralo y vuelve a ejecutar |
| `TypeError: can only concatenate str (not "int") to str` | Se suma texto con número | f-string: `f"Total: {n}"` o `str(n)` |
| `IndentationError` | Mezcla de tabuladores y espacios o sangría incorrecta | Usa siempre 4 espacios; `Ctrl + ]` / `Ctrl + [` en Jupyter |
| Los resultados cambian al volver a ejecutar | Celdas ejecutadas fuera de orden o que modifican variables "en el sitio" | *Restart Kernel and Run All Cells* |
| El kernel muere (*Kernel died*) al leer un fichero | Sin memoria suficiente | Lee por partes o solo las columnas necesarias ([manual de pandas](../pandas/manual_pandas.md), sección 18) |
| `!pip install` no hace efecto | Ha instalado en otro Python | Usa `%pip install` |

---

## 20. Chuleta rápida

```py
# ---------- Tipos y conversiones ----------
int("42"); float("3.5"); str(10); round(3.14159, 2)
f"{x:.2f}"  f"{x:,.0f}"  f"{r:.1%}"

# ---------- Cadenas ----------
s.strip().lower().replace(" ", "_");  s.split(";");  ";".join(lista)
s.startswith("a"); s.endswith(".csv"); "x" in s

# ---------- Colecciones ----------
lista.append(x); lista[-1]; lista[1:3]; sorted(lista); len(lista)
d.get("clave", defecto); d.items(); d.keys(); d.values()
[f(x) for x in it if cond];  {k: v for k, v in pares}

# ---------- Flujo ----------
for i, x in enumerate(it): ...
for a, b in zip(l1, l2): ...
try: ... except ValueError as e: ...

# ---------- Rutas ----------
from pathlib import Path
DATOS = Path("..") / "datos"
DATOS.exists(); sorted(DATOS.glob("*.csv")); DATOS.rglob("*.json")
ruta.name; ruta.stem; ruta.suffix; ruta.parent
Path("salida").mkdir(parents=True, exist_ok=True)

# ---------- Ficheros de texto ----------
with open(ruta, encoding="utf-8") as f:
    for linea in f: ...
with open(ruta, "w", encoding="utf-8") as f:
    f.write("texto\n")

# ---------- JSON ----------
import json
with open(r, encoding="utf-8") as f: obj = json.load(f)
with open(r, "w", encoding="utf-8") as f: json.dump(obj, f, ensure_ascii=False, indent=2)
[json.loads(l) for l in open(r, encoding="utf-8")]      # JSON Lines

# ---------- Comprimidos ----------
import gzip, zipfile
gzip.open(r, "rt", encoding="utf-8")
zipfile.ZipFile(r).namelist()

# ---------- Fechas ----------
from datetime import datetime, timedelta
datetime.strptime("20/10/2026", "%d/%m/%Y"); fecha.strftime("%Y-%m-%d")

# ---------- Notebook ----------
%pwd   %whos   %time   %%time   %pip install paquete   !ls -lh datos/
objeto?   Shift+Tab   Tab
```

---

## 21. Referencias

- Tutorial oficial de Python (en español): <https://docs.python.org/es/3/tutorial/>
- Biblioteca estándar: <https://docs.python.org/es/3/library/>
- `pathlib`: <https://docs.python.org/es/3/library/pathlib.html>
- Lectura y escritura de archivos: <https://docs.python.org/es/3/tutorial/inputoutput.html#reading-and-writing-files>
- `csv`: <https://docs.python.org/es/3/library/csv.html>
- `json`: <https://docs.python.org/es/3/library/json.html>
- `datetime` (códigos de formato): <https://docs.python.org/es/3/library/datetime.html#strftime-and-strptime-format-codes>
- Excepciones integradas: <https://docs.python.org/es/3/library/exceptions.html>
- Comandos mágicos de IPython: <https://ipython.readthedocs.io/en/stable/interactive/magics.html>
- Formato JSON Lines: <https://jsonlines.org/>
- *Python for Data Analysis*, 3.ª ed., Wes McKinney (gratuito en línea): <https://wesmckinney.com/book/>

---

## 22. Ejemplo resuelto: unificar una carpeta de ficheros

**Situación:** una red de estaciones meteorológicas deja cada día un fichero **JSON Lines comprimido** con las lecturas de sus sensores. Queremos juntar todos los días en un único CSV con una columna que indique la fecha del fichero, sin que un fichero corrupto detenga el proceso.

```
datos/meteo/
├── lecturas_2026-10-18.jsonl.gz
├── lecturas_2026-10-19.jsonl.gz
└── lecturas_2026-10-20.jsonl.gz     ← tiene una línea corrupta
```

**Paso 1 — Generar datos de ejemplo** (en un caso real ya existirían):

```python
from pathlib import Path
import gzip, json, random
from datetime import date, timedelta

DATOS = Path("datos/meteo")
DATOS.mkdir(parents=True, exist_ok=True)

random.seed(42)
for i in range(3):
    dia = date(2026, 10, 18) + timedelta(days=i)
    with gzip.open(DATOS / f"lecturas_{dia}.jsonl.gz", "wt", encoding="utf-8") as f:
        for estacion in ["Ferrol", "Coruña", "Vigo"]:
            lectura = {"estacion": estacion,
                       "temp": round(random.uniform(10, 20), 1),
                       "humedad": random.randint(60, 95)}
            f.write(json.dumps(lectura, ensure_ascii=False) + "\n")
        if i == 2:
            f.write('{"estacion": "Lugo", "temp": \n')     # línea corrupta a propósito
```

**Paso 2 — Leer todos los ficheros, línea a línea, registrando errores:**

```python
import re

registros, errores = [], []

for ruta in sorted(DATOS.glob("lecturas_*.jsonl.gz")):
    fecha = re.search(r"\d{4}-\d{2}-\d{2}", ruta.name).group()   # fecha desde el nombre
    with gzip.open(ruta, "rt", encoding="utf-8") as f:
        for n_linea, linea in enumerate(f, start=1):
            try:
                lectura = json.loads(linea)
            except json.JSONDecodeError as e:
                errores.append({"fichero": ruta.name, "linea": n_linea, "error": str(e)})
                continue
            lectura["fecha"] = fecha
            registros.append(lectura)

print(f"{len(registros)} lecturas válidas, {len(errores)} con error")
errores
```

```
9 lecturas válidas, 1 con error
[{'fichero': 'lecturas_2026-10-20.jsonl.gz', 'linea': 4, 'error': 'Expecting value: line 2 column 1 (char 30)'}]
```

**Paso 3 — Resumen con Python puro:**

```python
from collections import defaultdict

temps = defaultdict(list)
for r in registros:
    temps[r["estacion"]].append(r["temp"])

for estacion, valores in sorted(temps.items()):
    print(f"{estacion:<8} media: {sum(valores) / len(valores):5.1f} °C  ({len(valores)} lecturas)")
```

```
Coruña   media:  14.8 °C  (3 lecturas)
Ferrol   media:  15.1 °C  (3 lecturas)
Vigo     media:  11.5 °C  (3 lecturas)
```

**Paso 4 — Pasar a pandas y guardar el resultado:**

```python
import pandas as pd

df = pd.DataFrame(registros)
df["fecha"] = pd.to_datetime(df["fecha"])

SALIDA = Path("resultados")
SALIDA.mkdir(exist_ok=True)
df.to_csv(SALIDA / "lecturas_unificadas.csv", index=False, encoding="utf-8")
df.head()
```

**Qué hemos usado:** `pathlib` y `glob` para localizar ficheros, `gzip` para leerlos sin descomprimir en disco, lectura **línea a línea** (no carga todo en memoria), `json.loads` para cada línea, `re` para extraer la fecha del nombre, `try/except` para no detenerse ante errores, y pandas para guardar el resultado. Es el mismo patrón que aplicarás con miles de ficheros en la UD2.
