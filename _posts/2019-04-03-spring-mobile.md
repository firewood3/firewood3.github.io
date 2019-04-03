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





***
참고도서  
제목: 스프링5레시피(4판)  
지은이: 마틴데니엄, 다니엘 루비오, 조시 롱  
옮긴이: 이일웅  
펴낸곳: 한빛미디어  
