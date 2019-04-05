---
title: "@Import 어노테이션과 ImportSelector 인터페이스"
date: 2019-03-31 16:44:00 -0400
categories: java spring annotation
---

ImortSelector 인터에페이스를 사용하면 @Import 어노테이션을 사용할때 불러들일 구성 클래스를 선택할 수 있다.

***Import 어노테이션은 구성 클래스를 로드하는 어노테이션이다. @Import 어노테이션을 사용하면 @EnableWebMvc나 @EnableWebFlux 같은 자신만의 @EnableMyConfiguration 어노테이션을 제작하여 어노테이션 하나를 사용하므로써 필요한 구성클래스를 스프링 컨테이너에 로드할 수 있다.***

## @Import 어노테이션
스프링 프레임워크를 사용하다보면 @EnableWebMvc, @EnableWebFlux, @EnableWebSocket이라는 어노테이션을 찾아볼 수 있다. 이러한 'Enable' 이라는 접두어가 붙는 어노테이션들은 내부적으로 @Import라는 어노테이션을 사용하여 속성으로 특정 구성클래스나 ImportSelector를 지정할 수 있으며 런타임 시 스프링 컨텍스트가 관리할 수 있도록 하고 있는 것이다.

```java
/**
 * Indicates one or more {@link Configuration @Configuration} classes to import.
 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Import {
    /**
	 * {@link Configuration}, {@link ImportSelector}, {@link ImportBeanDefinitionRegistrar}
	 * or regular component classes to import.
	 */
	Class<?>[] value();
}
```

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Import(DelegatingWebFluxConfiguration.class)
public @interface EnableWebFlux {}
```

```java
@Configuration
public class DelegatingWebFluxConfiguration extends WebFluxConfigurationSupport {}
```

## ImportSelector 인터페이스
ImportSelector 인터페이스는 선택적으로 구성클래스를 로드 하고자 할때 사용하는 인터페이스이다. 가령 하나의 POJO 클래스를 빈으로 등록하려 할때 경우에 따라 POJO의 값을 다르게 하고 싶을 때가 있다. 이럴때는 빈을 생성하는 구성클래스를 두개 만들고 ImportSelecor를 사용하여 상황에 맞는 구성클래스를 선택해주면 된다.

ImportSelect 인터페이스의 selectImports() 메소드는 어노테이션 메타데이터로부터 값을 입력받아 구성클래스명을 리턴해준다.

```java
public interface ImportSelector {

	/**
	 * Select and return the names of which class(es) should be imported based on
	 * the {@link AnnotationMetadata} of the importing @{@link Configuration} class.
	 */
	String[] selectImports(AnnotationMetadata importingClassMetadata);

}
```

## 구성클래스를 선택하는 어노테이션 만들기

지금부터 속성값을 입력받아 구성클래스를 선택하는 어노테이션을 만들어 볼것이다. 먼저 빈으로 등록할 POJO 클래스를 만들자. Person 빈은 구성 클래스에 따라 jobName이 student나 person으로 바뀔 것이다.

```java
@AllArgsConstructor
public class Person {
    @Getter
    private String jobName;
}
```

그 다음으로 두개의 구성 클래스를 만들자. 이 두개의 구성클래스는 어노테이션의 입력값으로 어떤 구성클래스를 로드할지 선택되게 된다.

```java
@Configuration
public class PersonToStudentConfig {
    @Bean
    public Person person() {
        return new Person("Student");
    }
}
```

```java
@Configuration
public class PersonToTeacherConfig {
    @Bean
    public Person person() {
        return new Person("Teacher");
    }
}
```

그리고 구성클래스를 선택할 속성값을 받는 어노테이션을 만들자. 이 어노테이션은 @Import어노테이션을 사용하여 ImportSelect를 불러오고 있다. jobName 속성값은 ImportSelect에서 어떤 구성 클래스를 로드할지 판별할 때 사용하게 된다.

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Import(PersonImportSelector.class)
public @interface EnablePerson {
    String jobName() default "student";
}
```

ImportSelector 인터페이스의 구현체에서는 어노테이션의 속성값을 통해 선택적으로 구성 클래스명을 리턴하는 로직을 구현하고 있다.

```java
public class PersonImportSelector implements ImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        AnnotationAttributes attributes = AnnotationAttributes.fromMap(importingClassMetadata.getAnnotationAttributes(EnablePerson.class.getName(), false));
        assert attributes != null;
        String jobName = attributes.getString("jobName");

        if(jobName.equals("teacher")) {
            return new String[]{PersonToTeacherConfig.class.getName()};
        } else {
            return new String[]{PersonToStudentConfig.class.getName()};
        }
    }
}
```

## 어노테이션을 사용해 구성클래스 선택하기
아래의 구성클래스는 @EanblePerson이라는 어노테이션을 통해 Person이라는 POJO 클래스를 빈으로 등록하고 있으며 이때 호출되는 구성 클래스는 PersonToTeacherConfig가 되도록 속성값을 teacher로 넣어주고 있다.

```java
@Configuration
@EnablePerson(jobName = "teacher")
public class ChoosePersonConfig {}
```

ChoosePersonConfig 구성 클래스를 스프링 컨텍스트에 담아보자. @EablePerson -> PersonImportSelector -> PersonToTeacherConfig -> person 빈 생성 순서로 진행되며 Person의 jobName 프로퍼티는 "teacher"가 될 것이다.

```java
public static void main(String args[]){
    ApplicationContext context = new AnnotationConfigApplicationContext(ChoosePersonConfig.class);
    Person person = context.getBean(Person.class);
    System.out.println("JobName : " +  person.getJobName());
}
```

```code
JobName : Teacher
```
***

[firewood3's GitHub : Using ImportSelector interface to select configuration class](https://github.com/firewood3/spring/tree/master/springboot-annotations/springboot-importselector)

참고도서  
제목: 스프링 부트로 배우는 자바 웹 개발  
지은이: 윤석진  
펴낸곳: 제이펍  
