---
layout: post
title:  "Servlet with MVC Pattern, View Template(JSP)"
date:   2022-05-04
image:  /posts/servlet+mvc.png
tags:   SpringBoot MVC
---


## 서블릿, JSP, MVC 패턴

> **회원 관리 앱을 통한 서블릿, JSP, MVC 패턴 이해**<br>
‘서블릿만’을 이용한 앱 → ‘서블릿 + Template Engine(JSP)’ 을 이용한 앱 → ‘서블릿 + Template Engine(JSP) + MVC 패턴’을 이용한 앱 순으로 개발을 진행하며 구조적으로 어떻게 발전해 왔는지를 확인
> 

### 회원 관리 앱

- 요구사항
    - 회원 정보 : 이름, 나이
    - 기능 : 회원 저장, 회원 목록 조회
- 회원 도메인 모델 (Member.class)
    - `Long id` : 식별자
    - `String username` : 이름
    - `int age` : 나이
- 회원 저장소 (MemberRepository)
    - `static Map<Long, Member> store` : HashMap, 원래는 동시성을 고려해서 HashMap을 사용하면 안됨.
    - `static long sequence` : 식별자 부여 값
    - `static final MemberRepository instance` : 싱글톤. 하나의 인스턴스만을 사용
    - `save, findById, findAll, clearStore` Method 생성
    - 해당 저장소에 대한 Test 작성
        - `@AfterEach`를 통해 각 Test마다 store이 초기화 될 수 있도록 설정
        - `save, findById, findAll, clearStore` Method 각각에 대한 Test 설정. → `Assertions.assertThat` 으로 검증

### 서블릿(Servlet)으로 앱 만들기

- 회원 가입 form을 보여주는 servlet
    - `public class MemberFormServlet extends HttpServlet` -> `@WebServlet(name = "memberFormServlet", urlPatterns = "/servlet/members/new-form")` : 해당 url로 들어오게 되면 생성된 request, response 객체를 담은 MemberFormServlet의 service method 실행.
    - `protected void service( ... )` → from 을 보여주는 화면이기에 request는 사용하지 않고 response만을 사용 (회원 정보를 받는 html을 반환해줘야 함)
        
        ```java
        response.setContentType("text/html");
        response.setCharacterEncoding("utf-8");
        
        PrintWriter w = response.getWriter();
        w.write("<!DOCTYPE html>\n" +
                "<html>\n" +
                "<head>\n" +
                ... )
        ```
        
        - html 을 반환할거라는 정보를 알려주고 인코딩 진행.
        - 그 후 html 그 자체를 파일형식이 아닌 **text형식으로 보내줌**. (servlet을 사용할 경우. 이를 통한 동적 화면도 가능.)
- 저장된 회원 리스트를 보여주는 servlet
    - `public class MemberListServlet extends HttpServlet` → `@WebServlet(name = "memberListServlet", urlPatterns = "/servlet/members")`  : 해당 url로 들어오게 되면 생성된 request, response 객체를 담은 MemberListServlet의 **service method 실행.**
    - `private MemberRepository memberRepository = MemberRepository.*getInstance*();` → 저장된 회원들의 목록을 가져오기에 저장소 (MemberRepository) 사용
    - `protected void service( ... )` → 회원 목록만을 보여주는 화면이기에 request는 사용하지 않고 response만을 사용 (회원 정보를 받는 html을 반환해줘야 함)
        
        ```java
        response.setContentType("text/html");
        response.setCharacterEncoding("utf-8");
        
        List<Member> members = memberRepository.findAll();
        
        PrintWriter w = response.getWriter();
        w.write("<html>");
        ...
        for (Member member : members) {
            w.write(" <tr>");
            w.write(" <td>" + member.getId() + "</td>");
            w.write(" <td>" + member.getUsername() + "</td>");
            w.write(" <td>" + member.getAge() + "</td>");
            w.write(" </tr>");
        } // 동적으로 화면을 구성할 수 있음
        ...
        ```
        
        - html 을 반환할거라는 정보를 알려주고 인코딩 진행.
        - 그 후 html 그 자체를 파일형식이 아닌 text형식으로 보내줌. (servlet을 사용할 경우. 이를 통한 동적 화면도 가능.)
