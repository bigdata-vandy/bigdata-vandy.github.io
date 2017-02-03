---
layout: post
title:  "Using Spark with GPFS on the ACCRE Cluster"
author: Josh Arnold
categories: spark slurm 
---

GPFS (General Parallel File System) is a proprietary, resilient, replicated
network files system developed at IBM. 

Running Spark on the production partition of the ACCRE cluster is a good
way to grasp the inner workings of Spark's. 

### Spark basics
Abstracts away parallelism.
Client mode, communicate over `ssh`.

### SLURM Basics
Scheduler and Resource manager. 

The credit for working out most of the logic goes  to this post on
[serverfault](http://serverfault.com/questions/776687/how-can-i-run-spark-on-a-cluster-using-slurm).

The idea is to keep all of our SLURM code in a single script, to make the logic easier
to read. As such, we use recursive calls to the same script and pass along a flag
argment `LEVEL` which denotes the block of code that we wish to run. 

# Front-matter

The script is set up as a normal SLURM script; that is, to say, a bash script. No 
`#SBATCH` directives are given here, since we'll be setting those for each subsequent
SLURM job we submit.

```bash
#!/bin/bash
# spark.slurm

if [[ "$#" -ne 1 && "$#" -ne 0 ]]; then
    echo "USAGE: SBATCH [OPTION] $0 [LEVEL]"; exit 1
fi
if [ "$#" -eq 1 ]; then
    LEVEL=$1
else
    LEVEL="CLIENT"
fi

echo $(hostname)": $LEVEL" 
```

There's nothing to special about this snippet; the first `if` statement checks for
0 or 1 arguments, and the second specifies a default level of `CLIENT` if no level
was provided in the original call (this just saves the user from typing "CLIENT" every
time they want to initialize the job).

# The Client

The first code block that encapsulates one level of our recursion is the Client block.
The Client block starts with exporting the `SLURM_JOB_ID` of the client and a number of 
Spark-specific variables, and because these variables are exported, they will be
available to the child processes (SLURM jobs) that we start from Client.

```bash
if [ $LEVEL == "CLIENT" ]; then
   
    export CLIENT_JOB_ID=$SLURM_JOB_ID
    export JOB_HOME="/scratch/$USER/$CLIENT_JOB_ID"
    
    export SPARK_WORKER_DIR=$JOB_HOME/work
    export SPARK_LOCAL_DIR=$JOB_HOME/tmp
    export JAVA_HOME=/usr/local/java/1.8.0
    export SPARK_HOME="$HOME/packages/spark-2.1.0-bin-hadoop2.7"
    mkdir -p $JOB_HOME $SPARK_WORKER_DIR $SPARK_LOCAL_DIR 

    export SPARK_WORKER_CORES=1
    export SPARK_DAEMON_MEMORY=2g
    export SPARK_MEMORY=2g
    
    # Try to load stuff that the spark scripts will load
    source "$SPARK_HOME/sbin/spark-config.sh"
    source "$SPARK_HOME/bin/load-spark-env.sh"
   
```

Note that we create directories to keep all the job output together under our a 
parent directory named after the Client's job ID. 

Once the setup is out of the way, we're ready to allocate and launch a 
SLURM job that will run Spark Master program. Here, we're providing 10G of memory
on a single node to run the Master.

```bash
    # Launch a job to start the Master
    MASTER_JOB_ID=$(sbatch \
        --nodes=1 \
        --mem=10G \
        $SCRIPT_PATH MASTER)
    echo $MASTER_JOB_ID
    MASTER_JOB_ID=${MASTER_JOB_ID//[!0-9]/} 
   
    # Read the master url from shared location 
    MASTER_URL=''
    i=0
    while [ -z "$MASTER_URL" ]; do
        if (( $i > 100 )); then
            echo "Starting master timed out"; exit 1
        fi
        sleep 1s
        if [ -f $JOB_HOME/master_url ]; then
            MASTER_URL=$(head -1 $JOB_HOME/master_url)
        fi
        ((i++))
    done
    echo "Master URL = $MASTER_URL"
```

If we don't specify a node to SLURM, then we don't know *a priori* on which node
the Master job will land. So, instead, we wait for the Master to write its URL to 
a GPFS-shared drive, in this case `$JOB_HOME/master_url`. The Client waits for 
the master to write to this location with a timeout of 100 seconds. 
In Standalone deploy mode, Spark communicates via ssh, so there's no need to 
communicate between jobs with MPI via [srun][slurm-srun].

```bash
    # Launch a job to start the Worker nodes
    export MASTER_URL=$MASTER_URL # necessary to start a worker

    # Since Spark uses ssh in standalone cluster mode, there's
    # no need to invoke MPI
    WORKERS_JOB_ID=$(sbatch \
        --nodes=1 \
        --ntasks-per-node=1 \
        --cpus-per-task=$SPARK_WORKER_CORES \
        --mem=10G \
        --time=00:05:00 \
        $SCRIPT_PATH WORKER)
    echo $WORKERS_JOB_ID
    WORKERS_JOB_ID=${WORKERS_JOB_ID//[!0-9]/} 
  
```

Once the Workers are up, we are ready to submit our job using `spark-submit`. The 
options we're passing to the `spark-submit` specify the address of the Master and 
that the driver will run on the Client node.

```bash
    # TODO: Optionally, wait for workers to signal back
    sleep 5s

    # Specify input files
    INPUT="README.md"
    OUTPUT="wordcount_$(date +%Y%m%d_%H%M%S)"
    APP="spark-wc_2.11-1.0.jar"
    
    # Submit the Spark jar
    $SPARK_HOME/bin/spark-submit \
        --master $MASTER_URL \
        --deploy-mode client \
        $APP $INPUT $OUTPUT 
    
    # Tear down the master and workers
    scancel $MASTER_JOB_ID
    scancel $WORKERS_JOB_ID
```

At the end of the Client job, we cancel the SLURM jobs for the Master and Workers
since they would otherwise continue running until reaching their time limit.

# The Master

```bash
elif [ $LEVEL == MASTER ]; then

    export SPARK_MASTER_HOST=$(hostname)
    export SPARK_MASTER_PORT=7077
    export SPARK_MASTER_WEBUI_PORT=8080
    export MASTER_URL="spark://$SPARK_MASTER_HOST:$SPARK_MASTER_PORT"
    export MASTER_WEBUI_URL="spark://$SPARK_MASTER_HOST:$SPARK_MASTER_WEBUI_PORT"

    # Try to load stuff that the spark scripts will load
    source "$SPARK_HOME/sbin/spark-config.sh"
    source "$SPARK_HOME/bin/load-spark-env.sh"
    
    # Write the master url to shared disk so that the client can read it.
    echo $MASTER_URL > $JOB_HOME/master_url
    
    # Start the Spark master
    $SPARK_HOME/bin/spark-class org.apache.spark.deploy.master.Master \
        --ip $SPARK_MASTER_HOST \
        --port $SPARK_MASTER_PORT \
        --webui-port $SPARK_MASTER_WEBUI_PORT
```

# The Worker

```bash
elif [ $LEVEL == "WORKER" ]; then
    
    # Start the Worker
    "$SPARK_HOME/bin/spark-class" \
        org.apache.spark.deploy.worker.Worker $MASTER_URL
```





[slurm-srun]:   https://slurm.schedmd.com/srun.html
