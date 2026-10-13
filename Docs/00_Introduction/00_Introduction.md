# Introduction

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../README.md)를 확인하세요.

> 실행 절차 작성본 / OCI 미실행. Oracle 공식 본문 근거와 교육용 선택을 구분한다. secrets·실제 고객값·diagram 없음.

## 학습목표

- 승인 부모 아래 lab-root Compartment 한 개를 생성하고 ACTIVE·부모 OCID·purpose/owner/expiry 태그를 조회해 승인값과 대조한다.
- 후속 생성자가 사용할 LAB_COMPARTMENT_ID와 삭제 기한을 전달하고 모든 Region의 소비자 종료 뒤 마지막 삭제를 확인한다.

위 항목은 사용자 실행 후 확인할 완료 조건이며 현재 관측 결과가 아니다. 개념은 아래 해당 생성 필드·configuration·test 단계에서 함께 적용한다.

## 대응 Roadmap 항목

- 항목명: **Introduction**
- Roadmap 경로: **Introduction**
- Node ID: `sequence-0-0`
- 상위 항목: 없음 — 최상위 학습 항목
- 문서 범위: 이 항목의 생성·선택·기능검증·정리. 실습 미실행; 아래 기대 패턴은 실제 출력 아님.
- 번호는 navigation 식별자이며 실행 순서는 사전준비의 단계별 의존성에 따른다.

## 사전준비

### 기준 계약과 실제 단계 선행조건

[공통 baseline-contract.json](../../baseline-contract.json)의 Oracle Linux 9 / opc / CIDR / 비용 상한을 따른다. 문서 번호는 실행 순서가 아니다. **준비 station과 생성 이후 ready를 분리**한다. 아래 단계 외의 기능완료를 소급하여 요구하지 않는다.

- 외부 bootstrap: 조직 tenancy/로그인/MFA, 승인 부모와 Create/Get/Delete 위임. 다른 lab 완료를 요구하지 않는다.

생성 전 조직 승인(identity/권한/예산/expiry/서비스 지원/limit/capacity)을 확인한다. 실제 OCID/IP는 아직 생성하지 않았으면 비워 두고 해당 Create/readback 뒤 채운다. 동일 creation-event-id/OCID는 한 번의 제출만 뜻한다. `INSTANCE_A_ID`는 owner10 vm-a의 OCID, `INSTANCE_B_ID`는 owner22 private-vm의 OCID이며 owner07 vm-b와 다르다. 실습에 없는 자원을 추가하지 않는다. 실제 checkpoint는 사용자 실행 전 비워 둔다.

### 입력값과 실행 위치

Console에서 선택한 값만 사용한다. `<...>`는 예시 OCID나 비밀값이 아니라 **사용자가 대체할 입력 자리**다. 대체하지 않은 채 명령을 실행하지 않는다. Console → 우측 상단 Cloud Shell/Code Editor → Cloud Shell을 열면 OCI CLI가 사전 인증된다. Region 변경 뒤에는 Shell을 다시 열거나 명령의 `--region`을 바꾼다. SSH는 Cloud Shell이 아니라 본인 Linux/WSL/macOS 터미널에서 수행한다. 공인 IP/32도 그 터미널의 출구 주소여야 한다.

```bash
# 아래는 입력/출력의 대입 형식이다. 표의 단계 전에 생성 후 ID/IP export 행을 실행하지 않는다.
 # 생성 후 검증용 ID/IP는 해당 creator의 실제 응답에서 넣는다.
export REGION='<actual_REGION>'
export TENANCY_ID='<actual_TENANCY_ID>'
export LAB_COMPARTMENT_ID='<actual_LAB_COMPARTMENT_ID>'
oci --version
```

이 변수 중 해당 절차에서 사용하는 값만 채운다. CLI 조회의 `data`를 증거 파일에 저장하고 실제 시간·작업 OCID·선택값을 기록한다. API key/private key/passphrase/인증 토큰은 기록하지 않는다. Cloud Shell 설정파일이나 세션 토큰을 출력하지 않는다. `oci --version`을 실행하는 주제의 기대 패턴은 버전 문자열뿐이다. namespace는 Object Storage 주제에서 별도 조회하며 다른 주제에는 요구하지 않는다. [Cloud Shell 공식 본문](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/cloudshellgettingstarted.htm).

