---
layout: page
title: TID
permalink: /tid/
image:
---

<aside>
🏋️ <strong>Today I Developed</strong> <br>
매일 매일 제가 개발한 흔적을 남기는 공간입니다!
</aside>

<br>

## Project(SEMOGONG) 개발 일지

- Issue 를 바탕으로 개발 진행.
- Issue Labels
    - `Bug` : 구현한 기능들 중에서 서버에 올렸는데 작동하지 않거나 오류가 생김
    - `Back` : Back-End Engineer 만을 위한 이슈
    - `Front` : Front-End Engineer 만을 위한 이슈
    - `Enhancement` : 새로운 기능이 필요한 경우, front back 모두가 손봐야할 이슈

<hr>

### <220818>

> #99 `Bug` : 0-4 시 시작 시 시작시간 오류
>

- #99
    - 기존의 문제점
        - 0-4시 공부 시작 시, 로직에 따라 날짜는 전날로 설정되는데, 그때 공부 시작 시간 마저 createTime으로 설정되는 오류
        - Post Entity 생성 메서드 createPost 의 시간 List에 시간을 추가하는 times.add 가 createTime 으로 설정되어 있어 발생하는 오류
    - 해결
        - createPost의 times.add 를 현재 시간(LocalDateTime.now())으로 설정

            ```java
            public static Post createPost(Member member, LocalDateTime createTime){
                Post post = new Post();
                post.setMember(member);
                post.createTime = createTime;
                DateTimeFormatter timeFormatter = DateTimeFormatter.ofPattern("HH:mm");
                post.times.add(LocalDateTime.now().format(timeFormatter));
                post.state = StudyState.STUDYING;
                return post;
            }
            ```


### <220815>

> #96 `Back` : 공부 시작 시간이 0~4시 사이일 경우에 대한 처리 [시간 처리]
>

- #96
    - 기존의 문제점
        - 만약 사용자가 0~4시 사이에 공부를 시작할 경우, 해당 post는 비지니스 로직 특성 상 해당 날짜를 가지게 됨. (`LocalDateTime.now()` 이용)
        - 하지만, 서비스 특성 상 0~4시 사이의 시간은 이전 날짜로 인식되게 해야됨. (그렇지 않으면 해당 날짜의 Post가 2개가 되는 예외 발생)
        - ex) 13일이 넘어가는 1시에 공부를 시작했다면 14일로 인식 → 13일로 인식되게 설정 필요
    - 구현
        - 0 - 4 시 사이에 글을 작성할 경우, 전 날의 23:59 에 글을 작성한 것과 동일하게 취급하여 구현 → 기존의 로직에 방해되지 않으면서 현재 필요한 요구사항 충족 가능
        - 즉, post의 times에는 현재시간을 넣어주고, post의 createTime은 그전날의 23:59 로 설정

            ```java
            if (createDate.getHour() < 4) { // 0 - 4 시 사이에 글을 작성할 경우, 전 날의 23:59 에 글을 작성한 것과 동일하게 취급.
                LocalDateTime toChange = createDate.minusDays(1); // 전날로 설정
                createDate = LocalDateTime.of(toChange.getYear(), toChange.getMonthValue(), toChange.getDayOfMonth(), 23, 59, 59); // 전날 23시59분으로 설정
            }
            Long postId = postService.save(loginMember.getId(), createDate); // 저장할 때는 해당 글의 작성자 Member 연결, creatTime만 설정해줌
            ```


### <220814>

> #95 `Back` : 랭크 측정 기법 변경
>

- #95
    - 기능 필요성
        - 현재 랭크 측정 기준은 [어제 공부 시간을 기준으로 하루 목표 달성량 + 일주일 총 공부시간을 기준으로 일주일 목표 달성량 + 한달 출석률]
        - 하지만, ‘어제 공부 시간을 기준으로 하루 목표 달성량’ 은 변동량이 너무 크기 때문에 어제 하루의 공부량이 랭크에 큰 영향을 미침. → 랭크 변동률이 커짐
        - 고로 변동률을 줄일 필요가 존재
    - 구현
        - '어제 공부 시간을 기준으로 하루 목표 달성량’ → '일주일 기준 하루 평균 공부시간으로 하루 목표 달성량’ 으로 변경 (해당 사항은 추후 변동 가능)

            ```java
            Times weekAvgTimes = new Times();
            int weekAvgTime = weekAllTimes / 7;
            int dayGoalTimes = member.getGoal().getDayGoalTimes();
            allStatic.setGoalAttainmentToday(Math.round(((float) weekAvgTime / (float) dayGoalTimes) * 100));
            ```


### <220812>

> #93 `Bug` : 랭킹 페이지 및 My Page Times 계산 오류
>

- #93
    - 발생 오류
        - Ranking Page 이든 My Page 이든 공부 시간 계산 시 시간이 마무리되지 않은 Post에 대해선 Times가 Null 반환 → `NullPointerException` 발생
        - Post의 times size 가 홀수 일경우 그냥 null이 반환되도록 설정되어 있었음 → 처리 필요
    - 해결
        - times size 가 짝수라면 계산을 돌리고
        - 홀수 (마무리되지 않은 시간) 에 대해서는 null 이 아닌 0을 반환하여 `NullPointerException` 을 피할 수 있도록 설정

            ```java
            private Times getTimes(List<String> times) {
                Times resultTime;
                if (times.size() % 2 == 0) { // 짝수
            					int total1 = 0;
            					...
                    }
                    resultTime = new Times(total1);
                } else { // 홀수 -> 마무리되지 않은 post
                    resultTime = new Times(0);
                }
                return resultTime;
            }
            ```

### <220811>

> #88 `Bug` : My Page 에서의 post edit 오류 <br>
#89 `Bug` : My Page, Member Detail 에서의 출석률 계산 오류
>

- #88
    - 발생 오류
        - My Page 에서 글 수정 submit이 안되는 오류
        - focusedPostId 를 찾을 수 없어서 발생한 오류
        - 뿐만 아니라 이전에 home에서의 post edit 시 발생하는 오류와 동일한 오류 발생
        - 원인 : focusedPostId 변수이름 변경으로 발생한 오류 + 중복되는 id로 인한 오류 → 전체적인 post edit의 변경사항을 반영하지 않음
    - 해결
        - Home에서의 오류해결과 동일하게 My Page의 렌더링되는 html에 각 모든 post에 연결되는 simplemde + {post.id} 변수 생성 후 해당 변수에 새로 생성되는 Simple MDE 할당
        - focusedPostId 변수 이름 수정
    - 추후 필요 수정 사항
        - **[중요]** focusedPostId는 해당 비지니스 로직에서 사용되지 않음 → main.js 가 아닌 새로운 javascript를 생성하여 분리할 필요가 있음.
- #89
    - 발생 오류
        - 한달 공부 통계에서 출석률, 전체 공부시간, 평균 공부시간 등이 수정한 부분과 다르게 '오늘'까지 고려가 됨. ( → 원래는 '어제' 까지만 고려가 되어 계산되어야 함)
        - 계산 부분에 있어서 오류 발생 → 계산 해놓고 다른 변수로 적용하고 있었음
    - 해결
        - 공부 랭킹을 위해서 현재 달의 출석률을 구해주기 위해서 focusedDay, nowMonthDayLen 을 사용했지만, 현재 보고 있는 달력의 통계를 보여주기 위해 monthDate, mothPosts 를 사용했음. 여기서 만약 현재 보는 달력이 오늘 날짜의 달과 동일하다면 monthDate에 focusedDay 를 대입 하고 monthPosts에 어제날짜까지의 post를 넣어주어 해결
        - 즉, 보고 있는 달력의 통계를 상황에 따라 현재 달에 맞게 설정해주며 해결

        ```java
        List<Post> nowMonthPosts = postService.getMonthPosts(memberId, LocalDateTime.now().getMonthValue());
        int focusedDay;
        if (LocalDateTime.now().getHour() < 4) {
            focusedDay = LocalDateTime.now().getDayOfMonth() - 2;
        } else {
            focusedDay = LocalDateTime.now().getDayOfMonth() - 1;
        }
        List<Post> calculatedNowMonthPosts = new ArrayList<>();
        for (Post post : nowMonthPosts) {
            if (post.getCreateTime().getDayOfMonth() > focusedDay) {
                break;
            }
            calculatedNowMonthPosts.add(post);
        }
        
        ForCalender CalenderInfo = getCalenderInfo(weekDay, focusedDate);
        int monthDate = dayData[month - 1];
        if (month == LocalDateTime.now().getMonthValue()) {
            monthDate = focusedDay;
            monthPosts = new ArrayList<>(calculatedNowMonthPosts);
        }
        AllStatic allStatic = getAllStatus(oriMember, staticsData, days.get(days.size() - 1), monthPosts, monthDate, focusedDay, calculatedNowMonthPosts.size());
        ```

        - 먼저 focosedDay를 구해고 해당 변수에 맞는 현재 달의 posts를 가져옴
        - 그 후 만약 사용자가 확인하고 있는 달력이 현재 달과 동일하다면 해당 변수들을 현재 달에 사용되는 변수로 변경. (아니라면 기존의 변수 사용)

### <220809>

> #90 `Bug` : 새 글 작성 오류
>

- #90
    - 발생 오류
        - 새 글 작성 시 simpleMDE 가 content에 생성되지 않음
        - submit 시 제출되지 않음
        - 원인 : post edit 과 fragment를 공유하고 있기 때문에, 이전에 post edit 오류 수정 시 id 값을 변경하는 부분이 있었는데, 그 부분을 post new(새 글 작성)의 javascript 코드에 반영하지 않았기 때문에 발생
    - 해결
        - 기존의 content 라는 id를 가졌던 요소에 `content + ${post.id}` 의 id를 새로 부여 → 이제 해당 id로 Simple MDE 를 심어줄 수 있음
        - `simplemde + ${post.id}` 의 변수를 생성해주고 해당 변수에 새로 생성된 Simple MDE 를 할당 → 해당 MDE의 value(content) 를 가져올 수 있게 됨

<aside>
⚠️ <strong>[오늘의 깨달음] Fragment 사용</strong> <br>
무조건 fragment를 사용한다고 편한 것이 아님. fragment를 난무하지 말고, 개별 비지니스 로직이 좀 많은 부분에서 다르다 하면 사용하지 말고 (유지 보수가 힘듦), 정말 공통적인 요소에만 fragment를 사용하는 것이 좋음

</aside>

### <220808>

> #87 `Bug` : 다중 post edit 시 여러 오류 발생
>

- #87
    - 기존 문제점
        1. 가장 마지막에 edit버튼을 누른 post의 content로 모든 post의 내용이 변경됨 (simpleMDE를 **전역변수로 공유**하며 사용해서 발생한 문제)
        2. simpleMDE가 다른 post의 content에 생성됨 (postEditForm의 content가 **unique id가 아니면서 생기는 문제**. → edit하면 여러개의 content가 생성되고, 중복된 ID로 인해서 가장 처음 생성된 content에 simpleMDE가 발생)
        3. submit 하면 이전에 열었던 post에 fragment가 replacement되고 현재 post는 replacement되지 않음 (postEditForm의 postEdit_container의 id가 **unique하지 않아 발생하는 문제**, 2번과 동일한 이유로 발생하는 문제)
    - 해결
        1. simpleMDE 변수 하나를 전역변수로 공유하며 사용해서 발생한 문제
            - simpleMDE를 각 post의 ID로 생성한 후 (전역변수로 선언 → 다른 함수에서도 사용해야 되기 때문)
            - edit 시 해당 post의 ID 값을 가지는 simpleMDE를 가져와 해당 변수에 새로운 simpleMDE를 생성해줌. → unique ID를 가지고 있기 때문에, 각 post의 simpleMDE의 값을 구분하여 저장할 수 있게 됨
            - 구현 코드

                ```jsx
                // home.html의 script 부분 (변경 전: var simplemde;)
                [# th:each="post, stat : ${postDtos}"]
                var simplemde[[${post.id}]];
                [/]
                ```

                - home.html의 simplemde를 각 post에 mapping 될 수 있도록 id에 맞게 simplemde를 생성
                - thymeleaf의 javascirpt에서의 유용한 기능을 이용하여 **동적 변수 생성**

                ```jsx
                function edit_post(id) {
                    $.ajax({
                        url: '/posts/edit/' + id,
                        type: "GET",
                    })
                        .done(function (fragment) {
                            $('#postModal_content' + id).replaceWith(fragment);
                						/* (변경 전) => simplemde = new SimpleMDE({element: document.getElementById("content"+id.toString()),spellChecker: false});*/
                            eval("simplemde" + id + "= new SimpleMDE({element: document.getElementById("content"+id.toString()),spellChecker: false})");
                            ...
                        });
                }
                ```

                - edit 요청이 들어올 때 동적으로 해당 post에 mapping 되는 simplemde를 재정의 → 해당 post에 simpleMDE를 content에 심어주는 것
                - 이때 id에 맞는 **동적 변수**를 통해 전역 변수인 simpleMDE를 가져와야 하기 떄문에 `eval()` 함수 사용
        2. postEditForm의 content가 unique id가 아니면서 생기는 문제. → edit하면 여러개의 content가 생성되고, 중복된 ID로 인해서 가장 처음 생성된 content에 simpleMDE가 발생
            - postEditForm에 content id속성을 post의 ID를 연결해주어 unique한 id가 될 수 있도록 설정.

                ```jsx
                <textarea type="text" rows="4" 
                					th:field="*{content}" th:id="'content'+${postForm.id}" 
                					placeholder="Content" class="form-control"
                          autocomplete="off"></textarea>
                ```

            - 추가로 그에 맞게 simpleMDE를 심어줄 때도 `content+post.id` 로 찾아오고 심을 수 있도록 설정 → unique id이기 때문에 겹칠일도 없으며, 제대로된 위치에 생성 가능

              `new SimpleMDE({element: document.getElementById(\"content\"+id.toString()),spellChecker: false})`

        3. submit 하면 이전에 열었던 post에 fragment가 replacement되고 현재 post는 replacement되지 않음 (postEditForm의 postEdit_container의 id가 unique하지 않아 발생하는 문제, 2번과 동일한 이유로 발생하는 문제)
            - eidt 요청 시 postEditForm의 postEdit_container를 찾아와 교체해 준 후
            - 해당 postEdit_container id값을 `postEdit_container + post.id` 로 가질 수 있도록 설정 → 미리 id값으로 렌더링 후 찾으면 찾아와지지 않기 때문에 가져온 후 id 값을 변경
            - 코드

                ```jsx
                function edit_post(id) {
                    $.ajax({
                        url: '/posts/edit/' + id,
                        type: "GET",
                    })
                        .done(function (fragment) {
                            $('#postModal_content' + id).replaceWith(fragment);
                            ...
                            document.getElementById("postEdit_container").setAttribute("id", "postEdit_container" + id.toString());
                            document.getElementById("postEdit_container"+id.toString()).scrollIntoView();
                        });
                }
                ```

            - 마지막으로 post 시 `postEdit_container + post.id` 로 결과 fragment로 replace. → unique ID를 가지고 있기 때문에 제대로된 위치로 replace 됨

