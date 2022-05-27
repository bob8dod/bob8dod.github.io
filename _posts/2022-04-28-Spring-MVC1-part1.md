---
layout: post
title:  "Spring MVC #1 part1 "
date:   2022-04-28
image:  /posts/springMVC.jpg
tags:   SpringBoot MVC
---


## Chapter 1 [웹 애플리케이션 이해]

### 웹 서버, 웹 애플리케이션 서버

- 웹 서버 (Web Server)
    - HTTP 기반으로 동작
    - 정적 리소스 제공, 기타 부가 기능
    - 정적 HTML, CSS, JS, 이미지, 영상 등을 HTTP 응답으로 제공
- 웹 애플리케이션 서버 (WAS - Web Application Server)
    - HTTP 기반으로 동작
    - **웹 서버 기능** 포함 + 정적 리소스 제공
    - **프로그램 코드를 실행해서 애플리케이션 로직 수행**
        - 동적 HTML, HTTP API (JSON, XML, ..)
        - related with Servlet, JSP, Thymeleaf, 스프링 MVC
    - Tomcat, Jetty, Undertow
- WEB Server VS WAS
    - 결국 웹 서버는 정적 리소스(파일)을 실행하는 서버, **WAS는 애플리케이션 로직까지 실행하는 서버**
        - 보통 웹 시스템은 정적 리소스를 처리하는 Web Server + 애플리케이션 로직같은 동적인 처리를 하는 WAS + 데이터를 이용하는 DB 로 구성됨
        - 이런 식으로 구성하게 되면 잘 죽는 WAS를 대신해서 오류화면을 정적인 Web Server로 보여줄 수 있게 됨.
        

### **서블릿 (Servlet)**

- HTTP 요청과 응답의 모든 과정에서 비지니스 로직을 제외한 **모든 과정을 자동으로 처리**해줌

  ![]({{site.baseurl}}/images/posts/post-220428/Untitled 0.png)
    
- HTTP 스펙을 매우 편리하게 사용 가능하게 해줌
    - HTTP 요청 정보를 편리하게 사용할 수 있도록 `HttpServletRequest` 제공
    - HTTP 응답 정보를 편리하게 제공할 수 있도록 `HttpServletResponse` 제공
- HTTP 요청, 응답 흐름

  ![]({{site.baseurl}}/images/posts/post-220428/Untitled 1.png)
    
    (**request, response**는 요청이 들어올 때마다 다 다른 객체, but **helloServlet**이라는 서블릿 객체는 동일한 객체(**싱글톤**))
    
    - WAS는 Request, Response 객체를 새로 만들어서 서블릿 객체 호출
    - 개발자는 Request 객체에서 HTTP 요청 정보를 편리하게 꺼내서 사용
    - 개발자는 Response 객체에 HTTP 응답 정보를 편리하게 입력
    - WAS는 Response 객체에 담겨있는 내용으로 HTTP 응답 정보를 생성
- **Servlet Container**
    - 이 안에서 서블릿 객체를 자동으로 생성 및 호출해줌
    - WAS에 따른 생명주기 관리 (WAS가 시작될 때 생성하고, WAS가 종료될 때 같이 종료)
    - 서블릿 객체는 **싱글톤**으로 관리
        - 즉, 모든 요청은 동일한 서블릿 객체 인스턴스에 접근
    - 동시 요청을 위한 **멀티 쓰레드 처리 지원**
    
