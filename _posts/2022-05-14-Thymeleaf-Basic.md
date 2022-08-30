---
layout: post
title:  "Thymeleaf"
date:   2022-05-14
image:  /posts/thymeleaf.png
tags:   SpringBoot Thymeleaf
---
## Thymeleaf (타임리프)

### 타임리프

- **타임리프** 란?
    - 서버 사이드 자바 **템플릿 엔진**
    - 공식 사이트 : [https://www.thymeleaf.org/](https://www.thymeleaf.org/)
    - 공식 메뉴얼 - 기본 기능: [https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html](https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html)
    - 공식 메뉴얼 - 스프링 통합: [https://www.thymeleaf.org/doc/tutorials/3.0/thymeleafspring.html](https://www.thymeleaf.org/doc/tutorials/3.0/thymeleafspring.html)

- 타임리프 특징
    - **서버 사이드 HTML 렌더링 (SSR)**

      ![]({{site.baseurl}}/images/posts/post-220514/Untitled.png)
        
        - **동적**으로 **HTML 최종 결과를 서버에서 만들**어서 웹 브라우저에 전달 (Thymeleaf, JSP, …)
        
        <aside>
        ⚠️<strong> 클라이언트 사이드 렌더링(CSR) </strong> <br>
        HTML 결과를 자바스크립트를 사용해 <b>웹 브라우저</b>에서 동적으로 생성해서 적용 (서버에서 생성된 HTML 결과를 받아오는 것이 아닌) (React, Vue.js, …)
        
        </aside>
        
    - **네추럴 탬플릿**
        - 순수 **HTML을 최대한 유지**하면서 뷰 템플릿도 사용할 수 있음
        - 즉, 서버를 거치지 않고도 형식을 갖춘 HTML 결과물을 확인할 수 있음 (동적인 결과는 서버를 거쳐서 확인해야됨)
        - (JSP와 같은 다른 view template 들은 JSP 코드와 HTML 코드가 섞여 있어 서버릴 거치지 않고 확인할 경우 다 깨져서 확인 불가능)
    - **스프링 통합 지원**
        - 스프링과 자연스럽게 통합되고, 스프링의 다양한 기능을 타임리프 안에서 편리하게 사용할 수 있음.
- 타임리프 사용 선언
    - `<html xmlns:th="http://www.thymeleaf.org">`
    - HTML 파일에 위와 같이 선언해주면 `th:…` 와 같이 사용할 수 있음

<aside>
💡 <strong> 결국 CSR을 이용할 건데, SSR을 해야 될 필요가 있을까? </strong> <br>
Admin 페이지나, CSR 적용 전의 <b>기본 동작이 되는 서비스</b>같은 경우 <b>SSR</b>이 필요함! 따라서 백엔드 개발자는 깊게, 유연하게 사용할 수 있는 정도는 아니더라도 어느정도 SSR에서 사용되는 View Template 하나 정도는 알고 사용할 수 있는게 좋음!

</aside>

## 타임리프 기본 기능 (문법)

### 텍스트 (text, utext)

- **텍스트를 출력하는 기능**
- `th:text` : HTML의 **콘텐츠(content)에** 원하는 **데이터를 출력**할 때 사용
    - `<span th:text="${data}"> 데이터 </span>`
    - th:text 에 원하는 값 할당. 값은 Controller의 Model에 담겨져 온 속성 값. (`model.addAttribute(”data”,”렌더링 데이터”)`)
    - 렌더링 시 해당 data의 값을 **기존**의 ‘데이터’라는 **값을 그대로 치환**하여 보여줌.
    - 만약 서버(렌더링)를 거치지 않는다면 HTML을 본다면 해당 항목은 기존의 ‘데이터’ 라는 값을 가지게 됨 → **내추럴 템플릿 특징**
- `[[ ]]` : HTML 테그의 속성이 아니라 **HTML 콘텐츠 영역안에서 직접** 데이터를 출력하고 싶을 때 사용
    - `<span> 직접 출력 : [[${data}]] </span>`
    - 말 그대로 HTML 콘텐츠 영역 안에 직접 데이터를 넣어 출력하는 것.
    - 얘는 Default text가 없기에 서버(렌더링)을 거치지 않고 HTML을 뿌리면 `직접 출력 : [[${data}]]` 의 text 그대로 나오게 됨 (”깨지진 않는다”는 점이 핵심 → **내추럴 템플릿 특징** )
    - 추후 **자바스크립트에서 model의 속성(변수)를 사용할 때** 해당 문법 이용
- **Escape (text)**
    - HTML에서 사용하는 **특수 문자를 HTML 엔티티로 출력하는 것**
    - HTML은 `<`, `>` 와 같은 특수 문자 기반.
    - 그렇다면 만약 데이터에 해당 `<`, `>` 와 같은 특수문자가 있다면? → 그 **text 그대로 출력**됨.
        - ex) `data = “<b> Hello! </b>”` , `<span> 직접 출력 : [[${data}]] </span>`
        - 렌더링 후 → `직접 출력 : <b> Hello! </b>`
        - 즉, Hello가 b 태그를 먹어서 강조되는 것이 아닌, 그 html 문법 자체의 text가 그대로 나오게 됨.
        - HTML 렌더링 결과의 코드를 보면 `&lt;b&gt;Hello!&lt;/b&gt` 와 같이 되어 있음. 즉, 그냥 **text로써 인식할 수 있게 HTML 엔티티로 바꿔서 출력**.이게 바로 **ESCAPE**
- **Unescape (utext)**
    - Escape 기능을 사용하지 않은채로 출력하는 방법
    - Thymeleaf에서의 Escape → Unescape
        - `th:text` → `th:utext`
        - `[[ … ]]` → `[( … )]`
    - Unescape을 사용하게 되면 **HTML 코드 그대로 적용** 시킬 수 있음!
        - ex) `data = “<b> Hello! </b>”`
            - `<span th:utext="${data}"> 데이터 </span>` → **Hello!**
            - `<span> 직접 출력 : [(${data})] </span>` → 직접 출력 : **Hello!**
    - 마크다운 적용 시 이용 가능!
        
        

