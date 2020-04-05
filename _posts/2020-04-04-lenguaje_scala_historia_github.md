---
title: "Lenguaje Scala, historial en Github y proyectos Open Source"
date: 2020-04-04
excerpt: Analizamos el historial en Github del proyecto Scala y aprendemos a evaluar proyectos Open Source para contribuir código.
categories: [notebook]
tags: [data manipulation, data visualization]
header:
  teaser: ../assets/images/jupyter/lenguaje_scala_historia_github_files/scala_teaser.jpg
  og_image: ../assets/images/jupyter/lenguaje_scala_historia_github_files/scala_teaser.jpg
---

![image-center](/assets/images/jupyter/lenguaje_scala_historia_github_files/scala_header.jpg){: .align-center}

### Lenguaje Scala

Con casi 30 mil commits y una historia que abarca más de diez años, Scala es un lenguaje de programación maduro, de propósito general que recientemente se ha convertido en otro lenguaje destacado para los científicos de datos.

Scala también es un proyecto de código abierto. Los proyectos de código abierto tienen la ventaja de que todos sus historiales de desarrollo (quién realizó los cambios, qué se modificó, revisiones de código, etc.) están disponibles públicamente.

Vamos a leer, limpiar y visualizar el [repositorio de Scala](https://github.com/scala/scala) que se extiende por los datos de un sistema de control de versiones (Git) y un sitio de alojamiento de proyectos (GitHub). Descubriremos quién ha tenido más influencia en su desarrollo y quiénes son los expertos.

### Importando y limpiando los datos

El conjunto de datos que utilizaremos, que ha sido minado y extraído directamente de GitHub, está compuesto por dos archivos:

- `pulls.csv` contiene la información básica para cada pull request.
- `pull_files.csv` contiene los archivos que fueron modificados por cada pull request.

Vamos a cargarlos y echarles un vistazo.

```python
import pandas as pd

pulls = pd.read_csv('datasets/pulls.csv')

pulls.head()
```

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }

</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>pid</th>
      <th>user</th>
      <th>date</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>163314316</td>
      <td>hrhino</td>
      <td>2018-01-16T23:29:16Z</td>
    </tr>
    <tr>
      <td>1</td>
      <td>163061502</td>
      <td>joroKr21</td>
      <td>2018-01-15T23:44:52Z</td>
    </tr>
    <tr>
      <td>2</td>
      <td>163057333</td>
      <td>mkeskells</td>
      <td>2018-01-15T23:05:06Z</td>
    </tr>
    <tr>
      <td>3</td>
      <td>162985594</td>
      <td>lrytz</td>
      <td>2018-01-15T15:52:39Z</td>
    </tr>
    <tr>
      <td>4</td>
      <td>162838837</td>
      <td>zuvizudar</td>
      <td>2018-01-14T19:16:16Z</td>
    </tr>
  </tbody>
</table>
</div>

```python
pull_files = pd.read_csv('datasets/pull_files.csv')

pull_files.head()
```

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }

</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>pid</th>
      <th>file</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>163314316</td>
      <td>test/files/pos/t5638/Among.java</td>
    </tr>
    <tr>
      <td>1</td>
      <td>163314316</td>
      <td>test/files/pos/t5638/Usage.scala</td>
    </tr>
    <tr>
      <td>2</td>
      <td>163314316</td>
      <td>test/files/pos/t9291.scala</td>
    </tr>
    <tr>
      <td>3</td>
      <td>163314316</td>
      <td>test/files/run/t8348.check</td>
    </tr>
    <tr>
      <td>4</td>
      <td>163314316</td>
      <td>test/files/run/t8348/TableColumn.java</td>
    </tr>
  </tbody>
</table>
</div>

Los datos sin procesar extraídos de GitHub contienen fechas en el formato ISO8601. Sin embargo, `pandas` los importa como strings regulares. Para facilitar nuestro análisis, necesitamos convertir las cadenas en objetos `DateTime` de Python. Los objetos `DateTime` tienen la propiedad importante de que se pueden comparar y ordenar.

Los tiempos de las pull request están en UTC (también conocido como Coordinated Universal Time). Los tiempos de los commits, sin embargo, se encuentran en la hora local del autor con información de zona horaria (diferencia de horas de UTC). Para facilitar las comparaciones, debemos convertir todos los tiempos a UTC.

```python
pulls['date'] = pd.to_datetime(pulls['date'], utc=True)

pulls.head()
```

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }

