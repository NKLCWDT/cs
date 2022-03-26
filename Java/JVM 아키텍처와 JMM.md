## JVM (Java Virtual Machine)
Java 프로그램이 플랫폼에 의존하지 않고, 어디서든 동작 가능하도록 하기 위한 Java 가상머신이다.

C/C++ 언어는 CPU 아키텍처, 운영체제 등 플랫폼 환경에 의존성을 가지기 때문에, 플랫폼이 바뀌면 제대로 동작하지 않는 문제가 있다. (크로스 컴파일을 통해, 타겟 플랫폼에 맞춰서 컴파일 해줘야함)

> 크로스 컴파일  
> 참고 : https://kkhipp.tistory.com/160

Java의 경우 이러한 문제를 해결하기 위해 JVM을 만들었다.

![image](https://user-images.githubusercontent.com/70622731/160238400-cac9e118-138a-405d-81ee-feed44c3cb9a.png)

### JVM의 특징
__스택 기반의 가상머신__
- 대표적인 컴퓨터 아키텍처인 인텔 x86아키텍처, ARM 아키텍처와 같은 하드웨어가 레지스터 기반으로 동작하는 데 비해 JVM은 스택 기반으로 동작한다.

__심볼릭 레퍼런스__
- 기본 자료형(primity data type)을 제외한 모든 타입(클래스와 인터페이스)을 명시적인 메모리 주소 기반의 레퍼런스가 아니라 심볼릭 레퍼런스를 통해 참조한다.

__가비지 컬렉션__
- 클래스 인스턴스는 사용자 코드에 의해 명시적으로 생성되고 가비지 컬렉션에 의해 자동으로 파괴된다.

__기본 자료형을 명확하게 정의하여 플랫폼 독립성 보장__
- C/C++등의 전통적인 언어는 플랫폼에 따라 int형의 크기가 변한다. JVM은 기본 자료형을 명확하게 정의하여 호환성을 유지하고 플랫폼 독립성을 보장

> 심볼릭 레퍼런스  
> 참고하는 클래스의 특정 메모리 주소를 참조관계로 구성한 것이 아니라,
> 참조하는 대상의 이름만을 지칭한 것이다. Class 파일이 JVM에 올라가게 되면 
> 심볼릭 레퍼런스는 그 이름에 맞는 객체의 주소를 찾아서 연결하는 작업을 수행한다. 그러므로 실제 메모리 주소가 아니라 이름만을 가진다.

### JDK(Java Development Kit) vs JRE(Java Runtime Environment)

![image](https://user-images.githubusercontent.com/70622731/149750883-787d64f6-84db-4bd0-aff6-620825089de3.png)

JVM, JRE, JDK는 3대 자바 프로그래밍의 기술 패캐지로 불린다.  
JDK는 자바 기반의 소프트웨어를 개발하기 위한 도구이며, JRE는 자바 코드를 실행하기 위한 환경으로 JVM을 포함하고 있다. JDK는 JRE를 가지고 있고 컴파일러도 가지고 있다.

<br>

## JVM 아키텍처

![image](https://user-images.githubusercontent.com/70622731/149751223-bc8f19b7-c64b-4681-b8e0-af739582cbd0.png)

클래스로더(Class Loader)가 컴파일된 자바 바이트 코드를 메모리영역에 로드하고, 실행엔진(Execution Engine)이 자바 바이트코드를 실행한다.

> 바이트코드(bytecode)  
> 특정 하드웨어가 아닌 가상 컴퓨터(Virtual Machine)에서 돌아가는 실행 프로그램을 위한 이진 표현법으로 하드웨어가 아닌 소프트웨어에 의해 처리되기에 기계어보다 추상적이다.

### 1. Class Loader
Java 바이트 코드(.class)를 Runtime Memory Area에 적재하는 역할을 함

자바는 런타임에 클래스를 처음으로 참조할 때 해당 클래스를 로드하고 링크하는 특징이 있다.

이 동적 로드를 담당하는 부분이 JVM의 클래스 로더이다.  

클래스 로더가 아직 로드되지 않은 클래스를 찾으면, 다음 그림과 같은 과정을 거쳐 클래스를 로드하고 링크하고 초기화한다.

![image](https://user-images.githubusercontent.com/70622731/149752948-2df80575-4249-44d9-b635-e447a6c68384.png)

- 로딩 : 클래스를 읽어오는 과정
- 링크 : 레퍼런스 연결과정
- 초기화 : static 값 초기화 하고 변수 할당하는 과정

__1) Loading__  

- 클래스 로더가 .class 파일을 읽고, 내용에 맞는 binary 데이터를 생성한 뒤 메모리의 Method 영역에 저장한다.
  - FQCN (Fully Qualified Class Name) 클래스가 속한 패키지명을 모두 포함한 이름
  - 클래스, 인터페이스, Enum
  - 각 클래스/인터페이스의 메소드, 변수

클래스 로더의 종류는 세가지가 있다.

- Bootstrap ClassLoader
  - `JAVA_HOME\lib` 에 있는 코어 자바 API를 제공한다.
- Extension Class Loader
  - `JAVA_HOME\lib\ext` 폴더 또는 java.ext.dirs 시스템 변수에 해당하는 위치에 있는 클래스를 읽는다.
- Application Class Loader (System Class Loader 라도고 부른다.)
  - 애플리케이션 클래스 패스 (애플리케이션 실행할 때 주는 `-classpath` 옵션 또는 `java.class.path` 환경 변수에 값에 해당하는 위치)에서 클래스를 읽는다.

![image](https://user-images.githubusercontent.com/70622731/149753059-4f6edbaf-8178-4197-a455-a94851f8b000.png)

```java
public class App {
  public static void main(String[] args) {
    ClassLoader classLoader = App.class.getClassLoader();
    classLoader // Application ClassLoader
    classLoader.getParent() // Extension ClassLoader
    classLoader.getParent().getParent() // null Bootstrap CLassLoader는 native로 구현되어 있어 자바에서 확인이 불가능하다.
  }
}
```

클래스를 찾을 때 Application 부터 Bootstrap 순서로 위임하듯이 클래스를 찾게된다.


__2) Linking__  
- 검증(Verifying)
  - 읽어들인 클래스가 자바 언어 명세 및 JVM 명세에 명시된 대로 잘 구성되어 있는지 검사한다.
- 준비(Preparing)
  - 클래스가 필요로 하는 메모리를 할당하고, 클래스에서 정의된 필드, 메서드, 인터페이스들을 나타내는 데이터 구조를 준비한다.
- 분석(Resolving)
  - 클래스의 상수 풀 내 모든 심볼릭 레퍼런스를 다이렉트 레퍼런스로 변경한다.

`Book book = new Book()` 이라는 코드가 있을 때  
book이라는 참조변수가 Heap에 저장된 실제 Book 클래스를 가리킬 수 있도록 연결하는게 Resolve 과정이다.

__3) Initializing__

클래스 변수들을 적절한 값으로 초기화 한다. 즉, static initializer들을 수행하고, static 필드들을 설정된 값으로 초기화 한다.

### 2. Runtime Memory Area
JVM 메모리를 Runtime Memory Area라고 부른다.

__1) Method Area__  
- Class Area, Code Area, Static Area 등으로 불리는 해당 영역은 코드에서 사용되는 클래스 파일을 클래스로더로 읽어 클래스별로 런타임 상수풀, 필드데이터, 메소드 데이터, 메소드 코드, 생성자 코드 등을 분류해서 저장한다.
- 모든 스레드가 공유하는 공간  
- JVM이 실행될 때 생성됨

> Runtime Constant Pool (런타임 상수 풀)  
> 상수 풀은 말 그대로 상수를 저장하는 공간이다. 이외에도 필드나 메소드 등의 Reference 값들을 저장하고 있고 실행중에 중복되는 정보가 필요할 때에 기존의 정보를 사용하도록 도와준다.

__2) Heap Area__  
- new 연산자를 통해 동적으로 생성되는 객체가 저장되는 공간
- Heap에 저장된 데이터는 메모리 관리가 필요한 GC 대상
- 모든 스레드가 공유하는 공간
- JVM이 실행될 때 생성됨

