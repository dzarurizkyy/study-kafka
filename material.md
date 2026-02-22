# ⚡ Apache Kafka – Complete Guide
A comprehensive guide for learning Apache Kafka, a distributed streaming platform for high-performance messaging between applications.

---

## 📋 Table of Contents

- [What is Publish/Subscribe](#-what-is-publishsubscribe)
- [What is Kafka](#-what-is-kafka)
- [Installation](#-installation)
- [Running Kafka](#-running-kafka)
- [Topics](#-topics)
- [Messages](#-messages)
- [Producer](#-producer)
- [Consumer](#-consumer)
- [Consumer Group](#-consumer-group)
- [Offset](#-offset)
- [Partition](#-partition)
- [Routing & Keys](#-routing--keys)
- [Kafka Clients](#-kafka-clients)

---

## 📡 What is Publish/Subscribe

- #### Inter-Application Communication

  When building applications, we often need to communicate with other services. The most common mechanism is **RPC (Remote Procedure Call)**, where the sender determines who receives the data. A popular example is **RESTful API**.
  
  `Limitations of RPC in complex systems:`
  
  As more services are added (e.g., Product, Promo, Cart, Order, Logistics, Payment, Fraud Detection), every new receiver forces the sender to add more sending logic, making the system increasingly complex.

- #### Messaging / Publish-Subscribe

  In the **Messaging** model, the sender does **not** determine who receives the data. Instead:
  
  - The sender sends data to a **Message Broker** (intermediary)
  - All receivers retrieve data directly from the **Message Broker**
  - When a new receiver is added, the sender doesn't need to know about it

- #### Advantages of Messaging

  - Sender doesn't need to know the complexity of receivers
  - Adding or removing receivers doesn't affect the sender

- #### Disadvantages of Messaging

  - **Not real-time** like RPC — there can be a delay between sending and receiving
  - **No delivery confirmation** — if a receiver fails to process data, the sender won't know; receivers must implement retry mechanisms

---

## 🔥 What is Kafka

Apache Kafka is a message broker application used for messaging communication, also known as a distributed commit log or distributed streaming platform. It is open source and licensed under Apache License 2.0, making it free for both personal and commercial use, and is officially available at [https://kafka.apache.org/](https://kafka.apache.org/).


- #### Why Kafka?

  | Feature | Description |
  |---------|-------------|
  | **Scalable** | Designed to handle increased load; widely adopted by large companies |
  | **High Performance** | Originally built at LinkedIn to handle massive data throughput |
  | **Persistence** | Data is stored on disk, ensuring safety even when receivers fail |

- #### Ecosystem

  Kafka integrates with virtually all modern technologies:
  
  - **Languages**: Golang, Java, NodeJS, Python, and more
  - **Data Ecosystem**: Hadoop, Spark, Elasticsearch, and more
  - Highly relevant for **Backend Engineers** and **Data Engineers**

---

## 📦 Installation

- #### Prerequisites: Java

  Kafka is built with Java, so Java must be installed first.
  
  - Check supported Java versions: https://kafka.apache.org/documentation/#java
  - Download OpenJDK (Eclipse Temurin): https://adoptium.net/temurin/releases/
  - Check supported versions: https://endoflife.date/eclipse-temurin
  
  `Verify Java installation:`
  
  ```bash
  java --version
  # openjdk 25.0.1 2025-10-21
  ```

- #### Download Kafka

  - Download binary (not source code) at: https://kafka.apache.org/downloads
  - Extract the archive to your machine

- #### Directory Structure

  | Directory | Description |
  |-----------|-------------|
  | `bin/` | Executable files (Linux/Mac) |
  | `bin/windows/` | Executable files (Windows) |
  | `config/server.properties` | Main configuration file |

---

## ▶️ Running Kafka

- #### Configure Data Directory

  By default, Kafka stores data in `/tmp/kraft-combined-logs`, which is deleted on Linux/Mac restart. It's recommended to change this in `config/server.properties`:
  
  ```properties
  # config/server.properties
  log.dirs=data
  ```

- #### Format Data Directory

  This step is only required once when setting up a new data directory:
  
  ```bash
  # Generate a unique cluster ID
  ./bin/kafka-storage.sh random-uuid
  # Output: V5FpUw33Qbu6VS7QjSkf2Q
  
  # Format the directory (standalone mode for single node)
  ./bin/kafka-storage.sh format \
    --cluster-id V5FpUw33Qbu6VS7QjSkf2Q \
    --config config/server.properties \
    --standalone
  ```

- #### Start Kafka Server

  ```bash
  ./bin/kafka-server-start.sh config/server.properties
  ```
  
  > **Windows**: Use `bin\windows\kafka-server-start.bat` instead

---

## 📂 Topics

A **Topic** is similar to a table in a database, as it is used to store messages sent by producers; the data inside a topic is stored as a **log** (append-only and ordered by arrival time), which makes write operations extremely fast because new data is always appended to the end.

- **Create a Topic:**

  ```bash
  kafka-topics.sh --bootstrap-server localhost:9092 --create --topic <name>
  ```

- **List all Topics:**

  ```bash
  kafka-topics.sh --bootstrap-server localhost:9092 --list
  ```

- **Delete a Topic:**

  ```bash
  kafka-topics.sh --bootstrap-server localhost:9092 --delete --topic <name>
  ```

- **Describe a Topic:**

  ```bash
  kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic <name>
  ```

- **Example:**

  ```bash
  ./bin/kafka-topics.sh --bootstrap-server localhost:9092 --create --topic helloworld
  # Created topic helloworld.
  
  ./bin/kafka-topics.sh --bootstrap-server localhost:9092 --list
  # helloworld
  ```
---

## 📨 Messages

A **Message** is the unit of data sent to a Kafka topic. Each message has the following structure:

| Field | Description |
|-------|-------------|
| `Topic` | Name of the topic to store the message |
| `Partition` | Partition number where the message is stored |
| `Header` | Additional metadata for the message |
| `Key` | Identifier for routing (not unique like a primary key) |
| `Value` | The actual data/content of the message |

> **Note**: Keys in Kafka are not like primary keys in a database — duplicate keys are allowed.

---

## 📤 Producer

A **Producer** is an application that sends messages to Kafka. Each message is appended to the end of the topic log.

- #### Console Producer

  ```bash
  kafka-console-producer.sh --bootstrap-server localhost:9092 --topic <name>
  ```

- #### Sending messages with a Key

  ```bash
  ./bin/kafka-console-producer.sh \
    --bootstrap-server localhost:9092 \
    --topic helloworld \
    --property "parse.key=true" \
    --property "key.separator=:"
  ```
  
  `Example:`
  
  ```
  >1:Hello World
  >2:Hello Kafka
  ```

---

## 📥 Consumer

A **Consumer** is an application that reads messages from Kafka. Messages are read in order from the earliest to the latest.

- #### Console Consumer

  ```bash
  kafka-console-consumer.sh \
    --bootstrap-server localhost:9092 \
    --topic <name> \
    --from-beginning
  ```

- #### Reading messages with Keys

  ```bash
  ./bin/kafka-console-consumer.sh \
    --bootstrap-server localhost:9092 \
    --topic helloworld \
    --property "print.key=true"
  ```

  > **Pub/Sub Behavior**: When a producer sends new data to a topic that a consumer is actively reading, the consumer automatically receives the new messages without restarting.

---

## 👥 Consumer Group

A **Consumer Group** is a logical grouping of consumers that work together to read from a topic.

- #### Without Consumer Group

  - Each consumer that doesn't specify a group gets a **new unique group** automatically
  - Data will be received **multiple times** if consumers have different groups
  - Not recommended in production

- #### With Consumer Group

  - Consumers sharing the same group ID are treated as **one unit**
  - A message is delivered to **only one consumer** within the group
  - Prevents duplicate message processing

  <br />
  
  ```bash
  kafka-console-consumer.sh \
    --bootstrap-server localhost:9092 \
    --topic <name> \
    --group <group-name> \
    --from-beginning
  ```
  
  `Example:`
  
  ```bash
  ./bin/kafka-console-consumer.sh \
    --bootstrap-server localhost:9092 \
    --topic helloworld \
    --group payment \
    --from-beginning
  ```

---

## 🔖 Offset

**Offset** is Kafka's mechanism for tracking the last message read by a consumer group.

- #### How Offset Works

  | Scenario | Behavior |
  |----------|----------|
  | First-time consumer (no offset stored) | Reads only **new** messages by default |
  | First-time consumer with `--from-beginning` | Reads **all** messages from the start |
  | Consumer restarted after reading to offset 5 | Resumes reading from **offset 6** |
  | Consumer with a **different** group ID | Offset history is **lost** |
  
  > Offset is stored per **Consumer Group**, so changing the group ID resets offset tracking.

- #### View Offset Information

  ```bash
  kafka-consumer-groups.sh \
    --bootstrap-server localhost:9092 \
    --all-groups \
    --all-topics \
    --describe
  ```
  
  `Example output:`
  
  ```
  GROUP     TOPIC       PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG
  payment   helloworld  0          20              23              3
  ```

  `Describe:`
  
  | Column | Description |
  |--------|-------------|
  | `CURRENT-OFFSET` | Last message read by the consumer group |
  | `LOG-END-OFFSET` | Latest message available in the topic |
  | `LAG` | Number of unread messages remaining |

---

## 🗂️ Partition

A **Partition** is a unit of storage within a topic. By default, each topic has **1 partition**.

- ### Key Rules

  - Each partition can only be read by **1 consumer** at a time within a consumer group
  - Multiple partitions allow multiple consumers to work **in parallel**
  - Increasing partitions enables better **horizontal scaling**

- ### Commands

  - **Create a topic with multiple partitions:**
  
    ```bash
    kafka-topics.sh \
      --bootstrap-server localhost:9092 \
      --create \
      --topic <name> \
      --partitions <number>
    ```
  
  - **Modify partition count of an existing topic:**
  
    ```bash
    kafka-topics.sh \
      --bootstrap-server localhost:9092 \
      --alter \
      --topic <name> \
      --partitions <number>
    ```
  
    > ⚠️ You can only **increase** the partition count, not decrease it.
  
  - **Example:**
  
    ```bash
    # View current partitions
    ./bin/kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic helloworld
    # PartitionCount: 1
    
    # Increase to 2 partitions
    ./bin/kafka-topics.sh --bootstrap-server localhost:9092 --alter --topic helloworld --partitions 2
    # PartitionCount: 2
    ```

---

## 🔀 Routing & Keys

- #### How Kafka Routes Messages to Partitions

  Kafka uses the message **Key** to determine which partition receives a message:
  
  ```bash
  partition = hash(message.key) % total_partitions
  
  # Example
  hash("eko") = 8
  8 % 2 = 0  →  Message goes to Partition 0
  ```
  
  > Messages with the **same key** always go to the **same partition**, guaranteeing ordering per key.

- #### Producer with Keys

  ```bash
  ./bin/kafka-console-producer.sh \
    --bootstrap-server localhost:9092 \
    --topic helloworld \
    --property "parse.key=true" \
    --property "key.separator=:"
  
  # Input format: key:value
  >1:Hello World
  >2:Hello Kafka
  ```

- #### Consumer Showing Keys

  ```bash
  ./bin/kafka-console-consumer.sh \
    --bootstrap-server localhost:9092 \
    --topic helloworld \
    --group payment \
    --property "print.key=true"
  ```

---

## 🛠️ Kafka Clients

For real applications, use a Kafka client library in your programming language of choice.

- ### Golang

  `Setup:`
  
  ```bash
  mkdir belajar-golang-kafka-consumer
  go mod init belajar-golang-kafka-consumer
  go get github.com/confluentinc/confluent-kafka-go/v2/kafka
  ```
  
  `Consumer:`
  
  ```go
  package main
  
  import (
    "fmt"
    "time"
    "github.com/confluentinc/confluent-kafka-go/v2/kafka"
  )
  
  func main() {
    config := &kafka.ConfigMap{
      "bootstrap.servers": "localhost:9092",
      "group.id":          "golang",
      "auto.offset.reset": "earliest",
    }
  
    consumer, err := kafka.NewConsumer(config)
    if err != nil {
      panic(err)
    }
    defer consumer.Close()
  
    consumer.Subscribe("helloworld", nil)
  
    for {
      message, err := consumer.ReadMessage(1 * time.Second)
      if err == nil {
        fmt.Printf("Received: %s\n", message.Value)
      }
    }
  }
  ```
  
  `Producer:`
  
  ```go
  package main
  
  import (
    "fmt"
    "strconv"
    "github.com/confluentinc/confluent-kafka-go/v2/kafka"
  )
  
  func main() {
    config := &kafka.ConfigMap{
      "bootstrap.servers": "localhost:9092",
    }
  
    producer, err := kafka.NewProducer(config)
    if err != nil {
      panic(err)
    }
    defer producer.Close()
  
    topic := "helloworld"
  
    for i := 0; i < 10; i++ {
      msg := &kafka.Message{
        TopicPartition: kafka.TopicPartition{
          Topic:     &topic,
          Partition: kafka.PartitionAny,
        },
        Key:   []byte(strconv.Itoa(i)),
        Value: []byte(fmt.Sprintf("Hello %d", i)),
      }
      producer.Produce(msg, nil)
    }
  
    producer.Flush(5 * 1000)
  }
  ```

- ### NodeJS

  `Setup:`
  
  ```bash
  mkdir belajar-nodejs-kafka
  npm init
  npm install kafkajs
  # Set "type": "module" in package.json
  ```

  `Consumer:`
  
  ```javascript
  import { Kafka } from "kafkajs";
  
  const kafka = new Kafka({ brokers: ["localhost:9092"] });
  const consumer = kafka.consumer({ groupId: "nodejs" });
  
  await consumer.subscribe({ topic: "helloworld", fromBeginning: true });
  await consumer.connect();
  
  await consumer.run({
    eachMessage: async (record) => {
      console.info(record.message.value.toString());
    }
  });
  ```
  
  `Producer:`
  
  ```javascript
  import { Kafka, Partitioners } from "kafkajs";
  
  const kafka = new Kafka({ brokers: ["localhost:9092"] });
  const producer = kafka.producer({ createPartitioner: Partitioners.DefaultPartitioner });
  
  await producer.connect();
  
  for (let i = 0; i < 10; i++) {
    await producer.send({
      topic: "helloworld",
      messages: [{ key: `${i}`, value: `Hello ${i}` }]
    });
  }
  
  await producer.disconnect();
  ```

- ### Java

  `Setup:`
  
  ```
  Generate project at https://start.spring.io/ and add the Kafka dependency.
  ```
  
  `Consumer:`
  
  ```java
  package dzarurizky.kafka;
  
  import org.apache.kafka.clients.consumer.*;
  import org.apache.kafka.common.serialization.StringDeserializer;
  import java.time.Duration;
  import java.util.*;
  
  public class ConsumerApp {
    public static void main(String[] args) {
      Properties props = new Properties();
      props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
      props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
      props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
      props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
      props.put(ConsumerConfig.GROUP_ID_CONFIG, "java");
  
      KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
      consumer.subscribe(List.of("helloworld"));
  
      while (true) {
        ConsumerRecords<String, String> records = consumer.poll(Duration.ofSeconds(1));
        for (ConsumerRecord<String, String> record : records) {
          System.out.println("Received: " + record.value());
        }
      }
    }
  }
  ```
  
  `Producer:`
  
  ```java
  package dzarurizky.kafka;
  
  import org.apache.kafka.clients.producer.*;
  import org.apache.kafka.common.serialization.StringSerializer;
  import java.util.*;
  
  public class ProducerApp {
    public static void main(String[] args) throws Exception {
      Properties props = new Properties();
      props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
      props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
      props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
  
      KafkaProducer<String, String> producer = new KafkaProducer<>(props);
  
      for (int i = 0; i < 10; i++) {
        ProducerRecord<String, String> record =
          new ProducerRecord<>("helloworld", Integer.toString(i), "Hello " + i);
        producer.send(record).get();
      }
  
      producer.close();
    }
  }
  ```

---

## 💡 Best Practices

- #### Topic Design
  - Use descriptive topic names (e.g., `order-created`, `payment-processed`)
  - Plan partition count based on expected throughput
  - Partitions cannot be reduced once created — plan ahead

- #### Consumer Groups
  - Always explicitly set a `group.id` in production
  - Name consumer groups after the consuming application (e.g., `payment-service`)
  - Monitor consumer lag to detect processing bottlenecks

- #### Keys & Partitioning
  - Use consistent keys when message ordering matters (e.g., `user_id`, `order_id`)
  - Messages with `null` keys are always routed to the same partition
  - More partitions = better parallelism, but more overhead

- #### Performance
  - Use `Flush()` or `disconnect()` in producers to ensure all messages are sent
  - Set `auto.offset.reset` to `earliest` for consumers that must not miss messages
  - Use batching and compression for high-throughput producers

- #### Data Persistence
  - Change the default `log.dirs` away from `/tmp` to avoid data loss on restart
  - Kafka retains messages for a configurable duration — consumers can replay history
