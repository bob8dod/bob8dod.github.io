---
layout: page
title: TID
permalink: /tid/
image:
---

<aside>
๐๏ธ <strong>Today I Developed</strong> <br>
๋งค์ผ ๋งค์ผ ์ ๊ฐ ๊ฐ๋ฐํ ํ์ ์ ๋จ๊ธฐ๋ ๊ณต๊ฐ์๋๋ค!
</aside>

<br>

## Project(SEMOGONG, Capstone, ...) ๊ฐ๋ฐ ์ผ์ง

- Issue ๋ฅผ ๋ฐํ์ผ๋ก ๊ฐ๋ฐ ์งํ.
- Issue Labels
    - `Bug` : ๊ตฌํํ ๊ธฐ๋ฅ๋ค ์ค์์ ์๋ฒ์ ์ฌ๋ ธ๋๋ฐ ์๋ํ์ง ์๊ฑฐ๋ ์ค๋ฅ๊ฐ ์๊น
    - `Back` : Back-End Engineer ๋ง์ ์ํ ์ด์
    - `Front` : Front-End Engineer ๋ง์ ์ํ ์ด์
    - `Enhancement` : ์๋ก์ด ๊ธฐ๋ฅ์ด ํ์ํ ๊ฒฝ์ฐ, front back ๋ชจ๋๊ฐ ์๋ด์ผํ  ์ด์

<hr>

### <220719>

> #65 `Bug` : static ๊ทธ๋ํ ์ค๋ฅ
>

- #65
    - ๋ฐ์๋ Bug
        - 0~4์์ ์์ฒญ์ ๋ํด์  ์ดํ์ ๊น์ง์ ๋ฐ์ดํฐ๋ง ๋ณด์ฌ์ ธ์ผ๋๋๋ฐ, 0์20๋ถ ์ฏค์ ์์ฒญํ์ ๋, ํ๋ฃจ์ ์ ๋ฐ์ดํฐ๊ฐ ๋ณด์ฌ์ง
        - ๋์  ๋ฐ์ดํฐ์ธ๋ฐ, ์ค๊ฐ์ ๋์ ๋์ง ์๋ ํ์ ๋ฐ์
    - ํด๊ฒฐ
        - 0~4์์ ์์ฒญ์ ๋ํ ๋ถ๊ธฐ๋ฅผ if (0<nowTimeโฆ) ์ด๋ฐ ์์ผ๋ก ๋ฒ์ ์ค์ ์ ์๋ชปํจ. โ `if (now.getHour() < 4)` ๋ก ๋ฒ์ ์์ 
        - ๋์  ๋ฐ์ดํฐ ๋ฌธ์ 
            - ๋ ์ง์ ๋ํ HashMap์ key๋ฅผ ๊ฐ์ ธ์ฌ๋, ๋ฐ๋ก ์ ๋ ฌํ์ง ์์ ๋ฐ์ โ keySet์ ํตํด ๊ฐ์ ธ์ค๋ฉด ๋น์ฐํ ์ ๋ ฌ์์ด ๊ฐ์ ธ์์ง.
            - ์ด๋ keySet์ ๊ฐ์ ธ์ค๊ณ , ๋ ์ง๋ฅผ ์ค๋ฆ์ฐจ์์ผ๋ก ์ ๋ ฌํ์ฌ ํด๊ฒฐ

                ```java
                List<Integer> list = new ArrayList<>(dayTimes.keySet());
                list.sort(Comparator.naturalOrder());
                ```

                - keySet์ list๋ก ๋ณ๊ฒฝํ๊ณ  sort๋ฅผ ํตํด ์ ๋ ฌ
            - ์ด๋ ๊ฒ ๋๋ฉด ๋ ์ง๊ฐ ๋ค์ฃฝ๋ฐ์ฃฝ๋์ง ์๊ณ  ์ค๋ฆ์ฐจ์์ผ๋ก ์งํ๋๊ธฐ ๋๋ฌธ์, ์ฝ๋ ์ ๋์ ์ด ์ ๋๋ก ์งํ๋จ

### <220718>

> #63 `Bug` : ์ฒซ ๋ก๊ทธ์ธ ์ ๋ฐ์ํ๋ ์ค๋ฅ (session Id ์ด๋์ผ๋ก ์ธํ ๋ฌธ์ ) <br>
> #64 `Enhancement` : static ์๋น์ค ๊ธฐ๋ฅ ์ถ๊ฐ (์ผ์ฃผ์ผ ๊ฐ์ ๊ณต๋ถ ํ์, ๊ณต๋ถ์, ์ถ์์ ๊ธฐ๋ฅ ์ถ๊ฐ)
>

- #63
    - ๊ธฐ์กด์ ๋ฌธ์ ์ 
        - ์ฒซ ๋ก๊ทธ์ธ ์ (์ธ์ ์ ์ฅ ์) ์ค๋ฅ ๋ฐ์
        - ์ธ์ ์ ์ฅ ํ ํด๋น ๊ฐ์ url๋ก ๋ณด๋ด์ง๋ฉด์ ๋ฐ์ํ๋ ์ค๋ฅ๋ก ํ๋จ (์ฟ ํค๊ฐ ์ ์ฉ๋์ง ์๋ ์ํฉ์ ๋๋นํ servlet ์๋ ๊ธฐ๋ฅ โ traking-modes)
    - ํด๊ฒฐ
        - servlet ์ค์ ์ ํตํด ํด๋น ์๋ ๊ธฐ๋ฅ์ ๋ง์ (cookie๋ก ์ค์ ํจ์ผ๋ก์จ ์ฟ ํค๋ก๋ง session์ ์ธ์ํ๊ฒ ๋ค๋ ์ค์ )

        ```yaml
        server.servlet.session.tracking-modes: 'cookie'
        ```

- #64
    - ์ผ์ฃผ์ผ ๊ฐ์ ๊ณต๋ถ ํ์
        - ์ผ์๋ก count ํ๋ฉฐ ๊ณต๋ถ์๊ฐ์ด 0์ธ post๋ค์ ๋ํด์ counting
        - ๊ทธ ํ 7-cnt ๋ก ๊ณต๋ถ ํ์ ๊ธฐ๋ก
        - ๋ง์ง๋ง์ผ๋ก ํด๋น ๊ฐ์ memberDto์ ์ ์ฅ, veiw Template์ผ๋ก ๋๊ฒจ์ค
    - ๊ณต๋ถ์, ์ถ์์ ๊ธฐ๋ฅ
        - ๊ณต๋ถ์ : ์ผ์ฃผ์ผ๊ฐ์ ๋์  ๊ณต๋ถ์๊ฐ์ด ๊ฐ์ฅ ๋ง์ ์ฌ๋
        - ์ถ์์ : ์ผ์ฃผ์ผ๊ฐ์ ๊ณต๋ถ ํ์๊ฐ ๊ฐ์ฅ ๋ง์ ์ฌ๋

### <220717>

> #62 `Back` : ๋ฐฐ์ด๋ด์ฉ์ ์ฉ โ ์ต์ ํ
> <ol>
> <li> Spring Security ์์ด ๋ก๊ทธ์ธ ๊ฐ๋ฐ (authentication โ session) </li>
> <li> session timeout ์ฐ์ฅ </li>
> </ol>
> #61 `Front` : ๋ก๊ทธ์ธ ํ์ด์ง ์์ฑ
>

- #62-1
    - ๊ธฐ์กด์ ๋ฌธ์ ์ 
        - ์ ๋๋ก ๋ฐฐ์ฐ์ง ์์ Spring Security๋ฅผ ์ฌ์ฉํจ์ผ๋ก์จ ์ํ๋ ๊ธฐ๋ฅ์ ์ ๋ ์ ๋๋ก ์ด์ฉํ์ง ๋ชปํจ (session ์ค์  ๋ฑ) โ ์ถํ ํ์ต ํ ์ ์ฉ ์์ 
    - ํด๊ฒฐ
        - **session**์ ์ด์ฉํ login(์ธ์ฆ)์ผ๋ก ๊ตฌํ โ ๋ก๊ทธ์ธ ๊ฒ์ฆ ๋ฐ ๋ก๊ทธ์ธ์ ๋ชจ๋ ์ง์  ๊ตฌํ
        - Spring Security์ ๊ด๋ จ๋ ๋ชจ๋  ์ฝ๋๋ฅผ ์ ๊ฑฐํ๊ณ  session๊ณผ ๊ด๋ จ๋ ์ฝ๋๋ก ๋์ฒด _(๋น๋ฐ๋ฒํธ ์ธ์ฝ๋ฉ ์ ์ธ)_
        - ๋ก๊ทธ์ธ

            ```java
            @PostMapping("/members/login")
                public String login(@Validated @ModelAttribute("loginForm") LoginForm loginForm, BindingResult bindingResult,
                                    @RequestParam(defaultValue = "/") String redirectURL,
                                    HttpServletRequest request) {...}
            ```

            - ๋ก๊ทธ์ธ ์ ์ค๊ฐ์ ๋ก๊ทธ์ธ ์๊ตฌ์ ์ํด์ ๋ก๊ทธ์ธ์ ํ๋ ์ํฉ์ ๋๋นํด์ redirectURL์ ๋ฐ์์ด (์์ ๊ฒฝ์ฐ ๊ทธ๋ฅ home์ผ๋ก)
            - session์ ๋ฐ์์ค๊ธฐ ์ํด HttpServletRequest ์ด์ฉ
            - ๋ก๊ทธ์ธ ๊ฒ์ฆ

                ```java
                // form ํ์์ ๋ง์ง ์์ ์ ์ถ
                if (bindingResult.hasErrors()) {
                    return "login";
                }
                
                // ๋ก๊ทธ์ธ ๊ฒ์ฆ         // Global Error -> Object ๋จ ์ค๋ฅ! (์ง์  ์ฒ๋ฆฌํด์ผ ๋๋ ๋ถ๋ถ)
                Optional<Member> loginMember = memberService.findByLoginId(loginForm.getLoginId());
                BCryptPasswordEncoder passwordEncoder = new BCryptPasswordEncoder();
                if (loginMember.isEmpty() || !passwordEncoder.matches(loginForm.getPassword(), loginMember.get().getPassword())) {
                    bindingResult.reject("loginFail", "์์ด๋ ๋๋ ๋น๋ฐ๋ฒํธ๊ฐ ๋ง์ง ์์ต๋๋ค.");
                    return "login";
                }
                ```

                - Form ํ์์ ๋ง์ง ์์ ์ ์ถ์์์ ๊ฒ์ฆ์ Bean Validation ์ฌ์ฉ (`FieldError`)
                - ์์ด๋ ๋ฐ ๋น๋ฐ๋ฒํธ ์ค๋ฅ ์ `ObjectError` ๋ฐ์ ํ veiw Template์ผ๋ก ๋๊ฒจ ์ค
                - `Optional`์ ํตํด ์ค๋ฅ ๊ฒ์ฆ
            - ๋ก๊ทธ์ธ ์ฑ๊ณต

                ```java
                // ๋ก๊ทธ์ธ ์ฑ๊ณต
                HttpSession session = request.getSession();
                // ์ธ์์ ๋ก๊ทธ์ธ ํ์ ์ ๋ณด ๋ณด๊ด
                session.setAttribute("loginMember", loginMember.get().getId());
                return "redirect:"+ redirectURL;
                ```

                - ๋ก๊ทธ์ธ ์ฑ๊ณต ์ session์ ํ์์ ๋ณด(member id) ์ ์ฅ
                - **member ๊ฐ์ฒด ์์ฒด๋ฅผ ์ ์ฅํ์ง ์๊ณ , member id๋ง์ ์ ์ฅํ์ฌ, ์ถํ ์ด๋ฅผ ํตํด repo์์ member๊ฐ์ฒด๋ฅผ ๊ฐ์ ธ์ค๋ ๋ฐฉ์์ผ๋ก ์งํ**
- #62-2
    - ๊ธฐ์กด์ ๋ฌธ์ ์ 
        - session timeout์ ์ค์ ํ์ง ์์, ๋๋ฌด ์์ฃผ ๋ก๊ทธ์์๋์ด ์ฌ์ฉ ์ ๋ถํธํจ
    - ํด๊ฒฐ
        - spring securrity๋ฅผ ์ฌ์ฉํ์ง ์๊ณ  ์ง์  session์ ์ด์ฉํ๊ณ , session timeout์ applications.yml ์ ์ค์ ํ์ฌ ๊ด๋ฆฌ (timeout์ 10์๊ฐ์ผ๋ก ์ค์ )

            ```yaml
            server.servlet.session.timeout: 36000
            ```

