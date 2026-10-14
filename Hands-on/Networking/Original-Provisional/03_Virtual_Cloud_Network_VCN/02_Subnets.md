> **기존 실습 초안 — 미검증**: 원래 93개 문서의 절차를 보존한 자료입니다. 새 튜토리얼의 품질 승인이나 OCI 실행 검증을 뜻하지 않습니다.

# Subnets

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../../../../Docs/README.md)를 확인하세요.

> 실행 절차 작성본 / OCI 미실행. Oracle 공식 본문 근거와 교육용 선택을 구분한다. secrets·실제 고객값·diagram 없음.

## 학습목표

- shared-vcn에 빈 rt-private/sl-private를 만들고 Regional·Private·승인 CIDR·DNS/privateapp·기존 DHCP로 private-app을 생성한다.
- subnet의 route/SL/DHCP OCID와 public-IP 금지 flag를 조회해 creator 출력으로 전달하고 private 소비자 반환 후 subnet→RT/SL 순서로 정리한다.

## 대응 Roadmap 항목

- 항목명: **Subnets**
- Roadmap 경로: **Virtual Cloud Network (VCN) → Subnets**
- Node ID: `sequence-3-2`
- 상위 항목: **Virtual Cloud Network (VCN)** (`sequence-3-0`)
- 문서 범위: shared-vcn에 빈 rt-private/sl-private를 만들고 Regional·Private·승인 CIDR·DNS/privateapp·기존 DHCP로 private-app을 생성한다. subnet의 route/SL/DHCP OCID와 public-IP 금지 flag를 조회해 creator 출력으로 전달하고 private 소비자 반환 후 subnet→RT/SL 순서로 정리한다. 실제 OCI 실행 결과가 아니라 이 절차의 완료 조건이다.
- 번호는 navigation 식별자이며 실행 순서는 사전준비의 단계별 의존성에 따른다.

## 사전준비

### 기준 계약과 실제 단계 선행조건

[공통 baseline-contract.json](../../../../baseline-contract.json)의 Oracle Linux 9 / opc / CIDR / 비용 상한을 따른다. 문서 번호는 실행 순서가 아니다. **준비 station과 생성 이후 ready를 분리**한다. 아래 단계 외의 기능완료를 소급하여 요구하지 않는다.

