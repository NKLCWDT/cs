## REST(Representational State Transfer)
> REST는 웹의 창시자(HTTP) 중의 한 사람인 Roy Fielding의 2000년 논문에 의해서 소개되었다.  웹의 장점을 최대한 활용할 수 있는 네트워크 기반의 아키텍쳐를 소개했는데 그것이 바로 REST 이다.


- 해석 : **자원**을 이름으로 구분하여 해당 자원의 **상태** 를 **주고, 받는** 모든 것

<br>


### 구성 요소
#### 아래 3가지 요소가 REST 의 핵심 3가지 요소!
`자원, 조작, 표현`

<br>


1) **자원** 이란 ?
- 서버에 있는 것
- DB 안에 들어가 있는 데이터 하나하나를 의미한다.
ex) 유저 , 주문 등
- 또는, 이미지 하나하나를 의미한다. `number_24` , `number_25` 
ex) https://www.123rf.com/stock-photo/number_24.html
- URI 를 통해 자원을 명시하고, 구분할 수 있다.

<br>

2) **조작** 이란?
- Client는 HTTP Method(POST, GET, DELETE, PUT)를 이용하여 지정한 자원에 대한 조작을 요청한다.

<br>


3) **표현** 이란?
- Client가 Server에게, `자원에 대한 조작을 요청`하면 Server는 이에 대한 적절한 응답(Representation)을 보낸다.
  
<br>


### 정리
> REST의 구체적인 개념은 **클라이언트와 서버**사이에서, HTTP **URL**을 통해 자원을 명시하고, 
HTTP **Method**(POST, GET, DELETE, PUT 등)를 통해 해당 자원에 대한 
**조작**을 요청하고, 이에 대한 **응답**을 받는 것을 의미한다.

<br>
<br>


## Rest의 기본 원칙 6가지
> Rest 기본 원칙 6 가지를 지키는 것을 Restful 하다고 표현

### 1. 클라이언트-서버 모델
- 서버와 클라이언트 역할이 분리되어 **독립적**으로 교체 및 개발 될 수 있다.
- 사용자 인터페이스와 데이터 처리 영역이 분리될 수 있어 유지 보수가 매우 쉬워진다
  <br>


### 2. 무상태
- 세션 상태가 서버에 저장되어 있지 않은 상태, 필요에 따라 외부 DB 에 저장하기도 한다.
- server는 단순히 요청이 오면 응답을 보내는 역할만 수행하며, 세션 관리는 client에게 책임이 있다.
  - 장점 : scaling 이 자유롭다.
  
  
  ![](https://images.velog.io/images/annmj/post/b70611e7-e3ff-43aa-9ab8-d2b78af1474b/image.png)
  
  
  ![](https://images.velog.io/images/annmj/post/3dcda5ee-014d-49aa-bdf7-36940be49d99/image.png)
  

<br>


### 3. 캐시 가능
- REST는 웹 표준인 HTTP를 그대로 사용하기 때문에, 웹에서 사용하는 기존 인프라를 그대로 사용할 수 있다. 따라서 HTTP가 가진 캐싱 기능을 적용할 수 있다.
- HTTP 프로토콜에서 사용하는 태그들을 활용하여 캐싱 처리를 할 수 있다.
  - 캐싱 처리는 GET 요청에서 주로 처리 된다. (post, put 으로도 가능은 하나, 잘 안쓴다.)
  - 관련 헤더들
    - If-Match 
      - client 가 함께 보낸 Etag 식별자와 같은 값이 있으면 200 
      - 다른 값이 있으면 412 http code 로 응답, 덮어쓰기 용도로 사용된다.  
    - If-Modified-Since 
      - Last-Modified 의 값을 표기하여 전달해주고, 동일한 컨텐츠라면 ( 최종 수정 시간이 같다면 ) 304 http code 로 응답 
      - 만약 다른 컨텐츠라면 (If-Modified-Since 에 들어간 last-modified 값보다 이후에 데이터가 수정되었다면) 200 으로 응답
    - If-**None**-match : ETag 식별자와 같은 값이라면 304를 전달하고, **다른 값**이라면 200을 응답한다. 신규 값을 생성하고, 덮어쓰기를 금지하는 용도로 사용된다.
  - 캐싱 관련 검증 헤더
    - ETag
    - last-modified
  <br>
  
  

  - 참고
    - 412 : Precondition Failed , 클라이언트 오류 응답 코드는 대상 리소스에 대한 액세스가 거부되었음을 나타냅니다.
    - ETag : ETag HTTP 응답 헤더는 특정 버전의 리소스를 식별하는 식별자입니다. (like 주민등록번호)


  <br>



### 4. 자체 표현 구조
- REST API만 보고도 이를 쉽게 이해할 수 있다. 
  <br>

### 5. Layerd System (계층화)
- 클라이언트 입장에서는 REST API 서버만 호출한다.
- 하지만, REST 서버는 다중 계층으로 구성될 수 있으며, 로드 밸런싱, 암호화 계층을 추가해 구조상의 유연성을 둘 수 있고 PROXY, 게이트웨이 같은 네트워크 기반의 중간매체를 사용할 수 있음
- 내부 레이어를 숨기고, 인접한 레이어에만 공개함으로써 레이어간의 결합을 줄인다.
  <br>

### 6. Uniform Interface (유니폼 인터페이스)
- HTTP 표준만 따른다면 어떤 언어 혹은 어떤 플랫폼에서 사용하여도 사용이 가능한 인터페이스 스타일이다.
- 안드로이드 플랫폼, IOS 플랫폼 등 특정 언어나 플랫폼에 종속되지 않고 사용이 가능하다.

<br>
<br>
<br>
<br>

## REST API 디자인 가이드
1) URI 의 리소스명은 소문자를 사용하고, 동사보다는 명사를 사용하자
```
GET    /members/delete/1
```
delete 와 같은 행위를 나타내는 표현이 들어가는 것을 지양해야 한다.
단수 명사보다는 복수 명사를 사용하자

2) 자원에 대한 행위는 HTTP 메서드를 통해서 표현하자
```
DELETE   /members/1
```
의미 : 회원들 중에서 id 1번을 가진 회원 삭제

