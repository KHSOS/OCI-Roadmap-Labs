> **기존 실습 초안 — 미검증**: 원래 93개 문서의 절차를 보존한 자료입니다. 새 튜토리얼의 품질 승인이나 OCI 실행 검증을 뜻하지 않습니다.

# Users

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../../../Docs/README.md)를 확인하세요.

> **저술 완료 · OCI 실행 미검증** — Oracle 공식 작업 문서를 근거로 작성한 한국어 hands-on입니다. Oracle 공인 교육/실행 인증이 아니며 기대 결과와 실제 결과를 구분합니다. 비용 승인과 실제 선행 산출물 확인 후에만 실행하세요.

## 학습목표

- lab-outsider를 같은 Domain에 생성·활성화하고 사용자 본인 세션/API profile을 준비한다.
- 동일 A proof.txt 요청을 그룹 없음→가입→탈퇴 순으로 실행하여 인증 정상 상태의 거부→허용→거부를 기록한다.

완료 기준은 아래 실제 생성·설정·요청 결과로 판정한다. 이 목표는 수행할 작업이며 OCI에서 관찰한 결과가 아니다.

## 대응 Roadmap 항목

- 항목명: **Users**
- Roadmap 경로: **Identity and Access Management (IAM) → Users**
- Node ID: `sequence-4-1`
- 상위 항목: **Identity and Access Management (IAM)** (`sequence-4-0`)
- 문서 범위: 공식 개념 목표와 실제 구성·허용/불허 검증·정리 절차. 아래 실습은 저술되었으나 OCI에서 실행 검증되지 않았습니다.
- [전체 항목 목록](../../../Docs/README.md)

## 사전준비

33 IAM-fixture-ready에서 동일 Domain, Group OCID, IAM_LAB_OCID, REGION/NS, A/B 이름·proof hash, 관리자 same-target positive와 fixture expiry를 받는다. 신규 User의 활성화/로그인 및 LAB_OUTSIDER 인증을 완료한 뒤 그룹 없는 실제 User OCID를35/37/38 선택 branch/40/41에 반환한다. VM/IP/앱 포트 입력은 없다.

### 기준 계약과 실제 단계 선행조건

실제 tenancy/Region/Domain·주체·소유권은 [공통 baseline-contract.json](../../../baseline-contract.json)과 위 fixture 계약에 맞춘다. 빈 OCID/placeholder·미승인 권한은 실행 차단 입력이다. 이 API/Console 실습 때문에 VM을 생성하지 않는다.

- [33.ready Identity and Access Management (IAM) — FUNCTIONAL_COMPLETION](00_Identity_and_Access_Management_IAM.md#baseline-33-ready): 표시된 FUNCTIONAL_COMPLETION은 해당 linked ready 단계의 실제 검증/반환이 필요하다. PRE_CREATE_OR_INSTALL_PREPARATION으로 표시한 링크만 준비 출력까지 요구한다. 필요한 OCID/주소만 인계한다.

- [Identity and Access Management (IAM)](../../../Docs/04_Identity_and_Access_Management_IAM/00_Identity_and_Access_Management_IAM.md) **IAM-fixture-ready** 완료: Reader A read 성공과 관리자 B read 성공. 기존 Reader/Group/Policy/Bucket를 보존한다.
- 신규 사용자용 별도 테스트 이메일과 브라우저 세션이 필요하다. 동일 주소를 억지로 재사용하거나 Administrators 계정으로 deny 테스트하지 않는다.

## 권한

### 실제 task/API별 공식 권한 본문 대조

아래 공식 operation 표에서 이 실습의 읽기·변경·정리 권한을 대조한다. family grant/일부 dependency 인용은 검증된 minimum policy가 아니며 실제 principal과 resource scope를 관리자가 승인해야 한다. 조직 Identity Domain Console user/group 작업은 해당 Domain 관리자 역할도 필요하며 legacy IAM user API 권한만으로 대체하지 않는다.

- `GetObject`: OBJECT_READ For a customer-managed key encrypted bucket, the objectstorage-<location> subject must have KEY_DECRYPT. — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm).

