# Pooling
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdD5Sry%2FbtrG6Xhv3kQ%2FunJPhTb24eySmpab9OviCk%2Fimg.png)
- WebSocket이 나오기 전 사용했던 기술
- 일정 주기로 서버에 요청을 보내서 응답을 받는 방법
- 불필요한 요청과 커넥션이 발생하여 서버 리소스를 많이 소모시킨다.
- 주기에 따라 실시간으로 착시효과를 발생 시킬 수 있지만, 따지고 보면 실시간은 아니다.
- HTTP 통신이기 때문에 Response / Request 헤더가 불필요하게 크다.

# LongPooling
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FzxnyF%2FbtrG8r3vJmV%2FB10zbZkayqRkNZnnlF50I1%2Fimg.png)
- Pooling과 유사하게 일정 주기마다 요청을 보내지만 서버가 바로 응답하지 않는 방식
- 특정 이벤트나 타임아웃이 발생했을 때 응답을 전달한다.
- 때문에 불필요한 요청 / 커넥션을 하지 않기 때문에 Pooling보다 리소스 소모는 적다
- HTTP 통신이기 때문에 Response / Request 헤더가 불필요하게 크다.

# Streaming
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FpmsKD%2FbtrG9laqiAH%2F750kNT2TaEknEcTjqBwTBk%2Fimg.png)
- 커넥션을 맺고 이벤트 발생 시 응답을 내려준다.
- 단, 응답을 내려주고 연결은 계속 유지되는 방식
- 연결 시간이 길어질 수록 연결 유효성 관리 부담이 발생
- HTTP 통신이기 때문에 Response / Request 헤더가 불필요하게 크다.

# WebSocket / SSE
||WebSocket|SSE|
|---|---|---|
|브라우저 지원|대부분 브라우저에서 지원|대부분 모던 브라우저에서 지원(polyfill을 통한 크로스 브라우징 가능, polyfill이란 브라우저가 지원하지 않는 API를 플러그인이나 JavaScript 등으로 흉내 내 구현한 것)|
|통신 방향|양방향|단방향(서버 -> 클라이언트)|
|리얼 타임|O|O|
|데이터형태|Binary, UTF-8|UTF-8|
|자동 재접속|N|O(3초마다 재시도)|
|최대 동접 수|서버 셋업에 따라 달라짐|HTTP/1.1 브라우저당 6개, HTTP/2.0 브라우저당 100개|
|프로토콜|WebSocket|HTTP|

