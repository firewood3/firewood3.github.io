---
title: "간단한 커스텀 스프링부트 스타터 만들기"
date: 2019-04-03 11:52:00 -0400
categories: java sprig
---

스프링 부트의 'artifactId'는 spring-boot-starter-jpa, spring-boot-starter-webmvc 이런 식으로 spring-boot-starter로 시작되는데 이 starter라는 것은 AutoConfigurator를 기반으로 스프링 부트 애플리케이션에서 별도의 설정 없이 해당 모듈에서 지원하는 기능들을 편리하게 사용할 수 있도록 만든 모듈 규격이다. spring-boot-autoconfigure를 사용하면 커스텀하게 스프링부트 모듈을 제작하여 사용할 수 있다.

## 커스텀 스프링 부트 모듈 만들기

모듈로 제공할 프로젝트에 다음과 같이 spring-boot-autoconfigure 의존성을 추가하여 커스텀 모듈로 만들 수 있도록 한다.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-autoconfigure</artifactId>
</dependency>
```

PersonService라는 POJO클래스를 빈으로 등록하여 테스트해보자. PersonService는 Person을 생성할 수 있다.  

```java
public interface PersonService {
    Person getPerson();
}
```

```java
public class PersonServiceImpl implements PersonService{
    @Override
    public Person getPerson() {
        return new Person("alson", "from custom module") ;
    }
}
```

```java
@Configuration
public class CustomAutoConfiguration {
    @Bean
    public PersonService personService() {
        return new PersonServiceImpl();
    }
}
```

autoconfigure 클래스는 스프링 부트가 실행될때 classpath의 META-INF 하위의 spring.fatories라는 파일을 찾고, EnableAutoConfiguration 어노테이션을 사용하여 구성할 클래스를 로드한다.

***spring.fatories***
```xml
org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.firewood.myspringbootmodule.autoconfig.CustomAutoConfiguration
```

그리고 maven 빌드하여 jar 파일로 만들자.

## 커스텀 스프링 부트 사용하기
커스텀 스프링 부트 모듈을 사용하려면 jar 파일을 pom에 의존성을 추가하여야 한다.

[Adding dependencies to local .jar files in pom.xml](https://gist.github.com/timmolderez/92bea7cc90201cd3273a07cf21d119eb)

```java
@SpringBootApplication
public class MyCustomSpringbootApplication implements CommandLineRunner {
    
    @Autowired
    PersonService personService;

    public static void main(String[] args) {
        SpringApplication.run(MyCustomSpringbootApplication.class, args);
    }

    @Override
    public void run(String... args) throws Exception {
        Person person = personService.getPerson();
        System.out.println(person.getName());
        System.out.println(person.getAddress());
    }
}
```

실행하면 다음과 같이 jar 파일에 작성한 Person의 정보가 출력된다.

```code
alson
from custom module
```

***
[샘플 코드: firewood3's GitHub](https://github.com/firewood3/spring/tree/master/springboot-custom)  

참고자료  
[Createing a Custom Starter with Springboot(Baeldung)](https://www.baeldung.com/spring-boot-custom-starter)  
[spring-boot-starter 작성하기](https://supawer0728.github.io/2018/03/15/create-spring-boot-starter/)  
[Creating a Custom Starter with Spring Boot](https://www.youtube.com/watch?v=mi0GfmTv2wg)  


