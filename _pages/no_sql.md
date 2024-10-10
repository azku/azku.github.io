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

## CASANDRA
HBASE-en antzeko ideia baina ez dauka master noderik. Nodo guztiek software berdina exekutatzen dute.
availability over consistency. It will be consisten eventually.


## MongoDB

huMONGOus datatik dator izena. Dokumentu egiturak gordetzeko aukera egokia baina kontuz, ez zaitzala izenak big datarekin nahastu.
JSON formatuarekin lan egiten du. Datuen eskema izatea ez da derrigorrezkoa.
Mundu honetan: Datu-baseak, kolekzioak eta dokumentuak aurkituko ditugu.
Korporazioen mundura zuzendua.
