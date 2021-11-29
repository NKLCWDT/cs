# Cookie, Session, JWT



## Cookie

### 쿠키 등장 배경

기본적으로 HTTP 프로토콜 환경은 `Connectionless`, `Stateless`한 특성을 가지기 때문에 서버는 클라이언트가 누구인지 매번 확인해야 하는데 이 특성을 보완하기 위해서 쿠키가 등장하게 되었다.

`Stateless` 상태의 프로토콜에서 상태기반 정보를 기억할 수 있는 쿠키가 개발되고 서버는 쿠키를 이용하여 사용자를 인증 할 수 있게 되었다.



### 쿠키란

쿠키는 서버가 사용자의 웹 브라우저에 전송하는 작은 데이터 조각이다. 브라우저는 그 데이터 조각들을 저장해 놓았다가, 동일한 서버에 재 요청 시 저장된 데이터를 함께 전송한다.

쿠키는 주로 세 가지 목적을 위해 사용된다.

1. 세션관리

서버에 저장해야 할 로그인, 장바구니, 게임 스코어 등의 정보 관리

2. 개인화

사용자 선호, 테마 등의 세팅

3. 트래킹

사용자 행동을 기록하고 분석하는 용도



>  과거엔 클라이언트 측에 정보를 저장할 때 쿠키를 주로 사용했다. 쿠키를 사용하는 게 데이터를 클라이언트 축에 저장할 수 있는 유일한 방법이었을 때는 이 방법이 타당했지만, 지금은 웹 스토리지 API (localStorage와 sessionStorage) 를 사용하면 된다.



### Set-Cookie 그리고 Cookie 헤더

HTTP 요청을 수신할 때, 서버는 응답과 함께 `Set-Cookie` 헤더를 전송할 수 있다.

이 서버 헤더는 클라이언트에게 쿠키를 저장하라고 전달한다.

```
TTP/1.0 200 OK
Content-type: text/html
Set-Cookie: yummy_cookie=choco
Set-Cookie: tasty_cookie=strawberry
```

이제, 서버로 전송되는 모든 요청과 함께, 브라우저는 Cookie 헤더를 사용하여 서버로 이전에 저장했던 모든 쿠키들을 회신 한다.

```
GET /sample_page.html HTTP/1.1
Host: www.example.org
Cookie: yummy_cookie=choco; tasty_cookie=strawberry
```
### 쿠키 속성