- **멀티 쓰레드 (Multi Thread, 동시 요청)**
    - 쓰레드 (Thread)
        - **Process** : 프로그램을 실행하는 것 ↔ **Thread** : 프로그램 안에서 실행하는 것
        - 애플리케이션 코드를 하나하나 순차적으로 실행하는 것은 쓰레드
        - 자바 메인 메서드를 처음 실행하면 main이라는 이름의 쓰레드가 실행
        - 쓰레드가 없다면 자바 애플리케이션 실행이 불가능
        - 쓰레드는 한번에 하나의 코드 라인만 수행
        - 동시 처리가 필요하면 쓰레드를 추가로 생성
    - 요청 마다 Thread를 생성한다면?
        - 장점
            - 동시 요청을 처리할 수 있다.
            - 리소스(CPU, 메모리)가 허용할 때 까지 처리가능
            - 하나의 쓰레드가 지연 되어도, 나머지 쓰레드는 정상 동작한다.
        - 단점
            - 쓰레드는 생성 비용은 매우 비싸다.
            - 고객의 요청이 올 때 마다 쓰레드를 생성하면, 응답 속도가 늦어진다.
            - 쓰레드는 컨텍스트 스위칭(하나에서 다른 thread로 이동하는) 비용이 발생한다.
            - 쓰레드 생성에 제한이 없다. 즉, 고객 요청이 너무 많이 오면, CPU, 메모리 임계점을 넘어서 서버가 죽을 수 있다.
        - 그래서 우린 **Thread Pool**을 이용해야 됨.
    - **쓰레드 풀 (Thread Pool)**

      ![]({{site.baseurl}}/images/posts/post-220428/Untitled 2.png)
        
        - 특징
            - 필요한 쓰레드를 쓰레드 풀에 보관하고 관리한다.
            - 쓰레드 풀에 생성 가능한 쓰레드의 최대치를 관리한다. 톰캣은 최대 200개 기본 설정 (변경 가능)
        - 사용
            - 쓰레드가 필요하면, 이미 생성되어 있는 쓰레드를 쓰레드 풀에서 꺼내서 사용한다.
            - 사용을 종료하면 쓰레드 풀에 해당 쓰레드를 반납한다.
            - 최대 쓰레드가 모두 사용중이어서 쓰레드 풀에 쓰레드가 없으면? → 기다리는 요청은 거절하거나 특정 숫자만큼만 대기하도록 설정할 수 있다.
        - 장점
            - 쓰레드가 미리 생성되어 있으므로, 쓰레드를 생성하고 종료하는 비용(CPU)이 절약되고, 응답 시간이 빠르다.
            - 생성 가능한 쓰레드의 최대치가 있으므로 너무 많은 요청이 들어와도 기존 요청은 안전하게 처리할 수 있다.
        - WAS의 멀티 쓰레드 지원
            - 멀티 쓰레드에 대한 부분은 **WAS가 처리**
            - 개발자가 멀티 쓰레드 관련 코드를 신경쓰지 않아도 됨
            - 개발자는 마치 싱글 쓰레드 프로그래밍을 하듯이 편리하게 소스 코드를 개발
            - 멀티 쓰레드 환경이므로 싱글톤 객체(서블릿, 스프링 빈)는 주의해서 사용

### HTML, HTTP API, CSR, SSR

- HTML
    - 정적 리소스 (Web Server) VS **WAS(Web Application Server)**
        - 정적 리소스

          ![]({{site.baseurl}}/images/posts/post-220428/Untitled 3.png)
            
            고전된 HTML 파일, CSS, JS, 이미지, 영상 등을 제공
            
        - **WAS**

          ![]({{site.baseurl}}/images/posts/post-220428/Untitled 4.png)
            
            **동적으로 필요한 HTML 파일**을 생성해서 전달  
            
- HTTP API

  ![]({{site.baseurl}}/images/posts/post-220428/Untitled 5.png)
    
    - HTML이 아닌 **데이터(JSON, XML)**를 전달
    - 데이터만 주고 받음. 즉, UI 화면이 필요하면, 클라이언트 쪽에서 별도로 처리
    - 다양한 시스템 연동
        - 서버 to 서버
        - 앱 클라이언트 (아이폰, 안드로이드, PC 앱)
        - 웹 브라우저에서 자바스크립트를 통한 HTTP API 호출
        - React, Vue.js 같은 웹 클라이언트
