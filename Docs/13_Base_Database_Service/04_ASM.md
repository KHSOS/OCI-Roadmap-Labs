# ASM

> 문서 작성 검토와 실행 검증은 별개입니다. OCI 실행은 **USER_EXECUTION_PENDING**이며 사용자 캡처는 별도 자료입니다. 미실행 자체는 문서 작성 승인 차단 사유가 아닙니다. 실제 결과 필드는 실행자가 채웁니다.

> 실행용 한국어 교육 문서. OCI 리소스 생성·명령 실행 결과는 아직 검증하지 않았으며 모든 기대 패턴은 미실행이다.

## 학습목표

- 87 생성 전에 지원 shape/Oracle Grid Infrastructure를 승인하고 동일 single-node ASM DB를 한 번 생성한다. LVM 선택이면 ASM 미선택으로 남긴다.
- 승인 DBA의 실제 ASM environment에서 diskgroup name/state/total/free를 읽고 별도 database session의 file path와 대응시킨다.
- 비존재 diskgroup-name 필터와 정상 rows 복귀를 비교한 뒤 session을 종료하여87에 반환한다. diskgroup 생성/삭제·추가 DB는 실행하지 않는다.

실제 OCI 요청·응답/SQL·정리 증거는 실행자가 기록한다. 아래 기대값은 실측값이 아니다.

## 대응 Roadmap 항목

- 항목명: **ASM**
- Roadmap 경로: **Base Database Service → ASM**
- Node ID: `sequence-10-4`
- 상위 항목: **Base Database Service** (`sequence-10-0`)
- 문서 범위: 이 항목의 개념, 실제 설정 절차와 학습자 실행 검증.
- [전체 항목 목록](../README.md)

## 사전준비

**시작 조건:** 87 Submit 전ASM offer 확인 preparation;선택ASM 생성 후87 SQL 완료

