---
title: "스프링 소셜과 Oauth2"
date: 2019-03-26 11:20:00 -0400
categories: java spring
---

## 개요
스프링 레시피 책의 6장의 주 내용은 스프링 소셜을 사용하여 트위터와 페이스북 같은 소셜 네트워크의 API를 사용하여 사용자 정보를 가져오고, 이를 활용하여 소셜 계정으로 로그인을 구현하는 것이다. 하지만 스프링 소셜을 이용하여 사용자 계정을 가져오려 하니 예상하지 못한 문제가 많이 발생하였고, 스프링 소셜의 메커니즘도 잘 이해하기 힘들었다. 그렇게 스프링 소셜을 사용하여 사용자 계정정보를 가져오는 서비스를 구현하려고 노력하던 중, 대부분의 소셜에서 Oauth2 방식의 인증/인가 서비스를 제공한 다는 것을 알게되었다. 그래서 소셜로부터 사용자 계정을 가져오는 샘플 코드를 Oauth2 방식으로 제작하게 되었다.


## 스프링 소셜을 사용해 트위터와 페이스북의 사용자 정보 가져오기

*스프링 소셜 모듈*

| 모듈 | 설명 |
| ----------- | ---------------- |
| spring-social-core | 스프링 소셜의 코어 모듈. 메인/공유 인프라 클래스가 들어 있습니다. |
| spring-social-config | 스프링 소셜의 구설 모듈. 스프링 소셜(또는 일부)을 쉽게 구설할 수 있습니다. | 
| spring-social-web | 스프링 소셜의 웹 연계 모듈. 간편하게 사용할 수 있는 필터 및 컨트롤러가 있습니다. |
| spring-social-security | 스프링 시큐리티 연계 모듈(7장) |
| spring-social-twitter | 스프링 소셜의 트위터 모듈 |
| spring-social-facebook | 스프링 소셜의 페이스북 모듈 |

@Configuration 구성 클래스에 @EnableSocial을 붙이면 스프링 소셜 기능이 켜지고 SicialConfigurer 빈이 있으면 자동으로 감지합니다. SocialConfigurer는 하나 이상의 서비스 공급자 구성을 추가할 때 쓰는 빈입니다. SocialConfigurer 인터페이스 구현체인 SocialConfigurerAdapter 추상클래스의 구현체를 작성하여 스프링 소셜에 대한 설정을 할 수 있습니다. 스프링 소셜에서는 UserIdSource 인스턴스로 현재 유저를 식별하고 이 유저를 이용해 서비스 공급자에 접속 간으한 커넥션을 찾습니다. 이렇게 찾은 커넥션은 유저별로 ConnectionRepository에 보관합니다. 어떤 ConnectionRepository를 골라쓸지는 현재 유저에 해당하는 UsersConnecetionRepository 인터페이스에 의해 결정됩니다.

- 구성 클래스에서 @EnableSocial 어노테이션을 사용해 SocialConfigurer 인터페이스를 빈으로 등록한다.
- 유저별로 소셜에 접근한 커넥션을 보관하는 ConnectionRepository 제어하는 ConnectionController를 구성합니다.
- SocialConfigurer 빈의 addConnectionFactories 메소드를 사용해 소셜의 [AppId]와 [AppSecret]을 등록합니다.
- 특정 소셜의 명령들을 모아놓은 인터페이스(Twitter, Facebook)을 빈으로 등록합니다. ConnectionRepository를 매개변수로 받아 현재 유저 ID에 따라 정해진 접속 정보를 가져올 수 있도록 합니다.

*스프링 소셜 구성 설정*
```java
@Configuration
@EnableSocial
public class SocialConfig extends SocialConfigurerAdapter {

    @Bean
    public ConnectController connectController(ConnectionFactoryLocator connectionFactoryLocator,
                                               ConnectionRepository connectionRepository) {
        return new ConnectController(connectionFactoryLocator, connectionRepository);
    }

    @Override
    public UserIdSource getUserIdSource() {
        return () -> "100023590687841"; // 페이스북 fid,
    }


    @Configuration
    public static class TwitterConfigurer extends SocialConfigurerAdapter {
        @Override
        public void addConnectionFactories(ConnectionFactoryConfigurer connectionFactoryConfigurer, Environment env) {
            TwitterConnectionFactory twitterConnectionFactory = new TwitterConnectionFactory(
                    "app_id",
                    "app_secret");

            connectionFactoryConfigurer.addConnectionFactory(twitterConnectionFactory);
        }

        @Bean
        @Scope(value = "request", proxyMode = ScopedProxyMode.INTERFACES)
        public Twitter twitterTemplate(ConnectionRepository connectionRepository) {
            Connection<Twitter> connection = connectionRepository.findPrimaryConnection(Twitter.class);
            return connection != null ? connection.getApi() : null;
        }
    }

    @Configuration
    public static class FacebookConfiguration extends SocialConfigurerAdapter {

        @Override
        public void addConnectionFactories(ConnectionFactoryConfigurer connectionFactoryConfigurer, Environment env) {
            connectionFactoryConfigurer.addConnectionFactory(
                    new FacebookConnectionFactory("app_id", "app_secret"));
        }

        @Bean
        @Scope(value = "request", proxyMode = ScopedProxyMode.INTERFACES)
        public Facebook facebookTemplate(ConnectionRepository connectionRepository) {
            Connection<Facebook> connection = connectionRepository.findPrimaryConnection(Facebook.class);
            return connection != null ? connection.getApi() : null;
        }
    }
}
```