- **SSR - 서버 사이드 렌더링**

  ![]({{site.baseurl}}/images/posts/post-220428/Untitled 6.png)
    
    - 동적으로 **HTML 최종 결과를 서버에서 만들**어서 웹 브라우저에 전달
    - 주로 정적인 화면에 사용
    - 관련기술: JSP, 타임리프 -> 백엔드 개발자
- **CSR - 클라이언트 사이드 렌더링**

  ![]({{site.baseurl}}/images/posts/post-220428/Untitled 7.png)
    
    - HTML 결과를 자바스크립트를 사용해 웹 브라우저에서 동적으로 생성해서 적용 (서버에서 생성된 HTML 결과를 받아오는 것이 아닌)
    - 주로 동적인 화면에 사용, 웹 환경을 마치 앱 처럼 필요한 부분부분 변경할 수 있음
    - 예) 구글 지도, Gmail, 구글 캘린더
    - 관련기술: React, Vue.js -> 웹 프론트엔드 개발자
    

## Chapter 2 [서블릿]

### 프로젝트 생성

![]({{site.baseurl}}/images/posts/post-220428/Untitled 8.png)

- JSP 사용을 위해 War 파일로 선택
- Spring boot는 사용하지 않지만 (servlet만을 이용) 내장 서버 이용을 위해 Spring Boot로 생성

### Servlet 사용하기

- `@ServletComponentScan`
    - Spring에서 Servlet을 사용하기 위함. 서블릿 자동 등록
    - main application 에 해당 어노테이션을 달아주게 되면 이 하위 패키지에서 servlet을 사용할 수 있도록 해줌
- `@WebServlet(name="helloServlet", urlPatterns = “/hello”)`
    - 해당 class에서 servlet을 사용하겠다는 것.
    - 이름은 `helloServlet` , 요청은 hello url로 연결
    - 해당 class는 `extends HttpServlet`, HttpServlet을 상속받아야 함 → servlet을 사용하기 위함.
    - 그리고 해당 상속받은 객체의 service 메서드를 overriding. → `protected void service(HttpServletRequest request, HttpServletResponse response)`
        - HTTP 요청을 통해 매핑된 URL이 호출되면 서블릿 컨테이너는 `service`메서드를 실행
        - /hello 라는 url로 접속하게 되면 이 service가 작동하게 되는 것
        - http 요청이 오면 WAS 가 `request`, `response` 객체를 만들어서 servlet 에 던져줌
        - `request` 를 통해 요청정보를 얻어 올 수 있음
            - ! request는 요청이 오고 요청에 대한 응답이 나갈 때까지 생존 주기를 가짐.
            - `request.getParameter("username");` → 쿼리 파라미터(/hello?username=kim) 받기
            - 만약 http 스펙에 있는 파라미터를 다 파싱해서 읽는다고 하면 너무 복잡함. but servlet은 이걸 편하게 도움
        - `response` 를 통해 응답 메세지를 작성하고 보낼 수 있음
            
            ```java
            response.setContentType("text/plain"); // 단순 문자 -> header 정보에 들어감
            response.setCharacterEncoding("utf-8"); // 문자 인코딩 정보 -> header 정보에 들어감
            response.getWriter().write("hello" + username); // http Body에 들어가게 됨. 즉, 정보를 넘겨주게 되는 것
            ```
            
    - HTTP 요청의 모든 정보(url, header, body, … ) 보기 → application.properties 에서 `logging.level.org.apache.coyote.http11=debug` 추가
- WAS 요청 및 응답 구조

  ![]({{site.baseurl}}/images/posts/post-220428/Untitled 9.png)
    
    - 가장 먼저 spring boot가 내장 톰켓 서버를 통해 서블릿 컨테이너 안에 servlet 객체 (싱글톤)를 생성. (호출, 삭제 까지 담당)
    - 그 후 요청이 들어오면 그 요청 당 하나의 request 객체와 response 객체를 생성하고 helloServlet의 service 메서드에 넣어주며 호출
    - 그 후 request를 이용하여 필요한 비지니스 로직을 수행하고 response객체에 그 로직의 결과 정보를 담아 요청자에게 반환

### HttpServletRequest

