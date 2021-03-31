---
layout: post
title: "[spring boot] Spring Scheduler"
description: "spring boot scheduler 사용해보기"
date: 2021-03-31 00:00:00
tags: [spring boot]
comments: true
share: true
---

# Spring Scheduler

Spring에서는 일정한 간격, 일정한 시각에 실행 시키기 위해 Scheduler 기능을 제공하는데 Spring Scheduler와 Spring Quartz가 있습니다.



## Spring Scheduler

별도의 의존성이 필요하지 않고 @EnableScheduling 어노테이션만 추가하면 됩니다. 이 후 스케줄링을 할 메서드에 @Scheduled 어노테이션을 추가해주면 됩니다. 이 때 메서드는 2가지의 조건을 만족하여야 합니다.

1. 메서드의 반환형은 void이여야 한다.
2. method는 파라미터가 없어야 한다.



### 사용해보기

Spring Scheduler를 사용하는 방법은 3가지가 있습니다. 스케줄러는 기본적으로 1개의 thread를 사용하기 때문에 동기 형식으로 진행 됩니다.

- 종료 시간 기준

  ```java
  @Scheduled(fixedDelay = 1000)
  public void delayJob() {
      log.info("schedule..");
  }
  ```

  작업이 끝난 시점을 기준으로 스케줄링 (milliseconds)

  

- 시작 시간 기준

  ```java
  @Scheduled(fixedRate = 1000)
  public void rateJob() {
      log.info("schedule..");
  }
  ```

  작업이 시작한 시점을 기준으로 스케줄링(milliseconds)



- cron

```java
@Scheduled(cron = "* * * * * *")
public void cronJob(String test) {
    System.out.println("cron : " + LocalDateTime.now().toString());
}
```

cron 표현식을 사용하여 특정 시각마다  동작한다.

> Cron 표현식
>
> second minute hour days month week 순서로 총 6개의 필드로 이루어져 있다.
>
> | 필드   | 범위                                                         |
> | ------ | ------------------------------------------------------------ |
> | second | 0 - 59                                                       |
> | minute | 0-59                                                         |
> | hour   | 0-23                                                         |
> | days   | 1-31                                                         |
> | month  | 1-12 or (JAN, FEB, MAR, APR, MAY, JUN, JUL, AUG, SEP, OCT, NOV, DEC) |
> | week   | 1-7 or (SUN, MON, TUE, WED, THU, FRI, SAT)                   |
>
> 그 외 키워드
>
> | 특수 문자 |                                |
> | --------- | ------------------------------ |
> | *         | 모든 값                        |
> | ?         | 설정 없음 (days, week 만 가능) |
> | ,         | 배열  (1,5,9) : 1, 5, 9        |
> | -         | 범위 (1-3) : 1,2,3             |
> | /         | 누적 합 (1/3) : 1,4,7,10 ....  |



- 예시

  | cron 표현식           | 의미                               |
  | --------------------- | ---------------------------------- |
  | 0 0 0 * * *           | 매일 0시                           |
  | 0 0/5 * * * ?         | 5분마다                            |
  | 0 0 12-13 * * MON-FRI | 월요일부터 금요일 12시와 13시 동안 |

  등등



## Spring Quartz

Spring Quartz는 여러 노드의 Scheduler간에 클러스터링을 지원합니다. 



작성예정