---
layout: post
title:  "Spring Boot + JPA Basic [part 1]"
date:   2022-03-09
image:  /posts/jpa2.jpeg
tags:   SpringBoot JPA
---

## Chapter 1

![]({{site.baseurl}}/images/posts/post-220309/Untitled 0.png)

- 프로젝트 환경 설정
    - [start.spring.io](http://start.spring.io) 에서 손 쉽게 spring boot를 시작할 수 있음
    - Dependiencies
        - spring web : 웹 어플리케이션 개발 시 꼭 필요한 것. (RESTful API 지원, Spring MVC, Tomcat 지원)
        - Thymeleaf : 탬플릿 엔진. Web Viewer 역할. 웹MVC와 함께 필수적인 요소. SpringBoot 와 integration 되면서 많은 기능을 사용할 수 있어짐
        - Spring Data JPA : 영속성 객체를 가지고 데이터를 다루게 해줄 수 있음 (+ Hibernate) → JPA를 완전히 안 상태에서 쓰는 것을 권장.
        - H2 Database : 데이터 베이스는 H2 (SQL)
        - Lombok : 여러 번 중복되는 코드들을 줄여줌 (getter/setter 등)
        - % 얘네와 함께 필요한 애들은 알아서 SpringBoot가 땡겨와줌.
        
- 생성된 프로젝트에서 **build.gralde** 확인하기
    - **build.gradle** : 현재 spring boot의 버전, 어떤 dependencies를 가지고 생성되었는지, 각 depedencies의 설정은 어떻게 되었는지 등. 프로젝트의 통합적인 설정을 하는 곳.
    - 주요 라이브러리
        - Lombok 설정 (`annotationProcessor -> annotation` 만으로 편하게 중복되는 코드들을 제거할 수 있음)
            
            ```java
            configurations {
            	compileOnly {
            		extendsFrom annotationProcessor
            	}
            }
            ```
            
            - Lombok 활성화 하기
                1. settings에서 pulgins에 들어가 lombok pulgin을 설치한 후 IDE 재시작.
                2. settings에서 annotation processers에 들어가 ‘Enable annotation processing’ 체크.
            - Lombok의 장점
                - getter/setter를 자동으로 만들어 줌
                - 원래는 해당 객체 안에 getData, setData 등 getter, setter를 만들어 줘야 했지만, Lombok을 사용하게 되면 해당 객체에 @Getter @Setter annotation만 달아주게 되면 따로 만들지 않아도 get, set 가능!
        - dependecies
            - 프로젝트 설정 시 연결해주었던 요소들이 모두 들어가 있는 것을 확인할 수 있음
        - Thymeleaf
            - 템플릿 엔진 (용도 : Serving web service with Spring MVC)
            - 서버 사이드 렌더링 역할! → Server Side Rendering(SSR) : 서버에서 페이지를 그려 클라이언트(브라우저)로 보낸 후 화면에 표시하는 기법
            - html 마크업 자체에 thymeleaf 문법을 사용함. 즉, html 마크업을 깨지 않고 thymeleaf를 사용할 수 있음. → springboot를 띄우지 않아도 기본 웹 브라우저로 열림.
            - Spring과 integration되어 훨씬 더 유동적으로 사용할 수 있음. 사용할 수 있는 기능들도 많아짐.
            - But, 문법이 좀 까다로움 → Documnets, Manual 참고 필수
            - 사용하기 → [https://spring.io/guides/gs/serving-web-content/](https://spring.io/guides/gs/serving-web-content/) Spring Guide에서 잘 나와 있음. 그대로 따라하면 됨.
                - TIP! → [https://spring.io/guides](https://spring.io/guides) Spring은 guide를 제공하는데, 여기서 웬만한 경우는 다 있기에, 막히면 해당 가이드에서 찾아 해결할 수 있음.
            - 스프링 부트 thymeleaf viewName 매핑 : resources:templates/ +{ViewName}+ .html.
                - Controller에서 Mapping이 이루어진 하나의 method에서의 반환 값이 {ViewName}. 즉, 해당 {ViewName} 만을 반환해주면 thymeleaf에서 알아서 위의 경로로 html파일을 찾아 이를 method에서 받은 model과 함께 serving함.
            - % static에 index.html 을 통해 동적인 페이지가 아닌 정적인 페이지를 만들어 보낼 수도 있음. (클라이언트가 web에 main을 요청 시, 따로 기본 home에 rendering을 하지 않았다면 자동으로 static의 index.html로 넘어감.)
            - TIP! → html 변경 후 (server restart 없이) 바로 적용 하기! → devtools library 추가! build.gradle의 dependencies에 `implementation 'org.springframework.boot:spring-boot-devtools'` 추가하면 html 수정 후 html만 recompile 해주면 바로 적용 가능!
            - % Thymeleaf는 추가적으로 공부가 필요!
        - h2 DataBase
            - h2.bat 실행
            - 데이터베이스 파일 생성 방법
                - jdbc:h2:~/test (최초 한번 → db 파일 생성 및 연결 - 파일모드로 접근 하는 것!)
                ~/test.mv.db 파일 생성 확인 (C:\Users\ *H2가 저장된 사용자*)
                이후부터는 jdbc:h2:tcp://localhost/~/test 이렇게 접속 (네트워크 모드_tcp 로 접근하는 것)
                - localhost:8082로 접근 가능 (db 파일을 생성하고 네트워크로 접근하기에 서버로써 접근이 가능한 것. 하나의 포트를 차지하고 있는 것)
                - 즉, 처음 접근 시, db파일을 만들어줌과 동시에 포트를 할당해주고, 그 이후부터는 네트워크 모드로 해당 포트로 접근하여 db를 연결하는 것. 이를 통해 springboot application(포트8080)과 db(포트8082)를 연결하여 사용할 수 있는 것.
                
- JPA와 DB 설정
    - 기본적으로 속성 설정 파일은 application.properties로 제공되지만, application.yml (야믈)로 속성을 관리하는 것이 추후에 속성들이 늘어났을 때 관리하기 편함.
    - application.yml → DB와의 연결 정보, DB에 저장할 Entity들에 대한 설정 등이 담겨 있는 속성 설정 파일
        
        ```yaml
        spring: 
          datasource: #springboot에 연결할 DB관련 정보
            url: jdbc:h2:tcp://localhost/~/jpashop # 파일로 접근하는 것이 아닌 네트워크로 접근
            username: sa
            password:
            driver-class-name: org.h2.Driver # h2 DB 사용
        
          jpa: # DB와 상호작용하는 JPA설정
            hibernate:
              ddl-auto: create # 기존에 있는 Table을 날리고 Table을 자동으로 생성해주는 역할 <- 내가 생성한 Entity를 통해
            properties:
              hibernate: # JPA는항상 hibernate와 사용됨.
        #        show_sql: true # hiberante가 DB에 날리는 sql query를 complie내 output으로 보여줌
                format_sql : true # sql query를 보기 편하게 fomating
        
        logging:
          level:
            org.hibernate.SQL: debug # hibernate가 DB에 날리는 sql query를 로그에 남겨줌
            org.hibernate.type: trace # copile에 DB가 어떻게 Update되는지 출력해줌.
        ```
    
    - JPA와 DB 동작 확인
        - Entity (Member domain 객체)
        
        ```java
        @Entity // em이 관리하도록 Entity라고 표시해 주는 것
        @Getter @Setter // lombok을 통한 getter, setter 자동설정. -> 따로 method를 구현할 필요가 없음
        public class Member {
        
            @Id @GeneratedValue //식별자(PK, primary key)로 맵핑 | 자동생성
            @Column(name = "member_id") // member_id 로 DB에 Table에 저장한다.
            private long id;
        		...
        }
        ```
        
        - Repository (저장, 조회 등 직접적으로 DB와 상호작용)
            - `@Repository` : Spring Bean 으로 등록 (component, repository로써 등록)
            - `@PersistenceContext` : 영속성 컨텍스트, Spring이 JPA의 entity manager를 생성하여 em 변수에 injection 해주는 역할.
                - `EntityManager em` : injection 받으면, Entity annotation 을 가지고 있는 객체들을 관리(저장, 조회, 수정)
                    - `EntityManager` 를 통한 모든 데이터 접근은 항상 `@Transactional` 안에서 이루어져야됨. (Transactional이 어떻게 보면 하나의 context라고 생각하면 됨. 그 context 안에서 데이터 접근이 이루어지는 것! → 중요 개념!!)
        - 테스트 케이스 작성 (동작 확인)
            - `@SpringBootTest` : 연결된 DB에 접근하기에 단일테스트가 아닌 통합테스트로 진행. 즉, SpringBoot를 띄우고 그 안에서 직접 Test를 하는 것.
            - `@Autowired MemberRepository memberRepository;` : 현재 Test는 memberRepository를 이용해서 진행되므로 Test에 memberRepository를 DI(Dependency Injection) 해줌. (memberRepository는 component로 Spring Bean에 등록된 상태이기에 바로 Autowired로 DI가 가능한 것.)
            - `@Test` : Method 위에 달리는 annotation. 해당 method로 Test를 진행하겠다는 뜻.
            - `@Transactional` : 기본적으로 Service나 Repository annotation에 걸리긴 하지만, test에서는 rollback의 역할 등, 각각의 Test들이 독립적으로 실행될 수 있도록 하기위한 설정으로 사용됨. (Spring에서 제공하는 annotation을 사용해야 다양한 기능 활용 가능.)
                - `@Transactional` 은 Test에서 기본적으로 Rollback 성질을 가지고 있음. → DB에서 데이터가 잘 저장되었는지 등을 확인 못함. 이는 `@Rollback(false)` 혹은 `@Commit` 을 추가하여 직접 DB에 반영 가능.
            - `Assertions`
                - Test에서 굉장히 유용하게 잘 쓰이는 library.
                - `import org.assertj.core.api.Assertions;`
                - .isEqualTo() 와 같은 method들을 제공하며, 이를 통해 객체 비교, 에러 검출 등을 Test해서 수행할 수 있음
                - `Assertions.assertThat(findMember).isEqualTo(member);` 
                → member는 방금 저장된 Entity, findMember는 member의 id로 검색한 Entity. 결과는 True!
                → 현재 test는 transactional을 통해 영속성 컨텍스트 안에서 entity를 관리하고 있음.
                → 영속성 컨텍스트에서 식별자가 같으면 같은 Entity로 인식. (주소값 이런 것으로 판단하는 것이 아닌!)
                → 애초에 search하는 select query 조차도 나가지 않고 영속성 컨텍스트(1차 캐시)에 있는 것을 알고 바로 판단함.
        - TIP → 직접 DB에 날리는 query 확인하기
            - `implementation 'com.github.gavlyukovskiy:p6spy-spring-boot-starter:1.5.6' //  쿼리 파라미터를 로그로 남기는 외부 라이브러리` 를 추가하면 됨.
            - But, 이런 외부 공개 라이브러리는 기존 개발 단계에서는 괜찮지만, 실제로 사용자들에게 배포하고 운영할 때는 고민해봐야됨. (성능저하의 원인이 될 수 있기 때문!)
            
- jar로 실행하기 (Cloud Server에 올릴 때 이용 가능)
    1. cmd에서 해당 SpringBoot 파일이 있는 경로까지 이동
    2. gradle clean build → 현재 구축된 SpringBoot를 통해 jar 파일 build (gradle 구축, clean은 이전에 build된 것들을 삭제해주고 새로 build, clean을 빼면 그냥 누적 build)
    3. build/libs 로 이동 → build된 jar 파일이 저장된 곳
    4. java -jar jpashop-0.0.1-SNAPSHOT.jar → 빌드된 jar 파일 실행 → 완료!

- 더 빠르게 SpringBoot 가동하기
    ![]({{site.baseurl}}/images/posts/post-220309/Untitled 2.png)
 
    - Build and run using 과 Run tests using 을 IntelliJ IDEA로 변경하기

## Chapter 2 [도메인 분석 및 설계]

- 도메인 모델
  ![]({{site.baseurl}}/images/posts/post-220309/Untitled 3.png)
    
    - 회원과 주문은 “1대다” 관계. → 회원은 여러 개의 주문이 가능
    - 주문과 상품은 “다대다” 관계. → “다대다” 관계를 주문상품을 통해 “1대다”, “다대1” 관계로 풂. (”다대다” 관계는 RDB, Entity에서 거의 사용하지 않기에 → 사용하면 너무 복잡해짐)
    - 상품에는 “도서, 음반, 영화” 존재. 상품이라는 공통속성을 사용하여 상속 구조로 표현
- 테이블 설계
  ![]({{site.baseurl}}/images/posts/post-220309/Untitled 4.png)

    - Member : PK는 id. (모든 Entity의 PK는 id), address 는 embedded Type(Value Type, 값 자체로 들어가는 것. → 연관관계 X), orders를 통해 여러 주문을 List 형태로 받아들임
    - OrterItem : 다대다 관계를 표현하기 위한 설정. 추가로, 주문된 상품의 개수, 총 금액을 파악하기 위해서도 필요함. 또한, **Item과는 단방향 연관관계!!** (즉, Item에서는 OrderItem으로 접근할 수 없음)
    - % 사실 Member와 Order간의 “1대다” 관계는 옳지 않은 관계. 이같은 경우 Member를 Order 보다 높은 위치의 Entity로 본것. 하지만, 실무에선 Member와 Order는 동급! 즉, Member에 List로 Order를 받는 것이 아닌 Order에 그저 단 하나의 Member로 연결하면 됨 즉, 1대1관계로 표현이 가능하고, 훨씬 효율적임 → 그럼 만약 회원이 주문한 Order를 확인하고 싶다면? Order에서 필터링 조건으로 Member Id를 넣어주면 됨!
- 테이블 분석

  ![]({{site.baseurl}}/images/posts/post-220309/Untitled 5.png)
    
    - Member : 여기서 Enbedded Type의 성질이 드러남. → Embedded로 들어갔기에 그 Value Type(Address)의 값들이 Member Table에 그대로 들어감. (DELIVERY도 마찬가지.)
    - Item : 상속관계 매핑에서의 Single Table 전략 사용. (한 테이블에 모든 상속관계의 Entity들을 넣고 Dtype이라는 구분칸으로 자식들을 구분. 테이블이 복잡해진다는 단점이 있지만, 성능 측면에선 좋음)
    - % 각 관계에서의 FK가 중요! → FK를 가지고 있는 놈이 그 양방향 연관관계에서 우위를 점하고 있는 것! (1대다 관계는 무조건 다에 FK가 존재하게 됨)
        - 즉, 1대다 관계에서 1을 통해 직접적으로 다의 Data를 가져올 순 없는 것!
        - 즉, 양방향 연관관계의 주인이 그 관계의 데이터를 포함하고 있는 것이고, 주인이 아닌 을의 입장의 Table은 그 관계의 상대 데이터를 포함하고 있는 것이 아닌, **거울!** 단순히 읽기만 하는 입장!
        → Member와 Orders : Member에는 Orders와 관련된 데이터 자체가 저장 되지 않음 (RDB 입장에서). 즉, Order를 읽기만 하는 것. 하지만 Orders는 연관된 Member 자체를 가지고 있음 (RDB 입장에서). 즉, Order 자체에 Member가 연결되어 있는 것.
        
        > **참고**: 외래 키가 있는 곳을 연관관계의 주인으로 정해야 됨!
        연관관계의 주인은 단순히 외래 키를 누가 관리하냐의 문제이지 비즈니스상 우위에 있다고 주인으로 정하면 안됨!
        > 
        
- 엔티티 클래스 개발
    - 앞선 설계를 바탕으로 각각의 Entity를 개발
    - 실무에선 보통 Setter를 열어두면 안되지만, 간단한 예제를 위해 여기선 Setter를 열어둠. (보통 해당 Entity안에서 해당 값에 대한 변경이 이루어지도록 설정해야됨! 안 그러면 나중에 유지보수가 힘들어짐. Entity 외부에서 값이 변경되니깐!)
    - **잠깐!** Member와 Order를 매핑할 때, 누구에게 FK를 맞춰야하나, 즉 누가 이 연관관계의 주인으로 설정해야 되나! → Member에도 Order가 연관되어 있고 Order에도 Member가 연관되어 있는데 그럼 JPA는 값을 수정하거나 추가할 때 어떤놈을 기준으로 수정 및 추가를 진행해야 되냐? (즉, Order를 수정할 때 Member를 같이 수정해야되나, Member를 수정할 때 Order를 같이 수정해줘야 되나,,,! DB입장에선 서로 FK를 주며 연결할 수 있지만, JPA입장에서는 복잡해지기에 하나의 Table에만 FK를 주고 나머지는 자동으로 수정될 수 있도록 설정됨!) → JPA는 항상 다대일 관계에서 다에 해당하는 Entity를 연관관계의 주인으로 판단 → FK는 다대일에서 ‘다’에 설정해주면 됨.
        - 즉, 객체에서는 변경 포인트가 Member에서의 List<Order>, Order에서의 Member, 두군데이지만 JPA에서의 Table은 Order의 Member Foriegn Key 하나만 변경하면 됨. → JPA에서의 약속!
        - 즉, JPA는 둘다 바꾼다는 개념이 아닌, 다대일에서 연관관계의 주인을 ‘다’로 선정하고 ‘다’의 FK를 변경하여 그 연결된 ‘일’의 값도 변경되게 하는 것.
    - Member
        - Order와 “1대多” 양방향 연관관계에서 “1”에 해당, Address 라는 Ebedded Type을 내장.
        - `@Entity` : Entity Manager가 관리하도록 명시. JPA에서 활용되는 객체라는 것! → `class Member`
        - `@Getter @Setter` : Get, Set 자동생성. (원래 실무에선 Setter를 열어두면 안됨.)
        - `@Column(name = "member_id")` : 변수 위에 annotation 해주면 해당 변수명으로 Table에 저장되는 것이 아닌, 설정한 저 name으로 Table 속 Column에 저장됨.  →
        - `@Embedded` : 내장 타입이라는 것을 명시해 주는 것. → `Address address;`
        - `@OneToMany(mappedBy = "member")` : “일대다” 에서 “일”의 연관관계를 가지고 있다는 뜻. 양방향 연관관계에서 사용되며 얘는 연관관계에서 갑이 아닌, 그저 거울일 뿐이고 “다(Order)”의 member변수와 연결 당했다는 것을 알려주는 것. 즉, 읽기전용!!! → `List<Order> orders = new ArrayList<>();`
            - TIP! → Collection은 필드에서 바로 초기화 해주는 게 좋음. (`List<Order> orders = new ArrayList<>();`)
            → null 문제에 대해서도 안전함
            → collection은 영속성 컨텍스트에 들어가는 순간, hibernate만의 문법으로 정의됨. 즉, 초기화를 하지 않은상태에서 영속성 컨텍스트 안에 넣고, 그 이후로 초기화를 한다면 hibernate 문법에 맞지 않게됨!!
    - Address
        - Ebedded Type. Member와 Delivery에 내장됨.
        - `@Embeddable` : JPA의 내장 타입. 어떤 연관관계가 있다기 보다는 그냥 이 자체로 다른 Entity에 저장되는 것.
        - 각각에 대한 정보를 그저 변수로 저장하면 됨.
        
        > 참고: 값 타입은 변경 불가능하게 설계해야 한다.
        @Setter 를 제거하고, 생성자에서 값을 모두 초기화해서 변경 불가능한 클래스를 만들자. JPA 스펙상 엔티
        티나 임베디드 타입( @Embeddable )은 자바 기본 생성자(default constructor)를 public 또는
        protected 로 설정해야 한다. public 으로 두는 것 보다는 protected 로 설정하는 것이 그나마 더 안전
        하다.
        JPA가 이런 제약을 두는 이유는 JPA 구현 라이브러리가 객체를 생성할 때 리플랙션 같은 기술을 사용할 수
        있도록 지원해야 하기 때문이다
        > 
    - Order
        - `@Table(name = "orders")` : orders라는 이름으로 Table 저장.
        - Member와 “多대1” 양방향 연관관계에서 “多”에 해당. → `Member member;`
            - `@ManyToOne` : “多대1”, 연관관계에서 우선순위를 가져 FK를 가지게 됨.
            - `@JoinColumn(name = "member_id")` : Foreign Key! Meber의 member_id 로 Member를 연결하겠다.
        - OrderItems와 “1 대 多” 양방향 연관관계에서 “1”에 해당. → `List<OrderItem> orderItems = new ArrayList<>();`
            - `@OneToMany(mappedBy = "order")` : “일대다” 에서 “일”의 연관관계를 가지고 있다는 뜻. 양방향 연관관계에서 사용되며 얘는 연관관계에서 갑이 아닌, 그저 거울일 뿐이고 “다(OrderItems)”의 order변수와 연결 당했다는 것을 알려주는 것. 즉, 읽기전용!!!
            - `(cascade = CascadeType.*ALL*)` : order을 persist하게 된다면, 이에 따른 orderItems들도 persist가 자동적으로 수행 됨. (삭제도 마찬가지)
                - 원래는 order에 연결되는 각각의 orderItem들을 따로 persist해줘야 하지만,
                    
                    ```java
                    em.persist(orderItemA)
                    em.persist(orderItemB)
                    em.persist(orderItemC)
                    
                    em.persist(order)
                    ```
                    
                    cascade를 사용하게 되면 `em.persist(order)` 하나로 연계되어 있는 orderItem들을 자동으로 persist하게 됨.
                    
                - 이런 cascade는 보통 비지니스의 control하는 domain에 설정해주는 것이 편리하며, 비지니스 로직으로 봤을 때, 해당 객체에서만 사용되는 애들에 쓰는 것이 좋음. (orderItem은 Order가 생성 될때만 사용되는 Entity!)
        - Delivery와 “1대1” 양방향 연관관계. → `Delivery delivery;`
            - 1대1 양방향 연관관계에서 우선순위는 사실 정해진 것은 없음. 비지니스 관계에 따라 설정하면 됨. 여기서는 Order가 우선순위를 가지고 있다고 가정.
            - `@OneToOne(cascade = CascadeType.*ALL*)` : 1대1 양방향 연관관계,  order을 persist하게 된다면, 이에 따른 delivery도 persist가 자동적으로 수행 됨. (삭제도 마찬가지) (delivery는 order가 생성될때만 사용되는 entity!)
            - `@JoinColumn(name = "delivery_id")` : 양방향 연관관계에서 우선순위를 가지며, 연결되는 FK는 Delivery의 delivery_id 이다. 라는 뜻.
        - 주문 시각 저장 → `LocalDateTime orderDate;`
            - Tip! → JPA에서 orderDate 인 CamelCase로 선언하게 되면 DB Table에선 order_date 인 Under_score 형식으로 바뀌게됨. (DB에선 관례상 under score를 많이 쓰기에)
        - 주문 상태 저장 → `OrderStatus status;`
            - ORDER, CANCLE 이라는 두가지 상태 저장
            - `@Enumerated(EnumType.*STRING*)` : enum (열거형이라고 불리며, **서로 연관된 상수들의 집합을 의미**) Type을 Ordinary가 아닌 String으로 저장하겠다. (반드시 String으로 저장해야됨. Ordinary는 정보가 수정되면 순서가 바뀌게 되어 큰 오류를 초래!)
    - Delivery
        - Order와 “1대1” 양방향 연관관계. → `Order order;`
            - `@OneToOne(mappedBy = "delivery")` : 양방향 연관관계에서 우선순위를 갖지 못하고 그저 거울의 역할 만함 → mappedBy. 즉, Order의 delivery에 연결되어 있다는 것을 그저 명시하는 것. 읽기 전용
        - Address 라는 Ebedded Type 포함. (내장 타입) → `Address address;`
        - 배송 상태 저장 → `DelivieryStatus status;`
            - `@Enumerated(EnumType.*STRING*)`
    - OrderItems
        - Order와 Item을 다대다로 묶기 위해 사용 + 주문 가격과 주문 수량을 체크하기 위해 사용.
        - Items와 “多대1” 양방향 연관관계에서 “多”에 해당. → `Item item;`
        - Order과 “多대1” 양방향 연관관계에서 “多”에 해당. → `Order order;`
    - Item
        - 추상 클래스를 통해 Book, Album, Movie 가 상속받을 수 있도록 설정
            - `@Inheritance(strategy = InheritanceType.*SINGLE_TABLE*)` : Entity에서의 상속 전략 설정 (Single table 전략). 상속받는 모든 Class를 하나의 ITEM Class Table에 다 때려박는 전략
            - `@DiscriminatorColumn(name = "dtype")` : single_table 이기에 각 class에 대해 구분이 필요함. 이걸 DataType으로 구분한다는 뜻. → 각 Class는 별도의 Data Type으로 저장되게됨.
        - Category 와 “多대多” 양방향 연관관계. 을의 입장. → `List<Category> categories = new ArrayList<>();`
            - `@ManyToMany(mappedBy = "items")` : Category의 items에 연결당했다! → 읽기 전용.
    - Book
        - Item을 상속받는 class → `public class Book extends Item`
            - `@DiscriminatorValue("B")` : single table 전략으로 상속 전략을 사용하며 Dtype으로 구분한다고 설정했기에 이에 대한 DataType 설정 필요 (따로 설정 안하면 자동으로 해당 class 이름으로 Dtype 설정됨. 다른 Class들과 해당 Dtype으로 구분됨)
    - Category
        - Item과 “多대多” 양방향 연관관계. 연관관계의 우선순위를 가짐.
            - % 실무에선 직접적인 “多대多” 양방향 연관관계를 사용하지는 않음.
            - `@ManyToMany`
            - 말 그대로 join table을 생성하는 것. (하나의 테이블이다 생각하면 됨) 다대다를 JPA가 인식할 수 있도록 다대일, 일대다로 만들어주는 것. (하지만, 연관관계를 표시하는 것 이외에 다른 Col을 추가할 수 없기에 거의 사용하지 않음.)
                
                ```java
                @JoinTable(name = "category_item",
                    joinColumns = @JoinColumn(name="category_id"),
                    inverseJoinColumns = @JoinColumn(name="item_id"))
                ```
               ![]({{site.baseurl}}/images/posts/post-220309/Untitled 1.png)
                
                - `joinColumns = @JoinColumn(name="category_id")` : Category의 category_id 에 연결한다. (category와의 연관관계에서 우선순위를 가짐.)
                - `inverseJoinColumns = @JoinColumn(name="item_id")` : Item의 item_id에 연결한다. (Item과의 연관관계에서 우선순위를 가짐.)
                
                > 참고: 실무에서는 @ManyToMany 를 사용하지 말자 @ManyToMany 는 편리한 것 같지만, 중간 테이블( CATEGORY_ITEM )에 컬럼을 추가할 수 없고, 세밀하게 쿼리를 실행하기 어렵기 때문에 실무에서 사용하기에는 한계가 있다. 중간 엔티티( CategoryItem 를 만들고
                @ManyToOne , @OneToMany 로 매핑해서 사용하자. 정리하면 대다대 매핑을 일대다, 다대일 매핑으로 풀어
                내서 사용하자.
                > 
                
        - 계층 구조 형성
            - 다른 Entity와의 연관관계가 아닌, 자기 자신에서 연관관계를 만듦 (계층 구조) → 부모와 자식 연관관계. → 부모 Category와 자식 Category가 있다고 생각하면 됨.
            - 자식 Category의 입장에서 자식은 부모와의 관계에서 “多대1” 양방향 연관관계에서 “多”. 즉, 갑의 입장 → `Category parent;`
                - `@ManyToOne`
                - `@JoinColumn(name = "parent_id")` : parent의 parent_id에 연결하겠다.
            - 부모 Category의 입장에서 부모는 자식과의 관계에서 “多대1” 양방향 연관관계에서 “1”. 즉, 을의 입장 → `List<Category> child = new ArrayList<>();`
                - `@OneToMany(mappedBy = "parent")`
                
        - TIP! → fetch (LAZY vs EAGER)
            - Eager 와 Lazy
                - EAGER : 즉시 로딩. 해당 Table에 있는 모든 연관관계의 Table 또한 가지고 오는 것.
                - LAZY : 지연 로딩. 해당 Table만을 가져오는 것.
                - ManyToOne 을 Eager로 설정하게 되면 그와 연관된 모든 테이블을 가져오게됨 → em.find() 처럼 하나의 order를 가져오는거면 사실 상관 없지만
                - 만약 모든 Order를 가져온다고 할 때, ManyToOne이 Eager로 설정되어 있다면 모든 Order의 모든 Member가 딸려오게됨!!! (사실 모든 order를 확인하는 것이지 그 order의 member까지 알고싶지는 않아~) 그래서 Lazy를 사용한 다음, 필요하다면 그 때 order에서 member로 들어가는 설정으로 만들어야됨!! → Eager는 데이터나 서비스구조가 복잡해지면 사용해선 안됨!
                - xToMany는 자동으로 Lazy로 되어 있음. xToOne은 직접 Lazy로 설정해줘야됨.
                - 그래서 실무에서는 무조건 LAZY로 설정해야됨!
                
        - 양방향 연관관계 편의 메서드
            - Member와 Order같은 경우 양방향 연관관계.
            - Order가 생성될 때, meber를 따로 넣어줌. 그럼 member의 orderlist에 order는 언제, 어떻게 넣어줘야 할까? → 연관관계 메서드 필요!
            - 두 연관관계 중 핵심에 해당하는 부분에 구현해주는 게 좋음 → member와 order 둘을 기준으로 생각하면, order가 핵심.
            - 즉,  Member와 Order의 JPA와 DB관점에선 연관관계의 갑은 Order이며 Order와 Member는 양방향 연관관계. 근데 이 때 객체 관점에서 보면 Member에도 Order가 껴줘야됨. (양방향 연관관계) 이런 걸 실제 구현(Entity 외부 service 같은 곳)에서 설정할 수도 있지만, Order에 Member가 연결될 때, Member에도 Order를 넣어주는 method를 선언해주면 훨씬 깔끔하고 유용함
            - Order
                
                ```java
                public void setMember(Member member){
                    this.member = member;
                    member.getOrders().add(this); // 이 부분!
                }
                
                public void addOrderItem(OrderItem orderItem){
                    this.orderItems.add(orderItem);
                    orderItem.setOrder(this); // 이 부분!
                }
                
                public void setDelivery(Delivery delivery){
                    this.delivery = delivery;
                    delivery.setOrder(this); // 이 부분!
                }
                ```
                
            - Category
                
                ```java
                //==연관관계 메서드==//
                public void addChildCategory(Category child){
                    this.child.add(child); // 부모입장에서 현재 Entity에 chlid들을 넣어주는 것
                    child.setParent(this); // 그 각 child들에게 현재 부모를 지정해주는 것
                }
                ```
            