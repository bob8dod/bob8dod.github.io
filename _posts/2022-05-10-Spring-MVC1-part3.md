---
layout: post
title:  "Spring MVC"
date:   2022-05-10
image:  /posts/spring-mvc.png
tags:   SpringBoot MVC
---

## Spring MVC - 구조 이해

### 스프링 MVC 전체 구조

- Servlet + JSP + MVC 패턴 으로 직접 만든 MVC 프레임워크 구조

    ![]({{site.baseurl}}/images/posts/post-220510/Untitled.png)
- Spring MVC 구조

  ![]({{site.baseurl}}/images/posts/post-220510/Untitled 1.png)
    
- 이름만 다르지 기능과 **구조는 동일**
    - 스프링 MVC도 **프론트 컨트롤러 패턴**으로 구현
    - 스프링 MVC의 프론트 컨트롤러가 바로 **DispatcherServlet**
    - DispatcherServlet은 HttpServlet을 상속 받고 있음
    - 이게 **Spring의 핵심**.
    - 스프링 부트는 DispacherServlet을 서블릿으로 자동 등록하면서 모든 경로 하위에 대해서 매핑. (`@WebServlet(name = "...", urlPatterns = "/")` 으로 등록 되었다고 생각하면 됨) → 더 세세한 경로에 대해서는 우선순위가 밀리기 때문에 우리가 구현한 URL에 대한 Controller는 제대로 작동
    - 결국 얘도 **service로 동작**하는 것 (dispatcher 윗단으로 가면 존재)
    - DispatcherServlet은 결국 doDispatch() method가 가장 중요. 여기서 service에서 진행되었던 것처럼 **핸들러 매핑, 핸들러 어댑터 조회 등 의 로직을 수행**

### 핸들러 매핑과 핸들러 어댑터

- 예전엔 Annotation 기반이 아닌 interface 기반의 Controller를 사용
- 사용 (지금은 사용하지 않는 기법임)
    - Controller 를 구현한 OldController → `public class OldController implements Controller`
    - `@Component("/springmvc/old-controller")` → spring bean의 이름을 url패턴으로 맞춘 것 (for BeanNameUrlHandlerMapping)
    - `public ModelAndView handleRequest(request, response)` 구현 → process 부분이라고 생각하면 됨. 비지니스 로직을 수행하며 request와 response를 이용하고 ModelAndView 객체에 view name을 논리 이름으로 저장하여 반환
- Spring Boot가 자동 등록하는 핸들러 매핑과 핸들러 어댑터
    - **HandlerMapping**
        - 0 순위 ⇒ RequestMappingHandlerMapping : 애노테이션 기반의 컨트롤러인 `@RequestMapping`에서
        사용
        - 1 순위 ⇒ BeanNameUrlHandlerMapping : 스프링 빈의 이름으로 핸들러를 찾는다
    - **HandlerAdaptor**
        - 0 순위 ⇒ RequestMappingHandlerAdapter : 애노테이션 기반의 컨트롤러인 `@RequestMapping`에서 사용
        - 1 순위 ⇒ HttpRequestHandlerAdapter : HttpRequestHandler 처리
        - 2 순위 ⇒ SimpleControllerHandlerAdapter : Controller 인터페이스(애노테이션X, 과거에 사용) 처리
    - `@RequestMapping`
        - 사실 핸들러 매핑과 핸들러 매핑 어댑터는 이 놈밖에 안쓰임.
        - 즉, 우선순위가 가장 높은  `RequestMappingHandlerMapping` ,
        `RequestMappingHandlerAdapter` 만 쓰임.
        - 즉, requestmapping annotation이 달린 놈들에 대해 handlermapping과 adapter가 사용됨. (99.9%)

### 뷰 리졸버 (View Resolver)

- 뷰 리졸버가 동작하게 하기 위해선 application.properties에 추가적인 설정이 필요함. (in JSP)
    
    ```yaml
    spring.mvc.view.prefix = /WEB-INF/views/
    spring.mvc.view.suffix = .jsp
    ```
    
    - 이렇게 설정되면 논리이름만 ModelAndView에 넣어 줘도 **자동으로 물리이름으로 변경**하여 view를 찾아 보냄
