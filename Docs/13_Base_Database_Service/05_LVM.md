# LVM

> 문서 작성 검토와 실행 검증은 별개입니다. OCI 실행은 **USER_EXECUTION_PENDING**이며 사용자 캡처는 별도 자료입니다. 미실행 자체는 문서 작성 승인 차단 사유가 아닙니다. 실제 결과 필드는 실행자가 채웁니다.

> 실행용 한국어 교육 문서. OCI 리소스 생성·명령 실행 결과는 아직 검증하지 않았으며 모든 기대 패턴은 미실행이다.

## 학습목표

- 87 생성 전에 single-node Logical Volume Manager와 최소 승인 allocation을 선택하고 같은 system의 actual option/node count를 읽는다.
- LABPDB datafile/MB/SQL marker를 조회하고 승인됐을 때만 DB node lsblk/findmnt read-only 결과를 실제 file path와 비교한다.
- 없는 file-prefix query와 정상 datafile 재조회로 read-only negative/복귀를 확인한 뒤87에 반환한다. ASM 선택이면 미선택이며 pv/vg/lv 생성·resize·format은 하지 않는다.

실제 OCI 요청·응답/SQL·정리 증거는 실행자가 기록한다. 아래 기대값은 실측값이 아니다.

## 대응 Roadmap 항목

- 항목명: **LVM**
- Roadmap 경로: **Base Database Service → LVM**
- Node ID: `sequence-10-5`
- 상위 항목: **Base Database Service** (`sequence-10-0`)
- 문서 범위: 이 항목의 개념, 실제 설정 절차와 학습자 실행 검증.
- [전체 항목 목록](../README.md)

## 사전준비

**시작 조건:** 87 Submit 전single-node LVM 선택;같은system 생성 후PDB SQL 완료

