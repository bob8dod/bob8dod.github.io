---
layout: post
title:  "Querydsl - Basic"
date:   2022-09-17
image:  /posts/querydsl.png
tags:   SpringBoot JPA Querydsl
---

## Querydsl

### Querydsl 이란?

- **정적 타입**을 이용해서 SQL과 같은 쿼리를 생성할 수 있도록 해주는 프레임워크
- JPA나 Spring Data JPA와 같이 문자열로 JPQL을 작성하는 것과는 다르게 **자바코드로써 SQL 쿼리를 작성**할 수 있는 것
- 즉, SQL, JPQL을 자바 코드로 작성할 수 있도록 도와주는 **오픈소스 빌더 API**
- 장점
    - **쿼리를 자바 코드로 작성** 가능 (+ 쉬운 SQL 스타일 문법, 재사용 가능)
    - 그에 따라 문법 오류를 접근 시점이 아닌 **컴파일 시점에 확인 가능** (자바 코드로 작성하기 때문! → 자바 컴파일러가 문법 오류를 잡아줌!)
        - jpql 같은 경우 sql query를 문자열로 작성하기 때문에 컴파일 시점에 오류를 발견할 수 없음! (스프링 데이터 JPA의 `@Query`는 가능)
    - **동적 쿼리 가능**

### 사용해야 되는 이유

- 최신 자바 백엔드 기술은 **스프링 부트 + JPA + 스프링 데이터 JPA** 기술들을 조합해서 사용하는 추세
- 하지만 이런 기술들로는 해결하지 못하는 **한계점** 존재 → **복잡한 쿼리, 동적 쿼리** (실무에서 복잡한 쿼리와 동적 쿼리는 필수적으로 사용됨)
- **Querydsl**을 사용하게 되면 **복잡한 쿼리, 동적 쿼리를 자바 코드로 해결**해 낼 수 있음

### Querydsl 설정 [중요]

- **Querydsl**은 설정이 좀 까다로우며 **버전에 따라 설정이 계속 변하**여 **버전 업데이트 시 항상 설정 부분에 유의**해야 됨
1. **Querydsl 설정 (for Querydsl 5.0.0)**
    - `build.gradle` 에서 Querydsl을 사용할 수 있도록 `buildscript`, `plugins`, `dependencies`, **+별도의 세팅** 을 추가해주어야 함
    - **buildscript** 설정 (**버전** 명시용)
        
        ```java
        buildscript {
        	ext {
        		queryDslVersion = "5.0.0"
        	}
        }
        ```
        
    - **plugins** 추가 (**프레임워크로써 사용**할 수 있게끔 설정)
        
        `id "com.ewerk.gradle.plugins.querydsl" version "1.0.10"`
        
    - **dependecies** 추가 (의존관계 추가 → 직접 **가져와서 사용**할 수 있게끔 설정)
        
        ```java
        implementation "com.querydsl:querydsl-jpa:${queryDslVersion}"
        annotationProcessor "com.querydsl:querydsl-apt:${queryDslVersion}"
        ```
        
        - 이전 버전에서는 annotationProcessor가 필요없었지만, 5.0.0 에선 필요
    - **부가 설정** (**Q-Type을 뽑아내기** 위한 설정)
        
        ```java
        def querydslDir = "$buildDir/generated/querydsl" // 저장 경로
        querydsl {
        	jpa = true
        	querydslSourcesDir = querydslDir
        }
        sourceSets { // src 폴더에 해당 경로를 자동 import -> Q-Type을 쓸 수 있게끔
        	main.java.srcDir querydslDir
        }
        configurations {
        	querydsl.extendsFrom compileClasspath
        }
        compileQuerydsl {
        	options.annotationProcessorPath = configurations.querydsl
        }
        ```
        
    - 전체 build.gradle
        
        ```java
        buildscript {
        	ext {
        		queryDslVersion = "5.0.0"
        	}
        }
        
        plugins {
        	id 'org.springframework.boot' version '2.7.3'
        	id 'io.spring.dependency-management' version '1.0.13.RELEASE'
        	// querydsl 추가
        	id "com.ewerk.gradle.plugins.querydsl" version "1.0.10"
        	id 'java'
        }
        
        group = 'spring'
        version = '0.0.1-SNAPSHOT'
        sourceCompatibility = '17'
        
        configurations {
        	compileOnly {
        		extendsFrom annotationProcessor
        	}
        }
        
        repositories {
        	mavenCentral()
        }
        
        dependencies {
        	implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
        	implementation 'org.springframework.boot:spring-boot-starter-web'
        
        	// querydsl 추가
        	implementation "com.querydsl:querydsl-jpa:${queryDslVersion}"
        	annotationProcessor "com.querydsl:querydsl-apt:${queryDslVersion}"
        
        	// p6spy (qeury log)
        	implementation 'com.github.gavlyukovskiy:p6spy-spring-boot-starter:1.8.0'
        
        	// lombok for main
        	compileOnly 'org.projectlombok:lombok'
        	annotationProcessor 'org.projectlombok:lombok'
        	// lombok for test
        	testCompileOnly 'org.projectlombok:lombok'
        	testAnnotationProcessor 'org.projectlombok:lombok'
        
        	runtimeOnly 'com.h2database:h2'
        	testImplementation 'org.springframework.boot:spring-boot-starter-test'
        }
        
        tasks.named('test') {
        	useJUnitPlatform()
        }
        
        // querydsl 추가 세팅 시작
        def querydslDir = "$buildDir/generated/querydsl" // 저장 경로
        querydsl {
        	jpa = true
        	querydslSourcesDir = querydslDir
        }
        sourceSets { // src 폴더에 해당 경로를 자동 import -> Q-Type을 쓸 수 있게끔
        	main.java.srcDir querydslDir
        }
        configurations {
        	querydsl.extendsFrom compileClasspath
        }
        compileQuerydsl {
        	options.annotationProcessorPath = configurations.querydsl
        }
        // querydsl 세팅 끝
        ```