### <220807>

> #83 `Bug` : 시간 변경 오류
>

- #83
    - 기존 문제점
        - 오늘이 아닌 이전 post에서 시간 수정 시 오류 발생
        - 오늘 작성 중인 Post가 아닌데, **‘현재시간을 넘는 시간이 발생했다’ 는 validation** 작동
        - 또한 00시, 01시 등 0~4시에 대한 시간계산 오류 → 23시보다 00시, 01시가 더 큰 값인데, 단순하게 23>0 이므로 0~4시가 더 작은 시간으로 판별되는 오류
    - 해결
        - 0~4 시에 대한 추가적인 처리
            - 하루 시간의 기준을 4시로 잡있기 때문에 0~4시에 대해선 24를 더 추가해주어야 함
            - `if (hour < 4) {hour+=24}` 추가하여 전체적인 시간 계산이 정상적으로 이루어질 수 있도록 수정
        - 오늘 post에 대해서만 시간확인(1) 검증 실행
            - 이전 post들에 대해선 시간확인(1-현재시간과 비교)을 진행해선 안됨
            - `if (id == focusedPostId) {...}` 를 통해 오늘 post에 대해서만 해당 검증 진행

### <220806>

> #82 `Enhancement` : 시간양식 check
>

- #82
    - 기능의 필요성
        - 현재 시간을 제출할 때, 시간의 개수, 빈 시간인지 아닌지 등의 Validation만 거치고 있음
        - 하지만, 시간 계산을 위한 시간양식에서 벗어난 입력값에 대해선 처리가 되지 않고 있음
        - 시간 계산을 위해선 올바른 시간양식으로 입력할 수 있도록 유도 필요
    - 구현
        1. 형식 체크
            - 지정 형식 : (시간형식: ex_ ‘`17:20`’) `var reg = /^\d{1,2}:\d{1,2}$/;`
            - 정규표현식을 이용하여 형식 확인, 형식에서 벗어날 경우 alert 및 class 변경

            ```java
            if (!reg.test(times[idx].value)) {
                alert('잘못된 형식의 시간이 입력되었습니다.\n올바른 형식: "시간:분" ex) 17:20');
                var temp_times_id = "times" + idx;
                var timeHtml = document.getElementById(temp_times_id);
                timeHtml.setAttribute('class', 'form-control is-invalid');
                timeHtml.setAttribute('style', 'background-image: none; padding: 6px;');
                document.getElementById("postEdit_container").scrollIntoView();
                return;
            }
            ```

        2. 시간 확인 - 1
            - 입력된 시간 값이 현재 시간보다 큰지 확인
            - 크면 오류, alert을 통해 수정할 수 있도록 설정

            ```java
            var curr = times[idx].value.split(':')
            var currTime = parseInt(curr[0])*60 + parseInt(curr[1])
            if (currTime > totalTimeToday) {
                alert("현재시간을 초과한 시간값이 있습니다.");
                var curr_times_id = "times" + idx;
                var timeHtmlCurr = document.getElementById(curr_times_id);
                timeHtmlCurr.setAttribute('class', 'form-control is-invalid');
                timeHtmlCurr.setAttribute('style', 'background-image: none; padding: 6px;');
                document.getElementById("postEdit_container").scrollIntoView();
                return;
            }
            ```

        3. 시간 확인 -2
            - 이전 시간값이 이후 시간값보다 큰지 확인
            - 크면 오류, alert을 통해 수정할 수 있도록 설정

            ```java
            if (idx !== 0){
                var before = times[idx-1].value.split(':');
                var beforeTime = parseInt(before[0])*60 + parseInt(before[1]);
                if (beforeTime > currTime) {
                    alert("이후 시간이 이전시간보다 작습니다. 수정해주세요!");
                    var timeAfter = "times" + idx;
                    var timeBefore = "times" + (idx-1);
                    var timeHtmlAfter = document.getElementById(timeAfter);
                    var timeHtmlBefore = document.getElementById(timeBefore);
                    timeHtmlAfter.setAttribute('class', 'form-control is-invalid');
                    timeHtmlAfter.setAttribute('style', 'background-image: none; padding: 6px;');
                    timeHtmlBefore.setAttribute('class', 'form-control is-invalid');
                    timeHtmlBefore.setAttribute('style', 'background-image: none; padding: 6px;');
                    document.getElementById("postEdit_container").scrollIntoView();
                    return;
                }
            }
            ```


### <220804>

> #80 `Back` : 랭크 측정 기준 변경 <br>
#81 `Bug` : post edit 오류 <br>
#79 `Bug` : paging 오류
>

- #80
    - 기존의 문제점
        - 랭크 측정 기준 항목들의 기준이 일치하지 않았음 → 이번달 출석률(오늘까지 포함), 하루 목표 달성률(어제), 주간 목표 달성률(어제까지)
        - 변경 필요 : 모두 어제까지를 기준으로 측정. -> 이번달 출석률을 오늘이 아닌 어제까지만 포함하도록
    - 해결
        - 이번달 출석률에 포함되는 범위를 오늘에서 어제까지만 포함
        - DB 자체의 조회를 통해서도 할 수 있지만, 이미 오늘까지의 데이터를 불러오기에, 불러온 데이터를 서버 자체에서 판단하도록 실행

            ```java
            int focusedDay;
            int nowMonthPostsLen = 0;
            if (LocalDateTime.now().getHour() < 4) {
                focusedDay = LocalDateTime.now().getDayOfMonth() - 2;
            } else {
                focusedDay = LocalDateTime.now().getDayOfMonth() - 1;
            }
            for (Post post : nowMonthPosts) {
                if (post.getCreateTime().getDayOfMonth() > focusedDay) {
                    break;
                }
                nowMonthPostsLen = nowMonthPostsLen + 1;
            }
            ```

            - 요청시간에 따라 범위를 1에서2로 줄임 (어제까지의 날짜)
            - 또한 범위의 끝이 되는 focusedDay 이후의 post는 포함하지 않음
            - 이렇게 해서 범위를 해당 날짜(어제)까지로 설정
- #81
    - 기존의 문제점
        - edit 시 javascript(ajax)에서 post 후 받아오는 fragment로 replace를 하지 못하는 오류 발생
        - fragment 중 postModal id part를 찾아오지 못함 (id를 `th:id=”’postModal’+${post.id}”` 로 설정하여 발생한 오류)
    - 해결
        - `th:id=”’postModal’+${post.id}”` → id=”postModal” 로변경
        - [중요] 그 후 replace를 통해 요소를 바꿔주고 해당 부분의 id를 새로 `“postModal+post.id"` 로 부여함

    <aside>
    ⚠️ <strong>[오늘의 발견] Controller에서 Fragment 사용 시 주의점!</strong> <br>
    Controller에서 fragment를 id나 요소로 찾을 때는 렌더링 전의 값으로 찾기 때문에 위와같이 id를 동적으로 설정하면 fragment로 찾아낼 수가 없음 → 렌더링은 찾아 낸 후 진행되는 것!!

    </aside>

- #79
    - 기존의 문제점
        - page 버튼이 제대로 동작하지 않음 (누르면 아무 post도 뜨지 않는 오류 발생)
        - page 버튼이 가지는 a 요소의 href의 requestParam 이름이 잘못 설정됨 (content값)
    - 해결
        - page의 queryParameter 수정 (queryParam : content -> inputContent)
        - `th:href="@{/ (page=${i}, focus=${nav2}, selected=${searchForm.selected}, **content**=${searchForm.inputContent})}"` → `th:href="@{/ (page=${i}, focus=${nav2}, selected=${searchForm.selected}, **inputContent**=${searchForm.inputContent})}"`

### <220803>

> #29 `Back` : comment 수정 기능
>

- #29
    - 기능 필요성
        - member 정보를 수정, post를 수정 하는 등 기존의 모든 Entity는 수정기능이 있었지만, comment는 작성 및 삭제 기능밖에 없었음
        - comment도 잘못작성할 경우 수정 기능을 통해 변경이 가능해야 됨. (이전에는 수정을 원한다면 삭제 후 다시 작성)
    - 구현
        - Controller
            - 댓글 수정 Form 요청

                ```java
                // 댓글 수정 get
                @GetMapping("/comment/api/edit/{id}/{commentId}")
                public String editForm(@PathVariable("id") Long postId, @PathVariable("commentId") Long commentId, @SessionAttribute(name = SessionConst.LOGIN_MEMBER, required = false) Long loginMemberId,Model model) {
                    Post post = postService.findOne(postId);
                    PostViewDto postViewDto = new PostViewDto(post);
                    Comment comment = commentService.findOne(commentId);
                    CommentViewDto commentDto = new CommentViewDto(comment);
                    model.addAttribute("post", postViewDto);
                    model.addAttribute("check", true);
                    model.addAttribute("comment", commentDto);
                    return "components/commentList :: #comment";
                }
                ```

                - 기존의 내용을 보여주기 위해 해당 id를 가진 comment를 가져와 model에 넣어줌
                - post안의 comment를 보여주는 것이기 때문에 해당 comment를 가지고 있는 post도 model에 담아줌
            - 댓글 수정

                ```java
                @PostMapping("/comment/api/edit/{id}/{commentId}")
                public String edit(@PathVariable("id") Long postId, @PathVariable("commentId") Long commentId, @SessionAttribute(name = SessionConst.LOGIN_MEMBER, required = false) Long loginMemberId, Model model, @RequestParam Map<String, Object> paramMap) {
                    Member member = memberService.findOne(loginMemberId);
                    Post post = postService.findOne(postId);
                    commentService.edit(commentId, paramMap.get("comment").toString());
                    PostViewDto postViewDto = new PostViewDto(post);
                    MemberDto memberDto = new MemberDto(member);
                
                    model.addAttribute("post", postViewDto);
                    model.addAttribute("check", true);
                    model.addAttribute("member", memberDto);
                    return "components/commentList :: #commentPart";
                }
                ```

                - 기존의 새로운 댓글 작성과 비슷한 원리로 진행 (해당 post의 comment들을 새로 불러와 redering 후 해당 commentPart를 replace하는 것)
                - 하지만 다른 점은 save가 아닌 edit
                - `commentService.edit()`

                    ```java
                    public void edit(Long id, String content) {
                        Comment comment = em.find(Comment.class, id);
                        comment.editContent(content);
                    }
                    ```

                    - 해당 comment를 가져와 content의 내용을 수정해줌 (새로 저장) → `editContent` : `this.content = content;`
                    - 중요한 점은 이렇게 수정 후 새로 post에 연결할 필요가 없음. 이미 연결되어 있기에.

### <220802>

> #3 `Back` : post 검색 (필터를 통해 -> 조건[제목,이름,희망직무,내용]) <br>
#76 `Back` : member 검색 (필터를 통해 -> 조건[이름, 희망직무]) <br>
#8 `Enhancement` : error page 생성 (error.html, error/4xx.html, error/5xx.html) <br>
#77 `Bug` : edit 시 search bar에 simpleMDE가 생기는 오류
>

