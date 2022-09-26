---
layout: post
title:  "Querydsl - In Practice"
date:   2022-09-26
image:  /posts/querydsl.png
tags:   SpringBoot JPA Querydsl
---

## 순수 JPA와 Querydsl

순수 JPA와 Querydsl의 차이점을 확인하며 **Repository 생성**해보기

### Querydsl을 이용한 리포지토리

- `MemberQueryRepository` 생성
- Class단 Annotation
    
    ```java
    @Repository
    @RequiredArgsConstructor
    public class MemberQueryRepository {...}
    ```
    
    - `@Repository` : 컴포넌트로 등록, 하위 JPA의 오류를 **상위 Spring의 오류로 변경**해줌
    - `@RequiredArgsConstructor` : private final 로 설정된 field에 대해 **자동으로 DI실행** → 생성자를 통해
- 의존성 주입 (DI)
    
    ```java
    private final EntityManager em;
    private final JPAQueryFactory queryFactory;
    ```
    
    - `EntityManager em;`
        - **EntityManger** 주입
        - Spring에서 주입해주는 것이 아닌 **JPA에서 주입**해주는 것
        - `@PersistanceContext`로 **주입** 가능
    - `JPAQueryFactory queryFactory;`
        - **JPAQueryFactory** 주입
        - **[중요]** **항상 이용되는 EntityManger를 넣어주어야 함**
        - **JPAQueryFactory**를 **Bean으로 등록**하여 **Spring에서 주입**할 수 있게 설정 가능
            
            ```java
            @PersistenceContext
            private EntityManager em;
            
            @Bean // 빈으로 등록 -> 의존성 주입 가능
            JPAQueryFactory jpaQueryFactory() {
                return new JPAQueryFactory(em);
            }
            ```
            
        - 혹은 각각의 단에서 **생성자를 통해 em을 넣어줌**과 함께 생성할 수 있음
            
            ```java
            @PersistenceContext
            private EntityManager em;
            private final JPAQueryFactory queryFactory;
            
            public MemberJPARepository() {
                this.queryFactory = new JPAQueryFactory(em);
            }
            ```
        
    <aside>
    💡 <strong>EntityManger와 JPAQueryFactory의 동시성 문제</strong>
    <ul>
    <li> <b>EntityManger</b>는 <b>싱글톤</b>으로 운영되기 때문에 항상 같은 <b>EntityManger</b>를 받게됨 </li>
    <li> 각 메서드에 대한 <b>EntityManger</b>은 <b>Transcation 단위로 동작</b>하는 것 → <b>동시성 문제발생 X</b> </li>
    <li> <b>JPAQueryFactory</b> 또한 <b>싱글톤</b>으로 운영 </li>
    <li> <b>JPAQueryFactory</b>는 <b>EntityManger</b>을 받아서 동작하기 때문에 <b>똑같이 동시성 문제 X</b> </li>
    </ul>
    </aside>
    
    - **저장**
        
        ```java
        public void save(Member member) {
            em.persist(member);
        }
        ```
        
        - Querydsl로 코드를 작성하는 것보다 **영속성 컨텍스트까지 고려하여 JPA의 기본 저장** 로직인 `em.persist()`를 사용
    - 식별자 **조회**
        
        ```java
        public Optional<Member> findById(Long id) {
            Member member = em.find(Member.class, id);
            return Optional.ofNullable(member);
        }
        ```
        
        - Querydsl로 코드를 작성하는 것보다 **JPA에서 기본으로 제공**하는 `em.find()` 이용 → 더 편리
- **전체 조회**
    - **순수 JPA**
        
        ```java
        public List<Member> findAll() {
            return em.createQuery("select m from Member m", Member.class)
                    .getResultList();
        }
        ```
        
        - **직접 문자열로 JPQL을 작성**해주어야 함 → 개발 편의성 DOWN
        - **Class** 를 **직접 지정**해주어야 함
        - 문자열의 JPQL은 **컴파일 단계에서 오류 검출이 불가능** → 사용자가 직접 해당 method에 접근 시 오류 확인 가능 → 굉장히 불완전한 개발
    - **Querydsl 이용**
        
        ```java
        public List<Member> findAll_Querydsl() {
            return queryFactory
                    .selectFrom(member) // QMember.member
                    .fetch();
        }
        ```
        
        - `member` → `QMember.member` 를 **static import**
        - **자바 코드**로써 SQL을 작성할 수 있음 → 개발 편의성 UP
        - Class 직접 지정할 필요 없이, **Entity의 Q-Type으로 동작**
        - **Application 로딩 시점에 오류 확인 가능!** (**컴파일 단계에서 오류 검출**)