2. **Querydsl 동작 확인** (**Q-Type 생성 확인**)
    - 먼저 **External Libraries**에서 **querydsl** 관련된 것들이 **implementation** 되었다면 잘 받아와 진 것
    - 그 후 임시 Entity 하나를 생성하고 `Gradle → other → complieQuerydsl` 을 동작시켜 **해당 Entity에 대한 Q-Type**이 지정된 경로(여기선 build/generated/querydsl)에 **생성되는지 확인** **(Hello → QHello)**
        - 터미널로 실행하고자 하면 `./gredlew complieQuerydsl` or `./gredlewJava`
        - 생성된 모든 Q-Type을 제거하고자 하면 `./gredlew clean`
        - 참고로 해당 생성된 파일들은 **git에 올리면 안됨** (build 하위에 생성되게 설정하여 git에 올라가지 않게 함)
        - **[중요] Querydsl을 사용하려면 항상 Entity의 Q-Type이 필요**하기에 **Entity 생성 후 항상 complieQuerydsl을 실행**해 주어야 함
    - 위의 모든 과정이 제대로 동작한다면 Querydsl 사용 가능!
- Querydsl 라이브러리 살펴보기
    - `com.querydsl:querydsl-apt:5.0.0` → **Q-Type 을 generate 하는 용도**의 library
    - `com.querydsl:querydsl-jpa:5.0.0` → **실제 웹 어플리케이션 작성할 때 필요한 library** (jpa 특화용. sql,Mongodb 등 다양하게 지원)

## 사용 도메인

- 엔티티

  ![Untitled]({{site.baseurl}}/images/posts/post-220917/Untitled.png)
    
- ERD(Entity Relationship Diagram)

  ![Untitled]({{site.baseurl}}/images/posts/post-220917/Untitled 1.png)
    
