## Spring Security

스프링 시큐리티는 스프링 기반의 애플리케이션의 보안(인증과 권한, 인가 등)을 담당하는 스프링 하위 프레임워크이다.

인증(Authenticate)과 인가(Authorize)를 담당하는 프레임워크이다.

스프링 시큐리티에서는 주로 서블릿 필터(filter)와 이들로 구성된 필터체인으로의 구성된 위임모델을 사용한다. 그리고 보안과 관련해서 체계적으로 많은 옵션을 제공해주기 때문에 개발자 입장에서는 일일이 보안관련 로직을 작성하지 않아도 된다는 장점이 있다.

### 기본용어

- 접근 주체(Principal) : 보호된 리소스에 접근하는 대상
- 인증(Authentication) : 해당 사용자가 본인이 맞는지를 확인하는 절차
- 인가(Authorize) : 인증된 사용자가 요청한 자원에 접근 가능한지를 결정하는 절차
- 비밀번호(Credential) : 리소스에 접근하는 대상의 비밀번호
- 권한 : 어떠한 리소스에 대한 접근 제한, 모든 리소스는 접근 제어 권한이 걸려있다. 인가 과정에서 해당 리소스에 대한 제한된 최소한의 권한을 가졌는지 확인한다.

### 스프링 시큐리티 특징

- 보안과 관련하여 체계적으로 많은 옵션을 제공하여 편리하게 사용할 수 있다.
- Filter 기반으로 동작하여 MVC와 분리하여 관리 및 동작한다.
- 어노테이션을 통한 간단한 설정
- Spring Security는 기본적으로 세션 & 쿠키 방식으로 인증

스프링 시큐리티를 설정하게 된다면 DispatcherServlet에 도달하기 전에 서블릿 Filter 구현체에게 걸리게 된다.

스프링 시큐리티는 FilterChain들을 Servlet Container 기반의 필터 위에서 동작하기 위해 중간 연결을 위한 DelegatingFilterProxy를 사용한다.

따라서 DelegatingFilterProxy는 IOC 컨테이너에서 관리하는 빈이 아닌 표준 서블릿 필터를 구현하고 있으며 내부적으로는 요청을 위임할 FilterChainProxy를 가지고 있다.

<img width="887" alt="image" src="https://user-images.githubusercontent.com/70622731/160224657-f7ac1d54-1e5d-4ff4-999b-fb6d21f99f82.png">

### DelegatingFilterProxy

DelegatingFilterProxy는 Servlet Container 기반의 필터 위에서 동작하기 위해서 중간 역할만 하고 FilterChainProxy에게 요청을 위임한다.

그렇다면 FilterChainProxy는 무엇을 할까?

FilterChainProxy 역시 처리를 위임하기 위한 SecurityFilterChain을 들고 있다.

<img width="924" alt="image" src="https://user-images.githubusercontent.com/70622731/160224712-3dee81c7-fe04-4ef7-8479-303879521d51.png">

### 스프링 시큐리티 동작방식

![image](https://user-images.githubusercontent.com/70622731/160224438-9a564959-0b9b-4154-bddc-3ecec431ba9d.png)

__1. HTTP 요청 수신(Http Request) 및 AuthenticationFilter 통과__

Spring Security는 필터들을 가지고 있다.

요청(request)은, 인증(Authentication)과 권한부여(Authoization)를 위해 이 필터들을 통과하게 된다.

이 필터를 통과하는 과정은, 해당 요청과 관련된 인증 필터를 찾을 때 까지 지속된다.

예)

HTTP Basic 인증 요청은 `BasicAuthenticationFilter`에 도달할 때까지 필터 체인을 통과한다.

HTTP Digest 인증 요청은 `DigestAuthenticationFilter`에 도달할 때까지 필터 체인을 통과한다.

로그인 form submit 요청은 `UsernamePasswordAuthenticationFilter`에 도달할 때까지 필터 체인을 통과한다.

> OAuth2.0 인증이나 JWT를 이용한 인증요청은 `OAuth2ClientAuthenticationProcessingFilter`에 도달할 때까지 필터 체인을 통과한다.

이 필터 체인 중 인증을 담당하는 필터를 `AuthenticationFilter`라고 한다.

AuthenticationFilter는 사용자의 세션ID (JSESSIONID)가 Security Context에 있는지 확인한다. (Security Context란 아래의 모든 로직을 통과한 인증된 사용자의 정보(인증객체)를 저장하는 공간이다.)

Security Context에 세션ID가 없다면 아래 로직을 수행한다.

__2. 사용자 자격 증명을 기반으로 AuthenticationToken 생성__

인증 요청(request)이 관련 AuthenticationFilter에 의해 수신되면 수신된 요청에서 사용자 이름과 비밀번호를 추출한다.

이 추출된 사용자 자격 증명(credentials)을 기반으로 인증개체를 만들게 되는데, 이를 `UsernamePasswordAuthenticationToken`이라고 한다.

__3. AuthenticationManager를 위해 생성된 AuthenticationToken 위임__

만들어진 UsernamePasswordAuthenticationToken는 AuthenticaitonManager의 인증 메서드를 호출하는데 사용된다.

여기서 AuthenticationManager는 단순한 인터페이스이며 실제 구현은 ProviderManager이다.

