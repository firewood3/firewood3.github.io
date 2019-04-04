---
title: "스프링 모바일"
date: 2019-04-03 22:05:00 -0400
categories: java spring mobile
---

스프링 모바일은 사용자의 디바이스를 감지하는 프레임워크를 제공한다.

## HTTP 헤더 값을 확인하여 모바일 기기 감지하기

스프링 모바일을 사용하지 않고 HTTP 헤더의 User-Agent 값을 분석하여 모바일 기기를 감지할 수도 있지만, 모든 기기에서 전송하는 User-Agent 값을 분석하는 일은 불완전하기 때문에 스프링 모바일을 사용하여 모바일 기기를 감지하는 것이 바람직하다.

*User-Agent 헤더값을 필터링 하여 모바일 기기를 감지하는 코드*
```java
public class DeviceResolverRequestFilter extends OncePerRequestFilter {

    private static final String CURRENT_DEVICE_ATTRIBUTE = "currentDevice";

    private static final String DEVICE_MOBILE = "MOBILE";
    private static final String DEVICE_TABLET = "TABLET";
    private static final String DEVICE_NORMAL = "NORMAL";

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response,
                                    FilterChain filterChain) throws ServletException, IOException {
        String userAgent = request.getHeader("User-Agent");
        String device = DEVICE_NORMAL;

        if (StringUtils.hasText(userAgent)) {
            userAgent = userAgent.toLowerCase();
            if (userAgent.contains("android")) {
                device = userAgent.contains("mobile") ? DEVICE_NORMAL : DEVICE_TABLET;
            } else if (userAgent.contains("ipad") || userAgent.contains("playbook") || userAgent.contains("kindle")) {
                device = DEVICE_TABLET;
            } else if (userAgent.contains("mobil") || userAgent.contains("ipod") || userAgent.contains("nintendo DS")) {
                device = DEVICE_MOBILE;
            }
        }
        
        request.setAttribute(CURRENT_DEVICE_ATTRIBUTE, device);
        filterChain.doFilter(request, response);
    }
}
```

## 스프링 모바일을 이용해 기기 감지하기
스프링 모바일의 DeviceResolverHandlerInterceptor 클래스는 사용하면 모바일 기기의 감지를 쉽게 구현할 수 있다. 다음 코드는 DeviceResolverHandlerInterceptor 클래스인데 HTTP 요청을 해석하여 Device 객체로 반환하고 있다.

```java
public class DeviceResolverHandlerInterceptor extends HandlerInterceptorAdapter {
    private final DeviceResolver deviceResolver;

    public DeviceResolverHandlerInterceptor() {
        this(new LiteDeviceResolver());
    }

    public DeviceResolverHandlerInterceptor(DeviceResolver deviceResolver) {
        this.deviceResolver = deviceResolver;
    }

    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        Device device = this.deviceResolver.resolveDevice(request);
        request.setAttribute("currentDevice", device);
        return true;
    }
}
```

DeviceResolverHandlerInterceptor 사용하기 위해서는 WebMvc 인터셉터에 등록한다.

*DeviceResolverHandlerInterceptor를 WebMvc인터셉터에 추가하기*
```java
@Configuration
public class MobileConfiguration implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new DeviceResolverHandlerInterceptor());
    }
}
```

*Device 객체를 사용하여 디바이스 감지하기*
```java
@GetMapping("/home")
public String home(HttpServletRequest request) {
    
    Device device = (Device) request.getAttribute("currentDevice");
    device.getDevicePlatform(); // eunm DevicePlatForm : IOS, ANDROID, UNKOWN
    device.isMobile(); // boolean
    device.isNormal(); // boolean
    device.isTablet(); // boolean
    return "home";
}
```