- 연관관계 정의
    - Member ↔ Team
        - 다대일 ↔ 일대다 양방향 관계
    - Member Entity
        
        ```java
        @ManyToOne(fetch = FetchType.LAZY)
        @JoinColumn(name = "team_id")
        private Team team;
        ```
        
        - `@ManyToOne(fetch = FetchType.LAZY)` : Team Entity와 **다대일 관계**를 표시, **지연로딩**을 통해 Proxy객체가 설정될 수 있도록 세팅
        - `@JoinColumn(name = "team_id")` : DB Table의 team_id col과 매핑하여 **PK로 사용**하겠다는 뜻. → **연관관계의 주인** 설정
    - Team Entity
        
        ```java
        @OneToMany(mappedBy = "team")
        List<Member> members = new ArrayList<>();
        ```
        
        - `@OneToMany(mappedBy = "team")` : Member Entity와 **일대다 관계**. 연관관계의 주인이 아니며 Member의 team에 의해 **매핑된 field** 라고 선언

## 기본 문법

### JPAQueryFactory

- **Querydsl을 실행**해주는 역할 (JPA로 치면 **EntityManger**같은 역할)
- 사용법 2가지
    1. **new 오퍼레이션**을 통해 인스턴스 할당
        
        ```java
        @Repository
        public class MemberJPARepository {
        
            @PersistenceContext
            private EntityManager em;
        
            private final JPAQueryFactory queryFactory;
        
            public MemberJPARepository() {
                this.queryFactory = new JPAQueryFactory(em);
            }
        ```
        
        - 원래는 생성자에서 `MemberJPARepository(EntityManger em)` 으로 할 수 있었으나 EntityManger를 JPA에서 주입시키기 때문에 **먼저 JPA가 EntityManger를 주입**(`@PersistenceContext`)시킨 후에 **해당 em 을 JPAQueryFactory에 할당**해주어야 함
    2. **[권장] Bean으로 등록해서 의존성 주입(Dependency Injection)**하여 사용 
        - 최상단 Application(`@SpringBootApplication`)에서 **JPAQueryFactory**를 **Bean**으로 등록
            
            ```java
            @SpringBootApplication
            public class QuerydslApplication {
            
            	@PersistenceContext
            	EntityManager em;
            	
            	@Bean //빈으로 등록 
            	JPAQueryFactory jpaQueryFactory() {
            		return new JPAQueryFactory(em);
            	}
            
            	public static void main(String[] args) {...}
            }
            ```
            
            - 원래는 생성자에서 `MemberJPARepository(EntityManger em)` 으로 할 수 있었으나 EntityManger를 JPA에서 주입시키기 때문에 **먼저 JPA가 EntityManger를 주입**(`@PersistenceContext`)시킨 후에 **해당 em 을 JPAQueryFactory에 할당**해주어야 함
        - **[권장]** 혹은 **Config**(`@Configuration`)에서 **JPAQueryFactory를 Bean으로 등록**
            
            ```java
            @Configuration
            public class QuerydslConfig {
            
                @PersistenceContext
                private EntityManager em;
            
                @Bean //빈으로 등록
                JPAQueryFactory jpaQueryFactory() {
                    return new JPAQueryFactory(em);
                }
            }
            ```
            
        - **[권장]** 마지막으로 **Repository단**에 **의존성 주입**
            
            ```java
            @Repository
            @RequiredArgsConstructor
            public class MemberJPARepository {
            
                private final EntityManager em;
                private final JPAQueryFactory queryFactory;
            		...
            }
            ```
            
            - **JPAQueryFactory가 bean으로 등록**되었기 때문에 **생성자로 주입이 가능**하며, 따라서 `@RequiredArgsConstructor` 를 통해서 **자동 주입 가능!**

<aside>
💡 <strong>JPAQueryFactory를 Bean으로 등록 시 EntityManger의 동시성 문제</strong> <br>
Spring은 여러 Thread에서 동시에 같은 EntityManager에 접근해도, 트랜잭션 마다 <b>별도의 영속성 컨텍스트를 제공</b>하기 때문에, <b>동시성 문제는 걱정하지 않아도 됨</b>

</aside>

### 기본 Q-Type 활용

