---
layout: post
title:  "Entity Association"
date:   2022-08-09
image:  /posts/jpa.png
tags:   SpringBoot JPA
---

## 연관관계는 왜 필요한가?

- 객체를 테이블에 맞추어 모델링(외래키를 field로 설정 → ex) teamId) 하면 자유자재로 객체 그래프를 이용할 수 없음! → 객체 지향적 설계 불가능
- 즉, 협력 관계를 만들 수 없음.
- **테이블과 객체 사이**에는 이런 **큰 간격이 존재**
- 이 간격을 **JPA는 연관관계로 해결**하고자 하는 것!

<aside>
⚠️ <strong>연관관계 주요 목표</strong>
<ul>
<li> 객체와 테이블 <b>연관관계의 차이를 이해</b> </li>
<li> 객체의 <b>참조(Reference)</b> ↔ 테이블의 <b>외래키(Foriegn Key)</b> <b>매핑</b> </li>
</ul>
</aside>

<aside>
⚠️ <strong>연관관계 속성</strong>
<ul>
<li> <b>방향</b> : 단방향, 양방향(<code>mappedBy</code>) </li>
<li> <b>다중성</b> : 다대일(<code>@ManyToOne</code>), 일대다(<code>@OneToMany</code>), 일대일(<code>@OneToOne</code>), 다대다(<code>@ManyToMany</code>)</li>
<li> <b>연관관계의 주인</b> : 객체 <b>양방향</b> 연관관계에서의 <b>관계 관리자</b></li>
</ul>
</aside>

## 연관관계 (단방향, 양방향)

### 단방향 연관관계

- **객체 지향 모델링**
    - 객체를 테이블에 맞추어 모델링하는 것이 아닌, 테이블에 초점되지 않고 **객체가 협력관계를 이용할 수 있도록 모델링**하는 것 → **객체 연관관계** 사용
    - 즉, **외래키값을 그대로 갖는 것이 아닌**, 해당 연관관계를 가지고 있는 **객체 자체를 참조(Reference)**로 가질 수 있도록 설정하는 것
    - ex) `private Long teamId;` → `private Team team;` : 객체 그래프 탐색이 가능해짐
- **단방향 연관관계**
    - **Entity 의 입장**.  Table의 입장이 아님
    - 두 객체의 연관관계가 있을 때, **객체의 이동이 한 객체에서밖에 하지 못하는 경우**
    - 즉, **참조(Reference)가 한곳에만** 있는 것.
    - **[주의] Table에서는 단방향은 존재하지 않음. →  Join으로 양방향으로 이동 가능!**
- 연관관계 예시 (단방향)

  ![Untitled]({{site.baseurl}}/images/posts/post-220809/Untitled.png)
    
    - 참조가 Member에만 있음 → **단방향**
    - Member Class 참조 부분
        
        ```java
        // Class Member
        @ManyToOne 
        @JoinColumn(name = "TEAM_ID") 
        private Team team;
        ```
        
    - `@ManyToOne`
        - 다대일(N:1)
        - **참조된 객체(Reference)와 어떤 관계인지**를 알려주는 것
        - 즉, **DB 입장**에서 현재 Class의 **Entity와 Referecne된 Entity가 어떤 연관관계인지**를 알려주는 것 (DB에는 연관관계라는 개념이 없으므로!)
        - Member 입장 : Many | Team 입장 : One  ⇒ ManyToOne
    - `@JoinColumn(name = "TEAM_ID")`

      ![Untitled]({{site.baseurl}}/images/posts/post-220809/Untitled 1.png)
        
        - **Member Entity 의 team(Reference)과 Member Table 의 team_id(foreign key)를 매핑한다**는 것
        - **연관관계의 주인**이 갖는 어노태이션 (↔ `mappedBy`)
    - 즉, **두 객체의 관계**가 무엇인지(일대다, 다대일, ...), 그리고 이 객체를 **Table 의 입장에선 어떤 Column 과 매핑**할 것인지를 설정만 해주면 이를 통해 연관관계를 설정하고 결과적으로 **객체 지향적인 설계가 가능**함!
    - 결국 참조를 사용하여 연관된 객체를 찾을 수 있음 → **협력관계 설정 가능**
    - 객체 지향적인 설계(사용)를 위해 **DB Table 입장에서의 상황을 알려주는 것**
    - 이렇게 연관관계를 설정하면
        
        ```java
        Member member = em.find(Member.class, 1L);
        Team team = em.find(Team.class, member.getTeamId());
        ```
        
        이렇게 할 필요 없이 바로 **참조(협력관계)를 이용**하여
        
        ```java
        Member member = em.find(Member.class, 1L);
        Team team = member.getTeam();
        ```
        
        로 해당 **연관관계의 객체를 접근**할 수 있음! → **객체 지향적인 설계**
        

