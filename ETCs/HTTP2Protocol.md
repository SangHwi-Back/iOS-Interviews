주류인 HTTP 1.1의 상위버전으로, 인터넷 기술의 표준화를 주도하는 IETF에 의해 2015년 발표되었다. 정확한 명세는 RFD 7540 에 명시되어 있다. 구글의 SPDY에 기반하고 있다.

목표

클라이언트와 서버가 HTTP/1.0, 2.0 혹은 다른 프로토콜 사용을 자유롭게 오고갈 수 있는 메커니즘 구현
HTTP/1.1과 호환성 유지
다음의 방법들을 이용하여 지연시간을 감소시키고 웹 브라우저 페이지 로드 속도 개선
- HTTP Header 데이터 압축
- 서버 푸시 기술
- 요청을 HTTP 파이프라인으로 처리
- HTTP/1.x의 HOL blocking 문제 해결
- TCP 연결 하나로 여러 요청을 다중화 처리
데스크탑 브라우저, 모바일 웹 브라우저, 웹 API, 웹 서버, 프록시 서버, 리버스 프록시 서버, 방화벽, 콘텐츠 전송 네트워크 등 자주 쓰이는 것들을 지원

HTTP/1.1 의 문제점

HTTP Pipelining
HTTP/1.0은 기본적으로 Connection 하나 당 하나의 요청을 처리할 수 있다. 동시전송이 필요한 멀티미디어 리소스들이 있는 상황에서는 Network Latency를 발생시키게 마련이다.
그래서 HTTP Pipelineing이 도입된다. TCP 한번에 두 개 이상의 HTTP 요청을 담아 Network Latency를 줄이는 방식이다.
정확한 구현이 어렵고, HOL Blocking(앞선 요청에 지연이 발생하면 뒤의 요청도 지연되는 현상을 말함. 서버는 TCP에서 요청을 받은 순서대로 응답을 한다.)이 발생한다는 단점이 있다.

무거운 Header
클라이언트/서버 간에는 수많은 HTTP 요청이 발생할 것이고 Header 정보는 대부분 동일할 것이다. 하지만 HTTP/1.1에서는 Header를 중복으로 보내고 Cookie도 매 요청마다 Header에 포함되어 전송된다. 불필요한 데이터를 주고받는 데 네트워크 자원이 소비되고 있다.

새로운 개념들

HTTP/1.1에서는 평문을 사용하고 개행을 통해 구별하는 것과 달리 HTTP/2.0에서는 바이너리 포맷으로 인코딩 된 Message, Frame을 사용한다.

Stream = 구성된 연결 내에서 전달되는 바이트의 양방향 흐름. 하나 이상의 메시지가 전달 가능하다.
Message = 논리적 요청 또는 응답 메시지에 매핑되는 프레임의 전체 시퀀스를 뜻한다.
Frame = HTTP/2.0 에서 통신의 최소 단위. 각 최소 단위에는 하나의 프레임 헤더가 포함된다. 이 프레임 헤더는 최소한으로 프레임이 속하는 스트림을 식별한다. HEADERS Type Frame, DATA Type Frame으로 나뉜다.

![](../_Images/HTTP2_Image1.png)

https://velog.io/@taesunny/HTTP2HTTP-2.0-%EC%A0%95%EB%A6%AC