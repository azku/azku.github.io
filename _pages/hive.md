---
layout: page
title: Hive
permalink: /hive/
---

Hadoop sisteman datuak blokeetan banatzen dira makinen cluster batean. Hala ere garatzaile edo erabiltzaileen ikuspuntutik, fitxategi sistema bat bezala funtzionatzen du.
Hori dela eta, astuna gertatu daiteke datuekin konbinazioak egitea. Batzuetan eredu erlazionalak eskaintzen dizkigun abantailetaz baliatzea gustatuko litzaiguke baina Hadoopen eskaintzen duen datu egiturarekin.

Ain zuzen hori litzateke **Hive**. SQL antzeko lengoaia bat, **HiveSQL** izena duena, erabiltzeko aukera baina atzean Hadoop sistema dagoelarik.

## Ezaugarriak
Hivek egitura bat ezartzen du HDFSn biltegiratutako datuen gainean. Egitura horri Schema esaten zaio, eta Hivek bere datu-basean gordetzen du (metastore). Horri esker, automatikoki optimizatzen du gauzatze-plana, eta taulak zatituta erabiltzen ditu kontsulta jakin batzuetan. Fitxategien, kodifikazioen eta datu-iturrien hainbat formatu ere onartzen ditu, hala nola HBase.

Hivek SQLren paradigma zabaltzen du, serializazio-formatuak barne. Kontsulten prozesamendua ere pertsonaliza dezakegu, gure datuekin bat datorren taula-eskema bat sortuz, baina datuak ukitu gabe. SQL jatorrizko balio motekin bakarrik bateragarria den arren (datak, zenbakiak eta kateak, esaterako), Hive-ren tauletako balioak elementu egituratuak dira, adibidez, JSON objektuak edo erabiltzaileak definitutako edozein datu mota edo Java-n idatzitako edozein funtzio.

Hivek asko errazten du datu kopuru handien gaineko galdeketa baina dataset txikietan ez da eraginkorra.

## Ariketa

Hive tresna esku artean erabiltzeko aukera bat Hiven docker irudi ofiziala erabiltzea da. Guk **docker-compose** erabiliko dugu eta postgres datu base bat erabiliko dugu metastore bezala. Horretarako postgresqlrako Java driverrak jaitsi behar dira Docker executatuko dan makinara ``curl -O https://jdbc.postgresql.org/download/postgresql-42.7.4.jar`` eta aldagai batean esportatu ``export POSTGRES_LOCAL_PATH=$PWD/postgresql-42.7.4.jar``. Azkenik, ``docker-compose up -d`` exekutatu aurretik HIVE bertsioa ezarri behar da ``export HIVE_VERSION=4.0.0``.

Ondorengo agindu honek filmen balorazioa daukan fitxategiak jaizten ditu. Guk datu horiek HDFS sisteman sartuko ditugu lehenik eta gero, eskema bat sortu ostean, HIVEn sartuko ditugu.

``curl -O https://files.grouplens.org/datasets/movielens/ml-latest-small.zip``
Deskonprimitu:
``unzip ml-latest-small.zip``
Docker kontenedore barrura kopiatu:
``docker cp ml-latest-small dockerid:/opt/hive/``

Beelinera konektatu 

sortu taula``create table hive_ratings(userId int, movieId int, rating float, tmp int);``


## HIVE nodoa Hadoopeko isardvdi iruditik abiatuta
    ``/etc/hosts`` eta ``/etc/hostname`` eta ``/etc/netplan/50...`` aldatu ostean
``wget https://dlcdn.apache.org/hive/hive-4.0.0/apache-hive-4.0.0-bin.tar.gz``

``tar -xzvf apache-hive-4.0.0-bin.tar.gz``

``rrm apache-hive-4.0.0-bin.tar.gz``

.profile aldatu:
``PATH=/home/hadoop/apache-hive-4.0.0-bin:/home/hadoop/hadoop-3.4.0/bin:/home/hadoop/hadoop-3.4.0/sbin:$PATH``
