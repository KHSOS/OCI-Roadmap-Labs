> **기존 실습 초안 — 미검증**: 원래 93개 문서의 절차를 보존한 자료입니다. 새 튜토리얼의 품질 승인이나 OCI 실행 검증을 뜻하지 않습니다.

# IaaS / PaaS / SaaS

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../../../Docs/README.md)를 확인하세요.

> 실행 절차 작성본 / OCI 미실행. Oracle 공식 본문 근거와 교육용 선택을 구분한다. secrets·실제 고객값·diagram 없음.

## 학습목표

- 기존 vm-a의 SSH/guest 관리 작업과 같은 VCN의 빈 Functions application lab-platform 생성·ACTIVE·subnet/shape 조회를 각각 완료한다.
- function 목록이 empty임을 확인하고 application만 삭제한다. 함수 invocation·SaaS 배포는 수행 범위와 결과에서 제외한다.

위 항목은 사용자 실행 후 확인할 완료 조건이며 현재 관측 결과가 아니다. 개념은 아래 해당 생성 필드·configuration·test 단계에서 함께 적용한다.

## 대응 Roadmap 항목

- 항목명: **IaaS / PaaS / SaaS**
- Roadmap 경로: **Introduction → IaaS / PaaS / SaaS**
- Node ID: `sequence-0-2`
- 상위 항목: **Introduction** (`sequence-0-0`)
- 문서 범위: 이 항목의 생성·선택·기능검증·정리. 실습 미실행; 아래 기대 패턴은 실제 출력 아님.
- 번호는 navigation 식별자이며 실행 순서는 사전준비의 단계별 의존성에 따른다.

## 사전준비

### 기준 계약과 실제 단계 선행조건

[공통 baseline-contract.json](../../../baseline-contract.json)의 Oracle Linux 9 / opc / CIDR / 비용 상한을 따른다. 문서 번호는 실행 순서가 아니다. **준비 station과 생성 이후 ready를 분리**한다. 아래 단계 외의 기능완료를 소급하여 요구하지 않는다.

