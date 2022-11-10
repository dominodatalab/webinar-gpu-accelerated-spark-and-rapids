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

## Slides

[PDF copy](https://github.com/dominodatalab/webinar-gpu-accelerated-spark-and-rapids/blob/main/Spark%20Workloads%20RAPIDS.pdf)

## Prerequisites

This project uses standard python libraries and any base Domino image should work well. The last test was done on *standard-environment:ubuntu18-py3.8-r4.1-domino5.1*.

Dockerfile instructions used are below. You may not need all these to recreate the environment:

```
RUN mkdir -p /opt/domino

### Modify the Hadoop and Spark versions below as needed.
ENV HADOOP_VERSION=3.2.1
ENV HADOOP_HOME=/opt/domino/hadoop
ENV HADOOP_CONF_DIR=/opt/domino/hadoop/etc/hadoop
ENV SPARK_VERSION=3.0.0
ENV SPARK_HOME=/opt/domino/spark
ENV PATH="$PATH:$SPARK_HOME/bin:$HADOOP_HOME/bin"

### Install the desired Hadoop-free Spark distribution
RUN rm -rf ${SPARK_HOME} && \
    wget -q https://archive.apache.org/dist/spark/spark-${SPARK_VERSION}/spark-${SPARK_VERSION}-bin-without-hadoop.tgz && \
    tar -xf spark-${SPARK_VERSION}-bin-without-hadoop.tgz && \
    rm spark-${SPARK_VERSION}-bin-without-hadoop.tgz && \
    mv spark-${SPARK_VERSION}-bin-without-hadoop ${SPARK_HOME} && \
    chmod -R 777 ${SPARK_HOME}/conf

### Install the desired Hadoop libraries
RUN rm -rf ${HADOOP_HOME} && \
    wget -q http://archive.apache.org/dist/hadoop/common/hadoop-${HADOOP_VERSION}/hadoop-${HADOOP_VERSION}.tar.gz && \
    tar -xf hadoop-${HADOOP_VERSION}.tar.gz && \
    rm hadoop-${HADOOP_VERSION}.tar.gz && \
    mv hadoop-${HADOOP_VERSION} ${HADOOP_HOME}

### Setup the Hadoop libraries classpath and Spark related envars for proper init in Domino
RUN echo "export SPARK_HOME=${SPARK_HOME}" >> /home/ubuntu/.domino-defaults
RUN echo "export HADOOP_HOME=${HADOOP_HOME}" >> /home/ubuntu/.domino-defaults
RUN echo "export HADOOP_CONF_DIR=${HADOOP_CONF_DIR}" >> /home/ubuntu/.domino-defaults
RUN echo "export LD_LIBRARY_PATH=\$LD_LIBRARY_PATH:${HADOOP_HOME}/lib/native" >> /home/ubuntu/.domino-defaults
RUN echo "export PATH=\$PATH:${SPARK_HOME}/bin:${HADOOP_HOME}/bin" >> /home/ubuntu/.domino-defaults
RUN echo "export SPARK_DIST_CLASSPATH=\"\$(hadoop classpath):${HADOOP_HOME}/share/hadoop/tools/lib/*\"" >> ${SPARK_HOME}/conf/spark-env.sh

### Complete the PySpark setup from the Spark distribution files
WORKDIR $SPARK_HOME/python
RUN python setup.py install

### Optionally copy spark-submit to spark-submit.sh to be able to run from Domino jobs
RUN spark_submit_path=$(which spark-submit) && \
    cp ${spark_submit_path} ${spark_submit_path}.sh
    
ENV SPARK_RAPIDS_DIR=/opt/sparkRapidsPlugin
RUN wget -q -P $SPARK_RAPIDS_DIR https://repo1.maven.org/maven2/com/nvidia/rapids-4-spark_2.12/0.1.0/rapids-4-spark_2.12-0.1.0.jar
RUN wget -q -P $SPARK_RAPIDS_DIR https://repo1.maven.org/maven2/ai/rapids/cudf/0.14/cudf-0.14-cuda10-1.jar
ENV SPARK_CUDF_JAR=${SPARK_RAPIDS_DIR}/cudf-0.14-cuda10-1.jar
ENV SPARK_RAPIDS_PLUGIN_JAR=${SPARK_RAPIDS_DIR}/rapids-4-spark_2.12-0.1.0.jar
```

Plugable workshpace tools:

```
jupyter:
  title: "Jupyter (Python, R, Julia)"
  iconUrl: "/assets/images/workspace-logos/Jupyter.svg"
  start: [ "/var/opt/workspaces/jupyter/start" ]
  httpProxy:
    port: 8888
    rewrite: false
    internalPath: "/{{ownerUsername}}/{{projectName}}/{{sessionPathComponent}}/{{runId}}/{{#if pathToOpen}}tree/{{pathToOpen}}{{/if}}"
    requireSubdomain: false
  supportedFileExtensions: [ ".ipynb" ]
jupyterlab:
  title: "JupyterLab"
  iconUrl: "/assets/images/workspace-logos/jupyterlab.svg"
  start: [  /var/opt/workspaces/Jupyterlab/start.sh ]
  httpProxy:
    internalPath: "/{{ownerUsername}}/{{projectName}}/{{sessionPathComponent}}/{{runId}}/{{#if pathToOpen}}tree/{{pathToOpen}}{{/if}}"
    port: 8888
    rewrite: false
    requireSubdomain: false
vscode:
 title: "vscode"
 iconUrl: "/assets/images/workspace-logos/vscode.svg"
 start: [ "/var/opt/workspaces/vscode/start" ]
 httpProxy:
    port: 8888
    requireSubdomain: false
rstudio:
  title: "RStudio"
  iconUrl: "/assets/images/workspace-logos/Rstudio.svg"
  start: [ "/var/opt/workspaces/rstudio/start" ]
  httpProxy:
    port: 8888
    requireSubdomain: false
```

Note for Internal Domino Employees:

- A working version can be found [here](https://market4186.marketing-sandbox.domino.tech/u/nmanchev/Spark3GPUMortgage/overview)

- The original slides can be found [here](https://docs.google.com/presentation/d/1kvg2edb7naeZ_SAY8c1zpSe80EU5y5fS-AK2P5AHFC0/edit?usp=sharing)
