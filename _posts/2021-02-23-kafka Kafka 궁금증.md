---
layout: post
title: "[kafka] Kafka 궁금증"
description: "하나의 Consumer group으로 여러 Topic을 구독하면."
date: 2021-02-23 00:00:00
tags: [kafka, docker]
comments: true
share: true
---

## Consumer group

Kaka를 공부하면서 Consumer group이란 개념이 나온다.

Consumer group은 Consumer들을 하나로 묶는 논리적 그룹 단위이다. Consumer group은 토픽의 오프셋을 관리하기 때문에 컨슈머간에 장애가 발생했을 경우 동일 그룹 내의 다른  컨슈머가 데이터를 계속 읽을 수 있다.

하나의 Consumer group은 하나의 Topic에 대한 책임을 가지고 있고, 1개의 Topic을 여러 Consumer group이 구독을 할 수 있다고 공부했다.

그러다 테스트를 해보기로 하였다.

### 한개의 Consumer group이 여러개의 Topic을 구독해보자.

예제에 사용한 카프카 이미지는 다음과 같다.

- bitnami/kafka:2-debian-10
- bitnami/zookeeper:3-debian-10

먼저 구독할 토픽을 여러개 만들어 두었다.

```bash
$ kafka-topics.sh --bootstrap-server localhost:9092 --create --topic "test1" --partitions 1 --replication-factor 1
Created topic test1.
$ kafka-topics.sh --bootstrap-server localhost:9092 --create --topic "test2" --partitions 1 --replication-factor 1
Created topic test2.
$ kafka-topics.sh --bootstrap-server localhost:9092 --create --topic "test3" --partitions 1 --replication-factor 1
Created topic test3.
```

그 후 같은 컨슈머 그룹을 가진 컨슈머에 각각 다른 토픽을 설정해보았다.

```bash
$ kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic "test1" --group "my-group"
$ kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic "test2" --group "my-group"
$ kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic "test3" --group "my-group"
```

producer를 이용해 몇 차례 메시지를 게시 한 후 컨슈머 그룹을 확인해보았다.

```bash
$ kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group my-group
Consumer group 'my-group' has no active members.

GROUP           TOPIC           PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID     HOST            CLIENT-ID
my-group       test3           0          0               0               0               -               -               -
my-group       test2           0          32              32              0               -               -               -
my-group       test1           0          132             132             0               -               -               -
```

다음과 같이 한 Consumer group에서 여러 Topic의 오프셋을 각각 관리하는 것을 확인 할 수 있었다.
