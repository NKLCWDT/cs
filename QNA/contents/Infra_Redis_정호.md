# Redis 에 대해서 알고 계시나요?

- Redis 는 Remote dictionary server 로 데이터베이스가 아닌 메모리에 자주 참조하는 데이터를 값을 저장하고 조회하고자 탄생하였습니다.

### Q. Memcached 와 의 차이는 무엇인가요?

- 가장 큰 차이는 Redis 에서는 자바의 LinkedList, HashMap 과 같은 자료구조를 제공하는 것으로 알고 있습니다.

### Q. 자바의 HashMap 도 메모리에 저장하는건데, 왜 Redis 를 사용하는 것일까요?

- 서버가 여러대인 경우 서버마다 다른 값을 저장하기 때문에 `Consistency` 문제가 발생할 수 있습니다.
- 또한 멀티 쓰레드 환경에서 경쟁 상태(race condition)가 발생할 수도 있습니다.

이러한 Consistency 문제와 Race Condition 문제를 Redis 는 해결해 주는 것으로 알고 있습니다.

- Redis 는 기본적으로 Single Thread 이며
- Redis 자료 구조는 Atomic Critical Section 에 대한 동기화를 제공합니다.
- 또한 서로 다른 Transcation 의 read/write 동기화를 제공하여 원치 않은 결과가 조회되지 않도록 막아줍니다.

### Q. 레디스는 그럼 언제 사용하나요?

- 여러 서버에서 같은 데이터를 공유해야하는 경우 사용합니다.
- 또는 Single Server 라도 Atomic 한 자료구조를 제공해야하거나 Cache 기능을 사용하기 위해서도 사용됩니다.

### Q. 레디스 사용 시 주의할 점이 있을까요?

- Single Thread 서버라서 시간 복잡도를 고려해야 합니다.
  - 하나의 요청이 처리 중이라면, 다른 요청은 대기해야하기 때문에 `O(N)` 의 시간 복잡도가 나오지 않게 조심해서 사용해야 한다고 알고 있습니다.
  - Ex. KEYS, Flush, GetAll 연산 
- In-Memory 데이터베이스라는 특성상 메모리 파편화와 가상 메모리 등에 대한 이해가 필요합니다.
