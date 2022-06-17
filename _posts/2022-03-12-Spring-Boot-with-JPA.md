---
layout: post
title:  "JPA를 이용한 Entity, Service, Repository 개발"
date:   2022-03-12
image:  /posts/JPA+ESR.jpg
tags:   SpringBoot JPA
---

## 어플리케이션 구현 준비

- % 앞선 과정들(도메인 설계)은 Data측면에서의 준비였고, 이제 이런 도메인들을 가지고 어떻게 구현할 것인지에 대한 이야기.
- 구현 요구사항
    1. 회원 기능 → 회원 등록, 회원 조회
    2. 상품 기능 → 상품 등록, 상품 수정, 상품 조회
    3. 주문 기능 → 상품 주문, 주문 내역 조회, 주문 취소
- Application Architecture

  ![]({{site.baseurl}}/images/posts/post-220312/Untitled 6.png)

    - 계층형 구조: Controller, Service, Repository, DB, Domain
        - controller, web: 웹 계층
        - service: 비즈니스 로직, 트랜잭션 처리
        - repository: JPA를 직접 사용하는 계층, 엔티티 매니저 사용
        - domain: 엔티티가 모여 있는 계층, 모든 계층에서 사용 가능


## 회원 도메인 개발

- 회원 Repository 개발 (MemberRepository)
    - `@Repository` : component로써 Spring Bean으로 자동 등록
    - 영속성 컨텍스트 및 Entity를 관리할 Entity Manger injection하기  → `EntityManager em;`
        - `@PersistenceContext` : spring이 JPA의 entity manager을 생성하여 해당 em에 injection해줌
    - 회원 저장 → `void save(Member member)`
        - `em.persist(member)` : 받아온 member를 저장. (transaction이 commit 되는 시점(transactional로 감싼 method가 종료될 때 commit 됨)에 DB에 반영됨.)
    - 단일 조회 → `Member findOne`
        - `em.find(Member.class, id)` : 내가 조회할 Entity Class, PK
    - 다건 조회 → `List<Member> findAll()`
        - `em.createQuery(qlString, resultClass)` 사용 → jql query는 기존 sql query와는 다르게 Table이 아닌 Entity 객체를 대상으로 query를 작성해야됨. 즉, from의 대상이 table이 아닌 Entity
            - `qlString` : `"select m from Member m"` → 대상이 Table이 아닌 Entity!
            - `resultClass` : `Member.class` → 내가 조회할 Entity Class (DB입장에선 Table)
            - `.getResultList();` : 결과를 List로 반환
    - 다건 조회 (이름으로 검색) → `List<Member> findByName(String name)`
        - `qlString` : `"``select m from Member m where m.name = :name"` → where문 작성. name을 parameter로 바인딩
            - `.setParameter("name", name)` → name을 parameter로 바인딩
        - `resultClass` : `Member.class`
        - `.getResultList();` : 결과를 List로 반환

- 회원 Service 개발 (MemberService) → `class MemberService`
    - `@Service` : component로써 Spring Bean으로 자동 등록
    - `@Transactional` : [필수] JPA의 모든 데이터 변경, 조회는 Transactional 안에서 실행 되어야 함. → 본질적으로 repository에서 데이터 저장, 변경, 조회가 이루어지지만, repository는 service 하위에 존재하기에 service에 transactional annotation을 달아줌
        - 여기서 만약 조회만 한다 하면 `@Transactional(readOnly = true)` 를 사용하여 성능을 더 올릴 수 있음 → 이를 각각 method에 annotation을 달아 override할 수도 있음!
    - `@RequiredArgsConstructor` : final이 붙어 있는 애들에 대해서 자동으로 생성자를 통해 Autowired(DI) 실시. 즉, 자동으로 constructor injection 실시 -> lombok 기능 → `private **final** MemberRepository memberRepository;` (memberService에선 저장소를 가지고 기능을 구현하기에 repository를 DI 받아야함.)
        - field injection : 편리함. 하지만 추가적인 설정(injection 하는 대상을 변경하는 등)이 불가능
        - setter injection : 추가적인 설정 가능. but 외부에서 변경할 수 있다는 단점을 가지고 있음
        - constructor injection : 추가적인 설정 + 외부에서 변경 불가능

            ```java
            @Autowired
            public MemberService(MemberRepository memberRepository) {
                this.memberRepository = memberRepository;
            }
            ```

            - 이게 @RequiredArgsConstructor를 통해 자동으로 생성된다는 것!
            - 이를 바탕으로 다른 class에서 이전에 Autowired 된 것들을 변경해주면 됨!
    - 기능 구현
        - 회원 가입 → `Long join(Member member)`
            - 중복 회원 검증 : `validateDuplicated(member);`
                - 중복된 회원이면 EXCEPTION throw! → `throw new IllegalStateException("Already Exited Member!")`
                - 동일한 이름을 가진 member가 단 하나라도 발견되면 안됨 → `memberRepository.findByName(member.getName()).stream().findAny();` `if (findMember1.isPresent()) {...}`
            - `memberRepository.save(member);`
            - `return member.getId();` :  영속성 컨텍스트에 들어가는 순간, id값을 부여받기 때문에, DB에 commit하지 않아도 id를 확인할 수 있음
        - 회원 조회
            - 단일 회원 조회
                - `@Transactional(readOnly = true)` → 조회 성능 올리기
                - `return memberRepository.findOne(memberId);`
            - 전체 조회
                - `@Transactional(readOnly = true)` → 조회 성능 올리기
                - `return memberRepository.findAll();`
