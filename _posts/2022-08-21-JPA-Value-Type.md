---
layout: post
title:  "Value Type (Embedded Type, Collection Type, ... )"
date:   2022-08-21
image:  /posts/jpa.png
tags:   SpringBoot JPA
---

## JPA에서의 데이터 상위 타입 분류

### Entity Type

- **Table로도 설정**되는 객체
- `@Entity` 로 정의되는 객체
- **데이터가 변경되도 식별자(PK)로 지속적인 추적 가능**

### 값 타입

- int, Integer, Long, String 처럼 **단순히 값으로 사용하는 Java 기본 타입 or 객체**
- **식별자(pk)가 없고 값만 있기에 변경 시 추적 불가 [중요]** → 변경 시 아예 **대체가 되어버림**
- ex) Integer 100을 200으로 변경하면 해당 객체의 값이 100에서 200으로 변경 되는 것이 아닌 100을 새로운 Integer 200으로 대체하는 것

## 값 타입

### 값 타입 특징

- 특징
    - **생명주기를 엔티티에 의존** 
    → 해당 Entity를 저장하면 같이 저장되고 삭제하면 같이 삭제됨
    - **값 타입은 공유되면 안됨**
    → 한 Entity의 기본 값을 변경했는데 다른 Entity의 기본 값도 변경되면 안됨! (나쁜 의미의 부수효과) [**독립적**으로 사용해야 됨]
- 종류
    1. 기본값 타입 (Basic Type)
        - Java basic Type (int, double, …)
        - 래퍼 클래스(Integer, Long)
        - String
    2. 임베디드 타입 (Enbedded Type, 복합 값 타입)
    3. 컬렉션 값 타입 (Collection Value Type)
    
    ```java
    @Entity
    public class Member{
    		// 식별자 (기본값 타입)
        @Id @GeneratedValue(strategy = GenerationType.AUTO)
        @Column(name = "MEMBER_ID")
        private Long id;
    
    		// 1. 기본값 타임
        private String name; 
    
        @Embedded // 2. 임베디드 타입
        private Address address; 
    
        @ElementCollection // 3. 컬렉션 값 타입
        @CollectionTable(
                name = "ADDRESS"
                joinColumns = @JoinColumn(name = "MEMBER_ID")
        )
        private List<Address> addressHistory = new ArrayList<>(); 
    }
    ```
    

### 기본값 타입

- 특징
    - **단순히 값으로** 사용하는 Java 기본 타입
    - int, Integer, Long, String
- 주의점 : **기본 값 타입은 절대 공유해선 안됨**
    - int, double 같은 기본 타입(primitive type)은 절대 공유가 안되며 공유해선 안됨
        - 기본 타입(primitive type)은 **항상 값을 복사**함
        - `int a = 20; int b = a;` 라고 할 때, a의 복사된 값이 b에 들어가는 것
        - 즉, **레퍼런스를 주는 것이 아니라 값을 복제하여 집어 넣음**
    - Integer같은 래퍼 클래스나 String 같은 특수한 클래스는 공유 가능한 객체이지만 공유해서는 안됨! (변경이 안되긴 함)
        - `Integer a = new Integer(10); Integer b = a;` 라고 할 때, a의 **Reference가 b로 넘어가는 것**!
        - 만약 여기서 b의 값을 변경할 수 있다면, a의 값도 함께 변경됨
        - 즉, **둘을 공유해서 사용하면 부정적 부수 효과가 일어남! → 공유X**

### 임베디드 타입 (복합 값 타입)

- 개념
    - 사용할 수 있는 기존의 기본 값 타입 이외의 **새로운 값 타입을 직접 정의**할 수 있음 (물론 **Table** 저장 시 직접 정의된 **값 타입 속 기본 값 타입들로 저장** 됨)
    - JPA에서는 이를 **임베디드 타입(Embedded Type)**이라고 부름
    - 이는 주로 기본 값 타입들(int, String, … )을 모아서 만들기에 “**복합 값 타입**” 이라고도 부름
    - 값 타입이기에 **Entity가 아님!** → **변경 시 추적 불가능! 생명주기가 연결된 Entity를 따름**
