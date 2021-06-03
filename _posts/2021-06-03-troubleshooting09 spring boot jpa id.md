---
layout: post
title: "[Trouble shooting] 8.@GeneratedValue"
description: "JPA를 사용하면서 발생한 예외"
date: 2021-06-03 00:00:00
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
@Getter
@NoArgsConstructor
@Entity
@Table(name = "ORDERS")
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private String id;
    @OneToMany(cascade = CascadeType.ALL)
    @JoinColumn(name = "ORDER_ID")
    private List<OrderItem> orderItems;
}
```

### Payment 엔티티

```java
@Getter
@NoArgsConstructor
@Entity
@Table(name = "PAYMENTS")
public class Payment {
    public enum State {
        PREPARE, PAYED, COMPLETE
    }

    @Id
    private String id;
    @Enumerated(value = EnumType.ORDINAL)
    private State state;
    @OneToOne
    @JoinColumn(name = "ORDER_ID")
    private Order order;
}
```



### Order Service

```java
@Transactional
public PayRequest placeOrder(List<OrderItem> orderItems) {
    log.info("order service: {}", Thread.currentThread().getId());

    Order order = Order.builder()
        .id(UUID.randomUUID().toString())
        .orderItems(orderItems)
        .build();

    orderRepository.save(order);

    PayReady payReady = kakaoPay.prepare(order);

    return PayRequest.builder()
        .payReady(payReady)
        .order(order)
        .build();
}
```

### Payment Service

```java
@Transactional(propagation = Propagation.MANDATORY)
public PayReady prepare(Order order) {
    log.info("pay: {}", Thread.currentThread().getId());

    Payment payment = Payment.builder()
        .id(UUID.randomUUID().toString())
        .order(order)
        .build();

    paymentRepository.save(payment);

    return prepareKakaoPay(payment);
}
```



### 어떤일이 있었는가?


```bash
javax.persistence.EntityNotFoundException: Unable to find com.zkdlu.order.domain.Order with id 858e56e2-48ca-4baf-8c8d-77ad5b30a5d0
	at org.hibernate.jpa.boot.internal.EntityManagerFactoryBuilderImpl$JpaEntityNotFoundDelegate.handleEntityNotFound(EntityManagerFactoryBuilderImpl.java:163) ~[hibernate-core-5.4.28.Final.jar:5.4.28.Final]
	at org.hibernate.event.internal.DefaultLoadEventListener.load(DefaultLoadEventListener.java:216) ~[hibernate-core-5.4.28.Final.jar:5.4.28.Final]
	at org.hibernate.event.internal.DefaultLoadEventListener.proxyOrLoad(DefaultLoadEventListener.java:332) ~[hibernate-core-5.4.28.Final.jar:5.4.28.Final]
	at org.hibernate.event.internal.DefaultLoadEventListener.doOnLoad(DefaultLoadEventListener.java:108) ~[hibernate-core-5.4.28.Final.jar:5.4.28.Final]
	at org.hibernate.event.internal.DefaultLoadEventListener.onLoad(DefaultLoadEventListener.java:74) ~[hibernate-core-5.4.28.Final.jar:5.4.28.Final]
	at org.hibernate.event.service.internal.EventListenerGroupImpl.fireEventOnEachListener(EventListenerGroupImpl.java:104) ~[hibernate-core-5.4.28.Final.jar:5.4.28.Final]
	at org.hibernate.internal.SessionImpl.fireLoadNoChecks(SessionImpl.java:1186) ~[hibernate-core-5.4.28.Final.jar:5.4.28.Final]
	at org.hibernate.internal.SessionImpl.internalLoad(SessionImpl.java:1051) ~[hibernate-core-5.4.28.Final.jar:5.4.28.Final]
	at org.hibernate.type.EntityType.resolveIdentifier(EntityType.java:697) ~[hibernate-core-5.4.28.Final.jar:5.4.28.Final]
	at org.hibernate.type.EntityType.resolve(EntityType.java:464) ~[hibernate-core-5.4.28.Final.jar:5.4.28.Final]
	at org.hibernate.type.ManyToOneType.resolve(ManyToOneType.java:240) ~[hibernate-core-5.4.28.Final.jar:5.4.28.Final]
	at org.hibernate.type.EntityType.resolve(EntityType.java:457) ~[hibernate-core-5.4.28.Final.jar:5.4.28.Final]
	at org.hibernate.type.EntityType.replace(EntityType.java:358) ~[hibernate-core-5.4.28.Final.jar:5.4.28.Final]
	at org.hibernate.type.AbstractType.replace(AbstractType.java:164) ~[hibernate-core-5.4.28.Final.jar:5.4.28.Final]
	at org.hibernate.type.TypeHelper.replace(TypeHelper.java:204) ~[hibernate-core-5.4.28.Final.jar:5.4.28.Final]
	at org.hibernate.event.internal.DefaultMergeEventListener.copyValues(DefaultMergeEventListener.java:488) ~[hibernate-core-5.4.28.Final.jar:5.4.28.Final]
	at org.hibernate.event.internal.DefaultMergeEventListener.entityIsTransient(DefaultMergeEventListener.java:241) ~[hibernate-core-5.4.28.Final.jar:5.4.28.Final]
	at org.hibernate.event.internal.DefaultMergeEventListener.entityIsDetached(DefaultMergeEventListener.java:318) ~[hibernate-core-5.4.28.Final.jar:5.4.28.Final]
	at org.hibernate.event.internal.DefaultMergeEventListener.onMerge(DefaultMergeEventListener.java:172) ~[hibernate-core-5.4.28.Final.jar:5.4.28.Final]
	at org.hibernate.event.internal.DefaultMergeEventListener.onMerge(DefaultMergeEventListener.java:70) ~[hibernate-core-5.4.28.Final.jar:5.4.28.Final]
	at org.hibernate.event.service.internal.EventListenerGroupImpl.fireEventOnEachListener(EventListenerGroupImpl.java:93) ~[hibernate-core-5.4.28.Final.jar:5.4.28.Final]
	at org.hibernate.internal.SessionImpl.fireMerge(SessionImpl.java:793) ~[hibernate-core-5.4.28.Final.jar:5.4.28.Final]
	at org.hibernate.internal.SessionImpl.merge(SessionImpl.java:780) ~[hibernate-core-5.4.28.Final.jar:5.4.28.Final]
	at jdk.internal.reflect.GeneratedMethodAccessor45.invoke(Unknown Source) ~[na:na]
	at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:na]
	at java.base/java.lang.reflect.Method.invoke(Method.java:566) ~[na:na]
	at org.springframework.orm.jpa.ExtendedEntityManagerCreator$ExtendedEntityManagerInvocationHandler.invoke(ExtendedEntityManagerCreator.java:366) ~[spring-orm-5.2.13.RELEASE.jar:5.2.13.RELEASE]
	at com.sun.proxy.$Proxy76.merge(Unknown Source) ~[na:na]
	at jdk.internal.reflect.GeneratedMethodAccessor45.invoke(Unknown Source) ~[na:na]
	at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:na]
	at java.base/java.lang.reflect.Method.invoke(Method.java:566) ~[na:na]
	at org.springframework.orm.jpa.SharedEntityManagerCreator$SharedEntityManagerInvocationHandler.invoke(SharedEntityManagerCreator.java:314) ~[spring-orm-5.2.13.RELEASE.jar:5.2.13.RELEASE]
	at com.sun.proxy.$Proxy76.merge(Unknown Source) ~[na:na]
	at org.springframework.data.jpa.repository.support.SimpleJpaRepository.save(SimpleJpaRepository.java:557) ~[spring-data-jpa-2.3.7.RELEASE.jar:2.3.7.RELEASE]
	at jdk.internal.reflect.GeneratedMethodAccessor43.invoke(Unknown Source) ~[na:na]
	at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:na]
	at java.base/java.lang.reflect.Method.invoke(Method.java:566) ~[na:na]
	at org.springframework.data.repository.core.support.ImplementationInvocationMetadata.invoke(ImplementationInvocationMetadata.java:72) ~[spring-data-commons-2.3.7.RELEASE.jar:2.3.7.RELEASE]
	at org.springframework.data.repository.core.support.RepositoryComposition$RepositoryFragments.invoke(RepositoryComposition.java:382) ~[spring-data-commons-2.3.7.RELEASE.jar:2.3.7.RELEASE]
	at org.springframework.data.repository.core.support.RepositoryComposition.invoke(RepositoryComposition.java:205) ~[spring-data-commons-2.3.7.RELEASE.jar:2.3.7.RELEASE]
	at org.springframework.data.repository.core.support.RepositoryFactorySupport$ImplementationMethodExecutionInterceptor.invoke(RepositoryFactorySupport.java:550) ~[spring-data-commons-2.3.7.RELEASE.jar:2.3.7.RELEASE]
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186) ~[spring-aop-5.2.13.RELEASE.jar:5.2.13.RELEASE]
	at org.springframework.data.repository.core.support.QueryExecutorMethodInterceptor.doInvoke(QueryExecutorMethodInterceptor.java:155) ~[spring-data-commons-2.3.7.RELEASE.jar:2.3.7.RELEASE]
	at org.springframework.data.repository.core.support.QueryExecutorMethodInterceptor.invoke(QueryExecutorMethodInterceptor.java:130) ~[spring-data-commons-2.3.7.RELEASE.jar:2.3.7.RELEASE]
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186) ~[spring-aop-5.2.13.RELEASE.jar:5.2.13.RELEASE]
	at org.springframework.data.projection.DefaultMethodInvokingMethodInterceptor.invoke(DefaultMethodInvokingMethodInterceptor.java:80) ~[spring-data-commons-2.3.7.RELEASE.jar:2.3.7.RELEASE]
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186) ~[spring-aop-5.2.13.RELEASE.jar:5.2.13.RELEASE]
	at org.springframework.transaction.interceptor.TransactionAspectSupport.invokeWithinTransaction(TransactionAspectSupport.java:367) ~[spring-tx-5.2.13.RELEASE.jar:5.2.13.RELEASE]
	at org.springframework.transaction.interceptor.TransactionInterceptor.invoke(TransactionInterceptor.java:118) ~[spring-tx-5.2.13.RELEASE.jar:5.2.13.RELEASE]
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186) ~[spring-aop-5.2.13.RELEASE.jar:5.2.13.RELEASE]
	at org.springframework.dao.support.PersistenceExceptionTranslationInterceptor.invoke(PersistenceExceptionTranslationInterceptor.java:139) ~[spring-tx-5.2.13.RELEASE.jar:5.2.13.RELEASE]
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186) ~[spring-aop-5.2.13.RELEASE.jar:5.2.13.RELEASE]
	at org.springframework.data.jpa.repository.support.CrudMethodMetadataPostProcessor$CrudMethodMetadataPopulatingMethodInterceptor.invoke(CrudMethodMetadataPostProcessor.java:178) ~[spring-data-jpa-2.3.7.RELEASE.jar:2.3.7.RELEASE]
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186) ~[spring-aop-5.2.13.RELEASE.jar:5.2.13.RELEASE]
	at org.springframework.aop.interceptor.ExposeInvocationInterceptor.invoke(ExposeInvocationInterceptor.java:95) ~[spring-aop-5.2.13.RELEASE.jar:5.2.13.RELEASE]
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186) ~[spring-aop-5.2.13.RELEASE.jar:5.2.13.RELEASE]
	at org.springframework.aop.framework.JdkDynamicAopProxy.invoke(JdkDynamicAopProxy.java:212) ~[spring-aop-5.2.13.RELEASE.jar:5.2.13.RELEASE]
	at com.sun.proxy.$Proxy85.save(Unknown Source) ~[na:na]
	at com.zkdlu.payment.service.KakaoPay.prepare(KakaoPay.java:48) ~[main/:na]
```

> 발생한 예외 메시지



### 어떻게 하였는가?

Order 엔티티를 영속화 하고 Payment를 영속화 했는데, Payment를 영속화 하는 과정에서 Order 엔티티를 찾을 수 없다고 예외가 발생했다.

트랜잭션 전파 문제인가 싶어 여러 설정을 해보았는데도 마찬가지였고, cascade 옵션을 줘봤는데 update 쿼리가 나가면서 Order 엔티티가 2개가 생겼다.



아.. Id를 지정해줬는데 @GeneratedValue를 사용해버렸다.

**@GeneratedValue를 제거해준다.**



- IDENTITY : 데이터베이스에 위임(MYSQL)
  - Auto_Increment 
- SEQUENCE : 데이터베이스 시퀀스 오브젝트 사용(ORACLE)
  - @SequenceGenerator 필요

- TABLE : 키 생성용 테이블 사용, 모든 DB에서 사용
  - @TableGenerator 필요
- AUTO : 방언에 따라 자동 지정, 기본값