- 스프링 부트가 자동 등록하는 뷰 리졸버
    - 1 순위 : BeanNameViewResolver : 빈 이름으로 뷰를 찾아서 반환한다. (예: 엑셀 파일 생성 기능에 사용)
    - 2 순위 : InternalResourceViewResolver : JSP를 처리할 수 있는 뷰를 반환한다.
- BUT, JSP를 제외한 다른 View Template 들은 실제 뷰를 렌더링 함. 즉, forward() 가 필요 없음. (즉, InternalResourceViewResolver 로직을 깊게 이해할 필요는 없음)
- Thymeleaf 뷰 템플릿을 사용하면 ‘ThymeleafViewResolver’를 등록해야 함. 최근에는 자동화돼서 따로 등록 필요 없음

### Spring MVC - 시작하기

- 지금까지 설명한 MVC 패턴의 Spring MVC 적용은 `@RequestMapping` 어노테이션 하나로 끝난다고 생각하면 됨.
- `@RequestMapping` 에 따른 `RequestMappingHandlerMapping` 에 자동 등록 되고, `RequestMappingHandlerAdapter` 또한 자동 등록 됨. 즉, Controller Mapping 정보에 자동 등록 되고, 이를 유연하게 사용하게 해주는 Adapter 또한 자동 등록.
- FormController를 Spring MVC로
    - `public class SpringMemberFormControllerV1`
        - `@Controller` : `@Component + ...`
            - Component (빈 자동 등록)
            - handlermapping 및 handlerAdapter에 등록하겠다는 것. 즉, `RequestMappingHandlerMapping` ,  `RequestMappingHandlerAdapter` 지원 (이 Mapping들에게 등록되고자 하면 Controller 나 Component + ReqeustMapping Annotation을 Class 단에 작성해주면 됨)
        - `public ModelAndView process() { return new ModelAndView("new-form");}`
            - `@RequestMapping("/springmvc/v1/members/new-form")` - 해당 경로로 상세 Mapping 설정, 우선순위를 가짐
            - ModleAndView에 view의 논리이름을 넣음으로써 viewResolver 동작
- SaveController, ListController를 Spring MVC로
    - 로직은 동일
    - @Controller 와 @RequestMapping을 달아주는 것만 다름
    - 즉, 어노테이션 기반으로 동작
- Spring MVC - 컨트롤러 통합
    - `public class SpringMemberControllerV2`
        - `@Controller` : 자동 빈 등록 + Mapping 등록
        - `@RequestMapping("/springmvc/v2/members")` : 가장 상위 URL 설정
        - 각각의 conroller에 대해선 상세 url(requestMapping)을 달아주면 됨 → 그 url에 대해선 우선순위를 가지게 됨
        - 각각의 Controller는 Class 단에서 이루어지는 것이 아닌 mehtod 단에서 이루어지게 됨 → 통합 컨트롤러
        - FromController : `@RequestMapping("/new-form")`
        - SaveController : `@RequestMapping("/save")`
        - ListController : `@RequestMapping()`

### Spring MVC - 실용적인 방식

- Annotation 기반 Controller는 ModelAndView로 반환하는 것도 지원하지만, 굉장히 유연하기에 String 반환도 가능 ⇒ String 으로 반환 되는 값을 viewName으로 인식하고 ViewResolver 수행 → `public String newForm() return “new-form”;`
- 쿼리 파라미터, form 파라미터를 servlet의 request 없이 바로 인자로 받을 수 있음 
→ `@RequestParam(”username”) String username`
    
    → `@RequestParam(”age”) int age` : type 변환까지 자동으로 수행
    ⇒ request.getParam() 과 동일한 역할
    
