# Layer 3 / Layer 4

Layer 3는 IP 주소와 IP protocol을 통해 packet의 네트워크 목적지를 다루고, Layer 4는 TCP/UDP의 port와 flow를 다룬다. NLB에서 이 구분은 어떤 packet 정보를 분배 기준으로 사용할지를 설명한다. HTTP URL이나 cookie를 읽는 Layer 7의 판단과는 다르다.

## 역할과 구성

5-Tuple Hash는 source IP/port, destination IP/port, protocol을 사용한다. 3-Tuple Hash는 두 IP와 protocol을, 2-Tuple Hash는 두 IP를 사용한다. 5-Tuple은 한 TCP/UDP session의 affinity에 적합하고 3/2-Tuple은 session 수명을 넘는 주소 중심 affinity에 활용할 수 있다. L3 IP listener에서는 5-Tuple policy를 사용할 수 없다.

## 사용 예시

같은 client가 TCP service에 두 개의 연결을 열면 source port가 달라질 수 있다. 5-Tuple에서는 별개의 hash 입력이므로 다른 backend가 선택될 수 있지만 각 flow는 그 연결 동안 같은 backend로 전달된다. Payload가 HTTP처럼 생겼더라도 TCP listener가 URL 경로에 따라 backend를 선택하는 것은 아니다.

## 선택 기준과 제약

애플리케이션이 실제 사용하는 TCP/UDP/L3 IP 형태와 필요한 affinity를 먼저 구분한다. HTTP 경로 분기는 Load Balancer의 기능과 비교하고, UDP를 TCP의 신뢰성·연결 종료 방식과 동일하게 해석하지 않는다. ICMP는 private NLB의 Source/Destination Preservation 조건에서 지원된다. 한 backend에서 같은 서버 응답을 받은 사례는 다중 backend hash 분배의 증거가 아니다.

## 관련 개념

- [Network Load Balancer](00_Network_Load_Balancer.md)
- [IP Address Preservation](02_IP_Address_Preservation.md)
- [Long-running Connections](04_Long_running_Connections.md)

## 더 읽기

- [Oracle — Introduction to Network Load Balancer](https://docs.oracle.com/en-us/iaas/Content/NetworkLoadBalancer/introduction.htm)
- [Oracle — Overview of Flexible Network Load Balancer](https://docs.oracle.com/en-us/iaas/Content/NetworkLoadBalancer/overview.htm)

설정과 실습 절차는 [보존된 Layer 3 / Layer 4 Hands-on 문서](../../Hands-on/Networking/Original-Provisional/09_Network_Load_Balancer/01_Layer_3_Layer_4.md)에서 읽을 수 있다.
