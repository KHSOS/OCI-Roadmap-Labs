> **문서 검토 완료 — OCI 실행 미검증**: 이 Capacity Types 실습의 한정된 절차·검증·정리 문서를 검토했습니다. 실제 OCI 실행이나 전체 교육과정 승인을 뜻하지 않습니다.

# Capacity Types

> 문서 승인 범위: **한정된 실행 절차의 문서 검토 완료**. preemptible 검증·종료 순서와 선택 reservation/host의 소비 VM 관계·정리 절차를 검토했습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 본문의 권한·신뢰·예산·지원·capacity·복구 gate는 실행 전에 충족해야 합니다. 전체 교육과정·게시 상태는 [통합 상태](../../../Docs/README.md)와 별도입니다.

> 실행 절차 작성본 / OCI 미실행. Oracle 공식 본문 근거와 교육용 선택을 구분한다. secrets·실제 고객값·diagram 없음.

## 학습목표

- 기존 on-demand vm-a 증거와 신규 preemptible-vm의 capacity type·boot deletion 설정을 readback하고 새 SSH를 연다.
- preemption을 기다리거나 stop/reboot하지 않고 terminate한다. 별도 승인 reservation/host는 실제 소비 VM 연결까지 검증하고 VM/boot부터 정리한다.

## 대응 Roadmap 항목

- 항목명: **Capacity Types**
- Roadmap 경로: **Compute → Capacity Types**
- Node ID: `sequence-2-8`
- 상위 항목: **Compute** (`sequence-2-0`)
- 문서 범위: 기존 on-demand vm-a 증거와 신규 preemptible-vm의 capacity type·boot deletion 설정을 readback하고 새 SSH를 연다. preemption을 기다리거나 stop/reboot하지 않고 terminate한다. 별도 승인 reservation/host는 실제 소비 VM 연결까지 검증하고 VM/boot부터 정리한다. 실제 OCI 실행 결과가 아니라 이 절차의 완료 조건이다.
- 번호는 navigation 식별자이며 실행 순서는 사전준비의 단계별 의존성에 따른다.

## 사전준비

### 기준 계약과 실제 단계 선행조건

[공통 baseline-contract.json](../../../baseline-contract.json)의 Oracle Linux 9 / opc / CIDR / 비용 상한을 따른다. 문서 번호는 실행 순서가 아니다. **준비 station과 생성 이후 ready를 분리**한다. 아래 단계 외의 기능완료를 소급하여 요구하지 않는다.

