---
layout: post
title: "[Redis] 3.Spring boot에서 Cache로 사용하기"
description: "Spring boot에서 Redis 사용하기"
date: 2020-12-29 00:00:08
tags: [Redis, Spring boot, Cache]
comments: true
share: true
---

Spring boot에서 Redis를 Cache로 사용하기 위해 다음 과정을 따른다.



## Redis Cache

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
  cache:
    type: redis
```

3. CacheKey 추가

```java
public class CacheKey {
    private CacheKey() {}
    public static final int DEFAULT_EXPIRE_SEC = 60;
    public static final String USER = "user";
    public static final int USER_EXPIRE_SEC = 120;
}
```

4. RedisCacheConfig 추가

   @EnableCaching 어노에티션으로 Cache를 활성화 하고 캐시 정책을 변경한다.

```java
@EnableCaching
@Configuration
public class RedisCacheConfig {
    @Bean
    public RedisCacheManager redisCacheManager(RedisConnectionFactory connectionFactory) {
        RedisCacheConfiguration configuration = RedisCacheConfiguration.defaultCacheConfig()
                .disableCachingNullValues()
                .entryTtl(Duration.ofSeconds(CacheKey.DEFAULT_EXPIRE_SEC))
                .computePrefixWith(CacheKeyPrefix.simple())
                .serializeKeysWith(RedisSerializationContext
                                   .SerializationPair
                                   .fromSerializer(new StringRedisSerializer()));

        Map<String, RedisCacheConfiguration> cacheConfigurations = new HashMap<>();
        cacheConfigurations.put(CacheKey.USER, 
                                RedisCacheConfiguration.defaultCacheConfig()
                               .entryTtl(Duration.ofSeconds(CacheKey.USER_EXPIRE_SEC)));
        
        return RedisCacheManager.RedisCacheManagerBuilder
            .fromConnectionFactory(connectionFactory)
            .cacheDefaults(configuration)
            .withInitialCacheConfigurations(cacheConfigurations).build();
    }   
}
```

5. 어노테이션 추가

- @Cacheable

Redis에 캐싱된 데이터가 있으면 반환하고, 없으면 기존 로직을 수행후 Redis에 캐시한다.

고정된 value와 유동적인 key 를 조합해 캐시를 조회한다.

```java
@Cacheable(value = CacheKey.USER, key = "#id", unless = "#result == null")
@GetMapping("/{id}")
public String getUser(@PathVariable long id) {
    return userService.get(id);
}
```

- @CachePut

Redis에 저장된 캐시정보를 갱신한다.

```java
@CachePut(value = CacheKey.USER, key = "#id")
@PutMapping("/{id}")
public String putUser(@PathVariable long id) {
    return userService.put(id);
}
```

- @CacheEvict

Redis에 저장된 캐시 정보를 삭제

```java
@CacheEvict(value = CacheKey.USER, key = "#id")
@DeleteMapping("/{id}")
public String deleteUser(@PathVariable long id) {
    return userService.delete(id);
}
```



### RedisHash

redis를 jpa 처럼 쓸 수 있게 해주는 어노테이션

jpa의 @Entity 대신 @RedisHash("키 값")으로 생성 한 후 

```java
@Getter @Setter
@RedisHash("user")
public class User
```

repository 인터페이스에  JpaRepository 대신 CrudRepository를 상속 시킨다.

```java
public interface UserRepo extends CrudRepository<User, Long> {
```

