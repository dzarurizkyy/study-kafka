# Study Kafka ⚡
This repository contains a practical guide to learn Apache Kafka from installation to client integration, with hands-on scenarios to practice and strengthen your Kafka skills.

## Installation 🔧

1. **Install Java**:
   - Visit `https://adoptium.net/temurin/releases`
   - Download and install the JDK for your operating system
   - Verify: `java --version`

2. **Download Kafka**:
   - Visit `https://kafka.apache.org/downloads`
   - Download the **binary** archive (not source code)
   - Extract the downloaded file

3. **Setup Kafka**:
   ```bash
   # Generate a unique cluster ID
   ./bin/kafka-storage.sh random-uuid

   # Format data directory (run once)
   ./bin/kafka-storage.sh format \
     --cluster-id <your-uuid> \
     --config config/server.properties \
     --standalone

   # Start Kafka Server
   ./bin/kafka-server-start.sh config/server.properties
   ```
   > **Windows**: Use `bin\windows\` instead of `bin/`

## List of Material 📚

* 📘 **Kafka Basic**

    Contains core Kafka concepts including Publish/Subscribe messaging, Topics, Producers, Consumers, Consumer Groups, Offsets, Partitions, Key-based routing, and client integrations with Golang, NodeJS, and Java.

    ```bash
    # Create a topic
    ./bin/kafka-topics.sh --bootstrap-server localhost:9092 --create --topic orders

    # Create topic with multiple partitions
    ./bin/kafka-topics.sh --bootstrap-server localhost:9092 --create --topic orders --partitions 3

    # List all topics
    ./bin/kafka-topics.sh --bootstrap-server localhost:9092 --list

    # Describe a topic
    ./bin/kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic orders

    # Send messages (Producer)
    ./bin/kafka-console-producer.sh --bootstrap-server localhost:9092 --topic orders

    # Send messages with key
    ./bin/kafka-console-producer.sh --bootstrap-server localhost:9092 --topic orders \
      --property "parse.key=true" --property "key.separator=:"

    # Read messages (Consumer)
    ./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic orders --from-beginning

    # Read messages with Consumer Group
    ./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic orders \
      --group payment-service --from-beginning

    # Check consumer group offsets and lag
    ./bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 \
      --all-groups --all-topics --describe
    ```

    `Example with Golang client:`
    
    ```go
    // Consumer
    consumer, _ := kafka.NewConsumer(&kafka.ConfigMap{
      "bootstrap.servers": "localhost:9092",
      "group.id":          "my-app",
      "auto.offset.reset": "earliest",
    })
    consumer.Subscribe("orders", nil)

    // Producer
    producer, _ := kafka.NewProducer(&kafka.ConfigMap{
      "bootstrap.servers": "localhost:9092",
    })
    
    producer.Produce(&kafka.Message{
      TopicPartition: kafka.TopicPartition{Topic: &topic, Partition: kafka.PartitionAny},
      Key:   []byte("order-1"),
      Value: []byte(`{"id": 1, "total": 50000}`),
    }, nil)
    producer.Flush(5 * 1000)
    ```

## 📍 References
* [Udemy](https://www.udemy.com/course/belajar-kafka)

## 👨‍💻 Contributors
* [Dzaru Rizky Fathan Fortuna](https://www.linkedin.com/in/dzarurizky)