- 회원 저장, 저장 후 정보를 보여주는 Servlet
    - `public class MemberSaveServlet extends HttpServlet` → `@WebServlet(name = "memberSaveServlet", urlPatterns = "/servlet/members/save")` : 해당 url로 들어오게 되면 생성된 request, response 객체를 담은 MemberFormServlet의 service method 실행.
    - `private MemberRepository memberRepository = MemberRepository.*getInstance*();` → 저장된 회원들의 목록을 가져오기에 저장소 (MemberRepository) 사용
    - `protected void service( ... )` → 이전과는 다르게 회원을 저장하는 기능이 추가되어 response 뿐만 아닌 request도 사용.
        
        ```java
        // username=뭐시기&age=숫자
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));
        
        // 값에 따른 Member 객체 생성
        Member member = new Member(username, age);
        System.out.println("member = " + member);
        
        // member 저장
        memberRepository.save(member);
        
        // response(반환값) 설정
        response.setContentType("text/html");
        response.setCharacterEncoding("utf-8");
        PrintWriter w = response.getWriter();
        w.write("<html>\n" +
                "<head>\n" +
        				... +
        				"<ul>\n" +
                  " <li>id="+member.getId()+"</li>\n" +
                  " <li>username="+member.getUsername()+"</li>\n" +
                  " <li>age="+member.getAge()+"</li>\n" +
        				... 
        )
        ```
        
        - request를 통해 넘겨져 온 form 형식의 Form Data 를 조회
        - 조회된 값들을 MemberRepository를 통해 저장
        - 저장된 결과를 response를 통해 반환 (동적으로 html 생성)

<aside>
🚨 <strong>하지만!</strong><br>
이렇게 Servlet만을 사용해서 HTML을 생성하여 보내주는 것은 동적인 화면을 만들 수 있다는 장점이 있지만, 직접 HTML을 JAVA코드 안에서 Text 형태로 작성해야 된다는 단점이 있음. (광장히 귀찮아!) → Template Engine(JSP, Thymeleaf) 사용을 통해 해결 가능

</aside>

### + View Template (JSP)를 이용한 앱 만들기

> JSP는 현재 잘 사용되고 있는 View Template은 아니지만, 그래도 이전에 계속 사용되었기에 한번 맛보고 넘어가는 것. 이후에는 Veiw Template을 Thymeleaf로 사용할 것임.

 
- JSP
    - JSP implemantation
        
        ```java
        implementation 'org.apache.tomcat.embed:tomcat-embed-jasper'
        implementation 'javax.servlet:jstl'
        ```
        
    - JSP 는 webapp 하위에 설정해야 됨 (src/main/webapp/jsp/..)
    
    <aside>
    💡 <strong>JSP는 파일 하나 안에서 html, java 둘 다 사용이 가능.</strong><br>
    <ul>
    <li>즉, java로 비지니스 로직을 만들고</li>
    <li>그 결과에 따른 값들을 이용한 html파일을 만들 수 있음 (동적 화면)</li>
    <li>또한, jsp는 Survlet으로 자동 변환되어서 사용되기에 jsp 안에서 response와 request를 사용할 수 있음</li>
    </ul>
    </aside>
    
- 회원 저장 (save.jsp)
    - 비지니스 로직 (JAVA 코드)
        
        ```java
        ...
        <%@ page contentType="text/html;charset=UTF-8" language="java" %>
        <% // 로직
            MemberRepository memberRepository = MemberRepository.getInstance();
            String username = request.getParameter("username");
            ...
        %>
        ```
        
        - 여기서 받아오거나 사용된 변수는 html에서도 사용할 수 있음! → `<%=member.getId()%>` : 자바코드의 결과를 출력 하는 것 (`<%=  %>`, soutv() 라고 생각하면 됨)
    - 반환될 화면 (HTML 코드)
        
        ```html
        <html>
        <head>
        ...
        <ul>
            <li>id=<%=member.getId()%></li>
            <li>username=<%=member.getUsername()%></li>
            <li>age=<%=member.getAge()%></li>
        </ul>
        ...
        ```
        
- 회원 리스트
    - 비지니스 로직 (JAVA 코드)
        
        ```java
        ...
        <%@ page contentType="text/html;charset=UTF-8" language="java" %>
        <%
            MemberRepository memberRepository = MemberRepository.getInstance();
            List<Member> members = memberRepository.findAll();
        %>
        ```
        
    - 반환될 화면 (HTML 코드)
        
        ```html
        <html>
        <head>
        ...
            <%
                for (Member member : members) {
                    out.write(" <tr>");
                    out.write(" <td>" + member.getId() + "</td>");
                    out.write(" <td>" + member.getUsername() + "</td>");
                    out.write(" <td>" + member.getAge() + "</td>");
                    out.write(" </tr>");
                }
            %>
        ...
        ```
        
        - **HTML 코드 안에 자바 코드를 넣음**으로써 훨씬 **더 동적인 화면 생성** 가능. (servlet에서 작성했던 코드와 유사)
        - HTML 안에서 자바코드를 실행하려면 `<%``%>` 사용. (servlet에서와 같이 동작함)
        <aside>
        🚨 <strong>하지만!</strong><br>
         JSP(+servlet)만을 사용하게 된다면, 뷰의 항목과 비지니스 코드가 분리되지 않았다는 것을 알 수 있음. 즉, 비지니스 로직을 변경하려해도 JSP에서 수정해야 되고, 보여지는 화면에 대해 수정할 때도 JSP에서 수정해야됨!<br>
        → JSP는 목적에 맞게 HTML로 화면(View)을 그리는 일에 집중하도록 해야 함<br>
        → 분리할 필요가 있음!!! (MVC 패턴 도입 필요!)
        
        </aside>

