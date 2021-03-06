# Polling 폴링 방식

<p align="center"><img src = "http://2.bp.blogspot.com/-cvWY81etsao/ViZSUVxywxI/AAAAAAAAMHo/wxrd6dIntM8/s320/HttpPolling.gif" width="50%" height="100%"></img></p>   

* 클라이언트가 n초 간격으로 **http request를 서버로 계속 날려서 response를 전달받는 방식**
* HTTP는 단발성 통신이기 때문에 header 가 매우 무거운 프로토콜 중 하나로 이 프로토콜이 **계속해서 request를 날리면 서버의 부담 증가**

# Long Polling 롱폴링방식
<p align="center"><img src = "http://2.bp.blogspot.com/-eL9rxi8th2A/ViZSW0ggEwI/AAAAAAAAMH4/k4S4-dRz3t4/s320/HttpLongPolling.gif" width="50%" height="100%"></img></p>   

* 클라이언트가 서버로 일단 http 요청을 날린다.
이 상태로 계속 기다리다가 서버에서 해당 클라이언트로 **전달할 이벤트가 생기면 그순간 응답 메세지를 전달하면서 연결이 종료**된다. 

* 하지만 곧바로 클라이언트는 다시 요청을 날려 서버의 다음 이벤트를 기다리는 방식

* 이벤트가 생길때 서버에서 응답을 보내주고, 일반 폴링 방식보다 
서버에 부담이 줄겠지만 만약에 서버측에서 이벤트가 자주생겨 클라이언트가 요청을 보내는 시간이 짧아진다면 폴링과 별 차이가 없어진다.

* 또 다수의 클라이언트가 동시에 이벤트가 발생될 경우 곧바로 동시에 다수의 클라이언트에게 서버로 요청이 오기 때문에
서버의 부담이 단시간에 급증

1. 클라이언트가 웹서버로 HTTP 요청을보낸다
2. 요청을 받은 웹서버는 데이터가 있을 때 까지(이벤트가 발생할 때 까지) 기다린다
3. 데이터 혹은 이벤트가 발생하면 클라이언트로 HTTP응답을보낸다
4. 응답을 받은 웹브라우저(클라이언트)는 데이터를 출력하고 HTTP연결을 끊는다
5. 그리고 웹브라우저(클라이언트)는 다시 HTTP요청을 보낸다
6. 기다린다
7. 반복

# Streaming
<p align="center"><img src = "http://4.bp.blogspot.com/-sRVlAdeU-Kw/ViZSWw-wB2I/AAAAAAAAMH0/3CmKGISDV-A/s320/HttpStreaming.gif" width="50%" height="100%"></img></p>   

* 클라이언트가 HTTP 요청을 서버에게 보내고 **서버는 응답을 끊임 없이 흘려보낸다(trickles out).**
* 즉 long-polling처럼 **요청을 끊지 않고 하나의 HTTP연결에서 데이터를 보내는 방식.**

🙋‍♂️  long-polling에 비해 서버에서 메시지를 보내고 다시 http 요청을 보내지 않아도 연결이 이어져 있으므로 웹소켓의 대안이 될 수 있다?   

> 아니다. **스트리밍에서는 하나의 TCP포트로 읽기와 쓰기를 동시에 할 수 없다.**    
즉 서버에서 클라이언트로 메세지를 보낼 수 있으나 클라이언트에서 서버로 메세지를 보낼려면 꼼수가 필요하다.    
예를 들면 채팅 서비스를 구현할 경우 메시지 입력은 TCP가 아닌 포트를 통해 받아야할 것. 

🙏 간단히 말해 아래와 같은 문제가 있다.
* HTTP 통신규약 : 서버에게 요청보내고 응답 받으면 연결이 끊긴다. 서버-클라이언트 실시간 상호작용이 안된다.   

***그래서 실시간 상호작용할 수 있게 효과만 내본 기술들*** 

* Polling 기술 : 서버로 계속해서 요청보내기(근데 서버에 이벤트 없어도 계속 요청보냄>서버,클라이언트 무리)
* Long polling기술 : 서버에 요청보내고 이벤트가 생겨 응답 받을 때 까지 연결안끊기. 응답받으면 끊고 다시 새요청보냄(근데 새 이벤트 생기면 모든 사용자가 연결 끊고 동시에 새요청보내게됨>서버무리)
* Streaming 기술 : 서버에 요청보내고 끊기지 않은 연결상태에서 끊임없이 데이터받기(근데 클라이언트가 서버로 요청을 보내기 좀 힘듬. 스트리밍에서는 하나의 포트써서 읽고 쓰기 동시에 안됨)

