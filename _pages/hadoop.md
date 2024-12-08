---
layout: page
title: Hadoop
permalink: /hadoop/
---

Oinarrizko ubuntu zerbitzari baten instalazioa burutu eta eguneratu ostean, lehenengo pausoa hadoop erabiltzailea sortzea izango da.
Ondoren gako pribatu-publiko pare bat sortu eta sisteman, .ssh/authorized_keys fitxategian atxikituko dugu gako publikoa.
Gure helburua oinarrizko hadoop irudi bat sortzea izango da eta irudi hau erabiliko dugu gero clusterra osatzeko.

Hadoop erabiltzailea erabilita eta ``~/`` gaudela, hadoop jaitsiko dugu
{% highlight shell %}

wget https://dlcdn.apache.org/hadoop/common/hadoop-3.4.0/hadoop-3.4.0.tar.gz
tar -xvzf hadoop-3.4.0.tar.gz
rm hadoop-3.4.0.tar.gz

{% endhighlight %}

Ziurtatu java instalatuta dugula
{% highlight shell %}
sudo apt install openjdk-8-jdk
{% endhighlight %}

Editatu ``~/.bashrc`` eta ondorengo helbideak zehaztu:
{% highlight conf %}
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export HADOOP_HOME=/home/hadoop/hadoop-3.4.0
export HADOOP_INSTALL=$HADOOP_HOME
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export HADOOP_YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin
export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib/native"
{% endhighlight %}

Editatu ``$HADOOP_HOME/etc/hadoop/hadoop-env.sh`` eta ondorengo helbideak zehaztu:
{% highlight conf %}
JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
{% endhighlight %}

Editatu ``$HADOOP_HOME/.profile`` eta gehitu:
{% highlight conf %}
PATH=/home/hadoop/hadoop-3.4.0/bin:/home/hadoop/hadoop-3.4.0/sbin:$PATH
{% endhighlight %}

Ondoren namenode eta datanode direktorioak sortuko ditugu erabiltzailearen erroan:
{% highlight shell %}
mkdir -p ~/hadoopdata/hdfs/{namenode,datanode}
{% endhighlight %}

``$HADOOP_HOME/etc/hadoop/core-site.xml`` editatu:
{% highlight conf %}
<configuration>
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://namenode:9000</value>
  </property>
</configuration>
{% endhighlight %}

``$HADOOP_HOME/etc/hadoop/hdfs-site.xml`` editatu:
{% highlight conf %}
<configuration>
  <property>
    <name>dfs.replication</name>
    <value>1</value>
  </property>
  <property>
    <name>dfs.namenode.name.dir</name>
    <value>file:///home/hadoop/hadoopdata/hdfs/namenode</value>
  </property>
  <property>
    <name>dfs.datanode.data.dir</name>
    <value>file:///home/hadoop/hadoopdata/hdfs/datanode</value>
  </property>
</configuration>
{% endhighlight %}

``$HADOOP_HOME/etc/hadoop/mapred-site.xml`` fitxategia editatu:
{% highlight conf %}
</configuration>
  <property>
    <name>yarn.app.mapreduce.am.env</name>
    <value>HADOOP_MAPRED_HOME=$HADOOP_HOME/bin/hadoop</value>
  </property>
  <property>
    <name>mapreduce.map.env</name>
    <value>HADOOP_MAPRED_HOME=$HADOOP_HOME/bin/hadoop</value>
  </property>
  <property>
    <name>mapreduce.reduce.env</name>
    <value>HADOOP_MAPRED_HOME=$HADOOP_HOME/bin/hadoop</value>
  </property>
  </configuration>
{% endhighlight %}
``$HADOOP_HOME/etc/hadoop/yarn-site.xml`` fitxategia editatu:
{% highlight conf %}
<configuration>
  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  </property>
</configuration>
{% endhighlight %}

``/etc/hosts``
{% highlight conf %}
127.0.0.1 localhost

192.168.85.2 namenode
192.168.85.3 datanode
{% endhighlight %}


Puntu honetararte cluster guztiko makinentzako balioko luke instalazioak. Hemendik aurrera clusterreko makina bakoitza bereizten hasiko ginakete.
hostname aldatu
$HADOOP_HOME/etc/hadoop/workers aldatu
.ssh/config fitxategia ere editatu behar da nodoak alkarren artean komunikatu ahal izateko

## Azterketa
### 1 Atala: Hadoop Core