- #3, #76
    - 기능 필요성
        - 현재까지는 그냥 해당 메뉴(전체글, 나의글, 오늘의글 등)에 맞는 모든 글들을 보여줌
        - 하지만, 검색을 통해 원하는 글을 필터링하여 보는 기능은 당연히 필요
        - 여기서 검색 시, 조건을 달아 세부적으로 검색할 수 있도록 하는 것이 필요
        - 추가로 멤버 검색도 필요
    - 해결 및 구현
        - search bar를 만들고 해당 search bar 좌측에 조건[제목,이름,희망직무,내용]을 선택하여 검색할 수 있도록 설정
        - 해당 form에서 사용될 Object(`SearchForm`)를 HomeController에서 보내준 후 해당 SearchForm의 `selected`(조건), `inputContent`(검색내용) 을 받아와 검색 진행
        - `@ModelAttribute(name = "searchForm") SearchForm searchForm`
        - ReqeustMapping으로 진행, GetMapping에서는 ModelAttribute의 기능을 통해 serachForm을 새로 만들어 model에 포함시켜주어 넘기며, PostMapping 때는 ModelAttribute를 통해 selected와 inputContent의 입력된 값을 받아옴 → GET과 POST를 분리하지 않았기에 추가적인 코딩이 필요했음 (searchForm이 초기화가 되지 않은 상태 판단 (`if (searchForm.getSelected() == null)`) → GET, searchForm이 초기화 → POST)
        - 이를 이용하여 form html 작성

            ```html
            <form id="search-form" class="d-flex px-1" th:object="${searchForm}">
                <input hidden id="focus" name="focus" th:value="${nav2}"/>
                <select th:field="*{selected}" class="form-select form-select-sm" aria-label=".form-select-lg example" style="width: 35%">
                    <option th:if="${nav2 != 'all-members'}" selected value="title">제목</option>
                    <option th:if="${nav2 != 'my-posts'}" value="writer">이름</option>
                    <option th:if="${nav2 != 'my-posts'}" value="desiredJob">희망직무</option>
                    <option th:if="${nav2 != 'all-members'}" value="content">내용</option>
                </select>
                <input type="search" th:field="*{inputContent}" class="form-control" th:placeholder="'['+ ${nav2} + ']' + ' 에서 검색'"/>
                <button type="submit" class="btn btn-secondary">
                    <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" class="bi bi-search" viewBox="0 0 16 16">
                        <path d="M11.742 10.344a6.5 6.5 0 1 0-1.397 1.398h-.001c.03.04.062.078.098.115l3.85 3.85a1 1 0 0 0 1.415-1.414l-3.85-3.85a1.007 1.007 0 0 0-.115-.1zM12 6.5a5.5 5.5 0 1 1-11 0 5.5 5.5 0 0 1 11 0z"/>
                    </svg>
                </button>
            </form>
            ```

            - selected 와 inputContent를 th:field를 통해 연결
            - [중요] 추가로 Mapping된 Controller는 focus request param값을 요구하기 때문에 현재 focus를 담아 같이 form으로 전해줌
        - 중요한 점은 각 메뉴에 맞게 검색이 진행되며, 페이징까지 적용이 되어야 한다는 점
            - all, today, my-posts, members 의 메뉴에 맞게 검색이 진행되어야 함
            - all,today 에서는 모든 조건(필터) 사용 가능
            - my-posts 에서는 이미 이름과 희망직무는 정해져있기 때문에 [제목, 내용] 필터만 적용 가능
            - members 에서는 글이 아니므로 [이름, 희망직무]로만 검색 가능. (페이징도 적용X)
            - 이에 Controller, Service에서도 현재 메뉴와 들어오는 필터 조건, 검색내용에 따라 다른 쿼리 적용 [분기 순서 : menu → selected→ searchContent]
                - 또한 query 적용 시 like을 사용하고 해당 검색내용을 포함하는 모든 post를 가져와야 하기 때문에 ‘%’+검색내용+’%’ 형식 적용 → `String searchKeyword = '%' + content + '%';`
                - today : `posts = postService.findByTodaySearch(...)`
                    - selected[title,writer,desriedJob,content] → searchContent

                        ```java
                        if (selected.equals("title")) {
                            return postNativeRepository.findByTitleToday(searchKeyword, date, offset);
                        } else if (selected.equals("writer")) {
                            return postNativeRepository.findByWriterToday(searchKeyword, date, offset);
                        } else if (selected.equals("desiredJob")) {
                            return postNativeRepository.findByJobToday(searchKeyword, date, offset);
                        } else {
                            return postNativeRepository.findByContentToday(searchKeyword, date, offset);
                        }
                        ```

                    - today같은 경우 날짜가 들어가기에 native query로 DB 조회
                - my-posts : `posts = postService.findBySearchMy(...)`
                    - selected[title,content] → searchContent

                        ```java
                        if (selected.equals("title")) {
                            return postRepository.findByTitleMy(memberId, searchKeyword, offset);
                        } else {
                            return postRepository.findByContentMy(memberId, searchKeyword, offset);
                        }
                        ```

                    - my-posts는 날짜가 들어가지 않기 때문에 일반적인 JPQL 사용
                - all : `posts = postService.findBySearch(...)`
                    - selected[title,writer,desriedJob,content] → searchContent

                        ```java
                        if (selected.equals("title")) {
                            return postRepository.findByTitle(searchKeyword, offset);
                        } else if (selected.equals("writer")) {
                            return postRepository.findSearchByMember(searchKeyword, offset);
                        } else if (selected.equals("desiredJob")) {
                            return postRepository.findByJob(searchKeyword, offset);
                        } else {
                            return postRepository.findByContent(searchKeyword, offset);
                        }
                        ```

                - members: `bySearchMembers = memberService.findBySearch(...)`
                    - selected[writer,desriedJob] → searchContent

                        ```java
                        if (selected.equals("writer")) {
                            return memberRepository.findSearchByName(searchKeyword);
                        } else {
                            return memberRepository.findSearchByJob(searchKeyword);
                        }
                        ```

            - **[중요]** 또한 이런 검색 기능에서 페이지 넘기는 기능을 적용하기 위해 page index 버튼에 request param([현재메뉴, 선택된 필터, 검색 내용])을 포함 → `th:href="@{/ (page=${i}, focus=${nav2}, selected=${searchForm.selected}, inputContent=${searchForm.inputContent})}"`
- #8
    - 기능 필요성
        - error가 발생했을 때, Spring 기본 error page가 나와서 이질감을 줬음
        - 직접 개발한 error page를 보여줌으로써 어떤 오류인지 안내하고 다른 페이지로 쉽게 이동 가능 필요
    - 해결
        - Spring에서는 자동으로 에러페이지를 연결하는 기능이 있으며, 그 기능을 이용
        - templates/error/4xx.html → 400, 404 등 Error Code가 4로 시작하는 에러들에 대해 해당 html로 연결
        - templates/error/5xx.html → 500, 505 등 Error Code가 5로 시작하는 에러들에 대해 해당 html로 연결
        - templates/error.html → 기본 에러페이지. 위의 해당되지 않는 Error Code들에 대해 연결
        - 고로 해당 html 파일을 사용자친화적으로 만들어 해당 규칙에 맞게 설정.
- #77
    - 오류 항목
        - post edit을 누를 시 동적으로 페이지를 받아오는 데, 이때 post modal에서 진행되는 것이 아닌 search bar의 input 에 editor가 들어가게 됨.
        - 문제 원인 : input 태그의 searchForm의 field의 이름을 content로 설정하고 이를 th:field로 받아오면서 해당 태그가 content의 id를 가지게 되고, MDE가 content에 mapping되어 있었기에 해당 input태그에 mde가 달리게 됨
    - 해결
        - searchForm의 field를 inputContent로 변경 (`searchForm.content → searchForm.inputContent`)

### <220801>

> #46 `Enhancement` : post edit에서 시간 수정(시간 추가 및 삭제) <br>
#52 `Enhancement` : older page 기능 추가
>

- #46
    - 기능 필요성
        - 시간을 변경할 수는 있었지만, 시간을 추가하거나 삭제하는 기능이 없었음
        - Breaking을 잘못눌러, 쓸데없이 시간이 기록되거나 시간을 잘못 추가한 경우들이 있음
        - 또한 쉰 것을 기록하지 않을 때는 시간변경을 위해 breaking을 눌렀다가 시간을 수정해야 되는 불편함 존재
    - 해결
        - edit을 통해 시간을 추가하거나 삭제할 수 있도록 설정
        - 이때, 현재 상태에 맞지 않는 시간의 개수로 설정 될 때 알림을 통해 제대로 수정할 수 있도록 유도 (ex_ STUDYING 상태인데 [12:00] ~ [13:00] 처럼 공부가 마무리되어 있게 시간이 설정되어 있다면, 다시 수정할 수 있도록 alert 알림)

            ```jsx
            if (state == 'STUDYING' & times_len % 2 == 0) {
                alert("[STUDYING] 상태입니다. 시간 개수를 확인해주세요!");
                var times_id = "times" + (times_len-1);
                var timeInput = document.getElementById(times_id);
                timeInput.setAttribute('class','form-control is-invalid');
                document.getElementById("postEdit_container").scrollIntoView();
            } else if ((state == 'BREAKING' || state == 'END') & times_len % 2 != 0) {
                alert("[BREAKING] 혹은 [END] 상태입니다. 시간 개수를 확인해주세요!");
                var times_id = "times" + (times_len-1);
                var titleHtml = document.getElementById(times_id);
                titleHtml.setAttribute('class', 'form-control is-invalid');
                document.getElementById("postEdit_container").scrollIntoView();
            } else {
                $.ajax({
                    url: '/posts/edit/' + id,
                    type: "POST",
                    data: formData
                }).done(...)
            }
            ```

        - 추가로 비어있는 상태로 시간을 수정하면 빈 시간은 자동으로 삭제되도록 설정

            ```java
            List<String> times = postEditForm.getTimes();
            while (times.remove("")) {}
            ```

- #52
    - 추가 수정 필요 부분
        - 현재까지는 그냥 older page로 다음 페이지로 넘어가는 것만 적용.
        - 즉, 1~5page 까지는 선택하여 볼 수 있었지만, 6page 부터는 older page버튼을 눌러서 하나하나씩 다음페이지로 넘겨야지 확인할 수 있었음
    - 해결 (수정)
        - 1~5p 에서 6p 로 넘어가면 page index 가 6~10로 설정될 수 있도록 설정
        - 또한 7p도 6~11 범위에 있기에 7p에서도 6~11p 의 인덱스로 보여질 수 있도록 설정
        - page 값을 이용하여 인덱스 계산 실행
            - `th:each="i: ${#numbers.sequence(5*((page -1)/5)+1, 5*((page -1)/5)+5)}"`
            - 1~5, 6~10, 12~16 등 범위별로 5로 나눌때의 몫이 같다는 점을 이용하여 수식 설정 ⇒ page 값에서 1을 뺀 값을 5로 나눈 몫을 5를 곱하고 1~5을 더해 설정.

<aside>
⚠️ <strong>[오늘의 발견] Fragment 사용 시 주의 점!</strong> <br>
fragment로 html파일을 받아올 때는 해당 지정된 id의 요소만 가져옴! 즉, 해당 요소에서 적용되는 JS같은 경우도 그 요소안에다 <code>&lt;script&gt;</code> 를 넣어주거나 외부 js파일에 넣어주어야 함. 하지만 위같은 경우 외부 js에서는 사용되지 않지만 렌더링 후 사용되는 변수(<code>var state= [[${postForm.state}]]</code> 등)가 있으므로 요소 안에다 <code>&lt;script&gt;</code> 를 넣어주어야 함!

</aside>

### <220728>

> #69 `Bug` : post_times DB 저장 오류 <br>
#71 `Bug` : 인증에 따른 html 오류 변경 <br>
#59 `Enhancement` : 하루 목표 시간에 따른 알림
>

- #69
    - 기존의 문제점
        - 4월,5월에 대한 post들의 저장된 post_times가 중복되어 저장되어 있어서 시간계산등에서 기존의 2배의 시간으로 기록됨 → ex) [12:00~15:00] [16:00~17:30] [12:00~15:00][16:00~17:30] 과 같이 중복되어 저장.
        - 이렇게 되면 전체 공부 시간, 월별 공부시간 등 통계자료에  영향을 미침
    - 해결
        - DBM 에서 수정하려 했지만, post_times가 Embedded Type으로 저장되어 있기 때문에, unique key가 없어서 삭제 시 모든 post_times가 삭제되는 현상 발생
        - Spring 내에서 코드로 직접 수정하는 방식 선택

            ```java
            @Component
            @RequiredArgsConstructor
            public class DbChange {
            
                private final InitService initService;
            
                @PostConstruct
                public void dbInit() {
                    initService.dbInit1();
                }
            
                @Component
                @Transactional
                @RequiredArgsConstructor
                static class InitService {
            
                    private final MemberService memberService;
                    private final PostNativeRepository postNativeRepository;
                    private final PostService postService;
            
                    public void dbInit1() {
                        // 4/1 ~ 5/31
                        List<Member> all = memberService.findAll();
                        for (Member member : all) {
                            List<Post> posts = postNativeRepository.getBetween(member.getId(), "2022-04-17", "2022-06-01");
                            for (Post post : posts) {
                                Post one = postService.findOne(post.getId());
                                List<String> times = one.getTimes();
                                List<String> newTimes = new ArrayList<>();
                                for (String time : times) {
                                    if (newTimes.contains(time)) {
                                        break;
                                    }
                                    newTimes.add(time);
                                }
                                one.setTimes(newTimes);
                            }
                        }
            
                    }
                }
            }
            ```

            - 해당 로직이 이루어진 Class를 Component로 등록하고 PostCostruct를 통해 프로그램이 시작되자 마자 해당 로직이 진행될 수 있도록 설정
            - 로직
                - 모든 멤버들에 대해 진행
                - 해당 멤버의 4/1 ~ 5/31 의 post를 가져온 후
                - 해당 post들 중 중복되는 값이 나올때까지 반복을 진행하며 시간을 renew 한 후 저장
- #71
    - 기존의 문제점
        - my post와 동일한 html을 사용하다보니 member detail 화면 에서도 인증없이 post edit, 댓글 인증 등에서 사용이 가능하게됨
    - 해결
        - 각 상황에 따라 인증을 진행한 후 post, comment 수정 및 삭제 등을 적용
