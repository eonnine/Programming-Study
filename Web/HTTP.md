
> ## 목차  
>  
> HTTP란?  
> HTTP 메시지 구조  
> Handshake  
> HTTP 통신 과정  
> HTTS  

<br/>

## HTTP(HyperText Transfer Protocol)란?
클라이언트와 서버 간 데이터를 주고받기 위한 규칙이다. 여기서 데이터는 텍스트, 이미지, 동영상 등 모든 종류를 말한다.
TCP(Transmission Control Protocol)와 UDP(User Datagram Protocol)방식이 있으며 80포트를 사용한다.
TCP는 1:1 연결을 지향하며 신뢰할 수 있는 통신을 제공한다. 양단에 연결을 수립한 뒤 데이터를 전송하고 연결을 종료한다. 
반면, UDP는 1:1 혹은 1: 다의 비연결을 지향하며 신뢰할 수 없는 통신을 제공한다. 연결을 수립하는 과정없이 단방향으로 데이터를 전송한다.TCP과 비교했을 때 신뢰할 수 없지만 연결 수립에 필요한 부하가 없고 보다 빠르다.

<br/>

1. **신뢰성**: TCP는 메시지 수신을 확인할 수 있지만 UDP는 확인할 수 없다.
2. **순서**: TCP에서는 메시지가 보내진 순서를 보장하기 위해 재조립하지만 UDP는 메시지 도착 순서를 예측할 수 없다.
3. **부하**: TCP보다 속도가 일반적으로 빠르고 오버헤드가 적다.
4. TCP는 통신의 안전성 및 순차적인 전달이 필요한 경우 사용한다.
5. UDP는 오류의 검사 및 수정이 필요없는 경우에 사용한다. 대표적인 예로 DNS, VOIP, 온라인 게임 등이 있다.

<br/>

## HTTP 메시지 구조
HTTP 통신을 할 때 양쪽의 소통은 평문(ASCII) 메시지로 이루어지며 클라이언트는 서버에 Request 메시지를 보내고 서버는 클라이언트에 Response 메시지를 보낸다. 메세지의 구조는 단순하며 아래와 같다.

### \- Request 메시지
#### Header
+ GET / HTTP/1.1 : HTTP전송 방법과 프로토콜 버전
+ Host: 요청하는 서버 주소
+ User-Agent: OS/브라우저 정보
+ Accept: 클라이언트 이해 가능한 컨텐츠 타입
+ Accept-Language: 클라이언트 인식 언어
+ Accept-Encoding: 클라이언트 인코딩 방법
+ Connection: 전송 완료후 접속 유지 정보 (keep-alive)
+ Upgrade-Insecure-Requests:신호를 보낼때 데이터 암호화 여부
+ Content-Type: 클라이언트에게 반환되어야하는 컨텐츠 유형
+ Content-Length: 본문크기

#### Body
본문은 요청의 마지막 부분에 들어간다. 하지만 모든 요청에 본문이 들어가지는 않는다. 
GET, HEAD, DELETE 처럼 리소스를 가져오는 요청은 보통 본문이 필요가 없기 때문에 Body가 없다.
반면, POST, PUT, PATCH은 업데이트를 위해 서버에 데이터를 전송하므로 Body가 존재한다.

### \- Response 메시지
#### Header
+ HTTP/1.1 200 ok : 프로토콜 버전과 응답상태
+ Access-Control-Allow-Origin: 서버에 타 사이트의 접근을 제한하는 방침
+ Connection: 전송 완료후 접속 유지 정보 (keep-alive)
+ Content-Encoding: 미디어 타입을 압축한 방법
+ Date: 헤더가 만들어진 시간
+ ETag: 버전의 리소스를 식별하는 식별자
+ Keep-Alive: 연결에대한 타임아웃과 요청 최대 개수 정보
+ Last-Modified: 웹 시간을 가지고 있다 수정되었을때만 데이터 변경 ( 캐시연관 )
+ Server: 웹서버로 사용되는 프로그램 이름
+ Set-Cookie: 쿠키 정보
+ Transfer-Encoding: 인코딩 형식 지정
+ X-Frame-Options: frame/iframe/object 허용 여부

#### Body
본문은 응답의 마지막 부분에 들어간다. 하지만 모든 응답에 본문이 들어가지는 않는다. 
201, 204과 같은 상태 코드를 가진 응답에는 보통 본문이 없다.

<br/>

## TCP Handshake
통신할 때 일반적으로 TCP통신을 많이 사용하는데 이 때 연결을 수립할 때는 3way-HandShake, 연결을 종료할 때는 4way-HanShake를 사용한다. 


