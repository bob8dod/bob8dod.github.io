---
layout: post
title:  "Spring Boot 입문"
date:   2022-04-20
image:  /posts/post0/springbootpng.png
tags:   SpringBoot
---

## **Chapter 1**


- **프로젝트 생성**
    1. Maven Project vs Gradle Project
        - 필요한 라이브러리를 땡겨 오고 build하는 것 까지 관리해주는 Tool!
        - 즉,  의존관계를 관리. 해당 라이브러리에 필요한 의존관계를 자동으로 설치해주고 설정해주는 것!
        - 요즘 거의 대부분이 Gradle Project로 Spring boot 프로젝트를 생성함
    2. Spring Boot Version
        - SNAPSHOT, M1 같은 미정식 버전을 제외하고 최신 버전을 사용
    3. Dependecies
        - 프로젝트를 생성할 것인데, 어떤 라이브러리들을 땡겨 쓸거냐. 즉, 초기 라이브러리를 어떤 것들로 쓸것이냐의 설정
        - Web 프로젝트를 만들기에 spring web 선택 /  MVC에 이용 되는 템플릿 엔진 **Thymeleaf** 선택 (html에서 thymeleaf를 통해 백엔드의 데이터를 주고 받을 수 있음, 다양한 템플릿 엔진 존재, 때에 따라 선택하면 됨)


