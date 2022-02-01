# Stream

Stream 은 데이터 컬렉션 반복을 멋지게 처리하는 기능이다.

## Collection vs Stream

Collection 의 주제는 `데이터`이고, Stream 은 `계산`이다. 컬렉션은 Iterator 와 같은 외부 반복자를 직접 이용해야하며, 스트림은 내부 반복자를 사용한다.

## Loop vs Stream

### Stream 의 장점

Stream 이 Loop 에 비해 갖는 장점이 무엇일까? 

내가 생각했을때의 가장 큰 장점 중 하나는 `표현력`이 좋다는 것이다. 표현력이 좋다는 것은 `가독성`이 좋다는 것을 의미한다.

```java
// 내부 반복 사용 : WHAT 중심
Employee topDeveloper = employees.stream()
                .filter(employee -> "Developer".equals(employee.getPosition())) 
                .max(Comparator.comparingInt(Employee::getAnnualIncome)) 
                .get(); 
```

중간 연산자를 통해서 각각 어떠한 작업(What)을 하고 있는지 분명히 밝히고 있다.

> 내부 반복자(Internal Iterator)는 컬렉션 내부에서 요소들을 반복시키고, 개발자는 각 요소당 처리해야하는 코드만 제공하는 것을 말한다.

```java
int max = 0;
Employee topDeveloper;
// 외부 반복 사용 : HOW 중심
for(Employee employee : employees) {
    if("Developer".equals(employee.getPosition())) {
        if(max < employee.getAnnualIncome()) {
            max = employee.getAnnualIncome();
            topDeveloper = employee;
        }
    }
}
```
스트림의 중간 연산자 처럼, 어떠한 작업을 의미하는지 표현해주지 않고 있다. 단순히 "나는 지금 루프를 하고 있어요" 라고 말하는 정도이다. 따라서, 루프의 목적을 파악하는 과정이 스트림에 비해서 힘들다.

> 외부 반복자(External Iterator)는 개발자가 코드로 직접 컬렉션의 요소를 반복해서 가져오는 것을 말한다.
> 
> Employees 리스트에 존재하는 각 Employee 마다, 포지션이 개발자이면 연봉을 max 값과 비교하고 기존 max 값이 더 작다면 갱신시켜주고, topDeveloper 도 갱신시켜준다. (How 중심)

스트림과 견주는 표현력을 갖추기 위해서는 `메서드 추출(Method Extract)` 등을 활용하는 방법 밖에 없다.

- __표현력이 좋다.__
- __stream() 을 parallelStream() 로 바꿈으로써 쉽게 병렬로 실행할 수 있다.__

### Stream 의 단점

- __디버깅이 힘들다__
    - 스트림은 한번에 수행되기 때문에 처음부터 전부 확인해야한다.
- __재사용이 불가능하다__
    - Stream 은 한 번 사용하면 close 되기 때문에 재사용이 불가능하다.

## 특징

- Stream 은 데이터를 담고 있는 저장소가 아니라, `데이터 처리 과정`을 의미한다.
- Stream pipeline 은 크게 `중간 연산`과 `최종 연산`으로 구성된다.
    - __중간 연산(Intermediate Operation)__
        - 중간 연산은 새 스트림을 리턴한다.
        - `Lazy` : 필요할 때만 값을 계산한다. (필요할때 : 최종 연산을 수행하는 시점)
        - 모든 중간 연산은 최종연산을 수행하기 전 까지 `지연(Lazy)`된다.
    - __최종 연산(Terminal Operation)__
        - 스트림이 아닌 다른 타입을 리턴한다.
        - 스트림 파이프라인에서 결과를 도출한다.
- 스트림 파이프라인의 개념은 빌더 패턴(builder pattern)과 비슷하다. builder 패턴은 호출을 연결해서 설정을 만들고 마지막 build() 메서드를 호출해서 닫는다.
- 스트림의 동작 과정
    - 스트림 생성
    - 중간 연산
    - 최종 연산

## Short circuit

Lazy 와 더불어 스트림의 가장 큰 특징 중 하나는 `Short circuit` 이다. Short-circuiting operations 라고도 부른다.

boolean short-circuit evaluations 를 생각하면 된다.

```java
if(isDone() && isFinished() && isTerminated()) {}
```

위 조건식에서 isDone() 의 결과가 FALSE 이면, 뒤에 남아있는 조건들은 무시된다.

예를 들어 여러 and 연산으로 연결된 커다란 불린 표현식을 평가한다고 하면, 표현식에서 하나라도 거짓이라는 결과가 나오면 나머지 표현식의 결과와 상관없이 전체 결과도 거짓이 된다. 이러한 상황을 쇼트서킷이라고 부른다.

