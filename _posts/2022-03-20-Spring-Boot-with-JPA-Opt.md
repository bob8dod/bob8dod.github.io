---
layout: post
title:  "Spring Boot + API (JPA 최적화) [part 1]"
date:   2022-03-20
image:  /posts/jpa2.jpeg
tags:   SpringBoot JPA
---

<aside>
💡 <strong>Spring Boot에서의 API</strong>
<ul>
<li> 요즘같은 경우 직접 백에서 템플릿 엔진을 통해 데이터를 얹은 html을 랜더링 하여 client한테 주기보다는 Front에서 API로 받아온 데이터를 사용하여 Vue.js, React 등을 통해  Client한테 화면을 주는 추세. So, API 개발을 잘 할 수 있어야 됨. (APP에서는 무조건 API로 데이터를 받아옴) </li>
<li> API 같은 경우 기존 패키지들과 패키지를 분리하는 것이 좋음 -> 화면과 공통처리해야될 요소가 다르기 때문 </li>
</ul>
</aside>

## Chapter 1 [API 개발 기본]

- 회원 API
    - RestAPI 사용 → @RestController 
    = @Component(Spring Bean으로 등록) + @Responsebody(JSON 이나 XML로 데이터를 바로 보내는 용도)
    
    - 회원 등록 API
        1. **[Version 1]** Entity 그대로 데이터를 받아오는 Version (회원 등록) → `public CreateMemberResponse saveMemberV1(@RequestBody @Valid Member member)`
            - @PostMapping
            - @RequestBody : Json으로 온 HttpBody를 Member에 mapping(바인딩) 시켜줌 (즉, Json을 Member Entity로 바꿔준다 생각하면 됨)
            - @Valid : 해당 객체에 @NotEmpty가 있다면 그 부분에 대해서 validation을 진행해주는 것
            - Postman과 같은 API 사용 앱을 통해 Json형태로 해당 Entity에 맞는 형식으로 데이터를 보낼 수 있음. ver1에선 이렇게 받은 Entity를 memberservice를 통해서 저장까지 진행.

              ![]({{site.baseurl}}/images/posts/post-220320/Untitled 0.png)
                
            - 하지만 항상 강조하지만, 화면 자체에서 주고 받는 데이터는 Entity 그 자체이면 안됨! → 보안문제가 있을 수도 있고, 각 API 스펙에 맞는 Object가 필요함 (용도에 맞게 쓸 수 있는 객체인 DTO가 필요!)
                - API 스펙에 의한 별도의 DTO를 사용하자!
                - DTO: Data Transfer Object
        2. **[Version 2]**  Entity를 사용하지 않고 DTO를 사용하여 데이터를 받아옴 (회원 등록) → `public CreateMemberResponse saveMemberV2(@RequestBody @Valid CreateMemberRequest request)`
            - DTO 사용
                - 데이터를 받아 올 때 → `CreateMemberRequest request`
                - 데이터를 반환 해 줄 때 → `CreateMemberResponse`
                
                ```java
                @Data // @Getter + @Setter + ...
                static class CreateMemberRequest { // DTO
                    @NotEmpty
                    private String name;
                }
                ```
                
            - DTO를 통해 받아 오면, 그 DTO의 값을 새로운 Entity에 심어준 후 그 Entity를 EntityManger를 통해 저장하면 됨!
            - API 스펙에 맞게 DTO을 사용하면 유지보수에 큰 장점이 있음!
            
    - 회원 수정 API
        - REST API 의 Put을 사용 → `@PutMapping("/api/v2/members/{id}")`
        - url을 통해 값을 받아옴 → `@PathVariable("id") Long id`
        - Request, Response Data 모두 Entity가 아닌, DTO를 사용.
            - Request : `@RequestBody @Valid UpdateMemberRequest request`
            - Response :
                
                ```java
                @Data
                @AllArgsConstructor // 모든 파라미터를 받는 생성자 생성
                static class UpdateMemberResponse{
                    private Long id;
                    private String name;
                }
                ```
                
        - 변경 감지를 통해 회원 수정 진행 → Entitiy Manger가 관리 하는 영속성 컨텍스트 안에서 정보를 수정!
        
    - 회원 조회 API
        1. **[Version 1]** Entity 그대로 데이터를 보내는 Version (회원 조회) - Entity 를 그대로 반환 → `public List<Member> membersV1()`
            - @GetMapping 사용 → `@GetMapping("/api/v1/members")`
            - Member Entity에서의 orders에 @JsonIgnore 사용 : Entity를 API스펙에 맞추어, 보내지 않을 데이터(요소)에 설정해주는 것. (얘는 Json으로 보내지 않겠다!, JsonIgnore을 사용하지 않으면 API가 요구하는 스펙에 상관없이 해당 Entity의 모든 정보를 보내지게 됨.) → But, 다른 API 스펙에 맞지 않게됨. (실무에서는 같은 엔티티에 대해 API가 용도에 따라 다양하게 만들어지는데, 한 엔티티에 각각의 API를 위한 프레젠테이션 응답 로직을 담기는 어렵다.) → DTO사용 필요!
            - 이처럼 Entity를 직접 보내지게 된다면 보안 문제든, 쓸데없는 데이터가 보내지는 등, 좋지 않은 현상이 발생함! → DTO를 통해서 데이터를 보내줘야 함!
        2. **[Version 2]** Entity를 사용하지 않고 DTO를 사용하여 데이터를 보내는 Version (회원 조회) - DTO 사용 -> `public Result membersV2()`
            - Response: DTO 사용
            - Result<T> (Respone DTO)
                - 제네릭 사용.
                - 한번 감싸주기 위함. 그냥 MemberList를 반환하면 JSON으로 list만 반환 되므로 유연성이 떨어짐 (추후에 count등 부가적인 데이터를 추가하기 위해선 감싸줘야함)
                    
                    ```json
                    { 
                    	"count" : 3, //이렇게 부가적인 데이터를 넣어주기 위함
                    	"data" : [
                    		{ 
                    			"id" : 1,
                    			"name" : "PSI"
                    			},
                    		{ 
                    			"id" : 2,
                    			"name" : "PHB"
                    			}	 
                    	]
                    }
                    ```
                    
                    ```java
                    @Data
                    @AllArgsConstructor
                    static class Result<T>{ 
                        private int count; 
                        private T data;
                    }
                    ```
                    
            - 기본적으로 Entity를 조회 한 후에, 그 각 Entity를 DTO로 변환하여 반환하는 것.
                - `List<MemberDto> collect = findmembers.stream().map(m -> new MemberDto(m.getName())).collect(Collectors.*toList*());`
                - `return new Result(collect.size(), collect);`
            - DTO를 통해 정말 노출할 것들만 노출하는 것!
            
