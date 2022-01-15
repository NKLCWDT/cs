## JVM (Java Virtual Machine)
Java 프로그램이 플랫폼에 의존하지 않고, 어디서든 동작 가능하도록 하기 위한 Java 가상머신이다.

C/C++ 언어는 CPU 아키텍처, 운영체제 등 플랫폼 환경에 의존성을 가지기 때문에, 플랫폼이 바뀌면 제대로 동작하지 않는 문제가 있다. (크로스 컴파일을 통해, 타겟 플랫폼에 맞춰서 컴파일 해줘야함)

Java의 경우 이러한 문제를 해결하기 위해 JVM을 만들었다.

### JVM의 특징
__스택 기반의 가상머신__
- 대표적인 컴퓨터 아키텍처인 인텔 x86아키텍처, ARM 아키텍처와 같은 하드웨어가 레지스터 기반으로 동작하는 데 비해 JVM은 스택 기반으로 동작한다.

__심볼릭 래퍼런스__
- 기본 자료형(primity data type)을 제외한 모든 타입(클래스와 인터페이스)을 명시적인 메모리 주소 기반의 레퍼런스가 아니라 심볼릭 레퍼런스를 통해 참조한다.

__가비지 컬렉션__
- 클래스 인스턴스는 사용자 코드에 의해 명시적으로 생성되고 가비지 컬렉션에 의해 자동으로 파괴된다.

__기본 자료형을 명확하게 정의하여 플랫폼 독립성 보장__
- C/C++등의 전통적인 언어는 플랫폼에 따라 int형의 크기가 변한다. JVM은 기본 자료형을 명확하게 정의하여 호환성을 유지하고 플랫폼 독립성을 보장



## JVM 아키텍처
사진

### Class Loader
Java 바이트 코드(.class)를 Runtime Memory Area에 적재하는 역할을 함

### Runtime Memory Area
JVM 메모리를 Runtime Memory Area라고 부른다.

사진

__Method Area__  
- class data와 static 변수가 저장되는 공간  
- 모든 스레드가 공유하는 공간  
- JVM이 실행될 때 생성됨

__Heap Area__  
- new 연산자를 통해 동적으로 생성되는 객체가 저장되는 공간
- Heap에 저장된 데이터는 메모리 관리가 필요한 GC 대상
- 모든 스레드가 공유하는 공간
- JVM이 실행될 때 생성됨

__Stack__  
- 프레임(Frame)이 저장되는 공간  
    - Frame : 메소드 데이터가 저장되는공간 (메소드 파라미터, 지역변수, 참조 주소값 등)
- 다른 스레드와 공유하지 않고 각각의 스레드마다 가지는 공간
- 프레임은 메소드(method)가 실행될 때, Stack에 push 하여 추가된다.
- 반대로, 메소드가 종료되면 Stack에서 pop 되어 제거된다.

__PC Register__  
JVM이 현재 실행한 명령어의 주소를 저장

__Native Method Area__  
Java 바이트코드가 아닌 C/C++ 언어가 실행되기 위해 필요한 공간

### Execution Engine
Method Area의 바이트 코드를 Execution Engine에 제공하여 바이트코드를 실행한다.

Execution Engine이 실행하는 두 가지 방식이 존재한다.

__Interpreter__ 방식
- 인터프리터 방식은 바이트코드를 한줄씩 해석하고 실행한다. 속도가 느리다는 단점이 있다.

__JIT(Just In Time)__
- 기존 인터프리터 방식의 느리다는 단점을 극복하기 위한 방법
- 컴파일 방식과 인터프리터 방식을 혼합한 방식을 이용함

### Runtime Data Area

---

### Reference
https://memostack.tistory.com/226

https://happy-coding-day.tistory.com/123