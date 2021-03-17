---
layout: post
title: "[spring boot] 1. Auto Configuration"
description: "Spring boot에서 Auto Configuration 사용해보기"
date: 2021-03-16 00:00:00
tags: [spring boot]
comments: true
share: true
---

 Spring boot 프로젝트를 만들면서 dependency를 추가하는데 

# 프로젝트 구조

```
📦java
 ┗ 📂com
 ┃ ┗ 📂zkdlu
 ┃ ┃ ┗ 📂apiresponsespringbootstarter
 ┃ ┃ ┃ ┣ 📂api
 ┃ ┃ ┃ ┃ ┗ 📜TestController.java
 ┃ ┃ ┃ ┣ 📂config
 ┃ ┃ ┃ ┃ ┗ 📜WebConfig.java
 ┃ ┃ ┃ ┣ 📂interceptor
 ┃ ┃ ┃ ┃ ┗ 📜ResponseAdvice.java
 ┃ ┃ ┃ ┣ 📂model
 ┃ ┃ ┃ ┃ ┣ 📜CommonResult.java
 ┃ ┃ ┃ ┃ ┣ 📜ListResult.java
 ┃ ┃ ┃ ┃ ┗ 📜SingleResult.java
 ┃ ┃ ┃ ┣ 📂service
 ┃ ┃ ┃ ┃ ┗ 📜ResponseService.java
```



1. Intellij 를 이용해 Spring boot 프로젝트 생성

   >OpenJDK 11
   >
   >Group: com.zkdlu
   >
   >Artifact: api-response-spring-boot-starter
   >
   >Java Version: 11
   >
   >Gradle

2. 프로젝트에서 전반적으로 사용 될 응답 모델을 만든다.

   > ```json
   > {
   >   "success": 성공 여부,
   >   "code": "응답 코드",
   >   "msg": "성공/실패 메시지",
   >   "data": {
   >     //data
   >   }
   > }
   > ```
   >
   > API 호출이 성공했는지, 실패했는지에 대한 데이터를 위 응답 모델의 data영역에 위치하도록 한다.

   

   공통 모델

   ```java
   public class CommonResult {
       private boolean success;
       private String code;
       private String msg;
   
       // Getter and Setter
   }
   ```

   단일 응답 모델

   ```java
   public class SingleResult<T> extends CommonResult{
       private T data;
   
       // Getter and Setter
   }
   ```

   복수 응답 모델

   ```java
   public class ListResult<T> extends  CommonResult {
       private List<T> list;
       
       // Getter and Setter
   }
   ```

3. 사용자가 Controller에서 반환한 데이터를 응답모델로 래핑하는 응답 서비스를 만든다.

   ```java
   @Service
   public class ResponseService {
       public <T> CommonResult getResult(T data) {
           if (data instanceof Collection) {
               return getListResult((List)data);
           } else {
               return getSingleResult(data);
           }
       }
   
       private <T> SingleResult<T> getSingleResult(T data) {
           SingleResult<T> result = new SingleResult<>();
           result.setData(data);
           return result;
       }
   
       private <T> ListResult<T> getListResult(List<T> list) {
           ListResult<T> result = new ListResult<>();
           result.setList(list);
           return result;
       }
   }
   ```



### 잠깐! 문제 발생

사용자가 Controller에서 반환한 데이터를 Interceptor에서 공통 모델로 래핑하여 반환하는 작업을 하려했으나, Interceptor의 PostHandler 시점은 HttpMessageConverter가 응답 메시지를 쓰고 커밋이 된 상태라 변경이 불가능했다.



### 해결 방법

Spring boot에서 제공하는 ResponseBodyAdvice를 사용하면 응답 메시지의 가공이 가능하다.

```java
@RestControllerAdvice
public class ResponseAdvice implements ResponseBodyAdvice<Object> {
    private final ResponseService responseService;

    public ResponseAdvice(ResponseService responseService) {
        this.responseService = responseService;
    }

    @Override
    public boolean supports(MethodParameter returnType, Class<? extends HttpMessageConverter<?>> converterType) {
        return true;
    }

    @Override
    public Object beforeBodyWrite(Object body, MethodParameter returnType, MediaType selectedContentType, Class<? extends HttpMessageConverter<?>> selectedConverterType, ServerHttpRequest request, ServerHttpResponse response) {
        return responseService.getResult(body);
    }
}
```

처리 후 controller를 만들어 테스트를 해보면 이런 예외 메시지가 출력된다.

```bash
java.lang.ClassCastException: class com.zkdlu.apiresponsespringbootstarter.model.SingleResult cannot be cast to class java.lang.String (com.zkdlu.apiresponsespringbootstarter.model.SingleResult is in unnamed module of loader 'app'; java.lang.String is in module java.base of loader 'bootstrap')
	at org.springframework.http.converter.StringHttpMessageConverter.addDefaultHeaders(StringHttpMessageConverter.java:44) ~[spring-web-5.3.4.jar:5.3.4]
```

예외 메시지를 보면 StringHttpMessageConverter에서 예외가 발생한다. StringHttpMessageConverter으로는  SingleResult타입을 String으로 변환할 수 없기 때문에 발생하는 예외로, WebMvcConfigurer을 구현하여 converter를 변경해준다.



```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
        converters.clear();
        converters.add(new MappingJackson2HttpMessageConverter());
    }
}
```

0번째에 추가해봤는데 어쩜 StringHttpMessageConverter만 딱 골라서 사용해서 모든 Converter를 지우고 Jackson 컨버터를 추가했다.



# 작성중..



# Auto Configuration

스프링 프로젝트에서 별도로 jar를 다운받지 않고 gradle을 통해 GitHub에서 자동으로 의존성이 추가 되도록 해보자.

1. https://jitpack.io/ 접속하여 GitHub 연동
2. 코드 작성
3. resources에 META-INF 디렉토리 생성 후 spring.factories 파일 생성

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  com.zkdlu.apiresponsespringbootstarter.autoconfiguration.config.ResponseAutoConfiguration
```



4. jar파일을 만들어야 하므로 build.gradle 수정

```gradle
...


bootJar {  enabled = false  }
jar {  enabled = true  }
```



GitHub에 레포지토리를 Push를 한후 Release를 생성하면 잠시 후 jitpack.io에서 해당 레포지토리가 빌드 되는 것을 볼 수 있다.



## 빌드 실패 1

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



## 빌드 실패 2

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



## 드디어 성공

> 사실 위에서 var 키워드를 사용해서 한번 더 수정을 해줬다...

빌드가 완료되면 이제 샘플 프로젝트를 하나 생성해서 사용해본다.



```gradle
repositories {
    maven { url 'https://jitpack.io' }
}

dependencies {
    implementation 'com.github.zkdlu:api-response-spring-boot-starter:v0.0.3'
}
```

위 내용을 각각 추가해주면 jar가 다운로드 된 것을 볼 수 있다.

