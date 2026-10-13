# Block Volume

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../README.md)를 확인하세요.

> 실행 절차 작성본 / OCI 미실행. Oracle 공식 본문 근거와 교육용 선택을 구분한다. secrets·실제 고객값·diagram 없음.

## 학습목표

- 같은 AD에 신규 50 GB data-a를 생성·paravirtualized attach하고 non-root·empty·mountpoint guard를 통과한 장치에만 ext4를 만든다.
- 합성 파일의 checksum을 기록하고 unmount→detach→같은 OCID reattach 후 mkfs 없이 checksum을 비교한 뒤 volume만 정리한다.

## 대응 Roadmap 항목

- 항목명: **Block Volume**
- Roadmap 경로: **Compute → Block Volume**
- Node ID: `sequence-2-4`
- 상위 항목: **Compute** (`sequence-2-0`)
- 문서 범위: 같은 AD에 신규 50 GB data-a를 생성·paravirtualized attach하고 non-root·empty·mountpoint guard를 통과한 장치에만 ext4를 만든다. 합성 파일의 checksum을 기록하고 unmount→detach→같은 OCID reattach 후 mkfs 없이 checksum을 비교한 뒤 volume만 정리한다. 실제 OCI 실행 결과가 아니라 이 절차의 완료 조건이다.
- 번호는 navigation 식별자이며 실행 순서는 사전준비의 단계별 의존성에 따른다.

## 사전준비

### 기준 계약과 실제 단계 선행조건

[공통 baseline-contract.json](../../baseline-contract.json)의 Oracle Linux 9 / opc / CIDR / 비용 상한을 따른다. 문서 번호는 실행 순서가 아니다. **준비 station과 생성 이후 ready를 분리**한다. 아래 단계 외의 기능완료를 소급하여 요구하지 않는다.