- **프로젝트 실행**
    - 기본 메인 클래스 실행 (main/java/ ... /`SemogongApplication`)
    - [http://localhost:8080](http://localhost:8080/) 로 접속
    - 실행 속도 높이기
      Preferences → Build, Execution, Deployment → Build Tools → Gradle
    - Build and run using: Gradle IntelliJ IDEA
    - Run tests using: Gradle IntelliJ IDEA

    - 빌드하고 실행하기 (명령 프롬프트(cmd)로 진행, Cloud Server에 올릴때)
        1. 해당 프로젝트 경로로 이동
        2. gradlew build → 현재 spring boot application을 build
        3. cd build/libs → 빌드된 java 파일 저장 위치로 이동
        4. java -jar hello-spring-0.0.1-SNAPSHOT.jar → 빌드된 spring boot application java 파일 실행. 즉, 어플리케이션 가동
        5. 실행 확인 → [http://localhost:8080/](http://localhost:8080/) or domain


- **Static vs MVC+Template Engine(Thymeleaf) vs API**
    - Static
        - 스프링 부트 정적 컨텐츠 기능
        - static/index.html 을 올려두면 Welcome page 기능을 제공 → 가장 메인 화면
    - MVC+Template Engine(Thymeleaf)
        - MVC(Model, Veiw, Controller)
            - 사용자 인터페이스, 데이터 및 논리 제어를 구현하는데 널리 사용되는 소프트웨어 디자인 패턴
            - 요청이 들어오면 그 요청을 내장 톰켓 서버에서 받아서 스프링 컨테이너 속 Controller에 해당 요청 method가 있는지 확인 후 그 method가 반환하는 template html에 설정한 Model을 실어 viewResolver에게 보내줌. 그럼 viewResolver는 반환된 html에 Model을 보내주며 Thymleaf 템플릿 엔진으로 Model의 속성을 처리해주며 html을 HTTP로 보냄
        - @GetMapping(” ”) 사용. 컨테이너에서 해당 요청을 찾을 수 있도록
        
    - API : Application Programming Interface
        - Application Programming Interface :  컴퓨터나 소프트웨어를 서로 연결. 문자나 객체(JSON)를 반환
        - @ResponseBody 를 사용하면 뷰 리졸버( viewResolver )를 사용하지 않고 HTTP의 BODY에 문자 내용 or 객체(JSON)를 직접 반환
        - 객체를 반환하면 객체가 JSON으로 변환됨
        - @ResponseBody 사용 원리
            - HTTP의 BODY에 문자 내용을 직접 반환
            - viewResolver 대신에 HttpMessageConverter 가 동작
            - 기본 문자처리: StringHttpMessageConverter
            기본 객체처리: MappingJackson2HttpMessageConverter
            - byte 처리 등등 기타 여러 HttpMessageConverter가 기본으로 등록되어 있음
            
            ```
            참고: 클라이언트의 HTTP Accept 해더와 서버의 컨트롤러 반환 타입 정보 둘을 조합해서
            HttpMessageConverter 가 선택된다.
            ```

            ![]({{site.baseurl}}/images/posts/post0/0.png)

            
- **일반적인 백엔드 개발**
    - 비지니스 요구사항 정리
  ![]({{site.baseurl}}/images/posts/post0/1.png)

    1. 컨트롤러 : 웹 MVC에서 Controller 역할
        - HTTP 요청에 따른 정해진 작업을 실행하기 위한 컨트롤러.
        - model을 얹은 html을 반환 하거나 API(문자, 객체_Json을 반환))
    2. 서비스 : 핵심 비지니스 로직 구현. (ex. 회원 가입 or 회원 조회, ... )
        - 요청에 따른 컨트롤러의 선택을 받으면 해당 로직을 실행.
        - 보통 해당 서비스의 로직에 따라 repository를 통해 도메인을 관리
    3. 리포지토리 : 데이터베이스에 접근, 도메인 객체를 DB에 저장하고 관리함.
        - Interface를 통해 상황에 따른 다양한 DB server에 대한 다형성을 제공
        - 아직 데이터 저장소가 선정되지 않아서, 우선 인터페이스로 구현 클래스를 변경할 수 있도록 설계
        - 데이터 저장소는 RDB, NoSQL 등등 다양한 저장소를 고민중인 상황으로 가정
        - 개발을 진행하기 위해서 초기 개발 단계에서는 구현체로 가벼운 메모리 기반의 데이터 저장소 사용
    4. 도메인 : 비지니스 도메인 객체
        - (ex. 회원 객체, 게시물 객체, 댓글 객체, ...)
        - 주로 데이터베이스에 저장하고 관리됨

    - 테스트 케이스
        - 단순 테스트 (자바 자체로의 Test)
            - 스프링에 올려 Test를 진행하는 것이 아닌, 자바 코드 자체로의 테스트. 즉, Spring Boot는 실행되지 않음, 물론 DB도 연결되지 않음.
            - 개발한 기능을 실행해서 테스트 할 때 자바의 main 메서드를 통해서 실행하거나, 웹 애플리케이션의 컨트롤러를 통해서 해당 기능을 실행한다. 하지만 이러한 방법은 준비하고 실행하는데 오래 걸리고, 반복 실행하기 어렵고 여러 테스트를 한번에 실행하기 어렵다는 단점이 있다. 그래서 자바는 **JUnit이라는 프레임워크로 테스트를 실행**해서 이러한 문제를 해결
            - 해당 class에서 ctrl + alt + T 를 통해 자동으로 Test Case 생성 가능
            - given / when(what) / then 의 구조로 짜야됨!
                - “주어진 것들이 무엇이고 / 이들을 통해 언제 무엇을 할것이며 / 그 결과 어떤 것을 test할 것이냐!” 의 구조
            - @AfterEach : 테스트가 종료될 때 마다 이 기능을 실행. ex) 메모리 DB에 저장된 데이터를 삭제
            - @BeforeEach :  각 테스트 실행 전에 호출된다. 테스트가 서로 영향이 없도록 항상 새로운 객체를 생성하고, 의존관계도 새로 맺어준다.
            - **!!** 테스트는 각각 **독립적으로 실행**되어야 한다. 테스트 순서에 의존관계가 있는 것은 좋은 테스트가 아니다
            - Test에서 자주 사용되는 것

            ```java
            import static org.assertj.core.api.Assertions.*;
            assertThat(result).isEqualTo(member); // 동일한 멤버면 오류발생 X. 동일하지 않으면 오류 발생 → Test 미통과!
            assertThrows(IllegalStateException.class, () -> memberService.join(member2)); // 뒤의 로직을 실행했을 때 해당 오류가 발생하지 않는다면 미통과! 오류가 발생하면 통과! → 헷갈릴 수 있음 주의하길!
            ```
        
        - 스프링 통합 테스트 (테스트를 스프링에서 진행)
            - 스프링 위에서 테스트를 진행하는 것. 즉, 스프링 컨테이너와 DB까지 연결하여 통합적으로 테스트를 진행
            - @SpringBootTest : 스프링 컨테이너와 테스트를 함께 실행. spring을 가동하고 spring 안에서 test가 진행되는 것
            - @Transactional : 테스트 케이스에 이 애노테이션이 있으면, 테스트 시작 전에 트랜잭션을 시작하고, 테스트 완료 후에 항상 롤백. 즉, 각각의 test가 독립적, 반복적으로 진행되게 끔 해주는 것. 이렇게 하면 실제 DB에 영향을 주지 않음 (이게 Rollback 개념). 고로 Beforeach, Aftereach를 사용할 필요가 없음
        
    - DI(Dependency Injection, 의존성 주입) 직접 설정하기
        - memberservice는 기존에 각 class 자체에서 repository를 생성해서 사용 했음.
        - 이렇게 되면 service 전체에서 사용 되는 repository의 모호함이 생김. → DI 필요!
        - memberservice를 생성할 때 생성자를 통해 자동으로 설정한 repository를 주입 시켜줌 → DI!
        
        ```java
        public MemberService(MemberRepository memberRepository) {
         this.memberRepository = memberRepository; }
        ```