### 양방향 연관관계와 연관관계의 주인

- **양방향 연관관계**
    - **Entity 의 입장**.  Table의 입장이 아님
    - 두 객체의 연관관계가 있을 때, 객체의 이동이 **두 객체 모두에서 할 수 있는** 경우
    - 즉, **참조(Reference)가 양쪽**에 있는 것. (양쪽으로 참조해서 갈 수 있는 것)
    - 사실 **양방향은 단방향이 두개로 되어 있는 것**을 말하는 것
    - **[주의] Table에서는 항상 양방향이 가능!** 즉, Table은 **단뱡향이든 양방향이든 Table은 전혀 변화가 없음** (Join이 존재하기 때문! → 방향이란 개념이 없음)
- 양방향 연관관계 예시

  ![Untitled]({{site.baseurl}}/images/posts/post-220809/Untitled 2.png)
    
    - 참조가 Member, Team 둘 다에 있음 → **양방향**
    - **중요한 건 단방향이든 양방향이든 Table에 변화는 없음!** → Table에는 방향이라는 것이 없음 (즉, 당방향이든 양방향이든 모든 가능!)
    - Team Class 참조 부분 (Member Class 는 단방향에서와 동일)
        
        ```java
        @OneToMany(mappedBy = "team")
        private List<Member> members = new ArrayList<>();
        ```
        
        - `@OneToMany(mappedBy = "team")`
            - **해당 객체와 어떤 관계**인지를 알려주는 것 (DB입장에서 어떤 관계인지 알려주는 것)
            - Team 입장 : One | Member 입장 : Many ⇒ OneToMany
            - `mappedBy = "team"` : **MEMBER 객체의 어떤 Field 와 연관**이 있는가 (DB 의 입장은 아님. DB 는 단방향이든 양방향이든 연결 가능! 객체 지향적인 설계를 위한 설정임!)
    - 이렇게 되면 Team에서도 연관된 Member들을 List로 확인이 가능함 → 객체지향적인 설계(객체그래프탐색 가능)
    - 하지만, **양방향이 항상 좋은 것은 아님**. 객체는 단방향이어야 객체그래프 자체가 순환되지 않고 좋음 → **필수적인 상황(역추적)에서만 사용하는 것**이 좋음
- **연관관계의 주인**
    - 객체와 테이블간에 연관관계를 맺는 차이를 이해해야 됨
    - 차이
        - **객체의 연관관계 = 2개 (양방향)**
            - 사실 양방향 관계가 아니라 서로 다른 방향의 단뱡향 관계 2개
            - A → B 1개 (단방향)
            - B → A 1개 (단방향)
        - **테이블 연관관계 = 1개 (양방향)**
            - 테이블은 **외래 키 하나로** 두 테이블의 연관관계를 관리
            - A ↔ B 1개 (양방향)
    - `mappedBy`
        - 객체의 양방향 관계는 사실 양방향 관계가 아니라 서로 다른 방향의 단뱡향 관계 2개!
        - 즉, 객체를 양방향으로 참조하려면 단방향 연관관계를 2개 만들어 줘야 함!
        - 반면에, **테이블은 외래 키 하나로 두 테이블의 연관관계를 관리하고 있음** (외래 키 하나로 A→B 로, B→A로 갈 수 있음)
        - 즉, 외래 키 하나로 양방향 연관관계를 가짐! (양 쪽으로 JOIN 가능!)
        - 여기서 딜레마 발생! → **A에서 B를 바꿨을 때 Table 에 반영(외래키 변경)을 해야 되는지, B에서 A가 변경되었을 때 Table 에 반영(외래키 변경)해야 될 지 딜레마**에 빠지게 됨 (객체로 보면 **A,B 둘 다에 참조값이 있지만 Table 에는 A에만 FK** 가 있으니!)

          ![Untitled]({{site.baseurl}}/images/posts/post-220809/Untitled 3.png)
            
            예를 들어 Member에서 Team 을 바꿨을 때 Member의 변경된 Team으로 Member Table의 외래 키를 바꿔야 할지, Team에서 members의 Member가 변경되었을 때 변경된 Team으로 Member Table의 외래 키를 변경해야될지 딜레마에 빠지게 되는것.
            
        - **딜레마 해결! → 둘 중에 하나로 외래 키를 관리**하자! ⇒ "**연관관계의 주인**" → **외래 키가 있는 놈으로 주인을 설정**하자! (다대일 관계에서 "**다**"에 해당하는 객체!)
        - 즉, `mappedBy` 는 **관계 상의 ‘을’을 지칭**하는 것 (변경 권한이 없는 것!)
        - **[참고] 관계 상의 ‘을’은 비지니스 로직에서의 중요도를 말하는 것이 아닌 오직 관계에 있어서의 권한**을 이야기하는 것
    - **연관관계 주인 설정**
        - 객체의 두 관계 중 하나를 연관관계의 주인으로 지정 (Table FK는 한 Table에만 존재하므로)
        - **연관관계의 주인만이 외래 키를 관리** (등록, 수정)
        - 주인이 아닌쪽(`mappedBy`)은 **읽기만 가능**
        - 주인은 mappedBy 속성 사용 X
        - 주인이 아니면 mappedBy 속성으로 주인이 아닌 것을 표현
        - **그럼 누가 연관관계의 주인??**
            - **외래 키(FK)**가 있는 곳 (정해진 것은 없지만 권장하는 것)
            - Table 의 입장에서 외래키가 있는 곳은 다대일(N:1)관계에서 “다”
            - 즉, “다” 쪽이 연관관계의 주인으로 설정하는 것이 좋음
            - 만약 외래 키가 있지 않은 **반대 편의 Table을 연관관계의 주인공으로 설정**한다면, 수정 시 **다른 Table에 update query가 나가는 일이 발생!!** → 너무 복잡해짐
            ex) A에 외래키가 있는데, B가 연관관계의 주인이 된다면, B의 A값이 변경된다고 할 때 B Table에 update query가 나가는 것이 아닌 A Table에 UPDATE query가 나감. 즉, B를 수정했는데 B Table이 아닌 A Table이 수정되는 것!! → 복잡해~ 헷갈려~
        - **[주의] 연관관계의 주인이라고 비지니스적으로 우위에 있는 것이 아닌 그냥 관계상의 주인**
