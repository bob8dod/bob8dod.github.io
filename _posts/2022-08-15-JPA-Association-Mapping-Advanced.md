---
layout: post
title:  "Entity Association Mapping - Advanced"
date:   2022-08-15
image:  /posts/jpa.png
tags:   SpringBoot JPA
---

## 상속관계 매핑

### 상속관계 매핑

- 객체는 상속관계가 존재
- But **관계형 데이터베이스**는 객체에서 사용되는 **상속관계가 사실 없다고 볼 수 있음**
- 하지만 객체 상속과 유사한 개념이 **DB에선 슈퍼타입 서브타입 관계라는 모델링 기법**(논리 모델)으로 존재

![Untitled]({{site.baseurl}}/images/posts/post-220815/Untitled.png)

- 즉, **상속관계 매핑**이란 **객체의 상속 구조와 DB의 슈퍼타입 서브타입 관계를 매핑**하는 것
- 상속관계 매핑 방법
    - **DB의 슈퍼타입 서브타입 논리 모델을 실제 물리 모델로 구현**하는 방법
    1. **각각** 테이블로 변환 → **JOIN 전략** (정석적으로 사용되는 전략)
    2. **통합** 테이블로 변환 → **SINGLE TABLE 전략**
    3. **서브타입** 테이블로 변환 → **TABLE PER CLASS 전략**
    - **[중요] 객체 입장에서는 항상 똑같은 상속관계!**
- 사용 방법
    - 상위 Class → Item Entity
        
        ```java
        @Entity
        @Inheritance(strategy = ... )
        @DiscriminatorColumn(name = "DTYPE")
        public abstract class Item{ // 독단적으로 사용할 일이 없으므로 abstract로 설정
            @Id
            @GeneratedValue
            private Long id;
        		...
        } 
        ```
        
        - `@Inheritance(strategy = InheritanceType.XXX)`
            - **상속 전략** 선택
            - **JOINED** : 조인 전략
            - **SINGLE_TABLE**: 단일 테이블 전략
            - **TABLE_PER_CLASS**: 구현 클래스마다 테이블 전략
            - 해당 **어노테이션 없이** 상속을 진행하면 **자동으로 SINGLE_TABLE로 설정**됨
        - `@DiscriminatorColumn(name = … )`
            - 각각의 **서브타입을 구분하는 Column** 명 지정
            - 기본값 : **DTYPE**
            - 해당 어노테이션이 없다면 구분하는 Column 생성 X (Single Table에선 자동으로 DTYPE으로 설정됨)
            - 필수적인 항목은 아니지만, 있어야지 운영하기에 좋음 → Item이 저장될 때 어떤 하위 Entity로 저장되는지 알 수 있으므로 → 즉 **하위 Entity 없이 Item Entity하나로도 할 수 있는 것들이 많아짐**
            - JOIN 전략에선 필수 X
            - SINGLE TABLE 전략에선 필수! → 고로 SINGLE TABLE에 해당 어노테이션을 달지 않아도 자동으로 DTYPE이 설정됨
    - 하위 Class → Ablum Entity
        
        ```java
        @Entity
        @DiscriminatorValue("A")
        public class Album extends Item{
            private String artist;
        }
        ```
        
        - `extends Item` : Item 상속
        - `@DiscriminatorValue( … )`
            - 슈퍼타입의 구분 Column에 **어떤 값으로 저장될지** 설정
            - 즉, **어떤값으로 구분될 것인지**를 설정
            - 기본값 : Entity 명
        - 다른 하위 Class 인 Moive, Book Entity도 동일

<aside>
⚠️ <strong>관계에 따른 객체와 관계형 데이터베이스의 입장</strong>
<ul>
<li> <b>연관관계 매핑</b> : DB입장에선 동일(TABLE은 항상 동일)! 객체 입장에선 방향성(단,양), 연관관계의 주인을 어떻게 설정하느냐 등에 따라 달라짐! </li>
<li> <b>상속관계 매핑</b> : 객체 입장에선 동일(상속관계는 항상 동일)! DB 입장에선 어떤 전략을 세우냐에 따라 달라짐! → 즉, 상속전략을 어떻게 바꾸든 간에 App단(JAVA)에서 상속을 사용하는 코드는 수정할 필요가 없음! (항상 동일하기 때문) </li> 
</ul> 
</aside>

