+++
date = "2017-10-26T18:43:23-03:00"
math = false
tags = ["IMGpedia", "ISWC", "Linked Data", "Multimedia"]
title = "IMGpedia, la importancia de incluir multimedia en la Web de Datos"
authors = ["admin"]
draft = false

+++

IMGpedia es una base de datos enlazados que extrae características visuales a las imágenes de Wikimedia Commons y
que provee enlaces a DBpedia al contexto de la imagen, permitiendo así realizar consultas visuo-semánticas. IMGpedia
fue mi tesis de magíster guiada por Benjamin Bustos y Aidan Hogan. Muy probablemente será mi tesis de doctorado también.
En este post les voy a contar de qué se trata IMGpedia y mi experiencia compartiendo este trabajo en [ISWC 2017](https://iswc2017.semanticweb.org) en Viena, Austria.

La idea que motiva el trabajo es incluir fuertemente el análisis de contenido multimedia en la Web de Datos, tomando
en cuenta que la información multimedia es parte fundamental para la experiencia de los usuarios en la Web. IMGpedia
nos permitiría poder hacer consultas de similitud visual entre imágenes, obtener imágenes que cumplan un cierto predicado
y mezclar ambos enfoques, lo que llamamos una consulta _visuo-semántica_. Por ejemplo, nos permitiría responder 
la siguiente consulta: _dada una imagen de la catedral de Cusco, obtener imágenes similares de catedrales europeas_.

Para lograr el objetivo, debemos seguir la siguiente metodología:

- Obtener las imágenes para procesarlas localmente.
- Calcular los descriptores visuales (vectores de características) de las imágenes.
- Obtener estáticamente relaciones de similitud entre las imágenes.
- Enlazar las imágenes con otras fuentes de información ([DBpedia](http://dbpedia.org), [Wikidata](https://www.wikidata.org), [Freebase](http://freebase.com)).
- Publicar todo como RDF, a través de un terminal SPARQL.

Descargar los 14,7 millones de imágenes nos tomó 40 días con múltiples conexiones. El tamaño total de las imágenes es de
22 Terabytes. Una vez descargadas, calculamos los descriptores visuales, en particular calculamos histogramas de grises,
histogramas de la orientación de los bordes y un descriptor basado en color. Puedes encontrar más detalles sobre los descriptores y código visitando
el [GitHub](https://github.com/scferrada/imgpedia) del proyecto (en inglés).

Como se puede ver en la Figura 1, nuestro objetivo es enlazar las imágenes que sean visualmente similares. Para hacer esto,
tomamos un descriptor y encontramos sus 10 vecinos más cercanos, pues asumimos que si dos vectores están cerca, es por que son
similares. Esta decisión de tomar los _k_ vecinos más cercanos ha generado mucha discusión, pero si están interesados, hay una reflexión
al respecto al final del post.

<img src="/img/similarity.PNG" alt="Figura 1: Similitud entre imágenes" width="90%"/>
<center>**Figura 1:** Similitud entre imágenes</center>

Luego, necesitamos obtener contexto de la imagen: saber qué es, cómo se llama lo que aparece en ella o con qué está relacionada.
Para esto, usamos un [dump](https://dumps.wikimedia.org/) de la Wikipedia en inglés para reunir los pares `(nombre_img, nombre_wiki)`
de modo que la imagen aparece en la wiki. Entonces enlazamos la imagen con el recurso de DBpedia relacionado al articulo de Wikipedia correspondiente.
Por ahora, nada nos asegura que la imagen realmente contenga a la entidad con la que está relacionada, pero sí podemos
decir que la imagen está asociada a la entidad de alguna forma u otra. En la Figura 2 se muestra lo que se pretende hacer, por ejemplo
si una imagen aparece en la [wiki de Quentin Tarantino](http://en.wikipedia.org/wiki/Quentin_Tarantino), entonces enlazamos la imagen con el [recurso de Tarantino en DBpedia](http://dbpedia.org/resource/Quentin_Tarantino).

<img src="/img/context.PNG" alt="Figura 2: Obtener contexto de la imagen" width="90%"/>
<center>**Figura 2:** Obtener contexto de la imagen</center>

Finalmente, publicamos todo en formato [RDF](https://www.w3.org/2001/sw/wiki/RDF) que puede ser consultado a través de un terminal público de [SPARQL](https://www.w3.org/TR/sparql11-query/).
Los datos tienen un cierto esquema para modelar clases y atributos que puede ser encontrado [aquí](http://imgpedia.dcc.uchile.cl/ontology)
En [este terminal](http://imgpedia.dcc.uchile.cl/sparql), podemos hacer por ejemplo las siguientes consultas:

- Obtener todas las imágenes similares a la foto de Hopsten Marktplatz:

```
SELECT DISTINCT ?Target
WHERE {
    im:Hopsten_Marktplatz_3.jpg imo:similar ?Target .
}
```

<img src="/img/q1.PNG" alt="Figura 3: Resultados de la consulta a" width="90%"/>
<center>**Figura 3:** Resultados de la consulta a</center>

- Obtener imágenes de las pinturas hechas el siglo XVI, que están en exposición en el Louvre (ojo que también
podemos obtener otros datos como el nombre de la pintura, quién la pintó, etc.):

```
SELECT DISTINCT ?url ?label
WHERE {
  SERVICE <https://dbpedia.org/sparql> {
    ?res a yago:Wikicat16th-centuryPaintings ;
      dcterms:subject dbc:Paintings_of_the_Louvre ;
      rdfs:label ?label .
    FILTER(LANG(?label)="en")  }
  ?img imo:associatedWith ?res ;
      imo:fileURL ?url .  
}
```


<img src="/img/q2.PNG" alt="Figura 4: Resultados de la consulta b" width="90%"/>
<center>**Figura 4:** Resultados de la consulta b</center>

- Finalmente podemos combinar ambos tipos de preguntas en una consulta que llamamos _visuo-semántica_, por ejemplo
entre todas las imágenes de catedrales europeas, obtener imágenes similares que sean museos:

```
SELECT DISTINCT ?source ?target
WHERE {
  SERVICE <https://dbpedia.org/sparql> {
    ?sres dcterms:subject/skos:broader* dbc:Roman_Catholic_cathedrals_in_Europe } 
  ?source imo:associatedWith ?sres ;
          imo:similar ?target .
  ?target imo:associatedWith ?tres ;
          imo:fileURL ?urlt .
  SERVICE <https://dbpedia.org/sparql> {
    ?tres dcterms:subject ?subject .
    FILTER(CONTAINS(STR(?sub), “Museum”))} 
}
```

<img src="/img/q3.PNG" alt="Figura 5: Resultados de la consulta c" width="90%"/>
<center>**Figura 5:** Resultados de la consulta c</center>

Tras mi defensa del Magister, este trabajo fue aceptado en el track de recursos de la 16ta Conferencia Internacional de la Web Semántica (ISWC) y viajé
a Viena a presentarlo, puedes encontrar las diapositivas [aquí](/pptx/iswc2017.pptx). También participamos en la sesión
de pósters, donde recibimos feedback y preguntas muy interesantes, puedes ver el póster [aquí](/pdf/imgpediaISWCposter.pdf).
Tanto la presentación como la sesión de pósters fueron una gran oportunidad para validar nuestro trabajo, para saber cómo mejorarlo
y para saber qué cree la comunidad que sería útil agregar. En resumen, dejo algunos puntos:

- Redes Neuronales: el 90% de las personas que pasaron por mi póster me dijeron que no podían faltar. Ya sea para
clasificar el contenido, como para obtener otros descriptores visuales o incluso para predecir triples con hechos sobre las imágenes.
- Consultas por rango: muchos, al igual que yo, notan que el tener _k_ vecinos más cercanos por imagen es bastante restrictivo:
hay relaciones interesantes que se pierden y forzamos a otras imágenes a ser similares a _k_ otras. Sin embargo,
hacerlo de esta forma en la primera versión nos ayudó a comprender cómo se comporta la distribución de distancias entre
imágenes similares, por lo que podemos proponer un umbral más ajustado en una próxima versión.
- Enlazar a otas bases de conocimiento: DBpedia es una fuente de información muy útil, pero ha resultado ser poco flexible
para nuestros propósitos. Muchos piensan que enlazar con otras bases de conocimiento como Wikidata o Yago, puede enriquecer
y darle mayor expresividad a las consultas de IMGpedia.

¡Gracias a todos los que nos fueron a ver al stand!

<img src="/img/posteriswc17.jpg" alt="Figura 6: Sesión de pósters" width="90%"/>
<center>**Figura 6:** Sesión de pósters</center>

Para terminar con esta historia, el último día de conferencia anuncian a los ganadores de los mejores papers publicados en la conferencia.
¡[IMGpedia ganó dos premios](https://iswc2017.semanticweb.org/program/awards/)! Ganamos el premio al mejor póster y el premio al mejor paper de estudiante en el track de recursos (junto con BiOnIC).
Este reconocimiento significa mucho para mi y para mi carrera de científico, ¡vaya forma de comenzar! Y más aún, porque los otros nominados
son varios estudiantes de doctorado de centros y universidades reconocidos a nivel mundial, como la Universidad de Standford.
Más aún cuando mi trabajo es una tesis de magíster, es increíble que haya competido contra otros trabajos de tan alto nivel.
Además este premio, como el [mejor paper de estudiantes del track de investigación de ISWC](https://iswc2017.semanticweb.org/paper-359/) es un claro mensaje de que
tenemos que seguir trabajando en multimedia, no podemos seguir dejando estos datos de lado.
Ahora queda seguir trabajando y mejorando IMGpedia, para que se convierta en la base de conocimiento estándar para las imágenes de la Web.

Quiero destacar también el nivel de ciencia que se desarrolla en los institutos y centros de investigación chilenos financiados
por la [Iniciativa Científica Milenio](http://www.iniciativamilenio.cl/en/home/), en particular en el Núcleo Milenio [Centro
de investigación de la Web Semántica](http://ciws.cl), que en total nos llevamos 3 premios en ISWC (los dos de IMGpedia y el premio a mejor paper de investigación
para Jorge Pérez).

<img src="/img/awards.jpg" alt="Figura 7: Aidan y yo recibiendo el premio a mejor paper de estudiante" width="90%"/>
<center>**Figura 7:** Aidan y yo recibiendo el premio a mejor paper de estudiante</center>

Solo me falta agradecer a todos los que hicieron este (arduo) trabajo posible, en especial a mis profesores guía, Benjamin y Aidan; a
Sergio Aguilera por ayudarme tanto con la mantención del servidor; a Camila Faúndez por el trabajo enlazando las imágenes a sus respectivos
artículos; al Núcleo Milenio [Centro de Investigación de la Web Semántica](http://ciws.cl), por su apoyo financiero y académico; y a toda mi familia y amigos por su constante apoyo, motivación y cariño.

_________________________________________________________

_Sobre los 10 vecinos más cercanos_: Es cierto que al tomar esta decisión suceden dos cosas indeseables. Primero, es posible que
se pierdan relaciones de similitud relevantes, es decir que el 11mo, 12mo, etc. sean muy similares también. Segundo,
estamos forzando que si hay imágenes que no se parecen a nada, se parezcan a al menos 10. La justificación detrás de esta decisión tiene dos partes.
Por un lado lo que pretendemos es mantener una cantidad acotada de triples RDF, después de todo, esta es la primera versión de 
IMGpedia y necesitábamos probar el concepto primero. Por otro lado, tener estas relaciones de los 10 vecinos más cercanos y sus distancias
en los diferentes espacios vectoriales nos dan una visión más amplia sobre la distribuciónde las distancias, permitiéndonos en el
futuro utilizar una consulta por rango, utilizando estrategias de indexamiento y un umbral de distancia apropiado para cada descriptor.
