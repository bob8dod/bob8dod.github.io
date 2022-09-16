---
layout: post
title:  "Spring Data Jpa - 쿼리 메소드, 부가 기능"
date:   2022-09-08
image:  /posts/springData.jpg
tags:   SpringBoot JPA SpringData
---

## 복잡한 CRUD에 대한 Spring Data JPA 처리

- 기본 공통 인터페이스에서 제공하는 기능들은 말 그대로 기본 공통 CRUD
- 즉, **복잡한 CRUD에 대해선 직접 쿼리를 생성**해줘야 함
- 그럼 해당 **인터페이스를** **직접 구현해야 되는 것인가?** → 구현해야 되는 부분이 너무나도 많으며, 이미 Spring Data Jpa에서 구현체를 자동으로 만들어주고 있음
- 이때 필요한 것들이 하위 기능들
    - **메소드 이름으로 쿼리 생성**
    - **NamedQuery**
    - **@Query**
- 부가 설정 가능 부분
    - 페이징과 정렬
    - 벌크성 수정 쿼리
    - @EntityGraph

## 복잡한 CRUD 처리 (공통이 아닌 부분)

### 메소드 이름으로 쿼리 생성

- **메소드 이름을 분석**해서 **JPQL 쿼리 생성 및 실행**
- **정해진 규칙**에 따라 Spring Data Jpa가 메서드 이름을 **분석**하여 이에 맞는 **JPQL 쿼리를 생성하고 실행**해주는 것!
- 메서드 명에 Entity 에서 설정된 **필드명과 정해진 조건명 규칙에 따라 작성**만 해주면 되는 것 (정말 복잡한 조건이 아닌 이상 웬만한건 다 됨!) → [공식 문서](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods.query-creation)
- But, 조건이 길어질 수록 메서드 명이 길어지고, 메서드의 목적을 한눈에 파악하기가 어려움 (가독성 저하) → **조건이 많아지면 다른 방법을 선택**할 수 있음(**NamedQuery, @Query,** …)
- 예시
    - 회원 조회 by 이름, 나이
    - **순수 JPA (not Spring Data Jpa)**
        - method : `findByUsernameAndAgeGreaterThan(String username, int age)`
        - code
            
            ```java
            return em.createQuery("select m from Member m where m.username = :username and m.age > :age", Member.class)
            	 .setParameter("username", username)
            	 .setParameter("age", age)
            	 .getResultList();
            ```
            
        - **직접 JPQL을 짜야**되는 불편함과 **파라미터 바인딩, 반환 결과 설정** 등 **생각보다 많은 작업**을 해주어야 함
        - 또한 JPQL을 **직접 짜면서 오류 발생 가능성** 존재
    - **스프링 데이터 JPA**
        - method : `findByUsernameAndAgeGreaterThan(String username, int age)`
        - code : **스프링 데이터 JPA 가 자동으로 해당 메서드 이름을 분석해서 생성**해줌 
        → `find**By(**Name**And**(Age**GreaterThan**))` ⇒ `"select m from Member m where m.username = :name and m.age > :age"`
            - find → `select`
            - by → `where`
                - name → `member.name = :name`
                - and → `and`
                - age → `member.age`
                - greater than → `> :age`
    - 주요 규칙
        - **조회**
            - `find…By`, `read…By`, `query…By`, `get…By`
            - `…` 부분에는 해당 메서드가 어떤 목적으로 조회되고 있는 것인지에 대한 **설명(내용)이 들어가도 됨** → ex) `findTempBy…`
            - 반환타입 : **Entity** or **List<Entity>**, …
        - **COUNT**
            - `count…By`
            - 반환타입 : `long`
        - **EXISTS**
            - `exists…By`
            - 반환타입 : `boolean`
        - **DELETE**
            - `delete…By`, `reomve…By`
            - 반환타입 : `long`
        - **DISTINCT**
            - `find…DistinctBy`
        - **LIMIT**
            - `findFirst3`, `findFirst`, `findTop`, `findTop3`
        - 이외는 [공식 문서](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repositories.query-methods.query-creation) 참고

### 메서드 이름으로 JPA NamedQuery 호출

