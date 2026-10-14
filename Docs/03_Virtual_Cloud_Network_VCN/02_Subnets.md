# Subnets

Subnet은 VCN의 주소 공간을 나누는 **논리적인 network subdivision**이다. VNIC가 어느 주소 범위에 연결되고 어떤 Route Table·Security Lists·DHCP Options를 공통으로 사용할지 결정하는 구성 단위다. VM은 Subnet과 동일한 자원이 아니며, VM의 VNIC가 Subnet에 속한다.

## Subnet으로 나누는 이유

웹 frontend와 데이터베이스 backend처럼 공개 요구나 경로가 다른 자원을 분리할 때 유용하다. 예를 들어 인터넷과 직접 통신하는 자원과 public IPv4를 갖지 않는 자원을 나누면 주소 할당과 외부 경로의 의도를 읽기 쉽다. 애플리케이션별 보안 요구는 NSG로 추가 분리할 수 있으므로 보안 역할마다 반드시 별도 Subnet을 만들 필요는 없다.

## 서로 다른 두 선택 축

|선택 축|차이|
|---|---|
|Regional / AD-specific|Regional은 Region의 여러 AD에 있는 자원을 수용할 수 있고 AD-specific은 특정 AD로 제한된다.|
|Public / Private|Public은 public IPv4 할당을 허용하고 Private은 금지한다. 인터넷 IPv6 통신 허용도 Subnet access 설정의 영향을 받는다.|

Oracle은 더 유연한 Regional subnet을 권장한다. Regional이라고 인스턴스가 모든 AD에 동시에 존재하는 것은 아니며, Public이라고 모든 VNIC에 Public IP가 있는 것도 아니다.

## 구성 요소의 관계

Subnet은 하나의 Route Table, 하나 이상의 Security List, 한 세트의 DHCP Options와 연결된다. Custom 구성 요소를 선택하지 않으면 VCN의 default가 적용된다. Route Table은 경로, Security List는 허용 traffic, DHCP Options는 DNS 등의 설정을 담당한다. VNIC/IP에 별도 Route Table이 연결되는 Per-resource Routing에서는 해당 우선순위가 Subnet route보다 앞선다.

## 크기와 변경의 경계

IPv4 CIDR은 VCN 범위 안의 연속된 주소이며 다른 Subnet과 겹치지 않아야 한다. 필요한 VNIC와 서비스 성장 공간, 연결 예정 network를 함께 고려한다. 현재 Create Subnet 문서는 CIDR 크기 변경을 허용한다고 설명하므로 Subnet 주소 범위는 절대로 변경 불가능하다고 일반화하지 않는다. 변경 조건과 영향은 해당 작업 문서가 정하는 별도의 문제다. Subnet만으로 route·보안·guest 설정이 완성되거나 애플리케이션 가용성이 보장되지는 않는다.

## 관련 개념

- [Virtual Cloud Network (VCN)](00_Virtual_Cloud_Network_VCN.md)
- [CIDR Blocks](01_CIDR_Blocks.md)
- [Private Subnets](03_Private_Subnets.md)
- [Public Subnets](04_Public_Subnets.md)
- [Route Tables](05_Route_Tables.md)
- [Security Lists](07_Security_Lists.md)
- [VNICs](13_VNICs.md)

## Hands-on으로 이어가기

개념 설명과 자원 생성 절차는 구분된다. 기존 작업·검증·정리 내용은 [보존된 Hands-on](../../Hands-on/Networking/Original-Provisional/03_Virtual_Cloud_Network_VCN/02_Subnets.md)에서 읽을 수 있다.

## 공식문서

- [Networking Overview](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/overview.htm)
- [Creating a Subnet](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/create_subnet.htm)
- [VCN Route Tables](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/managingroutetables.htm)