- [Base Database Service](00_Base_Database_Service.md#db-sql-verification) — Create 4단계 전에 ASM/LVM 지원·승인값 결정. 동일 DB system 1회 생성 완료.

- [87 생성 전 선택](00_Base_Database_Service.md#db-create-review) — preparation-only: **87 Create 제출 전** ASM지원 shape/Oracle Grid Infrastructure를 결정한다. 아래 생성 후 SQL 링크는 별도 resource dependency이며 생성 전 완료 조건이 아니다.

### 87 network-ready 소유권 인수
[87 db-network-ready](00_Base_Database_Service.md#db-network-ready)의 actual retained external NSG OCID/owner/expiry/rule IDs/DB VNIC membership readback을 인수한다.25 임시 NSG는 prerequisite 출력이 아니다. 계약 미완료이면 DB 생성 및 본 branch는 BLOCKED_DB_NSG_NOT_READY다. 새 NSG/DB를 만들지 않으며 종료는87 creator→network owner의 exact added-rule 반환 순서다.

### 준비 완료 출력과 인수 확인

| 인수 항목 | 실제 출력·출처 | 다음 작업의 사용 위치 |
|---|---|---|
| 선행 완료 상태 | 87 Submit 전ASM offer 확인 preparation;선택ASM 생성 후87 SQL 완료 | 첫 변경 전 재조회 |
| 비밀 없는 실행 입력 | same actualDB/node/ASM option,approved GI owner/home/ASM SID,별도DBPDBservice,NSG계약/expiry | 아래 numbered Console/명령 단계 |
| 반환/후속 인계 | ASM 및별도DB session exit·read-only diskgroup/file 증거를87에 반환 | 종료 UTC 이전 exact owner 반환 |

실제 Compartment/Region은 승인 Console에서 읽고 동일 identity/Region을 고정한다. 87의기존VCN SQL client에서 `sqlplus -v` 및실제PDB서비스접속상태를확인한다.Console조회와SQL session의실행위치를구분한다. placeholder·명칭만 인수한 상태는 준비 완료가 아니다. 비밀번호·키·확인 token은 출력표에 적지 않는다.

## 권한

87 creator의Launch/Terminate 권한과이consumer의읽기권한을분리한다.본topic의scope는같은실제DB Compartment다.

storage option 생성은87 LaunchDbSystem 권한을 공유한다. read metadata=DB_SYSTEM_INSPECT/DATABASE_INSPECT; ASM catalog SQL 또는 SYSASM은 OS/DB administrator approval이며 OCI IAM grant와 다르다. ASM 내부 diskgroup CREATE/DROP 권한을 본 과제에 부여하지 않는다.

operation 근거: [공식 권한/작업 본문](https://docs.oracle.com/en/cloud/paas/base-database/iam-policy-details/index.html). family grant는 minimum-permission 정책으로 주장하지 않는다. 실행 전에 관리자 승인된 identity/domain·실제 Compartment scope를 확인한다.

## 비용및주의사항

87 제출 전 **현재 ASM 지원 offer/shape/version/capacity의 별도 승인**이 있어야 선택한다. 기본 VM.BaseDB.x86/LVM을 만든 뒤 ASM 비교용DB/Compute volume을 추가하지 않는다. 지원확인 불가면 BLOCKED_ASM_OFFER_UNVERIFIED, LVM 선택이면 NOT_SELECTED다. 선택ASM의 compute/license/data/recovery/software total은87의 sole 생성 예산에 포함하고 diskgroup 조회 자체를 무료 provision으로 주장하지 않는다. ASM/GI 관리권한 승인과 종료 UTC를 확인하며 diskgroup CREATE/DROP는 금지한다.

## 실습구성

### 워크숍 순서와 입력값

**진행 순서:** 87 생성전 ASM지원shape 선택 →동일 soleDB생성 →approved ASM session diskgroup →별도DBdatafile →87 반환

**선택 이유:** standalone ASM/Compute volume 생성이 아닌 Base Database 내부 옵션 확인이다.

| 입력 | 값 또는 정확한 출처 | 실행 위치 | 생성 전/후 | 이유 |
|---|---|---|---|---|
| shape/option | 87생성전ASM지원single-node /OracleGridInfrastructure | Console | 전 | x86generationagnostic·A1제외 |
| ASM_SID/DB_SID | 승인DBA의actualpmonSID별도 | DB node | 후 | ASM·DBsession구분 |


값이 다른 터미널에 자동 전파되지 않습니다. Console readback으로 비밀 없는 값만 인계하며, placeholder는 실제값으로 교체한 뒤 실행합니다. 새 리소스 생성 전 이름 충돌·소유자·최대 비용·종료 UTC를 확인하고 선택하지 않은 분기는 **NOT_SELECTED**로 기록합니다.


Base Database internal managed ASM 옵션 checkpoint다. 선택ASM system에서만diskgroup과별도database file path를조회한다.LVM system이면91=NOT_SELECTED이며92에서LVM을검증한다.

### 생성 전 현행 지원 확인
[87의 public retirement/shape 지원 근거](00_Base_Database_Service.md#db-current-shape-support)를 먼저 읽는다. ASM은 current 신규 offer/shape/version/capacity가 확인되지 않으면 BLOCKED_ASM_OFFER_UNVERIFIED다. [Oracle About DB Systems](https://docs.oracle.com/en/cloud/paas/base-database/about-dbs/index.html)의2026-07-31 retirement/KB921246를 선택 시점에 확인하며 새 DB를 추가하지 않는다.


### 이번 실습의 생성·재사용·앱 경계

| 구분 | 이번 주제의 실제 범위 |
|---|---|
| 새로 생성 | 독립리소스없음 |
| 재사용 | 87에서선택승인한soleASM DB |
| 앱/작업부하 및 변경 | 별도ASM/DB session 읽기;standaloneASM/newvolume 없음 |
| 실행/반환 순서 | 87 Submit 전ASM offer 확인 preparation;선택ASM 생성 후87 SQL 완료 → ASM 및별도DB session exit·read-only diskgroup/file 증거를87에 반환 |

새로 생성하지 않는 항목을선행완료로가장하지않는다.위표의실제OCID/주소와진행순서를첫mutation전에확인한다.

## 실습절차

### ASM 분기 선택과 실제 내부 파일 확인
91은 standalone ASM 생성 실습이 아니다.87의 single-node ASM 지원 shape에서 Oracle Grid Infrastructure storage 선택·DB system 생성이 생성 이벤트다. VM.BaseDB.x86/LVM 기본으로 생성한 경우 ASM branch NOT_SELECTED이고 다른 system을 만들지 않는다.
ASM query는 승인된 DBA가 actual ASM environment에서 read-only 조회하며 PDB의 SYSTEM 연결만으로 ASM diskgroup 정보 권한을 보장하지 않는다. CDB/ASM service 및 account를 owner에게 확인한다. 실제 rows/file +prefix를 기록한다. 승인 환경/권한이 없으면 ASM 실행 BLOCKED이지 빈 rows로 PASS가 아니다. DB의 ASM diskgroup 자체 삭제/수동 add는 금지하고87 terminate에 귀속한다.

[87 생성 전 선택 review](00_Base_Database_Service.md#db-create-review)는 preparation-only이고 [87 SQL 완료](00_Base_Database_Service.md#db-sql-verification)는 생성 후 resource dependency다. 최종 삭제는 [87 owner cleanup](00_Base_Database_Service.md#db-cleanup)을 따른다.

### 실행 단계

1. 87의 Create review 이전 Change storage → **Oracle Grid Infrastructure**가 ASM 옵션임을 확인한다. Standard provisioning/ASM은 Grid Infrastructure 및 ASM diskgroups로 Oracle-managed database files를 관리한다. **RAC는 ASM 필요**, **Ampere A1 및 generation-agnostic x86 shapes는 LVM 전용**이라는 shape 조건을 적용한다.[T1]
2. 비용 승인된 ASM 지원 shape로 처음부터 생성한 경우 기존 node의 승인된 SSH 접속을 사용한다. root로 disk를 변경하지 않는다. DB administrator가 이미 제공한 환경에서 read-only SQL을 실행한다. 권한이 없다면 `SYSASM`/`SYSDBA`를 무단 획득하지 말고 승인 계정의 조회 지원을 요청한다.

### Explicit approved ASM environment and separate database session
ASM선택system에서만승인DBA가**DBnodeguest**의actualGIowner/grid환경을사용한다. 계정명이grid가아니면actualowner를확인한다. 권한자동획득이아니라이미승인된DBA작업이다.
```bash
# Already approved ASM/GI owner prompt on actual DB node; no unauthorized account transition.
set -euo pipefail
id -un
hostname
pgrep -af asm_pmon
export ORACLE_SID='<DBA verified actual ASM SID>'
export ORACLE_HOME='<DBA verified actual Grid Infrastructure ORACLE_HOME>'
# DBA supplies approved owner and home from actual installation inventory, not guessed grid paths.
test -x "$ORACLE_HOME/bin/sqlplus"
export PATH="$ORACLE_HOME/bin:$PATH"
"$ORACLE_HOME/bin/sqlplus" / as sysasm
```
### ASM identity 확인 (diskgroup query보다 먼저)
87 승인 관리 SSH로 exact DB node에 접속한다. `opc` prompt이면 승인 DBA가 실제 GI owner로 전환하는 **조직 승인 계정 절차**를 수행한다(무승인 sudo/SYSASM 획득 없음). DBA가 실제 설치 inventory의 GI home, OS owner, asm_pmon SID를 제공하고 `id -un`/hostname/ORACLE_HOME/ORACLE_SID를 서로 대조한다. 명령이 없거나 승인 정보가 없으면 BLOCKED_ASM_ENVIRONMENT다. SQL prompt에서 다음 **읽기 전용** identity를 먼저 실행하고 실제 ASM SID/host/instance_type=asm을 확인한 후에만 v$asm_diskgroup을 읽는다.
```sql
select instance_name,host_name,status from v$instance;
show parameter instance_type
```
일반 DB instance면 종료하고 actual ASM 환경을 DBA와 재확인한다. .oraenv 존재를 가정하거나 DB ORACLE_HOME을 GI home으로 대체하지 않는다. [ASM environment 공식 본문](https://docs.oracle.com/en/database/oracle/oracle-database/19/ostmg/asm-instance.html)을 따른다.


ASM SQLprompt에서본문v$asm_diskgroup 및 아래 읽기전용 negative/복귀 query를 실행한 뒤`exit`로종료한다. 다음은**별도databaseoracle환경/PDBsession**이며87actualSYSTEMPDBconnection또는approveddatabasecatalogsession으로본문v$datafile/dba_data_files를조회한다. ASMinstance에서v$datafile를실행하지않는다. SYSASM승인이없으면DBA가read-only결과를제공하고branch권한제약을기록한다. standaloneASM생성/ASMdiskgroup삭제는없다.

```sql
-- ASM으로 생성한 system의 승인된 SYSASM/ASM catalog session에서 실행
-- 승인 DBA가 ASM instance session에서 실행: ASM diskgroup 조회
select name, state, type, total_mb, free_mb from v$asm_diskgroup;
-- 여기서 ASM session을 종료한다. 다음 datafile query는 별도 DB CDB/PDB session이다.
```
2a. 승인된 **database CDB/PDB** session에 별도로 연결해 `select name from v$datafile order by file#;`를 실행한다. ASM instance와 database instance는 다른 session이며 `+DATA/...` file path 검증은 database query다.
3. **LVM으로생성한기본트랙이면91=NOT_SELECTED**를기록하고ASM실행단계를중단한다.LVM선택/file path 검증은92를수행하며이를ASM완료로대체하지않는다.
4. 실제 ASM branch 결과에는 diskgroup name/state/redundancy/total/free와 `+DATA/...` 같은 file prefix를 기록한다. 이름이 DATA/RECO라고 미리 고정하지 않고 actual names를 읽는다. ASM의 total_mb는 OCI total provisioned storage와 동일 정의가 아니다.
5. ASM을 배우기 위해 Compute block volume을 attach하거나 `asmca`, `CREATE DISKGROUP`, fdisk/mkfs를 실행하지 않는다. 여기서 ASM은 Base Database 내부 관리 옵션이며 별도 standalone cluster 실습이 아니다.


### 진행·종료 판정
설정 readback·기능시험·원복·정리를 각각 실제 응답으로 기록한다. 기대값은 미실행이며 실패/만료 시 새 mutation을 중단하고 아래 topic별 종료 단계로 반환한다.

### 읽기 전용 negative control와 정상 query 복귀
승인된 **ASM session**에서 actual diskgroup positive rows를 먼저 읽는다. 실제 존재하지 않는다고 확인한 이름으로만 filter하고 원래 query로 복귀한다. 이는 읽기 filter 비교이며 ASM stop/drop 시험이 아니다.
```sql
select name, state, total_mb, free_mb from v$asm_diskgroup;
select name from v$asm_diskgroup where name='ROADMAP_NOT_CREATED_91';
select name, state, total_mb, free_mb from v$asm_diskgroup;
exit
```
이후 별도87 PDB session을 사용한다. ASM query 권한 실패를 no rows와 혼동하지 않는다.




### 번호 순서로 완료하는 원복·정리

6. **ASM 종료 및 실패 반환.** actualASM positive→없는name→positive query뒤ASM SQLexit와별도DB SQLexit를확인해87에반환한다.미선택/환경미승인은그대로남기고diskgroupDROP·새ASMsystem생성은없다.87owner만soleDB를종료한다.

실제 정리/원복 response, exact resource 또는rule ID,완료UTC와남은owner/expiry를비밀없는개인실행기록에남긴다.위의guarded복구가이미완료된경우중복mutation하지않는다.

## 결과검증

**기대 패턴(미실행):** 선택 ASM branch는 actual diskgroups/ASM file paths, LVM default branch는 실제 LVM 선택 및 file paths. **ASM 미선택**은 실행 branch 제약으로 명시한다. 비교만 한 ASM을 실습 완료로 과장하지 않는다.

실행 기록(미기입): 실행 UTC __ / 실제 resource OCID __ / 요청·SQL 실제 응답 __ / 공식 조건 차이 __ / 정리 완료 UTC __.

## 문제해결

ORA-01031이면 승인된 catalog/ASM 계정으로 조회 지원을 받는다. VM.BaseDB.x86에서 ASM 메뉴가 없으면 official shape 제한에 맞는 정상 동작이다. managed disk를 손대며 옵션을 바꾸지 않는다.

## 리소스정리

read-only checkpoint이므로 별도 diskgroup 삭제 없음. DB system owner 87의 terminate 절차가 내부 managed storage를 정리한다. 본 문서에서 ASM DROP DISKGROUP 명령을 실행하지 않는다.

### 소유자별 종료와 exact 대상 재조회
91 owner는 ASM SQL exit 및 별도 DB session exit를 확인하고 diskgroup 변경 없음/option/actual rows를87에 반환한다. diskgroup 직접 삭제는 없다. selected ASM sole system terminate는87 owner만 수행한다.

실제 정리 UTC __ / exact resource 또는 rule ID __ / 삭제 후 조회 응답 __ / 잔존 resource·만료·owner __. 승인 없는 잔존 비용은 다음 실습을 이유로 연장하지 않는다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 개념은 해당 생성·변경·조회 단계에서 입력 선택과 실제 결과 해석을 위해 사용하며, 공식 예시는 실제 실습 실행 결과가 아닙니다.

- [41] [About DB Systems — ASM](https://docs.oracle.com/en/cloud/paas/base-database/about-dbs/index.html)
  - 확인 범위: Fast Provisioning Option.

### 실습 task 본문 근거

- [T1] [about-dbs / index.html](https://docs.oracle.com/en/cloud/paas/base-database/about-dbs/index.html) — 본문 절차·입력 필드·지원 조건을 읽어 실습에 적용.
- [T2] [create-dbs-new / index.html](https://docs.oracle.com/en/cloud/paas/base-database/create-dbs-new/index.html) — 본문 절차·입력 필드·지원 조건을 읽어 실습에 적용.
- [T-P] [Policy Details for Base Database Service](https://docs.oracle.com/en/cloud/paas/base-database/iam-policy-details/index.html) — LaunchDBSystem/backup/restore API별 권한.


### 사용한 읽기 SQL의 정확한 Oracle Reference
- [V-ASM_DISKGROUP](https://docs.oracle.com/en/database/oracle/oracle-database/19/refrn/V-ASM_DISKGROUP.html) — 조회column의의미;새disk/volume생성근거가아님.
- [V-DATAFILE](https://docs.oracle.com/en/database/oracle/oracle-database/19/refrn/V-DATAFILE.html) — 조회column의의미;새disk/volume생성근거가아님.
- [V$INSTANCE identity fields](https://docs.oracle.com/en/database/oracle/oracle-database/19/refrn/V-INSTANCE.html) — INSTANCE_NAME/HOST_NAME/STATUS read-only 조회.
- [ASM instance environment and SYSASM connection](https://docs.oracle.com/en/database/oracle/oracle-database/19/ostmg/asm-instance.html) — actual ASM ORACLE_SID/Grid Infrastructure ORACLE_HOME 및승인SYSASM접속만;STARTUP/CREATE절차는실행하지않음.
