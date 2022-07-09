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

## Project(SEMOGONG, Capstone, ...) 개발 일지

- Issue 를 바탕으로 개발 진행.
- Issue Labels
    - `Bug` : 구현한 기능들 중에서 서버에 올렸는데 작동하지 않거나 오류가 생김
    - `Back` : Back-End Engineer 만을 위한 이슈
    - `Front` : Front-End Engineer 만을 위한 이슈
    - `Enhancement` : 새로운 기능이 필요한 경우, front back 모두가 손봐야할 이슈

<hr>

### <220618> ~ <220630>
- **Spring MVC 집중 공부! (강의 및 개인 학습으로 인해 따로 개발은 진행하지 않음)**
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
- **Spring MVC 집중 공부! (강의 및 개인 학습으로 인해 따로 개발은 진행하지 않음)**
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