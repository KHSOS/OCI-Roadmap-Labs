> **기존 실습 초안 — 미검증**: 원래 93개 문서의 절차를 보존한 자료입니다. 새 튜토리얼의 품질 승인이나 OCI 실행 검증을 뜻하지 않습니다.

# Exports

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../../../Docs/README.md)를 확인하세요.

> **저술 완료 · OCI 실행 미검증** — Oracle 공식 작업 문서를 근거로 작성한 한국어 hands-on입니다. Oracle 공인 교육/실행 인증이 아니며 기대 결과와 실제 결과를 구분합니다. 비용 승인과 실제 선행 산출물 확인 후에만 실행하세요.

## 학습목표

- 같은 File System/MT에 /lab-alt Export 하나를 생성하고 두 exact mount에서 동일 proof/shared file을 확인한다.
- 미존재 path 신규 mount를 대조한 뒤 alt unmount/Export 삭제 후 원래 /lab 읽기가 유지됨을 확인한다.

완료 기준은 아래 실제 생성·설정·요청 결과로 판정한다. 이 목표는 수행할 작업이며 OCI에서 관찰한 결과가 아니다.

## 대응 Roadmap 항목

- 항목명: **Exports**
- Roadmap 경로: **File Storage → Exports**
- Node ID: `service-file-2`
- 상위 항목: **File Storage** (`service-file-0`)
- 문서 범위: 공식 개념 목표와 실제 구성·허용/불허 검증·정리 절차. 아래 실습은 저술되었으나 OCI에서 실행 검증되지 않았습니다.
- [전체 항목 목록](../../../Docs/README.md)

## 사전준비

43 FSS-mounted와44 MT-tested의 원래 literal MT_IP:/lab mount·proof hash·FS_ID/MT_ID/EXPORT_SET_ID/B_PRIVATE_IP를 받는다.44가 제거한 ingress는 복원/readback돼 있어야 한다. 이 문서는 새 alt Export만 소유하며46 진입 전 alt unmount/Export 삭제 상태를 반환한다.

### 기준 계약과 실제 단계 선행조건

[공통 baseline-contract.json](../../../baseline-contract.json)의 OS/CIDR/소유권을 따른다. Oracle Linux 9 / `opc` / `dnf` 외의 숨은 Ubuntu 또는 probe VM은 만들지 않는다. OCID/IP의 null은 실행 전 반드시 실제 출력으로 채우는 **실행 차단 입력**이다. approval/capacity/quota/identity 미확인은 완료가 아니다.

- [43.ready File Storage — FUNCTIONAL_COMPLETION](00_File_Storage.md#baseline-43-ready): 표시된 FUNCTIONAL_COMPLETION은 해당 linked ready 단계의 실제 검증/반환이 필요하다. PRE_CREATE_OR_INSTALL_PREPARATION으로 표시한 링크만 준비 출력까지 요구한다. 필요한 OCID/주소만 인계한다.
- [44.ready Mount Targets — FUNCTIONAL_COMPLETION](01_Mount_Targets.md#baseline-44-ready): 표시된 FUNCTIONAL_COMPLETION은 해당 linked ready 단계의 실제 검증/반환이 필요하다. PRE_CREATE_OR_INSTALL_PREPARATION으로 표시한 링크만 준비 출력까지 요구한다. 필요한 OCID/주소만 인계한다.

공유 A/B consumer는 INSTANCE_A_ID에 owner10 vm-a의 실제 OCID, INSTANCE_B_ID에 owner22 private-vm의 실제 OCID, `A_PUBLIC_IP`, `A_PRIVATE_IP`, `B_PRIVATE_IP`를 실제 생성 응답에서 전달받는다. 해당 주제에 필요하지 않은 VM/변수는 생성·요구하지 않는다.

- [File Storage](../../../Docs/05_File_Storage/00_File_Storage.md) **FSS-mounted**, [Mount Targets](../../../Docs/05_File_Storage/01_Mount_Targets.md) **MT-tested** 완료 후 NFS security 복원. File System/MT/proof hash 유지.
- 추가 Export는 같은 File System과 같은 MT에 만들고46 전 두 Export의 source/access를 모두 확인한다. 두 번째 MT/File System을 만들지 않는다.

## 권한

### 실제 task/API별 공식 권한 본문 대조

단계2가 생성 후 options editor 저장을 사용하면 `UpdateExport` 작업이다. live 공식 operation 표에 이 행은 없으며 `UpdateExportSet`과 동일 API가 아니다. 최소 permission 대응은 미입증 상태로 유지한다. 조직 관리자가 exact Export OCID/scope의 변경·복구 권한을 명시 승인하면 조건부 저술 절차를 실행할 수 있다; 승인 없으면 BLOCKED_PERMISSION이다. [46 권한 범위](03_Export_Options.md#권한)는 참고하되 그 문서의 미입증 gate를 반복 조회로 해결했다고 주장하지 않는다. CreateExport 권한만으로 후속 editor 수정 권한이 확인됐다고 보고하지 않는다.

아래 공식 operation 표에서 이 실습의 읽기·변경·정리 권한을 대조한다. family grant/일부 dependency 인용은 검증된 minimum policy가 아니며 실제 principal과 resource scope를 관리자가 승인해야 한다.

- `CreateExport`: EXPORT_SET_UPDATE + FILE_SYSTEM_NFSv3_EXPORT — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/filestoragepolicyreference.htm).
- `DeleteExport`: EXPORT_SET_UPDATE + FILE_SYSTEM_NFSv3_UNEXPORT — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/filestoragepolicyreference.htm).
- `GetExport`: EXPORT_SET_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/filestoragepolicyreference.htm).

