# @Scheduled

스케줄러를 사용하기 위해서는 아래 조건을 다 만족해야 한다.

- Application 클래스에서 `@EnableScheduling` 활성화
- 스케줄러를 제공할 클래스를 빈으로 등록
- 스케줄러에 @Scheduled 사용

```kotlin
@Slf4j
@Component
class TimeScheduler {

    private val log = logger()
    private var formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")

    /** 5초 마다 현재 시간을 로그로 찍는 스케줄러 */
    @Scheduled(fixedRate = 5000)
    fun reportCurrentTime() {
        log.info("The time is now {}", LocalDateTime.now().format(formatter))
    }
}
```

@Scheduled 는 다양한 옵션들을 지원한다. 특히 특정 날짜, 시간에 동작하게끔 할 수도 있다.

```
@Scheduled("0 * * * * MON-FRI")
```

- __@Scheduled cron options__
  - second
  - minute
  - hour
  - day of month
  - month
  - day of week

# @Async

- @Async 는 비동기 작업을 어노테이션으로 처리할 수 있게끔 지원해주는 녀석이다.
- 기본 전략은 비동기 작업마다 스레드를 생성하는 `SimpleAsyncTaskExecutor` 를 사용한다.
- 스레드 관리 전략을 `ThreadPoolTaskExecutor` 로 바꿔서 스레드풀을 사용하게끔 할 수 있다.

## @Async 사용 전

```java
public class GreetingService {

    static ExecutorService executorService = Executors.newFixedThreadPool(10);

    public void method1(final String message) throws Exception {
        executorService.submit(new Runnable() {
            @Override
            public void run() {
                // do something
            }            
        });
    }
}
```

## @Async 를 사용하기 위한 설정

> SimpleAsyncTaskExecutor 사용

```java
@EnableAsync
@SpringBootApplication
public class Application {
    ...
}
```

## @Async 사용

```java
public class GreetingService {

    @Async
    public void method1(String message) throws Exception {
        // do something
    }
}
```

## ThreadPoolTaskExecutor 사용

> 기존에 Application 클래스에서 적용한 @EnableAsync 는 제거해야 한다.

```java
@Configuration
@EnableAsync
public class SpringAsyncConfig {

    @Bean(name = "threadPoolTaskExecutor")
    public Executor threadPoolTaskExecutor() {
        ThreadPoolTaskExecutor taskExecutor = new ThreadPoolTaskExecutor();
        taskExecutor.setCorePoolSize(3);
        taskExecutor.setMaxPoolSize(30);
        taskExecutor.setQueueCapacity(10);
        taskExecutor.setThreadNamePrefix("Executor-");
        taskExecutor.initialize();
        return taskExecutor;
    }
}
```

```java
public class GreetingService {
	
    @Async("threadPoolTaskExecutor")
    public void method1() throws Exception {
        // do something
    }
}
```

스레드 관리 전략을 여러개 가져간다면 저렇게 빈 이름을 지정해주면 된다. 

AsyncConfigurerSupport 클래스를 상속 받아서 스레드 관리 전략을 설정할 수도 있다.

```java
@Configuration
@EnableAsync
public class AsyncConfig extends AsyncConfigurerSupport {
    
    @Override
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(30);
        executor.setQueueCapacity(50);
        executor.setThreadNamePrefix("DDAJA-ASYNC-");
        executor.initialize();
        return executor;
    }
}
```

- `@EnableAsync` : Spring method 에서 비동기 기능을 사용가능하게 활성화 한다.
- `CorePoolSize` : 기본 실행 대기하는 Thread 의 수
- `MaxPoolSize` : 동시 동작하는 최대 Thread 의 수
- `QueueCapacity` : MaxPoolSize 초과 요청에서 Thread 생성 요청시, 해당 요청을 Queue 에 저장하는데 이때 최대 수용 가능한 Queue 의 수, Queue 에 저장되어있다가 Thread 에 자리가 생기면 하나씩 빠져나가 동작
- `ThreadNamePrefix` : 생성되는 Thread 접두사 지정

## @Asycn 사용 시 주의점

- __private method 는 사용 불가, public method 만 사용 가능__
- __self-invocation(자가 호출) 불가, 즉 inner method는 사용 불가__
- __QueueCapacity 초과 요청에 대한 비동기 method 호출시 방어 코드 작성__
    - 최대 수용 가능한 Thread Pool 수와 QueueCapacity 까지 초과된 요청이 들어오면 `TaskRejectedException` 에러가 발생한다.
    - 따라서, TaskRejectedException 에러에 대한 추가적인 핸들링이 필요하다.

__@Async 의 동작은 AOP 가 적용되어__ Spring Context 에서 등록된 Bean Object 의 method 가 호출 될 시에, Spring 이 확인할 수 있고 @Async 가 적용된 method 의 경우 Spring 이 method 를 가로채 다른 Thread 에서 실행 시켜주는 동작 방식이다. 이 때문에 Spring 이 해당 @Async method 를 가로챈 후, 다른 Class 에서 호출이 가능해야 하므로, private method는 사용할 수 없는 것이다.

또한 Spring Context에 등록된 Bean의 method의 호출이어야 Proxy 적용이 가능하므로, inner method 의 호출은 Proxy 영향을 받지 않기에 self-invocation 이 불가능하다.

> AsyncExecutionAspectSupport 클래스의 doSubmit() 메서드에 의해서 @Async 어노테이션을 달면 해당 메서드가 비동기로 동작할 수 있는 것이다.

## 장점

