# Availability Domains

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../README.md)를 확인하세요.

> 실행 절차 작성본 / OCI 미실행. Oracle 공식 본문 근거와 교육용 선택을 구분한다. secrets·실제 고객값·diagram 없음.

## 학습목표

- 현재 tenancy/Region의 실제 AD 목록에서 vm-a의 placement를 선택하고 owner10의 단일 launch에 반영한다.
- instance와 boot volume의 AD 일치 및 Regional subnet의 AD=null을 조회하고 같은 vm-a의 새 SSH를 확인한다.

위 항목은 사용자 실행 후 확인할 완료 조건이며 현재 관측 결과가 아니다. 개념은 아래 해당 생성 필드·configuration·test 단계에서 함께 적용한다.

## 대응 Roadmap 항목

- 항목명: **Availability Domains**
- Roadmap 경로: **Introduction to OCI → Availability Domains**
- Node ID: `sequence-1-2`
- 상위 항목: **Introduction to OCI** (`sequence-1-0`)
- 문서 범위: 이 항목의 생성·선택·기능검증·정리. 실습 미실행; 아래 기대 패턴은 실제 출력 아님.
- 번호는 navigation 식별자이며 실행 순서는 사전준비의 단계별 의존성에 따른다.

## 사전준비

### 기준 계약과 실제 단계 선행조건

[공통 baseline-contract.json](../../baseline-contract.json)의 Oracle Linux 9 / opc / CIDR / 비용 상한을 따른다. 문서 번호는 실행 순서가 아니다. **준비 station과 생성 이후 ready를 분리**한다. 아래 단계 외의 기능완료를 소급하여 요구하지 않는다.

