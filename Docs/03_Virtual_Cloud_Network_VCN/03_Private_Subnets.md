# Private Subnets

Private subnet은 VNIC에 **public IPv4 할당을 금지**하고 IPv6 endpoint와의 인터넷 통신을 금지하는 Subnet access 선택이다. 애플리케이션·데이터베이스 backend처럼 직접 인터넷에 노출할 필요가 없는 자원을 배치하는 데 사용한다. Private Cloud라는 배포 모델과는 다른 개념이다.

## Private이 막는 것과 남기는 것

Private subnet의 VM도 Private IP로 같은 VCN의 자원과 통신할 수 있다. Peering·DRG 등 적절한 연결이 있으면 다른 private network와의 통신도 가능하다. 따라서 Private은 모든 packet을 차단한다는 뜻이 아니다. 실제 허용은 경로·OCI security rules·guest firewall 및 애플리케이션 상태가 함께 결정한다.

## 인터넷과 Oracle 서비스의 접근 선택

|필요한 통신|설계 방향|
|---|---|
|Private VM이 일반 인터넷 요청 시작|NAT Gateway 등의 outbound 경로|
|같은 Region의 지원 Oracle 서비스 접근|Service Gateway와 service destination 경로|
|인터넷 사용자가 애플리케이션 접근|Public frontend가 요청을 받아 private backend와 통신하는 구조|
|관리자 접속|VCN에 들어오는 별도의 관리 경로와 인증|

IGW route만 넣는다고 public IPv4가 없는 VM이 직접 인터넷에 연결되는 것은 아니다. NAT와 SGW는 목적지가 다르며 둘을 동시에 사용할 수도 있고 어느 것도 사용하지 않을 수도 있다.

## 관리 접속의 예

Public 관리 VM을 통해 private VM에 SSH하는 구조에서는 관리 VM이 private VM으로 새 연결을 시작한다. 이에 맞는 관리 VM의 egress와 private VM의 ingress가 각각 필요하다. Stateful 응답 허용은 새 outbound 연결을 허용하는 규칙의 대체물이 아니다. 이 예시는 연결 관계 설명이며 추가 VM이나 키 준비를 이 개념 페이지의 선행조건으로 요구하지 않는다.

## 선택의 한계

공개 주소 금지는 중요한 노출 방지 장치지만 애플리케이션 권한·데이터 암호화·IAM을 대신하지 않는다. Private subnet에도 broad security allow가 있으면 private 경로에서 접근이 넓어질 수 있다. 반대로 route가 있다는 사실만으로 다운로드·인증·서비스 요청이 성공하지는 않는다. IPv4 Public IP 부재와 IPv6 인터넷 금지를 서로 다른 주소 계열의 제약으로 이해해야 한다.

## 관련 개념

- [Subnets](02_Subnets.md)
- [Public Subnets](04_Public_Subnets.md)
- [Network Security Groups](06_Network_Security_Groups.md)
- [Ingress / Egress](08_Ingress_Egress.md)
- [Service Gateway](12_Service_Gateway.md)

## Hands-on으로 이어가기

개념 설명과 자원 생성 절차는 구분된다. 기존 작업·검증·정리 내용은 [보존된 Hands-on](../../Hands-on/Networking/Original-Provisional/03_Virtual_Cloud_Network_VCN/03_Private_Subnets.md)에서 읽을 수 있다.

## 공식문서

- [Networking Overview](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/overview.htm)
- [Creating a Subnet](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/create_subnet.htm)
- [Internet Gateway](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/managingIGs.htm)
- [Access to Oracle Services: Service Gateway](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/servicegateway.htm)
