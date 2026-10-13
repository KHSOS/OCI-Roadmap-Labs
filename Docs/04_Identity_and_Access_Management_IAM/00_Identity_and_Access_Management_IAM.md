# Identity and Access Management (IAM)

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../README.md)를 확인하세요.

> **저술 완료 · OCI 실행 미검증** — Oracle 공식 작업 문서를 근거로 작성한 한국어 hands-on입니다. Oracle 공인 교육/실행 인증이 아니며 기대 결과와 실제 결과를 구분합니다. 비용 승인과 실제 선행 산출물 확인 후에만 실행하세요.

## 학습목표

- 격리 iam-lab과 private Standard Bucket A/B의 proof.txt를 만들고 관리자 다운로드로 두 원본을 확인한다.
- lab-reader/그룹/조건부 policy를 연결하여 Reader A 허용·B 거부와 관리자 same-target B 성공을 기록하고 34–42에 fixture를 인계한다.

완료 기준은 아래 실제 생성·설정·요청 결과로 판정한다. 이 목표는 수행할 작업이며 OCI에서 관찰한 결과가 아니다.

## 대응 Roadmap 항목

- 항목명: **Identity and Access Management (IAM)**
- Roadmap 경로: **Identity and Access Management (IAM)**
- Node ID: `sequence-4-0`
- 상위 항목: 없음 — 최상위 학습 항목
- 문서 범위: 공식 개념 목표와 실제 구성·허용/불허 검증·정리 절차. 아래 실습은 저술되었으나 OCI에서 실행 검증되지 않았습니다.
- [전체 항목 목록](../README.md)

## 사전준비

관리자에게 `LAB_COMPARTMENT_ID`(owner00 ACTIVE lab-root), `REGION`/Home Region/기존 Domain(04)을 받는다. 관리자/Reader 인증은 별도 사용자 PC에서 준비한다. 아래 생성 뒤 `IAM_LAB_OCID`, `LAB_READERS_GROUP_OCID`, 실제 User OCID, `NS`, A/B bucket name·proof bytes/hash와 consumer lease를34–42에 인계한다. VM/IP/앱 포트는 이 fixture에 필요 없다.

### 기준 계약과 실제 단계 선행조건

실제 tenancy/Region/Domain·주체·소유권은 [공통 baseline-contract.json](../../baseline-contract.json)과 위 fixture 계약에 맞춘다. 빈 OCID/placeholder·미승인 권한은 실행 차단 입력이다. 이 API/Console 실습 때문에 VM을 생성하지 않는다.

