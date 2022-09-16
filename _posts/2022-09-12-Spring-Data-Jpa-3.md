---
layout: post
title:  "Spring Data Jpa - 확장"
date:   2022-09-12
image:  /posts/springData.jpg
tags:   SpringBoot JPA SpringData
---

## 확장 기능

### 사용자 정의 REPO 구현

- 현재 스프링 데이터 JPA를 사용하는 REPO에서 JPA를 직접 사용하거나 JDBC Template, MyBatis 를 사용하는 등 **스프링 데이터 JPA가 아닌 메서드를 직접 구현하고 싶을 때**의 방법 (**특히 QueryDSL**을 사용해야 할 때)
- 만약 해당 방법을 사용하지 않고 현재 REPO의 **구현체를 만들어 메서드를 추가한다면?** →  Spring Data Jpa가 구현해주는 모든 메서드들을 직접 구현해준 후에 메서드를 추가해주어야 함! **너무나 복잡!**
- **사용자 정의 Repository** 구현
    1. 사용자 정의 **인터페이스** 생성
        
        ```java
        public interface MemberRepositoryCustom {
        	 List<Member> findMemberCustom();
        }
        ```
        
        - `List<Member> findMemberCustom();` : **Spring Data Jpa 를 사용하지 않은** 사용자 정의 메서드 → **JPA를 직접 사용하거나 QueryDSL 등을 사용**하고자 하는 메서드
    2. 사용자 정의 **인터페이스의 구현체** 생성 (구현체의 **이름은 정해진 규칙에 따라 생성**)
        
        ```java
        @RequiredArgsConstructor
        public class MemberRepositoryImpl implements MemberRepositoryCustom {
        	 private final EntityManager em;
        	 @Override
        	 public List<Member> findMemberCustom() {
        			 return em.createQuery("select m from Member m")
        					 .getResultList();
        	 }
        }
        ```
        
        - 해당 Interface의 **메서드를 구현** (`@Override`)
        - 해당 구현체는 **JPA를 직접 사용(JPQL)하기 위한 메서드**
        - 구현체의 이름은 현재 Spring Data JPA를 사용(JpaRepository<>)하는 Interface(`MeberRepository`)에 “`Impl`” 이라는 키워드를 붙여줘야함 (`MeberRepositoryImpl`) → 정해진 규칙 (**규칙: 리포지토리 인터페이스 이름 + Impl or 사용자 정의 인터페이스 명 + “Impl” →** `MeberRepositoryCustomImpl`)
        - 이렇게 규칙에 따라 구현 클래스를 만들어주면 따로 **스프링 빈으로 등록할 필요 없이 스프링 데이터 JPA가 인식하여 스프링 빈으로 등록!** → 따라서 규칙에 따라야하는 것은 필수!!
    3. 해당 사용자 정의 인터페이스를 현재 **Spring Data JPA를 사용(JpaRepository<>)하는 Interface에 상속**을 줌
        
        ```java
        public interface MemberRepository
        			 extends JpaRepository<Member, Long>, MemberRepositoryCustom {}
        ```
        
    4. 사용자 정의 메서드 호출 및 사용
        
        ```java
        List<Member> result = memberRepository.findMemberCustom();
        ```
        
        - 이질감 없이 그냥 **바로 memberRepository 에서 호출해서 사용** 가능

<aside>
⚠️ <strong>언제 사용해야 되나?</strong>
<br>
실무에서는 주로 <b>QueryDSL</b>이나 <b>SpringJdbcTemplate</b>을 <b>함께 사용</b>할 때 사용자 정의 리포지토리 기능 자주 사용

</aside>

<aside>
⚠️ <strong>사용자 정의 리포 주의점!</strong>
항상 사용자 정의 리포지토리가 필요한 것은 아님. <b>그냥 임의의 레포를 생성해서 사용해도 됨!!</b> 예를들어 MemberRepositoryForQuerydsl를 별도의 클래스로 만들고 스프링 빈으로 등록해서 그냥 직접 사용해도 됨 → <b>핵심 비지니스에 속하는 REPO랑 화면에 맞춘 REPO는 분리해주는 것이 좋음!</b>

