# Study Kafka ⚡

This repository contains a comprehensive reference guide for Apache Kafka — covering publish/subscribe messaging, installation, topics, producers and consumers, consumer groups, offsets, partitions, key-based routing, and client integrations with Golang, NodeJS, and Java.

## Installation 🔧

1. **Install Java**:
   - Visit `https://adoptium.net/temurin/releases`
   - Download and install the JDK for your operating system
   - Verify the installation:

     ```bash
     java --version
     ```

2. **Download Kafka**:
   - Visit `https://kafka.apache.org/downloads`
   - Download the **binary** archive (not the source code)
   - Extract the downloaded file

3. **Set Up Kafka**:

   ```bash
   # Generate a unique cluster ID
   ./bin/kafka-storage.sh random-uuid

   # Format the data directory (run once)
   ./bin/kafka-storage.sh format \
     --cluster-id <your-uuid> \
     --config config/server.properties \
     --standalone

   # Start the Kafka server
   ./bin/kafka-server-start.sh config/server.properties
   ```

   > **Windows**: Use `bin\windows\` instead of `bin/`
   >
   > Reference: [kafka.apache.org/downloads](https://kafka.apache.org/downloads)

## List of Material 📚

- ⚡ **[Kafka Basics](001-kafka-basics.md)**

  Publish/Subscribe messaging, topics, messages, producers, consumers, consumer groups, offsets, partitions, key-based routing, and client integrations with Golang, NodeJS, and Java:

  ```bash
  # Create a topic with multiple partitions
  ./bin/kafka-topics.sh --bootstrap-server localhost:9092 --create --topic orders --partitions 3

  # Send messages with a key
  ./bin/kafka-console-producer.sh --bootstrap-server localhost:9092 --topic orders \
    --property "parse.key=true" --property "key.separator=:"

  # Read messages as part of a consumer group
  ./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic orders \
    --group payment-service --from-beginning

  # Check consumer group offsets and lag
  ./bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 \
    --all-groups --all-topics --describe
  ```

  Example with the Golang client:

  ```go
  topic := "orders"

  // Consumer
  consumer, _ := kafka.NewConsumer(&kafka.ConfigMap{
    "bootstrap.servers": "localhost:9092",
    "group.id":          "my-app",
    "auto.offset.reset": "earliest",
  })
  consumer.Subscribe(topic, nil)

  // Producer
  producer, _ := kafka.NewProducer(&kafka.ConfigMap{
    "bootstrap.servers": "localhost:9092",
  })

  producer.Produce(&kafka.Message{
    TopicPartition: kafka.TopicPartition{Topic: &topic, Partition: kafka.PartitionAny},
    Key:            []byte("order-1"),
    Value:          []byte(`{"id": 1, "total": 50000}`),
  }, nil)
  producer.Flush(5 * 1000)
  ```

## 📍 References

- [Udemy](https://www.udemy.com/course/belajar-kafka)

## 👨‍💻 Contributors

- [Dzaru Rizky Fathan Fortuna](https://www.linkedin.com/in/dzarurizky)