### + MVC 패턴 도입

- 개요
    - 너무 많은 역할 : 현재 코드는 비지니스 로직을 구현하는 역할, 화면(View)를 그리는 역할 등 분리되지 않고 **한번에 많은 역할**을 맡고 있음
    - 변경의 라이프 사이클 : 또한 이 역할들의 **라이프 사이클이 다르다**는 것. 즉, 각각의 변경은 다르게 발생할 경우가 높고 서로 영향을 주지 않음. 그런데 둘이 같은 코드에 같이 존재하는 것은 옳지 않음
    - 기능 특화 : 화면을 렌더링 하는데에 최적화 된 Veiw Template은 **그 역할에만 집중**하는 것이 맞음
- MVC (Model View Controller)
    - 서로 동작하는 역할에 따라 Controller View 라는 영역으로 나누는 것
    - **Controller** : **HTTP 요청을 받**아 파라미터를 확인하고, **비지니스 로직을 실행**. 마지막으로 뷰에 전달할 최종 데이터를 모델에 담음 (비지니스 로직까지 Controller에 맡기면 Controller의 역할 이 방대해짐. 그래서 보통 Service, Repository(저장소)라는 추가적인 계층을 만들어 수행. 따라서 Controller는 Service의 비지니스 로직을 호출하는 역할만을 수행)
    - **Model** : 뷰에서 사용될 **데이터**를 담음. 이런 모델이 있기에 View는 비지니스 로직을 몰라도 되고 데이터를 직접 접근할 필요가 없어져 화면을 렌더링 하는 역할에 집중 가능
    - **View** : 모델에 담겨 있는 데이터를 사용하여 **화면을 렌더링 하는 일**에 집중. (HTML 생성 부분)
- 회원 가입 form을 보여주는 Servlet Controller
    - `public class MvcMemberFormServlet extends HttpServlet` → `@WebServlet(name = "mvcMemberFormServlet", urlPatterns = "/servlet-mvc/members/new-form")`
    - service 내에서 Controller의 역할 (비지니스 로직 수행, View 렌더링 후 반환) 수행
        
        ```java
        String viewPath = "/WEB-INF/views/new-form.jsp";
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);
        ```
        
        - WEB-INF :  이 하위 파일들은 외부에서 호출해도 호출되지 않음. 즉, controller를 통해서만 사용 가능
        - `request dispatcher`에 이동할 viewPath를 설정.
        - `forward`를 통해 작동(렌더링 및 반환 실시) →  서블릿에서 **jsp를 호출**하는 기능, 서버 내부에서 다시 호출 발생 (redirect가 아님! 메서드 호출하듯이 서버끼리 노는 것!)
    - View는 이전과 동일 (비지니스 로직을 제외한 View)
- 회원 저장 및 저장된 결과를 보여주는 Servlet Controller
    - `public class MvcMemberSaveServlet extends HttpServlet` → `@WebServlet(name = "mvcMemberSaveServlet", urlPatterns = "/servlet-mvc/members/save")`
    - service 내에서 Controller 역할 (비지니스 로직 수행, View 렌더링 후 반환) 수행. 추가적으로 렌더링 시 사용될 변수를 Model에 담아 넘겨줌
        
        ```java
        ...
        request.setAttribute("member", member);
        
        String viewPath = "/WEB-INF/views/save-result.jsp";
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);
        ```
        
        - `request.setAttribute()` 를 통해 reqeust 내부 저장소 사용. (Model part)
        - `forward`를 통해 Model이 저장된 request 및 response를 넘겨줌. → JSP에서 비지니스 로직의 결과 변수를 사용할 수 있음
    - View는 이전과 유사 (비지니스 로직을 제외한 View)
        - But, request에 member를 담았기 때문에, request를 통해서 값을 사용해야 됨. → `((Member) request.getAttribute("member")).id`
        - 혹은 JSP에서 제공하는 표현식 사용 → `${member.id}`
