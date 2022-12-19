# Network

# ISN(Initial Sequence Number)가 난수인 이유
## 1. 보안
모든 TCP 연결이 0부터 시작한다면, 번호 예측이 쉬워 TCP Connection Hijacking이 빈번할 것이다.

## 2. 같은 IP 연결에서 번호 중복
Sender가 ISN을 0으로 설정된 100B 세그먼트를 ISN을 보냈다. 시퀀스 번호가 100인 세그먼트(응답으로 와야 할)가 지연되거나 손실된 후 TCP 연결이 닫혔다고 가정한다. 

이전 연결과 동일한 IP 포트를 사용하는 새 연결이 설정되고 ISN이 0으로 초기화된다. 만약 시퀀스 번호가 100인 세그먼트가 도착한다면, 이전 연결으로부터 오는 것으로 인식할 수도 있다. 

<hr>

출처
- [In TCP connection, the initial sequence number is not '0' (zero). Why?](https://www.quora.com/In-TCP-connection-the-initial-sequence-number-is-not-0-zero-Why)
- [Wrap Around Concept and TCP Sequence Number](https://www.geeksforgeeks.org/wrap-around-concept-and-tcp-sequence-number/)

<hr>

# 세션(Session) vs 쿠키(Cookie)
## 세션(Session)
### 세션(Session)이란
- 세션은 웹사이트의 여러 페이지에 걸쳐 사용될 정보를 서버에 임시적으로 저장하는데 사용된다. 사용자가 특정 네트워크 응용 프로그램에 로그인할 때 시작되고, 로그아웃하거나 시스템을 종료할 때 종료된다. 
- 사용자가 인터넷을 통해 응용 프로그램에서 작업 할 때, 웹 서버는 HTTP 프로토콜이 상태를 유지하지 않기 때문에 사용자를 알지 못한다. 응용 프로그램의 한 페이지에서 사용자가 제공한 정보는 다른 페이지로 전송되지 않는다. 이러한 제한을 제거하기 위해 세션이 사용되는데, 사용자가 웹사이트에 처음 진입할 때마다 세션이 시작된다.
- 사용자 정보가 세션 변수에 저장되며, 이러한 변수는 객체의 모든 유형의 값 또는 데이터 유형을 저장할 수 있다.
- 세션 값은 이진 형식이나 암호화된 형식으로 저장되며 서버에서만 암호를 해독할 수 있기 때문에 매우 안전하다. 세션 값은 사용자가 시스템을 종료하거나 응용 프로그램에서 로그아웃할 때 자동으로 제거된다. 값을 영구적으로 저장하려면 데이터베이스에 값을 저장해야 한다.
- 각 세션은 각 사용자마다 고유하며 응용 프로그램에서 세션의 수를 제한하지 않는다.
- 사용자는 서버 내부에 저장된 고유 번호인 sessionID로 식별된다. 쿠키, form 필드, URL로 저장된다.

### 세션(Session) 동작 과정

<img src="../imgs/network-session.png">

1. 클라이언트가 GET/POST 메서드를 통해 서버에 요청을 보낸다.
2. 서버에서 sessionID를 생성하고 데이터베이스에 저장한다. 클라이언트에 대한 응답으로 쿠키와 함께 sessionID를 반환한다.
3. 브라우저에 저장된 sessionID가 포함된 쿠키는 서버로 다시 전송된다. 서버는 sessionID가 데이터베이스에 저장된 sessionID와 일치하면 응답으로 HTTP 상태코드 200을 전송한다.

### 세션(Session) 사용 이유
- 세션은 서버를 통해 UserID와 같은 정보를 안전하게 저장하는 데 사용된다. 
- 한 웹페이지에서 다른 웹 페이지로 정보를 전송하는 데 사용된다.
- 쿠키를 지원하지 않는 브라우저에서 쿠키 대신 사용하며, 쿠키보다 안전한 방식으로 변수를 저장할 수 있다.

## 쿠키(Cookie)
### 쿠키(Cookie)란
- 쿠키는 사용자의 컴퓨터에 저장된 작은 텍스트 파일이다. 쿠키의 최대 크기는 4KB이다. HTTP 쿠키, 웹 쿠키, 인터넷 쿠키라고도 한다. 사용자가 웹 사이트를 처음 방문할 때마다 사이트는 쿠키 형태의 데이터 패킷을 사용자의 컴퓨터로 보낸다.
- 쿠키는 웹 사이트가 사이트를 방문할 때 사용자의 검색 기록 또는 장바구니 정보를 추적하는 데 도움이 된다.
- 문자열 데이터 유형만 저장한다.
- 쿠키에 저장된 정보는 클라이언트 측에서 텍스트 형식으로 저장되어 누구나 읽을 수 있기 때문에 안전하지 않다.
- 요구 사항에 따라 쿠키를 활성화하거나 비활성화할 수 있다.
- 사용자가 생성한 쿠키는 사용자에게만 표시되며 다른 사용자는 해당 쿠키를 볼 수 없다.
- 쿠키는 HTTP 헤더를 사용하며 서버와 브라우저간에 만들어지고 공유된다.
- 쿠키가 저장되는 경로는 브라우저에 의해 결정된다. 인터넷 탐색기는 일반적으로 임시 인터넷 파일 폴더에 쿠키를 저장하기 때문이다.
- YouTube 채널을 방문하여 노래를 검색할 때 다음에 YouTube를 방문할 때마다 쿠키가 검색 기록을 읽고 유사한 노래 또는 마지막으로 재생된 노래를 표시합니다.

### 쿠키(Cookie) 만들기

```
setcookie(name, value, expire, path, domain, secure, httponly);  
```
name 인수만 필수이며 다른 인수는 선택 사항이다.
```
Example:

setcookie("Userid", "1005", "time()+3600");
```

### 쿠키(Cookie) 사용 이유
HTTP는 stateless 프로토콜이므로 사용자 정보를 저장하지 않는다. 이를 위해 쿠키를 사용할 수 있다. 쿠키는 사용자의 컴퓨터에 정보를 저장하고 응용 프로그램의 상태를 추적할 수 있게 도와준다.

## 세션(Session)과 쿠키(Cookie) 주요 차이점
- 세션은 사용자 정보를 저장하는 server-side(서버 측) 파일인 반면 쿠키는 로컬 컴퓨터의 사용자 정보를 저장하는 client-side(클라이언트 측) 파일이다.
- 세션은 쿠키에 종속되지만 쿠키는 세션에 종속되지 않는다.
- 사용자가 브라우저를 닫거나 응용프로그램에서 로그아웃하면 세션이 종료되는 반면 쿠키는 설정된 시간에 만료된다.
- 쿠키의 크기는 4KB로 제한되어 있는 반면 세션은 사용자가 원하는 만큼 데이터를 저장할 수 있다.
  - 그러나 스크립트가 한 번에 사용할 수 있는 최대 메모리 제한이 있으며 128MB다.

## 결론
세션은 사용자 정보를 서버 측에 일시적으로 저장하는 반면 쿠키는 만료 될 때까지 사용자 컴퓨터에 정보를 저장하는 방법이다.

<hr>

출처
- [Session vs. Cookies| Difference between Session and Cookies](https://www.javatpoint.com/session-vs-cookies)

<hr>

# REST? RESTful?
Representational State Transfer(REST)는 API 작동 방식에 대한 조건을 부과하는 소프트웨어 아키텍처이다. REST는 처음에 인터넷과 같은 복잡한 네트워크에서 통신을 관리하기 위한 지침으로 만들어졌다. REST 기반 아키텍처를 사용하여 대규모의 고성능 통신을 안정적으로 지원할 수 있다. 쉽게 구현하고 수정할 수 있어 모든 API 시스템을 파악하고 여러 플랫폼에서 사용할 수 있다. 

API 개발자는 여러 아키텍처를 사용하여 API를 설계할 수 있다. REST 아키텍처 스타일을 따르는 API를 REST API라고 한다. REST 아키텍처를 구현하는 웹 서비스를 RESTful 웹 서비스라고 한다. RESTful API라는 용어는 일반적으로 RESTful 웹 API를 나타낸다. 하지만 REST API와 RESTful API라는 용어는 같은 의미로 사용할 수 있다.

## REST의 아키텍처 제약
### 1. Client-Server
클라이언트-서버 스타일은 사용자 인터페이스에 대한 관심(concern)을 데이터 저장에 대한 관심으로부터 분리함으로써 클라이언트의 이식성과 서버의 규모확장성을 개선한다.

### 2. Stateless
클라이언트와 서버의 통신에는 상태가 없어야한다. 모든 요청은 필요한 모든 정보를 담고 있어야한다. 요청 하나만 봐도 바로 뭔지 알 수 있으므로 가시성이 개선되고, task 실패시 복원이 쉬우므로 신뢰성이 개선되며, 상태를 저장할 필요가 없으므로 규모확장성이 개선된다.

### 3. Cacheable
캐시가 가능해야한다. 즉 모든 서버 응답은 캐시 가능한지 그렇지 아닌지 알 수 있어야한다. 호율, 규모확장성, 사용자 입장에서의 성능이 개선된다.

### 4. Uniform interface
구성요소(클라이언트, 서버 등) 사이의 인터페이스는 균일(uniform)해야한다. 인터페이스를 일반화함으로써, 전체 시스템 아키텍처가 단순해지고, 상호작용의 가시성이 개선되며, 구현과 서비스가 분리되므로 독립적인 진화가 가능해진다.

### 5. Layered system
계층(hierarchical layers)으로 구성이 가능해야하며, 각 레이어에 속한 구성요소는 인접하지 않은 레이어의 구성요소를 볼 수 없어야한다. 따라서 load-balancer나 proxy를 쉽게 추가하여 보안이나 성능을 향상시킬 수 있다. 

<hr>

출처
- [바쁜 개발자들을 위한 REST 논문 요약](https://blog.npcode.com/2017/03/02/%EB%B0%94%EC%81%9C-%EA%B0%9C%EB%B0%9C%EC%9E%90%EB%93%A4%EC%9D%84-%EC%9C%84%ED%95%9C-rest-%EB%85%BC%EB%AC%B8-%EC%9A%94%EC%95%BD/)
- [RESTful API란 무엇입니까?](https://aws.amazon.com/ko/what-is/restful-api/)
- [What Are RESTful Web Services?](https://docs.oracle.com/javaee/6/tutorial/doc/gijqy.html)
- [REST API (RESTful API)](https://www.techtarget.com/searchapparchitecture/definition/RESTful-API)

<hr>

# 소켓
## 개요 
TCP는 인터넷의 client-server 응용 프로그램이 서로 통신하는 데 사용하는 신뢰할 수 있는 point-to-point 통신 채널을 제공한다. TCP를 통해 통신하기 위해 클라이언트 프로그램과 서버 프로그램은 서로 연결을 설정한다. 각 프로그램은 소켓을 연결 끝에 바인딩한다. 통신을 위해 클라이언트와 서버는 각각 연결에 바인딩된 소켓에서 읽고 쓴다.

소켓은 네트워크에서 실행되는 두 프로그램 사이의 양방향 통신 링크의 한 엔드 포인트이다. Socket 클래스는 클라이언트 프로그램과 클라이언트 프로그램 간의 연결을 나타내는 데 사용된다. java.net 패키지는 Socket 클래스와 ServerSocket 클래스를 제공한다. Socket은 연결의 클라이언트 측을, ServerSocket은 연결의 서버 측을 구현한다. 

웹에 연결하는 경우 Socket 클래스보다 URL 클래스와 관련된 클래스(URLConnection, URLNcoder)가 더 적합할 수 있다. 실제로 URL은 웹에 대한 비교적 높은 수준의 연결이며 기본 구현의 일부로 소켓을 사용한다. 

## 소켓이란

<img src="../imgs/network-socketconnect.gif">

서버: 일반적으로 서버는 특정 컴퓨터에서 실행되며 특정 포트 번호에 바인딩된 소켓이 있다. 서버는 클라이언트가 연결 요청을 할 때까지 소켓을 대기한다. 

클라이언트: 클라이언트는 서버가 실행 중인 컴퓨터의 호스트 이름과 서버가 수신 중인 포트 번호를 알고 있다. 연결 요청을 위해 클라이언트는 서버의 컴퓨터 및 포트에 있는 서버와 만남을 시도한다. 또한 클라이언트는 서버에 대해 자신을 식별하여 이 연결 중에 사용할 로컬 포트 번호를 바인딩해야 한다. 일반적으로 시스템에서 할당한다. 

<img src="../imgs/network-socketconnect.gif">

모든 것이 잘 되면, 서버는 연결을 승인한다. 승인 시 서버는 동일한 로컬 포트에 바인딩된 새 소켓을 가져오고, 원격 엔드포인트를 클라이언트의 주소와 포트로 설정한다. 연결된 클라이언트의 요구를 처리하면서 연결 요청을 위해 원래 소켓을 계속 listen 할 수 있도록 새로운 소켓이 필요하다.

클라이언트 측에서는 연결이 허용되면 소켓이 성공적으로 생성되고 클라이언트는 소켓을 사용하여 서버와 통신할 수 있다. 

클라이언트와 서버는 이제 소켓에 write 하거나 read 함으로써 통신할 수 있다.

```
Definition:

소켓은 네트워크에서 실행되는 두 프로그램 사이의 양방향 통신 링크의 한 끝점이다. 소켓은 포트 번호에 바인딩되어 TCP 계층이 데이터를 전송할 응용 프로그램을 식별할 수 있다.
```

엔드포인트는 IP 주소와 포트 번호의 조합이다. 모든 TCP 연결은 두 개의 엔드포인트에 의해 고유하게 식별될 수 있다. 일허게 하면 호스트와 서버 간에 여러 연결을 가질 수 있다. 

<hr>

출처
- [All About Sockets](https://docs.oracle.com/javase/tutorial/networking/sockets/index.html)

<hr>

# WebSocket vs Socket.io
## WebSocket
### 배경 - HTTP의 한계
초기 웹의 목적은 단순한 문서전달이었다. HTTP는 요청한대로 응답을 보내주기만 하는 단순한 프로토콜이다. 하지만 인터넷이 발전하면서 게임, 채팅 등 요청-응답 이상의 실시간 통신이 필요했다. HTTP 프로토콜은 매 요청과 응답마다 연결을 수립하고 끊는 과정을 반복해야 했기 때문에 유사한 통신을 계속 반복해야 한다는 비효율성 문제가 있었다. 

### 웹소켓의 탄생
HTTP를 이용한 실시간 통신의 문제를 해결하기 위해 HTML5부터 웹소켓이 등장했다. 웹소켓은 실시간 양방향 통신을 지원하며 한 번 연결이 수립되면 클라이언트와 서버 모두 자유롭게 데이터를 보낼 수 있다. 

### 웹소켓 프로토콜
웹소켓은 HTTP와 같은 OSI 모델의 7계층에 위치하는 프로토콜이며, 4계층의 TCP에 의존한다.

HTTP 프로토콜을 이용할 때 "http"를 이용하는 것처럼, 웹소켓을 이용할 때 "ws"를 이용한다. 또한 보완을 강화한 "https"를 사용하는 것처럼, "ws"에 비해 보완을 강화한 "wss"를 사용할 수 있다. 

HTTP 프로토콜을 이용해서 연결을 수립하며 연결된 이후에도 연결을 할 때 사용했던 80 포트와 443 포트를 이용한다. 연결 수립은 핸드쉐이크를 통해 이루어지며 핸드쉐이크 시 HTTP를 이용한다.

### 웹소켓 핸드쉐이크
핸드쉐이크는 한 번의 HTTP 요청과 HTTP 응답으로 이루어진다. 핸드쉐이크가 끝나면 HTTP 프로토콜을 웹소켓 프로토콜로 변환하여 통신을 하는 구조다.

## Socket.IO
Socket.IO는 클라이언트와 서버 간의 낮은 지연 시간, 양방향 및 이벤트 기반 통신을 가능하게 하는 라이브러리다. 웹소켓 프로토콜 위에 구축되었으며 HTTP long-polling 또는 자동 재연결에 대한 fallback과 같은 추가적인 보장을 제공한다. 

Socket.IO는 Websocket을 사용하지만, 각 패킷에 추가적인 메타데이터를 추가한다. 그것이 Websocket Client가 Socket.IO Server에 성공적인 연결을 할 수 없는 이유다. Socket.IO Client도 Websocket Server에 연결할 수 없다.

### Socket.IO 기능
- HTTP long-polling fallback
- Automatic reconnection
- Packet buffering
- Acknowledgements
- Broadcasting
- Multiplexing


<hr>

출처
- [웹소켓에 대해 알아보자](https://tecoble.techcourse.co.kr/post/2020-09-20-websocket/)
- [What Socket.IO is](https://socket.io/docs/v4/#is-socketio-still-needed-today)
- [웹소켓과 socket.io](https://www.peterkimzz.com/websocket-vs-socket-io/)
- [WebSocket과 Socket.io](https://d2.naver.com/helloworld/1336)

<hr>

# IPv4 vs IPv6
## IPv6 개요
많은 사람들이 휴대 전화 및 휴대용 컴퓨터 등과 같은 모바일 컴퓨터를 사용하게 됨에 따라, 무선 사용자의 증가하는 요구가 IPv4 주소 고갈의 원인이 되고 있다. 따라서 인터넷 프로토콜 버전 4(IPv4)를 대체하는 인터넷 프로토콜 버전 6(IPv6)가 나왔다. 

### 주소
- IPv4는 32비트 길이(4바이트). `nnn.nnn.nnn.nnn` 의 형태. (0<=nnn<=255). 주소 클래스에 따라 다른 네트워크 및 호스트 부분으로 구성됩니다.
- IPv6는 128비트 길이(16바이트). `xxxx:xxxx:xxxx:xxxx:xxxx:xxxx:xxxx:xxxx` 의 형태. 기본 구조는 네트워크 번호용 64비트와 호스트 번호용 64비트. 종종 IPv6 주소의 호스트 부분(또는 그 일부)은 MAC 주소나 인터페이스 ID에서 파생.

### IP 헤더
- IPv4는 현재 IP 옵션에 따라 20-60 바이트의 가변 길이
- IPv6는 40바이트의 고정 길이. IP 헤더 옵션이 없다. 

### 주소 유형
- IPv4 주소는 유니캐스트 주소, 멀티캐스트 주소 및 브로드캐스트 주소의 세 가지 기본 유형으로 분류
- IPv6 주소는 유니캐스트 주소, 멀티캐스트 주소 및 애니캐스트 주소의 세 가지 기본 유형으로 분류

### ARP(주소 해석 프로토콜)
- IPv4는 ARP(주소 해석 프로토콜)를 사용하여 IPv4 주소와 연관된 MAC 또는 링크 주소와 같은 물리적 주소를 찾는다.
- IPv6은 ICMPv6(Internet Control Message Protocol version 6)을 사용하여 스테이트리스 자동 구성 및 인접 노드 발견을 위한 알고리즘의 일부로서 IP 자체 내에 이러한 기능을 내장시킨다. 따라서 ARP6과 같은 것은 없다.

<hr>

출처
- [IPv4와 IPv6의 비교](https://www.ibm.com/docs/ko/i/7.3?topic=6-comparison-ipv4-ipv6)

<hr>

# Mac Address
## Mac Address란
- Mac 주소는 지정된 네트워크의 각 장치를 고유하게 식별하는 물리적 주소이다. 두 개의 네트워크 장치 간에 통신하려면 IP 주소와 MAC 주소 두 개가 필요하다. 인터넷에 연결할 수 있는 각 장치의 NIC(Network Interface card)에 할당된다.
- Media Access Control의 약자로 물리적 주소, 하드웨어 주소, BIA(Burned In Address)라고도 한다.
- 글로벌적으로 고유하다. 00 : 0a : 95 : 9d : 67 : 16과 같이 각 장치의 16 진수 형식으로 표시된다.
- 길이는 12. 48비트(6바이트). 이 중 처음 24비트는 OUI(Organization Unique Identifier)에 사용되며 24비트는 NIC/벤더별로 사용된다.
- OSI 7모델 중 데이터 링크 계층에서 작동한다.
- 제조 시 장비 공급업체에서 제공하며 NIC에 내장되어 변경할 수 없다.
- ARP 프로토콜은 논리 주소를 MAC 주소(물리적 주소)와 연결하는데 사용된다.

## Mac Address vs IP Address
- MAC는 Media Access Control, IP는 Internet Protocol
- MAC 주소는 제조업체에서 제공하는 고유 주소, IP 주소는 ISP(Internet Service Provider)에서 제공하는 논리적 주소
- MAC 주소는 네트워크 내에서 디바이스를 식별하는 데 사용되는 디바이스 NIC의 물리적 주소, IP 주소는 인터넷에서 네트워크나 디바이스를 식별하는 논리적 주소
- MAC 주소는 데이터 링크 계층에서 작동하고, IP 주소는 네트워크 계층에서 작동한다.
- MAC 주소는 6 바이트 16진수 주소, IPv4의 경우 4바이트, IPv6의 경우 8바이트다.
  
<hr>

출처
- [What is MAC Address?](https://www.javatpoint.com/what-is-mac-address)

# SOP (Same-origin policy)
## SOP(Same-origin policy)란
- 한 Origin에서 로드된 문서 또는 스크립트가 다른 Origin의 리소스와 상호작용할 수 있는 방법을 제한하는 중요한 보안 메커니즘이다. 보안을 위협하는 문서를 격리하여, 보안 위협으로부터 보호할 수 있다.
  - Origin은 출처(URL)을 말한다.
- 악의적인 웹 사이트가 타 웹 메일 서비스나 회사 인트라넷에 접근해 데이터를 읽고 해커에게 데이터를 전달하는 것을 막는다.

## Origin(출처)의 정의
- 두 개의 URL이 동일한 `protocol`, `port`, `host` 를 가지고 있으면 같은 Origin이다.
- `http://store.company.com/dir/page.html` 에 대한 예시다. 
  <img src="../imgs/network-origin.png">

## Cross-origin network access
- SOP 정책은 `XMLHttpRequest` 또는 `<img>` 요소를 사용할 때와 같이 두 개의 Origins 간의 상호작용을 제어한다. 이러한 상호작용들은 일반적으로 세 가지 범주로 분류된다.
- `Cross-origin writes`는 일반적으로 허용된다. 예를 들어 links, redirects, form submissions가 있다. 일부 HTTP 요청에는 preflight가 필요하다.
- `Cross-origin embedding`은 일반적으로 허용된다. 
- `Cross-origin reads`는 일반적으로 허용되지 않지만, 읽기 접근은 embedding에 의해 유출되는 경우가 많다. 예를 들어 embedded 이미지의 치수, embedded 스크립트의 작업, embedded 리소스의 가용성을 읽을 수 있다.
  

### `Cross-origin embedding`의 예시
- `<script src="…"></script>`이 포함된 JavaScript. syntax error에 대한 내용은 Same-origin 스크립트에서만 사용할 수 있다.
- `<link rel="stylesheet" href=..">`와 함께 적용된 CSS. 올바른 Content-Type 헤더가 필요하다. 브라우저는 MIME 타입이 올바르지 않고 유효한 CSS 구성으로 시작하지 않는 cross-origin은 style sheet 로드를 block한다. 
- `<img>`에 의해 로드된 이미지.
- `<video>`와 `<audio>`로 로드된 미디어.

### Cross-origin access 허용하는 방법
- [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)를 사용해서 허용한다. CORS는 서버가 브라우저가 컨텐츠 로드를 허용해야 하는 host드들을 지정할 수 있게 해주는 HTTP의 한 부분이다.

<hr>

출처
- [Same-origin policy](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy#cross-origin_script_api_access)

<hr>

# Stateless와 Connectionless
## Stateless(무상태)
- stateful은 서버가 클라이언트의 이전 상태를 보존한다는 의미다. 반대로 Stateless는 서버가 클라이언트의 이전 상태를 보존하지 않다는 의미다. 
- stateful의 경우 항상 같은 서버가 유지되어야 하지만 stateless는 상태를 보관하지 않기 때문에 클라이언트의 요청에 어느 서버가 응답해도 상관이 없다. 따라서 클라이언트의 요청이 대폭 증가해도 서버를 증설해 해결할 수 있다.
- 하지만 모든 것을 stateless로 설계할 수 없다. 단순히 로그인만 보더라도 사용자가 로그인한 상태를 서버가 유지시켜 주어야 한다. 이를 브라우저 쿠키나 서버 세션 등을 사용해 상태를 유지한다.

## Connectionless(비연결성)
- connectionless는 클라이언트가 서버에 요청을 하고 응답을 받으면 바로 TCP/IP connection을 끊어 연결을 유지하지 않는 것이다. 
- 서버의 자원을 효율적으로 관리하고, 수많은 클라이언트의 요청에 대응할 수 있게 해준다.
- HTTP는 connectionless 모델을 기본으로 한다. 

### Connectionless의 한계
- 연결이 끊어지고 새로 연결될 때 마다 TCP/IP 연결을 위한 3-way handshake의 시간 비용이 추가된다. 
- HTML 뿐만 아니라 JavaScript, CSS, 이미지 등 수많은 자원이 함께 다운로드된다. 이는 HTML을 받기 위해 연결하고 종류, JavaScript 파일을 받기 위해 연결 및 종료가 이어진다. 

### 해결
- HTTP 1.X 시대에는 HTTP 지속 연결(HTTP Persistent Connection)으로 문제를 해결했다. 
- 지속 연결은 요청에 따라 연결된 이후 일정 시간 연결을 유지하거나 여러 개의 요청(HTML, JavaScript, 이미지 등)에 대한 응답이 다 올 때까지 기다린 후 연결을 종료하는 것이다. 
- HTTP/2 버전은 멀티플렉싱(Multiplexing) 기능으로 단일 TCP 연결을 통해 (Persistent Connection in HTTP1.X), 다수의 HTTP 요청과 응답(HTTP Pipelining in HTTP1.1)이 클라이언트와 서버 사이에 응답 지연(HOL, Header of Line Blocking) 없이 Stream 형태로 주고 받을 수 있는 기술적 토대를 만들게 되었다. 따라서 HTTP/2 버전을 사용한다면 Persisten Connection에 대해 더 이상 고민할 필요가 없어졌다. 

<hr>

출처
- [HTTP, Stateless, Connectionless, HTTP 메시지 개념](https://velog.io/@duarufp06/HTTP-Stateless-Connectionless-HTTP-%EB%A9%94%EC%8B%9C%EC%A7%80-%EA%B0%9C%EB%85%90)
- [HTTP Persistent Connection](https://brunch.co.kr/@sangjinkang/4)
- [(번역) HTTP/3 - 차세대 웹 프로토콜에 대해 알아야하는 모든 것](https://velog.io/@sehyunny/everthing-you-need-to-know-about-http3)

<hr>

# 로드밸런서(Load Balancer)
## 로드밸런서(Load Balancer)란
- 서버에 가해지는 부하(=로드)를 분산(=밸런싱)해주는 장치 또는 기술을 통칭한다.
- 클라이언트의 서버풀(Server Pool, 분산 네트워크를 구성하는 서버들의 그룹) 사이에 존재하며, 한 대의 서버로 부하가 집중되지 않도록 트래픽을 관리해 각각의 서버가 최적의 성능을 보일 수 있도록 한다.

## 로드밸런싱은 모든 경우에 항상 필요한지?
- 로드밸런싱은 여러 대의 서버를 두고 서비스를 제공하는 분산 처리 시스템에서 필요한 기술이다.
- 서비스의 제공 초기 단계라면 적은 수의 클라이언트로 인해 서버 한 대로 요청에 응답하는 것이 가능하다. 하지만 사업의 규모가 확장되고, 클라이언트의 수가 늘어나게 되면 기존 서버만으로 정상적인 서비스가 불가능하다. 이처럼 증가한 트래픽에 대처할 수 있는 방법은 크게 두 가지다.
### Scale-up 그리고 Scale-out
- Scale-up은 서버 자체의 성능을 확장하는 것을 의미한다. 비유하자면 CPU가 i3인 컴퓨터를 i7로 업그레이드하는 것과 같다. 
- Scale-out은 기존 서버와 동일하거나 낮은 성능의 서버를 두 대 이상 증설하여 운영하는 것을 의미한다. CPU가 i3인 컴퓨터를 여러 대 추가 구입해 운영하는 것과 같다.
- Scale-out의 방식으로 서버를 증설하기로 결정했다면 여러 대의 서버로 트래픽을 균등하게 분산해주는 로드밸런싱이 반드시 필요하다.
- 클라이언트의 요청을 특정 서버에 분배하는 로드밸런싱 기법은 여러 가지가 있다. 

## 로드 밸런싱 알고리즘
### 라운드로빈 방식 (Round Robin Method)
- 서버에 들어온 요청을 순서대로 돌아가며 배정하는 방식이다.
- 클라이언트의 요청을 순서대로 분배하기 때문에 여러 대의 서버가 동일한 스펙을 갖고 있고, 서버와의 연결(세션)이 오래 지속되지 않는 경우에 활용하기에 적합하다.
  
### 가중 라운드로빈 방식 (Weighted Round Robin Method)
- 각각의 서버마다 가중치를 매기고 가중치가 높은 서버에 클라이언트 요청을 우선적으로 배분한다.
- 주로 서버의 트래픽 처리 능력이 상이한 경우 사용되는 부하 분산 방식이다.
- 예를 들어 A라는 서버가 5라는 가중치를 갖고 B라는 서버가 2라는 가중치를 갖는다면, 로드밸런서는 라운드로빈 방식으로 A 서버에 5개 B 서버에 2개의 요청을 전달한다.

### IP 해시 방식 (IP Hash Method)
- 클라이언트의 IP 주소를 특정 서버로 매핑하여 요청을 처리하는 방식이다.
- 사용자의 IP를 해싱해, 로드를 분배하기 때문에 사용자가 항상 동일한 서버로 연결되는 것을 보장한다.
  - 해싱(Hashing): 임의의 길이를 가진 데이터를 고정된 길이의 데이터로 매핑하는 것, 또는 그러한 함수

### 최소 연결 방식 (Least Connection Method)
- 요청이 들어온 시점에 가장 적은 연결상태를 보이는 서버에 우선적으로 트래픽을 배분한다. 자주 세션이 길어지거나, 서버에 분배된 트래픽들이 일정하지 않은 경우에 적합한 방식이다.

### 최소 리스폰 타임 (Least Response Time Method)
- 서버의 현재 연결 상태와 응답시간을 모두 고려하여 트래픽을 배분한다. 가장 적은 연결 상태와 가장 짧은 응답시간을 보이는 서버에 우선적으로 로드를 배분하는 방식이다.

<hr>

출처
- [로드밸런서(Load Balancer)의 개념과 특징](https://m.post.naver.com/viewer/postView.naver?volumeNo=27046347&memberNo=2521903)

<hr>

# 라우터 내의 패킷 포워딩 (Packet Forwarding in Router)
## 포워딩 (Forwarding)이란
- 라우팅 테이블(경로표에 적힌 목적지 주소에 대응된 출력 포트로 패킷을 전송하는 작업 

## 포워딩 과정 
- IP 담당 부분이 패킷을 보낼 때와 같다. 즉 패킷의 맨 앞부분에 MAC 헤더를 부가하고, 여기에 값을 설정하여 패킷을 완성시킨 후 전기 신호로 변환해 보낸다.
- 먼저 MAC 헤더의 맨 앞에 있는 MAC 주소 필드에 값을 설정하기 위해 경로표의 '게이트웨이' 항목에서 패킷을 건네줄 상대를 판단한다. '게이트웨이'항목에 IP 주소가 쓰여있으면 이 IP가 건네줄 상대이고, 이곳이 비어있으면 IP 헤더의 수신처 IP 주소가 건네줄 상대가 된다.
- 이를 통해 상대의 IP 주소가 결정되면 ARP로 IP 주소에서 MAC 주소를 조사하고, 결과를 수신처 MAC 두소로 설정한다. 라우터에도 ARP 캐시가 있으므로 먼저 ARP 캐시를 찾아보고, 해당하는 것이 없으면 ARP로 조회를 보내 MAC 주소를 조회한다.
- 그 다음은 송신처 MAC 주소 필드인데, 이것은 출력측의 포트에 할당된 MAC주소를 설정한다. 그리고 타입필드에 0800(16)진수을 설정한다.
- 이렇게 해서 송신 패킷이 만들어졌으므로 이것을 전기 신호로 변환하여 포트에서 송신한다.

<hr>

# 서브넷 마스크(Subnet Mask)
## 서브넷 마스크란
- 32비트의 숫자. 
- '0'의 비트는 호스트 부분을 나타내고 '1'의 비트는 네트워크 부분을 나타낸다.
- 기본적으로 자체 32비트 숫자를 이용하여 IP 주소를 마스킹하기 때문에 '마스크'라는 단어가 이용된다. 

## 서브넷이란
- 네트워크가 작은 조각으로 쪼개져 있는 경우 이러한 조각을 서브넷이라고 부른다. 즉 서브넷은 작은 네트워크라고 할 수 있다.
- 이 때 네트워크 성능 개선을 위해 네트워크 관리자가 효율적으로 자원을 분배하는 것을 서브네팅(Subnetting)이라고 한다.

## 서브넷 마스크 설명 
- IP 주소는 네트워크 주소와 호스트 주소 두 부분으로 구성되어 있다.
  - 네트워크 주소는 호스트들을 모은 네트워크를 지칭하는 주소를 뜻한다. 네트워크 주소가 동일한 네트워크를 로컬 네트워크라고 한다.
  - 호스트 주소는 하나의 네트워크 내에 존재하는 호스트를 구분하기 위한 주소다.
- 예를 들어 `192.168.123.132` 주소에 서브넷 마스크가 `255.255.255.0` 이면
  - `192.168.123.`은 네트워크를 나타내며 `132`는 네트워크에 연결된 기기를 나타낸다. 
  - 여기서 `2192.168.123.0`이 서브넷이며 `192.168.123.132`는 대상 주소(서브넷 내 내 기기)이다.


<hr>

출처
- [서브넷 마스크란 무엇인가요?](https://nordvpn.com/ko/blog/what-is-subnet-mask/)

<hr>

# 게이트웨이 (Gateway)
## 게이트웨이란
- 서로 다른 전송 프로토콜을 가진 두 네트워크를 연결하는 통신에 사용되는 네트워크 노드다.
- 모든 데이터가 라우팅되기 전에 게이트웨이를 통과하거나 게이트웨이와 통신해야 한다. 즉 네트워크의 입구 및 출구 지점 역할을 한다.
- 대부분의 IP 기반 네트워크에서 적어도 하나의 게이트웨이를 거치지 않는 유일한 트래픽은 동일한 LAN(Local Area Network) 세그먼트의 노드간에 흐르는 트래픽이다.
- 게이트웨이는 네트워크의 가장자리(edge)에 구현되며 해당 네트워크의 내부/외부로 전송되는 모든 데이터를 관리한다.
- 한 네트워크가 다른 네트워크와 통신하기를 원할 때, 데이터 패킷은 게이트웨이로 전달된 다음 가장 효율적인 경로를 통해 목적지로 라우팅된다. 
- 게이트웨이는 기본적으로 프로토콜 변환기로, 두 프로토콜의 호환성을 돕고, OSI 모델의 어떤 계층에서도 모두 작동한다.

## 게이트웨이 주소
- LAN(Local Area Network) 구역에서는 IP 주소와 서브넷마스크(subnet mask)만 있으면 주변 컴퓨터와 통신이 가능하다. 다른 네트워크 구역으로 나갈 필요가 없기 때문이다. 
- 하지만 인터넷 등의 이기종 네트워크로 나가기 위해서는 게이트웨이가 있어야 하고, IP 주소, 서브넷마스크와 함께 게이트웨이 주소까지 정확하게 설정해야 한다.
- 일반적으로 게이트웨이의 IP 주소는 해당 네트워크 내 컴퓨터에 할당된 IP 주소 중 끝자리만 다른 형태다. 대개 1을 지정한다.
  - Ex. 컴퓨터 IP 주소가 `123.123.123.123`라면 게이트웨이 주소는 `123.123.123.1`이 된다.

## 게이트웨이와 라우터의 차이점
- 게이트웨이와 라우터는 둘 이상의 개별 네트워크 간의 트래픽을 조절하는 데 사용될 수 있다는 점에서 유사하다.
- 그러나 라우터는 두 개의 유사한 유형의 네트워크를 결합하는 데 사용되고 게이트웨이는 두 개의 상이한 네트워크를 결합하는 데 사용된다.
- 이러한 논리로 라우터는 게이트웨이로 간주될 수 있으나 게이트웨이가 항상 라우터로 간주되는 것은 아니다.
  

<hr>

출처
- [gateway](https://www.techtarget.com/iotagenda/definition/gateway)
- [네트워크 항해의 첫 관문 - 게이트웨이(Gateway)](https://it.donga.com/6744/)

<hr>

# 다중화(Multiplexing, MUX)과 역다중화(Demultiplexing, DeMUX)
- 컴퓨터 네트워크에서 대부분 모든 프로토콜 아키텍처는 다중화와 역다중화를 사용한다. 
- 전송 계층 프로토콜에서의 다중화 및 역다중화, 즉 TCP 및 UDP는 출발지 포트 번호 필드와 도착지 포트 번호 필드를 헤더에 추가해 수행된다.
## 다중화(MUX, Multiplexing)란
- 다중화의 주요 목적은 `n`개의 입력 라인 중 하나를 출력 라인으로 전송하는 것이다. 
- 즉 송신자의 여러 개의 서로 다른 애플리케이션의 프로세스로부터 데이터를 수집하고, 그 데이터를 헤더로 감싸서, 수신자에게 전체를 전송하는 것을 다중화라고 한다.
  - 헤더: 송신자 IP 주소, 수신자 IP 주소, 송신자 Port 번호, 수신자 Port 번호
- 애플리케이션 계층에서 패킷이 소켓에 의해 전송 계층으로 전달될 때, 전송 계층에서는 여러 개의 소켓의 패킷을 수집하여 하나의 세그먼트에 캡슐화하여 네트워크 계층으로 전달된다. 
- 송신자 측의 애플리케이션에서 수신자 측의 애플리케이션으로 데이터를 보내려면, 송신자가 수신자의 IP 주소와 애플리케이션의 Port 번호를 알아야 데이터를 전송할 수 있다.

## 역다중화(Demultiplexing, DeMUX)
- 수신자 측에서 수신된 세그먼트를 올바른 애플리케이션 계층의 프로세스로 전달하는 것을 말한다. 

### Connectionless DeMUX
- Ex. UDP
- 송신자 IP 주소나 송신자 Port 번호가 달라도 수신자 IP 주소와 수신자 Port 번호만 같으면 같이 전달된다.

### Connection-oriented DeMUX
- Ex. TCP
- 웹 서버는 많은 TCP 소켓을 동시에 지원한다. 각 연결된 클라이언트는 서로 다른 소켓을 갖는다.
- TCP 소켓은 4개의 요소로 구성된 집합에 의해 식별된다.
  - 송신자 IP 주소
  - 수신자 IP 주소
  - 송신자 Port 번호
  - 수신자 Port 번호로

<hr>

출처
- [Multiplexing and Demultiplexing in Transport Layer](https://www.geeksforgeeks.org/multiplexing-and-demultiplexing-in-transport-layer/)
- [Transport Layer 서비스 개요, 다중화와 역다중화, UDP](https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=kostry&logNo=220903948846)

<hr>

# 크로스 사이트 스크립팅(Cross Site Scripting, XSS)
## XSS란
- XSS 공격은 웹 응용프로그램에 존재하는 취약점을 기반으로 웹 서버와 클라이언트 간 통신 방식인 HTTP 프로토콜 동작과정 중에 발생한다. XSS 공격은 웹사이트 관리자가 아닌 이가 웹페이지에 악성 스크립트를 삽입할 수 있는 취약점이다. 주로 여러 사용자가 보는 게시판에 악성 스크립트가 담긴 글을 올리는 형태로 이루어진다.
- 이 취약점은 웹 응용프로그램이 사용자로부터 입력받은 값을 제대로 검사하지 않고 사용할 경우 나타나며, 사용자의 정보인 ‘쿠키’, ‘세션’ 등을 탈취하거나, 자동으로 비정상적인 기능을 수행하게 할 수 있다. 주로 다른 웹사이트와 정보를 교환하는 식으로 작동하므로 사이트 간 스크립팅이라고도 한다.

## 공격
### 저장 XSS 공격
저장 XSS 공격은 웹 어플리케이션 취약점이 있는 웹 서버에 악성 스크립트를 영구적으로 저장해 놓는 방법이다. 이때 웹 사이트의 게시판, 사용자 프로필 및 코멘트 필드에 악성 스크립트를 삽입해 놓으면, 사용자가 사이트를 방문하여 저장되어 있는 페이지에 정보를 요청할 때, 서버는 악성 스크립트를 사용자에게 전달하여 사용자 브라우저에서 스크립트가 실행되면서 공격한다.
### 반사 XSS 공격
반사 XSS 공격은 웹 어플리케이션의 지정된 변수를 이용할 때 발생하는 취약점을 이용하는 것으로, 검색 결과, 에러 메시지 등 서버가 외부에서 입력받은 값을 받아 브라우저에게 응답할 때 전송하는 과정에서 입력되는 변수의 위험한 문자를 사용자에게 그대로 돌려주면서 발생한다.
### DOM 기반 XSS 공격
DOM 기반 XSS 공격은 피해자의 브라우저가 HTML 페이지를 구분 분석할 때마다 공격 스크립트가 DOM 생성의 일부로 실행되면서 공격한다. 페이지 자체는 변하지 않으나, 페이지에 포함되어 있는 브라우저측 코드가 DOM 환경에서 악성코드로 실행된다. 

<hr>

출처
- [innerHTML의 위험성, XSS에 대해 알아보자](https://tecoble.techcourse.co.kr/post/2021-04-26-cross-site-scripting/)
- [XSS 대응방안](http://blog.plura.io/?p=7614)

<hr>