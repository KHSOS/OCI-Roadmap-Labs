> **문서 검토 완료 / OCI 실행 미검증**: 이 파일의 제한된 lifecycle 설정·복구·정리 시나리오를 문서 수준에서 검토했습니다. 실제 OCI 실행이나 전체 교육과정의 승인을 뜻하지 않습니다.

# Object Lifecycle Management

> 이 파일의 문서 수용 범위: **제한된 시나리오의 절차·안전 조건 검토 완료**. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 아래 권한·비용·복구·공유 bucket 반환 조건은 실행 전에 충족해야 합니다. 전체 교육과정 상태와 게시 여부는 이 파일의 문서 검토와 별개입니다.

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
- 문서 범위: 이 항목의 개념과 제한된 실습절차는 문서 검토를 완료했습니다. 실제 OCI 실행은 NOT_EXECUTED이며 실행 전 권한·비용·복구 조건은 유지됩니다.
- [전체 항목 목록](../../../Docs/README.md)

## 사전준비


48의 REGION/COMPARTMENT_OCID/NS/BUCKET/actual bucket id와 단일 BUCKET_EXPIRY_UTC를 받는다. 기존 lifecycle rules/정책 없음 상태 및 관리자 region service principal statement 승인을 먼저 보관한다. config-only와 lease-renewed 실제1일 관찰을 구분한다; 후자 승인 없으면 rule 저장 검증 후 disable/delete한다.

