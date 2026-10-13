# Base Database Service

> 문서 작성 검토와 실행 검증은 별개입니다. OCI 실행은 **USER_EXECUTION_PENDING**이며 사용자 캡처는 별도 자료입니다. 미실행 자체는 문서 작성 승인 차단 사유가 아닙니다. 실제 결과 필드는 실행자가 채웁니다.

> 실행용 한국어 교육 문서. OCI 리소스 생성·명령 실행 결과는 아직 검증하지 않았으며 모든 기대 패턴은 미실행이다.

## 학습목표

- 예산·만료·private network·SQL client를 준비하고88/89/선택91또는92를 검토해 single-node DB system1개만 승인 shape/edition/license/storage로 생성한다.
- AVAILABLE와 실제 DB/home/node OCID를 읽고 VCN client에서 PDB administration service의 TCP1521 SQL 연결·CON_NAME·marker를 검증한다.
- 같은 DB의 비존재 PDB service 연결 실패를 정상 service 재접속과 비교하고 모든 consumer 반환·backup 잔존 승인 후 sole DB terminate 및 새 rule/본인 client 파일만 정리한다.

실제 OCI 요청·응답/SQL·정리 증거는 실행자가 기록한다. 아래 기대값은 실측값이 아니다.

## 대응 Roadmap 항목

- 항목명: **Base Database Service**
- Roadmap 경로: **Base Database Service**
- Node ID: `sequence-10-0`
- 상위 항목: 없음 — 최상위 학습 항목
- 문서 범위: 이 항목의 개념, 실제 설정 절차와 학습자 실행 검증.
- [전체 항목 목록](../README.md)

## 사전준비

**시작 조건:** 88/89/선택91또는92 preparation review;private subnet/OSN443 및SQL client 준비

- [Virtual Cloud Network (VCN)](../03_Virtual_Cloud_Network_VCN/00_Virtual_Cloud_Network_VCN.md) — VCN/DNS 활성화와 private subnet 생성 완료. DB subnet은 192.168.16.16/28와 겹치지 않아야 한다.
- [Load Balancer](../08_Load_Balancer/00_Load_Balancer.md) — 기존 VCN 내부 client Compute/접속 경로 준비 단계만 재사용. DB용 LB 추가 생성은 하지 않는다. client private IP→DB NSG :1521, 관리 SSH :22는 승인된 client/bastion에서만 허용.

- [Private Subnets](../03_Virtual_Cloud_Network_VCN/03_Private_Subnets.md) — private subnet 생성·CIDR/DNS·route 설정 완료. 192.168.16.16/28 overlap 금지. DB 종료까지 유지.

- [Network Security Groups](../03_Virtual_Cloud_Network_VCN/06_Network_Security_Groups.md) — security rule 학습 **preparation-only**다.25의 nsg-client/nsg-web은 임시 리소스이며 DB-ready 출력이 아니다.22의 nsg-private-admin도 자동 DB NSG가 아니다. 아래 **db-network-ready**의 승인된 외부 existing retained NSG 계약을 Create 전에 완료한다.

