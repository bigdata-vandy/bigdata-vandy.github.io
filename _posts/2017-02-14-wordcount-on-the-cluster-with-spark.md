---
layout: post
title:  "Wordcount on the Cluster with Spark"
author: Josh Arnold
date: 2017-02-14 14:46:00 -0600
categories: spark 
---

## Getting Started
For instructions on obtaining access to the Vanderbilt Big Data Cluster, 
see [this blog post]({{ site.baseurl }}{% link _posts/2017-02-02-intro-to-the-cluster.md %}).

## Wordcount 

Wordcount is the "hello world" of map-reduce jobs; the frequency of each word
in a given set of documents, or corpus, is counted, and finally, the list
of unique words is sorted by document frequency. The *map* step of this process
consists of teasing out each individual word from a document and emitting a 
tuple, (word, 1). In the *reduce* step, tuples are grouped together according 
to their primary key, which in this case is the word, and the values are summed
to get the total occurrences of each word. Finally, the records are sorted by
occurrence count.

## The Spark Shell

Spark is written in Scala, and Spark distributions provide their own 
Scala-Spark REPL (Read Evaluate Print Loop), a command-line environment for
toying around with code snippets. To this end, let's start implementing
wordcount in the REPL.

### Starting the REPL

Spark can run locally on a single machine on \\( n \\) nodes, it can run as a 
standalone Spark cluster, and it can run on top of YARN. If you're using
Spark locally, then to initialize the REPL:

```bash
$SPARK_HOME/bin/spark-shell
```

If you've connected to the BigData cluster through SFTP 
(via `ssh bigdata.accre.vanderbilt.edu`), you'll need
to start the REPL first:
```bash
spark-shell
```
and then specify that you're using YARN as the application Master:

```bash
$SPARK_HOME/bin/spark-shell --master=yarn
```

At this point, you should see Spark's splash screen and a Scala prompt:

```
[username@abd740 ~]$ spark-shell --master=yarn 
Setting default log level to "WARN".
To adjust logging level use sc.setLogLevel(newLevel).
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /___/ .__/\_,_/_/ /_/\_\   version 1.6.0
      /_/

Using Scala version 2.10.5 (Java HotSpot(TM) 64-Bit Server VM, Java 1.7.0_67)
Type in expressions to have them evaluated.
Type :help for more information.
Spark context available as sc (master = yarn-client, app id = application_1486918126261_0072).
SQL context available as sqlContext.
```

These last two lines indicate the entry points for using the Spark API, the
Spark context `sc` and the SQL context `sqlContext`. We won't be using the 
SQL context in this tutorial but we will invoke the Spark context, whcich we
can examine by simply typing `sc` into the Scala REPL:

```
scala> sc
res0: org.apache.spark.SparkContext = org.apache.spark.SparkContext@49d98c3e

scala> 
```

### Creating an RDD 

We're going to use the `textFile` method of `sc` to load in a file in the `/data`
directory of HDFS (note that we could also read in a local file as well) and 
store it as a *Resilient Distributed Dataset* (RDD):

```scala
scala> val lines = sc.textFile("hdfs:///data/Spark_README.md")
lines: org.apache.spark.rdd.RDD[String] = hdfs:///data/Spark_README.md MapPartitionsRDD[5] at textFile at <console>:27
```

The REPL indicates that lines is now a `MapPartitionsRDD`. 
In the grand scheme of things, mapping to different partitions is the key to the
map-reduce paradigm; instead of sending data to a single node and processing 
it in chunks,
the program is sent to where the data resides on disk and executed in parallel fashion.
Pretty cool right?

*Note that `lines` is declared as `val` for value; in addition to being a 
strongly-typed language, Scala emphasizes functional programming and 
immutable state. Thus, values can be thought of as **immutable** variables.
Once a value is created, it's properties cannot be changed.*

### Transformations 

To demonstrate our mapping capabilities, we'll first use the RDD method `flatMap`; 
when our RDD `lines` was created, Spark separated the input text at line endings so
that each element of the RDD corresponds to a single line of text (this can easily
be verified by executing `lines.take(5)` which will print the first 5 elements of 
the RDD). Now we want to take each line and break it up into words, and for this
we can use an anonymous function `line => line.split("\\s+")` which takes a string
and splits it at whitespace, storing the non-whitespace in an array of strings. 
But since we don't care about lines but only words in this problem, we'd rather
have just an RDD of strings with each element corresponding to a word; this is 
precisely what `flatMap` does, mapping one string to potentially many strings:

