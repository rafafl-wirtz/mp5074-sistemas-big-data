# <span class="ud ud2">UD2</span> Orixes de datos

**Obtención e tratamento de diferentes tipos e orixes de datos** · 36 sesiones (30 h) · semanas 4–9 · 1ª evaluación

Aprendemos a **obtener datos de cualquier origen** y a dejarlos en un DataFrame de pandas listo para analizar. Vamos de lo más sencillo a lo más complejo: fichero local → muchos ficheros → API → página web estática → página web dinámica → combinar fuentes.

## Resultados de aprendizaje

| RA | Criterios de evaluación |
|---|---|
| **RA3.** Gestiona y almacena datos facilitando la búsqueda de respuestas en grandes conjuntos de datos. | a) extraer y almacenar datos de diversas fuentes · d) gestión y almacenamiento eficiente y seguro, teniendo en cuenta la normativa |
| **RA1.** Aplica técnicas de análisis de datos que integran, procesan y analizan la información. | c) combinar diferentes fuentes y tipos de datos · d) construir conjuntos de datos complejos y relacionarlos |
| **RA4.** Aplica herramientas para la visualización de datos utilizadas en las soluciones Big Data. | a) escenarios y tipologías de datos no estructurados |

## Contenidos

1. **Tipos de datos:** estructurados, semiestructurados y no estructurados. Formatos CSV, JSON, XML y Excel.
2. **pandas:** `Series` y `DataFrame`, lectura de CSV, Excel y JSON, selección, filtros, agrupaciones y series temporales.
3. **APIs:** HTTP, REST frente a SOAP, consumo de servicios web con `requests`, paginación y errores.
4. **Web scraping:**
    - Árbol DOM y selectores CSS.
    - Páginas estáticas con `requests` y BeautifulSoup.
    - Páginas dinámicas con Selenium.
5. **Integración de fuentes:** `merge()`, `join()` y `concat()`.
6. **Ingesta Big Data:** Flume, Sqoop, Kafka y NiFi.
7. **Normativa y ética:** RGPD, LOPDGDD, licencias de datos abiertos y `robots.txt`.

## Librerías de la unidad

```bash
conda activate bigdata
conda install -c conda-forge pandas openpyxl requests beautifulsoup4 lxml selenium
```

!!! warning "Scraping responsable"
    Antes de extraer datos de una web, revisa su `robots.txt` y sus condiciones de uso. Deja pausas entre peticiones (`time.sleep`) y no recojas datos personales sin una base legal.
