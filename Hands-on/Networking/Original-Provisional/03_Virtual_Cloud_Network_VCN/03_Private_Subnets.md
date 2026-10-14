> **기존 실습 초안 — 미검증**: 원래 93개 문서의 절차를 보존한 자료입니다. 새 튜토리얼의 품질 승인이나 OCI 실행 검증을 뜻하지 않습니다.

# Private Subnets

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../../../../Docs/README.md)를 확인하세요.

> 실행 절차 작성본 / OCI 미실행. Oracle 공식 본문 근거와 교육용 선택을 구분한다. secrets·실제 고객값·diagram 없음.

## 학습목표

- private-app에 공인 IP 없는 private-vm과 nsg-private-admin을 생성하고 A→B 신규 연결의 A egress22/B ingress22 실제 rule ID를 확인한다.
- 로컬 key만 사용하는 ProxyJump 새 SSH와 public-ip=null을 검증하고 소비자 종료 후 B/boot→자기 jump rule→private NSG만 정리한다.

## 대응 Roadmap 항목

- 항목명: **Private Subnets**
- Roadmap 경로: **Virtual Cloud Network (VCN) → Private Subnets**
- Node ID: `sequence-3-3`
- 상위 항목: **Virtual Cloud Network (VCN)** (`sequence-3-0`)
- 문서 범위: private-app에 공인 IP 없는 private-vm과 nsg-private-admin을 생성하고 A→B 신규 연결의 A egress22/B ingress22 실제 rule ID를 확인한다. 로컬 key만 사용하는 ProxyJump 새 SSH와 public-ip=null을 검증하고 소비자 종료 후 B/boot→자기 jump rule→private NSG만 정리한다. 실제 OCI 실행 결과가 아니라 이 절차의 완료 조건이다.
- 번호는 navigation 식별자이며 실행 순서는 사전준비의 단계별 의존성에 따른다.

## 사전준비

### 기준 계약과 실제 단계 선행조건

[공통 baseline-contract.json](../../../../baseline-contract.json)의 Oracle Linux 9 / opc / CIDR / 비용 상한을 따른다. 문서 번호는 실행 순서가 아니다. **준비 station과 생성 이후 ready를 분리**한다. 아래 단계 외의 기능완료를 소급하여 요구하지 않는다.

