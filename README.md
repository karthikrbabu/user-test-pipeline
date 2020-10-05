# Tracking User Activity

### 07/02/2020 - Shyamkarthik Rameshbabu

In this project, I simulate as if I worked at an ed tech firm. We have created a service that
delivers assessments, and now lots of different customers (e.g., Pearson) want
to publish their assessments on it. As users interact with the platform, data is generated and needs to be handled properly. In preparation for data scientists to analyze this data we must set up a pipeline that allows for the data to be transformed and queried.


Below you will find the various stages of this project. The descriptions, code, and visuals for the respective steps are included. Please follow them in order, and hopefully by the end you will have seen one possible end-to-end solution for this data pipeline!


## Preparing the Pipeline

To set up the infrastructure where we will pipe the data through, transform it, and ultimately land it in in a queriable structure, we will need various docker containers to help out.

The `docker-compose.yml` file is the config file that specifies all the details in order to spin up our pipeline. Please refer to this to understand the various containers that are being used. You will also find the setup of the ports that allow us to connect the containers together. Comments are included in the `docker-compose.yml` file.


## Building the Pipeline

`commands.md` describes all the commands that are required to spin up the pipeline with all the necessary command line options. Here you will find details on the pipeline setup and an in order walk through of the commands that must be run to fetch the data and publish messages to Kafka.


## Consuming the Data

`p2_spark_notebook.ipynb` is the final stop to understand the various steps in consuming messages from Kafka, then using Spark to transform them and land them in HDFS. Here you will find all the Python code and steps in order that describe and execute the ETL phase. In addition within Spark we are able to query the data, and perform some analysis to answer some fundamental questions presented by our data. 



## Other Files:

* `karthikrbabu-history.txt` includes the complete history of the commands I have run on my console. (It is un-altered!)


* `/plot_visuals` is the directory that contains (.png) files which are screenshots of the graphs that are visualized and explained in the `p2_spark_notebook.ipynb`