![tcp](https://user-images.githubusercontent.com/48500660/66812353-b47bd980-ef6d-11e9-8122-c213ce188c19.png)

### \- 3way Handshake
1. 클라이언트는 접속을 요청하는 SYN 패킷을 보낸다. 이때 클라이언트는 SYN 패킷을 보냄과 동시에 SYN/ACK 응답을 기다리기위해 SYN_SENT 상태로 변하게 된다.

2. 서버는 SYN 요청을 받고 클라이언트에게 요청을 수락하는 ACK 패킷과 SYN 패킷을 보내고 SYN_RCVD(SYN_RECEIVED)상태로 변하여 클라이언트가 ACK 패킷을 보낼 때 까지 기다리게 된다.

3. 클라이언트는 서버에 ACK 패킷을 보내고 이 후 ESTABLISHED 상태가 되어 데이터 통신이 가능하게 된다.

* 즉, ACK 패킷의 Acknowledgement Number는 신뢰적 데이터 전송을 위해 사용되는 것이다.
* ISN( Initialized Sequence Number ): 초기 시퀀스 번호 - SYN 패킷의 Sequence Number는 운영체제의 의해서 랜덤하게 생성된다.
* 동기화 요청에 대한 답신 - 클라이언트의 시퀀스 넘버에 +1을 해서 ack로 돌려준다.

### \- 4way Handshake
1. 서버와 클라이언트가 TCP 연결이 되어있는 상태에서 클라이언트가 접속을 끊기 위해 CLOSE() 함수를 호출하게 된다. 이후 CLOSE() 함수를 호출하면서 FIN segment를 보내게 되고 클라이언트는 FIN_WAIT1 상태로 변하게 된다.

2. 서버는 클라이언트가 CLOSE() 한다는 것을 알게되고 CLOSE_WAIT 상태로 바꾼 후 ACK segment를 전송한다. 즉, 클라이언트가 끊을 것이라는 신호를 받았다는 의미이고 CLOSE_WAIT를 통해 자신의 통신이 끝날때까지 기다리는 상태가 된다.

3. ACK segment를 받은 클라이언트는 FIN_WAIT2로 변환되고 이때 서버는 CLOSE() 함수를 호출하고 FIN segment를 클라이언트에게 보낸다.

4. 서버도 연결을 닫았다는 신호를 클라이언트가 수신하면 ACK segment를 보낸 후 TIME_WAIT 상태로 전환된다. 이 후 모든것이 끝나면 CLOSED 상태로 변환된다. 

<br/>

## HTTP 통신 과정
위 개념들을 이해했다면 이제 HTTP 메시지가 전달되고 응답을 받는 과정을 살펴보자.

1. 먼저 브라우저에서 URL을 분석하여 HTTP Request 메세지를 만들고 웹 서버로 전송한다. 이 때 브라우저는 OS를 통해 메시지를 전달하는데 어디로 보낼 지는 IP주소로 지정해야 한다. 이 과정에서 웹 서버의 도메인명을 DNS서버에 조회하여 IP주소를 얻는다.
[도메인명으로 DNS를 통해 IP주소를 어떻게 가져오나](https://goodgid.github.io/Server-DNS/#dns-%EC%84%9C%EB%B2%84%EB%8A%94-2%EC%A2%85%EB%A5%98)
  
2. 프로토콜 스택(운영체제에 내장된 네트워크 제어용 소프트웨어)은 브라우저로부터 받은 메시지를 패킷 속에 저장한 뒤 LAN 어댑터를 통해 패킷을 LAN 케이블로 송출한다.
  
3. LAN 어댑터가 송신한 패킷은 스위칭 허브를 경유하여 인터넷 접속용 라우터에 도착하고 인터넷으로 들어간다.
  
4. 패킷은 인터넷의 입구에 있는 액세스 회선(통신 회선)에 의해 POP(Point Of Presence, 통신사용 라우터)까지 운반된다. 이후 POP 를 거쳐 인터넷의 핵심부로 들어가게 된다. 그리고 수 많은 고속 라우터들 사이로 패킷이 목적지를 향해 흘러가게 된다.
  
5. 패킷은 인터넷 핵심부를 통과하여 웹 서버측의 LAN 에 도착한다. 웹 서버의 방화벽이 도착한 패킷을 검사하고 패킷이 웹 서버까지 가야하는지 가지 않아도 되는지 캐시 서버가 판단하여 굳이 서버까지 가지 않아도 되는 경우를 골라낸다. 페이지의 데이터 중에 다시 이용할 수 있는 것이 있으면 캐시 서버에 저장된다.
  
6. 패킷이 물리적인 웹 서버에 도착하면 웹 서버의 프로토콜 스택은 패킷을 추출하여 메시지를 복원하고 웹 서버 애플리케이션에 넘긴다. 메시지를 받은 웹 서버 애플리케이션은 요청 메시지에 따른 데이터를 응답 메시지에 넣어 클라이언트로 회신한다. 
  
7. 왔던 방식대로 응답 메시지가 클라이언트에게 전달된다.

<br/>

## HTTPS
HTTP는 평문으로 통신을 하기 때문에 보안이 취약한 프로토콜이다. 이 문제 때문에 나타난 것이 HTTPS(Hypertext Transfer Protocol Over Secure Socket Layer)다. 

+ 기본 포트는 443이다.
+ HTTP 는 원래 TCP 와 직접 통신했지만, HTTPS 에서 HTTP는 SSL과 통신하고 SSL이 TCP와 통신한다. SSL을 사용한 HTTPS 는 암호화와 증명서, 안전성 보호를 이용할 수 있게 된다.
+ 대칭키와 공개키 암호화 방식을 혼합하여 보안 통신을 구현한다.
+ 공통키를 공개키 암호화 방식으로 교환한 다음에 그 후 통신은 공통키 암호를 사용하는 방식이다.

공개키 암호에서는 서로 다른 두 개의 키 페어(비밀키, 공개키)를 사용한다.
비밀키는 알려지면 안되는 키이며, 공개키는 누구에게나 알려져도 괜찮은 키다.

### \- 과정
1. 암호를 보내는 측(클라이언트)이 상대의 공개키를 사용해 암호화를 한다.
2. 암호화된 정보를 받아들인 상대(서버)는 자신의 비밀키를 사용해 복호화를 실시한다. 이 방식은 암호를 푸는 비밀키를 통신으로 보낼 필요가 없으며 도청에 의해서 키를 빼앗길 걱정이 없다.

하지만 공개키가 진짜인지 아닌지를 서버에서 증명할 수가 없다.  
이 문제를 해결하기 위해 인증기관(CA : Certificate Authority)과 그 기관이 발행하는 공개키 증명서가 이용되고 있다.  
인증 기관이란 클라이언트와 서버 모두 신뢰하는 제 3자 기관이다.  
공개키를 인증 기관에 제출하면 인증 기관은 제출된 공개키에 디지털 서명을 하고 서명이 끝난 공개키를 만든다.  
그 후 공개키 인증서에 서명이 끝난 공개키를 담는다.  

### \- 단점? No
#### 비용
HTTPS를 꺼려하는 가장 많은 이유 중 하나는 HTTPS 인증서에 대한 비용이다.  
하지만 이러한 고충을 잘 알고 있고 HTTPS의 빠른 보급을 원하는 Google, Mozilla 등의 재단 및 회사들은 [Let's Encrypt](https://letsencrypt.org/)라는 이름의 무료로 HTTPS 인증서를 보급해주는 기관을 만들었다. 이 기관을 통해 손쉽게 HTTPS 인증서를 무료로 사용할 수 있다.

#### 속도
HTTPS를 사용할 경우 HTTP를 사용할 때보다 느린 문제 때문에 민감한 정보를 다루는 페이지에만 HTTPS를 쓰곤 했다.  
하지만 2010년 이후에 사용되는 CPU들은 성능, 속도 측면에서 이미 충분히 빠르기 때문에 HTTP를 사용할 때와 별반 차이가 없다.  
또한, 크로미움 개발 팀이 향후 공개될 크롬 62 버전부터는 HTTP로 사이트에 접속했을 때 시크릿 모드에서는 항상 "안전하지 않음"으로 표시하고 일반 모드와 시크릿 모드 모두 데이터를 입력하려 할 때 "안전하지 않음"을 띄우겠다고 발표했기 때문에 웬만하면 HTTPS를 적용하는 것이 좋다.  
[HTTPS는 HTTP보다 빠르다](https://tech.ssut.me/https-is-faster-than-http/)

<br/>

## Reference
[Website는 어떻게 보여지게되는 걸까? — ( 1 )](https://medium.com/@pks2974/website%EB%8A%94-%EC%96%B4%EB%96%BB%EA%B2%8C-%EB%B3%B4%EC%97%AC%EC%A7%80%EA%B2%8C%EB%90%98%EB%8A%94-%EA%B1%B8%EA%B9%8C-1-108009d4bdb)  
[HTTP 메시지](https://developer.mozilla.org/ko/docs/Web/HTTP/Messages)  
[TCP Handshake란?](https://www.crocus.co.kr/1362)  
[DNS 서버의 역할](https://goodgid.github.io/Server-DNS/#dns-%EC%84%9C%EB%B2%84%EB%8A%94-2%EC%A2%85%EB%A5%98)  
[HTTPS는 HTTP보다 빠르다](https://tech.ssut.me/https-is-faster-than-http/)  
[웹 통신의 큰 흐름](https://github.com/JaeYeopHan/Interview_Question_for_Beginner/tree/master/Network)  
