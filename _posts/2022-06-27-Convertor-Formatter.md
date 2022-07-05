---
layout: post
title:  "TypeConvertor and Formatter"
date:   2022-06-27
image:  /posts/convertor.png
tags:   SpringBoot Convert
---

## 스프링 타입 컨버터

### 타입 컨버터

- Type Converting 상황
    - 문자를 숫자로 변환하거나 반대의 경우처럼 애플리케이션을 개발하다 보면 **타입을 변환해야 하는 경우**가 수없이 많음 → HTTP Body, 요청 파리미터 등으로 **들어오는 데이터는 모두 String**인데 해당 데이터를 Integer 등으로 사용하려면 변환은 필수!
    - TypeConverter(Spring)없이 **Servlet만을 이용**한 경우
        
        ```java
        @GetMapping("/hello-v1")
        public String helloV1(HttpServletRequest request) {
        	String data = request.getParameter("data"); //문자 타입 조회
        	Integer intValue = Integer.valueOf(data); //숫자 타입으로 변경
        	System.out.println("intValue = " + intValue);
        	return "ok";
        }
        ```
        
        - Servlet만을 이용할 경우 받아온 값을 Integer로 **직접 변환해야 해당 Type으로 사용 가능**
    - 스프링 MVC가 제공하는 `@RequestParam` 을 사용 (TypeConverter 자동적용)
        
        ```java
        @GetMapping("/hello-v2")
        public String helloV2(@RequestParam Integer data) {
        	 System.out.println("data = " + data);
        	 return "ok";
        }
        ```
        
        - 문자 10으로 값이 들어오지만 `@RequestParam`을 통해 **Integer로 Type이 알아서 변환**됨
        - **스프링이 중간에서 타입을 변환**해주었기 때문!!!
        - `@ModelAttribute` , `@PathVariable` 도 마찬가지

<aside>
💡 <strong>스프링의 <b>타입 자동 변환</b> 적용 예시</strong>
<ul>
<li>스프링 MVC 요청 파라미터 (<code>String → ?</code>) : <code>@RequestParam</code>, <code>@ModelAttribute</code>, <code>@PathVariable</code></li>
<li>뷰를 렌더링 할 때 (Model에 담긴 값이 렌더링 될 때 _ <code>? → String</code>)</li>
<li><code>@Value</code> 등으로 <b>YML 정보</b> 읽기</li>
<li>XML에 넣은 <b>스프링 빈 정보</b>를 변환</li>
</ul>
</aside>

- Custom Type Converter
    - **확장 가능한 Converter Interface를 제공**하기에 이를를 **구현**하면 됨
    
    ```java
    package org.springframework.core.convert.converter;
    public interface Converter<S, T> {
    	 T convert(S source);
    }
    ```
    
    - **S → T** 로 변환 되는 것
        - **X → Y** 로 변환 되는 것을 구현하고 **Y → X**로 변환되는 것을 구현해주면 **양방향으로 변환 가능**
    - 모든 타입에 적용 가능 (ex_ String → Boolean)

### 타입 컨버터 - Converter

