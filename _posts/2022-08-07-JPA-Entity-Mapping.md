---
layout: post
title:  "Entity Mapping"
date:   2022-08-07
image:  /posts/jpa2.jpeg
tags:   SpringBoot JPA 
---

## 엔티티 매핑 기본

- Entity - Table 매핑 : `@Entity`, `@Table`
- Field - Column 매핑 : `@Column`
- PK(식별자) 매핑 : `@Id`
- 연관관계 매핑
    - Table의 FK 매핑 : `@JoinColumn`
    - 다대일 : `@ManyToOne`
    - 일대다 : `@OneToMany`
    - 일대일 : `@OneToOne`
    - 다대다 : `@ManyToMany` (**실무에서 사용 X**, **일대다 + 다대일 로 풀어**냄)

## DB Schema 자동 생성

### 스키마 자동 생성

- **JPA**는 **매핑정보**만 알고 있다면 어떤 Table 인지, 어떤 query를 만들어야 할지 알고 있음
- 그에 따라 JPA는 **Application 로딩 시점에 DB Table을 생성**해주는 기능도 지원 → **스키마 자동 생성** ⇒ 직접 table을 생성하고 설정하고 할 필요가 없음
- 이를 통해 테이블 중심 개발에서 벗어나 **객체 중심으로 개발이 가능**
- **데이터베이스 방언(Dialect)을 활**용하여 각각의 데이터베이스에 맞는 적절한 DDL 생성
- 개발 단계에서 사용하는 것을 권장 (운영 시 사용 X)
- 사용 방법 (JPA설정 파일인 **persistence.xml** 에서 설정 ← Maven)
    - `hibernate.hbm2ddl.auto` 의 속성 설정
    - value (설정 값)
        - **create** : 기존 테이블 삭제 후 다시 생성 (drop and create)
        - **create-drop** : create와 동일하게 동작하나 종료 시점에 테이블 Drop
        - **update** : **변경분만 반영** (**alter Table**)
            - Entity의 Field를 추가하면 alter table로 반영
            - **But, Field를 지우는 것은 반영되지 않음**
            - **[주의] 운영DB에는 사용하면 안됨!**
        - **validate** : 엔티티와 테이블이 **정상 매핑되었는지만 확인**
            - Entity에는 있는 Field가 Table에는 없다면 Error 발생 → **검증**
        - **none** : 사용하지 않음 (속성 설정 부분을 삭제한 경우와 동일)
- DDL 생성 기능의 부가적인 기능
    - J**PA, Application에 영향을 주는 것이 아닌**, 오로지 **DB에만 영향**을 주는 기능 → 생성 시 DB의 Table이나 Column에 **제약조건**을 거는것
    - 즉, DDL을 생성할 때만 사용되고 **JPA의 실행 로직에는 영향을 주지 않는 것**
    - **제약조건** 추가
        - `@Column` 의 **nullable**(필수값), **length**(최대길이 지정)
        - `@Table` 의 **uniqueConstraints** → 유니크 제약조건 추가

### 사용 주의점

- 운영에서는 절대 create, create-drop, update 사용 XXX
- 개발 초기 단계에서는 create 또는 update 사용
- 테스트 서버에서는 update 또는 validate 사용
- 스테이징과 운영서버는 validate 또는 none 사용
- 근데 운영서버에서는 왠만하면 사용하지말자! (none) → 테이블이 다 날라가버리는 위험이 존재!

## 객체 ↔ 테이블 매핑

### @Entity

```java
@Entity
public class Member { ... }
```

- 정의
    - `@Entity` 가 붙은 Class(객체)는 **엔티티**라 부름
    - **JPA가 관리한다**는 뜻
    - JPA를 통해 **Table과 매핑**할 Class는 `@Entity` 필수
- 주의점
    - **기본 생성자 필수** → JPA가 이를 동적으로 활용(프록시 생성 등..)하게 하기 위함
    - final, enum, interface, inner Class 에는 Entity로 설정할 수 없음
    - 저장하고자 하는 Field에 final 사용 불가

