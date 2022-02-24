## Lambda Expression

java 8부터 추가된 Lambda Expression(람다식)이란 함수를 하나의 식(expression)으로 표현한 것이다.

- 람다식 이전
```java
public interface Goods {
    public void doSome();
}

public class Computer implements Goods {
    @Override
    public void doSome() {
        System.out.println("do Operation!");
    }
    
    public class Main {
        public static void main(String[] args) {
            Goods com = new Computer();
            com.doSome();
        }
    }
}
```

기존 자바에서 interface를 이용해 다형성을 제공하기 위해서는 interface를 만들고, 그것을 구현한 class를 작성한 뒤, 사용시에는 interface 타입의 참조변수에 interface를 구현한 class의 객체를 생성하여 사용했다.

```java
public class Main {
    public static void main(String[] args) {
        Goods com = new Goods() {
            public void doSome() {
                System.out.println("do Operation!");
            }
        };
        com.doSome();
    }
}
```

또는 위와 같이 실행하는 쪽에서 익명객체를 만들어 사용했다. 하지만 Goods를 구현한 객체가 자주 사용되어야 한다면, 위의 코드를 계속해서 반복해서 사용하게되면서 지저분한 코드가 될 것이다.

- 람다식 사용
```java
public class Main {
    public static void main(String[] args) {
        Goods com = () -> System.out.println("do Operation!");
        com.doSome();
    }
}
```

람다식도 익명객체이므로 람다식으로 대체가능하다.

이런식으로 람다식을 사용하게 되면 이전보다 훨씬 간단하게 표현할 수 있다.

### 람다식 표현법

람다식은 `매개변수 + 실행문`으로 구성된다. 즉 접근자, 반환형 모두 생략되는 구조이다.

모양은 () -> {}; 형태로 구성된다. 첫 괄호는 해당 interface 함수의 매개변수를 입력하면 된다. 그 다음 -> 를 입력하고, {} 안에 실행할 코드를 작성하면 된다.

```java
int max(int a, int b) {
    return a > b ? a : b;
}
```

위에 코드를 람다식으로 변환하면

1. (매개변수 타입) -> {};
```java
(int a, int b) -> {return a > b ? a : b;}
```

2. 리턴 생략
```java
(int a, int b) -> a > b ? a : b
```

3. 매개변수 타입 생략
```java
(a, b) -> a > b ? a : b
```

### 람다식의 사용조건

람다식은 인터페이스를 구현할 때 익명객체를 대신해 사용가능하고 인터페이스가 함수형 인터페이스(Functional Interface)인 경우에만 사용이 가능하다. 

함수형 인터페이스란 인터페이스가 단 한개의 추상 메서드를 정의하고 있는 인터페이스를 말한다. 이는 두개의 추상메소드를 가지고 있다면, 람다식을 사용할 수 없다는 말이다. 왜냐하면 구현해야할 인터페이스의 메소드가 2개 이상인 경우 어떤 메소드를 람다식으로 표현했는지 알 수 없기 때문이다.

이때 `@FunctionalInterface` 어노테이션을 인터페이스에 사용하면 해당 인터페이스는 하나의 메소드만 정의 할 수 있으며 두개 이상의 메소드를 정의하면 인터페이스에서 에러를 발생시킨다.

### 람다식의 특징
- 람다식 내에서 사용되는 지역변수는 final이 붙지 않아도 상수로 간주된다.
- 람다식으로 선언된 변수명은 다른 변수명과 중복될 수 없다.

### 람다식의 장점
- 코드를 간결하게 만들 수 있다.
- 식에 개발자의 의도가 명확히 드러나 가독성이 높아진다.
- 함수를 만드는 과정없이 한번에 처리할 수 있어 생산성이 높아진다.
- 병렬프로그래밍이 용이하다.
    - iteration방식은 반복대상을 일일히 루프에서 지정하는 반면에, 함수형 프로그래밍에서는 반복대상을 사용자코드에서 직접 지정하지 않는다. (이로 인해 Collection API가 크게 효과적으로 개선되었다.)
    - 람다기반 비동기식 병렬 프로그래밍 기법인 CompletableFuture 존재