- **스프링 빈과 의존관계**
    - 컴포넌트 스캔과 자동 의존관계 설정 (@Component[Controller, Service, Repository], @Autowired)
        - 회원 컨트롤러가 회원서비스와 회원 리포지토리를 사용할 수 있게 의존관계를 준비

      ![]({{site.baseurl}}/images/posts/post0/2.png)

        ```java
        @Controller
        public class MemberController {
         private final MemberService memberService;
         @Autowired
         public MemberController(MemberService memberService) {
         this.memberService = memberService;
         }
        }
        ```

        - @Component : 스프링 빈으로 자동 등록. 즉, 직접 인스턴스를 할당해서 다루는 것이 아니라, ComponentScanner(SpringbootAppication)이 스캔 시 확인이 가능하고 이를 Spring Container로 올려, 따로 인스턴스 할당 없이 사용됨.
        - @Controller : controller 역할로써의 component로 Spring Bean에 등록.
        - @Service :  service 역할로써의 component로 Spring Bean에 등록. 주로 controller에 Dependency Injection을 통해 연결(@Autowired)됨.
        - @Repository : 리포지터리 역할로써의 component로 Spirng Bean에 등록. 주로 Service와 연결(@Autowired)됨.
        - @Autowired : 스프링이 연관된 객체를 스프링 컨테이너에서 찾아서 넣어줌.  이렇게 객체 의존관계를 외부에서 넣어주는 것을 DI (Dependency Injection), 의존성 주입. 그렇기에 연결하려는 객체가 빈으로 등록되지 않았다면 오류 발생.

    - @Configuration (SpringConfig)를 통해 직접 스프링 빈 등록하기

        ```java
        @Configuration
        public class SpringConfig {
        
         @Bean
         public MemberRepository memberRepository() {
        return new MemoryMemberRepository();
         }
        }
        ```

        - @Configuration : Spring 환경설정 부분. Bean 등록 등 설정에 사용되는 객체라는 것을 명시
        - @Bean : 해당 method의 반환되는 객체를 스프링 빈으로 등록함. 여기서의 Autowired 기능은 해당 객체에서 직접 설정해야 됨.
        - 실무에서는 주로 정형화된 컨트롤러, 서비스, 리포지토리 같은 코드는 컴포넌트 스캔을 사용. 하지만 정형화 되지 않거나, **상황에 따라 구현 클래스를 변경해야 하면** 설정을 통해 스프링 빈으로
          등록한다
            - Repository 같은 경우 Bean의 mehtod의 return 값만 내가 원하는 DB로 연결하는 코드로 수정만 하면 원하는 DB로 연결 가능. 그래서 보통 repository는 따로 bean으로 등록. → 다형성을 위함!!!


