# Boot Volume

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../README.md)를 확인하세요.

> 실행 절차 작성본 / OCI 미실행. Oracle 공식 본문 근거와 교육용 선택을 구분한다. secrets·실제 고객값·diagram 없음.

## 학습목표

- owner10의 같은 vm-a launch에서 image-default boot 크기·성능·암호화를 선택하고 실제 boot OCID/AD/attachment를 조회한다.
- guest root mount와 API 용량을 비파괴 대조하고 root에 mkfs하지 않는다. 별도 승인 backup만 생성·조회·삭제한다.

## 대응 Roadmap 항목

- 항목명: **Boot Volume**
- Roadmap 경로: **Compute → Boot Volume**
- Node ID: `sequence-2-3`
- 상위 항목: **Compute** (`sequence-2-0`)
- 문서 범위: owner10의 같은 vm-a launch에서 image-default boot 크기·성능·암호화를 선택하고 실제 boot OCID/AD/attachment를 조회한다. guest root mount와 API 용량을 비파괴 대조하고 root에 mkfs하지 않는다. 별도 승인 backup만 생성·조회·삭제한다. 실제 OCI 실행 결과가 아니라 이 절차의 완료 조건이다.
- 번호는 navigation 식별자이며 실행 순서는 사전준비의 단계별 의존성에 따른다.

## 사전준비

### 기준 계약과 실제 단계 선행조건

[공통 baseline-contract.json](../../baseline-contract.json)의 Oracle Linux 9 / opc / CIDR / 비용 상한을 따른다. 문서 번호는 실행 순서가 아니다. **준비 station과 생성 이후 ready를 분리**한다. 아래 단계 외의 기능완료를 소급하여 요구하지 않는다.

