> **문서 검토 완료 · OCI 실행 미검증**: 이 Resource Principals 실습은 한정된 문서 검토 범위에서 생성·검증·복구·정리 절차를 검토했습니다. 실제 OCI build/deploy/invoke 성공이나 전체 교육과정 승인을 뜻하지 않습니다.

# Resource Principals

> 문서 승인 범위: **한정된 실행 절차의 문서 검토 완료**. 실제 권한·network·비용·복구 승인과 선행 산출물 확인은 아래 단계의 실행 조건으로 유지합니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 게시 및 전체 교육과정 상태는 [통합 상태](../../../Docs/README.md)와 구분합니다.

> **저술 완료 · OCI 실행 미검증** — Oracle 공식 작업 문서를 근거로 작성한 한국어 hands-on입니다. Oracle 공인 교육/실행 인증이 아니며 기대 결과와 실제 결과를 구분합니다. 비용 승인과 실제 선행 산출물 확인 후에만 실행하세요.

## 학습목표

- 기존 private-app/SGW와 fixture를 재사용해 Python Function 이미지 build/push/deploy 및 exact Function Dynamic Group를 생성한다.
- Resource Principal invocation의 A status/hash·B 거부를 확인하고 Function→Application→이번 OCIR image를 정리한다.

- 선택 심화: 별도 40분 비용·복구 승인을 받은 경우만 A grant 제거/복원과 재검증을 수행한다. 기본 완료에 포함하지 않는다.

완료 기준은 아래 실제 생성·설정·요청 결과로 판정한다. 이 목표는 수행할 작업이며 OCI에서 관찰한 결과가 아니다.

## 대응 Roadmap 항목

- 항목명: **Resource Principals**
- Roadmap 경로: **Identity and Access Management (IAM) → Resource Principals**
- Node ID: `sequence-4-9`
- 상위 항목: **Identity and Access Management (IAM)** (`sequence-4-0`)
- 문서 범위: 공식 개념 목표와 실제 구성·허용/불허 검증·정리 절차. 아래 실습은 저술되었으나 OCI에서 실행 검증되지 않았습니다.
- [전체 항목 목록](../../../Docs/README.md)

## 사전준비

33 A/B 실제 REGION/NS/hash와31 SGW Enabled/service route/443 egress를 받는다.36의 matching-rule 개념은 권장 읽기이며 Compute DG를 Function DG로 재사용하지 않는다.21 private-app OCID/여유 IP, 기존 승인 Function/Image/Network Compartment OCID·deployer Group와 Cloud Shell/Fn/OCIR endpoint를 확보한다. 기존 VM/IP는 Function 배포 prerequisite가 아니다.

### 기준 계약과 실제 단계 선행조건

실제 tenancy/Region/Domain·주체·소유권은 [공통 baseline-contract.json](../../../baseline-contract.json)과 위 fixture 계약에 맞춘다. 빈 OCID/placeholder·미승인 권한은 실행 차단 입력이다. 이 API/Console 실습 때문에 VM을 생성하지 않는다.

