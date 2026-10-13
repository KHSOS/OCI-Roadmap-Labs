# Fault Domains

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../README.md)를 확인하세요.

> 실행 절차 작성본 / OCI 미실행. Oracle 공식 본문 근거와 교육용 선택을 구분한다. secrets·실제 고객값·diagram 없음.

## 학습목표

- vm-a와 같은 AD·다른 FD에 승인된 작은 vm-b 한 개를 생성하고 두 instance readback을 비교한다.
- vm-b의 새 SSH를 확인하고 owner09/16 소비자가 반환한 뒤 vm-b와 boot만 종료한다. application failover를 시험한 것으로 기록하지 않는다.

위 항목은 사용자 실행 후 확인할 완료 조건이며 현재 관측 결과가 아니다. 개념은 아래 해당 생성 필드·configuration·test 단계에서 함께 적용한다.

## 대응 Roadmap 항목

- 항목명: **Fault Domains**
- Roadmap 경로: **Introduction to OCI → Fault Domains**
- Node ID: `sequence-1-3`
- 상위 항목: **Introduction to OCI** (`sequence-1-0`)
- 문서 범위: 이 항목의 생성·선택·기능검증·정리. 실습 미실행; 아래 기대 패턴은 실제 출력 아님.
- 번호는 navigation 식별자이며 실행 순서는 사전준비의 단계별 의존성에 따른다.

## 사전준비

### 기준 계약과 실제 단계 선행조건

[공통 baseline-contract.json](../../baseline-contract.json)의 Oracle Linux 9 / opc / CIDR / 비용 상한을 따른다. 문서 번호는 실행 순서가 아니다. **준비 station과 생성 이후 ready를 분리**한다. 아래 단계 외의 기능완료를 소급하여 요구하지 않는다.

