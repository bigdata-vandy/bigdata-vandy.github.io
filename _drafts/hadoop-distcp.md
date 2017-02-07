---
layout: post
title:  "Moving Data on the Cluster"
author: Josh Arnold
categories: hadoop io
---

## Getting Started
For instructions on obtaining access to the Vanderbilt Big Data Cluster, 
see [this blog post](page)

## Using the Hadoop FileSystem
To analyze big data you need big data. That is, you need to have data stored
on disk.



### Local <--> HDFS 

```bash
hadoop fs -copyFromLocal \
  file:///scratch/$USER/some/data hdfs:///user/$USER/some/dir
```


## DistCp

### HDFS <--> HDFS


### AWS <--> HDFS

A note on AWS credentials. 
Using the command line tool 

I've factored out setting these credentials into it's own script, since
setting these environment variables comes up fairly often: 

```bash
#!/bin/bash

export $(cat ~/.aws/credentials | grep -v "^\[" | awk '{print toupper($1)$2$3 }')

```


```bash
#!/bin/bash

. ~/.aws/set_credentials.sh

hadoop distcp \
-Dfs.s3n.awsAccessKeyId=$AWS_ACCESS_KEY_ID \
-Dfs.s3n.awsSecretAccessKey=$AWS_SECRET_ACCESS_KEY \
s3n://datasets.elasticmapreduce/ngrams/books/20090715/eng-us-all/ \
hdfs:///user/$USER/eng-us-all
```

Differences in `s3n` v `s3a`.
