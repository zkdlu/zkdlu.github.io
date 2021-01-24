---
layout: post
title: "[Redis] 4.Spring boot에서  pub/sub 모델 사용하기"
description: "Spring boot에서 Redis 사용하기"
date: 2020-12-29 00:00:09
tags: [Redis, Spring boot]
comments: true
share: true
---

Spring boot에서 Redis를 사용해 Pub/Sub 모델을 사용한다.



## Pub/Sub

특정한 주제(topic)에 대하여 해당 topic을 구독한 대상에게 메시지를 발행하는 통신 방법

Redis는 Kafka나 RabbitMQ같은 메시지 큐와 같이 고도화 된 기능은 제공하지 않지만 가볍고 빠른 기능을 제공한다.




1. Redis 의존성을 추가

```gradle
dependencies {
   implementation 'org.springframework.boot:spring-boot-starter-data-redis'
}
```

2. Redis Config 추가

   pub/sub은 항상 redis에 발행 된 데이터가 있는지 확인하고 있어야 하기 떄문에 Listener를 등록하여야 한다.

```java
@Configuration
public class RedisConfig {
    @Bean
    public RedisMessageListenerContainer redisMessageListener(
            RedisConnectionFactory connectionFactory) {
        RedisMessageListenerContainer container = new RedisMessageListenerContainer();
        container.setConnectionFactory(connectionFactory);

        return container;
    }

    @Bean
    public RedisTemplate<String, Object> redisTemplate(
            RedisConnectionFactory connectionFactory) {
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(connectionFactory);
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setValueSerializer(new Jackson2JsonRedisSerializer<>(String.class));

        return redisTemplate;
    }
}
```

3. 발행자(Publisher) 추가

```java
@Service
public class RedisPublisher {
    private final RedisTemplate<String, Object> redisTemplate;
    
    public void publish(ChannelTopic topic, String message) {
        redisTemplate.convertAndSend(topic.getTopic(), message);
    }
}
```

4. 구독자 (Subscriber) 추가

```java
@Service
public class RedisSubscriber implements MessageListener {
    private final ObjectMapper objectMapper;
    private final RedisTemplate redisTemplate;
    
    @Override
    public void onMessage(Message message, byte[] pattern) {
        try {
            String message = (String) redisTemplate.getStringSerializer()
                .deserialize(message.getBody());
            
            String data = objectMapper.readValue(message, String.class);
            System.out.println(data);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

5. Topic 생성 및 Message를 Listening하기 위한 Controller 생성

```java
@RequestMapping("/pubsub")
@RestController
public class PubSubController {
    private final RedisMessageListenerContainer redisMessageListener;
    private final RedisPublisher redisPublisher;
    private final RedisSubscriber redisSubscriber;
    private Map<String, ChannelTopic> channels;
    
    @PostConstruct
    public void init() {
        channels = new HashMap<>();
    }
    // 토픽 목록
    @GetMapping("/topics")
    public Set<String> getTopicAll() {
        return channels.keySet();
    }
    // 토픽 생성
    @PutMapping("/topics/{name}")
    public void createTopic(@PathVariable String name) {
        ChannelTopic channel = new ChannelTopic(name);
        redisMessageListener.addMessageListener(redisSubscriber, channel);
        channels.put(name, channel);
    }
    // 메시지 발행
    @PostMapping("/topics/{name}")
    public void pushMessage(@PathVariable String name, @RequestParam String message) {
        ChannelTopic channel = channels.get(name);
        redisPublisher.publish(channel, message);
    }
    // 토픽 제거
    @DeleteMapping("/topics/{name}")
    public void deleteTopic(@PathVariable String name) {
        ChannelTopic channel = channels.get(name);
        redisMessageListener.removeMessageListener(redisSubscriber, channel);
        channels.remove(name);
    }
}
```

