# 모델 바인딩 및 검증

핸들러 메서드에 @ModelAttribute 가 지정되면 다음과 같은 일이 일어난다.

- __파라미터 타입의 오브젝트를 만든다.__
  - Ex. @ModelAttribute User user : User 라는 새로운 오브젝트를 만든다.
  - `@SessionAttributes` 에 의해 세션에 저장된 모델 오브젝트가 있다면, 세션에 저장되어 있는 오브젝트를 가져온다.
- __웹 파라미터를 오브젝트 프로퍼티에 바인딩 한다.__
  - HTTP 를 통해 전달되는 파라미터는 기본적으로 문자열 형식이라서 String 객체가 아닌 이상 적절한 변환이 필요하다.
  - 이때, 스프링에서 제공하는 기본 프로퍼티 에디터를 통해서 `HTTP Parameter -> Object Property` 로 변환한다.
  - 변환이 불가능하면 BindingResult 안에 오류를 저장해서 적절한 처리를 해야 한다.
- __모델의 값을 검증한다.__
  - 타입에 대한 검증은 끝났지만, 그 외의 검증할 내용이 있다면 적절한 검증기를 등록해서 모델의 내용을 검증할 수 있다.

> 스프링에서 바인딩이란 오브젝트의 프로퍼티에 값을 넣는 것을 말한다.

## PropertyEditor

- 스프링이 기본적으로 제공하는 바인딩용 타입 변환 API 이다.
- PropertyEditor 는 사실 스프링 API 는 아니고 자바빈 표준에 정의된 인터페이스다.
- __XML 의 value 애트리뷰트랑, @Controller 의 파라미터에 적용된다.__

```xml
<bean id="dataSource" class="org...SimpleDriverDataSource">
 <!-- com.mysql.jdbc.Driver 클래스 타입의 오브젝트를 만들어서 driverClass 라는 프로퍼티에 바인딩한다. -->
 <property name = "driverClass"  value = "com.mysql.jdbc.Driver" />
</bean>
```

```java
// CharsetEditor 사용
@RequestMapping("/hello")
public void hello(@RequestParam Charset charset, Model model) {
}
```

### PropertyEditor 에서 제공하는 메서드

> HTTP 요청 파라미터와 같은 문자열은 스트링 타입으로 서블릿에서 가져온다.

- __String to Object__
  - setAsText() 로 String 타입의 문자열을 넣고 getValue() 로 변환된 오브젝트를 가져온다.
- __Object to String__
  - setValue() 로 오브젝트를 넣고 getAsText() 로 변환된 문자열을 가져온다.

따라서, 커스텀 프로퍼티 에디터를 만들 때는, setAsText(), getAsText() 부분만 손보면 된다.

### 커스텀 프로퍼티 에디터

> PropertyEditor 인터페이스를 직접 구현하기보다는 기본 구현이 되어있는 PropertyEditorSupport 클래스를 상속해서 필요한 메서드만 오버라이딩 하는게 낫다.

```kotlin
class LevelPropertyEditor: PropertyEditorSupport() {

    override fun getAsText(): String {
        // this.value -> setValue 에 의해 저장된 Level 타입의 오브젝트를 가져와서 값을 문자로 변환한다.
        return ((this.value as Level).intValue().toString())
    }

    override fun setAsText(text: String?) {
        this.value = text?.trim()?.let { Level.valueOf(it.toInt()) }
    }
}
```

만든 커스텀 프로퍼티 에디터가 스프링 MVC 에서 동작하게 하려면 `@InitBinder` 를 사용하면 된다.

## @InitBinder

- __컨트롤러 메서드에서 바인딩이 어떻게 일어날까?__
  - @Controller 핸들러 메서드를 호출해줄 책임이 있는 `AnnotationMethodHandlerAdapter` 는 @RequestParam, @ModelAttribute 등 HTTP 요청을 파라미터 변수에 바인딩 해주는 작업이 필요한 어노테이션을 만나면 먼저 `WebDataBinder` 라는 것을 만든다.
- __WebDataBinder__
  - HTTP 요청으로부터 가져온 문자열을 파라미터 타입의 오브젝트로 변환해주는 기능이 있다. 이때 `PropertyEditor` 를 사용하는 것이다.
  - 따라서, 커스텀 프로퍼티 에디터를 사용하기 위해서는 WebDataBinder 에 등록해줘야 한다.
  - WebDataBinder 는 커스텀 프로퍼티 에디터가 있으면 먼저 적용하고, 적절한 프로퍼티 에디터가 없다면 그때 스프링에서 제공하는 디폴트 프로퍼티 에디터중 하나를 사용하게 된다.

