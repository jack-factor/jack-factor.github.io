---
layout: post
title:  "Elastic Search"
date:   2019-01-01 12:17:00 -0500
categories: elasticsearch
---
Es un motor de búsqueda full text y de análisis open source de alta escalabilidad. Permite buscar, almacenar, analizar altos volumenes de data en  tiempo muy cerca al real.

## Casos de uso:

* Si tienes una página web puedes almacenar tu catalogo de productos para que tus clientes te compren utilizando un buscador que les brinde sugerencias.
* Recolectar logs e implementar un stack con logstash y kivana
* Correr un sistema de alertas de precio. Scrap precios de un producto. Puede que este producto baje de precio en el siguiente mes y lo analices y te notifique.
* Minería y análisis de datos.


## Conceptos básicos
Near Realtime(NRT): Cerca al tiempo real.
Cluster: Grupo de nodos(servidores)
Node: Un servidor que es parte de un cluster
Index: Colección de documentos
Document: Es la unidad básica que puede estar indexada
Shardd y réplicas:

## Explorando el cluster
Provee de un poderoso api rest para inter-actuar con el cluster
* Revisar el cluster.
* Administración el cluster.
* CRUD en el  index.
* Ejecutar operaciones avanzadas de búsqueda.



Revisar el cluster:

Para revisar la salud del cluster nos vamos apoyar del Api `_cat` 
Ejemplo:

{% highlight python %}

 GET /_cat/health?v\

{% endhighlight %}

Cantida de nodos:

{% highlight python %}

 GET /_cat/nodes?v

{% endhighlight %}


### Administración de índices
Lista de índices

{% highlight python %}

  GET /_cat/indices?v

{% endhighlight %}

Creación de índices

{% highlight python %}

 PUT /customer?pretty

{% endhighlight %}

Estamos creando un índice llamado "customer". El parámetro "pretty" indica que la respuesta será en formato json.
Si listamos los indices podremos obtener la siguiente información:

{% highlight shell %}

health status index          uuid                                                  pri rep docs.count docs.deleted store.size pri.store.size
yellow open   customer jmv3gwMCQCKOxD_oSFkY5Q   5   1          0            0      1.1kb          1.1kb

{% endhighlight %}

Nos indica que nuestro indice tiene 5 particiones primarias 1 replica y 0 documentos además que la salud del índice es amarrillo. 
Esto es porque tiene un indice que no ha sido asignado a ningún nodo todavía. Cuando el nodo sea asignado el estado pasara a verde.

### Indexs y querys

Subir un documento al index "customer" con el ID 1

{% highlight shell %}

 PUT /customer/_doc/1?pretty
 { "name": "John Doe" }

{% endhighlight %}

Consultar documento

{% highlight shell %}

 /customer/_doc/1?pretty

{% endhighlight %}

Delete index

{% highlight shell %}

DELETE /customer?pretty

{% endhighlight %}

consultar:
{% highlight shell %}

 GET /_cat/indices?v

{% endhighlight %}

Resumen de comando
{% highlight shell %}

 PUT /customer
 PUT /customer/_doc/1 
 { "name": "John Doe" } 
 GET /customer/_doc/1 
 DELETE /customer

{% endhighlight %}

### Remplazar

Re llamar el enpoint para modificar el documento

{% highlight shell %}

 PUT /customer/_doc/1?pretty 
 { "name": "Jane Doe" }

{% endhighlight %}

Registrar sin especificar el ID

{% highlight shell %}

 POST /customer/_doc?pretty 
 { "name": "Jane Doe" }

{% endhighlight %}

Al hacer esto el propio elasticsearch le asigna un hash como id
Modificar/Actualizar
Modificando un nombre

{% highlight shell %}

 POST /customer/_doc/1/_update?pretty 
 { "doc": 
            { "name": "Jane Doe" } 
 }

{% endhighlight %}

Moficando y agregando una columna

{% highlight shell %}

 POST /customer/_doc/1/_update?pretty 
 { "doc": 
    { "name": "Jane Doe", "age": 20 } 
 }

{% endhighlight %}

Actualizar determinada columna y autoincrementar

{% highlight shell %}

 POST /customer/_doc/1/_update?pretty 
 { "script" : "ctx._source.age += 5" }

{% endhighlight %}

`ctx._source` refiere al actual documento.

Eliminando documento

{% highlight shell %}

 DELETE /customer/_doc/2?pretty

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