- 임베디드 타입이 필요한 상황
    - Member Entity 에 다른 Entity에서 중복되며 사용되는 field 들(**공통적인 속성들**)이 있음
    - 이를 통합 시켜 사용하고자 하는 상황 (비지니스적으로도 깔끔해지고, 사용할 수 있는 방향이 다양해짐(다른 Entity에서도 공통적으로 사용할 수 있는 것!))

      ![Untitled]({{site.baseurl}}/images/posts/post-220821/Untitled.png)
        
    - 이 상황은 임베디드 타입을 생성하여 구현해 낼 수 있음

      ![Untitled]({{site.baseurl}}/images/posts/post-220821/Untitled 1.png)
        
- 특징
    - **재사용 가능** (**공통적인 속성을 재사용**하여 계속 깔끔히 사용 가능)
    - **높은 응집도**
        - 비지니스적으로 공통적인 속성들을 한데 묶을 수 있음
        - **해당 값 타입만 사용하는 의미 있는 메소드**를 만들 수 있음 → 임베디드 타입 객체 속 메서드 생성하여 활용 (객체 지향적인 설계)
    - 임베디드 타입은 값 타입(int, String 과 같은 타입이라 생각하면 됨)이기에 **이를 소유한 Entity에 생명주기를 의존**
    - 임베디드 타입의 값이 null 이면 해당 속성들은 모두 null로 저장됨
- 테이블과의 매핑

  ![Untitled]({{site.baseurl}}/images/posts/post-220821/Untitled 2.png)
    
    - 임베디드 타입을 사용하기 전과 후에 매핑하는 테이블은 같음 → **DB입장에서 기본값 타입으로 사용하든, 임베디드 타입을 사용하든 똑같은 Table로 저장**됨
    - 즉, Table에는 영향을 주지 않은 채로 Application 단에서는 재사용, 응집도 증가 등 다양한 장점을 얻어낼 수 있음
    - 객체와 테이블을 아주 세밀하게 매핑하는 것이 가능
    - 잘 설계한 ORM Application은 매핑한 테이블의 수보다 클래스의 수가 더 많음!
- 사용법
    - `@Embeddable` : 값 타입을 정의하는 곳에 표시
        
        ```java
        @Embeddable // 값 타입 선언
        public class Address {
        
            private String city;
            private String street;
            private String zipcode;
        
            public Address() {
            }
        		... // getter setter
        }
        ```
        
        - 공통적으로 사용되는 속성들을 한데 묶어서 표현할 수 있음
        - 이들만을 다루는 메서드 생성 가능
        - **기본 생성자 필수** → JPA가 해당 Embedded 객체를 생성하여 Entity에 넣어주기 때문
    - `@Embedded` : 값 타입을 사용하는 곳에 표시
        
        ```java
        // Member Entity
        @Embedded
        private Address address;
        ```
        
        - 이렇게 되면 Member Entity는 Address Enbedded Type을 가지게 되고, Address의 속성들(city,street, zipcode)을 사용할 수 있게 된 것
        - 어쨌든 Table은 동일하게 구성됨
- + 연관관계

    ![Untitled]({{site.baseurl}}/images/posts/post-220821/Untitled 3.png)
    
    1. 임베디드 타입이 임베디드 타입을 가질 수 있음
    2. **[중요] 임베디드 타입이 Entity를 가질 수 있음!**
- `@AttributeOverride`
    
    ```java
    // Member Entity
    @Embedded
    private Address homeAddress;
    
    @Embedded
    @AttributeOverrides({
    		@AttributeOrverride(name = "city",
    								column = @Column("work_city")),
    		@AttributeOrverride(name = "street",
    								column = @Column("work_street")),
    		...
    })
    private Address workAddress;
    ```
    
    - **속성 재정의**
    - **한 엔티티에서 같은 값 타입을 사용**할 때, 컬럼 명 **중복을 방지**하기 위함
    - @AttributeOverrides, @AttributeOverride를 사용해서 Col명 속성을 재정의

