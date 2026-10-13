# Cloud-init / User Data

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../README.md)를 확인하세요.

> 실행 절차 작성본 / OCI 미실행. Oracle 공식 본문 근거와 교육용 선택을 구분한다. secrets·실제 고객값·diagram 없음.

## 학습목표

- 전용 cloud-config 원문을 launch 전에 작성하고 size/hash를 확인하여 owner10의 최초 vm-a 생성 요청에 업로드한다.
- guest cloud-init status·합성 marker·UTC timestamp/log를 읽어 최초 boot 적용을 확인한다. clean/reboot 재실행으로 성공을 꾸미지 않는다.

## 대응 Roadmap 항목

- 항목명: **Cloud-init / User Data**
- Roadmap 경로: **Compute → Cloud-init / User Data**
- Node ID: `sequence-2-7`
- 상위 항목: **Compute** (`sequence-2-0`)
- 문서 범위: 전용 cloud-config 원문을 launch 전에 작성하고 size/hash를 확인하여 owner10의 최초 vm-a 생성 요청에 업로드한다. guest cloud-init status·합성 marker·UTC timestamp/log를 읽어 최초 boot 적용을 확인한다. clean/reboot 재실행으로 성공을 꾸미지 않는다. 실제 OCI 실행 결과가 아니라 이 절차의 완료 조건이다.
- 번호는 navigation 식별자이며 실행 순서는 사전준비의 단계별 의존성에 따른다.

## 사전준비

### 기준 계약과 실제 단계 선행조건

[공통 baseline-contract.json](../../baseline-contract.json)의 Oracle Linux 9 / opc / CIDR / 비용 상한을 따른다. 문서 번호는 실행 순서가 아니다. **준비 station과 생성 이후 ready를 분리**한다. 아래 단계 외의 기능완료를 소급하여 요구하지 않는다.

