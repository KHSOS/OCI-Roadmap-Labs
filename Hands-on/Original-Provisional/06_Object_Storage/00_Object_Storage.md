> **기존 실습 초안 — 미검증**: 원래 93개 문서의 절차를 보존한 자료입니다. 새 튜토리얼의 품질 승인이나 OCI 실행 검증을 뜻하지 않습니다.

# Object Storage

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../../../Docs/README.md)를 확인하세요.

> 절차 작성본 / OCI 미실행 — 아래 예상값은 실측 결과가 아닙니다. 비용·권한·사전준비 gate를 통과한 전용 환경에서만 실행하세요.

## 학습목표

- private Standard Bucket 하나와 lab/payload.txt를 생성하고 인증 GET/cmp 및 bucket 설정을 readback한다.
- 49–54의 단일 lease/반환을 관리하고 모든 owned key가 정리된 빈 bucket만 삭제한다.

완료 기준은 아래 실제 생성·설정·요청 결과로 판정한다. 이 목표는 수행할 작업이며 OCI에서 관찰한 결과가 아니다.

## 대응 Roadmap 항목

- 항목명: **Object Storage**
- Roadmap 경로: **Object Storage**
- Node ID: `sequence-6-0`
- 상위 항목: 없음 — 최상위 학습 항목
- 문서 범위: 이 항목의 개념과 작성된 실습절차. 통합 품질검토는 BLOCKED이며 실제 실행은 NOT_EXECUTED입니다.
- [전체 항목 목록](../../../Docs/README.md)

## 사전준비


00의 실제 ACTIVE LAB_COMPARTMENT_ID와 승인 REGION/CLI 인증을 받는다. NS는 아래 GetNamespace 실제 응답, BUCKET은 생성 전에 namespace 내 충돌 검사한 고유 이름이다. CREATED_BUCKET_ID·private/Standard/Versioning/Auto-tiering disabled readback과 payload source/GET/cmp를49–54에 반환한다. 앱/VM/CIDR 입력은 없다.

