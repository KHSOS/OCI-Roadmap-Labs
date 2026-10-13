# Policy Scope / Inheritance

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../README.md)를 확인하세요.

> **저술 완료 · OCI 실행 미검증** — Oracle 공식 작업 문서를 근거로 작성한 한국어 hands-on입니다. Oracle 공인 교육/실행 인증이 아니며 기대 결과와 실제 결과를 구분합니다. 비용 승인과 실제 선행 산출물 확인 후에만 실행하세요.

## 학습목표

- 기존 parent/child/sibling proof.txt를 같은 outsider로 읽고 parent-scope policy의 세 영역 허용을 기록한다.
- test parent grant 삭제 후 세 거부를 확인하고 child-only policy에서 child 허용/나머지 거부를 대조한다.

완료 기준은 아래 실제 생성·설정·요청 결과로 판정한다. 이 목표는 수행할 작업이며 OCI에서 관찰한 결과가 아니다.

## 대응 Roadmap 항목

- 항목명: **Policy Scope / Inheritance**
- Roadmap 경로: **Identity and Access Management (IAM) → Policy Scope / Inheritance**
- Node ID: `sequence-4-8`
- 상위 항목: **Identity and Access Management (IAM)** (`sequence-4-0`)
- 문서 범위: 공식 개념 목표와 실제 구성·허용/불허 검증·정리 절차. 아래 실습은 저술되었으나 OCI에서 실행 검증되지 않았습니다.
- [전체 항목 목록](../README.md)

## 사전준비

38의 child/sibling OCID·bucket name·proof bytes/hash,33의 parent/A fixture와34 outsider 인증을 받는다.37/40의 test policy/membership 정리·그룹 없는 baseline 반환이 먼저다. 세 영역은 동일 REGION/NS에서 bucket만 바꾸며 새 fixture는 만들지 않는다. child/parent creator lease를 각각 확인한다.

### 기준 계약과 실제 단계 선행조건

실제 tenancy/Region/Domain·주체·소유권은 [공통 baseline-contract.json](../../baseline-contract.json)과 위 fixture 계약에 맞춘다. 빈 OCID/placeholder·미승인 권한은 실행 차단 입력이다. 이 API/Console 실습 때문에 VM을 생성하지 않는다.

