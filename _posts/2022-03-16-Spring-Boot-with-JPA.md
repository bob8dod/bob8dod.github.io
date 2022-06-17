---
layout: post
title:  "JPA를 이용한 웹계층 개발"
date:   2022-03-16
image:  /posts/JPA+WEB.jpg
tags:   SpringBoot JPA
---


## 웹 계층 개발

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