---
layout: post
title: "[kafka] 3.Spring boot에서 Kafka 사용하기"
description: "Kafka를 Spring boot에서 사용하는 법"
date: 2021-01-01 00:00:00
tags: [kafka, spring boot]
comments: true
share: true
---

Spring boot에서 Kafka를 사용하기 위해 다음 과정을 따른다.



## Kafka 

### 공통
1. Spring-kafka 의존성을 추가

```gradle
dependencies {
    implementation 'org.springframework.kafka:spring-kafka'
}
```
> spring kafka : apache-kafka를 spring에서 사용하기 쉽게 만들어둔 패키지

2. application.yml 작성

```yaml
spring:
  kafka:
    consumer:
      group-id: myGroup
    bootstrap-servers: localhost:9092

kafka:
  topics:
    hello: test
```



3. Consumer, Producer, Topic에 대한 Config 파일 설정

```java
@Configuration
public class KafkaProducerConfig {
    @Value("${spring.kafka.bootstrap-servers}")
    private String bootstrapServers;

    @Bean
    public Map<String, Object> producerConfig() {
        Map<String, Object> props = new HashMap<>();
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        return props;
    }

    @Bean
    public ProducerFactory<String, String> producerFactory() {
        return new DefaultKafkaProducerFactory<>(producerConfig());
    }

    @Bean
    public KafkaTemplate<String, String> kafkaTemplate() {
        return new KafkaTemplate<>(producerFactory());
    }
}
```
> Kafka Producer 설정



```java
@Configuration
public class KafkaConsumerConfig {
    @Value("${spring.kafka.bootstrap-servers}")
    private String bootstrapServers;

    @Bean
    public Map<String, Object> consumerConfig() {
        Map<String, Object> props = new HashMap<>();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        return props;
    }

    @Bean
    public ConsumerFactory<String, String> consumerFactory() {
        return new DefaultKafkaConsumerFactory<>(consumerConfig());
    }

    @Bean
    public KafkaListenerContainerFactory<ConcurrentMessageListenerContainer<String, String>> kafkaListenerContainerFactory() {
        ConcurrentKafkaListenerContainerFactory<String, String> factory =
                new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(consumerFactory());
        return factory;
    }
}
```
> Kafka Consumer  설정



```java
@EnableKafka
@Configuration
public class KafkaTopicConfig {
    @Value("${spring.kafka.bootstrap-servers}")
    private String bootstrapAddress;

    @Value("${kafka.topics.hello}")
    private String topicName;

    @Bean
    public KafkaAdmin kafkaAdmin() {
        Map<String, Object> props = new HashMap<>();
        props.put(AdminClientConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapAddress);
        return new KafkaAdmin(props);
    }

    @Bean
    public NewTopic helloTopic() {
        return new NewTopic(topicName, 1, (short)1);
    }

    @Bean
    public NewTopic helloTopic2() {
        return TopicBuilder
                .name(topicName)
                .partitions(1)
                .build();
    }
}
```
> Kafka Topic 설정



### Producer

4. Producer 컴포넌트 생성

```java
@Component
public class Producer {
    @Autowired
    private KafkaTemplate<String, String> kafkaTemplate;

    public void sendMessage(NewTopic topic, String message) {
        kafkaTemplate.send(topic.name(), message);
    }
    
    public void sendMessageCallback(NewTopic topic, String message) {
        ListenableFuture<SendResult<String, String>> future =
                kafkaTemplate.send(topic.name(), message);

        future.addCallback(new ListenableFutureCallback<SendResult<String, String>>() {
            @Override
            public void onFailure(Throwable ex) {
                System.out.println(ex.getMessage());
            }

            @Override
            public void onSuccess(SendResult<String, String> result) {
                System.out.println(message);
                System.out.println(result.getRecordMetadata().offset());
            }
        });
    }
}
```

5. Controller 생성

```java
@RestController
public class TestController {
    @Autowired
    private NewTopic helloTopic;

    @Autowired
    private Producer producer;

    @GetMapping(value = "/publish/{message}")
    public String publish(@PathVariable String message) {
        producer.sendMessage(helloTopic, message);

        return "HELLO";
    }
}
```

### Consumer

@KafkaListener 어노테이션을 사용하기 위해서는  Config에 @EnalbeKafka가 추가 되어야 한다.

```java
@Component
public class Consumer {
    @KafkaListener(topics = "${kafka.topics.hello}", groupId = "${spring.kafka.consumer.group-id}")
    void listen(String message) {
        ...
    }
}
```

- 여러 개의 토픽에서 메시지를 수신 하는 법

 ```java
 @KafkaListener(topics = "topic-1, topic-2", groupId = "myGroup")
 void listenMulti(String message) {
     ....
 }
 ```

 - partition offset 설정하기

 ```java
 @KafkaListener(groupId = "myGroup", topicPartitions = @TopicPartition(
         topic = "topic-1",
         partitionOffsets = { @PartitionOffset(
             partition = "0", 
             initialOffset = "0") }))
 void listenOffset(
     @Payload String message,
     @Header(KafkaHeaders.RECEIVED_PARTITION_ID) int partition,
     @Header(KafkaHeaders.OFFSET) int offset) {
     ....
 }
 ```
> @Headers MessageHeaders messageHeaders로 전체 헤더 가능
- 메시지 수신 후 다른 Topic으로 전달하기

```java
@KafkaListener(topics = "${kafka.topics.hello}", 
               groupId = "${spring.kafka.consumer.group-id}")
@SendTo("topic-2")
void listenReply(String message) {
    ...
}
```



### Custom 메시지 만들기

1. 메시지로 사용할 클래스를 만든다.

```java
class User {
}
```

2. Producer와 Consumer 설정에서 사용했던 Value Serializer를 변경한다.

```java
@Configuration
public class KafkaProducerConfig {
    @Value("${spring.kafka.bootstrap-servers}")
    private String bootstrapServers;

    @Bean
    public Map<String, Object> producerConfig() {
        Map<String, Object> props = new HashMap<>();
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, JsonSerializer.class);
        return props;
    }

    @Bean
    public ProducerFactory<String, User> producerFactory() {
        return new DefaultKafkaProducerFactory<>(producerConfig());
    }

    @Bean
    public KafkaTemplate<String, User> kafkaTemplate() {
        return new KafkaTemplate<>(producerFactory());
    }
}
```

```java
@Configuration
public class KafkaConsumerConfig {
    @Value("${spring.kafka.bootstrap-servers}")
    private String bootstrapServers;

    @Bean
    public Map<String, Object> consumerConfig() {
        Map<String, Object> props = new HashMap<>();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        return props;
    }

    @Bean
    public ConsumerFactory<String, User> consumerFactory() {
        return new DefaultKafkaConsumerFactory<>(
            consumerConfig(),
            new StringDeserializer(),
            new JsonDeserializer<>(User.class);
        );
    }

    @Bean
    public KafkaListenerContainerFactory<ConcurrentMessageListenerContainer<String, User>> userkafkaListenerContainerFactory() {
        ConcurrentKafkaListenerContainerFactory<String, User> factory =
                new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(consumerFactory());
        return factory;
    }
}
```

3. 사용하기

메시지 발행
```java
@Autowired
private KafkaTemplate<String, User> kafkaTemplate;

public void send(User user) {
    kafkaTemplate.send("topic-1", user);
}
```
메시지 소비
```java
@KafkaListener(
    topics = "topic-1",
    groupId="myGroup",
    containerFactory="userkafkaListenerContainerFactory")
public void listen(User user) {
    
}
```

