---
layout: post
title: "[spring boot] 객체 바인딩"
description: "Spring boot에서 생성되는 객체에 필요한 것들"
date: 2021-10-07 00:00:00
tags: [spring boot, feign, rest, jpa]
comments: true
share: true
---



## 해당하는 부분

1. Controller에서 오는 요청의 @RequestBody 객체
2. Controller에서 오는 요청의 @ModelAttribute 객체
3. Jpa가 생성하는 Entity 객체 
4. Feign Client에서 사용되는 Return 객체



### RequestBody의 경우

```java
@AllArgsConstructor
@Getter
class BodyDemo {
    private String name;
    private String description;
}
```

기본 생성자가 없는 모델을 @RequestBody로 사용할 경우

```java
 @PostMapping("/body")
public String body(@RequestBody BodyDemo demo) {
    return "body";
}
```

아래의  Exception이 발생한다.

```bash
Caused by: org.springframework.http.converter.HttpMessageConversionException: Type definition error: [simple type, class com.zkdlu.communicate.BodyDemo]; nested exception is com.fasterxml.jackson.databind.exc.InvalidDefinitionException: Cannot construct instance of `com.zkdlu.communicate.BodyDemo` (no Creators, like default constructor, exist): cannot deserialize from Object value (no delegate- or property-based Creator)
 at [Source: (PushbackInputStream); line: 1, column: 2]
	at org.springframework.http.converter.json.AbstractJackson2HttpMessageConverter.readJavaType(AbstractJackson2HttpMessageConverter.java:386)
	at org.springframework.http.converter.json.AbstractJackson2HttpMessageConverter.read(AbstractJackson2HttpMessageConverter.java:342)
	at org.springframework.web.servlet.mvc.method.annotation.AbstractMessageConverterMethodArgumentResolver.readWithMessageConverters(AbstractMessageConverterMethodArgumentResolver.java:185)
	at org.springframework.web.servlet.mvc.method.annotation.RequestResponseBodyMethodProcessor.readWithMessageConverters(RequestResponseBodyMethodProcessor.java:160)
	at org.springframework.web.servlet.mvc.method.annotation.RequestResponseBodyMethodProcessor.resolveArgument(RequestResponseBodyMethodProcessor.java:133)
	at org.springframework.web.method.support.HandlerMethodArgumentResolverComposite.resolveArgument(HandlerMethodArgumentResolverComposite.java:121)
	at org.springframework.web.method.support.InvocableHandlerMethod.getMethodArgumentValues(InvocableHandlerMethod.java:179)
	at org.springframework.web.method.support.InvocableHandlerMethod.invokeForRequest(InvocableHandlerMethod.java:146)
	at org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod.invokeAndHandle(ServletInvocableHandlerMethod.java:117)
	at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandlerMethod(RequestMappingHandlerAdapter.java:895)
	at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.handleInternal(RequestMappingHandlerAdapter.java:808)
	at org.springframework.web.servlet.mvc.method.AbstractHandlerMethodAdapter.handle(AbstractHandlerMethodAdapter.java:87)
	at org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:1067)
	at org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:963)
	at org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:1006)
	... 74 more
Caused by: com.fasterxml.jackson.databind.exc.InvalidDefinitionException: Cannot construct instance of `com.zkdlu.communicate.BodyDemo` (no Creators, like default constructor, exist): cannot deserialize from Object value (no delegate- or property-based Creator)
 at [Source: (PushbackInputStream); line: 1, column: 2]

```



### ModelAttribute의 경우

```java
@Getter
class ModelDemo {
    private String name;
    private String description;
}
```

기본 생성자만 가지고 있는 경우

```java
@GetMapping("/model")
public ModelDemo model(@ModelAttribute ModelDemo demo) {
    return demo;
}
```