- 기능 TEST
    - tdd → inteliJ settings의 live template을 통해 기본 test case template을 만들어 두고 사용 가능!

      ![]({{site.baseurl}}/images/posts/post-220312/Untitled 7.png)

    - 회원 Service Test (MemberServiceTest) → `class MemberServiceTest`
        - `@SpringBootTest` : Spring Boot 상에서 Test하겠다는 뜻. 이 annotation이 없다면 Java상에서 Test가 돌아가게 됨.
        - `@Transactional` : Rollback을 위함. service에서 작성한 것과는 별개로 필요함 -> Rollback을 위해. 즉, Test에서의 transactional은 service에서의 transactional과 달리 기본적으로 Commit하지 않고 Rollback해버림
        - DI 해줘야 하는 것들 : `MemberService`, `MemberRepository`, `EntityManager` (em은 사실 flush를 위함, 필수는 아님)
            - memberservice와 memberrepository를 통한 Test가 이루어지므로
    - 회원 가입 Test → `void join() throws Exception`
        - `given (주어진 것들을 통해)` , `when (이런 기능을 동작했을 때)` , `then (이런 결과를 확인할 것)` 의 형식을 이용
        - given에 Member를 생성, when에 member를 memberservice를 통해 저장. then을 통해 생성된 member가 이미 존재한 회원인지를 확인 → *`assertEquals*(member, memberService.findOne(savedId));` (같은 transactional (영속성 컨텍스트) 안에서 Id(PK)값이 똑같으면 오류 발생!)
            - Assert는 Test에서 굉장히 유용하게 쓰이는 library
    - 회원 중복 가입 Test → `void duplicatedJoin() throws Exception`
        - given - 동일한 이름을 가진 member를 2개 생성
        - when - 먼저 첫번째 member를 저장
        - then - 동일한 이름을 가진 2번째 member를 저장. 여기서 오류 발생을 기대하는 것! (중복 감지 method에서 throw된 exception을 받아야됨.)

            ```java
            IllegalStateException thrown =assertThrows(IllegalStateException.class, () -> memberService.join(member2));
            assertEquals("Already Exited Member!", thrown.getMessage());
            ```

            - try catch 대신 assertEquals로 오류가 발생했는지 판단.
        - 추가로 `Assert`의 `fail()`을 통해 예외를 강제로 발생하여 해당 코드까지 진행되면 안되게끔 설정도 가능함.
    - 서버 DB를 사용하지 않고 내장 Memory DB를 사용하여 Test 진행하는 법!
        - 외부 DB를 키지 않고 내장 Membory DB를 통해 빠르게 Test가 가능함.
        - 또한, 실제 Main에서 진행하는 코드들과 Test에서 진행하는 test는 엄연히 다른 것! → DB를 구분해줄 필요가 있음!!!
        - 이는 Test에 resources를 새로 생성해서 Test용 application.yml을 설정해주면 됨.

          ![]({{site.baseurl}}/images/posts/post-220312/Untitled 8.png)

            - 그 후 해당 application.yml에 새로운 DB 연결 정보를 넣어주면 됨.

                ```yaml
                spring: #springboot에 연결할 DB관련 정보
                  datasource:
                    url: jdbc:h2:mem:test
                ```

            - *`jdbc:h2:mem:test` →* 내장 메모리를 사용하는 DB
            - 하지만 사실 이 모든 설정을 하지 않고 test에 resources만 생성해 준다면 Spring Boot는 자동으로 DB를 내장메모리로 사용하게 됨.

