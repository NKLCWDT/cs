## 컬렉션 프레임워크(Collection Framework)

자바에서 컬렉션 프레임워크란, 다수의 데이터를 쉽고 효과적으로 처리할 수 있는 표준화된 방법을 제공하는 클래스의 집합을 의미한다. (배열의 단점을 보완해준다)

JDK 1.2 이전까지는 컬렉션 클래스들을 서로 각자 다른 방식으로 처리해야 했으나 JDK 1.2부터 컬렉션 프레임워크가 등장하면서 다양한 종류의 컬렉션 클래스가 추가되고 모든 컬렉션 클래스를 표준화된 방식으로 다룰 수 있도록 체계화 되었다.

이러한 컬렉션 프레임워크는 자바의 인터페이스(interface)를 사용하여 구현된다.

> 컬렉션 (Collection) : 여러 객체(데이터)를 담을 수 있는 자료구조, 다수의 데이터 그룹  
> 프레임워크 (Framework) : 표준화, 정형화된 체계적인 프로그래밍 방식

![IMAGES](../images/collection%20framework.png)

### 컬렉션 프레임워크의 구성요소

- 컬렉션 인터페이스 : java.util 패키지 안에 있음
- 컬렉션 클래스 : 컬렉션 인터페이스를 상속받고 java.util 또는 java.util.concurrent 패키지 안에 있음
- 컬렉션 알고리즘 : 검색, 정렬, 셔플과 같은 기능을 제공

### 컬렉션 인터페이스(Collection Interface)

제네릭(Generics)으로 표현되어 컴파일 시점에서 객체의 타입을 체크하기 때문에 에러를 줄이는데 도움이된다.

__컬렉션 프레임워크의 핵심 인터페이스(List/Set/Map)__

List, Set 인터페이스는 Collection 인터페이스를 상속받기 때문에 두 인터페이스의 공통적인 부분을 Collection 인터페이스에서 정의하고 있다.

반면 Map 인터페이스는 구조상의 차이(Key-Value)로 인해 Collection 인터페이스를 상속받지 않고 별도로 정의된다.

인터페이스를 크게 3가지로 분류 할 수 있다.  
Collection 인터페이스 그룹 / Map 인터페이스 그룹 / 기타 인터페이스 그룹

1. Collection 인터페이스 그룹

1) Collection Interface
직접적인 구현은 제공하지 않으며 모든 컬렉션 클래스가 구현해야하는 메소드를 포함하고있다.

2) List Interface (순서 O, 중복 O)
순서가 있는 컬렉션이며 중복 요소를 포함 할 수 있으며 인덱스로 모든 요소에 접근할 수 있다.  
`ArrayList class`, `LinkedList class`는 List 인터페이스로 구현되었다.

![IMAGES](../images/List%20Interface.png)

3) Set Interface (순서 X, 중복 X)
중복 요소를 포함할 수 없으며 랜덤 액세스를 허용하지 않아 iterator 또는 foreach를 이용하여 요소를 탐색할 수 있다.  
`HashSet class`, `TreeSet class`, `LinkedHashSet class`는 Set 인터페이스로 구현되었다.

![IMAGES](../images/Set%20Interface.png)

4) SortedSet Interface
요소를 오름차순으로 유지하는 Set이다.  
`TreeSet class` 는 SortedSet 인터페이스로 구현 되었다.

5) Queue Interface
Queue 인터페이스는 처리하기 전에 요소를 보유하는 데에 사용된다. 기본 컬렉션 작업 외에도 삽입, 추출 및 검사 작업을 제공한다.

일반적으로 Queue는 요소를 FIFO 방식으로 정렬하며 예외에는 우선순위 큐가 있다.

`PriorityQueue class`는 Queue인터페이스로 구현되었다.

6) Deque Interface
양쪽 끝에 요소 삽입 및 제거를 지원한다. `double ended queue`의 약자이며 데크라고 읽는다.  
`ArrayDeque class`는 Deque 인터페이스로 구현되었다.

2. Map 인터페이스 그룹

1) Map Interface (순서 X, 중복(키 X, 값 O))
Map 인터페이스는 키와 값을 매핑한다. 중복 키가 존재할 수 없으며 각 키는 하나의 값만 매핑 할 수 있다.

