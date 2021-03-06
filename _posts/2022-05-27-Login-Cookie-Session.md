---
layout: post
title:  "Login with Cookie, Session"
date:   2022-05-27
image:  /posts/cookie_session.png
tags:   SpringBoot Login
---
<aside>
๐ก <strong>ํ๋ก์ ํธ ๊ตฌ์กฐ ํ (์ํคํ์ณ) [๋๋ฉ์ธ๊ณผ Web์ ์ ๊ตฌ๋ถํ๋ผ!]</strong>
<ul>
<li>๋๋ฉ์ธ : ํ๋ฉด, UI, ๊ธฐ์  ์ธํ๋ผ ๋ฑ์ ์์ญ์ ์ ์ธํ <b>์์คํ์ด ๊ตฌํํด์ผ ํ๋ ํต์ฌ ๋น์ฆ๋์ค ์๋ฌด ์์ญ</b> (<code>Entity</code>, <code>Service</code>, <code>Repository</code>, โฆ ) </li>
<li>Web : ํ๋ฉด, UI, ๊ธฐ์  ์ธํ๋ผ, ๋คํธ์ํฌ ์ธก๋ฉด์ ์์ญ (<code>DTO</code>, <code>Controller</code>, โฆ) </li>
<li>ํฅํ web์ API๋ก ๋ฐ๊พธ๋ , Spring์ด ์๋ ํ๋ ์์ํฌ๋ก ๋ฐ๊พธ๋  ๋๋ฉ์ธ์ ๊ทธ๋๋ก ์ ์งํ  ์ ์์ด์ผ ํ๋ค. (<b>์ฆ, web์ ๋ฐ๊ฟ๋ domain์ ๋ณ๊ฒฝํ  ์ ์ด ์์ด์ผ ํจ</b>) </li>
<li>์ด๋ web์ domain์ ์๊ณ (์์กดํ๊ณ ) ์์ง๋ง, domain์ web์ ๋ชจ๋ฅด๋๋ก(์์กดํ์ง ์๋๋ก) ์ค์ ํด์ผ๋จ <b>[๋จ๋ฐฉํฅ ์ค๊ณ]</b> </li>
<li>์ฆ, web์ domain์ ์ฐธ์กฐํ๊ณ  ์์ง๋ง, domain์ web์ ์ฐธ์กฐํ๊ณ  ์์ผ๋ฉด ์๋จ!! <b>[๋จ๋ฐฉํฅ ์ค๊ณ]</b> </li>
</ul>
</aside>

## ๋ก๊ทธ์ธ ๊ธฐ๋ฅ ๊ตฌํ

### ๊ธฐ๋ณธ ๋ก๊ทธ์ธ Repository, Service, Controller

- ๋ก๊ทธ์ธ ์ ์ฅ์(Repository) ๊ตฌํ โ `MemberRepository` (๋๋ฉ์ธ)
    
    ```java
    @Slf4j
    @Repository
    public class MemberRepository {
    	 
    	 private static Map<Long, Member> store = new HashMap<>(); //static ์ฌ์ฉ
    	
    	 ...
    	 public Optional<Member> findByLoginId(String loginId) {
    		 return findAll().stream()
    		 .filter(m -> m.getLoginId().equals(loginId))
    		 .findFirst();
    	 }
    	 ...
    }
    ```
    
    - `Optional<Member> findByLoginId` : `findAll()`์ ํตํด ๋ถ๋ฌ์จ ๋ชจ๋  `Member`๋ค์ stream์ผ๋ก ๋๋ฆฌ๋ฉด์ ๋ฐ์ ์จ `loginId` ์ ์ผ์นํ๋ `Member`๊ฐ ์๋์ง ํ๋จํ์ฌ ๋ฐํ.
    - ์์ผ๋ฉด `Optional<Member>`, ์์ผ๋ฉด `Optional<null>`
- ๋ก๊ทธ์ธ ๋น์ง๋์ค ๋ก์ง(Service) ๊ตฌํ โ `LoginService` (๋๋ฉ์ธ)
    
    ```java
    @Service // Component๋ก ๋ฑ๋ก
    @RequiredArgsConstructor // DI ํธ์ ์ฌ์ฉ
    public class LoginService {
    
    	 private final MemberRepository memberRepository;
    
    	 /**
    	 * @return null์ด๋ฉด ๋ก๊ทธ์ธ ์คํจ
    	 */
    	 public Member login(String loginId, String password) {
    		 return memberRepository.findByLoginId(loginId)
    		 .filter(m -> m.getPassword().equals(password))
    		 .orElse(null);
    	 }
    }
    ```
    
    - `public Member login` : `Optional<Member>` (`memberRepository.findByLoginId`์ ๋ฐํ)๋ก `loginId`๋ฅผ ๊ฒ์ํ ํ ํด๋น `Member`๊ฐ ์ธ์๋ก ๋ฐ์์จ `password`์ ์ผ์นํ๋ `password`๋ฅผ ๊ฐ์ง๊ณ  ์๋์ง ํ๋จ (`filter` ์ `orElse` ์ฌ์ฉ)
    - ์๋ค๋ฉด Member ๋ฐํ, ์๋ค๋ฉด null ๋ฐํ
- ๋ก๊ทธ์ธ Controller (Web)
    - ๋ก๊ทธ์ธ ํผ ์ ๋ฌ Controller
        
        ```java
        @GetMapping("/add")
        public String addForm(@ModelAttribute("member") Member member) {
        	return "members/addMemberForm";
        }
        ```
        
        - ๋น Member๋ฅผ `@ModelAttribute`๋ฅผ ํตํด ๋๊ฒจ์ค
        - form์ object, field๋ฅผ ๋ง์ถฐ์ฃผ๊ธฐ ์ํจ
    - ๋ก๊ทธ์ธ ํผ ๋ฑ๋ก Controller
        
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
        
        - Valid ๋ฅผ ํตํด **Bean Validator ๋ฅผ ํตํ ๊ฒ์ฆ** ์ํ
        - Validation ํต๊ณผ ๋ชปํ  ์ ๋ค์ ๋ฑ๋ก ํผ์ผ๋ก
        - ํต๊ณผ๋์๋ค๋ฉด ์ ์ฅ ํ redirect **(PRG)**