### 변수 - SpringEL

- **변수 표현식**
    - `${ … }`
    - 타임리프에서 **Model로 받아 온 변수** 사용하기
    - 즉, 타임리프를 통해 HTML 코드 안에서 Model로 받아 온 변수를 사용할 수 있게끔 하는 표현식
    - 추가로 이 변수 표현식에는 **SpringEL**이라는 **스프링이 제공하는 표현식**(ex_ `item.getName()`) 사용 가능 → **스프링 통합 지원 특징**
- 사용
    - username, age 의 속성을 가지고 있는 User 객체 하나, 이런 2개의 User를 담은 List, Map 를 Model에 담아서 View Template으로 넘긴다고 가정
        
        ```java
        model.addAttribute("user", userA);
        model.addAtribute("users", new ArrayList<User>(List.of(...)));
        model.addAtribute("userMap", new HashMap<User>(...));
        ```
        
    - **Object (User 하나)**
        - 속성 접근 (Java : `user.getUsername()`)
            - **SpringEL (in Thymeleaf)**
                - `${user.username}`
                - `${user[’username’]}`
                - `${user.getUsername()}`
            - 모두 **`user.getUsername()**` 으로 property를 가져오는 것
            
            <aside>
            🚨 <strong> SpringEL Obejct property 접근 주의! </strong> <br>
            Getter로 가져오는 것이기 때문에, Spring 안에서의 Object에서 Getter가 열려 있지 않으면 가져올 수 없음!! → 스프링 통합 지원 특징
            
            </aside>
            
    - **List**
        - 인덱스 및 속성 접근 (Java : `list.get(0).getUsername()`)
            - SpringEL (in Thymeleaf)
                - `users[0].username`
                - `users[0]['username']`
                - `users[0].getUsername()`
            - 모두 **`list.get(0)`** 으로 요소를 가져오는 것
    - Map
        - 접근 (Java: `map.get("userA").getUsername()`)
            - SpringEL (in Thymeleaf)
                - `userMap['userA'].username`
                - `userMap['userA']['username']`
                - `userMap['userA'].getUsername()`
            - 모두 **map.get("userA")`** 으로 요소를 가져오는 것
- 지역변수 사용
    - `th:with = "변수=${…}"`
    - 지역 변수를 **선언한 영역 내에서만 사용 가능**함 (지역 변수 특징)
    - Ex) `first` 라는 지역 변수 선언
        
        ```html
        <div th:with="first=${users[0]}">
        	<p>처음 사람의 이름은 <span th:text="${first.username}"></span></p>
        </div>
        ```
        
        - 이렇게 되면 선언된 div 영역 안(하위)에서만 `first` 사용 가능 → 지역 변수
        

### 기본 객체들

- 식 기본 객체 (Expression Basic Objects)
    - `${#request}` : HttpServletRequest 객체 접근
        - `${#request.getParameter(...)}` 와 같이 쿼리 파라미터 접근 가능
    - `${#response}` : HttpServletResponse 객체 접근
    - `${#session}` : **session 접근**
        - 중요!
        - 로그인 판단 상황에서 사용 가능
    - `${#servletContext}` : ApplicationContext 객체 접근
    - `${#locale}` : 언어 접근 → ex) ko, en, …
