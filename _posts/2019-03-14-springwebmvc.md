---
title: "스프링 MVC"
date: 2019-03-14 16:33:00 -0400
categories: java spring
---

## @RequestMapping을 사용하여 요청 매핑하기
- 메서드에 @RequestMapping 사용
- 클래스에 @RequestMapping 사용
- Http 요청 매핑 어노테이션 사용
    - @GetMapping
    - @PostMapping
    - @PutMapping
    - @DeleteMapping

***@RequestMapping 어노테이션 사용***
```java
@Controller
@RequestMapping("/member")
public class MemberController {

    private final MemberService memberService;

    @Autowired
    public MemberController(MemberService memberService) {
        this.memberService = memberService;
    }

    @RequestMapping(value = "/get/list", method = RequestMethod.GET)
    public String getMember(Model model) {
        model.addAttribute("members", memberService.getMemberList());
        return "memberList";
    }
}
```


## 인터셉터로 요청 가로채기
- HTTP 요청을 가로채 전/후처리 가능
- HandlerInterceptor 인터페이스를 빈으로 등록
- 등록된 인터셉터 빈을 WebMvc의 InterceptorRegistry에 등록
- 기본적으로 인터셉터는 모든 요청을 가로채지만 특정 URL만 가로채도록 지정 가능

***HandlerInterceptor POJO 구현***
```java
public class MemberInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("interceptor : pre");
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("interceptor : post");
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("interceptor : after");
    }
}
```
***InterceptorRegistry에 등록***
```java
@Configuration
public class InterceptorHandler implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(measurementInterceptor())
                .addPathPatterns("/member/get/list*");
    }

    @Bean
    public MemberInterceptor measurementInterceptor() {
        return new MemberInterceptor();
    }
}
```

 
## 유저 로케일 해석하기
- LocaleResolver 인터페이스
    - AcceptHeaderLocaleResolver : 브라우저의 Locale 값을 읽어옴, 기본 LocalResolver, 로케일 변경 불가
    - SessionLocaleResolver : Session에 설정되어있는 Locale값을 읽어옴, 최초 세션 생성시 디폴트 로케일 지정가능
    - CookieLocaleResolver : 브라우저 쿠키의 Locale 값을 읽어옴, 최초 쿠키에 디폴트 로케일 지정가능
    - 유저 로케일은 HandlerMapping에서 매개변수로 받을 수 있음
- 특수 인터셉터로 유저 로케일 변경하기
    - LocaleChangeInterceptor 인터페이스를 빈으로 등록
    - URL 파라미터로 로케일을 변경할 수 있는 특수한 인터셉터
    - AcceptHeaderLocaleResolver는 사용불가, SessionLocaleResolver와 CookieLocaleResolver만 사용가능
    - http://localhost:8080/member/get/list?language="locale(de, en_US)"
    

***상황에 맞는 로컬 리졸버 등록***
```java
@Configuration
public class LocaleConfiguration {

    @Bean
    public AcceptHeaderLocaleResolver localeResolver() {
        AcceptHeaderLocaleResolver acceptHeaderLocaleResolver = new AcceptHeaderLocaleResolver();
        acceptHeaderLocaleResolver.setDefaultLocale(new Locale("en"));
        return acceptHeaderLocaleResolver;
    }

    @Bean
    public CookieLocaleResolver localeResolver() {
        CookieLocaleResolver cookieLocaleResolver = new CookieLocaleResolver();
        cookieLocaleResolver.setCookieName("language");
        cookieLocaleResolver.setCookieMaxAge(3600);
        cookieLocaleResolver.setDefaultLocale(new Locale("en"));
        return cookieLocaleResolver;
    }

    @Bean
    public SessionLocaleResolver localeResolver() {
        SessionLocaleResolver localeResolver = new SessionLocaleResolver();
        localeResolver.setDefaultLocale(new Locale("en"));
        return localeResolver;
    }
}
```

