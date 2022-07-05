---
layout: post
title:  "Login with Cookie, Session"
date:   2022-05-27
image:  /posts/cookie_session.png
tags:   SpringBoot Login
---
<aside>
💡 <strong>프로젝트 구조 팁 (아키텍쳐) [도메인과 Web을 잘 구분하라!]</strong>
<ul>
<li>도메인 : 화면, UI, 기술 인프라 등의 영역을 제외한 <b>시스템이 구현해야 하는 핵심 비즈니스 업무 영역</b> (<code>Entity</code>, <code>Service</code>, <code>Repository</code>, … ) </li>
<li>Web : 화면, UI, 기술 인프라, 네트워크 측면의 영역 (<code>DTO</code>, <code>Controller</code>, …) </li>
<li>향후 web을 API로 바꾸든, Spring이 아닌 프레임워크로 바꾸든 도메인은 그대로 유지할 수 있어야 한다. (<b>즉, web을 바꿔도 domain은 변경할 점이 없어야 함</b>) </li>
<li>이는 web은 domain을 알고(의존하고) 있지만, domain은 web을 모르도록(의존하지 않도록) 설정해야됨 <b>[단방향 설계]</b> </li>
<li>즉, web은 domain을 참조하고 있지만, domain은 web을 참조하고 있으면 안됨!! <b>[단방향 설계]</b> </li>
</ul>
</aside>

## 로그인 기능 구현

### 기본 로그인 Repository, Service, Controller

- 로그인 저장소(Repository) 구현 → `MemberRepository` (도메인)
    
    ```java
    @Slf4j
    @Repository
    public class MemberRepository {
    	 
    	 private static Map<Long, Member> store = new HashMap<>(); //static 사용
    	
    	 ...
    	 public Optional<Member> findByLoginId(String loginId) {
    		 return findAll().stream()
    		 .filter(m -> m.getLoginId().equals(loginId))
    		 .findFirst();
    	 }
    	 ...
    }
    ```
    
    - `Optional<Member> findByLoginId` : `findAll()`을 통해 불러온 모든 `Member`들을 stream으로 돌리면서 받아 온 `loginId` 와 일치하는 `Member`가 있는지 판단하여 반환.
    - 있으면 `Optional<Member>`, 없으면 `Optional<null>`
- 로그인 비지니스 로직(Service) 구현 → `LoginService` (도메인)
    
    ```java
    @Service // Component로 등록
    @RequiredArgsConstructor // DI 편의 사용
    public class LoginService {
    
    	 private final MemberRepository memberRepository;
    
    	 /**
    	 * @return null이면 로그인 실패
    	 */
    	 public Member login(String loginId, String password) {
    		 return memberRepository.findByLoginId(loginId)
    		 .filter(m -> m.getPassword().equals(password))
    		 .orElse(null);
    	 }
    }
    ```
    
    - `public Member login` : `Optional<Member>` (`memberRepository.findByLoginId`의 반환)로 `loginId`를 검색한 후 해당 `Member`가 인자로 받아온 `password`와 일치하는 `password`를 가지고 있는지 판단 (`filter` 와 `orElse` 사용)
    - 있다면 Member 반환, 없다면 null 반환
- 로그인 Controller (Web)
    - 로그인 폼 전달 Controller
        
        ```java
        @GetMapping("/add")
        public String addForm(@ModelAttribute("member") Member member) {
        	return "members/addMemberForm";
        }
        ```
        
        - 빈 Member를 `@ModelAttribute`를 통해 넘겨줌
        - form의 object, field를 맞춰주기 위함
    - 로그인 폼 등록 Controller
        
        ```java
        @PostMapping("/add")
        public String save(@Valid @ModelAttribute Member member, BindingResult result) {
        	
        	if (result.hasErrors()) {
        		return "members/addMemberForm";
        	}
        	
        	memberRepository.save(member);
        	return "redirect:/";
        }
        ```
        
        - Valid 를 통해 **Bean Validator 를 통한 검증** 수행
        - Validation 통과 못할 시 다시 등록 폼으로
        - 통과되었다면 저장 후 redirect **(PRG)**

> 이렇게 로그인 기능이 잘 동작하는 것을 확인했지만, 로그인 된 사용자가 누구인지, 또 해당 로그인된 사용자가 다른 페이지로 이동해도 로그인의 상태가 유지될 수 있도록 설정이 필요함!!! → ( using cookie, session )
> 