- **Querydsl을 사용**하기 위해선 항상 **JPAQueryFactory** + **Entity의 Q-Type의 인스턴스**를 이용하여 코드를 작성해주어야 함
- 이런 **Q-Type Class의 인스턴스를 사용하는 방법**은 2가지
- **별칭을 직접 지정**하여 **인스턴스** 생성
    - `QMember qMember = new QMember("m");` → SQL로 치면 as 를 통해 **별칭을 주는** 역할
        - `from QMember as m` 으로 SQL을 짜는 거라고 생각하면 됨
    - **가장 기본**적인 방법
    - **[중요]** 잘 사용되지는 않지만, **같은 테이블(Entity)**에 대한 **Join, SubQuery**를 실행할 때 **각각을 구분하기 위해 사용**됨 → 즉, **인스턴스를 2개 할당**해야 될 때 (같은 인스턴스를 Join하거나 subQuery로 이용할 경우 충돌! ⇒ Querydsl - Advanced 의 subQuery 부분 참고)
- **[권장]** **기본 인스턴스** 사용
    - `QMember qMember = QMember.member;`
    - **자주 사용**되는 방법
    - Q-Type에서 **기본으로 제공하는 인스턴스**를 사용하는 것 → SQL로 치면 as 를 통해 별칭을 자동으로 주도록 하는 것
    - **static import 를 통해 `QMember.member` → `member` 로 사용하는 것을 권장**
- 권장 방법 예시
    
    ```java
    import static study.querydsl.entity.QMember.*;
    
    Member findMember = queryFactory
                .select(*member*) // 원래 QMember.member
                .from(*member*)
                .where(*member*.username.eq("member1"))
                .fetchOne();
    ```
    

### 기본 조회

```java
List<Member> all = queryFactory
            .select(*member*) // QMember.member
            .from(*member*)
            .fetchAll();
```

- JPQL의 기본 조회 `“select m form Member m”` 을 **Querydsl을 통해 자바코드로 작성**
- `.select()`
    - Querydsl 에서 자바코드를 통해 **조회**하는 방법
    - 항상 마지막에는 `fetch(), fetch…()` 을 통해 직접 가져와야 함 (순수 JPA의 `.getResultList()` 등과 같은 역할)
    - `.selectFrom()` 을 통해 **`from` 절과 통합 가능**
    - **distinct**를 사용하고 싶으면 `select()` 바로 뒤에 **체인 형식으로 `distinct()` 적용**해주면 됨 → `.select(…).distinct().from(…)`

> 참고 : Querydsl의 결과로 실행되는 JPQL 확인 방법 → application.yml 에서 `spring.jpa.properties.hibernate.use_sql_comments: true` 설정 추가
> 

### 검색 조건 쿼리

```java
Member result = queryFactory
          .selectFrom(member)
          .where(member.username.eq("member1")
                  .and(member.age.between(10, 20)))
          .fetchOne();
```

- JPQL의 `“where …"`  을 **Querydsl을 통해 자바코드**로 작성
- 검색 조건
    - `member.username.eq("member1")`
        - to JPQL : `m.username = ?1` , `.setParameter(”member1”)`
        - **체인 형식**으로 field에 **조건 및 파라미터 자동 바인딩** 가능
    - `.and(member.age.between(10, 20))`
        - to JPQL : `and m.age between 10 , 50`
        - `.and()` 와 같이 **체인 형식으로 조건 추가 가능** (`.or()` 도 가능)
    - 결과
        - `.where(member.username.eq("member1").and(member.age.between(10, 20)))`
        - to JPQL : `where m.username = ?1 and m.age between 10 and 50`
        - 추가로 `where()` 절의 `.and()` 는 `‘,’`로도 표현 가능 → `.where(member.username.eq("member1") , member.age.between(10, 20))`
            - **해당 파라미터에 null이 들어갈 경우 무시** → `where(member.username.eq("member1"), null)` **⇒ null은 무시**
            - 이 표현식은 **동적 쿼리 적용** 시 굉장히 유용함
