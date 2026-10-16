> **문서 검토 완료 — OCI 실행 미검증**: 이 Burstable Instances 시나리오의 실행 절차와 정리 경로를 문서 범위에서 검토했습니다. 실제 OCI 생성·SSH·metric 관찰·삭제를 검증한 결과는 아닙니다.

# Burstable Instances

> 문서 승인 범위: **이 시나리오의 실행 가능한 절차에 대한 문서 검토 완료**. 권한·지원 shape·예산·trust·Monitoring 준비·복구·정리 gate는 실제 실행 전에 충족해야 합니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 전체 교육과정 승인이나 원격 게시 완료를 뜻하지 않습니다.

> 실행 절차 작성본 / OCI 미실행. Oracle 공식 본문 근거와 교육용 선택을 구분한다. secrets·실제 고객값·diagram 없음.

## 학습목표

- 지원되는 E4 Flex의 burstable-vm 한 개를 승인된 1 OCPU/2 GB·12.5% baseline으로 생성하고 shape-config를 조회한다.
- 새 SSH에서 15초 유한 CPU 작업을 수행하고 실제 metric 시간창을 기록한 뒤 전용 VM/boot를 즉시 종료한다. full burst 보장은 합격 기준이 아니다.

## 대응 Roadmap 항목

- 항목명: **Burstable Instances**
- Roadmap 경로: **Compute → Burstable Instances**
- Node ID: `sequence-2-2`
- 상위 항목: **Compute** (`sequence-2-0`)
- 문서 범위: 지원되는 E4 Flex의 burstable-vm 한 개를 승인된 1 OCPU/2 GB·12.5% baseline으로 생성하고 shape-config를 조회한다. 새 SSH에서 15초 유한 CPU 작업을 수행하고 실제 metric 시간창을 기록한 뒤 전용 VM/boot를 즉시 종료한다. full burst 보장은 합격 기준이 아니다. 실제 OCI 실행 결과가 아니라 이 절차의 완료 조건이다.
- 번호는 navigation 식별자이며 실행 순서는 사전준비의 단계별 의존성에 따른다.

## 사전준비

### 기준 계약과 실제 단계 선행조건

[공통 baseline-contract.json](../../../baseline-contract.json)의 Oracle Linux 9 / opc / CIDR / 비용 상한을 따른다. 문서 번호는 실행 순서가 아니다. **준비 station과 생성 이후 ready를 분리**한다. 아래 단계 외의 기능완료를 소급하여 요구하지 않는다.

