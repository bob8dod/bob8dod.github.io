---
layout: post
title:  "SEMOGONG 아키텍쳐 및 코드 설명 (DEMO, Before Release)"
date:   2022-04-23
image:  /posts/semogong.jpg
tags:   Project SpringBoot
---


### 세모공(세상의 모든 공부) - TIL(Today I Learned) 공유 서비스

**사이트 (배포 및 개선 중) → " [SEMOGONG](http://semogong.site/) "** <br>
**개발 Repo → " [Github](https://github.com/Sejong-Talk-With/Semogong) "**

<aside>
💡 <strong> 오늘 한 공부를 공유하며 내가 공부한 시간을 기록해주는 서비스 </strong>
    <ul>
    <li> 사용자의 공부 상태에 따른 공부 시간 자동 기록 </li>
    <li> 공부 기록에 따른 분석 제공 </li>
    <li> 마크다운 에디터를 통해 통일된 TIL 작성 툴 제공 </li>
    <li> 웹서비스 배포를 통한 전반적인 DevOps 운영 </li>
    </ul>
</aside>

<br>
<br>

## Project Setting

![]({{site.baseurl}}/images/posts/post-220422/Untitled 2.png)

- Gradle Project, Java(ver 11), Spring Boot(ver 2.6.5)
- Spring MVC - Spring Web
- Annotation library - Lombok
- View Template - Thymeleaf
- DataBase - H2 Database (SQL), Spring Data JPA

- 스프링 부트 라이브러리
    - spring-boot-starter-web
        - spring-boot-starter-tomcat: 톰캣 (웹서버)
        - spring-webmvc: 스프링 웹 MVC
    - spring-boot-starter-thymeleaf: 타임리프 템플릿 엔진(View)
    - spring-boot-starter(공통): 스프링 부트 + 스프링 코어 + 로깅
    - spring-boot
        - spring-core
    - spring-boot-starter-logging
    logback, slf4j

- 테스트 라이브러리
    - spring-boot-starter-test
        - junit : 테스트 프레임워크
        - mockito : 목 라이브러리
        - aseertj : : 테스트 코드를 좀 더 편하게 작성하게 도와주는 라이브러리
        - spring-test : 스프링 통합 테스트 지원

- Gradle (build.gradle)
    
    ```python
    plugins {
    	id 'org.springframework.boot' version '2.6.4'
    	id 'io.spring.dependency-management' version '1.0.11.RELEASE'
    	id 'java'
    }
    
    group = 'Talk_with'
    version = '0.0.1-SNAPSHOT'
    sourceCompatibility = '11'
    
    configurations {
    	compileOnly {
    		extendsFrom annotationProcessor
    	}
    }
    
    repositories {
    	mavenCentral()
    }
    
    dependencies {
    	implementation 'org.springframework.boot:spring-boot-starter-data-jpa' //spring-data-jpa
    	implementation 'org.springframework.boot:spring-boot-starter-thymeleaf' //veiw template
    	implementation 'org.springframework.boot:spring-boot-starter-web' // spring-web
    	implementation 'org.springframework.boot:spring-boot-starter-validation' // for validation
    	implementation 'org.springframework.boot:spring-boot-devtools' // for easy rebuilding
    	implementation 'org.springframework.boot:spring-boot-starter-security' // spring-security
    	implementation 'org.springframework.security:spring-security-test' // spring-security-test
    	implementation 'com.atlassian.commonmark:commonmark:0.17.0' // for using markdown (SimpleMDE)
    	implementation 'org.apache.commons:commons-lang3:3.12.0' // for easy using 
    	compileOnly 'org.projectlombok:lombok' // for annotation
    	implementation 'com.github.gavlyukovskiy:p6spy-spring-boot-starter:1.8.0' //for tracking sql query
    	runtimeOnly 'mysql:mysql-connector-java' // for AWS RDS
    //	implementation 'com.h2database:h2' // for Test _ in Local DB
    	annotationProcessor 'org.projectlombok:lombok' // for annotation
    	testImplementation 'org.springframework.boot:spring-boot-starter-test' // spring-test
    
    	/*
    	*  AWS SDK (Using S3)
    	*/
    	implementation platform('com.amazonaws:aws-java-sdk-bom:1.11.1000')
    	implementation 'com.amazonaws:aws-java-sdk-s3'
    }
    
    tasks.named('test') {
    	useJUnitPlatform()
    }
    ```
    

- application.yml
    
    ```yaml
    spring:
      datasource: //DB Setting (AWS RDS)
        url: jdbc:mysql://${DB_URL}:3306/semogong
        username: ${DB_NAME}
        password: ${DB_PASSWORD}
        driver-class-name: com.mysql.cj.jdbc.Driver
    
      jpa:
        hibernate:
          ddl-auto : update
        properties:
          hibernate:
            format_sql : true
    
      servlet:
        multipart:
          maxFileSize: 10MB
          maxRequestSize: 20MB
    
    cloud:
      aws:
        credentials:
          accessKey: ${S3_ID}
          secretKey: ${S3_PASSWORD}
        s3:
          bucket: ${S3_NAME}
        region:
          static: ${S3_REGION}
        stack:
          auto: false
    
    logging:
      level:
        org.hibernate.SQL: debug
    ```
    