- **NamedQuery** 선언
    
    ```java
    @NamedQuery(
            name = "Member.findByUsername",
            query = "select m from Member m where m.name = :username")
    public class Member{...}
    
    ```
    
    - JPA 에서 지원하는 **NamedQuery** → **로딩 시점에 바인딩 및 검증, 재활용이 가능**하다는 장점 존재 → **SpringDataJpa 에서의 `@Query`**
    - 하지만, **정적인 쿼리만 동작**한다는 단점 존재
    - **Entity Class 단에 어노테이션**으로 정의
    - 호출될 이름과 쿼리를 해당 NamedQuery에 작성
- 사용
    
    ※ 실무에서는 잘 사용되지 않는 기능 → `@Query` 라는 기능이 존재하기에!!
    
    - **순수 JPA**
        
        ```java
        public List<Member> findByNameNamedQuery(String name) {
            return em.createNamedQuery("Member.findByUsername", Member.class)
                    .setParameter("username", name)
                    .getResultList();
        }
        ```
        
        - 파라미터도 직접 설정해주어야 하고 코드도 길어지며 여간 귀찮은 것이 아님
    - **Spring Data Jpa**
        
        ```java
        @Query(name = "Member.findByUsername") // 생략 가능 (메서드명을 잘 설정했다면)
        List<Member> findByUsername(@Param("username") String name);
        ```
        
        - `@Query(name = "Member.findByUsername")`
            - Entity 에 정의한 NamedQuery 연결
            - 생략 시 현재 JpaRepository 에 설정된 **Entity 와 Method 명을 통해 자동으로 찾아와** 연결해 줌 → Entity Type : `Member`, Method Name : `findByUsername` ⇒ `"Member.findByUsername"`
        
        <aside>
        ⚠️ <strong>@Query 없이 NamedQuery 인지 메서드명으로 쿼리 생성인지 어떻게 알지? → [동작 Rule에 따라]</strong>
        <ul>
        <li> 해당 메서드명 조건으로 NamedQuery 가 있다 → NamedQuery 로 동작 </li>
        <li> 없다 → Method 명으로 query 생성 → 관례를 따라서 자동으로 query 생성 </li>
        </ul>
        </aside>
        

### @Query (리포지토리 메서드에 쿼리 정의)

- **NamedQuery의 발전형**이라 생각하면 됨 (NamedQuery의 모든 장점을 가져오면서 **사용은 더 깔끔**하게.)
- NamedQuery는 Entity Class 단에 어노테이션으로 정의해야되지만, `@Query`는 **리포지토리 메서드에 바로 쿼리 정의**가 가능 (호출 이름도 정하지 않음)
- `@Query`
    
    ```java
    @Query("select m from Member m where m.name = :username and m.age = :age")
    List<Member> findUserByNameAndAge(@Param("username") String name, @Param("age") int age);
    ```
    
    - **직접 jpql 문을 작성하여 적용**하는 것.
    - NamedQuery 대신 이걸 **실무에서 많이 사용**함 → NamedQuery 의 장점을 가져가면서 **직접 Repo 에 jpql 을 작성**할 수 있음
    - 일명 이름이 없는 NamedQuery
    
    <aside>
    ⚠️ <strong>Tip!</strong>
    <br>
    실무에서는 메소드 이름으로 쿼리 생성 기능은 파라미터가 증가하면 메서드 이름이 매우 지저분해짐. 따라서 <code>@Query</code> 기능을 자주 사용
    
    </aside>
    
- **조회**
    - **단순 값** 조회
        
        ```java
        @Query("select m.username from Member m")
        List<String> findUsernameList();
        ```
        
        - JPA **값 타입**( … , `@Embedded`)도 해당 방식으로 조회 가능
    - **DTO로 직접** 조회
        
        ```java
        @Query("select new study.datajpa.dto.MemberDto(m.id, m.username, t.name) " +
         "from Member m join m.team t")
        List<MemberDto> findMemberDto();
        ```
        
        - JPA의 **new 명령어**를 사용 → `new study.datajpa.dto.MemberDto(m.id, m.username, t.name)`
        - **생성자가 맞는 DTO가 필요** → JPA의 JPQL사용방식과 동일
    - 파라미터 바인딩
        
        ```java
        @Query("select m from Member m where m.username = :name")
        Member findMembers(@Param("name") String username);
        ```
        
        - `@Param("name")` 을 통해서 바인딩
        - 위치기반도 존재하지만, 실수가 발생할 가능성이 크고 가독성이 떨어짐
        - 그래서 이름기반으로 바인딩하는 것을 권장
        - 컬렉션도 동일하게 바인딩 가능 (**in query** 에서 사용)
