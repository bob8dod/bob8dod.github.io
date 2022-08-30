---
layout: post
title:  "Proxy And Association Management"
date:   2022-08-18
image:  /posts/jpa2.jpeg
tags:   SpringBoot JPA
---

## 프록시

### 프록시(+지연로딩)는 왜 필요한가?

- 한 Entity를 조회할 때 그 **연관관계에 있는 Entity도 항상 함께 조회**해야 할까? ⇒ 로직에 따라 다름!
    - 만약, 현 Entity와 그 연관관계에 있는 Entity도 같이 사용된다 → 함께 조회가 좋음
    - 하지만, 현 Entity만 사용한다. → 연관관계에 있는 Entity는 쓸데없으므로 조회하지 않고 현 Entity만 조회하는 것이 좋음
    - 즉, **비지니스 로직에 따라 함께 조회하는 경우도 있고, 함께 조회하지 않고 단일 Entity만을 필요로 할 때**가 있음
    - 그러므로 모든 상황에서 모든 연관관계의 Entity를 함께 끌고 오는 것은 낭비!
    - 이런 문제점을 JPA는 **Proxy와 지연로딩으로 해결!**

### 프록시

- 프록시 기초 (`em.find(Entity)` vs `em.getReference(Entity)`)
    - `em.find(Entity)` : DB를 통해서 **실제 Entity 객체 조회**
        - `em.find(Entity)` 를 통해 반환된 객체는 `Member` 그 자체
    - `em.getReference(Entity)` : DB 조회를 미루는 **가짜(Proxy) Entity 객체 조회**
        - `em.getReference(Entity)` 를 통해 반환된 객체는 Member가 아닌 **`Member$HiberanteProxy…` 라는 가짜 객체!!**

          ![Untitled]({{site.baseurl}}/images/posts/post-220818/Untitled.png)
            
            해당 **Proxy 객체**는 텅텅 비어있으면서 **실제 Entity를 가리키는 tratget**, 실**제 Entity의 속성들을 접근할 수 있는 method들**로 설정되어 있음
            
        - DB에 query가 안나갔는데 객체가 조회됨 (물론 가짜 객체)
        - 해당 가짜 Entity 객체의 **속성값에 접근하게 되면 그제서야 실제Entity를 불러옴** (**DB에 query가 나가**는 것)
            
            ```java
            Member member = em.getReference(Member.class, 1L); // 이때 query가 발생하지 않음
            member.getId(); // id는 이미 파라미터로 넣었기 때문에 알고 있음 (이미 존재하는 값)
            member.getName(); // 이때 조회 query가 발생! → 속성 접근 (알고있지 않은 값)
            ```
            
- 프록시 기본 특징

  ![Untitled]({{site.baseurl}}/images/posts/post-220818/Untitled 1.png)
    
    - **실제 클래스를 상속** 받아 만들어짐 (Hibernate_JPA구현체가 내부적으로 동작)
    - 그래서 실제 클래스와 겉 모양이 같다. → Field에 접근할 수 있는 method들을 가지고 있는 것
    - 이론 상으로는 사용하는 입장에서는 진짜 객체인지 Proxy 객체인지 구분하지 않고 사용할 수 있음
    - Proxy 객체는 **실제 객체의 참조(target)를 보관**
    - Proxy 객체를 접근하면 Proxy 객체는 Target에 있는 실제 객체의 메소드를 호출
        - 이 때 DB에 직접 query를 날려 실제 Entity를 가져오는 것
        - 그렇다고 **해당 Proxy 객체가 실제 Entity로 교체되는 것이 아님**
        - 그저 **Traget에 실제 Entity가 채워지**는 것
        - 한번 채워지면 Transaction단위가 끝날 때까지 재사용 가능