## 디바이스 기기 정보를 선택하고 저장하기
스프링 모바일에서는 사용자로부터 디바이스 기기 정보를 입력받고, 이를 저장해 두는 특별한 인터셉터를 제공한다. 이 인터셉터의 구현체가 SitePreferenceHandlerInterceptor 클래스인데 이를 사용하면 요청 파라미터(site_preference)를 통해 디바이스 정보를 선택받고, 선택받은 값을 SitePreference 열거형 값으로 저장 할 수 있다.(기본 저장소는 쿠키)

***유저의 로케일을 선택받고 쿠키에 저장하는 LocaleChangeResolver 인터페이스와 닮아 있다!***

*SitePreference 열거형*
```java
public enum SitePreference {
    ...
    NORMAL
    ...
    MOBILE
    ...
    TABLET
    ...
}
```

SitePreferenceHandlerInterceptor는 WebMvc에 추가해주면 바로 사용할 수 있다.

*SitePreferenceHandlerInterceptor 추가하기*
```java
@Configuration
public class MobileConfiguration extends WebMvcConfigurationSupport {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        ...
        registry.addInterceptor(new SitePreferenceHandlerInterceptor());
    }
}
```

[샘플코드 : 디바이스 감지하기와 입력받은 디바이스 값 쿠키에 저장하기](https://github.com/firewood3/spring/tree/master/spring-mobile/mobile-filter)


## 기기 정보에 따라 뷰 렌더링 하기

### 방법1 수동으로 뷰 렌더링 선택하기
다음과 같이 HTTP 요청에서 Device 정보를 읽고 읽어들인 디바이스 정보를 분석하여 각각 다른 뷰를 내려주도록 코딩할 수 있다.

```java
@GetMapping("/home")
public String home(HttpServletRequest request) {
    Device device = DeviceUtils.getCurrentDevice(request);
    if (device.isMobile()) {
        return "mobile/home";
    } else if (device.isTablet()) {
        return "tablet/home";
    } else {
        return "home";
    }
}
```

다음과 같이 WebMvc 구성에 DeviceHandlerMethodArgumentResolver를 빈으로 등록하면 Device 값을 컨트롤러 메서드의 인수로 전달 받을 수 있다.
```java
@Configuration
public class MobileConfiguration extends WebMvcConfigurationSupport {
    ...
    @Override
    protected void addArgumentResolvers(List<HandlerMethodArgumentResolver> argumentResolvers) {
        argumentResolvers.add(new DeviceHandlerMethodArgumentResolver());
    }
}
```

```java
@GetMapping("/home")
public String home(Device device) {
    if (device.isMobile()) {
        return "mobile/home";
    } else if (device.isTablet()) {
        return "tablet/home";
    } else {
        return "home";
    }
}
```

### 방법2 뷰 렌더링 선택 자동화 하기
스프링 모바일의 LiteDeviceDelegatingViewResolver를 이용하면 Device 값과 SitePreferences값을 해석하여 뷰 이름을 진짜 뷰 리졸버로 넘겨 주기 전에 추가 접두어/접미어를 덧붙일 수 있다.

*LiteDeviceDelegatingViewResolver를 빈으로 등록*
```java
@Bean
public ViewResolver mobileViewResolver() {
    LiteDeviceDelegatingViewResolver delegatingViewResolver = new LiteDeviceDelegatingViewResolver(viewResolver());
    delegatingViewResolver.setOrder(1);
    delegatingViewResolver.setMobilePrefix("mobile/");
    delegatingViewResolver.setTabletPrefix("tablet/");
    return delegatingViewResolver;
}
```

*LiteDeviceDelegatingViewReoslver 클래스에서 뷰를 해석하는 코드*
```java
public class LiteDeviceDelegatingViewResolver extends AbstractDeviceDelegatingViewResolver {
...
	@Override
	protected String getDeviceViewNameInternal(String viewName) {
    	if (ResolverUtils.isNormal(device, sitePreference)) {
			resolvedViewName = getNormalPrefix() + viewName + getNormalSuffix();
		} else if (ResolverUtils.isMobile(device, sitePreference)) {
			resolvedViewName = getMobilePrefix() + viewName + getMobileSuffix();
		} else if (ResolverUtils.isTablet(device, sitePreference)) {
			resolvedViewName = getTabletPrefix() + viewName + getTabletSuffix();
		}
    }
}
```

LiteDeviceDelegatingViewResolver로 디바이스 별로 뷰를 선택하는 작업을 위임했으므로, 다음과 같이 핸들러 매핑 메소드는 간단해진다.

```java
@GetMapping("/home")
public String home() {
    return "home";
}
```

[샘플코드 : 스프링 모바일 / 디바이스에 따라 다른 뷰 랜더링 하기](https://github.com/firewood3/spring/tree/master/spring-mobile/mobile-depended-view)


## 디바이스에 따라 사이트 스위칭 구현하기
스프링 모바일의 SiteSwitcherHandlerInterceptor 클래스는 감지한 Device값에 따라 모바일 전용 웹 사이트로 전환할 수 있다. 다음과 같은 팩토리 메소드를 사용하여 리다이렉트 되는 URL을 설정하고 SiteSwitcherHandlerInterceptor 객체를 생성한다.

*SiteSwitcherHandlerInterceptor 팩토리 메소드*

| 팩토리 메서드 | 설명 | 
| ---- | ---- |
| mDot() | m.으로 시작하는 도메인으로 리다이렉트 합니다. <br>ex) http://www.yourdomain.com -> http://m.yourdomain.com |
| dotMobi() | .mobi로 끝나는 도메인으로 리다이렉트 합니다. <br>ex) http://www.yourdomain.com -> http://www.yourdomain.mobi |
| urlPath() | 기기마다 서로다른 컨텍스트 루트를 설정해서 정해진 URL 경로로 리다이렉트 합니다. <br>ex) http://www.yourdomain.com -> http://www.yourdomain.com/mobile |
| standard() | 웹 사이트의 도메인을 모바일, 태블릿, 그리고 일반 버전별로 지정 가능한 가장 유연한 팩토리 메서드 입니다. |

