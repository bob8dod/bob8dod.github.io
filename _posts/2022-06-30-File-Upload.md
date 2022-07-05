---
layout: post
title:  "File in Spring (and Servlet)"
date:   2022-06-30
image:  /posts/file.png
tags:   SpringBoot Servlet File
---

## 폼 전송 방법

- HTML Form 전송 방식
    - `application/x-www-form-urlencoded`
    - `multipart/form-data`
- `application/x-www-form-urlencoded`
    
    ![Untitled]({{site.baseurl}}/images/posts/post-220630/Untitled.png)
    
    - **Form 데이터**를 서버로 전송하는 **가장 기본적인 방식**
    - form 태그(<form …>)에 별도의 enctype 설정이 없으면 웹 브라우저는 요청 HTTP 메시지 헤더에 `Content-Type: application/x-www-form-urlencoded` 를 추가함
    - 그 후 폼에 입력된(전송할) 것들을 **HTTP Body**에 문자로 `username=kim&age=20` 와 같이 & 로 구분해서 전송함
    - 그렇다면 문자가 아닌 **바이너리로 구성된 첨부파일**도 함께 전송하고 싶다면 어떻게 할까? → **`multipart/form-data`**
- `multipart/form-data`

    ![Untitled]({{site.baseurl}}/images/posts/post-220630/Untitled 1.png)
    
    - **다른 종류의 여러 파일과 폼의 내용 함께 전송** 가능 (**문자**와 **첨부파일** 등)
    - 각각의 항목을 구분해서, 한번에 전송하는 것
    - form 태그(<form …>)의 **enctype**에 별도의 설정 필요 → `enctype="multipart/form-data"` 지정
    - Form의 입력 결과로 생성된 요청 HTTP 메시지를 보면 알 수 있듯이 각각의 전송 항목이 “**————XXX**” 로 **구분(Part)**되어 있으며 **Content-Dispositon 이라는 항목별 헤더**가 추가되어 있고 **부가 정보** 또한 포함되어 있음.
    - 해당 예시에서는 uesrname, age, file1이 분리되어 있고 내용에는 문자, **파일의 경우**는 **이름과 Content-Type이 추가되고 바이너리 데이터가 전송**됨
    - 그렇다면 이렇게 구분된 데이터를 포함한 요청 HTTP 메시지를 어떻게 서버에서 사용할 수 있을까? → **스프링 없이 서블릿만으로 사용 ⇒ 스프링에서 파일 받기 의 순서**로 알아보자잉

## File Upload in Servlet (without Spring)

### 서블릿에서 Multipart 형식의 Form 데이터 받아오기

- Spring에서 제공하는 파일 업로드 기능없이 **서블릿만으로 파일 업로드** 구현
- html
    
    ```html
    <form th:action method="post" enctype="multipart/form-data">
    	 <ul>
    		 <li>상품명 <input type="text" name="itemName"></li>
    		 <li>파일<input type="file" name="file" ></li>
    	 </ul>
    	 <input type="submit"/>
    </form>
    ```
    
    - multipart로 form을 받아올 때는 항상 `enctype="multipart/form-data"` 부분 추가!
    - multipart로 text, file 2가지 **Part**를 받아옴
- Controller
    
    ```java
    @PostMapping("/upload")
    public String saveFileV1(HttpServletRequest request) throws ServletException, IOException {
    	String itemName = request.getParameter("itemName");
    	log.info("itemName={}", itemName);
    	Collection<Part> parts = request.getParts();
    	log.info("parts={}", parts);
    	return "upload-form";
    }
    ```
    
    - `request.getParameter` 를 통해서 **text의 값**을 가져올 수는 있음. but file의 값은 가져올 수 없음! → file은 parameter형식으로 넘어오는게 아니니깐!
    - `request.getParts()` : `multipart/form-data` 전송 방식에서 **각각 나누어진 부분을 받**아서 확인 가능 → `parts=[org.apache.catalina.core.ApplicationPart@13766755, org.apache.catalina.core.ApplicationPart@30602154]`
    - **application.properties** 에서 `logging.level.org.apache.coyote.http11=debug` 설정을 통해 HTTP 요청 메시지를 확인해 보면 **Part로 구분되어 전달**되는 것을 확인할 수 있음

      ![Untitled]({{site.baseurl}}/images/posts/post-220630/Untitled 2.png)
        
