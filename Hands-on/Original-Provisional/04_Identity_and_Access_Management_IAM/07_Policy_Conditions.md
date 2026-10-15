> **문서 검토 완료 · OCI 실행 미검증**: 이 Policy Conditions 실습의 제한된 문서 범위에서 시나리오·허용/불허 대조·복구·정리 절차를 검토했습니다. 전체 교육과정 승인이나 실제 OCI 실행 성공을 뜻하지 않습니다.

# Policy Conditions

> 문서 상태: **제한된 실습 문서 검토 완료**. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 실행 전 권한·비용·선행 산출물·독립 복구 담당자와 절대 deadline 확인은 아래 절차대로 유지합니다. 전체 교육과정 및 공개 상태는 [통합 상태](../../../Docs/README.md)와 별개입니다.

> **저술 완료 · OCI 실행 미검증** — Oracle 공식 작업 문서를 근거로 작성한 한국어 hands-on입니다. Oracle 공인 교육/실행 인증이 아니며 기대 결과와 실제 결과를 구분합니다. 비용 승인과 실제 선행 산출물 확인 후에만 실행하세요.

## 학습목표

- A bucket과 OBJECT_READ를 all 조건으로 제한한 policy를 만들어 outsider A GET 성공·A LIST/B GET 거부를 기록한다.
- permission 조건만 잠시 제거해 A LIST 성공을 확인한 뒤 조건 복원과 LIST 거부를 재검증하고 test policy를 정리한다.

완료 기준은 아래 실제 생성·설정·요청 결과로 판정한다. 이 목표는 수행할 작업이며 OCI에서 관찰한 결과가 아니다.

## 대응 Roadmap 항목

- 항목명: **Policy Conditions**
- Roadmap 경로: **Identity and Access Management (IAM) → Policy Conditions**
- Node ID: `sequence-4-7`
- 상위 항목: **Identity and Access Management (IAM)** (`sequence-4-0`)
- 문서 범위: 공식 개념 목표와 실제 구성·허용/불허 검증·정리 절차. 아래 실습은 저술되었으나 OCI에서 실행 검증되지 않았습니다.
- [전체 항목 목록](../../../Docs/README.md)

## 사전준비

33의 REGION/NS/IAM_LAB_OCID/A/B 실제 bucket 이름·원본 SHA256와34의 group-free outsider/LAB_OUTSIDER를 받는다.37 Policies 실험 전체는 필수가 아니다. 이전 실험을 수행했다면 그 test grant/membership을 먼저 정리하고 group-free 반환을 확인한 뒤 새 condition Group를 사용한다. 관리자 same-target GET/LIST positive와 원래 statements/절대 복구 deadline을 변경 전에 보관한다.

### 기준 계약과 실제 단계 선행조건

실제 tenancy/Region/Domain·주체·소유권은 [공통 baseline-contract.json](../../../baseline-contract.json)과 위 fixture 계약에 맞춘다. 빈 OCID/placeholder·미승인 권한은 실행 차단 입력이다. 이 API/Console 실습 때문에 VM을 생성하지 않는다.

- [33.ready Identity and Access Management (IAM) — FUNCTIONAL_COMPLETION](00_Identity_and_Access_Management_IAM.md#baseline-33-ready): 표시된 FUNCTIONAL_COMPLETION은 해당 linked ready 단계의 실제 검증/반환이 필요하다. PRE_CREATE_OR_INSTALL_PREPARATION으로 표시한 링크만 준비 출력까지 요구한다. 필요한 OCID/주소만 인계한다.
- [34.ready Users — FUNCTIONAL_COMPLETION](01_Users.md#baseline-34-ready): 표시된 FUNCTIONAL_COMPLETION은 해당 linked ready 단계의 실제 검증/반환이 필요하다. PRE_CREATE_OR_INSTALL_PREPARATION으로 표시한 링크만 준비 출력까지 요구한다. 필요한 OCID/주소만 인계한다.
- [37.ready Policies — 선택 학습/기존 변경 반환](04_Policies.md#baseline-37-ready): 이 실험을 이미 수행한 경우에만 test policy/membership 정리 결과를 확인한다. 여기서 새 Group/policy를 만들므로37 전체 실험을 먼저 실행할 필요는 없다.

- 필수 실행 경로는 위33/34 Hands-on 링크다. Docs의 [IAM](../../../Docs/04_Identity_and_Access_Management_IAM/00_Identity_and_Access_Management_IAM.md), [Users](../../../Docs/04_Identity_and_Access_Management_IAM/01_Users.md), [Policies](../../../Docs/04_Identity_and_Access_Management_IAM/04_Policies.md)는 개념 참고다. outsider는 baseline reader group 및 이전 test group에 없어야 한다.
- A/B 두 private bucket `proof.txt`를 관리자가 모두 read 성공으로 확인한다. 조건 변수가 요청에 실제로 제공되는 서비스/API만 선택한다.

## 권한

- `GetPolicy`: `POLICY_READ` — [공식 operation 표](https://docs.oracle.com/en-us/iaas/Content/Identity/policyreference/iampolicyreference.htm); snapshot/복구 readback 관리자에게 해당 attachment scope의 읽기 권한이 필요하다.

### 실제 task/API별 공식 권한 본문 대조

아래 공식 operation 표에서 이 실습의 읽기·변경·정리 권한을 대조한다. family grant/일부 dependency 인용은 검증된 minimum policy가 아니며 실제 principal과 resource scope를 관리자가 승인해야 한다. 조직 Identity Domain Console user/group 작업은 해당 Domain 관리자 역할도 필요하며 legacy IAM user API 권한만으로 대체하지 않는다.

- `CreatePolicy`: POLICY_CREATE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/policyreference/iampolicyreference.htm).
- `DeletePolicy`: POLICY_DELETE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/policyreference/iampolicyreference.htm).
- `GetObject`: OBJECT_READ For a customer-managed key encrypted bucket, the objectstorage-<location> subject must have KEY_DECRYPT. — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm).
- `ListObjects`: OBJECT_INSPECT — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm).
- `UpdatePolicy`: POLICY_UPDATE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/policyreference/iampolicyreference.htm).