</aside>

### Auditing

- Entity를 생성하고 변경할 때 **자동으로 변경한 사람과 시간을 추적**하게 해줄 수 있는 기능
- 등록일, 수정일, (등록자, 수정자) 는 실무에 있어서 필수 Col→ **추적**을 위해
- 즉, 이들은 **공통적으로 들어가는 Field** 이고, 이들에 대해서 **자동으로 기록해주는 수단이 필요!** → 해당 Entity가 등록되기 직전, 자동으로 등록일과 등록자를 기록해주고, 수정되기 직전, 자동으로 수정일과 수정자를 기록해주는 수단
- 순수 JPA에서의 Auditing (`@PrePersist`, `@PreUpdate`)
    
    ```java
    @MappedSuperclass
    @Getter
    public class JpaBaseEntity {
    		@Column(updatable = false)
    		private LocalDateTime createdDate;
    		private LocalDateTime updatedDate;
    
    		@PrePersist
    		public void prePersist() {...}
    		@PreUpdate
    		public void preUpdate() {...}
    }
    ```
    
    - `JpaBaseEntity`
        - `@MappedSuperclass` : 해당 Class의 Field를 **공통 Field**로 만들겠다는  뜻. → 다른 Entity들은 해당 Class 를 상속받고 공통 Field를 사용할 수 있음
        - 등록일과 수정일 Field 보유 (공통 Field)
        - ex) `public class Member extends JpaBaseEntity {}` → Member는 createdDate, updatedDate Field를 보유하게 됨
    - `@PrePersist`
        - 해당 class 를 상속받은 Entity가 **저장되기 직전**에 해당 **어노테이션이 달린 메서드(createdDate, updatedDate 설정) 실행**
    - `@PreUpdate`
        - 해당 class 를 상속받은 Entity가 **수정되기 직전**에 해당 **어노테이션이 달린 메서드(updatedDate 설정) 실행**
    - JPA 주요 이벤트 어노테이션
        - **저장** 관련 : `@PrePersist`, `@PostPersist`
        - **수정** 관련 : `@PreUpdate`, `@PostUpdate`
- **스프링 데이터 JPA 에서의 Auditing** (`@CreatedDate`, `@LastModifiedDate`) 사용
    1. **스프링 부트 설정 클래스**에 `@EnableJpaAuditing` 적용 (추가로 등록자, 수정자 처리를 원한다면, `AuditorAware`를 스프링 빈 등록)
        
        ```java
        @EnableJpaAuditing // Auditing기능을 사용하기 위한 설정
        @SpringBootApplication
        public class DataJpaApplication {
        
        	... 
        	@Bean // 등록자, 수정자를 처리해주는 AuditorAware 스프링 빈 등록
        	public AuditorAware<String> auditorProvider() {
        		return () -> Optional.of(UUID.randomUUID().toString());
        	} // 내부 getCurrentAuditor() 를 람다로 구현한 것
        
        }
        ```
        
        - 여기서는 UUID를 통해서 진행했지만, **실무에선 세션 정보나, 스프링 시큐리티 로그인 정보에서 ID를 받아와 저장**해줌 (`@CreatedBy`, `@LastModifiedBy` 가 달린 Field에)
    2. Auditing을 사용하고자 하는 (등록일, 수정일 등을 가진) **엔티티에**  `@EntityListeners(AuditingEntityListener.class)` (**이벤트 기반으로 동작한다**는 것을 명시) 적용한 후 `@CreatedDate`(등록일), `@LastModifiedDate`(변경일), `@CreatedBy`(등록자), `@LastModifiedBy`(변경자)
        
        ```java
        @EntityListeners(AuditingEntityListener.class)
        @MappedSuperclass
        @Getter
        public class BaseEntity {
        		 @CreatedDate
        		 @Column(updatable = false)
        		 private LocalDateTime createdDate;
        
        		 @LastModifiedDate
        		 private LocalDateTime lastModifiedDate;
        
        		 @CreatedBy
        		 @Column(updatable = false)
        		 private String createdBy;
        
        		 @LastModifiedBy
        		 private String lastModifiedBy;
        }
        ```
        
    - 이렇게 설정하게 되면 순수 JPA처럼 직접 Method를 짤 필요 없이 **자동으로 등록, 수정과 관련된 필드를** 마치 순수 JPA에서의 Auditing 동작과 동일하게 **저장**해줌
    - 등록자, 변경자 등은 필수적이지 않는 경우가 더 많음 → Base 타입을 분리하고, 원하는 타입을 선택해서 상속하는 방법으로 진행하면 효율적

