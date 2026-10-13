# CIDR Blocks

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../README.md)를 확인하세요.

> 실행 절차 작성본 / OCI 미실행. Oracle 공식 본문 근거와 교육용 선택을 구분한다. secrets·실제 고객값·diagram 없음.

## 학습목표

- 실제 연결 예정망을 입력해 VCN/public/private CIDR의 strict parsing·containment·non-overlap을 로컬에서 통과시킨다.
- 동일 계획을 owner19/23/21의 실제 생성 필드에 반영하고 API CIDR를 비교한다. overlap negative는 로컬 계산으로만 수행한다.

## 대응 Roadmap 항목

- 항목명: **CIDR Blocks**
- Roadmap 경로: **Virtual Cloud Network (VCN) → CIDR Blocks**
- Node ID: `sequence-3-1`
- 상위 항목: **Virtual Cloud Network (VCN)** (`sequence-3-0`)
- 문서 범위: 실제 연결 예정망을 입력해 VCN/public/private CIDR의 strict parsing·containment·non-overlap을 로컬에서 통과시킨다. 동일 계획을 owner19/23/21의 실제 생성 필드에 반영하고 API CIDR를 비교한다. overlap negative는 로컬 계산으로만 수행한다. 실제 OCI 실행 결과가 아니라 이 절차의 완료 조건이다.
- 번호는 navigation 식별자이며 실행 순서는 사전준비의 단계별 의존성에 따른다.

## 사전준비

### 기준 계약과 실제 단계 선행조건

[공통 baseline-contract.json](../../baseline-contract.json)의 Oracle Linux 9 / opc / CIDR / 비용 상한을 따른다. 문서 번호는 실행 순서가 아니다. **준비 station과 생성 이후 ready를 분리**한다. 아래 단계 외의 기능완료를 소급하여 요구하지 않는다.