- [10 Compute — 완료 선행](00_Compute.md#baseline-10-ready): **기존 vm-a RUNNING/새 SSH 완료**. owner10이 이번 gate 종료까지 같은 vm-a를 보존한다. Monitoring 준비는 Compute 완료 산출물이 아니라 아래 paid Create 이전 metric 준비 gate에서 별도로 확인한다. chart 없음/권한 오류면 추가 paid Create 금지; 신규 VM으로 preflight를 대신하지 않는다.

- [15 SSH Keys — 단계 링크](05_SSH_Keys.md#key-preparation): **Part A 전용 공개키**. 실제 출력/소비자/expiry를 전달하고 해당 owner가 이 lab 종료까지 보존한다.
- [23 Public Subnets — 단계 링크](../../Networking/Original-Provisional/03_Virtual_Cloud_Network_VCN/04_Public_Subnets.md#public-network-preparation): **단계 A 완료**. 실제 출력/소비자/expiry를 전달하고 해당 owner가 이 lab 종료까지 보존한다.

생성 전 조직 승인(identity/권한/예산/expiry/서비스 지원/limit/capacity)을 확인한다. 실제 OCID/IP는 아직 생성하지 않았으면 비워 두고 해당 Create/readback 뒤 채운다. 동일 creation-event-id/OCID는 한 번의 제출만 뜻한다. `INSTANCE_A_ID`는 owner10 vm-a의 OCID, `INSTANCE_B_ID`는 owner22 private-vm의 OCID이며 owner07 vm-b와 다르다. 실습에 없는 자원을 추가하지 않는다. 실제 checkpoint는 사용자 실행 전 비워 둔다.

### 입력값과 실행 위치

Console에서 선택한 값만 사용한다. `<...>`는 예시 OCID나 비밀값이 아니라 **사용자가 대체할 입력 자리**다. 대체하지 않은 채 명령을 실행하지 않는다. Console → 우측 상단 Cloud Shell/Code Editor → Cloud Shell을 열면 OCI CLI가 사전 인증된다. Region 변경 뒤에는 Shell을 다시 열거나 명령의 `--region`을 바꾼다. SSH는 Cloud Shell이 아니라 본인 Linux/WSL/macOS 터미널에서 수행한다. 공인 IP/32도 그 터미널의 출구 주소여야 한다.

```bash
# 아래는 입력/출력의 대입 형식이다. 표의 단계 전에 생성 후 ID/IP export 행을 실행하지 않는다.
 # 생성 후 검증용 ID/IP는 해당 creator의 실제 응답에서 넣는다.
export REGION='<actual_REGION>'
export INSTANCE_ID='<actual_INSTANCE_ID>'
export KEY='<actual_KEY>'
export PUBLIC_IP='<actual_PUBLIC_IP>'
oci --version
```

이 변수 중 해당 절차에서 사용하는 값만 채운다. CLI 조회의 `data`를 증거 파일에 저장하고 실제 시간·작업 OCID·선택값을 기록한다. API key/private key/passphrase/인증 토큰은 기록하지 않는다. Cloud Shell 설정파일이나 세션 토큰을 출력하지 않는다. `oci --version`을 실행하는 주제의 기대 패턴은 버전 문자열뿐이다. namespace는 Object Storage 주제에서 별도 조회하며 다른 주제에는 요구하지 않는다. [Cloud Shell 공식 본문](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/cloudshellgettingstarted.htm).

### Workshop 입력 계약 — 단계와 host

| 입력/출력 | 값 또는 실제 획득 경로 | 사용할 시점 | 실행 위치 | 선택 이유 |
|---|---|---|---|---|
| Console/설정 필드 | Name=burstable-vm; OL9 x86; 지원 E4 Flex 1OCPU/2GB; baseline 12.5%; On-demand; public-admin/nsg-admin/ephemeral/.pub/default boot | 생성 전 선택 → 생성 뒤 readback | 본문 Console/로컬 지정 | 이 topic의 최소 생성 범위 |
| `PUBLIC_IP` | 본문 실제 Create/readback 출력 | 생성 후/선행 완료 | 본문 지정 host | 해당 topic 대상만 지정 |
| `INSTANCE_ID` | owner12 burstable-vm Create/readback id | 생성 후/선행 완료 | Cloud Shell | 현재 문서 대상 OCID에 명시 대입 |
| `KEY` | owner15 Part A 전용 private key 절대경로; 내용 전송 금지 | 준비 완료 | 로컬 SSH만 | 해당 공개키와 대응 |
| `LAB_COMPARTMENT_ID` | owner00 Create 응답 OCID; topic00에서는 생성 뒤 | 선행 완료/생성 후 | Console/Cloud Shell | 정책/자원 scope |
| `REGION` | 승인된 기존 subscribed Region identifier | 생성 전 | Console/Cloud Shell | API endpoint/데이터 위치 |
| `VNIC_ID` | 해당 INSTANCE_ID의 vnic-attachments.json data[].vnic-id; primary VNIC 확인 | 생성 후 | Cloud Shell | 다른 VM VNIC 혼동 방지 |

생성 전에는 행의 승인값/선행 완료값만 지정하고, 생성 후 ID/IP는 해당 owner 응답으로 채운다. Cloud Shell과 로컬 SSH의 환경변수는 공유되지 않는다. 로컬에는 KEY와 접속 주소만 다시 지정하고 guest disk 명령은 로그인한 guest에서만 실행한다. `<...>`가 남으면 해당 명령은 실행하지 않는다.

## 권한

burstable-vm 한 세트의 LaunchInstance/GetInstance/VNIC 조회/Terminate와 Compute Instance Monitoring 읽기를 승인한다. read metrics는 oci_computeagent namespace만 승인하며 metric write/alarm/autoscaling은 필요 없다. agent 활성화·데이터 접근 gate를 통과하기 전 추가 paid Create를 금지한다.

### 실제 operation과 공식 권한

- `GetInstance`: INSTANCE_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `GetVnic`: VNIC_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `LaunchInstance`: INSTANCE_CREATE and INSTANCE_IMAGE_READ and VNIC_CREATE and VNIC_ATTACH and SUBNET_ATTACH If putting the instance in a network security group during instance creation, also need NETWORK_SECURITY_GROUP _UPDATE_MEMBERS and VNIC_ASSOCIATE _NETWORK_SECURITY_GROUP If creating an instance in a compute cluster, also need COMPUTE_CLUSTER_LAUNCH_INSTANCE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `ListVnicAttachments`: VNIC_ATTACHMENT_READ and INSTANCE_INSPECT — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `TerminateInstance`: INSTANCE_DELETE and VNIC_DELETE and SUBNET_DETACH If volumes are attached, also need VOLUME_ATTACHMENT_DELETE and VOLUME_WRITE and INSTANCE_DETACH_VOLUME — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).

위 permission 이름은 IAM policy resource-type이 아니다. 조직 관리자가 기존 group/compartment scope와 conditional dependency를 실제 API 표에 대조한다. 편의 family grant를 최소권한 정책으로 부르지 않으며 실습자는 policy 작성 권한을 자동 인수하지 않는다.

**Boot attachment 조회 범위:** 이 문서는 미확인 `ListBootVolumeAttachments` 최소권한을 요구하는 CLI를 실행하지 않는다. 생성 owner가 승인된 Compute Console → 정확한 instance OCID → Storage → Boot volume에서 연결된 boot OCID/AD와 instance를 읽어 전달한다. boot를 검증하는 owner13만 GetBootVolume 권한을 승인받아 그 OCID를 조회한다. Console attachment를 볼 권한이 없으면 creator가 exact instance/boot 관계를 제공하기 전 중단한다. 권한 표에 없는 operation을 최소권한 검증 완료로 표시하지 않는다.

최초 SSH에 필요한 exact-instance Console connection/history는 관리자에게 해당 Create/Get/Delete/Capture 작업을 별도 승인받는다. [Compute의 독립 guest 신뢰 경로](00_Compute.md#guest-trust-bootstrap)의 BLOCKED_TRUST/BLOCKED_IAM gate를 보존한다. 승인 없이 public host fingerprint를 추정하거나 guest password를 만들지 않는다.

Monitoring의 ListMetrics=METRIC_INSPECT, SummarizeMetricsData=METRIC_INSPECT+METRIC_READ를 승인하고 namespace oci_computeagent read만 허용한다. [공식 Monitoring 권한 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/monitoringpolicyreference.htm). 실습절차의 paid Create 이전 metric gate가 실패하면 생성하지 않는다.

## 비용및주의사항

- 선택한 burstable-vm 1대(E4 Flex 1 OCPU/2 GB, baseline 12.5%)와 default boot/ephemeral IP의 짧은 사용시간만 별도 승인한다. vm-a 외 다른 extra VM이 남아 있으면 Create하지 않는다. A1에 Burstable 지원을 추정하지 않는다.
- CPU 작업은 15초 한 번이다. metric 대기는 최대10분 또는 expiry 중 이른 시각까지이며 빈 chart를 이유로 반복 launch/장시간 부하하지 않는다. 검증 또는 NOT_OBSERVED 기록 뒤 owner12가 전용 instance/boot를 즉시 종료·삭제 확인한다.

실제 OCI 작업은 아직 수행하지 않았다. 기대값은 실행 시 대조할 조건이며 무료 사용이나 과금 강제 차단을 보장하지 않는다. 승인 expiry가 끝나면 새 시험을 중단하고 위 소유권 순서로 원복·반환한다.

## 실습구성

### 이번 Workshop에서 만드는 것과 진행 순서

public-admin에 burstable-vm 한 세트만 잠깐 생성하고 15초 CPU 작업과 metric 관찰을 분리한다. burst 최고성능은 보장하지 않는다.

1. **진입/선택:** 지원 E4/baseline/예산 확인. 사전준비 링크의 지정 단계/실제 출력이 없으면 제출하지 않는다.
2. **실제 작업:** 1 OCPU/2 GB/12.5% Create → API baseline/SSH → 유한 task/UTC metric. 아래 Console 필드/명령을 해당 host에서 순서대로 실행한다. 같은 자원은 이름이 아니라 OCID/event로 재사용한다.
3. **완료/복구/반환:** 종료. 결과검증 실패는 문제해결로 이동하고 동일 Create를 무작정 재제출하지 않는다.

**전달/정리 범위:** 신규 burstable-vm/boot만 owner12 삭제; vm-a 유지. 아래 정리 단계는 이 소유권을 넘어서 실행하지 않는다.


- **이번 생성/재사용/출력:** 신규 burstable-vm 1세트. 기존 public-admin/NSG 재사용. 최대 1 extra VM·짧은 metrics 관찰 후 삭제.
- **생성 목표:** 승인된 지원 shape의 실제 burstable VM을 생성해 baseline을 확인하고 짧은 부하/idle metrics를 기록한다. 성능 보장 실험이 아니라 creation options lab이다.

## 실습절차

### 명령 실행 gate — 실패하면 다음 생성으로 진행하지 않는다

1. 아래 Bash는 **전용 Bash 세션**에서만 사용한다. Console/Cloud Shell 조회, 본인 로컬 SSH, 로그인한 Oracle Linux guest를 섞지 않는다. `exit 1`은 이 전용 세션을 닫는 안전 중단이다. `<...>`·빈 입력·타 owner OCID가 남으면 실행하지 않는다.
2. 성공을 요구하는 파일 작성/조회/설치/attach 작업은 실패 시 중단한다. 실패한 API 뒤 기존 JSON이나 기존 다운로드 파일을 성공 근거로 읽지 않는다. 기대 deny는 `if command; then ...; else RC=$?; ...; fi`로 별도 기록하고, 실제 timeout/IAM 오류/로컬 파일 오류를 구별한다.
3. 첫 변경 전에 정확한 OCID·owner·현재 설정과 승인 expiry를 저장한다. 공유 자원 변경 중 다른 소비자가 시작하거나 원복값을 확인할 수 없으면 변경을 중단한다. 삭제 성공은 API 수락이 아니라 같은 identity/Region의 정확한 대상 재조회로 판정한다. `NotAuthorizedOrNotFound` 단독은 삭제 증거가 아니다.


**최대 두 VM 실행 gate:** vm-a 외 vm-b/burstable-vm/ip-lab-vm/preemptible-vm/private-vm는 같은 시간에 한 개만 실행한다. 이전 extra VM/boot 정리를 readback한 후 다음 extra를 생성한다. owner07 vm-b를 owner16이 쓰는 경우에는 새로운 VM을 만들지 않고 09 검증을 먼저 끝낸다.

### 개별 완주 보정 — 생성·검증·소유권

INSTANCE_ID/PUBLIC_IP를 신규 burstable-vm에 지정한다. metrics 조회 권한·agent·수집시각을 따로 기록한다. 최대2VM 조건으로 private-vm와 겹치지 않게 실행하고 신규burst/boot만 terminate한다.

### paid Create 이전 metric 준비 gate

관리자가 `Allow group <domain>/<lab-group> to read metrics in compartment lab-root where target.metrics.namespace='oci_computeagent'`를 승인한다. 공식 [Monitoring permission table](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/monitoringpolicyreference.htm)의 ListMetrics=METRIC_INSPECT, SummarizeMetricsData=METRIC_INSPECT+METRIC_READ다. alarm 생성/metric write는 필요 없다. 기존 vm-a의 Compute → Metrics → namespace `oci_computeagent`에 실제 데이터가 표시되는지 확인하여 read 접근/공유 public path를 사전 점검한다. unauthorized면 paid Create를 금지한다. 새 instance 데이터 존재를 생성 전 확인했다고 쓰지 않는다.

[Enabling Monitoring](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/enablingmonitoring.htm)에 따라 새 launch의 Oracle Cloud Agent 탭에서 **Compute Instance Monitoring Enabled**, plugin 활성화를 선택한다. running 상태 확인은 생성 후에만 수행한다. 지원 OL9 platform image/agent 설치, public IP+IGW route와 승인 TCP443 egress가 필요하다. agent 설치가 빠진 custom/old image는 사용하지 않는다. RUNNING 후 같은 instance Oracle Cloud Agent 탭의 plugin enabled/running과 Metrics `oci_computeagent`/CpuUtilization을 대조한다. 작업 전후 UTC 창/1분 interval을 저장하고 30초 간격 **최대 10분 또는 승인 expiry 중 먼저 오는 시점**까지 조회한다. 빈 chart는 NOT_OBSERVED; 권한/API 오류는 BLOCKED다. 짧은 spike가 안 보인 것과 수집 자체가 없는 것을 구분하고 추가 장시간 부하·VM 재생성 없이 cleanup한다.

1. 현재 공식 supported shapes와 Console의 Burstable 옵션을 확인한다. baseline은 **12.5% 또는 50%** 이며 A1을 지원 shape로 추정하지 않는다. 비용 승인된 **VM.Standard.E4.Flex, total 1 OCPU, memory 2 GB, baseline 12.5%** 를 primary로 쓴다. 이 값/shape가 현재 UI에서 지원되지 않으면 제출을 중단하고 compatible supported shape를 재승인받는다. [Burstable Instances — baseline, supported shapes, limitations](https://docs.oracle.com/en-us/iaas/Content/Compute/References/burstable-instances.htm).
Create 제출 전에는 아래 두 항목을 준비한다. 먼저 [owner17 원본 YAML](07_Cloud_init_User_Data.md#user-data-preparation)의 public-host-key 출력을 제출한다. 또한 Create 실패·부분 생성·trust/SSH/metric 실패 또는 expiry 도달 시 재생성하지 않고 아래 리소스정리로 이동할 것을 확인한다. 정리에 필요한 신규 instance/boot OCID·AD는 생성 직후 기록한다.

2. Compute → Instances → Create: Name `burstable-vm`, lab-root, 승인 AD/FD, Oracle Linux x86-compatible image, On-demand, public-admin/nsg-admin, ephemeral IP, `.pub`, image-default boot. Image and shape → Change shape → E4 Flex → **Burstable** 선택 → total OCPU 1/memory 2/baseline 12.5%. 나머지 값은 공유 launch recipe와 같게 한 뒤 Create 제출한다. option이 launch UI의 다른 위치에 있으면 공식 body의 동등 항목을 사용하고 actual field 기록한다.


Create 성공 직후, 첫 SSH와 CPU 작업 **전에**, 아래 “Create 이후 값 전달”을 수행하고 Cloud Shell에서 결과검증의 세 조회 명령으로 RUNNING/shape-config/primary VNIC를 확인한다. 그 실제 public-ip를 로컬 PUBLIC_IP에 넣는다. RUNNING 뒤 첫 SSH 전에 [대상별 trust bootstrap](00_Compute.md#guest-trust-bootstrap)을 신규 burstable-vm의 **INSTANCE_ID**로 완료한다. 실제 OCID/UTC/ED25519 handoff 없으면 BLOCKED_TRUST이며 INSTANCE_A_ID의 신뢰값을 쓰지 않는다. 로컬 전용 Bash에 KEY/PUBLIC_IP를 이 guest 값으로 다시 지정한다.

3. RUNNING/SSH 후 instance shape-config의 baseline 설정을 조회한다. below-baseline idle와 아주 짧은 CPU 작업을 수행한다. 1시간 sustained 부하나 SLA 성능 시험은 하지 않는다.

```bash
# 로컬에서 guest에 유한 CPU task 실행
timeout --foreground 60s ssh -i "$KEY" -o IdentitiesOnly=yes -o HostKeyAlgorithms=ssh-ed25519 -o StrictHostKeyChecking=yes -o ConnectTimeout=10 -o ConnectionAttempts=1 -o ControlMaster=no -o ControlPath=none opc@"$PUBLIC_IP" python3 - <<'PY'
import time
end=time.monotonic()+15
x=0
while time.monotonic()<end:
    x=(x+1)%1000003
print('bounded CPU task complete')
PY
TASK_RC=$?
if [ "$TASK_RC" -ne 0 ]; then
    printf 'STOP: SSH/task exit=%s; diagnose then cleanup owned VM, do not repeat load\n' "$TASK_RC"
    exit 1
fi
```
SSH 또는 task가 실패하면 결과를 BLOCKED로 기록하고 문제해결 후 정리한다. 세션 중단은 VM 삭제가 아니므로 Console에서 신규 instance/boot 정리를 끝낸다.

4. Instance → Metrics → CPU Utilization에서 작업 전후 UTC 시간범위와 interval을 기록한다. 15초 task가 1분 평균에 희석될 수 있고 metric 수집에는 agent/시간 지연이 있을 수 있다. Bursting은 자동·자원상황 의존이며 필요한 시점에 full OCPU를 보장하지 않는다. metric 없음을 성공한 burst 측정으로 쓰지 않는다.
5. 종료 전에 아래 [결과검증](#baseline-12-ready)의 RUNNING/API 및 위 새 SSH/유한 task 결과를 대조하고 metric 결과 또는 NOT_OBSERVED를 기록한다. 그 뒤 [리소스정리](#리소스정리)에서 전용 burstable-vm/boot를 한 번만 종료한다. 이 단계에서 먼저 terminate하지 않는다.

### Create 이후 값 전달 — 이름으로 추정하지 않는다

Cloud Shell의 INSTANCE_ID는 해당 creator Create 응답 id로 지정한다. Console → Compute → Instances → 정확한 instance → Networking/Attached VNICs에서 primary VNIC OCID를 기록하고 아래 GetVnic의 `is-primary=true`, `subnet-id`, `private-ip`, `public-ip`를 대조한다. Storage/Boot volume의 연결 boot OCID·AD도 creator가 기록한다. Cloud Shell과 로컬 터미널의 변수는 공유되지 않으므로 SSH 전에 KEY와 **그 instance의** 실제 접속 주소를 로컬에 다시 지정한다. 이전 extra VM 주소를 재사용하지 않는다. 모든 JSON 조회는 실패하면 즉시 중단하고 기존 파일을 읽지 않는다.

## 결과검증


### baseline-12-ready


```bash
oci compute instance get --instance-id "$INSTANCE_ID" --region "$REGION" > instance-readback.json || exit 1
oci compute vnic-attachment list --compartment-id "$LAB_COMPARTMENT_ID" --instance-id "$INSTANCE_ID" --region "$REGION" > vnic-attachments.json || exit 1
# Cloud Shell: 현재 instance에 붙은 ATTACHED VNIC만 각각 조회한다.
export VNIC_ID='<현재_INSTANCE_ID에_ATTACHED이며_GetVnic_is-primary=true인_OCID>'
case "$VNIC_ID" in ocid1.vnic.*) ;; *) printf '%s\n' 'STOP: actual primary VNIC OCID required'; exit 1;; esac
oci network vnic get --vnic-id "$VNIC_ID" --region "$REGION" > vnic-readback.json || exit 1
```
기대 패턴: instance `lifecycle-state=RUNNING`, `availability-domain`/`fault-domain`/`shape`가 제출값과 같음, VNIC `subnet-id`가 선택값과 같음, public VM은 `public-ip`가 문자열이고 private VM은 null이다. creator Console Storage의 boot 연결이 대상 instance와 boot OCID를 연결한다. 파일을 실제로 읽어 조건을 대조한다.

기대 패턴 shape-config.ocpus=1/memory-in-gbs=2/baseline-ocpu-utilization=`BASELINE_1_8`(12.5%) 또는 실제 승인 50%의 `BASELINE_1_2`. 새 SSH exit 0/유한 task 종료. Metrics의 실제 time window와 CpuUtilization을 기록하되 100%에 도달해야 한다는 합격 기준을 쓰지 않는다.

**증거 기록:** 실제 명령·UTC 시간·exit code·가려진 OCID·옵션 readback·검증 조건을 남긴다. 현재 이 문서의 실행상태는 `NOT_EXECUTED`다. output/creation-event-id/result는 실행 전 비워 둔다. 오류/NOT_SELECTED를 PASS로 바꾸지 않는다.

### 이 시나리오의 functional acceptance

| Positive | Negative / 미완료 판정 | Recovery / 반환 |
|---|---|---|
| baseline flag/finite CPU task/metric window | unsupported Burst option/미수집 metric은 성공으로 기록 금지 | 현재 compatible option 재승인 또는 BLOCKED; 유한 task 종료 후 VM/boot cleanup |

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

Burstable 옵션이 없으면 지원 shape/total OCPU/baseline 조건을 확인한다. A1/무료 shape를 자동 대체하지 않는다. CpuUtilization 그래프에 spike가 없으면 수집 agent·interval/timezone·짧은 task 평균 희석을 확인하며 burst 실패라고 단정하지 않는다.

## 리소스정리

[공식 Terminating an Instance](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/terminatinginstance.htm): boot는 기본 보존이므로 아래 삭제 선택을 명시적으로 확인한다. Terminated 기록은 최소 12시간 목록에 남을 수 있으며 목록 행의 존재만으로 실행 중이라고 판정하지 않는다.

1. 신규 burstable-vm/boot만 owner12 삭제; vm-a 유지. 예약 소비자가 없거나 expiry가 끝나면 즉시 정리한다. 공유 vm-a/boot/public network는 삭제하지 않는다.
2. 대상 burstable-vm의 OCID와 생성 직후 기록한 boot OCID/AD를 확인한다. 이 실습은 추가 mount/volume을 만들지 않으며 공유 자원을 unmount/detach하지 않는다. task 실패나 연결 불가여도 정리를 미루지 말고 Console에서 정확한 신규 instance를 확인한다. Compute → 대상 → Terminate에서 Permanently delete attached boot volume을 선택한다. 정확한 instance TERMINATED와 기록한 boot의 잔존 여부를 Console에서 다시 확인하며 권한 오류를 삭제 성공으로 처리하지 않는다. boot가 남으면 기록한 OCID/AD와 신규 VM의 연결 관계를 owner12가 재확인한 뒤 해당 boot만 삭제한다. 확인·삭제 권한이 없으면 BLOCKED_CLEANUP으로 관리자에게 그 OCID/AD를 전달하고 잔존 비용을 알린다. vm-a·네트워크·NSG·키는 각 creator에게 반환하며 이 문서에서 삭제하지 않는다.


## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 아래 본문은 각 생성 필드·설정·검증·정리의 기술 근거이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [11] [Burstable Instances — Burstable Instances](https://docs.oracle.com/en-us/iaas/Content/Compute/References/burstable-instances.htm)
  - 확인 범위: A burstable instance is.

### 추가 실행 절차 근거 (본문 확인)

- [burst](https://docs.oracle.com/en-us/iaas/Content/Compute/References/burstable-instances.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [launch](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/launchinginstance.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [connect](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/connect-to-linux-instance.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [cloudshell](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/cloudshellgettingstarted.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
