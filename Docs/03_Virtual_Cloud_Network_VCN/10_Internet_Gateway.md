# Internet Gateway

Internet Gateway (IGW)는 VCN과 인터넷 사이의 **직접 연결 경로**를 제공하는 optional virtual router다. VCN에서 시작하는 Egress와 인터넷에서 시작하는 Ingress를 모두 지원한다. Public IP가 없는 자원의 outbound 인터넷 연결을 위한 NAT Gateway와 역할이 다르다.

## 언제 필요한가

인터넷 사용자가 public frontend에 연결하거나 Public IP를 가진 VM이 외부 endpoint에 직접 요청해야 하는 경우 IGW가 경로의 일부가 된다. 하나의 VCN에는 IGW를 하나만 둘 수 있고 여러 Public subnet이 이를 공유할 수 있다. Subnet마다 실제 사용할지는 route와 security 설정에 달려 있다.

## 다른 구성과의 관계

Compute의 IPv4 직접 통신에는 Public subnet과 실제 Public IP가 필요하다. IGW는 enabled 상태이고 해당 traffic이 IGW target으로 향하는 route가 있어야 하며 security rules·OS firewall·서비스 상태도 맞아야 한다. IGW 생성만으로 모든 Subnet이 공개되거나 모든 port가 열리는 것은 아니다.

0.0.0.0/0 → IGW는 더 구체적인 경로가 없는 IPv4 목적지의 다음 hop 선택 예시다. 같은 문자열을 ingress source에 쓰는 것은 모든 IPv4 source를 허용하는 별개의 의미다. 인터넷 경로와 관리 source의 범위를 혼동하지 않는다.

## IGW·NAT·SGW의 선택

|요구|주요 Gateway 후보|
|---|---|
|Public 자원의 직접 inbound/outbound 인터넷|IGW|
|Public IP 없는 자원의 outbound 일반 인터넷|NAT Gateway|
|같은 Region의 지원 Oracle 서비스로 private 접근|Service Gateway|

Private backend를 인터넷에 직접 노출하지 않으려는 설계에서는 Public frontend와 private backend의 역할을 나누는 것이 더 적합할 수 있다. IGW를 만들었다는 이유로 모든 VM에 Public IP를 붙일 필요는 없다.

## 범위와 수명의 한계

IGW는 자신의 VCN 자원만 사용할 수 있으며 peered VCN이나 연결된 on-premises host가 이를 그대로 인터넷 출구로 사용할 수 없다. Security Zone의 VCN에는 IGW를 추가·이동할 수 없다. Disable은 삭제와 다르지만 해당 경로를 쓰는 여러 자원에 영향을 줄 수 있다. Route Table이 target으로 참조하는 동안 IGW를 삭제할 수 없다. 고급 Gateway ingress routing은 별도의 route association이며 일반 public 연결에 항상 필요한 설정은 아니다.

## 관련 개념

- [Public IP Addresses](../02_Compute/06_Public_IP_Addresses.md)
- [Public Subnets](04_Public_Subnets.md)
- [Route Tables](05_Route_Tables.md)
- [NAT Gateway](11_NAT_Gateway.md)
- [Service Gateway](12_Service_Gateway.md)

## Hands-on으로 이어가기

개념 설명과 자원 생성 절차는 구분된다. 기존 작업·검증·정리 내용은 [보존된 Hands-on](../../Hands-on/Networking/Original-Provisional/03_Virtual_Cloud_Network_VCN/10_Internet_Gateway.md)에서 읽을 수 있다.

## 공식문서

- [Internet Gateway](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/managingIGs.htm)
- [Networking Overview](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/overview.htm)
- [VCN Route Tables](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/managingroutetables.htm)