- **편의 객체**
    - 많이 사용되는 기능들을 쉽게 접근할 수 있도록 지원
    - `${param}` : HTTP 요청 파라미터(query parameter, @RequestParm) 접근
        - …?paramData=Hello → `${param.paramData}` 로 접근 가능
        - `${#request.getParameter("paramData")}` 와 동일
        - 쿼리 파라미터를 워낙 많이 사용하기 때문에 param을 통해 편하게 접근하도록 지원
    - `${session}` : HTTP Session 접근
        - `session.setAttribute("sessionData", "Hello Session");` → `${session.sessionData}`
        - 로그인 여부에 대해 Session을 자주 사용하기 때문에 session을 통해 편하게 접근하도록 지원
    - `{@..}` : 스프링 빈(Bean) 및 메서드 접근
        - `@Component("helloBean")...` → `${@helloBean.hello('Spring!')}`
        - Component(Bean)로 등록된 스프링 빈을 더 편리하게 사용할 수 있음. 즉, **해당 Bean의 메서드 및 속성**을 사용할 수 있는 것 → **스프링 통합 지원 특징**
        - 이걸 통해서 thymeleaf에서 지원하지 않는 수식, 기능 등을 Bean을 통해서 사용할 수 있음! (굉장히 유용함)
        

### 유틸리티 객체와 날짜