> ์ด๋ ๊ฒ ๋ก๊ทธ์ธ ๊ธฐ๋ฅ์ด ์ ๋์ํ๋ ๊ฒ์ ํ์ธํ์ง๋ง, ๋ก๊ทธ์ธ ๋ ์ฌ์ฉ์๊ฐ ๋๊ตฌ์ธ์ง, ๋ ํด๋น ๋ก๊ทธ์ธ๋ ์ฌ์ฉ์๊ฐ ๋ค๋ฅธ ํ์ด์ง๋ก ์ด๋ํด๋ ๋ก๊ทธ์ธ์ ์ํ๊ฐ ์ ์ง๋  ์ ์๋๋ก ์ค์ ์ด ํ์ํจ!!! โ ( using cookie, session )
> 

## ๋ก๊ทธ์ธ, ์ฟ ํค ์ ์ฉ

### Login with Cookie

- **์ฟ ํค**๋ฅผ ์ฌ์ฉํด์ ๋ก๊ทธ์ธ, ๋ก๊ทธ์์ ๊ธฐ๋ฅ ๊ตฌํ
- ๋ก๊ทธ์ธ **์ํ๋ฅผ ์ ์ง**ํ๊ณ  **๋ก๊ทธ์ธ ํ์์ด ๋๊ตฐ์ง ์ ์ ์๋๋ก ํ๊ธฐ ์ํด ์ฌ์ฉ** (๋ฌผ๋ก  ์ฟผ๋ฆฌ ํ๋ผ๋ฏธํฐ๋ฅผ ๊ณ์ ์ ์งํ๋ฉฐ ํด๋น ๊ธฐ๋ฅ์ ์ํํ  ์๋ ์์ง๋ง, ๋๋ฌด ์ด๋ ต๊ณ  ๋ฒ๊ฑฐ๋ก์)
- ์ฟ ํค (Cookie)
    - **์๋ฒ์์ ์ฟ ํค ์์ฑ**

        ![]({{site.baseurl}}/images/posts/post-220527/Untitled.png)

        - ์๋ฒ์์ ๋ก๊ทธ์ธ์ ์ฑ๊ณตํ๋ฉด **HTTP ์๋ต์ ์ฟ ํค๋ฅผ ๋ด์**์ ๋ธ๋ผ์ฐ์ ์ ์ ๋ฌ (์ฌ๊ธฐ์๋ memberId๋ฅผ ์ฟ ํค๋ก ๋ณด๋ด์ค๋ค๊ณ  ๊ฐ์ . (๋ณด์์์ ๋ฌธ์ ๋ ๋ค์์ ๋ค๋ฃธ))
        - ํด๋น ์ฟ ํค๋ฅผ **์ฟ ํค ์ ์ฅ์์ ์ ์ฅ**
    - **ํด๋ผ์ด์ธํธ๊ฐ ์๋ฒ๋ก ์ฟ ํค ์ ๋ฌ**

        ![]({{site.baseurl}}/images/posts/post-220527/Untitled 1.png)

        - ๊ทธ ํ ๋ธ๋ผ์ฐ์ ๋ ๊ณ์ ํด๋น ์ฟ ํค๋ฅผ ์ง์ํด์ ๋ณด๋ด์ค
        - ์ด๋ค ์์น๋  ๊ฐ์ ํด๋น ์ฌ์ดํธ์์  ๊ณ์ **ํด๋น ์ฟ ํค๋ฅผ ์ง์ํด์ ๋ณด๋ด์ค**
        - ๊ทธ๋ผ **์๋ฒ**์์๋ ์ด๊ฐ์ **ํ์ฉ**ํด์ **์ฌ์ฉ์ ์ธ์ฆ** ์งํ
    
    <aside>
    ๐ก <strong>์ฟ ํค ์ข๋ฅ</strong>
    <ol>
    <li><b>์์ ์ฟ ํค</b> : ๋ง๋ฃ ๋ ์ง๋ฅผ ์๋ ฅํ๋ฉด ํด๋น ๋ ์ง๊น์ง ์ ์ง </li>
    <li><b>์ธ์ ์ฟ ํค</b> : ๋ง๋ฃ ๋ ์ง๋ฅผ ์๋ตํ๋ฉด ๋ถ๋ผ์ฐ์  ์ข๋ฃ์ ๊น์ง๋ง ์ ์ง (Http Session ๊ณผ๋ ๋ค๋ฅธ ๊ฐ๋) </li>
    </ol>
    </aside>
    
- ์ฟ ํค ์์ฑ ๋ฐ ์ ๋ฌ
    
    ```java
    Cookie idCookie = new Cookie("memberId", String.valueOf(loginMember.getId()));
    response.addCookie(idCookie);
    ```
    
    - ๋ก๊ทธ์ธ ์ฑ๊ณต ๋ก์ง์ ์ฟ ํค๋ฅผ `memberId = loginMember.getId()` ํ์์ผ๋ก ๋ฃ์ด์ฃผ๊ณ  ๋ณด๋ด์ฃผ๊ธฐ
    - ์ด๋ ๊ฒ ๋๋ฉด ๋ก๊ทธ์ธ ์ฑ๊ณต ์ **์ฟ ํค**์ **memberId = 1** ์ด๋ฐ ํ์์ผ๋ก ์ ์ฅ๋์ด ์๋ ๊ฒ์ ํ์ธํ  ์ ์๊ณ , ์ถ๊ฐ๋ก ํด๋ผ์ด์ธํธ๊ฐ ์๋ฒ์ ์์ฒญ์ ๋ณด๋ผ๋๋ ํญ์ ์ฟ ํค๊ฐ ํฌํจ๋์ด ์๋ ๊ฒ์ ํ์ธํ  ์ ์์
        - ์ฟ ํค ์ ๋ฌ (์๋ต ๊ฐ์ ์์ฑ๋ ์ฟ ํค๊ฐ ์ ๋ฌ๋๋ ๊ฒ ํ์ธ ๊ฐ๋ฅ)

          ![]({{site.baseurl}}/images/posts/post-220527/Untitled 2.png)
            
        - ์ถ๊ฐ๋ก ๋ค๋ฅธ ํ์ด์ง๋ฅผ ์์ฒญํ  ๋, **์์ฒญ ํค๋**์ ํด๋น **์ฟ ํค๊ฐ ๋ด๊ฒจ์๋ ๊ฒ**์ ํ์ธํ  ์ ์์