- [10 Compute — 단계 링크](../02_Compute/00_Compute.md#baseline-10-ready): **IaaS vm-a RUNNING/SSH 증거**. 실제 출력/소비자/expiry를 전달하고 해당 owner가 이 lab 종료까지 보존한다.
- [owner10 guest trust bootstrap](../02_Compute/00_Compute.md#guest-trust-bootstrap): **같은 vm-a OCID/public IP/serial ED25519/known_hosts 인계**, owner15 local KEY를 보존한다. 새 guest inventory 전 완료한다.
- [23 Public Subnets — 단계 링크](../../Networking/Original-Provisional/03_Virtual_Cloud_Network_VCN/04_Public_Subnets.md#public-network-preparation): **단계 A 완료: public-admin/IGW/NSG/SL OCID**. 실제 출력/소비자/expiry를 전달하고 해당 owner가 이 lab 종료까지 보존한다.

생성 전 조직 승인(identity/권한/예산/expiry/서비스 지원/limit/capacity)을 확인한다. 실제 OCID/IP는 아직 생성하지 않았으면 비워 두고 해당 Create/readback 뒤 채운다. 동일 creation-event-id/OCID는 한 번의 제출만 뜻한다. `INSTANCE_A_ID`는 owner10 vm-a의 OCID, `INSTANCE_B_ID`는 owner22 private-vm의 OCID이며 owner07 vm-b와 다르다. 실습에 없는 자원을 추가하지 않는다. 실제 checkpoint는 사용자 실행 전 비워 둔다.

### 입력값과 실행 위치

Console에서 선택한 값만 사용한다. `<...>`는 예시 OCID나 비밀값이 아니라 **사용자가 대체할 입력 자리**다. 대체하지 않은 채 명령을 실행하지 않는다. Console → 우측 상단 Cloud Shell/Code Editor → Cloud Shell을 열면 OCI CLI가 사전 인증된다. Region 변경 뒤에는 Shell을 다시 열거나 명령의 `--region`을 바꾼다. SSH는 Cloud Shell이 아니라 본인 Linux/WSL/macOS 터미널에서 수행한다. 공인 IP/32도 그 터미널의 출구 주소여야 한다.

```bash
# 아래는 입력/출력의 대입 형식이다. 표의 단계 전에 생성 후 ID/IP export 행을 실행하지 않는다.
 # 생성 후 검증용 ID/IP는 해당 creator의 실제 응답에서 넣는다.
export REGION='<actual_REGION>'
# APPLICATION_ID는 아래 Create 응답을 받은 뒤 한 번만 지정한다.
oci --version
```

이 변수 중 해당 절차에서 사용하는 값만 채운다. CLI 조회의 `data`를 증거 파일에 저장하고 실제 시간·작업 OCID·선택값을 기록한다. API key/private key/passphrase/인증 토큰은 기록하지 않는다. Cloud Shell 설정파일이나 세션 토큰을 출력하지 않는다. `oci --version`을 실행하는 주제의 기대 패턴은 버전 문자열뿐이다. namespace는 Object Storage 주제에서 별도 조회하며 다른 주제에는 요구하지 않는다. [Cloud Shell 공식 본문](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/cloudshellgettingstarted.htm).

### Workshop 입력 계약 — 단계와 host

| 입력/출력 | 값 또는 실제 획득 경로 | 사용할 시점 | 실행 위치 | 선택 이유 |
|---|---|---|---|---|
| `APPLICATION_ID` | Console lab-platform Create 실제 OCID | 생성 후 | Cloud Shell | 빈 application 조회/삭제 |
| Console/설정 필드 | Name=lab-platform; VCN=shared-vcn; Subnets=[public-admin]; Shape=Generic_X86; Functions=[] | 생성 전 선택 → 생성 뒤 readback | 본문 Console/로컬 지정 | 이 topic의 최소 생성 범위 |
| `REGION` | 승인된 기존 subscribed Region identifier | 생성 전 | Console/Cloud Shell | API endpoint/데이터 위치 |

생성 전에는 행의 승인값/선행 완료값만 지정하고, 생성 후 ID/IP는 해당 owner 응답으로 채운다. Cloud Shell과 로컬 SSH의 환경변수는 공유되지 않는다. 로컬에는 KEY와 접속 주소만 다시 지정하고 guest disk 명령은 로그인한 guest에서만 실행한다. `<...>`가 남으면 해당 명령은 실행하지 않는다.

## 권한

### 실제 task/API별 공식 권한 본문 대조

아래는 이 문서 생성·조회·변경·정리 API의 공식 표 본문을 읽은 인용이다. 위 family 예제는 편의상 넓은 compartment-scope grant이며 **최소권한 정책이라고 부르지 않는다**. verb/dependency 행 인용은 단일 API의 완전한 최소권한 도출이 아니다. 조직 Identity Domain Console user/group 작업은 해당 Domain 관리자 역할도 필요하며 legacy IAM user API 권한만으로 대체하지 않는다.

- `CreateApp`: FN_APP_CREATE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/functionspolicyreference.htm).
- `DeleteApp`: FN_APP_DELETE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/functionspolicyreference.htm).
- `GetApp`: FN_APP_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/functionspolicyreference.htm).
- ListApp(OCI CLI application list / API ListApplications)=FN_APP_LIST는 최종 app 전체목록 조회에 필요하다. [Functions operation 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/functionspolicyreference.htm).
- `ListFunctions`: FN_FUNCTION_LIST — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/functionspolicyreference.htm).