- Init DB 생성
    - 개발단계에서, 실제 App단에서 어떻게 진행되는지 Test할 때, 데이터를 계속해서 Create-Drop하기에, 애초에 시작할 때 부터 Data를 init해주는 것. (Entity의 요소들이 계속해서 수정되기에 create-drop 해주어야 됨)
    - init DB를 실행하는 class를 component로 등록해줌. → application안에서 동작하는 것이기에, Spring Bean으로 등록해줘야 함.  → `@Component`, `@RequiredArgsConstructor`
    - `public class InitDb`
        - Spring Bean
        - Spring Bean으로 등록되자마자 DB를 저장하게 됨. () → `@PostConstruct` : 스프링 빈에 등록되면 그 후 바로 실행되는 구문 (인스턴스 할당 없이 실행되는 생성자)
        - EntityManger를 사용하는 InitService Component를 injection 받음. (InitService는 InitDB에서만 사용하기에 내장 class로 선언해줌.)
    - `static class InitService`
        - @Component, @Transactional annotation 필수. (Service의 역할)
        - EntityManger를 injection 받음.
        - 실제 em을 통한 영속성 관리 (persist, 조회, ...)가 진행되는 곳
        - 굳이 postconstruct에 넣지 않고 service를 통해 em관리를 하는 이유 : postconstruct에서 직접적으로 transactional을 먹일 수 없기에.
        - TIP! ctrl + alt + M → method 추출

