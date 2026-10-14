# Public IP Addresses

Public IP Address는 인터넷에서 주소 지정이 가능한 IPv4 주소다. OCI에서는 주소 문자열만이 아니라 OCID와 수명·할당 관계를 가진 **Public IP object**이며, Compute에 사용할 때는 VNIC 자체가 아니라 그 VNIC의 **Private IP object**에 연결된다.

## 어떤 문제를 해결하는가

인터넷의 사용자가 웹 서버나 관리 대상에 직접 연결하려면 공개 주소가 필요하다. 반면 내부 애플리케이션 서버처럼 직접 공개할 필요가 없는 대상에는 Public IP를 붙이지 않는 설계가 적합하다. 공개 주소는 경로의 한 구성 요소이지, 포트 허용이나 사용자 인증을 대신하지 않는다.

## Ephemeral과 Reserved의 선택

|유형|수명과 이동|적합한 예|
|---|---|---|
|Ephemeral|할당 대상 Private IP의 수명에 연결되며 해제하면 삭제된다. 다른 Private IP로 이동할 수 없다.|주소를 장기 보존할 필요가 없는 단기 인스턴스|
|Reserved|명시적으로 삭제할 때까지 남으며, 해제 후 같은 Region의 다른 Private IP에 재할당할 수 있다.|인스턴스 교체와 주소 수명을 분리하려는 서비스|

Ephemeral은 VNIC의 primary private IP에만, Reserved는 primary 또는 secondary private IP에 연결할 수 있다. 인스턴스 Stop만으로 Ephemeral이 해제되지는 않는다. 기존 Ephemeral을 같은 주소의 Reserved로 변환하는 기능은 없다. Reserved를 선택해도 인스턴스·애플리케이션의 고가용성이 자동 확보되지는 않는다.

## 네트워크와의 관계

Compute의 직접 인터넷 통신에는 Public subnet, 실제 Public IP, enabled Internet Gateway, 적절한 route와 security rules가 함께 필요하다. Private subnet의 VNIC에는 public IPv4를 할당할 수 없다. NAT Gateway가 가진 공개 주소와 NAT 뒤 private VM의 주소도 구분해야 한다. NAT의 공개 주소는 뒤쪽 VM에 직접 연결할 수 있는 주소가 아니다.

## 설계의 경계

Reserved는 Region 범위이지 전 세계로 이동 가능한 주소가 아니다. 주소를 보존하는 동안의 비용·서비스 제한과 네트워크 전송 비용은 별도로 고려한다. Public IP가 있다는 사실은 SSH·HTTPS가 허용됐거나 애플리케이션이 동작한다는 뜻이 아니다.

## 관련 개념

- [Virtual Cloud Network (VCN)](../03_Virtual_Cloud_Network_VCN/00_Virtual_Cloud_Network_VCN.md)
- [Public Subnets](../03_Virtual_Cloud_Network_VCN/04_Public_Subnets.md)
- [Internet Gateway](../03_Virtual_Cloud_Network_VCN/10_Internet_Gateway.md)
- [VNICs](../03_Virtual_Cloud_Network_VCN/13_VNICs.md)

## Hands-on으로 이어가기

개념 설명과 자원 생성 절차는 구분된다. 기존 작업·검증·정리 내용은 [보존된 Hands-on](../../Hands-on/Original-Provisional/02_Compute/06_Public_IP_Addresses.md)에서 읽을 수 있다.

## 공식문서

- [Public IP Addresses](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/managingpublicIPs.htm)