- JPQL이 제공하는 모든 검색 조건 제공
    
    ```java
    member.username.eq("member1") // username = 'member1'
    member.username.ne("member1") // username != 'member1'
    member.username.eq("member1").not() // username != 'member1'
    member.username.isNotNull() // name is not null -> 이름이 null이 아닌것만
    member.age.in(10, 20) // age in (10,20)
    member.age.notIn(10, 20) // age not in (10, 20)
    member.age.between(10,30) // between 10 and 30
    member.age.goe(30) // age >= 30
    member.age.gt(30) // age > 30
    member.age.loe(30) // age <= 30
    member.age.lt(30) // age < 30
    member.username.like("member%") // 기본 like 검색 %필수
    member.username.contains("member") // like ‘%member%’ 검색
    member.username.startsWith("member") // like ‘member%’ 검색
    ...
    ```
    

### 결과 조회

```java
List<Member> fetch = queryFactory
	 .selectFrom(member)
	 .fetch(); // .fetchOne(), .fetchFirst(), ...
```

- **결과를 어떻게 가져오냐**에 대한 설정 (순수 JPA로 치면 `.getSingleResult()`, `.getResultList()` 역할)
- `fetch()` : **List 조회**, 데이터 없을 시 빈 리스트 반환
- `fetchOne()` : **단건 조회**
    - 결과 없으면 `null` 반환
    - 결과가 **둘 이상이면 Exception** 발생 (`com.querydsl.core.NonUniqueResultException`)
- `fetchFirst()`
    - 결과들 중 **가장 먼저나오는 결과 하나**를 반환
    - `limit(1).fetchOne()` 과 동일
- `~fetchResults()~`
    - content, count 를 구분하여 query 생성 → 2개의 query가 나감
    - **ver 5.0.0 부터 사용 금지 ⇒ 직접 content 쿼리와 count 쿼리를 구분해서 사용**해야됨
- `~~fetchCount()~~`
    - count query
    - **ver 5.0.0 부터 사용 금지 ⇒ `member.count()` 로 대체하여 사용**해야 됨

### 정렬

```java
List<Member> result = queryFactory
      .selectFrom(member)
      .where(member.age.eq(100))
      .orderBy(member.age.desc(), member.username.asc().nullsLast())
      .fetch();
```

- `.orderBy()` 사용
- 원하는 **필드에 체인형식**으로 `.desc()`, `.asc()` 를 달아 **정렬 기준 선택** 가능 (**정렬 기준 둘 중 하나를 무조건 선택**해주어야 함 → **default 설정이 없음**)
- 추가로 **`nullsLast()`** 를 통해 **null인 항목에 대한 순서 지정** 가능 → **null 이면 마지막**으로 (`nullsFirst()` : **null이면 처음**으로)
- 핵심
    - `desc()` , `asc()` : 일반 정렬 → 정렬을 위해 **필수적인 요소**
    - `nullsLast()` , `nullsFirst()` : **null 데이터 순서** 부여

### 페이징

```java
List<Member> result = queryFactory
          .selectFrom(member)
          .orderBy(member.age.asc())
          .offset(1) // 1번째 데이터부터 (0부터 시작)
          .limit(2) // 최대 2건 가져오기
          .fetch();
```

- **offset** 과 **limit** 을 통해 페이징 동작
- `.offset()` : **시작 데이터** 설정 (몇번째 데이터부터 가져올 것인지. **0부터 시작**)
- `.limit()` : 가져올 **데이터 개수** 설정 (몇개를 가져올 것인지)

<aside>
💡 <strong>Querydsl의 버전 변경에 따른 페이징 주의!</strong> <br>
원래는 <code>.fetchResults()</code> 를 통해서 페이징 정보를 얻어올 수 있지만, <b>ver 5.0.0 부터 <code>.fetchResults()</code> 를 사용하지 못하</b>므로 <b>직접 정보를 얻어오기 위한 쿼리를 동작</b>해주어야 함
</aside>

### 집합 함수

```java
Tuple tuple = queryFactory
            .select(
                    member.count(), // 개수
                    member.age.sum(), // 합
                    member.age.min(), // 최소값
                    member.age.max(), // 최대값
                    member.age.avg() // 평균
            )
						.from(member).fetchOne();
```

