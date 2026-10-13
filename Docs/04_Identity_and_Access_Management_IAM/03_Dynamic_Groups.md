# Dynamic Groups

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../README.md)를 확인하세요.

> **저술 완료 · OCI 실행 미검증** — Oracle 공식 작업 문서를 근거로 작성한 한국어 hands-on입니다. Oracle 공인 교육/실행 인증이 아니며 기대 결과와 실제 결과를 구분합니다. 비용 승인과 실제 선행 산출물 확인 후에만 실행하세요.

## 학습목표

- 기존 private-vm OCID 하나만 매칭하는 Dynamic Group과 A 읽기 policy를 생성한다.
- 사용자 키 없이 B에서 instance_principal A GET/hash와 B 거부를 확인하고 기존 vm-a 비회원 요청을 대조한다.

완료 기준은 아래 실제 생성·설정·요청 결과로 판정한다. 이 목표는 수행할 작업이며 OCI에서 관찰한 결과가 아니다.

## 대응 Roadmap 항목

- 항목명: **Dynamic Groups**
- Roadmap 경로: **Identity and Access Management (IAM) → Dynamic Groups**
- Node ID: `sequence-4-3`
- 상위 항목: **Identity and Access Management (IAM)** (`sequence-4-0`)
- 문서 범위: 공식 개념 목표와 실제 구성·허용/불허 검증·정리 절차. 아래 실습은 저술되었으나 OCI에서 실행 검증되지 않았습니다.
- [전체 항목 목록](../README.md)

## 사전준비

33의 REGION/NS/IAM_LAB_OCID/A/B 실제 proof hash,24의 A/B SSH 정상,10의 INSTANCE_A_ID와22의 INSTANCE_B_ID를 받는다.30 설치 준비 링크는 설치용이고 B API 호출 전에는 본문 full-test 재진입의 실제 NAT route/443 egress readback이 추가로 필요하다. instance_principal API에는 user private key가 없다.31 SGW-only 전환 전에 A/B principal 결과와 gateway/fixture lease를 반환한다.

### 기준 계약과 실제 단계 선행조건

[공통 baseline-contract.json](../../baseline-contract.json)의 OS/CIDR/소유권을 따른다. Oracle Linux 9 / `opc` / `dnf` 외의 숨은 Ubuntu 또는 probe VM은 만들지 않는다. OCID/IP의 null은 실행 전 반드시 실제 출력으로 채우는 **실행 차단 입력**이다. approval/capacity/quota/identity 미확인은 완료가 아니다.

- [33.ready Identity and Access Management (IAM) — FUNCTIONAL_COMPLETION](00_Identity_and_Access_Management_IAM.md#baseline-33-ready): 표시된 FUNCTIONAL_COMPLETION은 해당 linked ready 단계의 실제 검증/반환이 필요하다. PRE_CREATE_OR_INSTALL_PREPARATION으로 표시한 링크만 준비 출력까지 요구한다. 필요한 OCID/주소만 인계한다.
- [24.ready Route Tables — FUNCTIONAL_COMPLETION](../03_Virtual_Cloud_Network_VCN/05_Route_Tables.md#baseline-24-ready): 표시된 FUNCTIONAL_COMPLETION은 해당 linked ready 단계의 실제 검증/반환이 필요하다. PRE_CREATE_OR_INSTALL_PREPARATION으로 표시한 링크만 준비 출력까지 요구한다. 필요한 OCID/주소만 인계한다.
- [30.prepare NAT Gateway — PRE_CREATE_OR_INSTALL_PREPARATION](../03_Virtual_Cloud_Network_VCN/11_NAT_Gateway.md#nat-installation-preparation): 표시된 FUNCTIONAL_COMPLETION은 해당 linked ready 단계의 실제 검증/반환이 필요하다. PRE_CREATE_OR_INSTALL_PREPARATION으로 표시한 링크만 준비 출력까지 요구한다. 필요한 OCID/주소만 인계한다.

공유 A/B consumer는 INSTANCE_A_ID에 owner10 vm-a의 실제 OCID, INSTANCE_B_ID에 owner22 private-vm의 실제 OCID, `A_PUBLIC_IP`, `A_PRIVATE_IP`, `B_PRIVATE_IP`를 실제 생성 응답에서 전달받는다. 해당 주제에 필요하지 않은 VM/변수는 생성·요구하지 않는다.

- [Identity and Access Management (IAM)](00_Identity_and_Access_Management_IAM.md) **IAM-fixture-ready** 완료. [Route Tables](../03_Virtual_Cloud_Network_VCN/05_Route_Tables.md) **RT-ready** private B VM와 SSH 정상. CLI 설치용 NAT가 필요하면 [NAT 설치 준비](../03_Virtual_Cloud_Network_VCN/11_NAT_Gateway.md#nat-installation-preparation) **A 단계**의 경로를 일시 사용한다.
- CLI는 B에 설치하되 사용자 API private key/config는 복사하지 않는다. 두 번째 비회원 VM은 A를 재사용한다. Instance principal 지원 CLI 설치와 object fixture 존재를 먼저 확인한다.
- 실습은 Compute instance용 matching rule을 먼저 작성하고42에서 Function resource용 rule을 별도로 작성한다.42 전체 실행을 선행 요구하지 않는다.

## 권한

### 실제 task/API별 공식 권한 본문 대조

아래 공식 operation 표에서 이 실습의 읽기·변경·정리 권한을 대조한다. family grant/일부 dependency 인용은 검증된 minimum policy가 아니며 실제 principal과 resource scope를 관리자가 승인해야 한다. 조직 Identity Domain Console user/group 작업은 해당 Domain 관리자 역할도 필요하며 legacy IAM user API 권한만으로 대체하지 않는다.

- `CreateDynamicGroup`: DYNAMIC_GROUP_CREATE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/policyreference/iampolicyreference.htm).
- `CreatePolicy`: POLICY_CREATE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/policyreference/iampolicyreference.htm).
- `DeleteDynamicGroup`: DYNAMIC_GROUP_DELETE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/policyreference/iampolicyreference.htm).
- `DeletePolicy`: POLICY_DELETE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/policyreference/iampolicyreference.htm).
- `GetObject`: OBJECT_READ For a customer-managed key encrypted bucket, the objectstorage-<location> subject must have KEY_DECRYPT. — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm).

