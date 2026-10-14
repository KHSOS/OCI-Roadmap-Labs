> **기존 실습 초안 — 미검증**: 원래 93개 문서의 절차를 보존한 자료입니다. 새 튜토리얼의 품질 승인이나 OCI 실행 검증을 뜻하지 않습니다.

# Buckets / Objects

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../../../Docs/README.md)를 확인하세요.

> 절차 작성본 / OCI 미실행 — 아래 예상값은 실측 결과가 아닙니다. 비용·권한·사전준비 gate를 통과한 전용 환경에서만 실행하세요.

## 학습목표

- 같은 bucket의 payload를 version-two로 덮어쓰고 second key를 업로드하여 두 changed-state GET/cmp를 수행한다.
- 미존재 key 서비스 오류를 정상 second GET과 대조한 뒤 original payload를 복원하고 second만 삭제한다.

완료 기준은 아래 실제 생성·설정·요청 결과로 판정한다. 이 목표는 수행할 작업이며 OCI에서 관찰한 결과가 아니다.

## 대응 Roadmap 항목

- 항목명: **Buckets / Objects**
- Roadmap 경로: **Object Storage → Buckets / Objects**
- Node ID: `sequence-6-1`
- 상위 항목: **Object Storage** (`sequence-6-0`)
- 문서 범위: 이 항목의 개념과 작성된 실습절차. 통합 품질검토는 BLOCKED이며 실제 실행은 NOT_EXECUTED입니다.
- [전체 항목 목록](../../../Docs/README.md)

## 사전준비


48의 REGION/COMPARTMENT_OCID/NS/BUCKET·actual bucket id, private Standard/versioning disabled/auto-tiering disabled와 원래 payload bytes/hash를 받는다. owner48 단일 BUCKET_EXPIRY_UTC와 overwrite/복구 담당자가 확보되어야 한다. 새 bucket/VM은 없으며 second key 삭제·payload 원복을48에 반환한다.

