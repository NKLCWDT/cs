가장 기본적인 데이터베이스 구조   
<img src="/images/normal.png" width="300" height="300"/><br> 


## 클러스터(Cluster)
위 사진을 보면 DB서버와 DB스토리지가 한개씩 이뤄져있습니다.   
하지만, 만약에 이 상황에서 DB 서버가 죽는다면 어떻게 될까요? 이 데이터베이스를 쓰는 서비스 자체가 중단됩니다. 그래서 DB서버를 여러대를 두는 DB cluster을 사용하게 됩니다.
<br>

### 클러스터 사용장점
1. Data Redundancy<br>
여기서 말하는 redundancy는 DB 서버가 여러대 있어 데이터들이 모든 서버에 

2. 로드밸런싱<br>
일들을 각각의 서버에서 나눠서 처리해서 트래픽을 줄일 수 있다. 

3. High Availability<br>
데이터베이스에 접근이 거의 항상 가능하다. 로드밸런싱과 Server가 많은 덕에 한 서버가 고장 나더라도 데이터베이스에 접근이 가능하다.  
<br>

### Fail Over Clustering 
<img src="/images/active-active.png" width="300" height="300"/><br>  
위와 같이 두개의 active db server를 두게 되면 두개중 하나의 서버가 죽더라도 다른 하나의 서버가 있기에 정상적으로 서비스 가능해진다. 또 한개의 DB서버가 하던일을 두개의 DB서버가 나눠서 하므로 CPU, Memory자원의 부하도 적어지게 된다.

<br>
단점은, 두대의 DB서버가 한개의 DB스토리지를 공유하기에 병목이 생길 수 있다. 
그래서 이점을 보완하기 위해 한대는 Active로 두고 한대는 Stand-by상태로 둔다.
<img src="/images/active-standby.png" width="300" height="300"/><br>  

<br>

## 리플리케이션(Replication)
Replication의 뜻 그대로 저 기본적인 데이터베이스 구조를 복제본을 가진다는 것이다. 
<br>
<img src="/images/replication.png" width="300" height="300"/>  <br> 
여기서 사진에 보이는 Master db를 primary라고도 하고 slave db를 secondary db라고도 한다. 그리고 primary db의 복사본을 replica라고 한다. 

<br>
<br>

### 리플리케이션을 사용하는 이유

#### 1. 데이터를 잃지 않기 위해서
데이터가 여러 replica에 복제 되어 있기 때문에 한 DB에 문제가 생기더라도 다른 DB에 같은 데이터가 저장되어 있기 때문에 데이터를 잃지 않을 수 있다.

#### 2. Replica가 primary DB가 될수 있다. 
만일 문제가 생겨서 primary DB가 작동하지 않게 된다면 replica중 하나가 primary db가 될 수 있다.

#### 3. 지연(latency)를 줄일 수 있다.
예를 들어 사용자가 요청한 곳에서 지역적을 먼곳에 있는 Data center에 해당 database가 있게 되면 느리게 되는데 database를 여러군데 두게 되면 어느곳에서 접속해도 빠르다.  
<img src="/images/replicationLatency.png" width="300" height="300"/><br>   

#### 4. 서버 성능 향상

<br>

### 리플리케이션 래그(replication lag)
replication lag는 primary DB에서 secondary db로 copy 되는 시간을 말한다. 

<img src="/images/replicationLag.png" width="300" height="300"/><br> 
t1이 수행이 primary db를 변경 시키면 t2 또한 수행되는데 이때 t2가 수행되는 시간을 replication lag라고 할 수 있다. 
이 replication lag이 길어지게 되면(특히, replica가 많을시) secondary db들에서 읽는 데이터와 primary db에서 읽는 데이터가 달라지게 되므로 데이터 일관성이 지켜지지 않게 된다. 
<br>
<br>
### Synchronous replication
이 방법은 클라이언트가 쓰기를 했을 때 primary DB가 모든 replica에 바뀐 데이터를 적용한 후에 쓰기가 성공했다는 메시지를 보내주는것이다. 

__장점__<br>
데이터 일관성을 지킬 수 있다.