ProviderManager에는 사용자 요청을 인증에 필요한 AuthenticationProvider 목록이 있다. ProviderManager는 제공된 각 AuthenticationProvider를 살펴보고 전달된 인증 개체(UsernamePasswordAuthenticationToken)를 기반으로 사용자 인증을 시도한다.

__4. AuthenticationProvider 목록으로 인증시도__

AuthenticationProvider는 제공된 인증 개체로 사용자를 인증한다.

__5. UserDetailsService / 6. UserDetails / 7. User__

일부 AuthenticationProvider는 사용자 이름(username)을 기반으로 사용자 세부 정보를 검색하기 위해 UserDetailService를 사용할 수 있다.

UserDetailsService는 DB에 저장된 회원의 비밀번호와 비교해 일치하면 UserDetails 인터페이스를 구현한 객체를 반환한다.

UserDetailsService는 Spring Security의 인터페이스이며 이를 UserDetailsService를 구현한 서비스는 직접 개발해야한다.

```java
public interface UserDetailService {
    UserDetail loadUserByUsername(String username) throws UsernameNotFoundException;
}
```

즉, 위 코드의 `loadUserByUsername` 메소드를 오버라이딩해 DB와 비교하는 로직을 직접 짜야한다.

__8. AuthenticationException__

AuthenticationProvider 인터페이스에 의해 사용자가 성공적으로 인증되면, 완전히 채워진 인증객체가 반환된다.

인증에 실패하면 AuthenticationException이 발생한다.

AuthenticationException이 발생하면 인증 매커니즘을 지원하는 AuthenticationEntryPoint에 의해 처리된다.

__9. 인증완료__

AuthenticationManager는 획득한 완전히 채워진 인증객체를 관련 인증 필터(AuthenticationFilter)로 다시 반환한다.

__10. SecurityContext에서 인증 개체 설정__

관련 AuthenticationFilter는 향후 필터 사용을 위해 획득한 인증 개체를 SecurityContext에 저장한다.

### Spring Security Filter

<img width="903" alt="image" src="https://user-images.githubusercontent.com/70622731/159123381-cd133899-4726-4afa-8f81-f5cb95c6ea51.png">

- 인증관리자(Authentication Manager)와 접근 결정 관리자(Access Decision Manager)를 통해 사용자의 리소스 접근을 관리
- 인증관리자는 UsernamePasswordAuthenticationFilter, 접근결정관리자는 FilterSecurityInterceptor가 수행한다.

![image](https://user-images.githubusercontent.com/70622731/159123947-999d96b1-f8b0-4ca8-b1a7-ad5b060f1578.png)

|필터명 | 설명 |
| --- | --- |
| SecurityContextPersistenceFilter | SecurityContextRepository에서 SecurityContext를 로드하고 저장하는 일을 담당함 |
| LogoutFilter | 로그아웃 URL로 지정된 가상URL에 대한 요청을 감시하고 매칭되는 요청이 있으면 사용자를 로그아웃 시킨다 |
| UsernamePasswordAuthenticationFilter | 사용자명과 비밀번호로 이뤄진 폼기반 인증에 사용하는 가상의 URL요청을 감시하고 요청이 있으면 사용자의 인증을 진행함 |
| DefaultLoginPageGenerationFilter | 폼기반 또는 OpenID 기반 인증에 사용하는 가상URL에 대한 요청을 감시하고 로그인 폼 기능을 수행하는데 필요한 HTML을 생성함 |
| BasicAuthenticationFilter | HTTP 기반 인증 헤더를 감시하고 이를 처리함 |
| RequestCacheAwareFilter | 로그인 성공 이후 인증 요청에 의해 가로채어진 사용자의 원래 요청을 재구성하는데 사용됨 SecurityContextHolderAwareRequestFilter HttpServletRequest를 HttpServletRequestWrapper를 상속하는 하위 클래스(SecurityContextHolderAwareRequestWrapper)로 감싸서 필터 체인상 하단에 위치한 요청 프로세서에 추가 컨텍스트를 제공함|
| AnonymousAuthenticationFilter | 이 필터가 호출되는 시점까지 사용자가 아직 인증을 받지 못했다면 요청 관련 인증 토큰에서 사용자가 익명 사용자로 나타나게 됨 |
| SessionManagementFilter | 인증된 주체를 바탕으로 세션 트래킹으 처리해 단일 주체와 관련한 모든세션들이 트래킹되도록 도움 | 
| ExceptionTranslationFilter | 보호된 요청을 처리하는 동안 발생할 수 있는 기대한 예외의 기본 라우팅과 위임을 처리함 |
| FilterSecurityInterceptor | 권한부여와 관련한 결정을 AccessDecisionManager에게 위임해 권한부여 결정 및 접근 제어결정을 쉽게 만들어줌 |

---


### Reference

- https://devuna.tistory.com/55
- https://k3068.tistory.com/88
- https://doozi0316.tistory.com/entry/Spring-Security-Spring-Security%EC%9D%98-%EA%B0%9C%EB%85%90%EA%B3%BC-%EB%8F%99%EC%9E%91-%EA%B3%BC%EC%A0%95
- https://wildeveloperetrain.tistory.com/50
