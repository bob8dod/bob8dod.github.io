---
layout: post
title:  "Login with Filter(Servlet), Interceptor(Spring)"
date:   2022-05-30
image:  /posts/filter_interceptor.png
tags:   SpringBoot Login
---

<aside>
⚠️ <strong>필터나 인터셉터를 왜 써야 할까?</strong> 
<ul>
<li>필터나 인터셉터 없이 로그인되지 않은 사용자가 접근할 수 없는 페이지를 생성하려면 <b>모든 컨트롤러 로직에 공통적으로 로그인 여부를 확인</b>해야 됨. 이렇게 되면 복잡하기도 하고 추후 수정 시 이 모두를 수정해야되는 큰 문제가 발생함.</li>
<li>이를 해결하기 위해선 <strong>공통 관심사 개념</strong>을 도입해야 됨.</li>
<li>일반적인 공통 관심사를 효과적으로 해결하기 위해선 <b>AOP</b>도 사용 되지만, 웹과 관련된 공통 관심사는 HttpServletRequest를 제공하는 <b>Servlet Filter</b> 나 <b>Spring Interceptor</b>를 사용하는 것이 더 좋음!</li>
</ul>
</aside>

## Servlet Filter

### 서블릿 필터

- 서블릿에 들어가기 전 그 **입구를 지키는 수문장**과 같은 역할
- **서블릿**이 제공하는 기능
- 필터 전반적 흐름
    - `HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 컨트롤러`
    - **필터가 호출된 다음 서블릿**이 호출 (필터 적용 시)
    - 사용 시나리오 예시 → 모든 고객의 **요청 로그**를 남기는 요구사항
    - 추가로 필터는 **URL 패턴에 적용** 가능 (**웹 공통 관심사에 사용**하기 좋음)
- 필터 제한 흐름 (로그인)
    - 로그인 사용자 : `HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 컨트롤러`
    - 비 로그인 사용자 : `HTTP 요청 -> WAS -> 필터(적절하지 않은 요청이라 판단, **서블릿 호출X**)`
    - 즉, 필터에서 판단 후 **그 시점에 바로 끝낼 수 있**음
- 필터 **체인**
    - `HTTP 요청 -> WAS -> **필터1 -> 필터2 -> 필터3** -> 서블릿 -> 컨트롤러`
    - 필터는 **체인으로 구성**, 중간에 자유롭게 추가할 수 있음
    - 로그 남긴 후 로그인 체크 등 여러 필터를 갈아 끼울 수 있음
    - 필터의 의미 그대로의 역할인 것
- `Filter interface`
    
    ```java
    public interface Filter {
    	 public default void init(FilterConfig filterConfig) throws ServletException {}
    	 public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException;
    	 public default void destroy() {}
    }
    ```
    
    - `init()` : 필터 초기화 메서드, 서블릿 컨테이너가 생성될 때 호출
    - **`doFilter()`** : 고객의 요청이 올 때 마다 호출됨. **필터의 로직 구현 부분. (필수 구현 항목)**
    - `destroy()` : 종료 메서드, 서블릿 컨테이너가 종료될 때 호출됨
    - 해당 interface를 우리가 **사용하고 싶은 용도의 구현체로** 만들어 사용하면 됨

### 요청 로그 (by 서블릿 필터)

- 모든 요청을 로그로 남기는 필터 개발
- `LogFilter`
    - Filter interface를 구현하여 custom Filter 생성 → `public class LogFilter implements Filter {}`
    - `doFilter`
        
        ```java
        @Override // 필터 동작
        public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
            log.info("log filter doFilter");
        
            HttpServletRequest httpRequest = (HttpServletRequest) request; // Http로 사용하기 위함
            String requestURI = httpRequest.getRequestURI();
        
            String uuid = UUID.randomUUID().toString(); // 요청 구분을 위한 UUID
        
            try {
                log.info("REQUEST [{}][{}]", uuid, requestURI);
                chain.doFilter(request, response); // 다음 필터가 있으면 필터를 호출하고, 필터가 없으면 디스패치서블릿을 호출
            } catch (Exception e) {
                throw e;
            } finally {
                log.info("RESPONSE [{}][{}]", uuid, requestURI);
            }
        
        }
        ```
        
        - HTTP 요청이 오면 해당 **doFilter 호출**됨
        - **웹 상의 request**로 사용하기 위해 `HttpServletRequest` 로 **down casting** 진행 → `(HttpServletRequest) request;`
        - `try` : 요청 로그를 남기고 `chain.doFilter(request, response)` 를 통해 요청과 응답을 가진 상태로 **다음 필터 혹은 서블릿을 호출** (다음 체인으로 넘어가지 않으면 그대로 종료됨!)
        - `catch` : 에러 발생시 잡아서 그냥 던져줌
        - `finally` : 에러가 발생하든 발생하지 않든 무조건 실행