setter가 없기떄문에 바인딩이 되지 않는다.

 ![image-20211016142644823](https://zkdlu.github.io/images/etc/modelattribute.png)



### Feign Client의 경우

```java
@AllArgsConstructor
@Getter
class Demo {
    private String name;
    private String description;
}
```

기본 생성자가 없는 경우에 Feign Client를 호출할 경우.

```java
@Component
@FeignClient(name = "demo", url = "localhost:8080")
interface DemoClient {
    @GetMapping("/demo")
    Demo getDemo();
}
```

아래의 Exception이 발생한다.

```bash
com.fasterxml.jackson.databind.exc.InvalidDefinitionException: Cannot construct instance of `com.zkdlu.communicate.Demo` (no Creators, like default constructor, exist): cannot deserialize from Object value (no delegate- or property-based Creator)
 at [Source: (PushbackInputStream); line: 1, column: 2]
	at com.fasterxml.jackson.databind.exc.InvalidDefinitionException.from(InvalidDefinitionException.java:67) ~[jackson-databind-2.11.4.jar:2.11.4]
	at com.fasterxml.jackson.databind.DeserializationContext.reportBadDefinition(DeserializationContext.java:1615) ~[jackson-databind-2.11.4.jar:2.11.4]
	at com.fasterxml.jackson.databind.DatabindContext.reportBadDefinition(DatabindContext.java:400) ~[jackson-databind-2.11.4.jar:2.11.4]
	at com.fasterxml.jackson.databind.DeserializationContext.handleMissingInstantiator(DeserializationContext.java:1077) ~[jackson-databind-2.11.4.jar:2.11.4]
	at com.fasterxml.jackson.databind.deser.BeanDeserializerBase.deserializeFromObjectUsingNonDefault(BeanDeserializerBase.java:1332) ~[jackson-databind-2.11.4.jar:2.11.4]
	at com.fasterxml.jackson.databind.deser.BeanDeserializer.deserializeFromObject(BeanDeserializer.java:331) ~[jackson-databind-2.11.4.jar:2.11.4]
	at com.fasterxml.jackson.databind.deser.BeanDeserializer.deserialize(BeanDeserializer.java:164) ~[jackson-databind-2.11.4.jar:2.11.4]
	at com.fasterxml.jackson.databind.ObjectMapper._readMapAndClose(ObjectMapper.java:4526) ~[jackson-databind-2.11.4.jar:2.11.4]
	at com.fasterxml.jackson.databind.ObjectMapper.readValue(ObjectMapper.java:3521) ~[jackson-databind-2.11.4.jar:2.11.4]
	at org.springframework.http.converter.json.AbstractJackson2HttpMessageConverter.readJavaType(AbstractJackson2HttpMessageConverter.java:378) ~[spring-web-5.3.10.jar:5.3.10]
	at org.springframework.http.converter.json.AbstractJackson2HttpMessageConverter.read(AbstractJackson2HttpMessageConverter.java:342) ~[spring-web-5.3.10.jar:5.3.10]
	at org.springframework.web.client.HttpMessageConverterExtractor.extractData(HttpMessageConverterExtractor.java:105) ~[spring-web-5.3.10.jar:5.3.10]
	at org.springframework.cloud.openfeign.support.SpringDecoder.decode(SpringDecoder.java:57) ~[spring-cloud-openfeign-core-3.0.4.jar:3.0.4]
	at org.springframework.cloud.openfeign.support.ResponseEntityDecoder.decode(ResponseEntityDecoder.java:61) ~[spring-cloud-openfeign-core-3.0.4.jar:3.0.4]
	at feign.optionals.OptionalDecoder.decode(OptionalDecoder.java:36) ~[feign-core-10.12.jar:na]
	at feign.AsyncResponseHandler.decode(AsyncResponseHandler.java:115) ~[feign-core-10.12.jar:na]
	at feign.AsyncResponseHandler.handleResponse(AsyncResponseHandler.java:87) ~[feign-core-10.12.jar:na]
	at feign.SynchronousMethodHandler.executeAndDecode(SynchronousMethodHandler.java:138) ~[feign-core-10.12.jar:na]
	at feign.SynchronousMethodHandler.invoke(SynchronousMethodHandler.java:89) ~[feign-core-10.12.jar:na]
	at feign.ReflectiveFeign$FeignInvocationHandler.invoke(ReflectiveFeign.java:100) ~[feign-core-10.12.jar:na]
	at com.zkdlu.communicate.$Proxy61.getDemo(Unknown Source) ~[na:na]
	at com.zkdlu.communicate.DemoServiceImpl.getDemo(DemoServiceImpl.java:15) ~[main/:na]
```





**결론**

RequestBody와 Feign에서 사용되는 모델의 경우 application/json타입으로 데이터가 오며 이는 스프링부트의 MessageConverter가 Jackson 라이브러리를 이용해 Reflection으로 객체를 생성하게 되는데  기본 생성자가 없을 경우 Jackson 라이브러리가 deserialize 할 수 없어 예외가 발생한다.



- application/json타입이 매핑 되는 거라면 NoArgsConstructor를 둔다.

  > 객체 생성을 막고싶다면 private 생성자로

- 그게 아니라면 Setter나 AllArgsConstructor를 두자

  > 기왕이면 AllArgsConstructor를 두도록 하자