- **웹 MVC 개발**
    - 회원가입
        - 회원 가입 시 입력되는 데이터를 전달 받을 폼 객체 생성

            ```java
            // 입력된 데이터를 전달 받을 폼 (MVC의 Model)
            public class MemberForm {
                private String name;
                private String job;
                private String describe;
            // get set 설정 필수 (생략)
            }
            ```

            - 여기서 set을 통해 입력된 값들을 저장하는 것.
        - 입력되는 정보들은 submit을 할 시, 정해진 url을 통해 PostMapping을 진행. 그 때 이 전달된 정보들을 받을 객체가 필요함.

            ```java
            @PostMapping(value = "/join/new")
            public String create(MemberForm form) {
             Member member = new Member();
             member.setName(form.getName());
             memberService.join(member);
             return "redirect:/";
            }
            ```

            - 여기서 submit을 누르면 join/new 링크로 톰캣서버에 요청. 그럼 컨트롤러가 해당 link와 연결된 postmapping 의 method 실행. 이때 입력 받은 정보들은 MemberForm 객체 (form)에 할당됨. 이제 이 form객체를 통해서 DB에 저장하면 되는 것.
            - 이때 중요한 것이 html에서의 input. input의 name속성에 해당하는 이름에 맞게 객체에 저장됨.

            ```html
            <input type="text" id="name" name="name" placeholder="이름을
            입력하세요">
            ```

            - 예를 들면 input name=”name” 이면 MemberForm의 name 속성에 setName을 통해 해당 입력 값이 저장. input name=”job” 이면 MemberForm의 job 속성에 setJob을 통해 해당 입력 값이 저장...
    - 회원 조회
        - 이전과 같이 GetMapping으로, Model을 실어서 진행됨.
        - 이때는 Post가 아니기에 인자로 사용되는 모델은 html파일로 넘어가는 객체임.
        - 이를 html에서 thymeleaf sytax를 통해 표현.

        ```java
        @GetMapping("/")
            public String home(Model model){
                List<Member> members = memberService.findMembers();
                model.addAttribute("members", members);
                return "home";
            }
        ```

        ```html
        <tr th:each="member : ${members}">
         <td th:text="${member.id}"></td>
         <td th:text="${member.name}"></td>
        </tr>
        ```



## Chapter 2