### 람다식의 단점
- 람다를 사용하면서 만든 무명함수는 재사용이 불가능하다.
- 디버깅이 어렵다.
- 람다를 남발하면 비슷한 함수가 중복 생성되어 코드가 지저분해질 수 있다.
- 재귀로 만들경우에 부적합하다.

<br>

## Functional Interface

함수형 인터페이스(Functional Interface)는 1개의 추상 메소드를 갖고있는 인터페이스를 말한다.

```java
public interface FunctionalInterface {
    public abstract void doSomething(String text);
}
```

__함수형 인터페이스를 사용하는 이유?__

함수형 인터페이스를 사용하는 이유는 자바의 람다식은 함수형 인터페이스로만 접근이 되기 때문이다.

```java
public interface FunctionalInterface {
    public abstract void doSomething(String text);
}
FunctionalInterface func = text -> System.out.println(text);
func.doSomething("do something");
```

함수형 인터페이스를 사용하는 것은 람다식으로 만든 객체에 접근하기 위해서이다. 위의 예제처럼 람다식을 사용할 때마다 함수형 인터페이스를 매번 정의하기에는 불편하기 때문에 자바에서 라이브러리로 제공하는 것들이 존재한다.

### 기본 함수형 인터페이스

- Runnable
- Supplier
- Consumer
- Function
- predicate

### Runnable

Runnable은 인자를 받지않고 리턴값도 없는 인터페이스이다.

```java
public interface Runnable {
    public abstract void run();
}
```

아래 코드처럼 사용할 수 있다.

```java
Runnable runnable = () -> System.out.println("run anything!");
runnable.run();
// 결과
// run anything!
```

Runnable은 `run()`을 호출해야한다. 함수형 인터페이스마다 `run()`과 같은 실행 메소드 이름이 다르다. 인터페이스 종류마다 만들어진 목적이 다르고, 그 목적에 맞는 이름을 실행 메소드 이름으로 정하였기 때문이다.

### Supplier

Supplier<T>는 인자를 받지않고 T 타입의 객체를 리턴한다.

```java
public interface Supplier<T> {
    T get();
}
```

아래 코드처럼 사용할 수 있다.

```java
Supplier<String> getString = () -> "Happy new year!";
String str = getString.get();
System.out.println(str);
// 결과
// Happy new year!
```

### Consumer

Consumer<T>는 T타입의 객체를 인자로 받고 리턴 값은 없다.

```java
public interface Consumer<T> {
    void accept(T t);

    default Consumer<T> andThen(Consumer<? super T> after) {
        Objects.requireNonNull(after);
        return (T t) -> { accept(t); after.accept(t); };
    }
}
```

아래 코드처럼 사용할 수 있다. `accept()`메소드를 사용한다.

```java
Consumer<String> printString = text -> System.out.println("Miss " + text + "?");
printString.accept("me");
// 결과
// Miss me?
```

또한, `andThen()`을 사용하면 두개 이상의 Consumer를 연속적으로 실행 할 수 있다.
```java
Consumer<String> printString = text -> System.out.println("Miss " + text + "?");
Consumer<String> printString2 = text -> System.out.println("--> Yes");
printString.andThen(printString2).accept("me");
// 결과
// Miss me?
// --> Yes
```

### Function

Function<T,R>는 T타입의 인자를 받고, R타입의 객체를 리턴한다.

```java
public interface Function<T, R> {
    R apply(T t);

    default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
        Objects.requireNonNull(before);
        return (V v) -> apply(before.apply(v));
    }

    default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t) -> after.apply(apply(t));
    }

    static <T> Function<T, T> identity() {
        return t -> t;
    }
}
```