</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>pid</th>
      <th>user</th>
      <th>date</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>163314316</td>
      <td>hrhino</td>
      <td>2018-01-16 23:29:16+00:00</td>
    </tr>
    <tr>
      <td>1</td>
      <td>163061502</td>
      <td>joroKr21</td>
      <td>2018-01-15 23:44:52+00:00</td>
    </tr>
    <tr>
      <td>2</td>
      <td>163057333</td>
      <td>mkeskells</td>
      <td>2018-01-15 23:05:06+00:00</td>
    </tr>
    <tr>
      <td>3</td>
      <td>162985594</td>
      <td>lrytz</td>
      <td>2018-01-15 15:52:39+00:00</td>
    </tr>
    <tr>
      <td>4</td>
      <td>162838837</td>
      <td>zuvizudar</td>
      <td>2018-01-14 19:16:16+00:00</td>
    </tr>
  </tbody>
</table>
</div>

### Uniendo los DataFrames

Los datos extraídos vienen en dos archivos separados. Unir los dos DataFrames nos facilitará el análisis de los datos en tareas futuras.

```python
data = pd.merge(pulls, pull_files, on='pid')

data.head()
```

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }

</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>pid</th>
      <th>user</th>
      <th>date</th>
      <th>file</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>163314316</td>
      <td>hrhino</td>
      <td>2018-01-16 23:29:16+00:00</td>
      <td>test/files/pos/t5638/Among.java</td>
    </tr>
    <tr>
      <td>1</td>
      <td>163314316</td>
      <td>hrhino</td>
      <td>2018-01-16 23:29:16+00:00</td>
      <td>test/files/pos/t5638/Usage.scala</td>
    </tr>
    <tr>
      <td>2</td>
      <td>163314316</td>
      <td>hrhino</td>
      <td>2018-01-16 23:29:16+00:00</td>
      <td>test/files/pos/t9291.scala</td>
    </tr>
    <tr>
      <td>3</td>
      <td>163314316</td>
      <td>hrhino</td>
      <td>2018-01-16 23:29:16+00:00</td>
      <td>test/files/run/t8348.check</td>
    </tr>
    <tr>
      <td>4</td>
      <td>163314316</td>
      <td>hrhino</td>
      <td>2018-01-16 23:29:16+00:00</td>
      <td>test/files/run/t8348/TableColumn.java</td>
    </tr>
  </tbody>
</table>
</div>

### ¿Se encuentra actualmente mantenido el proyecto?

La actividad en un proyecto de código abierto no es muy consistente. Algunos proyectos pueden estar activos durante muchos años después del lanzamiento inicial, mientras que otros pueden ir disminuyendo lentamente hacia el olvido. Antes de comprometerse a contribuir a un proyecto, es importante comprender el estado del proyecto. ¿El desarrollo avanza constantemente o hay una caída? ¿Se ha abandonado el proyecto por completo?

Los datos utilizados en este proyecto se recopilaron en enero de 2018. Estamos interesados en la evolución del número de contribuciones hasta esa fecha.

Para Scala, haremos esto trazando un gráfico de la actividad del proyecto. Calcularemos el número de pull request enviadas cada mes (calendario) durante la vida útil del proyecto. Luego trazaremos estos números para ver la tendencia de las contribuciones.

```python
%matplotlib inline

# Agrupamos por mes y contamos las pull request al final del mismo
counts = pulls.groupby(pulls.date.dt.strftime('%Y-%m')).count()['pid']

# Graficamos
counts.plot(kind='bar', figsize=[17, 8])
```

    <matplotlib.axes._subplots.AxesSubplot at 0x1cbf6e70e88>