- 개념
    - 개발자가 HTTP 요청을 직접 파싱하지 않도록 **자동으로 파싱**해주는 것이 바로 서블릿, 해당 파싱 결과의 요청(request) 부분이 **HttpServletRequest**
    - Start Line (Http 메소드, URL, 쿼리 파라미터, 스키마, 프로토콜), Header(헤더), Body(form 파라미터 형식, message body, 데이터)를 편리하게 조회 가능하게 함
    - request의 생존 기간(Http 요청에 대한 응답이 나가기 전까지) 동안 **임시 저장소로써 사용** 가능
        - 저장 : `request.setAttribute(name, value);`
        - 조회 : `request.getAttribute(name);`
    - 세션 관리도 가능
        - `request.getSession(create: true);`
- 사용 (Start Line, Header 정보 조회)
    - if) GET 요청 URL 은 http://localhost:8080/request-header?username=hello
    - Start Line 정보 조회
        - 메서드 조회 : `request.getMethod()` ⇒ GET
        - 프로토콜 조회 : `request.getProtocal()` ⇒ HTTP/1.1
        - 스키마 조회 : `request.getScheme()` ⇒ http
        - URL 조회 : `request.getRequestURL()` ⇒ http://localhost:8080/request-header
        - URI 조회 : `request.getRequestURI()` ⇒ /request-header
        - 쿼리 파라미터 조회 : `request.getQueryString()` ⇒ username=hello
        - 보안 조회 : `request.isSecure()` ⇒ false
    - Header 정보 조회
        - server name : request.getServerName() ⇒ localhost
        - server port : request.getServerPort() ⇒ 8080
        - 이 뿐만 아니라 더 다양한 헤더 정보 조회 가능
- 사용 (HTTP 요청 데이터 조회)
    - **클라이언트에서 서버로 데이터를 전달하는 방법**
        1. GET - 쿼리 파라미터
            - /url?username=hello&age=20
            - 메시지 바디 없이, URL의 **쿼리 파라미터**에 데이터를 포함해서 전달
            - 예) 검색, 필터, 페이징등에서 많이 사용하는 방식
        2. POST - HTML Form
            - content-type: application/x-www-form-urlencoded
            - **메시지 바디에 쿼리 파리미터 형식**으로 전달 username=hello&age=20 (그래서 데이터를 조회하는 방식은 쿼리 파라미터와 동일)
            - 예) 회원 가입, 상품 주문, HTML Form 사용
        3. HTTP message body에 데이터를 직접 담아서 요청
            - **HTTP API**에서 주로 사용, **JSON**, XML, TEXT 등
            - POST, PUT, PATCH 방식 사용
    - “GET - 쿼리 파라미터” 조회
        - 가정
            - 전달 데이터가 “username = hello, age = 20” 이라면
            - ?username=hello&age=20 형식으로 url을 요청
        - 조회
        - 단일 파라미터 조회 →  `request.getParameter(”username”)`, `request.getParameter(”age”)`
        - 모든 파라미터 조회 → `request.getParameterNames().asIterator().forEachRemaining(paramName -> System.out.println(paramName + " = " + request.getParameter(paramName)));`
        - 이름이 같은 복수 파라미터 조회 (?username=kim&username=park) → `request.getParameterValues("username");` 값이 배열로 저장됨. 즉, 모두 가져올 수 있음. 여기서 `request.getParameter`를 사용하면 가장 첫번째 값만 가져오게 됨.
    - POST - HTML Form
        - 가정
            - content-type: application/x-www-form-urlencoded
            - message body: username=hello&age=20
        - 값이 보내지는 형식이 쿼리 파라미터와 동일하기 때문에 GET - 쿼리 파라미터 조회와 동일하게 `request.getParameter`로 값을 조회할 수 있음
        - 즉, request.getParameter() 는 GET URL 쿼리 파라미터 형식, POST HTML Form형식, 둘 다 지원.
        - 물론 form은 message body로 넘겨지므로 body 자체로 접근해서 값을 파싱할 수 있음. but 이미 request.getParameter로 충분
        
        <aside>
        💡 <strong>그럼 GET - URL 쿼리 파라미터랑 POST HTML form 형식은 차이가 없는 것인가?</strong><br>
        Nope! GET URL 쿼리 파라미터 형식으로 클라이언트에서 서버로 데이터를 전달할 때는 HTTP 메시지 바디를 사용하지 않기 때문에 content-type이 없지만, POST HTML Form 형식으로 데이터를 전달하면 HTTP 메시지 바디에 해당 데이터를 포함해서 보내기 때문에 바디에 포함된 데이터가 어떤 형식인지 content-type (application/x-www-form-urlencoded )을 꼭 지정해야 한다. [content-type은 HTTP 메시지 바디의 데이터 형식을 지정]
        
        </aside>
        
    - HTTP message body 에 데이터를 직점 담아서 요청
        - API 메시지 바디 - 단순 텍스트 (`Content-Type : text/plain`)
            
            ```java
            ServletInputStream inputStream = request.getInputStream(); // HTTP message body의 내용을 byte 코드로 받음
            String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8); // 인코딩 설정 (byte to String)
            ```
            
        - API 메시지 바디 - JSON
            - content-type: application/json
            - message body: {"username": "hello", "age": 20}
            - 단순 텍스트에서 진행했던 것과 같이 파싱하면 됨 → Json도 차피 똑같은 문자이기에.
            - 즉, content type : json으로 설정하면 text로 받아오지만 text를 json 형태로 파싱할 수 있는 로직을 진행할 수 있게 되는 것.
            - 이제 파싱한 해당 messge body를 기존에 설정한 Data에 맞게 파싱 진행 → `ObjectMapper` Class 사용 (JSON 결과를 파싱해서 사용할 수 있는 자바 객체로 변환 → `objectMapper.readValue()`)
            - `HelloData helloData = objectMapper.readValue(messageBody, HelloData.class);`
            - 결과 : HelloData → username = hello, age = 20 (Hellodata의 속성들은 json의 변수들과 이름이 같아야 됨)

