# UD1 · Estadística descriptiva y funciones de agregación en Big Data

## Índice

- [Introducción](#introducción)
- [1. ¿Por qué estadística en un sistema de Big Data?](#1-por-qué-estadística-en-un-sistema-de-big-data)
- [2. Medidas de tendencia central](#2-medidas-de-tendencia-central)
    - [2.1 Media aritmética](#21-media-aritmética)
    - [2.2 Mediana](#22-mediana)
    - [2.3 Moda](#23-moda)
    - [2.4 Ejemplo comparado](#24-ejemplo-comparado)
    - [2.5 Media ponderada](#25-media-ponderada)
- [3. Medidas de dispersión y de posición](#3-medidas-de-dispersión-y-de-posición)
    - [3.1 Rango](#31-rango)
    - [3.2 Varianza y desviación típica](#32-varianza-y-desviación-típica)
    - [3.3 Coeficiente de variación](#33-coeficiente-de-variación)
    - [3.4 Cuartiles, percentiles y rango intercuartílico](#34-cuartiles-percentiles-y-rango-intercuartílico)
    - [3.5 Percentiles en la práctica: tiempos de respuesta](#35-percentiles-en-la-práctica-tiempos-de-respuesta)
    - [3.6 Valores atípicos (outliers)](#36-valores-atípicos-outliers)
    - [3.7 Forma de la distribución](#37-forma-de-la-distribución)
- [4. Funciones de agregación](#4-funciones-de-agregación)
    - [4.1 Las funciones básicas](#41-las-funciones-básicas)
    - [4.2 Agrupar: GROUP BY](#42-agrupar-group-by)
    - [4.3 Filtrar grupos: HAVING](#43-filtrar-grupos-having)
- [5. Agregados en sistemas distribuidos](#5-agregados-en-sistemas-distribuidos)
    - [5.1 Ejemplo de partida](#51-ejemplo-de-partida)
    - [5.2 Distributivas](#52-distributivas)
    - [5.3 Algebraicas](#53-algebraicas)
    - [5.4 Holísticas](#54-holísticas)
    - [5.5 Resumen y consecuencias](#55-resumen-y-consecuencias)
- [6. Pincelada: agregados aproximados](#6-pincelada-agregados-aproximados)
- [7. Pincelada: agregación en el tiempo](#7-pincelada-agregación-en-el-tiempo)
- [8. Errores habituales](#8-errores-habituales)
- [9. Resumen](#9-resumen)
- [Lecturas y referencias](#lecturas-y-referencias)

---

## Introducción

Cualquier sistema de Big Data, por grande o complejo que sea, acaba respondiendo preguntas como estas: *¿cuánto hemos vendido esta semana?*, *¿cuál es el pedido típico?*, *¿cuánto tarda nuestra web en responder al 99 % de los usuarios?* Todas se resuelven **resumiendo millones de datos en unos pocos números**. Esos resúmenes son las **medidas estadísticas** y las **funciones de agregación**.

En esta unidad aprenderás qué significa cada medida, cuándo conviene usar una u otra y, sobre todo, **por qué algunas son fáciles de calcular cuando los datos están repartidos entre muchas máquinas y otras no**. Esta última idea es la que diferencia la estadística "de hoja de cálculo" de la estadística en un sistema de Big Data.

### Objetivos de aprendizaje

Al terminar esta unidad serás capaz de:

- Calcular e interpretar media, mediana y moda, y elegir la adecuada según la distribución de los datos.
- Medir la dispersión de un conjunto de datos con rango, varianza, desviación típica, coeficiente de variación, cuartiles y percentiles.
- Detectar valores atípicos y valorar su efecto sobre las medidas.
- Utilizar funciones de agregación con agrupaciones, tanto en SQL como en Pandas.
- Clasificar una agregación como distributiva, algebraica u holística y explicar sus consecuencias en un sistema distribuido.
- Reconocer cuándo un sistema de Big Data usa agregados aproximados o ventanas temporales.

### Requisitos previos

- Python básico (variables, listas, funciones).
- Nociones de SQL: `SELECT`, `WHERE`, `ORDER BY`.
- Saber qué es un fichero CSV.

---

## 1. ¿Por qué estadística en un sistema de Big Data?

Un humano no puede leer mil millones de filas. Un sistema de Big Data sí puede recorrerlas, pero lo que entrega a las personas (un informe, un *dashboard*, una alerta) son siempre **resúmenes**. Por eso las agregaciones aparecen en todas las capas:

| Capa | Ejemplo de agregación |
|---|---|
| Ingesta y calidad | Contar registros recibidos, nulos y duplicados por hora |
| Almacenamiento analítico | Tablas preagregadas de ventas por día y tienda |
| Procesamiento en tiempo real | Media de temperatura de cada sensor cada 5 minutos |
| Monitorización | Percentil 99 del tiempo de respuesta de un servicio |
| Machine Learning | Media y desviación típica para normalizar variables |

!!! abstract "Definición"
    **Agregación**: operación que toma un conjunto de valores y devuelve un único valor que los resume (una suma, un conteo, una media...).

!!! abstract "Definición"
    **Estadística descriptiva**: rama de la estadística que describe y resume un conjunto de datos, sin intentar sacar conclusiones sobre una población mayor (eso sería estadística inferencial).

---

## 2. Medidas de tendencia central

Responden a la pregunta *"¿cuál es el valor típico?"*.

### 2.1 Media aritmética

Suma de todos los valores dividida entre el número de valores:

$$\bar{x} = \frac{x_1 + x_2 + \dots + x_n}{n} = \frac{\sum x_i}{n}$$

!!! info "Cómo leer la fórmula"
    - $\bar{x}$ (se lee «x barra») es el símbolo de la **media**. Lo verás en muchas otras fórmulas del manual.
    - $x_i$ es cada uno de los datos: $x_1$ es el primero, $x_2$ el segundo, y así hasta $x_n$.
    - $n$ es el número de datos.
    - $\sum$ (sigma mayúscula) significa «suma de»: $\sum x_i$ es la suma de todos los datos.

Es la medida más usada, pero tiene un punto débil importante: **un solo valor extremo la desplaza mucho**.

### 2.2 Mediana

Es el valor que queda **en el centro** cuando ordenamos los datos. La mitad de los valores está por debajo y la otra mitad por encima.

- Si *n* es impar, es el valor central.
- Si *n* es par, es la media de los dos valores centrales.

La mediana es **robusta**: los valores extremos apenas la afectan.

**Ejemplo.** Con 1500, 1600, 1600, 1800, 2000, 2200 (*n* par), la mediana es (1600 + 1800) / 2 = **1700 €**: puede ser un valor que no aparece en los datos.

### 2.3 Moda

Es el valor **que más se repite**. Puede haber una moda, varias (distribución *bimodal* o *multimodal*) o ninguna clara si todos los valores son distintos.

Es la única de las tres que se puede usar con **datos categóricos**: tiene sentido hablar del "método de pago más frecuente", pero no del "método de pago medio".

**Ejemplo.** En los pagos `Tarjeta, Bizum, Tarjeta, PayPal, Bizum, Tarjeta, Bizum, Transferencia`, Tarjeta y Bizum aparecen 3 veces cada uno: la distribución es **bimodal**. Por eso en Pandas `.mode()` devuelve siempre una serie (`pagos.mode()` → `['Bizum', 'Tarjeta']`), y `.value_counts()` da la tabla de frecuencias.

### 2.4 Ejemplo comparado

Sueldos mensuales de una pequeña empresa de 7 personas (en euros):

```text
1500, 1600, 1600, 1800, 2000, 2200, 25000
```

| Medida | Resultado | ¿Representa el sueldo típico? |
|---|---|---|
| Media | 35 700 / 7 = **5100 €** | No: solo una persona cobra más de 2200 € |
| Mediana | Valor central (4.º) = **1800 €** | Sí |
| Moda | **1600 €** | Es el más repetido, pero no el centro |

En Pandas:

```python
sueldos = pd.Series([1500, 1600, 1600, 1800, 2000, 2200, 25000])
sueldos.mean()     # 5100.0
sueldos.median()   # 1800.0
sueldos.mode()     # [1600]
```

!!! tip "Consejo"
    Cuando la media y la mediana se separan mucho, es una señal de que la distribución es **asimétrica** o hay **valores atípicos**. Comparar ambas es una de las primeras cosas que debes hacer al explorar un dataset.

### 2.5 Media ponderada

A veces no todos los valores pesan lo mismo. La media ponderada multiplica cada valor por su peso:

$$\bar{x}_w = \frac{\sum w_i \, x_i}{\sum w_i}$$

Ejemplo: una tienda vende 100 unidades de un producto de 10 € y 1 unidad de uno de 100 €.

- Media simple de precios: (10 + 100) / 2 = **55 €**
- Precio medio por unidad vendida (ponderado por unidades): (100·10 + 1·100) / 101 = **10,89 €**

La segunda es la que responde a "¿cuánto paga de media un cliente por cada unidad?".

!!! warning "Aviso"
    En Big Data es muy frecuente calcular, sin darse cuenta, una media simple de valores que ya eran medias (por ejemplo, la media de las medias diarias). Eso es una media **no ponderada** y suele dar un resultado incorrecto. Lo verás en el apartado 5.

---

## 3. Medidas de dispersión y de posición

Dos conjuntos pueden tener la misma media y ser muy distintos:

```text
A: 48, 50, 52       → media 50
B:  0, 50, 100      → media 50
```

Las medidas de dispersión cuantifican esa diferencia.

### 3.1 Rango

Diferencia entre el máximo y el mínimo. Es fácil de calcular, pero depende solo de dos valores, así que un atípico lo dispara.

**Ejemplo.** Con los sueldos del apartado 2.4:

- Con todos los datos: 25 000 − 1500 = **23 500 €**.
- Sin el sueldo de 25 000 €: 2200 − 1500 = **700 €**.

Un solo valor multiplica el rango por más de 30. En Pandas: `sueldos.max() - sueldos.min()`.

### 3.2 Varianza y desviación típica

**¿Para qué sirven?** Para medir **cuánto se alejan los datos de la media**, es decir, si la media es un buen resumen o no. Si la desviación típica es pequeña, casi todos los datos están cerca de la media y podemos fiarnos de ella; si es grande, los datos están muy repartidos y la media por sí sola dice poco.

!!! example "Una idea intuitiva"
    Dos repartidores tardan de media 30 minutos en entregar un pedido. El primero tarda siempre entre 28 y 32 minutos; el segundo, a veces 10 y a veces 50. La media es la misma, pero ¿de cuál te fiarías para prometer una hora de entrega al cliente? La desviación típica es el número que mide esa diferencia: la del primero será pequeña y la del segundo, grande.

**Cómo se calcula, paso a paso:**

1. Se calcula la media.
2. Se mira cuánto se aleja cada dato de la media (su **desviación**).
3. Se elevan esas distancias al cuadrado. Así las negativas no anulan a las positivas y las distancias grandes pesan más.
4. Se hace la «media» de esos cuadrados: eso es la **varianza**.
5. Como la varianza está en unidades al cuadrado (minutos², €²), que no se pueden interpretar, se hace la raíz cuadrada: eso es la **desviación típica**, que vuelve a estar en minutos o euros.

La **varianza** es la media de las distancias al cuadrado respecto a la media. Con datos de una muestra se divide entre *n − 1*:

$$s^2 = \frac{\sum (x_i - \bar{x})^2}{n - 1}$$

La **desviación típica** es su raíz cuadrada, $s = \sqrt{s^2}$, y tiene la ventaja de estar en **las mismas unidades que los datos**.

**Ejemplo.** Cálculo paso a paso con los conjuntos A y B (los dos tienen media 50):

| | Datos | Desviaciones $x_i - \bar{x}$ | Cuadrados | Suma | $s^2 = \text{suma} / (n-1)$ | $s$ |
|---|---|---|---|---|---|---|
| A | 48, 50, 52 | −2, 0, 2 | 4, 0, 4 | 8 | 8 / 2 = **4** | **2** |
| B | 0, 50, 100 | −50, 0, 50 | 2500, 0, 2500 | 5000 | 5000 / 2 = **2500** | **50** |

Las dos tienen la misma media, pero los datos de B están, de media, 25 veces más lejos de ella.

**Cómo se interpreta.** La desviación típica se lee como «los datos se alejan de la media, más o menos, en *s* unidades»: en A, unos 2 puntos arriba o abajo de 50; en B, unos 50. Por eso decimos que A es un conjunto **homogéneo** y B uno **disperso**.

**¿Dónde se usa en Big Data?**

- **Detectar anomalías:** si la temperatura de un sensor suele variar ±2 °C y de repente se aleja 10 °C de su media, algo pasa.
- **Control de calidad:** un proceso es estable cuando su desviación típica es pequeña (tiempos de respuesta, peso de un producto envasado...).
- **Machine Learning:** para *normalizar* variables antes de entrenar un modelo se resta la media y se divide entre la desviación típica.

En Pandas: `s.var()` y `s.std()`.

### 3.3 Coeficiente de variación

Permite comparar la dispersión de variables con escalas muy diferentes (precios de libros frente a precios de ordenadores):

$$CV = \frac{s}{\bar{x}} \cdot 100\ \%$$

Un CV del 20 % indica datos bastante homogéneos; uno del 200 %, muy dispersos. Por ejemplo, unos libros con media 20 € y desviación 4 € (CV = 20 %) son relativamente **más dispersos** que unos ordenadores con media 800 € y desviación 80 € (CV = 10 %). En Pandas: `s.std() / s.mean() * 100`.

### 3.4 Cuartiles, percentiles y rango intercuartílico

Los **cuantiles** dividen los datos ordenados en partes iguales:

| Medida | Significado |
|---|---|
| Q1 (percentil 25) | El 25 % de los datos está por debajo |
| Q2 (percentil 50) | Es la **mediana** |
| Q3 (percentil 75) | El 75 % de los datos está por debajo |
| Percentil *p* | El *p* % de los datos está por debajo |

El **rango intercuartílico** es $IQR = Q3 - Q1$ y mide la dispersión del 50 % central de los datos, sin dejarse afectar por los extremos.

**Ejemplo.** Minutos que tardan en prepararse 9 pedidos, ya ordenados:

```text
posición:  1   2   3   4   5   6   7   8   9
minutos:  12  15  15  18  20  22  25  30  95
                ↑           ↑           ↑
               Q1          Q2          Q3
```

- **Q2** (mediana): el valor central, en la posición 5 → **20 min**.
- **Q1**: el valor que deja un 25 % por debajo → **15 min**.
- **Q3**: el valor que deja un 75 % por debajo → **25 min**.
- **IQR** = 25 − 15 = **10 min**: la mitad central de los pedidos se prepara en un margen de 10 minutos.

El pedido de 95 minutos no altera ni Q1 ni Q3. Sí altera la media (28 min) y el rango (83 min).

```python
prep = pd.Series([12, 15, 15, 18, 20, 22, 25, 30, 95])
q1, q3 = prep.quantile([0.25, 0.75])   # 15.0, 25.0
iqr = q3 - q1                          # 10.0
prep.describe()                        # count, mean, std, min, 25%, 50%, 75%, max
```

!!! tip "Consejo"
    `describe()` es la forma más rápida de ver de un vistazo la tendencia central, la dispersión y los cuartiles de una columna numérica.

!!! info "Información"
    Existen varios métodos para calcular cuantiles cuando el valor cae entre dos datos. Pandas usa interpolación lineal por defecto; otras herramientas pueden usar otro método y dar resultados ligeramente distintos.

### 3.5 Percentiles en la práctica: tiempos de respuesta

En monitorización de sistemas casi nunca se usa la media del tiempo de respuesta. Se usan el **p95** y el **p99**:

- *"El p99 es 800 ms"* significa que el 99 % de las peticiones tarda 800 ms o menos.
- La media puede ser 300 ms y, aun así, uno de cada cien usuarios esperar 8 segundos. La media lo esconde; el p99 lo muestra.

**Ejemplo.** Tiempos de respuesta de 10 peticiones (ms), ordenados:

```text
110, 120, 125, 130, 140, 150, 160, 180, 900, 2500
```

| Medida | Valor | Lectura |
|---|---|---|
| Media | 451,5 ms | Ningún usuario ha tardado eso: 8 tardan mucho menos y 2 mucho más |
| Mediana (p50) | 145 ms | La petición "normal" |
| p90 | 1060 ms | El 10 % más lento supera el segundo |
| p95 | 1780 ms | |

```python
tiempos = pd.Series([110, 120, 125, 130, 140, 150, 160, 180, 900, 2500])
tiempos.quantile([0.5, 0.90, 0.95])   # 145.0, 1060.0, 1780.0
```

Con solo 10 datos, el p90 y el p95 caen entre el 9.º y el 10.º valor, y Pandas los interpola (recuerda el aviso del apartado 3.4). En un sistema real se calculan sobre miles o millones de peticiones, y ahí el percentil sí describe bien la cola.

Por eso los acuerdos de nivel de servicio (SLA) suelen expresarse en percentiles.

### 3.6 Valores atípicos (outliers)

Un **valor atípico** es un dato muy alejado del resto. Puede ser un error (una temperatura de 900 °C en un sensor de interior) o un dato real pero excepcional (un pedido de empresa entre miles de pedidos particulares).

Un criterio habitual para detectarlos es la **regla del IQR**: se considera atípico todo valor fuera del intervalo

$$[\,Q1 - 1{,}5 \cdot IQR,\ \ Q3 + 1{,}5 \cdot IQR\,]$$

**Ejemplo.** Con los tiempos de preparación del apartado 3.4 (Q1 = 15, Q3 = 25, IQR = 10):

- Límite inferior: 15 − 1,5 · 10 = **0 min**
- Límite superior: 25 + 1,5 · 10 = **40 min**

Todos los pedidos están dentro de [0, 40] salvo el de **95 min**, que se marca como atípico. Antes de hacer nada con él habría que preguntar: ¿es un error de registro o un pedido que de verdad se retrasó?

```python
lim_inf = q1 - 1.5 * iqr
lim_sup = q3 + 1.5 * iqr
atipicos = prep[(prep < lim_inf) | (prep > lim_sup)]   # [95]
```

!!! danger "Importante"
    Detectar un atípico **no significa borrarlo**. Primero hay que averiguar si es un error o un dato real. Eliminar datos reales porque "estropean la media" falsea el análisis. Si los datos son reales, a menudo la solución correcta es **usar la mediana** o analizar ese grupo por separado.

!!! warning "Aviso"
    La regla del IQR supone una distribución más o menos simétrica. En datos muy asimétricos (importes de compra, tiempos de respuesta, ingresos) marcará como atípicos muchos valores perfectamente normales. Úsala como pista, no como veredicto.

### 3.7 Forma de la distribución

Un **histograma** (gráfico de barras de frecuencias por intervalos) muestra cómo se reparten los datos. Basta con comparar media y mediana:

- **Simétrica**: media ≈ mediana (alturas de personas).
- **Asimétrica a la derecha** o con **cola larga**: media > mediana (importes de compra, tiempos de respuesta, sueldos). Es la situación más habitual en datos de negocio y de sistemas.

```python
tiempos.plot.hist(bins=20)   # histograma (necesita matplotlib)
```

---

## 4. Funciones de agregación

### 4.1 Las funciones básicas

| Función | Qué devuelve | SQL | Pandas |
|---|---|---|---|
| Conteo de filas | Número de filas | `COUNT(*)` | `len(df)` / `.size()` |
| Conteo de valores | Número de valores no nulos | `COUNT(col)` | `.count()` |
| Distintos | Número de valores diferentes | `COUNT(DISTINCT col)` | `.nunique()` |
| Suma | Total | `SUM(col)` | `.sum()` |
| Media | Media aritmética | `AVG(col)` | `.mean()` |
| Mínimo / máximo | Extremos | `MIN(col)` / `MAX(col)` | `.min()` / `.max()` |

!!! danger "Importante"
    Las funciones de agregación **ignoran los nulos** (excepto `COUNT(*)`). Si una columna tiene muchos nulos, `AVG(col)` es la media de los valores *conocidos*, no de todas las filas. Revisa siempre cuántos nulos hay antes de interpretar un agregado.

**Ejemplo.** Tabla de 5 pedidos en la que dos no tienen importe registrado:

| pedido_id | importe | metodo_pago |
|---|---|---|
| 1 | 100 | Tarjeta |
| 2 | *NULL* | Bizum |
| 3 | 50 | Tarjeta |
| 4 | *NULL* | *NULL* |
| 5 | 150 | Tarjeta |

| Consulta | Resultado | Comentario |
|---|---|---|
| `COUNT(*)` | 5 | Cuenta filas |
| `COUNT(importe)` | 3 | Solo importes conocidos |
| `SUM(importe)` | 300 | |
| `AVG(importe)` | 300 / 3 = **100** | Media de los conocidos |
| `SUM(importe) / COUNT(*)` | 300 / 5 = **60** | Trata los nulos como 0: es otra pregunta |
| `COUNT(metodo_pago)` | 4 | |
| `COUNT(DISTINCT metodo_pago)` | 2 | Tarjeta y Bizum; el nulo no cuenta |

Ninguna de las dos medias es "la buena" en general: 100 es el importe medio de los pedidos con importe; 60 sería correcto solo si los nulos significaran de verdad "0 €".

```python
df = pd.DataFrame({
    "pedido_id":   [1, 2, 3, 4, 5],
    "importe":     [100, None, 50, None, 150],
    "metodo_pago": ["Tarjeta", "Bizum", "Tarjeta", None, "Tarjeta"],
})
len(df)                      # 5   ≈ COUNT(*)
df["importe"].count()        # 3   ≈ COUNT(importe)
df["importe"].mean()         # 100.0
df["metodo_pago"].nunique()  # 2   ≈ COUNT(DISTINCT metodo_pago)
df.isna().sum()              # nulos por columna
```

### 4.2 Agrupar: GROUP BY

Agregar todo el dataset da un único número. Lo interesante suele ser agregar **por grupos**: por tienda, por día, por categoría.

```sql
SELECT tienda,
       COUNT(*)          AS pedidos,
       SUM(importe)      AS facturacion,
       AVG(importe)      AS ticket_medio
FROM pedidos
GROUP BY tienda
ORDER BY facturacion DESC;
```

El equivalente en Pandas:

```python
(df.groupby("tienda")
   .agg(pedidos=("pedido_id", "count"),
        facturacion=("importe", "sum"),
        ticket_medio=("importe", "mean"))
   .sort_values("facturacion", ascending=False))
```

### 4.3 Filtrar grupos: HAVING

`WHERE` filtra **filas antes** de agrupar; `HAVING` filtra **grupos después** de agregar.

```sql
SELECT categoria, AVG(importe) AS ticket_medio
FROM pedidos
GROUP BY categoria
HAVING AVG(importe) > 100;
```

En Pandas no existe una palabra especial: se agrega y luego se filtra el resultado.

```python
res = df.groupby("categoria").agg(ticket_medio=("importe", "mean"))
res[res["ticket_medio"] > 100]
```

!!! note "Recuerda"
    Orden lógico de una consulta SQL: `FROM` → `WHERE` → `GROUP BY` → `HAVING` → `SELECT` → `ORDER BY`. Por eso en `WHERE` no puedes usar un agregado: todavía no se ha calculado.

---

## 5. Agregados en sistemas distribuidos

Este es el concepto central de la unidad.

En un sistema de Big Data (Hadoop, Spark, una base de datos distribuida) los datos no están en una sola máquina: están **repartidos en particiones** entre muchos **nodos**. Para calcular un agregado global, cada nodo calcula un **resultado parcial** con sus datos y después esos parciales se **combinan**.

La pregunta clave es: **¿se puede obtener el resultado global combinando solo los parciales?** Según la respuesta, las agregaciones se clasifican en tres tipos (clasificación propuesta por Gray et al. en 1997, en el trabajo que introdujo el operador *Data Cube*).

### 5.1 Ejemplo de partida

Imagina estos datos repartidos en dos nodos:

```text
Nodo A: 2, 4, 6
Nodo B: 10, 20, 30, 40, 50
Global: 2, 4, 6, 10, 20, 30, 40, 50   (8 valores)
```

### 5.2 Distributivas

El resultado global se obtiene aplicando una operación sencilla a los parciales.

| Función | Parcial A | Parcial B | Combinación | Global |
|---|---|---|---|---|
| `SUM` | 12 | 150 | 12 + 150 | **162** ✔ |
| `COUNT` | 3 | 5 | 3 + 5 | **8** ✔ |
| `MAX` | 6 | 50 | max(6, 50) | **50** ✔ |
| `MIN` | 2 | 10 | min(2, 10) | **2** ✔ |

Son las ideales: cada nodo envía **un único número**, sin importar cuántos datos tenga.

### 5.3 Algebraicas

No se pueden combinar directamente, pero sí a partir de **un número fijo de agregados distributivos**.

La **media** es el ejemplo típico. Si cada nodo envía su media:

- Media A = 4, media B = 30
- Media de las medias = (4 + 30) / 2 = **17** ✘ (incorrecto)

La media real es 162 / 8 = **20,25**. El error se produce porque el nodo B tiene más datos y debería pesar más.

La solución es que cada nodo envíe **(suma, conteo)** en lugar de su media:

$$\bar{x} = \frac{\text{suma}_A + \text{suma}_B}{\text{conteo}_A + \text{conteo}_B} = \frac{12 + 150}{3 + 5} = 20{,}25$$

La **varianza** (y por tanto la desviación típica) también es algebraica: basta con que cada nodo envíe **(suma, conteo, suma de cuadrados)**, tres agregados distributivos a partir de los cuales se puede reconstruir la varianza global. Promediar las varianzas de cada nodo, en cambio, da un resultado incorrecto.

### 5.4 Holísticas

No existe un número fijo de valores parciales que permita reconstruir el resultado exacto. Para obtenerlo hay que **reunir (o reordenar) todos los datos**.

Ejemplos: **mediana**, **moda**, **percentiles exactos** y **conteo exacto de distintos**.

Comprobación con el ejemplo:

- Mediana A = 4, mediana B = 30
- "Mediana de medianas" = (4 + 30) / 2 = **17** ✘
- Mediana real = (10 + 20) / 2 = **15**

Con la moda pasa lo mismo: un valor puede no ser el más frecuente en ningún nodo y, sin embargo, ser el más frecuente en total.

### 5.5 Resumen y consecuencias

| Tipo | Qué envía cada nodo | Coste de red | Ejemplos |
|---|---|---|---|
| Distributiva | 1 valor | Mínimo | SUM, COUNT, MIN, MAX |
| Algebraica | Unos pocos valores fijos | Mínimo | AVG, VAR, STDDEV |
| Holística | Todos los datos (o casi) | Muy alto | Mediana, moda, percentiles, COUNT DISTINCT |

Consecuencias prácticas:

- Por eso en MapReduce y Spark las sumas y conteos se pueden **precombinar** en cada nodo (los llamados *combiners*) y la red apenas se usa.
- Por eso calcular la mediana exacta de miles de millones de filas es **lento y caro**.
- Por eso **nunca debes promediar medias ni calcular la mediana de medianas** de tablas ya agregadas. Si vas a necesitar la media después, guarda la suma y el conteo.

!!! tip "Consejo"
    Regla práctica al diseñar tablas preagregadas: guarda siempre **sumas y conteos**, no medias. A partir de sumas y conteos puedes recalcular la media de cualquier combinación de grupos; a partir de medias, no.

---

## 6. Pincelada: agregados aproximados

Si las agregaciones holísticas son tan caras, ¿cómo calcula un sistema de Big Data la mediana o el número de usuarios distintos de miles de millones de registros? En muchos casos, **no lo hace de forma exacta**: acepta un resultado aproximado, con un error pequeño y conocido, a cambio de usar muchísima menos memoria y tiempo.

| Problema | Técnica aproximada | Idea |
|---|---|---|
| Contar elementos distintos | **HyperLogLog** | Estima la cantidad de distintos con unos pocos KB de memoria |
| Percentiles y mediana | **Resúmenes de cuantiles** (Greenwald-Khanna, t-digest) | Guardan un resumen compacto de la distribución que sí se puede combinar entre nodos |
| Elementos más frecuentes | **Count-Min Sketch** | Estima frecuencias con una tabla de contadores de tamaño fijo |
| Casi cualquier agregado | **Muestreo** | Calcular sobre una muestra aleatoria |

Por ejemplo, Spark ofrece `approx_count_distinct` y `percentile_approx`, que se basan en estas ideas.

!!! info "Información"
    Lo importante en esta unidad no es el funcionamiento interno de estos algoritmos, sino la idea: **en Big Data se negocia entre precisión, memoria y tiempo**. Lo veremos con más detalle en unidades posteriores.

---

## 7. Pincelada: agregación en el tiempo

Muchos datos de Big Data llegan como **flujo continuo** (*streaming*): clics, lecturas de sensores, transacciones. No tiene sentido esperar a "tener todos los datos" para agregarlos, porque nunca dejan de llegar. Se agregan por **ventanas temporales**:

| Tipo de ventana | Funcionamiento | Ejemplo |
|---|---|---|
| Fija (*tumbling*) | Intervalos consecutivos que no se solapan | Ventas de cada hora: 10:00-11:00, 11:00-12:00... |
| Deslizante (*sliding*) | Intervalos que se solapan y avanzan un paso | Media de los últimos 10 minutos, recalculada cada minuto |
| De sesión | Se cierra tras un tiempo de inactividad | Actividad de un usuario hasta que pasa 30 min sin hacer nada |

Una **media móvil** es una agregación con ventana deslizante: suaviza las variaciones diarias y deja ver la tendencia. En Pandas:

```python
ventas_diarias = df.set_index("fecha_hora").resample("D")["importe"].sum()   # ventana fija diaria
media_movil_7d = ventas_diarias.rolling(7).mean()                           # ventana deslizante de 7 días
```

En la ventana **de sesión**, a diferencia de las otras dos, la duración no se conoce de antemano: depende de los datos (cada vez que pasan más de 30 minutos sin actividad empieza una sesión nueva). En Spark Structured Streaming existe `F.session_window("hora", "30 minutes")`.

---

## 8. Errores habituales

| Error | Por qué está mal | Qué hacer |
|---|---|---|
| Usar la media con datos muy asimétricos | Los valores extremos la desplazan | Comparar con la mediana e informar de ambas |
| Promediar medias de grupos de distinto tamaño | Media no ponderada | Combinar sumas y conteos |
| Calcular la mediana de medianas | La mediana es holística | Recalcular sobre los datos o usar un resumen de cuantiles |
| Confundir `COUNT(*)` con `COUNT(col)` | El segundo ignora nulos | Elegir conscientemente cuál se necesita |
| Borrar outliers sin investigarlos | Se eliminan datos reales | Verificar si son errores o casos reales |
| No eliminar duplicados antes de agregar | Sumas y conteos inflados | Revisar calidad del dato antes de agregar |
| Informar de la media de tiempos de respuesta | Oculta a los usuarios afectados | Usar p95 / p99 |

---

## 9. Resumen

- **Media, mediana y moda** describen el valor típico. La media es sensible a extremos; la mediana es robusta; la moda sirve también para categorías.
- La **dispersión** se mide con rango, varianza, desviación típica, CV, cuartiles e IQR. Los **percentiles** (p95, p99) son clave en monitorización.
- Las **funciones de agregación** (`COUNT`, `SUM`, `AVG`, `MIN`, `MAX`...) se combinan con **`GROUP BY`** y **`HAVING`**, en SQL igual que en Pandas.
- En sistemas distribuidos, las agregaciones son **distributivas** (se combinan directamente), **algebraicas** (se combinan con unos pocos valores auxiliares) u **holísticas** (necesitan todos los datos). Esto determina su coste.
- Cuando el coste es demasiado alto, se usan **agregados aproximados**; cuando los datos no paran de llegar, **ventanas temporales**.

---

## Lecturas y referencias

- Documentación oficial de Pandas: *Group by: split-apply-combine* — https://pandas.pydata.org/docs/user_guide/groupby.html
- Documentación oficial de PostgreSQL: *Aggregate Functions* — https://www.postgresql.org/docs/current/functions-aggregate.html
- Documentación de Apache Spark: funciones SQL integradas (`approx_count_distinct`, `percentile_approx`) — https://spark.apache.org/docs/latest/api/sql/
- Gray, J. et al. (1997). *Data Cube: A Relational Aggregation Operator Generalizing Group-By, Cross-Tab, and Sub-Totals*. Data Mining and Knowledge Discovery, 1(1).

---