- [38.ready Compartments — FUNCTIONAL_COMPLETION](05_Compartments.md#baseline-38-ready): 표시된 FUNCTIONAL_COMPLETION은 해당 linked ready 단계의 실제 검증/반환이 필요하다. PRE_CREATE_OR_INSTALL_PREPARATION으로 표시한 링크만 준비 출력까지 요구한다. 필요한 OCID/주소만 인계한다.
- [34.ready Users — FUNCTIONAL_COMPLETION](01_Users.md#baseline-34-ready): 표시된 FUNCTIONAL_COMPLETION은 해당 linked ready 단계의 실제 검증/반환이 필요하다. PRE_CREATE_OR_INSTALL_PREPARATION으로 표시한 링크만 준비 출력까지 요구한다. 필요한 OCID/주소만 인계한다.
- [37.ready Policies — FUNCTIONAL_COMPLETION](04_Policies.md#baseline-37-ready): 표시된 FUNCTIONAL_COMPLETION은 해당 linked ready 단계의 실제 검증/반환이 필요하다. PRE_CREATE_OR_INSTALL_PREPARATION으로 표시한 링크만 준비 출력까지 요구한다. 필요한 OCID/주소만 인계한다.
- [40.ready Policy Conditions — FUNCTIONAL_COMPLETION](07_Policy_Conditions.md#baseline-40-ready): 표시된 FUNCTIONAL_COMPLETION은 해당 linked ready 단계의 실제 검증/반환이 필요하다. PRE_CREATE_OR_INSTALL_PREPARATION으로 표시한 링크만 준비 출력까지 요구한다. 필요한 OCID/주소만 인계한다.

- [Compartments](05_Compartments.md) **COMPARTMENT-fixture-ready** 완료: `iam-lab`/`iam-child`/`iam-sibling` OCID와 각 `proof.txt`가 실제 존재.
- [Users](01_Users.md) outsider deny control와 [Policies](04_Policies.md)/[Policy Conditions](07_Policy_Conditions.md) test 정책 정리 완료. 관리자는 기존 상위 tenancy allow policy가 test User를 포함하지 않는지 확인한다. resource fixture를42까지 필요한 경우 보존한다.

## 권한

- `GetPolicy`: `POLICY_READ` — [공식 operation 표](https://docs.oracle.com/en-us/iaas/Content/Identity/policyreference/iampolicyreference.htm); snapshot/복구 readback 관리자에게 해당 attachment scope의 읽기 권한이 필요하다.
- `ListPolicies`: `POLICY_READ` — [공식 operation 표](https://docs.oracle.com/en-us/iaas/Content/Identity/policyreference/iampolicyreference.htm); snapshot/복구 readback 관리자에게 해당 attachment scope의 읽기 권한이 필요하다.

### 실제 task/API별 공식 권한 본문 대조

아래 공식 operation 표에서 이 실습의 읽기·변경·정리 권한을 대조한다. family grant/일부 dependency 인용은 검증된 minimum policy가 아니며 실제 principal과 resource scope를 관리자가 승인해야 한다. 조직 Identity Domain Console user/group 작업은 해당 Domain 관리자 역할도 필요하며 legacy IAM user API 권한만으로 대체하지 않는다.

- `CreatePolicy`: POLICY_CREATE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/policyreference/iampolicyreference.htm).
- `DeletePolicy`: POLICY_DELETE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/policyreference/iampolicyreference.htm).
- `GetObject`: OBJECT_READ For a customer-managed key encrypted bucket, the objectstorage-<location> subject must have KEY_DECRYPT. — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm).

관리자만 User·Group·Dynamic Group·Policy·Compartment를 생성하거나 멤버십을 수정한다. Identity Domains의 사용자/그룹 관리에는 해당 Domain의 적절한 관리자 역할이 필요하고 OCI 리소스 Policy 관리 권한은 별개다. 기존 Administrators 또는 위임된 Domain 관리자에게 승인받는다. 학습자에게 `manage all-resources in tenancy`를 주는 방식으로 우회하지 않는다. 실습 작업자는 아래 절차에 명시한 Object Storage 읽기 권한만 받는다. [Overview of IAM](https://docs.oracle.com/en-us/iaas/Content/Identity/getstarted/identity-domains.htm)의 **The Administrators Group, Policy, and Administrator Roles**, [Managing Policies](https://docs.oracle.com/en-us/iaas/Content/Identity/Tasks/managingpolicies.htm)의 **Required IAM Policy**를 읽고 관리 주체와 검증 주체를 기록한다.

## 비용및주의사항

신규 항목은 scope test Group과 단계별 parent/child policy이며 bucket/proof는 owner33/38의 기존 자원을 재사용한다. 세 target GET/관리자 positive와 전송, child/sibling 보존기간을 두 creator의 승인 lease에 포함한다. 새 bucket/VM은 만들지 않는다. parent grant와 child grant를 겹쳐 deny를 주장하지 않고 test parent 삭제/readback 후 다음 상태를 시작한다. 만료에는 이번 grant/membership만 제거하고 fixture 삭제는 owner38/33에게 반환한다.

## 실습구성

### 이번 workshop 시나리오와 입력 계약

부모 scope grant를 제거하고 child-only policy로 세 영역 접근을 비교한다.

|입력|값/출처|시점|실행 위치|선택 이유|
|---|---|---|---|---|
|`IAM_LAB_OCID`|owner33의 actual output; `<...>` 대체 필수|실행 전 선행 완료값|Console/사용자 전용 shell|정확한 대상/소유관계 고정|
|`IAM_CHILD_OCID`|owner38의 actual output; `<...>` 대체 필수|실행 전 선행 완료값|Console/사용자 전용 shell|정확한 대상/소유관계 고정|
|`IAM_SIBLING_OCID`|owner38의 actual output; `<...>` 대체 필수|실행 전 선행 완료값|Console/사용자 전용 shell|정확한 대상/소유관계 고정|
|`OUTSIDER_USER_OCID`|owner34의 actual output; `<...>` 대체 필수|실행 전 선행 완료값|Console/사용자 전용 shell|정확한 대상/소유관계 고정|
|`SCOPE_GROUP_OCID`|owner41의 actual output; `<...>` 대체 필수|생성 후 readback|Console/사용자 전용 shell|정확한 대상/소유관계 고정|
|`NS`|owner33의 actual output; `<...>` 대체 필수|실행 전 선행 완료값|Console/사용자 전용 shell|정확한 대상/소유관계 고정|

|Console field|실제 입력/선택|이유|
|---|---|---|
|Policy attachment / statement location|p-parent-read attached iam-lab; p-child-read attached iam-child; separate scope IDs|이번 기능/실패 원인을 한 변수로 분리|

**생성·변경 범위:** lab-scope-test:IdentityGroup,p-parent-read:IamPolicy,p-child-read:IamPolicy. **완료 비교:** parent grant all allow→removed all deny→child-only child allow/others deny. 기대 패턴이며 OCI 실측값이 아니다. **정리:** 본 주제의 변경만 원복하고 전용 신규 resource만 dependency 역순으로 정리한다; 공유 자원은 creator와 consumer registry/expiry 기준으로 반환한다.

새 Group `lab-scope-test`, outsider membership, 부모 scope와 child scope policy 두 상태를 비교한다. policy가 **어디에 attach되는지**와 statement가 **어디를 허용하는지**를 분리한다. OCI allow 정책은 하위 범위를 제한하는 문장을 추가해 상위 허용을 취소하는 실험으로 설계하지 않는다.

## 실습절차

### 개별 완주 보정 — 생성·검증·소유권

세 영역 GetObject는 owner33의 signed get recipe에서 bucket-name을 각각 iam-proof-a/iam-child-proof/iam-sibling-proof로 바꾸고 LAB_OUTSIDER를 사용한다. 상태별 새 output 이름을 사용하고 부모 정책 삭제 후 전파와 다른 grant를 확인한 뒤 child policy를 추가한다.

### Same-target 세 영역 요청 — outsider PC

33의 CLI 인증준비를 LAB_OUTSIDER로 완료하고 actual REGION/NS를 다시 export한다. Console 정책 상태 하나마다 `STATE=parent-allow`, `no-parent-grant`, `child-only` 중 현재값을 넣는다. 관리자 positive control은 LAB_ADMIN 본인 shell에서 동일 bucket/key를 조회한다. 이 block은 mutation이 아니라 세 개의 direct GetObject이며 local output을 상태별 격리한다.
```bash
export REGION='<owner33-actual-region>'
export NS='<owner33-actual-namespace>'
export STATE='<parent-allow-or-no-parent-grant-or-child-only>'
mkdir "scope41-$STATE" || exit 1
for bucket in iam-proof-a iam-child-proof iam-sibling-proof; do
  if timeout -k 5s 15s oci os object get --profile LAB_OUTSIDER --region "$REGION" --namespace-name "$NS" --bucket-name "$bucket" --name proof.txt --file "scope41-$STATE/$bucket.txt" --max-retries 0 2> "scope41-$STATE/$bucket-error.txt"; then
    rc=0
    printf '%s\n' "$rc" > "scope41-$STATE/$bucket-rc.txt" || exit 1
    printf '%s GetObject exit=0\n' "$bucket"
    sha256sum "scope41-$STATE/$bucket.txt"
  else
    rc=$?
    printf '%s\n' "$rc" > "scope41-$STATE/$bucket-rc.txt" || exit 1
    printf '%s GetObject exit=%s; inspect authorization diagnostic/request-id\n' "$bucket" "$rc"
  fi
done
```
parent-allow는 세 exit0/hash 일치, no-parent-grant는 세 실제 authorization 거부, child-only는 child exit0/hash 일치·parent/sibling authorization 거부를 요구한다. 401/transport/local 파일 문제면 불허 checkpoint가 아니라 중단/입력복구다.

### 각 policy transition의 readback·유한 관찰

LAB_ADMIN API config를 준비한 **관리자 본인 PC의 전용 shell**에서 아래를 **Console Create 뒤** 실행한다. ID는 각각 Create의 실제 OCID이며 이름 검색으로 추정하지 않는다. parent policy 삭제 전에는 이 출력의 전체 statements/attachment를 복구담당자에게 전달한다.

관리자 snapshot shell은 다음 parent/child/list 출력 경로 모두 기존 file/symlink가 아닌지 확인하고 `set -C`를 켠 뒤 최초 생성한다. snapshot 재진입은 새 승인 경로를 사용한다; 기존 snapshot을 overwrite하지 않는다.

```bash
export P_PARENT_ID='<actual_p-parent-read_OCID>'
oci iam policy get --profile LAB_ADMIN --policy-id "$P_PARENT_ID" > scope41-parent-policy.json || exit 1
# child policy 생성 뒤에만 다음 두 행 실행
export P_CHILD_ID='<actual_p-child-read_OCID>'
oci iam policy get --profile LAB_ADMIN --policy-id "$P_CHILD_ID" > scope41-child-policy.json || exit 1
```

parent 삭제 뒤 관리자 `oci iam policy list --profile LAB_ADMIN --compartment-id "$IAM_LAB_OCID" --all > scope41-parent-after-delete.json || exit 1`로 exact P_PARENT_ID 부재와 다른 정책 불변을 대조한다. NotAuthorizedOrNotFound 단독은 삭제 확인이 아니다. child 조회의 compartment-id=IAM_CHILD_OCID/statement target=IAM_CHILD_OCID를 대조한다.

각 변경 뒤 **최대10분,60초 간격,최대10회** 같은 credentials의 세 GET을 새 `scope41-parent-allow-try1`/`no-parent-grant-try1`/`child-only-try1` 등 디렉터리로 실행한다.10분 또는 승인 expiry 중 먼저 도달하면 BLOCKED_PROPAGATION으로 중단한다. 원래 statements와 관리자가 준비한 rollback에 따라 교육 grant/membership만 복구/반환하고 새 상위 권한은 추가하지 않는다. 이 상한은 저술용 budget이며 OCI SLA가 아니다.

**첫 변경 전 독립 복구 station:**교육 grant/membership의 원래 배열과 정책 attachment/statements를 관리자가 보관하고 최대10분 UTC deadline에 이번 scope grant 제거/outsider 그룹 없는 baseline readback을 실행한다. matrix 한 회는 GET15초×3/kill 유예로 최대60초를 예약하며 종료60초 이전에만 시작한다. STATE는 parent-allow/no-parent-grant/child-only 뒤 `-try1`~`-try10`만 허용한다. parent-allow는3 성공, no-parent-grant는3 authorization deny, child-only는child 성공/나머지 deny여야 한다. 아래 validator를 각 matrix 직후 실행한다.
```bash
python3 - "$STATE" "scope41-$STATE" <<'PY'
import json,pathlib,re,sys
m=re.fullmatch(r'(parent-allow|no-parent-grant|child-only)(?:-try([1-9]|10))?',sys.argv[1]); assert m
p=pathlib.Path(sys.argv[2])
for bucket,expected in [('iam-proof-a',b'iam-proof-a\n'),('iam-child-proof',b'child\n'),('iam-sibling-proof',b'sibling\n')]:
 allow=m[1]=='parent-allow' or (m[1]=='child-only' and bucket=='iam-child-proof')
 f=p/(bucket+'.txt'); err=p/(bucket+'-error.txt')
 rc=int((p/(bucket+'-rc.txt')).read_text().strip())
 if allow: assert rc==0 and f.read_bytes()==expected and not err.read_text().strip()
 else:
  assert rc not in (0,124,125,126,127,130,137,143) and rc < 128
  s=err.read_text(); x=json.loads(s[s.index('{'):s.rindex('}')+1])
  assert x.get('status') in (403,404) and x.get('code') in ('NotAuthorized','NotAuthorizedOrNotFound')
PY
if [ "$?" -ne 0 ]; then exit 1; fi
```
관리자 same-target 세 positive/hash를 유지한다. 실패/만료는 자동 deny/PASS가 아니라 BLOCKED와 관리자 복구다.

### A. 부모 scope 상속
1. Group `lab-scope-test` 생성, outsider 추가. **Policies → iam-lab → Create** `p-parent-read`, attachment는 `iam-lab`.

```text
Allow group id <SCOPE_GROUP_OCID> to read objects in compartment id <IAM_LAB_OCID>
```

2. outsider direct GetObject로 parent `iam-proof-a`, child `iam-child-proof`, sibling `iam-sibling-proof` 모두 읽는다. Region/namespace는 동일, bucket 이름만 다르게 한다. 각 hash와 owner compartment OCID를 기록한다.
3. child에 narrower allow policy를 추가한다고 parent allow가 사라지지 않는다는 것을 공식 **Policies and Compartment Hierarchies/Policy Inheritance**와 연결한다. 운영 정책을 삭제하지 않는다.

### B. child scope만 남긴 테스트
4. 교육 test policy `p-parent-read`만 삭제한다. 전파 후 세 get이 모두 실패하는지 확인한다. 다른 상위 grant가 존재하면 이를 해결하기 전 scope deny 테스트를 완료로 처리하지 않는다.
5. **Policies → iam-child → Create** `p-child-read`, attachment는 `iam-child`:

```text
Allow group id <SCOPE_GROUP_OCID> to read objects in compartment id <IAM_CHILD_OCID>
```

6. child get 성공, parent/sibling get 실패를 검증한다. 관리자의 동일 세 get는 모두 성공해야 한다. Bucket list 없이 direct get로 수행하여 scope와 Console metadata를 섞지 않는다.
7. attachment OCID와 statement location OCID를 표로 기록한다. child에 attach한 policy가 root/sibling을 임의로 grant할 수 있다고 안내하지 않는다. Compartment path를 이름으로 표현할 경우 콜론 구분 규칙을 읽되 이 실험은 id syntax로 고정한다.

## 결과검증

<a id="baseline-41-ready"></a>

완료 단계 **SCOPE-tested**: parent scope 세 영역 성공, test parent grant 제거 후 deny, child-only grant에서 child 성공/parent+sibling 실패. attachment와 grant location을 분리한 표, exact target positive control을 남긴다. deny 정책을 생성하거나 상위 allow를 narrower allow로 override했다고 주장하지 않는다.

**실행 기록(아직 미작성):** 실행자/Region/시각, exact resource OCID, 설정 전·후, 성공 출력, deny 오류·exit code/request-id, positive control, 정리 결과. 문서 검증은 실제 OCI 실행 성공을 뜻하지 않는다.

## 문제해결

parent grant 제거 후 성공하면 tenancy/ancestor policy 또는 other group membership 확인. child get 실패면 policy attachment와 target id, namespace/name, 전파 확인. sibling get 실패가 wrong bucket name 때문이 아닌지 관리자 same get로 확인한다. 삭제 대상은 교육 `p-parent-read`만이다.

## 리소스정리

`p-child-read` 제거 → test membership 제거 → scope Group 삭제. Child/sibling resource를 더 쓰지 않으면38의 dependency 순서로 object/bucket→child compartment 정리한다. parent fixture는42까지 보존한다. 운영 tenancy policy는 변경하지 않는다.

**전용 로컬 산출물 정리:**실행 전에 새로 생성한 exact directory/file 경로를 소유 기록에 남기고, 증거 보존 필요가 끝난 뒤 해당 사용자/host에서 다음 파일만 삭제한다: scope41-parent-policy.json、scope41-child-policy.json、scope41-parent-after-delete.json 및 각 scope41-state/승인 retry directory의 iam-proof-a.txt/iam-child-proof.txt/iam-sibling-proof.txt와 각각 bucket-error.txt. 실패 요청이 부분 파일을 만들었으면 그것도 이번 owned 파일일 때만 제거한다. symlink/기존 재사용 파일은 삭제하지 않는다. 파일을 하나씩 exact path로 제거한 뒤 자신이 만든 빈 하위 directory부터 `rmdir`한다. retry는 실행 등록된 경로만 순회하고 wildcard/rm -rf는 사용하지 않는다; 미등록/타인 파일 잔존이면 중단한다.

**owner38 반환:**교육 policy/membership 정리와 원래 child/sibling fixture hash를38 담당자에게 전달하고 consumer41을 RETURNED 또는 승인 ABORTED로 기록한다. 원래 lease의 EXPIRY_UTC 이후 새 요청 금지; 38이 object/bucket/child 삭제 owner이며41은 parent/child fixture를 직접 삭제하지 않는다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 목표는 학습 후 설명·구분·확인할 내용이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [27] [Overview of IAM — Policy Scope / Inheritance](https://docs.oracle.com/en-us/iaas/Content/Identity/getstarted/identity-domains.htm)
  - 확인 범위: SECURITY POLICY.

### 실습 작업을 지원하는 추가 공식 본문

- [실습근거 1] [How Policies Work (without Identity Domains)](https://docs.oracle.com/en-us/iaas/Content/Identity/Concepts/policies.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
- [실습근거 2] [Policy Syntax](https://docs.oracle.com/en-us/iaas/Content/Identity/Concepts/policysyntax.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
- [실습근거 3] [Managing Policies](https://docs.oracle.com/en-us/iaas/Content/Identity/Tasks/managingpolicies.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