- #61
    - ๊ธฐ์กด์ ๋ฌธ์ ์ 
        - ๋ก๊ทธ์ธ ํ์ด์ง ์์ด home์์ ๋ก๊ทธ์ธ์ ์งํํ๋ค ๋ณด๋ ์ธ๋ฐ์์ด ๋ฐ์ดํฐ๊ฐ ์ฃผ๊ณ ๋ฐ์์ง๋ ๊ฒฝ์ฐ๊ฐ ์์. (home์์ ๋ก๊ทธ์ธ ์ด์ ์๋ ๋ค๋์ ๋ฐ์ดํฐ๋ฅผ ๋ณด์ฌ์ฃผ๊ณ , ๋ก๊ทธ์ธ ํ์๋ ๋ค๋์ ๋ฐ์ดํฐ๋ฅผ ๋ณด์ฌ์ฃผ๊ณ  ์์. โ ๋น ํจ์จ์ )
        - redirect๋ฅผ ํตํ ๋ก๊ทธ์ธ ํ ํ์ด์ง ์ด๋์ด ๋ถ๊ฐ๋ฅ.
    - ํด๊ฒฐ
        - ์ผ๋ฐ์ ์ธ ์น์ฌ์ดํธ์ฒ๋ผ ๋ก๊ทธ์ธ ํ์ด์ง๋ฅผ ๋ฐ๋ก ๋ง๋ค์ด ์ ๊ณต
        - ๋ก๊ทธ์ธ ํ์ด์ง๋ฅผ ์๋ก ์์ฑ


<aside>
โ ๏ธ <strong>[์ค๋์ ๋ฐ๊ฒฌ] session์ ์ ๋ณด ์ ์ฅ ์ ์ฃผ์์ !</strong> <br>
session์ member ๊ฐ์ฒด๋ฅผ ๊ทธ๋๋ก ์ ์ฅํ๋ฉด ๊ทธ ์ ์ฅ๋ member๋ฅผ ์ฌ์ฉํ๊ธฐ ๋๋ฌธ์ ๋ณ๊ฒฝ๋ ๊ฐ์ด ์ ์ฉ๋์ง ์์ ( โ ์ ์ฅ ์์ ์ member๋ฅผ ์ฌ์ฉํ๋ ๊ฒ) <br>
๊ทธ๋ ๊ธฐ์ <b>session์๋ id๋ง์ ์ ์ฅํ๊ณ  ์ฌ์ฉ ์ ๊ทธ id๋ฅผ ํตํด repo์ ์ ์ฅ๋์ด ์๋ member(๋ณ๊ฒฝ ์ฌํญ ๋ฐ์๋ member)๋ฅผ ๊ฐ์ ธ์์ผ</b>๋จ

</aside>

### <220714>

> #48 `Enhancement` : ์๋ก์ด ๋์๋ณด๋ (Static ์๋น์ค ๊ฐ์ )
>

- #48
    - ๊ธฐ์กด์ ๋ฌธ์ ์ 

      ํ์ด์ฌ์ผ๋ก ํต๊ณ๋ด์ด ํด๋น ๊ทธ๋ํ๋ฅผ ์ฌ์ง์ผ๋ก ์ ์ฅํ๊ณ , ๊ทธ ์ฌ์ง์ ๋ถ๋ฌ์ค๋ ํ์์ผ๋ก ์งํ, ํ์ง๋ง ์ด๋ ํตํฉ๋์ง ์๋ ๋ฌธ์ ๊ฐ ์์ผ๋ฉฐ, ์ฌ์ง์ผ๋ก ๋ณด์์ผ๋ก์จ ๋ถ๊ฐ์ ์ธ ์๋น์ค๋ฅผ ์ ๊ณตํ์ง ๋ชปํจ

    - ํด๊ฒฐ
        - javascript๋ฅผ ์ด์ฉํ์ฌ ๊ทธ๋ํ๋ฅผ ๋ณด์ฌ์ฃผ๋ ํ์์ผ๋ก ์งํ.
        - Model๋ก ๋ฐ์ดํฐ๋ฅผ ๋ด์์ thymeleaf in javascript ๊ธฐ์ ์ ์ฌ์ฉํ์ฌ ๋ฐ์ดํฐ ์ด์ฉ
        - ๋ฉค๋ฒ๋น ์ต๊ทผ 1์ฃผ์ผ post๋ฅผ ๊ฐ์ ธ์ ํด๋น post์ ๋ชจ๋  ์๊ฐ์ ๋ํ ์๊ฐ์ ์ ์ฉ
            - ์ค์ํ ์ ์ ์๋ฒฝ4์๋ฅผ ํ๋ฃจ์ ๋๊ณผ ์์์ผ๋ก ์ ํ๊ธฐ๋๋ฌธ์, ์์ฒญ ์๊ฐ์ด 0~4์ ์ฌ์ด์ผ ๊ฒฝ์ฐ์ ๋๋จธ์ง ์๊ฐ์ผ ๊ฒฝ์ฐ์ ๋ํ ์ฒ๋ฆฌ ๊ตฌ๋ถ ํ์
        - ๊ทธ๋ํ๋ก๋ ๋์  ์๊ฐ์ ๋ณด์ฌ์ค
        - ์ถ๊ฐ๋ก ํด๋น ์ผ์ฃผ์ผ๊ฐ์ ๋์  ์๊ฐ์ ๋ํ ๋ญํน ํ์
        - StaticController ์ฃผ์ ์ฝ๋

            ```java
            @GetMapping("/data")
            public String data(Model model, Authentication authentication) throws IOException {
                if (authentication == null) {
                    return "redirect:/";
                }
                List<MemberDto> members = memberService.findAll().stream().map(MemberDto::new).collect(Collectors.toList());
                Map<MemberDto, Map<Integer, Times>> memberStatic = new HashMap<>();
                List<Integer> days = getDays(LocalDateTime.now());
                for (MemberDto member : members) {
                    Map<Integer, Times> staticsData = getStaticsData(member);
                    memberStatic.put(member, staticsData);
                    member.setTime(staticsData.get(days.get(days.size()-1)));
                    if (member.getTime().getHour() != 0) {
                        Random rand = new Random();
                        int r = rand.nextInt(255);
                        int g = rand.nextInt(255);
                        int b = rand.nextInt(255);
                        member.setColor("rgba(" + String.valueOf(r) + "," + String.valueOf(g) + "," + String.valueOf(b) + "," + "1)");
                    } else {
                        member.setColor("rgba(236,236,236,1)");
                    }
                }
            
                members.sort(new TimeSorter()); 
            	  ...
            }
            ```

            - getDays : ๊ทธ๋ํ์ ํ์ํ  ๋ ์ง๋ชจ์(์ผ)
            - getStaticData : ๊ทธ๋ํ์ ํ์ํ  ๋ฐ์ดํฐ ์์ฑ
                - 0~4์์ ์์ฒญ : 8์ผ์  ~ ์ดํ์  ๊น์ง์ ๋ฐ์ดํฐ๋ฅผ ๊ฐ์ ธ์ด
                - ์ด์ธ์ ์์ฒญ : 7์ผ์  ~ ํ๋ฃจ ์  ๊น์ง์ ๋ฐ์ดํฐ๋ฅผ ๊ฐ์ ธ์ด
                - ์ถ๊ฐ๋ก ๋ ์ง๋ง๋ค์ ๋ฐ์ดํฐ๋ฅผ ๋ฃ์ด์ฃผ๊ธฐ ์ํด HashMap ์ฌ์ฉ (key: ๋ ์ง, value: ์๊ฐ) ์ด๋, ํด๋น ๋ ์ง์ post๊ฐ ์๋ ๊ฒฝ์ฐ ์ด์  ๋ฐ์ดํฐ๋ฅผ ๋๊ฒจ๋ฐ์ ์ฌ์ฉํ๋๋ก ์ค์  (๋์  ๋ฐ์ดํฐ์ด๋ฏ๋ก)
            - ๋ํ ๊ฐ ๋ฉค๋ฒ๋ณ๋ก ๋๋คํ ์๊น์ ์ค์ ํ์ฌ ๊ฐ ๋ฉค๋ฒ๋ณ์ ๊ทธ๋ํ๊ฐ ๊ตฌ๋ถ ๊ฐ๋ฅํ๋๋ก ์ค์ 
            - ๊ทธ ํ ๋ญํนํ์๋ฅผ ์ํด TimeSorter๋ก ์ ๋ ฌ
        - Javascript ์ฃผ์ ์ฝ๋

            ```jsx
            var datasets = [];
            [# th:each="user, stat : ${staticsDataMap}"]
            var user[[${stat.count}]] = [[${user.value}]];
            if (user[[${stat.count}]][staticDays[6]].hour !== 0) {
                datasets.push({
                    label: [[${user.key.name}]],
                    lineTension: 0.3,
                    backgroundColor: "rgba(33,37,41,0)",
                    borderColor: [[${user.key.color}]],
                    pointRadius: 3,
                    pointBackgroundColor: [[${user.key.color}]],
                    pointBorderColor: [[${user.key.color}]],
                    pointHoverRadius: 3,
                    pointHoverBackgroundColor: [[${user.key.color}]],
                    pointHoverBorderColor: [[${user.key.color}]],
                    pointHitRadius: 10,
                    pointBorderWidth: 2,
                    data: [user[[${stat.count}]][staticDays[0]].hour + Math.round(user[[${stat.count}]][staticDays[0]].min/60),
                        user[[${stat.count}]][staticDays[1]].hour+ Math.round(user[[${stat.count}]][staticDays[1]].min/60),
                        user[[${stat.count}]][staticDays[2]].hour+ Math.round(user[[${stat.count}]][staticDays[2]].min/60),
                        user[[${stat.count}]][staticDays[3]].hour+ Math.round(user[[${stat.count}]][staticDays[3]].min/60),
                        user[[${stat.count}]][staticDays[4]].hour+ Math.round(user[[${stat.count}]][staticDays[4]].min/60),
                        user[[${stat.count}]][staticDays[5]].hour+ Math.round(user[[${stat.count}]][staticDays[5]].min/60),
                        user[[${stat.count}]][staticDays[6]].hour+ Math.round(user[[${stat.count}]][staticDays[6]].min/60)],
                });
            }
            
            [/]
            
            var myLineChart = new Chart(ctx, {
                    type: 'line',
                    data: {
                        labels: [staticDays[0]+'์ผ',staticDays[1]+'์ผ',staticDays[2]+'์ผ',staticDays[3]+'์ผ',staticDays[4]+'์ผ',staticDays[5]+'์ผ',staticDays[6]+'์ผ'],
                        datasets: datasets,
                    },
            ...})
            ```

            - thymeleaf in javascript ๊ธฐ์ ์ for each ๋ฅผ ์ฌ์ฉํ์ฌ user๋ฅผ ๋ง๋ค๊ณ  ํด๋น user์ ๋ฐ์ดํฐ๋ฅผ dataset์ ๋ฃ์ด ๊ทธ๋ํ์ ๋์.


### <220713>

> #42 `Enhancement`: ํ๋ฃจ ๊ณต๋ถ์๊ฐ ๋ญํน <br>
#62 `Back` : MVC ์ต์ ํ
>

- #42
    - ๊ธฐ์กด์ ๋ฌธ์ ์ 

      ์ฐ์ธก ์ํฐํด์์ ๋์  ๊ณต๋ถ์๊ฐ ๊ทธ๋ํ๋ฅผ ์ฌ์ง์ผ๋ก ๋๋ต์ ์ผ๋ก ๋ณด์ฌ์ฃผ๋ ๋ถ๋ถ์ด ์์์ผ๋ ์ฌ์ค์ ์๋ฏธ๊ฐ ์๋ ๊ธฐ๋ฅ์ด์์ผ๋ฉฐ, ๋์๋ณด๋๋ฅผ ์ฌ์ง์ด ์๋ js๋ก ๊ตฌ์ฑํ๊ธฐ๋ก ๊ฒฐ์ . ์ถ๊ฐ๋ก ๊ณต๋ถ ๋๊ธฐ๋ถ์ฌ๋ฅผ ๋์ฌ์ฃผ๊ธฐ ์ํ ๊ธฐ๋ฅ์ด ํ์ํ์

    - ํด๊ฒฐ (**ํ๋ฃจ ๊ณต๋ถ์๊ฐ ๋ญํน**์ผ๋ก **๋์ฒด**)
        - ๊ธฐ์กด์ ์ ๊ณตํ๋ ๊ธฐ๋ฅ ์ค ํ๋ฃจ ๊ณต๋ถ์๊ฐ ๊ณ์ฐ ๋ก์ง(`getTotalStudyTimes`())์ ์ด์ฉ
        - ์ด ๋ก์ง์ ๋ชจ๋  Member๋ค์๊ฒ ์ ์ฉํ์ฌ ๊ฐ Member์ ํ๋ฃจ ๊ณต๋ถ์๊ฐ์ ๊ณ์ฐํ๋๋ก ์งํ

            ```java
            for (MemberDto m : memberDtos) {
                Optional<Post> eachOptionalPost = postService.getRecentPost(m.getId());
                PostViewDto memberRecentPostDto = eachOptionalPost.map(PostViewDto::new).orElse(null);
                if (memberRecentPostDto != null) {
                    m.setTime(getTotalStudyTimes(memberRecentPostDto));
                } else {
                    m.setTime(null);
                }
            }
            ```

        - ์ด ์๊ฐ์ ๊ฐ๊ฐ์ MemberDto์ time property์ ์ ์ฅ ํ ์ด๋ฅผ View Template์์ ๋ฐ์, ์ด์ฉ.
        - ์ฌ๊ธฐ์ ์ค์ํ ๋ถ๋ถ์, ํ๋ฃจ ๊ณต๋ถ์๊ฐ **๋ญํน**์ด๊ธฐ ๋๋ฌธ์, ์ด๋ค์ ์๊ฐ(`Times`)์ ๋ฐ๋ผ ์ ๋ ฌ์ด ํ์ โ  **custom Comparator**(`TimeSorter`) ์์ฑ ํ `sort` ์ ์ฉ

            ```java
            public class TimeSorter implements Comparator {
              @Override
              public int compare(Object o1, Object o2) {
                  //o1 - o2 = ASC , o2 - o1 = DESC
                  MemberDto m1 = (MemberDto) o1;
                  MemberDto m2 = (MemberDto) o2;
                  int totalTime1 = 0;
                  int totalTime2 = 0;
                  if (m1.getTime() != null) totalTime1 = m1.getTime().getHour() * 60 + m1.getTime().getMin();
                  if (m2.getTime() != null) totalTime2 = m2.getTime().getHour() * 60 + m2.getTime().getMin();
                  return totalTime2 - totalTime1;
              }
            }
            ```

            - Times๊ฐ hour์ min์ผ๋ก ๊ตฌ์ฑ๋์ด ์๊ธฐ ๋๋ฌธ์, ์ด๋ฅผ ๋ชจ๋ minute์ผ๋ก ๋ณ๊ฒฝ ํ ์ด๋ค์ ๋ํ ์ ๋ ฌ์ ์คํ.

            ```java
            List<MemberDto> memberRanking = new ArrayList<>(memberDtos);
            memberRanking.sort(new TimeSorter());
            ```

