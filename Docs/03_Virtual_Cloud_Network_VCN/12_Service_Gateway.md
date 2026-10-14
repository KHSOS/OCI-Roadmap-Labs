# Service Gateway

Service Gateway (SGW)는 VCN에서 **같은 Region의 지원 Oracle Services Network (OSN) 서비스**에 인터넷을 거치지 않고 접근하는 경로다. Public IP 없는 자원도 Oracle network fabric을 통해 Object Storage 같은 서비스를 사용할 수 있다. 특정 서비스로 가는 private 경로이지 모든 인터넷 목적지로 가는 Gateway는 아니다.

## 활용 예

Private DB system이 Object Storage로 backup을 보내거나 private application이 regional Oracle service를 호출하는 경우 SGW를 고려할 수 있다. 지원 서비스 접근 자체에는 IGW나 NAT Gateway가 필요하지 않다. 그러나 일반 외부 repository나 SGW 미지원 endpoint에 접근하는 workload는 NAT 등 다른 경로가 필요할 수 있다.

## Service CIDR Label의 의미

Oracle은 지원 서비스의 regional public IP 범위를 문자열 label로 표현한다. Object Storage-only label과 `All <Region> Services in Oracle Services Network` label이 대표적이다. 이 label을 Gateway service 선택과 route destination에 사용하면 실제 endpoint IP 범위가 바뀌어도 주소들을 직접 유지할 필요가 없다.

SGW가 All Services를 지원하면 route는 All Services 또는 Object Storage label을 사용할 수 있다. 반대로 Object Storage-only Gateway에 All Services route를 결합하면 다른 서비스가 blackhole될 수 있다. 두 label의 Object Storage 범위가 겹치므로 같은 Table에 각 label의 route를 별도로 동시에 두는 구성은 허용되지 않는다.

## 경로·허용·인증은 별개의 조건

Route는 service destination을 SGW로 보내고 security rule은 필요한 protocol·port를 허용한다. 예를 들어 stateful service egress TCP443은 HTTPS 요청과 tracked response의 조건을 표현한다. 이 경로가 Object Storage bucket을 익명 공개하거나 IAM OBJECT_READ를 자동 부여하지는 않는다. Application 또는 instance principal 등 요청자의 인증과 권한은 서비스가 별도로 판단한다.

OSN endpoint가 public IP로 resolve돼도 SGW route를 사용한 traffic은 인터넷을 통과하지 않는다. 따라서 주소 형태만으로 실제 네트워크 경로를 판단하지 않는다.

## 선택과 한계

SGW는 regional이고 같은 Region의 지원 서비스만 대상으로 한다. 한 VCN의 여러 Subnet이 route·security 조건에 따라 공유할 수 있다. 일반 peering만으로 다른 VCN이 이 SGW를 사용할 수 있는 것은 아니며 별도의 transit 설계가 필요하다. Block traffic은 route와 security가 있어도 Gateway를 통과하는 traffic을 막는다. Target route 참조가 남아 있으면 Gateway를 삭제할 수 없다. Gateway 경로 비용과 서비스의 저장·요청·전송 비용은 구분한다.

## 관련 개념

- [Private Subnets](03_Private_Subnets.md)
- [Route Tables](05_Route_Tables.md)
- [Network Security Groups](06_Network_Security_Groups.md)
- [NAT Gateway](11_NAT_Gateway.md)
- [Identity and Access Management (IAM)](../04_Identity_and_Access_Management_IAM/00_Identity_and_Access_Management_IAM.md)
- [Dynamic Groups](../04_Identity_and_Access_Management_IAM/03_Dynamic_Groups.md)

## 관련 튜토리얼

[ServiceGateway 튜토리얼](../../Hands-on/Networking/ServiceGateway.md)에서 자원 구성, 동작 확인과 정리 순서를 따라갈 수 있다.

## 공식문서

- [Access to Oracle Services: Service Gateway](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/servicegateway.htm)
- [Security Rules](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/securityrules.htm)
- [VCN Route Tables](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/managingroutetables.htm)