- 회원 목록 조회 Servlet Controller
    - `public class MvcMemberListServlet extends HttpServlet` → `@WebServlet(name = "mvcMemberListServlet", urlPatterns = "/servlet-mvc/members")`
    - service 내에서 Controller 역할 (이전 Controller와 동일)
        
        ```java
        List<Member> members = memberRepository.findAll();
        request.setAttribute("members", members);
        String viewPath = "/WEB-INF/views/members.jsp";
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);
        ```
        
    - View는 이전과 유사 (비지니스 로직을 제외한 View)
        - JSP 에서 제공하는 loop 문법 사용 → `<c:forEachvar="item" items="${members}">...` (`<%@ **taglib** prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>`)
- **MVC 패턴의 한계**
    - 포워딩의 중복 : `dispatcher.forward(request, response)`가 반복적으로 이루어지고 있음
    - ViewPath의 중복 : 기본 루트 경로의 중복(`”/WEB-INF/views/”`)이 있음
    - 사용하지 않는 코드 : request, response는 안쓰는 경우도 있음 (특히 현재는 response를 모든 코드에서 사용하지 않고 있음)
    - **공통 처리의 어려움** : 각 Controller에서 공통적으로 처리할 수 있게끔 로직을 설정하는 것이 까다로움 (현재는 중복적인 코드로 넣을 수 밖에 없고 이 중복을 메서드로 호출한다해도 그 또한 중복인 것은 마찬가지)
    - 이를 해결하기 위한 개념이 “**프론트 컨트롤러**” → 각 컨트롤러를 호출하기 전에 먼저 공통 기능을 처리하는 **수문장 역할**이 필요. 즉, 모든 요청의 입구를 하나로 만드는 것!
    

## MVC 패턴

> 주어져 있고 자동화되는 Spring 그 자체를 사용하는 것이 아닌, servlet을 이용하여 MVC 패턴을 구현 → 추후 사용되는 Spring Boot가 어떻게 동작하는지 이해하기 위함 (Servlet을 이용해 프론트 컨트롤러를 도입하고, View를 분리하고, Model을 추가하고 실용적이고 유연한 컨트롤러를 도입하면 결국 우리가 사용하고자 하는 **Spring MVC와 똑같은 구조**를 가지게 됨)
> 

### 프론트 컨트롤러 패턴

![]({{site.baseurl}}/images/posts/post-220504/Untitled.png)

- 각각의 controller에서 진행 되었던 **공통로직에 대한 처리**를 Front Controller에서 진행 가능
- 요청은 모두 Front Controller로 가게 되고, 그 후에 해당 하는 Controller를 호출하는 방식
- 즉, Front Controller를 제외한 각 Controller들은 Servlet에 의존하지 않아도 됨 (Front Controller에서 처리하니깐)
- **스프링 웹 MVC도 FrontController 패턴**으로 구현되어 있음

### Front Controller 도입 - ver1

- 일단 각 Controller들의 다형성을 위해 ControllerV1을 생성 → 나중에 front controller에서 각 Controller를 제약없이 사용하기 위함(다형성)
- `public interface ControllerV1` → response와 request를 받아 작동하는 `process` 메서드를 필수 메서드로 설정 (`void process(HttpServletRequest request, HttpServletResponse response) throws ...`)
- 그리고 이 ControllerV1을 구현한 FormController(Servlet), SaveController(Servlet), ListController(Servlet) 생성
    - 이렇게 되면 FrontController에서 각 Controller를 Controller V1으로써 사용 가능 (다형성)
    - FormController(Servlet), SaveController(Servlet), ListController(Servlet) 는 ControllerV1을 구현(`implements ControllerV1`)한 것만 다르고 이전과 비지니스 로직은 동일
