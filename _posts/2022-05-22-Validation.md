---
layout: post
title:  "Validation"
date:   2022-05-22
image:  /posts/validation.png
tags:   SpringBoot Validation
---

## Validation의 필요성

- 웹 서비스는 어떤 상황이든 오류가 발생하면, 그 **오류에 대한 처리는 필수**
- 특히 Form 입력 시 오류가 발생하면, 입력된 **데이터를 유지한 상태**로 **어떤 오류가 발생**했는지 친절하게 알려주어 사용자의 편의성을 높여야 됨
- 이런 검증 자체는 **Controller에서 판단**하고 그 결과에 따라 동작할 수 있도록 설정되어야 함

<aside>
⚠️ <strong>Client Validation VS Server Validation</strong>
<ul>
<li><strong>Client Validation</strong></li>
    <ul>
    <li>즉각적인 오류 검증 가능 → 사용성 증가</li>
    <li>외부적으로 조작이 가능하며, 보안에 취약</li>
    <li>JavaScript 등으로 동작</li>
    </ul>
<li><strong>Server Validation</strong></li>
    <ul>
    <li>즉각적인 오류 검증 불가능 (Client Validation에 비해)</li>
    <li>외부적인 조작이 불가능, 보안에 강함</li>
    <li>Controller 단에서 실행</li>
    <li>API 방식으로 검증 가능 (API 스펙을 잘 정의해서 검증 오류를 API 응답 결과에 잘 남겨주어야 함 )</li>
    </ul>
<b>결론 : 둘을 적절히 섞어서 사용하되, 보안을 위해 최종적으로 서버 검증은 필수</b>
</ul>
</aside>

## Validation 구현

- Spring이나 Thymeleaf에서 지원하는 검증 기능을 사용하지 않고 **직접 Validation을 처리**해보기 **(원리를 이해하기 위함)**
- Validation 로직 - 원리
    - 기존에 오류 없이 서버로 값이 전달될 때 (**검증 통과 로직**)

      ![]({{site.baseurl}}/images/posts/post-220522/Untitled.png)
        
        1. 상품 등록 폼을 요청하면 상품 등록 폼을 받게됨
        2. 상품 저장 시 입력된 값들에 대해 **오류가 없다면** 그대로 상품 상세 페이지로 **Redirect** 됨
        3. **Redirect**로 인해 상품상세 페이지를 받게 됨 **(PRG- Post, Redirect, Get)**
    - **오류 발생 (검증 미통과 로직)**

      ![]({{site.baseurl}}/images/posts/post-220522/Untitled 1.png)

        1. 상품 등록 폼을 요청하면 상품 등록 폼을 받게됨
        2. 상품 저장 시 입력된 값들에 대해 **오류가 발생**하면 **검증에 따른 로직 실행**
        3. 현재 **입력된 값들(오류를 포함된 값)**을 Model에 담아 다시 상품 등록 폼에 넘겨 렌더링 후 상품등록 폼을 오류 메시지와 함께 받게 됨
    - 즉, 검증에 실패한 경우 고객에게 **다시 상품 등록 폼**을
    보여주고, **어떤 값을 잘못 입력했는지** 친절하게 알려주어야 함 → **Server Validation**

### Validation 직접 처리