*스프링 소셜을 사용하여 사용자 정보 얻어오기*
```java
@Controller
public class SocialController {

    @Autowired
    private ConnectionRepository repository;

    @GetMapping(value="/twitter")
    @ResponseBody
    public String twitter(Model model) {
        Connection<Twitter> connection = repository.getPrimaryConnection(Twitter.class);
        model.addAttribute("profile", connection.fetchUserProfile());
        return "twitter-info";
    }

    @GetMapping(value="/facebook")
    @ResponseBody
    public String facebook(Model model) {
        Connection<Facebook> connection = repository.getPrimaryConnection(Facebook.class);
        model.addAttribute("profile", connection.fetchUserProfile());
        return "twitter-info";
    }
}
```

***Twitter/Facebook의 bean 사용 버그와 같은 각종 버그와 스프링 소셜 매커니즘 이해 실패로 스프링 소셜로 사용자 정보를 얻어오기 <del>실패....</del>***


## 대안! Oauth2 방식으로 구글의 사용자 정보를 얻어오기

[참고 스프링 부트 Oauth2 튜토리얼 중 Manual 부분](https://spring.io/guides/tutorials/spring-boot-oauth2/#_social_login_manual)

[참고 cheese10yun님의 GitHub](https://github.com/cheese10yun/spring-security-oauth2-social)

### 구현 절차
1. 구글에 프로젝트를 등록하고 사용자 인증정보를 받아온다.
2. 프로젝트에 사용자 인증을 생성하고 Rediret URL을 설정한다.
3. 스프링 OAuth2에서 제공하는 OAuth2ClientAuthenticationProcessingFilter을 상속 받아 특정 소셜의 필터를 만든다.
4. FilterRegistrationBean을 빈으로 등록하고 스프링 시큐리티 설정을 이용해 소셜 필터를 필터 저장소에 등록한다.

*의존성 설정*
```xml
<dependency>
    <groupId>org.springframework.security.oauth</groupId>
    <artifactId>spring-security-oauth2</artifactId>
    <version>2.3.5.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.security.oauth.boot</groupId>
    <artifactId>spring-security-oauth2-autoconfigure</artifactId>
    <version>2.1.3.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

*Oauth2 클라이언트 필터 구현*
```java
public class GoogleOAuth2Client extends OAuth2ClientAuthenticationProcessingFilter {

    private ObjectMapper mapper = new ObjectMapper();

    public GoogleOAuth2Client() {
        super("/login/google"); // 현자 앱에서 구글에 사용자 정보를 요청하는 API, 구글의 리다이렉션URL 등록도 이 URL로 해주어야 한다.
    }

    @Override
    protected void successfulAuthentication(HttpServletRequest request, HttpServletResponse response, FilterChain chain, Authentication authResult) throws IOException, ServletException {

        final OAuth2AccessToken accessToken = restTemplate.getAccessToken();
        final OAuth2Authentication auth = (OAuth2Authentication) authResult;
        final Object details = auth.getUserAuthentication().getDetails();

        final GoogleUserDetails userDetails = mapper.convertValue(details, GoogleUserDetails.class);
        userDetails.setAccessToken(accessToken);
        System.out.println(userDetails.toString()); // 사용자 정보를 제대로 받아오는지 확인
        super.successfulAuthentication(request, response, chain, ((OAuth2Authentication) authResult).getUserAuthentication());
    }
}
```

*스프링 시큐리티에 구글 Oauth2필터 등록*
```java
@EnableWebSecurity
@EnableOAuth2Client
public class Oauth2Config extends WebSecurityConfigurerAdapter {

    @Autowired
    private OAuth2ClientContext oauth2ClientContext;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable();

        http.antMatcher("/**").authorizeRequests().antMatchers("/**").permitAll().anyRequest()
                .authenticated().and().exceptionHandling()
                .authenticationEntryPoint(new LoginUrlAuthenticationEntryPoint("/")).and()
                .addFilterBefore(googleSsoFilter(), BasicAuthenticationFilter.class);

        http.logout()
                .invalidateHttpSession(true)
                .clearAuthentication(true)
                .logoutRequestMatcher(new AntPathRequestMatcher("/logout"))
                .logoutSuccessUrl("/")
                .permitAll();
    }

    @Bean
    public FilterRegistrationBean oauth2ClientFilterRegistration(OAuth2ClientContextFilter filter) {
        FilterRegistrationBean registration = new FilterRegistrationBean();
        registration.setFilter(filter);
        registration.setOrder(-100);
        return registration;
    }

    private Filter googleSsoFilter() {
        OAuth2RestTemplate restTemplate = new OAuth2RestTemplate(google().getClient(), oauth2ClientContext);
        GoogleOAuth2ClientFilter filter = new GoogleOAuth2ClientFilter();
        filter.setRestTemplate(restTemplate);
        UserInfoTokenServices tokenServices = new UserInfoTokenServices(google().getResource().getUserInfoUri(), google().getClient().getClientId());
        filter.setTokenServices(tokenServices);
        tokenServices.setRestTemplate(restTemplate);
        return filter;
    }

    @Bean
    @ConfigurationProperties("google")
    public ClientResources google() {
        return new ClientResources();
    }
}
```

[샘플코드 GitHub : Oauth2를 사용하여 구글의 사용자 정보 얻어오기](https://github.com/firewood3/spring/tree/master/spring-oauth/springboot-oauth2-client)




***  
참고도서    
제목: 스프링5레시피(4판)  
지은이: 마틴데니엄, 다니엘 루비오, 조시 롱  
옮긴이: 이일웅  
펴낸곳: 한빛미디어  