## 상품 도메인 개발

- 상품 Entity 개발
    - [중요!] Service에 비지니스를 새로 만드는 것이 아닌, 상품 Entity 자체에 비지니스 로직을 추가하는 것.
    - 즉, Item Entity에 비지니스 로직 method를 추가해주는 것.
        - 원래 같으면 Service에서 set 으로 해당 비지니스 로직을 구현하겠지만, 객체 지향적으로 생각해보면 해당 객체의 값을 변경하는 것은 그 객체 안에서 진행하는 것이 가장 BEST (응집력 UP) → 이를 통해 Setter를 생성하지 않아도 되는 것. Setter 없이 프로그램을 만드는 게 안전함!
        - 재고 증가 로직 (method) → `void addStock(int quantity)`
            - `this.stockQuantity += quantity;`
        - 재고 감소 로직 (method) → `void removeStock(int quantity)`
            - 감소된 재고가 0보다 작으면 Exception을 thorw 해줘야됨. → `throw new NotEnoughStockException("Not Enough Stock now!!");`
            - Exception 생성 : `class NotEnoughStockException extends RuntimeException` 생성 후 RuntimeException Override!
- 상품 Repository 개발
    - 상품 저장 → `void save(Item item)`
        - 새로 저장되는 item인지, 수정하는 item인지 구분 필요.
            - `if(item.getId() == null){ em.persist(item); }` : 영속성 컨텍스트에 올라기지 않은 상태의 새로운 Item은 Id가 부여되지 않음. → 그렇기에 persist를 통해 영속성 컨텍스트에 올리고 저장하면 됨.
            - `else` : 하지만, 이미 영속성 컨텍스트에 올라가 Id를 부여받은 상태 (즉, 저장되어 있는 item) 일 경우, `em.merge(item);` 를 통해 준영속 상태의 엔티티를 영속 상태로 변경 → (merge 부분은 뒤에서 더 자세히 다룰 에정)
    - 상품 조회
        - 단건 조회 : `return em.find(Item.class, id);`
        - 다건 조회 : `return em.createQuery("select i from Item i", Item.class.getResultList()`;
- 상품 Service 개발
    - 사실 이부분에선 상품 Repository에 기능을 위임하는 정도 밖에 없음
    - 그래서 이 부분을 굳이 Service를 거쳐야 하나를 고민해볼 필요가 있음
    - 기본적으로 필요한 `@Service @Transactional(readOnly = true) @RequiredArgsConstructor` annotation 달아주고
    - ItemRepository를 DI 시켜준 후, 저장, 조회기능을 repository와 연결해주면 끝

## 주문 도메인 개발