### 값 타입과 불변 객체

- 목표
    - Integer, String 과 같이 Java에서 제공 하는 기본 값타입들은 부작용(side effect)가 발생하지 않도록 잘 설계되어 있음
    - 하지만, 임베디드 값 타입과 같이 **직접 값 타입을 생성하여 사용할 때는 이런 부작용(side effect)**에 대해서 직접 생각하고 다루어 **안전하게 설계가 필요**
    - 해당 문제점들을 피하는 방식 학습 필요
- 값 타입 공유 참조 [잘못된 방식]

  ![Untitled]({{site.baseurl}}/images/posts/post-220821/Untitled 4.png)
    
    ```java
    Member m1 = new Member();
    Member m2 = new Member();
    Address address = new Address("old city", "12 street", "12111");
    
    // 공유 참조
    m1.setAddress(address);
    em.persist(m1);
    m2.setAddress(address);
    em.persist(m2);
    
    m1.getAddress().setCity("New city"); // 변경 -> m2 의 city도 변경됨
    ```
    
    - 임베디드 타입을 여러 엔티티에서 공유한 상태
    - 여기서 해당 임베디드 타입의 값을 변경 → **공유하고 있는 Entity들의 값이 모두 바뀜! (side effect)**
    - 이런 Side Effect로 인해 발생하는 버그는 잡기 굉장히 어려움!
    - 그렇다고 **객체의 공유 참조는 피할 수 없음**
- 값 타입 복사 [올바른 방법]

  ![Untitled]({{site.baseurl}}/images/posts/post-220821/Untitled 5.png)
    
    ```java
    Member m1 = new Member();
    Member m2 = new Member();
    Address address = new Address("old city", "12 street", "12111");
    
    // 공유 참조
    m1.setAddress(address);
    em.persist(m1);
    Addresss copied = new Address(address.getCity(), ...);
    m2.setAddress(copied);
    em.persist(m2);
    
    m1.getAddress().setCity("New city"); // 변경 -> m2 의 city가 변경되지 않음
    ```
    
    - 값 타입의 실제 인스턴스를 공유하는 것이 아닌
    - **값을 복사해서 사용** → **부작용(side effect)이 발생하지 않음**.
    - 말 그대로 **새로운 값 타입을 넣어 주는 것!**
- **불변 객체**
    - **생성 시점 이후** 절대 **값을 수정/변경 할 수 없는 객체**
    - 객체 타입(직접 생성한 값 타입. ex_임베디드 객체)은 **공유 참조를 막을 수 없음** → 이를 **해결하기 위함이 “불변 객체”**
    - 객체 타입을 수정할 수 없게 만들어 **부작용(side effect)을 원천 차단**
    - **값 타입**은 **불변 객체(immutable object)로 설계**해야 함
    - 불변 객체 설정 방법 : **생성자로만 값을 설정하고 수정자를 만들지 않으면 됨!!!**
        
        ```java
        @Embeddable // 값 타입 선언
        @Getter // getter만 열어두고 setter는 열지 않음 -> 불변 객체 설정
        public class Address {
        		...
        		public Address(String city, String street, String zipcode) {
        			...
            }
            public Address() {
            }
        }
        ```
        
    - Integer, String과 같은 객체들은 자바에서 기본으로 불변 객체로 설정

### 값 타입의 비교

- 값 타입은 인스턴스(주소)가 달라도 **그 안의 값이 모두 같다면 같은 것**으로 봐야 됨
    
    `int a = 10; int b = 10; a== b` → true
    
    ```java
    Address a1 = new Address("city1","12street");
    Address a2 = new Address("city1","12street");
    a1 == a2 // 항상 false
    a1.equals(a2) // true 로 만들어 줘야 함! -> 값 타입이므로
    ```
    