- 유틸리티 객체들 (날짜 포함)
    - 공식 문서 (설명 및 사용법):
        - [https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html#appendix-b-expression-utility-objects](https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html#appendix-b-expression-utility-objects)
        - 아래의 유틸리티 객체들에 대한 설명과 사용법
        - 필요할 때마다 공식 문서 확인해가며 쓰면 됨
    - `${#message}` : 메시지, 국제화 처리
        - `${#messages.msg('msgKey')}` : messages.properties에서 msgKey와 관련된 message를 가져오는 것
    - `${#uris}` : URI 이스케이프 지원
        - `${#uris.escapePath(uri)}` : 앞서 본 HTML escape과 같이 불러온 uri를 URL Escape 처리 해주는 것
    - `${#dates}` : java.util.Date 서식 지원
        - `${#dates.format(date, 'dd/MMM/yyyy HH:mm')}` : 불러온 date 형식을 지정된 형식에 맞게 표기할 때
    - `${#calendars}` : java.util.Calendar 서식 지원
        - Date와 기능은 동일
    - `${#temporals}` : 자바8 날짜 서식 지원
        - `${#temporals.day(localDateTime)}`, `${#temporals.hour(localDateTime)}`, … :  Java8 날짜 서식 지원
    - `${#numbers}` : 숫자 서식 지원
        - `${#numbers.formatInteger(num,3)}` : 최소 digits 설정
    - `${#strings}` : 문자 관련 편의 기능
        - `${#strings.toString(obj)}` : String으로 변환
        - `${#strings.isEmpty(name)}` : 빈 문자열 체크
    - `${#objects}` : 객체 관련 기능 제공
        - `${#objects.nullSafe(obj,default)}` : Null이면 default 반환
    - `${#bools}` : boolean 관련 기능 제공
        - `${#bools.isTrue(obj)}` : th:if 와 같은 기능
    - `${#arrays}` : 배열 관련 기능 제공
        - `${#arrays.length(array)}` : 배열 길이 반환
        - `${#arrays.isEmpty(array)}` : 빈 배열 확인
        - `${#arrays.contains(array, element)}` : 요소 포함 여부 확인
    - `${#lists}`  , `${#sets}` , `${#maps}` : 컬렉션 관련 기능 제공
        - array와 기능은 동일

### URL 링크

- URL in Thymeleaf
    - `th:herf = “@{...}"`
    - 단순 URL
        - `th:herf = "@{/hello}"` 
        → /hello
        - 사실 단순 URL에서는 굳이 th문법을 사용하지 않아도 됨 (`herf=”/hello”`)
    - 쿼리 파라미터
        - `th:herf = "@{/hello(param1=${param1}, param2=${param2})}"` 
        → /hello**?**param1=data1&param2=data2
        - 경로 상에 **변수가 없는 상태**에서 괄호_() 안에 있는 부분이 **쿼리 파라미터로 처리**됨 (자동으로 ‘?’ 도 넣어줌 → 쿼리 파라미터 형식)
        - Model로 받아온 속성값을 통해 쿼리파라미터를 요청할 수 있는 것
    - 경로 변수
        - `th:herf = "@{/hello/{param1}/{param2}(param1=${param1}, param2=${param2})}"` 
        → /hello/data1/data2
        - URL **경로상에 변수**가 있으면 () 부분은 **경로 변수로 처리**됨
    - 경로 변수 + 쿼리 파라미터
        - `th:herf = "@{/hello/{param1}(param1=${param1}, param2=${param2})}"` 
        → /hello/data1?param2=data2
        - 경로 변수와 쿼리 파라미터를 함께 사용 가능 → URL 경로 상에 있는 건 **경로 변수**로, 없는 변수는 자동으로 **쿼리 파라미터**로 설정 됨

### 리터럴 (Literals)

- **리터럴**
    - 소스 코드상에 **고정된 값**. (변수가 아닌)
        - `String a = “Hello”` 
        → a : 변수, **“Hello” : 리터럴**
    - Thymeleaf 리터럴 문법 : 변수를 th 문법으로 사용하는 것이 아닌 고정된 문자, 문자열, 숫자를 th 문법으로 사용할 때 쓰는 것
    - **항상 `‘ ’`(작은따옴표) 로 감싸**야 함.
    - 하지만 **공백 없이 쭉 이어진다**면 하나의 의미있는 토큰으로 인지해서 작은 따옴표 생략 가능 (`A-Z` , `a-z` , `0-9` , `[]` , `.` , `-` , `_` 에 해당하는 리터럴들만)
    - 즉, **공백이 들어간 리터럴** `th:text="hello world!"` 는 오류 발생 ->  `th:text="'hello world!'"` 처럼 **작은따옴표로 감싸**줘야 함
- **리터럴 대체 문법**
    - 대체 문법 없이 리터럴을 변수와 같이 사용할 때의 문법
        - `<span th:text="'hello '+${data}">`
        - 이런 식으로 작은따옴표와 같이 하나의 리터럴로 만들어 줘야 함 → 공백도 직접 지정해줘야 하고 복잡하며 귀찮음!
    - **대체 문법 사용**
        - `th:text = “| … |”`
        - `<span th:text="|hello ${data} world|">`
        - `| … |` 로 감싸주기만 하면 **‘리터럴과 변수 조합’**을 굉장히 편하고 자유롭게 사용 가능!

### 연산

- 비교 연산
    - < , > , .. 와 같이 **HTML 엔티티의 형식**이 사용되기 때문에 주의!
    - 근데, 연산 시에는 그냥 써주면 됨. → HTML 태그로 사용할 것이 아니기 때문!
    - 즉, th:text 안에서 그냥 사용해도 그에 따른 판별 결과가 나옴
        - `<span th:text = “1 < 10”> result </span>` → true
    - `> (gt), < (lt), >= (ge), <= (le), ! (not), == (eq), != (neq, ne)` 연산자 사용 가능!
- **조건식**
    - 비교 연산과 똑같이 **자바 문법에 맞는 조건식**을 사용하면 됨
    - `<span th:text="(10 % 2 == 0)? '짝수':'홀수'"> </span>` → 짝수
- **Elvis 연산자 [중요]**
    - 조건식의 편의 버전
    - 데이터가 있으면 그 데이터가 나오고, 없으면 설정된 값이 나옴
    - `<span th:text="${data}?: '데이터가 없습니다.'"></span>`
        - data 가 null이 아닐 때 → data에 들어 있는 값
        - data 가 null일 때 → ‘데이터가 없습니다.’
- **No-Operation**
    - 조건에 따라 `_`인 경우 **마치 타임리프가 실행되지 않는 것 처럼 동작**
    - 즉, `_` 가 선택되면 **렌더링 전의 HTML 코드**가 그대로 실행됨. (타임리프 문법이 설정이 안된 것 마냥)
    - `<span th:text="${data}?: _"> 데이터가 없습니다. </span>`
        - data 가 null이 아닐 때 → data에 들어 있는 값
        - data 가 **null**일 때 → ‘데이터가 없습니다.’ (Elvis와는 다르게 `_` 가 선택되었기 때문에 해당 부분은 **Thymeleaf가 적용되지 않고 마치 Thymelaef가 없이 기존 설정된 기본 HTML 태그**(`<span> 데이터가 없습니다. </span>`)로 **실행** 됨)

### 기본 표현식 한방 정리

<aside>
💡 <strong>Thymeleaf Basic Use</strong>

<ul>
<li> 간단한 표현 <br>
◦ 변수 표현식: ${...} <br>
◦ 선택 변수 표현식: *{...} <br>
◦ 메시지 표현식: #{...} <br>
◦ 링크 URL 표현식: @{...} <br>
◦ 조각 표현식: ~{...}
</li>
<li> 리터럴 <br>
◦ 텍스트: 'one text', 'Another one!',… <br>
◦ 숫자: 0, 34, 3.0, 12.3,… <br>
◦ 불린: true, false <br>
◦ 널: null <br>
◦ 리터럴 토큰: one, sometext, main,…
</li>
<li>문자 연산: <br>
◦ 문자 합치기: + <br>
◦ 리터럴 대체: |The name is ${name}|
</li>
<li>산술 연산: <br>
◦ Binary operators: +, -, *, /, % <br>
◦ Minus sign (unary operator): - 
</li>
<li>불린 연산: <br>
◦ Binary operators: and, or <br>
◦ Boolean negation (unary operator): !, not 
</li>
<li>비교와 동등: <br>
◦ 비교: >, <, >=, <= (gt, lt, ge, le) <br>
◦ 동등 연산: ==, != (eq, ne)
</li>
<li>조건 연산: <br>
◦ If-then: (if) ? (then) <br>
◦ If-then-else: (if) ? (then) : (else) <br>
◦ Default: (value) ?: (defaultvalue) 
</li>
<li>특별한 토큰: <br>
◦ No-Operation: _
</li>
</ul>
</aside>

### 속성 설정 및 추가

- 속성 (attribute) : <div class=”” name=””> 에서 class, name이 속성(attribute)
- 속성 설정
    - `th:`속성은 항상 **기존 속성보다 우선순위**를 가짐
    - 즉, 기존 속성과 `th:`속성이 있으면 기존 속성은 `th:`속성으로 **대체**됨
    - 만약 기존 속성이 없으면 `th:`속성에 따른 속성을 **새로 생성**
    - `<input type="text" name="mock" th:name="userA"/>` 를 Thymeleaf로 렌더링 하게 된다면 → `<input type="text" name="userA"/>`. 즉, 기존 속성을 대체함. (결국 대체하거나 새로 생성)
- 속성 추가
    - `th:attrappend` : 속성 값의 **뒤에** 값을 **추가**
        - `<input type="text" class="text" th:attrappend="class=' large'"/>` → `<input type="text" class="text large'"/>`
        - `"class=' large'"` : **앞의 공백** 주의! **리터럴 그대로 뒤에 추가** 되는 것이기 때문에 공백을 넣어줘야 됨
    - `th:attrprepend` : 속성 값의 **앞에** 값을 **추가**
        - `<input type="text" class="text" th:attrprepend="class='large '"/>` → `<input type="text" class="large text'"/>`
        - `"class='large '"` : **뒤의 공백** 주의! **리터럴 그대로 앞에 추가** 되는 것이기 때문에 공백을 넣어줘야 됨
    - `th:classappend` : class 속성에 **자연스럽게 추가** **[자주 쓰임]**
        - `<input type="text" class="text" th:classappend="large"/><br/>`
        - `th:attr` 과는 다르게 공백을 넣지 않아도 되고 리터럴 형식을 고려할 필요 없이 쉽게 추가 가능
        - **class 속성 추가**가 필요할 때는 해당 기능을 사용하는 것 추천
        - 나머지 속성 추가는 `th:attr` 을 리터럴 형식을 고려해서 사용해야 됨
- **checked 속성**
    - 특징 : checked = ‘true’, checked=’false’ 두 값 모두 checked 처리가 됨
    - 즉, checked가 어떤 값이든 간에 checked라는 속성이 있으면 해당 부분은 checked 처리가 됨
    - `th:checked`
        - `= “true”` : checked 속성을 **추가**해 주고
        - `= “false”` : checked 속성을 **제거**해 줌!
        - 즉, `th` 문법의 `checked`는 **boolean 형식**으로 돌아가도록 지원해줌! → 개발자 입장에서 BEST
        - `th:checked=”${isChecked}”` 와 같이 true false 값으로 checked를 설정할 수 있음

### 반복문 (반복)

- Java에서와 같이 반복문을 Thymeleaf에서 사용하는 방법
- **반복** 뿐만 아니라, 추가로 **반복에서 사용할 수 있는 여러 상태 값**을 지원
- `th:each` : 반복 기능
    - `<li th:each="user : ${users}">...</li>`
    - `users` 라는 컬렉션의 값을 하나씩 꺼내서 `user` 에 담아 해당 태그를 반복하는 것 → `li` 태그가  `users` 의 길이만큼 생김
    - 그 후 `user` 를 SringEL(변수 표현식)에 맞게 사용하면됨
        - `th:text="${user.username}"` , `th:text="${user.age}"` , …
    - `List` 뿐만 아니라 배열, `java.util.Iterable` , `java.util.Enumeration` 을 구현한 모든 객체를 반복에 사용 가능. `Map`의 경우 변수에 담기는 값은 `Map.Entry`
- `th:each` + **`Stat`**
    - `<li th:each="user, userStat : ${users}">`
    - `userStat` : 반복의 두번째 파라미터를 설정해서 **반복의 상태**를 확인 (each 문에서 생략하고 바로 사용 가능 → 그냥 “**지정한 변수명( user ) + Stat**” 으로 사용하면 됨 → `userStat`)
    - `.index`
        - `th:text="${userStat.index}"`
        - **0부터 시작**하는 값, 현재 반복되는 놈의 인덱스 반환
    - `.count`
        - `th:text="${userStat.count}`
        - **1부터 시작**하는 값, 현재 반복되는 놈이 몇번째 놈인지 확인 가능
    - `.size`
        - `th:text="${userStat.size}"`
        - 전체 사이즈, 현재 반복과 상관없이 해당 반복의 대상 컬렉션의 크기 확인
    - `.even` , `.odd`
        - `th:text="${userStat.even}"` , `th:text="${userStat.odd}"`
        - count의 **홀수, 짝수 여부**. boolean 값으로 반환됨
        - odd : 홀수면 true, 짝수면 false
        - even : 홀수면 false, 짝수면 true
    - `.first` , `.last`
        - `th:text="${userStat.first}"` , `th:text="${userStat.last}"`
        - **처음, 마지막 여부**. boolean 값으로 반환됨
    - `.current`
        - `th:text="${userStat.current}"` → User(username=userA, age=10)
        - **현재 객체**
        - 위의 user 객체는 @ToString으로 만들어줬기에 저런 식으로 출력
        - @ToString이 없으면 객체의 주소값 반환

### 조건부 평가

- 타임리프에서 조건식을 사용할 수 있게끔 하는 기능
- `if` , `unless` (if의 반대)
    - `th:if="${...}"` , `th:unless="${...}"`
    - 해당 조건이 맞지 않으면 태그 자체를 렌더링하지 않고 삭제함. 즉, **조건에 맞는 것만 렌더링하여 보여줌!**
    - `<span th:text="'미성년자'" th:if="${user.age < 20}"></span>`
    `<span th:text="'성인'" th:if="${user.age >= 20}"></span>`
        - user.age 가 18 이면 위의 if절(`미성년자`, true)만 렌더링, 밑의 if절(`성인`, false)은 삭제됨.
        - user.age 가 22 이면 위의 if절(`미성년자`, false)은 제거, 밑의 if절(`성인`, true)만 렌더링.
    - 굉장히 유용하게 쓰임!!
- `switch`
    - 기존 switch와 동일. 해당하는 case만 렌더링 후 보여짐.
    - Ex)
        
        ```html
        <td th:switch="${user.age}">
        	<span th:case="10">10살</span>
        	<span th:case="20">20살</span>
        	<span th:case="*">기타</span>
        </td>
        ```
        
        - `*`은 만족하는 조건이 없을 때 사용하는 디폴트
        