## 로그인, 쿠키 적용

### Login with Cookie

- **쿠키**를 사용해서 로그인, 로그아웃 기능 구현
- 로그인 **상태를 유지**하고 **로그인 회원이 누군지 알 수 있도록 하기 위해 사용** (물론 쿼리 파라미터를 계속 유지하며 해당 기능을 수행할 수는 있지만, 너무 어렵고 번거로움)
- 쿠키 (Cookie)
    - **서버에서 쿠키 생성**

        ![]({{site.baseurl}}/images/posts/post-220527/Untitled.png)

        - 서버에서 로그인에 성공하면 **HTTP 응답에 쿠키를 담아**서 브라우저에 전달 (여기서는 memberId를 쿠키로 보내준다고 가정. (보안상의 문제는 뒤에서 다룸))
        - 해당 쿠키를 **쿠키 저장소에 저장**
    - **클라이언트가 서버로 쿠키 전달**

        ![]({{site.baseurl}}/images/posts/post-220527/Untitled 1.png)

        - 그 후 브라우저는 계속 해당 쿠키를 지속해서 보내줌
        - 어떤 위치든 간에 해당 사이트에선 계속 **해당 쿠키를 지속해서 보내줌**
        - 그럼 **서버**에서는 이값을 **활용**해서 **사용자 인증** 진행
    
    <aside>
    💡 <strong>쿠키 종류</strong>
    <ol>
    <li><b>영속 쿠키</b> : 만료 날짜를 입력하면 해당 날짜까지 유지 </li>
    <li><b>세션 쿠키</b> : 만료 날짜를 생략하면 부라우저 종료시 까지만 유지 (Http Session 과는 다른 개념) </li>
    </ol>
    </aside>
    
- 쿠키 생성 및 전달
    
    ```java
    Cookie idCookie = new Cookie("memberId", String.valueOf(loginMember.getId()));
    response.addCookie(idCookie);
    ```
    
    - 로그인 성공 로직에 쿠키를 `memberId = loginMember.getId()` 형식으로 넣어주고 보내주기
    - 이렇게 되면 로그인 성공 시 **쿠키**에 **memberId = 1** 이런 형식으로 저장되어 있는 것을 확인할 수 있고, 추가로 클라이언트가 서버에 요청을 보낼때도 항상 쿠키가 포함되어 있는 것을 확인할 수 있음
        - 쿠키 전달 (응답 값에 생성된 쿠키가 전달되는 것 확인 가능)

          ![]({{site.baseurl}}/images/posts/post-220527/Untitled 2.png)
            
        - 추가로 다른 페이지를 요청할 때, **요청 헤더**에 해당 **쿠키가 담겨있는 것**을 확인할 수 있음
- 쿠키에 따른 회원 로그인 처리 (`HomeController.homeLogin`)
    
    ```java
    @GetMapping("/")
    public String homeLogin(@CookieValue(name = "memberId", required = false) Long memberId, Model model) {
    
    		// 쿠키 만료 or 로그인 되지 않음
        if (memberId == null) {
            return "home";
        }
    
        // 로그인
        Member loginMember = memberRepository.findById(memberId);
        if (loginMember == null) {
            return "home";
        } // 실패
    
        model.addAttribute("member", loginMember);
        return "loginHome";
    }
    ```
    
    - `@CookieValue` : **memberId로 되어 있는 쿠키** 가져오기, `required = false`로 값이 없어도 괜찮다는 것 (로그인 하지 않은 회원도 접근하도록)
    - 해당 값을 통해 Member를 찾아오고 해당 Member의 정보를 포함한 Home 화면을 보여줌
- 쿠키를 통한 회원 로그아웃 처리(`HomeController.homeLogout`)
    
    ```java
    @PostMapping("/logout")
    public String logout(HttpServletResponse response) {
        expireCookie(response, "memberId");
        return "redirect:/";
    }
    
    private void expireCookie(HttpServletResponse response, String cookieName) {
        Cookie cookie = new Cookie(cookieName, null);
        cookie.setMaxAge(0); // 죽이는 것
        response.addCookie(cookie); // 중복된 이름으로 추가
    }
    ```
    
    - 이름이 동일한 **쿠키의 기한을 0으로 설정**하여 새로 추가해줌으로써 해당 쿠키를 만료 시키는 것