***mDot, dotMobi, urlPath는 모두 standard 메소드를 사용한다.***

### urlPath() 펙토리 메소드를 사용하여 사이트 스위칭 구현하기
SiteSwitcherHandlerInterceptor 클래스를 urlPath() 펙토리 메소드로 생성하고 빈으로 등록하고 WebMvc의 인터셉터로 등록한다.

*디바이스별 리다이렉트 주소*
- 모바일: /m
- 테블릿: /t
- 노멀: /

```java
@Bean
public SiteSwitcherHandlerInterceptor siteSwitcherHandlerInterceptor() {
    return SiteSwitcherHandlerInterceptor.urlPath("/m", "/t", "/");
}
@Override
protected void addInterceptors(InterceptorRegistry registry) {
    registry.addInterceptor(siteSwitcherHandlerInterceptor());
}
```

디바이스별로 리다이렉트되는 요청을 받을 수 있도록 컨트롤러를 각각 만들어 준다.
```java
@Controller
@RequestMapping("/m")
public class MobileController {
    @GetMapping("/home")
    public String home() {
        return "home";
    }
}

@Controller
@RequestMapping("/t")
public class TabletController {
    @GetMapping("/home")
    public String home() {
        return "home";
    }
}

@Controller
public class NormalController {
    @GetMapping("/home")
    public String home() {
        return "home";
    }
}
```

[샘플코드 : 스프링 모바일 / 디바이스에 따라 사이트 스위칭 구현하기](https://github.com/firewood3/spring/tree/master/spring-mobile/mobile-switching-site)


***
참고도서  
제목: 스프링5레시피(4판)  
지은이: 마틴데니엄, 다니엘 루비오, 조시 롱  
옮긴이: 이일웅  
펴낸곳: 한빛미디어  
