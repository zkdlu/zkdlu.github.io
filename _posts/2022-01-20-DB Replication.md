---
layout: post
title: "[DB] Replication"
description: "Mariadb Replication 설정"
date: 2022-01-20 00:00:00
tags: [db]
comments: true
share: true
---

## Replication
- DB 복제. 비동기 복제 방식
- 실시간 데이터 백업이 가능
- DB 서버 부하 분산 (master : CUD, slave : R)

### master의 역할
데이터에 대한 변경 (Create, update, delete) event 발생 시 binary logs에 기록하고, slave 서버에 전달.

### slave의 역할
master에서 저달받은 binarylog를 읽어 DB에 반영


- binary log

   > mysql에서 발생하는 모든 내역들이 기록되는 파일. default 비활성화 상태


### 복제 방식
- STATEMENT : SQL 복사
- ROW : 변경된 ROW 복사
- MIXED : STATEMENT + ROW


### 예제

...