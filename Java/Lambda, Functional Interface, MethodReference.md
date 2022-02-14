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

### 람다식의 단점
- 람다를 사용하면서 만든 무명함수는 재사용이 불가능하다.
- 디버깅이 어렵다.
- 람다를 남발하면 비슷한 함수가 중복 생성되어 코드가 지저분해질 수 있다.
- 재귀로 만들경우에 부적합하다.


## Functional Interface



---

### Reference

- https://galid1.tistory.com/509
- https://mangkyu.tistory.com/113