- [15 SSH Keys — 단계 링크](05_SSH_Keys.md#key-preparation): **Part A 키/로컬 파일 접근**. 실제 출력/소비자/expiry를 전달하고 해당 owner가 이 lab 종료까지 보존한다.
- [10 Compute — 단계 링크](00_Compute.md#shared-launch): **파일 작성 뒤 공동 launch Management station; 완료 VM 불필요**. 실제 출력/소비자/expiry를 전달하고 해당 owner가 이 lab 종료까지 보존한다.

생성 전 조직 승인(identity/권한/예산/expiry/서비스 지원/limit/capacity)을 확인한다. 실제 OCID/IP는 아직 생성하지 않았으면 비워 두고 해당 Create/readback 뒤 채운다. 동일 creation-event-id/OCID는 한 번의 제출만 뜻한다.  실습에 없는 자원을 추가하지 않는다. 실제 checkpoint는 사용자 실행 전 비워 둔다.

### 입력값과 실행 위치

Console에서 선택한 값만 사용한다. `<...>`는 예시 OCID나 비밀값이 아니라 **사용자가 대체할 입력 자리**다. 대체하지 않은 채 명령을 실행하지 않는다. Console → 우측 상단 Cloud Shell/Code Editor → Cloud Shell을 열면 OCI CLI가 사전 인증된다. Region 변경 뒤에는 Shell을 다시 열거나 명령의 `--region`을 바꾼다. SSH는 Cloud Shell이 아니라 본인 Linux/WSL/macOS 터미널에서 수행한다. 공인 IP/32도 그 터미널의 출구 주소여야 한다.

```bash
# 아래는 입력/출력의 대입 형식이다. 표의 단계 전에 생성 후 ID/IP export 행을 실행하지 않는다.
 # 생성 후 검증용 ID/IP는 해당 creator의 실제 응답에서 넣는다.
export KEY='<actual_KEY>'
export A_PUBLIC_IP='<actual_A_PUBLIC_IP>'
export PUBLIC_IP="$A_PUBLIC_IP"
```

이 변수 중 해당 절차에서 사용하는 값만 채운다. CLI 조회의 `data`를 증거 파일에 저장하고 실제 시간·작업 OCID·선택값을 기록한다. API key/private key/passphrase/인증 토큰은 기록하지 않는다. Cloud Shell 설정파일이나 세션 토큰을 출력하지 않는다. `oci --version`을 실행하는 주제의 기대 패턴은 버전 문자열뿐이다. namespace는 Object Storage 주제에서 별도 조회하며 다른 주제에는 요구하지 않는다. [Cloud Shell 공식 본문](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/cloudshellgettingstarted.htm).

### Workshop 입력 계약 — 단계와 host

| 입력/출력 | 값 또는 실제 획득 경로 | 사용할 시점 | 실행 위치 | 선택 이유 |
|---|---|---|---|---|
| Console/설정 필드 | 전용 cloud-config.yaml 원문; marker/UTC runcmd; metadata 합계<=32000 bytes; 최초 owner10 launch; secrets/reboot 없음 | 생성 전 선택 → 생성 뒤 readback | 본문 Console/로컬 지정 | 이 topic의 최소 생성 범위 |
| `A_PUBLIC_IP` | owner10 vm-a VNIC public-ip | 선행 완료/생성 후 | 로컬 SSH | 관리 jump 주소 |
| `KEY` | owner15 Part A 전용 private key 절대경로; 내용 전송 금지 | 준비 완료 | 로컬 SSH만 | 해당 공개키와 대응 |
| `PUBLIC_IP` | A_PUBLIC_IP / owner10 VNIC public-ip | 생성 후/선행 완료 | 로컬 SSH | 대상 host 주소 |

생성 전에는 행의 승인값/선행 완료값만 지정하고, 생성 후 ID/IP는 해당 owner 응답으로 채운다. Cloud Shell과 로컬 SSH의 환경변수는 공유되지 않는다. 로컬에는 KEY와 접속 주소만 다시 지정하고 guest disk 명령은 로그인한 guest에서만 실행한다. `<...>`가 남으면 해당 명령은 실행하지 않는다.

## 권한

로컬 cloud-config 원문/size/hash 작성은 OCI IAM 작업이 아니다. 업로드와 최초 vm-a launch는 owner10의 [공동 launch](00_Compute.md#shared-launch)에서 승인받은 LaunchInstance·VNIC/subnet/boot 의존 권한을 사용한다. guest cloud-init status/log/marker 읽기는 opc/sudo다. 이 consumer는 metadata update·cloud-init clean·VM 재생성 권한을 요구하지 않는다.

## 비용및주의사항

- 신규 클라우드 자원은 없다. 원문 YAML을 owner10의 단일 vm-a launch에 넣으며 별도 cloud-init 검증 VM이나 metadata 재실행을 만들지 않는다. 비용은 같은 vm-a/boot 보존 창이다.
- metadata 전체32,000 bytes 제한과 prelaunch hash를 확인한다. secrets/reboot/cloud-init clean을 넣지 않는다. marker/UTC/public host fingerprint만 생성한다. warning이면 로그를 보존하고 무승인 재생성하지 않으며 local YAML과 두 guest marker 파일은 자기 소유 범위에서 정리한다.

실제 OCI 작업은 아직 수행하지 않았다. 기대값은 실행 시 대조할 조건이며 무료 사용이나 과금 강제 차단을 보장하지 않는다. 승인 expiry가 끝나면 새 시험을 중단하고 위 소유권 순서로 원복·반환한다.

## 실습구성

### 이번 Workshop에서 만드는 것과 진행 순서

전용 cloud-config 원문을 최초 vm-a launch에 넣고 실제 guest marker/UTC timestamp/log를 확인한다. metadata 변경/clean 재실행은 대체 성공이 아니다.

1. **진입/선택:** 새 YAML 파일/크기/hash. 사전준비 링크의 지정 단계/실제 출력이 없으면 제출하지 않는다.
2. **실제 작업:** owner10 Management 원문 업로드 → 단일 Create → cloud-init 상태. 아래 Console 필드/명령을 해당 host에서 순서대로 실행한다. 같은 자원은 이름이 아니라 OCID/event로 재사용한다.
3. **완료/복구/반환:** marker/timestamp/log. 결과검증 실패는 문제해결로 이동하고 동일 Create를 무작정 재제출하지 않는다.

**전달/정리 범위:** local YAML/hash와 guest 두 파일; VM owner10 유지, clean/reboot 금지. 아래 정리 단계는 이 소유권을 넘어서 실행하지 않는다.


- **이번 생성/재사용/출력:** local cloud-config + vm-a 최초 launch의 user-data 실행. 별도 VM 없음. 로컬 hash와 실제 guest marker/log output 연결.
- **생성 목표:** Compute의 단일 실제 생성 event에서 Cloud-init / User Data 옵션을 직접 선택하고 저장·기능 검증한다. 독립 중복 생성은 하지 않는다.

## 실습절차

### 명령 실행 gate — 실패하면 다음 생성으로 진행하지 않는다

1. 아래 Bash는 **전용 Bash 세션**에서만 사용한다. Console/Cloud Shell 조회, 본인 로컬 SSH, 로그인한 Oracle Linux guest를 섞지 않는다. `exit 1`은 이 전용 세션을 닫는 안전 중단이다. `<...>`·빈 입력·타 owner OCID가 남으면 실행하지 않는다.
2. 성공을 요구하는 파일 작성/조회/설치/attach 작업은 실패 시 중단한다. 실패한 API 뒤 기존 JSON이나 기존 다운로드 파일을 성공 근거로 읽지 않는다. 기대 deny는 `if command; then ...; else RC=$?; ...; fi`로 별도 기록하고, 실제 timeout/IAM 오류/로컬 파일 오류를 구별한다.
3. 첫 변경 전에 정확한 OCID·owner·현재 설정과 승인 expiry를 저장한다. 공유 자원 변경 중 다른 소비자가 시작하거나 원복값을 확인할 수 없으면 변경을 중단한다. 삭제 성공은 API 수락이 아니라 같은 identity/Region의 정확한 대상 재조회로 판정한다. `NotAuthorizedOrNotFound` 단독은 삭제 증거가 아니다.


### 개별 완주 보정 — 생성·검증·소유권

user-data-preparation은 launch 전이며 이미KEY를 준비해야 한다. 전용YAML 경로가 있으면 덮어쓰지 않고 새 경로를 택한다. cloud-init warning은 원인검토 후 남기며 clean/reboot로 결과를 조작하지 않는다.

<a id="user-data-preparation"></a>

### 생성 전 user-data 준비

1. Compute launch **이전**에 아래 파일을 본인 전용 lab 디렉터리에 저장한다. 이미 vm-a를 만든 뒤 metadata를 바꿔놓고 최초 boot 실행 성공이라고 쓰지 않는다. 놓쳤다면 이 checkpoint는 미완료이며 별도 VM 재생성은 비용 승인을 받은 경우만 한다.

```bash
# 본인 Linux/WSL 로컬 전용 Bash; HOME은 본인 소유의 승인 real directory여야 한다.
set -euo pipefail
for tool in python3 stat readlink; do command -v "$tool" >/dev/null || exit 1; done
test ! -L "$HOME" && test "$(readlink -f "$HOME")" = "$HOME" && test -O "$HOME" || exit 1
LAB_DIR="$HOME/oci-roadmap-lab"
test ! -L "$LAB_DIR" || { printf '%s\n' 'STOP: lab parent symlink'; exit 1; }
if test -e "$LAB_DIR"; then
  test -d "$LAB_DIR" && test -O "$LAB_DIR" || exit 1
  test "$(stat -c '%a' "$LAB_DIR")" = 700 || { printf '%s\n' 'STOP: parent mode requires owner approval'; exit 1; }
else
  (umask 077; mkdir "$LAB_DIR") || exit 1
fi
test ! -e "$LAB_DIR/cloud-config.yaml" && test ! -L "$LAB_DIR/cloud-config.yaml" || { printf '%s\n' 'STOP: cloud-config exists or is symlink'; exit 1; }
# noclobber prevents a late existing-file collision; private owner-only parent prevents other-user replacement.
set -C
umask 077
cat > "$LAB_DIR/cloud-config.yaml" <<'YAML'
#cloud-config
write_files:
  - path: /var/tmp/oci-roadmap-marker.txt
    permissions: '0644'
    owner: root:root
    content: |
      roadmap-lab-created
runcmd:
  - [sh, -c, 'date -u > /var/tmp/oci-roadmap-created-at.txt']
  - [sh, -c, 'printf "ROADMAP_GUEST_ED25519_SHA256\n" > /dev/console && ssh-keygen -E sha256 -lf /etc/ssh/ssh_host_ed25519_key.pub > /dev/console']
YAML
# 본인 local: 원문 bytes를 Create 전에 고정한다.
python3 - <<'PY'
import pathlib,hashlib,os,json
p=pathlib.Path(os.environ['HOME'])/'oci-roadmap-lab/cloud-config.yaml'
b=p.read_bytes()
assert b.startswith(b'#cloud-config\n') and 0<len(b)<=32000
record={'path':str(p),'bytes':len(b),'sha256':hashlib.sha256(b).hexdigest()}
with open(str(p)+'.prelaunch.json','x') as f: json.dump(record,f,indent=2)
print(record)
PY
```
위 cat는 **학습자가 lab 파일을 작성하는 명령**이며 운영 파일 덮어쓰기용이 아니다. 파일이 이미 있으면 내용을 확인하고 전용 다른 경로를 사용한다.
업로드 **직전** 같은 local 파일을 다시 읽어 prelaunch와 exact size/hash를 비교한다. 다른 metadata/ssh_authorized_keys를 포함한 총합 32,000 bytes도 Console Review에서 확인한다; 파일 한 개 크기만으로 총합 통과를 주장하지 않는다.

```bash
python3 - <<'PY'
import pathlib,hashlib,json,os
p=pathlib.Path(os.environ['HOME'])/'oci-roadmap-lab/cloud-config.yaml'
r=json.load(open(str(p)+'.prelaunch.json')); b=p.read_bytes()
assert len(b)==r['bytes'] and hashlib.sha256(b).hexdigest()==r['sha256']
print('upload bytes match prelaunch evidence')
PY
```
실패하면 Create하지 않는다. 파일을 수정했다면 이전 증거를 보존하고 새 승인 path/event로 준비부터 다시 시작한다; postlaunch hash를 prelaunch 증거로 소급하지 않는다.

2. Compute Create instance → Advanced options → Management → Initialization script에 이 파일을 업로드한다. Console이 base64 변환하므로 YAML 원문을 올린다. User data와 기타 metadata 최대 크기 합계 32,000 bytes, reboot 금지, secrets 금지를 적용한다. [Initialization script/User data exact constraints](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/launchinginstance.htm).
3. 다른 launch fields를 완료하고 vm-a Create를 한 번 제출한다. same launch event에 user-data 파일 hash를 연결한다. RUNNING이 cloud-init 완료를 뜻하지 않으므로 guest 상태를 별도로 읽는다.

```bash
timeout --foreground -k 5s 60s ssh -i "$KEY" -o IdentitiesOnly=yes -o StrictHostKeyChecking=yes -o ConnectTimeout=10 -o ConnectionAttempts=1 -o ControlMaster=no -o ControlPath=none opc@"$PUBLIC_IP" 'sudo timeout -k 5s 40s cloud-init status --wait && sudo cat /var/tmp/oci-roadmap-marker.txt && sudo cat /var/tmp/oci-roadmap-created-at.txt' || exit 1
timeout --foreground -k 5s 60s ssh -i "$KEY" -o IdentitiesOnly=yes -o StrictHostKeyChecking=yes -o ConnectTimeout=10 -o ConnectionAttempts=1 -o ControlMaster=no -o ControlPath=none opc@"$PUBLIC_IP" 'sudo tail -n 80 /var/log/cloud-init-output.log' || exit 1
```
4. marker/시간과 로그를 확인한다. 종료코드에 warning이 있으면 `sudo cloud-init status --long`, `/var/log/cloud-init.log`와 output log를 읽어 실제 failure/warning을 구분한다. `cloud-init clean` 후 재실행은 일반 성공 대체물이 아니며 원격 접근을 손상시킬 수 있어 하지 않는다.

## 결과검증

<a id="baseline-17-ready"></a>

```bash
sha256sum "$HOME/oci-roadmap-lab/cloud-config.yaml"
ssh -i "$KEY" opc@"$PUBLIC_IP" 'test "$(sudo cat /var/tmp/oci-roadmap-marker.txt)" = roadmap-lab-created && sudo cloud-init status --long'
```
기대 패턴 marker exact match, cloud-init done/실제 warning 없음 또는 원인 해소 기록, 생성 후 UTC marker timestamp. guest YAML 실행 없이 Console upload만으로 통과하지 않는다.

**증거 기록:** 실제 명령·UTC 시간·exit code·가려진 OCID·옵션 readback·검증 조건을 남긴다. 현재 이 문서의 실행상태는 `NOT_EXECUTED`다. output/creation-event-id/result는 실행 전 비워 둔다. 오류/NOT_SELECTED를 PASS로 바꾸지 않는다.

### 이 시나리오의 functional acceptance

| Positive | Negative / 미완료 판정 | Recovery / 반환 |
|---|---|---|
| 최초 boot marker/status/log | 기존 local YAML path guard STOP 또는 cloud-init warning/error면 미완료 | 전용 파일/indent/size 수정은 launch 전; 이미 만든 VM은 별도 승인 전 재생성 없음 |

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

cloud-init warning/error는 status --long/log의 failing module·YAML indent/user-data size를 확인한다. 최초 boot를 놓친 VM에 metadata를 바꾸는 것만으로 runcmd가 자동 재실행된다고 가정하지 않는다.

## 리소스정리

local YAML/hash와 guest 두 파일; VM owner10 유지, clean/reboot 금지. 공유 기반 리소스는 이 consumer가 직접 삭제하지 않는다. 실제 소비자/expiry를 creator에게 반환하고 creator의 리소스정리 절차를 사용한다. VM을 Stop했다고 boot/IP/backup 비용이 끝난 것으로 처리하지 않는다. 로컬/guest 파일은 자신이 만든 정확한 경로만 제거하며 기록된 증거를 보존한다.

guest marker/created-at가 10/17 checkpoint 검증에 더 이상 필요하지 않을 때만 두 lab 파일을 제거한다. 로컬 cloud-config/hash는 비민감 증거 보존 기한 후 정리한다.


## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 아래 본문은 각 생성 필드·설정·검증·정리의 기술 근거이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [16] [Creating an Instance — Cloud-init / User Data](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/launchinginstance.htm)
  - 확인 범위: cloud-init.

### 추가 실행 절차 근거 (본문 확인)

- [launch](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/launchinginstance.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [connect](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/connect-to-linux-instance.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [cloudshell](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/cloudshellgettingstarted.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