- SQL에서 지원하는 **대부분의 함수 사용 가능**
- JPQL과는 다르게 **field에 체인형식으로 적용**
- `Tuple`
    - **단일 객체의 결과가 아니기 때문에 Tuple로 반환**됨
    - `get()` 을 통해 **값 접근** 가능 → `tuple.get(*member*.count())`
    - Querydsl - Advanced 파트 참고

### 그룹화

```java
List<Tuple> result = queryFactory
                .select(team.name, member.age.avg())
                .from(member)
                .join(member.team, team)
                .groupBy(team.name)
                .fetch();
```

- **groupBy**를 통해서 **grouping** 진행
    - ex) member에 team을 join 시킨 후 team의 name으로 그룹화를 시킨 후 (teamA : {member1, member2}, teamB : {…}) member의 age를 평균내어 반환 → [{teamA, 20},{teamB,30}]
- `.groupBy()`
    - 그룹화하고자 하는 field 부여
    - `.having()` 도 사용 가능 → **그룹화된 결과 필터링**
        
        ```java
        .groupBy(team.name)
        .having(team.name.eq("teamA"))
        ```
        

### 조인

- 내부 조인(inner join) : `.join()` , `.innerJoin()`
- left 외부 조인(left outer join) : `.leftJoin()`
- right 외부 조인(right outer join) : `.rightJoin()`
- **기본 조인**
    
    ```java
    List<Member> result = queryFactory
    		 .selectFrom(member)
    		 .join(member.team, team)
    		 .where(team.name.eq("teamA"))
    		 .fetch();
    ```
    
    - `join(`조인 대상`,` 조인할 Q타입`)`
        - 다른 테이블 조인 → `.join(member.team, team)` ⇒ `join Team t on m.team_id = t.id`
        - **같은 테이블 조인**
            - **새로운 별칭(구분을 위해)**으로 **Q-Type 생성** → `QMember sub = new QMember(”sub”)`
            - 그 후 join → `.join(member, sub)` ⇒ `join Member sub on m.id = sub.id`
- **세타 조인** (막 조인)
    
    ```java
    List<Member> result = queryFactory
    			 .select(member)
    			 .from(member, team) // 세타 조인 (inner join. 외부 조인 불가능)
    			 .where(member.username.eq(team.name))
    			 .fetch();
    ```
    
    - **연관관계가 없는 필드로 조인** → `cross join`
    - 일명 **막조인**
    - **외부 조인 불가능** (hibernate를 사용하면 **Join on절을 통해 가능**)
        - null 일 발생하거나 이런 Exception 때문이 아닌, **from()을 통한 세타조인에서 지원을 안하는 것**
        - 하이버네이트 5.1부터 **on 을 사용해서 서로 관계가 없는 필드로 외부 조인하는 기능이 추가**
    - `.from(member, team)`
        - from 절에 여러 엔티티를 선택해서 세타 조인
        - JPQL : `select m from Member m, Team team`
        - SQL : `selet …. from member m **cross join team t**`
        - `.from(member).join(team)` 으로도 **막조인** 가능 (`join(member.team, team)`이 아닌 그냥 **조건 없이 team을 join** 하는 것)
    - member의 **row X team**의 row 의 결과가 생성됨 (where 전)
