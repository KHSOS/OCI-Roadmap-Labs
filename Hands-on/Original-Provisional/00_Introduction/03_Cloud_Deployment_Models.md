> **기존 실습 초안 — 미검증**: 원래 93개 문서의 절차를 보존한 자료입니다. 새 튜토리얼의 품질 승인이나 OCI 실행 검증을 뜻하지 않습니다.

# Cloud Deployment Models

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../../../Docs/README.md)를 확인하세요.

> 실행 절차 작성본 / OCI 미실행. Oracle 공식 본문 근거와 교육용 선택을 구분한다. secrets·실제 고객값·diagram 없음.

## 학습목표

- owner22의 private-vm 실제 VNIC를 조회하여 OCI public Region/private-app 배치와 public-ip=null을 기록하고 새 ProxyJump 세션을 연다.
- 생성 필드와 실제 배치를 대응표에 기록하고 Private Cloud/VPN은 NOT_SELECTED로 남긴 뒤 private-vm 사용을 creator에게 반환한다.

위 항목은 사용자 실행 후 확인할 완료 조건이며 현재 관측 결과가 아니다. 개념은 아래 해당 생성 필드·configuration·test 단계에서 함께 적용한다.

## 대응 Roadmap 항목

- 항목명: **Cloud Deployment Models**
- Roadmap 경로: **Introduction → Cloud Deployment Models**
- Node ID: `sequence-0-3`
- 상위 항목: **Introduction** (`sequence-0-0`)
- 문서 범위: 이 항목의 생성·선택·기능검증·정리. 실습 미실행; 아래 기대 패턴은 실제 출력 아님.
- 번호는 navigation 식별자이며 실행 순서는 사전준비의 단계별 의존성에 따른다.

## 사전준비

### 기준 계약과 실제 단계 선행조건

[공통 baseline-contract.json](../../../baseline-contract.json)의 Oracle Linux 9 / opc / CIDR / 비용 상한을 따른다. 문서 번호는 실행 순서가 아니다. **준비 station과 생성 이후 ready를 분리**한다. 아래 단계 외의 기능완료를 소급하여 요구하지 않는다.

