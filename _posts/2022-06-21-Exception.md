---
layout: post
title:  "Exception"
date:   2022-06-21
image:  /posts/exception.png
tags:   SpringBoot Servlet Exception
---

## 서블릿 예외처리

> 스프링이 아닌 순수 서블릿 컨테이너의 예외 처리
> 

### 서블릿 예외 처리 기본 흐름

- **서블릿**은 2가지 방식**[`Exception`, `response.sendError()`]** 로 **예외처리** 가능
- `Exception`
    - **In JAVA** : main 이라는 이름의 쓰레드로 실행되며 예외를 따로 잡지(try, catch) 못하여 main() 메서드를 넘어서 예외가 던져지면, **예외 정보를 남기고 쓰레드 종료**
    - **WAS**
        - 사용자 요청별로 별도의 쓰레드가 활성되며 서블릿 컨테이너 안에서 실행됨. 어플리케이션안에서 예외를 잡지 못하고, 서블릿 밖으로 까지 예외가 전달되면 **WAS까지 예외가 전달**됨
        - 예외 전파 흐름 : `WAS <- 필터 <- 서블릿 <- 인터셉터 <- 컨트롤러(예외발생)`
    - WAS의 예외 처리
        - 예외가 발생하여 WAS까지 전파되면 **tomcat이 기본으로 제공하는 오류 화면**을 보여줌 (`HTTP Status 500 – Internal Server Error`)
            
            ![Untitled]({{site.baseurl}}/images/posts/post-220621/Untitled.png)
            
        - WAS에서 발생한 Exception의 경우, **서버 내부에서 처리할 수 없는 오류가 발생한 것으로 생각해서** **항상 HTTP 상태 코드 500**을 반환
        - 지정되어 있지 않은 URL일 경우는 `HTTP Status 404 – Not Found` (서버 내부에서 발생한 Exception이 아니므로 404 반환)
            
            ![Untitled1]({{site.baseurl}}/images/posts/post-220621/Untitled 1.png)
            
- `response.sendError(HTTP status code, error msg)`
    - `HttpServletResponse` 가 제공하는 `sendError` 메서드 (**오류를 끼워서 response를 보내는 것**)
    - 호출하자마자 오류가 처리되는 것은 아니고 **WAS까지 response가 전달 된 후** 해당 error를 띄워주는 것 (즉, **모든 흐름은 정상적일 때와 동일하게 흘러감**)
    - 해당 메서드를 통해 **직접 HTTP 상태 코드와 오류 메시지를 추가**할 수 있음 (Exception과 다른 점)
    - **sendError** 전파 흐름 :  `WAS(response의 sendError 호출 기록 확인) <- 필터 <- 서블릿 <- 인터셉터 <- 컨트롤러(response.sendError() 실행)`
    - 500 에러 코드 발생 → `response.sendError(500, "500 오류!");` (서블릿 기본 500 오류 화면 발생)
    - 404 에러 코드 발생 → `response.sendError(404, "404 오류!");` (서블릿 기본 404 오류 화면 발생)

<aside>
🚨 <strong>개선 점!</strong> <br>
서블릿 컨테이너가 제공하는 기본 예외 처리 화면은 사용자 친화적이지 않음! → <b>의미 있는 오류 화면 제공 필요!!</b>

</aside>

### 오류 화면 제공

