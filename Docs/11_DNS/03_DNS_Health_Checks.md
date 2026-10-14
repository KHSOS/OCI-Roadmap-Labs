# DNS Health Checks

DNS Health Checks는 Traffic Management와 함께 endpoint의 가용성을 판단할 때 사용하는 OCI Health Checks를 설명하는 학습 주제다. 이름 자체나 zone의 record 존재만 검사하는 기능이 아니라 외부 위치에서 공개 endpoint에 HTTP 또는 ping probe를 보내는 서비스다.

## 역할과 구성

Monitor는 주기적 검사, On-demand Probe는 일회성 검사, Vantage Point는 검사를 실행하는 지리적 위치다. Probe 결과에는 endpoint가 외부에서 응답하는지와 latency 정보가 담긴다. Traffic Management는 health 결과를 DNS answer 선택에 활용할 수 있지만 monitor를 만들었다고 steering policy가 자동 연결되는 것은 아니다.

## 사용 예시

공개 웹 endpoint가 LB 내부 health에서는 정상인데 특정 외부 vantage에서 timeout이면 외부 network path나 접근 정책까지 함께 봐야 한다. 반대로 HTTP 200이 나와도 그 path가 대표하지 않는 DB transaction이나 로그인 기능의 정상 여부까지 알 수는 없다. 여러 vantage는 특정 경로 문제와 광범위한 endpoint 장애를 구분하는 데 도움이 된다.

## 선택 기준과 제약

대상은 public-facing endpoint이며 private LB에 대한 내부 NLB/LB health 검사와 다르다. 공식 문서는 서로 다른 provider의 최소 세 vantage 사용을 권장한다. 10초 interval은 Premium, 그보다 긴 interval은 Basic으로 분류되며 검사 빈도 외의 동작 차이는 없다. On-demand는 REST API로 제공되고 24시간 사용 한도가 있다. Probe를 허용할 network source 정보와 정책은 별도로 확인해야 하며 vantage 이름만으로 source CIDR를 추정하지 않는다.

## 관련 개념

- [DNS](00_DNS.md)
- [Traffic Management](02_Traffic_Management.md)
- [Health Check Policies](../09_Network_Load_Balancer/03_Health_Check_Policies.md)
- [Monitoring](../12_Monitoring/00_Monitoring.md)

## 더 읽기

- [Oracle — Overview of the Health Checks Service](https://docs.oracle.com/en-us/iaas/Content/HealthChecks/Concepts/healthchecks.htm)
- [Oracle — Overview of Traffic Management](https://docs.oracle.com/en-us/iaas/Content/TrafficManagement/Concepts/overview.htm)

설정과 실습 절차는 [보존된 DNS Health Checks Hands-on 문서](../../Hands-on/Networking/Original-Provisional/11_DNS/03_DNS_Health_Checks.md)에서 읽을 수 있다.
