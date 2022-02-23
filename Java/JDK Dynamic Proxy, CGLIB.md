# JDK Dynamic Proxy

> [Proxy Pattern](https://techvu.dev/112?category=975490)

- __동적 프록시(Dynamic Proxy)__
  - 개발자가 직접 프록시 클래스를 만들 필요 없이, 프록시 객체를 `동적(Runtime)`으로 생성해 주는 기술
- __JDK Dynamic Proxy__
  - JDK 동적 프록시는 `인터페이스를 기반`으로 프록시를 동적으로 만들어준다. 따라서 `인터페이스가 필수`이다.
  - JDK 동적 프록시에 적용할 로직은 `InvocationHandler` 인터페이스를 구현하여 작성하면 된다.

```java
package java.lang.reflect;

public interface InvocationHandler {
    /**
     * @params proxy the proxy instance that the method was invoked on(메서드가 호출된 프록시 자신)
     * @params method 호출한 메서드
     * @params args 메서드를 호출할 때 전달한 인수
     */
    public Object invoke(Object proxy, Method method, Object[] args)
        throws Throwable;
}
```

## Example

- AInterface

```java
public interface AInterface {
    String call();
}
```

- AImpl

```java
@Slf4j
public class AImpl implements AInterface {
    @Override
    public String call() {
        log.info("A 호출");
        return "a";
    }
}
```

- InvocationHandler 구현체
  - InvocationHandler 를 구현하여 JDK 동적 프록시에 적용할 공통 로직을 개발할 수 있다.

```java
@Slf4j
public class TimeInvocationHandler implements InvocationHandler {

    // 프록시가 호출할 대상
    private final Object target;

    public TimeInvocationHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        log.info("TimeProxy 실행");
        long startTime = System.currentTimeMillis();

        // method.invoke(target, args) : 리플렉션을 사용해서 target 인스턴스의 메서드를 실행한다. args 는 메서드 호출 시 넘겨줄 인수이다.
        Object result = method.invoke(target, args);

        long endTime = System.currentTimeMillis();
        long resultTime = endTime - startTime;
        log.info("TimeProxy 종료 resultTime={}", resultTime);
        return result;
    }
}
```

- 테스트

```java
@Test
void dynamicA() {
    AInterface target = new AImpl();
    TimeInvocationHandler handler = new TimeInvocationHandler(target);
    
    /*
      newProxyInstance
      first arg : class loader
      second arg : 어떤 인터페이스 기반으로 프록시를 만들지 지정(클래스 배열), 인터페이스 구현체가 여러개 있을 수 있기 때문이다.
      third arg : InvocationHandler 구현체 -> 프록시가 사용할 로직
     */
    AInterface proxy = (AInterface) Proxy.newProxyInstance(AInterface.class.getClassLoader(), new Class[]{AInterface.class}, handler);

    proxy.call();
    log.info("targetClass={}", target.getClass());
    log.info("proxyClass={}", proxy.getClass());
}
```

- 결과

```
18:42:48.209 [main] INFO reflection.study.dynamicproxy.code.TimeInvocationHandler - TimeProxy 실행
18:42:57.953 [main] INFO reflection.study.dynamicproxy.code.AImpl - A 호출
18:43:02.900 [main] INFO reflection.study.dynamicproxy.code.TimeInvocationHandler - TimeProxy 종료 resultTime=12964
18:43:04.451 [main] INFO reflection.study.dynamicproxy.DynamicProxyTest - targetClass=class reflection.study.dynamicproxy.code.AImpl
18:43:04.451 [main] INFO reflection.study.dynamicproxy.DynamicProxyTest - proxyClass=class com.sun.proxy.$Proxy9
```

### 만약에, 구체 클래스를 사용한다면 ?

- 구체 클래스

```java
@Slf4j
public class ConcreteC {
    public String call() {
        log.info("C 호출");
        return "C";
    }
}
```

- 테스트

```java
@Test
void concreteTest() {
    ConcreteC target = new ConcreteC();
    TimeInvocationHandler handler = new TimeInvocationHandler(target);

    ConcreteC proxy = (ConcreteC) Proxy.newProxyInstance(ConcreteC.class.getClassLoader(), new Class[]{ConcreteC.class}, handler);

    proxy.call();
    log.info("targetClass={}", target.getClass());
    log.info("proxyClass={}", proxy.getClass());
}
```

- 결과
  - 인터페이스가 아니기 때문에 프록시 객체를 생성하는 과정에서 에러가 발생한다.

```java
java.lang.IllegalArgumentException: reflection.study.dynamicproxy.code.ConcreteC is not an interface

	at java.base/java.lang.reflect.Proxy$ProxyBuilder.validateProxyInterfaces(Proxy.java:688)
	at java.base/java.lang.reflect.Proxy$ProxyBuilder.<init>(Proxy.java:627)
	at java.base/java.lang.reflect.Proxy$ProxyBuilder.<init>(Proxy.java:635)

  // 생략
```

- 올바른 테스트 코드

```java
@Test
void concreteTest() {
ConcreteC target = new ConcreteC();
TimeInvocationHandler handler = new TimeInvocationHandler(target);
assertThatThrownBy(() ->
	    Proxy.newProxyInstance(ConcreteC.class.getClassLoader(), new Class[]{ConcreteC.class}, handler))
	.isInstanceOf(IllegalArgumentException.class);
}
```

## 실행 순서 정리

```
Client -> call() 
-> `$proxy1`(동적 프록시 객체) 
-> handler.invoke() -> InvocationHandler 구현체
-> method.invoke -> 인터페이스 구현체의 메서드 호출(Ex. call())
```

## 장점

JDK Dynamic Proxy 을 도입함으로써 무슨 장점이 있을까?

- __도입 전__
	- 인터페이스 만큼(Ex. AInterface, BInteface ...) 프록시 객체를 `직접 생성`해줘야 한다.
- __도입 후__
	- JDK Dynamic Proxy 가 알아서 생성해 준다.

![jdkdynamic](https://user-images.githubusercontent.com/47518272/154453994-ba0a9fbf-9a95-4aac-86b3-6314dcafa5d7.png)

## 한계

JDK Dynamic Proxy 는 `인터페이스`가 필수이다. 만약에, 인터페이스 없이 클래스만 존재하는 경우에는 어떻게 동적 프록시를 적용해야 할까?

이러한 한계를 극복하고자 바이트 코드를 조작는 `CGLIB` 라이브러리를 사용해야 한다. 

## References

- [Proxy (Java Plaform SE 8)](https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/Proxy.html)
- [JDK Dynamic Proxies](https://www.byteslounge.com/tutorials/jdk-dynamic-proxies)

-------

# CGLIB

> CGLIB : Code Generator Library

- CGLIB 은 바이트코드를 조작해서 동적으로 클래스를 생성하는 기술을 제공하는 라이브러리이다.
- CGLIB 은 JDK Dynamic Proxy 와 달리 인터페이스 없이, 구체 클래스만 가지고 동적 프록시를 생성할 수 있다.
- CGLIB는 구체 클래스를 상속(extends)해서 프록시를 만든다.
    - 따라서, 상속에 의한 제약조건이 생긴다.
    - CGLIB는 자식 클래스를 동적으로 생성하기 때문에 기본 생성자가 필요하다.
    - 클래스 혹은 메서드에 final 이 붙으면 안된다.
- CGLIB 은 외부 라이브러리이지만, 스프링 프레임워크가 스프링 내부 소스 코드에 포함했다. 따라서, 스프링을 사용하는 경우 별도의 라이브러리를 추가할 필요 없이 사용할 수 있다.

![cglib](https://user-images.githubusercontent.com/47518272/154974123-265cc984-840a-4ef5-9736-4aef895f9c36.png)

```java
package org.springframework.cglib.proxy;

import java.lang.reflect.Method;

public interface MethodInterceptor extends Callback {
    /**
     * @params obj : CGLIB 이 적용된 객체
     * @params method : 호출된 메서드
     * @params args : 메서드를 호출하면서 전달된 인수
     * @params proxy : 메서드 호출에 사용
     */
    Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable;
}
```

## Example

> Proxy 는 항상 호출할 대상(target) 이 필요하다.

- ConcreteC

```java
@Slf4j
public class ConcreteC {
    public String call() {
        log.info("C 호출");
        return "C";
    }
}
```

- MethodInterceptor 구현체
  - MethodInterceptor 를 구현하여 CGLIB 프록시의 실행 로직을 정의한다.

```java
@Slf4j
public class TimeMethodInterceptor implements MethodInterceptor {

    // 프록시가 호출할 실제 대상
    private final Object target;

    public TimeMethodInterceptor(Object target) {
        this.target = target;
    }

    @Override
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
        log.info("TimeProxy 실행");
        long startTime = System.currentTimeMillis();

        // 실제 대상을 동적으로 호출
        Object result = methodProxy.invoke(target, args);

        long endTime = System.currentTimeMillis();
        long resultTime = endTime - startTime;
        log.info("TimeProxy 종료 resultTime={}", resultTime);
        return result;
    }
}
```

> Method 를 사용해도 되는데, CGLIB 자체에서 내부적으로 최적화 하는 코드가 존재해서 MethodProxy 를 사용하여 호출하는 것이 성능상 이점이 있다고 한다.

- 테스트

```java
@Test
void cglib() {
    ConcreteC target = new ConcreteC();

    // Enhancer 가 CGLIB Proxy 를 만든다.
    Enhancer enhancer = new Enhancer();
    
    // CGLIB 는 구체 클래스를 상속 받아서 프록시를 생성할 수 있다.
    // 어떤 구체 클래스를 상속 받을지 정한다.
    enhancer.setSuperclass(ConcreteC.class);
    
    // // 프록시에 적용할 실행 로직을 할당한다.
    enhancer.setCallback(new TimeMethodInterceptor(target));
    
    // 프록시 생성
    // ConcreteC$$EnhancerByCGLIB$$860aca8f@2209
    ConcreteC proxy = (ConcreteC) enhancer.create();
    
    log.info("targetClass={}", target.getClass()); // targetClass=class reflection.study.dynamicproxy.code.ConcreteC
    log.info("proxyClass={}", proxy.getClass()); // proxyClass=class reflection.study.dynamicproxy.code.ConcreteC$$EnhancerByCGLIB$$860aca8f

    proxy.call();
}
```

![CGLIB2](https://user-images.githubusercontent.com/47518272/154981191-ced061f4-9b99-437b-beb2-d7c038cd2567.png)

## References 

- [인프런. 스프링 핵심원리 고급](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B3%A0%EA%B8%89%ED%8E%B8/dashboard)
