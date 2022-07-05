---
layout: post
title:  "Exception in API"
date:   2022-06-24
image:  /posts/exception_API.png
tags:   SpringBoot Exception API
---

## API 예외 처리 소개

### API 예외 처리의 필요성

- HTML은 간단하게 스프링 부트에서 제공하는 예외 처리를 이용하면 되었음
- 그럼 그대로 API에서 발생한 예외도 HTML의 예외 처리 결과를 보내주면 될까? → NOPE!! **API의 경우 생각할 내용이 더 많음!!** (오류페이지는 단순히 오류 화면만 보여주면 끝)
- API는 각 **오류 상황에 맞는 오류 응답 스펙을 정해서** 보여줘야 됨. 또한 HTML이 아닌 **JSON으로 데이터를 보내줘야됨!**

### API 예외 처리 - 기본

- `WebServerCustomizer`  를 통해 **직접 ErrorPage를 등록**하고 해당 path(URI)를 통해 Controller를 호출하고 **JSON으로 데이터를 반환**해줄 수 있도록 설정
- 스프링 부트의 예외 페이지 처리가 아닌 **서블릿의 예외 처리** (직접 설정하여 진행하는) 이용!
- `WebServerCustomizer` : Exception 파트 참고 (Exception)[링크]
- `ApiExceptionController`
    - ErrorPage의 path(URI)에 따라 JSON 데이터를 반환할 수 있도록 설정한 Controller
    
    ```java
    @RestController // @ResponseBody + @Controller
    public class ApiExceptionController {
    
        @GetMapping("/api/members/{id}")
        public MemberDto getMember(@PathVariable("id") String id) {
            if (id.equals("ex")) {
                throw new RuntimeException("잘못된 사용자");
    				}
    
            return new MemberDto(id, "hello" + id);
        }
    }
    ```
    
    - 정상 호출 시 MemberDto가 Json으로 잘 반환되는 것을 확인할 수 있음
    - 예외 발생 호출 시 기존에 Error처리를 했던 Controller(`ErrorPageController`)에서 반환하는 HTML파일을 받음! → 해결 필요(웹페이지가 아니면 HTML을 직접 받아서 할 수 있는 것이 거의 없음)
- `ErrorPageController` 에 **API 응답 추가!**
    - ViewResolver를 통해 HTML을 반환했던 것에 추가로 **API로도 응답할 수 있도록** 설정 → **`accept=application/json` 에 해당하는 HandlerMethod 추가!**
    
    ```java
    @RequestMapping(value = "/error-page/500", produces = MediaType.APPLICATION_JSON_VALUE)
    public ResponseEntity<Map<String, Object>> errorPage500Api(
            HttpServletRequest request, HttpServletResponse response) {
    
        Map<String, Object> result = new HashMap<>();
        Exception ex = (Exception) request.getAttribute(ERROR_EXCEPTION);
        result.put("status", request.getAttribute(ERROR_STATUS_CODE));
        result.put("message", ex.getMessage());
    
        Integer statusCode = (Integer) request.getAttribute(RequestDispatcher.ERROR_STATUS_CODE);
    
        return new ResponseEntity<>(result, HttpStatus.valueOf(statusCode));
    }
    ```
    
    - `@RequestMapping(value = "/error-page/500", produces = MediaType.APPLICATION_JSON_VALUE)`
        - `produces = MediaType.APPLICATION_JSON_VALUE` 를 통해 **Accept 이 application/json 인 요청에** JSON으로 데이터를 반환할 수 있는 Controller 생성
        - "/error-page/500" 에 매핑되는 Controller도 있지만  accept = application.json 일 경우 해당 **url("/error-page/500")의 HandlerMethod 대해 우선순위를 가짐**. 즉, HTML로 반환되는 Controller가 아닌 JSON으로 반환되는 Controller로 매핑되는 것!)
    - `return new ResponseEntity<>(result, HttpStatus.valueOf(statusCode));`
        - result `Map` 을 만들고 status , message 키에 오류 정보를 담은 값을 할당
        - `ResponseEntity` 를 사용해서 응답하기 때문에 `HttpMessageConverter`가 동작하면서 클라이언트에 **JSON이 반환**됨!