- #62
    - ๊ธฐ์กด์ ๋ฌธ์ ์ 
        - ํ์ด์ง์ ๋ฐ๋ฅธ Controller๊ฐ ์กด์ฌ โ ๋ฒ์กํ ๊ตฌ์ฑ
        - Static ํ๋ฉด๋ HomeController์ ์กด์ฌ โ ๊ด์ฌ์ฌ ๋ถ๋ฆฌ ํ์
    - ํด๊ฒฐ
        - ํ์ด์ง Controller ํตํฉ โ `@RequestMapping({"/", "/{page}"})` , `@PathVariable(name = "page", required = false) Integer page` , `if (page == null) page = 1;`
            - url๋ก ๋ณ์๋ฅผ ๋ฐ์์ค๊ณ  ํด๋น ๋ณ์๊ฐ ์๋ค๋ฉด 1๋ก ์ค์ ํ์ฌ main์ page 1๋ก ๊ฐ ์ ์๋๋ก ์ค์ 
        - StaticController ์์ฑ โ (HomeController์ ๋ถ๋ฆฌ)

### <220618> ~ <220630>
- **Spring MVC ์ง์ค ๊ณต๋ถ! (๊ฐ์ ๋ฐ ๊ฐ์ธ ํ์ต์ผ๋ก ์ธํด ๊ฐ์์์ ์งํํ ๊ฐ๋ฐ ์์ธ์ ๊ฐ๋ฐ์ ์งํํ์ง ์์)**
- ์ถํ ํด๋น ๊ณต๋ถํ ๋ด์ฉ์ ๋ฐํ์ผ๋ก **Semogong Project๋ฅผ Upgrade ์ํฌ ์์ **


### <220615>

> `Front` : ์ฌ์ฉ์ ํธ์๋ฅผ ์ํ ์ ๋๋ฉ์ด์ ์ถ๊ฐ
>

- ํ์ฌ ๋ฌธ์ ์ 
    - Home ํ๋ฉด์์ ๋ฐ์ดํฐ๊ฐ ๋ณ๊ฒฝ๋๊ณ  ์๋ Point๊ฐ ์ด๋์ง ํ์ธ ๋ถ๊ฐ
    - Static ํ๋ฉด์์ ํ์ฌ ์ธ์์ด ๋ณ๊ฒฝ๋๊ณ  ์๋์ง ํ์ธ ๋ถ๊ฐ
- ํด๊ฒฐ
    - Home ํ๋ฉด์ ํด๋น Point์ ๋ฌผ๊ฒฐ ์ ๋๋ฉ์ด์์ ์ถ๊ฐํ์ฌ ํ์ฌ ๋ฐ์ดํฐ๊ฐ ๋ณํ๊ณ  ์๋ ์ง์ ์ ๋ํด Focusํ  ์ ์๋๋ก ์ค์ 
    - Static ํ๋ฉด์ Live ๋ฒํผ์ ํ์ฌ ๋ฐ์ดํฐ๋ฅผ ๋ถ๋ฌ์ค๊ณ  ์์ผ๋ฉด ์ด๋ก์ ๊น๋นก์์ ์ฃผ์ด ๋ฐ์ดํฐ๊ฐ ๋ฐ์๋๊ณ  ์๋ค๋ ์ฌ์ค์ ์ฌ์ฉ์์๊ฒ ์ธ์งํ  ์ ์๋๋ก ์ค์ 

### <220614>

> `Back` : ์ธํฐ์ํฐ๋ฅผ ํตํ ์ต์ ํ
>

- ํ์ฌ ๋ฌธ์ ์ 
    - ํ์ฌ Temp โ Info๋ก ๋ฐ๊พธ๋ ๋ก์ง์ ํ ์๋น์ค๊ฐ ์์๋  ๋ ๋ชจ๋  Temp๋ฅผ Info๋ก ๋ฐ๊พธ๋ ์์ ๋ฐฉ์์ผ๋ก ์งํ ์ค
    - ํ์ง๋ง ํ์ฌ ๋ฐฉ์์ ์ฌ์ค์ ๋นํจ์จ์ ์. ๋ชจ๋  DB๋ฅผ ๊ฐ์ ธ์ค๋ ๊ฒ๋ณด๋ค, ์ต๊ทผ ์์ฒญ์์ ์ ๊ธฐ์ตํ๊ณ  ๊ทธ ์ดํ๋ก ์๋ก ๋ค์ด์จ Temp๋ค๋ง์ Info๋ก ๋ฐ๊พธ๋ ๋ฐฉ๋ฒ์ด ํ์
- ํด๊ฒฐ
    - Point์ ์ต๊ทผ Info์ ๋ณ๊ฒฝ์์ ์ ๊ธฐ๋กํ์ฌ
    - ํด๋น ๋ณ๊ฒฝ์์  ์ดํ์ ์๋ก์ด Temp๊ฐ ์๋์ง ํ์ธํ์ฌ ํด๋น Temp๋ฅผ Info๋ก ๋ณ๊ฒฝํ๋ ๋ฐฉ์์ผ๋ก ์งํ
    - ๋ํ ์ด๋ ์ด๋ค ์๋น์ค๋  ์์๋ ๋ ์ ์ฉ์ด ๋์ด์ผ ํ๋ฏ๋ก **์ธํฐ์ํฐ** ๊ฐ๋ ๋์
- UpdateInterceptor

    ```java
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        List<Point> pointList = pointService.findAll(); // find all Point
        for (Point point : pointList) { // each point
            infoService.createInfo(point.getId()); // make ์ฐ๊ด๊ด๊ณ with Point (์๋ก ์์ฑ๋ Temp๋ค์ ๋ํด์ ์๋ก์ด Info ์์ฑ)
            pointService.updatePoint(point.getId()); // point์ lastCommittedTime update (์ต๊ทผ Info update ์๊ฐ update)
        }
        return true;
    }
    ```

    - CheckPoint ์ดํ ์๋ก ์์ฑ๋ Temp๋ค์ ๋ํด์ ์๋ก์ด Info ์์ฑ
    - CheckPoint Update
    - ๋ง์ง๋ง์ผ๋ก WebConfig๋ฅผ ํตํด Interceptor ๋ฑ๋ก

### <220612>

> `Enhancement` : ๋์  ํต๊ณ ์ ๋ณด ์กฐํ ๊ฐ๋ฐ
>

- Static์์ ๋ณด์ฌ์ค ๋์  ํต๊ณ ์ ๋ณด์ ๋ํ ๊ฐ๋ฐ ์งํ
- ํต๊ณ ๋ก์ง
    - ํด๋น Point์ ๋ํ ๋ชจ๋  Info๋ฅผ ์กฐํ
    - ์กฐํํ ํ ๋ ์ง๋ณ๋ก Info๋ฅผ ๋๋๊ณ  ํด๋น ๋ ์ง์ ๋ํ Info ๊ฐ์์ ํ๊ท ์น๋ฅผ model์ ๋ด์ rendering
    - ์ผ์ฃผ์ผ๊ฐ์ ๋ฐ์ดํฐ, ์๊ฐ๋ณ ๋ฐ์ดํฐ ๋ชจ๋ ํ๊ณณ์ ์ ์ฅ
    - ํด๋น ๋ฐ์ดํฐ๋ฅผ Pie,Arial Chart ๋ฅผ ๊ตฌ์ฑํ๋ js ์ ํ ๋น (ํฌํ๋ฆฟ ์ด์ฉ)

<aside>
๐ก <strong> JavaScript์์ model์ attribute(model์ ๋ด๊ฒจ์ ธ ๋๊ฒจ์จ ๊ฐ) ์ฌ์ฉํ๊ธฐ!</strong> <br>
<code>[[ ]]</code> ์ฌ์ฉ!(HTML ํ๊ทธ์ ์์ฑ์ด ์๋๋ผ <b>HTML ์ฝํ์ธ  ์์ญ์์์ ์ง์ </b> ๋ฐ์ดํฐ๋ฅผ ์ถ๋ ฅํ๊ณ  ์ถ์ ๋ ์ฌ์ฉ) โ <code>var temp = [[ ${temp} ]]</code>

</aside>

### <220610>

> `Back` : Temp, Info Repository, Service ์ฌ์ ์
>

- Temp, Info Repository, Service  ์ฌ์ ์
    - ๊ธฐ์กด์ ์ฌ์ฉํ๋ Info์ Domain, Service, Repository๋ฅผ ๋ชจ๋ Temp๋ก ์ด์ , ๊ทธ ํ Info Repository, Service ์๋ก ์ ์
    - ํต์ฌ์ Temp๋ก ๋ฐ์์จ ๋ผ์ด๋ธ ๋ฐ์ดํฐ๋ฅผ Info๋ก ๋ฐ๊พธ์ด ์ ์ฅํ๋ ๊ฒ. ์ฆ, Temp์ ๋ฐ์ดํฐ๋ฅผ ๊ทธ๋๋ก ๋ฐ์์ ์ฐ๊ด๊ด๊ณ๋ฅผ ์ ์ํ๊ณ  Entity๋ก์จ ๊ด๋ฆฌ๋๊ฒ ๋ง๋๋ ๊ฒ
- Info Repository
    - ์ ์ฅ๋ง ํ๋ฉด ๋จ (์กฐํ๋ ์ฐ๊ด๊ด๊ณ๋ฅผ ๋งบ์ Point๋ก ์งํ) โ `em.persist(info)`
- Info Service
    - Info ์์ฑ ๋ฉ์๋๋ง ์กด์ฌ โ (๋งคํ ๋ก์ง: ) ํด๋น ๊ตฌ์ญ์ ๋ฐ๋ผ Point์ ๋งคํ์์ผ์ค ํ ์ ์ฅ.  ๊ทธ ๊ฒฐ๊ณผ๋ก ์ธก์ ๋ ์์น๋ค์ ๋ฐํ์ผ๋ก Point๋ฅผ ์์ 

### <220609>

> `Back` : ์ด์  Info ๋ฐ์ดํฐ์ Point์ ์ฐ๊ด๊ด๊ณ ์์ฑ ํ์
`Back` : Temp, Info Domain ์ฌ์ ์
>

- ๋ฐ๊ฒฌ๋ ๋ฌธ์ ์ 
    - ์ค์๊ฐ ๋ฐ์ดํฐ๋ก ๋ฐ์์ค๋ Info๋ฅผ ๊ทธ ์ฆ์ Floating ํด์ฃผ๋ ๊ธฐ๋ฅ์ ๊ฐ๋ฅํ๋, Info์ ๊ณผ๊ฑฐ ๋ฐ์ดํฐ๋ฅผ ๊ฐ์ ธ์ฌ ๋ ์ด๋ฏธ ํ๋ฒ ๋งคํ๋์๋ Point์ ๋ค์ ํ๋จ ๋ก์ง์ ํตํด ๋งคํ์ด ํ์ํจ
    - ๋ํ Info๊ฐ Entity๋ก์จ **์ ์ฅ**๋์ง ์์๊ธฐ์ ๋ฐ์ํ๋ ๋ถํธํจ ํด๊ฒฐ ํ์
