> **문서 검토 완료 — OCI 실행 미검증**: 이 Snapshots 절차는 지정된 수정 범위에서 문서 검토를 완료했습니다. 전체 교육과정 승인이나 실제 OCI 실행 검증을 뜻하지 않습니다.

# Snapshots

> 문서 검토 범위: 이 파일의 시나리오·작업 순서·복구·정리를 검토했습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 권한·안전·비용·복구 등 실제 실행 조건은 그대로 적용됩니다. 전체 교육과정 및 게시 상태는 [통합 상태](../../../Docs/README.md)를 확인하세요.

> 절차 작성본 / OCI 미실행 — 아래 예상값은 실측 결과가 아닙니다. 비용·권한·사전준비 gate를 통과한 전용 환경에서만 실행하세요.

## 학습목표

- 기존 NFS mount의 lab47/data.txt를 작성하고 lab47-before Snapshot 하나를 생성한다.
- 현재 after-snapshot과 snapshot before-snapshot을 대조하고 restored.txt 복원/cmp 및 snapshot RO 오류를 검증한 뒤 이번 snapshot/파일만 삭제한다.

완료 기준은 아래 실제 생성·설정·요청 결과로 판정한다. 이 목표는 수행할 작업이며 OCI에서 관찰한 결과가 아니다.

## 대응 Roadmap 항목

- 항목명: **Snapshots**
- Roadmap 경로: **File Storage → Snapshots**
- Node ID: `service-file-4`
- 상위 항목: **File Storage** (`service-file-0`)
- 문서 범위: 이 항목의 개념과 작성된 실습절차. 지정 수정 범위의 문서 검토 완료이며 실제 OCI 실행은 NOT_EXECUTED입니다.
- [전체 항목 목록](../../../Docs/README.md)

## 사전준비


43의 실제 FILE_SYSTEM_ID/MT_PRIVATE_IP와22의 기존 B guest를 받는다.45 반환 뒤46의 B/32/RW/SYS/Squash=None 및 literal MT_IP:/lab+nfs same-mount write 성공이 필요하다.46 미선택이면43 owner가 같은 positive를 반환한다. FSS_MOUNT는 승인된 기존 /lab export root이고 SNAPSHOT_NAME=lab47-before다; 새 Compute를 생성하지 않는다.