- [19 Virtual Cloud Network (VCN) — 단계 링크](00_Virtual_Cloud_Network_VCN.md#baseline-19-ready): **AVAILABLE VCN/default DHCP**. 실제 출력/소비자/expiry를 전달하고 해당 owner가 이 lab 종료까지 보존한다.
- [20 CIDR Blocks — 단계 링크](01_CIDR_Blocks.md#cidr-planning): **단계 A 승인 private CIDR**. 실제 출력/소비자/expiry를 전달하고 해당 owner가 이 lab 종료까지 보존한다.

생성 전 조직 승인(identity/권한/예산/expiry/서비스 지원/limit/capacity)을 확인한다. 실제 OCID/IP는 아직 생성하지 않았으면 비워 두고 해당 Create/readback 뒤 채운다. 동일 creation-event-id/OCID는 한 번의 제출만 뜻한다.  실습에 없는 자원을 추가하지 않는다. 실제 checkpoint는 사용자 실행 전 비워 둔다.

### 입력값과 실행 위치

Console에서 선택한 값만 사용한다. `<...>`는 예시 OCID나 비밀값이 아니라 **사용자가 대체할 입력 자리**다. 대체하지 않은 채 명령을 실행하지 않는다. Console → 우측 상단 Cloud Shell/Code Editor → Cloud Shell을 열면 OCI CLI가 사전 인증된다. Region 변경 뒤에는 Shell을 다시 열거나 명령의 `--region`을 바꾼다. SSH는 Cloud Shell이 아니라 본인 Linux/WSL/macOS 터미널에서 수행한다. 공인 IP/32도 그 터미널의 출구 주소여야 한다.

```bash
# 아래는 입력/출력의 대입 형식이다. 표의 단계 전에 생성 후 ID/IP export 행을 실행하지 않는다.
 # 생성 후 검증용 ID/IP는 해당 creator의 실제 응답에서 넣는다.
export REGION='<actual_REGION>'
export PRIVATE_SUBNET_ID='<actual_PRIVATE_SUBNET_ID>'
export VCN_ID='<actual_VCN_ID>'
export PRIVATE_ROUTE_TABLE_ID='<actual_PRIVATE_ROUTE_TABLE_ID>'
export PRIVATE_SECURITY_LIST_ID='<actual_PRIVATE_SECURITY_LIST_ID>'
export SUBNET_ID="$PRIVATE_SUBNET_ID"
oci --version
```

이 변수 중 해당 절차에서 사용하는 값만 채운다. CLI 조회의 `data`를 증거 파일에 저장하고 실제 시간·작업 OCID·선택값을 기록한다. API key/private key/passphrase/인증 토큰은 기록하지 않는다. Cloud Shell 설정파일이나 세션 토큰을 출력하지 않는다. `oci --version`을 실행하는 주제의 기대 패턴은 버전 문자열뿐이다. namespace는 Object Storage 주제에서 별도 조회하며 다른 주제에는 요구하지 않는다. [Cloud Shell 공식 본문](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/cloudshellgettingstarted.htm).

### Workshop 입력 계약 — 단계와 host

| 입력/출력 | 값 또는 실제 획득 경로 | 사용할 시점 | 실행 위치 | 선택 이유 |
|---|---|---|---|---|
| Console/설정 필드 | Names=rt-private/sl-private/private-app; RT rules=[]; SL ingress/egress=[]; Regional/Private; DNS=privateapp; existing default DHCP | 생성 전 선택 → 생성 뒤 readback | 본문 Console/로컬 지정 | 이 topic의 최소 생성 범위 |
| `PRIVATE_SECURITY_LIST_ID` | owner21 sl-private Create id | 생성 후/선행 완료 | Console/Cloud Shell | 빈 private SL |
| `PRIVATE_ROUTE_TABLE_ID` | owner21 rt-private Create id | 생성 후/선행 완료 | Console/Cloud Shell | 빈 private routes |
| `LAB_COMPARTMENT_ID` | owner00 Create 응답 OCID; topic00에서는 생성 뒤 | 선행 완료/생성 후 | Console/Cloud Shell | 정책/자원 scope |
| `PRIVATE_SUBNET_CIDR` | owner20 승인 계획; 기본 후보 10.77.20.0/24 | 생성 전 | 로컬/Console | private subnet 주소 |
| `PRIVATE_SUBNET_ID` | owner21 subnet Create id | 선행 완료/생성 후 | Console/Cloud Shell | private-app 선택 |
| `PUBLIC_SUBNET_CIDR` | owner20 승인 계획; 기본 후보 10.77.10.0/24 | 생성 전 | 로컬/Console | public subnet 주소 |
| `REGION` | 승인된 기존 subscribed Region identifier | 생성 전 | Console/Cloud Shell | API endpoint/데이터 위치 |
| `ROUTE_TABLE_ID` | 대상 subnet readback data.route-table-id | 생성 후 | Cloud Shell | 실제 연결 RT 조회 |
| `SECURITY_LIST_ID` | 대상 subnet readback의 유일 security-list-ids 원소 | 생성 후 | Cloud Shell | 빈 SL/허용 합집합 조회 |
| `SUBNET_ID` | PRIVATE_SUBNET_ID | 생성 후/선행 완료 | Cloud Shell | 대상 subnet alias; 오래된 shell 값 재사용 금지 |
| `VCN_CIDR` | owner20 승인 계획; 기본 후보 10.77.0.0/16 | 생성 전 | 로컬/Console/Cloud Shell | 중첩/중복 검증 |
| `VCN_ID` | owner19 Create 응답 id | 선행 완료/생성 후 | Console/Cloud Shell | shared-vcn 참조 |

생성 전에는 행의 승인값/선행 완료값만 지정하고, 생성 후 ID/IP는 해당 owner 응답으로 채운다. Cloud Shell과 로컬 SSH의 환경변수는 공유되지 않는다. 로컬에는 KEY와 접속 주소만 다시 지정하고 guest disk 명령은 로그인한 guest에서만 실행한다. `<...>`가 남으면 해당 명령은 실행하지 않는다.

## 권한

빈 rt-private/sl-private 생성·조회·삭제와 private-app Create/Get/Delete를 승인한다. CreateSubnet에는 기존 VCN/route/security-list/DHCP 연결 권한이 함께 필요하다. 빈 route를 생성하므로 DRG/LPG/NAT/SGW target attach는 기본에서 선택하지 않는다. VM/boot launch·관리 SSH는 후속 owner22의 승인 범위다.

### 실제 operation과 공식 권한

- `CreateRouteTable`: ROUTE_TABLE_CREATE and VCN_ATTACH and INTERNET_GATEWAY_ATTACH (if creating a route rule that uses an internet gateway as a target) and DRG_ATTACH (if creating a route rule that uses a DRG as a target) and PRIVATE_IP _ROUTE_TABLE_ATTACH (if creating a route rule that uses a private IP as a target) and LOCAL_PEERING _GATEWAY_ATTACH (if creating a route rule that uses an LPG as a target) and NAT_GATEWAY_ATTACH (if creating a route rule that uses a NAT gateway as a target) and SERVICE_GATEWAY_ATTACH (if creating a route rule that uses a service gateway as a target) — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `CreateSecurityList`: SECURITY_LIST_CREATE and VCN_ATTACH — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `CreateSubnet`: SUBNET_CREATE and VCN_ATTACH and ROUTE_TABLE_ATTACH and SECURITY_LIST_ATTACH and DHCP_ATTACH — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `DeleteRouteTable`: ROUTE_TABLE_DELETE and VCN_DETACH and INTERNET_GATEWAY_DETACH (if deleting a route rule that uses an internet gateway as a target) and DRG_DETACH (if deleting a route rule that uses a DRG as a target) and PRIVATE_IP _ROUTE_TABLE_DETACH (if deleting a route rule that uses a private IP as a target) and LOCAL_PEERING _GATEWAY_DETACH (if deleting a route rule that uses an LPG as a target) and NAT_GATEWAY_DETACH (if deleting a route rule that uses a NAT gateway as a target) and SERVICE_GATEWAY_DETACH (if deleting a route rule that uses a service gateway as a target) — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `DeleteSecurityList`: SECURITY_LIST_DELETE and VCN_DETACH — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `DeleteSubnet`: SUBNET_DELETE and VCN_DETACH and ROUTE_TABLE_DETACH and SECURITY_LIST_DETACH and DHCP_DETACH — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `GetRouteTable`: ROUTE_TABLE_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `GetSecurityList`: SECURITY_LIST_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `GetSubnet`: SUBNET_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).

위 permission 이름은 IAM policy resource-type이 아니다. 조직 관리자가 기존 group/compartment scope와 conditional dependency를 실제 API 표에 대조한다. 편의 family grant를 최소권한 정책으로 부르지 않으며 실습자는 policy 작성 권한을 자동 인수하지 않는다.

## 비용및주의사항

- 생성 범위는 private-app Regional subnet 1개와 빈 rt-private/sl-private 각1개다. VM/공인 IP/NAT/SGW는 이 단계에서 생성하지 않는다. 후속 private-vm의 Compute/boot 비용은 owner22가 별도 승인한다.
- private flag·빈 양 방향 SL·route/DHCP association을 제출 후 확인한다. default wide allow를 대신 연결하지 않는다. 모든 private VNIC/서비스 소비자 반환 후 subnet→자기 RT/SL 순으로 정리하며 default DHCP/VCN은 owner19에게 반환한다.

실제 OCI 작업은 아직 수행하지 않았다. 기대값은 실행 시 대조할 조건이며 무료 사용이나 과금 강제 차단을 보장하지 않는다. 승인 expiry가 끝나면 새 시험을 중단하고 위 소유권 순서로 원복·반환한다.

## 실습구성

### 이번 Workshop에서 만드는 것과 진행 순서

shared-vcn에 private-app Regional subnet과 빈 rt-private/sl-private를 한 번 만든다. VM은 owner22 단계다.

1. **진입/선택:** 19 AVAILABLE/20 주소 계획. 사전준비 링크의 지정 단계/실제 출력이 없으면 제출하지 않는다.
2. **실제 작업:** rt-private rules=[] → sl-private ingress/egress=[] → Regional/private/DNS/DHCP 선택 Create. 아래 Console 필드/명령을 해당 host에서 순서대로 실행한다. 같은 자원은 이름이 아니라 OCID/event로 재사용한다.
3. **완료/복구/반환:** associations readback. 결과검증 실패는 문제해결로 이동하고 동일 Create를 무작정 재제출하지 않는다.

**전달/정리 범위:** PRIVATE_SUBNET_ID/PRIVATE_ROUTE_TABLE_ID/PRIVATE_SECURITY_LIST_ID; owner21가 소비자 종료 후 정리. 아래 정리 단계는 이 소유권을 넘어서 실행하지 않는다.


- **이번 생성/재사용/출력:** 신규 private-app/rt-private/sl-private, existing DHCP 재사용. 생성·삭제 owner는21 Subnets,22는 동일 subnet의 VM 소비자; public subnet과 중복하지 않는다.
- **생성 목표:** 21에서 private-app subnet을 한 번 생성하고22에 전달하는 station: 실제 Regional/Private/CIDR/route table/security/DHCP/DNS 설정을 선택하며 생성한다. Subnets 일반 개념을 위해 scope-lab이라는 추가 subnet을 만들지 않는다.
- **계약 보정:** 기존 creation-first 계약의 문서만 존재하는 선행(24/25/26/29 등)과 bootstrap 순환은 실제 절차를 포함한 00/19/23 단계 A로 대체했다. 변경된 typed dependency는 authoring ledger에 기록한다.

## 실습절차

### 명령 실행 gate — 실패하면 다음 생성으로 진행하지 않는다

1. 아래 Bash는 **전용 Bash 세션**에서만 사용한다. Console/Cloud Shell 조회, 본인 로컬 SSH, 로그인한 Oracle Linux guest를 섞지 않는다. `exit 1`은 이 전용 세션을 닫는 안전 중단이다. `<...>`·빈 입력·타 owner OCID가 남으면 실행하지 않는다.
2. 성공을 요구하는 파일 작성/조회/설치/attach 작업은 실패 시 중단한다. 실패한 API 뒤 기존 JSON이나 기존 다운로드 파일을 성공 근거로 읽지 않는다. 기대 deny는 `if command; then ...; else RC=$?; ...; fi`로 별도 기록하고, 실제 timeout/IAM 오류/로컬 파일 오류를 구별한다.
3. 첫 변경 전에 정확한 OCID·owner·현재 설정과 승인 expiry를 저장한다. 공유 자원 변경 중 다른 소비자가 시작하거나 원복값을 확인할 수 없으면 변경을 중단한다. 삭제 성공은 API 수락이 아니라 같은 identity/Region의 정확한 대상 재조회로 판정한다. `NotAuthorizedOrNotFound` 단독은 삭제 증거가 아니다.


### 개별 완주 보정 — 생성·검증·소유권

sl-private는 ingress/egress[]를 읽고 다르면 생성값을 교정한다. 빈SL CLI는 explicit[]를 쓰고OCID중복제출하지 않는다. subnet생성 owner21와 private-vm owner22를 분리한다.

### private-app 생성 station

1. Networking → Virtual cloud networks → `shared-vcn` → Route Tables → Create Route Table: Name `rt-private`, Compartment `lab-root`, **route rule 없음**으로 Create. VCN 내부 라우팅은 별도의 인터넷 default route가 없어도 제공된다. NAT/IGW를 추가하지 않는다. [VCN Route Tables](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/managingroutetables.htm).
2. 같은 VCN → Security Lists → Create Security List: `sl-private`, Compartment `lab-root`, ingress 없음, egress 없음으로 생성한다. default security list를 대신 선택하지 않는다. NSG로 필요한 허용만 나중에 추가한다. DHCP options는 VCN의 기본 Internet and VCN Resolver를 재사용한다. [Security Lists](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/securitylists.htm).


**빈 custom SL 입력 대안:** Console이 최소 규칙 1개를 요구하거나 자동 default SSH/egress 규칙을 표시하면 그 값을 그대로 생성하지 않는다. Cloud Shell에서 아래처럼 **두 배열을 명시적으로 빈 배열**로 지정해 생성하고 신규 OCID를 사용한다. 이미 같은 이름/OCID의 SL이 생성되어 있으면 중복 제출하지 않는다. `[]`를 생략하면 서비스 기본 규칙에 의존할 수 있으므로 반드시 명시한다. [Security List creation body](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/creating-securitylist.htm) 및 [CLI create — required ingress/egress JSON arrays](https://docs.oracle.com/iaas/tools/oci-cli/latest/oci_cli_docs/cmdref/network/security-list/create.html).

```bash
oci network security-list create --compartment-id "$LAB_COMPARTMENT_ID" --vcn-id "$VCN_ID" --display-name 'sl-private' --ingress-security-rules '[]' --egress-security-rules '[]' --region "$REGION" > empty-security-list-created.json || exit 1
```

응답의 `id`와 AVAILABLE 상태를 확인하고 `ingress-security-rules=[]`, `egress-security-rules=[]`를 readback한다. 이 API/CLI 경로도 사용자의 tenancy에서 미실행이므로 거부 시 실패를 그대로 기록하고 관리자에게 검토받는다.

3. Subnets → Create subnet: Name `private-app`, Compartment `lab-root`, Subnet type **Regional**, IPv4 CIDR `10.77.20.0/24`(사전 승인 계획으로 교체 가능), Subnet access **Private subnet**, Route Table `rt-private`, Security List **오직 sl-private**, DHCP options 기존 default, Use DNS hostnames 선택 / label `privateapp`. IPv6 없음, Resource logging 꺼짐, 별도 replication 없음. 선택값을 기록한 뒤 Create subnet을 제출한다. [Creating a Subnet — Regional/Private/DNS/security associations](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/create_subnet.htm).
4. subnet OCID와 AVAILABLE를 확인한다. route/security/DHCP OCID를 기록한다. subnet 자체 생성은 여기까지 완결되며 private VM의 실제 연결은 Private Subnets lab에서 수행한다. Subnets 주제 때문에 추가 `scope-lab` subnet을 만들지 않는다.
5. 같은 생성 event를 Private Subnets 주제와 공유한다. 전용 route/security association과 Regional 선택을 기록한다. `private-app` 생성이 끝나면 Private Subnets lab에서 private-vm의 실제 VNIC를 이 subnet에 넣고 테스트한다. 여기서는 public-admin을 다시 만들지 않는다.

## 결과검증


### baseline-21-ready


```bash
oci network subnet get --subnet-id "$SUBNET_ID" --region "$REGION" > subnet-readback.json || exit 1
export ROUTE_TABLE_ID='<subnet_data_route-table-id>'
export SECURITY_LIST_ID='<subnet_data_security-list-ids에서_유일한_OCID>'
oci network route-table get --rt-id "$ROUTE_TABLE_ID" --region "$REGION" > route-readback.json || exit 1
oci network security-list get --security-list-id "$SECURITY_LIST_ID" --region "$REGION" > security-readback.json || exit 1
```
기대 패턴: `lifecycle-state=AVAILABLE`, `availability-domain=null`(Regional), `vcn-id`/`cidr-block`/`route-table-id`/`security-list-ids`/`dhcp-options-id`가 registry와 일치. 이번 private-app은 `prohibit-public-ip-on-vnic=true`, initial rt-private의 `route-rules=[]`와 현재 security-readback의 `ingress-security-rules=[]`, `egress-security-rules=[]` 두 배열 모두를 요구한다. NAT/SGW 소비 중에는 해당 승인 route가 있으므로 initial stage와 시험 중 readback을 구분한다.추가로 `dns-label=privateapp` 및 `subnet-domain-name`이 VCN DNS suffix 아래인지 확인한다. private-vm 후속 검증 때 VNIC subnet-id가 이 OCID인지 확인하되 subnet 생성 완료와 VM 완료를 분리한다.

**증거 기록:** 실제 명령·UTC 시간·exit code·가려진 OCID·옵션 readback·검증 조건을 남긴다. 현재 이 문서의 실행상태는 `NOT_EXECUTED`다. output/creation-event-id/result는 실행 전 비워 둔다. 오류/NOT_SELECTED를 PASS로 바꾸지 않는다.

### 이 시나리오의 functional acceptance

| Positive | Negative / 미완료 판정 | Recovery / 반환 |
|---|---|---|
| Regional/private flag/rt-private/SL empty/DNS match | wide default SL/AD-specific 등 선택값 불일치면 제출 중단 | Create 전 exact fields 교정; 이미 생성한 target은 변경지원/creator 승인 필요 |

모두 실행 시 확인할 기대 패턴이다. 생성 전 guard negative는 OCI 장애를 주입한 것과 다르며 다른 자원을 추가하지 않는다. 오류 메시지와 실제 exit code가 없는 항목은 NOT_EXECUTED 또는 BLOCKED로 남긴다.

## 문제해결

### 공통 오류의 진단 순서

1. `NotAuthorizedOrNotFound` 또는 목록에 리소스가 없음: 먼저 Region/Compartment/OCID와 identity domain/group을 확인한다. 존재하지 않는 OCID와 권한 부족 모두 같은 오류로 보일 수 있다. 관리자에게 request-id·작업 시각·서비스·가려진 대상 OCID를 제공한다. 비밀키/토큰은 보내지 않는다.
2. `LimitExceeded`/quota/capacity 부족: Billing & Cost Management → Limits, Quotas and Usage와 생성 화면의 지원 shape/AD를 확인한다. 운영 quota 증가나 다른 Region 구독을 임의 요청하지 않는다. 승인 범위 안의 더 작은 compatible shape 또는 다른 AD를 선택하거나 `BLOCKED_CAPACITY`를 기록한다.
3. 장시간 provisioning/예상 밖 state: Work Requests/오류 상세를 읽고 같은 요청을 반복 제출하지 않는다. 만들어진 부분 리소스(boot/IP/attachment 포함)를 먼저 조사한다.
4. 메뉴 이름이 다름: 공식 task body의 동등한 탭(예: Storage 또는 Attached Block Volumes)을 찾는다. 필드가 문서와 다르면 값·스크린샷에서 secrets를 가린 뒤 관리자 또는 Oracle Support에 확인하고 제출을 중단한다. 이 문서는 현재 tenancy의 live Console 화면을 검사한 결과가 아니다.

### 이 주제의 실패 지점

DNS hostnames 체크가 없으면 VCN DNS label 사용 여부를 확인한다. AD field가 채워졌으면 Regional 대신 AD-specific을 선택한 것이므로 제출 전 수정한다. 잘못 생성했으면 live resource 변경 조건을 읽고 owner 승인 없이 삭제하지 않는다.

## 리소스정리

owner22 및 후속 private subnet 소비자의 VNIC/서비스를 먼저 제거한다. private-app의 정확한 OCID에 남은 참조가 없는지 읽은 뒤 subnet → 참조가 없는 sl-private/rt-private 순서로 Delete하고 get/list를 확인한다. default DHCP/RT/SL, public-admin/IGW/nsg-admin, 공유 VCN/Compartment는 이 owner가 삭제하지 않는다. 승인 소비자와 expiry가 남으면 그때까지 보존한다.


## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 아래 본문은 각 생성 필드·설정·검증·정리의 기술 근거이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [30] [Networking Overview — Subnets](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/overview.htm)
  - 확인 범위: SUBNETS.

### 추가 실행 절차 근거 (본문 확인)

- [subnet](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/create_subnet.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [route](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/managingroutetables.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [security](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/securitylists.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [cloudshell](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/cloudshellgettingstarted.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.

- [추가 세부 task: slcreate](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/creating-securitylist.htm) — 실제 본문을 확인한 필드·API/권한·삭제 단계 근거.

- [추가 세부 task: routecreate](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/create-routetable.htm) — 실제 본문을 확인한 필드·API/권한·삭제 단계 근거.