관리자만 User·Group·Dynamic Group·Policy·Compartment를 생성하거나 멤버십을 수정한다. Identity Domains의 사용자/그룹 관리에는 해당 Domain의 적절한 관리자 역할이 필요하고 OCI 리소스 Policy 관리 권한은 별개다. 기존 Administrators 또는 위임된 Domain 관리자에게 승인받는다. 학습자에게 `manage all-resources in tenancy`를 주는 방식으로 우회하지 않는다. 실습 작업자는 아래 절차에 명시한 Object Storage 읽기 권한만 받는다. [Overview of IAM](https://docs.oracle.com/en-us/iaas/Content/Identity/getstarted/identity-domains.htm)의 **The Administrators Group, Policy, and Administrator Roles**, [Managing Policies](https://docs.oracle.com/en-us/iaas/Content/Identity/Tasks/managingpolicies.htm)의 **Required IAM Policy**를 읽고 관리 주체와 검증 주체를 기록한다.

## 비용및주의사항

기존 vm-a와 private-vm, owner33의 두 bucket을 재사용하며 신규 VM/앱/HTTP endpoint는 없다. principal GET/다운로드·fixture 저장, 두 VM/boot volume 유지 및 승인 NAT 설치/API 경로 비용을 확인한다. 기본 호출은 상태당 1–3회이며 matching-rule 변경 branch의 최대75회/75분은 별도 승인 lease에서만 선택한다. 전파 대기 때문에 VM/NAT/fixture를 무기한 유지하지 않으며 owner30/31/33과 반환 시각을 맞춘다. SSH22와 기존 보안 규칙을 보존한다.

## 실습구성

### 이번 workshop 시나리오와 입력 계약

exact B instance Dynamic Group와 read policy를 만들고 VM workload identity로 get한다.

|입력|값/출처|시점|실행 위치|선택 이유|
|---|---|---|---|---|
|`INSTANCE_A_ID`|owner10의 actual output; `<...>` 대체 필수|실행 전 선행 완료값|Console/사용자 전용 shell|정확한 대상/소유관계 고정|
|`INSTANCE_B_ID`|owner22의 actual output; `<...>` 대체 필수|실행 전 선행 완료값|Console/사용자 전용 shell|정확한 대상/소유관계 고정|
|`IAM_LAB_OCID`|owner33의 actual output; `<...>` 대체 필수|실행 전 선행 완료값|Console/사용자 전용 shell|정확한 대상/소유관계 고정|
|`NS`|owner33의 actual output; `<...>` 대체 필수|실행 전 선행 완료값|Console/사용자 전용 shell|정확한 대상/소유관계 고정|
|`DG_PRIVATE_VM_OCID`|owner36의 actual output; `<...>` 대체 필수|생성 후 readback|Console/사용자 전용 shell|정확한 대상/소유관계 고정|

|Console field|실제 입력/선택|이유|
|---|---|---|
|Dynamic Group rule / policy subject / auth|instance.id exact B OCID; dynamic-group id; instance_principal|이번 기능/실패 원인을 한 변수로 분리|

**생성·변경 범위:** dg-private-vm:DynamicGroup,p-instance-read-a:IamPolicy. **완료 비교:** B A get allow/B get deny; nonmember VM A deny. 기대 패턴이며 OCI 실측값이 아니다. **정리:** 본 주제의 변경만 원복하고 전용 신규 resource만 dependency 역순으로 정리한다; 공유 자원은 creator와 consumer registry/expiry 기준으로 반환한다.

Dynamic Group `dg-private-vm` 하나, **exact instance OCID** matching rule 하나, Bucket A read 정책을 생성한다. Compartment 전체의 모든 instance를 자동 포함시키지 않는다. 실제 VM에서 `--auth instance_principal` GetObject를 호출하여 workload identity를 검증한다.

## 실습절차

### 작업 순서 체크포인트
exact instance membership와 bucket-scoped permission을 먼저 구성하고 같은 fixture의 principal별 새 GET/서비스 오류를 확인한다. NAT는 API 검증 뒤에만31로 반환한다. 단계 실패는 다음 생성으로 넘기지 않고 같은 target의 복구/반환으로 처리한다.

### 개별 완주 보정 — 생성·검증·소유권

NAT 설치 후 CLI가 존재한다는 사실만으로 API 네트워크가 준비되지 않는다. 본 시험은 NAT default route/egress443가 활성인 동안 두 bucket get을 먼저 검증한다. 31로 넘길 때만 NAT route를 제거하고 SGW-only 경로로 전환한다. Download 결과는 exit code=0의 새 파일만 hash한다.

**순서 고정:**CLI 설치 완료값은 network-ready가 아니다. [29 Internet Gateway](../03_Virtual_Cloud_Network_VCN/10_Internet_Gateway.md#baseline-29-ready)의 A public 경로를 완료하고, [30 NAT Gateway](../03_Virtual_Cloud_Network_VCN/11_NAT_Gateway.md)의 **full-test 재진입**에서 owner30가 승인한 B default-route tuple 및 정확한 egress443를 복원/readback한 뒤 B API를 시작한다. 30.prepare 종료의 제거된 경로를 그대로 사용하지 않는다. 승인 route/egress OCID·rule ID/UTC lease를 owner30와 기록하고 단계5/6 후에만31로 반환한다. 네트워크 복원이 미확인이면 BLOCKED_NETWORK로 중단한다.

### A. Dynamic Group과 최소 정책
1. **Compute → private-vm → Details**에서 instance OCID를 복사한다. VNIC OCID를 넣지 않는다.
2. **Identity & Security → Domains → 사용할 Domain → Dynamic groups → Create Dynamic Group**: `dg-private-vm`, Description `Exact B instance read proof`, matching rule:

```text
instance.id = '<INSTANCE_B_ID>'
```

3. 관리자 **Policies → iam-lab → Create**: `p-instance-read-a`. Dynamic Group 이름/Domain 표현이 환경마다 달라 혼동될 수 있으므로 공식 Policy Syntax의 `dynamic-group id`로 실제 Group OCID를 사용한다.

```text
Allow dynamic-group id <DG_PRIVATE_VM_OCID> to read objects in compartment id <IAM_LAB_OCID> where target.bucket.name = 'iam-proof-a'
```

Object namespace는 관리자가33에서 기록해 전달한다. instance에 namespace 조회를 위해 불필요한 tenancy 전체 read를 주지 않는다.

### B. 실제 instance principal 호출
4. B에 OCI CLI가 없는 경우 설치된 패키지/관리자 승인 방식으로 설치하고 `oci --version`을 기록한다. 설치 중 사용하는 NAT는 CLI 설치 후31의 SGW-only 검증에서 제거한다. Shell의 `OCI_CLI_AUTH`, `OCI_CLI_PROFILE`, config 환경변수를 확인한다. Oracle Linux 9 VM에서 CLI가 없으면 [OCI CLI Quickstart — Linux and UNIX](https://docs.oracle.com/en-us/iaas/Content/API/SDKDocs/cliinstall.htm)의 공식 installer를 **보존한 NAT 경로로** 내려받아 검토한 뒤 실행한다. 설치 prompt의 경로는 `$HOME` 내부를 선택한다.

```bash
set -euo pipefail
INSTALL_WORK=$(mktemp -d "$HOME/oci-cli-install36.XXXXXXXX") || exit 1
curl --fail --location --show-error --silent https://raw.githubusercontent.com/oracle/oci-cli/master/scripts/install/install.sh -o "$INSTALL_WORK/install.sh" || exit 1
test -s "$INSTALL_WORK/install.sh" || exit 1
sha256sum "$INSTALL_WORK/install.sh"
# Mutable upstream: this hash is identity evidence, not authenticity approval.
# Stop for owner review of exact body/hash and official URL before running.
read -r -p 'Owner-reviewed installer body/hash? type APPROVED: ' reviewed
test "$reviewed" = APPROVED || exit 1
bash "$INSTALL_WORK/install.sh"
# 설치 prompt의 PATH 반영 후 새 shell 또는 아래 경로 사용
export PATH="$HOME/bin:$PATH"
oci --version
```

Instance principal은 User API config 없이 실행하며 `oci setup config`로 User 키를 VM에 만들지 않는다. A 비회원 VM에서도 CLI를 동일하게 준비한다. Installation network가 없으면 먼저30의 임시 NAT를 승인받고 준비 후31의 route 제거를 수행한다.

```bash
set -euo pipefail
GET_WORK=$(mktemp -d "$HOME/oci-principal36.XXXXXXXX") || exit 1
# New directory per principal/state; record exact path and owner before API tests.
timeout -k 5s 45s oci os object get --auth instance_principal --region '<REGION>' --namespace-name '<NS>' --bucket-name iam-proof-a --name proof.txt --file "$GET_WORK/instance-a.txt" --max-retries 0
test "$?" -eq 0 || { printf 'STOP: failed A GET; no old-file hash\n'; exit 1; }
python3 -c 'import pathlib,sys; assert pathlib.Path(sys.argv[1]).read_bytes()==b"iam-proof-a\n"' "$GET_WORK/instance-a.txt" || exit 1
sha256sum "$GET_WORK/instance-a.txt"
ERR="$GET_WORK/b-error.json"
if timeout -k 5s 45s oci os object get --auth instance_principal --region '<REGION>' --namespace-name '<NS>' --bucket-name iam-proof-b --name proof.txt --file "$GET_WORK/instance-b.txt" --max-retries 0 2>"$ERR"; then
  printf 'STOP: unexpected allowed B GET\n'; exit 1
else rc=$?; fi
python3 - "$rc" "$ERR" <<'PY'
import sys,json
assert int(sys.argv[1]) not in (0,124,137)
s=open(sys.argv[2]).read(); x=json.loads(s[s.index('{'):s.rindex('}')+1])
assert x.get('status') in (403,404) and x.get('code') in ('NotAuthorized','NotAuthorizedOrNotFound')
PY
[ "$?" = 0 ] || exit 1

```

5. B의 A get 성공/B get 실패를 기록한다. 네트워크는 둘 다 동일 endpoint이므로 관리자 B positive control을 남긴다.
6. A VM에서 같은 `--auth instance_principal` A get을 실행한다. A instance OCID는 matching rule에 없으므로 실패해야 한다. A에 다른 Dynamic Group policy가 없는지 관리자가 확인한다.
**단계6 A VM 전용 negative (B positive fence를 복사하지 않음):**
```bash
A_DENY_WORK=$(mktemp -d "$HOME/oci-principal36-a-deny.XXXXXXXX") || exit 1
if timeout -k 5s 45s oci os object get --auth instance_principal --region '<REGION>' --namespace-name '<NS>' --bucket-name iam-proof-a --name proof.txt --file "$A_DENY_WORK/proof.txt" --max-retries 0 2>"$A_DENY_WORK/error.txt"; then
  printf 'STOP: nonmember unexpectedly allowed\n'; exit 1
else RC=$?; fi
python3 - "$RC" "$A_DENY_WORK/error.txt" <<'PY'
import sys,json
assert int(sys.argv[1]) not in (0,124,137)
s=open(sys.argv[2]).read(); x=json.loads(s[s.index('{'):s.rindex('}')+1])
assert x.get('status') in (403,404) and x.get('code') in ('NotAuthorized','NotAuthorizedOrNotFound')
PY
if [ "$?" -ne 0 ]; then exit 1; fi
```
관리자 same A positive/hash와 A credentials/clock/network 정상 증거를 짝지으며 A_DENY_WORK의 proof.txt(생성된 경우)/error.txt를 정확히 삭제 후 rmdir한다.

7. 정확한 rule을 저장하고 같은 이름으로 새 Group을 다시 만들지 않는다. Matching rule 수정의 적용은 공식문서상 **약1시간** 여유가 필요하므로 즉시 실패/성공을 캐시 없이 확정하지 않는다. 변경 전에 관리자 복구 담당자와 원래 matching rule/policy snapshot을 확보하고 최대75분(60초 간격 최대75회)의 예산·VM/NAT·fixture lease를 별도 승인한다. 종료 UTC 또는75분 중 먼저 도달한 시각에 중단/원래 rule·grant 복구 후 BLOCKED_PROPAGATION으로 남긴다. 이 시간은 저술용 budget이지 서비스 SLA가 아니다. 승인 안 되면 rule 변경시험은 NOT_SELECTED이며 기존 exact membership positive만 수행한다.

**다운로드 negative 안전:** 매 상태에서 새 전용 output filename을 사용하거나 이전에 본 실습이 만든 파일만 제거한 뒤 호출한다. local file이 이미 있다는 CLI overwrite prompt/기존 hash를 서비스 deny/success로 해석하지 않는다. 명령 종료코드와 새 파일 생성 여부를 함께 기록하고 실패한 요청 뒤 오래된 파일의 checksum을 성공 증거로 쓰지 않는다.

## 결과검증

<a id="baseline-36-ready"></a>

완료 단계 **INSTANCE-PRINCIPAL-tested**: B instance id exact match, API key 없이 A object get 성공/B bucket get 실패, 비회원 A VM A get 실패. 호출 profile 대신 instance auth를 실제 명령에 명시했다는 증거를 남긴다. access failure request-id와 대상 Region/namespace를 남긴다.

**실행 기록(아직 미작성):** 실행자/Region/시각, exact resource OCID, 설정 전·후, 성공 출력, deny 오류·exit code/request-id, positive control, 정리 결과. 문서 검증은 실제 OCI 실행 성공을 뜻하지 않는다.

## 문제해결

인증 token 발급 실패는 VM metadata 접근·clock·CLI install·region network부터 확인한다. 404는 namespace/object 존재 positive control과 rule/Policy를 확인한다. A에서도 성공하면 A를 포함하는 다른 dynamic group/tenancy policy를 찾는다. rule 수정 직후 결과는 전파 지연을 기다린 후 재확인한다. private key를 VM에 넣는 해결책은 사용하지 않는다.

## 리소스정리

31 및 승인된 후속42가 필요한 fixture를 각 owner의 consumer registry/expiry까지 보존한다. 전체 종료 시 `p-instance-read-a` 제거 → owner10/22의 모든 소비자 반환 뒤 creator가 수행하는 VM 종료(또는36 소유 matching rule membership 정리/전파 확인) → Dynamic Group 삭제 → registry의 exact GET_WORK instance-a.txt/instance-b.txt/b-error.json 새 파일만 제거하고 empty directory rmdir한다. INSTALL_WORK의 reviewed install.sh도 사용 종료 후 exact 삭제/rmdir하며 설치된 CLI는 공유 소비자 확인 전 uninstall하지 않는다. topic36은 공유 VM/Boot Volume을 삭제하지 않고 instance/CLI/fixture 사용 반환을 owner10(A)/owner22(B)에 전달한다. 모든 예약 consumer 반환 후에만 해당 creator가 자기 VM 및 boot를 정리한다. 공유 Dynamic Group은 삭제하지 않는다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 목표는 학습 후 설명·구분·확인할 내용이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [25] [Overview of Identity and Access Management — Dynamic Groups](https://docs.oracle.com/en-us/iaas/Content/Identity/Concepts/overview.htm)
  - 확인 범위: DYNAMIC GROUP .

### 실습 작업을 지원하는 추가 공식 본문

- [추가 작업 근거] [Calling Services from an Instance](https://docs.oracle.com/en-us/iaas/Content/Identity/Tasks/callingservicesfrominstances.htm) — 이 문서의 실제 생성/호출 작업을 지원하는 공식 본문.

- [실습근거 1] [Managing Dynamic Groups](https://docs.oracle.com/en-us/iaas/Content/Identity/Tasks/managingdynamicgroups.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
- [실습근거 2] [Policy Syntax](https://docs.oracle.com/en-us/iaas/Content/Identity/Concepts/policysyntax.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
- [실습근거 3] [Details for Object Storage and Archive Storage](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
