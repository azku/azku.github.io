---
layout: page
title: IsardVDI
permalink: /isardvdi/
---

Zerbitzu eta aplikazioak birtualki sortzeko eta kudeatzeko plataforma libre bat da IsardVDI. Errekurtsoak era sinple eta eroso batean kudeatzea eskaintzen digu.

Honen funtzionamenduaren adibide bat eskainiko dut ondoren. Big Data sismetak azaltzeko, beharrezkoa izaten da hainbat datu iturri desberdinekin elkarlanean ibiltzea.

Horretarako, Linux sistema eragilea duen oinarrizko instalazio bat prestatuko dugu lehenik. Instalazio honen gainean docker jarriko dugu eta hau oinarri hartuta zerbitzu desberdinak prestatuko ditugu: Hadoop, Spark, Postgres DBKS bat, No SQL sistema bat...

Hasteko beraz archlinux instalazioarekin abiatuko gara. Lehenik eta behin, media atalean archlinuxen iso-a nondik eskuratu daitekeen esango diogu:

![Archlinux media sortu](/assets/images/isardvdi_1.png)

URL, izena eta deskripzioa sartu ondoren, media sortutzen da:
![Archlinux media sortu](/assets/images/isardvdi_2.png)

![Archlinux media sortu](/assets/images/isardvdi_3.png)

Orain media horretan oinarritutako mahaigain bat sortuko dugu. Horretarako mahaigain atalera joan eta bertan "Mahaigain berria" zapalduko dugu. Izena eta deskripzioa jarri eta median sortutako Archlinux ISOa aukeratu behar da:
![Archlinux media sortu](/assets/images/isardvdi_4.png)

Memoria, CPU eta gainontzeko aukeraketak egin eta jadanik prest izango da mahaigaina Archlinux instalazioa burutzeko.
![Archlinux media sortu](/assets/images/isardvdi_5.png)

Archlinux Isarden instalatzerakoan 3 gauza kontutan izatea komeni da. Lehenengoa dagokigun teklatu mota aukeratzea **loadkeys** erabilita. Bigarrena, **archinstall** scripta erabiltzea instalaziorako.

![Archlinux media sortu](/assets/images/isardvdi_6.png)

Azkenik, instalazioa burututakoan, abiotik  **CD/DVD** kendu behar da sistema instalaziotik abiatu dadin.

![Archlinux media sortu](/assets/images/isardvdi_7.png)

Behin Archlinuxen oinarrizko instalazioa burututakoan komenigarria da erabiltzaile bat sortu eta kanpotik ssh bidez konektatzeko sshd aktibatzea. Horretarako openssh instalatu eta systemd erabilita aktibatu:
{% highlight bash %}
pacman -S openssh
systemctl start sshd
systemctl enable sshd
{% endhighlight %}

Kanpoko bezero batetik Wireguard erabilita VPN bidez konektatzeko, konektatu nahi dugun mahagainean **Wireguard VPN** sarea gehitu behar dugu.
![Archlinux media sortu](/assets/images/isardvdi_8.png)

Behin hau egin eta bezerotik mahaigainak adierazten digun IP helbidean ssh bidez konektatzeko aukera izango dugu.

Docker instalazioa...

### Moodle Dockerren
**Docker** eta **docker-compose** systeman instalaturik daudelarik,  bitnami/Moodle kontainer barruan aurredefinituta datorren Moodle bat da nola eraiki adieraziko da ondorengo lerroetan.
Horretarako, Mariadb Datu-Basea eta Moodle 4.4 barneratuta dituen docker-compose hau erabili daiteke:

{% highlight yaml %}
services:
  mariadb:
    image: docker.io/bitnami/mariadb:11.3
    environment:
      # ALLOW_EMPTY_PASSWORD is recommended only for development.
      - ALLOW_EMPTY_PASSWORD=yes
      - MARIADB_USER=bn_moodle
      - MARIADB_DATABASE=bitnami_moodle
      - MARIADB_CHARACTER_SET=utf8mb4
      - MARIADB_COLLATE=utf8mb4_unicode_ci
    volumes:
      - 'mariadb_data:/bitnami/mariadb'
  moodle:
    image: docker.io/bitnami/moodle:4.4
    ports:
      - '80:8080'
      - '443:8443'
    environment:
      - MOODLE_DATABASE_HOST=mariadb
      - MOODLE_DATABASE_PORT_NUMBER=3306
      - MOODLE_DATABASE_USER=bn_moodle
      - MOODLE_DATABASE_NAME=bitnami_moodle
      # ALLOW_EMPTY_PASSWORD is recommended only for development.
      - ALLOW_EMPTY_PASSWORD=yes
      # Fitxategi handiak inportatu behar badira ondorengo 2 parametroak erabili
      - PHP_UPLOAD_MAX_FILESIZE=400M
      - PHP_POST_MAX_SIZE=400M
    volumes:
      - 'moodle_data:/bitnami/moodle'
      - 'moodledata_data:/bitnami/moodledata'
    depends_on:
      - mariadb
