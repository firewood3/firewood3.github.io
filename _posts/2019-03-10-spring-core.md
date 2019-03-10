---
title: "스프링 코어"
date: 2019-03-18 11:15:00 -0400
categories: java spring
---

#### @Autoriewd를 사용한 자동 주입
- 인스턴스 변수에 바로 @Autowired 붙히기
- 생성자에 @Autoriewd 붙히기
- Setter에 @Autoried 붙히기


#### 타입이 같은 빈을 자동 연결하기 위한 어노테이션
- @Primary
- @Qulifier("name")


#### 자동 주입을 위한 자바 표준 어노테이션
- @Resource
- @Inject


#### 빈의 범위를 제어하는 @Scope("scope_name")
- singleton
- prototype
- request
- session
- globalSession


#### 빈 객체에서 외부 리소스를 사용하기 위한 어노테이션
- @PropertySource("classpath:file_name")
- @Value("${key:default_value}")
- @Value("classpath:file_name")


#### 외부 리소스를 가져오기 spring.core.io의 위한 객체
- ClasspathResource
- FileSystemResource
- UrlResource


#### MessageSource를 빈으로 등록해 다국어 메시지 불러오기
- MessageSource 인터페이스
- ResourceBundleMessageSource 구현체
- messages_[language]_[country].properties
- MessageSourcet의 getMessage 메소드