__3) Stack__  
- 프레임(Frame)이 저장되는 공간  
    - Frame : 메소드 데이터가 저장되는공간 (메소드 파라미터, 지역변수, 참조 주소값 등)
- 다른 스레드와 공유하지 않고 각각의 스레드마다 가지는 공간
- 프레임은 메소드(method)가 실행될 때, Stack에 push 하여 추가된다.
- 반대로, 메소드가 종료되면 Stack에서 pop 되어 제거된다.

__4) PC Register__

쓰레드가 생성될 때마다 생성되는 영역으로 현재 쓰레드가 실행되는 부분의 주소와 명령을 저장하고 있는 영역이다. 이것을 이용해 쓰레드를 돌아가며 수행할 수 있게한다.

__5) Native Method Stack__

자바 스레드와 네이티브 코드(C, C++)로 작성된 코드 사이를 매핑하는 역할을 한다.

이 메소드는 네이티브 라이브러리(Native Library)와 연결된다. 이 때의 라이브러리는 자바로 쓰여진 것이 아니라 C와 같은 다른 언어로 작성된 것이다.

이러한 라이브러리는 레거시 데이터 혹은 성능을 위해서 사용되었는데 자바의 라이브러리도 발전을 하면서 점차 쓰이지 않게 되었다.

이러한 네이티브 코드는 프로그래머가 JNI(Java Native Interface)를 통해서 호출할 수 있다.

### 3. Execution Engine
클래스 로더에 의해 메모리에 적재된 클래스(Bytecodes)들을 기계어로 변경해 명령어 단위로 실행하는 역할을 한다.

Execution Engine이 실행하는 두 가지 방식이 존재한다.

__1) Interpreter__
- 인터프리터 방식은 바이트코드를 한줄씩 해석하고 실행한다. 속도가 느리다는 단점이 있다.

