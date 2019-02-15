---
layout: post
title:  "Elastic Search"
date:   2019-01-01 12:17:00 -0500
categories: elasticsearch
---
Es un motor de búsqueda full text y de análisis open source de alta escalabilidad. Permite buscar, almacenar, analizar altos volúmenes de data en  tiempo muy cerca al real.

## Casos de uso:

* Si tienes una página web puedes almacenar tu catalogo de productos para que tus clientes te compren utilizando un buscador que les brinde sugerencias.
* Recolectar logs e implementar un stack con logstash y kivana
* Correr un sistema de alertas de precio. Scrap precios de un producto. Puede que este producto baje de precio en el siguiente mes y lo analices y te notifique.
* Minería y análisis de datos.


## Conceptos básicos:

Hay algunos conceptos básicos que se deben entender antes de continuar:

* Near Realtime(NRT): Cerca al tiempo real.
* Cluster: Grupo de nodos(servidores).
* Node(nodo): Un servidor que es parte de un cluster.
* Index: Colección de documentos.
* Document: Es la unidad básica que puede estar indexada.
* Shards y réplicas: Elasticsearch puede almacenar enormes cantidades de información en un nodo, sobre pasando los limites del hardware dándonos como consecuencia tiempos de respuesta muy lentos.
Para resolver este problema elasticsearch ofrece subdividir un index en varias partes llamadas `shards`.
Cuando se crea un index puedes definir la cantidad de shards que tu quieras. [Leer más aquí][fuente-02]

## Instalación:

La documentación nos explica como instalar elasticsearch en distintos sistemas operativos. [Leer más aquí][fuente-03]

## Explorando el cluster:

Elasticsearch nos provee de un poderoso api-rest para interactuar con el cluster. Las cosas que puedes hacer con el api-rest son las siguientes:

* Revisar la salud, el estado y estadísticas del cluster, nodo e index 
* Administración de la data y métadata.
* CRUD en el index y operaciones de búsqueda.
* Ejecutar operaciones avanzadas de búsqueda.


## Revisar el cluster:

Para revisar la salud del cluster nos vamos apoyar del Api `_cat` 
Ejemplo:

{% highlight python %}

    curl -X GET "localhost:9200/_cat/health?v"

{% endhighlight %}

Cuando preguntamos por la salud de nuestro cluster notaremos que en el `status` obtendremos uno de estos tres colores:
* Verde: Todo esta bien (El cluster funciona completamente).
* Amarillo: Todo la data esta disponible pero las replicas no están disponibles (El cluster funciona completamente).
* Rojo: Algo de la data no esta disponible por alguna razón ( El cluster funciona parcialmente).

Por defecto elasticsearch trae un nodo con 0 particiones.
listar la cantida de nodos:

{% highlight python %}

    curl -X GET "localhost:9200/_cat/nodes?v"

{% endhighlight %}


### Administración de índices

Lista de índices. En un pricipicio no encontraremos indices.

{% highlight python %}

    curl -X GET "localhost:9200/_cat/indices?v"

{% endhighlight %}

### Creación de índices

{% highlight python %}

     curl -X PUT "localhost:9200/customer?pretty"

{% endhighlight %}

Estamos creando un índice llamado "customer". El parámetro "pretty" indica que la respuesta será en formato json.
Si listamos los indices podremos obtener la siguiente información:

{% highlight shell %}

health status index          uuid              pri rep docs.count ...
yellow open   customer jmv3gwMCQCKOxD_oSFkY5Q   5   1          0  ...

{% endhighlight %}

Nos indica que nuestro indice tiene 5 particiones primarias 1 replica y 0 documentos además que la salud del índice es amarillo. 
Esto es porque tiene un replica que no ha sido asignado a ningún nodo todavía. Cuando el nodo sea asignado el estado pasara a verde. Por defecto elasticsearch nos crea una replica.

### Indexs y querys

Subir un documento al index "customer" con el ID 1.
Después de insetar nuestro primer documento se creará automáticamente el index customer.

{% highlight shell %}

 curl -X PUT "localhost:9200/customer/_doc/1?pretty" -H 'Content-Type: application/json' -d '
 {"name": "John Doe"}'

{% endhighlight %}

Consultar el documento recién indexado

{% highlight shell %}

 curl -X GET "localhost:9200/customer/_doc/1?pretty"

{% endhighlight %}

Eliminar índice customer

{% highlight shell %}

 curl -X DELETE "localhost:9200/customer?pretty"

{% endhighlight %}

Si todo ha salido bien al consultar ya no veremos nuestro índice customer:

{% highlight shell %}

 curl -X GET "localhost:9200/_cat/indices?v"

{% endhighlight %}

### Resumen de comando
{% highlight shell %}

 curl -X PUT "localhost:9200/customer"

 curl -X PUT "localhost:9200/customer/_doc/1" -H 'Content-Type: application/json' -d'
 {
  "name": "John Doe"
 }'
 
 curl -X GET "localhost:9200/customer/_doc/1"
 
 curl -X DELETE "localhost:9200/customer"

{% endhighlight %}

### Remplazar

Re llamar el enpoint para modificar el documento

{% highlight shell %}

 curl -X PUT "localhost:9200/customer/_doc/1?pretty" -H 'Content-Type: application/json' -d 
 '{"name": "John Doe"}'


{% endhighlight %}

Registrar sin especificar el ID

{% highlight shell %}

 curl -X POST "localhost:9200/customer/_doc?pretty" -H 'Content-Type: application/json' -d
 '{
  "name": "Jane Doe"
 }'


{% endhighlight %}

Al hacer esto el propio elasticsearch le asigna un hash como id

### Modificar/Actualizar
Modificando un nombre

{% highlight shell %}

 curl -X POST "localhost:9200/customer/_doc/1/_update?pretty" -H 'Content-Type: application/json' -d
 '{"doc": { "name": "Jane Doe" }}'

{% endhighlight %}

Moficando y agregando una columna

{% highlight shell %}

 curl -X POST "localhost:9200/customer/_doc/1/_update?pretty" -H 'Content-Type: application/json' -d
 '{"doc": { "name": "Jane Doe", "age": 20 }}'


{% endhighlight %}

Actualizar determinada columna y autoincrementar

{% highlight shell %}

 curl -X POST "localhost:9200/customer/_doc/1/_update?pretty" -H 'Content-Type: application/json' -d
 '{"script" : "ctx._source.age += 5"}'

{% endhighlight %}

`ctx._source` refiere al actual documento.

Eliminando documento

{% highlight shell %}

 curl -X DELETE "localhost:9200/customer/_doc/2?pretty"

{% endhighlight %}


### Modo batch

Elasticksearch te permite insertar, actualizar, eliminar en grupo para no hacer tanto viajes al servidor
Ejemplo de indexación múltiple de documentos para ello se va utilizar el `_bulk`

{% highlight shell %}

 POST /customer/_doc/_bulk?pretty 
 {"index":{"_id":"1"}} {"name": "John Doe" } {"index":{"_id":"2"}} {"name": "Jane Doe" }

{% endhighlight %}

Ejemplo de actualización y eliminación

{% highlight shell %}

 POST /customer/_doc/_bulk?pretty 
 {"update": {"_id":"1"}} 
 {"doc": { "name": "John Doe becomes Jane Doe" } } 
 {"delete":{"_id":"2"}}

{% endhighlight %}


### Fuente:
[https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html][fuente-01]


[fuente-01]: https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html
[fuente-02]: https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started-concepts.html
[fuente-03]: https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started-install.html