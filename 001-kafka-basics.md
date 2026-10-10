# ⚡ Apache Kafka Basics

A practical reference guide for learning Apache Kafka from scratch — covering publish/subscribe messaging, installation, topics, producers and consumers, consumer groups, offsets, partitions, and client libraries for Golang, NodeJS, and Java.

---

## 📋 Table of Contents

- [What Is Publish/Subscribe?](#-what-is-publishsubscribe)
  - [Inter-Application Communication](#inter-application-communication)
  - [Limitations of RPC](#limitations-of-rpc)
  - [Messaging / Publish-Subscribe](#messaging--publish-subscribe)
  - [Advantages of Messaging](#advantages-of-messaging)
  - [Disadvantages of Messaging](#disadvantages-of-messaging)
  - [RPC vs Messaging](#rpc-vs-messaging)
- [What Is Kafka?](#-what-is-kafka)
  - [Why Kafka?](#why-kafka)
  - [Ecosystem](#ecosystem)
- [Installation](#-installation)
  - [Prerequisites: Java](#prerequisites-java)
  - [Download Kafka](#download-kafka)
  - [Directory Structure](#directory-structure)
- [Running Kafka](#️-running-kafka)
- [Topics](#-topics)
  - [Topic Commands](#topic-commands)
  - [Topic Example](#topic-example)
- [Messages](#-messages)
  - [Message Structure](#message-structure)
- [Producer](#-producer)
  - [Console Producer](#console-producer)
  - [Sending Messages with a Key](#sending-messages-with-a-key)
- [Consumer](#-consumer)
  - [Console Consumer](#console-consumer)
  - [Reading Messages with Keys](#reading-messages-with-keys)
- [Consumer Group](#-consumer-group)
  - [Without a Consumer Group](#without-a-consumer-group)
  - [With a Consumer Group](#with-a-consumer-group)
- [Offset](#-offset)
  - [How Offset Works](#how-offset-works)
  - [Viewing Offset Information](#viewing-offset-information)
- [Partition](#️-partition)
  - [Partition Rules](#partition-rules)
  - [Partition Commands](#partition-commands)
  - [Partition Example](#partition-example)
- [Routing & Keys](#-routing--keys)
  - [How Kafka Routes Messages to Partitions](#how-kafka-routes-messages-to-partitions)
  - [Producing and Consuming with Keys](#producing-and-consuming-with-keys)
- [Message Flow](#-message-flow)
- [Kafka Clients](#️-kafka-clients)
  - [Golang](#golang)
  - [NodeJS](#nodejs)
  - [Java](#java)
  - [Client Configuration Compared](#client-configuration-compared)
- [Quick Reference](#-quick-reference)
  - [Concepts](#concepts)
  - [CLI Commands](#cli-commands)
- [Best Practices](#-best-practices)

---

## 📡 What Is Publish/Subscribe?

### Inter-Application Communication

When building applications, we often need to communicate with other services. The most common mechanism is **RPC (Remote Procedure Call)**, where the **sender determines who receives the data**. A popular example is a **RESTful API**.

### Limitations of RPC

As more services are added (e.g., Product, Promo, Cart, Order, Logistics, Payment, Fraud Detection), every new receiver forces the sender to add more sending logic, making the system increasingly complex.

### Messaging / Publish-Subscribe

In the **Messaging** model, the sender does **not** determine who receives the data. Instead:

- The sender sends data to a **Message Broker** (an intermediary)
- All receivers retrieve data directly from the **Message Broker**
- When a new receiver is added, the sender doesn't need to know about it

### Advantages of Messaging

- **Decoupled sender** — The sender doesn't need to know the complexity of its receivers
- **Flexible receivers** — Adding or removing receivers doesn't affect the sender

### Disadvantages of Messaging

- **Not real-time like RPC** — There can be a delay between sending and receiving
- **No delivery confirmation** — If a receiver fails to process data, the sender won't know; receivers must implement their own retry mechanisms

### RPC vs Messaging

| | RPC | Messaging |
| --- | --- | --- |
| Who decides the receiver | The sender | The receivers subscribe themselves |
| Adding a new receiver | Change the sender | No change to the sender |
| Timing | Immediate request–response | Possible delay |
| Failure visibility | Sender sees the error | Sender is unaware; receivers must retry |
| Example | RESTful API | Apache Kafka |

---

## 🔥 What Is Kafka?

- Apache Kafka is a **message broker** application used for messaging communication
- It is also known as a **distributed commit log** or **distributed streaming platform**
- It is open source under the **Apache License 2.0**, so it's free for both personal and commercial use

> Reference: [kafka.apache.org](https://kafka.apache.org/)

### Why Kafka?

| Feature | Description |
| --- | --- |
| **Scalable** | Designed to handle increased load; widely adopted by large companies |
| **High Performance** | Originally built at LinkedIn to handle massive data throughput |
| **Persistence** | Data is stored on disk, so it stays safe even when receivers fail |

### Ecosystem

Kafka integrates with virtually all modern technologies:

- **Languages** — Golang, Java, NodeJS, Python, and more
- **Data Ecosystem** — Hadoop, Spark, Elasticsearch, and more
- **Roles** — Highly relevant for both **Backend Engineers** and **Data Engineers**

---

## 📦 Installation

### Prerequisites: Java

Kafka is built with Java, so Java must be installed first.

- Check supported Java versions: `https://kafka.apache.org/documentation/#java`
- Download OpenJDK (Eclipse Temurin): `https://adoptium.net/temurin/releases/`
- Check the support lifecycle of each version: `https://endoflife.date/eclipse-temurin`
- Verify the installation:

  ```bash
  java --version
  ```

  ```text
  openjdk 25.0.1 2025-10-21
  ```

### Download Kafka

- Download the **binary** (not the source code) at `https://kafka.apache.org/downloads`
- Extract the archive to your machine

### Directory Structure

| Directory | Description |
| --- | --- |
| `bin/` | Executable scripts (Linux/Mac) |
| `bin/windows/` | Executable scripts (Windows) |
| `config/server.properties` | Main configuration file |

---

## ▶️ Running Kafka

> **Note:** Recent Kafka versions run in **KRaft mode** — Kafka manages its own cluster metadata, and ZooKeeper is no longer required (it was removed entirely in Kafka 4.0). That's why the steps below never start a separate ZooKeeper process.

1. **Configure the Data Directory**:

   By default, Kafka stores data in `/tmp/kraft-combined-logs`, which is deleted when Linux/Mac restarts. Change it in `config/server.properties`:

   **`config/server.properties`**

   ```properties
   log.dirs=data
   ```

2. **Format the Data Directory**:

   This step is only required once, when setting up a new data directory.

   - Generate a unique cluster ID:

     ```bash
     ./bin/kafka-storage.sh random-uuid
     ```

     ```text
     V5FpUw33Qbu6VS7QjSkf2Q
     ```

   - Format the directory (standalone mode for a single node):

     ```bash
     ./bin/kafka-storage.sh format \
       --cluster-id V5FpUw33Qbu6VS7QjSkf2Q \
       --config config/server.properties \
       --standalone
     ```

3. **Start the Kafka Server**:

   ```bash
   ./bin/kafka-server-start.sh config/server.properties
   ```

   > **Windows:** Use `bin\windows\kafka-server-start.bat` instead — the same applies to every other script in this guide.

---

## 📂 Topics

> **Key Insight:** A **Topic** is similar to a table in a database — it stores the messages sent by producers. But the data inside a topic is stored as a **log**: append-only and ordered by arrival time. Because new data is always appended to the end, writes are extremely fast.

### Topic Commands

```bash
./bin/kafka-topics.sh --bootstrap-server localhost:9092 --create --topic <name>     # create a topic
./bin/kafka-topics.sh --bootstrap-server localhost:9092 --list                      # list all topics
./bin/kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic <name>   # describe a topic
./bin/kafka-topics.sh --bootstrap-server localhost:9092 --delete --topic <name>     # delete a topic
```

### Topic Example

```bash
./bin/kafka-topics.sh --bootstrap-server localhost:9092 --create --topic helloworld
```

```text
Created topic helloworld.
```

```bash
./bin/kafka-topics.sh --bootstrap-server localhost:9092 --list
```

```text
helloworld
```

---

## 📨 Messages

A **Message** is the unit of data sent to a Kafka topic.

### Message Structure

| Field | Description |
| --- | --- |
| `Topic` | Name of the topic that stores the message |
| `Partition` | Partition number where the message is stored |
| `Header` | Additional metadata for the message |
| `Key` | Identifier used for routing (not unique like a primary key) |
| `Value` | The actual data/content of the message |

> **Note:** Keys in Kafka are not like primary keys in a database — duplicate keys are allowed, and are in fact how related messages end up together (see [Routing & Keys](#-routing--keys)).

---

## 📤 Producer

A **Producer** is an application that sends messages to Kafka. Each message is appended to the end of the topic log.

### Console Producer

```bash
./bin/kafka-console-producer.sh --bootstrap-server localhost:9092 --topic <name>
```

### Sending Messages with a Key

```bash
./bin/kafka-console-producer.sh \
  --bootstrap-server localhost:9092 \
  --topic helloworld \
  --property "parse.key=true" \
  --property "key.separator=:"
```

Each line you type is parsed as `key:value`:

```text
>1:Hello World
>2:Hello Kafka
```

---

## 📥 Consumer

A **Consumer** is an application that reads messages from Kafka, in order from the earliest to the latest.

### Console Consumer

```bash
./bin/kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic <name> \
  --from-beginning
```

### Reading Messages with Keys

```bash
./bin/kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic helloworld \
  --property "print.key=true"
```

> **Note:** When a producer sends new data to a topic that a consumer is actively reading, the consumer receives the new messages automatically — no restart needed. That's the publish/subscribe behaviour in action.
>
> **Gotcha:** Ordering is only guaranteed **within a single partition**. A topic with several partitions has no global order across them — see [Partition](#️-partition).

---

## 👥 Consumer Group

A **Consumer Group** is a logical grouping of consumers that work together to read from a topic.

### Without a Consumer Group

- Each consumer that doesn't specify a group automatically gets a **new, unique group**
- Data is received **multiple times** when consumers belong to different groups
- Not recommended in production

### With a Consumer Group

- Consumers sharing the same group ID are treated as **one unit**
- A message is delivered to **only one consumer** within the group
- Prevents duplicate message processing

```bash
./bin/kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic <name> \
  --group <group-name> \
  --from-beginning
```

Example:

```bash
./bin/kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic helloworld \
  --group payment \
  --from-beginning
```

> **Key Insight:** Within one group, messages are **divided** between consumers; across different groups, every group receives **all** messages. Give each application (payment, report, notification) its own group, and run several instances of that application in the same group to share the load.

---

## 🔖 Offset

**Offset** is Kafka's mechanism for tracking the last message read by a consumer group.

### How Offset Works

| Scenario | Behavior |
| --- | --- |
| First-time consumer (no offset stored) | Reads only **new** messages by default |
| First-time consumer with `--from-beginning` | Reads **all** messages from the start |
| Consumer restarted after reading up to offset 5 | Resumes reading from **offset 6** |
| Consumer with a **different** group ID | Starts with no offset history — the old group's offset doesn't apply |

> **Note:** Offset is stored **per Consumer Group**, so changing the group ID resets offset tracking. The old group's offset is still kept — it just isn't used by the new group.

### Viewing Offset Information

```bash
./bin/kafka-consumer-groups.sh \
  --bootstrap-server localhost:9092 \
  --all-groups \
  --all-topics \
  --describe
```

```text
GROUP     TOPIC       PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG
payment   helloworld  0          20              23              3
```

| Column | Description |
| --- | --- |
| `CURRENT-OFFSET` | Last message read by the consumer group |
| `LOG-END-OFFSET` | Latest message available in the topic |
| `LAG` | Number of unread messages remaining (`LOG-END-OFFSET − CURRENT-OFFSET`) |

---

## 🗂️ Partition

A **Partition** is a unit of storage within a topic. By default, each topic has **1 partition**.

### Partition Rules

- Each partition can only be read by **1 consumer** at a time within a consumer group
- Multiple partitions allow multiple consumers to work **in parallel**
- Increasing partitions enables better **horizontal scaling**

> **Gotcha:** The partition count is the ceiling on parallelism. A topic with 2 partitions and 3 consumers in the same group leaves **one consumer idle**.

### Partition Commands

```bash
# create a topic with multiple partitions
./bin/kafka-topics.sh \
  --bootstrap-server localhost:9092 \
  --create \
  --topic <name> \
  --partitions <number>

# change the partition count of an existing topic
./bin/kafka-topics.sh \
  --bootstrap-server localhost:9092 \
  --alter \
  --topic <name> \
  --partitions <number>
```

> ⚠️ **Warning:** You can only **increase** the partition count, never decrease it. Increasing it also changes which partition each key maps to, so messages for an existing key may land in a different partition afterwards.

### Partition Example

```bash
./bin/kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic helloworld
```

```text
PartitionCount: 1
```

```bash
./bin/kafka-topics.sh --bootstrap-server localhost:9092 --alter --topic helloworld --partitions 2
./bin/kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic helloworld
```

```text
PartitionCount: 2
```

---

## 🔀 Routing & Keys

### How Kafka Routes Messages to Partitions

Kafka uses the message **Key** to determine which partition receives a message:

```text
partition = hash(message.key) % total_partitions

Example:
hash("eko") = 8
8 % 2 = 0   →   message goes to Partition 0
```

> **Key Insight:** Messages with the **same key** always go to the **same partition**, which guarantees ordering per key — every event for `order-42` is processed in the order it was produced.

### Producing and Consuming with Keys

Send keyed messages with the console producer shown in [Sending Messages with a Key](#sending-messages-with-a-key), then read them back inside a consumer group with the keys printed:

```bash
./bin/kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic helloworld \
  --group payment \
  --property "print.key=true"
```

---

## 🔄 Message Flow

Putting the pieces together, this is the path every message takes:

```text
Producer                          sends { key, value } to a topic
  ↓
Partitioner                       hash(key) % partitions → picks a partition
  ↓
Partition log                     message appended to the end, assigned the next offset
  ↓                               stored on disk — kept even after it's read
Consumer group                    each partition is assigned to exactly one consumer in the group
  ↓
Consumer                          reads messages in offset order within its partition
  ↓
Offset commit                     the group records how far it has read
  ↓
Restart                           the group resumes from the committed offset, not from the start
```

That flow explains the behaviour covered throughout this guide:

| Question | Answer |
| --- | --- |
| Why does a second group receive every message again? | Offsets are stored per group, and reading never deletes data from the log |
| Why doesn't a restarted consumer re-read old messages? | It resumes from its group's committed offset |
| Why is one of my consumers idle? | There are more consumers in the group than partitions |
| Why are messages out of order? | Ordering is only guaranteed within a partition — use a key to keep related messages together |
| Why can old messages be replayed? | Kafka keeps messages for a configured retention period, regardless of who has read them |

---

## 🛠️ Kafka Clients

For real applications, use a Kafka client library in your programming language of choice instead of the console scripts.

### Golang

1. **Set Up the Project**:

   ```bash
   mkdir belajar-golang-kafka
   cd belajar-golang-kafka
   go mod init belajar-golang-kafka
   go get github.com/confluentinc/confluent-kafka-go/v2/kafka
   ```

   > Reference: [github.com/confluentinc/confluent-kafka-go](https://github.com/confluentinc/confluent-kafka-go)

2. **Consumer**:

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

3. **Producer**:

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

   > **Note:** `Produce` is asynchronous — it only queues the message. `Flush(5 * 1000)` waits up to 5 seconds for queued messages to be delivered; without it, the program can exit before anything is sent.

### NodeJS

1. **Set Up the Project**:

   ```bash
   mkdir belajar-nodejs-kafka
   cd belajar-nodejs-kafka
   npm init
   npm install kafkajs
   ```

   - Open `package.json` and add `"type": "module"`

   > Reference: [kafka.js.org](https://kafka.js.org/)

2. **Consumer**:

   ```javascript
   import { Kafka } from "kafkajs";

   const kafka = new Kafka({ brokers: ["localhost:9092"] });
   const consumer = kafka.consumer({ groupId: "nodejs" });

   await consumer.connect();
   await consumer.subscribe({ topic: "helloworld", fromBeginning: true });

   await consumer.run({
     eachMessage: async (record) => {
       console.info(record.message.value.toString());
     },
   });
   ```

   > **Gotcha:** Call `connect()` **before** `subscribe()` — kafkajs needs an open connection to subscribe.

3. **Producer**:

   ```javascript
   import { Kafka, Partitioners } from "kafkajs";

   const kafka = new Kafka({ brokers: ["localhost:9092"] });
   const producer = kafka.producer({
     createPartitioner: Partitioners.DefaultPartitioner,
   });

   await producer.connect();

   for (let i = 0; i < 10; i++) {
     await producer.send({
       topic: "helloworld",
       messages: [{ key: `${i}`, value: `Hello ${i}` }],
     });
   }

   await producer.disconnect();
   ```

### Java

1. **Set Up the Project**:

   - Generate a project at `https://start.spring.io/` and add the Kafka dependency

2. **Consumer**:

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

3. **Producer**:

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

   > **Note:** `send()` is asynchronous; calling `.get()` blocks until each message is acknowledged. That's simple for learning, but slow — in production, send without `.get()` and let `close()` flush the rest.

### Client Configuration Compared

| Setting | Golang (`ConfigMap`) | NodeJS (kafkajs) | Java (`Properties`) |
| --- | --- | --- | --- |
| Broker address | `bootstrap.servers` | `brokers` | `BOOTSTRAP_SERVERS_CONFIG` |
| Consumer group | `group.id` | `groupId` | `GROUP_ID_CONFIG` |
| Read from the start | `auto.offset.reset: earliest` | `fromBeginning: true` | `AUTO_OFFSET_RESET_CONFIG: earliest` |
| Ensure delivery before exit | `Flush()` | `disconnect()` | `close()` |

---

## 🎯 Quick Reference

### Concepts

| Concept | Purpose | Key Syntax |
| --- | --- | --- |
| **Message Broker** | Intermediary between senders and receivers | Kafka server on `localhost:9092` |
| **Topic** | Append-only log that stores messages | `--create --topic <name>` |
| **Message** | Unit of data: topic, partition, header, key, value | `key:value` in the console producer |
| **Producer** | Application that sends messages | `kafka-console-producer.sh` |
| **Consumer** | Application that reads messages | `kafka-console-consumer.sh` |
| **Consumer Group** | Consumers that share the work of one topic | `--group <group-name>` |
| **Offset** | Position of the last message read by a group | `kafka-consumer-groups.sh --describe` |
| **Lag** | Unread messages remaining for a group | `LOG-END-OFFSET − CURRENT-OFFSET` |
| **Partition** | Unit of storage and parallelism within a topic | `--partitions <number>` |
| **Key** | Routes related messages to the same partition | `hash(key) % total_partitions` |

### CLI Commands

| Command | Description |
| --- | --- |
| `kafka-storage.sh random-uuid` | Generate a cluster ID |
| `kafka-storage.sh format --cluster-id <id> --config <file> --standalone` | Format the data directory |
| `kafka-server-start.sh config/server.properties` | Start the Kafka server |
| `kafka-topics.sh --create --topic <name>` | Create a topic |
| `kafka-topics.sh --list` | List all topics |
| `kafka-topics.sh --describe --topic <name>` | Describe a topic |
| `kafka-topics.sh --delete --topic <name>` | Delete a topic |
| `kafka-topics.sh --alter --topic <name> --partitions <n>` | Increase the partition count |
| `kafka-console-producer.sh --topic <name>` | Send messages from the console |
| `kafka-console-consumer.sh --topic <name> --from-beginning` | Read all messages from the console |
| `kafka-console-consumer.sh --topic <name> --group <group>` | Read as part of a consumer group |
| `kafka-consumer-groups.sh --all-groups --all-topics --describe` | Show offsets and lag |

> **Note:** Every command above runs from the `bin/` directory and needs `--bootstrap-server localhost:9092`.

---

## 💡 Best Practices

**✅ Do This**

- **Use descriptive topic names** — e.g., `order-created`, `payment-processed`
- **Plan the partition count from expected throughput** — partitions can't be reduced later
- **Always set an explicit `group.id` in production**, named after the consuming application (e.g., `payment-service`)
- **Monitor consumer lag** to detect processing bottlenecks early
- **Use consistent keys when ordering matters** — e.g., `user_id` or `order_id`
- **Call `Flush()`, `disconnect()`, or `close()` in producers** so queued messages are actually sent
- **Set `auto.offset.reset` to `earliest`** for consumers that must not miss messages
- **Use batching and compression** for high-throughput producers
- **Move `log.dirs` away from `/tmp`** so data survives a restart
- **Rely on retention for replay** — Kafka keeps messages for a configurable duration, so consumers can re-read history

**❌ Avoid This**

- **Running consumers without a group in production** — every consumer becomes its own group and processes every message
- **Running more consumers in a group than there are partitions** — the extras sit idle
- **Expecting global ordering across partitions** — order is only guaranteed within a partition
- **Sending ordered events without a key** — keyless messages are spread across partitions, so their relative order is lost
- **Increasing partitions casually on a keyed topic** — existing keys may map to different partitions afterwards
- **Assuming the sender knows a message was processed** — receivers must handle failures and retries themselves
- **Changing a consumer's group ID to "fix" something** — the new group starts with no offset history

> Reference: [Apache Kafka Documentation](https://kafka.apache.org/documentation/) · [kafkajs](https://kafka.js.org/) · [confluent-kafka-go](https://github.com/confluentinc/confluent-kafka-go)
