
## HttpMessageConverter

HttpMessageConverter는 기존의 뷰 템플릿으로 HTML을 생성해서 응답하는 것이 아니라 HTTP API(REST API)처럼 JSON형식의 데이터 메시지바디를 직접 읽거나
쓰기 위해 메시지 본문을 다루는 방식을 말한다.

스프링 MVC는 다음의 경우에 HTTP 메시지 컨버터를 적용한다.
- HTTP 요청 : @RequestBody, HttpEntity(ResponseEntity) (ResponseEntity는 HttpEntity를 상속받았음) 
- HTTP 응답 : @ResponseBody, HttpEntity(ResponseEntity)

기존의 요청 URL 파라미터, HTML Form 방식에서의 viewResolver대신 HttpMessageConverter이 동작하게 된다.

메시지 컨버터 인터페이스 구조를 보면

```java
public interface HttpMessageConverter<T> {
    boolean canRead(Class<?> var1, @Nullable MediaType var2);

    boolean canWrite(Class<?> var1, @Nullable MediaType var2);

    List<MediaType> getSupportedMediaTypes();

    T read(Class<? extends T> var1, HttpInputMessage var2) throws IOException, HttpMessageNotReadableException;

    void write(T var1, @Nullable MediaType var2, HttpOutputMessage var3) throws IOException, HttpMessageNotWritableException;
}
```

- canRead(), canWrite() : 메시지 컨버터가 해당 클래스 혹은 미디어 타입을 지원하는지 체크하는 용도
- read(), write() : 메시지 컨버터를 통해서 메시지를 읽고 쓰는 기능 지원

### HTTP 요청 데이터 읽는 과정

- HTTP 요청이 들어오고, 컨트롤러에서 @RequestBody 혹은 HttpEntity 파라미터 사용하는 상황
- MessageConverter가 메시지를 읽을 수 있는지 확인하기 위해 canRead() 메서드 호출
- canRead() 조건을 만족할 경우 read() 메서드를 호출해서 객체 생성 후 반환

### HTTP 응답 데이터 생성하는 과정

- 컨트롤러에서 @ResponseBody 혹은 HttpEntity로 값이 반환되는 상황
- MessageConverter가 메세지를 쓸 수 있는지 확인하기 위해 canWrite() 메서드 호출
- canWrite() 조건을 만족할 경우 write() 메서드를 호출해서 HTTP 응답 메세지 바디 내 데이터 생성

### 주로 사용하는 메시지 컨버터 세가지 (우선순위 순)

1. ByteArrayHttpMessageConverter
   - 클래스 타입은 byte[]
   - content-type 미디어 타입은 `*/*`
   - byte[]로 데이터를 처리한다.
2. StringHttpMessageConverter
   - 클래스 타입은 String
   - content-type 미디어 타입은 `*/*`
   - String 문자로 데이터를 처리한다.
3. MappingJackson2HttpMessageConverter
   - 클래스 타입은 HashMap
   - content-type 미디어 타입은 `application/json`
   - json으로 데이터를 처리한다.

대상 클래스 타입과 미디어 타입 둘을 체크해서 사용여부를 결정한다. 만약 만족하지 않으면 다음 메시지 컨버터로 우선순위가 넘어간다.

```java
content-type : application/json

@RequestMapping
void hello(@RequestBody String data) {}
```

위 코드의 경우 일단 String 타입이니 ByteArrayHttpMessageConverter는 스킵되고 그 다음 컨버터인 StringHttpMessageConverter가 적합하다고 판단할 것이다. 그 후에 미디어 타입을 체크하게 되는데 해당 컨버터는 모든 미디어 타입을 받아들이니 이것 또한 통과되어 StringHttpMessageConverter를 사용하게 된다.

> request의 경우에는 content-type의 미디어 타입을 확인했다면 response를 하는 이 경우에는 accept의 미디어 타입을 확인한다는 점이 차이가 있다.


### HttpMessageConverter가 쓰이는 시점

<img width="715" alt="image" src="https://user-images.githubusercontent.com/70622731/156185167-31d7ef3f-dda3-4ace-8e8e-72315bf6a226.png">

@RequestMapping을 처리하는 HandlerAdapter인 RequestMappingHandlerAdapter에서 쓰인다.

<img width="875" alt="image" src="https://user-images.githubusercontent.com/70622731/157375173-c1c1423b-590e-497e-a428-902725f932b7.png">

- 어노테이션 기반 컨트롤러는 HttpServletRequest, Model, @RequestParam, @ModelAttribute, @RequestBody, HttpEntity와 같이 다양한 파라미터를 사용할 수 있는데 이렇게 유연하게 처리해줄 수 있는 이유는 ArgumentResovler 덕분