- FrontController

  ![]({{site.baseurl}}/images/posts/post-220504/Untitled 1.png)
    
    - 어떤 요청이 들어와도 가장 먼저 실행되는 Controller, 각 URL에 맞는 Controller를 ControllerV1의 구현체로써 호출하여 로직 실행
    - `public class FrontControllerServletV1 extends HttpServlet` → `@WebServlet(name = "frontControllerServletV1", urlPatterns = "/front-controller/v1/*")` 
    ⇒ HttpServlet을 상속받으면서 Servlet을 사용할 수 있게 설정. URL은 v1 하위 url(`urlPatterns = "/front-controller/v1/*"`)은 모두 받아 실행될 수 있도록 설정
    - **FrontController의 역할**
        1. URL 매핑 정보에서 컨트롤러 조회
            - **URL 매핑** 정보 저장소 : `private Map<String, ControllerV1> controllerV1Map = new HashMap<>();`
            - default 생성자를 통해 해당 FrontServlet이 서블릿 컨테이너에 등록 될 때 각 Mapping 정보가 저장 될 수 있도록 설정
                
                ```java
                public FrontControllerServletV1() {
                	  controllerV1Map.put("front-controller/v1/members/new-form", new MemberFormControllerV1());
                	  controllerV1Map.put("front-controller/v1/members/save", new MemberSaveControllerV1());
                	  controllerV1Map.put("front-controller/v1/members/members", new MemberListControllerV1());
                }
                ```
                
        2. 요청된 URL 정보를 얻어와 그 URL에 맞는 Controller를 조회 및 호출 → `ControllerV1 controller = controllerV1Map.get(request.getRequestURI());` (다형성이 사용되는 부분! Interface)
        3. 컨트롤러에서 역할에 맞는 비지니스 로직을 수행하고 JSP foward 진행 → `controller.process(request, response);`

### View 분리 - ver 2

![]({{site.baseurl}}/images/posts/post-220504/Untitled 2.png)

- 모든 Controller에서 View로 이동하는 부분에 중복 존재, 깔끔하지 않았음 (`dispatcher.forward(requset, response)` 부분)
- 이를 별도로 처리하는 객체를 통해 공통 처리 필요
- 즉, Controller를 호출하고 각 Controller에서 forward까지 진행하는 것이 아닌, View와 정보를 가지고 있는 **MyView 객체**를 반환하여 Front Controller에서 **MyView를 이용하여 rendering(forward)까지 진행**
- MyView 객체 → `public class MyView`
    - String ViewPath : view(jsp) 경로
    - `public void render(HttpServletRequest request, HttpServletResponse response) throws …` → forward method, 기존의 각 Controller에서 진행했던 forward를 공통적으로 MyView에서 진행할 수 있음 (공통 처리)
- ControllerV2 (interface) → `public interface ControllerV2`
    - 이전과 동일하게 각 Controller를 호출하고 사용할 수 있도록 Interface Controller 설정. (각 Controller는 해당 Controller V2를 구현하는 것)
    - 이전과는 다르게 각 Controller는 MyView를 통해 ViewPath 설정 및 forward를 진행하기에 MyView를 반환해 줘야 함. → `MyView process(HttpServletRequest ...) throws ...`
- ControllerV2를 구현한 각 Controller들 (Form, Save, List)
    - 이전 버전과 로직은 동일하고 forward 부분 대신 MyView 객체에 원하는 경로를 생성해서 반환만 해주면 됨
    - 즉, 각 Controller에서 forward를 진행하지 않고 FrontController로 반환된 MyView객체를 통해 forward가 진행됨 (공통 처리)
- FrontController
    - 이전과 동일한 흐름으로 가지만, controller 호출 후 **JSP forward 부분이 호출 후 반환 된 MyView를 통해 진행**됨.
        
        ```java
        MyView view = controller.process(request, response);
        view.render(request, response);
        ```
        
    - 즉, Front Controller 도입으로 `MyView` 객체의 `render()` 를 호출하는 부분을 모두 일관되게 처리할 수 있음. 각각의 Controller는 `MyView` 객체를 생성만 해서 반환해주면 됨

### Model 추가 - ver 3

- ver 2 까지 보면, HttpServletRequest, HttpServletResponse가 필요 없는 Controller가 존재. 즉, **Servlet의 의존성**을 벗어날 필요가 있음
- request가 필요한 곳은 따로 Java Map을 이용해서 넘겨주면 됨. → Servlet의 의존성을 벗어날 수 있음
- 또한 ViewPath 이름에서 **논리 이름**만을 반환 받도록 설정 필요 → 중복되는 경로와 파일 확장명을 공통처리 해줄 필요 있음 (추가로, 경로를 변경할 때 공통 처리부분에서만 변경해주면 됨)

![]({{site.baseurl}}/images/posts/post-220504/Untitled 3.png)

- MyView 객체에서 Model이 추가된 ModelView를 사용함으로써 Servlet의 의존성에서 벗어나고, viewResolver를 통해 Controller에서는 논리이름만 사용하도록 설정
- ModelView → `public class ModelView`
    - String viewName : view path를 저장 (논리 이름만을 저장, 추후 viewResolver 사용)
    - Map<String, Object> model : rendering할 떄 사용 되는 변수들 저장 ( request를 받지 않기 때문에 request.setAttribute 대신 사용. )
