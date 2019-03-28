---
title: "Springboot + spring security + h2 + jpa + thymeleaf로 URL 접근 보안 구현하기"
date: 2019-03-28 17:30:00 -0400
categories: java spring security
---

## 시작하기
이 글을 SpringBoot와 Spring Security5를 사용하여 URL 접근을 보호하고 사용자 로그인, 로그아웃을 구현하는 것을 설명한다.

*보안 기본 용어*

| 용어 | 설명 |
| ----------- | ---------------- |
| 인증(Authentication) | 인증은 주체의 신원을 증명하는 과정을 말한다. |
| 주체(Principal) | 주체는 해당 리소스에 접근하고자 하는 사용자를 말한다. |
| 신원정보(Credential) | 신원정보는 주체가 리소스를 얻기위해 제시하는 것으로서 일반적으로 비밀번호이다. |
| 인가(Authorization) | 인가는 인증을 마친 주체에게 권한(Authority)을 부여하여 특정 리소스에 접근할 수 있게 허가하는 과정을 말한다. |
| 접근 통제(Access controll) | 접근통제는 리소스에 접근하는 행위를 제어하는 것을 말한다. 어떤 유저가 어떤 리소스에 접근하도록 허락할지를 결정하는 행위, 즉 접근 통제 결정(Access controll decision)이 뒤따른다. |

프로젝트에 사용될 의존성은 다음과 같다.

*Dependency*
- spring-boot-starter-data-jpa
- spring-boot-starter-security
- spring-boot-starter-thymeleaf
- thymeleaf-extras-springsecurity5
- spring-boot-starter-web
- h2
- lombok

스프링 시큐리티에서는 @EnableWebSecurity 어노테이션과 WebSecurityConfigurerAdapter 추상클래스를 상속하여 웹 애플리케이션 보안을 구성한다. 

*WebSecurityConfigurerAdapter의 기본 보안 설정 목록*

| 기본 보안 | 설명 |
| ----------- | ---------------- |
| 폼 기반 로그인 서비스(form-based login service) | 유저가 애플리케이션에 로그인하는 기본 폼 페이지를 제공합니다. |
| HTTP 기본 인증(Basic authentication) | 요청 헤더에 표시된 HTTP 기본 인증 크레덴셜을 처리합니다. 원격 프로토콜, 웹 서비스를 이용해 인증 요청을 할 때에도 쓰입니다. |
| 로그아웃 서비스 | 유저를 로그아웃 시키는 핸들러를 기본 제공합니다. |
| 익명 로그인(anonymous login) | 익명 유저도 주체를 할당하고 권한을 부여해서 마치 일반 유저처럼 처리합니다. |
| 서블릿 API 연계 | HttpServletRequest.isUserInRole(), HttpServletRequest.getuserPrincipal() 같은 표준 서블릿 API를 이용해 웹 애플리케이션에 위치한 보안 정보에 접근합니다. |
| CSFR | 사이트 간 요청 위조 방어용 토큰을 생성해 HttpSession에 넣습니다. |
| 보안 헤더 | 보안이 적용된 패키지에 대해서 캐시를 해제하는 식으로 XSS 방어, 전송 보안(transfer security), X-Frame 보안 기능을 제공합니다. |