```kotlin
@RestController
class ConversionController {

    @InitBinder
    fun initBinder(webDataBinder: WebDataBinder) {
        webDataBinder.registerCustomEditor(Level::class.java, LevelPropertyEditor())
    }
    
    @GetMapping("/level")
    fun levelCustomEditor(@RequestParam level: Level): Int {
        return level.intValue()
    }
}
```

## 프로토타입 빈 프로퍼티 에디터

오브젝트를 매번 새로 만드는 대신 프로퍼티 에디터를 싱글톤 빈으로 등록해두고 공유해서 쓸 수 없을까? 아쉽지만 __프로퍼티 에디터는 싱글톤 빈으로 등록될 수 없다.__ 
프로퍼티 에디터에 의해 타입이 변경되는 오브젝트는 한 번은 프로퍼티 에디터 오브젝트 `내부에 저장`된다는 사실을 알 수 있다. __따라서, 멀티 스레드 환경에서 싱글톤으로 만들어 공유해서 사용하면 안된다.__

그런데 프로퍼티 에디터가 다른 스프링 빈을 참조해야 한다면 어떨까?

다른 빈을 참조해서 DI 받으려면 자신도 스프링 빈으로 등록되어야 한다. 이를 위해 프로퍼티 에디터가 다른 빈을 DI 받을 수 있도록 자신도 빈으로서 등록되면서 동시에 매번 새로운 오브젝트를 만들어서사용할 수 있으려면 `프로토타입 스코프의 빈`으로 만들어져야 한다.

__프로토타입 스코프 빈은 매번 빈 오브젝트를 요청해서 새로운 오브젝트를 가져올 수 있으면서 DI 도 가능하다.__

```java
@Component
@Scope("prototype")
public class CodePropertyEditor extends PropertyEditorSupport{
  @Atuworied codeService;

  @Override
	public void setAsText(String text) throws IllegalArgumentException {
		Code code = codeService.getCode(Integer.valueOf(text));

		this.setValue(code);
	}
}
```
```java
@Controller
public class UserController{
  @Inject Provider<CodePropertyEditor> codePropertyEditorProvider;

  @InitBinder
  public void initBinder(WebDataBinder dataBinder){
    dataBinder.registerCustomEditor(Code.class, codePropertyEditorProvider.get());
  }

  @RequestMapping("/user", method=RequestMethod.POST)
  public String userAdd(@ModelAttribute User user){
    // ...
  }
}
```

- 이 방식의 장점은 항상 완전한 도메인 오브젝트를 리턴해주므로, 앞서 제기했던 위험이 없어진다.
- 단점으로는 매번 DB에서 조회를 해야하므로 성능에 조금 부담을 주는 단점이 있다.
- JPA 와 같이 엔티티 단위의 캐싱 기법이 발달한 기술을 사용할 경우, DB에서 조회하는 대신 메모리에서 바로 읽어올 수 있으므로 DB 부하에 대한 걱정은 하지 않아도 된다.

# Converter

PropertyEditor는 근본적인 단점이 있다. __상태를 가지고 있으므로 싱글톤으로 등록할 수 없고, 항상 새로운 오브젝트를 만들어야 한다는 점이다.__ 물론 생성되는 오브젝트 자체가 가볍기 때문에 크게 문제될 것은 없지만, __싱글톤 서비스 오브젝트 중심의 스프링__ 과는 잘 어울리지 않는다. 특히, 빈으로 등록해서 사용할 떄는 반드시 `프로토타입 스코프`를 사용해야하기 때문에 불편하다.

스프링 3.0이후로 이러한 PropertyEditor 의 단점을 보완해주는 `Converter 라는 타입 변환 API` 가 등장하였다. Converter 는 PropertyEditor 와 달리 변환과정에서 메서드가 한번만 호출된다.
__즉, 상태를 가지지 않는다는 뜻이고, 싱글톤으로 등록할 수 있다는 뜻이다.__

## Converter

- Converter 인터페이스

```java
public interface Converter<S, T>{
  T convert(s source);
}
```

```kotlin
class LevelToStringConverter: Converter<Level, String> {
    override fun convert(source: Level): String {
        return source.intValue().toString()
    }
}

class StringToLevelConverter: Converter<String, Level> {
    override fun convert(source: String): Level {
        return Level.valueOf(source.toInt())
    }
}
```

