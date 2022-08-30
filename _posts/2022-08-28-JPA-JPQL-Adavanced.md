---
layout: post
title:  "JPQL - Advanced"
date:   2022-08-28
image:  /posts/jpa.png
tags:   SpringBoot JPA
---

## JPQL 이란?

### JPQL의 필요성

- 일반적인 조회(`em.find()`, 객체그래프 탐색)가 아닌, **조건이 들어간 조회**가 필요하다면?
- 모든 데이터를 조회하고 앱단에서 걸러야 하나? → NO!
- **필요한 데이터만 DB에서 불러 오는 것!** (**객체 중심**으로 조회!) → 이 때 결국 **검색 조건이 포함된 SQL**이 필요 ⇒ **JPQL 사용** 필요!

### JPQL

- **Java Persistence Query Language**
- **SQL을 추상화한 객체 지향 쿼리 언어**
- SQL과 문법이 유사함 → SELECT, FROM, WHERE, GROUP BY, HAVING, JOIN, …
- 핵심은 **Entity 객체를 대상으로 쿼리**를 한다는 것 (SQL은 데이터베이스 테이블을 대상으로 쿼리) 
⇒ **JPQL을 SQL로 번역하여 실제 DB로 보냄**
    - JPQL
        
        ```java
        // String jpql = "select m from Member m where m.age > 18";
        List<Member> result = 
        			em.createQuery(
        					"select m from Member m where m.age > 18", 
        					Member.class).getResultList();
        ```
        
        - `.cerateQuery` : JPQL을 통해 query 생성
        - `jpql` : **Entity 객체를 중심으로 query를 생성**할 수 있음
            - `select m` : 기존 SQL같은 경우 내가 조회할 항목(Col)들을 모두 작성해주어야 하지만 **JPQL은 Entity 중심이기에 해당 Entity로 반환**할 수 있음
        - `Member.class` : 반환할 Entity 지정
    - JPQL에 따라 실행된 SQL query
        
        ```sql
        select
        	m.id as id,
        	m.age as age,
        	m.USERNAME as USERNAME,
        	m.TEAM_ID as TEAM_ID
        from
        	Member m
        where
        	m.age>18
        ```
        
- 즉, 복잡한 쿼리를 사용할 때 발생할 수 있는 **JPA와 DB의 패러다임 차이를 극복**해주는 것
- 실무에서 사용되는 **복잡한 쿼리를 객체 지향적으로** 짤 수 있도록 지원 → 즉, 조회 시 DB Table이 아닌 **Entity 객체를 대상**으로 검색
- JPA가 지원하는 다양한 쿼리 방법 중 하나
    - JPA가 지원하는 다양한 쿼리
        - 자바 코드를 통해 **JPQL을 동적으로 빌드해주는 Genterator** (동적으로 query를 변경하게 할 수 있음)
            - JPA Criteria : JPA 표준 스펙. 사용이 복잡하며 실용성이 없음
            - **QueryDSL** : 오픈 소스. 실무에서 **동적인 query를 짤 때 자주 사용하는 기술**
        - **Native SQL** : JPQL을 사용해도 DB 종속적인 코드가 필요한데, 그 때 **SQL 문법 그대로 사용할 수 있게끔 지원**하는 것
- JPQL은 JDBC API 직접 사용 or MyBatis, SpringJdbc와 함께 사용이 가능함
- SQL을 추상화하기 때문에 **특정 데이터베이스 SQL에 의존X**

## JPQL 고급

### 경로 표현식

