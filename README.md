# CS 및 기술면접준비 스터디

- [스터디 RULE](https://baekjh.notion.site/NKLCWDT-f29b0d5c5194408dabf4532ce556d1fe)
- [스터디 모집 공고](https://okky.kr/article/1104939?note=2590724)
- [스터디 일지](https://baekjh.notion.site/CS-e4c215ee99b64a60a6c367aad37ca5fd)
- 발표 순서(Queue, FIFO)
  - [Seunguk](https://github.com/rlatmd0829)
  - [JungHo](https://github.com/BAEKJungHo)
  - [JiHye](https://github.com/jola7373)
  - [WestSea](https://github.com/westssun)
  - [MyungJin](https://github.com/ann-mj)

## ✔️ Network

- [쿠키, 세션, JWT 토큰](https://github.com/NKLCWDT/cs/blob/main/Network/Cookie%2C%20Session%2C%20JWT.md)
  - Cookie / Session
  - 세션 기반 인증 / 토큰 기반 인증
  - JWT
- [네트워크 시스템의 Layered Architecture](https://github.com/NKLCWDT/cs/blob/main/Network/%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC%20%EC%8B%9C%EC%8A%A4%ED%85%9C%EC%9D%98%20Layered%20Architecutre.md)
  - 웹의 동작 방식
  - TCP/IP 5 Layer
  - OSI 7 Layer
  - TCP, UDP, IP, PORT
- [HTTP 와 HTTPS](https://github.com/NKLCWDT/cs/blob/main/Network/http%EC%99%80%20https.md)
  - HTTP/HTTPS
  - [HTTP 2.0](https://github.com/NKLCWDT/cs/blob/main/Network/http%EC%9D%98%20%EC%A7%84%ED%99%94.md)
- [로드 밸런서(Load Balancer)](https://github.com/NKLCWDT/cs/blob/main/Network/%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC%20%EC%8A%A4%EC%9C%84%EC%B9%98%EC%9D%98%20%EB%A1%9C%EB%93%9C%EB%B0%B8%EB%9F%B0%EC%84%9C.md)
  - 로드밸런싱
  - Scale out, Scale up
- [프록시(Proxy)](https://github.com/NKLCWDT/cs/blob/main/Network/%ED%94%84%EB%A1%9D%EC%8B%9C.md)
  - 포워드 프록시, 리버스 프록시
  - DMZ
- [네트워크 보안(Network Security)](https://github.com/NKLCWDT/cs/blob/main/Network/%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC%20%EB%B3%B4%EC%95%88(Network%20Security).md)
    - 보안 3대요소, 웹 해킹, 방화벽
    - IPSec, SSL 과 TLS
    - SQL Injection, OS Command Injection, XSS, CSRF
- [웹 소켓(Web Socket)](https://github.com/NKLCWDT/cs/blob/main/Network/%EC%9B%B9%EC%86%8C%EC%BC%93.md)
    - HTTP Polling, HTTP Long Polling
    - HTTP vs 웹 소켓
    - SocketJs 와 Socket.io
- [CORS(Cross Origin Resource Sharing)](https://github.com/NKLCWDT/cs/blob/main/Network/CORS.md)
    - CORS
    - SOP
    - 스프링 부트에서 CORS 해결 하는 방법
- [REST and RESTful](https://github.com/NKLCWDT/cs/blob/main/Network/Rest.md)
    - REST 구성요소와 기본 원칙
    - REST API 디자인 가이드
- [OAuth(Open Authorization)](https://github.com/NKLCWDT/cs/blob/main/Network/OAuth.md)
    - OAuth1.0, OAuth2.0
    - 일반로그인과 OAuth로그인 차이
    - 인증 절차 종류

## ✔️ OS(Operating System)

- [운영체제와 컴퓨터 시스템 구조](https://github.com/NKLCWDT/cs/blob/main/Operating%20System/%EC%9A%B4%EC%98%81%EC%B2%B4%EC%A0%9C%EC%99%80%20%EC%BB%B4%ED%93%A8%ED%84%B0%20%EC%8B%9C%EC%8A%A4%ED%85%9C%20%EA%B5%AC%EC%A1%B0.md)
- [CPU](https://github.com/NKLCWDT/cs/blob/main/Operating%20System/CPU.md)
- 캐시의 지역성
- [메모리 구성](https://github.com/NKLCWDT/cs/blob/main/Operating%20System/Stack_Heap.md)
  - 스택 동작 과정
  - Stack 과 Heap
  - 프로세스/스레드에서의 스택
- [프로세스와 스레드](https://github.com/NKLCWDT/cs/blob/main/Operating%20System/%ED%94%84%EB%A1%9C%EC%84%B8%EC%8A%A4%EC%99%80%20%EC%93%B0%EB%A0%88%EB%93%9C.md)
  - Multi Process, Multi Thread, PCB, 프로세스 상태 전이
  - Interrupt, Context Switching
  - WAS, ThreadPool
  - 스프링에서 동시성 이슈를 피하는 방법
  - Thread Safe 하게 설계하는 방법
- [프로세스 동기화](https://github.com/NKLCWDT/cs/blob/main/Operating%20System/%ED%94%84%EB%A1%9C%EC%84%B8%EC%8A%A4%20%EB%8F%99%EA%B8%B0%ED%99%94.md)
  - 경쟁 상태(race condition)
  - 임계 구역(ciritical section)
  - 뮤텍스 락(Mutex Locks)
  - 세마포(semaphore)
- 교착상태와 기아상태
- [CPU 스케줄링 기법](https://github.com/NKLCWDT/cs/blob/main/Operating%20System/CPU%20%EC%8A%A4%EC%BC%80%EC%A4%84%EB%A7%81.md)
  - FIFO, SJF, STCF
  - Round Robin
  - Busy Waiting
  - Multi-Level Feedback Queue
  - Priority Boost
