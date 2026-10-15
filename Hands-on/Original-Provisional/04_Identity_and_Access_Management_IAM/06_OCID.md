> **문서 절차 검토 완료 · OCI 실행 미검증**: 기존 Compartment의 OCID 대조와 description 변경·복원 시나리오를 대상으로 문서 검토를 완료했습니다. 실제 OCI 실행 성공이나 전체 커리큘럼 승인을 뜻하지 않습니다.

# OCID

> 검토 범위: 이 문서의 절차·선행조건·복구·정리 경로에 대한 **문서 수준 검토 완료**입니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 실행 시 아래 권한·승인·실제 선행 산출물·복구 확인 조건을 충족해야 합니다. 게시·렌더링 및 전체 커리큘럼 상태와는 구분하며 [통합 상태](../../../Docs/README.md)를 확인하세요.

> **저술 완료 · OCI 실행 미검증** — Oracle 공식 작업 문서를 근거로 작성한 한국어 hands-on입니다. Oracle 공인 교육/실행 인증이 아니며 기대 결과와 실제 결과를 구분합니다. 비용 승인과 실제 선행 산출물 확인 후에만 실행하세요.

## 학습목표

- 기존 iam-lab의 Console OCID와 관리자 GetCompartment 응답 id/parent를 대조한다.
- 승인된 description 변경 전후 id 불변을 읽고 원래 description을 복원한다; 새 OCID나 VM은 만들지 않는다.

완료 기준은 아래 실제 생성·설정·요청 결과로 판정한다. 이 목표는 수행할 작업이며 OCI에서 관찰한 결과가 아니다.

## 대응 Roadmap 항목

- 항목명: **OCID**
- Roadmap 경로: **Identity and Access Management (IAM) → OCID**
- Node ID: `sequence-4-6`
- 상위 항목: **Identity and Access Management (IAM)** (`sequence-4-0`)
- 문서 범위: 공식 개념 목표와 실제 구성·허용/불허 검증·정리 절차. 아래 실습은 저술되었으나 OCI에서 실행 검증되지 않았습니다.
- [전체 항목 목록](../../../Docs/README.md)

## 사전준비

필수 입력은33의 IAM_LAB_OCID·Console User/Group OCID·LAB_ADMIN/REGION·원래 description 및 owner33 편집 승인이다. outsider 권한 비교는34 완료 때만, VCN/Instance 관찰은24 완료 때만 선택한다. 새 VM/VCN/OCID 생성은 필요 없다. description 복원 readback을33에 반환한다.

### 기준 계약과 실제 단계 선행조건

실제 tenancy/Region/Domain·주체·소유권은 [공통 baseline-contract.json](../../../baseline-contract.json)과 위 fixture 계약에 맞춘다. 빈 OCID/placeholder·미승인 권한은 실행 차단 입력이다. 이 API/Console 실습 때문에 VM을 생성하지 않는다.

- [33.ready Identity and Access Management (IAM) — FUNCTIONAL_COMPLETION](00_Identity_and_Access_Management_IAM.md#결과검증): 표시된 FUNCTIONAL_COMPLETION은 해당 linked ready 단계의 실제 검증/반환이 필요하다. PRE_CREATE_OR_INSTALL_PREPARATION으로 표시한 링크만 준비 출력까지 요구한다. 필요한 OCID/주소만 인계한다.

- [Identity and Access Management (IAM)](../../../Docs/04_Identity_and_Access_Management_IAM/00_Identity_and_Access_Management_IAM.md) **IAM-fixture-ready** 완료: Compartment/User/Group의 실제 OCID, Bucket A get 성공.
- API용 검증은 관리자 profile과 Reader profile을 분리한다. OCID는 일반적으로 password는 아니지만 공개 저장소에 실제 tenancy/customer 식별자를 붙이지 않는다.

## 권한

### 실제 task/API별 공식 권한 본문 대조

아래 공식 operation 표에서 이 실습의 읽기·변경·정리 권한을 대조한다. family grant/일부 dependency 인용은 검증된 minimum policy가 아니며 실제 principal과 resource scope를 관리자가 승인해야 한다. 조직 Identity Domain Console user/group 작업은 해당 Domain 관리자 역할도 필요하며 legacy IAM user API 권한만으로 대체하지 않는다.

- `GetCompartment`: COMPARTMENT_INSPECT — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/policyreference/iampolicyreference.htm).
- `GetObject`: OBJECT_READ For a customer-managed key encrypted bucket, the objectstorage-<location> subject must have KEY_DECRYPT. — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm).
- `UpdateCompartment`: COMPARTMENT_UPDATE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/policyreference/iampolicyreference.htm).

