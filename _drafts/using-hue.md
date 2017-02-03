---
layout: post
title:  "Using Hue on the Big Data Cluster"
author: Josh Arnold
categories: hue cloudera spark 
---

# Requirements
The bigdata cluster is available for use by the Vanderbilt community.
Users need
a valid Vanderbilt ID and password to log on, and once they've logged on,
should [contact ACCRE](http://www.accre.vanderbilt.edu/?page_id=367) 
about getting permission. Once approved, users will be
able to connect to `bigdata.accre.vanderbilt.edu` with `ssh`.


# Logging on to the Cluster via Hue
Navigate to `bigdata.accre.vanderbilt.edu:8888` in your web browser.

# Overview of Cloudera Services

The Hadoop ecosystem is thriving, and Cloudera implememnts many of these
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

In general 

# Using the HDFS File Browser
If you've used the web UIs for Dropbox, Google Drive, etc., then this step
is a piece of cake. The File Browser is accessed from the 
dog-eared-piece-of-paper icon near the top right of the screen.  

*Note: by default, logging in to Hue creates a new user's home directory
at /user/username with read and execute permissions enabled for the world!*

The file size for transferring through the WebUI is capped at 50GB ??.

# MapReduce

The origins of Big Data as we know it today start with MapReduce. 
MapReduce 1 was designed to move computation to the data. 

But no mechanism for caching... 

# Spark
Enter Spark

