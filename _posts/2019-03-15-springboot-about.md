---
title: "스프링 부트에 대해"
date: 2019-03-15 11:00:00 -0400
categories: java spring
---

## 스프링 부트
스프링이 처음으로 J2EE에서 나왔을 때는 의존성을 주입해주는 컨터이너를 관리하는 라이브러리였고 웹만을 위한 프레임워크가 아니었다. 스프링으로 웹개발을 하기위해서는 각종 설정 작업을 해주어야 하기때문에 웹 개발을 위해 나온 Ruby on Rails나 Node의 Express보다 초기 설정 작업이 복잡했다. 그리고 Ruby on Rails나 Node의 Express는 웹 서버를 내장하는 방식이기 때문에 클라우드 환경에서도 강력한 힘을 발휘했다. 스프링 진영에서는 이러한 환경에 대응하기 위해 설정을 간소화 하고 웹 서버를 내장한 스프링 부트를 만들었다.

- 스프링 보다 설정이 간소회 됨
- 웹 서비스를 보다 쉽게 만들 수 있음
- 웹 서버를 내장하고 있음
- Runnable JAR로 실행됨

***스프링 부트의 폴더 규약***

| 웹 자원 | 경로 |
| ----------- | ---------------- |
| 정적 파일 | src/main/resources/static |
| 파비콘 | src/main/resources/favicon.ico | 
| 템플릿 | src/main/resources/templates <br> html - Thymeleaf<br>tpl - Groovy<br>ftl - Freemarker<br>vm-velocity|

## 타임리프를 사용하기
- 템플릿 엔진은 서식(HTML, XML)과 데이터를 결합해주는 도구
- JSP는 스크립트릿(<%%>) 태그와 html 태그들이 혼합되어 있어 유지보수가 힘들다.
- 스프링 부트에서는 JSP를 권장하지 않는다.
- 타임리프는 HTML5와 호환 가능하여 JSP를 대체해서 사용할 수 있다.
- 타임리프 태그와 SpEL ) 
    - ```<p th:text="${message}"></p>```
    - 타임리프 태그 : th:text, th:object, th:if, th:each
    - SpEL : "${message}"

***데이터 출력***
```java
@GetMapping("/message")
public String simpleMessage(Model model) {
    model.addAttribute("message", "sprintboot!");
    return "message";
}
```
```html
<body>
<p th:text="${message}"></p>
</body>
```

***객체의 프로퍼티 출력***
```java
@GetMapping("/person")
public String object(Model model) {
    Person person = new Person("KIM", 35, LocalDate.of(1985,03,22));
    model.addAttribute("person", person);
    return "person";
}
```
```html
<div th:object="${person}">
    <p>name: <span th:text="*{name}"></span></p>
    <p>age: <span th:text="*{age}"></span></p>
    <p>birth: <span th:text="*{birth}"></span></p>
</div>
```

***조건문 출력***
```html
<p th:if="${person.age} >= 20 and ${person.age} < 30">20대 입니다.</p>
<p th:if="${person.age} >= 30 and ${person.age} < 40">30대 입니다.</p>
```

***반복문 표현***
```java
@GetMapping("/people")
public String each(Model model) {
    ArrayList<Person> people = new ArrayList<>();
    people.add(new Person("KIM", 35, LocalDate.of(1985,3,22)));
    people.add(new Person("LEE", 30, LocalDate.of(1990,2,2)));
    people.add(new Person("PARK", 40, LocalDate.of(1980,10,2)));
    model.addAttribute("people",people);
    return "people";
}
```

```html
<table border="1">
    <tr>
        <th>name</th>
        <th>age</th>
        <th>birth</th>
    </tr>
    <tr th:each="person : ${people}">
        <td th:text="${person.name}"></td>
        <td th:text="${person.age}"></td>
        <td th:text="${person.birth}"></td>
    </tr>
</table>
```

***


[SimpleCode: GitHub](https://github.com/firewood3/boot-jpub/tree/master/sprinboot-simple-thymeleaf)


참고도서  
제목: 스프링 부트로 배우는 자바 웹 개발  
지은이: 윤석진  
펴낸곳: 제이펍  