<br>
<br>

## Domain Analysis

### 요구사항 분석

- 기능 목록
    1. 회원 기능
        - 회원 등록
        - 회원 정보 수정
        - 회원 공부 상태 변경 (Studying, Breaking, End)
        - 모든 회원의 현재 State 조회
    2. 글 기능
        - 글 등록
        - 글 수정 및 삭제
        - 글 조회 (현재 로그인된 회원의 작성 중인 글 조회)
    3. 댓글 기능
        - 댓글 등록
        - 댓글 삭제
    4. 기타 요구사항
        - 회원의 공부 시간이 해당 글에 기록 필요. (공부 State에 따라 자동으로)
        - 또한, 회원이 공부 시작 시 바로 자동으로 Posting 할 수 있도록 설정 필요
        

### 도메인 모델과 테이블 설계 및 분석

- 도메인 모델 설계 및 분석

  ![]({{site.baseurl}}/images/posts/post-220422/Untitled 3.png)
    
    1. 회원과 글의 관계
        - 회원은 여러 글을 작성할 수 있다.
        - 회원이 Study 버튼을 누르면 자동으로 새로운 글이 생성된다.
    2. 글과 댓글의 관계
        - 글은 여러 댓글을 가지고 있다.
        - 글이 삭제되면 댓글 또한 같이 삭제되는 관계를 가지고 있다. `cascade={CascadeType.*REMOVE*}`
    3. 회원과 댓글의 관계
        - 회원은 여러 댓글을 작성할 수 있지만, 현재로써는 회원 안에 댓글을 참조 하지 않고 있기에 OneToMany가 아닌 OneToOne으로만 연관관계를 설정 (추후 변경 가능)

- 엔티티 설계 및 분석

  ![]({{site.baseurl}}/images/posts/post-220422/Untitled 4.png)
    
    - 회원(Member)
        - 로그인 정보 : login_id, password, role
        - 개인 정보 : name, nickname, desiredjob(희망 직무), introduce(자기 소개), state(공부 상태), links(링크 리스트, @ElementCollection), image(프로필 사진, Embedded Type)
        - **연관 관계** : posts(글 리스트, @OneToMany)
    - 글(Post)
        - 글 정보 : title, introduce(오늘의 한 줄), times(시간 리스트, 공부 시간 기록 정보, @ElementCollection), state(공부 상태), createTime(작성 시간), image(썸네일 사진, Embedded Type)
        - 글 내용(MarkDown) : content(마크다운 문법으로 저장), times(시간 리스트, @ElementCollection), html(content를 html형식으로 변환 후 저장)
        - **연관 관계** : member(작성자, @ManyToOne), comments(댓글 리스트, @OneToMany)
    - 댓글(Comment)
        - 댓글 정보 : createTime(작성 시간), content(일반 Text)
        - **연관 관계** : member(작성자, @OneToOne), post(댓글을 단 글, @ManyToOne)
    - 이미지(Image)
        - 값 타입(Embedded Type), imgSrc(이미지 저장 경로), imgName(이미지 이름)
        - 회원(Member)와 글(Post)에 사용
    
    <aside>
    🚨 <strong> 여기서 사실 회원이 글을 참조할 필요는 없다! </strong> <br> 글에서 회원에 해당하는 애들만 가져와도 되니깐. 하지만 아직 초기 개발 단계라 회원이 post를 참조할 수 있도록 설정. 추후에 변경 필요. (또한, post와 comment 관계도 마찬가지)
    
    </aside>
    

- 테이블 설계 및 분석

  ![]({{site.baseurl}}/images/posts/post-220422/Untitled 5.png)
    
    - MEMBER, POST : Image 임베디드 타입 정보가 회원 테이블에 그대로 들어감
    - LINK_LIST, POST_TIMES : ElementCollection 으로 인해, 테이블이 따로 설정된 것을 확인 가능

