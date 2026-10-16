> **문서 검토 완료 · 실제 OCI 실행 미검증**: 이 Groups 실습의 시나리오·권한 조건·검증·복구·정리 절차를 문서 범위에서 검토했습니다. 전체 교육과정 승인이나 OCI 실행 성공을 뜻하지 않습니다.

# Groups

> 문서 수용 범위: **이 파일의 실행 절차에 대한 문서 검토 완료**. 실제 OCI 실행은 수행하지 않았으며 모든 예상 출력은 패턴이지 실측값이 아닙니다. 아래 실제 권한·선행 산출물·비용·관찰 기한·복구 조건은 실행 전과 실행 중에 충족해야 합니다. 게시 및 렌더링 검증은 별도입니다.

> **저술 완료 · OCI 실행 미검증** — Oracle 공식 작업 문서를 근거로 작성한 한국어 hands-on입니다. Oracle 공인 교육/실행 인증이 아니며 기대 결과와 실제 결과를 구분합니다. 비용 승인과 실제 선행 산출물 확인 후에만 실행하세요.

## 학습목표

- lab-observers와 outsider membership을 생성한 뒤 A 읽기 policy를 추가하고 A/B GetObject 결과를 기록한다.
- policy를 유지한 채 membership 제거·재가입을 시험하고 outsider를 그룹 없는 baseline으로 반환한다.

완료 기준은 아래 실제 생성·설정·요청 결과로 판정한다. 이 목표는 수행할 작업이며 OCI에서 관찰한 결과가 아니다.

## 대응 Roadmap 항목

- 항목명: **Groups**
- Roadmap 경로: **Identity and Access Management (IAM) → Groups**
- Node ID: `sequence-4-2`
- 상위 항목: **Identity and Access Management (IAM)** (`sequence-4-0`)
- 문서 범위: 공식 개념 목표와 실제 구성·허용/불허 검증·정리 절차. 아래 실습은 저술되었으나 OCI에서 실행 검증되지 않았습니다.
- [전체 항목 목록](../../../Docs/README.md)

## 사전준비

33의 REGION/NS/IAM_LAB_OCID/A/B proof hash와34의 실제 OUTSIDER_USER_OCID·LAB_OUTSIDER/그룹 없는 membership 배열을 받는다. 기존 Reader/정책을 보존하고 이번 Group/policy만 생성한다. 마지막 탈퇴/readback과 Group/policy 삭제 후 outsider baseline을37/40에 반환한다.

### 기준 계약과 실제 단계 선행조건

실제 tenancy/Region/Domain·주체·소유권은 [공통 baseline-contract.json](../../../baseline-contract.json)과 위 fixture 계약에 맞춘다. 빈 OCID/placeholder·미승인 권한은 실행 차단 입력이다. 이 API/Console 실습 때문에 VM을 생성하지 않는다.

- [33.ready Identity and Access Management (IAM) — FUNCTIONAL_COMPLETION](00_Identity_and_Access_Management_IAM.md#baseline-33-ready): 표시된 FUNCTIONAL_COMPLETION은 해당 linked ready 단계의 실제 검증/반환이 필요하다. PRE_CREATE_OR_INSTALL_PREPARATION으로 표시한 링크만 준비 출력까지 요구한다. 필요한 OCID/주소만 인계한다.
- [34.ready Users — FUNCTIONAL_COMPLETION](01_Users.md#baseline-34-ready): 표시된 FUNCTIONAL_COMPLETION은 해당 linked ready 단계의 실제 검증/반환이 필요하다. PRE_CREATE_OR_INSTALL_PREPARATION으로 표시한 링크만 준비 출력까지 요구한다. 필요한 OCID/주소만 인계한다.

- [Identity and Access Management (IAM)](../../../Docs/04_Identity_and_Access_Management_IAM/00_Identity_and_Access_Management_IAM.md) **IAM-fixture-ready**, [Users](../../../Docs/04_Identity_and_Access_Management_IAM/01_Users.md) **USER-tested** 완료. `lab-outsider`가 어떤 권한 그룹에도 없어야 한다. fixture와 두 User를 보존한다.
- Group OCID를 기준으로 비교한다. 이름을 같게 다시 만들면 같은 Group이 아니므로 기존 정책 재사용을 기대하지 않는다.

## 권한

### 실제 task/API별 공식 권한 본문 대조

아래 공식 operation 표에서 이 실습의 읽기·변경·정리 권한을 대조한다. family grant/일부 dependency 인용은 검증된 minimum policy가 아니며 실제 principal과 resource scope를 관리자가 승인해야 한다. 조직 Identity Domain Console user/group 작업은 해당 Domain 관리자 역할도 필요하며 legacy IAM user API 권한만으로 대체하지 않는다.

- `CreatePolicy`: POLICY_CREATE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/policyreference/iampolicyreference.htm).
- `DeletePolicy`: POLICY_DELETE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/policyreference/iampolicyreference.htm).
- `GetObject`: OBJECT_READ For a customer-managed key encrypted bucket, the objectstorage-<location> subject must have KEY_DECRYPT. — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm).

