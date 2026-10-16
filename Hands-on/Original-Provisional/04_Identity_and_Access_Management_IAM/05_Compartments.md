> **문서 검토 완료 · 실제 OCI 실행 미검증**: 이 Compartments 실습의 한정된 시나리오·절차·복구·정리를 문서 기준으로 검토했습니다. 전체 교육과정 승인이나 OCI 실행 성공을 뜻하지 않습니다.

# Compartments

> 문서 수용 범위: **한정된 실행 절차 문서 검토 완료**. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 실행 전 실제 권한·선행 산출물·비용 승인과 아래 안전·복구·정리 조건을 충족해야 합니다. 전체 과정 및 게시 상태는 [통합 상태](../../../Docs/README.md)에서 별도로 확인하세요.

> **저술 완료 · OCI 실행 미검증** — Oracle 공식 작업 문서를 근거로 작성한 한국어 hands-on입니다. Oracle 공인 교육/실행 인증이 아니며 기대 결과와 실제 결과를 구분합니다. 비용 승인과 실제 선행 산출물 확인 후에만 실행하세요.

## 학습목표

- iam-lab 아래 두 자식 Compartment와 각각 private Standard Bucket/proof.txt를 생성하고 실제 parent/owner/Region/hash를 기록한다.
- 관리자 same-key 읽기와 Reader 두 거부를 대조한다. outsider 비교와41 scope 인계는 선택하며 미선택이면 이번 fixture를 정리한다.

완료 기준은 아래 실제 생성·설정·요청 결과로 판정한다. 이 목표는 수행할 작업이며 OCI에서 관찰한 결과가 아니다.

## 대응 Roadmap 항목

- 항목명: **Compartments**
- Roadmap 경로: **Identity and Access Management (IAM) → Compartments**
- Node ID: `sequence-4-5`
- 상위 항목: **Identity and Access Management (IAM)** (`sequence-4-0`)
- 문서 범위: 공식 개념 목표와 실제 구성·허용/불허 검증·정리 절차. 아래 실습은 저술되었으나 OCI에서 실행 검증되지 않았습니다.
- [전체 항목 목록](../../../Docs/README.md)

## 사전준비

33 IAM-fixture-ready의 parent IAM_LAB_OCID/REGION/NS·관리자 LAB_ADMIN 및 Reader LAB_READER 인증과 Reader A baseline positive를 받는다. child/sibling 실제 OCID·bucket/proof bytes/hash는 이 문서 생성 출력이며41에 인계한다. outsider deny 선택 시에만34 USER-tested의 그룹 없는 User OCID/LAB_OUTSIDER를 먼저 반환받는다; outsider 비교 생략 경로는 NOT_SELECTED로 남긴다.

### 기준 계약과 실제 단계 선행조건

실제 tenancy/Region/Domain·주체·소유권은 [공통 baseline-contract.json](../../../baseline-contract.json)과 위 fixture 계약에 맞춘다. 빈 OCID/placeholder·미승인 권한은 실행 차단 입력이다. 이 API/Console 실습 때문에 VM을 생성하지 않는다.

