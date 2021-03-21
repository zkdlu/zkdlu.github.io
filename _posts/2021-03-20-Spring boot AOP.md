---
layout: post
title: "[spring boot] AOP"
description: "Spring boot에서 AOP 사용해보기"
date: 2021-03-20 00:00:00
tags: [spring boot]
comments: true
share: true
---

# AOP (Aspect Oriented Programming)

관점지향 프로그래밍으로 OOP를 보완하는 수단으로 비즈니스 기능과 공통기능으로 구분한 후 흩어진 관심사(Aspect)를 모듈화 하여 비즈니스 로직에서 분리해준다.

성능검사, 트랜잭션 처리, 예외 변환, 로깅 등  여러 곳에서 사용되는 공통 기능을 Aspect로 모듈화한다.



![aop](https://zkdlu.github.io/images/spring/aop.png)



## AOP 주요 개념

### Aspect

 = Advice + PointCut

모듈화 된 관심사. 공통 기능

### Target

부가기능을 부여할 대상. Aspect를 적용 하는 곳

### Advice

Target에 제공하는 부가기능의 실질적인 기능을 담은 모듈

### Joint Point

관심 묘듈의 기능이 삽입되어 동작할 수 있는 곳. Advice가 적용될 위치, 

### Point cut

Joint Point의 상세 스펙을 정의한 표현식

- executeion(접근한정자 반환형 메서드)
- within(class path)
- bean(bean id)
- annotation(annotation name)

| Point cut                                      | Joint Point                                                  |
| ---------------------------------------------- | ------------------------------------------------------------ |
| execution(public * *(..))                      | public 한정자 *:모든 반환형, *:모든 경로, (..): 모든 메서드 호출 시 |
| execution(* *(..))                             | *: 모든 접근한정자와 반환형, *: 모든 경로, (..): 모든 메서드 호출 시 |
| execution(* set*(..))                          | *: 모든 접근한정자와 반환형, 이름이 set으로 시작하는 모든 메소드 호출 시 |
| execution(* com.zkdlu.demo.DemoService.todo()) | *: 모든 접근한정자와 반환형, 해당 classpath의 todo() 메서드 호출 시 |
| execution(* com.zkdlu..\*.\*())                | ..: 하위 패키지 모두 포함. 모든 클래스, 모든 메서드 호출 시  |
| within(com.zkdlu.demo.*)                       | 패키지 내의 클래스 메서드 호출 시                            |
| within(com.zkdlu.demo..*)                      | 패키지의 하위 패키지를 모두 포함한 모든 클래스의 메서드 호출 시 |
| bean(demo)                                     | 해당 bean id를 가지고 있는 bean의 모든 메서드 호출 시        |
| @annotation(MyLogger)                          | annotation이 적용된 메서드 호출 시                           |






## Spring AOP

프록시 패턴을 기반으로 된 구현 체. 

aop 의존성 추가

```gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-aop'
}
```



### 로깅 AOP 예제

```java
@Aspect
@Component
public class LogAspect {
    Logger logger = LoggerFactory.getLogger(LogAspect.class);

    @Around("execution(* com.zkdlu.demo.service..*.*(..))")
    public Object logging(ProceedingJoinPoint joinPoint) throws Throwable {
        logger.info("Begin method");
        Object result = joinPoint.proceed();
        logger.info("End method");
        return result;
    }
}
```

> com.zkdlu.demo.service패키지 내의 모든 메서드 호출 시 로깅 작성



### Annotation AOP 예제

샘플 Annotation

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface MyAnnotation {
}
```



AOP 예제

```java
@Aspect
@Component
public class MyAspect {
    Logger logger = LoggerFactory.getLogger(MyAspect.class);

    @Around("@annotation(MyAnnotation)")
    public Object test(ProceedingJoinPoint joinPoint) throws Throwable {
        logger.info("annotation aop");

        return joinPoint.proceed();
    }
}
```

> MyAnnotaion이 있는 모든 메서드 호출시 동작한다.