3) 하이픈(-) 은 URI 의 가독성을 위해서 사용하자
밑줄(_) 은 사용하지 않는다.

4) 슬래시 구분자(/) 는 계층 관계를 나타낼 때 사용하자
URI 의 마지막 문자로 사용하면 안된다.
```
GET    /members/1/ (X)
GET    /members/1  (O)
```
5) 파일 확장자는 URI 에 포함시키지 않는다.
Accept header 헤더를 사용하여 컨텐츠 타입을 알려줄 수 있으므로, 타입을 알려주기 위해선 이 헤더를 사용하면 된다.

<br>
<br>

### 리소스 간의 관계 표현하기 
1) 소유의 관계를 표현하기
```
GET     /users/{userId}/devices ( 유저가 가지고 있는 디바이스 목록 )
```

2) 구체적인 표현이 필요할 때
```
GET    /users/{userId}/likes/devices ( 유저가 좋아하는 디바이스 목록 )
```

<br>
<br>

### 응답 메세지 처리하기
`잘 설계된 API 는 리소스에 대한 응답까지도 잘 해줘야 한다.`

1) 200대 
- 200 : 클라이언트의 요청을 정상적으로 수행
- 201 : POST 를 통한 리소스 생성 작업 시, 해당 리소스가 성공적으로 생성됨을 알려줌

2) 400대
- 400 : 클라이언트의 요청이 부적절한 경우 사용
- 401 : 인증되지 않은 클라이언트의 요청에 대한 응답
- 403 : 인가되지 않은, 응답하고 싶지 않은 리소스에 접근하려 할 때 이에 대한 응답
- 405 : 요청 리소스에 대해 사용 불가능한 method 를 이용했을 때 사용하는 응답

3) 500대
- 500 : 서버에 문제가 있을 경우 사용하는 응답 코드

4) 300대
- 301 : 요청한 리소스에 대한 URI 가 변경되었을 때 사용하는 응답 코드

## Reference
- https://jaenyeong.github.io/interview/next-step-interview/#%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC
- https://velog.io/@couchcoding/%EA%B0%9C%EB%B0%9C-%EC%B4%88%EB%B3%B4%EB%A5%BC-%EC%9C%84%ED%95%9C-RESTful-API-%EC%84%A4%EA%B3%84-%EA%B0%80%EC%9D%B4%EB%93%9C
- https://velog.io/@shroad1802/REST
- https://withbundo.blogspot.com/2017/07/http-13-http-iii-if-match-if-modified.html
- https://developer.mozilla.org/ko/docs/Web/HTTP/Caching
- https://sharplee7.tistory.com/49
