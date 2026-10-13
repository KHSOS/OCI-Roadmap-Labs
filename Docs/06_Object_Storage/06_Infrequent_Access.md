# Infrequent Access

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../README.md)를 확인하세요.

> 절차 작성본 / OCI 미실행 — 아래 예상값은 실측 결과가 아닙니다. 비용·권한·사전준비 gate를 통과한 전용 환경에서만 실행하세요.

## 학습목표

- 기존 bucket에 InfrequentAccess key 하나를 업로드하고 즉시 GET/cmp 및 tier를 확인한다.
- 다른 local bytes의 cmp=1을 정상 GET/cmp=0과 대조하고 key 삭제 후에도 남는 최소기간 과금 의무를 기록한다.

완료 기준은 아래 실제 생성·설정·요청 결과로 판정한다. 이 목표는 수행할 작업이며 OCI에서 관찰한 결과가 아니다.

## 대응 Roadmap 항목

- 항목명: **Infrequent Access**
- Roadmap 경로: **Object Storage → Infrequent Access**
- Node ID: `sequence-6-6`
- 상위 항목: **Object Storage** (`sequence-6-0`)
- 문서 범위: 이 항목의 개념과 작성된 실습절차. 통합 품질검토는 BLOCKED이며 실제 실행은 NOT_EXECUTED입니다.
- [전체 항목 목록](../README.md)

## 사전준비


48 actual REGION/NS/BUCKET·private Standard/auto-tiering disabled와 lease를 받는다. 이번 lab/ia.txt 부재·IA31일/검색/조기삭제 비용을 먼저 승인한다. Cloud Shell의 전용 source/download directory에서 bytes를 대조하며 앱/VM/Archive restore는 필요 없다.