- ํด๊ฒฐ๋ฒ
    - ๊ธฐ์กด์ ๋ฐ์ดํฐ๋ฅผ ๊ฐ์ ธ์์ ์๋ก Entity๋ก ๊ด๋ฆฌ๋๋ ๋ฐ์ดํฐ๋ฅผ ์์ฑ โ Temp๋ผ๋ Object๋ก ๊ธฐ์กด์ ๋ฐ์ดํฐ ๋ฐ์์ด
    - ์ถ๊ฐ๋ก Info๋ฅผ ๊ฐ์ ธ์ฌ ๋ point์ ์ฐ๊ด๊ด๊ณ๋ฅผ ๋งบ์ด ์๋ก ์ฐธ์กฐํ  ์ ์๋๋ก ์ค์ 

        ```jsx
        //==์ฐ๊ด๊ด๊ณ ๋ฉ์๋==//
        private void setPoint(Point point) {
            this.point = point;
            point.getInfos().add(this);
        }
        ```

    - ๊ธฐ์กด์ ์ฌ์ฉํ๋ Info์ Domain, Service, Repository๋ฅผ ๋ชจ๋ Temp๋ก ์ด์ , ๊ทธ ํ Info ์๋ก ์ ์

### <220608>

> `Front` : Home, Static ํ๋ฉด ์์ 
`Back` : HomeRestController ๊ฐ๋ฐ
>

- Home, Static ํ๋ฉด ์์ 
    - Home
        - Kakao Map API๋ฅผ ์ด์ฉํ ์ง๋์ Point์ ์ขํ๋ฅผ ๋งคํ์์ผ floating
        - ๊ฐ Point๋ค์ ๋ํ ์ค์๊ฐ ๋ณ๋๋ ํ์๋ฅผ ์ํ ๊ฐ๋ฐ โ ajax ๋น๋๊ธฐ ํต์ , REST API ์ด์ฉ โ REST API ๋ฅผ ์ํ Controller ๊ฐ๋ฐ ํ์

        ```jsx
        $( document ).ready(function detection_change() {
            interval = setInterval(function () {
                $.ajax({
                    url: '/infos',
                    type: "GET"
                })
                    .done(function (response) { ... //๋ก์ง }
        })
        ```

    - Static
        - ์ค์๊ฐ ์ธ์ ์ธก์ ๋ ํ์ (๋ฒํผ์ ๋๋ฅด๋ฉด ๋ณผ ์ ์๊ณ , ์๋๋ฉด ์๋ณด์ฌ์ฃผ๋ ๋๋์ผ๋ก ์งํ โ ๋ฒํผ์ ๋๋ฅด๋ฉด interval ํจ์๋ฅผ ์ฌ์ฉํ์ฌ 3์ด๋ง๋ค ๋ฐ์ดํฐ๋ฅผ ์์ฒญ, JSON์ผ๋ก ๊ฐ์ ๋ฐ์์ด)
        - ์ด ๋ํ ajax ๋น๋๊ธฐ ํต์ , REST API ์ด์ฉ โ REST API ๋ฅผ ์ํ Controller ๊ฐ๋ฐ ํ์
- HomeRestController ๊ฐ๋ฐ
    - REST API ๋ฅผ ์ํ Controller ๊ฐ๋ฐ (`@RestController`)
    - Homeํ๋ฉด๊ณผ Static ํ๋ฉด์ ๋ฟ๋ ค์ค JSON ๋ฐ์ดํฐ ์์ฑ ๋ฐ ๋ฐํ
    - JSON ๋ฐ์ดํฐ๋ ์ค์๊ฐ์ผ๋ก ์ธก์ ๋ ์ธ์์ ์๋ฅผ ๋ณด์ฌ์ฃผ๋ count์ ์ด ๋ฐ์ดํฐ ๋ฆฌ์คํธ๋ฅผ ๋ฐํ (`Result<T>` ์ฌ์ฉ)

### <220606>

> `Back` : HomeController ๊ฐ๋ฐ
>

- ๊ณตํต
    - Home ํ๋ฉด์ด๋  Static ํ๋ฉด์ด๋  ๋ชจ๋  Point๋ค์ ๋ํ ์ ๋ณด๊ฐ ํ์ํ๊ธฐ ๋๋ฌธ์ point list๋ฅผ ๊ธฐ๋ณธ์ ์ผ๋ก Model์ ์ถ๊ฐํด์ค. (`@ModelAttribute` ์ฌ์ฉ)
- Home
    - API๋ฅผ ์ด์ฉํ๊ธฐ ๋๋ฌธ์ ๋ฐ๋ก Model์ ์ถ๊ฐํด์ค ๊ฒ๋ค์ด ์์.
    - Point List๋ ๋ฏธ๋ฆฌ ์ถ๊ฐ๋ ์ํ์ model ์ฌ์ฉ
- Statics
    - ๊ฒ์ํด์ ๋ค์ด์ค๋ ์ํฉ๊ณผ, ๊ธฐ๋ณธ static ํ๋ฉด ๋ถ๋ฆฌ (๋ฐ์ดํฐ ๋ถ๋ฆฌ)
    - ๊ทธ์ ๋ํ ์ถ๊ฐ์ ์ธ ์ ๋ณด(Live Count) ๋ฑ ์ถ๊ฐ
    - ์ถ๊ฐ์ ์ผ๋ก ํ์ํ ๋ถ๋ถ : ๋ฐ์ดํฐ๊ฐ ์ถ๊ฐ๋๋ฉด ๋์ ํต๊ณ ๋ฐ์ดํฐ์ ๋ํ ์ฒ๋ฆฌ ํ์.
    
### <220605>

> `Back` : Info Service, Repository ๊ฐ๋ฐ
>

- Info ๋๋ฉ์ธ์ ์ฃผ์์ 
    - ์ด๋ฏธ ์ค์๊ฐ์ผ๋ก ์ ์ฅ๋์ด ์๋ ๋ฐ์ดํฐ๋ฅผ ๊ฐ์ ธ์์ floatingํ๋ ์ญํ 
    - ๊ทธ๋ฌ๋ฏ๋ก ๋ฐ๋ก ์ ์ฅํ  ํ์๋ ์๊ณ , ๋ฐ์ดํฐ๋ฅผ ๊ฐ์ ธ์ค๋ ์ญํ ๋ง ํ๋ฉด ๋จ
    - ์ฌ๊ธฐ์ ๊ธฐ์กด์ ๋ฐ์ดํฐ๋ฅผ ๊ฐ์ ธ์ค๊ธฐ ์ํด์  JpaRepository, NativeQuery ๋ฅผ ์ฌ์ฉํด์ผ ๋จ
- ๊ฐ๋ฐ
    - 10์ด~ํ์ฌ ์ ๋ฐ์ดํฐ๋ฅผ ๋ฐ์์ค๋ method โ static ํ๋ฉด์์ ์ฌ์ฉ

        ```java
        @Query(value = "SELECT * " +
                    "FROM temp t " +
                    "where t.date >= DATE_SUB(CONVERT_TZ(NOW(),'SYSTEM','Asia/Seoul'), INTERVAL 10 SECOND);", nativeQuery = true)
            List<Temp> getLiveData();
        ```

        - ํด๋น Entity๋ก ์ ์ฅ๋ ๋ฐ์ดํฐ๋ฅผ ๊ฐ์ ธ์ค๋ ๊ฒ์ด ์๋ ๊ธฐ์กด์ ์๋ ๋ฐ์ดํฐ๋ฅผ ๊ฐ์ ธ์ค๊ธฐ ์ํด์ JpaRepository, NativeQuery ์ฌ์ฉ
        - 10์ด ์ ~ํ์ฌ์ ๋์  ๋ฐ์ดํฐ๋ฅผ ๊ฐ์ ธ์ด โ DATE_SUB() sql๋ฌธ ์ฌ์ฉ
    - ์ ์ฅ์ ๋ฐ๋ก ํ์์์

### <220604>

> `Back` : Basic Point Service, Repository ๊ฐ๋ฐ
>

- Repository ๋จ
    - ์ ํด์ง ์ง์ ์ ๋ํด์ ์ ์ฅํ๊ธฐ ๋๋ฌธ์ `@PostConstruct`๋ฅผ ํตํด ๋ฏธ๋ฆฌ DB ์ ์ฅ. (1ํ์ฑ initDB)
    - ๋ค๊ฑด์กฐํ, ๋จ๊ฑด์กฐํ, ์ด๋ฆ์ผ๋ก ์กฐํ. 3๊ฐ์ง method ์์ฑ
- Service ๋จ
    - ๋ฆฌํฌ์งํ ๋ฆฌ์ ์ญํ ์ ๋จ์ ์์
- ์ถ๊ฐํ  ์ 
    - ํ์ฌ๋ ๋์ ๋ ๋ฐ์ดํฐ๊ฐ ์๊ธฐ์ ๋์ ํต๊ณ๋์ ์ํ ๋ฐ์ดํฐ์ ๋ํ Repository๋ Service๊ฐ ์ ์๋์ด ์์ง์์ โ ๋ฐ์ดํฐ๊ฐ ์ด๋์ ๋ ๋์ ๋ ์ดํ ๊ฐ๋ฐ ํ์ 

### <220602>

> `Back` : Domain ๊ฐ๋ฐ
>

- ํ์์ฑ
    - ํ์ฌ ํ์ํ ๋ถ๋ถ์ ์ฅ์(๊ฒ์ถ ์ง์ )๋ฅผ ์ฐ๊ฒฐํ  Point์ ์ค์๊ฐ์ผ๋ก DB์ ์ ์ฌ๋๋ ๊ฒ์ถ๊ฐ์ฒด์ ๋ณด๋ฅผ ์ฐ๊ฒฐํ  Info๊ฐ ํ์
- ๊ฐ๋ฐ ๊ณผ์ 
    - Point ๋๋ฉ์ธ ๊ฐ๋ฐ
        - field ์ค์ , ์์ฑ ๋ฐ ์์  ๋ฉ์๋ ์์ฑ
    - Info ๋๋ฉ์ธ ๊ฐ๋ฐ
        - ๊ธฐ์กด์ DB์ ์ฐ๊ฒฐ
        - field ์ค์ , ์์ฑ ๋ฐ ์์  ๋ฉ์๋ ์์ฑ

### <220601>

> `Front` : ํฌํ๋ฆฟ ์ปค์คํ (home, static)
>

- ํ๋ก ํธ๋ฅผ ์ ๋ฌธ์ ์ผ๋ก ํ๋ ์ธ์์ด ์์ด, ๋ฌด๋ฃ๋ก ์ ๊ณต๋๋ ํํ๋ฆฟ์ ์ฌ์ฉํ๊ธฐ๋ก ๊ฒฐ์ . ์ฐ๋ฆฌ๊ฐ ํ๊ณ ์ ํ๋ ์๋น์ค์ ๋ง๋ ๋์๋ณด๋ ํฌํ๋ฆฟ์ ๋ฐ์์ ์ปค์คํ.
- ์ฌ๋ฌ html ํ์ผ ์ค home ํ๋ฉด๊ณผ static ํ๋ฉด๋ง ์ด๋ฆฌ๊ณ  ์ด๋ค์ ๋ํด ์ปค์คํ ์งํ.


### <2205010> ~ <220530>
- **Spring MVC ์ง์ค ๊ณต๋ถ! (๊ฐ์ ๋ฐ ๊ฐ์ธ ํ์ต์ผ๋ก ์ธํด ๊ฐ์์์ ์งํํ ๊ฐ๋ฐ ์์ธ์ ๊ฐ๋ฐ์ ์งํํ์ง ์์)**
- ์ถํ ํด๋น ๊ณต๋ถํ ๋ด์ฉ์ ๋ฐํ์ผ๋ก **Semogong Project๋ฅผ Upgrade ์ํฌ ์์ **

### <220508>

> **Tag : version 1.0.11** <br>
**#39** `Enhancement` : ์ค์๊ฐ ํ์ต ์๊ฐ reload ๋ฒํผ ์ ๋๋ฉ์ด์ ์ถ๊ฐ
>