- __메서드에 대한 수정 없이 처리가 가능하다.__ 
    - 기존에는 동기/비동기에 따라서 메서드의 내용이 달라졌다.
    - ExecutorService 를 사용하는 경우 Runnable 의 run() 을 재구현해야 하는 등 동일한 작업들의 반복되었는데, @Async 를 사용하면 그러한 반복작업 없이 코드를 깔끔하게 가져갈 수 있다.
- __동기/비동기에 대한 고민없이 로직 구현 과정에만 집중할 수 있다.__

# @SessionAttributes

- __모델 정보를 HTTP 세션에 저장해준다.__
- 이거대신 HttpSession 을 이용해서 setAttribute 로 HTTP 세션에 저장할 수 도 있다.
- __@SessionAttributes 를 사용하면 이 어노테이션을 설정한 이름에 해당하는 모델 정보를 자동으로 세션에 넣어준다.__
  - 즉, @ModelAttribute를 HTTP Session에 담는다고 생각하면 된다. (단, 키값이 같은경우)
- @ModelAttribute 는 세션에 있는 데이터도 바인딩한다.
- 여러 화면에서 사용해야하는 객체를 공유할 때 사용한다.
  - ex) 장바구니
  - ex) 폼에서 입력하는 값이 많은경우 화면을 나누기도하는데, 그때 A화면에서 input 값 a, b를 받고 그 다음 화면 B에서 c, d를 받아서 조합할 수 있다.

## SessionStatus

- SessionStatus를 사용해서 세션 처리 완료를 알려줄 수 있다.
- 폼 처리 끝나고 세션 비울때 사용한다.

> HttpSession 을 이용하는 것은 low level 이며 @SessionAttributes 는 더 추상화 된 level 이다.

## Example

```java
@Controller
@SessionAttributes("event")  
@RequestMapping("/events")
// @SessionAttributes({"event", "book"} 여러개도 지정가능
public class EventController {
  
  /**
  * @SessionAttributes 를 사용하여 모델에 담긴 event란 이름을 가진 객체를 세션에 담아준다.
  */
  @GetMapping("/form")
  public String eventsForm(Model model) {
    model.addAttribute("event, newEvent);
    return "/book/form";
  }
  
  /**
  * SessionStatus를 사용하여 폼 처리가 끝나고 세션을 완료시킬 수 있다.
  */
  @PostMapping
  public String createEvents(@Validated @ModelAttribute Event event,
                        BindingResult bindingResult,
                        SessionStatus sessionStatus) {
       if(bindingResult.hasErrors()) {
          return "/form";
       }
       sessionStatus.setComplete(); // 즉, 세션에 담긴값을 정리한다.
       return "redirct:/list";
   }
  
}
```

세션에 담긴 attribute 를 꺼내오려면 아래와 같이 하면 된다.

```java
Object event = request.getSession().getAttribute("event");
```

# @SessionAttribute 

- __HTTP 세션에 들어있는 값을 참조할 때 사용__
- HttpSession 을 사용할 때 비해 __타입 컨버전을 자동으로 지원__ 하기 때문에 편리함
- HTTP 세션에 값을 넣고 빼고싶은 경우는 HttpSession 을 사용
- @SessionAttributes 와는 다르다
  - @SessionAttributes 는 해당 컨트롤러 내에서만 작동
    - 즉, 해당 컨트롤러 안에서 다루는 특정 모델 객체를 세션에 넣고 공유할 때 사용
  - __@SessionAttribute 는 컨트롤러밖(인터셉터, 필터 등)에서 만들어준 세션 데이터에 접근할 때 사용__
  
사실 @SessionAttribute 를 이용해서 @SessionAttributes 로 HTTP Session 에 넣어준 값에 접근할 수 있지만 사실 그런 용도는 아니고, 컨트롤러 내에서는 @SessionAttributes, SessionStatus, @ModelAttribute 를 이용해서 다룬다.
  
 ## 웹 사이트에 방문했을때 방문한 시간을 기록 Example
 
 - 웹 사이트에 방문했을때 방문한 시간을 기록을 이용하면, 게임일 경우 2시간이상 과도한 게임할 경우 메시지를 뿌려준다던지 가능하다.
 
 예를들어 웹 사이트에 방문했을때 방문한 시간을 기록하는 인터셉터 VisitTimeInterceptor 가 있다고 가정하겠다.
 
 ```java
 public class VisitTimeInterceptor implements HandlerInterceptor {
 
  @Override
  public boolean preHandle(HttpServletReqeust request, HttpServletResponse response) {
    HttpSession session = request.getSession();
    if(session.getAttribute("visitTime") == null) {
      session.setAttribute("visitTime", LocalDateTime.now());
    }
    return true;
  }
}
```

컨트롤러에서는 아래와 같이 세션에 담긴 visitTime을 가져올 수 있다.

```java
@GetMapping
public String getEvents(Model model, @SessionAttribute LocalateTime visitTime) {
}
```

HttpSession 을 이용해서 꺼낼 수도 있는데 문제는 LocalDateTime 이 아니라 Object 로 나온다. 즉, 그러면 `Type Conversion` 이 일어나게된다. 하지만
@SessionAttribute를 이용하면 `Type Conversion` 을 자동으로 지원해준다.

```java
@GetMapping
public String getEvents(Model model, HttpSession httpSession) {
  Object visitTime = httpSession.getAttribute("visitTime");
}
```

## References

- http://dveamer.github.io/java/SpringAsync.html
- https://velog.io/@gillog/Spring-Async-Annotation%EB%B9%84%EB%8F%99%EA%B8%B0-%EB%A9%94%EC%86%8C%EB%93%9C-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0
