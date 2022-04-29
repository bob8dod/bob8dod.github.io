---
layout: post
title:  "Spring Boot + JPA Basic [part 2]"
date:   2022-03-12
image:  /posts/jpa2.jpeg
tags:   SpringBoot JPA
---

## Chapter 3 [어플리케이션 구현 준비]

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


## Chapter 4 [회원 도메인 개발]

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

## Chapter 5 [상품 도메인 개발]

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