Datuak ondorengo URLan aurkituko dituzue [https://opendata.euskadi.eus/katalogoa/-/euskadiko-ibaietako-uren-kalitatea-2023an/](https://opendata.euskadi.eus/katalogoa/-/euskadiko-ibaietako-uren-kalitatea-2023an/). Bertan 3 CSV daude: Lagin putntuak, Neuketak (biologikoa) eta Neurketak (fisiko/kimikoa). Fitxategiak Isardeko makinara jaitsi beharko dira.
Jasotako fitxategiak deskonprimitu eta HDFS sistema batean sartu beharko dira. 3 fitxategi mota aurkituko ditugu: Alde batetik langin puntuak izango ditugu. Hemen gainontzeko fitxategietan agertzen diren laginak non hartuak izan diren ikusi daiteke. Beste alde batetik neurketa biologikoak eta neurketa fisiko/kimikoak ditugu. Hauek, hasierako fitxategiko lerro bakoitzeko fitxategi bat izango dute. Kontuz fitxategien burukoekin.
Behin datuak jaitsita ditugunean, HDFS-ra sartzeko, 3 fitxategitan bihurtzea gomendatzen da. Horretarako neurketa biologiko guztiak alde batetik, eta neurketa fisiko/kimikoak, bestetik, fitxategi 2tara atxikitzea gomendatzen da. Hau burututakoan 3 fitxategi izango dituzue eta horiek kontenedore barrura sartzeko docker-composeko namenodean ibaiak izeneko karpeta baten parekatzea prestatu dela ikusiko duzue. Erabili direktorio hori fitxategi horiek antolatzeko.
Inportaziorako, Isardeko makinan aurrekonfiguratutako docker-compose.yml bat daukazue. Honek namenode1 eta 2 datanodeko ingurune bat ezartzen du.
Litekena da sarrera standarretik fitxategia irakurtzerakoan, enkoding arazoak izatea. Maperrean Irakurketa burutu baino lehen ``sys.stin = codecs.getwriter('utf-8')(sys.stdin)`` erabilita konpondu genezake.
Map Reduce algoritmoa erabiliz gune desberdinetako ur tenperatura batezbestekoa atera beharko da. Exekuzioaren ondoren, emaitzak 1_ariketa_emaitzak izeneko direktorio batean utzi beharko dira HDFS barnean. Ondoren direktorio hori fitxategi sistema lokalera atera beharko da, alegia Isardeko makinara (dockerretik kanpo eta erabiltzailearen erroan “~”). 


#### Soluzioa
Zihurtatu **unzip** programa instalatuta dagoela

biologiako fitxategiak alde batetik, eta fisika/kimikakoak bestetik kateatzea gomendatzen da. Kateaketan fitxategi bakoitzean dagoen burukoen lehenengo lerroa ez dugu kopiatu behar:

{% highlight shell %}
head -n 1 agu126_measure_bi.csv > biologia.csv
tail -q -n+2 *_bi.csv >> biologia.csv

head -n 1 agu126_measure_bi.csv > fisika_kimika.csv
tail -q -n+2 *_fq.csv >> fisika_kimika.csv


sudo docker exec -it namenode hdfs dfs -put ibaiak /
sudo docker cp ibaiak/ namenode:/tmp/
{% endhighlight %}
Hadoop clusterra martxan jartzeko ondorengo docker-compose fitxaregia eta konfigurazioak erabiliko ditugu.
{% highlight yaml %}
services:
  namenode:
    image: apache/hadoop:3.4
    container_name: namenode
    hostname: namenode
    user: root
    environment:
      - HADOOP_HOME=/opt/hadoop
    volumes:
      - ./hadoop_namenode:/opt/hadoop/data/nameNode
      - ./hadoop_config:/opt/hadoop/etc/hadoop
      - ./start-hdfs.sh:/start-hdfs.sh
    ports:
      - "9870:9870"
    command: [ "/bin/bash", "/start-hdfs.sh" ]
    networks:
      hdfs_network:
        ipv4_address: 172.20.0.2

  datanode1:
    image: apache/hadoop:3.4
    container_name: datanode1
    hostname: datanode1
    user: root
    environment:
      - HADOOP_HOME=/opt/hadoop
    volumes:
      - ./hadoop_datanode1:/opt/hadoop/data/dataNode
      - ./hadoop_config:/opt/hadoop/etc/hadoop
      - ./init-datanode.sh:/init-datanode.sh
    depends_on:
      - namenode
    command: [ "/bin/bash", "/init-datanode.sh" ]
    networks:
      hdfs_network:
        ipv4_address: 172.20.0.3

  datanode2:
    # Similar configuration to datanode1, with different container_name and IP
    image: apache/hadoop:3.4
    container_name: datanode2
    hostname: datanode2
    user: root
    environment:
      - HADOOP_HOME=/opt/hadoop
    volumes:
      - ./hadoop_datanode2:/opt/hadoop/data/dataNode
      - ./hadoop_config:/opt/hadoop/etc/hadoop
      - ./init-datanode.sh:/init-datanode.sh
    depends_on:
      - namenode
    command: [ "/bin/bash", "/init-datanode.sh" ]
    networks:
      hdfs_network:
        ipv4_address: 172.20.0.4
networks:
  hdfs_network:
    ipam:
      driver: default
      config:
        - subnet: 172.20.0.0/16
{% endhighlight %}
init-datanode.sh
{% highlight shell %}
#!/bin/bash
rm -rf /opt/hadoop/data/dataNode/*
chown -R hadoop:hadoop /opt/hadoop/data/dataNode
chmod 755 /opt/hadoop/data/dataNode
hdfs datanode
{% endhighlight %}
start-hdfs.sh
{% highlight shell %}
#!/bin/bash
if [ ! -d "/opt/hadoop/data/nameNode/current" ]; then
    echo "Formatting NameNode..."
    hdfs namenode -format
fi
hdfs namenode
{% endhighlight %}
core-site.xml
{% highlight xml %}
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://namenode:9000</value>
    </property>
</configuration>
{% endhighlight %}
Eta azkenik hsfs-site.xml
{% highlight xml %}
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>/opt/hadoop/data/nameNode</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>/opt/hadoop/data/dataNode</value>
    </property>
    <property>
        <name>dfs.webhdfs.enabled</name>
        <value>true</value>
    </property>
</configuration>
{% endhighlight %}
Ariketa honen azken puntua burutzeko, mapper bat eta reducer bat programatu behar ditugu. Gainera Streaming erabilita, pythonen programatuko ditugu.
{% highlight python %}
"""mapper.py"""

import sys
import codecs

# input comes from STDIN (standard input)
#sys.stdin.reconfigure(encoding='latin-1')
sys.stin = codecs.getwriter('utf-8')(sys.stdin)
for line in sys.stdin:
    # remove leading and trailing whitespace
    line = line.strip()
    # split the line into words
    sp, date, hour, t, sg, p, species, operator, value, u, add, situation, level, depth = line.split(';')
    # increase counters

    if "Temperatura" in p:
        print('%s;%s' % (sp, value))
{% endhighlight %}

Reducerra
{% highlight python %}
#!/usr/bin/env python
"""reducer.py"""

from operator import itemgetter
import sys

oraingo_lagin_puntua = None
kontua = 0
oraingo_balioa = 0.0
word = None

# input comes from STDIN
for line in sys.stdin:
    # remove leading and trailing whitespace
    line = line.strip()

    # parse the input we got from mapper.py
    lagin_puntua, balioa = line.split(';', 1)


    # this IF-switch only works because Hadoop sorts map output
    # by key (here: word) before it is passed to the reducer
    if oraingo_lagin_puntua == lagin_puntua:
        kontua += 1
        oraingo_balioa += float(balioa)
    else:
        if oraingo_lagin_puntua:
            # write result to STDOUT
            print ('%s\t%s' % (oraingo_lagin_puntua, oraingo_balioa / kontua))
        oraingo_lagin_puntua = lagin_puntua
        oraingo_balioa = float(balioa)
        kontua = 1

# do not forget to output the last word if needed!
if oraingo_lagin_puntua == lagin_puntua:
    print ('%s\t%s' % (oraingo_lagin_puntua, oraingo_balioa / kontua))

{% endhighlight %}
{% highlight shell %}
hadoop jar /opt/hadoop/share/hadoop/tools/lib/hadoop-streaming-3.4.0.jar  -file /opt/hadoop/etc/hadoop/mapper.py    -mapper "python /opt/hadoop/etc/hadoop/mapper.py" -file /opt/hadoop/etc/hadoop/reducer.py   -reducer "python /opt/hadoop/etc/hadoop/reducer.py" -input /ibaiak/fisika_kimika.csv -output /ibaiak/emaitzak
{% endhighlight %}


hdfs dfs -copyToLocal /ibaiak/1_ariketa_emaitzak /tmp/

### 2. Atala: Spark
Aurreko ariketan HDFS-n kargatutako datuetaz baliatuz, oraingo honetan Pysparken burutuko dugu ariketa. Pythonen idatzitako programa bat burutu beharko da, ondorendo eskakizunak beteaz:
3 csv fitxategi antolatu dira HDFSn, bakoitza Pysparkeko dataframe batean kargatu beharko da.
Biologia eta fisika-kimikako 2 fitxategietatik ondorengo zutabeak ez zaizkigu interesatzen: ['Date', 'Hour', 'Type',	'Operator',  'Unit', 'Additional information', 'Situation', 'Level', 'Depth'] DataFramean karga egin eta gero, ezabatu egin beharko dira.
Aurreko paragrafoan aipatutako 2 fitxategiek, zutabe berdinak dituzte. DataFrame biak errenkadak batzen diren beste batean konbinatu beharko dira. Alegia, emaitzan izango dugun DataFrame-ak, aurreko 2 Dataframen errenkada kopuruaren batura izango den errenkada kopurua izango du.
Lortu berri dugun DataFrame-a, lagin puntuak adierazten dituen beste DataFramearekin lotu (join) beharko da. Kontutan izan, izen bereko zutabeak egon daitezkeela. Kasu horretan zutabeak berrizendatu beharko dira.
Daturik galtzen ez dela ziurtatu beharko da. Horretarako programaren exekuzioan imprimatu eragiketa burutu aurretik eta ostean dataframeak dituen datu kopuruak zehaztuz.
Herriz herriko uraren tenperatura media, kontua, maximoa, minimoa eta taldean dauden elementu kopurua kalkulatu. Kontu izan balioen mota (type) zein den.
Agregatutako balioak erakutsi eta lortutako balioak logikoak diren arrazonatu.
Azkenik, lurralde historiko bakoitzean errekek daramaten berunaren konparazioa egin nahi dugu. Horretarako, berunaren (Plomo) neurketak filtratu beharko dira. Lurralde desberdinak zutabeetan egotea eskatzen da eta emaitzetan bataz-besteko balioak aterako dira. Emaitzak ondorengo itxura izan behar du:

#### Soluzioa

{% highlight python %}
import pyspark.sql.functions as f
from pyspark.sql import SparkSession

spark = SparkSession \
    .builder \
    .appName("Python Spark SQL basic example") \
    .config("spark.some.config.option", "some-value") \
    .getOrCreate()

fiki = spark.read.option("delimiter", ";").option("header", True).option("encoding", "ISO-8859-1").csv("hadoop_docker/ibaiak/fisika_kimika.csv",inferSchema =True)
puntuak = spark.read.option("delimiter", ";").option("header", True).option("encoding", "ISO-8859-1").csv("hadoop_docker/ibaiak/samplePoints.csv",inferSchema =True)
bio = spark.read.option("delimiter", ";").option("header", True).option("encoding", "ISO-8859-1").csv("hadoop_docker/ibaiak/biologia.csv",inferSchema =True)

#ezabatu behar direnak
ezabatu = ['Date', 'Hour', 'Type',    'Operator',  'Unit', 'Additional information', 'Situation', 'Level', 'Depth']
bio = bio.drop(*ezabatu)
fiki = fiki.drop(*ezabatu)

fiki.summary().show()
#fiki eta bio batu, bata bestearen azpian jarri
fiki_bio = fiki.union(bio)

#ziurtatu elementu kopurua
print("==================================================================")
print(fiki_bio.summary().collect()[0].__getitem__("Sample Point Code"))
print("==================================================================")
print("==================================================================")

#berizendatu
for old, new in zip(puntuak.columns, [x+"_p" for x in puntuak.columns]):
     puntuak = puntuak.withColumnRenamed(old, new)


#temperaturen analisia
r = puntuak.join(fiki_bio, puntuak.Code_p == f.col("Sample Point Code"), 'inner')
filtroa = r.filter(f.col("parameter").contains("Temperatura"))
taldekatzea = filtroa.groupby("Territory_p")
agregazioak = taldekatzea.agg(f.avg("Value"), f.max("Value"), f.min("Value"),f.count("Value"))


#Berunaren analisia
berun_filtroa = r.filter(f.col("parameter") == "Plomo")
berun_taldekatzea = berun_filtroa.groupby("parameter").pivot("Territory_p").avg("Value").round(2)
berun_taldekatzea = berun_taldekatzea.withColumn("Araba", f.round(berun_taldekatzea[1],2))
berun_taldekatzea = berun_taldekatzea.withColumn("Bizkaia", f.round(berun_taldekatzea[2],2))
berun_taldekatzea = berun_taldekatzea.withColumn("Gipuzkoa", f.round(berun_taldekatzea[3],2))
f.round(berun_taldekatzea[3],2))
berun_taldekatzea.select(berun_taldekatzea[0], berun_taldekatzea.Araba,
berun_taldekatzea.Bizkaia, berun_taldekatzea.Gipuzkoa)
{% endhighlight %}