`HashMap class`, `TreeMap class`, `LinkedHashMap class`는 Map 인터페이스로 구현되었다.

![IMAGES](../images/Map%20Interface.png)

2) SortedMap Interface
SortedMap 인터페이스는 오름차순의 키 순서로 매핑하는 인터페이스이다.
`TreeMap class`는 SortedMap 인터페이스로 구현되었다.

3. 기타 인터페이스 그룹

1) Iterator Interface
Iterator 인터페이스는 어떤 컬렉션이든 반복적으로 수행하기 위한 메소드를 제공한다.

2) ListIterator Interface
ListIterator 인터페이스는 어느 방향이든 목록을 탐색하고 반복하면서 목록을 정하고, 목록에서 반복자의 현재 위치를 가져올 수 있다.

3) Concurrent Interface
- BlockingQueue 인터페이스
- TransferQueue 인터페이스
- BlockingDeque 인터페이스
- ConcurrentMap 인터페이스
- ConcurrentNavigableMap 인터페이스

### 컬렉션 클래스 (Collection Class)
컬렉션 인터페이스에 대한 구현 클래스 제공

| 컬렉션 | 특징 |
| --- | --- |
| ArrayList | 배열기반, 데이터의 추가와 삭제에 불리, 순차적인 추가/삭제는 제일 빠름, 임의의 요소에 대한 접근성이 뛰어남 |
| Linked List | 연결기반, 데이터의 추가와 삭제에 유리, 임의의 요소에 대한 접근성이 좋지 않다. |
| HashMap | 배열과 연결이 결합된 형태, 추가/삭제/검색/접근성이 모두 뛰어남, 검색에는 최고 성능을 보인다. |
| TreeMap | 연결기반, 정렬과 검색(특히 범위검색)에 적합, 검색 성능은 HashMap 보다 떨어진다. |
| HashSet | 내부적으로 HashMap을 이용해서 구현 |
| TreeSet | 내부적으로 TreeMap을 이용해서 구현 |
| LinkedHashMap | HashMap에 저장순서 유지기능을 추가 |
| LinkedHashSet | HashSet에 저장순서 유지기능을 추가 |

- 일반적으로 쓰이는 클래스
  ArrayList, LinkedList, HashSet, TreeSet, PriorityQueue, ArrayDeque, HashMap, TreeMap, LinkedHashMap

- Concurrent 클래스
  CopyOnWriteArrayList, ConcurrentHashMap, CopyOnWriteArraySet

- Legacy 클래스
  Vector, Stack, Dictionary, Hashtable, Properties

- Abstract 클래스
  AbstractList, AbstractSequenctailList, AbstractSet, AbstractQueue

> __ArrayList와 Vector 차이__  
> 참고 : https://yeolco.tistory.com/94

> __HashMap/HashSet의 원리__  
> 참고 : https://papimon.tistory.com/74

### 컬렉션 알고리즘 (Collection Algorithm)

Collections 클래스는 모든 컬렉션의 알고리즘을 담당한다.

유틸리티 클래스로써 static 메소드로 구성되어 있고 컬렉션들을 컨트롤하는데에 사용된다.

주의할 점은 자바의 Collection은 인터페이스이며, Collections는 클래스라는 점이다.

- 정렬(Sorting) : 정렬 알고리즘은 요소가 오름차순이 되도록 리스트를 재 정렬함
- 셔플링(Shuffling) : 셔플링 알고리즘은 랜덤으로 목록을 재 정렬함 (우연한 게임을 구현할 때 유용)
- 탐색 (Searching) : 이진 검색 알고리즘은 정렬된 목록에서 지정된 요소를 검색


---

### Reference

https://steady-snail.tistory.com/74

https://prinha.tistory.com/entry/JAVA%EC%9E%90%EB%A3%8C%EA%B5%AC%EC%A1%B0-%EC%9E%90%EB%B0%94-%EC%BB%AC%EB%A0%89%EC%85%98-%ED%94%84%EB%A0%88%EC%9E%84%EC%9B%8C%ED%81%ACjava-collection-framework