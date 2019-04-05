---
title: "자바 어노테이션 만들기"
date: 2019-03-31 11:07:00 -0400
categories: java annotation
---

어노테이션은 1.5 버전부터 지원되는 기능으로 일종의 메타데이터(metadata)다. 어노테이션의 사전적인 의미는 '주석'인데 주석처럼 코드에 추가해서 사용할수 있고, 컴파일 또는 런타임 시에 해석된다.
==> 어노테이션을 사용해 코딩을 해보니 어노테이션으로 지정한 메타데이터(어트리뷰트)를 메소드나 클래스에서 파라메타 처럼 불러와서 사용할 수 있었다.

## 어노테이션 선언하기
어노테이션은 자바의 @interface 예약어를 사용하여 선언 할 수 있다.

```java
@Target(ElementType.METHOD) // 어노티에션이 적용될 요소타입
@Retention(RetentionPolicy.RUNTIME) // 어노테이션 적용 범위
public @interface MyAnnotation {}
```
- @interface: 어노테이션을 선언하기 위해 사용되는 키워드
- @Target: 어노테이션이 적용되는 대상을 지정하는 어노테이션
- @Retention: 어노테이션이 적용될 범위를 결정하는 어노테이션

**@Target 어노테이션으로 만들 어노테이션이 적용되는 대상을 지정**
```java
public enum ElementType {
    /** Class, interface (including annotation type), or enum declaration */
    TYPE,

    /** Field declaration (includes enum constants) */
    FIELD,

    /** Method declaration */
    METHOD,

    /** Formal parameter declaration */
    PARAMETER,

    /** Constructor declaration */
    CONSTRUCTOR,

    /** Local variable declaration */
    LOCAL_VARIABLE,

    /** Annotation type declaration */
    ANNOTATION_TYPE,

    /** Package declaration */
    PACKAGE,

    /**
     * Type parameter declaration
     *
     * @since 1.8
     */
    TYPE_PARAMETER,

    /**
     * Use of a type
     *
     * @since 1.8
     */
    TYPE_USE
}
```


**@Retention 어노테이션으로 만들 어노테이션이 적용될 범위를 결정**
```java
public enum RetentionPolicy {
    /**
     * Annotations are to be discarded by the compiler.
     */
    SOURCE,

    /**
     * Annotations are to be recorded in the class file by the compiler
     * but need not be retained by the VM at run time.  This is the default
     * behavior.
     */
    CLASS,

    /**
     * Annotations are to be recorded in the class file by the compiler and
     * retained by the VM at run time, so they may be read reflectively.
     *
     * @see java.lang.reflect.AnnotatedElement
     */
    RUNTIME
}
```

## 어노테이션 만들기
MyAnnotation은 문자열 값과 정수형 값을 입력할 수 있는 어노테이션이다.  
@interface로 선언한 객체의 메소드는 파라메터를 가질 수 없다.  
메소드 뒤에 default 키워드를 붙혀 반환될 기본값을 입력할 수 있다.

```java
@Target(ElementType.METHOD) // 어노티에션이 적용될 요소타입
@Retention(RetentionPolicy.RUNTIME) // 어노테이션 적용 범위
public @interface MyAnnotation {
    // 어노테이션은 interface에 @를 붙여서 선언하고 어노테이션이 적용될 대상과 동작 방식을 지정할 수 있다.
    String strValue() default "Default String";
    int intValue() default 0;
}
```

어노테이션을 사용할 때는 @키워드를 어노테이션 이름 앞에 붙히고 어노테이션에 선언된 메소드의 반환값을 입력한다. 입력하지 않으면 디폴트 반환값이 반환된다.

```java
public class MyService {
    @MyAnnotation(strValue = "hi", intValue = 607)
    public void settingValue() {
    }
    @MyAnnotation
    public void defaultValue() {
    }
}
```

## 어노테이션의 메소드 사용하기
어노테이션의 메소드를 사용해 보기 위해 자바 리플렉션 API를 사용한다.

***리플렉션이란 객체를 이용해 클래스의 정보를 분석해 내는 프로그램 기법을 말하고 자바에서는 이 리플렉션을 사용할 수 있는 API를 제공한다.***

```java
Method[] methods = Class.forName(MyService.class.getName()).getMethods();
for (Method method : methods) {
    if (method.isAnnotationPresent(MyAnnotation.class)) {
        MyAnnotation myAnnotation = method.getAnnotation(MyAnnotation.class);
        System.out.println(myAnnotation.strValue());
        System.out.println(myAnnotation.intValue());
    }
}
```

실행결과는 다음과 같다.
```code
hi
607
no String
0
```

***
예제 코드  
[firewood3's GitHub](https://github.com/firewood3/spring/tree/master/springboot-annotations/java-create-annotation)

참고 문서  
[자바 리플렉션 개념 및 사용법](https://gyrfalcon.tistory.com/entry/Java-Reflection)

[Java 커스텀 Annotation 만들기](https://medium.com/@ggikko/java-%EC%BB%A4%EC%8A%A4%ED%85%80-annotation-436253f395ad)

참고도서  
제목: 스프링 부트로 배우는 자바 웹 개발  
지은이: 윤석진  
펴낸곳: 제이펍  
