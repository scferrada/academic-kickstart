+++
date = "2018-10-29T13:43:23-03:00"
highlight = true
math = true
tags = ["Databases", "Relational Design", "Functional Dependencies"]
title = "Llaves y Dependencias Funcionales"

[header]
  caption = ""
  image = ""

+++

Este post provee de una introducción al concepto de llaves y dependencias funcionales en el campo de la teoría de las bases de datos
relacionales. Está guiado por ejemplos, pero se presentan también algunas de las formalidades matemáticas. Si encuentras algún error
o hay algo que no se logra entender muy bien, por favor deja un comentario debajo.

# Llaves

En el modelo relacional trabajamos con conjuntos de tuplas que se almacenan en una relación dada. Como en un conjunto no se permiten
elementos repetidos es necesario que cada tupla tenga una forma de identificarse y diferenciarse de las otras. La forma de identificar
tuplas es utilizando el concepto de llave:

- **Una llave es un conjunto de atributos de una relación, tal que ningún par de tuplas de la relación tiene los mismos valores en dichos
atributos**

Por ejemplo, consideremos los datos de la Tabla 1. Una llave trivial para cualquier relación es el conjunto de todos los atributos. En
este caso {vino, año, cepa, presentación}. Una llave más pequeña puede ser {vino, cepa} o {año, presentación} (considerando solo estos datos).
Dicho lo anterior, es conveniente recalcar que las llaves no deben seleccionarse condicionadas a los datos, sino que considerando la semántica
de las relaciones y atributos.

|Vino | año | cepa | presentación|
|-----|-----|------|-------------|
|130  | 2018 | Cabernet | 750cc |
|Clos de Paine | 2014 | Cabernet | 750cc |
|Santelena | 2018 | Sauvignon Blanc | 2000cc |

Ahora vamos con un par de definiciones. Una **superllave** es cualquier llave de una relación. Todas las mencionadas en el ejemplo son superllaves.
Muchas veces se prefiere contar con una llave lo más pequeña posible, pues es necesario que podamos comparar rápidamente un par de tuplas
para determinar si son idénticas o no. Para esto, introducimos la noción de **llave candidata**. Una llave candidata es una superllave tal que
ningún subconjunto propio de ella también sea superllave, es decir, es minimal. Por ejemplo {vino, año, cepa, presentación} es una superllave pero no una llave
candidata, pues {vino, cepa} también es una superllave. Esta última tampoco es llave candidata porque {nombre} también es superllave.
Finalmente, {nombre} si es llave candidata, debido a que no hay subconjuntos propios posibles que sean también llave.

Un atributo que pertenece a una llave candidata se llama atributo primo.

Para terminar, una **llave primaria** es alguna de las llaves candidatas que se selecciona como la forma estándar de determinarla igualdad
entre tuplas.

Es necesario notar en este punto, que una relación puede entonces tener más de una llave candidata y estas llaves pueden tener distinta
cantidad de atributos. Consideremos el siguiente esquema para almacenar datos de personas:

- **Persona**(RUT, nombre, fecha_hora_nacimiento, RUT_padre, RUT_madre)

Las llaves candidatas son {RUT} y {RUT_madre, fecha_hora_nacimiento}. [^1]  Como podemos apreciar, la segunda llave es minimal en el sentido
de que no hay un subconjunto de esos atributos que también sea llave, por lo que es una llave candidata a pesar de tener una cantidad de atributos
más grande que {RUT}. Sin embargo, probablemente se prefiera utilizar simplemente el RUT como llave primaria porque en este caso es lo más
"natural" además que siempre es mejor tener menos elementos que comparar.

[^1]: {RUT_padre, fecha_hora_nacimiento} no se considera, pues puede suceder que un hombre tenga hijos con dos mujeres distintas que por coincidencia nacieron el mismo día y a la misma hora.

# Dependencias Funcionales

Dados dos conjuntos de atributos de un esquema relacional $X$ e $Y$, se dice que $Y$ depende funcionalmente de $X$ si para todo
par de tuplas $t_1, t_2$ del esquema se cumple que si $t_1[X]=t_2[X]$ entonces $t_1[Y]=t_2[Y]$. [^2] Es decir, para cada valor de $X$
hay un único valor de $Y$ posible. Cuando $X$ determina funcionalmente a Y se escribe $X \rightarrow Y$. Cuando $Y$ es un subconjunto de $X$ se dice que la dependencia funcional es trivial.

