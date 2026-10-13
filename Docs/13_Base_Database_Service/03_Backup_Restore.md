# Backup / Restore

> 문서 작성 검토와 실행 검증은 별개입니다. OCI 실행은 **USER_EXECUTION_PENDING**이며 사용자 캡처는 별도 자료입니다. 미실행 자체는 문서 작성 승인 차단 사유가 아닙니다. 실제 결과 필드는 실행자가 채웁니다.

> 실행용 한국어 교육 문서. OCI 리소스 생성·명령 실행 결과는 아직 검증하지 않았으며 모든 기대 패턴은 미실행이다.

## 학습목표

- 별도 비용/downtime/retention 승인 후 기존 지원 destination에서 automatic backup을 활성화·재조회한다. LTR/새 DB/새 network는 선택하지 않는다.
- 본인 marker before-backup 저장→backup 완료→실제 recovery UTC 확보→after-backup commit을 수행하고 같은 DB를 해당 timestamp로 restore한다.
- 같은 PDB 재접속으로 before-backup 복구를 확인하고 본인 marker 삭제·backup 잔존/만료 owner를 기록한 뒤87 terminate에 반환한다. 미승인·미지원 분기는 미실행이다.

실제 OCI 요청·응답/SQL·정리 증거는 실행자가 기록한다. 아래 기대값은 실측값이 아니다.

## 대응 Roadmap 항목

- 항목명: **Backup / Restore**
- Roadmap 경로: **Base Database Service → Backup / Restore**
- Node ID: `sequence-10-3`
- 상위 항목: **Base Database Service** (`sequence-10-0`)
- 문서 범위: 이 항목의 개념, 실제 설정 절차와 학습자 실행 검증.
- [전체 항목 목록](../README.md)

## 사전준비

**시작 조건:** 87 같은DB/client SQL 완료;별도 supported backup/downtime/retention 승인