- Converter (`org.springframework.core.convert.converter.Converter`)
- **컨버터를 직접 구현해서 사용**해보기
    - String ↔ Integer
        - String to Integer
            
            ```java
            public class StringToIntegerConverter implements Converter<String, Integer> {
            	 @Override
            	 public Integer convert(String source) {
            			 return Integer.valueOf(source);
            	 }
            }
            ```
            
            - `Converter` **Interface**를 받아 **convert**라는 method를 구현
            - Integer.valueOf(source) 를 통해 **String → Integer 로 반환**
        - Integer to String
            - String to Integer와 S랑 T만 다르고 나머지는 동일
        - 테스트 코드를 통해서 **직접 구현체를 불러와 변환 결과를 확인**할 수 있음. (직접 구현체를 인스턴스로 만들어 사용해야되는 단점 확인)
    - IpProt ↔ String (**사용자 정의 타입 컨버터**)
        - IpPort
            
            ```java
            @Getter
            @EqualsAndHashCode
            @AllArgsContructor
            public class IpPort {
            	 private String ip;
            	 private int port;
            }
            ```
            
            - `@EqualsAndHashCode` : 모든 필드를 사용해서 equals() , hashcode() 를 생성
            - 모든 필드의 값이 같다면 `a.equals(b)` 의 결과가 참
        - IpPort to String
            
            ```java
            public class StringToIpPortConverter implements Converter<String, IpPort> {
            	 @Override
            	 public IpPort convert(String source) {
            			 String[] split = source.split(":");
            			 String ip = split[0];
            			 int port = Integer.parseInt(split[1]);
            			 return new IpPort(ip, port);
            	 }
            }
            ```
            
            - “127.0.0.1:8080” 같은 문자를 입력하면 IpPort 객체를 만들어 반환
        - String to IpPort
            
            ```java
            public class IpPortToStringConverter implements Converter<IpPort, String> {
            	 @Override
            	 public String convert(IpPort source) {
            		 return source.getIp() + ":" + source.getPort();
            	 }
            }
            ```
            
            - IpPort 객체를 입력하면 127.0.0.1:8080 같은 문자를 반환
        - 테스트 코드를 통해서 **직접 구현체를 불러와 변환 결과를 확인**할 수 있음. (직접 구현체를 인스턴스로 만들어 사용해야되는 단점 확인)
- 타입 컨버터 인터페이스를 구현해서 직접 컨버터를 구현해보았는데, **직접 구현한 객체를 계속해서 불러와 사용해야 된다는 단점**이 있음.
- 즉, 타입 컨버터를 **등록하고 관리하면서 알아서 기능이 제공되게 끔 하는 역할**을 가진 것이 **필요**! 즉, **중간 매개체 역할**이 필요! → **Spring의 ConversionService**

<aside>
💡 <strong>String이 제공하는 기본 Type Converter</strong> <br>
스프링은 String, Integer(int), Boolean, Enum등 일반적인 타입에 대한 대부분의 컨버터를 기본으로 제공 → 해당 객체들에 대한 수 많은 컨버터들이 이미 구현체로 등록되어 있음!!

</aside>

### ConversionService

- 컨버터들을 모아두고 편리하게 사용할 수 있는 기능을 제공
- `ConversionService` Interface
    
    ```java
    public interface ConversionService {
    	boolean canConvert(@Nullable Class<?> sourceType, Class<?> targetType);
    	boolean canConvert(@Nullable TypeDescriptor sourceType, TypeDescriptor targetType);
    	
    	<T> T convert(@Nullable Object source, Class<T> targetType);
    	Object convert(@Nullable Object source, @Nullable TypeDescriptor sourceType, TypeDescriptor targetType);
    }
    ```
    
    - 컨버팅이 **가능한지 확인하는 Method**와 **컨버팅을 수행하는 Method** 제공
- `ConversionService` 사용 (`DefaultConversionService`)
    - Test Code를 통해 직접 등록하고 사용해보기
    - `DefaultConversionService`
        - `ConversionService` 인터페이스 **구현체**
        - 추가로 **컨버터를 등록하는 기능**도 제공
        - **‘등록’과 ‘사용’ 의 분리**로 **인터페이스 분리 원칙(ISP - Interface Segregation Principal)**를 지키고 있음!
            - ConversionService : 컨버터 **사용**에 초점
            - ConverterRegistry : 컨버터 **등록**에 초점
            - 인터페이스를 분리하여 컨버터를 **사용하는 곳**과 **등록하고 관리하는 곳**의 관심사를 **명확하게 분리** 가능
            - 컨버터를 사용하는 곳은 ConversionService 만 의존하면 됨 즉, 컨버터를 어떻게 등록하고 관리하는지는 몰라도 되고 해당 등록 및 관리의 방법이 변경되더라도 의존하고 있지 않기때문에 그냥 사용하면 됨 → 관심사 분리. 인터페이스 분리 (ISP)
    - `DefaultConversionService` 를 통해 ConverterService, ConverterRegistry  이용
        
        ```java
        DefaultConversionService conversionService = new DefaultConversionService();
        // 등록 (ConverterRegistry에만 의존)
        conversionService.addConverter(new StringToIpPortConverter());
        conversionService.addConverter(new IpPortToStringConverter());
         
        // 사용 (ConversionService에만 의존)
        IpPort ipPort = conversionService.convert("127.0.0.1:8080", IpPort.class);
        String ipPortString = conversionService.convert(new IpPort("127.0.0.1", 8080), String.class);
        
        // 검증
        assertThat(ipPort).isEqualTo(new IpPort("127.0.0.1", 8080)); // 검증
        assertThat(ipPortString).isEqualTo("127.0.0.1:8080");
        ```
        