- [Service Gateway](../03_Virtual_Cloud_Network_VCN/12_Service_Gateway.md) — **DB 생성 전** Oracle Services Network 접근 완료. private DB subnet route: destination **All <actual Region> Services in Oracle Services Network** → 실제 service gateway. DB NSG/security list stateful egress TCP443 → 동일 service CIDR label. provisioning/patching/YUM 접근 자체에 필요한 선행조건이며 backup 미선택이어도 생략하지 않는다. 공유 gateway는 DB 종료 후 삭제하지 않는다. [Base DB VCN/Subnets 실제 task](https://docs.oracle.com/en/cloud/paas/base-database/vcn-subnets/index.html)와 대조한다. DRG/VPN은 이미 승인된 on-premises client가 있을 때만 필요하며 이 단기 VCN client 트랙에서 새로 만들지 않는다.

- client install preparation-only: [29 Internet Gateway](../03_Virtual_Cloud_Network_VCN/10_Internet_Gateway.md#실습절차)의vm-a publicoutbound/승인egress완료. **privateclient선택일때만** [30 NAT Gateway](../03_Virtual_Cloud_Network_VCN/11_NAT_Gateway.md#nat-installation-preparation)의existingoutbound단계를완료한다. DB Create 제출전 `sqlplus -v`까지준비하고선택client경로를기록한다.

### 준비 완료 출력과 인수 확인

| 인수 항목 | 실제 출력·출처 | 다음 작업의 사용 위치 |
|---|---|---|
| 선행 완료 상태 | 88/89/선택91또는92 preparation review;private subnet/OSN443 및SQL client 준비 | 첫 변경 전 재조회 |
| 비밀 없는 실행 입력 | actual VCN/subnet/retained external NSG OCID/owner/expiry/rule IDs,client IP/SQLPlus -v,SSH key,승인 shape/license/storage/UTC | 아래 numbered Console/명령 단계 |
| 반환/후속 인계 | soleDB/node/home/databaseOCID·PDBservice/SQL을88~92에 인계 | 종료 UTC 이전 exact owner 반환 |

실제 Compartment/Region은 승인 Console에서 읽고 동일 identity/Region을 고정한다. DB Create 전에 기존 VCN SQL client의 `sqlplus -v`를 확인한다. 실제 PDB 서비스는 sole DB 생성 후 step8에서 인수·접속 검증한다. 생성 전에는 PDB 접속을 완료 조건으로 요구하지 않으며 Console과 SQL client 실행 위치를 구분한다. placeholder·명칭만 인수한 상태는 준비 완료가 아니다. 비밀번호·키·확인 token은 출력표에 적지 않는다.

## 권한

관리자가 전용 Compartment에만 아래 범위를 승인한다.
```text
Allow group <LabGroup> to manage database-family in compartment <LabCompartment>
Allow group <LabGroup> to use subnets in compartment <LabNetworkCompartment>
Allow group <LabGroup> to use vnics in compartment <LabNetworkCompartment>
Allow group <LabGroup> to use network-security-groups in compartment <LabNetworkCompartment>
```
DB system creation은 DB_SYSTEM_CREATE/DB_HOME_CREATE/DATABASE_CREATE와 VNIC/SUBNET attach 권한을 함께 요구한다.[T-P] 네트워크 rule 수정은 network 관리자 승인 권한이다. 90의 optional backup은 db-backups 관리·database 읽기, Restore는 backup 읽기·database use가 추가로 필요하다. OS `oracle` 접속과 DB 계정 권한은 OCI IAM과 별개다.

LaunchDbSystem=DB_SYSTEM_CREATE+DB_HOME_CREATE+DATABASE_CREATE+VNIC_CREATE+VNIC_ATTACH+SUBNET_ATTACH. TerminateDbSystem=DB_SYSTEM_DELETE+DB_HOME_DELETE+DATABASE_DELETE+VNIC_DETACH+VNIC_DELETE+SUBNET_DETACH; automatic backups enabled이면 DB_BACKUP_DELETE 추가. GetDbSystem=DB_SYSTEM_INSPECT; GetDbHome=DB_HOME_INSPECT; GetDatabase=DATABASE_INSPECT. IAM이 SQL SYSTEM 계정 권한을 대체하지 않는다.

operation 근거: [공식 권한/작업 본문](https://docs.oracle.com/en/cloud/paas/base-database/iam-policy-details/index.html). family grant는 minimum-permission 정책으로 주장하지 않는다. 실행 전에 관리자 승인된 identity/domain·실제 Compartment scope를 확인한다.

## 비용및주의사항

**유료 선택 트랙: 승인 없으면 Create 금지.** sole single-node DB1개를87에서만 생성한다. 현재 shape/edition/RU/License Included 또는 권리 확인 BYOL, DB compute 및 data/recovery/software total allocation, 선택 backup destination/retention·보호비용을 실제 견적으로 승인한다. ECPU/OCPU를 같은 단위로 계산하지 않는다. provisioning/SQL/consumer 반환/terminate까지 종료 UTC와 책임자를 정하며 Stop=비용종료 또는 DB 삭제=모든 backup 삭제라고 보장하지 않는다.90 PITR 및91 ASM은 별도 승인·지원 gate이며 새 DB로 비교하지 않는다.

## 실습구성

### 워크숍 순서와 입력값

**진행 순서:** 유료승인/88·89·91·92생성전선택 →network/client 준비 →DB1회 생성 →AVAILABLE →actualPDB SQL →optional90 →terminate/잔존확인

**선택 이유:** ASM/LVM은 동일 생성이벤트의 배타적 옵션이며 DB자원 수를 늘리지 않는다.

| 입력 | 값 또는 정확한 출처 | 실행 위치 | 생성 전/후 | 이유 |
|---|---|---|---|---|
| budget/expiry/license | 실제통화·상한·종료UTC /LicenseIncluded또는권리있는BYOL | 승인자 | 전 | 비용hardcap아님 |
| VCN/subnet/NSG/client | existingactualOCIDs /실제client IP | Console/networkowner | 전 | OSN443·client1521/SSH22 |
| shape/storage | VM.BaseDB.x86 4ECPU·LVM기본 /현재최소승인total | Console | 전 | ASM옵션배타/소프트웨어비용포함 |
| DB/PDB/service | LABDB /LABPDB /actualdomain포함PDBservice | Console/SQLPlus | 후 | CDB접속오인금지 |
| IC_HOME/KEY | 승인actualdirectory /existingapprovedSSHkey | client guest | 전 | 비밀번호prompt만 |


값이 다른 터미널에 자동 전파되지 않습니다. Console readback으로 비밀 없는 값만 인계하며, placeholder는 실제값으로 교체한 뒤 실행합니다. 새 리소스 생성 전 이름 충돌·소유자·최대 비용·종료 UTC를 확인하고 선택하지 않은 분기는 **NOT_SELECTED**로 기록합니다.


새 private single-node DB system 1개(기본 LVM), initial CDB 1개/PDB 1개. 기존 VCN/private subnet/client 재사용. 88·89·91·92 설정 비교는 Create 직전 함께 수행한다.

<a id="db-create-review"></a>

<a id="db-current-shape-support"></a>
### 현행 shape/storage 지원 근거와 선택 차단
[About DB Systems의 Flexible Shapes/Generation-Agnostic X86/retirement notice](https://docs.oracle.com/en/cloud/paas/base-database/about-dbs/index.html)는 **2026-07-31 effective VM.Standard.x86/EE-EP/OCPU-metered SKU retirement** 및 기존 배포 지속/신규 생성·renewal 영향, MOS **KB921246**를 명시한다. 이 public 본문으로 retirement를 확인하며 MOS 상세는 로그인 미확인이다. VM.BaseDB.x86는 최소4 ECPU,19c/26ai,SE/EE/EE-HP,**single-node/LVM** 지원이다. Ampere A1은 LVM-only이며SE 지원이 아니므로 기본SE/19c x86 조합의 대안으로 바꾸지 않는다. 실제 edition/version/RU/shape/storage 조합을87 Submit 직전 Console 제공값과 public create task로 재조회한다. ASM 가능한 최신 shape/capacity를 이 문서가 보장하지 않는다. OCPU SKU retirement 이후 신규 ASM offer가 명확히 승인·확인되지 않으면 **BLOCKED_ASM_OFFER_UNVERIFIED**이고 과거 AMD/Intel 표만으로 제출하지 않는다. LVM으로 조용히 대체하거나 두 번째DB를 만들지 않는다. IAM operation 근거는 각 문서 T-P이며 현재 Console/OCI 실행은 미검증이다.


### 이번 실습의 생성·재사용·앱 경계

| 구분 | 이번 주제의 실제 범위 |
|---|---|
| 새로 생성 | single-nodeDB system/CDB/PDB 각1개 |
| 재사용 | 기존private network/retained externalNSG/client/SSHkey |
| 앱/작업부하 및 변경 | 앱은SQLPlus client→PDB marker;새LB/웹앱/VCN 없음 |
| 실행/반환 순서 | 88/89/선택91또는92 preparation review;private subnet/OSN443 및SQL client 준비 → soleDB/node/home/databaseOCID·PDBservice/SQL을88~92에 인계 |

새로 생성하지 않는 항목을선행완료로가장하지않는다.위표의실제OCID/주소와진행순서를첫mutation전에확인한다.

## 실습절차

**현재 Console destination 제한:** official create/backup task는 **2026-06-23부터** **tenancy 생성일·대상 Region·DB version 조건**에서 automatic backups의 Console destination이 Recovery Service만 노출될 수 있다고 명시한다. Object Storage를 반드시 선택할 수 있다고 가정하지 않는다. 실제 UI에서 제공되는 값과 기존 승인 network/protected-resource 준비를 확인하고, 미제공 선택을 CLI 우회로 강제하지 않는다. 90 추가 foundation/보호비용 승인이 없으면90을 NOT_SELECTED로 남긴다. 87 automatic backups off core track은 조직 정책이 허용할 때만 계속한다.

### 생성 전 예산·network·SQL 완료표
승인 기록에는 currency/최대 총액/shape·edition·License Included 또는 BYOL 권리/storage total GB/backup off 또는 retention/최대 종료 UTC/삭제 책임자를 기입한다. 이 값이 없으면 유료 생성은 BLOCKED다. 비용 알림만으로 hard cap이 구현된다고 주장하지 않는다. DB-LAB-ONE1회 생성 전에 storage option91/92를 결정하고 새 system을 만들며 비교하지 않는다.
VCN→DB TCP1521 및 관리 SSH22는 ingress뿐 아니라 restricted client의 새 연결 egress와 OS firewall도 확인한다. SQL*Plus 설치는 [Oracle Instant Client SQL*Plus](https://www.oracle.com/database/technologies/instant-client/downloads.html)의 같은 OS/architecture/version Basic+SQL*Plus 패키지를 관리자 승인 경로에 준비하고 library 경로 및 `sqlplus -v`를 확인한다. artifact 다운로드·패키지 설치를 실행한 것으로 기록하지 않는다. DB-node 대안은 client network path와 동등하지 않음을 결과에 남긴다.
DB system/DB home/database OCID는 AVAILABLE에서 읽고 실제 LABPDB connect→CON_NAME/service/marker를 모두 검증한다. DB user password는 prompt에서만 입력한다. 연결 실패를 `/ as sysdba` 성공으로 덮지 않는다. terminate는 보호 retention을 확인한 후 irreversible 확인을 누르고 final state/잔존 full backups를 다시 조회한다.

### 실행 단계

1. 생성 전 담당자에게 예상 shape/라이선스/최소 storage/최대 유지시간·삭제 책임을 승인받는다. 88·89·91·92를 읽고 **DB-LAB-ONE** 생성 이벤트로 선택사항을 기록한다. 선택 변경은 Submit 전만 하며 비교용 두 번째 system을 만들지 않는다.
<a id="db-network-ready"></a>
1a. **Create 전 external retained NSG 인수 계약**: network owner가 이미 보유한 실제 NSG의 OCID/VCN OCID/Compartment/생성 owner/DB 종료까지 유지할 expiry/기존 VNIC membership을 서면 인계한다.25 임시 NSG를 존재한다고 가정하거나 본 문서에서 새 NSG를 만들지 않는다. 외부 계약이 없으면 **BLOCKED_DB_NSG_NOT_READY**로 Submit 금지다. network owner는 Networking → VCN → actual VCN → Network security groups → exact existing NSG에서 Rules 및 VNICs를 읽어 shared VM에 새 rule이 미치는 영향까지 승인한다. 본 lab 사용자가 NSG 삭제 owner가 되지 않는다.
승인된 NSG에만 stateful ingress TCP1521/source actual client /32/source port All; 선택 SSH22/source actual management client /32; stateful egress TCP443/destination type Service/All actual Region Services in Oracle Services Network를 추가하거나 exact 기존 rule ID를 재사용한다. 추가 rule과 기존 rule ID를 별도 기록하고 route의 동일 service label→actual SGW를 확인한다. source가 NSG인 기존 규칙이나 broad Security List 허용도 읽고 private scope를 검토한다. client의 기존 제한 egress는 DB IP 할당 후 owner가 TCP1521/선택22→actual DB /32를 준비하고, SQL 전에 OS firewall/egress readback을 완료한다.
Create Network의 NSG 필드에는 **이 계약의 exact NSG OCID**만 선택한다. 생성 후 DB system의 NSG 연결 및 DB node actual VNIC membership을 같은 exact NSG의 VNICs와 대조한다. 이름만 같거나 node membership 불명이면 SQL-ready가 아니다. DB 종료 후 actual DB VNIC detach 확인→이번 branch에서 **새로 추가한** rule ID만 owner가 제거/Rules 부재 readback하며 기존 rules/NSG/client membership은 유지한다. 외부 network owner에게 종료·반환 UTC를 전달한다.

### Concrete approved client ZIP installation before SQL verification
**DB Create 제출과 step8접속보다먼저**,승인clientguest에서officialBasic+SQL*PlusZIP2개를같은version/architecture로다운로드한실제경로를설정한다. 새전용빈디렉터리에만해제한다. ZIP또는설치경로가충돌하면중단한다.
```bash
set -euo pipefail
export BASIC_ZIP='실제 official Basic ZIP 절대경로'
export SQLPLUS_ZIP='실제 같은version architecture SQLPlus ZIP 절대경로'
export IC_STAGE='승인된 새 전용 설치디렉터리 절대경로'
test -f "$BASIC_ZIP" && test -f "$SQLPLUS_ZIP"
test ! -e "$IC_STAGE" || { printf 'install directory exists: stop\n'; exit 1; }
sudo dnf install -y libaio unzip
mkdir -m 0700 "$IC_STAGE"
unzip -n "$BASIC_ZIP" -d "$IC_STAGE"
unzip -n "$SQLPLUS_ZIP" -d "$IC_STAGE"
```
실제만들어진동일instantclientdirectory를다음IC_HOME에설정해libclntsh/sqlplus와`sqlplus -v`를확인한다.
```bash
export IC_HOME='실제 Basic와 SQLPlus를 함께 푼 instantclient directory의 절대경로'
test -x "$IC_HOME/sqlplus"
test -e "$IC_HOME/libclntsh.so"
export LD_LIBRARY_PATH="$IC_HOME${LD_LIBRARY_PATH:+:$LD_LIBRARY_PATH}"
export PATH="$IC_HOME:$PATH"
sqlplus -v
```
 vm-a public-admin client는[29 Internet Gateway](../03_Virtual_Cloud_Network_VCN/10_Internet_Gateway.md#실습절차)의existingIGW/route와승인TCP443(필요시80)egress로officialdownload/repository에접근한다. **private client 대안인경우만** [30 NAT 설치준비](../03_Virtual_Cloud_Network_VCN/11_NAT_Gateway.md#nat-installation-preparation)의existingNAToutbound단계가필요하다. 새gateway를이문서에서추가하지않는다. DBnode의OSN443경로와client공식download경로는동일하지않다. 기존sqlplus가있으면설치하지않으며cleanup은본인새IC_STAGE만owner승인후제거한다.

2. Oracle AI Database → Oracle Base Database Service → DB systems → Create DB system. Compartment **실습 Compartment**, Name **roadmap-db**, AD **private subnet과 호환되는 실제 AD**, Node count **1**. Change shape에서 기본 **4 OCPU**를 그대로 쓰지 않는다. **LVM 기본 트랙**은 현재 제공 **VM.BaseDB.x86 4 ECPUs**가 지원되면 선택하고, 없으면 관리자가 승인한 현재 지원 최소 single-node shape를 선택한다. 구 VM.Standard.x86의 신규 생성 지원을 가정하지 않는다. 신규 OCPU-metered shape/SKU 지원도 현재 공식 retirement 공지와 actual Console로 확인한다. 과거 AMD/Intel shape를 ASM 대안으로 임의 제출하지 않는다. **91 ASM 실행 트랙을 승인한 경우** generation-agnostic VM.BaseDB.x86/AmpereA1을 선택하지 않고 현재 Console에서 ASM single-node를 지원하는 승인 shape/version을 선택한다. 없으면 ASM branch는 BLOCKED이며 LVM으로 몰래 대체하지 않는다.[T1]
3. Edition **Standard Edition**(현재 shape/version에서 허용될 때), License **License Included**를 기본 학습값으로 삼되 비용을 승인받는다. 기존 보유 라이선스 없이 BYOL을 선택해 비용을 줄이지 않는다. Database version **지원되는 19c 최신 RU**(Console 제공 선택값)를 선택한다. shape/version/edition 조합이 없으면 임의 대체 제출하지 않고 승인 변경을 받는다.
4. Change storage → Storage management **Logical Volume Manager**(기본 LVM) 또는 **Oracle Grid Infrastructure**(91의 비용승인 ASM shape일 때만), Performance **Balanced**, Available data storage **256 GB**(현재 최소값이 다르면 현재 최소값 승인), Recovery area는 자동 표기된 값, Total storage를 기록한다. 256 GB만 과금된다고 생각하지 않는다. software reserve 및 recovery를 포함한 total을 확인한다.[T2]
5. Network → 기존 VCN, **기존 private-app/backend subnet의 actual OCID**(vm-a public-admin client subnet이아님), 승인된 기존 DB NSG. SSH public key **승인된 실습 키의 .pub**를 업로드한다. Hostname prefix **roadmapdb**, private IP **automatic**. 관리 접속과 DB :1521을 client로만 제한한다. DB에 public IP/0.0.0.0/0 listener rule을 추가하지 않는다.
6. Database name **LABDB**, PDB name **LABPDB**, administrator password는 승인된 password manager로 생성하여 Console에 직접 입력한다. Database backup **automatic backup off**를 기본 단기 실습값으로 선택한다. 강제 정책으로 켜야 하는 계정은 destination/retention 추가 비용을 재승인한다. TDE key는 **Oracle-managed key** 기본값, 고객 Vault를 새로 만들지 않는다. Create review에서 node=1, shape/licensing/storage/network/version를 읽고 **단 한 번** Create한다.
7. 실제 DB system OCID, DB home OCID, database OCID, PDB 서비스명과 lifecycle **AVAILABLE**을 기록한다. Work request가 실패하면 세부 error를 확인하며 재생성을 반복하지 않는다. 생성된 partial resource가 있는지 먼저 확인한다.
<a id="db-sql-verification"></a>


8. **SQL*Plus가 client에없으면먼저 위 Linux ZIP 설치 task·Basic+SQL*Plus 준비·IC_HOME 세션설정·`sqlplus -v`를 완료한뒤 연결명령을 실행한다.** Database details → Connections에서 실제 **CDB administration Easy Connect string**을 복사한다. 공식 지침대로 service 부분의 DB_unique_name을 **LABPDB**로 바꾸고 DB_domain을 유지해 **PDB administration service**를 만든다. UI가 PDB-specific string을 직접 제공하면 그것을 사용한다. 기존 default administration CDB service를 PDB 서비스로 오인하지 않는다. 기존 VCN 내부 client에서 다음 SQL을 실행한다.[T-CONNECT][T-SQL]
```bash
# VCN 내부 기존 client에서 실행. 비밀번호는 SQL*Plus prompt에 직접 입력한다.
export DB_PRIVATE_IP='실제 DB node private IP'
export PDB_SERVICE='Database connection string 화면의 실제 PDB service name'
sqlplus -L /nolog
```
```sql
connect system@//실제_DB_private_IP:1521/실제_PDB_service_name
-- SQL*Plus의 password prompt에서만 입력. shell history에 비밀번호를 쓰지 않는다.
show con_name
select sys_context('USERENV','CON_NAME') as pdb,
       sys_context('USERENV','SERVICE_NAME') as service,
       systimestamp as checked_at from dual;
select 'roadmap-db-connected' as marker from dual;
exit
```
`SYSTEM`은 승인된 단기 lab에서 연결 검증용으로만 사용한다. 운영 DB의 application 계정 사용 원칙을 대체하지 않는다. client에는 승인된 Oracle Instant Client SQL*Plus를 OS/architecture에 맞게 설치하고 `sqlplus -v`로 확인한다. 기존 client 설치 분기를 선택한 경우 [실제 Linux ZIP 설치 task](https://docs.oracle.com/en/database/oracle/oracle-database/19/lacli/install-instant-client-using-zip.html)에 따라 같은 version/architecture의 **Basic + SQL*Plus ZIP**을 공식 download page에서 준비한다. 승인 전용 디렉터리에 두 ZIP을 풀고 OS libaio를 설치한다(Oracle Linux: `sudo dnf install -y libaio unzip`; 외부 repo/package 비용·network 승인 선행). ZIP이 만드는 **동일 actual instantclient directory**의 `IC_HOME`/PATH/LD_LIBRARY_PATH 설정은위 Concrete approved client ZIP installation 코드에서완료한다. 중복설치/임의versionpath를사용하지않는다.

unzip/install은 실제 client에서 수행해야 하며 이 문서 작성 중 실행한 결과가 아니다. 기존 client에서 다른 Oracle 설치가 있으면 ldconfig/global profile을 바꾸지 않고 세션 환경만 적용한다. CLI GET/AVAILABLE만으로 SQL 연결 성공을 기록하지 않는다.
8a. **별도 client 설치 비용을 줄이는 실행 대안:** 기존 client에 SQL*Plus가 없으면 이미 DB node에 포함된 SQL*Plus를 network client로 사용할 수 있다. 이 경우 DB-node 내부 listener 검증이지 client→DB TCP 경로 검증과 동일하다고 주장하지 않는다. 기존 VCN client에서 SSH 경로와 DB NSG :22 허용을 먼저 확인한다. 승인된 lab OS 계정으로 아래를 수행한다.[T-LOCAL]
```bash
ssh -i "$HOME/.ssh/승인된실습키" opc@실제_DB_private_IP
```
SSH 접속된 **DB node의 opc prompt**인지 hostname을 직접 확인한 뒤, 아래 **한 줄만** 실행한다. SSH가 종료된 로컬 shell에서 실행하지 않는다.
```bash
sudo su - oracle
```
전환된 **DB node의 oracle prompt**에서 다음을 실행한다. `sudo su -`가 종료된 뒤 local shell에 이어 실행하는 하나의 script로 묶지 않는다.
```bash
pgrep -af ora_pmon
# 위 실제 ora_pmon_<SID>의 SID를 oraenv prompt에 입력한다.
. oraenv
sqlplus -v
sqlplus -L /nolog
```
SQL prompt에서 위 `connect system@//실제_DB_private_IP:1521/실제_PDB_service_name`과 동일한 SQL을 실행한다. `/ as sysdba` bequeath 연결로 대체하지 않으므로 listener/PDB 서비스 검증은 유지한다. 외부 VCN client→DB :1521 검증을 요구하는 경우 client SQL*Plus 설치/기존 SQL Developer 준비를 별도 승인하고 실제 그 client에서도 같은 SQL을 실행해야 한다.

9. 실제 PDB/service/marker 결과를 기록한 뒤 같은 system을 88~92에서 재사용한다. 90 optional restore를 하지 않는 경우 모든 관찰 직후 삭제한다.


### 진행·종료 판정
설정 readback·기능시험·원복·정리를 각각 실제 응답으로 기록한다. 기대값은 미실행이며 실패/만료 시 새 mutation을 중단하고 아래 topic별 종료 단계로 반환한다.

### 동일 DB의 service-name negative control와 재접속
87의 정상 LABPDB 연결을 확보한 **같은 VCN client**에서 SQL*Plus `/nolog`로 시작한다. 비밀번호는 prompt에서만 입력하며 계정 잠금 위험이 있는 wrong-password 시험은 하지 않는다. 담당 DBA가 실제 등록되지 않았음을 확인한 service `ROADMAP_NOT_CREATED_87.<actual DB_domain>`를 같은 DB_PRIVATE_IP:1521에 사용한다.
```sql
connect system@//실제_DB_private_IP:1521/ROADMAP_NOT_CREATED_87.실제_DB_domain
```
실제 ORA-12514 또는 service resolution 실패를 기록한다. timeout이면 service-name denial이 아니라 network failure다. nonexistent service로 예기치 않은 접속이 되면 이 fixture는 무효이며 CON_NAME을 확인하고 중단한다. 같은 endpoint/계정으로 **실제 LABPDB service**에 재접속한다.
```sql
connect system@//실제_DB_private_IP:1521/실제_PDB_service_name
show con_name
select 'roadmap-db-connected' as marker from dual;
exit
```
정상→service 오류→동일정상 service의 LABPDB/marker 복귀가 이 control 완료다. DB listener stop·NSG broad deny·새 DB 생성은 하지 않는다.


### 번호 순서로 완료하는 원복·정리

10. **Base Database Service 종료 및 실패 반환.** 88/89/선택90/91또는92 실제반환과backup잔존/만료승인을받고soleDB exactOCID Terminate/Delete를owner87이한번수행한다.workrequest/TERMINATED 또는부재 및standalone/protectedbackup잔존을읽는다.DBVNIC detach뒤networkowner가이번새rule IDs만제거하고sharedNSG/key/VCN/client/gateway는보존한다.본인새IC_STAGE만승인삭제한다.

실제 정리/원복 response, exact resource 또는rule ID,완료UTC와남은owner/expiry를비밀없는개인실행기록에남긴다.위의guarded복구가이미완료된경우중복mutation하지않는다.

## 결과검증

**기대 패턴(미실행):** AVAILABLE + SQL*Plus connected + CON_NAME=LABPDB + marker roadmap-db-connected. 서비스 이름은 LABPDB 문자열 자체와 다를 수 있어 실제 connection string을 사용한다. 비용 승인/실제 OCID/SQL 검증을 별도 evidence에 기록한다.

실행 기록(미기입): 실행 UTC __ / 실제 resource OCID __ / 요청·SQL 실제 응답 __ / 공식 조건 차이 __ / 정리 완료 UTC __.

## 문제해결

ORA-12541은 listener/port :1521/NSG/route, ORA-12514는 실제 PDB service name와 PDB open state, ORA-01017은 credential/계정 scope를 확인한다. provisioning 실패 시 work request error와 AD capacity/quota/hostname 중복 확인. root로 디스크를 포맷해 복구하지 않는다.

<a id="db-cleanup"></a>

## 리소스정리

90 backup 분기 결과와 retention 계획을 확인한 뒤 DB system details → Delete/Terminate. final backup는 추가 비용 승인된 경우만 선택한다. automatic backups/standalone full backups/Recovery Service 보호 리소스 잔존 여부와 현재 선택한 Retain per policy 또는72hours deletion option을 각각 확인한다. 최신 backup task와 terminate task의 deletion 설명에 차이가 있으므로 당일 모든 backup 비용 종료를 보장하지 않는다. DB 삭제 후 이번 DB branch에서 추가한 exact NSG rule ID만 정리한다. **기존 공유 SSH key는 삭제하지 않는다.** 이번branch전용으로새키를만든경우만모든consumer반환/승인후그파일만정리한다. 공유 VCN/client/subnet은 삭제하지 않는다. Stop만 하면 storage 과금이 남을 수 있다.

### 소유자별 종료와 exact 대상 재조회
87 owner는88/89/선택90/91또는92 반환 후 exact DB system OCID의 Delete/Terminate 확인을 수행한다. Work request 종료 및 DB Systems에서 TERMINATED/부재를 읽고 자동/standalone backups와 Recovery Service protected resource 각각의 잔존 OCID·만료·삭제 책임을 확인한다. DB VNIC detach/NSG membership 부재 확인 뒤 이번 branch rule ID만 제거·Rules 부재를 읽는다. 새 IC_STAGE만 승인 후 제거하고 shared key/client/VCN/gateway는 보존한다.

실제 정리 UTC __ / exact resource 또는 rule ID __ / 삭제 후 조회 응답 __ / 잔존 resource·만료·owner __. 승인 없는 잔존 비용은 다음 실습을 이유로 연장하지 않는다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 개념은 해당 생성·변경·조회 단계에서 입력 선택과 실제 결과 해석을 위해 사용하며, 공식 예시는 실제 실습 실행 결과가 아닙니다.

- [43] [Oracle Base Database Service - Get Started — Base Database Service](https://docs.oracle.com/en/cloud/paas/base-database/index.html)
  - 확인 범위: Oracle Base Database Service enables / You can manage these DB systems.

### 실습 task 본문 근거

- [T1] [create-dbs-new / index.html](https://docs.oracle.com/en/cloud/paas/base-database/create-dbs-new/index.html) — 본문 절차·입력 필드·지원 조건을 읽어 실습에 적용.
- [T2] [about-dbs / index.html](https://docs.oracle.com/en/cloud/paas/base-database/about-dbs/index.html) — 본문 절차·입력 필드·지원 조건을 읽어 실습에 적용.
- [T3] [iam-policy-details / index.html](https://docs.oracle.com/en/cloud/paas/base-database/iam-policy-details/index.html) — 본문 절차·입력 필드·지원 조건을 읽어 실습에 적용.
- [T-P] [Policy Details for Base Database Service](https://docs.oracle.com/en/cloud/paas/base-database/iam-policy-details/index.html) — LaunchDBSystem/backup/restore API별 권한.


- [T-IC-INSTALL] [Linux Instant Client ZIP installation](https://docs.oracle.com/en/database/oracle/oracle-database/19/lacli/install-instant-client-using-zip.html) — 동일 version Basic/SQLPlus ZIP 해제 및 library 조건.
- [T-TERMINATE] [Terminate DB System](https://docs.oracle.com/en/cloud/paas/base-database/terminate-dbs/index.html) — irreversible termination과backup잔존.
- [T-BACKUP] [Backup deletion/retention tasks](https://docs.oracle.com/en/cloud/paas/base-database/backup-db/index.html) — automatic와standalone삭제구분.
- [T-CONNECT] [PDB administration service 연결 문자열 도출](https://docs.oracle.com/en/cloud/paas/base-database/connect/index.html) — PDB administration service 연결 문자열 도출 본문을 실제 읽어 적용.

- [T-SQL] [private VCN SQLNet :1521 / PDB open 조건](https://docs.oracle.com/en/cloud/paas/base-database/connect-sqlnet/index.html) — private VCN SQLNet :1521 / PDB open 조건 본문을 실제 읽어 적용.

- [T-LOCAL] [opc SSH / oracle environment / SQLPlus client](https://docs.oracle.com/en/cloud/paas/base-database/connect-bequeath/index.html) — opc SSH / oracle environment / SQLPlus client 본문을 실제 읽어 적용.
