## Immutable(불변)

자바의 객체의 타입에는 Immutable(불변) 타입과 mutable(가변) 타입이 있다.

객체들은 기본적으로 heap영역에 할당되고 stack영역에 래퍼런스 값을 갖는 참조변수들로 접근 가능하다,.

new 연산자로 객체를 생성하면 heap 영역에 객체가 생기고 래퍼런스 값을 가지는 변수가 stack에 생길 것이다. 불변 객체라는 것은 이 객체의 값을 heap영역에서 바꿀 수 없다는 뜻이다.

오직 새 객체를 만들어 래퍼런스 값을 주는 재할당만이 가능하다.

```java
// String, Boolean, Integer, Float, Long 등등이 해당
String str = "test";
str = "test2"; // 재할당
```

<img width="550" alt="image" src="https://user-images.githubusercontent.com/70622731/153700094-17776581-b322-4b75-a076-b381e3890a7d.png">

불변객체들 Integer, String 등의 API를 살펴보면 public final class Integer, public final class String으로 구성되어있다.

fianl을 선언해줌으로써 변경 불가능하게 만들어져 있다.

따라서, Class를 불변객체로 만들고자 한다면
- 맴버변수를 final로 설정해준다.
- setter 메서드를 구현해주지 않으면 된다.
- class를 상속하지 못하도록 선언 (class final로 선언하거나 생성자를 private로 선언)

### 불변객체의 장점

- 객체에 대한 신뢰가 올라간다.
- multi-thread 환경에서 동기화 처리없이 객체를 공유 가능하다.

### 불변객체의 단점

- 객체의 값이 할당될 때 마다 새로운 객체가 필요하다. 따라서 메모리 누수와 성능저하를 발생시킬 수 있다.

## Synchronized

자바에서 프로그래밍을 한다면 Multi-Thread로 인하여 동기화를 제어해야하는 경우가 생긴다. 그 때 자바에서 제공하는 키워드인 Synchronized 키워드를 사용하게 되는데,
Multi-Thread 상태에서 동일한 자원을 동시에 접근하게 되었을 때 동시접근을 막게된다.

즉 공유 데이터에 lock을 걸어서 데이터가 변경되지 않도록 보호함으로써 쓰레드의 동기화(Synchronization)를 가능하게 한다.

Synchronized는 두 가지 형태로 사용가능하다.

1. 메서드에 Synchronized

```java
public synchronized void synchronizedTest () {
    ...
}
```

2. 블록에 Synchronized
```java
synchronized(락(Lock) 객체) {
    ...
}
```

락 객체는 문지기 역할을 해서 오직 하나의 스레드만이 동기화 블록에 접근할 수 있다.

메서드에 Synchronized 붙인 경우는 락 객체를 지정하는 부분이 없고 내부적으로 자신의 객체를 락으로 사용한다.

이에 반해서 블록에 Synchronized는 락 객체를 자기 자신 객체를 의미하는 this를 사용할 수도 있고, 다른 객체를 락으로 사용할 수도 있다.

ex)
```java
public void withDraw(int money) {
    synchronized(this) {
        if(balance >= money) {
            try {
                Thread thread = Thread.currentThread();
                System.out.println(thread.getName() + " money : " + money);
                Thread.sleep(1000);
                balance -= money;
                System.out.println(thread.getName() + " balance : " + balance);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

## Java의 Stack 클래스

```java
public class Stack<E> extends Vector<E> {

    public Stack() {
    }

    public E push(E item) {
        addElement(item);

        return item;
    }

    public synchronized E pop() {
    }

    public synchronized E peek() {
    }