관리자만 User·Group·Dynamic Group·Policy·Compartment를 생성하거나 멤버십을 수정한다. Identity Domains의 사용자/그룹 관리에는 해당 Domain의 적절한 관리자 역할이 필요하고 OCI 리소스 Policy 관리 권한은 별개다. 기존 Administrators 또는 위임된 Domain 관리자에게 승인받는다. 학습자에게 `manage all-resources in tenancy`를 주는 방식으로 우회하지 않는다. 실습 작업자는 아래 절차에 명시한 Object Storage 읽기 권한만 받는다. [Overview of IAM](https://docs.oracle.com/en-us/iaas/Content/Identity/getstarted/identity-domains.htm)의 **The Administrators Group, Policy, and Administrator Roles**, [Managing Policies](https://docs.oracle.com/en-us/iaas/Content/Identity/Tasks/managingpolicies.htm)의 **Required IAM Policy**를 읽고 관리 주체와 검증 주체를 기록한다.

## 비용및주의사항

신규 항목은 `lab-observers` Group과 test policy뿐이며 outsider와 A/B bucket은 34/33에서 재사용한다. GET/관리자 positive 요청·전송 및 기존 fixture 보존 비용을 승인하고 새 bucket/VM은 만들지 않는다. membership/policy 변경은 승인된 관찰 기한 안에 끝내며 마지막 outsider는 그룹 없는 baseline으로 반환한다. Group 재생성은 새 OCID이므로 이전 policy subject를 재사용하지 않는다.

## 실습구성

### 이번 workshop 시나리오와 입력 계약

새 Group와 정책을 만들어 membership와 grant의 두 조건을 확인한다.


- 입력: `IAM_LAB_OCID`
  - 값/출처: owner33의 actual output; `<...>` 대체 필수
  - 시점: 실행 전 선행 완료값
  - 실행 위치: Console/사용자 전용 shell
  - 선택 이유: 정확한 대상/소유관계 고정

- 입력: `OUTSIDER_USER_OCID`
  - 값/출처: owner34의 actual output; `<...>` 대체 필수
  - 시점: 실행 전 선행 완료값
  - 실행 위치: Console/사용자 전용 shell
  - 선택 이유: 정확한 대상/소유관계 고정

- 입력: `LAB_OBSERVERS_GROUP_OCID`
  - 값/출처: owner35의 actual output; `<...>` 대체 필수
  - 시점: 생성 후 readback
  - 실행 위치: Console/사용자 전용 shell
  - 선택 이유: 정확한 대상/소유관계 고정

- 입력: `NS`
  - 값/출처: owner33의 actual output; `<...>` 대체 필수
  - 시점: 실행 전 선행 완료값
  - 실행 위치: Console/사용자 전용 shell
  - 선택 이유: 정확한 대상/소유관계 고정



- Console field: Group name / request access / policy subject
  - 실제 입력/선택: lab-observers; request access off; actual Group OCID
  - 이유: 이번 기능/실패 원인을 한 변수로 분리


**생성·변경 범위:** lab-observers:IdentityGroup,p-observer-read-a:IamPolicy. **완료 비교:** no policy deny→policy allow A/deny B→membership removal deny. 기대 패턴이며 OCI 실측값이 아니다. **정리:** 본 주제의 변경만 원복하고 전용 신규 resource만 dependency 역순으로 정리한다; 공유 자원은 creator와 consumer registry/expiry 기준으로 반환한다.

새 Group `lab-observers`를 생성하고 동일 User에 대한 멤버십과 읽기 정책을 연결한다. Group 자체 생성은 권한 부여가 아니며 policy와 membership 둘 다 필요하다는 것을 GetObject로 확인한다.

## 실습절차

### 개별 완주 보정 — 생성·검증·소유권

policy 생성 전에 outsider의 LAB_OUTSIDER user/fingerprint를 확인한다. B deny의 positive control은 admin same object get이다. test policy/membership가 남은 상태로37/40을 시작하지 않는다.

**첫 요청 전 LAB_OUTSIDER PC 입력:**33의 실제 출력으로 다음 값을 채운다. 빈 값/placeholder면 중단한다.
```bash
export REGION='<owner33-actual-region>'
export NS='<owner33-actual-namespace>'
export BUCKET_A='<owner33-actual-A-bucket-name>'
export BUCKET_B='<owner33-actual-B-bucket-name>'
export PROOF_A_SHA256='<owner33-actual-A-sha256>'
export PROOF_B_SHA256='<owner33-actual-B-sha256>'
```
위 입력은 관리자 PC에도 같은 값으로 설정한다. hash는33에서 관리자가 확인한 원본의 SHA256이며 이 실습에서 받은 파일로 새로 정의하지 않는다. 각 shell에서 다음 사전검사를 실행한다.

```bash
for name in REGION NS BUCKET_A BUCKET_B PROOF_A_SHA256 PROOF_B_SHA256; do
  value=${!name}
  if [[ -z "$value" || "$value" == *'<'* || "$value" == *'>'* ]]; then
    printf 'STOP: actual input required: %s\n' "$name"; exit 1
  fi
done
[[ "$BUCKET_A" != "$BUCKET_B" ]] || exit 1
[[ "$PROOF_A_SHA256" =~ ^[[:xdigit:]]{64}$ && "$PROOF_B_SHA256" =~ ^[[:xdigit:]]{64}$ ]] || exit 1
```

**첫 membership 변경 전 복구:** 개인 실습은 관리자 탭을 유지하고34의 그룹 없는 원래 배열·exact User/Group OCID·UTC 종료시각을 저장한다. 공유 교육환경에서만 별도 Console 복구담당자를 지정한다. 각 변경은 최대10분/60초 간격/최대10회이며 마지막 요청은 종료50초 이전에만 시작한다. 아래 directory는 재시도마다 `group35-removed-try1`처럼 충돌하지 않는 새 이름으로 전부 바꾼다. 요청 timeout45초/강제종료 유예5초/max-retries0, 만료/오류에는 관리자가 이번 policy/membership만 제거하고 그룹 없는 baseline을 readback한다; 예산 연장 금지. 아래 shell의 `exit 1`은 관리자 Console 변경을 자동 복구하지 않는다. 어떤 STOP/실패/터미널 종료든 관리자 탭에서 이번 policy 삭제 → 이번 Group의 outsider membership 제거 → 원래 그룹 없는 배열 readback 순으로 복구하고 중단한다. 다른 그룹/정책/사용자는 변경하지 않는다. 전파 관찰 재시도는 같은 상태에서만 하며 기존 증거 디렉터리를 덮어쓰지 않는다.

no-policy와 removed의 A deny를 판정하기 직전에 관리자 PC에서 동일 A의 존재와 원본 hash를 다음처럼 확인한다. 첫 실행은 단계2 요청 전, 두 번째는 단계5 제거 후 outsider 요청 전이다. 두 번째 실행은 아래 모든 `group35-admin-a` 경로를 `group35-admin-a-removed`로 바꾼다. 이 검사가 실패하면 outsider의 오류를 deny로 판정하지 않고 복구 후 중단한다. 관리자와 outsider의 Region/namespace/bucket/object를 같게 유지하고 outsider의 profile/user/fingerprint 및 다른 권한 경로를 바꾸지 않는다.

```bash
mkdir -m 700 group35-admin-a || exit 1
timeout -k 5s 45s oci os object get --profile LAB_ADMIN --region "$REGION" --namespace-name "$NS" --bucket-name "$BUCKET_A" --name proof.txt --file group35-admin-a/proof.txt --max-retries 0 || exit 1
printf '%s  %s\n' "$PROOF_A_SHA256" group35-admin-a/proof.txt | sha256sum -c - || exit 1
```

각 상태의 요청은 해당 mutation 뒤에 실행하되 위 관리자 대조를 먼저 끝낸다.

### A. Group 생성
1. **Domains → 동일 Domain → Groups → Create group**: Name `lab-observers`, Description `IAM group membership test`, **User can request access off**. 생성 시 User는 넣지 않는다. Group OCID를 기록한다.
2. **lab-observers → Users → Add user**로 `lab-outsider`를 추가한다.33의 `lab-readers`에는 넣지 않는다. 새로운 Group에 policy가 아직 없으므로 A read가 실패하는지 확인한다.

**단계2 no-policy 직후 LAB_OUTSIDER PC:**

```bash
mkdir -m 700 "group35-no-policy" || exit 1
if timeout -k 5s 45s oci os object get --profile LAB_OUTSIDER --region "$REGION" --namespace-name "$NS" --bucket-name "$BUCKET_A" --name proof.txt --file "group35-no-policy/proof.txt" --max-retries 0 2>"group35-no-policy/error.txt"; then
  printf '%s\n' 'STOP: unexpected allow; restore owner test grant/membership'; exit 1
else
  RC=$?
  printf 'deny candidate RC=%s; inspect request-id and service code\n' "$RC"
  python3 - "group35-no-policy/error.txt" "$RC" <<'PY'
import sys,json
assert int(sys.argv[2]) not in (0,124,137), 'STOP: timeout is not deny'
s=open(sys.argv[1]).read()
try: x=json.loads(s[s.index('{'):s.rindex('}')+1])
except (ValueError,json.JSONDecodeError): raise SystemExit('STOP: not a structured service authorization error')
assert x.get('status') in (403,404) and x.get('code') in ['NotAuthorizedOrNotFound','NotAuthorized'], 'STOP: transport/auth/local error is not deny'
print('authorization candidate; mandatory same-target admin positive and frozen credentials')
PY
  if [ "$?" -ne 0 ]; then exit 1; fi
fi
printf 'GetObject RC=%s\n' "$RC"
```



### B. Policy와 membership 두 조건 검증
3. **Policies → iam-lab → Create**: `p-observer-read-a`, actual Group OCID로 다음 공식 syntax를 적용한다.

```text
Allow group id <LAB_OBSERVERS_GROUP_OCID> to inspect buckets in compartment id <IAM_LAB_OCID>
Allow group id <LAB_OBSERVERS_GROUP_OCID> to read objects in compartment id <IAM_LAB_OCID> where target.bucket.name = '<actual_BUCKET_A_name>'
```

4. 변경 전파 후 outsider PC에서 입력 `REGION`, `NS`를33의 실제 fixture 값으로 다시 설정하고 전용 빈 `group35-policy` 디렉터리를 만든 뒤 아래 A/B를 실행한다. 기존 디렉터리가 있으면 중단하고 충돌하지 않는 새 전용 이름을 사용한다. A exit0/hash=33 A, B는 service authorization 거부를 요구하며 관리자는 same B GET/hash로 존재를 대조한다.
```bash
mkdir -m 700 group35-policy || exit 1
timeout -k 5s 45s oci os object get --profile LAB_OUTSIDER --region "$REGION" --namespace-name "$NS" --bucket-name "$BUCKET_A" --name proof.txt --file group35-policy/a.txt --max-retries 0 || exit 1
printf '%s  %s\n' "$PROOF_A_SHA256" group35-policy/a.txt | sha256sum -c - || exit 1
# deliberate negative: capture rc immediately, do not let set -e suppress the recovery steps
if timeout -k 5s 45s oci os object get --profile LAB_OUTSIDER --region "$REGION" --namespace-name "$NS" --bucket-name "$BUCKET_B" --name proof.txt --file group35-policy/b.txt --max-retries 0 2> group35-policy/b-error.txt; then
  printf 'UNEXPECTED B allow: stop and remove this test membership/grant\n'; exit 1
else
  rc=$?
fi
python3 - "$rc" group35-policy/b-error.txt <<'CHECK'
import json,sys
rc=int(sys.argv[1]); assert rc not in (0,124,137), 'STOP: timeout is not deny'
s=open(sys.argv[2]).read(); x=json.loads(s[s.index('{'):s.rindex('}')+1])
assert x.get('status') in (403,404) and x.get('code') in ('NotAuthorized','NotAuthorizedOrNotFound'), 'STOP: authentication/transport/local error'
CHECK
[ "$?" = 0 ] || exit 1
```

후속 membership 변경 전에 관리자 별도 PC에서 같은 REGION/NS/BUCKET_B와 LAB_ADMIN으로 B positive control을 실행한다. 관리자도 같은 실제 BUCKET_B 값을 설정한다.

```bash
mkdir -m 700 group35-admin-b || exit 1
timeout -k 5s 45s oci os object get --profile LAB_ADMIN --region "$REGION" --namespace-name "$NS" --bucket-name "$BUCKET_B" --name proof.txt --file group35-admin-b/proof.txt --max-retries 0 || exit 1
printf '%s  %s\n' "$PROOF_B_SHA256" group35-admin-b/proof.txt | sha256sum -c - || exit 1
```

관리자 B 성공과 outsider B 구조화 거부가 모두 있어야 단계5로 진행한다. 401/timeout/로컬 오류/원본 불일치는 복구 후 중단한다.
5. policy는 유지하고 Group에서 User를 제거한 뒤 아래 removed block으로 A service deny를 기록한다.

**단계5 membership 제거 직후:**

```bash
mkdir -m 700 "group35-removed" || exit 1
if timeout -k 5s 45s oci os object get --profile LAB_OUTSIDER --region "$REGION" --namespace-name "$NS" --bucket-name "$BUCKET_A" --name proof.txt --file "group35-removed/proof.txt" --max-retries 0 2>"group35-removed/error.txt"; then
  printf '%s\n' 'STOP: unexpected allow; restore owner test grant/membership'; exit 1
else
  RC=$?
  printf 'deny candidate RC=%s; inspect request-id and service code\n' "$RC"
  python3 - "group35-removed/error.txt" "$RC" <<'PY'
import sys,json
assert int(sys.argv[2]) not in (0,124,137), 'STOP: timeout is not deny'
s=open(sys.argv[1]).read()
try: x=json.loads(s[s.index('{'):s.rindex('}')+1])
except (ValueError,json.JSONDecodeError): raise SystemExit('STOP: not a structured service authorization error')
assert x.get('status') in (403,404) and x.get('code') in ['NotAuthorizedOrNotFound','NotAuthorized'], 'STOP: transport/auth/local error is not deny'
print('authorization candidate; mandatory same-target admin positive and frozen credentials')
PY
  if [ "$?" -ne 0 ]; then exit 1; fi
fi
printf 'GetObject RC=%s\n' "$RC"
```


관리자가 동일 Group에 User를 재가입하고 전파 확인 뒤 아래 rejoined block을 실행한다.

**단계5 재가입 전파 후:**

```bash
mkdir -m 700 "group35-rejoined" || exit 1
if timeout -k 5s 45s oci os object get --profile LAB_OUTSIDER --region "$REGION" --namespace-name "$NS" --bucket-name "$BUCKET_A" --name proof.txt --file "group35-rejoined/proof.txt" --max-retries 0 2>"group35-rejoined/error.txt"; then
  RC=0
  printf '%s  %s\n' "$PROOF_A_SHA256" "group35-rejoined/proof.txt" | sha256sum -c - || exit 1
else
  RC=$?
  printf 'STOP: expected positive RC=%s\n' "$RC"; exit 1
fi
printf 'GetObject RC=%s\n' "$RC"
```

마지막 outsider 탈퇴 readback 뒤37/40에 반환한다.

관리자가 테스트 끝에 다시 User를 제거하고 membership readback을 기록한다.
6. Group→User 관계와 Policy subject Group OCID를 대응표로 남긴다. Console에 그룹이 보이는 것과 Group member의 리소스 권한은 별개다.

## 결과검증


### baseline-35-ready


완료 단계 **GROUP-tested**: group+member/no-policy 실패, group+member+policy A 성공/B 실패, member 제거 A 실패, 재가입 성공. 새 Group OCID를 반드시 기록한다. 테스트 끝의 outsider는 그룹 없는 deny control이다.

**실행 기록(아직 미작성):** 실행자/Region/시각, exact resource OCID, 설정 전·후, 성공 출력, deny 오류·exit code/request-id, positive control, 정리 결과. 문서 검증은 실제 OCI 실행 성공을 뜻하지 않는다.

## 문제해결

정책 없이 성공하면 다른 membership 또는 상위 정책이 존재한다. policy 생성 후 실패하면 wrong Domain Group/Group OCID·전파·profile user 확인. Group 삭제 오류는 member가 남았는지 확인. 이름을 재생성하면 정책 subject가 이전 OCID를 가리킬 수 있다.

## 리소스정리

반환 위치: [37 Policies의 리소스정리](04_Policies.md#리소스정리), [40 Policy Conditions의 리소스정리](07_Policy_Conditions.md#리소스정리).

test policy `p-observer-read-a` 삭제 → User membership 제거 → `lab-observers` 삭제.33의 `lab-readers`/policy와 fixture는 후속 IAM까지 보존한다. 삭제 후 exact Group OCID 조회 불가와 기존 Reader A read 정상 여부를 확인한다.

**전용 로컬 산출물 정리:** 실행 전에 새로 생성한 exact directory/file 경로를 소유 기록에 남기고, 증거 보존 필요가 끝난 뒤 해당 사용자/host에서 다음 파일만 삭제한다: group35-no-policy의 proof.txt/error.txt、group35-admin-b/proof.txt、group35-admin-a/proof.txt、group35-admin-a-removed/proof.txt、group35-policy의 a.txt/b.txt/b-error.txt、group35-removed/rejoined의 proof.txt/error.txt 및 승인 retry directory의 동일 파일. 실패 요청이 부분 파일을 만들었으면 그것도 이번 owned 파일일 때만 제거한다. symlink/기존 재사용 파일은 삭제하지 않는다. 파일을 하나씩 exact path로 제거한 뒤 자신이 만든 빈 하위 directory부터 `rmdir`한다. retry는 실행 등록된 경로만 순회하고 wildcard/rm -rf는 사용하지 않는다; 미등록/타인 파일 잔존이면 중단한다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 목표는 학습 후 설명·구분·확인할 내용이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [25] [Overview of Identity and Access Management — Groups](https://docs.oracle.com/en-us/iaas/Content/Identity/Concepts/overview.htm)
  - 확인 범위: GROUP .

### 실습 작업을 지원하는 추가 공식 본문

- [실습근거 1] [Creating a Group](https://docs.oracle.com/en-us/iaas/Content/Identity/groups/create-groups.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
- [실습근거 2] [Managing Groups](https://docs.oracle.com/en-us/iaas/Content/Identity/Tasks/managinggroups.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
- [실습근거 3] [Policy Syntax](https://docs.oracle.com/en-us/iaas/Content/Identity/Concepts/policysyntax.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
- [실습근거 4] [Details for Object Storage and Archive Storage](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
