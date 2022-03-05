## 스프링 vs 스프링부트

### 스프링

스프링 프레임워크(Spring Framework)는 자바 플랫폼을 위한 오픈소스 애플리케이션 프레임워크로서 간단히 스프링(Spring)이라고도 불린다. 동적인 웹 사이트를 개발하기 위한 여러가지 서비스를 제공하고 있다.

### 스프링부트

스프링 프레임워크는 기능이 많은만큼 환경설정이 복잡한 편이다. 이것을 좀 더 편하게 하기위해 나온 것이 바로 스프링부트이다.

스프링과 스프링부트 차이점에 대해 자세히 알아보겠다.

### 스프링부트는 스프링에 어떤점이 개선된걸까?

1. 의존성관리
2. 자동설정
3. 내장 WAS

### 1. 의존성 관리

기존 Spring은 개발에 필요한 모듈의 의존성을 각각 다운받아줘야 했으며, 각 모듈의 버전을 개발자가 하나하나 명시해줘야했다.

SpringBoot는 `spring-boot-starter`를 통해 의존성 관리를 더 편리하게 해준다.

spring-boot-starter는 프로젝트에 설정해야 할 다수의 의존성들을 starter가 이미 포함하고 있기 때문에 우리는 starter에 대한 의존성 추가 만으로도 프로젝트를 시작하거나 새로운 기능을 추가할 수 있다.

예시로 spring-boot-starter-jpa 의존성을 추가했을 때 아래와 같은 일을 해준다.

- spring-aop, spring-jdbc등의 의존성을 걸어준다.
- classpath를 뒤져서 어떤 database를 사용하는지 파악하고, 자동으로 entityManager를 구성해준다.
- 해당 모듈들 설정에 필요한 properties 설정을 제공한다.


<img width="729" alt="image" src="https://user-images.githubusercontent.com/70622731/156871620-45bbc495-d399-4453-b90a-79377fec524e.png">

spring-boot-starter-jpa 안으로 들어가 보면 이미 여러 의존성들이 정의되어 있다.

<img width="689" alt="image" src="https://user-images.githubusercontent.com/70622731/156871566-091dd031-b332-4f74-a390-3c82d3591256.png">

maven인 경우 의존성들을 부모 자식관계로 관리하기 때문에 Spring-boot-stater-parent를 통해 각 모듈의 현재 스프링부트 버전에 가장 적합한 버전을 제공해준다.

<img width="751" alt="image" src="https://user-images.githubusercontent.com/70622731/156871732-abd5a8e2-b47e-4ebf-8bb4-bec3b6b177ef.png">

gradle인 경우는 io.spring.dependency-management 플러그인을 통해 각 모듈의 현재 스프링부트 버전에 가장 적합한 버전을 제공해준다.

<img width="834" alt="image" src="https://user-images.githubusercontent.com/70622731/156871454-ff5ffb7c-4761-4670-9173-2d4ca7e27aa0.png">

### 2. 자동설정

기존의 스프링은 환경설정이 복잡하고 많은 설정을 했어야 했는데(db연결할때 datasource 설정을 매번 직접했어야 했다.) 스프링부트에서는 이러한 환경설정을 @SpringBootApplication을 통해 자동설정을 해준다.

<img width="879" alt="image" src="https://user-images.githubusercontent.com/70622731/156871833-67330d59-6f1d-4a85-abea-1ba2aa662263.png">

@SpringBootApplication이 어떠한 역할을 하는지 알아보기 위해서 안으로 들어가보면

<img width="1193" alt="image" src="https://user-images.githubusercontent.com/70622731/156871862-9f1b9f09-1445-431d-a21c-a2c84f2f7461.png">

위 그림과 같이 여러 어노테이션이 붙어있다.

__@SpringBootConfiguration__

SpringBoot의 설정을 나타내는 어노테이션으로, Spring의 @Configuration을 대체한다.

__@EnableAutoConfiguration__

자동 설정의 핵심 어노테이션으로, 클래스 경로에 지정된 내용을 기반으로 설정 자동화를 수행한다.

__@ComponentScan__

basePackages 프로퍼티 값에 별도의 경로를 설정하지 않으면 해당 어노테이션이 위치한 패키지가 루트경로가 되어 빈으로 등록할 클래스들을 탐색한다.

