

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