- [00 Introduction — 단계 링크](../00_Introduction/00_Introduction.md#baseline-00-ready): **tenancy/REGION; AD 조회 허용**. 실제 출력/소비자/expiry를 전달하고 해당 owner가 이 lab 종료까지 보존한다.
- [10 Compute — 단계 링크](../02_Compute/00_Compute.md#shared-launch): **공동 launch station; RUNNING은 선택 이후**. 실제 출력/소비자/expiry를 전달하고 해당 owner가 이 lab 종료까지 보존한다.

생성 전 조직 승인(identity/권한/예산/expiry/서비스 지원/limit/capacity)을 확인한다. 실제 OCID/IP는 아직 생성하지 않았으면 비워 두고 해당 Create/readback 뒤 채운다. 동일 creation-event-id/OCID는 한 번의 제출만 뜻한다. `INSTANCE_A_ID`는 owner10 vm-a의 OCID, `INSTANCE_B_ID`는 owner22 private-vm의 OCID이며 owner07 vm-b와 다르다. 실습에 없는 자원을 추가하지 않는다. 실제 checkpoint는 사용자 실행 전 비워 둔다.

### 입력값과 실행 위치

Console에서 선택한 값만 사용한다. `<...>`는 예시 OCID나 비밀값이 아니라 **사용자가 대체할 입력 자리**다. 대체하지 않은 채 명령을 실행하지 않는다. Console → 우측 상단 Cloud Shell/Code Editor → Cloud Shell을 열면 OCI CLI가 사전 인증된다. Region 변경 뒤에는 Shell을 다시 열거나 명령의 `--region`을 바꾼다. SSH는 Cloud Shell이 아니라 본인 Linux/WSL/macOS 터미널에서 수행한다. 공인 IP/32도 그 터미널의 출구 주소여야 한다.

```bash
# 아래는 입력/출력의 대입 형식이다. 표의 단계 전에 생성 후 ID/IP export 행을 실행하지 않는다.
 # 생성 후 검증용 ID/IP는 해당 creator의 실제 응답에서 넣는다.
export REGION='<actual_REGION>'
export TENANCY_ID='<actual_TENANCY_ID>'
export INSTANCE_A_ID='<actual_INSTANCE_A_ID>'
export PUBLIC_SUBNET_ID='<actual_PUBLIC_SUBNET_ID>'
export AD='<actual_AD>'
export BOOT_VOLUME_ID='<actual_BOOT_VOLUME_ID>'
export INSTANCE_ID="$INSTANCE_A_ID"
export SUBNET_ID="$PUBLIC_SUBNET_ID"
oci --version
```

이 변수 중 해당 절차에서 사용하는 값만 채운다. CLI 조회의 `data`를 증거 파일에 저장하고 실제 시간·작업 OCID·선택값을 기록한다. API key/private key/passphrase/인증 토큰은 기록하지 않는다. Cloud Shell 설정파일이나 세션 토큰을 출력하지 않는다. `oci --version`을 실행하는 주제의 기대 패턴은 버전 문자열뿐이다. namespace는 Object Storage 주제에서 별도 조회하며 다른 주제에는 요구하지 않는다. [Cloud Shell 공식 본문](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/cloudshellgettingstarted.htm).

### Workshop 입력 계약 — 단계와 host

| 입력/출력 | 값 또는 실제 획득 경로 | 사용할 시점 | 실행 위치 | 선택 이유 |
|---|---|---|---|---|
| Console/설정 필드 | 실제 tenancy AD 이름; owner10 same launch; subnet=Regional/public-admin; boot same AD | 생성 전 선택 → 생성 뒤 readback | 본문 Console/로컬 지정 | 이 topic의 최소 생성 범위 |
| `SUBNET_ID` | 본문 실제 Create/readback 출력 | 생성 후/선행 완료 | 본문 지정 host | 해당 topic 대상만 지정 |
| `BOOT_VOLUME_ID` | 대상 instance creator Console Storage의 연결 boot OCID | 생성 후 | Cloud Shell | 같은 AD/boot 소유권 |
| `AD` | 현재 tenancy/REGION의 ListAvailabilityDomains 응답 data[].name에서 선택 | Create 이전 ad-selection | Cloud Shell | 실제 placement 후보; 생성 뒤 instance/boot AD와 대조 |
| `INSTANCE_A_ID` | owner10 실제 vm-a 생성 id | 선행 완료/생성 후 | Cloud Shell | 공유 A 조회; 이름은 OCID가 아님 |
| `INSTANCE_ID` | owner10 vm-a Create/readback id | 생성 후/선행 완료 | Cloud Shell | 현재 문서 대상 OCID에 명시 대입 |
| `LAB_COMPARTMENT_ID` | owner00 Create 응답 OCID; topic00에서는 생성 뒤 | 선행 완료/생성 후 | Console/Cloud Shell | 정책/자원 scope |
| `PUBLIC_SUBNET_ID` | owner23 단계 A subnet Create id | 선행 완료/생성 후 | Console/Cloud Shell | public-admin 선택 |
| `REGION` | 승인된 기존 subscribed Region identifier | 생성 전 | Console/Cloud Shell | API endpoint/데이터 위치 |
| `TENANCY_ID` | Console tenancy details의 실제 OCID | 생성 전 | Cloud Shell | AD/구독 조회 |
| `VNIC_ID` | 해당 INSTANCE_ID의 vnic-attachments.json data[].vnic-id; primary VNIC 확인 | 생성 후 | Cloud Shell | 다른 VM VNIC 혼동 방지 |

생성 전에는 행의 승인값/선행 완료값만 지정하고, 생성 후 ID/IP는 해당 owner 응답으로 채운다. Cloud Shell과 로컬 SSH의 환경변수는 공유되지 않는다. 로컬에는 KEY와 접속 주소만 다시 지정하고 guest disk 명령은 로그인한 guest에서만 실행한다. `<...>`가 남으면 해당 명령은 실행하지 않는다.

## 권한

### 실제 task/API별 공식 권한 본문 대조

아래는 이 문서 생성·조회·변경·정리 API의 공식 표 본문을 읽은 인용이다. 위 family 예제는 편의상 넓은 compartment-scope grant이며 **최소권한 정책이라고 부르지 않는다**. verb/dependency 행 인용은 단일 API의 완전한 최소권한 도출이 아니다. 조직 Identity Domain Console user/group 작업은 해당 Domain 관리자 역할도 필요하며 legacy IAM user API 권한만으로 대체하지 않는다.

- `GetBootVolume`: VOLUME_INSPECT — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `GetInstance`: INSTANCE_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `GetSubnet`: SUBNET_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `GetVnic`: VNIC_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `ListAvailabilityDomains`: COMPARTMENT_INSPECT — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/policyreference/iampolicyreference.htm).
- `ListVnicAttachments`: VNIC_ATTACHMENT_READ and INSTANCE_INSPECT — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).