- Validation 로직 (오류 검출)**직접 개발**하기
- 직접 **errors 라는 Map**을 만들어서 오류가 발생할 때마다 **errors 에 담아주는 형식**으로 개발 진행
- 모든 검증 과정을 거친 후의 errors Map에 하나 이상의 error가 존재한다면 그 **error를 model에 담아** 상품 등록 폼으로 넘겨주고 해당 **오류를 표시**해주는 방식
- **Controller 단**
    - **error를 검출**하는 역할
    - `Map<String, String> errors = new HashMap<>();` : 검증 오류 결과를 보관
    - **Field 검증** (Object의 각각의 field 에 대한 오류 검출)
        - 상품 이름이 비어있는 오류 검출
            
            ```java
            if (!StringUtils.hasText(item.getItemName())) {
            	 errors.put("itemName", "상품 이름은 필수입니다.");
            }
            ```
            
            - `item`은 `@ModelAttribute`를 통해 Form의 입력값들을 받아온 객체
            - `StringUtils.hasText` 를 통해 빈문자열이거나 Null이면 오류 검출 후 오류를 담도록 진행
        - 상품 가격이 비어있거나 정해진 범위를 벗어나는 오류 검출
            
            ```java
            if (item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() >1000000) {
            	 errors.put("price", "가격은 1,000 ~ 1,000,000 까지 허용합니다.");
            }
            ```
            
        - 상품 수량이 비여있거나 정해진 범위를 벗어나는 오류 검출
            
            ```java
            if (item.getQuantity() == null || item.getQuantity() >= 9999) {
            	 errors.put("quantity", "수량은 최대 9,999 까지 허용합니다.");
            }
            ```
            
    - **Object 검증** (**여러 Field를 복합적으로 판단**한 검증)
        - 상품 가격 * 상품 수량이 범위를 벗어나는 오류 검출
            
            ```java
            if (item.getPrice() != null && item.getQuantity() != null) {
            	 int resultPrice = item.getPrice() * item.getQuantity();
            	 if (resultPrice < 10000) {
            		 errors.put("globalError", "가격 * 수량의 합은 10,000원 이상이어야 합니다.현재 값 = " + resultPrice);
            	 }
            }
            ```
            
    - 이렇게 Field 검증, Object 검증을 수행하고 오류가 발견되면 **해당 오류에 대해 errors Map 에 넣어줌**
    - 검증 실패 (error가 하나라도 존재하는 경우)
        
        ```java
        if (!errors.isEmpty()) {
        	 model.addAttribute("errors", errors);
        	 return "validation/v1/addForm";
        }
        ```
        
        - 만약 검증에서 오류 메시지가 하나라도 있으면 오류 메시지를 출력하기 위해 **model에 errors 를 담고, 입력
        폼이 있는 뷰 템플릿으로 보냄**
        - 이때, **사용자가 입력한 데이터** 또한 그대로 model에 담겨 뷰 템플릿으로 넘어가게 됨
        
        <aside>
        ⚠️ <strong><code>model.addAttribute(”item”,item)</code> 을 하지 않았는데 어떻게 <code>model</code>에 담겨지는 거지?</strong> <br>
        <code>@ModelAttribute</code> 의 <b>특별한 기능</b> 때문. 입력된 값을 <code>@ModelAttribute</code> 를 통해 item에 넣어주는 작업을 parameter를 통해 했는데, 여기서 <code>@ModelAttribute</code>의 <b>두번째 기능</b>인 <b><code>model</code>에 자동으로 담겨지는 기능</b>이 발동해서 따로 추가하지 않아도 자동으로 들어간 것!
        
        </aside>
        
- 뷰 템플릿
    - **Object 오류** (복합 검증 실패 시 발생하는 오류)
        
        ```html
        <div th:if="${errors?.containsKey('globalError')}">
        	 <p class="field-error" th:text="${errors['globalError']}">전체 오류메시지</p>
        </div>
        ```
        
        - `th:if="${errors?.containsKey('globalError')}"` : errors 라는 Map에 globalError key가 들어있다면 오류 메시지 보여줌
            - **`errors` 의 `?` 는 null일 경우를 방지하는 것.**
            - 만약 `errors` 가 `null` 인데 `containsKey`를 이용하게 되면 `NullPointException`이 터지게 됨
            - 그래서 `?` 를 통해서 errors가 null이면 **null인 상태로 반환**하게 하는 것
    - **Field 오류** (Object의 각각의 Field에서 발생하는 오류)
        
        ```html
        <div>
        	 <label for="itemName" th:text="#{label.item.itemName}">상품명</label>
        	 <input type="text" id="itemName" th:field="*{itemName}"
        					 th:class="${errors?.containsKey('itemName')} ? 'form-control field-error' : 'form-control'"
        					 placeholder="이름을 입력하세요">
        					<!-- th:classappend="${errors?.containsKey('itemName')} ? 'fielderror' : _"
        							class="form-control"  사용 가능-->
        	 <div class="field-error" th:if="${errors?.containsKey('itemName')}" th:text="${errors['itemName']}">
        		 상품명 오류
        	 </div>
        </div>
        ```
        
        - `th:class="${errors?.containsKey('itemName')} ? 'form-control field-error' : 'form-control'"` : errors 라는 Map에 itemName key가 들어있다면 오류 custom class를 적용시켜줌
        - `th:if="${errors?.containsKey('itemName')}"` , `th:text="${errors['itemName']}"` : errors 라는 Map에 itemNamekey가 들어있다면 오류 메시지 보여줌
        
        <aside>
        ⚠️ <strong><code>th:classappend</code> 를 사용해도 됨</strong> <br>
        <code>class="form-control" th:classappend="${errors?.containsKey('itemName')} ? 'fielderror' : _"</code>
        
        </aside>
        