```scala
scala> val words = lines.flatMap(line => line.split("\\s+"))
words: org.apache.spark.rdd.RDD[String] = MapPartitionsRDD[9] at flatMap at <console>:29

scala> words take 5 foreach println
#
Apache
Spark

Spark
scala> 
```

Since we have whitespace in the elements, we can go ahead and filter that out:

```scala
scala> words.filter(word => word.length > 0) take 5 foreach println
#
Apache
Spark
Spark
is
```

Scala supports unlimited chaining, so we can just add our filter method where
we define `words`:

```scala
scala> val words = lines.flatMap(line => line.split("\\s+")).filter(word => word.length > 0)
words: org.apache.spark.rdd.RDD[String] = MapPartitionsRDD[13] at filter at <console>:29

scala> words take 5 foreach println
#
Apache
Spark
Spark
is
```

Now, we want to count each word, and to do that, we will map each word to a Tuple 
`(word, 1)` where the integer 1 signifies that this word has been encounted 
once at this particular location:

```scala
scala> val pairs = words.map(word => (word, 1))
pairs: org.apache.spark.rdd.RDD[(String, Int)] = MapPartitionsRDD[14] at map at <console>:31

scala> pairs take 5 foreach println
(#,1)
(Apache,1)
(Spark,1)
(Spark,1)
(is,1)
```

Now, we've transformed our data for a format suitable for the reduce phase

### Reductions

The reduce phase of map-reduce consists of grouping, or *aggregating*, some data
by a key and combining all the data associated with that key. In our example,
the keys to group by are just the words themselves, and to get a total occurrence
count for each word, we want to sum up all the values (`1`s) for a given key.
To do this, we use the method `reduceByKey`:

```scala
scala> val wordCounts = pairs.reduceByKey((a, b) => a + b) 
```

Then, if we want sorted results, we need this incantation:

```scala
scala> val summary = wordCounts.takeOrdered(10)(Ordering[Int].reverse.on(_._2))
summary: Array[(String, Int)] = Array((the,21), (to,14), (Spark,13), (for,11), (and,10), (##,8), (a,8), (run,7), (can,6), (is,6))

scala> summary.foreach(println)
(the,21)
(to,14)
(Spark,13)
(for,11)
(and,10)
(##,8)
(a,8)
(run,7)
(can,6)
(is,6)
```

### Writing to file

The Scala function `println` will in fact print to stdout, but in a multinode cluster,
this is not a particularly reliable way to write output in a place that is easily
reachable. Instead, we can use the RDD method `saveAsTextFile` to write our file
to HDFS. As you might have noticed, our value `summary` is not an RDD but an
ordinary Scala array. So:

```scala
scala> val summaryRDD = sc.makeRDD(summary)
summaryRDD: org.apache.spark.rdd.RDD[(String, Int)] = ParallelCollectionRDD[11] at makeRDD at <console>:33
```

Then, to write it to file, we need to specify an output directory (either absolute
HDFS path or relative to our own HDFS home directory) that will hold our results
in potentially many files named part-XXXXX:

```scala
scala> summaryRDD.saveAsTextFile("foo_directory")
```

Now, if we exit the Spark REPL, we can inquire about our HDFS files like so:
```bash
[username@abd740 ~]$ hadoop fs -ls foo_directory
Found 3 items
-rw-r--r--   3 usergroup usergroup          0 2017-02-14 13:04 foo_directory/_SUCCESS
-rw-r--r--   3 usergroup usergroup         46 2017-02-14 13:04 foo_directory/part-00000
-rw-r--r--   3 usergroup usergroup         36 2017-02-14 13:04 foo_directory/part-00001
```

and view the contents via:

```bash
[username@abd740 ~]$ hadoop fs -cat foo_directory/part-*
(the,21)
(to,14)
(Spark,13)
(for,11)
(and,10)
(##,8)
(a,8)
(run,7)
(can,6)
(is,6)
```

## Conclusions

This is an admittedly simple example, but it demonstrates the core components of 
a map-reduce job. And, it shows off the relative simplicity of writing distributed
jobs in Spark. The `hadoop fs` commands should be pretty intuitive for \*nix users,
but we've planned another blog post for moving data around on the cluster. Feel free
to contact us