### 주석

- 표준 HTML 주석 (`<!-- -->`)
    
    ```html
    <!--
    <span th:text="${data}">html data</span>
    -->
    ```
    
    - 렌더링 하지 않고 이 그대로 주석된 상태로 HTML로 보내짐
- 타임리프 파서 주석 (`<!--/* /-→`)
    
    ```html
    <!--/* [[${data}]] */-->
    <!--/*-->
    <span th:text="${data}">html data</span>
    <!--*/-->
    ```
    
    - 타임리프의 진짜 주석
    - **렌더링을 할때 해당 주석 부분을 아예 제거**해버림
    - 즉, 렌더링하지 않고 HTML을 보내면 그냥 보여짐
- 타임리프 프로토타입 주석
    
    ```html
    <!--/*/
    <span th:text="${data}">html data</span>
    /*/-->
    ```
    
    - HTML 파일을 그대로 열어보면 주석처리가 되지만, 타임리프를 렌더링 한 경우에만 보이는 기능
    - 자주 안 쓰임
    

### 블록

- HTML 태그가 아닌 **타임리프 자체 태그**
- HTML 태그에 **th문법을 사용하기 애매한 경우**에 사용.
- `<th:block>`
    
    ```html
    <th:block th:each="user : ${users}">
    	 <div>
    		 사용자 이름1 <span th:text="${user.username}"></span>
    		 사용자 나이1 <span th:text="${user.age}"></span>
    	 </div>
    	 <div>
    		 요약 <span th:text="${user.username} + ' / ' + ${user.age}"></span>
    	 </div>
    </th:block>
    ```
    
    - 위의 상황 처럼 HTML 태그 안에 th 문법 (특히 `each` 문)을 사용하기 애매할 때 사용.
    - **렌더링시 제거**됨. 즉, 반복 기능만 실행
    - `th:each` 문에서 자주 사용!!
    

