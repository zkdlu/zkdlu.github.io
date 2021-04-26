---
layout: post
title: "[kafka] 6.Kafka producer ack level"
description: "ack level에 따른 producer 동작"
date: 2021-03-12 00:00:00
tags: [kafka]
comments: true
share: true
---


## 내 뇌 속 오류 고쳐먹기

> 나는 지금까지 Topic의 Partition이 여러 개일 경우 메시지의 순서가 보장되지 않는다고 알고있었다.
>
> 그러나 생각해보니 Kafka는 Consumer Group을 통해 메시지 소비를 관리하기 때문에 메시지의 순서가 보장이 되는데 왜 잘못 알고있었는지 생각해보니 
>
> 메시지를 소비하고, 사용을 하는데, DB나 api 호출을 하면서 트랜잭션이나 네트워크 지연이 생기기 때문에 메시지 순서가 어긋났던 것이다.
>
> 이를 해결하기 위해 메시지에 index값을 부여하여 사용하는 곳에서 따로 순서를 보장하는 로직을 추가한다.

## + 추가

> 직접 테스트를 해보니 기존에 내가 알던 것처럼 메시지 소비 순서는 보장이 되지 않았다.
> Consumer group은 파티션 내의 오프셋은 관리를 해주지만 각 파티션 사이의 순서는 보장되지 않는 것 같다.
> 
> 그럼 나에게 위에처럼 알려줬던 그 분은 왜 저렇게 알고 있었던 걸까? 내가 제데로 테스트를 못한걸까?

## + 추가2

> 메시지를 발행할 때 키 값을 함께 보내면 Kafka는 메시지를 동일한 파티션에 넣어주어 동일한 키를 가진 메시지의 순서를 보장할 수 있다.

------

# Ack level별 Producer 동작

## 1. acks=0
Producer가 메시지를 보내고 Broker로부터 메시지를 수신했는지 기다리지 않는다.
메시지를 보내는 중간에 Broker가 다운될 경우 메시지가 손실된다.
> 속도 빠름, 손실률 높음

## 2. acks=1
Producer가 메시지를 보내고 Broker로부터 메시지를 수신했는지 기다린다.
Broker가 Replication에 메시지를 복제하기 전에 응답을 보내기 때문에, Producer로 응답을 보낸 후 Leader Broker가 다운되게 되면 Follwer Broker는 메시지를 받지 못해 메시지가 손실 된다.
> 속도 보통, 손실률 보통

## 3. acks=all(-1)
Producer가 메시지를 보내고, Broker는 Follwer Broker까지 복제가 완료 된 후 응답을 보낸다.
Leader와 Follower가 메시지를 정상적으로 받았는지 확인하기 때문에 손실률이 매우 적지만, 속도 또한 가장 느리다.
> 속도 느림, 손실률 적음

### **min.insync.replicas**
Broker에서 Producer가 acks=all 옵션으로 메시지를 보냈을 때, 응답을 보내기 위한 최소 복제 수를 설정하는 옵션.
기본 값은 1
