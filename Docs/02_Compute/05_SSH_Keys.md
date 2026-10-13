# SSH Keys

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../README.md)를 확인하세요.

> 실행 절차 작성본 / OCI 미실행. Oracle 공식 본문 근거와 교육용 선택을 구분한다. secrets·실제 고객값·diagram 없음.

## 학습목표

- 로컬 전용 Ed25519 키를 기존 파일 덮어쓰기 없이 생성하고 permission 400과 대응 public fingerprint를 확인한다.
- 공개키만 owner10 launch에 제출하고 새 SSH 두 세션을 검증한다. private key는 VM에 복사하지 않고 모든 소비자 종료 후만 정리한다.

## 대응 Roadmap 항목

- 항목명: **SSH Keys**
- Roadmap 경로: **Compute → SSH Keys**
- Node ID: `sequence-2-5`
- 상위 항목: **Compute** (`sequence-2-0`)
- 문서 범위: 로컬 전용 Ed25519 키를 기존 파일 덮어쓰기 없이 생성하고 permission 400과 대응 public fingerprint를 확인한다. 공개키만 owner10 launch에 제출하고 새 SSH 두 세션을 검증한다. private key는 VM에 복사하지 않고 모든 소비자 종료 후만 정리한다. 실제 OCI 실행 결과가 아니라 이 절차의 완료 조건이다.
- 번호는 navigation 식별자이며 실행 순서는 사전준비의 단계별 의존성에 따른다.

## 사전준비

### 기준 계약과 실제 단계 선행조건

[공통 baseline-contract.json](../../baseline-contract.json)의 Oracle Linux 9 / opc / CIDR / 비용 상한을 따른다. 문서 번호는 실행 순서가 아니다. **준비 station과 생성 이후 ready를 분리**한다. 아래 단계 외의 기능완료를 소급하여 요구하지 않는다.

