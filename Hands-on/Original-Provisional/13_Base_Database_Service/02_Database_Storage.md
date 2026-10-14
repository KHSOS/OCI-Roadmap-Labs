> **기존 실습 초안 — 미검증**: 원래 93개 문서의 절차를 보존한 자료입니다. 새 튜토리얼의 품질 승인이나 OCI 실행 검증을 뜻하지 않습니다.

# Database Storage

> 문서 작성 검토와 실행 검증은 별개입니다. OCI 실행은 **USER_EXECUTION_PENDING**이며 사용자 캡처는 별도 자료입니다. 미실행 자체는 문서 작성 승인 차단 사유가 아닙니다. 실제 결과 필드는 실행자가 채웁니다.

> 실행용 한국어 교육 문서. OCI 리소스 생성·명령 실행 결과는 아직 검증하지 않았으며 모든 기대 패턴은 미실행이다.

## 학습목표

- 87 Change storage에서 Balanced·data/recovery/software/total을 실제 최소 승인값으로 선택하고 생성 후 동일 allocation을 읽는다.
- 동일 LABPDB의 DBA_DATA_FILES/DBA_FREE_SPACE/parameter를 조회해 tablespace MB와 service provisioned storage를 비교한다.
- 존재하지 않는 전용 tablespace-name 필터의 빈 결과와 정상 query 복귀를 확인하고 변경 없는 read-only checkpoint를87에 반환한다. scale-up은 실행하지 않는다.

실제 OCI 요청·응답/SQL·정리 증거는 실행자가 기록한다. 아래 기대값은 실측값이 아니다.

## 대응 Roadmap 항목

- 항목명: **Database Storage**
- Roadmap 경로: **Base Database Service → Database Storage**
- Node ID: `sequence-10-2`
- 상위 항목: **Base Database Service** (`sequence-10-0`)
- 문서 범위: 이 항목의 개념, 실제 설정 절차와 학습자 실행 검증.
- [전체 항목 목록](../../../Docs/README.md)

## 사전준비

**시작 조건:** 생성 전87 storage review는preparation-only;생성 후같은PDB SQL 완료