- [22 Private Subnets — 단계 링크](../../Networking/Original-Provisional/03_Virtual_Cloud_Network_VCN/03_Private_Subnets.md#baseline-22-ready): **private-vm RUNNING/ProxyJump와 실제 VNIC 출력**. 실제 출력/소비자/expiry를 전달하고 해당 owner가 이 lab 종료까지 보존한다.

생성 전 조직 승인(identity/권한/예산/expiry/서비스 지원/limit/capacity)을 확인한다. 실제 OCID/IP는 아직 생성하지 않았으면 비워 두고 해당 Create/readback 뒤 채운다. 동일 creation-event-id/OCID는 한 번의 제출만 뜻한다. `INSTANCE_A_ID`는 owner10 vm-a의 OCID, `INSTANCE_B_ID`는 owner22 private-vm의 OCID이며 owner07 vm-b와 다르다. 실습에 없는 자원을 추가하지 않는다. 실제 checkpoint는 사용자 실행 전 비워 둔다.

### 입력값과 실행 위치

Console에서 선택한 값만 사용한다. `<...>`는 예시 OCID나 비밀값이 아니라 **사용자가 대체할 입력 자리**다. 대체하지 않은 채 명령을 실행하지 않는다. Console → 우측 상단 Cloud Shell/Code Editor → Cloud Shell을 열면 OCI CLI가 사전 인증된다. Region 변경 뒤에는 Shell을 다시 열거나 명령의 `--region`을 바꾼다. SSH는 Cloud Shell이 아니라 본인 Linux/WSL/macOS 터미널에서 수행한다. 공인 IP/32도 그 터미널의 출구 주소여야 한다.

```bash
# 아래는 입력/출력의 대입 형식이다. 표의 단계 전에 생성 후 ID/IP export 행을 실행하지 않는다.
 # 생성 후 검증용 ID/IP는 해당 creator의 실제 응답에서 넣는다.
export REGION='<actual_REGION>'
export INSTANCE_B_ID='<actual_INSTANCE_B_ID>'
export PRIVATE_SUBNET_ID='<actual_PRIVATE_SUBNET_ID>'
export INSTANCE_ID="$INSTANCE_B_ID"
export SUBNET_ID="$PRIVATE_SUBNET_ID"
oci --version
```

이 변수 중 해당 절차에서 사용하는 값만 채운다. CLI 조회의 `data`를 증거 파일에 저장하고 실제 시간·작업 OCID·선택값을 기록한다. API key/private key/passphrase/인증 토큰은 기록하지 않는다. Cloud Shell 설정파일이나 세션 토큰을 출력하지 않는다. `oci --version`을 실행하는 주제의 기대 패턴은 버전 문자열뿐이다. namespace는 Object Storage 주제에서 별도 조회하며 다른 주제에는 요구하지 않는다. [Cloud Shell 공식 본문](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/cloudshellgettingstarted.htm).

### Workshop 입력 계약 — 단계와 host

| 입력/출력 | 값 또는 실제 획득 경로 | 사용할 시점 | 실행 위치 | 선택 이유 |
|---|---|---|---|---|
| Console/설정 필드 | owner22 launch 선택: public REGION/shared-vcn/private-app/public IP 없음; Private Cloud/VPN=NOT_SELECTED | 생성 전 선택 → 생성 뒤 readback | 본문 Console/로컬 지정 | 이 topic의 최소 생성 범위 |
| `INSTANCE_B_ID` | owner22 실제 private-vm 생성 id | 선행 완료/생성 후 | Cloud Shell | private B 조회; vm-b와 다름 |
| `INSTANCE_ID` | owner22 private-vm Create/readback id | 생성 후/선행 완료 | Cloud Shell | 현재 문서 대상 OCID에 명시 대입 |
| `LAB_COMPARTMENT_ID` | owner00 Create 응답 OCID; topic00에서는 생성 뒤 | 선행 완료/생성 후 | Console/Cloud Shell | 정책/자원 scope |
| `PRIVATE_SUBNET_ID` | owner21 subnet Create id | 선행 완료/생성 후 | Console/Cloud Shell | private-app 선택 |
| `REGION` | 승인된 기존 subscribed Region identifier | 생성 전 | Console/Cloud Shell | API endpoint/데이터 위치 |
| `ROUTE_TABLE_ID` | 대상 subnet readback data.route-table-id | 생성 후 | Cloud Shell | 실제 연결 RT 조회 |
| `SECURITY_LIST_ID` | 대상 subnet readback의 유일 security-list-ids 원소 | 생성 후 | Cloud Shell | 빈 SL/허용 합집합 조회 |
| `SUBNET_ID` | PRIVATE_SUBNET_ID | 생성 후/선행 완료 | Cloud Shell | 대상 subnet alias; 오래된 shell 값 재사용 금지 |
| `VNIC_ID` | 해당 INSTANCE_ID의 vnic-attachments.json data[].vnic-id; primary VNIC 확인 | 생성 후 | Cloud Shell | 다른 VM VNIC 혼동 방지 |

생성 전에는 행의 승인값/선행 완료값만 지정하고, 생성 후 ID/IP는 해당 owner 응답으로 채운다. Cloud Shell과 로컬 SSH의 환경변수는 공유되지 않는다. 로컬에는 KEY와 접속 주소만 다시 지정하고 guest disk 명령은 로그인한 guest에서만 실행한다. `<...>`가 남으면 해당 명령은 실행하지 않는다.

## 권한

### 실제 task/API별 공식 권한 본문 대조

아래는 이 문서 생성·조회·변경·정리 API의 공식 표 본문을 읽은 인용이다. 위 family 예제는 편의상 넓은 compartment-scope grant이며 **최소권한 정책이라고 부르지 않는다**. verb/dependency 행 인용은 단일 API의 완전한 최소권한 도출이 아니다. 조직 Identity Domain Console user/group 작업은 해당 Domain 관리자 역할도 필요하며 legacy IAM user API 권한만으로 대체하지 않는다.

- `GetInstance`: INSTANCE_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `GetRouteTable`: ROUTE_TABLE_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `GetSecurityList`: SECURITY_LIST_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `GetSubnet`: SUBNET_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `GetVnic`: VNIC_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `ListVnicAttachments`: VNIC_ATTACHMENT_READ and INSTANCE_INSPECT — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).

**Boot attachment 조회 범위:** 이 문서는 미확인 `ListBootVolumeAttachments` 최소권한을 요구하는 CLI를 실행하지 않는다. 생성 owner가 승인된 Compute Console → 정확한 instance OCID → Storage → Boot volume에서 연결된 boot OCID/AD와 instance를 읽어 전달한다. boot를 검증하는 주제만 위 GetBootVolume 권한을 승인받아 그 OCID를 조회한다. Console attachment를 볼 권한이 없으면 creator가 exact instance/boot 관계를 제공하기 전 중단한다. 권한 표에 없는 operation을 최소권한 검증 완료로 표시하지 않는다.


