# Object Lifecycle Management

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../README.md)를 확인하세요.

> 절차 작성본 / OCI 미실행 — 아래 예상값은 실측 결과가 아닙니다. 비용·권한·사전준비 gate를 통과한 전용 환경에서만 실행하세요.

## 학습목표

- 기존 lifecycle rules를 먼저 저장하고 lab/expire/ prefix의 1-day DELETE rule 하나를 생성/readback한다.
- config-only는 즉시 disable/삭제하고 비동기 실제 삭제는 사전 갱신 lease가 승인된 분기에서만 관찰하며 pending과 완료를 구분한다.

완료 기준은 아래 실제 생성·설정·요청 결과로 판정한다. 이 목표는 수행할 작업이며 OCI에서 관찰한 결과가 아니다.

## 대응 Roadmap 항목

- 항목명: **Object Lifecycle Management**
- Roadmap 경로: **Object Storage → Object Lifecycle Management**
- Node ID: `sequence-6-2`
- 상위 항목: **Object Storage** (`sequence-6-0`)
- 문서 범위: 이 항목의 개념과 작성된 실습절차. 통합 품질검토는 BLOCKED이며 실제 실행은 NOT_EXECUTED입니다.
- [전체 항목 목록](../README.md)

## 사전준비


48의 REGION/COMPARTMENT_OCID/NS/BUCKET/actual bucket id와 단일 BUCKET_EXPIRY_UTC를 받는다. 기존 lifecycle rules/정책 없음 상태 및 관리자 region service principal statement 승인을 먼저 보관한다. config-only와 lease-renewed 실제1일 관찰을 구분한다; 후자 승인 없으면 rule 저장 검증 후 disable/delete한다.