- [33.ready Identity and Access Management (IAM) — FUNCTIONAL_COMPLETION](00_Identity_and_Access_Management_IAM.md#baseline-33-ready): 표시된 FUNCTIONAL_COMPLETION은 해당 linked ready 단계의 실제 검증/반환이 필요하다. PRE_CREATE_OR_INSTALL_PREPARATION으로 표시한 링크만 준비 출력까지 요구한다. 필요한 OCID/주소만 인계한다.
- [Dynamic Groups 개념](../../../Docs/04_Identity_and_Access_Management_IAM/03_Dynamic_Groups.md)은 권장 읽기다. 36의 VM/Instance Principal 실험 완료는 요구하지 않는다.
- [Service Gateway 준비 절차](../../Networking/Original-Provisional/03_Virtual_Cloud_Network_VCN/12_Service_Gateway.md): Function 배포 전 SGW Enabled, All Region Services label→SGW route, private-app 연결을 아래 Function 전용 준비 단계로 확인한다. VM 장애/CLI 연결 실험 완료는 요구하지 않는다.
- [Subnets 준비](../../Networking/Original-Provisional/03_Virtual_Cloud_Network_VCN/02_Subnets.md): actual private-app subnet OCID, VCN DNS와 여유 IP를 확인한다.

**Function 전용 network-ready (관리자 Console):** Networking → Virtual Cloud Networks → actual VCN → Service Gateways에서 기존 SGW 상태 Enabled와 Region service label을 읽는다. Subnets → private-app → Route Table에서 그 service label의 target이 같은 SGW OCID인지 확인한다. subnet OCID/route tuple/SGW OCID와 유지 기한을 기록한다. 둘 중 하나가 없으면 연결된 준비 절차의 생성/association 단계까지만 수행하고 VM 생성·차단 실험은 건너뛴다. Functions NSG egress443은 아래 A에서 생성한다. SGW creator에게 Function 소비자42를 등록하고 삭제 전 반환한다.

- Cloud Shell/Fn/Docker 사용 권한과 실제 OCIR login 가능한 사용자 auth token이 필요하다. token은 사용자 UI/보안 저장소에서만 다루고 본문/명령 history에 넣지 않는다. Function 완료까지31의 SGW/VCN와33 fixture를 보존한다.

## 권한

- `ListNetworkSecurityGroupSecurityRules`: `NETWORK_SECURITY_GROUP_LIST_SECURITY_RULES` — [공식 core API 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm). exact FUNCTIONS_NSG_ID를 조회하는 network 담당자/deployer의 Network compartment readback 권한을 생성권한과 별도로 확인한다.

### 실제 task/API별 공식 권한 본문 대조

아래 공식 operation 표에서 이 실습의 읽기·변경·정리 권한을 대조한다. family grant/일부 dependency 인용은 검증된 minimum policy가 아니며 실제 principal과 resource scope를 관리자가 승인해야 한다. 조직 Identity Domain Console user/group 작업은 해당 Domain 관리자 역할도 필요하며 legacy IAM user API 권한만으로 대체하지 않는다.

- `CreateApp`: FN_APP_CREATE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/functionspolicyreference.htm).
- `CreateDynamicGroup`: DYNAMIC_GROUP_CREATE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/policyreference/iampolicyreference.htm).
- `CreateFunction`: FN_FUNCTION_CREATE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/functionspolicyreference.htm).
- `CreatePolicy`: POLICY_CREATE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/policyreference/iampolicyreference.htm).
- `DeleteApp`: FN_APP_DELETE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/functionspolicyreference.htm).
- `DeleteDynamicGroup`: DYNAMIC_GROUP_DELETE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/policyreference/iampolicyreference.htm).
- `DeleteFunction`: FN_FUNCTION_DELETE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/functionspolicyreference.htm).
- `DeletePolicy`: POLICY_DELETE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/policyreference/iampolicyreference.htm).
- `GetObject`: OBJECT_READ For a customer-managed key encrypted bucket, the objectstorage-<location> subject must have KEY_DECRYPT. — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm).
- `InvokeFunction`: FN_INVOCATION — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/functionspolicyreference.htm).
- `ListFunctions`: FN_FUNCTION_LIST — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/functionspolicyreference.htm).