- 스프링은 즉, 이런 `ConversionService`를 사용해서 **자동으로 타입을 변환**해줌. (ex_`@RequestParam` 같은 곳에서 `ConversionService`를 이용하여 **Type Converting을 진행**)

### 스프링에서의 Converter 사용

- **웹 애플리케이션**에서 자동으로 **Converter가 실행**되게 하기
- custom Converter(`IpPort ↔ String`) 등록 후 사용하기
- `WebConfig` - 컨버터 등록
    
    ```java
    @Configuration
    public class WebConfig implements WebMvcConfigurer {
    	 @Override
    	 public void addFormatters(FormatterRegistry registry) {
    		 registry.addConverter(new StringToIpPortConverter());
    		 registry.addConverter(new IpPortToStringConverter());
    	 }
    }
    ```
    
    - 스프링은 내부에서 ConversionService 를 제공하고 있기에 **이 ConversionService에 내가 추가하고 싶은 Converter를 add** 해주면 됨!
    - 추가로, `Integer ↔ String` 과 같은 **기본 타입 변환은 이미 해당 ConversionService에 추가되어 있음!** → 그냥 사용하면 됨
- `HelloController` - 컨버터 적용
    
    ```java
    @GetMapping("/ip-port")
    public String ipPort(@RequestParam IpPort ipPort) {
    	 System.out.println("ipPort IP = " + ipPort.getIp());
    	 System.out.println("ipPort PORT = " + ipPort.getPort());
    	 return "ok";
    }
    ```
    
    - ?ipPort=127.0.0.1:8080 쿼리 스트링을 URL을 통해 입력해주면 **ipPort 객체로 Type Convert 됨!** ( ”127.0.0.1:8080” → IpPort(127.0.0.1, 8080) )
    - `@RequestParam` 은 `@RequestParam` 을 처리하는 **ArgumentResolver** 인
    **RequestParamMethodArgumentResolver** 에서 **ConversionService 를 사용**해서 **타입을 변환!**
    - 즉, **ArgumentResolver**가 해당 **String → ipPort** 로 변환되는 **Converter가 있는지 ConversionService에서 찾은 후 변환**해주는 것!
    

### View Template(Thymeleaf) with Converter

- 타임리프는 **렌더링 시에 컨버터를 적용**해서 렌더링 하는 방법을 편리하게 지원
- 렌더링 시 **모든 Object들은 String으로** 바뀌게 됨. 이때의 **Converting(**`Object → String`**)을 수행**하는 것
- 타임리프에서 ConversionService 이용
    - 일반 변수 ConversionService
        - 기존 변수 표현식 : `${...}`
            - `<li>${ipPort}: <span th:text="${ipPort}" ></span></li>` → ${ipPort}: hello.typeconverter.type.IpPort@59cb0946
            - **Object의 toString이 적용**됨
        - **컨버전 서비스 적용** : **`${{...}}` 괄호 2개!**
            - `<li>${{ipPort}}: <span th:text="${{ipPort}}" ></span></li>` → ${{ipPort}}: 127.0.0.1:8080
            - **ConversionService가 적용**됨
    - Form 에서의 ConversionService
        - 컨버전 서비스 적용 : `th:field` 이용
        - ipPort를 가지는 Form 객체(`@Data`)가 있다고 가정, 해당 Form객체를 통해 Form의 값을 받아오는 것
            
            ```html
            <form th:object="${form}" th:method="post">
            	 th:field <input type="text" th:field="*{ipPort}"><br/>
            	 th:value <input type="text" th:value="*{ipPort}">(보여주기 용도)<br/>
            	 <input type="submit"/>
            </form>
            ```
            
            ![Untitled]({{site.baseurl}}/images/posts/post-220627/Untitled.png)
            
        - 컨버전 서비스를 적용한 `th:field` 는 ConversionService를 통해 직접 등록한 Converter를 통해 Convert 되어 `IpPort → String`**으로 잘 convert 된 것**을 확인할 수 있음
        - 컨버전 서비스를 적용하지 않은 `th:value` 는 **그냥 toString이 적용**된 것을 확인할 수 있음
        - 뿐만 아니라 제출을 통해 **submit 될때도** String으로 받은 값을 IpPort로 변환해주는 **Converting이 수행**되는 것을 확인할 수 있음! `th:field` 를 통해!
        - 타임리프의 `th:field` 는 앞서 설명했듯이 id , name 를 출력하는 등 다양한 기능이 있는데, 여기에 **컨버전 서비스**도 함께 적용됨!
        