### Workshop 입력 계약 — 단계와 host

| 입력/출력 | 값 또는 실제 획득 경로 | 사용할 시점 | 실행 위치 | 선택 이유 |
|---|---|---|---|---|
| Console/설정 필드 | Name=lab-root; Parent=승인 부모; Description=OCI roadmap disposable learning resources; Free-form purpose/owner/expiry | 생성 전 선택 → 생성 뒤 readback | 본문 Console/로컬 지정 | 이 topic의 최소 생성 범위 |
| `TENANCY_ID` | Console tenancy details의 실제 OCID | 생성 전 | Cloud Shell | AD/구독 조회 |
| `REGION` | 승인된 기존 subscribed Region identifier | 생성 전 | Console/Cloud Shell | API endpoint/데이터 위치 |
| `LAB_COMPARTMENT_ID` | owner00 Create 응답 OCID; topic00에서는 생성 뒤 | 선행 완료/생성 후 | Console/Cloud Shell | 정책/자원 scope |
| `PARENT_COMPARTMENT_ID` | owner00 승인 부모 OCID | 생성 전 | Console/Cloud Shell | bootstrap 부모 경계 |

생성 전에는 행의 승인값/선행 완료값만 지정하고, 생성 후 ID/IP는 해당 owner 응답으로 채운다. Cloud Shell과 로컬 SSH의 환경변수는 공유되지 않는다. 로컬에는 KEY와 접속 주소만 다시 지정하고 guest disk 명령은 로그인한 guest에서만 실행한다. `<...>`가 남으면 해당 명령은 실행하지 않는다.

## 권한

### 실제 task/API별 공식 권한 본문 대조

아래는 이 문서 생성·조회·변경·정리 API의 공식 표 본문을 읽은 인용이다. 위 family 예제는 편의상 넓은 compartment-scope grant이며 **최소권한 정책이라고 부르지 않는다**. verb/dependency 행 인용은 단일 API의 완전한 최소권한 도출이 아니다. 조직 Identity Domain Console user/group 작업은 해당 Domain 관리자 역할도 필요하며 legacy IAM user API 권한만으로 대체하지 않는다.

- `CreateCompartment`: COMPARTMENT_CREATE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/policyreference/iampolicyreference.htm).
- `DeleteCompartment`: COMPARTMENT_DELETE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/policyreference/iampolicyreference.htm).
- `GetCompartment`: COMPARTMENT_INSPECT — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/policyreference/iampolicyreference.htm).