volumes:
  mariadb_data:
    driver: local
  moodle_data:
   driver: local
  moodledata_data:
    driver: local
{% endhighlight %}

Fitxategi hau dagoen tokian **docker-compose up -d** egin eta segundo batzuetan dagokion makinaren IP helbidean Moodle bat izan beharko genuke. Erabiltzaile balio lehenetsia **user** eta pasahitza **bitnami** izango da.
Denak ondo funtzionatu badu, systemd programatu dezakegu sistema abiatzerakoan Moodle hasteko. Horretarako **/etc/systemd/system** helbidean **docker-moodle-app.service**
izeneko fitxategia sortu behar da ondorengo edukiarekin:
{% highlight conf %}
[Unit]
Description=Docker Compose Application Service For Moodle
Requires=docker.service
After=docker.service

[Service]
Type=oneshot
RemainAfterExit=yes
WorkingDirectory=/home/moodle/moodle/
ExecStart=/usr/bin/docker-compose up -d
ExecStop=/usr/bin/docker-compose down
TimeoutStartSec=0

[Install]
WantedBy=multi-user.target
{% endhighlight %}

### Apache Hadoop Dockerren
Apache/Hadoop docker ofiziala erbiltzeko docker irudi ofiziala erabili dezakegu.
Bertan adierazten den konfigurazioa erabilita exekutatzerakoan arazoak izan ditugu "ulimits" delakoarekin. Konfigurazio fitxategian mugak aldatuta funtzionatzen du. Hona hemen adibide bat:

{% highlight yaml %}
services:
   namenode:
      image: apache/hadoop:3
      hostname: namenode
      ulimits:
        nofile:
          soft: 40000
          hard: 40000
      command: ["hdfs", "namenode"]
      ports:
        - 9870:9870
      env_file:
        - ./config
      environment:
          ENSURE_NAMENODE_DIR: "/tmp/hadoop-root/dfs/name"
   datanode:
      image: apache/hadoop:3
      ulimits:
        nofile:
          soft: 40000
          hard: 40000
      command: ["hdfs", "datanode"]
      env_file:
        - ./config      
   resourcemanager:
      image: apache/hadoop:3
      hostname: resourcemanager
      ulimits:
        nofile:
          soft: 40000
          hard: 40000
      command: ["yarn", "resourcemanager"]
      ports:
         - 8088:8088
      env_file:
        - ./config
      volumes:
        - ./test.sh:/opt/test.sh
   nodemanager:
      image: apache/hadoop:3
      ulimits:
        nofile:
          soft: 40000
          hard: 40000
      command: ["yarn", "nodemanager"]
      env_file:
        - ./config
{% endhighlight %}
Aurreko docker-compose exekutatu baino lehen, ondorengo **config** fitxategia ere sortu beharko da:
{% highlight shell %}
HADOOP_HOME=/opt/hadoop
CORE-SITE.XML_fs.default.name=hdfs://namenode
CORE-SITE.XML_fs.defaultFS=hdfs://namenode
HDFS-SITE.XML_dfs.namenode.rpc-address=namenode:8020
HDFS-SITE.XML_dfs.replication=1
MAPRED-SITE.XML_mapreduce.framework.name=yarn
MAPRED-SITE.XML_yarn.app.mapreduce.am.env=HADOOP_MAPRED_HOME=$HADOOP_HOME
MAPRED-SITE.XML_mapreduce.map.env=HADOOP_MAPRED_HOME=$HADOOP_HOME
MAPRED-SITE.XML_mapreduce.reduce.env=HADOOP_MAPRED_HOME=$HADOOP_HOME
YARN-SITE.XML_yarn.resourcemanager.hostname=resourcemanager
YARN-SITE.XML_yarn.nodemanager.pmem-check-enabled=false
YARN-SITE.XML_yarn.nodemanager.delete.debug-delay-sec=600
YARN-SITE.XML_yarn.nodemanager.vmem-check-enabled=false
YARN-SITE.XML_yarn.nodemanager.aux-services=mapreduce_shuffle
CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.maximum-applications=10000
CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.maximum-am-resource-percent=0.1
CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.resource-calculator=org.apache.hadoop.yarn.util.resource.DefaultResourceCalculator
CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.root.queues=default
CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.root.default.capacity=100
CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.root.default.user-limit-factor=1
CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.root.default.maximum-capacity=100
CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.root.default.state=RUNNING
CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.root.default.acl_submit_applications=*
CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.root.default.acl_administer_queue=*
CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.node-locality-delay=40
CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.queue-mappings=
CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.queue-mappings-override.enable=false
{% endhighlight %}
Docker ingurunea martxan jarri ``docker-compose up -d`` eta Hadoopetako batera konektatu gaitezke ``docker exec -it hadoop_docker-namenode-1 /bin/bash``
Oinarrizko Hadoop instalazioarekin batera, adibide programa batzuk datoz. Horietako bat exekutatzeko ``hadoop jar  share/hadoop/mapreduce/hadoop-mapreduce-examples-3.3.6.jar pi 5 5``
Hitzak kontatzeko programa exekutatzeko aldiz, lehenik fitxategi bat sortu edo kopiatu beharko dugu HDFS sitema barruan. Ondoren agindu hau exekutatu beharko dugu: ``hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.3.6.jar wordcount /fitxategiak/astronomia/planetak/input /fitxategiak/astronomia/planetak/output``