- “.”을 찍어 객체 그래프를 탐색하는 것
    
    ```java
    String query = "select m.username " +
                    "from Member m " +
                     "join m.team t " +
                      "join m.orders o " +
                       "where t.name = 'TeamA'";
    ```
    
    - **상태 필드**
        - `m.username` : 상태 필드 접근
        - 단순히 값을 저장하기 위한 필드
        - **경로 탐색의 끝** → 더 이상 탐색 X
    - **단일 값 연관 필드**
        - `m.team` : 단일 값 연관 필드 접근
        - **대상이 Entity**인 필드 → `@ManyToOne`, `@OneToOne`
        - m.team의 Team을 가져오기 위해 **묵시적 내부 조인** 발생 → 더 탐색 가능
            - 묵시적 조인 발생 예시
                
                `select m.team from Member m`
                
                ```sql
                select m.*
                from Member m
                inner join Team t on m.team_id = t.id
                ```
                
        - BUT 해당 **묵시적 내부 조인은 항상 피해**야 됨!
    - **컬렉션 값 연관 필드**
        - `m.orders` : 컬렉션 값 연관 필드 접근
        - 대상이 **Entity들의 모음인 Collection인 필드** → `@OneToMany`, `@ManyToMany`
        - **묵시적 내부 조인 발생** → **더 이상 탐색 X** (`select t.members.username from Team t` → 실패)
        - BUT! **From 절에서 명시적 조인을 통해** 해당 Entity에 접근한다면 탐색 가능 → `select m.username from Team t join t.members m` → 성공

<aside>
⚠️ <strong>묵시적 조인  vs 명시적 조인</strong>
<ul>
<li><strong>묵시적 조인</strong></li>
    <ul>
    <li> <code>select m.team from Member m</code> </li>
    <li> 해당 연관된 Entity의 값을 가져오기 위해 <b>묵시적으로 Inner Join을 실행</b> 하는 것 (내부 조인만 가능) </li>
    <li> 말 그대로 묵시적으로 JOIN이 이루어지기에 추후 <b>최적화를 위한 쿼리 튜닝 시 어려워 짐</b> (나중에 묵시적 JOIN에서 Join이 일어났다는 것을 인지하기가 어려워 짐) </li>
    <li> 묵시적 조인은 사용하면 안됨!!!! → <b>명시적 JOIN을 사용하자!</b> </li>
    </ul>
<li><strong>명시적 조인</strong></li>
    <ul>
    <li> <code>select m from Member m join m.team t</code> </li>
    <li> <b>Join 키워드 직접 사용</b> </li>
    </ul>
</ul>
</aside>

### 페치 조인(Fetch Join)

- 개념
    - 실무에서 굉장히 중요한 기능 (**1+N 문제의 해결법 → 한번에 가져오기 →** **Fecth Join**)
    - SQL의 JOIN 종류(Inner Join, Left Join, … ) 가 아님!!
    - JPQL에서 **성능 최적화**를 위해 제공하는 **JPQL 전용 기능**
        - 연관된 Entity 나 Collection을 **SQL 한 번에 함께 조회**하는 기능 → query가 2번 나갈걸 1번 나가게 끔 해주는 기능
    - `join fetch` 명령어 사용
    - Fetch Join → `[ Left [OUTER] | INNER ] JOIN FETCH ___`
- 동작
    - 해당 **FETCH JOIN 된 Entity까지 같이 조회** (SQL 한 번에) ↔ 일반 Join은 연관된 Entity를 함께 조회하진 않음
    - ex)
        - JPQL : `select m from Member m join fetch m.team`
        - SQL : `SELECT **M.*, T.*** FROM MEMBER M INNER JOIN TEAM T ON M.TEAM_ID=T.ID`
        - JPQL로는 team을 Join 해서 m만 조회했지만, **결과적으로 t 까지 같이 조회**됨
        - 결국 **연관된 team이 채워진 Member Entity를 얻어**낼 수 있는 것
    - **동작 자체는 즉시로딩(fetch = Eager)과 동일**!
    - fetch join은 내가 지금 **어떤 객체그래프를 한번에 조회해올 것인지를 명시적으로 정하여 조회**하는 것 → **객체 그래프를 SQL 한번에 조회**하는 개념
    - 추가로 **지연로딩** 시 발생하는 **1+N 문제 해결** 방법! (**한번에 가져와**버리기)
