> **기존 실습 초안 — 미검증**: 원래 93개 문서의 절차를 보존한 자료입니다. 새 튜토리얼의 품질 승인이나 OCI 실행 검증을 뜻하지 않습니다.

# Snapshots

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../../../Docs/README.md)를 확인하세요.

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
- 문서 범위: 이 항목의 개념과 작성된 실습절차. 통합 품질검토는 BLOCKED이며 실제 실행은 NOT_EXECUTED입니다.
- [전체 항목 목록](../../../Docs/README.md)

## 사전준비


43의 실제 FILE_SYSTEM_ID/MT_PRIVATE_IP와22의 기존 B guest를 받는다.45 반환 뒤46의 B/32/RW/SYS/Squash=None 및 literal MT_IP:/lab+nfs same-mount write 성공이 필요하다.46 미선택이면43 owner가 같은 positive를 반환한다. FSS_MOUNT는 승인된 기존 /lab export root이고 SNAPSHOT_NAME=lab47-before다; 새 Compute를 생성하지 않는다.

- [Exports](02_Exports.md#baseline-45-ready) 완료/반환 후 [Export Options](03_Export_Options.md#리소스정리)의 owner46 RW/Squash=None/SYS·B/32 및 같은 `/lab` mount 쓰기 성공 반환을 받는다. 46 NOT_SELECTED이면43 owner가 같은 설정 readback과 same-mount positive write를 직접 반환한다; RO/미확인 상태로47을 시작하지 않는다.
- [Compute](../02_Compute/00_Compute.md#실습절차) — 링크의 `실습절차`와 `결과검증` 완료; 실제 OCID/IP를 registry에 인계한다.




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

| 입력/산출물 | 실제 값 출처 | 준비 시점 |
|---|---|---|
| `FSS_MOUNT` | 본문 단계에서 승인/선택한 값 | 사용 전 |
| `MT_PRIVATE_IP` | creator의 실제 생성/조회 출력; 임의 OCID/IP 금지 | 사용 전 |
| `FILE_SYSTEM_ID` | creator의 실제 생성/조회 출력; 임의 OCID/IP 금지 | 사용 전 |
| `SNAPSHOT_NAME` | 본문 단계에서 승인/선택한 값 | 사용 전 |

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
치환 전 실행 금지. 생성된 OCID/IP/원래 설정/새 rule ID/owner/종료시각을 비밀 없는 registry에 기록하고 다음 터미널에서는 필요한 변수만 다시 설정한다.

[아래생성/변경단계의정확한Oracle task본문](https://docs.oracle.com/en-us/iaas/Content/File/Tasks/create-snapshot.htm) — Consolefield/CLI/API동작근거.

아래 단계는 Oracle task 본문을 근거로 작성한 lab recipe이며 공식 실행결과가 아니다. 각 단계의 근거는 끝의 정확한 task 링크를 따른다.

1. 아래 Mount 변수 첫 bash block을 실행해 실제 mount/전용 파일을 준비한다. 같은 작업을 두 번 수행하지 않는다.
2. **Create 이전** 같은 FILE_SYSTEM_ID의 Console Snapshots 목록 전체에서 lab47-before가 없음을 확인한다. 조회 실패/동명이름 존재면 중단하고 기존 snapshot을 재사용/삭제하지 않는다. Console → File Storage → File Systems → 실제 FILE_SYSTEM_ID → Snapshots → Create Snapshot. Name=lab47-before, expiration은 RO/복원 관측을 마치기 전에 snapshot을 만료시키지 않는 승인 시각을 선택하거나 이번 짧은 시험에서는 미선택 후 수동 삭제한다. Snapshot Policy는 새로 생성/연결하지 않는다. 선택값을 저장하고 생성 상태를 읽는다. Active 확인 전 두 번째 block을 실행하지 않는다.
3. 아래 두 번째 bash block의 원본 변경/별도 restored.txt 복구/cmp/RO negative control을 수행한다. mount root의 .snapshot 경로를 사용한다.

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
test ! -e "$FSS_MOUNT/lab47" && test ! -L "$FSS_MOUNT/lab47" || exit 1
sudo mkdir "$FSS_MOUNT/lab47"
sudo chown "$(id -u):$(id -g)" "$FSS_MOUNT/lab47"
printf 'before-snapshot\n' > "$FSS_MOUNT/lab47/data.txt"
sync
```
위 1단계에서 이 첫 block을 실행한다. Console snapshot 생성/Active 뒤 아래를 실행한다.
```bash
set -euo pipefail
SNAPSHOT_DIAG_DIR=$(mktemp -d "$PWD/oci-lab47.XXXXXXXX") || exit 1
mountpoint -q "$FSS_MOUNT" || exit 1
test "$(findmnt -rn -M "$FSS_MOUNT" -o SOURCE)" = "$MT_PRIVATE_IP:/lab" || exit 1
test "$(findmnt -rn -M "$FSS_MOUNT" -o FSTYPE)" = nfs || exit 1
printf 'after-snapshot\n' > "$FSS_MOUNT/lab47/data.txt"
cp "$FSS_MOUNT/.snapshot/$SNAPSHOT_NAME/lab47/data.txt" "$FSS_MOUNT/lab47/restored.txt"
printf 'before-snapshot\n' | cmp - "$FSS_MOUNT/lab47/restored.txt"
test -d "$FSS_MOUNT/.snapshot/$SNAPSHOT_NAME/lab47" || exit 1
test -r "$FSS_MOUNT/.snapshot/$SNAPSHOT_NAME/lab47/data.txt" || exit 1
if LC_ALL=C touch "$FSS_MOUNT/.snapshot/$SNAPSHOT_NAME/lab47/should-not-write" 2> "$SNAPSHOT_DIAG_DIR/snapshot-ro-error.txt"; then
  printf 'ERROR: snapshot was writable\n'; exit 1
fi
grep -F 'Read-only file system' "$SNAPSHOT_DIAG_DIR/snapshot-ro-error.txt" || { printf 'inconclusive: expiry/POSIX/transport failure is not RO proof\n'; exit 1; }
test -r "$FSS_MOUNT/.snapshot/$SNAPSHOT_NAME/lab47/data.txt" || exit 1
printf 'after-snapshot\n' | cmp - "$FSS_MOUNT/lab47/data.txt"
```
RO 실패는 같은 mount 현재파일 write 성공과 짝을 이룬다. IAM 거부와 NFS RO를 혼동하지 않는다.

## 결과검증

snapshot에서는 before-snapshot, 현재 파일에서는 after-snapshot 문자열을 확인한다. Snapshot 삭제 전 restored.txt는 snapshot bytes(before-snapshot)와 같고, 변경된 원본 data.txt(after-snapshot)와는 달라야 한다. 두 파일의 equality를 기대하지 않는다.

- 예상 pattern이며 실제 OCI 실행 결과가 아니다. timestamp, resource OCID, 설정 readback, 실제 HTTP/TCP/파일 결과를 실행자가 기록한다.

## 문제해결

snapshot경로ENOENT는mountroot와snapshot이름/상태확인. NFSpermissiondenied는exportoptions/UID/mount권한이며IAM확대하지않는다.

실패기록에는같은target/credentials/region/요청시간을고정한다. 복구는원래설정으로실행하고actualresponse/readback둘다확인한다;예상값을실측으로쓰지않는다.

## 리소스정리

guest는 위 exact mount/source/type guard를 다시 성공시킨 뒤만 파일을 삭제한다. owner43은47 반환 전 unmount를 금지한다. `rm -- "$FSS_MOUNT/lab47/data.txt" "$FSS_MOUNT/lab47/restored.txt"; rmdir "$FSS_MOUNT/lab47"`. Console snapshot lab47-before Delete→Snapshots list에서없음확인. FILE_SYSTEM_ID/export/mounttarget는owner43보존. 이번 mktemp 산출물만 `rm -- "$SNAPSHOT_DIAG_DIR/snapshot-ro-error.txt"; rmdir -- "$SNAPSHOT_DIAG_DIR"`로 정리한다. 실패 도중에도 이 owned diagnostic 경로를 registry에서 찾아 정리하며 기존 동명파일은 건드리지 않는다.

정리실행자의실제UTC/OCID/readback을기록한다. 공유resource는owner확인전삭제금지; stop만으로bootvolume/LB/image비용종료가아니다.

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