- 결론적으로 Accpet 가 application/json 이 아닌 요청에 대해선 HTML이 반환되지만, **Accept가 application/json 의 요청에 대해선 JSON으로 오류 처리가 진행**됨!

### 스프링 부트 기본 API 오류 처리

- 오류 화면 처리와 마찬가지로 API 예외 처리도 **스프링 부트가 제공하는 기본 오류 방식** 사용 가능 → `BasicErrorController`
- `BasicErrorController` (`@RequsetMapping(”/error”)`)
    
    ```java
    @RequestMapping(produces = MediaType.TEXT_HTML_VALUE)
    public ModelAndView errorHtml(HttpServletRequest request, HttpServletResponse response) {}
    
    @RequestMapping
    public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {}
    ```
    
    - `“/error”` 를 처리하는 `errorHtml()`, `error()` 두 메서드 존재
    - `errorHtml()` [**produces = MediaType.TEXT_HTML_VALUE**] : 요청의 Accept 헤더 값이 text/html 인 경우(**동일한 url에 대해 우선순위**를 가짐)에는 errorHtml()을 호출하여 ModelAndView를 반환하여 view를 제공! → html화면 제공
    - `error()` : 그 외의 경우에 호출(우선순위 하위 → 상세한 설정이 없으므로). `ResponseEntity`로 JSON 데이터 반환
- API 오류 메시지 결과 (JSON)
    
    ```json
    {
    	 "timestamp": "2021-04-28T00:00:00.000+00:00",
    	 "status": 500,
    	 "error": "Internal Server Error",
    	 "exception": "java.lang.RuntimeException",
    	 "trace": "java.lang.RuntimeException: 잘못된 사용자\n\tat hello.exception.web.api.ApiExceptionController.getMember(ApiExceptionController.java:19...,
    	 "message": "잘못된 사용자",
    	 "path": "/api/members/ex"
    }
    ```
    
    - HTML에서 오류 메시지 정보를 추가적으로 확인하기 위해 했던 설정 또한 API에서도 동일하게 적용 가능 → `server.error.include-message=always, server.error.include-stacktrace=always, …`

<aside>
⚠️ <strong>HTML 오류 페이지 vs API 오류 JSON</strong> <br>
<code>BasicErrorController</code> 를 이용하면 두 항목 모두 쉽게 오류처리가 가능! 하지만, <b>API는</b> 앞서 설명했던 것과 같이 <b>매 상황마다 보내줘야 하는 JSON의 스펙이 다름!</b> 즉, 공통적으로 처리할 수가 없다는 것. 그래서 <b><code>BasicErrorController</code>를 이용한 기본 오류 처리는 HTML만</b> 이용함. <b>API는 <code>@ExceptionHandler</code>를 이용!!</b>

</aside>

<aside>
🚨 <strong>즉, 이 스펙에 따라 달라져야 하는 복잡한 API는 추가적인 처리가 필요하다는 뜻!</strong>

</aside>

## API 예외 처리 실전

### HandlerExceptionResolver

- 사용 목적 : 현재 예외(`Exception`)가 발생하여 서블릿을 넘어 **WAS까지 예외가 전달**되면 HTTP Status는 무조건 어떤 예외든 **500으로 처리**가 됨. 하지만, 발생하는 예외에 따라 400,404 등 **다른 상태코드도 처리**하고 싶고 **요구하는 스펙에 따라** 메시지, 형식등을 **API마다 다르게 처리**하고 싶을 때 → `HandlerExceptionResolver`
- `HandlerExceptionResolver`
    - **Spring MVC**가 제공하는 기능
    - 컨트롤러 밖으로 예외가 던져진 경우 **예외를 해결하고, 동작을 새로 정의**할 수 있는 방법 (기존 동작은 WAS까지 갔다가 예외처리로 넘어가는 흐름을 가진 동작. 이를 벗어나 동작을 새로 정의할 수 있다는 것)
    - `HandlerExceptionResolver` 적용 전
        
        ![Untitled]({{site.baseurl}}/images/posts/post-220624/Untitled.png)
        
        - 컨트롤러에서 예외가 발생하고 그에 따라 예외가 WAS까지 전달되고 그 이후로는 예외처리 흐름에 따라 `BasicController`를 호출하게 됨
    - `HandlerExceptionResolver` 적용 후
        
        ![Untitled]({{site.baseurl}}/images/posts/post-220624/Untitled 1.png)
        
        - 컨트롤러에서 예외가 발생하고 해당 **예외가 `ExceptionResolver`로 넘어가게 됨**. 이제 여기서 WAS까지 예외가 전달되는 것이 아닌 `ExceptionResolver`가 **정의한 동작을 수행**(예를 들면 sendError를 동작하게 하는,,**)**하게 되고 **그 결과가 WAS로 넘어가게 됨.** 즉, **WAS로는 정상 응답이 넘어**가게 됨. (**마치 정상적인 동작이 이루어진 것 처럼!**)
        - 하지만, 컨트롤러에서 예외가 발생한 것 마찬가지 이므로 `postHandle()` 은 호출되지 않음
    - `HandlerExceptionResolver` Interface
        
        ```java
        public interface HandlerExceptionResolver {
        	 ModelAndView resolveException(
        				HttpServletRequest request, HttpServletResponse response,
        				Object handler, Exception ex);
        }
        ```
        
        - `Object handler` : 컨트롤러 정보
        - `Exception ex` : 컨트롤러에서 발생한 예외
