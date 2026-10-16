> **문서 검토 완료 — OCI 실행 미검증**: 이 문서의 기존 두 VM 조회·증거 평가·ledger 작성·소비자 반환 시나리오는 한정된 문서 검토 범위에서 승인되었습니다. 실제 OCI 실행 검증이나 전체 교육과정 승인을 뜻하지 않습니다.

# Well-Architected Framework

> 문서 승인 범위: **한정된 실행 절차 문서 검토 완료 / 실제 OCI 실행 미검증**. 사전준비·권한·예산·보존기한·서비스 지원·보안 증거·복구·소비자 반환 gate는 실행 시 충족해야 합니다. 모든 예상 출력은 패턴이며 실측값이 아닙니다. 전체 교육과정·게시 상태는 별도이며 [통합 상태](../../../Docs/README.md)를 확인하세요.

> 실행 절차 작성본 / OCI 미실행. Oracle 공식 본문 근거와 교육용 선택을 구분한다. secrets·실제 고객값·diagram 없음.

## 학습목표

- owner07의 같은 vm-b 배치와 owner08의 제한접근 설정을 OCID 기반 ledger에 연결하고 shape/boot/expiry를 API와 대조한다.
- 실제 구현한 placement·보안·운영·비용 조치만 다섯 pillar 표에 기록하고 미실행 failover/Distributed Cloud는 제외한 뒤 소비자를 반환한다.

위 항목은 사용자 실행 후 확인할 완료 조건이며 현재 관측 결과가 아니다. 개념은 아래 기존 배치 조회·증거 검토 단계에서 함께 적용한다.

## 대응 Roadmap 항목

- 항목명: **Well-Architected Framework**
- Roadmap 경로: **Introduction to OCI → Well-Architected Framework**
- Node ID: `sequence-1-5`
- 상위 항목: **Introduction to OCI** (`sequence-1-0`)
- 문서 범위: 기존 자원의 조회·평가·로컬 증거 정리. 실습 미실행; 아래 기대 패턴은 실제 출력 아님.
- 번호는 navigation 식별자이며 실행 순서는 사전준비의 단계별 의존성에 따른다.

## 사전준비

### 기준 계약과 실제 단계 선행조건

[공통 baseline-contract.json](../../../baseline-contract.json)의 Oracle Linux 9 / opc / CIDR / 비용 상한을 따른다. 문서 번호는 실행 순서가 아니다. **준비 station과 생성 이후 ready를 분리**한다. 아래 단계 외의 기능완료를 소급하여 요구하지 않는다.

