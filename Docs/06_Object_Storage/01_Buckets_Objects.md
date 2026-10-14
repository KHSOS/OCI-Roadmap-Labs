# Buckets / Objects

Bucket은 Object들을 담는 **논리적 컨테이너**이고, Object는 데이터와 Metadata를 담는 저장 단위다. Bucket은 한 Region과 한 Compartment에 연결되며 Object는 그 Bucket 안에 이름으로 식별된다. Bucket 이름, Bucket OCID와 Object 이름은 같은 식별자가 아니다.

## 컨테이너와 데이터의 구분

Bucket은 저장 Tier의 기본값, 접근 공개 여부, 암호화·Versioning 같은 관리 선택이 관계되는 단위다. Object는 이미지·문서·로그처럼 실제 저장하는 데이터다. Object에는 OCID가 없으며 Namespace, Bucket과 Object 이름을 조합해 대상을 구분한다.

예를 들어 같은 Bucket의 reports/january.csv와 reports/february.csv는 서로 다른 Object다. reports/는 Object 이름의 Prefix이지 별도로 생성된 파일시스템 디렉터리가 아니다. Console의 폴더 표시는 이름에 포함된 슬래시를 계층처럼 보여주는 기능이다.

## 이름과 Bucket Scope

Object 이름과 Bucket 이름은 대소문자를 구분한다. Object 이름에는 민감한 정보를 넣지 않는 것이 좋으며, 같은 Bucket의 기존 이름을 사용하면 새 데이터가 기존 Object를 교체하는 상황이 될 수 있다. 단, IAM Condition의 문자열 매칭은 대소문자를 구분하지 않으므로 이름의 구분과 권한 조건의 구분을 혼동하지 않는다.

|Bucket Scope|이름의 고유 범위와 주소 방식|
|---|---|
|NAMESPACE|Tenancy와 Region 안에서 Bucket 이름이 고유하며 Path-style 접근을 지원한다.|
|REGION|Region의 모든 Tenancy에 걸쳐 이름이 고유해야 하며 Path-style과 Virtual-hosted 방식 모두를 지원한다.|

REGION Scope는 Virtual-hosted 방식이 필요할 때 관계되지만 더 넓은 이름 충돌 범위를 고려해야 한다. NAMESPACE에서 REGION으로는 변경할 수 있어도 REGION에서 NAMESPACE로 되돌릴 수는 없다. Scope는 Bucket 공개 여부나 Compartment 권한 범위를 뜻하는 값이 아니다.

## 데이터 변경과 Versioning

Object는 하나의 단위로 처리하므로 기존 Object 내용을 부분 편집하거나 끝에 추가하는 모델이 아니다. 새 내용을 저장하면 전체 Object를 교체한다. 따라서 같은 이름을 사용하는 변경과 새로운 이름의 Object 추가는 다른 작업이다.

Versioning은 Object 변경이나 삭제 때 별도의 Version을 보존하는 기능이다. 우발적 덮어쓰기·삭제에 대한 보호에 관계되지만 Version 저장량과 수명 관리도 함께 생각해야 한다. Versioning을 쓰지 않는 경우 이전 내용을 자동으로 선택해 복구할 수 있다고 가정하지 않는다.

## 읽기와 관리 선택

목록 조회와 실제 데이터 다운로드는 다른 권한이다. Object의 이름을 알고 API로 접근하는 방식과 Console에서 Bucket·Object 목록을 탐색하는 방식은 필요한 탐색 권한도 다를 수 있다. 알려진 이름을 읽을 수 있다는 것이 모든 Object를 나열하거나 변경할 수 있다는 뜻은 아니다.

Standard Bucket은 즉시 사용하는 자료에 적합하고, Archive 자료는 읽기 전에 Restore가 관계된다. Prefix는 데이터 집합을 구분하거나 Lifecycle·권한 조건의 대상을 좁힐 때 유용하지만 실제 파일시스템 디렉터리의 권한 모델과 같지는 않다. Multipart Upload가 실패한 뒤 남은 미완료 Part는 자동으로 사라진다고 가정할 수 없으며 저장 과금과 Lifecycle 관리 대상이 된다.

## 함께 읽기

[Object Storage](00_Object_Storage.md)는 서비스 전체 역할을, [Object Lifecycle Management](02_Object_Lifecycle_Management.md)는 Object·Version·미완료 업로드의 자동 수명 관리를 설명한다. [보존된 Buckets / Objects 실습](../../Hands-on/Original-Provisional/06_Object_Storage/01_Buckets_Objects.md)은 기존 이름·덮어쓰기 비교 절차를 담은 OCI 실행 미검증 초안이다.

## 공식 참고 문서

- [Overview of Object Storage — Buckets, Objects and Namespace](https://docs.oracle.com/en-us/iaas/Content/Object/Concepts/objectstorageoverview.htm)
- [Object Storage Objects — Names, Prefixes, Replacement and Versioning](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/managingobjects.htm)
- [Creating a Bucket — Bucket Scope and Storage Options](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/managingbuckets_topic-To_create_a_bucket.htm)
- [Policy Syntax — Case-insensitive Conditions](https://docs.oracle.com/en-us/iaas/Content/Identity/Concepts/policysyntax.htm)
