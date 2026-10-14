# Security Lists

Security List는 Subnet과 연결되는 **ingress·egress 허용 규칙 모음**이다. 연결된 Subnet의 모든 VNIC가 같은 목록들의 영향을 받는다. 설정 단위는 Subnet이지만 규칙의 enforcement는 VNIC level에서 이루어진다.

## Subnet 공통 정책에 쓰이는 이유

특정 Subnet의 모든 VNIC에 필요한 공통 traffic 조건을 표현할 때 유용하다. VNIC가 추가돼도 그 Subnet에 연결된 Security Lists의 규칙이 적용된다. 반면 같은 Subnet 안에서도 애플리케이션별 보안 요구가 다르면 선택한 VNIC를 묶는 NSG가 더 유연하다. Oracle은 애플리케이션 보안을 Subnet architecture와 분리하는 NSG 사용을 권장한다.

## Default와 Custom의 차이

VCN은 기본 Security List와 초기 stateful rules를 제공한다. Subnet에서 custom 목록을 선택하지 않으면 default가 적용된다. Default에는 SSH·ICMP 관련 규칙과 broad egress가 포함되므로 이름만 보고 가장 좁은 정책이라고 생각하면 안 된다. 현재 연결된 목록과 규칙이 실제 적용 범위다.

Custom 목록은 Subnet의 목적에 맞는 공통 허용을 표현할 수 있다. Subnet은 최대 다섯 Security Lists와 연결되며 여러 Subnet이 같은 목록을 사용할 수도 있다. 한 목록 변경이 여러 Subnet의 VNIC에 영향을 줄 수 있다는 점이 중요하다.

## 허용의 합집합

해당 VNIC의 Subnet에 연결된 모든 Security Lists와 VNIC가 가입한 모든 NSG의 규칙을 함께 고려한다. 어느 유효 규칙이 traffic을 허용하면 그 허용이 적용되며, 좁은 목록 하나가 다른 목록의 broad allow를 상쇄하지 않는다. 기존 tracked response와 Stateful/Stateless overlap은 별도의 동작이다. Guest firewall이나 ZPR 같은 다른 제어는 이 합집합만으로 대체되지 않는다.

## 선택 예와 관리의 한계

Subnet 전체의 동일한 관리 source 범위를 표현하는 것과 Web VNIC만 app port를 받도록 하는 것은 범위가 다르다. 전자는 Security List의 후보이고 후자는 NSG의 후보다. 둘 모두 route를 생성하거나 application listener를 실행하는 기능은 아니다.

같은 Security List를 여러 Subnet이 공유하면 한 번의 규칙 변경이 여러 Subnet의 VNIC에 영향을 준다. 따라서 변경 범위는 한 VM이 아니라 연결된 Subnet 전체라는 관점에서 이해해야 한다.

## 관련 개념

- [Subnets](02_Subnets.md)
- [Network Security Groups](06_Network_Security_Groups.md)
- [Ingress / Egress](08_Ingress_Egress.md)
- [Stateful / Stateless Rules](09_Stateful_Stateless_Rules.md)

## Hands-on으로 이어가기

개념 설명과 자원 생성 절차는 구분된다. 기존 작업·검증·정리 내용은 [보존된 Hands-on](../../Hands-on/Networking/Original-Provisional/03_Virtual_Cloud_Network_VCN/07_Security_Lists.md)에서 읽을 수 있다.

## 공식문서

- [Security Lists](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/securitylists.htm)
- [Security Rules](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/securityrules.htm)
- [Network Security Groups](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/networksecuritygroups.htm)
