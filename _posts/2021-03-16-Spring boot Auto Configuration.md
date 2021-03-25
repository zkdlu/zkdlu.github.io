---
layout: post
title: "[spring boot] Auto Configuration"
description: "Spring boot에서 Auto Configuration 사용해보기"
date: 2021-03-16 00:00:00
tags: [spring boot]
comments: true
share: true
---

# Auto Configuration

Spring boot를 사용하면 여러 의존성들의 설정을 별도로 하지 않아도 정상적으로 작동 할 것이다.  이는 Spring Boot의 auto configuration이 추가한 jar에 따라 자동으로 설정을 해준다.

Spring boot가 실행될 때, classpath에 있는 spring.factories 파일에서 설정 클래스들을 읽어온다.

파일을 확인해보면 매우 많기 떄문에 모든 파일을 로드하면 많은 양의 메모리가 필요할 것 같지만 실제로 모든 설정이 실행되는 것이 아니라 classpath에 존재 하는 경우에만 실행된다.

### @Conditional

Spring4 에서 도입된 어노테이션으로 조건부로 Bean을 Spring Container에 등록하도록 해준다.

- @ConditionalOnClass(Test.class) : classpath에 Test class가 존재하면 Bean을 등록
- @ConditionalOnMissingClass(Test.class) : classpath에 Test class가 없으면 Bean을 등록
- @ConditionalOnBean(Test.class) : 스프링 컨테이너에 Test Bean이 존재하면 Bean을 등록
- @ConditionalOnMissingBean(Test.class) : 스프링 컨테이너에 Test Bean이 없으면 Bean을 등록



## 라이브러리를 직접 만들어보자

### 1. Spring boot 프로젝트 생성

라이브러리로 사용할 코드를 작성한다.  throw 된 RuntimeException을 Handling 하는 라이브러리를 만들어보자.

```java
@RestControllerAdvice
public class ExceptionAdvice {
    private final ExceptionProperties exceptionProperties;

    public ExceptionAdvice(ExceptionProperties exceptionProperties) {
        this.exceptionProperties = exceptionProperties;
    }
    
    @ExceptionHandler(RuntimeException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public Object handle(HttpServletRequest request, Exception e) {
        return exceptionProperties.getMsg();
    }
}
```

Auto Configuration에서 조건을 읽을 수 있는 클래스를 만든다.

```java
@Configuration
@ConditionalOnClass(ExceptionAdvice.class)
@EnableConfigurationProperties(ExceptionProperties.class)
public class ExceptionAutoConfiguration {
}
```

> classpath에 ExceptionAdvice가 존재할 경우 실행



application.properties (yml)에서 사용할 속성 클래스

```java
@ConfigurationProperties("spring.response")
public class ExceptionProperties {
    private String msg = "기본메시지";
    
    .. getter / setter ..
}
```



### 2. resources에 META-INF 디렉토리 생성 후 spring.factories 파일 생성

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  com.zkdlu.apiresponsespringbootstarter.autoconfiguration.config.ResponseAutoConfiguration
```



### 3. jar파일을 만들어야 하므로 build.gradle 수정

```gradle
...


bootJar {  enabled = false  }
jar {  enabled = true  }
```



### 4. https://jitpack.io/ 접속하여 GitHub 연동

GitHub에 레포지토리를 Push를 한후 Release를 생성하면 잠시 후 jitpack.io에서 해당 레포지토리가 빌드 되는 것을 볼 수 있다.



### 빌드 실패 1

```
Build starting...
Start: Wed Mar 17 12:21:58 UTC 2021 2b597c15fe02
Git:
v0.0.0-0-gb85cb80
commit b85cb8022a80f0314f2d04f21ede7a9d9ff4c0a1
Author: zkdlu 
Date:   Wed Mar 17 21:20:08 2021 +0900

    auto config


Found gradle
Gradle build script
Found gradle version: 6.8.3.
Using gradle wrapper
Picked up JAVA_TOOL_OPTIONS: -Dfile.encoding=UTF-8 -Dhttps.protocols=TLSv1.2
Downloading https://services.gradle.org/distributions/gradle-6.8.3-bin.zip
.10%.20%.30%.40%.50%.60%.70%.80%.90%.100%

------------------------------------------------------------
Gradle 6.8.3
------------------------------------------------------------

