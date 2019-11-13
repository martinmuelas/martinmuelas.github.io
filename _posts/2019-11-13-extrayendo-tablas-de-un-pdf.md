---
layout: single
title: "Extrayendo tablas de un PDF"
date: 2019-11-13
excerpt: Veamos como podemos extraer datos de archivos PDF utilizando `tabula-py`.
header:
  teaser: ../assets/images/posts/2019-11-13/pdf_to_csv.jpg
---

{% include figure image_path="../assets/images/posts/2019-11-13/pdf_to_csv.jpg" alt="dibujo archivo pdf a archivo csv" %}

Hace unos días, pensando en algún caso de estudio para poder jugar un poco con DataFrames y gráficos, se me ocurrió que podía intentar un análisis sobre mis ingresos mensuales y algunas variables económicas.

En principio pensé: _tengo archivados mis recibos de haberes de los últimos 7 u 8 años. Es una buena cantidad de datos para jugar_. Pero, había que extraerlos de alguna manera desde los archivos PDF en los que los tenía archivados. **OK, challenge accepted!** 😎

Comencé a navegar un poco y con unos pocos clicks llegué a [TabulaPDF](https://tabula.technology/), una app que se autodefine como _"Una herramienta para liberar tablas de datos bloqueadas dentro de archivos PDF"_. ¡Suena perfecto para lo que necesito!
