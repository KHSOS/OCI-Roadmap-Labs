> **문서 검토 완료 · OCI 실행 미검증**: 이 Policies 실습의 한정된 문서 결함은 보완되었습니다. 문서상 실행 절차의 검토이며 실제 OCI 실행 성공이나 전체 교육과정의 승인을 뜻하지 않습니다.

# Policies

> 검토 범위: **이 Policies 실습의 문서상 실행 절차**. 선행 입력, 권한, 비용·기한, 실패 복구 및 정리 조건을 충족한 경우에만 실행합니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 전체 교육과정·게시 상태는 별도이며 [통합 상태](../../../Docs/README.md)를 확인하세요.

> **저술 완료 · OCI 실행 미검증** — Oracle 공식 작업 문서를 근거로 작성한 한국어 hands-on입니다. Oracle 공인 교육/실행 인증이 아니며 기대 결과와 실제 결과를 구분합니다. 비용 승인과 실제 선행 산출물 확인 후에만 실행하세요.

## 학습목표

- 분리된 lab-policy-test 그룹의 inspect policy로 실제 ListObjects 성공/GetObject 거부를 기록한다.
- read로 바꾼 뒤 A GetObject/hash와 새 key PutObject 거부를 대조하고 이번 정책·멤버십만 제거한다.

완료 기준은 아래 실제 생성·설정·요청 결과로 판정한다. 이 목표는 수행할 작업이며 OCI에서 관찰한 결과가 아니다.

## 대응 Roadmap 항목

- 항목명: **Policies**
- Roadmap 경로: **Identity and Access Management (IAM) → Policies**
- Node ID: `sequence-4-4`
- 상위 항목: **Identity and Access Management (IAM)** (`sequence-4-0`)
- 문서 범위: 공식 개념 목표와 실제 구성·허용/불허 검증·정리 절차. 아래 실습은 저술되었으나 OCI에서 실행 검증되지 않았습니다.
- [전체 항목 목록](../../../Docs/README.md)

## 사전준비

33 실제 REGION/NS/IAM_LAB_OCID/A/B proof bytes/hash와34의 outsider User OCID/LAB_OUTSIDER 인증을 받는다.35를 실행했다면 test Group/policy 삭제·outsider group-free 반환이 선행이다. 이 문서는 새 isolated Group에서 LIST/GET/PUT-control을 시험하며 원래 Reader policy를 고치지 않는다.

### 기준 계약과 실제 단계 선행조건

실제 tenancy/Region/Domain·주체·소유권은 [공통 baseline-contract.json](../../../baseline-contract.json)과 위 fixture 계약에 맞춘다. 빈 OCID/placeholder·미승인 권한은 실행 차단 입력이다. 이 API/Console 실습 때문에 VM을 생성하지 않는다.

- [33.ready Identity and Access Management (IAM) — FUNCTIONAL_COMPLETION](00_Identity_and_Access_Management_IAM.md#baseline-33-ready): 표시된 FUNCTIONAL_COMPLETION은 해당 linked ready 단계의 실제 검증/반환이 필요하다. PRE_CREATE_OR_INSTALL_PREPARATION으로 표시한 링크만 준비 출력까지 요구한다. 필요한 OCID/주소만 인계한다.
- [34.ready Users — FUNCTIONAL_COMPLETION](01_Users.md#baseline-34-ready): 표시된 FUNCTIONAL_COMPLETION은 해당 linked ready 단계의 실제 검증/반환이 필요하다. PRE_CREATE_OR_INSTALL_PREPARATION으로 표시한 링크만 준비 출력까지 요구한다. 필요한 OCID/주소만 인계한다.
- [Groups 선택 선행학습](02_Groups.md#baseline-35-ready): 전체 실행은 필수가 아니다. 선택해서 실행했다면 test policy/Group 삭제와 outsider group-free 반환을 먼저 확인한다. 필수 입력은33 fixture와34 인증된 outsider이며 모든 임시 grant/membership가 없어야 한다.

- [Identity and Access Management (IAM)](../../../Docs/04_Identity_and_Access_Management_IAM/00_Identity_and_Access_Management_IAM.md) **IAM-fixture-ready**, [Users](../../../Docs/04_Identity_and_Access_Management_IAM/01_Users.md) **USER-tested** 완료.33의 `lab-readers` policy와 겹치는 새 정책을 만들면 권한 축소가 보이지 않으므로 outsider를 별도 Group에 넣는다.
- Groups 전체 실습은 선택이다. 여기서 새 `lab-policy-test` Group를 만들고 outsider만 추가한다. 같은 이름의 기존 Group/policy가 있으면 재사용·수정하지 말고 중단한다. 기존 Reader policy는 수정하지 않는다.

## 권한

### 실제 task/API별 공식 권한 본문 대조

아래 공식 operation 표에서 이 실습의 읽기·변경·정리 권한을 대조한다. family grant/일부 dependency 인용은 검증된 minimum policy가 아니며 실제 principal과 resource scope를 관리자가 승인해야 한다. 조직 Identity Domain Console user/group 작업은 해당 Domain 관리자 역할도 필요하며 legacy IAM user API 권한만으로 대체하지 않는다.

- `CreatePolicy`: POLICY_CREATE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/policyreference/iampolicyreference.htm).
- `DeletePolicy`: POLICY_DELETE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/policyreference/iampolicyreference.htm).
- `GetObject`: OBJECT_READ For a customer-managed key encrypted bucket, the objectstorage-<location> subject must have KEY_DECRYPT. — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm).
- `ListObjects`: OBJECT_INSPECT — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm).
- `PutObject`: The permission required depends on whether the object already exists in the bucket: OBJECT_CREATE is required when an object with that name doesn't already exist in the bucket. OBJECT_OVERWRITE is required when an object with that name already exists in the bucket. For a customer-managed key encrypted bucket, the objectstorage-<location> subject must have KEY_ENCRYPT. — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm).
- `UpdatePolicy`: POLICY_UPDATE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/policyreference/iampolicyreference.htm).