- **#39**
    - ๊ธฐ์กด์ ๋ฌธ์ ์ : ์ค์๊ฐ ํ์ต ์๊ฐ reload ๋ฒํผ์ ๋๋ฌ๋ ์ ๋๋ ค์ก๋์ง๋ฅผ ํ์ธํ  ๋ฐฉ๋ฒ์ด ์์์ โ alert์ ํ๋ คํ์ง๋ง, alert์ ๋จ๋ฉด ๊บผ์ผ๋๋ค๋ ๊ท์ฐฎ์ ํ์ธ
    - ํด๊ฒฐ : reload ๋ฒํผ์ ๋๋ฅผ๊ฒฝ์ฐ, reload img ๊ฐ 360๋ rotate ๋๋ฉด์ ์๊ฐ์ ์ผ๋ก ์ฌ๋ถํ ๋๋ ํจ๊ณผ ์ ๊ณต โ `<img id="reload" src="/images/reload.png" height="20px" width="20px">`

        ```css
        .reload {
            transform:             rotate( 360deg );
            -moz-transform:    rotate( 360deg );
            -ms-transform:     rotate( 360deg );
            -o-transform:      rotate( 360deg );
            -webkit-transform: rotate( 360deg );
            transition:             transform 1000ms ease;
            -moz-transition:    -moz-transform 1000ms ease;
            -ms-transition:     -ms-transform 1000ms ease;
            -o-transition:      -o-transform 1000ms ease;
            -webkit-transition: -webkit-transform 1000ms ease;
        }
        ```

        ```jsx
        var rotate = document.getElementById('reload')
        
        if (rotate) {
            rotate.addEventListener('click', function() {
                this.className = 'reload';
                window.setTimeout(function() {
                    rotate.className = ''
                }, 1000);
            }, false);
        }
        ```
    
### <220507>

> **#39** `Front` : ์ค์๊ฐ ํ์ต ์๊ฐ reload ๊ธฐ๋ฅ ์ถ๊ฐ <br>
**#39** `Bug` : 23์, 00์์ ์ฐจ์ด ์ค๋ฅ ํด๊ฒฐ
>

- **#39**

    - ๊ธฐ์กด์ ๋ฌธ์ ์  : ํ์ฌ ๊ณต๋ถ์๊ฐ์ ํ์ธํ๋ ค๋ฉด ๋ฌด์กฐ๊ฑด home์ reloadํด์ผ ๋๋ ๋นํจ์จ์ ์ธ ๋ฐฉ๋ฒ์ ์ ํํด์ผ ํ์. โ ๋คํธ์ํฌ, ๋ ๋๋ง ๊ณผ๋ถํ ๊ฐ๋ฅ
    - ํด๊ฒฐ : ํ์ต ์๊ฐ ์์ฒด์ reload ๋ฒํผ์ ์ถ๊ฐํ์ฌ ๋ฒํผ์ ํด๋ฆญํ  ๊ฒฝ์ฐ ajax api ํต์ ์ ํตํด ํด๋น ๊ณ์ฐ๋ ํ์ต ์๊ฐ๋ง ๋ฐ์์ ๋ ๋๋ง ํ  ์ ์๋๋ก ์ค์ 

        ```jsx
        function reload_times(memberId) {
            $.ajax({
                url: 'members/times/' + memberId,
                type: "GET"
            })
                .done(function (response) {
                    $('#member_time').replaceWith(
                        '<div id="member_time"> ์ค๋์ ํ์ต ์๊ฐ : ' + response.hour + '์๊ฐ '+ response.min +'๋ถ </div>'
                    )
                })
        }
        ```
      
### <220506>

> **Tag : version 1.0.10** <br>
**#2** `Bug` : ๋๊ธ submit enter ์๋ ฅํ  ์ ์๋๋ก ์ค์ , apple์์์ ์ค๋ฅ ์์  <br>
**#39** `Enhancement` : post edit ์์๋ ์ค์๊ฐ ํ์ต ์๊ฐ์ด ๋ณ๊ฒฝ๋๋๋ก <br>
>

- **#2**
    - ๊ธฐ์กด์ ๋ฌธ์ ์ : input์ onkeyup์ ์ฌ์ฉํ  ๊ฒฝ์ฐ apple๊ธฐ๊ธฐ์์ ํ๊ธ๋ก ์์ฑํ  ๊ฒฝ์ฐ ํด๋น ์ฐ๊ฒฐ๋ javascript ํจ์๊ฐ ์ฐ์ํด์ 2๋ฒ ์๋ํ๋ ์ค๋ฅ ๋ฐ์
    - ํด๊ฒฐ: onkeyup์ ๋ชจ๋ onkeypress๋ก ๋ณ๊ฒฝ
- **#39**
    - ๊ธฐ์กด์ ๋ฌธ์ ์  : post eidt ์ ์ค์๊ฐ ํ์ต ์๊ฐ์ ๋ณ๊ฒฝ๋์ง ์์์
    - ํด๊ฒฐ :
        - post edit ์ ๋ณ๊ฒฝ๋ ์๊ฐ์ ๋ฐ์์ ๊ทธ ์๊ฐ javascript ์์์ ํ์ฌ ํ์ต ์๊ฐ ๊ณ์ฐ ๋ก์ง(์์์ ์์ฑํ ๋ก์ง)์ ํตํด ํ์ฌ๊น์ง์ ํ์ต์๊ฐ์ ๋ฐ์ํ์ฌ ๋ณ๊ฒฝ.
        - ์ด ๋ํ replace๋ฅผ ํตํด ์์๋ฅผ ํ์ฌ ๋ณ๊ฒฝ๋ ์์๋ก ๊ต์ฒด.

        <aside>
        ๐จ ๋ชจ๋  post edit์ ๋ํด์ ์ ์ฉ๋๋ฉด ์๋๊ธฐ์ ํด๋น editํ post๊ฐ ์กฐ๊ฑด์ ๋ง๋ post์ผ ๊ฒฝ์ฐ์๋ง ๋ก์ง ์ ์ฉ (focusedPostId โ ์ต๊ทผ์ ์์ฑ๋ post์ id, recentPost๋ ํ์ฌ ์ฌ์ฉ์๊ฐ end์ผ ๊ฒฝ์ฐ์๋ ๋ฐํ๋์ง ์๊ธฐ์ focusedPost๋ฅผ ์๋ก ์ง์ )

        </aside>

### <220505>

> **Tag : version 1.0.9**  <br>
**#39** `Enhancement` : ์ค์๊ฐ ํ์ต ์๊ฐ ๊ธฐ๋ฅ ์ถ๊ฐ
>

- **#39**
    - ๊ธฐ์กด์ ๋ฌธ์ ์  : ํ์ต ์๊ฐ์ด ๊ธฐ๋ก์ ๋์์ง๋ง, ์ด๋ฅผ ๊ทธ์  ์๊ฐ์ผ๋ก๋ง ๋ณด์ฌ์ฃผ์ด์ ์ฌ์ฉ์๊ฐ ์ค๋ ํ๋ฃจ ์ผ๋ง๋ ๊ณต๋ถ๋ฅผ ํ๋์ง ์ง์  ๊ณ์ฐํ๊ฑฐ๋ ๋์น๋ ๊ฒฝ์ฐ๊ฐ ๋ฐ์
    - ํด๊ฒฐ: ํ๋ก๊ทธ๋จ์์ผ๋ก ์๋ ๊ณ์ฐํ์ฌ ์ง์์ ์ผ๋ก ์ง๊ธ๊น์ง ์์ ์ด ํ๋ฃจ ์ผ๋ง๋ ๊ณต๋ถ๋ฅผ ์งํํ๋์ง ํ์ธํ  ์ ์๋๋ก ์ค์ 
        - `getTotalStudyTimes`, `getTimes`๋ฅผ ํตํด ํ์ฌ ์ํฉ์ ๋ง๋ ์๊ฐ ๊ณ์ฐ ์งํ
        - ๋ก์ง :

          ์กฐํ ์๊ฐ ๊ธฐ์ค (home์ผ๋ก ์ด๋)

            1. 04:00 - 24:00
                - ๊ทธ member์ ๊ฐ์ฅ ์ต์  ๊ธ search
                1. ํด๋น member์ ์ต์ ๊ธ์ด ๋น์ผ 4์ ์ดํ create ๋์๋ค
                    - ๊ฐ์ ธ์์ times ๊ณ์ฐ
                2. ๋น์ผ 4์ ์ด์  โ ์ด์  ๊ธ์ด๋ผ ํ๋จ
                    - โ๊ณต๋ถ๋ฅผ ์์ํ์ง ์์์ต๋๋คโ ์ถ๋ ฅ
            2. 00:00 - 04:00
                - ๊ทธ member์ ๊ฐ์ฅ ์ต์  ๊ธ search
                1. ํด๋น member์ ์ต์ ๊ธ์ด ๋น์ผ create
                    - ๊ฐ์ ธ์์ times ๊ณ์ฐ
                2. ์ ๋  create
                    1. 04์์ดํ create
                        - ๊ฐ์ ธ์์ times ๊ณ์ฐ
                    2. 04์ ์ด์  create
                        - โ๊ณต๋ถ๋ฅผ ์์ํ์ง ์์์ต๋๋คโ ์ถ๋ ฅ

### <220504>
> **Tag : version 1.0.8** <br>
**#2** `Bug` : post edit์์ ์๊ฐ ๋ณ๊ฒฝ ์ ๋ฐ์๋์ง ์๋ bug ์์ , comment ์ญ์ , post ์ญ์  ์ ๋ชจ๋ฌ ์ ์ง
>

- **#2**
    - ๊ธฐ์กด์ ๋ฌธ์ ์ : post๋ฅผ editํ  ๋, main(home) ํ๋ฉด์ title, introduce๋ ์์ ๋์ง๋ง, ๋ณ๊ฒฝ๋ time์ด ๋ฐ์๋์ง ์์. โ ๋ times๋ค์ ๋ํด replace ํด์ค ํ์ ์์
      ๋ํ, comment ์ญ์ , post ์ญ์ ๋ **RestAPI**๋ก ๊ต์ฒด ํ์
    - ํด๊ฒฐ: ๊ธฐ์กด main ํ๋ฉด์ ํด๋น post์ times์ ์ ๋ณด๋ฅผ ๋ํ๋ด๊ณ  ์๋ ์์๋ฅผ ๋น ์์๋ก replaceํ๊ณ  ๋ณ๊ฒฝ๋ ์๊ฐ์ ๋ฐ๋ฅธ ์์๋ค์ appendํ๋ ๋ฐฉ์์ผ๋ก ์งํ
        - ์๊ฐ์ ๋ฐ๋ฅธ ๋ณด์ฌ์ฃผ๋ ๋ก์ง์ ๊ธฐ์กด์ ๋ก์ง๊ณผ ๋์ผ

            ```jsx
            $('#post_times' + id).replaceWith(
                                    '<div id="post_times'+ id +'" class="card-text overflow-hidden d-sm-flex" style="margin: 2px 0px 0px 0px; max-width: 100%;"></div>'
                                )
            var cnt = 0;
            for (var i = 1; i < times.length; i+=2, cnt++) {
                if (cnt % 2 != 0) {
                    $('#post_times' + id).append(
                        '<p class="badge" style="margin: 3px; background-color:#8AA6A3;">' +
                        times[i-1].value + '&nbsp;~&nbsp;' + times[i].value
                        + '</p>'
                    );
                } else {
                    $('#post_times' + id).append(
                        '<p class="badge" style="margin: 3px; background-color:#A1A185;">' +
                        times[i-1].value + '&nbsp;~&nbsp;' + times[i].value
                        + '</p>'
                    );
                }
            }
            if (times.length % 2 == 1) {
                if ((times.length - 1) % 4 == 0) {
                    $('#post_times' + id).append(
                        '<p class="badge" style="margin: 3px; background-color:#A1A185;">' +
                        times[times.length-1].value + '&nbsp;~'
                        + '</p>'
                    );
                } else {
                    $('#post_times' + id).append(
                        '<p class="badge" style="margin: 3px; background-color:#8AA6A3;">' +
                        times[times.length-1].value + '&nbsp;~'
                        + '</p>'
                    );
                }
            }
            ```

        - comment, post ์ญ์  ์์ฒญ์ RestAPI๋ก ๊ต์ฒด

            ```jsx
            function post_delete_check(id) {
                var check = confirm("์ ๋ง ์ญ์ ํ์๊ฒ ์ต๋๊น?");
                if (check){    //ํ์ธ
                    alert("์ญ์ ๊ฐ ์๋ฃ ๋์์ต๋๋ค.")
                    $.ajax({
                        url: 'posts/delete/' + id,
                        type: "DELETE"
                    })
                        .done(function () {
                            location.reload();
                        })
                }else{   //์ทจ์
                    return false;
                }
            }
            ```

            ```jsx
            function delete_check(postId, commentId) {
            
                var deleteForm = {
                    postId : postId,
                    commentId : commentId
                };
            
                var check = confirm("์ ๋ง ์ญ์ ํ์๊ฒ ์ต๋๊น?");
            
                if (check){ //ํ์ธ
                    $.ajax({
                        url: 'comment/delete/' + commentId,
                        type: "DELETE",
                        data: deleteForm
                    })
                        .done(function (fragment) {...}
            }
            ```

            - post๋ฅผ ์ญ์ ํ  ๋๋ ์ญ์  ํ ๋ฐ๋ก home์ผ๋ก ๊ฐ๋ฉด ๋์ง๋ง, comment๋ ์ญ์ ํด๋ ๋ชจ๋ฌ์ ์ ์งํ๋ฉฐ ํด๋น ์ญ์ ๋ฅผ ๋ฐ์ํ ์ํ์ comment list๋ฅผ ์๋ก ๋ณด์ฌ์ค ํ์๊ฐ ์๊ธฐ์ ์ญ์  ํ comment list(์ญ์ ๋ ํ์ comment list)๋ฅผ fragment๋ก ๋ฐ์์ replace ์งํ

