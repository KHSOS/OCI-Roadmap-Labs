# Compute

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../README.md)를 확인하세요.

> 실행 절차 작성본 / OCI 미실행. Oracle 공식 본문 근거와 교육용 선택을 구분한다. secrets·실제 고객값·diagram 없음.

## 학습목표

- 준비된 key/public-admin/AD/shape/boot/user-data를 선택하여 공유 vm-a 한 세트를 한 번 launch하고 RUNNING·VNIC·boot를 조회한다.
- 새 SSH로 opc·architecture·cloud-init marker를 검증하고 실제 소비자 종료 후 instance와 boot 잔존을 확인하며 정리한다.

## 대응 Roadmap 항목

- 항목명: **Compute**
- Roadmap 경로: **Compute**
- Node ID: `sequence-2-0`
- 상위 항목: 없음 — 최상위 학습 항목
- 문서 범위: 준비된 key/public-admin/AD/shape/boot/user-data를 선택하여 공유 vm-a 한 세트를 한 번 launch하고 RUNNING·VNIC·boot를 조회한다. 새 SSH로 opc·architecture·cloud-init marker를 검증하고 실제 소비자 종료 후 instance와 boot 잔존을 확인하며 정리한다. 실제 OCI 실행 결과가 아니라 이 절차의 완료 조건이다.
- 번호는 navigation 식별자이며 실행 순서는 사전준비의 단계별 의존성에 따른다.

## 사전준비

### 기준 계약과 실제 단계 선행조건

[공통 baseline-contract.json](../../baseline-contract.json)의 Oracle Linux 9 / opc / CIDR / 비용 상한을 따른다. 문서 번호는 실행 순서가 아니다. **준비 station과 생성 이후 ready를 분리**한다. 아래 단계 외의 기능완료를 소급하여 요구하지 않는다.

