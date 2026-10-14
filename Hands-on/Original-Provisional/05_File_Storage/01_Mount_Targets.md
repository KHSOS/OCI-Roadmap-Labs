> **기존 실습 초안 — 미검증**: 원래 93개 문서의 절차를 보존한 자료입니다. 새 튜토리얼의 품질 승인이나 OCI 실행 검증을 뜻하지 않습니다.

# Mount Targets

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../../../Docs/README.md)를 확인하세요.

> **저술 완료 · OCI 실행 미검증** — Oracle 공식 작업 문서를 근거로 작성한 한국어 hands-on입니다. Oracle 공인 교육/실행 인증이 아니며 기대 결과와 실제 결과를 구분합니다. 비용 승인과 실제 선행 산출물 확인 후에만 실행하세요.

## 학습목표

- 기존 mt-lab의 실제 IP/FQDN/AD/Subnet/Export Set을 기록하고 같은 proof를 읽는다.
- 전용 NFS ingress만 제거한 신규 RPC/mount 실패와 즉시 규칙 복원 후 정상 remount/hash를 기록한다.

완료 기준은 아래 실제 생성·설정·요청 결과로 판정한다. 이 목표는 수행할 작업이며 OCI에서 관찰한 결과가 아니다.

## 대응 Roadmap 항목

- 항목명: **Mount Targets**
- Roadmap 경로: **File Storage → Mount Targets**
- Node ID: `service-file-1`
- 상위 항목: **File Storage** (`service-file-0`)
- 문서 범위: 공식 개념 목표와 실제 구성·허용/불허 검증·정리 절차. 아래 실습은 저술되었으나 OCI에서 실행 검증되지 않았습니다.
- [전체 항목 목록](../../../Docs/README.md)

## 사전준비

43 FSS-mounted의 실제 FS/MT/Export Set OCID·MT_PRIVATE_IP/FQDN/AD/NSG rule ID·B_PRIVATE_IP와 literal MT_IP:/lab+nfs proof hash를 받는다. 기존 mount/client만 쓰고 신규 VM/FS/MT는 없다. network outage 전에43와 선택 consumer의 interruption/복구 승인을 받는다.

### 기준 계약과 실제 단계 선행조건

[공통 baseline-contract.json](../../../baseline-contract.json)의 OS/CIDR/소유권을 따른다. Oracle Linux 9 / `opc` / `dnf` 외의 숨은 Ubuntu 또는 probe VM은 만들지 않는다. OCID/IP의 null은 실행 전 반드시 실제 출력으로 채우는 **실행 차단 입력**이다. approval/capacity/quota/identity 미확인은 완료가 아니다.

- [43.ready File Storage — FUNCTIONAL_COMPLETION](00_File_Storage.md#baseline-43-ready): 표시된 FUNCTIONAL_COMPLETION은 해당 linked ready 단계의 실제 검증/반환이 필요하다. PRE_CREATE_OR_INSTALL_PREPARATION으로 표시한 링크만 준비 출력까지 요구한다. 필요한 OCID/주소만 인계한다.

공유 A/B consumer는 INSTANCE_A_ID에 owner10 vm-a의 실제 OCID, INSTANCE_B_ID에 owner22 private-vm의 실제 OCID, `A_PUBLIC_IP`, `A_PRIVATE_IP`, `B_PRIVATE_IP`를 실제 생성 응답에서 전달받는다. 해당 주제에 필요하지 않은 VM/변수는 생성·요구하지 않는다.

- [File Storage](../../../Docs/05_File_Storage/00_File_Storage.md) **FSS-mounted** 완료: 실제 write/remount read 성공, File System/MT/Export OCID·AD·MT IP를 보존한다.
- File System/MT를 추가 생성하지 않는다.43의 `nsg-fss-mt`/client egress와 mounts가 전용 자원인지 확인하고45/46까지 유지한다.

## 권한

### 실제 task/API별 공식 권한 본문 대조

아래 공식 operation 표에서 이 실습의 읽기·변경·정리 권한을 대조한다. family grant/일부 dependency 인용은 검증된 minimum policy가 아니며 실제 principal과 resource scope를 관리자가 승인해야 한다.

- `AddNetworkSecurityGroupSecurityRules`: NETWORK_SECURITY_GROUP _UPDATE_SECURITY_RULES and NETWORK_SECURITY_GROUP_INSPECT if writing a rule that specifies a network security group as the source (for ingress rules) or destination (for egress rules) — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `GetExport`: EXPORT_SET_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/filestoragepolicyreference.htm).
- `GetMountTarget`: MOUNT_TARGET_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/filestoragepolicyreference.htm).
- `RemoveNetworkSecurityGroupSecurityRules`: NETWORK_SECURITY_GROUP _UPDATE_SECURITY_RULES — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).