- [10 Compute — 단계 링크](00_Compute.md#shared-launch): **Part B에서만 필요; Part A는 로컬 OpenSSH만**. 실제 출력/소비자/expiry를 전달하고 해당 owner가 이 lab 종료까지 보존한다.

생성 전 조직 승인(identity/권한/예산/expiry/서비스 지원/limit/capacity)을 확인한다. 실제 OCID/IP는 아직 생성하지 않았으면 비워 두고 해당 Create/readback 뒤 채운다. 동일 creation-event-id/OCID는 한 번의 제출만 뜻한다.  실습에 없는 자원을 추가하지 않는다. 실제 checkpoint는 사용자 실행 전 비워 둔다.

### 로컬 실행 입력

Part A는 Linux/WSL Bash에서 OpenSSH·GNU stat/timeout과 본인 .ssh 디렉터리만 준비한다. KEY는 아래 절차가 정하는 새 절대경로이며 기존 파일을 재사용해 생성하지 않는다. Part B에만 owner10의 같은 vm-a PUBLIC_IP/OCID와 독립 확인 guest fingerprint를 인수한다. Cloud Shell/namespace/API credential은 Part A의 입력이 아니다.

### Workshop 입력 계약 — 단계와 host

| 입력/출력 | 값 또는 실제 획득 경로 | 사용할 시점 | 실행 위치 | 선택 이유 |
|---|---|---|---|---|
| Console/설정 필드 | Local filename=전용 KEY; Ed25519; passphrase interactive; private mode400; .pub만 Console 제출 | 생성 전 선택 → 생성 뒤 readback | 본문 Console/로컬 지정 | 이 topic의 최소 생성 범위 |
| `KEY` | owner15 Part A 전용 private key 절대경로; 내용 전송 금지 | 준비 완료 | 로컬 SSH만 | 해당 공개키와 대응 |
| `PUBLIC_IP` | A_PUBLIC_IP / owner10 VNIC public-ip | 생성 후/선행 완료 | 로컬 SSH | 대상 host 주소 |

생성 전에는 행의 승인값/선행 완료값만 지정하고, 생성 후 ID/IP는 해당 owner 응답으로 채운다. Cloud Shell과 로컬 SSH의 환경변수는 공유되지 않는다. 로컬에는 KEY와 접속 주소만 다시 지정하고 guest disk 명령은 로그인한 guest에서만 실행한다. `<...>`가 남으면 해당 명령은 실행하지 않는다.

## 권한

Part A의 로컬 ssh-keygen/stat/chmod는 OCI API 권한이 필요 없다. 본인 전용 KEY/KEY.pub 읽기·생성 권한만 사용한다. Part B의 공개키 제출은 owner10의 [공동 launch](00_Compute.md#shared-launch) 권한으로 수행하고 본인 local key로 opc SSH한다. Cloud Shell/identity group/network/volume manage 권한을 key 생성 때문에 요청하지 않는다. guest authorized_keys의 다른 사용자 key는 변경하지 않는다.

## 비용및주의사항

- Part A의 로컬 Ed25519 key pair 생성은 OCI 자원 생성이 아니다. Part B는 owner10의 기존 vm-a만 사용하며 새 VM/VCN/IP를 만들지 않는다. Compute/boot 유지 비용은 owner10 승인 창에 포함한다.
- private key/passphrase는 로컬에만 둔다. 기존 KEY/KEY.pub나 symlink가 있으면 chmod/keygen 전에 중단한다. 모든 소비 VM 종료·key 반환 확인 후 전용 두 파일만 정리하며 공용 .ssh/known_hosts 전체를 삭제하지 않는다.

실제 OCI 작업은 아직 수행하지 않았다. 기대값은 실행 시 대조할 조건이며 무료 사용이나 과금 강제 차단을 보장하지 않는다. 승인 expiry가 끝나면 새 시험을 중단하고 위 소유권 순서로 원복·반환한다.

## 실습구성

### 이번 Workshop에서 만드는 것과 진행 순서

로컬 전용 Ed25519 키를 먼저 만들고 같은 공개키를 vm-a에 제출한 뒤 새 SSH를 검증한다. OCI managed key resource는 없다.

1. **진입/선택:** Part A 기존 파일 guard/keygen/권한/fingerprint. 사전준비 링크의 지정 단계/실제 출력이 없으면 제출하지 않는다.
2. **실제 작업:** owner10 공개키 제출. 아래 Console 필드/명령을 해당 host에서 순서대로 실행한다. 같은 자원은 이름이 아니라 OCID/event로 재사용한다.
3. **완료/복구/반환:** Part B 신뢰된 host fingerprint/새 SSH. 결과검증 실패는 문제해결로 이동하고 동일 Create를 무작정 재제출하지 않는다.

**전달/정리 범위:** KEY/.pub/fingerprint; private key local only; 모든 소비 VM 종료 후 전용 두 파일만 정리. 아래 정리 단계는 이 소유권을 넘어서 실행하지 않는다.


- **이번 생성/재사용/출력:** 신규: 전용 local Ed25519 key pair. Part A 출력 KEY/.pub/fingerprint; Part B에서 vm-a와 새 SSH 연결. bootstrap cycle 없음.
- **생성 목표:** 실제 SSH key material 생성이 primary. Part A는 cloud/network 없이 먼저 수행; Part B는10 생성에 public key를 제출하고 private key로 실제 접속한다. OCI SSH Key resource를 꾸미지 않는다.

## 실습절차

### 명령 실행 gate — 실패하면 다음 생성으로 진행하지 않는다

1. 아래 Bash는 **전용 Bash 세션**에서만 사용한다. Console/Cloud Shell 조회, 본인 로컬 SSH, 로그인한 Oracle Linux guest를 섞지 않는다. `exit 1`은 이 전용 세션을 닫는 안전 중단이다. `<...>`·빈 입력·타 owner OCID가 남으면 실행하지 않는다.
2. 성공을 요구하는 파일 작성/조회/설치/attach 작업은 실패 시 중단한다. 실패한 API 뒤 기존 JSON이나 기존 다운로드 파일을 성공 근거로 읽지 않는다. 기대 deny는 `if command; then ...; else RC=$?; ...; fi`로 별도 기록하고, 실제 timeout/IAM 오류/로컬 파일 오류를 구별한다.
3. 첫 변경 전에 정확한 OCID·owner·현재 설정과 승인 expiry를 저장한다. 공유 자원 변경 중 다른 소비자가 시작하거나 원복값을 확인할 수 없으면 변경을 중단한다. 삭제 성공은 API 수락이 아니라 같은 identity/Region의 정확한 대상 재조회로 판정한다. `NotAuthorizedOrNotFound` 단독은 삭제 증거가 아니다.


### 개별 완주 보정 — 생성·검증·소유권

기존KEY/KEY.pub 또는 keygen실패는 exit1로 chmod전에 종료한다. 통합 OS는 OL9/opc다. privatecontents/passphrase는 기록하지 않고 임시client파일만 보존기한에 맞춰 정리한다.

<a id="key-preparation"></a>

### Part A — 인스턴스 없이 key 생성 (Compute의 필수 선행)

1. 본인 **Linux/WSL의 Bash + GNU coreutils + OpenSSH**를 사용한다. 아래 stat -c/timeout은 GNU 기준이며 macOS/Windows native에 그대로 실행하지 않는다. Bash/stat/timeout/ssh/ssh-keygen preflight가 실패하면 BLOCKED_HOST다. 조직의 passphrase 보관 규칙을 확인하고 전용 filename을 정한다. 기존 `id_rsa`/`id_ed25519`를 덮어쓰지 않는다. [Managing Key Pairs — supported platform image key types](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/managingkeypairs.htm).

```bash
set -euo pipefail
for tool in stat timeout ssh ssh-keygen; do command -v "$tool" >/dev/null || exit 1; done
stat --version >/dev/null || exit 1
ssh -V
export KEY="$HOME/.ssh/oci-roadmap-lab-ed25519"
if test -e "$KEY" || test -L "$KEY" || test -e "$KEY.pub" || test -L "$KEY.pub"; then
  printf '%s\n' 'STOP: key already exists; choose another filename'
  exit 1
else
# collision 검사가 끝난 뒤에만 directory 생성; 기존 mode는 변경하지 않는다.
if test -L "$HOME/.ssh"; then printf '%s\n' 'STOP: .ssh symlink needs owner review'; exit 1; fi
if test -e "$HOME/.ssh"; then
  test -d "$HOME/.ssh" || exit 1
  test "$(stat -c '%a' "$HOME/.ssh")" = 700 || { printf '%s\n' 'STOP: existing .ssh mode requires separate owner approval'; exit 1; }
else
  (umask 077; mkdir "$HOME/.ssh") || exit 1
fi

  ssh-keygen -t ed25519 -f "$KEY" -C 'oci-roadmap-lab' || exit 1
fi
chmod 400 "$KEY"
ssh-keygen -lf "$KEY.pub"
stat -c '%a %n' "$KEY" "$KEY.pub"
```
2. `ssh-keygen`의 passphrase는 interactive prompt에서만 입력한다. 문서/명령 인자/chat/git에 쓰지 않는다. `KEY` 파일은 private, `.pub`는 Console 제출용 public이다. fingerprint와 생성경로만 기록하고 private contents는 읽거나 공유하지 않는다.
3. Part A의 출력은 공개키 파일 경로, SHA256 fingerprint, private permission 400, KEY 환경변수다. 이 단계는 **OCI VM/VCN/SSH 접속 성공을 요구하지 않는다**. Compute 사전준비는 여기까지만 요구한다.

### Part B — Compute 생성 후 새 SSH 세션 (후속 검증)

4. Compute lab의 launch에서 본인 `.pub`를 업로드한다. VM이 RUNNING이 되고 `PUBLIC_IP`가 확인되면 같은 로컬 터미널에서 아래를 실행한다. passphrase는 interactive prompt에서 입력한다. [owner10 guest trust bootstrap](00_Compute.md#guest-trust-bootstrap)을 같은 vm-a OCID로 먼저 완료한다. 최초 ask 접속에서 기록된 ED25519 fingerprint와 정확히 비교한 뒤 수락한다; 불일치면 BLOCKED_TRUST다. [Connecting to a Linux Instance](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/connect-to-linux-instance.htm).

```bash
export PUBLIC_IP='<vm-a_실제_public_IP>'
timeout --foreground 60s ssh -i "$KEY" -o IdentitiesOnly=yes -o HostKeyAlgorithms=ssh-ed25519 -o StrictHostKeyChecking=ask -o ConnectTimeout=10 -o ConnectionAttempts=1 -o ControlMaster=no -o ControlPath=none opc@"$PUBLIC_IP" 'id; uname -m; hostname'
```
5. 이후 비파괴 로그인을 새 세션으로 한 번 더 검증한다. Console generated key를 대신 쓴다면 생성 시 한 번 제공되는 private key를 보관해야 한다. Oracle이 private key 사본을 제공한다고 가정하지 않는다.

### 안전 거부/복구 — 기존 key를 다시 생성하지 않는다

1. Part A의 key가 이미 존재하는 상태에서 **별도 전용 Bash**로 existing-file guard까지만 다시 실행한다. STOP과 exit1이 기대 패턴이다. `ssh-keygen`/chmod에는 도달하면 안 된다. 전·후 public fingerprint와 private file mode가 동일한지 `ssh-keygen -lf "$KEY.pub"; stat -c '%a %n' "$KEY"`로 비교한다.
2. 기존 key가 정상이고 fingerprint가 맞으면 **새 key를 만들지 않고** Part B의 같은 KEY로 새 SSH를 다시 연다. 복구는 정상 key를 재사용하는 것이며 private 파일 contents를 출력하지 않는다. 실제 key 손실/guest authorized_keys 변경은 관리자 복구 branch로만 수행한다.

## 결과검증

<a id="baseline-15-ready"></a>

```bash
set -euo pipefail
ssh-keygen -y -f "$KEY" | ssh-keygen -lf -
ssh-keygen -lf "$KEY.pub"
timeout --foreground 60s ssh -i "$KEY" -o IdentitiesOnly=yes -o HostKeyAlgorithms=ssh-ed25519 -o StrictHostKeyChecking=yes -o ConnectTimeout=10 -o ConnectionAttempts=1 -o ControlMaster=no -o ControlPath=none opc@"$PUBLIC_IP" 'test "$(id -un)" = opc && test -r /etc/os-release'
printf 'ssh exit status=%s
' "$?"
```
Part A 기대: public key fingerprint 두 개가 같고 private permission 400. Part B 기대: SSH exit 0와 사용자 opc. 이 통합 트랙은 Oracle Linux 9 / opc로 고정하며 Ubuntu 전환을 요구하지 않는다. Part A 완료와 Part B 완료를 다른 checkpoint로 기록한다.

**증거 기록:** 실제 명령·UTC 시간·exit code·가려진 OCID·옵션 readback·검증 조건을 남긴다. 현재 이 문서의 실행상태는 `NOT_EXECUTED`다. output/creation-event-id/result는 실행 전 비워 둔다. 오류/NOT_SELECTED를 PASS로 바꾸지 않는다.

### 이 시나리오의 functional acceptance

| Positive | Negative / 미완료 판정 | Recovery / 반환 |
|---|---|---|
| key pair fingerprint/permission400/새 SSH | existing-key guard STOP; contents/mode 불변 | 같은 정상 KEY 재사용; 관리자 serial 복구 이외 guest key 변경 없음 |

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

STOP existing key 메시지가 나오면 전용 다른 KEY path를 선택한 뒤 새 shell에서 Part A를 다시 시작한다. 기존 key의 내용·권한을 이 lab 때문에 변경하지 않는다. fingerprint 불일치는 public/private pair가 다른 경우이므로 Compute에 잘못 제출한 key를 확인한다.

## 리소스정리

Part B의 소비 VM이 모두 terminate되고 해당 공개키 접근이 더 이상 필요하지 않을 때만 이 lab 전용 `KEY`와 `KEY.pub`를 삭제한다. 기존 조직 key는 삭제하지 않는다. private key를 jump VM에 복사하지 않았음을 확인한다. 키 손실은 관리자 승인된 serial recovery 또는 새 VM/key 재생성으로 처리하며 문서에 private key를 남기지 않는다. SSH key는 독립 OCI managed resource가 아니라 로컬 파일/guest authorized key다.


## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 아래 본문은 각 생성 필드·설정·검증·정리의 기술 근거이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [16] [Creating an Instance — SSH Keys](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/launchinginstance.htm)
  - 확인 범위: Public SSH key.

### 추가 실행 절차 근거 (본문 확인)

- [keys](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/managingkeypairs.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [connect](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/connect-to-linux-instance.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [cloudshell](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/cloudshellgettingstarted.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
