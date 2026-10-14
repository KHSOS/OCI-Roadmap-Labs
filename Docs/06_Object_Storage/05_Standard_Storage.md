# Standard Storage

Standard는 빠르고 즉각적이며 빈번한 접근이 필요한 Object Storage 데이터의 hot tier다. 자주 읽는 이미지·영상·로그·콘텐츠, 접근 가능한 백업, 분석 엔진이 활용하는 데이터 집합에 적합하다. 객체를 읽기 전에 Archive 복원을 요청할 필요가 없다. [Oracle Standard Tier](https://docs.oracle.com/en-us/iaas/Content/Object/Concepts/understandingstoragetiers.htm)

## 즉시 접근과 접근 권한

Standard의 온라인 가용성은 데이터 접근 방식에 관한 설명이다. Bucket이 공개되어 있다는 뜻이나 모든 요청이 허용된다는 뜻은 아니다. Private bucket의 Standard 객체를 읽는 경우에도 별도의 인증·권한 조건이 적용된다. 따라서 읽기 요청 실패를 Archive 대기 문제로 해석하거나, Standard를 선택했다는 이유로 공개 URL 접근을 기대하면 안 된다.

Standard는 tier별 검색 요금과 최소 저장기간이 없지만 저장·요청·데이터 전송까지 모두 무료인 것은 아니다. 자주 읽는 데이터에서는 IA의 낮은 저장 단가보다 검색 비용이 없는 Standard가 총비용 관점에서 적합할 수 있다. [공식 tier 비교](https://docs.oracle.com/en-us/iaas/Content/Object/Concepts/understandingstoragetiers.htm)

## Standard bucket과 Standard 객체

Standard를 bucket default로 선택하면 별도 지정하지 않은 새 객체는 Standard tier가 된다. 그러나 Standard bucket은 IA와 Archive 객체도 함께 포함할 수 있다. Bucket 이름이나 default만으로 모든 객체가 현재 Standard라고 판단하면 안 된다. 개별 객체의 tier가 실제 읽기·비용 조건을 결정한다.

Bucket default는 생성 후 바꿀 수 없지만 객체 tier는 다른 요구에 맞춰 변경할 수 있다. Archive로 변경하거나 복원해도 같은 bucket 안에 남는다. 이 관계는 [Storage Tiers](03_Storage_Tiers.md)에서 비교한다.

## 접근 패턴이 바뀔 때

계속 자주 읽는 콘텐츠라면 Standard를 유지하는 것이 자연스럽다. 오랫동안 거의 읽지 않는 백업은 [Infrequent Access](06_Infrequent_Access.md)의 최소기간·검색 요금을 비교하고, 복원 대기를 허용하는 장기 자료는 [Archive](04_Archive.md)를 고려한다.

Standard bucket의 Auto-Tiering은 접근 패턴에 따라 Standard와 IA 사이의 이동을 자동으로 관리한다. 단, 1 MiB보다 큰 객체를 관찰하며 이동 자격을 충족해야 한다. 모든 작은 파일을 즉시 저렴한 tier로 바꾸는 기능이 아니고 Archive까지 자동으로 포함하는 기능도 아니다. IA 이동 Lifecycle 규칙과의 제약 및 서비스 권한이 있으므로 단순한 bucket 선택과 구분해야 한다. [Auto-Tiering](https://docs.oracle.com/en-us/iaas/Content/Object/Concepts/understandingstoragetiers.htm)

## 선택과 한계

낮은 접근 지연과 빈번한 읽기가 저장 단가보다 중요할 때 Standard를 우선 고려한다. 하지만 특정 애플리케이션의 전체 응답시간을 tier만으로 보장할 수는 없으며 객체를 파일시스템의 동일한 읽기·쓰기 인터페이스로 취급해서도 안 된다. 이 페이지의 범위는 Object Storage의 hot tier이며 네트워크 연결·IAM·애플리케이션 설계는 별도 요소다.

## Oracle 공식 참고자료

- [Object Storage Storage Tiers — Standard Tier와 Auto-Tiering](https://docs.oracle.com/en-us/iaas/Content/Object/Concepts/understandingstoragetiers.htm)

## 보존 실습으로 이동

[기존 전체 실습 초안](../../Hands-on/Original-Provisional/06_Object_Storage/05_Standard_Storage.md)에서 Standard 객체 접근의 기존 실습 절차를 읽을 수 있다. 보존된 초안으로, OCI 실행 검증은 완료되지 않았다.

