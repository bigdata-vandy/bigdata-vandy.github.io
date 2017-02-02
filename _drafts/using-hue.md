---
layout: post
title:  "Using Hue on the Big Data Cluster"
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

| Cloudera Services | Description |
|-------------------|-------------|
| Cloudera Manager  | Coordinates services |
| YARN              | Yet Another Resource Negotiator |
| Oozie             | Schedules jobs, combines jobs |
| MR2               | MapReduce 2 |
| Hue               | User interface for constructing Jobs |
| Spark             | Builds on MR2 to include caching |
| Hive              | Parallel DBMS |
| Impala            |  |
| HBase             | |
| Pig               | |


# Using the HDFS File Browser
If you've used the web UIs for Dropbox, Google Drive, etc., then this step
is a piece of cake. The File Browser is accessed from the 
dog-eared-piece-of-paper icon near the top right of the screen.  

*Note: by default, logging in to Hue creates a new user's home directory
at /user/username with read and execute permissions enabled for the world!*

There is a limit to the file size.

# Running MapReduce

# Running Spark Wordcount