- [21 Subnets — 단계 링크](02_Subnets.md#baseline-21-ready): **AVAILABLE private-app/빈 rt/sl**. 실제 출력/소비자/expiry를 전달하고 해당 owner가 이 lab 종료까지 보존한다.
- [10 Compute — 단계 링크](../../../Original-Provisional/02_Compute/00_Compute.md#baseline-10-ready): **vm-a 새 SSH/public+private IP**. 실제 출력/소비자/expiry를 전달하고 해당 owner가 이 lab 종료까지 보존한다.
- [15 SSH Keys — 단계 링크](../../../Original-Provisional/02_Compute/05_SSH_Keys.md#key-preparation): **Part A key local 보존**. 실제 출력/소비자/expiry를 전달하고 해당 owner가 이 lab 종료까지 보존한다.

생성 전 조직 승인(identity/권한/예산/expiry/서비스 지원/limit/capacity)을 확인한다. 실제 OCID/IP는 아직 생성하지 않았으면 비워 두고 해당 Create/readback 뒤 채운다. 동일 creation-event-id/OCID는 한 번의 제출만 뜻한다. `INSTANCE_A_ID`는 owner10 vm-a의 OCID, `INSTANCE_B_ID`는 owner22 private-vm의 OCID이며 owner07 vm-b와 다르다. 실습에 없는 자원을 추가하지 않는다. 실제 checkpoint는 사용자 실행 전 비워 둔다.

### 입력값과 실행 위치

Console에서 선택한 값만 사용한다. `<...>`는 예시 OCID나 비밀값이 아니라 **사용자가 대체할 입력 자리**다. 대체하지 않은 채 명령을 실행하지 않는다. Console → 우측 상단 Cloud Shell/Code Editor → Cloud Shell을 열면 OCI CLI가 사전 인증된다. Region 변경 뒤에는 Shell을 다시 열거나 명령의 `--region`을 바꾼다. SSH는 Cloud Shell이 아니라 본인 Linux/WSL/macOS 터미널에서 수행한다. 공인 IP/32도 그 터미널의 출구 주소여야 한다.

```bash
# 아래는 입력/출력의 대입 형식이다. 표의 단계 전에 생성 후 ID/IP export 행을 실행하지 않는다.
 # 생성 후 검증용 ID/IP는 해당 creator의 실제 응답에서 넣는다.
export REGION='<actual_REGION>'
export INSTANCE_B_ID='<actual_INSTANCE_B_ID>'
export KEY='<actual_KEY>'
export A_PUBLIC_IP='<actual_A_PUBLIC_IP>'
export B_PRIVATE_IP='<actual_B_PRIVATE_IP>'
export PRIVATE_SUBNET_ID='<actual_PRIVATE_SUBNET_ID>'
export INSTANCE_ID="$INSTANCE_B_ID"
export SUBNET_ID="$PRIVATE_SUBNET_ID"
oci --version
```

이 변수 중 해당 절차에서 사용하는 값만 채운다. CLI 조회의 `data`를 증거 파일에 저장하고 실제 시간·작업 OCID·선택값을 기록한다. API key/private key/passphrase/인증 토큰은 기록하지 않는다. Cloud Shell 설정파일이나 세션 토큰을 출력하지 않는다. `oci --version`을 실행하는 주제의 기대 패턴은 버전 문자열뿐이다. namespace는 Object Storage 주제에서 별도 조회하며 다른 주제에는 요구하지 않는다. [Cloud Shell 공식 본문](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/cloudshellgettingstarted.htm).

### Workshop 입력 계약 — 단계와 host

| 입력/출력 | 값 또는 실제 획득 경로 | 사용할 시점 | 실행 위치 | 선택 이유 |
|---|---|---|---|---|
| Console/설정 필드 | Name=private-vm; OL9 compatible small shape; On-demand; private-app/no public IPv4; nsg-private-admin source A_PRIVATE_IP/32 TCP22; A egress B_PRIVATE_IP/32 TCP22 | 생성 전 선택 → 생성 뒤 readback | 본문 Console/로컬 지정 | 이 topic의 최소 생성 범위 |
| `PRIVATE_SUBNET_ID` | owner21 subnet Create id | 선행 완료/생성 후 | Console/Cloud Shell | private-app 선택 |
| `KEY` | owner15 Part A 전용 private key 절대경로; 내용 전송 금지 | 준비 완료 | 로컬 SSH만 | 해당 공개키와 대응 |
| `B_PRIVATE_IP` | owner22 private-vm VNIC private-ip | 생성 후 | 로컬 SSH/Console | jump target 및 A egress /32 |
| `A_PUBLIC_IP` | owner10 vm-a VNIC public-ip | 선행 완료/생성 후 | 로컬 SSH | 관리 jump 주소 |
| `INSTANCE_B_ID` | owner22 실제 private-vm 생성 id | 선행 완료/생성 후 | Cloud Shell | private B 조회; vm-b와 다름 |
| `INSTANCE_ID` | owner22 private-vm Create/readback id | 생성 후/선행 완료 | Cloud Shell | 현재 문서 대상 OCID에 명시 대입 |
| `LAB_COMPARTMENT_ID` | owner00 Create 응답 OCID; topic00에서는 생성 뒤 | 선행 완료/생성 후 | Console/Cloud Shell | 정책/자원 scope |
| `REGION` | 승인된 기존 subscribed Region identifier | 생성 전 | Console/Cloud Shell | API endpoint/데이터 위치 |
| `ROUTE_TABLE_ID` | 대상 subnet readback data.route-table-id | 생성 후 | Cloud Shell | 실제 연결 RT 조회 |
| `SECURITY_LIST_ID` | 대상 subnet readback의 유일 security-list-ids 원소 | 생성 후 | Cloud Shell | 빈 SL/허용 합집합 조회 |
| `SUBNET_ID` | PRIVATE_SUBNET_ID | 생성 후/선행 완료 | Cloud Shell | 대상 subnet alias; 오래된 shell 값 재사용 금지 |
| `VNIC_ID` | 해당 INSTANCE_ID의 vnic-attachments.json data[].vnic-id; primary VNIC 확인 | 생성 후 | Cloud Shell | 다른 VM VNIC 혼동 방지 |

생성 전에는 행의 승인값/선행 완료값만 지정하고, 생성 후 ID/IP는 해당 owner 응답으로 채운다. Cloud Shell과 로컬 SSH의 환경변수는 공유되지 않는다. 로컬에는 KEY와 접속 주소만 다시 지정하고 guest disk 명령은 로그인한 guest에서만 실행한다. `<...>`가 남으면 해당 명령은 실행하지 않는다.

## 권한

private-vm 단일 launch/조회/terminate, nsg-private-admin Create/rule add/read/remove/Delete 및 A의 nsg-admin에 자기 B/32 TCP22 egress 추가/제거를 승인한다. 기존 subnet/VCN 사용 권한도 확인한다. guest key는 local에만 두며 VCN/gateway 생성 정책을 B launch 때문에 추가하지 않는다. 진단 console/history는 첫 Create 전 독립적으로 승인해야 한다.

### 실제 operation과 공식 권한

- `AddNetworkSecurityGroupSecurityRules`: NETWORK_SECURITY_GROUP _UPDATE_SECURITY_RULES and NETWORK_SECURITY_GROUP_INSPECT if writing a rule that specifies a network security group as the source (for ingress rules) or destination (for egress rules) — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `CreateNetworkSecurityGroup`: NETWORK_SECURITY_GROUP_CREATE and VCN_ATTACH — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `GetInstance`: INSTANCE_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `GetRouteTable`: ROUTE_TABLE_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `GetSecurityList`: SECURITY_LIST_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `GetSubnet`: SUBNET_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `GetVnic`: VNIC_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `LaunchInstance`: INSTANCE_CREATE and INSTANCE_IMAGE_READ and VNIC_CREATE and VNIC_ATTACH and SUBNET_ATTACH If putting the instance in a network security group during instance creation, also need NETWORK_SECURITY_GROUP _UPDATE_MEMBERS and VNIC_ASSOCIATE _NETWORK_SECURITY_GROUP If creating an instance in a compute cluster, also need COMPUTE_CLUSTER_LAUNCH_INSTANCE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `ListNetworkSecurityGroupSecurityRules`: NETWORK_SECURITY_GROUP _LIST_SECURITY_RULES — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `ListVnicAttachments`: VNIC_ATTACHMENT_READ and INSTANCE_INSPECT — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `RemoveNetworkSecurityGroupSecurityRules`: NETWORK_SECURITY_GROUP _UPDATE_SECURITY_RULES — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `TerminateInstance`: INSTANCE_DELETE and VNIC_DELETE and SUBNET_DETACH If volumes are attached, also need VOLUME_ATTACHMENT_DELETE and VOLUME_WRITE and INSTANCE_DETACH_VOLUME — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `DeleteNetworkSecurityGroup`: NETWORK_SECURITY_GROUP_DELETE and VCN_DETACH — [공식 API 권한 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).

위 permission 이름은 IAM policy resource-type이 아니다. 조직 관리자가 기존 group/compartment scope와 conditional dependency를 실제 API 표에 대조한다. 편의 family grant를 최소권한 정책으로 부르지 않으며 실습자는 policy 작성 권한을 자동 인수하지 않는다.

**Boot attachment 조회 범위:** 이 문서는 미확인 `ListBootVolumeAttachments` 최소권한을 요구하는 CLI를 실행하지 않는다. 생성 owner가 승인된 Compute Console → 정확한 instance OCID → Storage → Boot volume에서 연결된 boot OCID/AD와 instance를 읽어 전달한다. boot를 검증하는 owner13만 GetBootVolume 권한을 승인받아 그 OCID를 조회한다. Console attachment를 볼 권한이 없으면 creator가 exact instance/boot 관계를 제공하기 전 중단한다. 권한 표에 없는 operation을 최소권한 검증 완료로 표시하지 않는다.

최초 SSH에 필요한 exact-instance Console connection/history는 관리자에게 해당 Create/Get/Delete/Capture 작업을 별도 승인받는다. [Compute의 독립 guest 신뢰 경로](../../../Original-Provisional/02_Compute/00_Compute.md#guest-trust-bootstrap)의 BLOCKED_TRUST/BLOCKED_IAM gate를 보존한다. 승인 없이 public host fingerprint를 추정하거나 guest password를 만들지 않는다.

## 비용및주의사항

- 추가 비용은 public IP 없는 private-vm 1대와 default boot다. vm-a jump host와 기존 private-app을 재사용하며 vm-b/burstable/preemptible 등 다른 extra VM과 겹쳐 실행하지 않는다.
- B는 인터넷 default route 없이 SSH22만 허용한다. A→B egress22 추가 ID와 B ingress source A/32를 기록하고 private key는 A에 복사하지 않는다. 실제 app/service 소비자별 expiry/반환 후 B/boot→자기 jump rule→private NSG 순으로 정리한다.

실제 OCI 작업은 아직 수행하지 않았다. 기대값은 실행 시 대조할 조건이며 무료 사용이나 과금 강제 차단을 보장하지 않는다. 승인 expiry가 끝나면 새 시험을 중단하고 위 소유권 순서로 원복·반환한다.

## 실습구성

### 이번 Workshop에서 만드는 것과 진행 순서

private-app에 public IP 없는 private-vm 한 세트와 nsg-private-admin을 만들고 로컬 ProxyJump로 접속한다. jump egress TCP22가 필수다.

1. **진입/선택:** 21 subnet/10 jump/15 key 완료. 사전준비 링크의 지정 단계/실제 출력이 없으면 제출하지 않는다.
2. **실제 작업:** private NSG ingress A/32:22 → VM Create → B private IP → A NSG egress B/32:22. 아래 Console 필드/명령을 해당 host에서 순서대로 실행한다. 같은 자원은 이름이 아니라 OCID/event로 재사용한다.
3. **완료/복구/반환:** 새 ProxyJump. 결과검증 실패는 문제해결로 이동하고 동일 Create를 무작정 재제출하지 않는다.

**전달/정리 범위:** INSTANCE_B_ID/B_PRIVATE_IP/boot/NSG/추가 rule ID; owner22가 B/boot→자기 rule→private NSG만 정리. 아래 정리 단계는 이 소유권을 넘어서 실행하지 않는다.


- **이번 생성/재사용/출력:** 신규 nsg-private-admin/private-vm 세트. private-app은 owner21이 이미 생성한 exact OCID만 재사용. NAT/SGW/추가 bastion service 없음.
- **생성 목표:** owner21의 private-app Regional subnet을 재사용해 private-only VM만 실제 생성한다.통신검증은sharedadminVM의labprivateIP에서제한된SSH경로로수행;NAT/SGW를이lab에강제추가하지않음.
- **계약 보정:** 기존 creation-first 계약의 문서만 존재하는 선행(24/25/26/29 등)과 bootstrap 순환은 실제 절차를 포함한 00/19/23 단계 A로 대체했다. 변경된 typed dependency는 authoring ledger에 기록한다.

## 실습절차

### 작업 순서 체크포인트
creation 전 subnet/route/NSG 실제 값을 확인하고 Create 이후 VNIC/private-ip/jump를 확인한다. 이후 선택 consumer 목록을 등록한 다음에만 retention/cleanup을 결정한다. 단계 실패는 다음 생성으로 넘기지 않고 같은 target의 복구/반환으로 처리한다.

### 명령 실행 gate — 실패하면 다음 생성으로 진행하지 않는다

1. 아래 Bash는 **전용 Bash 세션**에서만 사용한다. Console/Cloud Shell 조회, 본인 로컬 SSH, 로그인한 Oracle Linux guest를 섞지 않는다. `exit 1`은 이 전용 세션을 닫는 안전 중단이다. `<...>`·빈 입력·타 owner OCID가 남으면 실행하지 않는다.
2. 성공을 요구하는 파일 작성/조회/설치/attach 작업은 실패 시 중단한다. 실패한 API 뒤 기존 JSON이나 기존 다운로드 파일을 성공 근거로 읽지 않는다. 기대 deny는 `if command; then ...; else RC=$?; ...; fi`로 별도 기록하고, 실제 timeout/IAM 오류/로컬 파일 오류를 구별한다.
3. 첫 변경 전에 정확한 OCID·owner·현재 설정과 승인 expiry를 저장한다. 공유 자원 변경 중 다른 소비자가 시작하거나 원복값을 확인할 수 없으면 변경을 중단한다. 삭제 성공은 API 수락이 아니라 같은 identity/Region의 정확한 대상 재조회로 판정한다. `NotAuthorizedOrNotFound` 단독은 삭제 증거가 아니다.


**최대 두 VM 실행 gate:** vm-a 외 vm-b/burstable-vm/ip-lab-vm/preemptible-vm/private-vm는 같은 시간에 한 개만 실행한다. 이전 extra VM/boot 정리를 readback한 후 다음 extra를 생성한다. owner07 vm-b를 owner16이 쓰는 경우에는 새로운 VM을 만들지 않고 09 검증을 먼저 끝낸다.

### 개별 완주 보정 — 생성·검증·소유권

새sshconfig기존파일 guard를 둔다. INSTANCE_ID=INSTANCE_B_ID로 조회하며 guestopc를확인한다. 24~32/36/42/43~46예약소비자와expiry가있을때만유지하고 소비자는숨은probeVM을만들지 않는다.

1. Subnets 생성 station의 private-app/rt-private/sl-private를 한 번 생성했다는 증거를 확인한다. 없으면 Subnets의 실제 절차를 완료한다. public-admin에서 접속 가능한 vm-a가 있어야 jump 경로를 검증할 수 있다. public key Part A는 같은 로컬 KEY를 재사용하고 private key를 vm-a에 복사하지 않는다.
2. Networking → shared-vcn → Network Security Groups → Create NSG: `nsg-private-admin`, lab-root. Stateful Ingress TCP Source `<vm-a_private_IP>/32`, source port All, destination port 22 규칙 1개를 생성한다. Egress는 이 최소 SSH lab에서는 비워 두며 응답은 stateful tracking을 사용한다. sl-private ingress/egress가 비어 있는지 읽는다. broad allow가 있으면 좁은 NSG만으로 상쇄되지 않는다.
3. **Create 전 B 독립 trust 준비:** owner22는 인증된 exact-OCID Console connection/history 사용 승인을 먼저 받는다. 승인되지 않으면 BLOCKED_TRUST로 Create 전 중단한다. Advanced options → Management → Initialization script에 아래 B 전용 YAML을 raw text/file로 넣는다(base64 선처리 금지). 공개 host key fingerprint만 출력하며 guest password/private key는 출력하지 않는다. 원본 파일/hash를 owner22 registry에 보관하고 Review에서 실제 업로드 bytes를 대조한다. 이 출력은 저술 recipe이며 OCI에서 관측하지 않았다.
```yaml
#cloud-config
runcmd:
  - [sh, -c, 'printf "ROADMAP_B_ED25519_SHA256\n" > /dev/console && ssh-keygen -E sha256 -lf /etc/ssh/ssh_host_ed25519_key.pub >> /dev/console']
```
두 번째 작은 VM/boot의 예산을 승인하고 Compute → Create instance: Name private-vm, lab-root, 승인 AD/FD, Oracle Linux 9 platform image(Compute와 같은 architecture), 같은 작은 shape, On-demand, existing shared-vcn/private-app, **Do not assign public IPv4 address**, `nsg-private-admin`, 공개키, default boot. Create 제출한다. 실제 UI에서 public IP 선택이 금지되어야 하며 API의 private flag도 확인한다. [Private subnet semantics](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/create_subnet.htm) · [Networking and SSH keys](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/launchinginstance.htm).


**필수 jump egress 추가:** vm-a의 `nsg-admin`은 기본 HTTPS443만 허용하므로 그대로는 private-vm TCP22를 시작할 수 없다. private-vm의 실제 private IP를 얻은 뒤 Networking → shared-vcn → nsg-admin → Add Rules에서 Stateful **Egress**, Destination CIDR `<private-vm_private_IP>/32`, Protocol TCP, Source port All, Destination port **22** 규칙 1개를 제출한다. 새 rule ID와 원래 규칙을 기록한다. proxy 접속 전 `oci network nsg rules list --nsg-id '<nsg-admin_OCID>' --region "$REGION"`로 정확한 /32 TCP22 egress를 읽는다. private-vm ingress source는 vm-a private /32이며 두 방향의 새 연결 허용이 모두 필요하다. [NSG rule direction/union/stateful 공식 본문](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/networksecuritygroups.htm).

**첫 B SSH 전에 host identity 검증:** owner22는 인증된 OCI Console → Compute → exact `INSTANCE_B_ID` → Console connection → Launch Cloud Shell connection으로 serial 출력 경로를 연다. guest login/password는 필요하지 않으며 opc password를 만들거나 reset하지 않는다. 이어 같은 OCID → Console history → View current history → Save and close → 해당 capture Actions → View details에서 초기 boot의 `ROADMAP_B_ED25519_SHA256` 바로 뒤 ED25519 SHA256 행을 읽는다. 30초 간격 최대5분 또는 expiry 중 먼저 오는 시점까지만 확인한다. 초기 출력이 없거나 모호하면 BLOCKED_TRUST; 생성 후 metadata/reboot로 최초 증거를 꾸미지 않는다. 로컬 첫 ProxyJump의 **B 주소/ED25519 fingerprint**가 독립 확인값과 일치할 때만 yes를 입력한다. A fingerprint는 owner10 값이며 B에 복사하지 않는다. Console service RSA fingerprint와 guest ED25519를 구분한다. `ssh-keyscan`/TOFU/StrictHostKeyChecking=no는 근거가 아니다. capture는 connection 생성이 선행돼야 한다. owner22가 만든 진단 connection/history만 소유권/정리 권한 승인 후 Delete/readback하고 guest/boot는 보존한다. **권한 조건부 gate:** Create/Get/Delete instance console connection 및 Capture/Get/Delete console history의 실제 operation scope를 관리자가 공식 표와 조직 정책으로 승인하기 전 Create/진단 작업은 BLOCKED_IAM이다; 여기서 최소권한 검증 완료를 주장하지 않는다. [Serial Console](https://docs.oracle.com/en-us/iaas/Content/Compute/References/serialconsole.htm) · [Capture task](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/displayingconsole-capturing.htm) · [View task](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/displayingconsole-viewing.htm).

4. RUNNING 뒤 VNIC private IP와 public-ip=null을 기록한다. 기존 local client에서 jump 명령을 실행한다. `-J`의 jump key 처리 혼선을 피하려면 아래 전용 ssh config를 새 파일로 만든다. private key는 local에서만 사용한다.

```bash
# 로컬 Linux/WSL: 값 교체 후 전용 파일 작성; 기존 파일 덮어쓰기 금지
if test -e "$HOME/.ssh/oci-roadmap-jump.conf" || test -L "$HOME/.ssh/oci-roadmap-jump.conf"; then printf '%s\n' 'STOP: choose a new dedicated config path'; exit 1; fi
cat > "$HOME/.ssh/oci-roadmap-jump.conf" <<'CONF' || exit 1
Host *
  ControlMaster no
  ControlPath none
  ControlPersist no
  ConnectTimeout 10
  ConnectionAttempts 1
  ServerAliveInterval 10
  ServerAliveCountMax 2
Host roadmap-admin
  HostName <vm-a_public_IP>
  User opc
  IdentityFile <local_private_KEY_absolute_path>
  IdentitiesOnly yes
Host roadmap-private
  HostName <private-vm_private_IP>
  User opc
  IdentityFile <local_private_KEY_absolute_path>
  IdentitiesOnly yes
  ProxyJump roadmap-admin
CONF
chmod 600 "$HOME/.ssh/oci-roadmap-jump.conf" || exit 1
ssh -o ControlMaster=no -o ControlPath=none -o ControlPersist=no -o ConnectTimeout=10 -o ConnectionAttempts=1 -o ServerAliveInterval=10 -o ServerAliveCountMax=2 -F "$HOME/.ssh/oci-roadmap-jump.conf" roadmap-private 'hostname; id -un; ip -4 addr' || exit 1
```
전용 config의 `Host *`는 target과 ProxyJump admin 양쪽의 multiplex 재사용을 금지하고 connect 10초/1회 및 무응답 keepalive 10초×2회를 제한한다. CLI에서도 target 옵션을 명시한다. 호스트 fingerprint 확인/사용자 입력 대기까지 전체 wall-clock 제한으로 주장하지 않는다. 이 옵션은 authored OpenSSH client 안전 설정이며 OCI 실행 결과가 아니다.

5. guest internet outbound 성공을 요구하지 않는다. rt-private에는 IGW/NAT/SGW default route가 없다. Private subnet은 public IPv4 할당 금지라는 network 설정이지 Private Cloud가 아니다. IPv6 주소/route는 생성하지 않으며 이 IPv4 ProxyJump 실습의 완료 조건에 포함하지 않는다.
6. owner22 registry에24–32/36/42/43–46뿐 아니라 선택56 image source, 61 app-only/LB와62–65, 66–70 NLB, 71–75 WAF, 76–77 DNS app-only, 78 steering, 79 external monitor를 각 SELECTED/NOT_SELECTED·consumer owner·INSTANCE_B_ID/boot OCID·expiry·반환 상태로 등록한다. 실제 선택 consumer 전원의 복구/반환 또는 승인된 abort 확인 전 VM/boot를 삭제하지 않는다. 56 source STOPPED는 capture/pool 총수량 gate일 뿐 반환이 아니며 source restart/termination은 owner22만 승인 실행한다. expiry에는 신규 시험 중단→각 owner 복구/반환→owner22 정리 순서다. 앱-only consumer에게 LB 생성을 강제하지 않는다. 삭제했다면 owner22의 이 생성 절차를 재승인받아 완료하고 신규 OCID를 registry에 기록한다. 소비자24에서 숨은 대체 VM을 만들지 않는다.

### Create 이후 값 전달 — 이름으로 추정하지 않는다

Cloud Shell의 INSTANCE_ID는 해당 creator Create 응답 id로 지정한다. Console → Compute → Instances → 정확한 instance → Networking/Attached VNICs에서 primary VNIC OCID를 기록하고 아래 GetVnic의 `is-primary=true`, `subnet-id`, `private-ip`, `public-ip`를 대조한다. Storage/Boot volume의 연결 boot OCID·AD도 creator가 기록한다. Cloud Shell과 로컬 터미널의 변수는 공유되지 않으므로 SSH 전에 KEY와 **그 instance의** 실제 접속 주소를 로컬에 다시 지정한다. 이전 extra VM 주소를 재사용하지 않는다. 모든 JSON 조회는 실패하면 즉시 중단하고 기존 파일을 읽지 않는다.

## 결과검증


### baseline-22-ready


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
기대 패턴: instance `lifecycle-state=RUNNING`, `availability-domain`/`fault-domain`/`shape`가 제출값과 같음, VNIC `subnet-id`가 선택값과 같음, public VM은 `public-ip`가 문자열이고 private VM은 null이다. creator Console Storage의 boot 연결이 대상 instance와 boot OCID를 연결한다. 파일을 실제로 읽어 조건을 대조한다.
```bash
oci network nsg rules list --nsg-id '<nsg-private-admin_OCID>' --region "$REGION" > private-rules.json || exit 1
# 로컬에서 새 session
ssh -o ControlMaster=no -o ControlPath=none -o ControlPersist=no -o ConnectTimeout=10 -o ConnectionAttempts=1 -o ServerAliveInterval=10 -o ServerAliveCountMax=2 -F "$HOME/.ssh/oci-roadmap-jump.conf" roadmap-private 'test "$(id -un)" = opc && hostname' || exit 1
```
기대 패턴 prohibit-public-ip-on-vnic=true, VNIC public-ip=null/정확한 private-app subnet, private route-rules=[], NSG source=vm-a private /32/TCP22, 새 ProxyJump SSH exit 0. 인터넷 probe 실패만으로 private 설정 성공을 판정하지 않는다.

**증거 기록:** 실제 명령·UTC 시간·exit code·가려진 OCID·옵션 readback·검증 조건을 남긴다. 현재 이 문서의 실행상태는 `NOT_EXECUTED`다. output/creation-event-id/result는 실행 전 비워 둔다. 오류/NOT_SELECTED를 PASS로 바꾸지 않는다.

### 이 시나리오의 functional acceptance

| Positive | Negative / 미완료 판정 | Recovery / 반환 |
|---|---|---|
| public-ip=null/새 ProxyJump | B ingress만 있고 A egress22 없으면 신규 ProxyJump 실패 | 기록된 exact B/32 A egress22 추가/readback; 관리 key 원격 복사 없음 |

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

ProxyJump time-out이면 local→vm-a ingress뿐 아니라 vm-a NSG의 private target /32 TCP22 egress와 private VM ingress source=vm-a private/32를 함께 확인한다. SSH key를 jump host에 복사해서 해결하지 않는다.

## 리소스정리

1. INSTANCE_B_ID/B_PRIVATE_IP/boot/NSG/추가 rule ID; owner22가 B/boot→자기 rule→private NSG만 정리. 예약 소비자가 없거나 expiry가 끝나면 즉시 정리한다. 공유 vm-a/boot/public network는 삭제하지 않는다.
2. 대상 private-vm의 OCID와 attachment를 확인하고 guest workload를 종료한다. mount가 있으면 unmount/detach한다. Compute → 대상 → Terminate에서 Permanently delete attached boot volume을 선택한다. 정확한 instance TERMINATED/boot 잔존을 다시 읽으며 권한 오류를 삭제 성공으로 처리하지 않는다.
3. private-vm 종료 후 이 lab이 nsg-admin에 추가한 B/32 TCP22 egress만 기록한 rule ID로 Remove한다. 원래 HTTPS443 rule이 유지되는지 readback한다. private NSG의 membership이 비었을 때 nsg-private-admin만 삭제한다.
4. 로컬 전용 jump.conf는 관련 소비자가 끝났을 때만 제거한다. private-app/rt-private/sl-private는 owner21, public-admin/IGW는 owner23, VCN은 owner19에게 반환한다.


## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 아래 본문은 각 생성 필드·설정·검증·정리의 기술 근거이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [30] [Networking Overview — Private Subnets](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/overview.htm)
  - 확인 범위: Private means VNICs in the subnet can't.

### 추가 실행 절차 근거 (본문 확인)

- [subnet](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/create_subnet.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [launch](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/launchinginstance.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [keys](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/managingkeypairs.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [connect](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/connect-to-linux-instance.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [security](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/securitylists.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [cloudshell](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/cloudshellgettingstarted.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.

- [NSG directions, union, memberships](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/networksecuritygroups.htm) — 규칙 방향·VNIC association·합집합 및 삭제 선행조건 본문.
