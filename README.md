# Apache Kafka Setup and Usage Guide

A comprehensive guide for setting up and using Apache Kafka on Windows with KRaft mode (without ZooKeeper).

## Table of Contents
- [Initial Setup](#initial-setup)
- [Topic Management](#topic-management)
- [Producer and Consumer Usage](#producer-and-consumer-usage)
- [Consumer Groups](#consumer-groups)
- [Scenarios](#scenarios)
  - [One Producer with Multiple Consumers (Same Group)](#scenario-1-one-producer-and-two-consumers-in-same-group)
  - [One Producer with Multiple Consumer Groups](#scenario-2-one-producer-and-two-consumer-groups)
- [Kafka UI](#kafka-ui)

## Initial Setup

### Step 1: Generate a Random UUID
Generate a cluster ID:
```powershell
PS C:\Kafka\bin\windows> .\kafka-storage.bat random-uuid
```

### Step 2: Format the Storage Directory
Format the storage directory with the generated cluster ID:
```powershell
PS C:\Kafka> .\bin\windows\kafka-storage.bat format -t keuRGfK5RPmjohSucR1Cfg -c .\config\kraft\server.properties
```

### Step 3: Start Kafka Server
Launch the Kafka server:
```powershell
PS C:\Kafka> .\bin\windows\kafka-server-start.bat .\config\kraft\server.properties
```

## Topic Management

### Create a Topic
```powershell
PS C:\Kafka> .\bin\windows\kafka-topics.bat --create --topic test-topic --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1
```

### List All Topics
```powershell
PS C:\Kafka\bin\windows> .\kafka-topics.bat --list --bootstrap-server localhost:9092
```

## Producer and Consumer Usage

### Basic Producer
Start a console producer to send messages:
```powershell
PS C:\Kafka> .\bin\windows\kafka-console-producer.bat --bootstrap-server localhost:9092 --topic test-topic
```

### Basic Consumer
Start a console consumer to receive messages:
```powershell
PS C:\Kafka> .\bin\windows\kafka-console-consumer.bat --bootstrap-server localhost:9092 --topic test-topic --from-beginning
```

## Consumer Groups

### View All Consumer Groups
```powershell
PS C:\Kafka> .\bin\windows\kafka-consumer-groups.bat --bootstrap-server localhost:9092 --list
```

### Describe a Consumer Group
Get detailed information about a specific consumer group:
```powershell
PS C:\Kafka> .\bin\windows\kafka-consumer-groups.bat --bootstrap-server localhost:9092 --group og --describe
```

## Scenarios

### Scenario 1: One Producer and Two Consumers in Same Group

#### Start Producer with Key
```powershell
PS C:\Kafka> .\bin\windows\kafka-console-producer.bat --bootstrap-server localhost:9092 --topic orders --property parse.key=true --property key.separator=:
```

When the producer starts, you can enter messages in this format:
```
key1:value1
key2:value2
```

#### Start Two Consumers in Same Group
```powershell
# Consumer 1
PS C:\Kafka> .\bin\windows\kafka-console-consumer.bat --bootstrap-server localhost:9092 --topic orders --from-beginning --group og

# Consumer 2
PS C:\Kafka> .\bin\windows\kafka-console-consumer.bat --bootstrap-server localhost:9092 --topic orders --from-beginning --group og
```

**Note:** When using keys, messages are distributed based on the hash of the key, not necessarily in a round-robin manner. All messages with the same key will always go to the same partition. This doesn't guarantee even distribution among consumers in the same group.

### Scenario 2: One Producer and Two Consumer Groups

#### Start Producer with Key
```powershell
PS C:\Kafka> .\bin\windows\kafka-console-producer.bat --bootstrap-server localhost:9092 --topic orders --property parse.key=true --property key.separator=:
```

#### Start Two Consumers in Different Groups
```powershell
# Consumer in Group 1
PS C:\Kafka> .\bin\windows\kafka-console-consumer.bat --bootstrap-server localhost:9092 --topic orders --from-beginning --group og1

# Consumer in Group 2
PS C:\Kafka> .\bin\windows\kafka-console-consumer.bat --bootstrap-server localhost:9092 --topic orders --from-beginning --group og2
```

**Note:** Different consumer groups maintain their own offset positions independently. Each consumer group will receive all messages, but if multiple consumers exist within a single group, messages will be distributed among them based on partition assignment.

## Kafka UI

To set up a web UI for Kafka management:

### Step 1: Update Kafka Configuration
Change the `advertised.listeners` in `server.properties` from:
```
advertised.listeners=PLAINTEXT://localhost:9092,CONTROLLER://localhost:9093
```
To your machine's IP address:
```
advertised.listeners=PLAINTEXT://172.28.112.1:9092,CONTROLLER://localhost:9093
```

### Step 2: Run Kafka UI in Docker
```powershell
docker run --rm -d -p 8080:8080 \
  -e KAFKA_CLUSTERS_0_NAME=my-cluster \
  -e KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS=172.28.112.1:9092 \
  --name kafkaui \
  provectuslabs/kafka-ui
```

Access the UI at http://localhost:8080

## Common Issues and Troubleshooting

If you encounter issues, check the following:

1. Ensure Kafka server is running
2. Verify port 9092 is not being used by another application
3. Check Kafka logs in `C:\Kafka\logs`
4. Make sure your firewall allows connections to Kafka ports

## Additional Resources

- [Apache Kafka Documentation](https://kafka.apache.org/documentation/)
- [Kafka CLI Tools Reference](https://kafka.apache.org/documentation/#cli)