### 페이징과 정렬 - web 확장

- 스프링 데이터에서 제공하는 **Paging과 Sorting 기능**을 **스프링 MVC에서 편리하게 사용** 가능
- 스프링 MVC에서 **파라미터를** **Pageable**로 받고, 해당 **Pageable을 그대로 사용하여 조건에 맞는 페이지 결과를 얻**어낼 수 있음
- 예시)
    
    ```java
    @RestController
    @GetMapping("/members")
    public Page<MemberDto> list(Pageable pageable) {
    	 Page<Member> page = memberRepository.findAll(pageable);
    	 return page.map(MemberDto::new);
    }
    ```
    
    - 파라미터 : `Pageable`
        - **Request로써** 받아올 수 있음
        - Pageable은 Interface이고, 요청파라미터를 넣어주게 되면 **스프링부트에서 자동으로 이에 대한 구현체인** `PageRequest` **객체를 생성**해줌 (PageRequest  는 이전 페이징과 정렬에서 `PageRequest.of(page,size,Sort)` 의 그 PageRequest)
        - 참고 : Spring Data Jpa 에서의 “공통 인터페이스 기능(method)들, 메서드이름으로 쿼리 생성”은 모두 PageRequest를 인자로 받을 수 있음
    - 반환 : `Page<MemberDto>`
        - 조건에 따른 결과 **List Content 포함**
        - 부가적인 **페이지 정보** 포함
        - 주의 : 항상 **Controller의 반환**은 Entity가 아닌 **DTO**로 해야되는 점!
    - 요청 파라미터 예시
        - `/members?page=0&size=3&sort=id,desc&sort=username,desc`
        - **page**
            - focus할 페이지 (주의 : 0부터 시작)
            - 값을 넣지 않으면 default=0
        - **size**
            - 해당 페이지에 가져올 데이터의 개수
            - 값을 넣지 않으면 default = 20
        - **sort**
            - 정렬 조건
            - 값을 넣지 않으면 정렬 X
- 요청 파라미터 Default 값 설정
    - **Global**
        - `spring.data.web.pageable.max-page-size=2000` : 최대 페이지 사이즈 설정
        - `spring.data.web.pageable.default-page-size=20` : size default 값 설정
    - **개별 설정**
        - `@PageableDefault` 어노테이션 사용
            
            ```java
            @GetMapping
            public String list(
            		@PageableDefault(size = 12, sort = “username”,
            		direction = Sort.Direction.DESC) Pageable pageable)
            ```
            
- 접두사
    - `@Qualifier`
    - 페이징 **정보가 둘 이상**일 때 사용
    - `@Qualifier` 에는 접두사명 부여(`@Qualifier("member")`), 요청파라미터에는 접두사로 구분(`"{접두사명}_xxx”`)
    - ex)
        - **요청 파라미터** : `/members?member_page=0&order_page=1`
        - **Controller 파라미터**
            
            ```java
            public String list(
             @Qualifier("member") Pageable memberPageable,
             @Qualifier("order") Pageable orderPageable, ...) {}
            ```
            

## 부가 기능

### Projections