### 자바스크립트 인라인

- 자바스크립트에서 타임리프를 편리하게 사용할 수 있는 **자바스크립트 인라인 기능**
- Thymeleaf in JavaScript
    - `<script th:inline="javascript">`
    - **텍스트 렌더링**
        - `var username = [[${user.username}]];`
            - 인라인 사용 전 → `var username = userA;`
            - 인라인 사용 후 → `var username = "userA";`
        - 인라인 사용 전에는 문자 형식이 아닌 userA라는 변수로 할당이 됨 → 오류.
        - 인라인 사용 후에는 `user.username` 가 문자라는 것을 인식하고 **문자의 형식으로 변수에 할당 됨** → 정상 동작
        - 즉, `th:inline="javascript"` 을 사용하게 되면 **렌더링 결과를 판단해서 해당 타입에 맞게** 자동으로 `“”` 를 포함하는 등, 그 타입에 맞게 동작함
        - 추가로 자바스크립트에서 문제가 될 수 있는 문자가 포함되어 있으면 **이스케이프 처리**까지 진행
    - 자바스크립트 **내추럴 템플릿**
        - Thymeleaf의 내추럴 템플릿 기능 활용
        - 자바스크립트 인라인 기능을 사용하면 **주석을 활용해서 이 기능을 사용 가능**하게 함
        - `var username2 = /*[[${user.username}]]*/ "test username";`
            - 인라인 사용 전 →  `var username2 = /*userA*/ "test username";`
            - 인라인 사용 후 → `var username2 = "userA";`
        - 인라인 사용 전에는 내추럴 템플릿 기능 동작 X, 그냥 test username이 할당됨
        - 인라인 사용 후에는 주석 부분이 제거되고, "userA” 값이 할당 됨 → **내추럴 탬플릿 기능** (렌더링 하지 않으면 “test username”, 렌더링 하면 “userA”)
    - **객체**
        - 객체를 JSON으로 자동으로 변환
        - `var user = [[${user}]];`
            - 인라인 사용 전 →  `var user = BasicController.User(username=userA, age=10);`
            - 인라인 사용 후 → `var user = {"username":"userA","age":10};`
        - 인라인 사용 전은 객체의 `toString()` 으로 호출된 값
        - 인라인 사용 후는 객체를 JSON으로 변환해준 결과 → 이렇게 되면 좀 더 효율적으로 객체의 속성에 접근할 수 있음
    - **자바스크립트 인라인 each**
        - 자바스크립트에서 반복문과 같이 **each**를 사용할 수 있음
        - th:each in JavaScript
            
            ```jsx
            [# th:each="user, stat : ${users}"]
            var user[[${stat.count}]] = [[${user}]];
            [/]
            ```
            
            - [ ] 와 # 을 이용해서 기존 th문법과 동일하게 적용 가능
            
            ```jsx
            var user1 = {"username":"userA","age":10};
            var user2 = {"username":"userB","age":20};
            var user3 = {"username":"userC","age":30};
            ```
            
            - 결과를 보면 **변수를 개수에 맞게 생성하는 것**처럼 굉장히 유용하게 쓸 수 있다는 것을 알 수 있음!