__2) [JIT(Just In Time)](https://github.com/NKLCWDT/cs/blob/main/Java/%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%A8%20%EC%8B%A4%ED%96%89%20%EA%B3%BC%EC%A0%95.md)__
- 기존 인터프리터 방식의 느리다는 단점을 극복하기 위한 방법
- 컴파일 방식과 인터프리터 방식을 혼합한 방식을 이용함

그리고 더 이상 참조되지 않는 객체를 모아서 정리하는 [GC(Garbage Collector)](https://github.com/NKLCWDT/cs/blob/main/Java/GarbageCollection.md) 가 있다.

<br>

## JMM (Java Memory Model)

JVM이 메모리에 어떻게 상주하는지, 다른 소프트웨어와 마찬가지로 JVM은 호스트 OS 메모리에서 사용 가능한 공간을 사용한다.

그러나 JVM 내부에는 런타임 데이터와 컴파일된 코드를 저장하기 위해 별도의 메모리 공간(Heap, Non-Heap, Cache)이 존재한다.

__1. Heap Memory__  

기본적으로 Heap Memory는 JVM이 시작될 때 공간을 할당받고, 애플리케이션이 작동 중일 때 사이즈에 있어 자유롭다. 즉, new 연산자를 활용해서 객체를 생성할 때 할당된다.

![image](https://user-images.githubusercontent.com/70622731/160238436-ecf28246-1045-4a0f-bd86-8e3e544a0dde.png)

__1) Young Generation__

Young Generation은 비교적 최근에 할당된 객체들을 가지고 있다.  

새로 생성된 객체는 Eden 공간으로 가고 Eden 공간이 객체들로 가득 찼을 때는 minor GC가 수행되며, Survivor(살아남은) 객체들은 Survivor 공간으로 이동한다. 
Minor GC는 survivor 객체들을 확인하고, 이들을 survivor 공간으로 이동시키는데 이 때 한개의 survivor 공간은 항상 비어있어야 한다.

여기서 말하는 survivor 객체들이라고 한다면, 계속해서 참조가 되고 있는 객체이다.

__2) Old Generation (= Tenured space)__

Old Generation은 Minor GC로 부터 살아남고, 오랫동안 사용될 예정의 객체들이다. 만약 Old Generation마저 가득 차게 된다면 Major GC가 작동해서, 최근에 사용되지 않은 객체들부터 가지고 간다.

__2. Non - Heap Memory (힙이 아닌 영역)__

![image](https://user-images.githubusercontent.com/70622731/160238448-2730498f-deaa-4df4-8ff5-ee74a06f3479.png)

힙이 아닌 영역으로써, 이는 모든 스레드가 공유하는 메서드 영역이다.

Non - Heap 영역은 Permanent Generation를 포함하고 있다. JRE에 포함되어 있지만, Heap 영역과 별도로 메모리에 존재한다.

![image](https://user-images.githubusercontent.com/70622731/160238471-eb6c63b6-c759-4293-839e-ff293cf72b7f.png)

JVM 벤더마다 다르지만, HotSpot에선 Method Area를 Permanent Generation이라고 부른다. Java8 부터는 HotSpot에서 JRockit과 일치시키는 과정으로 Perm 영역이 Native 영역으로 이동하여 Metaspace로 변경되었다. 기존 Perm 영역에 존재하던 Static Object는 Heap 영역으로 옮겨져서 GC의 대상이 최대한 될 수 있도록 하였다.

변경된 사항을 표로 정리하면

| | Java7 | Java8 |
| --- | --- | --- |
| Class 메타 데이터 | 저장 | 저장 |
| Method 메타 데이터 | 저장 | 저장 |
| Static Object 변수, 상수 | 저장 | Heap 영역으로 이동 |
| 메모리 튜닝 | Heap, Perm 영역 튜닝 | Heap 튜닝, Native 영역은 OS가 동적 조정 |

__Perm이 제거됐고 Metaspace 영역이 추가된 이유?__

Heap영역은 JVM에 의해 관리된 영역이며, Native 메모리는 OS레벨에서 관리하는 영역이므로 개발자는 영역 확보의 상한을 크게 의식할 필요가 없어지게 되는 장점이 있어 변경되었다.

__Native Memory 구조?__

> Native Memory는 Native Method Stack + C heap 으로 이루어진 것 같습니다.   
> (정확한 내용을 못 찾아 개인적인 생각이 들어있습니다.)  
> c heap 이란? : https://www.thegeekdiary.com/difference-between-the-java-heap-and-native-c-heap/  
> native memory 또는 c heap 이란? : https://stackoverflow.com/questions/32667299/what-is-native-memory-or-c-heap




__3. Cache Memory__

컴파일러에 의해 컴파일된 기계어가 저장된다.

<br>

---

### Reference
https://memostack.tistory.com/226

https://happy-coding-day.tistory.com/123

https://velog.io/@litien/JVM-%EA%B5%AC%EC%A1%B0

https://catsbi.oopy.io/df0df290-9188-45c1-b056-b8fe032d88ca

https://d2.naver.com/helloworld/1230

https://getchan.github.io/til/java_memory_model/

https://yeon-kr.tistory.com/114

https://sharplee7.tistory.com/54

https://inspirit941.tistory.com/296