## 스프링 포매터

### Formatter

- Converter 같은 경우 S → T 의 타입에 제한이 없음. 즉, 범용 타입 변환 기능 제공
- 하지만, 보통 **웹 애플리케이션 환경**에선 String 에서 특정 타입으로 변환( `String → ?` )하거나 특정 타입에서 String으로 변환( `? → String` )하는 상황이 대부분. ⇒ Ex) Integer 1000 → String “1,000” 형태로 **Formating 해야 될 때**
- 이렇게 **문자에 특화된 변환기**를 **Formatter**라고 함. 즉, **Converter의 특별한 버전**이라고 생각하면 됨
- **Converter VS Formatter**
    - **Converter** : 범용 타입 변환기 ( `? → ?` )
    - **Formatter** : **문자에 특화** (`String → ?` or `? → String`) + **현지화(Locale)**
- `Formatter` Interface
    
    ```java
    public interface Formatter<T> extends Printer<T>, Parser<T> {}
    
    public interface Printer<T> {
    	String print(T object, Locale locale);
    }
    
    public interface Parser<T> {
    	T parse(String text, Locale locale) throws ParseException;
    }
    ```
    
    - `String print(T object, Locale locale)` : 객체를 문자로 변경 (`Object → String`)
    - `T parse(String text, Locale locale)` : 문자를 객체로 변경 (`String → Object`)
- Formatter 직접 개발
    - 숫자 1000 을 문자 "1,000" 으로 변환해주는 Formatter ⇒ 1000 단위로 쉼표가 들어가는 포맷을 적용
    
    ```java
    public class MyNumberFormatter implements Formatter<Number> {
    	 @Override
    	 public Number parse(String text, Locale locale) throws ParseException {
    		 NumberFormat format = NumberFormat.getInstance(locale);
    		 return format.parse(text);
    	 }
    	 @Override
    	 public String print(Number object, Locale locale) {
    		 return NumberFormat.getInstance(locale).format(object);
    	 }
    }
    ```
    
    - `parse` : NumberFormat 을 통해 Locale 정보에 따른 단위 쉼표에 따른 숫자 반환 [”1,000” → 1000]. Long으로 반환
    - `print` : NumberFormat 을 통해 Locale 정보에 따른 단위 쉼표 반환 [1000 → ”1,000”]
- 개발한 Formatter Test
    
    ```java
    MyNumberFormatter formatter = new MyNumberFormatter();
    
    @Test
     void parse() throws ParseException {
    	 Number result = formatter.parse("1,000", Locale.KOREA);
    	 assertThat(result).isEqualTo(1000L); //Long 타입 주의
     }
     @Test
     void print() {
    	 String result = formatter.print(1000, Locale.KOREA);
    	 assertThat(result).isEqualTo("1,000");
     }
    ```
    
    - 직접 개발한 Formatter를 불러와서 해당 Formatter를 가지고 **parse, print Test**.
    - parse() 의 결과가 Long인점 주의!
    - 마찬가지로 **사용할때마다 불러서 적용해야된다는 불편함**이 있음! → Converter처럼 ConversionService같이 **등록하고 관리하여 편리하게 사용할 수 있는 Service가 필요**함!!