- Exception이 발생하여 서블릿 밖으로 전달되거나 response.sendError() 가 호출되었을 때, 각각의 상황에 맞춘 오류처리( **→ 오류 화면 제공**)
- 과거에는 xml 파일을 통해 오류화면을 등록
- 여기서는 **스프링부트가 제공하는 기능을 통해 서블릿 오류 페이지를 등록**
- 서블릿 오류 페이지 등록 (`WebServerFactoryCustomizer` interface 이용)
    - `WebServerFactoryCustomizer<ConfigurableWebServerFactory>` : web 오류에 따른 오류 화면 처리 기능을 이용할 떄 사용하는 설정같은 느낌 (Spring Boot에서 지정한 설정 Interface). **`customize` method overriding 필요**
    
    ```java
    @Component
    public class WebServerCustomizer implements WebServerFactoryCustomizer<ConfigurableWebServerFactory> {
    
        @Override
        public void customize(ConfigurableWebServerFactory factory) {
            ErrorPage errorPage404 = new ErrorPage(HttpStatus.NOT_FOUND, "/error-page/404"); // 오류가 발생하면 path에 Mapping된 Controller를 호출하는 것
            ErrorPage errorPage500 = new ErrorPage(HttpStatus.INTERNAL_SERVER_ERROR, "/error-page/500");
            ErrorPage errorPageEx = new ErrorPage(RuntimeException.class, "/error-page/500"); // runtime 자식 예외까지 처리
            factory.addErrorPages(errorPage404, errorPage500, errorPageEx); // 등록
        }
    }
    ```
    
    - `new ErrorPage(Http Status Code, path)` : **오류 처리 Controller를 호출**하는 것. **path**가 Controller(오류 페이지 View를 연결해주는)에 **매핑된 주소**. 즉, 해당 오류코드가 발생하면 path에 매핑된 Controller가 호출됨.
    - `factory.addErrorPages` : 설정한 오류 처리 class들 추가하여 Spring이 알 수 있도록 해주는 것
    - `Exception`, `response.sendError` 에 따른 호출
        - `response.sendError(404)` : errorPage404 호출 (HttpStatus.NOT_FOUND) → “/error-page/404” path에 매핑된 Controller 호출
        - `response.sendError(500)` : errorPage500 호출 (HttpStatus.INTERNAL_SERVER_ERROR) → “/error-page/500" path에 매핑된 Controller 호출
        - `RuntimeException` (Exception) 또는 그 자식 타입의 예외: errorPageEx 호출 (RuntimeException.class) → “/error-page/500" path에 매핑된 Controller 호출
        
        <aside>
        🚨 **ErrorPage의 path**
        path는 view path (html path)가 아닌, **Controller의 path!**
        즉, 바로 html을 랜더링해줘 보여주는 것이 아닌, **해당 path에 매핑된 Controller를 호출**하는 것!! (오류페이지 작동 원리 part에서 자세히..)
        
        </aside>
        
    - 해당 오류 화면 설정을 처리해주는 Controller (`ErrorPageController`)
        
        ```java
        @RequestMapping("/error-page/404")
        public String errorPage404() {
            return "error-page/404";
        }
        
        @RequestMapping("/error-page/500")
        public String errorPage500() {
            return "error-page/500";
        }
        ```
        
    - 해당 veiw path(resources/templates/error-page/404, …)에 맞는 **custom html을 만들어서 사용자 친화적인 오류 화면을 생성**하면 됨!

### 오류 페이지 작동 원리

- 오류 페이지가 어떤 흐름을 가지고 호출되는지에 대해 알아보기
- 예외, sendError 발생 흐름
    - `Exception` (서버 내부 예외 발생) : `WAS(여기까지 전파) <- 필터 <- 서블릿 <- 인터셉터 <- 컨트롤러(예외발생)`
    - `response.sendError()` : `WAS(sendError 호출 기록 확인) <- 필터 <- 서블릿 <- 인터셉터 <- 컨트롤러(response.sendError())`
    - **둘 다 WAS까지 전달**되는 것을 알 수 있음
    - 즉, 오류 발생 후 **WAS**가 그 **오류를 확인**한 후 **해당 예외를 처리하는 오류 페이지 정보를 확인**(오류 페이지 Config 부분(new ErrorPage())을 통해). 이제 **이 정보를 통해 다시 요청!**
- 오류 페이지 요청 흐름
    - `WAS "/error-page/500(404)" 다시 요청 -> 필터 -> 서블릿 -> 인터셉터 -> 컨트롤러(/error-page/500(404)) -> View`
    - **ErrorPage의 path의 URI**로 다시 요청! → 해당 URI를 가진 Controller가 동작!
    - 즉, **서버 내부**에서 오류 페이지를 찾기 위해 **추가적인 호출**이 일어나는 것! (**서버 외부_Client 는 알 수 없음!)**
- 오류 정보 확인
    - WAS는 오류 페이지를 단순히 다시 요청만 하는 것이 아니라, **오류 정보를 request 의 attribute 에 추가해서 넘겨줌**
    - 해당 정보를 Controller에서 request를 통해 사용할 수 있는 것! (필요하다면)
    - `request.getAttribute(정보 이름)`

<aside>
💡 <strong>예외 처리(오류 화면 제공) 전체 흐름</strong>
<ol>
<li>예외가 발생해서 WAS까지 전파됨 : <code>WAS(여기까지 전파) <- 필터 <- 서블릿 <- 인터셉터 <- 컨트롤러(예외발생)</code> </li>
<li>WAS는 오류 페이지 경로(path)를 찾아서 내부에서 오류 페이지를 호출 (정상 호출과 동일하게 동작) : <code>WAS "/error-page/500(400)" 다시 요청 -> 필터 -> 서블릿 -> 인터셉터 -> 컨트롤러(/errorpage/500(400)) -> View</code></li>
</ol>
</aside>

