# Kafka-stream-HDInsight
Tutorial: Use Apache Kafka streams API in Azure HDInsight
```
- how to create an application that uses the Apache Kafka Streams API and run it with Kafka on HDInsight

- This tutorial is a streaming word count. It reads text data from a Kafka topic, extracts individual words, and then stores the word and 
count into another Kafka topic.

- Kafka stream processing is often done using Apache Spark or Apache Storm
- Kafka version 1.1.0 (in HDInsight 3.5 and 3.6) introduced the Kafka Streams API
```

we will learn
- Understand the code
- Build and deploy the application
- Configure Kafka topics
- Run the code

## Prerequisites
### 1. A Kafka on HDInsight 3.6 cluster

Ref : https://docs.microsoft.com/en-us/azure/hdinsight/kafka/apache-kafka-get-started
The Apache Kafka API can only be accessed by resources inside the same virtual network. 
In this Quickstart, you access the cluster directly using SSH. To connect other services, networks, or virtual machines to Apache Kafka, 
you must first create a virtual network and then create the resources within the network

```
All services -> Analytics > Azure HDInsight 
RG = kafka-hdinsight-rg
cluster name  = kafka-hdi-cluster
Cluster type = kafka
Version= Kafka 1.1.0 (HD( 3.6)
Cluster login username = admin
Cluster login password =
Secure Shell (SSH) username =sshuser
Use cluster login password for SSH = 

 Next: Storage >>
	Primary storage type = default value Azure Storage
	Selection method = Select from list
	Primary storage account = kafkahdistorageact
Security + networking tab
Virtual network ->create one =  kafka-hdi-vnet (IP 10.0.0.0/16, subnet = default 10.0.0.0/24)
```

### Connect to the cluster
```
ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
```

Get the Apache Zookeeper and Broker host information
```
When working with Kafka, you must know the Apache Zookeeper and Broker hosts. 
These hosts are used with the Apache Kafka API and many of the utilities that ship with Kafka.

you get the host information from the Apache Ambari REST API on the cluster
```
```
Install jq, a command-line JSON processor
	sudo apt -y install jq
Set up password variable
	export password='PASSWORD'
Extract the correctly cased cluster name
	export clusterName=$(curl -u admin:$password -sS -G "http://headnodehost:8080/api/v1/clusters" | jq -r '.items[].Clusters.cluster_name')
To set an environment variable with Zookeeper host information
	export KAFKAZKHOSTS=$(curl -sS -u admin:$password -G https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/ZOOKEEPER/components/ZOOKEEPER_SERVER | jq -r '["\(.host_components[].HostRoles.host_name):2181"] | join(",")' | cut -d',' -f1,2);
To verify that the environment variable is set correctly
	echo $KAFKAZKHOSTS
	output
			zk0-kafka.eahjefxxp1netdbyklgqj5y1ud.ex.internal.cloudapp.net:2181,zk2-kafka.eahjefxxp1netdbyklgqj5y1ud.ex.internal.cloudapp.net:2181
To set an environment variable with Apache Kafka broker host information,
	export KAFKABROKERS=$(curl -sS -u admin:$password -G https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/KAFKA/components/KAFKA_BROKER | jq -r '["\(.host_components[].HostRoles.host_name):9092"] | join(",")' | cut -d',' -f1,2);
To verify that the environment variable is set correctly
	echo $KAFKABROKERS
	output
		wn1-kafka.eahjefxxp1netdbyklgqj5y1ud.cx.internal.cloudapp.net:9092,wn0-kafka.eahjefxxp1netdbyklgqj5y1ud.cx.internal.cloudapp.net:9092
```

### Manage Apache Kafka topics
Kafka stores streams of data in topics. You can use the kafka-topics.sh utility to manage topics.

