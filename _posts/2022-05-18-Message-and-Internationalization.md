---
layout: post
title:  "Message, internationalization"
date:   2022-05-18
image:  /posts/messages.png
tags:   SpringBoot
---
## 메시지와 국제화란?

- 메시지
    - **공통적으로 사용되는 문구**에 대해서 미리 설정을 해두는 것
    - ex) ‘상품 이름’ 이라는 label에 붙은 문구를 전체적으로 ‘상품 명’ 으로 변경하고 싶은 상황
    → 메시지 기능을 사용하지 않는다면, 직접 일일이 다 수정해주어야 됨 (수정해야 될 html 파일이 200개가 있다고 생각해보면 끔찍..!)
    → 메시지 기능을 사용하면, **message 설정 부분 하나만 수정**해주면 **공통적으로 적용** 됨!
- 국제화
    - 메시지에서 한 발 더 나아간 개념
    - **요청**하는 고객이 선택한 **언어에 맞는 메시지**를 보여주는 것!
    - 한국, 영어 에 대한 메시지 국제화 설정이 되어 있으면, 한국에서 요청한 것이면 한국어로, 영어권에서 요청한 것이라면 영어로 각 문구들을 보여주는 것
    - 접근 국가 인식 방법
        - **HTTP 헤더**들 중 **`accept-language` 헤더** 값 이용
        - 사용자가 직접 선택한 언어를 **쿠키에 녹여 처리**
- 이런 메시지와 국제화 기능은 직접 구현 가능. BUT, **Spring은 기본적인 메시지와 국제화 기능 모두 제공!** **(Thymeleaf 또한 제공 → Thymelaef, Spring Integration)**
- 즉, 스프링에서 정해준 메시지 규약을 따르면 편리하게 메시지, 국제화 기능을 사용할 수 있음

## 메시지, 국제화 in Spring

### 스프링 메시지 소스 설정

- Spring Boot를 사용하지 않을 때 (**Only Spring**)
    
    ```java
    @Bean
    public MessageSource messageSource() {
    	 ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource();
    	 messageSource.setBasenames("messages", "errors");
    	 messageSource.setDefaultEncoding("utf-8");
    	 return messageSource;
    }
    ```
    
    - `MessageSource` 를 **스프링 빈으로 등록**
    - `MessageSource` 는 Interface 이기에 구현체인 `ResourceBundleMessageSource` 를 스프링 Bean 으로 등록
    - `setBasenames` : **메시지 설정 파일의 이름 지정**
        - messages로 지정하면 `messages.properties` 파일을 읽어서 사용
        - 국제화 설정 :  `_en`, `_ko` 처럼 **뒤에 언어 정보**를 넣어 주어 생성해주면 됨(`messages_en.properties`, …) 지정되어 있지 않은 언어에 대해선 default인 `messages.properties` 이용
        - 메시지 설정 **파일의 위치**는 **`/resources` 하위**에 두면 됨
        - 여러 파일 한번에 지정 가능 (`messages`, `errors`, …)
    - `setDefaultEncoding` : 인코딩 정보 지정
- **Spring Boot**에서의 메시지 소스 설정
    - **자동**으로 `MessageSource`를 **스프링 빈으로 등록** (Spring Boot를 쓰는 이유 중 하나)
    - 추가적인 **basename, encoding 정보**만 `application.properties` 에 설정해주면 됨
    - 하지만 이것 또한 default로 basename = message, encoding=utf-8 로 되어 있음!!
    - 만약 추가적인 메시지 파일을 사용하려면 해당 설정에 추가해주면 됨 → `spring.messages.basename=messages,errors`
    - 국제화 기능은 동일하게 메시지 파일 뒤에 `_en`, `_ko` 처럼 언어 정보를 넣어 주어 생성해주면 됨
- 메시지 파일 예시
    - `/resources/messages.properties`
        
        ```yaml
        hello=안녕
        hello.name=안녕 {0}
        ```
        
        - `{0}` 은 **파라미터**를 넣는 공간
        - 즉, **message는 파라미터도 지원**함
    - `/resources/messages_en.properties`
        
        ```yaml
        hello=hello
        hello.name=hello {0}
        ```
        
        - 영어로 요청이 들어오면 해당 파일의 메시지가 적용 되는 것

### 스프링 메시지 소스 사용

