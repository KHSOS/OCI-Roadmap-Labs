# Archive

Archive는 거의 접근하지 않지만 장기간 보존해야 하는 Object Storage 데이터의 cold tier다. 세 tier 중 저장 비용이 가장 낮은 대신 데이터가 offline 상태이며 읽기 전에 복원이 필요하다. 법적 보관 자료, 과거 로그 분석, 오래된 콘텐츠, 향후 분석용 애플리케이션 기록처럼 읽는 시점에 대기를 허용할 수 있는 데이터에 적합하다. [Oracle Archive 설명](https://docs.oracle.com/en-us/iaas/Content/Object/Concepts/understandingstoragetiers.htm)

## 저장 tier와 복원 상태는 다르다

Archive 객체는 보관 중인 객체이고 복원은 그 객체를 일정 기간 읽을 수 있게 하는 작업이다. 공식 문서는 복원 요청부터 첫 byte를 읽을 수 있을 때까지 최대 한 시간이 걸린다고 설명한다. 복원 요청을 받았다는 상태와 실제 다운로드 가능한 상태는 같지 않다. 즉시 응답해야 하는 애플리케이션 원본이나 긴급 복구 기한이 짧은 자료에 Archive를 선택하면 이 대기가 요구사항과 충돌할 수 있다.

복원 완료까지의 대기 시간과 완료 후 데이터를 읽을 수 있는 기간도 별개다. 임시 읽기 가능 상태를 영구적인 Standard 전환으로 해석하면 안 된다. [Storage Tiers의 Archive 조건](https://docs.oracle.com/en-us/iaas/Content/Object/Concepts/understandingstoragetiers.htm)

## Bucket 구조와 관계

Archive 기본값의 bucket에는 Archive tier 객체만 저장된다. 반면 Standard 기본값의 bucket에도 특정 객체를 Archive로 지정할 수 있다. 복원되더라도 객체는 원래 bucket에 남는다. 따라서 Archive를 사용하는 데 별도의 Archive bucket이 항상 필요한 것은 아니다. Bucket 기본 tier는 생성 후 바꿀 수 없으므로 데이터 집합 전체를 cold 보관할지, 일부만 Archive로 둘지 먼저 판단하는 것이 중요하다.

[Storage Tiers](03_Storage_Tiers.md)는 bucket default와 객체 tier를 함께 설명하며, [Standard Storage](05_Standard_Storage.md)와 [Infrequent Access](06_Infrequent_Access.md)는 복원 없이 온라인 읽기가 필요한 대안이다.

## 장기 저장 비용과 삭제의 한계

Archive의 최소 저장기간은 90일이다. 그 전에 삭제·덮어쓰기하여 데이터가 제거되면 전체 최소기간에 대한 일할 비용이 청구될 수 있다. Versioning으로 이전 데이터가 남는 경우와 실제 제거되는 경우는 구분한다. 복원된 객체가 읽기 가능한 tier에 머무는 동안에는 Standard 저장 요금도 적용된다. 낮은 Archive 저장 단가만으로 총비용을 판단하면 이 부분을 빠뜨리게 된다.

공식 tier 비교에서 Archive의 retrieval fee는 없음으로 표시된다. 이것을 복원·요청·전송·임시 Standard 저장까지 모두 무료라는 뜻으로 확대하면 안 된다. 또한 최소 저장기간은 비용 조건이지 삭제를 막는 법적 보존 제어가 아니다. 규제 대응 설계에는 별도의 보존 요구와 제어를 확인해야 한다. [최소기간과 복원 비용](https://docs.oracle.com/en-us/iaas/Content/Object/Concepts/understandingstoragetiers.htm)

## 선택 기준

장기 보관하면서 드물게 읽고, 복원 지연을 업무가 수용할 수 있을 때 Archive를 고려한다. 읽기 시점이 예측되지 않아도 즉시 데이터가 필요하면 IA가 더 적합할 수 있다. 자주 읽는 데이터라면 Standard의 온라인 접근과 검색 요금 구조를 비교한다. Lifecycle을 통한 자동 보관은 Archive 자체와 다른 관리 기능이며 [Object Lifecycle Management](02_Object_Lifecycle_Management.md)에서 다룬다.

## Oracle 공식 참고자료

- [Object Storage Storage Tiers — Archive와 비용·접근 조건](https://docs.oracle.com/en-us/iaas/Content/Object/Concepts/understandingstoragetiers.htm)

## 보존 실습으로 이동

[기존 전체 실습 초안](../../Hands-on/Original-Provisional/06_Object_Storage/04_Archive.md)에서 Archive 보관과 복원의 기존 실습 절차를 읽을 수 있다. 보존된 초안으로, OCI 실행 검증은 완료되지 않았다.