이번 작업자는 위 GetMountTarget/GetExport 읽기와 전용 NFS ingress의 Remove/Add rule을 원복할 network owner 권한만 승인받는다. FS/MT 생성·삭제 권한을 이 소비자에게 요구하지 않는다. 기존 owner43 family grant가 남아 있다면 넓은 편의 grant이지 minimum이 아니다. Linux sudo mount/write 권한은 IAM과 별개다. NFS 접근은 기존 NSG/SL·Export source/access·고정 UID/GID/POSIX를 각각 확인한다.


## 비용및주의사항

새 File System/Mount Target/VM은 만들지 않고 owner43의 표준 mt-lab/FS/Export와 owner22 NFS client를 재사용한다. 기존 File Storage 저장량과 client VM/boot volume, 실제 AD 간 전송 여부 및 승인 endpoint 보존기간을 확인한다. 새 데이터는 없으며 RPC/mount/capture만 제한 실행한다. NFS ingress 중단은 모든 선택 consumer가 승인한 최대60초 복구 창 안에서 수행한다. High Performance MT 전환·두 번째 MT·Snapshot은 이 문서 범위 밖이다. endpoint 삭제는 owner43만 수행한다.

## 실습구성

### 이번 workshop 시나리오와 입력 계약

MT의 endpoint 배치를 readback하고 신규 RPC/mount로 network deny와 복구를 비교한다.

|입력|값/출처|시점|실행 위치|선택 이유|
|---|---|---|---|---|
|`B_PRIVATE_IP`|owner22의 actual output; `<...>` 대체 필수|실행 전 선행 완료값|Console/사용자 전용 shell|정확한 대상/소유관계 고정|
|`MT_PRIVATE_IP`|owner43의 actual output; `<...>` 대체 필수|실행 전 선행 완료값|B guest|정확한 대상/소유관계 고정|
|`MT_FQDN`|owner43 Console의 actual FQDN; DNS/mount 선택 branch에서만 요구|선택 branch 전|B guest|main은 literal MT_PRIVATE_IP:/lab mount로 반환|
|`MT_ID`|owner43의 actual output; `<...>` 대체 필수|실행 전 선행 완료값|Console/사용자 전용 shell|정확한 대상/소유관계 고정|
|`EXPORT_SET_ID`|owner43의 actual output; `<...>` 대체 필수|실행 전 선행 완료값|Console/사용자 전용 shell|정확한 대상/소유관계 고정|

|Console field|실제 입력/선택|이유|
|---|---|---|
|MT NSG / IP / FQDN / Export Set|existing mt-lab; exact Console FQDN; restricted TCP/UDP ingress restored|이번 기능/실패 원인을 한 변수로 분리|

**생성·변경 범위:** 새 infrastructure 없음; existing resource의 configuration만 변경. **완료 비교:** fresh RPC/mount deny→network restoration same proof hash. 기대 패턴이며 OCI 실측값이 아니다. **정리:** 본 주제의 변경만 원복하고 전용 신규 resource만 dependency 역순으로 정리한다; 공유 자원은 creator와 consumer registry/expiry 기준으로 반환한다.

