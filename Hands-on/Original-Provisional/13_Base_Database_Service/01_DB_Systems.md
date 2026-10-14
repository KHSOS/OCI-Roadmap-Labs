> **기존 실습 초안 — 미검증**: 원래 93개 문서의 절차를 보존한 자료입니다. 새 튜토리얼의 품질 승인이나 OCI 실행 검증을 뜻하지 않습니다.

# DB Systems

> 문서 작성 검토와 실행 검증은 별개입니다. OCI 실행은 **USER_EXECUTION_PENDING**이며 사용자 캡처는 별도 자료입니다. 미실행 자체는 문서 작성 승인 차단 사유가 아닙니다. 실제 결과 필드는 실행자가 채웁니다.

> 실행용 한국어 교육 문서. OCI 리소스 생성·명령 실행 결과는 아직 검증하지 않았으며 모든 기대 패턴은 미실행이다.

## 학습목표

- 87 제출 전 node1/shape/edition/license/network를 선택하고 동일 sole DB의 actual node/home/database/PDB hierarchy를 Console·CLI로 조회한다.
- node list1개와87 actual OCID를 대조하고 동일 PDB SQL marker로 metadata와 기능상태를 분리한다.
- 실제 조회 결과의 wrong display-name filter와 정상 filter 복구를 비교한 뒤 consumer 반환을87에 기록한다. RAC/새 DB 생성·resize는 하지 않는다.

실제 OCI 요청·응답/SQL·정리 증거는 실행자가 기록한다. 아래 기대값은 실측값이 아니다.

## 대응 Roadmap 항목

- 항목명: **DB Systems**
- Roadmap 경로: **Base Database Service → DB Systems**
- Node ID: `sequence-10-1`
- 상위 항목: **Base Database Service** (`sequence-10-0`)
- 문서 범위: 이 항목의 개념, 실제 설정 절차와 학습자 실행 검증.
- [전체 항목 목록](../../../Docs/README.md)

## 사전준비

**시작 조건:** 생성 전87 review는preparation-only;생성 후87 AVAILABLE/client SQL 완료