- **예외에 따른 상태코드 변환**
    - ExceptionResolver를 사용하지 않으면 **서버에서 발생한 Exception은 무조건 500**으로 나감
    - 이 Exception을 경우에 따라 API 스펙 변환보다는 우선적으로 **상태코드를 400,404 로 변환**하고 싶은 상황
    - 해당 상황에 맞게 `HandlerExceptionResolver` 를 구현한 ExceptionResolver를 만들고 WebConfig에 등록!
    - `MyHandlerExceptionResolver`
        
        ```java
        @Slf4j
        public class MyHandlerExceptionResolver implements HandlerExceptionResolver {
            @Override
            public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
                try {
                    if (ex instanceof IllegalArgumentException) { // Illegal.. 이 발생하면 직접 컨트롤 하겠다는 것
                        log.info("IllegalArgumentException resovler to 400");
                        response.sendError(HttpServletResponse.SC_BAD_REQUEST, ex.getMessage()); // 기존에 제공하는 500이 아닌, 400으로 변환해서 response에 error를 보내주는 것
                        return new ModelAndView(); // 빈 모델,View를 보내주면 알아서 그냥 return 되어 직접 설정한 Error만을 response하게 됨
                    } // 즉, 실제 발생한 예외를 먹어버리고 내가 만든 예외를 발동시키는 것
                } catch (IOException e) {
                    log.error("resolver ex", e);
                }
                return null; // 위의 로직이 처리되지 않으면 Exception이 그대로 WAS까지 날라감 (기존에 발생한 예외를 서블릿 밖으로 던진다!)
            }
        }
        ```
        
        - 앞서 봤던 흐름과 같이, 컨트롤러에서 Exception이 발생하면 **그 Exception을 해당 `ExceptionResovler`가 받게 됨**
        - `response.sendError()` 를 통해서 예외(Exception)처리 흐름을 **정상흐름으로 변경** (Exception을 잡아서 **내가 원하는 동작으로 처리**해주는 역할) → `IllegalArgumentException` 이 발생하면 `response.sendError(400)` 를 호출해서 HTTP 상태 코드를 400으로 지정하고, 빈 ModelAndView 를 반환
            
            <aside>
            ⚠️ <strong>Exception 발생 vs response.sendError() 추가</strong>
            <ul>
            <li>결론적인 오류에 대한 처리이기에 Exception 과 response.sendError는 똑같다고 생각할 수 있지만, <b>엄연히 다른 것!</b></li>
            <li><code>Exception</code>이 발생하면 정상흐름으로 가지 않고 <b>예외가 발생했다는 흐름으로 WAS가 받게</b> 됨</li>
            <li><code>response.sendError()</code> 는 response에다가 Error를 추가하는 것이므로 <b>WAS까지는 정상흐름으로 가게 되는 것!</b></li>
            <li>즉, 각각에 <b>흐름 자체가 달라짐!</b> (Exception은 ExceptionResolver가 발동, postHandler는 동작하지 않음. <code>response.sendError()</code>는 ExceptionResolver가 발동되지 않고 정상흐름이기에 postHandler 동작) </li>
            <li>결정적으로 <b><code>response.sendError()</code> 는 내가 원하는 Error로 custom이 가능!</b> (status 설정 등)</li>
            </ul>
            </aside>
            
        - 빈 모델,View를 보내주면 알아서 그냥 return 되어 직접 설정한 Error만을 response하게 됨. (ModelAndView 값이 없기에 렌더링 되지 않고 **해당 response가 WAS까지 전달**됨)
        - 만약 다른 Exception이 터지면 null을 반환함으로써 **기존 예외처리 흐름**을 가지게 함 (기존에 발생한 예외를 **서블릿 밖으로 던진다!** → ExceptionHandler 적용 전 예외처리 흐름)
            
            <aside>
            💡 <strong>ExceptionResolver의 반환 값에 따른 DispatcherServlet 동작 방식</strong>
            <ul>
            <li><strong>빈 ModelAndView</strong>: <code>new ModelAndView()</code> 처럼 빈 ModelAndView 를 반환하면 뷰를 렌더링 하지 않고, <b>정상 흐름</b>으로 서블릿이 리턴됨 </li>
            <li><strong>ModelAndView 지정</strong>: ModelAndView 에 View , Model 등의 정보를 지정해서 반환하면 <b>뷰를 렌더링</b>함 </li>
            <li><strong>null</strong>: null 을 반환하면, <b>다음 ExceptionResolver 를 찾아서 실행</b>. 만약 처리할 수 있는 ExceptionResolver 가 없으면 예외 처리가 안되고, 기존에 발생한 <b>예외를 서블릿 밖으로 던짐</b> </li>
            </ul>
            </aside>
            
    - `WebConfig`
        - 구현된 HandlerException을 등록
        
        ```java
        @Override
        public void extendHandlerExceptionResolvers(List<HandlerExceptionResolver> resolvers) {
        		resolvers.add(new MyHandlerExceptionResolver());
        }
        ```
        
