---
layout: post
title:  "Thymeleaf - Integration with Spring"
date:   2022-05-16
image:  /posts/spring+thymeleaf.png
tags:   SpringBoot Thymeleaf
---

## Thymeleaf with Spring (개념)

- 타임리프는 스프링 없이도 동작. 즉, **다른 프레임워크와도 사용 가능**.
- 하지만, **스프링과의 통합**을 위한 **다양한 기능**을 편리하게 제공.
- 타임리프 2가지 메뉴열
    - 기본 메뉴얼:  [https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html](https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html)
    - **스프링 통합 메뉴얼**: [https://www.thymeleaf.org/doc/tutorials/3.0/thymeleafspring.html](https://www.thymeleaf.org/doc/tutorials/3.0/thymeleafspring.html)
    - 기본 메뉴얼은 Thymeleaf의 기본 기능들에 대한 Docs
    - 스프링 통합 메뉴얼은 **Thymeleaf를 Spring과 함께 썼을 때의 추가적인 기능들**에 대한 Docs
- **스프링 통합으로 추가되는 기능들**
    - 스프링의 **SpringEL 문법 통합** (property 접근법 등)
    - `${@customBean.doMethod()}` 처럼 **스프링 빈 호출** 가능
    - 편리한 **Form 관리**를 위한 **추가 속성**
        - `th:object` : Form의 대상 객체(Object) 선정 가능
        - 그에 따른 `th:field` , `th:errors` , `th:errorclass` 이용 가능
    - **Form Component** 기능
        - checkbox, radio button, List 등을 편리하게 사용 가능
    - 스프링의 **Message, internationalization 통합**
    - 스프링의 **Validation, Error Control 통합**
    - 스프링의 **ConversionService 통합**

<aside>
⚠️ <strong>이러한 통합 특징 때문에라도 Spring을 쓰는 백엔드 개발자는 대부분 SSR시 View Template으로 Thymeleaf를 사용</strong>

</aside>

## Integration Feature (Thymeleaf & Spring)

> **Spring 과 Thymeleaf를 같이 사용함**으로써 이용할 수 있는 좋은 **Form 기능들** 소개
> 

### 입력 폼 처리

- Thymeleaf의 **입력 Form 에서 사용**할 수 있는 기능 (더 효율적으로 개선 가능)
- `th:object` : 커맨드 객체 지정
    - **Form에서 사용할 객체를 지정**하는 것
    - 이를 통해 `*{…}`(**선택 변수 식**)을 사용할 수 있음
    - 내가 입력 Form에서 사용(입력 받고, 입력 값을 저장)할 객체를 지정함으로 써 그 객체의 **property들을 입력 Form의 input 요소들과 맞추어** 오류를 방지할 수 있음!
- `*{…}` : 선택 변수 식
    - `th:object` 에서 선택한 **객체의 property에 접근**
    - 선택 변수 식을 사용하면 `${item.itemName}` → `*{itemName}` 으로 더 편리하게 사용 가능
- **`th:field`** : input의 속성 자동 처리
    - input 태그의 중요 속성인 `id` , `name` , `value` **속성을 자동으로 처리**
    - `value` 속성을 자동으로 처리함으로써, **수정 시 기존의 값을 편리하게 가져올 수 있음!**
    - `<input type="text" th:field="*{itemName}" />` 
    → `<input type="text" id="itemName" name="itemName" th:value="*{itemName}" />`
    
    <aside>
    💡 <strong>입력 폼 처리 장점!</strong> <br>
    원래는 일일히 <code>id</code> , <code>name</code> , <code>value</code> 를 직접 입력해야 돼서 귀찮았을 뿐만 아니라, 오류(오타로 인한) 발생이 굉장히 컸지만, <code>th:object</code>, <code>*{…}</code> , <code>th:field</code> 사용하게 되면서 <b>자동으로 해당 값들을 지정된 객체에 맞게 설정</b>해줌으로써 오류도 방지하고 편리해짐. <br>
    이뿐만 아니라 <b>입력 폼 처리의 장점은 Validation에서 진가를 보임!!</b> 
    
    </aside>
    
    <aside>
    ⚠️ <strong>입력 폼 처리 주의점!</strong> <br>
    <code>th:object</code> 를 사용하기 때문에 등록(수정 Form이 아닌)에서도 <code>th:object</code> 에 넣어줄 <b>빈 Object</b>를 Controller에서 지정하여 Model로 넘겨 줘야 됨! (<code>model.addAttribute(”item”, new Item())</code>)
    </aside>
    

### 체크 박스 처리 (Single)

- 체크 박스에서 사용되는 데이터 설정
    - **enum, 일반 class**. 두가지로 설정 (**다양한 데이터 설정**을 보여주기 위함)
    - **enum**
        - 상품 종류
        - BOOK(도서), FOOD(식품), ETC(기타)
        
        ```java
        public enum ItemType {
        
        	 BOOK("도서"), FOOD("식품"), ETC("기타");
        
        	 private final String description;
        
        	 ItemType(String description) {
        		 this.description = description;
        	 }
        
        	 public String getDescription() {
        		 return description;
        	 }
        }
        ```
        
        - **생성자를 통한 설명 추가** (저장은 영어코드로, 보여주는 것은 한국어로 보여주기 위함 → 자주 쓰이는 패턴)
    - **class**
        - 배송 종류
        - FAST : 빠른 배송, NORMAL : 일반 배송, SLOW : 느린 배송
        
        ```java
        @Data // @Getter + @Setter + @ToString + ...
        @AllArgsConstructor
        public class DeliveryCode {
        	 private String code;
        	 private String displayName;
        }
        ```
        
        - 앞선 enum의 설정과 같이 code는 시스템에 영어로 전달하는 값, displayName은 한국어로 보여주는 값
        - 종류들의 기본 값은 Controller에서 설정해서 보냄
- 단일 체크 박스 사용 **(Thymeleaf 기능 사용 전)**
    - 체크 박스를 선택했을 때 (체크했을 때)
        - 체크 박스 input의 name에 on이라는 값과 함께 HTML Form형식(쿼리파라미터 형식)으로 전달됨
        → `<input type="checkbox" id="open" name="open">` 
        → `open=on`
        - 그럼 Spring의 **TypeConverter가 on을 boolean형식에 맞게 true로 변환**해줌
        - 여기까지는 문제가 없음!
    - 체크 박스를 선택 해지할 때 **(체크하지 않았을 때)**
        - `open` 이라는 필드 자체가 **서버로 전송되지 않음**
        - 즉 @ModelAttribute 로 Binding 할 때 open 이라는 값이 넘어오지 않았기에 해당 Object의 **open은 null 값으로 저장**됨.
        - null 이기 때문에  Spring의 TypeConverter가 false로 변환해주지 않음!
        - 이렇게 되면 체크 했다가 체크를 푸는 수정 상황에서 open이 null이라는 값을 받아, 변경 감지가 진행되지 않음! → **값이 유지됨!**
    - **체크 해제를 인식하기 위한 히든 필드**
        - 이런 문제를 해결하기 위해 Spring MVC는 `_` 를 사용하는 **히든 필드**를 통해 true, false를 인지!
        - `<input type="hidden" name="_open" value="on"/>`
        - open 이 체크되어 서버로 값이 전송되면 히든 필드는 값을 넘기지 않음! → `open=on` → `open=true`로 정상적으로 저장
        - open이 체크되지 않았다면 **히든 필드인 _open이 서버로 값이 전송**되며 Spring MVC는 이를 인식하여 **체크되지 않았다는 작동을 진행!** → `_open=on` → `open=false` 로 정상적으로 저장
    
    <aside>
    🚨 <strong>BUT!!</strong> <br>
    이 동작들은 너무 귀찮음. 계속해서 히든 필드를 넣어줘야 하고, 뿐만 아니라 open의 값을 수정할 때 checked 특성상 추가적인 코드가 필요함!
    
    </aside>
    

- **단일 체크 박스 사용 (Thymeleaf 기능을 통한 개선)**
    - `th:field` 를 사용하면 모든 것이 해결됨
        - `<input type="checkbox" id="open" name="open">` → `<input type="checkbox" th:field="*{open}">`
        - 렌더링 후의 결과를 확인해보면 input의 히든 필드도 자동으로 들어간 것을 확인할 수 있음 → **field가 자동으로 히든 필드를 넣어 동작하게 설정하는 것!** (귀찮음 해결)
        - 또한 수정 시 **checked 속성에 대한 처리도 자동으로** 해줌 (input 태그에 checked 라는 속성이 있으면 무조건 체크되는 특징)
            - `th:field` 를 통해 value를 받아와 true라면 자동으로 checked 속성을 넣어주고
            - false라면 checked 속성을 빼줌

<aside>
💡 <strong>결론!</strong> <br>
체크박스 필드는 무조건 <code>th:field</code> 를 사용하자!

</aside>

### 체크 박스 처리 (Multi)

- 참고
    - 멀티 체크 박스에 데이터를 넘겨줘야 하는 상황
    - 이를 `@ModelAttribute` 로 좀 더 효율적으로 진행
    - 💡 `@ModelAttribute`**의 특별한 사용법**
        - 각 Controller의 Method에 데이터를 일일히 다 넣어줘야함. 즉, 각 Method에 `model.addAttribute`를 통해서 다 각각 넣어줘야함. BUT!! `@ModelAttribute` 를 통해서 **하나의 Controller에 공통적으로 Model에 attribute를 넣어**줄 수 있음!
        
        ```java
        @ModelAttribute("regions")
        public Map<String, String> regions() {
        	 Map<String, String> regions = new LinkedHashMap<>();
        	 regions.put("SEOUL", "서울");
        	 ...
        	 return regions;
        }
        ```
        
        - Controller 단에 이렇게 하나의 modelAttribute를 정의하는 method를 만들어 놓고 반환해주면, **Controller의 각 Method에 들어가는 Model에 기본값으로 설정** 됨.
        - 즉, 각 Method에서 `model.addAttribute` 를 통해 반복적으로 넣지 않고도 사용할 수 있는 것. (Controller 단에서 넣어줬기 때문에)
- 데이터는 일반적인 String + Map 활용
- 멀티 체크 박스 html
    
    ```html
    <div>
    	 <div>등록 지역</div>
    	 <div th:each="region : ${regions}" class="form-check form-check-inline">
    		 <input type="checkbox" th:field="*{regions}" th:value="${region.key}" class="form-check-input">
    		 <label th:for="${#ids.prev('regions')}" th:text="${region.value}" class="form-check-label">서울</label>
    	 </div>
    </div>
    ```
    
    ```html
    <!-- 렌더링 결과 -->
    <div class="form-check form-check-inline">
    	 <input type="checkbox" value="SEOUL" class="form-check-input" id="regions1" name="regions">
    	 <input type="hidden" name="_regions" value="on"/>
    	 <label for="regions1" class="form-check-label">서울</label>
    </div>
    ...
    ```
    
    - each 부분
        - `${regions}` : Model에 따로 넣어준 지역목록
        - 따로 넣어준 지역 목록에 따라 반복문이 도는 것
    - input 부분
        - each에 따라 `${regions}` 의 size 만큼 생김
        
        <aside>
        💡 <strong>그럼 그 이 size 만큼 생긴 각각의 input의 id와 name은?</strong> <br>
        name은 그대로 regions를 가지지만, id는 동일하면 안되기 때문에 순서대로 regions1 → regions2 → … 와 같이 <b>뒤에 count를 붙여</b>주게 됨 <br>
        ⇒ 즉, HTML의 <b>id 가 타임리프에 의해 동적으로 만들어짐</b> (<code>th:each</code> + <code>th:field</code> 특징)
        
        </aside>
        
        - 추가로 `th:field` 의 checkbox 특징으로 히든필드도 넣어진 것 확인 가능
    - label 부분
        - 여기서 label 또한 클릭할 경우 그에 따른 input이 체크가 되어야 되는데 (`label for=”?”`) input의 **id가 동적으로 생기기 때문에 추가적인 처리** 필요 → `${#ids}`유틸리티 객체를 이용
        - `<label th:for="${#ids.prev('regions')}"...>`
            - `#ids.prev('regions')` : 직전의 regions로 **동적으로 생성된 id를 이어 받는다**는 뜻.
    
    <aside>
    ⚠️ <strong>추가 주의 점</strong>
    <ul>
    <li><strong>Form 안의 List의 서버 전송 값</strong></li>
        <ul>
        <li>`regions=SEOUL&_regions=on&regions=BUSAN&_regions=on&_regions=on`</li>
        <li>즉, 체크된 놈들에 대해서 regions=’’ 라는 값으로 순서대로 넣어짐. 이렇게 되면 <b>서버에서는 순서대로 add</b> 한다고 생각하면 됨 → <code>regions.add(”SEOUL”);</code> , <code>regions.add(”BUSAN”)</code> </li>
        </ul>
    <li><strong>체크 확인 (수정 시)</strong></li>
        <ul>
        <li>이전과 달리 멀티 체크 박스는 true, false 값으로 인지할 수 없는 List 형식의 String 형식. </li>
        <li>그럼 어떻게? → <b><code>th:field</code> 에 지정한 값과 <code>th:value</code> 의 값을 비교해서 체크를 자동으로 처리</b> </li>
        <li>즉, item의 regions (<code>th:field</code>)에 현재 each로 돌고 있는 해당 region(<code>th:value</code>)의 값이 있다면 checked 속성이 들어가고 없으면 넣지 않는 것! </li>
        </ul>
    </ul>
    </aside>
    

### 라디오 버튼

- 데이터는 ENUM 활용
- 라디오 버튼의 특징 : **하나만 선택할 수 있고, 선택 해제가 불가능**
- 먼저 `@ModelAttrubute` 를 이용하여 Controller 단 Model에 ENUM List 추가
    
    ```java
    @ModelAttribute("itemTypes")
    public ItemType[] itemTypes() {
    	 return ItemType.values();
    }
    ```
    
- 라디오 버튼 사용 html
    
    ```html
    <div th:each="type : ${itemTypes}" class="form-check form-check-inline">
    	 <input type="radio" th:field="*{itemType}" th:value="${type.name()}"class="form-check-input">
    	 <label th:for="${#ids.prev('itemType')}" th:text="${type.description}"class="form-check-label"></label>
    </div>
    ```
    
    - each 부분
        - `@ModelAttrubute` 을 통해 Model에 넣은 itemsTypes를 반복 (선택될 수 있는 후보 목록들에 대한 반복)
    - input, label 부분
        - Multi 체크박스와 동일
        - But input의 **히든필드가 생기지 않음** → 라디오 버튼은 수정 시 선택해제가 불가능 하므로, 체크 이후에 null값이 발생할 일이 없기 때문!
    
    <aside>
    ⚠️ <strong>주의할 점</strong>
    <ul>
    <li><b>체크 확인 (수정 시):</b> <code>th:field</code> 에 지정한 값과 <code>th:value</code> 의 <b>값을 비교해서 체크를 자동으로 처리</b> → 라디오 버튼 같은 경우는 지정되어 있는 <code>th:field</code> 와 <code>th:value</code> 가 동일한 애만 checked 속성을 넣어줌 </li>
    <li>라디오 버튼은 <b>하나만 체크</b>할 수 있다는 점 주의! 즉, 서버로 선택된 하나의 값만 보내짐 </li>
    </ul>
    </aside>
    

### 셀렉트 박스

- 데이터는 자바 객체(Object) 를 이용
- 셀렉트 박스의 특징 : 하나만 선택 가능
- 먼저 `@ModelAttrubute` 를 이용하여 Controller 단 Model에 Object List추가
    
    ```java
    @ModelAttribute("deliveryCodes")
    public List<DeliveryCode> deliveryCodes() {
    	 List<DeliveryCode> deliveryCodes = new ArrayList<>();
    	 deliveryCodes.add(new DeliveryCode("FAST", "빠른 배송"));
    	 ...
    	 return deliveryCodes;
    }
    ```
    
    > **참고:** `@ModelAttrubute` 의 메서드는 컨트롤러가 호출 될 때 마다 사용되어 그 때마다 객체가 생성된다는 점에 유의 → **Bean, Component 로 등록**해서 **Static**하게 사용하여 객체를 미리 생성하고 불러와 사용하는 것이 더 좋음
    > 
- 셀렉트 박스 사용 html
    
    ```html
    <select th:field="*{deliveryCode}" class="form-select">
    	 <option value="">==배송 방식 선택==</option>
    	 <option th:each="deliveryCode : ${deliveryCodes}" 
    					 th:value="${deliveryCode.code}"
    					 th:text="${deliveryCode.displayName}">FAST</option>
    </select>
    ```
    
    - `@ModelAttrubute` 을 통해 Model에 넣은 deliveryCodes를 option 태그 에서 반복 (선택될 수 있는 후보 목록들에 대한 반복)
    - select 특성상 input과는 다르게 select에 `th:field` 가 들어감
    - 즉, 체크 확인은 option 중 `th:value` 가 select의 `th:field` 와 동일한 애에 대해서 체크해줌. → selected=”selected” 라는 속성을 추가해줌으로써 체크 표시.