- [Base Database Service](00_Base_Database_Service.md#db-sql-verification) — 1~6단계에서 shape/node/network/license를 결정하여 DB-LAB-ONE으로 단 1회 생성. 7~8단계 AVAILABLE와 SQL 연결까지 완료 후 본 검증 진행.

### 87 network-ready 소유권 인수
[87 db-network-ready](00_Base_Database_Service.md#db-network-ready)의 actual retained external NSG OCID/owner/expiry/rule IDs/DB VNIC membership readback을 인수한다.25 임시 NSG는 prerequisite 출력이 아니다. 계약 미완료이면 DB 생성 및 본 branch는 BLOCKED_DB_NSG_NOT_READY다. 새 NSG/DB를 만들지 않으며 종료는87 creator→network owner의 exact added-rule 반환 순서다.

### 준비 완료 출력과 인수 확인

| 인수 항목 | 실제 출력·출처 | 다음 작업의 사용 위치 |
|---|---|---|
| 선행 완료 상태 | 생성 전87 review는preparation-only;생성 후87 AVAILABLE/client SQL 완료 | 첫 변경 전 재조회 |
| 비밀 없는 실행 입력 | same actual DB_SYSTEM_OCID/node/home/database/PDB service,retained NSG 계약,87 budget/expiry | 아래 numbered Console/명령 단계 |
| 반환/후속 인계 | single-node hierarchy 및SQL marker를87에 read-only 반환 | 종료 UTC 이전 exact owner 반환 |

실제 Compartment/Region은 승인 Console에서 읽고 동일 identity/Region을 고정한다. 87의기존VCN SQL client에서 `sqlplus -v` 및실제PDB서비스접속상태를확인한다.Console조회와SQL session의실행위치를구분한다. placeholder·명칭만 인수한 상태는 준비 완료가 아니다. 비밀번호·키·확인 token은 출력표에 적지 않는다.

## 권한

87 creator의Launch/Terminate 권한과이consumer의읽기권한을분리한다.본topic의scope는같은실제DB Compartment다.

GetDbSystem=DB_SYSTEM_INSPECT, GetDbNode=DB_NODE_INSPECT, ListDbHomes/GetDbHome=DB_HOME_INSPECT, GetDatabase=DATABASE_INSPECT. 이 checkpoint는 inspect/read이며 node power/resize 권한을 소비하지 않는다. manage database-family는87 공동 생성/종료용 broad scoped grant이지88 minimum이 아니다.

operation 근거: [공식 권한/작업 본문](https://docs.oracle.com/en/cloud/paas/base-database/iam-policy-details/index.html). family grant는 minimum-permission 정책으로 주장하지 않는다. 실행 전에 관리자 승인된 identity/domain·실제 Compartment scope를 확인한다.


Console/CLI hierarchy의 ListDbSystems=DB_SYSTEM_INSPECT, ListDbHomes=DB_HOME_INSPECT, ListDatabases=DATABASE_INSPECT, PDB selector는PLUGGABLE_DATABASE_INSPECT도승인한다. DB node list task에필요한DB_NODE_INSPECT 범위와actual계정의목록조회가능여부를관리자가확인한다.

## 비용및주의사항

87의 유료 sole DB를 조회하는 checkpoint이며 node/home/PDB별 추가 DB를 만들지 않는다. node count1/승인 shape·단위·edition/licensing을87 생성 review와 대조한다. RAC·resize·node reboot는 미선택이며 metadata 조회가 끝나도 기존DB 유지비는87 종료까지 남는다. 실제 hierarchy/SQL 검증 종료 UTC를 owner87에 반환하고 consumer 때문에 비용 lease를 무승인 연장하지 않는다.

## 실습구성

### 워크숍 순서와 입력값

**진행 순서:** 87 생성전 node/shape선택 →87 sole생성/SQL →system/node/home/database/PDB 동일성 →87 반환

**선택 이유:** RAC는 추가생성 없이 제약 비교만 하며 node AVAILABLE와 SQL접속을 구분한다.

| 입력 | 값 또는 정확한 출처 | 실행 위치 | 생성 전/후 | 이유 |
|---|---|---|---|---|
| DB_SYSTEM_OCID | 87soleactualOCID | Cloud Shell | 후 | samehierarchy |
| node/home/database/PDB | actualOCIDs /LABPDB actualSQL | Console/SQLPlus | 후 | 다른system오인금지 |


값이 다른 터미널에 자동 전파되지 않습니다. Console readback으로 비밀 없는 값만 인계하며, placeholder는 실제값으로 교체한 뒤 실행합니다. 새 리소스 생성 전 이름 충돌·소유자·최대 비용·종료 UTC를 확인하고 선택하지 않은 분기는 **NOT_SELECTED**로 기록합니다.


87 Create 단계에 종속된 DB Systems checkpoint. 별도 DB 생성 이벤트 없음.

### 생성 전 현행 지원 확인
[87의 public retirement/shape 지원 근거](00_Base_Database_Service.md#db-current-shape-support)를 먼저 읽는다. VM.BaseDB.x86 single-node/LVM 기본의 actual edition/version/storage 조합을 재조회하고 다른 shape로 자동 대체하지 않는다. [Oracle About DB Systems](https://docs.oracle.com/en/cloud/paas/base-database/about-dbs/index.html)의2026-07-31 retirement/KB921246를 선택 시점에 확인하며 새 DB를 추가하지 않는다.


### 이번 실습의 생성·재사용·앱 경계

| 구분 | 이번 주제의 실제 범위 |
|---|---|
| 새로 생성 | 독립리소스없음 |
| 재사용 | 87 soleDB/node/home/PDB·SQLclient |
| 앱/작업부하 및 변경 | 생성전node/shape review+생성후hierarchy/SQL 읽기 |
| 실행/반환 순서 | 생성 전87 review는preparation-only;생성 후87 AVAILABLE/client SQL 완료 → single-node hierarchy 및SQL marker를87에 read-only 반환 |

새로 생성하지 않는 항목을선행완료로가장하지않는다.위표의실제OCID/주소와진행순서를첫mutation전에확인한다.

## 실습절차

### resource hierarchy를 실물로 검증
87 Create review가 preparation-only이고 생성/AVAILABLE/SQL 이후가 resource dependency다. 88이 별도 system을 생성하지 않는다. shape units ECPU/OCPU를 Console 그대로 쓰며 4ECPU=4OCPU라고 변환하지 않는다. node/home 목록에서 동일 system OCID를 확인하고 실제 database/PDB 연결을 재사용한다.
잘못된 DB_SYSTEM_OCID를 조회하는 destructive failure test는 없다. empty 목록이면 Compartment/system filter를 실제 record로 복구 후 목록 재조회한다. node AVAILABLE와 SQL marker가 다르면 network/service/PDB 문제로 기록하고 resource list만으로 완료 처리하지 않는다. cleanup은87 owner가 한 번 실행한다.

[87 생성 전 선택 review](00_Base_Database_Service.md#db-create-review)는 preparation-only이고 [87 SQL 완료](00_Base_Database_Service.md#db-sql-verification)는 생성 후 resource dependency다. 최종 삭제는 [87 owner cleanup](00_Base_Database_Service.md#db-cleanup)을 따른다.

### 실행 단계

1. 87의 Create review에서 **node count=1** 선택 이유를 적는다: RAC 비용·추가 노드·추가 조건 없이 client SQL 기능을 검증한다. AD/shape/edition/license/network/hostname 선택값을 표로 기록한다. RAC=2 nodes는 관찰 비교만 하며 생성하지 않는다.
2. 생성한 동일 system의 DB systems detail에서 node count, shape ECPU/OCPU, storage management, lifecycle, Database Home, database를 읽는다. DB System → DB Home → Database → PDB hierarchy의 actual OCID를 기록한다.[T1]
3. 실제 resource 조회:
```bash
export DB_SYSTEM_OCID='87에서 생성한 실제 DB system OCID'
oci db system get --db-system-id "$DB_SYSTEM_OCID"
oci db node list --compartment-id "$COMPARTMENT_OCID" --db-system-id "$DB_SYSTEM_OCID" --all
oci db db-home list --compartment-id "$COMPARTMENT_OCID" --db-system-id "$DB_SYSTEM_OCID" --all
```
4. 87 SQL*Plus connection으로 `show con_name`과 dual marker를 다시 확인한다. node AVAILABLE가 DB listener/PDB 연결까지 보장하지 않으므로 SQL을 직접 확인한다.
5. 현재 official body는 VM.BaseDB.x86 generation-agnostic shape는 LVM·single node만 지원하고 fault domain 선택을 지원하지 않는다고 설명한다. 현재 official 공지는 Standard x86/EE-EP/OCPU-metered SKU에 영향을 주는 retirement를 명시하므로 해당 공지를 읽고 과거 shape 비교표를 현재 신규 생성 선택값으로 혼동하지 않는다.[T2]
6. 실제 shape를 resize하거나 2-node system으로 재생성하지 않는다. 생성 시 edition은 나중에 자유롭게 바뀌는 옵션이 아니라는 제약을 확인한다.


### 진행·종료 판정
설정 readback·기능시험·원복·정리를 각각 실제 응답으로 기록한다. 기대값은 미실행이며 실패/만료 시 새 mutation을 중단하고 아래 topic별 종료 단계로 반환한다.

### 읽기 전용 negative control와 정상 query 복귀
Console DB Systems 목록에서 실제 **roadmap-db** 이름/OCID를 먼저 찾은 뒤 search display name을 `roadmap-db-not-created-in-this-lab`으로 바꿔 no match를 읽는다. 실제 목록에 같은 이름이 있으면 다른 미사용 문자열을 확인한다. search를 비워 actual OCID를 다시 열고 아래 node/home 조회와 PDB marker를 재확인한다. 이 filter 비교는 permission-deny·없는 OCID API 시험이 아니며 node reboot/다른 system 조회·생성을 하지 않는다.




### 번호 순서로 완료하는 원복·정리

7. **DB Systems 종료 및 실패 반환.** same system/node/home/PDB positive복귀와SQLmarker를읽고변경없음/session종료/반환UTC를87owner에게전달한다.본checkpoint는node/home삭제·reboot가없으며soleDB termination은87의consumer/backupgate 뒤한번이다.

실제 정리/원복 response, exact resource 또는rule ID,완료UTC와남은owner/expiry를비밀없는개인실행기록에남긴다.위의guarded복구가이미완료된경우중복mutation하지않는다.

## 결과검증

**기대 패턴(미실행):** node list 길이=1, same DB system OCID, same PDB/service 실제 SQL 연결. 선택되지 않은 RAC를 실행 결과로 표시하지 않는다.

실행 기록(미기입): 실행 UTC __ / 실제 resource OCID __ / 요청·SQL 실제 응답 __ / 공식 조건 차이 __ / 정리 완료 UTC __.

## 문제해결

list가 비면 node/home Compartment scope와 db-system-id를 확인한다. 생성은 됐지만 client timeout이면 관리 접속 경로와 DB :1521 NSG를 구분한다. shape별 FD 필드 없는 것을 오류로 판단하지 않는다.

## 리소스정리

독립 DB 삭제를 하지 않는다. 87이 공동 리소스 owner이며 89~92 완료 직후 87 삭제 단계를 따른다. 본 checkpoint에서 shape 변경·node reboot를 하지 않는다.

### 소유자별 종료와 exact 대상 재조회
88 owner는 actual system/node/home/database/PDB 관계와 read-only 변경 없음/반환 UTC를87에 전달한다. 별도 node/home 삭제나 reboot는 수행하지 않는다. final DB lifecycle/잔존비용 readback은87 owner에 귀속된다.

실제 정리 UTC __ / exact resource 또는 rule ID __ / 삭제 후 조회 응답 __ / 잔존 resource·만료·owner __. 승인 없는 잔존 비용은 다음 실습을 이유로 연장하지 않는다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 개념은 해당 생성·변경·조회 단계에서 입력 선택과 실제 결과 해석을 위해 사용하며, 공식 예시는 실제 실습 실행 결과가 아닙니다.

- [41] [About DB Systems — DB Systems](https://docs.oracle.com/en/cloud/paas/base-database/about-dbs/index.html)
  - 확인 범위: Oracle Cloud Infrastructure (OCI) offers.

### 실습 task 본문 근거

- [T1] [about-dbs / index.html](https://docs.oracle.com/en/cloud/paas/base-database/about-dbs/index.html) — 본문 절차·입력 필드·지원 조건을 읽어 실습에 적용.
- [T2] [create-dbs-new / index.html](https://docs.oracle.com/en/cloud/paas/base-database/create-dbs-new/index.html) — 본문 절차·입력 필드·지원 조건을 읽어 실습에 적용.
- [T-P] [Policy Details for Base Database Service](https://docs.oracle.com/en/cloud/paas/base-database/iam-policy-details/index.html) — LaunchDBSystem/backup/restore API별 권한.