### <220503>

> **#2** `Enhancement` : Post ์์  ์ ๋ชจ๋ฌ ์ ์ง <br>
**#28** `Enhancement` : ์ด๋ฏธ์ง Upload ์ API๋ก ๋ณด๋ผ ์ ์๋๋ก ์ค์  <br>
>

- **#2**
    - ๊ธฐ์กด์ ๋ฌธ์ ์ : comment ์์ฑ ์ ์ ๋ชจ๋ฌ์ ์ ์งํ  ํ์๊ฐ ์๋ค๊ณ  ํ๋จํ์ฌ ํด๋น ๊ธฐ๋ฅ์ ์ ์ฉ, ํ์ง๋ง post ๋ํ ์์  ํ submit์ ๋๋ฅด๋ฉด ๋ณ๊ฒฝ๋ ์ฌํญ์ ํ์ธํ๊ธฐ ์ํด์๋ home์ผ๋ก ๋์๊ฐ์ผ ๋๊ณ , modal์ด ๋ด๋ ค๊ฐ๋ ํ์ ๋ฐ์ โ edit ํ ๋ฐ๋ก ๊ทธ ๋ชจ๋ฌ ์์ฒด์์ ํ์ธํ  ์ ์๋๋ก ์ค์  ํ์ (comment์์์ ๋ฌธ์ ์ ์ ์ฌ)
    - ํด๊ฒฐ: ์ด ๋ํ comment ์์ฑ ์ ๋์ํ๋ ๋ก์ง๊ณผ ์ ์ฌํ๊ฒ ์งํ โ Ajax Web API ์์ฒญ์ ํตํด post eidt์ Postํ๊ณ  ํด๋น ๊ฒฐ๊ณผ ๊ฐ์ fragment๋ html๋ก ๋ฐ์์ ๊ธฐ์กด fragment๋ฅผ ๋ณ๊ฒฝํ๋ ๋ฐฉ์์ผ๋ก ์งํ (์ ๊ธ ์์ฑ์ ๋ชจ๋ฌ์ด ์๋, ๊ธฐ์กด ํผ์ผ๋ก ๋ณด์ฌ์ฃผ๋๋ก ์ค์ )
        - post๋ get, post ๋ชจ๋ Ajax api๋ก ์์ฒญ (edit ๋ฒํผ์ ๋๋ฅผ ๊ฒฝ์ฐ ๋ํ ๋ชจ๋ฌ์์์ ์์  ๋ฐ ์์  ์๋ฃ ํ ๋ณ๊ฒฝ ์ฌํญ ํ์ธ์ ์ํจ)
        - `@GetMapping("/posts/edit/{id}")` ์ Ajax api๋ก ์์ฒญํ๋๋ก ์ค์  (๊ฒ์๊ธ ์์  ํผ)

            ```jsx
            function edit_post(id) {
                $.ajax({
                    url: 'posts/edit/' + id,
                    type: "GET",
                })
                    .done(function (fragment) {
                        $('#postModal_content' + id).replaceWith(fragment);
                        simplemde = new SimpleMDE({element: document.getElementById("content"),spellChecker: false});
                        document.getElementById("postEdit_container").scrollIntoView();
                    });
            }
            ```

            - ๋๊ฐ์ด ๊ธฐ์กด ๋ด์ฉ์ fragment๋ก ๋ฐ์ ์จ editForm html(`return "post/editPostForm :: #postEdit_container";`)๋ก ๊ต์ฒด
            - ๊ต์ฒด ํ markdown editor ์ถ๊ฐ.
            - ์๋จ์ผ๋ก ์คํฌ๋กค ์๋ ์ด๋
        - `@PostMapping("/posts/edit/{id}")` ์ Ajax api๋ก ์์ฒญํ๋๋ก ์ค์  (๊ฒ์๊ธ ์์ )

            ```jsx
            function post_edit(id) {
                var formData = $("#formPost").serializeArray();
                formData[formData.length - 1].value = simplemde.value(); // content
                    $.ajax({
                                url: '/posts/edit/' + id,
                                type: "POST",
                                data: formData
                            })
                                .done(function (fragment) {...}
            }
            ```

            - form์ ์๋ ๋ฐ์ดํฐ๋ค์ ๋ฐ์ํ๊ธฐ ์ํด form์ ์๋ ฅ๋ ๋ชจ๋  ๋ฐ์ดํฐ๋ฅผ serializeArray๋ฅผ ํตํด ๋ฐ์์ด
            - ์ฌ๊ธฐ์ simplemde๋ฅผ ์ฌ์ฉํ๋ฉด์ content์ ๋ด์ฉ์ด serializeArray์ ๋ด๊ฒจ์ค์ง ์์ ์ง์  simplemde์ ์๋ ฅ๋ ๊ฐ์ form์์ content์ ๋ฃ์ด์ค
            - ์ดํ๋ ์์ comment ๋ณ๊ฒฝ ๋ก์ง๊ณผ ๋์ผํ๊ฒ ์งํ (ajax api, fragment replace, setAttribute(id), recentpost, ...)
- **#28**
    - ๊ธฐ์กด์ ๋ฌธ์ ์  : ์ด๋ฏธ์ง๋ฅผ iframe์ form์ผ๋ก upload ์งํ. โ ๋นํจ์จ์ ์ธ ๋ฐฉ๋ฒ, Rest API๋ฅผ ํตํด ํด๊ฒฐ ๊ฐ๋ฅ
    - ํด๊ฒฐ :  Ajax Web API ์์ฒญ์ Post๋ฅผ ์ด์ฉํ์ฌ ์๋ก๋๋ฅผ ํ๊ณ  ๋ฐํ ๊ฐ์ผ๋ก ํด๋น Image ๊ฐ์ฒด (value: imgSrc, imgName)๋ฅผ Json ํ์์ผ๋ก ๋ฐ์ ์ด๋ฅผ ์ด์ฉํ์ฌ ๋ณ๊ฒฝ๋ ์ฌ์ง ์ ๋ณด๋ฅผ ์ถ๋ ฅ

        ```java
        @ResponseBody
        @PostMapping("/posts/edit/{id}/img")
        public Image postImgEdit(@PathVariable("id") Long id, @RequestParam("file") MultipartFile[] files) throws IOException {
            MultipartFile file = files[0];
            Image image = s3Service.upload(file);
            postService.editPostImg(id, image);
            return image; // image ๊ฐ์ฒด๋ฅผ Json์ผ๋ก ๋ฐํ
        }
        ```

        ```jsx
        function image_upload(id){
        
            var formData = new FormData();
            var image = $("#file")[0].files[0];
        
            formData.append( "file", image);
        
            if(image == null){
                alert("ํ์ผ์ ์ ํํด์ฃผ์ธ์");
                return;
            }
        
            $.ajax({
                url : "/posts/edit/"+id+"/img"
                , type : "POST"
                , processData : false
                , contentType : false
                , data : formData
                })
        }
        ```

        - ์ด๋ฏธ์ง ์๋ก๋ ํ ๊ฐ์ฒด๋ฅผ Json์ผ๋ก ๋ฐํํด ์ค
        - ์ด๋ฏธ์ง์ ๊ฐ์ file ๋ฐ์ดํฐ๋ formdata๋ก ํ๋ฒ ๋ฌถ์ด์ค์ ๋ณด๋ด์ฃผ๋ ๊ฒ์ด ์์ ํจ
        - ๋ํ file ๋ฐ์ดํฐ๋ฅผ ๋ณด๋ผ ๋๋ ํญ์ `processData : false`, `contentType : false` ์ฃผ์

### <220502>

> **#25** `Enhancement` : ๋์ ๊ณต๋ถ์๊ฐ ํต๊ณ ๋์๋ณด๋
>


- **#25**
    - ๊ธฐ์กด์ ๋ฌธ์ ์  : ๋ด๊ฐ ์ผ์ฃผ์ผ๋์ ์ผ๋งํผ ๊ณต๋ถ๋ฅผ ํ๊ณ  ์๋์ง์ ๋ํ ์ ๋ณด๊ฐ ์์ด, ๋๊ธฐ๋ถ์ฌ๊ฐ ๋ถ์กฑ. ์ผ์ฃผ์ผ ๋์์ ๋์  ๊ณต๋ถ์๊ฐ ํต๊ณ๋์ ๋์๋ณด๋๋ฅผ ํตํด ์ฌ์ฉ์๋ค๋ผ๋ฆฌ ๋น๊ตํ์ฌ ๋ณด์ฌ์ฃผ๋ฉด ๋๊ธฐ๋ถ์ฌํ๋ ๋ฐ์ ์ข์ ํจ๊ณผ๊ฐ ์์ ๊ฒ์ด๋ผ๊ณ  ํ๋จ
    - ํด๊ฒฐ : ๋ถ์ ํด์ ํ์ด์ฌ์ด ๋ ์ต์ํ๊ธฐ์ ํ์ด์ฌ์ matplot์ ํตํด ์๊ฐ์ ๊ณ์ฐํ๊ณ  ploting. ๊ทธ ํ ํด๋น ๊ฒฐ๊ณผ๋ฅผ ์ด๋ฏธ์ง๋ก ์ ์ฅํ์ฌ S3์ ์ ์ฅํ๊ณ , ์น์์๋ ํด๋น ์ด๋ฏธ์ง๋ฅผ ๋ถ๋ฌ์ ๋ณด์ฌ์ฃผ๋ ํ์์ผ๋ก ์งํ. (Side Widget์ Statics, Nav bar์ Data ๋ถ๋ถ)
    - ์ถํ ํด๊ฒฐ : D3.js ์ ๊ฐ์ ํด์ ์ฌ์ฉํ์ฌ ์ข ๋ ์๊ฐ์  ํจ์จ์ฑ์ ๊ทน๋ํํ  ์์ 

### <220501>

> **#2** `Enhancement` : Comment ์์ฑ ์ ๋ชจ๋ฌ ์ ์ง
>