- 또한 ModelAndView의 Model을 그냥 인자로 사용 가능
→ `Model model`
→ `model.addAttribute(”member”, new Member(username, age);)`
- 요청 Method 구분 가능 (`@RequestMapping` 은 method 구분이 없음, method를 지정할 순 있음 → `@RequestMapping(”…”, method=ReqeustMehtod.GET)`)
→ Get 요청 : `@GetMapping()`
→ Post 요청 : `@PostMapping()`
- 전체 코드 (완벽히 현재 실무에서 사용하는 방식)
    
    ```java
    @Controller
    @RequestMapping("/springmvc/v3/members") // 가장 상위 URL 설정
    public class SpringMemberControllerV3 {
    
        private MemberRepository memberRepository = MemberRepository.getInstance();
    
        // RequestMapping은 method에 상관없이 다 받아냄
        @GetMapping(value = "/new-form")
        public String newForm() {
            return "new-form";
        }
    
        @PostMapping("/save")
        public String save(@RequestParam("username") String username, @RequestParam("age") int age, Model model) {
    
            Member member = new Member(username, age);
            System.out.println("member = " + member);
            memberRepository.save(member);
    
            model.addAttribute("member", member);
            return "save-result";
        }
    
        @GetMapping
        public String memberList(Model model) {
            List<Member> members = memberRepository.findAll();
            model.addAttribute("members", members);
            return "members";
        }
    }
    ```
    

## Spring MVC - 기본 기능

### 프로젝트 생성

![]({{site.baseurl}}/images/posts/post-220510/Untitled 2.png)

- Jar: 항상 내장 서버를 상뇽하고, WEBAPP 경로도 사용하지 않음. 내장 서버 사용에 최적화. 최근에 사용하는 방식
- War: 내장 서버 사용가능, but 주로 외부 서버에 배포하는 목적으로 사용. JSP사용 시 War를 사용해야 됨
- index.html 설정 → Jar 파일을 사용할 경우 ‘/resource/static/index.html’ 위치에 index.html 파일을 두면 main 화면에 index.html이 뜸

### 로깅

- 로깅 라이브러리 (SLF4J, Logback)
- SLF4J는 인터페이스, Logback은 그 구현체 중 하나
- 실무에서는 Logback을 주로 사용
- `@Slf4j` annotation으로 쉽게 사용 가능
    - log. 을 통해 사용 가능 (레벨로 나뉘어져 있음)
    - log.trace() > log.debug() > log,info() > log.warn() > log.error()
    - application.properties를 통해 직접 설정할 수 있음
    → `logging.level.hello.springmvc=trace` : hello.springmvc 하위들은 log를 trace부터 남길 것이다 라는 설정. (debug를 넣어 debug 부터 남기는 것도 설정 가능. 직접 레벨을 정할 수 있음)
    - default는 info. (info 레벨로 부터 남기는 것. `logging.level.root=debug` 로 설정되어 있는 것)
    - 보통 개발 서버는 debug 부터, 운영서버는 info 부터 출력
    
    <aside>
    🚨 <strong>log 안 메세지를 생성할 대, 연산자를 사용해선 안됨</strong><br>
    사용하지 않는 log 까지 연산자가 들어가 있으면 그 연산자가 작동됨! (자바 언어 특징, 연산자 우선 법칙) → 리소스 낭비
    
    </aside>
    
- 로그는 콘솔에만 출력하는 것이 아닌, 파일이나 네트워크 등, 로그를 별도의 위치에 남길 수 있음.

### 요청 매핑

- Class 단 Annotation
    - `@Slf4j` : 로그를 남길 수 있는 Annotation
    - `@RestContoller` : `@Controller` + `@ResponseBody`
        - Controller : Component + HandlerMapping과 HandlerAdaptor에 등록
        - ResponseBody : API로 통신하게끔. → 반환 타입 그대로 반환(viewResolver 작동 X, String 이면 text 그 자체, 그 이외(Object) Json으로)
        - return “ok”; → ok string 그대로 반환됨.
