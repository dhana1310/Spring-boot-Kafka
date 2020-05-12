# kafka-for-developers-using-spring-boot

This repository has the complete code related to kafka producers/consumers using spring boot.



- [Setup-Kafka](https://github.com/dilipsundarraj1/kafka-for-developers-using-spring-boot/blob/master/SetUpKafka.md)

## Securing your Kafka Cluster using SSL

- [Kafka SSL SetUp](https://github.com/dilipsundarraj1/kafka-for-developers-using-spring-boot/blob/master/Kafka_Security.md)

## H2 Database

- Access the h2 database in the following link - http://localhost:8081/h2-console

## Kafka using the command prompt(non-programmatically)

Kafka is a distributed stream platform to handle publish and subscribe model

cd C:\Kafka-2.5.0\bin\windows

Zookeeper managers the kafka brokers

start zookeeper
zookeeper-server-start.bat ..\..\config\zookeeper.properties

start kafka broker
kafka-server-start.bat ..\..\config\server.properties

create a topic
kafka-topics.bat --create --topic test-topic -zookeeper localhost:2181 --replication-factor 1 --partitions 4
replication-factor should be equal to less then the number of kafka brokers 
zookeeper divides the partitions of the topics to each broker based on replication-factor value

any message sent to specific broker is also copied to other brokers if replication-factor is more than 1 to maintain high availability

ISR - in-sync replicas
if brokers are 100, then no need to maintain 100 replicas of message for backup(default ISR value = replication-factor)
if we want to maintain less copies(assuming only 10 copies) than the number of brokers(100), then update the following propety min.insync.replicas

assigning the partitions of the topics to each broker is called leader assignment

if any broker goes down, the zookeeper continues the work with remaining brokers by updating partition leader and its ISR value

start a producer(without key - no ordering as the message will go to any partition in the broker)
kafka-console-producer.bat --broker-list localhost:9092 --topic test-topic

start a consumer(without key - no ordering as the message will be receive from any partition in the broker)
kafka-console-consumer.bat --bootstrap-server localhost:9092 --topic test-topic --from-beginning
--from-beginning -> this option reads all the messages from the beginning


start a producer(with - key)
kafka-console-producer.bat --broker-list localhost:9092 --topic test-topic --property "key.separator=-" --property "parse.key=true"

start a consumer(with - key)
kafka-console-consumer.bat --bootstrap-server localhost:9092 --topic test-topic --from-beginning -property "key.separator= - " --property "print.key=true"
--from-beginning -> this option reads all the messages from the beginning

to list down all the topics in a broker
kafka-topics.bat --zookeeper localhost:2181 --list

__consumer_offset topic (works like a bookmark)-> stores the groupid and the message-id of the last message it has read

consumer groups
group-id is mandatory

consumer groups contain multiple containers listening to the partitions to the topic.
For eg - topic has 4 partitions
and consumer group has 2 consumers, then 2 partitions will be read by each consumer


Commit logs and retention property- 
All the messages are stored in tmp/kafka-logs folder, and maintain for a certain period of time
This folder has unique log file for each partition

properties in server.properties file

folder to save the logs
log.dirs=/tmp/kafka-logs

For how many hours to save the messages in the logs (7 days by default)
log.retention.hours=168

Kafka cluster - it is a group of kafka brokers
Zookeeper manages these clusters

Kafka clusters send heart beats to the zookeeper to ensure availability.

for running multiple kafka brokers, we need to do the following changes

copy a new server-1.properties file and update the following properties
broker.id=1   #change the number
listeners=PLAINTEXT://localhost:9093   #change the port number
log.dirs=/tmp/kafka-logs-1   # change the folder name
auto.create.topics.enable=false  # optional

then run the following command
kafka-server-start.bat ..\..\config\server-1.properties