- ExceptionResolver 활용
    - **예외 상태 코드 변환**
        - 발생된 예외를 잡아서 response.sendError() 호출로 **자신이 원하는 상태코드로 변경**하여 처리 (빈 ModelAndView(`return new ModelAndView()`)를 반환하는 상황)
        - WAS는 `response.sendError()` 를 받아서 해당 상태코드에 맞는 오류 처리 진행 → `BasicErrorController`호출 ****(html 렌더링, API JSON 제공)
    - **뷰 템플릿 처리**
        - ModelAndView에 값을 넣어 **새로운 오류 화면 뷰 렌더링**
    - **API 응답 처리**
        - 요구되는 스펙에 맞게 객체를 생성하고 객체를 JSON으로 변경한 후
        - `response.sendError()` 가 아닌 `response.getWriter().println(JSON)`처럼 **응답 바디에 직접 데이터를 넣어주는 것**

### HandlerExceptionResolver 활용

- 예외를 ExceptionResolver에서 끝내기!
    - 지금까지 살펴본 것(Exception 발생 or ExceptionResolver에서 response.sendError() 추가)의 예외 발생 흐름을 보면 예외 발생 시 WAS까지 예외가 전달되고, **WAS에서 다시 그 예외에 대한 정보를 통해 “/error”를 호출하는 과정**은 생각해보면 너무 복잡함!
    - ExceptionResolver를 활용해서 직접 상태코드를 변경하고 뷰 템플릿을 처리하거나 API 응답에 대해 JSON으로 데이터를 보낼 수 있었음 즉, **ExceptionResolver를 활용하면 ExceptionResolver 그 자체에서 예외에 대한 처리를 끝낼 수 있음!**
