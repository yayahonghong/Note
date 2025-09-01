# Kafka

消息队列是现代分布式系统中至关重要的组件，它的存在主要解决了以下几个核心问题

1. **解耦系统组件**

2. **异步处理**

3. **流量削峰**



## 安装

使用docker compose部署

```yaml
services:
  kafka:
    image: bitnami/kafka:3.6
    container_name: kafka
    restart: unless-stopped
    environment:
      - KAFKA_CFG_NODE_ID=0
      - KAFKA_CFG_PROCESS_ROLES=controller,broker
      - KAFKA_CFG_LISTENERS=PLAINTEXT://:9092,CONTROLLER://:9093
      - KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP=CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT
      - KAFKA_CFG_CONTROLLER_QUORUM_VOTERS=0@kafka:9093
      - KAFKA_CFG_CONTROLLER_LISTENER_NAMES=CONTROLLER
      - KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://localhost:9092
    volumes:
      - kafka_data:/bitnami/kafka
    ports:
      - "9092:9092"
      - "9093:9093"
    networks:
      - app-network

volumes:
  kafka_data:
    driver: local

networks:
  app-network:
    driver: bridge
```



## Java客户端

**引入依赖**

```xml
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-clients</artifactId>
            <version>3.9.1</version>
        </dependency>
```



**生产者简单使用**

```java
        // 创建Kafka生产者配置
        Map<String, Object> config = Map.of(
                ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092",
                ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName(),
                ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName()
        );

        // 创建Kafka生产者
        try (KafkaProducer<String, String> producer = new KafkaProducer<>(config)) {
            for (int i = 0; i < 10; i++) {
                ProducerRecord<String, String> record = new ProducerRecord<>(
                        "test-topic",
                        "key-" + i,
                        String.valueOf(i)
                );
                producer.send(record);
            }
            producer.flush(); // 确保消息被发送
        }
```



**消费者简单使用**

```java
        // 创建Kafka消费者配置
        Map<String, Object> consumerConfig = Map.of(
                ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092",
                ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName(),
                ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName(),
                ConsumerConfig.GROUP_ID_CONFIG, "test-group",
                ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest" // 从最早的消息开始消费
        );

        // 创建Kafka消费者
        try (KafkaConsumer<String, String> consumer = new KafkaConsumer<>(consumerConfig)) {
            consumer.subscribe(List.of("test-topic"));

            // 多次轮询以确保获取到消息
            for (int i = 0; i < 5; i++) {
                ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(1000));
                for (ConsumerRecord<String, String> record : records) {
                    System.out.println("Received record: " + record);
                }
                if (!records.isEmpty()) {
                    break; // 如果获取到消息就退出循环
                }
            }
        }
```

