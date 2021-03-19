---
layout: post
title: "[Java] 4. Annotation"
description: "Java Annotation 만들기"
date: 2021-03-19 00:00:00
tags: [java]
comments: true
share: true
---

# Annotation

사전적의미로 '주석'이란 의미를 갖고있고,  자바 소스 코드에 메타 코드를 부여하는 용도로 사용된다.

클래스, 메소드, 변수, 패키지 등에 annotation 을 추가할 수 있고, annotation은 컴파일러에 의해 생성된는 .class파일에서 JVM이 실행될 때, annotation은 reflection을 사용해 읽어들인다.



## Built in Annotation

자바에서 제공하는 기본 어노테이션

> - @Override : 메서드가 오버라이드 됐는지 검증
> - @Deprecated
> - @SuppressWarnings
> - @SafeVarargs
> - @FunctionalInterface : 람다 함수를 위한 인터페이스를 지정



## Meta Annotation

커스텀 어노테이션을 만들때 사용하는 어노테이션

> @Retention : 어노테이션의 범위
> - @Retention(RetentionPolicy.RUNTIME) : 런타임시점에 JVM에 의해 참조 가능
> - @Retention(RetentionPolicy.CLASS) : 클래스를 참조할 때까지 참조 가능
> - @Retention(RetentionPolicy.SOURCE) : 컴파일 이후 참조 불가능
>
> @Documented : 문서에 어노테이션의 정보를 표현
>
> @Target : 어노테이션이 적용될 위치
>
> - @Target(ElementType.PACKAGE) : 패키지
> - @Target(ElementType.TYPE) : 클래스
> - @Target(ElementType.CONSTRUCTOR) : 생성자
> - @Target(ElementType.FIELD) : 멤버 필드
> - @Target(ElementType.METHOD) : 메서드
> - @Target(ElementType.ANNOTATION_TYPE) : 어노테이션 타입
> - @Target(ElementType.LOCAL_VARIABLE) : 지역 변수
> - @Target(ElementType.PARAMETER) : 매개 변수
>
> @Inherited : 자식클래스가 어노테이션을 상속받음
>
> @Repeatable : 반복적으로 어노테이션을 선언 할 수 있음



## Custom Annotation

@interface 키워드를 사용해 커스텀 어노테이션을 작성한다.

어노테이션은 Element를 멤버로 가질 수 있고, Element는 타입과 이름으로 구성되며 디폴트 값을 가지고 있다.

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface MyAnnotation {
    String test() default "test"
}
```

예제

```java
pubilc class Test() {
    @MyAnnotation(test = "hello")
    public void test() {
    }
}
```

요런식으로 어노테이션의 값을 가져올 수 있다.

```java
public void run() {
    Method[] methods = Test.class.getDeclaredMethods();
    for (var method : methods) {
        if (method.isAnnotationPresent(TestAnnotation.class)) {
            System.out.println(method.getName());
            TestAnnotation testAnnotation = method.getAnnotation(TestAnnotation.class);
            System.out.println(testAnnotation.test());
        }
    }
}
```