### 예외 처리 - 필터, 인터셉터

<aside>
⚠️ <strong>예외 처리(오류 화면 제공) 전체 흐름의 옥에티</strong> <br>
예외 처리 전체 흐름을 통해, <b>오류가 발생했을 때의 흐름</b>이 <b>정상호출과 동일</b>한 흐름을 가지는 것을 알 수 있음. 즉, <b>필터와 인터셉터</b> 모두 한번의 클라이언트의 요청에 <b>예외 발생 시 두번 작동</b>하게 됨. (즉, 로그인 인증같은 필터가 적용되어 있다면, 예외가 발생하면 인증이 쓸데없이 두번 동작하는 것!) <br>
-> 서블릿은 이런 것들을 어떻게 처리하는 지 확인해보자!

</aside>

- 서블릿이 제공하는 **`DispatchType`** 을 통해 예외 처리에 따른 필터, 인터셉터 이해하기
- **Exception with Servlet Filter**
    - DispatcherType
        - 필터의 쓸데없는 중복 적용을 막기 위한 기능
        - **요청에 따른 Type을 구분**할 수 있게 해주는 것 (사용자의 처음 요청 → `dispatcherType=REQUEST`, 오류로 인한 서버 내부 요청 → `dispatchType=ERROR`)
        - 즉, 실제 고객이 요청한 것인지, **서버가 내부에서 오류 페이지를 요청하는 것**인지 `DispatcherType` 으로 구분 가능!
        - **Type**
            - **REQUEST** : 클라이언트 요청
            - **ERROR** : 오류 요청
            - **FORWARD** : MVC에서 배웠던 서블릿에서 다른 서블릿이나 JSP를 호출할 때 [RequestDispatcher.forward(request, response);]
            - **INCLUDE** : 서블릿에서 다른 서블릿이나 JSP의 결과를 포함할 때 [RequestDispatcher.include(request, response);]
            - **ASYNC** : 서블릿 비동기 호출
        - 기존의 필터 설정(`WebConfig` 의 `logFilter`)에서 Dispatcher에 대한 설정을 다루면 됨 → `filterRegistrationBean.setDispatcherTypes(DispatcherType.REQUEST, DispatcherType.ERROR);`
            - `setDispatcherTypes`
            - default는 REQUEST. 즉, 기본으로 사용자의 요청에 대해서만 필터가 적용되도록 자동 설정되어 있음
            - 만약, ERROR에만 필터를 적용하고 싶다. 혹은 REQUEST에 더해 ERROR에도 필터를 적용하고 싶다면 해당 기능 이용!!
- **Exception with Spring Interceptor**
    - Interceptor는 Filter와는 다르게 DispatcherType Setting (Type에 따른 Filter 동작) 같은 역할을 하는 기능이 없음!  → **제외 경로 설정(`.excludePathPatterns(...)`)를 통해 해결!**
    - 또한 setDispatcherType 과 같이 default가 없으므로 **꼭 제외 경로에 추가**해줘야됨!
    - `.excludePathPatterns(...)`
        - 제외 요청 경로 설정
        - 여기다 해당 `new ErrorPage(Status, path)` 에서 설정한 **path에 대해서 `excludePathPatterns`에 추가**해주면 됨 → 해당 경로에 대한 요청(예외 처리 요청)에 대해서는 인터셉터가 동작하지 않게 됨!
        - `.excludePathPatterns(..., "/error-page/**");`
            - 예외 처리 요청에 대한 **URI 패턴을 동일하게 설정**해줘야 적용하기 편함 (ex_ `/error-page/**` → /error-page 하위에 대해선 모두 예외처리 요청으로 설정한 것)
