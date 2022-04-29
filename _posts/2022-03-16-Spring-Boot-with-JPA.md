---
layout: post
title:  "Spring Boot + JPA Basic [part 3]"
date:   2022-03-16
image:  /posts/jpa2.jpeg
tags:   SpringBoot JPA
---

## Chapter 6 [주문 도메인 개발]

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


## Chapter 7 [웹 계층 개발]

- Main 화면
    - Controller (HomeController), home.html,
    - 클라이언트의 요청을 1차적으로 받는 곳.
    - 클라이언트가 요청을 하면, 해당 요청을 TomCat 내장 서버가 이를 받아오고 Spring Container 속 Controller들 중 이 요청에  Mapping 하는 것이 있는지 확인함.
    - Main home 화면 매핑 → `String home()`
        - `@RequestMapping("/")` : main이 되는 화면의 Get 요청에 mapping
        - `return "home";` : 스프링 부트 thymeleaf viewName 맵핑 (resources:templates/ +{**ViewName**}+ .html.) 에 따라 resources의 templates의 home.html 을 HTTP 통신으로 Client에게 넘겨줌.
    - home.html
        - thymleaf 문법을 사용하며 header.html, bodyHeader.html, footer.html을 include하고 있음. (즉, include style. 이는 hierarchical style로 구성할 수도 있음)
        - `<html xmlns:th="http://www.thymeleaf.org">`
        - `<head th:replace="fragments/header :: header">`
        - `<div th:replace="fragments/bodyHeader :: bodyHeader"/>`
        - `<div th:replace="fragments/footer :: footer"/>`

