> **문서 검토 완료 · OCI 실행 미검증**: 이 Export Options 실습의 제한된 문서 검토 범위에서 절차·복구·정리 흐름을 확인했습니다. 실제 OCI 실행 성공이나 전체 교육과정의 승인을 뜻하지 않습니다.

# Export Options

> 문서 상태: **제한 범위 문서 검토 완료 / OCI 실행 미검증**. 예상 출력은 패턴이며 실측값이 아닙니다. 아래 UpdateExport 권한 확인, 실제 선행 산출물, 비용·독립 복구 담당자·deadline 및 owner 반환 조건을 충족한 경우에만 실행합니다. 이 문서의 검토 완료는 [통합 상태](../../../Docs/README.md)의 전체 승인이나 게시 완료를 뜻하지 않습니다.

> **저술 완료 · OCI 실행 미검증** — Oracle 공식 작업 문서를 근거로 작성한 한국어 hands-on입니다. Oracle 공인 교육/실행 인증이 아니며 기대 결과와 실제 결과를 구분합니다. 비용 승인과 실제 선행 산출물 확인 후에만 실행하세요.

## 학습목표

- 기존 /lab의 B/32 entry를 RW→RO로 바꾸고 client rw/POSIX 고정 상태의 read 성공·write 거부를 기록한다.
- source no-match 신규 mount 거부 뒤 exact B/32 RW를 복원하고 same-mount write 성공을 47 또는 owner43에 반환한다.

완료 기준은 아래 실제 생성·설정·요청 결과로 판정한다. 이 목표는 수행할 작업이며 OCI에서 관찰한 결과가 아니다.

## 대응 Roadmap 항목

- 항목명: **Export Options**
- Roadmap 경로: **File Storage → Export Options**
- Node ID: `service-file-3`
- 상위 항목: **File Storage** (`service-file-0`)
- 문서 범위: 공식 개념 목표와 실제 구성·허용/불허 검증·정리 절차. 아래 실습은 저술되었으나 OCI에서 실행 검증되지 않았습니다.
- [전체 항목 목록](../../../Docs/README.md)

## 사전준비

43 실제 FS/MT/EXPORT_ID/B_PRIVATE_IP와44 network 복원,45 alt Export 삭제를 반환받는다. 모든 FS Export 목록/원래 options, exact MT_IP:/lab+nfs/rw, 고정 POSIX/root baseline write를 확인한다. UpdateExport 실행 승인은 별도 gate이며 RW 복구/write를47 또는43에 반환한다.

### 기준 계약과 실제 단계 선행조건

[공통 baseline-contract.json](../../../baseline-contract.json)의 OS/CIDR/소유권을 따른다. Oracle Linux 9 / `opc` / `dnf` 외의 숨은 Ubuntu 또는 probe VM은 만들지 않는다. OCID/IP의 null은 실행 전 반드시 실제 출력으로 채우는 **실행 차단 입력**이다. approval/capacity/quota/identity 미확인은 완료가 아니다.

