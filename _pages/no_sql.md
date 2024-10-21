---
layout: page
title: NoSQL
permalink: /nosql/
---


## Ezaugarriak
Baliteke SQLek eskaintzen duen  galdera erraztasuna behar ez izatea. Baliteke datu kopuru handiak izatea esku artea eta baliteke, latentziak garrantzia izatea.

planeta baten tamainako datuak eta zuzeneko atzipena.

Gaitasun hauek Datu-base erlazional sistemetan lortzeko, denormalizazioa, cache memoriak eta shardina bezalako teknikak erabili daitezke. Baina ez dago berez horretarako diseinatuta.

HDFS-rekin zuzenean konparatuko bagenu ordea, latentzia litzateke datu base "handi" hauen abantaila.

## HBASE
HDFS gainean eraikitzen da. Google BigTable ideian modeatua. CRUD api-a eskaintzen du. Auto shardin egiten du.
Lerroetara atzipen azkarra eskaintzen du gako baten arabera.


## MongoDB

huMONGOus datatik dator izena. Dokumentu egiturak gordetzeko aukera egokia baina kontuz, ez zaitzala izenak big datarekin nahastu.
JSON formatuarekin lan egiten du. Datuen eskema izatea ez da derrigorrezkoa.
Mundu honetan: Datu-baseak, kolekzioak eta dokumentuak aurkituko ditugu.
Korporazioen mundura zuzendua.

## MongoDB ariketak

Ondoren  w3resourcen dauden ariketak burutuko ditugu, horretarako bertan eskaintzen dan fitxategia beharko dugu:
https://www.w3resource.com/mongodb-exercises/mongodb-sample-dataset/sample_mflix/movies.zip
Fitxategi hau jaitsi eta deskonprimitu beharko dugu.

MongoDB datubasea Dockerren sortutako irudi ofiziala erabiliz ezarri dezakegu. Horretarako ondorengo ``docker-compose`` fitxategia erabiliko dugu:

{% highlight yaml %}
services:

  mongo:
    image: mongo
    restart: always
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: example
    volumes:
      - ./data:/tmp/data

  mongo-express:
    image: mongo-express
    restart: always
    ports:
      - 8081:8081
    environment:
      ME_CONFIG_MONGODB_ADMINUSERNAME: root
      ME_CONFIG_MONGODB_ADMINPASSWORD: example
      ME_CONFIG_MONGODB_URL: mongodb://root:example@mongo:27017/
      ME_CONFIG_BASICAUTH: false
 {% endhighlight %}

Kontenedorea martxan jarri aurretik, garrantzitsua da ``data`` direktorioan ``movies.json`` fitxategia sartzea, gero kontenedore barruan atzigarri izateko.
Behin martxan dagoela, ``docker exec -it`` erabilita mondo kontenedorera konektatuko gara. Ondoren, ``movies.json`` fitxategia inportatzeko agindua exekutatuko dugu.
{% highlight shell %}
 mongoimport -u root -p example --db movies --collection film --file /tmp/data/movies.json --authenticationDatabase=admin
 {% endhighlight %}
 
Orain datu basea prest dago exekuzioarekin jarraitzeko.

Aurkitu 1893an estreinatu zen "film" bildumako informazio guztia duten film guztiak.
``db.film.find({ year: 1893 })``

Aurkitu "Film" bildumako informazio osoa duten eta 120 minututik gorako iraupena duten film guztiak.
``db.film.find({ "runtime": { $gt: 120 } })``

Aurki itzazu "Short" generoko "Movie" bildumako informazio osoa duten film guztiak.
``db.film.find({ "genres": "Short" })``

 "William K.L. Dicksonek" zuzendutako "Film" bildumako film guztiak berreskuratu eta film bakoitzerako informazio osoarekin
 ``db.film.find({ directors: "William K.L. Dickson" })``

Estatu Batuetan estreinatu zen "film" bildumako film guztiak berreskuratzea eta film bakoitzerako informazio osoarekin.
``db.film.find({ countries: "USA" })``

Informazio osoa duten eta IMDb-n 1000 boto baino gehiago jaso dituzten "film" guztiak berreskuratu.
``db.film.find({ "imdb.votes": { $gt: 1000 } })``
"Film" bildumako film guztiak berreskuratu, baldin eta 7 puntu baino gehiagoko IMDb kalifikazioa badute
``db.film.find({ "imdb.rating": { $gt: 7 } })``

Aurkitu saria jaso duten film guztiak.
``db.movies.find({'awards.wins': {$gt: 0}})``

Aurkitu  title, languages, released, directors, writers, awards, year, genres, runtime, cast eta  countries  datuak MongoDBko "film" bilduman eta gutxienez izendapen bat dutenak.
``db.movies.find({
  "awards.nominations": { $gt: 0 }
}, {
  "title": 1,
  "languages": 1,
  "released": 1,
  "directors": 1,
  "writers": 1,
  "awards": 1,
  "year": 1,
  "genres": 1,
  "runtime": 1,
  "cast": 1,
  "countries": 1
})``

Aurkitu title, languages, released, directors, viewer, writers eta  countries datuak dituzten  "film" bildumako  guztiak, non ikusleen kalifikazioa gutxienez 3 den eta tomatosen 4 baino gutxiagoko dutenak.
``db.movies.find(
{ 'tomatoes.viewer.rating': { $gte: 3, $lt: 4 } },
{ title: 1, languages: 1, released: 1, directors: 1, 'tomatoes.viewer': 1, writers: 1, countries: 1 })``

