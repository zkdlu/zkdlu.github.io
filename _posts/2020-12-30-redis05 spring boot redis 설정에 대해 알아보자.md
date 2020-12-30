---
layout: post
title: "[Redis] 5.Spring boot redis cache 설정"
description: "이전에 따라한 cache 설정"
date: 2020-12-30 00:00:00
tags: [Redis, Spring boot, Cache]
comments: true
share: true
---



### RedisCacheManager, RedisCacheConfiguration

캐쉬에 대한 설정을 유지하는 클래스. 실제 데이터에 대한 처리는 RedisCache가 담당

```java
@EnableCaching
@Configuration
public class RedisCacheConfig {
    @Bean
    public RedisCacheManager redisCacheManager(RedisConnectionFactory connectionFactory) {
        RedisCacheConfiguration configuration = RedisCacheConfiguration
            .defaultCacheConfig()
            .disableCachingNullValues()
            .entryTtl(Duration.ofSeconds(10))
            .computePrefixWith(CacheKeyPrefix.simple())
            .serializeKeysWith(RedisSerializationContext.SerializationPair
                               .fromSerializer(new StringRedisSerializer()));

        Map<String, RedisCacheConfiguration> cacheConfigurations = new HashMap<>();
        cacheConfigurations.put("val", RedisCacheConfiguration.defaultCacheConfig()
                                .entryTtl(Duration.ofSeconds(120)));

        return RedisCacheManager.RedisCacheManagerBuilder
            .fromConnectionFactory(connectionFactory)
            .cacheDefaults(configuration)
            .withInitialCacheConfigurations(cacheConfigurations)
            .build();
    }
}
```

RedisTemplate을 인자로 받아 인스턴스를 생성할 수 있지만 builder패턴을 이용해 RedisCacheConfiguration 인스턴스를 설정값으로 넘겨 줄 수 있다.

> - disableCachingNullValues() : null 값이 캐싱 되지 않도록 한다.
>
> - entryTtl() : 캐시 유효 기간
>
> - computePrefixWith(CacheKeyPrefix.simple()) : 캐시의 prefix를 설정 하며 기본 설정의 경우 {캐시 명}:{키 값}으로 서버에 저장하게 됩니다.
>
> - withInitialCacheConfigurations : 키 값에 따라 서로 다른 RedisCacheConfiguration을 사용 할 수 있다.
>
>   > 캐시를 사용 하는 곳에서 이렇게 사용하는데
>   >
>   > @Cacheable(value = "이 키 값" key = "#id", unless = "#result == null")
>   >
>   > value : key 의 형태로 서버에 저장한다.