- [Object Storage](00_Object_Storage.md#실습절차) — 링크의 `실습절차`와 `결과검증` 완료; 실제 OCID/IP를 registry에 인계한다.



- 변수 handoff: 48의 `REGION`, `COMPARTMENT_OCID`, `NS`, `BUCKET`; bucket private/Standard/versioning disabled/auto-tiering disabled 확인. 후속에서 새 bucket 생성 금지.

## 권한

GetBucket → BUCKET_READ; ListObjects → OBJECT_INSPECT; GetObject/HeadObject → OBJECT_READ; 새 객체 PutObject → OBJECT_CREATE, 동일 key overwrite는 OBJECT_OVERWRITE 추가; DeleteObject → OBJECT_DELETE.

[API operation별 공식 permission table](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm) — 가족 단위 manage를 최소권한이라 부르지 않는다. 전용 lab compartment의 group만 적용하고 정책작성 권한은 관리자 소유다.
```text
Allow group <identity-domain>/<lab-group> to manage object-family in compartment <lab-compartment> where any {request.permission='BUCKET_INSPECT', request.permission='BUCKET_READ', request.permission='OBJECT_INSPECT', request.permission='OBJECT_READ', request.permission='OBJECT_CREATE', request.permission='OBJECT_DELETE', request.permission='OBJECT_OVERWRITE'}
```
이 allowlist는 위 작업만 허용한다. Console 탐색 List API가 거부되면 실제 API permission을 공식 표에서 별도로 승인하며 all-resources로 확대하지 않는다.

## 비용및주의사항

owner48의 private Standard bucket 하나를 재사용한다. 추가 범위는 `lab/second.txt`와 기존 payload의 승인 overwrite/원복이며 새 bucket/VM·IA/Archive 전환은 없다. 작은 파일 저장·PUT/GET/LIST/DELETE·다운로드 비용을 승인하고 original bytes를 먼저 보존한다. Versioning disabled이므로 원래 payload 복원 성공이 반환 조건이다. lease 만료 전 second만 삭제하고 bucket 삭제는 owner48에게 맡긴다.

## 실습구성

- 작업 목표: 48의 bucket에 서로 다른 객체 key를 업로드하고 덮어쓰기/다운로드를 검증한다.
- 생성 owner: bucket=48, 이번 object/변경=49. 실제 결과 필드는 실행 전 비워 둔다.


### 이번 workshop 시나리오와 인수·반환

같은 bucket에서 key overwrite와 prefix를 실험하고 원래 payload를 돌려준다.

- **신규/변경:** lab/second.txt,lab/payload.txt overwrite/원복.
- **선행 재사용:** 48 private bucket/payload.
- **완주 흐름:** 승인/입력→본문의 생성/편집→version-two→missing key 오류+같은인증 positive→payload original cmp0→second만삭제→payload 원복→48 반환.
- **선택 이유/실패 경계:** ETag는 checksum 단정 금지; OBJECT_OVERWRITE 별도 승인.

| 입력/산출물 | 실제 값 출처 | 준비 시점 |
|---|---|---|
| `NS` | creator의 실제 생성/조회 출력; 임의 OCID/IP 금지 | 사용 전 |
| `BUCKET` | creator의 실제 생성/조회 출력; 임의 OCID/IP 금지 | 사용 전 |
| `lab-object.txt` | 본문 단계에서 승인/선택한 값 | 사용 전 |

입력별 실행위치는 본문 터미널 구분을 따른다: OCI CLI=인증 Cloud Shell/승인 OCI 터미널, OS 앱/NFS/부하=지정 guest, private HTTP/TCP=vm-a guest, YAML 작성/SSH private key=본인 local. 조회/변경/정리의 관리자 승인은 **실행 조건**이며 이 문서는 IAM 성공을 입증하지 않는다. 실제 허용/거부와 capacity는 사용자 실행에서 확인한다. 미승인/NOT_SELECTED/ASYNC_PENDING을 성공으로 바꾸지 않는다.

### 공유 bucket lease와 비동기 분기
owner48의 동일 bucket에는 단일 승인 `BUCKET_EXPIRY_UTC`/cleanup owner가 있다. 50의 하루 이상 관찰 또는52의 restore/download가 원래60분을 넘을 가능성이 있으면 **규칙/업로드 이전에** owner48과 budget 승인자가 새 종료시각/비용/담당자와 모든 선택 consumer를 기록하여 lease를 갱신한다. 갱신 승인 없으면 원래deadline 전에 disable/cancel 가능한 시험을중단하고 owned key를삭제/반환한다; 실제비동기 작업취소가 지원되지않으면작업상태를기록하고 객체삭제/목록readback을owner48과조율한다. owner48은 expiry때 독자삭제하지않고 모든consumer의abort/restore/return acknowledgement를받는다. 연장안됨을단순히무기한보존으로대체하지않는다. ASYNC_PENDING은검증성공아니다.

## 실습절차

### 실행 터미널과 지역 고정

전용 Bash에서 `set -euo pipefail`을 먼저 사용한다. PUT/update/HEAD/GET/cmp의 예기치 않은 실패는 다음 checkpoint로 진행하지 않는다. 모든 다운로드는 존재하지 않는 전용 filename을 확인한 뒤 실행한다. 의도적 negative는 아래 if 또는 `set +e`의 captured exit로만 처리하고 복구는별도owner경로로수행한다.
아래 OCI 명령과 local bytes 작성/비교는 같은 인증 Cloud Shell/승인 OCI 터미널의 전용 directory에서 실행한다. VM·CIDR·앱 포트는 사용하지 않는다.
```bash
export REGION="<approved-region>"
export OCI_CLI_REGION="$REGION"
export LAB_COMPARTMENT_ID="<actual-lab-compartment-ocid>"
export EXPIRY_UTC="<approved-session-end-UTC>"
```
치환 전 실행 금지. 생성된 OCID/IP/원래 설정/새 rule ID/owner/종료시각을 비밀 없는 registry에 기록하고 다음 터미널에서는 필요한 변수만 다시 설정한다.

[아래생성/변경단계의정확한Oracle task본문](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/managingobjects_topic-To_upload_objects_to_a_bucket.htm) — Consolefield/CLI/API동작근거.

아래 단계는 Oracle task 본문을 근거로 작성한 lab recipe이며 공식 실행결과가 아니다. 각 단계의 근거는 끝의 정확한 task 링크를 따른다.

1. [Object Storage](../../../Docs/06_Object_Storage/00_Object_Storage.md)의 bucket 생성과 cmp 성공 단계까지 완료한다. 동일 NS/BUCKET을 사용한다.
2. 먼저 owner49 전용 새 디렉터리를 만든다. owner48의 local source를 변경하지 않는다. overwrite 이전 owner48은 원래 payload의 실제 bytes/hash를 보관하고 독립 인증 Console 복구담당자·lease 이내 복구 deadline을 지정한다. 오류/세션 손실이면 그 담당자가 같은 key만 원래 bytes로 복원하고 새 GET/cmp를 확인한다. 원래 bytes가 본문 `oci-roadmap object payload` LF와 다르면 아래 synthetic 원복을 쓰지 말고 owner48 실제 원본으로 교체·승인하기 전 mutation을 중단한다.
```bash
set -euo pipefail
OWNER49_PARENT=$PWD
OWNER49_DIR=$(mktemp -d "$PWD/oci-lab49.XXXXXXXX") || exit 1
cd "$OWNER49_DIR"
```
`printf 'version-two\n' > lab-object.txt` 후 `oci os object put -ns "$NS" -bn "$BUCKET" --name lab/payload.txt --file lab-object.txt`를 실행한다. 덮어쓰기 확인에는 이 전용 key만 승인한다.
덮어쓰기 직후 원복 전에 `oci os object get -ns "$NS" -bn "$BUCKET" --name lab/payload.txt --file payload-version-two-read.txt`와 `cmp lab-object.txt payload-version-two-read.txt`로 changed payload를 실제 검증한다. 전용 다운로드 파일이 기존에 있으면 중단/새 이름 승인한다.

3. `oci os object put -ns "$NS" -bn "$BUCKET" --name lab/second.txt --file lab-object.txt` 후 `oci os object list -ns "$NS" -bn "$BUCKET" --prefix lab/`를 실행한다. `/`는 실제 디렉터리가 아닌 key prefix이다.
4. `oci os object get -ns "$NS" -bn "$BUCKET" --name lab/second.txt --file second-read.txt; cmp lab-object.txt second-read.txt`로 다운로드한다. 누락 key `lab/not-created.txt` GET은 ObjectNotFound이어야 하고 같은 인증/endpoint로 second.txt GET은 성공해야 한다. Versioning은 비용 승인 없이는 켜지 않는다.

### 실패 양성대조와 원복
```bash
test ! -e absent-read.txt && test ! -e absent-service-error.txt || exit 1
if oci os object get -ns "$NS" -bn "$BUCKET" --name lab/not-created.txt --file absent-read.txt 2> absent-service-error.txt; then
  printf 'ERROR: unexpected object exists\n'; exit 1
fi
grep -F 'ObjectNotFound' absent-service-error.txt || { printf 'not the expected missing-key service error\n'; exit 1; }
oci os object get -ns "$NS" -bn "$BUCKET" --name lab/second.txt --file second-control.txt
cmp lab-object.txt second-control.txt
printf 'oci-roadmap object payload\n' > owner48-original.txt
oci os object put -ns "$NS" -bn "$BUCKET" --name lab/payload.txt --file owner48-original.txt
oci os object get -ns "$NS" -bn "$BUCKET" --name lab/payload.txt --file owner48-restored.txt
cmp owner48-original.txt owner48-restored.txt
```
원복 overwrite는 전용 lab key만 허용한다.

## 결과검증

변경 checkpoint에서는 payload/second 모두 version-two를 확인한다. 원복 후 payload는 owner48 original bytes, second만 version-two이다. 정리후 second가 없어야 한다. ETag를 파일 checksum이라고 단정하지 않는다.

- 예상 pattern이며 실제 OCI 실행 결과가 아니다. timestamp, resource OCID, 설정 readback, 실제 HTTP/TCP/파일 결과를 실행자가 기록한다.

## 문제해결

overwriteprompt가예상기존labkey인지확인;객체versioningdisabled일때payload원복은원본bytes재upload이며이전version자동복구아님.

실패기록에는같은target/credentials/region/요청시간을고정한다. 복구는원래설정으로실행하고actualresponse/readback둘다확인한다;예상값을실측으로쓰지않는다.

## 리소스정리

모든 remote 원복/readback 완료 후 위 새 디렉터리에서만 local cleanup한다:
```bash
test "$PWD" = "$OWNER49_DIR" || exit 1
rm -f -- lab-object.txt payload-version-two-read.txt second-read.txt absent-read.txt absent-service-error.txt second-control.txt owner48-original.txt owner48-restored.txt
cd "$OWNER49_PARENT"
rmdir -- "$OWNER49_DIR"
```
알 수 없는 파일이 남으면 rmdir 실패로 중단하고 조사한다. owner48 원본 디렉터리는 정리하지 않는다.

`oci os object delete -ns "$NS" -bn "$BUCKET" --name lab/second.txt`; payload.txt는위original복원후48에보존인계. list --prefix lab/로second없음확인.

정리 UTC/namespace/bucket/owned key·목록 readback과 owner48 반환을 기록한다. shared bucket은 consumer가 삭제하지 않는다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 목표는 학습 후 설명·구분·확인할 내용이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [35] [Overview of Object Storage — Buckets / Objects](https://docs.oracle.com/en-us/iaas/Content/Object/Concepts/objectstorageoverview.htm)
  - 확인 범위: Buckets Buckets are logical / Objects Any type of data.

### 절차별 공식 task body 추가 근거
- [objectstorageoverview.htm](https://docs.oracle.com/en-us/iaas/Content/Object/Concepts/objectstorageoverview.htm) — 본문 검토 근거; 아래 정확한 task와 작성한 입력을 구분한다. OCI 실행 증거가 아니다.
- [managingobjects.htm](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/managingobjects.htm) — 본문 검토 근거; 아래 정확한 task와 작성한 입력을 구분한다. OCI 실행 증거가 아니다.



### 이번 절차의 정확한 공식 task
- [managingobjects_topic-To_upload_objects_to_a_bucket.htm](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/managingobjects_topic-To_upload_objects_to_a_bucket.htm) — 이 문서 실습절차의 입력/동작을 대조한 본문.
- [managingobjects_topic-To_download_an_object_from_a_bucket.htm](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/managingobjects_topic-To_download_an_object_from_a_bucket.htm) — 이 문서 실습절차의 입력/동작을 대조한 본문.
