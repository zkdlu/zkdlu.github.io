---
layout: post
title: "[MSA] Netflix OSS"
description: "hystrix"
date: 2021-01-13 00:00:00
tags: [MSA, Spring boot]
comments: true
share: true
---

# 1. Hystrix

Netflix가 만든 Fault Tolerance Library



###  주요 기능

- Circuit Breaker
- Fallback
- Thread Isolation
- Timeout



### 사용법

1. HystrixCommand 어노테이션 사용

```java
@HystrixCommand
public String method() {

}
```

2. HystrixCommand 상속

```java
public class Test extends HystrixCommand<String> {
    @Override
    protected String run() {

    }
}
```

   

### HystrixCommand 를 호출하면 벌어지는 일

1. 호출한 메서드를 Intercept하여 대신 실행한다.

   > 디버거를 보면 스레드가 변경 되는걸 확인 가능

   - Thread isolation

2. 메서드의 실행결과 성공 혹은 실패 발생 여부를 기록하고 통계를 낸다. 통계에 따라 Circuit Open 여부를 결정

   - Circuit breaker

3. 실패한 경우(Exception)  사용자가 제공한 메서드를 실행한다.

   - Fallback

4. 특정시간동안 메서드가 종료되지 않는 경우 Exception을 발생

   - Timeout



### 대부분 라이브러리에 타임아웃 주는데 별도 인터셉트하는 레이어에서 타임아웃 줄 필요 있는가?

소켓 커넥트 타임아웃은 타임아웃 값대로 종료되지 않는 경우가 많고 (이유는 뭘까?)

jdbc connect timeout , jdbc query timeout 등 수많은 요소로 타임아웃 결정하기 힘들어서 서비스레이어에 타임아웃을 걸기 적당함



### Circuit Breaker

(1)**일정 시간** 동안 (2)**일정 개수 이상**의 호출이 발생한 경우 (3)**일정 비율 이상**의 에러가 발생하면 Circuit Open(호출 차단)

(4)**일정 시간 경과** 후에 단 한개의 요청에 대해 호출을 허용하며(Half Open), 이 호출이 성공하면 Circuit Close (호출허용)

```properties
hysrix.command.( server id )

(1)metrics.rolllingStats.timeInMilliseconds # Default 10초
(2)circuitBreaker.requestVolumeThreshold # Default 20개
(3)circuitBreaker.errorThresholdPercentage # Default 50%
(4)circuitBreaker.sleepWindowInMilliseconds # Default 5초
```

> 10초동안 20개 이상의 호출이 발생한 경우 50%이상의 에러가 발생하면 5초간 Circuit Open



Hystrix Circuit Breaker는 한 프로세스 내에서 CommandKey 단위로 통계를 공유하고 동작한다. - CommandKey 단위로 Circuit Breaker가 생성된다.

```java
@HystrixCommand(commandKey = "key1")
...method...
    
@HystrixCommand(commandKey = "key2")
...other method...
```

서로 같은 commandKey를 사용하기 떄문에 하나가 차단되면 다른 것도 차단 됨



### Fallback

Fallback으로 지정된 메서드는 다음의 경우에 원본 메서드 대신 실행된다.

- Circuit Open
- Exception (HystrixBadRequestException 제외)
- Semaphore / Isolation에 의해 발생 한 ThreadPool Rejection
- Timeout

```java
@HystrixCommand(fallbackMethod="fallbackMethod")
public String method() {

}
public String fallbackMethod() {
    return "준비된 목록";
}
```

fallback으로 인해 exception이 감춰질 수 있다.



HystrixBadRequestException은 사용자의 코드에서 발생시킬 경우 이 오류는 Fallback을 실행하지 않고 Circuit Breaker가 통계에 집계하지 않는다.





### Timeout

Circuit Breaker단위로 타임아웃을 지정

```properties
hystrix.command.( server id )

exception.isolation.thread.timeoutInMilliseconds # Default 1초
```



주의 할 점

- Semaphore isolation의 경우 제 시간에 Timeout이 발생하지 않는 경우가  대부분
- Default값이 1초인데 매우 짧다.



### Isolation