![png](/assets/images/jupyter/lenguaje_scala_historia_github_files//lenguaje_scala_historia_github_9_1.png)

### ¿Hay camaradería en el proyecto?

La estructura organizacional varía de un proyecto a otro, y puede influir en el éxito como contribuyente. Un proyecto que tiene una comunidad muy pequeña podría no ser la mejor opción para comenzar a trabajar, ya que esto podría significar una alta barrera de entrada debido a varios factores, como por ejemplo, que dicha comunidad sea reacia a aceptar pull requests de "personas ajenas", con las que es difícil trabajar con la base del código, etc. Sin embargo, una comunidad grande puede servir como un indicador de que el proyecto está abierto a aceptar requests de nuevos contribuyentes. Esos proyectos serían un mejor lugar para comenzar.

Para evaluar la dinámica de la comunidad, trazaremos un histograma del número de pull requests enviadas por cada usuario. Una distribución que muestre que hay pocas personas contribuyendo solo con un pequeño número de pull requests se podría utilizar como un indicador de que el proyecto no acepta nuevos contribuyentes.

```python
# Agrupamos por usuario y contamos cantidad de contribuciones
by_user = pulls.groupby('user').count()['pid'].sort_values(ascending=False)
by_user
```

    user
    retronym     1030
    xeno-by       477
    adriaanm      474
    paulp         440
    som-snytt     376
                 ...
    larsrh          1
    lastland        1
    lexspoon        1
    lifuhuang       1
    0xmohit         1
    Name: pid, Length: 467, dtype: int64

```python
# Graficamos el histograma
ax = by_user.hist(figsize=[17,8])

# Aplicamos escala logaritmica en y para observar mejor los valores
ax.set_yscale("log")
```

![png](/assets/images/jupyter/lenguaje_scala_historia_github_files//lenguaje_scala_historia_github_12_0.png)

Podemos observar que en este proyecto un grupo reducido de usuarios hace una gran cantidad de contribuciones, pero a su vez existe una gran cantidad de usuarios aportando una cantidad menor de contribuciones al código.

### ¿Qué archivos fueron modificados en los últimos 10 pull requests?

Elegir el lugar correcto para hacer una contribución es tan importante como elegir el proyecto al que contribuir. Algunas partes del código pueden ser estables, algunas pueden estar muertas. Contribuir allí podría no tener el mayor impacto. Por lo tanto, es importante comprender las partes del sistema que se han cambiado recientemente. Esto nos permite identificar las áreas "calientes" del código donde está ocurriendo la mayor parte de la actividad. Centrarse en esas partes podría no ser el uso más efectivo de nuestro tiempo.

```python
# Ultimos 10 pull requests
last_10 = pulls.sort_values('date').tail(10)

# Cruzamos con pull_files para saber cuales son los archivos afectados por esos PR
joined_pr = pd.merge(last_10, pull_files, on='pid')

# Identificamos archivos afectados
files = joined_pr['file'].unique()

for filename in sorted(files):
    print(filename)
```

    LICENSE
    doc/LICENSE.md
    doc/License.rtf
    project/VersionUtil.scala
    src/compiler/scala/reflect/reify/phases/Calculate.scala
    src/compiler/scala/tools/nsc/backend/jvm/BCodeHelpers.scala
    src/compiler/scala/tools/nsc/backend/jvm/PostProcessor.scala
    src/compiler/scala/tools/nsc/backend/jvm/analysis/BackendUtils.scala
    src/compiler/scala/tools/nsc/profile/AsyncHelper.scala
    src/compiler/scala/tools/nsc/profile/Profiler.scala
    src/compiler/scala/tools/nsc/symtab/classfile/ClassfileParser.scala
    src/compiler/scala/tools/nsc/typechecker/Contexts.scala
    src/library/scala/Predef.scala
    src/library/scala/concurrent/Lock.scala
    src/library/scala/util/Properties.scala
    src/reflect/scala/reflect/internal/pickling/ByteCodecs.scala
    src/reflect/scala/reflect/internal/tpe/GlbLubs.scala
    src/scaladoc/scala/tools/nsc/doc/html/page/Entity.scala
    src/scalap/decoder.properties
    test/files/neg/leibniz-liskov.check
    test/files/neg/leibniz-liskov.scala
    test/files/pos/leibniz-liskov.scala
    test/files/pos/leibniz_liskov.scala
    test/files/pos/parallel-classloader.scala
    test/files/pos/t10568/Converter.java
    test/files/pos/t10568/Impl.scala
    test/files/pos/t10686.scala
    test/files/pos/t5638/Among.java
    test/files/pos/t5638/Usage.scala
    test/files/pos/t9291.scala
    test/files/run/t8348.check
    test/files/run/t8348/TableColumn.java
    test/files/run/t8348/TableColumnImpl.java
    test/files/run/t8348/Test.scala

### ¿Quién realizó la mayor cantidad de pull requests sobre un determinado archivo?

Al contribuir a un proyecto, podríamos necesitar alguna orientación. Es posible que necesitemos información sobre el código base. Es importante dirigir cualquier pregunta a la persona adecuada. Los contribuyentes a proyectos de código abierto generalmente tienen otros trabajos diarios, por lo que su tiempo es limitado. Es importante dirigir nuestras preguntas a las personas adecuadas. Una forma de identificar el objetivo correcto para nuestras consultas es utilizando su historial de contribuciones.

Identificamos `src/compiler/scala/reflect/reify/phase/Calculate.scala` como un archivo cambiado recientemente. Nos interesaremos en los 3 principales desarrolladores que cambiaron ese archivo. Esos desarrolladores son los más propensos a tener la mejor comprensión del código.

```python
# Este es el archivo que nos interesa
file = 'src/compiler/scala/reflect/reify/phases/Calculate.scala'

# Identificamos los commits que modificaron el archivo
file_pr = pull_files[pull_files['file'] == file]

# Contamos el numero de cambios realizado al archivo por cada desarrollador y lo ordenamos descendentemente
author_counts = pd.merge(file_pr, pulls, on='pid').groupby('user').count().reset_index().sort_values('pid', ascending=False)

# Top 3 desarrolladores
author_counts['user'].head(3)
```

    9     xeno-by
    6    retronym
    7         soc
    Name: user, dtype: object

### ¿Quienes realizaron los últimos 10 pull requests sobre un determinado archivo?

Los proyectos de código abierto sufren de membresía fluctuante. Esto hace que el problema de encontrar a la persona adecuada sea más desafiante: la persona tiene que estar bien informada y aún estar involucrada en el proyecto. Es posible que una persona que contribuyó mucho en el pasado ya no esté disponible (o dispuesta) a ayudar. Para obtener una mejor comprensión, necesitamos investigar la historia más reciente de esa parte particular del sistema.

Al igual que antes, veremos el historial de `src/compiler/scala/reflect/reify/phase/Calculate.scala`.

```python
file = 'src/compiler/scala/reflect/reify/phases/Calculate.scala'

# Pull Requests que cambiaron el archivo
file_pr = pull_files[pull_files['file'] == file]

# Cruzamos con pulls para obtener los usuarios que realizaron esos PR
joined_pr = pd.merge(file_pr, pulls, on='pid')

# Usuarios que realizaron los últimos 10 pull requests
users_last_10 = joined_pr.nlargest(10, 'date')['user'].unique()

for user in sorted(users_last_10):
    print(user)
```

    bjornregnell
    retronym
    soc
    starblood
    xeno-by
    zuvizudar

### Los pull requests de dos usuarios en particular

Ahora que hemos identificado dos contactos potenciales en el proyecto, necesitamos encontrar a la persona que estuvo más involucrada en el mismo en los últimos tiempos. Es más probable que esa persona responda nuestras preguntas. Para cada año calendario, estamos interesados en comprender la cantidad de pull requests que realizaron los autores. Esto nos dará una idea a grandes rasgos de su tendencia de contribución al proyecto.

```python
# Nos centraremos en estos dos usuarios
authors = ['xeno-by', 'soc']

# Obtenemos todos los pull requests de estos usuarios
by_author = pulls[pulls['user'].isin(authors)]

# Contamos los PR realizados cada año
counts = by_author.groupby(['user', by_author.date.dt.year]).agg({'pid': 'count'}).reset_index()

# Convertimos la tabla a formato ancho
counts_wide = counts.pivot_table(index='date', columns='user', values='pid', fill_value=0)

# Graficamos los resultados
counts_wide.plot(kind='bar', figsize=[17,8])
```

    <matplotlib.axes._subplots.AxesSubplot at 0x1cbfa3f8948>

![png](/assets/images/jupyter/lenguaje_scala_historia_github_files//lenguaje_scala_historia_github_21_1.png)

### Visualizando las contribuciones de cada desarrollador

Como se mencionó anteriormente, es importante hacer una distinción entre la experiencia global y los niveles de contribución y los niveles de contribución a un nivel más granular (archivo, submódulo, etc.). En nuestro caso, queremos ver cuál de nuestros dos desarrolladores de interés tiene la mayor experiencia con el código en un archivo determinado. Mediremos la experiencia según la cantidad de pull requests enviadas que afectan a ese archivo y qué tan recientes se enviaron.

```python
authors = ['xeno-by', 'soc']
file = 'src/compiler/scala/reflect/reify/phases/Calculate.scala'

# Pull requests enviados por los usuarios de interés
by_author = data[data['user'].isin(authors)]

# Nos quedamos solo con los que afectaron al archivo de interés
by_file = by_author[by_author['file'] == file]

# Agrupamos y contamos el número de PR realizados cada año
grouped = by_file.groupby(['user', by_file['date'].dt.year]).count()['pid'].reset_index()

# Transformamos la tabla a formato ancho
by_file_wide = grouped.pivot_table(index='date', columns='user', values='pid', fill_value=0)

# Graficamos los resultados
by_file_wide.plot(kind='bar', figsize=[17, 8])
```

    <matplotlib.axes._subplots.AxesSubplot at 0x1cbf9a57f48>

![png](/assets/images/jupyter/lenguaje_scala_historia_github_files//lenguaje_scala_historia_github_23_1.png)

### Conclusión

A lo largo de este informe hemos aprendido no solo un poco del historial de cambios del Lenguaje Scala, sino también una forma de evaluar proyectos Open Source para encontrar la mejor manera de comenzar a contribuir código en Github u otras plataformas de crowdsourcing.

---

Espero que este post te haya resultado interesante y si tenés alguna consulta o sugerencia no dudes en contactarme al mail o seguirme en Twitter.

¡Hasta la próxima!

---

Este trabajo forma parte de los proyectos propuestos en la carrera [Data Scientist with Python](https://www.datacamp.com/tracks/data-scientist-with-python?version=3) en Datacamp.
{: .notice}
