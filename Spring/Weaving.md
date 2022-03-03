# 위빙(Weaving)

AOP 는 `위빙(Weaving)` 이라는 방법을 사용하여 부가 기능 로직(Advice)을 실제 로직에 추가한다.

> 핵심 로직에 부가 기능 로직이 추가되는 것을 위빙(Weaving) 이라고 생각하면 된다.

## Compile Time Weaving(CTW)

- __`.java` 소스 코드를 컴파일러를 사용해서 `.class` 를 만드는 시점에 부가 기능 로직을 추가할 수 있다.__
  - AspectJ 가 제공하는 특별한 컴파일러를 사용해야 한다.
  - 컴파일 된 `.class` 를 디컴파일 해보면 Aspect 관련 호출 코드가 들어간다.
  - 즉, 부가 기능 로직이 핵심 로직이 있는 컴파일된 코드 주변에 실제로 붙는다. 
  - AspectJ 컴파일러는 Aspect를 확인해서 해당 클래스가 적용 대상인지 먼저 확인하고, 적용 대상인 경우에 부가 기능 로직을 적용한다.
- __단점__
  - 컴파일 시점에 부가 기능을 적용하려면 특별한 컴파일러도 필요하고 복잡하다.

## Load Time Weaving(LTW)

- 자바를 실행하면 자바 언어는 .class 파일을 JVM 내부의 클래스 로더에 보관한다.
- 이때 중간에서 .class 파일을 조작한 다음 JVM 에 올릴 수 있다.
- __자바 언어는 .class 를 JVM 에 저장하기 전에 조작할 수 있는 기능을 제공한다.__
  - JVM 이 제공하는 [Java Instrumentation API](https://www.baeldung.com/java-instrumentation) 를 활용하여 JVM 에 로드된 기존 바이트 코드를 변경한다.
- 이 시점에 Aspect 를 적용하는 것을 LTW 라고 한다.

> 수 많은 모니터링 툴들이 이 방식을 사용한다.

- __단점__
  - 로드 타임 위빙은 자바를 실행할 때 특별한 옵션(`java -javaagent`)을 통해 클래스 로더 조작기를 지정해야 하는데, 이 부분이 번거롭고 운영하기 어렵다.

## Run Time Weaving(RTW)

- 말 그대로 자바 main 메서드가 실행된 다음, `Runtime 에 발생`하는 위빙이다.
- 스프링과 같은 컨테이너의 도움을 받고 프록시와 DI, 빈 포스트 프로세서 같은 개념들을 총 동원해야 한다.
- 이렇게 하면 최종적으로 프록시를 통해 스프링 빈에 부가 기능을 적용할 수 있다.
- __이것이, `스프링`에서 차용하고 있는 방식이며 `프록시 방식의 AOP` 이다.__
- 스프링만 있으면 얼마든지 AOP 를 적용할 수 있으며, 프록시를 통해 부가 기능을 사용하는 것이다.

> [Spring AOP Docs](https://docs.spring.io/spring-framework/docs/3.2.x/spring-framework-reference/html/aop.html#aop-introduction-defn). Spring AOP, like other pure Java AOP frameworks, performs weaving at runtime.

- __단점__
  - 프록시를 사용하기 때문에 AOP 기능에 일부 제약이 있다.

> 스프링 AOP는 AspectJ의 문법을 차용하고, 프록시 방식의 AOP를 제공한다. AspectJ를 직접 사용하는 것이 아니다.
> 
> 스프링 AOP를 사용할 때는 @Aspect 어노테이션을 주로 사용하는데, 이 어노테이션도 AspectJ가 제공하는 어노테이션이다.

## AspectJ

- __@Aspect 를 포함한 org.aspectj 관련 기능은 `aspectjweaver.jar` 라이브러리에서 제공한다.__
  - ![aspectjweaver](https://user-images.githubusercontent.com/47518272/156456214-c77672c2-c870-4c0c-85a1-65e91a457524.png)
- __spring-boot-starter-aop__
  - 스프링의 AOP 관련 기능과 함께 aspectjweaver.jar 도 함께 사용할수 있다.
  - 그런데 스프링에서는 AspectJ가 제공하는 애노테이션이나 관련 인터페이스만 사용하는 것이고, 실제 AspectJ 가 제공하는 컴파일, 로드타임 위버 등을 사용하는 것은 아니다.
  - ![starteraop](https://user-images.githubusercontent.com/47518272/156457054-e392e0ea-ca48-4183-9e9a-0d4bbd4983c6.png)

## AspectJ 에서 제공하는 LTW 는 언제 활용될까?

- __LTW 는 언제 활용되는 것일까?__
  - `@Configurable 지원`
    - [@Configurable 활용 사례는 해당 링크를 통해 확인하자](https://dhsim86.github.io/web/2019/05/21/spring_@configurable-post.html)
    - 해당 아티클 내용을 보면 스프링에서 `DDD` 개념을 도입하는 경우 @Configurable 을 사용한다고 나와있다. 
    - 해당 아티클 Domain 객체 내부에 Repository 가 들어가 있는 것을 볼 수 있는데, 이러한 패턴을 [Active Record](https://martinfowler.com/eaaCatalog/activeRecord.html) 라고 부른다.
    - 자세한 내용이 궁금하면 martinfowler 의 [P of EAA](https://www.martinfowler.com/books/eaa.html) 를 참고하자.
    - 그러면 Hibernate JPA 를 사용할 때 쓰는 패턴은 이름이 뭘까 찾아봤는데, [Data Mapper](https://martinfowler.com/eaaCatalog/dataMapper.html) 라고 부른다.
    > 지금 당장 깊게 보기보단, 실무에서 DDD 를 사용 중이라면 그 때 깊게 공부하면 될 것 같다.
  - `트랜잭션 AOP 모드를 AspectJ 로 설정한 경우`
    - `<tx:annotation-driven mode="aspectj"/>`
    - 이 경우에 LTW 를 사용한다.
  - `AspectJ 가 아니라 JPA 에서 필요로 하는 LTW 로 사용되는 경우`
    - JPA 의 구현체에 따라서 다르지만 대부분 LTW 를 이용한 바이트코드 조작을 필요로 한다.
    - POJO 로 만든 도메인 객체에 지연된 로딩이나 변경 감지, 그룹 조회, JOIN 을 이용한 로딩 및 최적화 기능을 적용하려면 POJO 클래스의 바이트코드를 조작해야 한다. 이를 위해 JPA 는 각 구현체마다 전용 로드 타임 위버를 제공한다.
      - Ex. EclipseLink 는 eclipselink.jar 파일을 javaagent 로 설정해서 LTW 를 적용한다.
      - __독자적인 방식을 사용하는 Hibernate JPA 를 제외하면 대부분 JPA 구현체는 LTW 를 사용하도록 요구하고 있다.__
      - EclipseLink 의 Weaving 이나 OpenJPA 의 enhancement process 의 목적 중 하나는 `지연 로딩의 최적화`이다.
      - Hibernate 는 `hibernate.default_batch_fetch_size` 를 사용하여 지연 로딩을 최적화한다.
      - [참고. JPA performance optimization](https://dzone.com/articles/jpa-performance-optimization)

> 더 자세한 내용이 궁금하다면 [토비의 스프링3 2편, 5장. AOP 와 LTW](http://www.yes24.com/Product/Goods/4020006) 를 참고하자.

## References 

- https://github.com/spring-projects/spring-boot/issues/739
- https://martinfowler.com/eaaCatalog/
- https://groups.google.com/g/ksug/c/s7h0ONB8tcI
- https://docs.spring.io/spring-framework/docs/3.2.x/spring-framework-reference/html/aop.html
- [인프런. 스프링 핵심원리 고급](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B3%A0%EA%B8%89%ED%8E%B8/dashboard)
- [토비의 스프링 3](http://www.yes24.com/Product/Goods/4020006)
