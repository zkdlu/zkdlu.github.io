---
layout: post
title: "[Trouble shooting] 10.M1에서 발생한 Netty 오류"
description: "Mac M1 Pro에서 발생한 Netty 오류"
date: 2022-01-18 00:00:00
tags: [spring boot, mac]
comments: true
share: true
---

### 환경

- OpenJDK 17
- Spring Boot 2.5.2
- org.springframework.boot:spring-boot-starter-webflux
- 기타 reactive 의존성

---

## 무엇을 했는가?

- 난 webflux 의존성만 추가했을 뿐인...데..

---
### 어떤일이 있었는가?


```bash
java.lang.reflect.InvocationTargetException: null
	at java.base/jdk.internal.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method) ~[na:na]
	at java.base/jdk.internal.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:77) ~[na:na]
	at java.base/jdk.internal.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45) ~[na:na]
	at java.base/java.lang.reflect.Constructor.newInstanceWithCaller(Constructor.java:499) ~[na:na]
	at java.base/java.lang.reflect.Constructor.newInstance(Constructor.java:480) ~[na:na]
	at io.netty.resolver.dns.DnsServerAddressStreamProviders.<clinit>(DnsServerAddressStreamProviders.java:64) ~[netty-resolver-dns-4.1.72.Final.jar:4.1.72.Final]
	at io.netty.resolver.dns.DnsNameResolverBuilder.<init>(DnsNameResolverBuilder.java:60) ~[netty-resolver-dns-4.1.72.Final.jar:4.1.72.Final]
	at io.lettuce.core.resource.AddressResolverGroupProvider$DefaultDnsAddressResolverGroupWrapper.<clinit>(AddressResolverGroupProvider.java:56) ~[lettuce-core-6.1.5.RELEASE.jar:6.1.5.RELEASE]
	at io.lettuce.core.resource.AddressResolverGroupProvider.<clinit>(AddressResolverGroupProvider.java:35) ~[lettuce-core-6.1.5.RELEASE.jar:6.1.5.RELEASE]
	at io.lettuce.core.resource.DefaultClientResources.<clinit>(DefaultClientResources.java:112) ~[lettuce-core-6.1.5.RELEASE.jar:6.1.5.RELEASE]
	at org.springframework.boot.autoconfigure.data.redis.LettuceConnectionConfiguration.lettuceClientResources(LettuceConnectionConfiguration.java:69) ~[spring-boot-autoconfigure-2.5.8.jar:2.5.8]
	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:na]
	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:77) ~[na:na]
	at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:na]
	at java.base/java.lang.reflect.Method.invoke(Method.java:568) ~[na:na]
	at org.springframework.beans.factory.support.SimpleInstantiationStrategy.instantiate(SimpleInstantiationStrategy.java:154) ~[spring-beans-5.3.14.jar:5.3.14]
	at org.springframework.beans.factory.support.ConstructorResolver.instantiate(ConstructorResolver.java:653) ~[spring-beans-5.3.14.jar:5.3.14]
	at org.springframework.beans.factory.support.ConstructorResolver.instantiateUsingFactoryMethod(ConstructorResolver.java:486) ~[spring-beans-5.3.14.jar:5.3.14]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.instantiateUsingFactoryMethod(AbstractAutowireCapableBeanFactory.java:1352) ~[spring-beans-5.3.14.jar:5.3.14]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBeanInstance(AbstractAutowireCapableBeanFactory.java:1195) ~[spring-beans-5.3.14.jar:5.3.14]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:582) ~[spring-beans-5.3.14.jar:5.3.14]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:542) ~[spring-beans-5.3.14.jar:5.3.14]
	at org.springframework.beans.factory.support.AbstractBeanFactory.lambda$doGetBean$0(AbstractBeanFactory.java:335) ~[spring-beans-5.3.14.jar:5.3.14]
	at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:234) ~[spring-beans-5.3.14.jar:5.3.14]
	at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:333) ~[spring-beans-5.3.14.jar:5.3.14]
	at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:208) ~[spring-beans-5.3.14.jar:5.3.14]
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.preInstantiateSingletons(DefaultListableBeanFactory.java:953) ~[spring-beans-5.3.14.jar:5.3.14]
	at org.springframework.context.support.AbstractApplicationContext.finishBeanFactoryInitialization(AbstractApplicationContext.java:918) ~[spring-context-5.3.14.jar:5.3.14]
	at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:583) ~[spring-context-5.3.14.jar:5.3.14]
	at org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext.refresh(ServletWebServerApplicationContext.java:145) ~[spring-boot-2.5.8.jar:2.5.8]
	at org.springframework.boot.SpringApplication.refresh(SpringApplication.java:765) ~[spring-boot-2.5.8.jar:2.5.8]
	at org.springframework.boot.SpringApplication.refreshContext(SpringApplication.java:445) ~[spring-boot-2.5.8.jar:2.5.8]
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:338) ~[spring-boot-2.5.8.jar:2.5.8]
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:1354) ~[spring-boot-2.5.8.jar:2.5.8]
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:1343) ~[spring-boot-2.5.8.jar:2.5.8]
	at com.backend.study.BackendStudyApplicationKt.main(BackendStudyApplication.kt:15) ~[classes/:na]
Caused by: java.lang.UnsatisfiedLinkError: 'io.netty.resolver.dns.macos.DnsResolver[] io.netty.resolver.dns.macos.MacOSDnsServerAddressStreamProvider.resolvers()'
	at io.netty.resolver.dns.macos.MacOSDnsServerAddressStreamProvider.resolvers(Native Method) ~[netty-resolver-dns-classes-macos-4.1.72.Final.jar:4.1.72.Final]
	at io.netty.resolver.dns.macos.MacOSDnsServerAddressStreamProvider.retrieveCurrentMappings(MacOSDnsServerAddressStreamProvider.java:127) ~[netty-resolver-dns-classes-macos-4.1.72.Final.jar:4.1.72.Final]
	at io.netty.resolver.dns.macos.MacOSDnsServerAddressStreamProvider.<init>(MacOSDnsServerAddressStreamProvider.java:123) ~[netty-resolver-dns-classes-macos-4.1.72.Final.jar:4.1.72.Final]
	... 36 common frames omitted
```

> 발생한 예외 메시지



### 어떻게 하였는가?

아래 의존성을 추가해준다. 아직 이유를 잘 모르겠네..

```gradle
implementation(group = "io.netty",name = "netty-resolver-dns-native-macos", version = "4.1.70.Final", classifier = "osx-aarch_64")
```