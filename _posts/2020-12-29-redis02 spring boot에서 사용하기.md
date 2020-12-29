---
layout: post
title: "[Redis] 2.Spring boot에서 사용하기"
description: "Spring boot에서 Redis 사용하기"
date: 2020-12-29 00:00:07
tags: [Redis, Spring boot]
comments: true
share: true
---

Spring boot에서 Redis를 사용하기 위해 다음 과정을 따른다.



1. Redis 의존성을 추가

```gradle
dependencies {
   implementation 'org.springframework.boot:spring-boot-starter-data-redis'
}
```

2. application.yml 수정

   redis에 접속하기 위한 정보를 입력한다.

```yml
spring:
  redis:
    host: localhost
    port: 6379
```

3. Spring boot에서 사용하기

   Spring boot에서는 RedisTemplate을 이용해 쉽게 Redis를 접근하고 사용할 수 있다.

- RedisTemplate

  1. Config

  ```java
  @Configuration
  public class RedisConfig {
      @Bean
      public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory connectionFactory) {
          RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
          redisTemplate.setConnectionFactory(connectionFactory);
          redisTemplate.setKeySerializer(new StringRedisSerializer());
          redisTemplate.setValueSerializer(new Jackson2JsonRedisSerializer<>(String.class));
  
          return redisTemplate;
      }
  }
  ```

  > 다양한 Serializer가 있는데 기본 Serializer는 JdkSerializationRedisSerializer를 사용하며 이는 키 앞에 유니코드가 생성되는 문제가 있고, GenericJackson2JsonRedisSerializer는 오브젝트를 그대로 사용하기 떄문에 확장성이 부족한 단점이 있다.

  Jackson2JsonRedisSerializer또한 많은 타입들을 작업할 경우 엔티티마다 RedisTemplate을 생성해서 가지고 있어야 한다. 그렇지 않으면 멀티 스레드 환경에서 Serializer가 변경되어 예외가 발생 될 것이다.

  

  StringSerializer를 사용해 Json으로 파싱하는 방법이 있다.

  ```java
  // Bean
  @Bean
  public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory connectionFactory) {
      RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
      redisTemplate.setConnectionFactory(connectionFactory);
      redisTemplate.setKeySerializer(new StringRedisSerializer());
      redisTemplate.setValueSerializer(new StringRedisSerializer());
  
      redisTemplate.setHashKeySerializer(new StringRedisSerializer());
      redisTemplate.setHashValueSerializer(new StringRedisSerializer());
      
      return redisTemplate;
  }
  
  // 사용
  public <T> T getData(String key, Class<T> classType) { 
      String jsonResult = (String)redisTemplate.opsForValue().get(key);
      ObjectMapper mapper = new ObjectMapper();
      T obj = mapper.readValue(jsonResult, classType);
      return obj;
  }
  ```

  

  2. RedisTemplate

  ```java
  // 키 타입 조회 (return: DataType)
  redisTemplate.type("key");
  // 키 갯수 반환 (return: long)
  redisTemplate.countExistingKeys(Arrays.asList("key1", "key2"));
  // 키 존재 확인 (return: boolean)
  redisTemplate.hasKey("key1");
  // 키 만료 날짜 셋팅 (return: boolean)
  redisTemplate.expireAt("key1", Date.from(LocalDateTime.now().plusDays(1L)
                                         .atZone(ZoneId.systemDefault()).toInstance()));
  // 키 만료 시간 셋팅 (return: boolean)
  redisTemplate.expire("key1", 60, TimeUnit.SECONDS);
  // 키 만료 시간 조회 (return: long)
  redisTemplate.getExpire("key1");
  // 키 만료 시간 해제 (return: boolean)
  redisTemplate.persist("key1");
  // 키 삭제
  redisTemplate.delete("key1");
  // 키 일괄 삭제
  redisTemplate.delete(Arrays.asList("key1", "key2"));
  ```

- ValueOpertaions (string)

  ```java
  ValueOperations<String, String> valueOps = redisTemplate.opsForValue();
  valueOps.set("key1", "value1");
  valueOps.get("key1");
  valueOps.multiGet(Arrays.asList("key1", "key2"));
  ```
  
- ListOperations (list)

  ```java
  ListOperations<String, String> listOps = redisTemplate.opsForList();
  listOps.size("key1");
  listOps.range("key1", 0, 10);
  listOps.rightPop("key1");
  listOps.leftPop("key1");
  ```

- HashOperations (hash)

  ```java
  HashOperations<String, String, String> hashOps = redisTemplate.opsForHash();
  hashOps.keys("key1");
  hashOps.size("key1");
  hashOps.put("key1", "field1", "value1");
  hashOps.get("key1", "field1");
  ```

- SetOperations (set)

  ```java
  SetOperations<String, String> setOps = redisTemplate.opsForSet();
  setOps.add("key1", "value1");
  setOps.add("key1", "value2");
  setOps.size("key1");
  setOps.members("key1");
  setOps.isMember("key1", "value1");
  ```

- ZSetOperations (sorted set)

  ```java
  ZSetOperations<String, String> zsetOps = redisTemplate.opsForZSet();
  zsetOps.add("key1", "value1");
  zsetOps.add("key1", "value2");
  zsetOps.size("key1");
  zsetOps.range("key1", 0, 10);
  zsetOps.reverseRank("key1", "value2");
  ```

  