이번 Export creator는 위 CreateExport/GetExport/DeleteExport를 승인받는다. FS/MT 생성·삭제와 shared network 변경 권한은 필요하지 않으며 생성 후 options 편집은 위 UpdateExport 별도 gate를 따른다. Linux sudo mount/write 권한은 IAM과 별개다. NFS 접근은 기존 NSG/SL·Export source/access·고정 UID/GID/POSIX를 각각 확인한다.


## 비용및주의사항

같은 FS/표준 MT에 `/lab-alt` Export 하나와 작은 `alt-proof.txt`만 추가한다. 두 mount는 데이터 복제본이 아니며 두 번째 FS/MT/VM을 만들지 않는다. 기존 FSS/client 유지와 추가 파일의 저장·실제 전송 비용 및 owner43 lease를 확인한다. missing-path 시험은 승인60초 복구 창과 bounded mount를 따른다. alt mount/Export만 정리하고 원래 `/lab`과 파일은 다음 consumer/owner43 계약대로 반환한다.

## 실습구성

### 이번 workshop 시나리오와 입력 계약

같은 FS/MT에 별도 Export를 생성하고 두 path에서 공유 data를 읽고 쓴다.

|입력|값/출처|시점|실행 위치|선택 이유|
|---|---|---|---|---|
|`B_PRIVATE_IP`|owner22의 actual output; `<...>` 대체 필수|실행 전 선행 완료값|Console/사용자 전용 shell|정확한 대상/소유관계 고정|
|`MT_PRIVATE_IP`|owner43의 actual output; `<...>` 대체 필수|실행 전 선행 완료값|B guest|정확한 대상/소유관계 고정|
|`FS_ID`|owner43의 actual output; `<...>` 대체 필수|실행 전 선행 완료값|Console/사용자 전용 shell|정확한 대상/소유관계 고정|
|`MT_ID`|owner43의 actual output; `<...>` 대체 필수|실행 전 선행 완료값|Console/사용자 전용 shell|정확한 대상/소유관계 고정|
|`EXPORT_SET_ID`|owner43의 actual output; `<...>` 대체 필수|실행 전 선행 완료값|Console/사용자 전용 shell|정확한 대상/소유관계 고정|

|Console field|실제 입력/선택|이유|
|---|---|---|
|Export path / existing MT / options|/lab-alt; existing mt-lab/fs-lab; B/32 Privileged RW SYS|이번 기능/실패 원인을 한 변수로 분리|

**생성·변경 범위:** lab-alt-export:Export. **완료 비교:** two paths same hash/shared file; missing path deny; alt deletion original intact. 기대 패턴이며 OCI 실측값이 아니다. **정리:** 본 주제의 변경만 원복하고 전용 신규 resource만 dependency 역순으로 정리한다; 공유 자원은 creator와 consumer registry/expiry 기준으로 반환한다.

기존 `/lab`와 같은 File System에 추가 path `/lab-alt` Export 하나를 생성한다. Export path는 immutable이고 파일 시스템 내부 directory path와 다르다는 것을 **두 mount에서 같은 file hash**로 확인한다. 동일 MT에서 `/lab/child`처럼 prefix 충돌하는 path는 생성하지 않는다.

## 실습절차

### 작업 순서 체크포인트
same FS에 새 export를 만든 뒤 원본 mount positive를 유지하고 alt mount/hash와 missing-path negative를 각각 확인한다. 마지막에는 새 export만 제거하고 원본 FS는 반환한다. 단계 실패는 다음 생성으로 넘기지 않고 같은 target의 복구/반환으로 처리한다.

### 개별 완주 보정 — 생성·검증·소유권

missing path 검증은 아래 step6의 checked guard/mount/unmount/rmdir block만 사용한다. 첫 mount 전에 symlink/non-directory/occupied/nonempty/query-error 모두 STOP하며 예상 밖 mount 성공은 쓰기 없이 INCONCLUSIVE다.

