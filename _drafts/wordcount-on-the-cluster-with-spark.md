---
layout: post
title:  "Wordcount on the Cluster with Spark"
categories: spark 
---

## Getting Started
For instructions on obtaining access to the Vanderbilt Big Data Cluster, 
see [this blog post](page)

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

# Starting the REPL

Spark can run locally on a single machine on $n$ nodes, it can run as a 
standalone Spark cluster, and it can run on top of YARN. If you're using
Spark locally, then to initialize the REPL:

```bash
$SPARK_HOME/bin/spark-shell
```

If you've connected to the cluster through SFTP (via `ssh`), you'll need
to specify that you're using YARN as the application Master:

TODO: cluster v client

```bash
$SPARK_HOME/bin/spark-shell --master=yarn --deploy-mode=cluster
```

In order to use Spark, you need to create a Spark context, but conveniently,
the Spark REPL does this for us:

```scala
sc
val textFile = sc.textFile("README.md")
```

Notice that `textFile` is type as `val` for value; in addition to being a 
strongly-typed language, Scala emphasizes functional programming and 
immutable state. Thus, values can be thought of as *immutable* variables.
Once a value is created, it's properties cannot be changed.

```scala
val words = textFile.flatMap(line => line.split())
```



