---
layout: post
title:  "Querydsl - Advanced"
date:   2022-09-22
image:  /posts/querydsl.png
tags:   SpringBoot JPA Querydsl
---

## 프로젝션, 결과 반환

### 기본

- **하나**의 프로젝션 대상
    
    ```java
    List<String> result = queryFactory
              .select(member.username)
              .from(member)
              .fetch();
    ```
    
    - 결과로 가져올 값은 `member.username` 하나 (String)
    - 이렇게 프로젝션 대상이 하나면 **타입을 명확하게 지정** 가능 (가져오는 Type에 따른 결과 Type)
- **둘 이상**의 프로젝션 대상 → Tuple
    
    ```java
    List<Tuple> result2 = queryFactory
              .select(member.username, member.age)
              .from(member)
              .fetch();
    ```
    
    - 결과로 가져올 값이 `member.username`과 `member.age` 두 가지
    - 이들은 ~~Q Class~~로 지정되어 있지도 않고 ~~하나의 객체~~로 묶을 수 없음 (DTO로 지정하지 않는 한)→ **튜플로 반환**
        - **튜플**은 **Querydsl** 에서 제공되는 것이기 때문에, **한 계층(repository)에서만 사용**하는 것이 좋음. 만약 **두 계층(service, controller 단)까지** querydsl의 튜플이 이어지면 **Querydsl에 의존성이 짙어지고 계층 간 순수성이 떨어짐**
    - 혹은 해당 **field들을 포함한 DTO를 사용**하여 반환받을 수 있음

### DTO 조회

- **순수 JPA**에서의 DTO 직접 조회
    
    ```java
    List<MemberDto> result = em.createQuery(
    			 "select new study.querydsl.dto.MemberDto(m.username, m.age) " +
    			 "from Member m", MemberDto.class)
    				 .getResultList();
    ```
    
    - **new 오퍼레이션** 사용
    - **모든 패키지 명을 다 적어줘야한다**는 불편함 존재
    - **생성자 방식만** 지원 → 해당 생성자에 맞는 Type을 넣어주어야 함