- 요청 매핑 종류
    - 기본 요청 : `@RequestMapping("/hello-basic")`
        - 다중 Mapping(배열 mapping)도 가능 → `@RequestMapping({"/hello-basic", "/hello-temp"})`
    - RequestMapping을 통한 특정 method 사용
        - GET, HEAD, POST, PUT, PATCH, DELETE 사용 가능
        - `@RequestMapping(value = "/mapping-get-v1", method = RequestMethod.*GET*)`
    - 축약 Methond Mapping
        - `@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`, `@PatchMapping`
    - PathVariable 사용 (/edit/{userId})
        - `@GetMapping("/edit/{userId}")` 
        `public String mappingPath(@PathVariable("userId") String userId)`
        - 변수명이 같으면 생략 가능 → `@PathVariable String userId`
    - PathVariable 다중 사용
        - `@GetMapping("/mapping/users/{userId}/orders/{orderId}")`
        `public String mappingPath(@PathVariable String userId, @PathVariable Long orderId)`
    - 파라미터로 추가적인 조건 달기
        - `@GetMapping(value = "/mapping-param", params = "mode=debug")`
        - 파라미터 mode 값이 debug 일때만 호출 가능
        - params=”mode” : mode가 존재할때만 호출 가능
        - params=”!mode” : mode가 없을 때만 호출 가능
        - params = “mode!=debug” : ≠ 파라미터의 값이 debug가 아닐 때만 호출 가능
    - 특정 헤더로 추가 매핑
        - `@GetMapping(value = "/mapping-header", headers = "mode=debug")`
        - 헤더 값이 일치 해야지 호출 가능 (header에 mode라는 항목의 값이 debug 일때만 사용 가능)
        - 지원 항목들은 params와 동일
    - Content-Type 기반 추가 매핑 (Media Type)
        - `@PostMapping(value = "/mapping-consume", consumes = "application/json")`
        - **consumes** : 서버 입장에서 소비하는 입장 (요청할 때) - 헤더의 Content-Type과 mapping (서버 입장에서 받을 수 있는 content-type이라고 생각하면 됨)
        - consumes 안에는 Content-Type 지정 가능
    - Accept 헤더 기반 추가 매핑 (Media Type)
        - `@PostMapping(value = "/mapping-produce", produces = "text/html")`
        - **produces** : 서버가 생산해 내는 입장 (응답할 때) - 헤더의 Accept과 mapping (Client 입장에서 받을 수 있는 content-type 이라고 생각하면 됨)
        - produces 또한 똑같이 Content-Type 지정 가능

### HTTP 요청 - 기본, 헤더 조회

- RequestMapping 으로
    1. `HttpServletRequest requset` : 요청 request
    2. `HttpServletResponse response` : 응답 response
    3. `HttpMethod httpMethod` : http method (GET, PUT, …)
    4. `Locale locale` : 언어 정보
    5. `@RequestHeader MultiValueMap<String, String> headerMap` : 모든 header (MultiValueMap : 하나의 키에 여러 값이 존재하는 Map)
    6. `@RequestHeader("host") String host` : header 하나의 값
    7. `@CookieValue(value = "myCookie", required = false) String cookie` : cookie 값

### HTTP 요청 파라미터 - 쿼리 파라미터, HTML Form

- HTTP 요청 데이터
    - **GET - 쿼리 파라미터**
        - /url?username=hello&age=20
        - 메시지 바디 없이, URL의 쿼리 파라미터에 데이터를 포함해서 전달
        - 예) 검색, 필터, 페이징등에서 많이 사용하는 방식
    - **POST - HTML Form**
        - content-type: application/x-www-form-urlencoded
        메시지 바디에 쿼리 파리미터 형식으로 전달 username=hello&age=20
        - 예) 회원 가입, 상품 주문, HTML Form 사용
    - HTTP message body에 데이터를 직접 담아서 요청
        - HTTP API에서 주로 사용, JSON, XML, TEXT
        - 데이터 형식은 주로 JSON 사용
        - POST, PUT, PATCH
    - GET 쿼리 파리미터 전송 방식이든, POST HTML Form 전송 방식이든 둘다 형식이 같으므로 구분없이 조회할 수 있음 (request parameter 조회로 가능)
- 요청 파라미터 받기 (RequestParam, GET 쿼리 파라미터 전송, POST HTML Form 전송)
    - Servlet 이용 (request)
    - `@RequestMapping("/request-param-v1")public void requestParamV1(HttpServletRequest request, HttpServletResponse response)`
    - `request.getParameter(”username”); …` 을 통해 요청 파라미터를 받아올 수 있음
