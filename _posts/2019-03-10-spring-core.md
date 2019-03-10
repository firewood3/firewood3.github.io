---
title: "스프링 코어"
date: 2019-03-18 11:15:00 -0400
categories: java spring
---

### @Autoriewd를 사용한 자동 주입
- 인스턴스 변수에 바로 @Autowired 붙히기
- 생성자에 @Autoriewd 붙히기
- Setter에 @Autoried 붙히기


### 타입이 같은 빈을 자동 연결하기 위한 어노테이션
- @Primary
- @Qulifier("name")


### 자동 주입을 위한 자바 표준 어노테이션
- @Resource
- @Inject


### 빈의 범위를 제어하는 @Scope("scope_name")
- singleton
- prototype
- request
- session
- globalSession


### 빈 객체에서 외부 리소스를 사용하기 위한 어노테이션
- @PropertySource("classpath:file_name")
- @Value("${key:default_value}")
- @Value("classpath:file_name")


### 외부 리소스를 가져오기 spring.core.io의 위한 객체
- ClasspathResource
- FileSystemResource
- UrlResource


### MessageSource를 빈으로 등록해 다국어 메시지 불러오기
- MessageSource 인터페이스
- ResourceBundleMessageSource 구현체
- messages_[language]_[country].properties
- MessageSourcet의 getMessage 메소드


### Bean의 초기화와 폐기
- @Bean(initMethod = "method_name", destroyMethod = "method_name")
- @PostConstruct와 @PreDestroy
- @Lazy
- @DependsOn({"depended_beans_name"})


### 모든 Bean의 생성 시점 전/후를 감지하기
- BeanPostProcessor 인터페이스를 Bean으로 구현
- BeanPostProcessor 인터페이스의 postProcessBeforeInitializtion 메소드
- BeanPostProcessor 인터페이스의 postProcessAfterInitializtion 메소드


### 여러 POJO 클래스 중 Bean으로 만들 POJO 클래스 선택하기
- @Profiles("profile_name")
- ApplicationContext의 setActiveProfiles("string... profiles") 메소드


### POJO 클래스에서 스프링 IoC 컨테이너의 리소스 가져오기
- 목적에 맞는 Aware 인터페이스를 구현하여 사용
- Aware 인터페이스의 종류


| Aware 인터페이스 | 대상 리소스 |
| ----------- | ---------------- |
| BeanNameAware | 인스턴스의 빈 이름 |
| BeanFactoryAware | 현재 빈팩토리 | 
| ApplicationContextAware | 현재 어플리케이션 컨텍스트 |
| MessageSourceAware | 메시지 소스 |
| ApplicationEventPublisherAware | 이벤트 퍼블리셔 |
| ReourceLoaderAware | 리소스 로더  |
| EnvironmentAware | 어플리케이션 컨텍스트의 Environment 인스턴스   |