### 포맷터를 지원하는 컨버전 서비스

- 기존에 등록 및 관리에서 사용했던 ConversionService는 컨버터만 등록할 수 있었고 포맷터는 등록할 수 없음. 하지만 사실 **포맷터도 생각해보면 특별한 컨버터**일 뿐!
- 즉, **포맷터를 지원하는 ConversionService를 사용**하면 됨. 해당 ConversionService를 사용하면 포맷터를 등록할 수 있고, 내부에서 어댑터를 이용 하여 **Formatter가 Converter처럼 동작**하도록 지원함 → `FormattingConversionService` interface
- `DefaultFormattingConversionService`
    - `FormattingConversionService` 의 **구현체**
    - `FormattingConversionService` 에 기본적인 통화, 숫자 관련 몇가지 **기본 포맷터를 추가해서 제공**
    - `Converter`, `Formatter` 모두 등록이 가능하며, 사용할 때는 `Converter`, `Formatter` **구분 없이 사용 가능**
    - Test Code
        
        ```java
        DefaultFormattingConversionService conversionService = new DefaultFormattingConversionService();
        
        //컨버터 등록
        conversionService.addConverter(new StringToIpPortConverter());
        conversionService.addConverter(new IpPortToStringConverter());
        
        //포맷터 등록
        conversionService.addFormatter(new MyNumberFormatter());
        
        //컨버터 사용
        IpPort ipPort = conversionService.convert("127.0.0.1:8080", IpPort.class);
        assertThat(ipPort).isEqualTo(new IpPort("127.0.0.1", 8080));
        
        //포맷터 사용
        assertThat(conversionService.convert(1000, String.class)).isEqualTo("1,000");
        assertThat(conversionService.convert("1,000", Long.class)).isEqualTo(1000L);
        ```
        
        - **등록**
            - `addConverter` : 컨버터 등록
            - `addFormatter` : 포맷터 등록
        - **사용** : 컨버터, 포맷터 **모두 `convert`를 통해 사용 가능!**
    - 스프링 부트는 웹 어플리케이션에 `DefaultFormattingConversionService` 를 상속 받은 `WebConversionService` 를 내부에서 사용하기 때문에 위와 동일하게 **등록만 따로** 해주면, **Converter, Formatter 구분없이 사용 가능!**
    - 즉, 컨버터를 사용하든, 포맷터를 사용하든 등록 방법은 다르지만, **사용할 때는 컨버전 서비스를 통해서 일관성 있게 사용**이 가능함! (**등록 시에만 신경**쓰면 됨!)

### 웹 어플리캐이션에 포맷터 적용

- WebConfig에서 Formatter를 적용하기만 하면 됨
- 이때 주의할 것은 항상 같은 변환에 대해선 Converter가 Formatter보다 우선순위를 가지고 있기 때문에 **Converter에서 같은 변환이 이루어지는 지 확인**해야 됨 [ Ex) Converter (of String ↔ Integer) **>>** Formatter (of String ↔ Integer ) ]
- `WebConfig`
    
    ```java
    @Configuration
    public class WebConfig implements WebMvcConfigurer {
    	 @Override
    	 public void addFormatters(FormatterRegistry registry) {
    		 //registry.addConverter(new StringToIntegerConverter()); 
    		 //registry.addConverter(new IntegerToStringConverter());
    		 registry.addConverter(new StringToIpPortConverter());
    		 registry.addConverter(new IpPortToStringConverter());
    
    		 //포매터 추가
    		 registry.addFormatter(new MyNumberFormatter());
    	 }
    }
    ```
    
    - `StringToIntegerConverter()`, `IntegerToStringConverter()` 는 `String ↔ Integer` **Converter** 이므로 등록되있을 경우 `MyNumberFormatter()` 보다 우선순위를 가짐. → 주석처리
    - `addFormatter` 를 통해 **포매터 등록**
- 실행 결과
    - Integer → String :  `${{number}}` →  `10,000`
    - String → Integer : `input=10,000` → `data = 10000`

