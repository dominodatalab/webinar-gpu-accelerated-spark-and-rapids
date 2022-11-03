# RAPIDS Accelerator For Apache Spark
The RAPIDS Accelerator for Apache Spark provides a set of plugins for Apache Spark that leverage GPUs to accelerate processing via the RAPIDS libraries and UCX. Documentation on the current release can be found [here](https://nvidia.github.io/spark-rapids/). 

The RAPIDS Accelerator for Apache Spark provides a set of plugins for 
[Apache Spark](https://spark.apache.org) that leverage GPUs to accelerate processing
via the [RAPIDS](https://rapids.ai) libraries and [UCX](https://www.openucx.org/).

![](raw/latest/images/tpcxbb-like-results.png?inline=true)

The chart above shows results from running ETL queries based off of the 
[TPCxBB benchmark](http://www.tpc.org/tpcx-bb/default.asp). These are **not** official results in
any way. It uses a 10TB Dataset (scale factor 10,000), stored in parquet. The processing happened on
a two node DGX-2 cluster. Each node has 96 CPU cores, 1.5TB host memory, 16 V100 GPUs, and 512 GB
GPU memory.

## Using this project

To try the notebooks in this project you'll need to configure a Workspace with an on-demand Spark cluster. You can do this by executing the following steps:

1. Go to Workspaces -> Create New Workspace
2. Give your workspace a name (this is completely optional)
3. Make sure the Workspace Environment is set to Spark 3.0.0 RAPIDS Workspace Py3.6 (this is the default)
4. Select JupyterLab as your Workspace IDE
5. Set the Hardware Tier to Small (this tier will only run JupyterLab and its capacity is sufficient for the task)

At this point the Workspace dialog should look similar to this:

![](raw/latest/images/spark_workspace_1.png?inline=true)

Next, click on Compute cluster to set up on-demand Spark.

1. Select Spark in the Attach Compute Cluster section
2. Set the number of executors to 2
3. Set the executor and master hardware tiers to GPU
4. Set the cluster compute environment to Spark 3.0.0 GPU

![](raw/latest/images/spark_workspace_2.png?inline=true)

Click Launch Now. Once JupyterLab opens, follow the instructions in the SparkTest notebook to confirm that your cluster is operating properly. You can then move onto the GPU-accelerated mortgage notebook.