승인 부모에서 Compartment 생성/조회/삭제 가능한 관리자 또는 위임 사용자가 필요하다. 관리자가 정책을 작성하며 실습자가 관리자 그룹에 가입하는 절차를 요구하지 않는다. 예시 `Allow group <domain>/<lab-group> to manage compartments in compartment <approved-parent>`는 IAM compartment 관리 정책의 scope/상속을 관리자와 검토한 뒤 사용한다. 새 lab-root가 만들어졌다고 서비스 CRUD 권한이 자동 부여되는 것은 아니다. 후속 서비스 정책은 lab-root로 한정한다. 실제 권한 allow/deny 테스트는 미실행이다. [Required access](https://docs.oracle.com/en-us/iaas/Content/Identity/compartments/To_create_a_compartment.htm)

## 비용및주의사항

- 생성 전에 담당자에게 Region별 현재 단가, service limit/quota, 사용 가능한 capacity, 최대 수량, 종료시각, 누적 예산을 승인받는다. Billing & Cost Management → Cost Analysis에서 Compartment/서비스 필터를 사용하되 청구 반영 지연을 감안한다. 무료 표시나 trial credit만으로 전체 무료라고 판단하지 않는다.
- 기본은 `lab-root` 1개, `shared-vcn` 1개, public/private subnet 각 1개, 공유 `vm-a` 1개다. FD/private/burstable/preemptible용 두 번째 VM은 필요한 시간만 순차 사용하고 즉시 종료한다. Boot/data volume·backup·public address·요청/전송 비용은 VM과 따로 확인한다.
- 고객 데이터, 기존 운영망, 실제 secrets를 사용하지 않는다. 생성 화면에서 예상치 못한 유료 옵션·큰 기본 disk size·광범위 SSH 규칙이 보이면 제출하지 말고 수정한다. 예산 경보는 강제 과금 차단 장치가 아니다.
- 이 문서의 생성 절차는 Oracle 본문을 바탕으로 작성한 교육 절차이며 Oracle 공인 튜토리얼이라는 뜻이 아니다. **OCI 리소스 생성·명령 실행·권한 allow/deny 테스트는 수행하지 않았다.** 결과검증의 값은 기대 패턴이며 실행된 출력이 아니다. 선택 확장은 승인 없이 실행하지 않고 `NOT_SELECTED`로 남긴다.

## 실습구성

### 이번 Workshop에서 만드는 것과 진행 순서

승인 부모 아래 lab-root 한 개를 만들어 후속 자원의 삭제 책임을 고정한다. Region은 위치 선택이지 Compartment의 하위 자원이 아니다.

1. **진입/선택:** 부모/담당자/expiry 승인. 사전준비 링크의 지정 단계/실제 출력이 없으면 제출하지 않는다.
2. **실제 작업:** Console Compartment 생성 → ACTIVE/부모/tags readback. 아래 Console 필드/명령을 해당 host에서 순서대로 실행한다. 같은 자원은 이름이 아니라 OCID/event로 재사용한다.
3. **완료/복구/반환:** 후속 서비스 권한 별도 승인. 결과검증 실패는 문제해결로 이동하고 동일 Create를 무작정 재제출하지 않는다.

**전달/정리 범위:** lab-root의 실제 OCID/부모/owner/expiry; 모든 Region의 소비자가 끝난 뒤 owner00만 삭제. 아래 정리 단계는 이 소유권을 넘어서 실행하지 않는다.


- **이번 생성/재사용/출력:** 신규: lab-root Compartment 1개. 재사용: 승인 tenancy/부모. 출력: LAB_COMPARTMENT_ID, parent OCID, owner/expiry, REGION. VM/network는 만들지 않는다.
- **생성 목표:** 최소 실제 bootstrap: 승인된 부모 아래 학습용 Compartment를 생성하여 전체 실습의 소유권 경계를 시작한다. Compute·VCN까지 여기서 미리 만들지 않는다.
- **완료 증거:** 생성 전 옵션 선택 + 실제 제출 대상 OCID + API readback + 해당 기능검증 + dependency 순서 정리. 실제 OCI 실행 checkpoint는 비워 둔다.
- **retention:** 예약 소비자와 expiry가 있는 resource만 보존. idle 유료 resource를 다음 주제 때문에 무기한 유지하지 않는다.


## 실습절차

### 명령 실행 gate — 실패하면 다음 생성으로 진행하지 않는다

1. 아래 Bash는 **전용 Bash 세션**에서만 사용한다. Console/Cloud Shell 조회, 본인 로컬 SSH, 로그인한 Oracle Linux guest를 섞지 않는다. `exit 1`은 이 전용 세션을 닫는 안전 중단이다. `<...>`·빈 입력·타 owner OCID가 남으면 실행하지 않는다.
2. 성공을 요구하는 파일 작성/조회/설치/attach 작업은 실패 시 중단한다. 실패한 API 뒤 기존 JSON이나 기존 다운로드 파일을 성공 근거로 읽지 않는다. 기대 deny는 `if command; then ...; else RC=$?; ...; fi`로 별도 기록하고, 실제 timeout/IAM 오류/로컬 파일 오류를 구별한다.
3. 첫 변경 전에 정확한 OCID·owner·현재 설정과 승인 expiry를 저장한다. 공유 자원 변경 중 다른 소비자가 시작하거나 원복값을 확인할 수 없으면 변경을 중단한다. 삭제 성공은 API 수락이 아니라 같은 identity/Region의 정확한 대상 재조회로 판정한다. `NotAuthorizedOrNotFound` 단독은 삭제 증거가 아니다.


### 개별 완주 보정 — 생성·검증·소유권

생성 전 LAB_COMPARTMENT_ID는 비워 두고 Create 후 실제 새 id로 설정한다. 후속 서비스 정책은 관리자에게 별도 승인받는다. owner00는 모든 Region/자식/서비스 잔존을 확인한 뒤 마지막에 삭제한다.

### Compartment 한 개를 실제 생성한다

1. tenancy 가입/로그인/MFA는 조직이 제공한 계정으로 완료한다. 승인 부모 Compartment의 OCID·이름, 담당자 alias, 삭제 기한을 받는다. 이 lab은 기존 Compartments/IAM 초안의 완료를 요구하지 않는 **외부 bootstrap**이다. 부모 범위 위임은 관리자가 먼저 설정한다.
2. Console → Identity & Security → Identity → Compartments. 승인 부모를 클릭하고 Create compartment를 선택한다. Name `lab-root`, Description `OCI roadmap disposable learning resources`, Parent compartment = **승인 부모**. 이름은 부모 안에서 고유해야 하며 비민감 값만 쓴다. [Creating a Compartment — Console fields](https://docs.oracle.com/en-us/iaas/Content/Identity/compartments/To_create_a_compartment.htm).
3. Tags는 Free-form으로 `purpose=oci-roadmap`, `owner=<alias>`, `expiry=<승인_종료시각>`를 입력한다. Defined tags는 namespace 사용 권한이 없으면 선택하지 않는다. Create compartment를 제출하고 새 OCID를 기록한다.
4. 상세 상태 ACTIVE, 부모 OCID, description, tags를 다시 확인한다. 해당 Compartment를 선택해 후속 lab의 리소스 scope를 고정한다. Compartment는 tenancy-wide 관리 경계이며 Region/AD가 아니다. 같은 Compartment로 Region별 리소스를 관리할 수 있다.
5. Console 상단 Region picker에서 이미 구독된 승인 Region identifier를 기록하고 Cloud Shell을 연다. 아래 조회를 실제 생성값과 대조한다. 이 단계에서는 namespace/bucket/VM을 요구하거나 만들지 않는다. namespace 조회는 owner04의 Object Storage 절차에서 수행한다.

### ACTIVE 대기 상한

Create를 한 번 제출한 뒤 5분 안에 10초 간격으로 같은 OCID의 상태만 확인한다. 5분은 교육용 중단 상한이며 Oracle SLA가 아니다. 마지막 JSON/error/request-id/UTC 시각을 보존하고 timeout·권한 오류에는 재생성하지 않는다.

```bash
# Cloud Shell 전용 Bash; LAB_COMPARTMENT_ID는 방금 생성한 실제 OCID
set -euo pipefail
DEADLINE=$((SECONDS+300))
while :; do
  oci iam compartment get --compartment-id "$LAB_COMPARTMENT_ID" > compartment-wait.json || exit 1
  STATE=$(python3 -c 'import json; print(json.load(open("compartment-wait.json"))["data"]["lifecycle-state"])') || exit 1
  printf 'state=%s\n' "$STATE"
  if [ "$STATE" = ACTIVE ]; then break; fi
  if [ "$SECONDS" -ge "$DEADLINE" ]; then printf '%s\n' 'BLOCKED: ACTIVE deadline; retain evidence, do not Create again'; exit 1; fi
  sleep 10
done
```

## 결과검증

<a id="baseline-00-ready"></a>

```bash
export PARENT_COMPARTMENT_ID='<승인_부모_OCID>'
oci iam compartment get --compartment-id "$LAB_COMPARTMENT_ID" > compartment-readback.json || exit 1
python3 - <<'PY'
import json,os
x=json.load(open('compartment-readback.json'))['data']
assert x['lifecycle-state']=='ACTIVE'
assert x['name']=='lab-root'
assert x['compartment-id']==os.environ['PARENT_COMPARTMENT_ID']
assert x['freeform-tags']['purpose']=='oci-roadmap'
print('compartment assertions passed')
PY
```
기대 패턴은 assertion 종료코드 0과 `compartment assertions passed`; 위 문구는 실행 예시가 아니라 코드가 조건을 만족할 때 출력하는 패턴이다. owner/expiry도 승인값과 비교한다.

**증거 기록:** 실제 명령·UTC 시간·exit code·가려진 OCID·옵션 readback·검증 조건을 남긴다. 현재 이 문서의 실행상태는 `NOT_EXECUTED`다. output/creation-event-id/result는 실행 전 비워 둔다. 오류/NOT_SELECTED를 PASS로 바꾸지 않는다.

### 이 시나리오의 functional acceptance

| Positive | Negative / 미완료 판정 | Recovery / 반환 |
|---|---|---|
| lab-root ACTIVE/parent/tags match | 승인 parent 아닌 값/빈 OCID는 Create 전 gate에서 중단 | 승인 parent/actual newly created id를 다시 지정하고 Get 재대조 |

모두 실행 시 확인할 기대 패턴이다. 생성 전 guard negative는 OCI 장애를 주입한 것과 다르며 다른 자원을 추가하지 않는다. 오류 메시지와 실제 exit code가 없는 항목은 NOT_EXECUTED 또는 BLOCKED로 남긴다.

## 문제해결

### 공통 오류의 진단 순서

1. `NotAuthorizedOrNotFound` 또는 목록에 리소스가 없음: 먼저 Region/Compartment/OCID와 identity domain/group을 확인한다. 존재하지 않는 OCID와 권한 부족 모두 같은 오류로 보일 수 있다. 관리자에게 request-id·작업 시각·서비스·가려진 대상 OCID를 제공한다. 비밀키/토큰은 보내지 않는다.
2. `LimitExceeded`/quota/capacity 부족: Billing & Cost Management → Limits, Quotas and Usage와 생성 화면의 지원 shape/AD를 확인한다. 운영 quota 증가나 다른 Region 구독을 임의 요청하지 않는다. 승인 범위 안의 더 작은 compatible shape 또는 다른 AD를 선택하거나 `BLOCKED_CAPACITY`를 기록한다.
3. 장시간 provisioning/예상 밖 state: Work Requests/오류 상세를 읽고 같은 요청을 반복 제출하지 않는다. 만들어진 부분 리소스(boot/IP/attachment 포함)를 먼저 조사한다.
4. 메뉴 이름이 다름: 공식 task body의 동등한 탭(예: Storage 또는 Attached Block Volumes)을 찾는다. 필드가 문서와 다르면 값·스크린샷에서 secrets를 가린 뒤 관리자 또는 Oracle Support에 확인하고 제출을 중단한다. 이 문서는 현재 tenancy의 live Console 화면을 검사한 결과가 아니다.

### 이 주제의 실패 지점

동일 이름 충돌이면 승인 부모와 기존 OCID를 조회한다. 다른 사용자의 lab-root를 재사용하거나 삭제하지 말고 관리자와 새 승인 suffix를 정한다. suffix를 사용했다면 공통 registry의 Compartment 이름과 아래 name assertion 및 후속 정책의 lab-root 이름을 모두 동일 승인 이름으로 바꾸고 다시 검토한다. 다른 이름을 만든 뒤 lab-root assertion을 그대로 실행하지 않는다. 신규 Compartment가 목록에 늦게 나타나면 부모·ACTIVE·정책 전파를 확인한다.

## 리소스정리

후속 lab(Introduction to OCI, SSH preparation, VCN 및 이후 IAM/Storage)의 실행이 승인된 시간 안에 예정되어 있으면 lab-root를 보존한다. 새 Compartment는 소유권 경계일 뿐 다른 리소스 비용을 지우지 않는다. 아래 작업은 **모든 소비자의 반환과 owner00의 최종 삭제 승인 후에만** 수행한다. 조회를 위해 새 Region을 구독하거나 Resource Manager stack·유료 리소스를 만들지 않는다.

### 1. owner00의 삭제 전 inventory — 빈 목록만으로 승인하지 않는다

1. owner00는 생성 때 저장한 `LAB_COMPARTMENT_ID`, 승인 부모 OCID, owner/expiry, 소비자별 resource OCID와 반환 증거를 모은다. Console → Identity & Security → Identity → Compartments에서 승인 부모를 따라 lab-root의 상세를 열고 **이름이 아닌 OCID/부모**를 대조한다. 그 아래 자식 Compartment와 각 자식의 자식을 끝까지 열어 경로/OCID/담당자를 목록에 적는다. 이 lab은 자식을 새로 만들지 않지만 다른 lab이 남긴 자식도 누락하면 안 된다.
2. Console 상단 Region picker의 **기존 subscribed Region 전체**를 기록한다. Governance & Administration → Tenancy Management → Tenancy Explorer → 왼쪽 compartment picker에서 위 OCID의 lab-root를 선택 → **Show resources in subcompartments**를 켠다. 결과의 Compartment 열로 자식 경로를, Region/서비스/OCID로 위치와 소유자를 기록한다. 이 화면은 cross-region 발견을 돕지만 지원 리소스와 현재 사용자의 보기 권한에 한계가 있다. `GetCompartment`는 메타데이터이지 자원 inventory가 아니며 단일 all-resource API도 없다. [공식 Viewing Resources / Troubleshooting 본문](https://docs.oracle.com/en-us/iaas/Content/Identity/Tasks/managingcompartments.htm).
3. 각 subscribed Region으로 전환하고 lab-root 및 **모든 자식 OCID**를 각 서비스의 Compartment/List Scope로 선택한다. 아래 표를 Region × Compartment별로 작성한다. 알려진 creation/retention registry의 서비스가 표에 없으면 그 서비스의 공식 목록 화면도 추가한다. 사용하지 않았다는 owner 기록 없이 `N/A`로 건너뛰지 않는다. 숨겨진 필터·페이지가 없는지 확인하고 OCID/상태/owner/반환 여부/조회 시각을 기록한다. `NOT_VISIBLE`, 조회 실패, 미반환은 `EMPTY`가 아니다.

| 서비스별 확인 화면/대상 | 빠뜨리기 쉬운 잔여 항목 |
|---|---|
| Compute → Instances 및 사용한 Compute 관리 화면 | 종료되지 않은 VM, instance pool/configuration, custom image, autoscaling/capacity reservation |
| Storage → Block Storage / File Storage / Object Storage & Archive Storage | boot/data volume, backup, attachment; file system/mount target; bucket 안 object/version/multipart 및 retention lease |
| Networking → Virtual Cloud Networks 및 Load Balancers / Network Load Balancers | VNIC/IP, subnet, route/security/NSG, gateway, LB/NLB와 해당 owner의 app retention |
| Identity & Security → 사용한 WAF 화면; Networking → DNS Management | WAF policy/firewall, zone/record/resolver 및 공유 endpoint 반환 |
| Observability & Management → 사용한 Monitoring / Logging / Health Checks 화면 | alarm/log/log group/check와 서비스별 비지원 발견 항목 |
| Oracle Database → 사용한 DB 서비스; Developer Services → 사용한 Functions 화면 | DB/backup 보존 승인, application/function; 선택 유료 branch의 owner 정리 증거 |

4. **Policies는 별도 조회:** Identity & Security → Identity → Policies → 왼쪽 Compartment를 lab-root로 선택하고 attached policy 이름/OCID/owner를 기록한다. 각 자식에도 반복한다. Policies는 Search 결과에 포함되지 않는다. tenancy/부모에 붙었지만 lab-root를 참조하는 정책도 관리자와 별도 검토하며, 조직 공용 정책 전체를 삭제하지 않는다. 소비자 서비스 정리가 끝나기 전에 서비스 접근 정책을 없애지 않는다.
5. 권한 거부 또는 Explorer가 비어 있어도 전체 범위 보기 권한을 증명할 수 없으면 **BLOCKED_VISIBILITY**로 멈춘다. 사전 승인된 tenancy 관리자에게 대상/부모/자식 OCID, subscribed Region 목록, 서비스별 표, 반환 기록, 오류/request-id/UTC를 전달한다. 관리자는 자신의 승인된 계정으로 1–4를 반복하고 누락 자원과 담당자 또는 전체 inventory 확인을 서면 반환한다. 실습자가 관리자 그룹에 가입하거나 임의 정책을 추가하는 명령은 없다. 삭제/Work Request 조회 권한도 제출 전에 관리자에게 확인받는다. 토큰·키·Authorization header는 전달하지 않는다.
6. 각 creator가 자신의 공식 cleanup station으로 자원을 정리한다. 서비스별 의존성을 먼저 풀고 VM/VNIC → volume/backup/object → subnet/gateway/VCN → lab 소유 attached policy → **가장 깊은 자식 Compartment부터** 정리한다. 부모/타 owner 자원은 삭제하지 않는다. 자식에도 이 inventory와 아래 비동기 완료 확인을 적용한다. 마지막에 owner00와 관리자가 전체 표를 재조회하여 잔여 자원/attached policy/활성 자식 없음, 모든 소비자 반환, 삭제 승인 시각을 확인해야 lab-root Delete로 간다.

### 2. Delete 한 번 → 정확한 Work Request 조회 → bounded 완료/실패 반환

1. Console → Identity & Security → Identity → Compartments에서 **승인 부모 아래 실제 lab-root OCID**를 재확인한다. 삭제 제출 전에 본인 브라우저 Developer Tools → Network를 열고 기존 기록을 지운다. Actions(세 점) → Delete → 확인창 Delete를 **한 번만** 선택한다. 해당 대상의 `DELETE /20160918/compartments/<LAB_COMPARTMENT_ID>` 요청을 선택하고 **Response Headers의 `opc-workrequest-id`**만 작업 OCID로 복사한다. 이것은 DeleteCompartment 응답으로 생성된 작업 ID이며 resource OCID나 `opc-request-id`가 아니다. 요청 헤더/HAR/인증 정보는 저장하지 않는다. header가 표시되지 않으면 재제출하지 말고 관리자에게 이 삭제 요청 시각/대상/request-id로 작업 ID 확보를 요청한다. 그 전에는 `BLOCKED_WORKREQUEST_ID`다. [Deleting a Compartment](https://docs.oracle.com/en-us/iaas/Content/Identity/compartments/To_delete_a_compartment.htm), [IAM Work Request get — 응답 header와 필수 옵션](https://docs.oracle.com/en-us/iaas/tools/oci-cli/latest/oci_cli_docs/cmdref/iam/work-request/get.html).
2. Cloud Shell의 전용 Bash에서 실제 작업 ID와 **삭제 요청에 사용한 Region**을 지정한다. 아래는 조회만 한다. 10초 간격, 10분 교육용 deadline(Oracle SLA 아님), 각 API timeout/retry 제한을 적용한다. 증거 경로는 새 이름을 사용하고 기존 증거를 덮어쓰지 않는다. `errors`, `logs`, `resources`는 해당 IAM WorkRequest 응답 안에서 확인한다. 다른 서비스의 Work Requests 목록/명령을 대신 사용하지 않는다.

```bash
# Cloud Shell; 승인 inventory 완료 및 Console Delete 한 번 제출 후
set -euo pipefail
export DELETE_WORK_REQUEST_ID='<actual_opc-workrequest-id>'
: "${LAB_COMPARTMENT_ID:?}" "${REGION:?}" "${DELETE_WORK_REQUEST_ID:?}"
case "$LAB_COMPARTMENT_ID/$REGION/$DELETE_WORK_REQUEST_ID" in *'<'*|*'>'*) exit 1;; esac
EVIDENCE_DIR="intro00-delete-$(date -u +%Y%m%dT%H%M%SZ)"
mkdir -m 700 "$EVIDENCE_DIR" || exit 1
DEADLINE=$((SECONDS+600))
N=0
while [ "$SECONDS" -lt "$DEADLINE" ]; do
  N=$((N+1))
  oci iam work-request get --work-request-id "$DELETE_WORK_REQUEST_ID" --region "$REGION" --connection-timeout 10 --read-timeout 20 --max-retries 0 > "$EVIDENCE_DIR/work-$N.json" 2> "$EVIDENCE_DIR/work-$N.error" || exit 1
  oci iam compartment get --compartment-id "$LAB_COMPARTMENT_ID" --region "$REGION" --connection-timeout 10 --read-timeout 20 --max-retries 0 > "$EVIDENCE_DIR/compartment-$N.json" 2> "$EVIDENCE_DIR/compartment-$N.error" || exit 1
  python3 - "$EVIDENCE_DIR/work-$N.json" "$EVIDENCE_DIR/compartment-$N.json" <<'PY'
import json, os, sys
w=json.load(open(sys.argv[1]))['data']
c=json.load(open(sys.argv[2]))['data']
assert w['id']==os.environ['DELETE_WORK_REQUEST_ID']
assert w['operation-type']=='DELETE_COMPARTMENT'
assert c['id']==os.environ['LAB_COMPARTMENT_ID']
PY
  WSTATE=$(python3 -c 'import json,sys; print(json.load(open(sys.argv[1]))["data"]["status"])' "$EVIDENCE_DIR/work-$N.json") || exit 1
  CSTATE=$(python3 -c 'import json,sys; print(json.load(open(sys.argv[1]))["data"]["lifecycle-state"])' "$EVIDENCE_DIR/compartment-$N.json") || exit 1
  printf 'work=%s compartment=%s\n' "$WSTATE" "$CSTATE"
  if [ "$WSTATE" = SUCCEEDED ] && [ "$CSTATE" = DELETED ]; then
    printf '%s\n' 'Delete confirmed for the recorded OCID'; break
  fi
  case "$WSTATE" in FAILED|CANCELED) printf '%s\n' 'BLOCKED: owner/admin remediation required'; exit 1;; esac
  case "$WSTATE/$CSTATE" in ACCEPTED/ACTIVE|ACCEPTED/DELETING|IN_PROGRESS/ACTIVE|IN_PROGRESS/DELETING|SUCCEEDED/ACTIVE|SUCCEEDED/DELETING) ;; *) printf '%s\n' 'BLOCKED: unexpected state; retain evidence'; exit 1;; esac
  if [ "$SECONDS" -ge "$DEADLINE" ]; then break; fi
  sleep 10
done
if [ "${WSTATE:-}" != SUCCEEDED ] || [ "${CSTATE:-}" != DELETED ]; then
  printf '%s\n' 'BLOCKED: deadline; do not submit Delete again'; exit 1
fi
```

3. 기대 전이는 Compartment `DELETING` → `DELETED`, 작업 `SUCCEEDED`다. 실패 작업이면 Compartment는 `ACTIVE`로 돌아갈 수 있다. FAILED/CANCELED, API 오류, 권한 거부, timeout, 예상 밖 state에는 작업 ID·대상 OCID·Region·UTC·마지막 JSON/error를 보존하고 owner00/관리자에게 반환한다. `errors/logs/resources`를 읽어 owner를 지정하고 위 inventory에서 잔여 항목을 다시 찾는다. 실패 직후 아직 DELETING이면 같은 Get 조회를 10분/10초 한도로 별도 관찰하여 ACTIVE 반환을 확인하고, 상한을 넘으면 관리자/Oracle Support로 넘긴다. NotAuthorizedOrNotFound나 picker에서 사라짐만으로 성공 처리하지 않는다.
4. **재시도는 자동이 아니다:** 이전 작업 FAILED/CANCELED 및 같은 Compartment의 ACTIVE readback, creator의 잔여 자원 정리, 관리자 재inventory, 새 owner00 삭제 승인이 모두 있을 때만 이 station을 한 번 재시도한다. 새 `opc-workrequest-id`와 새 증거 디렉터리를 기록한다. 이번 재시도도 실패/상한이면 추가 Delete 없이 관리자/Support로 넘긴다. SUCCEEDED인데 상태 대조가 실패하거나 이전 작업이 진행 중이면 재시도하지 않는다. 비용 있는 잔여 자원은 해당 creator가 승인 expiry 안에서 정리하거나 별도 예산/기한 승인을 받고 반환한다.
5. Console Compartments의 Deleted 상태 필터에서 같은 OCID/변경된 이름을 대조하고 owner00 registry에 삭제 증거/소비자 종료를 남긴다. 공식 본문의 **90일 deleted compartment 표시/복구 기간**은 리소스 비용 제거 또는 즉시 영구 소실 보장이 아니다. 이 문서는 실제 삭제/권한/Console 검증을 실행하지 않았다. [WorkRequest status/errors/logs/resources 공식 모델](https://docs.oracle.com/en-us/iaas/tools/python/latest/api/identity/models/oci.identity.models.WorkRequest.html), [Compartment Get CLI](https://docs.oracle.com/en-us/iaas/tools/oci-cli/latest/oci_cli_docs/cmdref/iam/compartment/get.html).


## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 아래 본문은 각 생성 필드·설정·검증·정리의 기술 근거이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [45] [What Is Cloud Computing? — Introduction](https://www.oracle.com/cloud/what-is-cloud-computing/)
  - 확인 범위: Cloud computing delivers.

### 추가 실행 절차 근거 (본문 확인)

- [compartment](https://docs.oracle.com/en-us/iaas/Content/Identity/compartments/To_create_a_compartment.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [cloudshell](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/cloudshellgettingstarted.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
