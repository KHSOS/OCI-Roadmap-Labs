# WAF Policies

WAF Policy는 HTTP traffic에 적용할 access, rate limiting, protection 규칙과 재사용 가능한 action을 묶는 보안 설정이다. Firewall 연결 리소스나 backend 서버와는 다르며, 어느 요청에 어떤 동작을 적용할지를 정의한다.

## 역할과 구성

규칙의 condition은 적용 대상을, action은 조건과 탐지가 충족되었을 때의 동작을 정한다. Allow는 현재 module의 남은 rule을 건너뛰고 Check는 실행을 계속하면서 로그를 남긴다. Return HTTP response는 정의된 status/header/body로 처리를 종료한다. Action을 정의하는 일과 rule이 그 action을 참조하는 일은 별개다.

## 사용 예시

403 응답 action이 policy 안에 있어도 어느 rule도 참조하지 않으면 그것만으로 정상 요청을 차단하지 않는다. 동일한 응답 action을 여러 rule이 참조하면 body 변경이 각 rule의 결과에 영향을 줄 수 있다. Check와 Allow를 같은 뜻으로 선택하면 뒤 rule의 평가 여부가 달라진다.

## 선택 기준과 제약

Request 처리 순서는 Access Control → Rate Limiting → Request Protection → backend이며 이후 Response Access Control과 Response Protection이 이어진다. 앞 module에서 응답을 반환하면 backend 요청이 이루어지지 않는다. Policy 수정은 ACTIVE 상태에서만 가능하다. 공식 기본 한도는 tenancy당 WAF policy 100개와 network address list 100개이며 정책 하나는 여러 Region에 동시에 사용할 수 없다.

## 관련 개념

- [Web Application Firewall (WAF)](00_Web_Application_Firewall_WAF.md)
- [Request Protection Rules](02_Request_Protection_Rules.md)
- [Access Control](03_Access_Control.md)
- [Rate Limiting](04_Rate_Limiting.md)

## 더 읽기

- [Oracle — Getting Started with Web Application Firewall Policies](https://docs.oracle.com/en-us/iaas/Content/WAF/Policies/getting_started_with_waf-policies.htm)
- [Oracle — Adding a Web Application Firewall Action](https://docs.oracle.com/en-us/iaas/Content/WAF/Actions/create_action.htm)
- [Oracle — Overview of Web Application Firewall](https://docs.oracle.com/en-us/iaas/Content/WAF/Concepts/overview.htm)

설정과 실습 절차는 [보존된 WAF Policies Hands-on 문서](../../Hands-on/Original-Provisional/10_Web_Application_Firewall_WAF/01_WAF_Policies.md)에서 읽을 수 있다.