- #59
    - 기능 필요성
        - 목표시간을 넘기지 못했을 때 동기부여를 주기 위함
    - 구현

        ```jsx
        function check_end(memberId) {
            $.ajax({
                url: '/members/times/' + memberId,
                type: "GET"
            })
                .done(function (response) {
                    var currTotal = response.hour * 60 + response.min;
                    if (currTotal < memberTodayGoalTotal) {
                        var timeDiff = memberTodayGoalTotal - currTotal;
                        var check = confirm("아직 목표 시간에 도달하지 못하셨습니다. (" + Math.floor(timeDiff / 60) + "시간 " + timeDiff % 60 + "분 부족)\n사용을 종료하시겠습니까?");
                        if (check) {
                            window.location.href = "/end";
                        }
                    } else {
                        window.location.href = "/end";
                    }
                })
        }
        ```

        - javascript, ajax 통신을 이용하여 목표시간보다 작은 경우 alert을 통해 알림을 줌
        - 그게 아니라면 그냥 바로 “/end” 를 통해 공부 종료 진행

<aside>
⚠️ <strong>[오늘의 발견] @PostConstruct 주의점</strong> <br>
@PostConstruct 의 method에서 다른 method로 진행하는 것이 아닌 직접 로직을 작성하여 진행하면 안됨 → 다른 객체의 method를 실행하는 방식으로 진행해야지 PostConstruct가 정상적으로 진행됨

</aside>

### <220727>

> #58 `Enhancement`: 상단바 메뉴(all-members) 및  멤버 상세화면 설정
>

- #58
    - 기능 필요성
        - 멤버들에 대한 프로필을 볼 수 있는 화면이 없었음
        - all-members 기능을 통해 기본적인 간단한 모든 멤버의 프로필 확인할 수 있도록 설정
        - 또 해당 프로필을 클릭하면 회원의 상세 정보 (통계자료, 목표, 공부 캘린더) 등을 확인할 수 있게 설정
    - 구현
        - 기본 “/” 에 mapping 된 controller에서 모든 멤버를 이미 넘기고 있었기에 해당 정보를 이용해서 view Template에서 일부만을 수정하여 진행
        - focus request parameter가 all-members면 post를 rendering하지않고 해당 위치에 모든 멤버의 프로필을 보일 수 있도록 설정. (all-members가 아니면 post(all, today, my-posts) rendering)

            ```jsx
            <div class="col-lg-5 mb-4" th:if="${nav2 == 'all-members'}" th:each="mem:${allMembers}">
                <!-- All Members -->
                <div id="profile" class="card shadow">
                    <div class="card-header d-sm-flex justify-content-between">
                        <div style="margin-top: 5px" class="input-group">
                            <h6 class="m-0 font-weight-bold"> <span
                                    class="ml-1 h5 fw-bold">프로필</span></h6>
            								...
                    </div>
                </div>
            </div>
            ```


### <220726>

> **Article Nav바(All Members) 서비스** <br>
#58 `Enhancement`: 상단바 메뉴 (my-posts 인터셉터, all-members) <br>
#70 `Bug` : Interceptor 오류 <br>
>

- #58
    - 기존의 문제점
        - 상단바 메뉴에서 my-posts 같은 경우, 로그인된 회원의 전체 posts를 가져오는 것이기 때문에, my-posts에 들어갈 경우 인터셉터를 적용해줘야 됨
        - 또한, redirect를 통해 로그인 후 다시 my-posts로 넘어갈 수 있도록 redirectURL 포함 필수
    - 해결
        - WebConfig 에 인터셉터 적용 URL에 “/”를 추가하고 focus @RequestParam 이 “my-posts”인 경우만 로그인인증이 진행될 수 있도록 설정 → `.addPathPatterns("/members/my-page", "/members/edit/**", "/");`
        - Spring Interceptor 적용 시 포함 Url과 제외 Url은 설정이 가능하지만, request param과 같은 parameter 필터링은 지원하지 않아 Interceptor 내의 request를 통해 parameter를 직접 받아와 로직을 구성해줘야됨

            ```java
            String requestURI = request.getRequestURI();
            String focus = request.getParameter("focus");
            
            if (requestURI.equals("/")) {
                if (focus == null || !focus.equals("my-posts")) { // basic home -> / or parameters -> focus=all, today, all-members
                    return true;
                }
            } // my-posts 만 로그인 인증 진행
            ```

            - “/” url에 대해서 focus paramter가 my-posts가 아닌 모든 url은 true를 통해 로그인인증을 진행하지 않음
- #70
    - Bug : 추가적인 로직에 따라 My Page가 들어가지지 않아짐
    - 해당 로직을 짜는 과정에서 기존의 My Page에 대한 Interceptor 오류가 발생 → url과 request param 판정 과정에서의 실수 해결 (조건문 실수)

### <220725>

> **Article Nav바(All, Today, My Posts) 서비스** <br>
#58 `Enhancement`: 상단바 메뉴 개발 <br>
#44 `Enhancement`: Today 활성화
>

- #58, #44
    - 개발 필요성
        - 현재 모든 Post들에서 확인할 수 있지만, 오늘 작성된 Post, 나의 Post등 조건에 따른 post를 보여줄 필요가 있음
        - 즉, 필터링된 Post들에 대해서 보여줄 필요 존재
    - 구현
        - 이들을 Nav바를 통해 All(모든 Posts), Today(오늘의 Posts), My Posts(나의 모든 Posts) 를 선택하여 Post들을 볼 수 있음
        - 이들을 @RequestParam으로 구분 → `@RequestParam(name = "focus", defaultValue = "all") String focus` (all, today, my-posts)

            ```java
            if (focus.equals("today")) {
                DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd");
                LocalDateTime now = LocalDateTime.now();
                String date = now.format(dateTimeFormatter);
                if (now.getHour() < 4) {
                    date = now.minusDays(1).format(dateTimeFormatter);
                }
                posts = postService.getTodayPosts(date, (page - 1) * 12);
            } else if (focus.equals("my-posts")) {
                posts = postService.getMemberPosts(loginMemberId, (page - 1) * 12);
            } else {
                posts = postService.findByPage((page - 1) * 12);
            }
            ```

            - 여기서 all,today,my-posts 에 따라 repository에서 가져오는 post가 다름
            - 또한 여기서 paging기능도 있기 때문에, 각 조건에 따른 post에 더불어 paging기능까지 적용하여 repository에서 가져올 수 있도록 설정
            - 가져온 후 똑같이 home.html에 rendering → 모두 posts로 viewTemplate에 보내짐
            - 추가로 today는 요청 시간에 따라 설정되는 날짜를 지정
                - 0~4 요청 : 전날 글
                - 4~24 요청 : 당일 글

### <220723>

> #55 `Back` : 목표 설정 <br>
#52 `Back` : older page
>

- #55
    - 기능 필요성
        - 목표를 설정해서 해당 목표에 다른 달성률 등을 알려주며 공부 동기 부여를 줄 필요가 있음
    - 구현
        - 목표는 Member에 따라 설정되어야 하므로 Member 객체에 목표값을 정할 수 있는 Goal 객체 field 추가 (Embedded Type)
            - Goal

                ```java
                @Embeddable
                @Getter
                public class Goal {
                
                    private int dayGoalTimes;
                    private int weekGoalTimes;
                
                    public Goal(int day, int week) {
                        this.dayGoalTimes = day;
                        this.weekGoalTimes = week;
                    }
                
                    public Goal() { }
                }
                ```

                - `int dayGoalTimes` : 하루 목표 공부시간
                - `int weekGoalTimes` : 일주일 목표 공부시간
                - Entity에 Embedded Type으로 저장되기 때문에 Custom 객체인 Times로 저장이 안됨 → 해당 값을 분(int)으로 바꿔서 저장
        - 새로운 Member가 생성될 때, 기본적인 목표시간이 정해진 상태로 저장될 수 있도록 설정 → `member.setGoal(new Goal(300, 1500));`
        - 추가로 목표 설정은 My Page에서 가능하도록 설정
            - goal 설정 시 편의상 시간과 분을 따로 입력 받아야 하기 때문에 기존의 Goal 객체가 아닌 입력 Form을 위한 GoalDto 생성

                ```java
                @Data
                static class GoalDto {
                
                    private int dayHour;
                    private int dayMin;
                    private int weekHour;
                    private int weekMin;
                
                    public GoalDto(Goal goal) {
                        int day = goal.getDayGoalTimes();
                        this.dayHour = Math.floorDiv(day, 60);
                        this.dayMin = day % 60;
                        int week = goal.getWeekGoalTimes();
                        this.weekHour = Math.floorDiv(week, 60);
                        this.weekMin = week % 60;
                    }
                
                    public GoalDto() {
                    }
                }
                ```

                - Times로 하면 두단계로 modelAttribute가 동작해야됨 → 이 부분이 되지 않아 모두 한단계의 modelAttribute가 동작하도록 모든 field를 int로 설정
        - 또한, 이 목표에 따른 달성률(하루 목표 달성률, 일주일 목표 달성률) 보여줌
        - 이에 따른 랭크도 추가 (브론즈, 실버, 골드, …)
- #52
    - 기존의 문제점
        - paging에 따른 post들을 보여주긴했지만, 5페이지 넘겨서의 post들을 보여주지 못했음
        - older 버튼이 있었지만 활성화되고 있지 않았음
    - 해결
        - older 버튼을 누르면 이전 page로 계속해서 넘어갈 수 있도록 설정

            ```html
            <ul class="pagination justify-content-center my-4">
                <li class="page-item">
                    <a class="page-link" href="/" tabindex="-1" aria-disabled="true">Newer</a>
                </li>
                <li th:each="i: ${#numbers.sequence(5*((page -1)/5)+1, 5*((page -1)/5)+5)}" aria-current="page"
                    th:class="${page == i}? 'page-item active' : 'page-item'">
                    <a class="page-link" th:href="@{/ (page=${i}, focus=${nav2})}" th:text="${i}">#</a>
                </li>
                <li class="page-item">
                    <a class="page-link" th:href="@{/ (page=${page+1}, focus=${nav2})}">Older</a>
                </li>
            </ul>
            ```

            - 현재 focus 되어 있는 page에 1을 더하여 다음 page로 넘어가도록 설정
            - `th:each="i: ${#numbers.sequence(5*((page -1)/5)+1, 5*((page -1)/5)+5)}"` 를 통해 현재 페이지에 따라 1~5, 6~10, 11~15 등으로 page index가 보일 수 있도록 설정

<aside>
⚠️ <strong>[오늘의 발견1] JPA Entity로써 DB에 저장 시 주의점</strong> <br>
DB에 Entity를 저장할 때는 DB에서 지정된 객체의 field로만 저장 가능! 즉, 비지니스 로직 상 개발한 custom class로 DB로 저장이 안됨. 따라서 DB에 저장하기 위해선 기본 객체들을 이용한 field로 변경 후 저장 필요 (ex_ hour, min을 저장하는 Times Custom Class 자체로 DB에 저장이 안되고, 이를 int(Integer)로 변경하여 저장해야 됨)

</aside>

<aside>
⚠️ <strong>[오늘의 발견2] Form 입력 및 @ModelAttribute 주의점</strong> <br>
만약 Goal 이라는 객체의 field에 Times라는 또다른 객체가 있고 Times의 hour가 min field를 변경하고 싶을때 ModelAttribute를 사용하면 먹히지 않음. 즉, ModelAttribute는 두단계에 거쳐서 set을 하지는 못함 → http message를 보면 dayTimes.hour=3 이런식으로 set이 진행됨. 이 부분이 제대로 동작되지 못함. <br>
<b>결론 : ModelAttribute를 사용하려면 한단계의 field만 적용되도록 객체를 설정해야 됨. (두단계의 객체 설정X) </b>
</aside>

### <220722>

> **My Page (내 프로필) 서비스** <br>
#54 `Enhancement` : 공부 달력 및 해당 월별 통계
>

