# Regions

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../README.md)를 확인하세요.

> 실행 절차 작성본 / OCI 미실행. Oracle 공식 본문 근거와 교육용 선택을 구분한다. secrets·실제 고객값·diagram 없음.

## 학습목표

- READY인 기존 구독 Region identifier를 선택하고 owner19의 같은 shared-vcn 생성 요청에 그 endpoint를 사용한다.
- 선택 Region에서 VCN AVAILABLE를 조회하고 다른 기존 구독 Region이 있을 때만 목록 비교를 수행한다. 새 Region은 구독하지 않는다.

위 항목은 사용자 실행 후 확인할 완료 조건이며 현재 관측 결과가 아니다. 개념은 아래 해당 생성 필드·configuration·test 단계에서 함께 적용한다.

## 대응 Roadmap 항목

- 항목명: **Regions**
- Roadmap 경로: **Introduction to OCI → Regions**
- Node ID: `sequence-1-1`
- 상위 항목: **Introduction to OCI** (`sequence-1-0`)
- 문서 범위: 이 항목의 생성·선택·기능검증·정리. 실습 미실행; 아래 기대 패턴은 실제 출력 아님.
- 번호는 navigation 식별자이며 실행 순서는 사전준비의 단계별 의존성에 따른다.

## 사전준비

### 기준 계약과 실제 단계 선행조건

[공통 baseline-contract.json](../../baseline-contract.json)의 Oracle Linux 9 / opc / CIDR / 비용 상한을 따른다. 문서 번호는 실행 순서가 아니다. **준비 station과 생성 이후 ready를 분리**한다. 아래 단계 외의 기능완료를 소급하여 요구하지 않는다.