    public synchronized int search(Object o) {
    }
}
```

__Stack 클래스의 단점__

1. Stack 클래스는 synchronized 키워드가 붙어있기 때문에 Thread-safe 하다는 특징을 가지고 있다. 즉, 멀티스레드 환경이 아닐 때 사용하면 lock을 거는 작업 때문에 많은 오버헤드가 발생하게 된다.

2. Stack은 LIFO 구조로 한쪽 방향에서만 삽입, 삭제가 일어나야 하는데 Vector를 상속받아 중간에서 삽입, 삭제를 할 수 있다는 큰 문제가 발생한다. 이는 Stack, Vector가 JDK 1.0 부터 존재하다보니 자바에서 잘못 설계를 한 경우라고 한다. 

3. Stack은 초기 용량을 설정할 수 있는 생성자가 없다보니 데이터의 삽입을 많이 하게되면 배열을 복사해야하는 일이 빈번하게 발생 할 수 있다.

__Stack 클래스 대신 ArrayDeque를 사용하기__

ArrayDeque 공식문서에 보면 스택구조로 사용하면 Stack 클래스보다 빠르고, 큐 구조로 사용하면 Queue 클래스보다 빠르다고 한다.(ArraDeque는 Thread-Safe 하지 않기 때문이다.)

따라서 LIFO 구조를 만들기 위해 적합한 클래스는 Deque 인터페이스를 구현하는 ArrayDeque 클래스이다.

## String, StringBuffer, StringBuilder

### String

String과 StringBuffer/StringBuilder 클래스의 가장 큰 차이점은 String은 불변(immutable)의 속성을 갖는다는 것이다.

String은 불변성을 가지기 때문에 변하지 않는 문자열을 자주 읽어들이는 경우 String을 사용하면 좋은 성능을 기대할 수 있다.

그러나 문자열 추가,수정,삭제 등의 연산이 빈번하게 발생하는 알고리즘에 String 클래스를 사용하면 힙 메모리에 많은 임시 가비지(Gabege)가 생성되어 힙메모리가 부족으로 어플리케이션 성능에 치명적인 영향을 끼치게 된다.

이를 해결하기 위해서 Java에서는 가변(mutable)성을 가지는 StringBuffer/StringBuilder 클래스를 도입하였다.

> java에서 String은 왜 불변일까??  
> 참고 : https://starkying.tistory.com/entry/why-java-string-is-immutable?category=689625

### StringBuffer, StringBuilder

String 과는 반대로 StringBuffer/StringBuilder 는 가변성 가지기 때문에 .append() .delete() 등의 API를 이용하여 동일 객체내에서 문자열을 변경하는 것이 가능하다. 따라서 문자열의 추가,수정,삭제가 빈번하게 발생할 경우라면 String 클래스가 아닌 StringBuffer/StringBuilder를 사용해야 한다.

__StringBuffer StringBuilder 차이점__

그렇다면 동일한 API를 가지고 있는 StringBuffer, StringBuilder의 차이점은 무엇일까?

가장 큰 차이점은 동기화의 유무로써 StringBuffer는 동기화 키워드(내부적으로 메서드에 synchronized가 붙어있다.)를 지원하여 멀티쓰레드 환경에서 안전하다는 점이다. (String도 불변성을 가지기 때문에 마찬가지로 멀티쓰레드 환경에서의 안정성을 가지고 있다.)

반대로 StringBuilder는 동기화를 지원하지 않기 때문에 멀티쓰레드 환경에서 사용하는것은 적합하지 않지만 동기화를 고려하지 않는만큼 단일쓰레드에서의 성능은 StringBuffer보다 뛰어나다.

### 어느때 사용하는게 효율적일까??

- String : 문자열 연산이 적고 멀티쓰레드 환경일 경우
- StringBuffer : 문자열 연산이 많고 멀티쓰레드 환경일 경우
- StringBuilder : 문자열 연산이 많고 단일쓰레드이거나 동기화를 고려하지 않아도 되는경우

<img width="456" alt="image" src="https://user-images.githubusercontent.com/70622731/153701580-f6305d0f-d64d-4714-9bbd-ab432c3b3e72.png">

> String pool, String 생성시 new를 안쓰는이유  
> 참고 : https://dololak.tistory.com/718  
> 참고 : https://starkying.tistory.com/entry/what-is-java-string-pool

---

### Reference

- https://cantcoding.tistory.com/41
- https://it-mesung.tistory.com/83
- https://ifuwanna.tistory.com/221
- https://ktko.tistory.com/entry/%EC%9E%90%EB%B0%94-synchronized%EC%97%90-%EB%8C%80%ED%95%98%EC%97%AC
- https://devlog-wjdrbs96.tistory.com/244