- #54
    - 기능 필요성
        - 지금까지는 자신이 썼던 글을 확인할 수 없으며
        - 그에 따라 얼마나 공부를 했는지도 알 수 없고, 어떤 공부를 했는지도 알 수 없었음
        - 이를 월별로 어떤 공부를 했고 공부를 얼마나 했는지 확인할 수 있는 수단이 필요
    - 구현
        - 월별로 어떤 공부를 했고, 해당 일자의 글을 확인하고 수정할 수 있게끔 공부 달력 및 해당 월별 통계 서비스 제공
        - 공부 달력
            - 기본적으로 현재의 달을 기준으로 달력을 보여주고, < > 버튼을 통해 다른 달로 넘어갈 수 있게 설정
            - 각 날짜에 해당하는 post가 있는지 확인 후 있으면 해당 post의 제목을 달력에 보여줌
            - 또한, 그 제목을 클릭하면 모달형식으로 해당 post가 뜰 수 있도록 설정

            ```java
            if (month == null) {
                month = LocalDateTime.now().getMonthValue();
            }
            focusedDate = LocalDateTime.of(year, month, 1, 0, 0);
            List<Post> monthPosts = postService.getMonthPosts(member.getId(), month);
            List<PostViewDto> monthPostDtos = monthPosts.stream().map(PostViewDto::new).collect(Collectors.toList());
            int[] dayData = {31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31};
            List<String> weekDay = Arrays.asList("sunday", "monday", "tuesday", "wednesday", "thursday", "friday", "saturday");
            List<Integer> calenderDays = new ArrayList<>();
            Map<Integer, PostViewDto> calenderData = new HashMap<>();
            for (int i = 1; i <= dayData[month - 1]; i++) {
                calenderDays.add(i);
                calenderData.put(i, null);
            }
            for (PostViewDto monthPostDto : monthPostDtos) {
                int dayOfMonth = monthPostDto.getCreateTime().getDayOfMonth();
                calenderData.put(dayOfMonth, monthPostDto);
            }
            ForCalender CalenderInfo = getCalenderInfo(weekDay, focusedDate);
            ```

            - @RequestParam 으로 month를 받아 오는데, 이때 month가 없다면 현재 월을 focus
            - 해당 month의 1일을 시작 날짜로 설정
            - 해당 member의 해당 month의 post를 가져옴 (navtive query 사용) → `postService.getMonthPosts(member.getId(), month);`

                ```java
                @Query(value = "SELECT * " +
                        "FROM post p " +
                        "where p.member_id = :m_id " +
                        "and MONTH(p.create_time) = :request_month ;", nativeQuery = true)
                List<Post> getMonthPost(@Param(value = "m_id") Long id, @Param(value="request_month") int month);
                ```

            - 해당 월에 맞는 calender 생성 (List로 달력을 만들고, Map으로 해당 날짜에 맞는 post를 넣어줌)
            - 마지막으로 해당 calender의 정보(년도, 월, 시작요일 등)를 담아 View Template에 넘겨줌
        - 공부 통계
            - 현재 focus 되어 있는 달의 모든 post를 가져와서 “출석률, 총 공부시간, 평균 공부시간” 에 대한 통계 자료를 제공

            ```java
            int monthDate = dayData[month - 1];
            int nowMonthPostsLen = 0;
            if (month == LocalDateTime.now().getMonthValue()) {
                monthDate = LocalDateTime.now().getDayOfMonth();
                nowMonthPostsLen = monthPosts.size();
            } else {
                nowMonthPostsLen = postService.getMonthPosts(loginMemberId, LocalDateTime.now().getMonthValue()).size();
            }
            AllStatic allStatic = getAllStatus(oriMember, staticsData, days.get(days.size() - 1), monthPosts, monthDate, nowMonthPostsLen);
            ```

            - 출석률 : 해당 달의 모든 post의 개수 / 해당 달의 모든 날짜 (해당 달이 현재 달이면 현재까지의 출석률로 계산)
            - 총 공부시간 : 해당 달의 모든 post를 가져와서 시간계산
            - 평균 공부시간 : 총 공부시간 / 해당 달의 모든 날짜 (해당 달이 현재 달이면 현재까지의 출석률로 계산)

### <220721>

> **My Page (내 프로필) 서비스** <br>
> #65 `Enhancement`: 주간 공부 그래프 (누적 그래프 X → 일반 그래프 O)
>

- #65
    - 기능 필요성
        - Ranking 화면을 통해서 일주일간의 공부시간 누적 그래프는 확인할 수 있지만, 일반 그래프, 즉 일주일간의 내가 하루하루 얼마나 공부했는지에 대한 그래프가 없어 불편
        - 또한 Profile에서의 그래프는 누적 그래프보다는 일반 그래프가 더욱 필요 (Bar Chart)
    - 구현
        - 먼저 모든 멤버들의 일주일 간의 post를 가져온 후 시간 계산 (`getStaticsData`)

            ```java
            private Map<Integer, Times> getStaticsData(MemberDto member) {
            
                List<Post> posts;
                Map<Integer, Times> dayTimes = new HashMap<>();
                DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd");
                // 0-4 사이 요청 (이틀전까지의 데이터)
                if (LocalDateTime.now().getHour() < 4) {
                    String end = LocalDateTime.now().minusDays(1).format(dateTimeFormatter);
                    String start = LocalDateTime.now().minusDays(8).format(dateTimeFormatter);
                    posts = postService.getLast7(member.getId(), start, end);
                    for (int i = 8; i > 1; i--) {
                        dayTimes.put(LocalDateTime.now().minusDays(i).getDayOfMonth(), new Times(0));
                    }
                } else {...} // 이외의 요청 (4~24 사이 요청, 하루전까지의 데이터)
                }
                for (Post post : posts) {
                    Times time = getTimes(post.getTimes());
                    int today = (time.getHour() * 60 + time.getMin());
            				...
                }
                return dayTimes;
            }
            ```

            - 먼저 날짜별의 시간데이터를 저장하기 위해 Map 사용 (날짜와 인덱스가 일치하지 않기 때문에 List 말고 Map 사용)
            - nativequery에서 Between을 사용하여 요청시간에 따른 post list를 가져옴 (0-4 사이 요청 → 이틀전까지의 데이터, 이외의 요청(4~24 사이 요청) →  하루전까지의 데이터) [`getLast7(Long memberId, String start, String end)`]
            - 마지막으로 Times 객체를 이용하여 시간 계산 후 해당 정보가 들어있는 dayTimes Map 반환
        - 그 후 해당 데이터를 view Template으로 보낸 후 이 값을 Javascript로 받아와 BarChart의 데이터 값에 넣어줌 (staticDays 는 staticData의 key를 정렬한 값. → 정렬된 날짜)

            ```jsx
            var staticDays = [[ ${staticDays}]];
            var data = [[ ${staticData}]]
            
            // Bar Chart
            var ctx = document.getElementById("myBarChart");
            var myBarChart = new Chart(ctx, { datatsets: ...})
            ```


### <220720>

> **My Page (내 프로필) 서비스 추가** <br>
#13 `Enhancement` : 사용자 정보 보기 <br>
#53 `Back` : 모든 공부 시간 <br>
#67 `Back` : Spring Interceptor 적용
>

- #13
    - 기능 필요성
        - 지금까지는 상대방의 상세 프로필을 확인할 수 없을 뿐더러 자신의 상세 프로필도 확인할 수 없었음
        - 링크, 별명 등 자신의 정보, 다른 사용자들의 정보를 보여줄 필요가 있음
    - 해결
        - My Page를 생성해서 사용자의 상세 정보를 볼 수 있는 페이지 추가 (사용자는 Session에 저장된 member → 로그인된 멤버)
        - Session에 저장된 member의 Id를 받아와 해당 member의 정보를 DB에서 가져와 보여줌. (login id, name, nickname, introduce, links)
            - `@SessionAttribute(name = SessionConst.LOGIN_MEMBER, required = false) Long loginMemberId` → `Member oriMember = memberService.findOne(loginMemberId);`
            - DTO로 변경 후 보여줄 정보만을 veiw Template에 넘겨줌 → `MemberDto member = new MemberDto(oriMember);`
        - link는 클릭 시 해당 link로 이동할 수 있도록 설정
- #53
    - 기능 설명
        - 해당 멤버가 지금까지 얼마나 공부를 해왔는지  알려주는 기능 (시간)
    - 구현
        - 프로필에 추가 해당 기능 추가
        - 해당 멤버의 모든 post를 가져온 후 각 post의 times들의 시간 값을 계산, 그 후 member에 저장 → `member.setTotalTime(getAllTimes(oriMember));`

            ```java
            private Times getAllTimes(Member member) {
                List<Post> posts = member.getPosts();
                int total = 0;
                for (Post post : posts) {
                    if (post.getTimes().size() % 2 != 0) continue;
                    Times times = getTimes(post.getTimes());
                    total += times.getHour() * 60 + times.getMin();
                }
            
                return new Times(total);
            }
            ```

- #67
    - 기능 필요성
        - My Page 같은 경우, 자신의 정보를 확인하고 수정하는 공간이므로 로그인 인증 및 회원 일치 인증이 필요
    - 해결
        - Login Interceptor에 해당 url 추가
        - `.addPathPatterns("/members/my-page", … )`
        - 만약 로그인이 되어있지 않다면 로그인 후 들어올 수 있도록 설정

### <220719>

> #65 `Bug` : static 그래프 오류
>

- #65
    - 발생된 Bug
        - 0~4시의 요청에 대해선 이틀전까지의 데이터만 보여져야되는데, 0시20분 쯤에 요청했을 때, 하루전의 데이터가 보여짐
        - 누적 데이터인데, 중간에 누적되지 않는 현상 발생
    - 해결
        - 0~4시의 요청에 대한 분기를 if (0<nowTime…) 이런 식으로 범위 설정을 잘못함. → `if (now.getHour() < 4)` 로 범위 수정
        - 누적 데이터 문제
            - 날짜에 대한 HashMap의 key를 가져올때, 따로 정렬하지 않아 발생 → keySet을 통해 가져오면 당연히 정렬없이 가져와짐.
            - 이는 keySet을 가져오고, 날짜를 오름차순으로 정렬하여 해결

                ```java
                List<Integer> list = new ArrayList<>(dayTimes.keySet());
                list.sort(Comparator.naturalOrder());
                ```

                - keySet을 list로 변경하고 sort를 통해 정렬
            - 이렇게 되면 날짜가 뒤죽박죽되지 않고 오름차순으로 진행되기 때문에, 코드 상 누적이 제대로 진행됨

### <220718>

> #63 `Bug` : 첫 로그인 시 발생하는 오류 (session Id 이동으로 인한 문제) <br>
> #64 `Enhancement` : static 서비스 기능 추가 (일주일 간의 공부 횟수, 공부왕, 출석왕 기능 추가)
>

- #63
    - 기존의 문제점
        - 첫 로그인 시 (세션 저장 시) 오류 발생
        - 세션 저장 후 해당 값을 url로 보내지면서 발생하는 오류로 판단 (쿠키가 적용되지 않는 상황을 대비한 servlet 자동 기능 → traking-modes)
    - 해결
        - servlet 설정을 통해 해당 자동 기능을 막음 (cookie로 설정함으로써 쿠키로만 session을 인식하겠다는 설정)

        ```yaml
        server.servlet.session.tracking-modes: 'cookie'
        ```

- #64
    - 일주일 간의 공부 횟수
        - 일자로 count 하며 공부시간이 0인 post들에 대해서 counting
        - 그 후 7-cnt 로 공부 횟수 기록
        - 마지막으로 해당 값을 memberDto에 저장, veiw Template으로 넘겨줌
    - 공부왕, 출석왕 기능
        - 공부왕 : 일주일간의 누적 공부시간이 가장 많은 사람
        - 출석왕 : 일주일간의 공부 횟수가 가장 많은 사람

### <220717>

> #62 `Back` : 배운내용적용 → 최적화
> <ol>
> <li> Spring Security 없이 로그인 개발 (authentication → session) </li>
> <li> session timeout 연장 </li>
> </ol>
> #61 `Front` : 로그인 페이지 생성
>

- #62-1
    - 기존의 문제점
        - 제대로 배우지 않은 Spring Security를 사용함으로써 원하는 기능을 제때 제대로 이용하지 못함 (session 설정 등) → 추후 학습 후 적용 예정
    - 해결
        - **session**을 이용한 login(인증)으로 구현 → 로그인 검증 및 로그인을 모두 직접 구현
        - Spring Security와 관련된 모든 코드를 제거하고 session과 관련된 코드로 대체 _(비밀번호 인코딩 제외)_
        - 로그인

            ```java
            @PostMapping("/members/login")
                public String login(@Validated @ModelAttribute("loginForm") LoginForm loginForm, BindingResult bindingResult,
                                    @RequestParam(defaultValue = "/") String redirectURL,
                                    HttpServletRequest request) {...}
            ```

            - 로그인 시 중간에 로그인 요구에 의해서 로그인을 하는 상황을 대비해서 redirectURL을 받아옴 (없을 경우 그냥 home으로)
            - session을 받아오기 위해 HttpServletRequest 이용
            - 로그인 검증

                ```java
                // form 형식에 맞지 않은 제출
                if (bindingResult.hasErrors()) {
                    return "login";
                }
                
                // 로그인 검증         // Global Error -> Object 단 오류! (직접 처리해야 되는 부분)
                Optional<Member> loginMember = memberService.findByLoginId(loginForm.getLoginId());
                BCryptPasswordEncoder passwordEncoder = new BCryptPasswordEncoder();
                if (loginMember.isEmpty() || !passwordEncoder.matches(loginForm.getPassword(), loginMember.get().getPassword())) {
                    bindingResult.reject("loginFail", "아이디 또는 비밀번호가 맞지 않습니다.");
                    return "login";
                }
                ```

                - Form 형식에 맞지 않은 제출에서의 검증은 Bean Validation 사용 (`FieldError`)
                - 아이디 및 비밀번호 오류 시 `ObjectError` 발생 후 veiw Template으로 넘겨 줌
                - `Optional`을 통해 오류 검증
            - 로그인 성공

                ```java
                // 로그인 성공
                HttpSession session = request.getSession();
                // 세션에 로그인 회원 정보 보관
                session.setAttribute("loginMember", loginMember.get().getId());
                return "redirect:"+ redirectURL;
                ```

                - 로그인 성공 시 session에 회원정보(member id) 저장
                - **member 객체 자체를 저장하지 않고, member id만을 저장하여, 추후 이를 통해 repo에서 member객체를 가져오는 방식으로 진행**
- #62-2
    - 기존의 문제점
        - session timeout을 설정하지 않아, 너무 자주 로그아웃되어 사용 시 불편함
    - 해결
        - spring securrity를 사용하지 않고 직접 session을 이용하고, session timeout을 applications.yml 에 설정하여 관리 (timeout을 10시간으로 설정)

            ```yaml
            server.servlet.session.timeout: 36000
            ```

