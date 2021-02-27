---
layout: post
title: "[Jpa] Spring Data Jpa"
description: "Spring Data Jpa 사용하기"
date: 2021-02-26 00:00:00
tags: [spring boot, jpa]
comments: true
share: true
---



# JPA

Java Persistence API. 자바 ORM 기술 표준.

자바 어플리케이션에서 관계형 데이터베이스를 사용하는 방식을 정의한 인터페이스로 구현체로는 Hibernate가 있다.

SQL 중심적 개발에서 객체를 중심으로 개발 할 수 있다.



객체 지향과 관계형 데이터베이스의 패러다임이 불일치하면서 발생하는 문제를 해결 하고 값을 DB에 저장하는 것을 컬렉션에 저장하듯이 객체를 저장할 수 있다.



## 최적화

1차 캐시와 동일성 보장

> 같은 트랜잭션 안에서는 같은 엔티티를 반환
>
> DB Isolation Level이 Read Commit이어도 어플리케이션에서 Repeatable Read 보장

트랜잭션을 지원하는 쓰기 지연

> 트랜잭션이 커밋 될 때까지 INSERT SQL을 모아두고 JDBC BATCH SQL을 이용해 한번에 저장한다.

지연 로딩

> 지연로딩 : 객체가 실제 사용될 때 로딩
>
> 즉시로딩 : JOIN SQL로 한번에 연관된 객체까지 미리 조회



### 주요 어노테이션

> @Entity : JPA가 관리할 객체
>
> @Id: DB PK와 매핑할 필드
>
> @GeneratedValue
>
> >- IDENTITY : 데이터베이스에 위임 (MYSQL)
> >- SEQUENCE : 데이터베이스 시퀀스 오브젝트 사용 (ORACLE)
> >- TABLE : 키 생성용 테이블 사용
> >- AUTO : Dialect에 따라 자동 지정
>
> @Column
>
> > - name : 필드와 매핑할 테이블 컬럼명
> > - length : 길이
> > - insertable, updateable : 읽기전용
> > - nullable : null 허용 여부
> > - unique : 유니크 제약 조건
>
> @Temporal
>
> > 날짜 타입 매핑
>
> @Enumerated
>
> > 열거형 타입 매핑
> >
> > Enum을 필드로 가질 때 사용하는데 Ordinal과 String으로 설정할 수 있는데 Ordinal의 경우 enum 값 중간에 새로운 값이 생길 경우 기존 데이터의 전부 꼬이게 된다. 무조건 STRING으로 해야한다.
>
> @Lob
>
> > CLOB (String, char[]) , BLOB (byte[]) 매핑 
>
> @Trasient
>
> > 이 필드는 매핑하지 않고 DB에 저장하지 않는다.

```java
@Entity
public class Member {
	@Id @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
    @Column(name = "USERNAME")
    private String name;
    private int age;
    @Temporal(TemporalType.TIMESTAMP)
    private Date created;
    @Enumerated(EnumType.STRING)
    private MemberType memberType;
}
```


### Dialect 

SQL 표준을 지키지 않거나 특정 데이터베이스만의 고유한 기능

JPA에서 특정 데이터베이스에 종속되지 않기 위해 사용한다.



## 사용법

- 저장: jpa.persist(obj);
- 조회: T obj = jpa.find(id);
- 수정: obj.setName("new name");
- 삭제: jpa.remove(obj);



1. EntityManagerFactory 설정

2. EntityManager 설정

3. Transaction

4. BusinessLogic

> EntityManagerFactory는 하나만 생성해서 어플리케이션 전체에서 공유
>
>  EntityManager는 스레드간 공유 하면 안됨. 한 트랜잭션에서 사용하고 제거
>
>  JPA의 모든 데이터 변경은 트랜잭션 안에서 실행해야 함



```java
public void test() {
    EntityManagerFactory emf = Persistence.createEntityManagerFactory("unit name");
    EntityManager ef = emf.createEntityManager();
    EntityTransaction tx = em.getTransaction();

    tx.begin();
    
    try {
        Member member = getMember();
        em.persist(member);
        
        tx.commit();
    } catch (Exception e) {
        tx.rollback();
    } finally {
        em.close();
    }
    emf.close();
}
```



# Spring data JPA

Spring에서 제공하는 JPA를 쓰기 쉽게 만들어 둔 모듈



EntityManager를 직접 다루지 않고 Repository인터페이스를 사용하면 미리 정해진 규칙대로 메서드를 입력하면, 해당 메서드 이름에 적합한 쿼리를 날리는 구현체를 만들어 Bean에 등록 됨



Spring Data JPA의 Repository 구현체인 SimpleJpaRepository 클래스는 내부적으로 EntityManager를 사용하고 있다.



![jpa](https://zkdlu.github.io/images/jpa/jpa.png)



## 사용법

환경

> OpenJDK 11
>
> Spring Boot 2.3.8
>
> H2



1. 의존성 추가

   ```gradle
   dependencies {
       implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
       runtimeOnly 'com.h2database:h2'
   }
   ```

2. 데이터 소스 설정

   ```properties
   # 기본 설정 (h2, hsql의 메모리 db는 아무 설정 안해도 됨)
   spring.datasource.url=jdbc:h2:tcp://localhost/~/test
   # spring.datasource.url=jdbc:[db]://[host]:[db-name]
   spring.datasource.username=zkdlu
   spring.datasource.password=1234
   
   # 생성되는 sql 확인용
   spring.jpa.hibernate.ddl-auto=update
   spring.jpa.show-sql=true
   spring.jpa.properties.hibernate.format_sql=true
   
   # ? 로 출력되는거 로깅으로 확인하기
   logging.level.org.hibernate.SQL=debug
   logging.level.org.hibernate.type.descriptor.sql=trace
   ```

    > 데이터베이스 스키마 자동 생성 spring.jpa.hibernate.ddl-auto
    >
    > - create : 기존 테이블 삭제 후 다시 생성
    > - create-drop : create와 동일. 종료 시점에 테이블 drop
    > - update : 변경된 것만 반영
    > - validate : 엔티티와 테이블이 정상 매핑되었는지만 확인
    > - none : 사용 안함
   
3. Repository 생성

   ```java
   public interface MemberRepository  extends JpaRepository<Member, Long> {
   }
   ```

   > JpaRepository 구현체에 이미 @Repository어노테이션이 달려있음

4. 테스트

   ```java
   public class JpaTestApplication implements ApplicationRunner {
       ....
           
       @Autowired
       private MemberRepository memberRepository;
   
       @Override
       public void run(ApplicationArguments args) throws Exception {
           Member member = getMember();
   
           memberRepository.save(member);
       }
   }
   
   ```



# 연관관계