owner02는 공유 VM/network를 다시 생성·종료하지 않지만 **자신의 새 Functions application의 create/read/list/delete** 권한을 승인받는다. app list FN_APP_LIST도 cleanup에서 필요하다. 공동 launch/VCN 생성 station에 참여하는 경우에만 해당 [creator 사전준비](#사전준비)의 Create/attachment 권한을 관리자와 확인한다. readback 때문에 `manage instance-family`, `manage volume-family`, `manage virtual-network-family` 세 가족을 일괄 요청하지 않는다. OS 조회는 `opc`/필요한 sudo, 로컬 파일 작성은 OCI IAM과 별개다.

관리자는 application create/read/delete 권한을 추가한다. `Allow group <domain>/<lab-group> to manage fn-app in compartment lab-root`와 public-admin subnet 사용권한을 Functions required policy 본문과 대조한다. 결과검증의 empty function 목록 조회에는 `Allow group <domain>/<lab-group> to inspect fn-function in compartment lab-root`도 추가한다. [Functions resource/API verb table](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/functionspolicyreference.htm)와 [network/service policy prerequisite body](https://docs.oracle.com/en-us/iaas/Content/Functions/Tasks/functionscreatingpolicies.htm)를 확인한다. 함수 create/invoke/registry push 권한은 이 lab에 필요하지 않다. Functions service/network 관련 tenancy prerequisite는 관리자가 사전 확인해야 하며 allow/deny 실행은 미검증이다.

## 비용및주의사항

- 신규 범위는 빈 Functions application **1개**, 기존 vm-a/public-admin 재사용이다. 함수/code/image/OCIR push/invoke/SaaS 구매는 **0개**이므로 함수 실행 비용 실험을 주장하지 않는다. 관리자가 해당 Region의 application/network 조건과 보존 deadline을 승인하고 app 빈 목록 확인 뒤 즉시 삭제한다. 기존 VM/boot 비용은 owner10 lease와 별개로 계속되므로 consumer02 종료를 반환한다.
- 생성/사용 당일 공식 단가와 Billing & Cost Management → Cost Analysis의 실제 Compartment/서비스 필터를 확인하되 청구 반영 지연을 기록한다. 예산 경보는 과금 차단이 아니다. 고객 데이터/secret/운영 자원을 사용하지 않는다.
- 이 문서는 Oracle 기술 본문 기반 교육 절차다. OCI/guest/IAM allow·deny 실측은 없으며 기대 결과를 실행 결과로 기록하지 않는다. 승인되지 않은 선택 확장은 실행하지 않는다.


## 실습구성

### 이번 Workshop에서 만드는 것과 진행 순서

공유 IaaS VM과 빈 Functions application을 비교한다. 함수 배포/실행, OCIR image, SaaS 구독은 만들지 않는다.

1. **진입/선택:** vm-a 생성 책임표. 사전준비 링크의 지정 단계/실제 출력이 없으면 제출하지 않는다.
2. **실제 작업:** public-network 단계 A 확인 → lab-platform GENERIC_X86/public-admin 하나 선택 → Create/Get/ListFunctions. 아래 Console 필드/명령을 해당 host에서 순서대로 실행한다. 같은 자원은 이름이 아니라 OCID/event로 재사용한다.
3. **완료/복구/반환:** empty 확인. 결과검증 실패는 문제해결로 이동하고 동일 Create를 무작정 재제출하지 않는다.

**전달/정리 범위:** application OCID와 subnet-ids/shape; owner02가 빈 application 삭제, network/VM은 owner 유지. 아래 정리 단계는 이 소유권을 넘어서 실행하지 않는다.


- **이번 생성/재사용/출력:** 신규 Functions application 1개, IaaS vm-a 공동 event 재사용. 함수/OCIR image/새 SaaS 구독 없음.
- **생성 목표:** 기본은 공유 IaaS vm-a 생성과 최소 Functions application 생성(함수 배포·invoke 없음)에서 책임 경계를 선택하면서 학습한다. 별도 SaaS sandbox record 생성은 기존 구독이 있고 허가된 경우의 명시적 선택 확장이며, 새 SaaS 구독 구매를 기본에서 요구하지 않는다.
- **완료 증거:** 생성 전 옵션 선택 + 실제 제출 대상 OCID + API readback + 해당 기능검증 + dependency 순서 정리. 실제 OCI 실행 checkpoint는 비워 둔다.
- **retention:** 예약 소비자와 expiry가 있는 resource만 보존. idle 유료 resource를 다음 주제 때문에 무기한 유지하지 않는다.


## 실습절차

### 명령 실행 gate — 실패하면 다음 생성으로 진행하지 않는다

1. 아래 Bash는 **전용 Bash 세션**에서만 사용한다. Console/Cloud Shell 조회, 본인 로컬 SSH, 로그인한 Oracle Linux guest를 섞지 않는다. `exit 1`은 이 전용 세션을 닫는 안전 중단이다. `<...>`·빈 입력·타 owner OCID가 남으면 실행하지 않는다.
2. 성공을 요구하는 파일 작성/조회/설치/attach 작업은 실패 시 중단한다. 실패한 API 뒤 기존 JSON이나 기존 다운로드 파일을 성공 근거로 읽지 않는다. 기대 deny는 `if command; then ...; else RC=$?; ...; fi`로 별도 기록하고, 실제 timeout/IAM 오류/로컬 파일 오류를 구별한다.
3. 첫 변경 전에 정확한 OCID·owner·현재 설정과 승인 expiry를 저장한다. 공유 자원 변경 중 다른 소비자가 시작하거나 원복값을 확인할 수 없으면 변경을 중단한다. 삭제 성공은 API 수락이 아니라 같은 identity/Region의 정확한 대상 재조회로 판정한다. `NotAuthorizedOrNotFound` 단독은 삭제 증거가 아니다.


### 개별 완주 보정 — 생성·검증·소유권

App Create/Get/Delete와 ListFunctions의 FN_APP/FN_FUNCTION_LIST를 분리한다. App ACTIVE/GENERIC_X86/subnet-id 및 빈 함수 목록만 검증한다. SaaS는 제품·sandbox 입력 부재로 NOT_SELECTED이며 실제 생성 완료가 아니다.

1. Compute 생성 event에서 vm-a image/OS/shape/key/patch responsibility를 직접 선택한다. 이어서 **본인 Linux/WSL 터미널**에서 [owner10 trust bootstrap](../02_Compute/00_Compute.md#guest-trust-bootstrap)의 같은 A OCID/public IP/ED25519 인계를 확인하고 read-only guest inventory를 새 연결로 수행한다. local KEY는 owner15 전용 키 절대경로이며 Cloud Shell에 전송하지 않는다.

```bash
export KEY='<owner15_actual_local_KEY>' PUBLIC_IP='<owner10_actual_vm-a_public_IP>'
timeout --foreground 60s ssh -i "$KEY" -o BatchMode=yes -o IdentitiesOnly=yes -o HostKeyAlgorithms=ssh-ed25519 -o StrictHostKeyChecking=yes -o ConnectTimeout=10 -o ConnectionAttempts=1 -o ControlMaster=no -o ControlPath=none opc@"$PUBLIC_IP" 'test "$(id -un)" = opc && hostname && cat /etc/os-release && rpm -q openssh-server && systemctl is-active sshd' > owner02-guest.txt || exit 1
date -u +%FT%TZ > owner02-session-ended.txt || exit 1
```

기대값은 exit0, Oracle Linux 9 release, openssh-server package, sshd active다. patch/설치 작업은 하지 않는다. trust 미확인/unknown host는 BLOCKED_TRUST, timeout/publickey/package/service 실패는 오류와 실제 exit를 owner10에게 반환한다. 책임표의 열은 `model | actual resource OCID | customer action/evidence/UTC | provider-managed layer | not executed`; IaaS 행에 이 guest inventory, PaaS 행에 아래 app Get/List, SaaS 행에 NOT_SELECTED를 쓴다.

IaaS는 provider가 hardware를 관리하고 사용자가 guest OS/app를 관리한다. 신규 vm-a를 중복 생성하지 않는다.
2. PaaS 경계 학습용 **Functions application 1개**를 실제 생성한다. 함수 invoke/registry/Docker image 배포는 하지 않는다. Developer Services → Functions → Applications, Region/Compartment lab-root 선택 → Create Application. [Creating Applications — complete Console fields](https://docs.oracle.com/en-us/iaas/Content/Functions/Tasks/functionscreatingapps.htm).
3. Name `lab-platform`, VCN compartment lab-root, VCN shared-vcn, Subnets compartment lab-root, subnet public-admin **1개**를 선택한다. public subnet에는 IGW가 필요하며 public-admin 단계 A가 이것을 충족해야 한다. private subnet을 선택하려면 service gateway가 필요하므로 본 최소 lab에서는 사용하지 않는다.
4. Shape **Generic_X86**를 선택한다. future function image architecture를 x86로 전제한 교육값이다. ARM-only면 Generic_ARM, multiarch image이면 Generic_X86_ARM을 고를 수 있지만 생성 후 shape 변경이 불가능하므로 실제 향후 계획과 맞춘다. 다른 유료 execution/함수는 만들지 않는다. free-form tags purpose/owner/expiry만 입력하고 **Create**를 누른다.
5. application ACTIVE/OCID/subnet IDs/shape를 조회한다. 이 검증은 application 생성이지 함수 실행·API endpoint의 성공이 아니다. public subnet 또는 private subnet은 invoke authorization 자체를 대체하지 않으며 IAM이 관리한다.
6. 책임표를 작성한다: VM guest patch/application installation은 고객, Functions runtime infrastructure는 provider; Functions code/dependencies/identity access는 고객. SaaS는 완제품 업무 record/authorization/data 사용 중심이며 Functions application을 SaaS resource라고 부르지 않는다.

### SaaS 조건부 실습 범위

기존 승인된 SaaS sandbox와 제품명이 제공된 경우만 제품의 실제 신규 테스트 record 화면에서 비민감 record를 생성·조회·삭제한다. 본 요청에는 SaaS 제품/계정/schema가 없으므로 이 subtrack은 **NOT_SELECTED / vendor-specific procedure not asserted**이다. 신규 유료 SaaS 구독을 구입하지 않고 OCI에 가짜 SaaS instance를 만들지 않는다. 세 모델 모두 provisioning했다고 주장하지 않는다. core는 실제 IaaS VM + PaaS application 생성으로 완결된다.

## 결과검증


### baseline-02-ready


```bash
# Cloud Shell; Console Create 이후 실제 OCID를 지정한다.
set -euo pipefail
export APPLICATION_ID='<Console_Create의_lab-platform_OCID>'
export PUBLIC_SUBNET_ID='<owner23_actual_public-admin_OCID>'
export LAB_COMPARTMENT_ID='<owner00_actual_lab-root_OCID>'
APP_WORK=$(mktemp -d "$HOME/owner02-app.XXXXXXXX") || exit 1
cd "$APP_WORK" || exit 1
DEADLINE=$((SECONDS+300)); N=0; READY=0
while [ "$SECONDS" -lt "$DEADLINE" ]; do
  N=$((N+1))
  oci fn application get --application-id "$APPLICATION_ID" --region "$REGION" --connection-timeout 10 --read-timeout 20 --max-retries 0 > "platform-application-$N.json" 2> "platform-application-$N.error" || exit 1
  STATE=$(python3 - "platform-application-$N.json" <<'PY'
import json,os,sys
x=json.load(open(sys.argv[1]))['data']
assert x['id']==os.environ['APPLICATION_ID']
assert x['compartment-id']==os.environ['LAB_COMPARTMENT_ID']
assert x['subnet-ids']==[os.environ['PUBLIC_SUBNET_ID']]
assert x['shape']=='GENERIC_X86'
print(x['lifecycle-state'])
PY
) || exit 1
  case "$STATE" in ACTIVE) READY=1; break;; CREATING) ;; *) printf 'BLOCKED_APP: %s\n' "$STATE"; exit 1;; esac
  sleep 10
done
test "$READY" = 1 || { printf '%s\n' 'BLOCKED_APP_READY_DEADLINE'; exit 1; }
oci fn function list --application-id "$APPLICATION_ID" --all --region "$REGION" > platform-functions.json || exit 1
python3 - <<'PY'
import json
assert json.load(open('platform-functions.json'))['data']==[], 'BLOCKED: unknown functions; do not delete'
PY
```
기대 패턴 application lifecycle-state=ACTIVE, shape=GENERIC_X86, subnet-ids에 public-admin OCID 1개, functions data=[]; IaaS는 같은 vm-a RUNNING/SSH 증거를 연결한다. empty functions는 의도된 설정이며 함수 실행 성공을 뜻하지 않는다.

**증거 기록:** 실제 명령·UTC 시간·exit code·가려진 OCID·옵션 readback·검증 조건을 남긴다. 현재 이 문서의 실행상태는 `NOT_EXECUTED`다. output/creation-event-id/result는 실행 전 비워 둔다. 오류/NOT_SELECTED를 PASS로 바꾸지 않는다.

### 이 시나리오의 functional acceptance

| Positive | Negative / 미완료 판정 | Recovery / 반환 |
|---|---|---|
| application ACTIVE/subnet1/shape; functions empty | function list가 nonempty면 목표 불일치로 중단; invoke 성공이라 하지 않음 | 자기 함수가 아니면 삭제하지 않고 creator 확인; 빈 application scope만 완료/삭제 |

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

application create가 subnet 오류로 실패하면 public-admin의 IGW 경로·Region·use network 권한을 확인한다. fn-app 권한만으로 subnet 사용이 되지 않는다. function list가 unauthorized면 inspect fn-function을 관리자와 확인한다. application ACTIVE를 함수 invocation 성공으로 해석하지 않는다.

## 리소스정리

### 빈 app만 Delete 한 번, finite readback

APP_WORK/REGION/LAB_COMPARTMENT_ID/APPLICATION_ID/PUBLIC_SUBNET_ID를 owner02 registry에 저장하여 새 Cloud Shell 세션에서 정확히 재지정한다. 사전 관리자 승인에는 공식 IAM 표의 ListApp FN_APP_LIST, GetApp FN_APP_READ, DeleteApp FN_APP_DELETE(OCI CLI application list/get/delete) 및 `ListFunctions` FN_FUNCTION_LIST가 필요하다. [Functions permission table](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/functionspolicyreference.htm). 같은 identity가 Create 후 Get한 app의 compartment-wide app 목록을 성공 조회할 수 있음을 먼저 확인한다. visibility를 관리자 확인 없이 `[]`만으로 삭제 증명하지 않는다.

```bash
# Cloud Shell: 바로 Delete 전에 새 --all function list를 저장한다.
set -euo pipefail
CLOSE_WORK=$(mktemp -d "$HOME/owner02-close.XXXXXXXX") || exit 1
cd "$CLOSE_WORK" || exit 1
oci fn application get --application-id "$APPLICATION_ID" --region "$REGION" > app-before-delete.json || exit 1
oci fn function list --application-id "$APPLICATION_ID" --all --region "$REGION" > functions-before-delete.json || exit 1
oci fn application list --compartment-id "$LAB_COMPARTMENT_ID" --all --region "$REGION" > apps-before-delete.json || exit 1
python3 - <<'PY'
import json,os
x=json.load(open('app-before-delete.json'))['data']
assert x['id']==os.environ['APPLICATION_ID'] and x['compartment-id']==os.environ['LAB_COMPARTMENT_ID']
assert x['display-name']=='lab-platform' and x['subnet-ids']==[os.environ['PUBLIC_SUBNET_ID']]
assert json.load(open('functions-before-delete.json'))['data']==[], 'BLOCKED_UNKNOWN_FUNCTIONS'
assert any(x['id']==os.environ['APPLICATION_ID'] for x in json.load(open('apps-before-delete.json'))['data'])
PY
```

위가 성공하고 다른 function 작성자가 없음을 owner02가 확인한 뒤 Console → Developer Services → Functions → Applications → 같은 lab-root/Region → **actual app OCID의 Actions(세 점) → Delete**. 빈 app 확인창의 Delete를 한 번 선택한다. 함수 목록/`DELETE <APPLICATION-NAME>` 입력창이 보이면 **취소**하고 owner에게 반환한다; Console은 app 안 함수를 함께 지울 수 있다. [현재 Deleting an Application task](https://docs.oracle.com/en-us/iaas/Content/Functions/Tasks/functionsdeletingapplications.htm).

```bash
# Cloud Shell: 같은 identity/Region/compartment, 600초 교육용 상한(서비스 SLA 아님).
DEADLINE=$((SECONDS+600)); N=0; ABSENT=0
while [ "$SECONDS" -lt "$DEADLINE" ]; do
  N=$((N+1))
  oci fn application list --compartment-id "$LAB_COMPARTMENT_ID" --all --region "$REGION" --connection-timeout 10 --read-timeout 20 --max-retries 0 > "apps-after-$N.json" 2> "apps-after-$N.error" || exit 1
  PRESENT=$(python3 - "apps-after-$N.json" <<'PY'
import json,os,sys
r=json.load(open(sys.argv[1]))['data'];assert isinstance(r,list)
print(int(any(x['id']==os.environ['APPLICATION_ID'] for x in r)))
PY
) || exit 1
  if [ "$PRESENT" = 0 ]; then ABSENT=1; break; fi
  sleep 10
done
test "$ABSENT" = 1 || { printf '%s\n' 'BLOCKED_APP_DELETE_DEADLINE'; exit 1; }
```

기대 결과는 승인된 visibility 아래 성공한 전체 목록에서 exact app OCID 없음 및 Console 같은 목록 재확인이다. Get의 NotAuthorizedOrNotFound는 성공 판정으로 사용하지 않는다. nonempty functions/실패 Get/List/IAM 변경/timeout이면 삭제 추가 제출 없이 app OCID/request-id/UTC/마지막 evidence를 관리자에게 반환한다. shared subnet/VM 삭제 금지; 새로운 함수/image도 만들지 않는다. APP_WORK/CLOSE_WORK와 local guest/session evidence를 승인 보존경로에 저장하고 owner10/23에 consumer02 lease RELEASED, 종료 UTC, outstanding changes=none, owner acknowledgment를 기록한다. 이 run에서 만든 exact evidence 파일만 보존 확인 후 제거하고 빈 run directory만 rmdir한다.

## 공식문서

- [function list --application-id/--all](https://docs.oracle.com/en-us/iaas/tools/oci-cli/latest/oci_cli_docs/cmdref/fn/function/list.html) — 이 문서의 readiness/cleanup 조회 옵션 근거.

- [application list --compartment-id/--all](https://docs.oracle.com/en-us/iaas/tools/oci-cli/latest/oci_cli_docs/cmdref/fn/application/list.html) — 이 문서의 readiness/cleanup 조회 옵션 근거.

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 아래 본문은 각 생성 필드·설정·검증·정리의 기술 근거이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [45] [What Is Cloud Computing? — IaaS / PaaS / SaaS](https://www.oracle.com/cloud/what-is-cloud-computing/)
  - 확인 범위: Cloud Computing Services You can choose.

### 추가 실행 절차 근거 (본문 확인)

- [functions](https://docs.oracle.com/en-us/iaas/Content/Functions/Tasks/functionscreatingapps.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [launch](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/launchinginstance.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [cloudshell](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/cloudshellgettingstarted.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.

- [추가 세부 task: fnref](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/functionspolicyreference.htm) — 실제 본문을 확인한 필드·API/권한·삭제 단계 근거.

- [추가 세부 task: fnpolicy](https://docs.oracle.com/en-us/iaas/Content/Functions/Tasks/functionscreatingpolicies.htm) — 실제 본문을 확인한 필드·API/권한·삭제 단계 근거.
