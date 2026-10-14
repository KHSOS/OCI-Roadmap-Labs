# Virtual Cloud Network (VCN)

Virtual Cloud Network (VCN)은 OCI의 한 Region에 구성하는 **논리적인 private network**다. 주소 공간·Subnet·경로·보안 규칙·Gateway를 조합하여 클라우드 자원이 서로, 인터넷 또는 외부 네트워크와 어떻게 통신할지 정한다. VCN은 VM도 아니고 Compartment도 아니다.

## VCN 안에서 각 요소가 맡는 일

|요소|역할|
|---|---|
|CIDR Blocks|VCN이 사용하는 주소 공간|
|Subnets|주소 공간의 분할과 공통 network configuration|
|VNICs / Private IPs|인스턴스가 Subnet에 연결되고 주소를 갖는 접점|
|Route Tables|목적지에 대한 다음 경로 선택|
|Security Lists / NSGs|VNIC의 허용 traffic 정의|
|Gateways|인터넷·Oracle Services Network·다른 네트워크로 향하는 연결 경로|
|DHCP Options|guest에 제공되는 DNS 등 네트워크 설정 정보|

VCN 생성 시 default Route Table, Security List, DHCP Options가 함께 제공된다. 기본 Route Table에는 명시적인 외부 route가 없지만 VCN local routing은 암묵적으로 제공된다. 기본 구성 요소가 있다는 사실만으로 VM·Subnet·Gateway까지 준비되거나 인터넷이 연결된 것은 아니다.

## 설계 예시

공개 frontend와 private backend를 가진 웹 서비스는 같은 VCN에서 Public subnet과 Private subnet을 나누고 서로 다른 route를 사용할 수 있다. NSG로 역할별 허용 규칙을 분리하면 Subnet 배치와 애플리케이션 보안 요구를 독립적으로 다루기 쉽다. VCN이 private network라는 말은 내부의 모든 자원에 인터넷 연결이 금지된다는 뜻이 아니다.

## 선택 기준과 관계

VCN은 하나의 Region에 속하지만 Regional subnet은 여러 AD에 있는 자원을 수용할 수 있다. 인스턴스의 AD 배치는 별도 선택이며 Regional subnet 자체가 애플리케이션 복제를 해 주지는 않는다. Compartment는 IAM 관리 범위이고 VCN은 네트워크 범위이므로 같은 개념으로 그리지 않는다.

## 한계

Peering이나 on-premises 연결에서는 주소 공간이 겹치지 않아야 한다. Route는 통신 허용을 대신하지 않고 security rule도 존재하지 않는 경로를 만들지 않는다. 기본 Route Table·Security List·DHCP Options는 VCN이 유지되는 동안 개별 삭제할 수 없으며 내용을 변경하거나 custom 구성 요소를 사용할 수 있다.

## 관련 개념

- [CIDR Blocks](01_CIDR_Blocks.md)
- [Subnets](02_Subnets.md)
- [Route Tables](05_Route_Tables.md)
- [Network Security Groups](06_Network_Security_Groups.md)
- [Internet Gateway](10_Internet_Gateway.md)
- [Service Gateway](12_Service_Gateway.md)
- [VNICs](13_VNICs.md)

## Hands-on으로 이어가기

개념 설명과 자원 생성 절차는 구분된다. 기존 작업·검증·정리 내용은 [보존된 Hands-on](../../Hands-on/Networking/Original-Provisional/03_Virtual_Cloud_Network_VCN/00_Virtual_Cloud_Network_VCN.md)에서 읽을 수 있다.

## 공식문서

- [Networking Overview](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/overview.htm)
- [VCN Route Tables](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/managingroutetables.htm)
