---
title: "스프링 코어"
date: 2019-03-18 11:15:00 -0400
categories: java spring
---

## @Autoriewd를 사용한 자동 주입
- 인스턴스 변수에 바로 @Autowired 붙히기
- 생성자에 @Autoriewd 붙히기
- Setter에 @Autoried 붙히기


## 타입이 같은 Bean을 자동 연결하기 위한 어노테이션
- @Primary
- @Qulifier("name")


## 자동 주입을 위한 자바 표준 어노테이션
- @Resource
- @Inject


## Bean의 범위를 제어하는 @Scope("scope_name")
- singleton
- prototype
- request
- session
- globalSession

***POJO 스코프***
```java
@Component
@Scope("singleton")
public class BeanPOJO {
}

@Component
@Scope("prototype")
public class BeanPOJO {
}
```

## POJO에서 외부 리소스를 사용하기 위한 어노테이션
- @PropertySource("classpath:file_name")
- @Value("${key:default_value}")
- @Value("classpath:file_name")


***외부 리소스(data.properties)***
```code
value=100
```


***외부 리소스를 불러오는 POJO***
```java
@Component
@PropertySource("classpath:data.properties")
public class BeanPOJO {

    @Value("${value:0}")
    private int value;

    @PostConstruct
    public void post() {
        System.out.println(value);
    }
}
```


## 다국어 메시지를 불러오기 위해 MessageSource를 빈으로 등록하기
- MessageSource 인터페이스
- ResourceBundleMessageSource 구현체
- messages_[language]_[country].properties
- MessageSource의 getMessage(key, locale) 메소드


***message_ko_KR 프로퍼티***
```code
hello=안녕
```


***ResourceBundleMessageSource를 빈으로 등록하기***
```java
@Configuration
public class LanguageConfiguration {
    @Bean
    public ReloadableResourceBundleMessageSource messageSource() {
        ReloadableResourceBundleMessageSource messageSource = new ReloadableResourceBundleMessageSource();
        messageSource.setBasenames("classpath:messages");
        messageSource.setCacheSeconds(1);
        return messageSource;
    }
}
```


***스프링 IoC 컨테이너에서 message가져오기***
```java
    public static void main(String[] args) {
        ApplicationContext context = new AnnotationConfigApplicationContext("com.firewood");
        ReloadableResourceBundleMessageSource messageSource = context.getBean(ReloadableResourceBundleMessageSource.class);
        String sayHello = messageSource.getMessage("hello", null, Locale.KOREA);
        System.out.println(sayHello);
    }
```


## Bean의 초기화와 폐기
- @Configuration 클래스에서 @Bean(initMethod = "method_name", destroyMethod = "method_name")
- @PostConstruct와 @PreDestroy
- @Lazy
- @DependsOn({"depended_beans_name"})


***init, destroy, Lazy, depend POJO***
```java
@Component
@Lazy
@DependsOn("dependedPOJO")
public class BeanPOJO {
    @PostConstruct
    public void postConstruct() {
        // 빈으로 등록된 후 호출
    }

    @PreDestroy
    public void preDestroy() {
        // 빈 파괴 전 호출
    }
}
```

## 모든 Bean의 생성 시점 전/후를 감지하기
- BeanPostProcessor 인터페이스를 Bean으로 구현
- BeanPostProcessor 인터페이스의 postProcessBeforeInitializtion 메소드
- BeanPostProcessor 인터페이스의 postProcessAfterInitializtion 메소드

***BeanPostProcessor를 구현한 POJO***
```java
@Component
public class CheckBeans implements BeanPostProcessor {
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("before : " + beanName);
        return bean;
    }

    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("after : " + beanName);
        return bean;
    }
}
```

## 스프링 IoC 컨테이너 설정을 통해 여러 POJO 클래스 중 Bean으로 만들 POJO 클래스 선택하기
- @Profiles("profile_name")
- ApplicationContext의 setActiveProfiles("string... profiles") 메소드