### PostgreSQL Dockerren
Kasu honetan PostgreSQL irudi ofiziala erabiliko dugu. Gainera, kontainerrekin clusterrak sortzeko ahalmena baliatuz, adminer kontainerra erabili dezakegu PostgreSQL datu-basea konfiguratzeko.
Hona hemen docker-compose adibide bat:

{% highlight yaml %}
services:

  postgres_db:
    image: postgres
    restart: always
    # set shared memory limit when using docker-compose
    # shm_size: 128mb
    # or set shared memory limit when deploy via swarm stack
    volumes:
       - type: bind
         source: /var/postgres-data
         target: /var/lib/postgresql/data
    networks:
      - adminer-network
    ports:
      - 5432:5432
    environment:
      POSTGRES_PASSWORD: example

  adminer:
    image: adminer
    restart: always
    ports:
      - 8080:8080
    depends_on:
      - postgres_db
    networks:
      - adminer-network

networks:
  adminer-network:
{% endhighlight %}

Kontainerrak abiatu ondoren, datu-basea dagoen kontainerrera zuzenean sartu gura bagenuke, ``docker exec -it postgres_db psql -U postgres`` erabili dezakegu.

#### DVD rental datu-basea inportatu
DVD rentaleko datuak [hemen](https://www.postgresqltutorial.com/wp-content/uploads/2019/05/dvdrental.zip) lortu daitezke.
PostgreSQL datu-basea aurkitzen den sistemara kopiatu beharko da lortutako fitzategia ``unziop`` egin ondoren.
Jarraian postgreSQL datu basera konektatu eta datu-base berri bat sortu:

{% highlight yaml %}
CREATE DATABASE dvdrental;
{% endhighlight %}

Ondoren jaitsitako fitxategia inportatu behar da datu-basera. Horretarako kontenedore barruan izan beharko dugu fitxategia. Zuzenean kontenedorera jaistea izan daiteke aukera bat. Beste alde batetik lortu badugu ordea ``docker cp dvdrental.tar f8439e70d57f:/var/dvdrental.tar`` aginduarekin kopiatu dezakegu fitxategia kontenedore barrura.

Azkenik, ``pg_restore -U postgres -d dvdrental /var/dvdrental.tar`` aginduarekin berreskurapena burutuko genuke.

#### Bezero bat
``docker run -dit --name=pgclient codingpuss/postgres-client``


``docker exec -it pgclient psql postgresql://postgres:example@10.2.76.45:5432/postgres -c 'select * from pg_catalog.pg_tables'``

### HIVE
apache/docker irudian oinarrituta, Hive eraiki daiteke bere gainean. Horretarako, lehenengo **Dockerfile** bat sortu beharko dugu ondorengo edukiarekin:
{% highlight dockerfile %}
FROM apache/hadoop:3
WORKDIR /opt
ENV HIVE_HOME=/opt/apache-hive-4.0.0-bin
RUN curl https://dlcdn.apache.org/hive/hive-4.0.0/apache-hive-4.0.0-bin.tar.gz  -o /opt/apache-hive-4.0.0-bin.tar.gz
RUN tar xzf apache-hive-4.0.0-bin.tar.gz
RUN mkdir -p /opt/external/warehouse
COPY hive-site.xml $HIVE_HOME/conf/
ENV PATH="$PATH:$HIVE_HOME/bin" 
{% endhighlight %}

Docker containerra eraikitzerakoan, Hive jaitsiko du eta deskonprimitu ondoren, **hive-site.xml** konfigurazio fitxategia kontainer barrura kopiatzen saiatuko da. Horretarako fitxategi hori prest izan behar dugu:
{% highlight xml %}
<configuration>
    <property>
        <name>hive.tez.container.size</name>
        <value>1024</value>
    </property>
 
    <property>
        <name>hive.metastore.warehouse.external.dir</name>
        <value>/opt/external/warehouse</value>
    </property>
 
    <property>
        <name>hive.execution.engine</name>
        <value>tez</value>
    </property>
 
    <property>
        <name>tez.lib.uris</name>
        <value>hdfs://localhost:9000/apps/tez/apache-tez-0.10.3-bin.tar.gz,hdfs://localhost:9000/apps/tez/apache-tez-0.10.3-bin/lib,hdfs://localhost:9000/apps/tez/apache-tez-0.10.3-bin</value>
    </property>
 
    <property>
        <name>tez.configuration</name>
        <value>/yourpathtotez/apache-tez-0.10.3-bin/conf/tez-site.xml</value>
    </property>
 
    <property>
        <name>tez.use.cluster.hadoop-libs</name>
        <value>true</value>
    </property>
    <property>
      <name>hive.server2.enable.doAs</name>
      <value>false</value> 
    </property>
</configuration>
{% endhighlight %}
Azkenik **docker-compose** fitxategian ondorengoa gehitu beharko dugu dagokion kontainerra sortzeko
Hive configuraziorako ondorengo docker-compose erabili dugu:
{% highlight yaml %}
services:
  hiveserver:
      build: .
      hostname: hiveserver2
      ulimits:
        nofile:
          soft: 40000
          hard: 40000
      command: tail -F anything
      ports:
        - 10000:10000
        - 10002:10002
      env_file:
        - ./config
{% endhighlight %}

Instalazioa ez da iraunkorra izango baina volumenak erabiliz hobetu liteke.

Behin kontainerrak martxan daudenean, hive serverra dagoen kontainerrera sartu eta ondorengo agindu biak exekutatu behar dira (ez dut docker-composen funtzionatzerik lortzen). ``schematool -dbType derby -initSchema --verbose`` eta ondoren ``hiveserver2``
Docker-compose altxatu eta irudia berreraikitzeko ``docker-compose up -d --build`` exekutatu beharko da. Ondoren irudi barnera sartzeko ``docker exec -it nirehive-hiveserver-1 /bin/bash`` edo beeline zuzenean exekutatzeko ``
docker exec -it nirehive-hiveserver-1 beeline -u 'jdbc:hive2://localhost:10000/'``

****Ariketa
Postakodeak HDFSn sartu eta Hivera inportatzeko:
``wget https://raw.githubusercontent.com/spark-examples/spark-scala-examples/master/src/main/resources/zipcodes20.csv``

``create table zipcode (id string, country string, county string, code string, pr string) row format delimited fields terminated by ',';``

``LOAD DATA INPATH '/data' OVERWRITE INTO TABLE zipcode;``
### SPARK
Spark modu desberdinetan exekutatu daiteke baina guk, YARNek kudeatutako HADOOP clusterra dugunez, SPARKek hau erabiltzea izango dugu helburu.
Horretarako, HIVEkin egin dugun bezala, ``apache/hadoop`` irudia oinarri hartuko dugu eta bertan Spark instalatu. SPARKek HADOOP konfigurazio fitxategiak erabiltzen ditu eta hauek egoki konfiguratuta izateko bide erraz bat baino ez da irudi hau erabiltzea.
Beraz, gure **spark** karpetan ``Dockerfile`` fitxategi bat sortuko dugu ondorengo edukiarekin:
{% highlight dockerfile %}
FROM apache/hadoop:3
WORKDIR /opt
ENV SPARK_HOME=/opt/spark
RUN curl https://dlcdn.apache.org/spark/spark-3.5.2/spark-3.5.2-bin-hadoop3.tgz  -o /opt/spark-3.5.2-bin-hadoop3.tgz
RUN tar -xvf spark-3.5.2-bin-hadoop3.tgz
RUN mv spark-3.5.2-bin-hadoop3 spark
RUN mv spark/conf/spark-defaults.conf.template spark/conf/spark-defaults.conf
RUN echo "spark.master           yarn" >> spark/conf/spark-defaults.conf
RUN echo "spark.driver.memory    512m" >> spark/conf/spark-defaults.conf
RUN echo "spark.yarn.am.memory   512m" >> spark/conf/spark-defaults.conf
RUN echo "spark.executor.memory  512m" >> spark/conf/spark-defaults.conf
ENV PATH="$PATH:$SPARK_HOME/bin" 
{% endhighlight %}

Jarraian **docker-compose** fitxategian hau jarriko dugu:
{% highlight dockerfile %}
services:
  namenode:
    image: apache/hadoop:3
    hostname: namenode
    ulimits:
      nofile:
        soft: 40000
        hard: 40000
    command: ["hdfs", "namenode"]
    ports:
      - 9870:9870
    env_file:
      - ./config
    environment:
      ENSURE_NAMENODE_DIR: "/tmp/hadoop-root/dfs/name"
  datanode:
    image: apache/hadoop:3
    ulimits:
      nofile:
        soft: 40000
        hard: 40000
    command: ["hdfs", "datanode"]
    env_file:
      - ./config      
  resourcemanager:
    image: apache/hadoop:3
    hostname: resourcemanager
    ulimits:
      nofile:
        soft: 40000
        hard: 40000
    command: ["yarn", "resourcemanager"]
    ports:
      - 8088:8088
    env_file:
      - ./config
    volumes:
      - ./test.sh:/opt/test.sh
  nodemanager:
    image: apache/hadoop:3
    ulimits:
      nofile:
        soft: 40000
        hard: 40000
    command: ["yarn", "nodemanager"]
    env_file:
      - ./config
  spark:
    build: .
    hostname: spark
    ulimits:
      nofile:
        soft: 40000
        hard: 40000
    command: tail -F anything
    env_file:
      - ./config
{% endhighlight %}

Azkenik configurazio fitxategia:

{% highlight conf %}
SERVICE_NAME=spark
HADOOP_HOME=/opt/hadoop
HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
CORE-SITE.XML_fs.default.name=hdfs://namenode
CORE-SITE.XML_fs.defaultFS=hdfs://namenode
HDFS-SITE.XML_dfs.namenode.rpc-address=namenode:8020
HDFS-SITE.XML_dfs.replication=1
MAPRED-SITE.XML_mapreduce.framework.name=yarn
MAPRED-SITE.XML_yarn.app.mapreduce.am.env=HADOOP_MAPRED_HOME=$HADOOP_HOME
MAPRED-SITE.XML_mapreduce.map.env=HADOOP_MAPRED_HOME=$HADOOP_HOME
MAPRED-SITE.XML_mapreduce.reduce.env=HADOOP_MAPRED_HOME=$HADOOP_HOME
YARN-SITE.XML_yarn.resourcemanager.hostname=resourcemanager
YARN-SITE.XML_yarn.nodemanager.pmem-check-enabled=false
YARN-SITE.XML_yarn.nodemanager.delete.debug-delay-sec=600
YARN-SITE.XML_yarn.nodemanager.vmem-check-enabled=false
YARN-SITE.XML_yarn.nodemanager.aux-services=mapreduce_shuffle
CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.maximum-applications=10000
CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.maximum-am-resource-percent=0.1
CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.resource-calculator=org.apache.hadoop.yarn.util.resource.DefaultResourceCalculator
CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.root.queues=default
CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.root.default.capacity=100
CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.root.default.user-limit-factor=1
CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.root.default.maximum-capacity=100
CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.root.default.state=RUNNING
CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.root.default.acl_submit_applications=*
CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.root.default.acl_administer_queue=*
CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.node-locality-delay=40
CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.queue-mappings=
CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.queue-mappings-override.enable=false
{% endhighlight %}