- [07 Fault Domains — 단계 링크](03_Fault_Domains.md#baseline-07-ready): **같은 vm-b/FD/SSH 완료**. 실제 출력/소비자/expiry를 전달하고 해당 owner가 이 lab 종료까지 보존한다.
- [08 Shared Security Model — 단계 링크](04_Shared_Security_Model.md#baseline-08-ready): **private object/restricted network/OS 조회**. 실제 출력/소비자/expiry를 전달하고 해당 owner가 이 lab 종료까지 보존한다.

생성 전 조직 승인(identity/권한/예산/expiry/서비스 지원/limit/capacity)을 확인한다. 실제 OCID/IP는 아직 생성하지 않았으면 비워 두고 해당 Create/readback 뒤 채운다. 동일 creation-event-id/OCID는 한 번의 제출만 뜻한다. 이 문서의 두 대상은 owner10 vm-a와 owner07 vm-b다. owner22 private-vm은 소비하지 않는다. 정리의 `INSTANCE_B_FD_ID`는 owner07 vm-b의 실제 OCID다. 실습에 없는 자원을 추가하지 않는다. 실제 checkpoint는 사용자 실행 전 비워 둔다.

최소 보존 입력은 두 instance OCID, 각각에 실제 연결된 boot volume OCID, Region, owner/expiry tag, vm-b의 기존 SSH 신뢰/주소/키, owner08 보안 evidence 경로다. boot OCID는 각 creator의 Storage/Boot volume attachment 조회에서 instance와 연결된 대상을 확인해 받는다. 같은 AD라는 사실만으로 attachment를 추정하지 않는다. [Boot Volume 조회 설명](../02_Compute/03_Boot_Volume.md#baseline-13-ready)은 조회 필드 참고이며 추가 volume 생성·실험 완료는 필요 없다. 하나라도 없으면 조회 준비에서 중단하고 creator에게 확인한다.

### 입력값과 실행 위치

Console에서 선택한 값만 사용한다. `<...>`는 예시 OCID나 비밀값이 아니라 **사용자가 대체할 입력 자리**다. 대체하지 않은 채 명령을 실행하지 않는다. Console → 우측 상단 Cloud Shell/Code Editor → Cloud Shell을 열면 OCI CLI가 사전 인증된다. Region 변경 뒤에는 Shell을 다시 열거나 명령의 `--region`을 바꾼다. SSH는 Cloud Shell이 아니라 본인 Linux/WSL/macOS 터미널에서 수행한다. 공인 IP/32도 그 터미널의 출구 주소여야 한다.

```bash
# 아래는 입력/출력의 대입 형식이다. 표의 단계 전에 생성 후 ID/IP export 행을 실행하지 않는다.
 # 생성 후 검증용 ID/IP는 해당 creator의 실제 응답에서 넣는다.
export REGION='<actual_REGION>'
# KEY는 Cloud Shell에서 지정하지 않는다; 아래 local SSH station에서만 export한다.
oci --version
```

이 변수 중 해당 절차에서 사용하는 값만 채운다. CLI 조회의 `data`를 증거 파일에 저장하고 실제 시간·작업 OCID·선택값을 기록한다. API key/private key/passphrase/인증 토큰은 기록하지 않는다. Cloud Shell 설정파일이나 세션 토큰을 출력하지 않는다. `oci --version`을 실행하는 주제의 기대 패턴은 버전 문자열뿐이다. namespace는 Object Storage 주제에서 별도 조회하며 다른 주제에는 요구하지 않는다. [Cloud Shell 공식 본문](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/cloudshellgettingstarted.htm).

### Workshop 입력 계약 — 단계와 host


- 입력/출력: Console/설정 필드
  - 값 또는 실제 획득 경로: owner07 same vm-b event; actual OCID/shape/boot size/owner/expiry/UTC times; 다섯 pillar 구현범위
  - 사용할 시점: 기존 creator 출력 확인 → readback
  - 실행 위치: 본문 Console/로컬 지정
  - 선택 이유: 두 기존 자원의 조회 범위

- 입력/출력: `KEY`
  - 값 또는 실제 획득 경로: owner15 Part A 전용 private key 절대경로; 내용 전송 금지
  - 사용할 시점: 준비 완료
  - 실행 위치: 로컬 SSH만
  - 선택 이유: 해당 공개키와 대응

- 입력/출력: `REGION`
  - 값 또는 실제 획득 경로: 승인된 기존 subscribed Region identifier
  - 사용할 시점: 생성 전
  - 실행 위치: Console/Cloud Shell
  - 선택 이유: API endpoint/데이터 위치


생성 전에는 행의 승인값/선행 완료값만 지정하고, 생성 후 ID/IP는 해당 owner 응답으로 채운다. Cloud Shell과 로컬 SSH의 환경변수는 공유되지 않는다. 로컬에는 KEY와 접속 주소만 다시 지정하고 guest disk 명령은 로그인한 guest에서만 실행한다. `<...>`가 남으면 해당 명령은 실행하지 않는다.

## 권한

### 실제 task/API별 공식 권한 본문 대조

아래는 이 문서 생성·조회·변경·정리 API의 공식 표 본문을 읽은 인용이다. 이 consumer는 아래 읽기 operation만 사용한다. verb/dependency 행 인용은 단일 API의 완전한 최소권한 도출이 아니다. 조직 Identity Domain Console user/group 작업은 해당 Domain 관리자 역할도 필요하며 legacy IAM user API 권한만으로 대체하지 않는다.

- `GetInstance`: INSTANCE_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- GetBootVolume=VOLUME_INSPECT, GetVnic=VNIC_READ, GetSubnet=SUBNET_READ를 추가 조회에 승인받는다. ListNetworkSecurityGroupSecurityRules: NETWORK_SECURITY_GROUP_LIST_SECURITY_RULES — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).

이번 consumer는 선행 creator의 리소스를 다시 생성·종료하지 않는다. 위 Get/List operation의 읽기 권한만 별도 승인한다. 공동 launch/VCN 생성 station에 참여하는 경우에만 해당 [creator 사전준비](#사전준비)의 Create/attachment 권한을 관리자와 확인한다. readback 때문에 `manage instance-family`, `manage volume-family`, `manage virtual-network-family` 세 가족을 일괄 요청하지 않는다. OS 조회는 `opc`/필요한 sudo, 로컬 파일 작성은 OCI IAM과 별개다.

## 비용및주의사항

- 신규 cloud resource **0개**, Cloud Shell private run directory/비민감 ledger만 작성한다. 두 VM/두 boot는 owner07/10의 기존 actual OCID를 읽으며 세 번째 VM/새 NSG/backup/LB를 만들지 않는다. 현재 shape/boot size/생성 UTC/승인 expiry를 기록하되 예상 비용을 actual billed cost로 쓰지 않는다. B 소비 종료를 owner07에 즉시 반환하며 distributed cloud/failover/restore는 NOT_SELECTED 또는 NOT_EXECUTED다.
- 생성/사용 당일 공식 단가와 Billing & Cost Management → Cost Analysis의 실제 Compartment/서비스 필터를 확인하되 청구 반영 지연을 기록한다. 예산 경보는 과금 차단이 아니다. 고객 데이터/secret/운영 자원을 사용하지 않는다.
- 이 문서는 Oracle 기술 본문 기반 교육 절차다. OCI/guest/IAM allow·deny 실측은 없으며 기대 결과를 실행 결과로 기록하지 않는다. 승인되지 않은 선택 확장은 실행하지 않는다.


## 실습구성

### 이번 Workshop에서 만드는 것과 진행 순서

owner07의 FD-separated 두 VM과 restricted network를 그대로 사용해 다섯 pillar의 실현/미실현 범위를 ledger에 남긴다.

1. **진입/선택:** 07 same event 확보. 사전준비 링크의 지정 단계/실제 출력이 없으면 제출하지 않는다.
2. **실제 작업:** 08 security 결과 → shape/expiry 기록 → 전용 ledger 생성 → tags/API 대조. 아래 Console 필드/명령을 해당 host에서 순서대로 실행한다. 같은 자원은 이름이 아니라 OCID/event로 재사용한다.
3. **완료/복구/반환:** 미구현 항목 명시. 결과검증 실패는 문제해결로 이동하고 동일 Create를 무작정 재제출하지 않는다.

**전달/정리 범위:** ledger는 local artifact; vm-b owner07, vm-a owner10; distributed cloud/restore/failover 미구현. 아래 정리 단계는 이 소유권을 넘어서 실행하지 않는다.


- **이번 생성/재사용/출력:** vm-b/NSG actual event 재사용 + 운영 ledger 신규 파일. five-pillar 개선 범위·미실현 항목 기록. diagram 없음.
- **검토 목표:** owner07이 이미 생성한 FD-separated vm-b와 owner08/23의 restricted NSG 증거를 검토하고 API 기반 운영 ledger와 pillar별 개선안 한 가지를 작성한다. 이 문서에서는 cloud 자원을 생성하지 않는다. Distributed Cloud는 NOT_SELECTED로 남긴다.
- **완료 증거:** 기존 두 대상 OCID + API readback + placement/SSH/보안 증거 검토 + pillar별 개선안 + creator 반환. 실제 OCI 실행 checkpoint는 비워 둔다.
- **retention:** 예약 소비자와 expiry가 있는 resource만 보존. idle 유료 resource를 다음 주제 때문에 무기한 유지하지 않는다.


## 실습절차

### 명령 실행 gate — 실패하면 다음 생성으로 진행하지 않는다

1. 아래 Bash는 **전용 Bash 세션**에서만 사용한다. Console/Cloud Shell 조회, 본인 로컬 SSH, 로그인한 Oracle Linux guest를 섞지 않는다. `exit 1`은 이 전용 세션을 닫는 안전 중단이다. `<...>`·빈 입력·타 owner OCID가 남으면 실행하지 않는다.
2. 성공을 요구하는 파일 작성/조회/설치/attach 작업은 실패 시 중단한다. 실패한 API 뒤 기존 JSON이나 기존 다운로드 파일을 성공 근거로 읽지 않는다. 기대 deny는 `if command; then ...; else RC=$?; ...; fi`로 별도 기록하고, 실제 timeout/IAM 오류/로컬 파일 오류를 구별한다.
3. 첫 변경 전에 정확한 OCID·owner·현재 설정과 승인 expiry를 저장한다. 공유 자원 변경 중 다른 소비자가 시작하거나 원복값을 확인할 수 없으면 변경을 중단한다. 삭제 성공은 API 수락이 아니라 같은 identity/Region의 정확한 대상 재조회로 판정한다. `NotAuthorizedOrNotFound` 단독은 삭제 증거가 아니다.


### 개별 완주 보정 — 생성·검증·소유권

ledger의 OCID/time/owner/expiry는 실행 후 실제값만 입력하며 미실행 항목은 NOT_EXECUTED다. owner07의 동일vm-b를 사용하고 이 주제에서 별도 VM을 제출하지 않는다.

1. 사전준비에서 완료한 owner07의 same AD/different FD 실제 vm-b 생성 event를 읽어 두 instance OCID와 placement 선택을 ledger에 연결한다. 여기서 Create를 다시 누르지 않는다. Reliability pillar의 placement 선택이며 app failover 성공은 아니다.
2. Shared Security Model의 nsg-admin 실제 creation-time 규칙을 연결한다. owner08/23이 이미 만든 `/32 TCP22`/private bucket의 실제 evidence를 재사용하고 유효 security list/NSG 전체를 읽어 security/compliance pillar의 제한접근 조치를 검증한다.
3. Performance and cost optimization은 두 VM의 actual shape/OCPU/memory, 생성·종료시각, boot size, expiry를 registry에 기록한다. 최대 두 VM·큰 default volume 금지·한 번의 network creation을 적용한다. 청구 반영 전 추정 비용을 actual billed cost처럼 쓰지 않는다.
4. Operational efficiency는 아래 ledger를 비민감 Cloud Shell 파일로 만든다. **먼저** 바로 아래 readback station의 두 instance/두 boot Get을 성공 조회한다. 아래 코드는 OCID·숫자·tags를 성공한 API 응답에서 자동 추출한다. boot OCID와 instance의 실제 attachment 관계는 creator Storage handoff와 대조한다. 보안/FD 검증을 완료한 뒤 pillar의 implemented에 해당 증거 경로/UTC를 적고 검증하지 않은 항목은 NOT_EXECUTED로 둔다. 실제 OCID와 작업시각만 입력하고 secret은 제외한다. vm-a/vm-b의 state와 owner/expiry tags가 기록과 같은지 확인한다.

```bash
# Cloud Shell: 새 owner09 run directory, 이전 evidence/ledger를 덮어쓰지 않는다.
set -euo pipefail
umask 077
WA_WORK=$(mktemp -d "$HOME/owner09-wa.XXXXXXXX") || exit 1
cd "$WA_WORK" || exit 1
printf '%s\n' "$WA_WORK" # 이 경로를 owner09 registry에 기록한다.
# 재진입은 이 경로의 evidence 조회/반환만; 전체 recipe 재실행은 새 directory.
# ledger 작성 전에 같은 두 instance/boot readback을 완료한다.
oci compute instance get --instance-id '<vm-a_OCID>' --region "$REGION" > wa-a.json || exit 1
oci compute instance get --instance-id '<vm-b_OCID>' --region "$REGION" > wa-b.json || exit 1
oci bv boot-volume get --boot-volume-id '<vm-a_creator_boot_OCID>' --region "$REGION" > wa-boot-a.json || exit 1
oci bv boot-volume get --boot-volume-id '<vm-b_creator_boot_OCID>' --region "$REGION" > wa-boot-b.json || exit 1
```

```bash
python3 - <<'PY'
import json
resources=[]
for role,suffix in (('vm-a','a'),('vm-b','b')):
    d=json.load(open('wa-'+suffix+'.json'))['data']
    boot=json.load(open('wa-boot-'+suffix+'.json'))['data']
    assert d['lifecycle-state']=='RUNNING'
    assert d['availability-domain']==boot['availability-domain']
    tags=d['freeform-tags']; cfg=d['shape-config']
    assert tags.get('owner') and tags.get('expiry'), 'STOP: creator owner/expiry handoff required'
    r={'role':role,'ocid':d['id'],'owner':tags['owner'],'expiry':tags['expiry'],
       'created_at':d['time-created'],'shape':d['shape'],'ocpus':cfg['ocpus'],
       'memory_in_gbs':cfg['memory-in-gbs'],'boot_volume_id':boot['id'],
       'boot_size_in_gbs':boot['size-in-gbs'],'deleted_at':None}
    for key in ('ocpus','memory_in_gbs','boot_size_in_gbs'):
        assert type(r[key]) in (int,float) and r[key]>0
    resources.append(r)
ledger={'lab':'oci-roadmap','resources':resources,
 'pillars':{
  'Security and compliance':{'implemented':'NOT_EXECUTED','observation':'owner08 private object/전체 SSH allow 합집합 확인 후 evidence 경로와 UTC 기록','unimplemented':'full audit'},
  'Reliability and resilience':{'implemented':'NOT_EXECUTED','observation':'아래 same AD/different FD 검증 후 evidence 경로와 UTC 기록','unimplemented':'application failover/backup restore'},
  'Performance and cost optimization':{'implemented':'API shape/boot/expiry readback','unimplemented':'billed cost/performance benchmark'},
  'Operational efficiency':{'implemented':'API-derived resource inventory','unimplemented':'automated operations'},
  'Distributed cloud':{'implemented':'NOT_SELECTED','unimplemented':'hybrid/multiregion'}},
 'distributed_cloud':'NOT_SELECTED','failover_test':'NOT_EXECUTED'}
with open('well-architected-lab-ledger.json','x') as f:
    json.dump(ledger,f,ensure_ascii=False,indent=2)
PY
```
ledger는 **Cloud Shell 전용 lab 작업 디렉터리**에서 작성한다. 위 작성 명령 전에 위 readback station에서 이미 `wa-a.json`/`wa-b.json`을 먼저 성공 조회한 뒤 각 resource의 `ocid`, `shape`, `shape-config.ocpus`, `shape-config.memory-in-gbs`, `time-created`, `freeform-tags.owner/expiry`를 ledger와 대조한다. 숫자 필드는 API에서 자동 추출하며 직접 placeholder를 편집하지 않는다. boot는 creator Storage의 실제 attachment OCID와 [Boot Volume 조회](../02_Compute/03_Boot_Volume.md#baseline-13-ready)의 GetBootVolume `size-in-gbs`/AD를 사용한다. 추가 GetBootVolume의 VOLUME_INSPECT 권한도 승인받는다. 증거 파일 경로/UTC를 pillar implemented에 연결하고 미실행 항목은 그대로 남긴다. `deleted_at=null`은 아직 삭제되지 않음을 뜻하며 owner07/10의 정확한 TERMINATED/boot 잔존 readback 후에만 actual UTC로 갱신한다. 이 consumer가 삭제하거나 예상 종료시각을 실제 삭제시간으로 쓰지 않는다.

5. [Oracle Well-Architected Framework — 현재 다섯 pillar 본문](https://docs.oracle.com/en/solutions/oci-best-practices/index.html) 기준으로 **Security/compliance, Reliability/resilience, Performance/cost optimization, Operational efficiency, Distributed cloud** 다섯 pillar를 검토한다. 현재 body가 Distributed cloud를 다섯 번째로 설명하므로 타 cloud의 다른 5-pillar 이름을 복사하지 않는다. Distributed cloud는 Deployment Models의 승인 hybrid/private 확장으로 연결하되 현재 NOT_SELECTED다. 한 Region VM 두 개를 multi-region/distributed-cloud 구현으로 꾸미지 않는다.
6. 각 pillar에서 이번 조회로 확인한 것, 부족한 것(backup restore/app HA/full audit/distributed cloud), 비용 승인 확장을 구분한 결과표를 남긴다. 완료조건은 기존 owner07 배치의 현재 API/SSH 확인과 owner08 보안 증거 검토, API 기반 ledger 및 pillar별 관찰·개선안 작성이다. vm-b 생성은 owner07의 완료 이력이며 여기서 다시 생성하지 않는다.

### 결과표 작성 예시 — 검증 후 한 줄씩

아래는 관찰/제안 형식이지 실행 결과가 아니다. 결과검증을 끝낸 뒤 ledger를 로컬 텍스트 편집기로 열어 각 pillar에 `observation`과 `improvement` 한 문장씩 추가한다. 검증 실패 항목은 implemented를 NOT_EXECUTED로 유지하고 실패 원인을 적는다. 새 cloud 자원을 만들 필요는 없다.


- Pillar: Security and compliance
  - 이번 검증 후 적을 관찰 예: vm-a 전체 허용 합집합 evidence의 UTC와 제한접근 범위
  - 실행하지 않을 개선 제안 예: 변경 시 전체 규칙 재검토

- Pillar: Reliability and resilience
  - 이번 검증 후 적을 관찰 예: 두 instance의 같은 AD/다른 FD 확인, app failover 미시험
  - 실행하지 않을 개선 제안 예: 별도 승인 후 app 복구 시험 설계

- Pillar: Performance and cost optimization
  - 이번 검증 후 적을 관찰 예: API의 shape/boot 크기/expiry, 청구액 아님
  - 실행하지 않을 개선 제안 예: 실제 비용 반영 후 적정 크기 검토

- Pillar: Operational efficiency
  - 이번 검증 후 적을 관찰 예: 두 creator 출력과 API inventory 대조
  - 실행하지 않을 개선 제안 예: 반복 조회 절차 자동화 검토

- Pillar: Distributed cloud
  - 이번 검증 후 적을 관찰 예: NOT_SELECTED; 단일 Region만 사용
  - 실행하지 않을 개선 제안 예: 요구가 생길 때 별도 설계 검토


## 결과검증


### baseline-09-ready


```bash
oci compute instance get --instance-id '<vm-a_OCID>' --region "$REGION" > wa-a.json || exit 1
oci compute instance get --instance-id '<vm-b_OCID>' --region "$REGION" > wa-b.json || exit 1
python3 - <<'PY'
import json
a=json.load(open('wa-a.json'))['data'];b=json.load(open('wa-b.json'))['data']
assert a['lifecycle-state']==b['lifecycle-state']=='RUNNING'
assert a['availability-domain']==b['availability-domain']
assert a['fault-domain']!=b['fault-domain']
print('FD placement assertions passed')
PY
```

로컬 전용 터미널에서 먼저 `export KEY='<owner15_actual_local_KEY>'`를 지정하고 [owner07 신뢰 handoff](03_Fault_Domains.md#baseline-07-ready)의 같은 vm-b fingerprint/주소와 owner15 KEY를 지정한다. 개인키를 Cloud Shell로 복사하지 않는다. 로컬에도 `umask 077` 뒤 `mktemp -d "$HOME/consumer09-ssh.XXXXXXXX"`로 새 전용 디렉터리를 만들고 그 안으로 이동한다. 출력된 절대경로를 기록하고 기존 SSH 증거를 덮어쓰지 않는다.
```bash
timeout --foreground 60s ssh -i "$KEY" -o IdentitiesOnly=yes -o HostKeyAlgorithms=ssh-ed25519 -o StrictHostKeyChecking=yes -o ConnectTimeout=10 -o ConnectionAttempts=1 -o ControlMaster=no -o ControlPath=none opc@'<vm-b_public_IP>' 'test "$(id -un)" = opc && hostname' > consumer09-ssh.txt || exit 1
date -u +%FT%TZ > consumer09-session-ended.txt || exit 1
```
기대 패턴 동일 AD/다른 FD, vm-b SSH exit 0/opc. 실제 장애복구 시험은 수행하지 않는다. **이 local fence 뒤 Cloud Shell의 WA_WORK로 돌아간 다음 아래 OCI 조회를 실행한다.** 새 Cloud Shell이면 기록한 절대경로로 `WA_WORK`를 다시 지정하고 해당 경로가 본인 소유 디렉터리이며 symlink가 아닌지 확인한 뒤 `cd "$WA_WORK"`한다. REGION도 같은 실제 Region으로 다시 지정한다. 경로를 잃었으면 추측하거나 기존 ledger를 재생성하지 말고 creator 반환에 필요한 증거부터 확인한다. local에서 wa-a/wa-b 파일을 새로 만들지 않는다.
```bash
oci compute instance get --instance-id '<vm-a_OCID>' --region "$REGION" > wa-a.json || exit 1
oci compute instance get --instance-id '<vm-b_OCID>' --region "$REGION" > wa-b.json || exit 1
# single NSG list is deliberately not used as whole-union acceptance.
# owner08 전체 membership/NSG/SL evidence를 아래 station에서 검증한다.
```
**보안 acceptance station(Cloud Shell):** [owner08 전체 VNIC→모든 NSG/SL 검증](04_Shared_Security_Model.md#baseline-08-ready)의 직전 성공 evidence 경로와 UTC를 ledger에 기록한다. owner08의 security-vnic.json/security-subnet.json 및 `security-nsg-ids.txt`/`security-sl-ids.txt`, 모든 ID별 성공 `--all` rule/list Get 파일, 전체 ingress 검토 record가 필요하다. 같은 actual vm-a primary VNIC/selected subnet인지 대조한다. 혼자 관리하는 격리 실습에서는 본인이 그 evidence 이후 NSG rules/SL rules/membership 변경이 없었는지 확인하고 조회시각을 기록한다. 공유 환경에서는 owner08/23이 같은 변경 없음 확인을 서면 인계한다. 현재 membership을 다시 읽는다:

```bash
export VNIC_ID='<owner10_actual_vm-a_primary_VNIC_OCID>'
oci network vnic get --vnic-id "$VNIC_ID" --region "$REGION" > wa-security-vnic-now.json || exit 1
export SUBNET_ID='<위_response의_actual_subnet-id>'
oci network subnet get --subnet-id "$SUBNET_ID" --region "$REGION" > wa-security-subnet-now.json || exit 1
export SECURITY_EVIDENCE_DIR='<owner08_actual_evidence_directory>'
python3 - <<'PY'
import json,os,pathlib
p=pathlib.Path(os.environ['SECURITY_EVIDENCE_DIR'])
a=json.load(open(p/'security-vnic.json'))['data']; b=json.load(open('wa-security-vnic-now.json'))['data']
s=json.load(open(p/'security-subnet.json'))['data']; t=json.load(open('wa-security-subnet-now.json'))['data']
assert a['id']==b['id']==os.environ['VNIC_ID'] and b['is-primary'] is True
assert a['subnet-id']==b['subnet-id']==s['id']==t['id']==os.environ['SUBNET_ID']
assert set(a['nsg-ids'])==set(b['nsg-ids'])
assert set(s['security-list-ids'])==set(t['security-list-ids'])
PY
```

미조회/실패/부분 페이지, membership 변경, 공유 환경의 no-change acknowledgment 누락은 **BLOCKED_SECURITY**다. owner08에게 모든 `--all` NSG+모든 SL 조회/ingress 재검증을 반환하며 single NSG나 미확인 과거 evidence로 통과하지 않는다. 이번 security claim은 **vm-a의 지정 UTC·변경 없음 승인 범위**만이며 vm-b/tenancy 전체 compliance는 주장하지 않는다.

기대 패턴 same AD/different FD/두 RUNNING+SSH, 실제 최소 shape/tags가 ledger와 일치; broad TCP22 없음은 위 owner08 범위에서만 판정한다. 분산/restore/failover는 미실행이라고 정확히 기록한다.

**증거 기록:** 실제 명령·UTC 시간·exit code·가려진 OCID·옵션 readback·검증 조건을 남긴다. 현재 이 문서의 실행상태는 `NOT_EXECUTED`다. output/creation-event-id/result는 실행 전 비워 둔다. 오류/NOT_SELECTED를 PASS로 바꾸지 않는다.

### 이 시나리오의 functional acceptance

| Positive | Negative / 미완료 판정 | Recovery / 반환 |
|---|---|---|
| resource ledger OCID/tags/API 일치 | API 필수 tag/숫자 누락 또는 기존 ledger면 작성 거부 | creator tags/attachment를 확인하고 기존 ledger 보존/새 전용 경로에서 생성 |

모두 실행 시 확인할 기대 패턴이다. 생성 전 guard negative는 OCI 장애를 주입한 것과 다르며 다른 자원을 추가하지 않는다. 오류 메시지와 실제 exit code가 없는 항목은 NOT_EXECUTED 또는 BLOCKED로 남긴다.

## 문제해결

### 공통 오류의 진단 순서

1. `NotAuthorizedOrNotFound` 또는 목록에 리소스가 없음: 먼저 Region/Compartment/OCID와 identity domain/group을 확인한다. 존재하지 않는 OCID와 권한 부족 모두 같은 오류로 보일 수 있다. 관리자에게 request-id·작업 시각·서비스·가려진 대상 OCID를 제공한다. 비밀키/토큰은 보내지 않는다.
2. `LimitExceeded`/quota/capacity 부족: Billing & Cost Management → Limits, Quotas and Usage와 생성 화면의 지원 shape/AD를 확인한다. 운영 quota 증가나 다른 Region 구독을 임의 요청하지 않는다. 승인 범위 안의 더 작은 compatible shape 또는 다른 AD를 선택하거나 `BLOCKED_CAPACITY`를 기록한다.
3. 장시간 provisioning/예상 밖 state: Work Requests/오류 상세를 읽고 같은 요청을 반복 제출하지 않는다. 만들어진 부분 리소스(boot/IP/attachment 포함)를 먼저 조사한다.
4. 메뉴 이름이 다름: 공식 task body의 동등한 탭(예: Storage 또는 Attached Block Volumes)을 찾는다. 필드가 문서와 다르면 값·스크린샷에서 secrets를 가린 뒤 관리자 또는 Oracle Support에 확인하고 제출을 중단한다. 이 문서는 현재 tenancy의 live Console 화면을 검사한 결과가 아니다.

### SSH 실패

`Connection timed out`이면 공인 IP → IGW enabled → subnet route의 IGW target → security list와 모든 NSG의 유효 허용 규칙 → guest sshd/firewall 순으로 확인한다. NSG와 security list는 허용 규칙의 합집합이므로 좁은 NSG가 넓은 security list를 취소하지 않는다. `Permission denied (publickey)`이면 Linux 사용자(opc; 이 트랙은 Oracle Linux 9), 제출한 `.pub`, 대응 private key와 권한을 확인한다. 호스트키 변경은 재생성 여부를 확인한 뒤 처리하며 `StrictHostKeyChecking=no`로 우회하지 않는다. 복구가 필요하면 공식 serial Console 절차를 관리자의 승인하에 사용한다. Ping 불가만으로 SSH 실패를 판정하지 않는다.

### 이 주제의 실패 지점

ledger state가 실제 API state와 다르면 문서값을 믿지 말고 OCID·Region을 확인하고 갱신한다. 여러 FD VM이 있다고 app failover/Distributed Cloud 구현 완료로 표시하지 않는다.

## 리소스정리

consumer09는 vm-b(owner07)/vm-a(owner10)를 직접 종료하지 않는다. owner07 registry에 `consumer=09`, actual INSTANCE_B_FD_ID/REGION, WA_WORK/ledger/placement/boot/security evidence 경로, local consumer09-ssh.txt와 session-end UTC, 실제 SSH exit, `outstanding_changes=none`, `lease=RELEASED` 또는 미완료 사유/승인 retention deadline, owner07 acknowledgment UTC/alias를 남긴다. owner10/08의 evidence 사용도 반환한다. [owner07 정리](03_Fault_Domains.md#리소스정리)는 09/16 반환 이후만 실행한다. `deleted_at=null`은 creator가 exact resource 종료/boot closeout 증거와 실제 UTC를 반환하기 전 유지한다. 증거와 ledger를 승인 보존 경로로 복사·hash 확인 후 WA_WORK에서 이번에 생성한 wa-a.json, wa-b.json, wa-boot-a.json, wa-boot-b.json, wa-security-vnic-now.json, wa-security-subnet-now.json, well-architected-lab-ledger.json만 exact path로 제거하고 rmdir한다. local SSH 증거도 보존 확인 후 기록한 전용 로컬 디렉터리의 exact 두 파일만 제거하고 빈 디렉터리를 rmdir한다. 혼자 하는 실습에서는 owner 번호는 자신의 생성 단계 식별자다. 다른 소비자가 없음을 확인하고 본인 기록에 반환 시각을 남기면 되며 별도 서면 승인자는 필요 없다. 공유 환경의 owner acknowledgment와 다른 소비자 반환 gate는 그대로 필요하다. 재진입은 저장한 WA_WORK의 소유권/비symlink 확인 후 반환·정리만 수행하며 기존 ledger를 덮어쓰지 않는다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 아래 본문은 각 생성 필드·설정·검증·정리의 기술 근거이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [44] [Learn About the Well-Architected Framework for Oracle Cloud Infrastructure — Well-Architected Framework](https://docs.oracle.com/en/solutions/oci-best-practices/index.html)
  - 확인 범위: The best practices for.

### 추가 실행 절차 근거 (본문 확인)

- [regions](https://docs.oracle.com/en-us/iaas/Content/General/Concepts/regions.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [launch](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/launchinginstance.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [security](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/securitylists.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [cloudshell](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/cloudshellgettingstarted.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
