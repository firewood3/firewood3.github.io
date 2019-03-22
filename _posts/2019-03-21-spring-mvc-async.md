---
title: "스프링 MVC 비동기"
date: 2019-03-21 14:35:00 -0400
categories: java spring
---

- [개요](#s)  
- [HTTP 요청 비동기 처리](#p1)  
- [청크 데이터 비동기 처리](#p2)  
- [비동기를 감지하는 인터셉터 사용](#p3)  
- [웹 소켓](#p4)  
- [리액티브 웹](#p5)  
- [리액티브 REST](#p6)  
- [Handler 기반의 리액티브 REST](#p7)  

<a id="s"></a>

## 스프링 MVC 비동기에 대하여
- 최초에는 서블릿 API 컨테이너가 요청당 스레드 하나만 사용(컨테이너가 요청을 받아 처리를 끝내고 클라이언트에 응답을 돌려주기 전까지 스레드는 항상 블로킹 됐었음)
- 서블렛 3부터, HTTP 요청을 비동기로 처리할 수 있었음
- 스프링 5부터는 리엑티브 웹 애플리케이션 개발이 가능
- 리엑티브 프로그래밍은 한마디로 넌블로킹 함수형 프로그래밍을 실천하는 방법

<a id="p1"></a>

## 컨트롤러에서 HTTP 요청을 비동기 처리하기
- HTTP 요청을 비동기 처리하기 위해서 TaskExecutor를 사용한다.
- 서블릿을 등록할때 isAsyncSupport를 true로 해주면 서블릿이 비동기를 지원하지 않는다.
- isAsyncSupport의 기본값은 true이다.
- HTTP 요청을 비동기 처리하는 4가지 방법
    - Callable : 사용할 쓰레드를 선택할 수 없음(미리 설정된 쓰레드를 사용하거나, 스프링에서 생성해주는 쓰레드를 사용해야함)
    - DeferredResult: 사용할 쓰레드를 선택할 수 있음
    - CompletableFuture: 자바의 Future를 구현, 사용할 쓰레드를 선택할 수 있으며, 쓰레드를 선택하지않으면 JVM에서 가용한 기본 Fork/Join 풀을 사용하여 실행됨
    - ListenableFuture: 자바의 Future를 구현, 스프링의 AsyncListenableTaskExecutor를 미리 빈으로 등록 후 해당 쓰레드를 사용해야함


***서블릿의 isAsyncSupport의 기본값***
```java
public abstract class AbstractDispatcherServletInitializer extends AbstractContextLoaderInitializer {
    ...
    protected void registerDispatcherServlet(ServletContext servletContext) {
    ...
        Dynamic registration = servletContext.addServlet(servletName, dispatcherServlet);
        registration.setAsyncSupported(this.isAsyncSupported());
    ...
    }
    ...
}

protected boolean isAsyncSupported() {
    return true;
}
```

***서블릿의 비동기를 끄는 방법***
```java
@Configuration
public class WebInit extends AbstractAnnotationConfigDispatcherServletInitializer {
    @Override
    public void onStartup(ServletContext servletContext) throws ServletException {
        DispatcherServlet dispatcherServlet = new DispatcherServlet();
        ServletRegistration.Dynamic registration = servletContext.addServlet("dispatcher", dispatcherServlet);
        registration.setAsyncSupported(false);
    }
```

***AsyncTaskExecutor를 MVC 설정에 등록하여 비동기 처리하는 방법***
```java
@Configuration
public class AsyncConfiguration implements WebMvcConfigurer {

    @Override
    public void configureAsyncSupport(AsyncSupportConfigurer configurer) {
        configurer.setDefaultTimeout(TimeUnit.MILLISECONDS.convert(5, TimeUnit.SECONDS));
        configurer.setTaskExecutor(mvcTaskExecutor());
    }

    @Bean
    public AsyncTaskExecutor mvcTaskExecutor() {
        ThreadPoolTaskExecutor taskExecutor = new ThreadPoolTaskExecutor();
        taskExecutor.setThreadGroupName("mvc-executor");
        return taskExecutor;
    }
}
```

***HTTTP 통신을 비동기 처리***
```java
@Controller
public class AsyncController {

    private final Logger logger = LoggerFactory.getLogger(AsyncController.class);

    @Autowired
    @Qualifier("myThreadPool")
    private TaskExecutor taskExecutor;

    @Autowired
    @Qualifier("myListenable")
    private AsyncListenableTaskExecutor asyncListenableTaskExecutor;

    @RequestMapping("/sync")
    public String sync() {
        Delayer.threadSleep();
        return "sync";
    }

    @RequestMapping("/async-callable")
    public Callable<String> asyncCallable() {
        return ()->{
            Delayer.threadSleep();
            return "async-callable";
        };
    }

    @RequestMapping("/async-deferredResult")
    public DeferredResult<String> asyncDeferredResult() {
        DeferredResult<String> result = new DeferredResult<>();
        taskExecutor.execute(()->{
            Delayer.threadSleep();
            result.setResult("async-deferredResult");
        });
        return result;
    }

    @RequestMapping("/async-completableFuture")
    public CompletableFuture<String> asyncCompletableFuture() {
        return CompletableFuture.supplyAsync(()->{
            Delayer.threadSleep();
            return "/async-completableFuture";
        }, taskExecutor);
    }

    @RequestMapping("/async-listenableFuture")
    public ListenableFuture<String> asyncListenableFuture() {
        return asyncListenableTaskExecutor.submitListenable(()->{
            Delayer.threadSleep();
            return "async-listenableFuture";
        });
    }
}
```

<a id="p2"></a>

## HttpMessageConverter 인프라를 사용하여 데이터를 청크로 나누어 전송하기
- (@ResponseBody와 비슷)
- ResponseBodyEmitter 사용
- SseEmitter 사용
    - event-stream 기반

***ResponseBodyEmitter와 SseEmitter 사용***

```java
@Controller
public class PersonController {

    @Autowired
    private PersonService personService;

    @GetMapping("/body")
    @ResponseBody
    public List body() {
        return personService.findAllPerson();
    }

    @GetMapping("/emitter")
    public ResponseBodyEmitter emitter() {
        final ResponseBodyEmitter emitter = new ResponseBodyEmitter();
        this.executeEmitter(new ResponseBodyEmitter());
        return emitter;
    }

    @GetMapping("/sseEmitter")
    public SseEmitter sseEmitter() {
        final SseEmitter sseEmitter = new SseEmitter();
        this.executeEmitter(sseEmitter);
        return sseEmitter;
    }

    private void executeEmitter(ResponseBodyEmitter emitter) {
        ExecutorService service = Executors.newSingleThreadExecutor();
        service.execute(()->{
            ArrayList<Person> allPerson = personService.findAllPerson();
            try {
                for(Person person : allPerson) {
                    Thread.sleep(1000);
                    emitter.send(person);
                }
                emitter.complete();
            } catch (IOException e) {
                e.printStackTrace();
                emitter.completeWithError(e);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
    }
}
```

<a id="p3"></a>

## AsyncHandlerInterceptor 인터셉터 활용하기
- HandlerInterceptor 대신 AsyncHandlerInterceptor를 사용하면 HandingMapping단계에서 비동기 실행의 시작을 가로챌수 있음
- preHandle : HandlerMapping 이전, HandelrInterceptor에도 존재
- postHandle : HandlerMapping 이후, 모델 조작 가능, HandelrInterceptor에도 존재
- afterCompletion : 뷰 랜더링 마친 이후, HandelrInterceptor에도 존재
- afterConcurrentHandlingStarted : 비동기 함수 호출 직전에 호출됨, (postHandle이나 afterCompletion 대신 호출된다고함..)

```java
public class InterceptConfig implements AsyncHandlerInterceptor {
    @Override
    public void afterConcurrentHandlingStarted(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("afterConcurrentHandlingStarted" + request.getRequestURI());
    }
}
```

<a id="p4"></a>

## 웹 소켓을 사용하여 양방향 통신하기
- HTML5에서 웹 소켓을 지원하여 사용 가능
- TCP(Transmission Control Protocol) 프로토콜로 양방향 통신하기
    - @EnableWebSocket
    - SocketHandler를 만들어 수신과 송신 처리
- STOMP(Simple Text-Oriented Protocol) 프로토콜로 양방향 통신하기
    - @EnableWebSocketMessageBroker
    - @Controller에서 수신과 송친 처리
    - 전송과 수신 URL의 접두어를 붙힐 수 있는 브로커를 사용할 수 있음

***TCP 프로토콜로 양방향 통신***

*server side*
```java
@Configuration
@EnableWebSocket
public class WebSocketConfiguration implements WebSocketConfigurer {

    @Bean
    public EchoHandler echoHandler() {
        return new EchoHandler();
    }

    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry webSocketHandlerRegistry) {
        webSocketHandlerRegistry.addHandler(echoHandler(), "/echo");
    }
}

public class EchoHandler extends TextWebSocketHandler {
    @Override
    public void afterConnectionEstablished(WebSocketSession session) throws Exception {
        session.sendMessage(new TextMessage("CONNECTION ESTABLISHED"));
    }

    @Override
    public void afterConnectionClosed(WebSocketSession session, CloseStatus status) throws Exception {
        session.sendMessage(new TextMessage("CONNECTION CLOSED"));
    }

    @Override
    protected void handleTextMessage(WebSocketSession session, TextMessage message) throws Exception {
        String msg = message.getPayload();
        session.sendMessage(new TextMessage("RECEIVED: " + msg));
    }
}
```

*client side*
```js
var url = "ws://localhost:8080/echo";
ws = new WebSocket(url);

ws.send(message);

ws.onmessage = function (event) {
    log(event.data);
};

ws.close();
```

***STOMP 프로토콜로 양방향 통신***

*server side*
```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfiguration extends AbstractWebSocketMessageBrokerConfigurer {
    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        registry.enableSimpleBroker("/topic");//브로커가 전송할때 프리픽스
        registry.setApplicationDestinationPrefixes("/app");//브로커가 받을 때 프리픽스
    }

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/echo-endpoint");
    }
}

@Controller
public class EchoHandler {
    @MessageMapping("/echo")
    @SendTo("/topic/echo")
    public String echo(String msg) {
        return "RECEIVED: " + msg;
    }
}
```

*client side*
```js
var url = "ws://localhost:8080/echo-endpoint";
ws = webstomp.client(url);
ws.connect({}, function(frame) {
    ws.subscribe('/topic/echo', function(message){
        log(message.body);
    })
});

ws.send("/app/echo", message);

ws.disconnect();
```

<a id="p5"></a>

## 스프링 웹 플럭스로 리액티브 애플리케이션 개발하기
- 리액티브 프로그래밍은 데이터 흐름과 변화 전파에 중점을 둔 프로그래밍 패러다임
- 리액티브 프로그래밍은 넌블로킹 함수형 프로그래밍이 사용됨
- 리액티브 프로그래밍은 많은 요청을 동시에 처리하고 대기 시간이 긴 작업을 효율적으로 처리할 수 있는 장점이 있음
- Flux와 Momo는 데이터의 흐름을 나타내는 객체

스프링 웹 플럭스는 사용자의 요청을 HttpHandler 인터페이스로 처리하게된다.

```java
public interface HttpHandler {
	Mono<Void> handle(ServerHttpRequest request, ServerHttpResponse response);
}
```

HttpHandler 인터페이스를 사용하기 위해서는 컨테이너(WAS)를 스프링 웹플럭스에 등록하여야한다. 스프링 웹 플럭스는 다양한 컨테이너를 등록시키기 위한 HandlerAdapter 인터페이스를 제공한다.

| 런타임 | 어댑터 |
| ----------- | ---------------- |
| 서블릿 3.1 컨테이너 | ServletHttpHandlerAdapter |
| 톰캣 | ServletHttpHadlerAdapter 또는 TomcatHttpHandlerAdapter | 
| 제티 | ServletHttpHandlerAdapter 도는 JettyHttpHandlerAdapter |
| 리액터 네티 | ReactorHttpHandlerAdapter |
| Rx 네티 | RxNettyHttpHadlerAdatpter |
| 언더토우 | UndertowHttpHandlerAdapter |

***WEB Flux 설정***
```java
public static void main(String[] args) {
    AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContex(WebFluxConfiguration.class);
    HttpHandler handler = WebHttpHandlerBuilder.applicationContext(context).build();
    ReactorHttpHandlerAdapter adapter = new ReactorHttpHandlerAdapter(handler);
    HttpServer.create("localhost", 8090).newHandler(adapter).block();
}

@Configuration
@EnableWebFlux
@ComponentScan
public class WebFluxConfiguration implements WebFluxConfigurer {

    @Bean
    public SpringResourceTemplateResolver thymeleafTemplateResolver() {

        final SpringResourceTemplateResolver resolver = new SpringResourceTemplateResolver();
        resolver.setPrefix("classpath:/templates/");
        resolver.setSuffix(".html");
        resolver.setTemplateMode(TemplateMode.HTML);
        return resolver;
    }

    @Bean
    public ISpringWebFluxTemplateEngine thymeleafTemplateEngine(){

        final SpringWebFluxTemplateEngine templateEngine = new SpringWebFluxTemplateEngine();
        templateEngine.addDialect(new Java8TimeDialect());
        templateEngine.setTemplateResolver(thymeleafTemplateResolver());
        return templateEngine;
    }


    @Bean
    public ThymeleafReactiveViewResolver thymeleafReactiveViewResolver() {

        final ThymeleafReactiveViewResolver viewResolver = new ThymeleafReactiveViewResolver();
        viewResolver.setTemplateEngine(thymeleafTemplateEngine());
        viewResolver.setResponseMaxChunkSizeBytes(16384);
        return viewResolver;
    }

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        registry.viewResolver(thymeleafReactiveViewResolver());
    }
}
```

***간단한 검색 기능의 웹 플럭스 프로그램***
```java
@Controller
public class StudentController {

    private final StudentService studentService;

    @Autowired
    public StudentController(StudentService studentService) {
        this.studentService = studentService;
    }

    @GetMapping("/")
    public String getStudent(Model model) {
        Flux<Student> students = studentService.findAll();
        IReactiveDataDriverContextVariable react = new ReactiveDataDriverContextVariable(students, 1);
        model.addAttribute("students", react);
        return "student";
    }

    @PostMapping("/")
    public String addStudent(ServerWebExchange exchange, Model model) {
        Flux<Student> searched = exchange
                                .getFormData()
                                .map(form-> form.get("name"))
                                .flatMapMany(Flux::fromIterable)
                                .concatMap(studentService::search);

        IReactiveDataDriverContextVariable variable = new ReactiveDataDriverContextVariable(searched, 1);

        model.addAttribute("students", variable);
        return "student";
    }
}

@Service
public class StudentService {

    private ArrayList<Student> students = new ArrayList<>();

    public StudentService() {
        students.add(new Student("kim rara", 20));
        students.add(new Student("kim sin young", 52));
        students.add(new Student("lee sun sin", 21));
        students.add(new Student("lee san", 21));
        students.add(new Student("park chan ho", 22));
        students.add(new Student("hong gil dong", 31));
        students.add(new Student("ki soun young", 22));
        students.add(new Student("son hug min", 22));
        students.add(new Student("kang ho dong", 22));
    }

    public Flux<Student> findAll() {
        return Flux.fromIterable(students).delayElements(Duration.ofSeconds(2));
    }

    public Flux<Student> search(String name) {
        return Flux
                .fromStream(students.stream()
                .filter(student -> student.getName().startsWith(name))).delayElements(Duration.ofSeconds(2));
    }
}
```

<a id="p6"></a>

## 리액티브 REST 서비스를 만들고 비동기 웹 클라이언트로 REST 정보 가져오기
@ResponseBody 또는 @RestController를 사용하고 반환형이 Mono나 Flux인 핸들러 매퍼를 사용하면 리액티브 REST 서비스를 사용할 수 있다.  
스프링에서는 REST API를 비동기적으로 소비하기위해 WebClient라는 인터페이스를 제공한다. 이전에 사용되던 AsyncRestTemplate은 권장되지 않는다.  

***리액티브 Rest 발행과 웹 클라이언트로 REST data 얻기***

```java
@GetMapping("/rest")
@ResponseBody
public Flux<Student> getRest() {
    return studentService.findAll();
}
```

```java
@Test
public void restTest() throws IOException {
    final String url = "http://localhost:8080/";
    WebClient.create(url)
            .get()
            .uri("/rest")
            .accept(MediaType.APPLICATION_STREAM_JSON)
            .exchange()
            .flatMapMany(clientResponse -> clientResponse.bodyToFlux(String.class))
            .subscribe(System.out::println);
    System.in.read();
}
```

<a id="p7"></a>

## 핸들러 기반의 REST API 만들기
스프링에서 리액티브 REST를 만들때, 컨트롤러 기반이 아닌 핸들러 기반으로도 REST 서비스를 만들 수 있다.  
만드는 방법은 ServerRequest를 받아 Mono<ServerResponse>를 반환하는 메서드를 작성한 다음, 이 메서드를 라우터 함수로 매핑한다.

***핸들러 함수 작성하기***
```java
public Mono<ServerResponse> getAllNotice(ServerRequest request) {
    return ServerResponse.ok()
            .contentType(MediaType.APPLICATION_PROBLEM_JSON_UTF8)
            .body(BodyInserters.fromObject(noticeRepository.findAll()));
}
public Mono<ServerResponse> getOneNotice(ServerRequest request) {
    return ServerResponse.ok()
            .contentType(MediaType.APPLICATION_PROBLEM_JSON_UTF8)
            .body(BodyInserters.fromObject(noticeRepository.findById(Long.parseLon(request.pathVariable("id")))));
}
```


***요청을 핸들러 함수로 보내기***
```java
@Bean
RouterFunction<ServerResponse> getAll(NoticeHandler noticeHandler) {
    return RouterFunctions.route(RequestPredicates.GET("/notice/get/all").an(RequestPredicates.accept(MediaType.APPLICATION_JSON_UTF8))
            , noticeHandler::getAllNotice);
}
@Bean
RouterFunction<ServerResponse> getOne(NoticeHandler noticeHandler) {
    return RouterFunctions.route(RequestPredicates.GET("/notice/get/one/{id}").an(RequestPredicates.accept(MediaType.APPLICATION_JSON_UTF8))
            , noticeHandler::getOneNotice);
}
```


***  
[GitHub: Srping MVC Async](https://github.com/firewood3/spring/tree/master/spring-mvc-async)


참고자료  
[WebApplicationInitializer로 Spring MVC 설정하기](http://blog.naver.com/PostView.nhn?blogId=take0415&logNo=221017059396&redirect=Dlog&widgetTypeCall=true)  
[Understanding Reactive types](https://spring.io/blog/2016/04/19/understanding-reactive-types)  
[리엑티브 선언문](https://www.reactivemanifesto.org/ko)  



참고도서  
제목: 스프링5레시피(4판)  
지은이: 마틴데니엄, 다니엘 루비오, 조시 롱  
옮긴이: 이일웅  
펴낸곳: 한빛미디어  