- [00.ready Introduction — FUNCTIONAL_COMPLETION](../00_Introduction/00_Introduction.md#baseline-00-ready): 표시된 FUNCTIONAL_COMPLETION은 해당 linked ready 단계의 실제 검증/반환이 필요하다. PRE_CREATE_OR_INSTALL_PREPARATION으로 표시한 링크만 준비 출력까지 요구한다. 필요한 OCID/주소만 인계한다.
- [04.ready Introduction to OCI — FUNCTIONAL_COMPLETION](../01_Introduction_to_OCI/00_Introduction_to_OCI.md#baseline-04-ready): 표시된 FUNCTIONAL_COMPLETION은 해당 linked ready 단계의 실제 검증/반환이 필요하다. PRE_CREATE_OR_INSTALL_PREPARATION으로 표시한 링크만 준비 출력까지 요구한다. 필요한 OCID/주소만 인계한다.

- [Introduction to OCI](../01_Introduction_to_OCI/00_Introduction_to_OCI.md): **Tenancy/Home Region/실습 Region 확인** 완료. 관리자가 제공한 실습 Compartment가 있으면 그 OCID를 재사용한다. 없다면 아래 A에서 `iam-lab`를 생성한다. [Compartments](05_Compartments.md) 전체 실습을 미리 완료할 필요는 없다.
- 관리자와 신규 테스트 사용자용 브라우저 프로필/시크릿 창을 분리한다. 이메일 수신 가능한 테스트 주소 두 개를 준비한다. 기존 업무 사용자 또는 관리자 그룹을 변경하지 않는다.
- [Introduction to OCI bucket 생성](../01_Introduction_to_OCI/00_Introduction_to_OCI.md#baseline-04-ready)의 private Standard/PUT/GET 절차를 참고한다. 이 IAM fixture는 아래 B에서 독립 owner33으로 두 bucket을 생성·검증하며48/49 완료를 선행 요구하지 않는다. 여기서 만든 fixture는34~42까지 보존한다.

## 권한

### 실제 task/API별 공식 권한 본문 대조

아래 공식 operation 표에서 이 실습의 읽기·변경·정리 권한을 대조한다. family grant/일부 dependency 인용은 검증된 minimum policy가 아니며 실제 principal과 resource scope를 관리자가 승인해야 한다. 조직 Identity Domain Console user/group 작업은 해당 Domain 관리자 역할도 필요하며 legacy IAM user API 권한만으로 대체하지 않는다.

- `CreateBucket`: BUCKET_CREATE If the KMS Key ID is provided to the operation, the following additional permissions are required: KEY_ASSOCIATE The objectstorage-<location> subject must also have: KEY_ENCRYPT, KEY_DECRYPT, KEY_READ. — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm).
- `CreateCompartment`: COMPARTMENT_CREATE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/policyreference/iampolicyreference.htm).
- `CreatePolicy`: POLICY_CREATE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/policyreference/iampolicyreference.htm).
- `DeleteBucket`: BUCKET_DELETE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm).
- `DeleteCompartment`: COMPARTMENT_DELETE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/policyreference/iampolicyreference.htm).
- `DeleteObject`: OBJECT_DELETE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm).
- `DeletePolicy`: POLICY_DELETE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/policyreference/iampolicyreference.htm).
- `GetObject`: OBJECT_READ For a customer-managed key encrypted bucket, the objectstorage-<location> subject must have KEY_DECRYPT. — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm).
- `PutObject`: The permission required depends on whether the object already exists in the bucket: OBJECT_CREATE is required when an object with that name doesn't already exist in the bucket. OBJECT_OVERWRITE is required when an object with that name already exists in the bucket. For a customer-managed key encrypted bucket, the objectstorage-<location> subject must have KEY_ENCRYPT. — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm).