- **반환 타입**
    - 컬렉션(`List, Set, …`), 단건, 단건 Optional(`Optional<>`) 가능

## 부가 기능 (페이징, 벌크연산, …)

### 스프링 데이터 JPA 페이징과 정렬

- 순수 JPA 페이징과 정렬
    - Spring Data Jpa 를 사용하지 않고 순수 JPA로 페이징과 정렬
    - jpql을 통해 `.setFirstResult()` [**offset**] , `.setMaxResults()` [**limit**] 사용
    - 예시)
        - 나이가 10살인 회원을 이름으로 내림차순하며 페이지당 보여줄 데이터는 3건인 상황
        - 페이지에 따른 Content 와 총 count를 반환하고 싶은 상황
        - **페이징**
            
            ```java
            public List<Member> findByPage(int age, int offset, int limit) {
            	 return em.createQuery("select m from Member m where m.age = :age order by m.username desc", Member.class)
            		 .setParameter("age", age)
            		 .setFirstResult(offset)
            		 .setMaxResults(limit)
            		 .getResultList();
            }
            ```
            
            - `.setFirstResult(offset)` 을 통해서 가져올 **처음 순서**를 정하고 → **offset**
            - `.setMaxResults(limit)` 을 통해서 **가져올 양**을 정함 → **limit**
        - **총 COUNT**
            
            ```java
            public long totalCount(int age) {
            	 return em.createQuery("select count(m) from Member m where m.age = :age", Long.class)
            		 .setParameter("age", age)
            		 .getSingleResult();
            }
            ```
            
            - `count()` 사용
    - 순수 JPA를 이용한 페이징과 정렬은 물론 다양한 DB언어에 맞게 페이징 쿼리를 자동으로 날려주지만, **부가적인 기능을 제공하지 않아** 페이지 계산 공식 등 **개발자가 직접 복잡한 계산 공식을 사용하여 부가적인 기능을 구현해 내야 됨** → 스프링데이터JPA는 지원해줌!