- [00 Introduction — 단계 링크](../00_Introduction/00_Introduction.md#baseline-00-ready): **조직 승인 주소/연결 예정망; 로컬 계획 자체에는 VCN 불필요**. 실제 출력/소비자/expiry를 전달하고 해당 owner가 이 lab 종료까지 보존한다.

생성 전 조직 승인(identity/권한/예산/expiry/서비스 지원/limit/capacity)을 확인한다. 실제 OCID/IP는 아직 생성하지 않았으면 비워 두고 해당 Create/readback 뒤 채운다. 동일 creation-event-id/OCID는 한 번의 제출만 뜻한다.  실습에 없는 자원을 추가하지 않는다. 실제 checkpoint는 사용자 실행 전 비워 둔다.

### 입력값과 실행 위치

Console에서 선택한 값만 사용한다. `<...>`는 예시 OCID나 비밀값이 아니라 **사용자가 대체할 입력 자리**다. 대체하지 않은 채 명령을 실행하지 않는다. Console → 우측 상단 Cloud Shell/Code Editor → Cloud Shell을 열면 OCI CLI가 사전 인증된다. Region 변경 뒤에는 Shell을 다시 열거나 명령의 `--region`을 바꾼다. SSH는 Cloud Shell이 아니라 본인 Linux/WSL/macOS 터미널에서 수행한다. 공인 IP/32도 그 터미널의 출구 주소여야 한다.

```bash
# 아래는 입력/출력의 대입 형식이다. 표의 단계 전에 생성 후 ID/IP export 행을 실행하지 않는다.
 # 생성 후 검증용 ID/IP는 해당 creator의 실제 응답에서 넣는다.
export REGION='<actual_REGION>'
export VCN_ID='<actual_VCN_ID>'
export VCN_CIDR='10.77.0.0/16'
export PUBLIC_SUBNET_CIDR='10.77.10.0/24'
export PRIVATE_SUBNET_CIDR='10.77.20.0/24'
oci --version
```

이 변수 중 해당 절차에서 사용하는 값만 채운다. CLI 조회의 `data`를 증거 파일에 저장하고 실제 시간·작업 OCID·선택값을 기록한다. API key/private key/passphrase/인증 토큰은 기록하지 않는다. Cloud Shell 설정파일이나 세션 토큰을 출력하지 않는다. `oci --version`을 실행하는 주제의 기대 패턴은 버전 문자열뿐이다. namespace는 Object Storage 주제에서 별도 조회하며 다른 주제에는 요구하지 않는다. [Cloud Shell 공식 본문](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/cloudshellgettingstarted.htm).

### Workshop 입력 계약 — 단계와 host

| 입력/출력 | 값 또는 실제 획득 경로 | 사용할 시점 | 실행 위치 | 선택 이유 |
|---|---|---|---|---|
| Console/설정 필드 | 승인 VCN/public/private IPv4 CIDR; 실제 connected CIDR JSON 배열; local negative overlap만 | 생성 전 선택 → 생성 뒤 readback | 본문 Console/로컬 지정 | 이 topic의 최소 생성 범위 |
| `CONNECTED_CIDRS_JSON` | 조직이 확인한 실제 연결 예정 IPv4 CIDR JSON 배열; 독립망이면 승인된 `[]` | 생성 전 | 로컬 | 누락된 VPN/on-prem overlap 방지 |
| `PRIVATE_SUBNET_CIDR` | owner20 승인 계획; 기본 후보 10.77.20.0/24 | 생성 전 | 로컬/Console | private subnet 주소 |
| `PUBLIC_SUBNET_CIDR` | owner20 승인 계획; 기본 후보 10.77.10.0/24 | 생성 전 | 로컬/Console | public subnet 주소 |
| `REGION` | 승인된 기존 subscribed Region identifier | 생성 전 | Console/Cloud Shell | API endpoint/데이터 위치 |
| `VCN_CIDR` | owner20 승인 계획; 기본 후보 10.77.0.0/16 | 생성 전 | 로컬/Console/Cloud Shell | 중첩/중복 검증 |
| `VCN_ID` | owner19 Create 응답 id | 선행 완료/생성 후 | Console/Cloud Shell | shared-vcn 참조 |

생성 전에는 행의 승인값/선행 완료값만 지정하고, 생성 후 ID/IP는 해당 owner 응답으로 채운다. Cloud Shell과 로컬 SSH의 환경변수는 공유되지 않는다. 로컬에는 KEY와 접속 주소만 다시 지정하고 guest disk 명령은 로그인한 guest에서만 실행한다. `<...>`가 남으면 해당 명령은 실행하지 않는다.

## 권한

로컬 ipaddress strict/containment/overlap 계산에는 OCI 권한이 없다. 생성값 비교에는 정확한 owner19 VCN의 GetVcn과 owner21/23 subnet의 GetSubnet만 읽는다. CIDR 추가/변경·새 subnet 생성 정책은 이 계획 consumer가 요구하지 않고 해당 creator에서 승인한다.

### 실제 operation과 공식 권한

- `GetSubnet`: SUBNET_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `GetVcn`: VCN_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).

위 permission 이름은 IAM policy resource-type이 아니다. 조직 관리자가 기존 group/compartment scope와 conditional dependency를 실제 API 표에 대조한다. 편의 family grant를 최소권한 정책으로 부르지 않으며 실습자는 policy 작성 권한을 자동 인수하지 않는다.

## 비용및주의사항

- CIDR 계산 자체는 로컬 Python 작업이며 독립 유료 자원은 없다. 승인 계획을 owner19 VCN·owner23 public-admin·owner21 private-app의 동일 생성 필드에 적용하며 cidr-check용 VCN/subnet/VM은 만들지 않는다.
- 실제 연결 예정 VPN/on-prem/기존 VCN 대역을 입력하기 전 non-overlap을 승인하지 않는다. 충돌 negative는 로컬 계산으로만 수행한다. OCI에 겹친 subnet을 제출하거나 현재 공유 subnet CIDR를 임의 변경하는 시험은 제외한다.