- [Object Storage](00_Object_Storage.md#실습절차) — 링크의 `실습절차`와 `결과검증` 완료; 실제 OCID/IP를 registry에 인계한다.



- 변수 handoff: 48의 `REGION`, `COMPARTMENT_OCID`, `NS`, `BUCKET`; bucket private/Standard/versioning disabled/auto-tiering disabled 확인. 후속에서 새 bucket 생성 금지.

## 권한

GetBucket → BUCKET_READ; ListObjects → OBJECT_INSPECT; GetObject/HeadObject → OBJECT_READ; 새 객체 PutObject → OBJECT_CREATE, 동일 key overwrite는 OBJECT_OVERWRITE 추가; DeleteObject → OBJECT_DELETE. GetObjectLifecyclePolicy → BUCKET_READ; PutObjectLifecyclePolicy/DeleteObjectLifecyclePolicy → BUCKET_UPDATE; 실제 Delete action은 service principal의 OBJECT_DELETE 필요.

[API operation별 공식 permission table](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm) — 가족 단위 manage를 최소권한이라 부르지 않는다. 전용 lab compartment의 group만 적용하고 정책작성 권한은 관리자 소유다.
```text
Allow group <identity-domain>/<lab-group> to manage object-family in compartment <lab-compartment> where any {request.permission='BUCKET_INSPECT', request.permission='BUCKET_READ', request.permission='OBJECT_INSPECT', request.permission='OBJECT_READ', request.permission='OBJECT_CREATE', request.permission='OBJECT_DELETE', request.permission='BUCKET_UPDATE'}
```
이 allowlist는 위 작업만 허용한다. Console 탐색 List API가 거부되면 실제 API permission을 공식 표에서 별도로 승인하며 all-resources로 확대하지 않는다.

## 비용및주의사항

owner48 Standard bucket에 lab/expire/test.txt와 prefix 제한 DELETE rule 하나만 추가한다. IA/Archive 전환을 하지 않으므로 해당 tier의 최소기간 비용을 이 실습 비용으로 넣지 않는다. rule 저장/readback만 선택하면 원래60분 안에 disable/delete와 key 삭제를 끝낸다. 실제1일 삭제 관찰은 그 전에 owner48 단일 lease/저장·요청 비용/담당자를 별도 갱신한 경우만 선택한다. ASYNC_PENDING은 완료가 아니며 service principal policy는 관리자 소유 변경분만 반환한다.

## 실습구성

- 작업 목표: 실제 lifecycle rule을 저장하고 전용 prefix에 제한한다.
- 생성 owner: 50. 동일 owner의 자원을 재사용하며 실제 결과 필드는 실행 전 비워 둔다.


### 이번 workshop 시나리오와 인수·반환

전용 prefix에만 1일 삭제 규칙을 저장하고 짧은 세션의 비동기 한계를 확인한다.

- **신규/변경:** lab-prefix-expiry rule1개,lab/expire/test.txt.
- **선행 재사용:** 48 bucket,관리자 승인 region service principal.
- **완주 흐름:** 승인/입력→본문의 생성/편집→prefix/action/age/enabled readback; 실제삭제 별도 ASYNC_PENDING→실습rule disable/delete→key삭제→관리자 principal statement 반환.
- **선택 이유/실패 경계:** 60분 내 하루 규칙 실행을 PASS로 바꾸지 않는다.

| 입력/산출물 | 실제 값 출처 | 준비 시점 |
|---|---|---|
| `NS` | creator의 실제 생성/조회 출력; 임의 OCID/IP 금지 | 사용 전 |
| `BUCKET` | creator의 실제 생성/조회 출력; 임의 OCID/IP 금지 | 사용 전 |
| `REGION` | 본문 단계에서 승인/선택한 값 | 사용 전 |

입력별 실행위치는 본문 터미널 구분을 따른다: OCI CLI=인증 Cloud Shell/승인 OCI 터미널, OS 앱/NFS/부하=지정 guest, private HTTP/TCP=vm-a guest, YAML 작성/SSH private key=본인 local. 조회/변경/정리의 관리자 승인은 **실행 조건**이며 이 문서는 IAM 성공을 입증하지 않는다. 실제 허용/거부와 capacity는 사용자 실행에서 확인한다. 미승인/NOT_SELECTED/ASYNC_PENDING을 성공으로 바꾸지 않는다.

### 공유 bucket lease와 비동기 분기
owner48의 동일 bucket에는 단일 승인 `BUCKET_EXPIRY_UTC`/cleanup owner가 있다. 50의 하루 이상 관찰 또는52의 restore/download가 원래60분을 넘을 가능성이 있으면 **규칙/업로드 이전에** owner48과 budget 승인자가 새 종료시각/비용/담당자와 모든 선택 consumer를 기록하여 lease를 갱신한다. 갱신 승인 없으면 원래deadline 전에 disable/cancel 가능한 시험을중단하고 owned key를삭제/반환한다; 실제비동기 작업취소가 지원되지않으면작업상태를기록하고 객체삭제/목록readback을owner48과조율한다. owner48은 expiry때 독자삭제하지않고 모든consumer의abort/restore/return acknowledgement를받는다. 연장안됨을단순히무기한보존으로대체하지않는다. ASYNC_PENDING은검증성공아니다.

## 실습절차

### 작업 순서 체크포인트
정책 입력과 서비스권한 승인→새 source/PUT→정책 readback→config-only disable 또는 lease-renewed observation→소유 rule/key cleanup 순서다. 단계 실패는 다음 생성으로 넘기지 않고 같은 target의 복구/반환으로 처리한다.

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

[아래생성/변경단계의정확한Oracle task본문](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/usinglifecyclepolicies_topic-To_create_a_lifecycle_policy_rule.htm) — Consolefield/CLI/API동작근거.

아래 단계는 Oracle task 본문을 근거로 작성한 lab recipe이며 공식 실행결과가 아니다. 각 단계의 근거는 끝의 정확한 task 링크를 따른다.

### Source/download 작업 디렉터리 — 첫 printf/PUT 이전
Cloud Shell/승인 OCI 터미널의 **같은 shell**에서 실행한다. 새 directory 생성 실패면 진행하지 않는다. 이전 cwd의 같은 파일을 읽거나 덮어쓰지 않는다.
```bash
set -euo pipefail
LAB_WORK=$(mktemp -d "$HOME/oci-roadmap-50.XXXXXXXX") || exit 1
cd -- "$LAB_WORK" || exit 1
for name in expiry.txt; do
  if test -e "$name" || test -L "$name"; then printf 'STOP: local collision %s\n' "$name"; exit 1; fi
done
# Record actual LAB_WORK and file creation ownership in consumer registry.
```
Remote PUT도 같은 key의 기존 object 충돌을 owner48 목록에서 확인하고 충돌이면 중단한다. 새 PUT은 `--no-overwrite`를 추가한다. 다음 절차의 출력 파일은 이 directory에서만 생성한다.


1. [Object Storage](00_Object_Storage.md)의 bucket/객체 업로드 완료 단계와 실제 NS/BUCKET을 확보한다.
2. **먼저** Console → exact bucket → Lifecycle Policy에서 기존 전체 rules 또는 정책 없음 상태를 저장하고, 정책이 있을 때 `oci os object-lifecycle-policy get -ns "$NS" -bn "$BUCKET"`도 저장한다. 없을 때의 Get 정책 없음 응답은 이 preflight의 예상 분기이며 fail-fast shell에서 그 실패 뒤 다음 명령을 억지 실행하지 않는다; 새 shell에서 확인한 없음 상태와 승인 후 Create를 진행한다. lab-prefix-expiry 이름 충돌을 확인한다. 충돌이면 중단한다. `LIFECYCLE_BRANCH=config-only`(기본) 또는 승인된 renewed bucket lease의 `extended-observation`을 선택/등록한다. 그 다음 관리자에게 root compartment 정책 `Allow service objectstorage-<region_identifier> to manage object-family in compartment <lab-compartment> where any {request.permission='BUCKET_INSPECT', request.permission='BUCKET_READ', request.permission='OBJECT_INSPECT', request.permission='OBJECT_DELETE'}`를 승인받는다. 실제 region identifier 사용, 타 region principal 금지. 이 delete-only 규칙은 versioning disabled를 전제한다. 이후 Console → bucket → Lifecycle Policy → Create Rule에서 이름 `lab-prefix-expiry`, 대상 Objects, prefix `lab/expire/`, action Delete, age 1 day, Enabled를 설정한다. 전체 bucket 또는 빈 prefix는 선택하지 않는다. 승인하지 않은 key를 삭제할 수 있으므로 입력을 다시 확인한다.
3. `printf 'expiry test\n' > expiry.txt` 후 `oci os object put -ns "$NS" -bn "$BUCKET" --name lab/expire/test.txt --file expiry.txt --no-overwrite`를 실행한다.
4. `oci os object-lifecycle-policy get -ns "$NS" -bn "$BUCKET"`로 저장된 prefix/action/age/is-enabled를 확인한다. 정책 실행은 하루 단위, 변경 반영/첫 실행 최대24시간과 이후 처리시간이 필요하다. 60분 세션에서 삭제까지 합격할 수 없으며 `ASYNC_PENDING`으로 분리한다. Lifecycle은 비동기이며 즉시 삭제 테스트로 합격 처리하지 않는다. 관찰 대기 비용 승인이 없다면 정책 저장 검증 후 비활성화한다.
5. 처리 실제 검증이 필요한 경우 승인된 관찰 시각까지 객체 목록을 재조회하여 삭제 여부를 기록한다. 미관찰은 pending이지 성공이 아니다.

### 비동기 관찰과 실패 복구의 분리
Console Lifecycle policy에서 rule `lab-prefix-expiry`의 prefix=lab/expire/, timeAmount=1/timeUnit=DAYS/action=DELETE/isEnabled=true를 읽는다. 삭제시험은 공식비동기시간 때문에 짧은세션에서 `ASYNC_PENDING`이다. 설정실패에는 잘못된 rule/prefix를 저장하지 말고 즉시 취소한다. config-only는 정상 readback 직후 Enabled=false로 편집/조회한다. extended-observation은 renewed lease 안에서 실제 비동기 결과를 관찰할 때만 enabled를 유지하고 결과/실패 또는 새deadline 뒤 disable/delete한다. 두분기disable시점을혼합하지않는다. 기존 policy 전체를 CLI delete하는 방식은 다른 rule이 있을 때 사용하지 않는다. policy 추가 이전값을 저장하고 실습 rule만 Console에서 Delete한다. 관리자만 이번실습 service principal policy statement를 제거하며 다른 lifecycle consumer가 있으면 보존한다.

## 결과검증

정확한 prefix와 1 day rule 저장을 기대한다. 실제 삭제는 별도 checkpoint이며 실행 시간을 날조하지 않는다.

- 예상 pattern이며 실제 OCI 실행 결과가 아니다. timestamp, resource OCID, 설정 readback, 실제 HTTP/TCP/파일 결과를 실행자가 기록한다.

## 문제해결

policy저장과서비스실행은별개. serviceprincipalregion/BUCKETREAD/INSPECT/OBJECTDELETE없으면관리자가수정;24시간전미삭제를장애라고단정하지않는다.

실패기록에는같은target/credentials/region/요청시간을고정한다. 복구는원래설정으로실행하고actualresponse/readback둘다확인한다;예상값을실측으로쓰지않는다.

## 리소스정리

로컬은 registry의 이 실행 LAB_WORK 안에 생성한 expiry.txt만 확인 후 `rm -- expiry.txt`로 제거한다(생성되지 않은 파일은 건너뛴다). 먼저 증거를 저장하고 `cd "$HOME"` 후 exact LAB_WORK에 `rmdir -- "$LAB_WORK"`를 실행한다. nonempty이면 추가 파일을 조사하고 recursive/glob 삭제하지 않는다.

Consolelab-prefix-expiry만disable/delete→`oci os object-lifecycle-policy get -ns "$NS" -bn "$BUCKET"`로rule없음(마지막rule이면policy없음)확인. `oci os object delete -ns "$NS" -bn "$BUCKET" --name lab/expire/test.txt`는이미비동기삭제되었으면추가실행하지않는다.

정리실행자의실제UTC/OCID/readback을기록한다. 공유resource는owner확인전삭제금지; stop만으로bootvolume/LB/image비용종료가아니다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 목표는 학습 후 설명·구분·확인할 내용이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [37] [Object Storage Object Lifecycle Management — Object Lifecycle Management](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/usinglifecyclepolicies.htm)
  - 확인 범위: Object Lifecycle Management works.

### 절차별 공식 task body 추가 근거
- [usinglifecyclepolicies.htm](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/usinglifecyclepolicies.htm) — 본문 검토 근거; 아래 정확한 task와 작성한 입력을 구분한다. OCI 실행 증거가 아니다.



### 이번 절차의 정확한 공식 task
- [usinglifecyclepolicies_topic-To_create_a_lifecycle_policy_rule.htm](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/usinglifecyclepolicies_topic-To_create_a_lifecycle_policy_rule.htm) — 이 문서 실습절차의 입력/동작을 대조한 본문.
- [usinglifecyclepolicies.htm](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/usinglifecyclepolicies.htm) — 이 문서 실습절차의 입력/동작을 대조한 본문.