- 회원 등록 화면
    - 회원 등록 Form → `class MemberForm`
        - **DTO : Data Transaction Object** 로직을 가지지 않는 데이터 객체이고 getter/setter 메소드만 가진 클래스. 즉, data를 옮기는 역할만 하는 객체
        - 화면에서 name, address(city, street, zipcode) 를 받아오는 역할
        - `private String name;` ← `@NotEmpty(message = "이름은 필수로 입력하셔야 됩니다.")` ⇒ `@NotEmpty` : validation 역할. 비어있으면 안되는 변수로 설정. 비어 있으면 해당 message를 날림.

        <aside>
        
      💡 <strong> memberForm을 쓰는 이유 </strong> (그냥 member로 입력된 값을 객체 자체로 받아오면 안됨?)
        <ul>
        <li> Entity를 web상에서 왔다갔다하는 form 데이터로 사용하면 화면에서 요구하는 데이터와 실제 Entity가 요구하는 값들이 안맞을 수도 있고 이걸 억지로 맞춰야하는 번거러움이 존재하기 때문 </li>
        <li> 또한 Validation을 사용하려면 그냥 form을 만들어서 그 form으로 데이터를 주고 받는 것이 남. </li>
        <li> 실무에서는 실제 화면에서 받는 데이터들은 훨씬 복잡하기 때문!! </li>
        <li> JPA를 사용한다면, Entity는 최대한 순수하게 유지해야 됨!!!! 
        </ul>
        </aside>

    - (Controller) 회원 등록 시 열어보는 용도 → Get → `String register(Model model)`
        - `@GetMapping("/members/new")`
        - `Model model`은 html로 넘어가 thymeleaf와 상호작용하는 놈. 즉, html에 데이터를 넘겨주는 역할. (model에 속성으로 데이터를 넣어주면 thymeleaf를 통해 html에서 받아온 데이터를 사용할 수 있음.)
        - `model.addAttribute("memberForm",new MemberForm());` : 빈 껍데기의 MemberForm 객체를 가지고 감
            - Validation 가능해짐.
            - 입력된 정보를 mebmerform을 통해 PostMapping으로 보내줄 수 있음. (즉, 객체로 보내지는 것)
        - `return "members/createMemberForm";` : resources의 templates의 members에서 createMemberForm.html을 HTTP 통신으로 Client에게 보냄.
    - (Controller) 회원 정보 작성 후 제출, 등록하는 용도 → Post → `String join(@Valid MemberForm memberForm, BindingResult result)`
        - `memberForm` : Get에서 담겨 보냈던 등록 Form으로 Post하면 해당 Form 객체에 정보가 담긴 채 Postmapping으로 오게 됨. (`@Valid` 를 통해 내가 해당 form에서 설정한 필수 값인 name이 담겨 있는지 체크 가능. 즉, 없다면 Error(오류화면)를 발생 시킬 수 있게 됨.)
        - `BindingResult result` : 오류화면으로 넘어가지않고, 그 값이 없는 상태의 error form자체를 그냥 받아와줌. 즉, 오류가 BindingResult에 담겨짐. 즉, 어떤 것이 Error인지 아는 상태가 되며, 그 오류를 담은 상태로 하위 코드들이 실행 되는 것. (결과를 바인딩한다.)
        - `if (result.hasErrors()){ return "members/createMemberForm";}` : 그 result에 error가 담겨 있다면, MemberFrom, BindingResult를 담아서 다시 creatememberForm으로 넘김. 그럼 방금 담겨 있던 정보들과 name 자체에 error가 발생했다는 것을 아는 상태로 넘어가는 것. → 이를 thymeleaf를 통해 이벤트를 생성할 수 있음. (-> spring과 thymleaf가 연동(integration)되어 있기에 가능한 것)
        - `memberService.join(member);` : 에러가 발생하지 않았다면, Address를 저장하고 Member를 저장.
        - `return "redirect:/";` : 재로딩하지 않고 다시 home으로 돌아감
        - html에서의 thymeleaf는 생략.

        <aside>
        💡 이런 Error 검출같은 것들과 같은 부가적인 기능은 Thymeleaf + Spring Docs를 통해서 원하는 기능을 찾아 쓰면됨 (굉장히 잘 나와 있음) [[Docs]](https://www.thymeleaf.org/doc/tutorials/3.0/thymeleafspring.html)

        </aside>


- 회원 조회 화면
    - (Controller) 회원 조회 → `String list(Model model)`
        - `@GetMapping("/members")`
        - `List<Member> members = memberService.findMembers();` : memberService (비지니스 로직) 을 통해 전체 회원 조회.
        - `model.addAttribute("members", members);` : html에 넘겨주는 model에 members list를 속성에 추가하여 넘겨줌.
        - `return "members/memberList";` : members의 memberList.html에 model을 넘겨주고 이를 바탕으로 해당 html을 Client에게 넘겨줌.

        <aside>
        💡 이 또한 사실 DTO를 통해 넘겨주는 것이 좋음. template 엔진에게 Etity를 넘기는 것은 괜찮다고 해도, 만약 API로 데이터를 넘긴다면 절대 Entity를 사용하지말고 DTO를 사용해야됨.

        </aside>


---

- 상품 등록 및 조회
    - 회원 등록 및 조회와 똑같은 로직으로 진행.
        - MemberForm → BookForm (수정을 위해 id를 추가한 form 생성)

- 상품 수정 **[중요!]**
    - 수정 (Get) → `String updateItemForm(@PathVariable("itemId") Long itemId, Model model)`
        - `@GetMapping("/items/{itemId}/edit")` : get url 자체에 변수를 넣어준 것.
        - 수정버튼을 누르게 되면 해당 item의 id가 url에 실리고 그 id를 바탕으로  DB에서 em을 통해 해당 객체를 가져오게 됨 → 그 후, 객체의 정보를 form에 옮긴 후 이를 화면에 출력.
        - `@PathVariable("itemId") Long itemId` : url에 담긴 변수를 itemId에 받아줌.
        - `Book item = (Book) itemService.findOne(itemId);` : 그 id를 바탕으로  DB에서 em을 통해 해당 객체를 가져오게 됨
        - `BookForm form = new BookForm();` `from.set(item.get())...` : 객체의 정보를 form에 옮긴 후
        - `model.addAttribute("form", form);` : 이를 화면에 출력.
    - 수정 (Post) → `String updateItem(@PathVariable("itemId") Long itemId, @ModelAttribute("form") BookForm form`
        - `@PostMapping("/items/{itemId}/edit")` : get url 자체에 변수를 넣어준 것.
        - 여기선 사실 PathVariable은 필요없음. → BookForm을 통해 id를 받아왔으므로.
        - 받아온 form를 통해 새로운 Book Entity를 생성해 준 후 itemService를 통해 저장하면 됨.
        - `Book book = new Book();` : 새로운 Book Entity를 생성 (수정하려고 하는 item의 id와 똑같은 id를 가지고 있는 book entity 생성)
        - `book.set(form.get())...` : 받아온 form의 정보를 새로운 Entity에 심어줌
        - `itemService.saveItem(book);` : 해당 Entity 저장.
            - 여기서 saveItem(저장 로직 method)를 보면 id가 있을 때와 없을 때를 구분지어 저장함.
            - id가 없을 때 → `em.persist(item)` (즉, 새로 저장되는 Entity)
            - **[중요!]** id가 있을 때  ****→  `em.merge(item)` (즉, 수정된 Entity는 병합 진행! merge는 덮어씌워주는 개념. but 실무에서 잘 안씀. → 변경하지 않은 값들에 대해서도 덮어씌어지며 이상하게 변할 수 있기 때문)
- 변경감지 vs merge
    - 공통점 : 어쨌든 변경감지를 통해 Update를 진행
    - 차이점 : 변경감지는 내가 원하는 값들만을 수정할 수 있지만, merge는 선택권없이 모든 값들을 수정(병합)함. → 변경감지를 쓰는 이유
    - merge → `em.merge()`
        - 위에서 진행한 로직 그대로 사용할 경우의 상황
        - 즉, 준영속 상태의 엔티티를 영속 상태로 변경할 때 사용하는 기능
        - 준영속 엔티티 : 영속성 컨텍스트가 더는 관리하지 않는 엔티티.
            - 위에서 진행한 로직을 보면, Book Entity를 새로 생성하고 Id 등 기존 정보들의 수정본을 심어줬음. 이러한 객체가 바로 준영속 Entity.
            - em으로 가져온 것이 아니기에 영속 Entity가 아니지만, id는 부여되어 id를 통해 em이 DB에서 영속 Entity를 찾아올 수 있기에 준영속 엔티티라고 불림.
        - 병합시 동작 방식을 간단히 정리

          ![]({{site.baseurl}}/images/posts/post-220316/Untitled 9.png)

            1. 준영속 엔티티의 식별자 값으로 영속 엔티티를 조회
            2. 영속 엔티티의 값을 준영속 엔티티의 값으로 **모두** 교체 (병합)
            3. 트랜잭션 커밋 시점에 변경 감지 기능이 동작해서 데이터베이스에 UPDATE SQL이 실행
    - 변경감지
        - 영속성 컨텍스트에서 엔티티를 다시 조회한 후에 데이터를 수정하는 방법 (즉, em을 통해 Entity를 가져와 영속성 컨텍스트에 올리고, 그 상태에서 set을 통해 데이터를 수정)
        - 트랜잭션 안에서 엔티티를 다시 조회 후 변경 → 트랜잭션 커밋 시점에 **변경 감지(Dirty Checking)**이 동작해서 데이터베이스에 UPDATE SQL 실행 ⇒ 즉, 따로 persist를 하지 않아도 됨. (이미 영속성 컨텍스트 안에 있기에)
        - `itemService.updateItem(itemId, form.getName(), form.getPrice(), form.getStockQuantity());`

        ```java
        @Transactional // merge는 사실 이 변경감지 코드와 동일하게 동작함. 단지 merge하나만 적어주면 그 과정을 알아서 진행 (but, 모두 변경)
        public void updateItem(Long itemId, String name, int price, int quantity){ // merge가 아닌 변경감지로 저장하는 것. -> 영속성인 객체를 가져와서 context에 올리는 것
            Item findItem = itemRepository.findOne(itemId);
            findItem.setName(name); // setter 없이 Entity안에서 직접 수정이 일어나도록 메서드를 짜는 게 더 좋은 코딩
            findItem.setPrice(price);
            findItem.setStockQuantity(quantity);
            // 영속성 객체이기에 따로 em을 이용해 persist할 필요가 없음. -> 변경감지를 통해 자동으로 update
        } // 해당 Transactional이 끝나고 commit 시점에 자동으로 변경사항에 대한 query를 날려주는 것.
        ```
    <aside>
    
  💡 <strong> [참고] </strong> <br> 실무에서는 보통 업데이트 기능이 매우 재한적이다. 그런데 병합은 모든 필드를 변경해버리고, 데이터가 없으면 null 로 업데이트 해버린다. 병합을 사용하면서 이 문제를 해결하려면, 변경 폼 화면에서 모든 데이터를 항상 유지해야 한다. 실무에서는 보통 변경가능한 데이터만 노출하기 때문에, 병합을 사용하는 것이 오히려 번거롭다. 즉, <strong> 엔티티를 변경할 때는 항상 변경 감지를 사용해야 됨.</strong>

    </aside>

    <aside>
    
  💡 <strong> [참고] </strong> <br>
    <ul>
    <li> 컨트롤러에서 어설프게 엔티티를 생성하지 마라.</li>
    <li> 트랜잭션이 있는 서비스 계층에 식별자( id )와 변경할 데이터를 명확하게 전달하라.(파라미터 or 파라미터가 많으면 dto를 통해)</li>
    <li> <strong>트랜잭션</strong>이 있는 서비스 계층에서 영속 상태의 <strong> 엔티티를 조회</strong>하고, <strong>엔티티의 데이터를 직접 변경</strong>하라.</li>
    <li> 트랜잭션 커밋 시점에 <strong>변경 감지(dirty checking)</strong>가 실행.</li>
    </ul>
    </aside>


- 주문 (상품 주문)
    - Order Entity는 앞선 도메인 개발에서 알 수 있듯이 member와 item등 여러 연관관계를 가지고 있기에 member,item service의 DI를 받아야 됨.
    - 주문 등록
        - (Controller) 주문 페이지 → Get → `String createForm(Model model)`
            - `@GetMapping("/order")`
            - 멤버 선택 및 상품 선택을 위해 모든 멤버와 상품을 불러들임. → `List<Member> members = memberService.findMembers();` , `List<Item> items = itemService.findAll();`
            - `model.addAttribute("members", members);`, `model.addAttribute("items", items);`
            - `return "order/orderForm";`
        - (Controller) 주문 등록 → Post →

            ```java
            @PostMapping("/order")
            public String order(@RequestParam("memberId") Long memberId,
            	                  @RequestParam("itemId") Long itemId,
            	                  @RequestParam("count") int count){
            	  orderService.order(memberId, itemId, count);
            	  return "redirect:/orders";
            }
            ```

            - 주문 등록 시 html을 통해 각 멤버와 상품의 id를 받아 와줌 (+ 주문 수량)
            - 이를 바탕으로 Order Entity를 생성 (생성메서드 + 연관관계 메서드를 통해 Entity를 생성)
            - 이때 중요한 것은 Entity 자체를 method에 보내주는 것이 아닌 식별자를 보내 줌으로 써 Service 내에서 Entity를 다룰 수 있도록 함. (즉, Transactional 안에서 Entity를 다루게 하는 것) → Entity는 Controller 보다는 Transactional이 있는 Service에서 다루는 것이 Best!!
    - 주문 조회 및 취소
        - 주문 조회 → Get → `String orderList(OrderSearch orderSearch, Model model)`
            - `OrderSearch` 는 MemberForm과 같이 web상에서 입력되는 값을 객체로 받아드려오는 것.
            - 그 OrderSearch의 값들을 통해 동적쿼리를 통해 해당 조건에 맞는 Order들을 불러옴 → `List<Order> orders = orderService.findOrders(orderSearch);` (findOrders는 `findAllByCriteria`를 통해 동적쿼리 실행.)
        - 주문 취소 → Post → `String cancelOrder(Long orderId)`
            - 주문 List에서 취소 버튼을 누르면 작동
            - `@PostMapping("/orders/{orderId}/cancel")` → 얘 또한 식별자 값을 받아와 Service에서 취소 진행. (Transactional 안에서 값 변경 → 변경 감지 가능!) → `orderService.cancelOrder(orderId);`