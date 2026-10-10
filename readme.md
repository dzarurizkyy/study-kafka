# Study Kafka ⚡

This repository contains a practical reference guide for learning Apache Kafka from installation to client integration — covering publish/subscribe messaging, topics, producers and consumers, consumer groups, offsets, partitions, key-based routing, and clients for Golang, NodeJS, and Java.

## Installation 🔧

1. **Clone the Repository**:

   ```bash
   git clone https://github.com/dzarurizkyy/study-kafka.git
   cd study-kafka
   ```

2. **Install Java**:
   - Visit `https://adoptium.net/temurin/releases`
   - Download and install the JDK for your operating system
   - Verify the installation:

     ```bash
     java --version
     ```

3. **Download Kafka**:
   - Visit `https://kafka.apache.org/downloads`
   - Download the **binary** archive (not the source code)
   - Extract the downloaded file

4. **Set Up Kafka** (from inside the extracted Kafka folder):

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

   > **Note:** On Windows, use `bin\windows\` instead of `bin/`. By default Kafka stores data in `/tmp`, which is cleared on restart — set `log.dirs` in `config/server.properties` to keep your data.

5. **Open the Guide**:
   - Start with [Kafka Basics](001-kafka-basics.md) — every command in it assumes the server above is running on `localhost:9092`

## List of Material 📚

All topics below live in a single chapter — **[Kafka Basics](001-kafka-basics.md)** — and each entry links straight to its section.

- 📡 **[Publish/Subscribe](001-kafka-basics.md#-what-is-publishsubscribe)**

  Messaging through a broker instead of direct calls — the sender never needs to know who receives its data:

  ```text
  Producer ──→ Message Broker (Kafka) ──→ Consumer A
                                      ──→ Consumer B
                                      ──→ Consumer C  (new receivers need no change to the sender)
  ```

- 📂 **[Topics](001-kafka-basics.md#-topics)**

  An append-only log that stores the messages sent by producers — similar to a table in a database:

  ```bash
  # Create a topic
  ./bin/kafka-topics.sh --bootstrap-server localhost:9092 --create --topic orders

  # List all topics
  ./bin/kafka-topics.sh --bootstrap-server localhost:9092 --list

  # Describe a topic
  ./bin/kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic orders
  ```

- 📨 **[Messages](001-kafka-basics.md#-messages)**

  The unit of data sent to a topic — every message carries a topic, partition, header, key, and value:

  ```text
  Topic     → orders
  Partition → 0
  Header    → (optional metadata)
  Key       → order-1
  Value     → {"id": 1, "total": 50000}
  ```

- 📤 **[Producer](001-kafka-basics.md#-producer)**

  An application that sends messages to Kafka. Each message is appended to the end of the topic log:

  ```bash
  # Send messages
  ./bin/kafka-console-producer.sh --bootstrap-server localhost:9092 --topic orders

  # Send messages with a key (type key:value)
  ./bin/kafka-console-producer.sh --bootstrap-server localhost:9092 --topic orders \
    --property "parse.key=true" --property "key.separator=:"
  ```

- 📥 **[Consumer](001-kafka-basics.md#-consumer)**

  An application that reads messages from Kafka, in order from the earliest to the latest:

  ```bash
  # Read all messages from the beginning
  ./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic orders --from-beginning
  ```

- 👥 **[Consumer Group](001-kafka-basics.md#-consumer-group)**

  Consumers that share a group ID work as one unit — each message is delivered to only one consumer in the group:

  ```bash
  ./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic orders \
    --group payment-service --from-beginning
  ```

- 🔖 **[Offset](001-kafka-basics.md#-offset)**

  Tracks the last message read by a consumer group, so a restarted consumer resumes where it stopped:

  ```bash
  # Check consumer group offsets and lag
  ./bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 \
    --all-groups --all-topics --describe
  ```

- 🗂️ **[Partition](001-kafka-basics.md#️-partition)**

  A unit of storage within a topic — more partitions let more consumers in a group read in parallel:

  ```bash
  # Create a topic with multiple partitions
  ./bin/kafka-topics.sh --bootstrap-server localhost:9092 --create --topic orders --partitions 3

  # Increase the partition count of an existing topic
  ./bin/kafka-topics.sh --bootstrap-server localhost:9092 --alter --topic orders --partitions 6
  ```

- 🔀 **[Routing & Keys](001-kafka-basics.md#-routing--keys)**

  The message key decides the partition — messages with the same key always land in the same partition, which keeps them in order:

  ```text
  partition = hash(message.key) % total_partitions

  hash("order-1") = 8
  8 % 2 = 0   →   message goes to Partition 0
  ```

- 🔄 **[Message Flow](001-kafka-basics.md#-message-flow)**

  The path every message takes from producer to consumer, and why offsets let a group resume after a restart:

  ```text
  Producer → Partitioner → Partition log → Consumer group → Consumer → Offset commit
  ```

- 🛠️ **[Kafka Clients](001-kafka-basics.md#️-kafka-clients)**

  Client libraries for real applications — Golang, NodeJS, and Java. Example with the Golang client:

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