- 연관관계의 주인에서 **주의해야할 것들**
    - 연관관계의 주인에 값을 입력하지 않는 경우
        - 문제
            
            ```java
            Member member = new Member();
            member.setName("park");
            // member.setTeam(team) 을 까먹은 것
            
            team.getMembers().add(member); // 역방향(주인X)만 연관관계 설정
            ```
            
        - 해결
            - **연관관계 편의 매서드**를 따로 지정하자!
                
                ```java
                public void changeTeam(Team team) { // changeTeam
                    this.team = team;
                    team.getMembers().add(this);
                }
                ```
                
                - 연관관계의 주인을 기준으로 **이와 연관된 객체까지 놓치지 않도록 메서드**를 만들어주는 것
                - member의 team을 변경하거나 설정할 때, **항상 역방향의 객체에도 해당 사항을 적용**해주는 것!
    - 양방향 매핑시의 **무한루프**
        - 문제
            - `toString()`, **Lombok** , **JSON 생성 라이브러리** 사용 시 **해당 되는 모든 값들을 가져와 적용**하기 때문에 양방향 매핑을 해두었다면 계속해서 순환되는 **무한루프**에 빠지게 됨
            - ex) Member.getTeam() → Team.getMembers() → 해당 결과(Member List)의 각 Member.getTeam() → 해당 결과 객체(Team)의 Team.getMembers() → …
        - 해결
            - `toString()` 이나 **Lombok** 사용 시 **해당 양방향을 반환하지 않도록 따로 설정** 필요!
            - **JSON 생성 라이브러리** 같은 경우 Conroller의 return 반환 값으로 자주 사용 되는데, 그 때 Entity를 직접 반환하지 말고 양방향관계의 객체를 제외한 DTO를 사용하여 반**환**해라!

<aside>
⚠️ <strong>양방향 매핑 때 고려해야될 점 (TIP)</strong>
<ul>
<li> 단방향 매핑만으로도 이미 연관관계 매핑은 완료. 즉, <b>단방향 매핑만으로도 모든 Entity의 설계를 완료</b>할 수 있고, 처음 설계 작업에서는 그렇게 해야만 함. </li>
<li> <b>양방향 매핑</b>은 <b>그저 반대 방향으로 조회(객체 그래프 탐색) 기능만 추가된 것</b> 뿐!! → <b>Table은 동일</b>함! Application단에서만 적용되는 것 </li>
<li> 즉, 일단 <b>단방향 매핑을 잘해두고</b> 필수적으로 <b>역방향 탐색이 필요한 경우에만 양방향 매핑을 걸어</b>주는 것이 좋음 (JPQL에서 역방향 탐색이 필요한 경우가 종종 있음) </li>
</ul>
</aside>