- **단일 연관관계 Fetch Join**
    
    ```java
    String jpql = "select m from Member m join fetch m.team";
    List<Member> members = em.createQuery(jpql, Member.class)
    	 .getResultList();
    
    for (Member member : members) {
    	 System.out.println("username = " + member.getUsername() + ", " 
    		+ "teamName = " + member.getTeam().name());
    }
    ```
    
    - `join fetch` : Fetch Join 사용 → 한번에 가져와버리기
        - 만약 **Team이 없는 Member를 포함해서 조회**하고 싶다면 `left join fetch` 를 통해 **Outer Fetch Join 을 적용**하면 됨
    - `member.getTeam().name()` : 연관객체 접근
        - Lazy로딩 상태이기에 **Fetch Join을 사용하지 않았다면** Proxy 객체의 초기화로 인해 **1+N 문제가 발생!**
        - Fetch Join을 사용했기에 애초에 Proxy 객체가 아닌 **실제 Entity로 채워져 있어 query가 1개만 발생함**! → **1+N 문제 해결**
        - 즉, 애초에 Lazy로딩이 아닌 **즉시로딩처럼 접근**하는 것
- **컬렉션 연관관계 Fetch Join**
    
    ```java
    String jpql = "select t from Team t join fetch t.members";
    List<Team> teams = em.createQuery(jpql, Team.class)
    	 .getResultList();
    
    for(Team team : teams) {
    	 System.out.println("teamname = " + team.getName() + ", team = " + team);
    	 for (Member member : team.getMembers()) {
    		 System.out.println(“-> username = " + member.getUsername()+ ", member = " + member);
    	 }
    }
    ```
    
    - `join fetch` : Fetch Join 사용 → 한번에 가져와버리기
        - 만약 Members가 없는 Team을 포함해서 조회하고 싶다면 `left join fetch` 를 통해 **Outer Fetch Join** 을 적용하면 됨
    - `member.getUsername()` : 연관객체 접근
        - Lazy로딩 상태이기에 **Fetch Join을 사용하지 않았다면** Proxy 객체의 초기화로 인해 **1+N 문제**가 발생!
        - Fetch Join을 사용했기에 애초에 Proxy 객체가 아닌 **실제 Entity로 채워**져 있어 query가 1개만 발생함! → 1+N 문제 해결
        - 즉, 애초에 **Lazy로딩이 아닌 즉시로딩처럼 접근**하는 것 → 페치 조인으로 팀과 회원을 함께 조회해서 지연 로딩 발생 안함
    - **여기서 주의 점**
        - 만약 TeamA에 속해 있는 Member가 2명이다? → 위의 fetch join으로 조회 시 TeamA가 두번 조회 됨.
        - 이는 **일대다 join 의 특성 상** 어쩔 수 없는 것 → **App단에서 처리**가 필요함

          ![Untitled]({{site.baseurl}}/images/posts/post-220828/Untitled.png)
            
            - Group By가 아니기 때문에 Member 개수에 맞게 TeamA가 mapping되어 2개로 조회가 되는 것
            - 이는 **DISTINCT로 해결** 가능
- **일대다 Fect Join과 DISTINCT**
    - JPQL **DISTINCT** 로 **일대다 FETCH JOIN 시의 중복 문제를 해결**할 수 있음
    - SQL의 DISTINCT : **완벽히 일치하는 중복 데이터**를 제거
    - **JPQL의 DISTINCT**
        - 기본적으로 SQL에 DISTINCT 추가
        - APP단에서 **Entity 중복 제거 (식별자를 통해)**

          ![Untitled]({{site.baseurl}}/images/posts/post-220828/Untitled 1.png)
            
