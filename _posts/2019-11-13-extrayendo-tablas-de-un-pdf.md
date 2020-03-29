---
layout: single
title: "Extrayendo tablas de un PDF"
date: 2019-11-13
excerpt: Veamos como podemos extraer datos de archivos PDF utilizando `tabula-py`.
categories: [blog]
tags: [scrapping, data wrangling, python, tabula, pdf]
header:
  teaser: ../assets/images/posts/2019-11-13/pdf_to_csv.jpg
  og_image: ../assets/images/posts/2019-11-13/pdf_to_csv.jpg
---

{% include figure image_path="../assets/images/posts/2019-11-13/pdf_to_csv.jpg" alt="dibujo archivo pdf a archivo csv" %}

Hace unos días, pensando en algún caso de estudio para poder jugar un poco con DataFrames y gráficos, se me ocurrió que podía intentar un análisis sobre mis ingresos mensuales y algunas variables económicas.

En principio pensé: _tengo archivados mis recibos de haberes de los últimos 7 u 8 años. Es una buena cantidad de datos para jugar_. Pero... había que extraerlos de alguna manera desde los archivos PDF en los que los tenía archivados. **OK, challenge accepted!** 😎

## El PDF

Así es el recibo. La misma información re replica a ambos lados (copias para el empleado y empleador).
A su vez algunos archivos, como este, contienen una segunda hoja con los ítems que no entraron en la primera.

{% include figure image_path="../assets/images/posts/2019-11-13/pdf01.jpg" alt="imagen del recibo en pdf" caption="**1.** Imagen del recibo en PDF" %}

## Buscando la herramienta

