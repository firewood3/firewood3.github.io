---
title: "스프링 REST"
date: 2019-03-18 12:33:00 -0400
categories: java spring
---

## REST에 대하여

REST(Representational State Transfer)는 월드 와이드 웹과 같은 분산 하이퍼미디어 시스템을 위한 소프트웨어 아키텍처의 한 형식이다.
REST는 네트워크 아키텍처 원리의 모은인데, 여기서 '네트워크 아키텍처 원리'란 자원을 정의하고 자원에 대한 주소를 지정하는 방법 전반을 일컫는다.

*=>HTTP 통신으로 직렬화된 데이터를 특정 규칙에 의해서 주고받는 API*

### REST 아키텍처에 적용되는 6가지 제한 조건
- 클라이언트/서버 구조: 일관적인 인터페이스로 분리되어야 한다.
- 무상태(Stateless): 각 요청 간 클라이언트의 콘텍스트가 서버에 저장되어서는 안 된다.
- 캐시 처리 가능(Cacheable): WWW에서와 같이 클라리언트는 응답을 캐싱할 수 있어야 한다.
- 계층화(Layered System): 클라리언트는 보통 대상 서버에 직접 연결되었는지, 또는 중간 서버를 통해 연결되었는지를 알 수 없다. 중간 서버는 로드 밸런싱 기능이나 공유 캐시 기능을 제공함으로써 시스템 규모 확장성을 향상시키는 데 유용하다.
- Code on demand(optional): 자바 애플릿이나 자바스크립트 같은 서버가 클라이언트를 실행시킬 수 있는 로직을 전송하여 기능을 확장시킬 수 있다.
- 인터페이스 일관성: 아키텍처를 단순화시키고 작은 단위로 분리(decouple)함으로써 클라이언트-서버의 각 파트가 독립적으로 개선될 수 있도록 해준다.

### REST 인터페이스 원칙 가이드
- 자원의 식별: URL의 사용과 같이 개별 자원을 식별할 수 있어야 한다.
- 메시지를 통한 리소스의 조작: 클라이언트가 서버의 자원을 조작하기위해 자원을 지칭하는 특정 데이터를 가지고 있는 것을 말한다.
- 자기서술적 메시지: 각 메시지는 자기 자신을 어떻게 처리해야 하는지에 대한 충분한 정보를 포함해야 한다.(ex, 헤더를 보고 데이터의 구조, HTML이나 JSON, XML을 아는 것)
- 애플리케이션의 상태에 대한 엔진으로서 하이퍼 미디어(HAREOAS, Hypermedia AS the Engin Of Application State) : 해당 메시지에 관련된 리소스의 위치를 알려주는 것을 말한다.


## REST 서비스로 XML 발행하기
- MarshallingView를 이용하여 xml을 View로 발행하기
    - Marshalling은 객체를 특정한 데이터 형식으로 변환하는 과정
- @ResponseBody를 이용하여 xml을 http 응답 데이터로 발행하기
    - @ResponseBody는 내부적으로 HttpMessageConverter를 사용하며, View를 거치지 않고 HTTP Response Body에 직접 입력된다.

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
    "name": "dog"
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