- 엔티티 클래스 속 주요 메서드
    
    > Entity 생성 및 수정은 되도록이면 Entity안에서 이루어지도록 설정
    Setter는 되도록이면 열어두지 않도록 설정 (변경이나 저장 자체를 추후에 추적하기 편하게 만들기 위함)
    >    

    1. 회원 (Member Class)
        - 생성 메서드
            
            생성할 때 필요한 값들만을 가지고 생성 (이때, 이미지는 추후 정보 수정 시 업로드가 가능하기에 따로 빼 둠) (추후 변동 가능성 있음)
            
            ```java
            public static Member createMember(String loginId, String password, String name, String nickname, String desiredJob, Image image, String introduce){
                 Member member = new Member();
                 member.loginId = loginId;
                 member.password = password;
                 member.name = name;
                 member.nickname = nickname;
                 member.desiredJob = desiredJob;
                 member.introduce = introduce;
                 member.state = StudyState.END;
                 return member;
            }
            public void setImage(Image image) {
                this.image = image;
            }
            ```
            
        - 수정 메서드
            
            수정에 사용되는 값들만을 가지고 생성 (로그인 정보는 수정하지 않음, 상태는 editForm이 아니여도 수시로 change할 수 있기에 따로 빼 둠)
            
            ```java
            public void changeState(StudyState state){
                this.state = state;
            }
            
            public void edit(MemberEditForm memberForm) {
                this.name = memberForm.getName();
                this.nickname = memberForm.getNickname();
                this.desiredJob = memberForm.getDesiredJob();
                this.introduce = memberForm.getIntroduce();
                this.links = memberForm.getLinks();
            }
            ```
        
    2. 글 (Post Class)
          - 연관관계 메서드
            
              member와 양방향 연관 관계 : Post가 생성되면 Member의 post list에 더해줄 필요가 있음. 연관관계 메서드는 생성메서드 안에서만 사용되도록 private으로 설정 (추후 변경 가능 사항)
            
              ```java
              private void setMember(Member member){
                  this.member = member;
                  member.getPosts().add(this);
              }
              ```
            
          - 생성 메서드
            
              연관관계를 이용하며 생성할 필요가 있음 (Member와의 연관관계), 추가로 Member와 마찬가지로 Image는 따로 Upload하기에 빼둠
            
              ```java
              public static Post createPost(Member member, String title, String introduce, String content, String html ,LocalDateTime time){
                  Post post = new Post();
                  post.setMember(member);
                  post.title = title;
                  post.introduce = introduce;
                  post.content = content;
                  post.html = html;
                  DateTimeFormatter timeFormatter = DateTimeFormatter.ofPattern("HH:mm");
                  post.times.add(time.format(timeFormatter));
                  post.createTime = time;
                  post.state = StudyState.STUDYING;
                  return post;
              }
              public void setImage(Image image) {
                  this.image = image;
              }
              ```
            
          - 수정 메서드
            
              수정 가능한 요소들에 대해서만 수정 메서드 실행, 추가로 state는 Member와 같이 수시로 변할 수 있기에 따로 메서드를 정의하고 이 state에 따라 post의 time list가 변하기에 이 또한 따로 정의
            
              ```java
              public void edit(PostEditForm postEditForm) {
                  this.title = postEditForm.getTitle();
                  this.introduce = postEditForm.getIntroduce();
                  this.content = postEditForm.getContent();
                  this.html = postEditForm.getHtml();
                  this.times = postEditForm.getTimes();
              }
            
              public void addTime(LocalDateTime time) {
                  DateTimeFormatter formatter = DateTimeFormatter.ofPattern("HH:mm");
                  String timeString = time.format(formatter);
                  this.times.add(timeString);
              }
            
              public void editState(StudyState state) {
                  this.state = state;
              }
              ```
            
    3. 댓글 (Comment)
        - 연관 관계 메서드
            
            Post와 양방향 연관 관계를 가지고 있음. 즉, commet가 생기면 이에 따른 post의 comment list에도 추가해 줄 필요가 있음. 연관관계 메서드는 생성메서드 안에서만 사용되도록 private으로 설정 (추후 변경 가능 사항). 
            
            ```java
            private void setPost(Post post){
                this.post = post;
                post.getComments().add(this);
            }
            ```
            
        - 생성 메서드
            
            생성 될 때 연관관계를 사용해서 생성
            
            ```java
            public static Comment makeComment(String content, Post post, Member member, LocalDateTime createTime){
                Comment comment = new Comment();
                comment.content = content;
                comment.setPost(post);
                comment.member = member;
                comment.createTime = createTime;
                return comment;
            }
            ```

<br>
<br>

## Application Implementation

### 구현 요구사항

- 기능 목록
    1. 회원 기능
        - 회원 등록
        - 회원 정보 수정
        - 회원 공부 상태 변경 (Studying, Breaking, End)
        - 모든 회원의 현재 State 조회
    2. 글 기능
        - 글 등록
        - 글 수정 및 삭제
        - 글 조회 (현재 로그인된 회원의 작성 중인 글 조회)
    3. 댓글 기능
        - 댓글 등록
        - 댓글 삭제
    4. 기타 요구사항
        - 회원의 공부 시간이 해당 글에 기록 필요. (공부 State에 따라 자동으로)
        - 또한, 회원이 공부 시작 시 바로 자동으로 Posting 할 수 있도록 설정 필요

### 아키텍쳐 구성

![]({{site.baseurl}}/images/posts/post-220422/Untitled 6.png)

