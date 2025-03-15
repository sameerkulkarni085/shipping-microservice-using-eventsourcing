# Event Sourcing with Spring Boot Microservices

This project demonstrates the implementation of an **event sourcing** strategy using **Spring Boot microservices**. The application consists of two services:
- **order-service**: Handles order creation and confirmation.
- **shipping-service**: Manages shipping and delivery processes.

Both services communicate asynchronously using **Apache Kafka**, showcasing an event-driven architecture for order lifecycle management.

---

## Event Sourcing - Overview

**Event Sourcing** is an architectural pattern where all changes to the application state are stored as a sequence of events.  
Instead of only saving the current state, it records each state change as an event, which can:
- Reconstruct past states.
- Replay events to restore or predict system behavior.

Traditional systems only retain the latest state, losing the history of transitions. For example:
- An order may transition through `CREATED → CONFIRMED → SHIPPED → DELIVERED`.
- Without event sourcing, previous states like `CREATED` or `CONFIRMED` are lost, making it hard to audit or troubleshoot.


### Architecture Overview

- **order-service**: Publishes events to Kafka when orders are created or confirmed.
- **shipping-service**: Listens to order events from Kafka and processes shipping actions.

---

### Order Microservice (`order-service`)

#### Overview:
The `order-service` is responsible for:
- Creating orders.
- Confirming orders.

#### Key Technologies:
- Spring Boot Web
- MongoDB with Spring Data
- Apache Kafka for messaging
- Lombok for simplifying code
- Spring Boot Devtools for development ease

#### Configuration Snippet:
```yaml
spring:
  application:
    name: order-service
  kafka:
    template:
      default-topic: order-events
server:
  port: 8081
```

#### Kafka Producer Setup:
```yaml
spring:
  kafka:
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.springframework.kafka.support.serializer.JsonSerializer
```

#### API Endpoints:
- **POST `/orders/create`**: Adds a new order and sends an event to Kafka.
- **PUT `/orders/confirm/{orderId}`**: Updates an order to "confirmed" and publishes an event.

#### Key Features:
- **Serialization**:  
  - **Key Serializer**: Converts keys to strings for Kafka.
  - **Value Serializer**: Converts order objects to JSON for seamless data exchange.


## Shipping Microservice (`shipping-service`)

### Shipping Microservice (`shipping-service`)

#### Overview:
The `shipping-service` manages:
- Orders confirmed by the `order-service`.
- Shipping and delivery processes.

#### Key Technologies:
- Uses the same dependencies as the `order-service` (Spring Boot, Kafka, MongoDB).
- Focuses on Kafka listeners for consuming messages and MongoDB for data storage.

#### Configuration Snippet:
```yaml
spring:
  kafka:
    consumer:
      group-id: shipping-service
server:
  port: 8082
```

#### Kafka Consumer Settings:
```yaml
spring:
  kafka:
    consumer:
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.springframework.kafka.support.serializer.ErrorHandlingDeserializer
      properties:
        spring.deserializer.key.delegate.class: org.apache.kafka.common.serialization.StringDeserializer
        spring.deserializer.value.delegate.class: org.springframework.kafka.support.serializer.JsonDeserializer
        spring.json.trusted.packages: '*'
```

#### API Endpoints:
- **POST `/shipping/ship/{orderId}`**: Automatically processes shipping when an order is confirmed. Triggered by the Kafka listener.
- **POST `/shipping/deliver/{orderId}`**: Manually processes the delivery of an order.

#### Key Features:
- **Deserialization**:  
  - **Key Deserializer**: Converts Kafka message keys back to string format.  
  - **Value Deserializer**: Converts JSON messages into order event objects and handles errors gracefully.

---