### JOIN 전략

- 개념

  ![Untitled]({{site.baseurl}}/images/posts/post-220815/Untitled 1.png)
    
    - ITEM 이라는 Table(**상위 Table**) 을 만들고 **각각의 하위 Table** (ALBUM, MOVIE, BOOK)을 만들어 나눈 다음에 **하위 Table을 가져올 때 JOIN으로 구성**하는 것
    - 예를 들어 ALBUM을 설정 한다면 앨범의 NAME, PRICE는 ITEM에 들어오게 되고 ARTIST는 ALBUM에 들어가게 되는 것
    - **저장** 시 INSERT query는 ITEM 1번, ALBUM 1번, **총 2번** 들어가게 됨
    - **조회** 시 **JOIN을 통해** ITEM과 ALBUM을 동시에 가져오게 됨 → JOIN 전략
    - 각각의 하위 Table을 구분하기 위해 ITEM에서 DTYPE을 사용하게 됨
    - 특징 : **하위 Table은 상위 Table과 동일한 id값**을 가지게됨 → **하위 Table에서의 id값은 PK이자 FK!**
- `@Inheritance(strategy = InheritanceType.JOINED)`
    - 저장
        - `em.persist(album)` 를 통해 하위 Entity를 저장
        - **저장 시 Item 1개, Album 1개가 저장**됨 (INSERT query 2개)
        - Item에는 해당 속성에 맞는 값이 저장됨
        - Album에는 해당 Item과 동일한 Id값(PK이자 FK)를 갖게 됨
    - 조회
        - `em.find(Album.class, album.getId())` 를 통해 하위 Entity를 조회
        - **조회 시 Album에 Item을 JOIN** 하여 가져오게 됨 (INNER JOIN query 1개)
        - JOIN을 통해 해당 album 객체를 상속관계로써 정상적으로 사용할 수 있음
- 장점
    - **테이블 정규화** (객체 그래프와 유사 → 유연성, 활용도가 높음)
    - 외래 키 참조 무결성 제약조건 활용 가능! → 상위 Class를 보고자할 때, 하위 Class를 참고해야되는 것이 아닌 **상위 Class의 key만으로도 파악이 가능!**
    - 저장 공간 효율화
- 단점
    - 성능 저하 ← JOIN
    - 조회 쿼리 복잡 ← JOIN
    - 데이터 저장시 INSERT SQL 2번 호출 ← JOIN
    - 그렇게 큰 단점은 아님! (단일테이블에 비해서의 단점)

### SINGLE TABLE 전략

- 개념

  ![Untitled]({{site.baseurl}}/images/posts/post-220815/Untitled 2.png)
    
    - DB의 슈퍼타입 서브타입 논리 모델을 **슈퍼타입의 한 테이블로 표현**한 것 → **서브타입의 Table은 존재X**
    - 각각의 서브타입의 모든 Column을 슈퍼타입 Table에 다 때려 박는 것 → 해당되지 않는 서브타입의 속성에 대해선 **null 발생**
    - **DTYPE을 통해 각 서브타입을 구분** (DTYPE이 필수! → 고로 `@DiscriminatorColumn`이 없어도 자동으로 DTYPE이 생성됨 )
    - 성능측면에서 가장 우수 (JOIN없이 하나의 테이블을 그대로 가져오니)
    - **단순한 모델링일 경우 자주 사용**됨
- `@Inheritance(strategy = InheritanceType.SINGLE_TABLE)`
    - 저장
        - `em.persist(album)` 를 통해 하위 Entity를 저장
        - **저장 시 Item 1개가 저장**됨 (INSERT query 1개)
        - Item에 album에서 지정된 값들이 저장됨
        - **나머지 하위 Entity들에 대한 값들은 null로 설정**됨
    - 조회
        - `em.find(Album.class, album.getId())` 를 통해 하위 Entity를 조회
        - **조회 시 DTYPE을 통해 구분하여 Item의 테이블 하나만 조회**함 (일반 SELECT query 1개) → 성능 UP
        - **Item 테이블의 기본 속성과 album 속성을 빼내서 Album Entity를 만들어줌** (**DTYPE** 을 통해서)
- 장점
    - JOIN을 하지 않으므로 조회 성능이 빠름
    - 조회 query가 단순
