# Manual de Big Data: conceptos fundamentales

**Módulo:** MP5074 – Sistemas de Big Data · Curso 2026-27

> Este manual recoge las ideas de base que se usan en todo el módulo: qué convierte un problema en «de Big Data», cómo se organizan sus herramientas, qué arquitecturas existen, por qué la complejidad de un algoritmo obliga a repartir el trabajo y cómo se valoran el coste y la calidad de un sistema.

---

## Índice

1. [¿Qué es Big Data?](#1-qué-es-big-data)
2. [Las cinco V](#2-las-cinco-v)
3. [Tipos de datos según su estructura](#3-tipos-de-datos-según-su-estructura)
4. [Escalar: hacia arriba o hacia los lados](#4-escalar-hacia-arriba-o-hacia-los-lados)
5. [El ecosistema Big Data](#5-el-ecosistema-big-data)
6. [Procesamiento por lotes y en flujo: arquitecturas Lambda y Kappa](#6-procesamiento-por-lotes-y-en-flujo-arquitecturas-lambda-y-kappa)
7. [Perfiles profesionales](#7-perfiles-profesionales)
8. [Fundamentos: matemática discreta y lógica algorítmica](#8-fundamentos-matemática-discreta-y-lógica-algorítmica)
9. [Complejidad computacional y notación O](#9-complejidad-computacional-y-notación-o)
10. [Costes y calidad de un sistema Big Data](#10-costes-y-calidad-de-un-sistema-big-data)
11. [Transferencia de datos por la red](#11-transferencia-de-datos-por-la-red)
12. [Planificar un proyecto de datos](#12-planificar-un-proyecto-de-datos)
13. [Conceptos clave](#13-conceptos-clave)
14. [Errores de concepto frecuentes](#14-errores-de-concepto-frecuentes)
15. [Chuleta rápida](#15-chuleta-rápida)
16. [Referencias](#16-referencias)
17. [Ejemplo resuelto: tiques duplicados](#17-ejemplo-resuelto-tiques-duplicados)
18. [Ejemplo resuelto: subir el histórico de tiques a la nube](#18-ejemplo-resuelto-subir-el-histórico-de-tiques-a-la-nube)

---

## 1. ¿Qué es Big Data?

Durante décadas, los datos de una empresa cabían en una base de datos relacional en un único servidor. Hoy, un móvil genera datos de ubicación cada pocos segundos, una tienda online registra cada clic y un aerogenerador manda lecturas de sus sensores varias veces por segundo. Esta es la **revolución digital**: casi toda actividad deja un rastro de datos.

El problema no es solo que haya *muchos* datos. El problema es que las herramientas clásicas dejan de servir: un solo ordenador no puede guardarlos, no puede procesarlos a tiempo o no sabe interpretar su formato. **Big Data** es el conjunto de técnicas y tecnologías que aparecen para resolver ese problema, y también la nueva forma de trabajar que traen consigo: tomar decisiones a partir de los datos y no de la intuición.

**Definición de trabajo**

> Un problema es de Big Data cuando el **volumen**, la **velocidad** o la **variedad** de los datos superan la capacidad de las herramientas tradicionales para almacenarlos, procesarlos y analizarlos en un tiempo razonable.

---

## 2. Las cinco V

Las «V» son la forma clásica de describir qué hace difíciles a estos datos:

| V | Pregunta que responde | Ejemplo |
|---|---|---|
| **Volumen** | ¿Cuántos datos hay? | Un operador de telefonía guarda petabytes de registros de llamadas. |
| **Velocidad** | ¿A qué ritmo llegan y con qué rapidez hay que responder? | Detectar un pago fraudulento con tarjeta en menos de un segundo. |
| **Variedad** | ¿Qué forma tienen? | Tablas, JSON de una API, textos de redes sociales, imágenes, audio. |
| **Veracidad** | ¿Podemos fiarnos de ellos? | Sensores que fallan, encuestas con respuestas falsas, duplicados. |
| **Valor** | ¿Qué ganamos al analizarlos? | Reducir el stock parado de un almacén un 15 %. |

- Las tres primeras describen la **dificultad técnica**.
- Las dos últimas recuerdan que almacenar datos no sirve de nada si no son fiables y no aportan algo.

Algunos autores añaden más V (variabilidad, visualización…), pero estas cinco son las que se usan en este módulo.

---

## 3. Tipos de datos según su estructura

| Tipo | Descripción | Ejemplos |
|---|---|---|
| **Estructurados** | Filas y columnas con un esquema fijo | Tabla SQL, CSV |
| **Semiestructurados** | Tienen etiquetas o claves, pero no un esquema rígido | JSON, XML |
| **No estructurados** | Sin organización interna predefinida | Texto libre, imágenes, audio, vídeo |

Los datos no estructurados son la mayor parte de los que se generan hoy. Lo veremos en detalle más adelante.

---

## 4. Escalar: hacia arriba o hacia los lados

Cuando un servidor se queda corto hay dos salidas:

| Estrategia | En qué consiste | Ventajas | Inconvenientes |
|---|---|---|---|
| **Escalado vertical** (*scale up*) | Comprar una máquina más potente | Sencillo, no cambia el software | Techo físico; el precio sube mucho más deprisa que la potencia |
| **Escalado horizontal** (*scale out*) | Repartir datos y trabajo entre muchas máquinas normales (un **clúster**) | Sin techo claro; tolera fallos de máquinas sueltas | Hay que repartir, coordinar y recuperarse de fallos |

En un clúster de mil nodos alguna máquina se rompe cada día, así que el software tiene que contar con ello.

Big Data apuesta por el escalado horizontal. La idea clave, popularizada por Hadoop, es:

> **Llevar el cálculo a donde están los datos**, en lugar de mover los datos hasta el cálculo, porque mover terabytes por la red es lo más lento de todo.

---

## 5. El ecosistema Big Data

Las herramientas se entienden mejor si se agrupan por la función que cumplen dentro del recorrido de los datos:

| Fase | Qué hace | Herramientas típicas |
|---|---|---|
| **Ingesta** | Traer los datos desde su origen | APIs, *scraping*, Flume, Sqoop, Kafka, NiFi |
| **Almacenamiento** | Guardarlos de forma escalable | HDFS, almacenamiento de objetos (S3), MongoDB, Cassandra |
| **Procesamiento** | Limpiar, transformar, agregar | MapReduce, Spark, Flink, pandas (en pequeño) |
| **Análisis** | Extraer conocimiento | SQL, Python, aprendizaje automático |
| **Visualización** | Comunicar los resultados | Matplotlib, Power BI, Grafana, cuadros de mando |

Este módulo recorre ese mismo camino: primero obtener los datos (ingesta), después almacenarlos y analizarlos, integrarlos y, por último, visualizarlos.

---

## 6. Procesamiento por lotes y en flujo: arquitecturas Lambda y Kappa

### 6.1 Dos formas de procesar

| Modo | Cómo funciona | Ventajas | Inconvenientes |
|---|---|---|---|
| **Por lotes** (*batch*) | Se acumulan los datos y se procesan todos juntos de vez en cuando (p. ej., cada noche) | Preciso y completo | Lento: el resultado llega tarde |
| **En flujo** (*streaming*) | Cada dato se procesa en cuanto llega | Rápido | Más complejo y a veces menos preciso |

### 6.2 Arquitectura Lambda

Combina los dos modos en tres capas:

1. **Capa batch:** guarda todos los datos en bruto y recalcula periódicamente vistas completas y exactas.
2. **Capa de velocidad:** procesa en tiempo real solo lo que ha llegado desde el último lote.
3. **Capa de servicio:** une los dos resultados para responder a las consultas.

**Inconveniente:** hay que escribir y mantener la misma lógica dos veces (una para el lote y otra para el flujo).

### 6.3 Arquitectura Kappa

Simplifica: lo trata todo como un flujo. Los datos se guardan en un registro de eventos (como Kafka) y, si hay que recalcular algo, se vuelve a reproducir el flujo desde el principio.

- **Ventaja:** una sola lógica que mantener.
- **Inconveniente:** depende mucho del sistema de mensajería.

### 6.4 Comparación visual

```mermaid
flowchart LR
    subgraph Lambda
      A[Datos] --> B[Capa batch]
      A --> C[Capa de velocidad]
      B --> D[Capa de servicio]
      C --> D
    end
    subgraph Kappa
      E[Datos] --> F[Registro de eventos] --> G[Procesamiento en flujo] --> H[Servicio]
    end
```

---

## 7. Perfiles profesionales

| Perfil | Qué hace |
|---|---|
| **Ingeniero/a de datos** | Construye y mantiene las tuberías (*pipelines*) que traen, limpian y guardan los datos |
| **Analista de datos** | Consulta los datos y prepara informes y cuadros de mando para el negocio |
| **Científico/a de datos** | Construye modelos estadísticos y de aprendizaje automático |
| **Administrador/a de sistemas Big Data** | Despliega y vigila los clústeres |
| **Responsable de gobierno del dato** | Vela por la calidad, la seguridad y el cumplimiento de la normativa |

En la práctica estos perfiles trabajan juntos y se solapan: por eso el currículo habla de **entornos multidisciplinares**.

---

## 8. Fundamentos: matemática discreta y lógica algorítmica

Para razonar sobre datos hace falta un poco de matemática discreta (la que trabaja con valores contables):

| Área | Para qué sirve en datos | Ejemplo |
|---|---|---|
| **Conjuntos** | Unión, intersección y diferencia al combinar tablas | `merge` en pandas, `JOIN` en SQL |
| **Lógica booleana** | `AND`, `OR` y `NOT` son la base de cualquier filtro | `df[(df.precio > 100) & (df.region == "Galicia")]` |
| **Combinatoria** | Contar combinaciones posibles | Comparar todos los clientes con todos: n·(n−1)/2 parejas |
| **Grafos** | Modelar relaciones con nodos y aristas | Redes sociales, rutas, dependencias entre tareas |

Un **algoritmo** es una secuencia finita y ordenada de pasos para resolver un problema. Lo que nos interesa en Big Data no es solo que funcione, sino **cuánto tarda cuando los datos crecen**.

---

## 9. Complejidad computacional y notación O

La **notación O grande** describe cómo crece el número de operaciones de un algoritmo en función del tamaño de la entrada *n*, ignorando constantes. No mide segundos, mide **tendencia**.

| Complejidad | Nombre | Ejemplo |
|---|---|---|
| O(1) | Constante | Acceder a un elemento de un diccionario por su clave |
| O(log n) | Logarítmica | Búsqueda binaria en una lista ordenada |
| O(n) | Lineal | Recorrer un fichero una vez para sumar una columna |
| O(n log n) | Casi lineal | Ordenar (los buenos algoritmos de ordenación) |
| O(n²) | Cuadrática | Comparar cada registro con todos los demás |
| O(2ⁿ) | Exponencial | Probar todos los subconjuntos posibles |

Con pocos datos, todas parecen rápidas. Con miles de millones, solo las lineales o casi lineales son viables, y aun así puede que un solo ordenador no baste.

> Por eso en Big Data se buscan algoritmos que se puedan **dividir**: si un trabajo O(n) se reparte entre *k* máquinas, cada una hace n/k.

---

## 10. Costes y calidad de un sistema Big Data

### 10.1 Costes

| Tipo de coste | Incluye |
|---|---|
| **Infraestructura** | Servidores propios (*on-premise*) o en la nube (pago por uso), almacenamiento, red |
| **Personal** | Perfiles especializados y formación |
| **Software y licencias** | Muchas herramientas son libres, pero el soporte empresarial no |
| **Operación** | Energía, mantenimiento, seguridad, copias |

### 10.2 Criterios de calidad

| Criterio | Pregunta |
|---|---|
| **Disponibilidad** | ¿Está siempre funcionando? |
| **Escalabilidad** | ¿Aguanta el doble de datos? |
| **Rendimiento** | ¿Responde a tiempo? |
| **Tolerancia a fallos** | ¿Sigue funcionando si cae una máquina? |
| **Seguridad** | ¿Están protegidos los datos y los accesos? |
| **Calidad del dato** | ¿Son los datos exactos, completos, actuales y coherentes? |

> **Regla práctica:** no todo problema con muchos datos necesita un clúster. Si los datos caben en la memoria de un portátil, pandas suele ser más barato y rápido que Spark.

---

## 11. Transferencia de datos por la red

### 11.1 Por qué hay que saber calcularlo

En la sección 4 vimos que **mover datos por la red es lo más lento de todo**. Antes de diseñar un sistema hay que saber cuánto se tarda en llevar los datos hasta donde se procesan. Pasa al migrar a la nube, al copiar entre centros de datos, al replicar o al hacer copias de seguridad. Un cálculo de dos minutos puede revelar que el plan pide semanas de transferencia.

### 11.2 Bits y bytes: la trampa de las unidades

- La **velocidad** de una red se mide en **bits por segundo**: Mbps, Gbps.
- El **tamaño** de un fichero se mide en **bytes**: MB, GB, TB.
- **1 byte = 8 bits.** Una «fibra de 300 megas» son 300 Mbps = 300 / 8 = **37,5 MB/s**.

Además, hay dos formas de contar los múltiplos:

- **Decimal (SI):** cada escalón multiplica por 1 000. La usan los fabricantes de discos, las redes y la nube.
- **Binaria (IEC):** cada escalón multiplica por 1 024. La usa a menudo el sistema operativo, sobre todo Windows, aunque escriba «GB» cuando en realidad son GiB.

**Tabla de conversión**

| Decimal | Bytes | Binaria | Bytes | 1 unidad decimal = | 1 unidad binaria = | Diferencia |
|---|---|---|---|---|---|---|
| 1 kB (kilobyte) | 10³ | 1 KiB (kibibyte) | 2¹⁰ = 1 024 | 0,977 KiB | 1,024 kB | 2,4 % |
| 1 MB (megabyte) | 10⁶ | 1 MiB (mebibyte) | 2²⁰ = 1 048 576 | 0,954 MiB | 1,049 MB | 4,9 % |
| 1 GB (gigabyte) | 10⁹ | 1 GiB (gibibyte) | 2³⁰ ≈ 1,074·10⁹ | 0,931 GiB | 1,074 GB | 7,4 % |
| 1 TB (terabyte) | 10¹² | 1 TiB (tebibyte) | 2⁴⁰ ≈ 1,100·10¹² | 0,909 TiB | 1,100 TB | 10,0 % |
| 1 PB (petabyte) | 10¹⁵ | 1 PiB (pebibyte) | 2⁵⁰ ≈ 1,126·10¹⁵ | 0,888 PiB | 1,126 PB | 12,6 % |
| 1 EB (exabyte) | 10¹⁸ | 1 EiB (exbibyte) | 2⁶⁰ ≈ 1,153·10¹⁸ | 0,867 EiB | 1,153 EB | 15,3 % |

**Cómo se convierte**

- De decimal a binaria: se divide entre 1 024ⁿ, donde *n* es el escalón (kilo = 1, mega = 2, giga = 3…).
  - Ejemplo: 500 GB = 500·10⁹ / 1 024³ ≈ **465,7 GiB**.
- De binaria a decimal: se multiplica por 1 024ⁿ.
  - Ejemplo: 2 TiB = 2 × 1 024⁴ ≈ **2,2 TB**.

> **Por qué «falta» espacio en el disco:** compras un disco de 1 TB y Windows muestra unos **931 GB**. No falta nada: son 931 GiB, que es lo mismo que 1 TB. Cuanto más grande es la unidad, mayor es la diferencia: con petabytes ya supera el 12 %.

> Para estimar tiempos de transferencia basta con las unidades decimales, porque las velocidades de red siempre van en decimal. Lo que nunca se puede olvidar es **multiplicar por 8** al pasar de bytes a bits.

### 11.3 La fórmula

```text
            tamaño (bytes) × 8
tiempo = ─────────────────────────────────
         velocidad (bits/s) × eficiencia
```

La **eficiencia** suele estar entre **0,7 y 0,8**. Nunca se alcanza la velocidad contratada porque parte se pierde en cabeceras TCP/IP, retransmisiones, cifrado, otros usuarios que comparten la línea o un disco más lento que la red.

### 11.4 Factores que cambian el resultado

| Factor | Qué hay que tener en cuenta |
|---|---|
| **Subida y bajada** | Muchas conexiones (4G, cable, algunas fibras) suben más despacio de lo que bajan. Para enviar datos a la nube cuenta la **subida**. |
| **Ancho de banda y latencia** | El ancho de banda es *cuánto cabe* por segundo (lo ancha que es la tubería). La latencia es *cuánto tarda* en llegar el primer bit (lo larga que es). |
| **Muchos ficheros pequeños** | Cada fichero paga su latencia, su apertura y su confirmación. Un millón de ficheros de 1 MB tarda mucho más que un único fichero de 1 TB → conviene **empaquetar** (tar, zip). |
| **Cuello de botella** | Manda el eslabón más lento: el disco, el wifi o el servidor de destino. |
| **Compresión** | El texto (CSV, JSON, logs) se comprime de 4 a 10 veces. JPG, MP4, ZIP o Parquet ya van comprimidos y apenas se reducen. |
| **Horario** | Si la línea se comparte con el resto de la empresa, quizá solo pueda usarse por la noche. |

### 11.5 Tabla de referencia

Tiempos teóricos (eficiencia 100 %). En la práctica hay que sumar entre un 25 % y un 40 %.

| Tipo de fichero | Tamaño | 4G (20 Mbps) | Fibra doméstica (300 Mbps) | Fibra 1 Gbps | Red de CPD (10 Gbps) |
|---|---|---|---|---|---|
| Foto de móvil (JPG) | 4 MB | 1,6 s | 0,1 s | < 0,1 s | < 0,1 s |
| CSV de ventas mensual | 50 MB | 20 s | 1,3 s | 0,4 s | < 0,1 s |
| Película HD (MP4) | 4 GB | 27 min | 1,8 min | 32 s | 3,2 s |
| Logs de un día de una web grande | 50 GB | 5,6 h | 22 min | 6,7 min | 40 s |
| Copia de seguridad de una base de datos | 500 GB | 2,3 días | 3,7 h | 1,1 h | 6,7 min |
| Data lake de una empresa mediana | 10 TB | 46 días | 3,1 días | 22 h | 2,2 h |
| Datos de una gran plataforma | 1 PB | 12,7 años | 309 días | 93 días | 9,3 días |

> Hasta los gigabytes la red no preocupa. A partir de los **terabytes**, la transferencia pasa a ser una parte importante del diseño.

### 11.6 Cuando la red no basta: mover discos

Andrew Tanenbaum lo resumió así: *«nunca subestimes el ancho de banda de una furgoneta llena de cintas circulando por la autopista»*. Una furgoneta con 100 discos de 20 TB (2 PB) que tarda 10 horas en llegar equivale a una red de 2·10¹⁵ × 8 / 36 000 s ≈ **444 Gbps**. Por eso los grandes proveedores de nube ofrecen dispositivos físicos que se llenan en las instalaciones del cliente y se envían por mensajería.

---

## 12. Planificar un proyecto de datos

Un proyecto de datos sigue unas fases parecidas a las de cualquier proyecto, y conviene fijar objetivos y prioridades desde el principio:

1. **Definir la pregunta de negocio** (qué queremos saber y para qué).
2. **Identificar las fuentes** y comprobar que se pueden usar legalmente.
3. **Obtener y almacenar** los datos.
4. **Limpiar y transformar.** Suele llevarse más de la mitad del tiempo.
5. **Analizar y modelar.**
6. **Comunicar** los resultados (visualización, informe).
7. **Desplegar y mantener.**

Para organizar el tiempo:

- Trocea el trabajo en **tareas pequeñas**.
- **Priorízalas**: lo imprescindible antes que lo deseable.
- **Revisa el avance** a menudo.

Es lo que haremos en el proyecto integrador.

---

## 13. Conceptos clave

| Concepto | Explicación breve |
|---|---|
| Big Data | Datos cuyo volumen, velocidad o variedad superan las herramientas tradicionales. |
| Las cinco V | Volumen, velocidad, variedad, veracidad y valor. |
| Escalado horizontal | Añadir más máquinas al clúster en lugar de una máquina más grande. |
| Clúster | Conjunto de ordenadores que trabajan coordinados como si fueran uno. |
| Procesamiento por lotes | Procesar datos acumulados de una vez, cada cierto tiempo. |
| Procesamiento en flujo | Procesar cada dato en el momento en que llega. |
| Arquitectura Lambda | Combina una capa batch y una capa de velocidad que se unen en una capa de servicio. |
| Arquitectura Kappa | Lo procesa todo como flujo y recalcula reproduciendo el registro de eventos. |
| Notación O | Forma de expresar cómo crece el coste de un algoritmo con el tamaño de los datos. |
| Calidad del dato | Grado en que los datos son exactos, completos, actuales y coherentes. |
| Ancho de banda | Cantidad de datos que una red puede transmitir por segundo; se mide en bits por segundo (Mbps, Gbps). |
| Latencia | Tiempo que tarda en llegar el primer bit de un mensaje; pesa mucho cuando hay muchos ficheros pequeños. |
| Eficiencia de la red | Fracción de la velocidad contratada que se aprovecha de verdad; suele estar entre 0,7 y 0,8. |
| Unidades binarias | Múltiplos de 1 024 (KiB, MiB, GiB, TiB) frente a los decimales de 1 000 (kB, MB, GB, TB). |

---

## 14. Errores de concepto frecuentes

| Idea equivocada | Por qué no es así |
|---|---|
| «Big Data = muchos datos» | El volumen es solo una V: también cuentan la velocidad y la variedad, y sin veracidad ni valor no sirve de nada. |
| «Con más datos siempre necesito Spark» | Si caben en la memoria de una máquina, pandas suele ser más rápido y barato. |
| «Un servidor más potente lo resuelve todo» | El escalado vertical tiene techo y su precio crece mucho más deprisa que la potencia. |
| «La notación O mide el tiempo en segundos» | Mide cómo **crece** el coste con *n*; las constantes y el hardware se ignoran. |
| «Lambda y Kappa son herramientas» | Son **arquitecturas** (formas de organizar el sistema), que se implementan con herramientas como Spark o Kafka. |
| «Lo difícil es analizar» | La limpieza y transformación suelen llevarse más de la mitad del tiempo del proyecto. |
| «Mi fibra de 300 megas descarga a 300 MB/s» | Son 300 **Mbps** (megabits): 300 / 8 = **37,5 MB/s**. |
| «Da igual un fichero grande que muchos pequeños» | Cada fichero paga su sobrecarga (latencia, apertura, confirmación); conviene empaquetarlos. |
| «Si contrato más velocidad, iré a esa velocidad» | Manda el eslabón más lento y la eficiencia real ronda el 70–80 %. |
| «Mi disco de 1 TB solo tiene 931 GB» | Son 931 **GiB**, que equivalen exactamente a 1 TB; el sistema operativo cuenta en binario. |

---

## 15. Chuleta rápida

```text
# ---------- Las cinco V ----------
Volumen · Velocidad · Variedad      → dificultad técnica
Veracidad · Valor                    → ¿fiables? ¿útiles?

# ---------- Escalado ----------
Vertical  (scale up)   → máquina más grande  (techo físico)
Horizontal (scale out) → más máquinas         (clúster)
Principio: llevar el cálculo a los datos

# ---------- Recorrido del dato ----------
Ingesta → Almacenamiento → Procesamiento → Análisis → Visualización

# ---------- Arquitecturas ----------
Lambda → batch + velocidad + servicio  (lógica duplicada)
Kappa  → todo es flujo + registro de eventos (una sola lógica)

# ---------- Complejidad (de mejor a peor) ----------
O(1) < O(log n) < O(n) < O(n log n) < O(n²) < O(2ⁿ)
Repartir un O(n) entre k máquinas → n/k por máquina

# ---------- Transferencia ----------
t = tamaño(B) × 8 / (velocidad(bps) × 0,8)
1 byte = 8 bits   ·   300 Mbps = 37,5 MB/s
Comprimir texto · empaquetar ficheros pequeños · ojo a la subida

# ---------- Unidades ----------
Decimal ×1 000 (kB, MB, GB, TB)  ·  Binaria ×1 024 (KiB, MiB, GiB, TiB)
1 TB = 0,909 TiB   ·   1 TiB = 1,1 TB
Disco de 1 TB → Windows muestra 931 «GB» (en realidad GiB)
```

---

## 16. Referencias

- Apache Hadoop: <https://hadoop.apache.org/>
- Apache Spark: <https://spark.apache.org/>
- Apache Kafka: <https://kafka.apache.org/>
- Apache Flink: <https://flink.apache.org/>
- J. Kreps, *Questioning the Lambda Architecture* (O'Reilly Radar): <https://www.oreilly.com/radar/questioning-the-lambda-architecture/>
- Resumen de complejidades de estructuras de datos y algoritmos: <https://www.bigocheatsheet.com/>

---

## 17. Ejemplo resuelto: tiques duplicados

**Situación.** Una cadena de supermercados tiene **1 000 millones** (10⁹) de tiques de compra y quiere encontrar los tiques duplicados (el mismo tique registrado dos veces por un error de la caja). Un ordenador hace unos **10⁸ operaciones sencillas por segundo**. Se plantean dos algoritmos.

**Algoritmo A: comparar todos con todos.** Para cada tique, se compara con todos los demás.

- Número de comparaciones: n·(n−1)/2 ≈ n²/2 = (10⁹)² / 2 = 5·10¹⁷.
- Tiempo: 5·10¹⁷ / 10⁸ = 5·10⁹ segundos.
- En años: 5·10⁹ / (60·60·24·365) ≈ 5·10⁹ / 3,15·10⁷ ≈ **159 años**.

Complejidad O(n²): inviable.

**Algoritmo B: usar un diccionario (tabla hash).** Se recorre la lista una vez; para cada tique se mira si su identificador ya está en un diccionario (O(1)) y, si no, se añade.

- Operaciones: unas n = 10⁹.
- Tiempo: 10⁹ / 10⁸ = **10 segundos** de cálculo.

Complejidad O(n): el algoritmo es viable. Pero aparece otro límite: si cada entrada del diccionario ocupa unos 100 bytes, harían falta 10⁹ · 100 B = 10¹¹ B = **100 GB de memoria RAM**, más de lo que tiene un portátil (16 GB).

**Algoritmo C: repartir entre 10 máquinas.** Se reparten los tiques según el resto de dividir su identificador entre 10 (así, dos tiques iguales caen siempre en la misma máquina). Cada máquina aplica el algoritmo B a su parte:

- Datos por máquina: 10⁹ / 10 = 10⁸ tiques.
- Memoria por máquina: 10⁸ · 100 B = 10 GB, que **sí cabe**.
- Tiempo por máquina: 10⁸ / 10⁸ = 1 segundo, más el tiempo de repartir los datos por la red.

**Coste.** Supongamos que alquilar en la nube una máquina con 16 GB cuesta 0,20 € por hora y que el trabajo completo, contando la transferencia, tarda 15 minutos:

- 10 máquinas × 0,25 h × 0,20 €/h = **0,50 €** por ejecución.
- Una única máquina con 128 GB podría costar 1,50 €/h; con 0,25 h saldrían **0,375 €**, más barato en este caso, pero con techo: si mañana hay 10 veces más tiques, no existe una máquina 10 veces mayor a un precio razonable, mientras que el clúster solo necesita 100 máquinas.

**Conclusión.** El ejemplo junta las ideas del manual: la **complejidad** descarta el algoritmo A; el **volumen** hace que el algoritmo B no quepa en una máquina; el **escalado horizontal** lo resuelve repartiendo los datos; y el **análisis de costes** muestra que la decisión depende del tamaño actual y del crecimiento esperado.

---

## 18. Ejemplo resuelto: subir el histórico de tiques a la nube

**Situación.** La cadena de supermercados de la sección 17 quiere subir a la nube su histórico de tiques para procesarlo con Spark: **2 TB en CSV**. Tiene **fibra simétrica de 300 Mbps**, pero de día la usan las cajas, así que solo puede dedicarla a la subida de **22:00 a 06:00 (8 h por noche)**.

**Paso 1: velocidad efectiva.** 300 Mbps × 0,8 = 240 Mbps = 240 / 8 = **30 MB/s**.

**Paso 2: tiempo de transferencia.**

- t = 2·10¹² B × 8 / 240·10⁶ bit/s = 66 667 s ≈ **18,5 horas**.
- En noches: 18,5 / 8 ≈ 2,3 → hacen falta **3 noches**.

**Paso 3: ¿y si se comprime?** Un CSV se comprime al menos 4:1, así que 2 TB pasan a 500 GB:

- t = 500·10⁹ × 8 / 240·10⁶ ≈ 16 667 s ≈ **4,6 horas** → cabe en **una noche**. Comprimir cuesta algo de CPU, pero ahorra dos noches.

**Paso 4: ¿y si se contrata 1 Gbps?** Eficiencia 0,8 → 800 Mbps:

- Sin comprimir: 2·10¹² × 8 / 800·10⁶ = 20 000 s ≈ **5,6 h**.
- Comprimido: 500·10⁹ × 8 / 800·10⁶ = 5 000 s ≈ **1,4 h**.

Comprimir mejora más que contratar una línea tres veces más rápida, y además es gratis.

**Paso 5: la trampa de los ficheros pequeños.** El histórico no es un único fichero: son **20 millones de ficheros de 100 KB** (uno por tique). Si cada fichero añade 50 ms de sobrecarga (abrirlo, negociar, confirmar) y se envían uno a uno:

- Sobrecarga: 20·10⁶ × 0,05 s = 10⁶ s ≈ **11,6 días**, más que toda la transferencia de datos.
- Solución: **empaquetar** (por ejemplo, un `.tar.gz` por día de ventas) o enviar muchos ficheros en paralelo.

**Paso 6: la carga diaria.** Una vez subido el histórico, cada día se generan unos 5 GB nuevos:

- t = 5·10⁹ × 8 / 240·10⁶ ≈ 167 s ≈ **3 minutos** por noche. Sin problema.

**Conclusión.** Lo que parecía «subir unos ficheros» son en realidad **3 noches**, o **12 días** si nadie se fija en el número de ficheros. Con **comprimir y empaquetar** se queda en **una noche** sin gastar nada en una línea mejor. Calcular antes de empezar evita sorpresas y permite comparar opciones con números.