- [00 Introduction — 단계 링크](../00_Introduction/00_Introduction.md#baseline-00-ready): **ACTIVE lab-root; VCN 생성 전 Region 선택**. 실제 출력/소비자/expiry를 전달하고 해당 owner가 이 lab 종료까지 보존한다.
- [19 Virtual Cloud Network (VCN) — 단계 링크](../03_Virtual_Cloud_Network_VCN/00_Virtual_Cloud_Network_VCN.md#vcn-create-station): **공동 생성 station: 사전준비 충족만; VCN 완료 요구 없음**. 실제 출력/소비자/expiry를 전달하고 해당 owner가 이 lab 종료까지 보존한다.

생성 전 조직 승인(identity/권한/예산/expiry/서비스 지원/limit/capacity)을 확인한다. 실제 OCID/IP는 아직 생성하지 않았으면 비워 두고 해당 Create/readback 뒤 채운다. 동일 creation-event-id/OCID는 한 번의 제출만 뜻한다. `INSTANCE_A_ID`는 owner10 vm-a의 OCID, `INSTANCE_B_ID`는 owner22 private-vm의 OCID이며 owner07 vm-b와 다르다. 실습에 없는 자원을 추가하지 않는다. 실제 checkpoint는 사용자 실행 전 비워 둔다.

### 입력값과 실행 위치

Console에서 선택한 값만 사용한다. `<...>`는 예시 OCID나 비밀값이 아니라 **사용자가 대체할 입력 자리**다. 대체하지 않은 채 명령을 실행하지 않는다. Console → 우측 상단 Cloud Shell/Code Editor → Cloud Shell을 열면 OCI CLI가 사전 인증된다. Region 변경 뒤에는 Shell을 다시 열거나 명령의 `--region`을 바꾼다. SSH는 Cloud Shell이 아니라 본인 Linux/WSL/macOS 터미널에서 수행한다. 공인 IP/32도 그 터미널의 출구 주소여야 한다.

```bash
# 아래는 입력/출력의 대입 형식이다. 표의 단계 전에 생성 후 ID/IP export 행을 실행하지 않는다.
 # 생성 후 검증용 ID/IP는 해당 creator의 실제 응답에서 넣는다.
export REGION='<actual_REGION>'
export TENANCY_ID='<actual_TENANCY_ID>'
export VCN_ID='<actual_VCN_ID>'
oci --version
```

이 변수 중 해당 절차에서 사용하는 값만 채운다. CLI 조회의 `data`를 증거 파일에 저장하고 실제 시간·작업 OCID·선택값을 기록한다. API key/private key/passphrase/인증 토큰은 기록하지 않는다. Cloud Shell 설정파일이나 세션 토큰을 출력하지 않는다. `oci --version`을 실행하는 주제의 기대 패턴은 버전 문자열뿐이다. namespace는 Object Storage 주제에서 별도 조회하며 다른 주제에는 요구하지 않는다. [Cloud Shell 공식 본문](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/cloudshellgettingstarted.htm).

### Workshop 입력 계약 — 단계와 host

| 입력/출력 | 값 또는 실제 획득 경로 | 사용할 시점 | 실행 위치 | 선택 이유 |
|---|---|---|---|---|
| Console/설정 필드 | 기존 subscribed Region identifier 한 곳; 새 Subscribe 없음; owner19 same VCN event | 생성 전 선택 → 생성 뒤 readback | 본문 Console/로컬 지정 | 이 topic의 최소 생성 범위 |
| `LAB_COMPARTMENT_ID` | owner00 Create 응답 OCID; topic00에서는 생성 뒤 | 선행 완료/생성 후 | Console/Cloud Shell | 정책/자원 scope |
| `REGION` | 승인된 기존 subscribed Region identifier | 생성 전 | Console/Cloud Shell | API endpoint/데이터 위치 |
| `TENANCY_ID` | Console tenancy details의 실제 OCID | 생성 전 | Cloud Shell | AD/구독 조회 |
| `VCN_ID` | owner19 Create 응답 id | 선행 완료/생성 후 | Console/Cloud Shell | shared-vcn 참조 |

생성 전에는 행의 승인값/선행 완료값만 지정하고, 생성 후 ID/IP는 해당 owner 응답으로 채운다. Cloud Shell과 로컬 SSH의 환경변수는 공유되지 않는다. 로컬에는 KEY와 접속 주소만 다시 지정하고 guest disk 명령은 로그인한 guest에서만 실행한다. `<...>`가 남으면 해당 명령은 실행하지 않는다.

## 권한

### 실제 task/API별 공식 권한 본문 대조

아래는 이 문서 생성·조회·변경·정리 API의 공식 표 본문을 읽은 인용이다. 위 family 예제는 편의상 넓은 compartment-scope grant이며 **최소권한 정책이라고 부르지 않는다**. verb/dependency 행 인용은 단일 API의 완전한 최소권한 도출이 아니다. 조직 Identity Domain Console user/group 작업은 해당 Domain 관리자 역할도 필요하며 legacy IAM user API 권한만으로 대체하지 않는다.

- `GetVcn`: VCN_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `ListRegionSubscriptions`: TENANCY_INSPECT — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/policyreference/iampolicyreference.htm).
- `ListVcns`: VCN_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).