### @Table

```java
@Entity
@Table(name = "BASIC_USER")
public class Member { ... }
```

- **엔티티와 매핑할 테이블을 지정**할 때 사용
- 속성
    - **name** : 매핑할 테이블 이름 (기본값 : 엔티티 이름 사용)
    - **catalog** : DB catalog 매핑
    - **schema** : DB schema 매핑
    - **uniqueConstraints** (DDL) : **DDL 생성 시** 유니크 제약 조건 생성

## 필드 ↔ 컬럼 매핑

### @Column

```java
@Column(name="username", unique = true, length = 10, ...)
private String name;
```

- **DB Table의 컬럼 매핑**
- 속성
    - **name**
        - 필드와 매핑할 테이블의 **컬럼 이름** 지정
        - 기본값 : Field 명
        - `name = “username”`
    - **insertable/updatable**
        - **등록/변경 가능 여부**
        - insert/update할 때 해당 column을 등록/변경할 것인지 말것인지 정하는 것
        - 변경되서는 안되는 주요 Field 에서 자주 사용
        - 기본값 : true
        - `updateable = false`
    - **nullable** (DDL)
        - **null 값의 허용 여부** 설정.
        - DDL 생성 시에만 영향을 미치는 속성
        - `nullable = false` → 해당 column은 null이면 안되는 제약조건
    - **unique** (DDL)
        - 한 컬럼에 간단히 **유니크 제약조건**을 걸 때 사용.
        - 잘 사용하지 않는 속성 → 해당 속성 대신 `@Table`의 **uniqueConstraints**를 사용
        - DDL 생성 시에만 영향을 미치는 속성
    - **columnDefinition**(DDL)
        - 데이터베이스 **컬럼 정보를 직접 설정** 가능
        - DDL 생성 시에만 영향을 미치는 속성
        - `columnDefinition = “varchar(50) default ‘NULL’”` → 해당 조건으로 column 생성
        - 기본값 : Field의 Java Type과 Dialect 정보를 사용하여 설정
    - **length** (DDL)
        - **문자 길이 제약**조건
        - Java String Type에만 적용됨
        - 기본값 : 255
        - `length = 20`
    - **precision, scale** (DDL)
        - **아주 큰 숫자나 정밀한 소수를 다룰 때 사용**
        - **BigDecimal 타입**에서 사용 (+ BingInteger)
        - precision은 소수점을 포함한 전체 자릿 수, scale은 소수의 자릿 수
        - double, float 타입에는 적용 X
        - 기본값 : precision=19 | scale=2

### @Enumerated

```java
@Enumerated(EnumType.STRING)
private RoleType roleType;
```

- **Java Enum Type을 Table에 매핑**할 때 사용
- DB 에는 ENUM 타입이 없기 때문에 **대신 String 으로 저장**되지만 **JAVA 안에서는 Enum 같이 사용**할 수 있게 해줌
- **주의! EnumType.ORDINAL 사용X** → ORDINAL은 순서로 저장되기 때문에, 추후 순서가 뒤바뀌거나 새로운 항목이 추가되면 순서가 꼬이게 됨!!!
- 속성
    - **value**
        - EnumType.ORDINAL : enum 순서를 데이터베이스에 저장
        - **EnumType.STRING** : **enum 이름**을 데이터베이스에 저장
        - 기본값 : EnumType.ORDINAL
        - **주의점 : 항상 EnumType.STRING 으로 사용!** 즉, default 말고 항상 EnumType.STRING으로 설정 필요

### @Temporal

```java
@Temporal(TemporalType.TIMESTAMP) // 이전
private Date createDate;

private LocalDate createLocalDate; // 최신
```