```
To create a topic,
	/usr/hdp/current/kafka-broker/bin/kafka-topics.sh --create --replication-factor 3 --partitions 8 --topic test --zookeeper $KAFKAZKHOSTS
This command connects to Zookeeper using the host information stored in $KAFKAZKHOSTS. It then creates an Apache Kafka topic named test.

To list topics
	/usr/hdp/current/kafka-broker/bin/kafka-topics.sh --list --zookeeper $KAFKAZKHOSTS
To delete a topic
	/usr/hdp/current/kafka-broker/bin/kafka-topics.sh --delete --topic topicname --zookeeper $KAFKAZKHOSTS
```
For more information on the commands available with the kafka-topics.sh utility
```
/usr/hdp/current/kafka-broker/bin/kafka-topics.sh
```

### Produce and consume records
Kafka stores records in topics. Records are produced by producers, and consumed by consumers. 
Producers and consumers communicate with the Kafka broker service. Each worker node in your HDInsight cluster is an Apache Kafka broker host.

To store records into the test topic you created earlier, and then read them using a consumer, use the following steps:
```
To write records to the topic, use the kafka-console-producer.sh utility from the SSH connection:
	/usr/hdp/current/kafka-broker/bin/kafka-console-producer.sh --broker-list $KAFKABROKERS --topic test
Type a text message on the empty line and hit enter
To read records from the topic, use the kafka-console-consumer.sh utility from the SSH connection:
	/usr/hdp/current/kafka-broker/bin/kafka-console-consumer.sh --bootstrap-server $KAFKABROKERS --topic test --from-beginning
```

### 2.  Apache Kafka Consumer and Producer API
Tutorial: Use the Apache Kafka Producer and Consumer APIs
ref : https://docs.microsoft.com/en-us/azure/hdinsight/kafka/apache-kafka-producer-consumer-api
The example application is located at https://github.com/Azure-Samples/hdinsight-kafka-java-get-started

### Build and deploy the example
```
	Use pre-built JAR files
		ownload the jars from the Kafka Get Started Azure sample
			scp kafka-producer-consumer*.jar sshuser@CLUSTERNAME-ssh.azurehdinsight.net:kafka-producer-consumer.jar
	Build the JAR files from code
		Download and extract the examples from https://github.com/Azure-Samples/hdinsight-kafka-java-get-started
			mvn clean package
			scp ./target/kafka-producer-consumer*.jar sshuser@CLUSTERNAME-ssh.azurehdinsight.net:kafka-producer-consumer.jar
```
### Run the example
```
ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
```
### To get the Kafka broker hosts, substitute the values for <clustername> and <password> in the following command and execute it

```
sudo apt -y install jq
export clusterName='<clustername>'
export password='<password>'
export KAFKABROKERS=$(curl -sS -u admin:$password -G https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/KAFKA/components/KAFKA_BROKER | jq -r '["\(.host_components[].HostRoles.host_name):9092"] | join(",")' | cut -d',' -f1,2);
```
### Create Kafka topic, myTest
```
java -jar kafka-producer-consumer.jar create myTest $KAFKABROKERS
```
### To run the producer and write data to the topic,
```
java -jar kafka-producer-consumer.jar producer myTest $KAFKABROKERS
```
### Once the producer has finished, use the following command to read from the topic:
```
java -jar kafka-producer-consumer.jar consumer myTest $KAFKABROKERS
scp ./target/kafka-producer-consumer*.jar sshuser@CLUSTERNAME-ssh.azurehdinsight.net:kafka-producer-consumer.jar
```
Use Ctrl + C to exit the consumer

note- >Multiple consumers

## Understand the code

The example application is located at https://github.com/Azure-Samples/hdinsight-kafka-java-get-started, in the Streaming subdirectory

