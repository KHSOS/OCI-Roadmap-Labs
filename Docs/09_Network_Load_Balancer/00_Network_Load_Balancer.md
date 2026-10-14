# Network Load Balancer

Network Load Balancer는 하나의 IP 진입점으로 들어오는 네트워크 흐름을 여러 backend server에 분배하는 OCI 서비스다. HTTP 요청 내용이 아니라 Layer 3/Layer 4의 주소·포트·프로토콜 정보를 기준으로 연결을 배치한다. 낮은 지연과 높은 처리량이 중요한 TCP 서비스, 실시간 스트리밍, 네트워크 appliance에 적합하다.

## 역할과 구성

Listener는 수신 protocol과 port를 정의한다. Backend Set은 서버 목록, 분배 policy, Health Check Policy를 묶는 논리 단위다. Public NLB는 인터넷에서 접근 가능한 주소를, Private NLB는 VCN subnet의 private 주소를 사용한다. 어느 쪽이든 주소만으로 접근이 허용되는 것은 아니며 route와 NSG/Security List가 데이터 경로를 허용해야 한다.

## 사용 예시

내부 TCP 애플리케이션 두 대를 하나의 private 주소 뒤에 배치하면 client는 그 진입점에 연결하고 NLB는 해당 flow를 선택된 서버로 보낸다. Backend 하나가 건강하지 않을 때의 새 연결 선택과 이미 존재하는 연결의 연속성은 서로 다른 문제다. 서버 하나만 등록한 구성은 진입점을 통합하지만 애플리케이션 이중화가 되지는 않는다.

## 선택 기준과 제약

HTTP 경로 기반 처리, TLS termination, WAF 연결이 필요하면 Load Balancer와 비교한다. NLB는 TLS를 종료하는 HTTP reverse proxy가 아니다. Full NAT, Source Preservation, Transparent mode는 backend가 보는 packet과 반환 경로가 달라지므로 보안·routing 요구에 맞춰 선택한다. 공식 introduction의 구성 한도는 NLB당 Backend Set 50개, Listener 50개, Set당 backend 512개, 전체 backend 1,024개다. 기본 동시 연결 한도는 AD당 330,000이다. 공식 Overview는 AD가 세 개인 Region의 기본 한도를 약 1,000,000으로 설명한다. 이 값은 애플리케이션 처리량의 보장이 아니며, 설계할 때 tenancy의 적용 한도와 필요한 capacity를 함께 확인한다.

## 관련 개념

- [Layer 3 / Layer 4](01_Layer_3_Layer_4.md)
- [IP Address Preservation](02_IP_Address_Preservation.md)
- [Health Check Policies](03_Health_Check_Policies.md)
- [Long-running Connections](04_Long_running_Connections.md)

## 더 읽기

- [Oracle — Overview of Flexible Network Load Balancer](https://docs.oracle.com/en-us/iaas/Content/NetworkLoadBalancer/overview.htm)
- [Oracle — Introduction to Network Load Balancer](https://docs.oracle.com/en-us/iaas/Content/NetworkLoadBalancer/introduction.htm)

설정과 실습 절차는 [보존된 Network Load Balancer Hands-on 문서](../../Hands-on/Networking/Original-Provisional/09_Network_Load_Balancer/00_Network_Load_Balancer.md)에서 읽을 수 있다.