- 계층형 구조 사용
    - controller, web: 웹 계층
    - service: 비즈니스 로직, 트랜잭션 처리
    - repository: JPA를 직접 사용하는 계층, 엔티티 매니저 사용
    - domain: 엔티티가 모여 있는 계층, 모든 계층에서 사용
- 패키지 구조
    
    Talk_with.semogong
    
    - config : security관련 설정
    - controller : web 계층 관련 구현
    - domain : Entity, Dto, Embedded Type, Enum Class 정의 및 구현
    - service : Business Logic, Transaction 정의 및 구현
    - repository :  EM(JPA), DB connect 정의 및 구현

<aside>
💡 <strong> 개발 순서 </strong>: 서비스, 리포지토리 계층을 개발하고, 테스트 케이스를 작성해서 검증(테스트 코드는 생략), 마지막에 웹 계층 적용

</aside>

### 회원 기능 개발

> 구현 기능 
> <ul> <li> 회원 등록 </li>
> <li> 회원 정보 수정 </li> </ul>
> 구현 순서
> <ul> <li> 회원 레포지토리 개발 </li>
> <li> 회원 서비스 개발 </li>
> <li> 회원 컨트롤러(웹 계층) 개발 </li> </ul>


1. 기능 설명
    - 회원 등록
        - spring security 사용 (로그인, 회원가입 로직)
        - 회원가입 시  “ID, PassWord, 이름” 을 필수 값으로 입력하도록 설정
        - submit (회원가입 버튼) 을 누르는 순간, 회원 정보가 DB에 들어갈 수 있도록 설정 (password는 암호화하여 저장)
    - 회원 정보 수정
        - 여기서 이미지를 업로드할 수 있도록 설정 (이미지 업로드 후 )
        - 수정 시 기존에 어떤 값을 입력했었는지 보여준 상태에서 수정이 가능하도록 설정
        - JPA의 변경 감지 기술 사용 (dirty checking, not merge)
2. 회원 레포지토리, 서비스, 컨트롤러
    - 회원 레포지토리 (MemberRepository)
        - JPA를 직접 사용하여 Member Entity와 관련된 Table에 접근하는 계층
        - DI(Dependency Injection) : EntityManager(JPA)
        - `public void save(Member member)` : 회원 저장. `em.persist` 사용
        - `public Member findOne(Long id)` : id를 통해 하나의 회원 조회. `em.find` 사용
        - `public List<Member> findAll()` : 모든 회원 조회. jpql 사용
        - `public Member findByName(String name)` : 이름을 통해 하나의 회원 조회. jpql, where문 사용
        - `public Member findByLoginId(String loginId)` : LoginId로 회원 조회. jpql, where문 사용
    - 회원 서비스 (MemberService)
        - Business Logic, Transaction 정의 및 구현
        - DI(Dependency Injection) : 회원 레포지토리(MemberRepository)
        - **@Transactional(readOnly = true)**
            - `public Member findOne(Long id)` : id를 통해 하나의 회원 조회, 리포지토리 단순 위임
            - `public List<Member> findAll()`
                - 전체 회원 조회
                - 리포지토리의 `findAll` 을 통해 전체 회원을 조회한 후 비지니스 로직 사용
                - 비지니스 로직 : 전체 회원을 Memory단에서 StudyState에서 Studying, Breaking, End 순으로 정렬하여 반환
            - `public Member findByLoginId(String loginId)` : LoginId를 통해 회원 조회, 리포지토리 단순 위임
            - `public StudyState checkState(Long memberId)` : 현재(Login 된) 회원의 state 조회, 리포지토리의 `findOne` 사용, 해당 멤버를 가져온 후 state 조회
        - **@Transactional**
            - `public Long save(Member member)`
                - 회원 저장 (레포지토리의 `save` 사용)
                - 비밀번호는 `BCryptPasswordEncoder` 를 이용하여 암호화 하여 DB에 저장
            - `public void changeState(Long memberId, StudyState state)`
                - 현재(Login 된) 회원의 state 변경, 변경 감지 사용
                - 리포지토리의 `findOne` 사용
                - 인자로 받는 state로 현재 조회한 회원의 state를 변경
            - `public void editMember(Long id, MemberEditForm memberEditForm)`
                - 현재(Login 된) 회원의 정보 변경, 변경 감지 사용
                - 리포지토리의 `findOne` 사용
                - Member Entity 안의 edit method를 사용하여 변경 실행. Entity의 순수성 유지 (Setter를 통해 변경하지 않음)
                    - MemberEditForm의 값을 Member에 적용하여 변경 감지 실행
            - `public void editMemberImg(Long id, Image image)`
                - 현재(Login 된) 회원의 이미지 업로드, 변경 감지 사용
                - 이미지 업로드는 다른 값들과는 독립적으로 업로드 되고, 변경되기에 Setter를 통해 변경
    - 회원 컨트롤러(MemberController)
        - 웹 계층 구현, DTO 사용
        - DI(Dependency Injection) : 회원 서비스(MemberService), AWS S3 서비스(S3Service)
        - `@GetMapping("/members/signup")`
            - 회원 가입 폼
            - Model Attribute :
                - MemberForm DTO : 입력 값을 받아 오고, Valid 또한 실행
        - `@PostMapping("/members/signup")`
            - 회원 가입
            - MemberForm DTO를 통해 Valid 진행.
            - 받아 온 MemberForm의 값을 Member Entity에 위임. (Entity 안의 createMember method 사용, role을 user로 설정(Spring Security))
            - 그 후 회원 서비스의 `save` 를 통해 Entity를 DB에 저장.
        - `@GetMapping("/members/edit/{id}")`
            - 회원 정보 수정 폼
            - 수정할 Member의 Id를 URL 경로를 통해서 받음
            - 해당 Id를 통해 회원 서비스의 `findOne` 을 통해 Member Entity를 가져옴
            - Model Attribute :
                - MemberEditForm DTO : 해당 Member Entity의 값들을 위임한 후 ViewTemplate에 전달. Valid 진행
        - `@PostMapping("/members/edit/{id}")`
            - 회원 정보 수정
            - 수정 시 변경된 link list를 받아 와 빈 링크값들을 제거
            - 회원 서비스의 `editMember` 를 이용하여 회원 수정
        - `@PostMapping("/members/edit/{id}/img")`
            - 회원 이미지 업로드 및 수정
            - 회원 이미지는 다른 Form들과 같이 동시에 저장되고 수정될 수 없기에 독립적으로 진행하도록 설정
            - `@RequestParam("file") MultipartFile[] files` 를 통해 업로드된 파일을 받아옴 → **MultipartFile file 과 같이 단일로 받아오면 오류!!**
            - AWS S3 서비스의 `upload` 를 통해 S3 Bucket에 업로드
            - 회원 서비스의 `editMemberImg` 를 통해 회원의 이미지 정보 변경