### SpringBoot에 bean 등록과정

SpringBoot는 Bean을 두 번에 걸쳐 등록한다.

1. @ComponentScan

처음에는 Spring과 마찬가지로 Component-scan을 통해 component를 찾고 bean 생성을 진행한다. 그 과정에서 우리가 설정한 bean들이 생성된다.

2. EnableAutoConfiguration

그 후에는 @EnableAutoConfiguration으로 추가적인 Bean들을 읽어서 등록하게 된다. 메인 클래스(@SpringBootApplication)를 실행하면, @EnableAutoConfiguration에 의해 META-INF/spring.factories 안에 들어있는 수많은 자동 설정들이 조건에 따라 적용이 되어 수 많은 Bean들이 생성되고, 스프링 부트 어플리케이션이 실행되는것이다.

즉, SpringBoot는 Component scan을 통해서 모은 component들의 정보와 META-INF/spring.factories 파일에 사전 정의한 auto-configuration 내용에 의해 bean 생성을 진행한다.

<img width="845" alt="image" src="https://user-images.githubusercontent.com/70622731/156872544-8ffef322-9cc5-42e3-8fff-0b1db4586e1a.png">

### 3. 내장 WAS

기존 Spring을 이용한 프로젝트는 war(Web application ARchive)파일로 배포할 수 있었다. war파일은 웹 어플리케이션을 압축해 저장해놓은 파일로, tomcat과 같은 was에서 돌아갈 수 있는 구조를 담고 있었다.

따라서 Spring으로 개발한 프로젝트를 배포하기 위해서는 웹어플리케이션이 압축된 war파일과 프로그램을 실행시킬 was가 필요했다.

<img width="698" alt="image" src="https://user-images.githubusercontent.com/70622731/156872719-d24361b1-d159-4e30-a5d6-c6e7b7ce53b1.png">

war파일을 실행시킬 was를 직접 설정해주어야 한다.

하지만 SpringBoot는 Tomcat이나 Jetty같은 내장 서버를 가지고 있기 때문에 jar 파일로 배포할 수 있다. 따라서 SpringBoot로 개발한 프로젝트를 배포하기 위해서는 단순히 jar 파일만 필요하다.

즉, 스프링부트가 tomcat을 내장하면서, Servlet Container에 종속되던 WebApplication이 역으로 WebApplication에 Servlet Container이 종속할 수 있도록해주게 된다.

### 언제 무엇을 사용해야 할까?

<img width="851" alt="image" src="https://user-images.githubusercontent.com/70622731/156873087-f656cc32-a2ae-417c-b4fa-e7d3a660e92e.png">

2017년까지만 해도 SpringBoot보다 Spring MVC의 비중이 더 높았지만 시간이 흐를수록 SpringBoot 사용 비중이 커지고 있음을 확인할 수 있다.

SpringBoot에 비중도 많이 높아지고 있고 현재 SpringBoot이 많은 편리성을 제공해주기 때문에 새로운 프로젝트를 개발할 때 많은 사람들이 SpringBoot를 찾는 것 같다고 생각한다.

비교적 규모가 작은 어플리케이션을 실행시키기위해 그보다 큰 WAS를 따로 설치하기엔 효율적이지 않다 이런경우 SpringBoot를 쓰는게 적당하고 규모가 큰 웹사이트에 경우 이런 구조로 만드는 것보단 SpringMVC 형태로 만들어 WAS에 배포하는 스타일이 더 좋다.

하지만 프로젝트에 규모적로는 Spring과 SpringBoot를 고르는게 잘못됐다는 글들도 있어 정확히 어느것을 사용해야한다 이런 기준은 없는 것 같다.

[SpringBoot을 언제써야할까? (3)외장 WAS 가 반드시 필요한가?](https://zepinos.tistory.com/87)


---

### Reference

- [에어의 Spring vs Spring Boot](https://www.youtube.com/watch?v=Y11h-NUmNXI)
- https://ssoco.tistory.com/66
- https://velog.io/@courage331/Spring-%EA%B3%BC-Spring-Boot-%EC%B0%A8%EC%9D%B4
- https://lifeinprogram.tistory.com/7
- https://beam307.github.io/2018/03/21/springboot/