- ์ฟ ํค์ ๋ฐ๋ฅธ ํ์ ๋ก๊ทธ์ธ ์ฒ๋ฆฌ (`HomeController.homeLogin`)
    
    ```java
    @GetMapping("/")
    public String homeLogin(@CookieValue(name = "memberId", required = false) Long memberId, Model model) {
    
    		// ์ฟ ํค ๋ง๋ฃ or ๋ก๊ทธ์ธ ๋์ง ์์
        if (memberId == null) {
            return "home";
        }
    
        // ๋ก๊ทธ์ธ
        Member loginMember = memberRepository.findById(memberId);
        if (loginMember == null) {
            return "home";
        } // ์คํจ
    
        model.addAttribute("member", loginMember);
        return "loginHome";
    }
    ```
    
    - `@CookieValue` : **memberId๋ก ๋์ด ์๋ ์ฟ ํค** ๊ฐ์ ธ์ค๊ธฐ, `required = false`๋ก ๊ฐ์ด ์์ด๋ ๊ด์ฐฎ๋ค๋ ๊ฒ (๋ก๊ทธ์ธ ํ์ง ์์ ํ์๋ ์ ๊ทผํ๋๋ก)
    - ํด๋น ๊ฐ์ ํตํด Member๋ฅผ ์ฐพ์์ค๊ณ  ํด๋น Member์ ์ ๋ณด๋ฅผ ํฌํจํ Home ํ๋ฉด์ ๋ณด์ฌ์ค
- ์ฟ ํค๋ฅผ ํตํ ํ์ ๋ก๊ทธ์์ ์ฒ๋ฆฌ(`HomeController.homeLogout`)
    
    ```java
    @PostMapping("/logout")
    public String logout(HttpServletResponse response) {
        expireCookie(response, "memberId");
        return "redirect:/";
    }
    
    private void expireCookie(HttpServletResponse response, String cookieName) {
        Cookie cookie = new Cookie(cookieName, null);
        cookie.setMaxAge(0); // ์ฃฝ์ด๋ ๊ฒ
        response.addCookie(cookie); // ์ค๋ณต๋ ์ด๋ฆ์ผ๋ก ์ถ๊ฐ
    }
    ```
    
    - ์ด๋ฆ์ด ๋์ผํ **์ฟ ํค์ ๊ธฐํ์ 0์ผ๋ก ์ค์ **ํ์ฌ ์๋ก ์ถ๊ฐํด์ค์ผ๋ก์จ ํด๋น ์ฟ ํค๋ฅผ ๋ง๋ฃ ์ํค๋ ๊ฒ

<aside>
๐จ <strong>BUT!! ์ด ๋ฐฉ์์ ๋ณด์์์ ํฐ ๋ฌธ์ ์ ์ด ์์!</strong>

</aside>

### Cookie์ ๋ณด์ ๋ฌธ์ 

- **๋ณด์ ๋ฌธ์ **
    - ์ฟ ํค ๊ฐ์ **์ฌ์ฉ์๊ฐ ์์๋ก ๋ณ๊ฒฝ** ๊ฐ๋ฅ
        - ์ฟ ํค๋ **ํด๋ผ์ด์ธํธ์์ ์๋ฒ๋ก ์ ์กํ๋ ๊ฐ**, ๊ทธ๋ ๊ธฐ์ ์ธ์ ๋  ๋ณ๊ฒฝ ๊ฐ๋ฅ! (๊ฐ๋ฐ์ ๋ชจ๋/Application/Cookie ์์ ๋ณ๊ฒฝ ๊ฐ๋ฅ)
        - memberId ๋ฅผ ์์๋ก 2๋ก ์ค์ ํด๋ฒ๋ฆฌ๋ฉด ๋ก๊ทธ์ธ ํ์ง ์๊ณ ๋ memberId ๊ฐ 2์ธ Member๋ก ์ ๊ทผ์ด ๊ฐ๋ฅํด์ง๋ ๊ฒ
    - ์ฟ ํค์ ๋ณด๊ด๋ ์ ๋ณด๋ **๋๊ตฌ๋  ๋ณผ ์ ์์**
        - ๋ง์ฝ ์ฟ ํค์ ๊ฐ์ธ์ ๋ณด ๋ฑ์ด ์๋ค๋ฉด? ๊ต์ฅํ ๋ฌธ์ ๊ฐ ์ปค์ง
        - **์น ๋ธ๋ผ์ฐ์ ์๋ ๋ณด๊ด**๋๊ณ , ๋คํธ์ํฌ ์์ฒญ๋ง๋ค ์๋ฒ๋ก ์ ์ก๋จ (์ด ๋๋ง๋ค ์ ๋ณด๋ฅผ ํ์ณ ๋ณผ ์ ์๋ ๊ฒ)
        - ๊ฒฐ๊ตญ ์ฟ ํค์ ์ ๋ณด๋ก ์ธํด ๋์ ๋ก์ปฌPC๊ฐ ํธ๋ฆด ์๋ ์๊ณ , ๋คํธ์ํฌ ์ ์ก ์์ ํธ๋ฆด ์๋ ์๋ ๊ฒ
    - ์ฟ ํค๋ฅผ ํ๋ฒ ์๋ฉด ํ์ ์ฌ์ฉํ  ์ ์์
        - ์ฟ ํค์ ๊ฐ์ **์ผ์ ํ ๊ฐ์ผ๋ก ์ฌ์ฉํ๋ฉด** ํ๋ฒ ๋๊ฐ ํ์ณ๊ฐ๋ฉด ๊ทธ ์ ๋ณด๋ฅผ ํตํด **๊ณ์ํด์ ์ ๊ทผ์ด ๊ฐ๋ฅ**ํจ
