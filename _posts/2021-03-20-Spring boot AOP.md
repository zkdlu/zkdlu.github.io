---
layout: post
title: "[spring boot] AOP"
description: "Spring boot에서 AOP 사용해보기"
date: 2021-03-20 00:00:00
tags: [spring boot]
comments: true
share: true
---

# AOP (Aspect Oriented Programming)

OOP를 보완하는 수단으로, 흩어진 관심사(Aspect)를 모듈화 하여 비즈니스 로직에서 분리해준다.

성능검사, 트랜잭션 처리, 예외 변환, 로깅 등  여러 곳에서 사용되는 공통 기능을 Aspect로 모듈화한다.



![aop](https://zkdlu.github.io/images/spring/aop.png)



## AOP 주요 개념

### Aspect

모듈화 된 관심사

### Target

Aspect를 적용 하는 곳

### Advice

실질적인 기능을 담은 구현체

### Joint Point

Advice가 적용될 위치, 

### Point cut

Joint Point의 상세 스펙을 정의


## Spring AOP

프록시 패턴을 기반으로 된 구현 체. 

aop 의존성 추가

```gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-aop'
}
```

