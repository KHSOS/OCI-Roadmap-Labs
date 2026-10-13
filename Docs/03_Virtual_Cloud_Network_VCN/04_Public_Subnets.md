# Public Subnets

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../README.md)를 확인하세요.

> 실행 절차 작성본 / OCI 미실행. Oracle 공식 본문 근거와 교육용 선택을 구분한다. secrets·실제 고객값·diagram 없음.

## 학습목표

- VM 없는 단계 A에서 igw-lab/rt-public/빈 sl-public/restricted nsg-admin/Regional public-admin을 생성하고 association을 조회한다.
- owner10의 같은 vm-a launch 뒤 단계 B로 돌아와 새 SSH와 유한 HTTPS를 검증하고 service 소비자까지 반환한 후 public network만 정리한다.

## 대응 Roadmap 항목

- 항목명: **Public Subnets**
- Roadmap 경로: **Virtual Cloud Network (VCN) → Public Subnets**
- Node ID: `sequence-3-4`
- 상위 항목: **Virtual Cloud Network (VCN)** (`sequence-3-0`)
- 문서 범위: VM 없는 단계 A에서 igw-lab/rt-public/빈 sl-public/restricted nsg-admin/Regional public-admin을 생성하고 association을 조회한다. owner10의 같은 vm-a launch 뒤 단계 B로 돌아와 새 SSH와 유한 HTTPS를 검증하고 service 소비자까지 반환한 후 public network만 정리한다. 실제 OCI 실행 결과가 아니라 이 절차의 완료 조건이다.
- 번호는 navigation 식별자이며 실행 순서는 사전준비의 단계별 의존성에 따른다.

## 사전준비

### 기준 계약과 실제 단계 선행조건

[공통 baseline-contract.json](../../baseline-contract.json)의 Oracle Linux 9 / opc / CIDR / 비용 상한을 따른다. 문서 번호는 실행 순서가 아니다. **준비 station과 생성 이후 ready를 분리**한다. 아래 단계 외의 기능완료를 소급하여 요구하지 않는다.