- **이름에 따른 조회**
    - **순수 JPA**
        
        ```java
        public List<Member> findByUsername(String username) {
            return em.createQuery("select m from Member m where m.username = :username", Member.class)
                    .setParameter("username", username)
                    .getResultList();
        }
        ```
        
        - **where 조건** 마저 **문자열로** 작성
        - 파라미터 **바인딩을 직접** 진행
    - **Querydsl**
        
        ```java
        public List<Member> findByUsername_Querydsl(String username) {
            return queryFactory
                    .selectFrom(member)
                    .where(member.username.eq(username))
                    .fetch();
        }
        ```
        
        - **where 조건**을 **자바 코드**로 작성 → **동적 쿼리 가능**
        - `.eq(username)`을 통해 **파라미터 바인딩**을 **Querydsl에서 자동**으로 진행

### 동적 쿼리, 성능 최적화 조회(DTO 조회)

- **DTO 생성**
    - **순수 JPA**를 통해 DTO로 직접 조회한다면?
        - **패키지명을 모두** 적어줘야 함
        - **문자열**로 진행되기 때문에 **IDE의 도움을 받을 수 없**으며 **컴파일 시점에 오류 확인 불가**
        - **Querydsl**의 `Projection`, `@QueryProjection` 를 통해 해결 가능
    - Member 정보와 Team 정보를 함께 받은 **DTO 생성 by Querydsl**
        
        ```java
        @Data
        public class MemberTeamDto {
            // Member와Team 정보를 같이 반환해줄 DTO
            private Long memberId;
            ...
            private Long teamId;
        		..
        
            @QueryProjection // 해당 DTO 생성자에 대한 QType 생성
            public MemberTeamDto(Long memberId, ...) {
                this.memberId = memberId;
                ...
            }
        }
        ```
        
        - `@QueryProjection` 사용
            - 해당 **DTO 생성자에 대한 QType 생성**
            - 항상 어노테이션을 달아주고 `compileQuerydsl` 을 통해 **실제 QType을 생성**해 주어야 함 (실행 후 저장 경로(`build/generated/…`)에서 확인)
            - 더 편리하게 **new 오퍼레이션으로 DTO 직접 조회**가 가능
            - DTO가 **Querydsl을 의존하게 된다는 단점** 존재 → **해당 DTO를 사용하는 모든 단에서 또한 Querydsl을 의존하게 됨**
            - 대신 **Projection**의 `.bean()`, `.fields()`, `.constructor()` 을 사용해도 됨 → **의존성 X**
