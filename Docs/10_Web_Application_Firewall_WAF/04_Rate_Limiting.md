# Rate Limiting

Rate Limiting은 HTTP 요청 빈도를 제한하여 과도한 사용이나 abuse가 앱에 미치는 영향을 줄이는 WAF 기능이다. Content가 악성인지 검사하는 Protection Rule과 달리 일정 시간에 얼마나 많은 요청이 발생했는지가 중심이다.

## 역할과 구성

Requests limit는 요청 수 기준, Period는 그 수를 평가할 시간 범위, Action duration은 제한 동작이 유지될 기간이다. Condition은 적용할 endpoint 범위를 정하고 action은 제한 시 반환할 HTTP 응답이나 Check 동작을 정한다. Regional WAF의 공식 rate rule은 각 unique client IP 기준으로 빈도를 제한한다.

## 사용 예시

/api/search에 짧은 시간 동안 요청이 몰리면 그 경로에만 429 응답을 반환하는 정책을 생각할 수 있다. /health 같은 다른 경로는 별도 condition에 의해 제한 대상에서 제외될 수 있다. 같은 NAT를 쓰는 이용자들은 WAF가 보는 IP가 같아 서로의 요청이 같은 한도에 영향을 줄 수 있다.

## 선택 기준과 제약

Period와 action duration을 같은 필드로 보지 않는다. Endpoint마다 정상 burst가 다르므로 지나치게 낮은 한도는 오탐성 제한을 만들 수 있다. 정확히 특정 번째 요청부터 전역적으로 차단된다는 단순 모델로 해석하지 말고 정책 적용·분산 처리와 실제 traffic을 고려한다. Request Access Control이 Rate Limiting보다 먼저 평가되고 Request Protection은 뒤에 평가된다. Rate Limiting은 앱의 사용자별 quota나 재시도·backoff 설계를 대체하지 않는다.

## 관련 개념

- [Web Application Firewall (WAF)](00_Web_Application_Firewall_WAF.md)
- [WAF Policies](01_WAF_Policies.md)
- [Request Protection Rules](02_Request_Protection_Rules.md)
- [Access Control](03_Access_Control.md)

## 더 읽기

- [Oracle — Adding a Rate Limiting Rule to a Web Application Firewall Policy](https://docs.oracle.com/en-us/iaas/Content/WAF/RateLimiting/create_rate_limiting_rule.htm)
- [Oracle — Getting Started with Web Application Firewall Policies](https://docs.oracle.com/en-us/iaas/Content/WAF/Policies/getting_started_with_waf-policies.htm)

설정과 실습 절차는 [보존된 Rate Limiting Hands-on 문서](../../Hands-on/Original-Provisional/10_Web_Application_Firewall_WAF/04_Rate_Limiting.md)에서 읽을 수 있다.