- HTTP 요청 파라미터 - @RequestParam
    - servlet의 request를 사용하지 않고 Spring 에서 제공 하는 `@RequestParam`을 사용해서 더 편리하게 쿼리 파라미터(GET, POST Form Data) 받아 오기 (기본)
        
        ```java
        @RequestMapping("/request-param-v2")
        public String requestParamV2(
                @RequestParam("username") String username,
                @RequestParam("age") int age) {...}
        ```
        
        - 변수 이름과 받아오는 parameter의 이름이 동일하다면 parameter 이름 생략 가능 → `@RequestParam String username, @RequestParam int age`
        - 동시에 `@RequestParam` 도 생략 가능. But Annotation을 달아주지 않으면 인지하기에 모호한 경우가 발생할 수도 있음. 웬만하면 써주기
    - 파라미터를 필수로 설정하기 (default는 사실 필수)
        
        ```java
        @RequestMapping("/request-param-v5")
        public String requestParamV5(
                @RequestParam(required = true) String username, // required=true : default
                @RequestParam(required = false) Integer age) {...}
        ```
        
        - 만약 username이 주어지지 않는다면 404에러 발생, age가 주어지지 않는다면 null값 부여
        - `reuired = true` 가 default. 즉, 설정해주지 않으면 항상 필수로 받게 됨
        
        <aside>
        🚨 <strong>주의!</strong><br>
        <ul>
        <li>int 는 null을 받을 수 없기에 Integer로 설정하여 null로 설정할 수 있도록 세팅</li>
        <li>파라미터 이름만 사용하는 경우 (/request-param?username=) username이 빈문자로 통과됨</li>
        </ul>
        </aside>
        
    - 기본 값 적용
        
        ```java
        @RequestMapping("/request-param-default")
        public String requestParamDefault(
        	 @RequestParam(required = true, defaultValue = "guest") String username,
        	 @RequestParam(required = false, defaultValue = "-1") int age) {...)
        ```
        
        - 만약 파라미터 값이 입력되지 않았다면 defaultVlaue로 설정 된 값으로 세팅됨 (이미 기본 값이 있기 때문에 required는 의미 없음)
        - 위와 동일하게 파라미터 이름만 사용하는 경우 (/request-param?username=) username이 빈문자로 통과됨 주의
    - Map 으로 한번에 받기
        
        ```java
        @RequestMapping("/request-param-map")
        public String requestParamMap(@RequestParam Map<String, Object> paramMap) {
            log.info("username: {}, age: {}", paramMap.get("username"), paramMap.get("age"));
            return "ok";
        }
        ```
        
        - Map 으로 한번에 받아오고, get을 통해 원하는 parameter를 조회 할 수 있음
        - 또한, MultiValueMap ( -> key1=[value1, value2, ..,], key2=...) 를 통해 같은 파라미터에 대한 여러 값을 받아올 수도 있음. 일반 Map 을 사용하거나 RequestParam을 사용하게 되면 첫번째값만 조회 가능
- HTTP 요청 파라미터 - @ModelAttribute
    - `@ModelAttribute`
        - 요청 파라미터를 받아서 필요한 객체를 만들고 그 **객체에 값을 넣어주는 것을 자동으로** 해주는 것
        - `@ModelAttribute`가 자동으로 해주는 것
            
            ```java
            HelloData helloData = new HelloData():
            helloData.setUsername(username);
            helloData.setAge(age);
            //-> What modelAttribute actioned
            ```
            
        - 사용
            
            ```java
            @RequestMapping("/model-attribute-v1")
            public String modelAttributeV1(@ModelAttribute HelloData helloData) {
                log.info("username={}, age={}", helloData.getUsername(), helloData.getAge());
                return "ok";
            }
            ```
            
            - HelloData
                - `@Data` : @Getter , @Setter , @ToString , @EqualsAndHashCode , @RequiredArgsConstructor 자동 적용 annotation
                - String username, int age
            - 생략 가능
                - 생략 후 인식방법 (생략 시 규칙을 적용)
                    - String, int 같은 단순 타입 = @RequestParam
                    - argument resolver 로 지정해둔 타입 외 = @ModelAttribute
                - But 모호함을 피하기 위해 생략하지 않는 것을 추천
            
            <aside>
            🚨 <strong>주의!</strong> <br>
            <ul>
            <li>age = ‘abc’ 처럼 숫자가 들어가야 하는 곳에 문자가 넣어지게 되면 Java 자체에서 BindException 발생.</li>
            <li>ModelAttribute는 기본생성자를 통해 작동함. 즉, 기본생성자가 없으면 오류 발생!</li>
            </ul>
            </aside>
            