- **동적 쿼리** 작성
    - **순수 JPA**로 동적쿼리 작성한다면?
        - **문자열 조합**으로 진행해야 됨
        - **재사용이 불가**능하며 **오류 검출을 컴파일 단계에서 불가**능하여 개발 생산성이 낮아짐
        - **Querydsl**의 `BooleanBuilder`, `BooleanExpression` 사용하여 해결 가능
    - **BooleanBuilder**를 통한 **동적 쿼리 by Querydsl**
        
        ```java
        public List<MemberTeamDto> searchByBuilder(MemberSearchCond cond) {
            BooleanBuilder builder = new BooleanBuilder();
            if (hasText(cond.getUsername())) { // 검색된 Member의 이름이 null 혹은 ""이 아니면
                builder.and(member.username.eq(cond.getUsername())); // 해당 조건 builder에 추가
            }
            ...
            if (cond.getAgeGoe() != null) {
                builder.and(member.age.goe(cond.getAgeGoe()));
            }
            ...
        
            return queryFactory
                    .select(new QMemberTeamDto(
                            member.id, member.username, member.age, team.id, team.name))
                    .from(member) // inner join이 아니라 outer join -> where 문에서 join된 team에 대해서 판단하므로 team은 null이면 안됨 (team이 null인 member는 제외)
                    .leftJoin(member.team, team) // fetchJoin() X -> 연관된 Entity를 넣는 것이 아니기때문
                    .where(builder) // join한 후 where을 넣는 이유 : join한 결과에서 판단하므로
                    .fetch();
        }
        ```
        
        - `BooleanBuilder builder = new BooleanBuilder();`
            - builder 생성
            - `.and()`, `.or()` 로 **체인식으로 연결**지어 **조건을 추가**할 수 있음
            - 혹은 **생성자에 기본 조건**을 넣을 수 있음 → `new BooleanBuilder(member.username.eq(cond.getUsername));`
        - **조건 추가**
            - `if (hasText(cond.getUsername()))` : MemberSearchCond의 username 이 null 혹은 “”(빈 문자열)이 아니라면
                - `builder.and(member.username.eq(cond.getUsername()));` : 빌더에 **조건 추가** → `where m.uesrname = :username`
        - **buider**를 **where문에 추가**
            - `.where(builder)`
            - 조건에 따라 **완성된 builder를 where 문에 넣으면 조건에 따른 조회가 가능**해짐
            - builder를 통해 **동적 쿼리**를 작성하는 부분
            - MemberSearchCond 에 아무 조건도 걸리지 않는다면 **builder에 조건이 추가되지 않은 상태**로 where문에 들어감 → **전체 조회와 동일**해짐. (해당 부분도 따로 처리가 필요)
    - **Where절 파라미터 사용** (`BooleanExpression` or `Predicate` 사용)
        
        ```java
        public List<MemberTeamDto> search(MemberSearchCond cond) {
            return queryFactory
                    .select(new QMemberTeamDto(
                            member.id, member.username, member.age, team.id, team.name))
                    .from(member)
                    .leftJoin(member.team, team)
                    .where(usernameEq(cond.getUsername()), // 해당 메서드에서 null을 반환하게 되면 무시됨
                            teamNameEq(cond.getTeamName()),
                            ageGoe(cond.getAgeGoe()),
                            ageLoe(cond.getAgeLoe()))
                    .fetch();
        }
        ```
        
        - where 조건들을 **method로** 빼내서 **해당 조건이 포함되어 있다면 그 조건을 반환**해주고 **아니면 null을 반환**하여 해당 조건을 무시하도록 설정
        - **재사용이 가능**하며 **조합** 또한 가능 → **실무에서 자주 사용**
        - 예시) `usernameEq(cond.getUsername())`
            
            ```java
            private BooleanExpression usernameEq(String username) {
                return hasText(username) ? member.username.eq(username) : null;
            }
            ```
            
            - **MemberSearchCond**의 **uesrname이 null이거나 빈문자열이면 null을 반환** 하고 **아니라면 조건 추가**
        - **조건 조합** 예시
            
            ```java
            private BooleanExpression ageBetween(Integer ageGoe, Integer ageLoe) {
                if (ageGoe == null) return ageLoe(ageLoe);
                if (ageLoe == null) return ageGoe(ageGoe);
                return ageGoe(ageGoe).and(ageLoe(ageLoe));
            }
            ```
            
            - 조건들을 조합(`.and()`, `.or()`) 해서 반환해주는 방식
            - 이때 `~~Predicate`로 반환~~하면 동작하지 않음 → 조합을 위해서 `BooleanExpression` 사용
            - **조합** 시 항상 **null 조건에 대해 유의**해야 됨 → 조합중 하나만 null일 경우, 둘다 null일 경우 등
- **Respository 활용** (조회 API 컨트롤러 개발)
    - `MemberController` 개발
    - Class 단 Annotation
        
        ```java
        @RestController // Rest API
        @RequiredArgsConstructor
        public class MemberController {...}
        ```
        
        - `@RestController` : **API 반환**을 위한 **Controller** 설정
        - `@RequiredArgsConstructor` : private final 로 설정된 field에 대해 **자동으로 DI**실행 → 생성자를 통해
    - 의존성 주입 (DI)
        
        ```java
        private final MemberQueryRepository memberQueryRepository;
        ```
        
        - MemberQueryRepository 주입 : Querydsl을 통해 개발한 Repository
    - 검색조건에 따른 데이터 반환
        
        ```java
        @GetMapping("/v1/members")
        public List<MemberTeamDto> searchMemberV1(@ModelAttribute MemberSearchCond cond) {
            return memberQueryRepository.search(cond);
        }
        ```
        
        - `@GetMapping`
            - /v1/members 에 응답
        - `@ModelAttribute MemberSearchCond cond`
            - RequestParam을 통해 들어오는 값들을 **MemberSearchCond에 Mapping** 시켜줌
            - ex) **?username=kim&goe=20** → `MemberSearchCond{username = kim, goe = 20, …}`
        - `return memberJPARepository.search(cond)`
            - 리포지토리를 통해 **자바코드로 작성한 동적 쿼리의 결과를 반환**
            - JSON형식으로 반환 → API호출 결과값

