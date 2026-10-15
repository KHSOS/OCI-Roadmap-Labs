> **문서 검토 완료 — OCI 실행 미검증**: 이 Public IP Addresses 실습의 한정된 절차·복구·정리 범위를 문서 기준으로 검토했습니다. 실제 OCI 실행 검증이나 전체 교육과정 승인을 뜻하지 않습니다.

# Public IP Addresses

> 문서 승인 범위: **한정된 실행 절차의 문서 검토 완료**. 확인된 기존 보정사항을 반영한 이 문서의 시나리오와 복구·정리 절차에 한정합니다. 실제 OCI 실행은 미검증이며 모든 예상 출력은 실측값이 아닙니다. 실행 시 권한·안전·예산·복구·지원 gate는 그대로 적용됩니다. 게시 및 렌더링 검증은 별도이며 [통합 상태](../../../Docs/README.md)는 전체 교육과정의 상태입니다.

> 실행 절차 작성본 / OCI 미실행. Oracle 공식 본문 근거와 교육용 선택을 구분한다. secrets·실제 고객값·diagram 없음.

## 학습목표

- 폐기 가능한 vm-b 또는 승인 ip-lab-vm의 primary private IP OCID에 신규 Region-scoped reserved IPv4 한 개를 할당한다.
- ephemeral 해제→reserved 할당 후 새 SSH와 address/OCID 대응을 검증하고 reserved unassign/delete 후 전용 VM/boot를 정리한다.

## 대응 Roadmap 항목

- 항목명: **Public IP Addresses**
- Roadmap 경로: **Compute → Public IP Addresses**
- Node ID: `sequence-2-6`
- 상위 항목: **Compute** (`sequence-2-0`)
- 문서 범위: 폐기 가능한 vm-b 또는 승인 ip-lab-vm의 primary private IP OCID에 신규 Region-scoped reserved IPv4 한 개를 할당한다. ephemeral 해제→reserved 할당 후 새 SSH와 address/OCID 대응을 검증하고 reserved unassign/delete 후 전용 VM/boot를 정리한다. 실제 OCI 실행 결과가 아니라 이 절차의 완료 조건이다.
- 번호는 navigation 식별자이며 실행 순서는 사전준비의 단계별 의존성에 따른다.

## 사전준비

### 기준 계약과 실제 단계 선행조건

[공통 baseline-contract.json](../../../baseline-contract.json)의 Oracle Linux 9 / opc / CIDR / 비용 상한을 따른다. 문서 번호는 실행 순서가 아니다. **준비 station과 생성 이후 ready를 분리**한다. 아래 단계 외의 기능완료를 소급하여 요구하지 않는다.