## URL 접근 제어하기
HttpSecurity의 authorizeRequests 메소드를 이용하여 특정 리소스(URL)의 접근 통제를 할 수 있다. 접근 리소스를 등록해두면 해당 요청이 올때마다 인증 여부를 확인한다. 아래 예제에서는 USER와 ADMIN이라는 권한으로 각각 /user와 /admin 리소스에 접근할 수 있도록 하고있다.

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
            .antMatchers("/anyone").permitAll()
            .antMatchers("/").permitAll()
            .antMatchers("/user").hasAnyRole("USER")
            .antMatchers("/admin").hasAnyRole("ADMIN")
            .antMatchers("/h2-console/**").hasAnyRole("ADMIN")
            .and()
    }
}
```

## 로그인과 로그아웃 요청 처리하기
HttpSecurity에 formLogin()을 설정하면 스프링 시큐리티에서 제공하는 로그인 폼이 자동으로 생성된다. 로그인 페이지를 커스텀하게 사용하려면 formLogin().loginPage("url")을 사용하면된다. loginProcessingUrl("url")메서드를 사용하여 인증 주체가 권한을 얻기 위한 URL을 지정한다. 이 URL의 메서드는 POST이다.

로그아웃을 사용하기 위해서는 logout()메서드를 사용한다. logoutUrl("url")을 사용하여 로그아웃 URL을 지정해 줄 수 있다. 로그인 URl과 마찬가지로 메서드는 POST이다.

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
        http
            ...
            .and()
            .formLogin()
            .loginProcessingUrl("/login")
            .loginPage("/")
            .usernameParameter("memberId")
            .passwordParameter("password")
            .and()

            .logout()
            .logoutUrl("/logout")
            .logoutSuccessUrl("/")
            .and()
    ;
}
```

## CSFR 공격방어
CSFR 방어기능은 CSFR 공격에 노출된 위험을 에방해준다. .csrf().disable() 메서드를 사용하여 CSFR 방어기능을 해제할 수도 있다.

그리고 로그인과 로그아웃 시에 CSFR을 위해 발급한 토큰을 전송해주는 코드를 사용자쪽에서도 구현해주어야 한다.

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
        http
            ...
            .and()
            .csrf()
    ;
}
```

```html
<input type="hidden" th:name="${_csrf.parameterName}" th:value="${_csrf.token}" />
```

## 유저 인증하기
유저를 만들어 해당 유저가 특정 리소스에 접근할 수 있도록 하자.

```java
@Data
@Entity
public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String memberId;
    private String password;
    private String memberRole;
    private String email;
    private String phoneNumber;
}
```

스프링 시큐리티에서는 AuthenticationManager를 이용해 인증을 수행한다. 위에서 정의한 Member 객체로 인증을 하기 위해서는 다음과 같이 AuthenticationManagerBuilder를 사용하고, UserDetailsService 인터페이스 구현체를 생성해야하는데, UserDetailsService는 Member 객체를 로드해 오는 일을 한다.


```java
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.userDetailsService(userDetailsService);
}
```

```java
@Service
public class UserDetailServiceImpl implements UserDetailsService {

    private final MemberRepository memberRepository;

    @Autowired
    public UserDetailServiceImpl(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }

    @Override
    public UserDetails loadUserByUsername(String memberId) {
        Member member = memberRepository.findMemberByMemberId(memberId);
        if (ObjectUtils.isEmpty(member))
            throw new UsernameNotFoundException("아이디 또는 비밀번호를 확인해 주세요.");
        return new AuthorityMember(member);
    }
}
```

## 비밀번호 암호화 하기
Member 객체에 저장된 비밀번호가 암호화 되어 있다면, 사용자로부터 인증요청을 받을때 받아온 비밀번호도 같은 형식으로 암호화하여 매칭해야한다. AuthenticationManager가 특정 암화화 알고리즘으로 비밀번호를 암호화 할 수 있도록 설정할 수 있다.

```java
@Bean
public PasswordEncoder bCryptPasswordEncoder() {
    return new BCryptPasswordEncoder(11);
}
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.userDetailsService(userDetailsService)
            .passwordEncoder(bCryptPasswordEncoder());
}
```


이번 글에 사용된 프로젝트는 [GitHub](https://github.com/firewood3/spring/tree/master/spring-security/springboot-security-login)에서 확인할 수 있다. 샘플코드에서 암호화된 비밀번호는 1111이다.

***  

참고문서  
[Thymeleaf + Spring Security integration basics](https://www.thymeleaf.org/doc/articles/springsecurity.html)  
[CSRF Protection with Spring MVC and Thymeleaf](https://www.baeldung.com/csrf-thymeleaf-with-spring-security)

참고도서    
제목: 스프링5레시피(4판)  
지은이: 마틴데니엄, 다니엘 루비오, 조시 롱  
옮긴이: 이일웅  
펴낸곳: 한빛미디어  
