# Infrequent Access

Infrequent Access(IA)는 읽는 빈도는 낮지만 필요할 때 즉시 접근해야 하는 Object Storage 데이터의 cool tier다. Standard보다 저장 비용이 낮고 Archive와 달리 읽기 전 복원이 필요하지 않다. 드물게 읽는 백업, 온프레미스 데이터의 백업, 다른 Region에서 복제·복사한 데이터처럼 온라인 접근은 유지하되 읽기량이 작은 경우에 적합하다. [Oracle Infrequent Access](https://docs.oracle.com/en-us/iaas/Content/Object/Concepts/understandingstoragetiers.htm)

## 낮은 저장 단가의 조건

IA에는 31일 최소 저장기간과 GiB당 데이터 검색 요금이 있다. 최소기간 전에 객체를 삭제하거나 덮어써서 데이터가 제거되면 남은 최소기간에 대한 일할 비용이 부과될 수 있다. Versioning이 기존 데이터를 이전 버전으로 남겨 제거하지 않는 경우는 공식 문서의 예외를 구분해야 한다.

이 최소기간은 객체를 31일 동안 삭제할 수 없다는 보존 잠금이 아니다. 삭제 가능 여부와 비용 의무는 다른 문제다. 작은 임시 파일을 잠깐 저장하고 지우는 용도나 반복적으로 많은 데이터를 읽는 용도에는 낮은 저장 단가만으로 IA를 선택하지 않는다. [최소기간·검색·Versioning 조건](https://docs.oracle.com/en-us/iaas/Content/Object/Concepts/understandingstoragetiers.htm)

## Bucket와 다른 tier의 관계

IA는 bucket 생성 때 선택하는 default tier가 아니다. Standard default bucket에 객체를 IA로 지정할 수 있으며, 같은 bucket 안에 Standard·IA·Archive 객체가 함께 있을 수 있다. 개별 객체의 tier 선택과 bucket 기본값을 분리해서 이해해야 한다. [Storage Tiers](03_Storage_Tiers.md)

[Standard Storage](05_Standard_Storage.md)도 온라인 접근을 제공하지만 저장 단가와 검색 요금 구조가 다르다. [Archive](04_Archive.md)는 더 긴 최소 저장기간과 복원 대기가 있는 cold tier다. IA를 읽을 때 문제가 발생했다는 이유로 Archive 복원을 호출하는 것은 해당 tier의 접근 모델과 맞지 않는다.

## 수동 선택과 자동 이동

특정 데이터가 드물게 읽힌다는 사실을 알고 있으면 객체 단위로 IA를 선택할 수 있다. 접근 패턴을 알 수 없거나 변화가 크면 Standard bucket의 Auto-Tiering이 Standard와 IA 사이를 관리하는 대안이다. 다만 Auto-Tiering은 1 MiB보다 큰 객체, 이동 자격 조건, 서비스 권한, IA 전환 Lifecycle 규칙과의 충돌 제약을 가진다.

IA에서 Archive로 이동할 때도 IA 검색 요금이 적용되며, 수동 변경인지 Lifecycle·Auto-Tiering 관련 전환인지에 관계없이 공식 비용 조건을 확인해야 한다. 자동 기능을 사용한다고 모든 전환 비용이 사라지는 것은 아니다. [Auto-Tiering와 tier 이동 비용](https://docs.oracle.com/en-us/iaas/Content/Object/Concepts/understandingstoragetiers.htm)

## 선택 기준

보관기간이 최소기간보다 충분히 길고 읽기량은 적으며 요청 즉시 데이터가 필요한지 판단한다. 읽기량이 늘어나는 데이터라면 Standard와 저장·검색 총비용을 비교한다. 읽기 지연을 허용하는 매우 장기적인 자료라면 Archive와 비교하되 복원 운영과 임시 Standard 저장 비용도 포함한다. IA는 접근 패턴에 맞춘 비용 선택이며 IAM 권한이나 공개 접근 설정을 대신하지 않는다.

## Oracle 공식 참고자료

- [Object Storage Storage Tiers — Infrequent Access, Auto-Tiering, 전환 조건](https://docs.oracle.com/en-us/iaas/Content/Object/Concepts/understandingstoragetiers.htm)

## 보존 실습으로 이동

[기존 전체 실습 초안](../../Hands-on/Original-Provisional/06_Object_Storage/06_Infrequent_Access.md)에서 Infrequent Access 비교의 기존 실습 절차를 읽을 수 있다. 보존된 초안으로, OCI 실행 검증은 완료되지 않았다.