- [Object Storage](00_Object_Storage.md#실습절차) — 링크의 `실습절차`와 `결과검증` 완료; 실제 bucket OCID/NS/BUCKET/REGION과 승인된 `BUCKET_EXPIRY_UTC`를 인계한다. VM/IP는 필요하지 않다.



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

이 문서의 모든 CLI/파일 작업은 같은 인증 Cloud Shell/승인 OCI 터미널에서 수행한다. VM/SSH/NFS는 필요하지 않다. 조회/변경/정리의 관리자 승인은 **실행 조건**이며 이 문서는 IAM 성공을 입증하지 않는다. 실제 허용/거부와 capacity는 사용자 실행에서 확인한다. 미승인/NOT_SELECTED/ASYNC_PENDING을 성공으로 바꾸지 않는다.

### 공유 bucket lease와 비동기 분기
owner48의 동일 bucket에는 단일 승인 `BUCKET_EXPIRY_UTC`/cleanup owner가 있다. 50의 하루 이상 관찰 또는52의 restore/download가 원래60분을 넘을 가능성이 있으면 **규칙/업로드 이전에** owner48과 budget 승인자가 새 종료시각/비용/담당자와 모든 선택 consumer를 기록하여 lease를 갱신한다. 갱신 승인 없으면 원래deadline 전에 disable/cancel 가능한 시험을중단하고 owned key를삭제/반환한다; 실제비동기 작업취소가 지원되지않으면작업상태를기록하고 객체삭제/목록readback을owner48과조율한다. owner48은 expiry때 독자삭제하지않고 모든consumer의abort/restore/return acknowledgement를받는다. 연장안됨을단순히무기한보존으로대체하지않는다. ASYNC_PENDING은검증성공아니다.

## 실습절차

### 작업 순서 체크포인트
정책 입력과 서비스권한 승인→새 source/PUT→정책 readback→config-only disable 또는 lease-renewed observation→소유 rule/key cleanup 순서다. 단계 실패는 다음 생성으로 넘기지 않고 같은 target의 복구/반환으로 처리한다.

### 실행 터미널과 지역 고정

전용 Bash에서 `set -euo pipefail`을 먼저 사용한다. PUT/update/HEAD/GET/cmp의 예기치 않은 실패는 다음 checkpoint로 진행하지 않는다. 모든 다운로드는 존재하지 않는 전용 filename을 확인한 뒤 실행한다. 필터 음성 대조는 아래 문자열 비교이며 OCI 오류를 의도적으로 발생시키지 않는다. 복구는 아래 독립 Console 경로로 수행한다.
아래 OCI 명령과 local bytes 작성/비교는 같은 인증 Cloud Shell/승인 OCI 터미널의 전용 directory에서 실행한다. VM·CIDR·앱 포트는 사용하지 않는다.
```bash
export REGION="<approved-region>"
export OCI_CLI_REGION="$REGION"
export NS="<owner48-actual-namespace>"
export BUCKET="<owner48-actual-bucket-name>"
export LAB_COMPARTMENT_ID="<actual-lab-compartment-ocid>"
export BUCKET_EXPIRY_UTC="<owner48-approved-session-end-UTC>"
export EXPIRY_UTC="$BUCKET_EXPIRY_UTC"
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


### rule 저장 전 복구와 fresh-terminal 재개
private 기록에 actual REGION/NS/BUCKET/bucket OCID/LAB_WORK, rule name=`lab-prefix-expiry`, 원래 전체 rules 또는 NONE, 추가 service policy OCID/statement, cleanup 담당자와 UTC deadline을 저장한다. 별도 인증 Console을 exact bucket의 **Policies → Lifecycle policy rules**에 열어 둔다. 개인 실습은 본인의 두 번째 세션을 사용한다. 잘못된 prefix 저장, PUT/Get 실패 또는 terminal loss이면 요청 shell 종료와 무관하게 다음을 즉시 수행한다.

1. 기록한 Region/Compartment/bucket OCID를 Console에서 다시 대조한다. exact rule의 Actions → **Disable**을 선택한다. 이미 Disabled이면 재변경하지 않는다. 새 목록의 State=Disabled를 확인한다. rule이 없으면 목록 정상 조회를 확인하고 다음으로 간다. 다른 rules는 변경하지 않는다.
2. rule details의 prefix/action/age를 기록하고 owner48에게 잘못된 prefix가 노출한 key 범위를 알린다. 삭제된 데이터가 자동 복구된다고 가정하지 않는다. 해당 rule을 Delete하고 새 목록의 부재를 확인한다. CLI 전체 policy delete는 사용하지 않는다.
3. fresh shell은 private 기록의 REGION/OCI_CLI_REGION/NS/BUCKET/LAB_WORK를 다시 export하고 actual bucket OCID를 readback한다. `cd -- "$LAB_WORK"` 후 이번 owned lab/expire/test.txt를 목록으로 확인해 존재할 때만 삭제/readback하고 owner48에게 반환한다. local expiry.txt는 remote rule/key 반환 후에만 삭제한다. 원래 전체 rules/상태가 그대로인지 확인하고 이번 service statement만 관리자가 반환한다.

복구 Console의 권한/담당자 확인이 없으면 활성 rule을 저장하지 않는다. config-only는 단계4의 Enabled readback 직후 위 Disable/새 목록 확인을 수행한다.


1. [Object Storage](00_Object_Storage.md#실습절차)의 bucket/객체 업로드 완료 단계와 실제 NS/BUCKET을 확보한다.
2. **먼저** Console → exact bucket → Lifecycle Policy에서 기존 전체 rules 또는 정책 없음 상태를 저장하고, 정책이 있을 때 `oci os object-lifecycle-policy get -ns "$NS" -bn "$BUCKET"`도 저장한다. Console이 정책 없음/빈 규칙 목록을 보여주면 UTC·bucket OCID와 NONE 상태를 private 기록에 저장하고 Get을 호출하지 않은 채 같은 shell을 유지한다. Get 오류를 정책 없음으로 취급하지 않는다. 기존 정책이 있으면 Get 성공/전체 items 저장이 필수이며 오류면 시작하지 않는다. lab-prefix-expiry 이름 충돌을 확인한다. 충돌이면 중단한다. `LIFECYCLE_BRANCH=config-only`(기본) 또는 승인된 renewed bucket lease의 `extended-observation`을 선택/등록한다. 그 다음 관리자에게 root compartment 정책 `Allow service objectstorage-<region_identifier> to manage object-family in compartment <lab-compartment> where any {request.permission='BUCKET_INSPECT', request.permission='BUCKET_READ', request.permission='OBJECT_INSPECT', request.permission='OBJECT_DELETE'}`를 승인받는다. 실제 region identifier 사용, 타 region principal 금지. 이 delete-only 규칙은 versioning disabled를 전제한다. 이후 위 `rule 저장 전 복구와 fresh-terminal 재개` station을 준비한 뒤 Console → bucket → Policies → Lifecycle policy rules → Create Rule에서 이름 `lab-prefix-expiry`, 대상 Objects, prefix `lab/expire/`, action Delete, Number of days=`1`, State=`Disabled`로 먼저 생성한다. Add filter → Filter type=`Prefix` → Filter value=`lab/expire/` 하나만 추가하며 pattern inclusion/exclusion은 추가하지 않는다. 저장 후 rule details와 Get에서 inclusion prefix가 정확히 한 개이고 target=objects/action=DELETE/timeAmount=1/timeUnit=DAYS/isEnabled=false인지 확인한다. 다른 값 또는 조회 실패면 위 복구 station으로 이동한다. 같은 이름의 기존 rule을 인수하지 않는다. 전체 bucket 또는 빈 prefix는 선택하지 않는다. 승인하지 않은 key를 삭제할 수 있으므로 입력을 다시 확인한다.
3. 활성화 전에 Console Objects에서 `lab/expire/`로 필터해 전체 목록(다음 페이지 포함)을 조회한다. 기존 key가 하나라도 있으면 중단하고 이번 Disabled rule만 삭제한다. 다른 consumer가 이 prefix에 쓰지 않는다는 owner48 확인도 받는다. `printf 'expiry test\n' > expiry.txt` 후 `oci os object put -ns "$NS" -bn "$BUCKET" --name lab/expire/test.txt --file expiry.txt --no-overwrite`를 실행한다.
4. PUT 성공과 exact key 목록을 확인한 뒤 rule Actions → Enable을 선택한다. Enabled 상태를 확인하고 `oci os object-lifecycle-policy get -ns "$NS" -bn "$BUCKET"`로 저장된 prefix/action/age/is-enabled를 확인한다. age=1일을 만족한 뒤에도 비동기 처리이며 일반적으로 trigger 후10분 내 실행되지만 best effort로24시간 이상 걸릴 수 있다. 24시간을 보장된 상한으로 쓰지 않는다. 60분 세션에서 삭제까지 합격할 수 없으며 `ASYNC_PENDING`으로 분리한다. Lifecycle은 비동기이며 즉시 삭제 테스트로 합격 처리하지 않는다. 관찰 대기 비용 승인이 없다면 정책 저장 검증 후 비활성화한다.
5. 처리 실제 검증이 필요한 경우 승인된 관찰 시각까지 객체 목록을 재조회하여 삭제 여부를 기록한다. 미관찰은 pending이지 성공이 아니다.

### 필터 음성 대조와 복구 확인
활성화 전 rule details에서 prefix `lab/expire/`가 `lab/expire/test.txt`에는 일치하고 `lab/keep/test.txt`에는 일치하지 않음을 문자열로 대조한다. 후자는 생성하지 않는다. 이 대조는 필터 설정 검증이지 서비스 삭제 실행의 증거가 아니다. config-only는 Enable/readback 직후 Disable하고 Get/Console에서 isEnabled=false 및 나머지 rules 불변을 확인한다. Disabled 이후 짧은 시간 객체가 남아 있다는 사실만으로 비동기 실행 중단을 입증하지 않는다. extended-observation은 기존 rule 영향이 없고 관찰기간 동안 수동 DeleteObject가 없었던 경우에만 실제 삭제를 lifecycle 결과로 기록한다.

### 비동기 관찰과 실패 복구의 분리
오류/세션 손실은 위 rule 저장 전 복구 station으로 즉시 이동한다.
Console Lifecycle policy에서 rule `lab-prefix-expiry`의 prefix=lab/expire/, timeAmount=1/timeUnit=DAYS/action=DELETE/isEnabled=true를 읽는다. 삭제시험은 공식비동기시간 때문에 짧은세션에서 `ASYNC_PENDING`이다. 설정실패에는 잘못된 rule/prefix를 저장하지 말고 즉시 취소한다. config-only는 정상 readback 직후 Enabled=false로 편집/조회한다. extended-observation은 renewed lease 안에서 실제 비동기 결과를 관찰할 때만 enabled를 유지하고 결과/실패 또는 새deadline 뒤 disable/delete한다. 두분기disable시점을혼합하지않는다. 기존 policy 전체를 CLI delete하는 방식은 다른 rule이 있을 때 사용하지 않는다. policy 추가 이전값을 저장하고 실습 rule만 Console에서 Delete한다. 관리자만 이번실습 service principal policy statement를 제거하며 다른 lifecycle consumer가 있으면 보존한다.

## 결과검증

정확한 prefix와 1 day rule 저장을 기대한다. 실제 삭제는 별도 checkpoint이며 실행 시간을 날조하지 않는다.

- 예상 pattern이며 실제 OCI 실행 결과가 아니다. timestamp, resource OCID, 설정 readback, 실제 HTTP/TCP/파일 결과를 실행자가 기록한다.

## 문제해결

policy저장과서비스실행은별개. serviceprincipalregion/BUCKETREAD/INSPECT/OBJECTDELETE없으면관리자가수정;24시간전미삭제를장애라고단정하지않는다. 실행은 하루 단위이고 변경 반영/첫 실행 시작에 최대 24시간이 걸릴 수 있으며 UTC last-modified의 일 단위 올림과 처리 지연을 고려한다. 승인 deadline 도달 시 추가 대기 대신 복구 station으로 반환한다.

실패기록에는같은target/credentials/region/요청시간을고정한다. 복구는원래설정으로실행하고actualresponse/readback둘다확인한다;예상값을실측으로쓰지않는다.

## 리소스정리

먼저 위 복구 station으로 이번 rule을 disable/delete하고 원래 rules를 보존한다. exact owned key는 목록에서 존재할 때만 삭제하고 목록 readback으로 부재를 확인한다. 이미 비동기 삭제됐으면 반복 삭제하지 않는다. owner48 반환과 service statement 반환 확인 뒤 LAB_WORK의 이번 regular/non-symlink expiry.txt만 존재할 때 제거하고, directory 밖에서 `rmdir -- "$LAB_WORK"`한다. local diagnostic/원래 rules 기록은 증거 보존 종료 후 exact 소유 파일만 정리하며 glob/recursive 삭제는 하지 않는다.

정리실행자의실제UTC/OCID/readback을기록한다. 공유 bucket은 owner48 확인 전 삭제하지 않는다. 이 consumer는 bucket을 삭제하지 않고 rule/key 반환만 수행한다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 목표는 학습 후 설명·구분·확인할 내용이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [37] [Object Storage Object Lifecycle Management — Object Lifecycle Management](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/usinglifecyclepolicies.htm)
  - 확인 범위: Object Lifecycle Management works.

### 절차별 공식 task body 추가 근거
- [usinglifecyclepolicies.htm](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/usinglifecyclepolicies.htm) — 본문 검토 근거; 아래 정확한 task와 작성한 입력을 구분한다. OCI 실행 증거가 아니다.



### 이번 절차의 정확한 공식 task
- [usinglifecyclepolicies_topic-To_create_a_lifecycle_policy_rule.htm](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/usinglifecyclepolicies_topic-To_create_a_lifecycle_policy_rule.htm) — 이 문서 실습절차의 입력/동작을 대조한 본문.
- [usinglifecyclepolicies.htm](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/usinglifecyclepolicies.htm) — 이 문서 실습절차의 입력/동작을 대조한 본문.
