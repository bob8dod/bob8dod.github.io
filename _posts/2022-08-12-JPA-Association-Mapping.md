---
layout: post
title:  "Entity Association Mapping - Basic"
date:   2022-08-12
image:  /posts/jpa2.jpeg
tags:   SpringBoot JPA
---

## 연관관계 매핑시 고려해야될 사항

### 다중성

- 다대일, 일대다, 일대일, 다대다(실무X)
- **DB 관점에서의 다중성**을 고려하면 됨
- 다중성의 관계가 헷갈릴 때는 반대편으로 생각해내면 됨 → **대칭성**이 있기 때문 (ex_회원과 팀의 관계가 헷갈릴 때는 팀과 회원의 관계로 생각해내면 됨)

### 방향성

- 단방향, 양방향
- **테이블은 외래 키 하나로** 양방향(양쪽 JOIN) 가능 → **방향이라는 개념이 없음**
- 하지만 객체는 **Reference Field**가 있는 쪽으로만 참조 가능
    - 한쪽만 참조하면 **단방향**
    - 양쪽이 서로 참조하면 **양방향**(← 단방향 2개)

### 연관관계의 주인 (+ mappedBy)

- 테이블은 외래 키 하나로 두테이블이 연관관계를 맺음
- 객체의 양방향 관계는 A→B, B→A 처럼 참조가 2군데가 있음
- 여기서 딜레마가 발생 → **객체는 참조가 2군데인데 테이블의 외래키는 하나** → 즉, 둘중 하나만 이 외래키를 관리해야 되는 것.
- 그럼 누가 관리하나? → **연관관계의 주인**
- **연관관계의 주인** : **외래 키를 관리하는 참조(Reference)** → 보통 FK가 존재하는 Table의 Entity 로 설정 (`@JoinColumn`)
- **주인의 반대편** : 외래 키에 영향을 주지 않으며 단순히 조회만 가능 (`mappedBy`)

## 연관관계 종류

### 다대일 [N:1] (@ManyToOne)

- 다대일 관계
    - 가장 많이 사용하는 연관관계
    - 다대일(N:1)의 반대쪽은 일대다(1:N)
- **단방향** 다대일 (`@ManyToOne`)
    
    ```java
    // Member Entity (N:1 :: N)
    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;
    ```
    
    - RDB 설계상 항상 다대일의 **‘다’쪽에 외래키** 존재
    - 그러므로 항상 **Many 쪽이 연관관계의 주인**이 됨
    - `@JoinColumn(name = "TEAM_ID")` 을 통해 **Table의 FK와 매핑**
    - **FK 변경 및 등록 등의 권한을 가지고 있는 연관관계의 주인**
- **양방향** 다대일 (+ `@OneToMany`)
    
    ```java
    // Team Entity (N:1 :: 1)
    @OneToMany(mappedBy = "team") // 연관이 되어버린 놈. 주인이 아님! 당한 놈
    private List<Member> members = new ArrayList<>();
    ```
    
    - 단방향에서 **양방향으로 전환**된다 해도 **테이블 변경 X**
    - 그저 다대일 중 **‘일’에 참조가 추가**되는 것
    - 이때 `@OneToMany` 를 통해 해당 Entity와의 관계가 현재 Entity의 입장에서 일대다 라는 것을 알려주고
    - **연관관계의 주인이 아닌, 매핑당하고 있다**는 `mappedBy` 설정
    - **조회만** 가능

### 일대다 [1:N] (@OneToMany)

- 일대다 관계
    - 다대일에서 연관관계의 **주인이 반대로 설정**된 경우
    - 객체와 테이블의 차이 때문에 **반대편 테이블의 외래 키를 관리하는 특이한 구조** → 잘 사용하는 관계는 아님 → 되도록 다대일을 사용하자!
    - 원래는 일대다에서 “다”쪽에 외래키가 있지만, 다대일에서의 방식과는 다르게 “다”쪽이 아닌 **“일”쪽에서 외래키를 관리**하는 것 → 연관관계의 주인을 “일” 쪽으로 설정한 것
    - 즉, **“일”쪽에서 값이 변할 때 “다”쪽의 외래키가 변함** → 지금 변경하고 있는 Entity가 아닌 다른 Table에 UPDATE query를 날리게 되는 것 (추후 유지보수 및 최적화에 있어서 큰 단점이 될 수 있음)