- #61
    - 기존의 문제점
        - 로그인 페이지 없이 home에서 로그인을 진행하다 보니 쓸데없이 데이터가 주고받아지는 경우가 있음. (home에서 로그인 이전에도 다량의 데이터를 보여주고, 로그인 후에도 다량의 데이터를 보여주고 있음. → 비 효율적)
        - redirect를 통한 로그인 후 페이지 이동이 불가능.
    - 해결
        - 일반적인 웹사이트처럼 로그인 페이지를 따로 만들어 제공
        - 로그인 페이지를 새로 생성
- #30
    - 기존의 문제점
        - 세션 만료 시 에러페이지 발생
        - 세션 만료에도 서비스를 유동적으로 이용하기 위해 로그인 페이지 후 redirect를 통해 되돌아가는 것이 필요
    - 해결
        - Interceptor에서 로그인 페이지로 넘어가는 request에 redirectURL query parameter를 추가.
        - 로그인 후 해당 redirectURL query parameter 를 통해 이전 페이지로 되돌아감. 
        - `response.sendRedirect( "/members/login?redirectURL=" + request.getRequestURI())`
        - `@RequestParam(name = "redirectURL", defaultValue = "/") String redirectURL` → `return "redirect:" + redirectURL`

<aside>
⚠️ <strong>[오늘의 발견] session에 정보 저장 시 주의점!</strong> <br>
session에 member 객체를 그대로 저장하면 그 저장된 member를 사용하기 때문에 변경된 값이 적용되지 않음 ( → 저장 시점의 member를 사용하는 것) <br>
그렇기에 <b>session에는 id만을 저장하고 사용 시 그 id를 통해 repo에 저장되어 있는 member(변경 사항 반영된 member)를 가져와야</b>됨

</aside>

### <220714>

> #48 `Enhancement` : 새로운 대시보드 (Static 서비스 개선)
>

