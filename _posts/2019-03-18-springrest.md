---
title: "스프링 REST"
date: 2019-03-18 10:33:00 -0400
categories: java spring
---

## REST 서비스로 XML 발행하기
- MarshallingView를 이용하여 xml을 View로 발행하기
    - Marshalling은 객체를 특정한 데이터 형식으로 변환하는 과정
- @ResponseBody를 이용하여 xml을 http 응답 데이터로 발행하기
    - @ResponseBody는 스프링 MVC의 HttpMessageConverter를 이용하여 객체를 특정한 데이터 형식으로 변환

***MarshallingView를 이용하여 XML 발행하기***
```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@XmlRootElement
public class Student {
    private int id;
    private String name;
}
```

```java
@GetMapping(value = "/student")
public String getStudent(Model model) {
    model.addAttribute("xmlStudent", new Student(1, "LEE"));
    return "student";
}
```

```java
@Configuration
public class MarshallingViewConfiguration {

    @Bean
    public View student() {
        return new MarshallingView(jaxb2marshaller());
    }

    @Bean
    public Marshaller jaxb2marshaller() {
        Jaxb2Marshaller marshaller = new Jaxb2Marshaller();
        marshaller.setClassesToBeBound(Student.class);
        marshaller.setMarshallerProperties(Collections.singletonMap(javax.xml.bind.Marshaller.JAXB_FORMATTED_OUTPUT, Boolean.TRUE));
        return marshaller;
    }
}
```

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<student>
    <id>1</id>
    <name>LEE</name>
</student>
```

***ResponseBody를 이용하여 XML발행하기***

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@XmlRootElement
public class Student {
    private int id;
    private String name;
}
```

```java
@GetMapping(value = "/httpConverter/student")
@ResponseBody
public Student getHttpConverterStudent() {
    return new Student(10, "PARK");
}
```

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<student>
    <id>10</id>
    <name>PARK</name>
</student>
```

## REST 서비스로 JSON 발행하기
- Mappingjacson2JsonView로 json을 View로 발행하기
- @ResponseBody를 이용하여 json을 http 응답 데이터로 발행 하기

***뷰를 생성하여 Json받기***
```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Animal {
    private String name;
}
```

```java
@Configuration
public class JsonViewConfiguration {
    @Bean
    public View viewAnimal() {
        MappingJackson2JsonView jsonView = new MappingJackson2JsonView();
        jsonView.setPrettyPrint(true);
        System.out.println("view");
        return jsonView;
    }
}
```

```java
@GetMapping("/view/animal")
public String getXmlAnimal(Model model) {
    System.out.println("dd");
    model.addAttribute("animal", new Animal("cat"));
    return "viewAnimal";
}
```

```json
{
    "animal": {
        "name": "cat"
    }
}
```


***ResponseBody를 이용하여 JSON받기***

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Animal {
    private String name;
}
```

```java
@GetMapping("/body/animal")
@ResponseBody
public Animal getBodyAnimal() {
    return new Animal("dog");
}
```

```json
{
    "animal": {
        "name": "dog"
    }
}
```

## RestTemplate으로 Rest 서비스 접근하기
- RestTemplate : Rest 서버와 연동하기 위한 클래스

| 메서드 | 설명 |
| ----------- | ---------------- |
| headForHeaders(String, Object) | HTTP HEAD 작업을 합니다. |
| getForObject(String, Class, Object) | HTTP GET 작업을 한 다음, 주어진 클래스 타입으로 결과를 반환합니다. | 
| getForEntity(String, Class, Object) | HTTP GET 작업을 한 다음, ResponseEntity를 반환합니다. | 
| postForLocation(String, Object, Object) | HTTP POST 작업을 한 다음, location 헤더값을 반환합니다. |
| postForObject(String, Object, Class, Object) | HTTP POST 작업을 한 다음, 주어진 클래스 타입으로 결과를 반환합니다. |
| postForEntity(String, Object, Class, Object) | HTTP POST 작업을 한 다음, ResponseEntity를 반환합니다. |
| put(String, Object, Object) | HTTP PUT 작업을 합니다. |
| delete(String, Object) | HTTP DELETE 작업을 합니다. |
| optionsForAllow(String, Object) | HTTP 작업을 합니다. |
| execute(String, HttpMethod, RequestCallback, ResponseExtractor, Object) | CONNECT를 제외한 모든 HTTP 작업이 가능한 메서드입니다.  |

***RestTemplete 사용***

```java
public class NoticeRestTest {

    @Test
    public void getAllTest() {
        RestTemplate restTemplate = new RestTemplate();
        ArrayList result = restTemplate.getForObject("http://localhost:8080/notice/get/all", ArrayList.class);
        System.out.println(result);
    }
    
    @Test
    public void getOneTest() {
        RestTemplate restTemplate = new RestTemplate();
        Map<String, String> params = new HashMap<>();
        params.put("id", "3");
        Notice notice = restTemplate.getForObject("http://localhost:8080/notice/get/one/{id}", Notice.class, params);
        System.out.println(notice);
    }


    @Test
    public void createTest() {
        RestTemplate restTemplate = new RestTemplate();
        Notice postNotice = new Notice("새롭게 추가", "내용입니다.");
        Notice notice = restTemplate.postForObject("http://localhost:8080/notice/create", postNotice, Notice.class);
        System.out.println(notice);
    }

    @Test
    public void updateTest() {
        RestTemplate restTemplate = new RestTemplate();
        Notice postNotice = new Notice("제목 변경", "내용 변경");
        Map<String, String> params = new HashMap<>();
        params.put("id", "4");
        restTemplate.put("http://localhost:8080/notice/update/{id}", postNotice, params);
    }

    @Test
    public void deleteTest() {
        RestTemplate restTemplate = new RestTemplate();
        Map<String, String> params = new HashMap<>();
        params.put("id", "1");
        restTemplate.delete("http://localhost:8080/notice/delete/{id}", params);
    }
}
```




***
참고도서  
제목: 스프링5레시피(4판)  
지은이: 마틴데니엄, 다니엘 루비오, 조시 롱  
옮긴이: 이일웅  
펴낸곳: 한빛미디어  