- Multipart 사용 옵션
    - application.properties 에서 `multipart` 옵션 설정 가능
    - 업로드 사이즈 제한
        - `spring.servlet.multipart.max-file-size=1MB` : 파일 하나의 최대 사이즈(default = 1MB)
        - `spring.servlet.multipart.max-request-size=10MB` : 전체 파일(여러 파일의 합)의 최대 사이즈(default = 10MB)
    - mutlipart 허용
        - `spring.servlet.multipart.enabled=true`
            - defalut = true
            - 이 옵션을 켜면 복잡한 멀티파트 요청을 처리해서 사용할 수 있게 제공
            - 해당 옵션을 키면 기본으로 사용했던 `HttpServletRequest` 가 `MultipartResolver`를 통해서 **`MultipartHttpServletRequest`**로 변환됨
            - **MultipartHttpServletRequest**
                - HttpServletRequest 의 자식 인터페이스
                - **멀티파트와 관련된 추가 기능**을 제공
                - 이것을 사용하면 **멀티파트**와 관련된 여러가지 처리를 **편리하게 사용** 가능
            - BUT 우린 HttpServletRequest 가 변환된 MultipartHttpServletRequest를 사용하지 않고 **`MultipartFile`** 이라는 **스프링에서 제공하는 유용한 기능을 사용**할 것!

### 서블릿에서 Multipart형식의 File 업로드

- **서블릿**이 제공하는 **Part**를 이용해서 실제로 파일 업로드 구현 (Spring의 기능을 이용하지 않고 서블릿의 Part 기능만을 이용하여)
- 경로 설정
    - application.properties 에 `file.dir` 속성에 업로드 경로 설정 (ex_`/Users/park/servlet/file/`)
    - 해당 경로 값은 `@Value` 를 통해 가져와 사용할 예정
- ServletUploadController
    - 경로 불러오기
        
        ```java
        @Value("${file.dir}")
        private String fileDir;
        ```
        
        - `@Value` 를 통해 **application.propreties의 속성 값**을 가져옴
    - 파일 확인 및 업로드 (**Servlet의 Part 이용**)
        
        ```java
        @PostMapping("/upload")
        public String saveFileV1(HttpServletRequest request) throws ServletException, IOException {
        
            Collection<Part> parts = request.getParts(); // 각각의 부분(Part)들을 가져옴
        
            for (Part part : parts) {
                Collection<String> headerNames = part.getHeaderNames(); // parts 도 header 와 body 로 구분됨!
                for (String headerName : headerNames) {
                    log.info("header {}: {}", headerName, part.getHeader(headerName));
                }
        
                //편의 메서드
                //content-disposition 의 filename 따오기
                log.info("submittedFilename={}", part.getSubmittedFileName()); // 자동 파싱해주는 것
                log.info("size={}", part.getSize()); //part body size
        
                //데이터 읽기
                InputStream inputStream = part.getInputStream(); // 실제 받아온 바이너리 값을 읽는 것
                String body = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8); // 그 읽어 온 값을 String 으로 보기 편하게 변경
                log.info("body={}", body);
        
                //파일에 저장하기
                if (StringUtils.hasText(part.getSubmittedFileName())) { // 파일 제출인지 확인
                    String fullPath = fileDir + part.getSubmittedFileName();
                    log.info("파일 저장 fullPath={}", fullPath);
                    part.write(fullPath); // 해당 설정된 경로를 통해 저장
                }
            }
        
            return "upload-form";
        }
        ```
        
        - **Servlet의 Part** : Multipart의 각각의 부분들 중 **File과 관련된 Part에 대해**서 확인하거나 업로드할 때 사용
        - `part.getInputStream()`: Part의 전송 데이터를 읽을 수 있음
        - 중요한 부분은 “파일에 저장하기” 부분
            - `StringUtils.hasText(part.getSubmittedFileName())` 을 통해 현재 part가 제출된 파일인지 확인 → (`part.getSubmittedFileName()` : 클라이언트가 전달한 파일명. 파일이 없다면 `null`)
            - `part.write(fullPath)` : 지정된 경로에 해당 파일 저장
- 이렇게 서블릿이 제공하는 **Part 로도 파일을 저장할 수 있**지만, 추가적으로 **HttpServletRequest 를 사용**해야 하기도 하고. 추가적으로 파일인지 아닌지 **확인 하고 구분해야되는 여러코드를 요구**함. → **Spring을 사용하면 훨씬 더 편리하게 사용 가능!!**

## File Upload in Spring

### 스프링과 파일 업로드 (MultipartFile)