- [15 SSH Keys — 단계 링크](05_SSH_Keys.md#key-preparation): **Part A 전용 공개키/키경로/fingerprint**. 실제 출력/소비자/expiry를 전달하고 해당 owner가 이 lab 종료까지 보존한다.
- [23 Public Subnets — 단계 링크](../03_Virtual_Cloud_Network_VCN/04_Public_Subnets.md#public-network-preparation): **단계 A AVAILABLE subnet/IGW/route/SL/NSG**. 실제 출력/소비자/expiry를 전달하고 해당 owner가 이 lab 종료까지 보존한다.
- [06 Availability Domains — 단계 링크](../01_Introduction_to_OCI/02_Availability_Domains.md#ad-selection): **실제 AD 조회/선택만**. 실제 출력/소비자/expiry를 전달하고 해당 owner가 이 lab 종료까지 보존한다.
- [11 Shapes — 단계 링크](01_Shapes.md#shape-selection): **architecture-compatible shape 선택만**. 실제 출력/소비자/expiry를 전달하고 해당 owner가 이 lab 종료까지 보존한다.
- [13 Boot Volume — 단계 링크](03_Boot_Volume.md#boot-selection): **boot 선택만**. 실제 출력/소비자/expiry를 전달하고 해당 owner가 이 lab 종료까지 보존한다.
- [17 Cloud-init / User Data — 단계 링크](07_Cloud_init_User_Data.md#user-data-preparation): **YAML 작성/크기/hash만**. 실제 출력/소비자/expiry를 전달하고 해당 owner가 이 lab 종료까지 보존한다.

생성 전 조직 승인(identity/권한/예산/expiry/서비스 지원/limit/capacity)을 확인한다. 실제 OCID/IP는 아직 생성하지 않았으면 비워 두고 해당 Create/readback 뒤 채운다. 동일 creation-event-id/OCID는 한 번의 제출만 뜻한다. `INSTANCE_A_ID`는 owner10 vm-a의 OCID, `INSTANCE_B_ID`는 owner22 private-vm의 OCID이며 owner07 vm-b와 다르다. 실습에 없는 자원을 추가하지 않는다. 실제 checkpoint는 사용자 실행 전 비워 둔다.

### 입력값과 실행 위치

Console에서 선택한 값만 사용한다. `<...>`는 예시 OCID나 비밀값이 아니라 **사용자가 대체할 입력 자리**다. 대체하지 않은 채 명령을 실행하지 않는다. Console → 우측 상단 Cloud Shell/Code Editor → Cloud Shell을 열면 OCI CLI가 사전 인증된다. Region 변경 뒤에는 Shell을 다시 열거나 명령의 `--region`을 바꾼다. SSH는 Cloud Shell이 아니라 본인 Linux/WSL/macOS 터미널에서 수행한다. 공인 IP/32도 그 터미널의 출구 주소여야 한다.

```bash
# 아래는 입력/출력의 대입 형식이다. 표의 단계 전에 생성 후 ID/IP export 행을 실행하지 않는다.
 # 생성 후 검증용 ID/IP는 해당 creator의 실제 응답에서 넣는다.
export REGION='<actual_REGION>'
export INSTANCE_A_ID='<actual_INSTANCE_A_ID>'
export KEY='<actual_KEY>'
export A_PUBLIC_IP='<actual_A_PUBLIC_IP>'
export INSTANCE_ID="$INSTANCE_A_ID"
export PUBLIC_IP="$A_PUBLIC_IP"
oci --version
```

이 변수 중 해당 절차에서 사용하는 값만 채운다. CLI 조회의 `data`를 증거 파일에 저장하고 실제 시간·작업 OCID·선택값을 기록한다. API key/private key/passphrase/인증 토큰은 기록하지 않는다. Cloud Shell 설정파일이나 세션 토큰을 출력하지 않는다. `oci --version`을 실행하는 주제의 기대 패턴은 버전 문자열뿐이다. namespace는 Object Storage 주제에서 별도 조회하며 다른 주제에는 요구하지 않는다. [Cloud Shell 공식 본문](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/cloudshellgettingstarted.htm).

### Workshop 입력 계약 — 단계와 host

| 입력/출력 | 값 또는 실제 획득 경로 | 사용할 시점 | 실행 위치 | 선택 이유 |
|---|---|---|---|---|
| Console/설정 필드 | Name=vm-a; lab-root; 승인 AD/FD; OL9 compatible A1(1/6) 또는 승인 E4(1/2); public-admin/nsg-admin/ephemeral; .pub; default boot; cloud-config 원문 | 생성 전 선택 → 생성 뒤 readback | 본문 Console/로컬 지정 | 이 topic의 최소 생성 범위 |
| `A_PUBLIC_IP` | owner10 vm-a VNIC public-ip | 선행 완료/생성 후 | 로컬 SSH | 관리 jump 주소 |
| `INSTANCE_A_ID` | owner10 실제 vm-a 생성 id | 선행 완료/생성 후 | Cloud Shell | 공유 A 조회; 이름은 OCID가 아님 |
| `INSTANCE_ID` | owner10 vm-a Create/readback id | 생성 후/선행 완료 | Cloud Shell | 현재 문서 대상 OCID에 명시 대입 |
| `KEY` | owner15 Part A 전용 private key 절대경로; 내용 전송 금지 | 준비 완료 | 로컬 SSH만 | 해당 공개키와 대응 |
| `LAB_COMPARTMENT_ID` | owner00 Create 응답 OCID; topic00에서는 생성 뒤 | 선행 완료/생성 후 | Console/Cloud Shell | 정책/자원 scope |
| `PUBLIC_IP` | A_PUBLIC_IP / owner10 VNIC public-ip | 생성 후/선행 완료 | 로컬 SSH | 대상 host 주소 |
| `REGION` | 승인된 기존 subscribed Region identifier | 생성 전 | Console/Cloud Shell | API endpoint/데이터 위치 |
| `VNIC_ID` | 해당 INSTANCE_ID의 vnic-attachments.json data[].vnic-id; primary VNIC 확인 | 생성 후 | Cloud Shell | 다른 VM VNIC 혼동 방지 |

생성 전에는 행의 승인값/선행 완료값만 지정하고, 생성 후 ID/IP는 해당 owner 응답으로 채운다. Cloud Shell과 로컬 SSH의 환경변수는 공유되지 않는다. 로컬에는 KEY와 접속 주소만 다시 지정하고 guest disk 명령은 로그인한 guest에서만 실행한다. `<...>`가 남으면 해당 명령은 실행하지 않는다.

## 권한

vm-a 단일 launch와 동일 instance/VNIC 조회·소비자 반환 후 terminate에 필요한 권한을 lab-root에 한정 승인한다. 기존 public-admin/NSG에 연결할 권한은 필요하지만 이 Compute 단계에서 VCN/subnet/gateway를 새로 만들 권한은 요청하지 않는다. boot 생성/종료 후 잔존 확인은 같은 creator의 승인 범위에 포함한다.

### 실제 operation과 공식 권한

- `GetInstance`: INSTANCE_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `GetVnic`: VNIC_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `LaunchInstance`: INSTANCE_CREATE and INSTANCE_IMAGE_READ and VNIC_CREATE and VNIC_ATTACH and SUBNET_ATTACH If putting the instance in a network security group during instance creation, also need NETWORK_SECURITY_GROUP _UPDATE_MEMBERS and VNIC_ASSOCIATE _NETWORK_SECURITY_GROUP If creating an instance in a compute cluster, also need COMPUTE_CLUSTER_LAUNCH_INSTANCE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `ListVnicAttachments`: VNIC_ATTACHMENT_READ and INSTANCE_INSPECT — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `TerminateInstance`: INSTANCE_DELETE and VNIC_DELETE and SUBNET_DETACH If volumes are attached, also need VOLUME_ATTACHMENT_DELETE and VOLUME_WRITE and INSTANCE_DETACH_VOLUME — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).

위 permission 이름은 IAM policy resource-type이 아니다. 조직 관리자가 기존 group/compartment scope와 conditional dependency를 실제 API 표에 대조한다. 편의 family grant를 최소권한 정책으로 부르지 않으며 실습자는 policy 작성 권한을 자동 인수하지 않는다.

**Boot attachment 조회 범위:** 이 문서는 미확인 `ListBootVolumeAttachments` 최소권한을 요구하는 CLI를 실행하지 않는다. 생성 owner가 승인된 Compute Console → 정확한 instance OCID → Storage → Boot volume에서 연결된 boot OCID/AD와 instance를 읽어 전달한다. boot를 검증하는 owner13만 GetBootVolume 권한을 승인받아 그 OCID를 조회한다. Console attachment를 볼 권한이 없으면 creator가 exact instance/boot 관계를 제공하기 전 중단한다. 권한 표에 없는 operation을 최소권한 검증 완료로 표시하지 않는다.

최초 SSH에 필요한 exact-instance Console connection/history는 관리자에게 해당 Create/Get/Delete/Capture 작업을 별도 승인받는다. [Compute의 독립 guest 신뢰 경로](00_Compute.md#guest-trust-bootstrap)의 BLOCKED_TRUST/BLOCKED_IAM gate를 보존한다. 승인 없이 public host fingerprint를 추정하거나 guest password를 만들지 않는다.

## 비용및주의사항

- 신규 과금 범위는 vm-a 1대의 승인 OCPU/memory, image-default boot volume, ephemeral IPv4 및 전송이다. 기존 public-admin/IGW/NSG를 재사용하고 추가 웹 서버·VM·KMS·backup policy는 만들지 않는다. Create Review에서 A1 1/6 또는 승인 E4 1/2와 실제 boot 크기를 예산 승인값과 대조한다.
- vm-a는 Shapes/Boot/Cloud-init의 단일 생성 event다. 각 주제에서 재제출하지 않는다. 소비자별 expiry까지 보존하되 전원 반환 후 owner10이 instance와 boot 삭제 여부를 따로 확인한다. 공유 sshd 중지 실험은 제외한다.

실제 OCI 작업은 아직 수행하지 않았다. 기대값은 실행 시 대조할 조건이며 무료 사용이나 과금 강제 차단을 보장하지 않는다. 승인 expiry가 끝나면 새 시험을 중단하고 위 소유권 순서로 원복·반환한다.

## 실습구성

### 이번 Workshop에서 만드는 것과 진행 순서

공유 vm-a 한 세트에 AD/shape/boot/SSH key/cloud-init 선택을 묶는다. 웹 application 설치 없이 sshd와 marker만 검증한다.

1. **진입/선택:** 15 key/23 network A/06 AD/11 shape/13 boot/17 YAML 준비. 사전준비 링크의 지정 단계/실제 출력이 없으면 제출하지 않는다.
2. **실제 작업:** Console Create 한 번 → RUNNING/VNIC/boot readback. 아래 Console 필드/명령을 해당 host에서 순서대로 실행한다. 같은 자원은 이름이 아니라 OCID/event로 재사용한다.
3. **완료/복구/반환:** 새 SSH/cloud-init/marker. 결과검증 실패는 문제해결로 이동하고 동일 Create를 무작정 재제출하지 않는다.

**전달/정리 범위:** INSTANCE_A_ID/A_PUBLIC_IP/A_PRIVATE_IP/boot OCID; owner10가 실제 소비자 종료 후 정리. 아래 정리 단계는 이 소유권을 넘어서 실행하지 않는다.


- **이번 생성/재사용/출력:** 신규 vm-a/VNIC/primary private IP/ephemeral public IP/boot volume 1세트와 marker. AD/Shapes/Boot/Cloud-init는 같은 launch event. 후속 lab까지 보존.
- **생성 목표:** 공유 vm-a를 실제 launch하고 login·VNIC·boot volume까지 확인하는 핵심 생성 lab. 06 AD,11 Shapes,13 Boot,17 cloud-init을 하나의 launch cohort로 묶는다.

## 실습절차

### 명령 실행 gate — 실패하면 다음 생성으로 진행하지 않는다

1. 아래 Bash는 **전용 Bash 세션**에서만 사용한다. Console/Cloud Shell 조회, 본인 로컬 SSH, 로그인한 Oracle Linux guest를 섞지 않는다. `exit 1`은 이 전용 세션을 닫는 안전 중단이다. `<...>`·빈 입력·타 owner OCID가 남으면 실행하지 않는다.
2. 성공을 요구하는 파일 작성/조회/설치/attach 작업은 실패 시 중단한다. 실패한 API 뒤 기존 JSON이나 기존 다운로드 파일을 성공 근거로 읽지 않는다. 기대 deny는 `if command; then ...; else RC=$?; ...; fi`로 별도 기록하고, 실제 timeout/IAM 오류/로컬 파일 오류를 구별한다.
3. 첫 변경 전에 정확한 OCID·owner·현재 설정과 승인 expiry를 저장한다. 공유 자원 변경 중 다른 소비자가 시작하거나 원복값을 확인할 수 없으면 변경을 중단한다. 삭제 성공은 API 수락이 아니라 같은 identity/Region의 정확한 대상 재조회로 판정한다. `NotAuthorizedOrNotFound` 단독은 삭제 증거가 아니다.


### 개별 완주 보정 — 생성·검증·소유권

Oracle Linux 9 platform image의 architecture와 승인 shape를 동시에 확인한다. KEY/공개키/NSG source와 cloud-init 파일을 Create 전에 선택하고 같은 event id를 기록한다. owner10만 모든 실제 소비자 종료 후 vm-a/boot를 삭제한다.

<a id="shared-launch"></a>

### Console 화면의 실제 진행 순서

Oracle 최신 [Creating an Instance](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/launchinginstance.htm)의 단계는 **Basic Information → Security → Networking → Storage → Review**다. 예전 tenancy에서는 단일 화면/탭 위치가 다를 수 있다. Basic Information의 Placement/Advanced Options에서 AD/FD/capacity와 initialization script를 확인하고, Networking에서 기존 VCN/subnet/NSG/public IPv4/공개키를, Storage에서 boot 옵션을 선택한다. Security의 Shielded/Confidential/보안 속성은 선택 image/shape 지원 및 조직 승인이 있을 때만 변경한다. 이 최소 lab은 새 보안 서비스나 KMS를 추가하지 않는다. 각 Next에서 값이 초기화되지 않았는지 확인하고 Review에서 전체 설정을 다시 비교한다. 메뉴 위치가 다르다고 없는 필드를 추정해 제출하지 않는다.

### 공유 launch station — 생성 전 선택부터 실제 제출까지

1. Console → Compute → Instances → Create instance. Name은 `vm-a`, Create in compartment는 `lab-root`; 같은 이름만 믿지 말고 Compartment OCID도 확인한다. Basic information/Placement에서 tenancy AD 목록에 있는 정확한 AD와 `FAULT-DOMAIN-1`을 선택한다. 빈 capacity면 승인된 다른 FD/AD를 사용하고 이유를 기록한다. 기본 Capacity type은 **On-demand**이다. [Creating an Instance — placement/capacity](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/launchinginstance.htm).
2. Image and shape → Change image: 무료 사용이 보장되는 것이 아니라 라이선스 조건을 먼저 읽고 **Oracle Linux 9 platform image**를 선택한다. ARM A1을 사용할 때는 ARM-compatible Oracle Linux image를 고른다. Change shape에서 `VM.Standard.A1.Flex`, **1 OCPU / 6 GB**를 1순위로 검토한다. tenancy eligibility/limit/capacity가 충족될 때만 선택한다. 지원되지 않으면 승인된 `VM.Standard.E4.Flex`, **1 OCPU / 2 GB**를 사용한다. 둘 다 없으면 자동으로 큰 shape를 고르지 말고 중단한다. shape와 image architecture를 동시에 확인한다. [Compute Shapes — Flexible Shapes](https://docs.oracle.com/en-us/iaas/Content/Compute/References/computeshapes.htm).
3. Networking: **Select existing virtual cloud network** → `shared-vcn`, **Select existing subnet** → `public-admin` Regional subnet. Assign a public IPv4 address = **Yes / Ephemeral**. NSG 선택(Show advanced options가 필요할 수 있음)에서 `nsg-admin`을 지정한다. 모든 NSG와 security list에 본인 출구 IP/32 외 광범위 TCP22 허용이 없는지 확인한다. 새로운 VCN wizard를 선택하지 않는다.
4. Add SSH keys → **Upload public key files / Paste public keys**에서 Part A의 `.pub`만 제출한다. `.pub`가 아닌 private key를 업로드하지 않는다. Boot volume은 선택한 image의 기본 최소 크기(화면에 표시되는 값)를 그대로 사용한다. Custom boot size, backup policy, replication, customer-managed KMS key는 선택하지 않는다. Balanced/default 성능과 Oracle-managed encryption을 확인한다. [Boot Volumes — launch creates a boot volume](https://docs.oracle.com/en-us/iaas/Content/Block/Concepts/bootvolumes.htm).
5. Advanced options → Management → **Initialization script/User data**에서 아래 `cloud-config` 파일을 업로드한다. Console이 base64 처리하므로 미리 encode하지 않는다. secrets/reboot 명령을 넣지 않는다. AD/Shapes/Boot/Cloud-init의 생성 전 선택을 한 번에 기록한다. [Creating an Instance — Initialization script](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/launchinginstance.htm).

```yaml
#cloud-config
write_files:
  - path: /var/tmp/oci-roadmap-marker.txt
    permissions: '0644'
    owner: root:root
    content: |
      roadmap-lab-created
runcmd:
  - [sh, -c, 'date -u > /var/tmp/oci-roadmap-created-at.txt']
  - [sh, -c, 'printf "ROADMAP_GUEST_ED25519_SHA256\n" > /dev/console && ssh-keygen -E sha256 -lf /etc/ssh/ssh_host_ed25519_key.pub > /dev/console']
```

6. Free-form tags `purpose=oci-roadmap`, `owner=<비민감_담당자_alias>`, `expiry=<승인_종료시각>`를 입력한다. Review에서 1 VM, 작은 shape, 기존 subnet, ephemeral IP, 공개키, boot size, initialization script를 다시 확인하고 **Create**를 누른다. OCID를 저장한다. 화면만 보는 것은 생성 완료가 아니다. Work request와 instance가 `RUNNING`인지 확인한다.
### 최초 SSH fingerprint의 신뢰 경로

<a id="guest-trust-bootstrap"></a>

첫 SSH 이전 신뢰 획득은 **guest password/기존 SSH 없이** 다음 순서로 한다. 이 절차는 같은 승인 instance의 진단 경로이며 새 VM/네트워크/서비스를 만들지 않는다.

1. Create 전 owner17의 원본 YAML에 아래 public-host-key 출력 runcmd가 있는지 확인한다. Compute 본문의 inline YAML도 같은 bytes로 유지한다. private host key/사용자 key/토큰은 출력하지 않는다. 이미 생성했는데 출력이 없으면 metadata 수정·reboot로 최초 증거를 꾸미지 않고 BLOCKED_TRUST로 반환한다.
2. 조직이 같은 대상의 일시적 진단 connection/history 사용까지 금지하면 **Create 이전 BLOCKED_TRUST**로 중단한다. 별도 인프라를 추가하거나 우회하지 않는다. 인증된 OCI Console → Compute → **정확한 대상 OCID** → OS Management/Resources → Console connection → Launch Cloud Shell connection. 관리자에게 공식 instance-console-connection 권한을 먼저 승인받는다. 이 진단 connection은 guest login이 아니라 serial 출력 경로다; opc password를 요구하거나 reset하지 않는다. Console service RSA fingerprint는 guest ED25519와 구분한다. [공식 Cloud Shell serial 경로](https://docs.oracle.com/en-us/iaas/Content/Compute/References/serialconsole.htm).
3. 같은 OCID의 Console history → View current history → Save and close → 해당 capture Actions → View details로 초기 boot 출력의 `ROADMAP_GUEST_ED25519_SHA256` 바로 뒤 ED25519 SHA256 행을 읽는다. [Capture 선행 connection/필드](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/displayingconsole-capturing.htm), [View details](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/displayingconsole-viewing.htm). 30초 간격 최대 5분 또는 expiry 중 먼저 오는 시점까지만 확인한다. 최근 1MB/point-in-time history라 누락될 수 있다; 없거나 모호하면 BLOCKED_TRUST, TOFU/keyscan 우회 금지.
4. 관리자와 학습자가 대상 OCID/Region/creation-event-id/capture UTC/ED25519 SHA256를 연결해 전달한다. vm-a는 INSTANCE_A_ID, vm-b/burstable/preemptible/reserved/dedicated/ip-lab은 **각 실제 신규 OCID**다. 다른 guest 신뢰값을 복사하지 않는다. 로컬 KEY와 PUBLIC_IP도 해당 creator의 실제 값으로 다시 지정한다.
5. 로컬 첫 접속은 아래 bounded 비다중화 세션의 표시를 신뢰값과 exact 비교한 경우만 yes로 수락한다. 불일치/timeout은 중단한다. 이후는 StrictHostKeyChecking=yes로 새 세션을 사용한다. 검사 뒤 담당자가 해당 진단 connection만 disconnect/delete, 해당 history capture만 Actions → Delete하고 목록 readback한다. 다른 capture/connection은 삭제하지 않는다. [History cleanup](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/displayingconsole-deleting.htm). 기존 trusted 관리 세션이 있을 때만 같은 OCID에서 `sudo ssh-keygen -E sha256 -lf /etc/ssh/ssh_host_ed25519_key.pub`로 재확인할 수 있다.

```bash
# 본인 Linux/WSL 로컬 전용 Bash: 값은 같은 guest의 실제 creator/trust handoff
export KEY='<actual_local_private_key_absolute_path>'
export PUBLIC_IP='<same_guest_actual_public_IP>'
# 60초 전체 상한: passphrase/fingerprint 입력도 이 시간 안에서만; 실패 시 무한 재시도 금지
 timeout --foreground 60s ssh -i "$KEY" -o IdentitiesOnly=yes -o HostKeyAlgorithms=ssh-ed25519 -o StrictHostKeyChecking=ask -o ConnectTimeout=10 -o ConnectionAttempts=1 -o ControlMaster=no -o ControlPath=none opc@"$PUBLIC_IP" 'hostname; id -un' || exit 1
```

GNU timeout/OpenSSH가 없으면 설치를 임의 진행하지 않고 로컬 preflight에서 BLOCKED다. 이 public-host-key 출력은 작성한 Linux/cloud-init recipe이며 OCI에서 실측한 결과가 아니다.

7. Instance access의 공인 IP를 얻고 위 trust bootstrap의 bounded local 세션으로 최초 fingerprint를 비교한다. 신뢰 확인 뒤 후속 접속은 `timeout --foreground 60s ssh -i "$KEY" -o IdentitiesOnly=yes -o HostKeyAlgorithms=ssh-ed25519 -o StrictHostKeyChecking=yes -o ConnectTimeout=10 -o ConnectionAttempts=1 -o ControlMaster=no -o ControlPath=none opc@"$PUBLIC_IP"`로 새 세션을 연다. guest에서 `uname -m; id; sudo timeout -k 5s 40s cloud-init status --wait && sudo cat /var/tmp/oci-roadmap-marker.txt`를 실행한다. 기대 패턴: image에 맞는 `aarch64` 또는 `x86_64`, 사용자 `opc`, cloud-init 완료, marker 한 줄. cloud-init warning/error면 로그를 먼저 읽는다. [Linux SSH 연결](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/connect-to-linux-instance.htm).

### Create 이후 값 전달 — 이름으로 추정하지 않는다

Cloud Shell의 INSTANCE_ID는 해당 creator Create 응답 id로 지정한다. Console → Compute → Instances → 정확한 instance → Networking/Attached VNICs에서 primary VNIC OCID를 기록하고 아래 GetVnic의 `is-primary=true`, `subnet-id`, `private-ip`, `public-ip`를 대조한다. Storage/Boot volume의 연결 boot OCID·AD도 creator가 기록한다. Cloud Shell과 로컬 터미널의 변수는 공유되지 않으므로 SSH 전에 KEY와 **그 instance의** 실제 접속 주소를 로컬에 다시 지정한다. 이전 extra VM 주소를 재사용하지 않는다. 모든 JSON 조회는 실패하면 즉시 중단하고 기존 파일을 읽지 않는다.

### guest health와 허용 범위를 비파괴 확인한다

새 SSH 후 **vm-a guest**에서 다음을 실행한다. 이 Compute lab의 application은 sshd와 cloud-init marker이며 웹 서버를 추가하지 않는다.

```bash
# Oracle Linux9 vm-a guest
set -euo pipefail
grep '^ID=' /etc/os-release
id -un
sudo systemctl is-active sshd
sudo ss -lntp '( sport = :22 )'
if systemctl is-active --quiet firewalld; then
  sudo firewall-cmd --get-active-zones
  # primary interface zone을 실제 출력으로 대조하고 아래 값 지정
  export FIREWALL_ZONE='<primary_interface_actual_zone>'
  case "$FIREWALL_ZONE" in ''|*'<'*|*'>'*) exit 1;; esac
  sudo firewall-cmd --zone="$FIREWALL_ZONE" --list-all
fi
```

기대값은 opc, sshd active, TCP22 listener와 실제 primary zone의 SSH 허용이다. inactive firewalld를 켜거나 전체 disable하지 않는다. OCI 허용은 guest와 별개로 source local/32 ingress22, A→B jump 시에만 destination B/32 egress22다. 잘못된 public key/file 권한 negative는 owner15의 로컬 안전 gate를 사용하고 공유 vm-a의 sshd를 중단하는 destructive test는 기본에서 제외한다.

## 결과검증

<a id="baseline-10-ready"></a>

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
# 아래는 로컬 Linux/WSL 터미널에서 실행
export KEY='<Part_A의_private_key_경로>'
export PUBLIC_IP='<vnic-readback의_public-ip>'
timeout --foreground -k 5s 60s ssh -i "$KEY" -o IdentitiesOnly=yes -o HostKeyAlgorithms=ssh-ed25519 -o StrictHostKeyChecking=yes -o ConnectTimeout=10 -o ConnectionAttempts=1 -o ControlMaster=no -o ControlPath=none opc@"$PUBLIC_IP" 'uname -m; id; sudo timeout -k 5s 40s cloud-init status --wait && sudo cat /var/tmp/oci-roadmap-marker.txt' || exit 1
```
검증은 API RUNNING만이 아니라 실제 새 SSH 세션과 guest marker까지 요구한다. AD/Shapes/Boot/cloud-init의 생성 전 선택 증거가 누락되면 해당 checkpoint를 소급해 생성 참여라고 쓰지 않는다.

**증거 기록:** 실제 명령·UTC 시간·exit code·가려진 OCID·옵션 readback·검증 조건을 남긴다. 현재 이 문서의 실행상태는 `NOT_EXECUTED`다. output/creation-event-id/result는 실행 전 비워 둔다. 오류/NOT_SELECTED를 PASS로 바꾸지 않는다.

### 이 시나리오의 functional acceptance

| Positive | Negative / 미완료 판정 | Recovery / 반환 |
|---|---|---|
| 새 SSH/sshd/marker/architecture | RUNNING만인데 marker 없음이면 user-data checkpoint 실패 | 최초 script/log 확인; clean/reboot나 웹 app 설치로 대체하지 않음 |

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

RUNNING인데 marker가 없으면 초기 user-data 제출 파일/지원 image/cloud-init 로그를 조사한다. 동일 이름만 보고 다른 VM에 접속한 것은 아닌지 OCID/public IP/host identity를 대조한다.

## 리소스정리

1. 다른 lab의 소비자와 보존 종료시각을 먼저 확인한다. 생성 작업에 참여한 AD/Shapes/Boot/cloud-init 주제는 같은 `vm-a` OCID를 공유하므로 별도 VM을 삭제하지 않는다.
2. guest의 실습 파일을 제거하고 mount를 사용했다면 먼저 unmount한다. Console → Compute → Instances → 대상 → Terminate에서 OCID/name을 재확인하고 **Permanently delete attached boot volume**을 선택한다. 이 체크를 생략하면 보존된 boot 비용이 남는다. 인스턴스 상세 상태 `TERMINATED`와 Storage → Boot Volumes의 잔여 volume을 다시 읽는다.
3. data volume은 instance terminate만으로 지워졌다고 가정하지 않는다. attachment를 제거한 뒤 Storage → Block Volumes에서 이번 lab의 OCID만 삭제한다. backup·예약 public IP·Functions application·NSG는 별도 정리한다.
4. 기본 `vm-a`는 같은 승인 session의 후속 SSH/Block/Private Subnet 검증까지 보존한다. 예약된 소비자나 승인 expiry가 없으면 즉시 정리한다. Stop만으로 storage/public IP 등 모든 비용이 끝났다고 주장하지 않는다.


## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 아래 본문은 각 생성 필드·설정·검증·정리의 기술 근거이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [16] [Creating an Instance — Compute](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/launchinginstance.htm)
  - 확인 범위: Creating an Instance.

### 추가 실행 절차 근거 (본문 확인)

- [launch](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/launchinginstance.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [shapes](https://docs.oracle.com/en-us/iaas/Content/Compute/References/computeshapes.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [keys](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/managingkeypairs.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [connect](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/connect-to-linux-instance.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [boot](https://docs.oracle.com/en-us/iaas/Content/Block/Concepts/bootvolumes.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [cloudshell](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/cloudshellgettingstarted.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
