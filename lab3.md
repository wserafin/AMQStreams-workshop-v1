## Kafka replication

#### Step 1: Creating the Kafka Topic

Using the AMQ Streams operator, create a Kafka topic named "demo-3" with 3 partitions and 3 replicas:

In the OpenShift console

Select project kafka-devXX
Select "Installed Operators" in the left menu
Create a new Topic with the "Create New" button

![Console](images/lab2-partitions-01.png)

In the topic configuration page, use the following settings:

name: demo-3
sheet music: 3
replicas: 3

![Topic YAML](images/lab3-replication-02.png)

The new Topic demo-3 appears in the list of available topics.

![Topic Liste](images/lab3-replication-03.png)

To validate the topic configuration in the zookeeper, use the following command:

```
oc exec my-cluster-zookeeper-0 -it -- bin/kafka-topics.sh --zookeeper localhost:12181  --describe --topic demo-3
```

![ISR](images/lab3-replication-04.png)

The following columns are displayed:
* Leader - the Kafka broker identifier that will serve all queries for a given partition.
* Replicas - the order of replication of a partition on other brokers.
* ISR - In-Sync Replicas - Up-to-date brokers in messages for a given partition (replication completed)


#### Step 2: Replication

Publish 20 messages in the topic demo-3 with the use kafka-verifiable-producer, messages are published with an index indexed from 0 to 20:

```
oc run kafka-producer -ti --image=registry.access.redhat.com/amq7/amq-streams-kafka:1.1.0-kafka-2.1.1 --rm=true --restart=Never -- bin/kafka-verifiable-producer.sh --broker-list my-cluster-kafka-bootstrap:9092 --topic demo-3 --max-messages 20
```

Validate that messages are persisted and available in the cluster.

```
 oc run kafka-consumer -ti --image=registry.access.redhat.com/amq7/amq-streams-kafka:1.1.0-kafka-2.1.1 --rm=true --restart=Never -- bin/kafka-console-consumer.sh --bootstrap-server my-cluster-kafka-bootstrap:9092 --topic demo-3 --from-beginning
 ```

 Simulate a failure by destroying one of the brokers:
```
oc delete pod my-cluster-kafka-0
```

Validate that the messages are persisted and still available in the cluster.

```
 oc run kafka-consumer -ti --image=registry.access.redhat.com/amq7/amq-streams-kafka:1.1.0-kafka-2.1.1 --rm=true --restart=Never -- bin/kafka-console-consumer.sh --bootstrap-server my-cluster-kafka-bootstrap:9092 --topic demo-3 --from-beginning
 ```

Validate the new configuration of the brokers:


```
oc exec my-cluster-zookeeper-0 -it -- bin/kafka-topics.sh --zookeeper localhost:21810  --describe --topic demo-3
```

Leaders and Replicas have changed

![ISR](images/lab3-replication-05.png)