- ๋์
    - ์ฟ ํค์ ์ค์ํ ๊ฐ์ ๋ธ์ถํ์ง ์๊ณ , ์ฌ์ฉ์ ๋ณ๋ก **์์ธก ๋ถ๊ฐ๋ฅํ ์์์ ํ ํฐ(๋๋ค ๊ฐ)**์ ๋ธ์ถํ๊ณ , **์๋ฒ์์ ํ ํฐ๊ณผ ์ฌ์ฉ์ id๋ฅผ ๋งคํ**ํด์ ์ธ์. ๋ํ **์๋ฒ์์ ํ ํฐ์ ๊ด๋ฆฌ**
    - **ํ ํฐ์ ์์ ๋ถ๊ฐ๋ฅ** ํด์ผ ๋จ
    - ์๋ฒ์์ ํด๋น **ํ ํฐ์ ๋ง๋ฃ์๊ฐ์ ์งง๊ฒ** ์ ์ง ํ์. ๋ํ ํดํน์ด ์์ฌ๋๋ ๊ฒฝ์ฐ ์๋ฒ์์ ํด๋น ํ ํฐ์ ๊ฐ์ ๋ก ์ ๊ฑฐ ํ ์๋ก ์์ฑํ๋ ๋ฐฉ์์ผ๋ก ์งํ
    - ํ์ง๋ง **ํด๋ผ์ด์ธํธ์ ์๋ฒ๋ ๊ฒฐ๊ตญ ์ฟ ํค๋ก ์ฐ๊ฒฐ**์ด ๋์ด์ผ ๋จ! โ **์ฟ ํค์  Session ๊ฐ๋ ๋์**
    

## ๋ก๊ทธ์ธ, ์ธ์ ์ ์ฉ

### Login with Session

- ๋ชฉํ
    - **์ฟ ํค**๋ฅผ ์ฌ์ฉํ์ ๋์ **๋ณด์ ์ด์ ํด๊ฒฐ**
    - ์ค์ํ ์ ๋ณด๋ฅผ **๋ชจ๋ ์๋ฒ์ ์ ์ฅ**
    - ํด๋ผ์ด์ธํธ์ ์๋ฒ๋ **์ถ์  ๋ถ๊ฐ๋ฅํ ์์์ ์๋ณ์ ๊ฐ**์ผ๋ก ์ฐ๊ฒฐ
    
    โ **Session ๋์**
    
- ์ธ์ (Session)
    - ๋ก๊ทธ์ธ

      ![]({{site.baseurl}}/images/posts/post-220527/Untitled 3.png)
        
        - ํด๋ผ์ด์ธํธ์์ ์ฃผ์ด์ง ๋ก๊ทธ์ธ, ๋น๋ฐ๋ฒํธ๋ฅผ ํตํด Member ์กฐํ
    - ์ธ์ ์์ฑ

      ![]({{site.baseurl}}/images/posts/post-220527/Untitled 4.png)
        
        - ํด๋น Member์ ๋ํ sessionId ์์ฑ
        - ์ธ์ ID๋ **์ถ์  ๋ถ๊ฐ๋ฅํ UUID ๊ฐ**์ผ๋ก ์ค์ 
        ex) `Cookie: mySessionId=zz0101xx-bab9-4b92-9b32-dadb280f4b61`
        - ์์ฑ๋ ์ธ์ ID์ ํด๋น Member๋ฅผ **์๋ฒ์ ์ธ์ ์ ์ฅ์์ ๋ณด๊ด**
    - ์ธ์ ์์ฑ ๋ฐ ์ ๋ฌ (**์๋ต ์ฟ ํค**๋ก. **Serve โ Client**)

      ![]({{site.baseurl}}/images/posts/post-220527/Untitled 5.png)
        
        - ์๋ฒ๋ ํด๋ผ์ด์ธํธ์ **sessionId๊ฐ๋ง์ ์ฟ ํค์ ๋ด์์ ์ ๋ฌ**
        - ํด๋ผ์ด์ธํธ๋ ์ฟ ํค ์ ์ฅ์์ ํด๋น ์ฟ ํค(์ธ์๊ฐ)๋ฅผ ๋ณด๊ด
        - ๊ฒฐ๊ตญ ํ์๊ณผ ๊ด๋ จ๋ ์ ๋ณด๋ ์ ํ ํด๋ผ์ด์ธํธ์ ์ ๋ฌ๋์ง ์๊ณ  ์ค์ง **์ถ์  ๋ถ๊ฐ๋ฅํ ์ธ์ ID๋ง์ด ์ฟ ํค๋ฅผ ํตํด ์ ๋ฌ**๋จ
    - ์ธ์Id ์ฟ ํค ์ ๋ฌ (**์์ฒญ ์ฟ ํค**๋ก. **Client โ Server**)

      ![]({{site.baseurl}}/images/posts/post-220527/Untitled 6.png)
        
        - **ํด๋ผ์ด์ธํธ**๋ ์์ฒญ ์ ํญ์ **mySessionId ์ฟ ํค๋ฅผ ์ ๋ฌ**
        - **์๋ฒ**์์๋ ํด๋น mySissionId ์ฟ ํค ์ ๋ณด๋ก **์๋ฒ ๋ด์ ์ธ์ ์ ์ฅ์๋ฅผ ์กฐํ**ํด์ ๋ก๊ทธ์ธ ์ ๋ณด๊ดํ ์ธ์ ์ ๋ณด ์ฌ์ฉ
