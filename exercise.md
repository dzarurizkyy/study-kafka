# 🚚 Kafka Hands-On Practice — Food Delivery Order System

A complete hands-on scenario that covers all core Apache Kafka concepts: topics, producers, consumers, consumer groups, offsets, partitions, key-based routing, and client integrations — all in one real-world food delivery use case.

---

## 📖 Scenario

You are building the messaging backbone for a food delivery platform called **MakanCepat**. When a customer places an order, multiple services need to react to it:

- **Kitchen Service** — receives the order to start cooking
- **Payment Service** — processes the payment
- **Logistics Service** — assigns a driver to pick up the order

Instead of the Order Service calling each downstream service directly (RPC), you will use **Kafka as the message broker** so the Order Service only needs to publish — and every other service subscribes independently.

By the end of this practice, you will have hands-on experience with every concept in the Kafka guide.

---

## 🗂️ Table of Contents

1. [Setup & Verify](#1-setup--verify)
2. [Create Topics](#2-create-topics)
3. [Produce & Consume Messages](#3-produce--consume-messages)
4. [Consumer Groups](#4-consumer-groups)
5. [Offset Management](#5-offset-management)
6. [Partitions & Parallel Consumers](#6-partitions--parallel-consumers)
7. [Key-Based Routing](#7-key-based-routing)
8. [Kafka Clients](#8-kafka-clients)
9. [Challenge Tasks](#-challenge-tasks)

---

## 1. Setup & Verify

Make sure Kafka is running before starting.

```bash
# Generate cluster ID (only once)
./bin/kafka-storage.sh random-uuid

# Format data directory (only once)
./bin/kafka-storage.sh format \
  --cluster-id <your-uuid> \
  --config config/server.properties \
  --standalone

# Start Kafka
./bin/kafka-server-start.sh config/server.properties
```

> ⚠️ Make sure `log.dirs=data` is set in `config/server.properties` to avoid data loss on system restart.

Verify Kafka is up by listing topics (should return empty, no error):

```bash
./bin/kafka-topics.sh --bootstrap-server localhost:9092 --list
```

---

## 2. Create Topics

MakanCepat needs three topics — one per downstream service.

```bash
# Orders received by kitchen
./bin/kafka-topics.sh --bootstrap-server localhost:9092 --create --topic order-created

# Payment events
./bin/kafka-topics.sh --bootstrap-server localhost:9092 --create --topic payment-processed

# Driver assignment events
./bin/kafka-topics.sh --bootstrap-server localhost:9092 --create --topic driver-assigned
```

Verify all topics were created:

```bash
./bin/kafka-topics.sh --bootstrap-server localhost:9092 --list
```

Expected output:

```
driver-assigned
order-created
payment-processed
```

Describe a topic to see its details:

```bash
./bin/kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic order-created
```

Expected output:

```
Topic: order-created  PartitionCount: 1  ReplicationFactor: 1
  Topic: order-created  Partition: 0  Leader: 1  Replicas: 1  Isr: 1
```

> ❓ **Notice** — every topic starts with 1 partition by default. We will scale this up in Step 6.

---

## 3. Produce & Consume Messages

### 3a. Start a Consumer (Terminal 1)

Open a terminal and start listening to `order-created`:

```bash
./bin/kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic order-created \
  --from-beginning
```

> Keep this terminal open and running.

### 3b. Send Messages from a Producer (Terminal 2)

Open a second terminal and send some orders:

```bash
./bin/kafka-console-producer.sh \
  --bootstrap-server localhost:9092 \
  --topic order-created
```

Type these messages one by one, pressing Enter after each:

```
{"order_id": "ORD-001", "customer": "Dzaru", "item": "Nasi Goreng", "total": 25000}
{"order_id": "ORD-002", "customer": "Sari", "item": "Mie Ayam", "total": 18000}
{"order_id": "ORD-003", "customer": "Budi", "item": "Ayam Bakar", "total": 35000}
```

Watch Terminal 1 — each message should appear instantly as you send it.

> ❓ **This is Pub/Sub in action** — the producer has no idea who is consuming. It just publishes to the topic and moves on.

### 3c. Start a Second Consumer (Terminal 3)

Without stopping the first consumer, open a third terminal with a **new consumer** (no group specified):

```bash
./bin/kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic order-created \
  --from-beginning
```

> ❓ **What do you observe?** Both consumers receive all 3 messages independently. This is because they each belong to a **different auto-generated group** — both are getting the full feed.

---

## 4. Consumer Groups

In real life, Kitchen, Payment, and Logistics are separate services. Each should receive **every** order, but within each service, only **one instance** should process each message.

### 4a. Open Three Consumer Terminals — One Per Service

**Terminal 1 — Kitchen Service:**

```bash
./bin/kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic order-created \
  --group kitchen-service \
  --from-beginning
```

**Terminal 2 — Payment Service:**

```bash
./bin/kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic order-created \
  --group payment-service \
  --from-beginning
```

**Terminal 3 — Logistics Service:**

```bash
./bin/kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic order-created \
  --group logistics-service \
  --from-beginning
```

### 4b. Send a New Order (Terminal 4)

```bash
./bin/kafka-console-producer.sh \
  --bootstrap-server localhost:9092 \
  --topic order-created
```

```
{"order_id": "ORD-004", "customer": "Rizky", "item": "Soto Betawi", "total": 30000}
```

> ❓ **What do you observe?** All three consumers (kitchen, payment, logistics) receive the same message — because they are in **different groups**. Each group gets its own independent feed.

### 4c. Add a Second Instance of Kitchen Service (Terminal 5)

```bash
./bin/kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic order-created \
  --group kitchen-service
```

Send another order from Terminal 4:

```
{"order_id": "ORD-005", "customer": "Indah", "item": "Gado-Gado", "total": 22000}
```

> ❓ **What do you observe?** Only **one** of the two kitchen-service consumers receives `ORD-005` — not both. This is consumer group load balancing in action. Since there is only 1 partition, one consumer sits idle. We will fix this in Step 6.

---

## 5. Offset Management

### 5a. Check Current Offsets

Stop all consumers first (Ctrl+C on each), then run:

```bash
./bin/kafka-consumer-groups.sh \
  --bootstrap-server localhost:9092 \
  --all-groups \
  --all-topics \
  --describe
```

Expected output:

```
GROUP              TOPIC         PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG
kitchen-service    order-created 0          5               5               0
payment-service    order-created 0          5               5               0
logistics-service  order-created 0          5               5               0
```

> `LAG: 0` means all groups are fully caught up — no unread messages.

### 5b. Simulate a Consumer Falling Behind

Start only the kitchen-service consumer:

```bash
./bin/kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic order-created \
  --group kitchen-service
```

Send 3 new orders **without** starting payment-service or logistics-service:

```bash
./bin/kafka-console-producer.sh \
  --bootstrap-server localhost:9092 \
  --topic order-created
```

```
{"order_id": "ORD-006", "customer": "Andi", "item": "Bakso", "total": 20000}
{"order_id": "ORD-007", "customer": "Dewi", "item": "Rendang", "total": 45000}
{"order_id": "ORD-008", "customer": "Fajar", "item": "Es Teh", "total": 8000}
```

Stop the kitchen-service consumer, then check offsets again:

```bash
./bin/kafka-consumer-groups.sh \
  --bootstrap-server localhost:9092 \
  --all-groups \
  --all-topics \
  --describe
```

Expected output:

```
GROUP              TOPIC         PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG
kitchen-service    order-created 0          8               8               0
payment-service    order-created 0          5               8               3
logistics-service  order-created 0          5               8               3
```

> `LAG: 3` on payment and logistics means they missed 3 messages. When they restart, they will automatically resume from offset 5 — Kafka remembers where they left off.

### 5c. Verify Resume Behavior

Start payment-service and watch it catch up:

```bash
./bin/kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic order-created \
  --group payment-service
```

> ❓ **What do you observe?** It immediately reads ORD-006, ORD-007, and ORD-008 — the 3 messages it missed — without needing `--from-beginning`.

---

## 6. Partitions & Parallel Consumers

Currently, `order-created` has only 1 partition, which means only 1 consumer in a group can work at a time. Let's scale it.

### 6a. Increase Partitions

```bash
./bin/kafka-topics.sh \
  --bootstrap-server localhost:9092 \
  --alter \
  --topic order-created \
  --partitions 3
```

Verify:

```bash
./bin/kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic order-created
```

Expected output:

```
Topic: order-created  PartitionCount: 3  ReplicationFactor: 1
  Topic: order-created  Partition: 0  Leader: 1
  Topic: order-created  Partition: 1  Leader: 1
  Topic: order-created  Partition: 2  Leader: 1
```

> ⚠️ Partitions can only be **increased**, never decreased.

### 6b. Start 3 Kitchen-Service Consumers in Parallel

Open 3 separate terminals, each running:

```bash
# Terminal 1
./bin/kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic order-created \
  --group kitchen-service

# Terminal 2
./bin/kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic order-created \
  --group kitchen-service

# Terminal 3
./bin/kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic order-created \
  --group kitchen-service
```

### 6c. Send a Burst of Orders

```bash
./bin/kafka-console-producer.sh \
  --bootstrap-server localhost:9092 \
  --topic order-created
```

```
{"order_id": "ORD-009", "customer": "A", "item": "Nasi Uduk", "total": 15000}
{"order_id": "ORD-010", "customer": "B", "item": "Ketoprak", "total": 12000}
{"order_id": "ORD-011", "customer": "C", "item": "Lontong Sayur", "total": 18000}
{"order_id": "ORD-012", "customer": "D", "item": "Bubur Ayam", "total": 14000}
{"order_id": "ORD-013", "customer": "E", "item": "Pempek", "total": 28000}
{"order_id": "ORD-014", "customer": "F", "item": "Siomay", "total": 20000}
```

> ❓ **What do you observe?** Each of the 3 consumers receives a different subset of messages — Kafka distributes partitions across consumers in the same group, enabling true parallel processing.

---

## 7. Key-Based Routing

In MakanCepat, all orders from the same customer must be processed **in order** (pay before cook, cook before deliver). We use the `customer_id` as the message key to guarantee ordering per customer.

### 7a. Start a Consumer That Prints Keys (Terminal 1)

```bash
./bin/kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic order-created \
  --group kitchen-service \
  --property "print.key=true"
```

### 7b. Send Keyed Messages (Terminal 2)

```bash
./bin/kafka-console-producer.sh \
  --bootstrap-server localhost:9092 \
  --topic order-created \
  --property "parse.key=true" \
  --property "key.separator=:"
```

Send these messages (format is `key:value`):

```
cust-1:{"order_id": "ORD-015", "customer": "Dzaru", "item": "Nasi Goreng", "total": 25000}
cust-2:{"order_id": "ORD-016", "customer": "Sari", "item": "Mie Ayam", "total": 18000}
cust-1:{"order_id": "ORD-017", "customer": "Dzaru", "item": "Es Jeruk", "total": 8000}
cust-3:{"order_id": "ORD-018", "customer": "Budi", "item": "Ayam Bakar", "total": 35000}
cust-2:{"order_id": "ORD-019", "customer": "Sari", "item": "Teh Manis", "total": 5000}
cust-1:{"order_id": "ORD-020", "customer": "Dzaru", "item": "Kerupuk", "total": 3000}
```

> ❓ **What do you observe?** All messages with key `cust-1` always land on the same partition — guaranteeing that Dzaru's orders (ORD-015, ORD-017, ORD-020) are always processed in sequence. The routing formula is: `partition = hash(key) % total_partitions`.

---

## 8. Kafka Clients

Now let's implement a real producer and consumer using code — simulating how the **Order Service** publishes events and how the **Payment Service** consumes them.

### 8a. Golang

`Setup:`

```bash
mkdir makancepat-golang
cd makancepat-golang
go mod init makancepat-golang
go get github.com/confluentinc/confluent-kafka-go/v2/kafka
```

`Producer — Order Service (producer.go):`

```go
package main

import (
  "fmt"
  "github.com/confluentinc/confluent-kafka-go/v2/kafka"
)

func main() {
  producer, err := kafka.NewProducer(&kafka.ConfigMap{
    "bootstrap.servers": "localhost:9092",
  })
  if err != nil {
    panic(err)
  }
  defer producer.Close()

  topic := "order-created"
  orders := []struct {
    Key   string
    Value string
  }{
    {"cust-1", `{"order_id":"ORD-101","customer":"Dzaru","item":"Nasi Goreng","total":25000}`},
    {"cust-2", `{"order_id":"ORD-102","customer":"Sari","item":"Mie Ayam","total":18000}`},
    {"cust-1", `{"order_id":"ORD-103","customer":"Dzaru","item":"Es Jeruk","total":8000}`},
    {"cust-3", `{"order_id":"ORD-104","customer":"Budi","item":"Ayam Bakar","total":35000}`},
  }

  for _, order := range orders {
    producer.Produce(&kafka.Message{
      TopicPartition: kafka.TopicPartition{Topic: &topic, Partition: kafka.PartitionAny},
      Key:            []byte(order.Key),
      Value:          []byte(order.Value),
    }, nil)
    fmt.Printf("Published: %s\n", order.Value)
  }

  producer.Flush(5 * 1000)
  fmt.Println("All orders published.")
}
```

`Consumer — Payment Service (consumer.go):`

```go
package main

import (
  "fmt"
  "time"
  "github.com/confluentinc/confluent-kafka-go/v2/kafka"
)

func main() {
  consumer, err := kafka.NewConsumer(&kafka.ConfigMap{
    "bootstrap.servers": "localhost:9092",
    "group.id":          "payment-service",
    "auto.offset.reset": "earliest",
  })
  if err != nil {
    panic(err)
  }
  defer consumer.Close()

  consumer.Subscribe("order-created", nil)
  fmt.Println("Payment Service listening for orders...")

  for {
    message, err := consumer.ReadMessage(1 * time.Second)
    if err == nil {
      fmt.Printf("[Payment] Processing order from customer %s: %s\n",
        string(message.Key), string(message.Value))
    }
  }
}
```

Run the consumer first, then the producer in a separate terminal:

```bash
go run consumer.go
go run producer.go
```

### 8b. NodeJS

`Setup:`

```bash
mkdir makancepat-nodejs
cd makancepat-nodejs
npm init -y
npm install kafkajs
# Add "type": "module" to package.json
```

`Producer — Order Service (producer.mjs):`

```javascript
import { Kafka, Partitioners } from "kafkajs";

const kafka = new Kafka({ brokers: ["localhost:9092"] });
const producer = kafka.producer({ createPartitioner: Partitioners.DefaultPartitioner });

await producer.connect();

const orders = [
  { key: "cust-1", value: JSON.stringify({ order_id: "ORD-201", customer: "Dzaru", item: "Nasi Goreng", total: 25000 }) },
  { key: "cust-2", value: JSON.stringify({ order_id: "ORD-202", customer: "Sari", item: "Mie Ayam", total: 18000 }) },
  { key: "cust-1", value: JSON.stringify({ order_id: "ORD-203", customer: "Dzaru", item: "Es Jeruk", total: 8000 }) },
  { key: "cust-3", value: JSON.stringify({ order_id: "ORD-204", customer: "Budi", item: "Ayam Bakar", total: 35000 }) },
];

for (const order of orders) {
  await producer.send({ topic: "order-created", messages: [order] });
  console.log(`Published: ${order.value}`);
}

await producer.disconnect();
console.log("All orders published.");
```

`Consumer — Kitchen Service (consumer.mjs):`

```javascript
import { Kafka } from "kafkajs";

const kafka = new Kafka({ brokers: ["localhost:9092"] });
const consumer = kafka.consumer({ groupId: "kitchen-service" });

await consumer.connect();
await consumer.subscribe({ topic: "order-created", fromBeginning: true });

console.log("Kitchen Service listening for orders...");

await consumer.run({
  eachMessage: async ({ message }) => {
    console.log(`[Kitchen] Preparing order from customer ${message.key}: ${message.value}`);
  },
});
```

### 8c. Java

`Setup:` Generate a project at [https://start.spring.io](https://start.spring.io) and add the **Spring for Apache Kafka** dependency.

`Producer — Order Service:`

```java
package makancepat.kafka;

import org.apache.kafka.clients.producer.*;
import org.apache.kafka.common.serialization.StringSerializer;
import java.util.*;

public class OrderProducer {
  public static void main(String[] args) throws Exception {
    Properties props = new Properties();
    props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
    props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
    props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());

    KafkaProducer<String, String> producer = new KafkaProducer<>(props);

    String[][] orders = {
      {"cust-1", "{\"order_id\":\"ORD-301\",\"customer\":\"Dzaru\",\"item\":\"Nasi Goreng\",\"total\":25000}"},
      {"cust-2", "{\"order_id\":\"ORD-302\",\"customer\":\"Sari\",\"item\":\"Mie Ayam\",\"total\":18000}"},
      {"cust-1", "{\"order_id\":\"ORD-303\",\"customer\":\"Dzaru\",\"item\":\"Es Jeruk\",\"total\":8000}"},
      {"cust-3", "{\"order_id\":\"ORD-304\",\"customer\":\"Budi\",\"item\":\"Ayam Bakar\",\"total\":35000}"},
    };

    for (String[] order : orders) {
      ProducerRecord<String, String> record =
        new ProducerRecord<>("order-created", order[0], order[1]);
      producer.send(record).get();
      System.out.println("Published: " + order[1]);
    }

    producer.close();
    System.out.println("All orders published.");
  }
}
```

`Consumer — Logistics Service:`

```java
package makancepat.kafka;

import org.apache.kafka.clients.consumer.*;
import org.apache.kafka.common.serialization.StringDeserializer;
import java.time.Duration;
import java.util.*;

public class LogisticsConsumer {
  public static void main(String[] args) {
    Properties props = new Properties();
    props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
    props.put(ConsumerConfig.GROUP_ID_CONFIG, "logistics-service");
    props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
    props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
    props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());

    KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
    consumer.subscribe(List.of("order-created"));

    System.out.println("Logistics Service listening for orders...");

    while (true) {
      ConsumerRecords<String, String> records = consumer.poll(Duration.ofSeconds(1));
      for (ConsumerRecord<String, String> record : records) {
        System.out.printf("[Logistics] Assigning driver for customer %s: %s%n",
          record.key(), record.value());
      }
    }
  }
}
```

---

## 🏆 Challenge Tasks

Once you've completed the practice above, try these on your own:

1. **Create a `payment-failed` topic** and simulate a payment failure flow — produce a failed payment event and consume it from a `retry-service` group.

2. **Scale `payment-processed`** to 3 partitions, then start 3 consumers in the `payment-service` group. Send 9 messages and observe how Kafka distributes them across consumers.

3. **Simulate lag recovery** — stop all consumers for `logistics-service`, produce 10 new orders, then restart the consumer and verify it catches up all 10 messages using the offset describe command.

4. **Use `driver_id` as the message key** on the `driver-assigned` topic so that all assignments for the same driver always land on the same partition (preserving ordering per driver).

5. **Implement a Golang producer** that sends to `payment-processed` and a **NodeJS consumer** that reads from the same topic using `group.id: audit-service` — proving Kafka clients are language-agnostic.

6. **Delete the `driver-assigned` topic** when it's no longer needed and verify it no longer appears in the topic list.

---

## ✅ Concepts Covered

| Concept | Where Practiced |
|---|---|
| Pub/Sub vs RPC | Scenario intro — Order Service publishes, 3 services subscribe |
| Topic creation & deletion | Step 2 |
| Describe topic | Step 2 |
| Console Producer | Step 3b, 7b |
| Console Consumer | Step 3a, 7a |
| Multiple independent consumers | Step 3c |
| Consumer Groups — different groups | Step 4a, 4b |
| Consumer Groups — same group load balancing | Step 4c |
| Offset tracking | Step 5a |
| Lag simulation & recovery | Step 5b, 5c |
| Increasing partitions | Step 6a |
| Parallel consumers per partition | Step 6b, 6c |
| Key-based routing & ordering guarantee | Step 7 |
| Golang Producer & Consumer | Step 8a |
| NodeJS Producer & Consumer | Step 8b |
| Java Producer & Consumer | Step 8c |