- `WebConfig`
    
    ```java
    @Bean // 필터 등록
    public FilterRegistrationBean logFilter() {
        FilterRegistrationBean<Filter> filterRegistrationBean = new FilterRegistrationBean<>();
        filterRegistrationBean.setFilter(new LogFilter()); // 필터 지정
        filterRegistrationBean.setOrder(1); // 순서 지정
        filterRegistrationBean.addUrlPatterns("/*"); // 적용할 URI 패턴 설정
    
        return filterRegistrationBean;
    }
    ```
    
    - `@Configuration` : Spring의 설정 부분
    - 해당 LogFilter 사용하도록 **등록**
    - Spring Boot에선 `FilterRegistrationBean` 를 통해 **Filter 등록 가능**
    - `filterRegistrationBean.setFilter()` : 사용할 필터 지정
    - `filterRegistrationBean.setOrder(1);` : 필터 **순서** 지정 (1이 가장 먼저)
    - `filterRegistrationBean.addUrlPatterns("/*");` : **적용할 URI 패턴** 설정 (한번에 여러 패턴을 지정 가능)

<aside>
⚠️ <strong>Filter 등록의 다른 방법</strong> <br>
<code>@ServletComponentScan</code> , <code>@WebFilter(filterName = "logFilter", urlPatterns = "/*")</code> 의 어노태이션 기반으로도 간편히 등록할 수 있지만, 해당 어노테이션들로는 <b>필터 순서 조절 불가</b>! 그래서 <code>Confinguration</code>에서 <code>FilterRegistrationBean</code> 을 <code>Bean</code>으로 등록하여 설정하는게 BEST!

</aside>

> 참고 : HTTP 요청 시 같은 요청의 로그에 모두 같은 식별자를 자동으로 남기는 방법은 **logback mdc**
> 

### 로그인 인증 (by 서블릿 필터)