Build time:   2021-02-22 16:13:28 UTC
Revision:     9e26b4a9ebb910eaa1b8da8ff8575e514bc61c78

Kotlin:       1.4.20
Groovy:       2.5.12
Ant:          Apache Ant(TM) version 1.10.9 compiled on September 27 2020
JVM:          1.8.0_252 (Private Build 25.252-b09)
OS:           Linux 4.10.0-28-generic amd64

0m2.900s
Getting tasks: ./gradlew tasks --all
Picked up JAVA_TOOL_OPTIONS: -Dfile.encoding=UTF-8 -Dhttps.protocols=TLSv1.2
Tasks: 
Found javadoc task

WARNING:
Gradle 'install' task not found. Please add the 'maven' or 'android-maven' plugin.
See the documentation and examples: https://jitpack.io/docs/
```

에러 메시지를 보면 gradle에 install이란 태스크가 없으니 maven 플러그인을 추가하라고 한다. 

시키는대로 해보자.



```gradle
# build.gradle
plugins {
    id 'org.springframework.boot' version '2.3.9.RELEASE'
    id 'io.spring.dependency-management' version '1.0.11.RELEASE'
    id 'java'
    id 'maven' // 추가
}
...
```



### 빌드 실패 2

```
Build starting...
Start: Wed Mar 17 12:29:18 UTC 2021 af93306995d1
Git:
v0.0.1-0-gf6ee03e
commit f6ee03e0441b3853a332d2e3b3e7d2aba70d1522
Author: zkdlu 
Date:   Wed Mar 17 21:28:14 2021 +0900

    add maven plugin


Found gradle
Gradle build script
Found gradle version: 6.8.3.
Using gradle wrapper
Picked up JAVA_TOOL_OPTIONS: -Dfile.encoding=UTF-8 -Dhttps.protocols=TLSv1.2
Downloading https://services.gradle.org/distributions/gradle-6.8.3-bin.zip
.10%.20%.30%.40%.50%.60%.70%.80%.90%.100%

------------------------------------------------------------
Gradle 6.8.3
------------------------------------------------------------

Build time:   2021-02-22 16:13:28 UTC
Revision:     9e26b4a9ebb910eaa1b8da8ff8575e514bc61c78

Kotlin:       1.4.20
Groovy:       2.5.12
Ant:          Apache Ant(TM) version 1.10.9 compiled on September 27 2020
JVM:          1.8.0_252 (Private Build 25.252-b09)
OS:           Linux 4.10.0-28-generic amd64

0m2.812s
Getting tasks: ./gradlew tasks --all
Picked up JAVA_TOOL_OPTIONS: -Dfile.encoding=UTF-8 -Dhttps.protocols=TLSv1.2
Tasks: install,
Found javadoc task
Running: ./gradlew clean -Pgroup=com.github.zkdlu -Pversion=0.0.1 -xtest install
Picked up JAVA_TOOL_OPTIONS: -Dfile.encoding=UTF-8 -Dhttps.protocols=TLSv1.2
> Task :clean UP-TO-DATE
> Task :compileJava FAILED

FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':compileJava'.
> Could not target platform: 'Java SE 11' using tool chain: 'JDK 8 (1.8)'.
```

아 jitpack에서는 jdk 11이 안되나 본다. Gradle에 설정된 JDK 버전을 변경해보자.

```.
..
group = 'com.zkdlu'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '8'  // 11에서 변경
..
```



### 드디어 성공

> 사실 위에서 var 키워드를 사용해서 한번 더 수정을 해줬다...

빌드가 완료되면 이제 샘플 프로젝트를 하나 생성해서 사용해본다.



```gradle
repositories {
    maven { url 'https://jitpack.io' }
}

dependencies {
    implementation 'com.github.zkdlu:api-response-spring-boot-starter:1.0.0'
}
```

위 내용을 각각 추가해주면 jar가 다운로드 된 것을 볼 수 있다. 





## 추가

Spring AOP와 @Import 어노테이션을 이용하면 @Enable 어노테이션을 만들 수 있다.



```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Import({ExceptionAdvice.class})
public @interface EnableException {

}
```



@EnableException 어노테이션을 추가하면 ExceptionAdvice를 Import 하게 되고  @ConditionalOnClass(ExceptionAdvice.class) 조건을 충족된다.