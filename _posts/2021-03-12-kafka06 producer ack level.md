---
layout: post
title: "[kafka] 6.kafka producer ack level"
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
> > 멍청했다..
>
> 이를 해결하기 위해 메시지에 index값을 부여하여 사용하는 곳에서 따로 순서를 보장하는 로직을 추가한다.




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