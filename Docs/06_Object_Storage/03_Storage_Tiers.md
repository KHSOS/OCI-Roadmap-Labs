# Storage Tiers

Object Storage의 Storage Tier는 객체를 얼마나 자주, 얼마나 빨리 읽어야 하는지에 맞춰 저장 비용과 읽기 조건을 선택하는 속성이다. 객체마다 tier가 있고, 객체의 tier가 저장 비용과 관련 검색 요금을 결정한다. Standard·Infrequent Access·Archive는 서로 다른 접근 패턴을 위한 선택이지 동일한 기능의 가격표만 다른 이름이 아니다. [Oracle Storage Tiers](https://docs.oracle.com/en-us/iaas/Content/Object/Concepts/understandingstoragetiers.htm)

## Bucket 기본값과 객체 속성

Bucket 생성 때 정하는 default tier는 Standard 또는 Archive이며 이후 바꿀 수 없다. 그러나 Standard bucket에는 Standard·Infrequent Access·Archive 객체가 함께 들어갈 수 있다. 객체 tier가 바뀌거나 Archive 객체가 복원되어도 그 객체가 다른 bucket으로 이동하는 것은 아니다. Archive bucket은 Archive 객체만 포함하므로 Standard bucket과 같은 혼합 보관 구조로 취급하면 안 된다.

이 구분은 데이터 집합 안에서 일부만 장기 보관하거나 접근 패턴이 변할 때 중요하다. Bucket의 default는 새 객체의 기본 선택이고, 현재 객체의 실제 tier는 별도로 확인할 속성이다. [Bucket와 객체 tier의 관계](https://docs.oracle.com/en-us/iaas/Content/Object/Concepts/understandingstoragetiers.htm)

## 접근 요구에 따른 선택

|선택|적합한 요구|비용·시간 경계|
|---|---|---|
|[Standard Storage](05_Standard_Storage.md)|자주 읽는 콘텐츠·로그·분석 데이터, 즉시 접근이 중요한 백업|세 tier 중 저장 단가가 높지만 최소 저장기간과 tier 검색 요금은 없음|
|[Infrequent Access](06_Infrequent_Access.md)|읽는 빈도는 낮아도 요청 즉시 데이터가 필요|31일 최소 저장기간과 GiB당 검색 요금|
|[Archive](04_Archive.md)|거의 읽지 않는 장기 기록·감사 자료|90일 최소 저장기간, 읽기 전에 복원 대기 및 복원된 데이터의 Standard 저장 요금|

낮은 저장 단가만으로 IA나 Archive를 선택하면 자주 읽는 데이터의 검색 비용이나 복원 지연을 놓친다. 최소 저장기간은 데이터 삭제를 금지하는 retention rule과 다르다. 데이터가 실제 제거되는 조기 삭제·덮어쓰기에는 남은 기간에 대한 비용이 발생할 수 있다. Versioning으로 이전 버전이 남아 제거되지 않는 경우에는 공식 문서가 설명하는 과금 예외를 구분해야 한다.

## 수동 tier, Lifecycle, Auto-Tiering

수동 tier 선택은 특정 객체의 요구를 명시한다. [Object Lifecycle Management](02_Object_Lifecycle_Management.md)는 규칙에 따라 객체를 전환하거나 삭제하는 별도 기능이다. Auto-Tiering은 Standard bucket에서 접근 패턴을 관찰하여 Standard와 IA 사이를 자동으로 조정한다. Archive 복원을 대신하는 기능은 아니다.

Auto-Tiering은 1 MiB보다 큰 객체를 대상으로 하며 활성화 직후 모든 객체가 IA로 이동하는 것은 아니다. 접근·보관 조건을 충족해야 이동 대상이 된다. IA로 이동시키는 Lifecycle 규칙과 Auto-Tiering은 함께 활성화할 수 없고, Object Storage 서비스가 객체를 관리할 권한도 필요하다. IA에서 Archive로 이동하는 경로에는 IA 검색 요금이 적용된다. [Auto-Tiering 조건과 권한](https://docs.oracle.com/en-us/iaas/Content/Object/Concepts/understandingstoragetiers.htm)

## 설계 판단의 범위

Tier는 비용과 데이터 접근 방식을 정하지만 공개 접근이나 IAM 권한을 정하지 않는다. 온라인 tier의 객체도 요청 주체에 읽기 권한이 있어야 한다. 또한 예상 접근 빈도뿐 아니라 실제 보관기간, 검색 데이터량, 복원 대기를 허용할 수 있는 업무 기한을 함께 판단해야 한다.

## Oracle 공식 참고자료

- [Object Storage Storage Tiers — Standard, Infrequent Access, Archive, Auto-Tiering](https://docs.oracle.com/en-us/iaas/Content/Object/Concepts/understandingstoragetiers.htm)

## 보존 실습으로 이동

[기존 전체 실습 초안](../../Hands-on/Original-Provisional/06_Object_Storage/03_Storage_Tiers.md)에서 저장 계층 비교의 기존 실습 절차를 읽을 수 있다. 보존된 초안으로, OCI 실행 검증은 완료되지 않았다.

