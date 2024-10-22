## Spark osoa instalatu

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

## Pyspark bakarrik instalatu
Ubuntu 24 zerbitzari makina batean spark instalatu nahi badugu, aukera bat pip instalatzea da.
Systema honetan pip instalatzeko, ingurune berezi bat ezartzera behartuta gaude. Behin ingurunea ezarrita, 

{% highlight shell %}
sudo apt install python3-pip
sudo apt install python3.12-venv
python3 -m venv .venv
.venv/bin/pip3 install pyspark[sql,ml,mllib]
.venv/bin/pyspark
{% endhighlight %}

## M&M jarduera
fitxategia lortu
https://raw.githubusercontent.com/databricks/LearningSparkV2/refs/heads/master/chapter2/py/src/data/mnm_dataset.csv

{% highlight python %}
from pyspark.sql.functions import count as _count
mnm_file ="mnm_dataset.csv"
count_mnm_df = (mnm_df .select("State", "Color", "Count") .groupBy("State", "Color") .agg(_count("Count").alias("Total")) .orderBy("Total", ascending=False))
count_mnm_df.show(n=60, truncate=False)
{% endhighlight %}

## Jarduera
Filmak eta beraien balorazioak dituzten fitxategiak hartuta, hauek HDFS barruan sartuko ditugu eta Spark bidez balorazio txarrena daukaten filmak atera beharko ditugu.
Bigarren pauso batean, balorazio txarrenek gutxienezko 10 balorazio izan beharko dituzte.


{% highlight python %}
from pyspark import SparkConf, SparkContext

# This function just creates a Python "dictionary" we can later
# use to convert movie ID's to movie names while printing out
# the final results.
def loadMovieNames():
    movieNames = {}
    with open("u.item", encoding='latin-1') as f:  #Fitxategia lokaletik irakurtzen du eta adi encodinagaz!
        for line in f:
            fields = line.split('|')
            movieNames[int(fields[0])] = fields[1]
    return movieNames

# Take each line of u.data and convert it to (movieID, (rating, 1.0))
# This way we can then add up all the ratings for each movie, and
# the total number of ratings for each movie (which lets us compute the average)
def parseInput(line):
    fields = line.split()
    return (int(fields[1]), (float(fields[2]), 1.0))

if __name__ == "__main__":
    # The main script - create our SparkContext
    conf = SparkConf().setAppName("WorstMovies")
    sc = SparkContext(conf = conf)

    # Load up our movie ID -> movie name lookup table
    movieNames = loadMovieNames()

    # Load up the raw u.data file
    lines = sc.textFile("/ml-100k/u.data")

    # Convert to (movieID, (rating, 1.0))
    movieRatings = lines.map(parseInput)

    # Reduce to (movieID, (sumOfRatings, totalRatings))
    ratingTotalsAndCount = movieRatings.reduceByKey(lambda movie1, movie2: ( movie1[0] + movie2[0], movie1[1] + movie2[1] ) )
    
    #Filter movies rated 10 or fewer times
    popularTotalsAndCounts = ratingTotalsAndCount.filter(lambda x: x[1][1]>10)

    # Map to (movieID, averageRating)
    averageRatings = popularTotalsAndCounts.mapValues(lambda totalAndCount : totalAndCount[0] / totalAndCount[1])

    # Sort by average rating
    sortedMovies = averageRatings.sortBy(lambda x: x[1])

    # Take the top 10 results
    results = sortedMovies.take(10)

    # Print them out:
    for result in results:
        print(movieNames[result[0]], result[1])

{% endhighlight %}

## pyspark instalatzeko
pip python instalatzailea erabiliko dugu. Gaur eguneko sistema eragile askok **pip**ren erabilera mugatuta dutenez, python ingurumen bat sortu beharko dugu aurrena.

Beste alde batetik, kontutan izan sistema eragilean JAVA instalazioa egotea beharrezkoa dela.

{% highlight shell %}
sudo apt install jdk-openjdk
python3 -m venv .venv
source ~/.venv/bin/activate
pip3 install pyspark[sql,ml,mllib]
{% endhighlight %}

## Spark connect
Bezero zerbitzari modeloa aplikatu daikete Sparkekin. Horretaraka ordea, aurretik instalatutako pyspark inguruneari pakete gehiago gehitu behar dizkiogu:

{% highlight shell %}
pip install distutils
pip install setuptools
pip install grpcio
pip install google
pip install --upgrade google-api-python-client
pip install --upgrade grpc_status
pip install --upgrade grpc_statuspip install grpcio-status
pip install grpcio-status
pyspark --remote sc://192.168.85.2
{% endhighlight %}

{% highlight python %}
spark.read.load("hdfs:///ml-100k/u.item)
{% endhighlight %}
