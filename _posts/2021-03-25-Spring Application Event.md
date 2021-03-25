---
layout: post
title: "[spring boot] Application Event"
description: "spring boot event 사용해보기"
date: 2021-03-25 00:00:00
tags: [spring boot]
comments: true
share: true
---

# Application Event

Spring에서는 Application Context에서 이벤트를 발생시키고 구독하는 기능을 제공한다.

Event를 사용하기 위해서는 3가지가 필수적이다.

1. 이벤트 발행자

   ​	이벤트를 발행하는 대상. 실제로 이벤트가 발행되는 것은 Application Context에서 이루어진다.

2. 이벤트 수신자

   ​	이벤트를 수신하는 대상. 어노테이션을 이용하는 방법과 ApplicationListener인터페이스를 구현하는 방법이 있다.

3. 이벤트

   ​	전달하기 위한 이벤트로 ApplicationEvent 상속받는다.






어노테이션

- @EntityListeners

    @Entity에 사용되며, DB에 반영되기 전/후 에 실행 할 수 있는 핸들러이다.

    > @PreUpdate, @PostUpdate
    >
    > @PrePersist, @PostPersist 등

    

- @EventListener

    비즈니스 로직에서 이벤트를 발행할 때 사용

    > 핸들링도 같은 트랜잭션 안에서 동작하기 때문에 함께 롤백된다.
    >
    > ex) 메일 발송이 실패되면 다른 로직도 롤백 되버림

    

- @TransactionEventListener

    트랜잭션에 따른 처리가 가능한 EventListener

    > phase : 트랜잭션 커밋 전/후 로 실행 순서 제어
    >
    > fallbackExecution : 트랜잭션 성공 실패  무관하게 실행 할 것인지
    >
    > condtion : 이벤트 리스너 동작 조건

