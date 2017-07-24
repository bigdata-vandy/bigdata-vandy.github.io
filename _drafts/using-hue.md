---
layout: post
title:  "Using Hue on the Big Data Cluster"
author: Josh Arnold
categories: hue 
---

* TOC
{:toc}

# Requirements

The bigdata cluster is available for use by the Vanderbilt community.
Users should [contact ACCRE](http://www.accre.vanderbilt.edu/?page_id=367) 
to get access to the cluster. 

# Logging on to the Cluster via Hue
Once approved, users will be
able to connect to `bigdata.accre.vanderbilt.edu` via `ssh`, but Cloudera 
Manager provides a WebUI to interact with the cluster called Hue.
To access Hue, simply to `bigdata.accre.vanderbilt.edu:8888` in your web browser
and enter your credentials.

# Using the HDFS file browser

If you've used the web UIs for Dropbox, Google Drive, etc., then this step
is a piece of cake. The File Browser is accessed from the 
dog-eared-piece-of-paper icon near the top right of the screen. In the file
broswer, you're able to navigate the directory structure of HDFS and even
view the contents of text files.

When a new user logs into Hue, Hue creates an HDFS directory for that user
at `/user/<vunetid>` which becomes that user's home directory.
*Note that, by default, logging in to Hue creates a new user's home directory
with read and execute permissions enabled for the world!*

Files can be uploaded to your directories using the drag-and-drop mechanism; however, 
the file size for transferring through the WebUI is capped at around 50GB, 
so other tools like `scp` or `rsync` are necessary for moving large files
onto the cluster.

In addition to your own data, ACCRE hosts some publicly available datasets
at `/data/`:

Directory             | Description
--------------------- | -----------
babs                  | Bay Area bikeshare data
capitalbikeshare data | DC area bikeshare data
citibike-tripdata     | NYC bikeshare data
google-ngrams         | n-grams collected from Google Books
nyc-tlc               | NYC taxi trip data
stack-archives        | historic posts from StackOverflow, et al.

If you know of other datasets that may appeal to the Vanderbilt community at
large, just let us know!

# Building an application

Hue uses Oozie to compose workflows on the cluster, and to access it, you'll 
need to follow the tabs `Workflows -> Editors -> Workflows`. From here, click
the `+ Create` button, and you'll arrive at the workflow composer screen. You
can drag and drop an application into your workflow, for instance a Spark job. 
Here you can specify the jar file (which, conveniently, 
you can generate from our [GitHub repo][spark-wc-gh], and specify options and inputs.
If you want to interactively select your input and output files each time you
execute the job, you can use the special keywords `${input}` and `${output}`, which
is a nice feature for generalizing your workflows.

# Overview of Cloudera services

The Hadoop ecosystem is rich with applications 
and Cloudera implememnts many of these
technologies out of the box.

| Cloudera Services     | Description 
|---------------------- |-------------
| Cloudera Manager      | Coordinates services 
| YARN                  | Yet Another Resource Negotiator 
| Oozie                 | Web app for scheduling Hadoop jobs 
| MR2 (MapReduce 2)     | MapReduce jobs running on top of YARN 
| Hue                   | User interface for constructing Jobs 
| Spark                 |  
| Hive                  | ETL transformations expressed as SQL
| Impala                | Interactive SQL 
| HBase                 | Random, realtime read/write access to distributed big data store
| Pig                   | High-level language for expressing data analysis programs 
| Solr                  | Text search engine supporting free form queries 

[spark-wc-gh]: https://github.com/bigdata-vandy/spark-wordcount