### 스프링이 제공하는 기본 포맷터

- 스프링은 **기본 타입들에 대해 수 많은 Formatter를 기본으로 제공**
- Formatter Interface의 구현체들을 살펴보면 기본 객체들과 관련된 수 많은 구현체들이 있는 것을 확인할 수 있음
- 이렇게 Formatter는 기본 형식이 지정되어 있기 때문에, 각 상황(ex_컨트롤러마다 다르게 지정하고 싶은 Formatter, 객체의 각 필드마다 다르게 지정하고 싶은 Formatter)마다 **다른 형식으로 포맷을 지정하기는 어려움**
- 이런 문제를 해결하기 위해 스프링은 **어노테이션 기반으로 원하는 형식을 지정해서 사용**할 수 있는 **두가지 포맷터를 제공**
- Annotation 기반 Formatter
    - `@NumberFormat` : **숫자** 관련 형식 지정 포맷터
    - `@DateTimeFormat` : **날짜** 관련 형식 지정 포맷터 사용
- Annotation 기반 Formatter 사용
    
    ```java
    @Controller
    public class FormatterController {
    	 ...
    
    	 @Data
    	 static class Form {
    			
    		 @NumberFormat(pattern = "###,###")
    		 private Integer number;
    		 
    		 @DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    		 private LocalDateTime localDateTime;
    	 }
    
    }
    ```
    
    - 해당 컨트롤러에서의 Form 데이터 사용에 있어서 각 필드는 해당 어노테이션의 영향을 받는 것.
    - 즉, 이 컨트롤러에서 적용되는 모든 서비스에서 사용되는 이 Form데이터의 **각 필드에만 해당 Formatter가 적용**되는 것 → **세부적으로 원하는 상황에서만 사용 가능**
    - `@NumberFormat(pattern = "###,###")` : 10000 ↔ “10,000”
    - `@DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss")` : 2022-01-01T00:00:00 ↔ “2022-01-01 00:00:00”

<aside>
🚨 <strong>그럼 HttpMessageConverter 에서도 ConversionService가 적용 되는 건가?</strong>
<ul>
<li>NOPE! HttpMessageConverter 같은 경우 <b>Jasckson 같은 라이브러리를 사용</b>. 즉, 변환 과정에 있어서 모든 동작은 Jackson 같은 라이브러리를 따른다는 것! (ConvresionService가 낄 수 없음). 결국 HttpMessageConverter 를 통해 JSON → Object의 과정에서 원하는 포맷으로 변환하고 싶으면 <b>Jackson 라이브러리가 제공하는 설정을 통해서 포맷을 지정</b>해야됨. </li>
<li>결론</li>
    <ul>
    <li><b>API 의 Converting</b>(@RequestBody, @ResponseBody, HttpEntity, …) → HttpMessageConverter </li>
    <li><b>일반 MVC의 Converting</b>(@RequestParam, @ModelAttribute, @PathVariable, Veiw Template Rendering, …) → ConversionService </li>
    </ul>
</ul>
</aside>

<aside>
⚠️ <strong>@ResponseBody(HttpMessageConverter) vs @ModelAttribute(ConversionServcie)</strong>
<ul>
<li><strong>@ResponseBody</strong></li>
    <ul>
    <li>API 기반. HttpMessageConverter로 변환됨 </li>
    <li>Jackson 라이브러리로 찍어내듯이 동작 (각 필드에 각각 대입한다기 보다는 객체를 찍어내듯이 한번에 변환) </li>
    <li>@RequestBody, @ResponseBody, HttpEntity, …  </li>
    </ul>
<li><strong>@ModelAttribute</strong></li>
    <ul>
    <li>Client ↔ Server 기반. ConversionService로 변환 동작 </li>
    <li>한번에 찍어내듯이 객체가 생성되는 것이 아닌, 객체의 필드에 각각 대입하면서 객체가 생성됨 </li>
    <li>@RequestParam, @ModelAttribute, @PathVariable, Veiw Template Rendering, … </li>
    </ul>
</ul>
</aside>