Comencé a navegar un poco y después de algunos clicks llegué a [TabulaPDF](https://tabula.technology/), una app que se autodefine como _"Una herramienta para liberar tablas de datos bloqueadas dentro de archivos PDF"_. ¡Suena perfecto para lo que necesito!

TabulaPDF se ofrece como una app independiente y con una librería de Java llamada `tabula-java`. Afortunadamente existe un wrapper para python de esa librería llamado [tabula-py](https://github.com/chezou/tabula-py).

## Utilizando tabula-py

Luego de investigar un poco la [documentación](https://tabula-py.readthedocs.io/en/latest/) y pelearme un poco con la API, llegué a este script que extrae las tablas que buscaba en el PDF.

```python
import tabula

pdf = tabula.read_pdf(
        file_name,
        pages='all', multiple_tables=True,
        stream=True, pandas_options={'header': None},
        area=(147.6, 417.6, 345.6, 813.6),
        columns=(450.0, 557.28, 595.4, 638.64, 697.68, 748.08, 811.44))
```

Ahora bien, veamos un poco más en detalle los parámetros que configuramos:

Con `pages='all'` y `multiple_tables=True` no hay mucho secreto. Estamos extrayendo información de todas las páginas del PDF e indicando que son tablas distintas.

Con `stream=True` lo que estamos estableciendo es el modo de extracción. Tabula trabaja con dos modos distintos:

- **lattice**: Ideal cuando existen líneas de separación para cada celda, por ejemplo en un PDF que contenga una tabla de Excel.
- **stream**: Para el caso contrario, cuando no hay líneas que delimiten las celdas de la tabla.

Como se puede observar en la imagen de más arriba (1), las tablas en el PDF tienen delimitadas las columnas pero no las celdas. En este caso, probé con ambos métodos y el que me dio mejores resultados fue el método _stream_.

Al parámetro `pandas_options` se le puede pasar un diccionario de python con la configuración deseada para el DataFrame que devolverá. En este caso solo me interesaba quitarle el header.

Los parámetros `area` y `columns` son un poco más confusos. Veamos por qué... 👀

Para empezar, podemos ver que en el archivo no hay una única tabla. Incluso las mismas tablas se encuentran duplicadas del lado _"Original para la empresa"_ y _"Original para el empleado"_. Para lidiar con esto, Tabula permite configurar que áreas del PDF se desean extraer. ¡Genial! 👍, ¿pero que son esos números y como se obtienen? 🤔

Esos números surgen al medir los límites del área a capturar, desde el borde izquierdo del PDF y desde el borde superior. Esas longitudes se miden en pulgadas y se las multiplica por un coeficiente de 72.

💡 _Muchos lectores de PDF, como Acrobat Reader o Foxit PDF, tienen herramientas para obtener las mediciones._
{: .notice}

{% include figure image_path="../assets/images/posts/2019-11-13/pdf02.jpg" alt="área de captura con sus distancias relativas a los bordes" caption="**2.** Área de captura con sus distancias relativas a los bordes" %}

```
1. 2.05 (in) * 72 ~= 147.6
2. 5.80 (in) * 72 ~= 417.6
3. 4.80 (in) * 72 ~= 345.6
4. 11.3 (in) * 72 ~= 813.6

area=(147.6, 417.6, 345.6, 813.6)
```

Hasta ahí definimos el área de captura. Con esto debería ser suficiente para que Tabula traiga la tabla que delimitamos tal cual está en el PDF. Sin embargo no todas las filas tienen valores para todas las columnas, y por lo tanto pude comprobar que en ocasiones confundía de columnas algunos valores o incluso los perdía.
Para solucionar ese problema utilizamos el último parámetro `columns`, donde pasamos una serie de valores para delimitar el ancho de cada columna.

La lógica es similar a la del área, se miden (en pulgadas) las longitudes límites de las columnas respecto del borde izquierdo del PDF y se las multiplica por 72.

```
A. 6.25 (in) * 72 ~= 450.0
B. 7.74 (in) * 72 ~= 557.28
...
F. 10.39 (in) * 72 ~= 748.08
G. 11.27 (in) * 72 ~= 811.44

columns=(450.0, 557.28, 595.4, 638.64, 697.68, 748.08, 811.44)
```

{% include figure image_path="../assets/images/posts/2019-11-13/pdf03.jpg" alt="imagen con las delimitaciones de columna" caption="**3.** Las distancias al borde izquierdo delimitan el ancho de las columnas" %}

¡Listo! Ahora al ejecutar el script Tabula devuelve sobre la variable `pdf` una lista con tantos DataFrames como hojas tenga el recibo.
El próximo paso era acomodar un poco la información.

## Limpiando y preparando la información

Lo primero fue juntar cada una de las tablas provenientes del PDF, seteando el nombre para las columnas y eliminando la fila proveniente al header de la primera tabla.
Luego concatenamos todo en un único DataFrame y reseteamos el índice para identificar cada fila:

```python
df_partials = []

for df_part in pdf:
    df_part.columns = ['codigo','concepto','unidad','valor','haberes_s_ret','haberes_c_ret','descuentos']
    df_part = df_part[df_part.codigo != 'Código']
    df_partials.append(df_part)

df = pd.concat(df_partials)
df.reset_index(inplace=True, drop=True)
```

Utilizamos el módulo `glob` para procesar todos los PDF del directorio. Para poder identificar el período más tarde, extraemos el mismo del nombre del archivo y lo incorporamos como dato en una nueva columna.

```python
import pandas as pd
import tabula
from glob import glob

for file_name in glob('Recibos\*.pdf'):

  pdf = tabula.read_pdf(
    file_name,
    pages='all', multiple_tables=True,
    stream=True, pandas_options={'header': None},
    area=(147.6, 417.6, 345.6, 813.6),
    columns=(450.0, 557.28, 595.4, 638.64, 697.68, 748.08, 811.44))

  df_partials = []

  for df_part in pdf:
      df_part.columns = ['codigo','concepto','unidad','valor','haberes_s_ret','haberes_c_ret','descuentos']
      df_part = df_part[df_part.codigo != 'Código'] # Elimina header de la página 1
      df_partials.append(df_part)

  # Concatena las paginas del recibo y resetea el índice
  df = pd.concat(df_partials)
  df.reset_index(inplace=True, drop=True)

  # Extrae período del nombre del archivo y lo anexa al dataframe
  df['periodo'] = re.search('(\d{4}-\d{2})', file_name).group(1)
```

El próximo paso es convertir los campos necesarios a tipo numérico. Para ello es necesario primero trabajar sobre los strings para:

- Eliminar el separador de miles "."
- Sustituir el separador decimal "," por "."
- Para los números negativos, además, llevar el signo "-" por delante del número y no por detrás.

Para realizar esto implementemos una pequeña función que haga uso de expresiones regulares utilizando el módulo `re` y luego la conversión a tipo numérico directamente con `pandas`:

```python
def to_num(column):
    """ Convierte el formato de las columnas para poder convertirlas a tipo numérico """
    column = str(column)
    column = re.sub('(\d*.\d*,\d{2})-','-\\1', column)
    column = column.replace('.','')
    column = column.replace(',','.')
    return pd.to_numeric(column, errors='coerce')

# Convierto tipo de datos para campos numéricos:
df['unidad'] = pd.to_numeric(df['unidad'].str.replace(',','.'), errors='coerce')
df['valor'] = pd.to_numeric(df['valor'].str.replace(',','.'), errors='coerce')
df['haberes_s_ret'] = df['haberes_s_ret'].apply(to_num)
df['haberes_c_ret'] = df['haberes_c_ret'].apply(to_num)
df['descuentos'] = df['descuentos'].apply(to_num)
```

Finalmente concatenamos los DataFrames de cada recibo en uno solo y estamos listos para exportarlos:

```python
df_recibos.to_csv('Recibos.csv', index=False, encoding='ANSI')
```

---

## Conclusión

Si el formato del PDF medianamente lo permite, extraer la información es relativamente fácil.
En este caso lo que nos ayudó mucho fue el hecho de que la posición de las tablas es siempre la misma y con dimensiones fijas. Esto permitió establecer el área de captura sin ningún problema. La cosa se pone más difícil cuando las dimensiones de la tabla son variables, como en el caso de los resúmenes de las tarjetas de crédito.

En fin, todo este script lo tengo en un Jupyter Notebook que podes consultar en [este repositorio](https://github.com/martinmuelas/Convertidor-Recibos-PDF-a-CSV/blob/master/Convertidor%20Recibos%20PDF%20a%20CSV.ipynb)

---

Si te interesó algún contenido y querés contactarme, no dudes en ponerte en contacto conmigo escribiéndome al mail o en Twitter.

Hasta la próxima!