### 게시글 기능 개발

> 구현 기능
> <ul> <li> 게시글 등록 </li>
> <li> 게시글 수정 및 삭제 </li> </ul>
> 구현 순서
> <ul> <li> 게시글 레포지토리 개발 </li>
> <li> 게시글 서비스 개발 </li>
> <li> 게시글 컨트롤러(웹 계층) 개발 </li> </ul>

1. 기능 설명
    - 게시글 등록
        - “새 글 작성” 과 같은 기능으로 글이 등록 되는 것이 아닌, 회원이 End → Study로 State를 변경 할 시 자동으로 새 글 등록(및 작성)
        - 제목 값이 필수로 입력될 수 있도록 설정
        - submit (게시글 제출 버튼) 을 누르는 순간, 게시글 정보가 DB에 들어갈 수 있도록 설정
        - **주의할 점은 첫 등록은 무조건 빈 post, 그 후 바로 그 post를 eidt하는 형식으로 등록 로직을 구성 (공부 시작 시간 기록을 위함)**
    - 게시글 수정 및 삭제
        - 사실상 글 작성은 수정에서만 진행 (가장 먼저 새글 작성은 빈 글이 작성된 상태에서 그 빈 글을 불러 와 수정하는 로직, **공부 시작 시간 기록을 위함**)
        - 수정 시 기존에 어떤 값을 입력했었는지 보여준 상태에서 수정이 가능하도록 설정
        - JPA의 변경 감지 기술 사용 (dirty checking, not merge)