관리자만 User·Group·Dynamic Group·Policy·Compartment를 생성하거나 멤버십을 수정한다. Identity Domains의 사용자/그룹 관리에는 해당 Domain의 적절한 관리자 역할이 필요하고 OCI 리소스 Policy 관리 권한은 별개다. 기존 Administrators 또는 위임된 Domain 관리자에게 승인받는다. 학습자에게 `manage all-resources in tenancy`를 주는 방식으로 우회하지 않는다. 필수 경로의 실행자는 승인된 관리자 profile로 위 Compartment 읽기·description 변경/복원 권한을 사용한다. Object Storage 권한 비교는 기존 USER-tested 선택 분기에서만 수행한다. [Overview of IAM](https://docs.oracle.com/en-us/iaas/Content/Identity/getstarted/identity-domains.htm)의 **The Administrators Group, Policy, and Administrator Roles**, [Managing Policies](https://docs.oracle.com/en-us/iaas/Content/Identity/Tasks/managingpolicies.htm)의 **Required IAM Policy**를 읽고 관리 주체와 검증 주체를 기록한다.

## 비용및주의사항

새 유료 리소스는 없고 owner33 iam-lab의 GetCompartment와 description 변경/복원만 수행한다. 기존 fixture 보존 비용은 owner33 lease를 따른다. VM/VCN 식별자 관찰과 outsider 요청은 이미 해당 prerequisite가 완료된 선택 branch에서만 수행하며 이를 위해 새 자원을 만들지 않는다. description 변경 전 before.json 원래값을 저장하고 별도 관리자 탭을 유지한다. 공유환경에서만 독립 복구담당자를 지정한다. OCID를 공개 로그에 남기거나 문자열을 조합해 새 식별자를 만들지 않는다.

## 실습구성

### 이번 workshop 시나리오와 입력 계약

실제 Compartment OCID를 Console/API로 비교하고 description만 원복 가능한 변경을 한다.

|입력|값/출처|시점|실행 위치|선택 이유|
|---|---|---|---|---|
|`IAM_LAB_OCID`|owner33의 actual output; `<...>` 대체 필수|실행 전 선행 완료값|Console/사용자 전용 shell|정확한 대상/소유관계 고정|
|`OUTSIDER_USER_OCID`|선택 USER-tested branch에서만 owner34 actual output; 미선택 NOT_SELECTED|선택 branch 실행 전만|Console/사용자 전용 shell|정확한 대상/소유관계 고정|
|`NS`|owner33의 actual output; `<...>` 대체 필수|실행 전 선행 완료값|Console/사용자 전용 shell|정확한 대상/소유관계 고정|

|Console field|실제 입력/선택|이유|
|---|---|---|
|OCID / description / original value|exact copied identifier; no string repair; original description saved|이번 기능/실패 원인을 한 변수로 분리|

**생성·변경 범위:** description-check:ConfigurationChange. **완료 비교:** same id/name/parent; unchanged id after description; known-target auth control. 기대 패턴이며 OCI 실측값이 아니다. **정리:** 본 주제의 변경만 원복하고 전용 신규 resource만 dependency 역순으로 정리한다; 공유 자원은 creator와 consumer registry/expiry 기준으로 반환한다.

새 OCID/리소스를 만들지 않는다. 필수 작업은33의 실제 Compartment/User/Group Console 식별자와 Compartment API의 비교 및 description 원복이다. VCN/Instance 비교는24.ready 완료·승인 때만 선택한다. display name/description 변경과 전체 OCID의 불변을 분리해 기록한다.

## 실습절차

### 개별 완주 보정 — 생성·검증·소유권

outsider variant는 [Users](01_Users.md#결과검증) 완료 때만 선택한다. VCN/VM 식별자 비교는 [Route Tables](../../Networking/Original-Provisional/03_Virtual_Cloud_Network_VCN/05_Route_Tables.md#결과검증) 완료 때만 선택한다. 필수 Compartment 비교를 위해 이 variant 자원을 새로 생성하지 않는다.

### A. Console 식별자 수집
1. **Compartments → iam-lab → OCID Show/Copy**에서 OCID를 복사한다. Group `lab-readers`, User `lab-reader`에서도 같은 방식으로 기록한다. **24.ready를 이미 완료하고 해당 선택 branch를 승인한 경우에만** 기존 VCN/VM의 OCID/display name을 추가 기록한다. 미선택이면 NOT_SELECTED이며 새 VCN/VM은 만들지 않는다.
2. [Resource Identifiers](https://docs.oracle.com/en-us/iaas/Content/General/Concepts/identifiers.htm)의 일반 형식 `ocid1.<RESOURCE TYPE>.<REALM>.[REGION].[FUTURE USE].<UNIQUE ID>`와 비교한다. 모든 서비스가 동일한 region segment를 갖는다고 가정하지 않고 전체 값을 문자열로 취급한다. 부분 문자열을 조합해 새 OCID를 만들지 않는다.

### B. 조회 대상과 주체 검증
3. 실제 Compartment OCID를 관리자 CLI로 조회한다. Group/User의 identity domain 계정은 Identity Domain API와 legacy IAM API를 혼용하지 말고 Console에서 actual OCID를 확인한다. 이 실습의 API 대조는 OCI Compartment에 고정한다.

```bash
oci iam compartment get --profile LAB_ADMIN --region '<actual-approved-region>' --compartment-id '<ACTUAL_IAM_LAB_OCID>'
# USER-tested 분기를 선택/승인한 경우에만 아래 요청 수행; standalone은 NOT_SELECTED
# oci iam compartment get --profile LAB_OUTSIDER --compartment-id '<ACTUAL_IAM_LAB_OCID>'
```

4. 관리자 응답 `data.id`/name/parent를 Console 값과 대조한다. outsider 조회 실패가 발생하면 관리자의 동일 target 성공을 남기고 target 존재/권한을 분리한다. outsider가 상위 정책으로 compartment read를 받는 환경이면 실패를 강제하지 않고 그런 정책을 기록한다.
5.33 policy의 `group id`/`compartment id`가 이 기록과 정확히 일치하는지 대조한다. USER-tested branch를 선택한 경우에만 outsider의 Bucket A get deny로 OCID knowledge와 권한을 분리한다. 미선택은 NOT_SELECTED이며 outsider를 새로 만들지 않는다.
**단계6 전 저장/복구 station (관리자 PC의 Linux/WSL Bash, OCI CLI·Python3 준비):** owner33에게 이 Compartment의 description 편집을 승인받고 원래값을 보존한다. 개인 실습은 별도 관리자 Console 탭을 유지한다. 공유환경에서만 독립 Console 복구담당자를 지정한다. 다음 exact snapshot 성공 전에는 Edit를 누르지 않는다. 같은 터미널에서 진행하며 새 터미널에서는 이 작업 디렉터리로 돌아와 동일 입력을 다시 설정한다.
```bash
# A에서 복사한 전체 OCID와 승인 Region을 입력한다. profile LAB_ADMIN은33의 기존 설정이다.
export IAM_LAB_OCID='<ACTUAL_IAM_LAB_OCID>'
export REGION='<actual-approved-region>'
umask 077
mkdir -m 700 ocid39-description || exit 1
oci iam compartment get --profile LAB_ADMIN --region "$REGION" --compartment-id "$IAM_LAB_OCID" > ocid39-description/before.json || exit 1
python3 - <<'PY'
import json, os
x=json.load(open('ocid39-description/before.json'))['data']
assert all(k in x for k in ('id','name','compartment-id','description'))
assert x['id']==os.environ['IAM_LAB_OCID']
assert isinstance(x['description'],str)
assert x['description']!='OCI IAM learning only; OCID check', '변경값이 이미 동일: Edit하지 말고 owner33과 다른 시험값을 정한 뒤 절차를 조정한다.'
print('Console에서 name/parent도 대조:', x['name'], x['compartment-id'])
print('복원할 description:', repr(x['description']))
PY
if [ "$?" -ne 0 ]; then exit 1; fi
```
Console의 name/parent도 snapshot과 일치해야 진행한다. 변경 전에 before.json 원래 description을 별도 관리자 탭에서 볼 수 있도록 보관한다. 빈 문자열이면 복원 시 Description을 비운다. 이 작업 폴더는 복원 확인 전 삭제하지 않는다.
6. Console → Compartments → exact iam-lab → Edit → Description만 `OCI IAM learning only; OCID check` → Save한다. 이름/parent는 변경하지 않는다. 이제 changed GET을 수행한다.
changed GET 성공 뒤 **한 번만** before.json의 원래 description으로 Console Edit/Save 복원하고 restored GET을 수행한다. 실패/세션 손실 시 유지한 관리자 탭에서 before.json 원래값으로 즉시 복원한다.
```bash
(set -C; oci iam compartment get --profile LAB_ADMIN --region "$REGION" --compartment-id "$IAM_LAB_OCID" > ocid39-description/changed.json) || {
  printf '%s\n' 'GET 실패: 여기서 중단하고 관리자 탭에서 before.json 원래 description 복원/확인. 파일 보존.'
  exit 1
}
```
관리자 탭에서 before.json의 description으로 복원/Save한 뒤에만 다음 블록을 실행한다.
```bash
(set -C; oci iam compartment get --profile LAB_ADMIN --region "$REGION" --compartment-id "$IAM_LAB_OCID" > ocid39-description/restored.json) || {
  printf '%s\n' 'readback 실패: Console에서 원래값 확인, 파일 보존. OCID-checked 반환 금지.'
  exit 1
}
```

Save 뒤 새 `changed.json`, 복원 뒤 새 `restored.json`에 같은 GET을 저장한다(기존 파일/symlink가 있으면 중단; noclobber `set -C` 사용). 다음 비교가 끝나야 반환한다.
```bash
python3 - <<'PY'
import json
b,c,r=[json.load(open('ocid39-description/'+p+'.json'))['data'] for p in ('before','changed','restored')]
assert all(b[k]==c[k]==r[k] for k in ('id','name','compartment-id'))
assert c['description']=='OCI IAM learning only; OCID check'
assert r['description']==b['description']
PY
if [ "$?" -ne 0 ]; then exit 1; fi
```
오류/세션 손실 시 관리자 탭에서 before.json의 description으로 즉시 복원/readback한다. 복원 불명은 BLOCKED_RECOVERY이며 parent/fixture 삭제 금지. GET 실패로 빈 JSON이 남아도 전체 폴더를 재생성하거나 before.json을 덮어쓰지 않는다. 복원 뒤 재조회가 필요하면 실패한 changed/restored 파일만 별도 이름으로 보존하고 해당 GET을 다시 수행한다. changed 관측을 잃었으면 복원 확인만으로 완료를 주장하지 말고 owner33과 새 시도 여부를 정한다. 반환 후 이 세 파일과 이번 시도의 실패 출력만 삭제하고 빈 ocid39-description을 rmdir한다.

## 결과검증

### baseline-39-ready

완료 단계 **OCID-checked**: Console/API 동일 Compartment id·parent, 각 resource type 형식의 실제 관찰, policy subject/target 대응, changed description=`OCI IAM learning only; OCID check`, restored description==before description 및 id/name/parent 불변 assertion 성공, 선택 USER-tested 분기에서만 target을 아는 outsider의 GetObject 실패(미선택이면 NOT_SELECTED). 무작위 malformed OCID404를 authorization deny로 사용하지 않는다.

**실행 기록(아직 미작성):** 실행자/Region/시각, exact resource OCID, 설정 전·후, 성공 출력, deny 오류·exit code/request-id, positive control, 정리 결과. 문서 검증은 실제 OCI 실행 성공을 뜻하지 않는다.

## 문제해결

404는 wrong OCID/permission을 모두 나타낼 수 있으므로 관리자 exact id 조회로 구별한다. Domain 사용자 API와 `oci iam user get`은 환경별 차이가 있어 legacy endpoint의 실패를 사용자 부재로 판정하지 않는다. 복사한 값의 공백/newline/다른 Region endpoint를 확인한다.

## 리소스정리

새 리소스는 없으므로 로컬 OCID 작업표와 CLI 출력에서 민감 식별정보를 제거한다. restored.json과 before.json의 description 일치 및 id/name/parent 불변을 확인한다. 앞서 복원을 마쳤으면 다시 Edit하지 않는다.33 fixture는40/41/42 종료까지 유지한다. OCID 자체를 삭제하는 독립 작업은 없으며 실제 resource 삭제는 소유 실습의 정리 절차로 수행한다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 목표는 학습 후 설명·구분·확인할 내용이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [22] [Resource Identifiers — OCID](https://docs.oracle.com/en-us/iaas/Content/General/Concepts/identifiers.htm)
  - 확인 범위: Most types of Oracle.

### 실습 작업을 지원하는 추가 공식 본문

- [실습근거 1] [Resource Identifiers](https://docs.oracle.com/en-us/iaas/Content/General/Concepts/identifiers.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
- [실습근거 2] [Policy Syntax](https://docs.oracle.com/en-us/iaas/Content/Identity/Concepts/policysyntax.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
- [실습근거 3] [Managing Compartments](https://docs.oracle.com/en-us/iaas/Content/Identity/Tasks/managingcompartments.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