- **ON절**
    - ON절을 활용한 조인(JPA 2.1부터 가능)
    - 조인 후 **where을 통한 필터링이 아닌**, **조인 하는 시점에 on을 통해 필터링** 하고자 할 때 사용  (ex. 회원과 팀을 조인하면서, 팀 이름이 **teamA인 팀만 조인**, 회원은 모두 조회)
        
        ```java
        List<Tuple> result = queryFactory
        			 .select(member, team)
        			 .from(member)
        			 .leftJoin(member.team, team).on(team.name.eq("teamA"))
        			 .fetch();
        ```
        
        - JPQL: `SELECT m, t FROM Member m LEFT JOIN m.team t **on t.name = 'teamA'**`
        - SQL: `SELECT m.*, t.* FROM Member m LEFT JOIN Team t ON m.TEAM_ID=t.id **and t.name='teamA'**`
        - **필터링 시 outer join을 사용하고자 할때 on은 필수** → where을 통해 join 후 필터링이 불가능( ← Null) ⇒ 그래서 **on을 통해 join 하는 시점에 필터링을 진행**하는 것 (**join할 것을 가져올때 필터링**함)
        - 필터링 시 **inner join을 사용하고자 할 때는 on은 필수가 아님** → **where을 통해 join 후 필터링 가능** ⇒ `.join(member.team, team).where(team.name.eq("teamA"))`
        
        <aside>
        💡 <strong>핵심 정리</strong>
        <ul>
        <li> <b>inner join + where</b> 을 할 경우에는 모든 회원을 조회할 수 없음 (null이 제외됨) </li>
        <li> <b>outer join + where</b> 은 조건을 만족시킬 수 없음 → <b>join 후 team이 null인 member에서 team.name을 조회할 수 없</b>으므로 </li>
        <li> 해결 : <b>outer join + on</b> 으로 <b>조건을 만족시키는 team에 대해서만 join을 진행</b>하는 것 → sql에서의 <code>on __ and __</code> </li>
        <li> 결국 <b>inner join 을 사용하면 on 을 사용하든 where을 사용하든 똑같</b>음 → 익숙한 where을 사용하자 </li>
        <li> But <b>outer join 은 where이 먹히지 않으므로 절대 on 사용!</b> </li>
        </ul>
        </aside>
        
    - **연관관계가 없는 엔티티를 외부 조인**하고자 할 때 사용 (**세타외부조인**. ex. 회원의 이름과 팀의 이름이 같은 대상 외부 조인)
        
        ```java
        List<Tuple> result = queryFactory
              .select(member, team)
              .from(member)
              .leftJoin(team).on(member.username.eq(team.name)) // 주의 : 연관관계가 없기때문에 leftJoin()에 team Entity 하나만 들어감
              .fetch();
        ```
        
        - JPQL: `SELECT m, t FROM Member m LEFT JOIN Team t on m.username = t.name`
        - SQL: `SELECT m.*, t.* FROM Member m LEFT JOIN Team t ON m.username = t.name`
        - 하이버네이트 5.1부터 **on 을 사용해서 서로 관계가 없는 필드로 외부 조인하는 기능이 추가**됨
        - join 할 때, **해당 Member의 이름과 동일한 Team만 join**이 되는 것. **join되지 않은 Member도 반환 (외부 조인)**
        - `leftJoin(team)`
            - 연관관계가 없기때문에 `leftJoin()`에 **team Entity 하나만** 들어감
            - 일반조인: `leftJoin(member.team, team)` ↔ 세타 on 조인: `from(member).leftJoin(team).on(xxx)`
- **Fetch Join**
    
    ```java
    Member findMember = queryFactory
    		 .selectFrom(member)
    		 .join(member.team, team).fetchJoin()
    		 .where(member.username.eq("member1"))
    		 .fetchOne();
    ```
    
    - `join()`, `leftJoin()` 등 조인 기능 뒤에 `fetchJoin()` 이라고 추가
        - join 뒤에 `fetchJoin()` 설정을 통해 가능
        - member만 조회했지만, member와 **지연로딩으로 연관된 team까지 member에 넣어**주는 것
    - `select(member)…`
        - 일반 조인 → `select m.*`
        - **패치 조인** → `select m.*, t.*`
    - 조인을 활용해서 **연관된 엔티티를 SQL 한번에 조회**하는 기능 (SQL에서 제공하는 기능은 아님)
    - 지연 로딩의 연관 Entity를 마치 **즉시로딩처럼 한번에 가져오**는 기능 (**1+N 문제 해결 가능**)

### 서브 쿼리

```java
QMember memberSub = new QMember("memberSub");

List<Member> result = queryFactory
          .selectFrom(member)
          .where(member.age.eq( // goe 등 다양하게 사용 가능
                  JPAExpressions
                          .select(memberSub.age.max()) // avg 가능
                          .from(memberSub)
          ))
          .fetch();
```