실제 OCI 작업은 아직 수행하지 않았다. 기대값은 실행 시 대조할 조건이며 무료 사용이나 과금 강제 차단을 보장하지 않는다. 승인 expiry가 끝나면 새 시험을 중단하고 위 소유권 순서로 원복·반환한다.

## 실습구성

### 이번 Workshop에서 만드는 것과 진행 순서

로컬 CIDR 계획을 먼저 계산하여 VCN/public/private 주소 입력으로 전달한다. CIDR은 설정값이지 독립 자원 노드가 아니다.

1. **진입/선택:** 실제 연결 예정망 입력. 사전준비 링크의 지정 단계/실제 출력이 없으면 제출하지 않는다.
2. **실제 작업:** strict parse/containment/non-overlap → 19 VCN/23 public 생성에 적용 → API CIDR 비교. 아래 Console 필드/명령을 해당 host에서 순서대로 실행한다. 같은 자원은 이름이 아니라 OCID/event로 재사용한다.
3. **완료/복구/반환:** private는 21 생성 후 비교. 결과검증 실패는 문제해결로 이동하고 동일 Create를 무작정 재제출하지 않는다.

**전달/정리 범위:** 승인 세 CIDR/overlap 결과; 별도 subnet 없음, 삭제는 owner19/21/23. 아래 정리 단계는 이 소유권을 넘어서 실행하지 않는다.


- **이번 생성/재사용/출력:** 신규 독립 resource 없음. 승인 CIDR local artifact + VCN/public-admin의 실제 creation-time 선택값. 단계 A/단계 B로 순환 분리.
- **생성 목표:** Virtual Cloud Network (VCN)의 단일 실제 생성 event에서 CIDR Blocks 옵션을 직접 선택하고 저장·기능 검증한다. 독립 중복 생성은 하지 않는다.
- **계약 보정:** 기존 creation-first 계약의 문서만 존재하는 선행(24/25/26/29 등)과 bootstrap 순환은 실제 절차를 포함한 00/19/23 단계 A로 대체했다. 변경된 typed dependency는 authoring ledger에 기록한다.

## 실습절차

### 명령 실행 gate — 실패하면 다음 생성으로 진행하지 않는다

1. 아래 Bash는 **전용 Bash 세션**에서만 사용한다. Console/Cloud Shell 조회, 본인 로컬 SSH, 로그인한 Oracle Linux guest를 섞지 않는다. `exit 1`은 이 전용 세션을 닫는 안전 중단이다. `<...>`·빈 입력·타 owner OCID가 남으면 실행하지 않는다.
2. 성공을 요구하는 파일 작성/조회/설치/attach 작업은 실패 시 중단한다. 실패한 API 뒤 기존 JSON이나 기존 다운로드 파일을 성공 근거로 읽지 않는다. 기대 deny는 `if command; then ...; else RC=$?; ...; fi`로 별도 기록하고, 실제 timeout/IAM 오류/로컬 파일 오류를 구별한다.
3. 첫 변경 전에 정확한 OCID·owner·현재 설정과 승인 expiry를 저장한다. 공유 자원 변경 중 다른 소비자가 시작하거나 원복값을 확인할 수 없으면 변경을 중단한다. 삭제 성공은 API 수락이 아니라 같은 identity/Region의 정확한 대상 재조회로 판정한다. `NotAuthorizedOrNotFound` 단독은 삭제 증거가 아니다.


### 개별 완주 보정 — 생성·검증·소유권

Python 계산은 VCN_CIDR/PUBLIC_SUBNET_CIDR/PRIVATE_SUBNET_CIDR를 사용한다. 실제연결예정망 모두 입력해 overlap=False를 얻기전생성금지. collision negative는 local계산만하고OCI에충돌subnet을만들지 않는다.

<a id="cidr-planning"></a>

### 단계 A — VCN 제출 전에 주소 계획 계산

