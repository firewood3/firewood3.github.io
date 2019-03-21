---
title: "스프링 MVC 비동기"
date: 2019-03-21 14:35:00 -0400
categories: java spring
---

- 최초에는 서블릿 API 컨테이너가 요청당 스레드 하나만 사용(컨테이너가 요청을 받아 처리를 끝내고 클라이언트에 응답을 돌려주기 전까지 스레드는 항상 블로킹 됐었음)  
- 서블렛 3부터, HTTP 요청을 비동기로 처리할 수 있었음  
- 스프링 5부터는 리엑티브 웹 애플리케이션 개발이 가능  
- 리엑티브 프로그래밍은 한마디로 넌블로킹 함수형 프로그래밍을 실천하는 방법  

## 컨트롤러에서 HTTP 요청을 비동기 처리하기
- HTTP 요청을 비동기 처리하기 위해서 TaskExecutor를 사용한다.
- 서블릿을 등록할때 isAsyncSupport를 true로 해주면 서블릿이 비동기로 작동한다.
- isAsyncSupport의 기본값은 true이다.
- HTTP 요청을 비동기 처리하는 4가지 방법
    - Callable
    - DeferredResult
    - CompletableFuture
    - ListenableFuture


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

## HttpMessageConverter 인프라를 사용하여 데이터를 청크로 나누어 전송하기
- (@ResponseBody와 비슷)
- ResponseBodyEmitter 사용
- SseEmitter 사용
    - event-stream 기반

***ResponseBodyEmitter와 SseEmitter 사용 ***

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

## AsyncHandlerInterceptor 인터셉터 활용하기
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


참고자료  
[WebApplicationInitializer로 Spring MVC 설정하기](http://blog.naver.com/PostView.nhn?blogId=take0415&logNo=221017059396&redirect=Dlog&widgetTypeCall=true)


참고도서  
제목: 스프링5레시피(4판)  
지은이: 마틴데니엄, 다니엘 루비오, 조시 롱  
옮긴이: 이일웅  
펴낸곳: 한빛미디어  
