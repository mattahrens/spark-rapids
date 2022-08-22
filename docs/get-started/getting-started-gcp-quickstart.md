---
layout: page
title: GCP Dataproc Quickstart
nav_order: 4
parent: Getting-Started
---

# Quickstart Guide for the Spark RAPIDS Accelerator on GCP Dataproc
 [Google Cloud Dataproc](https://cloud.google.com/dataproc) is Google Cloud's fully managed Apache
 Spark and Hadoop service.  This guide will walk through the steps to:

* [Qualify Existing CPU workloads On Dataproc](#qualify-existing-cpu-workloads-on-dataproc)
* [Set Up GPU Cluster On Dataproc With Spark RAPIDS](#set-up-gpu-cluster-on-dataproc-with-spark-rapids)
* [Tune GPU workload On Dataproc With Spark RAPIDS](#tune-gpu-workload-on-dataproc-with-spark-rapids)

## Qualify Existing CPU Workloads On Dataproc

To determine the benefit of migrating to Spark RAPIDS for Dataproc workloads, you can leverage the
Spark RAPIDS qualification tool to help quantify the expected acceleration of migrating a Spark
application to GPU.

Prerequisites:
* [GCP Cloud Shell](https://cloud.google.com/shell) or [gcloud CLI](https://cloud.google.com/sdk/docs/install)
* [Spark Qualification tool](https://pypi.org/project/rapids-spark-qualification)

After your job completes on your Dataproc cluster, run the following command to see the Spark RAPIDS
qualification output:
```bash
rapids-spark-qualification --cluster <cluster-name>
```

The output will include information per app about the estimated speed-up and cost savings by migrating
to the Spark RAPIDS accelerator on GPU.  Additionally, a recommended GPU cluster will be generated.

<pre>
The following applications will benefit from running on the GPU:
<b>Application Name</b>          <b>Estimated Cost Savings</b>          <b>Estimated GPU Speedup</b>
Customer analysis app             <span style="color:green">50.86%</span>                          <span style="color:green">2.35x</span>
<p>
<b>Recommended GPU cluster</b>
--master-machine-type n1-standard-16
--num-workers 4
--worker-accelerator type=nvidia-tesla-t4,count=2
--worker-machine-type n1-standard-32
</pre>

## Set Up GPU Cluster On Dataproc With Spark RAPIDS

Create a Dataproc cluster using T4s
- One 16-core master node and 4 32-core worker nodes
- 2 NVIDIA T4 GPUs for each worker node

```bash
export REGION=[Your Preferred GCP Region]
export GCS_BUCKET=[Your GCS Bucket]
export CLUSTER_NAME=[Your Cluster Name]
export NUM_GPUS=2
export NUM_WORKERS=4

gcloud dataproc clusters create $CLUSTER_NAME  \
    --region=$REGION \
    --image-version=2.0-ubuntu18 \
    --master-machine-type=n1-standard-16 \
    --num-workers=$NUM_WORKERS \
    --worker-accelerator=type=nvidia-tesla-t4,count=$NUM_GPUS \
    --worker-machine-type=n1-highmem-32\
    --num-worker-local-ssds=4 \
    --initialization-actions=gs://goog-dataproc-initialization-actions-${REGION}/gpu/install_gpu_driver.sh,gs://goog-dataproc-initialization-actions-${REGION}/rapids/spark-rapids.sh \
    --optional-components=JUPYTER,ZEPPELIN \
    --metadata=rapids-runtime=SPARK \
    --bucket=$GCS_BUCKET \
    --enable-component-gateway
```

The `spark-rapids.sh` initialization script will install the latest Spark RAPIDS jar and configure the proper default settings based on the cluster shape.  Default settings are stored at `/usr/lib/spark/conf/spark-defaults.conf`.


## Tune GPU workload On Dataproc With Spark RAPIDS

After you have run your Spark application on Dataproc with Spark RAPIDS, you can run the Spark RAPIDS profiling tool to analyze your job and see recommended configuration updates based on your workload:
```bash
rapids-spark-profiler --cluster <cluster-name>
```

The output will include information per app about the recommended configuration settings to update.

<pre>
The following applications will benefit from tuning:
<b>Application Name</b>
Customer analysis app

<b>Recommended config setting updates</b>
spark.sql.shuffle.partitions=320
spark.sql.files.maxPartitionBytes=2g
spark.rapids.memory.pinnedPool.size=2g
spark.executor.memoryOverhead=8.40g
spark.driver.memory=30g
</pre>
