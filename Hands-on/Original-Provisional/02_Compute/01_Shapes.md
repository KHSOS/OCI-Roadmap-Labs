> **기존 실습 초안 — 미검증**: 원래 93개 문서의 절차를 보존한 자료입니다. 새 튜토리얼의 품질 승인이나 OCI 실행 검증을 뜻하지 않습니다.

# Shapes

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../../../Docs/README.md)를 확인하세요.

> 실행 절차 작성본 / OCI 미실행. Oracle 공식 본문 근거와 교육용 선택을 구분한다. secrets·실제 고객값·diagram 없음.

## 학습목표

- 선택 Oracle Linux image와 호환되는 shape 목록에서 승인된 OCPU/memory를 골라 owner10의 같은 vm-a launch에 반영한다.
- instance shape-config와 guest uname/lscpu/free를 대조하고 비교용 VM나 resize 없이 architecture 일치를 확인한다.

## 대응 Roadmap 항목

- 항목명: **Shapes**
- Roadmap 경로: **Compute → Shapes**
- Node ID: `sequence-2-1`
- 상위 항목: **Compute** (`sequence-2-0`)
- 문서 범위: 선택 Oracle Linux image와 호환되는 shape 목록에서 승인된 OCPU/memory를 골라 owner10의 같은 vm-a launch에 반영한다. instance shape-config와 guest uname/lscpu/free를 대조하고 비교용 VM나 resize 없이 architecture 일치를 확인한다. 실제 OCI 실행 결과가 아니라 이 절차의 완료 조건이다.
- 번호는 navigation 식별자이며 실행 순서는 사전준비의 단계별 의존성에 따른다.

## 사전준비

### 기준 계약과 실제 단계 선행조건

[공통 baseline-contract.json](../../../baseline-contract.json)의 Oracle Linux 9 / opc / CIDR / 비용 상한을 따른다. 문서 번호는 실행 순서가 아니다. **준비 station과 생성 이후 ready를 분리**한다. 아래 단계 외의 기능완료를 소급하여 요구하지 않는다.

