---
layout: page
title: Hive
permalink: /hive/
---

Hadoop sisteman datuak blokeetan banatzen dira makinen cluster batean. Hala ere garatzaile edo erabiltzaileen ikuspuntutik, fitxategi sistema bat bezala funtzionatzen du.
Hori dela eta, astuna gertatu daiteke datuekin konbinazioak egitea. Batzuetan eredu erlazionalak eskaintzen dizkigun abantailetaz baliatzea gustatuko litzaiguke baina Hadoopen eskaintzen duen datu egiturarekin.

Ain zuzen hori litzateke **Hive**. SQL antzeko lengoaia bat, **HiveSQL** izena duena, erabiltzeko aukera baina atzean Hadoop sistema dagoelarik.

## Ezaugarriak
Hivek egitura bat ezartzen du HDFSn biltegiratutako datuen gainean. Egitura horri eskema esaten zaio, eta Hivek bere datu-basean gordetzen du (metastore). Horri esker, automatikoki optimizatzen du gauzatze-plana, eta taulak zatituta erabiltzen ditu kontsulta jakin batzuetan. Fitxategien, kodifikazioen eta datu-iturrien hainbat formatu ere onartzen ditu, hala nola HBase.

Hivek SQLren paradigma zabaltzen du, serializazio-formatuak barne. Kontsulten prozesamendua ere pertsonaliza dezakegu, gure datuekin bat datorren taula-eskema bat sortuz, baina datuak ukitu gabe. SQL jatorrizko balio motekin bakarrik bateragarria den arren (datak, zenbakiak eta kateak, esaterako), Hive-ren tauletako balioak elementu egituratuak dira, adibidez, JSON objektuak edo erabiltzaileak definitutako edozein datu mota edo Java-n idatzitako edozein funtzio.

Hivek asko errazten du datu kopuru handien gaineko galdeketa baina dataset txikietan ez da eraginkorra.

## Ariketa

Hive tresna esku artean erabiltzeko aukera bat Hiven docker irudi ofiziala erabiltzea da. Guk **docker-compose** erabiliko dugu eta postgres datu base bat erabiliko dugu metastore bezala. Horretarako postgresqlrako Java driverrak jaitsi behar dira Docker executatuko dan makinara ``curl -O https://jdbc.postgresql.org/download/postgresql-42.7.4.jar`` eta aldagai batean esportatu ``export POSTGRES_LOCAL_PATH=$PWD/postgresql-42.7.4.jar``.
Docker composea ondorengoa izan daiteke:
{% highlight yaml %}
services:
  postgres:
    image: postgres
    restart: unless-stopped
    container_name: postgres
    hostname: postgres
    environment:
      POSTGRES_DB: 'metastore_db'
      POSTGRES_USER: 'hive'
      POSTGRES_PASSWORD: 'password'
    ports:
      - '5432:5432'
    volumes:
      - hive-db:/var/lib/postgresql
    networks:
      - hive


  metastore:
    image: apache/hive:4.0.0
    container_name: metastore
    hostname: metastore
    extra_hosts:
      - "host.docker.internal:host-gateway"
    environment:
      DB_DRIVER: postgres
      SERVICE_NAME: 'metastore'
      SERVICE_OPTS: '-Xmx1G -Djavax.jdo.option.ConnectionDriverName=org.postgresql.Driver
                     -Djavax.jdo.option.ConnectionURL=jdbc:postgresql://postgres:5432/metastore_db
                     -Djavax.jdo.option.ConnectionUserName=hive
p                     -Djavax.jdo.option.ConnectionPassword=password'
    ports:
        - '9083:9083'
    volumes:
        - warehouse:/opt/hive/data/warehouse
        - type: bind
          source: ./postgresql-42.7.4.jar
          target: /opt/hive/lib/postgres.jar
    networks:
      - hive

  hiveserver2:
    image: apache/hive:4.0.0
    depends_on:
      - metastore
    restart: unless-stopped
    container_name: hiveserver2
    environment:
      HIVE_SERVER2_THRIFT_PORT: 10000
      SERVICE_OPTS: '-Xmx1G -Dhive.metastore.uris=thrift://metastore:9083'
      IS_RESUME: 'true'
      SERVICE_NAME: 'hiveserver2'
    ports:
      - '10000:10000'
      - '10002:10002'
    volumes:
      - warehouse:/opt/hive/data/warehouse
    networks:
      - hive

volumes:
  hive-db:
  warehouse:
    
networks:
  hive:
    name: hive

{% endhighlight %}
``docker-compose up -d`` exekutatu eta HIVE matxan izan beharko genuke.