기존 `mt-lab`의 AD·Subnet·IP/FQDN·Export Set·NSG를 추적한다. IP와 hostname은 access endpoint이며 File System 자체의 위치/경로와 다르다. MT NSG attachment는 유지하고 전용 NFS ingress rule만 제거해 신규 RPC/mount 차단을 확인한 뒤 같은 rule을 원복한다. 고성능 MT upgrade는 비용/승인 범위 밖이다.

## 실습절차

### 개별 완주 보정 — 생성·검증·소유권

rpcinfo/mount deny 시험은 Console 복구자가 바로 exact TCP/UDP rules를 복원할 수 있을 때만 수행한다. hard NFS mount는 단순 timeout으로 확실히 종료된다고 보장하지 않는다. rule 복원으로 요청을 정상 완료시킨 뒤 실제 mount 상태를 확인하고 clean unmount한다.

### A. endpoint와 소유관계 조회
1. **Storage → File Storage → Mount Targets → mt-lab** details에서 AD, Compartment OCID, Subnet OCID, private IP, hostname/FQDN, NSG, Export Set OCID를 기록한다. File System AD와 같아야 한다.
2. B에서 `getent ahostsv4 <MT_FQDN>`, `ip route get <MT_PRIVATE_IP>`, `findmnt -rn -M /mnt/oci-lab -o SOURCE,FSTYPE`를 확인한다. DNS 이름은 실제 Console FQDN을 사용하며 `mtlab`만으로 모든 VCN에서 resolve된다고 가정하지 않는다.
```bash
# B guest: owner43 actual endpoint contract; no enclosing-filesystem inference.
export MT_PRIVATE_IP='<owner43_actual_MT_private_IP>'
assert_mt44_mount() {
  test -d /mnt/oci-lab && test ! -L /mnt/oci-lab || return 1
  test "$(findmnt -rn -M /mnt/oci-lab -o SOURCE)" = "$1:/lab" || return 1
  test "$(findmnt -rn -M /mnt/oci-lab -o FSTYPE)" = nfs || return 1
}
assert_mt44_mount "$MT_PRIVATE_IP" || exit 1
```
3. 위 exact IP:/lab+nfs assertion 성공 후에만 `sudo cat /mnt/oci-lab/proof.txt`와 hash를 기록한다. **모든 unmount 직전** `assert_mt44_mount "$MT_PRIVATE_IP" || exit 1`를 다시 실행한 뒤 clean unmount한다; wrong/enclosing mount는 읽거나 unmount하지 않는다.

### B. MT network deny/restore
**step4 mutation 전 deadline:** owner43와 모든44~47 선택 consumer에게 최대60초 interrupt 승인/UTC 종료를 받고 복구 Console 담당자가 원래 MT NSG의 exact TCP/UDP source B/32 규칙/ID를 저장한다. capture B terminal에서 `sudo timeout 25s tcpdump -ni any 'host <MT_PRIVATE_IP> and (port 111 or port 2048 or port 2049 or port 2050)'`를 먼저 시작한다. 요청은 별도 B terminal에서 아래 finite rpcinfo를 한 번만 수행한다. deadline에 결과와 무관하게 ingress 복원; hard NFS request가 kill되지 않아도 Console 복구가 우선이다.

```bash
# B request terminal, ingress 제거 전 생성; 충돌/기존 symlink면 mutation 금지
mkdir -m 700 mt44-work || exit 1
# 실제 ingress 제거 직후 이 directory의 새 output만 사용
if timeout -k 5s 15s rpcinfo -p "$MT_PRIVATE_IP" > mt44-work/deny-rpc.out 2>mt44-work/deny-rpc.err; then
  printf '%s\n' 'INCONCLUSIVE: RPC still succeeds, restore rules and inspect overlapping allows'
else RC=$?; printf 'RPC RC=%s; classify network transport not credentials\n' "$RC"; fi
```

