# Object Storage

OCI Object Storage는 이미지·영상·로그·백업처럼 다양한 비정형 데이터를 **Object 단위로 보관하는 지역 서비스**다. 특정 Compute Instance의 디스크가 아니라 별도의 서비스이며, API나 Console 등의 인터페이스로 데이터를 저장하고 읽는다.

## 어떤 문제에 쓰는가

웹 서비스의 이미지 자료, 분석용 데이터 집합, 장기 보관 파일, 데이터베이스 백업 등 많은 파일을 안정적으로 보관하고 필요할 때 읽는 용도에 적합하다. VM 한 대의 수명과 데이터를 분리하고 여러 애플리케이션이 같은 저장 서비스를 이용할 수 있다. Object를 로컬 파일처럼 중간 부분에 계속 덧붙이는 모델은 아니며 내용 변경은 전체 Object 교체로 이해해야 한다.

## Namespace, Bucket, Object의 관계

|단위|역할|
|---|---|
|Namespace|Tenancy에 할당되는 Object Storage의 최상위 이름 공간|
|Bucket|한 Region과 한 Compartment에 속하는 Object 컨테이너|
|Object|데이터 내용과 그에 대한 Metadata를 담는 저장 단위|
|Compartment|Bucket의 관리·권한 범위를 구분하는 논리적 단위|

Namespace 이름은 Tenancy에 할당되는 고유하고 변경할 수 없는 값이며 Region들에서 같은 문자열을 사용한다. 실제 데이터 대상은 Region, Namespace, Bucket과 Object 이름을 함께 구분해야 한다. 같은 Namespace 문자열이 여러 Region의 데이터를 하나의 Bucket처럼 합친다는 뜻은 아니다.

## 접근과 보안의 관계

서비스 Endpoint에 도달할 수 있는 네트워크 연결과 데이터를 읽거나 변경할 수 있는 권한은 별개다. Private Subnet의 워크로드는 Service Gateway로 OCI 백본을 통해 Object Storage 공용 Endpoint에 연결할 수 있지만, 이 경로 자체가 Bucket 읽기 권한을 주지는 않는다.

Bucket의 Private/Public 접근 설정과 저장 데이터 암호화도 다른 보호 요소다. 암호화되어 있다는 사실만으로 공개 읽기가 제한되지는 않으며, Private Bucket을 선택했다고 모든 IAM User가 데이터를 읽을 수 있는 것도 아니다. IAM Policy는 주체·작업·대상 범위를 결정한다.

## 저장 계층 선택

Standard는 빠르고 즉시 필요한 자료에 적합한 기본 Tier다. Standard Bucket 안에서도 Object별로 Infrequent Access나 Archive를 사용할 수 있다. Archive 자료는 읽기 전에 Restore가 필요하므로 보관 비용뿐 아니라 자료를 다시 사용하는 데 허용되는 지연을 고려해야 한다. Auto-Tiering은 Standard와 Infrequent Access 사이의 자동 이동과 관계되며 Lifecycle Rule은 명시적인 수명·이동·삭제 정책과 관계된다.

## 성질과 한계

Object Storage는 읽기 요청에 가장 최근에 저장된 데이터를 제공하는 Strong Consistency를 설명한다. 이것은 IAM Policy 전파나 Lifecycle 비동기 처리까지 즉시 완료된다는 뜻은 아니다. 서비스가 여러 저장 서버에 데이터를 중복 보관해 내구성을 제공하더라도 실수한 덮어쓰기·삭제를 모두 자동 복구하는 백업 정책으로 대체해서는 안 된다.

과금은 저장량 외에도 API 요청과 관련 기능의 사용에 관계된다. 다른 서비스의 백업이나 Custom Image가 Object Storage를 사용하면 관련 Object Storage 비용이 별도로 발생할 수 있다. Free Tier 적격 여부를 모든 저장·요청이 무제한 무료라는 뜻으로 읽지 않는다. 개별 Object 크기, 업로드 방식, 요청 처리량과 Quota도 설계의 제한 요소다.

## 함께 읽기

[Buckets / Objects](01_Buckets_Objects.md), [Object Lifecycle Management](02_Object_Lifecycle_Management.md), [Storage Tiers](03_Storage_Tiers.md)는 저장 단위·자동 관리·접근 패턴의 선택을 각각 설명한다. [보존된 Object Storage 실습](../../Hands-on/Original-Provisional/06_Object_Storage/00_Object_Storage.md)은 기존 생성·업로드·다운로드 절차를 담은 OCI 실행 미검증 초안이다.

## 공식 참고 문서

- [Overview of Object Storage — Resources, Consistency, Access and Billing](https://docs.oracle.com/en-us/iaas/Content/Object/Concepts/objectstorageoverview.htm)
- [Object Storage Objects — Object Data and Replacement](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/managingobjects.htm)
- [Creating a Bucket — Storage Tier, Auto-Tiering and Access Options](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/managingbuckets_topic-To_create_a_bucket.htm)