![img](https://postfiles.pstatic.net/MjAxODA3MzBfMzAw/MDAxNTMyOTM4MTk3ODkx.rSB1zUbmvPVKghA5J50pf4zy5IL7pWABmV7YMxY3l6Ig.RuoKDsVXHpa-SWvvwjLeaMI4KGvyE6V_FkMQmTh8ivcg.PNG.rudwls008/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7_2018-07-30_%EC%98%A4%ED%9B%84_5.09.42.png?type=w773)

크롬 개발자 도구에서 쿠키를 볼 수 있고 쿠키는 Name = Value 쌍으로 이루어져있고 여러가지 속성에 대해서 살펴 보겠다.



- 쿠키의 스코프

`Domain` 그리고 `Path`는 어떤 URL을 쿠키가 보내야 하는지 정의한다.

`Domain`은 쿠키가 전송되게 될 호스트들을 명시한다. 만약 명시되지 않는다면, 현재 문서 위치의 호스트 일부를 기본값으로 한다.

`Path`는 Cookie 헤더를 전송하기 위하여 요청되는 URL 내에 반드시 존재하야 하는 URL 경로이다.



- Secure과 HttpOnly 쿠키

`Secure` 쿠키는 HTTPS 프로토콜 상에서 암호화된 요청일 경우에만 전송된다.

`HttpOnly` 쿠키는 XSS 공격을 방지하기 위해 JavaScript의 Document.cookie API에 접근할 수 없다.

예를 들어, 서버쪽에서 지속되고 있는 세션의 쿠키는 JavaScript를 사용할 필요성이 없기 때문에 `HttpOnly` 플래그가 설정될 것이다.



> XSS (Cross-Site Scripting)
>
> 웹 사이트 관리자가 아닌 이가 웹 페이지에 악성 스크립트를 삽입할 수 있는 취약점 이다.



- SameSite 쿠키

`SameSite` 쿠키는 쿠키가 cross-site 요청과 함께 전송되지 않았음을 요구하게 만들어, CSRF 위조 공격에 대해 어떤 보호 방법을 제공한다.



> CSRF (Cross-Site Request Forgery)
>
> 사용자가 자신의 의지와는 무관하게 공격자가 의도한 행위를 특정 웹사이트에 요청하게 하는 공격을 말한다.
>
> XSS를 이용한 공격이 사용자가 특정 웹사이트를 신용하는 점을 노린 것이라면 (공격대상 Client),
>
> CSRF는 특정 웹사이트가 사용자의 웹 브라우저를 신용하는 상태를 노린 것이다 (공격대상 Server).



### 쿠키 종류

| 쿠키이름           | 특징                                                         |
| ------------------ | ------------------------------------------------------------ |
| Session Cookie     | 보통 만료시간(Expire date) 설정하고 메모리에만 저장되며 브라우저 종료시 쿠키를 삭제. |
| Persistent Cookie  | 장기간 유지되는 쿠키(예를 들어 Max-Age 1년), 파일로 저장되어 브라우저 종료와 관계없이 사용. |
| Secure Cookie      | HTTPS에서만 사용, 쿠키 정보가 암호화 되어 전송.              |
| Third-Party Cookie | 방문한 도메인과 다른 도메인의 쿠키, 보통 광고 베너 등을 관리할 때 유입 경로를 추적하기 위해 사용. |

Set-Cookie를 할때 Expires 속성이 존재하지 않으면 Session cookie 간주하고 Expires 속성이 존재한다면 기간동안은 브라우저를 닫아도 사라지지 않는 persistent cookie로 간주한다.



### 쿠키 단점

  - 보안에 취약하다 : 요청 시 쿠키 값을 그대로 보낸다.
  - 작은 허용 용량 : 4096bytes 이하
  - 웹 브라우저 마다 지원 형태가 다르다.
  - 한번에 하나의 정보만 저장 할 수 있다.
  - 문자열 그대로 통신하여 위변조가 가능하다.



## Session

세션이란 일정시간 동안 같은 사용자로 부터 들어오는 일련의 요구를 하나의 상태로 보고 그 상태를 일정하게 유지시키는 기술이다.

세션은 중요한 정보를 클라이언트에 저장하는 쿠키와는 다르게 서버에 저장하여 관리하기 때문에 사용자 정보가 노출되지 않는다.

서버에서는 클라이언트를 구별하기 위해 각각의 세션ID를 클라이언트마다 부여하게 되며 클라이언트가 종료되기 전까지 유지한다.



## 서버(세션) 기반 인증

이전에 쿠키만을 사용하여 인증을 했을 경우에는 클라이언트에서 사용자 정보를 관리하여 보안에 매우 취약하다는 단점이 있어  요즘에는 로그인과 같이 보안상 중요한 작업을 할 때는 세션을 사용한다.

예를들어 사용자가 로그인을 하면, 세션에 사용자 정보를 저장해두고 서비스를 제공할 때 사용하곤 한다. 이러한 서버 기반의 시스템은 다음과 같은 흐름을 갖는다.

![img](https://blog.kakaocdn.net/dn/be5HFu/btqAsR8iEdh/rk9Xno6XlQAwbTWFiGIXIk/img.png)

1. 사용자가 아이디와 비밀번호로 로그인을 한다.
2. 서버 측에서는 해당 정보를 검증한다.
3. 정보가 정확하다면 서버측에서 Set-Cookie를 통해 새로 발행한 SessionId를 보낸다.
4. 클라이언트 요청 시 마다 서버에 저장된 세션Id와 클라이언트에 있는 sessionId가 일치 한지 확인한다.



이러한 인증 방식은 소규모 시스템에서는 아직 많이 사용되고 있지만, 웹/앱 어플리케이션이 발달하게 되면서 서버를 확장하기가 어렵다는 등 다음과 같은 문제점을 보이기 시작했다.



### 서버 기반 인증의 문제점

1. 서버 부하

서버에서 클라이언트의 상태를 모두 유지하고 있어야 하므로, 클라이언트 수에 따른 메모리나 디스크 또는 DB에 부하가 심하다.

2. 확장성

사용자가 늘어나게 되면 더 많은 트래픽을 처리하기 위해 서버를 확장해야 하는데 서버를 확장할때 세션이 저장되어 있는 서버로만 요청이 가도록 분산처리를 해줘야 하기 때문에 서버를 확장하기 어렵다.

3. CORS

웹 브라우저에서 세션 관리에 사용하는 쿠키는 단일 도메인 및 서브 도메인에서만 작동하도록 설계되어 CORS 방식(여러 도메인에 request를 보내는 브라우저)을 사용할 때 쿠키 및 세션 관리가 어렵다.



## 토큰 기반 인증

토큰 기반 인증 시스템은 인증받은 사용자들에게 토큰을 발급하고, 서버에 요청을 할 때 헤더에 토큰을 함께 보내도록 하여 유효성 검사를 한다. 이러한 시스템에서는 더이상 사용자의 인증 정보를 서버나 세션에 유지하지 않고 클라이언트 측에서 들어오는 요청만으로 작업을 처리한다.

즉, 서버 기반의 인증 시스템과 달리 상태를 유지하지 않으므로 stateless한 구조를 갖는다.

![img](https://blog.kakaocdn.net/dn/ogoAg/btqAriyT5sY/YYt2wkEz50kKN47mLwRDXK/img.png)

1. 사용자가 아이디와 비밀번호로 로그인을 한다.
2. 서버 측에서는 해당 정보를 검증한다.
3. 정보가 정확하다면 서버측에서 사용자에게 토큰을 발급한다.
4. 클라이언트 측에서 전달받은 토큰을 저장해두고, 서버에 요청을 할 때마다 해당 토큰을 서버에 함께 전달한다.
5. 서버는 토큰을 검증하고, 요청에 응답한다.



### 토큰 기반 인증 시스템의 이점

1. 무상태성 & 확장성

토큰은 클라이언트 측에 저장되기 때문에 서버는 완전히 Stateless하며, 클라이언트와 서버의 연결고리가 없기 때문에 확장하기에 매우 적합하다.

2. 여러 플랫폼 및 도메인

서버 기반 인증 시스템의 문제점 중 하나인 CORS를 해결할 수 있다. 토큰을 사용한다면 어떤 디바이스, 어떤 도메인에서도 토큰의 유효성 검사를 진행한 후에 요청을 처리할 수 있다



최근에는 Json 포맷을 이용하는 JWT(Json Web Token)을 주로 사용한다.



## JWT

JWT(JSON Web Token)란 인증에 필요한 정보들을 암호화 시킨 토큰을 의미한다.

### JWT 구조

![jwt](https://tecoble.techcourse.co.kr/static/50692d28ac56af0c30039d81eaebeb01/244f0/2021-05-22-jwt-total.png)

JWT는 .을 구분자로 나누어지는 세 가지 문자열의 조합이다.

- Header

![jwt-header](https://tecoble.techcourse.co.kr/static/e794bd3bc3eca832350bd730318c5ebb/f7886/2021-05-22-jwt-header.png)

alg과 typ는 각각 정보를 암호화할 해싱 알고리즘 및 토큰의 타입을 지정한다.

- Payload

![jwt-payload](https://tecoble.techcourse.co.kr/static/52fc7e21e5ba27aa1634ca26836785ce/1e7a9/2021-05-22-jwt-payload.png)

Payload는 토큰에 담을 정보를 지니고 있다. Key-value 형식으로 이루어진 한 쌍의 정보를 Claim이라고 칭한다.

- Signature

![jwt-signature](https://tecoble.techcourse.co.kr/static/445b9916a7dd4f17b9682ef353457484/c4bf7/2021-05-22-jwt-signature.png)

Signature는 인코딩된 Header와 Payload를 더한 뒤 비밀키로 해싱하여 생성한다.

Header와 Payload는 단순히 인코딩된 값이기 때문에 제 3자가 복호화 및 조작할 수 있지만, Signature는 서버 측에서 관리하는 비밀키가 유출되지 않는 이상 복호화 할 수 없다.

따라서 Signature는 토큰의 위변조 여부를 확인하는데 사용한다.



### 장점

1. Header와 Payload를 가지고 Signature를 생성하므로 데이터 위변조를 막을 수 있다.
2. 인증 정보에 대한 별도의 저장소가 필요없다.
3. JWT는 토큰에 대한 기본정보와 전달할 정보 및 토큰이 검증됬음을 증명하는 서명 등 필요한 모든 정보를 자체적으로 지니고 있다.



### 단점

1. 쿠키/세션과 다르게 JWT는 토큰의 길이가 길어, 인증 요청이 많아질수록 네트워크 부하가 심해진다.
2. Payload 자체는 암호화 되지 않기 때문에 유저의 중요한 정보는 담을 수 없다.
3. 토큰은 한번 발급 되면 유효기간이 만료될 때 계속 사용되어 탈취 당하게 되면 대처하기 힘들다.



### 보안전략

1. 짧은 만료기한 설정

토큰의 만료 시간을 짧게 설정하여 토큰이 탈취되더라도 빠르게 만료되기 때문에 피해를 최소화 할 수 있다. 그러나 사용자가 자주 로그인 해야 하는 불편함이 수반된다.

2. Refresh Token

클라이언트가 로그인 요청을 보내면 서버는 Access Token 및 그보다 긴 만료 기간을 가진 Refresh Token을 발급하는 전략이다.

Access Token이 만료되었을 때 Refresh Token을 사용하여 Access Token의 재발급을 요청한다. 서버는 DB에 저장된 Refresh Token과 비교하여 유효한 경우 새로운 Access Token을 발급하고, 만료된 경우 사용자에게 로그인을 요구한다.

해당 전략을 사용하면 AccessToken의 만료 기한을 짧게 설정할 수 있으며, 사용자가 자주 로그인 할 필요도 없고 서버가 강제로 RefreshToken을 만료시킬 수 있다.

그러나 검증을 위해 서버는 RefreshToken을 별도로 저장하고 있어야 하므로 이는 JWT에 장점을 완벽하게 누릴 수는 없다.

---

### Reference

https://tecoble.techcourse.co.kr/post/2021-05-22-cookie-session-jwt/

https://velog.io/@stampid/%EC%BF%A0%ED%82%A4-%EC%84%B8%EC%85%98-%EA%B7%B8%EB%A6%AC%EA%B3%A0-JWT

https://nesoy.github.io/articles/2017-03/Session-Cookie

https://dololak.tistory.com/535

https://velog.io/@dogy/%EC%84%9C%EB%B2%84-%EC%9D%B8%EC%A6%9D.-%EC%BF%A0%ED%82%A4Cookie-%EC%84%B8%EC%85%98Session-%ED%86%A0%ED%81%B0JWT

https://mangkyu.tistory.com/55

https://developer.mozilla.org/ko/docs/Web/HTTP/Cookies