- [Base Database Service](00_Base_Database_Service.md#db-sql-verification) — 동일 private DB 생성과 client SQL verification 완료. backup/restore 비용·다운타임·retention 별도 승인 필요.

- [Service Gateway](../03_Virtual_Cloud_Network_VCN/12_Service_Gateway.md) — 87 생성 전에 이미 구성한 **동일 actual service gateway**와 Oracle service CIDR route/egress 검증 완료. backup 작업과 retention/restore 종료까지 유지. 87에서 provisioning용으로 이미 준비한 동일 gateway를 재사용하며 backup topic 때문에 새 gateway를 추가하지 않는다. 운영 gateway 변경 금지.

### 87 network-ready 소유권 인수
[87 db-network-ready](00_Base_Database_Service.md#db-network-ready)의 actual retained external NSG OCID/owner/expiry/rule IDs/DB VNIC membership readback을 인수한다.25 임시 NSG는 prerequisite 출력이 아니다. 계약 미완료이면 DB 생성 및 본 branch는 BLOCKED_DB_NSG_NOT_READY다. 새 NSG/DB를 만들지 않으며 종료는87 creator→network owner의 exact added-rule 반환 순서다.

### 준비 완료 출력과 인수 확인

| 인수 항목 | 실제 출력·출처 | 다음 작업의 사용 위치 |
|---|---|---|
| 선행 완료 상태 | 87 같은DB/client SQL 완료;별도 supported backup/downtime/retention 승인 | 첫 변경 전 재조회 |
| 비밀 없는 실행 입력 | actual database/backup destination existing foundation,retained NSG/OSNpath,actual recoverable window 근거,marker ownership receipt/run_id,expiry | 아래 numbered Console/명령 단계 |
| 반환/후속 인계 | marker cleanup/backup 잔존만료·owner를87에 반환;window 부재시UPDATE 금지 | 종료 UTC 이전 exact owner 반환 |

실제 Compartment/Region은 승인 Console에서 읽고 동일 identity/Region을 고정한다. 87의기존VCN SQL client에서 `sqlplus -v` 및실제PDB서비스접속상태를확인한다.Console조회와SQL session의실행위치를구분한다. placeholder·명칭만 인수한 상태는 준비 완료가 아니다. 비밀번호·키·확인 token은 출력표에 적지 않는다.

## 권한

본인same database의승인backup/restore operation만수행한다.87 Launch/Terminate/network grant를최소권한으로재복사하지않는다.

CreateBackup=DB_BACKUP_CREATE+DATABASE_CONTENT_READ; Get/ListBackup=DB_BACKUP_INSPECT; RestoreDatabase=DB_BACKUP_INSPECT+DB_BACKUP_CONTENT_READ+DATABASE_CONTENT_WRITE; DeleteBackup=DB_BACKUP_DELETE+DB_BACKUP_INSPECT. scoped database-family는 이들을 포괄하지만 Recovery Service protected resource 권한·network는 destination에 따라 별도 승인이다. automatic backup 설정 UpdateDatabase=DATABASE_UPDATE이며 활성화에는DB_BACKUP_CREATE+DATABASE_CONTENT_READ가추가된다.최초mutation전에관리자가승인한다. SQL CREATE TABLE/quota·catalog/marker SELECT·DML·본인DROP승인은OCI IAM과별개다.final backup 권한/비용은자동승인하지않는다.

operation 근거: [공식 권한/작업 본문](https://docs.oracle.com/en/cloud/paas/base-database/iam-policy-details/index.html). family grant는 minimum-permission 정책으로 주장하지 않는다. 실행 전에 관리자 승인된 identity/domain·실제 Compartment scope를 확인한다.

## 비용및주의사항

**별도 유료/downtime/retention 승인 분기.** 동일 DB의 managed automatic backup 설정, on-demand backup1개, in-place timestamp restore1회만 사용한다. Object Storage full/Recovery Service incremental과 잔존 보호기간·삭제잠금·실제 termination option의 비용 owner/만료를 승인한다. restore 완료시간·당일 backup 삭제를 보장하지 않는다. supported destination foundation 또는 실제 recoverable window 증거가 없으면 mutation 전 BLOCKED이며 새 network/clone DB를 만들지 않는다. LTR은 이 in-place branch에서 선택하지 않는다. marker PURGE도 exact 생성 receipt와 별도 owner 승인이 필요하다.

## 실습구성

### 워크숍 순서와 입력값

**진행 순서:** 별도유료/downtime/retention승인 →existingnetwork destination준비 →managedbackup →marker/backup →UTCrestore →before값 →retentionfollowup

**선택 이유:** Object Storage full과Recovery Service incremental을 구분하고 자동백업/보호기간 잔존비용을 승인한다.

| 입력 | 값 또는 정확한 출처 | 실행 위치 | 생성 전/후 | 이유 |
|---|---|---|---|---|
| destination/retention | actualsupportedObjectStorage또는existingRecoveryService | Console/승인자 | 전 | 추가network자원없음/잔존승인 |
| recoverytargetUTC | 백업완료후·mutation전actualUTC | SQLPlus/Console | 후 | Latest와구분 |
| marker/backupOCID | roadmap_restore_marker /actualbackupOCID | SQLPlus/Console | 후 | 같은DB만복구 |


값이 다른 터미널에 자동 전파되지 않습니다. Console readback으로 비밀 없는 값만 인계하며, placeholder는 실제값으로 교체한 뒤 실행합니다. 새 리소스 생성 전 이름 충돌·소유자·최대 비용·종료 UTC를 확인하고 선택하지 않은 분기는 **NOT_SELECTED**로 기록합니다.


선택 분기: 동일 DB에서 on-demand backup 1개(Object Storage=Full, Recovery Service=Incremental) → 동 DB timestamp restore 1회. LTR은 in-place restore 미지원이므로 선택하지 않는다. 별도 승인 없으면 이 분기를 실행하지 않았다고 표시하고 87 SQL 검증만 완료한다.

### 생성 전 현행 지원 확인
[87의 public retirement/shape 지원 근거](00_Base_Database_Service.md#db-current-shape-support)를 먼저 읽는다. VM.BaseDB.x86 single-node/LVM 기본의 actual edition/version/storage 조합을 재조회하고 다른 shape로 자동 대체하지 않는다. [Oracle About DB Systems](https://docs.oracle.com/en/cloud/paas/base-database/about-dbs/index.html)의2026-07-31 retirement/KB921246를 선택 시점에 확인하며 새 DB를 추가하지 않는다.


### 이번 실습의 생성·재사용·앱 경계

| 구분 | 이번 주제의 실제 범위 |
|---|---|
| 새로 생성 | 본인marker table/선택on-demandbackup1개 |
| 재사용 | 87 sole DB 및 기존 supported backup foundation/SQL client |
| 앱/작업부하 및 변경 | automaticbackup 설정·before/after/동DBtimestamp restore;clone/새network 없음 |
| 실행/반환 순서 | 87 같은DB/client SQL 완료;별도 supported backup/downtime/retention 승인 → marker cleanup/backup 잔존만료·owner를87에 반환;window 부재시UPDATE 금지 |

새로 생성하지 않는 항목을선행완료로가장하지않는다.위표의실제OCID/주소와진행순서를첫mutation전에확인한다.

## 실습절차

**현재 Console destination 제한:** official create/backup task는 **2026-06-23부터** **tenancy 생성일·대상 Region·DB version 조건**에서 automatic backups의 Console destination이 Recovery Service만 노출될 수 있다고 명시한다. Object Storage를 반드시 선택할 수 있다고 가정하지 않는다. 실제 UI에서 제공되는 값과 기존 승인 network/protected-resource 준비를 확인하고, 미제공 선택을 CLI 우회로 강제하지 않는다. 90 추가 foundation/보호비용 승인이 없으면90을 NOT_SELECTED로 남긴다. 87 automatic backups off core track은 조직 정책이 허용할 때만 계속한다.

### retention과 recovery point의 승인 gate
승인에는 destination, 실제 제공 retention, immutable/delete protection, 잔존 예상 비용·종료일·cleanup owner, restore interruption과 최대 wait budget을 포함한다. backup complete timestamp와 **복구 목표 UTC**를 기록하고 after-backup update보다 이전 목표라는 것을 확인한다. PITR/backup-specific choice가 없으면 restore는 BLOCKED로 남긴다. restore 성공만으로 marker before-backup 복구를 주장하지 않는다.
marker object가 이미 있으면 drop/recreate하거나 기존 object를 재사용하지 않고 이 분기를 BLOCKED로 반환한다. SYSTEM으로 marker 생성은87의 단기 승인 계정 범위에 한정한다. 첫 CREATE 전에 아래 **Marker 삭제 gate와 실패 후 반환**을 읽고 성공 CREATE/빈 table/COMMIT 불명/소유권 불명의 반환 방식을 승인한다. 다른 session이 이 marker에 DDL/DML을 수행하지 않는 exclusive lab 사용을 owner에게 확인한다. 실패 시 before/after 실제 값·work request error를 남기고 무한 retry나 clone DB 생성으로 대체하지 않는다.
공식 [Terminate task](https://docs.oracle.com/en/cloud/paas/base-database/terminate-dbs/index.html)는 incremental 종료 제거/full standalone 잔존을 기술하지만, 최신 [Backup task](https://docs.oracle.com/en/cloud/paas/base-database/backup-db/index.html)와 [Backup/Recovery](https://docs.oracle.com/en/cloud/paas/base-database/backup-recover/index.html)는 **Retain backups according to retention** 또는 **Retain72hours, then delete** 종료 옵션을 명시한다. 문서 간 차이가 있으므로 현재 Console의 실제 deletion option·destination·policy를 기록하고 당일 전체 삭제를 보장하지 않는다. Object Storage full standalone 잔존을 별도 확인한다. full backup/Recovery Service protected resource의 실제 삭제 가능 여부를 조회하고 retention lock이면 해당 만료시각·잔존 비용 승인·owner follow-up을 남긴다. DB 삭제=모든 backup 과금 종료라고 기록하지 않는다.

### mutation 전 run receipt 취득과 PITR 입력 차단
Cloud Shell의 비밀 없는 생성 토큰을 다음으로 얻고 SQL*Plus의 기존32HEX gate에 그대로 입력한다. 재접속마다 같은 token을 사용한다.
```bash
python3 - <<'PY'
import secrets
print(secrets.token_hex(16).upper())
PY
```
CREATE 성공 직후 **외부 개인 receipt**에 `run_id / actual DB OCID / endpoint / service / PDB / SESSION_USER / CURRENT_SCHEMA / OBJECT_ID / CREATED / CREATE successful / receipt UTC / owner PURGE approval`을 기록한다. CREATED의 NLS 출력은 다음 조회로 모호함을 없애고 DB session timezone도 별도로 남긴다.
```sql
select object_id,to_char(created,'YYYY-MM-DD HH24:MI:SS') created_db_date,
       sessiontimezone,dbtimezone,object_type from user_objects
where object_name='ROADMAP_RESTORE_MARKER' and object_type='TABLE';
select to_char(systimestamp at time zone 'UTC',
       'YYYY-MM-DD"T"HH24:MI:SS.FF6"Z"') target_utc from dual;
```
Oracle DATE CREATED에는 timezone이 포함되지 않는다. 위 CREATED 값과 원래 DB identity를 exact 비교하며 UTC라고 재라벨하지 않는다. Restore timestamp UI가 timezone을 표시하면 UTC target과 **같은 instant**로 변환한다. 예컨대 `2026-10-13T12:00:00+09:00`은 다음 local 변환으로 UTC를 얻는다(샘플일 뿐 실제 target 아님).
```bash
python3 - <<'PY'
import datetime
v=datetime.datetime.fromisoformat('2026-10-13T12:00:00+09:00')
print(v.astimezone(datetime.timezone.utc).isoformat())
PY
```
**recoverable window acquisition gate:** T-RESTORE의 public body는 timestamp/SCN 선택을 설명하지만 universally available earliest/latest window field/CLI를 명시하지 않는다. retention 기간 또는 backup Active를 actual recoverable window로 대신하지 않는다. Database details → Actions → Restore에서 현재 UI가 실제 허용 range/timezone/precision을 명시하는지 읽고 **Cancel**하여 field label/value/UTC conversion/출처를 receipt에 남긴다. 실제 earliest/latest/redo 포함 근거가 제공되지 않으면 승인 DBA가 해당 backup destination의 Oracle-supported recovery evidence를 취득할 때까지 **BLOCKED_PITR_WINDOW_UNAVAILABLE: UPDATE 및 Restore 금지**다. 존재하지 않는 get-window command를 만들지 않는다. 초 정밀도 UI면 target과 mutation을 최소 다음 초로 분리하고 미지원 소수초를 잘라 성공이라고 주장하지 않는다.
Restore failed이면 DB Systems → same system → Databases → same database에서 Restore failed 상태와 Work requests의 exact operation error/log를 읽고, 승인 DBA가 **host RMAN logs**를 검토하여 수정하기 전 재Restore하지 않는다.[T-RESTORE] 임의 RMAN path/다른 restore option/clone으로 성공을 꾸미지 않는다. 위 입력 확보는 기존 session/CREATE/rowcount/COMMIT/PURGE gate를 대체하지 않는다.

### 실행 단계

1. **선택 승인 gate:** backup destination(**Object Storage** 또는 **Autonomous Recovery Service**)·retention·비용·network를 확인한다. Oracle 관리 backup은 단순 임의 bucket upload와 다르다. destination 제공 여부/Region 정책에 맞춰 선택하고 Recovery Service가 강제되는 계정은 보호 리소스 비용도 승인한다.[T1]
2. **90 선택 분기에서만** Database details → Actions → Configure automatic backups를 열어 Enable automatic backups를 켠다. 87의 기본 off에서 바뀌는 비용을 먼저 승인한다. Destination **Object Storage** 또는 **Recovery Service**를 실제 지원값으로 선택하고 retention을 선택한다(Object Storage 현재7/15/30/45/60일, Recovery Service Bronze14/Silver35/Gold65/Platinum95일 또는 승인 custom). Real-time protection 및 LTR은 별도 승인이 없으면 끈다. Backup termination deletion option도 실제 Console값을 기록한다. 저장 후 Backup destination/retention/enabled를 재조회하고 자동 backup의 recoverable window를 확인한다.[T1] Recovery Service는19.18+와 전용 database VCN network path가 필요하며 service gateway 하나만으로 충족하지 않는다. 기존 VCN service gateway/route/egress와 해당 destination 접근을 준비한다. 이워크숍은**추가자원없는분기**다. Recovery Service에필요한registeredRecoveryServiceSubnet/전용privatepath·보호리소스/IAM이이미승인·준비된경우만선택한다. 없는경우새subnet/NSG/service를이실습에서자동생성하지않고RecoveryService선택을NOT_SELECTED로남긴다. 계정이RecoveryService를강제하고기존foundation이없으면**90만조건부미선택**이며87 core SQL/저장은계속가능하다. ObjectStorage가실제제공되면existingservicegateway경로를재사용한다. Ampere A1은 Recovery Service를 destination으로 하는 provisioning/restore 제한이 있으므로 현재 shape를 공식 지원과 대조한다.[T2]

### Fail-closed SQL session과 생성 소유권 receipt
**실행 위치:** 87에서 검증한 같은 VCN client의 SQL*Plus, 같은 private endpoint/PDB administration service/승인 schema. 새 DB나 schema를 자동 생성하지 않는다. 아래는 작성한 SQL recipe이며 실제 Oracle 실행 결과가 아니다.

**수동 단계 gate:** 각 fence/statement를 하나씩 실행하고 성공 응답을 확인한 뒤만 다음으로 진행한다. SQL*Plus 종료, ORA-/PLS-/SP2- 오류, OS 오류, 예상과 다른 행 수, 연결 끊김 또는 COMMIT 응답 불명은 즉시 BLOCKED다. 여러 fence를 한꺼번에 붙여 넣거나 오류 후 다음 SQL을 실행하지 않는다. 새 세션마다 다음 보호 설정과 identity 확인을 다시 수행한다. `WHENEVER SQLERROR`는 SQL/PLSQL 오류에 적용되지만 SQL*Plus 명령 오류(SP2-)는 잡지 않으므로 화면 오류도 반드시 확인한다.[T-SQLERROR][T-OSERROR]

```sql
whenever oserror exit failure rollback
whenever sqlerror exit failure rollback
select sys_context('USERENV','CON_NAME') as pdb,
       sys_context('USERENV','SESSION_USER') as schema_owner,
       sys_context('USERENV','CURRENT_SCHEMA') as current_schema,
       sys_context('USERENV','SERVICE_NAME') as service from dual;
```
PDB=87 실제 LABPDB, SESSION_USER=CURRENT_SCHEMA=승인 계정, service=87 실제 서비스와 **모두 일치해야** 계속한다. endpoint·DB system/database OCID도87 receipt와 대조한다. 불일치/조회 실패는 아무 mutation 없이 종료한다. 승인 계정은 이 lab의 table을 만들 CREATE TABLE/quota가 필요하며 IAM이 SQL 권한을 부여하지 않는다.

본 run의 비밀 아닌 **32자리 대문자 hexadecimal run_id**를 승인 기록에 정하고 아래 한 곳의 placeholder를 교체한다. 새 세션에서도 같은 값을 다시 정의하며 새 값으로 바꾸지 않는다.
```sql
define run_id = '실제_32자리_대문자_HEX_RUN_ID'
begin
  if not regexp_like('&&run_id','^[0-9A-F]{32}$','c') then
    raise_application_error(-20001,'STOP: invalid run_id');
  end if;
end;
/
```
marker 이름 충돌은 TABLE뿐 아니라 같은 schema의 object/synonym도 확인한다. 조회 성공 및 count=0만 CREATE를 허용한다. 기존 object가 있으면 drop/recreate하지 않고 **90을 BLOCKED**로 반환한다.
```sql
declare n number;
begin
  select count(*) into n from user_objects
  where object_name='ROADMAP_RESTORE_MARKER';
  if n <> 0 then raise_application_error(-20002,'STOP: marker collision'); end if;
end;
/
```

3. 동일 승인 PDB/schema에서 다음 **CREATE만** 실행한다.
```sql
create table roadmap_restore_marker (
  id number primary key,
  run_id varchar2(32) not null,
  note varchar2(40) not null
);
```
`Table created.` 성공을 확인한 뒤 다음 조회를 실행한다. 정확히 TABLE 한 행의 OBJECT_ID/CREATED를 읽는다. **이 run의 성공 CREATE transcript + UTC + actual endpoint/database OCID/PDB/schema/service + run_id + OBJECT_ID/CREATED**를 비밀 없는 외부 실행 receipt에 기록한다. 이 receipt 없이는 INSERT/UPDATE/DROP를 허용하지 않는다. CREATE 실패는 기존 table을 재사용할 근거가 아니다. Oracle DDL은 implicit commit하므로 이후 ROLLBACK이 CREATE를 없애지 않는다.[T-COMMIT]
```sql
select object_id, created, object_type from user_objects
where object_name='ROADMAP_RESTORE_MARKER' and object_type='TABLE';
```
receipt 작성 후 INSERT를 단독 실행하고 성공 `1 row created.`를 확인한다.
```sql
insert into roadmap_restore_marker (id,run_id,note)
values (1,'&&run_id','before-backup');
```
다음 assertion 성공 후만 COMMIT을 단독 실행한다. 실패하면 rollback/종료하고 backup을 생성하지 않는다.
```sql
declare total_rows number; exact_rows number;
begin
  select count(*), count(case when id=1 and run_id='&&run_id'
    and note='before-backup' then 1 end) into total_rows,exact_rows
  from roadmap_restore_marker;
  if total_rows<>1 or exact_rows<>1 then
    raise_application_error(-20003,'STOP: before marker mismatch');
  end if;
end;
/
```
```sql
commit write wait immediate;
```
`Commit complete.`를 확인하고 `exit rollback`으로 종료한다. 87의 **같은 endpoint/PDB/schema**에 새로 접속하여 위 session/identity/run_id gate를 재수행하고 같은 before assertion 및 다음 조회를 실행한다. 새 세션의 exact row 확인 전에는 backup으로 진행하지 않는다. COMMIT 응답이 불명이면 재INSERT/reCREATE하지 않고 새 세션 조회로 상태를 조사한다.
```sql
select id,run_id,note from roadmap_restore_marker order by id;
```
성공 기대값은 **전체 정확히 한 행**, id=1/run_id=receipt 값/note=before-backup이며 실제 응답은 실행자가 기록한다.
4. Database details → Backups → Create backup. Name **roadmap-on-demand**, Backup retention **Retain backups per backup retention period**. **Specify long-term backup retention period/LTR은 선택하지 않는다.** destination은 이전 Configure automatic backups에서 승인한 실제 값이다. Object Storage는 full, Recovery Service는 incremental on-demand를 생성한다. State **Active**와 UTC completion/backup OCID를 읽고 백업 완료 전 데이터를 바꾸지 않는다.[T1] 완료 직후 marker before-backup을 읽고 `select systimestamp at time zone 'UTC' as recovery_target_utc from dual;`의 UTC 목표시각을 기록한다. archive redo 및 earliest/latest recoverable window에 이 목표가 포함되는지 확인한 뒤만 다음 mutation을 수행한다.
5. UPDATE 직전 session/identity/run_id gate·생성 receipt·OBJECT_ID/CREATED 일치·before assertion을 다시 확인한다. recovery_target_utc를 COMMIT 이후/UPDATE 이전으로 고정하고 타임존·소수초와 actual recoverable window를 기록한다. archive redo가 아직 목표까지 포함되지 않으면 승인 wait budget 안에서 재조회하고, 포함 확인 전에는 UPDATE하지 않는다. 아래 mutation block은 같은 run의 before 한 행만 변경하며 affected rows가1이 아니면 rollback/종료한다.
```sql
begin
  update roadmap_restore_marker set note='after-backup'
  where id=1 and run_id='&&run_id' and note='before-backup';
  if sql%rowcount <> 1 then
    raise_application_error(-20004,'STOP: update row count');
  end if;
end;
/
```
성공 후 step3 assertion의 note만 `after-backup`으로 바꿔 전체 한 행을 확인하고 `commit write wait immediate;`를 **단독** 실행한다. Commit complete 확인→exit rollback→같은 endpoint/PDB/schema 새 세션→session/identity/run_id gate→after assertion 및 exact row 재조회를 완료한다. 응답 불명/오류에는 Restore를 누르지 않는다. 실제 after commit UTC가 recovery target보다 엄격히 뒤인지 확인하고 recovery window가 해당 목표를 포함하는지 다시 읽는다. 초 단위만 입력 가능한 UI라 목표와 mutation이 같은 초가 되면 계속하지 말고 지원 정밀도에 맞는 새 목표/대기 계획을 승인받는다.

동 DB Restore가 중단을 유발할 수 있음을 승인받은 상태에서 Database → Actions → Restore → **Restore to a timestamp → 직전 기록한 실제 UTC recovery target**으로 복구한다.[T-RESTORE] **Latest**는 after-backup redo가 적용될 수 있으므로 선택하지 않는다. timestamp가 미제공/범위 밖이거나 최대 wait budget이 부족하면 BLOCKED이며 추가 비용 승인을 받는다. 두 번째 DB/clone/RMAN destructive command로 대체하지 않는다.
6. work request 완료 후 같은 private client/endpoint/PDB service/schema에 **새 세션**으로 접속한다. session/identity/run_id gate와 receipt OBJECT_ID/CREATED를 대조하고 step3의 **before assertion**을 실행한 뒤 `select id,run_id,note from roadmap_restore_marker order by id;`를 읽는다. **전체 정확히 한 행, id=1/동일 run_id/note=before-backup**, 정상 assertion과 actual response가 모두 있어야 데이터 복구 완료다. missing table/다른 token/extra row/after 값/권한 오류이면 복구 검증 실패다. CREATE/INSERT로 before를 다시 만들어 성공을 꾸미지 않는다. Restore는 managed automatic backup/redo chain을 사용하며 이름 붙인 on-demand backup 하나만 읽었다고 주장하지 않는다. 상태 성공만으로 데이터 복구를 주장하지 않는다.
7. marker table 삭제 및 승인 backup retention/삭제 계획을 확인한다. 배포 시간이 길어 승인된 세션을 넘으면 추가 대기/비용 승인을 요청하고 새 DB를 대안으로 생성하지 않는다.


### 진행·종료 판정
설정 readback·기능시험·원복·정리를 각각 실제 응답으로 기록한다. 기대값은 미실행이며 실패/만료 시 새 mutation을 중단하고 아래 topic별 종료 단계로 반환한다.

### on-demand와 standalone 정리 경로
full backup삭제는DBsystems→동일DB→Databases→동일database→Backups→**본인on-demandfullbackup** Actions→Delete→Confirm→목록재조회다. automaticbackup은명시적Delete가없을수있어정책만료와ownerfollowup을사용한다. standalone은DBsystems→Standalonebackups에서exactOCID로본인full만삭제·재조회한다. RecoveryServiceprotectedresource는DatabaseBackupdestination링크로actualprotectedresource/retention을확인하고승인owner가만료까지추적한다. forcedretention을지우기위해추가DB/복제/OSdisk조작을하지않는다.




### 번호 순서로 완료하는 원복·정리

8. **Backup / Restore 종료 및 실패 반환.** same-PDB actual before복구를확인해아래Marker 삭제gate의성공CREATE receipt/OBJECT_ID/CREATED/identity/run_id/행수/PURGE승인에따라본인table만정리한다.이요약은DROP대체recipe가아니다.본인backup삭제가능여부/retentionlock/expiry/protectedresourceowner를readback해87에반환하며잔존비용종료를꾸미지않는다.

실제 정리/원복 response, exact resource 또는rule ID,완료UTC와남은owner/expiry를비밀없는개인실행기록에남긴다.위의guarded복구가이미완료된경우중복mutation하지않는다.

## 결과검증

**기대 패턴(미실행, 선택 승인 후만):** marker before-backup→after-backup→restore 이후 before-backup. actual backup OCID/선택 recovery point/work request/SQL 결과를 함께 기록. 미승인 분기는 실행 안 함으로 명시한다.

실행 기록(미기입): 실행 UTC __ / 실제 resource OCID __ / 요청·SQL 실제 응답 __ / 공식 조건 차이 __ / 정리 완료 UTC __.

## 문제해결

backup failure는 destination 권한·service gateway/egress·FRA/full·work request 세부 오류를 확인한다. restore 후 after-backup이면 latest redo 적용/잘못된 recovery point 가능성을 조사한다. RMAN destructive commands나 OS disk 조작으로 우회하지 않는다.

## 리소스정리

marker는 아래 **생성 소유권·삭제 gate**를 통과한 본 run table만 제거한다 → 승인된 backup 삭제/retention 유지 결정 → [87 owner cleanup](00_Base_Database_Service.md#db-cleanup)에 반환. standalone backup/Recovery Service protected database/retention 리소스가 실제 남는지 각각 확인. 삭제 불가 retention이 있으면 잔존 및 종료일을 기록한다.

### Marker 삭제 gate와 실패 후 반환
**DROP는 irreversible이며 PURGE는 recycle bin 복구를 제거한다.[T-DROP]** 같은 client/endpoint/PDB/schema에 새로 접속해 session/identity/run_id gate를 다시 수행한다. 외부 생성 receipt의 **성공 CREATE**, actual database OCID/PDB/schema/service/OBJECT_ID/CREATED와 현재 다음 조회가 모두 일치해야 한다. current_schema 변경/다른 owner·object·token/receipt 없음/조회 실패이면 DROP를 실행하지 않는다. table 이름만 같다는 이유로 소유권을 인정하지 않는다.
```sql
select object_id,created,object_type from user_objects
where object_name='ROADMAP_RESTORE_MARKER' and object_type='TABLE';
select id,run_id,note from roadmap_restore_marker order by id;
```
정상 cleanup은 전체 정확히 한 행 id=1/receipt run_id/note=before-backup(복구 완료) 또는 after-backup(복구 실패·중단 기록)인 경우만 허용한다. 아래 assertion 성공, 위 object receipt 일치, 삭제 책임자의 **이 exact table PURGE 승인**을 모두 확인한 뒤 DROP를 단독 실행한다. 다른 계정의 DROP ANY TABLE 권한에 의존하지 않는다. 이 검사와 DROP 사이 다른 session이 DDL/DML을 하지 않도록 owner가 이 lab table의 exclusive 사용을 확인한다. 동시 사용 가능성이 있으면 삭제는 BLOCKED다.
```sql
declare total_rows number; owned_rows number;
begin
  select count(*), count(case when id=1 and run_id='&&run_id'
    and note in ('before-backup','after-backup') then 1 end)
    into total_rows,owned_rows from roadmap_restore_marker;
  if total_rows<>1 or owned_rows<>1 then
    raise_application_error(-20005,'STOP: cleanup ownership mismatch');
  end if;
end;
/
```
```sql
drop table roadmap_restore_marker purge;
```
`Table dropped.` 확인 후 exact object count를 읽고 **0**만 table cleanup 완료로 기록한다.
```sql
select count(*) as remaining_objects from user_objects
where object_name='ROADMAP_RESTORE_MARKER';
```
DROP 오류/응답 불명은 재DROP하지 않고 새 guarded session에서 exact object를 조회한다. 부재이면 actual 응답을 기록하고, 존재하면 receipt 대조·owner 재승인 후만 재시도한다. **CREATE 성공 뒤 INSERT/COMMIT 실패로 빈 table이 남은 경우** 위 정상 삭제 assertion은 의도적으로 막는다. CREATE receipt/OBJECT_ID/CREATED/identity가 일치하고 actual row count=0임을 owner/DBA가 별도 검토·삭제 승인한 때만 단독 DROP 및 부재 readback을 수행한다. CREATE 성공 증거가 없거나 충돌한 object는 삭제하지 않는다. ownership 불명이면 BLOCKED/잔존 owner를87에 보고하며 production/shared DB를 종료해 우회하지 않는다. sole lab DB 전체 종료는87의 독립 생성 소유권·consumer 반환·retention 승인 gate를 따른다.

### 소유자별 종료와 exact 대상 재조회
90 owner는 위 **Marker 삭제 gate와 실패 후 반환**을 통과한 exact LABPDB/schema/object만 DROP PURGE하고 user_objects exact count=0을 읽는다. 이 요약은 gate를 생략하는 별도 DROP recipe가 아니다. on-demand backup Delete가 허용되면 exact OCID 삭제·Backups 부재, lock/retention이면 actual expiry/owner 비용 승인을 남긴다. automatic backup 설정의 off/종료 retained 정책을 readback하고87에 반환한다. 잔존 보호 리소스의 완료 전 비용종료로 판정하지 않는다.

실제 정리 UTC __ / exact resource 또는 rule ID __ / 삭제 후 조회 응답 __ / 잔존 resource·만료·owner __. 승인 없는 잔존 비용은 다음 실습을 이유로 연장하지 않는다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 개념은 해당 생성·변경·조회 단계에서 입력 선택과 실제 결과 해석을 위해 사용하며, 공식 예시는 실제 실습 실행 결과가 아닙니다.

- [42] [Back Up and Recovery in Base Database Service — Backup / Restore](https://docs.oracle.com/en/cloud/paas/base-database/backup-recover/index.html)
  - 확인 범위: The Oracle-managed automatic backups.

### 실습 task 본문 근거

- [T1] [backup-db / index.html](https://docs.oracle.com/en/cloud/paas/base-database/backup-db/index.html) — 본문 절차·입력 필드·지원 조건을 읽어 실습에 적용.
- [T2] [backup-recover / index.html](https://docs.oracle.com/en/cloud/paas/base-database/backup-recover/index.html) — 본문 절차·입력 필드·지원 조건을 읽어 실습에 적용.
- [T-RESTORE] [Recover a database using the Console](https://docs.oracle.com/en/cloud/paas/base-database/recover-db/index.html) — automatic managed backup, timestamp/SCN restore fields and restore-failed handling.
- [T3] [On-demand backup/retention/status/delete task](https://docs.oracle.com/en/cloud/paas/base-database/backup-db/index.html) — 본문 절차·입력 필드·지원 조건을 읽어 실습에 적용.
- [T-P] [Policy Details for Base Database Service](https://docs.oracle.com/en/cloud/paas/base-database/iam-policy-details/index.html) — LaunchDBSystem/backup/restore API별 권한.
- [T-SQLERROR] [WHENEVER SQLERROR](https://docs.oracle.com/en/database/oracle/oracle-database/19/sqpug/WHENEVER-SQLERROR.html) — SQL/PLSQL 오류의 EXIT FAILURE ROLLBACK; SQL*Plus 명령 오류는 별도 수동 stop gate.
- [T-OSERROR] [WHENEVER OSERROR](https://docs.oracle.com/en/database/oracle/oracle-database/19/sqpug/WHENEVER-OSERROR.html) — OS 오류 exit; 기본 continue를 fail-closed로 변경.
- [T-COMMIT] [COMMIT](https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/COMMIT.html) — explicit WAIT IMMEDIATE와 DDL implicit commit; ROLLBACK으로 CREATE/PURGE가 원복되지 않음.
- [T-DROP] [DROP TABLE](https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/DROP-TABLE.html) — own schema 삭제 및 PURGE의 recycle bin 비복구성.
- [T-OBJECT] [USER_OBJECTS](https://docs.oracle.com/en/database/oracle/oracle-database/19/refrn/USER_OBJECTS.html) — 현재 user 소유 object 조회; 이름만으로 run 소유권을 증명하지 않음.
- [T-ROWS] [Implicit Cursor Attribute](https://docs.oracle.com/en/database/oracle/oracle-database/19/lnpls/implicit-cursor-attribute.html) — PL/SQL SQL%ROWCOUNT로 UPDATE의 exact affected rows 확인.
- [T-IDENTITY] [SYS_CONTEXT](https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/SYS_CONTEXT.html) — USERENV PDB/schema/service 조회와 manual exact identity gate.
- [T-DATE-FORMAT] [TO_CHAR datetime](https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/TO_CHAR-datetime.html) — DATE/TIMESTAMP를 지정 형식으로 기록; CREATED에 존재하지 않는 timezone을 부여하지 않음.