- 주문 Entity 개발
    - Order Class
        1. Order에는 양방향 연관관계가 있으며 Order가 그 양방향 연관관계에서 우위를 점하고 있는 것들이 많음. → 생성 메서드를 Entity 자체에 만들어주면 좋음! → `**static** Order createOrder(Member member, Delivery delivery, OrderItem... orderItems)`
            - Order가 생성될 때 무조건 호출되어야 하는 놈
            - 연관관계 매핑 및 값을 설정하는 역할

            ```java
            //==생성 메서드==// (응집성 향상)
            public static Order createOrder(Member member, Delivery delivery, OrderItem... orderItems){ // ...은 args 느낌
              Order order = new Order();
              order.setMember(member); // 연관관계 매핑
              order.setDelivery(delivery); // 연관관계 매핑
              for (OrderItem orderItem : orderItems){
                  order.addOrderItem(orderItem); // 각 orderItem에 대한 연관관계 매핑
              }
              order.setStatus(OrderStatus.ORDER); // 값 설정
              order.setOrderDate(LocalDateTime.now()); // 값 설정
              return order; // 생성된 order 반환
            }
            ```

            - 이런 생성메서드만을 사용하게 하기 위해서 Order Class에 `@NoArgsConstructor(access = AccessLevel.PROTECTED)` 를 추가해줌
                - `@NoArgsConstructor(access = AccessLevel.PROTECTED)` : preotected 형태의 생성자를 자동으로 생성해 주는 것. 즉, 외부에서 생성자를 사용할 수 없게 하여, 생생 메서드를 필수로 사용하게끔 하는 전략.
        2. [중요!] Service에 비지니스를 새로 만드는 것이 아닌, 상품 Entity 자체에 비지니스 로직을 추가하는 것. (”비지니스 로직”)
            - 즉, Order Entity에 비지니스 로직 method를 추가해주는 것.
            - 주문 취소
                - Service에서 Setter를 사용한 값 변경이 아닌 Entity 자체에서 변경 -> 더 안정적인 프로그램 가능

                ```java
                public void cancel(){
                	  if(delivery.getStatus() == DelivieryStatus.COMPLETE){
                	      throw new IllegalStateException("이미 배송 완료된 상품은 취소가 불가능합니다.");
                	  }
                	
                	  this.setStatus(OrderStatus.CANCEL);
                	  for (OrderItem orderItem : orderItems) { // 주문에서 요구했던 item들에 대해 취소시켜줌
                	      orderItem.cancel(); // Order와 orderItem은 일대다 이기에 그 '다'의 모든 orderItem 각각에 대해 취소해줘야됨
                	  }
                }
                ```

            - 전체 가격 조회
                - 주문에서의 각 orderItem에는 해당 아이템이 얼만지, 아이템을 몇개를 샀는지 의 정보가 있음, 이들을 곱해서 더해줘야됨

                ```java
                public int getTotalPrice(){
                        return orderItems.stream()
                                .mapToInt(OrderItem::getTotalPrice)
                                .sum(); // 가격*수량 이 누적되어야 됨
                    }
                ```

    - OrderItem Class
        1. 생성 메서드 → `static OrderItem createOrderItem`
            - Order 처럼 생성될 때 연관관계도 있고 해서 간단하지 않음.
            - 그래서 Service에서 생성로직을 구현하는 것 보다는 Entity 자체에서 생성메서드를 구현하는 것이 응집력을 높임
            - 하지만, OrderItem과 Item은 양방향이 아닌, 단방향 연관관계이기에 연관관계 매핑은 필요 없어서, 생성메서드에 연관관계 매핑 관련 method는 존재하지 않음.
            - 하지만, Item에서 order에 따른 수량이 변경되기 때문에, 그 부분에 있어서의 연관관계에 따른 설정은 필요함.

            ```java
            //==생성 메서드==//
            public static OrderItem createOrderItem(Item item, int orderPrice, int count){
                OrderItem orderItem = new OrderItem();
                orderItem.setItem(item); // 단방향 연관관계 매핑 
                orderItem.setOrderPrice(orderPrice); // 값 설정 
                orderItem.setCount(count); // 값 설정
            
                item.removeStock(count); // OrderItem에서 Item과의 연관관계 메서드가 없는 이유. 
            		//-> Item에서 OrderItem은 일대 다 관계이지만 이 OrderItem을 리스트로 담아 관리하는 것이 아닌 그저 수량만을 관리하기 때문
                return orderItem;
            }
            ```

            - 이런 생성메서드만을 사용하게 하기 위해서 OrderItem Class에 `@NoArgsConstructor(access = AccessLevel.PROTECTED)` 를 추가해줌
        2. 비지니스 로직
            - 주문 취소 (Order및 Item과 연계되어 동작)
                - 주문 취소는 Order에 의해 연계되어 실행되고, 해당 Orderitem의 취소는 다시 Item에 연계되어 동작하게 됨. (취소하면 Item의 수량은 늘어나야 됨.)
                - `public void cancel() { getItem().addStock(count); }`
            - 전체 가격 조회
                - OrderItem이 Item을 통해 받은 수량과 가격의 곱을 통해 전체 가격을 조회할 수 있음
                - `public int getTotalPrice() { return getOrderPrice() * getCount(); }`