- 값 타입에서의 비교
    - **동일성(identity) 비교** : 인스턴스의 **참조 값**을 비교, `==` 사용
    - **동등성(equivalence) 비교** : 인스턴스의 **값**을 비교, `equals()` 사용
    - **값 타입**은 값을 비교해야 되기 때문에 항상 `equals()` 를 사용하여 **동등성을 비교**해야 됨
    - 즉, 값타입의 `equals()`메서드를 **적절하게 재정의** 필요 → **모든 필드를 사용**하여
    - 예시
        - 사용자 정의 값 타입 (임베디드 타입)의 equals(), hashCode() Override를 통해 재정의
            
            ```java
            @Override
                public boolean equals(Object o) {
                    if (this == o) return true;
                    if (o == null || getClass() != o.getClass()) return false;
                    Address address = (Address) o;
                    return Objects.equals(getCity(), address.getCity()) && Objects.equals(getStreet(), address.getStreet()) && Objects.equals(getZipcode(), address.getZipcode());
                }
            
            @Override
            public int hashCode() {
                return Objects.hash(getCity(), getStreet(), getZipcode());
            }
            ```
            
            - `equals()` 에서 **getter**를 사용하여 속성을 가져오는 것이 좋음 → 프록시 등 나중에 예기치못한 오류를 맞이할 가능성이 큼
        - 이렇게 재정의를 하지 않은 경우에는 `equals()` 를 사용하면 false가 반환됨 (`equals()` 의 기본은 `==` 비교)
        - 재정의 후 → **값 비교 가능!**

### 값 타입 컬렉션

- 개념

  ![Untitled]({{site.baseurl}}/images/posts/post-220821/Untitled 6.png)
    
    - 값 타입을 하나 이상 저장할 때 사용
    - **DB는 Collection을 해당 Entity와 같은 Table에 저장할 수 없음**
    - Collection을 저장하기 위한 **별도의 Table이 필요**
    - Collection으로 생성된 Table은 **모든 속성이 묶여서 PK가 됨** → 값 타입의 특징 (만약 PK가 하나고 나머지는 그냥 속성값으로 남게 되면 그건 그냥 Entity가 되어버림)
    - 값 타입 Collection도 당연히 나머지 **값 타입과 같이 동작** → 모든 라이프사이클이 해당 Entity에 의존 (persist, remove 등이 필요 없음) ⇒ **영속성 전이 + 고아 객체 제거 기능을 필수로 가짐**
