## ViewResolver

dispatcherServlet은 Handler Adapter에서 받아온 view 이름을 가지고 viewResolver에게 이러한 view가 있는지 물어보고 viewResolver는 view가 존재하는지 아닌지 판단하는 역할을 하게된다.

즉, ViewResolver는 사용자가 요청한 것에 대한 응답 view를 렌더링하는 역할이다.

### Springboot ViewResolver

SpringBoot를 사용하면서 view로는 thymeleaf를 사용했던 경험이 있는데 이 때 ViewResolver를 생성하지도 Bean으로 등록하지도 않았는데 어떻게 처리되는지에 대해서 알아보겠다.

일단 먼저 DispatcherServlet은 SpringBoot에 자동설정 기능으로 자동 등록 된다.

```java
package org.springframework.boot.autoconfigure.web.servlet;

@AutoConfigureOrder(-2147483648)
@Configuration
@ConditionalOnWebApplication(
    type = Type.SERVLET
)
@ConditionalOnClass({DispatcherServlet.class})
@AutoConfigureAfter({ServletWebServerFactoryAutoConfiguration.class})
public class DispatcherServletAutoConfiguration {
    public static final String DEFAULT_DISPATCHER_SERVLET_BEAN_NAME = "dispatcherServlet";
    public static final String DEFAULT_DISPATCHER_SERVLET_REGISTRATION_BEAN_NAME = "dispatcherServletRegistration";

    public DispatcherServletAutoConfiguration() {
    }
    ...
}
```

DispatcherServlet에서 initViewResolvers를 실행한다.

```java
package org.springframework.web.servlet;

public class DispatcherServlet extends FrameworkServlet {
    
    ...
    protected void initStrategies(ApplicationContext context) {
        this.initMultipartResolver(context);
        this.initLocaleResolver(context);
        this.initThemeResolver(context);
        this.initHandlerMappings(context);
        this.initHandlerAdapters(context);
        this.initHandlerExceptionResolvers(context);
        this.initRequestToViewNameTranslator(context);
        this.initViewResolvers(context);
        this.initFlashMapManager(context);
    }
    ...
}
```

기본적으로 빈으로 등록된 viewResolver를 찾아온다. (커스텀으로 등록된 viewResolver가 있는지) 만약 없다면 기본전략에 따라 InternalResourceViewResolver를 사용한다.

InternalResourceViewResolver는 기본적으로 jsp를 지원한다는 특징이 있다.

```java
package org.springframework.web.servlet;

public class DispatcherServlet extends FrameworkServlet {
    private void initViewResolvers(ApplicationContext context) {
        this.viewResolvers = null;
        if (this.detectAllViewResolvers) {
            Map<String, ViewResolver> matchingBeans = BeanFactoryUtils.beansOfTypeIncludingAncestors(context, ViewResolver.class, true, false);
            if (!matchingBeans.isEmpty()) {
                this.viewResolvers = new ArrayList(matchingBeans.values());
                AnnotationAwareOrderComparator.sort(this.viewResolvers);
            }
        } else {
            try {
                ViewResolver vr = (ViewResolver) context.getBean("viewResolver", ViewResolver.class);
                this.viewResolvers = Collections.singletonList(vr);
            } catch (NoSuchBeanDefinitionException var3) {
            }
        }

        if (this.viewResolvers == null) {
            this.viewResolvers = this.getDefaultStrategies(context, ViewResolver.class);
            if (this.logger.isTraceEnabled()) {
                this.logger.trace("No ViewResolvers declared for servlet '" + this.getServletName() + "': using default strategies from DispatcherServlet.properties");
            }
        }
    }
}
```

SpringBoot에서는 InternalResourceViewResolver를 기본 viewResolver로 사용하는데 jsp를 지원하지 않기 때문에 thymeleaf를 사용하거나

jsp를 사용할 수 있도록 만들어줘야한다.

### 스프링부트에서 thymeleaf 사용하는법

스프링부트에서 spring-starter-thymeleaf 의존성을 주입하고 프로젝트 생성 시 만들어져있는 template 폴더안에 html만 만들어주면된다.

<img width="185" alt="image" src="https://user-images.githubusercontent.com/70622731/158051837-71593f34-34f2-4795-936a-9b825a926360.png">

```java
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
}
```

컨터롤러 예시

```java
@Controller
public class TestController {

    @GetMapping("/test")
    public String welcome() {
        return "index";
    }
}
```

컨트롤러에서 index를 호출하기만 하면 spring-starter-thymeleaf안에 있는 viewResolver가 template 폴더안에 index.html 폴더를 찾아 view로 보여지게 한다.

> 여기서 만약 @ResponseBody가 붙어있다면 리턴값은 view를 통해 출력되지 않고 HTTP Response Body에 직접 쓰여진다.
> 이때, 해당 메소드 리턴값의 데이터 타입에 따라 MessageConverter에서 변환이 이뤄진 후 쓰여진다.

spring-starter-thymeleaf 안에 AbstractCachingViewResolver를 상속받아 thymeleaf viewResolver가 구현되어 있어 SpringBoot DispatcherServlet에서 viewResolver를 찾을 때 thymeleaf viewResolver를 찾게된다.

```java
package org.thymeleaf.spring5.view;

public class ThymeleafViewResolver extends AbstractCachingViewResolver implements Ordered {
    ...
}
```

### 스프링부트에서 jsp 사용하는법

스프링 공식문서에 보면 내장된 서블릿 컨테이너에는 jsp 제한사항이 있다.

> https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-developing-web-applications

스프링부트는 가능하다면 jsp를 피하고 Thymeleaf와 같은 템플릿 엔진을 사용하라고 권장한다

jsp 파일은 Springboot의 templates 폴더안에서 작동하지 않는다. 그래서 jsp를 적용하기 위해서는 아래와 같은 의존성을 추가해야한다.

```java
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>jstl</artifactId>
</dependency>
<dependency>
    <groupId>org.apache.tomcat.embed</groupId>
    <artifactId>tomcat-embed-jasper</artifactId>
</dependency>
```

기본적으로 스프링부트는 내장톰캣을 가지고 있지만, jsp 엔진이 존재하지 않기 때문에 jasper와 jsp의 라이브러리 jstl을 사용할 수 있는 의존성을 추가해줘야한다.

그리고 뷰의 경로를 지정해줘야 스프링부트가 jsp를 찾아 사용할 수 있다.

```java
// application.properties
spring.mvc.view.prefix = /WEB-INF/
spring.mvc.view.suffix = .jsp
```

위와 같이 view 이름 prefix, suffix에 붙는 것들을 설정할 수 있다.

viewResolver에서 찾을 수 있도록 폴더 구조는 webapp/WEB-INF/ 안에 .jsp 파일들을 넣어주면 된다. 

> spring-starter-thymeleaf 의존성이 들어있으면 viewResolver가 선언되어있어서 InternalResourceViewResolver를 viewResolver로 등록하지못해 jsp를 찾지 못한다.

---

### Reference

- https://galid1.tistory.com/527
- https://goddaehee.tistory.com/204
- https://devlog-wjdrbs96.tistory.com/199