- [์ฟ ํค โ **์ธ์**] **๊ฐ์ ์ **
    - ์ฟ ํค ๊ฐ ๋ณ์กฐ ํ ์ ๊ทผ โ UUID๋ฅผ ํตํด **์์ธก ๋ถ๊ฐ๋ฅํ ์ฟ ํค ๊ฐ**
    - ์ฟ ํค๊ฐ์ ํตํ ์ ๋ณด ํดํน โ ์ธ์ Id, ์ฆ ๋๋ฌ๋ ์๋ ์ ๋ณด์๋ **๊ฐ์ธ์ ๋ณด ๋ฐ ์ค์ ์ ๋ณด๊ฐ ์์**
    - ์ฟ ํค ํ์ทจ ํ ์ฌ์ฉ โ ์๋ฒ์์ ์ธ์์ **๋ง๋ฃ์๊ฐ์ ์งง๊ฒ ์ ์ง ๋ฐ ์ธ์ ๊ฐ์  ์ ๊ฑฐ**

- ์ธ์ ์ง์  ๊ฐ๋ฐ
    - Spring์์ ์ ๊ณตํ๋ Session์ธ `HttpSession` **์ ์ฌ์ฉํ์ง ์๊ณ ** ์๋ฆฌ๋ฅผ ์ดํดํ๊ธฐ ์ํด ์ง์  session ๊ฐ๋ฐ
    - ์ธ์ ์์ฑ, ์กฐํ, ๋ง๋ฃ(์ ๊ฑฐ) โ `SessionManager`
        - ์ธ์ ์ ์ฅ์
            - `private Map<String, Object> sessionStore = new ConcurrentHashMap<>();`
            - ๋์์ฑ ๋ฌธ์ ๋ก `ConcurrentHashMap` ์ฌ์ฉ
        - ์ธ์ ์์ฑ
            
            ```java
            public void createSession(Object value, HttpServletResponse response) {
            	 //์ธ์ id๋ฅผ ์์ฑํ๊ณ , ๊ฐ์ ์ธ์์ ์ ์ฅ
            	 String sessionId = UUID.randomUUID().toString();
            	 sessionStore.put(sessionId, value);
            	 //์ฟ ํค ์์ฑ
            	 Cookie mySessionCookie = new Cookie("mySessionId", sessionId);
            	 response.addCookie(mySessionCookie);
            }
            ```
            
            - UUID ๋ฅผ ํตํ ์ถ์  ๋ถ๊ฐ๋ฅํ ์์์ `sessionId` ์์ฑ
            - **์ธ์ ์ ์ฅ์(**`sessionStore`**)**์ `sessionId`์ ๋ณด๊ดํ  ๊ฐ(`value`) ์ ์ฅ
            - `sessionId`๋ก ์๋ต ์ฟ ํค(`mySessionCookie`)๋ฅผ ์์ฑํด์ ํด๋ผ์ด์ธํธ์ ์ ๋ฌ(`response.addCookie(mySessionCookie)`)
        - ์ธ์ ์กฐํ
            
            ```java
            public Object getSession(HttpServletRequest request) {
            	 Cookie sessionCookie = findCookie(request, SESSION_COOKIE_NAME);
            	 if (sessionCookie == null) {
            		 return null;
            	 }
            	 return sessionStore.get(sessionCookie.getValue());
            }
            ```
            
            - findCookie๋ฅผ ํตํด ํด๋น request์์ ํด๋น ์ด๋ฆ์ ๊ฐ์ง cookie๋ฅผ ๊ฐ์ ธ์ด
            - ํด๋น ์ฟ ํค๊ฐ ๋น์์ผ๋ฉด null ๋ฐํ, ์ฟ ํค๊ฐ ์กด์ฌํ๋ฉด ํด๋น ์ฟ ํค์ mySessionId๋ฅผ ํตํด **์ ์ฅ์์์ ๋งคํ๋ Object(Member) ๋ฐํ**
        - ์ธ์ ๋ง๋ฃ
            
            ```java
            public void expire(HttpServletRequest request) {
            	 Cookie sessionCookie = findCookie(request, SESSION_COOKIE_NAME);
            	 if (sessionCookie != null) {
            		 sessionStore.remove(sessionCookie.getValue());
            	 }
            }
            ```
            
            - findCookie๋ฅผ ํตํด ํด๋น request์์ ํด๋น ์ด๋ฆ์ ๊ฐ์ง cookie๋ฅผ ๊ฐ์ ธ์ด
            - ํด๋น ์ฟ ํค์ mySessionId๋ฅผ ํตํด **์ ์ฅ์์์ ํด๋น ์ ๋ณด๋ฅผ ์ ๊ฑฐ** โ ์๋ฒ ๋ด๋ถ ์ ์ฅ์์์๋ง ์ญ์ ํด์ฃผ๋ฉด ์ฟ ํค์ mySessionId๊ฐ ์๋๋ผ๋ **๋งคํ๋ ์ ๋ณด๊ฐ ์๊ธฐ์ ํด๋น Session์ ์ด์ฉํ  ์ ์์**
    - ์ง์  ๋ง๋  ์ธ์ ์ ์ฉ
        - `private final SessionManager sessionManager;` ์ฃผ์ (**์ง์  ๊ฐ๋ฐํ Session ๊ด๋ฆฌ์**)
        - Login [`LoginController` ์ login ์ฒ๋ฆฌ ๋ถ๋ถ (**์ฟ ํค โ ์ธ์** ์ผ๋ก ๋ณ๊ฒฝ)]
            - ์ธ์ ๊ด๋ฆฌ์๋ฅผ ํตํด **์ธ์ ์์ฑ ๋ฐ ํ์ ๋ฐ์ดํฐ ๋ณด๊ด** (์ธ์ ์ ์ฅ์์), ์ธ์ ID๋ฅผ ๋ฐํ์ผ๋ก **์ฟ ํค ์์ฑ ๋ฐ ์ ๋ฌ**
            - `Cookie idCookie = new Cookie("memberId", String.valueOf(loginMember.getId())); response.addCookie(idCookie);` **โ** **`sessionManager.createSession(loginMember, response)`**
            - ์ง์  ๋ง๋  sessionId๋ก ์ฟ ํค๊ฐ ์ ์ ๋ฌ๋๋ ๊ฒ์ ํ์ธํ  ์ ์์

              ![]({{site.baseurl}}/images/posts/post-220527/Untitled 7.png)
                
            - ๋ํ, ์์ฒญ ์์๋ ํด๋น ์ฟ ํค๋ก ์ ์์ฒญ์ด ๋ค์ด์ค๋ ๊ฒ ํ์ธ ๊ฐ๋ฅ
        - LogOut [`LoginController` ์ logout ์ฒ๋ฆฌ ๋ถ๋ถ (**์ฟ ํค โ ์ธ์** ์ผ๋ก ๋ณ๊ฒฝ)]
            - ๋ก๊ทธ ์์์ **ํด๋น ์ธ์์ ์ ๋ณด๋ฅผ ์ ๊ฑฐ**
            - `expireCookie(response, "memberId");` โ **`sessionManager.expire(request);`**
            - ํน์ง : ์ฟ ํค๋ฅผ ์ง์  ์ญ์ ํ๋ ๊ฒ์ด ์๋ **์ธ์์ ์ฅ์์ ํด๋น ์ธ์ ์ ๋ณด๋ง์ ์ญ์ **ํ๋ ๊ฒ (์ธ์ ์ ์ฅ์์์ ์ญ์ ํ๋ฉด ๋ ์ด์ ํด๋น ์ธ์ID๋ ์ฌ์ฉ ๋ถ๊ฐ!)
        - Home [`HomeController`์ `@GetMapping(โ/โ)`]
            
            ```java
            @GetMapping("/") // Session ์ด์ฉ
            public String homeLoginV2(HttpServletRequest request, Model model) {
            
                // ์ธ์ ๊ด๋ฆฌ์์ ์ ์ฅ๋ ํ์ ์ ๋ณด ์กฐํ
                Member member = (Member) sessionManager.getSession(request);
            
                // ๋ก๊ทธ์ธ
                if (member == null) {
                    return "home";
                } // ์คํจ
            
                model.addAttribute("member", member);
                return "loginHome"; // ์ฑ๊ณต
            }
            ```
            
            - `@CookieValue` โ **`HttpServletRequest`, `(Member) sessionManager.getSession(request);`** : request๋ฅผ ํตํด cookie์ **sessionId์ ๋งคํ๋ Object(Member)** ๋ฅผ ๊ฐ์ ธ์ด (์๋ฒ ๋ด์ **์ธ์ ์ ์ฅ์์์**)
    
    <aside>
    โ ๏ธ <strong>์ธ์ ์ง์  ๊ฐ๋ฐ์์ ์์์ผ ํ  ์ </strong>
    <ul>
    <li>์ฌ์ค ์ธ์์ด๋ผ๋ ๊ฒ์ด ๋ญ๊ฐ ํน๋ณํ ๊ฒ์ด ์๋๋ผ ๋จ์ง <b>์ฟ ํค๋ฅผ ์ฌ์ฉ</b>ํ๋๋ฐ, <b>์๋ฒ์์ ๋ฐ์ดํฐ๋ฅผ ์ ์งํ๋ ๋ฐฉ๋ฒ</b>์ผ ๋ฟ </li>
    <li>ํ์ง๋ง ์ด๋ ๊ฒ ์ง์  ๊ฐ๋ฐํ๋ฉด ๊ต์ฅํ ๋ณต์กํด์ง ๊ฒ์ ์ ์ ์์ </li>
    <li>๊ทธ๋์ <b>์๋ธ๋ฆฟ์์๋ ๊ณต์์ ์ผ๋ก ์ธ์์ ์ง์</b>ํจ โ <b><code>HttpSession</code></b> </li>
    <li><code>HttpSession</code>์ ์ฐ๋ฆฌ๊ฐ ์ง์  ๋ง๋  ์ธ์๊ณผ ๋์ ๋ฐฉ์์ด ๊ฑฐ์ ๋์ผ!! </li>
    </ul>
    </aside>
    