이렇게 두 개의 컨버터를 사용하면 PropertyEditor 를 사용했을 때 처럼 동일한 효과를 낼 수 있다. 또한 `Thread-safe` 하기 때문에 멀티 스레드 환경에서도 안전하게 사용할 수 있다.

## ConversionService

- `ConversionService` 는 여러 종류의 컨버터를 이용해서 하나 이상의 타입 변환 서비스를 제공해주는 오브젝트를 만들 때 사용하는 인터페이스다.
- 보통 ConversionService 를 구현한 `GenericConversionService` 클래스를 빈으로 등록해서 사용하면 된다.
- `GenericConversionService` 는 스프링의 다양한 타입 변환 기능을 가진 오브젝트를 등록할 수 있는 `ConverterRegistry` 인터페이스도 구현하고 있다.
- 새로운 타입 변환 오브젝트느 `GenericConverter, ConverterFactory` 를 사용해서 만들 수도 있다.
- GenericConversionService 는 일반적으로 빈으로 등록하고 필요한 컨트롤러에서 DI 받아서 @InitBinder 메서드를 통해 WebDataBinder 에 설정하는 방식으로 사용한다.
- 컨트롤러의 바인딩 작업에 컨버터를 적용하기 위해서는 ConversionService 타입의 오브젝트를 통해 WebDataBinder 에 설정해줘야 한다.

### 스프링 XML 설정

```xml
<bean class="org.springframework..ConversionServiceFactoryBean">
  <property name="converters">
    <set>
      <bean class="LevelConverter" />
      <!-- 추가하고 싶은 Converter들... -->
    </set>
  </property>
</bean>
```
```java
@Controller
public class UserController{
  @Autowired ConversionService conversionService;

  @InitBinder
  public void initBinder(WebDataBinder dataBinder){
    dataBinder.setConversionService(this.conversionService);
  }

  @RequestMapping("/user", method=RequestMethod.GET)
  public String userSearch(@RequestParam Level level){
    // ...
  }
}
```

### 스프링 부트 설정

스프링 부트에서는 아래처럼 편리하게 사용할 수 있다. `@InitBinder` 를 컨트롤러에서 사용하지 않아도 컨버터가 동작한다.

```kotlin
@Configuration
class WebMvcConfig: WebMvcConfigurer {

    override fun addFormatters(registry: FormatterRegistry) {
        registry.addConverter(StringToLevelConverter())
        registry.addConverter(LevelToStringConverter())
    }
}
```

# Formatter

- 웹 애플리케이션에서 객체를 문자로, 문자를 객체로 변환하는 예
  - 화면에 숫자를 출력해야 하는데, Integer String 출력 시점에 숫자 1000 문자 "1,000" 이렇게 1000 단위에 쉼표를 넣어서 출력하거나, 또는 "1,000" 라는 문자를 1000 이라는 숫자로 변경해야 한다.
  - 날짜 객체를 문자인 "2021-01-01 10:50:11" 와 같이 출력하거나 또는 그 반대의 상황
  - 여기에 추가로 날짜 숫자의 표현 방법은 Locale 현지화 정보가 사용될 수 있다.

__이렇게 객체를 특정한 포맷에 맞추어 문자로 출력하거나 또는 그 반대의 역할을 하는 것에 특화된 기능이 바로 포맷터(Formatter)이다. 포맷터는 컨버전의 특별한 버전으로 이해하면 된다.__

- __Converter vs Formatter__
  - Converter 는 범용(객체 -> 객체)
  - Formatter 는 문자에 특화(객체 -> 문자, 문자 -> 객체) + 현지화(Locale)
  - Converter 의 특별한 버전

## 포맷터 - Formatter 만들기

포맷터(Formatter)는 객체를 문자로 변경하고, 문자를 객체로 변경하는 두 가지 기능을 모두 수행한다.

- String print(T object, Locale locale) : 객체를 문자로 변경한다.
- T parse(String text, Locale locale) : 문자를 객체로 변경한다.

```java
/**
 * Formats objects of type T.
 * A Formatter is both a Printer <i>and</i> a Parser for an object type.
 *
 * @author Keith Donald
 * @since 3.0
 * @param <T> the type of object this Formatter formats
 */
public interface Formatter<T> extends Printer<T>, Parser<T> {

}
```
```java
public interface Printer<T> {
  String print(T object, Locale locale);
}

public interface Parser<T> {
  T parse(String text, Locale locale) throws ParseException;
}
```