- 남은 문제점
    - 뷰 템플릿의 오류 처리부분을 보면 하나의 오류처리에 너무나 조잡하고 중복되는 코드가 많은 것을 확인할 수 있음
    - **타입 오류 처리 불가**. → 타입 오류같은 경우 스프링MVC에서 **컨트롤러에 진입하기도 전에 예외가 발생**하기 때문에 (현재는 컨트롤러 안에서 오류 처리를 진행 중), 컨트롤러가 호출되지도 않고, 400 예외가 발생하면서 오류 페이지가 띄워짐
    - 이러한 문제점들은 **스프링이 제공하는 검증 방법**을 통해 **해결 가능!**

### BindingResult (Spring이 제공하는 검증 오류) - [Version 1]

- **스프링이 제공하는 검증 오류 처리 방법**
- **BindingResult**
    - **연결된 객체 바인딩 시** 발생하는 **오류를 담아**주는 역할
    - View Template에 자동으로 넘겨짐 (Integration with Spring)
    - 직접 오류를 검출하여 **custom error를 담아줄 수**도 있음
- **Controller 단** (BindingResult 사용 후 코드 변경 점)
    - Controller Method
        - `public String addItemV1(@ModelAttribute Item item, BindingResult bindingResult, ... ) {...}`
        - 항상 BindingResult는 `@ModelAttribute` 의 **대상 객체와 연동**되어야 하기 때문에 **순서 상 `@ModelAttribute` 뒤에 붙여줘야 됨!!**
    - **FieldError** :  `bindingResult.addError(new FieldError(...))` ( ← `errors.put(…)` )
        - Ex)  `bindingResult.addError(new FieldError("item", "itemName", "상품 이름은 필수입니다."));`
        - **FieldError 생성자**
            - `public FieldError(String objectName, String field, String defaultMessage) {…}`
            - String objectName : `@ModelAttribute` 대상 **객체 이름**
            - String field : 오류가 발생한 객체의 **필드 이름**
            - String defaultMessage : 보여줄 **오류 메시지**
    - ObjectError : `bindingResult.addError(new ObjectError(...))` ( ← `errors.put(…)` )
        - Ex) `bindingResult.addError(new ObjectError("item", "가격 * 수량의 합은 10,000원 이상이어야 합니다. 현재 값 = " + resultPrice));`
        - **ObjectError 생성자**
            - `public ObjectError(String objectName, String defaultMessage) {...}`
            - String objectName : `@ModelAttribute` 대상 **객체 이름**
            - String defaultMessage : 보여줄 **오류 메시지**
- 뷰 템플릿 단 (BindingResult 사용 후 코드 변경 점)
    - **Object Error**
        
        ```html
        <div th:if="${#fields.hasGlobalErrors()}">
        	 <p class="field-error" th:each="err : ${#fields.globalErrors()}" th:text="${err}">글로벌 오류 메시지</p>
        </div>
        ```
        
        - `${#fields.hasGlobalErrors()}` ( ← `${errors?.containsKey('globalError')}`)
        - `th:each="err : ${#fields.globalErrors()}" th:text="${err}"` ( ←  `th:text="${errors['globalError']}"` )
        - **`#fields`** 를 통해 BindingResult가 제공하는 **검증 오류에 접근 가능**
            - `hasGlobalErrors()` : Global(Object) Error가 존재하는 지 확인
            - `globalErros()` : Global(Object) Error들
    - Field Error
        
        ```html
        <input type="text" id="itemName" **th:field**="*{itemName}"
        			 **th:errorclass**="field-error" class="form-control"
        			 placeholder="이름을입력하세요">
        <div class="field-error" **th:errors**="*{itemName}">
        	 상품명 오류
        </div>
        ```
        
        - `th:errorclass="field-error" class="form-control"` ( ← `th:class="${errors?.containsKey('itemName')} ? 'form-control field-error' : 'form-control'"` )
        - **`th:errorclass`**
            - `th:field` 와 연동해서  `BindingResult`를 통해  해당 **field의 오류가 있다**고 인지가 되면 **자동**으로 `errorclass`의 `class`를 기존 `class`에 **append 해줌** (`th:field`가 검증에서도 역할을 하는 모습)
        - `th:errors="*{itemName}"` ( ← `th:if="${errors?.containsKey('itemName')}" th:text="${errors['itemName']}"`)
        - `th:errors`
            - `th:if` 와 `th:text` 역할을 **모두 해줌**
            - `errors`에 연동된 field에 오류가 있다면(`th:if`), BindingResult를 통해 오류 메시지를 보여줌(`th:text`)