- **단방향** 일대다 (`@OneToMany`)

  ![Untitled]({{site.baseurl}}/images/posts/post-220812/Untitled.png)
    
    ```java
    // Team Entity (1:N :: 1)
    @OneToMany
    @JoinColumn(name = "TEAM_ID")
    private List<Member> members = new ArrayList<>();
    ```
    
    - `@OneToMany` : 해당 Entity와의 관계 (일대다)
    - `@JoinColumn(name = “TEAM_ID”)`
        - 해당 Entity(Team)에서 **외래키를 관리하겠다**는 뜻
        - 고로 **Member Table의 FK인 TEAM_ID 와 매핑**하겠다는 것
        - **연관관계의 주인**을 가지게 됨
    - **참고**
        - Team을 생성하고 Team의 members에 Member를 추가하게 되면
        - Team Table에 INSERT query가 하나 나가고
        - **Member Table에 UPDATE query가 하나 나감.** (**외래키(FK)를 Member가 가지**고 있기 때문)
        - 즉, **다대일과는 다르게 query가 하나 더 나감**
- **양방향** 일대다 (+ **@ManyToOne, @JoinColumn**)

  ![Untitled]({{site.baseurl}}/images/posts/post-220812/Untitled 1.png)
    
    ```java
    // Member Entity (1:N :: N)
    @ManyToOne
    @JoinColumn(name = "TEAM_ID", insertable = false, updateable = false)
    private Team team;
    ```
    
    - `@ManyToOne` : 해당 Entity와의 관계를 알려주는 것
    - `@JoinColumn(name = "TEAM_ID", insertable = false, updateable = false)`
        - 읽기 전용 매핑
        - **FK와 매핑해준 후 강제로 읽기 전용 필드로 사용해서 양방향처럼 사용하는 방법** → 설정이 없으면 해당 Entity도 연관관계에서 FK를 관리할 수 있는 권한을 얻게되어 꼬이게 됨!!
    - 이런 복잡성, 강제성, 유지보수 및 최적화 문제 때문에 **보통 다대일 양방향을 주로 사용**
- 결론 : 일대다 말고 다대일 관계를 사용하자!!

### 일대일 [1:1] (@OneToOne)

- 일대일 관계
    - 일대일 관계의 반대 또한 일대일
    - **외래 키 선택 가능**! → 완전히 대칭적(1대1)이므로 **어디에 두든 상관없음**
        - 주 테이블에 외래 키 설정 방법
        - 대상 테이블에 외래 키 설정 방법
    - 외래 키에 DB UNIQUE 제약 조건 추가됨
- **단방향** 일대일

  ![Untitled]({{site.baseurl}}/images/posts/post-220812/Untitled 2.png)
    
    ```java
    // Member Entity
    @OneToOne
    @JoinColumn(name = "LOCKER_ID")
    private Locker locker;
    ```
    
    - 다대일(`@ManyToOne`) **단방향 매핑과 유사**
    - **FK 및 UNI 를 어디다 두든 상관없음!**
        - Member에 LOCKER_ID로 FK및 UNI 를 넣어도 되고
        - Locker에 MEMBER_ID로 FK및 UNI 를 넣어도 됨
    
    <aside>
    🚨 <strong>단방향 일대일 연관관계 주인 역설정 (대상 테이블에 외래 키)</strong> <br>
    만약 단방향 일대일 관계에서 FK는 Member Table에 있는데 Locker Entity에 연관관계의 주인을 주는 경우(연관관계 주인 역설정)로 설정하고 싶다면? <br>
    ⇒ 불가능!! JPA에서 지원하지 않음
    </aside>
    