관리자만 User·Group·Dynamic Group·Policy·Compartment를 생성하거나 멤버십을 수정한다. Identity Domains의 사용자/그룹 관리에는 해당 Domain의 적절한 관리자 역할이 필요하고 OCI 리소스 Policy 관리 권한은 별개다. 기존 Administrators 또는 위임된 Domain 관리자에게 승인받는다. 학습자에게 `manage all-resources in tenancy`를 주는 방식으로 우회하지 않는다. 실습 작업자는 아래 절차에 명시한 Object Storage 읽기 권한만 받는다. [Overview of IAM](https://docs.oracle.com/en-us/iaas/Content/Identity/getstarted/identity-domains.htm)의 **The Administrators Group, Policy, and Administrator Roles**, [Managing Policies](https://docs.oracle.com/en-us/iaas/Content/Identity/Tasks/managingpolicies.htm)의 **Required IAM Policy**를 읽고 관리 주체와 검증 주체를 기록한다.

## 비용및주의사항

신규 항목은 condition test Group/policy뿐이다. 기존 A/B fixture에 GET/LIST를 제한된 횟수로 수행하고 요청·전송 및 fixture 보존 비용을 owner33 lease에 포함한다. 새 bucket·VM·앱은 없다. widening은 별도 grant 추가가 아니라 이번 statement만 변경하며 성공/오류와 무관하게 전체 original statements를 즉시 복원한다. 승인 절대 deadline 안에 복구 readback이 없으면 후속 실습을 중단한다.

## 실습구성

### 이번 workshop 시나리오와 입력 계약

bucket/permission all 조건으로 direct GetObject와 ListObjects를 분리한다.

|입력|값/출처|시점|실행 위치|선택 이유|
|---|---|---|---|---|
|`IAM_LAB_OCID`|owner33의 actual output; `<...>` 대체 필수|실행 전 선행 완료값|Console/사용자 전용 shell|정확한 대상/소유관계 고정|
|`OUTSIDER_USER_OCID`|owner34의 actual output; `<...>` 대체 필수|실행 전 선행 완료값|Console/사용자 전용 shell|정확한 대상/소유관계 고정|
|`CONDITION_GROUP_OCID`|owner40의 actual output; `<...>` 대체 필수|생성 후 readback|Console/사용자 전용 shell|정확한 대상/소유관계 고정|
|`NS`|owner33의 actual output; `<...>` 대체 필수|실행 전 선행 완료값|Console/사용자 전용 shell|정확한 대상/소유관계 고정|

|Console field|실제 입력/선택|이유|
|---|---|---|
|Policy condition / permission / bucket|단계 A의 따옴표 포함 all 식에 실제 BUCKET_A 이름 입력|이번 기능/실패 원인을 한 변수로 분리|

**생성·변경 범위:** lab-condition-test:IdentityGroup,p-condition-test:IamPolicy. **완료 비교:** A get allow/list deny/B get deny; remove condition list allow→restore deny. 기대 패턴이며 OCI 실측값이 아니다. **정리:** 본 주제의 변경만 원복하고 전용 신규 resource만 dependency 역순으로 정리한다; 공유 자원은 creator와 consumer registry/expiry 기준으로 반환한다.

새 `lab-condition-test` Group에 outsider를 넣고 Bucket A의 GetObject만 허용한다. `target.bucket.name`와 `request.permission`을 **all** 조건으로 결합한다. ListObjects는 `OBJECT_INSPECT`이므로 condition에서 제외되는 것을 직접 검증한다. 조건은 case-insensitive라는 공식 제약 때문에 대소문자만 다른 bucket으로 deny를 설계하지 않는다.

## 실습절차

### 개별 완주 보정 — 생성·검증·소유권

Policy Create 전 outsider가37의 test group과 lab-readers에 없는지 확인한다. 각 조건 상태의 direct get 파일은 새 이름을 사용한다. error code만으로 deny라 하지 말고 같은 사용자 인증 정상/admin same object+list positive control을 함께 기록한다.

### 유한 propagation request station — 본인 PC LAB_OUTSIDER 전용 Bash

`PROP_DEADLINE_EPOCH`은 관리자가 해당 mutation **이전** 승인한 절대 deadline이다. 같은 mutation의 여러 operation/retry/새 terminal에서 그대로 유지하며 초기화하지 않는다. 다음 별도 mutation은 직전 상태 확인/readback 및 새 원복값·deadline 재승인 뒤에만 새 절대 deadline을 전달받는다. 첫 grant/membership 변경 **이전** 관리자는 원래 전체 statement/group 배열, exact OCID, 독립 복구담당자와 실제 UTC deadline을 저장한다. 각 변경 관찰은 최대600초/60초 시작 간격/10시도이며 교육용 상한이지 OCI SLA가 아니다. 비용/fixture expiry가 더 빠르면 그 시각을 사용한다. 담당자는 shell 종료와 무관하게 deadline에 원래 제한된 statement/group-free 상태를 복원/readback한다. 허용 관찰 성공 후에도 다음 단계의 원복을 생략하지 않는다. 아래 함수를 같은 전용 shell에 정의하고 각 단계의 호출을 실행한다. retry는 mutation 재제출이 아니라 동일 credentials/Region/namespace/key의 새 request뿐이다. stale만 재시도하고 auth401/timeout124·137/로컬오류/변조bytes는 즉시 중단한다.

```bash
set -euo pipefail
umask 077
export REGION='<owner33_actual_region>'
export NS='<owner33_actual_namespace>'
export BUCKET_A='<owner33_actual_A_bucket_name>'
export BUCKET_B='<owner33_actual_B_bucket_name>'
export EXPECTED_A_SHA256='<owner33_original_A_SHA256>'
export EXPECTED_B_SHA256='<owner33_original_B_SHA256>'
for h in "$EXPECTED_A_SHA256" "$EXPECTED_B_SHA256"; do [[ $h =~ ^[0-9a-f]{64}$ ]] || exit 1; done
for b in "$BUCKET_A" "$BUCKET_B"; do case "$b" in ''|*'<'*|*'>'*) exit 1;; esac; done
[[ ${BUCKET_A,,} != "${BUCKET_B,,}" ]] || exit 1
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
  case "$bucket" in "$BUCKET_A"|"$BUCKET_B") ;; *) return 2;; esac
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
        env=__import__('os').environ
        expected_hash=env['EXPECTED_A_SHA256'] if bucket==env['BUCKET_A'] else env['EXPECTED_B_SHA256']
        assert hashlib.sha256((d/'proof.txt').read_bytes()).hexdigest()==expected_hash, 'STOP: unexpected bytes'
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

### A. 조건부 policy 생성
1. 관리자가 Group `lab-condition-test` 생성, outsider 추가. **Policies → iam-lab → Create** `p-condition-test`:

```text
Allow group id <CONDITION_GROUP_OCID> to inspect buckets in compartment id <IAM_LAB_OCID>
Allow group id <CONDITION_GROUP_OCID> to read objects in compartment id <IAM_LAB_OCID> where all {target.bucket.name = '<actual_BUCKET_A_name>', request.permission = 'OBJECT_READ'}
```

2. 공식 Object Storage policy reference의 GetObject=`OBJECT_READ`, ListObjects=`OBJECT_INSPECT`를 읽고 condition 평가 대상을 표시한다. `read` verb가 기본적으로 inspect를 포함해도 `where`가 그 operation의 permission을 제한할 수 있다.

### B. 허용/불허 요청 비교
3. outsider profile로 아래 세 operation을 실행하고 return code/request-id를 기록한다.

```bash
propagate_probe condition40-all-get-a get allow "$BUCKET_A" || exit 1
propagate_probe condition40-all-list list deny "$BUCKET_A" || exit 1
propagate_probe condition40-all-get-b get deny "$BUCKET_B" || exit 1
```

4. A GetObject 성공, A ListObjects 실패, B GetObject 실패를 기대한다. Console의 Objects list가 숨겨져도 direct get 성공이 유효하므로 Console 메타데이터 권한을 추가해 조건을 무력화하지 않는다.
**관리자 snapshot 준비:** outsider PC shell과 별개의 LAB_ADMIN 전용 Bash shell을 열어 `umask 077`과 `export REGION='<owner33_actual_region>'`을 먼저 실행하고 실제 Region으로 대체했는지 확인한다. 이 shell의 현재 directory 절대 경로를 기록한다.

**snapshot 충돌 gate:**관리자 shell에서 `test ! -e condition40-original-policy.json && test ! -L condition40-original-policy.json` 성공 후 `set -C`(noclobber)를 켠다. 실패면 widening을 시작하지 않는다.

**조건 widening 전 recovery gate:** LAB_ADMIN config를 준비한 관리자 본인 PC 전용 shell에서 exact p-condition-test OCID/원래 전체 statements를 `oci iam policy get --policy-id '<actual_p-condition-test_OCID>' --profile LAB_ADMIN --region "$REGION" > condition40-original-policy.json || exit 1`로 저장한다. 복구 Console 담당자·원래 all statement와 최대10분 UTC deadline/fixture lease를 승인받는다. widened 관찰은 아래 유한 함수로 stale만 최대10회 관찰하되 성공/invalid/절대 deadline 도달 즉시 원래 all 조건으로 복원한다. PROP_DEADLINE_EPOCH은 widening 이전 승인된 deadline이며 restored 관찰에도 동일 값으로 남은 budget만 사용한다. 시간 내 전파 미확인은 BLOCKED_PROPAGATION이며 담당자가 deadline에 원래 statements를 반드시 복원/readback한다. 새 broad grant를 추가하거나 deadline을 연장하지 않는다.

관리자 shell은 위 outsider shell과 별개이므로 실제 `REGION`을 다시 설정한다. snapshot을 JSON으로 열어 `data.id`가 이번 policy OCID이고 `data.statements` 전체가 단계 A의 두 문장과 일치하는지 확인한다. 빈 파일/오류/불일치면 widening하지 않는다. snapshot의 절대 경로와 전체 문장을 복구 담당자에게 전달하고 복구 Console을 미리 연다. snapshot 충돌 guard와 저장은 같은 관리자 shell/작업 directory에서 수행한다. `set -C`는 이 전용 shell에만 적용한다.

이번 widening용 deadline을 변경 전에 outsider shell의 `PROP_DEADLINE_EPOCH`에 `export`하고 숫자·미래 시각·fixture expiry 이내인지 확인한다. 앞선 단계 A/B 관찰용 deadline을 무심코 재사용하지 않는다. widening 후에는 이 값을 갱신하지 않으며 restored 관찰도 같은 값만 쓴다. 새 terminal이면 같은 함수/입력/절대 deadline을 복원하고, `WIDENED_RESULT`는 보존한 관찰 기록이 PASS일 때만 PASS로 복원한다. 기록이 없거나 불명확하면 BLOCKED_PROPAGATION으로 두고 복구만 수행한다.

5. test statement를 일시적으로 `where target.bucket.name = '<actual_BUCKET_A_name>'`로 바꾸고 readback한다 → **아래 widened block**으로 A ListObjects 성공을 확인한다 → 즉시 원래 all 조건으로 복원/readback한다 → **아래 restored block**으로 ListObjects 거부와 A GetObject 성공을 확인한다. 다른 scope policy는 바꾸지 않는다.
**단계5 widened 조건 저장 후 outsider PC:**

```bash
# Widened observation only; failure is NOT permission to retain the wider condition.
if propagate_probe condition40-widened-list list allow "$BUCKET_A"; then WIDENED_RESULT=PASS; else WIDENED_RESULT=BLOCKED_PROPAGATION; fi
printf '%s\n' 'MANDATORY: independent Console owner restore complete original all statements NOW and read back'
```

**위 요청의 결과/exception/shell 종료와 무관하게** 독립 담당자가 즉시 또는 사전 승인 deadline에 원래 전체 all statements를 복원/readback한다. 복원 readback 없으면 다음 fence 실행 금지. widened 관찰 실패이면 복구 관찰은 하되 전체 결과는 BLOCKED이며 widening을 다시 제출하지 않는다.

```bash
# Restored-state only, after independent Console restoration readback
propagate_probe condition40-restored-list list deny "$BUCKET_A" || exit 1
propagate_probe condition40-restored-get-a get allow "$BUCKET_A" || exit 1
 test "$WIDENED_RESULT" = PASS || { printf 'BLOCKED_PROPAGATION: widening was not observed\n'; exit 1; }
```


6. 사용자 OCID 조건을 개념 목표와 연결해 읽는다: policygetstarted 본문은 특정 사용자 제한 `request.user.id`를 설명하지만 instance/function principals까지 이를 일반화하지 않는다. 이번 실제 operation은 bucket/permission 조건이며 사용자 조건 variant는 별도 승인을 받아야 한다.

**다운로드 negative 안전:** 매 상태에서 새 전용 output filename을 사용하거나 이전에 본 실습이 만든 파일만 제거한 뒤 호출한다. local file이 이미 있다는 CLI overwrite prompt/기존 hash를 서비스 deny/success로 해석하지 않는다. 명령 종료코드와 새 파일 생성 여부를 함께 기록하고 실패한 요청 뒤 오래된 파일의 checksum을 성공 증거로 쓰지 않는다.

## 결과검증


### baseline-40-ready


완료 단계 **CONDITION-tested**: all 조건 A get 성공/list 실패/B get 실패, permission condition 제거 시 ListObjects 성공, 원래 전체 all statements 복원/readback 성공 후 ListObjects 거부·GetObject 성공. 관리자 같은 object/list 성공 positive control. 공식문서의 변수 미존재 요청은 거부될 수 있다는 내용을 결과 해석에 적용한다.

**실행 기록(아직 미작성):** 실행자/Region/시각, exact resource OCID, 설정 전·후, 성공 출력, deny 오류·exit code/request-id, positive control, 정리 결과. 문서 검증은 실제 OCI 실행 성공을 뜻하지 않는다.

## 문제해결

ListObjects가 성공하면 broad read objects 정책 또는 다른 Group를 찾는다. GetObject까지 실패하면 namespace/name/exact condition/subject/전파 확인. Bucket condition은 case-insensitive이므로 A/B 이름을 case만 다르게 만들지 않는다. 요청에 없는 변수를 condition에 넣어 deny가 난 것은 목표 permission 제한과 다른 원인이다.

## 리소스정리

원래 statements 복구/readback을 먼저 확인한 뒤 test policy 삭제 → outsider membership 제거 → test Group 삭제 → 다운로드 파일 제거. Console에서 이번 policy와 Group이 사라지고 outsider가 원래 group-free 상태인지 확인한다. baseline Reader/Bucket는 이 문서가 삭제하지 않는다. 선택한41/42 등 소비자가 있으면 creator33/34에 보존을 알리고, 없으면 이번 소비 종료를 반환해 creator의 정리 절차로 종료한다. 조정했던 condition statement를 남긴 채 broad 정책을 추가해 해결하지 않는다.

**전용 로컬 산출물 정리:** 각 probe가 이번 run에 독점 생성한 `$HOME/<phase>/try1`부터 실제 등록된 `try10`까지의 exact 파일 `proof.txt`(부분 download 포함), `error.txt`, `list.json`만 증거 보존 승인 종료 후 제거한다. regular/non-symlink/현재 사용자 소유를 각각 확인하고 실제 생성된 파일만 `rm -- <exact-path>`, 빈 try directory와 phase만 `rmdir`한다. 미등록 phase/타인 파일은 보존; wildcard/recursive 삭제 금지. 관리자 condition40-original-policy.json은 해당 관리자가 collision guard 후 만든 exact snapshot만 증거 보존 승인 뒤 별도 정리한다.37 PUT 파일은 이 topic이 삭제하지 않는다. retry fixture를 이전 출력과 바꾸지 않는다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 목표는 학습 후 설명·구분·확인할 내용이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [26] [Getting Started with Policies — Policy Conditions](https://docs.oracle.com/en-us/iaas/Content/Identity/Concepts/policygetstarted.htm)
  - 확인 범위: If you need to grant access to a particular user.

### 실습 작업을 지원하는 추가 공식 본문

- [실습근거 1] [Policy Syntax](https://docs.oracle.com/en-us/iaas/Content/Identity/Concepts/policysyntax.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
- [실습근거 2] [How Policies Work (without Identity Domains)](https://docs.oracle.com/en-us/iaas/Content/Identity/Concepts/policies.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
- [실습근거 3] [Details for Object Storage and Archive Storage](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
- [실습근거 4] [Getting Started with Policies](https://docs.oracle.com/en-us/iaas/Content/Identity/Concepts/policygetstarted.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
