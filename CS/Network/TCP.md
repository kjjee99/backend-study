# TCP

### 목차
* [특징](#특징)
* [Header](#tcp-header)

> Transmission Control Protocol
>
> 인터넷 상에서 데이터를 메세지의 형태로 보내기 위해 IP와 함께 사용하는 프로토콜

## 특징
- 연결형 서비스로 가상 회선 방식 제공
    - 발신지와 수신지를 연결하여 패킷을 전송하기 위한 논리적 경로를 배정하는 것
- _3-way handshaking_ 과정을 통해 **연결을 설정**하고, _4-way handshaking_을 통해 **해제**
    - 목적지와 수신지를 확실히하여 정확한 전송을 보장하기 위해서 세션을 수립하는 과정 - UDP보다 속도가 느리게 됨
- 흐름제어 및 혼잡제어
    - 흐름제어: 데이터를 송신하는 곳과 수신하는 곳의 데이터 처리 속도를 조절하여 수신자의 버퍼 오버플로우를 방지하는 것
    - 혼잡제어: 네트워크 내의 패킷 수가 넘치게 증가하지 않도록 방지하는 것
- 바이트 스트림 서비스 : 데이터의 경계를 구분하지 않음
- **높은 신뢰성 보장**
- UDP보다 속도가 느림
- 전이중(Full-Duplex), Point-to-Point

## TCP Header
![tcp-header](https://grdp.co/cdn-cgi/image/width=500,height=500,quality=50,f=auto/https://gs-post-images.grdp.co/2022/5/picture1-img1653387926778-85.png-rs-high-webp.png)
| Field | Size(bit) | Content |
|---|---|---|
|송수신자의 포트 번호|16|TCP로 연결되는 가상 회선 양단의 송수신 프로세스에 할당되는 포트 주소|
|시퀀스 번호<br>(Sequence Number)|32|송신자가 지정하는 순서번호, 전송되는 바이트 수를 기준으로 증가(1B 당 1 증가)<br> `SYN = 1` : 초기 시퀀스 번호, ACK 번호 = 1을 더한 값<br>`SYN = 0` : 현재 세션의 이 세그먼트 데이터의 최초 바이트 값의 누적 시퀀스 번호|
|응답번호<br>(Acknowledgment Number)|32|수신 프로세스가 제대로 수신한 바이트 수를 응답하기 위해 사용 => 다음에 보내줘야하는 데이터의 시작점<br>`ACK = 0` : 필드 자체가 무시됨|
|데이터 Offset|4|TCP 세그먼트의 시작 위치를 기준으로 데이터의 시작 위치를 표현(TCP 헤더의 크기)|
|예약 필드(Reserved)|6|사용을 하지 않지만 나중을 위한 예약 필드이며 0으로 채워져야 함|
|제어 비트(flag)|6|SYN, ACK, FIN 등의 제어 번호|
|윈도우 크기|16|한번에 전송할 수 있는 데이터의 양<br> 수신 윈도우의 버퍼 크기를 지정할 때 사용, 0이면 송신 프로세스의 전송 중지|
|Checksum|16|TCP 세그먼트에 포함되는 프로토콜 헤더와 데이터에 대한 오류 검출 용도|
|긴급 위치|16|긴급 데이터를 처리하기 위함, URG 플래그 비트가 지정된 경우에만 유효|

### Flag Bit
|Type|Content|
|---|---|
|ACK|응답 번호 필드가 유효한지 설정할 때 사용하며 **상대방으로부터 패킹을 받았다**는 걸 알려주는 패킷.<br>클라이언트가 보낸 최초의 SYN 패킷 이후에 전송되는 모든 패킷은 이 플래그가 설정되어야한다.|
|SYN|**연결 설정 요구.** 동기화 시퀀스 번호. 양쪽이 보낸 최초의 패킷에만 이 플래그가 설정되어있어야 한다. TCP에서 세션을 성립할 때 가장 먼저 보내는 패킷, 시퀀스 번호를 임의적으로 설정하여 세션을 연결하는 데에 사용되며 초기에 시퀀스 번호를 보내게 된다.|
|PSH|수신 애플리케이션에 버퍼링된 데이터를 _상위 계층에 즉시 전달할 때_ 사용|
|RST|연결의 리셋이나 유효하지 않은 세그먼트에 대한 응답용으로 사용|
|URG|긴급 위치를 필드가 유효한지 설정(긴급한 데이터는 다른 데이터에 비해 우선순위가 높음)|
|FIN|세션 연결을 종료시킬 때 사용되며 더 이상 전송할 데이터가 없을 때 연결 종료 의사 표시|

---

## TCP 3-way handshaking & 4-way handshaking

> 정확한 전송을 보장하기 위해 3-way handshaking으로 논리적인 접속을 성립시킴

### 역할
- 양쪽 모두 데이터를 전송할 준비가 되었다는 것을 보장하고, 실제로 데이터 전달이 시작하기 전에 한 쪽이 다른 한 쪽이 준비되었다는 것을 알 수 있도록 함
- *3-way handshaking*은 **연결을 초기화**하기 위해, *4-way handshaking*은 **세션을 종료**하기 위해 수행되는 절차
![tcp-session](https://www.researchgate.net/publication/350778963/figure/fig1/AS:1010863871885312@1618020136341/The-base-TCP-session-start-and-end-37.png)

### 3-way handshaking
1. Client(A)가 Server(B)에게 _SYN Flag_ 를 보냄 => A는 SYN을 보내고 SYN/ACK 응답을 기다리는 `SYN_SENT` 상태, B는 `Wait For Client` 상태
2. B가 SYN을 받고, A에게 요청을 수락하는 _SYN/ACK Flag_ 가 설정된 패킷 발송
3. A는 B에게 **ACK**를 보내면 연결이 이루어지고 데이터가 오가게 됨 ⇒ B는 `ESTABLISHED` 상태

### 4-way handshaking
1. Client(A)가 연결을 종료하겠다는 _FIN Flag_ 를 전송함 ⇒ A는 `FIN-WAIT` 상태
2. Server(B)는 FIN Flag를 받고 확인메시지 _ACK Flag_ 를 보내고 자신의 통신이 끝날 때까지 기다림 ⇒ B는 `CLOSE-WAIT` 상태
3. 연결을 종료할 준비가 되면, 연결 해지를 위한 준비가 되었음을 알리기 위해 A에게 _FIN Flag_ 를 전송 ⇒ B는 `LAST-ACK` 상태
4. A는 해지할 준비가 되었다는 **ACK**를 전송 ⇒ A는 `TIME-WAIT` 상태, B는 `CLOSE` 상태
5. A는 FIN을 수신하더라도 일정시간(Default = 240s)동안 세션을 남겨놓고 잉여 패킷을 기다림(TIME-WAIT) → 일정 시간이 지나면, 세션을 만료하고 연결을 종료시키며, `CLOSE` 상태로 변화
