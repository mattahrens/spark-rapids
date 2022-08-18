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

## Qualify Existing CPU Workloads On Dataproc

To determine the benefit of migrating to Spark RAPIDS for Dataproc workloads, you can leverage the
Spark RAPIDS qualification tool to help quantify the expected acceleration of migrating a Spark
application to GPU.

Prerequisites:
* [GCP Cloud Shell](https://cloud.google.com/shell] or [gcloud CLI](https://cloud.google.com/sdk/docs/install)
* [Spark Qualification tool](https://pypi.org/project/rapids-spark-qualification)

After your job completes on your Dataproc cluster, run the following command to see the Spark RAPIDS
qualification output:
```bash
rapids-spark-qualification --cluster <cluster-name>
```