숫자 1000 을 문자 "1,000" 으로 그러니까, 1000 단위로 쉼표가 들어가는 포맷을 적용해보자. 그리고 그
반대도 처리해주는 포맷터를 만들어보자.

```java
@Slf4j
public class MyNumberFormatter implements Formatter<Number> {

    // 문자를 숫자로
    @Override
    public Number parse(String text, Locale locale) throws ParseException {
        log.info("text={}, locale={}", text, locale);
        // "1,000" -> 1000
        NumberFormat format = NumberFormat.getInstance(locale);
        return format.parse(text);
    }

    // 숫자를 문자로
    @Override
    public String print(Number object, Locale locale) {
        log.info("object={}, locale={}", object, locale);
        return NumberFormat.getInstance(locale).format(object);
    }
}
```

"1,000" 처럼 숫자 중간의 쉼표를 적용하려면 자바가 기본으로 제공하는 `NumberFormat` 객체를 사용하면
된다. 이 객체는 Locale 정보를 활용해서 나라별로 다른 숫자 포맷을 만들어준다.

- 테스트 코드

```java
class MyNumberFormatterTest {

    MyNumberFormatter formatter = new MyNumberFormatter();

    @Test
    void parse() throws ParseException {
        Number result = formatter.parse("1,000", Locale.KOREA);
        assertThat(result).isEqualTo(1000L); // Long 타입 주의
    }

    @Test
    void print() {
        String result = formatter.print(1000, Locale.KOREA);
        assertThat(result).isEqualTo("1,000");
    }
}
```

parse() 의 결과가 Long 이기 때문에 isEqualTo(1000L) 을 통해 비교할 때 마지막에 L 을 넣어주어야
한다.

- 실행 결과

```
MyNumberFormatter - text=1,000, locale=ko_KR
MyNumberFormatter - object=1000, locale=ko_KR
```

스프링은 용도에 따라 다양한 방식의 포맷터를 제공한다. 

- Formatter 포맷터
- AnnotationFormatterFactory 필드의 타입이나 애노테이션 정보를 활용할 수 있는 포맷터

> 자세한 내용은 공식 문서를 참고하자.
>
> https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#format

## 포맷터를 지원하는 컨버전 서비스

컨버전 서비스에는 컨버터만 등록할 수 있고, 포맷터를 등록할 수 는 없다. 그런데 생각해보면 포맷터는 객체 -> 문자, 문자 -> 객체로 변환하는 특별한 컨버터일 뿐이다.
포맷터를 지원하는 컨버전 서비스를 사용하면 컨버전 서비스에 포맷터를 추가할 수 있다. 내부에서 `어댑터 패턴`을 사용해서 Formatter 가 Converter 처럼 동작하도록 지원한다.

`FormattingConversionService` 는 포맷터를 지원하는 컨버전 서비스이다.

`DefaultFormattingConversionService` 는 FormattingConversionService 에 기본적인 통화, 숫자 관련 몇가지 기본 포맷터를 추가해서 제공한다.

- 테스트 코드

```java
public class FormattingConversionServiceTest {

    @Test
    void formattingConversionService() {
        DefaultFormattingConversionService conversionService = new DefaultFormattingConversionService();
        
        // 컨버터 등록
        conversionService.addConverter(new StringToIpPortConverter());
        conversionService.addConverter(new IpPortToStringConverter());
        
        // 포맷터 등록
        conversionService.addFormatter(new MyNumberFormatter());

        // 컨버터 사용
        IpPort ipPort = conversionService.convert("127.0.0.1:8080", IpPort.class);
        assertThat(ipPort).isEqualTo(new IpPort("127.0.0.1", 8080));
        
        // 포맷터 사용
        assertThat(conversionService.convert(1000, String.class)).isEqualTo("1,000");
        assertThat(conversionService.convert("1,000", Long.class)).isEqualTo(1000L);

    }
}
```

### DefaultFormattingConversionService 상속 관계

FormattingConversionService 는 ConversionService 관련 기능을 상속받기 때문에 결과적으로
컨버터도 포맷터도 모두 등록할 수 있다. 그리고 사용할 때는 ConversionService 가 제공하는 convert
를 사용하면 된다.

추가로 스프링 부트는 `DefaultFormattingConversionService` 를 상속 받은 `WebConversionService` 를 내부에서 사용한다.