### HTTP 요청 메시지 - 단순 텍스트

- HTTP 메시지 바디에 직접 데이터가 담겨 오는 경우 (요청 파라미터와 다른 방식), **`@RequestParam`, `@ModelAttribute` 는 사용할 수 없음 (Form 제외)**
- Servlet 의 reqeust를 사용하여 조회 → `public void requestBodyStringV1(HttpServletRequest request, ...) throws IOException`
    - 이전과 동일하게 `request.getInputStream()` 과 `StreamUtils.copyToString()`을 통해 body에 담겨진 text를 가져 올 수 있음
- **InputStream** 사용 → `public void requestBodyStringV2(InputStream inputStream, Writer responseWriter) throws...`
    - 받아올 때 부터 request가 아닌 inputStream으로 받아와 사용 가능
    - 똑같이 `StreamUtils.copyToString()` 을 통해 String으로 변환은 해줘야 됨
- **HttpEntity** 사용 → `public HttpEntity<String> requestBodyStringV3(HttpEntity<String> httpEntity)`
    - HttpEntity를 통해 HttpMessageConverter 기반으로 받아온 데이터가 text라면 이를 자동으로 String 으로 변환해줌.
    - 스프링 MVC 내부에서 HTTP 메시지 바디를 읽어서 문자나 객체로 변환해서 전달. 이때 HTTP 메시지 컨버터(HttpMessageConverter)라는 기능을 사용
    - 즉, `StreamUtils.copyToString()`가 따로 필요 없음. (자동으로 변환해주기 때문)
- **`@RequestBody` 사용 (실무에서 사용)** → `public String requestBodyStringV4(@RequestBody String messageBody)`
    - HttpEntity의 역할을 하지만 훨씬 쉽게 이용할 수 있도록 annotation 기반으로 만든 것.
    - 똑같이 응답도 `@ResponseBody`를 사용하면 됨
    - 얘 또한 HTTP 메시지 컨버터(HttpMessageConverter) 이용
    - 만약 여기서 헤더정보도 필요하다면 HttpEntity를 사용하거나 `@RequestHeader` 를 추가로 사용해주면 됨

### HTTP 요청 메시지 - JSON

- {”username”:”hello”, “age”:20} 이라는 Json 데이터가 Http Body에 담겨오는 상황 (content-type : application/json)
- **ObjectMapper**를 사용하여 Json 받기
    - Servlet의 reqeust
        - 이전과 동일하게 request의 input stream을 받아와 string으로 변환하고 `objectMapper.readValue(messageBody, HelloData.class)` 를 통해 Json string을 HelloData 객체에 Mapping 시켜줌
    - @RequestBody
        - @RequestBody (with HttpMessageConverter) 를 통해 바로 String을 받아오고 `objectMapper.readValue(messageBody, HelloData.class)` 사용
- ObjectMapper 없이 **`@RequestBody**` 만을 사용하기 (with **HttpMessageConverter**)
    - `@RequestBody` 를 통해 바로 HelloData에 Mapping 시킬 수 잇음 → HttpMessageConverter 덕분
    - `public String requestBodyJsonV3(@RequestBody HelloData helloData)`,  `helloData.getUsername(), helloData.getAge()`
    - 또한 Response도 `@ResponseBody`를 이용하여 변환없이 바로 Json으로 보낼 수 있음
    - `@RequestBody` 는 생략하면 안됨! → 생략하면 Spring MVC 법칙에 따라 객체이기에 `@ModelAttribute`로 인식하게 됨
- `HttpEntity<HelloData>` 로도 사용 가능 (`httpEntity.getBody()`)

### HTTP 응답 - 정적 리소스, 뷰 템플릿

