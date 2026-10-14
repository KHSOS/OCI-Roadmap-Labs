# Ingress / Egress

Ingress와 Egress는 **특정 VNIC를 기준으로 packet이 들어오는지 나가는지**를 구분한다. 요청과 응답의 이름이 아니라 관찰 지점에 대한 방향이다. 인터넷 통신뿐 아니라 같은 VCN 안의 두 VM 사이에도 적용된다.

## 요청과 응답을 분리해서 읽기

Client A가 Server B의 TCP8080에 연결하는 교육용 예에서는 요청이 A에서 Egress, B에서 Ingress다. A의 source port는 보통 client ephemeral port이고 B의 destination port는8080이다. 응답은 B에서 Egress, A에서 Ingress이며 source port가8080, destination port가 client port로 뒤집힌다. 숫자8080은 예시 application port이지 OCI가 요구하는 고정 포트가 아니다.

|관찰 대상|새 요청의 조건|
|---|---|
|Client A|목적지 B와 destination port8080을 허용하는 Egress|
|Server B|source A와 destination port8080을 허용하는 Ingress|

두 endpoint 모두 OCI VNIC라면 새 연결의 sender egress와 receiver ingress를 각각 고려한다. 한쪽의 허용이 다른 쪽의 신규 연결 허용을 자동 생성하지 않는다.

## Stateful 응답과 새 역방향 연결

Stateful rule에 맞는 요청의 response는 tracking으로 허용된다. 그러나 B가 A의 별도 listener로 새로운 연결을 시작하는 것은 A→B 연결의 response가 아니다. 그 요청에는 B의 Egress와 A의 Ingress라는 별도 시작 조건이 필요하다. 방향만 뒤집혔다고 동일한 tracked flow로 보는 것은 잘못이다.

## 규칙을 선택하는 기준

Ingress는 source, Egress는 destination을 지정한다. 범위는 CIDR을 사용할 수 있고 NSG rule에서는 같은 VCN의 NSG를 source/destination으로 참조할 수 있다. TCP·UDP의 source port와 destination port도 서로 다른 조건이다. 일반 client 요청에 server port를 source로 지정하면 의도한 요청과 맞지 않을 수 있다.

## 해석의 한계

허용 rule을 제거해도 다른 NSG·Security List의 overlapping allow나 기존 tracked connection이 남을 수 있다. Traffic 성공·실패는 route, listener, OS firewall, 인증과도 관련되므로 단순 timeout을 한 방향 규칙의 효과로 단정하지 않는다. 이 방향 모델은 packet-level 접근을 설명하며 IAM API 권한이나 HTTP 사용자 인증과는 별개의 제어다.

## 관련 개념

- [Route Tables](05_Route_Tables.md)
- [Network Security Groups](06_Network_Security_Groups.md)
- [Security Lists](07_Security_Lists.md)
- [Stateful / Stateless Rules](09_Stateful_Stateless_Rules.md)

## Hands-on으로 이어가기

개념 설명과 자원 생성 절차는 구분된다. 기존 작업·검증·정리 내용은 [보존된 Hands-on](../../Hands-on/Networking/Original-Provisional/03_Virtual_Cloud_Network_VCN/08_Ingress_Egress.md)에서 읽을 수 있다.

## 공식문서

- [Security Rules](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/securityrules.htm)
- [Network Security Groups](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/networksecuritygroups.htm)