### A. 동일 File System 새 Export
1. **File Systems → fs-lab → Exports → Create export**에서 path `/lab-alt`, **Use secure export options on**, LDAP off, Resource lock **No lock**, **Select existing mount target** `mt-lab`를 선택한다. AD/Compartment/MT OCID를 재확인한다.
2. 새 Export options를 즉시43와 같은 B/32, Privileged, RW, Squash None, SYS, Anonymous Not Allowed로 제한한다. default0.0.0.0/0를 남기지 않는다.
3. 두 Export OCID·path·FileSystem id·ExportSet id를 기록한다. path edit가 불가능하다는 공식 제한을 읽고 필요한 새 path를 **새 Export**로 생성했다는 사실을 남긴다.

### B. 두 path가 같은 데이터라는 검증
4. B에서 기존 `/mnt/oci-lab` mount를 유지하고 별도 mount point를 만든다.

```bash
set -euo pipefail
mountpoint -q /mnt/oci-lab || exit 1
test "$(findmnt -rn -M /mnt/oci-lab -o SOURCE)" = "$MT_PRIVATE_IP:/lab" || exit 1
test "$(findmnt -rn -M /mnt/oci-lab -o FSTYPE)" = nfs || exit 1
MOUNTPOINT="/mnt/oci-lab-alt"
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
ALT_DIR_CREATED=0
if ! test -d "$MOUNTPOINT"; then sudo mkdir "$MOUNTPOINT" || exit 1; ALT_DIR_CREATED=1; fi
# ALT_DIR_CREATED와 exact path를 cleanup 소유 기록에 저장한다.
# 첫 mount 전 독립 복구담당자/원래 MT·/lab 설정/60초 deadline 승인.
# timeout은 kernel request 종료 보장이 아니다; 오류면 쓰기/rmdir 금지.
if sudo timeout -k 5s 20s mount -t nfs -o vers=3,proto=tcp,nosuid,resvport,sec=sys,retry=0 "$MT_PRIVATE_IP:/lab-alt" /mnt/oci-lab-alt; then :;
else RC=$?; printf 'BLOCKED_RECOVERY: mount rc=%s; owner inspect residual request/D-state/exact mount before cleanup\n' "$RC"; exit 1; fi
test "$(findmnt -rn -M /mnt/oci-lab-alt -o SOURCE)" = "$MT_PRIVATE_IP:/lab-alt" || exit 1
test "$(findmnt -rn -M /mnt/oci-lab-alt -o FSTYPE)" = nfs || exit 1
sudo test ! -L /mnt/oci-lab-alt/alt-proof.txt && sudo test ! -e /mnt/oci-lab-alt/alt-proof.txt || exit 1
sudo cmp /mnt/oci-lab/proof.txt /mnt/oci-lab-alt/proof.txt
sudo sha256sum /mnt/oci-lab/proof.txt /mnt/oci-lab-alt/proof.txt
sudo sh -c "printf 'same-filesystem\n' > /mnt/oci-lab-alt/alt-proof.txt"
sudo cat /mnt/oci-lab/alt-proof.txt
```

5. 두 hash가 같은지, 새 file이 원래 path에도 보이는지 확인한다. 같은 ExportSet path 두 개가 파일 복제본 두 개를 만든 것이 아니다.
6. 다음 전용 guard를 B guest에서 먼저 수행한 뒤 negative는 존재하지 않는 path `/lab-missing`으로 새 mount를 시도한다.

```bash
set -euo pipefail
MOUNTPOINT="/mnt/oci-lab-missing"
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
MISSING_DIR_CREATED=0
if ! test -d "$MOUNTPOINT"; then sudo mkdir "$MOUNTPOINT" || exit 1; MISSING_DIR_CREATED=1; fi
# 원래 MT/export/ingress를 복구담당자가 확인; 최대60초 시험 창 승인 뒤 수행
test ! -e export45-missing.error && test ! -L export45-missing.error || exit 1
if sudo timeout -k 5s 20s mount -t nfs -o vers=3,proto=tcp,nosuid,resvport,sec=sys,retry=0 "$MT_PRIVATE_IP:/lab-missing" "$MOUNTPOINT" 2>export45-missing.error; then
  printf 'INCONCLUSIVE: unexpected mount; no writes\n'
  findmnt -rn -M "$MOUNTPOINT" -o SOURCE,FSTYPE || exit 1
  sudo umount "$MOUNTPOINT" || exit 1
else
  rc=$?; printf 'actual missing-path mount rc=%s; inspect RPC/error, timeout is inconclusive\n' "$rc"
fi
# timeout/강제종료는 kernel mount 종료를 보장하지 않음. exact mount가 남으면 쓰기/rmdir 금지.
if findmnt -rn -M "$MOUNTPOINT"; then printf '%s\n' 'STOP: mount remains; owner recover and clean unmount'; exit 1; else RC=$?; [ "$RC" = 1 ] || exit 1; fi
if [ "$MISSING_DIR_CREATED" = 1 ]; then sudo rmdir "$MOUNTPOINT" || exit 1; fi
```
 timeout15/20초 또는 오류 시 Console 복구담당자는60초 안에 원래 /lab·MT ingress를 확인/복원한다. B가 D-state/잔존 mount면 완료/cleanup이 아니라 BLOCKED_RECOVERY로 기록하고 정상 종료 전 새 명령을 시작하지 않는다. `export45-missing.error`의 access denied/no such file 같은 실제 diagnostic과 정상 /lab-alt positive를 함께 기록하며 transport timeout은 INCONCLUSIVE다.
 Console에서 그 path가 실제 없고 `/lab-alt` mount는 정상인 상태를 기록한다. 실패가 network timeout이면 wrong export path deny라고 단정하지 않는다. 해당 mountpoint는 별도로 사용한다.