- 엔티티 대신, **DTO를 편리하게 조회**할 때 사용
- **단순한 DTO 조회 시 유용**
- 단점 : 연관관계를 포함한 DTO 조회 시 최적화 불가능 + LEFT OUTER JOIN만 사용 가능
- 예시)
    - 회원 이름만 조회하고 싶은 상황
    - 일반 스프링 데이터 JPA
        
        ```java
        @Query("select m.name from Member m where m.name = :name")
        List<String> findNamesByName(@Param("name") String name);
        ```
        
        - `@Query` 를 통해 **직접 jpql문을 작성**해줘야 하고, **바인딩도 직접**해주는 **번거러움** 존재
        - 다른 조건으로 검색 시 JPQL을 또 name으로 반환할 수 있도록 작성해주어야 함
    - **스프링 데이터 JPA에서 Projections 사용** **(Closed Projections)**
        - Interface 생성
            
            ```java
            public interface NameOnly {
            	 String getName();
            }
            ```
            
            - 해당 Interface로 **Projection의 결과**를 받음
            - `getName()` 을 통해서 **이름만** 받아옴
        - MemberRepository에 메서드 추가
            - `List<NameOnly> findProjectionsByName(String name);`
            - **반환 타입을 방금 생성한 Interface로 설정**
            - Member Entity의 모든 Field를 select해서 가져오는 것이 아닌 **name field만 select해서 가져옴 → 최적화**
            - 반환 타입으로 인지하기에 `find…By`~ 에서 … 은 자유
    - **스프링 데이터 JPA에서 Projections 사용 (Open Proejctions)**
        
        ```java
        public interface UsernameOnly {
        	 @Value("#{target.username + ' ' + target.age + ' ' + target.team.name}")
        	 String getUsername();
        }
        ```
        
        - `@Value` 를 통해서 **직접 select 해올 field를 지정하고 해당 결과를 어떻게 표현**해 낼지 지정 가능
        - 하지만 이렇게 할 경우 DB에서 **엔티티 필드를 다 조회해온 다음에 계산** → JPQL SELECT 절 **최적화 불가능**
- DTO class Projection
    
    ```java
    @Getter
    public class NameOnlyDto {
    	 private final String name;
    	 private final int age;
    
    	 public NameOnlyDto(String name, int age) {
    		 this.name= username;
    		 this.age = age;
    	 }
    }
    ```
    
    - **구체적인 DTO 형식으로 Projection**
    - 생성자의 **파라미터 이름으로 매칭** → `public NameOnlyDto(String name, int age)`
- 동적 class Projections
    - MemberRepository의 Spring Data JPA의 method에서 **Generic Type을 부여해주면 동적으로도 프로젝션 변경 가능**
        
        ```java
        <T> List<T> findProjectionsByUsername(String username, Class<T> type);
        ```
        
    - 사용
        
        ```java
        List<UsernameOnly> result = memberRepository.findProjectionsByUsername("m1", UsernameOnly.class);
        ```
        

### 네이티브 쿼리

- **SQL 언어로 직접 query를 작성**하는 것 (Spring Data Jpa의 페이징 기능 지원)
- 네이티브 쿼리는 가급적 사용하지 않는게 좋음 → **반환타입도 제한되고 제약조건이 많음**
    - 반환 타입
        - Object[], Tuple, DTO(스프링 데이터 인터페이스 Projections 지원)
    - 제약
        - Sort 파라미터를 통한 **정렬이 정상 동작하지 않을 수 있음**
        - JPQL처럼 애플리케이션 **로딩 시점에 문법 확인 불가**
        - **동적 쿼리 불가**
- **네이티브 쿼리 + Projections**
    - 네이티브 쿼리가 Projections를 통해 DTO로 어렵지 않게 반환 가능 → 그나마 유용한 기능
    - MemberProjection Interface
        
        ```java
        public interface MemberProjection {
        	Long getId();
        	String getName();
        	String getTeamName();
        }
        ```
        
    - NativeQuery with Projections
        
        ```java
        @Query(value = "SELECT m.member_id as id, m.username, t.name as teamName " +
        	 "FROM member m left join team t",
        	 countQuery = "SELECT count(*) from member",
        	 nativeQuery = true)
        Page<MemberProjection> findByNativeProjection(Pageable pageable);
        ```
        

<aside>
⚠️ <strong>Native Query가 필요하다면?</strong> <br>
스프링 데이터 JPA를 사용하기보다는 스프링 JdbcTemplate, myBatis 등을 통해 해결하는 것이 훨씬 나음!

</aside>