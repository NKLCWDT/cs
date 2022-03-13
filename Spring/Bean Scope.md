## 빈 스코프란 ?

- 말그대로 빈이 사용되어지는 범위이다.
- 스프링이 관리하는 빈이 생성되고, 존재하고, 적용되는 범위를 말한다.
- 기본은 싱글톤이다.
- 요청마다 빈이 하나씩 생성되면, 서버는 부하에 걸리기 쉽기 때문에 스프링에서는 기본적으로 싱글톤을 채택하였다.

## 빈 스코프의 종류

- singleton : 기본값, 스프링 IoC 컨테이너당 하나의 인스턴스만 사용한다. 앱이 구동되는 동안 하나만 사용된다는 뜻이다. **스프링의 빈은 기본으로 싱클톤 스코프로 생성된다.**
- prototype : 매번 새로운 빈을 정의해서 사용한다.
- request : HTTP 요청 하나가 들어오고 나갈 때 까지 유지되는 스코프, 각각의 HTTP 요청마다 별도의 빈 인스턴스가 생성되고, 관리된다.
- session : HTTP Session과 동일한 생명주기를 가지는 스코프
- application : ServletContext 와 동일한 생명주기를 가지는 스코프
- websocket : 웹 소켓과 동일한 생명주기를 가지는 스코프

## 사용방법(빈 스코프를 등록하는 방법)

```java
@Component
@Scope(value = "prototype") // 이부분 처럼, 컴포넌트로 등록 하면서 어노테이션 붙여주면 됨.
public class ProtoType {
}
```

- @Scope(value = "prototype")
- @Scope(value = "singleton")
- @Scope(value = "request")
- @Scope(value = "session")
- @Scope(value = "application")
- @Scope(value = "websocket")

## 프로토타입 빈을 사용할 때 주의 사항

- 싱글톤 스코프의 빈이 prototype 의 빈을 주입받는 경우, prototype 의 경우에는 항상 새로운 빈이 생성되어야 하는데, 매번 같은 빈을 사용하게 된다

#### Single.java

```java
@Component
public class Single {

    @Autowired
    ProtoType protoType; // 이 프로토타입 객체의 경우, 항상 같은 빈이 주입된다.

    public ProtoType getProtoType() {
    }
}
```

#### Prototype.java

```java
@Component
@Scope(value = "prototype")
public class ProtoType {

}
```

- 프로토 타입이지만, 매번 같은 빈을 사용하는 이유
    - **singleton빈**은 **ApplicationContext**가 처음 앱을 구동할때 빈을 만들고 빈을 주입해서 앱이 종료될 때 까지 계속 사용 되기 때문에 **singleton 빈**안에 있는 **prototype빈**도 처음 주입된 채로 그대로 사용된다.

#### 해결 방법

- 프로토타입으로 쓸 빈이 계속 새로운 빈으로 생성되게 하려면 어떻게 해야할까?
- proxy mode 를 이용하자.
- 클래스라면,

```java
@Component
@Scope(value = "prototype", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class ProtoType {
}
```

- 만약 인터페이스라면,

```java
@Component
@Scope(value = "prototype", proxyMode = ScopedProxyMode.INTERFACES)
public interface ProtoType {
}
```

- scopedProxyMode.DEFAULT : Proxy를 사용하지 않음
- **프로토타입 모드에서의 프록시의 원리**
    - ApplicationContext 가 빈을 처음에 생성할 때, 프로토타입의 빈을 그대로 등록하는 것이 아니다.
    - proxy클래스를 만드는데, 이 때 proxy 클래스는 프로토타입의 클래스(또는 인터페이스)를 상속받아서 만든다. 즉, proxy 클래스와 프로토타입의 클래스는 타입이 같다.
    - 이 proxy 클래스를 빈으로 등록하고, proxy 클래스에서 내부적으로 매번 새로운 프로토타입의 빈을 생성해서 사용하도록 되어있다.

<br>
<br>
<br>

## request 스코프

- 여러 고객들의 요청이 뒤섞이면서 로그가 뒤죽박죽으로 남는 경우를 방지해줄 수 있다.
- 로그를 각 클라이언트 별로 구분될 수 있도록 하는 것이다.
- 예시

```java
[clientA...][http://localhost:8080/login] controller test 
[clientB...][http://localhost:8080/login] controller test 
[clientA...][http://localhost:8080/login] service id = testId 
[clientB...][http://localhost:8080/login] service id = testId
```