이번 consumer는 선행 creator의 리소스를 다시 생성·종료하지 않는다. 위 Get/List operation의 읽기 권한만 별도 승인한다. 공동 launch/VCN 생성 station에 참여하는 경우에만 해당 [creator 사전준비](#사전준비)의 Create/attachment 권한을 관리자와 확인한다. readback 때문에 `manage instance-family`, `manage volume-family`, `manage virtual-network-family` 세 가족을 일괄 요청하지 않는다. OS 조회는 `opc`/필요한 sudo, 로컬 파일 작성은 OCI IAM과 별개다.

## 비용및주의사항

- 새 자원은 **0개**, 기존 owner22 private-vm에 새 ProxyJump 세션 **1회**다. A/B VM·boot가 유지되는 시간과 관련 요청/전송 비용은 기존 creator 승인 기한 안에서만 사용한다. Private Cloud/CPE/VPN/DRG는 NOT_SELECTED이며 추가 구독·추가 VM을 만들지 않는다. 세션 종료 즉시 B lease 반환; 연장 필요하면 owner22 예산/종료시각을 다시 승인한다.
- 생성/사용 당일 공식 단가와 Billing & Cost Management → Cost Analysis의 실제 Compartment/서비스 필터를 확인하되 청구 반영 지연을 기록한다. 예산 경보는 과금 차단이 아니다. 고객 데이터/secret/운영 자원을 사용하지 않는다.
- 이 문서는 Oracle 기술 본문 기반 교육 절차다. OCI/guest/IAM allow·deny 실측은 없으며 기대 결과를 실행 결과로 기록하지 않는다. 승인되지 않은 선택 확장은 실행하지 않는다.


## 실습구성

### 이번 Workshop에서 만드는 것과 진행 순서

Oracle public Region의 private-vm 배치를 해석한다. Private subnet을 Private Cloud로, SSH를 Hybrid VPN으로 표시하지 않는다.

1. **진입/선택:** owner22의 private-vm launch 참여. 사전준비 링크의 지정 단계/실제 출력이 없으면 제출하지 않는다.
2. **실제 작업:** private flag/VNIC 확인 → 새 ProxyJump 로그인. 아래 Console 필드/명령을 해당 host에서 순서대로 실행한다. 같은 자원은 이름이 아니라 OCID/event로 재사용한다.
3. **완료/복구/반환:** Public/Private/Hybrid 구현 여부 표. 결과검증 실패는 문제해결로 이동하고 동일 Create를 무작정 재제출하지 않는다.

**전달/정리 범위:** private-vm OCID는 owner22; VPN/DRG/CPE 추가 없음. 아래 정리 단계는 이 소유권을 넘어서 실행하지 않는다.


- **이번 생성/재사용/출력:** 공동 신규 private-vm event 1개 사용. public/private/hybrid 모델을 실제 OCI public placement 선택과 연결. 별도 Private Cloud/VPN 생성 없음.
- **생성 목표:** 저비용 public Cloud 실제 생성 cohort(공유 VCN/private subnet VM)로 기본 목표 달성; Public/Private/Hybrid 구분을 이 선택과 함께 학습한다. 실제 CPE/VPN·Private Cloud 제공환경 구축은 피할 수 없는 추가 환경·비용이므로 명시적 선택 트랙이며 승인 전 미실행. Private Subnet을 Private Cloud로 잘못 대체하지 않는다.
- **완료 증거:** 생성 전 옵션 선택 + 실제 제출 대상 OCID + API readback + 해당 기능검증 + dependency 순서 정리. 실제 OCI 실행 checkpoint는 비워 둔다.
- **retention:** 예약 소비자와 expiry가 있는 resource만 보존. idle 유료 resource를 다음 주제 때문에 무기한 유지하지 않는다.
- **계약 보정:** 기존 creation-first 계약의 문서만 존재하는 선행(24/25/26/29 등)과 bootstrap 순환은 실제 절차를 포함한 00/19/23 단계 A로 대체했다. 변경된 typed dependency는 authoring ledger에 기록한다.


## 실습절차

### 명령 실행 gate — 실패하면 다음 생성으로 진행하지 않는다

1. 아래 Bash는 **전용 Bash 세션**에서만 사용한다. Console/Cloud Shell 조회, 본인 로컬 SSH, 로그인한 Oracle Linux guest를 섞지 않는다. `exit 1`은 이 전용 세션을 닫는 안전 중단이다. `<...>`·빈 입력·타 owner OCID가 남으면 실행하지 않는다.
2. 성공을 요구하는 파일 작성/조회/설치/attach 작업은 실패 시 중단한다. 실패한 API 뒤 기존 JSON이나 기존 다운로드 파일을 성공 근거로 읽지 않는다. 기대 deny는 `if command; then ...; else RC=$?; ...; fi`로 별도 기록하고, 실제 timeout/IAM 오류/로컬 파일 오류를 구별한다.
3. 첫 변경 전에 정확한 OCID·owner·현재 설정과 승인 expiry를 저장한다. 공유 자원 변경 중 다른 소비자가 시작하거나 원복값을 확인할 수 없으면 변경을 중단한다. 삭제 성공은 API 수락이 아니라 같은 identity/Region의 정확한 대상 재조회로 판정한다. `NotAuthorizedOrNotFound` 단독은 삭제 증거가 아니다.


### 개별 완주 보정 — 생성·검증·소유권

이 주제 INSTANCE_ID=INSTANCE_B_ID, SUBNET_ID=PRIVATE_SUBNET_ID로 지정한다. Private Cloud/VPN은 미구축이며 public cloud 배치만 검증한다. 공유private-vm은 creator22가 후속 소비자 확인 후 정리한다.

1. 사전준비에서 owner22가 실제 생성한 private-vm의 launch 선택/OCID를 읽는다. 신규 VM을 만들지 않고 Oracle public Region/shared-vcn/private-app의 같은 compatible 작은 VM에서 공인 IP 없음·ProxyJump 새 SSH를 검증한다. 이 생성 event는 **public cloud 안의 private subnet 배치**이다. private cloud 제공환경을 만든 것이 아니다.
2. Console/API에서 Region/Compartment/VCN/subnet/VM/IP를 기록한다. local client(사용자 장소)와 Oracle public Region(실행 장소)을 구분한다. local SSH 접속만으로 hybrid network가 구축됐다고 쓰지 않는다.
3. [Oracle What Is Cloud Computing — Public/Private/Hybrid 정의 본문](https://www.oracle.com/cloud/what-is-cloud-computing/)과 [Creating a Subnet — Private/Public access 선택](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/create_subnet.htm)을 읽고 다음 실제 결과표를 작성한다: Public Cloud=이번 OCI Region deployment(실제 생성), Private Subnet=이번 network access configuration(실제 생성), Private Cloud=dedicated cloud environment(본 lab 미구축), Hybrid=private/public environment 연결(본 lab 미구축).
4. public/admin과 private/app를 분리한 이유를 각 생성 필드(public IP prohibition/route/NSG)에 연결한다. 이 표는 생성 결과의 해석이며 문서만 보는 대체 실습이 아니다.
5. private cloud/hybrid 실행 확장을 하려면 승인된 private environment/CPE 주소, vendor/model, 양쪽 non-overlapping CIDR, static/BGP 계획, 비용·권한·rollback이 필요하다. 그 입력이 제공되지 않은 본 최소 track에서는 VPN/DRG/IPSec를 만들지 않는다. **가짜 CPE IP를 넣거나 public client를 on-prem CPE로 표시하지 않는다.** 기존 계약의 hybrid 선택 확장은 이 staged batch에서 NOT_SELECTED이며 해당 절차의 제품별 CPE 구성은 검증하지 않았다. 최소 primary 생성 목표는 실제 public-cloud private-vm 배치다.

### Create 이후 값 전달 — 이름으로 추정하지 않는다

Cloud Shell의 INSTANCE_ID는 해당 creator Create 응답 id로 지정한다. Console → Compute → Instances → 정확한 instance → Networking/Attached VNICs에서 primary VNIC OCID를 기록하고 아래 GetVnic의 `is-primary=true`, `subnet-id`, `private-ip`, `public-ip`를 대조한다. Storage/Boot volume의 연결 boot OCID·AD도 creator가 기록한다. Cloud Shell과 로컬 터미널의 변수는 공유되지 않으므로 SSH 전에 KEY와 **그 instance의** 실제 접속 주소를 로컬에 다시 지정한다. 이전 extra VM 주소를 재사용하지 않는다. 모든 JSON 조회는 실패하면 즉시 중단하고 기존 파일을 읽지 않는다.

### 새 ProxyJump 세션 — 본인 로컬 Linux/WSL

[owner22 전용 config 작성/검증](../../Networking/Original-Provisional/03_Virtual_Cloud_Network_VCN/03_Private_Subnets.md#baseline-22-ready)의 두 Host 블록을 실제 A public/B private 주소·local KEY로 준비한 뒤 아래 **새 연결**을 연다. 이전 성공 증거만 복사하지 않는다. config가 없거나 owner22가 B를 반환했다면 중단하고 creator에게 복구를 요청한다.

```bash
# owner22의 serial-console ED25519 인계와 owner10 guest trust를 먼저 완료한다.
test -f "$HOME/.ssh/oci-roadmap-jump.conf" && test ! -L "$HOME/.ssh/oci-roadmap-jump.conf" || exit 1
ssh -G -F "$HOME/.ssh/oci-roadmap-jump.conf" roadmap-admin > consumer03-admin-config.txt || exit 1
ssh -G -F "$HOME/.ssh/oci-roadmap-jump.conf" roadmap-private > consumer03-private-config.txt || exit 1
# 두 파일을 아래 신뢰 station의 실제 OCID/address/key와 대조한 뒤에만 연결한다.
timeout --foreground 60s ssh -F "$HOME/.ssh/oci-roadmap-jump.conf" -o BatchMode=yes -o IdentitiesOnly=yes -o HostKeyAlgorithms=ssh-ed25519 -o StrictHostKeyChecking=yes -o ConnectTimeout=10 -o ConnectionAttempts=1 -o ControlMaster=no -o ControlPath=none roadmap-private 'test "$(id -un)" = opc && hostname' > consumer03-ssh.txt || exit 1
date -u +%FT%TZ > consumer03-session-ended.txt || exit 1
```
연결 전 [owner10 A trust bootstrap](../02_Compute/00_Compute.md#guest-trust-bootstrap)과 [owner22 B 신뢰·config 인계](../../Networking/Original-Provisional/03_Virtual_Cloud_Network_VCN/03_Private_Subnets.md#baseline-22-ready)의 **serial Console ED25519 획득 절차**를 읽는다. 인계 record에는 A/B의 서로 다른 actual OCID, A public/B private 주소, 각각의 ED25519 SHA256 fingerprint, capture UTC, local KEY 절대경로가 있어야 한다. 현재 creator API VNIC/OCID 결과와 주소를 대조한다. `ssh -G` 두 결과의 hostname/user=opc/identityfile, private의 proxyjump=roadmap-admin, 각 Host의 stricthostkeychecking=yes 및 hostkeyalgorithms=ssh-ed25519를 확인하고 known_hosts의 각 주소 ED25519 fingerprint를 `ssh-keygen -F '<actual_address>'`와 `ssh-keygen -lf '<creator가 저장한_public_host_key_file>'`로 인계와 비교한다. 한 항목이라도 없거나 바뀌면 **BLOCKED_TRUST**, creator에게 재확인을 요청하며 unknown-host prompt/TOFU/keyscan을 수락하지 않는다. config의 admin Host에도 BatchMode yes를 확인한다. 60초 전체 상한/exit 124는 성공이 아니며 명령 오류에는 세션 종료 증거를 성공처럼 쓰지 않는다. private key는 A에 복사하지 않는다.

## 결과검증


### baseline-03-ready


```bash
oci network subnet get --subnet-id "$SUBNET_ID" --region "$REGION" > subnet-readback.json || exit 1
export ROUTE_TABLE_ID='<subnet_data_route-table-id>'
export SECURITY_LIST_ID='<subnet_data_security-list-ids에서_유일한_OCID>'
oci network route-table get --rt-id "$ROUTE_TABLE_ID" --region "$REGION" > route-readback.json || exit 1
oci network security-list get --security-list-id "$SECURITY_LIST_ID" --region "$REGION" > security-readback.json || exit 1
```
기대 패턴: `lifecycle-state=AVAILABLE`, `availability-domain=null`(Regional), `vcn-id`/`cidr-block`/`route-table-id`/`security-list-ids`/`dhcp-options-id`가 registry와 일치. 이번 private-app은 `prohibit-public-ip-on-vnic=true`, initial rt-private의 `route-rules=[]`를 요구한다. NAT/SGW 소비 중에는 해당 승인 route가 있으므로 initial stage와 시험 중 readback을 구분한다.
```bash
oci compute instance get --instance-id "$INSTANCE_ID" --region "$REGION" > instance-readback.json || exit 1
oci compute vnic-attachment list --compartment-id "$LAB_COMPARTMENT_ID" --instance-id "$INSTANCE_ID" --region "$REGION" > vnic-attachments.json || exit 1
# Cloud Shell: 현재 instance에 붙은 ATTACHED VNIC만 각각 조회한다.
export VNIC_ID='<현재_INSTANCE_ID에_ATTACHED이며_GetVnic_is-primary=true인_OCID>'
case "$VNIC_ID" in ocid1.vnic.*) ;; *) printf '%s\n' 'STOP: actual primary VNIC OCID required'; exit 1;; esac
oci network vnic get --vnic-id "$VNIC_ID" --region "$REGION" > vnic-readback.json || exit 1
```
기대 패턴: instance `lifecycle-state=RUNNING`, `availability-domain`/`fault-domain`/`shape`가 제출값과 같음, VNIC `subnet-id`가 선택값과 같음, public VM은 `public-ip`가 문자열이고 private VM은 null이다. creator Console Storage의 boot 연결이 대상 instance와 boot OCID를 연결한다. 파일을 실제로 읽어 조건을 대조한다.private-vm 기대: 선택한 public REGION endpoint에서 RUNNING, private-app membership, public-ip=null, 실제 ProxyJump SSH exit 0. responsibility/deployment 표에 `Private Cloud deployment=NOT_SELECTED`, `Hybrid VPN connectivity=NOT_SELECTED`를 명시한다. private IP와 private cloud의 차이를 올바르게 설명해야 한다.

**증거 기록:** 실제 명령·UTC 시간·exit code·가려진 OCID·옵션 readback·검증 조건을 남긴다. 현재 이 문서의 실행상태는 `NOT_EXECUTED`다. output/creation-event-id/result는 실행 전 비워 둔다. 오류/NOT_SELECTED를 PASS로 바꾸지 않는다.

### 이 시나리오의 functional acceptance

| Positive | Negative / 미완료 판정 | Recovery / 반환 |
|---|---|---|
| private VNIC/ProxyJump 성공 | Private Cloud/Hybrid VPN 구현란 NOT_SELECTED | false claim 없이 public-cloud placement 표와 creator output을 다시 대조 |

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

private-vm에 public IP가 생겼으면 private subnet OCID/금지 flag/target VNIC를 먼저 확인한다. 이를 private-cloud 성공으로 표시하지 않는다. 실제 CPE/vendor가 없는 Hybrid 선택 확장은 가짜 tunnel 설정으로 채우지 않는다.

## 리소스정리

private-vm은 owner22 소유이며 여기서 Terminate/Stop하지 않는다. 아래 **consumer03 반환 record**를 creator registry의 새 record에 실제값으로 기록한다: `consumer=03`, `REGION`, `INSTANCE_B_ID`(private-vm), A OCID, B private 주소, `session_end_utc`(consumer03-session-ended.txt), SSH exit 0 또는 실패 exit, config/SSH/VNIC 증거 경로, `outstanding_changes=none`, `lease=RELEASED` 또는 미완료 사유와 승인 retention deadline, owner22 acknowledgment UTC/alias. 실패 세션도 종료 여부와 미완료를 구분하여 반환한다. owner22 acknowledgment 전에는 lease 반환 완료가 아니다. [owner22 리소스정리](../../Networking/Original-Provisional/03_Virtual_Cloud_Network_VCN/03_Private_Subnets.md#리소스정리)로 넘기고 공유 config/KEY/known_hosts는 삭제하지 않는다. consumer03 증거는 승인 보존 경로에 복사·hash 확인한 뒤 자신이 만든 exact 4개 파일만 제거한다; 타 owner 파일/glob 삭제 금지.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 아래 본문은 각 생성 필드·설정·검증·정리의 기술 근거이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [45] [What Is Cloud Computing? — Cloud Deployment Models](https://www.oracle.com/cloud/what-is-cloud-computing/)
  - 확인 범위: Public Cloud.

### 추가 실행 절차 근거 (본문 확인)

- [subnet](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/create_subnet.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [launch](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/launchinginstance.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [regions](https://docs.oracle.com/en-us/iaas/Content/General/Concepts/regions.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [cloudshell](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/cloudshellgettingstarted.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