- 주문 Repository 개발
    - 필요한 annotation인 `@Repository` `@RequiredArgsConstructor` 를 달아주고 EntityManager를 DI 받은 후 em을 통해 저장과 단건조회 method를 만들어주면 됨.
    - 주문 저장 → `public void save(Order order){ em.persist(order); }`
    - 주문 단건 조회 → `public Order findOne(Long id){ return em.find(Order.class, id); }`
    - 주문 다건 조회 →  `public List<Order> findAll(OrderSearch orderSearch)` [동적 쿼리 필요 → QueryDSL(추후 이걸 주제로 배울 예정), 현재는 JPA Criteria로 보충]
        - OrderSearch class 필요.
            - 동적 쿼리의 선택지가 되는 두가지 변수 (회원 이름, 주문 상태)를 저장해서 OrderService로 보내주게 되는 객체.
            - 회원명이 있으면 회원에 따른 where문 , 주문상태가 있으면 주문상태에 따른 where문, 둘다면 둘다에 따른 where 문이 동작
- 주문 Service 개발
    - `@Service` `@Transactional(readOnly = true)` `@RequiredArgsConstructor` annotation을 통해서 Bean 등록, Transactional에서 동작, 자동으로 생성자를 통한 DI 실시 진행
    - Order Service에서는 Member와 Item에 접근하여 Order에 저장하는 비지니스 로직을 가지고 있기에 orderRepository 뿐만 아니라 memberRepository, itemRepository 또한 DI가 필요함.
    - 주문 기능 → `Long order(Long memberId, Long itemId, int count)`
        - 엔티티 조회
            - 멤버id를 통해 Member를 가지고 오는 것. → `memberRepository.findOne(memberId);`
            - Item id를 통해 Item을 가지고 오는 것. → `itemRepository.findOne(itemId);`
        - 배송정보 생성
            - Delivery Entity 생성 → `Delivery delivery = new Delivery(); delivery.setAddress(member.getAddress());`
        - 주문 상품 생성
            - Entity 관계도를 참고하면 Order에 바로 Item을 넣어 주는 것이 아닌 OrderItem을 거쳐서 집어넣음 -> 다대다 관계이기 때문
            - OrderItem 자체는 연관관계도 있고 값들도 많기에 Entity 자체에 생성메서드를 만들어 놓았기에, 이를 활용. → `OrderItem orderItem = OrderItem.***createOrderItem***(item, item.getPrice(), count);`
            - 또한, Order 전에 OrderItem을 생성하여 Order 생성메서드의 인자로 넣어줘야 하기에 Order 전에 생성.
        - 주문 생성
            - Order도 Entity 자체가 간단하지 않기에 Entity 자체에 생성 메서드를 만들었었음
            - `Order order = Order.*createOrder*(member, delivery, orderItem);`
        - 주문 저장
            - `orderRepository.save(order);` : 앞선 과정들을 바탕으로 orderRepository를 통해 save 진행.
            - 원래같으면 order와 연관되어 있지만 따로 persist를 진행하지 않았던 delivery, orderItem도 따로 save 하여 presist해줘야됨. **BUT!** 이 둘다 Order Entity 속 연관관계에서 `cascade = CascadeType.ALL` 로 설정 했기에 자동으로 저장됨
            - 이는 Order Entity가 em을 persist 될 경우 영속성 컨텍스트 안으로 들어가기에 가능한 부분
                - 영속성 컨텍스트에 Order가 올라가게 되면 그에 연계된 Delivery, OrderItem 또한 올라가게 됨. 이때 Transactional 에서 commit을 진행하면, `cascade = CascadeType.ALL` 인 연계 Entity에 대해서도 commit을 해주는 것!
                - `cascade = CascadeType.ALL` 은 다른 Entity에서 참조하지 않고 해당 객체에서만 참조를 할때 사용해야됨.
    - 취소 기능 **[중요! → JPA의 장점을 확인할 수 있는 부분] →** `void cancelOrder(Long orderId)`
        - 주문 Entity 조회 → `orderRepository.findOne(orderId);`
        - 주문 취소 → `order.cancel();`
            - 만약 JPA를 활용하지 않았더라면 -> cancel하면서 변경된 **order의 state**, **orderitems의 item의 수량**들을 모두 변경하고 다시 DB에 업데이트 시켜줘야됨 (즉, 각각에 대한 query를 날려줘야 된다는 것.)
            - 하지만 JPA는 Entity내에서 변경된 점을 자동으로 업데이트 해줌. 즉, 변경된 지점을 찾아서 그 지점에 관한 update query를 자동으로 DB로 보낸다는 것 -> dirty checking(변경 감지)
            - 이게 가능한 이유는 지금 취소하려는 주문 자체가 em을 통해 DB에서 가져왔기 때문 (`orderRepository.findOne(orderId);`) → em이 DB에서 Entity를 가져오게 되면, 그 Entity는 em의 영속성 컨텍스트에 머무르게 됨. → 이때 그 Entity의 값들이 변경되게 되면 em은 이를 감지하고 (변경감지, dirty checking) 해당 method의 Transactional이 끝나 commit이 이루어지면, 이와 관련된 모든 qeury를 자동으로 날려주는 것.
    - 주문 조회 → 동적 쿼리 사용 (뒤에서 설명)

    <aside>

  💡 <strong> [참고] </strong> <br> 주문 서비스의 주문과 주문 취소 메서드를 보면 비즈니스 로직 대부분이 엔티티에 있다. 서비스 계층은 단순히 엔티티에 필요한 요청을 위임하는 역할을 한다. 이처럼 엔티티가 비즈니스 로직을 가지고 객체 지향의 특성을 적극 활용하는 것을 <strong> 도메인 모델 패턴 </strong>이라 한다. 반대로 엔티티에는 비즈니스 로직이 거의 없고 서비스 계층에서 대부분의 비즈니스 로직을 처리하는 것을 트랜잭션 스크립트 패턴이라 한다. (보통 JPA나 ORM을 사용하는 경우 도메인 모델 패턴을 이용함)

    </aside>