<aside>
🚨 <strong>BUT!! 이 방식은 보안상의 큰 문제점이 있음!</strong>

</aside>

### Cookie의 보안 문제

- **보안 문제**
    - 쿠키 값은 **사용자가 임의로 변경** 가능
        - 쿠키는 **클라이언트에서 서버로 전송하는 값**, 그렇기에 언제든 변경 가능! (개발자 모드/Application/Cookie 에서 변경 가능)
        - memberId 를 임의로 2로 설정해버리면 로그인 하지 않고도 memberId 가 2인 Member로 접근이 가능해지는 것
    - 쿠키에 보관된 정보는 **누구든 볼 수 있음**
        - 만약 쿠키에 개인정보 등이 있다면? 굉장히 문제가 커짐
        - **웹 브라우저에도 보관**되고, 네트워크 요청마다 서버로 전송됨 (이 때마다 정보를 훔쳐 볼 수 있는 것)
        - 결국 쿠키의 정보로 인해 나의 로컬PC가 털릴 수도 있고, 네트워크 전송 시에 털릴 수도 있는 것
    - 쿠키를 한번 알면 평생 사용할 수 있음
        - 쿠키의 값을 **일정한 값으로 사용하면** 한번 누가 훔쳐가면 그 정보를 통해 **계속해서 접근이 가능**함
- 대안
    - 쿠키에 중요한 값을 노출하지 않고, 사용자 별로 **예측 불가능한 임의의 토큰(랜덤 값)**을 노출하고, **서버에서 토큰과 사용자 id를 매핑**해서 인식. 또한 **서버에서 토큰을 관리**
    - **토큰은 예상 불가능** 해야 됨
    - 서버에서 해당 **토큰의 만료시간을 짧게** 유지 필요. 또한 해킹이 의심되는 경우 서버에서 해당 토큰을 강제로 제거 후 새로 생성하는 방식으로 진행
    - 하지만 **클라이언트와 서버는 결국 쿠키로 연결**이 되어야 됨! ⇒ **쿠키에  Session 개념 도입**
    

## 로그인, 세션 적용

### Login with Session

- 목표
    - **쿠키**를 사용했을 때의 **보안 이슈 해결**
    - 중요한 정보를 **모두 서버에 저장**
    - 클라이언트와 서버는 **추정 불가능한 임의의 식별자 값**으로 연결
    
    ⇒ **Session 도입**
    
- 세션 (Session)
    - 로그인

      ![]({{site.baseurl}}/images/posts/post-220527/Untitled 3.png)
        
        - 클라이언트에서 주어진 로그인, 비밀번호를 통해 Member 조회
    - 세션 생성

      ![]({{site.baseurl}}/images/posts/post-220527/Untitled 4.png)
        
        - 해당 Member에 대한 sessionId 생성
        - 세션 ID는 **추정 불가능한 UUID 값**으로 설정
        ex) `Cookie: mySessionId=zz0101xx-bab9-4b92-9b32-dadb280f4b61`
        - 생성된 세션 ID와 해당 Member를 **서버의 세션 저장소에 보관**
    - 세션 생성 및 전달 (**응답 쿠키**로. **Serve → Client**)

      ![]({{site.baseurl}}/images/posts/post-220527/Untitled 5.png)
        
        - 서버는 클라이언트에 **sessionId값만을 쿠키에 담아서 전달**
        - 클라이언트는 쿠키 저장소에 해당 쿠키(세션값)를 보관
        - 결국 회원과 관련된 정보는 전혀 클라이언트에 전달되지 않고 오직 **추정 불가능한 세션 ID만이 쿠키를 통해 전달**됨
    - 세션Id 쿠키 전달 (**요청 쿠키**로. **Client → Server**)

      ![]({{site.baseurl}}/images/posts/post-220527/Untitled 6.png)
        
        - **클라이언트**는 요청 시 항상 **mySessionId 쿠키를 전달**
        - **서버**에서는 해당 mySissionId 쿠키 정보로 **서버 내의 세션 저장소를 조회**해서 로그인 시 보관한 세션 정보 사용
