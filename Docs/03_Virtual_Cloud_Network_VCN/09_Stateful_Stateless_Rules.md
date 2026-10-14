# Stateful / Stateless Rules

Stateful / Stateless는 security rule에 맞는 traffic에 **connection tracking을 사용할지** 정하는 선택이다. NSG와 Security List 모두 이 구분을 사용한다. 애플리케이션이 데이터를 저장하는 stateful workload라는 표현과는 다른 뜻이다.

## Stateful의 역할

Stateful ingress에 맞는 요청을 받으면 그 response는 egress 규칙과 관계없이 tracking으로 허용된다. Stateful egress에 맞는 요청을 보내면 들어오는 response도 자동 허용된다. 일반적인 요청·응답 구조를 적은 규칙으로 표현하기 쉬우며 기본 rule 선택은 Stateful이다. 자동 허용은 response에 대한 것이지 상대방의 새로운 연결 시작까지 허용하는 것은 아니다.

## Stateless의 역할

Stateless rule에 맞는 traffic은 tracking하지 않으므로 response를 별도 반대 방향 규칙으로 허용해야 한다. 많은 traffic을 처리하는 internet-facing HTTP/HTTPS에서는 connection tracking table의 부담을 피하려는 선택이 될 수 있다. 대신 요청·응답의 주소와 포트 관계를 명시적으로 설계해야 한다.

## 두 OCI VM의 TCP 예

Client A가 B의8080으로 요청하는 Stateless 구성에서는 A egress와 B ingress가 forward를, B egress와 A ingress가 response를 허용한다. Forward destination은8080이고 response source는8080이다. Response destination은 A가 사용한 client ephemeral port 범위다. 같은8080을 양방향 destination에 반복하는 것은 이 응답 모델과 다르다.

이 예시는 rule 관계 설명이며8080이나 특정 ephemeral 범위를 모든 OS의 기본값으로 규정하지 않는다. 주소 범위·protocol·client port 사용 방식이 달라지면 response 조건도 달라진다.

## 중복과 규모의 한계

같은 방향의 traffic이 Stateful과 Stateless에 모두 맞으면 Stateless가 우선하여 tracking하지 않는다. 이 경우 반대 방향의 허용 rule이 필요하므로 broad Stateful allow를 추가했다고 tracking이 복원된다고 생각하면 안 된다. 기존 tracked flow와 rule 변경 후 새 flow의 동작도 구분해야 한다.

Stateful connection table 용량은 Shape에 따라 제한되며 table이 가득 차면 신규 연결의 packet loss가 발생할 수 있다. Stateless에서는 외부 endpoint와의 Path MTU Discovery에 필요한 ICMP traffic도 중요하다. 따라서 선택 기준은 단순히 Stateful은 안전, Stateless는 불안전이라는 이분법이 아니라 traffic volume, 응답 규칙의 정확성, 운영 복잡성이다.

## 관련 개념

- [Network Security Groups](06_Network_Security_Groups.md)
- [Security Lists](07_Security_Lists.md)
- [Ingress / Egress](08_Ingress_Egress.md)

## Hands-on으로 이어가기

개념 설명과 자원 생성 절차는 구분된다. 기존 작업·검증·정리 내용은 [보존된 Hands-on](../../Hands-on/Networking/Original-Provisional/03_Virtual_Cloud_Network_VCN/09_Stateful_Stateless_Rules.md)에서 읽을 수 있다.

## 공식문서

- [Security Rules](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/securityrules.htm)