- [Object Storage](00_Object_Storage.md#실습절차) — 링크의 `실습절차`와 `결과검증` 완료; 실제 OCID/IP를 registry에 인계한다.



- 변수 handoff: 48의 `REGION`, `COMPARTMENT_OCID`, `NS`, `BUCKET`; bucket private/Standard/versioning disabled/auto-tiering disabled 확인. 후속에서 새 bucket 생성 금지.

## 권한

GetBucket → BUCKET_READ; ListObjects → OBJECT_INSPECT; GetObject/HeadObject → OBJECT_READ; 새 객체 PutObject → OBJECT_CREATE, 동일 key overwrite는 OBJECT_OVERWRITE 추가; DeleteObject → OBJECT_DELETE.

[API operation별 공식 permission table](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm) — 가족 단위 manage를 최소권한이라 부르지 않는다. 전용 lab compartment의 group만 적용하고 정책작성 권한은 관리자 소유다.
```text
Allow group <identity-domain>/<lab-group> to manage object-family in compartment <lab-compartment> where any {request.permission='BUCKET_INSPECT', request.permission='BUCKET_READ', request.permission='OBJECT_INSPECT', request.permission='OBJECT_READ', request.permission='OBJECT_CREATE', request.permission='OBJECT_DELETE'}
```
이 allowlist는 위 작업만 허용한다. Console 탐색 List API가 거부되면 실제 API permission을 공식 표에서 별도로 승인하며 all-resources로 확대하지 않는다.

## 비용및주의사항

owner48 Standard bucket에 `lab/ia.txt` InfrequentAccess object 하나만 추가한다. IA 저장·즉시 검색/다운로드·31일 최소 저장기간·조기삭제 및 실제 전송 비용을 업로드 전에 승인한다. 삭제가 최소기간 의무를 없애지 않으므로 반환 기록에 잔여 비용을 남긴다. Archive restore·새 bucket/VM은 없다. local cmp mismatch는 서비스 장애가 아니며 정상 GET/cmp를 확인한 뒤 lease 안에 이번 key만 삭제한다.

## 실습구성

- 작업 목표: Infrequent Access 객체의 실제 업로드/검색을 검증한다.
- 생성 owner: 54. 동일 owner의 자원을 재사용하며 실제 결과 필드는 실행 전 비워 둔다.


### 이번 workshop 시나리오와 인수·반환

IA key를 바로 읽고 일부러 다른 local bytes와 비교한 뒤 정상 대조한다.

- **신규/변경:** lab/ia.txt IA object1개.
- **선행 재사용:** 48 private Standard bucket.
- **완주 흐름:** 승인/입력→본문의 생성/편집→HEAD IA,wrong local cmp실패,정상 GET/cmp0→ia.txt삭제→48 반환;31일 잔여 기록.
- **선택 이유/실패 경계:** local mismatch는 service 장애가 아니다; Archive restore를 호출하지 않는다.

| 입력/산출물 | 실제 값 출처 | 준비 시점 |
|---|---|---|
| `NS` | creator의 실제 생성/조회 출력; 임의 OCID/IP 금지 | 사용 전 |
| `BUCKET` | creator의 실제 생성/조회 출력; 임의 OCID/IP 금지 | 사용 전 |
| `ia.txt` | 본문 단계에서 승인/선택한 값 | 사용 전 |

입력별 실행위치는 본문 터미널 구분을 따른다: OCI CLI=인증 Cloud Shell/승인 OCI 터미널, OS 앱/NFS/부하=지정 guest, private HTTP/TCP=vm-a guest, YAML 작성/SSH private key=본인 local. 조회/변경/정리의 관리자 승인은 **실행 조건**이며 이 문서는 IAM 성공을 입증하지 않는다. 실제 허용/거부와 capacity는 사용자 실행에서 확인한다. 미승인/NOT_SELECTED/ASYNC_PENDING을 성공으로 바꾸지 않는다.

### 공유 bucket lease와 비동기 분기
owner48의 동일 bucket에는 단일 승인 `BUCKET_EXPIRY_UTC`/cleanup owner가 있다. 50의 하루 이상 관찰 또는52의 restore/download가 원래60분을 넘을 가능성이 있으면 **규칙/업로드 이전에** owner48과 budget 승인자가 새 종료시각/비용/담당자와 모든 선택 consumer를 기록하여 lease를 갱신한다. 갱신 승인 없으면 원래deadline 전에 disable/cancel 가능한 시험을중단하고 owned key를삭제/반환한다; 실제비동기 작업취소가 지원되지않으면작업상태를기록하고 객체삭제/목록readback을owner48과조율한다. owner48은 expiry때 독자삭제하지않고 모든consumer의abort/restore/return acknowledgement를받는다. 연장안됨을단순히무기한보존으로대체하지않는다. ASYNC_PENDING은검증성공아니다.

## 실습절차

### 작업 순서 체크포인트
새 IA key PUT→즉시 GET/cmp→wrong local bytes rc1 control→새 authenticated GET/cmp→exact key cleanup을 수행한다. 단계 실패는 다음 생성으로 넘기지 않고 같은 target의 복구/반환으로 처리한다.

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

### Source/download 작업 디렉터리 — 첫 printf/PUT 이전
Cloud Shell/승인 OCI 터미널의 **같은 shell**에서 실행한다. 새 directory 생성 실패면 진행하지 않는다. 이전 cwd의 같은 파일을 읽거나 덮어쓰지 않는다.
```bash
set -euo pipefail
LAB_WORK=$(mktemp -d "$HOME/oci-roadmap-54.XXXXXXXX") || exit 1
cd -- "$LAB_WORK" || exit 1
for name in ia.txt ia-read.txt ia-wrong.txt ia-control.txt; do
  if test -e "$name" || test -L "$name"; then printf 'STOP: local collision %s\n' "$name"; exit 1; fi
done
# Record actual LAB_WORK and file creation ownership in consumer registry.
```
Remote PUT도 같은 key의 기존 object 충돌을 owner48 목록에서 확인하고 충돌이면 중단한다. 새 PUT은 `--no-overwrite`를 추가한다. 다음 절차의 출력 파일은 이 directory에서만 생성한다.


1. [Object Storage](00_Object_Storage.md)의 private bucket을 사용하고 IA 검색/최소기간 과금을 승인한다.
2. `printf 'ia readable payload\n' > ia.txt; oci os object put -ns "$NS" -bn "$BUCKET" --name lab/ia.txt --file ia.txt --storage-tier InfrequentAccess --no-overwrite`를 실행한다.
3. Console의 객체 tier를 확인하고 `oci os object get -ns "$NS" -bn "$BUCKET" --name lab/ia.txt --file ia-read.txt; cmp ia.txt ia-read.txt`로 검증한다. Archive restore 절차를 IA에 적용하지 않는다.
4. 사용패턴상 읽기 횟수가 증가할 때 Standard로 돌아가는 판단 근거를 검색비용과 함께 기록한다.

### IA 온라인 읽기와 데이터 오류 양성대조
```bash
oci os object head -ns "$NS" -bn "$BUCKET" --name lab/ia.txt
printf 'wrong local control\n' > ia-wrong.txt
test -r ia-wrong.txt && test -r ia-read.txt || exit 1
set +e
cmp ia-wrong.txt ia-read.txt
rc=$?
set -e
test "$rc" -eq 1 || { printf 'expected different bytes only, rc=%s\n' "$rc"; exit 1; }
oci os object get -ns "$NS" -bn "$BUCKET" --name lab/ia.txt --file ia-control.txt
cmp ia.txt ia-control.txt
```
이negative는 데이터대조 실패이지 storage service failure를 주입한것이아니다. IA에서는 restore 요청이필요없다.

## 결과검증

InfrequentAccess tier와 즉시 다운로드 cmp 0을 기대한다. 작은 객체라도 비용 무료를 보장하지 않는다.

- 예상 pattern이며 실제 OCI 실행 결과가 아니다. timestamp, resource OCID, 설정 readback, 실제 HTTP/TCP/파일 결과를 실행자가 기록한다.

## 문제해결

IA에는Archive복구지연없다. retrieval실패는IAM/region/key/전송을점검하고restore로우회금지.

실패기록에는같은target/credentials/region/요청시간을고정한다. 복구는원래설정으로실행하고actualresponse/readback둘다확인한다;예상값을실측으로쓰지않는다.

## 리소스정리

로컬은 registry의 이 실행 LAB_WORK 안에 생성한 ia.txt / ia-read.txt / ia-wrong.txt / ia-control.txt만 확인 후 `rm -- ia.txt ia-read.txt ia-wrong.txt ia-control.txt`로 제거한다(생성되지 않은 파일은 건너뛴다). 먼저 증거를 저장하고 `cd "$HOME"` 후 exact LAB_WORK에 `rmdir -- "$LAB_WORK"`를 실행한다. nonempty이면 추가 파일을 조사하고 recursive/glob 삭제하지 않는다.

`oci os object delete -ns "$NS" -bn "$BUCKET" --name lab/ia.txt`; objectlist없음확인;31일최소기간과금잔여기록.

정리 UTC/namespace/bucket/owned key·목록 readback과 owner48 반환을 기록한다. shared bucket은 consumer가 삭제하지 않는다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 목표는 학습 후 설명·구분·확인할 내용이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [36] [Object Storage Storage Tiers — Infrequent Access](https://docs.oracle.com/en-us/iaas/Content/Object/Concepts/understandingstoragetiers.htm)
  - 확인 범위: The Infrequent Access tier is "cool".

### 절차별 공식 task body 추가 근거
- [understandingstoragetiers.htm](https://docs.oracle.com/en-us/iaas/Content/Object/Concepts/understandingstoragetiers.htm) — 본문 검토 근거; 아래 정확한 task와 작성한 입력을 구분한다. OCI 실행 증거가 아니다.



### 이번 절차의 정확한 공식 task
- [managingobjects_topic-To_upload_objects_to_a_bucket.htm](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/managingobjects_topic-To_upload_objects_to_a_bucket.htm) — 이 문서 실습절차의 입력/동작을 대조한 본문.