### Login with โServlet HTTP Session(HttpSession)โ

- ์ง์  ๊ฐ๋ฐํ๋ Session์ ๊ฐ๋์ ์ด๋ฏธ **์๋ธ๋ฆฟ์ด `HttpSession` ์ด๋ผ๋ ๊ธฐ๋ฅ์ผ๋ก ์ ๊ณต**
- ์ง์  ๊ตฌํํ๋ ๋ก์ง๊ณผ ๊ฑฐ์ ๋์ผํ๋ฉฐ ๋ ํธ๋ฆฌํ๊ฒ ์ฌ์ฉํ  ์ ์๋๋ก ๊ตฌํ๋์ด ์์
- `HttpSession`
    - ์ง์  ๋ง๋ค์ด ๋ณด์๋ SessionManger์ ๊ฐ์ ๋ฐฉ์์ผ๋ก ๋์
    - JSESSIONID ๋ผ๋ ์ด๋ฆ์ ์ฟ ํค ์์ฑ, ๊ฐ์ ์ถ์  ๋ถ๊ฐ๋ฅ โ `Cookie: JSESSIONID=5B78E23B513F50164D6FDD8C97B0AD05`
- HttpSession ์ฌ์ฉ **[Version 1]**
    - Login [`LoginController` ์ login ์ฒ๋ฆฌ ๋ถ๋ถ (์ง์  ๊ฐ๋ฐํ ์ธ์ โ HttpSession ์ด์ฉ)]
        - `sessionManager.createSession(loginMember, response)` โ **`HttpSession session = request.getSession();` , `session.setAttribute(SessionConst.LOGIN_MEMBER, loginMember)`**
        - `request.getSession(true)` : ์ธ์์ด ์์ผ๋ฉด ๊ธฐ์กด ์ธ์ ๋ฐํ, ์์ผ๋ฉด ์๋ก์ด ์ธ์์ ์์ฑํด์ ๋ฐํ (false๋ฉด ์๋ก์ด ์ธ์ ๋ฐํ X, null ๋ฐํ)
        - `session.setAttribute(...)` : ์ธ์์ **๋ฐ์ดํฐ ๋ณด๊ด**. ํ๋์ ์ธ์์ key, value ๋ก ์ฌ๋ฌ ๊ฐ ์ ์ฅ ๊ฐ๋ฅ (์ธ์ ์ ์ฅ์ ์ด์ฉ)
        - ์ฆ, ๊ธฐ์กด ์ธ์์ด ์๋ค๋ฉด ๊ทธ ์ธ์์ ๊ฐ์ ธ์์ ๋ด๊ฐ ์ ์ฅํ๊ณ  ์ถ์ **๋ฐ์ดํฐ๋ฅผ ์ ์ฅ** (**๋ฉ๋ชจ๋ฆฌ ์ฌ์ฉ**). ๊ธฐ์กด์ ์ ์ผํ ์์ด๋๋ฅผ ๊ฐ์ง๊ณ  ์๋ **JSESSIONID์ ํด๋น ๋ฐ์ดํฐ๋ค์ด ๋งคํ**๋๋ ๊ฒ (์ด์ ์๋ ์ง์  sessionId, ์ ์ฅ์๋ฅผ ์์ฑํ๊ณ  ์์ด๋์ ๊ฐ์ฒด๋ฅผ ๋งคํํจ)
        - `HttpSession`์ ์ด์ฉํ์ฌ JSESSIONID๋ก ์ฟ ํค๊ฐ ์ ์ ๋ฌ๋ ๊ฒ์ ํ์ธํ  ์ ์์

          ![]({{site.baseurl}}/images/posts/post-220527/Untitled 8.png)
            
    - LogOut [`LoginController` ์ logout ์ฒ๋ฆฌ ๋ถ๋ถ (์ง์  ๊ฐ๋ฐํ ์ธ์ โ HttpSession ์ด์ฉ)]
        - `sessionManager.expire(request);` โ **`HttpSession session = request.getSession(false);` , `if (session != null) { session.invalidate(); }`**
        - `HttpSession session = request.getSession(false);` : ์ธ์์ ์์ฑํ์ง ์๊ณ  ์๋ค๋ฉด ์ธ์ ๋ฐํ
        - `session.invalidate()` : ์ธ์ ์ ๊ฑฐ
            - ์ด์ ๊ณผ ๋์ผํ๊ฒ **ํด๋น ์ธ์์ ๋งคํ๋ ๋ชจ๋  ๋ฐ์ดํฐ๋ฅผ ์ญ์ **ํ๋ ๊ฒ. (์ฟ ํค๋ฅผ ์ญ์ ํ์ง ์์. ๊ตณ์ด ์ญ์ ํ  ํ์๊ฐ ์๊ธฐ ๋๋ฌธ)
    - Home [`HomeController`์ `@GetMapping(โ/โ)`]
        
        ```java
        HttpSession session = request.getSession(false);
        
        if (session == null) {
        	return "home";
        }
        
        Member loginMember = (Member) session.getAttribute(SessionConst.LOGIN_MEMBER);
        ```
        
        - `session.getAttribute(SessionConst.LOGIN_MEMBER);` : ๊ฐ์ ธ์จ ์ธ์์์ key์ value๋ก ์ ์ฅํ์๋ ๊ทธ key๋ก Member ๊ฐ์ฒด๋ฅผ ๊ฐ์ ธ์ด