- 즉, 컨트롤러를 처리하는 RequestMappingHandlerAdapter가 ArgumentResolver를 호출하면서 필요로 하는 다양한 파라미터의 값을 생성한 후 컨트롤러를 호출하며 파라미터를 넘겨줌 (사진 내 1번과 2번 과정)

- ReturnValueHandler는 응답 값을 변환하고 처리하는데, 컨트롤러에서 String으로 뷰의 논리적 이름만 반환하더라도 동작하는 이유가 ReturnValueHandler 덕분 (사진 내 3번 과정)

- 이 과정에서 HttpMessageConverter는 요청과 응답을 처리하는 과정에서 모두 필요한데 아래와 같은 상황에서 사용이 됨
  - 요청: @RequestBody 혹은 HttpEntity를 처리하는 ArgumentResovler가 HttpMessageConverter를 사용해서 필요하는 객체를 생성
  - 응답: @ResponseBody 혹은 HttpEntity를 처리하는 ReturnValueHandler에서 HttpMessageConverter를 사용해서 응답 결과 생성


### ObjectMapper

ObjectMapper란 json 형식을 사용할 때, 응답들을 직렬화하고 요청들을 역직렬화 할 때 사용하는 기술이다.

스프링 부트의 경우, spring-boot-starter-web에 기본적으로 jackson 라이브러리가 들어있다.

만약 @RestController 어노테이션을 사용중이라면 @ResponseBody가 포함되어있어 json 요청을 받을경우 MappingJackson2HttpMessageConverter 안에 ObjectMapper를 통해 json을 Object로 변환시켜준다.

ObjectMapper를 직접 사용할 수도 있다.

__Object -> Json__

```java
ObjectMapper mapper = new ObjectMapper();
Car car = new Car("K5", "gray");

String text = mapper.WriteValueAsString(car); //{"name":"K5","color":"gray"}
```

__Json -> Object__

```java
// text = {name='k5',color='gary'}
Car carObject = mapper.readValue(text, Car.class); //Car{name='k5',color='gary'}
```


## ResponseEntity

ResponseEntity에 앞서 먼저 @ResponseBody는 HTTP 규격에 맞는 응답을 만들어주기 위한 어노테이션 이지만 단점으로 HTTP 규격 구성요소중 하나인 Header에 대해서 유연하게 설정을 할 수 없다는 단점이 있다. 또한 status도 메서드 밖에서
Annotation을 사용하여 따로 설정을 해주어야 한다는점이 있다.

이와 같은 점들을 해결해 줄 수 있는 것이 ResponseEntity라는 객체이다.

@ResponseBody 와 달리 Annotation 이 아닌 객체로 사용된다. 즉, 응답으로 변환될 정보를 모두 담은 요소들을 객체로 만들어서 반환한다.

객체의 구성요소에서 HttpMessageConverter 는 응답이 되는 본문을 처리해준 뒤에, RESTTemplate 에 나머지 구성 요소인 Status 를 넘겨주게 된다.

```java
//ResponseEntity 선언 구조
public class ResponseEntity extends HttpEntity {

  private final Object status;
  
   ...
  // 여러 생성자와 빌더 메서드가 있다.
}
```

위와 같이 Status를 필드값으로 가지고 있어 ResponseEntity에서 직접적으로 Status Code를 지정할 수 있다.

나머지 부분들은 HttpEntity에 구현되어 있다.

```java
//HttpEntity 선언 구조
public class HttpEntity<T> {
    public static final HttpEntity<?> EMPTY = new HttpEntity<>();
  
  
    private final HttpHeaders headers;
  
    @Nullable
    private final T body;
}
```

위와 같이 HttpHeaders를 필드타입으로 가지고 있어 @ResponseBody와 다르게 객체 안에서 Header를 설정해 줄 수 있다.

ResponseEntity를 사용할 때, Constructor를 사용하기 보다는 Builder를 활용하는 것을 권장하고 있다. 그 이유는 빌더를 사용하면 유연성을 확보할 수 있으며 가독성을 높일 수 있다.

```java
// 생성자
return new ResponseEntity<MoveResponseDto>(moveResponseDto, headers, HttpStatus.valueOf(200));

// 빌더
return ResponseEntity.ok()
     .headers(headers)
     .body(moveResponseDto);
```

---

### Reference

- https://cornarong.tistory.com/37
- https://bepoz-study-diary.tistory.com/374
- https://jaimemin.tistory.com/1823
- https://escapefromcoding.tistory.com/341
