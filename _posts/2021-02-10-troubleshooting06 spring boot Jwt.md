---
layout: post
title: "[Trouble shooting] 5.Spring boot Jwt"
description: "Spring boot에서 Jwt를 발급 할때 발생 한 예외"
date: 2021-02-10 00:00:00
tags: [trouble shooting, spring boot, jwt]
comments: true
share: true
---

### 무엇을 했는가?

Spring Security와 JWT를 이용해 유저 인증 후 로그인 시도를 하면 Jwt 토큰을 발급 해 주려고 함.

```java
@PostMapping("/signin")
public SingleResult<String> signin(@RequestParam String id,
                                   @RequestParam String password){
    User user = userRepository.findByUid(id).orElseThrow(UserNotFoundException::new);

    if (!passwordEncoder.matches(password, user.getPassword())) {
        throw new PasswordNotMatchException();
    }

    return responseService.getSingleResult(
        jwtTokenProvider.createToken(user.getUid(), user.getRole()));
}
```



### 어떤일이 있었는가?

인증을 시도하면 토큰을 발급해주는데 암호화를 하는 과정에서 예외가 발생하였다.

```java
public String createToken(String uid, String role) {
    Claims claims = Jwts.claims().setSubject(uid);
    claims.put("role", role);
    Date now = new Date();

    return Jwts.builder()
        .setClaims(claims)
        .setIssuedAt(now)
        .setExpiration(new Date(now.getTime() + tokenValidMillisecond))
        .signWith(SignatureAlgorithm.HS256, secretKey) // 예외가 발생 한 부분
        .compact();
}
```

> 토큰을 생성하는 코드



```bash
java.lang.ClassNotFoundException: javax.xml.bind.DatatypeConverter
	at java.base/jdk.internal.loader.BuiltinClassLoader.loadClass(BuiltinClassLoader.java:583) ~[na:na]
	at java.base/jdk.internal.loader.ClassLoaders$AppClassLoader.loadClass(ClassLoaders.java:178) ~[na:na]
	at java.base/java.lang.ClassLoader.loadClass(ClassLoader.java:521) ~[na:na]
	at io.jsonwebtoken.impl.Base64Codec.decode(Base64Codec.java:26) ~[jjwt-0.9.1.jar:0.9.1]
	at io.jsonwebtoken.impl.DefaultJwtBuilder.signWith(DefaultJwtBuilder.java:99) ~[jjwt-0.9.1.jar:0.9.1]
	at com.zkdlu.gateway.service.auth.JwtTokenProvider.createToken(JwtTokenProvider.java:39) ~[main/:na]
	at com.zkdlu.gateway.api.auth.SignController.signin(SignController.java:37) ~[main/:na]
	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:na]
	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62) ~[na:na]
	at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:na]
	at java.base/java.lang.reflect.Method.invoke(Method.java:566) ~[na:na]
	at org.springframework.web.method.support.InvocableHandlerMethod.doInvoke(InvocableHandlerMethod.java:190) ~[spring-web-
	....
```

> 발생한 예외 메시지



처음보는 예외메시지다. 

javax.xml.bind.DatatypeConverter 와 ClassNotFoundException을 보면 

XML을 바인딩하여 타입 변경을 하지 못해 발생한 예외로 보인다.



### 어떻게 하였는가?

프로젝트에 jaxb-api  의존성을 추가하여 해결하였다.

```gradle
dependencies {
    implementation 'javax.xml.bind:jaxb-api:2.1'
    implementation 'io.jsonwebtoken:jjwt:0.9.1'
    ...
```



Java 9버전부터 XML과 관련된 모듈이 분리되면서 빠진 모듈인데 이 모듈이 JWT의 암호화와 무슨 관련일까?