- **로그인 인증 체크 필터**. 로그인 되지 않은 사용자는 접근 불가하도록!
- `LoginCheckFilter`
    - 접근 불가 페이지를 지정하기 보다는 **접근 가능한 white list 를 통해 관리**
        - `private static final String[] *whiteList* = new String[]{"/", "/login", "/logout", "/members/add", "/css/*"};`
        - 홈, 로그인화면, 로그아웃, 회원가입, “**css 파일”** 등은 로그인 없이도 접근 가능해야 됨!
        - whilteList에 포함되는 지 확인 (포함되지 않은 애들에 대해서만 인증체크 실행!)
            
            ```java
            private boolean isLoginCheckPath(String requestURI) {
                return !PatternMatchUtils.simpleMatch(whiteList, requestURI);
            }
            ```
            
    - Filter interface를 구현하여 custom Filter 생성 → `public class LoginCheckFilter implements Filter {}`
    - `doFilter`
        
        ```java
        @Override
        public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
            HttpServletRequest httpRequest = (HttpServletRequest) request;
            String requestURI = httpRequest.getRequestURI();
        
            HttpServletResponse httpResponse = (HttpServletResponse) response;
        
            try {
                log.info("인증 체크 필터 시작 : [{}]", requestURI);
        
                if (isLoginCheckPath(requestURI)) {
                    log.info("인증 체크 로직 실행 : [{}]", requestURI);
                    HttpSession session = httpRequest.getSession(false);
                    if (session == null || session.getAttribute(SessionConst.LOGIN_MEMBER) == null) {
                        log.info("미인증 사용자 요청 : [{}]", requestURI);
                        // 로그인으로 redirect. url을 이렇게 보내는 것은 로그인 후 다시 현재 화면으로 돌아가기 위함 -> pathVariable 이용
                        httpResponse.sendRedirect("/login?redirectURL=" + requestURI);
                        return;
                    }
                }
        
                chain.doFilter(request, response); // 다음 필터로 이동. 필터가 없으면 디스패치서블릿으로 이동하여 컨테이너 동작
            } catch (Exception e) {
                throw e;
            } finally {
                log.info("인증 체크 필터 종료 : [{}]", requestURI);
            }
        }
        ```
        
        - HTTP 요청이 오면 해당 **doFilter 호출**됨
        - **웹 상의 request**로 사용하기 위해 `HttpServletRequest` 로 **down casting** 진행 → `(HttpServletRequest) request;`
        - try
            - `isLoginCheckPath(requestURI)` 를 통해 whiteList에 포함되는 지 확인 (포함되지 않은 애들에 대해서만 인증체크 실행!)
            - 로그인 인증이 필요한 경우
                - Session을 통해 로그인 되어 있는지 확인
                - 로그인 되어 있지 않거나 미인증 사용자일 경우 **`httpResponse.sendRedirect("/login?redirectURL=" + requestURI);`** 를 통해 **redirect 실행**
                    
                    <aside>
                    ⚠️ <strong>RedirectURL</strong> <br>
                    redirect를 보낼 때, 로그인 후 바로 <b>다시 그 요청했던 페이지로 돌아가게</b> 하기 위해서 <b>해당 페이지 정보를 보내줌</b>. <br>
                    → 로그인 후 controller에서 <code>@RequestParam</code> 을 통해 이전 페이지로 다시 돌아갈 수 있음 [사용자 편의성 증대] 
                    
                    </aside>
                    
                - `return` 을 통해 다음 체인으로 넘어가지 않고 **해당 redirect를 바로 실행**할 수 있도록 설정 **[중요]**
            - 로그인 인증이 필요하지 않거나 로그인 인증에서 통과한 경우
                - `chain.doFilter(request, response)` 를 통해 요청과 응답을 가진 상태로 **다음 필터 혹은 서블릿을 호출**
    - `WebConfig`
        - `loginCheckFilter()` 추가
            - 이전과 코드는 유사
            - 하지만, order를 2로 둠으로써 **LogFilter 이후에 진행**될 수 있도록 설정
            - 모든 URL에 작동하도록 패턴 지정 → **필터 안에서 whiteList로 걸러**냄 (미래의 추가적인 인증 페이지를 편하게 넣기 위함)
    - **RedirectURL 처리 [중요]**
        - 로그인에 성공하면 **처음 요청한 URL로 이동**하는 기능
        - `@PostMapping("/login")` 부분
        
        ```java
        @PostMapping("/login")
        public String loginV4(
        		@Validated @ModelAttribute("loginForm") LoginForm form, BindingResult bindingResult,
            @RequestParam(defaultValue = "/") String redirectURL,
            HttpServletRequest request) {
        
        		...
        
            return "redirect:"+redirectURL;
        }
        ```
        
        - `@RequestParam(defaultValue = "/") String redirectURL` 을 통해서 query parameter 가 있으면 해당 url로 갈 수 있도록 설정 (없으면 그냥 Home 으로)
        - 로그인 체크 필터에서, 미인증 사용자는 요청 경로를 포함해서 `/login` 에 **redirectURL 요청 파라미터**를 추가해서 요청. 이 값을 사용해서 로그인 성공시 해당 경로로 고객을 **redirect** 함
        - 즉, 이를 통해서 만약 인증이 필요한 URL로 들어갔다가 로그인이 필요하여 **로그인 한 후에 다시 그 URL로 갈 수 있도록 설정**이 가능!! **[사용자 편의성을 위해 많이 쓰는 방식, 중요!]**

