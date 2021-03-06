# 블록체인에 대해 설명해 주세요.

- 블록체인은 모든 이용자의 `과반수 합의`에 기초합니다.
- 예를들어 비트코인을 전송하면 그 사실이 모든 이용자에게 공개되고, 그 거래가 맞다고 과반수의 사람들이 검증해주면 그 사실은 영구적으로 변경이 불가능하게 됩니다.
- 이러한 `검증과정`이 `채굴`이며, 채굴자들은 여러 거래정보들을 묶어서 하나의 블록을 만듭니다. 블록안에는 수십, 수천개의 거래가 들어있으며, 블록은 `해시값`으로 이루어져 있습니다.
- 따라서 하나의 거래 정보만 변경되더라도 해시 값이 변하기 때문에 `조작 여부`를 알 수 있습니다.
- 채굴자들은 블록을 만드는 수고비로 `비트코인`을 얻게 됩니다.
- 다음 채굴자는 블록을 만들 때, 직전 블록의 해시값을 새로운 거래 정보들에 포함해 해시함수를 암호화합니다.
- 매 블록들의 해시값이 다음 블록의 해시값에 직접적으로 영향을 미치기 때문에 `블록체인(Blockchain)`이라고 합니다.

> 블록체인과 암호화폐가 더 궁금하다면 [제가 정리한 블로그 내용](https://medium.com/webeveloper/%ED%99%94%ED%8F%90%ED%98%81%EB%AA%85%EC%9D%84-%EC%9D%BD%EA%B3%A0-%EC%A0%95%EB%A6%AC%ED%95%9C-%EA%B8%80-2f0d7306e867)을 참고하시면 도움 될 것같습니다.