2. 게시글 레포지토리, 서비스, 컨트롤러(웹 계층)
    - 게시글 레포지토리 (PostRepository)
        - JPA를 직접 사용하여 Post Entity와 관련된 Table에 접근하는 계층
        - DI(Dependency Injection) : EntityManger (JPA)
        - `public void save(Post post)` : 게시글 저장, `em.persist` 이용
        - `public Post findOne(Long id)` : Post id를 통한 게시글 단일 조회, `em.find` 이용
        - `public List<Post> findAll()` : 게시글 다건 조회. jpql 이용
        - `public Optional<Post> findByMember(Long memberId)`
            - Member Id에 따른 최근 게시글 단일 조회.
            - jpql, where, order by문 사용 (작성 날짜로 sort)
            - `.getResultList().stream().findAny()` 를 통해 가장 최신 글 하나만 찾아 올 수 있도록 설정
            - Optional<T>을 통해 Null 값에 대해서도 다룰 수 있도록 처리
        - `public List<Post> findByPaging(Integer offset)`
            - 게시글 페이징 다건 조회.
            - jpql 사용
            - `.setFirstResult(Integer.*parseInt*(offset.toString()))` , `.setMaxResults(12)` 를 통해 페이징 기능 구현
        - `public void deleteOne(Post post)` : 게시글 단일 삭제, `em.remove` 이용 (관련 Entity인 Comment도 같이 삭제)
    - 게시글 서비스 (PostService)
        - Business Logic, Transaction 정의 및 구현
        - DI(Dependency Injection) : 회원 리포지토리(MemberRepository), 게시글 레포지토리(PostRepository)
        - **@Transactional**
            - `public Long save(Long memberId, LocalDateTime createTime)`
                - 게시글 저장, 비지니스 로직 포함
                - 빈 post를 생성하여 해당 post의 membe와 공부중 click했을 때의 시각만을 기록한채 post를 저장 → 공부 중 click 시점의 시간을 기록하기 위해 설정한 로직 (그 후 바로 edit하게 설정)
                - Post Entity의 `createPost` method 사용, 게시글 리포지토리의 `save` 이용
            - `public void edit(PostEditForm postEditForm)`
                - 게시글 수정 (게시글 첫 작성 포함 - 첫 작성 시 빈 Post를 생성한 후 기본 정보만을 가진 채 바로 수정할 수 있도록 로직 설정)
                - PostEditForm DTO 를 통해 입력된 값들을 바탕으로 Post Entity에 값을 위임 (Post Entity의 edit method 이용) , 변경 감지 사용
                - 게시글 리포지토리의 `findOne` 이용, Post Entity의 `eidt` method 사용
            - `public void addTime(Long memberId, LocalDateTime time)`
                - 상태 변화에 따른 post 시간 추가
                - member Id를 통해 현재 멤버의 최신 글을 가져와 시간 추가, 변경 감지 사용
                - 게시글 리포지토리의 `findByMember` 이용, Post Entity의 `addTime` method 사용
            - `public void changeState(Long postId, StudyState state)`
                - 게시글 상태 변경
                - 회원의 상태 변경에 따라 게시글도 같이 상태 변경 시켜주는 로직, 변경 감지 사용
                - 게시글 리포지토리의 `findOne` 이용, Post Entity의 `eidtState` method 사용
            - `public void editPostImg(Long id, Image image)`
                - 게시글 이미지 업로드 및 변경
                - Post 의 이미지 업로드, 변경 감지 사용
                - 이미지 업로드는 다른 값들과는 독립적으로 업로드 되고, 변경되기에 Setter를 통해 변경
                - 게시글 리포지토리의 `findOne` 이용, Post Entity의 Setter 사용
            - `public void deletePost(Post post)`
                - 게시글 삭제
                - 게시글 리포지토리의 `deleteOne` 이용
        - **@Transactional(readOnly = true)**
            - `public Post findOne(Long id)` : Post id 를 통한 게시글 단건 조회 (리포지토리 `findOne` 단순 위임)
            - `public List<Post> findByPage(Integer offset)`
                - 게시글 페이징 조회
                - 인자로 받아 오는 offset을 기준으로 12개의 post 를 조회
                - 리포지토리의 `findByPaging` 단순 위임
            - `public Optional<Post> getRecentPost(Long memberId)`
                - 해당 회언의 최근 게시글 조회
                - 없는 경우(신규 회원)도 존재 하기에 Null 값을 포함할 수 있도록 Optional 사용
                - 리포지토리의 `findByMember` 단순 위임
    - 게시글 컨트롤러 (PostController)
        
        > 게시글 생성은 항상 자동 생성. 즉, 요청 자체에 새로운 글을 작성하는 요청이 없음.
        > 
   
        - 웹 계층 구현 부분, 글은 항상 Member에 의해 생성되고 변경되기에 MemberService 또한 DI 받음
        - DI : 게시글 서비스(PostService), 회원 서비스(MemberService), AWS S3 서비스(S3Service)
        - `@GetMapping("/posts/edit/{id}")`
            - 게시글 수정 폼, 비지니스 로직 포함
            - URL을 통해 받아 온 post id 에 해당 하는 post edit
            - 이때, Authentication을 통해 해당 post의 member id와 접근하는 (login된) 회원의 id 가 일치한지 확인 (권한 인증)
            - Model Attribute
                - PostEditForm DTO : 기존 post의 값을 가져와 이 값들을 수정할 수 있도록 설정, Valid 가능
        - `@PostMapping("/posts/edit/{id}")`
            - 게시글 수정
            - Valid를 통해 필수 값이 입력되지 않으면 다시 입력하도록 설정
            - 받아온 postEditForm의 markdown content를 html로 변경 후 editform에 저장
            - 게시글 서비스의 해당 `edit` 이용 : postEditForm 값을 그대로 변경 해준 후 저장 (변경 감지)
        - `@PostMapping("/posts/edit/{id}/img")`
            - 게시글 이미지 업로드 및 변경
            - 게시글 이미지는 다른 Form들과 같이 동시에 저장되고 수정될 수 없기에 독립적으로 진행하도록 설정
            - `@RequestParam("file") MultipartFile[] files` 를 통해 업로드된 파일을 받아옴 → **MultipartFile file 과 같이 단일로 받아오면 오류!!**
            - AWS S3 서비스의 `upload` 를 통해 S3 Bucket에 업로드
            - 게시글 서비스의 `editPostImg` 를 통해 회원의 이미지 정보 변경
        - `@GetMapping("/posts/delete/{id}")`
            - 게시글 삭제, 비지니스 로직
            - 권한 인증 진행
            - 비지니스 로직 : 삭제하는 글의 state가 END가 아니라면, 회원이 공부중, 혹은 휴식중에 글을 삭제하는 것이므로 회원의 state를 강제로 end로 변경. 삭제하는 글의 state가 END라면, 이미 종료된 글을 삭제하는 것이기에 회원의 State는 그대로 유지
            - 게스글 서비스의 `deletePost` 이용
            - ! 추후 DeleteMapping으로 변경 및 API 사용 필요
        

