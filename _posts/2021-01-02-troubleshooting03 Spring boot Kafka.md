---
layout: post
title: "[Trouble shooting] 3.Spring boot Kafka 오류"
description: "Spring boot에서 Kafka를 연동하면서 생긴 오류"
date: 2021-01-02 00:00:00
tags: [Trouble shooting, Kafka, Spring boot]
comments: true
share: true
---

Spring boot에서 Kafka를 연동하면서 생겼던 오류와 궁금증을 정리

### 사용 환경

- Spring boot 2.4.1
- OpenJDK 11
- bitnami/kafka:2-debian-10
- bitnami/zookeeper:3-debian-10



### 예제 오류

인터넷에서 예제를 보고 카프카를 사용하는데 아래와 같은 오류 로그가 계속 출력되었다.
```bash
[AdminClient clientId=adminclient-1] Error connecting to node c5e1603cf7db:9092 (id: 1001 rack: null)
java.net.UnknownHostException: c5e1603cf7db
	at java.base/java.net.InetAddress$CachedAddresses.get(InetAddress.java:797) ~[na:na]
	at java.base/java.net.InetAddress.getAllByName0(InetAddress.java:1505) ~[na:na]
	at java.base/java.net.InetAddress.getAllByName(InetAddress.java:1364) ~[na:na]
	at java.base/java.net.InetAddress.getAllByName(InetAddress.java:1298) ~[na:na]
	at org.apache.kafka.clients.ClientUtils.resolve(ClientUtils.java:110) ~[kafka-clients-2.6.0.jar:na]
	at org.apache.kafka.clients.ClusterConnectionStates$NodeConnectionState.currentAddress(ClusterConnectionStates.java:403) ~[kafka-clients-2.6.0.jar:na]
	at org.apache.kafka.clients.ClusterConnectionStates$NodeConnectionState.access$200(ClusterConnectionStates.java:363) ~[kafka-clients-2.6.0.jar:na]
	at org.apache.kafka.clients.ClusterConnectionStates.currentAddress(ClusterConnectionStates.java:151) ~[kafka-clients-2.6.0.jar:na]
	at org.apache.kafka.clients.NetworkClient.initiateConnect(NetworkClient.java:958) ~[kafka-clients-2.6.0.jar:na]
	at org.apache.kafka.clients.NetworkClient.ready(NetworkClient.java:294) ~[kafka-clients-2.6.0.jar:na]
	at org.apache.kafka.clients.admin.KafkaAdminClient$AdminClientRunnable.sendEligibleCalls(KafkaAdminClient.java:1039) ~[kafka-clients-2.6.0.jar:na]
	at org.apache.kafka.clients.admin.KafkaAdminClient$AdminClientRunnable.processRequests(KafkaAdminClient.java:1281) ~[kafka-clients-2.6.0.jar:na]
	at org.apache.kafka.clients.admin.KafkaAdminClient$AdminClientRunnable.run(KafkaAdminClient.java:1224) ~[kafka-clients-2.6.0.jar:na]
	at java.base/java.lang.Thread.run(Thread.java:834) ~[na:na]
```

localhost에서 접속하려면 kafka에 환경변수 2개를 설정해주라고 한다.

```yml
KAFKA_CFG_LISTENERS=PLAINTEXT://:9092
KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://localhost:9092
```



### Kafka를 사용하기 위해 @EnableKafka는 필수인가?

Spring boot에서 메시지를 소비하기 위해 @KafkaListener 어노테이션을 사용한다.

@KafkaListener 어노테이션을 사용하기 위해서는 Consumer Configuration class에 @EnalbeKafka 어노테이션을 추가해야 한다. 

확인을 위해 @EnableKafka 어노테이션을 제거 해봤는데 정상적으로 동작이 되었다.

이유를 찾아보니 Spring boot는 auto configuration을 제공하기 때문에 class path에서 spring-kafka를 찾으면 @EnableKafka를 자동으로 추가 해준다.

이를 직접 테스트해보려면 **@SpringBootApplication(exclude={KafkaAutoConfiguration.class})**를 추가하면 된다.