- **양방향** 일대일

  ![Untitled]({{site.baseurl}}/images/posts/post-220812/Untitled 3.png)
    
    ```java
    // Locker Entity
    @OneToOne(mappedBy = "locker")
    private Locker locker;
    ```
    
    - **읽기 전용**. FK를 등록하거나 수정하는 권한이 없음 (`mappedBy`)
    - 다대일(`@ManyToOne`) 양방향 매핑과 유사 (`mappedBy` 사용)
    
    <aside>
    ⚠️ <strong>양방향 일대일 연관관계 주인 역설정 (대상 테이블에 외래 키)</strong>

    <img src="/images/posts/post-220812/Untitled 4.png">
    
    만약 양방향 일대일 관계에서 FK는 Member Table에 있는데 Locker Entity에 연관관계의 주인을 주는 경우(연관관계 주인 역설정)로 설정하고 싶다면? <br>
    ⇒ 양방향은 가능! 두 Entity 중 FK를 가지고 있는 Entity로 연관관계 주인을 재설정하면 됨. 즉, FK가 Locker Table로 가게 되는 것 (사실 양방향 일대일 연관관계 주인 역설정이라고 할 수 없음 → 제대로된 연관관계의 주인이기 때문)
    
    </aside>
    
- **연관관계 주인 正설정 vs 易설정**
    - **正설정 (주 테이블에 외래 키)**
        - 주 객체가 대상 객체의 참조를 가지는 것 처럼 **주 테이블에 외래 키를 두고 대상 테이블을 찾음**
        - 객체지향 개발자 선호
        - JPA 매핑 편리
        - 장점 : 주 테이블만 조회해도 대상 테이블에 데이터가 있는지 확인 가능
        - 단점 : 값이 없으면 외래 키에 **null 허용**
    - **易설정 (대상 테이블에 외래 키)**
        - **대상 테이블에 외래 키**가 존재
        - 전통적인 데이터베이스 개발자 선호
        - 장점: 주 테이블과 대상 테이블을 일대일에서 일대다 관계로 변경할 때 테이블 구조 유지 가능
        - 단점
            - **강제적으로 양방향** 일대일관계를 만들어야 함.
            - **프록시 기능의 한계로 지연 로딩으로 설정해도 항상 즉시 로딩**됨 → Member Entity에서 Locker Entity를 프록시로 설정하려면 Member Table의 LOCKER_ID(FK)가 null인지 아닌지 확인을 해야되는데, FK가 Locker Table에 있기 때문에 (대상 테이블에 외래키) 무조건 해당 Locker Table을 조회해봐야됨 → **프록시 불가**

### 다대다 [N:M] (@ManyToMany)

- 다대다 관계
    - RDB는 애초에 다대다(N:M) 관계를 표현할 수 없음
    - **연결 테이블을 추가해서 다대다(N:M)를 일대다(1:N), 다대일(M:1) 관계로 풀어내야 가능!**

      ![Untitled]({{site.baseurl}}/images/posts/post-220812/Untitled 5.png)
        
    - BUT! **객체는 다대다관계가 가능!! (Collection 이용)**

      ![Untitled]({{site.baseurl}}/images/posts/post-220812/Untitled 6.png)
        
    - 그래서 **다대다 관계**가 가능하도록 **JPA 에서 지원**하는 것
    - **하지만 해당 관계는 사용하면 안됨!!**
    - 즉, `@ManyToMany`를 사용하지 말고 **연결 객체 Entity를 생성**해서 **`@OneToMany` + `@ManyToOne`** 으로 풀어내야함
- 지원되는 다대다 **(실무에선 사용X)**

  ![Untitled]({{site.baseurl}}/images/posts/post-220812/Untitled 7.png)
    
    ```java
    // Member Entity
    @ManyToMany
    @JoinTable(name = "MEMBER_PRODUCT",
    				joinColumns = @JoinColumn(name = "MEMBER_ID"),
    				inverseJoinColumns = @JoinColumn(name = "PRODUCT_ID"))
    private List<Product> products = new ArrayList<>();
    ```
    
    - `@ManyToMany` : 해당 객체와 다대다 관계라는 것을 선언
    - `@JoinTable(name = "MEMBER_PRODUCT")` : 해당 이름으로 **연결 테이블 생성** → **서로의 PK와FK를 가지는** MEMBER_PRODUCT **CREATE**
    - BUT 안씀!!!
    
    ```java
    // Product Entity
    @ManyToMany(mappedBy = "products")
    private List<Member> members = new ArrayList<>();
    ```
    
    - `@ManyToMany(mappedBy = "products")` : 해당 객체와 다대다 관계라는 것은 선언하며, products에 의해 **연관당하는 입장**이라는 것을 선언
    - BUT 안씀!!!
