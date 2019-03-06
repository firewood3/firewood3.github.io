---
title: "스프링 코어에 대하여"
date: 2019-03-06 21:50:00 -0400
categories: java
---

## 스프링 코어 대하여
  - 스프링 코어
    - 스프링 IoC 컨테이너가 빈들을 관리(생성에서 파괴까지)
    - 스프링 IoC 컨테이너는 DI를 사용하여 애플리케이션 컴포넌트를 관리
    - 애플리케이션 컴포넌트란 @Component, @Repository, @Service, @Controller 등을 붙힌 자바 객체
    - 스프링 IoC 컨테이너는 애플리케이션 컴포넌트들을 스캐닝하여 애플리케이션의 일부인 것처럼 빈을 구성
  - 빈(Bean)
    - 스프링 IoC 컨테이너에 의해 관리 되는 객체
    - 스프링 문서에는 빈과 POJO 인스턴스를 같은 의미로 혼용
  - 스프링 IoC 컨테이너 인터페이스
    - BeanFactory
    - ApplicationContext
  - 스프링 IoC 컨테이너의 구현체 예시
    - ClasspathXmlApplicationContext
    - AnnotationConfigApplicationContext
  - 의존성 주입
    - 강한 결합
        ```java
            public class TextEditor {
                private SpellChecker spellChecker;

                public TextEditor() {
                    spellChecker = new SpellChecker();
                }
            }
        ```
    - 약한 결합(제어 역전)
        ```java
            public class TextEditor {
                private SpellChecker spellChecker;
   
                public TextEditor(SpellChecker spellChecker {
                    this.spellChecker = spellChecker;
                }
            }
        ```
  
[참고: 스프링 Ioc 컨테이너](https://www.tutorialspoint.com/spring/spring_ioc_containers.htm)  
[참고: 빈(Bean) 정의](https://www.tutorialspoint.com/spring/spring_bean_definition.htm)  
[참고: 의존성 주입](https://www.tutorialspoint.com/spring/spring_dependency_injection.htm)