Aurkitu  "film" bildumako title, languages, fullplot, released, directors, writers eta countries guztiak, "fire" hitza duen trama betea dutenak.
``db.movies.find({
fullplot: { $regex: /fire/i } 
}, 
{ 
title: 1, 
languages: 1, 
fullplot: 1, 
released: 1, 
directors: 1, 
writers: 1, 
countries: 1 
})``


## Cassandra

HBASE-en antzeko ideia baina ez dauka master noderik. Nodo guztiek software berdina exekutatzen dute.
availability over consistency. It will be consisten eventually.

Dockerren node bakarreko Cassandra ezarri. Bonus, 3 nodokoa lortu eskero.

{% highlight yaml %}
services:
  cassandra-p:
    image: "cassandra:latest"
    container_name: "cassandra"
    ports:
      - "9042:9042"
    environment:
      - "MAX_HEAP_SIZE=256M"
      - "HEAP_NEWSIZE=128M"
 {% endhighlight %}
 Behin docker-compose fitxategia sortuta, contenedorea martxan jarri eta cassandraren shelera konektatu gaitezkez:

{% highlight shell %}
docker-compose up -d
docker exec -it cassandra cqlsh
{% endhighlight %}

Cassandra atzitzeko, SQL antzeko lengoaia erabiltzen da baina ez da SQL eta taula bakoitzak derrigorrez izan behar du gako nagusia.


### Pelikulen balorazioa ariketa
Ariketa honetan, CSV fitxategi batzutatik abiatuta, inportazio bat eta ondoren query batzuk burutuko ditugu.
Jaitsi beharreko fitxategiak [url honetan](https://dw9ne0o7jcasn.cloudfront.net/hadoop/HadoopMaterials.zip) aurkitu daitezke. Gure kasuan ratings.data eta u.item fitxategiak inportatu beharko ditugu. Batean filmen datuak azaltzen dira eta bestean balorazioak. Bi fitxategia hauek inportatu eta konbinatuta, helburua puntuazio txarrena jaso duten 10 filmak ateratzea izango da. Adi fitxategiak banaketarako karaktere desberdinak erabiltzen bait dituzte.

Puntuaketetan zutabeak ondorengoak dira: user_id;movie_id;rating;epoch
Pelikulen zutabeak ondorengoak dira: movie id | movie title | release date | video release date |
              IMDb URL | unknown | Action | Adventure | Animation |
              Children's | Comedy | Crime | Documentary | Drama | Fantasy |
              Film-Noir | Horror | Musical | Mystery | Romance | Sci-Fi |
              Thriller | War | Western 
Azken 19 zutabeek, pelikula zein generotakoa den adierazten dute.

Aztertu nola banatuta dauden fitxategiak.

Datu-basera konektatutakoan, lehenengo pausoa gako-espazioa (KEYSPACE) sortzea da:

{% highlight sql %}
DROP KEYSPACE IF EXISTS films;
CREATE KEYSPACE films WITH replication = {'class': 'SimpleStrategy', 'replication_factor': '1'} AND durable_writes = 'True';
USE films;
DROP TABLE if exists ratings;
CREATE TABLE ratings(user_id int, movie_id int, rating int, epoch int, PRIMARY KEY (user_id));
DROP TABLE IF EXISTS movie;

CREATE TABLE movie(movie_id int, title text, release_date text, video_release_date text, url text,unkown text,action int, adventure int, animation int, children int, comedy int, crime int, documentary int, drama int, fantasy int, noir int, horror int, musical int, mystery int, romance int, sci_fi int, thriller int, war int, western int, PRIMARY KEY (movie_id));
#DESCRIBE TABLE ratings;
#SELECT * FROM ratings;

COPY ratings (user_id, movie_id, rating,epoch) FROM '/tmp/ratings.data' WITH  DELIMITER=';';
COPY movie (movie_id, title, release_date, video_release_date, url,unkown,action, adventure, animation, children, comedy, crime, documentary, drama, fantasy, noir, horror, musical, mystery, romance, sci_fi, thriller, war, western)  FROM '/tmp/u.item' WITH DELIMITER='|';

{% endhighlight %}



Bide honetatik jarraituta, laister konturatzen gara Cassandra ez dela datu-base erlazional bat. Beraz datu-basea denormalizatu beharko dugu.

{% highlight sql %}
CREATE TABLE IF NOT EXISTS movies_db.movie_ratings (
    movie_id UUID,
    title TEXT,
    release_date DATE,
    video_release_date DATE,
    url TEXT,
    unknown BOOLEAN,
    action BOOLEAN,
    adventure BOOLEAN,
    animation BOOLEAN,
    children BOOLEAN,
    comedy BOOLEAN,
    crime BOOLEAN,
    documentary BOOLEAN,
    drama BOOLEAN,
    fantasy BOOLEAN,
    noir BOOLEAN,
    horror BOOLEAN,
    musical BOOLEAN,
    mystery BOOLEAN,
    romance BOOLEAN,
    sci_fi BOOLEAN,
    thriller BOOLEAN,
    war BOOLEAN,
    western BOOLEAN,
    user_id UUID,
    rating FLOAT,
    epoch TIMESTAMP,
    PRIMARY KEY (movie_id, user_id)
);
{% endhighlight %}

Eta python erabilitz fitxategi biak batu behar dira. Cassandra python erabilitz atzitzeko pakete bat instalatu behar dugu.
{% highlight python %}
pip install cassandra-driver

{% endhighlight %}
