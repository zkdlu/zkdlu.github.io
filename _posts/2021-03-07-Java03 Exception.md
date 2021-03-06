---
layout: post
title: "[Java] 3. Exception"
description: "Checked Exception과 Unchecked Exception"
date: 2021-03-07 00:00:00
tags: [java]
comments: true
share: true
---



# Exception

프로그램을 만든 프로그래머가 상정한 정상적인 처리에서 벗어나는 경우에 이를 처리하기 위한 방법

![exception](https://zkdlu.github.io/images/java/exception.png)



## Checked Exception

Exception을 상속받음

컴파일 시간에 검사하는 예외. 처리 하지 않으면 컴파일 에러가 나기 때문에 반드시 처리해야 함

try/catch 나 throws로 처리해주어야 한다.

트랜잭션이 Rollback되지 않는다.

## Unchecked Exception

Runtime Exception을 상속받음

런타임 시간에 검사하는 예외. 주로 프로그래머의 실수, 사용자가 잘못된 사용을 할 때 발생 함.

트랜잭션이 Rollback 됨.



### Exception Handling (Effective Java)

- 예외는 예외 상황에서만 사용되어야 한다. 예외를 생성하고 던지고 잡는 것은 비용이 많이 들고. 프로그램의 흐름을 예외로 제어하려 하면 안된다. 

- 복구 가능한 조건에 **Checked Exception**, 프로그래밍 에러에 **Runtime Exception**을 사용한다. 

- Checked Exception은 꼭 필요할 때만 던져야 하고, catch에서 특별히 하는 작업이 없는 API를 checked exception으로 처리하는 것은 프로그램을 복잡하게 만든다.
- Java의 표준 예외를 활용한다.
- 예외를 적절하게 추상화 한다. 높은 계층에서 낮은 계층의 예외를 잡고 높은 계층의 추상화에 맞게 변환해서 져야 한다.
- 실패에 대한 자세한 정보를 상세 메시지에 담아야 한다. 실패원인을 포착하려면, 예외의 문자열 표현에 반드시 예외 발생에 영향을 준 모든 필드와 인자와 값이 들어 있어야 한다.
- 실패 원자성을 얻어야 한다. 메서드 호출이 실패하더라고 객체 상태는 메서드 호출 전과 같아야 한다. 
- 예외를 잡아서 버리지 마라. 진짜 하지 마라.



### 흠. 자바는 왜 Checked와 Unchecked로 나뉘어져 있는가?

API를 설계할 때 해당 메서드가 어떤 예외를 발생 시킬 수 있는가 또한 API 규약의 중요한 내용이고, 때로는 그런 예외에 대한 처리를 호출자에게 강제할 수 있어야 한다.

비즈니스 로직에서 발생한 예외는 전부 롤백이 되어야 하나?

예외 복구 전략이 명확하다면 Checked Exception을 사용해 try-catch로 잡고 복구를 수행한다. 그렇지 않다면 더 구체적인 Unchecked Exception을 발생시킨다.

무책임하게 상위 메서드로 throw를 던지는 행위는 상위 메서드들이 책임이 그만큼 책임이 증가한다.