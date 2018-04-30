# Running Apache Kafka on Docker

In this guide, you will learn how to install and run Apache Kafka on Docker.

## 1. Clone the Confluent Platform Docker Images Github Repository.

```
git clone https://github.com/confluentinc/cp-docker-images
```

An example Docker Compose file is included that will start up ZooKeeper and Kafka. Navigate to `cp-docker-images/examples/kafka-single-node` , where it is located. Alternatively, you can download the file directly from [GitHub](https://raw.githubusercontent.com/confluentinc/cp-docker-images/master/examples/kafka-single-node/docker-compose.yml).

## 2. Running the ZooKeeper and Kafka containers 
Start the ZooKeeper and Kafka containers in detached mode (`-d`). Run this command from the directory that contains the `docker-compose.yml` file. For example, use this path to launch a single node environment:
```
cd <path-to-cp-docker-images>/examples/kafka-single-node/
docker-compose up -d
```
You should see the following:
```
Pulling kafka (confluentinc/cp-kafka:latest)...
latest: Pulling from confluentinc/cp-kafka
ad74af05f5a2: Already exists
d02e292e7b5e: Already exists
8de7f5c81ab0: Already exists
ed0b76dc2730: Already exists
cfc44fa8a002: Already exists
f441b84ed9ba: Already exists
d42bb38e2f0e: Already exists
Digest: sha256:61373cf6eca980887164d6fede2552015db31a809c99d6c3d5dfc70867b6cd2d
Status: Downloaded newer image for confluentinc/cp-kafka:latest
Creating kafkasinglenode_zookeeper_1 ...
Creating kafkasinglenode_zookeeper_1 ... done
Creating kafkasinglenode_kafka_1 ...
Creating kafkasinglenode_kafka_1 ... done
```
**Tip**: You can run this command to verify that the services are up and running:
```
docker-compose ps
```
You should see the following:
```
           Name                        Command            State   Ports
-----------------------------------------------------------------------
kafkasinglenode_kafka_1       /etc/confluent/docker/run   Up
kafkasinglenode_zookeeper_1   /etc/confluent/docker/run   Up
```
If the state is not Up, rerun the `docker-compose up -d` command.
Now check the ZooKeeper logs to verify that ZooKeeper is healthy.
```
docker-compose logs zookeeper | grep -i binding
```
You should see the following:
```
zookeeper_1  | [2016-07-25 03:26:04,018] INFO binding to port 0.0.0.0/0.0.0.0:32181 (org.apache.zookeeper.server.NIOServerCnxnFactory)
```
Next, check the Kafka logs to verify that broker is healthy.
```
docker-compose logs kafka | grep -i started
```
You should see the following:
```
kafka_1      | [2017-08-31 00:31:40,244] INFO [Socket Server on Broker 1], Started 1 acceptor threads (kafka.network.SocketServer)
kafka_1      | [2017-08-31 00:31:40,426] INFO [Replica state machine on controller 1]: Started replica state machine with initial state -> Map() (kafka.controller.ReplicaStateMachine)
kafka_1      | [2017-08-31 00:31:40,436] INFO [Partition state machine on Controller 1]: Started partition state machine with initial state -> Map() (kafka.controller.PartitionStateMachine)
kafka_1      | [2017-08-31 00:31:40,540] INFO [Kafka Server 1], started (kafka.server.KafkaServer)
```

## 3. Test the broker by following these instructions.
Now you can take this basic deployment for a test drive. You’ll verify that the broker is functioning normally by creating a topic and producing data to it. You’ll use the client tools directly from another Docker container.

1. Create a topic named foo and keep things simple by just giving it one partition and one replica. For a production environment you would have many more broker nodes, partitions, and replicas for scalability and resiliency.
```
docker-compose exec kafka  \
kafka-topics --create --topic foo --partitions 1 --replication-factor 1 --if-not-exists --zookeeper localhost:32181
```
You should see the following:
```
Created topic "foo".
```
2. Verify that the topic was created successfully:
```
docker-compose exec kafka  \
  kafka-topics --describe --topic foo --zookeeper localhost:32181
```
You should see the following:
```
Topic:foo   PartitionCount:1    ReplicationFactor:1 Configs:
Topic: foo  Partition: 0    Leader: 1    Replicas: 1  Isr: 1
```

3. Publish some data to your new topic. This command uses the built-in Kafka Console Producer to produce 42 simple messages to the topic.

```
docker-compose exec kafka  \
  bash -c "seq 42 | kafka-console-producer --request-required-acks 1 --broker-list localhost:29092 --topic foo && echo 'Produced 42 messages.'"
```
After running the command, you should see the following:

```
Produced 42 messages.
```

4. Read back the message using the built-in Console consumer:

```
docker-compose exec kafka  \
  kafka-console-consumer --bootstrap-server localhost:29092 --topic foo --from-beginning --max-messages 42
```

If everything is working as expected, each of the original messages you produced should be written back out:

```
1
....
42
Processed a total of 42 messages
```

5. You must explicitly shut down Docker Compose. For more information, see the [docker-compose down](https://docs.docker.com/compose/reference/down/) documentation. This will delete all of the containers that you created in this quickstart.

```
docker-compose down
```