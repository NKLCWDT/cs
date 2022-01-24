# Java Annotation

- 어노테이션은 Java SE 5 에서 Generics 와 함께 등장하였다.
- 어노테이션의 사전적 의미는 주석이지만, 자바 언어에서 사용하는 주석(`//, /*, /**`) 과는 다르다.

## 주석의 등장 배경

어노테이션의 사전적 의미인 주석의 등장 배경을 먼저 보자. 

주석이 없던 시절에는, 소스 코드와 문서화가 별도로 진행되었다. 따라서, 소스 코드가 변경되면 그에 알맞은 문서를 찾아서 변경해줘야 했었다.

자고로 개발자는 귀찮은 것을 매우 싫어한다. 

![IMAGES](../images/metamong.jpg)

따라서, 소스 코드만 변경하고 문서를 변경하지 않는 일이 자주 발생하였고, 코드와 문서의 버전 불일치 문제를 해결하고자 탄생하게 되었다.

## 어노테이션의 등장 배경

어노테이션이 등장하기 전에는 프로그램 설정 파일들이 `XML` 형태로 작성되었다.

`컴포넌트 스캔(Component-scan)`을 예로 들어보자.

스프링에서는 XML 에서 컴포넌트 스캔을 사용하였다.

```xml
<context:component-scan base-package="com.spring.study">
    <context:exclude-filter type="annotation" 
        expression="org.springframework.stereotype.Controller"/>
</context:component-scan>
```

하지만 스프링 부트에서는 `@Configuration` 과 `@ComponentScan` 어노테이션을 활용하여 XML 이 아닌 자바 소스코드를 활용하여 설정 정보를 관리할 수 있게 되었다.

```java
@ComponentScan(basePackages = "com.spring.study")
@Configuration
public class AppConfig {
}
```

> @Test 어노테이션도 마찬가지로 해당 메서드가 테스트 대상임을 Junit Framework 에게 알린다.

어노테이션의 등장으로 인한 장점은 다음과 같다.

- 어노테이션을 통해 소스 코드와 설정 정보를 같이 관리할 수 있어서 편하다.
- 비지니스 로직을 방해하지 않고, 필요한 정보를 제공할 수 있다.

## 특징

Annotations do not directly affect program semantics, but they do affect the way programs are treated by tools and libraries, which can in turn affect the semantics of the running program. Annotations can be read from source files, class files, or reflectively at run time.