- 프록시 객체의 초기화
    
    ```java
    Member member = em.getReference(Member.class, 1L);
    member.getName();
    ```

  ![Untitled]({{site.baseurl}}/images/posts/post-220818/Untitled 2.png)
    
    1. .**getName()**
        - `em.getReference()`로 Proxy 객체를 불러온 후 Proxy 객체의 method인 `.getName()` 호출 → **실제 객체 접근 시도**
        - 강제 **초기화를 시도**하는 것 (해당 방법말고 `Hibernate.initialize(entity)` 를 통해서도 강제 초기화 가능)
    2. **초기화 요청**
        - 현재 Target에 값이 존재하지 않기 때문에 **영속성 컨텍스트에 Target의 값(실제 Entity)을 요청**
    3. **DB 조회**
        - 그에 때라 영속성 컨텍스트는 **실제 Entity를 찾아오기 위해 DB를 조회**
        - 이때 **조회 query**가 나가는 것
    4. **실제 Entity 생성**
        - DB에서 조회한 결과로 **실제 Entity를 생성하여 Target의 값을 마련**하고 **Proxy 객체의 Target에 해당 실제 Entity를 연결** 해줌
    5. **target.getName()**
        - 결론적으로 **traget(실제 Entity)의 getName() 을 통해 값을 반환**
    - **[참고]** 한번 초기화가 되면 Target에 값이 연결되기 때문에 다시 DB를 조회할 일은 없음 → **초기화 후 재사용 가능**
- 프록시 주요 특징
    - Proxy  객체는 처음 사용할 때 **한 번만 초기화**
    - Proxy 객체를 **초기화** 할 때, **Proxy 객체가 실제 Entity로 바뀌는 것은 아님** → 초기화 되면 Proxy 객체를 통해서 **실제 Entity에 접근하는 것** (Proxy 객체의 Target을 통해 연결된 Entity에 접근)
        
        ```java
        Member member = em.getReference(Member.class, 1L);
        System.out.println(findMember.getClass()) // MemberHibernateProxy...
        member.getName(); // 실제 Entity 조회 (Target 연결)
        System.out.println(findMember.getClass()) // Member? X -> MemberHibernateProxy...
        ```
        
    - 그에 따라 **Type 체크 시 주의**해야됨
        - Proxy 객체는 원본 Entity를 **상속**받으며 Proxy 객체는 **원본 Entity로 교체되지 않음**
        - ⇒ ‘`==`’ 비교 X, ‘`instance of`’ **O**
        - 해당 문제로 실수할 일이 없다고 생각할 수도 있지만, 직접 개발에 들어가면 예상치 못한 Proxy 객체로 `==` 비교에서 오류를 얻을 수 있음! → `instance of` 사용 필수!
        - 이런 특징 상 Proxy든 아니든 개발에 문제가 없도록 진행해야 됨
    - 영속성 컨텍스트에 찾는 엔티티가 이미 있으면 `em.getReference()` 를 호출해도 실제 Entity 반환
        - **영속성 컨텍스트에 Entity가 존재하면, 사실 Proxy를 쓸 이유가 없어짐!** (Proxy 객체의 target에 연결할 필요가 없음! → 이미 있었으니깐)
        - 그래서 그냥 해당 Proxy 객체를 **존재하고 있었던 Entity로 변환**하는 것!
    - Proxy객체가 영속성 컨텍스트의 도움을 받을 수 없는 **준영속 상태**일 때, **프록시를 초기화하면 예외 발생** → `org.hibernate.LazyInitializationException`
        - Proxy 객체를 detach를 통해 준영속 상태로 만든 경우,
        - EntityManger를 끈 경우,
        - Transaction 단위에서 실행되지 않을 경우 등
        - **Proxy 객체가 초기화 시(실제 Entity를 가져올 때) 사용하는 것이 영속성 컨텍스트이므로 당연히 오류 발생!**

## 즉시 로딩과 지연 로딩

### 즉시 로딩과 지연 로딩 설정은 왜 있는가?

- **Proxy 의 필요성**과 연관되어 있음
- 즉, **비지니스 로직에 따라 함께 조회하는 경우도 있고, 함께 조회하지 않고 단일 Entity만을 필요로 할 때**가 있음
- 이때의 사용되는 설정이 **즉시로딩과 지연로딩**

### 지연 로딩