- **Querydsl** 에서의 DTO 직접 조회
    - **Projections** 사용 → `com.querydsl.core.types.**Projections**`
    - 결과를 직접 DTO로 반환할 시 **3가지 방법** 지원
        - **property** 접근 (**Setter**를 통한 값 설정) : `Projections.bean()`
        - **field** 직접 접근 (**this.field**를 통한 값 설정) : `Projections.fields()`
        - **Constructor** 사용 (**new 오퍼레이션**을 통한 값 설정) : `Projections.constructor()`
        - `@QueryProjection` 을 통해 **DTO을 Q-Type으로** 만들어서 **new 오퍼레이션** 사용
    - **주의점**
        - **프로퍼티, 필드 접근**은 항상 **넣고자하는 값의 필드명과 DTO의 필드명은 동일**해야 됨! → 넣고자하는 **`QClass : QMember = {username, age}`** 이면 DTO 또한 **해당 필드명과 동일한 필드명**을 가져야함 ⇒ **`MemberDto = {username, age, … }`**
        - **생성자**를 통한 값 설정은 **Type만 주의**하면 됨. (기본적으로 생성자를 통해 값을 설정하는 과정을 생각하면 됨)
    1. **프로퍼티 접근**
        
        ```java
        List<MemberDto> memberDtos = queryFactory
                  .select(Projections.bean(
                          MemberDto.class,
                          member.username, member.age
                  ))
                  .from(member)
                  .fetch();
        ```
        
        - `Projections.bean()` 사용 → **Setter**를 이용해서 값을 넣어줌
        - `Projections.bean(MemberDto.class,member.username, member.age)`
            - MemberDto.class 에 username, age에 대한 **Setter를 통해 값설정** → **기본 생성자**를 통한 **인스턴스 생성 후 Setter를 통해 값 설정** (그러므로 DTO에 **기본 생성자가 필수**적으로 존재해야 함)
            - 동작 과정
                
                ```java
                memberDto = new MemberDto();
                memberDto.setUsername(member.username);
                memberDto.setAge(member.age);
                ```
                
        - **넣고자하는 값의 필드명**과 **DTO의 필드명은 동일**해야됨 → **필드명을 통해 동작**하기 때문
            
            <aside>
            💡 <strong>만약 넣고자 하는 값의 이름과 DTO의 field 명이 다르다면?</strong>
            <ul>
            <li> 넣고자 하는 값에 <b>as 를 통해 별칭을 부여</b>하여 <b>일치시켜</b>야 함 </li>
            <li> UserDto = {name, age} | Member = {username, age} 라고 할 때 </li>
            <li> → <code>Projections.bean(UserDto.class, member.username.as("name"))</code> </li>
            </ul>
            </aside>
            
    2. **필드 직접 접근**
        
        ```java
        List<MemberDto> memberDtos = queryFactory
                  .select(Projections.fields(
                          MemberDto.class,
                          member.username, member.age
                  ))
                  .from(member)
                  .fetch();
        ```
        
        - `Projections.fields()` 사용 → **field에다 값을 바로 주입** (setter X)
        - `Projections.fields(MemberDto.class, member.username, member.age)`
            - MemberDto.class 에 username, age에 대한 **field에 값을 직접 넣어 설정** → private field 이지만, Querydsl에서 해당 부분에 대해서만 가능하게 설정되어 가능한 것
            - 동작 과정
                
                ```java
                memberDto = new MemberDto();
                memberDto.username = member.username;
                memberDto.age = member.age;
                ```
                
        - **넣고자하는 값의 필드명**과 **DTO의 필드명은 동일**해야됨 → **필드명을 통해 동작**하기 때문
    3. **생성자 사용**
        
        ```java
        List<MemberDto> memberDtos = queryFactory
                  .select(Projections.constructor(
                          MemberDto.class,
                          member.username, member.age
                  ))
                  .from(member)
                  .fetch();
        ```
        
        - `Projections.constructor()` 사용 → 생성자를 통해 값 설정
        - `Projections.constructor(MemberDto.class, member.username, member.age)`
            - MemberDto.class 에 **username, age 모두를 인자로 받는 생성자**를 통해 값 설정 → **해당 인자들에 대한 생성자는 필수**
            - 동작 과정
                
                ```java
                memberDto = new MemberDto(member.username, member.age);
                ```
                
        - 이름을 일치하지 않아도 됨. → **생성자의 인자 Type, 순서만을 신경써주면 됨**
    4. **Dto를 Q-Type으로 만들어서 사용** → `@QueryProjection`
        - `@QueryProjection` 을 통해 **DTO를 Q-Type으로** 만들어 **직접 new 오퍼레이션**을 통해 생성자로 DTO 조회하는 방법 (**Q-Type 생성자 사용**)
        - DTO 생성자에 `@QueryProjection` 설정
            
            ```java
            @QueryProjection
            public MemberDto(String username, int age) {
            		this.username = username;
            		this.age = age;
            }
            ```
            
        - 그 후 `./gradlew compileQuerydsl`을 통해 **Q-Type 생성**
        - **QMemberDto 생성 확인** 필수
        - **new 오퍼레이션** 사용
            
            ```java
            List<MemberDto> result = queryFactory
                      .select(new QMemberDto(member.username, member.age))
                      .from(member)
                      .fetch();
            ```
            
        - 장점
            - 다른 도구를 사용하는 것이 아닌 **바로 new 오퍼레이션로 생성** 가능
            - **[중요]** `Projections.constructor()` 는 다르게 **Q-Type을 생성하고 해당 Q-Type의 생성자를 사용하기 때문에 컴파일 시점에 오류 검출 가능!** (컴파일러로 타입 체크)
        - 단점
            - **DTO가 Querydsl을 의존한채로 여러 단(controller, service, repository)에서 사용**된다는 것이 가장 큰 단점 → **여러 단에서 Querydsl을 의존**하는 형태가 되는 것
    
