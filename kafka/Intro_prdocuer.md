**Kafka** is a distributed event streaming platform that lets services communicate asynchronously through events/messages.

2. Kafka Key Concepts
3. 
Term	Meaning
| Term               | Meaning                                   |
| ------------------ | ----------------------------------------- |
| **Producer**       | Sends messages (ex: Order Service)        |
| **Consumer**       | Reads messages (ex: Inventory Service)    |
| **Topic**          | A category/channel to publish messages    |
| **Broker**         | Kafka server node (holds topics/messages) |
| **Partition**      | Subdivision of a topic (for parallelism)  |
| **Offset**         | Unique ID of a message in a partition     |
| **Consumer Group** | Group of consumers sharing the load       |

Run Kafka and Zookeeper Using Docker

Terminal 
Make sure docker is up on running and before we work with kafka we have to first make sure zookeeper and kafka broker is up on running. 

      # Zookeeper
      docker run -d --name zookeeper -e ZOOKEEPER_CLIENT_PORT=2181 confluentinc/cp-zookeeper
 What this does:

      Spins up a Zookeeper container
      Assigns it to kafka-net (so Kafka can find it)
      Sets port 2181 for Zookeeper client connections
      -d = run in background (daemon mode)
      Uses Confluent's official Zookeeper image

Zookeeper is like the "control tower" for Kafka. Kafka won’t start without it. It handles broker leadership, topic configs, health checks.
    
      # Kafka Broker
      docker run -d --name kafka -e KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181 \
        -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://localhost:9092 \
        -e KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR=1 \
        -p 9092:9092 confluentinc/cp-kafka
What this does:

      Spins up a Kafka Broker container
      Connects it to Zookeeper at zookeeper:2181
      Exposes Kafka on localhost:9092 (so our apps can hit it)
      Tells Kafka that the replication factor is 1 (minimum, since we only have 1 broker)
      -p maps Kafka inside Docker (9092) to your local system's 9092 (so Spring Boot can connect)
      Again, uses Confluent's official Kafka image

This is your Kafka server running and ready to accept connections!        

Create a Topic for Orders

    docker exec kafka kafka-topics --create --topic orders \
    --bootstrap-server localhost:9092 --partitions 3 --replication-factor 1