- MessageSource Interface 메서드 살펴보기
    - `String getMessage(String code, @Nullable Object[] args, @Nullable String defaultMessage, Locale locale);`
        - `String code` : message code (messages.properties에 **등록되어 있는 code**)
        - `Object[] args` : message code가 파라미터를 요구할 때 넣어주는 **파라미터 값들**
        - `String defaultMessage` : 해당 되는 message code가 없을 때 대신해서 보여주는 ‘**기본 메시지**’
        - `Locale locale` : **지역 정보** (_en, _ko, …) 등에 맞는 언어 메시지 선택을 위함.
    - `String getMessage(String code, @Nullable Object[] args, Locale locale);`
        - defaultMessage를 잘 사용하지 않기에 defaultMessage 제외된 method
- MessageSource 사용 (위의 `messages.properties` 를 바탕으로)
    - messages.properties 에서 message 불러오기 (args : null, locale : null)
    `ms.getMessage("hello", null, null);` → "안녕”
    - messages.properties 에 없는 message 불러오기(No deafultMessage) -> **예외 발생**
    `ms.getMessage("no_code", null, null))` → `NoSuchMessageException` 발생
    - messages.properties 에 없는 message 불러오기(**기본 메시지**로 출력하기)
    `ms.getMessage("no_code", null,"기본 메시지", null);` → “기본 메시지”
    - message args 와 함께 사용하기
    `ms.getMessage("hello.name", new Object[]{"Spring"}, null);` → “안녕 Spring” ( `hello.name = 안녕 {0}`)
    - locale에 따른 message 불러오기
        - `ms.getMessage("hello", null,` `Locale.*KOREA*)` → "안녕” (_ko 로 설정된 메시지 파일이 없기에 **기본 messages.properties에서** 가져온 것)
        - `ms.getMessage("hello", null, Locale.*ENGLISH` →* “hello” (_en 의 메시지 파일이 있기에 **messages_en.properties에서** 가져온 것)

<aside>
⚠️ <strong>그럼 웹에서 보여줄 때는?</strong> <br>
이렇게 항상 getMessage 를 통해서 웹에다 넘겨줘야 되는 것인가..? 
Nope! <b>Thymeleaf와 Spring가 통합</b>되어 제공하는 기능 중 하나가 메시지, 국제화. 즉, <b>Thymleaf에서</b> 별도의 설정없이 <b>메시지 규칙에 따르면 편리하게 사용가능!!</b>

</aside>

### 웹 애플리케이션에 메시지, 국제화 적용

- **Thymeleaf(View Template)는 Spring과 통합**되어 **메시지, 국제화 기능을 제공**함.
- 즉, 별다른 설정없이 Thymeleaf의 **메시지 사용 규율**만 따르면 편리하게 사용 가능
- **Thymeleaf**에서 **메시지** 사용하기
    - `th:text="#{...}"`
        - messages.properties
        
        ```yaml
        label.item=상품
        label.item.name=상품명
        label.item.price=상품가격
        label.item.param=상품 {0}
        ```
        
        - 메시지 사용 전 : `<h2> 상품명 </h2>`
        - **메시지 사용 후** : `<h2 th:text="#{label.item.name}"> 상품명 </h2>` → 렌더링 → `<h2> 상품명 </h2>`
            - messages.properites에서 label.item.name 이라는 message를 가져와 text content로 사용하는 것!
        - 이렇게 되면 만약 “상품명” 이라는 text를 전체적으로 “상품 이름” 으로 변경하고 싶으면 그냥 **messages.propertes에서 label.item.name 부분만 수정**해주면 됨!! → **유지 보수 효율 UP**
        - **파라미터** **사용**
            - `"#{...(...)}"`
            - `<p th:text="#{label.item.param(${item.param})}"></p>`
- **Thymeleaf**에서 **국제화** 사용하기
    - 사용법은 메시지와 동일!
    - messages_en.properties를 설정해 두었다면 **Thymeleaf가 알아서 자동으로 Accept-Language 헤더의 locale 정보를 통해 국제화를 실행**시켜 줌! (`LocaleResolver` interface의 구현체인 `AcceptHeaderLocaleResolver` 를 통해)
    - Ex)
        - messages_en.properties
        
        ```yaml
        label.item=item
        label.item.name=item name
        label.item.price=item price
        label.item.param=item {0}
        ```
        
        - Accept-Language : **ko**,en;q=0.9,…
            - Local : _ko
            - 하지만 messages_ko.properties가 따로 없기에 **default messages인 messages.properties 사용**
            - `<h2 th:text="#{label.item.name}"> 상품명 </h2>` → 렌더링 → `<h2> 상품명 </h2>`
        - Accept-Language : **en**,ko;q=0.9,…
            - Local : _en
            - **messages_en.properties**가 있기에 해당 메시지 정보를 이용
            - `<h2 th:text="#{label.item.name}"> 상품명 </h2>` → 렌더링 → `<h2> item name </h2>`