<aside>
⚠️ <strong>서블릿 필터를 사용한 결과!</strong> <br>
서블릿 필터를 잘 사용한 덕분에 로그인 하지 않은 사용자는 나머지 경로에 들어갈 수 없게 됨! 또한 <b>공통 관심사를 서블릿 필터를 사용해서 해결</b>한 덕분에 향후 로그인 관련 정책이 변경되어도 이 부분만 변경하면 원하는대로 동작하게 할 수 있음!

</aside>

## Spring Interceptor

### 스프링 인터셉터

- **웹과 관련된 공통 관심 사항을 효과적으로 해결**할 수 있는 기술
- **서블릿 이후** 컨트롤러 입구를 지키는 **수문장**같은 역할
- **스프링 MVC**가 제공하는 기술
- 인터셉터 전반적인 흐름
    - `HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 스프링 인터셉터 -> 컨트롤러`
    - 디스패처 서블릿과 컨트롤러 사이에서 **컨트롤러 호출 직전에 호출** (스프링 MVC가 제공하는 기능이기 때문에 **디스패처 서블릿 이후**에 동작. **[스프링 MVC의 시작점 = 디스패처 서블릿]**)
    - 서블릿 URL 패턴보다 더 정밀한 설정 가능
- 인터셉터 제한
    - 로그인 사용자 : `HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 스프링 인터셉터 -> 컨트롤러`
    - 비 로그인 사용자 : `HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 스프링 인터셉터`
    - 필터와 마찬가지로 **인터셉터에서** 적절한 요청이 아니라고 판단하면 **그 즉시 끝낼 수 있음**
- 인터셉터 체인
    - `HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 인터셉터1 -> 인터셉터2 -> 컨트롤러`
    - 필터와 마찬가지로 **여러 인터셉터를 끼어 사용**할 수 있음

<aside>
⚠️ <strong>필터랑 인터셉터랑 차이가 없는 것 같은데? 인터셉터는 왜 있는 거지?</strong> <br>
물론 호출되는 순서만 다르고 제공하는 기능을 비슷해보이지만, 인터셉터는 서블릿 필터보다 <b>편리하게 사용이 가능하고 더 정교하며 다양한 기능을 지원</b>함!!

</aside>

- `HandlerInterceptor Interface`
    
    ```java
    public interface HandlerInterceptor {
    	default boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {}
    
    	default void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
    			@Nullable ModelAndView modelAndView) throws Exception {}
    
    	default void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler,
    			@Nullable Exception ex) throws Exception {}
    }
    ```
    
    - **`preHandle`**
        - 컨트롤러 **호출 전** 실행되는 Method
        - 반환값이 **true** 이면 **다음 체인**으로 넘어가고 **false** 이면 뒤에 체인은 진행되지 않음
    - `postHandle` : 컨트롤러 **호출 후**에 호출됨
    - `afterCompletion` : 뷰가 **렌더링 된 이후**에 호출됨
    - 해당 interface를 우리가 사용하고 싶은 **구현체**로 만들어 사용하면 됨
- 스프링 인터셉터 호출 흐름

  ![]({{site.baseurl}}/images/posts/post-220530/Untitled.png)
    
    - Interceptor가 존재한 경우(하나), **HTTP 요청**이 들어오면 `DispatcherServlet`에서 등록된 Interceptor의 **`preHandle`을 호출**, 그 후 요청에 매핑된 컨트롤러를 다룰 수 있는 `handlerAdaptor`를 호출해서 `controller` 실행. 그 후 실행 결과로 `ModelAndView`를 반환하고 `postHandle`로 넘어감 (고로 `postHandle`에선 `ModleAndView` 사용 가능) 이후 `Veiw`를 rendering하고 마지막으로 `afterCompletion`이 호출되고 마지막으로 **HTTP 응답**이 나감.
    - 여기서도 **필터와의 차이**를 느낄 수 있음 (더 세부적임!)
