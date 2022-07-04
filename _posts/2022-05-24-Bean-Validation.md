---
layout: post
title:  "Bean Validation"
date:   2022-05-24
image:  /posts/bean-validation.png
tags:   SpringBoot Validation
---

## Bean Validation 소개

- 지금까지 작성했던 **검증 로직** 관련 코드를 **하나의 어노테이션으로 끝낼 수 있는 기능**
- 대상 객체에 대해서 **검증 로직**을 모든 프로젝트에 적용할 수 있게 **공통화하고, 표준화** 한 것
- **Bean Validation** 이란?
→ 특정한 구현체가 아니라 Bean Validation 2.0(JSR-380)이라는 기술 표준. 즉, 검증 애노테이션과 여러 **인터페이스의 모음**. 이를 구현한 기술 중에서 일반적으로 사용하는 **구현체**는 **하이버네이트 Validator** (ORM이랑은 관련 없음!)
    - 공식 메뉴얼: [https://docs.jboss.org/hibernate/validator/6.2/reference/en-US/html_single/](https://docs.jboss.org/hibernate/validator/6.2/reference/en-US/html_single/)
    - 검증 애노테이션 모음: [https://docs.jboss.org/hibernate/validator/6.2/reference/en-US/html_single/#validator-defineconstraints-spec](https://docs.jboss.org/hibernate/validator/6.2/reference/en-US/html_single/#validator-defineconstraints-spec)
- Bean Validation 사용
    - **Spring 통합없이**, **순수 Bean Validation** 사용법
    - Bean Validation 사용을 위한 의존관계 추가 (라이브러리 추가)
    → `implementation 'org.springframework.boot:spring-boot-starter-validation'`
    - Item 객체 (Bean Validation 적용 객체)
        
        ```java
         ...
         @NotBlank // 빈 문자나 null이면 안됨
         private String itemName;
         
         @NotNull // null이면 안됨
         @Range(min = 1000, max = 1000000) // 1000~1000000 범위
         private Integer price;
         
         @NotNull // null이면 안됨
         @Max(9999) // 최댓값이 9999보다 크면 안됨
         private Integer quantity;
         ...
        ```
        
        - 검증 **어노테이션**으로 **Validation 편리하게 적용** 가능
        - `@NotBlank` : 빈 문자나 null이면 안됨
        - `@NotNull` : null이면 안됨
        - `@Range(min = 1000, max = 1000000)` : 범위 안의 값이어야 됨
        - `@Max(9999)` : 최대 범위 설정
        
        <aside>
        ⚠️ <strong>검증 어노테이션 참고</strong> <br>
        <code>javax.validation</code> 으로 시작하면 정해진 구현에 관계없이 <b>모든 구현체에서 사용가능한 표준 인터페이스</b> (ex_NotBlank, NotNull, Max) <br>
        <code>org.hibernate.validator</code> 로 시작하면 <b>하이버네이트 validator 구현체</b>를 사용할 때<b>만 제공</b>하는 검증 (ex_Range)
        
        </aside>
        
        <aside>
        💡 <strong>Bean Validation 오류 메시지</strong> <br>
        따로 설정하지 않으면 Bean Validation 자체에서 제공하는 오류 메시지 노출. 만약 <b>default message를 수정</b>하고 싶으면 <b>어노테이션에 추가</b>해주면 됨! → <code>@NotBlank(message=”빈 값입니다!”)</code>
        더 <b>자세한 메시지</b>는 이전 <b>오류 코드 설정</b>(errors.properties <b>메시지 파일을 이용한 오류 메시지 지정</b>)을 통해 진행하면 됨
        
        </aside>
        
    - 검증 Test
        - 검증기 생성
            
            ```java
            ValidatorFactory factory = Validation.buildDefaultValidatorFactory();
            Validator validator = factory.getValidator();
            ```
            
        - 검증 실행
            
            ```java
            Set<ConstraintViolation<Item>> violations = validator.validate(item);
            ```
            
            - item에 검증에 벗어나는 값을 설정해서 넣어주면, 그에 맞는 **검증 결과를 violations에 담아**두는 것.
            - violations를 통해 어떤 검증 실패가 발생했는지 확인 가능

## Bean Validation 적용 (Spring)

### Bean Validation with Spring

- Bean Validation의 Validator 등록을 위해 사용했던 `@InitBinder` 제거! → 검증이 중복으로 적용됨!! (Bean Validation의 Validator 랑 custom Validator(ItemValidator) 가 중복으로 실행 되는 것)
- 그리고 다른 코드는 수정할 필요 없음! → **Bean Validation이 자동으로 Global Validator로 등록** 되고 **이를 사용하고자 하는 객체에 `@Validated`** 를 달아주면 알아서 Validator가 적용됨! (`@Validated @ModelAttribute Item item`)
    
    ```java
    @PostMapping("/add")
    public String addItem(@Validated @ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes) {
    	 
    	 // 실패 시
    	 if (bindingResult.hasErrors()) {
    		 log.info("errors={}", bindingResult);
    		 return "validation/v3/addForm";
    	 }
    	 ... // 성공 로직
    }
    ```
    
- 어떻게 Bean Validator가 적용 되나?
    - 스프링 부트는 `spring-boot-starter-validation` 라이브러리가 있으면 **자동으로 Bean Validator를 인지하고 스프링에 통합**시킴 (즉, **Spring 내에서 사용**할 수 있게 해준다는 것)
    - 그리고, `LocalValidatorFactoryBean` 을 Global Validator (프로젝트 내의 모든 곳에서 사용할 수 있는 Validator)로 등록. ( `LocalValidatorFactoryBean` 는 `@NotNull`과 같은 애노테이션을 보고 검증을 수행. 추가로 Global Validator를 직접 등록하는 경우, 해당 Validator는 동작하지 않음 → Global은 직접 등록한 애들만 있다고 생각하기에)
    - 이렇게 **Bean Validation 기반 Global Validator가 자동으로 등록**되었기 때문에 `@Valid` , `@Validated` 만 적용해주면 해당 **Validator를 원하는 객체에 적용**할 수 있음!
    - 적용한 객체에서 검증 오류가 발생하면, 이전과 동일하게 `FieldError`, `ObjectError` 를 생성해서 `BindingResult`에 담아줌!
- **Bean Validator 검증 순서**
    1. `@ModelAttribute` 각각의 필드에 **Type 변환 (String to X)** 시도
        1. 성공하면 2번
        2. 실패하면 `typeMismatch`로 `FieldError` 추가
    2. **어노테이션 기반 Validator 적용** (Bean Validation)
    
    <aside>
    ⚠️ <strong>Binding에 성공한 필드만 Bean Validation 적용!!</strong> <br>
    즉, <b>바인딩에 실패한 필드(typeMismatch 등)는 Bean Validation을 적용하지 않음!</b> → 적용할 이유가 없음. 예를 들어 Range를 검증하는 Field에서 숫자가 아닌 문자 타입이 들어오면 Range는 사실상 의미가 없어짐. 그러므로 <b>바인딩에 실패한 필드는 검증 가치가 없다 판단</b>하여 따로 Bean Validation을 적용하지 않음. <br>
    <strong>⇒</strong> 즉, <code>@ModelAttribute</code> → 각각의 필드 타입 변환시도 →  변환에 성공한 필드만 BeanValidation 적용
    
    </aside>
    

### Bean Validation with Error Message(Code)

- **더 자세히, 계층적**으로 **Bean Validation의 오류 메시지 설정**하기
- Bean Validation으로 인해 발생된 bindingResult의 검증 오류 코드를 확인해보면 **오류 코드가 애노테이션 이름으로 등록**되는 것을 확인할 수 있음. → `codes=[NotBlank.item.itemName, NotBlank.itemName, NotBlank.java.lang.String, NotBlank]`
- 즉, Bean Validation도 이전과 동일하게 **errors.propertes 메시지 소스에 해당 어노테이션을 이름으로 하는 오류 코드를 지정**해주면 됨
- errors.properties
    
    ```yaml
    NotBlank={0} 공백X
    Range={0}, {2} ~ {1} 허용
    Max={0}, 최대 {1}
    ```
    
    - {0} : 필드 명. {1},{2}, … 는 각 애노테이션에서 설정한 값들

### Bean Validation in ObjectError

- 지금까지는 Bean Validation을 FieldError에 적용하는 것을 봤음
- 그럼 **ObjectError는 어떻게?**
    - `@ScriptAssert()` 이용
    - 기존에 사용했던, 직접 `ObjectError`를 생성해서 만들기
- `@ScriptAssert()`
    
    ```java
    @Data
    @ScriptAssert(lang = "javascript", script = "_this.price * _this.quantity >= 10000", message = "10000원 이상")
    public class Item {
    	 //...
    }
    ```
    
    - item의 price, quantity의 값의 곱이 10000 이상인 검증 수행
    - message 설정 가능
    - But, script 언어로 사용하기도 하고 **사용 방법이 단순하지 않음**
    - 또한, **검증 과정 자체가 복잡할 경우는 사용하기 까다로움**
    - **보통 많이 사용하지는 않는 방법**
- **[주로 사용하는 방법]** 직접 검증 후 `ObjectError`를 생성해서 넣어주기 (Controller 단에서)
    
    ```java
    if (item.getPrice() != null && item.getQuantity() != null) {
    	 int resultPrice = item.getPrice() * item.getQuantity();
    	 if (resultPrice < 10000) {
    		 bindingResult.reject("totalPriceMin", new Object[]{10000, resultPrice}, null);
    	 }
    }
    ```
    
    - **직접 검증을 실행**하여 검증 조건에 맞지 않다면 **BindingResult의** `reject`**를 사용**하여 `ObjectError`를 생성하여 넣어줌
    - 코드가 길어지지만, 복잡한 검증도 가능하며 원하는 형태로 구성할 수 있음

<aside>
💡 <strong>Bean Validation (FieldError, ObjectError)</strong>
<ul>
<li>`FieldError`인 경우 Bean Validation의 Field에 다는 어노테이션(@NotNull, @Max, … )을 통해서 편리하게 진행</li>
<li>ObjectError인 경우 `@ScriptAssert` 보다는 Controller 단에서 직접 ObjectError를 추가해주는 방법을 많이 이용</li>
</ul>
</aside>

<aside>
🚨 <strong>Bean Validation의 한계</strong>
<ul>
<li>만약 Item 객체 등록 시의 검증 요구사항과 수정 시의 검증 <b>요구사항이 다르면?</b> → Bean Validation을 사용하기 어려움 (ex_등록 시에는 Max 검증이 있었는데, 수정 시에는 Max 검증을 없애고 싶은 상황. <b>각각에 맞는 어노테이션 변경이 필요함!!</b>)</li>
<li>즉, 각 <b>요구사항에 맞는 검증 설정</b>이 필요!!</li>
<li>이건 기존 Bean Validation으로는 어려움!</li>
</ul>
</aside>

### Bean Validation - 요구사항에 따른 검증

- 요구사항에 따른 검증 2가지 방법 (with Bean Validation)
    - BeanValidation의 **groups 기능** 사용
    - Item을 직접 From Data로 사용하는 것이 아닌, ItemSaveForm, ItemUpdateForm 같은 **DTO를 사용 (DTO : Data Transfer Object)**
- 실무에서는 groups 보다는 **DTO를 주로 사용**

### BeanValidation의 groups

- 등록시에 검증할 기능과 수정시에 **검증**할 기능을 **각각 그룹으로 나누어 적용**
- group은 **interface로 지정**만 해주면 됨
    - 등록용 groups interface 생성 → `public interface SaveCheck {}`
    - 수정용 groups interface 생성 → `public interface UpdateCheck {}`
    - 등록용은 SaveCheck.class로 지정, 수정용은 UpdateCheck.class로 group을 지정해주면 됨
- Item 객체에 **group 지정**하기
    - 등록만 적용 : `@Max` ->`@Max(value = 9999, groups = SaveCheck.class)`
    - 수정만 적용 : `@NotNull` → `@NotNull(groups = UpdateCheck.class)`
    - 둘 다 적용 : `@NotNull` → `@NotNull(groups = {SaveCheck.class, UpdateCheck.class})`
- Controller의 `@Validated`에 group 지정
    - `@Validated()` 안에 해당 group interface를 지정해주면 됨
    - 등록 method (`@PostMapping(”/add”)`) → `public String addItemV2(@Validated(SaveCheck.class) @ModelAttribute Item item ...`
    - 수정 method (`@PostMapping(”/{itemId}/edit”)`) → `public String editV2(@PathVariable Long itemId, @Validated(UpdateCheck.class) @ModelAttribute Item item, …`
- BUT, 실무에서는 groups 보다는 **DTO 형식을 이용!!!**

> 참고 : `@Valid` 에서는 groups 사용 못함. 즉, groups를 사용하려면 `@Validated` 를 사용해야 됨
> 

### Bean Validation with DTO (Form 전송 객체 분리)

- 실무에서 groups 를 잘 사용하지 않는 이유 → **폼**에서 전달하는 데이터가 **서비스 도메인 객체와 정확히 딱 드러맞지 않기 때문**
- 즉, 사용자가 입력한 데이터와 실제로 저장하려 하는 도메인 객체가 딱 맞지 않음! (추가적은 정보를 넣어야할 때도 있고, 부가적으로 작업 후에 새로운 값으로 넣어주는 경우도 있기 때문)
- 그래서 보통 도메인으로 사용되는 객체를 Form으로 보내 직접 받아 오는 것이 아니라, 폼의 데이터를 컨트롤러까지 전달할 **별도의 객체(DTO)를 만들어서 전달**함. 즉, Form을 전달받는 전용 객체를 만들어 `@ModelAttribute` 로 사용하는 것. 이후 해당 데이터를 받아 컨트롤러에서 필요한 데이터만을 사용하여 서비스 도메인을 생성하는 것!

<aside>
⚠️ <strong>폼데이터 전달에 도메인 객체 사용 VS DTO 사용</strong>
<ul>
<li><b>도메인 객체 사용</b></li>
    <ul>
    <li><code>HTML Form -> Item -> Controller -> Item -> Repository</code></li>
    <li>장점 : 도메인 자체를 사용하기에 추가적인 설정이 필요없이 간단함</li>
    <li>단점 : 도메인, 데이터 자체가 복잡해지는 경우 사용이 까다로움. 수정 시 검증이 중복될 가능성 높음 (groups 사용 필수)</li>
    </ul>
<li><b>DTO 사용</b></li>
    <ul>
    <li><code>HTML Form -> ItemSaveForm -> Controller -> Item 생성 -> Repository</code></li>
    <li>장점 : 폼 데이터가 복잡해도 그 스펙에 맞춘 폼 객체를 사용하여 데이터를 전달 받을 수 있음. <b>요구사항(용도)에 맞는 별도의 폼 객체</b>를 만들어 사용하기 때문에 검증 중복 X (groups 사용X)</li>
    <li>단점 : 컨트롤러에서 추가적인 작업(DTO → 도메인 객체 이전 작업)이 필요함</li>
    </ul>
</ul>
</aside>

- **DTO 사용**
    - 기존 Item 도메인 객체의 Bean Validation은 모두 제거 (Form DTO를 통해 검증을 마친 후 사용되므로!)
    - **등록용 DTO (ItemSaveForm)** → `public class ItemSaveForm`
        - 등록 스펙에 맞는 property들(id는 등록시 넘어오지 않으므로 id를 제외한 item의 모든 proerty)만을 설정
        - 등록 시 **요구사항에 맞는 Bean Validation 진행**
    - **수정용 DTO (ItemUpdateForm)** → `public class ItemUpdateForm`
        - 수정 스펙에 맞는 property들 (수정 시 어떤 item을 수정하는 지 알아야 하기 떄문에 id 도 포함) 만을 설정
        - 수정 시 **요구사항에 맞는 Bean Validation 진행**
    - 이제 각 등록, 수정 Controller에서 해당 하는 Form 객체(DTO)를 사용하여 **데이터를 받아 온 후 도메인 객체로 값을 이전(변환)하여 저장**하면 됨
        - 등록 Controller (`@PostMapping("/add")`) → `public String addItem(@Validated @ModelAttribute("item") ItemSaveForm form ...`
            - ItemSaveForm 등록 DTO 사용
            - `@ModelAttribute("item")` 를 통해 model 에 itemSaveForm의 이름이 아닌 item의 이름으로 View template에 넘겨줌 (view template에서 item의 이름으로 값에 접근하기 위함)
            - 도메인 객체로 값 이전 후 repository에 저장
                
                ```java
                Item item = new Item();
                item.setItemName(form.getItemName());
                item.setPrice(form.getPrice());
                item.setQuantity(form.getQuantity());
                ```
                
        - 수정 Controller(`@PostMapping("/{itemId}/edit")`) → `public String edit(@PathVariable Long itemId, @Validated @ModelAttribute("item") ItemUpdateForm form, …`
            - ItemUpdateForm 수정 DTO 사용
            - 나머지는 등록 Controlle과 유사
    - Form 전송 객체 분리해서 등록과 수정에 딱 맞는 기능을 구성하고, 검증도 명확히 분리함

### Bean Validation in API (@RequestBody, HttpMessageConverter)

> 참고 : `@ModelAttribute` 는 HTTP 요청 파라미터(URL 쿼리 스트링, POST Form)를 다룰 때 사용, `@RequestBody` 는 **HTTP Body의 데이터를 객체로 변환할 때 사용(API JSON 요청)**
> 

- `@Validated` 를 통한 검증은 `@RequestBody`(**HttpMessageConverter**) 에도 적용 가능
- API 3가지 경우
    - 성공
    - 실패 : JSON 객체로 **생성하는 것 자체**가 실패
    - 검증 오류 : JSON 객체로 **생성 후 검증**에서 실패
- 여기서 중요한 부분은 **JSON 객체로 생성하는 것 자체가 실패한 경우**
- JSON 객체로 생성하는 것 자체가 실패한 경우는 오류 코드(메세지)들이 반환되는 것이 아닌 “400 Bad Request” 가 반환됨
- 즉, price에 **Integer가 아닌 String으로 값을 넣는** 경우 J**SON 객체 생성 자체가 안되**며 Controller가 호출 자체가 안되고 검증 자체가 실행되지 않고 오류가 발생해버림!!

<aside>
⚠️ <strong><code>@ModelAttribute</code>(Post Form 형식) 에서는 TypeMismatch도 잘 잡아서 검증이 그대로 진행 되었는데 왜 <code>@RequestBody</code>(<b>HttpMessageConverter</b>)는 안되는거지? 똑같은 검증 로직 아닌가…?</strong>
<ul>
<li><code>@ModelAttribute</code> 는 <b>필드 단위</b>로 정교하게 <b>바인딩이 적용</b>됨. 고로 특정 필드가 바인딩 되지 않아도 나머지 필드는 정상 바인딩 되고, Validator를 사용한 검증도 적용. 즉, <b>각각에 적용되므로 오류가 발생해도 해당 부분에 대해서 오류를 추가해주고 Controller로 넘어가는 것</b>
<li><code>@RequestBody</code>(<b>HttpMessageConverter</b>)는 <code>@ModelAttribute</code> 와 다르게 각각의 필드 단위로 적용되는 것이 아니라, <b>전체 객체 단위</b>로 적용. 따라서 HttpMessageConverter 단계에서 JSON 데이터를 객체로 변경하지 못하면 <b>이후 단계 자체가 진행되지 않고 예외가 발생</b>. 컨트롤러도 호출되지 않고, Validator도 적용할 수 없음. 즉, 세부적으로 진행하지 못하고 <b>전체를 바인딩하기에 오류가 발생하면 다음으로 넘어가지 못하고 예외를 발생</b>시키는 것
</ul>
</aside>