- [43.ready File Storage — FUNCTIONAL_COMPLETION](00_File_Storage.md#baseline-43-ready): 표시된 FUNCTIONAL_COMPLETION은 해당 linked ready 단계의 실제 검증/반환이 필요하다. PRE_CREATE_OR_INSTALL_PREPARATION으로 표시한 링크만 준비 출력까지 요구한다. 필요한 OCID/주소만 인계한다.
- [44.ready Mount Targets — FUNCTIONAL_COMPLETION](01_Mount_Targets.md#baseline-44-ready): 표시된 FUNCTIONAL_COMPLETION은 해당 linked ready 단계의 실제 검증/반환이 필요하다. PRE_CREATE_OR_INSTALL_PREPARATION으로 표시한 링크만 준비 출력까지 요구한다. 필요한 OCID/주소만 인계한다.
- [45.ready Exports — FUNCTIONAL_COMPLETION](02_Exports.md#baseline-45-ready): 표시된 FUNCTIONAL_COMPLETION은 해당 linked ready 단계의 실제 검증/반환이 필요하다. PRE_CREATE_OR_INSTALL_PREPARATION으로 표시한 링크만 준비 출력까지 요구한다. 필요한 OCID/주소만 인계한다.

공유 A/B consumer는 INSTANCE_A_ID에 owner10 vm-a의 실제 OCID, INSTANCE_B_ID에 owner22 private-vm의 실제 OCID, `A_PUBLIC_IP`, `A_PRIVATE_IP`, `B_PRIVATE_IP`를 실제 생성 응답에서 전달받는다. 해당 주제에 필요하지 않은 VM/변수는 생성·요구하지 않는다.

- [File Storage](00_File_Storage.md#baseline-43-ready) **FSS-mounted**, [Mount Targets](01_Mount_Targets.md#baseline-44-ready) **MT-tested**, [Exports](02_Exports.md#baseline-45-ready) **EXPORT-tested** 완료. `/lab-alt`는 삭제됐고 File System으로 통하는 모든 Export 목록을 관리자 확인한다.
- B의 실제 source IP와 NFS mount options/POSIX UID/GID를 고정한다.43의 RW에서 동일 root write가 이미 성공해야 한다. 비용 승인 시간 안에서 네트워크/NSG를 변경하지 않고 Export만 바꾼다.

## 권한

### 실제 task/API별 공식 권한 본문 대조

**EXECUTION_PERMISSION_GATE — `UpdateExport` (live official-body 확인):** 현재 [공식 File Storage operation 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/filestoragepolicyreference.htm)의 전체 Export 행에는 CreateExport/GetExport/DeleteExport/Export lock 작업과 **UpdateExportSet**은 있지만 **UpdateExport 행은 없다**. UpdateExportSet의 `EXPORT_SET_UPDATE`를 UpdateExport 권한으로 치환하지 않는다. 따라서 exact operation→permission mapping은 공식 표만으로 확정할 수 없으며 alias나 추정 permission을 발명하지 않는다. 관리자/Oracle의 명시 확인 전 Console mutation 실행은 차단한다. family grant는 최소정책으로 검증되지 않았다.

**변경 전 실행 승인:** actual principal/EXPORT_ID/Compartment, UpdateExport 편집·전체 original options 복구 범위, 승인 관리자/UTC/expiry와 독립 복구담당자를 기록한다. 해당 작업 권한을 명시 승인받지 못하면 mutation 금지/NOT_EXECUTED다. UpdateExportSet alias나 추정 permission을 사용하지 않는다.

아래 공식 operation 표에서 이 실습의 읽기·변경·정리 권한을 대조한다. family grant/일부 dependency 인용은 검증된 minimum policy가 아니며 실제 principal과 resource scope를 관리자가 승인해야 한다.

- `DeleteExport`: EXPORT_SET_UPDATE + FILE_SYSTEM_NFSv3_UNEXPORT — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/filestoragepolicyreference.htm).
- `DeleteFileSystem`: FILE_SYSTEM_DELETE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/filestoragepolicyreference.htm).
- `DeleteMountTarget`: MOUNT_TARGET_DELETE + VNIC_DELETE(vnicCompartment) + SUBNET_DETACH(subnetCompartment) + VNIC_DETACH(vnicCompartment) + PRIVATE_IP_DELETE(subnetCompartment) + PRIVATE_IP_UNASSIGN(subnetCompartment) + VNIC_UNASSIGN(vnicCompartment) — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/filestoragepolicyreference.htm).
- `GetExport`: EXPORT_SET_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/filestoragepolicyreference.htm).
- `ListExports`: EXPORT_SET_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/filestoragepolicyreference.htm).

이번 작업자는 위 GetExport/ListExports와 exact Export의 UpdateExport/전체 options 복구 실행 승인을 받는다. DeleteFileSystem/DeleteMountTarget/DeleteExport는 최종 owner43 작업이며46 작업자의 필수 권한이 아니다. Linux sudo mount/write 권한은 IAM과 별개다. NFS 접근은 기존 NSG/SL·Export source/access·고정 UID/GID/POSIX를 각각 확인한다.


## 비용및주의사항

신규 FS/MT/VM은 없고 owner43 Export의 access/source만 변경하며 작은 baseline/recovery 파일을 작성한다. 기존 FSS 저장·NFS client/boot 및 실제 전송 비용은 owner43/22 lease를 따른다. main RO/source 시험은 최대120초 독립 복구 창이며 추가 first-match/root-squash branch는 별도 승인 없으면 NOT_SELECTED다. High Performance·Replication·Snapshot policy는 추가하지 않는다. RW 복구와 same-mount write 성공 뒤 자신의 파일만 삭제하고 endpoint는43/47에 반환한다.

## 실습구성

### 이번 workshop 시나리오와 입력 계약

같은 Export source/access만 RW→RO→no-match→RW로 바꾸고 실제 write deny를 확인한다.


- 입력: `B_PRIVATE_IP`
  - 값/출처: owner22의 actual output; `<...>` 대체 필수
  - 시점: 실행 전 선행 완료값
  - 실행 위치: Console/사용자 전용 shell
  - 선택 이유: 정확한 대상/소유관계 고정

- 입력: `MT_PRIVATE_IP`
  - 값/출처: owner43의 actual output; `<...>` 대체 필수
  - 시점: 실행 전 선행 완료값
  - 실행 위치: B guest
  - 선택 이유: 정확한 대상/소유관계 고정