- 스프링 DB 접근 기술
    - h2 DataBase
        - h2.bat 실행
        - 데이터베이스 파일 생성 방법
            - jdbc:h2:~/test (최초 한번 → db 파일 생성 및 연결 - 파일모드로 접근 하는 것!)
              ~/test.mv.db 파일 생성 확인 (C:\Users\ *H2가 저장된 사용자*)
              이후부터는 jdbc:h2:tcp://localhost/~/test 이렇게 접속 (네트워크 모드_tcp 로 접근하는 것)
            - localhost:8082로 접근 가능 (db 파일을 생성하고 네트워크로 접근하기에 서버로써 접근이 가능한 것. 하나의 포트를 차지하고 있는 것)
            - 즉, 처음 접근 시, db파일을 만들어줌과 동시에 포트를 할당해주고, 그 이후부터는 네트워크 모드로 해당 포트로 접근하여 db를 연결하는 것. 이를 통해 springboot application(포트8080)과 db(포트8082)를 연결하여 사용할 수 있는 것.
        - Spring Boot에 jdbc, h2 DB 관련 라이버러리 및 DataBase Source 연결 설정 추가

            ```java
            // 1. in build.gradle
            implementation 'org.springframework.boot:spring-boot-starter-jdbc'
            runtimeOnly 'com.h2database:h2'
            
            // 2. in resources/application.properties
            spring.datasource.url=jdbc:h2:tcp://localhost/~/test
            spring.datasource.driver-class-name=org.h2.Driver
            spring.datasource.username=sa
            spring.datasource.password=1234
            ```
        
    1. 순수 Jdbc
        - JPA를 사용하지 않고 JDBC API로 직접 코딩.
        - 현재는 주로 JPA를 사용하므로 사용하지는 않는 방법. 하지만 JPA가 없던 시절 해당 방법을 통해 DB에 접근함
        - 직접 SQL문을 사용하며 이를 다루는 코드 또한 복잡하고 지속적인 예외처리가 필요하고 반복적인 코드가 다수임.
        - 주요 코드
            
            ```java
            // public class JdbcMemberRepository implements MemberRepository
            private final DataSource dataSource; //데이터베이스 커넥션을 획득할 때 사용하는 객체
            
            // Constructor
            public JdbcMemberRepository(DataSource dataSource) { //DI
             this.dataSource = dataSource;
             }
            ```
            
            DataSource 는 DB Connection을 획들학 때 사용되는 객체. 스프링 부트는 application.properties 에서 설정한 DB Connection 정보(사용 DB, DB 서버 URL 등)를 바탕으로 DataSource를 생성하고 이를 스프링 빈으로 만들어줌. 그래서 이를 생성자에서 Dependency Injection이 가능한 것.
            
            ```java
            // in SpringConfig (스프링 환경설정 객체, @Configuration)
            @Bean
             public MemberRepository memberRepository() { //DI
            // return new MemoryMemberRepository(); 
            return new JdbcMemberRepository(dataSource); // 내가 연결하고 싶은 DB에 따라 이 한줄만 변경하면 DB에 연결됨
             }
            ```
            
            다형성 활용하여 코드변경 없이 연결만 변경해줌. 이 부분이 바로 스프링을 쓰는 가장 큰 이유 → 객체지향적인 설계가 좋기 때문.
            
            다형성을 굉장히 편리하게 사용할 수 있게 스프링이 컨테이너를 통해 지원함
        
    2. 스프링 JdbcTemplate
        - 순수 Jdbc와 동일한 환경설정
        - 지금도 JPA와 함께 실무에서 많이 쓰이는 DB 접근 기술
        - 스프링 JdbcTemplate과 MyBatis 같은 라이브러리는 JDBC API에서 본 반복 코드를 대부분 제거. 하지만 아직 SQL은 직접 작성해야 한다는 단점 존재.
        - JdbcTemplate 객체 사용. 생성자를 통해 JdbcTemplate에 DataSource를 연결해줘야함.
        - 주요 코드
        
        ```java
        // public class JdbcTemplateMemberRepository implements MemberRepository
        private final JdbcTemplate jdbcTemplate;
        
        public JdbcTemplateMemberRepository(DataSource dataSource) { //DI
         jdbcTemplate = new JdbcTemplate(dataSource);
         }
        ```
        
        생성자를 통해 JdbcTemplate에 DataSource 연결
        
        ```java
        @Override
        public Member save(Member member) {
        	    SimpleJdbcInsert jdbcInsert = new SimpleJdbcInsert(jdbcTemplate);
        	    jdbcInsert.withTableName("member").usingGeneratedKeyColumns("id");
        	    Map<String, Object> parameters = new HashMap<>();
        	    parameters.put("name", member.getName());
        	    Number key = jdbcInsert.executeAndReturnKey(new MapSqlParameterSource(parameters));
        	    member.setId(key.longValue());
        	    return member;
        	}
        
        @Override
        public Optional<Member> findById(Long id) {
                List<Member> result = jdbcTemplate.query("select * from member where id = ?", memberRowMapper(), id);
                return result.stream().findAny();
            }
        
        // mapping tool
        private RowMapper<Member> memberRowMapper() {
        	 return (rs, rowNum) -> { // ReusltSet(rs)를 통해 해당 조건에 맞는 Member를 가져옴
        	 Member member = new Member();
        	 member.setId(rs.getLong("id"));
        	 member.setName(rs.getString("name"));
        	 return member;
        	 };
         }
        ```
        
        JdbcTemplate이 제공하는 method들 덕분에 순수 JDBC에서 길고 반복적인 코드가 획기적으로 줄어들며 효율적으로 사용가능해짐. 하지만 여전히 SQL문은 필요로 함.
        
        ```java
        // in SpringConfig
        @Bean
         public MemberRepository memberRepository() { //다형성
        // return new MemoryMemberRepository();
        // return new JdbcMemberRepository(dataSource);
         return new JdbcTemplateMemberRepository(dataSource); // JdbcTemplat으로 연결만 바꿔주면 됨
         }
        ```