- [Base Database Service](00_Base_Database_Service.md#db-sql-verification) — Create 4단계의 storage review 기록 및 단일 DB 생성/SQL 검증 완료.

### 87 network-ready 소유권 인수
[87 db-network-ready](00_Base_Database_Service.md#db-network-ready)의 actual retained external NSG OCID/owner/expiry/rule IDs/DB VNIC membership readback을 인수한다.25 임시 NSG는 prerequisite 출력이 아니다. 계약 미완료이면 DB 생성 및 본 branch는 BLOCKED_DB_NSG_NOT_READY다. 새 NSG/DB를 만들지 않으며 종료는87 creator→network owner의 exact added-rule 반환 순서다.

### 준비 완료 출력과 인수 확인

| 인수 항목 | 실제 출력·출처 | 다음 작업의 사용 위치 |
|---|---|---|
| 선행 완료 상태 | 생성 전87 storage review는preparation-only;생성 후같은PDB SQL 완료 | 첫 변경 전 재조회 |
| 비밀 없는 실행 입력 | 87 actual system/shape/storage management/data/recovery/software/total/performance,PDBservice/catalog approval | 아래 numbered Console/명령 단계 |
| 반환/후속 인계 | allocation 불변·SQL datafile/free/negative복귀를87에 반환 | 종료 UTC 이전 exact owner 반환 |

실제 Compartment/Region은 승인 Console에서 읽고 동일 identity/Region을 고정한다. 87의기존VCN SQL client에서 `sqlplus -v` 및실제PDB서비스접속상태를확인한다.Console조회와SQL session의실행위치를구분한다. placeholder·명칭만 인수한 상태는 준비 완료가 아니다. 비밀번호·키·확인 token은 출력표에 적지 않는다.

## 권한

87 creator의Launch/Terminate 권한과이consumer의읽기권한을분리한다.본topic의scope는같은실제DB Compartment다.

Console allocation 조회는 DB_SYSTEM_INSPECT, database metadata는 DATABASE_INSPECT. SQL DBA_DATA_FILES/DBA_FREE_SPACE/V$PARAMETER 읽기는 승인 DB catalog 권한이며 OCI database-family로 자동 부여되지 않는다. storage scale-up DB_SYSTEM_UPDATE는 여기서 사용하지 않는다.

operation 근거: [공식 권한/작업 본문](https://docs.oracle.com/en/cloud/paas/base-database/iam-policy-details/index.html). family grant는 minimum-permission 정책으로 주장하지 않는다. 실행 전에 관리자 승인된 identity/domain·실제 Compartment scope를 확인한다.

## 비용및주의사항

새 storage/volume 생성이나 scale-up 없이87의 data/recovery/software/total 및 performance를 읽는다. 데이터256GB 예를 전체 과금량으로 사용하지 않고 실제 Console total과 현재 계약 단위를 승인값과 대조한다. SQL logical free_mb는 서비스 provisioned allocation의 환불가능 여유분이 아니다. irreversible 증가·성능변경은 별도 범위이며 실행하지 않는다. 관찰 후87로 반환하고 같은DB 비용을 주제별 중복 승인/생성하지 않는다.

## 실습구성

### 워크숍 순서와 입력값

**진행 순서:** 87 생성전 allocation 선택 →같은system actualtotal →PDBdatafile/free/query →87 반환

**선택 이유:** data/recovery/software 및 logical free extents는 서로 다른 계층이다. resize는 하지 않는다.

| 입력 | 값 또는 정확한 출처 | 실행 위치 | 생성 전/후 | 이유 |
|---|---|---|---|---|
| allocation/performance | actualdata·recovery·software·total /Balanced | Console | 전/후 | 예712GB를실제값으로오인금지 |
| PDB/service | 87actualLABPDBconnection | SQLPlus | 후 | logicalcatalog 별도계층 |


값이 다른 터미널에 자동 전파되지 않습니다. Console readback으로 비밀 없는 값만 인계하며, placeholder는 실제값으로 교체한 뒤 실행합니다. 새 리소스 생성 전 이름 충돌·소유자·최대 비용·종료 UTC를 확인하고 선택하지 않은 분기는 **NOT_SELECTED**로 기록합니다.


동일 DB의 logical tablespace/free space와 서비스의 data/recovery/total allocated storage를 구별한다. storage scale-up 없음.

### 생성 전 현행 지원 확인
[87의 public retirement/shape 지원 근거](00_Base_Database_Service.md#db-current-shape-support)를 먼저 읽는다. VM.BaseDB.x86 single-node/LVM 기본의 actual edition/version/storage 조합을 재조회하고 다른 shape로 자동 대체하지 않는다. [Oracle About DB Systems](https://docs.oracle.com/en/cloud/paas/base-database/about-dbs/index.html)의2026-07-31 retirement/KB921246를 선택 시점에 확인하며 새 DB를 추가하지 않는다.


### 이번 실습의 생성·재사용·앱 경계

| 구분 | 이번 주제의 실제 범위 |
|---|---|
| 새로 생성 | 독립storage/volume없음 |
| 재사용 | 87 soleDB allocation/PDB SQLclient |
| 앱/작업부하 및 변경 | 생성전storage 선택+생성후read-only logical/physical 비교 |
| 실행/반환 순서 | 생성 전87 storage review는preparation-only;생성 후같은PDB SQL 완료 → allocation 불변·SQL datafile/free/negative복귀를87에 반환 |

새로 생성하지 않는 항목을선행완료로가장하지않는다.위표의실제OCID/주소와진행순서를첫mutation전에확인한다.

## 실습절차

### allocated storage·SQL free space 검증
storage choice는87 Submit 전에 preparation checkpoint이며 SQL은 생성 후 dependency다. ASM/LVM, data/recovery/software reserve/total 및 performance를 review와 actual detail 양쪽에서 비교한다. 현재 Console이 다른 최소값을 보이면 기존712GB 예를 그대로 주문하지 않고 예산 재승인을 받는다.
SQL `dba_free_space`가 특정 tablespace row를 안 보여도 즉시 데이터 손실로 판단하지 않는다. fully allocated extents/권한/actual tablespace와 대조한다. 장애·복구는 query 권한/잘못된 container를 올바른 승인 PDB connection으로 고치는 범위며 datafile 삭제/resize는 하지 않는다. read-only 완료 후 정리는87 terminate 및 잔존 backup 확인이다.

[87 생성 전 선택 review](00_Base_Database_Service.md#db-create-review)는 preparation-only이고 [87 SQL 완료](00_Base_Database_Service.md#db-sql-verification)는 생성 후 resource dependency다. 최종 삭제는 [87 owner cleanup](00_Base_Database_Service.md#db-cleanup)을 따른다.

### 실행 단계

1. 87 Create → Change storage에서 **LVM/ASM**, **Balanced/Higher performance**, **Available data storage**, **Recovery area**, **Total storage** 필드를 비교하고 승인값을 선택한다. 본 checkpoint는 이 선택에 연결되며 storage를 추가 생성하지 않는다.[T1]
2. 같은 DB system detail에서 실제 data storage, recovery storage, total storage, performance를 기록한다. Current official table는 256 data + 256 recovery + 200 software reserve=712 GB total 예를 제시한다. 현재 Console이 다른 값을 제시하면 Console/정식 계약 값을 우선하고 차이·shape를 기록한다.[T2]
3. 87의 실제 PDB client connection에서 읽기 SQL을 실행한다.
```sql
select tablespace_name, round(sum(bytes)/1024/1024) mb
from dba_data_files group by tablespace_name order by tablespace_name;
select tablespace_name, round(sum(bytes)/1024/1024) free_mb
from dba_free_space group by tablespace_name order by tablespace_name;
select name, value from v$parameter where name in ('db_create_file_dest','db_recovery_file_dest');
```
4. PDB datafile MB 합계가 OCI allocated data storage와 같지 않은 이유를 기록한다: allocation, shared CDB files, recovery area/software reserve, file free extents는 다른 계층이다. DB storage 전체를 `df` 숫자 하나로 판정하지 않는다.
5. LVM fast provisioning의 최대 scale-up 가능 값은 initial storage에 의존한다. scale-up은 비용 증가·축소 불가 조건을 공식 Scale 문서와 확인한 후 별도 승인에서만 수행한다. 본 실습은 용량 변경하지 않는다.


### 진행·종료 판정
설정 readback·기능시험·원복·정리를 각각 실제 응답으로 기록한다. 기대값은 미실행이며 실패/만료 시 새 mutation을 중단하고 아래 topic별 종료 단계로 반환한다.

### 읽기 전용 negative control와 정상 query 복귀
87의 실제 LABPDB SQL*Plus에서 다음 읽기만 수행한다. 첫/마지막 query의 actual rows를 비교하고 중간의 승인 미사용 tablespace 이름은 no rows이어야 한다. 이름이 실제 존재하면 미사용 이름을 확인한다. CDB root에 잘못 연결한 것을 negative 성공으로 쓰지 않는다.
```sql
show con_name
select tablespace_name from dba_data_files order by tablespace_name;
select tablespace_name from dba_data_files where tablespace_name='ROADMAP_NOT_CREATED_89';
select tablespace_name from dba_data_files order by tablespace_name;
```




### 번호 순서로 완료하는 원복·정리

6. **Database Storage 종료 및 실패 반환.** 정상tablespacequery복귀·samePDB와Console allocation/performance불변을읽고87에반환한다.storage/datafile삭제·scale축소는하지않는다.서비스전체종료는87 soleDB gate에귀속된다.

실제 정리/원복 response, exact resource 또는rule ID,완료UTC와남은owner/expiry를비밀없는개인실행기록에남긴다.위의guarded복구가이미완료된경우중복mutation하지않는다.

## 결과검증

**기대 패턴(미실행):** 실제 tablespace rows와 destination parameter, Console allocation 값. 특정 tablespace MB를 fabricated expected output로 고정하지 않는다. actual allocation과 SQL free_mb가 다른 계층임을 설명한다.

실행 기록(미기입): 실행 UTC __ / 실제 resource OCID __ / 요청·SQL 실제 응답 __ / 공식 조건 차이 __ / 정리 완료 UTC __.

## 문제해결

ORA-00942/01031은 catalog view 권한을 확인한다. root/system disk 부족으로 오해하여 mount/mkfs/lvextend를 실행하지 않는다. 데이터파일 path와 allocation을 ASM/LVM 선택에 맞춰 해석한다.

## 리소스정리

읽기 checkpoint 자체는 삭제 리소스 없음. capacity/performance를 바꾸지 않았는지 Console에서 확인 후 87 공통 삭제를 따른다.

### 소유자별 종료와 exact 대상 재조회
89 owner는 original/actual allocation·performance 불변 및 PDB query 복귀를 기록해87에 반환한다. datafile/volume 삭제나 scale 축소를 하지 않는다. 서비스 storage 정리는87 terminate로만 수행한다.

실제 정리 UTC __ / exact resource 또는 rule ID __ / 삭제 후 조회 응답 __ / 잔존 resource·만료·owner __. 승인 없는 잔존 비용은 다음 실습을 이유로 연장하지 않는다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 개념은 해당 생성·변경·조회 단계에서 입력 선택과 실제 결과 해석을 위해 사용하며, 공식 예시는 실제 실습 실행 결과가 아닙니다.

- [41] [About DB Systems — Database Storage](https://docs.oracle.com/en/cloud/paas/base-database/about-dbs/index.html)
  - 확인 범위: A DB system can have only.

### 실습 task 본문 근거

- [T1] [create-dbs-new / index.html](https://docs.oracle.com/en/cloud/paas/base-database/create-dbs-new/index.html) — 본문 절차·입력 필드·지원 조건을 읽어 실습에 적용.
- [T2] [about-dbs / index.html](https://docs.oracle.com/en/cloud/paas/base-database/about-dbs/index.html) — 본문 절차·입력 필드·지원 조건을 읽어 실습에 적용.
- [T-SCALE] [Scale a DB System](https://docs.oracle.com/en/cloud/paas/base-database/scale-dbs/index.html) — 별도승인scale제약참고이며본실습은scale하지않음.
- [T-P] [Policy Details for Base Database Service](https://docs.oracle.com/en/cloud/paas/base-database/iam-policy-details/index.html) — LaunchDBSystem/backup/restore API별 권한.


### 사용한 읽기 SQL의 정확한 Oracle Reference
- [DBA_DATA_FILES](https://docs.oracle.com/en/database/oracle/oracle-database/19/refrn/DBA_DATA_FILES.html) — 조회column의의미;새disk/volume생성근거가아님.
- [DBA_FREE_SPACE](https://docs.oracle.com/en/database/oracle/oracle-database/19/refrn/DBA_FREE_SPACE.html) — 조회column의의미;새disk/volume생성근거가아님.
- [V-PARAMETER](https://docs.oracle.com/en/database/oracle/oracle-database/19/refrn/V-PARAMETER.html) — 조회column의의미;새disk/volume생성근거가아님.
