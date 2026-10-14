# VNICs

Virtual Network Interface Card (VNIC)는 Compute instance를 **Subnet과 VCN에 연결하는 가상 network interface**다. Private IP, MAC, 선택한 NSG membership, 선택적 route association 같은 network 속성을 가진다. 물리 NIC, VNIC, Private IP object는 서로 다른 요소다.

## Primary와 Secondary

|유형|수명과 역할|
|---|---|
|Primary VNIC|Instance 생성과 함께 만들어지고 붙는다. Instance에서 개별 제거할 수 없다.|
|Secondary VNIC|추가 network 연결에 사용하며 detach 시 즉시 삭제된다. 독립적으로 남기거나 다른 Instance로 옮길 수 없다.|

Secondary VNIC는 다른 Subnet 또는 다른 VCN에도 연결할 수 있지만 모든 VNIC는 instance와 같은 AD 조건을 따른다. Regional subnet이라는 사실이 instance의 AD 제약을 없애는 것은 아니다.

## 여러 주소와 여러 Interface의 선택

하나의 VNIC에 secondary Private IP를 추가하는 것과 Secondary VNIC를 추가하는 것은 다르다. 같은 Subnet의 주소가 여러 개 필요한 요구라면 여러 Private IP가 더 단순할 수 있다. 반면 서로 다른 Subnet·VCN에 연결해야 하는 network appliance 같은 경우에는 Secondary VNIC가 후보가 된다. VNIC별 NSG membership과 route가 달라질 수 있다.

VNIC가 많아져도 instance의 총 network bandwidth가 자동으로 늘어나지는 않으며 지원 VNIC 수는 Shape에 따라 제한된다.

## OCI 객체와 Guest OS의 관계

OCI에서 attach됐다는 사실만으로 guest가 주소·route를 모두 사용할 준비가 됐다고 보장할 수 없다. MAC와 Private IP는 OCI VNIC와 guest interface를 대응하는 단서다. Oracle Linux는 Secondary VNIC OS 구성에 oci-network-config 사용을 권장하며 다른 OS는 지원 지침이 다르다. 같은 Subnet에 여러 VNIC가 있으면 asymmetric routing이 생길 수 있어 응답 경로도 중요하다.

## Forwarding과 관리 범위의 한계

기본 source/destination check는 VNIC가 source 또는 destination이 아닌 packet을 drop한다. 일반 application server와 traffic을 전달하는 router appliance의 요구는 다르며 Secondary VNIC를 붙였다는 이유만으로 check를 끌 필요는 없다. Forwarding에는 OCI 속성뿐 아니라 guest 구성도 필요하다.

VNIC와 Subnet은 같은 Compartment에 있고 VNIC attachment는 Instance의 Compartment에 있는 별도 객체다. 따라서 network 관리 범위와 Compute 관리 범위가 다를 수 있다. NSG membership·Public IP·OS interface 상태를 VM 이름 하나로 모두 동일시하지 않는다.

## 관련 개념

- [Public IP Addresses](../02_Compute/06_Public_IP_Addresses.md)
- [Virtual Cloud Network (VCN)](00_Virtual_Cloud_Network_VCN.md)
- [Subnets](02_Subnets.md)
- [Route Tables](05_Route_Tables.md)
- [Network Security Groups](06_Network_Security_Groups.md)

## Hands-on으로 이어가기

개념 설명과 자원 생성 절차는 구분된다. 기존 작업·검증·정리 내용은 [보존된 Hands-on](../../Hands-on/Networking/Original-Provisional/03_Virtual_Cloud_Network_VCN/13_VNICs.md)에서 읽을 수 있다.

## 공식문서

- [Virtual Network Interface Cards (VNICs)](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/managingVNICs.htm)
- [Networking Overview](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/overview.htm)