- [10 Compute — 단계 링크](00_Compute.md#baseline-10-ready): **vm-a RUNNING/SSH/AD/root inventory**. 실제 출력/소비자/expiry를 전달하고 해당 owner가 이 lab 종료까지 보존한다.

생성 전 조직 승인(identity/권한/예산/expiry/서비스 지원/limit/capacity)을 확인한다. 실제 OCID/IP는 아직 생성하지 않았으면 비워 두고 해당 Create/readback 뒤 채운다. 동일 creation-event-id/OCID는 한 번의 제출만 뜻한다. `INSTANCE_A_ID`는 owner10 vm-a의 OCID, `INSTANCE_B_ID`는 owner22 private-vm의 OCID이며 owner07 vm-b와 다르다. 실습에 없는 자원을 추가하지 않는다. 실제 checkpoint는 사용자 실행 전 비워 둔다.

### 입력값과 실행 위치

Console에서 선택한 값만 사용한다. `<...>`는 예시 OCID나 비밀값이 아니라 **사용자가 대체할 입력 자리**다. 대체하지 않은 채 명령을 실행하지 않는다. Console → 우측 상단 Cloud Shell/Code Editor → Cloud Shell을 열면 OCI CLI가 사전 인증된다. Region 변경 뒤에는 Shell을 다시 열거나 명령의 `--region`을 바꾼다. SSH는 Cloud Shell이 아니라 본인 Linux/WSL/macOS 터미널에서 수행한다. 공인 IP/32도 그 터미널의 출구 주소여야 한다.

```bash
# 아래는 입력/출력의 대입 형식이다. 표의 단계 전에 생성 후 ID/IP export 행을 실행하지 않는다.
 # 생성 후 검증용 ID/IP는 해당 creator의 실제 응답에서 넣는다.
export REGION='<actual_REGION>'
export INSTANCE_A_ID='<actual_INSTANCE_A_ID>'
export DATA_VOLUME_ID='<actual_DATA_VOLUME_ID>'
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
| Console/설정 필드 | Name=data-a; same AD as vm-a; 50GB; Balanced; Oracle-managed; backup/replication None; Paravirtualized/ReadWrite single instance | 생성 전 선택 → 생성 뒤 readback | 본문 Console/로컬 지정 | 이 topic의 최소 생성 범위 |
| `PUBLIC_IP` | 본문 실제 Create/readback 출력 | 생성 후/선행 완료 | 본문 지정 host | 해당 topic 대상만 지정 |
| `KEY` | owner15 Part A 전용 private key 절대경로; 내용 전송 금지 | 준비 완료 | 로컬 SSH만 | 해당 공개키와 대응 |
| `DATA_VOLUME_ID` | owner14 data-a 신규 Create id | 생성 후 | Console/Cloud Shell | format 대상/삭제 scope |
| `A_PUBLIC_IP` | owner10 vm-a VNIC public-ip | 선행 완료/생성 후 | 로컬 SSH | 관리 jump 주소 |
| `DEVICE` | 신규 data-a attachment path + serial + root inventory 실제 대조 | attach 이후 | vm-a guest | 파괴적 mkfs 대상 확인 |
| `INSTANCE_A_ID` | owner10 실제 vm-a 생성 id | 선행 완료/생성 후 | Cloud Shell | 공유 A 조회; 이름은 OCID가 아님 |
| `INSTANCE_ID` | owner10 vm-a Create/readback id | 생성 후/선행 완료 | Cloud Shell | 현재 문서 대상 OCID에 명시 대입 |
| `LAB_COMPARTMENT_ID` | owner00 Create 응답 OCID; topic00에서는 생성 뒤 | 선행 완료/생성 후 | Console/Cloud Shell | 정책/자원 scope |
| `REGION` | 승인된 기존 subscribed Region identifier | 생성 전 | Console/Cloud Shell | API endpoint/데이터 위치 |

생성 전에는 행의 승인값/선행 완료값만 지정하고, 생성 후 ID/IP는 해당 owner 응답으로 채운다. Cloud Shell과 로컬 SSH의 환경변수는 공유되지 않는다. 로컬에는 KEY와 접속 주소만 다시 지정하고 guest disk 명령은 로그인한 guest에서만 실행한다. `<...>`가 남으면 해당 명령은 실행하지 않는다.

## 권한

data-a 생성/조회/삭제와 기존 vm-a에 대한 AttachVolume/DetachVolume/ListVolumeAttachments만 승인한다. volume-family만으로 instance attach/detach 의존 권한이 충족됐다고 가정하지 않는다. VM launch/terminate/network CRUD는 이 단계에서 필요 없다. guest의 장치 inventory/mkfs/mount에는 opc의 승인 sudo가 필요하다.

### 실제 operation과 공식 권한

- `AttachVolume`: VOLUME_ATTACHMENT _CREATE and VOLUME_WRITE and INSTANCE_ATTACH_VOLUME — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `CreateVolume`: VOLUME_CREATE (and VOLUME_BACKUP_READ if creating volume from a backup) — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `DeleteVolume`: VOLUME_DELETE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `DetachVolume`: VOLUME_ATTACHMENT _DELETE and VOLUME_WRITE and INSTANCE_DETACH_VOLUME — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `GetVolume`: VOLUME_INSPECT — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `ListVolumeAttachments`: VOLUME_ATTACHMENT _INSPECT and VOLUME_INSPECT and INSTANCE_INSPECT — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).

위 permission 이름은 IAM policy resource-type이 아니다. 조직 관리자가 기존 group/compartment scope와 conditional dependency를 실제 API 표에 대조한다. 편의 family grant를 최소권한 정책으로 부르지 않으며 실습자는 policy 작성 권한을 자동 인수하지 않는다.

## 비용및주의사항

- 추가 승인 대상은 같은 AD의 data-a 50 GB Balanced volume 1개다. Create의 큰 default 크기를 그대로 제출하지 않고 backup policy/replication/reservation을 선택하지 않는다. vm-a/boot는 기존 자원이며 data 실습 때문에 다시 만들지 않는다.
- 50 GB 빈 신규 장치와 root backing chain·signature·mountpoint를 확인하기 전 mkfs 금지다. detach/reattach에서는 같은 OCID에 기존 filesystem을 재사용한다. 종료 후 자기 mount→attachment→data-a 순으로 정리하고 vm-a/boot는 삭제하지 않는다.

실제 OCI 작업은 아직 수행하지 않았다. 기대값은 실행 시 대조할 조건이며 무료 사용이나 과금 강제 차단을 보장하지 않는다. 승인 expiry가 끝나면 새 시험을 중단하고 위 소유권 순서로 원복·반환한다.

## 실습구성

### 이번 Workshop에서 만드는 것과 진행 순서

새 50 GB data-a를 vm-a에 paravirtualized attach하여 안전 확인 후 ext4를 만들고 detach/reattach checksum을 비교한다.

1. **진입/선택:** 기존 root/disk inventory. 사전준비 링크의 지정 단계/실제 출력이 없으면 제출하지 않는다.
2. **실제 작업:** same AD 50GB Create/Attach → 신규 empty disk 대조/guard → mkfs/mount/write → unmount/detach/reattach. 아래 Console 필드/명령을 해당 host에서 순서대로 실행한다. 같은 자원은 이름이 아니라 OCID/event로 재사용한다.
3. **완료/복구/반환:** mkfs 없이 cmp. 결과검증 실패는 문제해결로 이동하고 동일 Create를 무작정 재제출하지 않는다.

**전달/정리 범위:** DATA_VOLUME_ID/attachment/checksum; owner14는 unmount→detach→data volume 삭제, vm-a 유지. 아래 정리 단계는 이 소유권을 넘어서 실행하지 않는다.


- **이번 생성/재사용/출력:** 신규 data-a 50 GB 1개 + paravirtualized attachment + ext4 synthetic file. 기존 vm-a 재사용. 파일 생존 검증 뒤 삭제.
- **생성 목표:** 실제 빈 Block Volume 생성 -> paravirtualized attach -> 안전한 신규 장치 식별 -> 파일 기록 -> detach/reattach -> 파일 checksum 확인.

## 실습절차

### 작업 순서 체크포인트
format 이전의 장치/root/empty 확인 → mount/checksum → 같은 OCID remount 체크를 순서대로 수행한다. 기존 filesystem이 있으면 format하지 않고 owner에게 반환한다. 단계 실패는 다음 생성으로 넘기지 않고 같은 target의 복구/반환으로 처리한다.

### 명령 실행 gate — 실패하면 다음 생성으로 진행하지 않는다

1. 아래 Bash는 **전용 Bash 세션**에서만 사용한다. Console/Cloud Shell 조회, 본인 로컬 SSH, 로그인한 Oracle Linux guest를 섞지 않는다. `exit 1`은 이 전용 세션을 닫는 안전 중단이다. `<...>`·빈 입력·타 owner OCID가 남으면 실행하지 않는다.
2. 성공을 요구하는 파일 작성/조회/설치/attach 작업은 실패 시 중단한다. 실패한 API 뒤 기존 JSON이나 기존 다운로드 파일을 성공 근거로 읽지 않는다. 기대 deny는 `if command; then ...; else RC=$?; ...; fi`로 별도 기록하고, 실제 timeout/IAM 오류/로컬 파일 오류를 구별한다.
3. 첫 변경 전에 정확한 OCID·owner·현재 설정과 승인 expiry를 저장한다. 공유 자원 변경 중 다른 소비자가 시작하거나 원복값을 확인할 수 없으면 변경을 중단한다. 삭제 성공은 API 수락이 아니라 같은 identity/Region의 정확한 대상 재조회로 판정한다. `NotAuthorizedOrNotFound` 단독은 삭제 증거가 아니다.


### 개별 완주 보정 — 생성·검증·소유권

format/mount 명령 실패시 즉시 종료한다. readlink/serial/Consoleattachment/root/기존signature를 사람과 guard로 대조한다. 재attach시 mkfs는 하지 않으며 정확한dataOCID만 detach/delete한다.

### 할당 이전 guest 도구 준비

paid Create/Attach 전에 vm-a guest에서 아래 **비변경** preflight를 통과해야 한다. 하나라도 없으면 BLOCKED_TOOLING으로 중단한다. 제한 TCP443 경로에서 즉석 package install을 시도하지 않는다. 관리자가 승인된 Oracle Linux 9 repository/image에서 e2fsprogs(mkfs.ext4), util-linux(lsblk/findmnt/wipefs/blkid/mount), coreutils(sha256sum/cmp)을 준비하고 기존 inventory와 변경/rollback 승인을 기록한 후 이 gate부터 재개한다.

```bash
# vm-a guest; 새 volume 생성 이전
set -euo pipefail
for phase in before after; do
  out="$HOME/roadmap-data.$phase.sha256"
  test ! -e "$out" && test ! -L "$out" || { printf '%s\n' 'STOP: previous hash evidence exists; reconcile owner/run before allocation'; exit 1; }
done
for tool in python3 mkfs.ext4 lsblk findmnt wipefs blkid mount umount sha256sum cmp readlink; do
  command -v "$tool" >/dev/null || { printf 'BLOCKED_TOOLING: %s\n' "$tool"; exit 1; }
done
sudo -v || exit 1
```
도구 존재는 format 허가가 아니다. 아래 non-root/신규 empty disk/정확한 OCID 확인은 그대로 필수다.

### 1. 새 data volume과 attachment 생성

1. vm-a에서 기존 disk inventory/root device를 먼저 읽어 보관한다. `findmnt /; lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS; sudo blkid`를 실행한다. 이 lab는 빈 **신규** volume만 format하며 운영/기존 volume을 대상으로 하지 않는다.
2. Console → Storage → Block Storage → Block Volumes → Create block volume. Name `data-a`, Compartment lab-root, **vm-a와 같은 AD**, Custom size **50 GB**, Target performance **Balanced**. 화면 default 1024 GB를 그대로 제출하지 않는다. Backup policy None, replication Off, reservations Off, Oracle-managed encryption. Create 제출 후 AVAILABLE를 기다린다. [Creating a Block Volume — size/AD/performance](https://docs.oracle.com/en-us/iaas/Content/Block/Tasks/creatingavolume.htm).
3. Compute → vm-a → Storage/Attached Block Volumes → Attach block volume: data-a OCID, Attachment type **Paravirtualized**, access **Read/Write (single instance)**, consistent Device path 중 미사용 `/dev/oracleoci/oraclevdb`를 선택한다(지원되지 않으면 실제 attachment path/serial로 식별한 뒤 경로를 입력). in-transit encryption은 지원 시 On. Attach 후 ATTACHED를 확인한다. iSCSI를 선택하지 않는다. [Paravirtualized attachment and device path](https://docs.oracle.com/en-us/iaas/Content/Block/Tasks/attachingavolume.htm).

### 2. 새 장치를 입증한 뒤에만 format

4. guest에서 attachment의 device path와 `lsblk` 크기/새 장치/serial을 대조한다. 아래 DEVICE는 **새 50 GB disk임을 사람이 OCID/attachment와 대조한 뒤** 입력한다. 자동 `/dev/sdb` 추정은 금지한다. root나 partition/이미 formatted disk면 중단한다.

```bash
export DEVICE='<Console_attachment의_신규_data_disk_경로>'
readlink -f "$DEVICE"
findmnt /
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS "$DEVICE"
sudo wipefs --no-act "$DEVICE"
sudo blkid "$DEVICE" || true
# 위 출력에서 root 아님 / partition 없음 / filesystem signature 없음 / OCID 장치 일치 확인 후만 진행
read -r -p 'Verified NEW data volume OCID and empty non-root disk? Type FORMAT-NEW-DATA: ' CONFIRM
if [ "$CONFIRM" != FORMAT-NEW-DATA ]; then exit 1; fi
if [ "$(lsblk -dn -o TYPE "$DEVICE")" != disk ]; then exit 1; fi
if [ "$(lsblk -nr -o NAME "$DEVICE" | wc -l)" -ne 1 ]; then exit 1; fi
if [ -n "$(lsblk -dn -o FSTYPE "$DEVICE")" ]; then exit 1; fi
if [ -n "$(lsblk -nr -o MOUNTPOINTS "$DEVICE" | tr -d '[:space:]')" ]; then exit 1; fi
SIGNATURES="$(sudo wipefs --no-act --noheadings -o TYPE "$DEVICE")" || exit 1
if [ -n "$SIGNATURES" ]; then printf '%s\n' 'STOP: disk signature exists'; exit 1; fi
REAL_DEVICE="$(readlink -f "$DEVICE")" || exit 1
test -b "$REAL_DEVICE" || exit 1
ROOT_SOURCE="$(findmnt -rn -M / -o SOURCE)" || exit 1
# lsblk parent chain must not contain this new disk. Complex mapper/root layouts need human review.
ROOT_CHAIN="$(lsblk -s -n -r -p -o NAME "$ROOT_SOURCE")" || exit 1
if printf '%s\n' "$ROOT_CHAIN" | grep -Fx "$REAL_DEVICE"; then
  printf '%s\n' 'STOP: device is in root backing chain'; exit 1
fi
MOUNTPOINT=/mnt/roadmap-data
# Exact mount query: rc 1 is no match; all other errors stop.
if test -L "$MOUNTPOINT"; then printf 'STOP: symlink target\n'; exit 1; fi
if test -e "$MOUNTPOINT" && ! test -d "$MOUNTPOINT"; then printf 'STOP: non-directory target\n'; exit 1; fi
if mounted=$(findmnt -rn -M "$MOUNTPOINT" -o TARGET); then
  printf 'STOP: occupied mountpoint %s\n' "$mounted"; exit 1
else
  rc=$?; test "$rc" -eq 1 || { printf 'STOP: mount inspection error %s\n' "$rc"; exit 1; }
fi
if test -d "$MOUNTPOINT"; then
  entries=$(sudo find "$MOUNTPOINT" -mindepth 1 -maxdepth 1 -print -quit) || { printf 'STOP: directory inspection failed\n'; exit 1; }
  test -z "$entries" || { printf 'STOP: nonempty local directory\n'; exit 1; }
fi
sudo mkfs.ext4 "$REAL_DEVICE" || exit 1
# Before mkdir, persist directory ownership with data-volume OCID/DEVICE in registry.
MOUNT_DIR_CREATED_BY14=false
if ! test -d "$MOUNTPOINT"; then
  sudo mkdir -- "$MOUNTPOINT" || exit 1
  MOUNT_DIR_CREATED_BY14=true
fi
printf 'MOUNT_DIR_CREATED_BY14=%s MOUNTPOINT=%s DEVICE=%s\n' "$MOUNT_DIR_CREATED_BY14" "$MOUNTPOINT" "$REAL_DEVICE"
# Save this exact flag/path/device in owner14 durable registry before mount;
# fresh-terminal cleanup re-loads the approved registry, never guesses true.
sudo mount "$REAL_DEVICE" "$MOUNTPOINT" || exit 1
ACTUAL_SOURCE="$(findmnt -rn -M "$MOUNTPOINT" -o SOURCE)" || exit 1
test "$(readlink -f "$ACTUAL_SOURCE")" = "$REAL_DEVICE" || exit 1
test "$(findmnt -rn -M "$MOUNTPOINT" -o FSTYPE)" = ext4 || exit 1
findmnt -rn -M "$MOUNTPOINT" -o SOURCE,TARGET,FSTYPE
set -euo pipefail
printf '%s\n' 'synthetic durable data' | sudo tee /mnt/roadmap-data/check.txt >/dev/null || exit 1
# guest 전용 Bash: 실패한 sha256sum을 빈 성공 증거로 저장하지 않는다.
set -euo pipefail
capture_data_hash() {
  local out="$1" line digest
  test ! -e "$out" && test ! -L "$out" || { printf '%s\n' 'STOP: hash evidence path occupied'; return 1; }
  line=$(sudo sha256sum /mnt/roadmap-data/check.txt) || return 1
  digest="${line%% *}"
  [[ "$digest" =~ ^[0-9a-f]{64}$ ]] || return 1
  test "$line" = "$digest  /mnt/roadmap-data/check.txt" || return 1
  (set -o noclobber; printf '%s\n' "$digest" > "$out") || return 1
  test -s "$out" || return 1
}
capture_data_hash "$HOME/roadmap-data.before.sha256" || exit 1
sync
sudo umount /mnt/roadmap-data || exit 1
```
5. 자동 mount를 위해 fstab를 편집하지 않는다. mount guard는 필요한 안전조건이지 OCI OCID 동일성을 대신 입증하지 않으므로 사람이 대응관계를 확인하는 단계는 생략할 수 없다.

### 3. detach/reattach durability 검증

6. guest unmount 후 Console attached volume → Detach를 제출한다. attachment DETACHED 또는 사라짐, volume AVAILABLE를 확인한다. 같은 data-a OCID를 같은 vm-a에 Paravirtualized/ReadWrite로 다시 Attach한다. filesystem을 다시 만들지 않는다.
7. guest에서 새 attachment의 device path를 다시 대조하고 아래를 실행한다. `cmp`/hash 일치로 데이터 생존을 검증한다. reboot 자동 mount/SLA 증명은 아니다.

```bash
export DEVICE='<reattach에서_확인한_같은_data-volume_device_path>'
REAL_DEVICE="$(readlink -f "$DEVICE")" || exit 1
test -b "$REAL_DEVICE" || exit 1
sudo blkid "$REAL_DEVICE" || exit 1
test "$(lsblk -dn -o FSTYPE "$REAL_DEVICE")" = ext4 || exit 1
MOUNTPOINT="/mnt/roadmap-data"
# Exact mount query: rc 1 is no match; all other errors stop.
if test -L "$MOUNTPOINT"; then printf 'STOP: symlink target\n'; exit 1; fi
if test -e "$MOUNTPOINT" && ! test -d "$MOUNTPOINT"; then printf 'STOP: non-directory target\n'; exit 1; fi
if mounted=$(findmnt -rn -M "$MOUNTPOINT" -o TARGET); then
  printf 'STOP: occupied mountpoint %s\n' "$mounted"; exit 1
else
  rc=$?; test "$rc" -eq 1 || { printf 'STOP: mount inspection error %s\n' "$rc"; exit 1; }
fi
if test -d "$MOUNTPOINT"; then
  entries=$(sudo find "$MOUNTPOINT" -mindepth 1 -maxdepth 1 -print -quit) || { printf 'STOP: directory inspection failed\n'; exit 1; }
  test -z "$entries" || { printf 'STOP: nonempty local directory\n'; exit 1; }
fi
sudo mount "$REAL_DEVICE" /mnt/roadmap-data || exit 1
ACTUAL_SOURCE="$(findmnt -rn -M /mnt/roadmap-data -o SOURCE)" || exit 1
test "$(readlink -f "$ACTUAL_SOURCE")" = "$REAL_DEVICE" || exit 1
test "$(findmnt -rn -M /mnt/roadmap-data -o FSTYPE)" = ext4 || exit 1
findmnt -rn -M /mnt/roadmap-data -o SOURCE,TARGET,FSTYPE
# 같은 guest Bash 세션의 위 capture_data_hash 함수를 사용한다.
# 새 guest 세션이면 위 함수 정의만 다시 읽어 적용하고 before 파일을 덮어쓰지 않는다.
capture_data_hash "$HOME/roadmap-data.after.sha256" || exit 1
python3 - <<'PY'
import pathlib,re,os
for phase in ('before','after'):
    raw=(pathlib.Path(os.environ['HOME'])/f'roadmap-data.{phase}.sha256').read_bytes()
    assert re.fullmatch(rb'[0-9a-f]{64}\n',raw), 'STOP: nonempty exact SHA256 evidence required'
PY
cmp "$HOME/roadmap-data.before.sha256" "$HOME/roadmap-data.after.sha256" || exit 1
findmnt /mnt/roadmap-data
sudo cat /mnt/roadmap-data/check.txt
```

## 결과검증

<a id="baseline-14-ready"></a>

```bash
oci bv volume get --volume-id '<data-a_OCID>' --region "$REGION" > data-volume.json || exit 1
oci compute volume-attachment list --compartment-id "$LAB_COMPARTMENT_ID" --instance-id "$INSTANCE_ID" --region "$REGION" > data-attachments.json || exit 1
```
기대 패턴 volume.size-in-gbs=50/AD=vm-a AD/lifecycle-state=AVAILABLE, attachment ATTACHED/type=paravirtualized/volume-id=data-a/instance-id=vm-a. guest ext4 mounted, check.txt 내용 일치, before/after cmp exit 0. 재연결 후 mkfs 없이 성공해야 한다.

**증거 기록:** 실제 명령·UTC 시간·exit code·가려진 OCID·옵션 readback·검증 조건을 남긴다. 현재 이 문서의 실행상태는 `NOT_EXECUTED`다. output/creation-event-id/result는 실행 전 비워 둔다. 오류/NOT_SELECTED를 PASS로 바꾸지 않는다.

### 이 시나리오의 functional acceptance

| Positive | Negative / 미완료 판정 | Recovery / 반환 |
|---|---|---|
| new volume exact ext4 mount/checksum | occupied mountpoint/signature/root-chain이면 mkfs 이전 STOP | 잘못된 대상에 쓰지 않고 정확한 신규 장치 재대조; same OCID reattach hash 복구 |

모두 실행 시 확인할 기대 패턴이다. 생성 전 guard negative는 OCI 장애를 주입한 것과 다르며 다른 자원을 추가하지 않는다. 오류 메시지와 실제 exit code가 없는 항목은 NOT_EXECUTED 또는 BLOCKED로 남긴다.

## 문제해결

### 공통 오류의 진단 순서

1. `NotAuthorizedOrNotFound` 또는 목록에 리소스가 없음: 먼저 Region/Compartment/OCID와 identity domain/group을 확인한다. 존재하지 않는 OCID와 권한 부족 모두 같은 오류로 보일 수 있다. 관리자에게 request-id·작업 시각·서비스·가려진 대상 OCID를 제공한다. 비밀키/토큰은 보내지 않는다.
2. `LimitExceeded`/quota/capacity 부족: Billing & Cost Management → Limits, Quotas and Usage와 생성 화면의 지원 shape/AD를 확인한다. 운영 quota 증가나 다른 Region 구독을 임의 요청하지 않는다. 승인 범위 안의 더 작은 compatible shape 또는 다른 AD를 선택하거나 `BLOCKED_CAPACITY`를 기록한다.
3. 장시간 provisioning/예상 밖 state: Work Requests/오류 상세를 읽고 같은 요청을 반복 제출하지 않는다. 만들어진 부분 리소스(boot/IP/attachment 포함)를 먼저 조사한다.
4. 메뉴 이름이 다름: 공식 task body의 동등한 탭(예: Storage 또는 Attached Block Volumes)을 찾는다. 필드가 문서와 다르면 값·스크린샷에서 secrets를 가린 뒤 관리자 또는 Oracle Support에 확인하고 제출을 중단한다. 이 문서는 현재 tenancy의 live Console 화면을 검사한 결과가 아니다.

### SSH 실패

`Connection timed out`이면 공인 IP → IGW enabled → subnet route의 IGW target → security list와 모든 NSG의 유효 허용 규칙 → guest sshd/firewall 순으로 확인한다. NSG와 security list는 허용 규칙의 합집합이므로 좁은 NSG가 넓은 security list를 취소하지 않는다. `Permission denied (publickey)`이면 Linux 사용자(opc; 이 트랙은 Oracle Linux 9), 제출한 `.pub`, 대응 private key와 권한을 확인한다. 호스트키 변경은 재생성 여부를 확인한 뒤 처리하며 `StrictHostKeyChecking=no`로 우회하지 않는다. 복구가 필요하면 공식 serial Console 절차를 관리자의 승인하에 사용한다. Ping 불가만으로 SSH 실패를 판정하지 않는다.

장치가 보이지 않음: attachment state/type/같은 AD/consistent path 지원을 확인하고 `lsblk`와 `dmesg`를 비파괴 조회한다. 기존 root disk를 대신 format하지 않는다. mount 실패: `blkid`, filesystem type, 이전 unmount 여부를 확인한다. `wrong fs type`에 mkfs 재실행으로 대응하면 원본 데이터를 잃으므로 중단한다. detach 실패는 먼저 모든 프로세스가 mount를 사용하지 않는지 `sudo fuser -vm /mnt/roadmap-data`로 조사한다. 강제 detach는 데이터 손상 위험 때문에 기본 절차에 없다.

## 리소스정리

guest에서 registry의 exact data OCID/REAL_DEVICE와 `findmnt -rn -M /mnt/roadmap-data -o SOURCE,FSTYPE`의 동일 device/ext4를 확인한 뒤에만 `sudo umount /mnt/roadmap-data`를 수행한다. `findmnt -rn -M /mnt/roadmap-data` rc1(부재)만 허용하고 오류/잔존 mount는 중단한다. registry의 `MOUNT_DIR_CREATED_BY14=true`인 경우에만 regular non-symlink empty directory를 `sudo rmdir /mnt/roadmap-data`로 제거한다; false/미확인이면 기존 directory를 보존한다. Console attachment를 Detach한다. volume AVAILABLE/attachment 해제 확인 후 Storage → Block Volumes → data-a OCID → Terminate/Delete를 제출한다. `oci bv volume get --volume-id '<data-a_OCID>' --region "$REGION"`의 TERMINATED 또는 권한과 구분한 not-found를 기록한다. backup policy를 켜지 않았지만 잘못 생긴 backup/replica도 조회한다. vm-a와 boot는 다른 소비자가 끝날 때까지 생성 owner가 보존한다. 로컬 guest hash 파일 두 개만 삭제한다.


## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 아래 본문은 각 생성 필드·설정·검증·정리의 기술 근거이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [8] [Overview of Block Volume — Block Volume](https://docs.oracle.com/en-us/iaas/Content/Block/Concepts/overview.htm)
  - 확인 범위: Block volume: / Scenario B: Persistent.

### 추가 실행 절차 근거 (본문 확인)

- [block](https://docs.oracle.com/en-us/iaas/Content/Block/Tasks/creatingavolume.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [attach](https://docs.oracle.com/en-us/iaas/Content/Block/Tasks/attachingavolume.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [policy](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [cloudshell](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/cloudshellgettingstarted.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.

- [추가 세부 task: format](https://docs.oracle.com/en-us/iaas/Content/Block/Tasks/connectingtoavolume.htm) — 실제 본문을 확인한 필드·API/권한·삭제 단계 근거.