- ControllerV3 (interface) → `public interface ControllerV3`
    - `ModelView process(Map<String, String> paramMap);` : request, response를 사용하지 않아도 됨 (request → Model로 대체, response는 View로 대체. 즉, Servlet을 ModelView로 대체함으로써 Servlet 의존성 제거)
- ControllerV3 (interface)를 구현하는 각 Controller들
    - 이전 버전과 역시 비지니스 로직은 동일. but ViewPath를 저장할 때는 논리이름으로,  Save와 List 등 reqeust.getParam이 사용되는 부분은 넘겨 받은 paramMap을 통해 처리(get) → (request를 사용하지 않으면서 Servlet의 의존성 제거), request.setParam이 사용되는 부분은 ModelView의 Model을 이용. (Model에 저장하여 front controller에 반환)
- FrontController
    - 이전과 다르게 controller에 request,response가 사용되는 것이 아니라 frontController 안에서 request에서 넘겨오는 값들을 따로 조회하여 **paramMap**에 저장한 후 이를 각 controller에 보내고 있음
    - 또한, 각 controller는 Model 과 View의 정보를 담은 **ModelView** 객체를 반환. 결국 frontController는 이를 받아 앞선 로직과 동일하게 처리해 줘야 됨.
    - 추가로, ModelView의 viewPath는 논리이름으로만 저장하고 있기에 이를 따로 처리해줄 **viewResolver** 필요
    - front controller의 처리 로직
        
        ```java
        Map<String, String> paramMap = createParamMap(request);
        ModelView mv = controller.process(paramMap);
        String viewName = mv.getViewName(); // 논리 이름 (ex. new-form)
        MyView view = viewResolver(viewName);
        view.render(mv.getModel(), request, response); 
        ```
        
    - paramMap을 생성하는 createParamMap() method
        
        ```java
        private Map<String, String> createParamMap(HttpServletRequest request) {
            Map<String, String> paramMap = new HashMap<>();
            request.getParameterNames().asIterator()
                    .forEachRemaining(paramName -> paramMap.put(paramName, request.getParameter(paramName)));
            return paramMap;
        }
        ```
        
        - request에서 들어온 모든 파라미터 값들을 paramMap에 저장
        - 각 controller는 이를 이용하여 request 없이 parameter 사용 가능
    - 논리 view path를 물리 viewpath로 변경해주는 viewResolver
        
        ```java
        private MyView viewResolver(String viewName) {
            return new MyView("/WEB-INF/views/" + viewName + ".jsp");
        }
        ```
        
        - 각 Controller는 viewResolver 덕분에 중복되고 길었던 물리이름말고 간단한 논리이름만 ModelView 객체에 저장해주면 됨
    - forward 시 사용될 값들을 넣어주며 동작하는 `view.render()`
        
        ```java
        public void render(Map<String, Object> model, HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
            model.forEach(request::setAttribute); // 기존에는 각각의 controller 안에서 request setAttribute를 통해 설정함. but V3에서는 각각의 controller가 servlet에 의존하지 않기 때문에 render에서 받아온 paramMap을 통해 setAttribute 실시
            RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath); // 해당 viewPath로 이동하는 역할
            dispatcher.forward(request, response); //작동. 서블릿에서 jsp를 호출하는 기능, 서버 내부에서 다시 호출 발생 (redirect가 아님! 메서드 호출하듯이 서버끼리 노는 것!)
        }
        ```
        
        - ModelView의 Model의 값들을 request에 넣어줌으로써 forward 시 해당 값을 JSP에서 사용할 수 있게 설정 (JSP 특성 상 해주는 부분)

### 단순하고 실용적인 컨트롤러 - ver 4

- ver 3 는 아키텍쳐 구조적으로는 좋음. but 개발하는 개발자가 단순하고 편리하게 사용할 수가 없음. 즉, 실용성이 없음

![]({{site.baseurl}}/images/posts/post-220504/Untitled 4.png)

- 기본적인 구조는 v3와 동일. but Controller가 ModelView를 반환하지 않고, ViewName만을 반환 → 실용성 UP
- ControllerV4 (interface) → `public interface ControllerV4`
    - ModeView를 반환하는 것이 아닌 String (viewName)을 반환 → `String process(``Map<String, String> paramMap, Map<String, Object> model)`  → 여기서의 model은 frontController에서 설정해서 인자로 넘겨주는 것, 각 Controller는 받아온 이 model에 값을 저장해주면 됨. (즉, ModelView 객체를 사용할 필요가 없어짐)
