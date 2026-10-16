> **문서 검토 완료 — OCI 실행 미검증**: 이 Archive 실습의 한정된 절차·복구·정리 문서 검토를 완료했습니다. 실제 OCI 실행, 전체 커리큘럼 승인 또는 게시 검증을 뜻하지 않습니다.

# Archive

> 문서 상태: **한정된 실습절차 검토 완료**. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 권한·비용·lease·복구·정리의 실행 조건은 아래 절차대로 확인하세요. 전체 범위의 상태는 [통합 상태](../../../Docs/README.md)를 확인하세요.

> 절차 작성본 / OCI 미실행 — 아래 예상값은 실측 결과가 아닙니다. 비용·권한·사전준비 gate를 통과한 전용 환경에서만 실행하세요.

## 학습목표

- 기존 Standard bucket에 Archive key 하나를 업로드하고 Restore 전 같은 key GET 거부와 HEAD 상태를 기록한다.
- Restore를 한 번만 제출하여 Restored 상태의 GET/cmp를 확인하거나 lease 기한 전 중단/owned-key 삭제/반환을 기록한다.

완료 기준은 아래 실제 생성·설정·요청 결과로 판정한다. 이 목표는 수행할 작업이며 OCI에서 관찰한 결과가 아니다.

## 대응 Roadmap 항목

- 항목명: **Archive**
- Roadmap 경로: **Object Storage → Archive**
- Node ID: `sequence-6-4`
- 상위 항목: **Object Storage** (`sequence-6-0`)
- 문서 범위: 이 항목의 개념과 작성된 실습절차. 이 Archive 절차의 한정된 문서 검토는 완료했으며 실제 OCI 실행은 NOT_EXECUTED입니다.
- [전체 항목 목록](../../../Docs/README.md)

## 사전준비


48 actual REGION/NS/BUCKET/Compartment·private Standard bucket과 단일 lease를 받는다. Archive90일/복원된 Standard 저장·다운로드 비용과 필요시 갱신 BUCKET_EXPIRY_UTC를 업로드 전에 승인한다. Restoring/Restored 상태와 새 GET/cmp를 대조하며 새 Archive bucket/VM은 필요 없다.

