---
layout: page
title: SQL
permalink: /sql/
---


## Ezaugarriak
Gai honen barruan datu-base erlazionalak sartuko ditugu. Gaur egun SQLren erabilera zabala den arren, oinarrian datu-base normalizatuek oinarri eta garrantzi handia izaten jarraitzen dut. antzeko lengoaia erabiltzen da baina ez da SQL eta taula bakoitzak derrigorrez izan behar du gako nagusia.

## Postgress

### Importazio eta kontsulta ariketa

Behean emandako estekan, komaz bereizitako fitxategi bat aurkituko duzu. Aipatutako fitxategia deskargatu eta zeregin hauek egin
behar dituzu:
1. Lortu CSV fitxategia:
https://opendata.euskadi.eus/contenidos/ds_localizaciones/centros_especialidad_modalidad/es_euskadi/adjuntos/espe_reggen.csv
2. Inportatu testu-dokumentua (CSV formatuan) zure postgreSQL instalaziora. PostgreSQL makina IsardVDI hedapen bitartez
jasoko da. Ikasle bakoitzak bere Datu Basea sortu beharko du BDS_ebal1 _[ZureIzena] izenarekin . Non [Zure izena] zure izena
den. Zutabe kopuru handia dela eta, inportazioa errazteko, zutabeen izenen zerrenda 1. eranskinean ematen da. Gainera,
inportatzerakoan 'iso-8859-1' kodeketa erabili beharko da.
3. Datu-basean 2 eskema sortu beharko dira: Inportatzeko eskema bat (testu motan inportatu) eta datuak normalizatuta uzteko
beste bat. Prozesu osoaren scripta idatzi beharko da eta hau idempotentea izan behar da.
4. Eskema normalizatuan, honako hauek izan behar dira (2 puntu)
1. Taula bat espezialitateen zerrendarekin. Jatorrizko taularen despee zutabea erabili beharko da. Normalizatutako taula
honetan gako nagusia zenbaki natural autoinkrementala izango da: espezialitatea: {id,izena}
2. Taula bat udalerriekin. Jatorrizko fitxategiko dmunie zutabea erabili behar duzu. Normalizatutako taula honetan gako
nagusia zenbaki natural autoinkrementala izango da: udalerria: {id, izena}
3. Taula bat zentroarekin . Taula honek inportatutako taulako erregistro guztiak izan behar ditu. Espezialitatea eta udalerria
taulak erabili beharko dira gako arrotzak sortu eta dagokien identifikadoreak sartzeko. Normalizatutako taula honetan gako
nagusia zenbaki natural autoinkrementala izango da: zentroa: { id, id_udalerria, id_espezialitatea, izena, helbidea}
5. Taula normalizatuetan ondorengo kontsultak egin:
1. Hautatu espezialitatea hutsik duten zentroen izenak (espazio bat dauka espezialitatean).
2. Zerrendatu “SARE-INFORMATIKA-SISTEMEN ADMINISTRAZIOA” irakasten duten zentro guztiak (zentroaren izena eta
biztanleriaren izena). Ez da bikoizturik egon behar.
3. Zerrendatu zentroen artean gehien errepikatzen diren 10 espezialitateak, errepikapen kopurua eta espezialitatearen
izenarekin batera.
4. Udalerrien taulatik abiatuta, “ADMINISTRAZIOA ETA FINANTZAK” espezialitaterik ematen ez den udalerriak zerrendatu.
(54 daude)

#### Ingurunea ezarri
Docker comppose bat erabiliko dugu postgres exekutatzeko. Adi partekatutako karpetari, sortu egin beharko da eta:

{% highlight yaml %}
services:
  db:
    image: postgres
    container_name: postgres
    restart: always
    # set shared memory limit when using docker-compose
    shm_size: 128mb
    #command: bash -c "ls -la /tmp" 
    # or set shared memory limit when deploy via swarm stack
    #volumes:
    #  - type: tmpfs
    #    target: /dev/shm
    #    tmpfs:
    #      size: 134217728 # 128*2^20 bytes = 128Mb
    volumes:
      - ./partekatua:/tmp/partekatua
      - ./postgres-data:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD: 3ia3
{% endhighlight %}
#### Ingurunea erabili
{% highlight sql %}
create schema if not exists inportazioa;
create schema if not exists normalizazioa;

drop table if exists inportazioa.inportazio_zentroa;
create table inportazioa.inportazio_zentroa(
CCEN     text,
NOM     text,
NOME     text,
DGENRC     text,
DGENRE     text,
GENR     text,
MUNI     text,
DMUNIC     text,
DMUNIE     text,
DEPE     text,
DTITUC     text,
DTITUE     text,
DOMI     text,
CPOS     text,
TEL1     text,
TFAX     text,
EMAIL     text,
PAGINA     text,
COOR_X     text,
COOR_Y     text,
NIVE     text,
DNIVEC     text,
DNIVEE     text,
OBSE     text,
ESPE     text,
DESPEC     text,
DESPEE    text);

COPY inportazioa.inportazio_zentroa
FROM '/tmp/partekatua/espe_reggen.csv' CSV header delimiter ';' encoding 'iso-8859-1';


drop table if exists normalizazioa.zentroa;
drop table if exists normalizazioa.espezialitatea;
drop table if exists normalizazioa.udalerria;

create table normalizazioa.espezialitatea(
       id serial primary key,
       izena varchar(200));

insert into normalizazioa.espezialitatea(izena) (select despee from inportazioa.inportazio_zentroa group by despee);


create table normalizazioa.udalerria(
       id serial primary key,
       izena varchar(200));

insert into normalizazioa.udalerria(izena) (select dmunie from inportazioa.inportazio_zentroa group by dmunie);



create table normalizazioa.zentroa(
       id serial primary key,
       izena varchar(200),
       helbidea varchar(200),
       id_udalerria int,
       id_espezialitatea int references normalizazioa.espezialitatea(id),
       CONSTRAINT fk_udalerria FOREIGN KEY (id_udalerria) REFERENCES normalizazioa.udalerria(id));
       
insert into normalizazioa.zentroa(izena, helbidea, id_udalerria, id_espezialitatea)
(select a.nom, a.domi, u.id, e.id
from inportazioa.inportazio_zentroa a inner join normalizazioa.udalerria u on a.dmunie=u.izena
     inner join normalizazioa.espezialitatea e on a.despee=e.izena);


select z.izena
from normalizazioa.zentroa z inner join normalizazioa.espezialitatea e on z.id_espezialitatea=e.id
where e.izena=' ';

select z.izena
from normalizazioa.zentroa z inner join normalizazioa.espezialitatea e on z.id_espezialitatea=e.id
where e.izena='SARE-INFORMATIKA SISTEMEN ADMINISTRAZIOA';

select e.izena, count(*) as kopurua
from normalizazioa.zentroa z left join normalizazioa.espezialitatea e on z.id_espezialitatea=e.id
group by e.izena
order by kopurua desc
limit 10;

select *
from normalizazioa.udalerria u
where not exists (select *
      	  	 from normalizazioa.zentroa z inner join normalizazioa.espezialitatea e on z.id_espezialitatea=e.id
		      where e.izena like 'ADMINISTRAZIOA ETA FINANTZAK' and z.id_udalerria=u.id)
{% endhighlight %}