## Chapter 2 [API 개발 고급 (1)]

- **FethType.LAZY (지연로딩)일 때의 조회 성능 최적화! (xToOne)**
- 주문 조회 API
    - 주문 + 배송정보 + 회원을 조회하는 API
    - RestAPI 사용 → @RestController 
    = @Component(Spring Bean으로 등록) + @Responsebody(JSON 이나 XML로 데이터를 바로 보내는 용도)
    - 지연 로딩에 의한 성능저하를 해결하는 방향으로.
    - xToOne (FetchType.LAZY) 과 관련된 조회 성능을 올리는 방법
    - 즉, Order에서 Member와 Delivery 정보까지 받아오기
        - Order ↔ Member : ManyToOne
        - Order ↔ Delivery : OneToOne
        - ~~Order ↔ OrderItems : OneToMany (Collection 조회, API 개발 고급 (2)에서 진행)~~
        
    1. **[Version1]** 엔티티를 그대로 직접 노출 → `@GetMapping("/api/v1/simple-orders")`
        - `public List<Order> ordersV1()` : Entity를 그대로 직접 반환
        - @JsonIgnore 없이 그대로 모든 Order를 조회하면 Member와 Delivery, OrderItems는 양방향 관계이기에 무한루프에 빠지게 됨. Order→ Member → Order → Member → ...
        - 즉, 줄 중 하나는 JsonIgnore을 통해 Json으로 반환될 때 연결을 끊어줘야됨. (Order 조회 이므로 Member에서의 Order를 참조하는 부분에 @JsonIgnore 추가해주기!)
        - But @JsonIgnore 로 인한 프록시 문제가 발생함.
            
            <aside>
            💡 <strong> 왜 @JsonIgnore을 하면 프록시 문제가 발생하는가? </strong> <br> 이는 Member의 #serializeFields 메소드에서 props 변수가 직렬화할 필드목록 배열의 순서와 관계가 있음.
            Member를 기준으로 배열 순서에 따르면 ‘orders’가 먼저 오고, 그 다음으로 ‘hibernateLazyInitializer(Lazy설정으로 인한 프록시 초기화 상태라는 것을 알려주는 변수)’ 가 오는데 [orders → @JsonIgnore을 하지 않으면 무한루프 오류 발생 / hibernateLazyInitializer → Lazy 강제 초기화를 하지 않으면 프록시 오류 발생],
            이 때 @JsonIgnore을 하지 않으면 orders (member와 양방향 참조 관계)에 접근하게 되고 해당 값을 가져오기 위해 order에 접근하게 됨. 이 때 무한루프가 발생하게 되고, 이 무한루프의 에러에서 빠져나오지 못하기 때문에 프록시 문제가 발생하지 않는 것. (hibernateLazyInitializer의 순서는 order 뒤임!)
            하지만, @JsonIgnore을 달아주게 되면 order를 무시하게 되고 바로 hibernateLazyInitializer로 가게 되어 프록시 문제가 발생하게 되는 것.
            !! 중요한 점은 이 모든 오류(무한루프, 프록시)는 JSON으로 변환하는 시점에서 발생하는 것 !!
            
            </aside>
            
            - `Hibernate5Module()` : Json이 프록시 상태의 Entity를 무시하도록 설정하기 (프록시 상태의 Entity값을 null로 설정해줌.) (Entity에 해당하는 것!, 즉 DTO를 사용할 때는 사용할 수 없음) → HibernateModule을 사용하는 것이 아닌 DTO를 사용하는 것이 최고! → 즉, Entity를 직접 노출하지 말고 DTO를 사용하자!
                
                ```java
                @Bean // 프록시와 관련된 데이터를 처리하기 위한 모듈 설정 및 등록
                    Hibernate5Module hibernate5Module() {
                        Hibernate5Module hibernate5Module = new Hibernate5Module();
                //		hibernate5Module.configure(Hibernate5Module.Feature.FORCE_LAZY_LOADING, true); // 강제 지연 로딩 설정 -> 프록시 상태가 아닌 실제 Entity를 가져올 수 있도록 하는 것. -> query가 무지막지하게 나감(성능 저하!)
                        return hibernate5Module;
                    } 
                ```
                
            - 직접 Lazy 강제 초기화
                - 각 order에서 member의 값에 강제로 접근하여 member를 프록시 상태에서 벗어나게끔. →
                `order.getMember().getName();`
                `order.getDelivery().getAddress();`
                    - 원리 : member의 값을 가져오라는 명령이 없으면 프록시 상태로 두지만, member의 일부 값을 가져오라는 명령이 주어지면, member의 실제 값을 알아야 하기 때문에, em을 통해서 해당 Entity를 실제로 가져오게 됨 → Lazy 강제 초기화. (getname을 하면 member의 실제 값을 보내줘야되기 때문에 DB에 쿼리를 날리는 것!)
            - 그럼 FetchStyle.Eager로 사용하면 안됨? → 똑같은 문제가 발생하며, 또 다른 문제가 발생할 가능성도 큼. 또한, 다른 API스펙에 맞추기 어려움. (ex, 난 Order만 필요한데, Eager땜시 order와 관련된 Member, Delivery 등등 스펙에 맞지 않는 데이터도 보내줌! → 성능문제의 원인)
        - 하지만 당연히 이렇게 Entity를 직접 노출하는 것은 유지 보수, 보안에 있어서 치명적이게 됨! → **DTO 사용!**
    
    2. **[Version 2]** Entity를 DTO로 변환해서 반환 → `@GetMapping("/api/v2/simple-orders")`
          - `public List<SimpleOrderDto> ordersV2()` : DTO로 변환해서 반환
          - Entity값을 DTO로 변환해주어 반환 → `List<SimpleOrderDto> result = orders.stream().map(o -> new SimpleOrderDto(o)).collect(Collectors.*toList*());`
          - API가 요구하는 스펙은 주문ID, 주문자 이름, 주문지, 주문시간 이라고 가정. → DTO을 해당 스펙에 맞게 설정해야 됨.
          - DTO (Entity를 생성자로 받아와 Entity의 값들을 얻어옴)
              - 이때 Order Entity의 Member는 Lazy로 프록시상태. → getName()을 통해 접근하여 실제 Entity를 가져옴
              - 똑같이 Order Entity의 Delivery는 Lazy로 프록시상태. → getAddress()을 통해 접근하여 실제 Entity를 가져옴
            
              ```java
              @Data
              public class SimpleOrderDto { // Data의 스펙을 명확하게 규정해야됨. -> 내가 받을 것, 또 내가 보내줄 것을 명확하게 규정해야됨 (이런 이유때문에 DTO를 사용하는 것)
                  private Long orderId;
                  private String name;
                  private LocalDateTime orderDate;
                  private OrderStatus orderStatus;
                  private Address address;
            
                  // DTO는 Entity를 참조해도 괜찮음
                  public SimpleOrderDto(Order o) {
                      orderId = o.getId();
                      **name = o.getMember().getName();** // Lazy 강제 초기화 및 변수 설정 -> 여기서 조회 qeury가 나가는 것 (Lazy니깐!)
                      orderDate = o.getOrderDate();
                      orderStatus = o.getStatus();
                      **address = o.getDelivery().getAddress();** // Lazy 강제 초기화 및 변수 설정 -> 여기서 조회 qeury가 나가는 것 (Lazy니깐!)
                  }
              }
              ```
            
          - 하지만 ver2 는 order에 따른 member와 delivery를 조회하기 위해 query가 너무 많이 나감! → 성능저하의 원인
          - 해당 문제가 1+N 문제!!
              - 1 : 모든 Order를 조회하기 위해 하나의 쿼리가 날라가지만
              - N : 1을 통해 얻어온 Order List에서의 각각의 Order는 또 member를 참조하고 있고 Delivery를 참조하고 있음! 그에 맞는 쿼리를 또 날려줘야 함!
              - 흐름을 통해 이해 → 만약 OrderList에 Order가 2개가 있다면, 이 2개의 Order를 모두 조회하기 위한 첫번째 query 1가 나감. 해당 query를 통해 받아온 OrderList의 각 Order에 접근하는데 첫번째 Order에 접근했을 때, member와 delivery를 참조하고 있기 때문에 이에 대한 member와 delivery를 가져오는 2개의 query가 나감. 또 다음 Order에 접근할 때도 똑같이 member와 delivery를 가져오기 위해 2개의 query가 또 나감. 즉, 1 + [2 + 2](N) 개의 쿼리가 나가는 꼴. → 코드 흐름대로 이해하면 됨!
              - ver1도 똑같은 성능저하를 얻게됨. ( → getName(), getAddress(), ...)
          - 해당 문제를 해결해야 됨! (굉장히 복잡한 비지니스 구조라고 생각해보면 이는 명백한 성능 저하의 원인이 될 수 있음!)
          - 이는 **fetch join**을 통해서 해결 가능! (join 개념)

    3. **[Version 3]** Fetch Join을 통한 성능 최적화 (DTO) → `@GetMapping("/api/v3/simple-orders")`
        - Repository에서 Order 들을 조회할 때, fetch join을 사용하는 것!
        - 테이블을 한번에 묶어서(join) 조회하는 것 → query 하나로 Order, Member, Delivery 를 한번에 조회할 수 있음! (성능 개선)
        - 프록시로 채우는 것이 아닌, 실제 Entity로 채워버리는 것. (즉, 조회할 때부터 Lazy 강제 초기화를 해버리는 것)
        
            ```java
            public List<Order> findAllWithMemberDelivery() {
                    return em.createQuery(
                                            "select o from Order o " +
                                    "join fetch o.member m " +
                                    "join fetch o.delivery d", Order.class)
                            .getResultList(); // join fetch는 해당 연관관계의 Table을 그냥 합쳐서 한번에 다 가져오는 것
                    // 프록시로 채우는 것이 아닌, 실제 Entity로 채워버리는 것. (즉, Lazy를 무시하는 것 -> Lazy를 무시하는 것이지 Eager가 아님)
                }
            ```
        
            - JPA는 ORM이기에 이렇게 한번에 묶은 Fetch Join을 사용하게 되면 그대로 Order Entity와 그 묶여진 테이블이 매핑되어 Order.getMember.getName() 과 같이 Spring Boot에서 정의한 변수명 그대로 접근하여 가져올 수 있음.
        
            <aside>
            💡 <strong> Fecth Join vs Join </strong>
            <ul>
            <li> <strong> Fecth Join </strong> : 사실 join과 똑같은데, select절에 그 join된 데이터를 넣어주냐 마느냐의 차이. 즉, join해서 DB에서 해당 관련 모든 데이터를 select해서 반환해 주는게 fetch join. (즉, ORM과 관련되어 있음!)
            <li> <strong> join </strong> : 일반 join. join하고 select할 것을 따로 설정해 줘야됨
    
        - 나머지 Entity를 가져와서 DTO로 변환하는 과정은 Ver2와 동일
        - Qeury 수가 1+N 에서 1로 성능 최적화!
        - 성능 최적화는 사실 여기까지면 어느 정도 해결됨. 하지만, 불러와야 하는 데이터 Col의 수가 너무나도 많다면, API 스펙에 맞는 DTO의 Col에 해당하는 것들만으로 조회가 가능함 → DB를 DTO로 바로 조회
    
    4. **[Version 4]** DTO로 바로 반환
        - Entity로써 반환하는 것이 아닌 DTO 자체로 바로 반환
        - 불러와야 하는 데이터 Col의 수가 너무나도 많을 때 사용. But, 그렇게 자주 사용하지는 않음.
        - 또한, Ver 3까지만 해도 어느정도의 성능 최적화는 가능! (웬만하면 Ver 3을 쓰자~)
        - DTO 클래스로 조회
        
            ```java
            public List<OrderSimpleQueryDto> findByDto() { //JPA의 Entity조회가 아닌 DTO를 조회는 것이므로 fetch는 사용 불가
                    return em.createQuery(
                            "select new jpabook.jpashop.repository.OrderSimpleQueryDto(o.id, m.name, o.orderDate, o.status, d.address) " +
                                    // o 자체로 넣을 수 없음 -> o는 객체가 아닌, 식별자이기에. (DB입장) -> 그래서 각 colum을 넣어줘야됨.
                                    "from Order o " +
                                    "join o.member m " +
                                    "join o.delivery d", OrderSimpleQueryDto.class)
                            .getResultList();
                }
            ```
        
            - JPA의 Entity조회가 아닌 DTO를 조회는 것이므로 fetch는 사용 불가 → join 사용. (차피 DTO에 원하는 Col들을 지정해서 넣어줄 것이라 상관없음) (Fecth Join은 ORM과 관련되어 있음!)
            - 생성자 안에 Order Object 즉, o 자체로 넣을 수 없음 -> o는 객체가 아닌, 식별자이기에. (DB입장) -> 그래서 각 colum을 넣어줘야됨.
        - API가 원하는 스펙으로 DTO를 설정.
        
            ```java
            @Data
            public class OrderSimpleQueryDto { // Data의 스펙을 명확하게 규정해야됨. -> 내가 받을 것, 또 내가 보내줄 것을 명확하게 규정해야됨 (이런 이유때문에 DTO를 사용하는 것)
                private Long orderId;
                private String name;
                private LocalDateTime orderDate;
                private OrderStatus orderStatus;
                private Address address;
        
                // DTO는 Entity를 참조해도 괜찮음!
                public OrderSimpleQueryDto(Long orderId, String name, LocalDateTime orderDate, OrderStatus orderStatus, Address address) { // 각 order에 대한 member와 delivery를 serach하기 위한 query가 추가적으로 발생함
                    this.orderId = orderId;
                    this.name = name;
                    this.orderDate = orderDate;
                    this.orderStatus = orderStatus;
                    this.address = address;
                }
            }
            ```
        
        - 위에서 봤던 것들 처럼 기존에 사용할 수 있는 JPA의 장점들이 많이 사라지며, 코드 또한 복잡해짐. So, 웬만하면 Ver 3를 통해 간단히 최적화를 진행하되, Col이 너무 많은 데이터에서 적은 Col만을 조회할 것이라면 DTO로 조회하는 것도 좋은 선택.
        - SELECT 절에서 원하는 데이터를 직접 선택하므로 DB 애플리케이션 네트웍 용량 최적화(생각보다 미비)
        - 리포지토리 재사용성 떨어짐, API 스펙에 맞춘 코드가 리포지토리에 들어가는 단점
        - So, 보통 이런 API 스펙에 맞게 조회 및 반환하는 Ver4 같은 경우 조회 로직 (리포지토리) 자체를 기존 리포지토리와 구분해서 다른 package에 만들어 놓음.  (유지 보수를 위함!)