- **TypeMismatch** 처리하기
    - 이전에는 TypeMismatch 와 같이 Binding 시 발생하는 오류는 컨트롤러를 호출하기 전에 발생하는 문제라 처리하지 못했음 (400 오류가 발생, 오류페이지로 이동)
    - 하지만, **BindingResult** 가 있으면 `@ModelAttribute` 에 데이터 바인딩 시 오류가 발생해도 컨트롤러가 호출됨 (오류 정보( FieldError )를 BindingResult 에 담음)
    
    <aside>
    💡 <strong>BindingResult에 검증 오류를 적용하는 3가지 방법</strong>
    <ol>
    <li><code>@ModelAttribute</code> 로 지정된 객체에 <code>TypeMismatchError</code> 등으로 Binding이 실패하는 경우 자동으로 <code>FieldError</code> 를 생성해서 BindingResult에 담아줌</li>
    <li>개발자가 직접 비지니스 검증 로직을 수행 (직접 <code>FieldError</code>를 생성하여 BindingResult에 담는 것)</li>
    <li><code>Validator</code> 사용</li>
    </ol>
    </aside>
    
    <aside>
    🚨 <strong>BindingResult 사용 시 주의점!</strong>
    <ul>
    <li>BindingResult 는 순서 상 <b>검증할 대상</b>(ex_ <code>@ModelAttribute</code>) <b>바로 다음에 와</b>야함. (<code>ObjectError</code>, <code>FieldError</code> 와 연동하기 위해서!)</li>
    <li>BindingResult 는 <b>Model에 자동으로 포함</b>되기에 따로 add할 필요가 없음!</li>
    <li>BindingResult 는 인터페이스이고, <code>Errors</code> 인터페이스를 상속. 즉, <code>Errors</code>를 사용하여 넘겨줘도 되지만 <b>BindingResult가 더 많은 기능을 제공</b>함 (주로 관례상 BindingResult를 사용함)</li>
    </ul>
    </aside>
    
- **현재 문제점**
    - 오류 처리는 잘 되었지만, 오류가 발생하는 경우 **고객이 입력한 내용이 모두 사라짐!**

### BindingResult [Version 2]

- 오류가 발생하더라도 사용자가 입력한 값이 유지되도록 설정하기 → **FieldError, ObjectError 의 또 다른 생성자를 통해 해결 가능!**
    
    <aside>
    🚨 <strong>근데 왜 값이 없어지지? <code>@ModelAtrribute</code>를 통해서 값이 넘겨지지 않나?</strong>
    <ul>
    <li>이 부분은 뷰 템플릿에서 사용하는 `th:field="..."` 와 연관이 있음!</li>
    <li><code>th:field</code> 는 정상 상황에는 모델 객체의 값(Model로 넘겨진 값)을 사용하지만, <b>오류가 발생하면</b> <b><code>FieldError</code>에서 보관한 값을 사용</b>해서 값을 출력함</li>
    <li>그런데, 이전 version을 확인해 보면 FieldError에 따로 값을 보관하지 않았기에 오류가 발생했을 때 빈 값이 출력된 것!</li>
    <li>즉, 이제 <code>FieldError</code>에 사용자가 방금 <b>입력한 값을 보관</b>해주면 됨! → <b><code>FieldError</code>의 또 다른 생성자 사용</b></li>
    </ul>
    </aside>
    
- FieldError의 또 다른 생성자 (rejectedValue, …)
    
    ```java
    public FieldError(
    			String objectName, String field, 
    			@Nullable Object rejectedValue, boolean bindingFailure, 
    			@Nullable String[] codes, 
    			@Nullable Object[] arguments, @Nullable String defaultMessage )
    ```
    
    - String objectName : `@ModelAttribute` 대상 **객체 이름**
    - String field : 오류가 발생한 객체의 **필드 이름**
    - **Object** **rejectedValue** : 사용자가 입력한 값(**거절된 값**)
    - boolean bindingFailure : 타입 오류 같은 바인딩 실패인지, 커스텀 검증 실패인지 구분 값
    - String[] codes : 메시지 코드
    - Object[] arguments : 메시지에서 사용하는 인자
    - String defaultMessage : 보여줄 오류 메시지
    
    <aside>
    💡 <strong>ObjectError도 유사하게 [objectName, codes, arguments, defaultMessage] 를 제공하는 또 다른 생성자를 가지고 있음</strong>
    
    </aside>
    
