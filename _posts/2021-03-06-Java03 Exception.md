---
layout: post
title: "[Java] 3. Exception"
description: "Checked Exception과 Unchecked Exception"
date: 2021-03-06 00:00:00
tags: [java]
comments: true
share: true
---



# Exception

![exception](https://zkdlu.github.io/images/java/exception.png)



## Checked Exception

컴파일 시간에 검사하는 예외. 처리 하지 않으면 컴파일 에러가 나기 때문에 반드시 처리해야 함

try/catch 나 throws로 처리해주어야 한다.

트랜잭션이 Rollback되지 않는다.

## Unchecked Exception

런타임 시간에 검사하는 예외. 주로 프로그래머의 실수, 사용자가 잘못된 사용을 할 때 발생 함.

트랜잭션이 Rollback 됨.