- [참고] Querydsl을 통해 DTO 직접 조회 + 서브쿼리
    - 만약 **서브쿼리를 사용하여 나온 결과를 select절에 넣어주며 DTO에 넣고 싶다**면? → ex) DTO의 age field에 member의 최대 나이를 넣어주고 싶다면?
    - `ExpressionUtils.as` 사용
        
        ```java
        .select(Projections.bean(
                UserDto.class,
                member.username.as("name"),
                ExpressionUtils.as(
                        JPAExpressions
                                .select(sub.age.max())
                                .from(sub),
                        "age" // 해당 결과 값을 age로 표현
                )
        )).from(member)
        ```
        
        - `ExpressionUtils.as` **를 통해 서브쿼리의 결과를 **원하는 별칭으로 설정하여 DTO에 넣어주면 됨**
        - **프로퍼티, 필드 접근** 에서 사용 (생성자 사용은 Type만 주의하면 됨)

## 동적 쿼리

**동적 쿼리** 해결 방식

1. **BooleanBuilder**
2. **Where 다중 파라미터** 사용

### BooleanBuider

```java
BooleanBuilder builder = new BooleanBuilder(); // 초기 조건도 넣어줄 수 있음
if (usernameCond != null) {
    builder.and(member.username.eq(usernameCond));
}
if (ageCond != null) {
    builder.and(member.age.eq(ageCond));
}
return queryFactory
        .selectFrom(member)
        .where(builder) // 추가적으로 and, or 추가 가능 -> builder.and().or()
        .fetch();
```

- `com.querydsl.core.**BooleanBuilder`** 를 통해 **동적 쿼리 작성**
- 원하는 조건을 계속 **BooleanBuilder에 추가(`.and()`, `.or()`)** 해주어 **조건을 동적으로 추가**해주는 것
- `BooleanBuilder builder = new BooleanBuilder();`
    - **빌더** 생성
    - **초기 조건(값)을 넣어 줄 수 있음** → `new BooleanBuilder(member.username.eq(usernameCond))` (usernameCond이 null이 아니라는 가정하에)
- 이제 해당 Cond에 따라 **null이 아니면 계속해서 조건을 추가**해주는 것
    - `builder.and(member.username.eq(usernameCond));`
    - `builder.and(member.age.eq(ageCond));`
    - `.and()` 뿐만 아니라 `.or()` 도 가능 → 즉, **모든 조건을 동적으로 추가 가능**
- **주의점** : builder에 아무것도 추가 되지 않으면 **아무 조건도 붙지 않는 것!** → 조건없는 **전체 조회**

### Where 다중 파라미터 사용 (BooleanExpression)

```java
List<Member> members =  queryFactory
          .selectFrom(member)
          .where(usernameEq(usernameCond), ageEq(ageCond)) // null이 들어갈 경우 무시됨!
			 // .where(allEq(usernameCond, ageCond)) // 조합해서 적용 가능
          .fetch();
```

- `com.querydsl.core.types.dsl.**BooleanExpression`** 를 통해 **동적 쿼리** 작성
- 원하는 조건을 **메서드로 뽑아**내어서 **이를 where절 안에서 사용**하는 것
    - 메서드로 뽑아서 **적용하고 싶다면 해당** **조건식 (`~.eq()`, `~.goe()` 등)을 반환**해주고 조건에 맞지않아 **적용하지 않으려면 null 을 반환**하여 적용되지 않게 함 (where(~.eq(), null) | where(~.eq().or(null)) 에서 **null 부분은 무시**됨!!)
    - `usernameEq(usernameCond)`
        
        ```java
        private BooleanExpression usernameEq(String usernameCond) {
            return usernameCond != null ? member.username.eq(usernameCond) : null;
        }
        ```
        
    - `ageEq(ageCond)`
        
        ```java
        private BooleanExpression ageEq(Integer ageCond) {
            return ageCond != null ? member.age.eq(ageCond) : null;
        }
        ```
        
