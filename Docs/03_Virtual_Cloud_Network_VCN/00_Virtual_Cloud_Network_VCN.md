# Virtual Cloud Network (VCN)

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../README.md)를 확인하세요.

> 실행 절차 작성본 / OCI 미실행. Oracle 공식 본문 근거와 교육용 선택을 구분한다. secrets·실제 고객값·diagram 없음.

## 학습목표

- manual Create VCN으로 승인 CIDR 한 개·DNS label roadmap의 shared-vcn을 생성하고 default RT/SL/DHCP OCID를 조회한다.
- subnet이 없는 최초 상태를 확인해 owner21/23에게 실제 VCN_ID를 전달하고 모든 소비자와 gateway 참조가 해제된 뒤 마지막 network 삭제를 확인한다.

## 대응 Roadmap 항목

- 항목명: **Virtual Cloud Network (VCN)**
- Roadmap 경로: **Virtual Cloud Network (VCN)**
- Node ID: `sequence-3-0`
- 상위 항목: 없음 — 최상위 학습 항목
- 문서 범위: manual Create VCN으로 승인 CIDR 한 개·DNS label roadmap의 shared-vcn을 생성하고 default RT/SL/DHCP OCID를 조회한다. subnet이 없는 최초 상태를 확인해 owner21/23에게 실제 VCN_ID를 전달하고 모든 소비자와 gateway 참조가 해제된 뒤 마지막 network 삭제를 확인한다. 실제 OCI 실행 결과가 아니라 이 절차의 완료 조건이다.
- 번호는 navigation 식별자이며 실행 순서는 사전준비의 단계별 의존성에 따른다.

## 사전준비

### 기준 계약과 실제 단계 선행조건

[공통 baseline-contract.json](../../baseline-contract.json)의 Oracle Linux 9 / opc / CIDR / 비용 상한을 따른다. 문서 번호는 실행 순서가 아니다. **준비 station과 생성 이후 ready를 분리**한다. 아래 단계 외의 기능완료를 소급하여 요구하지 않는다.

