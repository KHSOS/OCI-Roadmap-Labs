# Health Check Policies

Health Check Policy는 NLB가 backend를 새 트래픽의 대상으로 사용할 수 있는지 판단하기 위한 주기적 검사 조건이다. Backend Set에 연결되며 정상적인 health 기반 분배에서는 검사 실패 시 서버를 rotation에서 일시 제외하고 회복 시 다시 포함한다. 모든 backend가 실패한 상황의 전달 여부는 Backend Set의 별도 동작 설정까지 함께 고려해야 한다. 서비스 lifecycle이 Active인 것과 애플리케이션이 정상 응답하는 것은 같은 의미가 아니다.

## 역할과 구성

TCP 검사는 연결 성립을 확인하고, HTTP 검사는 지정 URI의 status 또는 body를 확인한다. 검사 protocol·port·path·기대 응답·interval·timeout·retry 조건이 무엇을 건강하다고 볼지 결정한다. Health 검사 port는 client traffic listener port와 같은 개념이 아니다. 실제 probe source IP는 backend details 또는 HealthCheckResult의 sourceIpAddress로 제공되며 NLB VIP로 단정할 수 없다.

## 사용 예시

TCP 연결은 열리지만 HTTP /health가 500인 web backend를 생각할 수 있다. TCP handshake만 검사하면 healthy로 판단할 수 있지만 HTTP 검사에서는 기대 status와 달라 제외될 수 있다. 반대로 검사 URI에 인증이 걸려 401이 나오면 실제 업무 endpoint가 정상이어도 검사 조건 때문에 unhealthy가 될 수 있다.

## 선택 기준과 제약

검사 대상은 애플리케이션의 준비 상태를 대표하되 불필요하게 무거운 작업은 피하는 방향으로 설계한다. 너무 민감한 timeout은 일시 지연을 장애로, 너무 느린 주기는 장애를 늦게 반영할 수 있다. NSG·Security List·guest firewall가 probe 경로를 막으면 앱 장애와 구분해야 한다. 공식 문서의 health status 표시는 3분마다 갱신되고 과거 health 이력을 제공하지 않는다. Probe interval과 Console status 표시 주기를 혼동하지 않는다.

## 관련 개념

- [Network Load Balancer](00_Network_Load_Balancer.md)
- [IP Address Preservation](02_IP_Address_Preservation.md)
- [DNS Health Checks](../11_DNS/03_DNS_Health_Checks.md)

## 더 읽기

- [Oracle — Health Check Policies for Network Load Balancers](https://docs.oracle.com/en-us/iaas/Content/NetworkLoadBalancer/HealthCheckPolicies/health-check-policy-management.htm)

설정과 실습 절차는 [보존된 Health Check Policies Hands-on 문서](../../Hands-on/Networking/Original-Provisional/09_Network_Load_Balancer/03_Health_Check_Policies.md)에서 읽을 수 있다.