- [10 Compute — 단계 링크](../02_Compute/00_Compute.md#baseline-10-ready): **vm-a RUNNING/새 SSH/AD/FD**. 실제 출력/소비자/expiry를 전달하고 해당 owner가 이 lab 종료까지 보존한다.

생성 전 조직 승인(identity/권한/예산/expiry/서비스 지원/limit/capacity)을 확인한다. 실제 OCID/IP는 아직 생성하지 않았으면 비워 두고 해당 Create/readback 뒤 채운다. 동일 creation-event-id/OCID는 한 번의 제출만 뜻한다. `INSTANCE_A_ID`는 owner10 vm-a의 OCID, `INSTANCE_B_ID`는 owner22 private-vm의 OCID이며 owner07 vm-b와 다르다. 실습에 없는 자원을 추가하지 않는다. 실제 checkpoint는 사용자 실행 전 비워 둔다.

### 입력값과 실행 위치

Console에서 선택한 값만 사용한다. `<...>`는 예시 OCID나 비밀값이 아니라 **사용자가 대체할 입력 자리**다. 대체하지 않은 채 명령을 실행하지 않는다. Console → 우측 상단 Cloud Shell/Code Editor → Cloud Shell을 열면 OCI CLI가 사전 인증된다. Region 변경 뒤에는 Shell을 다시 열거나 명령의 `--region`을 바꾼다. SSH는 Cloud Shell이 아니라 본인 Linux/WSL/macOS 터미널에서 수행한다. 공인 IP/32도 그 터미널의 출구 주소여야 한다.

```bash
# 아래는 입력/출력의 대입 형식이다. 표의 단계 전에 생성 후 ID/IP export 행을 실행하지 않는다.
 # 생성 후 검증용 ID/IP는 해당 creator의 실제 응답에서 넣는다.
export REGION='<actual_REGION>'
export INSTANCE_A_ID='<actual_INSTANCE_A_ID>'
export INSTANCE_B_FD_ID='<actual_INSTANCE_B_FD_ID>'
# KEY는 Cloud Shell에서 지정하지 않는다; 아래 local SSH station에서만 export한다.
oci --version
```

이 변수 중 해당 절차에서 사용하는 값만 채운다. CLI 조회의 `data`를 증거 파일에 저장하고 실제 시간·작업 OCID·선택값을 기록한다. API key/private key/passphrase/인증 토큰은 기록하지 않는다. Cloud Shell 설정파일이나 세션 토큰을 출력하지 않는다. `oci --version`을 실행하는 주제의 기대 패턴은 버전 문자열뿐이다. namespace는 Object Storage 주제에서 별도 조회하며 다른 주제에는 요구하지 않는다. [Cloud Shell 공식 본문](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/cloudshellgettingstarted.htm).

### Workshop 입력 계약 — 단계와 host

| 입력/출력 | 값 또는 실제 획득 경로 | 사용할 시점 | 실행 위치 | 선택 이유 |
|---|---|---|---|---|
| Console/설정 필드 | Name=vm-b; same AD/different FD; owner10와 같은 승인 image/shape/boot/.pub/public-admin/nsg-admin | 생성 전 선택 → 생성 뒤 readback | 본문 Console/로컬 지정 | 이 topic의 최소 생성 범위 |
| `INSTANCE_B_FD_ID` | 본문 실제 Create/readback 출력 | 생성 후/선행 완료 | 본문 지정 host | 해당 topic 대상만 지정 |
| `INSTANCE_A_ID` | owner10 실제 vm-a 생성 id | 선행 완료/생성 후 | Cloud Shell | 공유 A 조회; 이름은 OCID가 아님 |
| `KEY` | owner15 Part A 전용 private key 절대경로; 내용 전송 금지 | 준비 완료 | 로컬 SSH만 | 해당 공개키와 대응 |
| `REGION` | 승인된 기존 subscribed Region identifier | 생성 전 | Console/Cloud Shell | API endpoint/데이터 위치 |

생성 전에는 행의 승인값/선행 완료값만 지정하고, 생성 후 ID/IP는 해당 owner 응답으로 채운다. Cloud Shell과 로컬 SSH의 환경변수는 공유되지 않는다. 로컬에는 KEY와 접속 주소만 다시 지정하고 guest disk 명령은 로그인한 guest에서만 실행한다. `<...>`가 남으면 해당 명령은 실행하지 않는다.

## 권한

### 실제 task/API별 공식 권한 본문 대조

아래는 이 문서 생성·조회·변경·정리 API의 공식 표 본문을 읽은 인용이다. 위 family 예제는 편의상 넓은 compartment-scope grant이며 **최소권한 정책이라고 부르지 않는다**. verb/dependency 행 인용은 단일 API의 완전한 최소권한 도출이 아니다. 조직 Identity Domain Console user/group 작업은 해당 Domain 관리자 역할도 필요하며 legacy IAM user API 권한만으로 대체하지 않는다.

- `GetInstance`: INSTANCE_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `LaunchInstance`: INSTANCE_CREATE and INSTANCE_IMAGE_READ and VNIC_CREATE and VNIC_ATTACH and SUBNET_ATTACH If putting the instance in a network security group during instance creation, also need NETWORK_SECURITY_GROUP _UPDATE_MEMBERS and VNIC_ASSOCIATE _NETWORK_SECURITY_GROUP If creating an instance in a compute cluster, also need COMPUTE_CLUSTER_LAUNCH_INSTANCE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- **Dedicated host 선택 확장에만 해당** `LaunchInstance`: DEDICATED_VM_HOST_LAUNCH_INSTANCE in dedicated virtual machine host compartment INSTANCE_CREATE in compartment for the instance launched on the dedicated virtual machine host — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `TerminateInstance`: INSTANCE_DELETE and VNIC_DELETE and SUBNET_DETACH If volumes are attached, also need VOLUME_ATTACHMENT_DELETE and VOLUME_WRITE and INSTANCE_DETACH_VOLUME — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).

