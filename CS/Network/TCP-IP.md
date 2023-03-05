# TCP/IP

### 목차
* [정의](#정의)
* [TCP/IP 4 Layer](#tcpip-layer)
* [흐름제어/혼잡제어](#흐름제어혼잡제어)

---

## 정의
> Transmission Control Protocol/Internet Protocol
- 컴퓨터 사이의 통신 표준 및 네트워크의 라우팅 및 상호연결에 대한 자세한 규칙을 지정하는 프로토콜 스위트

> :bulb: **_Protocol?_**
> 
> 서로 다른 하드웨어와 운영체제 등이 서로 통신하기 위해 모든 요소의 규칙

## TCP/IP Layer

### OSI Layar와 차이점
- 네트워크 통신을 위한 구조
![osi-tcp-ip](https://static.packt-cdn.com/products/9781789349863/graphics/6c40b664-c424-40e1-9c65-e43ebf17fbb4.png)

### 계층 별 특징
#### 1. Application Layer
- TCP/UDP 기반의 응용 프로그램 구현
- HTTP, Telnet, FTP, SSH

#### 2. Transport Layer(= Host-to-Host Layer)
- 통신 노드 간의 연결 제어, 신뢰성있는 전송 담당
- TCP/UDP

#### 3. Network Layer(= Internet Layer)
- 통신 노드 간 IP 패킷 전송 기능, 라우팅 담당
1. IP : Internet Protocol, 비신뢰성 & 비연결성 지향 데이터그램 프로토콜
2. ICMP : Internet Control Message Protocol, 상태 진단 프로토콜(ping)
3. ARP : Address Resolution Protocol, 주소 변환 프로토콜, IP 주소를 MAC 주소로 변환하는 프로토콜

#### 4. Network Interface Layer(= Link Layer)
- 노드 간의 신뢰성있는 데이터 전송을 담당하는 계층
- 물리적 주소로 MAC address tkdyd
- LAN, 패킷망 사용

---

## 흐름제어/혼잡제어

> :warning: 신뢰성있는 네트워크를 보장하는 것의 문제점
> 
> 1. **손실** : packet이 손실될 수 있는 문제
> 2. **순서 바뀜** : packet의 순서가 바뀌는 문제
> 3. **혼잡(congestion)** : 네트워크가 혼잡한 문제
> 4. **오버로드** : 수신자가 overload되는 문제

### :arrows_counterclockwise: 흐름제어
- 송신 측과 수신 측의 TCP 버퍼 크기 차이로 인해 생기는 데이터 처리 속도 차이를 해결하기 위한 기법<br>
=> 수신 측이 패킷을 지나치게 많이 받지 않도록 조절하는 것

### :o: 해결방법
#### 1. Stop and Wait
- 매번 전송한 패킷에 대한 확인 응답을 받아야만 그 다음 패킷을 전송하는 방법

![stop-and-wait-protocol](https://media.geeksforgeeks.org/wp-content/uploads/Stop-and-Wait-ARQ.png)
#### 2. Sliding Window
![sliding-window](https://scaler.com/topics/images/working-of-the-sliding-window-protocol.webp)

- 수신 측에서 설정한 윈도우 크기만큼 송신 측에서 확인 응답(ACK)없이 패킷을 전송할 수 있게 하여 데이터 흐름을 **동적으로** 조절하는 제어 기법
- 목적
    - 전송은 되었지만 `Acked`를 받지 못한 바이트의 숫자를 파악하기 위해 사용하는 프로토콜
    - (마지막에 보내진 Byte - 마지막에 확인 바이트 <= 남아있는 공간) == (현재 공중에 떠있는 패킷 수 <= sliding window)
- 윈도우 크기
    - 최초의 윈도우 크기는 `3-way handshaking`을 통해 수신 측 윈도우 크기 설정
    - 이후 수신 측의 버퍼에 남아있는 공간에 따라 변함
    - 윈도우 크기는 수신 측에서 송신 측으로 ACK를 보낼 때 TCP 헤더에 담아서 보냄
    - 윈도우 = **메모리 버퍼의 일정 영역**
- 동작 방식
    - 윈도우에 포함된 패킷을 계속 전송하고, 수신 측으로부터 ACK이 윈도우를 옆으로 옮겨 다음 패킷들을 전송함
- 재전송
    - 송신 측은 _일정 시간동안_ 수신 측으로부터 ACK을 받지 못하면 패킷을 재전송함
    - 송신 측에서 재전송을 했는데 패킷이 소실된 경우가 아니라 수신 측의 버퍼에 남는 공간이 없는 경우면 문제가 생김 => 송신 측은 ACK와 함께 남은 버퍼의 크기(Window)도 함께 보내줌

### :twisted_rightwards_arrows: 혼잡제어(Congestion Control)
- 송신 측의 데이터 전달과 네트워크 데이터 처리 속도 차이를 해결하기 위한 기법
- Congestion : 네트워크 내에 패킷의 수가 과도하게 증가하는 현상

### :o: 해결방법
#### 1. AIMD
- Additive Increase/Multicative Decrease; 합 증가/곱 감소 방식
- 처음에 패킷을 하나씩 보내고 문제없이 도착하면 윈도우의 크기를 1씩 증가시켜가며 전송
- 전송에 실패하면 윈도우 크기를 **1/2으로 줄임**
- 윈도우 크기를 너무 조금씩 늘리기 때문에 네트워크의 모든 대역을 활용하여 제대로 된 속도로 통신하기까지 시간이 오래 걸림
#### 2. Slow Start
![slow-start](https://upload.wikimedia.org/wikipedia/commons/2/24/TCP_Slow-Start_and_Congestion_Avoidance.svg)
- 윈도우 크기를 **지수적으로 증가**시켜 혼잡이 감지되면 윈도우 크기를 1로 줄이는 방식
- ACK가 도착할 때마다 윈도우 크기를 증가시키기 때문에 처음에는 윈도우 크기가 조금 느리게 증가할지라도 시간이 가면 갈수록 윈도우 크기가 점점 빠르게 증가한다는 장점이 있음
#### 3. Fast Retransmit
- 수신 측에서 세그먼트로 분할된 내용들이 순서대로 도착하지 않는 경우 발생<br>
    -> 수신 측에서는 *잘 도착한 마지막 패킷의 다음 순번*을 ACK 패킷에 실어서 보냄
    => 이런 중복 ACK를 3개 받으면 재전송이 이루어짐
- 송신 측은 자신이 설정한 타임 아웃 시간이 지나지 않았어도 바로 해당 패킷을 재전송할 수 있기 때문에 보다 빠른 재전송률을 유지할 수 있음
#### 4. Fast Recovery
- 혼잡한 상태가 되면 윈도우 크기를 1로 줄이지 않고 반으로 줄이고 선형증가시키는 방법
- 한 번 겪고 나서부터는 **AIMD 방식**으로 동작

---
<details>
<summary>출처</summary>
:one: https://www.ibm.com/docs/ko/aix/7.1?topic=management-transmission-control-protocolinternet-protocol
</details>