### 댓글 기능 개발


> 구현 기능
> <ul> <li> 댓글 등록 </li>
> <li> 댓글 삭제 </li> </ul>
> 구현 순서
> <ul> <li> 댓글 레포지토리 개발 </li>
> <li> 댓글 서비스 개발 </li>
> <li> 댓글 컨트롤러(웹 계층) 개발 </li> </ul>

1. 기능 설명
    - 댓글 등록
        - 올라온 글에서 바로 댓글을 달 수 있도록 Post View 에 form 생성
        - 로그인 된 유저만이 댓글 작성 가능하도록 설정 (권한 인증)
    - 댓글 삭제
        - 해당 글의 작성자나 댓글 작성자만이 댓글 삭제 가능 (권한 인증)
2. 댓글 리포지토리, 서비스, 컨트롤러(웹 계층)
    - 댓글 리포지토리 (CommentRepository)
        - JPA를 직접 사용하여 Comment Entity와 관련된 Table에 접근하는 계층
        - DI(Dependency Injection) : EntityManger(JPA)
        - `public void save(Comment comment)` : 댓글 저장, `em.persist` 이용
        - `public void deleteOne(Long id)` : 댓글 삭제, `em.find` 를 통해 해당 댓글을 찾고 `em.remove`를 통해 Entity 삭제 (casecadeType.Remove로 Post와 연관 지었기 때문에 따로 Post에서 comment를 삭제할 필요가 없음)
    - 댓글 서비스 (CommentService)
        - Business Logic, Transaction 정의 및 구현
        - DI(Dependency Injection) : 댓글 리포지토리(CommentRepository)
        - **@Transactional**
            - `public Long save(Comment comment)` : 댓글 저장, 리포지토리 단순 위임
            - `public void deleteComment(Long id)` : 댓글 삭제, 리포지토리 단순 위임
    - 댓글 컨트롤러 (CommentController)
        - 웹 계층 구현
        - DI(Dependency Injection) : 회원 서비스(MemberService), 게시글 서비스(PostService), 댓글 서비스(CommentService)
        - `@PostMapping("/posts/comment/{id}")`
            - 댓글 작성
            - 로그인된 회원의 Member Entity와 현재 댓글을 작성하고자 하는 게시글 Post Entity를 가지고 와서 Comment와 연결한 채로 Entity 생성.
            - 회원 서비스의 `findByLoginId` , 게시글 서비스의 `findOne` , Comment Entity의 `makeComment` method, 댓글 서비스의 `save` 사용.
        - `@GetMapping("/comment/delete/{id}")`
            - 댓글 삭제
            - URL로 받아 온 id에 해당하는 comment 삭제
            - 댓글 서비스의 `deleteComment` 사용
            - ! 추후 DeleteMapping으로 변경 및 API 사용 필요

### 기타 기능 개발

> 구현 기능
> <ul> <li> state 변경 </li>
> <li> AWS S3 이미지 업로드 </li> </ul>
> 구현 Class
> <ul> <li> State 컨트롤러 (웹 계층 및 비지니스 로직 구현) 개발 </li>
> <li> S3 서비스 개발 </li>
> <li> 홈 컨트롤러 (기본 웹 계층) 개발 </li> </ul>

