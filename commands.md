## Pipeline Commands

This document details out all of the commands and steps taken from within the following directory:

```
/home/jupyter/w205/project-2-karthikrbabu
```

In short, below are the relevant commands that provide a glimpse at fetching the "assessments" data, minimally unrolling the data with <em> jq </em>, and producing messages to be fed into Kafka. The remaining analysis will follow in 'p2_spark_notebook.ipynb'.


### Download the data from google source.
CURL is a command-line tool for transferring data using various network protocols. The name stands for "Client URL". This tool will fetch the JSON response from the URL we are hitting.
* -L indicates that curl will redo the request if the hit URL produces a 3** response indicating that the site has moved to a different location.
* -o indicates that we write the response to an output file of the specifed name, instead of stdout.


```
curl -L -o assessment-attempts-20180128-121051-nested.json https://goo.gl/ME6hjp
```

### Rename the data file to 'assessment_attempts.json' so that the name is shorter
```
mv assessment-attempts-20180128-121051-nested.json assessment_attempts.json
```

### Look at the data in pretty printed json structure, piece by piece with 'less' rather than all blobbed at once.
* <em>cat</em> lets us concatenate the file and print it on standard output
* <em>jq</em> is the tool we use to parse JSON data and print it on standard output
* <em>less</em> lets us print to standard output with backwards and forwards movement so that we don't crowd the output all at once
* We use '|', also known as pipes to glue the steps together taking one output as an input to the next.
* <em>jq . </em> pretty prints the json structure as is.

```
cat assessment_attempts.json | jq . | less
```


### Try to get length of the number of entries, this however results in just 1 because we haven't unrolled the array
* Herer we use <em>cat</em> to print our data, <em>jq</em> as the tool to parse its json stucture with -c to compress each item of the array into one line
* <em>wc</em> as word count to count each item as one line in our output

```
cat assessment_attempts.json | jq . -c | wc -l
```

### Unrolling one layer deeper, with the same details from the previous command, but this way we can open up the outer array, capture total number of entries (3280 entries)
```
cat assessment_attempts.json | jq .[] -c | wc -l
```

### Spin up all the docker containers we want to run together, again this must be run from the project 2 directory specified at the top.
* The below command takes all the different containers and their options which are defined in the <em>docker-compose.yml</em> file and starts them together 
* Assuming all the port associations are correct and there are no errors in the file this should set up the various pieces of our pipeline through their docker containers.
* -d option allows docker to run in detached mode, therefore the containers will run in the background
```
docker-compose up -d
```

### Inspect Kafka loading up logs to see some of the things Kafka is doing under the hood when booting up
```
docker-compose logs -f kafka
```

### Via the Cloudera version of Hadoop, through that container we look at the <em>/tmp</em> directory in hdfs and see that it is currently empty to start!
* <em>fs</em> is the filesystems command that then with the option of -ls lets us list everything in the directory
```
docker-compose exec cloudera hadoop fs -ls /tmp/
```

### Using the kafka container and kafka-topics tool we can create a topic called "assessments". For each of the 3280 assessment entries we will push one message to Kafka into #this topic.

* through the various options we specify that we are creating a topic if it already doesn't exist
* setting the number of partitions (in this case 1)
* replication factor also to 1 (no redundancies)
* connecting Kafka to Zookeeper via port 32181
```
docker-compose exec kafka kafka-topics --create --topic assessments --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:32181
```

### Using the mids container we can take advantage of the jq tool and kafkacat tool in producer mode to output our JSON data line by line (entry by entry) as messages produced into Kafka to the #topic "assessments"
* on port: 29092 we produce messages to Kafka.
* -P indicates producer mode
* -b specifies the broker
* -t specifies the topic

```
docker-compose exec mids bash -c "cat /w205/project-2-karthikrbabu/assessment_attempts.json | jq '.[]' -c | kafkacat -P -b kafka:29092 -t assessments"
```

### Spin up jupyter notebook through the spark container. This will allow us to run Spark commands in the notebook with the appropriate context and access our Kafka queue
* For this to work I have added a firewall rule on my VM that exposes port 8890 on the IP Address of my VM. 
* I have also included a custom option that sets the data limit very high (10000000000) virtually getting rid of any restrictions placed on the memory allocated to the notebook.
* Once the command runs it will produce an output with a unique token. Hitting that URL and swapping out the IP with the my VM's IP will let me access the jupyter notebook.

* --port specifies the exposed port
* --allow-root gives the notebook root access
* specify the directory, and run with pyspark

```
docker-compose exec spark env PYSPARK_DRIVER_PYTHON=jupyter PYSPARK_DRIVER_PYTHON_OPTS='notebook --NotebookApp.iopub_data_rate_limit=10000000000 --no-browser --port 8890 --ip 0.0.0.0 --allow-root --notebook-dir=/w205/' pyspark
```


### On to the next step! 
At this point please continue on to the file 

```
p2_spark_notebook.ipynb
```

This will have the remaining steps that show the consumption of messages from the Kafka topic "assessments". From there after some transformations I write data to HDFS and perform some analysis via SparkSQL. 


<hr> 


### P.S. - After running the Jupyter notebook run the following command again to see that we have created the respective directories in HDFS
* <em>fs</em> is the filesystems command that then with the option of -ls lets us list everything in the directory
```
docker-compose exec cloudera hadoop fs -ls /tmp/
```


### P.P.S - Capture all the commands ran in my terminal history amd write it to a file named 'karthikrbabu-history.txt'
```
history > karthikrbabu-history.txt
```

### P.P.P.S - This will bring down our entire docker setup. All the containers and dependencies on the docker containers that are running will be killed or impacted by this command. Subsequently running 'docker ps' should show that no docker containers are running.
```
docker-compose down


docker ps
```