- `com.querydsl.jpa.JPAExpressions` 사용
    - `JPAExpressions` 뒤에 보통 Querydsl 짜듯이 **select, from 을 체인형식으로 작성**해주면 됨
    - `JPAExpressions.*` 를 **static import** 하여 바로 `.select()` 로도 사용 가능
- `QMember memberSub = new QMember("memberSub");`
    - 서브쿼리 사용 시 **똑같은 인스턴스를 서브쿼리에 넣으면 충돌** 발생 → 구분되는 **새로운 인스턴스 생성** 필요
    - SQL로 치면 **as 를 통해 별칭을 주는 역할** (`from QMember as m` 으로 SQL을 짜는 거라고 생각하면 됨)
    - 결과적으로 `QMember as member1, QMember as memberSub` **두가지 인스턴스를 활용**하는 것
- `where()` 절 안에 사용하는 것 뿐만 아니라 **`select()` 절 안에서도 사용 가능**
    
    ```java
    List<Tuple> result = queryFactory
              .select(member.username,
                      JPAExpressions
                              .select(memberSub.age.avg())
                              .from(memberSub)
              ).from(member)
              .fetch();
    ```
    
    - **하이버네이트 구현체에서 가능**한 것
- But, **from절에선 사용 불가**능! → JPA JPQL에서 지원하지 않으므로
    - 서브쿼리를 **join으로 변경**하여 해결하거나 **쿼리를 2번 분리해서 실행**하는 등의 방법으로 해결해야 됨 (웬만하면 대안 가능)

### Case 문

- **select, 조건절(where), order by**에서 사용 가능
- 간단 Case절
    
    ```java
    List<String> result = queryFactory
                    .select(member.age
                            .when(10).then("열살")
                            .when(20).then("스무살")
                            .otherwise("기타"))
                    .from(member)
                    .fetch();
    ```
    
    - **filed에 체인형식으로 case절 작성**
- 복잡한 Case조건
    
    ```java
    List<String> result2 = queryFactory
                .select(new CaseBuilder()
                        .when(member.age.between(0, 20)).then("0~20살")
                        .when(member.age.between(21, 30)).then("21~30살")
                        .otherwise("기타"))
                .from(member)
                .fetch();
    ```
    
    - `CaseBuilder` 에 **체인형식**으로 case절 작성
- **[중요]** **Case문 + Orderby**
    
    ```java
    NumberExpression<Integer> rankPath = new CaseBuilder()
    		 .when(member.age.between(0, 20)).then(2)
    		 .when(member.age.between(21, 30)).then(1)
    		 .otherwise(3);
    
    List<Tuple> result = queryFactory
    		 .select(member.username, member.age, rankPath)
    		 .from(member)
    	   .orderBy(rankPath.desc())
    	   .fetch();
    ```
    
    - Case를 통해 **Sort 기준을 줄 때** 사용
    - Querydsl은 **자바 코드**로 작성하기 때문에 rankPath 처럼 **복잡한 조건을 변수로 선언해서 select 절, orderBy 절에서 함께 사용**

### 상수 추가

```java
List<Tuple> addString = queryFactory
          .select(member.username, Expressions.constant("A"))
          .from(member)
          .fetch();
```

- `Expressions.constant("A")`
    - `“A”` 라는 상수를 `member.username` 와 함께 **결과에 추가**
    - 결과 : [{”member1”, “A”}, {”member2”, “A”}, … ]
    - constant 를 사용하게 되면 SQL에서 A를 생성해서 가져오는 것이 아니라 **SQL에서 member.username을 가져온 결과에 “A”를 추가**하는 것 → **최적화**

### 문자 더하기

```java
String concat = queryFactory
          .select(member.username.concat("_").concat(member.age.stringValue())) // : member.age.stringValue() 부분이 중요 (int -> String)
          .from(member)
          .where(member.username.eq("member1"))
          .fetchOne();
```

- `.concat()` : 애초에 **지정문자를 합쳐서 조회**해 오는 것
- `.stringValue()` : **문자로 만들어주는 기능** (중요: **ENUM 처리 시 자주 사용!**) → SQL의 cast() 역할
- 결과 : member1_10