---
title: "스프링 웹 MVC에 대하여 알아보자"
date: 2019-03-12 11:46:00 -0400
categories: java spring
---

스프링 웹 MVC는 Model-View-Controller 구조로 구성되어 있으며 이는 개발자가 유연하고 느슨하게 개발된 웹 어플리케이션을 개발 할 수 있도록 한다.
- Model : 모델은 데이터를 캡슐화 한 것이다.
- View : 뷰는 모델 데이터를 렌더링한 것이며 이는 Http 응답으로 요청자에게 전달된다. (ex. html, jsp, pdf)
- Controller : 컨트롤러는 사용자 요청을 처리하여 모델을 생성하고 뷰에 전달한다.


디스패쳐 서블렛(DispatcherServlet)은 스프링 웹 MVC의 핵심적인 컴포넌트이다. 디스패쳐 서블렛은 HTTP 요청과 응답을 처리하고 모델, 뷰, 컨트롤러를 제어 한다.


![screensh](https://www.tutorialspoint.com/spring/images/spring_dispatcherservlet.png)

1. 디스패쳐 서블릿은 HTTP 요청을 받으면 HandlerMapping을 찾아 적절한 Controller가 호출될 수 있도록 한다.
2. 컨트롤러는 비즈니스 로직(Service method)을 실행하여 해당 HTTP 요청에 대한 적절한 모델 데이터를 생성하고 뷰 이름을 디스패쳐 서블릿에게 리턴한다.
3. 디스패쳐 서블릿은 컨트롤러로부터 받은 뷰 이름을 뷰 리졸버에게 전달하여 뷰를 가져온다.
4. 디스패쳐 서블릿이 뷰를 가져오면 모델과 뷰를 렌더링 하여 HTTP 응답으로 요청자에게 전달한다.


***Controller와 HandlerMapping***
```java
@Controller
public class HelloController {

    @RequestMapping(value = "/", method = RequestMethod.GET)
    public String printHello(ModelMap model) {
        model.addAttribute("message", "Hello Spring WEB MVC!");
        return "index";
    }
}
```


***ViewResolver***
```java
@Configuration
public class WebConfiguration {

    @Bean
    public ViewResolver internalResourceViewResolver() {
        System.out.println("ddd");
        InternalResourceViewResolver viewResolver = new InternalResourceViewResolver();
        viewResolver.setPrefix("/WEB-INF/jsp/");
        viewResolver.setSuffix(".jsp");
        return viewResolver;
    }
}
```

***View(jsp)***
```html
<html>
<head>
    <title>Hello Spring MVC</title>
</head>

<body>
<h2>${message}</h2>
</body>
</html>
```


[samplecode: 스브링부트에서 ViewResolver를 설정하여 JSP 적용](https://github.com/firewood3/spring-recipe/tree/master/springboot-jsp-simple)


***
참고자료
[스프링 튜토리얼 Spring Web MVC](https://www.tutorialspoint.com/spring/spring_web_mvc_framework.htm)