- ControllerV4 (interface)를 구현한 각 Controller 들
    - 기존에는 ModelView를 생성해서 view name을 넣어주고, ModelView의 model에 값들을 저장 → 그냥 받아 온 인자 **model에 값을 넣어주고 view name을 반환**해주면 됨
        
        ```java
        // ex) save controller의 process
        model.put("member", member);
        return "save-result";
        ```
        
- FrontController
    - viewName은 각 Controller의 반환값으로 받아 오고 model은 현재 frontController에서 초기화 해준 후 각 Controller로 넘겨 값을 받아 오면 됨
        
        ```java
        Map<String, String> paramMap = createParamMap(request);
        Map<String, Object> model = new HashMap<>(); // model 생성
        
        String viewName = controller.process(paramMap, model); // controller 자체에서 viewname 반환
        MyView view = voewResolver(viewName);
        
        view.render(model, request, response); // controller 내부의 model 사용
        ```
        
    - 하지만 V4의 단점은. ConrollerV4에만 한정되어 있음. 즉, 내가 어떤 url에 대해서 V3를 쓰고 싶은데 현재는 V4로만 고정되어 있기 때문에 **유연하게 사용하지 못함** → **Handler 개념** 도입!

### 유연한 컨트롤러 - v5

<aside>
💡 <strong>현재 Spring Boot와 가장 유사한 구조를 보이고 잇는 버전</strong>

</aside>

- ControllerV3와 ControllerV4를 유연하게 변경하며 사용할 수 있도록
- 어댑터 패턴
    - 지금까지는 한가지 방식의 컨트롤러 인터페이스만 사용 가능 했음
    - but, **여러가지의 컨트롤러 인터페이스**를 사용할 수 있어야 함
    - 따라서 이렇게 여러가지가 호활될 수 있도록 **어댑터 패턴**을 적용해야 됨

  ![]({{site.baseurl}}/images/posts/post-220504/Untitled 5.png)
    
    - 핸들러 어댑터 목록을 통해 현재 요청이 들어온 controller를 처리할 수 있는 어댑터를 조회
    - 그래서 해당 요청에 대한 컨트롤러를 호출할 때, 직접 호출하는 것이 아닌 이를 **호환해주는 핸들러 어댑터를 통해 호출**
- **Handler Adaptor (interface)** → `public interface MyHandlerAdapter`
    - `boolean supports(Object handler);`
        - 요청에 따른 컨트롤러를 다룰 수 있는지, 즉, 해당 컨트롤러를 사용하게끔 하는 어댑터가 있는지 확인
    - `ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws ServletException, IOException;`
        - 실제 어댑터의 역할을 하는 메서드.
        - 실제 컨트롤러를 호출하고, 그 결과로 ModelView를 반환
        - 이전과 다르게 직접 컨트롤러를 호출하는 것이 아닌, **어댑터를 통해 호출**
    - 즉, Front Controller에서 요청을 받으면, 그 요청에 맞는 Controller를 찾고 해당 Controller를 지원하는 지 Handler Adaptor를 통해 확인(`supports`). 만약 지원한다면 그 해당 Adaptor를 통해 Controller에 접근(조회)하여 로직을 수행하고 ModelView를 반환하는 것 (`handle`)
- ControllerV3를 지원하는 Handler → `public class ControllerV3HandlerAdapter implements MyHandlerAdapter`
    - **supports**
        - front controller에서 받아온 handler(요청된 controller)가 ControllerV3이면 true
        
        ```java
        public boolean supports(Object handler) {
            return (handler instanceof ControllerV3);
        }
        ```
        
    - **handler**
        - ControllerV3에 맞는 비지니스 로직 수행
        - frontController에서 supports로 V3인지 확인 후 가동된다고 생각하면 됨, 그래서 바로 cascade 하는 것
        
        ```java
        public ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws ServletException, IOException {
        	  ControllerV3 controller = (ControllerV3) handler; 
        	  Map<String, String> paramMap = createParamMap(request);
        	  ModelView mv = controller.process(paramMap);
        	  return mv;
        }
        ```
        