1. JPA
    - JPA는 기존의 복잡하고 반복적인 코드는 물론이고, 기본적인 SQL도 JPA가 직접 만들어서 실행하여 훨씬 간편해짐.
    - JPA를 사용하면, SQL과 데이터 중심의 설계에서 **객체 중심의 설계**로 패러다임을 전환. 즉 ORM(Object Relational Mapping, 객체 자체를 DB와 연관 지어 Mapping 하는 것). 이로 인해 JAP를 사용하면 개발 생상성 크게 증가.
    - JPA는 일종의 interface. 이를 받는 여러 구현체들(hiberante 등)이 있음. But, 기본적으로 hibernate를 사용.
    - `implementation 'org.springframework.boot:spring-boot-starter-data-jpa'` 을 build.gradle 에 추가하여 라이브러리를 추가.  `spring.jpa.show-sql` 을 통해 해석되는 sql문을 확인할 수 있고, `spring.jpa.hibernate.ddl-auto=none` 를 통해 자동으로 Table이 생성되는 것을 방지해야 함.
    - 주요 코드

        ```java
        // in damain/Member
        @Entity // Object - Relational Mapping Annotaition
        public class Member {
         @Id @GeneratedValue(strategy = GenerationType.IDENTITY) // ID config
         private Long id;
         private String name;
        	.
        	.
        }
        ```

      @Entity : ORM에서의 Mapping 기능. 해당 Object를 RelationalDB에 Mapping 하겠다는 의미의 어노테이션

      @Id @GeneratedValue(strategy = GenerationType.IDENTITY) : Id 속성은 자동으로 생성되며 값이 중복되지 않아야 하므로 해당 설정을 통해 결정.

      나머지는 Table의 속성과 같은 이름을 가지게 하면 됨. 만약 다르다면 @Column(name = "username") 을 통해 이름을 Mapping 시킬 수 있음

        ```java
        //	public class JpaMemberRepository implements MemberRepository
        
         private final EntityManager em; // Jpa는 entitymanger로 모든 것을 동작함 cause ORM!!
         public JpaMemberRepository(EntityManager em) { // em은 스프링 내부에서 자동으로 만들어주기에 우린 injection만 해주면 됨.
        	 this.em = em;
         }
        
         public Member save(Member member) {
        	 em.persist(member); // 저장
        	 return member;
         }
         public Optional<Member> findById(Long id) {
        	 Member member = em.find(Member.class, id); // (조회할 타입, 식별자 pk)
        	 return Optional.ofNullable(member);
         }
        ```

      이전의 JdbcTemplate 보다 훨씬 간단해졌으며, SQL문도 작성하지 않아도 됨. 또한 ORM을 기반하였기에 개발 생산성이 향상됨.

        ```java
        // in SpringConfig
        private final EntityManager em;
        
        @Bean
         public MemberRepository memberRepository() {
        // return new MemoryMemberRepository();
        // return new JdbcMemberRepository(dataSource);
        // return new JdbcTemplateMemberRepository(dataSource);
         return new JpaMemberRepository(em);
         }
        ```

    - 추가로 JPA를 통한 모든 데이터 변경은 트랜잭션 안에서 실행해야 하므로 @Service Component에 @Transaction 추가 필수.