1. State 컨트롤러 (StateController) 기능 및 코드 설명
    - STATE 변경에 따른 작동
    - 항상 현재 로그인된 회원의 상태에서 요청한 State의 변화로 판단
    - ? → 공부 중 | ? → 휴식 중 | ? → 공부 끝 3가지로 구분하여 비지니스 로직 구현
    - DI : 게시글 서비스 (PostService), 회원 서비스 (MemberSerivce)
    1. `@GetMapping("/studying")` → “공부 중” 으로 상태 변경
        - Case #1
            - `if (state == StudyState.*END*)`
            - 사용자의 현 상태가 공부완료. 즉, 공부 시작
            - 비지니스 로직
                
                > 일단 기본적으로 04시를 하루의 시작과 끝으로 설정 (새벽반 고려)
                사용자가 End -> Studying 으로 State를 변경할 시 (공부완료 -> 공부시작)의 상황
                > 
          
                1. 사용자가 현재 쓴 글이 존재하는 지 판단 (즉, 신규 회원인지 아닌지 판단) → `if (optionalPost.isPresent())`
                    - true (쓴 글이 존재)
                        1. 사용자가 가장 최근에 작성한 글이 해당 날짜에 작성한 글이면서 작성 한 시간이 04시 이후인지 판단 → `if (createDate.isEqual(nowDate) & createTime > 4)`
                            - true => 해당 글을 가져와서 이어 작성할 수 있도록 설정
                            - false
                                1. 현재 State 변경 시점이 04시 이전인지 판단 → `if (nowTime < 4)`
                                    - true
                                        1. 그 전날 작성한 글이 존재하는 지, 해당 글이 04시 이후 작성한 글인지 판단 → `if (period.getDays() == 1 & createTime > 4)`
                                            - true => 해당 글을 가져와서 이어 작성할 수 있도록 설정
                                            - flase => 새로운 글 작성
                                    - false => 새로운 글 작성
                    - false => 새로운 글 작성
        - Case #2
            - `if (state == StudyState.*BREAKING*)`
            - 사용자의 현상태가 휴식 중 → 즉, 공부 재시작
            - 바로 현재 작성 중인 글을 가져와 이어 작성할 수 있도록 설정
        - Case #3
            - 사용자의 현상태가 공부 중 → 즉, 잘못 누른 Case
            - “현재 상태를 확인해주세요.” 오류 Alert
    2. `@GetMapping("/breaking")` → “휴식 중” 으로 상태 변경
        - Case #1
            - `if (state == StudyState.*STUDYING*)`
            - 사용자의 현상태가 공부 중 -> 즉, 휴식 시작
            - 회원과 글의 state를 변경해주고 글의 학습 시간 기록 추가
        - Case #2
            - 나머지 경우 “현재 상태를 확인해주세요.” 오류 Alert
    3. `@GetMapping("/end")` → “공부 완료” 로 상태 변경
        - Case #1
            - `if (state == StudyState.*STUDYING*)`
            - 사용자의 현상태가 공부 중 -> 즉, 공부 완료
            - 회원과 글의 state를 변경해주고 글의 학습 시간 기록 추가
        - Case #2
            - 사용자의 현상태가 휴식 중 -> 그대로 상태 변경 후 종료
            - 회원과 글의 state를 변경
        - Case #3
            - 사용자의 현상태가 공부 완료 -> 오류 alert 던지기

1. AWS S3 Bucket 사용 서비스 (S3Service)
    - 설정 변수들을 환경 변수들을 통해 받아 온 후
    - `@PostConstruct`로  S3Client를 초기에 설정해주고 → `public void setS3Client()`
    - 해당 Client로 S3 Bucket에 이미지 업로드 → `public Image upload(MultipartFile file)`
    
2. 홈 컨트롤러 (Root 웹 계층, HomeController)
    - 메인 페이지, 페이징에 따른 post들과 로그인된 member를 DTO로 보여줌
    - DI (Dependency Injection) : 게시글 서비스 (PostService), 회원 서비스 (MemberService)
    - DTO :
        - MemberForm : 로그인 된 회원의 정보를 담음
        - PostViewDto : 글 정보를 담음
        - MemberViewDto : All Members 사이드바에 보여줄 회원 정보를 담음(상태, 이름, 프로필 사진)
    - 로그인 된 상태와 되지 않은 상태에 따라 보여주는 화면, 사용할 수 있는 기능이 다르게 설정
    - `@Requestmapping("/")` -> main 홈, 가장 첫 화면
        - 로그인 된 상태
            - Model Attribute
                - check : 로그인 여부, boolean (true)
                - member : 로그인된 회원, DTO
                - recentPost : 회원이 "공부 중" 이거나 "휴식 중" 일 때의 현재 자신의 작성 중인 글, DTO
                - allMembers : 모든 회원의 공부상태,프로필사진,이름 정보, DTO
        - 로그인 안 된 상태
            - Model Attribute
                - check : 로그인 여부, boolean (false)
        - 공통 (로그인 되든 안되든 항상 일정함)
            - Model Attribute
                - page : 현재 페이지 (페이지 기능을 위한 attribute), -1(default value)
                - posts : 현재 페이지의 개수에 맞는 post list (DTO list)
                - commentForm : 댓글 Form (각 post에 댓글 작성 칸이 있으므로)
    - `@Requestmapping("/{page}")`
        - main 홈과 동일 하지만, 페이징 기능을 활성화 하는 요청
    - `@GetMapping("/auth/login")`
        - main 홈과 동일 하지만, 로그인 오류 화면을 띄우기 위한 요청