- [10 Compute — 단계 링크](00_Compute.md#shared-launch): **공동 launch 사전준비와 image/shape station; 완료 VM 불필요**. 실제 출력/소비자/expiry를 전달하고 해당 owner가 이 lab 종료까지 보존한다.

생성 전 조직 승인(identity/권한/예산/expiry/서비스 지원/limit/capacity)을 확인한다. 실제 OCID/IP는 아직 생성하지 않았으면 비워 두고 해당 Create/readback 뒤 채운다. 동일 creation-event-id/OCID는 한 번의 제출만 뜻한다.  실습에 없는 자원을 추가하지 않는다. 실제 checkpoint는 사용자 실행 전 비워 둔다.

### 입력값과 실행 위치

Console에서 선택한 값만 사용한다. `<...>`는 예시 OCID나 비밀값이 아니라 **사용자가 대체할 입력 자리**다. 대체하지 않은 채 명령을 실행하지 않는다. Console → 우측 상단 Cloud Shell/Code Editor → Cloud Shell을 열면 OCI CLI가 사전 인증된다. Region 변경 뒤에는 Shell을 다시 열거나 명령의 `--region`을 바꾼다. SSH는 Cloud Shell이 아니라 본인 Linux/WSL/macOS 터미널에서 수행한다. 공인 IP/32도 그 터미널의 출구 주소여야 한다.

```bash
# 아래는 입력/출력의 대입 형식이다. 표의 단계 전에 생성 후 ID/IP export 행을 실행하지 않는다.
 # 생성 후 검증용 ID/IP는 해당 creator의 실제 응답에서 넣는다.
export REGION='<actual_REGION>'
export INSTANCE_A_ID='<actual_INSTANCE_A_ID>'
export LAB_COMPARTMENT_ID='<actual_LAB_COMPARTMENT_ID>'
export AD='<actual_AD>'
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
| Console/설정 필드 | OL9 image architecture; compatible A1 1OCPU/6GB 또는 승인 E4 1OCPU/2GB; same vm-a Create; resize 없음 | 생성 전 선택 → 생성 뒤 readback | 본문 Console/로컬 지정 | 이 topic의 최소 생성 범위 |
| `AD` | owner06의 실제 tenancy AD 목록에서 선택한 이름 | ListShapes/launch 전 | Cloud Shell/Console | 생성 후 값을 선행으로 요구하지 않음 |
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

생성 전 ListShapes로 선택 AD/image의 호환 shape를 확인하고 생성 뒤 owner10의 같은 INSTANCE_A_ID/GetInstance·primary VNIC만 읽는다. Shapes 비교용 LaunchInstance/UpdateInstance/Terminate 권한은 요구하지 않는다. 실제 단일 launch의 의존 권한은 owner10 Compute에서 확인한다.

### 실제 operation과 공식 권한

- `GetInstance`: INSTANCE_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `GetVnic`: VNIC_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `ListShapes`: INSTANCE_INSPECT — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `ListVnicAttachments`: VNIC_ATTACHMENT_READ and INSTANCE_INSPECT — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).

위 permission 이름은 IAM policy resource-type이 아니다. 조직 관리자가 기존 group/compartment scope와 conditional dependency를 실제 API 표에 대조한다. 편의 family grant를 최소권한 정책으로 부르지 않으며 실습자는 policy 작성 권한을 자동 인수하지 않는다.

## 비용및주의사항

- Shapes 때문에 신규 VM이나 비교용 A1/E4 두 대를 만들지 않는다. owner10 vm-a의 OCPU와 memory 선택이 실제 Compute 비용을 결정하므로 Create 이전 승인 shape/1 OCPU/6 GB 또는 2 GB를 확인한다.
- architecture 불일치 또는 capacity 부족이면 Create를 중단한다. 큰 shape 자동 대체, resize, reboot 시험은 기본 범위가 아니다. guest 메모리 차이를 이유로 유료 증설하지 않고 vm-a는 owner10에게 반환한다.

실제 OCI 작업은 아직 수행하지 않았다. 기대값은 실행 시 대조할 조건이며 무료 사용이나 과금 강제 차단을 보장하지 않는다. 승인 expiry가 끝나면 새 시험을 중단하고 위 소유권 순서로 원복·반환한다.

## 실습구성

### 이번 Workshop에서 만드는 것과 진행 순서

vm-a launch의 architecture/OCPU/memory 선택을 실제 guest와 비교한다. A1/E4 비교용 추가 VM과 resize는 하지 않는다.

1. **진입/선택:** image architecture. 사전준비 링크의 지정 단계/실제 출력이 없으면 제출하지 않는다.
2. **실제 작업:** compatible shapes 조회 → 승인 작은 shape 선택 → owner10 같은 Create. 아래 Console 필드/명령을 해당 host에서 순서대로 실행한다. 같은 자원은 이름이 아니라 OCID/event로 재사용한다.
3. **완료/복구/반환:** API shape-config/guest 비교. 결과검증 실패는 문제해결로 이동하고 동일 Create를 무작정 재제출하지 않는다.

**전달/정리 범위:** shape/OCPU/memory/architecture와 same event; owner10 VM 보존. 아래 정리 단계는 이 소유권을 넘어서 실행하지 않는다.


- **이번 생성/재사용/출력:** 신규 독립 VM 없음; vm-a actual create event의 shape/OCPU/memory 선택·guest 비교.
- **생성 목표:** Compute의 단일 실제 생성 event에서 Shapes 옵션을 직접 선택하고 저장·기능 검증한다. 독립 중복 생성은 하지 않는다.

## 실습절차

### 명령 실행 gate — 실패하면 다음 생성으로 진행하지 않는다

1. 아래 Bash는 **전용 Bash 세션**에서만 사용한다. Console/Cloud Shell 조회, 본인 로컬 SSH, 로그인한 Oracle Linux guest를 섞지 않는다. `exit 1`은 이 전용 세션을 닫는 안전 중단이다. `<...>`·빈 입력·타 owner OCID가 남으면 실행하지 않는다.
2. 성공을 요구하는 파일 작성/조회/설치/attach 작업은 실패 시 중단한다. 실패한 API 뒤 기존 JSON이나 기존 다운로드 파일을 성공 근거로 읽지 않는다. 기대 deny는 `if command; then ...; else RC=$?; ...; fi`로 별도 기록하고, 실제 timeout/IAM 오류/로컬 파일 오류를 구별한다.
3. 첫 변경 전에 정확한 OCID·owner·현재 설정과 승인 expiry를 저장한다. 공유 자원 변경 중 다른 소비자가 시작하거나 원복값을 확인할 수 없으면 변경을 중단한다. 삭제 성공은 API 수락이 아니라 같은 identity/Region의 정확한 대상 재조회로 판정한다. `NotAuthorizedOrNotFound` 단독은 삭제 증거가 아니다.


### 개별 완주 보정 — 생성·검증·소유권

shape-selection 준비에서는 compatibility/OCPU/memory만 고른다. baseline-11-ready는 owner10의 같은 launch 뒤 readback이다. 다른 shape 자동 대체/resize는 별도 비용승인 없이 수행하지 않는다.


### shape-selection


1. Compute의 생성 전 Image and shape station에 참여한다. Oracle Linux 9 image의 CPU architecture를 먼저 선택한 뒤 같은 architecture를 지원하는 shape를 선택한다. Flexible shape의 OCPU와 memory slider가 서로 제약되므로 최대값이나 default memory를 그대로 쓰지 않는다. [Flexible Shapes and memory ratios](https://docs.oracle.com/en-us/iaas/Content/Compute/References/computeshapes.htm).
2. 최소 VM primary option은 eligibility/limit/capacity가 확인된 A1 1 OCPU/6 GB, 불가능하면 승인 E4 1 OCPU/2 GB다. ARM 앱 image를 x86 shape로, x86 binary를 ARM shape로 혼용하지 않는다. A1과 E4를 비교하려고 두 VM을 동시에 만들지 않는다.
```bash
# Cloud Shell: Create 이전 compatibility 선택만
oci compute shape list --compartment-id "$LAB_COMPARTMENT_ID" --availability-domain '<선택_AD>' --image-id '<선택_image_OCID>' --all --region "$REGION" > compatible-shapes.json || exit 1
```

3. 선택된 shape/OCPU/memory를 기록한 후 Compute의 나머지 필드와 함께 **vm-a Create를 한 번 제출**한다. RUNNING readback에서 shape-config를 읽고 실제 guest CPU/memory를 확인한다. CPU thread 수와 OCPU 수가 항상 같다고 가정하지 않는다.

여기서 멈추고 owner10의 Create/RUNNING/VNIC/최초 fingerprint 확인을 완료한다. 아래는 **그 뒤 본인 로컬 터미널**에서만 실행한다; Cloud Shell fence와 연속 paste하지 않는다.

```bash
export KEY='<owner15_local_private_key_absolute_path>'
export PUBLIC_IP='<owner10_actual_VNIC_public_ip>'
ssh -i "$KEY" -o ConnectTimeout=10 opc@"$PUBLIC_IP" 'lscpu; free -m; uname -m' || exit 1
```
4. guest에는 OS reserved memory와 단위 차이 때문에 free의 total이 Console GB와 정확히 같지 않을 수 있다. shape 설정값과 architecture 일치를 primary validation으로 쓴다. 실행 후 shape resize 비교는 비용 승인/재부팅 조건이 별도로 필요하며 본 최소 lab에서는 하지 않는다.

### Create 이후 값 전달 — 이름으로 추정하지 않는다

Cloud Shell의 INSTANCE_ID는 해당 creator Create 응답 id로 지정한다. Console → Compute → Instances → 정확한 instance → Networking/Attached VNICs에서 primary VNIC OCID를 기록하고 아래 GetVnic의 `is-primary=true`, `subnet-id`, `private-ip`, `public-ip`를 대조한다. Storage/Boot volume의 연결 boot OCID·AD도 creator가 기록한다. Cloud Shell과 로컬 터미널의 변수는 공유되지 않으므로 SSH 전에 KEY와 **그 instance의** 실제 접속 주소를 로컬에 다시 지정한다. 이전 extra VM 주소를 재사용하지 않는다. 모든 JSON 조회는 실패하면 즉시 중단하고 기존 파일을 읽지 않는다.

## 결과검증


### baseline-11-ready


```bash
oci compute instance get --instance-id "$INSTANCE_ID" --region "$REGION" > instance-readback.json || exit 1
oci compute vnic-attachment list --compartment-id "$LAB_COMPARTMENT_ID" --instance-id "$INSTANCE_ID" --region "$REGION" > vnic-attachments.json || exit 1
# Cloud Shell: 현재 instance에 붙은 ATTACHED VNIC만 각각 조회한다.
export VNIC_ID='<현재_INSTANCE_ID에_ATTACHED이며_GetVnic_is-primary=true인_OCID>'
case "$VNIC_ID" in ocid1.vnic.*) ;; *) printf '%s\n' 'STOP: actual primary VNIC OCID required'; exit 1;; esac
oci network vnic get --vnic-id "$VNIC_ID" --region "$REGION" > vnic-readback.json || exit 1
```
기대 패턴: instance `lifecycle-state=RUNNING`, `availability-domain`/`fault-domain`/`shape`가 제출값과 같음, VNIC `subnet-id`가 선택값과 같음, public VM은 `public-ip`가 문자열이고 private VM은 null이다. creator Console Storage의 boot 연결이 대상 instance와 boot OCID를 연결한다. 파일을 실제로 읽어 조건을 대조한다.

기대 패턴 A1이면 shape=VM.Standard.A1.Flex, shape-config.ocpus=1/memory-in-gbs=6, guest aarch64; E4이면 shape=VM.Standard.E4.Flex, 1/2, guest x86_64. 다른 승인 shape면 제출값으로 비교하며 lscpu만으로 과금 vCPU를 판정하지 않는다.

**증거 기록:** 실제 명령·UTC 시간·exit code·가려진 OCID·옵션 readback·검증 조건을 남긴다. 현재 이 문서의 실행상태는 `NOT_EXECUTED`다. output/creation-event-id/result는 실행 전 비워 둔다. 오류/NOT_SELECTED를 PASS로 바꾸지 않는다.

### 이 시나리오의 functional acceptance

| Positive | Negative / 미완료 판정 | Recovery / 반환 |
|---|---|---|
| image-compatible shape/shape-config/guest arch | compatible list 없는 shape는 제출 전 거부 | 승인 architecture-compatible 작은 shape 재선택; 자동 resize/추가 VM 없음 |

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

shape list가 image와 호환되지 않으면 CPU architecture와 region/AD를 확인한다. guest free total과 Console GB의 소폭 차이만으로 memory provisioning 실패를 판정하지 않는다.

## 리소스정리

shape/OCPU/memory/architecture와 same event; owner10 VM 보존. 공유 기반 리소스는 이 consumer가 직접 삭제하지 않는다. 실제 소비자/expiry를 creator에게 반환하고 creator의 리소스정리 절차를 사용한다. VM을 Stop했다고 boot/IP/backup 비용이 끝난 것으로 처리하지 않는다. 로컬/guest 파일은 자신이 만든 정확한 경로만 제거하며 기록된 증거를 보존한다.


## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 아래 본문은 각 생성 필드·설정·검증·정리의 기술 근거이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [12] [Compute Shapes — Shapes](https://docs.oracle.com/en-us/iaas/Content/Compute/References/computeshapes.htm)
  - 확인 범위: A shape is a template / A flexible shape is a shape / Flexible memory is also.

### 추가 실행 절차 근거 (본문 확인)

- [shapes](https://docs.oracle.com/en-us/iaas/Content/Compute/References/computeshapes.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [launch](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/launchinginstance.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [connect](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/connect-to-linux-instance.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [cloudshell](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/cloudshellgettingstarted.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
