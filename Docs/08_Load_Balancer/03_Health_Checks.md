# Health Checks

## 트래픽을 받을 준비가 되었는가

Load Balancer의 Health Check는 backend server의 가용성을 반복해서 검사하여 트래픽 분배 대상에 포함할지를 판단한다. 지정한 간격마다 검사하고 실패한 서버를 rotation에서 일시적으로 제외하며, 다시 통과하면 rotation으로 돌려보낸다. 서버를 삭제하거나 애플리케이션 자체를 복구하는 기능은 아니다. [Oracle Health Check Policies](https://docs.oracle.com/en-us/iaas/Content/Balance/Tasks/load_balancer_health_management.htm)

## TCP와 HTTP가 답하는 질문

TCP-level 검사는 TCP three-way handshake가 성립하는지를 본다. 연결을 받아들이는 프로세스가 있다는 신호지만 애플리케이션이 올바른 응답을 만들 수 있다는 보장은 아니다. HTTP-level 검사는 특정 URI로 요청하고 응답 status code 또는 body를 검증한다. 웹/API에서 실제 서비스 준비 상태를 표현하는 endpoint가 있다면 HTTP 검사가 더 구체적인 의미를 갖는다.

예를 들어 TCP 연결이 가능한 웹 서버라도 health URI가 틀리거나 예상 status와 다르면 HTTP Health Check는 실패할 수 있다. 반대로 단순한 정적 페이지의 성공은 모든 업무 의존성이 정상이라는 뜻이 아니다. 어떤 준비 상태를 endpoint가 대표할지는 애플리케이션 설계의 선택이다. [Oracle Health Check Policies](https://docs.oracle.com/en-us/iaas/Content/Balance/Tasks/load_balancer_health_management.htm)

## Backend Set 및 TLS와의 관계

Health Check policy는 [Backend Set](02_Backend_Sets.md)을 만들 때 구성하며 그 안의 서버들을 대상으로 한다. 클라이언트가 Listener에 접속하는 경로와 Load Balancer가 backend에 검사 요청을 보내는 경로는 서로 다르다. backend healthy는 클라이언트의 DNS, frontend 규칙, Listener 또는 인증서까지 모두 정상이라는 증거가 아니다.

TCP와 HTTP Health Check에는 SSL을 사용할 수 있다. Use SSL로 구성된 Backend Set의 HTTP 검사는 plain text health check를 별도로 선택하지 않았다면 HTTPS를 사용한다. 따라서 서비스 트래픽과 검사 트래픽의 암호화 선택이 불일치하지 않는지 판단해야 한다. [SSL Certificates](04_SSL_Certificates.md)

## 관측 결과를 해석하는 한계

검사가 주기적으로 실행되므로 설정 변경과 health 표시는 동일 순간의 관측이 아니다. 경로, 응답 조건, 검사 간격은 장애 감지의 의미를 결정한다. 너무 넓은 검사는 업무 장애를 놓칠 수 있고, 실제 준비 상태와 무관한 endpoint는 정상 서버를 제외할 수 있다. Overall Health와 개별 Backend Health는 집합 요약과 개별 서버 관측을 각각 보여 준다. DNS용 외부 Health Checks와 이 Backend Set 검사를 혼동하지 않는다. [Oracle Health Check Policies](https://docs.oracle.com/en-us/iaas/Content/Balance/Tasks/load_balancer_health_management.htm)

## 관련 실습

아래 보존 초안은 잘못된 health 경로와 복원의 관계를 다룬다. 절차는 개념 본문에서 분리했으며 OCI 실행 검증을 뜻하지 않는다.

[전체 보존 실습](../../Hands-on/Original-Provisional/08_Load_Balancer/03_Health_Checks.md)