- #48
    - 기존의 문제점

      파이썬으로 통계내어 해당 그래프를 사진으로 저장하고, 그 사진을 불러오는 형식으로 진행, 하지만 이는 통합되지 않는 문제가 있으며, 사진으로 보임으로써 부가적인 서비스를 제공하지 못함

    - 해결
        - javascript를 이용하여 그래프를 보여주는 형식으로 진행.
        - Model로 데이터를 담아와 thymeleaf in javascript 기술을 사용하여 데이터 이용
        - 멤버당 최근 1주일 post를 가져와 해당 post의 모든 시간을 더한 시간을 적용
            - 중요한 점은 새벽4시를 하루의 끝과 시작으로 정했기때문에, 요청 시간이 0~4시 사이일 경우와 나머지 시간일 경우에 대한 처리 구분 필요
        - 그래프로는 누적 시간을 보여줌
        - 추가로 해당 일주일간의 누적 시간에 대한 랭킹 표시
        - StaticController 주요 코드

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

            - getDays : 그래프에 표시할 날짜모음(일)
            - getStaticData : 그래프에 표시할 데이터 생성
                - 0~4시의 요청 : 8일전 ~ 이틀전 까지의 데이터를 가져옴
                - 이외의 요청 : 7일전 ~ 하루 전 까지의 데이터를 가져옴
                - 추가로 날짜마다의 데이터를 넣어주기 위해 HashMap 사용 (key: 날짜, value: 시간) 이때, 해당 날짜의 post가 없는 경우 이전 데이터를 넘겨받아 사용하도록 설정 (누적 데이터이므로)
            - 또한 각 멤버별로 랜덤한 색깔을 설정하여 각 멤버별의 그래프가 구분 가능하도록 설정
            - 그 후 랭킹표시를 위해 TimeSorter로 정렬
        - Javascript 주요 코드

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
                        labels: [staticDays[0]+'일',staticDays[1]+'일',staticDays[2]+'일',staticDays[3]+'일',staticDays[4]+'일',staticDays[5]+'일',staticDays[6]+'일'],
                        datasets: datasets,
                    },
            ...})
            ```

            - thymeleaf in javascript 기술의 for each 를 사용하여 user를 만들고 해당 user의 데이터를 dataset에 넣어 그래프에 대입.


### <220713>

> #42 `Enhancement`: 하루 공부시간 랭킹 <br>
#62 `Back` : MVC 최적화
>

- #42
    - 기존의 문제점

      우측 아티클에서 누적 공부시간 그래프를 사진으로 대략적으로 보여주는 부분이 있었으나 사실상 의미가 없는 기능이였으며, 대시보드를 사진이 아닌 js로 구성하기로 결정. 추가로 공부 동기부여를 높여주기 위한 기능이 필요했음

    - 해결 (**하루 공부시간 랭킹**으로 **대체**)
        - 기존의 제공하는 기능 중 하루 공부시간 계산 로직(`getTotalStudyTimes`())을 이용
        - 이 로직을 모든 Member들에게 적용하여 각 Member의 하루 공부시간을 계산하도록 진행

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

        - 이 시간을 각각의 MemberDto의 time property에 저장 후 이를 View Template에서 받아, 이용.
        - 여기서 중요한 부분은, 하루 공부시간 **랭킹**이기 때문에, 이들을 시간(`Times`)에 따라 정렬이 필요 →  **custom Comparator**(`TimeSorter`) 생성 후 `sort` 적용

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

            - Times가 hour와 min으로 구성되어 있기 때문에, 이를 모두 minute으로 변경 후 이들에 대한 정렬을 실행.

            ```java
            List<MemberDto> memberRanking = new ArrayList<>(memberDtos);
            memberRanking.sort(new TimeSorter());
            ```

- #62
    - 기존의 문제점
        - 페이징에 따른 Controller가 존재 → 번잡한 구성
        - Static 화면도 HomeController에 존재 → 관심사 분리 필요
    - 해결
        - 페이징 Controller 통합 → `@RequestMapping({"/", "/{page}"})` , `@PathVariable(name = "page", required = false) Integer page` , `if (page == null) page = 1;`
            - url로 변수를 받아오고 해당 변수가 없다면 1로 설정하여 main은 page 1로 갈 수 있도록 설정
        - StaticController 생성 → (HomeController와 분리)

### <220618> ~ <220630>
- **Spring MVC 집중 공부! (강의 및 개인 학습으로 인해 강의에서 진행한 개발 의외의 개발은 진행하지 않음)**
- 추후 해당 공부한 내용을 바탕으로 **Semogong Project를 Upgrade 시킬 예정**


### <220615>

> `Front` : 사용자 편의를 위한 애니메이션 추가
>

- 현재 문제점
    - Home 화면에서 데이터가 변경되고 있는 Point가 어딘지 확인 불가
    - Static 화면에서 현재 인원이 변경되고 있는지 확인 불가
- 해결
    - Home 화면은 해당 Point에 물결 애니메이션을 추가하여 현재 데이터가 변하고 있는 지점에 대해 Focus할 수 있도록 설정
    - Static 화면은 Live 버튼에 현재 데이터를 불러오고 있으면 초록색 깜빡임을 주어 데이터가 반영되고 있다는 사실을 사용자에게 인지할 수 있도록 설정

### <220614>

> `Back` : 인터셉터를 통한 최적화
>

- 현재 문제점
    - 현재 Temp → Info로 바꾸는 로직은 한 서비스가 시작될 때 모든 Temp를 Info로 바꾸는 식의 방식으로 진행 중
    - 하지만 현재 방식은 사실상 비효율적임. 모든 DB를 가져오는 것보다, 최근 요청시점을 기억하고 그 이후로 새로 들어온 Temp들만을 Info로 바꾸는 방법이 필요
- 해결
    - Point에 최근 Info의 변경시점을 기록하여
    - 해당 변경시점 이후에 새로운 Temp가 있는지 확인하여 해당 Temp를 Info로 변경하는 방식으로 진행
    - 또한 이는 어떤 서비스든 시작될때 적용이 되어야 하므로 **인터셉터** 개념 도입
- UpdateInterceptor

    ```java
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        List<Point> pointList = pointService.findAll(); // find all Point
        for (Point point : pointList) { // each point
            infoService.createInfo(point.getId()); // make 연관관계 with Point (새로 생성된 Temp들에 대해서 새로운 Info 생성)
            pointService.updatePoint(point.getId()); // point의 lastCommittedTime update (최근 Info update 시간 update)
        }
        return true;
    }
    ```

    - CheckPoint 이후 새로 생성된 Temp들에 대해서 새로운 Info 생성
    - CheckPoint Update
    - 마지막으로 WebConfig를 통해 Interceptor 등록

### <220612>

> `Enhancement` : 누적 통계 정보 조회 개발
>

- Static에서 보여줄 누적 통계 정보에 대한 개발 진행
- 통계 로직
    - 해당 Point에 대한 모든 Info를 조회
    - 조회한 후 날짜별로 Info를 나누고 해당 날짜에 대한 Info 개수의 평균치를 model에 담아 rendering
    - 일주일간의 데이터, 시간별 데이터 모두 한곳에 저장
    - 해당 데이터를 Pie,Arial Chart 를 구성하는 js 에 할당 (탬플릿 이용)

<aside>
💡 <strong> JavaScript에서 model의 attribute(model에 담겨져 넘겨온 값) 사용하기!</strong> <br>
<code>[[ ]]</code> 사용!(HTML 테그의 속성이 아니라 <b>HTML 콘텐츠 영역안에서 직접</b> 데이터를 출력하고 싶을 때 사용) → <code>var temp = [[ ${temp} ]]</code>

</aside>

### <220610>

> `Back` : Temp, Info Repository, Service 재정의
>

- Temp, Info Repository, Service  재정의
    - 기존에 사용했던 Info의 Domain, Service, Repository를 모두 Temp로 이전, 그 후 Info Repository, Service 새로 정의
    - 핵심은 Temp로 받아온 라이브 데이터를 Info로 바꾸어 저장하는 것. 즉, Temp의 데이터를 그대로 받아와 연관관계를 정의하고 Entity로써 관리되게 만드는 것
- Info Repository
    - 저장만 하면 됨 (조회는 연관관계를 맺은 Point로 진행) → `em.persist(info)`
- Info Service
    - Info 생성 메서드만 존재 → (매핑 로직: ) 해당 구역에 따라 Point와 매핑시켜준 후 저장.  그 결과로 측정된 수치들을 바탕으로 Point를 수정

### <220609>

> `Back` : 이전 Info 데이터와 Point의 연관관계 생성 필요
`Back` : Temp, Info Domain 재정의
>

- 발견된 문제점
    - 실시간 데이터로 받아오는 Info를 그 즉시 Floating 해주는 기능은 가능하나, Info의 과거 데이터를 가져올 때 이미 한번 매핑되었던 Point와 다시 판단 로직을 통해 매핑이 필요함
    - 또한 Info가 Entity로써 **저장**되지 않았기에 발생하는 불편함 해결 필요
- 해결법
    - 기존의 데이터를 가져와서 새로 Entity로 관리되는 데이터를 생성 → Temp라는 Object로 기존의 데이터 받아옴
    - 추가로 Info를 가져올 때 point와 연관관계를 맺어 서로 참조할 수 있도록 설정

        ```jsx
        //==연관관계 메서드==//
        private void setPoint(Point point) {
            this.point = point;
            point.getInfos().add(this);
        }
        ```

    - 기존에 사용했던 Info의 Domain, Service, Repository를 모두 Temp로 이전, 그 후 Info 새로 정의

### <220608>

> `Front` : Home, Static 화면 수정
`Back` : HomeRestController 개발
>

- Home, Static 화면 수정
    - Home
        - Kakao Map API를 이용한 지도에 Point의 좌표를 매핑시켜 floating
        - 각 Point들에 대한 실시간 변동량 표시를 위한 개발 → ajax 비동기 통신, REST API 이용 → REST API 를 위한 Controller 개발 필요

        ```jsx
        $( document ).ready(function detection_change() {
            interval = setInterval(function () {
                $.ajax({
                    url: '/infos',
                    type: "GET"
                })
                    .done(function (response) { ... //로직 }
        })
        ```

    - Static
        - 실시간 인원 측정량 표시 (버튼을 누르면 볼 수 있고, 아니면 안보여주는 느낌으로 진행 → 버튼을 누르면 interval 함수를 사용하여 3초마다 데이터를 요청, JSON으로 값을 받아옴)
        - 이 또한 ajax 비동기 통신, REST API 이용 → REST API 를 위한 Controller 개발 필요
- HomeRestController 개발
    - REST API 를 위한 Controller 개발 (`@RestController`)
    - Home화면과 Static 화면에 뿌려줄 JSON 데이터 생성 및 반환
    - JSON 데이터는 실시간으로 측정된 인원의 수를 보여주는 count와 이 데이터 리스트를 반환 (`Result<T>` 사용)

### <220606>

> `Back` : HomeController 개발
>

- 공통
    - Home 화면이든 Static 화면이든 모든 Point들에 대한 정보가 필요하기 때문에 point list를 기본적으로 Model에 추가해줌. (`@ModelAttribute` 사용)
- Home
    - API를 이용하기 때문에 따로 Model에 추가해줄 것들이 없음.
    - Point List는 미리 추가된 상태의 model 사용
- Statics
    - 검색해서 들어오는 상황과, 기본 static 화면 분리 (데이터 분리)
    - 그에 대한 추가적인 정보(Live Count) 등 추가
    - 추가적으로 필요한 부분 : 데이터가 추가되면 누적통계 데이터에 대한 처리 필요.
    
### <220605>

> `Back` : Info Service, Repository 개발
>

- Info 도메인의 주의점
    - 이미 실시간으로 저장되어 있는 데이터를 가져와서 floating하는 역할
    - 그러므로 따로 저장할 필요는 없고, 데이터를 가져오는 역할만 하면 됨
    - 여기서 기존의 데이터를 가져오기 위해선 JpaRepository, NativeQuery 를 사용해야 됨
- 개발
    - 10초~현재 의 데이터를 받아오는 method → static 화면에서 사용

        ```java
        @Query(value = "SELECT * " +
                    "FROM temp t " +
                    "where t.date >= DATE_SUB(CONVERT_TZ(NOW(),'SYSTEM','Asia/Seoul'), INTERVAL 10 SECOND);", nativeQuery = true)
            List<Temp> getLiveData();
        ```

        - 해당 Entity로 저장된 데이터를 가져오는 것이 아닌 기존에 있는 데이터를 가져오기 위해서 JpaRepository, NativeQuery 사용
        - 10초 전~현재의 누적 데이터를 가져옴 → DATE_SUB() sql문 사용
    - 저장은 따로 필요없음

### <220604>

> `Back` : Basic Point Service, Repository 개발
>

- Repository 단
    - 정해진 지점에 대해서 저장하기 때문에 `@PostConstruct`를 통해 미리 DB 저장. (1회성 initDB)
    - 다건조회, 단건조회, 이름으로 조회. 3가지 method 생성
- Service 단
    - 리포지토리의 역할을 단순 위임
- 추가할 점
    - 현재는 누적된 데이터가 없기에 누적통계량을 위한 데이터에 대한 Repository나 Service가 정의되어 있지않음 → 데이터가 어느정도 누적된 이후 개발 필요 

### <220602>

> `Back` : Domain 개발
>

- 필요성
    - 현재 필요한 부분은 장소(검출 지점)를 연결할 Point와 실시간으로 DB에 적재되는 검출객체정보를 연결할 Info가 필요
- 개발 과정
    - Point 도메인 개발
        - field 설정, 생성 및 수정 메서드 생성
    - Info 도메인 개발
        - 기존의 DB와 연결
        - field 설정, 생성 및 수정 메서드 생성

### <220601>

> `Front` : 탬플릿 커스텀 (home, static)
>

- 프론트를 전문적으로 하는 인원이 없어, 무료로 제공되는 템플릿을 사용하기로 결정. 우리가 하고자 하는 서비스에 맞는 대시보드 탬플릿을 받아와 커스텀.
- 여러 html 파일 중 home 화면과 static 화면만 살리고 이들에 대해 커스텀 진행.


### <2205010> ~ <220530>
- **Spring MVC 집중 공부! (강의 및 개인 학습으로 인해 강의에서 진행한 개발 의외의 개발은 진행하지 않음)**
- 추후 해당 공부한 내용을 바탕으로 **Semogong Project를 Upgrade 시킬 예정**

### <220508>

> **Tag : version 1.0.11** <br>
**#39** `Enhancement` : 실시간 학습 시간 reload 버튼 애니메이션 추가
>

- **#39**
    - 기존의 문제점: 실시간 학습 시간 reload 버튼을 눌러도 잘 눌려졌는지를 확인할 방법이 없었음 → alert을 하려했지만, alert은 뜨면 꺼야된다는 귀찮음 확인
    - 해결 : reload 버튼을 누를경우, reload img 가 360도 rotate 되면서 시각적으로 재부팅 되는 효과 제공 → `<img id="reload" src="/images/reload.png" height="20px" width="20px">`

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

> **#39** `Front` : 실시간 학습 시간 reload 기능 추가 <br>
**#39** `Bug` : 23시, 00시의 차이 오류 해결
>

- **#39**

    - 기존의 문제점 : 현재 공부시간을 확인하려면 무조건 home을 reload해야 되는 비효율적인 방법을 선택해야 했음. → 네트워크, 렌더링 과부화 가능
    - 해결 : 학습 시간 자체에 reload 버튼을 추가하여 버튼을 클릭할 경우 ajax api 통신을 통해 해당 계산된 학습 시간만 받아와 렌더링 할 수 있도록 설정

        ```jsx
        function reload_times(memberId) {
            $.ajax({
                url: 'members/times/' + memberId,
                type: "GET"
            })
                .done(function (response) {
                    $('#member_time').replaceWith(
                        '<div id="member_time"> 오늘의 학습 시간 : ' + response.hour + '시간 '+ response.min +'분 </div>'
                    )
                })
        }
        ```
      
### <220506>

> **Tag : version 1.0.10** <br>
**#2** `Bug` : 댓글 submit enter 입력할 수 있도록 설정, apple에서의 오류 수정 <br>
**#39** `Enhancement` : post edit 시에도 실시간 학습 시간이 변경되도록 <br>
>

- **#2**
    - 기존의 문제점: input에 onkeyup을 사용할 경우 apple기기에서 한글로 작성할 경우 해당 연결된 javascript 함수가 연속해서 2번 작동하는 오류 발생
    - 해결: onkeyup을 모두 onkeypress로 변경
- **#39**
    - 기존의 문제점 : post eidt 시 실시간 학습 시간은 변경되지 않았음
    - 해결 :
        - post edit 시 변경된 시간을 받아와 그 순간 javascript 안에서 현재 학습 시간 계산 로직(위에서 작성한 로직)을 통해 현재까지의 학습시간을 반영하여 변경.
        - 이 또한 replace를 통해 요소를 현재 변경된 요소로 교체.

        <aside>
        🚨 모든 post edit에 대해서 적용되면 안되기에 해당 edit한 post가 조건에 맞는 post일 경우에만 로직 적용 (focusedPostId → 최근에 작성된 post의 id, recentPost는 현재 사용자가 end일 경우에는 반환되지 않기에 focusedPost를 새로 지정)

        </aside>

### <220505>

> **Tag : version 1.0.9**  <br>
**#39** `Enhancement` : 실시간 학습 시간 기능 추가
>

- **#39**
    - 기존의 문제점 : 학습 시간이 기록은 되었지만, 이를 그저 시간으로만 보여주어서 사용자가 오늘 하루 얼마나 공부를 했는지 직접 계산하거나 놓치는 경우가 발생
    - 해결: 프로그램상으로 자동 계산하여 지속적으로 지금까지 자신이 하루 얼마나 공부를 진행했는지 확인할 수 있도록 설정
        - `getTotalStudyTimes`, `getTimes`를 통해 현재 상황에 맞는 시간 계산 진행
        - 로직 :

          조회 시간 기준 (home으로 이동)

            1. 04:00 - 24:00
                - 그 member의 가장 최신 글 search
                1. 해당 member의 최신글이 당일 4시 이후 create 되었다
                    - 가져와서 times 계산
                2. 당일 4시 이전 → 어제 글이라 판단
                    - “공부를 시작하지 않았습니다” 출력
            2. 00:00 - 04:00
                - 그 member의 가장 최신 글 search
                1. 해당 member의 최신글이 당일 create
                    - 가져와서 times 계산
                2. 전날 create
                    1. 04시이후 create
                        - 가져와서 times 계산
                    2. 04시 이전 create
                        - “공부를 시작하지 않았습니다” 출력

### <220504>
> **Tag : version 1.0.8** <br>
**#2** `Bug` : post edit에서 시간 변경 시 반영되지 않는 bug 수정, comment 삭제, post 삭제 시 모달 유지
>

- **#2**
    - 기존의 문제점: post를 edit할 때, main(home) 화면의 title, introduce는 수정되지만, 변경된 time이 반영되지 않음. → 또 times들에 대해 replace 해줄 필요 있음
      또한, comment 삭제, post 삭제도 **RestAPI**로 교체 필요
    - 해결: 기존 main 화면의 해당 post의 times에 정보를 나타내고 있는 요소를 빈 요소로 replace하고 변경된 시간에 따른 요소들을 append하는 방식으로 진행
        - 시간에 따른 보여주는 로직은 기존의 로직과 동일

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

        - comment, post 삭제 요청을 RestAPI로 교체

            ```jsx
            function post_delete_check(id) {
                var check = confirm("정말 삭제하시겠습니까?");
                if (check){    //확인
                    alert("삭제가 완료 되었습니다.")
                    $.ajax({
                        url: 'posts/delete/' + id,
                        type: "DELETE"
                    })
                        .done(function () {
                            location.reload();
                        })
                }else{   //취소
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
            
                var check = confirm("정말 삭제하시겠습니까?");
            
                if (check){ //확인
                    $.ajax({
                        url: 'comment/delete/' + commentId,
                        type: "DELETE",
                        data: deleteForm
                    })
                        .done(function (fragment) {...}
            }
            ```

            - post를 삭제할 때는 삭제 후 바로 home으로 가면 되지만, comment는 삭제해도 모달을 유지하며 해당 삭제를 반영한 상태의 comment list를 새로 보여줄 필요가 있기에 삭제 후 comment list(삭제된 후의 comment list)를 fragment로 받아와 replace 진행

### <220503>

> **#2** `Enhancement` : Post 수정 시 모달 유지 <br>
**#28** `Enhancement` : 이미지 Upload 시 API로 보낼 수 있도록 설정 <br>
>

- **#2**
    - 기존의 문제점: comment 작성 시 에 모달을 유지할 필요가 있다고 판단하여 해당 기능을 적용, 하지만 post 또한 수정 후 submit을 누르면 변경된 사항을 확인하기 위해서는 home으로 돌아가야 되고, modal이 내려가는 현상 발생 → edit 후 바로 그 모달 자체에서 확인할 수 있도록 설정 필요 (comment에서의 문제와 유사)
    - 해결: 이 또한 comment 작성 시 동작하는 로직과 유사하게 진행 → Ajax Web API 요청을 통해 post eidt을 Post하고 해당 결과 값을 fragment된 html로 받아와 기존 fragment를 변경하는 방식으로 진행 (새 글 작성은 모달이 아닌, 기존 폼으로 보여주도록 설정)
        - post는 get, post 모두 Ajax api로 요청 (edit 버튼을 누를 경우 또한 모달안에서 수정 및 수정 완료 후 변경 사항 확인을 위함)
        - `@GetMapping("/posts/edit/{id}")` 을 Ajax api로 요청하도록 설정 (게시글 수정 폼)

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

            - 똑같이 기존 내용을 fragment로 받아 온 editForm html(`return "post/editPostForm :: #postEdit_container";`)로 교체
            - 교체 후 markdown editor 추가.
            - 상단으로 스크롤 자동 이동
        - `@PostMapping("/posts/edit/{id}")` 을 Ajax api로 요청하도록 설정 (게시글 수정)

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

            - form에 있는 데이터들을 반영하기 위해 form의 입력된 모든 데이터를 serializeArray를 통해 받아옴
            - 여기서 simplemde를 사용하면서 content의 내용이 serializeArray에 담겨오지 않아 직접 simplemde에 입력된 값을 form안의 content에 넣어줌
            - 이후는 위의 comment 변경 로직과 동일하게 진행 (ajax api, fragment replace, setAttribute(id), recentpost, ...)
- **#28**
    - 기존의 문제점 : 이미지를 iframe의 form으로 upload 진행. → 비효율적인 방법, Rest API를 통해 해결 가능
    - 해결 :  Ajax Web API 요청의 Post를 이용하여 업로드를 하고 반환 값으로 해당 Image 객체 (value: imgSrc, imgName)를 Json 형식으로 받아 이를 이용하여 변경된 사진 정보를 출력

        ```java
        @ResponseBody
        @PostMapping("/posts/edit/{id}/img")
        public Image postImgEdit(@PathVariable("id") Long id, @RequestParam("file") MultipartFile[] files) throws IOException {
            MultipartFile file = files[0];
            Image image = s3Service.upload(file);
            postService.editPostImg(id, image);
            return image; // image 객체를 Json으로 반환
        }
        ```

        ```jsx
        function image_upload(id){
        
            var formData = new FormData();
            var image = $("#file")[0].files[0];
        
            formData.append( "file", image);
        
            if(image == null){
                alert("파일을 선택해주세요");
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

        - 이미지 업로드 후 객체를 Json으로 반환해 줌
        - 이미지와 같은 file 데이터는 formdata로 한번 묶어줘서 보내주는 것이 안전함
        - 또한 file 데이터를 보낼 때는 항상 `processData : false`, `contentType : false` 주의

### <220502>

> **#25** `Enhancement` : 누적공부시간 통계 대시보드
>


- **#25**
    - 기존의 문제점 : 내가 일주일동안 얼만큼 공부를 하고 있는지에 대한 정보가 없어, 동기부여가 부족. 일주일 동안의 누적 공부시간 통계량을 대시보드를 통해 사용자들끼리 비교하여 보여주면 동기부여하는 데에 좋은 효과가 있을 것이라고 판단
    - 해결 : 분석 툴은 파이썬이 더 익숙하기에 파이썬의 matplot을 통해 시간을 계산하고 ploting. 그 후 해당 결과를 이미지로 저장하여 S3에 저장하고, 웹에서는 해당 이미지를 불러와 보여주는 형식으로 진행. (Side Widget의 Statics, Nav bar의 Data 부분)
    - 추후 해결 : D3.js 와 같은 툴을 사용하여 좀 더 시각적 효율성을 극대화할 예정

### <220501>

> **#2** `Enhancement` : Comment 작성 시 모달 유지
>

- **#2**
    - 기존 문제점 : Post 클릭 시 생성되는 모달에서 댓글을 작성할 시 작성 후 submit 을 클릭할 경우 modal 이기에 작성된 댓글을 확인하려면 home으로 reload됨. → 작성 후 현재 모달을 유지하면서 작성된 댓글목록만을 reload할 필요가 있음
    - 해결 : Ajax Web API 요청을 통해 comment를 post하고 해당 결과 값을 fragment된 html로 받아와 기존 fragment를 변경하는 방식으로 진행
        - `@PostMapping("/comment/api/new/{id}")` 를 ajax의 post 요청으로 mapping

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

            - fragment는 모달의 일부분인 commentList 를 replace하기 위한 렌더링된 html 파일
            - 즉, post가 완료되면 새로 갱신된 해당 Post의 comment list (방금 작성된 댓글을 추가한 댓글 리스트)를 렌더링한 html 파일을 받아와 이를 기존의 comment list와 교체. → 작성된 댓글을 url reload 필요 없이 바로 확인 가능
        - fragment는 thymeleaf에서 제공하는 html 일부만을 반환해주는 기능 사용 : `return "components/commentList :: #commentPart";` → 해당 id를 가진 요소만 반환해줌.
        - 반환된 fragment로 기존 요소를 교체한 후, id를 새로 할당 → 모달이기에, 다른 post들의 commentPart와 구분하기 위함.

            ```jsx
            var commentPart =document.getElementById("commentPart");
            commentPart.setAttribute("id", "commentPart"+id.toString());
            ```

            <aside>
            🚨 controller의 각 mapping return 의 fragment 값에 id를 넣으면 안되는 이유

            1. return "components :: #commentList" + id.toString(); -> 될 것 같지만 사실 지금 우리는 각 요소의 id 값에 thymeleaf로 값을 할당 하고 있음 (th:id="${post.id}")
            2. 하지만 이는 렌더링 후에 값이 할당되어 완성 되는 것.
            3. 즉 controller에서 viewtemplate을 통한 html fragment search 시점에는 값이 할당되어 있지 않다는 것.
            4. 그러므로 id 값으로 return을 주어 viewtemplate을 통해 해당 fragment를 찾아낼 수 없음
            5. 이는 id값을 추후에 변경하면 됨 ( `setAttribute(”id”, ... )` )
            </aside>

        - 또한, 변경된 사항을 modal 뿐만아닌, home의 해당 post의 comment 개수에도 반영해줄 필요가 있음 → 해당 post id를 가진 post의 comment 개수를 표현하는 요소를 replace 해줌
        - 또한, recent post라고 현재 자신이 작성 중(공부를 end하지 않은 상태의 post)인 post를 보여주는 기능이 있기 때문에, 댓글 작성이 이루어진 post가 그 recent post와 동일할 경우 recent post 또한 위의 과정을 적용 (comment 개수 변경)


### <220427>

> **Tag : version 1.0.7** <br>
**#28** `Enhancement` : Post Image 변경 시  변경된 이미지 띄워주기 <br>
**#31** `Back` : 희망직무 선택지 부여 (선택지에 원하는 직무가 없을 시 직접 추가) <br>
**#15** `Front` : OG 태그 부여 <br>
>


- **#28**
    - 기존 문제점 : 현재 이미지의 이름만을 알려주고, 현재 이미지 자체를 보여 주진 않았음. 또한, Upload 시 바로 변경이 반영되지 않은 채로 edit을 진행하게 설정함
    - 해결 : 이미지를 띄워주고 Upload 시 바로 변경하여 변경된 이미지를 보여줄 수 있도록 설정 필요
        - Upload 버튼을 누를 시 a 태그를 통해 바로 img upload 로직을 실행하는 것이 아닌, onclick=’upload_check()’을 통해 javascript 코드를 거쳐 진행되도록 설정

            ```jsx
            function upload_check() {
                setTimeout(function(){alert("Upload완료 되었습니다!");location.reload();},500);
            } // 0.5초 후에 자동으로 페이지가 reload 되도록 설정. -> reload 되면 변경된 정보를 띄울 수 있음.
            ```

- **#31**
    - 기존 문제점 : 희망 직무의 선택지를 주지 않아, 정해진 형식 없이 희망 직무를 적게 됨 → 똑같은 직무라도 사용자마다 다른 이름으로 설정하는 경우가 발생
    - 해결 : ViewTemplate 에 jobs 라는 설정된 직무 List를 넘겨줘 datalist로 선택할 수 있도록 설정 + 원하는 직무가 없으면 직접 적을 수 있도록 Input을 text로 설정

        ```java
        public class DesiredJob {
            static final List<String> jobList =  Arrays.asList("백엔드 엔지니어", "데이터 엔지니어", "ML 엔지니어", "프론트엔드 엔지니어");
            public static List<String> getJobList() {
                return jobList;
            }
        }
        ```

        ```html
        <div class="input-group mb-3">
            <input **type="text"** autocomplete="off" class="form-control" list="datalistOptions" th:field="*{desiredJob}" placeholder="희망 직무를 선택하거나 입력하세요!">
            <**datalist** id="datalistOptions" >
                <th:block **th:each="job : ${jobs}"**>
                    <option **th:value="${job}"**>
                </th:block>
            </**datalist**>
        </div>
        ```

- **#15**
    - 기존 문제점 : 링크를 공유해도 정해진 형식 없이 이상한 형태로 공유 화면이 띄워짐
    - 해결 : Og 태그를 통해서 정해진 깔끔한 형식으로 공유될 수 있도록 설정

        ```html
        <meta property="og:title" content="세모공 - 세상의 모든 공부" />
        <meta property="og:url" content="http://semogong.site/" />
        <meta property="og:type" content="website" />
        <meta property="og:image" content="https://semogong-bucket.s3.ap-northeast-2.amazonaws.com/SEMOGONG.jpg" />
        <meta property="og:description" content="TIL 공유 사이트, 오늘 한 공부를 공유해보세요!" />
        ```

### <220425>

> **Tag : version 1.0.6** <br>
**#32** `Enhancement` : Comment 작성 시간 표시 <br>
**#35** `Bug` `Back` : State 변경 시 오류 발생 <br>
**#36** `Front` : 글 및 댓글 삭제 시 Confirm <br>
>


- **#32**
    - 기존 문제점 : 댓글을 작성했을 때, 해당 댓글이 언제 달린 건지 구분이 안될 때가 있었음. → 댓글이 언제 달린 것인지 표기 필요
    - 해결 : 댓글이 달린 시간 표기
        - 로직 : CommetViewDto에서 댓글이 달린 시간인 CreateDateTime과 현재 시간인 LocalDateTime.now()의 초차이 를 Duration.between을 통해 계산하여 해당 DTO에 저장하고 veiwTemplate에 보내준 후, ViewTemplate(Thymeleaf) 안에서

            1. 60분이 지나지 않았으면 ‘x분 전’으로
            2. 60분이 지났고 24시간이전이면  ‘x시간 전’으로
            3. 24시간이 지났으면 ‘x일 전’으로 표기

        - 이때, Thmeleaf에서 Math.round를 적용하기 위해 ThymeMath Component를 생성하여 스프링 빈으로 등록 → 빈으로 등록된 Object는 Thymeleaf 에서 `@thymeMath` 형태로 사용 가능.
- **#35**
    - 기존 문제점 : State 변경 오류 시 별 다른 오류 처리 없이 그냥 홈으로 돌아가게 설정 → 변경되었는지, 오류가 난지 사용자가 인지하기 어려움
    - 해결 : State 변경 오류 시 alert을 통해 잘못된 state 변경이라고 알려줌
        - 로직 : ViewTemplate(Thymeleaf) 에서 현재 state 값을 받아온 후 <br>
          ( Studying -> Studying / Breaking, End -> Breaking / End -> End ) 와 같은 변경 오류에 대해서 if 절을 통해 alert을 띄울 수 있도록 설정

- **#36**
    - 기존 문제점 :  글 및 댓글 삭제 시 “삭제” 버튼을 누르면 그냥 바로 삭제 됨 → 사용자가 실수로 삭제를 누르는 경우에 대한 처리가 필요.
    - 해결 :
        - 로직 : 바로 a 태그로 삭제 URL로 요청을 보내는 것이 아닌, button으로 바꾼 후 onclick으로 javascript delete_check 함수를 통해 confirm할 수 있도록 설정

          detele_check : 인자로 삭제하고자 하는 Entity의 id를 받아오고 confrim 진행. confirm에서 확인을 누를 시 location.href 를 통해 delete 요청을 진행. 취소를 누를 시 아무동작하지 않도록 설정

### <220420>

> **Tag : version 1.0.5** <br>
#10 `Back` `Front`: State마다 구분할 수 있도록 고유 색 부여 필요 <br>
#11 `Back` : 각 Post 당 comment 개수를 인지할 수 있도록 설정 필요 <br>
#16 `Back` : Side Widget의 All Members를 Studying→Breaking→End 순으로 정렬 필요 <br>
#18 `Front` : 테마 글꼴 설정 필요 <br>
#20 `Front` : 통일된 Header 생성 필요 <br>
#24 `Back` `Front` : 각 Post에 CreateDate 표기 필요 <br>
>


- **#10**
    - 기존의 문제점 : State마다 구분이 안되어 있어, 시각적으로 효율적이지 못함. 즉각 인지하기가 어려움.
    - 해결 : 각 State에 맞는 색을 부여 (thymeleaf의 if 문법을 사용)
- **#11**
    - 문제점 : comment를 달 수는 있지만, 각 post에 comment가 달린지 안달린지 외부에서는 확인 불가
    - 해결 : ViewTemplate에게 DTO를 보내줄 때, DTO안에 Comment Count라는 변수를 추가하여 각 post 당 달린 comment의 개수를 표출해줌. (각 post 우상단에 표현)
- **#16**
    - 기존에는 이름순으로 정렬했던 All members를 Studying → Breaking → End 순으로 정렬하여 표현
    - 해결 로직 :
        - 일단 모든 멤버를 DB에서 가져온 후 Memory 단에서 직접 정렬 실행. → O(N)
        - 빈 리스트를 생성하여
            1. 모든 멤버를 돌면서 Studying인 멤버 추가
            2. 그 후 또 모든 멤버를 돌면서 Breaking인 멤버 추가
            3. 그 후 마지막으로 모든 멤버를 돌면서 End인 멤버 추가
- **#18**
    - Body에 font-family를 NanumSquareRound 적용 → 기존보다 훨씬 보기 좋아짐
- **#20**
    - 통일된 header,footer를 만든 후 모든 html이 fragment로 받아올 수 있도록 설정
    - edit,create 화면 같은 경우 bootstarp 이외의 css를 받으면 안되므로 addheader를 추가로 생성하여 home.html이 fragment로 받도록 설정
- **#24**
    - Post Entity에 FormatCreateDate를 추가하여 LocalDateTime class가 아닌 String으로 DB에 해당 날짜를 원하는 format 형식으로 저장할 수 있도록 설정 (“April 14, 2022” 형식으로 저장)
    - 그 후 DTO에도 FormatCreateDate를 추가하여 ViewTemplate에 값을 전달할 수 있도록 설정
    - 각 post 좌상단에 표기


### <220418>

> **#14** `Back` : Member가 이미 그날 공부를 종료했지만, 다시 공부를 시작할 경우에 대한 조치 <br>
> **#21** `Front` : 새로운 Member 가입과 기존 Member 정보 수정 html이 분리되지 않음 <br>
> **#7** `Bug` : 로그인이 안된 상태에서 post에 댓글을 달 경우 오류 발생 <br>
> **#6** `Back` : Posting 삭제 기능 및 삭제 버튼 생성 <br>
> **#4** `Front` : 여러 입력 placeholder 수정 <br>



- **#14**
    - 기존의 문제점
        
        Member가 공부를 종료한 후 다시 공부를 시작하고 싶을 때 공부 기록을 진행하던 글이 불러와지는 것이 아닌, 새로운 글을 작성게끔 설정되어 있었음. 즉, **공부종료를 했지만, 다시 공부를 하는 경우에 대한 예외처리**가 필요했음
        
    - 해결 로직
        
        > 일단 기본적으로 04시를 하루의 시작과 끝으로 설정 (새벽반 고려) <br>
        사용자가 End -> Studying 으로 State를 변경할 시 (공부완료 -> 공부시작)의 상황
        >
  
        1. 사용자가 현재 쓴 글이 존재하는 지 판단 (즉, 신규 회원인지 아닌지 판단)
            - true (쓴 글이 존재)
                1. 사용자가 가장 최근에 작성한 글이 해당 날짜에 작성한 글이면서 작성 한 시간이 04시 이후인지 판단
                    - true => 해당 글을 가져와서 이어 작성할 수 있도록 설정
                    - false
                        1. 현재 State 변경 시점이 04시 이전인지 판단
                            - true
                                1. 그 전날 작성한 글이 존재하는 지, 해당 글이 04시 이후 작성한 글인지 판단
                                    - true => 해당 글을 가져와서 이어 작성할 수 있도록 설정
                                    - flase => 새로운 글 작성
                            - false => 새로운 글 작성
            - false => 새로운 글 작성
- **#21**
    - 추후의 bug가 발생할 것을 대비하여 sign up 화면 과 eidt 화면을 분리
    - createMemberForm.html → createMemberForm.html + editMemberForm.html
- **#7**
    - 로그인이 안된 유저가 Post에 댓글을 달 경우, alert (”로그인 후 댓글을 다실 수 있습니다.”) 창을 띄우도록 설정
- **#6**
    - 임시적으로 DeleteMapping으로 Controller 에 설정한 것이 아닌 GetMapping으로 설정해둠. → 추후 DeleteMapping과 API를 사용하여 해결 예정
    - edit 화면에 post 삭제 버튼 생성
- **#4**
    - placeholder 수정 (도시를 입력하세요. 등 관련없는 값들이 default값으로 주어 졌었음)