관리자만 User·Group·Dynamic Group·Policy·Compartment를 생성하거나 멤버십을 수정한다. Identity Domains의 사용자/그룹 관리에는 해당 Domain의 적절한 관리자 역할이 필요하고 OCI 리소스 Policy 관리 권한은 별개다. 기존 Administrators 또는 위임된 Domain 관리자에게 승인받는다. 학습자에게 `manage all-resources in tenancy`를 주는 방식으로 우회하지 않는다. 실습 작업자는 아래 절차에 명시한 Object Storage 읽기 권한만 받는다. [Overview of IAM](https://docs.oracle.com/en-us/iaas/Content/Identity/getstarted/identity-domains.htm)의 **The Administrators Group, Policy, and Administrator Roles**, [Managing Policies](https://docs.oracle.com/en-us/iaas/Content/Identity/Tasks/managingpolicies.htm)의 **Required IAM Policy**를 읽고 관리 주체와 검증 주체를 기록한다.

## 비용및주의사항

Function invocation/GB-second·OCIR storage·network transfer는 현행 Region 가격/계정 조건을 확인한다. Application 하나, Function 하나, 256MiB/30초, invocation 기본 상태당1~3회(초기 readiness는 별도 승인된 최대75회/75분 cap으로 대체), object는 수십 bytes로 제한한다. 무한 재시도·loop·tracing/APM·Vault·고성능 Compute를 만들지 않는다. Cloud Shell build가 실패하면 비용 승인 범위 내 해결하고 실제 build 성공을 관찰하기 전 배포 완료라 하지 않는다. 추가 subnet/SGW는31에서 재사용한다.

## 실습구성

### 이번 workshop 시나리오와 입력 계약

Python Function를 image build/deploy/invoke하고 Resource Principal A/B read를 검증한다.


- 입력: `VCN_ID`
  - 값/출처: owner19의 actual output; `<...>` 대체 필수
  - 시점: 실행 전 선행 완료값
  - 실행 위치: Console/사용자 전용 shell
  - 선택 이유: 정확한 대상/소유관계 고정

- 입력: `SUBNET_PRIVATE_ID`
  - 값/출처: owner21의 actual output; `<...>` 대체 필수
  - 시점: 실행 전 선행 완료값
  - 실행 위치: Console/사용자 전용 shell
  - 선택 이유: 정확한 대상/소유관계 고정

- 입력: `NS`
  - 값/출처: owner33의 actual output; `<...>` 대체 필수
  - 시점: 실행 전 선행 완료값
  - 실행 위치: Console/사용자 전용 shell
  - 선택 이유: 정확한 대상/소유관계 고정

- 입력: `FUNCTION_COMPARTMENT_OCID`
  - 값/출처: 관리자가 승인한 기존 Compartment의 actual OCID; 신규 Compartment 생성 없음
  - 시점: 배포/생성 전
  - 실행 위치: Console/deployer Cloud Shell
  - 선택 이유: 정확한 대상/소유관계 고정

- 입력: `IMAGE_COMPARTMENT_OCID`
  - 값/출처: 관리자가 승인한 기존 Compartment의 actual OCID; 신규 Compartment 생성 없음
  - 시점: 배포/생성 전
  - 실행 위치: Console/deployer Cloud Shell
  - 선택 이유: 정확한 대상/소유관계 고정

- 입력: `OCIR_HOST`
  - 값/출처: 승인 Region의 공식 OCIR endpoint/Console Registry 안내에서 배포 전 취득; `<...>` 대체 필수
  - 시점: 생성 후 readback
  - 실행 위치: Console/사용자 전용 shell
  - 선택 이유: 정확한 대상/소유관계 고정



- Console field: Application architecture / subnet / memory / timeout
  - 실제 입력/선택: iam-rp-app x86; private-app; 256MiB/30s; exact Function DG
  - 이유: 이번 기능/실패 원인을 한 변수로 분리


**생성·변경 범위:** iam-rp-app:FunctionsApplication,rp-read-proof:Function,iam-rp-image:ContainerImage,dg-rp-proof:DynamicGroup,p-rp-read-a:IamPolicy. **완료 비교:** real invocation allow200/hash and deny service error; grant removal deny→restore는 별도 선택 심화만 해당. 기대 패턴이며 OCI 실측값이 아니다. **정리:** 본 주제의 변경만 원복하고 전용 신규 resource만 dependency 역순으로 정리한다; 공유 자원은 creator와 consumer registry/expiry 기준으로 반환한다.

Application `iam-rp-app`와 Python Function `rp-read-proof` 하나를 **실제 image build/deploy/invoke**한다. Resource Principal signer로 A GetObject 성공, B GetObject deny를 JSON으로 반환한다. Dynamic Group은 `resource.id` exact Function OCID로 한정한다. 이 문서는 실제 OCI 실행을 수행하지 않은 authored tutorial이다.

## 실습절차

### 개별 완주 보정 — 생성·검증·소유권

SDK ObjectStorageClient의 Region은 Resource Principal signer가 제공하는 실제 function Region을 확인한다. Console Application config 값/Function OCID/DG subject를 readback한 뒤 invoke한다. SDK 또는 FDK build 실패는 성공 출력으로 대체하지 않는다. private-app function egress는 service destination443/DNS가 실제 연결된 NSG/SL에 허용되어야 한다; Compute B의 NSG만 허용돼 있다는 것으로 Function 허용을 추정하지 않는다.

**단계1 전 입력/승인 (deployer Cloud Shell):** 관리자가 기존 deployer Group의 실제 OCID/Domain, Network compartment의 실제 OCID와 image/function compartment scope를 제공/승인한다. 새 Group/compartment를 몰래 만들지 않는다. Console Region selector와 Registry의 Region endpoint/namespace를 대조해 REGION/OCIR_HOST를 먼저 기록한다(이미지 생성 출력이 아님). Application 생성 후 tailored Getting Started 값을 재대조한다.
```bash
export REGION='<approved-actual-region>'
export NETWORK_COMPARTMENT_OCID='<network-owner-approved-compartment-OCID>'
export DEPLOYER_GROUP_OCID='<administrator-approved-existing-group-OCID>'
export OCIR_HOST='<official-Registry-endpoint-for-approved-region>'
```
placeholder 또는 권한 승인 누락이면 network/application 생성 전 BLOCKED_INPUT으로 중단한다.

**생성 전 충돌·복구 기록:** Console에서 같은 compartment의 `iam-rp-app`, `dg-rp-proof`, `p-rp-read-a`, `nsg-functions-rp`와 Registry의 이번 `iam-rp/rp-read-proof` repository를 확인한다. 기존 대상이면 생성/수정/삭제하지 않고 중단한다. 재개는 이번 실행의 OCID/소유권 기록과 일치할 때만 허용한다. 새 auth token만 발급하여 token 식별자(비밀값 제외)를 기록하고 기존 token을 revoke하지 않는다. Docker login은 기존 login을 교체할 수 있으므로 기존 인증을 복원할 수 있는 전용 Cloud Shell 세션에서만 진행한다.

첫 mutation 전에 private 작업 기록을 열어 이미 확보한 입력/원래값을 Cloud Shell 밖에도 보존하고, 생성 출력은 각 생성 직후 다음 단계 전에 추가한다: Region, A/B 실제 bucket/namespace/object와 A/B 원본 hash 및 관리자 B 성공 시각, subnet/SGW/NSG와 rule ID, 생성한 각 resource OCID, 배포 grant snapshot/ADDED 구분, 이전 Fn context/registry, token 식별자, 이번 source 절대 경로와 각 RP42_WORK 절대 경로, 만료 시각. 새 terminal에서는 이 기록으로 입력과 context를 복원하고 Console에서 exact OCID를 재조회한다. 기록이 없으면 이름만으로 삭제하지 말고 관리자에게 반환한다.

### A. 배포 사용자 권한과 Application
1. 관리자 Console → Identity & Security → Policies에서 대상 policy의 OCID·Compartment·전체 기존 statements와 시각을 private 기록으로 보존한다. 아래 각 statement별로 PREEXISTING 또는 ADDED를 구분하고 추가 policy OCID/정확한 statement/반환 담당자를 기록한다. 이미 있는 문장은 추가하지 않는다. 가능하면 이번 추가분만 담은 전용 policy를 생성한다. 기존 policy 편집 시 동시 변경 여부를 확인하고 원래 전체 문서를 덮어쓰지 않는다. 관리자가 아래 official Functions policy statements를 실제 deployer Group/Compartment OCID로 적용한다. 정책 저장은 관리자, 배포/invoke는 deployer다. Logging/Tracing/Vault/ZPR는 이번 실습에서 활성화하지 않으므로 그 optional 권한을 임의 추가하지 않는다.

```text
Allow group id <DEPLOYER_GROUP_OCID> to read objectstorage-namespaces in tenancy
Allow group id <DEPLOYER_GROUP_OCID> to manage repos in compartment id <IMAGE_COMPARTMENT_OCID>
Allow group id <DEPLOYER_GROUP_OCID> to manage functions-family in compartment id <FUNCTION_COMPARTMENT_OCID>
Allow group id <DEPLOYER_GROUP_OCID> to use virtual-network-family in compartment id <NETWORK_COMPARTMENT_OCID>
Allow group id <DEPLOYER_GROUP_OCID> to use cloud-shell in tenancy
```

공식 **Functions policy statements** 및 **Common Policies — Cloud Shell** 근거를 확인한다. network 생성은 관리자 별도 관리권한이 필요하다. Bucket payload read 권한을 배포 사용자에게 더 주는 것이 Resource Principal 권한을 대신하지 않는다.
**2 직전 network administrator station:** Networking → shared-vcn → Network Security Groups → Create NSG: Name `nsg-functions-rp`, network Compartment, 규칙 없이 생성한다. 실제 NSG OCID를 FUNCTIONS_NSG_ID에 기록한다. Add Rules: Direction **Egress**, Stateless **off**, Destination Type **Service**, Destination **owner31 SGW와 같은 All <REGION> Services in Oracle Services Network label**, TCP source All / destination443. Ingress 없음. route는 owner31 rt-private의 동일 service label→SGW_ID를 재사용한다. NSG를 Compute B에만 붙이지 않는다. default VCN resolver는169.254.169.254이며 사용자 DNS appliance를 쓰면 별도 승인된 UDP/TCP53 destination/owner 규칙이 필요하다; 확인 불가면 BLOCKED_NETWORK로 중단한다.

Console Create Application에 NSG 선택이 보이면 **Use network security groups → nsg-functions-rp**를 명시한다. 선택 필드가 없으면 Console에서 생성하지 말고 아래 documented CLI alternative를 **한 번만** 수행한 뒤 Application Console에서 shape/NSG/subnet을 readback한다. [CLI Application create — network-security-group-ids](https://docs.oracle.com/iaas/tools/oci-cli/latest/oci_cli_docs/cmdref/fn/application/create.html).

```bash
# Deployer Cloud Shell, 실제 network creator 출력 사용
export FUNCTIONS_NSG_ID='<actual_nsg-functions-rp_OCID>'
export PRIVATE_SUBNET_ID='<owner21_private-app_OCID>'
export FUNCTION_COMPARTMENT_OCID='<approved_function_compartment_OCID>'
oci fn application create --compartment-id "$FUNCTION_COMPARTMENT_OCID" --display-name iam-rp-app --subnet-ids "[\"$PRIVATE_SUBNET_ID\"]" --network-security-group-ids "[\"$FUNCTIONS_NSG_ID\"]" --shape GENERIC_X86 --region "$REGION" > function42-created-app.json || exit 1
# CLI 제출을 했으면 다음 Console step2는 신규 Create가 아니라 readback만이다.
oci network nsg rules list --all --nsg-id "$FUNCTIONS_NSG_ID" --region "$REGION" > function42-network-rules.json || exit 1
```

Create 응답의 id/network-security-group-ids/subnet-ids/shape와 Console의 exact Application을 확인한다. Functions image pull/SDK TLS가 이 NSG/route에서 허용되는지 실제 invoke로 검증하기 전 OCI path 성공이라고 기록하지 않는다.

2. **Developer Services → Functions → Applications → Create Application**: `iam-rp-app`, Function Compartment, `shared-vcn`, `private-app`, Shape **Generic_X86**(Console가 제공하는 x86 선택), tracing off. Cloud Shell Actions → Architecture **X86_64**로 맞춘다. 지원 선택값이 다르면 official supported shape와 이미지 architecture를 맞추기 전 진행하지 않는다.
3. Application **Getting Started → Cloud Shell setup**의 현재 Region context와 OCIR endpoint/login username을 복사한다. Region key를 추측하지 않는다.

```bash
# Cloud Shell: 기존 context 파일을 변경하지 않고 새 dedicated context를 사용
fn list context || exit 1
export PRIOR_FN_CONTEXT='<list_context에서_asterisk로_확인한_기존_context>'
export PRIOR_FN_REGISTRY='<기존_context의_registry_값_비밀아님>'
export REGION_CONTEXT='<actual-region-context>'
export FUNCTION_COMPARTMENT_OCID='<actual-function-compartment-OCID>'
export IMAGE_COMPARTMENT_OCID='<actual-image-compartment-OCID>'
export OCIR_HOST='<actual-OCIR-host-from-Console>'
export NAMESPACE='<actual-namespace>'
# [공식 Cloud Shell quickstart] 전용 context의 provider는 oracle-cs; 기존 Region context 수정 금지
fn create context roadmap-rp42 --provider oracle-cs || exit 1
fn use context roadmap-rp42 || { fn use context "$PRIOR_FN_CONTEXT"; exit 1; }
export API_URL='<Console_Getting_Started의_actual_API_URL>'
fn update context api-url "$API_URL" || { fn use context "$PRIOR_FN_CONTEXT"; exit 1; }
fn update context oracle.compartment-id "$FUNCTION_COMPARTMENT_OCID" || { fn use context "$PRIOR_FN_CONTEXT"; exit 1; }
fn update context oracle.image-compartment-id "$IMAGE_COMPARTMENT_OCID" || { fn use context "$PRIOR_FN_CONTEXT"; exit 1; }
fn update context registry "$OCIR_HOST/$NAMESPACE/iam-rp" || { fn use context "$PRIOR_FN_CONTEXT"; exit 1; }
# 사용자 UI에서 생성한 auth token을 Docker password prompt에 입력
# 아래 username은 Console setup 값 그대로 사용 (Domain/federation에 따라 다름)
docker login -u '<CONSOLE_OCIR_USERNAME>' "$OCIR_HOST" || { fn use context "$PRIOR_FN_CONTEXT"; exit 1; }
if test -e rp-read-proof || test -L rp-read-proof; then fn use context "$PRIOR_FN_CONTEXT" || exit 1; printf '%s\n' 'STOP: dedicated source directory exists'; exit 1; fi
fn init --runtime python rp-read-proof || { fn use context "$PRIOR_FN_CONTEXT"; exit 1; }
cd rp-read-proof || { fn use context "$PRIOR_FN_CONTEXT"; exit 1; }
```

### B. 실제 함수 코드
4. 생성된 `func.py`를 아래 코드로 교체한다. SDK/FDK token을 읽거나 출력하지 않는다. `NAMESPACE`, `BUCKET_ALLOW`, `BUCKET_DENY`, `OBJECT_NAME`은 Application configuration에만 넣는다.

```python
import io
import json
import hashlib
import oci
from fdk import response

def handler(ctx, data: io.BytesIO = None):
    cfg = dict(ctx.Config())
    signer = oci.auth.signers.get_resource_principals_signer()
    client = oci.object_storage.ObjectStorageClient({}, signer=signer)
    results = {}
    for label, key in [('allow', 'BUCKET_ALLOW'), ('deny', 'BUCKET_DENY')]:
        try:
            r = client.get_object(cfg['NAMESPACE'], cfg[key], cfg['OBJECT_NAME'])
            payload = r.data.content
            results[label] = {'status': r.status,
                              'sha256': hashlib.sha256(payload).hexdigest()}
        except oci.exceptions.ServiceError as e:
            results[label] = {'status': e.status, 'code': e.code,
                              'request_id': e.request_id}
    return response.Response(ctx, response_data=json.dumps(results),
                             headers={'Content-Type': 'application/json'})
```

5. 생성된 `requirements.txt`의 기존 FDK dependency를 유지하고 `oci` dependency를 추가한다. 최초 작성 예시는 아래이며, 설치/build에서 해결된 버전을 기록하고 실제 환경 재현용 lock은 실행 후 별도 작성한다. 사전에 검증하지 않은 SDK 최신 버전번호를 invent하지 않는다.

```text
fdk
oci
```

생성된 `func.yaml`에서 name `rp-read-proof`, runtime python, memory **256** MiB, timeout **30**초를 확인한다. Fn-generated build/runtime 항목은 사용 중 template 값을 유지한다. 수정 전 sample handler 이름이 `handler`인지 확인한다.

```bash
fn -v deploy --app iam-rp-app || { fn use context "$PRIOR_FN_CONTEXT"; exit 1; }
fn config app iam-rp-app NAMESPACE '<ACTUAL_NAMESPACE>' || { fn use context "$PRIOR_FN_CONTEXT"; exit 1; }
fn config app iam-rp-app BUCKET_ALLOW '<owner33_actual_A_bucket_name>' || { fn use context "$PRIOR_FN_CONTEXT"; exit 1; }
fn config app iam-rp-app BUCKET_DENY '<owner33_actual_B_bucket_name>' || { fn use context "$PRIOR_FN_CONTEXT"; exit 1; }
fn config app iam-rp-app OBJECT_NAME proof.txt || { fn use context "$PRIOR_FN_CONTEXT"; exit 1; }
fn list functions iam-rp-app || { fn use context "$PRIOR_FN_CONTEXT"; exit 1; }
```

### C. exact Function membership와 payload 권한
6. Console **Application → Functions → rp-read-proof**에서 **Function OCID**를 복사한다. Application OCID가 아니다. 관리자 **Dynamic groups → Create** `dg-rp-proof`, rule:

```text
resource.id = '<ACTUAL_FUNCTION_OCID>'
```

7. 관리자 `iam-lab` policy `p-rp-read-a`:

```text
Allow dynamic-group id <DG_RP_PROOF_OCID> to read objects in compartment id <IAM_LAB_OCID> where target.bucket.name = '<actual_BUCKET_A_name>'
```

Function에 사용자 API key/env credential을 넣지 않는다. SDK signer는 official Functions 예제 그대로 Resource Principal을 사용한다.
8. **초기 readiness budget**은 optional grant-removal 40분과 별개다. 관리자에게 fixture/Function expiry 안에서 최대75분·최대75회·60초 간격을 명시 승인받는다(전파 SLA 아님). 승인 없으면 아래 단일 bounded invoke만 수행하고 실패는 BLOCKED_PROPAGATION으로 반환한다. 매 승인된 재시도는 새 output 파일을 쓰며 A200/exact owner33 hash와 B403/404 authorization code를 함께 검증한다. timeout/transport/build 오류는 전파로 분류하지 않고 즉시 중단한다. expiry 또는 budget 도달 시 원래 context로 복귀하고 owner와 Function/policy/fixture 반환·정리한다. 무한/자동 반복 루프는 제공하지 않는다. 단일 시도:


```bash
RP42_WORK=$(mktemp -d "$HOME/rp42-readiness.XXXXXXXX") || { fn use context "$PRIOR_FN_CONTEXT"; exit 1; }
# owner33 실제 A SHA256를 입력; OCI 미실행 상태에서는 입력을 비워 둔다.
export EXPECTED_A_SHA256='<owner33_actual_A_sha256>'
printf '{}' | timeout -k 5s 45s fn invoke iam-rp-app rp-read-proof > "$RP42_WORK/result.json" 2>"$RP42_WORK/error.txt" || { fn use context "$PRIOR_FN_CONTEXT"; exit 1; }
python3 - "$RP42_WORK/result.json" <<'PY'
import json,os,re,sys
expected=os.environ['EXPECTED_A_SHA256']; assert re.fullmatch('[0-9a-f]{64}',expected)
x=json.load(open(sys.argv[1]))
assert x['allow']['status']==200 and x['allow']['sha256']==expected
assert x['deny']['status'] in (403,404) and x['deny']['code'] in ('NotAuthorized','NotAuthorizedOrNotFound')
PY
[ "$?" = 0 ] || { fn use context "$PRIOR_FN_CONTEXT"; exit 1; }
```

반환 JSON의 allow status200/hash는33의 A hash와 같아야 하고 deny는 status403/404 및 code NotAuthorized/NotAuthorizedOrNotFound일 때만 authorization 후보이며 관리자 same B positive/hash와 frozen target을 짝지어야 한다. 401/429/5xx/timeout/기타 ServiceError는 BLOCKED_ERROR이지 deny가 아니다. deny200이면 다른 Dynamic Group/상위 policy가 허용했는지 조사한다. Function invoke의 HTTP 성공은 handler 내부 GetObject 두 건의 성공/실패와 별개다.

### D. 선택 심화 — 정책 제거 deny와 복구

기본 A/B 검증 후 이 단계를 선택하지 않으면 NOT_SELECTED로 기록하고 리소스정리로 진행한다.
**grant 제거 전 approval/recovery:** exact p-rp-read-a OCID/statements와 Function/DG/Application config를 저장한다. 담당 관리자가 원래 grant 복원 Console를 열고 fixture/Function/OCIR/SGW **최대40분 승인 lease와 UTC deadline**을 기록한다. token15분 캐시를 기다린 뒤60초 간격 **상태당 최대3회**(각 invoke timeout=45초, 새 process) 관찰한다. 복원 뒤에도15분 token cache를 고려하되 상태당3회 상한은 유지한다.40분 deadline에 복구 positive가 미확인이면 원래 grant를 유지하고 BLOCKED_PROPAGATION으로 반환한다. matching rule은 이 시험에서 변경하지 않는다. deadline에 결과와 무관하게 원래 grant 복원/readback 후 새 allow200/hash·deny서비스오류가 확인되기 전 후속 실습으로 넘기지 않는다. 새 예산승인 없으면 이 제거 branch는 NOT_SELECTED다.

9. 관리자 `p-rp-read-a` test grant만 제거한다. 공식문서의 Resource Principal token **15분 cache** 및 matching rule 변경 지연을 고려해 새 invocation으로 A도 실패하는지 확인한다. 로그/JSON에서 credentials는 출력하지 않는다. 제거 상태는 단계8의 새 RP42_WORK/timeout invoke를 그대로 사용하되 Python 판정만 아래로 바꾼다. 이 판정 성공 뒤에도 반드시 원래 grant를 복원/readback하고 단계8의 원래 A200/hash·B deny 판정을 새 directory에서 다시 수행한다. 제거 판정 실패나 terminal loss에도 관리자가 저장한 exact grant를 즉시 복원한다.

```python
# 제거 상태 result.json을 읽는 단계8 Python 블록의 대체 본문
import json, sys
x = json.load(open(sys.argv[1]))
for label in ("allow", "deny"):
    assert x[label]["status"] in (403, 404)
    assert x[label]["code"] in ("NotAuthorized", "NotAuthorizedOrNotFound")
```

401/429/5xx·timeout·JSON 오류는 제거 성공이 아니다. 동일 Region/namespace/object/config를 고정하고 관리자 A와 B GetObject/hash positive를 함께 재확인한다. 정책을 원래대로 복원한 후 A성공/B실패를 다시 확인한다. 관리자 같은 Region·namespace·B object get는 계속 성공해야 한다.

## 결과검증


### baseline-42-ready


완료 단계 **FUNCTION-RP-tested**: Fn build/push/deploy 성공 output, Function/Application/Image OCID/architecture, exact Function DG rule, 실제 invocation JSON과 A hash 일치/B deny, 제거 branch가 SELECTED면 grant 제거 A authorization deny·복구 A200/hash도 필수다. NOT_SELECTED면 제거/복구는 미실행으로 기록하고 기본 A200/hash·B authorization deny 계약만 반환한다; 제거시험 완료라고 쓰지 않는다. 작성 단계에서는 이 필드를 비워두며 코드 build/runtime·현재 SDK/FDK dependency compatibility는 OCI 실행 전 미검증이다. 존재하지 않는 function deploy 성공 출력은 제시하지 않는다.

**실행 기록(아직 미작성):** 실행자/Region/시각, exact resource OCID, 설정 전·후, 성공 출력, deny 오류·exit code/request-id, positive control, 정리 결과. 문서 검증은 실제 OCI 실행 성공을 뜻하지 않는다.

## 문제해결

|오류|확인|
|---|---|
|Docker401|Console OCIR username/auth token/Region endpoint, `manage repos` scope 확인|
|Image architecture mismatch|Cloud Shell x86와 Application x86/func build platform 일치|
|Invoke timeout|private Subnet SGW service label/egress443/DNS, Function limits/memory, image pull access 확인|
|Handler A/B 둘 다404|관리자 same get positive control, namespace/object/Region, exact fnfunc OCID/DG 정책 확인|
|deny가200|다른 DG matching rule·상위 정책·private Bucket/PAR 조건, handler target config 확인|
|정책 제거 직후 A성공|15분 RP cache/정책 전파를 기다리고 새 invocation; 즉시 결론 금지|

ZPR-enabled 환경의 additional security attribute 정책은 관리자 확인 게이트다. 문서에는 미확인 ZPR/FaaS 자동 정책을 invent하지 않는다.

## 리소스정리

관리자는 배포 권한 원복 기록을 먼저 열고 Function/OCIR/network 정리 API에 필요한 권한은 정리가 끝날 때까지 유지한다. 정리 후 Policies에서 ADDED statements만 제거하고 readback한다. 전용 policy는 이번 문장만 있는지 재조회한 뒤 삭제한다. PREEXISTING statements와 동시 추가된 다른 문장은 보존한다. 유지가 필요한 추가 grant는 policy OCID/정확한 문장/새 owner의 명시적 인수로 반환하며 자동 유지하지 않는다. 도중 실패/terminal loss에서도 이 기록을 사용해 남은 리소스 정리 후 확대 권한을 반환한다.

Function 삭제 → Application 삭제 → OCIR의 이번 function image/repository 삭제(다른 image는 보존) → `p-rp-read-a` 삭제 → exact function 없는 DG membership/전파 확인 후 `dg-rp-proof` 삭제 → 이번 Application 삭제/readback 뒤 nsg-functions-rp의 members 빈 상태 확인 → 자기 NSG/service443 rule 삭제 → `docker logout "$OCIR_HOST"`/이번 auth token revoke → `fn use context "$PRIOR_FN_CONTEXT"` 성공과 `fn list context`의 원래 active/registry 불변 확인 → `fn delete context roadmap-rp42` → Cloud Shell의 이번 source·download만 정리. 등록된 각 RP42_WORK의 result.json/error.txt만 증거 보존 종료 후 제거하고 빈 directory를 rmdir한다. 기존 context registry를 바꾸지 않았으므로 덮어쓰기 복원하지 않는다. 실패/중단에서도 prior context 선택을 먼저 복구한다. 이후 이번 소비자42의 반환을31 SGW와33 fixture creator에게 기록한다. 공유 SGW/NAT/VCN/Bucket은 여기서 직접 삭제하지 않는다. creator는 다른 등록 consumer의 반환도 확인한 뒤 자기 실습의 정리 순서를 수행한다. 이번 Cloud Shell 로컬 build image는 기록한 exact image ID만 확인해 제거하고 다른 image/cache 전체 prune은 하지 않는다. Functions 삭제만으로 OCIR 저장 이미지 비용이 없어지지 않는다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 목표는 학습 후 설명·구분·확인할 내용이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [20] [Accessing Other Oracle Cloud Infrastructure Resources from Running Functions — Resource Principals](https://docs.oracle.com/en-us/iaas/Content/Functions/Tasks/functionsaccessingociresources.htm)
  - 확인 범위: To enable a function to access.

### 실습 작업을 지원하는 추가 공식 본문

- [추가 작업 근거] [Creating Applications](https://docs.oracle.com/en-us/iaas/Content/Functions/Tasks/functionscreatingapps.htm) — 이 문서의 실제 생성/호출 작업을 지원하는 공식 본문.

- [실습근거 1] [Functions QuickStart on Cloud Shell Using Container Images](https://docs.oracle.com/en-us/iaas/Content/Functions/Tasks/functionsquickstartcloudshell.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
- [실습근거 2] [Creating Policies to Control Access to Network and Function-Related Resources](https://docs.oracle.com/en-us/iaas/Content/Functions/Tasks/functionscreatingpolicies.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
- [실습근거 3] [Accessing Other Oracle Cloud Infrastructure Resources from Running Functions](https://docs.oracle.com/en-us/iaas/Content/Functions/Tasks/functionsaccessingociresources.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
- [실습근거 4] [Common Policies](https://docs.oracle.com/en-us/iaas/Content/Identity/Concepts/commonpolicies.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
- [실습근거 5] [Details for Object Storage and Archive Storage](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
