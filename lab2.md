## Partitions Kafka

#### Step 1: Creating the Kafka Topic

Using the AMQ Streams operator, create a Kafka topic named "demo-2" with 3 partitions and 3 replicas:

In the OpenShift console

1.Select project userXX-kafka
2.Select "Installed Operators" in the left menu
3.Create a new Topic with the "Create New" button

![Console](images/lab2-partitions-01.png)

In the topic configuration page, use the following settings:
* name: demo-2
* sheet music: 3
* replicas: 3

![Topic YAML](images/lab2-partitions-02.png)

The new Topic demo-2 appears in the list of available topics. The topic "my-topic" should also be present. This topic was created automatically at lab 1. The operator has detected the configuration change and added the topic to the list of available topics. 

![Topic Liste](images/lab2-partitions-03.png)

To validate the topic configuration in the zookeeper, use the following command:

```
oc exec my-cluster-zookeeper-0 -it -- bin/kafka-topics.sh --zookeeper localhost:12181  --describe --topic demo-2
```

This command executes the kafka-topics utility inside the zookeeper container

#### Step 2: Exploring the partitions

Publish 20 messages in the topic demo-2 with the use kafka-verifiable-producer, messages are published with an index indexed from 0 to 20:

```
oc run kafka-producer -ti --image=registry.access.redhat.com/amq7/amq-streams-kafka:1.1.0-kafka-2.1.1 --rm=true --restart=Never -- bin/kafka-verifiable-producer.sh --broker-list my-cluster-kafka-bootstrap:9092 --topic demo-2 --max-messages 20
```

![Messages](images/lab2-producer-01.png)

The messages are distributed on the three partitions and each partition starts at offset 0

Read messages on all partitions:


```
 oc run kafka-consumer -ti --image=registry.access.redhat.com/amq7/amq-streams-kafka:1.1.0-kafka-2.1.1 --rm=true --restart=Never -- bin/kafka-console-consumer.sh --bootstrap-server my-cluster-kafka-bootstrap:9092 --topic demo-2 --from-beginning
 ```

The order is not guaranteed between the partitions.

On the same partition, the order is preserved

```
oc run kafka-consumer -ti --image=registry.access.redhat.com/amq7/amq-streams-kafka:1.1.0-kafka-2.1.1 --rm=true --restart=Never -- bin/kafka-console-consumer.sh --bootstrap-server my-cluster-kafka-bootstrap:9092 --topic demo-2 --from-beginning --partition 0
```