[^2]: En este artículo se considera que si $t$ es una tupla y $X$ es un conjunto atributos, la operación $t[X]$ corresponde a la proyección de los atributos $X$ de la tupla $t$, es decir $\pi_X(t)$

Conocer las dependencias funcionales de un esquema es importante para el diseño relacional, pues nos ayudan a determinar si las
relaciones propuestas por un modelo pueden o no presentar anomalías.

Inicialmente, se sabe que una llave candidata determina funcionalmente a todos los atributos. Sin embargo, es posible que otros atributos
se determinen entre ellos. Supongamos que contamos con la siguiente información de contacto:

- **Contacto**(RUT, email, dirección, comuna, región)

Claramente el RUT e email son llaves candidatas y, por lo tanto, determinan funcionalmente a todo el resto de los atributos. Por otro lado
también sabemos que una comuna se encuentra en una sola región y entonces dos contactos de la misma comuna van a estar necesariamente
en la misma región. Entonces sabemos que {comuna} $\rightarrow$ {región}. Como los nombres de calles pueden repetirse entre comunas,
podemos descartar la dependencia {dirección} $\rightarrow$ {comuna}.

Cuando un esquema cuenta con varias dependencias funcionales, podemos razonar lógicamente sobre ellas y concluir otras dependencias nuevas
y coherentes con el esquema. Existe un conjunto de reglas de razonamiento que nos permite obtener el conjunto completo de dependencias funcionales
de un esquema, a partir de un conjunto inicial. Estas reglas se conocen como los axiomas de Armstrong. Sea $(R, F)$ un esquema de relación $R$ y dependencias funcionales $F$.
Se puede obtener el conjunto de todas las dependencias funcionales deducibles desde $F$, es decir, su clausura $F^+$ aplicando las siguientes reglas:

Sean $X$, $Y$, $Z$, $A$ y $B$ conjuntos de atributos de $R$:

- Reflexión: Si $Y\subseteq X$, entonces $X\rightarrow Y$
- Aumento: Si $X\rightarrow Y$, entonces $XZ\rightarrow YZ$ para cualquier $Z$
- Transitividad: Si $X\rightarrow Y$ e $Y\rightarrow Z$, entonces $X\rightarrow Z$
- Descomposición: Si $X\rightarrow YZ$, entonces $X\rightarrow Y$ y $X\rightarrow Z$
- Composición: Si $X\rightarrow Y$ y $A\rightarrow B$, entonces $XA\rightarrow YB$
- Pseudo transitividad: Si $X\rightarrow Y$ y $YZ\rightarrow A$, entonces $XZ\rightarrow A$
- Autodeterminación: $X\rightarrow X$
- Extensión: Si $X\rightarrow Y$, entonces $X\rightarrow XY$

Los axiomas de Armstrong son especialmente útiles para encontrar las llaves candidatas de las relaciones, pues basta con encontrar
aquellos conjuntos de atributos que determinan funcionalmente a todos los demás. Por ejemplo consideremos $R(A,B,C,D,E)$ y las siguientes
dependencias:

- $A\rightarrow E$
- $BC\rightarrow D$
- $E\rightarrow B$

Por transitividad, sabemos que $A\rightarrow B$ y por aumento que $AC\rightarrow CE$ y $AC\rightarrow BC$. Nuevamente la transitividad nos indica que 
$AC\rightarrow D$. Finalmente, uniendo todos los resultados, tenemos que $AC\rightarrow ABCDE$, por lo que
$AC$ es la llave de $R$. Este proceso de aplicar reglas exhaustivamente a un conjunto de atributos para encontrar todos los otros que
son determinados funcionalmente por ellos, se llama calcular la clausura de ese conjunto de atributos. En este caso $AC^+$.

Tal vez haya parecido un procedimiento antojadizo, pero la intuición detrás del éste es seleccionar los atributos que no aparecen
en el lado derecho de las dependencias para comenzar. En este caso $A$ y $C$.


## Referencias

1. Armstrong, William. Dependency Structures of Data Base Relationships, page 580-583. IFIP Congress, 1974.
2. Ramakrishnan, Raghu, and Johannes Gehrke. Database management systems. McGraw Hill, 2000.