__단점__<br>
모든 replica가 쓰기가 완료되기까지 기다려야 하므로 시간이 오래걸린다. 만일, replica중에 하나가 쓰기 도중 문제가 생기면 해당 쓰기 자체가 실패하게 된다. 
<br>
<br>
### Asynchronous replication
클라이언트가 쓰기 요청시 primary DB에 적용후 바로 클라이언트에서 쓰기 성공메시지를 보낸 후 변경 메시지를 보낸 후에 바로 쓰기 완료가 된다. 

__장점__<br>
쓰기가 빨라진다.
<br>

__단점__<br>
하지만 만약 update가 실패된다면 또 데이터 일관성이 지켜지지 않게 된다. 

<br>

__그래서 상황에 따라 어느것을 사용할 지 정해야 한다. Banking 시스템이나 Flight 시스템 같이 데이터 일관성이 중요한곳에서는 asynchronous 방법을 사용하고 perfomance가 더 중요한 경우에는 Synchronous방법을 사용해야 한다.__ 
<br>
<br>

### Partitioning(파티셔닝)
하나의 큰 데이터셋을 여러개의 작은 테이블으로 나누는것을 파티셔닝이라고 한다. 
<br>

#### Vertical Partitioning
<img src="/images/verticalPartitioning.png" width="300" height="300"/><br> 
한개의 테이블에서 큰데이터를 가진 컬럼을 다른 테이블로 빼서 사용한다. 서버의 performance증진을 하기 위해 사용한다. 특히, 대용량 데이터를 가진 컬럼을 query가 모든 테이블을 검색할때 시간이 오래걸리는데 이런경우 Vertical Partitioning을 이용하면 access time을 줄일 수 있다.<br>  

<img src="/images/partitioning.png" width="300" height="300"/><br>
사진을 보면 employee라는 테이블이 있다. 여기서 용량이 큰 Picture 컬럼만 빼서 따로 테이블을 만든다. 

<br>

__언제 테이블을 파티션 해야할까?__ <br>
- 테이블이 2GB보다 큰경우 파티셔닝 하는것을 고려해야한다.
- History table같이 최근 데이터에만 적기를 하는 경우, 적을 필요가 없는  데이터들을 따로 파티셔닝할 수 있다.
<br>
<br>

### Sharding(샤딩)
<img src="/images/shading.png" width="350" height="300"/><br>
데이터 파티셔닝의 종류이다. Horizontal Partitioning이라고도 한다. Row 단위로 나눠서 테이블을 저장하는 방법.
샤딩은 데이터를 샤드키에 따라 각자 다른 데이터 서버에 저장한다. 모든 분산된 테이블은 하나의 샤드키 만을 가지고 있다. 

> 샤드: 전체중에 작은 부분(즉, 분산된 부분)
<br>

#### Shard key(샤드키)
어떤 샤드에 저장되어야 할지 결정하는 키이다. 
<br>

#### 언제 테이블을 샤드해야할까?
1. 인덱스 사이즈가 너무큰 경우 (인덱스 사이즈를 줄여준다.) 줄여주면 데이터베이스에 접근시 시간이 줄어든다.

2. 지역별로 나누는 경우일 때. 예를 들어, 미국,유럽, 아시아권으로 사용자들을 나눠서 저장하면 

3. 데이터베이스 분산이된다. 각, 샤드를 다른 데이터베이스 서버에 저장해서 더 빠른 처리 속도를 가진다.
<br>
#### 샤딩의 단점
사실 샤딩을 쓰기 전에 최대한 다른 방법을 알아봐야한다. 데이터베이스 퍼포먼스를 올릴 방법(vertical partitioning, storage 키우기)을 최대한 찾아보고 그 방법들을 다 사용하지 못할 때 사용해야 한다. 샤딩은 프로그래밍과 조작하는데에 복잡도를 더한다. 한곳에서 데이터를 쓰지 못하게 된다. 


<br>
<br>
<br>
<br>

> 참고자료
- https://www.youtube.com/watch?v=y42TXZKFfqQ
- https://www.youtube.com/watch?v=RIcNswROzCc&list=RDCMUCMrRRZxUAXRzjai0SSoFgdw&start_radio=1&rv=RIcNswROzCc&t=975
- https://www.youtube.com/watch?v=NnA1WgPSbgY&t=18s
- https://www.singlestore.com/blog/database-sharding-vs-partitioning-whats-the-difference/
- https://www.ndimensionz.com/2018/01/05/what-is-database-clustering-introduction-and-brief-explanation/
- https://jordy-torvalds.tistory.com/94
