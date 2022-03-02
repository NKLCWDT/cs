# Aspect Orientied Programming

- [Proxy Pattern](https://techvu.dev/112?category=975490)
- [JDK Dynamic Proxy](https://github.com/BAEKJungHo/deepdiveinreflection/blob/main/contents/JDK%20Dynamic%20Proxy.md)
- [CGLIB](https://github.com/BAEKJungHo/deepdiveinreflection/blob/main/contents/CGLIB.md)

## 정의

- AOP 는 관점 지향 프로그래밍이라고 한다.
- 관점 지향 프로그래밍의 핵심은 `역할`과 `책임`을 적절하게 분리하는데에 있다.
  - 여러 비지니스 로직에 공통으로 적용되어있는 `부가 기능 로직`을 따로 `분리`하여 관리할 수 있다.

## ProxyFactory

![cglib](https://user-images.githubusercontent.com/47518272/155869847-b90dd6f0-5375-4642-abc2-89ba0eb0b839.png)

- 스프링은 `ProxyFactory` 라는 기능을 사용하여 인터페이스 여부에 따라서 JDK Dynamic Proxy 를 쓸지, CGLIB 을 쓸지 정한다.

![proxyfactory](https://user-images.githubusercontent.com/47518272/155869966-c338d163-7186-4088-8087-c471b0519f3c.png)

- 부가 기능을 적용하기 위해서 JDK Dynamic Proxy 의 InvocationHandler 와 CGLIB 의 MethodInterceptor 를 따로 만들 필요 없고 `Advice` 만 만들면 된다.
  - 즉, InvocationHandler 나 MethodInterceptor 는 Advice 를 호출하게 된다.

![advice](https://user-images.githubusercontent.com/47518272/155869975-46984cad-3cd5-4a9c-be7b-14a28d43481a.png)

## Advice 만드는 방법

> Advice 는 프록시에서 제공할 부가 기능 로직이라고 생각하면 된다.

- 스프링이 제공하는 MethodInterceptor(CGLIB 과 이름이 동일하다.)
- MethodInterceptor 는 Interceptor 를 상속하고 Interceptor 는 Advice 인터페이스를 상속한다.

```java
package org.aopalliance.intercept;

public interface MethodInterceptor extends Interceptor {
  Object invoke(MethodInvocation invocation) throws Throwable;
}
```

- __Advice__

```java
@Slf4j
public class TimeAdvice implements MethodInterceptor {

    @Override
    public Object invoke(MethodInvocation invocation) throws Throwable {
        StopWatch stopWatch = new StopWatch();
        stopWatch.start();

        // target 클래스를 호출하고 결과를 받는다
        // ProxyFactory 로 프록시를 생성하는 단계에서 이미 target 정보를 파라미터로 받기 때문에
        // 프록시가 호출할 실제 대상인 target 은 invocation 안에 존재한다.
        Object result = invocation.proceed();

        stopWatch.stop();
        log.info(stopWatch.prettyPrint());

        return result;
    }
}
```

- __Test__

```java
@Slf4j
class ProxyFactoryTest {

    @Test
    void interfaceProxy() {
        ServiceInterface target = new ServiceImpl();
        
        // target 이 인터페이스로 구현되어있으면 JDK Dynamic Proxy 생성, 구체 클래스 기반이면 CGLIB Proxy 생성
        ProxyFactory proxyFactory = new ProxyFactory(target);
        proxyFactory.addAdvice(new TimeAdvice());

        ServiceInterface proxy = (ServiceInterface) proxyFactory.getProxy();
        log.info("targetClass={}", target.getClass());
        log.info("proxyClass={}", proxy.getClass());

        proxy.save();

        assertThat(AopUtils.isAopProxy(proxy)).isTrue();
        assertThat(AopUtils.isJdkDynamicProxy(proxy)).isTrue();
        assertThat(AopUtils.isCglibProxy(proxy)).isFalse();
    }
}
```

> `proxyFactory.setProxyTargetClass(true);` 를 적용하면 인터페이스가 있어도 CGLIB 을 사용한다.

## Pointcut, Advice, Advisor

- __Pointcut__
  - 어디에 부가 기능을 적용할지 판단하는 `필터링 로직`
  - 주로 클래스와 메서드 이름으로 필터링
- __Advice__
  - 프록시가 호출하는 `부가 기능 로직`(프록시 로직)
- __Advisor__
  - Pointcut + Advice 한쌍을 의미
  - Advisor 는 `어떤 부가 기능 로직`을 `어디에` 적용할지를 알고 있다.

> AOP 의 핵심은 역할과 책임을 잘 구분하는 것이다.(핵심 기능과 부가 기능을 분리하는 것) Pointcut 은 필터링 로직만 담당하고, Advice 는 부가 기능 로직만 담당한다.

### 다양한 포인트컷

- __AspectJExpressionPointcut__
  - spectJ 표현식으로 매칭한다.
  - 실무에서 가장 많이 사용되는 포인트컷이다. 
  - ```java
    @Pointcut("execution(public * come.weave..repository.*.*(..))")
    public void webLog(){}

    @Around("webLog()")
    public Object doAround(ProceedingJoinPoint joinPoint) throws Throwable {
        // 스톱워치로 시간 측정
        StopWatch stopWatch = new StopWatch();
        stopWatch.start();
        
        // 생략
    }
    ```
- __NameMatchMethodPointcut__
  - 메서드 이름을 기반으로 매칭한다. 내부에서는 PatternMatchUtils 를 사용한다.
- __JdkRegexpMethodPointcut__
  - JDK 정규 표현식을 기반으로 포인트컷을 매칭한다.
- __TruePointcut__
  - 항상 참을 반환한다.
- __AnnotationMatchingPointcut__
  - 어노테이션으로 매칭한다.

## @Aspect

- 스프링은 `@Aspect` 어노테이션으로 Advisor 생성을 지원한다.
- @Aspect 는 관점 지향 프로그래밍(AOP)을 가능하게 하는 AspectJ 프로젝트에서 제공하는 어노테이션이다.
- @Aspect 를 통해서 스프링이 알아서 프록시를 생성해 주는 것이다.

```java
@Slf4j
@Aspect
public class LogTraceAspect {
 
 private final LogTrace logTrace;
 
 public LogTraceAspect(LogTrace logTrace) {
    this.logTrace = logTrace;
 }
 
 // Advisor
 @Around("execution(* hello.proxy.app..*(..))")
 public Object execute(ProceedingJoinPoint joinPoint) throws Throwable {
   TraceStatus status = null;
   
   // log.info("target={}", joinPoint.getTarget()); // 실제 호출 대상
   // log.info("getArgs={}", joinPoint.getArgs()); // 전달인자
   // log.info("getSignature={}", joinPoint.getSignature()); // join point 시그니처
   
   try {
       String message = joinPoint.getSignature().toShortString();
       status = logTrace.begin(message);
       
       // 로직 호출
       Object result = joinPoint.proceed();
       
       logTrace.end(status);
       return result;
     } catch (Exception e) {
       logTrace.exception(status, e);
       throw e;
     }
   }
}
```

- `@Around` 는 target 로직 실행 전, 후에 적용된다.
- AOP 를 빈으로 등록하기 위해서는 ComponentScan 방법을 쓰거나, 수동 빈으로 등록하면 된다.

![advisor](https://user-images.githubusercontent.com/47518272/155872536-d4edffa5-0b44-4ec1-9d37-1a97b1c75bc5.png)

## 횡단 관심사(cross-cutting concerns)

- 횡단 관심사(cross-cutting concerns) 는 애플리케이션의 여러 기능들 사이에 적용되는 부가 기능 로직을 말한다.
  - Ex. Log 를 남기는 기능은 `3 Layer Architectures` 전반에 걸쳐서 동작 되어야 한다.

## AOP 

- 스프링은 하나의 프록시에 여러 Advisor 를 적용할 수 있도록 해준다. 따라서, target 에 여러 AOP 가 동시에 적용되더라도, 스프링 AOP 는 target 마다 하나의 프록시만 생성한다.
- AOP 는 OOP 를 대체하기 위한 것이 아니라 횡단 관심사를 깔끔하게 처리하기 어려운 OOP 의 부족한 부분을 보조하는 목적으로 개발되었다.
- 실무에서는 AOP 를 구현하기 위해 `AspectJ 프레임워크`를 주로 사용한다.

## @ControllerAdvice

- __@ControllerAdvice 는 특정 컨트롤러에서 발생하는 예외 처리와 같은 작업을 전역적으로 처리할 수 있게 해준다.__
  - 즉, @ControllerAdvice 와 @ExceptionHandler 를 통해서, 기존 컨트롤러에서 처리되어야하는 `API 예외` 를 `Advice(부가 기능 로직)` 로 판단하여, 별도의 클래스에서 관리할 수 있다.
- __Advice 에서 알 수 있듯이, AOP 를 사용한다.__
- __Controller 에서 알 수 있듯이, Controller 를 대상으로 한다.__

> Why is it called "Controller Advice"?
> 
> The term `Advice` comes from `Aspect-Oriented Programming (AOP)` which allows us to inject cross-cutting code (called "advice") around existing methods. A controller advice allows us to intercept and modify the return values of controller methods, in our case to handle exceptions.

- __특정 경로나 어노테이션등을 @ControllerAdvice 속성에 지정할 수 있다.__
  - @ControllerAdvice("com.reflectoring.controller")
  - @ControllerAdvice(annotations = Advised.class)

- __동작 과정__
  - 실제로 Exception 이 발생하면 DispatcherServlet 의 processHandlerException 메서드에서 예외가 처리된다.
  - 메서드를 쭉 타고 들어가보면 `InvocationHandlerMethod` 까지 도달하게 될 것이다. 

## References

- [인프런. 스프링 핵심 원리 고급](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B3%A0%EA%B8%89%ED%8E%B8/dashboard)