- 비지니스 로직 자체가 현 Entity만 사용하는 일이 많을 때(**연관관계의 Entity를 같이 사용하는 일이 많이 없는 경우**) 사용
- `fetch = FetchType.LAZY`
- 설정 및 원리
    
    ```java
    // Member Entity
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "TEAM_ID")
    private Team team;
    ```

  ![Untitled]({{site.baseurl}}/images/posts/post-220818/Untitled 3.png)
    
    - `@ManyToOne(fetch = FetchType.LAZY)` : 해당 연관된 객체를 **Proxy 객체로 조회한다**는 뜻
    - 즉, Member Entity를 조회할 때, 해당 Team Entity를 부가적으로 가져오는 것이 아니라 **Member Entity 만을 조회하고, Team Entity는 Proxy로 채워주는 것**
    - **Team Entity는 Proxy 상태**. → 접근(초기화)해야지만 실제 Entity를 가져와(DB에 query가 날라감) Proxy 객체의 Target에 연결해주게 됨 (ex_ `team.getName();`)

### 즉시 로딩

- 비지니스 로직 자체가 **현 Entity와 연관관계의 Entity를 함께 사용하는 일이 많을 때** 사용 (이런 상황에서 지연로딩을 사용하게 되면, 쓸데없이 query 2방이 나오는 것! → Member 조회 후 Team의 Proxy로 인해 또 Team을 조회하므로)
- `fetch = FetchType.Eager` (`@ManyToOne`, `@OneToOne`의 fetch는 Default 가 Eager)
- 설정 및 원리
    
    ```java
    // Member Entity
    @ManyToOne(fetch = FetchType.Eager)
    @JoinColumn(name = "TEAM_ID")
    private Team team;
    ```

  ![Untitled]({{site.baseurl}}/images/posts/post-220818/Untitled 4.png)
    
    - `@ManyToOne(fetch = FetchType.Eager)` : 해당 연관된 객체를 **실제 Entity로 조회**한다는 뜻 (**JOIN**을 통해 가져오게 됨)
    - 즉, Member Entity를 조회할 때, 해당 **실제 Team Entity를 자동으로 가져오는 것**
    - Team Entity는 실제 Entity 상태 **(Not Proxy)**
- 즉시로딩과 프록시 주의 사항
    - 즉시로딩은 사실 실무에서 사용하면 안됨! (**실무에선 지연 로딩 권장**)
    - 즉시 로딩은 **예상치 못한 SQL이 추가적으로 발생**할 수 있음 → 자동으로 연관된 Entity들을 DB에서 가져오기 때문에
    - 특히 **JPQL에서 1+N 문제**가 발생
        - 예를 들어 JPQL로 Member List를 가져온다고 하면, 그 때 query는 1개가 나가게 됨. 하지만 즉시로딩으로 설정되어 있다고 하면 각 Member의 Team Entity까지 조회해오게 됨. 즉, 추가적으로 Member의 개수인 N만큼 query가 발생하는 것 → 네트워크측면에서의 성능이 굉장히 저하
        - 해결법 : **모든 연관관계를 LAZY로 설정한 후 JPQL의 fetch join 으로 한방에 묶어서 조회**
    - `@ManyToOne`, `@OneToOne`은 **Defalut가 Eager**. ( → **Lazy로 변경**해주어야 함)
    - `@OneToMany`, `@ManyToMany` 는 **Defalut가 Lazy** ( → 그대로 두면 됨)
    - 즉, 일단 **모든 연관관계는 지연로딩으로 설정**하고, 한번에 같이 사용해야 된다 하면 즉시로딩 대신 **fetch join을 통해 한방에 묶어서 사용**하면 됨

## 영속성 전이와 고아 객체

### 영속성 전이 : CASCADE

- 연관관계를 매핑하는 것이나 즉시,지연 로딩과는 관계없는 개념
- 두 객체간의 관계에서 **자동으로 저장하거나 삭제하고 싶을 때 사용**하는 개념
- 즉, **특정 엔티티를 영속 상태(persist)로 만들 때 이와 연관된 엔티티도 함께 영속 상태(persist)로 만들고 싶을 때** 사용
- 사용
    - `cascade=CascadeType.ALL`
    - 부모와 자식간의 관계에서 **영속성 전이를 적용**하고 싶은 상황

      ![Untitled]({{site.baseurl}}/images/posts/post-220818/Untitled 5.png)
        
    - Child Entity
        
        ```java
        @Entity
        public class Child{
        	...
        	@ManyToOne(fetch = FetchType.LAZY)
        	@JoinColumn(name = "PARENT_ID")
        	private Parent parent;
        	...
        }
        ```
        
    - Parent Entity
        
        ```java
        @Entity
        public class Parent{
        	...
        	@OneToMany(mappedBy = "parent", cascade=CascadeType.ALL)
        	private List<Child> childList = new ArrayList<>();
        	...
        }
        ```
        
        - `@OneToMany(mappedBy = "parent", cascade=CascadeType.PERSIST)` : 해당 객체와의 관계에 있어서 **Parent가 저장되면 알아서 연관된 Child들도 저장**하겠다는 것
        - 즉, `em.persist(parent);` 해주면 **자동으로 딸린 child 들에 대해서도 개수만큼** `em.persist(child);` 를 해주는 것
        - **삭제도 마찬가지**