- Semaphore Isolation

  > - CircuitBreaker 1개당 1개의 Semaphore 생성
  >
  > - Semaphore별로 최대 동시 요청 개수 지정
  >
  > - 최대 개수 초과시 Semaphore Rejection 발생 - Fallback 실행
  >
  > - Command를 호출한 스레드에서 메서드가 실행된다.
  >
  > - * Timeout이 제 시간에 발생하지 못한다. 
  >
  >   * > Java의 태생적 한계 - 현재 버전의 자바에서는 다른 스레드를 인위적으로 중단할 수 없음. thread.interrupt도 실제 스레드 내부에서 InterruptedException을 처리할 경우에만 중단 가능 함

- Thread Isolation

  > - Circuit Breaker 별로 사용할 Thread Pool을 지정 (ThreadPoolKey)
  > - Circuit Breaker:Thread Pool = N : 1관계 가능
  > - 최대 개수 초과시 Thread Pool Rejection 발생 - Fallback 실행
  > - 실제 메서드를 호출한 스레드가 아닌 Thread Pool에서 실행한다.



# 2. Ribbon

Netflix가 만든 Software Load Balancer를 내장한 RPC(REST) Library

- Client Load Balancer with HTTP Client



Spring Cloud에서는 Ribbon 클라이언트를 사용자가 직접 사용하지 않음

> 옵션이나 설정으로 사용

Spring Cloud의 HTTP 통신이 필요한 요소에 내장되어 있음

1. Zuul API Gateway

2. RestTemplate (@LoadBalanced를 사용)

   > spring cloud가 있다면 resttemplate이 빈으로 설정된 경우에 한해서 @LoadBalanced를 사용하면 로드밸런싱이 된다.

3. Spring Cloud Feign - 선언적 Http Client



리본을 사용하면 로드밸런싱을 Programmable하게 제어가 가능하다.

Spring Cloud에서는 아래의 BeanType으로 제어

> - IRule - 주어진 서버 목록에서 어떤 서버를 선택?
> - IPing - 각 서버가 살아있는지 검사
> - ServerList<Server> - 대상 서버 목록 제공
> - ServerListFilter<Server> - 대상 서버들 중 호출할 대상 Filter
> - ServerListUpdater
> - IClientConfig
> - ILoadBalancer





# 3.Eureka

Netfilx가 만든 Dynamic Service Discovery

- 등록: 서버가 자신의 서비스 이름과 IP주소, 포트를 등록
- 조회: 서비스 이름을 갖고 서버 목록을 조회



1. Server가 시작 시 Eureka 서버에 자동으로 자신의 상태 등록 (UP)
2. 주기적 HeartBeat으로 Eureka Server에 자신이 살아있음을 알림
3. Server 종료 시 Eureka 서버에 자신의 상태 변경 (Down) 혹은 자신의 목록 삭제
4. Eureka 상에 등록된 이름은 spring.application.name



# Eureka + Ribbon in Spring cloud

하나의 서버에 Eureka Client와 Ribbon Client가 함꼐 설정되면 Spring Cloud는 Ribbon Bean을 대체한다.

1. ServerList<Server>
   * 기본: ConfigurationBasedServerList
   * 변경: DiscoveryEnabledNIWSServerList
2. IPing
   * 기본: DummyPing
   * 변경: NIWSDiscoveryPing

서버의 목록을 설정으로 명시하는 대신 Eureka를 통해서 Lookup 해온다.





# Zuul

Spring Cloud Zuul은 API Routing을 Hysrix, Ribbon, Eureka를 통해서 구현



Zuul에서 Hysrix Isolation은 Semaphore Isolation을 기본으로 한다.





# Server간의 호출

전부터 외부 호출은 API Gateway를 통해 들어오지만 내부 서버들은 어떻게 호출하는지 고민을 했다. 내부 서버도 Gateway를 이용할 경우 모든 API호출을 통제할 수는 있지만 Single Point Failure(게이트웨이가 죽으면 전체 장애)가 많이 발생하고 서비스의 수가 늘어나면 엄청난 트래픽이 생긴다.

"Ribbon + Eureka 조합으로 Peer to Peer 호출을 한다."

1. RestTemplate에 @LoadBalanced 어노테이션을 붙이면 Ribbon + Eureka 기능을 갖는다.
2. Spring Cloud Feign





# 모니터링

Zipkin