- HttpSession **[Version2]**
    - `@SessionAttribute`
        - ์คํ๋ง์ด ์ธ์์ ๋ ํธ๋ฆฌํ๊ฒ ์ฌ์ฉํ  ์ ์๋๋ก ์ง์ํ๋ ๊ธฐ๋ฅ
        - ๋ฌผ๋ก  ์ฟ ํค๋ฅผ ์์ฑํ๊ณ  ํ๋ ๋ถ๋ถ์ ์ง์  request์์ session์ ๊ฐ์ ธ์ setAttribute๋ก ์งํํด์ผ๋จ.
        - ํด๋น ๊ธฐ๋ฅ์ **session์ ์ ์ฅ๋ ๋ฐ์ดํฐ๋ฅผ ๊ฐ์ ธ์ฌ ๋ ํธ๋ฆฌ**ํ๊ฒ ์ฌ์ฉ ๊ฐ๋ฅ!!
    - Home [`HomeController`์ `@GetMapping(โ/โ)`]
        - ์ด์  ๋ฒ์ ์์ request์์ **session์ ๊ฐ์ ธ์ค๊ณ  session์ ์ ์ฅ๋ ๋ฐ์ดํฐ๋ฅผ ๊ฐ์ ธ์ค**๋ ๋ถ๋ถ์ด **๋จ ํ์ค๋ก ๋๋ผ ์ ์์**
        - `@SessionAttribute(name = SessionConst.LOGIN_MEMBER, required = false) Member loginMember`
        - ์ธ์์ด ์๋ค๋ฉด ๊ทธ ์ธ์๊ณผ ๋งคํ๋ ๋ฐ์ดํฐ๋ค ์ค`SessionConst.LOGIN_MEMBER` ์ ํด๋น ํ๋ Member ๊ฐ์ฒด๋ฅผ ๊ฐ์ ธ์ค๋ ์ญํ 
        - `required = false` ๋ฅผ ํตํด ๋ก๊ทธ์ธ๋์ง ์์ ํ์์ ๋ํด์๋ ๋ฐ์์ค
        - ์ธ์์ด ์๊ฑฐ๋, ํด๋น ๋ฐ์ดํฐ์ ๋ง๋ Member๊ฐ ์์ ๊ฒฝ์ฐ null

