---
layout: post
title:  "Tensorflow on Spark"
author: Josh Arnold
categories: spark machine-learning deep-learning tensorflow 
---

TensorFlow on Spark
-------------------

As per Dan Fabbri's suggestion, we installed TensforFlow on Spark, which 
provides model-level parallelism for Tensorflow. Following the instructions 
from [Yahoo](https://github.com/yahoo/TensorFlowOnSpark/wiki/GetStarted_YARN).

### Install Python 2.7

This version of python is to be copied to HDFS.

    # download and extract Python 2.7
    export PYTHON_ROOT=~/Python
    curl -O https://www.python.org/ftp/python/2.7.12/Python-2.7.12.tgz
    tar -xvf Python-2.7.12.tgz
    rm Python-2.7.12.tgz

    # compile into local PYTHON_ROOT
    pushd Python-2.7.12
    ./configure --prefix="${PYTHON_ROOT}" --enable-unicode=ucs4
    make
    make install
    popd
    rm -rf Python-2.7.12

    # install pip
    pushd "${PYTHON_ROOT}"
    curl -O https://bootstrap.pypa.io/get-pip.py
    bin/python get-pip.py
    rm get-pip.py

    # install tensorflow (and any custom dependencies)
    export JAVA_HOME=/usr/java/jdk1.7.0_67-cloudera
    # This next step was necessary because the install wasn't finding
    # javac
    export PATH=$PATH:/usr/java/jdk1.7.0_67-cloudera/bin
    ${PYTHON_ROOT}/bin/pip install pydoop
    # Note: add any extra dependencies here
    popd

### Install and compile TensorFlow w/ RDMA Support

The instructions recommend installing tensorflow from source, but this 
requires installing bazel, which I expect to be a major pain. Instead, 
for now, I've installed via `pip`:

```bash
${PYTHON_ROOT}/bin/pip install tensorflow
```

### Install and compile Hadoop InputFormat/OutputFormat for TFRecords

[TFRecords](https://github.com/tensorflow/ecosystem/tree/master/hadoop)

Here are the original instructions:

```bash
git clone https://github.com/tensorflow/ecosystem.git
# follow build instructions to generate tensorflow-hadoop-1.0-SNAPSHOT.jar
# copy jar to HDFS for easier reference
hadoop fs -put tensorflow-hadoop-1.0-SNAPSHOT.jar
```

Building the jar is fairly involved and requires its own branch of installs.

#### protoc 3.1.0

[Google's data interchange format](https://developers.google.com/protocol-buffers/)
[(with source code)](https://github.com/google/protobuf/tree/master/src)

> Protocol buffers are Google's language-neutral, platform-neutral, 
> extensible mechanism for serializing structured data – think XML, 
> but smaller,
> faster, and simpler. You define how you want your data to be structured 
> once, then you can use special generated source code to easily write and 
> read your structured data to and from a variety of data streams and using a
> variety of languages. 

```bash
wget https://github.com/google/protobuf/archive/v3.1.0.tar.gz
```

I had to yum install `autoconf`, `automake`, and `libtool`. In the future,
we should make sure to include these in cfengine.


Then

```bash
./autogen.sh
./configure
make
make check
sudo make install
sudo ldconfig # refresh shared library cache
```

Note that `make check` passed all 7 tests.

#### Apache Maven

```bash
wget 
tar -xvzf apache-
```

```bash
$MAVEN_HOME/mvn clean package
$MAVEN_HOME/mvn install
```

This will generate the jar in the directory `ecosystem/hadoop/target/tensorflow-hadoop-1.0-SNAPSHOT.jar`.

```bash
hadoop fs -put tensorflow-hadoop-1.0-SNAPSHOT.jar
```

### Create a Python w/ TensorFlow zip package for Spark

```bash
pushd "${PYTHON_ROOT}"
zip -r Python.zip *
popd
```

Copy this Python distribution into HDFS:
```bash
hadoop fs -put ${PYTHON_ROOT}/Python.zip
```

### Install TensorFlowOnSpark

Next, clone this repo and build a zip package for Spark:

```bash
git clone git@github.com:yahoo/TensorFlowOnSpark.git
pushd TensorFlowOnSpark/src
zip -r ../tfspark.zip *
popd
```