- [15 SSH Keys — 단계 링크](05_SSH_Keys.md#key-preparation): **Part A 키**. 실제 출력/소비자/expiry를 전달하고 해당 owner가 이 lab 종료까지 보존한다.
- [23 Public Subnets — 단계 링크](../../Networking/Original-Provisional/03_Virtual_Cloud_Network_VCN/04_Public_Subnets.md#public-network-preparation): **단계 A 완료**. 실제 출력/소비자/expiry를 전달하고 해당 owner가 이 lab 종료까지 보존한다.
- [07 Fault Domains — 단계 링크](../01_Introduction_to_OCI/03_Fault_Domains.md#baseline-07-ready): **조건부: 남아 있는 disposable vm-b 재사용; 없으면 owner16가 승인 temporary VM 생성**. 실제 출력/소비자/expiry를 전달하고 해당 owner가 이 lab 종료까지 보존한다.

생성 전 조직 승인(identity/권한/예산/expiry/서비스 지원/limit/capacity)을 확인한다. 실제 OCID/IP는 아직 생성하지 않았으면 비워 두고 해당 Create/readback 뒤 채운다. 동일 creation-event-id/OCID는 한 번의 제출만 뜻한다. `INSTANCE_A_ID`는 owner10 vm-a의 OCID, `INSTANCE_B_ID`는 owner22 private-vm의 OCID이며 owner07 vm-b와 다르다. 실습에 없는 자원을 추가하지 않는다. 실제 checkpoint는 사용자 실행 전 비워 둔다.

### 입력값과 실행 위치

Console에서 선택한 값만 사용한다. `<...>`는 예시 OCID나 비밀값이 아니라 **사용자가 대체할 입력 자리**다. 대체하지 않은 채 명령을 실행하지 않는다. Console → 우측 상단 Cloud Shell/Code Editor → Cloud Shell을 열면 OCI CLI가 사전 인증된다. Region 변경 뒤에는 Shell을 다시 열거나 명령의 `--region`을 바꾼다. SSH는 Cloud Shell이 아니라 본인 Linux/WSL/macOS 터미널에서 수행한다. 공인 IP/32도 그 터미널의 출구 주소여야 한다.

```bash
# 아래는 입력/출력의 대입 형식이다. 표의 단계 전에 생성 후 ID/IP export 행을 실행하지 않는다.
 # 생성 후 검증용 ID/IP는 해당 creator의 실제 응답에서 넣는다.
export REGION='<actual_REGION>'
export PUBLIC_IP_OCID='<actual_PUBLIC_IP_OCID>'
oci --version
```

이 변수 중 해당 절차에서 사용하는 값만 채운다. CLI 조회의 `data`를 증거 파일에 저장하고 실제 시간·작업 OCID·선택값을 기록한다. API key/private key/passphrase/인증 토큰은 기록하지 않는다. Cloud Shell 설정파일이나 세션 토큰을 출력하지 않는다. `oci --version`을 실행하는 주제의 기대 패턴은 버전 문자열뿐이다. namespace는 Object Storage 주제에서 별도 조회하며 다른 주제에는 요구하지 않는다. [Cloud Shell 공식 본문](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/cloudshellgettingstarted.htm).

### Workshop 입력 계약 — 단계와 host

| 입력/출력 | 값 또는 실제 획득 경로 | 사용할 시점 | 실행 위치 | 선택 이유 |
|---|---|---|---|---|
| Console/설정 필드 | Name=ip-lab-reserved; REGION/RESERVED; target=disposable VM primary private IP OCID; vm-a IP 변경 금지 | 생성 전 선택 → 생성 뒤 readback | 본문 Console/로컬 지정 | 이 topic의 최소 생성 범위 |
| `PUBLIC_IP_OCID` | owner16 신규 reserved public IP id | 생성 후 | Console/Cloud Shell | address 문자열/VNIC id와 구분 |
| `KEY` | owner15 Part A 전용 private key 절대경로; 내용 전송 금지 | 준비 완료 | 로컬 SSH만 | 해당 공개키와 대응 |
| `REGION` | 승인된 기존 subscribed Region identifier | 생성 전 | Console/Cloud Shell | API endpoint/데이터 위치 |

생성 전에는 행의 승인값/선행 완료값만 지정하고, 생성 후 ID/IP는 해당 owner 응답으로 채운다. Cloud Shell과 로컬 SSH의 환경변수는 공유되지 않는다. 로컬에는 KEY와 접속 주소만 다시 지정하고 guest disk 명령은 로그인한 guest에서만 실행한다. `<...>`가 남으면 해당 명령은 실행하지 않는다.

## 권한

ip-lab-reserved의 Create/Get/assign/unassign/Delete와 대상 disposable private IP/VNIC 조회를 승인한다. UpdatePublicIp의 공식 use private-ips/manage public-ips 의존을 관리자가 확인하기 전 전환은 BLOCKED_IAM이다. 신규 ip-lab-vm branch를 실제 선택할 때만 owner10 launch recipe의 Compute/boot 권한을 추가 승인하며 vm-b 재사용에서는 Launch/Terminate를 자동 위임하지 않는다.

### 실제 operation과 공식 권한

- `CreatePublicIp`: For ephemeral public IPs: PRIVATE_IP_ASSIGN_PUBLIC_IP For reserved public IPs: PUBLIC_IP_CREATE and PUBLIC_IP_ASSIGN _PRIVATE_IP and PRIVATE_IP _ASSIGN_PUBLIC_IP — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `DeletePublicIp`: For ephemeral public IPs: PRIVATE_IP_UNASSIGN_PUBLIC_IP For reserved public IPs: PUBLIC_IP_DELETE and PUBLIC_IP _UNASSIGN_PRIVATE_IP and PRIVATE_IP _UNASSIGN_PUBLIC_IP — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `GetPrivateIp`: PRIVATE_IP_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `GetPublicIp`: For ephemeral public IPs: PRIVATE_IP_READ For reserved public IPs: PUBLIC_IP_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `GetVnic`: VNIC_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `LaunchInstance`: INSTANCE_CREATE and INSTANCE_IMAGE_READ and VNIC_CREATE and VNIC_ATTACH and SUBNET_ATTACH If putting the instance in a network security group during instance creation, also need NETWORK_SECURITY_GROUP _UPDATE_MEMBERS and VNIC_ASSOCIATE _NETWORK_SECURITY_GROUP If creating an instance in a compute cluster, also need COMPUTE_CLUSTER_LAUNCH_INSTANCE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `TerminateInstance`: INSTANCE_DELETE and VNIC_DELETE and SUBNET_DETACH If volumes are attached, also need VOLUME_ATTACHMENT_DELETE and VOLUME_WRITE and INSTANCE_DETACH_VOLUME — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `UpdatePublicIp`: use  /  READ + PRIVATE_IP_UPDATE PRIVATE_IP_ASSIGN PRIVATE_IP_UNASSIGN PRIVATE_IP_CREATE PRIVATE_IP_DELETE PRIVATE_IP_ASSIGN_PUBLIC_IP PRIVATE_IP_UNASSIGN_PUBLIC_IP  /  READ + For ephemeral public IPs: UpdatePublicIp, CreatePublicIp, DeletePublicIp  /  CreatePrivateIp, DeletePrivateIp (both also need use subnets and use vnics ) UpdatePrivateIp (also needs use vnics ) For reserved public IPs: UpdatePublicIp, CreatePublicIp, DeletePublicIp (all also need manage public-ips ) Note: The above operations in this cell are totally covered with just use virtual-network-family . — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm). 단일 API 최소권한 미확정.
- `UpdatePublicIp`: use  /  READ + PUBLIC_IP_ASSIGN_PRIVATE_IP PUBLIC_IP_UNASSIGN_PRIVATE_IP  /  no extra  /  For reserved public IPs: UpdatePublicIp, CreatePublicIp, DeletePublicIp (all of these also need use private-ips and manage public-ips ). Note: The above operations in this cell are totally covered with just manage virtual-network-family . — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm). 단일 API 최소권한 미확정.
- `UpdatePublicIp`: manage  /  USE + PUBLIC_IP_UPDATE PUBLIC_IP_CREATE PUBLIC_IP_DELETE  /  no extra  /  USE + For reserved public IPs: UpdatePublicIp, CreatePublicIp, DeletePublicIp (all of these also need use private-ips ). Note: The above operations in this cell are totally covered with just manage virtual-network-family . — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm). 단일 API 최소권한 미확정.