1. 승인 후보를 IPv4로 한정하고 아래 local 계산을 수행한다. VCN의 10.77.0.0/16 안에서 public 10.77.10.0/24, private 10.77.20.0/24를 쓰는 예시다. 실제 연결 예정망을 입력하지 않은 채 겹치지 않는다고 판정하지 않는다. 이 단계는 VCN이 없어도 완료 가능하다.

```bash
python3 - <<'PY'
import os
from ipaddress import ip_network,ip_address
v=ip_network(os.environ['VCN_CIDR'])
a,b=map(ip_network,[os.environ['PUBLIC_SUBNET_CIDR'],os.environ['PRIVATE_SUBNET_CIDR']])
assert all(n.version==4 for n in [v,a,b]), "STOP: IPv4 only"
assert a.subnet_of(v) and b.subnet_of(v)
assert not a.overlaps(b)
# 실제 연결 예정망을 모두 입력한다. 연결망이 없는 독립 lab만 조직 확인 후 []를 사용한다.
import json
connected=json.loads(os.environ['CONNECTED_CIDRS_JSON'])
assert isinstance(connected,list)
for cidr in connected:
 other=ip_network(cidr)
 assert other.version==4 and not v.overlaps(other), f'STOP overlap: {cidr}'
 print('connected overlap=', cidr, v.overlaps(other))
for n in [a,b]:
 print(str(n), 'addresses=', n.num_addresses, 'range=', n.network_address, n.broadcast_address)
# 사용자 계획과 무관한 고정 outside 주소를 합격 조건으로 쓰지 않는다.
print('VCN range=', v.network_address, v.broadcast_address)
print('negative-overlap=', a.overlaps(next(a.subnets(prefixlen_diff=1))))
PY
if [ "$?" -ne 0 ]; then printf '%s\n' 'STOP: plan/readback failed; no stage B'; exit 1; fi
```
2. 연결 예정 on-prem/VPN/기존 VCN CIDR 각각을 동일 `ip_network(...).overlaps(v)`로 검사하고 False를 요구한다. CIDR prefix와 network boundary가 맞지 않으면 strict parser 오류를 수정한다. OCI subnet의 첫 두 주소와 마지막 주소 예약 제약은 공식 Networking Overview의 Allowed VCN Size and Address Ranges를 확인한다. 일반 IPv4 host 계산을 OCI 할당 가능 주소 수라고 쓰지 않는다.

### 단계 B — 실제 VCN/subnet 생성값에 반영

3. VCN lab의 **생성 전 CIDR 입력 단계**에서 승인 /16을 제출하고 Public Subnets 단계 A의 실제 public-admin 생성에 /24를 선택한다. 로컬 계산만으로 이 주제 전체를 수료하지 않는다. private CIDR은 Subnets의 private-app 생성에서 같은 계획을 사용한다. 단독 cidr-check subnet을 추가하지 않는다. [VCN CIDR fields](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/create_vcn.htm) · [Subnet containment/non-overlap](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/create_subnet.htm).
4. 아래 readback으로 API의 CIDR와 계획을 비교한다. negative-overlap은 local True로 끝내며 운영망에 겹친 subnet을 의도적으로 생성하지 않는다.

## 결과검증

<a id="baseline-20-ready"></a>

```bash
oci network vcn get --vcn-id "$VCN_ID" --region "$REGION" > cidr-vcn.json || exit 1
oci network subnet get --subnet-id '<public-admin_OCID>' --region "$REGION" > cidr-subnet.json || exit 1
python3 - <<'PY'
import json,ipaddress,os
v=json.load(open('cidr-vcn.json'))['data'];s=json.load(open('cidr-subnet.json'))['data']
n=ipaddress.ip_network(s['cidr-block'])
assert any(n.subnet_of(ipaddress.ip_network(x)) for x in v['cidr-blocks'])
assert s['vcn-id']==v['id']
assert s['cidr-block']==os.environ['PUBLIC_SUBNET_CIDR']
print('CIDR readback assertions passed')
PY
if [ "$?" -ne 0 ]; then printf '%s\n' 'STOP: plan/readback failed; no stage B'; exit 1; fi
```
실제 승인 CIDR를 교체했다면 코드 값도 교체한다. 기대 패턴 containment True/nonoverlap False, negative overlap True, readback assertions 종료 0. 아직 private-app이 없으면 private readback은 후속 생성 때 수행한다.