## 스프링 데이터 JPA와 Querydsl

**스프링 데이터 JPA 와 Querydsl을 동시에 사용**하기

### 스프링 데이터 JPA 기반 Repository 생성

- **MemberRepository interface** 생성
    - `extends JpaRepository<Member, Long>` : **스프링 데이터 JPA를 사용**하기 위해서 JpaRepository 상속
    - `public interface MemberRepository extends JpaRepository<Member, Long> { }`
    - **공통 인터페이스 기능 제공**됨
        - 저장(`save()`), 아이디를 통한 조회(`findById()`), 전체 조회(`findAll()`), …
    - **메서드 이름을 통해 자동 쿼리 생성** 가능
        - ex) 이름을 통한 조회(`findByUsername()` → **find…ByField()**)
    - 하지만 **해당 Repository에는 Querydsl을 사용할 수 없음** → Interface 이므로 Querydsl을 사용하고 싶다면 **해당 Repository의 모든 메서드를 구현**하면서 **Querydsl항목을 추가 구현**해주어야 됨 → 불가능 **⇒ [중요] 사용자 정의 리포지토리를 통해 Querydsl만 따로 구현**

### 사용자 정의 리포지토리 생성

- 사용자 정의 리포지토리 사용법
    ![Untitled]({{site.baseurl}}/images/posts/post-220926/Untitled.png)
 
    1. 사용자 정의 **인터페이스 생성** (MemberRepoCustom interface)
    2. 사용자 정의 **인터페이스 구현** (MemberRepositoryImpl class (**구현체**))
    3. 스프링 데이터 JPA를 사용하는 리포지토리 인터페이스에 사용자 정의 **인터페이스 상속** 
    (`... extends JPARepository<,> , MemberRepoCustom`)
1. **사용자 정의 인터페이스 생성 → `MemberRepoCustom interface`**
    
    ```java
    public interface MemberRepositoryCustom {
        List<MemberTeamDto> search(MemberSearchCond condition);
    }
    
    ```
    
    - QueryDSL과 Spring Data JPA를 **같이 사용하기 위한 custom repo 인터페이스**
    - `search()` : 이전 파트에서 진행했던 **조건에 따른 검색 쿼리 생성**을 **Querydsl의 BooleanExpression 으로 구현**한 메서드
    - 이렇게 Interface로 선언해주고 **구현체에서 해당 메서드를 Override 해서 구현**해주면 됨!
2. **사용자 정의 인터페이스 구현 → `MemberRepositoryImpl class` (구현체)**
    
    ```java
    @RequiredArgsConstructor
    public class MemberRepositoryImpl implements MemberRepositoryCustom{ 
    		
        private final JPAQueryFactory queryFactory;
    
        @Override
        public List<MemberTeamDto> search(MemberSearchCond cond) {...}
    	  ...
    }
    ```
    
    - QueryDSL과 Spring Data JPA를 **같이 사용하기 위한 custom repo 구현체**
    - 주의할 점은 **구현체의 이름**을 항상 **스프링 데이터 JPA를 사용하는 Interface의 이름(”MemberRepository”) + “Impl” 로 설정**해주어야 함 → 이래야지 스프링 데이터 JPA가 **구현체를 인식하고 Custom Interface와 연결**해줌!
    - QueryDSL을 사용하기 위해 **JPAQueryFactory를 의존성 주입**해줌
    - custom Interface에서 정의한 `search()` **메서드 구현** → 이전과 동일한 method (Where절에 파라미터를 이용하여 동적 쿼리 작성)
3. **스프링 데이터 JPA를 사용하는 리포지토리 인터페이스에 사용자 정의 인터페이스 상속 → `extends JPARepository<Member, Long> , MemberRepoCustom)`**
    
    ```java
    public interface MemberRepository extends JpaRepository<Member, Long>, MemberRepositoryCustom {
        List<Member> findByUsername(String username); // 메서드 이름을 통한 자동 쿼리 생성
    }
    ```
    
    - 이렇게 되면 **스프링 데이터 JPA를 사용하는 MemberRepository 에서도 Querydsl을 이용한 method 사용 가능** → `memberRepository.search(…)`