## SSE
![](https://velog.velcdn.com/images/alswn9938/post/d466fefc-6fb4-4c1c-b7e9-7f28e4d7b8af/image.png)
- Server Sent Event
- `서버 -> 클라이언트`로 `단방향`으로 `실시간` 이벤트를 전송하는 웹 기술
- 기존 HTTP 통신은 하나의 요청에 대하여 하나의 응답만 하는 것에 반해 서버에서 이벤트가 발생했을 때 실시간으로 클라이언트에 정보를 전달할 수 있다.
- SSE는 단방향 통신이기 때문에 서버에서 클라이언트로만 데이터 전송 가능
- 클라이언트는 HTTP 프로토콜을 통해 SSE 연결을 설정
- 서버는 HTTP 응답을 유지한 상태에서 데이터 전송
- SSE는 재연결 기능을 제공하기 때문에 연결이 끊어졌을 때 자동으로 다시 연결 시도
- SSE는 이벤트 스트림 형태로 데이터를 전송, 클라이언트는 이벤트를 수신하여 처리가능
- 통신 과정
    - Client 측 - SSE Subscribe 요청
        - 클라이언트가 서버의 이벤트를 구독하기 위한 요청을 전송
        - 이벤트의 mediaType은 text/event-stream이 표준 스펙
    - Server 측 - Subscription에 대한 응답
        - Response의 mediaType은 text/event-stream.
        - 서버는 동적으로 생성된 컨텐츠를 스트리밍하기 때문에 본문의 크기를 미리 알 수 없으므로 Transfer-Encoding 헤더 값을 chunked로 설정해야 한다.
    - Server 측 - 이벤트 생성 및 전송
        - 자신을 구독하고 있는 클라이언트에게 비동기적으로 데이터를 전송할 수 있다.
        - 데이터는 utf-8로 인코딩된 텍스트 데이터만 가능
        - 서로 다른 이벤트는 개행 문자 두개(\n\n)로 구분
        - 각 이벤트는 한 개 이상의 name:value로 구성된다.
            ~~~
            event: type1
            data: An event of type1.

            event: type2
            data: An event of type2.
            ~~~
    - 요청
        ~~~
        GET /connect?articleId=news HTTP/1.1
        Accept: text/event-stream
        Cache-Control: no-cache
        ~~~
    - 응답
        ~~~
        HTTP/1.1 200
        Content-Type: text/event-stream;charset=UTF-8
        Transfer-Encoding: chunked
        ~~~


![](https://amaran-th.github.io/static/b4d730418c22cda965ad10ebcdf37d93/eb2af/1.png)

![](https://amaran-th.github.io/static/8e438c9529721236dbd1709fd1ae2bcd/eb2af/2.png)

## WebSocket
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdDiLTo%2FbtrG4Iebgdo%2F8KL22qu1Iu4rQ1YJlziWY1%2Fimg.png)
- `서버` <-> `클라이언트`로 `양방향`으로 연속적 연결을 지원하는 기술
- WebSocket이 연결되면 클라이언트 혹은 서버에서 연결을 종료할 떄 까지 연결이 지속된다.
- 빨간색 `Opening Handshake` / 노란색 `Data Transfer` / 보라색 `Closing Handshake`
- Handshake
    - 일반적인 HTTP TCP 통신
    - 접속 요청은 HTTP로 하고, 웹소켓 프로토콜로 변경되어 통신한다.
    - Request 헤더
        ~~~
        GET /chat HTTP/1.1
        Host: localhost:8080
        Upgrade: websocket
        Connection: Upgrade
        Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw==
        Sec-WebSocket-Protocol: chat, superchat
        Sec-WebSocket-Version: 13
        Origin: http://localhost:9000
        ~~~
        - Connection: Upgrade 
            - 클라이언트 측에서 프로토콜을 바꾸고 싶다는 신호를 보냈다는 것을 나타냄
        - Upgrade: websocket
            - 클라이언트측에서 요청한 프로토콜은 'websocket’이라는걸 의미
        - Sec-WebSocket-Key
            - 보안을 위해 브라우저에서 생성한 키로, 서버가 웹소켓 프로토콜을 지원하는지를 확인하는데 사용
        - Sec-WebSocket-Version
            - 웹소켓 프로토콜 버전이 명시됩니다. 예시에서 버전은 13
    - Response 헤더
        ~~~
        HTTP/1.1 101 Switching Protocols
        Upgrade: websocket
        Connection: Upgrade
        Sec-WebSocket-Accept: HSmrc0sMlYUkAGmm5OPpG2HaGWk=
        Sec-WebSocket-Protocol: chat
        ~~~
        - HTTP/1.1 101 Switching Protocols
            - 101은 HTTP에서 WS로 프로토콜 전환이 승인 되었다는 응답코드
        - Sec-WebSocket-Accept
            - 요청 헤더의 Sec-WebSocket-Key에 유니크 아이디를 더해서 SHA-1로 해싱한 후 base64로 인코딩한 결과
            - 웹 소켓 연결이 개시되었음을 알림
- Transfer
    - 연결이 수립되면 클라이언트와 서버 양측간의 데이터 통신 단계가 시작
    - 서로는 `메세지`를 보내며 통신하는데, 이 메세지는 `프레임(Frame)` 단위로 이루어진다.
    - 브라우저 환경의 개발자는 바이너리와 텍스트 프레임만 다루게 된다.
    - 또한, 연결 수립 이후에는 서버와 클라이언트는 언제든 상대방에게 ping 패킷을 보낼 수 있다.
        - Ping 을 수신한 측은 가능한 빨리 pong 패킷을 상대방에게 전송해야한다.
        - 이런 방식으로 서로의 연결이 살아있는지를 주기적으로 확인할 수 있는데, 이를 Heartbeat 라고 한다.
    - 전송량이 많은 앱을 개발한다 가정할 때 수신자의 네트워크 속도가 낮을 수 있다.
    - 데이터 수신 속도가 느린데 서버(혹은 클라이언트)에서 무작정 send를 할 수 없는 상황
    - 이때 송신자는 버퍼에 얼만큼의 데이터가 쌓여있는지 보고 소켓에 전송을 요청할 수 있는지, 없는지 판단할 수 있다.