- [Base Database Service](00_Base_Database_Service.md#db-sql-verification) — Create 4단계에서 LVM을 선택한 단일 DB의 AVAILABLE/client SQL verification 완료. ASM 선택 트랙이면 이 LVM branch는 미선택으로 기록.

- [87 생성 전 선택](00_Base_Database_Service.md#db-create-review) — preparation-only: **87 Create 제출 전** single-node/Logical Volume Manager를 결정한다. 아래 생성 후 SQL 링크는 별도 resource dependency이며 생성 전 완료 조건이 아니다.

### 87 network-ready 소유권 인수
[87 db-network-ready](00_Base_Database_Service.md#db-network-ready)의 actual retained external NSG OCID/owner/expiry/rule IDs/DB VNIC membership readback을 인수한다.25 임시 NSG는 prerequisite 출력이 아니다. 계약 미완료이면 DB 생성 및 본 branch는 BLOCKED_DB_NSG_NOT_READY다. 새 NSG/DB를 만들지 않으며 종료는87 creator→network owner의 exact added-rule 반환 순서다.

### 준비 완료 출력과 인수 확인

| 인수 항목 | 실제 출력·출처 | 다음 작업의 사용 위치 |
|---|---|---|
| 선행 완료 상태 | 87 Submit 전single-node LVM 선택;같은system 생성 후PDB SQL 완료 | 첫 변경 전 재조회 |
| 비밀 없는 실행 입력 | same actualDB/LVMoption/nodecount/PDBservice,승인catalog·선택SSH권한,NSG계약/expiry | 아래 numbered Console/명령 단계 |
| 반환/후속 인계 | SQL/선택OS layout 읽기결과·변경없음·session 종료를87에 반환 | 종료 UTC 이전 exact owner 반환 |

실제 Compartment/Region은 승인 Console에서 읽고 동일 identity/Region을 고정한다. 87의기존VCN SQL client에서 `sqlplus -v` 및실제PDB서비스접속상태를확인한다.Console조회와SQL session의실행위치를구분한다. placeholder·명칭만 인수한 상태는 준비 완료가 아니다. 비밀번호·키·확인 token은 출력표에 적지 않는다.

## 권한

87 creator의Launch/Terminate 권한과이consumer의읽기권한을분리한다.본topic의scope는같은실제DB Compartment다.

GetDbSystem=DB_SYSTEM_INSPECT와 승인 DB catalog SELECT를 사용한다. LVM device 조회는 OS read permission이며 use volumes/manage volume-family를 추가하면 해결된다는 전제를 쓰지 않는다. pv/vg/lv 생성·format 권한은 필요하지 않다.

operation 근거: [공식 권한/작업 본문](https://docs.oracle.com/en/cloud/paas/base-database/iam-policy-details/index.html). family grant는 minimum-permission 정책으로 주장하지 않는다. 실행 전에 관리자 승인된 identity/domain·실제 Compartment scope를 확인한다.

## 비용및주의사항

87의 single-node LVM 선택에 붙는 read-only checkpoint이며 별도 boot/block volume/VM/LV를 만들지 않는다. 실제 data/recovery/software total과 초기 allocation의 향후 scale 한계를 current 계약·공식 table로 확인하되 무제한 확장/축소를 보장하지 않는다. pv/vg/lv 생성·resize·format 및 OS 패키지 upgrade는 실행하지 않는다. OS metadata 미조회는 LIMITED_LAYOUT_EVIDENCE로 기록하고 권한/비용을 늘려 대체하지 않는다. ASM 선택이면 NOT_SELECTED, 선택LVM 반환 뒤87이 soleDB를 종료한다.

## 실습구성

### 워크숍 순서와 입력값

**진행 순서:** 87 생성전 single-node LVM선택 →동일soleDB생성 →actualoption/PDBfile/marker →optionalread-onlyOS →87 반환

**선택 이유:** managed LVM에 pv/vg/lv 생성이나format 명령을 실행하지 않는다.

| 입력 | 값 또는 정확한 출처 | 실행 위치 | 생성 전/후 | 이유 |
|---|---|---|---|---|
| option/node | 87생성전LogicalVolumeManager /1 | Console | 전 | RAC제외 |
| PDB/service | 87실제connection | SQLPlus | 후 | SQLmarker확인 |


값이 다른 터미널에 자동 전파되지 않습니다. Console readback으로 비밀 없는 값만 인계하며, placeholder는 실제값으로 교체한 뒤 실행합니다. 새 리소스 생성 전 이름 충돌·소유자·최대 비용·종료 UTC를 확인하고 선택하지 않은 분기는 **NOT_SELECTED**로 기록합니다.


Base Database fast provisioning LVM 옵션 검증. 일반 Compute boot disk에 LVM을 구축하거나 existing DB storage manager를 전환하지 않는다.

### 생성 전 현행 지원 확인
[87의 public retirement/shape 지원 근거](00_Base_Database_Service.md#db-current-shape-support)를 먼저 읽는다. VM.BaseDB.x86 single-node/LVM 기본의 actual edition/version/storage 조합을 재조회하고 다른 shape로 자동 대체하지 않는다. [Oracle About DB Systems](https://docs.oracle.com/en/cloud/paas/base-database/about-dbs/index.html)의2026-07-31 retirement/KB921246를 선택 시점에 확인하며 새 DB를 추가하지 않는다.


### 이번 실습의 생성·재사용·앱 경계

| 구분 | 이번 주제의 실제 범위 |
|---|---|
| 새로 생성 | 독립리소스없음 |
| 재사용 | 87에서선택한soleLVM DB/SQLclient |
| 앱/작업부하 및 변경 | read-onlySQL·선택OSlayout;일반Compute LVM구축 없음 |
| 실행/반환 순서 | 87 Submit 전single-node LVM 선택;같은system 생성 후PDB SQL 완료 → SQL/선택OS layout 읽기결과·변경없음·session 종료를87에 반환 |

새로 생성하지 않는 항목을선행완료로가장하지않는다.위표의실제OCID/주소와진행순서를첫mutation전에확인한다.

## 실습절차

### LVM branch 생성과 read-only 완주
87에서 single-node LVM 선택·실제 생성 완료한 system만 이 실행 branch다. ASM system이면 비교만 하고 NOT_SELECTED로 기록한다. LVM 옵션값, 실제 SQL datafile path/MB/marker와 read-only OS 조회 허용 여부를 나눠 기록한다. `findmnt`가 현재 shell namespace에서 보이지 않는 것만으로 Oracle managed layout 실패라고 판정하지 않는다.
`lsblk`의 MOUNTPOINTS 미지원은 `lsblk -o NAME,TYPE,SIZE,MOUNTPOINT`로 읽기만 재시도한다. SQL 권한 오류는87 승인 PDB 계정으로 복구 후 동일 query를 다시 실행한다. internal managed storage를 Compute volume 예제로 대체하거나 pv/vg/lv creation을 해서는 안 된다. cleanup은90 retention 잔존 확인→87 terminate이며 shared disks를 umount/format하지 않는다.

[87 생성 전 선택 review](00_Base_Database_Service.md#db-create-review)는 preparation-only이고 [87 SQL 완료](00_Base_Database_Service.md#db-sql-verification)는 생성 후 resource dependency다. 최종 삭제는 [87 owner cleanup](00_Base_Database_Service.md#db-cleanup)을 따른다.

### 실행 단계

1. 87 Create review에서 **Storage management software=Logical Volume Manager**, **single-node**, **data/recovery/total storage**를 다시 확인한다. LVM fast provisioning의 초기 data allocation이 향후 maximum scale-up 한도에 영향을 준다는 official table를 읽는다. minimum 생성으로 무제한 scale-up이 가능한 것은 아니다.[T1]
2. 같은 DB system detail에서 LVM 선택값과 node count를 기록한다. 실제 PDB client connection에서 다음을 실행한다.
```sql
select file_name, tablespace_name, round(bytes/1024/1024) mb
from dba_data_files order by tablespace_name,file_name;
select 'roadmap-lvm-db-readable' as marker from dual;
```
3. 승인된 SSH management path가 이미 있으면 DB node에서 **읽기 전용** 명령만 실행한다. 서비스 OS 계정 권한으로 조회가 허용되는 것만 읽는다. 명령이 없거나 root 권한이 요구되면 OS 권한 확대 없이 Console 및 SQL evidence로 마친다.
```bash
lsblk -o NAME,TYPE,SIZE,MOUNTPOINTS
findmnt -t xfs,ext4
```
### 한 actual datafile → mount → LV/VG/PV 읽기 과제
승인 DB node와 SQL 결과에서 실제 **존재하는 일반 filesystem datafile 한 개**를 고른다. ASM `+` path에는 적용하지 않는다. OS 읽기 권한이 있는 계정에서 다음을 실행한다. package 설치·sudo 확대·mount/LVM mutation은 없다.
```bash
set -euo pipefail
export DATAFILE='<same node actual dba_data_files absolute filename>'
test -f "$DATAFILE" || { printf 'LIMITED_LAYOUT_EVIDENCE: inaccessible file\n'; exit 1; }
findmnt -T "$DATAFILE" -o TARGET,SOURCE,FSTYPE
lsblk -o NAME,TYPE,PKNAME,SIZE,MOUNTPOINT
# Available and separately DBA-approved read-only metadata only:
command -v lvs && lvs -o lv_path,lv_name,vg_name,devices
command -v vgs && vgs -o vg_name,pv_count,lv_count,vg_size,vg_free
command -v pvs && pvs -o pv_name,vg_name,pv_size,pv_free
```
작업표에 SQL datafile→findmnt TARGET(포함 mount)/SOURCE/FSTYPE→lsblk의 동일 source/TYPE=lvm→lvs lv_path와 vg_name→lvs devices의 PV→pvs 같은 vg_name 연결을 실제값으로 기록한다. `findmnt -T`가 enclosing root FS를 반환할 수 있으므로 결과와 실제 datafile 존재/같은 node namespace를 확인하며 TYPE=lvm 없는 결과를 LVM 증거로 만들지 않는다. metadata 조회 불가/도구 없음이면 **LIMITED_LAYOUT_EVIDENCE**로 정확히 남긴다. Console LVM 선택+SQL 접근 완료는 기록 가능하지만 LV/VG/PV 해석 완료나 LVM 관리 실습으로 과장하지 않는다. 아래 없는 file-prefix negative는 SQL filter control일 뿐 LVM 동작 시험이 아니다.


4. device mapper/LVM과 filesystems가 실제로 보이는 경우 이름·mount point를 기록하되 Oracle가 관리하는 layout를 임의 변경하지 않는다. `pvcreate/vgcreate/lvcreate/lvextend/mkfs`나 fstab editing은 이 실습 범위에 없다.
5. **제약 확인:** RAC에는 LVM fast provisioning을 쓰지 않는다. LVM 생성에 custom database software image를 사용할 수 없다는 official 조건을 확인한다. current supported shape/version과 Console 선택값을 우선한다. ASM 선택 system을 LVM으로 바꾸기 위해 재생성하지 않는다.
6. 실제 SQL marker를 기록하고 87 common cleanup을 당일 수행한다. storage resize/OS package upgrade는 수행하지 않는다.


### 진행·종료 판정
설정 readback·기능시험·원복·정리를 각각 실제 응답으로 기록한다. 기대값은 미실행이며 실패/만료 시 새 mutation을 중단하고 아래 topic별 종료 단계로 반환한다.

### 읽기 전용 negative control와 정상 query 복귀
동일87 LABPDB session에서 읽기 filter를 바꿨다가 복귀한다. 중간 문자열이 actual file prefix가 아님을 첫 목록에서 확인한다. SQL 권한과 container를 변경하지 않는다.
```sql
show con_name
select file_name from dba_data_files order by file_name;
select file_name from dba_data_files where file_name like '/ROADMAP_NOT_CREATED_92/%';
select file_name from dba_data_files order by file_name;
select 'roadmap-lvm-db-readable' from dual;
```




### 번호 순서로 완료하는 원복·정리

7. **LVM 종료 및 실패 반환.** actualPDB file positive→미존재prefix→positive 및SQLmarker복귀를읽고SQL/선택SSHsession을종료해87에반환한다.제한된OS결과는LIMITED_LAYOUT_EVIDENCE로남기고umount/pv/vg/lv삭제나resize는없다.87owner가soleDB를종료한다.

실제 정리/원복 response, exact resource 또는rule ID,완료UTC와남은owner/expiry를비밀없는개인실행기록에남긴다.위의guarded복구가이미완료된경우중복mutation하지않는다.

## 결과검증

**기대 패턴(미실행):** actual LVM Console 선택값, tablespace/file rows, roadmap-lvm-db-readable. OS 읽기 결과가 없어도 서비스 옵션+실제 SQL 접근을 기록할 수 있다. ASM branch 선택 시 LVM 미선택을 분명히 기록한다.

실행 기록(미기입): 실행 UTC __ / 실제 resource OCID __ / 요청·SQL 실제 응답 __ / 공식 조건 차이 __ / 정리 완료 UTC __.

## 문제해결

lsblk MOUNTPOINTS 컬럼이 없으면 현재 버전 `lsblk --help`를 확인해 MOUNTPOINT를 사용한다. 파일 path만으로 unsupported RAC/LVM 조합을 추정하지 않는다. OS 조회 permission denied를 위험한 sudo/format으로 우회하지 않는다.

## 리소스정리

본 checkpoint에서 OS storage 삭제/umount를 하지 않는다. 90 선택 backup 잔존을 확인한 뒤 87 DB terminate와 전용 NSG rule 정리를 따른다. shared VCN/client는 보존한다.

### 소유자별 종료와 exact 대상 재조회
92 owner는 SQL/SSH session 종료·storage option/allocation 변경 없음을87에 반환한다. filesystem umount/pv/vg/lv 삭제는 없다. selected LVM sole DB 종료는87 owner가 수행한다.

실제 정리 UTC __ / exact resource 또는 rule ID __ / 삭제 후 조회 응답 __ / 잔존 resource·만료·owner __. 승인 없는 잔존 비용은 다음 실습을 이유로 연장하지 않는다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 개념은 해당 생성·변경·조회 단계에서 입력 선택과 실제 결과 해석을 위해 사용하며, 공식 예시는 실제 실습 실행 결과가 아닙니다.

- [41] [About DB Systems — LVM](https://docs.oracle.com/en/cloud/paas/base-database/about-dbs/index.html)
  - 확인 범위: Fast Provisioning Option.

### 실습 task 본문 근거

- [T1] [about-dbs / index.html](https://docs.oracle.com/en/cloud/paas/base-database/about-dbs/index.html) — 본문 절차·입력 필드·지원 조건을 읽어 실습에 적용.
- [T2] [create-dbs-new / index.html](https://docs.oracle.com/en/cloud/paas/base-database/create-dbs-new/index.html) — 본문 절차·입력 필드·지원 조건을 읽어 실습에 적용.
- [T-P] [Policy Details for Base Database Service](https://docs.oracle.com/en/cloud/paas/base-database/iam-policy-details/index.html) — LaunchDBSystem/backup/restore API별 권한.


### 사용한 읽기 SQL의 정확한 Oracle Reference
- [DBA_DATA_FILES](https://docs.oracle.com/en/database/oracle/oracle-database/19/refrn/DBA_DATA_FILES.html) — 조회column의의미;새disk/volume생성근거가아님.
- [Oracle Linux LVM tutorial: pvs/vgs/lvs readback](https://docs.oracle.com/en/learn/ol-lvm/) — PV/VG/LV 출력 해석만 참고. 일반 Linux 교육용 생성/format/resize 절차를 Base Database managed storage에 적용하지 않는다. 추가 출력 column은 설치된 `lvs/vgs/pvs --help`에서 지원 여부를 확인하며 미지원이면 기본 읽기 출력과 LIMITED_LAYOUT_EVIDENCE를 기록한다.