- 각 HTTP 요청마다 스코프들이 관리된다.

![IMAGES](../images/Bean Scope1.png)


- Request 스코프를 만들어보는 실습은 아래 링크를 통해 참고하여 진행할 수 있다.
    - 링크 : [https://chung-develop.tistory.com/64](https://chung-develop.tistory.com/64)


<br>
<br>
<br>

## Session 스코프

- Session : 브라우저가 최초로 서버에 요청을 하게 되면 브라우저당 하나씩 메모리 공간을 서버에 할당하게 된다.
    - 이 영역은 브라우저 당 하나씩 지정되며, 새로운 요청이 발생해도 같은 공간을 사용하게 된다. 이런 메모리 공간을 session이라고 한다.
    - 브라우저를 종료할 때까지 서버에서 사용할 수 있다.
- Session 스코프 : 브라우저가 최초의 요청을 발생시키고, 브라우저를 닫을 때까지를 session 스코프라고 부른다.

#### 예시

- build.gradle 에 추가

```java
dependencies {
	implementation 'org.springframework.session:spring-session-core'
}
```

#### UserInfo.java

```java
@Getter
@Setter
@Component
@Scope(value = WebApplicationContext.SCOPE_SESSION, proxyMode = ScopedProxyMode.TARGET_CLASS)
@ToString
public class UserInfo implements Serializable {
	
	private static final long serialVersionUID = 1L;
	
	private Long userId;
	private String userNm;
	
}
```

- **proxyMode=TARGET_CLASS** : Bean Scope를 session으로 설정할 경우에는 proxyMode를 TARGE_CLASS로 설정해주어야 한다.
- **Serializable** : 객체를 직렬화(Serializable)를 하여 외부 Redis와 같은 데이터베이스에 세션을 저장할 수 있도록 한다. 외부에 저장하지 않고 애플리케이션 서버 내에 메모리에서 해결할 계획이라면 굳이 직렬화하지 않아도 된다.

#### UserController.java

```java
@RestController
@RequestMapping("user")
public class UserController {

	@Resource
	private UserInfo userInfo;
	
	@Autowired
	private UsersRepo usersRepo;

	@PostMapping("signin")
	public UsersDoc signin(@RequestBody UsersDoc reqBody) {
		String userId = reqBody.getUserId();
		String passwd = reqBody.getPasswd();
		UsersDoc doc = usersRepo.findByUserId(userId)
				.filter((data) -> data.getPasswd().equals(passwd))
				.orElseThrow(() -> new RestException(HttpStatus.NOT_FOUND, "Check your infomation."));
		userInfo.setUserId(doc.getUserId());
		userInfo.setUserNm(doc.getUserNm());
		return doc;
	}
	
	@GetMapping("session")
	public String get() {
		return userInfo.toString();
	}

}
```

- 설명
    - 서버를 실행하고 `/user/signin`을 호출해보면,
    - 그럼 서버에서 JSESSIONID를 발급해주어 쿠키에 들어가있는 것을 확인할 수 있다. 클라이언트에서 서버로 요청할 때, 이 발급받은 JSESSIONID를 보낼것이다.
    - `/user/session` 을 호출해보면 userInfo 객체에 저장한 데이터를 확인할 수 있다.
    - 다른 브라우저를 통해서, `/user/signin` 호출하지 않고, `/user/session` 을 호출하게 되면 값이 null 일 것이다.

#### application.yml

- 서버에서 발급해준 JSESSIONID가 몇 초 동안 유효한지 설정할 수도 있다.
- 아래처럼 30초로 지정하면 30초 뒤에 JSESSIONID가 만료되어 삭제되고 다시 Session에 데이터를 재입력(재로그인 , 즉, `/user/signin 요청`) 해주어야 한다.

```java
server:
  servlet:
    session:
      timeout: 30s
```

- 대부분의 경우에는 세션은 Redis, MongoDB, MySQL 등 데이터베이스에 저장이 된다. 외부와 연동하는 방법은 필요할 때 알아보면서 적용해보면 될 듯 하다.


<br>


## Reference

- [https://velog.io/@probsno/Bean-스코프란](https://velog.io/@probsno/Bean-%EC%8A%A4%EC%BD%94%ED%94%84%EB%9E%80)
- [https://gofnrk.tistory.com/42](https://gofnrk.tistory.com/42)
- [도서] 토비의 스프링 3.1 스프링의 이해와 원리