What this does:

    Goes inside the Kafka container
    Uses kafka-topics CLI tool
    Creates a topic called orders
    Partition count = 3 (parallel consumers can read in chunks)
    Replication = 1 (we're running only one broker)

Topic is like a "channel" or "queue" — we send our order messages here
Partitions = scalability; more partitions = more throughput (can be consumed in parallel)
Topic orders is created
Split into 3 partitions for parallelism

lets see how producers and consumers can be created using clis(terminals) and how they communicate.
Terminal (Test Kafka Using CLI)

Creating a producer console to send messages/events 

      docker exec -it kafka kafka-console-producer \
    --broker-list localhost:9092 --topic orders
What this does:
  
    Goes inside the Kafka container
    Launches interactive producer terminal
    Sends your typed messages into the orders topic
Type

    Order Placed: #1001 for item Mobile

this message is send to the topic from producer now lets create a consumer who listens to this topic ( orders)

    docker exec -it kafka kafka-console-consumer \
      --bootstrap-server localhost:9092 --topic orders --from-beginning
What this does:

    Opens a consumer terminal that listens to the orders topic
    --from-beginning means it will read all messages from start (offset 0)

We can see this message read by our consumer
  
      Order Placed: #1001 for item Mobile


**Build a Spring Boot Kafka Producer**

This service:
    
    Exposes an API → /api/orders
    Accepts an Order (orderId, itemName, quantity)
    Sends this order to a Kafka topic called orders

**Configure Kafka Connection**

application.properties

    spring.kafka.bootstrap-servers=localhost:9092

    spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer
    spring.kafka.producer.value-serializer=org.apache.kafka.common.serialization.StringSerializer
    
    app.topic.name=orders


Kafka Connection  This is where you tell Spring Kafka where to connect to your Kafka server.localhost: The Kafka broker is expected to be running on your local machine.
9092: This is the default port Kafka listens on.
    
             spring.kafka.bootstrap-servers=localhost:9092


    
This is a custom Spring property, not a Kafka-native one. You are defining a topic name as a configurable property:  This is just like saying:
"Hey, in my code, wherever I need the topic name, I’ll read it from config instead of hardcoding it."

      app.topic.name=orders


 What are Key and Value Serializers in Kafka?
 
In Kafka, all data is sent and received in byte[] format. But your application usually works with Java objects (like String, Integer, or even 
custom objects like Order, User, etc).So before sending a message to Kafka, the Producer must convert (i.e., serialize) your Java object into a 
byte array.

    Key: Used for partitioning logic (e.g., which partition the message should go to).
    Value: The actual data payload.

Why do we need them?
Because Kafka itself only understands bytes, not Java objects.
So, Kafka Producer uses serializers to convert your keys and values into byte arrays that Kafka can store and transfer between brokers.
Without specifying serializers, Kafka doesn’t know how to turn your Java String, int, or POJO into byte[] — and you’ll get runtime errors.

      spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer  
    spring.kafka.producer.value-serializer=org.apache.kafka.common.serialization.StringSerializer
Key will be a String → Serialize using StringSerializer 
Value will also be a String

If you're sending JSON, use JsonSerializer

      spring.kafka.producer.value-serializer=org.springframework.kafka.support.serializer.JsonSerializer
If you're using custom objects like Order, and want them as JSON in Kafka, you'll need:

JsonSerializer for value
Possibly custom serializer if it’s a complex type



Create Order.java (POJO)
Why?
To hold the data of the incoming order from API:
     
       {
      "orderId": "ORD-101",
      "itemName": "iPhone",
      "quantity": 2
    }

model/Order.java

    package com.kafka.producer.model;
    
    import lombok.AllArgsConstructor;
    import lombok.Data;
    import lombok.NoArgsConstructor;
    
    @Data
    @AllArgsConstructor
    @NoArgsConstructor
    public class Order {
        private String orderId;
        private String itemName;
        private int quantity;
    }

 KafkaProducerService.java (Business Layer)

 This class:
    
    Takes an Order object
    Converts it into a string (for now
    Sends it to Kafka topic using KafkaTemplate
In Spring Boot, we use the class KafkaTemplate<K, V> to communicate with Kafka — it's the main helper class to send messages from your 
Java code to Kafka topics.

service/KafkaProducerService.java

    package com.kafka.producer.service;
    
    import com.kafka.producer.model.Order;
    import org.springframework.beans.factory.annotation.Value;
    import org.springframework.kafka.core.KafkaTemplate;
    import org.springframework.stereotype.Service;
    
       @Service
    @RequiredArgsConstructor
    public class KafkaProducerService {
    
        @Value("${app.topic.name}")
        private String topicName;
    
        private final KafkaTemplate<String, String> kafkaTemplate;
    
        public void sendOrder(Order order) {
            String message = "OrderID: " + order.getOrderId()
                    + ", Item: " + order.getItemName()
                    + ", Quantity: " + order.getQuantity();
    
            kafkaTemplate.send(topicName, message);
            System.out.println("✅ Order sent to Kafka → " + message);
        }
    }


OrderController.java (API Layer)

 Why?
 
      It exposes the POST API:
      POST /api/orders
      Accepts order JSON → maps to Order → calls service to send to Kafka.

 controller/OrderController.java

      package com.kafka.producer.controller;
    
    import com.kafka.producer.model.Order;
    import com.kafka.producer.service.KafkaProducerService;
    import org.springframework.http.ResponseEntity;
    import org.springframework.web.bind.annotation.*;
    
    @RestController
    @RequestMapping("/api/orders")
    public class OrderController {
    
        private final KafkaProducerService producerService;
    
        public OrderController(KafkaProducerService producerService) {   // or use @RequriedArgsConstructor from lombok and need not to write this method
            this.producerService = producerService;
        }
    
        @PostMapping
        public ResponseEntity<String> placeOrder(@RequestBody Order order) {
            producerService.sendOrder(order);
            return ResponseEntity.ok("✅ Order sent to Kafka successfully!");
        }
    }


Open Postman

      POST http://localhost:8080/api/orders

      {
    "orderId": "ORD-2025",
    "itemName": "AirPods Pro",
    "quantity": 3
    }

check Kafka Consumer in CLI:

    docker exec -it kafka kafka-console-consumer \
      --bootstrap-server localhost:9092 \
      --topic orders \
      --from-beginning

Now let's build a consumer
Create the Project

Application.properties

          # Kafka Setup
    spring.kafka.bootstrap-servers=localhost:9092
    
    # Consumer group name
    spring.kafka.consumer.group-id=inventory-group
    
    # How to convert Kafka byte messages to Strings
    spring.kafka.consumer.key-deserializer=org.apache.kafka.common.serialization.StringDeserializer
    spring.kafka.consumer.value-deserializer=org.apache.kafka.common.serialization.StringDeserializer
    
    # Optional: start from beginning if no previous offset
    spring.kafka.consumer.auto-offset-reset=earliest
    
    # Run on different port from producer
    server.port=8082

This sets up our service to:

    Connect to Kafka broker on localhost:9092
    Join a consumer group inventory-group
    Listen to topic orders

  com.kafka.consumer.model.Order.java
    
      package com.kafka.consumer.model;
    
    import lombok.AllArgsConstructor;
    import lombok.Data;
    import lombok.NoArgsConstructor;
    
    @Data
    @AllArgsConstructor
    @NoArgsConstructor
    public class Order {
        private String orderId;
        private String itemName;
        private int quantity;
    }

com.kafka.consumer.service.KafkaConsumerService.java

This is the actual Kafka listener method. It will get triggered automatically whenever a message is published to topic orders.

    
    package com.kafka.consumer.service;
    
    import org.springframework.kafka.annotation.KafkaListener;
    import org.springframework.stereotype.Service;
    
    @Service
    public class KafkaConsumerService {
    
        @KafkaListener(topics = "orders", groupId = "inventory-group")
        public void consume(String message) {
            System.out.println(" Received order from Kafka:");
            System.out.println(" " + message);
            System.out.println("Inventory updated for this order.\n");
        }
    }

This uses @KafkaListener which:
    
    Subscribes to orders topic
    Triggers this method when a message arrives
    Prints the received order and simulates inventory update

  Run InventoryService on port 8082
It starts and waits for Kafka messages

 2. Go to OrderService (on port 8080)
    
        Send a POST request via Postman to /api/orders
    
        {
          "orderId": "ORD-2026",
          "itemName": "MacBook Air",
          "quantity": 1
        }
As soon as you send this:
      
      Kafka broker stores the message
      
      InventoryService immediately consumes it
      
      Console will show:

       Received order from Kafka:
     OrderID: ORD-2026, Item: MacBook Air, Quantity: 1
    Inventory updated for this order.