- 스프링은 **`MultipartFile`** 이라는 인터페이스로 multipart의 file을 쉽게 다룰 수 있도록 지원
- Controller
    
    ```java
    @PostMapping("/upload")
    public String saveFile(@RequestParam String itemName, @RequestParam MultipartFile file) throws IOException {
    
        log.info("itemName={}", itemName);
        log.info("multipartFile={}", file);
    
        // 파일 저장
        if (!file.isEmpty()) {
            String fullPath = fileDir + file.getOriginalFilename();
            file.transferTo(new File(fullPath));
        }
    
        return "upload-form";
    }
    ```
    
    - `@RequestParam MultipartFile file` : 제출된 파일을 **MultipartFile**로 받아옴 → 서블릿의 복잡한 과정이 **어노테이션과 Spring에서 지원하는 인터페이스로 한번에 해결**되는 것 (HTML Form의 name에 맞추어 `@RequestParam` 을 적용. `@ModelAttribute` 도 가능!)
    - `file.getOriginalFilename()` : 업로드 파일 명
    - `file.transferTo(new File(fullPath));` : 해당 경로로 파일 저장

<aside>
⚠️ <strong>ArgumentResolver, ConversionService in @RequestParam MultipartFile file</strong> <br>
먼저 ArguementResolver 가 @RequestParam과 MultipartFile 을 지원하는지 확인한 후, ConversionService를 통해 String → MultipartFile Converter가 있는지 확인한 후 Type Converting을 진행. 그 후 생성된 객체(MultipartFile)를 ArgumentResolver가 마련해줌 <br>
⇒ 즉, <b>서블릿만으로 파일을 확인하는 과정 자체를 MultipartFile을 만들어주는 ArgumentResolver가 다 자동으로</b> 해주는 것!

</aside>

### 파일 업로드 및 확인, 다운로드 (실전)

- 실제 파일,이미지 업로드 및 확인, 다운로드에서는 고려해야할 점들이 있음. 그런 사항들을 확인
- 상품 객체 및 DTO
    - 상품 이름, 첨부파일 하나, 이미지 파일 여러개(multiple)
    - 업로드 파일 정보 보관
        
        ```java
        @Data
        @AllArgsConstructor
        public class UploadFile {
        	 private String uploadFileName;
        	 private String storeFileName;
        }
        ```
        
        - `uploadFileName` 은 파일의 원본명
        - `storeFileName` 은 Repository에 저장되는 파일의 이름 → UUID 사용 (똑같은 이름으로 저장되면 안됨! **충돌 예방**)
    - Repository에 저장되는 상품 객체
        
        ```java
        @Data
        public class Item {
        		private Long id;
        		private String itemName;
        		private UploadFile attachFile; // 단일 파일
        		private List<UploadFile> imageFiles; // 파일 여러개
        }
        ```
        
    - Item 객체의 DTO
        
        ```java
        @Data
        public class ItemForm {
        	 private Long itemId;
        	 private String itemName;
        	 private MultipartFile attachFile;
        	 private List<MultipartFile> imageFiles;
        }
        ```
        
        - Form 태그로 받아올 때 사용되는 **Item DTO** (상품 저장용 폼)
        - Form 에서 받아올 때의 **단일 파일은 MultipartFile**로, **여러 파일은 List<MultipartFile>**로 받아올 수 있음
        
        <aside>
        ⚠️ <strong>DTO를 사용하는 이유!</strong> <br>
        <code>@ModelAttribute</code>로 Form 의 입력 값들을 받아오는데, 이때 <b>파일을 받아오기 위해선 MultipartFile 을 사용</b>해야됨. 하지만 저장소에는 이 그대로 저장하면 안되고 해당 파일을 <b>저장한 경로를 저장</b>해야됨. 그러기 때문에 <b>DTO를 사용해서 MultipartFile로 받아오고</b> 이를 Storage에 저장 후 <b>저장 경로를 가지고 있는 실제 Item 객체를 Repository에 저장</b>해야 됨
        
        </aside>
        
    - 상품은 업로드 및 확인, 다운로드 가능