관리자만 User·Group·Dynamic Group·Policy·Compartment를 생성하거나 멤버십을 수정한다. Identity Domains의 사용자/그룹 관리에는 해당 Domain의 적절한 관리자 역할이 필요하고 OCI 리소스 Policy 관리 권한은 별개다. 기존 Administrators 또는 위임된 Domain 관리자에게 승인받는다. 학습자에게 `manage all-resources in tenancy`를 주는 방식으로 우회하지 않는다. 실습 작업자는 아래 절차에 명시한 Object Storage 읽기 권한만 받는다. [Overview of IAM](https://docs.oracle.com/en-us/iaas/Content/Identity/getstarted/identity-domains.htm)의 **The Administrators Group, Policy, and Administrator Roles**, [Managing Policies](https://docs.oracle.com/en-us/iaas/Content/Identity/Tasks/managingpolicies.htm)의 **Required IAM Policy**를 읽고 관리 주체와 검증 주체를 기록한다.

## 비용및주의사항

신규 항목은 `lab-outsider` User 하나이며 bucket A/B는 owner33의 기존 fixture를 재사용한다. 새 bucket·VM·Domain은 만들지 않는다. 추가 비용 검토 대상은 membership 전후의 작은 GET 요청과 다운로드 전송, 보존 중인 fixture 저장량이다. 각 mutation의 10분 관찰 상한과 owner33 lease 중 이른 시각에 복구/반환한다. 사용자 credentials/MFA는 본인이 설정하고 관리자 세션으로 deny 시험하지 않는다.

## 실습구성

### 이번 workshop 시나리오와 입력 계약

outsider User를 만들고 membership 추가·제거로 인증과 권한을 분리한다.

|입력|값/출처|시점|실행 위치|선택 이유|
|---|---|---|---|---|
|`IAM_LAB_OCID`|owner33의 actual output; `<...>` 대체 필수|실행 전 선행 완료값|Console/사용자 전용 shell|정확한 대상/소유관계 고정|
|`LAB_READERS_GROUP_OCID`|owner33의 actual output; `<...>` 대체 필수|실행 전 선행 완료값|Console/사용자 전용 shell|정확한 대상/소유관계 고정|
|`NS`|owner33의 actual output; `<...>` 대체 필수|실행 전 선행 완료값|Console/사용자 전용 shell|정확한 대상/소유관계 고정|
|`OUTSIDER_USER_OCID`|owner34의 actual output; `<...>` 대체 필수|생성 후 readback|Console/사용자 전용 shell|정확한 대상/소유관계 고정|

|Console field|실제 입력/선택|이유|
|---|---|---|
|Domain / Username / Group membership|existing approved Domain; lab-outsider; initially no grant-bearing group|이번 기능/실패 원인을 한 변수로 분리|

**생성·변경 범위:** lab-outsider:IdentityUser. **완료 비교:** authenticated outsider deny→member allow→removed deny. 기대 패턴이며 OCI 실측값이 아니다. **정리:** 본 주제의 변경만 원복하고 전용 신규 resource만 dependency 역순으로 정리한다; 공유 자원은 creator와 consumer registry/expiry 기준으로 반환한다.

새 User `lab-outsider` 하나를 생성하고 처음에는 `lab-readers`에 넣지 않는다. 로그인 성공(인증)과 Bucket A read 실패(권한 없음)를 나누고, 최소 Group 추가 후 read 성공을 검증한다. 사용자 credentials는 각자가 설정하며 신규 Domain을 만들지 않는다.

## 실습절차

### 개별 완주 보정 — 생성·검증·소유권

실습 생성 Username은 lab-outsider로 명시한다(email as username을 선택한 환경이면 실제 생성 username을 후속에서 그대로 사용). API deny/allow는 owner33의 CLI 인증 준비를 자신의 LAB_OUTSIDER profile로 완료한 뒤 같은 A object에 수행한다.

### A. 사용자와 credentials 준비
1. 관리자 **Domains → 동일 Domain → Users → Create**: First name `Lab`, Last name `Outsider`, 실제 테스트 이메일, Username 선택을 기록. 그룹 선택에서 `lab-readers`와 Administrators를 모두 **선택하지 않는다**. Domain settings의 primary email 요구를 충족한다.
2. 활성화 메일을 사용자 본인이 처리하고 MFA 정책을 따른다. credential 화면에서는 password, API keys, auth tokens가 별개임을 확인하되 불필요한 token은 발급하지 않는다.
3. 별도 브라우저/시크릿 창에서 신규 User 로그인 성공과 user OCID를 기록한다. 관리자 기존 세션 cookie로 테스트하지 않는다.

### B. 인증과 권한 분리

### 유한 propagation request station — 본인 PC LAB_OUTSIDER 전용 Bash

`PROP_DEADLINE_EPOCH`은 관리자가 해당 mutation **이전** 승인한 절대 deadline이다. 같은 mutation의 여러 operation/retry/새 terminal에서 그대로 유지하며 초기화하지 않는다. 다음 별도 mutation은 직전 상태 확인/readback 및 새 원복값·deadline 재승인 뒤에만 새 절대 deadline을 전달받는다. 첫 grant/membership 변경 **이전** 관리자는 원래 전체 statement/group 배열, exact OCID, 독립 복구담당자와 실제 UTC deadline을 저장한다. 각 변경 관찰은 최대600초/60초 시작 간격/10시도이며 교육용 상한이지 OCI SLA가 아니다. 비용/fixture expiry가 더 빠르면 그 시각을 사용한다. 담당자는 shell 종료와 무관하게 deadline에 원래 제한된 statement/group-free 상태를 복원/readback한다. 허용 관찰 성공 후에도 다음 단계의 원복을 생략하지 않는다. 아래 함수를 같은 전용 shell에 정의하고 각 단계의 호출을 실행한다. retry는 mutation 재제출이 아니라 동일 credentials/Region/namespace/key의 새 request뿐이다. stale만 재시도하고 auth401/timeout124·137/로컬오류/변조bytes는 즉시 중단한다.

```bash
set -euo pipefail
umask 077
export REGION='<owner33_actual_region>'
export NS='<owner33_actual_namespace>'
export EXPIRY_EPOCH='<approved_UTC_expiry_epoch_seconds>'
export PROP_DEADLINE_EPOCH='<independent_owner_absolute_mutation_deadline_epoch>'
case "$REGION:$NS" in *'<'*|*'>'*|:|*:|:*) exit 1;; esac
for n in "$EXPIRY_EPOCH" "$PROP_DEADLINE_EPOCH"; do case "$n" in ''|*[!0-9]*) exit 1;; esac; done
# PARENT must be owned/real. Every phase exclusively creates a private run directory.
test -d "$HOME" && test ! -L "$HOME" && test -O "$HOME" || exit 1
propagate_probe() {
  local phase=$1 op=$2 expected=$3 bucket=$4 work now deadline i started rc verdict
  case "$phase" in ''|*[!a-zA-Z0-9-]*) return 2;; esac
  case "$op:$expected" in get:allow|get:deny|list:allow|list:deny) ;; *) return 2;; esac
  now=$(date +%s) || return 2
  deadline=$((now+600)); [ "$EXPIRY_EPOCH" -ge "$deadline" ] || deadline=$EXPIRY_EPOCH
  [ "$PROP_DEADLINE_EPOCH" -ge "$deadline" ] || deadline=$PROP_DEADLINE_EPOCH
  work="$HOME/$phase"
  if test -e "$work" || test -L "$work"; then return 2; fi
  mkdir -m 700 -- "$work" || return 2
  for i in {1..10}; do
    started=$(date +%s) || return 2
    [ $((started+50)) -le "$deadline" ] || break
    mkdir -m 700 -- "$work/try$i" || return 2
    if [ "$op" = get ]; then
      if timeout -k 5s 45s oci os object get --profile LAB_OUTSIDER --region "$REGION" --namespace-name "$NS" --bucket-name "$bucket" --name proof.txt --file "$work/try$i/proof.txt" --max-retries 0 2>"$work/try$i/error.txt"; then rc=0; else rc=$?; fi
    else
      if timeout -k 5s 45s oci os object list --profile LAB_OUTSIDER --region "$REGION" --namespace-name "$NS" --bucket-name "$bucket" --all --max-retries 0 >"$work/try$i/list.json" 2>"$work/try$i/error.txt"; then rc=0; else rc=$?; fi
    fi
    if python3 - "$op" "$expected" "$bucket" "$rc" "$work/try$i" <<'PY'
import hashlib,json,pathlib,sys
op,expected,bucket,rc,d=sys.argv[1:]; rc=int(rc); d=pathlib.Path(d)
if rc==0:
    if op=='get':
        assert (d/'proof.txt').read_bytes()==(bucket+'\n').encode(), 'STOP: unexpected bytes'
        print('fresh proof SHA256',hashlib.sha256((d/'proof.txt').read_bytes()).hexdigest())
    else:
        x=json.loads((d/'list.json').read_text())['data']['objects']
        assert isinstance(x,list) and any(o['name']=='proof.txt' for o in x), 'STOP: unexpected list'
    observed='allow'
else:
    assert rc not in (124,137), 'STOP: timeout is not propagation'
    s=(d/'error.txt').read_text()
    x=json.loads(s[s.index('{'):s.rindex('}')+1])
    assert x.get('status') in (403,404) and x.get('code') in ('NotAuthorized','NotAuthorizedOrNotFound'), 'STOP: authentication/transport/local error'
    observed='deny'
if observed!=expected:
    print('RETRYABLE_STALE', observed, 'expected', expected); sys.exit(75)
print('EXPECTED_STATE', observed)
PY
    then return 0; else verdict=$?; fi
    [ "$verdict" = 75 ] || return 2
    now=$(date +%s) || return 2
    [ "$i" -lt 10 ] && [ $((started+60+50)) -le "$deadline" ] || break
    if [ "$now" -lt $((started+60)) ]; then sleep $((started+60-now)) || return 2; fi
  done
  printf '%s\n' 'BLOCKED_PROPAGATION: independent owner restore original state/readback now' >&2
  return 3
}
```

각 호출 실패(2 invalid/3 budget)는 후속 mutation을 금지하고 독립 담당자가 원복한다. `75`는 함수 내부 stale 전용이며 shell 종료가 아니다. 전체 성공은 모든 지정 상태와 원복 readback을 요구한다. directory 충돌 시 삭제/재사용하지 말고 owner evidence를 보존한다. 새 terminal로 재개하면 같은 실제 입력과 **원래 deadline**을 다시 넣고 승인된 새 phase 이름만 사용하며600초 budget을 초기화하지 않는다.

단계4의 group-free baseline: `propagate_probe user34-no-group get deny iam-proof-a || exit 1`. 단계5 가입 저장/readback 후: `propagate_probe user34-joined get allow iam-proof-a || exit 1`. 단계6 제거 저장/readback 후: `propagate_probe user34-removed get deny iam-proof-a || exit 1`. no-group의 예상밖 allow도 제한된 budget 동안만 관찰하며 관리자 positive와 alternate grant audit 없이는 deny를 승인하지 않는다.

4. Bucket A `proof.txt` get/download를 시도한다. 로그인은 성공했으나 read는 실패해야 한다. Console가 compartment 탐색을 숨기면33의 GetObject 명령을 신규 사용자 전용 profile로 실행한다. 본인 PC에 API key 설정이 필요하면 private key를 저장소 밖에 두고 credential 등록 후 user OCID/fingerprint가 이 User의 것인지 확인한다.
**membership 전파 관찰 budget:** 위 함수만 사용한다. joined/removed 각각 mutation 이전 독립 담당자의 절대 deadline을 입력하며 try1~10은 함수가 독점 생성한다. stale75만 재시도하고10분/expiry 내 미확인은 BLOCKED_PROPAGATION 및 원래 group-free 복구다.

5. 관리자가 **lab-readers → Users → Add user**로 `lab-outsider`를 추가한다. 변경 시각을 기록하고 적용 전파 후 새 세션/새 요청으로 동일 object를 읽는다.
6. 멤버십을 다시 제거하고 동일 read가 실패하는지 확인한다. 실패 판정 전에 정책 적용 지연·다른 그룹·cached credentials를 확인한다. `lab-outsider`는 deny control로 남긴다.

## 결과검증


### baseline-34-ready


완료 단계 **USER-tested**: 사용자 활성화/로그인 성공, group 없음 read 실패 → 가입 read 성공 → 탈퇴 read 실패. 관리자 object positive control 및 테스트 User OCID를 함께 기록한다. 비밀번호를 잘못 넣은 로그인 실패는 authorization deny가 아니다.

**실행 기록(아직 미작성):** 실행자/Region/시각, exact resource OCID, 설정 전·후, 성공 출력, deny 오류·exit code/request-id, positive control, 정리 결과. 문서 검증은 실제 OCI 실행 성공을 뜻하지 않는다.

## 문제해결

메일이 오지 않으면 실제 주소/Domain email 설정/스팸함 확인. 로그인 성공인데 A read 실패면 Group membership·policy Group OCID·전파·실행 profile을 확인. 탈퇴 후 성공하면 다른 Group/tenancy policy와 적용 지연을 조사한다. MFA를 꺼서 해결하지 않는다.

## 리소스정리

36~42 deny control이 필요하면 그룹 없는 `lab-outsider`와 fixture를 보존한다. 단독 종료라면 API key/token revoke → membership 없는지 확인 → 신규 User 삭제.33의 Reader나 업무 User는 삭제하지 않는다.

**전용 로컬 산출물 정리:** 각 probe가 이번 run에 독점 생성한 `$HOME/<phase>/try1`부터 실제 등록된 `try10`까지의 exact 파일 `proof.txt`(부분 download 포함), `error.txt`, `list.json`만 증거 보존 승인 종료 후 제거한다. regular/non-symlink/현재 사용자 소유를 각각 확인하고 실제 생성된 파일만 `rm -- <exact-path>`, 빈 try directory와 phase만 `rmdir`한다. 미등록 phase/타인 파일은 보존; wildcard/recursive 삭제 금지. 이 Users lab은 PUT-control/다른 topic 파일을 만들거나 정리하지 않는다. retry fixture를 이전 출력과 바꾸지 않는다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 목표는 학습 후 설명·구분·확인할 내용이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [25] [Overview of Identity and Access Management — Users](https://docs.oracle.com/en-us/iaas/Content/Identity/Concepts/overview.htm)
  - 확인 범위: USER .

### 실습 작업을 지원하는 추가 공식 본문

- [실습근거 1] [Creating a User](https://docs.oracle.com/en-us/iaas/Content/Identity/users/create-user-accounts.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
- [실습근거 2] [Managing Users](https://docs.oracle.com/en-us/iaas/Content/Identity/Tasks/managingusers.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
- [실습근거 3] [Managing Groups](https://docs.oracle.com/en-us/iaas/Content/Identity/Tasks/managinggroups.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
- [실습근거 4] [Details for Object Storage and Archive Storage](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