위 permission 이름은 IAM policy resource-type이 아니다. 조직 관리자가 기존 group/compartment scope와 conditional dependency를 실제 API 표에 대조한다. 편의 family grant를 최소권한 정책으로 부르지 않으며 실습자는 policy 작성 권한을 자동 인수하지 않는다.

최초 SSH에 필요한 exact-instance Console connection/history는 관리자에게 해당 Create/Get/Delete/Capture 작업을 별도 승인받는다. [Compute의 독립 guest 신뢰 경로](00_Compute.md#guest-trust-bootstrap)의 BLOCKED_TRUST/BLOCKED_IAM gate를 보존한다. 승인 없이 public host fingerprint를 추정하거나 guest password를 만들지 않는다.

## 비용및주의사항

- reserved IPv4는 ip-lab-reserved 1개만 승인한다. 남아 있는 disposable vm-b를 우선 재사용하고 없을 때만 승인된 ip-lab-vm/boot 1세트를 순차 생성한다. vm-a 관리 주소는 변경하지 않는다. 주소 보유·VM·boot·전송 단가와 종료시각을 따로 확인한다.
- ephemeral 해제 후 같은 주소 복구를 보장하지 않는다. same private IP OCID/reserved OCID와 열린 Console 관리 창을 확보하고 전환 최대120초 창을 지킨다. 개인 기본 경로는 SSH negative를 생략하며 독립 담당자 시험은 선택이다. reserved unassign/delete를 먼저 확인한 뒤 전용 VM owner에게 반환·종료한다.

실제 OCI 작업은 아직 수행하지 않았다. 기대값은 실행 시 대조할 조건이며 무료 사용이나 과금 강제 차단을 보장하지 않는다. 승인 expiry가 끝나면 새 시험을 중단하고 위 소유권 순서로 원복·반환한다.

## 실습구성

### 이번 Workshop에서 만드는 것과 진행 순서

공유 vm-a IP는 건드리지 않고 disposable vm-b 또는 승인 ip-lab-vm의 primary private IP에 reserved IPv4 한 개를 배정한다.

1. **진입/선택:** 폐기 VM 확보. 사전준비 링크의 지정 단계/실제 출력이 없으면 제출하지 않는다.
2. **실제 작업:** private IP OCID 기록 → reserved 생성 → ephemeral 해제 → same private IP에 assign → 새 주소 SSH. 아래 Console 필드/명령을 해당 host에서 순서대로 실행한다. 같은 자원은 이름이 아니라 OCID/event로 재사용한다.
3. **완료/복구/반환:** unassign/delete. 결과검증 실패는 문제해결로 이동하고 동일 Create를 무작정 재제출하지 않는다.

**전달/정리 범위:** reserved public IP OCID/address와 private IP OCID; owner16가 reserved 먼저 정리 후 disposable owner에게 반환. 아래 정리 단계는 이 소유권을 넘어서 실행하지 않는다.


- **이번 생성/재사용/출력:** 신규 reserved IPv4 1개, disposable VM 1개 재사용 또는 순차 생성. 관리 VM 접속 보호. Primary private IP object에 assign.
- **생성 목표:** 10 ephemeral public IP 생성과 별도 reserved public IP 생성/배정을 실제 비교. 관리 연결이 끊길 수 있어 disposable vm-b 또는 second lab private IP를 이용한 승인된 전환 트랙으로 구성.

## 실습절차

### 명령 실행 gate — 실패하면 다음 생성으로 진행하지 않는다

1. 아래 Bash는 **전용 Bash 세션**에서만 사용한다. Console/Cloud Shell 조회, 본인 로컬 SSH, 로그인한 Oracle Linux guest를 섞지 않는다. `exit 1`은 이 전용 세션을 닫는 안전 중단이다. `<...>`·빈 입력·타 owner OCID가 남으면 실행하지 않는다.
2. 성공을 요구하는 파일 작성/조회/설치/attach 작업은 실패 시 중단한다. 실패한 API 뒤 기존 JSON이나 기존 다운로드 파일을 성공 근거로 읽지 않는다. 기대 deny는 `if command; then ...; else RC=$?; ...; fi`로 별도 기록하고, 실제 timeout/IAM 오류/로컬 파일 오류를 구별한다.
3. 첫 변경 전에 정확한 OCID·owner·현재 설정과 승인 expiry를 저장한다. 공유 자원 변경 중 다른 소비자가 시작하거나 원복값을 확인할 수 없으면 변경을 중단한다. 삭제 성공은 API 수락이 아니라 같은 identity/Region의 정확한 대상 재조회로 판정한다. `NotAuthorizedOrNotFound` 단독은 삭제 증거가 아니다.


**최대 두 VM 실행 gate:** vm-a 외 vm-b/burstable-vm/ip-lab-vm/preemptible-vm/private-vm는 같은 시간에 한 개만 실행한다. 이전 extra VM/boot 정리를 readback한 후 다음 extra를 생성한다. owner07 vm-b를 owner16이 쓰는 경우에는 새로운 VM을 만들지 않고 09 검증을 먼저 끝낸다.

### 개별 완주 보정 — 생성·검증·소유권

Console 복구세션을 유지하고 폐기대상만 지정한다. reservedIP는 먼저 unassign/delete한 뒤 임시VM/boot를 정리한다. UpdatePublicIp는 official resource verb/dependency본문 근거이며 단일operation최소권한은 미확정으로 남긴다.

1. public-admin의 **폐기 가능한 vm-b**가 필요하다. Fault Domains에서 즉시 이어진 승인 session이면 그 VM을 재사용한다. 없으면 [Compute 공동 launch](00_Compute.md#shared-launch)의 small launch recipe와 [user-data 준비](07_Cloud_init_User_Data.md#user-data-preparation)를 사용해 temporary `ip-lab-vm` 1개를 새로 생성하고 예산/expiry를 승인한다. 새 instance OCID에 대해 [guest trust bootstrap](00_Compute.md#guest-trust-bootstrap)을 완료하여 ED25519 fingerprint를 먼저 보존한다. 기존 vm-a fingerprint를 재사용하지 않는다. shared vm-a 관리 IP를 바꾸지 않는다. Networking/primary VNIC → IPv4 Addresses에서 primary private IP의 **private IP OCID**와 ephemeral public IP를 기록한다.
2. Networking → IP Management → Reserved Public IPs(또는 Public IPs) → Reserve/Create public IP. Name `ip-lab-reserved`, Compartment lab-root, **Region-scoped reserved**로 생성한다. 아직 private IP에 연결하지 않은 상태를 조회한다. public IPv4 비용/limit를 별도 승인한다. [Create reserved public IP — name/compartment/default Oracle pool](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/reserved-public-ip-create.htm).
### 개인 기본 경로 — Console에서 바로 전환

1인 학습자는 아래 고급 negative 시험 대신 이 경로를 사용한다. vm-a가 아닌 disposable 대상의 instance/VNIC/private IP OCID와 새 reserved OCID/address를 기록하고 열린 Console의 IP administration 화면에서 두 OCID를 다시 대조한다. 같은 Region의 reserved object가 unassigned인지 확인한다. 기존 접속이 끊길 수 있으므로 SSH 안에서 변경하지 않는다.

3. primary private IP 행 → Edit → **No public IP** → Update. 같은 행을 다시 열어 public IP가 없음을 확인한다. 이전 주소로 negative 접속을 시도하지 않고 즉시 아래 4단계로 진행한다. 전환은 120초 안에 끝내며 할당 실패면 Console을 유지하고 이 문서 문제해결의 Region/private IP/기존 assignment부터 확인한다. 예산 기한이 끝나면 reserved 정리 후 VM creator에게 반환한다.
4. 같은 행 → Edit → **Reserved public IP** → **Select Existing Reserved IP Address** → `ip-lab-reserved` → Update. [공식 assign 절차](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/reserved-public-ip-assign.htm).
5. 아래 고급 절차의 5–6단계와 결과검증을 완료한다. negative는 `NOT_SELECTED`로 두며 기본 완료에 필요하지 않다.

### 선택 고급 경로 — transport negative와 독립 복구

아래 3–4단계는 위 개인 경로와 **택일**한다. 고급 시험을 선택할 때만 독립 Console 담당자와 기한을 합의한다. 개인 기본 경로를 마친 뒤 다시 해제하지 않는다.

### 전환 전 positive control / 복구 gate

로컬 Linux/WSL 전용 세션에서 `command -v timeout; command -v ssh-add`를 먼저 확인한다. 기존 본인 ssh-agent가 없으면 이 전용 세션에서 `eval "$(ssh-agent -s)"`를 실행한다. `ssh-add -t 15m "$KEY"`의 interactive prompt로만 unlock하고 `ssh-add -l`의 public fingerprint가 owner15 공개키와 일치하는지 확인한다. key/passphrase contents는 기록하지 않는다. agent lifetime/승인 expiry 안에서 같은 KEY/agent/사용자/endpoint/host trust/네트워크/OS 권한을 고정한다; agent 만료면 No public IP 변경 금지 또는 Console로 복구부터 한다. positive/negative/recovery 모두 BatchMode=yes와 동일한 새 세션 옵션을 사용한다. 검사 종료 후 본인 추가 key만 `ssh-add -d "$KEY"`; 이 실습에서 시작한 agent만 종료한다.

정확한 disposable INSTANCE_ID/VNIC_ID/PRIVATE_IP_ID/기존 ephemeral **주소 문자열**을 기록하고 `OLD_EPHEMERAL_IP`에 지정한다. 본인 local KEY의 host 신뢰값은 먼저 [Compute fingerprint 경로](00_Compute.md#shared-launch)로 비교한다. 로컬에서 `timeout --foreground 30s ssh -i "$KEY" -o IdentitiesOnly=yes -o BatchMode=yes -o HostKeyAlgorithms=ssh-ed25519 -o StrictHostKeyChecking=yes -o ConnectTimeout=10 -o ConnectionAttempts=1 -o ControlMaster=no -o ControlPath=none opc@"$OLD_EPHEMERAL_IP" 'test "$(id -un)" = opc'` exit0를 기록한다. 새 reserved object의 OCID/address, 같은 private IP에 할당 가능한 상태와 Console 독립 관리 경로를 확인하지 못하면 No public IP로 변경하지 않는다. vm-a OCID/VNIC이면 금지한다.

**3 시작 전 독립 recovery gate:** Console 담당자가 PRIVATE_IP_ID/PUBLIC_IP_OCID와 원래값을 보존하고 No public IP 저장부터 최대120초 또는 expiry 중 이른 시각에4를 한 번만 수행/readback한다. local 요청이 멈춰도 이 deadline은 유지된다. 담당자/승인 없으면3을 시작하지 않는다.

3. ip-lab VM의 primary VNIC → IPv4 Addresses → primary private IP 행 → Edit에서 기존 ephemeral public IP를 해제(**No public IP**)하고 저장한다. 기존 연결은 끊길 수 있으므로 다른 Console 관리자 경로를 유지하고 disposable 대상에만 수행한다.
No public IP 저장 직후 Cloud Shell GetVnic의 `public-ip=null`/같은 VNIC·private IP를 readback한다. 그 다음 **본인 local의 새 비다중화 SSH**로 이전 주소에 한 번만 아래 시험한다. Console 관리자는 즉시 다음 4단계로 같은 reserved object를 같은 private IP에 할당할 준비를 유지한다. negative가 뜻밖에 성공해도 반복하지 않고 복구 먼저 수행한 후 wrong target/IP 재할당 여부를 조사한다.

```bash
# 본인 로컬; OLD_EPHEMERAL_IP는 변경 전 positive에서 실제 사용한 주소
if timeout --foreground 30s ssh -i "$KEY" -o IdentitiesOnly=yes -o BatchMode=yes -o HostKeyAlgorithms=ssh-ed25519 -o StrictHostKeyChecking=yes -o ConnectTimeout=10 -o ConnectionAttempts=1 -o ControlMaster=no -o ControlPath=none opc@"$OLD_EPHEMERAL_IP" 'test "$(id -un)" = opc'; then
  NEGATIVE_RESULT=UNEXPECTED_SUCCESS
else
  RC=$?
  printf 'new SSH exit=%s\n' "$RC"
  NEGATIVE_RESULT=BLOCKED_DIAGNOSIS
  # transport 오류 text + exit255일 때만 bounded transport negative로 기록한다.
  # publickey/hostkey 오류는 IP negative 성공이 아니다.
fi
```
실제 stderr가 timeout/refused/no-route 등 transport 오류이고 exit255인지 기록한다. released ephemeral이 다른 대상에 재할당될 수도 있으므로 negative 자체만으로 same-target 상태를 증명하지 않는다. null readback을 함께 사용한다. negative의 결과와 무관하게 **4단계로 바로 이동**한다. 여기서는 assignment를 실행하지 않는다. No public IP 변경 **전** 독립 Console 담당자에게 같은 PRIVATE_IP_ID/PUBLIC_IP_OCID와 원복값을 전달하고 변경 시점부터 **최대120초 또는 expiry 중 이른 시각**까지4의 assignment/readback을 한 번만 수행하도록 합의한다. 요청 terminal 종료/timeout에도 담당자 시계는 계속된다.4에서 이미 복구했으면 재할당하지 않고 readback만 한다.4 완료 후5에서 새 주소 trust/SSH를 검증한다.

4. 같은 private IP 행 → Edit → **Reserved public IP** → `ip-lab-reserved` 선택 → Update. 또는 reserved 상세에서 assign target을 실제 private IP OCID로 선택한다. VNIC OCID와 private IP OCID를 혼동하지 않는다. 예약 주소/OCID가 동일한지 다시 읽는다. [Assign task의 IP administration/Public IP type 필드](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/reserved-public-ip-assign.htm), [Ephemeral 삭제 task](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/deleting-ephemeral-public-ip-from-instance.htm).
5. GetPublicIp/GetVnic에서 같은 PRIVATE_IP_ID에 같은 reserved object/address를 읽고 로컬 `RESERVED_IP`에 전달한다. owner10 trust bootstrap의60초 StrictHostKeyChecking=ask 세션에서 **전환 전 동일 guest ED25519**를 비교하여 새 주소 known_hosts를 등록한다. 불일치는 BLOCKED_TRUST이고 keyscan만으로 우회하지 않는다. 이후 새 reserved 주소로 본인 local에서 SSH를 실제 수행한다. 추가 VM 재배정은 최소 비용 기본에서 필요하지 않다. 비교하려면 먼저 unassign하고 same reserved object가 AVAILABLE/unassigned가 된 것을 읽은 뒤 승인된 다른 private IP에 assign한다. reserved 수명과 ephemeral 수명을 구분한다.
6. 새로운 reserved public IP가 생성·할당돼야 primary 완료다. 화면만 비교하거나 vm-a의 ephemeral IP 관찰로 끝내지 않는다.

## 결과검증


### baseline-16-ready


```bash
oci network private-ip get --private-ip-id '<대상_primary_private_IP_OCID>' --region "$REGION" > ip-private.json || exit 1
oci network public-ip get --public-ip-id '<ip-lab-reserved_OCID>' --region "$REGION" > ip-reserved.json || exit 1
oci network vnic get --vnic-id '<대상_VNIC_OCID>' --region "$REGION" > ip-vnic.json || exit 1
# local 터미널에서
```

```bash
# 본인 local only:5의 same guest trust 등록 뒤; 개인 경로는 passphrase를 prompt에 입력한다.
export RESERVED_IP='<actual_reserved_IP_address>'
timeout --foreground -k 5s 30s ssh -i "$KEY" -o IdentitiesOnly=yes -o HostKeyAlgorithms=ssh-ed25519 -o StrictHostKeyChecking=yes -o ConnectTimeout=10 -o ConnectionAttempts=1 -o ControlMaster=no -o ControlPath=none opc@"$RESERVED_IP" 'id -un; hostname' || exit 1
```
기대 패턴 public-ip lifetime=RESERVED/scope=REGION/private-ip-id=대상 private IP OCID/ip-address=VNIC public-ip, 신규 SSH exit 0. unassign 후 주소/OCID 동일/private-ip-id=null. ephemeral 주소가 같은 값으로 되살아난다고 보장하지 않는다.

**증거 기록:** 실제 명령·UTC 시간·exit code·가려진 OCID·옵션 readback·검증 조건을 남긴다. 현재 이 문서의 실행상태는 `NOT_EXECUTED`다. output/creation-event-id/result는 실행 전 비워 둔다. 오류/NOT_SELECTED를 PASS로 바꾸지 않는다.

### 이 시나리오의 functional acceptance

| Positive | Negative / 미완료 판정 | Recovery / 반환 |
|---|---|---|
| reserved assignment/새 SSH | disposable IP No public 상태의 신규 SSH는 transport failure 예상(복구 Console 유지) | 같은 reserved object 할당 후 새 주소 SSH; 기존 ephemeral 값 복귀 보장 없음 |

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

Reserved IP assign 실패면 private IP에 기존 public IP가 아직 붙어 있는지, public subnet인지, Region이 같은지, 올바른 private IP OCID인지 확인한다. ephemeral을 reserved로 직접 변환하는 기능은 없으며 새 주소가 선택될 수 있다.

## 리소스정리

예약 IP를 대상 private IP에서 먼저 Unassign한 뒤 Networking → Reserved Public IPs → 해당 OCID → Actions → **Terminate**(동등 Delete label인 경우 삭제)를 제출하고 get으로 없는지 확인한다. 그 후 VM branch에 따라 반환한다. **owner07 vm-b 재사용:** owner16은 VM/boot를 terminate하지 않는다. reserved 제거와 새 public-IP 상태를 owner07에게 반환하고 확인을 받은 뒤 [Fault Domains 정리](../01_Introduction_to_OCI/03_Fault_Domains.md#리소스정리)를 creator가 수행한다. **owner16 신규 ip-lab-vm:** 이번 Create의 exact instance/boot OCID만 Compute → Terminate에서 boot 영구삭제를 선택해 종료하고 TERMINATED/boot 잔존을 재조회한다. shared vm-a public IP/NSG는 유지한다. [Reserved unassign 공식 task](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/reserved-public-ip-unassign.htm) 및 [Delete/Terminate task](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/reserved-public-ip-delete.htm)의 메뉴를 따른다. reserved IP는 VM terminate 후에도 남을 수 있으므로 별도 조회/삭제가 필수다. 생성된 ephemeral IP는 VM/VNIC 수명과 연결되며 reserved처럼 별도 보존될 것이라 가정하지 않는다.


## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 아래 본문은 각 생성 필드·설정·검증·정리의 기술 근거이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [32] [Public IP Addresses — Public IP Addresses](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/managingpublicIPs.htm)
  - 확인 범위: public IP.

### 추가 실행 절차 근거 (본문 확인)

- [ip](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/managingpublicIPs.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [launch](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/launchinginstance.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [connect](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/connect-to-linux-instance.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [cloudshell](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/cloudshellgettingstarted.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.

- [추가 세부 task: ipcreate](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/reserved-public-ip-create.htm) — 실제 본문을 확인한 필드·API/권한·삭제 단계 근거.

- [추가 세부 task: ipassign](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/reserved-public-ip-assign.htm) — 실제 본문을 확인한 필드·API/권한·삭제 단계 근거.

- [추가 세부 task: ipunassign](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/reserved-public-ip-unassign.htm) — 실제 본문을 확인한 필드·API/권한·삭제 단계 근거.

- [추가 세부 task: ipdelete](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/reserved-public-ip-delete.htm) — 실제 본문을 확인한 필드·API/권한·삭제 단계 근거.

- [추가 세부 task: ephemeraldelete](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/deleting-ephemeral-public-ip-from-instance.htm) — 실제 본문을 확인한 필드·API/권한·삭제 단계 근거.