***REST API CONTROLLER***
```java
@RestController
@RequestMapping("/notice")
public class NoticeController {

    private final NoticeService noticeService;

    @Autowired
    public NoticeController(NoticeService noticeService) {
        this.noticeService = noticeService;
    }

    @GetMapping("/get/all")
    public ResponseEntity<List<Notice>> getAll() {
        return new ResponseEntity<>(noticeService.getAllNotice(), HttpStatus.OK);
    }

    @GetMapping(value = "/get/one/{id}", produces = MediaType.APPLICATION_JSON_UTF8_VALUE)
    public ResponseEntity<Notice> getOne(@PathVariable Long id) {
        return noticeService
                .getOneNotice(id)
                .map(notice -> new ResponseEntity<>(notice, HttpStatus.OK))
                .orElseGet(()->new ResponseEntity<>(HttpStatus.NOT_FOUND));
    }

    @PostMapping(value = "/create", produces = MediaType.APPLICATION_JSON_UTF8_VALUE)
    public ResponseEntity<Notice> createMemo(@RequestBody NoticeBuilder noticeBuilder) {
        try {
            return new ResponseEntity<>(noticeService.createNotice(noticeBuilder), HttpStatus.OK);
        } catch (RuntimeException e) {
            return new ResponseEntity<>(HttpStatus.BAD_REQUEST);
        }
    }

    @PutMapping(value = "/update/{id}", produces = MediaType.APPLICATION_JSON_UTF8_VALUE)
    public ResponseEntity<Notice> updateMemo(@PathVariable Long id, @RequestBody NoticeBuilder noticeBuilder) {
        return noticeService
                .updateNotice(id, noticeBuilder)
                .map(notice -> new ResponseEntity<>(notice, HttpStatus.OK))
                .orElseGet(() -> new ResponseEntity<>(HttpStatus.BAD_REQUEST));
    }

    @DeleteMapping(value = "/delete/{id}", produces = MediaType.APPLICATION_JSON_UTF8_VALUE)
    public ResponseEntity<Void> deleteMemo(@PathVariable Long id) {
        try {
            noticeService.deleteNotice(id);
            return new ResponseEntity<>(HttpStatus.OK);
        } catch (EntityNotFoundException e) {
            return new ResponseEntity<>(HttpStatus.NOT_FOUND);
        }
    }
}
```


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

## HATEOAS(Hypermedia as the Engine of Application State)를 이용한 자기주소정보 표현
- URL을 요청했을 때 관련된 URL 정보를 표현해주는 라이브러리
- spring-boot-starter-hateoas

***HATEOAS 적용***
```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Book extends ResourceSupport {
    private String title;
    private String author;
    private String publisher;
}

@RestController
@RequestMapping("/hateoas")
public class BookController {

    @GetMapping("/book")
    public ResponseEntity<Book> getBook() {
        Book book = new Book("스프링", "java", "people");
        book.add(linkTo(methodOn(BookController.class).getBook()).withSelfRel());
        return new ResponseEntity<>(book, HttpStatus.OK);
    }
}
```

```json
{
    "title": "스프링",
    "author": "java",
    "publisher": "people",
    "_links": {
        "self": {
            "href": "http://localhost:8080/hateoas/book"
        }
    }
}
```


## swagger를 이용한 REST API 문서화
- swagger는 API 메서드를 자동으로 문서화 해주는 라이브러리
- springfox-swagger-ui
- springfox-swagger2

***swagger설정***
```java
@Configuration
@EnableSwagger2
public class SwaggerConfig {

    @Bean
    public UiConfiguration uiConfig() {
        return UiConfigurationBuilder.builder().build();
    }

    private ApiInfo metadata() {
        return new ApiInfoBuilder()
                .title("SPRING REST")
                .description("LEARN REST")
                .version("1.0")
                .build();
    }

    @Bean
    public Docket api() {
        return new Docket(DocumentationType.SWAGGER_2)
                .select()
                .apis(RequestHandlerSelectors.any())
                .paths(PathSelectors.any())
                .build()
                .apiInfo(metadata());
    }
}
```

***swagger 적용***
![swagger 적용](../images/swagger_적용.png)

***
[샘플 코드](https://github.com/firewood3/spring/tree/master/spring-rest-01)

참고자료  
[위키백과 REST](https://ko.wikipedia.org/wiki/REST)   
[Srping Hateoas](https://spring.io/understanding/HATEOAS)  


참고도서  
제목: 스프링5레시피(4판)  
지은이: 마틴데니엄, 다니엘 루비오, 조시 롱  
옮긴이: 이일웅  
펴낸곳: 한빛미디어  


참고도서  
제목: 스프링 부트로 배우는 자바 웹 개발  
지은이: 윤석진  
펴낸곳: 제이펍  

