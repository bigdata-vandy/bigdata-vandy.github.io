---
layout: post
title:  "Introduction to the Big Data Cluster"
date:   2017-02-02 12:34:00 -0600
categories: introduction specs
---

In this post, we'll introduce you to the hardware, software, and entry points 
of our big data cluster, "bigdata".

# The Hardware
The current bigdata cluster is a test cluster comprised of commercial hardware.
To conform to Cloudera recommended configuration, three nodes are dedicated
to the management of the cluster:

- abd740
- abd741
- abd742

The computational *guts* of bigdata are the datanodes, 

- abd743 
- abd744 
- abd745 
- abd746 
- abd747 
- abd748

where MapReduce jobs actually take place. Each of these nodes have one 
250 GB drive reserved mainly
for the OS and software, but for mounting the Hadoop Distributed File System 
(HDFS), each node has 2 x 4 TB hard disk drives.

# The Software

Cloudera Manager is deployed on the cluster to coordinate job submission, with 
the following services running on each node:

- abd740
  * Cloudera Manager 
    + Alert Publisher
    + Event Server
    + Host Monitor
    + Service Monitor
  *  HBase Master
  *  Hive
    + Hiveserver2
    + Hive Gateway
  *  Hue Server
  *  Oozie Server
  *  Spark Gateway
  *  YARN (MR2 Included)
    + Resource Manager
    + Job History Server
  *  Zookeeper Server

- abd741
  * HDFS NameNode
  * Hive
    + Hive Metastore Server
    + Hive Gateway
  * Spark Gateway
  * YARN (MR2 Included) Resource Manager
  * ZooKeeper Server

- abd742
  * HDFS SecondaryNameNode
  * Hive Gateway
  * Impala
    + Impala StateStore
    + Impala Catalog Server
  * Spark Gateway
  * ZooKeeper Server

# Interacting with the Cluster
Obviously, there's a lot going on in managing the cluster, but, luckily,
Cloudera provides the Hadoop User Experience (Hue) tool to coordinate
constructing and executing jobs (not to mention browsing and managing HDFS data)
through the interface `bigdata.accre.vanderbilt.edu:8888`. Users need
a valid Vanderbilt ID and password to log on, and once they've logged on,
should [contact ACCRE](http://www.accre.vanderbilt.edu/?page_id=367) 
about getting permission. Once approved, users will be
able to connect to `bigdata.accre.vanderbilt.edu` with `ssh`.

Getting started with Hue will be the subject of it's own future post, 
but most people should find
it fairly intuitive.

