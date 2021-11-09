---
layout: post
title: "[MSA]Outbox Pattern"
description: "Outbox Pattern"
date: 2021-11-09 00:00:00
tags: [spring boot, msa, kafka]
comments: true
share: true
---

## Eventual Consistency

MSA에서 도메인의 상태가 변경되면, Eventual Consistency를 위해 이벤트를 발행하여 상태가 변경되었다고 알리기 위해 메시지 브로커를 이용합니다. 

상태 변경과 이벤트 발행은 한 트랜잭션 안에 이루어지지만, 메시지 브로커는 데이터베이스 트랜잭션을 함께 사용할 수 없어서 서비스간 데이터 일관성이 깨지게 됩니다.

1. 상태 변경은 성공했으나, 메시지 발행에 실패 
2. 메시지 발행에 성공했으나, 상태 변경에 실패 

일관성을 유지하기 위해 데이터베이스 트랜잭션과 메시지 발행은 하나의 트랜잭션으로 동작하여야 합니다.

## Outbox Pattern

RDB를 메시지 큐로서 사용하는 것

이벤트 발행을 메시지 브로커에 하지 않고, OUTBOX라는 테이블에 메시지를 저장합니다.

저장된 메시지는 별도의 Message Relay를 통해 메시지 브로커에 메시지를 발행하여, 메시지 발행과 상태 변경을 동일한 트랜잭션으로 처리할 수 있게 됩니다.

```sql
CREATE TABLE OUTBOX (
   id varchar(255) primary key,
   aggregate_id varchar(255) not null, -- 순서 처리를 위해 사용 (partition key)
   aggregate_type varchar(255) not null, -- 변경이 발생한 도메인
   event_type varchar(255) not null, -- 발생한 이벤트
   payload text not null -- 도메인 변경사항
);
```