- **ObjectError, FieldError 사용**
    - `bindingResult.addError(new ObjectError("item", null, null, "가격 *
    수량의 합은 10,000원 이상이어야 합니다. 현재 값 = " + resultPrice));`
    - `bindingResult.addError(new FieldError("item", "itemName", item.getItemName(), false, null, null, "상품 이름은 필수입니다."));`
        - `rejectedValue` 에는 사용자가 입력한 값을 저장한 item에서 가져옴
        - custom 검증 이기에 (바인딩 오류가 아님) bindingFailure 에는 false 값으로 설정
- 바인딩 실패 시 **자동으로 rejectedValue 등을 포함한 FieldError를 생성**하여 BindingResult에 담아줌!! → 타입 오류 같은 바인딩 실패시에도 사용자의 오류 메시지를 정상 출력 가능
    
    <aside>
    ⚠️ <strong>굳이 왜 FieldError의 rejectedValue에 입력 값을 저장해서 보내는 거지?</strong> <br>
    
    사용자의 입력 데이터가 컨트롤러의 <code>@ModelAttribute</code> 에 Binding 되는 시점에 오류가 발생하면 <b>대상 객체에 사용자 입력 값을 유지하기 어려움!</b> 예를 들어서 가격에 <b>Integer</b>가 아닌 <b>String</b>이 입력된다면 <b>Type이 다르므로 당연히 String을 보관할 수 있는 방법이 없음.</b> 그래서 오류가 발생한 경우 사용자 입력 값을 <b>보관하는 별도의 방법</b>이 필요하고 이를 <code>FieldError</code>의 <code>rejectedValue</code>를 통해서 수행하는 것!
    
    </aside>
    

## 오류 코드와 메시지 처리

- Validation Part에서 FieldError, ObjectError의 두번째 생성자 에서 볼 수 있었던  `@Nullable String[] codes, @Nullable Object[] arguments` 와 관련된 처리 부분 (**codes** : message source에서 **key에 해당하는** **message**를 가져오는 것, **arguments** : 그에 따른 **매개변수**들)
- **오류 메시지를** **체계적으로** 다루어보는 부분
- version이 차근차근 증가되면서 더 좋은 사용법을 알아보는 것

### Version 1

- `@Nullable String[] codes, @Nullable Object[] arguments`
    - 메시지 파트에서 다루었던 개념! (message source를 지정하고 그 message를 가져오는…)
    - `codes` : message source에서 **key에 해당하는 message**를 가져오는 것 (MessageSource.getMessage() 라고 생각하면 됨)
    - `arguments` : 가져오려 하는 message에서 요구하는 **매개변수들**
- errors.properties
    - messages.properties 에다 설정해도 되지만 구분을 위해 새로운 **message source 파일 생성**
    - `spring.messages.basename=messages,errors` : messages 는 기본 값이지만 errors를 message source로써 사용하려면 **basename에 추가** 필요 (`application.properties` 에). 이렇게 하면 두 파일 모두에서 참고. ( 추가로 errors_en.properties를 통해 **국제화 기능도 활용 가능** )
    - src/main/resources/errors.properties
        
        ```yaml
        required.item.itemName=상품 이름은 필수입니다.
        range.item.price=가격은 {0} ~ {1} 까지 허용합니다.
        max.item.quantity=수량은 최대 {0} 까지 허용합니다.
        totalPriceMin=가격 * 수량의 합은 {0}원 이상이어야 합니다. 현재 값 = {1}
        ```
        
    - **ObjectError, FieldError**
        - `bindingResult.addError(new ObjectError("item", new String[]
        {"totalPriceMin"}, new Object[]{10000, resultPrice}, null));`
            - codes : `new String[]{"totalPriceMin"}` 
            ⇒ errors.properties에서 totalPriceMin의 이름을 가진 message를 가져오는 것
            ⇒ Type이 `String[]{}` 인 부분 주의!
                
                <aside>
                ⚠️ <strong>codes는 왜 String 배열로 받는거지?</strong><br>
                codes는 <b>String 배열의 순서대로 우선순위</b>를 가짐. 즉, 첫번째 요소가 message source에 존재한다면 그대로 사용, 만약 없다면 다음 요소로 message source 탐색.
                
                </aside>
                
            - arguments :  `new Object[]{10000, resultPrice}`
            ⇒ totalPriceMin message에서 필요로 하는 매개변수들
            ⇒ Type이 `Object[]{}` 인 부분 주의!
            - 결과 오류 메시지 예시 : `가격 * 수량의 합은 10000원 이상이어야 합니다. 현재 값 = 2000`
            - `FieldError` 도 사용법은 동일!
            
            <aside>
            ⚠️ <strong>참고로 해당 codes를 message source에서 찾을 수 없다면 defaultMessage가 출력됨 (defaultMessage가 null 이면 오류!)</strong>
            
            </aside>
            