- 메서드로 뽑았기에 **다른 쿼리에서 재사용이 가능**하며, 메서드명을 통해 **쿼리 자체의 가독성**을 높임 (메서드의 목적을 분명히 알 수 있음)
- 추가로 이 메서드들을 조합해서 **또 다른 메서드를 생성하여 적용 가능**
    - `allEq(usernameCond, ageCond)`
        
        ```java
        private BooleanExpression ageEq(Integer ageCond) {
        	  return ageCond != null ? member.age.eq(ageCond) : null;
        }
        ```
        
        - 메서드를 **조합**할 때 **null 체크는 특히 유의**해야 됨

## 벌크 연산

### 수정

```java
long resultCnt = queryFactory
          .update(member)
          .set(member.username, "잼민이")
          .where(member.age.lt(18))
          .execute(); // bulk 연산

em.clear();
```

- **쿼리 한번으로 대량 데이터를 수정**하는 방법
- `execute()` 활용 (순수 JPA에서의 `executeUpdate()`와 같은 역할)
- 해당 쿼리로 인해 **변경된 데이터의 개수를 반환**
- **주의점(`em.clear()`)** : 벌크 연산은 **영속성 컨텍스트를 거쳐서 진행되지 않**기 때문에 항상 벌크 연산 후 **영속성 컨텍스트를 초기화(`em.clear()`)**하여 조회 시 **변경된 점을 반영**할 수 있도록 설정해야 됨

### 삭제

```java
queryFactory
    .delete(member)
    .where(member.age.gt(18))
    .execute(); // bulk 연산
```

- **쿼리 한번으로 대량 데이터를 삭제**
- `execute()` 활용 (**벌크연산은 모두 `execute()` 사용**)

## SQL function 호출하기

- SQL function은 JPA와 같이 **Dialect에 등록된 내용만 호출 가능**
- 사용법 : `Expressions.stringTemplate(”function(’함수명’, {0}, {1}, … ), 매핑할 파라미터들”)`
- ex 1) 회원명의 member 부분을 m 으로 변경하는 **SQL function(replace 함수)** 사용
    
    ```java
    List<String> result = queryFactory
            .select(Expressions.stringTemplate
                        ("function('replace', {0}, {1}, {2})",
                                member.username, "member", "M"))
            .from(member)
            .fetch();
    ```
    
    - `Expressions.stringTemplate(사용 함수 설정, 파라미터 설정)` : **함수 지정 및 파라미터 설정 가능하게 하는 Expressions**
        - 사용함수 설정: `function('replace', {0}, {1}, {2})"` : Dialect의 **replace 함수 사용**
        - 파라미터 설정: `member.username, "member", "M"` : 각 {0},{1},{2} 에 들어가는 **파라미터**
- ex 2) 소문자로 변경해서 비교 (소문자인 애들만 반환) → **SQL function(lower 함수)** 적용
    
    ```java
    List<String> result2 = queryFactory
            .select(member.username)
            .from(member)
    		//  .where(member.username.eq(member.username.lower())) // 내장함수로도 있음
            .where(member.username.eq(Expressions.stringTemplate("function('lower', {0})", member.username)))
            .fetch();
    ```
    
    - `Expressions.stringTemplate(사용 함수 설정, 파라미터 설정)` : **함수 지정 및 파라미터 설정 가능하게 하는 Expressions**
        - 사용함수 설정: `"function('lower', {0})"` : Dialect의 **lower 함수 사용** (Querydsl에서의 lower() 내장함수로 대체 가능)
        - 파라미터 설정: `member.username` : {0} 에 들어가는 **파라미터**