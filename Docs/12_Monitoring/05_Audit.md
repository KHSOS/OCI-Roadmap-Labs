# Audit

Audit는 지원 OCI public API endpoint 호출을 자동으로 기록하는 서비스다. 누가 언제 어떤 resource에 어떤 작업을 요청했고 어떤 응답을 받았는지 추적하는 데 사용한다. 앱의 access log나 database SQL audit와는 기록 대상이 다르다.

## 역할과 구성

Console, CLI, SDK, custom client, 다른 OCI service의 API 호출이 기록 대상이다. Event에는 시각, source/identity, target resource, request 정보, response 정보가 담긴다. Request identifier와 resource·operation을 함께 보면 하나의 작업을 다른 관측 정보와 연결할 수 있다.

## 사용 예시

Bucket 생성 API의 caller, resource name, response status를 살펴보면 실제 생성 작업과 이후 Notifications 수신 시각을 구분할 수 있다. Email이 늦게 도착한 것과 API 호출이 실패한 것은 같은 사건이 아니다. 삭제된 resource도 기록에 남을 수 있으므로 현재 목록 부재만으로 과거 활동이 사라지지는 않는다.

## 선택 기준과 제약

Audit의 Object Storage 범위는 bucket 관련 event이며 object별 읽기·쓰기는 포함하지 않는다. Object access 진단은 service log 등 해당 기록을 비교한다. Version 2 schema에는 version 1에서 변환된 기록의 일부 field가 비어 있을 수 있어 빈 field를 만들어 보완하지 않는다. Audit 데이터는 read-only이며 민감한 principal/request 정보의 열람 scope를 제한한다. 미관측은 잘못된 Region·Compartment·시간 범위 또는 반영 지연과 구분한다.

## 관련 개념

- [Events](03_Events.md)
- [Logging](04_Logging.md)
- [Notifications](06_Notifications.md)

## 더 읽기

- [Oracle — Overview of Audit](https://docs.oracle.com/en-us/iaas/Content/Audit/Concepts/auditoverview.htm)
- [Oracle — Logging Overview](https://docs.oracle.com/en-us/iaas/Content/Logging/Concepts/loggingoverview.htm)

설정과 실습 절차는 [보존된 Audit Hands-on 문서](../../Hands-on/Original-Provisional/12_Monitoring/05_Audit.md)에서 읽을 수 있다.
