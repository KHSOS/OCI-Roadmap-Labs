> **문서 절차 검토 완료 / OCI 실행 미검증**: 이 문서의 private Standard Bucket 생성·업로드·다운로드·정리 시나리오에 한정한 문서 검토 상태입니다. 실제 OCI 실행 검증이나 전체 교육과정의 승인을 뜻하지 않습니다.

# Object Storage

> **한정된 문서 검토 범위**: 아래 두 실행 경로와 선택 후속 보존·반환 및 정리 절차를 문서 수준에서 검토했습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 권한·비용·안전·보존 조건은 실행 전 충족해야 하며, 전체 교육과정 및 게시 상태는 별도입니다. [전체 항목 목록](../../../Docs/README.md)

> 절차 작성본 / OCI 미실행 — 아래 예상값은 실측 결과가 아닙니다. 비용·권한·사전준비 gate를 통과한 전용 환경에서만 실행하세요.

## 학습목표

- private Standard Bucket 하나와 lab/payload.txt를 생성하고 인증 GET/cmp 및 bucket 설정을 readback한다.
- 다운로드 내용 확인 후 owned key와 빈 bucket을 삭제한다. 후속 실습을 선택할 때만 아래 공통 보존 계약을 사용한다.

완료 기준은 아래 실제 생성·설정·요청 결과로 판정한다. 이 목표는 수행할 작업이며 OCI에서 관찰한 결과가 아니다.

## 대응 Roadmap 항목

- 항목명: **Object Storage**
- Roadmap 경로: **Object Storage**
- Node ID: `sequence-6-0`
- 상위 항목: 없음 — 최상위 학습 항목
- 문서 범위: 이 항목의 개념과 작성된 실습절차. 이 파일의 한정된 문서 절차 검토 완료; 실제 OCI 실행은 NOT_EXECUTED입니다.
- [전체 항목 목록](../../../Docs/README.md)

## 사전준비


00의 실제 ACTIVE LAB_COMPARTMENT_ID와 선택 REGION을 준비한다. Console 경로는 브라우저 로그인과 해당 Compartment 접근만 필요하며 Cloud Shell/CLI 인증은 CLI 경로 또는 선택 후속 실습에서만 준비한다. NS는 아래 GetNamespace 실제 응답, BUCKET은 생성 전에 namespace 내 충돌 검사한 고유 이름이다. 선택 후속49–54에서만 CREATED_BUCKET_ID·설정 readback과 원본/다운로드 확인 결과를 인계한다. 앱/VM/CIDR 입력은 없다.