### Version 2

- 방금까지 사용했던 `FieldError`, `ObjectError` 는 코드도 너무 길고 다루기 너무 번거러움
- 또한 오류코드(String 배열)도 다 일일히 작성해주어서 귀찮음!
- 이를 해결하기 위한 것이 `BiningResult`에서 제공하는 `rejectValue()` [←FieldError], `reject()` [←ObjectError]. ⇒ 각각을 개발자가 직접 생성할 필요 없이 자동으로 생성해주는 것!
- `rejectValue()` [←FieldError]
    - `void rejectValue(@Nullable String field, String errorCode, @Nullable Object[] errorArgs, @Nullable String defaultMessage);`
    - field : 오류 **필드명**
    - errorCode : **오류 코드** (**`messageResolver`**를 위한 오류 코드)
    - errorArgs : 오류 메시지에서 요구하는 **파라미터**
    - defaultMessage : 오류 메시지를 찾을 수 없을 때 사용하는 **기본 메시지**
    - Ex) `bindingResult.rejectValue("price", "range", new Object[]{1000, 1000000}, null)`
        - field는 price → item.price
        - errorCode는 range → **`MessageResolver`** 를 통해 **reange.item.price**로 인식
- `reject()` [←ObjectError]
    - `void reject(String errorCode, @Nullable Object[] errorArgs, @Nullable String defaultMessage);`
    - `rejectValue()` 와 거의 동일

<aside>
⚠️ <strong>자세하게 적지 않아도 <code>rejectValue</code>, <code>reject</code> 가 동작되는 이유!</strong>
<ul>
<li>BindingResult는 <b>자기가 검증해야되는 객체가 누군지 이미 알고</b> 있기 때문(<code>@ModelAttribute</code> 와 연계되어). 그래서 <code>FieldError</code> 시에는 해당 Field만 지정해주면 되고 <code>ObjectError</code> 시에는 지정해주지 않아도 됨!</li>
<li>또 message 자체도 <b>대상 객체와 대상 필드를 안 상태</b>에서 <b><code>MessageResolver</code>가 동작</b>하여 자동으로 message source에서 해당 key를 통해 message를 찾아오는 것!</li>
</ul>
</aside>

### MessageResolver

- `errorCode` 의 단순한 error message key를 통해 어떻게 `errors.properties`에서 세밀한 error message를 가져오는 지에 관한 내용
- 소개
    
    ```yaml
    required.item.itemName=상품 이름은 필수 입니다.
    range.item.price=상품의 가격 범위 오류 입니다.
    
    required=필수 값 입니다.
    range=범위 오류 입니다
    ```
    
    - 이런식으로 message source가 짜여 있다고 가정
    - required 처럼 단순하게 만들면 범용성이 좋아 여러 곳에서 일관되게 사용 가능, but 세밀하게 동작하게 하려면 사용이 힘듦
    - 반대로 required.item.itemName 처럼 너무 자세하게 만들면 범용성이 떨어짐.
    - 결국 가장 좋은 방법은 **먼저 범용성으로 사용**하다가, 세밀하게
    작성해야 하는 경우에는 세밀한 내용이 적용되도록 **메시지에 단계를 두는 방법**
    - 즉, 세밀한 부분이 아닐 경우에는 required 만 오류 코드에 넣어주고, 세밀한 부분 (객체의 필드에서의 오류) 에서는 required.item.itemName 와 같이 객체명과 필드명을 조합한 **세밀한 메시지를 넣어**주고 이에 대해 우선순위를 높여주면 됨. → codes = [required.item.itemName, required]
    - **이런 동작을 Spring 의 `MessageCodesResolver` 가 자동으로 해줌!**