**증거 기록:** 실제 명령·UTC 시간·exit code·가려진 OCID·옵션 readback·검증 조건을 남긴다. 현재 이 문서의 실행상태는 `NOT_EXECUTED`다. output/creation-event-id/result는 실행 전 비워 둔다. 오류/NOT_SELECTED를 PASS로 바꾸지 않는다.

### 이 시나리오의 functional acceptance

| Positive | Negative / 미완료 판정 | Recovery / 반환 |
|---|---|---|
| strict parse/containment/nonoverlap/readback | local negative-overlap=True; OCI collision 생성 없음 | 겹치는 실제 connected CIDR이면 새 계획 승인을 받고 전체 입력 동시 수정 |

모두 실행 시 확인할 기대 패턴이다. 생성 전 guard negative는 OCI 장애를 주입한 것과 다르며 다른 자원을 추가하지 않는다. 오류 메시지와 실제 exit code가 없는 항목은 NOT_EXECUTED 또는 BLOCKED로 남긴다.

## 문제해결

### 공통 오류의 진단 순서

1. `NotAuthorizedOrNotFound` 또는 목록에 리소스가 없음: 먼저 Region/Compartment/OCID와 identity domain/group을 확인한다. 존재하지 않는 OCID와 권한 부족 모두 같은 오류로 보일 수 있다. 관리자에게 request-id·작업 시각·서비스·가려진 대상 OCID를 제공한다. 비밀키/토큰은 보내지 않는다.
2. `LimitExceeded`/quota/capacity 부족: Billing & Cost Management → Limits, Quotas and Usage와 생성 화면의 지원 shape/AD를 확인한다. 운영 quota 증가나 다른 Region 구독을 임의 요청하지 않는다. 승인 범위 안의 더 작은 compatible shape 또는 다른 AD를 선택하거나 `BLOCKED_CAPACITY`를 기록한다.
3. 장시간 provisioning/예상 밖 state: Work Requests/오류 상세를 읽고 같은 요청을 반복 제출하지 않는다. 만들어진 부분 리소스(boot/IP/attachment 포함)를 먼저 조사한다.
4. 메뉴 이름이 다름: 공식 task body의 동등한 탭(예: Storage 또는 Attached Block Volumes)을 찾는다. 필드가 문서와 다르면 값·스크린샷에서 secrets를 가린 뒤 관리자 또는 Oracle Support에 확인하고 제출을 중단한다. 이 문서는 현재 tenancy의 live Console 화면을 검사한 결과가 아니다.

### 이 주제의 실패 지점

strict IPv4 parser가 ValueError면 host bits/prefix를 확인한다. overlap=True를 CIDR가 잘 맞았다는 신호로 읽지 않는다. 기존 CIDR 변경은 주소 계획 파일 수정과 다른 별도 승인 작업이다.

## 리소스정리

승인 세 CIDR/overlap 결과; 별도 subnet 없음, 삭제는 owner19/21/23. 공유 기반 리소스는 이 consumer가 직접 삭제하지 않는다. 실제 소비자/expiry를 creator에게 반환하고 creator의 리소스정리 절차를 사용한다. VM을 Stop했다고 boot/IP/backup 비용이 끝난 것으로 처리하지 않는다. 로컬/guest 파일은 자신이 만든 정확한 경로만 제거하며 기록된 증거를 보존한다.


## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 아래 본문은 각 생성 필드·설정·검증·정리의 기술 근거이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [30] [Networking Overview — CIDR Blocks](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/overview.htm)
  - 확인 범위: CIDR NOTATION.

### 추가 실행 절차 근거 (본문 확인)

- [vcn](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/create_vcn.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [subnet](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/create_subnet.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [cloudshell](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/cloudshellgettingstarted.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