- ExceptionResolver에서 끝내기
    - custom Exception UserException 추가 → `public class UserException extends RuntimeException`
    - UserException 을 처리하는 `UserHandlerExceptionResolver` 생성 및 추가하여 ExceptionResolver 안에서 **해당 예외에 대한 처리를 끝내버리기** (즉, 예외에 대한 처리가 WAS까지 가지 않는 것 → **뷰 렌더링 해버리거나 JSON 데이터를 response에 담는 것**)
    - `UserHandlerExceptionResolver`
        
        ```java
        @Slf4j
        public class UserHanlderExceptionResolver implements HandlerExceptionResolver {
        
            private ObjectMapper objectMapper = new ObjectMapper();
        
            @Override
            public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
        
                try {
                    if (ex instanceof UserException) {
                        ...
                        // json 용 에러 반환
                        if ("application/json".equals(acceptHeader)) {
                            ... // Map 생성
                            response.setContentType("application/json");
                            response.setCharacterEncoding("utf-8");
                            response.getWriter().write(result); // Error가 발생했을 때, Was로 갔다가 다시 error에 대한 Controller 호출하는 것이 아니라.
                            // 이 자체에서 write을 통해 끝내버리는 것! (효율적)
                            return new ModelAndView(); // 아무것도 반환하지 않게 하여 그냥 response로 동작하게 설정
                        } else {
                            // TEXT/HTML
                            return new ModelAndView("error/500"); // accept가 html일 때의 경우 그냥 View Rendering
                            // 이 자체에서 ModelAndView에 view를 넣어주면서 끝내버리는 것! (효율적)
        								...
                }
                return null;
            }
        }
        ```
        
        - **요청 헤더의 Aceept에 따라** 반환되는 결과물을 **지정**해 줘야됨
        1. 요청 헤더의 **Accept이 application/json** 인 경우
            - **JSON으로 보내줄 데이터**를 Map(`errorResult`)으로 만들어서 오류 정보를 생성 → `String result = objectMapper.writeValueAsString(errorResult);`
            - 해당 데이터에 대한 설정 후 **response에 결과물을 작성**해줌 → `response.getWriter().write(result);` , `return new ModelAndView();` ⇒ **이 자체에서 write을 통해 끝내버리는 것!**
        2. 요청 헤더의 **Accept이 text/html** 인 경우
            - `return new ModelAndView("error/500");` → accept가 html일 때의 경우 그냥 View Rendering하여 ModelAndView가 동작하여 끝내버림
    - `WebConfig` 를 통해 `ExceptionResolver` 등록
        
        ```java
        @Override
        public void extendHandlerExceptionResolvers(List<HandlerExceptionResolver> resolvers) {
        	 resolvers.add(new MyHandlerExceptionResolver());
        	 resolvers.add(new UserHandlerExceptionResolver());
        }
        ```
        
- **결론**
    - ExceptionResolver 를 사용하면 컨트롤러에서 예외가 발생해도 ExceptionResolver 에서 예외를 처리.  따라서 예외가 발생해도 서블릿 컨테이너까지 예외가 전달되지 않고, 스프링 MVC에서 예외 처리가 끝남
    - 즉, 예외가 WAS 까지 갔다가 다시 WAS가 그에 따른 오류처리 과정이 다시 진행되는 비효율적인 흐름을 방지하는 것! → 효율적
    - 결론적으로 WAS입장에서는 정상처리로 진행된 것.
    - 이렇게 Exception을 ExceptionResolver에서 모두 처리할 수 있다는 것이 핵심
    - 근데 이렇게 매번 모든 Exception에 대해 ExceptionResolver를 정의하고 해당 처리에 대한 구현을 진행하기에는 코드가 너무 복잡함! (객체를 Json으로 바꾸고… 하는 과정들)
    - 스프링은 이런 과정을 자동으로 처리하게끔 기능을 지원함! ⇒ `@ExceptionHandler` (**ExceptionHandlerExceptionResolver**)

## 스프링이 제공하는 ExceptionResolver

- 스프링 부트가 **기본으로 제공하는 ExceptionResolver (우선순위 순으로)**
    1. **ExceptionHandlerExceptionResolver** : **`@ExceptionHandler`** 을 처리. API 예외 처리는 대부분 해당 어노테이션으로 해결 가능!
    2. ResponseStatusExceptionResolver : **HTTP 상태 코드 지정** 가능
    3. DefaultHandlerExceptionResolver : **스프링 내부 기본 예외**를 처리

### ResponseStatusExceptionResolver