- 입력: `EXPORT_ID`
  - 값/출처: owner43의 actual output; `<...>` 대체 필수
  - 시점: 실행 전 선행 완료값
  - 실행 위치: Console/사용자 전용 shell
  - 선택 이유: 정확한 대상/소유관계 고정

- 입력: `FS_ID`
  - 값/출처: owner43의 actual output; `<...>` 대체 필수
  - 시점: 실행 전 선행 완료값
  - 실행 위치: Console/사용자 전용 shell
  - 선택 이유: 정확한 대상/소유관계 고정



- Console field: Export Access / Source / client options
  - 실제 입력/선택: B/32; server Read Only while client rw; restore exact B/32 RW
  - 이유: 이번 기능/실패 원인을 한 변수로 분리


**생성·변경 범위:** export-options-change:ConfigurationChange. **완료 비교:** RW write→server RO read/deny-write→no-match deny-mount→RW restored. 기대 패턴이며 OCI 실측값이 아니다. **정리:** 본 주제의 변경만 원복하고 전용 신규 resource만 dependency 역순으로 정리한다; 공유 자원은 creator와 consumer registry/expiry 기준으로 반환한다.

동일 `/lab` Export의 B/32 entry를 RW → RO → source 불일치 → RW로 바꿔 실제 NFS read/write/deny를 검증한다. ordered first-match, secure ports, identity squash와 POSIX 권한의 역할을 구분한다. Root squash는 별도 optional variant이며 main RO 실험은 Squash None을 고정한다.

## 실습절차

### 개별 완주 보정 — 생성·검증·소유권

UpdateExport operation→minimum permission 행은 공식 표만으로 미확정이다. exact principal/EXPORT_ID/scope의 편집·복구 권한을 관리자가 승인하기 전 Console mutation을 시작하지 않는다. family grant를 minimum으로 주장하지 않는다. RO/source 실험은 Export editor 저장 후 실제 옵션 readback을 하고 client rw mount와 POSIX를 고정한다. 최종 파일 삭제는 actual /lab NFS source 확인 후 mounted RW 상태에서 한다.

### 변경 전 복구 담당자와 deadline

Export 편집 전에 exact EXPORT_ID/전체 original options를 저장하고 다른 관리자 Console 세션을 복구 담당자로 확보한다. 첫 RO/source 변경부터 최대120초 또는 승인 expiry 중 이른 시각에 원래 B/32 RW/SYS/Squash=None을 복원한다. unexpected write/권한 오류/hard mount 대기/세션 손실은 완료가 아니라 즉시 복원→새 mount→same-mount RW 검증이다. 담당자/원래값 저장이 없으면 negative를 시작하지 않는다.

### A. RW baseline와 설정 payload
1. **File Systems → fs-lab → Exports → /lab → NFS Export Options → Edit**에서 entry 한 개: source `<B_PRIVATE_IP>/32`, Ports **Privileged**, Access **Read/Write**, Squash **None**, Allowed authentication **SYS**, Anonymous Not Allowed. broad CIDR entry나 다른 Export 우회가 없는지 확인한다.
2. 아래 Bash는 모두 B guest에서 실행한다. 새 shell에서는 owner43이 반환한 실제 주소로 `MT_PRIVATE_IP`를 다시 설정한다(`export MT_PRIVATE_IP='<actual mount target private IPv4>'`; placeholder 그대로 실행 금지). 오류로 block이 종료되면 다음 block으로 진행하지 말고 독립 복구담당자에게 원래 options 복원을 요청한다. 전용 작업 directory에서 실행하며 이번 run이 만든 파일 목록과 전체 original options는 shell 밖에도 보존한다. B에서 `findmnt -T /mnt/oci-lab`로 client mount가 **rw**인지 확인한다. 아래 baseline write/read를 수행한다.

```bash
mountpoint -q /mnt/oci-lab || { printf '%s\n' 'STOP: no NFS mount'; exit 1; }
findmnt -rn -T /mnt/oci-lab -o SOURCE,FSTYPE,OPTIONS
# exact source/type/options must pass before the first write
test "$(findmnt -rn -M /mnt/oci-lab -o SOURCE)" = "$MT_PRIVATE_IP:/lab" || exit 1
test "$(findmnt -rn -M /mnt/oci-lab -o FSTYPE)" = nfs || exit 1
findmnt -rn -M /mnt/oci-lab -o OPTIONS | tr \, '\n' | grep -Fx rw || exit 1
sudo test ! -e /mnt/oci-lab/export-proof.txt && sudo test ! -L /mnt/oci-lab/export-proof.txt || exit 1
sudo timeout -k 5s 10s sh -c "printf 'rw-baseline\n' > /mnt/oci-lab/export-proof.txt" || exit 1
printf 'rw-baseline\n' | sudo timeout -k 5s 10s cmp - /mnt/oci-lab/export-proof.txt || exit 1
sudo timeout -k 5s 10s stat -c '%u:%g %a' /mnt/oci-lab/export-proof.txt || exit 1
sudo timeout -k 5s 20s umount /mnt/oci-lab || exit 1
```

