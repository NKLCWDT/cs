# Http의 진화과정
예전에는 리소스의 양도 많지 않고 요청의 수가 적어서 TCP의 성능에 문제가 없었다. 하지만 요즘은 리소스도 많아지고 웹페이지에 요청의 수도 많아져서 TCP의 성능이 저하되는 한계를 겪으면서 그 문제점들을 하나씩 해결해 나가면서 HTTP0.9에서 HTTP1.0,Http1.1, Http2로 발전해 나간다.

## HTTP 0.9
원라인 프로토콜이라고도 불린 HTTP 0.9는 아주 간단한 프로토콜이다. 요청은 하나의 라인으로만 구성되어 있었으며 메서드는 get이 유일했다. 그래서 매우 제한적이였고 다른 기능들을 위해 확장이 필요했다.
![images](/images/http1.png)

## HTTP 1.0
- 버전 정보가 요청사이에 전송되기 시작했고
- 브라우저가 요청에 대한 실패와 성공에 대한것을 보낼 수 있게 상태코드 또한 응답에 붙어서 나갔다.
- 요청과 응답을 위해 HTTP헤더 가 도입되어 프로토콜을 유연하고 확장 가능하게 만들어주었다.
- HTML파일들 외에 다른 문서들을 전송하는 것이 가능해졌다.(Content-type)
- 1번의 connection에 1번의 요청 그리고 1번의 응답만 가능했다. (여러개의 resource요청을 위해서는 여러개의 connection을 맺고 끊어야한다.)
![images](/images/http2.png)
- TCP는 느린전송을 수행하는데 이것으로 인해 데이터 전송횟수가 늘어나며 latency증가한다.
>느린전송은 전송할 수 있는 데이터의 크기에 한계를 두고 여러번 왔다갔다 하며 최대의 크기를 찾는과정이다.

이후 사람들은 tcp connection을 재활용하는 방법을 생각해내는데 그것이 keep-alive이다. keep-alive는 http1.0의 표준이 아니였기 때문에 어떤곳에서는 지원을 하지만 지원 하지 않는 곳도 있었다. 

## HTTP 1.1
- persistent connection이라는 개념이 도입된다.
> 지정 시간동안 connection을 열어놓는 방식  
응답과 요청이 순서대로 이루어져야했기 때문에 데이터를 정기적으로 밖에 가져오지 못했다.(latency)
- 그래서 pipelining이라는 개념이 도입되었다
> pipelining은 요청에 대한 응답이 돌아 오지 않아도 다음 요청 순차적으로 보내고 그 순서에 맞춰 응답을 받는 방식으로 latency를 줄임 
![images](/images/pipelining.png)

__문제점__
- Head Of Line Blocking
![images](/images/headOfLineBlocking.png)
  - 응답을 순차적으로 보내기때문에 먼저 보낸 요청이 시간이 많이 걸리거나 밀리는 경우 뒤에 있는 응답들도 처리가 끝나도 응답을 보내지 못한다.

- Header의 중복
  - 연속된 요청일 경우 header가 같은 경우가 많은데 이렇게 header가 중복이 되면서 header가 커진다.

## HTTP 2.0
- Http메시지 전송 방식 변화
  - 일반 텍스트를 바이너리 프레이밍 계층을 사용해 프레임이라는 단위로 변환하고 binary로 인코딩한다.
  - 멀티플렉싱 
  > 멀티플렉싱은 tcp하나의 커넥션에 여러개의 데이터를 섞이지 않게 요청의 응답과 순서에 관계없이 보내는것.
  > stream을 통해 가능하게 되었다.(stream은 데이터에 식별자가 붙어 있는 형태)

![images](/images/stream.png)

- 서버 푸시 
  - 클라이언트에서 요청하지 않은 것들을 보내준다.
  - index.html파일을 요청하면 script.js파일과 style.css같이 보내준다.

- Header 압축
![images](/images/headerCompression.png)
  - 중복을 검출해 그 부분을 제외하고 huffman 인코딩으로 압축을해 헤더 크기를 줄여 페이지 로드 시간을 줄여줬다.


하지만 또 TCP Head Of Line Blocking발생
Http2는 stream으로 순서에 구애받지 않고 분리할 수 있었지만 tcp는 신뢰성을 확보하기 위해 발생하는 지연을 해결하기 힘듬.

## Quic
- UDP를 사용해 구글이 만들었다
> 어플리케이션 계층에서 신뢰성을 구현
- 전송계층 프로토콜이다
- 전송 속도 향상
- connection uuid라는 고유 식별자로 연결해 커넥션 재수립할 필요 없어짐
![images](/images/quic.png)
- stream을 독립시켜서 tcp head of line blocking
  - 하나의 tcp chain에 데이터들이 엮인 상태로 전송됨
  - 하나의 커넥션 안에서 stream이 다른 chain을 가져서 문제가 생기더라도 다른 stream에 영향을 안줌

>참고자료

https://www.youtube.com/watch?v=ZgSC5K1sUYM

https://www.youtube.com/watch?v=xcrjamphIp4

https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/Evolution_of_HTTP
