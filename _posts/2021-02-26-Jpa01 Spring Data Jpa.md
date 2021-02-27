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

Java Persistence API. 자바 ORM 기술 표준.

자바 어플리케이션에서 관계형 데이터베이스를 사용하는 방식을 정의한 인터페이스

SQL 중심적 개발에서 객체를 중심으로 개발 할 수 있다.



객체 지향과 관계형 데이터베이스의 패러다임이 불일치하면서 발생하는 문제를 해결 하고 값을 DB에 저장하는 것을 컬렉션에 저장하듯이 객체를 저장할 수 있다.



## 사용법

- 저장: jpa.persist(obj);
- 조회: T obj = jpa.find(id);
- 수정: obj.setName("new name");
- 삭제: jpa.remove(obj);



## 최적화

1차 캐시와 동일성 보장

> 같은 트랜잭션 안에서는 같은 엔티티를 반환
>
> DB Isolation Level이 Read Commit이어도 어플리케이션에서 Repeatable Read 보장

트랜잭션을 지원하는 쓰기 지연

지연 로딩



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