- [33.ready Identity and Access Management (IAM) — FUNCTIONAL_COMPLETION](00_Identity_and_Access_Management_IAM.md#baseline-33-ready): 표시된 FUNCTIONAL_COMPLETION은 해당 linked ready 단계의 실제 검증/반환이 필요하다. PRE_CREATE_OR_INSTALL_PREPARATION으로 표시한 링크만 준비 출력까지 요구한다. 필요한 OCID/주소만 인계한다.

- [Identity and Access Management (IAM)](../../../Docs/04_Identity_and_Access_Management_IAM/00_Identity_and_Access_Management_IAM.md) **IAM-fixture-ready**의 `iam-lab` OCID와 관리자 permissions 확보. 초기 network bootstrap은 관리자가 제공한 compartment에서 실행하므로 본 실습을 network 선행으로 역연결하지 않는다.
- 실제 생성할 child/sibling 이름이 중복되거나 업무 compartment와 혼동되지 않는지 확인한다.41을 선택한 경우에만 등록된 expiry까지 자원을 보존한다. 선택하지 않으면 이 문서 검증 후 아래 정리를 수행한다.

## 권한

### 실제 task/API별 공식 권한 본문 대조

아래 공식 operation 표에서 이 실습의 읽기·변경·정리 권한을 대조한다. family grant/일부 dependency 인용은 검증된 minimum policy가 아니며 실제 principal과 resource scope를 관리자가 승인해야 한다. 조직 Identity Domain Console user/group 작업은 해당 Domain 관리자 역할도 필요하며 legacy IAM user API 권한만으로 대체하지 않는다.

- `CreateBucket`: BUCKET_CREATE If the KMS Key ID is provided to the operation, the following additional permissions are required: KEY_ASSOCIATE The objectstorage-<location> subject must also have: KEY_ENCRYPT, KEY_DECRYPT, KEY_READ. — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm).
- `CreateCompartment`: COMPARTMENT_CREATE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/policyreference/iampolicyreference.htm).
- `DeleteBucket`: BUCKET_DELETE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm).
- `DeleteCompartment`: COMPARTMENT_DELETE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/policyreference/iampolicyreference.htm).
- `DeleteObject`: OBJECT_DELETE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm).
- `GetObject`: OBJECT_READ For a customer-managed key encrypted bucket, the objectstorage-<location> subject must have KEY_DECRYPT. — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm).
- `PutObject`: The permission required depends on whether the object already exists in the bucket: OBJECT_CREATE is required when an object with that name doesn't already exist in the bucket. OBJECT_OVERWRITE is required when an object with that name already exists in the bucket. For a customer-managed key encrypted bucket, the objectstorage-<location> subject must have KEY_ENCRYPT. — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm).

