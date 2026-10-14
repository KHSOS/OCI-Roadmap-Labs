# Route Tables

Route Table은 목적지에 따라 packet의 **다음 hop**을 선택하는 규칙 모음이다. VCN의 외부 목적지뿐 아니라 custom Intra-VCN routing에도 사용한다. Protocol·port를 허용하는 firewall과 달리, Route Table은 통신을 어디로 보낼지 결정한다.

## Rule의 두 핵심 값

일반 route는 destination CIDR과 target으로 구성된다. Target은 IGW·NAT Gateway·SGW·DRG·LPG 또는 적절히 구성된 Private IP object일 수 있다. SGW에서는 주소를 직접 나열하는 대신 Oracle의 service CIDR label을 destination으로 사용한다.

|목적|교육용 경로 예|
|---|---|
|Public 자원의 직접 인터넷 통신|기타 IPv4 목적지 0.0.0.0/0 → IGW|
|Public IP 없는 자원의 outbound 인터넷|0.0.0.0/0 → NAT Gateway|
|지원 regional Oracle 서비스|Service CIDR label → SGW|
|연결된 on-premises network|해당 network CIDR → DRG|

이 표는 다음 hop 선택 예시이며 gateway 생성만으로 연결이 완성된다는 뜻은 아니다.

## Local routing과 우선순위

VCN 안의 Subnet 사이에는 기본적으로 implicit local routing이 있다. 따라서 명시적인 route 배열이 비어 있어도 모든 내부 통신이 불가능한 것은 아니다. Custom Intra-VCN route는 일부 기본 경로를 바꿀 수 있다. 같은 Subnet 안의 목적지에는 직접 forwarding되며 Intra-subnet routing은 지원하지 않는다.

겹치는 목적지는 longest prefix match로 더 구체적인 경로를 선택한다. 같은 prefix 길이의 static route는 implicit local route보다 우선한다. 외부 목적지에 일치하는 route가 없으면 traffic이 drop된다.

## 어느 Table을 사용하는가

기본은 Subnet과 연결된 한 Route Table이다. Per-resource Routing에서는 IP address에 연결된 Table이 VNIC Table보다, VNIC Table이 Subnet Table보다 우선한다. 선택된 하나의 Table로 lookup하며 나머지를 합쳐 적용하지 않는다. Guest의 OS route와 OCI Route Table도 서로 다른 계층이다.

## 설계와 운영의 경계

경로가 맞아도 security rule·guest firewall·DNS·인증·listener 문제로 요청이 실패할 수 있다. 하나의 timeout으로 route-only 원인을 단정하기 어렵다. Network appliance의 Private IP를 target으로 쓸 때는 forwarding과 source/destination check 등 추가 설정이 필요하다. Gateway를 route target으로 참조하는 동안 그 gateway는 삭제할 수 없으며, custom Table도 association이 남아 있으면 삭제할 수 없다.

## 관련 개념

- [Virtual Cloud Network (VCN)](00_Virtual_Cloud_Network_VCN.md)
- [Subnets](02_Subnets.md)
- [Network Security Groups](06_Network_Security_Groups.md)
- [Internet Gateway](10_Internet_Gateway.md)
- [NAT Gateway](11_NAT_Gateway.md)
- [Service Gateway](12_Service_Gateway.md)
- [VNICs](13_VNICs.md)

## Hands-on으로 이어가기

개념 설명과 자원 생성 절차는 구분된다. 기존 작업·검증·정리 내용은 [보존된 Hands-on](../../Hands-on/Networking/Original-Provisional/03_Virtual_Cloud_Network_VCN/05_Route_Tables.md)에서 읽을 수 있다.

## 공식문서

- [VCN Route Tables](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/managingroutetables.htm)
- [Networking Overview](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/overview.htm)