## Build and deploy the example
```
cd hdinsight-kafka-java-get-started-master\Streaming
mvn clean package
scp ./target/kafka-streaming-1.0-SNAPSHOT.jar sshuser@clustername-ssh.azurehdinsight.net:kafka-streaming.jar
```
## Create Apache Kafka topics
```
ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
sudo apt -y install jq
export password='PASSWORD'
export clusterName=$(curl -u admin:$password -sS -G "http://headnodehost:8080/api/v1/clusters" | jq -r '.items[].Clusters.cluster_name')

export KAFKAZKHOSTS=$(curl -sS -u admin:$password -G https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/ZOOKEEPER/components/ZOOKEEPER_SERVER | jq -r '["\(.host_components[].HostRoles.host_name):2181"] | join(",")' | cut -d',' -f1,2);
export KAFKABROKERS=$(curl -sS -u admin:$password -G https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/KAFKA/components/KAFKA_BROKER | jq -r '["\(.host_components[].HostRoles.host_name):9092"] | join(",")' | cut -d',' -f1,2);

#To create the topics used by the streaming operation
/usr/hdp/current/kafka-broker/bin/kafka-topics.sh --create --replication-factor 3 --partitions 8 --topic test --zookeeper $KAFKAZKHOSTS
/usr/hdp/current/kafka-broker/bin/kafka-topics.sh --create --replication-factor 3 --partitions 8 --topic wordcounts --zookeeper $KAFKAZKHOSTS
/usr/hdp/current/kafka-broker/bin/kafka-topics.sh --create --replication-factor 3 --partitions 8 --topic RekeyedIntermediateTopic --zookeeper $KAFKAZKHOSTS
/usr/hdp/current/kafka-broker/bin/kafka-topics.sh --create --replication-factor 3 --partitions 8 --topic wordcount-example-Counts-changelog --zookeeper $KAFKAZKHOSTS

```

## Run the code

To start the streaming application as a background process			
```
java -jar kafka-streaming.jar $KAFKABROKERS $KAFKAZKHOSTS &
```
To send records to the test topic
```
java -jar kafka-producer-consumer.jar producer test $KAFKABROKERS
```
Once the producer completes, use the following command to view the information stored in the wordcounts topic
```
/usr/hdp/current/kafka-broker/bin/kafka-console-consumer.sh --bootstrap-server $KAFKABROKERS --topic wordcounts --formatter kafka.tools.DefaultMessageFormatter --property print.key=true --property key.deserializer=org.apache.kafka.common.serialization.StringDeserializer --property value.deserializer=org.apache.kafka.common.serialization.LongDeserializer --from-beginning
```
Use the Ctrl + C to exit the producer

To delete the topics used by the streaming operation
```
/usr/hdp/current/kafka-broker/bin/kafka-topics.sh --delete --topic test --zookeeper $KAFKAZKHOSTS
/usr/hdp/current/kafka-broker/bin/kafka-topics.sh --delete --topic wordcounts --zookeeper $KAFKAZKHOSTS
/usr/hdp/current/kafka-broker/bin/kafka-topics.sh --delete --topic RekeyedIntermediateTopic --zookeeper $KAFKAZKHOSTS
/usr/hdp/current/kafka-broker/bin/kafka-topics.sh --delete --topic wordcount-example-Counts-changelog --zookeeper $KAFKAZKHOSTS
```

## Clean up resources
```
delete resource group
```





## Reference
- Tutorial: Use Apache Kafka streams API in Azure HDInsight
```
https://docs.microsoft.com/en-us/azure/hdinsight/kafka/apache-kafka-streams-api
```

- KAFKA STREAMS
```
https://kafka.apache.org/10/documentation/streams/
```

- Quickstart: Create Apache Kafka cluster in Azure HDInsight using Azure portal
```
https://docs.microsoft.com/en-us/azure/hdinsight/kafka/apache-kafka-get-started
```

- Connect to Apache Kafka on HDInsight through an Azure Virtual Network
```
https://docs.microsoft.com/en-us/azure/hdinsight/kafka/apache-kafka-connect-vpn-gateway
```

- jq
```
https://stedolan.github.io/jq/
```

- Use managed disks for VMs in an availability set
```
https://docs.microsoft.com/en-us/azure/virtual-machines/windows/manage-availability#use-managed-disks-for-vms-in-an-availability-set
```

- To ensure high availability, use the Apache Kafka partition rebalance tool
```
https://github.com/hdinsight/hdinsight-kafka-tools
```

- Tutorial: Use the Apache Kafka Producer and Consumer APIs
```
https://docs.microsoft.com/en-us/azure/hdinsight/kafka/apache-kafka-producer-consumer-api
```
- Analyze logs for Apache Kafka on HDInsight
```
https://docs.microsoft.com/en-us/azure/hdinsight/kafka/apache-kafka-log-analytics-operations-management
```