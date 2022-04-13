
## Promotion, Casting

### type 형 변환

연산을 수행하는 코드에서는 같은 타입끼리만 가능하다. 그래서 연산을 수행하기 전에 같은 타입으로 만들어야 하는데 타입 변환을 하는 것을 형변환 이라고 한다.


### Promotion

자동 형변환, 묵시적 형변환 이라고도 부른다.

작은 데이터 타입에서 큰 데이터 타입으로 형 변환

자바는 타입이 서로 일치하지 않는 경우에도 타입끼리 서로 호환성이 있으면 자동으로 변환을 실시한다.  
자동 변환이 가능한 방향

![image](https://user-images.githubusercontent.com/70622731/163184308-9e81352a-0a8a-4bf0-949a-bd09bf36e949.png)

예를 들어 short 타입은 16비트 정수이고, int 타입은 32bit 정수이므로 값이 변하는 문제는 발생하지 않는다.

그렇다면 long 데이터 타입의 메모리 크기는 8byte이고, float 데이터 타입의 메모리 크기는 4byte인데, long 데이터 타입에서 float 데이터 타입으로 자동 형변환이 가능하다. 그 이유는 표현할 수 있는 값의 범위가 float가 더 크기 때문이다.

참고 : [java 데이터타입](https://github.com/NKLCWDT/cs/blob/main/Java/java%20%EB%8D%B0%EC%9D%B4%ED%84%B0%ED%83%80%EC%9E%85.md)

```java
short shortNum = 100;

int intNum = shortNum;                // 확장성에 의한 자동 변환 (short -> int)
System.out.println(intNum);
```

```java
int a = 1.0; // 컴파일 에러
```

### Casting

강제 형변환, 명시적 형변환 이라고도 불린다.

데이터의 손실이 발생하더라도, 형변환을 하고 싶을 때 사용하는 것이 Casting이다.

```java
int a = (int)1.0; //컴파일 에러 해결
```

1.0이 int a 변수에 들어가면 1로 바뀌기 때문에 실제 데이터가 손실되는 것을 방지하기 위해 막고자 컴파일 에러가 나게되는데, 코드를 작성하는 프로그래머가 데이터가 손실된 다는 것을 이미 알고 있어도 이용하고 싶을 때 사용하는 것이 Casting이다.

### 참조형 Casting
- 업 캐스팅

```java
int a = (int)true;
```
boolean 자료형이랑 int 자료형이랑은 서로 성질이 맞지 않는 데이터이므로, Casting을 해도 형변환이 되지 않는다.

참조형 데이터 역시 마찬가지다.

참조형 데이터가 Casting이 되려면 다음 조건중 1개는 만족해야한다.
1. 상속관계가 맺어진 경우
2. 인터페이스로 인해 확장이 된 경우

상속 관계로 예를 들어보면

`class Parent` , `class Child extends Parent` 클래스 가 있다고 가정하자.

Child는 Parent 클래스를 상속 받으므로 Child 클래스가 Parent 클래스보다 가지고 있는 데이터 양이 무조건 많다.

```java
Parent parent = new Child();
```

new Child라는 인스턴스는 변수가 원하는 데이터를 모두 가지고 있으므로 형변환 코드를 붙이지 않아도 가능한 코드이다.

위와 같은 캐스팅을 업캐스팅 이라 부른다.

![image](https://user-images.githubusercontent.com/70622731/163184380-53db73c1-74f8-44c1-95d7-c9560f3fd5cb.png)

Parent에 Child 데이터를 넣으므로 화살표 방향이 위로 가게 되어 업캐스팅이라 부른다.

- 다운 캐스팅

위와 같이 상속 관계에 있을 경우 캐스팅이 가능하지만 아래와 같은 코드는 성립하지 않는다.

```java
Child child = new Parent();
```

왜냐하면, 변수가 원하는 정보를 인스턴스가 모두 가지고 있지 않기 때문이다.

```java
Child child = (Child) new Parent();
```
위 코드는 컴파일 에러가 나지는 않지만 런타임 오류가 발생한다.

컴파일러는 형변환을 해줌으로써 데이터를 맞게 넣어준것처럼 보여 문법이 맞다고 생각하여 넘어가지만

프로그램이 실제로 동작할 때, new Parent() 인스턴스는 Child형 데이터로 바꾸지 못한다는 것을 깨닫고, 런타임 오류를 내며 종료된다.

![image](https://user-images.githubusercontent.com/70622731/163184422-2c6afee7-4fab-4ff1-b6bc-886e2a247813.png)

Child 데이터에 Parent 데이터를 넣는 경우는 화살표가 아래로 향하므로 다운캐스팅 이라 부른다.

위와 같은 예시를 봤을 때 다운캐스팅은 일반적으로 성립하지 않는다. 하지만 다운캐스팅이 성립되는 경우가 존재한다.

```java
Parent parent = new Child();
Child child = (Child) parent;
```

위에 예시는 다운 캐스팅이 성립된다.

parent 변수는 Parent 클래스 형태의 변수지만, 태생이 Child 인스턴스인 데이터를 넣어주었다.

그러한 정보를 가지고 있는 parent 변수를 다시 Child 클래스 형태로 다운캐스팅을 하였으므로 가능한 코드이다.

결과적으로 다운캐스팅은 보통 성립되지 않는 문법이지만, 업캐스팅이 선행된 경우, 다운캐스팅이 성립되는 경우가 존재한다.

<br>


## boxing, unboxing

![image](https://user-images.githubusercontent.com/70622731/163184466-27ae9f2a-89ea-4e3e-a871-517d815d0bb9.png)

박싱(boxing)이란 기본형을 참조형으로 변환하는 것이고 언박싱(unboxing)이란 반대로 참조형을 기본형으로 바꾸는 것이다.

그리고 JDK 1.5 부터는 이것을 자동으로 해주는 기능이 추가되었다.

```java
public class Test {
    public static void main(String[] args) {
        Integer testInteger = new Integer(123);
        int testInt = 123;
        
        int a = (int)testInteger; // 언박싱(unboxing)
        int b = testInteger; // 오토언박싱(auto unboxing)
        int c = (Integer)testInt; // 박싱(boxing)
        int d = testInt; // 오토박싱(auto boxing)
    }
}
```

__왜 Boxing을 지양해야 할까?__

Boxing을 지양해야 하는 이유는 비용이 추가적으로 발생하기 때문이다.

Boxing한 값(Wrapper class)은 reference Type 이므로 Heap영역에 저장된다.

그렇기 때문에, primitive Type을 가져오려고 할 때도 Wrpper class로 인해 메모리를 추가적으로 탐색 해야하는 과정이 필요하기 때문이다.

```java
primitive Type = Stack에서 바로 조회
boxing된 reference Type = Stack에서 조회 + Heap 영역까지 조회
```

위와 같은 이유로 원시타입(Primitive Type)이 성능상 이점을 가져가게 된다.

<br>

---

### Reference

https://league-cat.tistory.com/412

https://mommoo.tistory.com/41?category=577684

https://mommoo.tistory.com/51

https://jamesdreaming.tistory.com/154

https://n1tjrgns.tistory.com/290