- **다대다 매핑의 한계**
    - 편리해 보이지만 **실무에서 사용X**
    - 중간에 매핑되고 있는 **연결 테이블**이 있기 때문에, 내가 생각지도 못한 query가 나갈 수 있음 → **유지,보수 및 최적화 장애**
    - 연결 테이블이 단순히 연결만 하고 끝나지 않음 → 주문시간, 수량과 같은 **부가적인 데이터가 필요할 경우**가 언젠가는 옴

      ![Untitled]({{site.baseurl}}/images/posts/post-220812/Untitled 8.png)
        
- **다대다 매핑의 한계 극복**
    - **“대다다”**를 “**일대다, 다대일”로 표현**하며 **중간에 엔티티를 추가!**
    - 즉, 연결 테이블용 엔티티 추가 → **연결 테이블을 엔티티로 승격**
    - 추가로 PK,FK 를 혼용하지 않고 **PK를 따로 줌! → 엔티티로써 사용, 유연성 증가**

  ![Untitled]({{site.baseurl}}/images/posts/post-220812/Untitled 9.png)
    
    - MemberProduct Entity (연결 엔티티)
        
        ```java
        @Entity 
        public class MemberProduct { 
            
        
            @Id
            @GeneratedValue
            private Long id;
        
            @ManyToOne
            @JoinColumn(name = "MEMBER_ID")
            private Member member;
        
            @ManyToOne
            @JoinColumn(name = "PRODUCT_ID")
            private Product product;
        
            private int count;
            private int price;
        }
        ```
        
        - `@Entity`
            - 다대다를 **일대다+다대일 로 풀어낼 때 사용되는 객체**
            - `@ManyToMany` 의 연결테이블을 엔티티로 승격
        - `@ManyToMany` -> `@OneToMany` (Member, Product입장 → **다대다를 하고자하는 Entity들**), `@ManyToOne` (MemberProduct입장 → **연결 Entity**)
            - MemberProduct ↔ ****Member : Many ↔ One ⇒ `@ManyToOne`
            MemberProduct Table의 **MEMBER_ID FK와 매핑**
            - MemberProduct ↔ Product : Many ↔ One ⇒ `@ManyToOne`
            MemberProduct Table의 **PRODUCT_ID FK와 매핑**
        - 추가로 다대다를 풀어냄과 동시에 **추가적인 정보**를 담을 수 있음!!
        → **중간 객체를 사용하는 이유**
    - Member Entity (연결 엔티티와 일대다)
        
        ```java
        @OneToMany(mappedBy = "member")
        private List<MemberProduct> memberProducts = new ArrayList<>();
        ```
        
        - `@ManyToMany` -> `@OneToMany` (Member, Product입장 → **다대다를 하고자하는 Entity들**), `@ManyToOne` (MemberProduct입장 → **연결 Entity**)
            - Member ↔ MemberProduct : One ↔ Many ⇒ `@OneToMany`
            MemberProduct의 member에 의해 **연관당하는 입장**
    - Product Entity (연결 엔티티와 다대일)
        
        ```java
        @OneToMany(mappedBy = "product")
        private List<MemberProduct> memberProducts = new ArrayList<>();
        ```
        
        - `@ManyToMany` -> `@OneToMany` (Member, Product입장 → **다대다를 하고자하는 Entity들**), `@ManyToOne` (MemberProduct입장 → **연결 Entity**)
            - Product ↔ MemberProduct : One ↔ Many ⇒ `@OneToMany`
            MemberProduct의 product에 의해 **연관당하는 입장**
    - 결론적으로 `@ManyToMany` 를 `@OneToMany` (Member, Product입장 → **다대다를 하고자하는 Entity들**) + `@ManyToOne` (MemberProduct입장 → **연결 Entity**) 로 풀어내 **부수적인 효과(유지보수 및 최적화 여지, 필요 데이터 추가)를 얻어냄**