이번 consumer는 선행 creator의 리소스를 다시 생성·종료하지 않는다. 위 Get/List operation의 읽기 권한만 별도 승인한다. 공동 launch/VCN 생성 station에 참여하는 경우에만 해당 [creator 사전준비](#사전준비)의 Create/attachment 권한을 관리자와 확인한다. readback 때문에 `manage instance-family`, `manage volume-family`, `manage virtual-network-family` 세 가족을 일괄 요청하지 않는다. OS 조회는 `opc`/필요한 sudo, 로컬 파일 작성은 OCI IAM과 별개다.

## 비용및주의사항

- Region은 생성하는 과금 자원이 아니며 새 Subscribe를 수행하지 않는다. owner19의 shared-vcn **같은 생성 1회**를 사용하고 여기서 두 번째 VCN/subnet/gateway/VM을 추가하지 않는다. READY/데이터 위치/해당 Region 서비스 가용성을 먼저 승인한다. 다른 이미 구독된 Region 비교는 목록 조회만이며 DR/복제 비용 실험이 아니다.
- 생성/사용 당일 공식 단가와 Billing & Cost Management → Cost Analysis의 실제 Compartment/서비스 필터를 확인하되 청구 반영 지연을 기록한다. 예산 경보는 과금 차단이 아니다. 고객 데이터/secret/운영 자원을 사용하지 않는다.
- 이 문서는 Oracle 기술 본문 기반 교육 절차다. OCI/guest/IAM allow·deny 실측은 없으며 기대 결과를 실행 결과로 기록하지 않는다. 승인되지 않은 선택 확장은 실행하지 않는다.


## 실습구성

### 이번 Workshop에서 만드는 것과 진행 순서

기존 subscribed Region 하나를 선택하여 owner19와 같은 VCN 제출을 사용한다. 추가 구독/VCN/DR은 만들지 않는다.

1. **진입/선택:** Region 선택/서비스 가용성 확인. 사전준비 링크의 지정 단계/실제 출력이 없으면 제출하지 않는다.
2. **실제 작업:** owner19 shared-vcn Create → 선택 endpoint Get. 아래 Console 필드/명령을 해당 host에서 순서대로 실행한다. 같은 자원은 이름이 아니라 OCID/event로 재사용한다.
3. **완료/복구/반환:** 기존 다른 Region 있을 때만 목록 비교. 결과검증 실패는 문제해결로 이동하고 동일 Create를 무작정 재제출하지 않는다.

**전달/정리 범위:** REGION과 VCN_ID; VCN은 owner19만 정리. 아래 정리 단계는 이 소유권을 넘어서 실행하지 않는다.


- **이번 생성/재사용/출력:** 신규 독립 Region 자원 없음. 실제 shared-vcn create event의 Region 선택. VCN 재사용.
- **생성 목표:** Virtual Cloud Network (VCN)의 단일 실제 생성 event에서 Regions 옵션을 직접 선택하고 저장·기능 검증한다. 독립 중복 생성은 하지 않는다.
- **완료 증거:** 생성 전 옵션 선택 + 실제 제출 대상 OCID + API readback + 해당 기능검증 + dependency 순서 정리. 실제 OCI 실행 checkpoint는 비워 둔다.
- **retention:** 예약 소비자와 expiry가 있는 resource만 보존. idle 유료 resource를 다음 주제 때문에 무기한 유지하지 않는다.


## 실습절차

### 명령 실행 gate — 실패하면 다음 생성으로 진행하지 않는다

1. 아래 Bash는 **전용 Bash 세션**에서만 사용한다. Console/Cloud Shell 조회, 본인 로컬 SSH, 로그인한 Oracle Linux guest를 섞지 않는다. `exit 1`은 이 전용 세션을 닫는 안전 중단이다. `<...>`·빈 입력·타 owner OCID가 남으면 실행하지 않는다.
2. 성공을 요구하는 파일 작성/조회/설치/attach 작업은 실패 시 중단한다. 실패한 API 뒤 기존 JSON이나 기존 다운로드 파일을 성공 근거로 읽지 않는다. 기대 deny는 `if command; then ...; else RC=$?; ...; fi`로 별도 기록하고, 실제 timeout/IAM 오류/로컬 파일 오류를 구별한다.
3. 첫 변경 전에 정확한 OCID·owner·현재 설정과 승인 expiry를 저장한다. 공유 자원 변경 중 다른 소비자가 시작하거나 원복값을 확인할 수 없으면 변경을 중단한다. 삭제 성공은 API 수락이 아니라 같은 identity/Region의 정확한 대상 재조회로 판정한다. `NotAuthorizedOrNotFound` 단독은 삭제 증거가 아니다.


### 개별 완주 보정 — 생성·검증·소유권

region-selection은 owner19 제출 전 준비이며 baseline-05-ready는 생성 후 검증이다. ListRegionSubscriptions의 TENANCY_INSPECT를 승인받고 신규 Region subscribe는 하지 않는다.

<a id="region-selection"></a>

1. 생성 전 승인 데이터 위치/사용자 접근 위치/서비스 가용성 기준을 작성한다. Console Region picker에서 **기존 subscribed Region** 한 곳을 선택한다. Region 이름과 identifier를 구분하며 추가 Subscribe to region은 관리자 승인 없이 누르지 않는다. [Regions and Availability Domains](https://docs.oracle.com/en-us/iaas/Content/General/Concepts/regions.htm).
**Create 이전 Cloud Shell station:** Console tenancy details의 실제 TENANCY_ID와 승인 Region identifier를 지정하고 새 전용 증거 디렉터리에서 아래를 먼저 실행한다. READY가 아니면 owner19에게 BLOCKED_REGION을 반환하며 새 구독/VCN 제출을 하지 않는다.

```bash
set -euo pipefail
umask 077
REGION_WORK=$(mktemp -d "$HOME/owner05-region.XXXXXXXX") || exit 1
cd "$REGION_WORK" || exit 1
oci iam region-subscription list --tenancy-id "$TENANCY_ID" --region "$REGION" > region-subscriptions.json || exit 1
python3 - <<'PY'
import json,os
r=json.load(open('region-subscriptions.json'))['data']
assert os.environ['REGION'] and '<' not in os.environ['REGION']
assert any(x['region-name']==os.environ['REGION'] and x['status']=='READY' for x in r), 'BLOCKED_REGION: do not submit Create'
PY
```

2. READY 검증 후 Console Region picker와 `$REGION`이 같은 identifier인지 대조한다. VCN 생성 station에 참여하여 이 Region에서 Name shared-vcn/CIDR/DNS label을 실제 입력하고 Create를 한 번 제출한다. 이미 같은 session에서 생성했다면 같은 생성 event 증거를 연결한다. 다음 조회로 선택 Region의 API 범위를 확인한다.

```bash
oci network vcn get --vcn-id "$VCN_ID" --region "$REGION" > region-vcn.json || exit 1
```
3. 이미 구독된 다른 Region이 있을 때만 그 Region의 같은 Compartment VCN 목록을 조회한다. 별도 VCN을 만들 필요 없다. `oci network vcn list --compartment-id "$LAB_COMPARTMENT_ID" --region '<다른_기존_구독_Region>' --all`에서 생성 OCID가 없음을 확인한다. 하나의 Region만 있으면 비교 substep은 NOT_AVAILABLE로 남기고 새 구독을 만들지 않는다.
다른 Region 비교는 Cloud Shell에서 승인된 READY identifier를 `OTHER_REGION`에 지정한 뒤 성공한 전체 목록으로만 판정한다:

```bash
export OTHER_REGION='<다른_기존_READY_Region_identifier>'
oci network vcn list --compartment-id "$LAB_COMPARTMENT_ID" --region "$OTHER_REGION" --all > other-region-vcns.json || exit 1
python3 - <<'PY'
import json,os
assert os.environ['OTHER_REGION']!=os.environ['REGION']
subs=json.load(open('region-subscriptions.json'))['data']
assert any(x['region-name']==os.environ['OTHER_REGION'] and x['status']=='READY' for x in subs)
rows=json.load(open('other-region-vcns.json'))['data']
assert isinstance(rows,list)
assert all(x['id']!=os.environ['VCN_ID'] for x in rows)
print('complete other-region list: exact VCN absent')
PY
```
같은 identity/Compartment/UTC/exit0을 기록한다. 실패한 조회/첫 페이지/권한 오류는 absence 증거가 아니다.

4. 같은 Compartment는 다른 Region에도 존재하지만 VCN은 생성 Region의 리소스임을 설명한다. Region간 독립성이 cross-region DR 성공을 뜻하지 않는다.

## 결과검증

<a id="baseline-05-ready"></a>

```bash
python3 - <<'PY'
import json,os
r=json.load(open('region-subscriptions.json'))['data']
assert any(x['region-name']==os.environ['REGION'] and x['status']=='READY' for x in r)
v=json.load(open('region-vcn.json'))['data']
assert v['id']==os.environ['VCN_ID'] and v['lifecycle-state']=='AVAILABLE'
print('region assertions passed')
PY
```
기대 패턴 READY subscription, 선택 endpoint에서 VCN AVAILABLE. Region별 가격/서비스 지원은 생성 당일 공식 서비스 정보를 추가 확인한다.

**증거 기록:** 실제 명령·UTC 시간·exit code·가려진 OCID·옵션 readback·검증 조건을 남긴다. 현재 이 문서의 실행상태는 `NOT_EXECUTED`다. output/creation-event-id/result는 실행 전 비워 둔다. 오류/NOT_SELECTED를 PASS로 바꾸지 않는다.

### 이 시나리오의 functional acceptance

| Positive | Negative / 미완료 판정 | Recovery / 반환 |
|---|---|---|
| READY subscription/selected endpoint VCN exists | 다른 기존 subscribed Region 목록에 같은 regional VCN OCID 없음(있을 때만) | 원래 REGION endpoint로 다시 get 성공; 구독1개면 comparison NOT_AVAILABLE |

모두 실행 시 확인할 기대 패턴이다. 생성 전 guard negative는 OCI 장애를 주입한 것과 다르며 다른 자원을 추가하지 않는다. 오류 메시지와 실제 exit code가 없는 항목은 NOT_EXECUTED 또는 BLOCKED로 남긴다.

## 문제해결

### 공통 오류의 진단 순서

1. `NotAuthorizedOrNotFound` 또는 목록에 리소스가 없음: 먼저 Region/Compartment/OCID와 identity domain/group을 확인한다. 존재하지 않는 OCID와 권한 부족 모두 같은 오류로 보일 수 있다. 관리자에게 request-id·작업 시각·서비스·가려진 대상 OCID를 제공한다. 비밀키/토큰은 보내지 않는다.
2. `LimitExceeded`/quota/capacity 부족: Billing & Cost Management → Limits, Quotas and Usage와 생성 화면의 지원 shape/AD를 확인한다. 운영 quota 증가나 다른 Region 구독을 임의 요청하지 않는다. 승인 범위 안의 더 작은 compatible shape 또는 다른 AD를 선택하거나 `BLOCKED_CAPACITY`를 기록한다.
3. 장시간 provisioning/예상 밖 state: Work Requests/오류 상세를 읽고 같은 요청을 반복 제출하지 않는다. 만들어진 부분 리소스(boot/IP/attachment 포함)를 먼저 조사한다.
4. 메뉴 이름이 다름: 공식 task body의 동등한 탭(예: Storage 또는 Attached Block Volumes)을 찾는다. 필드가 문서와 다르면 값·스크린샷에서 secrets를 가린 뒤 관리자 또는 Oracle Support에 확인하고 제출을 중단한다. 이 문서는 현재 tenancy의 live Console 화면을 검사한 결과가 아니다.

### 이 주제의 실패 지점

Region에 VCN이 안 보이면 Console picker와 CLI --region을 대조한다. IAM Compartment가 tenancy-wide라는 사실은 VCN이 모든 Region에 보인다는 뜻이 아니다.

## 리소스정리

새 Region/subnet/VCN을 만들거나 삭제하지 않는다. REGION_WORK의 region-subscriptions.json, region-vcn.json, 선택 비교 시 other-region-vcns.json을 승인 evidence 경로에 보존한다. creator registry에 `consumer=05`, actual `REGION/VCN_ID`, READY 조회 UTC, VCN Get UTC/증거경로, comparison NOT_AVAILABLE 또는 actual OTHER_REGION/증거, `outstanding_changes=none`, `lease=RELEASED`, 반환 UTC, owner19 acknowledgment UTC/alias를 기록한다. 미완료면 사유/승인 보존기한을 대신 기록한다. [owner19 리소스정리](../03_Virtual_Cloud_Network_VCN/00_Virtual_Cloud_Network_VCN.md#리소스정리)에서 모든 소비자 반환 후만 VCN을 정리한다. 증거 보존을 확인한 뒤 이번 REGION_WORK의 위 exact 파일만 rm하고 rmdir한다. 공유 CIDR 계획이나 다른 owner evidence는 변경하지 않는다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 아래 본문은 각 생성 필드·설정·검증·정리의 기술 근거이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [23] [Regions and Availability Domains — Regions](https://docs.oracle.com/en-us/iaas/Content/General/Concepts/regions.htm)
  - 확인 범위: About Regions and Availability Domains / Regions are independent.

### 추가 실행 절차 근거 (본문 확인)

- [regions](https://docs.oracle.com/en-us/iaas/Content/General/Concepts/regions.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [vcn](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/create_vcn.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [cloudshell](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/cloudshellgettingstarted.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