- [쿠키 → **세션**] **개선점**
    - 쿠키 값 변조 후 접근 → UUID를 통해 **예측 불가능한 쿠키 값**
    - 쿠키값을 통한 정보 해킹 → 세션 Id, 즉 드러나 있는 정보에는 **개인정보 및 중요 정보가 없음**
    - 쿠키 탈취 후 사용 → 서버에서 세션의 **만료시간을 짧게 유지 및 세션 강제 제거**

- 세션 직접 개발
    - Spring에서 제공하는 Session인 `HttpSession` **을 사용하지 않고** 원리를 이해하기 위해 직접 session 개발
    - 세션 생성, 조회, 만료(제거) → `SessionManager`
        - 세션 저장소
            - `private Map<String, Object> sessionStore = new ConcurrentHashMap<>();`
            - 동시성 문제로 `ConcurrentHashMap` 사용
        - 세션 생성
            
            ```java
            public void createSession(Object value, HttpServletResponse response) {
            	 //세션 id를 생성하고, 값을 세션에 저장
            	 String sessionId = UUID.randomUUID().toString();
            	 sessionStore.put(sessionId, value);
            	 //쿠키 생성
            	 Cookie mySessionCookie = new Cookie("mySessionId", sessionId);
            	 response.addCookie(mySessionCookie);
            }
            ```
            
            - UUID 를 통한 추정 불가능한 임의의 `sessionId` 생성
            - **세션 저장소(**`sessionStore`**)**에 `sessionId`와 보관할 값(`value`) 저장
            - `sessionId`로 응답 쿠키(`mySessionCookie`)를 생성해서 클라이언트에 전달(`response.addCookie(mySessionCookie)`)
        - 세션 조회
            
            ```java
            public Object getSession(HttpServletRequest request) {
            	 Cookie sessionCookie = findCookie(request, SESSION_COOKIE_NAME);
            	 if (sessionCookie == null) {
            		 return null;
            	 }
            	 return sessionStore.get(sessionCookie.getValue());
            }
            ```
            
            - findCookie를 통해 해당 request에서 해당 이름을 가진 cookie를 가져옴
            - 해당 쿠키가 비었으면 null 반환, 쿠키가 존재하면 해당 쿠키의 mySessionId를 통해 **저장소에서 매핑된 Object(Member) 반환**
        - 세션 만료
            
            ```java
            public void expire(HttpServletRequest request) {
            	 Cookie sessionCookie = findCookie(request, SESSION_COOKIE_NAME);
            	 if (sessionCookie != null) {
            		 sessionStore.remove(sessionCookie.getValue());
            	 }
            }
            ```
            
            - findCookie를 통해 해당 request에서 해당 이름을 가진 cookie를 가져옴
            - 해당 쿠키의 mySessionId를 통해 **저장소에서 해당 정보를 제거** → 서버 내부 저장소에서만 삭제해주면 쿠키에 mySessionId가 있더라도 **매핑된 정보가 없기에 해당 Session을 이용할 수 없음**
    - 직접 만든 세션 적용
        - `private final SessionManager sessionManager;` 주입 (**직접 개발한 Session 관리자**)
        - Login [`LoginController` 의 login 처리 부분 (**쿠키 → 세션** 으로 변경)]
            - 세션 관리자를 통해 **세션 생성 및 회원 데이터 보관** (세션 저장소에), 세션 ID를 바탕으로 **쿠키 생성 및 전달**
            - `Cookie idCookie = new Cookie("memberId", String.valueOf(loginMember.getId())); response.addCookie(idCookie);` **→** **`sessionManager.createSession(loginMember, response)`**
            - 직접 만든 sessionId로 쿠키가 잘 전달되는 것을 확인할 수 있음

              ![]({{site.baseurl}}/images/posts/post-220527/Untitled 7.png)
                
            - 또한, 요청 시에도 해당 쿠키로 잘 요청이 들어오는 것 확인 가능
        - LogOut [`LoginController` 의 logout 처리 부분 (**쿠키 → 세션** 으로 변경)]
            - 로그 아웃시 **해당 세션의 정보를 제거**
            - `expireCookie(response, "memberId");` → **`sessionManager.expire(request);`**
            - 특징 : 쿠키를 직접 삭제하는 것이 아닌 **세션저장소의 해당 세션 정보만을 삭제**하는 것 (세션 저장소에서 삭제하면 더 이상 해당 세션ID는 사용 불가!)
        - Home [`HomeController`의 `@GetMapping(”/”)`]
            
            ```java
            @GetMapping("/") // Session 이용
            public String homeLoginV2(HttpServletRequest request, Model model) {
            
                // 세션 관리자에 저장된 회원 정보 조회
                Member member = (Member) sessionManager.getSession(request);
            
                // 로그인
                if (member == null) {
                    return "home";
                } // 실패
            
                model.addAttribute("member", member);
                return "loginHome"; // 성공
            }
            ```
            
            - `@CookieValue` → **`HttpServletRequest`, `(Member) sessionManager.getSession(request);`** : request를 통해 cookie의 **sessionId에 매핑된 Object(Member)** 를 가져옴 (서버 내의 **세션 저장소에서**)
    
    <aside>
    ⚠️ <strong>세션 직접 개발에서 알아야 할 점</strong>
    <ul>
    <li>사실 세션이라는 것이 뭔가 특별한 것이 아니라 단지 <b>쿠키를 사용</b>하는데, <b>서버에서 데이터를 유지하는 방법</b>일 뿐 </li>
    <li>하지만 이렇게 직접 개발하면 굉장히 복잡해진 것을 알 수 있음 </li>
    <li>그래서 <b>서블릿에서는 공식적으로 세션을 지원</b>함 → <b><code>HttpSession</code></b> </li>
    <li><code>HttpSession</code>은 우리가 직접 만든 세션과 동작 방식이 거의 동일!! </li>
    </ul>
    </aside>
    