- **Cascade 주의!**
    - 영속성 전이는 **연관관계를 매핑하는 것과 아무 관련이 없음**
    - 그저 엔티티를 영속화할 때 **연관된 엔티티도 함께 영속화하는 편리함을 제공**할 뿐
    - 보통 `cascade=CascadeType.ALL` 을 많이 사용함 → **persist+remove 모두 가능**
    - 하지만, **remove는 항상 조심**해야 될 필요가 있음
    - 고로 비지니스 로직 상 **무조건 함께 움직이는 관계**다 → **ALL**
    - 무조건 함께는 아니지만, **저장시 함께 움직이는 관계** → **PERSIST**
    - **REMOVE 도 적용되는 것은 항상 조심!!**
    - Cascade는 **소유자가 하나 일때만 사용**하는 것! → **공동 소유자일 경우에는 사용해서는 안됨**
    - 즉, **정말 함께 움직이는 것들(라이프사이클이 동일한 놈들)에 대해서만 적용**할 것!

### 고아 객체 자동 제거 : orphanRemoval

- **고아 객체 자동 제거** : 부모 Entity와 **연관관계가 끊어진 자식 엔티티를 자동으로 삭제**하는 기능
- `orphanRemoval = true`
- ex)
    
    ```java
    // Parent Entity
    @OneToMany(mappedBy = "parent", cascade=CascadeType.ALL, orpahRemoval = true)
    private List<Child> childList = new ArrayList<>();
    ```
    
    ```java
    Parent parent = em.find(Parent.class, 1L);
    parent.getChilds().remove(0);
    ```
    
    - 이와 같이 **자식 Entity를 부모 Entity의 Collection에서 제거**할 때 → **관계가 끊어**졌기 때문에 **제거될 필요**가 있음! → **고아 객체 자동 제거**
    - `orphanRemoval = true` 설정으로 **자동으로 DELETE query 가 나감**
- 주의할 점
    - 참조가 **제거된 Entity는 다른 곳에서 참조되지 않고 있어야 됨**
    - 즉, **참조하는 곳이 하나일 때만 사용**해야 함!! (삭제 되므로)
    ⇒ **특정 Entity가 개인 소유할 때 사용**해야 되는 것
    - `@OneToOne`, `@OneToMany` 에서만 사용 가능
    
    <aside>
    ⚠️ <strong>고아객체 참고할 점</strong> <br>
    일반적으로 부모를 제거하면 자식은 당연히 고아 객체가 됨. 따라서 고아객체 제거 기능(<code>orphanRemoval = true</code>)을 활성화하면 부모를 제거할 때 자식도 함께 제거됨 → 마치 <code>CascadeType.REMOVE</code> 처럼 동작함
    
    </aside>
    

### 영속성 전이 + 고아 객체

- `CascadeType.ALL` + `orphanRemovel=true` ⇒ `@OneToMany(mappedBy = "parent", cascade=CascadeType.ALL, orpahRemoval = true)`
- **생명주기**에 따라 관리!
- **스스로 생명주기를 관리하는 엔티티**는 `em.persist()`로 영속화,
`em.remove()`로 제거
- 두 옵션을 모두 활성화 하면 **부모 엔티티를 통해서 자식의 생명주기를 관리**할 수 있음
    - 즉, 자식의 저장, 삭제가 부모로 인해 관리되는 것
    - 자식의 저장(영속) → `parent.getChilds().add(child)`
    - 자식의 삭제(제거) → `parent.getChilds().remove(0)`
    - 즉, **자식이 자기 의지대로 저장이나 삭제를 하지 않아도 알아서 영속되거나 제거됨**
- 도메인 주도 설계(DDD)의 Aggregate Root 개념을 구현할 때 유용