**Boot attachment 조회 범위:** 이 문서는 미확인 `ListBootVolumeAttachments` 최소권한을 요구하는 CLI를 실행하지 않는다. 생성 owner가 승인된 Compute Console → 정확한 instance OCID → Storage → Boot volume에서 연결된 boot OCID/AD와 instance를 읽어 전달한다. boot를 검증하는 주제만 위 GetBootVolume 권한을 승인받아 그 OCID를 조회한다. Console attachment를 볼 권한이 없으면 creator가 exact instance/boot 관계를 제공하기 전 중단한다. 권한 표에 없는 operation을 최소권한 검증 완료로 표시하지 않는다.


이번 consumer는 선행 creator의 리소스를 다시 생성·종료하지 않는다. 위 Get/List operation의 읽기 권한만 별도 승인한다. 공동 launch/VCN 생성 station에 참여하는 경우에만 해당 [creator 사전준비](#사전준비)의 Create/attachment 권한을 관리자와 확인한다. readback 때문에 `manage instance-family`, `manage volume-family`, `manage virtual-network-family` 세 가족을 일괄 요청하지 않는다. OS 조회는 `opc`/필요한 sudo, 로컬 파일 작성은 OCI IAM과 별개다.

## 비용및주의사항

- AD 자체는 생성·삭제하는 자원이 아니다. owner10 vm-a/boot의 **같은 launch 1회**에 placement를 선택하고 조회/새 SSH만 추가한다. 두 번째 AD 비교 VM은 기본 범위 밖이며 별도 수량·shape·boot·expiry 승인이 없으면 실행하지 않는다. 단일 AD Region에서 존재하지 않는 AD를 만들거나 다른 tenancy AD 문자열을 강제하지 않는다.
- 생성/사용 당일 공식 단가와 Billing & Cost Management → Cost Analysis의 실제 Compartment/서비스 필터를 확인하되 청구 반영 지연을 기록한다. 예산 경보는 과금 차단이 아니다. 고객 데이터/secret/운영 자원을 사용하지 않는다.
- 이 문서는 Oracle 기술 본문 기반 교육 절차다. OCI/guest/IAM allow·deny 실측은 없으며 기대 결과를 실행 결과로 기록하지 않는다. 승인되지 않은 선택 확장은 실행하지 않는다.


## 실습구성

### 이번 Workshop에서 만드는 것과 진행 순서

계정별 실제 AD 이름을 선택하여 vm-a와 boot-a를 같은 AD에 배치한다. Regional subnet은 AD 아래 놓지 않는다.

1. **진입/선택:** ListAvailabilityDomains. 사전준비 링크의 지정 단계/실제 출력이 없으면 제출하지 않는다.
2. **실제 작업:** launch AD 선택 → owner10 단일 Create. 아래 Console 필드/명령을 해당 host에서 순서대로 실행한다. 같은 자원은 이름이 아니라 OCID/event로 재사용한다.
3. **완료/복구/반환:** instance/boot AD 및 subnet Regional 비교. 결과검증 실패는 문제해결로 이동하고 동일 Create를 무작정 재제출하지 않는다.

**전달/정리 범위:** 선택 AD/boot OCID/같은 launch event; 독립 AD 생성/삭제 없음. 아래 정리 단계는 이 소유권을 넘어서 실행하지 않는다.


- **이번 생성/재사용/출력:** vm-a launch에 AD placement 선택. 독립 AD resource 없음. boot-a 같은 AD/readback 출력.
- **생성 목표:** Compute의 단일 실제 생성 event에서 Availability Domains 옵션을 직접 선택하고 저장·기능 검증한다. 독립 중복 생성은 하지 않는다.
- **완료 증거:** 생성 전 옵션 선택 + 실제 제출 대상 OCID + API readback + 해당 기능검증 + dependency 순서 정리. 실제 OCI 실행 checkpoint는 비워 둔다.
- **retention:** 예약 소비자와 expiry가 있는 resource만 보존. idle 유료 resource를 다음 주제 때문에 무기한 유지하지 않는다.


## 실습절차

### 명령 실행 gate — 실패하면 다음 생성으로 진행하지 않는다

1. 아래 Bash는 **전용 Bash 세션**에서만 사용한다. Console/Cloud Shell 조회, 본인 로컬 SSH, 로그인한 Oracle Linux guest를 섞지 않는다. `exit 1`은 이 전용 세션을 닫는 안전 중단이다. `<...>`·빈 입력·타 owner OCID가 남으면 실행하지 않는다.
2. 성공을 요구하는 파일 작성/조회/설치/attach 작업은 실패 시 중단한다. 실패한 API 뒤 기존 JSON이나 기존 다운로드 파일을 성공 근거로 읽지 않는다. 기대 deny는 `if command; then ...; else RC=$?; ...; fi`로 별도 기록하고, 실제 timeout/IAM 오류/로컬 파일 오류를 구별한다.
3. 첫 변경 전에 정확한 OCID·owner·현재 설정과 승인 expiry를 저장한다. 공유 자원 변경 중 다른 소비자가 시작하거나 원복값을 확인할 수 없으면 변경을 중단한다. 삭제 성공은 API 수락이 아니라 같은 identity/Region의 정확한 대상 재조회로 판정한다. `NotAuthorizedOrNotFound` 단독은 삭제 증거가 아니다.


### 개별 완주 보정 — 생성·검증·소유권

ad-selection에서 실제 ListAvailabilityDomains 출력만 선택한다. boot attachment의 minimum-permission 대응은 수집 core표에 없으므로 scoped family 예제를 최소정책으로 주장하지 않는다. 같은 launch 뒤 SSH/boot AD를 대조한다.

<a id="ad-selection"></a>

1. OCI tenancy에서 AD 이름은 계정별 prefix를 포함하므로 다른 예제의 AD 문자열을 복사하지 않는다. Cloud Shell에서 `oci iam availability-domain list --compartment-id "$TENANCY_ID" --region "$REGION"`를 실행해 목록을 기록한다. [AD와 tenancy-specific names](https://docs.oracle.com/en-us/iaas/Content/General/Concepts/regions.htm).
**Launch 이전 Cloud Shell membership station:** 새 owner06 전용 디렉터리를 예약하고 실제 tenancy/Region 목록을 성공 저장한 뒤 그 `data[].name`에서 승인된 정확한 이름을 AD로 지정한다. 실패한 조회 파일/다른 tenancy 예제를 사용하지 않는다.

```bash
set -euo pipefail
umask 077
AD_WORK=$(mktemp -d "$HOME/owner06-ad.XXXXXXXX") || exit 1
cd "$AD_WORK" || exit 1
oci iam availability-domain list --compartment-id "$TENANCY_ID" --region "$REGION" > ad-candidates.json || exit 1
export AD='<위_data[].name에서_승인한_정확한_이름>'
python3 - <<'PY'
import json,os
rows=json.load(open('ad-candidates.json'))['data']
names={x['name'] for x in rows}
assert os.environ['AD'] in names, 'BLOCKED_AD: do not submit launch'
assert 'invalid-owner06-ad' not in names, 'choose a distinct invalid guard fixture'
# negative control is local membership rejection, not a failing OCI launch.
assert not ('invalid-owner06-ad' in names)
print('selected AD membership confirmed; invalid input rejected locally')
PY
```

2. membership 검증 후 Compute의 생성 전 Placement station에 참여한다. 실제 목록에서 AD를 고르고 compatible image/작은 shape/기존 Regional public-admin을 함께 선택하여 **같은 vm-a launch를 제출**한다. AD resource를 새로 생성하는 버튼은 없다. [Creating an Instance — Availability domain](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/launchinginstance.htm).
3. RUNNING 이후 instance/boot attachment/boot volume의 AD를 대조한다. boot volume의 AD가 instance와 같은지 확인한다. Regional subnet은 특정 AD field가 null이라 여러 AD의 리소스를 수용할 수 있다. AD-specific subnet을 임의 만들지 않는다.
4. 다른 AD를 비교하는 VM은 선택 비용 승인이 있을 때만 생성한다. 단일 AD Region에서 두 번째 AD를 만들어낼 수 없으며 전체 Region간 독립성과 AD 장애 격리를 혼동하지 않는다.

### Create 이후 값 전달 — 이름으로 추정하지 않는다

Cloud Shell의 INSTANCE_ID는 해당 creator Create 응답 id로 지정한다. Console → Compute → Instances → 정확한 instance → Networking/Attached VNICs에서 primary VNIC OCID를 기록하고 아래 GetVnic의 `is-primary=true`, `subnet-id`, `private-ip`, `public-ip`를 대조한다. Storage/Boot volume의 연결 boot OCID·AD도 creator가 기록한다. Cloud Shell과 로컬 터미널의 변수는 공유되지 않으므로 SSH 전에 KEY와 **그 instance의** 실제 접속 주소를 로컬에 다시 지정한다. 이전 extra VM 주소를 재사용하지 않는다. 모든 JSON 조회는 실패하면 즉시 중단하고 기존 파일을 읽지 않는다.

## 결과검증

<a id="baseline-06-ready"></a>

```bash
oci compute instance get --instance-id "$INSTANCE_ID" --region "$REGION" > instance-readback.json || exit 1
oci compute vnic-attachment list --compartment-id "$LAB_COMPARTMENT_ID" --instance-id "$INSTANCE_ID" --region "$REGION" > vnic-attachments.json || exit 1
# Cloud Shell: 현재 instance에 붙은 ATTACHED VNIC만 각각 조회한다.
export VNIC_ID='<현재_INSTANCE_ID에_ATTACHED이며_GetVnic_is-primary=true인_OCID>'
case "$VNIC_ID" in ocid1.vnic.*) ;; *) printf '%s\n' 'STOP: actual primary VNIC OCID required'; exit 1;; esac
oci network vnic get --vnic-id "$VNIC_ID" --region "$REGION" > vnic-readback.json || exit 1
```
기대 패턴: instance `lifecycle-state=RUNNING`, `availability-domain`/`fault-domain`/`shape`가 제출값과 같음, VNIC `subnet-id`가 선택값과 같음, public VM은 `public-ip`가 문자열이고 private VM은 null이다. creator Console Storage의 boot 연결이 대상 instance와 boot OCID를 연결한다. 파일을 실제로 읽어 조건을 대조한다.
```bash
oci bv boot-volume get --boot-volume-id '<creator_Console_Storage의_boot-volume-id>' --region "$REGION" > ad-boot.json || exit 1
oci network subnet get --subnet-id '<public-admin_OCID>' --region "$REGION" > ad-subnet.json || exit 1
```
Cloud Shell에서 위 파일을 실제 assertion한다:

```bash
python3 - <<'PY'
import json,os
i=json.load(open('instance-readback.json'))['data']
b=json.load(open('ad-boot.json'))['data'];s=json.load(open('ad-subnet.json'))['data']
assert i['id']==os.environ['INSTANCE_A_ID']
assert b['id']==os.environ['BOOT_VOLUME_ID']
assert s['id']==os.environ['PUBLIC_SUBNET_ID']
assert i['availability-domain']==b['availability-domain']==os.environ['AD']
assert s['availability-domain'] is None
PY
```

기대 패턴 instance.availability-domain == boot.availability-domain == 승인 선택 AD; Regional subnet.availability-domain=null. Compute의 같은 vm-a OCID를 재사용하되 이번 checkpoint는 새 local SSH 세션을 요구한다. 첫 SSH 전 [owner10 신뢰 bootstrap](../02_Compute/00_Compute.md#guest-trust-bootstrap)을 같은 vm-a OCID로 완료한다. API 조회 뒤 본인 로컬 전용 Bash에서 아래 새 연결을 수행한다. 성공 stdout/종료 UTC를 consumer06 record에 연결하며 이전 SSH 결과만 재사용하지 않는다.

```bash
export KEY='<owner15_actual_local_KEY>' PUBLIC_IP='<owner10_actual_vm-a_primary_VNIC_public_IP>'
timeout --foreground 60s ssh -i "$KEY" -o BatchMode=yes -o IdentitiesOnly=yes -o HostKeyAlgorithms=ssh-ed25519 -o StrictHostKeyChecking=yes -o ConnectTimeout=10 -o ConnectionAttempts=1 -o ControlMaster=no -o ControlPath=none opc@"$PUBLIC_IP" 'test "$(id -un)" = opc && hostname' > consumer06-ssh.txt || exit 1
date -u +%FT%TZ > consumer06-session-ended.txt || exit 1
```

unknown host/identity 불일치/timeout는 BLOCKED_TRUST 또는 실제 접속 실패이며 성공 UTC 대신 실패 종료 record를 creator에게 전달한다.

**증거 기록:** 실제 명령·UTC 시간·exit code·가려진 OCID·옵션 readback·검증 조건을 남긴다. 현재 이 문서의 실행상태는 `NOT_EXECUTED`다. output/creation-event-id/result는 실행 전 비워 둔다. 오류/NOT_SELECTED를 PASS로 바꾸지 않는다.

### 이 시나리오의 functional acceptance

| Positive | Negative / 미완료 판정 | Recovery / 반환 |
|---|---|---|
| instance AD=boot AD; subnet AD null | 실제 AD 목록에 없는 입력은 launch 전 membership 검토에서 중단 | 현재 tenancy AD 이름을 선택하여 같은 launch 진행; fake AD resource 없음 |

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

AD 문자열이 invalid면 현재 tenancy ListAvailabilityDomains 결과를 다시 읽는다. 다른 tenancy의 prefix/다른 Region AD를 복사하지 않는다. Boot와 instance AD 불일치는 다른 OCID를 조회했는지 먼저 확인한다.

## 리소스정리

AD 자체는 생성/삭제하지 않는다. owner10 registry에 `consumer=06`, REGION, actual INSTANCE_A_ID/BOOT_VOLUME_ID, 선택 AD, ad-candidates.json 및 instance/boot/subnet 대조 증거 경로/UTC, 새 local SSH exit 및 session-end UTC, `outstanding_changes=none`, `lease=RELEASED` 또는 미완료 사유/승인 retention deadline, owner10 acknowledgment UTC/alias를 남긴다. SSH 실패를 완료로 바꾸지 않으며 열린 세션 종료 후만 반환한다. [owner10 리소스정리](../02_Compute/00_Compute.md#리소스정리)로 넘기고 공유 vm-a/boot를 직접 Stop/Terminate하지 않는다. 승인 증거 보존 뒤 owner06 AD_WORK에서 이번에 만든 exact JSON 파일만 제거하고 비었을 때 rmdir; creator 파일/KEY는 보존한다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 아래 본문은 각 생성 필드·설정·검증·정리의 기술 근거이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [23] [Regions and Availability Domains — Availability Domains](https://docs.oracle.com/en-us/iaas/Content/General/Concepts/regions.htm)
  - 확인 범위: About Regions and Availability Domains Oracle / Availability domains are isolated.

### 추가 실행 절차 근거 (본문 확인)

- [regions](https://docs.oracle.com/en-us/iaas/Content/General/Concepts/regions.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [launch](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/launchinginstance.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [boot](https://docs.oracle.com/en-us/iaas/Content/Block/Concepts/bootvolumes.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [cloudshell](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/cloudshellgettingstarted.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
