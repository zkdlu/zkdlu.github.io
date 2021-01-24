---
layout: post
title: "[kafka] 5.Kafka sasl"
description: "Kafka에 sasl 적용하기"
date: 2021-01-18 00:00:00
tags: [kafka, docker]
comments: true
share: true
---

# Kafka SASL
Simple Authentication and Security Layer

카프카를 사용하기 위해 간단한 계정 인증을 해보자.

## 1. Broker 설정 파일 만들기

kafka_server.jaas.conf

```conf
KafkaServer {
  org.apache.kafka.common.security.plain.PlainLoginModule required
  username="admin"
  password="admin-secret"
  user_admin="admin-secret";
};

KafkaClient {
  org.apache.kafka.common.security.plain.PlainLoginModule required
  username="admin"
  password="admin-secret";
};
```

## 2. Producer 설정 파일

producer_jaas.conf

```conf
bootstrap.servers=SASL_PLAINTEXT://localhost:9092
security.protocol=SASL_PLAINTEXT
sasl.mechanism=PLAIN
 
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="admin" password="admin-secret";
```

## 3. Consumer 설정 파일

consumer_jaas.conf

```conf
bootstrap.servers=SASL_PLAINTEXT://localhost:9092
security.protocol=SASL_PLAINTEXT
sasl.mechanism=PLAIN
 
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="admin" password="admin-secret";
```

## 4. docker-compose 작성

docker-compose.yml

```yaml
version: '3.5'
services:
  zookeeper:
    image: wurstmeister/zookeeper
    container_name: zookeeper-ssl
    ports:
      - "2181:2181"
  kafka:
    image: wurstmeister/kafka
    container_name: kafka-ssl
    depends_on:
      - zookeeper
    environment:
      KAFKA_CREATE_TOPICS: "test:1:1"
      KAFKA_LISTENERS: SASL_PLAINTEXT://:9092
      KAFKA_ADVERTISED_LISTENERS: SASL_PLAINTEXT://localhost:9092
      KAFKA_ZOOKEEPER_CONNECT: zookeeper-ssl:2181
      KAFKA_OPTS: "-Djava.security.auth.login.config=/etc/kafka/kafka_server_jaas.conf"
      KAFKA_INTER_BROKER_LISTENER_NAME: SASL_PLAINTEXT
      KAFKA_SASL_ENABLED_MECHANISMS: PLAIN
      KAFKA_SASL_MECHANISM_INTER_BROKER_PROTOCOL: PLAIN
    ports:
    - "9092:9092"
    volumes:
    - ./kafka_server_jaas.conf:/etc/kafka/kafka_server_jaas.conf
    - ./producer_jaas.conf:/etc/kafka/producer_jaas.conf
    - ./consumer_jaas.conf:/etc/kafka/consumer_jaas.conf
```

# api 예제

- producer

```bash
$ kafka-console-producer.sh --broker-list localhost:9092 --topic "test" --producer.config=/etc/kafka/producer_jaas.conf
```

- consumer

```bash
$ kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic "test" --consumer.config=/etc/kafka/consumer_jaas.conf
```

- topic

```bash
$ kafka-topics.sh --bootstrap-server localhost:9092 --list --command-config /etc/kafka/consumer_jaas.conf
```
> topic은 jaas config가 필요없고 security protocol이랑 sasl mechanism만 있으면 되는데 따로 파일 만들기 귀찮아서 consumer_jaas.conf를 사용