- **#2**
    - ๊ธฐ์กด ๋ฌธ์ ์  : Post ํด๋ฆญ ์ ์์ฑ๋๋ ๋ชจ๋ฌ์์ ๋๊ธ์ ์์ฑํ  ์ ์์ฑ ํ submit ์ ํด๋ฆญํ  ๊ฒฝ์ฐ modal ์ด๊ธฐ์ ์์ฑ๋ ๋๊ธ์ ํ์ธํ๋ ค๋ฉด home์ผ๋ก reload๋จ. โ ์์ฑ ํ ํ์ฌ ๋ชจ๋ฌ์ ์ ์งํ๋ฉด์ ์์ฑ๋ ๋๊ธ๋ชฉ๋ก๋ง์ reloadํ  ํ์๊ฐ ์์
    - ํด๊ฒฐ : Ajax Web API ์์ฒญ์ ํตํด comment๋ฅผ postํ๊ณ  ํด๋น ๊ฒฐ๊ณผ ๊ฐ์ fragment๋ html๋ก ๋ฐ์์ ๊ธฐ์กด fragment๋ฅผ ๋ณ๊ฒฝํ๋ ๋ฐฉ์์ผ๋ก ์งํ
        - `@PostMapping("/comment/api/new/{id}")` ๋ฅผ ajax์ post ์์ฒญ์ผ๋ก mapping

            ```jsx
            function submit_comment(id) {
            
                var commentForm = {
                    postId : id,
                    comment : $("#comment_content"+id).val()
                };
            
                $.ajax({
                    url: 'comment/api/new/' + id,
                    type: "POST",
                    data: commentForm,
                })
                    .done(function (fragment) {...}
            ```

            - fragment๋ ๋ชจ๋ฌ์ ์ผ๋ถ๋ถ์ธ commentList ๋ฅผ replaceํ๊ธฐ ์ํ ๋ ๋๋ง๋ html ํ์ผ
            - ์ฆ, post๊ฐ ์๋ฃ๋๋ฉด ์๋ก ๊ฐฑ์ ๋ ํด๋น Post์ comment list (๋ฐฉ๊ธ ์์ฑ๋ ๋๊ธ์ ์ถ๊ฐํ ๋๊ธ ๋ฆฌ์คํธ)๋ฅผ ๋ ๋๋งํ html ํ์ผ์ ๋ฐ์์ ์ด๋ฅผ ๊ธฐ์กด์ comment list์ ๊ต์ฒด. โ ์์ฑ๋ ๋๊ธ์ url reload ํ์ ์์ด ๋ฐ๋ก ํ์ธ ๊ฐ๋ฅ
        - fragment๋ thymeleaf์์ ์ ๊ณตํ๋ html ์ผ๋ถ๋ง์ ๋ฐํํด์ฃผ๋ ๊ธฐ๋ฅ ์ฌ์ฉ : `return "components/commentList :: #commentPart";` โ ํด๋น id๋ฅผ ๊ฐ์ง ์์๋ง ๋ฐํํด์ค.
        - ๋ฐํ๋ fragment๋ก ๊ธฐ์กด ์์๋ฅผ ๊ต์ฒดํ ํ, id๋ฅผ ์๋ก ํ ๋น โ ๋ชจ๋ฌ์ด๊ธฐ์, ๋ค๋ฅธ post๋ค์ commentPart์ ๊ตฌ๋ถํ๊ธฐ ์ํจ.

            ```jsx
            var commentPart =document.getElementById("commentPart");
            commentPart.setAttribute("id", "commentPart"+id.toString());
            ```

            <aside>
            ๐จ controller์ ๊ฐ mapping return ์ fragment ๊ฐ์ id๋ฅผ ๋ฃ์ผ๋ฉด ์๋๋ ์ด์ 

            1. return "components :: #commentList" + id.toString(); -> ๋  ๊ฒ ๊ฐ์ง๋ง ์ฌ์ค ์ง๊ธ ์ฐ๋ฆฌ๋ ๊ฐ ์์์ id ๊ฐ์ thymeleaf๋ก ๊ฐ์ ํ ๋น ํ๊ณ  ์์ (th:id="${post.id}")
            2. ํ์ง๋ง ์ด๋ ๋ ๋๋ง ํ์ ๊ฐ์ด ํ ๋น๋์ด ์์ฑ ๋๋ ๊ฒ.
            3. ์ฆ controller์์ viewtemplate์ ํตํ html fragment search ์์ ์๋ ๊ฐ์ด ํ ๋น๋์ด ์์ง ์๋ค๋ ๊ฒ.
            4. ๊ทธ๋ฌ๋ฏ๋ก id ๊ฐ์ผ๋ก return์ ์ฃผ์ด viewtemplate์ ํตํด ํด๋น fragment๋ฅผ ์ฐพ์๋ผ ์ ์์
            5. ์ด๋ id๊ฐ์ ์ถํ์ ๋ณ๊ฒฝํ๋ฉด ๋จ ( `setAttribute(โidโ, ... )` )
            </aside>

        - ๋ํ, ๋ณ๊ฒฝ๋ ์ฌํญ์ modal ๋ฟ๋ง์๋, home์ ํด๋น post์ comment ๊ฐ์์๋ ๋ฐ์ํด์ค ํ์๊ฐ ์์ โ ํด๋น post id๋ฅผ ๊ฐ์ง post์ comment ๊ฐ์๋ฅผ ํํํ๋ ์์๋ฅผ replace ํด์ค
        - ๋ํ, recent post๋ผ๊ณ  ํ์ฌ ์์ ์ด ์์ฑ ์ค(๊ณต๋ถ๋ฅผ endํ์ง ์์ ์ํ์ post)์ธ post๋ฅผ ๋ณด์ฌ์ฃผ๋ ๊ธฐ๋ฅ์ด ์๊ธฐ ๋๋ฌธ์, ๋๊ธ ์์ฑ์ด ์ด๋ฃจ์ด์ง post๊ฐ ๊ทธ recent post์ ๋์ผํ  ๊ฒฝ์ฐ recent post ๋ํ ์์ ๊ณผ์ ์ ์ ์ฉ (comment ๊ฐ์ ๋ณ๊ฒฝ)


### <220427>

> **Tag : version 1.0.7** <br>
**#28** `Enhancement` : Post Image ๋ณ๊ฒฝ ์  ๋ณ๊ฒฝ๋ ์ด๋ฏธ์ง ๋์์ฃผ๊ธฐ <br>
**#31** `Back` : ํฌ๋ง์ง๋ฌด ์ ํ์ง ๋ถ์ฌ (์ ํ์ง์ ์ํ๋ ์ง๋ฌด๊ฐ ์์ ์ ์ง์  ์ถ๊ฐ) <br>
**#15** `Front` : OG ํ๊ทธ ๋ถ์ฌ <br>
>


- **#28**
    - ๊ธฐ์กด ๋ฌธ์ ์  : ํ์ฌ ์ด๋ฏธ์ง์ ์ด๋ฆ๋ง์ ์๋ ค์ฃผ๊ณ , ํ์ฌ ์ด๋ฏธ์ง ์์ฒด๋ฅผ ๋ณด์ฌ ์ฃผ์ง ์์์. ๋ํ, Upload ์ ๋ฐ๋ก ๋ณ๊ฒฝ์ด ๋ฐ์๋์ง ์์ ์ฑ๋ก edit์ ์งํํ๊ฒ ์ค์ ํจ
    - ํด๊ฒฐ : ์ด๋ฏธ์ง๋ฅผ ๋์์ฃผ๊ณ  Upload ์ ๋ฐ๋ก ๋ณ๊ฒฝํ์ฌ ๋ณ๊ฒฝ๋ ์ด๋ฏธ์ง๋ฅผ ๋ณด์ฌ์ค ์ ์๋๋ก ์ค์  ํ์
        - Upload ๋ฒํผ์ ๋๋ฅผ ์ a ํ๊ทธ๋ฅผ ํตํด ๋ฐ๋ก img upload ๋ก์ง์ ์คํํ๋ ๊ฒ์ด ์๋, onclick=โupload_check()โ์ ํตํด javascript ์ฝ๋๋ฅผ ๊ฑฐ์ณ ์งํ๋๋๋ก ์ค์ 

            ```jsx
            function upload_check() {
                setTimeout(function(){alert("Upload์๋ฃ ๋์์ต๋๋ค!");location.reload();},500);
            } // 0.5์ด ํ์ ์๋์ผ๋ก ํ์ด์ง๊ฐ reload ๋๋๋ก ์ค์ . -> reload ๋๋ฉด ๋ณ๊ฒฝ๋ ์ ๋ณด๋ฅผ ๋์ธ ์ ์์.
            ```

- **#31**
    - ๊ธฐ์กด ๋ฌธ์ ์  : ํฌ๋ง ์ง๋ฌด์ ์ ํ์ง๋ฅผ ์ฃผ์ง ์์, ์ ํด์ง ํ์ ์์ด ํฌ๋ง ์ง๋ฌด๋ฅผ ์ ๊ฒ ๋จ โ ๋๊ฐ์ ์ง๋ฌด๋ผ๋ ์ฌ์ฉ์๋ง๋ค ๋ค๋ฅธ ์ด๋ฆ์ผ๋ก ์ค์ ํ๋ ๊ฒฝ์ฐ๊ฐ ๋ฐ์
    - ํด๊ฒฐ : ViewTemplate ์ jobs ๋ผ๋ ์ค์ ๋ ์ง๋ฌด List๋ฅผ ๋๊ฒจ์ค datalist๋ก ์ ํํ  ์ ์๋๋ก ์ค์  + ์ํ๋ ์ง๋ฌด๊ฐ ์์ผ๋ฉด ์ง์  ์ ์ ์ ์๋๋ก Input์ text๋ก ์ค์ 

        ```java
        public class DesiredJob {
            static final List<String> jobList =  Arrays.asList("๋ฐฑ์๋ ์์ง๋์ด", "๋ฐ์ดํฐ ์์ง๋์ด", "ML ์์ง๋์ด", "ํ๋ก ํธ์๋ ์์ง๋์ด");
            public static List<String> getJobList() {
                return jobList;
            }
        }
        ```

        ```html
        <div class="input-group mb-3">
            <input **type="text"** autocomplete="off" class="form-control" list="datalistOptions" th:field="*{desiredJob}" placeholder="ํฌ๋ง ์ง๋ฌด๋ฅผ ์ ํํ๊ฑฐ๋ ์๋ ฅํ์ธ์!">
            <**datalist** id="datalistOptions" >
                <th:block **th:each="job : ${jobs}"**>
                    <option **th:value="${job}"**>
                </th:block>
            </**datalist**>
        </div>
        ```

- **#15**
    - ๊ธฐ์กด ๋ฌธ์ ์  : ๋งํฌ๋ฅผ ๊ณต์ ํด๋ ์ ํด์ง ํ์ ์์ด ์ด์ํ ํํ๋ก ๊ณต์  ํ๋ฉด์ด ๋์์ง
    - ํด๊ฒฐ : Og ํ๊ทธ๋ฅผ ํตํด์ ์ ํด์ง ๊น๋ํ ํ์์ผ๋ก ๊ณต์ ๋  ์ ์๋๋ก ์ค์ 

        ```html
        <meta property="og:title" content="์ธ๋ชจ๊ณต - ์ธ์์ ๋ชจ๋  ๊ณต๋ถ" />
        <meta property="og:url" content="http://semogong.site/" />
        <meta property="og:type" content="website" />
        <meta property="og:image" content="https://semogong-bucket.s3.ap-northeast-2.amazonaws.com/SEMOGONG.jpg" />
        <meta property="og:description" content="TIL ๊ณต์  ์ฌ์ดํธ, ์ค๋ ํ ๊ณต๋ถ๋ฅผ ๊ณต์ ํด๋ณด์ธ์!" />
        ```

### <220425>

> **Tag : version 1.0.6** <br>
**#32** `Enhancement` : Comment ์์ฑ ์๊ฐ ํ์ <br>
**#35** `Bug` `Back` : State ๋ณ๊ฒฝ ์ ์ค๋ฅ ๋ฐ์ <br>
**#36** `Front` : ๊ธ ๋ฐ ๋๊ธ ์ญ์  ์ Confirm <br>
>


- **#32**
    - ๊ธฐ์กด ๋ฌธ์ ์  : ๋๊ธ์ ์์ฑํ์ ๋, ํด๋น ๋๊ธ์ด ์ธ์  ๋ฌ๋ฆฐ ๊ฑด์ง ๊ตฌ๋ถ์ด ์๋  ๋๊ฐ ์์์. โ ๋๊ธ์ด ์ธ์  ๋ฌ๋ฆฐ ๊ฒ์ธ์ง ํ๊ธฐ ํ์
    - ํด๊ฒฐ : ๋๊ธ์ด ๋ฌ๋ฆฐ ์๊ฐ ํ๊ธฐ
        - ๋ก์ง : CommetViewDto์์ ๋๊ธ์ด ๋ฌ๋ฆฐ ์๊ฐ์ธ CreateDateTime๊ณผ ํ์ฌ ์๊ฐ์ธ LocalDateTime.now()์ ์ด์ฐจ์ด ๋ฅผ Duration.between์ ํตํด ๊ณ์ฐํ์ฌ ํด๋น DTO์ ์ ์ฅํ๊ณ  veiwTemplate์ ๋ณด๋ด์ค ํ, ViewTemplate(Thymeleaf) ์์์

            1. 60๋ถ์ด ์ง๋์ง ์์์ผ๋ฉด โx๋ถ ์ โ์ผ๋ก
            2. 60๋ถ์ด ์ง๋ฌ๊ณ  24์๊ฐ์ด์ ์ด๋ฉด  โx์๊ฐ ์ โ์ผ๋ก
            3. 24์๊ฐ์ด ์ง๋ฌ์ผ๋ฉด โx์ผ ์ โ์ผ๋ก ํ๊ธฐ

        - ์ด๋, Thmeleaf์์ Math.round๋ฅผ ์ ์ฉํ๊ธฐ ์ํด ThymeMath Component๋ฅผ ์์ฑํ์ฌ ์คํ๋ง ๋น์ผ๋ก ๋ฑ๋ก โ ๋น์ผ๋ก ๋ฑ๋ก๋ Object๋ Thymeleaf ์์ `@thymeMath` ํํ๋ก ์ฌ์ฉ ๊ฐ๋ฅ.
- **#35**
    - ๊ธฐ์กด ๋ฌธ์ ์  : State ๋ณ๊ฒฝ ์ค๋ฅ ์ ๋ณ ๋ค๋ฅธ ์ค๋ฅ ์ฒ๋ฆฌ ์์ด ๊ทธ๋ฅ ํ์ผ๋ก ๋์๊ฐ๊ฒ ์ค์  โ ๋ณ๊ฒฝ๋์๋์ง, ์ค๋ฅ๊ฐ ๋์ง ์ฌ์ฉ์๊ฐ ์ธ์งํ๊ธฐ ์ด๋ ค์
    - ํด๊ฒฐ : State ๋ณ๊ฒฝ ์ค๋ฅ ์ alert์ ํตํด ์๋ชป๋ state ๋ณ๊ฒฝ์ด๋ผ๊ณ  ์๋ ค์ค
        - ๋ก์ง : ViewTemplate(Thymeleaf) ์์ ํ์ฌ state ๊ฐ์ ๋ฐ์์จ ํ <br>
          ( Studying -> Studying / Breaking, End -> Breaking / End -> End ) ์ ๊ฐ์ ๋ณ๊ฒฝ ์ค๋ฅ์ ๋ํด์ if ์ ์ ํตํด alert์ ๋์ธ ์ ์๋๋ก ์ค์ 