- **[중요]** **페치 조인(Fetch Join)의 한계**
    - 페치 조인은 항상 그 특징과 주의점(한계)을 잘 알고 사용해야 됨
    - **Fetch Join 의 한계**
        - **Fetch Join의 대상에는 별칭을 줄 수 없음** → `join fetch t.members m` X (hibernate 구현체에선 사용 가능하나 예기치 못한 버그가 발생할 수 있으므로 가급적이면 자제하는 것이 좋음!)
        - **둘 이상의 컬렉션은 Fetch Join 불가능!** → 가능은 하나 가급적 사용 X  (일대다 도 데이터가 뻥튀기가 되는데 일대다에 또 일대다 를 적용할 시 데이터가 예기치 못하게 수없이 뻥튀기가 될 수 있음, 데이터 매핑도 제대로 안되는 경우가 있음)
        - **컬렉션을 패치 조인하면 페이징 API(setFirstResult, setMaxResults)를 사용 못함**
            - 일대일, 다대일 같은 단일 값 연관 필드 FETCH JOIN은 데이터 뻥튀기가 되지 않기 때문에 페이징 가능
            - **일대다 같은 컬렉션 값 연관 필드 FETCH JOIN은 데이터 뻥튀기가 되기 때문에** 페이징 불가능 → **페이징은 SQL단에서 이루어지는 것**이기에 JPQL DISTINCT로도 막을 수 없음 (JPQL DISTINCT는 App단에서 실행됨) [hibernate 구현체는 가능은 하나 메모리 단에서 실행되기에 메모리 초과가 발생함]
    - **컬렉션 Fetch Join 페이징 해결법**
        - 일대다 FETCH JOIN을 **다대일 FETCH JOIN으로 조회**한 후 페이징 : `select t from Team t join fetch t.members;` → `select m from Member m join fetch m.team;`
        - **[중요] BATCH 조회**
            - **FETCH JOIN 없이 조회**하여 페이징 하는데 **LAZY로딩의 Entity들은 BATCH_SIZE를 이용하여 조회하여 최적화**하기
            - **필요로 하는 Entity를 한번의 in query SQL로 묶어서 가져**오는 것
                - 지연로딩 시의 1+N 문제에서 **N에 해당하는 것들을 한번에 in query로 가져**옴 → `where m.team_id in (?,?,?, … )`
            - `@BatchSize` 로 Entity 단위로 적용 하거나 **Global Setting**으로 전체 Entity에 적용 가능
            - `@BatchSize` (Entity 속 `@OneToMay` 관계에 적용)
                
                ```java
                // Team Entity
                @BatchSize(size = 100)
                @OneToMany(mappedBy = "team")
                private List<Member> members = new ArrayList<>();
                ```
                
                - `@BatchSize(size = 100)` : 해당 관계의 Entity를 Lazy로딩으로 가져올 때 최대 100개를 한번에 가져온다는 뜻 (**in query를 사용**하여)
            - **Gloabl Setting**
                - property.xml 에서 JPA 설정
                - `<property name=”hibernate.default_batch_fetch_size” value=”100”/>`
                - **모든 일대다 관계에서 Lazy 로딩 조회 시 in query 로 한번에 가져오겠다**는 설정

### 다형성 쿼리

- **TYPE**
    - **조회 대상을 특정 자식으로 한정**
    - `type(_)`
    - 예시) 부모 Entity Item 의 자식 중 Book, Movie 만 조회
        
        `select i from Item i where type(i) in (Book, Movie)` →
        
        ```sql
        select i from Item i
        where i.DTYPE in ('B','M')
        ```
        
- **TREAT**
    - Java의 **타입 캐스팅**과 유사 (업,다운 캐스팅)
    - 상속 구조에서 **부모 타입을 특정 자식 타입으로** 다룰 때 사용 → ex) 부모 타입을 자식 타입으로 바꿔 필드 탐색을 진행할 때
    - FROM, WHERE, SELECT(hibernate 구현체) 에서 사용
    - `treat( 부모 as 자식 )`
    - 예시) 부모로 조회하는데 조건에 자식의 필드를 사용하고 싶을 때
        
        `select i from Item i where treat(i as Book).auter = kim` → 
        
        ```sql
        select i.* from Item i
        where i.DTYPE = ‘B’ and i.auther = ‘kim’
        ```
        

### 엔티티 직접 사용