관리자는 실습용 identity domain/group을 확인한 뒤 아래를 **lab-root에만** 위임한다. `<domain>/<lab-group>`와 Compartment 이름은 실제 값으로 바꾼다. 이 정책은 네트워크 생성 실습용 scoped 예시이며 조직의 조건부 정책·Security Zone·quota와 함께 검토한다. 운영 tenancy 전체 `manage all-resources`는 쓰지 않는다.

```text
Allow group <domain>/<lab-group> to manage virtual-network-family in compartment lab-root
```

VCN/subnet/route/security list/NSG/gateway/public IP CRUD와 VNIC 사용·조회에 필요한 범위다. [Core Services resource type·verb·API 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm)를 확인한다. Cloud Shell 접근은 별도 관리자가 허용해야 한다. 실습자에게 정책 작성 권한을 자동 부여하지 않는다. 실제 권한 허용/거부 및 cross-compartment 사용 테스트는 미실행이다.

```text
Allow group <domain>/<lab-group> to manage instance-family in compartment lab-root
Allow group <domain>/<lab-group> to manage volume-family in compartment lab-root
```

인스턴스 launch/terminate 및 boot/block attachment·삭제용이다. Platform image 조회와 tenancy AD 조회가 추가로 필요하면 관리자가 해당 read/inspect 권한을 승인한다. 다른 Compartment의 image/network를 쓰는 경우 이 세 줄만으로 충분하다고 가정하지 않는다. VM guest에서는 기본 Linux 사용자 `opc`와 sudo 권한을 사용한다.

## 비용및주의사항

- 이 lab만 승인된 **vm-b 1개+boot/VNIC/ephemeral IP 1세트**를 추가한다. 전역 실행 VM 상한은 vm-a 포함 **2개**, extra VM은 동시 1개이며 private-vm/burstable/preemptible을 동시에 만들지 않는다. shape/boot 실제 크기와 종료 UTC를 Create 전에 승인한다. 09/16은 같은 B를 즉시 재사용하고 반환 후 B/boot를 삭제한다. Stop은 boot/backup/IP 비용 제거가 아니며 FD 분리는 application HA/DR 보장이 아니다.
- 생성/사용 당일 공식 단가와 Billing & Cost Management → Cost Analysis의 실제 Compartment/서비스 필터를 확인하되 청구 반영 지연을 기록한다. 예산 경보는 과금 차단이 아니다. 고객 데이터/secret/운영 자원을 사용하지 않는다.
- 이 문서는 Oracle 기술 본문 기반 교육 절차다. OCI/guest/IAM allow·deny 실측은 없으며 기대 결과를 실행 결과로 기록하지 않는다. 승인되지 않은 선택 확장은 실행하지 않는다.


## 실습구성

### 이번 Workshop에서 만드는 것과 진행 순서

같은 AD의 다른 FD에 vm-b 한 세트만 추가한다. 배치 분리이지 application HA/failover 구현이 아니다.

1. **진입/선택:** vm-a AD/FD readback. 사전준비 링크의 지정 단계/실제 출력이 없으면 제출하지 않는다.
2. **실제 작업:** 두 번째 VM 예산 → 다른 FD Create → 두 RUNNING/새 SSH. 아래 Console 필드/명령을 해당 host에서 순서대로 실행한다. 같은 자원은 이름이 아니라 OCID/event로 재사용한다.
3. **완료/복구/반환:** 즉시 소비자/expiry 확인. 결과검증 실패는 문제해결로 이동하고 동일 Create를 무작정 재제출하지 않는다.

**전달/정리 범위:** vm-b/boot만 owner07 정리; 09/16 즉시 소비자는 동일 OCID 재사용. 아래 정리 단계는 이 소유권을 넘어서 실행하지 않는다.