- 예외에 따라서 **HTTP 상태 코드를 지정**해주는 역할
- `@ResponseStatus` 가 달려있는 custom Exception, `ResponseStatusException` 을 통한 기본 Exception, 의 두가지 경우 처리
- `@ResponseStatus`
    - HTTP 상태 코드를 변경 (500 → ? )
    - `BadRequestException` 이라는 custom Exception 에 `@ResponseStatus` 를 통해 원하는 상태 코드로 변경
        
        ```java
        @ResponseStatus(code = HttpStatus.BAD_REQUEST, reason = "잘못된 요청 오류")
        		public class BadRequestException extends RuntimeException {
        }
        ```
        
        - 500 status 를 400 status로 변경
    - `BadRequestException` 가 Controller 에서 발생하여 ExceptionResolver 까지 넘어가게 되면 ExceptinoResolver 중 `ResponseStatusExceptionResolver` 가 **해당 예외의 애노테이션을 확인**해서 **오류 상태코드를 400 으로 변경하고 메시지도 담음**
    - 해당 `ExceptionResolver`는 사실 **`response.sendError(statusCode, resolvedReason)`를 통해 상태코드를 변경하는 것**을 확인할 수 있음
    - 즉, `sendError(400)` 를 호출했기 때문에 WAS에서 다시 오류 페이지 ” /error “ 를 내부에서 요청하는 것!!
    - 추가로 reason 에 message source를 이용하여 메시지를 전달할 수도 있음 → `reason = "error.bad"` (messages.properties 의 error.bad 메시지 값을 가져옴)
- `ResponseStatusException`
    - `@ResponseStatus` 는 **개발자가 직접 변경할 수 없는 예외에는 적용할 수 없음**. (애노테이션을 직접 다는 것이기에 Custom Exception, 변경 가능한 Exception 만 적용 가능). 또한 **동적으로 변경하는 것도 어려움**
    - 이럴 때는 `ResponseStatusException` 을 사용
    - `ApiExceptionController`
        
        ```java
        @GetMapping("/api/response-status-ex2")
        public String responseStatusEx2() {
         throw new ResponseStatusException(
        			HttpStatus.NOT_FOUND, "error.bad", 
        			new IllegalArgumentException() );
        }
        ```
        
        - `IllegalArgumentException` 에 대해 404로 상태코드를 변경하고 원하는 메시지도 넣어줌

### DefaultHandlerExceptionResolver

- 스프링 내부에서 발생하는, 스프링 예외를 해결
- 대표적인 것이 바로 `TypeMismatchException`
    - 파라미터 바인딩 시점에 타입이 맞지 않으면 발생
    - ExceptionResolver 없이 예외가 처리될 경우 **서블릿 컨테이너까지 오류가 올라**가고, **결과적으로 500 오류가 발생**
    - 하지만 `TypeMismatchException` 는 서버 오류라기 보다는 클라이언트가 요청 정보를 잘못 호출해서 발생하는 문제. **즉, 400으로 상태코드가 나가는게 옳음!**
- `DefaultHandlerExceptionResolver` 는 이것을 500 오류가 아니라 **HTTP 상태 코드 400 오류로 변경**
- 해당 오류뿐만 아니라 굉장히 **다양한 Exceptioin들에 대해서 스프링이 해당 Exception에 맞는 상태코드로 변경해서 나갈 수 있도록 자동으로 설정**해 둠. (`response.sendError()` 를 통해 → WAS에서 다시 오류 페이지 ”/error“ 를 내부 요청하는 것)

<aside>
💡 <strong>지금까지의 ExceptionResolver</strong>
<ol>
<li>ResponseStatusExceptionResolver : HTTP 응답 코드 변경 가능 (Custom Exception → <code>@ResponseStatus</code>, 기존 Exception → <code>ResponseStatusException</code>)</li>
<li>DefaultHandlerExceptionResolver : 스프링 내부 예외 처리 (기본적으로 발생할 수 있는 Exception들에 대해 자동으로 응답 코드를 변경해서 보내줌)</li>
<li>HandlerExceptionResolver를 직접 구현하여 사용 → 실용적으로 사용하기에는 복잡! (API 오류 응답의 경우 response 에 직접 데이터를 넣어야 함) → 스프링이 제공하는 <b>ExceptionHandlerExceptionResolver</b>를 통해 간편하게 처리 가능!  ⇒ <code>@ExceptionHandler</code> (<b>ExceptionHandlerExceptionResolver</b>)</li>
</ol>
</aside>

### @ExceptionHandler (ExceptionHandlerExceptionResolver)