7. `sudo umount /mnt/oci-lab-alt` 후 새 Export `/lab-alt`를 **Delete export**한다. 원래 `/lab`에서 proof가 유지되는지 확인한다. Export 삭제는 File System 삭제가 아니다. 기존 mounted client를 남겨 삭제 효과를 단정하지 않는다.

## 결과검증


### baseline-45-ready


완료 단계 **EXPORT-tested**: 두 Export가 같은 File System/MT/ExportSet을 참조, 각 exact path mount 성공과 동일 hash/shared new file, nonexistent path 실패, `/lab-alt` 삭제 후 `/lab` 데이터 유지.45 종료 시 original Export 하나만 남겨46 우회 접근 confounder를 제거한다.

**실행 기록(아직 미작성):** 실행자/Region/시각, exact resource OCID, 설정 전·후, 성공 출력, deny 오류·exit code/request-id, positive control, 정리 결과. 문서 검증은 실제 OCI 실행 성공을 뜻하지 않는다.

## 문제해결

Create export 실패: path prefix/`/`/trailing slash/`.`/`..` 제한과 same AD 확인. mount path wrong: FileSystem display name 대신 실제 Export path 사용. Export 삭제 후 file 접근이 계속되면 old mount/cache 존재를 조사하고 신규 mount로 검증한다. File System 삭제로 path를 바꾸지 않는다.

## 리소스정리

단계7의 alt unmount/Export 삭제가 완료·readback됐으면 이를 반복하지 않는다. 중단 재개만 exact alt SOURCE 및 actual Export OCID 상태를 확인해 남은 작업을 수행한다. mount 부재/잔존 요청 없음과 ALT_DIR_CREATED=1 소유 기록인 경우에만 `sudo rmdir /mnt/oci-lab-alt`한다; 기존 빈 directory는 보존한다. 이번 exact `export45-missing.error`도 관찰 종료 후 삭제한다. `/lab`와 fs/mt는46 및 선택47의 반환 완료 또는 승인 expiry까지 owner43의 consumer registry에 따라 보존한다. `alt-proof.txt`는 **owner43 최종 정리에서 실제 `/lab` NFS mount/source를 확인하고 mounted 상태에서 삭제한 뒤 unmount**한다. owner46은 공유 파일/자원 최종 삭제 owner가 아니다. 새 Export만 삭제해야 하며 원래 File System은 삭제하지 않는다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 목표는 학습 후 설명·구분·확인할 내용이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [19] [Overview of File Storage — Exports](https://docs.oracle.com/en-us/iaas/Content/File/Concepts/filestorageoverview.htm)
  - 확인 범위: Exports.

### 실습 작업을 지원하는 추가 공식 본문

- [실습근거 1] [Creating an Export](https://docs.oracle.com/en-us/iaas/Content/File/Tasks/create-export.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
- [실습근거 2] [Working with NFS Exports and Export Options](https://docs.oracle.com/en-us/iaas/Content/File/Tasks/exportoptions.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
- [실습근거 3] [Mounting File Systems From UNIX-Style Instances](https://docs.oracle.com/en-us/iaas/Content/File/Tasks/mountingunixstyleos.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
- [실습근거 4] [Details for the File Storage Service](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/filestoragepolicyreference.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.

**조건부 저술 승인:** UpdateExport의 exact 최소 permission 대응은 미입증이다. 별칭 permission을 발명하지 않는다. 조직 관리자가 exact Export OCID/scope의 변경 권한·복구 권한을 명시 승인한 경우만 Console 변경을 실행한다. 승인 전 BLOCKED_PERMISSION; 공식 permission 표 미입증은 조건부 authoring을 막지 않지만 실행/최소권한 검증 완료를 뜻하지 않는다. 반복 권한 조회 루프로 이를 자동 승인하지 않는다.
