# OSI 7 Layer
---
### 목차
* [등장 배경](#등장배경)
* [작동 원리](#작동-원리)
* [계층 별 특징](#계층-별-특징)

### 등장배경
> 컴퓨터마다 독자적인 구조를 가지고 있었기 때문에 네트워크로 연결하기가 어려웠음 -> 국제적인 표준을 만듦

## 작동 원리
![OSI-Working](https://faq.hostway.co.kr/files/attach/images/784/434/001/679c0026b17573f5f0ba7edcafe8ad20.jpg)

1. 전송 시 7계층에서 1계층으로 각각의 층마다 인식할 수 있어야 하는 **Header**를 붙임(캡슐화)
2. 수신 시 1계층에서 7계층으로 Header를 떼어냄(De-캡슐화)
3. 출발지에서 데이터가 전송될 때 헤더가 추가되는데 2계층에서만 오류제어를 위해 꼬리부분에 추가됨
4. 물리계층에서 1, 0의 신호가 되어 전송매체(동축 케이블, 광섬유 등)을 통해 전송

## 계층 별 특징
![OSI-7-Layer](https://media.geeksforgeeks.org/wp-content/uploads/computer-network-osi-model-layers.png)
### 1. Application Layer
- 서비스를 제공하는 방법 구현
- 단위: Message
#### - Function
1. Network Virtual Terminal
2. FTAM(File Transfer Access and Management)
3. Mail Services
4. Directory Services

### 2. Presentation Layer
- 데이터를 통신에 적절한 형태로 변환
- 단위: Message
#### - Function
1. Translation
  <br>ex. ASCII to EBCDIC
2. Encryption/Decryption
3. Compression 

### 3. Session Layer
- Session 연결 및 유지, 인가, 보안
- 단위: Message
- 장치 : Gateway
#### - Function
1. Session establishment, maintenance, and termination
2. Synchronization : 데이터의 동기화 지점을 고려해 체크 포인트를 데이터에 추가할 수 있음 -> 오류 식별에 도움
3. Dialog Controller : 반이중 통식 혹은 전이중 방식 채택

### 4. Transport Layer
- 완전한 메시지를 End-to-End 전달
- 성공적인 데이터 전송과 오류 발생 시 재전송
- 단위 : `segment(TCP)` or `datagram(UDP)`
#### :arrow_right: At `sender`'s side
- Flow & Error Control
- 출발지와 목적지의 **port number**가 헤더에 포함되어 있음
#### :arrow_left: At `receiver`'s side
- 순서에 맞춰 segmented 데이터를 재조립
#### - Function
1. Segmentation and Reassembly : 더 작은 유닛으로 메시지를 나누거나 재조립함
2. Service Point Addressing : 정확한 프로세스로 전달하기 위해 **service point address** 혹은 **port address**를 포함함 -> 정확성을 높이기 위함
#### - Service
1. Connection-Oriented Service : 신뢰성 & 안전성
    - Connection Establishment
    - Data Transfer
    - Termination / disconnection
2. Connectionless Service : 빠른 속도(신속성)

### 5. Network Layer
- 다른 네트워크 상에 있는 사용자에게 전송할 때 주소와 경로의 선택 방법 규정
- packet routing을 제어
- **IP 주소**가 헤더에 위치함
- 단위 : `packet`
- 장치 : Router
#### - Function
1. Routing : 목적지에 어떤 경로가 적합한지 결정
2. Logical Addressing : 각 장치마다 고유한 번호를 부여하기 위함

### 6. Data Link Layer (DLL)
- 오류 없이 데이터 전송을 확실히 함
- NIC(Network Interface Card)를 통해 handling
- 단위 : `packet` or `Frame`
- 장치 : Switch, Bridge
#### 1. LLC(Logical Link Control)
- 프로토콜을 식별하여 캡슐화해서 연결하는 기능 수행
- 여러 다양한 MAC 부계층과 Network 계층 간의 **접속** 담당
<details>
<summary>패킷 전송 서비스 방식</summary>
1. 비연결형(Connectionless; LLC Type 1)<br>
    - 데이터그램 전송방식<br>
2. 연결지향형(Connection-oriented;LLC Type 2)<br>
    - 신뢰적인 전송 기능 제공<br>
3. 비연결 확인형(Acknowledged Connectionless; LLC Type 3)<br>
    - 점대점 접속, 이전 프레임의 전송완료가 확인될 때만 다음 진행
</details>

#### 2. MAC(Media Access Control)
- 어떻게 프레임을 전송할 것인지 정의
- 물리 게층의 토폴로지나 기타 특성에 맞추어주는 **제어** 담당
#### - Function
1. Framing : 송신자가 수신자에게 의미있는 비트 세트를 전송할 수 있는 방법 제공
2. Physical Addressing : 프레임 생성 후 물리적 주소(MAC address)를 더함
3. Error Control : 손상되거나 손실된 프레임을 발견하고 재전송
4. Flow Control : 속도는 양쪽에서 일정해야 함 -> 승인을 받기 전에 보낼 수 있는 데이터 양 조절
5. Access Control : 단일 통신 채널이 여러 장치에서 공유될 때, MAC은 주어진 시간에 채널을 제어할 장치를 결정하는 데에 도움을 줌

### 7. Physical Layer
- 신호를 받아 0과 1로 변환시킴
- 단위 : `bit`
- 장치 : Hub, Repeater, Modem, Cables
#### - Functions
1. Bit Synchronization : 수신자와 송신자의 동기화 제어
2. Bit rate control : 전송 속도 조절
3. Physical topologies : 서로 다른 장치/노드가 네트워크에 배열되는 방식 지정
    <br>ex. bus, start or mesh topology
4. Transmission mode : 데이터 전송 방식을 지정
    <br> ex. Simplex, half-duplex, full-duplex

<details>
<summary>출처</summary>
<div markdown="1">
:one: https://www.geeksforgeeks.org/layers-of-osi-model/
</details>