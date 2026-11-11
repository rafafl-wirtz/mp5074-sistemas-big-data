# pandas para estadística

**Módulo:** MP5074 – Sistemas de Big Data · **UD1** · Curso 2026-27

> Este manual recoge **solo el pandas que hace falta** para seguir el [manual de estadística descriptiva y agregados](../estadistica/manual_estadistica.md): crear datos, resumirlos con una función, filtrar, tratar valores ausentes y agrupar. Todo lo demás (leer y limpiar ficheros reales, unir tablas, fechas, rendimiento…) está en el [manual completo de pandas](../pandas/manual_pandas.md), que se ve después de la estadística.
>
> Todos los ejemplos usan **los mismos datos que el manual de estadística**, para que al llegar allí ya sepas escribir el código.

!!! abstract "Cómo usar este manual"
    Ábrelo junto a un notebook de JupyterLab y **escribe tú cada ejemplo** (no lo copies y pegues). Debajo de cada bloque de código está la salida que tienes que obtener.

---

## Índice

1. [Qué es pandas y cómo se importa](#1-qué-es-pandas-y-cómo-se-importa)
2. [`Series`: una columna de datos](#2-series-una-columna-de-datos)
3. [Resumir una columna en un número](#3-resumir-una-columna-en-un-número)
4. [Datos categóricos: contar y frecuencias](#4-datos-categóricos-contar-y-frecuencias)
5. [Filtrar con condiciones](#5-filtrar-con-condiciones)
6. [Valores ausentes (`NaN`)](#6-valores-ausentes-nan)
7. [`DataFrame`: una tabla](#7-dataframe-una-tabla)
8. [Agrupar: `groupby`](#8-agrupar-groupby)
9. [Juntar y repartir datos](#9-juntar-y-repartir-datos)
10. [Gráficos rápidos](#10-gráficos-rápidos)
11. [Pincelada: datos con fecha](#11-pincelada-datos-con-fecha)
12. [Errores frecuentes](#12-errores-frecuentes)
13. [Chuleta](#13-chuleta)

---

## 1. Qué es pandas y cómo se importa

**pandas** es la librería de Python para trabajar con **datos en forma de tabla**. Con una sola línea calcula la media de un millón de valores, cuenta los ausentes o agrupa por tienda. Por dentro usa **NumPy**, otra librería que hace los cálculos numéricos muy rápido.

Por convenio, siempre se importan así, al principio del notebook:

```python
import numpy as np
import pandas as pd

print(pd.__version__)
```

```text
3.0.5
```

Si da `ModuleNotFoundError`, falta instalarlo en tu entorno: `conda install pandas`.

pandas tiene dos estructuras de datos:

| Estructura | Qué es | Ejemplo |
|---|---|---|
| `Series` | **Una columna** de datos con una etiqueta por fila | Los sueldos de una empresa |
| `DataFrame` | **Una tabla**: varias `Series` que comparten las filas | Los pedidos, con su importe, tienda y método de pago |

En estadística casi siempre trabajaremos con **una columna** (una `Series`) y la resumiremos en un número. Por eso empezamos por ahí.

---

## 2. `Series`: una columna de datos

### 2.1 Crear una `Series`

Se crea a partir de una **lista** de Python:

```python
sueldos = pd.Series([1500, 1600, 1600, 1800, 2000, 2200, 25000], name="sueldo")
sueldos
```

```text
0     1500
1     1600
2     1600
3     1800
4     2000
5     2200
6    25000
Name: sueldo, dtype: int64
```

Cada fila tiene dos cosas:

- A la izquierda, el **índice** (`0, 1, 2…`): la etiqueta de cada fila. Por defecto es la posición.
- A la derecha, el **valor**.

Abajo aparecen el nombre (`sueldo`) y el **tipo de dato** (`dtype: int64`, números enteros). Si un solo valor tuviera decimales, toda la columna sería `float64`.

### 2.2 Acceder a los valores

```python
print(len(sueldos))        # cuántos valores hay
print(sueldos.iloc[0])     # el primero (posición 0)
print(sueldos.iloc[-1])    # el último
print(sueldos.iloc[:3].tolist())   # los tres primeros, como lista
```

```text
7
1500
25000
[1500, 1600, 1600]
```

### 2.3 Operar con toda la columna a la vez

Una operación sobre una `Series` se aplica a **todos sus valores**, sin escribir bucles. Esto se llama **operación vectorizada**:

```python
sueldos * 1.03          # subida del 3 % a todo el mundo
```

```text
0     1545.0
1     1648.0
2     1648.0
3     1854.0
4     2060.0
5     2266.0
6    25750.0
Name: sueldo, dtype: float64
```

```python
(sueldos - sueldos.mean()).round(1)   # cuánto se aleja cada sueldo de la media
```

```text
0    -3600.0
1    -3500.0
2    -3500.0
3    -3300.0
4    -3100.0
5    -2900.0
6    19900.0
Name: sueldo, dtype: float64
```

La segunda línea es justo la **desviación** de cada dato respecto a la media, el primer paso para calcular la varianza (apartado 3.2 del manual de estadística).

### 2.4 Ordenar

```python
sueldos.sort_values().tolist()                    # de menor a mayor
```

```text
[1500, 1600, 1600, 1800, 2000, 2200, 25000]
```

```python
sueldos.sort_values(ascending=False).head(3)      # los 3 mayores
```

```text
6    25000
5     2200
4     2000
Name: sueldo, dtype: int64
```

`head(n)` devuelve las `n` primeras filas y `tail(n)` las `n` últimas. Fíjate en que, al ordenar, **cada valor conserva su índice original** (el 25 000 sigue siendo la fila 6).

---

## 3. Resumir una columna en un número

Esta es la parte que más usarás en estadística. Una **función de agregación** recibe toda la columna y devuelve **un único valor**.

### 3.1 Tendencia central

```python
print("Media:  ", sueldos.mean())
print("Mediana:", sueldos.median())
print("Moda:   ", sueldos.mode().tolist())
```

```text
Media:   5100.0
Mediana: 1800.0
Moda:    [1600]
```

!!! warning "`mode()` devuelve varias filas"
    La media y la mediana son siempre un número, pero puede haber **varias modas**. Por eso `mode()` devuelve una `Series`, no un número. Para quedarte con la primera: `sueldos.mode().iloc[0]`.

### 3.2 Contar y extremos

```python
print("Valores:", sueldos.count())
print("Suma:   ", sueldos.sum())
print("Mínimo: ", sueldos.min(), " Máximo:", sueldos.max())
print("Rango:  ", sueldos.max() - sueldos.min())
```

```text
Valores: 7
Suma:    35700
Mínimo:  1500  Máximo: 25000
Rango:   23500
```

### 3.3 Dispersión

```python
a = pd.Series([48, 50, 52])
b = pd.Series([0, 50, 100])

print("Varianza A:", a.var(), "  desviación típica A:", a.std())
print("Varianza B:", b.var(), "  desviación típica B:", b.std())
```

```text
Varianza A: 4.0   desviación típica A: 2.0
Varianza B: 2500.0   desviación típica B: 50.0
```

El **coeficiente de variación** no tiene función propia: se calcula con las dos anteriores.

```python
cv = sueldos.std() / sueldos.mean() * 100
print(f"CV de los sueldos: {cv:.1f} %")
```

```text
CV de los sueldos: 172.1 %
```

!!! info "pandas divide entre n − 1"
    `var()` y `std()` de pandas calculan la varianza **de una muestra** (dividen entre *n − 1*). NumPy (`np.var`, `np.std`) divide entre *n*. Si alguna vez necesitas la de NumPy: `a.std(ddof=0)`.

### 3.4 Cuartiles y percentiles

`quantile(p)` devuelve el valor que deja por debajo la proporción `p` de los datos (entre 0 y 1):

```python
prep = pd.Series([12, 15, 15, 18, 20, 22, 25, 30, 95], name="minutos")

print("Q1:", prep.quantile(0.25))
print("Q3:", prep.quantile(0.75))
prep.quantile([0.25, 0.5, 0.75])      # varios a la vez
```

```text
Q1: 15.0
Q3: 25.0
0.25    15.0
0.50    20.0
0.75    25.0
Name: minutos, dtype: float64
```

Para usar los resultados, se guardan en variables:

```python
q1, q3 = prep.quantile([0.25, 0.75])
iqr = q3 - q1
print(f"Q1 = {q1}   Q3 = {q3}   IQR = {iqr}")
```

```text
Q1 = 15.0   Q3 = 25.0   IQR = 10.0
```

### 3.5 Todo de golpe: `describe()`

```python
prep.describe()
```

```text
count     9.000000
mean     28.000000
std      25.729361
min      12.000000
25%      15.000000
50%      20.000000
75%      25.000000
max      95.000000
Name: minutos, dtype: float64
```

`describe()` es el primer comando que se ejecuta al explorar una columna numérica: número de valores, media, desviación típica, mínimo, cuartiles y máximo. Se le pueden pedir otros percentiles:

```python
tiempos = pd.Series([110, 120, 125, 130, 140, 150, 160, 180, 900, 2500], name="ms")
tiempos.describe(percentiles=[0.5, 0.9, 0.95]).round(1)
```

```text
count      10.0
mean      451.5
std       758.7
min       110.0
50%       145.0
90%      1060.0
95%      1780.0
max      2500.0
Name: ms, dtype: float64
```

### 3.6 Forma de la distribución

```python
print("Asimetría de los sueldos:", round(sueldos.skew(), 2))
print("Asimetría de A:          ", a.skew())
```

```text
Asimetría de los sueldos: 2.64
Asimetría de A:           0.0
```

### 3.7 Resumen: de la estadística a pandas

| Medida (manual de estadística) | pandas |
|---|---|
| Media / mediana / moda | `s.mean()` / `s.median()` / `s.mode()` |
| Número de valores / suma | `s.count()` / `s.sum()` |
| Mínimo / máximo / rango | `s.min()` / `s.max()` / `s.max() - s.min()` |
| Varianza / desviación típica | `s.var()` / `s.std()` |
| Coeficiente de variación | `s.std() / s.mean() * 100` |
| Cuartil, percentil *p* | `s.quantile(0.25)`, `s.quantile(0.95)` |
| Resumen completo | `s.describe()` |
| Asimetría | `s.skew()` |

---

## 4. Datos categóricos: contar y frecuencias

Con texto no se puede calcular una media, pero sí **contar**:

```python
pagos = pd.Series(["Tarjeta", "Bizum", "Tarjeta", "PayPal",
                   "Bizum", "Tarjeta", "Bizum", "Transferencia"])
pagos.value_counts()          # tabla de frecuencias, de más a menos
```

```text
Tarjeta          3
Bizum            3
PayPal           1
Transferencia    1
Name: count, dtype: int64
```

```python
pagos.value_counts(normalize=True)   # frecuencias relativas (proporciones)
```

```text
Tarjeta          0.375
Bizum            0.375
PayPal           0.125
Transferencia    0.125
Name: proportion, dtype: float64
```

```python
print("Valores distintos:", pagos.nunique())
print("Moda:", pagos.mode().tolist())    # dos modas: distribución bimodal
```

```text
Valores distintos: 4
Moda: ['Bizum', 'Tarjeta']
```

---

## 5. Filtrar con condiciones

### 5.1 Comparar toda la columna

Una comparación sobre una `Series` devuelve otra `Series` de `True` / `False`, una por fila. Se llama **máscara**:

```python
prep > 40
```

```text
0    False
1    False
2    False
3    False
4    False
5    False
6    False
7    False
8     True
Name: minutos, dtype: bool
```

### 5.2 Quedarse con las filas que cumplen

Si se pone la máscara entre corchetes, se quedan solo las filas con `True`:

```python
prep[prep > 40]
```

```text
8    95
Name: minutos, dtype: int64
```

Así se aplica la **regla del IQR** para encontrar atípicos:

```python
lim_inf = q1 - 1.5 * iqr
lim_sup = q3 + 1.5 * iqr
atipicos = prep[(prep < lim_inf) | (prep > lim_sup)]
atipicos
```

```text
8    95
Name: minutos, dtype: int64
```

!!! danger "Paréntesis y `&`, `|`, `~`"
    Para combinar condiciones se usan `&` (y), `|` (o) y `~` (no), **no** `and`, `or`, `not`. Y **cada condición va entre paréntesis**: `(prep < lim_inf) | (prep > lim_sup)`. Sin paréntesis da error.

### 5.3 Contar y calcular proporciones con una máscara

Python trata `True` como 1 y `False` como 0. Por eso:

- `máscara.sum()` cuenta cuántas filas la cumplen.
- `máscara.mean()` da **la proporción** de filas que la cumplen.

```python
print("Peticiones de más de 500 ms:", (tiempos > 500).sum())
print(f"Proporción de 500 ms o menos: {(tiempos <= 500).mean():.0%}")
```

```text
Peticiones de más de 500 ms: 2
Proporción de 500 ms o menos: 80%
```

Es la forma de responder preguntas como «¿qué porcentaje de peticiones cumple el SLA?».

---

## 6. Valores ausentes (`NaN`)

Cuando falta un dato, pandas lo representa como **`NaN`** (*Not a Number*). En una lista de Python se escribe `None`:

```python
importes = pd.Series([100, None, 50, None, 150])
importes
```

```text
0    100.0
1      NaN
2     50.0
3      NaN
4    150.0
dtype: float64
```

Fíjate en que la columna ha pasado a `float64` (decimales): `NaN` es un valor decimal.

```python
print("Filas:          ", len(importes))
print("Valores (count):", importes.count())
print("Ausentes:       ", importes.isna().sum())
print("Media:          ", importes.mean())
```

```text
Filas:           5
Valores (count): 3
Ausentes:        2
Media:           100.0
```

!!! danger "Las funciones de agregación ignoran los ausentes"
    `count()`, `sum()`, `mean()`, `median()`… **saltan los `NaN`**: la media de arriba es 300 / 3 = 100, no 300 / 5. `len()` en cambio cuenta todas las filas. Es lo mismo que pasa en SQL con `COUNT(importe)` frente a `COUNT(*)` (apartado 4.1 del manual de estadística).

---

## 7. `DataFrame`: una tabla

### 7.1 Crear un `DataFrame`

Se crea con un **diccionario**: cada clave es el nombre de una columna y cada valor, una lista con sus datos:

```python
pedidos = pd.DataFrame({
    "pedido_id": [1, 2, 3, 4, 5, 6, 7, 8],
    "tienda":    ["Vigo", "Lugo", "Vigo", "Ourense", "Lugo", "Vigo", "Ourense", "Vigo"],
    "unidades":  [1, 4, 2, 1, 2, 3, 1, 1],
    "importe":   [50, 200, 70, 30, 100, 90, 45, None],
    "metodo_pago": ["Tarjeta", "Bizum", "Tarjeta", None, "Tarjeta", "Bizum", "Tarjeta", "Bizum"],
})
pedidos
```

```text
   pedido_id   tienda  unidades  importe metodo_pago
0          1     Vigo         1     50.0     Tarjeta
1          2     Lugo         4    200.0       Bizum
2          3     Vigo         2     70.0     Tarjeta
3          4  Ourense         1     30.0         NaN
4          5     Lugo         2    100.0     Tarjeta
5          6     Vigo         3     90.0       Bizum
6          7  Ourense         1     45.0     Tarjeta
7          8     Vigo         1      NaN       Bizum
```

### 7.2 Primer vistazo

```python
print(pedidos.shape)        # (filas, columnas)
pedidos.info()
```

```text
(8, 5)
<class 'pandas.DataFrame'>
RangeIndex: 8 entries, 0 to 7
Data columns (total 5 columns):
 #   Column       Non-Null Count  Dtype  
---  ------       --------------  -----  
 0   pedido_id    8 non-null      int64  
 1   tienda       8 non-null      str    
 2   unidades     8 non-null      int64  
 3   importe      7 non-null      float64
 4   metodo_pago  7 non-null      str    
dtypes: float64(1), int64(2), str(2)
memory usage: 534.0 bytes
```

`info()` muestra cada columna, cuántos valores **no nulos** tiene y su tipo. Aquí se ve de un vistazo que faltan un importe y un método de pago.

```python
pedidos.describe()          # resumen de las columnas numéricas
```

```text
       pedido_id  unidades     importe
count    8.00000  8.000000    7.000000
mean     4.50000  1.875000   83.571429
std      2.44949  1.125992   57.060953
min      1.00000  1.000000   30.000000
25%      2.75000  1.000000   47.500000
50%      4.50000  1.500000   70.000000
75%      6.25000  2.250000   95.000000
max      8.00000  4.000000  200.000000
```

### 7.3 Elegir columnas

- **Una** columna, con un nombre entre corchetes: devuelve una `Series`, y a partir de ahí funciona todo lo de los apartados 2 a 6.
- **Varias** columnas, con una **lista** de nombres (dobles corchetes): devuelve un `DataFrame`.

```python
pedidos["importe"].mean()
```

```text
83.57142857142857
```

```python
pedidos[["tienda", "importe"]].head(3)
```

```text
  tienda  importe
0   Vigo     50.0
1   Lugo    200.0
2   Vigo     70.0
```

### 7.4 Crear una columna nueva

Se asigna una operación sobre otras columnas; se calcula para todas las filas a la vez:

```python
pedidos["precio_unidad"] = pedidos["importe"] / pedidos["unidades"]
pedidos.head(3)
```

```text
   pedido_id tienda  unidades  importe metodo_pago  precio_unidad
0          1   Vigo         1     50.0     Tarjeta           50.0
1          2   Lugo         4    200.0       Bizum           50.0
2          3   Vigo         2     70.0     Tarjeta           35.0
```

### 7.5 Filtrar filas

Igual que con una `Series`: una condición sobre una columna entre corchetes.

```python
pedidos[pedidos["tienda"] == "Vigo"]
```

```text
   pedido_id tienda  unidades  importe metodo_pago  precio_unidad
0          1   Vigo         1     50.0     Tarjeta           50.0
2          3   Vigo         2     70.0     Tarjeta           35.0
5          6   Vigo         3     90.0       Bizum           30.0
7          8   Vigo         1      NaN       Bizum            NaN
```

```python
pedidos[(pedidos["tienda"] == "Vigo") & (pedidos["importe"] > 60)]
```

```text
   pedido_id tienda  unidades  importe metodo_pago  precio_unidad
2          3   Vigo         2     70.0     Tarjeta           35.0
5          6   Vigo         3     90.0       Bizum           30.0
```

```python
pedidos[pedidos["metodo_pago"].isin(["Bizum", "PayPal"])]
```

```text
   pedido_id tienda  unidades  importe metodo_pago  precio_unidad
1          2   Lugo         4    200.0       Bizum           50.0
5          6   Vigo         3     90.0       Bizum           30.0
7          8   Vigo         1      NaN       Bizum            NaN
```

---

## 8. Agrupar: `groupby`

Es el equivalente en pandas del `GROUP BY` de SQL: **divide** las filas en grupos, **aplica** una función a cada grupo y **combina** los resultados en una tabla.

### 8.1 Una medida por grupo

```python
pedidos.groupby("tienda")["importe"].sum()
```

```text
tienda
Lugo       300.0
Ourense     75.0
Vigo       210.0
Name: importe, dtype: float64
```

Se lee de izquierda a derecha: *agrupa por tienda* → *toma la columna importe* → *súmala en cada grupo*. El resultado es una `Series` cuyo índice son las tiendas.

### 8.2 Varias medidas con nombre: `agg`

```python
resumen = (pedidos
           .groupby("tienda")
           .agg(pedidos=("pedido_id", "count"),
                facturacion=("importe", "sum"),
                ticket_medio=("importe", "mean"))
           .sort_values("facturacion", ascending=False))
resumen
```

```text
         pedidos  facturacion  ticket_medio
tienda                                     
Lugo           2        300.0         150.0
Vigo           4        210.0          70.0
Ourense        2         75.0          37.5
```

Cada medida se escribe `nombre_nueva_columna=("columna", "función")`. Funciones habituales: `"count"`, `"sum"`, `"mean"`, `"median"`, `"min"`, `"max"`, `"std"`, `"nunique"`.

Fíjate en Vigo: tiene 4 pedidos, pero su ticket medio está calculado con **3 importes** (el cuarto es `NaN`). Para contar pedidos usa una columna sin ausentes, como `pedido_id`.

### 8.3 Filtrar grupos (el `HAVING` de SQL)

En pandas no hace falta nada especial: primero se agrupa y después se filtra el resultado como cualquier tabla.

```python
resumen[resumen["facturacion"] > 100]
```

```text
        pedidos  facturacion  ticket_medio
tienda                                    
Lugo          2        300.0         150.0
Vigo          4        210.0          70.0
```

---

## 9. Juntar y repartir datos

En el apartado 5 del manual de estadística se **simulan los nodos de un clúster** con varias `Series`. Hacen falta dos herramientas:

- Un **diccionario** de Python para guardar una `Series` por nodo.
- **`pd.concat`** para juntarlas en una sola.

```python
nodos = {
    "A": pd.Series([2, 4, 6]),
    "B": pd.Series([10, 20, 30, 40, 50]),
}
todos = pd.concat(nodos.values(), ignore_index=True)
todos.tolist()
```

```text
[2, 4, 6, 10, 20, 30, 40, 50]
```

```python
for nombre, datos in nodos.items():
    print(nombre, "-> suma:", datos.sum(), " conteo:", datos.count(), " media:", datos.mean())
print("Global -> media real:", todos.mean())
```

```text
A -> suma: 12  conteo: 3  media: 4.0
B -> suma: 150  conteo: 5  media: 30.0
Global -> media real: 20.25
```

`ignore_index=True` crea un índice nuevo `0, 1, 2…`; sin él, se repetirían los índices de cada nodo.

Para tomar una **muestra aleatoria** (apartado 6 del manual de estadística) se usa `sample`:

```python
todos.sample(frac=0.5, random_state=1).tolist()    # el 50 % de los valores, al azar
```

```text
[50, 6, 4, 40]
```

`random_state` fija el azar: con el mismo número, siempre sale la misma muestra.

---

## 10. Gráficos rápidos

pandas dibuja directamente desde una `Series` (usa la librería **matplotlib** por debajo):

```python
tiempos.plot.hist(bins=10)        # histograma: forma de la distribución
prep.plot.box(vert=False)         # diagrama de caja: cuartiles y atípicos
resumen["facturacion"].plot.bar() # barras: comparar grupos
```

En un notebook, el gráfico aparece debajo de la celda. Para el detalle de cómo personalizarlos, consulta el manual completo de pandas.

---

## 11. Pincelada: datos con fecha

Solo para el apartado 7 del manual de estadística (ventanas temporales). Si la tabla tiene una columna de fecha, se puede agregar **por día** con `resample` y calcular una **media móvil** con `rolling`:

```python
ventas = pd.DataFrame({
    "fecha": pd.date_range("2026-03-01", periods=10, freq="D"),
    "importe": [120, 135, 150, 110, 160, 200, 180, 170, 140, 190],
})
diario = ventas.set_index("fecha")["importe"].resample("D").sum()   # ventana fija de 1 día
diario.rolling(3).mean().round(2).tolist()                          # media móvil de 3 días
```

```text
[nan, nan, 135.0, 131.67, 140.0, 156.67, 180.0, 183.33, 163.33, 166.67]
```

Las fechas, `resample` y `rolling` se estudian a fondo en el manual completo de pandas. Aquí basta con saber leer estas dos líneas.

---

## 12. Errores frecuentes

| Error o síntoma | Causa | Solución |
|---|---|---|
| `NameError: name 'pd' is not defined` | No se ha ejecutado la celda de `import` (o se reinició el kernel) | Ejecuta `import pandas as pd` |
| `ValueError: The truth value of a Series is ambiguous` | Se usó `and` / `or` con columnas | Usa `&` / `|` y paréntesis en cada condición |
| `TypeError` al calcular la media | La columna es de texto | Comprueba el tipo con `info()` o `dtypes` |
| La moda sale como una tabla | `mode()` devuelve una `Series` | `s.mode().iloc[0]` para la primera |
| `KeyError: 'Importe'` | El nombre de columna no coincide (mayúsculas, espacios) | Mira los nombres con `df.columns` |
| La media no coincide con la calculada a mano | Hay `NaN` y la media los ignora | Revisa `isna().sum()` |
| La desviación típica no coincide con NumPy | pandas divide entre *n − 1* y NumPy entre *n* | `s.std(ddof=0)` |

---

## 13. Chuleta

```python
import numpy as np
import pandas as pd

s = pd.Series([3, 1, 4, 1, 5])                   # una columna
df = pd.DataFrame({"x": [1, 2], "g": ["a", "b"]})   # una tabla

s.mean(); s.median(); s.mode()                   # tendencia central
s.count(); s.sum(); s.min(); s.max()             # contar y extremos
s.var(); s.std(); s.skew()                       # dispersión y forma
s.quantile([0.25, 0.5, 0.75]); s.describe()      # posición
s.value_counts(); s.nunique()                    # categorías
s[s > 2]; (s > 2).sum(); (s > 2).mean()          # filtrar, contar, proporción
s.isna().sum()                                   # ausentes

df.shape; df.info(); df.describe()               # primer vistazo
df["x"]; df[["x", "g"]]                          # columnas
df["y"] = df["x"] * 2                            # columna nueva
df[(df["x"] > 1) & (df["g"] == "b")]             # filtrar filas
df.groupby("g")["x"].sum()                       # agrupar
df.groupby("g").agg(n=("x", "count"), media=("x", "mean"))
pd.concat([s, s], ignore_index=True)             # juntar
s.sample(frac=0.1, random_state=1)               # muestra
```

**Siguiente paso:** el [manual de estadística descriptiva y agregados](../estadistica/manual_estadistica.md). Después, el [manual completo de pandas](../pandas/manual_pandas.md).