- 정리
    - **정상 요청**
        - 오류가 발생하지 않는 “/hello” URI로 요청이 들어온 경우
        - `WAS(/hello, dispatchType=REQUEST) -> 필터 -> 서블릿 -> 인터셉터 -> 컨트롤러 -> View`
        - `dispatchType=REQUEST` 로 들어오고 해당 `URI` 는 exclude에 포함되지 않기 떄문에 필터, 인터셉터 모두 정상 동작
    - **오류 요청**
        - 오류가 발생하는 “/error-ex” URI로 요청이 들어온 경우
        - 흐름
            1. `WAS(/error-ex, dispatchType=REQUEST) -> 필터 -> 서블릿 -> 인터셉터 -> 컨트롤러`
            2. `WAS(여기까지 전파) <- 필터 <- 서블릿 <- 인터셉터 <- 컨트롤러(예외발생)`
            3. `WAS 오류 페이지 확인` ( `new ErrorPage(HTTP Status, path)` 를 통해 )
            4. `WAS(/error-page/500, dispatchType=ERROR) -> 필터(x) -> 서블릿 -> 인터셉터(x) -> 컨트롤러(/error-page/500) -> View`
        - `dispatchType=ERROR` , `/error-page/**` 로 인해 필터, 인터셉터가 모두 한번만 동작하게 됨

## 스프링 예외 처리

<aside>
⚠️ <strong>서블릿의 복잡한 예외 처리 페이지 적용 과정</strong> <br>
<code>WebServerCustomizer(Config)</code> 생성 → 예외 종류에 따른 <code>ErrorPage</code> 추가 → 해당 예외 처리용 <code>Controller</code> 생성 → 각 <code>Path</code>에 맞는 <code>View</code> 연결

</aside>

- 하지만 스프링 부트를 사용하여 예외 처리 페이지를 적용하면 위와 같은 과정을 모두 자동으로 처리해주기 때문에 아주 간단히 처리가 가능!! **(규칙에 따라** **View만 만들어주면 됨!)**

### 오류 페이지 처리 with Spring Boot

- Spring Boot가 자동으로 처리하는 기능들
    - **`ErrorPage` 자동으로 등록** ( “/error” 라는 경로로 기본 오류 페이지를 설정. path = “/error”) →  서블릿 밖으로 **예외가 발생(`Exception`)**하거나, **`response.sendError(...)` 가 호출**되면 모든 오류는 “/error” 라는 path를 가진 Controller 를 호출하게 됨
    - `BasicErrorController` 라는 예외 처리용 Controller 자동 등록 (`ErrorPage`를 통해 등록한 “/error” 를 매핑해서 처리하는 컨트롤러)
- 즉, **개발자는 오류 페이지만 등록** 하면 됨!! (Spring Boot에서 지정한 **경로 규칙을 따라서**)
- 경로 규칙
    - View Template (`resources/templates/error/…`)
        - resources/templates/error/500.html : 500 처리
        - resources/templates/error/5xx.html : 505,503,… 등을 모두 처리
    - 정적 html (`resources/static/error/…`)
        - resources/static/error/400.html : 400 처리
        - resources/static/error/404.html : 404 처리
        - resources/static/error/4xx.html : 405, 440, … 등을 모두 처리
    - 적용 대상이 없을 때
        - resources/templates/error.html
        - 발생된 오류 Status에 대한 html이 설정되어 있지 않을 때
    - 정해진 경로, 정해진 이름 규칙을 따라야 함. 추가로 더 상세하게 설정되어 있으면 그 상세 html이 우선순위를 갖게 됨
    - **해당 규칙에 따라서 html 파일만 만들어주면 알아서 예외 오류 페이지 처리를 해주는 것!**
- 오류 정보 렌더링
    - `BasicErrorController`는 다음 정보를 `Model`에 담아서 `View`에 전달
    - View Template은 이를 이용하여 `View`에 렌더링 가능 (but, 자주 쓰이는 건 아님 → 오류 정보를 사용자에게 보여주는 건 고려해봐야될 상황이므로)
    - **오류 정보**
        - timestamp: 시간 (Fri Feb 05 00:00:00 KST 2021)
        - status: 상태 코드 (400)
        - error: 에러 이름 (Bad Request)
        - exception: 예외 발생 파트 (org.springframework.validation.BindException)
        - trace: 예외 trace
        - message: 예외 메시지 (Validation failed for object='data'. Error count: 1)
        - errors: 예외들 (Errors(BindingResult))
        - path: 클라이언트 요청 경로 (`/hello`)
    - 물론 해당 정보들은 보안상의 문제가 있으므로 `BasicErrorController`에서 다음 오류 정보를 `model` 에 포함할지 여부 선택할 수 있음 (Spring Boot 설정 - application.properties에서 가능!)
    - `application.properties`
        
        ```yaml
        server.error.include-exception=true # 항상
        server.error.include-message=on_param # 쿼리 파라미터로 들어올 때만
        server.error.include-stacktrace=on_param
        server.error.include-binding-errors=on_param
        ```
        
        - true, false
        - never, always, on_param(개발 서버에서 디버그 시 이용 가능)