- **스프링 데이터 JPA 페이징과 정렬**
    - **JPA에만 국한 된 것이 아닌**, 스프링 데이터 공통을 사용할 수 있게끔 **공통 표준으로 페이징과 정렬을 지원** (`org.springframework.data.domain.Pageable`)
    - **페이징**뿐만 아니라 **슬라이싱**으로도 반환 가능 (슬라이싱 : 더보기 형태의 슬라이스로 페이지를 구성하는 기능). 추가로 페이징기능을 사용해도 일반 **컬렉션**으로도 반환 가능
        - 페이징 : `org.springframework.data.domain.Page` : **추가 count 쿼리 실행, 결과 포함**
        - 슬라이싱 : `org.springframework.data.domain.Slice` : 추가 count 쿼리 없이 **다음 페이지만 확인 가능**
        - 일반 List (자바 컬렉션):  추가 count 쿼리 없이 **결과만 반환**
    - 페이징과 정렬 사용법
        
        ```java
        Page<Member> findByUsername(String name, Pageable pageable); // 페이징 : count 쿼리 사용
        
        @Query(value = "select m from Member m left join m.team t",
                    countQuery = "select count(m) from Member m")
        Page<Member> findByUsername(String name, Pageable pageable); // 페이징 : count 쿼리 사용
        
        Slice<Member> findByUsername(String name, Pageable pageable); // 슬라이싱 : count 쿼리 사용 안함
        List<Member> findByUsername(String name, Pageable pageable); // 일반 결과 : count 쿼리 사용 안함
        List<Member> findByUsername(String name, Sort sort); // 정렬만 사용. no paging
        ```
        
    - **메서드 이름으로 쿼리 생성 + 페이징**
        
        ```java
        Page<Member> findByAge(int age, Pageable pageable);
        ```
        
        - 기존에 알아봤던 ‘메서드 이름으로 쿼리 생성’에 **Pageable(paging 조건)**만 넣어주면 해당 이름으로 생성된 쿼리의 결과에 **page 조건을 자동으로 추가해서 query를 날리게 됨**
        - 사용
            
            ```java
            PageRequest pageRequest = PageRequest.of(0, 3, Sort.by(Sort.Direction.DESC, "name"));
            Page<Member> page = memberRepository.findByAge(10, pageRequest);
            Page<MemberDto> memberDtos = page.map(member -> new MemberDto(...));
            ```
            
            - **Pageable**에 **paging 조건**(`PageRequest pageRequest`)을 넣어주고 사용해주면 됨
            - `PageRequest pageRequest = PageRequest.*of*(page, size, Sort);`
                - **페이징 조건**
                - `page` : 원하는 페이지 (**0 부터** 시작)
                - `size` : 가져올 개수
                - `Sort` : 정렬 기준
            - `Page<Member> page = memberRepository.findByAge(10, pageRequest);`
                - 조건(`pageRequest`  → 첫번째 페이지, 3개의 content, 이름 오름차순으로 정렬) 에 맞는 결과 반환
                - **반환되는 결과** : `List<Member>` **결과 +** `totalCount` **결과** (query를 통해 확인할 수 있음)
                - `Page<Member>`로 반환 받지 않고 **일반적인 `List<Member>` 로도 반환 가능!** (페이지 조건을 달았지만, 페이징 기능을 쓰지 않는 결과를 받을 수 있는 것)
            - `Page<MemberDto> memberDtos = page.map(member -> new MemberDto(...));`
                - 추가로 항상 Entity는 Entity 자체로 반환해주면 안됨! → **DTO로 반환 필요**
                - Page는 `map`을 통해 쉽게 Entity를 **DTO로 변환** 할 수 있음 ⇒ `page.map()`
        - Page를 통해 쉽게 얻을 수 있는 **부가적인 정보**
            - `Page<Member> page = memberRepository.findByAge(10, pageRequest);` 를 통해 page를 얻었다고 가정
            - 반환 결과 **내용** : `List<Member> members = page.getContent();`
            - 해당 **조건을 따른 Entity의 전체 개수**(`count()` 사용) : `long totalElements = page.getTotalElements();`
            - **현재 페이지** 번호 : `int pageNum = page.getNumber();`
            - 전체 **페이지 개수**(복잡한 계산 과정을 거칠 필요가 없어짐) : `int totalPages = page.getTotalPages();`
            - 첫번째 페이지인지 확인 : `page.isFirst()`
            - 다음 페이지가 존재하는지 확인 : `page.hasNext()`
    - **메서드 이름으로 쿼리 생성 + 슬라이싱**
        
        ```java
        PageRequest pageRequest = PageRequest.of(0, 3, Sort.by(Sort.Direction.DESC, "name"));
        Slice<Member> page = memberRepository.findByAge(10, pageRequest);
        Page<MemberDto> memberDtos = page.map(member -> new MemberDto(...));
        ```
        
        - 나머지는 Page 와 동일. 즉, **반환값만 Slice로 바꿔주면 Slice로써 사용할 수 있음 (Slice는 Page의 하위)**
        - 다른점은 Slice로 반환할 시 **전체 count, 전체 page 개수 를 가져와 주지 않음** (쓸일이 없으니)
    - **`@Query` 를 통해 레포에 직접 쿼리 생성 + 페이지/슬라이싱**
        
        ```java
        @Query(value = "select m from Member m left join m.team t",
                    countQuery = "select count(m) from Member m")
        Page<Member> findByUsername(String name, Pageable pageable);
        ```
        
        - 메서드 이름으로 쿼리 생성 뿐만 아니라, `@Query`를 통해 **직접 쿼리를 생성하면서 Page 사용 가능!**
        - 추가로 **count Query를 분리하는 것**도 가능 (**실무에서 중요**)
            - `countQuery = "select count(m) from Member m"`
            - 항상 페이징 시 주의할 점은 "c**ontent를 얻고자 하는 query와 count 쿼리는 다를 수 있다**"는 것 → count 쿼리는 말 그대로 개수만 필요로 하는 것. 즉, **최적화가 가능**함 ⇒ content에서 필요로 할 수 도 있는 **join문이 count에서는 필요 없을 수도 있다**는 것! (Member에 Team을 Join해줘서 content를 만든다해도 **총 count는 사실 Member 만 있어도 됨!** → count에서는 join을 빼서 query를 날릴 수 있음 )
    - 결국 이런 **Page의 부가적인 기능,정보**를 통해 **웹의 페이지 기능을 구현**할 수 있음 (총 페이지 개수 보여주기, 다음 페이지로 넘길 수 있는지 확인 하기, … 등)
    - 추가로, **Pageable 기능은 어느 method(메소드이름으로, @Query 사용하여, … )에도 적용이 가능**함!

