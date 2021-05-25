---
layout: post
title: "[Trouble shooting] 7.Jpa Table 이름"
description: "JPA를 사용하면서 발생한 예외"
date: 2021-05-25 00:00:00
tags: [trouble shooting, spring boot, jpa]
comments: true
share: true
---

### 환경

- OpenJDK 11
- Spring Boot 2.3.8
- org.springframework.boot:spring-boot-starter-data-jpa
- com.h2database:






### 무엇을 했는가?

### Order 엔티티

```java
@Entity
@Getter
@NoArgsConstructor
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private String id;
    @OneToMany
    @JoinColumn(name = "ORDER_ID")
    private List<OrderItem> orderItems;

    @Builder
    public Order(String id, List<OrderItem> orderItems) {
        this.id = id;
        this.orderItems = orderItems;
    }
}

```



### 어떤일이 있었는가?


```bash
org.h2.jdbc.JdbcSQLSyntaxErrorException: Syntax error in SQL statement "SELECT ORDER0_.ID AS ID1_0_0_ FROM ORDER[*] ORDER0_ WHERE ORDER0_.ID=?"; expected "identifier"; SQL statement:
select order0_.id as id1_0_0_ from order order0_ where order0_.id=? [42001-200]
```

> 발생한 예외 메시지



### 어떻게 하였는가?

Order는 SQL 예약어라서 사용이 불가능하다. @Table 어노테이션으로 테이블 명을 명시해줌.