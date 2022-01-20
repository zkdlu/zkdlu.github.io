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

## master의 역할
데이터에 대한 변경 (Create, update, delete) event 발생 시 binary logs에 기록하고, slave 서버에 전달.

## slave의 역할
master에서 저달받은 binarylog를 읽어 DB에 반영


- binary log

   > mysql에서 발생하는 모든 내역들이 기록되는 파일. default 비활성화 상태


## 복제 방식
- STATEMENT : SQL 복사
- ROW : 변경된 ROW 복사
- MIXED : STATEMENT + ROW


## 예제

### DB 준비
```yaml
version: '3'
services:
  mariadb-container-master:
    image: mariadb:latest
    restart: always
    ports:
      - '3307:3306'
    volumes:
      - ./master/init.sql:/docker-entrypoint-initdb.d/init.sql
      - ./master/my.cnf:/etc/mysql/conf.d/my.cnf
    environment:
      - MYSQL_ROOT_PASSWORD=backend
  mariadb-container-slave:
    image: mariadb:latest
    restart: always
    ports:
      - '3308:3306'
    volumes:
      - ./slave/init.sql:/docker-entrypoint-initdb.d/init.sql
      - ./slave/my.cnf:/etc/mysql/conf.d/my.cnf
    environment:
      - MYSQL_ROOT_PASSWORD=backend
    depends_on:
      - mariadb-container-master
```

### Master DB
1. Master DB 설정

```cnf
log-bin=mysql-bin
server-id=1
binlog_format=row
```

2. DB 재시작
3. slave에서 접속할 계정 생성

```bash
mysql> grant replication slave on *.* to 'slaveuser'@'%' identified by 'slavepassword';
```

4. binary log 확인
```bash
> show master status;
```
> File명과 Position은 DB 재시작시 변경될 수 있어 기록해두어야 함.

### Slave DB

1. Slave DB 설정
```cnf
log-bin=mysql-bin
server-id=2
binlog_format=row
```

2. DB 재시작
3. master 연동

```bash
mysql> change master to
   \ master_host="master-host",
   \ master_user="slaveuser",
   \ master_password="slavepassword",
   \ master_port=3307,
   \ master_log_file="bin log 파일명",
   \ master_log_pos="bin log position";
```
4. slave 실행
```bash
mysql> start slave;
```
5. slave 상태 확인
```bash
mysql> show slave status;
```



### Connection Refusing
slave에서 master 를 설정해줄 때 master_host로 master 컨테이너를 해주는데 아래와 같은 오류가 출력된다.

```bash
database-mariadb-container-slave-1   | 2022-01-20 14:14:31 6 [ERROR] Slave I/O: error connecting to master 'slaveuser@database-mariadb-container-master-1:3307' - retry-time: 60  maximum-retries: 100000  message: Can't connect to server on 'database-mariadb-container-master-1' (111 "Connection refused"), Internal MariaDB error code: 2003
```

> Unknown server host가 출력되는 것이 아니니 도커 호스트를 제데로 읽은거 같은데 이유를 알아봐야 겠다. 아이피로 직접 적어주이 되긴 하는데,, 흠...