- 스프링 인터셉터 예외 상황

  ![]({{site.baseurl}}/images/posts/post-220530/Untitled 1.png)
    
    - 컨트롤러에서 예외 발생 시
        - `preHandle` 은 전과 동일하게 호출 됨
        - `postHandle` 은 **호출되지 않음**
        - `afterCompletion` 은 전과 동일하게 호출 됨. 추가로 예외 발생 시 **예외를 파라미터로** 받아서 어떤 예외가 발생했는지 로그로 남길 수 있음
    - 즉, **예외와 무관하게 공통 처리**를 하려면 **`afterCompletion`**을 사용 해야 됨 (`postHandle`은 예외 발생 시 호출되지 않음)
    - 여기서 **필터와의 큰 차이**를 느낄 수 있음 (더 정밀함!)

### 요청 로그 (by 스프링 인터셉터)

- **모든 요청 로그**를 남기는 Interceptor 개발
- `LogInterceptor`
    - Interceptor interface를 **구현**하여 custom Incerceptor 생성 → `public class LogInterceptor implements HandlerInterceptor {}`
    - `preHandle`
        
        ```java
        @Override
        public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
            String requestURI = request.getRequestURI();
            String uuid = UUID.randomUUID().toString(); // filter 처럼 uuid를 인터셉터의 끝단(afterCompletion)에 넘겨줘야됨
        
            request.setAttribute(LOG_ID,uuid); // afterCompletion에서 사용 가능
        
            // @RequestMapping : HandlerMethod
            // 정적 리소스 : ResourceHttpRequestHandler
            if (handler instanceof HandlerMethod) { // @RequestMapping 과 관련된 Handler
                HandlerMethod hm = (HandlerMethod) handler; // 호출할 컨트롤러 메서드의 모든 정보가 포함되어 있음
            }
        
            log.info("REQUEST [{}][{}][{}]", uuid, requestURI, handler);
            return true; // 다음으로 넘어가게 해줌
        }
        ```
        
        - HTTP 요청이 오면 **서블릿 실행 이후** **`preHandle` 이 호출**됨
        - `request.setAttribute(LOG_ID, uuid);` : LogInterceptor 도 싱글톤 처럼 사용되기 때문에 맴버변수를 사용하면 안됨. 따라서 **uuid를 request 에 담아 넘겨줌** (이렇게 하면 `postHandle`, `afterCompletion` 에서도 uuid를 사용할 수 있음 → **request는 들어오고 나갈 때 까지 동일하므로!**) (필터는 지역변수 사용 가능 → doFilter밖에 사용하지 않음!)
        - `return true` : **true로 반환**해야 다음 체인으로 넘어감. false면 그 즉시 중단되어 response로 나감
        
        <aside>
        ⚠️ <strong>HandlerMethod vs ResourceHttpRequestHandler</strong>
        <ul>
        <li>HandlerMethod  : 스프링을 사용하면 일반적으로 <code>@Controller</code> , <code>@RequestMapping</code> 을 활용한 핸들러 매핑을 사용하는데, 이 경우 핸들러 정보로 HandlerMethod 가 넘어옴 </li>
        <li>ResourceHttpRequestHandler : <code>@Controller</code> 가 아니라 /resources/static 와 같은 정적 리소스가 호출 되는 경우 <code>ResourceHttpRequestHandler</code> 가 핸들러 정보로 넘어 옴 </li>
        </ul>
        </aside>
        
        <aside>
        ⚠️ <strong>HandlerAdaptor vs HandlerMapping vs HandlerMethod</strong>
        <ul>
        <li>HandlerAdaptor : 해당 handler(Controller)를 다룰 수 있게 끔하는 역할 (Adaptor) </li>
        <li>HandlerMapping : 해당되는 handler(Controller) </li>
        <li>HanlderMethod : 해당되는 handler 속 해당 method </li>
        </ul>
        </aside>
        
    - `postHandle`
        - **modelAndView** 값 사용 가능
        - 예외가 발생하면 호출되지 않으므로 종료 Log를 담지 않음
    - `afterCompletion`
        - error 확인 가능
        - **예외가 발생해도 호출**되므로 종료 Log를 남기기에 적합