관리자만 User·Group·Dynamic Group·Policy·Compartment를 생성하거나 멤버십을 수정한다. Identity Domains의 사용자/그룹 관리에는 해당 Domain의 적절한 관리자 역할이 필요하고 OCI 리소스 Policy 관리 권한은 별개다. 기존 Administrators 또는 위임된 Domain 관리자에게 승인받는다. 학습자에게 `manage all-resources in tenancy`를 주는 방식으로 우회하지 않는다. 실습 작업자는 아래 절차에 명시한 Object Storage 읽기 권한만 받는다. [Overview of IAM](https://docs.oracle.com/en-us/iaas/Content/Identity/getstarted/identity-domains.htm)의 **The Administrators Group, Policy, and Administrator Roles**, [Managing Policies](https://docs.oracle.com/en-us/iaas/Content/Identity/Tasks/managingpolicies.htm)의 **Required IAM Policy**를 읽고 관리 주체와 검증 주체를 기록한다.

## 비용및주의사항

신규 유료 데이터 범위는 private Standard Bucket A/B와 각 `proof.txt`뿐이다. 두 bucket의 저장 bytes·PUT/GET/LIST 요청·실제 다운로드 경로의 전송 비용을 승인한다. User/Group/Policy 학습을 위해 VM·VCN·앱·LB를 만들지 않는다. fixture는 34–42 및 등록된 31 consumer의 동일 승인 lease까지 보존하며, 연장 없이 만료하면 소비자 복구/반환을 받은 뒤 owner33이 정리한다. Versioning/Retention/Replication/PAR/고객관리 키는 선택하지 않는다. 관리자 성공을 Reader 권한 증거로 사용하지 않는다.

## 실습구성

### 이번 workshop 시나리오와 입력 계약

격리 Compartment·User·Group·두 private Bucket를 만들고 Reader A/B 접근을 비교한다.

|입력|값/출처|시점|실행 위치|선택 이유|
|---|---|---|---|---|
|`LAB_COMPARTMENT_ID`|owner0의 actual output; `<...>` 대체 필수|실행 전 선행 완료값|Console/사용자 전용 shell|정확한 대상/소유관계 고정|
|`REGION`|owner4의 actual output; `<...>` 대체 필수|실행 전 선행 완료값|Console/사용자 전용 shell|정확한 대상/소유관계 고정|
|`IAM_LAB_OCID`|owner33의 actual output; `<...>` 대체 필수|생성 후 readback|Console/사용자 전용 shell|정확한 대상/소유관계 고정|
|`LAB_READERS_GROUP_OCID`|owner33의 actual output; `<...>` 대체 필수|생성 후 readback|Console/사용자 전용 shell|정확한 대상/소유관계 고정|
|`NS`|owner33의 actual output; `<...>` 대체 필수|생성 후 readback|Console/사용자 전용 shell|정확한 대상/소유관계 고정|

|Console field|실제 입력/선택|이유|
|---|---|---|
|Compartment parent / Group / bucket tier/access|iam-lab parent lab-root; lab-readers; Standard/Private/Oracle-managed; no versioning|이번 기능/실패 원인을 한 변수로 분리|

**생성·변경 범위:** iam-lab:Compartment,lab-readers:IdentityGroup,lab-reader:IdentityUser,iam-proof-a:Bucket,iam-proof-b:Bucket,p-lab-read-a:IamPolicy. **완료 비교:** Reader A success/B deny; admin same B success. 기대 패턴이며 OCI 실측값이 아니다. **정리:** 본 주제의 변경만 원복하고 전용 신규 resource만 dependency 역순으로 정리한다; 공유 자원은 creator와 consumer registry/expiry 기준으로 반환한다.

`iam-lab` Compartment, `lab-readers` Group, `lab-reader` User, private Standard Bucket `iam-proof-a`/`iam-proof-b`, 각 `proof.txt`를 만든다. 읽기 주체·허용 대상·허용 작업을 고정해 IAM deny를 네트워크 실패와 구별한다. Bucket 이름이 이미 사용되면 서로 대소문자 외에도 구별되는 고유 이름으로 바꾸고 후속 문서에서 일괄 반영한다.

## 실습절차

### 개별 완주 보정 — 생성·검증·소유권

아래 CLI 인증 준비를 B-2의 첫 관리자 CLI get보다 먼저 완료한다. 별도 사용자 config의 fingerprint/User OCID를 각자 확인한 뒤 새 전용 다운로드 디렉터리에 a.txt/b.txt를 생성한다. 관리자는 B를 같은 namespace/Region/name으로 읽는다.

**owner33 fixture lease (새 자원 없음):** 첫 fixture 생성/대여 전 아래 표를 실제 값으로 채운다. 미선택은 NOT_SELECTED, 선택은 소비자 owner/실제 필요 object·계정/EXPIRY_UTC/RETURNED 또는 ABORTED 복구증거를 기록한다. 값 미기록이면 보존/삭제 모두 새 승인 전 중단한다.

|consumer|사용 범위|owner/expiry/반환|
|---|---|---|
|34 Users|A fixture/Reader policy/outsider 준비|34, 실제 승인시각, 미작성|
|35 Groups|A/B fixture|35, 실제 승인시각, 미작성|
|36 Dynamic Groups +31 SGW|A/B principal/hash|36/31, 실제 승인시각, 미작성|
|37 Policies|A/B fixture/outsider|37, 실제 승인시각, 미작성|
|38 Compartments +41 scope|부모 baseline/Reader, child는38 owner|38/41, 실제 승인시각, 미작성|
|39 OCID|iam-lab/Group/User, 선택 outsider|39, 실제 승인시각, 미작성|
|40 Conditions|A/B fixture/outsider|40, 실제 승인시각, 미작성|
|42 Functions|A/B fixture, 별도 Function42 owner|42, 실제 승인시각, 미작성|

expiry에 신규 요청을 중단하고 각 consumer의 정책/membership 복구/RETURNED 또는 승인 ABORTED를 받아 owner33만 정리한다. 31의 lease는34~42 숫자 범위 밖이어도 반드시 반환받는다. 비용 deadline 연장은 별도 승인 없이는 금지다.



### A. Identity와 격리 범위 생성
1. 관리자로 **Identity & Security → Compartments → Create compartment**: 이름 `iam-lab`, Description `OCI IAM learning only`, 상위는 owner00의 ACTIVE `lab-root`(LAB_COMPARTMENT_ID). 기존 교육 compartment면 재사용한다. OCID/parent를 기록한다.
2. **Identity & Security → Domains → 사용할 Domain → Groups → Create group**: `lab-readers`, Description `Read one lab bucket`, **User can request access 해제**. Administrators에 연결하지 않는다.
3. 같은 Domain **Users → Create**: First name `Lab`, Last name `Reader`, 이메일 실제 테스트 주소, Use email address as username 여부를 선택/기록한다. `lab-readers`만 선택한다. 활성화 메일로 사용자 본인이 password/MFA를 설정한다. password/API private key를 이 문서에 기록하지 않는다.

### CLI 인증 준비 (첫 관리자 API 호출 전 필수)

본인 PC 또는 사용자별 격리 shell에서 [CLI Quickstart](https://docs.oracle.com/en-us/iaas/Content/API/SDKDocs/cliinstall.htm)로 CLI를 설치하고 `oci --version`을 확인한다. local API private key는 본인만 읽을 수 있게 두고 compute VM/공개 저장소로 복사하지 않는다. 관리자/reader/outsider가 서로의 private key를 공유하지 않는다.

```bash
# setup interactive prompt의 config path는 존재하지 않는 전용 파일로 지정; 기존 config/key overwrite 요청이면 중단
oci setup config
# 테스트 사용자 본인이 자신의 실제 User OCID/Tenancy OCID/Region 입력
# 생성된 API public key만 Console User → API keys → Add API key에 등록
# config의 fingerprint와 Console fingerprint 일치 확인
# setup 뒤 editor에서 이번 생성 profile header [DEFAULT]를 [LAB_READER]로 지정
# 기존 profile이 있으면 덮어쓰지 않고 별도 파일로 생성 후 필요한 section만 병합
export THIS_USER_CONFIG_PATH='<setup에서_선택한_전용_신규_config_경로>'
export OCI_CLI_CONFIG_FILE="$THIS_USER_CONFIG_PATH"
export THIS_USER_GENERATED_PRIVATE_KEY_PATH='<actual-generated-private-key-path>'
chmod 600 "$THIS_USER_CONFIG_PATH" "$THIS_USER_GENERATED_PRIVATE_KEY_PATH"
```

CLI setup의 현재 command reference에는 `--profile-name` 옵션이 없으므로 그런 옵션을 가정하지 않는다. 생성 config의 profile section 이름을 local editor로 명시한다. 같은 방법으로 실제 각 사용자 본인이 `LAB_ADMIN`, 후속34의 `LAB_OUTSIDER` profile을 준비한다. 공용 host가 아닌 경우 타 사용자의 profile을 자신의 config에 억지로 모으지 말고 사용자별 shell에서 같은 이름의 profile을 사용한다. CLI authentication command `--profile`에 지정한 user/fingerprint가 해당 사용자라는 확인이 선행이다. Profile 생성/키 업로드는 본인 UI에서 수행한다.


### B. 작은 Object Storage fixture 생성
4. **Storage → Object Storage & Archive Storage → Buckets → iam-lab → Create bucket**: `iam-proof-a`, Default Storage Tier **Standard**, Access **Private**, Encryption **Oracle-managed**, Versioning off, auto-tiering off. 같은 값으로 `iam-proof-b` 생성. retention/replication/PAR는 만들지 않는다.
5. 먼저 아래 B-2의 로컬 fixture 준비를 수행한 뒤 각각 **Objects → Upload**에서 비민감 `proof.txt`를 올린다. 내용은 A `iam-proof-a`, B `iam-proof-b` 한 줄로 달리한다. 관리자 get/download와 hash를 저장해 fixture 존재를 먼저 확인한다. Namespace/Region/Bucket명/object명은 다음 테스트의 고정 입력이다.

### B-2. 정확한 fixture bytes 준비·관리자 positive

본인 PC에서 아래 전용 directory를 만들고 두 proof.txt를 각각 해당 bucket의 **Objects → Upload → Object name proof.txt**로 올린다. 기존 object면 owner33가 내용 확인 후 재사용하며 임의 덮어쓰지 않는다. 관리자 LAB_ADMIN 인증을 아래 CLI 준비 station에서 먼저 완료하고 REGION/NS는 Console의 동일 Region/namespace를 지정한다.

```bash
# 로컬 비민감 synthetic bytes 준비; OCI 호출 아님
mkdir -m 700 iam33-fixtures || exit 1
mkdir iam33-fixtures/a iam33-fixtures/b || exit 1
printf '%s\n' iam-proof-a > iam33-fixtures/a/proof.txt || exit 1
printf '%s\n' iam-proof-b > iam33-fixtures/b/proof.txt || exit 1
sha256sum iam33-fixtures/a/proof.txt iam33-fixtures/b/proof.txt || exit 1
```

Upload와 각 사용자 인증 준비 뒤 **관리자 별도 PC shell**에서:

```bash
mkdir -m 700 "iam33-admin-a" || exit 1
if timeout 45s oci os object get --profile LAB_ADMIN --region "$REGION" --namespace-name "$NS" --bucket-name iam-proof-a --name proof.txt --file "iam33-admin-a/proof.txt" --max-retries 0 2>"iam33-admin-a/error.txt"; then
  RC=0
  python3 -c 'import pathlib,sys; assert pathlib.Path(sys.argv[1]).read_bytes()==b'"'"'iam-proof-a\n'"'"'' "iam33-admin-a/proof.txt" || exit 1
  sha256sum "iam33-admin-a/proof.txt" || exit 1
else
  RC=$?
  printf 'STOP: expected positive RC=%s\n' "$RC"; exit 1
fi
printf 'GetObject RC=%s\n' "$RC"
mkdir -m 700 "iam33-admin-b" || exit 1
if timeout 45s oci os object get --profile LAB_ADMIN --region "$REGION" --namespace-name "$NS" --bucket-name iam-proof-b --name proof.txt --file "iam33-admin-b/proof.txt" --max-retries 0 2>"iam33-admin-b/error.txt"; then
  RC=0
  python3 -c 'import pathlib,sys; assert pathlib.Path(sys.argv[1]).read_bytes()==b'"'"'iam-proof-b\n'"'"'' "iam33-admin-b/proof.txt" || exit 1
  sha256sum "iam33-admin-b/proof.txt" || exit 1
else
  RC=$?
  printf 'STOP: expected positive RC=%s\n' "$RC"; exit 1
fi
printf 'GetObject RC=%s\n' "$RC"
```

각 관리자 다운로드 hash를 원본 local proof.txt hash와 비교하고 A 실제 SHA256를31/36/42에 전달한다. admin도 실패하면 Reader deny 시험을 시작하지 않는다.

### C. 최소 읽기 policy와 주체별 테스트
6. **Identity & Security → Policies → iam-lab → Create Policy**, 이름 `p-lab-read-a`, manual editor에 실제 Group OCID/Compartment OCID를 넣는다.

```text
Allow group id <LAB_READERS_GROUP_OCID> to inspect buckets in compartment id <IAM_LAB_OCID>
Allow group id <LAB_READERS_GROUP_OCID> to read objects in compartment id <IAM_LAB_OCID> where target.bucket.name = 'iam-proof-a'
```



7. 별도 세션 `lab-reader`로 로그인해 Bucket A Objects에서 `proof.txt`를 다운로드한다. Bucket B의 동일 이름 object를 다운로드하면 허용되지 않아야 한다. Console가 메타데이터 추가 권한 때문에 목록 표시를 실패하면 **목록 실패를 GetObject deny로 단정하지 않는다**. 사용자 전용 API config를 본인 PC에 설정한 경우 아래 GetObject로 직접 검증한다. 세션/CLI config의 user OCID가 실제 `lab-reader`인지 확인한다.

```bash
set -euo pipefail
IAM_GET_WORK=$(mktemp -d "$HOME/oci-iam33.XXXXXXXX") || exit 1
timeout -k 5s 45s oci os object get --profile LAB_READER --region '<REGION>' --namespace-name '<NS>' --bucket-name iam-proof-a --name proof.txt --file "$IAM_GET_WORK/a.txt" --max-retries 0 || exit 1
python3 -c 'import pathlib,sys; assert pathlib.Path(sys.argv[1]).read_bytes()==b"iam-proof-a\n"' "$IAM_GET_WORK/a.txt" || exit 1
sha256sum "$IAM_GET_WORK/a.txt" || exit 1
# 실제 owner33 admin A SHA256와 동일해야 한다.
ERR="$IAM_GET_WORK/b-error.json"
if timeout -k 5s 45s oci os object get --profile LAB_READER --region '<REGION>' --namespace-name '<NS>' --bucket-name iam-proof-b --name proof.txt --file "$IAM_GET_WORK/b.txt" --max-retries 0 2>"$ERR"; then
  printf 'STOP: unexpected B allow\n'; exit 1
else rc=$?; fi
python3 - "$rc" "$ERR" <<'PY'
import sys,json
assert int(sys.argv[1]) not in (0,124,137)
s=open(sys.argv[2]).read(); x=json.loads(s[s.index('{'):s.rindex('}')+1])
assert x.get('status') in (403,404) and x.get('code') in ('NotAuthorized','NotAuthorizedOrNotFound')
PY
[ "$?" = 0 ] || exit 1

```


8. 관리자 세션에서 Bucket B의 같은 object를 즉시 읽어 positive control을 남긴다. 허용 정책/Group/멤버십·오류 request-id·시간을 기록한다. 다른 상위 정책이 학습자에 권한을 주는지 관리자가 확인한다.

## 결과검증

<a id="baseline-33-ready"></a>

완료 단계 **IAM-fixture-ready**: 실제 Compartment/User/Group/Bucket 존재, Reader의 A get 성공·B get 실패, 관리자의 B get 성공. `inspect buckets`는 ListBuckets, `read objects`는 ListObjects/GetObject를 포함하는 공식 reference를 대조한다. Bucket의 private 설정만으로 IAM 정책 없이 사용자가 읽을 수 있는 것이 아니다.

**실행 기록(아직 미작성):** 실행자/Region/시각, exact resource OCID, 설정 전·후, 성공 출력, deny 오류·exit code/request-id, positive control, 정리 결과. 문서 검증은 실제 OCI 실행 성공을 뜻하지 않는다.

## 문제해결

401/auth 오류는 credentials/MFA/profile/clock부터, 404 NotAuthorizedOrNotFound는 동일 Region·namespace·case-sensitive object·관리자 positive control부터 확인한다. 정책 문법·Group id·membership·상위 policy와 적용 전파를 확인한다. Domain 관리자 역할은 Object Storage read 정책을 대신하지 않는다. Console bucket 목록을 보고 다운로드 성공이라고 결론내리지 않는다.

## 리소스정리

registry에 기록한 신규 IAM_GET_WORK의 이번 다운로드만 제거하고 empty directory rmdir한다. 다른 사용자 config/key/fixture는 삭제하지 않는다.

34~42가 남으면 fixture와 Reader 계정을 보존한다. 종료 시 모든 테스트 정책 제거 → API key/auth token revoke → Group membership 제거 → test User 삭제 → Group 삭제 → object(모든 version 여부 확인) 삭제 → Bucket 삭제 → 남은 VM/Function/FSS resource 확인 → 빈 compartment 삭제. 업무 Domain 전체를 삭제하지 않는다.

**전용 로컬 산출물 정리:**실행 전에 새로 생성한 exact directory/file 경로를 소유 기록에 남기고, 증거 보존 필요가 끝난 뒤 해당 사용자/host에서 다음 파일만 삭제한다: iam33-fixtures/a/proof.txt、iam33-fixtures/b/proof.txt、iam33-admin-a/proof.txt/error.txt、iam33-admin-b/proof.txt/error.txt 및 IAM_GET_WORK의 a.txt/b.txt/b-error.json. 실패 요청이 부분 파일을 만들었으면 그것도 이번 owned 파일일 때만 제거한다. symlink/기존 재사용 파일은 삭제하지 않는다. 파일을 하나씩 exact path로 제거한 뒤 자신이 만든 빈 하위 directory부터 `rmdir`한다. retry는 실행 등록된 경로만 순회하고 wildcard/rm -rf는 사용하지 않는다; 미등록/타인 파일 잔존이면 중단한다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 목표는 학습 후 설명·구분·확인할 내용이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [25] [Overview of Identity and Access Management — Identity and Access Management (IAM)](https://docs.oracle.com/en-us/iaas/Content/Identity/Concepts/overview.htm)
  - 확인 범위: Oracle Cloud Infrastructure Identity.

### 실습 작업을 지원하는 추가 공식 본문

- [추가 작업 근거] [Object Storage Buckets](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/managingbuckets.htm) — 이 문서의 실제 생성/호출 작업을 지원하는 공식 본문.
- [추가 작업 근거] [Object Storage Objects](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/managingobjects.htm) — 이 문서의 실제 생성/호출 작업을 지원하는 공식 본문.

- [실습근거 1] [Creating a User](https://docs.oracle.com/en-us/iaas/Content/Identity/users/create-user-accounts.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
- [실습근거 2] [Creating a Group](https://docs.oracle.com/en-us/iaas/Content/Identity/groups/create-groups.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
- [실습근거 3] [Managing Policies](https://docs.oracle.com/en-us/iaas/Content/Identity/Tasks/managingpolicies.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
- [실습근거 4] [Details for Object Storage and Archive Storage](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