관리자만 User·Group·Dynamic Group·Policy·Compartment를 생성하거나 멤버십을 수정한다. Identity Domains의 사용자/그룹 관리에는 해당 Domain의 적절한 관리자 역할이 필요하고 OCI 리소스 Policy 관리 권한은 별개다. 기존 Administrators 또는 위임된 Domain 관리자에게 승인받는다. 학습자에게 `manage all-resources in tenancy`를 주는 방식으로 우회하지 않는다. 실습 작업자는 아래 절차에 명시한 Object Storage 읽기 권한만 받는다. [Overview of IAM](https://docs.oracle.com/en-us/iaas/Content/Identity/getstarted/identity-domains.htm)의 **The Administrators Group, Policy, and Administrator Roles**, [Managing Policies](https://docs.oracle.com/en-us/iaas/Content/Identity/Tasks/managingpolicies.htm)의 **Required IAM Policy**를 읽고 관리 주체와 검증 주체를 기록한다.

## 비용및주의사항

신규 항목은 isolated Group과 `p-verb-test`이며 A fixture와 outsider는 33/34 재사용이다. LIST/GET/관리자 PUT-control/DELETE 및 다운로드 비용을 승인한다. control key `deny-upload.txt`는 관리자 성공 확인 뒤 삭제하고 outsider PUT 전에 부재를 확인한다. 예상 밖 PUT 성공이면 이번 key만 관리자 정리한다. 새 bucket/VM 또는 운영 Reader grant 변경은 하지 않는다. 관찰·복구는 승인 deadline/fixture lease 안에서 끝낸다.

## 실습구성

### 이번 workshop 시나리오와 입력 계약

isolated Group의 inspect/read를 바꾸고 실제 list/get/put를 비교한다.


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

- 입력: `POLICY_TEST_GROUP_OCID`
  - 값/출처: 이 문서 단계 A.1에서 생성한 Group의 actual output; `<...>` 대체 필수
  - 시점: 생성 후 readback
  - 실행 위치: Console/사용자 전용 shell
  - 선택 이유: 정확한 대상/소유관계 고정

- 입력: `NS`
  - 값/출처: owner33의 actual output; `<...>` 대체 필수
  - 시점: 실행 전 선행 완료값
  - 실행 위치: Console/사용자 전용 shell
  - 선택 이유: 정확한 대상/소유관계 고정



- Console field: Policy verb / bucket condition
  - 실제 입력/선택: p-verb-test; inspect→read objects; bucket <actual_BUCKET_A_name> only
  - 이유: 이번 기능/실패 원인을 한 변수로 분리


**생성·변경 범위:** lab-policy-test:IdentityGroup,p-verb-test:IamPolicy. **완료 비교:** inspect list allow/get deny; read get allow/new put deny. 기대 패턴이며 OCI 실측값이 아니다. **정리:** 본 주제의 변경만 원복하고 전용 신규 resource만 dependency 역순으로 정리한다; 공유 자원은 creator와 consumer registry/expiry 기준으로 반환한다.

`lab-policy-test` Group에 `inspect objects`와 `read objects`를 차례로 적용한다. 같은 Bucket A에 **ListObjects 성공/GetObject 실패 → GetObject 성공**을 검증하여 verb와 실제 API operation을 대응한다.

## 실습절차

### 개별 완주 보정 — 생성·검증·소유권

6번 put 전에 관리자가 선택된 새 object 이름이 존재하지 않음을 확인한다. 사용자 PC 전용 디렉터리에 한 줄 deny-upload.txt를 준비하고 oci os object put --profile LAB_OUTSIDER --region "$REGION" --namespace-name "$NS" --bucket-name "$BUCKET_A" --name deny-upload.txt --file ./deny-upload.txt 로 호출하되 아래6의 if/else fenced block만 사용한다. 예상 밖 성공이면 admin이 이번 object만 삭제하고 다른 grant를 조사한다.

### 유한 propagation request station — 본인 PC LAB_OUTSIDER 전용 Bash

`PROP_DEADLINE_EPOCH`은 관리자가 해당 mutation **이전** 승인한 절대 deadline이다. 같은 mutation의 여러 operation/retry/새 terminal에서 그대로 유지하며 초기화하지 않는다. 다음 별도 mutation은 직전 상태 확인/readback 및 새 원복값·deadline 재승인 뒤에만 새 절대 deadline을 전달받는다. 첫 grant/membership 변경 **이전** 관리자는 원래 전체 statement/group 배열, exact OCID, 복구용 관리자 Console 탭과 실제 UTC deadline을 저장한다. 각 변경 관찰은 최대600초/60초 시작 간격/10시도이며 교육용 상한이지 OCI SLA가 아니다. 비용/fixture expiry가 더 빠르면 그 시각을 사용한다. 개인 실습자는 shell 종료 시 관리자 탭으로 즉시 복구하고 늦어도 deadline에 원래 제한된 statement/group-free 상태를 복원/readback한다. 허용 관찰 성공 후에도 다음 단계의 원복을 생략하지 않는다. 아래 함수를 같은 전용 shell에 정의하고 각 단계의 호출을 실행한다. retry는 mutation 재제출이 아니라 동일 credentials/Region/namespace/key의 새 request뿐이다. stale만 재시도하고 auth401/timeout124·137/로컬오류/변조bytes는 즉시 중단한다.

```bash
set -euo pipefail
umask 077
export REGION='<owner33_actual_region>'
export NS='<owner33_actual_namespace>'
export BUCKET_A='<owner33_actual_A_bucket_name>'
export EXPECTED_A_SHA256='<owner33_original_A_SHA256>'
[[ $EXPECTED_A_SHA256 =~ ^[0-9a-f]{64}$ ]] || exit 1
export EXPIRY_EPOCH='<approved_UTC_expiry_epoch_seconds>'
export PROP_DEADLINE_EPOCH='<saved_absolute_mutation_deadline_epoch>'
case "$BUCKET_A" in ''|*'<'*|*'>'*) exit 1;; esac
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
        assert hashlib.sha256((d/'proof.txt').read_bytes()).hexdigest()==__import__('os').environ['EXPECTED_A_SHA256'], 'STOP: unexpected bytes'
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
  printf '%s\n' 'BLOCKED_PROPAGATION: restore original state/readback in admin tab now' >&2
  return 3
}
```

각 호출 실패(2 invalid/3 budget)는 후속 mutation을 금지하고 관리자 탭에서 즉시 원복한다. `75`는 함수 내부 stale 전용이며 shell 종료가 아니다. 전체 성공은 모든 지정 상태와 원복 readback을 요구한다. directory 충돌 시 삭제/재사용하지 말고 owner evidence를 보존한다. 새 terminal로 재개하면 같은 실제 입력과 **원래 deadline**을 다시 넣고 승인된 새 phase 이름만 사용하며600초 budget을 초기화하지 않는다.

개인 격리 실습에서는 별도 복구담당자가 필수가 아니다. 본인이 유지한 관리자 탭으로 복구한다. 공유 환경에서만 별도 담당자를 정한다. 실패/중단/시간 초과 시 아래 리소스정리 순서로 이번 test policy와 membership/Group을 제거하고 group-free를 확인한다. 기존 Reader나33 fixture는 제거하지 않는다.

### A. 분리된 테스트 그룹/정책 생성
1. 관리자가 **Domains → Groups → Create**로 `lab-policy-test`를 만들고 `lab-outsider`만 넣는다. Subject는 실제 Group OCID.
2. **Policies → iam-lab → Create Policy**: `p-verb-test`, manual editor:

```text
Allow group id <POLICY_TEST_GROUP_OCID> to inspect buckets in compartment id <IAM_LAB_OCID>
Allow group id <POLICY_TEST_GROUP_OCID> to inspect objects in compartment id <IAM_LAB_OCID> where target.bucket.name = '<actual_BUCKET_A_name>'
```

3. outsider의 별도 profile로 실행한다. API authentication 준비가 안 되면34 Users의 LAB_OUTSIDER 준비 절차대로 본인 PC config를 설정하고 CLI user OCID를 확인한다.

```bash
# initial grant: list positive must precede get negative, same frozen identity
propagate_probe verb37-inspect-list list allow "$BUCKET_A" || exit 1
propagate_probe verb37-inspect-get get deny "$BUCKET_A" || exit 1
```

### B. inspect/read 작업 차이
4. ListObjects에 `proof.txt`가 보이나 GetObject는 실패해야 한다. 관리자 same get 성공으로 object 존재를 확인한다.
5. 두 번째 statement의 `inspect objects`를 **read objects**로 바꾸고 전파 후 same GetObject 성공을 확인한다. `use objects`로 범위를 넓히지 않는다. read에는 inspect도 포함된다.
**단계5 read statement 전파 후 같은 outsider PC:**

```bash
propagate_probe verb37-read get allow "$BUCKET_A" || exit 1
```


**단계6 앞 관리자 same-operation control:** 관리자 PC에서 아래 순서대로33 fixture와 구별되는 `deny-upload.txt` 새 key를 확인한다. Console A Objects 전체 목록에서 exact key 부재를 확인하고, 새 로컬 directory에 동일 `iam-deny-create` LF bytes를 준비한다. Objects → Upload로 **동일 key/동일 bytes** PUT 성공과 GET bytes/hash를 확인한다 → 이 control key만 Delete → 새 목록에서 부재 readback. 원래 proof.txt를 변경하지 않는다. 이 성공/삭제가 미확인이면 outsider PUT을 시작하지 않는다. control 작업자는 OBJECT_CREATE/OBJECT_READ/OBJECT_DELETE가 필요하다. outsider 후 예상밖 성공이면 이 key만 관리자가 삭제/readback하며 다른 key는 보존한다.

6. 새 object put를 outsider로 시도하여 실패를 기록한다.

```bash
# LAB_OUTSIDER 본인 PC: 새 전용 작업 directory 안에서만
# inspect→read 변경 때 저장한 deadline/fixture expiry를 그대로 사용한다.
NOW=$(date +%s) || exit 1
[ $((NOW+50)) -le "$PROP_DEADLINE_EPOCH" ] && [ $((NOW+50)) -le "$EXPIRY_EPOCH" ] || exit 1
mkdir -m 700 verb37-put || exit 1
cd verb37-put || exit 1
pwd -P # 정리용 exact 절대 경로를 기록한다
if test -e ./deny-upload.txt || test -L ./deny-upload.txt; then printf '%s\n' 'STOP: local fixture exists'; exit 1; fi
test ! -e verb37-put-error.txt && test ! -L verb37-put-error.txt || exit 1
(umask 077; set -C; printf '%s\n' 'iam-deny-create' > ./deny-upload.txt) || exit 1
if timeout -k 5s 45s oci os object put --profile LAB_OUTSIDER --region "$REGION" --namespace-name "$NS" --bucket-name "$BUCKET_A" --name deny-upload.txt --file ./deny-upload.txt --max-retries 0 2>verb37-put-error.txt; then
  RC=0
  printf '%s\n' 'STOP: unexpected put allowed; admin remove only this test object and restore test grant'; exit 1
else RC=$?; fi
printf 'put exit status=%s\n' "$RC"
python3 - "$RC" verb37-put-error.txt <<'PY'
import sys,json
assert int(sys.argv[1]) not in (0,124,137)
s=open(sys.argv[2]).read(); x=json.loads(s[s.index('{'):s.rindex('}')+1])
assert x.get('status') in (403,404) and x.get('code') in ('NotAuthorized','NotAuthorizedOrNotFound'), 'STOP: not authorization deny'
PY
if [ "$?" -ne 0 ]; then exit 1; fi
cd .. || exit 1
# 성공이면 즉시 중단, 다른 grant 조사와 admin exact test object 정리
```

 파일은 한 줄 텍스트, 새 이름 `deny-upload.txt`. PutObject는 create/overwrite 권한이 필요하며 read는 이를 주지 않는다. 만일 업로드가 성공하면 정책 overlap을 조사하고 관리자로 object를 정리한다.

**다운로드 negative 안전:** 매 상태에서 새 전용 output filename을 사용하거나 이전에 본 실습이 만든 파일만 제거한 뒤 호출한다. local file이 이미 있다는 CLI overwrite prompt/기존 hash를 서비스 deny/success로 해석하지 않는다. 명령 종료코드와 새 파일 생성 여부를 함께 기록하고 실패한 요청 뒤 오래된 파일의 checksum을 성공 증거로 쓰지 않는다.

## 결과검증


### baseline-37-ready


완료 단계 **POLICY-tested**: inspect ListObjects 성공/GetObject 실패, read GetObject 성공/PutObject 실패. official reference의 `OBJECT_INSPECT`/`OBJECT_READ`/`OBJECT_CREATE`와 API를 대응한다. bucket 탐색 실패나 빈 list를 GetObject deny와 혼동하지 않는다.

**실행 기록(아직 미작성):** 실행자/Region/시각, exact resource OCID, 설정 전·후, 성공 출력, deny 오류·exit code/request-id, positive control, 정리 결과. 문서 검증은 실제 OCI 실행 성공을 뜻하지 않는다.

## 문제해결

inspect인데 get 성공하면 User가 다른 권한 그룹에 있는지 확인한다. ListObjects도 실패면 metadata/subject/region/namespace 입력과 source network를 확인한다. 정책 편집은 manual editor에서 문장 하나씩 검증하며 존재하지 않는 permission identifier를 invent하지 않는다.

## 리소스정리

`p-verb-test` 삭제 → outsider membership 제거 → `lab-policy-test` 삭제 → test download/upload 파일 제거.33 baseline은 유지한다. 삭제한 policy/Group의 exact OCID와 outsider membership 부재를 Console에서 확인한다. 삭제 mutation 직전 저장한 새 deadline으로 같은 probe를 `propagate_probe verb37-cleanup-get get deny "$BUCKET_A" || exit 1` 호출하고,33의 기존 Reader profile로 같은 A/proof.txt를 새 파일에 GET하여 원래 EXPECTED_A_SHA256와 비교한다. 실패하면 다음 실습으로 넘기지 말고 관리자 탭에서 잔여 test grant를 조사한다. fixture 소비 종료를33 creator에게 반환하며 A/B bucket·proof·Reader·outsider 자체는 각 creator의 정리 절차에 맡긴다.

**전용 로컬 산출물 정리:** 각 probe가 이번 run에 독점 생성한 `$HOME/<phase>/try1`부터 실제 등록된 `try10`까지의 exact 파일 `proof.txt`(부분 download 포함), `error.txt`, `list.json`만 증거 보존 승인 종료 후 제거한다. regular/non-symlink/현재 사용자 소유를 각각 확인하고 실제 생성된 파일만 `rm -- <exact-path>`, 빈 try directory와 phase만 `rmdir`한다. 미등록 phase/타인 파일은 보존; wildcard/recursive 삭제 금지. PUT 작업 directory는 생성 시 `pwd -P`로 절대 경로를 기록한다. 그 exact `verb37-put` 안의 이번 run 소유 regular/non-symlink `deny-upload.txt`, `verb37-put-error.txt`만 제거하고 빈 directory를 `rmdir`한다. 관리자 PUT-control의 원본/다운로드 파일도 기록한 exact 경로만 제거한다. 관리자 snapshot은 정책·멤버십 삭제 readback 이후에만 제거한다. retry fixture를 이전 출력과 바꾸지 않는다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 목표는 학습 후 설명·구분·확인할 내용이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [25] [Overview of Identity and Access Management — Policies](https://docs.oracle.com/en-us/iaas/Content/Identity/Concepts/overview.htm)
  - 확인 범위: POLICY .

### 실습 작업을 지원하는 추가 공식 본문

- [실습근거 1] [Managing Policies](https://docs.oracle.com/en-us/iaas/Content/Identity/Tasks/managingpolicies.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
- [실습근거 2] [Policy Syntax](https://docs.oracle.com/en-us/iaas/Content/Identity/Concepts/policysyntax.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
- [실습근거 3] [Details for Object Storage and Archive Storage](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