***GlovalPOJO와 LocalPOJO 생성***
```java
@Component
@Profile("global")
public class GlobalPOJO {
    @PostConstruct
    public void init() {
        System.out.println("global");
    }
}
@Component
@Profile("local")
public class LocalPOJO {

    @PostConstruct
    public void init() {
        System.out.println("local");
    }
}
```


***스프링 IoC 컨테이너에 프로파일 환경 설정***
```java
    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
        context.getEnvironment().setActiveProfiles("global");
        context.scan("com.firewood");
        context.refresh(); // global 출력
    }
```


## POJO 클래스에서 스프링 IoC 컨테이너의 리소스 가져오기
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
| EnvironmentAware | 어플리케이션 컨텍스트의 Environment 인스턴스 |


***POJO에서 ApplicationContext가져오기***
```java
@Component
public class ContainerResourcePOJO implements ApplicationContextAware {
    private ApplicationContext applicationContext;

    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
}
```


## TaskExecutor로 동시성 프로그래밍 하기
- TaskExecutor 구현체를 빈으로 등록하고 POJO에서 TaskExecutor 구현체를 주입받아 사용가능
- TaskExecutor 구현체의 종류
    - syncTaskExecutor
    - SimpleAsyncTaskExecutor
    - ThreadPoolTaskExecutor


***TaskExecutor 구현체 등록***
```java
@Configuration
public class ConcurrencyConfiguration {
    @Bean
    public ThreadPoolTaskExecutor threadPoolTaskExecutor(){
        ThreadPoolTaskExecutor taskExecutor = new ThreadPoolTaskExecutor();
        taskExecutor.setCorePoolSize(50);
        taskExecutor.setMaxPoolSize(100);
        taskExecutor.setAllowCoreThreadTimeOut(true);
        taskExecutor.setWaitForTasksToCompleteOnShutdown(true);
        return taskExecutor;
    }
}
```


***POJO에서 받아 사용***
```java
@Component
public class ConcurrencyPOJO {

    @Autowired
    private ThreadPoolTaskExecutor taskExecutor;

    public void executeRunnable(Runnable runnable) {
        taskExecutor.execute(runnable);
    }
}
```

## POJO끼리 Event기반으로 통신하기
- 이벤트 : 이벤트는 ApplicationEvent를 상속받아 구현
- 이벤트 발신 POJO : 이벤트를 전송할 POJO에서 ApplicationEventPublisherAware 인터페이스를 구현하거나 스프링 IoC 컨테이너에서 제공하는 applicationEventPublisher 빈을 상속받아 publishEvent('event_type') 메소드를 실행하여 이벤트를 발신
- 이벤트 수신 POJO : 이벤트를 수신할 POJO에서 ApplicationList<'event_type'> 인터페이스를 구현하거나 @EventListener 어노테이션을 사용한 메소드를 정의하여 이벤트를 수신

***이벤트***
```java
public class HelloEvent extends ApplicationEvent {
    @Getter
    private final String message;
    public HelloEvent(String message) {
        super(message);
        this.message = message;
    }
}
```


***이벤트 발신자***
```java
@Component
public class Student {

    @Autowired
    private ApplicationEventPublisher applicationEventPublisher;

    public void sendHelloMessage() {
        HelloEvent helloEvent = new HelloEvent("hi");
        applicationEventPublisher.publishEvent(helloEvent);
    }
}
```


***이벤트 수신자***
```java
@Component
public class Teacher {

    @EventListener
    public void onApplicationEvent(HelloEvent helloEvent) {
        System.out.println("message : [" + helloEvent +"]");
    }
}
```


***
참고도서  
제목: 스프링5레시피(4판)  
지은이: 마틴데니엄, 다니엘 루비오, 조시 롱  
옮긴이: 이일웅  
펴낸곳: 한빛미디어  