- [10 Compute — 단계 링크](00_Compute.md#shared-launch): **공동 launch boot 선택 station; 완료 VM 불필요**. 실제 출력/소비자/expiry를 전달하고 해당 owner가 이 lab 종료까지 보존한다.

생성 전 조직 승인(identity/권한/예산/expiry/서비스 지원/limit/capacity)을 확인한다. 실제 OCID/IP는 아직 생성하지 않았으면 비워 두고 해당 Create/readback 뒤 채운다. 동일 creation-event-id/OCID는 한 번의 제출만 뜻한다.  실습에 없는 자원을 추가하지 않는다. 실제 checkpoint는 사용자 실행 전 비워 둔다.

### 입력값과 실행 위치

Console에서 선택한 값만 사용한다. `<...>`는 예시 OCID나 비밀값이 아니라 **사용자가 대체할 입력 자리**다. 대체하지 않은 채 명령을 실행하지 않는다. Console → 우측 상단 Cloud Shell/Code Editor → Cloud Shell을 열면 OCI CLI가 사전 인증된다. Region 변경 뒤에는 Shell을 다시 열거나 명령의 `--region`을 바꾼다. SSH는 Cloud Shell이 아니라 본인 Linux/WSL/macOS 터미널에서 수행한다. 공인 IP/32도 그 터미널의 출구 주소여야 한다.

```bash
# 아래는 입력/출력의 대입 형식이다. 표의 단계 전에 생성 후 ID/IP export 행을 실행하지 않는다.
 # 생성 후 검증용 ID/IP는 해당 creator의 실제 응답에서 넣는다.
export REGION='<actual_REGION>'
export INSTANCE_A_ID='<actual_INSTANCE_A_ID>'
export BOOT_VOLUME_ID='<actual_BOOT_VOLUME_ID>'
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
| Console/설정 필드 | owner10 same boot-a event; image default size; Balanced/default; Oracle-managed; backup/replication None; 선택 backup 별도 승인 | 생성 전 선택 → 생성 뒤 readback | 본문 Console/로컬 지정 | 이 topic의 최소 생성 범위 |
| `BOOT_VOLUME_ID` | 대상 instance creator Console Storage의 연결 boot OCID | 생성 후 | Cloud Shell | 같은 AD/boot 소유권 |
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

기본은 owner10이 전달한 boot OCID/AD와 GetBootVolume, 같은 instance/VNIC 관계 조회다. boot 생성은 owner10의 launch 한 번이며 이 reader가 VM을 생성·종료하지 않는다. 선택 backup의 Create/Get/Delete는 별도 승인을 받아야 하며 기본 읽기 권한으로 backup 생성 허가를 대신하지 않는다.

### 실제 operation과 공식 권한

- `CreateBootVolumeBackup`: BOOT_VOLUME_BACKUP_CREATE, VOLUME_WRITE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `DeleteBootVolumeBackup`: BOOT_VOLUME_BACKUP_DELETE, VOLUME_INSPECT — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `GetBootVolume`: VOLUME_INSPECT — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `GetBootVolumeBackup`: BOOT_VOLUME_BACKUP_INSPECT, VOLUME_INSPECT — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `GetInstance`: INSTANCE_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `GetVnic`: VNIC_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `ListVnicAttachments`: VNIC_ATTACHMENT_READ and INSTANCE_INSPECT — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).

위 permission 이름은 IAM policy resource-type이 아니다. 조직 관리자가 기존 group/compartment scope와 conditional dependency를 실제 API 표에 대조한다. 편의 family grant를 최소권한 정책으로 부르지 않으며 실습자는 policy 작성 권한을 자동 인수하지 않는다.

**Boot attachment 조회 범위:** 이 문서는 미확인 `ListBootVolumeAttachments` 최소권한을 요구하는 CLI를 실행하지 않는다. 생성 owner가 승인된 Compute Console → 정확한 instance OCID → Storage → Boot volume에서 연결된 boot OCID/AD와 instance를 읽어 전달한다. boot를 검증하는 owner13만 GetBootVolume 권한을 승인받아 그 OCID를 조회한다. Console attachment를 볼 권한이 없으면 creator가 exact instance/boot 관계를 제공하기 전 중단한다. 권한 표에 없는 operation을 최소권한 검증 완료로 표시하지 않는다.

## 비용및주의사항

- boot-a는 owner10 vm-a 최초 launch에서 한 번 생성되며 이 문서에서 두 번째 boot/VM을 만들지 않는다. image-default 실제 크기와 Balanced/default 성능의 비용을 확인하고 custom size·replication·backup policy·KMS는 추가하지 않는다.
- 선택 boot-a-lab-backup 1개는 별도 저장비/expiry/삭제 권한 승인이 있을 때만 생성한다. retention lock/hold/prevent deletion을 켜지 않는다. root에는 mkfs/fdisk를 실행하지 않으며 boot 삭제는 owner10의 소비자 반환 후, 선택 backup 삭제는 owner13의 책임이다.

실제 OCI 작업은 아직 수행하지 않았다. 기대값은 실행 시 대조할 조건이며 무료 사용이나 과금 강제 차단을 보장하지 않는다. 승인 expiry가 끝나면 새 시험을 중단하고 위 소유권 순서로 원복·반환한다.

## 실습구성

### 이번 Workshop에서 만드는 것과 진행 순서

boot-a는 vm-a와 함께 만들어지는 AD-scoped boot volume이다. root 조회만 하며 backup 한 개는 별도 승인 확장이다.

1. **진입/선택:** default boot size/encryption/performance 선택. 사전준비 링크의 지정 단계/실제 출력이 없으면 제출하지 않는다.
2. **실제 작업:** owner10 같은 Create → attachment/AD/root 조회. 아래 Console 필드/명령을 해당 host에서 순서대로 실행한다. 같은 자원은 이름이 아니라 OCID/event로 재사용한다.
3. **완료/복구/반환:** 승인시에만 backup 생성/조회. 결과검증 실패는 문제해결로 이동하고 동일 Create를 무작정 재제출하지 않는다.

**전달/정리 범위:** boot-a owner10; 선택 backup owner13가 별도 삭제; restore VM 없음. 아래 정리 단계는 이 소유권을 넘어서 실행하지 않는다.


- **이번 생성/재사용/출력:** 신규 boot-a는 vm-a launch event 소유. 선택 backup 최대 1개, restore VM은 기본에서 생성하지 않는다.
- **생성 목표:** Compute의 단일 실제 생성 event에서 Boot Volume 옵션을 직접 선택하고 저장·기능 검증한다. 독립 중복 생성은 하지 않는다.

## 실습절차

### 명령 실행 gate — 실패하면 다음 생성으로 진행하지 않는다

1. 아래 Bash는 **전용 Bash 세션**에서만 사용한다. Console/Cloud Shell 조회, 본인 로컬 SSH, 로그인한 Oracle Linux guest를 섞지 않는다. `exit 1`은 이 전용 세션을 닫는 안전 중단이다. `<...>`·빈 입력·타 owner OCID가 남으면 실행하지 않는다.
2. 성공을 요구하는 파일 작성/조회/설치/attach 작업은 실패 시 중단한다. 실패한 API 뒤 기존 JSON이나 기존 다운로드 파일을 성공 근거로 읽지 않는다. 기대 deny는 `if command; then ...; else RC=$?; ...; fi`로 별도 기록하고, 실제 timeout/IAM 오류/로컬 파일 오류를 구별한다.
3. 첫 변경 전에 정확한 OCID·owner·현재 설정과 승인 expiry를 저장한다. 공유 자원 변경 중 다른 소비자가 시작하거나 원복값을 확인할 수 없으면 변경을 중단한다. 삭제 성공은 API 수락이 아니라 같은 identity/Region의 정확한 대상 재조회로 판정한다. `NotAuthorizedOrNotFound` 단독은 삭제 증거가 아니다.


### 개별 완주 보정 — 생성·검증·소유권

root에는 mkfs/fdisk를 실행하지 않는다. backup lookup은 실제 생성하고 신규 OCID가 있을 때만 실행한다. 부트 기본 수명은 owner10, 승인 backup은 owner13가 잠금/hold/잔존을 읽어 별도 정리한다.

<a id="boot-selection"></a>

1. Compute의 생성 전 Boot volume station에서 Oracle Linux platform image 기본 boot size를 화면에서 읽고 기록한다. 최소 size를 고정 50 GB라고 가정하지 말고 image가 요구하는 실제 default를 쓴다. Custom size를 올리지 않는다. Boot volume은 instance create 시 같이 만들어진다. [About Boot Volumes and Custom Boot Volume Sizes](https://docs.oracle.com/en-us/iaas/Content/Block/Concepts/bootvolumes.htm).
2. Default/Balanced performance, Oracle-managed encryption을 확인한다. KMS key/backup policy/replication을 추가하지 않고 vm-a actual Create를 함께 제출한다. boot volume의 생성 OCID를 instance Storage 탭/boot attachment에서 찾는다.
3. 아래 비파괴 조회로 root filesystem과 block device 관계를 확인한다. root 장치에 mkfs/fdisk를 실행하지 않는다. API size와 filesystem size 차이는 partition/OS 영역으로 해석한다.

```bash
# Cloud Shell only: owner10 exact boot attachment handoff
oci bv boot-volume get --boot-volume-id '<boot-a_OCID>' --region "$REGION" > boot-volume.json || exit 1
```

**본인 Linux/WSL 로컬 전용 terminal:** Cloud Shell fence 종료 후 owner15 KEY와 owner10 동일 vm-a 실제 주소를 다시 지정한다. owner10 trust bootstrap 완료 없으면 중단한다.

```bash
export KEY='<owner15_actual_local_private_key_absolute_path>'
export PUBLIC_IP='<owner10_actual_vm-a_public_IP>'
test -f "$KEY" && test ! -L "$KEY" && test -r "$KEY" || exit 1
case "$PUBLIC_IP" in ''|*'<'*|*'>'*) exit 1;; esac
timeout --foreground -k 5s 60s ssh -i "$KEY" -o IdentitiesOnly=yes -o HostKeyAlgorithms=ssh-ed25519 -o StrictHostKeyChecking=yes -o ConnectTimeout=10 -o ConnectionAttempts=1 -o ControlMaster=no -o ControlPath=none opc@"$PUBLIC_IP" 'findmnt /; lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS; df -h /' || exit 1
```
4. 같은 AD, 실제 boot size, encryption(KMS field null이면 Oracle-managed), attachment와 instance를 대조한다. Stop/terminate에서 보존 옵션의 의미는 비용과 데이터 생존을 결정한다. primary cleanup은 boot 삭제이며 데이터 복구/backup 실제 생성은 별도 승인 확장이다.

### 선택 확장 — boot backup 1개 (승인한 경우만)

Storage → Block Storage → Boot Volumes → boot-a → Backups → Create Boot Volume Backup. Name `boot-a-lab-backup`, Type **Incremental**, Retention **No retention period (keep until deleted)**, Retention lock Off, Prevent deletion Off, Indefinite hold Off. Create 제출 후 AVAILABLE를 확인한다. 이 설정은 영구 보존 약속이 아니라 lab 직후 수동 삭제를 허용하기 위한 선택이다. 현재 retention feature 권한을 승인하지 못하면 실행하지 않는다. [Create Backup — retention/lock/hold fields](https://docs.oracle.com/en-us/iaas/Content/Block/Tasks/create-bv-boot-volume-backup.htm).

### Create 이후 값 전달 — 이름으로 추정하지 않는다

Cloud Shell의 INSTANCE_ID는 해당 creator Create 응답 id로 지정한다. Console → Compute → Instances → 정확한 instance → Networking/Attached VNICs에서 primary VNIC OCID를 기록하고 아래 GetVnic의 `is-primary=true`, `subnet-id`, `private-ip`, `public-ip`를 대조한다. Storage/Boot volume의 연결 boot OCID·AD도 creator가 기록한다. Cloud Shell과 로컬 터미널의 변수는 공유되지 않으므로 SSH 전에 KEY와 **그 instance의** 실제 접속 주소를 로컬에 다시 지정한다. 이전 extra VM 주소를 재사용하지 않는다. 모든 JSON 조회는 실패하면 즉시 중단하고 기존 파일을 읽지 않는다.

## 결과검증

<a id="baseline-13-ready"></a>

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
oci bv boot-volume get --boot-volume-id '<boot-a_OCID>' --region "$REGION" > boot-volume.json || exit 1
# 선택 확장은 별도 실행한다. 아래 placeholder가 남았으면 이 줄을 실행하지 않는다.
# 선택 확장을 실제 수행했을 때만
# 승인/실제 생성 뒤 별도 실행: oci bv boot-volume-backup get --boot-volume-backup-id '<신규_backup_OCID>' --region "$REGION" > boot-backup.json
```
primary 기대: boot AVAILABLE/same AD/선택 size, boot attachment instance-id 일치, guest root mounted. optional backup 기대 AVAILABLE/정확한 boot-volume-id/잠금·hold 비활성; 생성하지 않았으면 NOT_SELECTED.