- **#36**
    - ๊ธฐ์กด ๋ฌธ์ ์  :  ๊ธ ๋ฐ ๋๊ธ ์ญ์  ์ โ์ญ์ โ ๋ฒํผ์ ๋๋ฅด๋ฉด ๊ทธ๋ฅ ๋ฐ๋ก ์ญ์  ๋จ โ ์ฌ์ฉ์๊ฐ ์ค์๋ก ์ญ์ ๋ฅผ ๋๋ฅด๋ ๊ฒฝ์ฐ์ ๋ํ ์ฒ๋ฆฌ๊ฐ ํ์.
    - ํด๊ฒฐ :
        - ๋ก์ง : ๋ฐ๋ก a ํ๊ทธ๋ก ์ญ์  URL๋ก ์์ฒญ์ ๋ณด๋ด๋ ๊ฒ์ด ์๋, button์ผ๋ก ๋ฐ๊พผ ํ onclick์ผ๋ก javascript delete_check ํจ์๋ฅผ ํตํด confirmํ  ์ ์๋๋ก ์ค์ 

          detele_check : ์ธ์๋ก ์ญ์ ํ๊ณ ์ ํ๋ Entity์ id๋ฅผ ๋ฐ์์ค๊ณ  confrim ์งํ. confirm์์ ํ์ธ์ ๋๋ฅผ ์ location.href ๋ฅผ ํตํด delete ์์ฒญ์ ์งํ. ์ทจ์๋ฅผ ๋๋ฅผ ์ ์๋ฌด๋์ํ์ง ์๋๋ก ์ค์ 

### <220420>

> **Tag : version 1.0.5** <br>
#10 `Back` `Front`: State๋ง๋ค ๊ตฌ๋ถํ  ์ ์๋๋ก ๊ณ ์  ์ ๋ถ์ฌ ํ์ <br>
#11 `Back` : ๊ฐ Post ๋น comment ๊ฐ์๋ฅผ ์ธ์งํ  ์ ์๋๋ก ์ค์  ํ์ <br>
#16 `Back` : Side Widget์ All Members๋ฅผ StudyingโBreakingโEnd ์์ผ๋ก ์ ๋ ฌ ํ์ <br>
#18 `Front` : ํ๋ง ๊ธ๊ผด ์ค์  ํ์ <br>
#20 `Front` : ํต์ผ๋ Header ์์ฑ ํ์ <br>
#24 `Back` `Front` : ๊ฐ Post์ CreateDate ํ๊ธฐ ํ์ <br>
>


- **#10**
    - ๊ธฐ์กด์ ๋ฌธ์ ์  : State๋ง๋ค ๊ตฌ๋ถ์ด ์๋์ด ์์ด, ์๊ฐ์ ์ผ๋ก ํจ์จ์ ์ด์ง ๋ชปํจ. ์ฆ๊ฐ ์ธ์งํ๊ธฐ๊ฐ ์ด๋ ค์.
    - ํด๊ฒฐ : ๊ฐ State์ ๋ง๋ ์์ ๋ถ์ฌ (thymeleaf์ if ๋ฌธ๋ฒ์ ์ฌ์ฉ)
- **#11**
    - ๋ฌธ์ ์  : comment๋ฅผ ๋ฌ ์๋ ์์ง๋ง, ๊ฐ post์ comment๊ฐ ๋ฌ๋ฆฐ์ง ์๋ฌ๋ฆฐ์ง ์ธ๋ถ์์๋ ํ์ธ ๋ถ๊ฐ
    - ํด๊ฒฐ : ViewTemplate์๊ฒ DTO๋ฅผ ๋ณด๋ด์ค ๋, DTO์์ Comment Count๋ผ๋ ๋ณ์๋ฅผ ์ถ๊ฐํ์ฌ ๊ฐ post ๋น ๋ฌ๋ฆฐ comment์ ๊ฐ์๋ฅผ ํ์ถํด์ค. (๊ฐ post ์ฐ์๋จ์ ํํ)
- **#16**
    - ๊ธฐ์กด์๋ ์ด๋ฆ์์ผ๋ก ์ ๋ ฌํ๋ All members๋ฅผ Studying โ Breaking โ End ์์ผ๋ก ์ ๋ ฌํ์ฌ ํํ
    - ํด๊ฒฐ ๋ก์ง :
        - ์ผ๋จ ๋ชจ๋  ๋ฉค๋ฒ๋ฅผ DB์์ ๊ฐ์ ธ์จ ํ Memory ๋จ์์ ์ง์  ์ ๋ ฌ ์คํ. โ O(N)
        - ๋น ๋ฆฌ์คํธ๋ฅผ ์์ฑํ์ฌ
            1. ๋ชจ๋  ๋ฉค๋ฒ๋ฅผ ๋๋ฉด์ Studying์ธ ๋ฉค๋ฒ ์ถ๊ฐ
            2. ๊ทธ ํ ๋ ๋ชจ๋  ๋ฉค๋ฒ๋ฅผ ๋๋ฉด์ Breaking์ธ ๋ฉค๋ฒ ์ถ๊ฐ
            3. ๊ทธ ํ ๋ง์ง๋ง์ผ๋ก ๋ชจ๋  ๋ฉค๋ฒ๋ฅผ ๋๋ฉด์ End์ธ ๋ฉค๋ฒ ์ถ๊ฐ
- **#18**
    - Body์ font-family๋ฅผ NanumSquareRound ์ ์ฉ โ ๊ธฐ์กด๋ณด๋ค ํจ์ฌ ๋ณด๊ธฐ ์ข์์ง
- **#20**
    - ํต์ผ๋ header,footer๋ฅผ ๋ง๋  ํ ๋ชจ๋  html์ด fragment๋ก ๋ฐ์์ฌ ์ ์๋๋ก ์ค์ 
    - edit,create ํ๋ฉด ๊ฐ์ ๊ฒฝ์ฐ bootstarp ์ด์ธ์ css๋ฅผ ๋ฐ์ผ๋ฉด ์๋๋ฏ๋ก addheader๋ฅผ ์ถ๊ฐ๋ก ์์ฑํ์ฌ home.html์ด fragment๋ก ๋ฐ๋๋ก ์ค์ 
- **#24**
    - Post Entity์ FormatCreateDate๋ฅผ ์ถ๊ฐํ์ฌ LocalDateTime class๊ฐ ์๋ String์ผ๋ก DB์ ํด๋น ๋ ์ง๋ฅผ ์ํ๋ format ํ์์ผ๋ก ์ ์ฅํ  ์ ์๋๋ก ์ค์  (โApril 14, 2022โ ํ์์ผ๋ก ์ ์ฅ)
    - ๊ทธ ํ DTO์๋ FormatCreateDate๋ฅผ ์ถ๊ฐํ์ฌ ViewTemplate์ ๊ฐ์ ์ ๋ฌํ  ์ ์๋๋ก ์ค์ 
    - ๊ฐ post ์ข์๋จ์ ํ๊ธฐ


### <220418>

> **#14** `Back` : Member๊ฐ ์ด๋ฏธ ๊ทธ๋  ๊ณต๋ถ๋ฅผ ์ข๋ฃํ์ง๋ง, ๋ค์ ๊ณต๋ถ๋ฅผ ์์ํ  ๊ฒฝ์ฐ์ ๋ํ ์กฐ์น <br>
> **#21** `Front` : ์๋ก์ด Member ๊ฐ์๊ณผ ๊ธฐ์กด Member ์ ๋ณด ์์  html์ด ๋ถ๋ฆฌ๋์ง ์์ <br>
> **#7** `Bug` : ๋ก๊ทธ์ธ์ด ์๋ ์ํ์์ post์ ๋๊ธ์ ๋ฌ ๊ฒฝ์ฐ ์ค๋ฅ ๋ฐ์ <br>
> **#6** `Back` : Posting ์ญ์  ๊ธฐ๋ฅ ๋ฐ ์ญ์  ๋ฒํผ ์์ฑ <br>
> **#4** `Front` : ์ฌ๋ฌ ์๋ ฅ placeholder ์์  <br>



- **#14**
    - ๊ธฐ์กด์ ๋ฌธ์ ์ 
        
        Member๊ฐ ๊ณต๋ถ๋ฅผ ์ข๋ฃํ ํ ๋ค์ ๊ณต๋ถ๋ฅผ ์์ํ๊ณ  ์ถ์ ๋ ๊ณต๋ถ ๊ธฐ๋ก์ ์งํํ๋ ๊ธ์ด ๋ถ๋ฌ์์ง๋ ๊ฒ์ด ์๋, ์๋ก์ด ๊ธ์ ์์ฑ๊ฒ๋ ์ค์ ๋์ด ์์์. ์ฆ, **๊ณต๋ถ์ข๋ฃ๋ฅผ ํ์ง๋ง, ๋ค์ ๊ณต๋ถ๋ฅผ ํ๋ ๊ฒฝ์ฐ์ ๋ํ ์์ธ์ฒ๋ฆฌ**๊ฐ ํ์ํ์
        
    - ํด๊ฒฐ ๋ก์ง
        
        > ์ผ๋จ ๊ธฐ๋ณธ์ ์ผ๋ก 04์๋ฅผ ํ๋ฃจ์ ์์๊ณผ ๋์ผ๋ก ์ค์  (์๋ฒฝ๋ฐ ๊ณ ๋ ค) <br>
        ์ฌ์ฉ์๊ฐ End -> Studying ์ผ๋ก State๋ฅผ ๋ณ๊ฒฝํ  ์ (๊ณต๋ถ์๋ฃ -> ๊ณต๋ถ์์)์ ์ํฉ
        >
  
        1. ์ฌ์ฉ์๊ฐ ํ์ฌ ์ด ๊ธ์ด ์กด์ฌํ๋ ์ง ํ๋จ (์ฆ, ์ ๊ท ํ์์ธ์ง ์๋์ง ํ๋จ)
            - true (์ด ๊ธ์ด ์กด์ฌ)
                1. ์ฌ์ฉ์๊ฐ ๊ฐ์ฅ ์ต๊ทผ์ ์์ฑํ ๊ธ์ด ํด๋น ๋ ์ง์ ์์ฑํ ๊ธ์ด๋ฉด์ ์์ฑ ํ ์๊ฐ์ด 04์ ์ดํ์ธ์ง ํ๋จ
                    - true => ํด๋น ๊ธ์ ๊ฐ์ ธ์์ ์ด์ด ์์ฑํ  ์ ์๋๋ก ์ค์ 
                    - false
                        1. ํ์ฌ State ๋ณ๊ฒฝ ์์ ์ด 04์ ์ด์ ์ธ์ง ํ๋จ
                            - true
                                1. ๊ทธ ์ ๋  ์์ฑํ ๊ธ์ด ์กด์ฌํ๋ ์ง, ํด๋น ๊ธ์ด 04์ ์ดํ ์์ฑํ ๊ธ์ธ์ง ํ๋จ
                                    - true => ํด๋น ๊ธ์ ๊ฐ์ ธ์์ ์ด์ด ์์ฑํ  ์ ์๋๋ก ์ค์ 
                                    - flase => ์๋ก์ด ๊ธ ์์ฑ
                            - false => ์๋ก์ด ๊ธ ์์ฑ
            - false => ์๋ก์ด ๊ธ ์์ฑ
- **#21**
    - ์ถํ์ bug๊ฐ ๋ฐ์ํ  ๊ฒ์ ๋๋นํ์ฌ sign up ํ๋ฉด ๊ณผ eidt ํ๋ฉด์ ๋ถ๋ฆฌ
    - createMemberForm.html โ createMemberForm.html + editMemberForm.html
- **#7**
    - ๋ก๊ทธ์ธ์ด ์๋ ์ ์ ๊ฐ Post์ ๋๊ธ์ ๋ฌ ๊ฒฝ์ฐ, alert (โ๋ก๊ทธ์ธ ํ ๋๊ธ์ ๋ค์ค ์ ์์ต๋๋ค.โ) ์ฐฝ์ ๋์ฐ๋๋ก ์ค์ 
- **#6**
    - ์์์ ์ผ๋ก DeleteMapping์ผ๋ก Controller ์ ์ค์ ํ ๊ฒ์ด ์๋ GetMapping์ผ๋ก ์ค์ ํด๋ . โ ์ถํ DeleteMapping๊ณผ API๋ฅผ ์ฌ์ฉํ์ฌ ํด๊ฒฐ ์์ 
    - edit ํ๋ฉด์ post ์ญ์  ๋ฒํผ ์์ฑ
- **#4**
    - placeholder ์์  (๋์๋ฅผ ์๋ ฅํ์ธ์. ๋ฑ ๊ด๋ จ์๋ ๊ฐ๋ค์ด default๊ฐ์ผ๋ก ์ฃผ์ด ์ก์์)