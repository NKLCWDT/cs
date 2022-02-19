# Redis 에 대해서 알고 계시나요?

- Redis 는 Remote dictionary server 로 데이터베이스가 아닌 메모리에 자주 참조하는 데이터를 값을 저장하고 조회하고자 탄생하였습니다.

### Q. Memcached 와 의 차이는 무엇인가요?

- 가장 큰 차이는 Redis 에서는 자바의 LinkedList, HashMap 과 같은 자료구조를 제공하는 것으로 알고 있습니다.

> 참고. Strings, List, Set, Sotred Set, Hash Collection 을 제공

### Q. 자료구조를 제공하면서 얻는 이점이 무엇인가요?

- 외부의 Collection 을 잘 이용하면, 개발 시간을 단축 시킬 수 있고, 직접 만들어 사용하는 것 보다 문제가 줄어들기 때문이라고 생각합니다.

### Q. 자바의 HashMap 도 메모리에 저장하는건데, 왜 Redis 를 사용하는 것일까요?

- 서버가 여러대인 경우 서버마다 다른 값을 저장하기 때문에 Consistency 문제가 발생할 수 있습니다.
- 또한 멀티 쓰레드 환경에서 경쟁 상태(race condition)가 발생할 수도 있습니다.

이러한 `Consistency` 문제와 `Race Condition` 문제를 Redis 는 해결해 주는 것으로 알고 있습니다.

- Redis 는 기본적으로 `Single Thread` 이며
- Redis 자료 구조는 `Atomic Critical Section` 에 대한 동기화를 제공합니다.
- 또한 서로 다른 Transcation 의 read/write 동기화를 제공하여 원치 않은 결과가 조회되지 않도록 막아줍니다.

### Q. 레디스는 그럼 언제 사용하나요?

- `[Remote Data Store]` : 여러 서버에서 같은 데이터를 공유해야하는 경우 사용합니다.
  - A, B, C 서버에서 데이터를 공유하고 싶은 경우
- Single Server 라도 `Atomic` 한 자료구조를 제공해야하거나 `Cache` 기능을 사용하기 위해서도 사용됩니다.
- 주로 많이 쓰는 곳들
  - 인증 토큰 등을 저장(Strings or hash)
  - Ranking 보드로 사용(Sorted set)
  - 유저 API Limit
  - 좌표(list)
  - 등등

### Q. 레디스 사용 시 주의할 점이 있을까요?

__`메모리 관리`를 잘하는 것과 `O(N) 관련 명령어`를 주의해야 합니다.__

- Single Thread 서버라서 시간 복잡도를 고려해야 합니다.
  - 하나의 요청이 처리 중이라면, 다른 요청은 대기해야하기 때문에 `O(N)` 관련 명령어들을 조심해서 사용해야 한다고 알고 있습니다.
  - Ex. O(N) 명령들 : KEYS, FLUSHALL, FLUSHDB, Delete Collections, Get All Collections 
- In-Memory 데이터베이스라는 특성상 메모리 파편화와 가상 메모리 등에 대한 이해가 필요합니다.
- __대표적인 실수 사례__
  - Key 가 백만개 이상인데 확인을 위해 KEYS 명령을 사용하는 경우
  - 아이템이 몇만개 든 hash, sorted set, set 에서 모든 데이터를 가져오는 경우
  - 예전의 Spring Security OAuth RedisTokenStore (지금 최신버전은 써도 된다.)

### Q. KEYS 대신 무엇을 사용하는게 좋을까요?

- scan 명령을 사용하여, 하나의 긴 명령을 짧은 여러번의 명령으로 바꾸는 것이 좋습니다.

### Q. Spring Security OAuth RedisTokenStore 이슈

- Access Token 의 저장을 `List(O(N))` 자료구조를 통해서 이루어졌습니다.
  - 검색, 삭제시에 모든 item 을 매번 찾아봐야 했었습니다.
  - 100만개쯤 되면 전체 성능에 영향을 줍니다.
- 현재는 `Set(O(1))` 을 이용해서 검색, 삭제를 하도록 수정되어있습니다.

### Q. 메모리 관리에 대해서 설명해 주세요.

- Redis 는 `In-Memory Data Store` 입니다.
- Physical Memory 이상을 사용하게되면 문제가 발생하는데
  - Swap 이 있다면 Swap 사용으로 인해(디스크에 접근하므로) 메모리 Page 접근시마다 속도가 늦어집니다.
- Swap 없이 Maxmemory 를 설정하더라도 이보다 더 사용할 가능성이 큽니다.
- 따라서, RSS 값을 모니터링 해야 합니다.
- 큰 메모리를 사용하는 instance 하나보다는 적은 메모리를 사용하는 instance 여러개가 안전합니다.
  - Ex. 24 GB Instance < 8 GB Instance x 3
- Redis 는 메모리 파편화가 발생할 수 있습니다.
  - 4.x 부터 메모리 파편화를 줄이도록 `jemalloc` 에 힌트를 주는 기능이 탑재되었으나, jemalloc 버전에 따라 다르게 동작할 수 있습니다.

> jemalloc 은 메모리 파편화를 줄이기 위한, 메모리 할당자 입니다.

### Q. 메모리가 부족하면 어떻게 해야할까요?

- Cache is Cash !!
  - 좀 더 메모리 많은 장비로 Migration 해야 합니다.
- 있는 데이터 줄이기
  - 데이터를 일정 수준에서만 사용하도록 특정 데이터를 줄여야 합니다.
  - 다만, 이미 Swap 을 사용중이라면, 프로세스를 재시작 해야 합니다.

### Q. 메모리를 줄이기 위한 설정에는 무엇이 있을까요?

- 기본적으로 Collection 은 다음과 같은 자료구조를 사용합니다.
  - Hash -> HashTable 을 하나 더 사용
  - Sorted Set -> Skiplist 와 Hash Table 사용
  - Set -> HashTable 사용
  - 해당 자료구조들은 메모리를 많이 사용합니다.
- [Ziplist](http://redisgate.kr/redis/configuration/internal_ziplist.php) 를 사용하면 속도는 조금 줄어들지만 메모리를 적게 사용할 수 있습니다.
  - List, Hash, Sorted Set 등을 ziplist 로 대체해서 처리하는 설정이 존재 합니다.
