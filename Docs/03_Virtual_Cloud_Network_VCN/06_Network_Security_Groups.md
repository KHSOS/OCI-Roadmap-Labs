# Network Security Groups

Network Security Group (NSG)은 같은 VCN 안에서 **같은 보안 요구를 가진 VNIC 집합**에 ingress·egress 허용 규칙을 적용하는 virtual firewall이다. Web tier·app tier·관리 대상 같은 애플리케이션 역할을 Subnet 주소 배치와 분리해 다룰 때 유용하다.

## Rule과 Membership의 관계

NSG에는 규칙과 member VNIC라는 두 종류의 관계가 있다. 규칙만 존재하고 대상 VNIC가 member가 아니면 그 NSG의 규칙은 대상에 적용되지 않는다. Compute에서 instance를 NSG에 넣는다는 표현은 실제로 해당 VNIC를 member로 지정하는 것이다. Secondary VNIC는 별도로 자신의 membership을 가진다.

VCN에는 default NSG가 없고 새 NSG에는 default rules가 없다. 같은 Subnet이라는 이유로 NSG에 자동 가입되지 않는다. VNIC는 최대 다섯 NSG에 속할 수 있으며 NSG와 member VNIC는 같은 VCN에 있어야 한다.

## IP 대신 역할을 참조하는 예

App tier가 Web tier에서 시작한 TCP 연결만 받는 구조라면 App NSG의 ingress source를 Web NSG로 표현할 수 있다. Web 쪽에는 해당 app 목적지로 새 연결을 시작할 egress 허용도 필요하다. 이는 두 NSG의 같은 VCN membership을 사용하는 방식이며 이름이 같다는 사실만으로 다른 VCN의 NSG를 참조할 수 있는 것은 아니다.

이 참조는 private 경로의 역할 그룹화를 이해하는 데 유용하다. 같은 VCN의 VNIC라도 public IP끼리 통신하면 IGW 지점에서 NSG 정보가 없으므로 관련 source/destination에 Public IP를 사용해야 한다.

## Security List와 함께 사용할 때

NSG는 선택한 VNIC에, Security List는 연결된 Subnet의 모든 VNIC에 적용된다. 두 종류를 함께 쓰면 적용 가능한 허용 규칙은 합집합이다. 좁은 NSG가 broad Security List의 허용을 취소하는 deny 계층은 아니다. Stateful/Stateless 중복 규칙의 특수 동작과 guest firewall은 따로 고려한다.

## 선택과 한계

역할별 보안 분리를 원하면 NSG가 적합하지만 모든 OCI 서비스가 NSG를 지원하는 것은 아니므로 부모 서비스 지원을 확인해야 한다. NSG rule에는 개별 식별자가 있고 Security List의 rule-array 교체와 관리 모델이 다르다. NSG 삭제에는 member가 없어야 한다. ZPR security attribute를 사용한 endpoint는 해당 ZPR policy도 충족해야 하므로 NSG 허용만으로 전체 접근 허용을 보장하지 않는다.

## 관련 개념

- [Subnets](02_Subnets.md)
- [Security Lists](07_Security_Lists.md)
- [Ingress / Egress](08_Ingress_Egress.md)
- [Stateful / Stateless Rules](09_Stateful_Stateless_Rules.md)
- [VNICs](13_VNICs.md)

## 관련 튜토리얼

[NetworkSecurityGroups 튜토리얼](../../Hands-on/Networking/NetworkSecurityGroups.md)에서 자원 구성, 동작 확인과 정리 순서를 따라갈 수 있다.

## 공식문서

- [Network Security Groups](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/networksecuritygroups.htm)
- [Security Rules](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/securityrules.htm)
