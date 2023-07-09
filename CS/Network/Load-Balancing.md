# Load Balancing

### 목차
- [정의](#정의)

## 정의
> 컴퓨터 네트워크 기술의 일종으로 둘 혹은 셋이상의 중앙처리장치 혹은 저장장치와 같은 컴퓨터 자원들에게 작업을 나누는 것을 의미
> 즉, 여러 서버가 분산 처리하는 것

## 종류
- OSI 7 계층에 따라 나뉨
- L4: Transport 계층, IP 주소와 포트 번호 부하 분산이 가능
- L7: Application 계층, URL 또는 HTTP 헤더에서 부하 분산이 가능


### NLB와 ALB의 차이
- 몇 계층에서 분산작업을 수행하느냐에 따라 나뉨 -> NLB & ALB

## NLB
> Network LoadBalancer

- Client IP와 서버 사이에 서버로 들어오는 트래픽은 로드밸런서를 통하고 나가는 트래픽은 Client IP와 직접 통신함
- Security Group 적용이 되지 않아서 서버에 적용된 Security Group에서 보안이 가능함
- Client -> Server에서 Access 제한 가능
- 할당한 Elastic IP를 Static IP로 사용이 가능하여 DNS Name과 IP 주소 모두 사용이 가능함

## ALB
> Application LoadBalancer

- Reverse Proxy 대로 Client IP와 서버 사이에 들어오고 나가는 트래픽이 모두 로드밸런서와 통신함
- Security Group을 통한 보안이 가능함
- Client -> Load Balancer의 Access 제한 가능
- IP 주소가 변동되기 때문에 Client에서 Access할 ELB의 DNS Name을 이용해야 함

## 알고리즘
1. Round Robin
  - 단순히 Round Robin으로 분산하는 방식
2. Least Connections
  - 연결 개수가 가장 적은 서버를 선택하는 방식
  - 트래픽으로 인해 세션이 길어지는 경우 권장하는 방식
3. Source
  - 사용자의 IP를 해싱하여 분배하는 방식
  - 사용자는 항상 같은 서버로 연결되는 것을 보장