- 데이터가 들어갈 자리에 엔티티를 직접 넣게 될 때의 상황
- **엔티티를 직접 사용**하게 되면 **기본 키(PK)값이 들어가게 됨**
    
    JPQL : `select count(m) from Member m` → SQL : `select count(m.id) as cnt from Member m` [`count(m) → count(m.id)`]
    
- **기본 키(PK)**
    - **엔티티를 파라미터로 전달**하는 경우
        
        ```java
        String jpql = “select m from Member m where m = :member”;
        List resultList = em.createQuery(jpql)
        					.setParameter("member", member)
        					.getResultList();
        ```
        
    - 식별자를 직접 전달하는 경우
        
        ```java
        String jpql = “select m from Member m where m.id = :memberId”;
        List resultList = em.createQuery(jpql)
        					.setParameter("memberId", memberId)
        					.getResultList();
        ```
        
    - 결과 SQL
        
        `select m.* from Member m where m.id=?`
        
    - 중요한 것은 엔티티를 파라미터로 전달하는 경우 where m.id 가 아닌 그냥 m 임 → 즉, **엔티티를 직접 사용하면 조건 자체도 엔티티로 판별**해야됨 (SQL은 항상 식별자로 판단됨)
- **외래 키(FK)**
    - 외래 키도 기본 키 사용법과 동일
    - 엔티티를 파라미터로 전달 : `where m.team = :team` , `.setParameter("team", team)`
    - 식별자를 직접 전달 : `where m.team.id = :teamId` , `.setParameter("teamId", teamId)`
    - 실행된 SQL
        
        `select m.* from Member m where m.team_id=?`
        

### Named 쿼리

- **미리 query를 정의**해서 이름을 부여해두고 **사용하는 JPQL**
- **정적 쿼리** (동적 쿼리 불가능)
- 어노테이션으로 정의 or XML에 정의 해서 사용
- **Application 로딩 시점에 초기화 후 재사용 가능** → SQL로 파싱한 후부터는 다시 파싱하지 않고 재사용 (성능 최적화)
- **[중요] Application 로딩 시점에 쿼리를 검증** → App이 돌아가는 와중에 오류 뜰 일이 없음
- 정의
    
    ```java
    @Entity
    @NamedQuery(
    		 name="Member.findByUsername",
    		 query="select m from Member m where m.username = :username")
    public class Member {
    		 ...
    }
    ```
    
    - 어노테이션 기반으로 정의.
    - 실행될 query와 사용될 이름을 정의
    - query 속에는 파라미터 설정도 가능
- 사용
    
    ```java
    em.createNamedQuery("Member.findByUsername", Member.class)
    			.setParameter("username", "park")
    			.getSingleResult();
    ```
    
    - `em.createNamedQuery()` : 정의된 namedQuery를 불러오는 함수. 정의된 이름으로 query를 불러옴
    - 그 이후로는 기존 createQuery와 사용법 동일
- **[중요]** `@NamedQuery` 는 **Spring Data JPA 에서 `@Query` 로 사용**됨 !! (실무에선 `@Query`가 굉장히 많이 사용됨 → **빌드 시점에 SQL 파싱을 하기 때문에 오류를 빌드시점에 찾아낼 수 있고 또한 재사용이 가능**하기에!)

### 벌크 연산

- **한번에 여러 개의 Entity를 수정하고 이를 DB에 반영**하고 싶다면?
    - 기존의 수정 방법인 **변경 감지**를 이용하여 진행한다면 **각 Entity별로 query가 나가기 때문에 비효율적!**
    - 이럴 때 사용하는 것이 **벌크 연산**
- **벌크 연산** : **쿼리 한번으로 여러 Table Row(Entity)를 변경**
- `executeUpdate()`
    - 표준으로 UPDATE, DELETE 지원
    - hibernate 구현체는 INSERT 까지 지원
    - **반환 값은 영향받은 엔티티의 수** (int)
