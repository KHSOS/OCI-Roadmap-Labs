# Backend Sets

## Backend Set이 맡는 역할

Backend Set은 Load Balancer가 요청을 전달할 서버 목록과 그 목록을 사용하는 규칙을 묶은 논리적 단위다. Load balancing policy, Health Check policy, backend server 목록이 핵심이며 SSL과 session persistence는 선택 구성이다. Listener는 클라이언트 연결을 받는 입구이고 Backend Set은 그 연결 뒤의 처리 대상 집합이다. 작동하는 Load Balancer에는 Backend Set과 하나 이상의 Listener 연결이 필요하다. [Oracle Backend Sets](https://docs.oracle.com/en-us/iaas/Content/Balance/Tasks/managingbackendsets.htm)

## 애플리케이션을 나누는 기준

웹 서비스와 API가 서로 다른 서버 집합이나 health 기준을 사용한다면 별도 Backend Set으로 표현하는 편이 서비스 경계를 명확하게 한다. 반대로 동일 애플리케이션의 복제 서버라면 같은 집합에서 분산 대상이 된다. 서버 목록만 모아 놓는 것이 아니라 그 집합 전체에 적용할 가용성 판정과 분산 정책을 함께 설계하는 것이 중요하다.

예를 들어 Listener가 받은 웹 요청을 HTTP backend 집합에 전달할 때, Health Check는 사용할 수 있는 서버를 판단하고 balancing policy는 그 대상들 사이의 전달 방식을 결정한다. 인증서가 Listener에 연결되는 frontend TLS와 Backend Set에 연결되는 backend TLS는 보호하는 구간이 다르다. [Health Checks](03_Health_Checks.md), [SSL Certificates](04_SSL_Certificates.md)를 함께 읽으면 이 관계를 구분할 수 있다.

## 선택과 변경의 경계

Backend Set의 분리 기준은 애플리케이션 역할, health 의미, backend 보안 요구사항이다. 서버 하나인 집합도 구성할 수 있지만 분산이나 다중 서버 고가용성을 보여 주지는 않는다. Backend Set을 만들었다는 사실만으로 Listener 연결이나 backend 접근 경로가 준비되는 것도 아니다.

Oracle은 balancing policy 변경이 일시적으로 트래픽을 중단하고 활성 연결을 끊을 수 있다고 명시한다. 따라서 정책 변경을 단순한 메타데이터 수정으로 취급하면 안 된다. Backend Set 개수에도 서비스 제한이 있으므로 무제한으로 세분화하는 설계는 적절하지 않다. [Oracle Backend Sets](https://docs.oracle.com/en-us/iaas/Content/Balance/Tasks/managingbackendsets.htm)

## 관련 개념과 실습

상위 개념은 [Load Balancer](00_Load_Balancer.md), 클라이언트 입구는 [Listeners](01_Listeners.md)다. 이 문서는 구성 요소의 역할을 설명한다. 실제 정책 변경, drain 비교, 복원은 아래 보존 실습에 분리되어 있으며 기존 실습은 미검증 초안이다.

[전체 보존 실습](../../Hands-on/Original-Provisional/08_Load_Balancer/02_Backend_Sets.md)

