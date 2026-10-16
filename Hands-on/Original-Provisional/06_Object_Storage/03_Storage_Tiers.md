> **범위 한정 문서 검토 완료 / OCI 실행 미검증**: 이 Storage Tiers 절차의 문서 검토와 해당 잔여 결함 수정이 완료되었습니다. 전체 교육과정 승인이나 실제 OCI 실행 검증을 뜻하지 않습니다.

# Storage Tiers

> 문서 검토 범위: 이 파일의 실행 절차·복구·정리는 범위 한정 문서 검토를 완료했습니다. 전체 교육과정 승인·공개·렌더링 검증과는 별개입니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 전체 범위의 상태는 [통합 상태](../../../Docs/README.md)를 확인하세요.

> 절차 작성본 / OCI 미실행 — 아래 예상값은 실측 결과가 아닙니다. 비용·권한·사전준비 gate를 통과한 전용 환경에서만 실행하세요.

## 학습목표

- 같은 Standard bucket의 lab/tier.txt를 Standard→InfrequentAccess로 변경하여 tier readback과 GET/cmp를 수행한다.
- Standard로 한 번 복원하여 tier/bytes를 재검증하고 key 삭제 및 IA 최소기간 비용 의무를 기록한다.

완료 기준은 아래 실제 생성·설정·요청 결과로 판정한다. 이 목표는 수행할 작업이며 OCI에서 관찰한 결과가 아니다.

## 대응 Roadmap 항목

- 항목명: **Storage Tiers**
- Roadmap 경로: **Object Storage → Storage Tiers**
- Node ID: `sequence-6-3`
- 상위 항목: **Object Storage** (`sequence-6-0`)
- 문서 범위: 이 항목의 개념과 실행 절차에 대한 범위 한정 문서 검토 완료. 실제 OCI 실행은 NOT_EXECUTED이며 권한·비용·복구·공유 bucket 반환 조건은 실행 시 충족해야 합니다.
- [전체 항목 목록](../../../Docs/README.md)

## 사전준비


48 actual REGION/NS/BUCKET/Compartment·Standard/private/auto-tiering disabled 및 bucket lease를 받는다. 이번 전용 lab/tier.txt의 부재·IA31일/검색·삭제 비용 승인을 먼저 확인한다. platform/VM/Archive 입력은 필요 없고 tier/bytes 원복과 key 삭제를48에 반환한다.