### 템플릿 조각

- 웹에서 **공통적으로 보여지는 부분에 대한 처리**
- 상단 영역(header), 하단 영역(footer) 등 여러 페이지에서 함께 사용하는 영역을 **공통적인 HTML 파일로 불러와 사용**할 수 있도록 하는 것
- `/resources/templates/template/fragment/footer.html`
    
    ```jsx
    <!DOCTYPE html>
    <html xmlns:th="http://www.thymeleaf.org">
    	<body>
    		<footer th:fragment="copy">
    			 푸터 영역
    		</footer>
    		<footer th:fragment="copyParam (param1, param2)">
    			 <p>파라미터를 사용한 푸터 영역</p>
    			 <p th:text="${param1}"></p>
    			 <p th:text="${param2}"></p>
    		</footer>
    	</body>
    </html>
    ```
    
    - fragment로써 외부에서 사용할 영역은 `th:fragment=”이름”` 으로 지정해 줌
    - 공통적으로 사용은 하지만 각 사용에서 다른 용도로 사용할 수도 있기에 **파라미터**까지 지원함. → 파라미터를 사용하는 fragment는 함수처럼 선언 (`th:fragment="copyParam (param1, param2)"`)
    - 이제 공통적인 부분은 이 HTML 속 Fragment를 불러와 사용하면 됨
- Fragment 사용
    - **어떻게 포함**할 것(`th:*`)인지, 어느 위치의 **어떤 fragment**(`~{경로 :: 이름}`)인지 지정해주면 됨
    - **insert** (`th:insert`)
        - `<div th:insert="~{template/fragment/footer :: copy}"></div>`
        - `<div>` 영역 **안으로** fragment가 들어감 (`<div> <footer> … </footer> </div>`)
    - **replace (**`th:replace`**)**
        - `<div th:replace="template/fragment/footer :: copy"></div>`
        - `<div>` 영역을 **대체하여** fragment가 들어감 (`<footer> … </footer>`)
    - **단순 표현식**
        - `~{...}` 을 생략하여 사용하는 것
        - 템플릿 조각을 사용하는 코드가 단순할 때만 사용 가능
        - `<div th:insert="template/fragment/footer :: copy"></div>`
    - **파라미터 사용**
        - 공통으로 사용하지만 각 용도에 맞게 사용하기 위함
        - 파라미터를 전달해서 동적으로 fragment를 렌더링
        - `<div th:replace="~{template/fragment/footer :: copyParam ('데이터1', '데이터2')}"></div>`
        - 해당 인자에 맞는 fragment를 생성
    

### 템플릿 레이아웃

