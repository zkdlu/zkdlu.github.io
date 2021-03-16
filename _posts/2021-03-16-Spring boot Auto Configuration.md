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





# 작성중