- [19 Virtual Cloud Network (VCN) — 단계 링크](00_Virtual_Cloud_Network_VCN.md#baseline-19-ready): **AVAILABLE VCN/default DHCP**. 실제 출력/소비자/expiry를 전달하고 해당 owner가 이 lab 종료까지 보존한다.
- [20 CIDR Blocks — 단계 링크](01_CIDR_Blocks.md#cidr-planning): **단계 A 승인 public CIDR**. 실제 출력/소비자/expiry를 전달하고 해당 owner가 이 lab 종료까지 보존한다.
- [10 Compute — 단계 링크](../02_Compute/00_Compute.md#shared-launch): **단계 B에서만 launch/SSH 필요; 단계 A에서는 VM 불필요**. 실제 출력/소비자/expiry를 전달하고 해당 owner가 이 lab 종료까지 보존한다.

생성 전 조직 승인(identity/권한/예산/expiry/서비스 지원/limit/capacity)을 확인한다. 실제 OCID/IP는 아직 생성하지 않았으면 비워 두고 해당 Create/readback 뒤 채운다. 동일 creation-event-id/OCID는 한 번의 제출만 뜻한다.  실습에 없는 자원을 추가하지 않는다. 실제 checkpoint는 사용자 실행 전 비워 둔다.

### 입력값과 실행 위치

Console에서 선택한 값만 사용한다. `<...>`는 예시 OCID나 비밀값이 아니라 **사용자가 대체할 입력 자리**다. 대체하지 않은 채 명령을 실행하지 않는다. Console → 우측 상단 Cloud Shell/Code Editor → Cloud Shell을 열면 OCI CLI가 사전 인증된다. Region 변경 뒤에는 Shell을 다시 열거나 명령의 `--region`을 바꾼다. SSH는 Cloud Shell이 아니라 본인 Linux/WSL/macOS 터미널에서 수행한다. 공인 IP/32도 그 터미널의 출구 주소여야 한다.

```bash
# 아래는 입력/출력의 대입 형식이다. 표의 단계 전에 생성 후 ID/IP export 행을 실행하지 않는다.
 # 생성 후 검증용 ID/IP는 해당 creator의 실제 응답에서 넣는다.
export REGION='<actual_REGION>'
export VCN_ID='<actual_VCN_ID>'
export IGW_ID='<actual_IGW_ID>'
export PUBLIC_SUBNET_ID='<actual_PUBLIC_SUBNET_ID>'
export PUBLIC_ROUTE_TABLE_ID='<actual_PUBLIC_ROUTE_TABLE_ID>'
export NSG_ADMIN_ID='<actual_NSG_ADMIN_ID>'
export PUBLIC_SECURITY_LIST_ID='<actual_PUBLIC_SECURITY_LIST_ID>'
export KEY='<actual_KEY>'
export A_PUBLIC_IP='<actual_A_PUBLIC_IP>'
export SUBNET_ID="$PUBLIC_SUBNET_ID"
export PUBLIC_IP="$A_PUBLIC_IP"
oci --version
```

이 변수 중 해당 절차에서 사용하는 값만 채운다. CLI 조회의 `data`를 증거 파일에 저장하고 실제 시간·작업 OCID·선택값을 기록한다. API key/private key/passphrase/인증 토큰은 기록하지 않는다. Cloud Shell 설정파일이나 세션 토큰을 출력하지 않는다. `oci --version`을 실행하는 주제의 기대 패턴은 버전 문자열뿐이다. namespace는 Object Storage 주제에서 별도 조회하며 다른 주제에는 요구하지 않는다. [Cloud Shell 공식 본문](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/cloudshellgettingstarted.htm).

### Workshop 입력 계약 — 단계와 host

| 입력/출력 | 값 또는 실제 획득 경로 | 사용할 시점 | 실행 위치 | 선택 이유 |
|---|---|---|---|---|
| Console/설정 필드 | Names=igw-lab/rt-public/sl-public/nsg-admin/public-admin; enabled IGW; default route0.0.0.0/0→IGW; empty SL; source local/32 TCP22; egress TCP443; Regional/Public; DNS=publicadmin | 생성 전 선택 → 생성 뒤 readback | 본문 Console/로컬 지정 | 이 topic의 최소 생성 범위 |
| `PUBLIC_SECURITY_LIST_ID` | owner23 sl-public Create id | 생성 후/선행 완료 | Console/Cloud Shell | 빈 public SL association |
| `PUBLIC_ROUTE_TABLE_ID` | owner23 rt-public Create id | 생성 후/선행 완료 | Console/Cloud Shell | public subnet route association |
| `PUBLIC_IP` | 본문 실제 Create/readback 출력 | 생성 후/선행 완료 | 본문 지정 host | 해당 topic 대상만 지정 |
| `NSG_ADMIN_ID` | owner23 nsg-admin Create id | 생성 후/선행 완료 | Console/Cloud Shell | A VNIC membership |
| `KEY` | owner15 Part A 전용 private key 절대경로; 내용 전송 금지 | 준비 완료 | 로컬 SSH만 | 해당 공개키와 대응 |
| `IGW_ID` | owner23 igw-lab Create id | 생성 후/선행 완료 | Console/Cloud Shell | rt-public default target |
| `A_PUBLIC_IP` | owner10 vm-a VNIC public-ip | 선행 완료/생성 후 | 로컬 SSH | 관리 jump 주소 |
| `LAB_COMPARTMENT_ID` | owner00 Create 응답 OCID; topic00에서는 생성 뒤 | 선행 완료/생성 후 | Console/Cloud Shell | 정책/자원 scope |
| `PRIVATE_SUBNET_CIDR` | owner20 승인 계획; 기본 후보 10.77.20.0/24 | 생성 전 | 로컬/Console | private subnet 주소 |
| `PUBLIC_SUBNET_CIDR` | owner20 승인 계획; 기본 후보 10.77.10.0/24 | 생성 전 | 로컬/Console | public subnet 주소 |
| `PUBLIC_SUBNET_ID` | owner23 단계 A subnet Create id | 선행 완료/생성 후 | Console/Cloud Shell | public-admin 선택 |
| `REGION` | 승인된 기존 subscribed Region identifier | 생성 전 | Console/Cloud Shell | API endpoint/데이터 위치 |
| `ROUTE_TABLE_ID` | 대상 subnet readback data.route-table-id | 생성 후 | Cloud Shell | 실제 연결 RT 조회 |
| `SECURITY_LIST_ID` | 대상 subnet readback의 유일 security-list-ids 원소 | 생성 후 | Cloud Shell | 빈 SL/허용 합집합 조회 |
| `SUBNET_ID` | PUBLIC_SUBNET_ID | 생성 후/선행 완료 | Cloud Shell | 대상 subnet alias; 오래된 shell 값 재사용 금지 |
| `VCN_CIDR` | owner20 승인 계획; 기본 후보 10.77.0.0/16 | 생성 전 | 로컬/Console/Cloud Shell | 중첩/중복 검증 |
| `VCN_ID` | owner19 Create 응답 id | 선행 완료/생성 후 | Console/Cloud Shell | shared-vcn 참조 |

생성 전에는 행의 승인값/선행 완료값만 지정하고, 생성 후 ID/IP는 해당 owner 응답으로 채운다. Cloud Shell과 로컬 SSH의 환경변수는 공유되지 않는다. 로컬에는 KEY와 접속 주소만 다시 지정하고 guest disk 명령은 로그인한 guest에서만 실행한다. `<...>`가 남으면 해당 명령은 실행하지 않는다.

## 권한

VM 없는 단계 A에서 IGW/route table/SL/NSG/public-admin 생성·조회·정리와 subnet association 권한을 승인한다. route target IGW attach/detach와 VCN/DHCP/security 연결 의존을 확인한다. 단계 B의 Compute launch는 owner10 권한으로 수행하며 네트워크 정책이 instance-family 권한을 대신하지 않는다.

### 실제 operation과 공식 권한

- `UpdateRouteTable`: ROUTE_TABLE_UPDATE and INTERNET_GATEWAY_ATTACH (if creating a route rule that uses an internet gateway as a target) and INTERNET_GATEWAY_DETACH (if deleting a route rule that uses an internet gateway as a target) and DRG_ATTACH (if creating a route rule that uses a DRG as a target) and DRG_DETACH (if deleting a route rule that uses a DRG as a target) and PRIVATE_IP _ROUTE_TABLE_ATTACH (if creating a route rule that uses a private IP as a target) and PRIVATE_IP _ROUTE_TABLE_DETACH (if deleting a route rule that uses a private IP as a target) and LOCAL_PEERING _GATEWAY_ATTACH (if creating a route rule that uses an LPG as a target) and LOCAL_PEERING _GATEWAY_DETACH (if deleting a route rule that uses an LPG as a target) and NAT_GATEWAY_ATTACH (if creating a route rule that uses a NAT gateway as a target) and NAT_GATEWAY_DETACH (if deleting a route rule that uses a NAT gateway as a target) and SERVICE_GATEWAY_ATTACH (if creating a route rule that uses a service gateway as a target) and SERVICE_GATEWAY_DETACH (if deleting a route rule that uses a service gateway as a target) — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `AddNetworkSecurityGroupSecurityRules`: NETWORK_SECURITY_GROUP _UPDATE_SECURITY_RULES and NETWORK_SECURITY_GROUP_INSPECT if writing a rule that specifies a network security group as the source (for ingress rules) or destination (for egress rules) — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `CreateInternetGateway`: INTERNET_GATEWAY_CREATE and VCN_ATTACH — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `CreateNetworkSecurityGroup`: NETWORK_SECURITY_GROUP_CREATE and VCN_ATTACH — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `CreateRouteTable`: ROUTE_TABLE_CREATE and VCN_ATTACH and INTERNET_GATEWAY_ATTACH (if creating a route rule that uses an internet gateway as a target) and DRG_ATTACH (if creating a route rule that uses a DRG as a target) and PRIVATE_IP _ROUTE_TABLE_ATTACH (if creating a route rule that uses a private IP as a target) and LOCAL_PEERING _GATEWAY_ATTACH (if creating a route rule that uses an LPG as a target) and NAT_GATEWAY_ATTACH (if creating a route rule that uses a NAT gateway as a target) and SERVICE_GATEWAY_ATTACH (if creating a route rule that uses a service gateway as a target) — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `CreateSecurityList`: SECURITY_LIST_CREATE and VCN_ATTACH — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `CreateSubnet`: SUBNET_CREATE and VCN_ATTACH and ROUTE_TABLE_ATTACH and SECURITY_LIST_ATTACH and DHCP_ATTACH — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `DeleteInternetGateway`: INTERNET_GATEWAY_DELETE and VCN_DETACH — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `DeleteNetworkSecurityGroup`: NETWORK_SECURITY_GROUP_DELETE and VCN_DETACH — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `DeleteRouteTable`: ROUTE_TABLE_DELETE and VCN_DETACH and INTERNET_GATEWAY_DETACH (if deleting a route rule that uses an internet gateway as a target) and DRG_DETACH (if deleting a route rule that uses a DRG as a target) and PRIVATE_IP _ROUTE_TABLE_DETACH (if deleting a route rule that uses a private IP as a target) and LOCAL_PEERING _GATEWAY_DETACH (if deleting a route rule that uses an LPG as a target) and NAT_GATEWAY_DETACH (if deleting a route rule that uses a NAT gateway as a target) and SERVICE_GATEWAY_DETACH (if deleting a route rule that uses a service gateway as a target) — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `DeleteSecurityList`: SECURITY_LIST_DELETE and VCN_DETACH — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `DeleteSubnet`: SUBNET_DELETE and VCN_DETACH and ROUTE_TABLE_DETACH and SECURITY_LIST_DETACH and DHCP_DETACH — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `GetInternetGateway`: INTERNET_GATEWAY_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `GetRouteTable`: ROUTE_TABLE_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `GetSecurityList`: SECURITY_LIST_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `GetSubnet`: SUBNET_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `ListNetworkSecurityGroupSecurityRules`: NETWORK_SECURITY_GROUP _LIST_SECURITY_RULES — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).

위 permission 이름은 IAM policy resource-type이 아니다. 조직 관리자가 기존 group/compartment scope와 conditional dependency를 실제 API 표에 대조한다. 편의 family grant를 최소권한 정책으로 부르지 않으며 실습자는 policy 작성 권한을 자동 인수하지 않는다.

## 비용및주의사항

- VM 없는 단계 A는 public-admin·igw-lab·rt-public·빈 sl-public·nsg-admin만 생성한다. NAT/SGW/VM을 함께 만드는 wizard는 사용하지 않는다. 단계 B는 owner10 vm-a의 동일 boot/public IPv4/전송 비용 창에서 수행한다.
- SSH source는 본인 로컬 출구/32이고 인터넷 TCP22 전체 허용은 금지다. public subnet 이름만으로 접속 성공을 판단하지 않는다. Compute와 public frontend 소비자 반환 후 public network owner23가 subnet/자기 route·security·IGW를 정리하며 VCN은 보존한다.

실제 OCI 작업은 아직 수행하지 않았다. 기대값은 실행 시 대조할 조건이며 무료 사용이나 과금 강제 차단을 보장하지 않는다. 승인 expiry가 끝나면 새 시험을 중단하고 위 소유권 순서로 원복·반환한다.

## 실습구성

### 이번 Workshop에서 만드는 것과 진행 순서

VM 없는 단계 A에서 public-admin/IGW/rt-public/sl-public/nsg-admin을 만들고 단계 B는 vm-a launch 뒤 SSH/유한 HTTPS를 검증한다.

1. **진입/선택:** 19 VCN/20 CIDR. 사전준비 링크의 지정 단계/실제 출력이 없으면 제출하지 않는다.
2. **실제 작업:** IGW → default route → 빈 SL/restricted NSG → Regional public subnet → A readback → 10 launch. 아래 Console 필드/명령을 해당 host에서 순서대로 실행한다. 같은 자원은 이름이 아니라 OCID/event로 재사용한다.
3. **완료/복구/반환:** B SSH/HTTPS. 결과검증 실패는 문제해결로 이동하고 동일 Create를 무작정 재제출하지 않는다.

**전달/정리 범위:** PUBLIC_SUBNET_ID/IGW_ID/route/SL/NSG/source32; owner23가 service 소비자까지 끝난 후 정리. 아래 정리 단계는 이 소유권을 넘어서 실행하지 않는다.


- **이번 생성/재사용/출력:** 단계 A 신규 igw-lab, rt-public, sl-public, nsg-admin, public-admin. 단계 B Compute와 같은 vm-a event를 사용. 24/26/29 초안 완료를 요구하지 않고 해당 bootstrap 절차를 여기서 완결한다.
- **생성 목표:** Subnets의 단일 실제 생성 event에서 Public Subnets 옵션을 직접 선택하고 저장·기능 검증한다. 독립 중복 생성은 하지 않는다.
- **계약 보정:** 기존 creation-first 계약의 문서만 존재하는 선행(24/25/26/29 등)과 bootstrap 순환은 실제 절차를 포함한 00/19/23 단계 A로 대체했다. 변경된 typed dependency는 authoring ledger에 기록한다.

## 실습절차

### 명령 실행 gate — 실패하면 다음 생성으로 진행하지 않는다

1. 아래 Bash는 **전용 Bash 세션**에서만 사용한다. Console/Cloud Shell 조회, 본인 로컬 SSH, 로그인한 Oracle Linux guest를 섞지 않는다. `exit 1`은 이 전용 세션을 닫는 안전 중단이다. `<...>`·빈 입력·타 owner OCID가 남으면 실행하지 않는다.
2. 성공을 요구하는 파일 작성/조회/설치/attach 작업은 실패 시 중단한다. 실패한 API 뒤 기존 JSON이나 기존 다운로드 파일을 성공 근거로 읽지 않는다. 기대 deny는 `if command; then ...; else RC=$?; ...; fi`로 별도 기록하고, 실제 timeout/IAM 오류/로컬 파일 오류를 구별한다.
3. 첫 변경 전에 정확한 OCID·owner·현재 설정과 승인 expiry를 저장한다. 공유 자원 변경 중 다른 소비자가 시작하거나 원복값을 확인할 수 없으면 변경을 중단한다. 삭제 성공은 API 수락이 아니라 같은 identity/Region의 정확한 대상 재조회로 판정한다. `NotAuthorizedOrNotFound` 단독은 삭제 증거가 아니다.


### 개별 완주 보정 — 생성·검증·소유권

단계A는 owner19VCN+owner20주소계획만필요하다. CLI nsg조회는 --nsg-id. 단계B는 owner10launch이후새SSH/HTTPS이며source는본인로컬출구/32이다.

### public-admin bootstrap — VM 없이 완결되는 단계 A

1. Networking → Virtual cloud networks → `shared-vcn` → Internet Gateways → Create Internet Gateway. Name `igw-lab`, Compartment `lab-root`, Enabled 체크 후 생성한다. 단일 VCN에는 IGW 1개를 공유한다. [Internet Gateway 관리 본문](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/managingIGs.htm).
2. Route Tables → Create Route Table: Name `rt-public`, Compartment `lab-root`. Route rule의 Target Type **Internet Gateway**, Destination Type **CIDR Block**, Destination CIDR **0.0.0.0/0**, Target compartment `lab-root`, Target `igw-lab`. 생성하고 target OCID가 IGW와 같은지 확인한다. 이 CIDR는 목적지 route이며 SSH의 source CIDR가 아니다. [Route rule fields](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/managingroutetables.htm).
3. Security Lists → Create Security List: `sl-public`. Ingress 없음; Egress 없음으로 생성한다. default SL에 SSH-wide allow가 남아 있어도 **public-admin에 연결하지 않는다**. Network Security Groups → Create Network Security Group: `nsg-admin`, Compartment `lab-root`. Add rules에서 다음 두 규칙만 만든다. [Security Lists and NSG 허용 관계](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/securitylists.htm).

| 방향 | Stateful/Stateless | 주소 유형·값 | Protocol | source port | destination port | 이유 |
|---|---|---|---|---|---|---|
| Ingress | Stateful (Stateless 체크 해제) | Source CIDR `<본인_터미널_출구_public_IP>/32` | TCP | All | 22 | 본인 SSH만 허용 |
| Egress | Stateful | Destination CIDR `0.0.0.0/0` | TCP | All | 443 | 승인된 짧은 HTTPS 확인; 나머지 아웃바운드 불허 |

본인 출구 IP는 관리자 승인된 네트워크 확인 도구로 구한다. Cloud Shell의 주소를 본인 WSL 주소로 착각하지 않는다. Stateful SSH 응답은 연결 추적으로 허용된다. 이 최소 구성은 모든 OS update/DNS외부/NTP/ICMP 경로를 제공하는 일반 production 정책이 아니다. 필요한 서비스는 별도 승인 규칙을 추가한다.


**빈 custom SL 입력 대안:** Console이 최소 규칙 1개를 요구하거나 자동 default SSH/egress 규칙을 표시하면 그 값을 그대로 생성하지 않는다. Cloud Shell에서 아래처럼 **두 배열을 명시적으로 빈 배열**로 지정해 생성하고 신규 OCID를 사용한다. 이미 같은 이름/OCID의 SL이 생성되어 있으면 중복 제출하지 않는다. `[]`를 생략하면 서비스 기본 규칙에 의존할 수 있으므로 반드시 명시한다. [Security List creation body](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/creating-securitylist.htm) 및 [CLI create — required ingress/egress JSON arrays](https://docs.oracle.com/iaas/tools/oci-cli/latest/oci_cli_docs/cmdref/network/security-list/create.html).

```bash
oci network security-list create --compartment-id "$LAB_COMPARTMENT_ID" --vcn-id "$VCN_ID" --display-name 'sl-public' --ingress-security-rules '[]' --egress-security-rules '[]' --region "$REGION" > empty-security-list-created.json || exit 1
```

응답의 `id`와 AVAILABLE 상태를 확인하고 `ingress-security-rules=[]`, `egress-security-rules=[]`를 readback한다. 이 API/CLI 경로도 사용자의 tenancy에서 미실행이므로 거부 시 실패를 그대로 기록하고 관리자에게 검토받는다.

4. Subnets → Create subnet: `public-admin`, Compartment `lab-root`, **Regional**, CIDR `10.77.10.0/24`, **Public subnet**, Route Table `rt-public`, Security List **sl-public만**, DHCP 기본, DNS hostnames Yes / label `publicadmin`. IPv6와 resource logging은 선택하지 않는다. Create 제출 후 AVAILABLE/route association/DNS suffix를 읽는다. [Public subnet 생성 필드](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/create_subnet.htm).
5. 출력 `PUBLIC_SUBNET_ID`, `NSG_ADMIN_ID`, `IGW_ID`, `PUBLIC_ROUTE_TABLE_ID`, learner source/32를 registry에 저장한다. **단계 A는 Compute 완료를 요구하지 않는다.** Compute는 이 출력으로 launch하므로 순환이 없다. 단계 B는 Compute launch 뒤 공인 IP·새 SSH 세션·finite HTTPS 요청을 검증한다.
<a id="public-network-preparation"></a>

### 단계 A 완료·Compute 전달 (VM 불필요)

Cloud Shell에서 아래 조회가 모두 성공한 뒤 JSON의 AVAILABLE/public flag=false, 빈 sl-public 양 방향, IGW enabled, rt-public default target=IGW_ID, nsg-admin SSH source/32·443 egress를 제출값과 대조한다. 실패하면 Compute launch로 진행하지 않는다. 출력 ID와 source/32, owner23, consumer10, 승인 expiry를 전달한다. 이 anchor는 **완료 확인 station**이며 여기 도착했다고 자동 완료가 아니다.

```bash
export PUBLIC_SUBNET_ID='<A4_Create_response_id>'
export PUBLIC_ROUTE_TABLE_ID='<A2_Create_response_id>'
export PUBLIC_SECURITY_LIST_ID='<A3_Create_response_id>'
export IGW_ID='<A1_Create_response_id>'
export NSG_ADMIN_ID='<A3_NSG_Create_response_id>'
oci network subnet get --subnet-id "$PUBLIC_SUBNET_ID" --region "$REGION" > public-A-subnet.json || exit 1
oci network route-table get --rt-id "$PUBLIC_ROUTE_TABLE_ID" --region "$REGION" > public-A-route.json || exit 1
oci network security-list get --security-list-id "$PUBLIC_SECURITY_LIST_ID" --region "$REGION" > public-A-security.json || exit 1
oci network internet-gateway get --ig-id "$IGW_ID" --region "$REGION" > public-A-igw.json || exit 1
oci network nsg rules list --all --nsg-id "$NSG_ADMIN_ID" --region "$REGION" > public-A-nsg.json || exit 1
python3 - <<'PY'
import json,os
read=lambda n:json.load(open('public-A-'+n+'.json'))['data']
s,r,l,g=map(read,['subnet','route','security','igw'])
assert s['lifecycle-state']=='AVAILABLE' and s['availability-domain'] is None
assert s['prohibit-public-ip-on-vnic'] is False
assert s['route-table-id']==os.environ['PUBLIC_ROUTE_TABLE_ID']
assert s['security-list-ids']==[os.environ['PUBLIC_SECURITY_LIST_ID']]
assert l['ingress-security-rules']==[] and l['egress-security-rules']==[]
assert g['is-enabled'] is True
assert any(x['destination']=='0.0.0.0/0' and x['network-entity-id']==os.environ['IGW_ID'] for x in r['route-rules'])
print('A network association checks complete; inspect exact NSG rules before handoff')
PY
if [ "$?" -ne 0 ]; then exit 1; fi
```

NSG JSON의 두 규칙 값까지 사람이 확인하고 서명한 뒤에만 A 완료로 기록한다. 신규 추가 rule이 이미 있으면 그 owner/consumer도 기록한다.

### 단계 B — launch 후 기능 검증

Compute lab에서 기존 public-admin을 선택하여 vm-a를 생성한 뒤 이 단계로 돌아온다. public IP가 존재하고 NSG membership이 nsg-admin인지 확인한다. 본인 source IP가 바뀌면 승인 /32 규칙을 수정하고 readback한다. 새 SSH 세션으로 로그인한 뒤 guest에서 `curl --connect-timeout 5 --max-time 15 -I https://www.oracle.com/`를 한 번 실행한다(이 endpoint가 조직 승인된 경우). 실제 상태코드와 종료코드를 기록한다. HTTP 2xx/3xx는 경로 확인 패턴이며 콘텐츠/SLA 보장이 아니다. 외부 DNS/프록시가 차단되면 route와 분리해 진단한다. 승인 endpoint가 없으면 HTTPS 검증은 BLOCKED_POLICY로 남긴다.

## 결과검증

<a id="baseline-23-ready"></a>

```bash
oci network subnet get --subnet-id "$SUBNET_ID" --region "$REGION" > subnet-readback.json || exit 1
export ROUTE_TABLE_ID='<subnet_data_route-table-id>'
export SECURITY_LIST_ID='<subnet_data_security-list-ids에서_유일한_OCID>'
oci network route-table get --rt-id "$ROUTE_TABLE_ID" --region "$REGION" > route-readback.json || exit 1
oci network security-list get --security-list-id "$SECURITY_LIST_ID" --region "$REGION" > security-readback.json || exit 1
```
기대 패턴: `lifecycle-state=AVAILABLE`, `availability-domain=null`(Regional), `vcn-id`/`cidr-block`/`route-table-id`/`security-list-ids`/`dhcp-options-id`가 registry와 일치. private는 `prohibit-public-ip-on-vnic=true`, public은 false. 빈 rt-private의 `route-rules=[]`; rt-public은 `destination=0.0.0.0/0`, `network-entity-id=IGW_ID`다. false만으로 인터넷 도달을 판정하지 않는다.
```bash
oci network internet-gateway get --ig-id '<igw-lab_OCID>' --region "$REGION" > igw-readback.json || exit 1
oci network nsg rules list --nsg-id '<nsg-admin_OCID>' --region "$REGION" > admin-rules.json || exit 1
```
단계 A 기대: IGW is-enabled=true, public-admin public flag false, rt-public IGW target 일치, NSG ingress source=본인/32/destination-port=22, sl-public ingress=[]; 단계 B 기대: VNIC public-ip non-null, source에서 SSH exit 0, 승인 HTTPS finite 요청 성공. VNIC의 모든 NSG와 subnet의 모든 SL을 읽어 광범위 SSH 허용이 없는지 확인한다.

**증거 기록:** 실제 명령·UTC 시간·exit code·가려진 OCID·옵션 readback·검증 조건을 남긴다. 현재 이 문서의 실행상태는 `NOT_EXECUTED`다. output/creation-event-id/result는 실행 전 비워 둔다. 오류/NOT_SELECTED를 PASS로 바꾸지 않는다.

### 이 시나리오의 functional acceptance

| Positive | Negative / 미완료 판정 | Recovery / 반환 |
|---|---|---|
| VM없는 network A ready; VM뒤 SSH/HTTPS | public flag만 있고 IP/IGW/route/rules 누락이면 미완료 | 실제 누락된 creator 출력 복구/readback; default broad SL 추가 없음 |

모두 실행 시 확인할 기대 패턴이다. 생성 전 guard negative는 OCI 장애를 주입한 것과 다르며 다른 자원을 추가하지 않는다. 오류 메시지와 실제 exit code가 없는 항목은 NOT_EXECUTED 또는 BLOCKED로 남긴다.

## 문제해결

### 공통 오류의 진단 순서

1. `NotAuthorizedOrNotFound` 또는 목록에 리소스가 없음: 먼저 Region/Compartment/OCID와 identity domain/group을 확인한다. 존재하지 않는 OCID와 권한 부족 모두 같은 오류로 보일 수 있다. 관리자에게 request-id·작업 시각·서비스·가려진 대상 OCID를 제공한다. 비밀키/토큰은 보내지 않는다.
2. `LimitExceeded`/quota/capacity 부족: Billing & Cost Management → Limits, Quotas and Usage와 생성 화면의 지원 shape/AD를 확인한다. 운영 quota 증가나 다른 Region 구독을 임의 요청하지 않는다. 승인 범위 안의 더 작은 compatible shape 또는 다른 AD를 선택하거나 `BLOCKED_CAPACITY`를 기록한다.
3. 장시간 provisioning/예상 밖 state: Work Requests/오류 상세를 읽고 같은 요청을 반복 제출하지 않는다. 만들어진 부분 리소스(boot/IP/attachment 포함)를 먼저 조사한다.
4. 메뉴 이름이 다름: 공식 task body의 동등한 탭(예: Storage 또는 Attached Block Volumes)을 찾는다. 필드가 문서와 다르면 값·스크린샷에서 secrets를 가린 뒤 관리자 또는 Oracle Support에 확인하고 제출을 중단한다. 이 문서는 현재 tenancy의 live Console 화면을 검사한 결과가 아니다.

### SSH 실패

`Connection timed out`이면 공인 IP → IGW enabled → subnet route의 IGW target → security list와 모든 NSG의 유효 허용 규칙 → guest sshd/firewall 순으로 확인한다. NSG와 security list는 허용 규칙의 합집합이므로 좁은 NSG가 넓은 security list를 취소하지 않는다. `Permission denied (publickey)`이면 Linux 사용자(opc; 이 트랙은 Oracle Linux 9), 제출한 `.pub`, 대응 private key와 권한을 확인한다. 호스트키 변경은 재생성 여부를 확인한 뒤 처리하며 `StrictHostKeyChecking=no`로 우회하지 않는다. 복구가 필요하면 공식 serial Console 절차를 관리자의 승인하에 사용한다. Ping 불가만으로 SSH 실패를 판정하지 않는다.

### 이 주제의 실패 지점

public flag false는 할당을 허용한다는 뜻일 뿐 실제 public IP가 있음을 뜻하지 않는다. VNIC public IP/IGW enabled/default route/security/guest firewall을 각각 조회한다. PMTU 대형 전송 문제는 ICMP3/4 및 path MTU 규칙을 별도 승인 후 추가한다.

## 리소스정리

1. owner10/07/12/16/18 VM/VNIC뿐 아니라 owner02 `lab-platform` Functions application의 `subnet-ids` 참조도 확인한다. 승인 소비자/expiry가 남으면 public-admin/IGW/route/SL/NSG를 보존한다. 소비자는 자기 owner의 정리 절차로 삭제/반환한다; owner23가 다른 owner의 VM/application을 임의 삭제하지 않는다.
2. 모든 public-admin 참조가 해제된 뒤 정확한 PUBLIC_SUBNET_ID만 Delete하고 get/list에서 제거를 확인한다. private-app은 owner21의 대상이다.
3. rt-public의 IGW route를 제거한 뒤 igw-lab을 삭제한다. nsg-admin의 membership이 비었는지 읽은 후 nsg-admin, 참조가 없는 sl-public/rt-public을 지운다. 각 정확한 OCID의 get/list를 확인한다. default RT/SL/DHCP 및 private rt/sl은 삭제하지 않는다.
4. VCN과 Compartment는 owner19/00에게 반환한다. API 수락 또는 NotAuthorized 오류를 삭제 성공으로 쓰지 않는다.


## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 아래 본문은 각 생성 필드·설정·검증·정리의 기술 근거이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [30] [Networking Overview — Public Subnets](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/overview.htm)
  - 확인 범위: Public means VNICs in the subnet can.

### 추가 실행 절차 근거 (본문 확인)

- [subnet](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/create_subnet.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [igw](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/managingIGs.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [route](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/managingroutetables.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [security](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/securitylists.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [launch](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/launchinginstance.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [cloudshell](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/cloudshellgettingstarted.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.

- [추가 세부 task: slcreate](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/creating-securitylist.htm) — 실제 본문을 확인한 필드·API/권한·삭제 단계 근거.

- [추가 세부 task: routecreate](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/create-routetable.htm) — 실제 본문을 확인한 필드·API/권한·삭제 단계 근거.

- [추가 세부 task: igwcreate](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/create-ig.htm) — 실제 본문을 확인한 필드·API/권한·삭제 단계 근거.

- [NSG directions, union, memberships](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/networksecuritygroups.htm) — 규칙 방향·VNIC association·합집합 및 삭제 선행조건 본문.