- 사용
    
    ```java
    String qlString = 
    		 "update Product p " +
    		 "set p.price = p.price * 1.1 " +
    		 "where p.stockAmount < :stockAmount";
    
    int resultCount = em.createQuery(qlString)
    											 .setParameter("stockAmount", 10)
    											 .executeUpdate();
    ```
    
    - 기존에 사용했던 JPQL과 비슷하지만 마지막에 `.executeUpdate()` 로 **벌크 연산을 수행**하는 것이 다름
    - 해당 벌크 연산을 통해 10개 이하의 모든 상품들은 가격이 10프로 상승하게 됨
    - 만약 해당 조건에 맞는 상품들이 100개 라면 기**존의 변경감지를 사용했을 때는 100개의 쿼리**가 나가지만, **벌크 연산을 통해 진행하면 1개의 쿼리만**이 나감 → 성능 최적화 Good!!
- **벌크 연산 사용 주의점**
    - 벌크 연산은 **영속성 컨텍스트를 무시하고 DB에 직접 쿼리를 날리는 방법** 이므로 항상 주의 해야됨 ( JPA는 영속성 컨텍스트와 함께 동작하는 애이므로 )
    - 해결
        - **벌크 연산을 먼저 실행**하는 방법
            - 영속성 컨텍스트 작업을 진행하지 않고 그냥 **가장 먼저 벌크 연산을 실행**해서 영속성 컨텍스트에 영향을 주지 않는 방법
        - **벌크 연산 수행 후 영속성 컨텍스트를 초기화(`em.clear()`)**하는 방법
            - 벌크 연산도 JPQL로 동작하기에 `flush()`는 됨
            - 하지만, 초기화는 되지 않기에 따로 **영속성 컨텍스트를 초기화하는 것이 중요**
            - **초기화의 필요성** : 영속성 컨텍스트를 초기화하지 않으면 벌크 연산 이후에 조회한 Entity가 만약 벌크 연산 이전에 조회되어 영속성 컨텍스트에 있다고 하면, 영속성 컨텍스트가 초기화되지 않았기 때문에 **변경 이전의 Entity로 반영이 됨. (1차 캐시)**
            - 그러므로 **벌크연산이 반영된 Entity의 결과로 조회하려면 영속성 컨텍스트는 초기화가 필수!**
    
    <aside>
    ⚠️ <strong>[참고] JPQL이 flush만 하고 초기화하지 않는 데 벌크 연산은 초기화를 진행해야되는 이유</strong>
    <ul>
    <li> <strong>JPQL : flush 자동 O , clear 필수 X</strong> <br>
    기본 JPQL은 사실 조회로직이 대다수 이며 <b>JPQL을 통해서 조회 시 항상 바로 SQL문이 실행되어 바로 DB에서 가져오</b>기 때문에 그 이전의 영속성 컨텍스트를 통해 변경되거나 저장된 애들에 대한 <b>반영이 필요하기에 flush가 동작</b>함. 즉, 조회 로직만이 수행되기 때문에 초기화가 따로 필요 없는 것 (<b>이미 영속성 컨텍스트에 반영된 내용</b>이므로) <br> (+ JPQL을 통해 INSERT,UPDATE,DELETE 시 초기화 필수!) </li>
    <li> <strong>벌크 연산 : flush 자동 O, claer 필수 O</strong> <br>
    하지만, <b>벌크 연산같은 경우 UPDATE 쿼리가 나감</b>. 이때 UPDATE 로직은 <b>영속성 컨텍스트를 거치지 않고 바로 DB로 SQL을 날리기 때문에 변경내용이 영속성 컨텍스트에 반영되지 않음</b>. 이에 따라 벌크 연산 이후 Entity를 조회할 시 <b>이전의 영속성 컨텍스트에 남아 있는 (변경이 반영되지 않은) Entity를 가져오게 됨</b>. 그러므로 벌크 연산 시 항상 <b>초기화가 필요</b> </li>
    </ul>
    </aside>
    
- **[참고]** 해당 **벌크 연산**은 **Spring Data JPA 에선 `@Modifiying` 으로 편리하게 사용** 가능 (원리가 동일!)