관리자만 User·Group·Dynamic Group·Policy·Compartment를 생성하거나 멤버십을 수정한다. Identity Domains의 사용자/그룹 관리에는 해당 Domain의 적절한 관리자 역할이 필요하고 OCI 리소스 Policy 관리 권한은 별개다. 기존 Administrators 또는 위임된 Domain 관리자에게 승인받는다. 학습자에게 `manage all-resources in tenancy`를 주는 방식으로 우회하지 않는다. 실습 작업자는 아래 절차에 명시한 Object Storage 읽기 권한만 받는다. [Overview of IAM](https://docs.oracle.com/en-us/iaas/Content/Identity/getstarted/identity-domains.htm)의 **The Administrators Group, Policy, and Administrator Roles**, [Managing Policies](https://docs.oracle.com/en-us/iaas/Content/Identity/Tasks/managingpolicies.htm)의 **Required IAM Policy**를 읽고 관리 주체와 검증 주체를 기록한다.

## 비용및주의사항

iam-lab 아래 child/sibling Compartment와 각 private Standard Bucket/작은 proof 하나를 새로 만든다. owner33 A/B bucket을 이 두 새 fixture로 혼동하지 않는다. 두 추가 bucket의 저장·PUT/GET/LIST/DELETE와 실제 전송 비용, 41 선택 시 보존 종료시각을 정한다. VM·VCN·앱은 추가하지 않는다. outsider deny는 USER-tested가 반환된 선택 branch에서만 수행한다.41 미선택이면 지금, 선택이면41 반환 후 owner38이 object/bucket/child를 정리하고 parent는 owner33에게 반환한다.

## 실습구성

### 이번 workshop 시나리오와 입력 계약

child/sibling Compartment와 각 Bucket를 실제 생성하고 소유경계를 기록한다.


- 입력: `IAM_LAB_OCID`
  - 값/출처: owner33의 actual output; `<...>` 대체 필수
  - 시점: 실행 전 선행 완료값
  - 실행 위치: Console/사용자 전용 shell
  - 선택 이유: 정확한 대상/소유관계 고정

- 입력: `OUTSIDER_USER_OCID`
  - 값/출처: [Users](01_Users.md#baseline-34-ready)의 실제 User OCID/그룹 없는 LAB_OUTSIDER; outsider 비교 생략 경로는 NOT_SELECTED
  - 시점: outsider deny branch 선택 전만
  - 실행 위치: outsider 본인 PC
  - 선택 이유: 관리자 fixture 생성에는 User34가 필요하지 않음

- 입력: `IAM_CHILD_OCID`
  - 값/출처: owner38의 actual output; `<...>` 대체 필수
  - 시점: 생성 후 readback
  - 실행 위치: Console/사용자 전용 shell
  - 선택 이유: 정확한 대상/소유관계 고정

- 입력: `IAM_SIBLING_OCID`
  - 값/출처: owner38의 actual output; `<...>` 대체 필수
  - 시점: 생성 후 readback
  - 실행 위치: Console/사용자 전용 shell
  - 선택 이유: 정확한 대상/소유관계 고정

- 입력: `NS`
  - 값/출처: owner33의 actual output; `<...>` 대체 필수
  - 시점: 실행 전 선행 완료값
  - 실행 위치: Console/사용자 전용 shell
  - 선택 이유: 정확한 대상/소유관계 고정



- Console field: Compartment parent / bucket compartment / Region
  - 실제 입력/선택: iam-child and iam-sibling parent iam-lab; same fixture Region; private Standard
  - 이유: 이번 기능/실패 원인을 한 변수로 분리


**생성·변경 범위:** iam-child:Compartment,iam-sibling:Compartment,iam-child-proof:Bucket,iam-sibling-proof:Bucket. **완료 비교:** admin each hash success; Reader each deny; outsider each deny는 선택; record owner vs Region. 기대 패턴이며 OCI 실측값이 아니다. **정리:** 본 주제의 변경만 원복하고 전용 신규 resource만 dependency 역순으로 정리한다; 공유 자원은 creator와 consumer registry/expiry 기준으로 반환한다.

`iam-lab` 아래 child `iam-child`와 동등한 자식 `iam-sibling`를 생성한다. 각 private Standard Bucket 하나에 서로 다른 `proof.txt`를 넣는다. compartment는 Subnet/Region 경계가 아니라 리소스 소유/정책 범위라는 것을 owner와 Region 속성을 분리해서 기록한다.

## 실습절차

### 개별 완주 보정 — 생성·검증·소유권

아래4와 결과검증의 outsider deny를 수행하려면 [Users](01_Users.md#baseline-34-ready)의 USER-tested와 그룹 없는 LAB_OUTSIDER 인증/반환이 필수다. 사전준비의 OUTSIDER_USER_OCID는 그 실제 출력이다. outsider 비교 생략 선택이면4/outsider deny를 NOT_SELECTED로 기록하고 COMPARTMENT-fixture-ready만 반환한다; USER-tested를 수행했다고 쓰지 않는다. 두 경로 모두33의 관리자 LAB_ADMIN과 Reader LAB_READER 인증·A baseline positive가 필수이며 관리자/Reader 검증은 같은 실제 namespace/Region을 사용한다. outsider 비교 생략 경로는 User34만 생략하고 단계5 Reader 두 deny는 완료한다.

### A. 두 child compartment 생성
1. **Identity & Security → Compartments → iam-lab → Create compartment**: Name `iam-child`, Description `IAM inheritance child`, Parent `iam-lab`. `iam-sibling`도 같은 parent로 만든다. 두 OCID와 parent OCID를 기록한다.
**로컬 PC fixture 준비 (단계2 Upload 전):**

```bash
mkdir -m 700 compartment38-fixtures || exit 1
mkdir compartment38-fixtures/child compartment38-fixtures/sibling || exit 1
printf '%s\n' child > compartment38-fixtures/child/proof.txt || exit 1
printf '%s\n' sibling > compartment38-fixtures/sibling/proof.txt || exit 1
sha256sum compartment38-fixtures/child/proof.txt compartment38-fixtures/sibling/proof.txt || exit 1
```

기존 directory면 mkdir에서 중단하며 덮어쓰지 않는다. 아래 각 Upload에 해당 proof.txt만 사용한다.

2. Console Region을 기존 fixture Region으로 고정한다. **Object Storage → Buckets** Compartment를 `iam-child`로 바꾸고 `iam-child-proof`를 Private/Standard/Oracle-managed key/Versioning off로 생성한다. `proof.txt` 내용 `child` 업로드. sibling에서도 `iam-sibling-proof`, 내용 `sibling` 생성한다.
3. 관리자 각 GetObject 성공·hash로 fixture를 확인한다. Compartment 생성 후 목록에 안 보이면 Region별 resource와 IAM list 전파를 구별한다.

### A-2. 실제 bytes와 주체별 GET

실행 환경은 로컬 Linux/WSL Bash이며 `oci`, `timeout`, `python3`, `sha256sum`이 설치되어 있어야 한다. API signing 설정은33에서 완료한 각 본인 profile을 사용하고 키를 서로 복사하지 않는다. REGION/NS는33과 같은 실제 값으로 각 사용자 shell에 지정한다. 새 터미널에서도33의 보존 기록에서 두 값을 다시 지정한다(빈 값/예시 문자열이면 중단).

```bash
read -r -p '33 fixture Region: ' REGION
read -r -p '33 Object Storage namespace: ' NS
[ -n "$REGION" ] && [ -n "$NS" ] || exit 1
export REGION NS
```

각 station은 별도 신규 출력 directory를 만든다. 재시도는 이전 증거를 삭제하거나 기존 경로를 재사용하지 말고 소유 증거를 보존한 뒤 새 빈 작업 directory에서 수행한다. 아래 관리자/Reader/outsider station은 각각 표시된 번호의 작업 직후에만 실행하며 policy/membership를 바꾸지 않는다.

모든 deny는 같은 target 관리자 성공/hash와 한 세트로 기록한다. 각 Reader 시험 직전과 직후33의 A baseline positive를 같은 LAB_READER 인증으로 다시 확인한다. 두 시험 사이 credential/profile, Region/namespace, bucket/key, 객체 존재, policy/membership를 고정한다. baseline positive 실패 또는 관리자 재검증 실패이면 접근 거부 판정은 보류하고33의 정상 상태부터 복구한다. 관리자 재검증은 새 빈 작업 directory에서 위 관리자 station을 다시 실행한다. 뜻밖 allow이면 이 문서는 policy/membership를 변경하지 않았으므로 임의로 권한을 지우지 말고 소유 관리자에게 추가 grant를 조사하도록 요청한다. 관리자 조치 후 baseline positive와 두 deny를 새 증거 경로에서 재시험한다. request-id/서비스 오류가 없으면 deny 완료가 아니다. 관리자 bytes/hash를38 owner가41로 전달한다.

**관리자 PC (Upload 후 단계3):**

```bash
mkdir -m 700 "comp38-admin-child" || exit 1
if timeout 45s oci os object get --profile LAB_ADMIN --region "$REGION" --namespace-name "$NS" --bucket-name iam-child-proof --name proof.txt --file "comp38-admin-child/proof.txt" --max-retries 0 2>"comp38-admin-child/error.txt"; then
  RC=0
  python3 -c 'import pathlib,sys; assert pathlib.Path(sys.argv[1]).read_bytes()==b'"'"'child\n'"'"'' "comp38-admin-child/proof.txt" || exit 1
  sha256sum "comp38-admin-child/proof.txt" || exit 1
else
  RC=$?
  printf 'STOP: expected positive RC=%s\n' "$RC"; exit 1
fi
printf 'GetObject RC=%s\n' "$RC"
mkdir -m 700 "comp38-admin-sibling" || exit 1
if timeout 45s oci os object get --profile LAB_ADMIN --region "$REGION" --namespace-name "$NS" --bucket-name iam-sibling-proof --name proof.txt --file "comp38-admin-sibling/proof.txt" --max-retries 0 2>"comp38-admin-sibling/error.txt"; then
  RC=0
  python3 -c 'import pathlib,sys; assert pathlib.Path(sys.argv[1]).read_bytes()==b'"'"'sibling\n'"'"'' "comp38-admin-sibling/proof.txt" || exit 1
  sha256sum "comp38-admin-sibling/proof.txt" || exit 1
else
  RC=$?
  printf 'STOP: expected positive RC=%s\n' "$RC"; exit 1
fi
printf 'GetObject RC=%s\n' "$RC"
```

**Reader 본인 PC (단계5; 선택 단계4보다 먼저 수행해도 됨):**

```bash
mkdir -m 700 "comp38-reader-child" || exit 1
if timeout 45s oci os object get --profile LAB_READER --region "$REGION" --namespace-name "$NS" --bucket-name iam-child-proof --name proof.txt --file "comp38-reader-child/proof.txt" --max-retries 0 2>"comp38-reader-child/error.txt"; then
  printf '%s\n' 'STOP: unexpected allow; ask owner to investigate additional grants'; exit 1
else
  RC=$?
  printf 'deny candidate RC=%s; inspect request-id and service code\n' "$RC"
  python3 - "comp38-reader-child/error.txt" "$RC" <<'PY'
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
mkdir -m 700 "comp38-reader-sibling" || exit 1
if timeout 45s oci os object get --profile LAB_READER --region "$REGION" --namespace-name "$NS" --bucket-name iam-sibling-proof --name proof.txt --file "comp38-reader-sibling/proof.txt" --max-retries 0 2>"comp38-reader-sibling/error.txt"; then
  printf '%s\n' 'STOP: unexpected allow; ask owner to investigate additional grants'; exit 1
else
  RC=$?
  printf 'deny candidate RC=%s; inspect request-id and service code\n' "$RC"
  python3 - "comp38-reader-sibling/error.txt" "$RC" <<'PY'
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

**USER-tested 선택 시에만 outsider 본인 PC (단계4):**

```bash
mkdir -m 700 "comp38-outsider-child" || exit 1
if timeout 45s oci os object get --profile LAB_OUTSIDER --region "$REGION" --namespace-name "$NS" --bucket-name iam-child-proof --name proof.txt --file "comp38-outsider-child/proof.txt" --max-retries 0 2>"comp38-outsider-child/error.txt"; then
  printf '%s\n' 'STOP: unexpected allow; ask owner to investigate additional grants'; exit 1
else
  RC=$?
  printf 'deny candidate RC=%s; inspect request-id and service code\n' "$RC"
  python3 - "comp38-outsider-child/error.txt" "$RC" <<'PY'
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
mkdir -m 700 "comp38-outsider-sibling" || exit 1
if timeout 45s oci os object get --profile LAB_OUTSIDER --region "$REGION" --namespace-name "$NS" --bucket-name iam-sibling-proof --name proof.txt --file "comp38-outsider-sibling/proof.txt" --max-retries 0 2>"comp38-outsider-sibling/error.txt"; then
  printf '%s\n' 'STOP: unexpected allow; ask owner to investigate additional grants'; exit 1
else
  RC=$?
  printf 'deny candidate RC=%s; inspect request-id and service code\n' "$RC"
  python3 - "comp38-outsider-sibling/error.txt" "$RC" <<'PY'
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

검증 증거는 위의 deny 판정 조건에 따라 보존한다.

### B. 리소스 배치와 접근 범위
4. **선택:** USER-tested를 선택한 경우에만 group 없는 outsider로 child와 sibling 두 get 모두 실패를 확인한다.41에서 child scope 권한을 부여할 때 같은 입력을 사용할 수 있게 namespace/Region/name/OCID를 기록한다.
5. 기존33 Reader가 `iam-proof-a`만 read 조건을 받으므로 child의 새 이름 bucket은 읽지 못하는지 확인한다. 부모 위치 policy라고 모든 child bucket 이름이 조건 없이 허용되는 것이 아니다.
6. Console의 bucket **Compartment**와 **Region**을 함께 적고 VCN/Subnet membership과 비교한다. 실제 Resource 이동은 별도 영향을 줄 수 있으므로 이번 생성 실습에서는 move를 수행하지 않는다.

## 결과검증


### baseline-38-ready


완료 단계 **COMPARTMENT-fixture-ready**: child/sibling ACTIVE, 부모 OCID 동일, bucket owner OCID/Region/hash, 관리자 read 성공과 Reader 두 새 bucket deny. outsider 분기를 선택한 경우에만 두 outsider deny도 필수이고 outsider 비교 생략 경로는 NOT_SELECTED를 기록한다.41을 선택한 경우만 모든 fixture를 반환/expiry까지 보존한다. 미선택이면 이번 두 fixture를 정리하고 parent 사용을33에 반환한다. Console에 compartment가 표시되는 것과 resource 접근 권한은 동일하지 않다.

**실행 기록(아직 미작성):** 실행자/Region/시각, exact resource OCID, 설정 전·후, 성공 출력, deny 오류·exit code/request-id, positive control, 정리 결과. 문서 검증은 실제 OCI 실행 성공을 뜻하지 않는다.

## 문제해결

create403이면 compartment creation 권한과 parent scope를 관리자 확인한다. list에 없으면 compartment filter/Region/resource 생성 상태와 전파 확인. delete pending은 잔여 resource/version/policy/child가 있는지 확인하며 업무 자원을 이동해 억지로 비우지 않는다.

## 리소스정리

41 선택 시 반환/expiry까지 보존하고, 미선택이면 검증 직후 정리한다. 종료 시 child/sibling object와 version 삭제 → bucket 삭제 →41을 선택했다면41 소유자가 자신이 추가한 scope policy만 원복/삭제하고 반환한 증거 확인(미선택이면 policy 변경 없음) → child compartment 삭제 후 상태 확인 → sibling 삭제 → parent iam-lab은 owner33 소유이므로38은 삭제하지 않고33 lease에 RETURNED/expiry 및 child/sibling 정리 결과를 반환. Compartment가 비어 보인다는 Console 한 화면만으로 삭제 완료라 주장하지 않는다.

Compartment 삭제는 **Identity & Security → Compartments**에서 기록한 child/sibling OCID와 parent를 대조한 뒤 각 **Actions → Delete Compartment → OK**로 진행한다. work request와 상태를 확인한다. `Deleting`은 진행 중이고 `Active`로 돌아오면 실패이므로 남은 소유 자원/부착 policy를 확인한다. `Deleted`를 확인한 뒤33에 반환한다. 삭제 실패를 우회하려고 parent 또는 타인 policy를 삭제하지 않는다. [Managing Compartments — Deleting Compartments](https://docs.oracle.com/en-us/iaas/Content/Identity/Tasks/managingcompartments.htm)

**전용 로컬 산출물 정리:** 실행 전에 새로 생성한 exact directory/file 경로를 소유 기록에 남기고, 증거 보존 필요가 끝난 뒤 해당 사용자/host에서 다음 파일만 삭제한다: compartment38-fixtures/child/proof.txt、compartment38-fixtures/sibling/proof.txt 및 comp38-admin/reader/outsider-child/sibling의 proof.txt/error.txt. 실패 요청이 부분 파일을 만들었으면 그것도 이번 owned 파일일 때만 제거한다. symlink/기존 재사용 파일은 삭제하지 않는다. 파일을 하나씩 exact path로 제거한 뒤 자신이 만든 빈 하위 directory부터 `rmdir`한다. retry는 실행 등록된 경로만 순회하고 wildcard/rm -rf는 사용하지 않는다; 미등록/타인 파일 잔존이면 중단한다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 목표는 학습 후 설명·구분·확인할 내용이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [25] [Overview of Identity and Access Management — Compartments](https://docs.oracle.com/en-us/iaas/Content/Identity/Concepts/overview.htm)
  - 확인 범위: COMPARTMENT .

### 실습 작업을 지원하는 추가 공식 본문

- [추가 작업 근거] [Object Storage Buckets](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/managingbuckets.htm) — 이 문서의 실제 생성/호출 작업을 지원하는 공식 본문.
- [추가 작업 근거] [Object Storage Objects](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/managingobjects.htm) — 이 문서의 실제 생성/호출 작업을 지원하는 공식 본문.

- [실습근거 1] [Managing Compartments](https://docs.oracle.com/en-us/iaas/Content/Identity/Tasks/managingcompartments.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
- [실습근거 2] [How Policies Work (without Identity Domains)](https://docs.oracle.com/en-us/iaas/Content/Identity/Concepts/policies.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
- [실습근거 3] [Details for Object Storage and Archive Storage](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