- **날짜타입을 매핑**할 때 사용 → 날짜를 어떻게 저장할 것인지를 설정하는 것
- [**참고] Java8 의 LocalDate, LocalDateTime**이 생기며 **현재는 사용하지 않음** → **LocalDate나 LocalDateTime은 따로 매핑정보를 줄 필요가 없음!!**
- 속성
    - **value**
        - TemporalType.DATE: 날짜, 데이터베이스 date 타입과 매핑 (2022-08-01) → Java8 LocalDate
        - TemporalType.TIME: 시간, 데이터베이스 time 타입과 매핑 (11:10:59)
        - TemporalType.TIMESTAMP: 날짜와 시간, 데이터베이스 timestamp 타입과 매핑 (2022-08-01 11:10:59) → Java8 LocalDateTime

### @Lob

```java
@Lob
private String description;
```

- **varchar 의 범위를 넘어서는 큰 용량의 글을 저장**할 때 사용
- 데이터베이스 BLOB, CLOB Type과 매핑
- 속성 없음
- 매핑되는 Field 타입이 문자 → CLOB 과 매핑 (나머지는 BLOB과 매핑)
    - CLOB: String, char[], java.sql.CLOB
    - BLOB: byte[], java.sql. BLOB

### @Transient

```java
@Transient
private int temp;
```

- **DB 에 반영되지 않는, 해당 Field 는 Application 메모리에서만 사용하겠다**는 선언
- 필드 매핑X → 데이터베이스에 저장X, 조회X
- 주로 **메모리상에서만 임시로 어떤 값을 보관하고 싶을 때** 사용

## 기본키(PK) 매핑

### @Id

```java
@Id
private Long id;
```

- **지정 Field를 PK(Primary Key)로 사용**하겠다는 뜻
- **필수 값** → **영속성 컨텍스트(1차 캐시)에서 Map으로 해당 id 값을 key 로 사용하고 Entity 를 value 로 사용하여 저장**하기 때문
- 위와 같이 `@Id` 어노태이션만 사용하면 Id를 직접 할당해 줘야함 (`setId(…)`)
    - `@GeneratedValue` 와 함께 사용 시 자동 생성

### @GeneratedValue

```java
@Id @GeneratedValue(strategy = ...)
private Long id;
```