### Login with “Servlet HTTP Session(HttpSession)”

- 직접 개발했던 Session의 개념은 이미 **서블릿이 `HttpSession` 이라는 기능으로 제공**
- 직접 구현했던 로직과 거의 동일하며 더 편리하게 사용할 수 있도록 구현되어 있음
- `HttpSession`
    - 직접 만들어 보았던 SessionManger와 같은 방식으로 동작
    - JSESSIONID 라는 이름의 쿠키 생성, 값은 추정 불가능 → `Cookie: JSESSIONID=5B78E23B513F50164D6FDD8C97B0AD05`
- HttpSession 사용 **[Version 1]**
    - Login [`LoginController` 의 login 처리 부분 (직접 개발한 세션 → HttpSession 이용)]
        - `sessionManager.createSession(loginMember, response)` → **`HttpSession session = request.getSession();` , `session.setAttribute(SessionConst.LOGIN_MEMBER, loginMember)`**
        - `request.getSession(true)` : 세션이 있으면 기존 세션 반환, 없으면 새로운 세션을 생성해서 반환 (false면 새로운 세션 반환 X, null 반환)
        - `session.setAttribute(...)` : 세션에 **데이터 보관**. 하나의 세션에 key, value 로 여러 값 저장 가능 (세션 저장소 이용)
        - 즉, 기존 세션이 있다면 그 세션을 가져와서 내가 저장하고 싶은 **데이터를 저장** (**메모리 사용**). 기존의 유일한 아이디를 가지고 있는 **JSESSIONID에 해당 데이터들이 매핑**되는 것 (이전에는 직접 sessionId, 저장소를 생성하고 아이디와 객체를 매핑함)
        - `HttpSession`을 이용하여 JSESSIONID로 쿠키가 잘 전달된 것을 확인할 수 있음

          ![]({{site.baseurl}}/images/posts/post-220527/Untitled 8.png)
            
    - LogOut [`LoginController` 의 logout 처리 부분 (직접 개발한 세션 → HttpSession 이용)]
        - `sessionManager.expire(request);` → **`HttpSession session = request.getSession(false);` , `if (session != null) { session.invalidate(); }`**
        - `HttpSession session = request.getSession(false);` : 세션을 생성하지 않고 있다면 세션 반환
        - `session.invalidate()` : 세션 제거
            - 이전과 동일하게 **해당 세션에 매핑된 모든 데이터를 삭제**하는 것. (쿠키를 삭제하지 않음. 굳이 삭제할 필요가 없기 때문)
    - Home [`HomeController`의 `@GetMapping(”/”)`]
        
        ```java
        HttpSession session = request.getSession(false);
        
        if (session == null) {
        	return "home";
        }
        
        Member loginMember = (Member) session.getAttribute(SessionConst.LOGIN_MEMBER);
        ```
        
        - `session.getAttribute(SessionConst.LOGIN_MEMBER);` : 가져온 세션에서 key와 value로 저장했었던 그 key로 Member 객체를 가져옴