3. Console 선택값의 API payload 의미를 기록한다. 아래 JSON은 **API model camelCase**이며 docs의 CLI examples에 섞인 hyphen/문자열 boolean 표기를 그대로 복사하지 않는다. CLI로 실행하려면 설치된 버전의 `--generate-param-json-input export-options`와 schema를 먼저 비교한다. 이 실습의 실제 수정은 Console로 한다.

[공식 ClientOptions model](https://docs.oracle.com/en-us/iaas/tools/python/latest/api/file_storage/models/oci.file_storage.models.ClientOptions.html)의 타입과 공식 task API 예시를 함께 대조했다: `source` string, `requirePrivilegedSourcePort` boolean, `access` string enum READ_WRITE/READ_ONLY, `identitySquash` string enum NONE/ROOT/ALL, `anonymousUid`/`anonymousGid` integer, `allowedAuth` array of strings (SYS/KRB5/KRB5I/KRB5P), `isAnonymousAccessAllowed` boolean. Console의 Source / Ports / Access / Squash / Squash UID / Squash GID / Allowed Authentication Options / Anonymous Access에 각각 대응한다. SDK snake_case property는 wire alias가 아니다. task의 일부 예시에 문자열 boolean·scalar allowedAuth·`identitysquash`가 섞여 있어 해당 예시를 payload schema로 채택하지 않는다. Console task의 Squash default None과 SDK의 미지정 ROOT 설명이 다르므로 모든 값을 명시하고 실제 readback한다.

```json
{
  "exportOptions": [
    {
      "source": "<B_PRIVATE_IP>/32",
      "requirePrivilegedSourcePort": true,
      "access": "READ_WRITE",
      "identitySquash": "NONE",
      "anonymousUid": 65534,
      "anonymousGid": 65534,
      "allowedAuth": ["SYS"],
      "isAnonymousAccessAllowed": false
    }
  ]
}
```

**RO 첫 변경 전 실행 gate:** 복구담당자가 독립 Console에서 원래 options를 열고120초 wall-clock deadline을 감시한다. RO 요청은 mount20초/kill유예5초, read10초, touch10초로 한 번만 수행한다. timeout/잔존 D-state는 RO deny 증거가 아니고 즉시 원래 options 복구 후 정상 요청 종료/clean unmount를 기다린다. deadline을 요청 프로세스 종료 여부에 의존시키지 않는다; 복원 확인 전 source 시험 금지.

### B. 서버 RO와 실제 write deny
4. Console에서 같은 entry의 **Access만 Read Only**로 바꾼다. B를 client **rw** options로 remount한다. client `-o ro`를 사용해 서버 RO와 혼동하지 않는다.

```bash
if findmnt -rn -M /mnt/oci-lab; then exit 1; else RC=$?; test "$RC" = 1 || exit 1; fi
test -d /mnt/oci-lab && test ! -L /mnt/oci-lab || exit 1
ENTRIES=$(sudo find /mnt/oci-lab -mindepth 1 -maxdepth 1 -print -quit) || exit 1
test -z "$ENTRIES" || exit 1
sudo timeout -k 5s 20s mount -t nfs -o vers=3,proto=tcp,nosuid,resvport,sec=sys,rw,retry=0 "$MT_PRIVATE_IP:/lab" /mnt/oci-lab || exit 1
findmnt -T /mnt/oci-lab
sudo timeout -k 5s 10s cat /mnt/oci-lab/export-proof.txt || exit 1
test "$(findmnt -rn -M /mnt/oci-lab -o SOURCE)" = "$MT_PRIVATE_IP:/lab" || exit 1
test "$(findmnt -rn -M /mnt/oci-lab -o FSTYPE)" = nfs || exit 1
findmnt -rn -M /mnt/oci-lab -o OPTIONS | tr \, '\n' | grep -Fx rw || exit 1
test ! -e export-ro-error.txt && test ! -L export-ro-error.txt || exit 1
sudo test ! -e /mnt/oci-lab/ro-should-fail.txt && sudo test ! -L /mnt/oci-lab/ro-should-fail.txt || exit 1
if sudo timeout -k 5s 10s env LC_ALL=C touch /mnt/oci-lab/ro-should-fail.txt 2> export-ro-error.txt; then
  printf '%s\n' 'UNEXPECTED WRITE: restore RW/source and investigate before proceeding'
  sudo timeout -k 5s 20s umount /mnt/oci-lab || exit 1
  exit 1
else
  RC=$?
  printf 'denied write exit status=%s\n' "$RC"
  case "$RC" in 124|137) printf 'INCONCLUSIVE timeout: restore now\n'; exit 1;; esac
fi
grep -F 'Read-only file system' export-ro-error.txt || { printf 'INCONCLUSIVE: restore Export RW; not proven server RO\n'; exit 1; }
# 복구 담당자는 중단/예외에도 Console에서 원래 B/32 RW를 즉시 복원한다.
sudo timeout -k 5s 20s umount /mnt/oci-lab || exit 1
```

read는 성공, 동일 root write는 실패해야 한다. client RO/permissions/root squash를 바꾸지 않은 상태여야 서버 Export RO 효과를 분리할 수 있다.

**4b. RO 종료 후 RW 기준 복원 (source 변경 전 필수):** Console에서 **source는 원래 B/32 그대로**, Access만 **Read/Write**로 복원하고 나머지 original options와 새로고침 readback한다. 위 B 요청이 정상 종료되고 D-state/잔존 요청이 없는지 확인한 뒤 아래를 수행한다. 오류면 [43 residual 복구](00_File_Storage.md#c-실제-linux-mount와-공유-데이터)로 반환하고 source를 편집하지 않는다.

```bash
if findmnt -rn -M /mnt/oci-lab; then exit 1; else RC=$?; test "$RC" = 1 || exit 1; fi
test -d /mnt/oci-lab && test ! -L /mnt/oci-lab || exit 1
ENTRIES=$(sudo find /mnt/oci-lab -mindepth 1 -maxdepth 1 -print -quit) || exit 1
test -z "$ENTRIES" || exit 1
sudo timeout -k 5s 20s mount -t nfs -o vers=3,proto=tcp,nosuid,resvport,sec=sys,rw,retry=0 "$MT_PRIVATE_IP:/lab" /mnt/oci-lab || exit 1
test "$(findmnt -rn -M /mnt/oci-lab -o SOURCE)" = "$MT_PRIVATE_IP:/lab" || exit 1
test "$(findmnt -rn -M /mnt/oci-lab -o FSTYPE)" = nfs || exit 1
findmnt -rn -M /mnt/oci-lab -o OPTIONS | tr , '\n' | grep -Fx rw || exit 1
sudo test ! -e /mnt/oci-lab/export46-between.txt && sudo test ! -L /mnt/oci-lab/export46-between.txt || exit 1
sudo timeout -k 5s 10s sh -c "printf 'rw-between\n' > /mnt/oci-lab/export46-between.txt" || exit 1
printf 'rw-between\n' | sudo timeout -k 5s 10s cmp - /mnt/oci-lab/export46-between.txt || exit 1
sudo timeout -k 5s 20s umount /mnt/oci-lab || exit 1
if findmnt -rn -M /mnt/oci-lab; then exit 1; else RC=$?; test "$RC" = 1 || exit 1; fi
```

### C. source no-match deny와 복구
5. 4b의 B/32 RW 복원·write positive·정상 unmount 후 **Access RW와 다른 options는 그대로 두고** Console source만 B가 아닌 전용 미사용 host `/32`(예: subnet에서 실제 비사용을 확인한 `10.77.20.250/32`)로 바꾼다. 기존0.0.0.0/0/넓은 CIDR entry는 없어야 한다. B 신규 mount는 access denied 등 실패해야 한다. 같은 MT의 rpcinfo/network는 정상인 상태를 확인해 network drop과 구별한다.
**step5 변경 직후 B request terminal (복구담당자120초 deadline 유지):** 4b에서 정상 unmount한 뒤 actual mount absence/local empty/!symlink를 먼저 확인한다. network/RPC positive와 no-match export readback이 준비되지 않으면 수행하지 않는다.

```bash
# B guest, caller source는 기존 B_PRIVATE_IP로 고정
if findmnt -rn -M /mnt/oci-lab; then exit 1; else RC=$?; [ "$RC" = 1 ] || exit 1; fi
test -d /mnt/oci-lab && test ! -L /mnt/oci-lab || exit 1
ENTRIES=$(sudo find /mnt/oci-lab -mindepth 1 -maxdepth 1 -print -quit) || exit 1
test -z "$ENTRIES" || exit 1
test ! -e export46-source-error.txt && test ! -L export46-source-error.txt || exit 1
timeout -k 5s 15s rpcinfo -p "$MT_PRIVATE_IP" || exit 1
if sudo timeout -k 5s 20s env LC_ALL=C mount -t nfs -o vers=3,proto=tcp,nosuid,resvport,sec=sys,rw,retry=0 "$MT_PRIVATE_IP:/lab" /mnt/oci-lab 2>export46-source-error.txt; then
  printf '%s\n' 'UNEXPECTED source allow: no writes; Console restore original B/32 RW now'
  sudo timeout -k 5s 20s umount /mnt/oci-lab || exit 1
  exit 1
else RC=$?; printf 'source no-match mount RC=%s\n' "$RC"; fi
case "$RC" in 124|137) printf 'INCONCLUSIVE timeout: restore now\n'; exit 1;; esac
# Console restoration is mandatory even if the next diagnostic fails.
grep -Ei 'access denied|permission denied' export46-source-error.txt || { printf '%s\n' 'INCONCLUSIVE transport/local error; restore before proceeding'; exit 1; }
if findmnt -rn -M /mnt/oci-lab; then printf '%s\n' 'STOP: mounted after failed request'; exit 1; else RC=$?; [ "$RC" = 1 ] || exit 1; fi
```

timeout은 source deny 증거가 아니다. kernel 잔존/D-state이면 복구담당자가 원래 export/통신을 복원하고 정상 unmount를 확인하기 전 다음 mount를 금지한다.

6. source를 실제 B/32, Access RW로 복구하고 remount하여 baseline read+new write를 확인한다. source 변경 후 old mount/cache를 재사용하지 않는다.
**step6 Console original source B/32 RW/SYS/Squash None readback 뒤 B guest:**

```bash
if findmnt -rn -M /mnt/oci-lab; then exit 1; else RC=$?; [ "$RC" = 1 ] || exit 1; fi
test -d /mnt/oci-lab && test ! -L /mnt/oci-lab || exit 1
ENTRIES=$(sudo find /mnt/oci-lab -mindepth 1 -maxdepth 1 -print -quit) || exit 1
test -z "$ENTRIES" || exit 1
sudo timeout -k 5s 20s mount -t nfs -o vers=3,proto=tcp,nosuid,resvport,sec=sys,rw,retry=0 "$MT_PRIVATE_IP:/lab" /mnt/oci-lab || exit 1
test "$(findmnt -rn -M /mnt/oci-lab -o SOURCE)" = "$MT_PRIVATE_IP:/lab" || exit 1
test "$(findmnt -rn -M /mnt/oci-lab -o FSTYPE)" = nfs || exit 1
findmnt -rn -M /mnt/oci-lab -o OPTIONS | tr \, '\n' | grep -Fx rw || exit 1
printf 'rw-baseline\n' | sudo cmp - /mnt/oci-lab/export-proof.txt || exit 1
sudo test ! -e /mnt/oci-lab/export46-recovered.txt && sudo test ! -L /mnt/oci-lab/export46-recovered.txt || exit 1
sudo timeout -k 5s 10s sh -c "printf 'rw-recovered\n' > /mnt/oci-lab/export46-recovered.txt" || exit 1
printf 'rw-recovered\n' | sudo cmp - /mnt/oci-lab/export46-recovered.txt || exit 1
sudo sha256sum /mnt/oci-lab/export46-recovered.txt || exit 1
```

**반환:** 실제 IP-source mounted/rw의 새 write/hash가 확인되어야46 completion이다. 실패하면 원래 options 유지/복구 담당자에게 BLOCKED_RECOVERY 반환하고47을 시작하지 않는다.

7. **first-match 선택 실험(선택)**: 동일 source를 포함하는 넓은 subnet RO entry를 앞에, B/32 RW entry를 뒤에 두면 RO가 적용된다. 두 entry 순서를 바꾸면 B/32 RW가 적용된다. 테스트마다 clean remount한다. 마지막에는 좁은 B/32 RW 한 entry로 복원한다. `0.0.0.0/0`는 쓰지 않는다.
8. **root squash 선택 variant — conditional SELECTED / NOT_SELECTED:** main RO/source exercise는 그대로 필수다. 이 variant 미승인은 `NOT_SELECTED`이며 저술 완료의 필수 blocker가 아니다. 실행 권한·owner43 승인·독립 복구담당자·expiry 안의 추가120초가 없으면 변경하지 않는다. 새 FS/MT/client/유료 기능은 추가하지 않는다.

### D. 승인된 same-mount root squash 비교 (선택)

main step6 복구 후 같은 B와 mounted `/mnt/oci-lab`을 사용한다. owner46은 정확한 directory `/mnt/oci-lab/export46-squash`만 소유한다. B에서 `id -u opc`, `id -g opc`를 기록하고 둘 다 0/65534가 아님을 확인한다. UID/GID는 숫자로 NFS에 전달되며 사용자 이름이 server identity가 아니다. Console에는 Squash **Root**, Squash UID/GID **65534**를 명시한다. Anonymous Access **Not Allowed**는 Kerberos LDAP lookup fallback 설정이며 SYS root squash를 끄는 설정이 아니다.

**첫 mutation 전:** 독립 관리자에게 EXPORT_ID/전체 original options/원래 B/32 RW Privileged SYS None/anonymous65534 값을 저장하고, Root 저장 시각부터120초 또는 expiry 중 이른 시각에 **전체 원래 options 복원 + Console 새 readback**을 수행하도록 맡긴다. B request 실패/세션 손실/timeout이어도 복구 시계는 계속된다. concurrent editor/다른 Export 우회/잔존 kernel request가 있으면 시작하지 않는다. 아래 setup은 None 상태에서 수행한다.

```bash
set -euo pipefail
: "${MT_PRIVATE_IP:?owner43 actual IP required}"
test "$(findmnt -rn -M /mnt/oci-lab -o SOURCE)" = "$MT_PRIVATE_IP:/lab"
test "$(findmnt -rn -M /mnt/oci-lab -o FSTYPE)" = nfs
findmnt -rn -M /mnt/oci-lab -o OPTIONS | tr , '\n' | grep -Fx rw
U=$(id -u opc); G=$(id -g opc)
for n in "$U" "$G"; do test "$n" -gt 0 && test "$n" -ne 65534; done
sudo test ! -e /mnt/oci-lab/export46-squash
sudo test ! -L /mnt/oci-lab/export46-squash
sudo timeout -k 2s 10s mkdir /mnt/oci-lab/export46-squash
sudo timeout -k 2s 10s chown "$U:$G" /mnt/oci-lab/export46-squash
sudo timeout -k 2s 10s chmod 0700 /mnt/oci-lab/export46-squash
test "$(sudo timeout -k 2s 10s stat -c '%u:%g %a' /mnt/oci-lab/export46-squash)" = "$U:$G 700"
```

다음 probe를 **phase마다 새 B shell**에서 실행한다. `PHASE=none`은 변경 전, `PHASE=root`는 Console에서 **Squash만 Root** 저장/readback 직후, `PHASE=restored`는 독립 담당자의 **None 전체 복원/readback** 뒤다. source/access/ports/auth/anonymous UID/GID, client mount와 directory UID/GID/mode는 고정한다. timeout은 deny가 아니며 즉시 복구한다. None의 root와 opc 성공 → Root의 root EACCES 거부와 opc 성공 → None 복원 root와 opc 성공이 모두 있어야 selected variant 완료다. 기존 파일 collision은 실패이며 재실행으로 덮어쓰지 않는다.

```bash
set -euo pipefail
PHASE='<none|root|restored>'
case "$PHASE" in none|root|restored) ;; *) exit 1;; esac
: "${MT_PRIVATE_IP:?}"
test "$(findmnt -rn -M /mnt/oci-lab -o SOURCE)" = "$MT_PRIVATE_IP:/lab"
test "$(findmnt -rn -M /mnt/oci-lab -o FSTYPE)" = nfs
findmnt -rn -M /mnt/oci-lab -o OPTIONS | tr , '\n' | grep -Fx rw
U=$(id -u opc); G=$(id -g opc)
for n in "$U" "$G"; do test "$n" -gt 0 && test "$n" -ne 65534; done
test "$(sudo timeout -k 2s 10s stat -c '%u:%g %a' /mnt/oci-lab/export46-squash)" = "$U:$G 700"
# Isolated exclusive creates, no shell redirection through an existing NFS file.
for ACTOR in root opc; do
  if sudo timeout -k 2s 10s python3 - "$PHASE" "$ACTOR" "$U" "$G" <<'PY'
import errno, os, sys
phase, actor, uid, gid = sys.argv[1:]
if actor == 'opc':
    os.setgroups([]); os.setgid(int(gid)); os.setuid(int(uid))
path = '/mnt/oci-lab/export46-squash/' + phase + '-' + actor
try:
    fd = os.open(path, os.O_WRONLY | os.O_CREAT | os.O_EXCL | os.O_NOFOLLOW, 0o600)
except OSError as e:
    if phase == 'root' and actor == 'root' and e.errno == errno.EACCES:
        print('EXPECTED_ROOT_SQUASH_EACCES'); sys.exit(0)
    raise
with os.fdopen(fd, 'wb') as f:
    f.write(b'squash-control\n'); f.flush(); os.fsync(f.fileno())
if phase == 'root' and actor == 'root':
    raise SystemExit('UNEXPECTED root write: restore None immediately')
assert open(path, 'rb').read() == b'squash-control\n'
print('WRITE_READ_SUCCESS', phase, actor)
PY
  then :; else printf '%s\n' 'STOP/INCONCLUSIVE: owner restore original options now'; exit 1; fi
done
```

Root probe 이후 즉시 독립 담당자는 전체 original options를 복원한다. same-mount restored probe 성공과 잔존 요청 없음 뒤 owner46이 exact source/type/rw를 다시 확인하고 이 run에서 생성된 `none-root`, `none-opc`, `root-opc`, `restored-root`, `restored-opc`만 삭제한다. 예상 밖 `root-root` 생성은 실패 증거로 보존하고 owner와 조사 후 그 exact owned 파일만 제거한다. `sudo rmdir /mnt/oci-lab/export46-squash`로 빈 전용 directory만 제거한다; recursive/glob 삭제 금지. D-state/복구 실패이면 `BLOCKED_RECOVERY`, owner43에게 반환하고47 진입 금지. 선택하지 않았으면 directory 생성도0건, 결과 `NOT_SELECTED`; main completion은 계속 요구된다. 위 절차는 authored Linux recipe이며 실제 OCI/NFS 실행 결과가 아니다.

## 결과검증


### baseline-46-ready


**단일 최종 checkpoint:** main RO/source/recovery 완료는 필수이며 D squash 결과는 `SELECTED + 실제 세 phase 증거` 또는 `NOT_SELECTED`로 별도 기록한다. optional 성공이 main 미완료를 대체하지 않는다.

완료 단계 **EXPORT-OPTIONS-tested**: RW write/read 성공 → server RO read성공/write실패 → source 불일치 신규 mount실패 → exact source RW 복구 write성공. client rw, 동일 UID/GID·POSIX, active network, all-export audit를 증거로 남긴다. first-match/root squash variant는 실제 실행한 경우만 결과를 기재한다.

**실행 기록(아직 미작성):** 실행자/Region/시각, exact resource OCID, 설정 전·후, 성공 출력, deny 오류·exit code/request-id, positive control, 정리 결과. 문서 검증은 실제 OCI 실행 성공을 뜻하지 않는다.

## 문제해결

RO인데 write 성공: 첫 matching entry의 access, 다른 Export path/old mount, same File System으로 가는 다른 MT, client cache를 확인. RW에서 Permission denied: Root squash UID/GID와 POSIX directory permission을 확인; Server RO test 전에 baseline부터 복구. no-match인데 mount성공: 실제 packet source IP, CIDR order, alternate Export, old mount를 점검. Privileged source port이면 `resvport` 필요하며 SYS allowedAuth를 빈 배열로 만들지 않는다.

## 리소스정리

마지막 RW/source B/32를 복원하고 remount/RW read-write positive control을 기록한다. **owner46은 공유 FSS 삭제 owner가 아니다.** [47 Snapshots](04_Snapshots.md)가 승인 session에 선택되면 mounted `/lab`, fs/mt/export OCID·source·RW 증거를 47과 owner43에 반환하며 `proof.txt`와 공유 자원을 보존한다. 46 소유 `export-proof.txt`/`ro-should-fail.txt`(실제로 생성된 경우만)/`export46-recovered.txt`/`export46-between.txt`만 확인 후 삭제한다. registry에 47의 SELECTED/NOT_SELECTED, 담당자, 종료시각, 비용 상한, 반환 완료를 명시한다. 예약 consumer가 없거나 expiry에 도달하면 idle 보존하지 않고 owner43의 [최종 정리](00_File_Storage.md#리소스정리)로 인계한다. 47 snapshot 삭제 완료 후 owner43만 전체 client unmount → 교육 Export → File System → 다른 export 없는 MT → 전용 NSG/Subnet/table 순으로 삭제한다.

**로컬 owned 오류 파일 종료:** 복원/options readback·IP-source RW positive와 잔존 요청 없음이 확인된 뒤 이번 B shell이 collision guard 후 만든 `export-ro-error.txt`/`export46-source-error.txt`만 삭제한다. 실패 중에는 복구 evidence를 보존한다. 사용자/기존 파일/symlink를 glob으로 제거하지 않는다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 목표는 학습 후 설명·구분·확인할 내용이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [19] [Overview of File Storage — Export Options](https://docs.oracle.com/en-us/iaas/Content/File/Concepts/filestorageoverview.htm)
  - 확인 범위: Export Options.

### 실습 작업을 지원하는 추가 공식 본문

- [실습근거 1] [Working with NFS Exports and Export Options](https://docs.oracle.com/en-us/iaas/Content/File/Tasks/exportoptions.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
- [실습근거 2] [Mounting File Systems From UNIX-Style Instances](https://docs.oracle.com/en-us/iaas/Content/File/Tasks/mountingunixstyleos.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
- [실습근거 3] [Configuring VCN Security Rules for File Storage](https://docs.oracle.com/en-us/iaas/Content/File/Tasks/securitylistsfilestorage.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
- [실습근거 4] [Details for the File Storage Service](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/filestoragepolicyreference.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