새 mount 시험은 기존 `/mnt/oci-lab` clean unmount/source 검증 후 `sudo timeout -k 5s 20s mount -t nfs -o vers=3,proto=tcp,nosuid,resvport,sec=sys,retry=0 "$MT_PRIVATE_IP:/lab" /mnt/oci-lab`를 if/else로 호출해 RC를 기록한다. 실패/timeout은 쓰기 허가가 아니며 담당자가60초 안에 복원한다. 종료 뒤 findmnt의 exact mount 존재 여부를 확인하고 mounted면 정상 unmount 후6의 IP mount를 수행한다. D-state/잔존 요청이 있으면 새 mount/디렉터리 삭제를 하지 않고 복구담당자와 정상 종료까지 기다린다.

4. **ingress 제거 이전 B guest:** `assert_mt44_mount "$MT_PRIVATE_IP" || exit 1` → `sudo timeout -k 5s 20s umount /mnt/oci-lab || exit 1` → exact `findmnt -rn -M /mnt/oci-lab`의 exit1/잔존 요청 없음과 실제 directory empty를 확인한다. 실패면 ingress를 제거하지 않고43 residual station에 반환한다. 그 다음 MT의 전용 NSG에서 **NFS TCP/UDP ingress**만 임시 제거한다. 다른 SL/NSG가 NFS를 허용하지 않는지 확인한다. File System/Export option과 B POSIX 권한은 바꾸지 않는다.
5. B에서 **step4 직전 deadline station의15초 rpcinfo/20초 mount guarded 요청만** 수행한다. bare rpcinfo/mount를 추가 실행하지 않는다. mount가 대기하면 복구담당자가60초 이내 MT rule을 복원한다. 실패/대기 RC·request 시각·packet을 기록하고 잔존 요청/정확한 mount 상태를 확인한다. 억지 lazy unmount로 data correctness를 주장하지 않는다.
6. 원래 TCP111/2048-2050·UDP111/2048 source B/32 stateful ingress를 복원한다. B에서 아래 **remount-only**를 수행한다(43의 신규 proof 생성 fence는 다시 실행하지 않는다).
```bash
if findmnt -rn -M /mnt/oci-lab; then printf 'STOP: residual mount\n'; exit 1; else RC=$?; [ "$RC" = 1 ] || exit 1; fi
test -d /mnt/oci-lab && test ! -L /mnt/oci-lab || exit 1
sudo timeout -k 5s 20s mount -t nfs -o vers=3,proto=tcp,nosuid,resvport,sec=sys,retry=0 "$MT_PRIVATE_IP:/lab" /mnt/oci-lab || exit 1
test "$(findmnt -rn -M /mnt/oci-lab -o SOURCE)" = "$MT_PRIVATE_IP:/lab" || exit 1
test "$(findmnt -rn -M /mnt/oci-lab -o FSTYPE)" = nfs || exit 1
printf 'oci-fss-proof\n' | sudo cmp - /mnt/oci-lab/proof.txt || exit 1
sudo sha256sum /mnt/oci-lab/proof.txt || exit 1
```
7. FQDN 기반 mount는 선택 branch다. `assert_mt44_mount "$MT_PRIVATE_IP" || exit 1` 후 기존 mount clean unmount 뒤 `<MT_FQDN>:/lab`으로 같은 옵션을 쓰고 proof hash를 비교한다. 끝에는 아래 **literal IP:/lab 재마운트 반환**을 반드시 수행해45/46의 입력과 맞춘다. DNS 실패면 IP mount를 positive로 복구하고 DNS branch를 BLOCKED로 남긴다.

```bash
# B guest; fresh terminal은 A의 함수를 다시 정의한다. actual FQDN을 설정한다.
export MT_FQDN='<owner43_actual_MT_FQDN>'
assert_mt44_mount "$MT_FQDN" || exit 1
sudo umount /mnt/oci-lab || exit 1
if findmnt -rn -M /mnt/oci-lab; then exit 1; else RC=$?; [ "$RC" = 1 ] || exit 1; fi
sudo timeout -k 5s 20s mount -t nfs -o vers=3,proto=tcp,nosuid,resvport,sec=sys,retry=0 "$MT_PRIVATE_IP:/lab" /mnt/oci-lab || exit 1
test "$(findmnt -rn -M /mnt/oci-lab -o SOURCE)" = "$MT_PRIVATE_IP:/lab" || exit 1
test "$(findmnt -rn -M /mnt/oci-lab -o FSTYPE)" = nfs || exit 1
printf 'oci-fss-proof\n' | sudo cmp - /mnt/oci-lab/proof.txt || exit 1
sudo sha256sum /mnt/oci-lab/proof.txt || exit 1
```