- 업로드
    - Repository는 이전과 동일하므로 생략 (이전 파트 참고)
    - 업로드 Form
        
        ```java
        @GetMapping("/items/new")
        public String newItem(@ModelAttribute ItemForm form) {
        	return "item-form";
        }
        ```
        
        - 빈 itemDTO를 보내주어 form의 object를 맞춰 줌
    - 업로드 진행
        
        ```html
        <form th:action method="post" enctype="multipart/form-data">
        	 <ul>
        		 <li>상품명 <input type="text" name="itemName"></li>
        		 <li>첨부파일<input type="file" name="attachFile" ></li>
        		 <li>이미지 파일들<input type="file" multiple="multiple" name="imageFiles" ></li>
        	 </ul>
        	 <input type="submit"/>
        </form>
        ```
        
        - 단일 파일은 그냥 file을 통해서 받아와 주면 됨 → Controller에서 `MultipartFile` 로 받음
        - 여러 파일의 input은 multiple 설정을 통해 받아와야 됨! → Controller에서 `List<MultipartFile>` 로 받음
        
        ```java
        @PostMapping("/items/new")
        public String saveItem(@ModelAttribute ItemForm form, RedirectAttributes redirectAttributes) throws IOException {
            UploadFile attachFile = fileStore.storeFile(form.getAttachFile());
            List<UploadFile> storeImageFiles = fileStore.storeFiles(form.getImageFiles());
        
        		// DTO → Item
            Item item = new Item();
        		item.setItemName(form.getItemName());
            item.setAttachFile(attachFile);
        		item.setImageFiles(storeImageFiles);
            itemRepository.save(item); //데이터베이스에 저장
        
            redirectAttributes.addAttribute("itemId", item.getId());
            return "redirect:/items/{itemId}";
        }
        ```
        
        - `@ModelAttribute` 를 통해서 Form의 입력 값들(text, file, multiple files)을 받아옴
        - **DTO → Item** : 받아온 **MutlipartFile 을 UploadFile로 변경**함으로써 실제 파일이 아닌 원본 **파일 명과 파일 저장 경로만을 Item에 저장**
        - **PRG** : Post 후 Redircet를 통해 Get 요청 실행
- 조회 및 다운로드
    - Item 조회
        
        ```java
        @GetMapping("/items/{id}")
        public String items(@PathVariable Long id, Model model) {
            Item item = itemRepository.findById(id);
            model.addAttribute("item", item);
            return "item-view";
        }
        ```
        
    - Item의 첨부파일, 이미지 파일 조회 및 다운로드
        
        ```html
        상품명: <span th:text="${item.itemName}">상품명</span><br/>
        첨부파일: <a th:if="${item.attachFile}" th:href="|/attach/${item.id}|"
        						 th:text="${item.getAttachFile().getUploadFileName()}" /><br/>
        <img th:each="imageFile : ${item.imageFiles}" 
        		 th:src="|/images/${imageFile.getStoreFileName()}|" 
        		width="300" height="300"/>
        ```
        
    - **이미지 조회** (각 이미지들을 보여주기 위함) → `th:src="|/images/${imageFile.getStoreFileName()}|"`
        
        ```java
        @ResponseBody
        @GetMapping("/images/{filename}")
        public Resource imgViewer(@PathVariable String filename) throws MalformedURLException {
            return new UrlResource("file:" + fileStore.getFullPath(filename)); // 직접 경로를 따라 가서 파일을 찾아와서 아웃풋스트림으로 반환
        }
        ```
        
        - `@ResponseBody` : 반환된 **아웃풋 스트림 그대로 HTTP Body에** 담기위함 (View Resolver가 아닌 HttpMessageConverter 동작) → img 태그가 이를 받아서 이미지를 띄워주는 것
        - **Resource, UrlResource** : 해당 경로에 대한 파일을 **아웃풋스트림으로 변환하여 전달**
    - 첨부파일 다운로드 (클릭 시 다운로드) → `th:href="|/attach/${item.id}|"`
        
        ```java
        @GetMapping("/attach/{itemId}")
            public ResponseEntity<Resource> downloadAttach(@PathVariable Long itemId) throws MalformedURLException {
                Item item = itemRepository.findById(itemId);
                String storeFileName = item.getAttachFile().getStoreFileName();
                String uploadFileName = item.getAttachFile().getUploadFileName();
        
                UrlResource resource = new UrlResource("file:" + fileStore.getFullPath(storeFileName));
        
                String encodedUploadFileName = UriUtils.encode(uploadFileName, StandardCharsets.UTF_8); // 인코딩 (한글을 위함)
                String contentDisposition = "attachment; filename=\"" + encodedUploadFileName + "\""; // 다운로드하기 위한 규약
        
                return ResponseEntity.ok()
                        .header(HttpHeaders.CONTENT_DISPOSITION, contentDisposition) // 다운로드를 위한 헤더
                        .body(resource);
            }
        ```
        
        - 일반적인 `@ResponseBody` 로 UrlResource를 반환하면 그냥 조회만 됨.
        - 다운로드를 위해선 **response Http Header 값을 변경**해줘야 됨 → **ResponseEntity 사용**
        - body에는 조회의 방식 그대로 UrlResource로 채워줌 (아웃풋스트림)
        - `ResponseEntity.ok().header(...).body(resource)`
            - `ResponseEntity.ok()` : `ResponseEntity` 의 status를 200으로 설정
            - `.header(HttpHeaders.CONTENT_DISPOSITION, contentDisposition)` : 다운르도를 위해 header의 **content-disposition 이라는 항목을 따로 설정**해줌(한글을 포함한 제대로된 경로를 위해 인코딩 설정 필수) → **다운로드 규약!**