> HTTP 응답 에는 정적 리소스(정적인 HTML, css, js, 파일), 뷰 템플릿(동적인 HTML), HTTP 메시지(HTTP API(text, JSON, …)) 가 사용됨


- 정적 리소스
    - 정적 리소스는 해당 파일을 변경 없이 그대로 서비스하는 것
    - 정적 리소스 경로 : src/main/resources/static
    - 즉, 정적 리소스는 이 하위로 경로가 할당 되게 됨
        - ex) src/main/resources/static/basic/hello-form.html → http://localhost:8080/basic/hello-form.html
- 뷰 템플릿
    - 뷰 템플릿을 거쳐서 HTML이 생성되고, 뷰가 응답을 만들어서 전달
    - 뷰 템플릿 경로 : src/main/resources/templates
    - ex) src/main/resources/templates/response/hello.html → Controller속 return “response/hello” 로 Mapping 할 수 있음 (`@ResponseBody` 없이!)
    - 추가로 Controller의 Url Mapping과 해당 html의 경로가 일치하다면 return 없이 Mapping 가능 (추천X, 모호함)
    
    <aside>
    🚨 <strong>JSP에서 처럼 prefix랑 suffix 설정(View Resolver)은 없어도 되나?</strong><br>
    thymeleaf를 땡겨오게 되면 자동으로 설정됨.<br>
    → prefix = classpath:/templates/ <br>
    → suffix = .html
    
    </aside>
    
- HTTP 응답 - HTTP API, 메시지 바디에 직접 입력
    - Text
        - ResponseEntity<String> (HttpEntity) 사용 → `return new ResponseEntity<>("ok", HttpStatus.*OK*);`
        - `@ResponseBody` 사용 (with HttpMessageConverter) → `return "ok";` (만약 상태코드를 쓰고 싶다면 `@ResponseSatus` 추가 이용,동적인 상태 변화를 원한다면 조건에 따른 ResponseEntity를 사용해야 됨)
    - JSON (Entity)
        - ResponseEntity<HelloData> (HttpEntity) 사용 → `return new ResponseEntity<>(helloData, HttpStatus.*OK*);`
        - `@ResponseBody` 사용 (with HttpMessageConverter) → `return helloData;` (만약 상태코드를 쓰고 싶다면 `@ResponseSatus` 추가 이용, 동적인 상태 변화를 원한다면 조건에 따른 ResponseEntity를 사용해야 됨)
    - 추가로 각각의 method에 ResponseBody를 다는 것이 아닌 Class 단에 ResponseBody를 달게 되면 통합적으로 적용 됨. 또한 이를 Controller annotation과 통합 가능 → `@RestController` ⇒ `@Controller + @ResponseBody`

### HTTP 메시지 컨버터

- 뷰 템플릿으로 HTML 을 넘기는 것이 아닌 데이터를 직접 넘기거나 받아 올 때, `input stream`으로 받아와 `String`, `Object`로의 변환을 따로 처리해주는 것이 아닌, `@RequestBody`, `@ResponseBody`를 사용해 **자동으로 처리**해 줄 때 쓰이는 것이 `HttpMessageConverter`.
- 즉, 데이터를 바로 넘겨줄 때는 `viewResolver` 대신 `HttpMessageConverter` 가 쓰이는 것
- 대상 **클래스 타입**과 **미디어 타입** 둘을 체크해서 어떤 converter를 사용할지 결정
    - 요청의 경우 Content-type 과 서버의 컨트롤러 인자 타입 정보 둘을 조합해서 HttpMessageConverter가 선택됨
    - 응답의 경우 클라이언트의 HTTP Accept 헤더와 서버의 컨트롤러 반환 타입 정보 둘을 조합해서 HttpMessageConver 가 선택됨
