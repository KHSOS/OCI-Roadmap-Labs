# Listeners

Listener는 Load Balancer IP에서 들어오는 트래픽을 특정 protocol과 port로 받는 logical entity다. Backend application을 설치하거나 server에서 port를 여는 기능이 아니라 LB의 frontend 접속과 routing을 정의한다.

## frontend와 backend의 구분

Listener는 기본 Backend Set을 참조한다. Client가 frontend port 8081로 들어와도 Backend Set의 server가 application port 8080으로 응답할 수 있다. 두 port가 같아야 하는 것은 아니다. Health Check의 port와 URI도 별도로 정의된다.

하나의 LB에 여러 Listener를 두면 같은 IP에서 서로 다른 protocol·port의 접속을 제공할 수 있다. Listener를 추가한다고 Backend Set이나 server가 자동으로 늘어나는 것은 아니다. Network 보안 규칙은 해당 frontend port를 허용해야 하고 LB에서 backend로 가는 경로도 따로 필요하다.

## protocol과 SSL의 선택

지원 protocol에는 HTTP, HTTPS, HTTP/2, gRPC, TCP가 있다. HTTP 계열 Listener는 application-level 요청 처리를, TCP Listener는 transport-level 연결을 다룬다. 현재 공식 문서는 HTTP/HTTPS의 HTTP 1.0·1.1과 HTTP/2 Listener를 설명하며 HTTP L7 backend 통신은 HTTP 1.1로 proxy한다고 명시한다. Client와 backend의 HTTP version이 반드시 같은 것은 아니다.

HTTPS·HTTP/2·gRPC Listener에는 SSL 구성이 필요하며 HTTP·TCP에서는 선택적이다. Listener의 SSL certificate bundle과 backend SSL은 별도 관계다. HTTPS frontend가 있다고 backend도 자동으로 TLS를 사용하는 것은 아니다. Certificate source는 Certificates service-managed 또는 LB-managed 방식을 선택할 수 있다.

## routing 옵션의 관계

- **Virtual Hostname**은 Listener의 request routing에 쓰는 이름이다. LB configuration에 정의된 hostname을 참조하며 hostname 선택 자체가 DNS record 생성은 아니다.
- **Routing Policy 또는 Path Route Set**은 요청 특성이나 path에 따라 Backend Set을 선택하는 선택적 routing 기능이다.
- **Rule Set**은 Listener traffic에 적용할 별도 규칙 모음이다. Path Route Set과 같은 자원이 아니다.
- **Proxy Protocol**은 backend에 연결 정보를 전달하는 format이다. Backend가 해당 header format을 이해할 수 있어야 한다.

복잡한 routing이 필요 없는 접속은 default Backend Set 관계로 이해할 수 있다. Listener에서 routing/rule set 연결을 해제하는 것과 그 자원 자체를 삭제하는 것은 다르다.

## 이름·timeout·수량의 한계

Listener name은 LB 안에서 유일해야 하고 생성 후 변경할 수 없다. Virtual Hostname은 Listener에 최대 16개 선택할 수 있으며, 공식 LB overview의 Listener 한도는 LB당 16개다.

Idle timeout은 HTTP request-response 중 연속된 두 receive 또는 두 send network I/O 사이의 최대 idle 시간이며 최대 7200초다. 요청 전체 처리시간이나 backend application의 고유 timeout과 같지 않다. Listener 설정만으로 network 허용·healthy backend·application 응답을 보장하지 않으므로 이 관계들을 나누어 이해하는 것이 중요하다.

## 함께 읽기

- [Load Balancer](00_Load_Balancer.md)
- [Backend Sets](02_Backend_Sets.md)
- [Health Checks](03_Health_Checks.md)
- [SSL Certificates](04_SSL_Certificates.md)
- [보존된 Hands-on 초안](../../Hands-on/Original-Provisional/08_Load_Balancer/01_Listeners.md) — 실습 절차는 별도 자료에서 다룬다.

## 공식 참고 문서

- [Listeners for Load Balancers](https://docs.oracle.com/en-us/iaas/Content/Balance/Tasks/managinglisteners.htm)
- [Creating a Load Balancer Listener](https://docs.oracle.com/en-us/iaas/Content/Balance/Tasks/managinglisteners_topic-Creating_Listeners.htm)
- [Overview of Load Balancer](https://docs.oracle.com/en-us/iaas/Content/Balance/Concepts/balanceoverview.htm)
- [Load Balancer Concepts](https://docs.oracle.com/en-us/iaas/Content/Balance/Concepts/balanceoverview_topic-Load_Balancing_Concepts.htm)