***LocaleChangeInterceptor를 등록하여 로케일 변경하기***
```java
@Configuration
public class LocaleConfiguration implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(localeChangeInterceptor());
    }

    @Bean
    public LocaleChangeInterceptor localeChangeInterceptor() {
        LocaleChangeInterceptor localeChangeInterceptor = new LocaleChangeInterceptor();
        localeChangeInterceptor.setParamName("language");
        return localeChangeInterceptor;
    }

    @Bean
    public CookieLocaleResolver localeResolver() {
        CookieLocaleResolver cookieLocaleResolver = new CookieLocaleResolver();
        cookieLocaleResolver.setCookieName("language");
        cookieLocaleResolver.setCookieMaxAge(3600);
        cookieLocaleResolver.setDefaultLocale(new Locale("en"));
        return cookieLocaleResolver;
    }
}
```


***HandlerMapping에서 Locale 가져오기***
```java
@RequestMapping(value = "/get/list", method = RequestMethod.GET)
public String getMember(Model model, Locale locale) {
    System.out.println(locale.toString());
    model.addAttribute("members", memberService.getMemberList());
    return "memberList";
}
```


## 여러가지 뷰 리졸버 등록 방법
- InternalResourceViewResolver
    - prefix
    - suffix
- XMLViewResolver
    - xml 파일로 뷰 빈 구성
- ResourceBundleViewResolver
    - properties 파일로 뷰 빈 구성
- 여러 뷰 리졸버를 동시에 등록할 때는 뷰리졸버의 우선순위를 정함

***InternalResourceViewResolver 등록***
```java
@Configuration
public class ViewResolverConfiguration {
    @Bean
    public InternalResourceViewResolver internalResourceViewResolver() {
        InternalResourceViewResolver viewResolver = new InternalResourceViewResolver();
        viewResolver.setPrefix("/WEB-INF/jsp/");
        viewResolver.setSuffix(".jsp");
        return viewResolver;
    }
}

```

***XMLViewResolver 등록***
```java
@Configuration
public class ViewResolverConfiguration {
    private final ResourceLoader resourceLoader;

    public ViewResolverConfiguration(ResourceLoader resourceLoader) {
        this.resourceLoader = resourceLoader;
    }

    @Bean
    public XmlViewResolver viewResolver() {
        XmlViewResolver viewResolver = new XmlViewResolver();
        viewResolver.setLocation(resourceLoader.getResource("/WEB-INF/court-views.xml"));
        return viewResolver;
    }
}
```
```xml
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns="http://www.springframework.org/schema/beans"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="welcome" class="org.springframework.web.servlet.view.JstlView">
        <property name="url" value="/WEB-INF/jsp/welcome.jsp"/>
    </bean>

    <bean id="reservationQuery" class="org.springframework.web.servlet.view.JstlView">
        <property name="url" value="/WEB-INF/jsp/reservationQuery.jsp"/>
    </bean>

    <bean id="welcomeRedirect" class="org.springframework.web.servlet.view.RedirectView">
        <property name="url" value="welcome"/>
    </bean>
</beans>

```

***ResourceBundleViewResolver 등록***
```java
@Configuration
public class ViewResolverConfiguration {
    @Bean
    public ResourceBundleViewResolver viewResolver() {
        ResourceBundleViewResolver viewResolver = new ResourceBundleViewResolver();
        viewResolver.setBasename("court-views");
        return viewResolver;
    }
}
```
```code
welcome.(class)=org.springframework.web.servlet.view.JstlView
welcome.url=/WEB-INF/jsp/welcome.jsp
reservationQuery.(class)=org.springframework.web.servlet.view.JstlView
reservationQuery.url=/WEB-INF/jsp/reservationQuery.jsp
welcomeRedirect.(class)=org.springframework.web.servlet.view.RedirectView
welcomeRedirect.url=welcome

```

***뷰 리졸버 우선순위 등록***
```java
@Configuration
public class ViewResolverConfiguration {

    @Bean
    public ResourceBundleViewResolver viewResolver() {
        ResourceBundleViewResolver viewResolver = new ResourceBundleViewResolver();
        viewResolver.setOrder(0);
        viewResolver.setBasename("court-views");
        return viewResolver;
    }

    @Bean
    public InternalResourceViewResolver internalResourceViewResolver() {
        InternalResourceViewResolver viewResolver = new InternalResourceViewResolver();
        viewResolver.setOrder(1);
        viewResolver.setPrefix("/WEB-INF/jsp/");
        viewResolver.setSuffix(".jsp");
        return viewResolver;
    }
}
```