- **이번 생성/재사용/출력:** 신규 vm-b/VNIC/ephemeral IP/boot 1세트. 같은 AD 다른 FD. 09/16 소비자가 같은 승인 시간에 있으면 shared vm-b 사용 후 삭제.
- **생성 목표:** vm-a와 동일 AD의 다른 Fault Domain을 명시하여 vm-b를 생성한다. 각 FD는 선택되는 provider placement boundary이지 신규 OCI resource가 아니다.
- **완료 증거:** 생성 전 옵션 선택 + 실제 제출 대상 OCID + API readback + 해당 기능검증 + dependency 순서 정리. 실제 OCI 실행 checkpoint는 비워 둔다.
- **retention:** 예약 소비자와 expiry가 있는 resource만 보존. idle 유료 resource를 다음 주제 때문에 무기한 유지하지 않는다.


## 실습절차

### 명령 실행 gate — 실패하면 다음 생성으로 진행하지 않는다

1. 아래 Bash는 **전용 Bash 세션**에서만 사용한다. Console/Cloud Shell 조회, 본인 로컬 SSH, 로그인한 Oracle Linux guest를 섞지 않는다. `exit 1`은 이 전용 세션을 닫는 안전 중단이다. `<...>`·빈 입력·타 owner OCID가 남으면 실행하지 않는다.
2. 성공을 요구하는 파일 작성/조회/설치/attach 작업은 실패 시 중단한다. 실패한 API 뒤 기존 JSON이나 기존 다운로드 파일을 성공 근거로 읽지 않는다. 기대 deny는 `if command; then ...; else RC=$?; ...; fi`로 별도 기록하고, 실제 timeout/IAM 오류/로컬 파일 오류를 구별한다.
3. 첫 변경 전에 정확한 OCID·owner·현재 설정과 승인 expiry를 저장한다. 공유 자원 변경 중 다른 소비자가 시작하거나 원복값을 확인할 수 없으면 변경을 중단한다. 삭제 성공은 API 수락이 아니라 같은 identity/Region의 정확한 대상 재조회로 판정한다. `NotAuthorizedOrNotFound` 단독은 삭제 증거가 아니다.


**최대 두 VM 실행 gate:** vm-a 외 vm-b/burstable-vm/ip-lab-vm/preemptible-vm/private-vm는 같은 시간에 한 개만 실행한다. 이전 extra VM/boot 정리를 readback한 후 다음 extra를 생성한다. owner07 vm-b를 owner16이 쓰는 경우에는 새로운 VM을 만들지 않고 09 검증을 먼저 끝낸다.

### 개별 완주 보정 — 생성·검증·소유권

vm-b는 private-vm 생성 전 승인 세션에서 순차 사용한다. owner09/16 소비자가 즉시 있으면 같은 OCID를 유지하고 종료시 vm-b/boot만 삭제한다. same AD/different FD 실패는 BLOCKED_CAPACITY로 남긴다.