- [Exports](02_Exports.md#baseline-45-ready) 완료/반환 후 [Export Options](03_Export_Options.md#리소스정리)의 owner46 RW/Squash=None/SYS·B/32 및 같은 `/lab` mount 쓰기 성공 반환을 받는다. 46 NOT_SELECTED이면43 owner가 같은 설정 readback과 same-mount positive write를 직접 반환한다; RO/미확인 상태로47을 시작하지 않는다.
- [File Storage](00_File_Storage.md#baseline-43-ready)의 actual FSS-mounted와 [Private Subnets의 기존 client](../../Networking/Original-Provisional/03_Virtual_Cloud_Network_VCN/03_Private_Subnets.md#baseline-22-ready)를 재사용한다. 새 Compute 생성은 하지 않는다.




## 권한

CreateSnapshot → FILE_SYSTEM_CREATE_SNAPSHOT; GetSnapshot/ListSnapshots → FILE_SYSTEM_READ; DeleteSnapshot → FILE_SYSTEM_DELETE_SNAPSHOT. OS의 NFS 읽기/쓰기 권한은 IAM과 별개다.

[API operation별 공식 permission table](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/filestoragepolicyreference.htm) — 가족 단위 manage를 최소권한이라 부르지 않는다. 전용 lab compartment의 group만 적용하고 정책작성 권한은 관리자 소유다.
```text
Allow group <identity-domain>/<lab-group> to manage file-systems in compartment <lab-compartment> where any {request.permission='FILE_SYSTEM_CREATE_SNAPSHOT', request.permission='FILE_SYSTEM_READ', request.permission='FILE_SYSTEM_DELETE_SNAPSHOT'}
```
이 allowlist는 위 작업만 허용한다. Console 탐색 List API가 거부되면 실제 API permission을 공식 표에서 별도로 승인하며 all-resources로 확대하지 않는다.

## 비용및주의사항

owner43의 FS/표준 MT/Export와 owner22의 기존 NFS client를 재사용한다. 신규 범위는 `lab47-before` Snapshot 하나와 lab47의 작은 data/restored 파일뿐이다. 원본 FSS 저장량에 더해 snapshot 보존/변경 데이터 저장 비용과 실제 전송, client VM/boot 유지 비용을 승인한다. Snapshot Policy/Replication/두 번째 FS는 만들지 않는다. 승인 lease 안에 파일 복원·비교 후 이번 snapshot/파일만 정리하며 전체 FS 삭제는 owner43에게 반환한다.

## 실습구성

- 작업 목표: NFS로 마운트된 파일 시스템에서 전용 디렉터리와 snapshot 1개를 만든다.
- 생성 owner: 47. 동일 owner의 자원을 재사용하며 실제 결과 필드는 실행 전 비워 둔다.


### 이번 workshop 시나리오와 인수·반환

현재 NFS 파일과 시점 복사본을 분리하여 파일 하나를 복원한다.

- **신규/변경:** snapshot lab47-before 1개; lab47/data.txt 및 restored.txt.
- **선행 재사용:** 43 File System/MT/Export,22 private-vm NFS client.
- **완주 흐름:** 승인/입력→본문의 생성/편집→현재 after-snapshot / snapshot before-snapshot / 복원 cmp0 / snapshot write 거부→47 snapshot·lab47 파일 삭제→43 mount/FS 반환.
- **선택 이유/실패 경계:** mount 아닌 local directory의 false success와 root-owned mount UID를 먼저 차단한다.


- 입력/산출물: `FSS_MOUNT`
  - 실제 값 출처: 본문 단계에서 승인/선택한 값
  - 준비 시점: 사용 전

- 입력/산출물: `MT_PRIVATE_IP`
  - 실제 값 출처: creator의 실제 생성/조회 출력; 임의 OCID/IP 금지
  - 준비 시점: 사용 전

- 입력/산출물: `FILE_SYSTEM_ID`
  - 실제 값 출처: creator의 실제 생성/조회 출력; 임의 OCID/IP 금지
  - 준비 시점: 사용 전

- 입력/산출물: `SNAPSHOT_NAME`
  - 실제 값 출처: 본문 단계에서 승인/선택한 값
  - 준비 시점: 사용 전


입력별 실행위치는 본문 터미널 구분을 따른다: OCI CLI=인증 Cloud Shell/승인 OCI 터미널, OS 앱/NFS/부하=지정 guest, private HTTP/TCP=vm-a guest, YAML 작성/SSH private key=본인 local. 조회/변경/정리의 관리자 승인은 **실행 조건**이며 이 문서는 IAM 성공을 입증하지 않는다. 실제 허용/거부와 capacity는 사용자 실행에서 확인한다. 미승인/NOT_SELECTED/ASYNC_PENDING을 성공으로 바꾸지 않는다.

## 실습절차

### 실행 터미널과 지역 고정
Snapshot47의 OS 명령은 기존 NFS client guest에서만, OCI 명령은 승인 인증 터미널에서 수행한다.
```bash
export REGION="<approved-region>"
export OCI_CLI_REGION="$REGION"
export LAB_COMPARTMENT_ID="<actual-lab-compartment-ocid>"
export EXPIRY_UTC="<approved-session-end-UTC>"
```
치환 전 실행 금지. 생성된 OCID/IP/원래 설정/새 rule ID/owner/종료시각을 비밀 없는 registry에 기록하고 새 터미널에서는 필요한 변수만 복원한다.

[아래 생성/변경 단계의 정확한 Oracle task 본문](https://docs.oracle.com/en-us/iaas/Content/File/Tasks/create-snapshot.htm) — Console field/CLI/API 동작 근거.

아래 단계는 Oracle task 본문을 근거로 작성한 lab recipe이며 공식 실행결과가 아니다. 각 단계의 근거는 끝의 정확한 task 링크를 따른다.

### 첫 파일 mutation 전 재개·복구 기록
개인 실습은 별도 관리 terminal, 공유 실습은 owner43 복구담당자를 열어 둔다. [43 residual-kernel station](00_File_Storage.md#c-실제-linux-mount와-공유-데이터)의 통신/options 복구와 D-state·잔존 요청 확인을 먼저 읽는다. NFS 데이터 작업과 경로 검사에는 아래 timeout 요청을 사용한다. timeout은 kernel 종료 보장이 아니므로 실제 호출이 반환하지 않으면 다른 관리 terminal에서 복구한다. hard mount를 soft로 바꾸지 않는다. 실패 시 새 I/O/스냅샷·endpoint 삭제를 멈추고 원래 B/32 RW/MT 통신 복구→요청 종료 확인→부분 정리를 진행한다.

- B guest의 로컬(해당 NFS mount 밖) private 기록에 먼저 run 식별자와 입력을 보존한다.
- 같은 lab47 경로에는 이 run만 접근하도록 owner43과 정한다.
- 첫 mkdir 전 저장할 private 기록: FILE_SYSTEM_ID/MT_PRIVATE_IP/FSS_MOUNT/REGION/EXPIRY_UTC, LAB47_DIR_CREATED=0, LAB47_DIR_PENDING=0, SNAPSHOT_ID=NONE, SNAPSHOT_DIAG_DIR=NONE.
- 충돌 검사를 통과한 직후 mkdir 전 LAB47_DIR_PENDING=1을 저장한다.
- mkdir 성공 직후 LAB47_DIR_CREATED=1/LAB47_DIR_PENDING=0을 저장하고 snapshot Create 응답 직후 **exact snapshot OCID/FS ID/name/created-by47**을 저장한다.
- 응답 유실이면 같은 FS snapshot 목록에서 생성 시각/name을 관리자가 대조할 때까지 재Create/추측 삭제하지 않는다.
- Diagnostic mktemp 직후 exact 경로도 저장한다.
- fresh terminal은 이 기록의 입력을 다시 export하고 아래 부분 정리 station으로 이동한다.
- PENDING=1이면 재mkdir하지 않는다.
- owner43과 잔존 요청 종료 및 독점 실행 기록을 대조하여 이번 mkdir의 성공 여부/소유권을 먼저 확정한다.
- 확정할 수 없으면 BLOCKED_RECOVERY로 남긴다.

1. 아래 Mount 변수 첫 bash block을 실행해 실제 mount/전용 파일을 준비한다. 같은 작업을 두 번 수행하지 않는다.
2. **Create 이전** 같은 FILE_SYSTEM_ID의 Console Snapshots 목록 전체에서 lab47-before가 없음을 확인한다. 조회 실패/동명이름 존재면 중단하고 기존 snapshot을 재사용/삭제하지 않는다. Console → File Storage → File Systems → 실제 FILE_SYSTEM_ID → Snapshots → Create Snapshot. Name=lab47-before, expiration은 RO/복원 관측을 마치기 전에 snapshot을 만료시키지 않는 승인 시각을 선택하거나 이번 짧은 시험에서는 미선택 후 수동 삭제한다. Snapshot Policy는 새로 생성/연결하지 않는다. 선택값을 저장하고 생성 상태를 읽는다. Active 확인 전 두 번째 block을 실행하지 않는다.
3. 아래 두 번째 bash block의 원본 변경/별도 restored.txt 복구/cmp/RO negative control을 수행한다. 사용 경로는 mount root의 .snapshot이다.

### Mount 변수로 끝까지 실행
```bash
export FSS_MOUNT='<actual-mounted-export-root>'
export SNAPSHOT_NAME='lab47-before'
set -euo pipefail
export MT_PRIVATE_IP='<owner43-actual-mount-target-ip>'
mountpoint -q "$FSS_MOUNT" || { printf 'STOP: not a mountpoint\n'; exit 1; }
test "$(findmnt -rn -M "$FSS_MOUNT" -o SOURCE)" = "$MT_PRIVATE_IP:/lab" || exit 1
test "$(findmnt -rn -M "$FSS_MOUNT" -o FSTYPE)" = nfs || exit 1
# owner46 must first return RW/Squash=None/SYS and a same-mount positive write.
timeout -k 5s 10s sh -eu -c '
  test ! -e "$1/lab47" && test ! -L "$1/lab47"
' sh "$FSS_MOUNT" || exit 1
# private 기록에 LAB47_DIR_PENDING=1 저장 후만 mkdir 실행
sudo timeout -k 5s 10s mkdir "$FSS_MOUNT/lab47"
# 성공 직후 private 기록의 LAB47_DIR_CREATED=1/LAB47_DIR_PENDING=0 저장
sudo timeout -k 5s 10s chown "$(id -u):$(id -g)" "$FSS_MOUNT/lab47"
timeout -k 5s 10s sh -c 'printf "before-snapshot\n" > "$1/lab47/data.txt"' sh "$FSS_MOUNT"
timeout -k 5s 10s sync "$FSS_MOUNT/lab47/data.txt"
```
위 1단계에서 이 첫 block을 실행한다. Console snapshot 생성/Active 뒤 아래를 실행한다.
```bash
set -euo pipefail
# B guest의 NFS 밖 private 로컬 작업 directory로 cd한 뒤 실행한다.
SNAPSHOT_DIAG_DIR=$(mktemp -d "$PWD/oci-lab47.XXXXXXXX") || exit 1
# private 기록에 exact SNAPSHOT_DIAG_DIR을 즉시 저장한다.
mountpoint -q "$FSS_MOUNT" || exit 1
test "$(findmnt -rn -M "$FSS_MOUNT" -o SOURCE)" = "$MT_PRIVATE_IP:/lab" || exit 1
test "$(findmnt -rn -M "$FSS_MOUNT" -o FSTYPE)" = nfs || exit 1
timeout -k 5s 10s sh -c 'printf "after-snapshot\n" > "$1/lab47/data.txt"' sh "$FSS_MOUNT"
timeout -k 5s 10s cp "$FSS_MOUNT/.snapshot/$SNAPSHOT_NAME/lab47/data.txt" "$FSS_MOUNT/lab47/restored.txt"
printf 'before-snapshot\n' | timeout -k 5s 10s cmp - "$FSS_MOUNT/lab47/restored.txt"
timeout -k 5s 10s test -d "$FSS_MOUNT/.snapshot/$SNAPSHOT_NAME/lab47" || exit 1
timeout -k 5s 10s test -r "$FSS_MOUNT/.snapshot/$SNAPSHOT_NAME/lab47/data.txt" || exit 1
RO_RC=0
timeout -k 5s 10s env LC_ALL=C touch "$FSS_MOUNT/.snapshot/$SNAPSHOT_NAME/lab47/should-not-write" 2> "$SNAPSHOT_DIAG_DIR/snapshot-ro-error.txt" || RO_RC=$?
# touch의 정상적인 RO 거부(exit 1)만 후보로 인정; timeout/signal은 복구로 이동한다.
if test "$RO_RC" -eq 0; then
  printf 'ERROR: snapshot was writable\n'; exit 1
fi
test "$RO_RC" -eq 1 || { printf 'STOP: transport/timeout; residual recovery required\n'; exit 1; }
grep -F 'Read-only file system' "$SNAPSHOT_DIAG_DIR/snapshot-ro-error.txt" || { printf 'inconclusive: expiry/POSIX/transport failure is not RO proof\n'; exit 1; }
timeout -k 5s 10s test -r "$FSS_MOUNT/.snapshot/$SNAPSHOT_NAME/lab47/data.txt" || exit 1
printf 'after-snapshot\n' | timeout -k 5s 10s cmp - "$FSS_MOUNT/lab47/data.txt"
```
RO 실패는 같은 mount 현재파일 write 성공과 짝을 이룬다. IAM 거부와 NFS RO를 혼동하지 않는다.

## 결과검증

snapshot에서는 before-snapshot, 현재 파일에서는 after-snapshot 문자열을 확인한다. Snapshot 삭제 전 restored.txt는 snapshot bytes(before-snapshot)와 같고, 변경된 원본 data.txt(after-snapshot)와는 달라야 한다. 두 파일의 equality를 기대하지 않는다.

- 예상 pattern이며 실제 OCI 실행 결과가 아니다. timestamp, resource OCID, 설정 readback, 실제 HTTP/TCP/파일 결과를 실행자가 기록한다.

## 문제해결

snapshot 경로의 ENOENT는 mount root와 snapshot 이름/상태를 확인한다. NFS permission denied는 export options/UID/mount 권한 문제이므로 IAM을 확대하지 않는다.

실패 기록에는 같은 target/credentials/region/요청 시간을 고정한다. 복구는 원래 설정으로 실행하고 actual response/readback을 둘 다 확인한다. 예상값을 실측으로 쓰지 않는다.

## 리소스정리

### 부분 실패와 fresh-terminal 정리
먼저 private 기록의 변수/created flags를 다시 설정하고 owner43과 요청 종료/D-state 없음, 원래 RW/options/통신 복구를 확인한다. mounted exact SOURCE/FSTYPE guard를 다시 통과한 뒤 **LAB47_DIR_CREATED=1인 경우만** 아래 존재하는 owned 파일을 지운다. Snapshot 생성 실패나 cp 실패 때 없는 파일은 건너뛴다. symlink/예상 밖 타입은 중단한다.

아래는 B guest에서 실행한다. `LAB47_DIR_CREATED`는 기록의 실제 0/1 값만 사용한다. PENDING 미해결이면 실행 금지다. mount guard는 로컬 mount table을 확인하고, 경로/타입 검사와 삭제는 하나의 timeout 안에서 수행한다. 디렉터리 자체의 symlink도 거부한다.

```bash
set -euo pipefail
: "${FSS_MOUNT:?}" "${MT_PRIVATE_IP:?}" "${LAB47_DIR_CREATED:?}"
: "${LAB47_DIR_PENDING:?}"
test "$LAB47_DIR_PENDING" = 0 || exit 1
case "$LAB47_DIR_CREATED" in 0|1) ;; *) exit 1 ;; esac
mountpoint -q "$FSS_MOUNT" || exit 1
test "$(findmnt -rn -M "$FSS_MOUNT" -o SOURCE)" = "$MT_PRIVATE_IP:/lab" || exit 1
test "$(findmnt -rn -M "$FSS_MOUNT" -o FSTYPE)" = nfs || exit 1
if test "$LAB47_DIR_CREATED" = 1; then
  timeout -k 5s 10s sh -eu -c '
    dir="$1/lab47"
    test ! -L "$dir" || exit 1
    if test -e "$dir"; then
      test -d "$dir" || exit 1
      for name in data.txt restored.txt; do
        file="$dir/$name"
        test ! -L "$file" || exit 1
        if test -e "$file"; then
          test -f "$file" || exit 1
          rm -- "$file" || exit 1
        fi
      done
      rmdir -- "$dir" || exit 1
    fi
  ' sh "$FSS_MOUNT" || exit 1
  LAB47_DIR_CREATED=0
  # private 기록에도 즉시 LAB47_DIR_CREATED=0 저장
fi
```
위 block이 실패/timeout이면 residual 복구로 돌아가며 snapshot/endpoint를 삭제하지 않는다. rmdir 뒤 기록 저장 전 세션이 끊겨도 재개 시 디렉터리 부재는 허용한다. 예상 밖 파일은 rmdir 실패로 보존하며 recursive/glob 삭제하지 않는다.

다음 Console → exact FILE_SYSTEM_ID → Snapshots에서 기록된 SNAPSHOT_ID의 이름/FS/created-by47을 대조해 Delete하고 새 목록에서 OCID 부재를 확인한다. 삭제 재개 시에도 조회 성공/필터 범위를 확인한 exact OCID 부재만 완료로 인정한다. SNAPSHOT_ID=NONE이면 생성 여부 조사를 끝내기 전 추측 삭제하지 않는다. 생성 시도 자체가 없었다면 그 기록으로 snapshot 정리를 건너뛴다.

Diagnostic은 기록된 mktemp directory가 이번 run 소유인 경우만 아래로 정리한다. `NONE`이면 생성하지 않았다는 기록을 확인하고 건너뛴다.

```bash
: "${SNAPSHOT_DIAG_DIR:?}"
if test "$SNAPSHOT_DIAG_DIR" != NONE; then
  test ! -L "$SNAPSHOT_DIAG_DIR" || exit 1
  if test -e "$SNAPSHOT_DIAG_DIR"; then
    test -d "$SNAPSHOT_DIAG_DIR" || exit 1
    diag="$SNAPSHOT_DIAG_DIR/snapshot-ro-error.txt"
    test ! -L "$diag" || exit 1
    if test -e "$diag"; then
      test -f "$diag" || exit 1
      rm -- "$diag" || exit 1
    fi
    rmdir -- "$SNAPSHOT_DIAG_DIR" || exit 1
  fi
  SNAPSHOT_DIAG_DIR=NONE
  # private 기록에도 즉시 SNAPSHOT_DIAG_DIR=NONE 저장
fi
```
Snapshot 부재/owned files 부재/잔존 요청 없음과 기존 mount를43에 반환한다. FILE_SYSTEM_ID/export/mounttarget는47에서 삭제하지 않는다. owner22의 B도 여기서 종료하지 않는다.

정리 실행자의 실제 UTC/OCID/readback을 기록한다. 공유 resource는 owner 확인 전 삭제하지 않는다. stop만으로 boot volume/LB/image 비용이 종료되는 것은 아니다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 목표는 학습 후 설명·구분·확인할 내용이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [19] [Overview of File Storage — Snapshots](https://docs.oracle.com/en-us/iaas/Content/File/Concepts/filestorageoverview.htm)
  - 확인 범위: Snapshots.

### 절차별 공식 task body 추가 근거
- [filestorageoverview.htm](https://docs.oracle.com/en-us/iaas/Content/File/Concepts/filestorageoverview.htm) — 본문 검토 근거; 아래 정확한 task와 작성한 입력을 구분한다. OCI 실행 증거가 아니다.
- [managingsnapshots.htm](https://docs.oracle.com/en-us/iaas/Content/File/Tasks/managingsnapshots.htm) — 본문 검토 근거; 아래 정확한 task와 작성한 입력을 구분한다. OCI 실행 증거가 아니다.



### 이번 절차의 정확한 공식 task
- [create-snapshot.htm](https://docs.oracle.com/en-us/iaas/Content/File/Tasks/create-snapshot.htm) — 이 문서 실습절차의 입력/동작을 대조한 본문.
- [restore-from-snapshot.htm](https://docs.oracle.com/en-us/iaas/Content/File/Tasks/restore-from-snapshot.htm) — 이 문서 실습절차의 입력/동작을 대조한 본문.