- **ExceptionHandlerExceptionResolver**를 사용하지 않을 때
    - **API 오류 처리를 위해 스프링이 제공하는 ExceptinoResolver**
    - API 오류는 해당 상황에 따른 **스펙에 맞게 오류 처리 데이터**를 보내줘야함.
    - 이를 하기 위해선 우린 HandlerExceptionResolver를 **직접 구현해 봤지만**, 직접 JSON 데이터를 생성하고 response에 담아주는 등, **과정이 복잡하고 효율적이지 않았음**.
    - 추가로 HandlerExceptionResolver는 **필수적으로 ModelAndView를 반환**해야 했음. → API에는 필요 없는 것
    - 또한, 같은 Exception이라도 각각의 **상황에 따라 반환되어야 하는 스펙이 다름!** → 구현이 더욱 더 복잡해짐
- **`@ExceptionHandler`**
    - 스프링은 **API 예외 처리 문제를 해결**하기 위해 `@ExceptionHandler` 라는 애노테이션을 사용하는 매우 편리한 예외 처리 기능을 제공. 이게 바로 **ExceptionHandlerExceptionResolver**
    - ExceptionResolver 중에서 **우선순위가 가장 높은** Resolver!
    - 실무에서 API 예외 처리는 **대부분 이 기능을 사용**함
    - **각각의 컨트롤러 만의 예외처리가 가능**하기 때문에 API 오류처리에 효율적임!
- `@ExceptionHandler` 사용
    - 응답 객체 (ErorrResult)
        
        ```java
        @Data
        @AllArgsConstructor
        public class ErrorResult {
        	 private String code;
        	 private String message;
        }
        ```
        
    - `@ExceptionHandler` (Controller 안에)
        - 해당 컨트롤러에서 **예외가 발생하면 얘가 잡아서 이 로직을 수행**함.
        - 즉, 이 어노테이션 하나로 custom ExceptionHandler를 설정하는 것
        - 해당 예외가 발생하면 **먼저 ExceptionHandler에게 해당 예외를 처리하라고 요청**
        - 그럼 ExceptionHandler에게 **우선순위의 Handler들을 통해 예외 처리**를 시도
        - 여기서 `@ExceptionHandler`가 **우선순위**에 있기 때문에 얘가 수행 되는 것 (해당 Controller에 있다면)
        - 근데 여기서 Handle을 해줬기 때문에 **정상흐름으로 인식** → 즉, **따로 설정하지 않으면 status가 200이 반환**됨 ⇒ **상태코드 변경 필요**!
        - **다른 컨트롤러 Method와 똑같이 동작**함. 즉, 컨트롤러가 하는 **역할과 파라미터, 어노테이션, 반환 값 사용 가능**!
        - 참고로 Controller 처럼 작동하기 때문에 String을 반환할 경우 ViewResolver가 동작하여 HTML로도 응답할 수 있음 (But 잘 사용 안함) → **`@ExceptionHandler` 는 보통 API 오류 처리용**
        - + `@ResponseSatus()`
            
            ```java
            @ResponseStatus(HttpStatus.BAD_REQUEST)
            @ExceptionHandler(IllegalArgumentException.class)
            public ErrorResult illegalExHandle(IllegalArgumentException e) {
            	 log.error("[exceptionHandle] ex", e);
            	 return new ErrorResult("BAD", e.getMessage());
            }
            ```
            
            - `@ResponseSatus` 를 통해 **상태코드 변경** (200 → 오류코드) → (`@ResponseSatus`는 원래 Controller에 붙일 수 있는 놈)
            - 해당 `IllegalArgumentException` 포함 하위 Exception에 대해선 이 ExceptionHandler가 동작
            - 컨트롤러와 같이 동작하기 때문에 **ResponseBody가 적용 되어 MessageConverter가 동작!** → JSON으로 데이터가 반환됨
        - + `ResponseEntity<>`
            
            ```java
            @ExceptionHandler // 생략하면 자동으로 인자에 있는 Exception 할당
            public ResponseEntity<ErrorResult> userExHandler(UserException e) {
                log.error("[exceptionHandler] ex", e);
                ErrorResult errorResult = new ErrorResult("USER-EX", e.getMessage());
                return new ResponseEntity<>(errorResult, HttpStatus.BAD_REQUEST);
            } // @ExceptionHandler 기존 Controller와 같이 정상 흐름으로 작동하기에
             // 기존 Controller와 같이 사용할 수 있음 -> ResponseEntity를 이용하여 status 설정정
            ```
            
            - `ResponseEntity<>` 를 통해 **JSON을 반환하며 상태코드를 변경** (200 → 오류코드)
            - 해당 `UserException`  포함 하위 Exception에 대해선 이 ExceptionHandler가 동작
            - Method의 파라미터가 `@ExceptionHandler`가 다루는 Exception과 동일하다면 생략 가능
        - 다양한 예외 공통 처리
            
            ```java
            @ExceptionHandler({AException.class, BException.class})
            public String ex(Exception e) {
            	 log.info("exception e", e);
            }
            ```
            
            - 두가지 예외에 대해서도 같이 처리 가능
            - 두 예외의 부모로 인자를 받으면 됨