- HttpMessageConverter는 Interface, 이를 구현 한 것이
**ByteArrayHttpMessageConverter** : byte[] 데이터를 처리
**StringHttpMessageConverter** : 기본 문자처리
**MappingJackson2HttpMessageConverter** : 기본 객체처리
- 주요 메시지 컨버터
    - **ByteArrayHttpMessageConverter** : byte[] 데이터를 처리
        - 클래스 타입: byte[] , 미디어타입: **/**
        - 요청 예) `@RequestBody byte[] data`
        - 응답 예) `@ResponseBody` `return byte[]` 쓰기 미디어타입 application/octet-stream
    - **StringHttpMessageConverter** : String 문자로 데이터를 처리한다.
        - 클래스 타입: String , 미디어타입: **/**
        - 요청 예) `@RequestBody String data`
        - 응답 예) `@ResponseBody` `return "ok"` 쓰기 미디어타입 text/plain
    - **MappingJackson2HttpMessageConverter** : application/json
        - 클래스 타입: 객체 또는 HashMap , 미디어타입 application/json 관련
        - 요청 예) `@RequestBody` `HelloData data`
        - 응답 예) `@ResponseBody` `return helloData` 쓰기 미디어타입 application/json 관련
- HTTP 요청 데이터 읽기
    - HTTP 요청이 오고, 컨트롤러에서 `@RequestBody` , `HttpEntity` 파라미터를 사용
    - 메시지 컨버터가 메시지를 읽을 수 있는지 확인하기 위해 canRead() 를 호출
        - 대상 **클래스 타입**을 지원하는가?
        예) `@RequestBody` 의 대상 클래스 ( byte[] , String , HelloData )
        - HTTP 요청의 **Content-Type 미디어 타입**을 지원하는가?
        예) text/plain , application/json ,**/**
    - canRead() 조건을 만족하면 read() 를 호출해서 객체 생성하고, 반환.
- HTTP 응답 데이터 생성
    - 컨트롤러에서 `@ResponseBody` , `HttpEntity` 로 값이 반환.
    - 메시지 컨버터가 메시지를 쓸 수 있는지 확인하기 위해 canWrite() 를 호출.
        - 대상 **클래스 타입**을 지원하는가?
        예) return의 대상 클래스 ( byte[] , String , HelloData )
        - HTTP 요청의 **Accept 미디어 타입**을 지원하는가? (더 정확히는 @RequestMapping 의 produces )
        예) text/plain , application/json , **/**
    - canWrite() 조건을 만족하면 write() 를 호출해서 HTTP 응답 메시지 바디에 데이터를 생성.

### 요청 매핑 핸들러 어뎁터 구조 (RequestMappingHandlerAdapter)

![]({{site.baseurl}}/images/posts/post-220510/Untitled 3.png)

- **ArgumentResolver**
    - 우리가 사용하고 있는 **Controller**들은 각각의 **다양한 파라미터들**을 사용하고 있음. ArgumentResolver는 이런 각각의 Controller들의 파라미터를 먼저 확인하고 **그 파라미터에 해당하는 것들을 준비해 주는 역할**을 가지고 있음. 이를 통해 어떤 Controller 든 **유연**하게 처리할 수 있는 것.
    - 즉, 애노테이션 기반 컨트롤러를 처리하는 **RequestMappingHandlerAdaptor** 는 바로 이
    **ArgumentResolver** 를 호출해서 컨트롤러(핸들러)가 필요로 하는 다양한 파라미터의 값(객체)을 생성한 후에 Controller를 호출하면서 값을 넘겨주는 것
- **ReturnValueHandler**
    - **ArgumentResolver와 같은 역할**을 컨트롤러의 반환 시에 실행 되는 것. 즉, Converting 작업 실시
- **HTTP 메시지 컨버터**

  ![]({{site.baseurl}}/images/posts/post-220510/Untitled 4.png)
    
    - Http 메시지 컨버터는 ArgumentResolver 와 ReturnValueHandler에서 사용 , ( Http 메시지 컨버터가 사용되는 `@RequestBody` 는 컨트롤러가 필요로 하는 파라미터의 값에 사용, `@ResponseBody` 또한 컨트롤러의 반환 값 이용 )
    - 요청 : `@RequestBody` 등을 처리할 때, **ArgumentResolver**는 **HTTP 메시지 컨버터**를 사용해서 현재 상황에 맞는 필요한 객체를 생성하는 것
    - 응답 : `@ResponseBody` 등을 처리할 때, **ReturnValueHandler**는 **HTTP 메시지 컨버터**를 사용해서 현재 상황에 맞는 응답 결과를 만드는 것