# Logging

Logging은 서비스와 애플리케이션이 내보내는 진단 기록을 관리하고 검색하는 OCI 서비스다. 숫자의 추세를 보는 Metrics와 달리 개별 요청·오류·상태에 대한 상세 context를 찾는 데 적합하다.

## 역할과 구성

Service Log는 지원 OCI 서비스의 category별 기록, Custom Log는 앱·다른 cloud·on-premises에서 수집한 기록이다. Audit Log는 Audit 서비스의 API 활동 기록이며 읽기 전용으로 검색할 수 있다. 일반 log는 OCID를 갖고 Compartment 안의 Log Group에 속하며 group은 권한·검색·관리의 논리 경계를 제공한다.

## 사용 예시

WAF가 차단한 요청의 rule/action과 backend 앱의 오류를 같은 시간 범위에서 비교하면 traffic이 어디까지 도달했는지 이해할 수 있다. Custom Log는 API로 직접 수집하거나 Unified Monitoring Agent로 보낼 수 있으므로 모든 사용 방식에 VM agent 설치가 필수는 아니다.

## 선택 기준과 제약

Log resource 활성화와 실제 entry 수집, Search 결과는 다른 단계다. Service별 category와 payload schema가 달라 공통 field를 추정하면 잘못된 필터가 된다. Region·Compartment·group/log scope·시간 범위를 먼저 구분한다. UTF-8 text만 지원하며 보존·수집·검색량과 민감 정보 노출을 고려한다. Audit 기록과 사용자가 만든 custom log의 변경·삭제 가능 범위도 동일하지 않다.

## 관련 개념

- [Monitoring](00_Monitoring.md)
- [Metrics](01_Metrics.md)
- [Audit](05_Audit.md)
- [Web Application Firewall (WAF)](../10_Web_Application_Firewall_WAF/00_Web_Application_Firewall_WAF.md)

## 더 읽기

- [Oracle — Logging Overview](https://docs.oracle.com/en-us/iaas/Content/Logging/Concepts/loggingoverview.htm)

설정과 실습 절차는 [보존된 Logging Hands-on 문서](../../Hands-on/Original-Provisional/12_Monitoring/04_Logging.md)에서 읽을 수 있다.