- `WebConfig` (Fileter와는 **다르게 MVC에서 제공하는 Config 기능**을 사용해야 하므로 `WebMvcConfigurer` **을 implements** 해야됨) → `public class WebConfig implements WebMvcConfigurer`
    
    ```java
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
    		registry.addInterceptor(new LogInterceptor())
    						.order(1)
    						.addPathPatterns("/**")
    						.excludePathPatterns("/css/**", "/*.ico", "/error");
    }
    ```
    
    - `WebMvcConfigurer` 가 제공하는 `addInterceptors()` 를 사용하여 **인터셉터를 등록 및 설정**
    - `.addInterceptor(new LogInterceptor())` : 해당 LogInterceptor 사용하도록 **등록**
    - `.order(1)` : 사용할 필터 **순서** 지정 (1이 가장 먼저)
    - `.addPathPatterns("/**")` : 인터셉터를 적용할 **URL 패턴을 지정** (한번에 여러 패턴을 지정 가능)
    - `excludePathPatterns("/css/**", "/*.ico", "/error")` : 인터셉터에서 제외할 패턴을 지정 (**Filter와 다르게 더 정밀하고 편리한 부분**)
    - 해당 URL 패턴 규칙은 스프링 공식 메뉴얼에 잘 나와 있음! [[공식 메뉴얼]](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/util/pattern/PathPattern.html)

### 로그인 인증 (by 스프링 인터샙터)

- **로그인 인증 체크 인터샙터**. 로그인 되지 않은 사용자는 접근 불가하도록!
- `LoginCheckInterceptor`
    - 해당 interceptor 안에서 white list (접근 가능 페이지 따로 설정) 설정은 필요 없음! → Inteceptor 등록 시 **include, exclude 설정**을 통해서 가능! (**Filter 보다 좋은 점**)
    - 로그인 인증은 컨트롤러 호출 전에만 실행되면 됨 → **preHandler만 구현**해주면 됨 (filter는 try except 다해줘야됨 즉, 처음부터 끝까지 다 신경써야되지만 i**nterceptor는 prehandle만 신경 써도 됨**. (로그인 부분에서) 즉, **내 관심사에 집중**할 수 있음)
    - 또한 로그인이 필요한 상태의 웹사이트에서만 해당 Inteceptor가 진행되는 것 (Filter는 어떤 상황이든 filter가 동작하고 그 filter 안에서 웹사이트 판단을 진행함(whiteList))
    - `preHandle`
        
        ```java
        @Override
        public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
            String requestURI = request.getRequestURI();
            log.info("인증 체크 인터셉터 실행 : {}", requestURI);
            HttpSession session = request.getSession();
        
            if (session == null || session.getAttribute(SessionConst.LOGIN_MEMBER)  == null) {
                log.info("미인증 사용자 요청!");
                // 로그인 화면으로 redirect
                response.sendRedirect("/login?redirectURL=" + requestURI);
                return false; // 더 이상 진행되지 않음.
            }
        
            return true; // 다음 체인으로 진행
        }
        ```
        
        - HTTP 요청이 오면 서블릿 실행 이후 **`preHandle` 이 호출**됨
        - **Session**을 통해 로그인 되어 있는지 확인
        - 로그인 되어 있지 않거나 미인증 사용자일 경우 `response.sendRedirect("/login?redirectURL=" + requestURI);` 를 통해 **redirect 실행**
            - **`return false`**을 통해 **다음 체인으로 넘어가지 않고 해당 redirect를 바로 실행**할 수 있도록 설정 **[중요]**
        - 로그인 인증에서 통과한 경우
            - **`return true`**를 통해 **다음 Interceptor 혹은 컨트롤러 호출**
    - `WebConfig`
        - `LoginCheckInterceptor()` 추가
            - 이전과 코드는 유사
            - 하지만, **order를 2**로 둠으로써 `LogInterceptor` **이후에 진행**될 수 있도록 설정
            - `excludePathPatterns` 를 통해 **whiteList의 역할**을 수행. (인터셉터를 적용하지 않을 URL)

<aside>
⚠️ <strong>스프링 인터셉터를 사용한 결과!</strong> <br>
<b>공통 관심사</b>를 스프링 인터셉터를 사용해서 <b>해결</b>! 추가로 필터보다 훨씬 편리하고 세밀하게 설정이 가능함! → <b>필터보단 인터셉터</b>를 사용하자!

