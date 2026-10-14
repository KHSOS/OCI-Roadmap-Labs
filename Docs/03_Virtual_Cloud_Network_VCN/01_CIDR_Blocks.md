# CIDR Blocks

CIDR (Classless Inter-Domain Routing)은 IP 주소 범위를 **주소/접두사 길이**로 표현하는 표기다. VCN·Subnet의 주소 공간뿐 아니라 route 목적지와 security rule의 source/destination에도 쓰인다. 같은 CIDR 문자열이어도 어느 필드에 쓰였는지에 따라 역할이 다르다.

## 표기를 읽는 방법

IPv4의 /24는 앞 24 bits가 network 부분이라는 뜻이다. 접두사 길이가 길수록 주소 범위는 작아진다. /32는 IPv4 주소 하나를 나타내므로 특정 관리 PC나 서버만 허용하는 source 범위로 사용할 수 있다. 0.0.0.0/0은 전체 IPv4 목적지 또는 source 범위이지, 실제 인스턴스에 할당할 주소가 아니다.

## VCN과 Subnet의 주소 계획

교육용 주소 계획에서 VCN을 10.77.0.0/16으로, 서로 다른 Subnet을 10.77.10.0/24와 10.77.20.0/24로 표현할 수 있다. 두 Subnet은 VCN 범위 안에 있고 서로 겹치지 않는다. 이 주소들은 설계 예시이며 특정 tenancy의 실제 배포 결과가 아니다.

VCN의 여러 CIDR은 서로 겹치지 않아야 한다. 같은 VCN의 Subnet도 서로 겹치지 않아야 하며, 연결할 on-premises network나 peered VCN의 범위와도 중복을 피해야 한다. 오늘은 독립된 네트워크라도 나중의 연결 계획이 주소 선택에 영향을 준다.

## 어떤 크기를 선택하는가

작은 범위는 주소 낭비를 줄이지만 VM·Load Balancer·서비스별 VNIC가 늘어날 여유를 줄인다. 큰 범위는 성장 공간을 주지만 다른 network와 연결할 때 중복을 피하기 어려울 수 있다. 주소 공간 크기와 security 허용 범위를 같게 잡을 필요는 없다. 예를 들어 /24 Subnet 안의 특정 서버 한 개만 허용할 때는 /32가 더 좁은 조건이다.

## 할당과 연결성의 한계

OCI는 각 IPv4 Subnet의 첫 두 주소와 마지막 주소를 예약한다. 따라서 일반적인 IPv4 host 계산을 그대로 OCI의 할당 가능 주소 수로 읽으면 안 된다. VCN 주소 범위에 publicly routable range를 사용하더라도 VNIC의 private address가 자동으로 인터넷 Public IP가 되지는 않는다. 공개 여부는 Public IP·Subnet access·Gateway·route·security의 조합이다. IPv6는 별도의 prefix·주소·접속 규칙을 가진다.

## 관련 개념

- [Virtual Cloud Network (VCN)](00_Virtual_Cloud_Network_VCN.md)
- [Subnets](02_Subnets.md)
- [Route Tables](05_Route_Tables.md)
- [Ingress / Egress](08_Ingress_Egress.md)

## Hands-on으로 이어가기

개념 설명과 자원 생성 절차는 구분된다. 기존 작업·검증·정리 내용은 [보존된 Hands-on](../../Hands-on/Networking/Original-Provisional/03_Virtual_Cloud_Network_VCN/01_CIDR_Blocks.md)에서 읽을 수 있다.

## 공식문서

- [Networking Overview](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/overview.htm)
- [Creating a Subnet](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/create_subnet.htm)
