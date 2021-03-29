---
layout: post
title: "[spring boot] Application Event"
description: "spring boot event 사용해보기"
date: 2021-03-29 00:00:00
tags: [spring boot]
comments: true
share: true
---

# Application Event

Spring에서는 Application Context에서 이벤트를 발생시키고 구독하는 기능을 제공한다. 이를 사용하면 도메인간의 강한 의존성을 느슨하게 만들 수 있다.

Event를 사용하기 위해서는 3가지가 필수적이다.



## 1. 이벤트

전달하기 위한 이벤트로 ApplicationEvent를 상속받는다.

> Spring 4.2 부터는 일반 POJO로 가능함

```java
@Getter
public class OrderEvent extends ApplicationEvent {
    private long orderId;
    
    public OrderEvent(Object source, long orderId) {
        super(source);
        this.orderId = orderId;
    }
}
```

## 2. 이벤트 발행자

이벤트를 발행하는 대상. 실제 이벤트가 발행되는 것은 Application Context에서 이루어진다.

### 이벤트를 발행하는 방법

- ApplicationEventPublish를 주입 받는 방법

```java
@RequiredArgsConstructor
@Service
public class OrderService {
    private final ApplicationEventPublisher eventPublisher;

    public void placeOrder(OrderRequest orderRequest) {
        eventPublisher.publishEvent(new OrderEvent(this, orderRequest.getOrderId()));
    }
}
```
> ApplicationEventPublisherAware인터페이스를 구현해도 됨



- ApplicationContext를 주입받아 사용해도 된다.

> ApplicationContext는 ApplicationEventPublisher를 구현한다.



## 3. 이벤트 수신자

이벤트를 수신하는 대상. 

### 이벤트를 수신하는 방법

- ApplicationListener 인터페이스를 구현하는 방법

```java
@Component
public class OrderEventListener implements ApplicationListener<OrderEvent> {
	@Override
    public void onApplicationEvent(OrderEvent event) {
        System.out.println("Received" + event.getOrderId());
    }
}
```

- 어노테이션을 이용하는 방법

```java
@Component
public class OrderEventListener {
	@EventListener
    public void receivedOrderEvent(OrderEvent event) {
        System.out.println("Received" + event.getOrderId());
    }
}
```



### 순서를 지정하는 방법

이벤트 핸들러는 하나의 thread에서 순차적으로 실행되기 때문에 이벤트에 대한 핸들러가 2개 이상인 경우 @Order 어노테이션을 사용해 순서를 지정해준다.

```java
@Order(1)
@EventListener
public void receivedOrderEvent(OrderEvent event) {
    System.out.println("Received" + event.getOrderId());
}
```

> 숫자가 클 수록 우선순위가 낮다.



### 비동기로 이벤트를 핸들링 하는 방법

이벤트 핸들러는 기본적으로 SpringApplication이 실행될 때 동작하는 리스너에서 동기적으로 실행되고, 이를 비동기적으로 처리하기 위해서는 @Async  어노테이션을 사용한다.

```java
@Component
public class OrderEventListener {
    @Async
    @EventListener
    public void receivedOrderEvent(OrderEvent event) {
        System.out.println("Received" + event.getOrderId());
    }
}
```

> @EnableAsync 어노테이션을 추가해줘야 한다.



## @EventListener와 @TransactionEventListener의 차이

EventListener는 이벤트를 발행하는 곳과 같은 트랜잭션에서 동작하기 때문에 RuntimeException이 발생하면 롤백이 된다. 하지만 TransactionEventListener을 이용하면 트랜잭션이 커밋 되기 전/후 로 실행 순서를 제어할 수 있다.

> 예를 들면, 주문이 접수 되고 이벤트를 발행해 메일을 발송하는 로직이 있다면, 메일 발송이 실패할 경우 주문이 취소 되면 안된다.

```java
@Component
public class OrderEventListener {
    @Async
    @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT, fallbackExecution = true)
    public void receivedOrderEvent(OrderEvent event) {
        System.out.println("Received" + event.getOrderId());
    }
}
```

> phase : 트랜잭션 커밋 전/후 로 실행 순서 제어
>
> - BEFORE_COMMIT
> - AFTER_COMMIT //기본 동작
> - AFTER_ROLLBACK
> - AFTER_COMPLETION
>
> fallbackExecution : 현재 진행중인 트랜잭션이 없을 때도 실행 할 것인지
>