1. 스프링 데이터 JPA
    - 이미 스프링 부트와 JPA를 통해 개발 생산성이 많이 증가했지만 스프링 데이터 JPA는 마법처럼 인터페이스 +  추가적인 구현 클래스 없이 인터페이스 만으로 리포지토리를 개발할 수 있음. 또한,  CRUD 기능도 스프링 데이터 JPA가 모두 제공.
    - Class가 아닌 Interface로 사용하며 JpaRepository<E, key> 사용.
      JpaRepository를 상속받으면 자동으로 컴포넌트 스캐너가 인식하여 이를 빈으로 등록함.
    - 주요 코드

        ```java
        public interface SpringDataJpaMemberRepository extends JpaRepository<Member,
        Long>, MemberRepository {
        	 Optional<Member> findByName(String name);
        } // 
        ```

      이렇게 단 한줄로 이름으로 Member를 찾는게 가능해짐. → 스프링 데이터 JPA가 모두 구현 했기 때문, 또한 findBY00 형식을 지켜 원하는 속성으로 데이터를 찾아낼 수도 있음. 혁신 그 자체.

        ```java
        @Configuration
        public class SpringConfig {
        	 private final MemberRepository memberRepository;
        	
        	 @Autowired // 생략 가능
        	 public SpringConfig(MemberRepository memberRepository) {
        	 this.memberRepository = memberRepository;
        	}
        
        @Bean
        public MemberService memberService() {
        		return new MemberService(memberRepository);
        	 }
        }
        ```

      스프링 데이터 JPA가 SpringDataJpaMemberRepository 를 스프링 빈으로 자동 등록


- **AOP**
    - AOP : Aspect Oriented Programming
    - 모든 메소드의 호출 시간을 측정하고 싶을 때와 같이 공통 관심 사항에 대해 구현하고 싶은 것이 생길 때 쓰는 기술.
    - 모든 메소드에 대해 각 시간 측정 코드를 추가하는 것이 아닌 AOP 하나만으로 모든 메서드에 적용.
    - 즉, 공통 관심 사항(cross-cutting concern)과 핵심 관심 사항(core concern)을 명확히 구분하여 효율적인 개발을 위한 기술
    - AOP는 DI가 존재하기에 가능한 기술. → 내가 만약 DI를 설정하지 않고 직접 연결했다고 하면 당연히 각 코드별로 AOP에 해당하는 기능을 넣어줘야 함. 하지만, DI가 있기에 AOP를 따로 만들어주기만 하면 DI에 따라 AOP가 자동으로 실행되는 것.

      ![가짜 객체(프록시)를 만들어 해당 객체에 접근하기 전에 이를 인지할 수 있도록 함]({{site.baseurl}}/images/posts/post0/3.png)
      가짜 객체(프록시)를 만들어 해당 객체에 접근하기 전에 이를 인지할 수 있도록 함

      ![Untitled]({{site.baseurl}}/images/posts/post0/4.png)

    - 주요 코드

    ```java
    @Component
    	@Aspect
    	public class TimeTraceAop {
    		 @Around("execution(* hello.hellospring..*(..))")
    		 public Object execute(ProceedingJoinPoint joinPoint) throws Throwable {
    			 long start = System.currentTimeMillis();
    			 System.out.println("START: " + joinPoint.toString());
    			 try {
    				 return joinPoint.proceed();
    			 } finally {
    				 long finish = System.currentTimeMillis();
    				 long timeMs = finish - start;
    				 System.out.println("END: " + joinPoint.toString()+ " " + timeMs + "ms");
    			 }
    			}
    }
    ```

  시간을 측정하는 로직을 별도의 공통 로직으로 만들어 회원가입, 회원 조회등 핵심 관심사항과 시간을 측정하는 공통 관심 사항을 분리. 고로 핵심 관심 사항을 깔끔하게 유지 가능. 변경이 필요할 때 해당 AOP만 수정해주면 되며 적용 대상을 직접 선택할 수 있음.