---
layout: post
title: "[Jpa] Spring Data Jpa"
description: "Spring Data Jpa 사용하기"
date: 2021-02-26 00:00:00
tags: [spring boot, jpa]
comments: true
share: true
---



# JPA

Java Persistence Api. 자바 어플리케이션에서 관계형 데이터베이스를 사용하는 방식을 정의한 인터페이스

# Hibernate

JPA 구현체

# Spring data JPA

Spring에서 제공하는 JPA를 쓰기 쉽게 만들어 둔 모듈



EntityManager를 직접 다루지 않고 Repository인터페이스를 사용하면 미리 정해진 규칙대로 메서드를 입력하면, 해당 메서드 이름에 적합한 쿼리를 날리는 구현체를 만들어 Bean에 등록 됨



Spring Data JPA의 Repository 구현체인 SimpleJpaRepository 클래스는 내부적으로 EntityManager를 사용하고 있다.



![jpa](https://zkdlu.github.io/images/jpa/jpa.png)





데이터 소스 설정

```properties
# 기본 설정 (h2, hsql의 메모리 db는 아무 설정 안해도 됨)
spring.datasource.url=jdbc:[db]://[host]:[db-name]
spring.datasource.username=zkdlu
spring.datasource.password=1234

# 생성되는 sql 확인용
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true

# ? 로 출력되는거 로깅으로 확인하기
logging.level.org.hibernate.SQL=debug
logging.level.org.hibernate.type.descriptor.sql=trace
```



