# Web Application Firewall (WAF)

Web Application Firewall (WAF)는 HTTP 요청·응답을 애플리케이션 보안 규칙으로 검사하여 악성 또는 원치 않는 traffic을 제어한다. SQL Injection, XSS 같은 웹 공격 패턴과 접근 조건을 다루며, IP/port 허용만 판단하는 NSG의 역할을 대체하지 않는다.

## 역할과 구성

Regional WAF Policy는 규칙과 action을 담고 Firewall resource는 policy와 Load Balancer라는 enforcement point를 연결하는 논리 리소스다. Policy가 존재해도 연결된 처리 경로가 없으면 해당 앱 요청을 보호하지 않는다. Edge Policy는 전 세계 Oracle node와 DNS/CNAME을 사용하는 별도 모델이며 regional policy와 같은 배포 구조가 아니다.

## 사용 예시

HTTP 앱에 대해 일반 요청은 backend로 보내고 의심스러운 요청은 Check로 기록하거나 정의된 HTTP 응답으로 끝낼 수 있다. 앱 자체의 403과 WAF가 반환한 403은 같은 status지만 원인이 다르므로 어떤 rule/action이 적용되었는지가 해석에 중요하다. TLS, 애플리케이션 인증, 안전한 SQL 처리는 여전히 별도 보안 계층이다.

## 선택 기준과 제약

Regional WAF Policy는 Load Balancer의 HTTP listener를 지원하고 NLB나 TCP listener에는 사용할 수 없다. Policy는 regional 범위이며 같은 Region의 여러 LB에서 재사용할 수 있다. Edge와 Regional 사이의 자동 변환 도구는 없고 설정을 수동으로 재구성해야 한다. 오탐을 고려해 조건·capability·action을 구분하며 WAF를 모든 공격과 앱 취약점의 완전한 해결책으로 보지 않는다.

## 관련 개념

- [WAF Policies](01_WAF_Policies.md)
- [Request Protection Rules](02_Request_Protection_Rules.md)
- [Access Control](03_Access_Control.md)
- [Rate Limiting](04_Rate_Limiting.md)
- [Logging](../12_Monitoring/04_Logging.md)

## 더 읽기

- [Oracle — Overview of Web Application Firewall](https://docs.oracle.com/en-us/iaas/Content/WAF/Concepts/overview.htm)
- [Oracle — Getting Started with Web Application Firewall Policies](https://docs.oracle.com/en-us/iaas/Content/WAF/Policies/getting_started_with_waf-policies.htm)

설정과 실습 절차는 [보존된 Web Application Firewall (WAF) Hands-on 문서](../../Hands-on/Original-Provisional/10_Web_Application_Firewall_WAF/00_Web_Application_Firewall_WAF.md)에서 읽을 수 있다.