- [00 Introduction — 단계 링크](../00_Introduction/00_Introduction.md#baseline-00-ready): **ACTIVE lab-root/승인 REGION**. 실제 출력/소비자/expiry를 전달하고 해당 owner가 이 lab 종료까지 보존한다.
- [05 Regions — 단계 링크](../01_Introduction_to_OCI/01_Regions.md#region-selection): **기존 subscribed Region 선택만**. 실제 출력/소비자/expiry를 전달하고 해당 owner가 이 lab 종료까지 보존한다.
- [20 CIDR Blocks — 단계 링크](01_CIDR_Blocks.md#cidr-planning): **단계 A 승인 주소계획만**. 실제 출력/소비자/expiry를 전달하고 해당 owner가 이 lab 종료까지 보존한다.

생성 전 조직 승인(identity/권한/예산/expiry/서비스 지원/limit/capacity)을 확인한다. 실제 OCID/IP는 아직 생성하지 않았으면 비워 두고 해당 Create/readback 뒤 채운다. 동일 creation-event-id/OCID는 한 번의 제출만 뜻한다.  실습에 없는 자원을 추가하지 않는다. 실제 checkpoint는 사용자 실행 전 비워 둔다.

### 입력값과 실행 위치

Console에서 선택한 값만 사용한다. `<...>`는 예시 OCID나 비밀값이 아니라 **사용자가 대체할 입력 자리**다. 대체하지 않은 채 명령을 실행하지 않는다. Console → 우측 상단 Cloud Shell/Code Editor → Cloud Shell을 열면 OCI CLI가 사전 인증된다. Region 변경 뒤에는 Shell을 다시 열거나 명령의 `--region`을 바꾼다. SSH는 Cloud Shell이 아니라 본인 Linux/WSL/macOS 터미널에서 수행한다. 공인 IP/32도 그 터미널의 출구 주소여야 한다.

```bash
# 아래는 입력/출력의 대입 형식이다. 표의 단계 전에 생성 후 ID/IP export 행을 실행하지 않는다.
 # 생성 후 검증용 ID/IP는 해당 creator의 실제 응답에서 넣는다.
export REGION='<actual_REGION>'
export LAB_COMPARTMENT_ID='<actual_LAB_COMPARTMENT_ID>'
export VCN_ID='<actual_VCN_ID>'
export VCN_CIDR='10.77.0.0/16'
oci --version
```

이 변수 중 해당 절차에서 사용하는 값만 채운다. CLI 조회의 `data`를 증거 파일에 저장하고 실제 시간·작업 OCID·선택값을 기록한다. API key/private key/passphrase/인증 토큰은 기록하지 않는다. Cloud Shell 설정파일이나 세션 토큰을 출력하지 않는다. `oci --version`을 실행하는 주제의 기대 패턴은 버전 문자열뿐이다. namespace는 Object Storage 주제에서 별도 조회하며 다른 주제에는 요구하지 않는다. [Cloud Shell 공식 본문](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/cloudshellgettingstarted.htm).

### Workshop 입력 계약 — 단계와 host

| 입력/출력 | 값 또는 실제 획득 경로 | 사용할 시점 | 실행 위치 | 선택 이유 |
|---|---|---|---|---|
| Console/설정 필드 | Name=shared-vcn; lab-root; 승인 VCN_CIDR 하나; DNS=roadmap; IPv6 없음; manual Create VCN | 생성 전 선택 → 생성 뒤 readback | 본문 Console/로컬 지정 | 이 topic의 최소 생성 범위 |
| `LAB_COMPARTMENT_ID` | owner00 Create 응답 OCID; topic00에서는 생성 뒤 | 선행 완료/생성 후 | Console/Cloud Shell | 정책/자원 scope |
| `PRIVATE_SUBNET_CIDR` | owner20 승인 계획; 기본 후보 10.77.20.0/24 | 생성 전 | 로컬/Console | private subnet 주소 |
| `PUBLIC_SUBNET_CIDR` | owner20 승인 계획; 기본 후보 10.77.10.0/24 | 생성 전 | 로컬/Console | public subnet 주소 |
| `REGION` | 승인된 기존 subscribed Region identifier | 생성 전 | Console/Cloud Shell | API endpoint/데이터 위치 |
| `VCN_CIDR` | owner20 승인 계획; 기본 후보 10.77.0.0/16 | 생성 전 | 로컬/Console/Cloud Shell | 중첩/중복 검증 |
| `VCN_ID` | owner19 Create 응답 id | 선행 완료/생성 후 | Console/Cloud Shell | shared-vcn 참조 |

생성 전에는 행의 승인값/선행 완료값만 지정하고, 생성 후 ID/IP는 해당 owner 응답으로 채운다. Cloud Shell과 로컬 SSH의 환경변수는 공유되지 않는다. 로컬에는 KEY와 접속 주소만 다시 지정하고 guest disk 명령은 로그인한 guest에서만 실행한다. `<...>`가 남으면 해당 명령은 실행하지 않는다.

## 권한

shared-vcn Create/Get/ListSubnets/최종 Delete 범위를 승인한다. 기본 RT/SL/DHCP 읽기는 생성 응답 및 해당 Console 상세 확인 권한으로 인계받는다. 후속 subnet/gateway/VM 생성은 해당 owner가 별도 승인하며 VCN creator가 자동 수행하지 않는다. VCN 삭제 전에 기존 모든 참조의 목록 조회를 허용받지 못하면 cleanup을 중단한다.

### 실제 operation과 공식 권한

- `CreateVcn`: VCN_CREATE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `DeleteVcn`: VCN_DELETE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `GetVcn`: VCN_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `ListSubnets`: SUBNET_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).

위 permission 이름은 IAM policy resource-type이 아니다. 조직 관리자가 기존 group/compartment scope와 conditional dependency를 실제 API 표에 대조한다. 편의 family grant를 최소권한 정책으로 부르지 않으며 실습자는 policy 작성 권한을 자동 인수하지 않는다.

## 비용및주의사항

- 이 단계는 shared-vcn 1개와 자동 default RT/SL/DHCP만 생성한다. VM·subnet·NAT·SGW를 자동 생성하는 connectivity wizard는 쓰지 않는다. 후속 Compute/storage/공인주소/전송 비용을 VCN 생성 자체와 혼동하지 않는다.
- 승인 CIDR/Region 밖으로 생성하지 않는다. 운영망과 겹치면 Create 전 중단한다. 기본 SL을 후속 subnet에 연결하지 않으며 소비자 VNIC/subnet/gateway 참조가 남아 있으면 VCN 삭제를 중단한다.

실제 OCI 작업은 아직 수행하지 않았다. 기대값은 실행 시 대조할 조건이며 무료 사용이나 과금 강제 차단을 보장하지 않는다. 승인 expiry가 끝나면 새 시험을 중단하고 위 소유권 순서로 원복·반환한다.

## 실습구성

### 이번 Workshop에서 만드는 것과 진행 순서

manual Create VCN으로 shared-vcn 하나와 자동 default RT/SL/DHCP만 만든다. subnet/gateway/VM은 후속 owner의 별도 단계다.

1. **진입/선택:** 00 lab-root/05 Region/20 CIDR 준비. 사전준비 링크의 지정 단계/실제 출력이 없으면 제출하지 않는다.
2. **실제 작업:** manual VCN 생성 → AVAILABLE/CIDR/DNS/default OCID readback. 아래 Console 필드/명령을 해당 host에서 순서대로 실행한다. 같은 자원은 이름이 아니라 OCID/event로 재사용한다.
3. **완료/복구/반환:** 다음 단계 전달. 결과검증 실패는 문제해결로 이동하고 동일 Create를 무작정 재제출하지 않는다.

**전달/정리 범위:** VCN_ID/default RT/SL/DHCP; owner19만 모든 소비자 종료 후 VCN 삭제. 아래 정리 단계는 이 소유권을 넘어서 실행하지 않는다.


- **이번 생성/재사용/출력:** 신규: shared-vcn 1개와 서비스 자동 생성 default RT/SL/DHCP. 출력 VCN_ID/CIDR/DNS/default OCIDs. 전체 승인 network consumers 종료까지 보존.
- **생성 목표:** 단한개의shared-vcn을05Region lab와공동생성하고93개curriculum의공통networkdependency로사용. subnet/gateway/rules는21-23및후속lab가단계적으로추가.

## 실습절차

### 명령 실행 gate — 실패하면 다음 생성으로 진행하지 않는다

1. 아래 Bash는 **전용 Bash 세션**에서만 사용한다. Console/Cloud Shell 조회, 본인 로컬 SSH, 로그인한 Oracle Linux guest를 섞지 않는다. `exit 1`은 이 전용 세션을 닫는 안전 중단이다. `<...>`·빈 입력·타 owner OCID가 남으면 실행하지 않는다.
2. 성공을 요구하는 파일 작성/조회/설치/attach 작업은 실패 시 중단한다. 실패한 API 뒤 기존 JSON이나 기존 다운로드 파일을 성공 근거로 읽지 않는다. 기대 deny는 `if command; then ...; else RC=$?; ...; fi`로 별도 기록하고, 실제 timeout/IAM 오류/로컬 파일 오류를 구별한다.
3. 첫 변경 전에 정확한 OCID·owner·현재 설정과 승인 expiry를 저장한다. 공유 자원 변경 중 다른 소비자가 시작하거나 원복값을 확인할 수 없으면 변경을 중단한다. 삭제 성공은 API 수락이 아니라 같은 identity/Region의 정확한 대상 재조회로 판정한다. `NotAuthorizedOrNotFound` 단독은 삭제 증거가 아니다.


### 개별 완주 보정 — 생성·검증·소유권

VCN_CIDR registry와 actualcidr-blocks를 비교한다. 기본SL은 public-admin/private-app/FSS subnet에 연결하지 않는다. owner19는 모든VNIC/subnet/gateway소비자종료뒤 VCN을 삭제한다.

<a id="vcn-create-station"></a>

### shared-vcn을 한 번만 만든다

1. 승인된 기존 subscribed Region을 Console에서 선택한다. 승인 lab-root Compartment를 고정한다. 아래 예시 `10.77.0.0/16`은 본인/VPN/회사/다른 VCN망과 겹치지 않는다고 관리자가 확인한 경우만 사용한다. 다른 승인값이면 public/private subnet도 계획에 맞게 바꾼다. CIDR 계획 계산은 CIDR Blocks의 생성 전 단계에서 수행한다.
2. Networking → Virtual cloud networks → **Create VCN**을 선택한다. VCN with Internet Connectivity wizard/Start VCN wizard는 쓰지 않는다. wizard가 추가 NAT/SGW/subnet을 만들면 최소 구성과 다른 비용/정리 계약이 된다. [Creating a VCN — manual follow-up resources](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/create_vcn.htm).
3. Name `shared-vcn`, Compartment `lab-root`, IPv4 CIDR Blocks **10.77.0.0/16 한 개**, Use DNS Hostnames in this VCN Yes, DNS label `roadmap`. IPv6 prefix 없음. 정부 Region의 VN encryption 기능이 본인 tenancy에 없으면 추가하지 않는다. free-form purpose/owner/expiry tags 입력 후 Create VCN을 제출한다.
4. AVAILABLE 상태/VCN OCID/CIDR/DNS label/default RT/SL/DHCP OCID를 기록한다. 기본 SL은 custom subnet에 연결하지 않는다. Region/CIDR 주제는 이 **같은 submission**에서 선택값을 기록하며 중복 VCN을 만들지 않는다.
5. 아직 subnet/gateway/VM이 없으므로 인터넷 경로를 성공했다고 하지 않는다. Public Subnets 단계 A에서 실제 bootstrap subnet/IGW/security를 만들고 Subnets에서 private-app을 만든다. VCN delete는 전체 소비자 종료 때만 수행한다.

## 결과검증

<a id="baseline-19-ready"></a>

```bash
oci network vcn get --vcn-id "$VCN_ID" --region "$REGION" > vcn-readback.json || exit 1
oci network subnet list --all --compartment-id "$LAB_COMPARTMENT_ID" --vcn-id "$VCN_ID" --region "$REGION" > subnet-list.json || exit 1
python3 - <<'PY'
import json,os
x=json.load(open('vcn-readback.json'))['data']
assert x['lifecycle-state']=='AVAILABLE'
assert x['display-name']=='shared-vcn'
assert x['dns-label']=='roadmap'
assert x['cidr-blocks']==[os.environ['VCN_CIDR']]
assert json.load(open('subnet-list.json'))['data']==[], 'STOP: initial creation station requires no subnets; later consumers use their own readbacks'
print('vcn assertions passed')
PY
if [ "$?" -ne 0 ]; then exit 1; fi
```
CIDR를 다른 승인값으로 썼으면 assert도 같은 값으로 변경한다. 기대 패턴 AVAILABLE, 승인 CIDR와 DNS label, default RT/SL/DHCP OCID 문자열. 생성 직후 subnet 목록은 empty이며 이후 lab에서는 public/private 2개를 기대한다.

**증거 기록:** 실제 명령·UTC 시간·exit code·가려진 OCID·옵션 readback·검증 조건을 남긴다. 현재 이 문서의 실행상태는 `NOT_EXECUTED`다. output/creation-event-id/result는 실행 전 비워 둔다. 오류/NOT_SELECTED를 PASS로 바꾸지 않는다.

### 이 시나리오의 functional acceptance

| Positive | Negative / 미완료 판정 | Recovery / 반환 |
|---|---|---|
| VCN AVAILABLE/CIDR/default ids; no subnets yet | wizard의 예상치 못한 subnet/gateway 생성은 scope 불일치 | 미제출이면 manual form; 이미 생성이면 정확한 owner 자원만 비용/cleanup 재승인 |

모두 실행 시 확인할 기대 패턴이다. 생성 전 guard negative는 OCI 장애를 주입한 것과 다르며 다른 자원을 추가하지 않는다. 오류 메시지와 실제 exit code가 없는 항목은 NOT_EXECUTED 또는 BLOCKED로 남긴다.

## 문제해결

### 공통 오류의 진단 순서

1. `NotAuthorizedOrNotFound` 또는 목록에 리소스가 없음: 먼저 Region/Compartment/OCID와 identity domain/group을 확인한다. 존재하지 않는 OCID와 권한 부족 모두 같은 오류로 보일 수 있다. 관리자에게 request-id·작업 시각·서비스·가려진 대상 OCID를 제공한다. 비밀키/토큰은 보내지 않는다.
2. `LimitExceeded`/quota/capacity 부족: Billing & Cost Management → Limits, Quotas and Usage와 생성 화면의 지원 shape/AD를 확인한다. 운영 quota 증가나 다른 Region 구독을 임의 요청하지 않는다. 승인 범위 안의 더 작은 compatible shape 또는 다른 AD를 선택하거나 `BLOCKED_CAPACITY`를 기록한다.
3. 장시간 provisioning/예상 밖 state: Work Requests/오류 상세를 읽고 같은 요청을 반복 제출하지 않는다. 만들어진 부분 리소스(boot/IP/attachment 포함)를 먼저 조사한다.
4. 메뉴 이름이 다름: 공식 task body의 동등한 탭(예: Storage 또는 Attached Block Volumes)을 찾는다. 필드가 문서와 다르면 값·스크린샷에서 secrets를 가린 뒤 관리자 또는 Oracle Support에 확인하고 제출을 중단한다. 이 문서는 현재 tenancy의 live Console 화면을 검사한 결과가 아니다.

### 이 주제의 실패 지점

VCN create 후 internet이 안 되는 것은 subnet/IGW/route/security가 아직 없기 때문일 수 있다. 기본 SL의 wide SSH를 custom subnet에 선택해 해결하지 않는다.

## 리소스정리

1. owner10/22/07 및 각 service creator에게 자기 VM/VNIC/LB/mount target 반환을 요청하고 exact 참조 제거 readback을 받는다. owner19가 다른 creator 자원을 직접 삭제하지 않는다. subnet 삭제 전 모든 참조 해제를 확인한다.
2. owner21/23 및 별도 subnet creator가 Networking → VCN → Subnets → 자기 exact OCID → Delete 후 get/list 제거 증거를 전달한다. owner19는 이를 읽는다. 후속 소비자가 승인된 종료시각 안에 남아 있으면 생성 owner가 보존하고 소비자/기한을 기록한다.
3. owner30/31이 소비자 반환 후 자기 NAT/SGW target route→gateway를 제거하고, owner23이 자기 IGW route→igw-lab, public NSG/SL/RT를 정리한다. owner21은 private SL/RT를 정리한다. owner19는 해당 exact OCID readback만 받는다. 공유 VCN 기본 RT/SL/DHCP는 별도 삭제 대상으로 임의 지우지 않는다.
4. VCN owner만 shared-vcn을 마지막으로 삭제한다. IAM Compartment는 모든 하위 리소스와 실습 정책을 제거한 후 마지막이다. Console Delete VCN의 일괄 삭제 기능을 쓰더라도 삭제 대상 목록을 먼저 읽고 다른 사용자 리소스가 있으면 중단한다. 삭제 API 수락만으로 성공이라 하지 않고 정확한 OCID 목록/readback을 남긴다.


## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 아래 본문은 각 생성 필드·설정·검증·정리의 기술 근거이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [30] [Networking Overview — Virtual Cloud Network (VCN)](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/overview.htm)
  - 확인 범위: virtual cloud network.

### 추가 실행 절차 근거 (본문 확인)

- [vcn](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/create_vcn.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [policy](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [cloudshell](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/cloudshellgettingstarted.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