- [Introduction](../00_Introduction/00_Introduction.md#실습절차) — 링크의 `실습절차`와 `결과검증` 완료; 실제 OCID/IP를 registry에 인계한다.



- 변수 handoff: 48의 `REGION`, `COMPARTMENT_OCID`, `NS`, `BUCKET`; bucket private/Standard/versioning disabled/auto-tiering disabled 확인. 후속에서 새 bucket 생성 금지.

## 권한

GetBucket → BUCKET_READ; ListObjects → OBJECT_INSPECT; GetObject/HeadObject → OBJECT_READ; 새 객체 PutObject → OBJECT_CREATE, 동일 key overwrite는 OBJECT_OVERWRITE 추가; DeleteObject → OBJECT_DELETE. CreateBucket/DeleteBucket → BUCKET_CREATE/BUCKET_DELETE. GetNamespace(자기 namespace, compartmentId 생략)는 추가 permission 없음.

[API operation별 공식 permission table](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm) — 가족 단위 manage를 최소권한이라 부르지 않는다. 전용 lab compartment의 group만 적용하고 정책작성 권한은 관리자 소유다.
```text
Allow group <identity-domain>/<lab-group> to manage object-family in compartment <lab-compartment> where any {request.permission='BUCKET_INSPECT', request.permission='BUCKET_READ', request.permission='OBJECT_INSPECT', request.permission='OBJECT_READ', request.permission='OBJECT_CREATE', request.permission='OBJECT_DELETE', request.permission='BUCKET_CREATE', request.permission='BUCKET_DELETE'}
```
이 allowlist는 위 작업만 허용한다. Console 탐색 List API가 거부되면 실제 API permission을 공식 표에서 별도로 승인하며 all-resources로 확대하지 않는다.

## 비용및주의사항

새 private Standard Bucket 하나와 `lab/payload.txt`만 만든다. 저장 bytes·PUT/GET/LIST/DELETE 요청과 실제 전송 비용을 승인하고 Versioning/Auto-tiering/Retention/Replication/PAR는 켜지 않는다.49–54는 이 bucket을 재사용한다. IA/Archive를 선택하는 consumer만 해당 최소 저장기간·검색/조기삭제 비용을 별도 승인한다.50 하루 관찰/52 restore가 기본60분을 넘으면 규칙/업로드 전에 단일 BUCKET_EXPIRY_UTC를 갱신해야 한다. 미승인 비동기 연장은 중단/owned-key 정리와 반환으로 처리한다.

## 실습구성

- 작업 목표: 전용 private bucket 1개와 실제 작은 객체를 생성한다.
- 생성 owner: 48. 동일 owner의 자원을 재사용하며 실제 결과 필드는 실행 전 비워 둔다.


### 이번 workshop 시나리오와 인수·반환

private bucket 하나를 만들어 작은 payload를 넣고 인증된 다운로드를 끝낸다.

- **신규/변경:** private Standard bucket1개,lab/payload.txt.
- **선행 재사용:** 0 lab-root/기존 namespace.
- **완주 흐름:** 승인/입력→본문의 생성/편집→GET/cmp0와 private/Standard/disabled 설정 readback→49–54 반환→payload 포함 keys 삭제→빈 bucket 삭제.
- **선택 이유/실패 경계:** namespace/global 이름과 Region endpoint를 구분하고 기존 파일/객체 충돌은 중단한다.

| 입력/산출물 | 실제 값 출처 | 준비 시점 |
|---|---|---|
| `NS` | creator의 실제 생성/조회 출력; 임의 OCID/IP 금지 | 사용 전 |
| `BUCKET` | 생성 전 승인/namespace 충돌 검사한 고유 bucket name; OCID와 다름 | 사용 전 |
| `COMPARTMENT_OCID` | creator의 실제 생성/조회 출력; 임의 OCID/IP 금지 | 사용 전 |
| `REGION` | 본문 단계에서 승인/선택한 값 | 사용 전 |

입력별 실행위치는 본문 터미널 구분을 따른다: OCI CLI=인증 Cloud Shell/승인 OCI 터미널, OS 앱/NFS/부하=지정 guest, private HTTP/TCP=vm-a guest, YAML 작성/SSH private key=본인 local. 조회/변경/정리의 관리자 승인은 **실행 조건**이며 이 문서는 IAM 성공을 입증하지 않는다. 실제 허용/거부와 capacity는 사용자 실행에서 확인한다. 미승인/NOT_SELECTED/ASYNC_PENDING을 성공으로 바꾸지 않는다.

### 공유 bucket lease와 비동기 분기
owner48의 동일 bucket에는 단일 승인 `BUCKET_EXPIRY_UTC`/cleanup owner가 있다. 50의 하루 이상 관찰 또는52의 restore/download가 원래60분을 넘을 가능성이 있으면 **규칙/업로드 이전에** owner48과 budget 승인자가 새 종료시각/비용/담당자와 모든 선택 consumer를 기록하여 lease를 갱신한다. 갱신 승인 없으면 원래deadline 전에 disable/cancel 가능한 시험을중단하고 owned key를삭제/반환한다; 실제비동기 작업취소가 지원되지않으면작업상태를기록하고 객체삭제/목록readback을owner48과조율한다. owner48은 expiry때 독자삭제하지않고 모든consumer의abort/restore/return acknowledgement를받는다. 연장안됨을단순히무기한보존으로대체하지않는다. ASYNC_PENDING은검증성공아니다.

## 실습절차

### 실행 터미널과 지역 고정
아래 OCI 명령과 local bytes 작성/비교는 같은 인증 Cloud Shell/승인 OCI 터미널의 전용 directory에서 실행한다. VM·CIDR·앱 포트는 사용하지 않는다.
```bash
export REGION="<approved-region>"
export OCI_CLI_REGION="$REGION"
export LAB_COMPARTMENT_ID="<actual-lab-compartment-ocid>"
export EXPIRY_UTC="<approved-session-end-UTC>"
```
치환 전 실행 금지. 생성된 OCID/IP/원래 설정/새 rule ID/owner/종료시각을 비밀 없는 registry에 기록하고 다음 터미널에서는 필요한 변수만 다시 설정한다.

[아래생성/변경단계의정확한Oracle task본문](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/managingbuckets_topic-To_create_a_bucket.htm) — Consolefield/CLI/API동작근거.

아래 단계는 Oracle task 본문을 근거로 작성한 lab recipe이며 공식 실행결과가 아니다. 각 단계의 근거는 끝의 정확한 task 링크를 따른다.

1. Cloud Shell/승인 OCI 터미널에서 **먼저 전용 빈 작업 디렉터리**를 선택한다. `test ! -e lab-object.txt && test ! -e downloaded.txt || exit 1`로 충돌을 확인하고 기존 파일을 덮어쓰지 않는다. OCI CLI 인증과 Region을 확인하고 아래 입력값을 설정한다. OCID는 승인된 Compartment의 실제 값이며 예시 문자열로 실행하지 않는다.
```bash
set -euo pipefail
LAB48_WORK=$(mktemp -d "$HOME/oci-roadmap-48.XXXXXXXX") || exit 1
cd -- "$LAB48_WORK" || exit 1
# record actual LAB48_WORK/owner before creating local bytes
export COMPARTMENT_OCID="$LAB_COMPARTMENT_ID"
# dedicated empty local directory; stop on CreateBucket conflict rather than uploading into an existing bucket
export BUCKET='<unique-lab-bucket-name>'
export NS="$(oci os ns get --query data --raw-output)"
test ! -e lab-object.txt && test ! -L lab-object.txt && test ! -e downloaded.txt && test ! -L downloaded.txt || exit 1
printf 'oci-roadmap object payload\n' > lab-object.txt
if ! CREATED_BUCKET_ID=$(oci os bucket create --namespace-name "$NS" --compartment-id "$COMPARTMENT_OCID" --name "$BUCKET" --public-access-type NoPublicAccess --storage-tier Standard --versioning Disabled --auto-tiering Disabled --object-events-enabled false --query data.id --raw-output); then
  printf 'STOP: CreateBucket failed; no PUT or reuse permitted\n'; exit 1
fi
test -n "$CREATED_BUCKET_ID" && test "$CREATED_BUCKET_ID" != null || exit 1
READBACK_BUCKET_ID=$(oci os bucket get -ns "$NS" -bn "$BUCKET" --query data.id --raw-output) || exit 1
test "$READBACK_BUCKET_ID" = "$CREATED_BUCKET_ID" || { printf 'STOP: bucket identity changed\n'; exit 1; }
oci os object put --namespace-name "$NS" --bucket-name "$BUCKET" --name lab/payload.txt --file lab-object.txt
oci os object get --namespace-name "$NS" --bucket-name "$BUCKET" --name lab/payload.txt --file downloaded.txt
cmp lab-object.txt downloaded.txt
oci os bucket get -ns "$NS" -bn "$BUCKET"
oci os object head -ns "$NS" -bn "$BUCKET" --name lab/payload.txt
```
2. Console의 Create bucket 대안 입력은 name=`$BUCKET`, compartment=승인 OCID, default storage tier=Standard, visibility=Private, encryption=Oracle-managed, auto-tiering/Versioning/Events=disabled이다. 생성은 CLI 또는 Console 중 한 번만 한다. Objects → Upload에서 prefix `lab/`, file lab-object.txt, tier Standard를 지정한다. Console에서 bucket OCID, private visibility, 객체 크기와 tier를 readback한다. Pre-authenticated request는 생성하지 않는다.

### 실행 디렉터리와 지역 고정
모든 OCI 명령은 아래 REGION 설정을 먼저 사용한다. 기존 local download 파일을 덮어쓰지 않도록 전용 빈 디렉터리에서 수행한다. 같은 lab key의 overwrite prompt는 예상 밖 기존객체면 취소한다.
```bash
export REGION='<approved-region-identifier>'
export OCI_CLI_REGION="$REGION"
oci os bucket get -ns "$NS" -bn "$BUCKET"
oci os object list -ns "$NS" -bn "$BUCKET" --prefix lab/
```
`namespace/bucket/key`는 API 정확한 target이며 49–54에 인계한다. upload/GET/cmp와 bucket readback 모두 필요하다.

## 결과검증

cmp 종료 코드 0과 Console 객체 존재를 기대한다. Namespace는 tenancy 값이며 Compartment 이름과 다르다.

- 예상 pattern이며 실제 OCI 실행 결과가 아니다. timestamp, resource OCID, 설정 readback, 실제 HTTP/TCP/파일 결과를 실행자가 기록한다.

## 문제해결

namespace/bucket-region오류는OCI_CLI_REGION/NS/BUCKET세값을같은target으로고정. localfileexists는전용새다운로드filename사용;privatemode를public으로수정하지않는다.

실패기록에는같은target/credentials/region/요청시간을고정한다. 복구는원래설정으로실행하고actualresponse/readback둘다확인한다;예상값을실측으로쓰지않는다.

## 리소스정리

remote 반환/정리가 끝나고 증거 보존 승인이 종료되면 registry의 LAB48_WORK 안에 이번 생성한 regular/non-symlink `lab-object.txt`/`downloaded.txt`(실패 partial 포함)만 exact 경로로 제거한다. 디렉터리를 벗어난 뒤 `rmdir -- "$LAB48_WORK"`로 빈 owned directory만 제거한다.49의 전용 파일과 기존 사용자의 파일은 정리하지 않는다.

위 단일 bucket lease의 선택49–54가 모두 return/abort acknowledgement를 기록한 뒤 owner48만 정리한다. expiry가 먼저면 새 시험을 중단하고 각 owner의 disable/owned-key 삭제/반환을 조율한다. 갱신 lease가 있으면 갱신된 deadline을 사용한다. 기존60분 문장으로 renewed lease를 덮어쓰거나 consumer 확인 없이 삭제하지 않는다. 그 다음 owner48의 `lab/payload.txt`도 `oci os object delete -ns "$NS" -bn "$BUCKET" --name lab/payload.txt`로 삭제한다. 이어 `oci os object list -ns "$NS" -bn "$BUCKET" --all`로빈bucket확인. 남은실습key를각owner가삭제하고 `oci os bucket delete -ns "$NS" -bn "$BUCKET"`를확인prompt후실행. `oci os bucket get`의BucketNotFound와동일namespace정상조회로삭제검증.

정리 UTC/namespace/bucket/owned key·목록 readback과 owner48 반환을 기록한다. shared bucket은 consumer가 삭제하지 않는다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 목표는 학습 후 설명·구분·확인할 내용이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [35] [Overview of Object Storage — Object Storage](https://docs.oracle.com/en-us/iaas/Content/Object/Concepts/objectstorageoverview.htm)
  - 확인 범위: The Object Storage service is.

### 절차별 공식 task body 추가 근거
- [objectstorageoverview.htm](https://docs.oracle.com/en-us/iaas/Content/Object/Concepts/objectstorageoverview.htm) — 본문 검토 근거; 아래 정확한 task와 작성한 입력을 구분한다. OCI 실행 증거가 아니다.
- [managingbuckets.htm](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/managingbuckets.htm) — 본문 검토 근거; 아래 정확한 task와 작성한 입력을 구분한다. OCI 실행 증거가 아니다.
- [managingobjects.htm](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/managingobjects.htm) — 본문 검토 근거; 아래 정확한 task와 작성한 입력을 구분한다. OCI 실행 증거가 아니다.



### 이번 절차의 정확한 공식 task
- [managingbuckets_topic-To_create_a_bucket.htm](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/managingbuckets_topic-To_create_a_bucket.htm) — 이 문서 실습절차의 입력/동작을 대조한 본문.
- [managingobjects_topic-To_upload_objects_to_a_bucket.htm](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/managingobjects_topic-To_upload_objects_to_a_bucket.htm) — 이 문서 실습절차의 입력/동작을 대조한 본문.