- [Object Storage](00_Object_Storage.md#실습절차) — 같은 bucket의 생성·조회와 `결과검증`까지 완료하고 bucket은 보존한다. REGION/NS/BUCKET/Compartment와 bucket OCID를 인계한다. VM/IP는 필요 없다.



- 변수 handoff: 48의 `REGION`, `COMPARTMENT_OCID`, `NS`, `BUCKET`; bucket private/Standard/versioning disabled/auto-tiering disabled 확인. 후속에서 새 bucket 생성 금지.

## 권한

GetBucket → BUCKET_READ; ListObjects → OBJECT_INSPECT; GetObject/HeadObject → OBJECT_READ; 새 객체 PutObject → OBJECT_CREATE, 동일 key overwrite는 OBJECT_OVERWRITE 추가; DeleteObject → OBJECT_DELETE. UpdateObjectStorageTier → OBJECT_UPDATE_TIER.

[API operation별 공식 permission table](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm) — 가족 단위 manage를 최소권한이라 부르지 않는다. 전용 lab compartment의 group만 적용하고 정책작성 권한은 관리자 소유다.
```text
Allow group <identity-domain>/<lab-group> to manage object-family in compartment <lab-compartment> where any {request.permission='BUCKET_INSPECT', request.permission='BUCKET_READ', request.permission='OBJECT_INSPECT', request.permission='OBJECT_READ', request.permission='OBJECT_CREATE', request.permission='OBJECT_DELETE', request.permission='OBJECT_UPDATE_TIER'}
```
이 allowlist는 위 작업만 허용한다. Console 탐색 List API가 거부되면 실제 API permission을 공식 표에서 별도로 승인하며 all-resources로 확대하지 않는다.

## 비용및주의사항

owner48 Standard bucket의 `lab/tier.txt` 한 개만 Standard→InfrequentAccess→Standard로 바꾼다. IA 저장·검색·31일 최소 저장기간 및 데이터 삭제/덮어쓰기에 따른 조기삭제 비용을 변경 전에 승인한다. tier 전환의 실제 과금 적용은 해당 계정 가격조건으로 확인한다. 즉시 Standard 복원/삭제해도 최소기간 과금 의무가 없어지지 않는다. Archive object/restore는 이 문서에서 만들지 않는다. GET/cmp·tier readback과 key 삭제를 owner48 lease 안에 끝내고 잔여 비용 의무를 반환 기록에 남긴다.

## 실습구성

- 작업 목표: 동일 private bucket에서 객체별 tier 선택과 변경을 수행한다.
- 생성 owner: 51. 동일 owner의 자원을 재사용하며 실제 결과 필드는 실행 전 비워 둔다.


### 이번 workshop 시나리오와 인수·반환

객체 한 개의 tier를 바꾸면서 같은 bytes를 읽고 Standard로 복원한다.

- **신규/변경:** lab/tier.txt; Standard→IA→Standard.
- **선행 재사용:** 48 Standard bucket.
- **완주 흐름:** 승인/입력→본문의 생성/편집→각 tier HEAD, GET/cmp0,원복 Standard→tier.txt삭제→48 반환;31일 과금 인계.
- **선택 이유/실패 경계:** bucket default immutable; IA 검색/조기삭제 비용 승인.


- 입력/산출물: `NS`
  - 실제 값 출처: creator의 실제 생성/조회 출력; 임의 OCID/IP 금지
  - 준비 시점: 사용 전

- 입력/산출물: `BUCKET`
  - 실제 값 출처: creator의 실제 생성/조회 출력; 임의 OCID/IP 금지
  - 준비 시점: 사용 전

- 입력/산출물: `tier.txt`
  - 실제 값 출처: 본문 단계에서 승인/선택한 값
  - 준비 시점: 사용 전


입력별 실행위치는 본문 터미널 구분을 따른다: OCI CLI=인증 Cloud Shell/승인 OCI 터미널, OS 앱/NFS/부하=지정 guest, private HTTP/TCP=vm-a guest, YAML 작성/SSH private key=본인 local. 조회/변경/정리의 관리자 승인은 **실행 조건**이며 이 문서는 IAM 성공을 입증하지 않는다. 실제 허용/거부와 capacity는 사용자 실행에서 확인한다. 미승인/NOT_SELECTED/ASYNC_PENDING을 성공으로 바꾸지 않는다.

### 공유 bucket lease와 비동기 분기
owner48의 동일 bucket에는 단일 승인 `BUCKET_EXPIRY_UTC`/cleanup owner가 있다. 50의 하루 이상 관찰 또는52의 restore/download가 원래60분을 넘을 가능성이 있으면 **규칙/업로드 이전에** owner48과 budget 승인자가 새 종료시각/비용/담당자와 모든 선택 consumer를 기록하여 lease를 갱신한다. 갱신 승인 없으면 원래 deadline 전에 disable/cancel 가능한 시험을 중단하고 owned key를 삭제/반환한다. 실제 비동기 작업 취소가 지원되지 않으면 작업 상태를 기록하고 객체 삭제/목록 readback을 owner48과 조율한다. owner48은 expiry 때 독자 삭제하지 않고 모든 consumer의 abort/restore/return acknowledgement를 받는다. 연장 불가를 무기한 보존으로 대체하지 않는다. ASYNC_PENDING은 검증 성공이 아니다.

## 실습절차

### 작업 순서 체크포인트
새 object Standard→IA 변경→GET/cmp→Standard 원복→새 GET/cmp→exact key cleanup을 수행한다. 단계 실패는 다음 생성으로 넘기지 않고 같은 target의 복구/반환으로 처리한다.

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

[아래생성/변경단계의정확한Oracle task본문](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/managingobjects_topic-To_update_the_storage_tier_of_an_object.htm) — Consolefield/CLI/API동작근거.

아래 단계는 Oracle task 본문을 근거로 작성한 lab recipe이며 공식 실행결과가 아니다. 각 단계의 근거는 끝의 정확한 task 링크를 따른다.

### Source/download 작업 디렉터리 — 첫 printf/PUT 이전
Cloud Shell/승인 OCI 터미널의 **같은 shell**에서 실행한다. 새 directory 생성 실패면 진행하지 않는다. 이전 cwd의 같은 파일을 읽거나 덮어쓰지 않는다.
```bash
set -euo pipefail
umask 077
LAB_WORK=$(mktemp -d "$HOME/oci-roadmap-51.XXXXXXXX") || exit 1
cd -- "$LAB_WORK" || exit 1
for name in tier.txt tier-read.txt tier-restored.txt; do
  if test -e "$name" || test -L "$name"; then printf 'STOP: local collision %s\n' "$name"; exit 1; fi
done
# Record actual LAB_WORK and file creation ownership in consumer registry.
```
Remote PUT도 같은 key의 기존 object 충돌을 owner48 목록에서 확인하고 충돌이면 중단한다. 새 PUT은 `--no-overwrite`를 추가한다. 다음 절차의 출력 파일은 이 directory에서만 생성한다.


### IA mutation 전 기록과 fresh-terminal 복구
이 절은 실패 시 돌아올 복구 station이다. 정상 경로는 다음 `객체 업로드와 tier 변경`으로 진행한다. PUT 전에 private 실행 기록을 LAB_WORK 밖의 본인 보존 위치에 작성한다. REGION/NS/BUCKET/Compartment/bucket OCID, key=`lab/tier.txt`, original tier=Standard, LAB_WORK, 비용 deadline/복구 담당자, exact key 부재를 확인한 시각을 저장하고 상태는 `PUT_PENDING`으로 둔다. source hash와 PUT 결과를 같은 기록에 추가한다. credential은 저장하지 않는다. 기록 위치도 별도로 보존하며 shell history에만 의존하지 않는다.
PUT 성공/etag로 이번 key 소유권을 확인한 직후 상태를 `OWNED`로 기록하고, **첫 tier 변경 이전** 기록을 다시 열어 모든 입력과 hash가 보존되었는지 확인한다. owner48은 이 key를 다른 consumer가 쓰거나 변경하지 않도록 한다. 원본 bytes는 remote 정리까지 보존한다. API/Get/cmp 실패나 terminal loss이면 새 시험 대신 아래를 수행한다.

PUT 응답 전에 연결이 끊겨 `PUT_PENDING`인 경우에는 재업로드/삭제/변경하지 않는다. owner48과 성공한 목록/객체 상세 및 업로드 시각·내용을 대조하여 이번 실행의 생성인지 확인한 뒤에만 `OWNED`로 바꾼다. 확인할 수 없으면 잔존 key와 deadline을 owner48에게 반환하고 중단한다.

1. fresh 인증 shell에서 기록의 actual REGION/NS/BUCKET/LAB_WORK를 다시 설정하고 OCI_CLI_REGION=REGION을 export한다. bucket OCID를 readback하고 `cd -- "$LAB_WORK"`로 이동한다. tier.txt의 hash가 기록과 같은지 확인한다. 없거나 다르면 원복 성공을 추정하지 말고 owner48에게 반환한다.
2. Console → exact bucket → Objects → lab/tier.txt details의 현재 tier를 읽는다. Infrequent Access면 Actions → **Update storage tier → Standard → Update**; 이미 Standard면 변경을 반복하지 않는다. Archive/다른 key/오류면 중단한다. CLI 대안은 아래 Standard update 한 건이며 Console과 중복하지 않는다. 새 details/HEAD로 Standard를 확인한다.
3. 새 download filename을 선택해 GET하고 보존한 tier.txt와 cmp한다. 성공/실패를 기록한 뒤 owner48과 이번 key 삭제를 진행하고 목록 부재를 확인한다. Standard 복귀로 IA 최소기간/검색 비용이 취소되지 않는다. local 원본은 remote 삭제/반환 뒤 정리한다.


### 객체 업로드와 tier 변경
1. [Object Storage](00_Object_Storage.md#실습절차)에서 보존한 같은 bucket을 확인한다. 위 private 기록을 작성한 뒤 다음을 같은 LAB_WORK에서 실행한다. source hash 출력과 PUT 성공 응답/etag를 기록하고 `OWNED` 기록을 확인하기 전에는 2로 진행하지 않는다.
```bash
printf 'tier payload\n' > tier.txt
sha256sum -- tier.txt
oci os object put -ns "$NS" -bn "$BUCKET" --name lab/tier.txt --file tier.txt --storage-tier Standard --no-overwrite
oci os object head -ns "$NS" -bn "$BUCKET" --name lab/tier.txt
```
HEAD의 tier=Standard를 확인한다. PUT 실패/충돌은 덮어쓰지 말고 위 복구 station에서 판정한다.
2. Console → 같은 bucket → Objects → lab/tier.txt의 Actions → **Update storage tier → Infrequent access → Update**를 선택한다. 최소 저장기간/검색비용을 먼저 승인한다. 전체 bucket default를 변경하지 않는다.
3. `oci os object update-storage-tier -ns "$NS" -bn "$BUCKET" --name lab/tier.txt --storage-tier InfrequentAccess`는 2의 Console 변경을 수행하지 않은 경우에만 쓰는 대안이다. `oci os object head -ns "$NS" -bn "$BUCKET" --name lab/tier.txt`로 **tier=InfrequentAccess**를 확인한 뒤, `oci os object get -ns "$NS" -bn "$BUCKET" --name lab/tier.txt --file tier-read.txt; cmp tier.txt tier-read.txt`로 검증한다. cmp exit 0만 bytes 동일 성공이다. 다음 negative control은 비어 있지 않은 다운로드가 빈 파일과 다름을 확인하는 **로컬 검사**이며 OCI 접근 거부 시험은 아니다.
```bash
if cmp tier-read.txt /dev/null; then
  printf 'STOP: unexpectedly empty payload\n'; exit 1
else
  rc=$?
  test "$rc" -eq 1 || { printf 'STOP: cmp I/O error\n'; exit 1; }
fi
```
Standard 복원은 아래 원복 block에서 한 번만 수행한다.
4. 이 문서는 Standard 복원과 key 정리로 끝낸다. [Archive](04_Archive.md)는 별도 비용/대기시간을 승인한 다음 선택 실습이며 현재 완료 기준에 포함하지 않는다. Auto-Tiering은 켜지 않는다.

### tier 원복과 객체 양성대조
```bash
oci os object update-storage-tier -ns "$NS" -bn "$BUCKET" --name lab/tier.txt --storage-tier Standard
oci os object head -ns "$NS" -bn "$BUCKET" --name lab/tier.txt
oci os object get -ns "$NS" -bn "$BUCKET" --name lab/tier.txt --file tier-restored.txt
cmp tier.txt tier-restored.txt
test -s tier-restored.txt || { printf 'ERROR: empty or missing payload\n'; exit 1; }
```
HEAD의 tier=Standard와 cmp0이 원복조건; bucket의 immutable default tier는 변경하지 않는다.

## 결과검증

Standard→Infrequent Access→Standard 상태와 payload 동일성을 기록한다. 조기 변경/삭제 비용이 없어지는 것은 아니다.

- 예상 pattern이며 실제 OCI 실행 결과가 아니다. timestamp, resource OCID, 설정 readback, 실제 HTTP/TCP/파일 결과를 실행자가 기록한다.

## 문제해결

tier 변경 권한 OBJECT_UPDATE_TIER를 확인한다. IA retrieval/early delete 비용을 읽고 Auto Tiering과 충돌하지 않게 한다.

실패 기록에는 같은 target/credentials/region/요청 시간을 고정한다. 복구는 원래 설정으로 실행하고 actual response/readback을 둘 다 확인한다. 예상값을 실측으로 쓰지 않는다.

## 리소스정리

먼저 `oci os object delete -ns "$NS" -bn "$BUCKET" --name lab/tier.txt`로 이번 key를 삭제한다. 성공한 object list에서 exact key 부재와 owner48 반환을 확인한다. 부분 실패/terminal loss이면 위 복구 station으로 이동한다. IA31일 최소 비용 의무도 반환한다. 그 뒤에만 LAB_WORK에서 존재하는 owned regular 파일을 지운다.

```bash
cd -- "$LAB_WORK" || exit 1
for name in tier.txt tier-read.txt tier-restored.txt; do
  test ! -L "$name" || exit 1
  if test -e "$name"; then test -f "$name" || exit 1; rm -- "$name" || exit 1; fi
done
cd "$HOME" || exit 1
rmdir -- "$LAB_WORK" || exit 1
```
다른 재개 download 파일은 기록한 exact 소유 경로만 별도 정리하며 nonempty directory는 조사한다.

정리 UTC/namespace/bucket/owned key·목록 readback과 owner48 반환을 기록한다. shared bucket은 consumer가 삭제하지 않는다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 목표는 학습 후 설명·구분·확인할 내용이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [36] [Object Storage Storage Tiers — Storage Tiers](https://docs.oracle.com/en-us/iaas/Content/Object/Concepts/understandingstoragetiers.htm)
  - 확인 범위: Standard Tier The / Infrequent Access The / Archive The.

### 절차별 공식 task body 추가 근거
- [understandingstoragetiers.htm](https://docs.oracle.com/en-us/iaas/Content/Object/Concepts/understandingstoragetiers.htm) — 본문 검토 근거; 아래 정확한 task와 작성한 입력을 구분한다. OCI 실행 증거가 아니다.



### 이번 절차의 정확한 공식 task
- [managingobjects_topic-To_update_the_storage_tier_of_an_object.htm](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/managingobjects_topic-To_update_the_storage_tier_of_an_object.htm) — 이 문서 실습절차의 입력/동작을 대조한 본문.