### HttpServletResponse

- HTTP 응답 메시지 생성
    - HTTP 응답코드 지정, 헤더 생성, 바디 생성
    - Content-Type, 쿠키, Redirect 설정 가능
- Status Line
    - 상태 설정 : `response.setStatus(HttpServletResponse.*SC_OK*);`
- Header
    - content-type 설정 : `response.setHeader("Content-Type", "text/plain;charset=utf-8");`
    - 쿠키 설정 : `response.setHeader("Cache-Control", "no-cache, no-store, mustrevalidate");`, `response.setHeader("Pragma", "no-cache");`
    - 사용자 정의 header 설정 : `response.setHeader("my-header","hello");`
    - redirect 설정 : `response.sendRedirect("/basic/hello-form.html");`
- Body
    - 단순 text : `response.getWriter().println(”ok”);`
    - HTML
        - Content-Type : text/html;charset=utf-8
        
        ```java
        response.setContentType("text/html"); // html로 반환
        response.setCharacterEncoding("utf-8"); // encoding 정보
        
        PrintWriter writer = response.getWriter();
        writer.println("<html>");
        writer.println("<body>");
        writer.println(" <div>안녕?</div>");
        writer.println("</body>");
        writer.println("</html>"); // 직접 작성해 줘야 됨
        ```
        
    - API JSON
        - Content-Type: application/json
        - `ObjectMapper` Class 사용 (자바 객체를 JSON으로 변환해서 사용할 수 있도록) → `objectMapper.writeValueAsString()`
        
        ```java
        response.setContentType("application/json");
        response.setCharacterEncoding("utf-8");
        
        HelloData helloData = new HelloData();
        helloData.setUsername("kim");
        helloData.setAge(20);
        
        String result = objectMapper.writeValueAsString(helloData); // json string으로 작성
        response.getWriter().write(result); // 이 자체를 보내줌
        ```