`allMatch, noneMatch, findFirst, findAny` 등의 연산은 모든 스트림의 요소를 처리하지 않고도 결과를 반환할 수 있다. 원하는 요소를 찾았으면 즉시 결과를 반환할 수 있다. 마찬가지로 스트림의 모든 요소를 처리할 필요없이 주어진 크기의 스트림을 생성하는 limit도 쇼트서킷 연산이다. 특히 무한한 요소를 가진 스트림을 유한한 크기로 줄일수 있는 유용한 연산이다.

```java
Optional<String> nameFrom = names.stream()
    .filter(s -> s.startsWith("R")).findFirst();
```

> [Java 8 Stream Short circuit](https://javagoal.com/java-8-stream-short-circuit/)

## 스트림 생성하기

```java
@Test
void createStream() throws Exception {
    // Collection 에서 제공하는 Stream 사용
    Stream<Employee> stream1 = employees.stream();

    // Arrays 에서 제공하는 Stream 사용
    Employee[] emps = new Employee[10];
    Stream<Employee> stream2 = Arrays.stream(emps);

    // Stream 클래스의 static 메서드 of 사용
    Employee jungho = new Employee("weave", "Developer", "Java", 80000000);
    Stream<Employee> stream3 = Stream.of(jungho);

    // Infinite Stream 사용
    Stream<Integer> infiniteStream = Stream.iterate(0, i -> i + 2);
    List<Integer> collect = infiniteStream
            .limit(10)
            .collect(Collectors.toList());
    assertEquals(collect, Arrays.asList(0, 2, 4, 6, 8, 10, 12, 14, 16, 18));

    // Stream 의 static 메서드 Generate 사용
    Stream.generate(Math::random).limit(5).forEach(System.out::println);
}
```

## map vs filter

- __map__
    - 특정 값을 `추출`해서 Stream 형식으로 저장
- __filter__
    - 어떠한 조건으로 `필터링`해서 Stream 형식으로 저장

### Loop Fusion

map 과 filter 와 같은 중간 연산자들은 서로 다른 연산이지만, 실제로는 하나의 과정으로 병합되는데, 이러한 특징을 `Loop Fusion` 이라고 한다.

더미 데이터가 아래와 같다.

```java
List<Employee> employees = List.of(
                new Employee("weave", "Developer", "Java", 80000000),
                new Employee("john", "Designer", "UX", 47000000),
                new Employee("rochelle", "Publisher", "Javascript", 25000000),
                new Employee("bill", "Designer", "UI", 50000000),
                new Employee("may", "Publisher", "Javascript", 35000000),
                new Employee("joy", "Developer", "Python", 62000000),
                new Employee("ellis", "Developer", "C", 58000000)
        );
```

그리고 스트림을 사용하여 `연봉이 5천만원 이상인 직원들의 이름`을 리스트 형식으로 받아보자.


```java
@Test
void loopFusion() {
    List<String> employeeNames = employees.stream()
            .filter(employee -> {
                System.out.println("FILTER : " + employee.getName() + " " + Thread.currentThread().getName());
                return employee.getAnnualIncome() >= 50000000;
            })
            .map(employee -> {
                System.out.println("MAP : " + employee.getName() + " " + Thread.currentThread().getName());
                return employee.getName().toUpperCase();
            })
            .collect(Collectors.toList());
}
```

이제 위 코드의 결과가 어떻게 될지 예상해보자.

```
FILTER : weave main
MAP : weave main
FILTER : john main
FILTER : rochelle main
FILTER : bill main
MAP : bill main
FILTER : may main
FILTER : joy main
MAP : joy main
FILTER : ellis main
MAP : ellis main
```

동작이 `filter -> map` and `filter -> map` 이런식으로 반복되는 것을 볼 수 있다. 이런 식의 데이터 처리 플로우를 Loop Fusion 이라고 한다.

## 필터링

## 프레디케이트로 필터링

```java
List<Dish> vegetarianMenu = menu.stream()
    .filter(Dish::isVegetarian)
    .collect(toList())
```

> 프레디케이트 : boolean 을 반환하는 함수

### 고유 요소 필터링

```java
List<Integer> numbers = Arrays.asList(1, 2, 1, 3, 3, 2, 4);
numbers.stream()
    .filter(i -> i % 2 == 0)
    .distinct()
    .forEach(System.out::println)
```

> distinct() 는 객체 내의 `equals` 와 `hashCode` 를 사용하여 중복 제거한다.

## References

- [Modern Java In Action](http://www.kyobobook.co.kr/product/detailViewKor.laf?mallGb=KOR&ejkGb=KOR&barcode=9791162242025)
- https://www.baeldung.com/java-inifinite-streams
- https://www.logicbig.com/tutorials/core-java-tutorial/java-util-stream/short-circuiting.html
- https://12bme.tistory.com/461
