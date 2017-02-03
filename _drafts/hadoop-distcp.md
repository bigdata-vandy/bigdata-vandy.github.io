---
layout: post
title:  "Using Hadoop DistCp on the Cluster"
author: Josh Arnold
categories: hadoop io
---

## Getting Started
For instructions on obtaining access to the Vanderbilt Big Data Cluster, 
see [this blog post](page)

## DistCp



## Copying Data Locally 


## Copying from AWS to HDFS

A note on AWS credentials. 

```bash
#!/bin/bash

export $(cat ~/.aws/credentials | grep -v "^\[" | awk '{print toupper($1)$2$3 }')

```

I've factored out setting these credentials into it's own script, since
setting these environment variables comes up often. 


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