- 사용
    - `@ElementCollection`, `@CollectionTable` 사용
    
    ```java
    // Member Entity
    @ElementCollection
    @CollectionTable(
            name = "FAVORITE_FOOD", // 테이블 명
            joinColumns = @JoinColumn(name = "MEMBER_ID") // FK 선택
    )
    @Column(name = "FOOD_NAME") // String 속성 하나이기 때문에 예외적으로 설정 가능
    private Set<String> favoriteFoods = new HashSet<>();
    
    @ElementCollection
    @CollectionTable(
            name = "ADDRESS", // 테이블 명
            joinColumns = @JoinColumn(name = "MEMBER_ID") // FK 선택
    )
    private List<Address> addressHistory = new ArrayList<>();
    ```
    
    - `@ElementCollection` : 해당 속성을 **Collection Table로 저장**한다는 것
    - `@CollectionTable` : Collection 은 Table로 저장되기 때문에, 해당 Colleciton Table 설정해주는 것
        - name : 테이블 명
        - joinColumns : **Table과 매핑할 FK** 선택 및 설정 → 어쨌든 **해당 Entity와 연결**되어야 하므로 **Foriegn Key를 설정**해야됨
    - 저장
        - Entity 저장 시 따로 Collection은 persist 할 필요 없음 → **영속성 전이 + 고아 객체 제거 기능**을 필수로 가짐
        - 테이블 저장 확인

          ![Untitled]({{site.baseurl}}/images/posts/post-220821/Untitled 7.png)
            
        - 이런 식으로 Table로 저장되는 것을 확인할 수 있음
    - 조회
        - 조회 시 같이 한번에 끌어다 오는 것이 아니라 **해당 collection에 접근해야지 실제로 값을 가져옴** → **지연로딩 전략** 사용
    - 수정
        - 값 타입이기에 따로 저장하거나 그럴 필요 없이 알아서 Entity에 저장됨
        - Set Collection 수정
            
            ```java
            findMember.getFavoriteFoods().remove("치킨");
            findMember.getFaveriteFoods().add("짜장면");
            ```
            
            - Set Collection은 수정 가능, But 값 자체를 수정하는 것이 불가 → **제거한 후 새로 넣어주어야 함**
        - List Collection 수정
            
            ```java
            findMember.getAddressHistory().remove(new Address(...)); // equals() 로 동작
            findMember.getAddressHistory().add(new Address(...));
            ```
            
            - [중요] 값 타입 List Collection은 **변경 사항이 발생**하면, 관련된 **모든 데이터를 삭제하고 값 타입 컬렉션에 있는 현재 값을 모두 다시 저장**
                - 즉, 원하는 Address를 삭제하고 새로운 Address를 넣은 **새로운 Collection이 해당 Entity에 넣어지게 되는 것** → DB query가 날라가는 것을 통해 확인 가능 (Address 전체를 삭제 한 후, 새로운 Address를 저장)
                - 이는 값 타입이 엔티티와 다르게 **식별자 개념이 없이 모든 컬럼을 묶어서 PK를 구성하기 때문** → 변경 시 추적이 안되는 이유 (수정하게 되면 그 구성된 Pk가 깨지는 것이니깐!)
                - 추가로 Set과는 다르게 **순서값이 존재**하므로! (Set은 순서가 없어서 상관 없음)
            - **[중요]** 고로 List Collection은 값 타입 Collection 보다는 `@OneToMany`로 연결해주는 것이 낫다! → 즉, **자체적인 PK를 가지게 하여 추적할 수 있게하는 것**
- **값 타입 컬렉션 대안**
    - 실무에서는 상황에 따라 **값 타입 Collection 대신 일대다 관계를 고려**
    - 일대다 관계만을 위한 Entity를 만들고, 여기서 값 타입을 사용하는 것 (PK가 딸린 Collection 느낌)
    - **[중요] 영속성 전이 + 고아 객체 제거 기능을 이용해서 값 타입 Collection 같이 사용**하는 것!
    
    ```java
    @Entity
    public class AddressEntity {
        @Id
        @GeneratedValue
        private Long id;
    
        @Embedded
        private Address address;
    }
    ```
    
    ```java
    // Member Entity (@ElementCollection -> @OneToMany)
    @OneToMany(cascade = CascadeType.ALL, orphanRemoval = true) // 
    @JoinColumn(name = "MEMBER_ID")
    private List<AddressEntity> addressHistories = new ArrayList<>();
    ```
    
    - `@OneToMany` 에서 `cascade = CascadeType.ALL, orphanRemoval = true` 를 통해서 더 **Collection 의 느낌으로 사용**할 수 있음
    - 이렇게 Collection을 일대다 관계로 풀어내게 되면 활용할 수 있는 것들이 훨씬 많아짐 (전용 Method 추가 등)

<aside>
⚠️ <strong>값 타입 사용 시 유의할 점</strong>
<ul>
<li> 값 타입은 정말 <b>값 타입이라고 판단 될 때만 사용</b>해야됨 </li>
<li> Entity와 값 타입을 혼동해서 사용하면 안됨! 명확한 구분이 필요! </li>
<li> <b>식별자가 필요하고, 지속해서 값을 추적하며 해당 값을 자주 사용하게 된다</b>면 값 타입이 아닌 <b>Entity로 사용</b>해야 됨! </li>
</ul>
</aside>