다음과 같이 사용할 수 있다. `apply()` 메소드를 사용한다.

```java
Function<Integer, Integer> multiply = (value) -> value * 2;
Integer result = multiply.apply(3);
System.out.println(result);
// 결과
// 6
```

### Predicate

Predicate<T>는 T타입 인자를 받고 결과로 boolean을 리턴한다.

```java
public interface Predicate<T> {
    boolean test(T t);

    default Predicate<T> and(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) && other.test(t);
    }

    default Predicate<T> negate() {
        return (t) -> !test(t);
    }

    default Predicate<T> or(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) || other.test(t);
    }

    static <T> Predicate<T> isEqual(Object targetRef) {
        return (null == targetRef)
                ? Objects::isNull
                : object -> targetRef.equals(object);
    }
}
```

다음과 같이 사용할 수 있다. `test()` 메소드를 사용한다.

```java
Predicate<Integer> isBiggerThanFive = num -> num > 5;
System.out.println("10 is bigger than 5? -> " + isBiggerThanFive.test(10));
// 결과
// 10 is bigger than 5? -> true
```

<br>

## Method Reference

메소드 래퍼런스(Method Reference)는 Lambda 표현식을 더 간단하게 표현하는 방법이다.

```java
Function<String, Integer> f = (String s) -> Integer.parseInt(s);
```

위의 람다식을 메서드 참조를 통해 더 간단하게 만들 수 있다.

```java
Function<String, Integer> f = Integer::parseInt;
```

위 메소드 래퍼런스에서 람다식의 일부가 생략되었지만, 컴파일러는 생략된 부분을 좌변의 Function인터페이스에 지정된 제네릭 타입으로부터 쉽게 알아낼 수 있다.

메소드 래퍼런스를 표현하는 3가지 방법이 있다.

1. 정적 메서드 래퍼런스

    파라미터로 전달받은 변수의 메서드를 사용하지 않고, 정적 메서드의 인자로 사용됨 (String x) -> Integer.parseInt(x) 의 경우 파라미터 x를 parseInt의 인자로 사용되는 것을 볼 수 있다.

2. 다양한 형식의 인스턴스 메서드 래퍼런스

    파라미터로 전달받은 변수의 메서드를 사용함 (Instant x) -> x.toEpochMilli()의 경우 파라미터 x를 받아서 x 자신의 메서드 (toEpochMilli())를 수행한다.

3. 기존 객체의 인스턴스 메서드 래퍼런스

    1번과 비슷한데 차이점이라면 정적 메서드의 인자가 아닌 기존에 이미 생성된 인스턴스의 인자로 사용된다. (Integer가 아닌 member같은)

<img width="1013" alt="image" src="https://user-images.githubusercontent.com/70622731/154053223-68f1b034-b2e3-477b-928e-89b8e7beead6.png">

하나의 메서드만 호출하는 람다식은 `클래스이름::메서드이름` 또는 `참조변수::메서드이름`으로 바꿀 수 있다.

<img width="654" alt="image" src="https://user-images.githubusercontent.com/70622731/154054684-9c07cd89-78ca-422c-84ac-924ffe203b23.png">


---

### Reference

- [자바의 정석](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=76083001&partner=googlek&BSCPN=ORM&BSPRG=ADWORDS&BSCCN1=89221&utm_source=ADWORDS&utm_medium=cpc&utm_term=JAVA%C0%C7%C1%A4%BC%AE&gclid=Cj0KCQiAu62QBhC7ARIsALXijXQ_zzNL5DNVg_7CtLhLBBEmWHEYNeuSDyfs4bo_e_VfuRXWFElnEoIaAgM-EALw_wcB)
- [모던 자바 인 액션](http://www.yes24.com/Product/Goods/77125987)
- https://galid1.tistory.com/509
- https://mangkyu.tistory.com/113
- https://codechacha.com/ko/java8-functional-interface/
- https://codechacha.com/ko/java8-method-reference/