**증거 기록:** 실제 명령·UTC 시간·exit code·가려진 OCID·옵션 readback·검증 조건을 남긴다. 현재 이 문서의 실행상태는 `NOT_EXECUTED`다. output/creation-event-id/result는 실행 전 비워 둔다. 오류/NOT_SELECTED를 PASS로 바꾸지 않는다.

### 이 시나리오의 functional acceptance

| Positive | Negative / 미완료 판정 | Recovery / 반환 |
|---|---|---|
| actual boot OCID/AD/root mount | root device format 시도는 금지; 미승인 backup은 NOT_SELECTED | 비파괴 Storage/guest identity 재조회; backup은 승인 신규 OCID에만 작업 |

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

boot 연결을 못 찾으면 creator Console의 AD/compartment/instance OCID와 Storage 탭을 대조한다. partition size가 volume size보다 작은 경우 resize/확장 task를 먼저 읽고 root filesystem을 format하지 않는다.

## 리소스정리

boot-a owner10; 선택 backup owner13가 별도 삭제; restore VM 없음. 공유 기반 리소스는 이 consumer가 직접 삭제하지 않는다. 실제 소비자/expiry를 creator에게 반환하고 creator의 리소스정리 절차를 사용한다. VM을 Stop했다고 boot/IP/backup 비용이 끝난 것으로 처리하지 않는다. 로컬/guest 파일은 자신이 만든 정확한 경로만 제거하며 기록된 증거를 보존한다.

선택 backup을 만든 경우에만 owner13가 Boot Volume Backups → 신규 OCID → Delete를 제출한다. 잠금/hold가 있으면 관리자 승인 전 임의 해제하지 않는다. get/list에서 신규 backup만 제거되었는지 읽고 boot-a 삭제와 혼동하지 않는다.


## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 아래 본문은 각 생성 필드·설정·검증·정리의 기술 근거이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [8] [Overview of Block Volume — Boot Volume](https://docs.oracle.com/en-us/iaas/Content/Block/Concepts/overview.htm)
  - 확인 범위: Boot volume:.

- [7] [Working with Boot Volumes — Boot Volume](https://docs.oracle.com/en-us/iaas/Content/Block/Concepts/bootvolumes.htm)
  - 확인 범위: About Boot Volumes / Troubleshooting and repair:.

### 추가 실행 절차 근거 (본문 확인)

- [boot](https://docs.oracle.com/en-us/iaas/Content/Block/Concepts/bootvolumes.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [launch](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/launchinginstance.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [bootbackup](https://docs.oracle.com/en-us/iaas/Content/Block/Tasks/create-bv-boot-volume-backup.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [cloudshell](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/cloudshellgettingstarted.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