### 벌크성 수정 쿼리

- 벌크연산 : 변경 감지를 통해 UPDATE 쿼리를 날리는 것이 아닌, 변경할 것들에 대해 **한번에 UPDATE 쿼리를 in query로** 한방으로 날리는 것
- 주의점 : 벌크연산은 영속성컨텍스트를 거쳐가는것이 아니기에 **변경된 사항들을 반영해주기 위해선 항상 벌크연산 후에 영속성컨텍스트를 초기화**(`em.clear()`) 해주어야 함
- **순수 JPA**를 이용한 벌크 연산
    
    ```java
    public int bulkAgePlus(int age) {
    	 int resultCount = em.createQuery(
    		 "update Member m set m.age = m.age + 1" +
    		 "where m.age >= :age")
    			 .setParameter("age", age)
    			 .executeUpdate();
    	 return resultCount;
    }
    ```
    
    - `executeUpdate()` : 벌크성 쿼리를 날리기 위한 설정
    - 순수 JPA는 해당 method를 실행해준 직후에 **직접 `em.clear()` 를 호출하여 영속성 컨텍스트를 비어줘야 하는 단점** 존재
- **스프링 데이터 JPA**를 사용한 벌크 연산
    
    ```java
    @Modifying(clearAutomatically = true)
    @Query("update Member m set m.age = m.age + 1 where m.age >= :age")
    int bulkAgePlus(@Param("age") int age);
    ```
    
    - `@Modifying` : 벌크성 쿼리를 날리기 위한 설정 (JPA의 `executeUpdate` 역할). 해당 `@Query`에 지정된 update query를 날려줌
        - 사용하지 않으면 예외 발생
        - `clearAutomatically = true` : 해당 벌크연산이 **끝난 직후 영속성 컨텍스트를 비워줌** (false가 default 이며, 벌크연산이 영속성컨텍스트에 Entity가 존재하지 않은 상태에서 이루어지는 것이면 false로 진행해도 됨) → **변경된 점 반영 가능**

<aside>
⚠️ <strong>memberRepository를 통해 사용중인 EntityManger(영속성컨텍스트)와 해당 Test단에서 새로 불러 오는 EntityManger는 같나?</strong> <br>
⇒ 같은 Transaction 안에 있으면 항상 같은 Entity Manger를 사용하기 때문에 같음! ⇒ 즉, em을 따로 불러와도 memberRepository와 동일 한 em이 할당 됨!

</aside>

### @EntityGraph (Fetch Join)

- **연관된 Entity 들을 SQL 한번에 조회**하는 방법
- Lazy 로딩인 연관 Entity들을 **마치 Eager 로딩과 같이 조회**하는 것 (**Fetch Join**)
- 특징 : INNER JOIN이 아닌 **LEFT OUTER JOIN** 실행
- **N+1 문제의 해결법**
    - N+1 문제
        - 예를 들어 Member를 조회하는 데, 조회 후 연관 Entity인 Team의 정보도 함께 확인한다. (이때 Member와 Team은 다대일 관계, **Lazy 로딩**)
        - `findAll()` 로 Member를 모두 조회한 후 Member를 훑으면서 각 Member의 Team의 이름을 조회
            
            ```java
            List<Member> members = memberRepository.findAll();
            for (Member member : members) {
            	 member.getTeam().getName();
            }
            ```
            
        - 이때, **N+1 문제** 발생 → **전체 Member를 조회**하는 query **1개** (`findAll()`) + **N개의 각 Member에서 Team을 조회**하는 query **N개**(`member.getTeam().getName();` x N)
    - **해결법 1 :** `@Query` **+ Fetch Join 사용**
        
        ```java
        @Query("select m from Member m left join fetch m.team")
        List<Member> findMemberFetchJoin();
        ```
        
        - m.team과 관련된 Team Entity를 가져와 **fetch join** 시켜 주는 것 → Eager Loading처럼 동작(fetch 까지 진행했기에, **member의 team field(Team Entity) 를 채워**줌)
    - **해결법 2 : `@EntityGraph` 사용**
        - 스프링 데이터 JPA에서 JPA가 제공하는 **엔티티 그래프 기능을 편리하게 사용** 가능 → **JPQL 없이 페치 조인을 사용 가능!**
        - `@EntityGraph(attributePaths = {"team"})`
            - `attributePaths = {”team”}` : 조회하고자하는 객체의 **team field(연관 Entity)**를 Fetch Join을 통해서 가져온다는 뜻
            - 해당 기능을 설정해주면 JPQL에서 **Fetch Join을 사용한 것과 동일한 Query**가 나가게 됨 (fetch join을 **편리하게 사용**할 수 있음!)
        - 유연한 사용법
            - **공통 인터페이스에서 제공하는 메서드를 오버라이드** 해서 사용
                
                ```java
                //공통 메서드 오버라이드
                @Override
                @EntityGraph(attributePaths = {"team"})
                List<Member> findAll();
                ```
                
            - **JPQL로 직접 쿼리를 작성하고 Fetch Join을 사용하지 않고** `@EntityGraph`를 사용
                
                ```java
                @EntityGraph(attributePaths = {"team"})
                @Query("select m from Member m")
                List<Member> findMemberEntityGraph();
                ```
                
            - “**메서드 이름으로 쿼리 생성**”에서 `@EntityGraph` 사용
                
                ```java
                @EntityGraph(attributePaths = {"team"})
                List<Member> findByUsername(String username)
                ```
                
                - `@EntityGraph` 의 사용 효율이 극대화되는 방법
                - 직접 JPQL을 짜지 않고도 간단히 Fetch JOIN 을 사용할 수 있게됨
                - 즉, 간단히 메서드 명으로도 만들 수 있는 조회에 대해서 Fetch Join을 사용하고 싶을 때 자주 사용됨