</aside>

## Login with ArgumentResolver

- **ArgumentResolver를 활용**해서 로그인 인증 객체 받아오기
- HanlderMethod의 parameter에 `@Login` 이라는 **custom annotation**을 만들어서 custom ArgumentResolver를 통해 로그인 인증 객체를 받는 것 → `public String homeLoginV3ArgumentResolver(**@Login** Member loginMember, Model model)`
- `@Login` annotation
    
    ```java
    @Target(ElementType.PARAMETER)
    @Retention(RetentionPolicy.RUNTIME)
    public @interface Login { }
    ```
    
    - `@Target(ElementType.PARAMETER)` : **파라미터에만** 사용한다는 뜻!
    - `@Retention(RetentionPolicy.RUNTIME)` : 런타임까지 **애노테이션 정보가 남아**있어 활용 가능
- `LoginMemberArgumentResolver` 설정
    - `@Login` annotation 을 처리해줄 수 있는 **custom ArgumentResolver**
    - **`HandlerMethodArgumentResolver` 구현체** (Controller의 Method 단의 argument에 대한 resolver 이므로)
    - `public class LoginMemberArgumentResolver implements HandlerMethodArgumentResolver`
    - 파라미터가 해당 어노테이션에 매핑되는 애인지 확인
        
        ```java
        @Override
        public boolean supportsParameter(MethodParameter parameter) {
        	boolean hasLoginAnnotation = parameter.hasParameterAnnotation(Login.class);
        	boolean hasMemberType = Member.class.isAssignableFrom(parameter.getParameterType());
        	
        	return hasLoginAnnotation && hasMemberType;
        }
        ```
        
        - `parameter.hasParameterAnnotation(Login.class);` : 파라미터에 달린 annotation이 Login annotation인지 확인
        - `Member.class.isAssignableFrom(parameter.getParameterType());` : 해당 파라미터가 Member의 하위 class인지 확인
        - **둘 다에 해당 하면 true** 반환 → **annotation 사용** 가능!
        - 결론적으로 `@Login` 애노테이션이 있으면서 `Member` 타입이면 해당 ArgumentResolver 가 사용된다.
    - 파라미터와 어노테이션에 따른 Argument Resolve 진행
        
        ```java
        @Override // annotation 동작
        public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer, NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception {
        
            HttpServletRequest request = (HttpServletRequest) webRequest.getNativeRequest();
            HttpSession session = request.getSession(false);
        
            // 로그인이 안된 상황
            if (session == null) {
                return null;
            }
        
            // 로그인이 된 상황 (Member 객체를 반환), 즉 어노테이션이 할당된 파라미터에 해당 반환 객체를 할당함
            return session.getAttribute(SessionConst.LOGIN_MEMBER);
        }
        ```
        
        - `@Login` 어노테이션의 **동작 로직**
        - 컨트롤러 호출 직전에 호출 되어서 **필요한 파라미터 정보를 생성**해줌
        - **로그인 인증 과정** 진행
        - 결론적으로 로그인이 된 상태라면 해당 **로그인 된 회원의 객체(Member)가 반환**됨!
        - 이후 스프링MVC는 컨트롤러의 메서드를 호출하면서 여기에서 반환된 member 객체를 파라미터에 전달해 줌!
- `LoginMemberArgumentResolver` 등록 (`WebConfig`)
    - `WebMvcConfigurer`  → `public class WebConfig implements WebMvcConfigurer`
        
        ```java
        @Override
        public void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {
        	 resolvers.add(new LoginMemberArgumentResolver());
        }
        ```
        
        - resolver에 custom ArgumentResolver인 `LoginMemberArgumentResolver` 등록
        

<aside>
⚠️ <strong>WebConfig</strong>
항상 기본 기능에서 내가 interface를 통해 <b>구현 한 것을 Spring에서 사용</b>하려면 <code>WebConfing</code>를 통해 <b>등록</b>해줘야됨!

</aside>