여기서 실시간 상호작용처럼 꼼수를 부린 long polling 과 streaming은 HTTP Comet으로도 불린다. 
이는 특정기술 이름이 아니고 HTTP에서 데이터를 push하기 위한 방식자체기술을 어울러 말한다. 추가로 이 HTTP COMET들은 TCP 연결과정(Threeway handshake)에서의 오버헤드때문에 **비효율적!**

❗ 하지만 **웹 소켓은 실시간 네트워킹이 가능하다.**

# WebSocket 웹 소켓
<p align="center"><img src = "https://miro.medium.com/max/533/1*UDwuuzptZlTpH-jJ9rDYgw.png" width="50%" height="100%"></img></p>   

* 웹소켓은 **HTTP 기반**으로 하면서 **HTTP의 문제점을 해결하는 것을 목표**로 나온 기술이다.
이전의 통신과 달리 **이중통신(full-duplex 통신, 2-way communication)** 즉 **수신과 송신을 동시**에 처리할 수 있다.
socket connection을 유지하기 떄문에 **양방향 통신**, 데이터 전송이 가능하고 HTML5에 포함돼 프로토콜로 제정되어 있다.

✔ 기존의 TCP Socket과 다른점
* 웹소켓은 최초 접속이 일반 http 요청을 이용한 handshaking으로 이루어진다는 점이다. 
또 TCP socket에서는 바이트 스트림을 사용하지만 **웹소켓을 통해 전달되는 텍스트들은 UTF-8 포멧**을 가짐.

* Stateful : 클라이언트와 한번 연결이 되면 계속 같은 라인 사용해 통신하기 때문에 
**HTTP사용시 필요없이 발생되는 HTTP와 TCP연결 트래픽을 피할 수 있다.**
(웹소켓은 HTTP와 달리 **최초 접속을 제외하고 헤더정보를 보내지 않기에** 네트워크 비용측면에서 이득)

* HTTP 요청을 그대로 사용하기에 기존 80(HTTP포트), 443(HTTPS)포트로 접속하므로 추가 방화벽을 열지 않아도 되고, 
**HTTP의 규격인 CORS적용이나 인증등의 과정을 기존과 동일하게 가져갈 수 있다.**

> 현재 이 웹소켓기술을 real-time web application
(서버 또는 클라이언트 쪽 데이터가 실시간으로 업데이트 되는 웹 어플리케이션)에서 많이 사용한다.

## 작동원리
<p align="center"><img src = "https://t1.daumcdn.net/cfile/tistory/99C87F335C79684D1D" width="50%" height="100%"></img></p>   

1. 클라이언트와 서버간에 전이중 통신을 수행하려면 클라이언트가 서버로 **HTTP UPGRADE 요청**을 보내야 한다.
이를 **Web Socket Protocol Hand Shake**라고한다.

2. 서버가 커넥션을 UPGRADE 할 수 있는 경우,  HTTP 101 응답을 클라이언트에게 보낸다.

3. 서버는 핸드 쉐이크가 성공적으로 수행되었다고 판단하고,
서버와 클라이언트 사이의 커넥션을 웹 소켓 프로토콜로 UPGRADE 한다. 클라이언트와 서버 사이의 HTTP 101 응답이 전달되는 순간,
서버와 클라이언트 사이의 커넥션은 HTTP 프로토콜이라고 하지 않는다. 그리고 이순간 양방향 통신이 가능해진다.

4.  웹 소켓으로 연결된 모든 클라이언트는 다른 클라이언트에게 커넥션을 끊는 요청을 전송할 수 있다.

> 그림을 보면 위에서 나온 방식으로 클라이언트와 서버간의 **전이중 통신을 지원**한다는 것을 알 수 있다. 
그러면 가운데 API GateWay는 무엇일까?!

>**API GateWay**는 **EndPoint 관리를 지원해주는 프록시 서버**이다. 
클라이언트로부터 수신한 메세지 내용을 바탕으로 적절한 서버를 호출한다.
그림을 보면 클라이언트는 서버와 **직접적으로 연결을 맺는 것이 아닌**, **API GateWay를 경유해서 연결**을 하고 있는 것을 볼 수 있다. 
그리고 API GateWay는 **클라이언트의 종류에 따라 여러 프로토콜이 지원 가능**하다.   


* 대표적인 사용 예

1. 페이스북과 같은 SNS 어플리케이션
2. LOL 같은 멀티플레이어 게임들
3. 구글 Doc 같이 여러 명이 동시 접속해서 수정할 수 있는 Tool
4. 증권 거래 정보 사이트 및 어플
5. 화상 채팅 어플
6. 위치 기반 어플