- ControllerV4를 지원하는 Handler → `public class ControllerV4HandlerAdapter implements MyHandlerAdapter`
    - **supports**
        - front controller에서 받아온 handler(요청된 controller)가 ControllerV4이면 true
        
        ```java
        public boolean supports(Object handler) {
            return (handler instanceof ControllerV4);
        }
        ```
        
    - **handler**
        - ControllerV4에 맞는 비지니스 로직 수행
        - frontController에서 supports로 V4인지 확인 후 가동된다고 생각하면 됨, 그래서 바로 cascade 하는 것
        
        ```java
        public ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws ServletException, IOException {
            ControllerV4 controller = (ControllerV4) handler;
            Map<String, String> paramMap = createParamMap(request);
            HashMap<String, Object> model = new HashMap<>();
        
        		// adaptor가 필요한 이유 (controller의 반환값이 버전마다 다름)
            String viewName = controller.process(paramMap,model);
            mv.setModel(model); // 여기서 처리해주면 됨 -> adpator의 역할
        		
            return mv;
        }
        ```
        
        <aside>
        💡  <strong> adaptor가 꼭 필요한 이유 : V4 와 V3의 스펙이 다르면서 생기는 문제 -> adaptor로 해결 가능 </strong><br>
        <ul>
        <li>V3는 model 과 view 정보를 담는 ModelView 객체를 사용</li>
        <li>V4는 front controller에서 model, view 정보 setting을 다함 → V4는 ModelView를 사용하지 않기 때문 (`ModelView mv = new ModelView(viewName);`)</li>
        </ul>
        </aside>
        
- FrontController
    - Mapping Info 초기화
        
        ```java
        public FrontControllerServletV5() {
            initHandlerMappingMap();
            initHandlerAdapters();
        }
        ```
        
    - 이전과 다르게 Controller Mapping Map에 ControllerV4 뿐만 아니라 ControllerV3까지 mapping이 가능해야 됨 (ControllverV4 → Object) → `private final Map<String, Object> handlerMappingMap = new HashMap<>();` , v3,v4 매핑 정보를 넣어줌 ()
        
        ```java
        private void initHandlerMappingMap() {
            handlerMappingMap.put("front-controller/v5/v3/members/new-form", new MemberFormControllerV3());
            handlerMappingMap.put("front-controller/v5/v3/members/save", new MemberSaveControllerV3());
            handlerMappingMap.put("front-controller/v5/v3/members/members", new MemberListControllerV3());
        
            // V4 추가
            handlerMappingMap.put("front-controller/v5/v4/members/new-form", new MemberFormControllerV4());
            handlerMappingMap.put("front-controller/v5/v4/members/save", new MemberSaveControllerV4());
            handlerMappingMap.put("front-controller/v5/v4/members/members", new MemberListControllerV4());
        }
        ```
        
    - 또한 어댑터들에 대한 정보들을 넣어줘야 함. (ControllerV3, ControllerV4 를 지원하는 adapter들의 정보를 List로 저장) →  `private final List<MyHandlerAdapter> handlerAdapters = new ArrayList<>();`
        
        ```java
        private void initHandlerAdapters() {
            handlerAdapters.add(new ControllerV3HandlerAdapter());
            handlerAdapters.add(new ControllerV4HandlerAdapter());
        }
        ```
        
    - **service**
        - 요청된 URL에 맞는 Controller(Handler) 조회 (Mapping 된 Map 이용)
            
            ```java
            String requestURI = request.getRequestURI(); // 요청된 URL의 정보를 얻어 올 수 있음
            return handlerMappingMap.get(requestURI); // 요청된 URL에 따른 Controller 사용. (이 부분 때문에 Interface를 사용한 것 -> 다형성)
            ```
            
        - 해당 Controller(handler)를 지원하는 Adaptor 조회 (Adaptor 목록을 돌리면서 support를 통해 지원 여부 확인)
            
            ```java
            for (MyHandlerAdapter adapter : handlerAdapters) {
                if (adapter.supports(handler)) {
                    return adapter;
                }
            }
            ```
            
        - 지원 가능한 Adaptor를 통해 요청된 Controller(handler) 조회 및 로직 수행 후 ModelView 반환, 해당 ModelView의 정보를 통해 view forward 수행
            
            ```java
            ModelView mv = adapter.handle(request,response,handler);
            String viewName = mv.getViewName(); // 논리 이름 (ex. new-form)
            MyView view = viewResolver(viewName);
            
            view.render(mv.getModel(), request, response);
            ```
            
    
    <aside>
    💡 <strong> V3에서 V4까지 지원하게 끔 확장할 때, main 코드를 거의 건들이지 않고, init 부분에 해당 버전에 대한 정보들(Mapping)만 넣어줌 → OCP를 지킬 수 있음! </strong>
    
    </aside>
    
    <aside>
    💡 <strong>이 구조가 결국 Spring MVC에서 구축된 구조!!! Spring MVC는 이 구조를 Annotation 기반으로 훨씬 더 쉽게 이용할 수 있게 설정해 둠</strong>
    
    </aside>