- [Introduction](../00_Introduction/00_Introduction.md#실습절차) — 링크의 `실습절차`와 `결과검증` 완료; 실제 Compartment OCID와 Region을 기록한다. 이 실습은 IP를 사용하지 않는다.



- 변수 handoff: 48의 `REGION`, `COMPARTMENT_OCID`, `NS`, `BUCKET`; bucket private/Standard/versioning disabled/auto-tiering disabled 확인. 후속에서 새 bucket 생성 금지.

## 권한

GetBucket → BUCKET_READ; ListObjects → OBJECT_INSPECT; GetObject → OBJECT_READ; HeadObject → OBJECT_READ 또는 OBJECT_INSPECT; 새 객체 PutObject → OBJECT_CREATE, 동일 key overwrite는 OBJECT_OVERWRITE 추가; DeleteObject → OBJECT_DELETE. CreateBucket/DeleteBucket → BUCKET_CREATE/BUCKET_DELETE. GetNamespace(자기 namespace, compartmentId 생략)는 추가 permission 없음.

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
- **완주 흐름:** 입력 확인→CLI GET/cmp0 또는 Console 다운로드/내용 일치→private/Standard/disabled 설정 확인→payload 삭제→빈 bucket 삭제. 49–54를 선택한 경우에만 bucket과 원본을 보존하고 선택 consumer의 반환 후 정리한다.
- **선택 이유/실패 경계:** namespace/global 이름과 Region endpoint를 구분하고 기존 파일/객체 충돌은 중단한다.

| 입력/산출물 | 실제 값 출처 | 준비 시점 |
|---|---|---|
| `NS` | creator의 실제 생성/조회 출력; 임의 OCID/IP 금지 | 사용 전 |
| `BUCKET` | 생성 전 승인/namespace 충돌 검사한 고유 bucket name; OCID와 다름 | 사용 전 |
| `COMPARTMENT_OCID` | creator의 실제 생성/조회 출력; 임의 OCID/IP 금지 | 사용 전 |
| `REGION` | 본문 단계에서 승인/선택한 값 | 사용 전 |

실행 위치: CLI는 인증 Cloud Shell/승인 OCI 터미널, Console은 본인 PC와 브라우저다. VM·SSH·NFS 준비는 필요 없다. 권한과 비용은 실제 실행 전에 확인한다.

### 선택 후속 실습의 공유 bucket lease와 비동기 분기
개인 첫 성공 경로는 다음 consumer를 예약하지 않고 다운로드 확인 후 바로 정리한다. 49–54를 선택하는 경우에만 다음 계약을 적용한다.

owner48의 동일 bucket에는 단일 승인 `BUCKET_EXPIRY_UTC`/cleanup owner가 있다. 50의 하루 이상 관찰 또는52의 restore/download가 원래60분을 넘을 가능성이 있으면 **규칙/업로드 이전에** owner48과 budget 승인자가 새 종료시각/비용/담당자와 모든 선택 consumer를 기록하여 lease를 갱신한다. 갱신 승인 없으면 원래deadline 전에 disable/cancel 가능한 시험을중단하고 owned key를삭제/반환한다; 실제비동기 작업취소가 지원되지않으면작업상태를기록하고 객체삭제/목록readback을owner48과조율한다. owner48은 expiry때 독자삭제하지않고 모든consumer의abort/restore/return acknowledgement를받는다. 연장안됨을단순히무기한보존으로대체하지않는다. ASYNC_PENDING은검증성공아니다.

## 실습절차

### 실행 터미널과 지역 고정
CLI 경로를 선택한 경우에만 아래 변수 block과 OCI 명령, local bytes 작성/비교를 같은 인증 Cloud Shell/승인 OCI 터미널의 전용 directory에서 실행한다. Console 경로는 이 block을 건너뛰고 단계2로 이동한다. VM·CIDR·앱 포트는 사용하지 않는다.
```bash
export REGION="<approved-region>"
export OCI_CLI_REGION="$REGION"
export LAB_COMPARTMENT_ID="<actual-lab-compartment-ocid>"
# 개인 경로는 이번 세션 안에 정리한다. 선택 consumer만 별도 BUCKET_EXPIRY_UTC를 기록한다.
```
치환 전 실행 금지. 실제 Region/Compartment/bucket identity와 선택 보존 종료시각을 비밀 없는 registry에 기록하고 다음 터미널에서는 필요한 변수만 다시 설정한다.

[아래생성/변경단계의정확한Oracle task본문](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/managingbuckets_topic-To_create_a_bucket.htm) — Consolefield/CLI/API동작근거.

아래 단계는 Oracle task 본문을 근거로 작성한 lab recipe이며 공식 실행결과가 아니다. 각 단계의 근거는 끝의 정확한 task 링크를 따른다.

**먼저 경로 하나 선택:** A=CLI(아래1), B=Console(아래2). B는1의 Bash block을 실행하지 않는다. 두 경로는 같은 신규 bucket/key를 중복 생성·업로드하지 않는다.

1. **A: CLI** Cloud Shell/승인 OCI 터미널에서 **먼저 전용 빈 작업 디렉터리**를 선택한다. `test ! -e lab-object.txt && test ! -e downloaded.txt || exit 1`로 충돌을 확인하고 기존 파일을 덮어쓰지 않는다. OCI CLI 인증과 Region을 확인하고 아래 입력값을 설정한다. OCID는 승인된 Compartment의 실제 값이며 예시 문자열로 실행하지 않는다.
```bash
set -euo pipefail
LAB48_WORK=$(mktemp -d "$HOME/oci-roadmap-48.XXXXXXXX") || exit 1
cd -- "$LAB48_WORK" || exit 1
# record actual LAB48_WORK/owner before creating local bytes
export COMPARTMENT_OCID="$LAB_COMPARTMENT_ID"
# dedicated empty local directory; stop on CreateBucket conflict rather than uploading into an existing bucket
export BUCKET='<unique-lab-bucket-name>'
NS=$(oci os ns get --query data --raw-output) || { printf 'STOP: namespace query failed\n'; exit 1; }
test -n "$NS" && test "$NS" != null || exit 1
case "$NS" in *[!a-zA-Z0-9_-]*) printf 'STOP: invalid namespace\n'; exit 1;; esac
export NS
test ! -e lab-object.txt && test ! -L lab-object.txt && test ! -e downloaded.txt && test ! -L downloaded.txt || exit 1
printf 'oci-roadmap object payload\n' > lab-object.txt
if ! CREATED_BUCKET_ID=$(oci os bucket create --namespace-name "$NS" --compartment-id "$COMPARTMENT_OCID" --name "$BUCKET" --public-access-type NoPublicAccess --storage-tier Standard --versioning Disabled --auto-tiering Disabled --object-events-enabled false --query data.id --raw-output); then
  printf 'STOP: CreateBucket failed; no PUT or reuse permitted\n'; exit 1
fi
test -n "$CREATED_BUCKET_ID" && test "$CREATED_BUCKET_ID" != null || exit 1
printf 'Record owner48: REGION=%s NS=%s BUCKET=%s CREATED_BUCKET_ID=%s LAB48_WORK=%s\n' "$REGION" "$NS" "$BUCKET" "$CREATED_BUCKET_ID" "$LAB48_WORK"
# 위 실제 값을 개인 작업 기록에 보존한 뒤 진행한다. 중간 실패 시 생성 명령을 재실행하지 않는다.
READBACK_BUCKET_ID=$(oci os bucket get -ns "$NS" -bn "$BUCKET" --query data.id --raw-output) || exit 1
test "$READBACK_BUCKET_ID" = "$CREATED_BUCKET_ID" || { printf 'STOP: bucket identity changed\n'; exit 1; }
oci os object put --namespace-name "$NS" --bucket-name "$BUCKET" --name lab/payload.txt --file lab-object.txt --no-overwrite
oci os object get --namespace-name "$NS" --bucket-name "$BUCKET" --name lab/payload.txt --file downloaded.txt
cmp lab-object.txt downloaded.txt
oci os bucket get -ns "$NS" -bn "$BUCKET"
oci os object head -ns "$NS" -bn "$BUCKET" --name lab/payload.txt
```
2. **B: Console만으로 완료 (본인 PC + 브라우저)**
   1. PC의 새 전용 폴더에 텍스트 편집기로 **payload.txt**를 만들고 `oci-roadmap object payload` 한 줄을 저장한다. Cloud Shell 파일 chooser를 사용하지 않는다. 원본 폴더 경로를 기록하고 기존 파일은 덮어쓰지 않는다.
   2. Console Region selector와 승인 Compartment를 확인한다. Storage → Object Storage & Archive Storage → Buckets → Create bucket: 고유 name, Namespace scope(선택 필드가 있으면 NAMESPACE), Standard, Private, Oracle-managed encryption, Auto-tiering/Versioning/Events off. retention/replication/PAR를 만들지 않는다. 충돌이면 취소하고 다른 이름으로 새로 생성한다. Bucket details의 actual Region/namespace/name/OCID와 private/disabled 상태를 기록한다.
   3. Objects → Upload: prefix=`lab/`, storage tier=Standard, Choose files from your computer에 PC의 **payload.txt** 한 개를 선택한다. 예상 key는 **lab/payload.txt**다. 중복/overwrite 경고면 취소한다. Upload 후 목록의 exact key/크기/tier를 확인한다.
   4. 같은 object의 Actions → Download를 누르고 새 download 폴더에 저장한다. PC 텍스트 편집기로 다운로드를 열어 원본과 같은 한 줄인지 확인한다. CLI cmp는 이 Console 첫 성공에 필수가 아니다. 후속 CLI consumer를 선택하면 원본/다운로드를 보존하고 그 consumer의 actual NS/BUCKET/REGION 입력을 다시 설정한다.
   5. 후속 미선택이면 아래 Console cleanup을 수행한다. Console 경로의 완료 기준은 upload/download/내용 일치와 bucket 설정이며 CLI를 실행한 것으로 기록하지 않는다.

### 실행 디렉터리와 지역 고정
CLI 추가 조회는 단계1에서 설정한 동일 REGION/NS/BUCKET을 유지한다. 새 터미널에서는 기록한 실제 값을 먼저 복원한다. 기존 local download 파일을 덮어쓰지 않도록 전용 빈 디렉터리에서 수행한다. 같은 lab key의 overwrite prompt는 예상 밖 기존객체면 취소한다.
```bash
# REGION은 생성 때 기록한 실제 값; 다른 Region으로 바꾸지 않는다.
export OCI_CLI_REGION="$REGION"
oci os bucket get -ns "$NS" -bn "$BUCKET"
oci os object list -ns "$NS" -bn "$BUCKET" --prefix lab/
```
`namespace/bucket/key`는 정확한 target이다. CLI 경로는 GET/cmp, Console 경로는 PC download/내용 비교와 bucket readback으로 끝낸다. 위 CLI 조회 block은 Console 경로에서 선택이다.

## 결과검증

CLI는 cmp 종료 코드0, Console은 원본/다운로드의 동일 내용과 exact key 존재를 확인한다. Namespace는 tenancy 값이며 Compartment 이름과 다르다.

- 예상 pattern이며 실제 OCI 실행 결과가 아니다. timestamp, resource OCID, 설정 readback, 실제 파일 비교 결과를 실행자가 기록한다.

### 선택: 내용 불일치 검출과 복구
첫 성공 뒤 비교 기능을 확인하고 싶을 때만 수행한다. cloud 객체/권한은 변경하지 않는다.
- CLI: 같은 LAB48_WORK에서 아래 block을 실행한다. 원본과 다운로드는 수정하지 않고 별도 대조 파일만 만든다.
```bash
test ! -e mismatch.txt && test ! -L mismatch.txt || exit 1
printf 'intentional mismatch\n' > mismatch.txt
if cmp -s lab-object.txt mismatch.txt; then
  printf 'STOP: unexpected equal content\n'; exit 1
else
  CMP_STATUS=$?
  test "$CMP_STATUS" -eq 1 || { printf 'STOP: comparison error\n'; exit 1; }
fi
cmp lab-object.txt downloaded.txt || exit 1
rm -- mismatch.txt
```
- Console: 다운로드 파일의 복사본만 만들어 내용을 다른 한 줄로 바꾸어 원본과 다름을 확인한다. 복사본만 삭제하고 원본/실제 다운로드를 다시 열어 동일 내용을 확인한다. 원본을 잃었다면 단계2.1의 한 줄과 exact key를 확인하고 재다운로드한다. 이 검사는 내용 비교 연습이며 OCI 접근 거부 검증은 아니다.

## 문제해결

namespace/bucket-region오류는OCI_CLI_REGION/NS/BUCKET세값을같은target으로고정. localfileexists는전용새다운로드filename사용;privatemode를public으로수정하지않는다.

CLI/Console 모두 bucket 생성 이후 실패하면 새 bucket을 만들지 않는다. 기록한 Region/namespace/name/OCID를 다시 확인하고, PUT 이전 실패면 빈 bucket 정리로, PUT/GET 이후 실패면 exact key 목록 확인과 다운로드 재시도로 돌아간다. 부분 다운로드는 이번 전용 폴더의 파일임을 확인한 뒤 새 파일명으로 재다운로드하며 원본은 보존한다. 권한/인증 오류는 public 전환이나 PAR 생성으로 우회하지 않는다.

실패기록에는같은target/credentials/region/요청시간을고정한다. 복구는원래설정으로실행하고actualresponse/readback둘다확인한다;예상값을실측으로쓰지않는다.

## 리소스정리

**Console 개인 종료:** 후속 consumer가 없으면 exact bucket → Objects → lab/payload.txt → Actions → Delete를 확인한다. 목록을 새로고침해 key가 없고 다른 object도 없는지 확인한 뒤 bucket Actions → Delete를 확인한다. 같은 Region/Compartment 목록에서 bucket 부재를 확인한다. 실패하면 PC 원본을 보존하고 권한/남은 object를 확인하며 다른 bucket은 삭제하지 않는다.

**CLI 개인 종료:** 후속 미선택이면 lease 대기 없이 다음 key→목록→bucket 순서로 정리한다. 삭제 전 `oci os bucket get -ns "$NS" -bn "$BUCKET"`의 id가 기록한 CREATED_BUCKET_ID와 같은지 확인하고, 불일치/조회 오류면 중단한다. 새 터미널은 기록한 REGION/OCI_CLI_REGION/NS/BUCKET/CREATED_BUCKET_ID를 먼저 복원한다.

**선택 후속 종료(개인 경로는 다음 공통 삭제 순서로 바로 이동):** 위 단일 bucket lease의 선택49–54가 모두 return/abort acknowledgement를 기록한 뒤 owner48만 정리한다. expiry가 먼저면 새 시험을 중단하고 각 owner의 disable/owned-key 삭제/반환을 조율한다. 갱신 lease가 있으면 갱신된 deadline을 사용한다. 기존60분 문장으로 renewed lease를 덮어쓰거나 consumer 확인 없이 삭제하지 않는다. 

**공통 삭제 순서:** owner48의 `lab/payload.txt`도 `oci os object delete -ns "$NS" -bn "$BUCKET" --name lab/payload.txt`로 삭제한다. 이어 `oci os object list -ns "$NS" -bn "$BUCKET" --all`로빈bucket확인. 남은실습key를각owner가삭제하고 `oci os bucket delete -ns "$NS" -bn "$BUCKET"`를확인prompt후실행. `oci os bucket get`의BucketNotFound와동일namespace정상조회로삭제검증.


remote 반환/정리가 끝나고 증거 보존 승인이 종료되면 registry의 LAB48_WORK 안에 이번 생성한 regular/non-symlink `lab-object.txt`/`downloaded.txt`/선택 `mismatch.txt` 및 이번 재다운로드 파일(실패 partial 포함)만 exact 경로로 제거한다. 디렉터리를 벗어난 뒤 `rmdir -- "$LAB48_WORK"`로 빈 owned directory만 제거한다.49의 전용 파일과 기존 사용자의 파일은 정리하지 않는다.
정리 UTC/namespace/bucket/owned key·목록 readback과 owner48 반환을 기록한다. shared bucket은 consumer가 삭제하지 않는다. Console 경로는 원격 정리 후 PC의 이번 원본/다운로드/선택 복사본만 삭제하고 작업 기록은 남긴다.

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