- `MessageCodesResolver`
    - **Interface**, 구현체로는 보통 `DefaultMessageCodesResolver` 를 사용
    - `MessageCodesResolver codesResolver = new DefaultMessageCodesResolver();`
    - Object 와 Field 지정에 따라 **오류 코드 생성**
        - 순서에 따라 우선순위를 가짐
        - Object 만 지정
            - `String[] messageCodes = codesResolver.resolveMessageCodes("required", "item");`
            - **messageCodes = ["required.item", "required”]**
            - 에러 코드 생성 규칙
                - [Level 1] : code + "." + object name
                - [Level 2] : code
        - Object + Field 지정
            - `String[] messageCodes = codesResolver.resolveMessageCodes("required", "item", "itemName", String.class);`
            - messageCodes = ["required.item.itemName", "required.itemName", "required.java.lang.String", "required"]
            - 에러 코드 생성 규칙
                - [Level 1] : code + "." + object name + "." + field
                - [Level 2] : code + "." + field
                - [Level 3] : code + "." + field type
                - [Level 4] : code
    - 즉, `rejectValue()`, `reject()` 는 내부에서 이런 `MessageCodesResolver` 를 사용해서 에러 메시지 코드들을 생성.
- `rejectValue()` 동작 과정 (`reject()`도 유사)
    - 먼저 BindingResult에 연결된 Object와 rejectValue에서 설정된 Field, 기본 error 코드를 가지고 **MessageCodesResolver**를 통해 codes(에러 메시지 코드들) 생성
        
        → codes = `codesResolver.resolveMessageCodes("required", "item", "itemName", String.class);`
        
    - 그 후 **FieldError**를 해당 Object, Field, codes를 가지고 생성
    → fieldError = `new FieldError("item", "itemName", item.getItemName(), false, **codes**, null, "상품 이름은 필수입니다.")`
    - 마지막으로 **BindingReuslt에 해당 Error 추가**
    → `bindingResult.addErrors(**fieldError**)`
- 이렇게 설정된 `BindingReuslt`를 View Template에 보내주면 `th:errors` 에서 메시지 코드들의 우선순위에 맞는 메시지를 노출 (없으면 defaultMessage 출력)

<aside>
💡  <strong>오류 코드 관리 전략</strong>
<ul>
<li> 먼저 기본적으로 가장 하위 Level에 해당하는 오류 메시지 (ex_required)를 만들어주고 추후에 상세한 설명이 필요한 부분이 생겼다면 상세적인 오류 메시지(ex_required.item.itemName)를 만들어 주기만 하면 됨 (message source 기반이므로 유지 보수성 UP!)</li>
<li> 즉, 크게 중요하지 않은 메시지는 범용성 있는 requried 같은 메시지로 끝내고, 정말 중요한 메시지는 꼭 필요할 때 구체적으로 적어서 사용하는 방식이 더 효과적이다.</li>
</ul>
</aside>

- 스프링이 직접 만든 오류 메시지 처리
    - **타입 오류**와 같은 오류는 컨트롤러가 호출하기 전, Binding 시 발생
    - 즉, 우리가 직접 `rejectValue()`로 처리해줄 수 없고, Spring 자체에서 자동으로 `BindingResult`에다 오류를 담아주는 것 (`rejectValue()`와 똑같은 원리로)
    - 이때 담긴 Error의 에러 메시지 코드를 확인하면 
    ⇒ codes = `[typeMismatch.item.price, typeMismatch.price, typeMismatch.java.lang.Integer, typeMismatch]`
    - 즉, `bindingResult.rejectValue(”price”, “typeMismatch”)` 가 **자동으로 적용** 된 것!
    - 이를 처리하려면 그냥 우선순위(Level)에 따른 **`typeMismatch`에 대한 오류 메시지**를 errors.properties에 추가해 주면 됨!! (만약 설정해주지 않는 다면 스프링에서 생성한 defaultMessage(개발자스러운 오류 메시지..)로 노출됨)
        
        ```yaml
        typeMismatch.java.lang.Integer=숫자를 입력해주세요.
        typeMismatch=타입 오류입니다.
        ```
        
- 지금 까지 알아본 `MessageResolver`의 원리와 동작은 실제 자주 사용되는 **Bean Validation**에서 굉장히 핵심 개념임!!

## Validator 사용 (검증 로직 분리)

- 현재 Controller의 코드를 보면 성공 로직보다 **검증 로직이 차지하는 부분이 더 많음!** 이런 경우 **검증 로직을 분리해서 실행**해줄 필요가 있음 (반복되는 코드도 줄이고, 로직을 재사용할 수 있음)
- **Version 1** (`Validator` 를 DI 받아 사용하기)
    - `Validator` Interface 를 구현한 `ItemValidator` 생성
    - `ItemValidator`는 i**tem 객체와 관련된 검증 로직을 수행**해주는 부분
    - 즉, 기존의 Controller에 있던 item 객체의 검증 로직을 `ItemValidator`로 옮겨 주고, 사용할 때는 그냥 **`itemValidator`를 통해 검증을 수행**하면 됨
    - **ItemVaidator**
        
        ```java
        @Component
        public class ItemValidator implements Validator {
        
        	 @Override
        	 public boolean supports(Class<?> clazz) {
        		 return Item.class.isAssignableFrom(clazz); // 지원하는 class인지 확인 여기선 item 객체만을 사용
        	 }
        
        	 @Override
        	 public void validate(Object target, Errors errors) {
        		 Item item = (Item) target;
        		  ... //(Controller에서 사용했던 검증 로직 부분. bindingResult를 errors로 바꿔주기만 하면 됨)
        	 }
        }
        ```
        
        - `supprots` 를 통해 해당 검증 로직이 **해당 Object를 지원하는 지** 확인 후 `validate` 를 통해 검증 로직 수행
        - `validate`를 통해 **검증로직 수행**
            - 검증했을 때 오류가 발견되면 errors(BindingResult)에 담기게 됨
    - **Controller**에서 `ItemValidator` 사용
        
        ```java
        // if (itemValidator.supprot(item)) {
        itemValidator.validate(item, bindingResult);
        // }
        if (bindingResult.hasErrors()) {
        	 log.info("errors={}", bindingResult);
        	 return "validation/v2/addForm";
        }
        ```
        
        - `itemValidator`는 DI로 받아옴 → `private final ItemValidator itemValidator;`
        - validate에 `item`과 `bindingResult`를 보내줌
        - `bindingResult`는 `Error`의 자식이므로 validate에서 Error로써 동작할 수 있음. 추가로 해당 검증 로직으로 인한 오류가 `bindingResult`에 담기게 됨
        - 이후 로직은 동일
    - 이렇게 하면 검증로직을 Controller와 분리하고 깔끔하게 재사용할 수 있음!!
    
- **Version 2** (`WebDataBinder`, `@Validated` 를 사용하여 **더 간편하게 사용**하기)
    - **WebDataBinder**
        - 스프링의 파라미터 바인딩의 역할을 해주고 검증 기능도 내부에 포함
        - `WebDataBinder`와 `@InitBinder` 를 통해 **하나의 Controller안의 모든 method에 동일한 Validator를 일괄적으로 적용**해 줄 수 있음
            
            ```java
            @InitBinder
            public void init(WebDataBinder dataBinder) {
            	 log.info("init binder {}", dataBinder);
            	 dataBinder.addValidators(itemValidator);
            }
            ```
            
            - Controller 가 실행 되는 시점에 `WebDataBinder`를 생성하고 그 binder에 `itemValidator`를 넣어줌
            - 그럼 모든 method (해당 Controller 안)에서 `itemValidator` 를 통한 검증 가능!
        
        > **참고 (글로벌 설정)**: 모든 컨트롤러에 다 적용하려면 `Application`에 `getValidator()` 를 통해 등록해주면 됨 (자주 안 쓰임)
        > 
        
    - **@Validated** (등록된 validator 사용)
        - 바인딩 되는 객체(ex_`@ModelAttribute`)에 해당 **어노테이션**을 달아주면 (`@Validated @ModelAttribute Item item`) 등록된 `Validator`를 통해 검증 실행, 실행 후 오류가 발견되면 바인딩 되는 객체 뒤에 따라오는 **`BindingResult`에 담아**줌!! → 이전과 버전과 동일하게 실행 가능
            
            ```java
            @PostMapping("/add")
            public String addItemV6(
            	@Validated @ModelAttribute Item item, BindingResult bindingResult,
            	RedirectAttributes redirectAttributes) {
            
            	 // 오류 검출 시 동작
            	 if (bindingResult.hasErrors()) {
            		 log.info("errors={}", bindingResult);
            		 return "validation/v2/addForm";
            	 }
            
            	 ... // 성공 로직 부분
            }
            ```
            
        - 즉, 모든 Controller에 Method에서 검증이 무작정 실행되는 것이 아니라 **`@Validated` 가 달린 객체에 대해서만 검증을 실행**하는 것
        
        <aside>
        ⚠️ <strong>만약 Validator가 item 뿐만 아닌 다른 객체에 대해서도 존재한다면?</strong> <br>
        이때 Vaidator의 <b><code>support</code> 가 동작</b>하는 것! <code>@Validated</code> 가 달린 객체에 대해 등록된 모든 Validator의 <code>support</code>를 통해 <b>지원하는 Validator만을 이용하여 검증을 시행</b>하는 것!
        
        </aside>
        

<aside>
🚨 <strong>지금까지 Validation에 대해 배웠지만, 사실 이 모든 것들은 Bean Validation 이라는 어노테이션 하나로 해결할 수 있다</strong>

</aside>