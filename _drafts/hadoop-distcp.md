---
layout: post
title:  "Moving Data on the Cluster"
author: Josh Arnold
categories: hadoop io
---

## Using Hadoop Distributed File System
To analyze big data you need big data. That is, you need to have data stored
on disk. The *de facto* standard for storing big data in resilient, distributed
manner is Apache's Hadoop Distributed File System ([HDFS][apache-hadoop]).
This post walks through different methods of storing data in HDFS on the
[ACCRE BigData Cluster]({{ site.baseurl }}{% link _posts/2017-02-02-intro-to-the-cluster.md %}), and along the way, we'll introduce some basic 
[Hadoop File System shell commands][hadoop-commands].


### Local to HDFS 

Probably the most common workflow for new users is to `scp` some data to
`bigdata.accre.vanderbilt.edu` and then move that data to HDFS. The command
for doing that is:

```bash
hadoop fs -copyFromLocal \
  file:///scratch/$USER/some/data hdfs:///user/$USER/some/dir
```

or, equivalently:

```bash
hadoop fs -copyFromLocal some/data some/dir
```

The second option highlights the use of paths relative to the user's home 
directory in both the local and the hadoop file systems.

We also have the option to use `-moveFromLocal` which will delete
the local source file once it is copied to HDFS. This command is useful if 
you have many large files that you don't want hanging around on the native 
file system on the cluster. One solution is to combine an `scp` command with a
remote `ssh` command:

```bash
for f in *.txt; do 
  scp $f bigdata:$f; 
  ssh bigdata "hadoop fs -moveFromLocal $f $f"; 
done
```

### HDFS to Local

Copying from HDFS to a local drive works in very much the same with with the 
analogous `hadoop fs` commands `-copyToLocal` and `-moveToLocal`.

### Intra-HDFS



### HDFS <--> HDFS


### AWS <--> HDFS

To use Amazon Web Services (AWS), a user needs to have credentials. Getting 
credentialed is a slightly tedious but well-documented process that warrants
no further explanation here. Instead, I assume that you have credentials
stored in the file `~/.aws/credentials` on node `abd740`.

Your AWS credentials need to be passed as command-line arguments to distcp,
and I've found that a convenient and somewhat conventional way is to simply 
set the credentials as 
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

*Note that `s3`, `s3n`, `s3a` are all distinct specifications, and you should modify
your java `-D` options according to the data source.*

[apache-hadoop]: http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HdfsUserGuide.html
[hadoop-commands]: http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/FileSystemShell.html 
