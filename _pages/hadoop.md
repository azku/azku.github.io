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
  <property>
    <name>yarn.app.mapreduce.am.env</name>
    <value>HADOOP_MAPRED_HOME=$HADOOP_HOME/home/hadoop/hadoop/bin/hadoop</value>
  </property>
  <property>
    <name>mapreduce.map.env</name>
    <value>HADOOP_MAPRED_HOME=$HADOOP_HOME/home/hadoop/hadoop/bin/hadoop</value>
  </property>
  <property>
    <name>mapreduce.reduce.env</name>
    <value>HADOOP_MAPRED_HOME=$HADOOP_HOME/home/hadoop/hadoop/bin/hadoop</value>
  </property>
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

Puntu honetararte cluster guztiko makinentzako balioko luke instalazioak. Hemendik aurrera clusterreko makina bakoitza bereizten hasiko ginakete.