- 단점
    - 자식 Entity 가 매핑한 Col은 모두 **null 허용** → DB 입장에서 큰 단점
    - 모든 것을 저장하기에 테이블이 커져 상황에 따라 조회 성능이 오히려 저하될 수 있다

### TABLE PER CLASS 전략

- 개념

  ![Untitled]({{site.baseurl}}/images/posts/post-220815/Untitled 3.png)
    
    - **구현 클래스마다 테이블을 만드는 전략**
    - ITEM TABLE은 만들지 않고 **각각의 서브타입에 해당되는 TABLE들만 생성**
    - 각각의 TABLE이 ITEM TABLE의 모든 COL을 가지고 있는 것
    - 하지만 권장되지 않는 전략 → 부모 Class 로 조회 시 해당 모든 하위 Class를 다 뒤져서 가져옴 **(UNION query 발생!)**
    - 해당 전략은 DB,ORM 두 입장 모두에서 비효율적인 전략 → 쓰지말자
- `@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)`
    - 저장
        - `em.persist(album)` 를 통해 하위 Entity를 저장
        - **저장 시 Item 은 저장되지 않음**
        - **해당 Album Entity만 저장**되게 됨 → Item의 속성 + album 속성을 가진 채로
    - 조회
        - `em.find(Album.class, album.getId())` 를 통해 하위 Entity를 조회
        - **조회 시 Album 테이블 하나만 조회**함 (일반 SELECT query 1개)
    - 즉, Item Table은 생성조차 안되며 사용되지도 않음
- 장점
    - 서브 타입을 명확히 구분할 수 있음
    - NotNull 제약조건 사용 가능
- 단점
    - 여러 자식 테이블을 함께 조회할 때 **UNION SQL이 발생하기에 조회 성능 DOWN**
    - 자식 테이블을 **통합해서 Query 할 시 어려움**
    - 변경이라는 관점에서 안좋은 전략
    

## 매핑 정보 상속 - Mapped Superclass

### @MappedSuperclass

- 개념
    - **공통 매핑 정보**가 필요할 때 사용 → 즉, 공통 속성이 존재할 때 이를 **하나의 객체로 만들고 상속 받아 쓸** 수 있게 해주는 것 [ ex) id, name ]
    - 상속관계 매핑 X
    - Entity X, Table과 매핑되는 것도 X
    - 부모 클래스를 상속 받는 자식 클래스에 **매핑 정보만 제공**
    - Entity, Table이 아니므로 조회 및 검색이 불가 (`em.find()` 불가)
    - 직접 생성해서 사용할일이 없으므로 **추상 클래스** 권장
    - 테이블과 관계 없고, 단순히 엔티티가 **공통으로 사용하는 매핑 정보를 모으는 역할**
    - 주로 등록일, 수정일, 등록자, 수정자 같은 전체 엔티티에서 **공통으로 적용하는 정보를 모을 때 사용**
- 사용
    - 매핑 정보를 담은 Class (Not Entity)
        
        ```java
        @MappedSuperclass // Entity 로 등록 되지 않고, 이를 상속받은 Entity 들이 해당 field 를 사용할 수 있는 것!
        public abstract  class BaseEntity {
            private String createdBy;
            private LocalDateTime createdDate;
            private String lastModifiedBy;
        		private LocalDateTime lastModifiedDate;
        		... //Getter, Setter
        }
        ```
        
        - `@MappedSuperclass` : Entity 로 등록 되지 않고, 이를 상속받은 Entity 들이 해당 field 를 사용할 수 있는 것!
    - 공통 정보를 상속 받는 Class (Entity)
        
        ```java
        @Entity
        public class Member extends BaseEntity{ ... }
        ```
        
        - `extends BaseEntity` : `@MappedSuperclass` 로 지정된 BaseEntity를 상속 받아 해당 **공통 속성을 사용**할 수 있게 됨!
        - 즉, 해당 Entity에 속성을 등록해 놓지 않아도 BaseEntity의 공통 속성들을 사용하고 이를 DB로 부터 저장 및 조회가 가능한 것!

<aside>
⚠️ <strong>[주의] @Entity 클래스의 상속</strong> <br>
<code>@Entity</code> Class는 다른 <code>@Entity</code>나 <code>@MappedSuperClass</code>로 지정한 Class만 상속이 가능!!

</aside>