- 주문 기능 Test
    - `@SpringBootTest @Transactional` annotation 필수 + `@Test`
    - 사실 Test는 SpringBoot 위에서 진행하는 것 보다는 SpringBoot 없이, 즉 DB 연결 없이 단일적으로 비지니스만을 체크하는 Test가 좋은 Test. (하지만 여기선 JPA의 동작을 확인하는 것이기에 SpringBoot 위에서 Test 진행)
    - 상품 주문 Test
        - 주문이 들어 갔을 때 → 주문 상태, 주문된 상품의 종류, 주문 총 금액, 주문 후 해당 상품의 남은 수량이 비지니스 로직에 맞게 잘 설정 되는지 *`assertEquals(expected, actual, message)`* 를 통해 확인
        - given : `Member member, Item item, int orderCount`
        - when : `Long orderId = orderService.order(member.getId(), item.getId(), orderCount);`
        - then

            ```java
            Order ordered = orderRepository.findOne(orderId);
            assertEquals(OrderStatus.ORDER, ordered.getStatus(), "상품 주문 시 상태는 ORDER");
            assertEquals(1,ordered.getOrderItems().size(), "주문한 상품 종류 수가 정확해야 한다");
            assertEquals(10000*2, ordered.getTotalPrice(), "주문 가격은 가격 * 수량");
            assertEquals(20-2, item.getStockQuantity(), "주문 수량만큼 재고가 줄어야 한다.");
            ```

    - 상품주문 시 재고수량 초과 Test
        - 주문한 수량이 재고보다 넘을 경우 *`assertEquals(expected, actual)`*를 통해 Exception이 발생하고 그 Exception message가 expected와 동일한지 확인
        - given : `Member member, Item item, int orderCount` → 여기서 orderCount가 item에 설정된 수량보다 많게 설정
        - when : `NotEnoughStockException thrown = *assertThrows*(NotEnoughStockException.class, () -> orderService.order(member.getId(), book.getId(), orderCount));` → 초과된 수량으로 주문을 할 때의 튀어나오는 Exception을 catch해서 저장해둠
        - then : *`assertEquals*("Not Enough Stock now!!", thrown.getMessage());` → 동일한 message로 Exception이 발생했는지 확인
    - 주문 취소 Test
        - 주문 취소 시 → order의 status가 CANCEL 로 변하는지 확인, 취소 수량이 원복되는 지 확인
        - given : `Member member, Item item, int orderCount`, `Long orderedId = orderService.order(member.getId(), item.getId(), orderCount);`
        - when : `orderService.cancelOrder(orderedId);`
        - then

            ```java
            Order ordered = orderRepository.findOne(orderedId);
            assertEquals(OrderStatus.CANCEL, ordered.getStatus(), "주문 취소 시 상태는 CANCEL 이다.");
            assertEquals(20, item.getStockQuantity(), "취소된 상품은 재고가 다시 원복되어야 함");
            ```