Ondorengo agindu honek filmen balorazioa daukan fitxategiak jaizten ditu. Guk datu horiek HDFS sisteman sartuko ditugu lehenik eta gero, eskema bat sortu ostean, HIVEn sartuko ditugu.

``curl -O https://files.grouplens.org/datasets/movielens/ml-latest-small.zip``
Deskonprimitu:
``unzip ml-latest-small.zip``
Docker kontenedore barrura kopiatu:
``docker cp ml-latest-small dockerid:/opt/hive/``

Beelinera konektatu 

sortu taula``create table hive_ratings(userId int, movieId int, rating float, tmp int);``

**Prozesu hau jarraituta, agregazioek funtzionatzerik ez dut lortu**
## HIVE nodoa Hadoopeko isardvdi iruditik abiatuta
Hadoopeko instalazio batetik abiatuta, HIVE instalatu nahi badugu:

    ``/etc/hosts`` eta ``/etc/hostname`` eta ``/etc/netplan/50...`` aldatu ostean
``wget https://dlcdn.apache.org/hive/hive-4.0.0/apache-hive-4.0.0-bin.tar.gz``

``tar -xzvf apache-hive-4.0.0-bin.tar.gz``

``rrm apache-hive-4.0.0-bin.tar.gz``

.profile aldatu:
``PATH=/home/hadoop/apache-hive-4.0.0-bin:/home/hadoop/hadoop-3.4.0/bin:/home/hadoop/hadoop-3.4.0/sbin:$PATH``


## HIVE-n ml-100k jarduera

u.data eta u.index fitxategiak HDFS sisteman daudelarik ``/ml-100k/`` direktorioi barruan. Datuak HDFSn daudenez **external** gakoa erabiliko dugu.

{% highlight sql %}
create external table ratings(
userID int,
movieID int,
rating int,
epochseconds int)
row format delimited fields terminated by '\t';

LOAD DATA INPATH '/ml-100k/u.data' INTO TABLE ratings;
{% endhighlight %}

Behin taulak lotutakoan (LOAD), Hadoopen bere HDFS partera eramaten ditu. kontuz ibili ezabatzerakoan!

{% highlight sql %}
CREATE EXTERNAL TABLE names(
movieID int,
title string,
release_date string,
c1 string,
url string,
c01 string,
c02 string,
c03 string,
c04 string,
c05 string,
c06 string,
c07 string,
c08 string,
c09 string,
c10 string,
c11 string,
c12 string,
c13 string,
c14 string,
c15 string,
c16 string,
c17 string,
c18 string,
c19 string
)
    row format delimited fields terminated by '|';

LOAD DATA INPATH '/ml-100k/u.item' INTO TABLE names;
{% endhighlight %}

Datuetan dauden pelikuletatik puntuazio onena izan dutenak atera nahi dugu ondoren. Kontutan izan behar da, posible dela pelikularen batek puntuazio bakarra izatea. Horrelako kasuka baztertzeok, datuetan aterako ditugun pelikulek gutxienez 10 puntuazio jaso izan beharko dute.

{% highlight sql %}
create view if not exists avgRatings AS
select movieid, avg(rating) as avgRating, count(movieid) as ratingCount
from ratings
group by movieid
order by avgRating;

select n.title, avgRating
from avgRatings t join names n on t.movieid=n.movieID
where ratingCount>10
order by avgRating desc;
{% endhighlight %}

## Zipkodeak jarduera
Zipkodeak fitxategi [honetatik]( https://github.com/spark-examples/spark-scala-examples/blob/master/src/main/resources/zipcodes20.csv) abiatuta, ondorengo ariketan, fitxategi hau HIVEra inportatu eta posta_kodearen arabera HIVEn zatitidura **partition** sortuko ditugu. Ariketa honetan arazo bat dago. Alegia, zaitu gura dogun zutabea ez dela azkena. Zatidurak azkenean definitzen direnez, aldi baterako taula bat erabili beharko dugu eraldaketa burutzeko.


{% highlight sql %}
drop table if exists zipcodes_aldi_baterako;
create external table zipcodes_aldi_baterako(
       id int,
       herrialdea string,
       herria string,
       posta_kodea int,
       estatuta string)
row format delimited fields terminated by ','; 



drop table if exsists zipcodes;
create external table zipcodes(
       id int,
       herrialdea string,
       herria string,
       estatuta string)
PARTITIONED BY(posta_kodea int)
row format delimited fields terminated by ',';

INSERT INTO zipcodes PARTITION(posta_kodea)
SELECT id, herrialdea, herria, estatuta, posta_kodea
FROM zipcodes_aldi_baterako;


LOAD DATA INPATH '/tmp/zipcodes20.csv' INTO TABLE zipcodes;

{% endhighlight %}