미선택이면6의 IP-source mount를 그대로 반환하며 추가 unmount는 하지 않는다.

## 결과검증


### baseline-44-ready


완료 단계 **MT-tested**: endpoint AD/Subnet/IP/FQDN/ExportSet/NSG 대응, ingress 제거의 신규 RPC/mount 실패·원복 성공, proof hash 보존. guest `ip route`는 OCI route table/gateway를 직접 보여주는 것이 아니며 network association도 함께 기록한다. mount target를 File System 데이터 저장 VM처럼 SSH 대상으로 설명하지 않는다.

**실행 기록(아직 미작성):** 실행자/Region/시각, exact resource OCID, 설정 전·후, 성공 출력, deny 오류·exit code/request-id, positive control, 정리 결과. 문서 검증은 실제 OCI 실행 성공을 뜻하지 않는다.

## 문제해결

RPC111만 성공하고 mount 실패하면 TCP2049/2048/2050 또는 UDP2048, privileged source port, Export source를 확인. DNS 실패지만 IP mount 성공이면 VCN/Subnet DNS labels/hostname/resolver 문제. NSG 제거 후 성공이면 broad SL 허용 또는 기존 session/cache가 confounder다. 신규 mount로 검증한다.

## 리소스정리

MT rule을 반드시 복원하고 mounted `/lab`과 proof hash를 owner43에 반환한다. 45/46 및 선택47까지 endpoint를 owner43의 승인 consumer registry/EXPIRY_UTC에 따라 유지한다. owner44는 공유 File System/Export/MT의 삭제 owner가 아니다. 예약 consumer가 없거나 expiry에 도달한 최종 종료는 [owner43의 정리](00_File_Storage.md#리소스정리)로 인계한다: 실제 NFS mount/source 확인 → mounted 상태에서 전용 test 파일 삭제 → 모든 client 정상 unmount → associated Export 소유관계 확인/삭제 → File System 삭제 → 다른 File System의 Export가 없는 MT 삭제. shared MT면 다른 File System을 건드리지 않는다. owner43은 `mt-lab` 삭제 뒤 private IP/NSG membership 잔존을 확인한다.

**network negative 분류/로컬 정리:**rpc timeout/connection refused/transport unreachable과 packet 관찰은 same MT의 전후 rpc/mount positive와 한 세트로 기록한다. command missing/permission/local 오류는 network deny가 아니며 INCONCLUSIVE로 복원한다. MT ingress 제거 전 mt44-work를 예약하고 재시도는 새 승인 directory만 사용한다. 복원/readback/잔존 요청 종료 뒤 본인이 만든 deny-rpc.out/deny-rpc.err 및 own capture/rule snapshot만 exact 경로로 삭제하고 빈 mt44-work를 rmdir한다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 목표는 학습 후 설명·구분·확인할 내용이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [19] [Overview of File Storage — Mount Targets](https://docs.oracle.com/en-us/iaas/Content/File/Concepts/filestorageoverview.htm)
  - 확인 범위: Mount Targets.

### 실습 작업을 지원하는 추가 공식 본문

- [실습근거 1] [Managing Mount Targets](https://docs.oracle.com/en-us/iaas/Content/File/Tasks/managingmounttargets.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
- [실습근거 2] [Configuring VCN Security Rules for File Storage](https://docs.oracle.com/en-us/iaas/Content/File/Tasks/securitylistsfilestorage.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
- [실습근거 3] [Mounting File Systems From UNIX-Style Instances](https://docs.oracle.com/en-us/iaas/Content/File/Tasks/mountingunixstyleos.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
