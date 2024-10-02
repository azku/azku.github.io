

Spark instalatzeko hadoopeko namenodea erabiliko dugu. Bertan Spark jaitzi eta deskonprimitu ostean:
 `` wget https://www.apache.org/dyn/closer.lua/spark/spark-3.5.3/spark-3.5.3-bin-hadoop3.tgz ``
 ``tar -xzvf spark-3.5.3-bin-hadoop3.tgz``

.bashrb fitxategian ondorengo balioak txertatu ditugu:

{% highlight conf %}
#Spark
export YARN_CONF_DIR=$HADOOP_HOME/etc/hadoop
export SPARK_HOME=/home/hadoop/spark-3.5.3-bin-hadoop3
export PATH=$PATH:$SPARK_HOME/bin
{% endhighlight %}

ariketa adibidea

{% highlight python %}
textFile = sqlContext.read.option("header", "true").option("delimiter", ",").format("csv").load("/donor_choose_data/donations.csv")

{% endhighlight %}
