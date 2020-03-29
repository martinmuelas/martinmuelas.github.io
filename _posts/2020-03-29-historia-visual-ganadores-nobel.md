---
title: "Historia visual de los ganadores del Premio Nobel"
date: 2020-03-29
excerpt: El Premio Nobel es quizás el premio científico más conocido del mundo. Conozcamos un poco de su historia a través de los datos.
categories: [notebook]
tags: [data manipulation, data visualization]
header:
  teaser: ../assets/images/jupyter/historia_visual_ganadores_nobel_files/nobel_teaser.jpg
---

{% include figure image_path="../assets/images/jupyter/historia_visual_ganadores_nobel_files/nobel_header.jpg" alt="cartel con frase: for the greatest benefit to humankind" %}

### El más Nobel de los premios

El Premio Nobel es quizás el premio científico más conocido del mundo. Además del honor, el prestigio y una considerable suma de dinero como premio, el destinatario recibe también una medalla de oro con la imagen de [Alfred Nobel](https://es.wikipedia.org/wiki/Alfred_Nobel) (1833-1896), quien instituyó el premio en 1895. Cada año se entrega a científicos y académicos en las categorías química, literatura, física, fisiología o medicina, economía y paz. El primer Premio Nobel se entregó en 1901, y en ese momento el Premio era muy Eurocéntrico y centrado en los hombres, aunque hoy en día ya no está sesgado de ninguna manera. Seguro... ¿No? 🤔

Bueno ok, vamos a averiguarlo! La Fundación Nobel ha puesto a disposición un [dataset](https://www.kaggle.com/nobelfoundation/nobel-laureates) con todos los ganadores del premio desde el comienzo del mismo, en 1901, hasta 2016. Vamos a cargarlo y echar un vistazo.

```python
# Cargamos las librerías necesarias
import pandas as pd
import seaborn as sns
import numpy as np
import matplotlib.pyplot as plt
%matplotlib inline

# Establecemos el theme para Seaborn
sns.set()
# y el tamaño para todos los gráficos
plt.rcParams['figure.figsize'] = [18, 7]

# Cargamos el dataset
nobel = pd.read_csv("datasets/nobel.csv")

# Veamos que contiene
nobel.info()
```

    RangeIndex: 911 entries, 0 to 910
    Data columns (total 18 columns):
    year                    911 non-null int64
    category                911 non-null object
    prize                   911 non-null object
    motivation              823 non-null object
    prize_share             911 non-null object
    laureate_id             911 non-null int64
    laureate_type           911 non-null object
    full_name               911 non-null object
    birth_date              883 non-null object
    birth_city              883 non-null object
    birth_country           885 non-null object
    sex                     885 non-null object
    organization_name       665 non-null object
    organization_city       667 non-null object
    organization_country    667 non-null object
    death_date              593 non-null object
    death_city              576 non-null object
    death_country           582 non-null object
    dtypes: int64(2), object(16)
    memory usage: 128.2+ KB

```python
# Y un vistazo a la información
nobel.head(10)
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
      <th>year</th>
      <th>category</th>
      <th>prize</th>
      <th>motivation</th>
      <th>prize_share</th>
      <th>laureate_id</th>
      <th>laureate_type</th>
      <th>full_name</th>
      <th>birth_date</th>
      <th>birth_city</th>
      <th>birth_country</th>
      <th>sex</th>
      <th>organization_name</th>
      <th>organization_city</th>
      <th>organization_country</th>
      <th>death_date</th>
      <th>death_city</th>
      <th>death_country</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>1901</td>
      <td>Chemistry</td>
      <td>The Nobel Prize in Chemistry 1901</td>
      <td>"in recognition of the extraordinary services ...</td>
      <td>1/1</td>
      <td>160</td>
      <td>Individual</td>
      <td>Jacobus Henricus van 't Hoff</td>
      <td>1852-08-30</td>
      <td>Rotterdam</td>
      <td>Netherlands</td>
      <td>Male</td>
      <td>Berlin University</td>
      <td>Berlin</td>
      <td>Germany</td>
      <td>1911-03-01</td>
      <td>Berlin</td>
      <td>Germany</td>
    </tr>
    <tr>
      <td>1</td>
      <td>1901</td>
      <td>Literature</td>
      <td>The Nobel Prize in Literature 1901</td>
      <td>"in special recognition of his poetic composit...</td>
      <td>1/1</td>
      <td>569</td>
      <td>Individual</td>
      <td>Sully Prudhomme</td>
      <td>1839-03-16</td>
      <td>Paris</td>
      <td>France</td>
      <td>Male</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1907-09-07</td>
      <td>Châtenay</td>
      <td>France</td>
    </tr>
    <tr>
      <td>2</td>
      <td>1901</td>
      <td>Medicine</td>
      <td>The Nobel Prize in Physiology or Medicine 1901</td>
      <td>"for his work on serum therapy, especially its...</td>
      <td>1/1</td>
      <td>293</td>
      <td>Individual</td>
      <td>Emil Adolf von Behring</td>
      <td>1854-03-15</td>
      <td>Hansdorf (Lawice)</td>
      <td>Prussia (Poland)</td>
      <td>Male</td>
      <td>Marburg University</td>
      <td>Marburg</td>
      <td>Germany</td>
      <td>1917-03-31</td>
      <td>Marburg</td>
      <td>Germany</td>
    </tr>
    <tr>
      <td>3</td>
      <td>1901</td>
      <td>Peace</td>
      <td>The Nobel Peace Prize 1901</td>
      <td>NaN</td>
      <td>1/2</td>
      <td>462</td>
      <td>Individual</td>
      <td>Jean Henry Dunant</td>
      <td>1828-05-08</td>
      <td>Geneva</td>
      <td>Switzerland</td>
      <td>Male</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1910-10-30</td>
      <td>Heiden</td>
      <td>Switzerland</td>
    </tr>
    <tr>
      <td>4</td>
      <td>1901</td>
      <td>Peace</td>
      <td>The Nobel Peace Prize 1901</td>
      <td>NaN</td>
      <td>1/2</td>
      <td>463</td>
      <td>Individual</td>
      <td>Frédéric Passy</td>
      <td>1822-05-20</td>
      <td>Paris</td>
      <td>France</td>
      <td>Male</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1912-06-12</td>
      <td>Paris</td>
      <td>France</td>
    </tr>
    <tr>
      <td>5</td>
      <td>1901</td>
      <td>Physics</td>
      <td>The Nobel Prize in Physics 1901</td>
      <td>"in recognition of the extraordinary services ...</td>
      <td>1/1</td>
      <td>1</td>
      <td>Individual</td>
      <td>Wilhelm Conrad Röntgen</td>
      <td>1845-03-27</td>
      <td>Lennep (Remscheid)</td>
      <td>Prussia (Germany)</td>
      <td>Male</td>
      <td>Munich University</td>
      <td>Munich</td>
      <td>Germany</td>
      <td>1923-02-10</td>
      <td>Munich</td>
      <td>Germany</td>
    </tr>
    <tr>
      <td>6</td>
      <td>1902</td>
      <td>Chemistry</td>
      <td>The Nobel Prize in Chemistry 1902</td>
      <td>"in recognition of the extraordinary services ...</td>
      <td>1/1</td>
      <td>161</td>
      <td>Individual</td>
      <td>Hermann Emil Fischer</td>
      <td>1852-10-09</td>
      <td>Euskirchen</td>
      <td>Prussia (Germany)</td>
      <td>Male</td>
      <td>Berlin University</td>
      <td>Berlin</td>
      <td>Germany</td>
      <td>1919-07-15</td>
      <td>Berlin</td>
      <td>Germany</td>
    </tr>
    <tr>
      <td>7</td>
      <td>1902</td>
      <td>Literature</td>
      <td>The Nobel Prize in Literature 1902</td>
      <td>"the greatest living master of the art of hist...</td>
      <td>1/1</td>
      <td>571</td>
      <td>Individual</td>
      <td>Christian Matthias Theodor Mommsen</td>
      <td>1817-11-30</td>
      <td>Garding</td>
      <td>Schleswig (Germany)</td>
      <td>Male</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1903-11-01</td>
      <td>Charlottenburg</td>
      <td>Germany</td>
    </tr>
    <tr>
      <td>8</td>
      <td>1902</td>
      <td>Medicine</td>
      <td>The Nobel Prize in Physiology or Medicine 1902</td>
      <td>"for his work on malaria, by which he has show...</td>
      <td>1/1</td>
      <td>294</td>
      <td>Individual</td>
      <td>Ronald Ross</td>
      <td>1857-05-13</td>
      <td>Almora</td>
      <td>India</td>
      <td>Male</td>
      <td>University College</td>
      <td>Liverpool</td>
      <td>United Kingdom</td>
      <td>1932-09-16</td>
      <td>Putney Heath</td>
      <td>United Kingdom</td>
    </tr>
    <tr>
      <td>9</td>
      <td>1902</td>
      <td>Peace</td>
      <td>The Nobel Peace Prize 1902</td>
      <td>NaN</td>
      <td>1/2</td>
      <td>464</td>
      <td>Individual</td>
      <td>Élie Ducommun</td>
      <td>1833-02-19</td>
      <td>Geneva</td>
      <td>Switzerland</td>
      <td>Male</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1906-12-07</td>
      <td>Bern</td>
      <td>Switzerland</td>
    </tr>
  </tbody>
</table>
</div>

### Asi que... quienes obtienen el Premio Nobel?

Tan solo mirando a los primeros ganadores del premio, ya vemos a una celebridad: [Wilhelm Conrad Röntgen](https://es.wikipedia.org/wiki/Wilhelm_R%C3%B6ntgen), el físico e ingeniero alemán que descubrió los rayos X. Y, de hecho, vemos que todos los ganadores en 1901 fueron hombres provenientes de Europa. Pero eso fue en 1901, mirando a todos los ganadores en el conjunto de datos, de 1901 a 2016, ¿qué sexo y de qué país son los más comúnmente representados?

(Para el país, utilizaremos el país de nacimiento "`birth_date`" del ganador, ya que el país de organización "`organization_country`" es `NaN` para todos los Premios Nobel compartidos).

```python
# Premios Nobel entregados entre 1901 y 2016
print("Cantidad de premios Nobel entregados hasta 2016: " + str(len(nobel)))
```

    Cantidad de premios Nobel entregados hasta 2016: 911

```python
# Número de premios entregados según sexo del ganador
nobel["sex"].value_counts()
```

    Male      836
    Female     49
    Name: sex, dtype: int64

```python
# Top 10 de nacionalidades con mayor cantidad de premios entregados
nobel["birth_country"].value_counts().head(10)
```

    United States of America    259
    United Kingdom               85
    Germany                      61
    France                       51
    Sweden                       29
    Japan                        24
    Canada                       18
    Netherlands                  18
    Russia                       17
    Italy                        17
    Name: birth_country, dtype: int64

### Dominación de EE.UU.

Quizás no sea tan sorprendente: el premio Nobel más típico entre 1901 y 2016 fue para hombres nacidos en los Estados Unidos de América. Pero en 1901 todos los ganadores eran europeos. ¿Cuándo comenzó Estados Unidos a dominar las listas del Premio Nobel?

```python
# Creamos una columna para ganadores estadounidense
nobel["usa_born_winner"] = nobel["birth_country"].apply(lambda x: True if x == "United States of America" else False)

# Creamos una columna con la década en que fue otorgado el premio
nobel["decade"] = nobel["year"].apply(lambda x: x - (x % 10))

# Calculamos la proporción de ganadores estadounidenses por década
prop_usa_winners = nobel.groupby("decade", as_index=False)["usa_born_winner"].mean()

# Proporción de ganadores estadounidenses por década
print(prop_usa_winners)
```

        decade  usa_born_winner
    0     1900         0.017544
    1     1910         0.075000
    2     1920         0.074074
    3     1930         0.250000
    4     1940         0.302326
    5     1950         0.291667
    6     1960         0.265823
    7     1970         0.317308
    8     1980         0.319588
    9     1990         0.403846
    10    2000         0.422764
    11    2010         0.292683

Conviene mejor visualizarlo con un gráfico

```python
# Creamos gráfico de línea para la proporción de ganadores estadounidenses
ax = sns.lineplot(x="decade", y="usa_born_winner", data=nobel, ci=False)

# Agregamos formato porcentual para el eje y
from matplotlib.ticker import PercentFormatter
ax.yaxis.set_major_formatter(PercentFormatter(1.0))

# Establecemos el nombre de los ejes
ax.set_ylabel("Proporcion de ganadores nacidos en EE.UU.")
ax.set_xlabel("Década")

plt.show()
```

![png](/assets/images/jupyter/historia_visual_ganadores_nobel_files/historia_visual_ganadores_nobel_10_0.png)

### Cuál es el género de un típico ganador del Premio Nobel?

Como podemos observar, Estados Unidos se convirtió en el ganador dominante del Premio Nobel por primera vez en la década de 1930 y mantiene la posición de liderazgo desde entonces. Pero un grupo que estuvo a la cabeza desde el principio, y que nunca parece ceder terreno, son los hombres. Tal vez no debería sorprender que haya algún desequilibrio entre ganadores de premios masculinos y femeninos, pero ¿qué tan significativo es este desequilibrio? ¿Y es mejor o peor dentro de cada una de las categorías específicas como física, medicina, literatura, etc.?

```python
# Creamos una columna para las ganadoras femeninas
nobel["female_winner"] = nobel["sex"].apply(lambda x: True if x == "Female" else False)

# Calculamos la proporción de ganadoras femeninas a lo largo de las décadas y para cada categoría
prop_female_winners = nobel.groupby(["decade","category"], as_index=False)["female_winner"].mean()

# Creamos gráfico de líneas
ax = sns.lineplot(x="decade", y="female_winner", data=nobel, ci=False, hue="category")

# Agregamos formato porcentual para el eje y
ax.yaxis.set_major_formatter(PercentFormatter(1.0))

# Establecemos el nombre de los ejes
ax.set_ylabel("Proporción de ganadoras femeninas")
ax.set_xlabel("Década")

plt.show()
```

![png](/assets/images/jupyter/historia_visual_ganadores_nobel_files/historia_visual_ganadores_nobel_12_0.png)

### Primera mujer en ganar el Premio Nobel

El gráfico anterior quedó bastante confuso ya que las líneas se superponen, pero muestra algunas tendencias y patrones interesantes. En general, el desequilibrio es bastante grande, sobre todo en física, economía y química que tienen las menores proporciones. Medicina tiene una tendencia algo positiva, y desde la década de 1990 el premio de literatura está más equilibrado. El gran valor atípico es el premio de la paz durante la década de 2010, pero tengamos en cuenta que esto solo cubre los años 2010 a 2016.

Dado este desequilibrio, ¿quién fue la primera mujer en recibir un Premio Nobel? ¿Y en qué categoría?

```python
# Filtramos las mujeres y nos quedamos con la ganadora más antigua
nobel[nobel["sex"] == "Female"].nsmallest(1, "year")[["year","full_name","category"]]
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
      <th>year</th>
      <th>full_name</th>
      <th>category</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>19</td>
      <td>1903</td>
      <td>Marie Curie, née Sklodowska</td>
      <td>Physics</td>
    </tr>
  </tbody>
</table>
</div>

### Ganadores repetidos

Para la mayoría de los científicos / escritores / activistas, un Premio Nobel sería el mayor logro de una larga carrera. Pero para algunas personas, uno simplemente no es suficiente, y pocos lo han conseguido más de una vez. ¿Quiénes son estos pocos afortunados? (Al no haber ganado el Premio Nobel yo mismo, asumiré que se trata solo de suerte 😋).

```python
# Seleccionamos los ganadores con 2 o más premios
nobel.groupby("full_name").filter(lambda x: x["full_name"].count() > 1)[["year","full_name","category"]]
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
      <th>year</th>
      <th>full_name</th>
      <th>category</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>19</td>
      <td>1903</td>
      <td>Marie Curie, née Sklodowska</td>
      <td>Physics</td>
    </tr>
    <tr>
      <td>62</td>
      <td>1911</td>
      <td>Marie Curie, née Sklodowska</td>
      <td>Chemistry</td>
    </tr>
    <tr>
      <td>89</td>
      <td>1917</td>
      <td>Comité international de la Croix Rouge (Intern...</td>
      <td>Peace</td>
    </tr>
    <tr>
      <td>215</td>
      <td>1944</td>
      <td>Comité international de la Croix Rouge (Intern...</td>
      <td>Peace</td>
    </tr>
    <tr>
      <td>278</td>
      <td>1954</td>
      <td>Linus Carl Pauling</td>
      <td>Chemistry</td>
    </tr>
    <tr>
      <td>283</td>
      <td>1954</td>
      <td>Office of the United Nations High Commissioner...</td>
      <td>Peace</td>
    </tr>
    <tr>
      <td>298</td>
      <td>1956</td>
      <td>John Bardeen</td>
      <td>Physics</td>
    </tr>
    <tr>
      <td>306</td>
      <td>1958</td>
      <td>Frederick Sanger</td>
      <td>Chemistry</td>
    </tr>
    <tr>
      <td>340</td>
      <td>1962</td>
      <td>Linus Carl Pauling</td>
      <td>Peace</td>
    </tr>
    <tr>
      <td>348</td>
      <td>1963</td>
      <td>Comité international de la Croix Rouge (Intern...</td>
      <td>Peace</td>
    </tr>
    <tr>
      <td>424</td>
      <td>1972</td>
      <td>John Bardeen</td>
      <td>Physics</td>
    </tr>
    <tr>
      <td>505</td>
      <td>1980</td>
      <td>Frederick Sanger</td>
      <td>Chemistry</td>
    </tr>
    <tr>
      <td>523</td>
      <td>1981</td>
      <td>Office of the United Nations High Commissioner...</td>
      <td>Peace</td>
    </tr>
  </tbody>
</table>
</div>

### ¿A qué edad obtienen el premio?

¡La lista de ganadores repetidos contiene algunos nombres ilustres! Nuevamente nos encontramos con [Marie Curie](https://es.wikipedia.org/wiki/Marie_Curie), que obtuvo el premio en física por descubrir la radiación y en química por aislar el radio y el polonio. [John Bardeen](https://es.wikipedia.org/wiki/John_Bardeen) lo obtuvo dos veces en física por sus trabajos sobre transistores y superconductividad, [Frederick Sanger](https://es.wikipedia.org/wiki/Frederick_Sanger) lo obtuvo dos veces en química, y [Linus Carl Pauling](https://es.wikipedia.org/wiki/Linus_Pauling) lo obtuvo primero en química y luego en paz por su trabajo en la promoción del desarme nuclear. También vemos que las organizaciones pueden obtener el premio, ya que tanto la [Cruz Roja](https://es.wikipedia.org/wiki/Cruz_Roja) como el [ACNUR](https://es.wikipedia.org/wiki/Alto_Comisionado_de_las_Naciones_Unidas_para_los_Refugiados) lo han recibido más de una vez.

Pero, ¿cuántos años tienes generalmente cuando obtienes el premio?

```python
# Convertimos birth_date de string a datetime
nobel["birth_date"] = pd.to_datetime(nobel["birth_date"])

# Calculamos la edad de los ganadores de Premio Nobel
nobel["age"] = nobel["year"] - nobel["birth_date"].dt.year

# Creamos gráfico con una regresión de tipo Lowess
sns.lmplot(x="year", y="age", data=nobel, lowess=True, aspect=2, line_kws={'color': 'green'}, height=7)

plt.show()
```

![png](/assets/images/jupyter/historia_visual_ganadores_nobel_files/historia_visual_ganadores_nobel_18_0.png)

### Diferencias de edad entre categorías

¡El gráfico de arriba nos muestra mucho! Vemos que al principio las personas solían tener alrededor de 55 años cuando recibían el premio, pero hoy en día el promedio está más cerca de los 65. Pero hay una gran dispersión en las edades de los galardonados, y aunque la mayoría tienen más de 50 años, algunos son muy jóvenes.

También vemos que la densidad de puntos es mucho más alta hoy en día que a principios del siglo XX: hoy en día se comparten muchos más premios, por lo que hay muchos más ganadores. También vemos que hubo una interrupción en los premios otorgados en torno a la Segunda Guerra Mundial (1939 - 1945).

Veamos las tendencias de edad dentro de las diferentes categorías de premios.

```python
# Misma visualización que arriba, pero con gráficos separados para cada categoría
sns.lmplot(x="year", y="age", data=nobel, col="category", col_wrap=2, lowess=True, aspect=1.5, line_kws={'color': 'green'})

plt.show()
```

![png](/assets/images/jupyter/historia_visual_ganadores_nobel_files/historia_visual_ganadores_nobel_20_0.png)

### Los más jóvenes y más viejos

Vemos que los ganadores del premio de química, medicina y física han envejecido con el tiempo. La tendencia es más fuerte para física: la edad promedio solía ser inferior a 50 años, y ahora es de casi 70. Literatura y la economía son más estables. También vemos que economía es una categoría relativamente nueva. ¡Pero la categoría paz muestra una tendencia opuesta donde los ganadores son cada vez más jóvenes!

En la categoría de paz también tenemos a un ganador que parece excepcionalmente joven alrededor de 2010. Esto plantea las preguntas, ¿quiénes son las personas más viejas y más jóvenes que hayan ganado un Premio Nobel?

```python
# El ganador más viejo del Premio Nobel (hasta 2016)
nobel.nlargest(1, "age")[["year","category","full_name", "age"]]
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
      <th>year</th>
      <th>category</th>
      <th>full_name</th>
      <th>age</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>793</td>
      <td>2007</td>
      <td>Economics</td>
      <td>Leonid Hurwicz</td>
      <td>90.0</td>
    </tr>
  </tbody>
</table>
</div>

```python
# El ganador más joven del Premio Nobel (hasta 2016)
nobel.nsmallest(1, "age")[["year","category","full_name", "age"]]
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
      <th>year</th>
      <th>category</th>
      <th>full_name</th>
      <th>age</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>885</td>
      <td>2014</td>
      <td>Peace</td>
      <td>Malala Yousafzai</td>
      <td>17.0</td>
    </tr>
  </tbody>
</table>
</div>

### ¡Orgullo nacional!

Argentina también cuenta con ganadores del Premio Nobel. Repasemos sus logros.

```python
nobel[nobel["birth_country"] == "Argentina"][["year","category","full_name","birth_city","age"]]
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
      <th>year</th>
      <th>category</th>
      <th>full_name</th>
      <th>birth_city</th>
      <th>age</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>187</td>
      <td>1936</td>
      <td>Peace</td>
      <td>Carlos Saavedra Lamas</td>
      <td>Buenos Aires</td>
      <td>58.0</td>
    </tr>
    <tr>
      <td>236</td>
      <td>1947</td>
      <td>Medicine</td>
      <td>Bernardo Alberto Houssay</td>
      <td>Buenos Aires</td>
      <td>60.0</td>
    </tr>
    <tr>
      <td>513</td>
      <td>1980</td>
      <td>Peace</td>
      <td>Adolfo Pérez Esquivel</td>
      <td>Buenos Aires</td>
      <td>49.0</td>
    </tr>
    <tr>
      <td>548</td>
      <td>1984</td>
      <td>Medicine</td>
      <td>César Milstein</td>
      <td>Bahia Blanca</td>
      <td>57.0</td>
    </tr>
  </tbody>
</table>
</div>
---
Bueno, eso es todo por ahora. Espero que este post te haya resultado interesante y si tenés alguna consulta o sugerencia no dudes en contactarme al mail o seguirme en Twitter.

¡Hasta la próxima!

---

Este trabajo forma parte de los proyectos propuestos en la carrera [Data Scientist with Python](https://www.datacamp.com/tracks/data-scientist-with-python?version=3) en Datacamp.
{: .notice}