### Session Info and Timeout

- HttpSession์์ ์ ๊ณตํ๋ ์ ๋ณด
    - `session.getAttribute()` : ์ธ์์ ๋ณด๊ด๋์ด ์๋ ๋ฐ์ดํฐ
    - `session.getId()` : ์ธ์Id, JSESSIONID์ ๊ฐ
    - **`session.getMaxInactiveInterval()`** :  **[์ค์]** ์ธ์์ **์ ํจ ์๊ฐ**. ๊ธฐ๋ณธ๊ฐ : 1800์ด
    - `session.getCreationTime()` : ์ธ์ ์์ฑ์ผ์
    - **`session.getLastAccessedTime()`** : **[์ค์]** ์ธ์๊ณผ ์ฐ๊ฒฐ๋ **์ฌ์ฉ์๊ฐ** **์ต๊ทผ์ ์๋ฒ์ ์ ๊ทผํ ์๊ฐ**. ํด๋ผ์ด์ธํธ์์ ์๋ฒ๋ก sessionId ( JSESSIONID )๋ฅผ ์์ฒญํ ๊ฒฝ์ฐ์ ๊ฐฑ์ ๋จ
    - `session.isNew()` :  ์๋ก ์์ฑ๋ ์ธ์์ธ์ง, ์๋๋ฉด ์ด๋ฏธ ๊ณผ๊ฑฐ์ ๋ง๋ค์ด์ก๊ณ , ํด๋ผ์ด์ธํธ์์ ์๋ฒ๋ก sessionId ( JSESSIONID )๋ฅผ ์์ฒญํด์ ์กฐํ๋ ์ธ์์ธ์ง ์ฌ๋ถ
- **์ธ์ ํ์์์ ์ค์ **
    - ์ธ์ ์ข๋ฃ ์์ ์ ์ฌ์ฉ์๊ฐ ์๋ฒ์ ์ต๊ทผ์ ์์ฒญํ ์๊ฐ์ ๊ธฐ์ค์ผ๋ก 30๋ถ ์ ๋๋ฅผ ์ ์งํด์ฃผ๊ณ  ๊ทธ ์ดํ๋ก ๋๋ ๊ฒ์ด ์ข์ ๋์. (์ฌ์ฉ์๊ฐ ์ฌ์ฉํ๊ณ  ์์ ๋ ์ธ์์ด ๋ง๋ฃ๋์ง ์๋๋ก ํ๊ธฐ ์ํจ)
    - HttpSession ์ ์ด ๋ฐฉ์์ ์ฌ์ฉ
    - ๊ทธ๋ ๋ค๋ฉด 30๋ถ ๋ง๊ณ  **์ง์  ๋ง๋ฃ ์๊ฐ์ ์ค์ **ํ๊ณ  ์ถ๋ค๋ฉด? โ **๊ธ๋ก๋ฒ ์ค์ ** ์ด์ฉ
        - ์คํ๋ง์ `application.properties` ์์ `server.servlet.session.timeout` ์ ๊ฐ์ ์ค์ ํด์ฃผ๋ฉด ๋จ (๊ธฐ๋ณธ ๊ฐ์ด 1800_30๋ถ)
- ์ธ์ ํ์์์ ๋ฐ์
    - ์ธ์์ ํ์์์ ์๊ฐ์ ํด๋น ์ธ์๊ณผ ๊ด๋ จ๋ JSESSIONID ๋ฅผ ์ ๋ฌํ๋ HTTP ์์ฒญ์ด ์์ผ๋ฉด ํ์ฌ ์๊ฐ์ผ๋ก ๋ค์ ์ด๊ธฐํ ๋จ
    - ์ฆ, **์์ฒญ์ด ๋ค์ด์๋ค ํ๋ฉด ๋ค์ ์ฒ์๋ถํฐ 30๋ถ์ ํ์ด๋จธ๋ฅผ ๋๋ฆฌ๋ ๊ฒ**
    - ๊ฒฐ๊ตญ ์ด์ ๊ด๋ จ๋ session info ๋ **`session.getLastAccessedTime()` (์ต๊ทผ ์ธ์ ์ ๊ทผ ์๊ฐ) , `session.getMaxInactiveInterval()` (์ธ์์ ์ ํจ ์๊ฐ)**
    - ์ฆ, `LastAccessedTime` ์ดํ๋ก timeout ์๊ฐ์ด ์ง๋๋ฉด, WAS๊ฐ ๋ด๋ถ์์ ํด๋น ์ธ์์ ์ ๊ฑฐํ๊ณ  timeout ์ด์ ์ ๋ค์ ์์ฒญ์ด ๋ค์ด์ค๋ฉด `LastAccessedTime`์ ๊ฐฑ์ ํ๊ณ  ๊ทธ ์์ ์ผ๋ก ๋ค์ timeout ์ฒดํฌ

<aside>
โ ๏ธ <strong>์ค๋ฌด์์ Session ์ฌ์ฉ ์ฃผ์์ !</strong> <br>
<b>์ธ์</b>์๋ <b>์ต์ํ์ ๋ฐ์ดํฐ๋ง ๋ณด๊ด</b>ํด์ผ ํ๋ค! <br>
๋ณด๊ดํ ๋ฐ์ดํฐ ์ฉ๋ * ์ฌ์ฉ์ ์๋ก ์ธ์์ ๋ฉ๋ชจ๋ฆฌ ์ฌ์ฉ๋์ด ๊ธ๊ฒฉํ๊ฒ ๋์ด๋์ ์ฅ์ ๋ก ์ด์ด์ง ์ ์๊ธฐ ๋๋ฌธ! ๋ํ ์ธ์์ ์๊ฐ์ ๋๋ฌด ๊ธธ๊ฒ ์ก์ผ๋ฉด ๋ฉ๋ชจ๋ฆฌ ์ฌ์ฉ์ด ๊ณ์ ๋์ ! ์ ๋นํ ์๊ฐ ์ค์  ํ์ํจ!

</aside>