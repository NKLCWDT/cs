# Thread

Thread 를 배우기 전에, Process 와 Thread 의 개념에 대해서 아래의 링크를 통해서 먼저 공부하고 오면 좋다. 왜냐하면 이번 글에서는 프로세스, 쓰레드, 멀티 프로세스, 멀티 쓰레드에 대한 개념을 숙지했다고 가정하고 설명할 것이기 때문이다.

> Thread 면접 포인트는 아래 링크 안에서 많이 나오지 않을까 생각한다. 만약에, 회사에서 Thread 를 직접 생성해서 비동기로 처리해야할 일이 있다고하면, `모던 자바 인 액션` 책을 사서 보는 것을 추천한다.

- [OS : Process and Thread](https://github.com/NKLCWDT/cs/blob/main/Operating%20System/%ED%94%84%EB%A1%9C%EC%84%B8%EC%8A%A4%EC%99%80%20%EC%93%B0%EB%A0%88%EB%93%9C.md)
    - [동시성 이슈 : 공유 객체를 사용할 때의 주의점](https://github.com/NKLCWDT/cs/blob/main/Operating%20System/%ED%94%84%EB%A1%9C%EC%84%B8%EC%8A%A4%EC%99%80%20%EC%93%B0%EB%A0%88%EB%93%9C.md#%EB%8F%99%EC%8B%9C%EC%84%B1-%EC%9D%B4%EC%8A%88)
    - [ThreadLocal](https://github.com/NKLCWDT/cs/blob/main/Operating%20System/%ED%94%84%EB%A1%9C%EC%84%B8%EC%8A%A4%EC%99%80%20%EC%93%B0%EB%A0%88%EB%93%9C.md#threadlocal)
- [Scheduling Algorithms](https://github.com/NKLCWDT/cs/blob/main/Operating%20System/CPU%20%EC%8A%A4%EC%BC%80%EC%A4%84%EB%A7%81.md#%EC%8A%A4%EC%BC%80%EC%A4%84%EB%A7%81-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98)
- [동기 vs 비동기, 블로킹 vs 논블로킹](https://github.com/NKLCWDT/cs/blob/main/Operating%20System/%EB%8F%99%EA%B8%B0_%EB%B9%84%EB%8F%99%EA%B8%B0_%EB%B8%94%EB%A1%9C%ED%82%B9_%EB%85%BC%EB%B8%94%EB%A1%9C%ED%82%B9.md)

## Thread 란?

Thread 의 사전적 의미는 `한 가닥의 실`이라는 의미인데, 한 가지 작업을 실행하기 위해 순차적으로 실행할 코드를 실처럼 이어 놓았다고 해서 유래된 이름이다.
자바에서는 메인 스레드가 main() 메서드를 실행하면서 시작되는데, 멀티 스레드의 경우 메인 스레드가 종료 되더라도, 실행 중인 작업 스레드가 하나라도 있으면 애플리케이션이 종료되지 않는다.

## Thread 생성

- Runnable 을 매개변수로 갖는 생성자를 호출하여 Thread 를 생성할 수 있다.

```java
Thread thread = new Thread(Runnable target);
```

Runnable 은 작업 스레드가 실행할 수 있는 코드를 가지고 있는 객체라고 해서 붙여진 이름이다.

```java
class Task implements Runnable {
    public void run() {
        // do Something
    }
}
```

위 코드를 가독성 좋게 람다식을 사용하여 표현할 수 있다.

```java
Thread thread = new Thread(() -> {
    // do Something
});
thread.start();
```

다음과 같은 방식으로도 스레드를 생성할 수 있다.

```java
Thread thread = new Thread() {
    public void run() {
         // do Something
    }
};
```

## 스레드 이름 설정하기

따로 설정을 하지 않으면 `Thread-n` (n 은 숫자) 이라는 이름으로 설정된다.

```java
thread.setName("스레드 이름");
```

## 현재 스레드 확인하기

```java
Thread.currentThread();
```

## 동시성과 병렬성

멀티 스레드 환경에서는 동시성(Concurrency) 또는 병렬성(Parallelism) 으로 실행된다.

### Parallelism

When two threads are running in parallel, __they are both running at the same time.__ For example, if we have two threads, A and B, then their parallel execution would look like this:

```java
CPU 1: A ------------------------->

CPU 2: B ------------------------->
```

### Concurrency

When two threads are running concurrently, their execution overlaps. Overlapping can happen in one of two ways: either the threads are executing at the same time (i.e. in parallel, as above), or their executions are being interleaved on the processor, like so:

```java
CPU 1: A -----------> B ----------> A -----------> B ---------->
```

동시성은 스레드 스케줄링에 의해서 어떤 순서로 동시성이 실행될 것인지 정해지는데, 스레드 스케줄링에 의해 스레드들은 아주 짧은 시간에 번갈아가면서 그들의 작업(run())을 조금씩 실행한다.

## 자바의 스레드 스케줄링

자바의 스레드 스케줄링은 `우선순위(Priority)` 방식과 `순환 할당(Round-robin)` 방식을 사용한다.

> 숫자가 높을 수록 우선순위가 높다. 1이 가장 낮고, 10이 가장 높다. 우선순위를 부여하지 않으면 모든 스레드들은 기본적으로 5의 우선순위를 할당 받는다.

우선순위 방식은 스레드 객체에 우선순위 번호를 부여할 수 있다. 하지만 순환 할당 방식은 JVM 에 의해서 정해지기 때문에 코드로 제어할 수 없다.

```java
thread.setPriority(Thread.MAX_PRIORITY);
thread.setPriority(Thread.NORM_PRIORITY);
thread.setPriority(Thread.MIN_PRIORITY);
```

## ExecutorService

![executorservice](https://user-images.githubusercontent.com/47518272/153747154-2e4c8906-31b9-4d07-9df7-d377334dfaee.jpg)

> 출처 : https://gompangs.tistory.com/entry/JAVA-ExecutorService-%EA%B4%80%EB%A0%A8-%EA%B3%B5%EB%B6%80

여기서 Customer 라는건 Application 에서 해당 ExecutorService 를 사용하는 클래스 정도로 해석하면 될 것 같고, 해당 클래스에서 ExecutorService 에 작업을 submit 을 하게 되면, ExecutorService 내부에서 해당 작업을 내부적으로 스케쥴링 하면서 적절하게 일을 처리한다.

## Callable 

Callable 은 Runnable 의 단점을 보완하기 위해서 만들어졌다.

```java
public interface Runnable {
    public void run();
}
```

run() 메소드는 결과 값을 리턴하지 않기 때문에, run() 메소드의 실행 결과를 구하기 위해서는 공용 메모리나 파이프와 같은 것들을 사용해서 결과 값을 받아야만 했다. 이런 Runnable 인터페이스의 단점을 없애기 위해 추가된 것이 바로 Callable 인터페이스이다.

```java
public interface Callable<V> {
    V call() throws Exception
}
```

## Future

자바 5부터 미래의 어느 시점에 결과를 얻는 모델로 `Future` 인터페이스를 제공하고 있다. 비동기 계산을 모델링하는데 주로 사용되며, 시간이 걸릴 수 있는 작업을 Future 내부로 설정하면된다.

> Ex. 단골 세탁소에 한 무더기의 옷을 드라이클리닝 서비스를 맡기는 동작에 비유할 수 있다. 세작소 주인은 드라이클리닝이 언제 끝날지 적힌 `영수증(Future)`를 줄 것이며, 드라이니클리닝이 진행되는 동안 우리들은 다른 일을 할 수 있다.

Future 를 사용하려면 시간이 오래 걸리는 작업을 `Callable` 객체 내부로 감싼 다음에 `ExecutorService` 에 제출해야 한다.

### 자바 8 이전의 코드

```java
// ThreadPool 에 task 를 제출하려면 ExecutorService 를 만들어야 한다.
// Callable 을 ExecutorService 에 제출한다.
ExecutorService executor = Executors.newCachedThreadPool();
Future<Double> future = executor.submit((Callable<Double>) () -> {
    return doSomeLongComputation(); // 시간이 오래 걸리는 작업은 다른 스레드에서 비동기적으로 실행한다.
});

doSomethingElse(); // 비동기 작업을 수행하는 동안 다른 작업 수행

try {
    // 비동기 작업의 결과를 가져온다. 결과가 준비되어 있지 않으면 호출 스레드(doSomethingElse())가 블록된다. 
    // 하지만 최대 1초까지만 기다린다.
    Double result = future.get(1, TimeUnit.SECONDS); 
} catch (ExecutionException e) {
    // 계산 중 예외 발생
} catch (InterruptedException e) {
    // 현재 스레드에서 대기 중 인터럽트 발생
} catch (TimeoutException e) {
    // Future 가 완료되기 전에 타임아웃 발생
}
```

![executor](https://user-images.githubusercontent.com/47518272/153747162-ef086c83-5237-462e-9fb2-e97c2a701882.png)

이 시나리오의 문제는 오래 걸리는 작업이 영원히 끝나지 않으면 문제가 생길 수 있다는 것이다. 따라서, get 메서드를 오버로드해서 우리 스레드가 대기할 최대 타임아웃 시간을 정하는 것이 좋다.

### Future 의 단점

Future 인터페이스는 java5부터 java.util.concurrency 패키지에서 비동기의 결과값을 받는 용도로 사용했다. 하지만 `비동기의 결과값을 조합`하거나, `error 핸들링`을 할 수가 없었다.

이러한 단점을 개선하고자 Java 8 에서 CompletableFuture 가 등장했다.

## CompletableFuture

CompletableFuture 는 Future 인터페이스를 구현함과 동시에 CompletionStage 인터페이스를 구현한다. `CompletionStage` 는 비동기 연산 Step 을 제공해서 계속 `체이닝(Chaining)` 형태로 조합이 가능하다.

```java
public Future<Double> getPriceAsync(String product) {
    // 계산 결과를 포함할 CompletableFuture 를 생성한다.
    CompletableFuture<Double> futurePrice = new CompletableFuture<>();
    new Thread(() -> {
        double price = calcaulatePrice(amount); // 다른 스레드에서 비동기적으로 계산을 수행한다.
        futurePrice.complete(price); // 오랜 시간이 걸리는 계산이 완료되면 Future 에 값을 설정한다.
    }).start();
    return futurePrice; // 계산 결과가 완료되길 기다리지 않고 Future 를 반환한다.
}
```

해당 메서드를 사용하는 코드는 다음과 같다.

```java
Shop shop = new Shop("BestShop");
long start = System.nanoTime();
Future<Double> futurePrice = shop.getPriceAsync("my favorite product"); // 상점에 제품 가격 요청
long invocationTime = ((System.nanoTime() - start) / 1_000_000); 
System.out.println("Invocation returned after " + invocationTime + " msecs");

// 제품의 가격을 계산하는 동안 다른 상점 검색 등 다른 작업 수행
doSomethingElse();

try {
    double price = futurePrice.get(); // 가격 정보가 있으면 Future 에서 가격 정보를 일고, 가격 정보가 없으면 가격 정보를 받을 때 까지 블록한다.
    System.out.println("Price is %.2f%n", price);
} catch (Exception e) {
    throw new RuntimeExceptione(e);
}
long retrievalTime = ((System.nanoTime() - start) / 1_000_000);
System.out.println("Price returned after " + retrievalTime + " msecs");
```

상점은 비동기 API 를 제공하므로 즉시 `Future`를 반환한다. 클라이언트는 반환된 Future 를 이용하여 나중에 결과를 얻을 수 있다. 그 사이 클라이언트는 다른 상점에 가격 정보를 요청하는 등
첫 번째 상점의 결과를 기다리면서 대기하지 않고 다른 작업을 처리할 수 있다.

> [Future vs CompletableFuture](https://www.linkedin.com/pulse/java-8-future-vs-completablefuture-saral-saxena)

# Sync, Async, Blocking, Non-Blocking

- __async__
    - 작업을 실행하고 요청 스레드는 다른 작업을 실행
    - 실행 결과를 신경쓰지 않음
- __sync__
    - 이벤트를 자신이 직접 처리(확인의 주체가 유저 프로세스이며, 다 될때까지 기다리거나 스스로 확인)
    - 메소드,작업등을 실행하고 해당 메소드의 결과를 얻어올 때 까지 기다리는 방식
- __block__
    - 완료까지 대기(리턴되기 전까지 멈춤)
- __non-block__
    - 미완료라도 즉시 리턴

## 동기 API vs 비동기 API

- __동기 API__
    - 동기 API 에서는 메서드를 호출한 다음 메서드가 계산을 완료할 때까지 기다렸다가 메서드가 반환되면 호출자는 반환된 값으로 계속 다른 동작을 수행한다. 
이처럼 호출자는 피호출자의 `작업 완료`를 기다리며, 동기 API 를 사용하는 상황을 `블록 호출(blocking call)`이라고 한다.
- __비동기 API__
    - 비동기 API 에서는 메서드가 즉시 반환되며 끝내지 못한 나머지 작업을 호출자 스레드와 동기적으로 실행될 수 있도록 다른 스레드에 할당한다. 이와 같은 비동기 API 를 사용하는 상황을 `비블록 호출(non-blocking call)` 이라고 한다.

## Blocking I/O vs Non-Blocking I/O

웹 애플리케이션에서 외부 API 를 호출하여 사용하는 경우 `RestTemplate` 혹은 `WebClient` 를 사용할 것이다. RestTemplate 은 동기 방식으로 동작하며, WebClient 는 비동기 방식으로 동작한다.

- __Blocking__
    - 다른 메서드를 실행하고 종료를 기다림 (Block)
- __Non-Blocking__
    - 다른 메서드를 실행하고 종료를 기다리지 않음 (Non-block)

### non-blocking API는 요청 처리가 완료된것을 어떻게 통지할까?

- non-blocking API 를 호출하면 호출측은 처리가 완료되었다는것을 반드시 알아야한다.
- 호출측은 어떻게 알게될까?
- CPU 는 인스트럭션(명령어)을 순차적으로 처리한다. 어떻게 처리가 완료되었다는것을 통지할까?
- 운영체제 ready queue 에 들어가지 않고 바로 응답을 한다. 바로 응답하기 힘든경우 에러를 반환하는데 정상데이터를 받을 때까지 계속해서 요청을 다시 보낸다.
- blocking 방식의 비효율적인 부분을 해결하기 위해서 non-blocking 을 사용하는데 계속해서 요청을 다시 보내면서 자원을 낭비하게 된다(시스템 호출이 빈번하게 발생)

### 이벤트 통지 모델

매번 호출측에서 데이터가 준비되었는지 확인하는것을 비효율적이다. 수신 버퍼나 출력 버퍼측에서 이벤트롤 통지하면 어떨까? I/O 작업 결과 반환 방식에 따라 동기,비동기 모델로 구분할 수 있다.

- __Synchronous Model__
    - I/O 작업이 진행되는 동안 유저 프로세스는 결과를 기다렸다가 이벤트를 직접 처리하는 방식
    - notify 를 유저 프로세스(호출측)이 담당하여 주체적으로 진행하면 커널은 유저 프로세스의 요청에 수동적을 응답한다.
- __Asynchronous Model__
    - I/O 작업이 진행되는 동안 유저 프로세스 는 자신의 일을 하다가 이벤트 핸들러에 의해 알림이 오면 처리하는 방식이다.
    - notify 를 커널이 담당하여 주체적으로 진행(콜백을 넘겨주는 형태)하고 호출측은 수동적인 입장에서 통지가 오면 그때 I/O 결과를 반환 받는다.

### Non-Blocking I/O 는 내부에서 어떻게 처리될까?

- 대부분의 non-blocking 프레임워크들은 무한루프를 통해 응답데이터가 존재하는지 지속적으로 확인한다.(poll) 이를 `이벤트 루프`라고 부른다.
    - 즉, 이벤트 루프를 통해서 socket 에서 읽을 데이터가 있는지 계속 확인한다.
- 리눅스 epoll, io_uring (자바 NIO에서 윈도우는 select, 맥은 kqueue, 리눅스는 epoll을 지원)


## References

- [이것이 자바다](http://www.yes24.com/Product/Goods/15651484)
- [Modern Java In Action](http://www.yes24.com/Product/Goods/77125987?pid=123487&cosemkid=go15646485055614872&gclid=Cj0KCQiA0p2QBhDvARIsAACSOONw97-MO96DgAzl2A1bNi53QOCFceL8n6E2VRdPnxDE-U0Q-vD6Lj4aAggPEALw_wcB)
- https://www.eginnovations.com/blog/java-threads/
- https://stackoverflow.com/questions/1050222/what-is-the-difference-between-concurrency-and-parallelism
- https://stackoverflow.com/questions/34689709/java-threads-and-number-of-cores/34689857#34689857
- https://javacan.tistory.com/entry/134
- https://pjh3749.tistory.com/280
- https://stackoverflow.com/questions/1241429/blocking-io-vs-non-blocking-io-looking-for-good-articles
- https://alwayspr.tistory.com/44
- https://github-wiki-see.page/m/GANGNAM-JAVA/JAVA-STUDY/wiki/Blocking,Non-Blocking,-Synchronous,-Asynchronous
