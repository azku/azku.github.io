---
layout: page
title: Cassandra
permalink: /cassandra/
---

## Pelikulen balorazioa ariketa
Ariketa honetan, CSV fitxategi batzutatik abiatuta, inportazio bat eta ondoren query batzuk burutuko ditugu.
Jaitsi beharreko fitxategiak [url honetan](https://dw9ne0o7jcasn.cloudfront.net/hadoop/HadoopMaterials.zip) aurkitu daitezke. Gure kasuan ratings.data eta u.item fitxategiak inportatu beharko ditugu.

Lehenengo pausoa Cassandra martxan jartzea da:
{% highlight shell %}
docker run --name cass_cluster cassandra:latest
{% endhighlight %}

Bigarren exekuzioan
{% highlight shell %}
docker exec -it cass_cluster cqlsh 
{% endhighlight %}


Orain Cassandraren shelera konektatu eta KEYSPACEa sortu beharko dugu lehenik:



CREATE TABLE ratings ( 
    id UUID PRIMARY KEY, 
    user_id int,
    movie_id int,
    rating int,
    epochseconds int
); 
COPY ratings (user_id, movie_id, rating,epochseconds) FROM 'u.data WITH HEADER = true;