- Spring Data Jpa 만의 기능이 아닌 원래 JPA 표준 기능
    - 즉, **JPA에서도 사용 가능** → `NamedEntityGraph`
    - `NamedEntityGraph`
        - 선언
            
            ```java
            @NamedEntityGraph(name = "Member.all", attributeNodes =
            @NamedAttributeNode("team"))
            @Entity
            public class Member {}
            ```
            
            - 이름을 설정하고 어떤 Field의 연관객체를 한번에 가져올 지 설정
        - 사용
            
            ```java
            @EntityGraph("Member.all")
            @Query("select m from Member m")
            List<Member> findMemberEntityGraph();
            ```
            
            - 설정된 정보를 가지고 Fetch Join 실행

### JPA Hint & Lock

- HINT
    - JPA Query 힌트 (SQL에 영향을 주는 HINT가 아닌, **JPA 구현체에게 제공되는 HINT**)
    - `@Transcational(readOnly = true)` 와 동일한 역할을 하는 기능
    - 사용
        
        ```java
        @QueryHints(value = @QueryHint(name = "org.hibernate.readOnly", value ="true"))
        Member findReadOnlyByUsername(String username);
        ```
        
        - `@QueryHints` : 적용할 `@QueryHint` 모음
        - `@QueryHint(name = "org.hibernate.readOnly", value ="true")` : Entity를 가져올 때, readOnly mode로 가져오겠다는 설정
        - `readOnly`로 가져오기 때문에 **변경감지가 이루어지지 않음!**
    
    <aside>
    ⚠️ <strong>readOnly 를 사용하는 이유</strong> <br>
    원래 JPA는 Entity를 DB에서 가져오면 추후 변경감지 등을 위해 스냅샷(초기 Entity모습)을 생성하고 <code>flush()</code>가 발생하면 각 field를 비교하여 변경된 점에 대한 UPDATE Query를 날림. 하지만, 해당 Entity가 읽기모드라면, 말 그대로 변경할 것이 아니기 때문에 <b>스냅샷을 생성할 필요가 없어지고, field 비교 또한 이루어지지 않음 → 성능 최적화 가능!</b>
    
    </aside>
    
- LOCK
    - SQL의 **select for update**
    - **동시성 제어**를 위하여 특정 데이터(ROW)에 대해 **베타적 LOCK을 거는 기능**
    - "**데이터 수정하려고 SELECT 하는 중이야! 다른 사람들은 데이터에 손 대지 마!**" 의 느낌 (영화 좌석 예매를 위한 선택 등에 사용됨)
    - 사용
        - Spring Data JPA
            
            ```java
            @Lock(LockModeType.PESSIMISTIC_WRITE)
            List<Member> findByUsername(String name);
            ```
            
        - SQL 결과
            
            ```sql
            select *
            from member m
            where m.username = :username for update
            ```