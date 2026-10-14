# Access Control

WAF Access Control은 HTTP 요청 또는 응답의 속성이 정해진 조건에 맞을 때 Allow, Check, Return HTTP response를 적용하는 기능이다. 여기서 access는 애플리케이션 traffic의 조건부 처리이며 OCI 사용자에게 API 권한을 부여하는 IAM Policy와 다른 계층이다.

## 역할과 구성

Condition은 JMESPath 표현식으로 URL path, method, header, client 정보 같은 속성을 평가한다. Request Access Control은 요청이 backend로 가기 전, Response Access Control은 backend 응답 이후에 동작한다. Allow는 현재 module의 남은 rule만 건너뛰며 전체 WAF 보호를 무조건 해제한다는 뜻이 아니다.

## 사용 예시

Path가 /deny인 요청만 제어하는 정책에서 /deny?test=1의 path 부분도 /deny이지만 /denied는 다른 path다. Query parameter와 path를 분리하면 차단 범위를 더 정확히 설명할 수 있다. IP 기반 규칙은 여러 사용자가 같은 NAT 주소로 보일 수 있어 사용자 계정 단위 권한과 같지 않다.

## 선택 기준과 제약

조건을 넓게 잡으면 정상 요청까지 영향을 받고 앞선 Allow rule은 같은 module 뒤의 제한을 건너뛸 수 있다. Exact match와 prefix match, request와 response 속성, default action과 개별 rule을 구분한다. HTTP 403은 WAF 응답일 수도 앱 응답일 수도 있으며 rule/action 정보가 원인 해석을 돕는다. NSG의 TCP 허용과 WAF의 HTTP 허용은 둘 다 필요한 별도 판단이다.

## 관련 개념

- [Web Application Firewall (WAF)](00_Web_Application_Firewall_WAF.md)
- [WAF Policies](01_WAF_Policies.md)
- [Request Protection Rules](02_Request_Protection_Rules.md)
- [Rate Limiting](04_Rate_Limiting.md)

## 더 읽기

- [Oracle — Adding a Request Control Rule to a Web Application Firewall Policy](https://docs.oracle.com/en-us/iaas/Content/WAF/AccessControl/create_access_rule_request_control.htm)
- [Oracle — Overview of Web Application Firewall](https://docs.oracle.com/en-us/iaas/Content/WAF/Concepts/overview.htm)
- [Oracle — Understanding Conditions](https://docs.oracle.com/en-us/iaas/Content/WAF/Concepts/understanding_conditions.htm)

설정과 실습 절차는 [보존된 Access Control Hands-on 문서](../../Hands-on/Original-Provisional/10_Web_Application_Firewall_WAF/03_Access_Control.md)에서 읽을 수 있다.
