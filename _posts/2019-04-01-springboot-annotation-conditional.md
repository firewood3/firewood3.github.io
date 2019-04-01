---
title: "Spring @Conditional annotation"
date: 2019-04-01 10:15:00 -0400
categories: java spring annotation
---

## @Conditional 어노테이션
Conditional 어노테이션은 조건에 따라 자바 설정 클래스를 선택할 수 있게 해주는 어노테이션이다. 이를 사용하면 환경변수 설정 값으로 빈으로 등록할 POJO 클래스를 선택할 수 있다.

Student와 Teacher라는 두 개의 POJO 클래스가 있고, 이 클래스들은 Person이라는 인터페이스를 타입로 갖는다. 그리고 우리는 Person이라는 타입으로 빈을 불러 올 수 있다. 이때, Person이라는 빈이 Student가 될지 Teacher가 될지 환경변수를 지정하여 선택해보자.

먼저 빈의 타입이 되는 Person 인터페이스와 이를 빈으로 등록할 Teacher와 Student POJO를 정의한다.

```java
public interface Person {
    String getJob();
}
```

```java
@Component
public class Teacher implements Person{
    @Override
    public String getJob() {
        return "My job is teacher";
    }
}
```

```java
@Component
public class Student implements Person{
    @Override
    public String getJob() {
        return "My job is student";
    }
}
```

그 다음에는 Student와 Teacher POJO에 @Conditional 어노테이션을 사용하여 어떤 Person 타입이 빈으로 등록될지 선택되게 만들자. @Conditonal 어노테이션은 타입으로 Condition 인터페이 구현체를 받으며, 이 Condition 인터페이스는 matchers라는 메소드의 반환값을 통해 해당 클래스가 빈으로 등록될지 말지 여부를 결정한다.

```java
@Component
@Conditional(StudentConditional.class)
public class Student implements Person{
    @Override
    public String getJob() {
        return "My job is student";
    }
}
```

```java
public class StudentConditional implements Condition {
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        // True이면 빈으로 등록
        // 자바 클래스파일이나 Jar 파일을 실행할때, -Dperson=student 를 입력하면 Student 빈이 등록됨!
        return "student".equals(context.getEnvironment().getProperty("person"));
    }
}
```

```java
@Component
@Conditional(TeacherConditional.class)
public class Teacher implements Person{
    @Override
    public String getJob() {
        return "My job is teacher";
    }
}
```

```java
public class TeacherConditional implements Condition {
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        // True이면 빈으로 등록
        // 자바 클래스파일이나 Jar 파일을 실행할때, -Dperson=teacher 를 입력하면 Person 빈이 등록됨!
        return "teacher".equals(context.getEnvironment().getProperty("person"));
    }
}
```

마지막으로 테스트를 해보자. @Conditonal이 사용된 코드를 바로 작동시키려고 하면 오류가 발생한다. 꼭 환경변수를 넣어주자. ex) java -Dperson=teacher -jar filename.jar argumenets | java -Dperson=teacher filename.class argumenets

```java
public static void main(String[] args) {
    Package pack = ConditionalTestMain.class.getPackage();
    AnnotationConfigApplicationContext context = newAnnotationConfigApplicationContext();
    context.scan(pack.getName());
    context.refresh();
    Person person = context.getBean(Person.class);
    System.out.println(person.getJob());
}
```

콘솔 출력  

```code
My job is teacher
```

***
[firewood3's GitHub : Spring @Conditional annotataion](https://github.com/firewood3/spring/tree/master/springboot-annotations/conditional)

참고자료  
[java 명령어의 옵션 정리](http://sjava.net/2008/02/java-%EB%AA%85%EB%A0%B9%EC%96%B4%EC%9D%98-%EC%98%B5%EC%85%98-%EC%A0%95%EB%A6%AC/)

참고도서  
제목: 스프링 부트로 배우는 자바 웹 개발  
지은이: 윤석진  
펴낸곳: 제이펍  
