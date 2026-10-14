# Object Lifecycle Management

Object Lifecycle Management는 Bucket 안의 데이터를 **정해진 수명 규칙에 따라 자동 이동하거나 삭제하는 기능**이다. 사람이 매번 오래된 자료를 찾아 처리하는 부담을 줄이고, 접근 빈도와 보관 목적에 맞는 저장 Tier를 사용하도록 돕는다. 한 Bucket의 규칙들을 묶어 Lifecycle Policy라고 부른다.

## 어떤 데이터를 관리하는가

|관리 대상|가능한 역할|
|---|---|
|Object|다른 Storage Tier로 이동하거나 삭제한다.|
|이전 Object Version|Versioning 환경에서 이전 자료의 수명을 관리한다.|
|미완료·실패 Multipart Upload|확정되지 않은 Part를 삭제해 남은 저장량을 관리한다.|

예를 들어 오래된 분석 자료는 즉시 읽는 Standard에서 보관용 Archive로 옮기고, 더 오래된 자료는 삭제하는 관계를 생각할 수 있다. 반면 자주 읽는 최신 자료와 법적으로 보존해야 하는 자료에 같은 삭제 규칙을 적용하면 안 된다. Lifecycle은 보관 목적과 대상 집합을 먼저 정해야 하는 자동화다.

## 대상 필터와 규칙의 관계

Object 이름 Prefix나 전체 이름의 Pattern을 이용해 처리할 자료 집합을 정할 수 있다. logs/라는 Prefix는 Object 이름의 일부이며 실제 디렉터리 리소스가 아니다. Pattern은 전체 이름에 적용되고, 포함·제외 조건은 대상이 얼마나 넓어지는지를 결정한다.

공식 문서는 Pattern exclusion → Pattern inclusion → Prefix inclusion 순서의 우선순위를 설명한다. 전체 Object 대상과 이름 필터 대상을 구분해야 하며, 빈 필터를 좁은 범위라고 해석해서는 안 된다. 한 Bucket은 최대 1,000개 Rule을 담는 하나의 Lifecycle Policy를 가질 수 있다.

## 수명 기준과 비동기 처리

삭제 Policy는 Object의 생성 또는 마지막 덮어쓰기 시각에 관계되는 Last Modification Time을 UTC의 다음 일자 경계로 올림해 계산한다. 플랫폼은 하루에 한 번 Lifecycle Policy를 실행하며 생성·변경 후 반영과 첫 실행 시작에 최대 24시간이 걸릴 수 있다.

규칙이 Trigger된 뒤 보통 10분 안에 실행된다는 설명도 있지만 Best-effort이며 더 오래 걸릴 수 있다. Trigger·첫 실행·많은 Object에 대한 처리 완료는 서로 다른 시점이다. Rule이 저장되어 있다는 사실만으로 Object가 이미 이동되거나 삭제되었다고 볼 수 없고, 처리 전에는 현재 Tier 기준의 과금이 계속된다.

## 권한과 변경 범위

Policy를 관리하는 사용자의 권한과 Object Storage 서비스가 사용자를 대신해 이동·삭제하는 권한은 별개다. Object Storage는 지역 서비스이므로 Lifecycle을 사용하는 Region의 서비스 주체에 해당 범위의 권한이 필요하다.

Console은 개별 Rule을 관리할 수 있지만 CLI/API로 Policy를 갱신하는 경우 새 Policy가 기존 내용을 대체하는 모델이다. 새 Rule 하나만 고려해 다른 Rule을 빠뜨리면 기존 수명 관리 의도가 사라질 수 있다. 관리 방식 선택에서 기존 전체 Policy와 각 Rule의 책임 범위를 함께 생각해야 한다.

## 비용과 복구 한계

Tier 이동은 저장비와 접근 지연의 선택이다. Infrequent Access에서 Archive로 이동할 때에도 Infrequent Access Retrieval 요금이 관계된다. Archive로 옮기는 것이 즉시 온라인 읽기를 유지하는 선택은 아니다.

Lifecycle이 삭제한 Object는 해당 삭제를 되돌리는 복구를 기대할 수 없다. 자동 삭제는 법적 보존을 구현하는 Retention Rule이나 별도 백업과 같은 기능이 아니다. 삭제 대상·수명·기존 Version의 처리와 보존 의무가 서로 맞아야 한다.

## 함께 읽기

[Buckets / Objects](01_Buckets_Objects.md)는 이름·Prefix·Version의 관계를, [Storage Tiers](03_Storage_Tiers.md)는 데이터 접근 패턴의 선택을 설명한다. [보존된 Object Lifecycle Management 실습](../../Hands-on/Original-Provisional/06_Object_Storage/02_Object_Lifecycle_Management.md)은 기존 Rule 설정과 비동기 관찰 절차를 담은 OCI 실행 미검증 초안이다.

## 공식 참고 문서

- [Object Storage Object Lifecycle Management — Rule Targets, Filters, Timing and Service Permissions](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/usinglifecyclepolicies.htm)
- [Object Storage Objects — Prefixes and Uncommitted Multipart Uploads](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/managingobjects.htm)
