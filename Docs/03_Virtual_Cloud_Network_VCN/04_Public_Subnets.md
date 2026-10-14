# Public Subnets

Public subnet은 VNIC에 **public IPv4 할당을 허용**하고 IPv6 endpoint와 인터넷 통신을 허용하는 Subnet access 선택이다. Internet-facing frontend처럼 직접 공개 연결이 필요한 자원의 배치 후보다. 이름에 public이 들어갔다는 사실이나 access 설정만으로 모든 자원이 인터넷에 공개되는 것은 아니다.

## 직접 인터넷 통신의 구성

Compute의 IPv4 직접 통신은 Public subnet, 실제 Public IP, enabled Internet Gateway, 올바른 route와 security rules가 함께 작동해야 한다. Guest firewall과 서비스 listener도 해당 통신을 허용해야 한다. Public subnet의 VM이라도 Public IP를 할당하지 않을 수 있다.

## 활용 예와 판단 기준

Public Load Balancer를 frontend에 두고 backend를 Private subnet에 두면 사용자-facing 연결과 내부 처리 계층을 분리할 수 있다. 관리 접속을 위한 Public VM을 두는 경우에는 해당 관리자 source만 좁게 허용하는 보안 요구가 별도로 있다. 반대로 직접 공개할 이유가 없는 데이터 계층은 Private subnet이 기본 후보다. 이 예시는 구성 선택을 설명하며 모든 웹 애플리케이션이 Public VM을 필요로 한다는 뜻은 아니다.

## Route와 보안 규칙은 다른 질문이다

Route destination의 0.0.0.0/0 → Internet Gateway는 다른 더 구체적인 경로가 없는 IPv4 목적지의 다음 hop을 표현한다. 이것은 SSH ingress source를 0.0.0.0/0으로 허용하라는 뜻이 아니다. Route는 어디로 보낼지, security rule은 어떤 protocol·port·peer를 허용할지 결정한다.

NSG와 Security List의 유효 허용 규칙은 합집합이므로 좁은 NSG가 다른 목록의 넓은 허용을 취소하지 않는다. VNIC/IP의 Per-resource Routing이 있으면 Subnet Route Table만으로 실제 경로를 설명할 수도 없다.

## 노출과 주소 수명의 한계

공개 주소를 제거하거나 IGW를 disable하면 관리·서비스 경로에 영향을 줄 수 있다. Public IP의 Ephemeral/Reserved 수명은 Subnet access와 별개의 선택이다. IPv4 공개 주소나 route만으로 IPv6 도달성까지 판단하지 않으며, HTTPS egress 허용은 inbound HTTPS 웹 서버 제공과 다른 요구다.

## 관련 개념

- [Public IP Addresses](../02_Compute/06_Public_IP_Addresses.md)
- [Subnets](02_Subnets.md)
- [Private Subnets](03_Private_Subnets.md)
- [Route Tables](05_Route_Tables.md)
- [Network Security Groups](06_Network_Security_Groups.md)
- [Internet Gateway](10_Internet_Gateway.md)

## Hands-on으로 이어가기

개념 설명과 자원 생성 절차는 구분된다. 기존 작업·검증·정리 내용은 [보존된 Hands-on](../../Hands-on/Networking/Original-Provisional/03_Virtual_Cloud_Network_VCN/04_Public_Subnets.md)에서 읽을 수 있다.

## 공식문서

- [Networking Overview](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/overview.htm)
- [Creating a Subnet](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/create_subnet.htm)
- [Public IP Addresses](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/managingpublicIPs.htm)
- [Internet Gateway](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/managingIGs.htm)