- 해당 Field의 **id값(PK) 자동 생성** 해주는 어노테이션
- 3가지 전략
    1. **IDENTITY**
        - `@GeneratedValue(strategy = GenerationType.IDENTITY)`
        - **기본 키(PK) 생성을 데이터베이스에 위임** (DB에서 id값을 설정하는 것 → 즉, **DB에 저장이 되어야 Id 값을 알 수 있음**)
        - MySQL, PostgreSQL, SQL Server, DB2 에서 사용 (MySQL → AUTO_ INCREMENT)
        
        <aside>
        ⚠️ <strong>[주의] IDENTITY 전략의 특이점</strong> <br>
        
        < IDENTITY 전략은 <code>em.persist()</code> 시점에 <b>즉시 INSERT SQL 실행하고 DB에서 식별자를 조회</b> >
        <ul>
        <li> 원래의 JPA는 <code>em.persist()</code> 시 바로 DB로 보내지 않고 <b>INSERT query를 “쓰기 지연 SQL 저장소”에 쌓아</b>둔 후 <b>Trascation Commit(flush) 시점에 한번에 모아</b> 보냄 </li>
        <li> IDENTITY 특성 상 <b>키 생성을 DB에 위임</b>하기 때문에 이런 JPA의 특징에서 <b>그 즉시 해당 Entity에 Id 값을 부여하기 힘듦</b> (DB에 저장이 되어야만 Id값을 부여받음) </li>
        <li> 그래서 IDENTITY 전략을 사용하면 <code>em.persist()</code> 시 <b>바로 INSERT SQL을 실행하여 DB에 저장해준 후 해당 객체에 Id 값을 부여</b>해주는 것. </li>
        <li> 성능에 크게 영향을 미치진 않기 때문에 사용해도 됨 </li>
        </ul>   
        </aside>
        
    2. **SEQUENCE**
        
        ```java
        ...
        @SequenceGenerator(
                name = "MEMBER_SEQ_GENERATOR",
                sequenceName = "MEMBER_SEQ", // 매핑할 데이터베이스 시퀀스 이름
                initialValue = 1, allocationSize = 1 )
        public class User {
        	@Id 
          @GeneratedValue(strategy = GenerationType.SEQUENCE,
                          generator = "MEMBER_SEQ_GENERATOR") // 해당 시퀀스 rule 을 따르게 됨
          private Long id;
        }
        ```
        
        - **데이터베이스 시퀀스 오브젝트** 사용
        - `@SequenceGenerator` 필요
            - 만약 **각 테이블마다의 시퀀스를 사용하고 싶을 때** 사용
            - 해당 전략을 사용하면 **Entity 를 저장하기 직전에 DB에 있는 지정된 시퀀스의 다음 값을 가져오게 됨** (**DB로 시퀀스값을 요청**해오는 것) → `SQL: call next value for MEMBER_SEQ`
            - 이름(**name**), 매핑할 DB 시퀀스 이름(**sequenceName**), 시작값(**initialValue** → DDL 생성 시에만 사용), 미리 땡겨올 씨퀀스 크기(**allocationSize**) 지정
            
            <aside>
            ⚠️ <strong>[중요] allocationSize</strong>
            <ul>
            <li> 시퀀스 <b>한 번 호출에 증가하는 수</b> </li>
            <li> 예를 들어 값이 50이면 미리 50개의 ID값을 <b>땡겨와서 메모리 상에서 id 할당</b> → 네트워크적 측면에서의 이득 ( id가 50까지는 다음 시퀀스값을 DB에 요청하지 않아도 되는 것 )</li>
            <li> <b>동시성 문제 X</b> → <b>각각의 서버가 미리 숫자를 allocationSize 값만큼 확보하고 사용하는 시스템</b>이기 때문 </li>
            </ul>
            </aside>
            
            - 사용하지 않으면 기본 시퀀스 사용 → **모든 Table이 공유하는 시퀀스** (ex_**hibernate_sequence**)
        - **데이터베이스 시퀀스**는 **유일한 값을 순서대로 생성**하는 특별한 데이터베이스 오브젝트
        - ORACLE, PostgreSQL, DB2, H2 데이터베이스에서 사용
    3. **TABLE**
        
        ```java
        @Entity
        @TableGenerator(
        			 name = "MEMBER_SEQ_GENERATOR",
        			 table = "MY_SEQUENCES",
        			 pkColumnValue = "MEMBER_SEQ", allocationSize = 1)
        public class Member {
        	 @Id
        	 @GeneratedValue(strategy = GenerationType.TABLE,
        									 generator = "MEMBER_SEQ_GENERATOR")
        	 private Long id;
        }
        ```
        
        - **키 생성 전용 테이블을 하나 만들어서 데이터베이스 시퀀스를 흉내**내는 전략
        - `@TableGenerator` 필요
        - **모든 DB에서 사용 가능**
        - 하지만 성능에 있어서 좋지 않아 많이 쓰지는 않는 전략
    4. **AUTO**
        - **방언에 따라 자동 지정** 
        - ORACLE → SEQUENCE / MYSQL → IDENTITY
        - `@GneratedValue` 기본 값

<aside>
⚠️ <strong>올바른 식별자 전략</strong>
<ul>
<li> <b>비지니스에 사용되는 Field를 키로 선택하면 안됨!</b></li>
<li> 추후까지 고려해보면 결국 <b>대리키(대체키)를 사용하는 것이 좋음</b> → 즉, 비지니스에서 사용되지 않는 <b>별도의 키</b> (ex_ <b>별도의 id Field</b> → <code>private Long id;</code>) </li>
<li> 결론 : <b>식별자</b>는 “<b>Long + 대체키 + 상황에 맞는 strategy</b>” 를 사용하자 </li>
</ul>
</aside>