> [Annotations docs](https://docs.oracle.com/javase/8/docs/technotes/guides/language/annotations.html)

Document 에 나와있는 내용을 읽어보자.

> 주석은 프로그램 의미론에 직접적인 영향을 미치지 않지만 프로그램이 도구 및 라이브러리에서 처리되는 방식에 영향을 미치므로 실행 중인 프로그램의 의미론에 영향을 줄 수 있습니다. 주석은 소스 파일, 클래스 파일에서 읽거나 런타임에 반사적으로 읽을 수 있습니다.

여기서 `런타임에 반사적`으로 읽을 수 있다는 의미만 살짝 다뤄보겠다.

## 어노테이션을 런타임에 반사적으로 읽기

반사적이란 의미가 무엇일까? 

![IMAGES](../images/jjanggu.jpg)

- __반사적__
    - 사전적 의미 : 어떠한 자극에 순간적으로 무의식적 반응을 보이는 것
    - Ex. 스프링 DI 를 예시로 들면, 런타임에 클래스가 생성되면(`자극`) 어노테이션 정보를 읽어서 의존성 주입(`무의식적 반응`)을 한다.
	
- __비지니스 로직을 방해하지 않고, 필요한 정보를 제공할 수 있다.__
    - Ex. 스프링 DI
    	- 런타임에 반사적으로 읽는다.
    - Ex. @Getter, @Setter
     	- 컴파일 시, 바이트 코드에 Getter, Setter 코드 생성 

우리가 스프링을 사용하게되면 DI(Dependency Injection) 을 자주 사용하게 되는데, 어떻게 런타임에 의존성이 주입이 될까?

```java
@Target({TYPE, FIELD, METHOD})
@Retention(RUNTIME)
@Repeatable(Resources.class)
public @interface Resource {
    // 생략
}
```

```java
@Service
public class UserService {

    @Resource(name = "userRepository")
    private UserRepository userRepository
}
```

단순히, 어노테이션만 추가했다고 의존성이 주입되는건가? 어노테이션 등장 배경을 살펴보면 그러한 목적을 위해서 탄생 한 것 같진 않다.

여기에는 `Reflection` 이라는 기술이 사용된다.

> Reflection 기술을 이용하여 어노테이션을 효과적으로 활용할 수 있다.

런타임에 UserService 객체가 생성되는 시점(클래스 로더에 의해 메모리에 적재 되는 순간)에 해당 클래스의 필드에 선언 되어있는 어노테이션 정보를 읽어서 해당 필드의 객체를 생성하여 주입해준다.

## 표준 어노테이션

- __내장 어노테이션__
	- __@Override__
	    - 컴파일러에게 오버라이딩하는 메서드임을 알린다.
	- __@Deprecated__
	    - 앞으로 사용하지 말 것을 권장하는 대상에게 붙인다.
	- __@SuppressWarnings__
	    - 컴파일러의 특정 경고 메시지가 나타나지 않게 해준다.
	    - Effective Java Item 27
	- __@SafeVarargs__
	    - 제네릭 타입의 가변인자에 사용한다. (JDK 1.7)
	    - Effective Java Item 32
	- __@FunctionalInterface__
	    - 함수형 인터페이스라는 것을 알린다. (JDK 1.8)
- __메타 어노테이션__
	- __@Target__
	    - 어노테이션이 적용 가능한 대상을 지정하는데 사용한다.
	- __@Documented__
	    - 어노테이션 정보가 javadoc 으로 작성된 문서에 포함되게 한다.
	- __@Inherited__
	    - 어노테이션이 하위 클래스에게 상속 되도록 한다.
	- __@Retention__
	    - 어노테이션이 유지되는 범위를 지정하는데 사용한다.

> [Meta Annotations](https://en.wikibooks.org/wiki/Java_Programming/Annotations/Meta-Annotations)
>
> @Target, @Documented, @Inherited, @Retention, @Repeatable 은 메타 어노테이션이라고도 부른다. 메타 어노테이션을 활용하여 어노테이션을 커스터마이징 할 수 있다.
>
> 메타 데이터란 어플리케이션이 처리해야 할 데이터가 아니라, 컴파일 타임과 런타임에서 코드를 어떻게 컴파일하고 처리할 것인지 알려주는 정보이다.

## @ComponentScan 을 통한 메타 어노테이션 알아보기

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Repeatable(ComponentScans.class)
public @interface ComponentScan {
    // 생략
}
```

### @Retention

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Retention {
    /**
     * Returns the retention policy.
     * @return the retention policy
     */
    RetentionPolicy value();
}
```

- __@Retention__
    - 어노테이션이 어느 시점 까지 유지되는지를 정한다.
    - @Target 이 `ElementType.ANNOTATION_TYPE` 으로 되어있는 것으로 봐서, 어노테이션에만 지정할 수 있는 어노테이션이다.

잠시 `RetentionPolicy` 를 까보자.

```java
/**
 * Annotation retention policy.  The constants of this enumerated type
 * describe the various policies for retaining annotations.  They are used
 * in conjunction with the {@link Retention} meta-annotation type to specify
 * how long annotations are to be retained.
 *
 * @author  Joshua Bloch
 * @since 1.5
 */
public enum RetentionPolicy {
    /**
     * Annotations are to be discarded by the compiler.
     */
    SOURCE,

    /**
     * Annotations are to be recorded in the class file by the compiler
     * but need not be retained by the VM at run time.  This is the default
     * behavior.
     */
    CLASS,

    /**
     * Annotations are to be recorded in the class file by the compiler and
     * retained by the VM at run time, so they may be read reflectively.
     *
     * @see java.lang.reflect.AnnotatedElement
     */
    RUNTIME
}
```

위 주석에서 핵심 문구는 `how long annotations are to be retained.` 이다.

- __정리__
    - @Retention 어노테이션은, 어노테이션에만 적용할 수 있다.
    - @Retention 어노테이션에 RetentionPolicy 를 지정할 수 있는데, RetentionPolicy 는 어노테이션을 유지할 기간을 의미한다.

RetentionPolicy 에서 `RUNTIME` 주석 부분을 보자. RUNTIME 주석의 핵심 부분은 `retained by the VM at run time, so they may be read reflectively.` 이다.

> 런타임에 VM에 의해 유지되므로 반사적으로 읽을 수 있습니다.

그러면 다시 위로 올라가서, [어노테이션을 런타임에 반사적으로 읽기](https://github.com/BAEKJungHo/deepdiveinreflection/blob/main/contents/Annotation.md#%EC%96%B4%EB%85%B8%ED%85%8C%EC%9D%B4%EC%85%98%EC%9D%84-%EB%9F%B0%ED%83%80%EC%9E%84%EC%97%90-%EB%B0%98%EC%82%AC%EC%A0%81%EC%9C%BC%EB%A1%9C-%EC%9D%BD%EA%B8%B0) 에 있는 @Resource 어노테이션을 보자. RetentionPolicy 가 RUNTIME 으로 되어있는 것을 볼 수 있다. 따라서, @Resource 어노테이션을 필드에 사용하면 Reflection 을 통한 DI 가 가능한 것이다.

#### @Getter, @Setter, @Override 의 RetentionPolicy ?

@Getter, @Setter, @Override 의 RetentionPolicy 가 무엇으로 되어있을지 생각해 보자.

정답은 `SOURCE`로 되어있다. 

테스트를 위해 User 라는 클래스를 생성하고 컴파일 해보자.

```
@Getter @Setter
public class User {

    private Long id;
}
```

컴파일 결과는 아래와 같다.

```java
public class User {
    private Long id;

    public User() {
    }

    public Long getId() {
        return this.id;
    }

    public void setId(Long id) {
        this.id = id;
    }
}
```

RetentionPolicy 가 `SOURCE` 로 되어있어서 컴파일될 때 어노테이션은 사라지고, 어노테이션 정보를 가지고 실제 코드를 생성해준다.

### @Target

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Target {
    /**
     * Returns an array of the kinds of elements an annotation type
     * can be applied to.
     * @return an array of the kinds of elements an annotation type
     * can be applied to
     */
    ElementType[] value();
}
```

- __@Target__
    - 어노테이션을 적용할 대상을 지정한다.
    - ElementType enum 에 지정되어있는 타입 중 하나 이상을 선택할 수 있다.

```java
public enum ElementType {
    /** Class, interface (including annotation type), or enum declaration */
    TYPE,

    /** Field declaration (includes enum constants) */
    FIELD,

    /** Method declaration */
    METHOD,

    /** Formal parameter declaration */
    PARAMETER,

    /** Constructor declaration */
    CONSTRUCTOR,

    /** Local variable declaration */
    LOCAL_VARIABLE,

    /** Annotation type declaration */
    ANNOTATION_TYPE,

    /** Package declaration */
    PACKAGE,

    /**
     * Type parameter declaration
     * @since 1.8
     */
    TYPE_PARAMETER,

    /**
     * Use of a type
     * @since 1.8
     */
    TYPE_USE,

    /**
     * Module declaration.
     * @since 9
     */
    MODULE
}
```

### @Documented

@Documented 의 주석 일부를 보면 다음과 같다.

```java
/**
 * Concretely, if an annotation type is annotated with {@code
 * Documented}, by default a tool like javadoc will display
 * annotations of that type in its output while annotations of
 * annotation types without {@code Documented} will not be displayed.
 */
```

- __@Documented__
    - @Documented 를 사용하면 javadoc tool 을 사용하여 문서를 생성하면, 문서에 어노테이션 정보까지 같이 보여진다.

### @Repeatable

```java
/**
 * The annotation type {@code java.lang.annotation.Repeatable} is
 * used to indicate that the annotation type whose declaration it
 * (meta-)annotates is <em>repeatable</em>. The value of
 * {@code @Repeatable} indicates the <em>containing annotation
 * type</em> for the repeatable annotation type.
 *
 * @since 1.8
 * @jls 9.6.3 Repeatable Annotation Types
 * @jls 9.7.5 Multiple Annotations of the Same Type
 */
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Repeatable {
    /**
     * Indicates the <em>containing annotation type</em> for the
     * repeatable annotation type.
     * @return the containing annotation type
     */
    Class<? extends Annotation> value();
}
```

- __@Repeatable__
    - 반복해서 붙일 수 있는 어노테이션을 정의할 때 사용
    - 반복해서 표현할 어노테이션을 묶을 `컨테이너 어노테이션`도 함께 정의해서 사용해야 함
        - @ComponentScans 가 컨테이너 어노테이션에 해당된다.

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
public @interface ComponentScans {

	ComponentScan[] value();
}
```

@ComponentScan 은 `@Repeatable` 어노테이션 덕분에 아래와 같은 형태로도 사용이 가능하다.

```java
@ComponentScan(basePackages = "hello.test")
@ComponentScan(basePackages = "hello.src")
public class AppConfig {
}
```

### @Inherited

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Inherited {
}
```

- __@Inherited__
    - 해당 어노테이션이 붙어있는 어노테이션을 클래스에 적용하면, 하위 클래스에서도 그 어노테이션이 적용된다.

@Inherited 를 적용한 Dto 어노테이션과, @Inherited 가 없는 Dao 어노테이션을 만들어서 테스트 해 보자.

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Inherited
public @interface Dto {
}

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface Dao {
}
```

다음으로 User 와 Team 을 아래와 같이 구성하자.

```java
@Dto
@Dao
public class User {
}

public class Team extends User {
}
```

테스트 코드와 결과는 다음과 같다.

```java
@Slf4j
class AnnotationTest {

    @Test
    void inheritedTest() {
        Class<User> userClass = User.class;

        // User Dto Annotation = @reflection.study.annotation.code.Dto()
        log.info("User Dto Annotation = {}", userClass.getAnnotation(Dto.class));

        // User Dto Annotation = @reflection.study.annotation.code.Dao()
        log.info("User Dto Annotation = {}", userClass.getAnnotation(Dao.class));

        Class<Team> teamClass = Team.class;

        // Team Dto Annotation = @reflection.study.annotation.code.Dto()
        log.info("Team Dto Annotation = {}", teamClass.getAnnotation(Dto.class));

        // Team Dao Annotation = null
        log.info("Team Dao Annotation = {}", teamClass.getAnnotation(Dao.class));
    }
}
```

누군가 어노테이션은 상속 가능하냐라고 물으면, `@Inherited` 가 적용되었는지에 따라 답변하면 될 것 같다.

## 내장 어노테이션

### @Override

슈퍼 클래스의 기능을 오버라이딩할 때 `@Override` 를 작성하면 컴파일 타임에 오탈자를 확인할 수 있다.

```java
@Slf4j
public class SuperClass {
    
    public void run() {
        log.info("Run by SuperClass");
    }
}

@Slf4j
public class SubClass extends SuperClass {

    // java: method does not override or implement a method from a supertype
    @Override
    public void run1() {
        log.info("Run by SubClass");
    }

    /**
     * 오탈자를 입력했지만 @Override 를 적용하지 않아서 
     * 하위 클래스에서 만든 새로운 메서드라고 인식한다.
     */
    public void run2() {
    }
}
```

### @Deprecated

@Dprecated 는 앞으로 사용하지 말 것을 권장하는 필드나 메서드에 붙인다.

Date 클래스를 예로 들어보자. Date 에 들어있는 대부분의 메서드들은 `@Deprecated` 되었다.
주석을 읽어보면 Date 대신 Calendar 를 권장하는 것 같다.


```java
/**
    * Returns the day of the month represented by this {@code Date} object.
    * The value returned is between {@code 1} and {@code 31}
    * representing the day of the month that contains or begins with the
    * instant in time represented by this {@code Date} object, as
    * interpreted in the local time zone.
    *
    * @return  the day of the month represented by this date.
    * @see     java.util.Calendar
    * @deprecated As of JDK version 1.1,
    * replaced by {@code Calendar.get(Calendar.DAY_OF_MONTH)}.
    */
@Deprecated
public int getDate() {
    return normalize().getDayOfMonth();
}
```

하지만 @Deprecated 가 붙어있다고 해당 메서드를 사용 할 수 없다는 것은 아니다.

그럼에도 불구하고 @Deprecated 를 사용하는 이유는 무엇일까?

자바는 `하위 호환성`을 중요하게 여기는데, 과거에 getDate() 를 사용하여 작성된 프로그램이 상위 JDK 버전에서 동작이 안된다면 에러가 발생할 것이다. 따라서, 상위 버전에서 하위 버전에서 사용했던 메서드 등이 Deprecated 되었더라도, 상위 버전에서 정상적으로 동작할 수 있도록 해주며, 상위 버전에서는 더 이상 사용하지 말라는 의미를 담고있다고 보면 된다.

### @SuppressWarnings 

비검사 형변환 경고와 같은 `비검사 경고`를 사용하지 않도록 @SuppressWarnings 를 사용하여 설정할 수 있다.

__중요한 점은, @SuppressWarnings("unchecked") 를 사용할 때, 그 경고를 무시해도 안전한 이유를 항상 주석으로 남겨야한다.__

```java
public String methodA() {
    // @SupressWarnings 를 사용한 이유
    @SuppressWarnings("unchecked")
    // 코드 
}
```

비검사 경고는 `ClassCastException` 을 일으킬 수 있는 가능성을 포함하고 있기 때문에, 가급적 비검사 경고를 제외해야 하며, 방법을 찾지 못할 때에만 @SuppressWarnings("unchecked") 를 사용하는 것이 좋다.

### @Safevarargs

- __@Safevarargs__
	- 자바 7 전에는 제네릭 가변인수 메서드의 작성자가 호출자 쪽에서 발생하는 경고에 대해서 해줄 수 있는 일이 없었다.
	- 따라서, 경고를 그냥 두거나 @SuppressWarnings("unchecked")  를 추가하여 경고를 숨기곤 했다.
	- @SafeVarargs 는 제네릭 타입의 가변인자를 사용할 때 나타나는 경고를 숨길 수 있다.
	- `Effective Java Item32` 에서는 제네릭이나 매개변수화 타입의 varargs 매개변수를 받는 모든 메서드에 `@SafeVarargs` 를 추가하라고 나와있다.

```java
@SafeVarargs
static <T> List<T> flatten(List<? extends T>... lists) {
	List<T> result = new ArrayList<>();
	for (List<? extends T> list : lists) {
		result.addAll(list);
	}
	return result;
}
```

### @FunctioanlInterface

Conceptually, a functional interface has exactly one abstract method. 따라서, FunctionalInteface 를 작성할 때 두 개 이상의 추상 메서드가 정의되지 않도록 `컴파일러가 체킹` 해준다.

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface FunctionalInterface {}
```

아니 근데 왜 ?! RetentionPolicy 가 RUNTIME 으로 되어있을까? 

[StackOverflow. Why does FunctionalInterface have a runtime retetion](https://stackoverflow.com/questions/27121563/why-does-functionalinterface-have-a-runtime-retention) 답변을 참고하면 다음과 같다.

예를 들어 API를 만들고 클래스를 미리 컴파일된 jar로 제공하면 컴파일러에서 더 이상 정보를 사용할 수 없기 때문에 "소스"로는 충분하지 않다.

리플렉션을 사용하여 주석을 찾고 경고도 표시해야 하는 스크립팅 엔진과 같이 런타임에 클래스에 대해 "컴파일"하는 그런 종류의 컴파일러를 지원하려는 경우 "클래스"도 충분하지 않을 것이라고 생각한다.

## Annotation Processing Tool

APT(Annotation Processing Tool) 는 Java 1.5 에서 어노테이션이 추가되었을 때, 같이 추가되고 있던 녀석이다. 어노테이션을 컴파일시에 처리하는 구조이다.

이 녀석에 대한 자료가 너무 부족해서, [oracle docs](https://docs.oracle.com/javase/7/docs/technotes/guides/apt/index.html) 를 보았다.

The apt tool and its associated API contaiined in the `pakcage com.sun.mirror have been deprecated since Java SE 7`. Use the options available in the javac tool and the APIs contained in the packages javax.annotation.processing and javax.lang.model to process annotations.

`The apt tool first runs annotation processors` that can produce new source code and other files. Next, apt can cause compilation of both original and generated source files, thus easing the development cycle.

`JSR 269(Pluggable Annotation Processing API)`, also known as the Language Model API, has two basic pieces: an API that models the Java programming language, and an API for writing annotation processors. This functionality is accessed through new options to the `javac command; by including JSR 269 support`, javac now acts analogously to the apt command in JDK 5.

그리고 이어서 [APT 사용법](https://www.inf.unibz.it/~calvanese/teaching/java-docs/5.0/guide/apt/GettingStarted.html)에 대한 문서를 보았다.

해당 문서에는 아래와 같이 설명이 되어있다.

- 주석 처리기를 작성하려면 다음 네 가지 패키지가 필요합니다.
- com.sun.mirror.apt : 도구와 상호 작용하기 위한 인터페이스
- com.sun.mirror.declaration : 필드, 메소드, 클래스 등의 소스 코드 선언을 모델링하기 위한 인터페이스
- com.sun.mirror.type : 소스 코드에서 찾은 모델 유형에 대한 인터페이스
- com.sun.mirror.util : 방문자를 포함한 유형 및 선언 처리를 위한 다양한 유틸리티

그런데 oracle docs 에서는 com.sun.mirror 안에 있는 패키지가 Java SE 7 부터 Deprecated 되었다고 한다. (pakcage com.sun.mirror have been deprecated since Java SE 7)

즉, 상위 버전에서는 사용하지 않기를 권장하고 있다는 것이다. 그러면 APT 를 대체할 다른 수단이 생겼다는건데 아래에서 배워보자.

## Pluggable Annotation Processing API : JSR269

Java 1.6 부터 추가된 컴파일시에 어노테이션을 처리하기 위한 구조이다. `Lombok` 같은 곳에서 사용되고 있다.

JSR269 는 `Annotation Processor` 를 사용하여 런타임이 아닌 컴파일 중에 어노테이션을 처리한다.
Annotation Processor 는 컴파일러의 플러그인에 해당하므로 플러그인 주석 처리라고도 한다.

lombok 이 컴파일 타임에 자바 코드를 생성하는데, lombok 이 `Annotation Processor`를 이용하여 생성하는 것이다. 또한 IDEA 가 코드를 작성할 때 문법 오류를 표시하는 빨간색 밑줄도 이 기능을 통해 구현된다.

> KAPT(Annotation Processing for Kotlin) 또는 Kotlin 의 컴파일도 이 기능을 통해 이루어진다.

Pluggable Annotation Processing API 의 핵심은 Annotation Processor 로, 일반적으로 추상 클래스인 `javax.annotation.processing.AbstractProcessor` 를 상속받아야 한다. 런타임 주석 RetentionPolicy.RUNTIME 과 달리 주석 프로세서는 컴파일 타임 주석, 즉 Java 코드 컴파일 중에 처리되는 `RetentionPolicy.SOURCE` 주석 유형만 처리한다.

### 사용 방법

플러그인 주석 처리 API의 사용 단계는 다음과 같다.

1. Annotation Processor 를 커스텀하여 정의하려면 `javax.annotation.processing.AbstractProcessor` 를 상속하고 프로세스 메소드를 재정의해야 한다.
2. 커스텀 어노테이션을 만든다. 메타 어노테이션은 `@Retention(RetentionPolicy.SOURCE)` 을 지정해야 한다.
3. 선언된 사용자 정의 어노테이션 프로세서에서 `javax.annotation.processing.SupportedSourceVersion` 을 사용하여 컴파일된 버전을 지정해야 한다.
4. 선언된 사용자 정의 어노테이션 프로세서에서 `javax.annotation.processing.SupportedOptions` 를 사용하여 컴파일 매개변수를 지정할 수 있다.

> [Annotation Processor 를 활용한 @BuilderProperty 만들기 예제](https://www.baeldung.com/java-annotation-processing-builder)

## Referneces

- https://www.nextree.co.kr/p5864/
- https://ahnyezi.github.io/java/javastudy-12-annotation/
- https://catch-me-java.tistory.com/49
- https://www.baeldung.com/google-autoservice
- https://www.inf.unibz.it/~calvanese/teaching/java-docs/5.0/guide/apt/GettingStarted.html
- https://qiita.com/opengl-8080/items/beda51fe4f23750c33e9
- [JSR269 : Pluggable Annotation Processing API](https://jcp.org/en/jsr/detail?id=269)
- http://hannesdorfmann.com/annotation-processing/annotationprocessing101/
- https://pluu.github.io/blog/android/2015/12/24/annotation-processing-api/
- https://programmer.group/pluggable-annotation-processing-api.html