## 뷰 예외 페이지 매핑하기
- HandlerMapping 메소드에서 특정 런타임 예외가 발생할때 미리 지정된 뷰를 내려주는 기술
- HandlerExceptionResolver 인터페이스를 빈으로 등록하여 WebMvc에 등록 하는 방법
    - properties로 예외 클래스와 예외 패이지 매핑
    - HandlerMapping 메소드에서 해당 예외 발생시 연결된 예외 페이지를 내려줌
- @ExceptionHanlder 어노테이션으로 예외 매핑 하는 방법

***HendlerExceptionResolver 사용***
```java
@Configuration
public class ExceptionHandlerConfiguration implements WebMvcConfigurer {

    @Override
    public void configureHandlerExceptionResolvers(List<HandlerExceptionResolver> resolvers) {
        resolvers.add(handlerExceptionResolver());
    }

    @Bean
    public HandlerExceptionResolver handlerExceptionResolver() {
        Properties exceptionProperties = new Properties();
        exceptionProperties.setProperty(MemberException.class.getName(), "memberError");

        SimpleMappingExceptionResolver exceptionResolver = new SimpleMappingExceptionResolver();
        exceptionResolver.setExceptionMappings(exceptionProperties);
        exceptionResolver.setDefaultErrorView("error");
        return exceptionResolver;
    }
}

@Controller
@RequestMapping("/member")
public class MemberController {
    @GetMapping("/exception")
    public String exception() {
        throw new MemberException();
    }
}
```

***_@ExceptionHanlder 사용***
```java
@ControllerAdvice
public class MemberExceptionHandler {

    @ExceptionHandler(MemberException.class)
    public String memberExceptionHandle(MemberException ex) {
        return "memberError";
    }

    @ExceptionHandler
    public String defaultExceptionHandle(Exception e) {
        return "error";
    }
}
@Controller
@RequestMapping("/member")
public class MemberController {
    @GetMapping("/exception")
    public String exception() {
        throw new MemberException();
    }
}
```

## @Valid 어노테이션을 사용하여 검증하기
- HandlerMapping 메소드에서 @Valid 어노테이션을 사용하여 RequestBody를 검증 할 수 있음
- BindingResult : 검증 결과를 HandlerMapping 메소드의 매개변수로 받음
- 유효성 검증 어노테이션
    - @Min(value="", message="")
    - @Max(value="", message="")
    - @NotNull(message="")
    - @NotBlank(message="")
    - @Pattern(regexp="", message)

***유효성 검증하기***
```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Member {
    @Size(min = 3, message = "아이디는 6글자 이상이어야 합니다.")
    @Size(max = 10, message = "아이디는 10글자 이하이어야 합니다.")
    @NotNull(message = "아이디를 입력해 주세요.")
    @NotBlank(message = "아이디를 입력해 주세요.")
    @Pattern(regexp = "[A-Za-z0-9]{6,10}$", message = "아이디는 3글자 이상, 10글자 이하의 영문 및 숫자만 가능합니다.")
    private String id;
}

@PostMapping(value = "/create", produces = MediaType.APPLICATION_JSON_UTF8_VALUE)
public String create(@RequestBody @Valid Member member, BindingResult result, Model model) {
    if(result.hasErrors()) {
        result
            .getFieldErrors()
            .stream()
            .map(DefaultMessageSourceResolvable::getDefaultMessage)
            .collect(Collectors.toList())
            .forEach(System.out::println);
        return "memberError";
    } else {
        memberService.addMember(member);
        model.addAttribute("members", memberService.getMemberList());
        return "memberList";
    }
}
```


***
참고도서  
제목: 스프링5레시피(4판)  
지은이: 마틴데니엄, 다니엘 루비오, 조시 롱  
옮긴이: 이일웅  
펴낸곳: 한빛미디어  