1. vm-a의 API readback에서 availability-domain/fault-domain을 읽는다. 같은 AD의 FD 3개는 논리적 hardware/maintenance 격리 그룹이며 사용자 생성 리소스가 아니다. [Fault Domains](https://docs.oracle.com/en-us/iaas/Content/General/Concepts/regions.htm).
2. **Create 이전 필수 gate:** owner17 YAML의 public-host-key 출력 runcmd와 hash를 확인하고 vm-b 생성 event에 연결한다. 누락이면 제출하지 않는다. 두 번째 VM 1개와 boot의 짧은 실행 예산/기한을 승인받는다. Compute → Create instance: Name `vm-b`, Compartment lab-root, **vm-a와 같은 AD**, **vm-a와 다른 FD**(예: 기존 FD1이면 FD2), On-demand. image/shape/키/public-admin/nsg-admin/boot는 [Compute 공유 launch station의 2–6단계](../02_Compute/00_Compute.md#shared-launch)에서 정한 승인값을 재사용한다. user-data도 동일한 비민감 marker YAML을 유지하며 vm-b용 event/OCID에 별도 hash를 연결한다; vm-a의 생성 timestamp를 복사하지 않는다. Name/FD/신규 VNIC·IP·boot 출력만 vm-b 대상으로 구분한다. 실제 Create 제출을 해야 한다. [Fault Domain placement](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/launchinginstance.htm).
3. vm-b RUNNING 뒤 첫 SSH 전에 [trust bootstrap](../02_Compute/00_Compute.md#guest-trust-bootstrap)을 **INSTANCE_B_FD_ID**로 완료한다. vm-a 값 재사용은 금지하고 handoff 없으면 BLOCKED_TRUST다. 결과검증의 별도 local fence에서 새 SSH를 확인한다. vm-a를 중단하거나 장애를 주입하지 않는다. 두 VM이 다른 FD라는 결과는 전체 application HA/DR이나 자동 failover를 입증하지 않는다.
4. 같은 AD에 다른 FD capacity가 없으면 오류와 미생성 상태를 기록한다. 다른 AD로 옮겨놓고 같은-AD 다른-FD 검증이라고 쓰지 않는다.

## 결과검증

<a id="baseline-07-ready"></a>

```bash
# Cloud Shell 전용: REGION은 승인 Region을 다시 지정한다.
oci compute instance get --instance-id '<vm-a_OCID>' --region "$REGION" > fd-a.json || exit 1
oci compute instance get --instance-id '<vm-b_OCID>' --region "$REGION" > fd-b.json || exit 1
python3 - <<'PY'
import json
a=json.load(open('fd-a.json'))['data'];b=json.load(open('fd-b.json'))['data']
assert a['lifecycle-state']==b['lifecycle-state']=='RUNNING'
assert a['availability-domain']==b['availability-domain']
assert a['fault-domain']!=b['fault-domain']
print('FD placement assertions passed')
PY
```

검증은 단계2에서 이미 제출한 owner17 YAML/hash와 단계3의 **INSTANCE_B_FD_ID** OCID/UTC/ED25519 handoff를 readback한다. 누락이면 BLOCKED_TRUST이며 생성 이후 YAML 제출로 소급 보정하지 않는다. vm-a의 fingerprint는 재사용하지 않는다.

```bash
# 본인 Linux/WSL 로컬 전용 Bash; 위 API/Python fence는 Cloud Shell에서 종료한다.
export KEY='<owner15_actual_local_KEY>'
export PUBLIC_IP='<INSTANCE_B_FD_ID_actual_public_IP>'
timeout --foreground 60s ssh -i "$KEY" -o IdentitiesOnly=yes -o HostKeyAlgorithms=ssh-ed25519 -o StrictHostKeyChecking=yes -o ConnectTimeout=10 -o ConnectionAttempts=1 -o ControlMaster=no -o ControlPath=none opc@"$PUBLIC_IP" 'test "$(id -un)" = opc && hostname' > consumer07-ssh.txt || exit 1
date -u +%FT%TZ > consumer07-session-ended.txt || exit 1
```
consumer07-ssh.txt의 hostname을 B trust bootstrap의 같은 OCID guest hostname 인계와 비교하고 기록한다. hostname은 단독 instance identity 증거가 아니라 B actual OCID↔현재 VNIC public IP↔serial ED25519 fingerprint↔SSH stdout/UTC 조합의 acceptance다. 미일치/누락은 BLOCKED_IDENTITY이며 A hostname을 비교값으로 복사하지 않는다. 기대 패턴 동일 AD/다른 FD, vm-b SSH exit 0/opc. 실제 장애복구 시험은 수행하지 않는다.

**증거 기록:** 실제 명령·UTC 시간·exit code·가려진 OCID·옵션 readback·검증 조건을 남긴다. 현재 이 문서의 실행상태는 `NOT_EXECUTED`다. output/creation-event-id/result는 실행 전 비워 둔다. 오류/NOT_SELECTED를 PASS로 바꾸지 않는다.

### 이 시나리오의 functional acceptance

| Positive | Negative / 미완료 판정 | Recovery / 반환 |
|---|---|---|
| same AD/different FD; vm-b SSH | 같은 FD 또는 다른 AD 결과면 placement 불합격 | 미생성 시 승인 FD 재선택; 이미 생성이면 owner 비용/cleanup 승인 후 재작업; 장애주입 없음 |

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

FD capacity 오류는 같은 AD의 다른 승인 FD로 재선택하거나 BLOCKED_CAPACITY로 기록한다. vm-b가 다른 AD에 생성되면 같은-AD 다른-FD 검증을 통과시키지 않는다.

## 리소스정리

### owner07 B/boot만 finite terminate closeout

1. owner09/16 consumer 반환 record(같은 INSTANCE_B_FD_ID, session-end UTC, evidence, outstanding changes=none, lease RELEASED, owner07 acknowledgment)를 모두 받고 이후 소비자가 없음을 확인한다. 미반환/expiry 초과는 consumer abort/반환 또는 별도 예산승인으로 해결한다. vm-a/boot-a 및 owner22 private-vm에는 손대지 않는다.
2. **Terminate 전** Console → Compute → Instances → vm-b **actual OCID** → Storage/Boot volume에서 연결 boot OCID/AD/compartment를 읽어 B creation record에 저장한다. guest workload/mount가 생겼으면 해당 consumer가 종료/unmount/detach 반환한 뒤 진행한다. 이 FD lab 자체는 data volume을 추가하지 않는다. Cloud Shell에서 새 private evidence 디렉터리로 같은 instance/boot를 Get하여 owner07/expiry/AD를 대조한다.

```bash
set -euo pipefail
export INSTANCE_B_FD_ID='<owner07_actual_vm-b_OCID>'
export BOOT_B_ID='<위_Console_Storage_actual_B_boot_OCID>'
export AD_B='<위_B_actual_AD_name>'
export LAB_COMPARTMENT_ID='<actual_lab-root_OCID>'
FD_CLOSE=$(mktemp -d "$HOME/owner07-close.XXXXXXXX") || exit 1
cd "$FD_CLOSE" || exit 1
oci compute instance get --instance-id "$INSTANCE_B_FD_ID" --region "$REGION" > b-before.json || exit 1
oci bv boot-volume get --boot-volume-id "$BOOT_B_ID" --region "$REGION" > boot-before.json || exit 1
python3 - <<'PY'
import json,os
b=json.load(open('b-before.json'))['data'];v=json.load(open('boot-before.json'))['data']
assert b['id']==os.environ['INSTANCE_B_FD_ID'] and b['id']!=os.environ['INSTANCE_A_ID']
assert b['availability-domain']==v['availability-domain']==os.environ['AD_B']
assert v['id']==os.environ['BOOT_B_ID'] and v['compartment-id']==os.environ['LAB_COMPARTMENT_ID']
PY
```

관리자는 GetBootVolume/ListBootVolumes의 **VOLUME_INSPECT**, 수동 잔존 boot 삭제가 필요한 경우 DeleteBootVolume **VOLUME_DELETE**, 기존 TerminateInstance/dependency permission을 승인한다. [Core operation 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm). IAM deny는 부재가 아니다.
3. Console Instances에서 exact B → Actions → **Terminate** → **Permanently delete the attached boot volume 체크**(기본은 preserve이므로 생략 금지) → **Terminate Instance 한 번**. 승인되지 않은 Block Volume(s) 삭제 옵션은 선택하지 않는다. [Terminating an Instance task](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/terminatinginstance.htm). 삭제는 회복 불가이며 IP는 나중에 해제될 수 있다.
4. Cloud Shell에서 exact B만 10초 간격/600초 교육용 상한으로 관찰한다. API당 timeout/retry 제한과 실패 중단을 적용한다.

```bash
DEADLINE=$((SECONDS+600)); N=0; TERMINATED=0
while [ "$SECONDS" -lt "$DEADLINE" ]; do
  N=$((N+1))
  oci compute instance get --instance-id "$INSTANCE_B_FD_ID" --region "$REGION" --connection-timeout 10 --read-timeout 20 --max-retries 0 > "b-after-$N.json" 2> "b-after-$N.error" || exit 1
  STATE=$(python3 - "b-after-$N.json" <<'PY'
import json,os,sys
x=json.load(open(sys.argv[1]))['data'];assert x['id']==os.environ['INSTANCE_B_FD_ID']
print(x['lifecycle-state'])
PY
) || exit 1
  if [ "$STATE" = TERMINATED ]; then TERMINATED=1; break; fi
  case "$STATE" in RUNNING|STOPPED|STOPPING|TERMINATING) ;; *) printf 'BLOCKED_TERMINATE: %s\n' "$STATE"; exit 1;; esac
  sleep 10
done
test "$TERMINATED" = 1 || { printf '%s\n' 'BLOCKED_TERMINATE_DEADLINE'; exit 1; }
```

5. Console → Storage → Block Storage → **Boot Volumes**에서 REGION/lab-root/AD_B를 정확히 선택하고 lifecycle 필터를 해제하여 모든 페이지를 읽는다. 저장한 BOOT_B_ID의 deleted 상태 또는 목록 부재를 관리자가 확인한다. GetBootVolume도 같은 identity/Region/exact OCID로 읽어 JSON/error/request-id를 보존한다. NotAuthorizedOrNotFound만이면 **BLOCKED_BOOT_VISIBILITY**, 관리자 전체보기 확인 전 삭제 완료로 처리하지 않는다. 10초 간격 600초 한도로 관찰한다. 상한 후 **AVAILABLE로 남는 같은 boot**이면 owner07 삭제 재승인과 새 GetBootVolume으로 소유권/AD를 확인하고, Console 해당 boot의 Attached instances 목록에서 연결 없음도 별도 확인한다(GetBootVolume만으로 attachment 없음을 주장하지 않는다). 그 boot Actions → **Terminate** → exact ID 확인 후 confirmation 한 번 → 같은 600초 observation. [Deleting a Boot Volume 현재 Console/CLI task](https://docs.oracle.com/en-us/iaas/Content/Block/Tasks/delete-bv-boot-volume.htm). attached/unknown/deleting/IAM 오류이면 수동 삭제하지 않는다. 권한/timeout/예상밖 state에는 마지막 evidence·B/boot/AD/consumer 반환/request-id/UTC를 owner07/관리자에게 넘기며 Terminate를 재제출하지 않는다.
6. actual TERMINATED UTC 및 boot closeout UTC/evidence/visibility acknowledgment를 registry에 남겨 owner09 ledger의 실제 deleted_at 갱신 근거를 제공한다. instance 기록은 Terminated로 최소 12시간 표시될 수 있으며 보임 자체가 실패가 아니다. FD_CLOSE와 local SSH 증거는 승인 보존경로에서 hash 확인 후 이번 생성 exact파일만 정리/rmdir한다. 공유 vm-a/network/KEY는 보존한다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 아래 본문은 각 생성 필드·설정·검증·정리의 기술 근거이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [23] [Regions and Availability Domains — Fault Domains](https://docs.oracle.com/en-us/iaas/Content/General/Concepts/regions.htm)
  - 확인 범위: A fault domain is a grouping.

### 추가 실행 절차 근거 (본문 확인)

- [regions](https://docs.oracle.com/en-us/iaas/Content/General/Concepts/regions.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [launch](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/launchinginstance.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [connect](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/connect-to-linux-instance.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [cloudshell](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/cloudshellgettingstarted.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
