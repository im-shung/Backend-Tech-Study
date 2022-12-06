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