### 포맷터 적용하기

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addFormatters(FormatterRegistry registry) {
        // 주석처리 우선순위
//        registry.addConverter(new StringToIntegerConverter());
//        registry.addConverter(new IntegerToStringConverter());
        registry.addConverter(new StringToIpPortConverter());
        registry.addConverter(new IpPortToStringConverter());

        // 추가
        registry.addFormatter(new MyNumberFormatter());
    }
}
```

MyNumberFormatter 도 숫자 -> 문자, 문자 -> 숫자로 변경하기 때문에 둘의 기능이 겹친다. 우선순위는
컨버터가 우선하므로 포맷터가 적용되지 않고, 컨버터가 적용된다.

## 스프링이 제공하는 기본 포맷터

스프링은 자바에서 기본으로 제공하는 타입들에 대해 수 많은 포맷터를 기본으로 제공한다.

그런데 포맷터는 기본 형식이 지정되어 있기 때문에, 객체의 각 필드마다 다른 형식으로 포맷을 지정하기는
어렵다.

스프링은 이런 문제를 해결하기 위해 애노테이션 기반으로 원하는 형식을 지정해서 사용할 수 있는 매우
유용한 포맷터 두 가지를 기본으로 제공한다.

- `@NumberFormat` : 숫자 관련 형식 지정 포맷터 사용, `NumberFormatAnnotationFormatterFactory`
- `@DateTimeFormat` : 날짜 관련 형식 지정 포맷터 사용, Jsr310DateTimeFormatAnnotationFormatterFactory

- FormatterController

```java
@Controller
public class FormatterController {

    @GetMapping("/formatter/edit")
    public String formatterForm(Model model) {
        Form form = new Form();
        form.setNumber(10000);
        form.setLocalDateTime(LocalDateTime.now());
        model.addAttribute("form", form);
        return "formatter-form";
    }

    @PostMapping("/formatter/edit")
    public String formatterEdit(@ModelAttribute Form form) {
        return "formatter-view";
    }

    @Data
    static class Form {
        @NumberFormat(pattern = "###,###")
        private Integer number;

        @DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss")
        private LocalDateTime localDateTime;
    }
}
```

- formatter-from.html

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
 <meta charset="UTF-8">
 <title>Title</title>
</head>
<body>
<form th:object="${form}" th:method="post">
 number <input type="text" th:field="*{number}"><br/>
 localDateTime <input type="text" th:field="*{localDateTime}"><br/>
 <input type="submit"/>
</form>
</body>
</html>
```

- formatter-view.html

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
 <meta charset="UTF-8">
 <title>Title</title>
</head>
<body>
<ul>
 <li>${form.number}: <span th:text="${form.number}" ></span></li>
 <li>${{form.number}}: <span th:text="${{form.number}}" ></span></li>
 <li>${form.localDateTime}: <span th:text="${form.localDateTime}" ></span></li>
 <li>${{form.localDateTime}}: <span th:text="${{form.localDateTime}}" ></span></li>
</ul>
</body>
</html>
```

- 결과

```
${form.number}: 10000
${{form.number}}: 10,000
${form.localDateTime}: 2021-01-01T00:00:00
${{form.localDateTime}}: 2021-01-01 00:00:00
```

> https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#format-CustomFormatAnnotations

## 주의

`메시지 컨버터(HttpMessageConverter)`에는 컨버전 서비스가 적용되지 않는다.

특히 객체를 JSON으로 변환할 때 메시지 컨버터를 사용하면서 이 부분을 많이 오해하는데,
HttpMessageConverter 의 역할은 HTTP 메시지 바디의 내용을 객체로 변환하거나 객체를 HTTP 메시지
바디에 입력하는 것이다.

예를 들어서 JSON을 객체로 변환하는 메시지 컨버터는 내부에서 Jackson 같은
라이브러리를 사용한다. 객체를 JSON으로 변환한다면 그 결과는 이 라이브러리에 달린 것이다. 따라서
JSON 결과로 만들어지는 숫자나 날짜 포맷을 변경하고 싶으면 해당 라이브러리가 제공하는 설정을 통해서
포맷을 지정해야 한다. 

결과적으로 이것은 컨버전 서비스와 전혀 관계가 없다. 컨버전 서비스는 @RequestParam , @ModelAttribute , @PathVariable , 뷰 템플릿 등에서 사용할 수 있다.

## References

- 토비의 스프링3
