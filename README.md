# etl_clase_07_05

Repositorio de la clase del 07/05 sobre cómo plantear un proyecto **ETL** (Extract, Transform, Load) básico de principio a fin: planteamiento, arquitectura, descubrimiento de datos, transformaciones, calidad y salida.

## Contenido del repositorio

```
etl_clase_07_05-1/
├── README.md
├── ppt_clase.pdf          # Diapositivas de la clase (11 páginas)
├── data/
│   └── data_raw.csv       # Dataset crudo (NASA Exoplanet Archive, ~36 MB)
└── venv/                  # Entorno virtual de Python 3.14
```

## 1. `ppt_clase.pdf` — Diapositivas de la clase

PDF de **11 páginas** (v1.7) que guía la sesión práctica de ETL. Resumen por página:

| Página | Sección | Contenido |
|--------|---------|-----------|
| 1 | Portada | *Haciendo un proyecto de ETL básico* |
| 2 | Vamos al código | Objetivo: procesar datos del baloncesto desde [Basketball-Reference.com](https://www.basketball-reference.com/). Como ETL es **agnóstico de la finalidad**, también se proponen como alternativas: *Materials Project*, *Exoplanet Archive*, *GenBank* (estas últimas con API y sin necesidad de web scraping). La página será la base de datos; el resto de decisiones (incluido un dashboard opcional) son del alumno. |
| 3 | Planeamiento | *Lo primero es lo primero*: planear, y después echar un ojo a los datos. |
| 4 | Las dos realidades del ETL | Distinción entre **flujo de planeamiento/diseño** y **flujo de datos**. Fuente citada: *The Data Warehouse ETL Toolkit*. |
| 5 | Arquitectura del proceso | Considerar solo **staging** (y algo de front-end), porque la base de datos no se puede tocar. Pregunta clave: ¿procesamiento **batch** o **streaming**? |
| 6 | Descubrimiento y mapas de datos | Introducción al *data discovery*. |
| 7 | ¿Qué vamos a extraer? | Tipos de datos disponibles y expectativa de **anomalías**. |
| 8 | Transformaciones / Calidad | Qué transformar y cómo asegurar la calidad. |
| 9 | Salida de los datos | Formato deseado de la salida. |
| 10 | A picar código | ¿Por dónde empezamos? |
| 11 | (en blanco) | — |

## 2. `data/data_raw.csv` — Dataset

Volcado en **CSV** del **NASA Exoplanet Archive** (`http://exoplanetarchive.ipac.caltech.edu`).

- **Producido el:** Thu May 7 11:00:29 2026
- **Tamaño:** 38 189 180 bytes (~36 MB)
- **Filas de datos:** 39 852
- **Columnas:** 92
- **Líneas de cabecera comentadas (`#`):** 96 (metadatos + diccionario de columnas)
- **Planetas únicos (`pl_name`):** 6 286
- **Estrellas anfitrionas únicas (`hostname`):** 4 707
- **Rango temporal de descubrimiento (`disc_year`):** 1992 – 2026

> Para leer el CSV con pandas hay que ignorar las líneas que empiezan por `#`:
> ```python
> import pandas as pd
> df = pd.read_csv("data/data_raw.csv", comment="#", low_memory=False)
> ```

### 2.1 Estructura del fichero

1. **Líneas 1–96**: comentarios `#` con metadatos de la exportación y un bloque `# COLUMN <nombre>: <descripción>` por cada una de las 92 columnas.
2. **Línea 97**: cabecera CSV con los nombres de columna.
3. **Línea 98 en adelante**: registros de datos (un planeta por fila, con varias entradas por planeta cuando hay múltiples publicaciones; ver `default_flag`).

### 2.2 Diccionario de columnas (92)

Convenciones de sufijos:
- `*err1` / `*err2` → incertidumbre superior / inferior asociada al valor.
- `*lim` → *limit flag* (cuando el valor es un límite y no una medida).

#### Identificación y metainformación del registro
| Columna | Descripción |
|---|---|
| `pl_name` | Nombre del planeta |
| `hostname` | Nombre de la estrella anfitriona |
| `default_flag` | 1 si la fila es el set de parámetros por defecto del planeta, 0 si no |
| `sy_snum` | Nº de estrellas en el sistema |
| `sy_pnum` | Nº de planetas en el sistema |
| `discoverymethod` | Método de descubrimiento (Transit, Radial Velocity, Microlensing, Imaging, …) |
| `disc_year` | Año de descubrimiento |
| `disc_facility` | Instalación/observatorio del descubrimiento |
| `soltype` | Tipo de solución (Published Confirmed, Kepler/TESS Candidate, …) |
| `pl_controv_flag` | 1 si el planeta es **controvertido** |
| `pl_refname` | Referencia bibliográfica de los parámetros planetarios |

#### Parámetros orbitales y físicos del planeta
| Columna | Descripción |
|---|---|
| `pl_orbper` (+`err1`/`err2`/`lim`) | Periodo orbital [días] |
| `pl_orbsmax` (+`err1`/`err2`/`lim`) | Semieje mayor [au] |
| `pl_rade` (+`err1`/`err2`/`lim`) | Radio del planeta [radios terrestres] |
| `pl_radj` (+`err1`/`err2`/`lim`) | Radio del planeta [radios jovianos] |
| `pl_bmasse` (+`err1`/`err2`/`lim`) | Masa o M·sin(i) [masas terrestres] |
| `pl_bmassj` (+`err1`/`err2`/`lim`) | Masa o M·sin(i) [masas jovianas] |
| `pl_bmassprov` | Origen de la masa (`Mass`, `Msini`, …) |
| `pl_orbeccen` (+`err1`/`err2`/`lim`) | Excentricidad |
| `pl_insol` (+`err1`/`err2`/`lim`) | Flujo de insolación [flujos terrestres] |
| `pl_eqt` (+`err1`/`err2`/`lim`) | Temperatura de equilibrio [K] |
| `ttv_flag` | 1 si presenta variaciones en el tiempo de tránsito (TTV) |

#### Parámetros estelares
| Columna | Descripción |
|---|---|
| `st_refname` | Referencia bibliográfica de los parámetros estelares |
| `st_spectype` | Tipo espectral |
| `st_teff` (+`err1`/`err2`/`lim`) | Temperatura efectiva [K] |
| `st_rad` (+`err1`/`err2`/`lim`) | Radio estelar [radios solares] |
| `st_mass` (+`err1`/`err2`/`lim`) | Masa estelar [masas solares] |
| `st_met` (+`err1`/`err2`/`lim`) | Metalicidad [dex] |
| `st_metratio` | Ratio de metalicidad (p. ej. `[Fe/H]`) |
| `st_logg` (+`err1`/`err2`/`lim`) | Gravedad superficial [log10(cm/s²)] |

#### Parámetros del sistema (posición, distancia, magnitudes)
| Columna | Descripción |
|---|---|
| `sy_refname` | Referencia bibliográfica del sistema |
| `rastr` / `ra` | Ascensión recta (sexagesimal / grados) |
| `decstr` / `dec` | Declinación (sexagesimal / grados) |
| `sy_dist` (+`err1`/`err2`) | Distancia [pc] |
| `sy_vmag` (+`err1`/`err2`) | Magnitud V (Johnson) |
| `sy_kmag` (+`err1`/`err2`) | Magnitud Ks (2MASS) |
| `sy_gaiamag` (+`err1`/`err2`) | Magnitud Gaia |

#### Fechas
| Columna | Descripción |
|---|---|
| `rowupdate` | Fecha de la última actualización de la fila |
| `pl_pubdate` | Fecha de publicación de la referencia planetaria |
| `releasedate` | Fecha de release en el archivo |

### 2.3 Distribuciones relevantes

**Método de descubrimiento (`discoverymethod`)**
| Método | Filas |
|---|---:|
| Transit | 35 749 |
| Radial Velocity | 2 890 |
| Microlensing | 798 |
| Imaging | 184 |
| Transit Timing Variations | 162 |
| Eclipse Timing Variations | 26 |
| Orbital Brightness Modulation | 21 |
| Pulsar Timing | 13 |
| Astrometry | 6 |
| Pulsation Timing Variations | 2 |

**Tipo de solución (`soltype`)**
| Solución | Filas |
|---|---:|
| Published Confirmed | 20 369 |
| Kepler Project Candidate (q1_q17_dr25_sup_koi) | 2 735 |
| Kepler Project Candidate (q1_q16_koi) | 2 722 |
| Kepler Project Candidate (q1_q17_dr25_koi) | 2 718 |
| Kepler Project Candidate (q1_q17_dr24_koi) | 2 701 |
| Kepler Project Candidate (q1_q12_koi) | 2 680 |
| Kepler Project Candidate (q1_q8_koi) | 2 307 |
| Published Candidate | 2 195 |
| TESS Project Candidate | 1 425 |

**Banderas**
- `default_flag`: 6 286 filas con valor 1 (una por planeta) y 33 566 con 0.
- `pl_controv_flag`: 145 planetas marcados como controvertidos.

### 2.4 Notas para el ETL

- Las celdas `pl_refname`, `st_refname` y `sy_refname` contienen **HTML** (`<a refstr=… href=… target=ref>…</a>`); habrá que limpiarlas si se quieren las referencias en texto plano.
- Hay **múltiples filas por planeta** (una por publicación). Para análisis "un planeta = una fila", filtra por `default_flag == 1`.
- Muchas columnas numéricas están vacías; conviene revisar nulos por columna antes de transformar.
- Los flags `*lim` indican que el valor es un límite (superior/inferior), no una medida; tenerlos en cuenta antes de calcular estadísticos.

## 3. Entorno de desarrollo

Python **3.14.0** en `venv/`.

```bash
source venv/bin/activate          # activar
pip install pandas pypdf          # paquetes ya instalados durante la exploración
deactivate                        # salir del entorno
```