- HttpSession **[Version2]**
    - `@SessionAttribute`
        - 스프링이 세션을 더 편리하게 사용할 수 있도록 지원하는 기능
        - 물론 쿠키를 생성하고 하는 부분은 직접 request에서 session을 가져와 setAttribute로 진행해야됨.
        - 해당 기능은 **session에 저장된 데이터를 가져올 때 편리**하게 사용 가능!!
    - Home [`HomeController`의 `@GetMapping(”/”)`]
        - 이전 버전에서 request에서 **session을 가져오고 session에 저장된 데이터를 가져오**는 부분이 **단 한줄로 끝낼 수 있음**
        - `@SessionAttribute(name = SessionConst.LOGIN_MEMBER, required = false) Member loginMember`
        - 세션이 있다면 그 세션과 매핑된 데이터들 중`SessionConst.LOGIN_MEMBER` 에 해당 하는 Member 객체를 가져오는 역할
        - `required = false` 를 통해 로그인되지 않은 회원에 대해서도 받아줌
        - 세션이 없거나, 해당 데이터에 맞는 Member가 없을 경우 null

### Session Info and Timeout

- HttpSession에서 제공하는 정보
    - `session.getAttribute()` : 세션에 보관되어 있는 데이터
    - `session.getId()` : 세션Id, JSESSIONID의 값
    - **`session.getMaxInactiveInterval()`** :  **[중요]** 세션의 **유효 시간**. 기본값 : 1800초
    - `session.getCreationTime()` : 세션 생성일시
    - **`session.getLastAccessedTime()`** : **[중요]** 세션과 연결된 **사용자가** **최근에 서버에 접근한 시간**. 클라이언트에서 서버로 sessionId ( JSESSIONID )를 요청한 경우에 갱신됨
    - `session.isNew()` :  새로 생성된 세션인지, 아니면 이미 과거에 만들어졌고, 클라이언트에서 서버로 sessionId ( JSESSIONID )를 요청해서 조회된 세션인지 여부
- **세션 타임아웃 설정**
    - 세션 종료 시점은 사용자가 서버에 최근에 요청한 시간을 기준으로 30분 정도를 유지해주고 그 이후로 끊는 것이 좋은 대안. (사용자가 사용하고 있을 때 세션이 만료되지 않도록 하기 위함)
    - HttpSession 은 이 방식을 사용
    - 그렇다면 30분 말고 **직접 만료 시간을 설정**하고 싶다면? → **글로벌 설정** 이용
        - 스프링의 `application.properties` 에서 `server.servlet.session.timeout` 의 값을 설정해주면 됨 (기본 값이 1800_30분)
- 세션 타임아웃 발생
    - 세션의 타임아웃 시간은 해당 세션과 관련된 JSESSIONID 를 전달하는 HTTP 요청이 있으면 현재 시간으로 다시 초기화 됨
    - 즉, **요청이 들어왔다 하면 다시 처음부터 30분의 타이머를 돌리는 것**
    - 결국 이와 관련된 session info 는 **`session.getLastAccessedTime()` (최근 세션 접근 시간) , `session.getMaxInactiveInterval()` (세션의 유효 시간)**
    - 즉, `LastAccessedTime` 이후로 timeout 시간이 지나면, WAS가 내부에서 해당 세션을 제거하고 timeout 이전에 다시 요청이 들어오면 `LastAccessedTime`을 갱신하고 그 시점으로 다시 timeout 체크

<aside>
⚠️ <strong>실무에서 Session 사용 주의점!</strong> <br>
<b>세션</b>에는 <b>최소한의 데이터만 보관</b>해야 한다! <br>
보관한 데이터 용량 * 사용자 수로 세션의 메모리 사용량이 급격하게 늘어나서 장애로 이어질 수 있기 때문! 또한 세션의 시간을 너무 길게 잡으면 메모리 사용이 계속 누적! 적당한 시간 설정 필요함!

</aside>