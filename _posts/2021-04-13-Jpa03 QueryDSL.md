---
layout: post
title: "[Jpa] QueryDSL"
description: "Spring Data JPA에서 QueryDSL 사용해보기"
date: 2021-04-13 00:00:00
tags: [spring boot, jpa, querydsl]
comments: true
share: true
---

# QueryDSL

정적 타입을 사용해 JPQL을 생성할 수 있는 프레임워크이다. JPA를 사용하여 여러 조건에 해당하는 조회 쿼리를 작성하려면 findByIdAndNameAnd... 등의 메서드를 생성하거나 @Query 어노테이션을 사용하여 직접 JPQL을 작성하여야 한다.

메서드 방식은 동적으로 조회 쿼리를 생성하는데 어려움이 있고, JPQL은 문자열을 기반으로 하기 떄문에 런타임 시점에 쿼리가 잘못되었는지 알 수 있음.

QueryDSL은 Q클래스를 사용해 타입에 안정적인 방식으로 쿼리를 **동적**으로 생성할 수 있다.



## 환경

- OpenJDK 11
- Spring boot 2.3.9.RELEASE

**의존성**

```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    runtimeOnly 'com.h2database:h2'
    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'

    compile("com.querydsl:querydsl-core")
    compile("com.querydsl:querydsl-jpa")
    annotationProcessor "com.querydsl:querydsl-apt:${dependencyManagement.importedProperties['querydsl.version']}:jpa"
    annotationProcessor("jakarta.persistence:jakarta.persistence-api")
    annotationProcessor("jakarta.annotation:jakarta.annotation-api")
}
```

> 아직 구체적으로 어떤 의존성인지 모르겠다.



### 1. Entity 생성

```java
@Getter
@NoArgsConstructor
@Entity
public class Person {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private long id;
    private String name;

    @Builder
    public Person(String name) {
        this.name = name;
    }
}
```

### 2. QClass 생성을 위한 gradle task 작성

```groovy

def generated='src/main/generated'
sourceSets {
    main.java.srcDirs += [ generated ]
}

tasks.withType(JavaCompile) {
    options.annotationProcessorGeneratedSourcesDirectory = file(generated)
}

clean.doLast {
    file(generated).deleteDir()
}
```

### 3. Gradle Task에 other하위에 있는 compileJava를 실행

src/main/generated 디렉토리에 QPerson 클래스가 생성된다.

```java
@Generated("com.querydsl.codegen.EntitySerializer")
public class QPerson extends EntityPathBase<Person> {

    private static final long serialVersionUID = 2074619141L;

    public static final QPerson person = new QPerson("person");

    public final NumberPath<Long> id = createNumber("id", Long.class);

    public final StringPath name = createString("name");

    public QPerson(String variable) {
        super(Person.class, forVariable(variable));
    }

    public QPerson(Path<? extends Person> path) {
        super(path.getType(), path.getMetadata());
    }

    public QPerson(PathMetadata metadata) {
        super(Person.class, metadata);
    }

}
```



### 4. QueryDSL Config파일 작성

```java
@Configuration
public class QuerydslConfiguration {
    @PersistenceContext
    private EntityManager entityManager;

    @Bean
    public JPAQueryFactory jpaQueryFactory() {
        return new JPAQueryFactory(entityManager);
    }
}
```



### 5. 조회용 QueryDSL Repository 작성

```java
import static com.zkdlu.querydsl.domain.QPerson.person;

@RequiredArgsConstructor
@Repository
public class PersonQueryRepository {
    private final JPAQueryFactory queryFactory;

    public List<Person> findByName(String name) {
        return queryFactory.selectFrom(person)
                .where(person.name.eq(name))
                .fetch();
    }
}
```



### 6. 테스트

```java
@ExtendWith(SpringExtension.class)
@SpringBootTest
class PersonQueryRepositoryTest {
    @Autowired
    private PersonRepository personRepository;
    @Autowired
    private PersonQueryRepository personQueryRepository;

    @Test
    @DisplayName("QueryDSL 테스트")
    public void queryDsl_Test() {
        //given
        String name = "geon";
        personRepository.save(new Person(name));

        //when
        List<Person> result = personQueryRepository.findByName(name);

        //then
        assertThat(result.size()).isEqualTo(1);
    }
}
```

조회 로그

```
Hibernate: select person0_.id as id1_0_, person0_.name as name2_0_ from person person0_ where person0_.name=?
```



### 동적 조회 조건 생성하기

QueryDSL의 BooleanExpression을 사용하면 동적으로 Where 조건을 생성할 수 있다. 

QueryDSL의 where  조건에 ,(쉼표)  로 구분하여 파라미터를 넘겨주면 And와 함께 조건을 생성하고 만약 파라미터로 null이 올 경우 조건에서 제외를 한다.

```java
import static com.zkdlu.querydsl.domain.QPerson.person;

@RequiredArgsConstructor
@Repository
public class PersonQueryRepository {
    private final JPAQueryFactory queryFactory;

    public List<Person> findDynamic(String id, String name) {
        return queryFactory.selectFrom(person)
                .where(equalId(id)), 
                        equalName(name))
                .fetch();
    }
    
    private BooleanExpression equalId(String id) {
        if (StringUtils.isEmpty(id)) {
            return null;
        }
        return person.id.eq(id);
    }
    
    private BooleanExpression equalName(String name) {
        if (StringUtils.isEmpty(name)) {
            return null;
        }
        return person.name.eq(name);
    }
}
```

BooleanBuiler를 이용한 and(), or() 메서드를 사용하여 조건들을 추가할 수 도 있다.