- [Object Storage](00_Object_Storage.md#실습절차) — 링크의 `실습절차`와 `결과검증` 완료; 실제 OCID/IP를 registry에 인계한다.



- 변수 handoff: 48의 `REGION`, `COMPARTMENT_OCID`, `NS`, `BUCKET`; bucket private/Standard/versioning disabled/auto-tiering disabled 확인. 후속에서 새 bucket 생성 금지.

## 권한

GetBucket → BUCKET_READ; ListObjects → OBJECT_INSPECT; GetObject/HeadObject → OBJECT_READ; 새 객체 PutObject → OBJECT_CREATE, 동일 key overwrite는 OBJECT_OVERWRITE 추가; DeleteObject → OBJECT_DELETE. RestoreObjects → OBJECT_RESTORE.

[API operation별 공식 permission table](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm) — 가족 단위 manage를 최소권한이라 부르지 않는다. 전용 lab compartment의 group만 적용하고 정책작성 권한은 관리자 소유다.
```text
Allow group <identity-domain>/<lab-group> to manage object-family in compartment <lab-compartment> where any {request.permission='BUCKET_INSPECT', request.permission='BUCKET_READ', request.permission='OBJECT_INSPECT', request.permission='OBJECT_READ', request.permission='OBJECT_CREATE', request.permission='OBJECT_DELETE', request.permission='OBJECT_RESTORE'}
```
이 allowlist는 위 작업만 허용한다. Console 탐색 List API가 거부되면 실제 API permission을 공식 표에서 별도로 승인하며 all-resources로 확대하지 않는다.

## 비용및주의사항

owner48의 Standard bucket에 `lab/archive.txt` Archive object 하나만 추가한다. Archive90일 최소 저장기간·조기삭제, 복원 중 Standard tier 저장 및 요청/다운로드·전송 비용을 업로드 전에 승인한다. 즉시 삭제해도 최소기간 의무가 없어지지 않는다. 복원 완료까지 최대1시간과 완료 후 선택한1시간 다운로드 window는 서로 다른 시간이다. 전체 restore/검증이 원래 lease를 넘을 가능성이 있으면 업로드 전 BUCKET_EXPIRY_UTC를 갱신한다. 승인 없으면 시작하지 않거나 원래deadline 전에 owner와 owned-key 삭제/반환을 조율한다. Restoring은 성공이 아니다.

## 실습구성

- 작업 목표: 작은 Archive 객체를 실제 restore 후 다운로드한다.
- 생성 owner: 52. 동일 owner의 자원을 재사용하며 실제 결과 필드는 실행 전 비워 둔다.


### 이번 workshop 시나리오와 인수·반환

같은 Standard bucket에 Archive key를 만들고 restore 후에만 다운로드한다.

- **신규/변경:** lab/archive.txt Archive object1개,restore1시간 window.
- **선행 재사용:** 48 bucket.
- **완주 흐름:** 승인/입력→본문의 생성/편집→Archived GET 거부→Restoring→Restored GET/cmp0→object삭제→48 반환;90일/restore-window 잔여 인계.
- **선택 이유/실패 경계:** 복원 최대1시간과 별도 다운로드1시간을 혼동하지 않는다.


- 입력/산출물: `NS`
  - 실제 값 출처: creator의 실제 생성/조회 출력; 임의 OCID/IP 금지
  - 준비 시점: 사용 전

- 입력/산출물: `BUCKET`
  - 실제 값 출처: creator의 실제 생성/조회 출력; 임의 OCID/IP 금지
  - 준비 시점: 사용 전

- 입력/산출물: `archive.txt`
  - 실제 값 출처: 본문 단계에서 승인/선택한 값
  - 준비 시점: 사용 전


입력별 실행위치는 본문 터미널 구분을 따른다: OCI CLI=인증 Cloud Shell/승인 OCI 터미널, OS 앱/NFS/부하=지정 guest, private HTTP/TCP=vm-a guest, YAML 작성/SSH private key=본인 local. 조회/변경/정리의 관리자 승인은 **실행 조건**이며 이 문서는 IAM 성공을 입증하지 않는다. 실제 허용/거부와 capacity는 사용자 실행에서 확인한다. 미승인/NOT_SELECTED/ASYNC_PENDING을 성공으로 바꾸지 않는다.

### 공유 bucket lease와 비동기 분기
owner48의 동일 bucket에는 단일 승인 `BUCKET_EXPIRY_UTC`/cleanup owner가 있다. 50의 하루 이상 관찰 또는52의 restore/download가 원래60분을 넘을 가능성이 있으면 **규칙/업로드 이전에** owner48과 budget 승인자가 새 종료시각/비용/담당자와 모든 선택 consumer를 기록하여 lease를 갱신한다. 갱신 승인 없으면 원래 deadline 전에 disable/cancel 가능한 시험을 중단하고 owned key를 삭제/반환한다. 실제 비동기 작업 취소가 지원되지 않으면 작업 상태를 기록하고 객체 삭제/목록 readback을 owner48과 조율한다. owner48은 expiry 때 독자 삭제하지 않고 모든 consumer의 abort/restore/return acknowledgement를 받는다. 연장 불가를 무기한 보존으로 대체하지 않는다. ASYNC_PENDING은 검증 성공이 아니다.

## 실습절차

### 작업 순서 체크포인트
새 Archive key 생성→HEAD/Archived GET refusal 확인→단일 Restore→실제 Restored GET/cmp→lease 안의 key cleanup을 수행한다. 대기는 완료 증거가 아니다. 단계 실패는 다음 생성으로 넘기지 않고 같은 target의 복구/반환으로 처리한다.

### 실행 터미널과 지역 고정

전용 Bash에서 `set -euo pipefail`을 먼저 사용한다. PUT/update/HEAD/GET/cmp의 예기치 않은 실패는 다음 checkpoint로 진행하지 않는다. 모든 다운로드는 존재하지 않는 전용 filename을 확인한 뒤 실행한다. 의도적 negative는 아래 if 또는 `set +e`의 captured exit로만 처리하고 복구는 별도 owner 경로로 수행한다.
아래 OCI 명령과 local bytes 작성/비교는 같은 인증 Cloud Shell/승인 OCI 터미널의 전용 directory에서 실행한다. VM·CIDR·앱 포트는 사용하지 않는다.
```bash
export REGION="<approved-region>"
export OCI_CLI_REGION="$REGION"
export LAB_COMPARTMENT_ID="<actual-lab-compartment-ocid>"
export EXPIRY_UTC="<approved-session-end-UTC>"
```
치환 전 실행 금지. 생성된 OCID/IP/원래 설정/새 rule ID/owner/종료시각을 비밀 없는 registry에 기록하고 새 터미널에서는 필요한 변수만 복원한다.

[아래 생성/변경 단계의 정확한 Oracle task 본문](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/managingobjects_topic-To_restore_objects_from_Archive_Storage.htm) — Console field/CLI/API 동작 근거.

아래 단계는 Oracle task 본문을 근거로 작성한 lab recipe이며 공식 실행결과가 아니다. 각 단계의 근거는 끝의 정확한 task 링크를 따른다.

### Source/download 작업 디렉터리 — 첫 printf/PUT 이전
- 실행 위치: Cloud Shell/승인 OCI 터미널의 **같은 shell**.
- 새 directory 생성 실패 시 중단한다.
- 이전 cwd에 있는 같은 이름의 파일은 읽거나 덮어쓰지 않는다.
```bash
set -euo pipefail
LAB_WORK=$(mktemp -d "$HOME/oci-roadmap-52.XXXXXXXX") || exit 1
cd -- "$LAB_WORK" || exit 1
for name in archive.txt archive-before.txt archive-before-error.txt restored-archive.txt; do
  if test -e "$name" || test -L "$name"; then printf 'STOP: local collision %s\n' "$name"; exit 1; fi
done
# Record actual LAB_WORK and file creation ownership in consumer registry.
```
- Remote PUT 전 owner48의 목록에서 같은 key의 기존 object를 확인한다. 충돌 시 중단한다.
- 새 PUT에는 `--no-overwrite`를 추가한다.
- 이후 출력 파일은 이 directory에서만 생성한다.


1. [Object Storage](00_Object_Storage.md#실습절차)의 private Standard bucket을 사용한다. Archive bucket을 별도 생성하지 않는다. 복구와 최소 보관 과금을 승인한다.

2. 같은 shell에서 아래 payload 생성과 Archive PUT을 실행한다.

   ```bash
   printf 'archive restore payload\n' > archive.txt
   oci os object put -ns "$NS" -bn "$BUCKET" --name lab/archive.txt --file archive.txt --storage-tier Archive --no-overwrite
   ```

3. **Restore 제출 전에** 아래 same-key Archived negative를 실행하고 HEAD 성공/실제 archive 상태와 GET 서비스 진단을 함께 확인한다.
```bash
oci os object head -ns "$NS" -bn "$BUCKET" --name lab/archive.txt
test ! -e archive-before.txt && test ! -e archive-before-error.txt || exit 1
if oci os object get -ns "$NS" -bn "$BUCKET" --name lab/archive.txt --file archive-before.txt 2> archive-before-error.txt; then
  printf 'not an Archived baseline; stop without restore resubmission\n'; exit 1
fi
# Do not accept a substring match: this file can mention lab/archive.txt in any error.
# Pause here. Inspect returned ServiceError status/code/message/request-id and record it.
# Require a same-key archive-state refusal paired with successful HEAD showing Archive/Archived.
# 401/auth, timeout, local output collision, missing object or unrelated error is INCONCLUSIVE.
```
실행자는 위 오류 진단을 실제 archived-state 서비스 거부라고 확인하기 전 Restore를 제출하지 않는다. 공식 task의 다운로드 제한을 근거로 사용하되 미확인 오류 code를 문서에서 invent하지 않는다. Archived GET 거부가 아닌 인증/transport/로컬파일오류면 inconclusive로 중단한다. 다운로드가 바로 가능하다고 가정하지 않는다. Console 객체 메뉴 Restore에서 Time available for download=1 hour(허용1–240, default24)을 입력한다. CLI 대안: `oci os object restore -ns "$NS" -bn "$BUCKET" --name lab/archive.txt --hours 1`.

4. `oci os object restore-status -ns "$NS" -bn "$BUCKET" --name lab/archive.txt`와 Details의 Archived→Restoring→Restored를 읽는다. 공식 복구 최대1시간이며 짧은세션 초과시 정리담당/expiry를 인계하고 완료를 주장하지 않는다. 복구 완료 전에는 성공으로 기록하지 않는다. Restored 상태일 때 `oci os object get -ns "$NS" -bn "$BUCKET" --name lab/archive.txt --file restored-archive.txt; cmp archive.txt restored-archive.txt`를 실행한다.

5. 다운로드 가능 기간 만료 후 Archive 상태로 돌아감을 관찰하는 것은 별도 비용 승인 선택 checkpoint이다.

### Restore 이후 관찰/반환
Restore는 위3에서 한 번만 제출한다. Restoring이면 재제출하지 않고 조회한다. 60분 초과 관찰은 위 bucket lease 갱신 승인이 필수이며, 미승인 시 deadline 전에 52/48이 중단/삭제/반환한다. download window 1시간은 복원 완료부터이고 90일 최소 비용은 삭제로 취소되지 않는다.


### 5분 간격 관찰과 재개
첫 PUT 직후 private 기록에 REGION/NS/BUCKET/bucket OCID/key/LAB_WORK/원본 hash/BUCKET_EXPIRY_UTC/cleanup 담당자를 저장한다. Restore 제출 성공 시 UTC와 hours=1을 추가한다. 응답 손실이면 실제 상태부터 조회하고 재제출하지 않는다. fresh shell은 이 기록의 실제 변수를 다시 설정하고 같은 Region/bucket OCID를 readback한 뒤 restore-status를 한 번 조회한다.


- **상태: Restoring**
  - 다음 행동: 다음 확인을 **현재 UTC+5분**으로 기록하고 그때 한 번 조회; 자동 tight loop/Restore 재제출 금지
  - 종료 조건: lease 10분 전까지 Restored가 아니면 owner48과 abort/owned-key 삭제·반환; 연장은 별도 사전 승인

- **상태: Restored**
  - 다음 행동: 실제 restore expiration time을 object details/HEAD에서 읽고 즉시 새 filename GET/cmp
  - 종료 조건: download deadline은 `min(BUCKET_EXPIRY_UTC, 실제 expiration time)-10분`; 종료 전에 key 정리

- **상태: Archived(미제출/만료)**
  - 다음 행동: 제출 기록·서비스 상태를 대조한다. 미제출이 확실한 최초3단계 외에는 새 Restore를 제출하지 않는다
  - 종료 조건: 만료/불명 상태는 미완료로 반환하고 owned key 정리

- **상태: 조회 실패/다른 상태**
  - 다음 행동: credential/Region/bucket을 고정하고 담당자에 인계; 상태를 추측하지 않는다
  - 종료 조건: deadline에 정리/반환, 무기한 대기하지 않는다


예: lease 종료15:00Z, 실제 restore expiration14:40Z이면 다운로드/정리 deadline은14:30Z다. Restored를14:05Z에 관찰했다면 관찰 시각+1시간을 만료라고 가정하지 않는다. 실제 expiration을 읽지 못하면 다운로드 창을 확정하지 말고 즉시 GET/cmp 또는 owner와 abort한다. 5분 주기는 이 작은 실습의 요청 상한을 위한 선택이며 Oracle 보장 주기가 아니다.

## 결과검증

Archive tier, restore 진행/완료 상태, cmp 0을 순서대로 확인한다. 복구는 즉시 완료를 보장하지 않는다.

- 예상 pattern이며 실제 OCI 실행 결과가 아니다. timestamp, resource OCID, 설정 readback, 실제 HTTP/TCP/파일 결과를 실행자가 기록한다.

## 문제해결

Archived GET 실패는 restore 상태와 구분하고 OBJECT_RESTORE 권한을 확인한다. Restoring이면 같은 restore를 중복 제출하지 않는다. 1시간 복구 window 초과 시 서비스 오류를 기록하고 owner에게 인계한다.

실패 기록에는 같은 target/credentials/region/요청 시간을 고정한다. 복구는 원래 설정으로 실행하고 actual response/readback을 둘 다 확인한다. 예상값을 실측으로 쓰지 않는다.

## 리소스정리

먼저 owner52 PUT 성공/etag와 exact key를 대조하고 `oci os object delete -ns "$NS" -bn "$BUCKET" --name lab/archive.txt`를 수행한다. 성공한 object list에서 exact key 부재를 확인하고 owner48에게 반환한다. HEAD 실패만으로 부재를 단정하지 않는다. Restore 진행 중에도 삭제/조회가 실패하면 원본/evidence를 보존하고 owner48과 expiry 전에 잔여 정리를 조율한다.90일 최소기간/조기삭제 비용은 취소되지 않는다. remote 정리/반환 후에만 아래 local cleanup을 수행한다.

```bash
cd -- "$LAB_WORK" || exit 1
for name in archive.txt archive-before.txt archive-before-error.txt restored-archive.txt; do
  test ! -L "$name" || exit 1
  if test -e "$name"; then test -f "$name" || exit 1; rm -- "$name" || exit 1; fi
done
cd "$HOME" || exit 1
rmdir -- "$LAB_WORK" || exit 1
```
예상 Archive GET 거부로 archive-before.txt가 없으면 건너뛴다. nonempty directory는 조사하며 recursive/glob 삭제하지 않는다.

정리 UTC/namespace/bucket/owned key·목록 readback과 owner48 반환을 기록한다. shared bucket은 consumer가 삭제하지 않는다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 목표는 학습 후 설명·구분·확인할 내용이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [36] [Object Storage Storage Tiers — Archive](https://docs.oracle.com/en-us/iaas/Content/Object/Concepts/understandingstoragetiers.htm)
  - 확인 범위: Archive The / Objects in the Archive tier must be restored.

### 절차별 공식 task body 추가 근거
- [understandingstoragetiers.htm](https://docs.oracle.com/en-us/iaas/Content/Object/Concepts/understandingstoragetiers.htm) — 본문 검토 근거; 아래 정확한 task와 작성한 입력을 구분한다. OCI 실행 증거가 아니다.



### 이번 절차의 정확한 공식 task
- [managingobjects_topic-To_restore_objects_from_Archive_Storage.htm](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/managingobjects_topic-To_restore_objects_from_Archive_Storage.htm) — 이 문서 실습절차의 입력/동작을 대조한 본문.
- [managingobjects_topic-To_check_the_status_of_an_Archive_Storage_object_restoration.htm](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/managingobjects_topic-To_check_the_status_of_an_Archive_Storage_object_restoration.htm) — 이 문서 실습절차의 입력/동작을 대조한 본문.