### Querydsl 페이징 연동

- **Spring Data 의 Page, Pageable을 사용**해서 **Querydsl로도 페이징 기능을 구현**해보기
- Queyrdsl을 위한 **Custom Repository에 페이징 기능을 포함한 method 추가**
    
    ```java
    public interface MemberRepositoryCustom {
    
        List<MemberTeamDto> search(MemberSearchCond condition);
    
        Page<MemberTeamDto> searchPage(MemberSearchCond condition, Pageable pageable);
    }
    ```
    
    - `Page<MemberTeamDto> searchPage(MemberSearchCond condition, Pageable pageable);`
        - pageable에 **page 조건을 넣어주면 Page로 사용할 수 있게 끔 반환**
        - 해당 method를 구현체에서 구현해주기만 하면 됨
        - 즉, **Pageable을 사용한 Querydsl의 동적 쿼리로 Page 반환 가능** → 그대로 웹계층에서 Page를 통해 페이징 기능 사용 가능!
- `searchPage()` method 구현
    - 쿼리 결과(content) 부분
        
        ```java
        List<MemberTeamDto> content = queryFactory
        	      .select(new QMemberTeamDto(
        	              member.id, member.username, member.age, team.id, team.name))
        	      .from(member)
        	      .leftJoin(member.team, team)
        	      .where(usernameEq(cond.getUsername()), // 해당 메서드에서 null을 반환하게 되면 무시됨
        	              teamNameEq(cond.getTeamName()),
        	              ageGoe(cond.getAgeGoe()),
        	              ageLoe(cond.getAgeLoe()))
        	      .orderBy(member.id.asc())
        	      .offset(pageable.getOffset())
        	      .limit(pageable.getPageSize())
        	      .fetch();
        ```
        
        - **Pageable(페이징 정보를 담은 객체)** 에서 **offset**과 **page size**를 얻어와 `.offset()` 과 `.limit()` 설정을 통해 **페이징 구현**
            - `.offset(pageable.getOffset())`
            - `.limit(pageable.getPageSize())`
        - `~~.fetchResults()~~` 를 사용할 수 없게 됨으로써 **직접 count query 작성 필요**
    - 전체 개수 조회 부분
        
        ```java
        Long total = queryFactory
              .select(member.count()) // count query -> fetchCount 대용
              .from(member)
              .leftJoin(member.team, team)
              .where(usernameEq(cond.getUsername()), // 해당 메서드에서 null을 반환하게 되면 무시됨
                      teamNameEq(cond.getTeamName()),
                      ageGoe(cond.getAgeGoe()),
                      ageLoe(cond.getAgeLoe()))
              .fetchOne();
        ```
        
        - 성능 최적화 및 `~~.fetchResults()~~` 를 사용할 수 없게됨으로써 **직접 count query** 생성
        - `~~.fetchCount()~~` 가 없어짐으로써 `member.count()` 를 통해 **전체 개수를 가져**옴
    - content(쿼리 결과 내용)와 total(전체 개수)를 통해 **Page 생성**
        
        ```java
        return new PageImpl<>(content, pageable, total);
        ```
        
        - **Page**를 통해 반환하여 **웹 계층에서 직접 페이징 기능을 사용**할 수 있도록 설정
        - `PageImpl<>` 에 **쿼리 결과 내용, 페이징 정보, 전체 개수**를 넣어 **Page 객체를 생성**하여 반환
- Querydsl + Page 를 사용하기 위한 **컨트롤러** 개발
    
    ```java
    @GetMapping("/v2/members")
    public Page<MemberTeamDto> searchMemberByPage(MemberSearchCond cond, Pageable pageable) {
        return memberRepository.searchPage(cond, pageable);
    }
    ```
    
    - 기존에 생성했던 `MemberController`에 `searchMemberByPage`추가
    - `“/v2/members”` 에서의 RequestParam 으로 받아온 **MemberSearchCond(검색 조건)** 과 **Pageable(페이징 정보)**를 통해 searchPage 동작
    - ex) **?uesrname=kim&page=0&size=16** ⇒ username이 “kim”인 member를 조회하는데 16개로 끊어진 페이지에서 가장 첫번째 페이지의 결과를 가져오라 → **Page정보를 가진 JSON으로 반환**
    - Page로 반환되었기 때문에 **content 뿐만 아닌 page 정보들이 JSON으로 반환**됨 → API 호출의 결과