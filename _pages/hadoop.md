---
layout: page
title: Hadoop Ubuntu Server batean
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
Datuak ondorengo URLan aurkituko dituzue [https://opendata.euskadi.eus/katalogoa/-/euskadiko-ibaietako-uren-kalitatea-2023an/](https://opendata.euskadi.eus/katalogoa/-/euskadiko-ibaietako-uren-kalitatea-2023an/)
Jasotako fitxategiak deskonprimitu eta HDFS sistema batean sartu beharko dira. 3 fitxategi mota aurkituko ditugu: Alde batetik langin puntuak izango ditugu. Hemen gainontzeko fitxategietan agertzen diren laginak non hartuak izan diren ikusi daiteke. Beste alde batetik neurketa biologikoak eta neurketa fisiko/kimikoak ditugu. Hauek, hasierako fitxategiko lerro bakoitzeko fitxategi bat izango dute.
Behin datuak jeitsita ditugunean, HDFS-ra sartzeko, 3 fitxategitan bihurtzea gomendatzen da. Horretarako neurketa biologiko guztiak alde batetik, eta neurketa fisiko/kimikoak, bestetik, fitxategi 2tara atzxikitzea gomendatzen da.
Inportaziorako, Isardeko makinan aurrekonfigurtutako docker-compose.yml bat daukazue. Honek namenode1 eta 2 datanodeko ingurune bat ezartzen du.
Litekena da sarrera standarretik fitxategia irakurtzerakoan, enkoding arazoak izatea. Irakurketa burutu baino lehen ``sys.stdin.reconfigure(encoding='latin-1')`` erabilita konpondu genezake.
Map Reduce algoritmoa erabiliz gune desberdinetako ur tenperatura mediak atera beharko dira

#### Soluzioa
Zihurtatu **unzip** programa instalatuta dagoela

biologiako fitxategiak alde batetik, eta fisika/kimikakoak bestetik kateatzea gomendatzen da: ``cat *_fq.csv > fisika_kimika.csv``
HDFSra inportatzeko


{% highlight shell %}
head -n 1 agu126_measure_bi.csv > biologia.csv
tail -q -n+2 *_bi.csv >> biologia.csv

head -n 1 agu126_measure_bi.csv > fisika_kimika.csv
tail -q -n+2 *_fq.csv >> fisika_kimika.csv


sudo docker exec -it namenode hdfs dfs -put ibaiak /
sudo docker cp ibaiak/ namenode:/tmp/
{% endhighlight %}
Hadoop clusterra martxan jartzeko ondorengo docker-compose fitxaregia eta konfigurazioak erabiliko ditugu.
{% highlight xml %}
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

### 2. Atala

#### Soluzioa

{% highlight python %}
import pyspark.sql.functions as f

fiki = spark.read.option("delimiter", ";").option("header", True).option("encoding", "ISO-8859-1").csv("hadoop_docker/ibaiak/fisika_kimika.csv")
puntuak = spark.read.option("delimiter", ";").option("header", True).option("encoding", "ISO-8859-1").csv("hadoop_docker/ibaiak/samplePoints.csv")
bio = spark.read.option("delimiter", ";").option("header", True).option("encoding", "ISO-8859-1").csv("hadoop_docker/ibaiak/biologia.csv")
#ezabatu behar direnak
bio_ezabatu = ['Date', 'Hour', 'Type',  'Parameter',  'Operator',  'Unit', 'Additional information', 'Situation', 'Level', 'Depth']
fiki_ezabatu =
bio = bio.drop(*bio_ezabatu)

print(fiki.summary().collect()[0].__getitem__("Sample Point Code"))


#berizendatu 
for old, new in zip(fiki.columns, [x+"_fiki" for x in fiki.columns]):
    fiki = fiki.withColumnRenamed(old, new)
for old, new in zip(bio.columns, [x+"_bio" for x in bio.columns]):
    bio = bio.withColumnRenamed(old, new)



r = puntuak.join(fiki, puntuak.Code == f.col("Sample Point Code_fiki"), 'inner').join(bio, puntuak.Code == f.col("Sample Point Code_bio"))

print(r.summary().collect()[0].__getitem__("Sample Point Code"))

{% endhighlight %}
