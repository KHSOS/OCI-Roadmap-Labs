# Request Protection Rules

Request Protection Rules는 HTTP 요청의 악성 content를 protection capability로 검사하는 WAF 규칙이다. 특정 IP나 URL 자체를 제한하는 Access Control과 달리, 선택한 보안 검사 기능이 요청 내용에서 무엇을 탐지하는지가 핵심이다.

## 역할과 구성

Protection rule에는 적용 condition, rule action, 선택한 capability와 version, 필요 시 body inspection 설정이 들어간다. 각 capability는 action을 별도로 지정할 수 있으므로 rule-level action만으로 전체 검사 결과를 설명할 수 없다. Capability는 나열된 순서대로 반드시 실행되는 것은 아니다.

## 사용 예시

같은 /search 요청에서 일반 query와 SQLi 형태의 query를 비교하는 상황이라면 URL condition은 검사 범위를 제한하고 SQLi capability는 query의 악성 패턴을 판단한다. Check 결과는 로그를 남기면서 앱 응답을 계속 받을 수 있고 Return HTTP response는 backend 처리 전에 응답을 끝낼 수 있다. 예시 문자열 하나가 항상 탐지된다고 보장하지 않는다.

## 선택 기준과 제약

요청 body를 검사해야 하는 API와 query/header 중심 요청은 필요한 검사 범위가 다르다. Capability version, action override, exclusion, collaborative weight/threshold는 탐지와 enforcement에 영향을 준다. 정상 입력의 오탐도 고려해야 하며, 단지 403이 나왔다는 사실만으로 특정 SQLi capability의 동작을 단정할 수 없다. WAF 검사와 애플리케이션의 parameterized query·입력 검증은 서로 보완한다.

## 관련 개념

- [Web Application Firewall (WAF)](00_Web_Application_Firewall_WAF.md)
- [WAF Policies](01_WAF_Policies.md)
- [Access Control](03_Access_Control.md)
- [Rate Limiting](04_Rate_Limiting.md)

## 더 읽기

- [Oracle — Adding a Web Application Firewall Request Protection Rule](https://docs.oracle.com/en-us/iaas/Content/WAF/Protections/create_request_control.htm)
- [Oracle — Overview of Web Application Firewall](https://docs.oracle.com/en-us/iaas/Content/WAF/Concepts/overview.htm)

설정과 실습 절차는 [보존된 Request Protection Rules Hands-on 문서](../../Hands-on/Original-Provisional/10_Web_Application_Firewall_WAF/02_Request_Protection_Rules.md)에서 읽을 수 있다.