<aside>
💡  <strong>Exception 발생 시 <code>@ExceptionHandler</code> 실행 흐름</strong>
<ul>
<li>컨트롤러를 호출한 결과 Exception이 컨트롤러 밖으로 던져짐</li>
<li>예외가 발생했으로 ExceptionResolver가 작동. ExceptionResolver 중 가장 <b>우선순위가 높은 ExceptionHandlerExceptionResolver 가 실행</b>됨 </li>
<li>ExceptionHandlerExceptionResolver 는 해당 컨트롤러에 <b>해당 Exception을 처리할 수 있는 <code>@ExceptionHandler</code> 가 있는 지 확인</b> </li>
<li>해당 <code>@ExceptionHandler</code> 의 Method 실행. <code>@RestController</code> 의 영향을 받아 <code>@ResponseBody</code> 가 적용(혹은 <code>ResponseEntity</code> 적용) → HttpMessageConverter 적용 → JSON으로 반환 </li>
<li><code>@ResponseStatus(HttpStatus.BAD_REQUEST)</code> 를 지정(혹은 ResponseEntity의 상태코드를 지정)했으므로 HTTP 상태 코드 400으로 응답 </li>
</ul>
</aside>

### @ExceptionHandler with @ControllerAdvice

- `@ExceptionHandler` 를 사용해서 예외를 깔끔하게 처리할 수 있게 되었지만, **정상 코드와 예외 처리 코드가 하나의 컨트롤러에 섞여 있음!**
- `@ControllerAdvice`(, `@RestControllerAdvice`)를 사용하면 둘을 분리하여 좀 더 깔끔한 코드를 작성할 수 있음
- `@ControllerAdvice`
    - ExceptinonHanlder를 **기본 Controller와 분리해서 사용할 수 있는 Advice**
    - 대상으로 지정한 여러 컨트롤러에 `@ExceptionHandler` , `@InitBinder` 기능을 부여해주는 역할
    - 여러 다양한 Controller에서 발생하는 Exception을 공통적으로(글로벌) 처리 가능 (대상을 지정하지 않으면)
- `ExControllerAdvice` ← `@RestControllerAdvice`
    
    ```java
    @RestControllerAdvice // @ControllerAdvice + @ResponseBody
    public class ExControllerAdvice {
    
        @ResponseStatus(HttpStatus.BAD_REQUEST) // 아래 코드는 정상 흐름으로 인식하므로, 직접 status 정의
        @ExceptionHandler(IllegalArgumentException.class) // 해당 Exception에 대해서 처리하겠다는 뜻
        public ErrorResult illegalExHandler(IllegalArgumentException e) {
            log.error("[exceptionHandler] ex", e);
            return new ErrorResult("BAD", e.getMessage());
        }
    		...
    }
    ```
    
    - 이런 식으로 Controller를 생성하듯이 ControllerAdvice를 생성하면 됨!
    - 해당 ControllerAdvice에 **원하는 ExceptionHandler를 지정**해주면 됨
    - 만약 ControllerAdvice에 **대상을 지정하지 않으면 글로벌 적용**
- **대상 컨트롤러 지정 방법**
    - 어노테이션 단 → `@ControllerAdvice(annotations = RestController.class)`
    - 패키지 단 → `@ControllerAdvice("org.example.controllers")`
    - 컨트롤러 단 → `@ControllerAdvice(assignableTypes = {ControllerInterface.class, AbstractController.class})`
- `@ExceptionHandler` 와 `@ControllerAdvice` 를 조합하면 예외를 보다 더 깔끔하게 해결 가능!!