- Fragment 는 말 그대로 일부 코드 조각을 가지고 와서 넣거나, 대체하는 것
- Layout 은 개념을 더 확장해서 코드 조각을 레이아웃에 넘겨서 그 레이아웃을 받아오는 것 → 즉, 일부만 교체하고 그런 것이 아니라, 거의 전부를 넘겨 받아 사용하는 것

<aside>
💡 <strong> Fragment 와 Layout 둘 중 어떤 것을 사용해야 하나? </strong> <br>
페이지가 적을 때는 그냥 간단하고 사용하기 편한 <b>Fragment</b>를 사용, 하지만 페이지가 많고 공통적인 부분이 많다 하면 <b>Layout</b>을 사용하는 게 유지 보수에 좋음

</aside>

- **base layout (일부 layout)**
    - 시나리오 :<head> 에 공통으로 사용하는 css , javascript 같은 정보들이 있는데, 이러한 공통 정보들을 한 곳에 모아두고, 공통으로 사용하지만, 각 페이지마다 필요한 정보를 더 추가해서 사용하는 상황
    - base layout (`/resources/templates/template/layout/base.html`)
        
        ```html
        <html xmlns:th="http://www.thymeleaf.org">
        <head th:fragment="common_header(title,links)">
        	<!-- 개별 타이틀 -->
        	<title th:replace="${title}">레이아웃 타이틀</title>
        	
        	<!-- 공통 -->
        	<link rel="stylesheet" type="text/css" media="all" th:href="@{/css/awesomeapp.css}">
        	...
        	
        	<!-- 개별 추가 -->
        	<th:block th:replace="${links}" />
        </head>
        ```
        
        - 개별 html에서 base의 공통 head layout(fragment)를 사용하는 것
        - 여기서 인자로 받아온 값을 통해 개별 html의 **개별 요구사항을 만족**
        - 즉, 인자로 받아온 값을 통해 공통 head를 개별 head layout으로 만들어 보내 주는 것
    - base layout 사용 (`/resources/templates/template/layout/layoutMain.html`)
        
        ```html
        ...
        <head th:replace="template/layout/base :: common_header(~{::title},~{::link})">
        	 <title>메인 타이틀</title>
        	 <link rel="stylesheet" th:href="@{/css/bootstrap.min.css}">
        	 <link rel="stylesheet" th:href="@{/themes/smoothness/jquery-ui.css}">
        </head>
        ...
        ```
        
        - 현재 html의 개별 title과 추가 link를 인자로 넘겨줘서 base head layout을 만들어서 replace를 통해 head를 대체함
            - `~{::title}` : 현재 페이지의 title 태그들
            - `~{::link}` : 현재 페이지의 link 태그들
        - 그럼 layout을 통해 현재 title과 추가 link를 설정할 수 있음
- **base html layout (전체 layout)**
    - 상황 : <head> 정도에만 적용하는게 아니라 <html> **전체에 적용 하는 확장**해야 되는 상황
    - base html layout (`/resources/templates/template/layoutExtend/layoutFile.html`)
        
        ```html
        <!DOCTYPE html>
        <html th:fragment="layout (title, container)" xmlns:th="http://www.thymeleaf.org">
        <head>
         <title th:replace="${title}">레이아웃 타이틀</title>
        </head>
        <body>
        	<h1>레이아웃 H1</h1>
        	<div th:replace="${container}">
        		 <p>레이아웃 컨텐츠</p>
        	</div>
        	<footer>
        		 레이아웃 푸터
        	</footer>
        </body>
        </html>
        ```
        
        - 개별 title 과 container(전체 내용)을 받아와 layout을 만들어 줌
        - html 자체를 통으로 주는 것! (진정한 레이아웃)
    - base html layout 사용 (`/resources/templates/template/layoutExtend/layoutExtendMain.html`)
        
        ```html
        <!DOCTYPE html>
        <html th:replace="~{template/layoutExtend/layoutFile :: layout(~{::title},~{::section})}" xmlns:th="http://www.thymeleaf.org">
        <head>
        	 <title>메인 페이지 타이틀</title>
        </head>
        <body>
        	<section>
        		 <p>메인 페이지 컨텐츠</p>
        		 <div>메인 페이지 포함 내용</div>
        	</section>
        </body>
        </html>
        ```
        
        - 현재 html의 개별 title과 내용을 담은 section을 인자로 넘겨줘서 base html layout을 만들어서 replace를 통해 html를 대체함
            - `~{::title}` : 현재 페이지의 title 태그들
            - `~{::section}` : 현재 페이지의 section(container,내용) 태그
        - 그럼 layout을 통해 현재 title과 내용을 담은 html을 설정할 수 있음
        
        <aside>
        ⚠️ <strong> replace → replace … </strong> <br>
        둘 다 replace를 사용해서 값을 대체하고 있음! 고로 replace의 흐름과 순서를 통해 대체되는 순서를 파악하면 됨
        
        </aside>