- [15 SSH Keys — 단계 링크](05_SSH_Keys.md#key-preparation): **Part A 키**. 실제 출력/소비자/expiry를 전달하고 해당 owner가 이 lab 종료까지 보존한다.
- [23 Public Subnets — 단계 링크](../../Networking/Original-Provisional/03_Virtual_Cloud_Network_VCN/04_Public_Subnets.md#public-network-preparation): **단계 A 완료**. 실제 출력/소비자/expiry를 전달하고 해당 owner가 이 lab 종료까지 보존한다.
- [10 Compute — 단계 링크](00_Compute.md#shared-launch): **on-demand 생성 증거; preemptible 생성에는 network/key 준비만 필요**. 실제 출력/소비자/expiry를 전달하고 해당 owner가 이 lab 종료까지 보존한다.

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
| Console/설정 필드 | Name=preemptible-vm; 지원 E4 Flex1/2/OL9 x86; Preemptible; boot deletion on preemption Yes; optional reservation/host separate approval | 생성 전 선택 → 생성 뒤 readback | 본문 Console/로컬 지정 | 이 topic의 최소 생성 범위 |
| `PUBLIC_IP` | 본문 실제 Create/readback 출력 | 생성 후/선행 완료 | 본문 지정 host | 해당 topic 대상만 지정 |
| `INSTANCE_ID` | 본문 실제 Create/readback 출력 | 생성 후/선행 완료 | 본문 지정 host | 해당 topic 대상만 지정 |
| `KEY` | owner15 Part A 전용 private key 절대경로; 내용 전송 금지 | 준비 완료 | 로컬 SSH만 | 해당 공개키와 대응 |
| `REGION` | 승인된 기존 subscribed Region identifier | 생성 전 | Console/Cloud Shell | API endpoint/데이터 위치 |

생성 전에는 행의 승인값/선행 완료값만 지정하고, 생성 후 ID/IP는 해당 owner 응답으로 채운다. Cloud Shell과 로컬 SSH의 환경변수는 공유되지 않는다. 로컬에는 KEY와 접속 주소만 다시 지정하고 guest disk 명령은 로그인한 guest에서만 실행한다. `<...>`가 남으면 해당 명령은 실행하지 않는다.

## 권한

기본 preemptible-vm launch/조회/terminate와 boot 정리만 승인한다. reservation 또는 dedicated host를 실제 선택할 때만 해당 Create/Get/Delete 및 소비 VM 연결 권한을 별도로 승인한다. tenancy default reservation 변경/광범위 quota 변경은 기본 범위가 아니다. reservation/host와 소비 VM의 Compartment가 다르면 각 scope 의존 권한을 재확인한다.

### 실제 operation과 공식 권한

- `CreateComputeCapacityReservation`: CAPACITY_RESERVATION_CREATE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `CreateDedicatedVmHost`: DEDICATED_VM_HOST_CREATE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `DeleteComputeCapacityReservation`: CAPACITY_RESERVATION_DELETE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `DeleteDedicatedVmHost`: DEDICATED_VM_HOST_DELETE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `GetComputeCapacityReservation`: CAPACITY_RESERVATION_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `GetDedicatedVmHost`: DEDICATED_VM_HOST_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `GetInstance`: INSTANCE_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `LaunchInstance`: INSTANCE_CREATE and INSTANCE_IMAGE_READ and VNIC_CREATE and VNIC_ATTACH and SUBNET_ATTACH If putting the instance in a network security group during instance creation, also need NETWORK_SECURITY_GROUP _UPDATE_MEMBERS and VNIC_ASSOCIATE _NETWORK_SECURITY_GROUP If creating an instance in a compute cluster, also need COMPUTE_CLUSTER_LAUNCH_INSTANCE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- **Dedicated host 선택 확장에만 해당** `LaunchInstance`: DEDICATED_VM_HOST_LAUNCH_INSTANCE in dedicated virtual machine host compartment INSTANCE_CREATE in compartment for the instance launched on the dedicated virtual machine host — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `TerminateInstance`: INSTANCE_DELETE and VNIC_DELETE and SUBNET_DETACH If volumes are attached, also need VOLUME_ATTACHMENT_DELETE and VOLUME_WRITE and INSTANCE_DETACH_VOLUME — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).

위 permission 이름은 IAM policy resource-type이 아니다. 조직 관리자가 기존 group/compartment scope와 conditional dependency를 실제 API 표에 대조한다. 편의 family grant를 최소권한 정책으로 부르지 않으며 실습자는 policy 작성 권한을 자동 인수하지 않는다.

최초 SSH에 필요한 exact-instance Console connection/history는 관리자에게 해당 Create/Get/Delete/Capture 작업을 별도 승인받는다. [Compute의 독립 guest 신뢰 경로](00_Compute.md#guest-trust-bootstrap)의 BLOCKED_TRUST/BLOCKED_IAM gate를 보존한다. 승인 없이 public host fingerprint를 추정하거나 guest password를 만들지 않는다.

## 비용및주의사항

- 기본 추가 자원은 짧은 preemptible-vm/boot 1세트다. vm-a의 On-demand 증거를 재사용하고 extra VM은 전역 한 개 상한이다. 회수 시 boot 영구삭제 선택과 실제 종료 후 잔존을 확인한다.
- reservation count1와 dedicated host1은 각각 별도 높은 비용/shape/capacity/expiry 승인이 필요하다. idle reservation/host도 VM 수0으로 비용 종료를 판단하지 않는다. preemptible→VM/boot 정리 확인→예약 branch→VM/boot와 예약 정리 확인→host branch 순차 진행하며 미선택은 NOT_SELECTED다.

실제 OCI 작업은 아직 수행하지 않았다. 기대값은 실행 시 대조할 조건이며 무료 사용이나 과금 강제 차단을 보장하지 않는다. 승인 expiry가 끝나면 새 시험을 중단하고 위 소유권 순서로 원복·반환한다.

## 실습구성

### 이번 Workshop에서 만드는 것과 진행 순서

공유 on-demand 증거와 짧은 preemptible 생성/종료를 비교한다. reservation/host는 비용 승인된 선택 트랙으로 기본 모델에 넣지 않는다.

1. **진입/선택:** on-demand 선택 증거. 사전준비 링크의 지정 단계/실제 출력이 없으면 제출하지 않는다.
2. **실제 작업:** 지원 preemptible/boot 삭제 설정 → Create/Get/SSH → Terminate. 아래 Console 필드/명령을 해당 host에서 순서대로 실행한다. 같은 자원은 이름이 아니라 OCID/event로 재사용한다.
3. **완료/복구/반환:** 승인 선택 트랙만 별도 실행. 결과검증 실패는 문제해결로 이동하고 동일 Create를 무작정 재제출하지 않는다.

**전달/정리 범위:** preemptible/boot owner18; optional VM/boot→reservation 또는 host 순서. 아래 정리 단계는 이 소유권을 넘어서 실행하지 않는다.


- **이번 생성/재사용/출력:** 기본 신규 preemptible-vm 1세트, on-demand vm-a 재사용. optional reservation 1 count + consumer 1 또는 dedicated host 1 + compatible consumer 1. 동시에 실행하지 않는다.
- **생성 목표:** 저비용 실제 creation으로 capacity 선택 목표를 달성: 10 on-demand VM 생성과 지원 smallest-fit preemptible VM의 짧은 순차생성·즉시정리를 기본으로 한다. Reservation·Dedicated Host는 핵심 선택 차이를 creation-time 비교한 후, 실제 생성 경험이 꼭 필요한 경우에만 명시적 선택 확장으로 승인받아 실행한다. 미실행 확장은 실패나 전체기본실습 미완료로 간주하지 않는다.

## 실습절차

### 명령 실행 gate — 실패하면 다음 생성으로 진행하지 않는다

1. 아래 Bash는 **전용 Bash 세션**에서만 사용한다. Console/Cloud Shell 조회, 본인 로컬 SSH, 로그인한 Oracle Linux guest를 섞지 않는다. `exit 1`은 이 전용 세션을 닫는 안전 중단이다. `<...>`·빈 입력·타 owner OCID가 남으면 실행하지 않는다.
2. 성공을 요구하는 파일 작성/조회/설치/attach 작업은 실패 시 중단한다. 실패한 API 뒤 기존 JSON이나 기존 다운로드 파일을 성공 근거로 읽지 않는다. 기대 deny는 `if command; then ...; else RC=$?; ...; fi`로 별도 기록하고, 실제 timeout/IAM 오류/로컬 파일 오류를 구별한다.
3. 첫 변경 전에 정확한 OCID·owner·현재 설정과 승인 expiry를 저장한다. 공유 자원 변경 중 다른 소비자가 시작하거나 원복값을 확인할 수 없으면 변경을 중단한다. 삭제 성공은 API 수락이 아니라 같은 identity/Region의 정확한 대상 재조회로 판정한다. `NotAuthorizedOrNotFound` 단독은 삭제 증거가 아니다.


**최대 두 VM 실행 gate:** vm-a 외 vm-b/burstable-vm/ip-lab-vm/preemptible-vm/private-vm는 같은 시간에 한 개만 실행한다. 이전 extra VM/boot 정리를 readback한 후 다음 extra를 생성한다. owner07 vm-b를 owner16이 쓰는 경우에는 새로운 VM을 만들지 않고 09 검증을 먼저 끝낸다.

### 개별 완주 보정 — 생성·검증·소유권

대상INSTANCE_ID는 신규preemptible에 고정한다. 기본과 optionalget는 각각 선택 후 따로 실행한다. optionalcost미승인/지원shape미확인은 NOT_SELECTED/BLOCKED이며 신규VM/boot→예약/host 순서로 삭제한다.

첫 SSH 전에 모든 신규 guest는 [owner17 원본 YAML](07_Cloud_init_User_Data.md#user-data-preparation)의 public-host-key 출력을 Create에 넣고 [대상별 trust bootstrap](00_Compute.md#guest-trust-bootstrap)을 완료한다. 기본 preemptible은 INSTANCE_ID, 선택 reserved-vm은 RESERVED_INSTANCE_ID, dedicated-vm은 해당 실제 consumer OCID로 각각 OCID/UTC/ED25519 handoff를 얻는다. 각 local KEY/PUBLIC_IP를 creator 실제 응답으로 다시 지정하고 bounded 60초·비다중화·StrictHostKeyChecking=yes 새 SSH만 사용한다. handoff 없음/다른 guest fingerprint면 BLOCKED_TRUST; 기존 vm-a 신뢰값 재사용 금지.

### 기본 — on-demand와 preemptible 실제 생성

1. Compute의 same vm-a launch에서 On-demand를 선택한 증거를 확보한다. capacity를 보장하는 reservation과 달리 현재 가용 capacity에 의존한다. 기존 vm-a는 공유하고 중복하지 않는다.
2. preemptible 작은 VM의 짧은 실행 예산/boot/전송 비용을 승인한다. Compute → Instances → Create: `preemptible-vm`, lab-root, 승인 AD/FD, compatible Oracle Linux x86 image, **VM.Standard.E4.Flex 1 OCPU/2 GB**(현재 supported shape인 경우), public-admin/nsg-admin/.pub/default boot. Placement → Advanced options → Capacity type **Preemptible capacity**. **Permanently delete attached boot volume on preemption = Yes**. 선택 후 Create를 제출한다. [Creating a preemptible instance, limitations](https://docs.oracle.com/en-us/iaas/Content/Compute/Concepts/preemptible.htm).
3. RUNNING과 preemptible config/boot delete option을 readback한다. 새 SSH로 `hostname; id -un`을 확인한다. preemption은 provider가 회수하는 것이며 실습자가 억지로 기다리지 않는다. preemptible은 start/stop/reboot가 제한되고 on-demand로 convert하지 못한다. Terminate 전 [baseline-18-ready](#baseline-18-ready)의 API/새 SSH 검증을 먼저 완료하고 UTC·exit·JSON을 저장한다. 검증 후 아래 비용/수량 gate에서 한 번만 **Terminate**한다.

### 비용/수량 gate — optional은 반드시 순차로 진행한다

preemptible 검증이 끝나면 먼저 해당 instance/boot를 Terminate하고 exact OCID의 TERMINATED/boot 잔존을 readback한다. 그 뒤에만 reservation/소비 VM branch를 시작한다. reservation branch 종료 시 reserved-vm/boot→reservation의 삭제/잔존을 읽은 후에만 dedicated host branch를 시작한다. 기본 vm-a 외 **동시에 한 개의 extra VM** 상한은 optional에도 적용한다. running instance 전체를 Console Compartment 목록에서 조회하여 다른 extra VM이 남았으면 제출하지 않는다. reservations/hosts 자체는 VM count가0이어도 비용이 남을 수 있으므로 별도 expiry를 지킨다.

### 선택 확장 A — 실제 reservation과 소비 VM (별도 승인)

4. 기본 수료에 필수는 아니며 idle reserved capacity도 과금이 있을 수 있다. approval 없는 상태는 NOT_SELECTED다. Compute → Capacity Reservations → Create capacity reservation. Basic: Name `lab-reservation`, Compartment lab-root, AD=`<승인_AD>`, **Make default for this AD Off**. Next. Capacity config: FD First available(또는 승인 exact FD), shape 지원 small E4 Flex, Cores 1, Memory 2 GB, Count **1**, 추가 shape 없음. Review/Create. [Creating a Capacity Reservation — partial/zero allocation and default consequences](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/capacity-reservations-create_tabs.htm).
5. Work request가 끝나도 capacity가 실제 **1** 확보됐는지 상세의 reserved/available/used count를 읽는다. count=0이면 capacity 확보 성공이 아니며 소비 VM을 만들지 말고 reservation 삭제로 끝낸다.
Cloud Shell에서 같은 예약을 조회하고 **생성 전/소비 후/반환 후** 세 상태를 분리한다. [Oracle InstanceReservationConfig 모델](https://docs.oracle.com/en-us/iaas/tools/python/latest/api/core/models/oci.core.models.InstanceReservationConfig.html)은 reserved_count=총 수용량, used_count=사용 중 수량으로 정의한다. [ComputeCapacityReservation 모델](https://docs.oracle.com/en-us/iaas/tools/python/latest/api/core/models/oci.core.models.ComputeCapacityReservation.html)의 configs/AD/id와 CLI kebab-case JSON을 아래대로 비교한다.

```bash
# Cloud Shell: optional reservation을 실제 생성한 경우만
export RESERVATION_ID='<actual_lab-reservation_OCID>'
# Create 전에 승인/기록한 선택 그대로; First available만 NONE, exact 선택은 FAULT-DOMAIN-1/2/3
export APPROVED_RESERVATION_FD='<NONE_or_exact_approved_FD>'
oci compute capacity-reservation get --capacity-reservation-id "$RESERVATION_ID" --region "$REGION" > reservation-before.json || exit 1
python3 - <<'PY'
import json,os
r=json.load(open('reservation-before.json'))['data']
assert r['id']==os.environ['RESERVATION_ID'] and r['lifecycle-state']=='ACTIVE'
assert r['is-default-reservation'] is False
cfg=r['instance-reservation-configs']; assert len(cfg)==1
c=cfg[0]
assert c['instance-shape']=='VM.Standard.E4.Flex'
assert c['instance-shape-config']['ocpus']==1 and c['instance-shape-config']['memory-in-gbs']==2
assert c['reserved-count']==1 and c['used-count']==0
selected=os.environ['APPROVED_RESERVATION_FD']
assert selected in ('NONE','FAULT-DOMAIN-1','FAULT-DOMAIN-2','FAULT-DOMAIN-3')
expected_fd=None if selected=='NONE' else selected
assert c['fault-domain']==expected_fd, 'BLOCKED: reservation FD differs from pre-Create approval'
print('one actual unused reserved slot ready')
PY
```
현재 승인 shape가 E4가 아니면 Create 이전 선택·assertion을 함께 재승인한다; count0/부분할당/미확인 FD는 소비 VM 생성 gate 실패다. 아래 6단계 launch의 exact AD/shape/OCPU/memory/FD가 config와 일치해야 한다. 6. actual count 1이면 Compute Create에서 `reserved-vm`, 같은 AD/shape/OCPU/memory/FD 조건, public-admin/.pub/NSG/default boot, Capacity type **Capacity reservation** → lab-reservation을 선택하고 제출한다. RUNNING/SSH와 reservation 사용 count 증가를 확인한다. 종료 시 reserved-vm/boot 먼저, reservation 두 번째로 삭제한다. default reservation은 만들지 않는다.

RUNNING 뒤 다음을 읽는다:

```bash
export RESERVED_INSTANCE_ID='<actual_reserved-vm_OCID>'
oci compute instance get --instance-id "$RESERVED_INSTANCE_ID" --region "$REGION" > reserved-instance.json || exit 1
oci compute capacity-reservation get --capacity-reservation-id "$RESERVATION_ID" --region "$REGION" > reservation-used.json || exit 1
python3 - <<'PY'
import json,os
r=json.load(open('reservation-used.json'))['data']; v=json.load(open('reserved-instance.json'))['data']
assert r['id']==v['capacity-reservation-id']==os.environ['RESERVATION_ID']
assert v['id']==os.environ['RESERVED_INSTANCE_ID'] and v['lifecycle-state']=='RUNNING'
assert v['availability-domain']==r['availability-domain']
c=r['instance-reservation-configs'][0]
assert c['reserved-count']==c['used-count']==1
assert v['shape']==c['instance-shape']
assert v['shape-config']['ocpus']==c['instance-shape-config']['ocpus']
assert v['shape-config']['memory-in-gbs']==c['instance-shape-config']['memory-in-gbs']
selected=os.environ['APPROVED_RESERVATION_FD']
assert selected in ('NONE','FAULT-DOMAIN-1','FAULT-DOMAIN-2','FAULT-DOMAIN-3')
expected_fd=None if selected=='NONE' else selected
assert c['fault-domain']==expected_fd, 'BLOCKED: reservation FD differs from pre-Create approval'
assert expected_fd is None or v['fault-domain']==expected_fd
print('same reservation consumer relationship verified')
PY
```
소비 VM/boot termination을 exact readback한 뒤 reservation Get을 다시 성공 조회해 `used-count=0`을 확인하고 reservation만 Delete한다. 반환 대기는 30초 간격 최대5분/expiry 안으로 한정하며 timeout이면 evidence 보존/BLOCKED; default flag를 바꾸거나 추가 VM을 만들지 않는다.

### 선택 확장 B — dedicated host 1개와 compatible VM (높은 비용 승인)

7. Compute → Dedicated Virtual Machine Hosts → Create. Name `lab-dedicated-host`, lab-root, 승인 AD/FD, **현재 supported host shape 중 승인된 최소 적합 shape**를 선택한다. UI의 host shape는 VM shape가 아니며 비용이 크게 다를 수 있다. host shape/시간 cap을 승인하지 못하면 제출 금지. Create 후 ACTIVE까지 기다린다(공식 본문은 최대 약 15분 언급). [Creating a Dedicated VM Host — AD/FD/ACTIVE](https://docs.oracle.com/en-us/iaas/Content/Compute/Concepts/dedicatedvmhosts_topic-Creating_a_Dedicated_Virtual_Machine_Host.htm).
8. Compute Create에서 Name `dedicated-vm`, Capacity type **Dedicated host** → lab-dedicated-host, host와 같은 AD/FD, **host에서 지원하는 VM shape**/최소 승인 OCPU/memory, compatible image/public-admin/key/NSG/default boot를 선택한다. RUNNING/SSH와 dedicated-vm-host-id를 확인한다. 삭제는 VM/boot → host 순이다.
Cloud Shell에서 dedicated consumer가 RUNNING일 때 다음 관계를 확인한 후 VM/boot를 종료한다. 두 ID는 이번 Create의 실제 응답에서만 채운다.

```bash
export DEDICATED_HOST_ID='<actual_lab-dedicated-host_OCID>'
export DEDICATED_INSTANCE_ID='<actual_dedicated-vm_OCID>'
oci compute dedicated-vm-host get --dedicated-vm-host-id "$DEDICATED_HOST_ID" --region "$REGION" > dedicated-host.json || exit 1
oci compute instance get --instance-id "$DEDICATED_INSTANCE_ID" --region "$REGION" > dedicated-instance.json || exit 1
python3 - <<'VERIFY'
import json,os
h=json.load(open('dedicated-host.json'))['data']; v=json.load(open('dedicated-instance.json'))['data']
assert h['id']==os.environ['DEDICATED_HOST_ID'] and h['lifecycle-state']=='ACTIVE'
assert v['id']==os.environ['DEDICATED_INSTANCE_ID'] and v['lifecycle-state']=='RUNNING'
assert v['dedicated-vm-host-id']==h['id']
assert v['availability-domain']==h['availability-domain']
assert v['fault-domain']==h['fault-domain']
print('same dedicated host consumer AD/FD verified')
VERIFY
[ "$?" = 0 ] || exit 1
```

9. reservation/host를 생성했다는 사실과 실제 instance가 그 capacity를 사용했다는 사실을 분리한다. 전용 host를 private cloud와 같다고 쓰지 않는다. optional host shape 자동 추정이나 tenancy default 변경을 하지 않는다.

## 결과검증


### baseline-18-ready

이 station은 기본 단계3에서 **Terminate 전에** 실행한다. optional branch 이후에는 저장한 preemptible.json·SSH exit/UTC 증거를 읽으며 종료한 VM에 다시 SSH하지 않는다.


```bash
oci compute instance get --instance-id '<preemptible-vm_OCID>' --region "$REGION" > preemptible.json || exit 1
```

```bash
# 본인 Linux/WSL 로컬 전용 Bash; Cloud Shell API fence와 분리
export KEY='<owner15_actual_local_KEY>'
export PUBLIC_IP='<INSTANCE_ID_actual_preemptible_public_IP>'
timeout --foreground 60s ssh -i "$KEY" -o IdentitiesOnly=yes -o HostKeyAlgorithms=ssh-ed25519 -o StrictHostKeyChecking=yes -o ConnectTimeout=10 -o ConnectionAttempts=1 -o ControlMaster=no -o ControlPath=none opc@"$PUBLIC_IP" 'hostname; id -un' || exit 1
# 선택 reservation 실행 때만
# 승인/실제 생성 뒤 별도 실행: oci compute capacity-reservation get --capacity-reservation-id '<lab-reservation_OCID>' --region "$REGION" > reservation.json
# 선택 dedicated 실행 때만
# 승인/실제 생성 뒤 별도 실행: oci compute dedicated-vm-host get --dedicated-vm-host-id '<host_OCID>' --region "$REGION" > dedicated-host.json
```
기대 패턴 preemptible config non-null/boot preserve=false/RUNNING/SSH exit 0; preemption 실제 발생은 미요구. 예약은 아래 verified model의 `instance-reservation-configs[].reserved-count=1`, 소비 전 `used-count=0`/소비 후 `used-count=1` 및 instance의 `capacity-reservation-id` exact 일치, dedicated host ACTIVE/instance dedicated-vm-host-id 일치. optional 미생성이면 JSON output를 꾸미지 않고 NOT_SELECTED.

**증거 기록:** 실제 명령·UTC 시간·exit code·가려진 OCID·옵션 readback·검증 조건을 남긴다. 현재 이 문서의 실행상태는 `NOT_EXECUTED`다. output/creation-event-id/result는 실행 전 비워 둔다. 오류/NOT_SELECTED를 PASS로 바꾸지 않는다.

### 이 시나리오의 functional acceptance

| Positive | Negative / 미완료 판정 | Recovery / 반환 |
|---|---|---|
| preemptible config/새 SSH | stop/start/reboot 지원 안 됨; 일부러 금지 API 호출 안 함 | Terminate로 정상 종료; capacity 실패는 승인 가능한 작은 지원값 또는 BLOCKED |

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

reservation work request 성공이어도 reserved count=0이면 actual capacity 확보 실패다. preemptible에 stop/reboot를 요청하지 않는다. dedicated host ACTIVE 전에는 VM launch를 제출하지 않는다.

## 리소스정리

1. preemptible/boot owner18; optional VM/boot→reservation 또는 host 순서. 예약 소비자가 없거나 expiry가 끝나면 즉시 정리한다. 공유 vm-a/boot/public network는 삭제하지 않는다.
2. 대상 preemptible-vm의 OCID와 attachment를 확인하고 guest workload를 종료한다. mount가 있으면 unmount/detach한다. Compute → 대상 → Terminate에서 Permanently delete attached boot volume을 선택한다. 정확한 instance TERMINATED/boot 잔존을 다시 읽으며 권한 오류를 삭제 성공으로 처리하지 않는다.
3. 선택 track은 소비 VM/boot부터 종료한 뒤 예약/host의 정확한 OCID를 Delete하고 get/list로 잔존 capacity/host를 확인한다. 미선택 자원의 삭제/조회 명령은 실행하지 않는다.


## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 아래 본문은 각 생성 필드·설정·검증·정리의 기술 근거이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [9] [Overview of the Compute Service — Capacity Types](https://docs.oracle.com/en-us/iaas/Content/Compute/Concepts/computeoverview.htm#capacity_types)
  - 확인 범위: Capacity Types.

### 추가 실행 절차 근거 (본문 확인)

- [launch](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/launchinginstance.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [preempt](https://docs.oracle.com/en-us/iaas/Content/Compute/Concepts/preemptible.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [reserve](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/capacity-reservations-create_tabs.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [host](https://docs.oracle.com/en-us/iaas/Content/Compute/Concepts/dedicatedvmhosts_topic-Creating_a_Dedicated_Virtual_Machine_Host.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [policy](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [cloudshell](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/cloudshellgettingstarted.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
