> **문서 검토 완료 — OCI 실행 미검증**: 이 Shared Security Model 시나리오는 한정된 문서 보정·절차 검토 범위에서 확인했습니다. 실제 OCI·guest·IAM 실행 검증이나 전체 교육과정 승인을 뜻하지 않습니다.

# Shared Security Model

> 문서 상태: **한정된 실행 절차 문서 검토 완료 / OCI 미실행**. 실제 권한·안전·예산·복구·지원 조건은 아래 실행 gate를 그대로 따릅니다. 모든 예상 출력은 패턴이며 실측값이 아닙니다. 게시 및 렌더링 검증은 이 문서 검토와 별개입니다.

> 실행 절차 작성본 / OCI 미실행. Oracle 공식 본문 근거와 교육용 선택을 구분한다. secrets·실제 고객값·diagram 없음.

## 학습목표

- owner23의 /32 TCP22 규칙과 모든 SL/NSG 허용 합집합을 대조하고 private bucket에 합성 security-demo object를 업로드한다.
- 새 다운로드 cmp와 guest sshd/package inventory를 확인하고 고객 책임 설정의 실제 증거를 남긴다. OS patch 설치 성공으로 대체하지 않는다.

위 항목은 사용자 실행 후 확인할 완료 조건이며 현재 관측 결과가 아니다. 개념은 아래 해당 생성 필드·configuration·test 단계에서 함께 적용한다.

## 대응 Roadmap 항목

- 항목명: **Shared Security Model**
- Roadmap 경로: **Introduction to OCI → Shared Security Model**
- Node ID: `sequence-1-4`
- 상위 항목: **Introduction to OCI** (`sequence-1-0`)
- 문서 범위: 이 항목의 생성·선택·기능검증·정리. 실습 미실행; 아래 기대 패턴은 실제 출력 아님.
- 번호는 navigation 식별자이며 실행 순서는 사전준비의 단계별 의존성에 따른다.

## 사전준비

### 기준 계약과 실제 단계 선행조건

[공통 baseline-contract.json](../../../baseline-contract.json)의 Oracle Linux 9 / opc / CIDR / 비용 상한을 따른다. 문서 번호는 실행 순서가 아니다. **준비 station과 생성 이후 ready를 분리**한다. 아래 단계 외의 기능완료를 소급하여 요구하지 않는다.

- [23 Public Subnets — 단계 링크](../../Networking/Original-Provisional/03_Virtual_Cloud_Network_VCN/04_Public_Subnets.md#public-network-preparation): **NSG 생성 station/단계 A; VM 완료 불필요**. 실제 출력/소비자/expiry를 전달하고 해당 owner가 이 lab 종료까지 보존한다.
- [04 Introduction to OCI — 단계 링크](00_Introduction_to_OCI.md#baseline-04-ready): **private bucket 완료**. 실제 출력/소비자/expiry를 전달하고 해당 owner가 이 lab 종료까지 보존한다.
- [10 Compute — 단계 링크](../02_Compute/00_Compute.md#baseline-10-ready): **첫 PUT 전 membership 조회에 RUNNING/primary VNIC 필요; OS inventory에는 SSH 필요**. 실제 출력/소비자/expiry를 전달하고 해당 owner가 이 lab 종료까지 보존한다.

생성 전 조직 승인(identity/권한/예산/expiry/서비스 지원/limit/capacity)을 확인한다. 실제 OCID/IP는 아직 생성하지 않았으면 비워 두고 해당 Create/readback 뒤 채운다. 동일 creation-event-id/OCID는 한 번의 제출만 뜻한다. `INSTANCE_A_ID`는 owner10 vm-a의 OCID, `INSTANCE_B_ID`는 owner22 private-vm의 OCID이며 owner07 vm-b와 다르다. 실습에 없는 자원을 추가하지 않는다. 실제 checkpoint는 사용자 실행 전 비워 둔다.

### 입력값과 실행 위치

Console에서 선택한 값만 사용한다. `<...>`는 예시 OCID나 비밀값이 아니라 **사용자가 대체할 입력 자리**다. 대체하지 않은 채 명령을 실행하지 않는다. Console → 우측 상단 Cloud Shell/Code Editor → Cloud Shell을 열면 OCI CLI가 사전 인증된다. Region 변경 뒤에는 Shell을 다시 열거나 명령의 `--region`을 바꾼다. SSH는 Cloud Shell이 아니라 본인 Linux/WSL/macOS 터미널에서 수행한다. 공인 IP/32도 그 터미널의 출구 주소여야 한다.

```bash
# 아래는 입력/출력의 대입 형식이다. 표의 단계 전에 생성 후 ID/IP export 행을 실행하지 않는다.
 # 생성 후 검증용 ID/IP는 해당 creator의 실제 응답에서 넣는다.
export REGION='<actual_REGION>'
export VCN_ID='<actual_VCN_ID>'
# KEY/A_PUBLIC_IP/PUBLIC_IP는 아래 본인 local SSH station에서만 지정한다.
export NSG_ADMIN_ID='<actual_NSG_ADMIN_ID>'
export BUCKET='<actual_BUCKET>'
export NAMESPACE='<actual_NAMESPACE>'
oci --version
```

이 변수 중 해당 절차에서 사용하는 값만 채운다. CLI 조회의 `data`를 증거 파일에 저장하고 실제 시간·작업 OCID·선택값을 기록한다. API key/private key/passphrase/인증 토큰은 기록하지 않는다. Cloud Shell 설정파일이나 세션 토큰을 출력하지 않는다. `oci --version`을 실행하는 주제의 기대 패턴은 버전 문자열뿐이다. namespace는 Object Storage 주제에서 별도 조회하며 다른 주제에는 요구하지 않는다. [Cloud Shell 공식 본문](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/cloudshellgettingstarted.htm).

### Workshop 입력 계약 — 단계와 host

| 입력/출력 | 값 또는 실제 획득 경로 | 사용할 시점 | 실행 위치 | 선택 이유 |
|---|---|---|---|---|
| Console/설정 필드 | owner23 nsg-admin ingress local source/32 TCP22; 합성 object intro/security-demo.txt; OS inventory만 | 생성 전 선택 → 생성 뒤 readback | 본문 Console/로컬 지정 | 이 topic의 최소 생성 범위 |
| `VCN_ID` | owner19 Create 응답 id | 선행 완료/생성 후 | Console/Cloud Shell | shared-vcn 참조 |
| `A_PUBLIC_IP` | owner10 vm-a VNIC public-ip | 선행 완료/생성 후 | 로컬 SSH | 관리 jump 주소 |
| `BUCKET` | owner04 Console 생성한 고유 이름 | 선행 완료/생성 후 | Console/Cloud Shell | object scope |
| `KEY` | owner15 Part A 전용 private key 절대경로; 내용 전송 금지 | 준비 완료 | 로컬 SSH만 | 해당 공개키와 대응 |
| `NAMESPACE` | oci os ns get --region "$REGION" --query data --raw-output | 생성 전/선행 완료 | Cloud Shell | Object Storage namespace |
| `NSG_ADMIN_ID` | owner23 nsg-admin Create id | 생성 후/선행 완료 | Console/Cloud Shell | A VNIC membership |
| `PUBLIC_IP` | A_PUBLIC_IP / owner10 VNIC public-ip | 생성 후/선행 완료 | 로컬 SSH | 대상 host 주소 |
| `REGION` | 승인된 기존 subscribed Region identifier | 생성 전 | Console/Cloud Shell | API endpoint/데이터 위치 |

생성 전에는 행의 승인값/선행 완료값만 지정하고, 생성 후 ID/IP는 해당 owner 응답으로 채운다. Cloud Shell과 로컬 SSH의 환경변수는 공유되지 않는다. 로컬에는 KEY와 접속 주소만 다시 지정하고 guest disk 명령은 로그인한 guest에서만 실행한다. `<...>`가 남으면 해당 명령은 실행하지 않는다.

## 권한

### 실제 task/API별 공식 권한 본문 대조

아래는 이 문서 생성·조회·변경·정리 API의 공식 표 본문을 읽은 인용이다. family grant를 별도 사용하는 경우 편의상 넓은 compartment-scope grant이며 **최소권한 정책이라고 부르지 않는다**. verb/dependency 행 인용은 단일 API의 완전한 최소권한 도출이 아니다. 조직 Identity Domain Console user/group 작업은 해당 Domain 관리자 역할도 필요하며 legacy IAM user API 권한만으로 대체하지 않는다.

- Network는 read-only: GetVnic=VNIC_READ, GetSubnet=SUBNET_READ, GetSecurityList=SECURITY_LIST_READ, ListNetworkSecurityGroupSecurityRules=NETWORK_SECURITY_GROUP_LIST_SECURITY_RULES를 [core 공식 operation 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm)와 대조하여 승인받는다. rule add/update/delete는 owner23에 반환하며 이 consumer에 위임하지 않는다.
- ListObjects=OBJECT_INSPECT는 첫 collision/final absence 조회에 필요하다. [Object operation 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm).
- `DeleteObject`: OBJECT_DELETE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm).
- `GetBucket`: BUCKET_READ For a customer-managed key encrypted bucket, the objectstorage-<location> subject must have KEY_DECRYPT. — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm).
- `GetNamespace`: API requires no permissions and returns the caller's namespace. Use the API to validate your credentials. OBJECTSTORAGE_NAMESPACE_READ permission is required if you include the optional compartmentId parameter. Use the compartmentId parameter to find the namespace for a third-party tenancy. — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm).
- `GetObject`: OBJECT_READ For a customer-managed key encrypted bucket, the objectstorage-<location> subject must have KEY_DECRYPT. — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm).
- `ListNetworkSecurityGroupSecurityRules`: NETWORK_SECURITY_GROUP _LIST_SECURITY_RULES — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `PutObject`: The permission required depends on whether the object already exists in the bucket: OBJECT_CREATE is required when an object with that name doesn't already exist in the bucket. OBJECT_OVERWRITE is required when an object with that name already exists in the bucket. For a customer-managed key encrypted bucket, the objectstorage-<location> subject must have KEY_ENCRYPT. — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm).

관리자는 actual lab-root/bucket으로 한정한 object create/read/list/delete 및 bucket read, 위 network read operation을 승인한다. 새 bucket/network CRUD 권한은 필요하지 않는다. 실제 조건부 IAM 허용/거부는 미검증이며 Cloud Shell 사용 승인과 local guest opc/sudo 조회 접근은 별개다.

object-only 실습자는 `manage buckets` 대신 아래 **read buckets**가 반드시 필요하다. 이 절차는 GetBucket을 두 번 호출하므로 object 권한만으로 진행하지 않는다. 관리자 승인 후 같은 identity/Region/bucket의 GetBucket과 collision ListObjects가 모두 성공한 경우만 Upload/PUT한다; 실패하면 BLOCKED_PERMISSION이다.

```text
Allow group <domain>/<lab-group> to read buckets in compartment lab-root where target.bucket.name='<actual_bucket_name>'
```

`GetBucket`=BUCKET_READ, `ListObjects`=OBJECT_INSPECT — [공식 operation 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm). 기존 manage objects도 같은 bucket 조건으로 좁혀 승인한다; 조건의 bucket 이름 비교는 case-insensitive이므로 이름 충돌도 관리자 확인한다. 정책은 교육용 scoped 예시이고 실제 identity-domain membership/조건부 정책 allow·deny 테스트는 미실행이다. Object Storage create/upload 공식문서의 Required IAM Policy 링크를 관리자와 확인한다. Cloud Shell을 쓰려면 별도 Cloud Shell 사용 허용이 필요하다.
OS inventory 후속에는 Compute의 guest sudo 권한 필요.

### namespace 조회와 조건부 권한

`oci os ns get --region "$REGION" --query data --raw-output`는 caller namespace를 읽으며 optional compartmentId를 넣지 않는다. 공식 Object Storage permission 표는 이 경로가 별도 namespace permission을 요구하지 않는다고 명시한다. third-party tenancy namespace를 optional compartmentId로 조회하는 별도 확장만 OBJECTSTORAGE_NAMESPACE_READ를 확인한다. 이 lab에 불필요한 tenancy-wide namespace grant를 추가하지 않는다.

## 비용및주의사항

- 신규는 합성 security-demo object **1개**이며 vm-a/bucket/NSG는 기존 creator의 같은 자원이다. PUT/GET/LIST/DELETE 요청 및 작은 파일 저장/전송 비용과 creator lease 기한만 승인한다. NSG/SL 변경·OS patch 설치/재부팅·logging resource는 만들지 않는다. read-only 조사에서 불일치가 발견되면 변경하지 말고 해당 owner에게 반환한다.
- 생성/사용 당일 공식 단가와 Billing & Cost Management → Cost Analysis의 실제 Compartment/서비스 필터를 확인하되 청구 반영 지연을 기록한다. 예산 경보는 과금 차단이 아니다. 고객 데이터/secret/운영 자원을 사용하지 않는다.
- 이 문서는 Oracle 기술 본문 기반 교육 절차다. OCI/guest/IAM allow·deny 실측은 없으며 기대 결과를 실행 결과로 기록하지 않는다. 승인되지 않은 선택 확장은 실행하지 않는다.


## 실습구성

### 이번 Workshop에서 만드는 것과 진행 순서

restricted nsg-admin과 private object 선택을 고객 책임에 연결하고 guest sshd/patch inventory를 조회한다. 업데이트 설치는 하지 않는다.

1. **진입/선택:** 23 단계 A NSG 선택. 사전준비 링크의 지정 단계/실제 출력이 없으면 제출하지 않는다.
2. **실제 작업:** 전체 VNIC/subnet/NSG/SL 조회·합집합 판정 → private bucket 확인 → security-demo PUT/GET → OS 조회. 아래 Console 필드/명령을 해당 host에서 순서대로 실행한다. 같은 자원은 이름이 아니라 OCID/event로 재사용한다.
3. **완료/복구/반환:** SL+모든 NSG 합집합 확인. 결과검증 실패는 문제해결로 이동하고 동일 Create를 무작정 재제출하지 않는다.

**전달/정리 범위:** security-demo object는 owner08 삭제; NSG owner23, VM owner10, bucket owner04. 아래 정리 단계는 이 소유권을 넘어서 실행하지 않는다.


- **이번 생성/재사용/출력:** 신규 security-demo object 1개 + 기존 nsg-admin 선택값 조회. OS patch inventory는 조회이며 patch 실행 아님.
- **생성 목표:** owner23이 이미 준비하여 vm-a에 연결한 restricted NSG와 모든 Security List를 읽기 전용으로 점검하고, 기존 private bucket에 synthetic object 1개를 생성한다. 이 소비자 절차에서는 NSG를 생성하거나 변경하지 않는다.
- **완료 증거:** 생성 전 옵션 선택 + 실제 제출 대상 OCID + API readback + 해당 기능검증 + dependency 순서 정리. 실제 OCI 실행 checkpoint는 비워 둔다.
- **retention:** 예약 소비자와 expiry가 있는 resource만 보존. idle 유료 resource를 다음 주제 때문에 무기한 유지하지 않는다.
- **계약 보정:** 기존 creation-first 계약의 문서만 존재하는 선행(24/25/26/29 등)과 bootstrap 순환은 실제 절차를 포함한 00/19/23 단계 A로 대체했다. 변경된 typed dependency는 authoring ledger에 기록한다.


## 실습절차

### 명령 실행 gate — 실패하면 다음 생성으로 진행하지 않는다

1. 아래 Bash는 **전용 Bash 세션**에서만 사용한다. Console/Cloud Shell 조회, 본인 로컬 SSH, 로그인한 Oracle Linux guest를 섞지 않는다. `exit 1`은 이 전용 세션을 닫는 안전 중단이다. `<...>`·빈 입력·타 owner OCID가 남으면 실행하지 않는다.
2. 성공을 요구하는 파일 작성/조회/설치/attach 작업은 실패 시 중단한다. 실패한 API 뒤 기존 JSON이나 기존 다운로드 파일을 성공 근거로 읽지 않는다. 기대 deny는 `if command; then ...; else RC=$?; ...; fi`로 별도 기록하고, 실제 timeout/IAM 오류/로컬 파일 오류를 구별한다.
3. 첫 변경 전에 정확한 OCID·owner·현재 설정과 승인 expiry를 저장한다. 공유 자원 변경 중 다른 소비자가 시작하거나 원복값을 확인할 수 없으면 변경을 중단한다. 삭제 성공은 API 수락이 아니라 같은 identity/Region의 정확한 대상 재조회로 판정한다. `NotAuthorizedOrNotFound` 단독은 삭제 증거가 아니다.


### 개별 완주 보정 — 생성·검증·소유권

nsg rules list는 --nsg-id를 사용한다. 실제 sl-public 전체와 VNIC 모든 NSG를 읽어 broad22가 없는지 확인한다. rpm 조회는 patch 설치가 아니며 object만 정리한다.

### 고객 책임을 실제 안전한 configuration으로 만든다

먼저 owner10의 같은 vm-a 생성/RUNNING과 primary VNIC handoff를 완료한다. PUT 전에 전용 evidence 경로를 만들고 모든 NSG/SL을 조회·판정한다.

```bash
set -euo pipefail
# Cloud Shell: 본 lab 전용 작업 경로와 원격 이름 충돌을 먼저 확인한다.
LAB_DIR="$HOME/oci-roadmap-lab"
test ! -L "$LAB_DIR" || exit 1
if test -e "$LAB_DIR"; then
  test -d "$LAB_DIR" && test -O "$LAB_DIR" || exit 1
  test "$(stat -c %a "$LAB_DIR")" = 700 || { printf 'STOP: owner-approved private parent required; do not chmod shared directory\n'; exit 1; }
else
  (umask 077; mkdir -m 700 -- "$LAB_DIR") || exit 1
fi
SECURITY_WORK=$(mktemp -d "$LAB_DIR/owner08-evidence.XXXXXXXX") || exit 1
cd "$SECURITY_WORK" || exit 1
printf '%s\n' "$SECURITY_WORK" # owner08 registry와 consumer09 SECURITY_EVIDENCE_DIR로 전달
```

Cloud Shell에서 owner10 vm-a의 실제 primary VNIC를 기준으로 membership을 먼저 읽는다. nsg-admin만 읽고 전체 합집합이라고 판정하지 않는다:

```bash
export VNIC_ID='<owner10_actual_primary_VNIC_OCID>'
oci network vnic get --vnic-id "$VNIC_ID" --region "$REGION" > security-vnic.json || exit 1
SUBNET_ID=$(python3 -c 'import json; print(json.load(open("security-vnic.json"))["data"]["subnet-id"])') || exit 1
oci network subnet get --subnet-id "$SUBNET_ID" --region "$REGION" > security-subnet.json || exit 1
python3 - <<'PY'
import json
v=json.load(open('security-vnic.json'))['data']; s=json.load(open('security-subnet.json'))['data']
assert v['id'] and v['is-primary'] is True and v['subnet-id']==s['id']
import os
assert os.environ['NSG_ADMIN_ID'] in v['nsg-ids'], 'STOP: expected nsg-admin is not attached'
assert s['vcn-id']==os.environ['VCN_ID'], 'STOP: wrong VCN'
with open('security-nsg-ids.txt','w') as f:
    f.write(''.join(x+'\n' for x in v['nsg-ids']))
with open('security-sl-ids.txt','w') as f:
    f.write(''.join(x+'\n' for x in s['security-list-ids']))
PY
while IFS= read -r id; do
  oci network nsg rules list --nsg-id "$id" --all --region "$REGION" > "security-$id.json" || exit 1
done < security-nsg-ids.txt
while IFS= read -r id; do
  oci network security-list get --security-list-id "$id" --region "$REGION" > "security-$id.json" || exit 1
done < security-sl-ids.txt
```
모든 저장 파일의 ingress를 함께 대조한다: protocol `all` 또는 TCP `6`의 destination port 생략/22 포함 범위, source CIDR 또는 source NSG의 실제 members를 검사한다. 허용 합집합에 승인 local /32 외 TCP22 경로가 하나라도 있으면 BLOCKED다. subnet/VNIC read와 NSG rule read의 공식 core 권한을 관리자에게 추가 승인받는다. 이 절차는 읽기 전용이며 broad rule 삭제/수정은 owner23에게 반환한다; 이 consumer가 승인만 받아 직접 수정하는 대체 branch는 없다.



1. 사전준비에서 완료한 Public Subnets 단계 A의 같은 nsg-admin OCID/creation-time 선택을 읽는다. source=본인 local 출구 /32, Stateful TCP22, HTTPS443 egress와 현재 모든 SL/NSG readback을 비교한다. 신규 NSG를 생성하지 않는다. 기존 값이 다르면 **BLOCKED_NETWORK**로 중단한다. 이 topic은 read-only이며 규칙을 수정하지 않는다. owner23에게 실제 VNIC/subnet/NSG/SL OCID, 조회 UTC, NSG의 exact rule ID(있는 경우), Security List는 전체 ingress 항목/프로토콜/source/port와 해당 list OCID를 전달한다. owner23가 자신의 승인·before-state·복구 절차로 수정하고 전체 membership/합집합 새 readback을 반환하기 전 PUT/보안 acceptance를 진행하지 않는다. SL 항목에 NSG rule ID가 있다고 가정하지 않는다. 안전 설정 검증을 위해 일부러 전체 인터넷 SSH를 열지 않는다. [Default security lists and allowed traffic](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/securitylists.htm).
2. Introduction to OCI에서 만든 private bucket에 합성 `security-demo.txt` object를 새로 생성한다. **CLI만 업로드한다.** Console → Storage → Object Storage & Archive Storage → Buckets → 실제 bucket → Objects는 아래 PUT **이후 관찰 화면**이며 Upload를 누르지 않는다. Cloud Shell에서 아래 합성 파일 생성/collision 확인/`--no-overwrite` PUT을 한 번 수행한다. object name `intro/security-demo.txt`, tier Standard; browser의 Choose files에 Cloud Shell 경로를 넣지 않는다. [Authenticated object upload](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/managingobjects_topic-To_upload_objects_to_a_bucket.htm).

```bash
# Exclusive owner fixture scope: no concurrent writer in this real private directory.
test ! -e "$HOME/oci-roadmap-lab/security-demo.txt" && test ! -L "$HOME/oci-roadmap-lab/security-demo.txt" || { printf "%s\n" "STOP: dedicated file exists"; exit 1; }
test ! -e "$HOME/oci-roadmap-lab/security-demo.download.txt" && test ! -L "$HOME/oci-roadmap-lab/security-demo.download.txt" || { printf "%s\n" "STOP: dedicated file exists"; exit 1; }
oci os bucket get --namespace-name "$NAMESPACE" --name "$BUCKET" --region "$REGION" > intro-bucket-before-write.json || exit 1
python3 - <<'PY'
import json
x=json.load(open('intro-bucket-before-write.json'))['data']
assert x['public-access-type']=='NoPublicAccess'
assert x['storage-tier']=='Standard'
assert x['versioning']=='Disabled'
assert x['kms-key-id'] is None
PY
oci os object list --namespace-name "$NAMESPACE" --bucket-name "$BUCKET" --prefix intro/ --all --region "$REGION" > intro-collision-check.json || exit 1
python3 - <<'PY'
import json
rows=json.load(open('intro-collision-check.json'))['data']
if isinstance(rows,dict): rows=rows['objects']
owned=['intro/security-demo.txt']
assert not set(owned).intersection(x['name'] for x in rows), 'STOP: owned object name exists; do not overwrite'
PY

set -C
printf '%s\n' 'synthetic non-sensitive security demo' > "$HOME/oci-roadmap-lab/security-demo.txt"
oci os object put --namespace-name "$NAMESPACE" --bucket-name "$BUCKET" --name intro/security-demo.txt --file "$HOME/oci-roadmap-lab/security-demo.txt" --region "$REGION" --no-overwrite || exit 1
oci os bucket get --namespace-name "$NAMESPACE" --name "$BUCKET" --region "$REGION" > security-bucket.json || exit 1
oci network nsg rules list --nsg-id "$NSG_ADMIN_ID" --all --region "$REGION" > security-nsg.json || exit 1
```
3. vm-a 생성 뒤 결과검증의 **trusted local SSH station**에서 OS/openssh/sshd inventory를 읽는다. OS patch를 실제 수행했다고 쓰지 않는다. package update는 허용 outbound/현재 repos/재부팅 필요/예산을 별도 승인한 후에만 한다.
4. 결과를 책임표로 작성한다: 고객은 object 공개설정/credentials/IAM/NSG/OS patching, Oracle은 물리 DC/hardware/provider isolation. Oracle 책임 항목을 바꾸는 fictitious resource는 만들지 않는다. Shared Security Model 본문의 실제 customer/provider 표와 대조한다.

5. PUT 전에 확인한 모든 subnet security lists + VNIC NSGs의 membership과 허용 규칙이 그대로인지 재확인한다. Cloud Shell의 같은 전용 Bash 세션에서 `RECHECK_WORK=$(mktemp -d "$SECURITY_WORK/recheck.XXXXXXXX") || exit 1`과 `cd "$RECHECK_WORK" || exit 1`을 실행한 다음, 위 VNIC/subnet/전체 NSG/SL 조회 블록을 다시 실행한다. 새 디렉터리를 사용하므로 앞의 `set -C`와 충돌하거나 최초 증거를 덮어쓰지 않는다. 최초 SECURITY_WORK와 새 조회의 membership·규칙을 대조하고 변경되었으면 같은 전체 합집합 판정을 다시 수행한다. 판정 뒤 `cd "$SECURITY_WORK" || exit 1`로 돌아와 GET/cmp와 정리를 계속한다. bucket private와 NSG /32만으로 전체 보안 compliance를 입증했다고 하지 않는다.

## 결과검증


### baseline-08-ready


```bash
oci os object get --namespace-name "$NAMESPACE" --bucket-name "$BUCKET" --name intro/security-demo.txt --file "$HOME/oci-roadmap-lab/security-demo.download.txt" --region "$REGION" || exit 1
cmp "$HOME/oci-roadmap-lab/security-demo.txt" "$HOME/oci-roadmap-lab/security-demo.download.txt" || exit 1
sha256sum "$HOME/oci-roadmap-lab/security-demo.txt" "$HOME/oci-roadmap-lab/security-demo.download.txt" > security-object-sha256.txt || exit 1
```

**개인 격리 경로:** [Compute 개인 첫 SSH](../02_Compute/00_Compute.md#개인-격리-실습의-첫-ssh)에서 같은 vm-a의 첫 host key를 등록한 뒤 아래 StrictHostKeyChecking=yes 조회를 실행한다. 개인 실습자가 모든 creator인 경우 OCID·원래 설정·종료시각을 본인 기록에 남기고 직접 정리한다. owner09와 evidence를 공유하는 경로에서는 기존 반환·보존 계약을 유지한다.

**공유 환경의 고급 신뢰 station:** [owner10 신뢰 bootstrap](../02_Compute/00_Compute.md#guest-trust-bootstrap)의 현재 vm-a OCID↔VNIC public IP↔serial Console ED25519 fingerprint↔known_hosts 등록을 완료한다. 불명확하거나 바뀌면 BLOCKED_TRUST이며 새 key를 수락하지 않는다. Cloud Shell로 개인키를 복사하지 않는다. Cloud Shell의 위 GET/cmp와 아래 SSH는 서로 다른 host다.

```bash
set -euo pipefail
# 본인 Linux/WSL/macOS 전용 Bash; 새 비공개 결과 디렉터리에서 실행한다.
LOCAL_SECURITY_WORK=$(umask 077; mktemp -d "$HOME/owner08-guest.XXXXXXXX") || exit 1
cd "$LOCAL_SECURITY_WORK" || exit 1
printf '%s\n' "$LOCAL_SECURITY_WORK" # Cloud Shell 증거와 별도로 보존
export KEY='<owner15_actual_local_private_key_absolute_path>'
export PUBLIC_IP='<owner10_actual_vm-a_primary_VNIC_public_IP>'
timeout --foreground 60s ssh -i "$KEY" -o BatchMode=yes -o IdentitiesOnly=yes -o HostKeyAlgorithms=ssh-ed25519 -o StrictHostKeyChecking=yes -o ConnectTimeout=10 -o ConnectionAttempts=1 -o ControlMaster=no -o ControlPath=none opc@"$PUBLIC_IP" 'test "$(id -un)" = opc && cat /etc/os-release && rpm -q openssh-server && sudo -n systemctl is-active sshd' > consumer08-guest.txt || exit 1
date -u +%FT%TZ > consumer08-session-ended.txt || exit 1
```
기대 패턴 cmp exit 0, bucket NoPublicAccess, nsg-admin TCP22 source/32, sl-public ingress empty, sshd active. 책임표의 physical security 행은 문서 기반 설명이며 lab 실행 출력이 아니다.

**증거 기록:** 실제 명령·UTC 시간·exit code·가려진 OCID·옵션 readback·검증 조건을 남긴다. 현재 이 문서의 실행상태는 `NOT_EXECUTED`다. output/creation-event-id/result는 실행 전 비워 둔다. 오류/NOT_SELECTED를 PASS로 바꾸지 않는다.

### 이 시나리오의 functional acceptance

| Positive | Negative / 미완료 판정 | Recovery / 반환 |
|---|---|---|
| private object cmp; SSH source /32; sshd active | 다른 NSG/SL broad SSH allow 발견 시 compliant로 통과 금지 | BLOCKED_NETWORK으로 owner23에 전체 근거 반환; owner23의 새 전체 readback 전까지 보안 acceptance 금지 |

모두 실행 시 확인할 기대 패턴이다. 생성 전 guard negative는 OCI 장애를 주입한 것과 다르며 다른 자원을 추가하지 않는다. 오류 메시지와 실제 exit code가 없는 항목은 NOT_EXECUTED 또는 BLOCKED로 남긴다.

## 문제해결

### 공통 오류의 진단 순서

1. `NotAuthorizedOrNotFound` 또는 목록에 리소스가 없음: 먼저 Region/Compartment/OCID와 identity domain/group을 확인한다. 존재하지 않는 OCID와 권한 부족 모두 같은 오류로 보일 수 있다. 관리자에게 request-id·작업 시각·서비스·가려진 대상 OCID를 제공한다. 비밀키/토큰은 보내지 않는다.
2. `LimitExceeded`/quota/capacity 부족: Billing & Cost Management → Limits, Quotas and Usage와 생성 화면의 지원 shape/AD를 확인한다. 운영 quota 증가나 다른 Region 구독을 임의 요청하지 않는다. 승인 범위 안의 더 작은 compatible shape 또는 다른 AD를 선택하거나 `BLOCKED_CAPACITY`를 기록한다.
3. 장시간 provisioning/예상 밖 state: Work Requests/오류 상세를 읽고 같은 요청을 반복 제출하지 않는다. 만들어진 부분 리소스(boot/IP/attachment 포함)를 먼저 조사한다.
4. 메뉴 이름이 다름: 공식 task body의 동등한 탭(예: Storage 또는 Attached Block Volumes)을 찾는다. 필드가 문서와 다르면 값·스크린샷에서 secrets를 가린 뒤 관리자 또는 Oracle Support에 확인하고 제출을 중단한다. 이 문서는 현재 tenancy의 live Console 화면을 검사한 결과가 아니다.

### 이 주제의 실패 지점

nsg-admin이 제한적인데 SSH가 넓게 열려 있으면 모든 security lists/NSGs를 조사한다. broad allow를 좁은 NSG가 deny하지 않는다. object가 public이면 PUT 전 BLOCKED_BUCKET으로 owner04에 반환한다. owner04가 private 복구/새 readback을 반환하기 전 소비자가 visibility를 변경하지 않는다.

## 리소스정리

실패·중단도 이 순서로 정리한다. PUT 전 중단했다면 remote object를 삭제하지 않는다. PUT 응답이 불명확하면 같은 identity/Region/bucket의 exact name을 조회하고 owner04와 실제 제출·내용을 대조하여 본인 생성임을 확인하기 전 삭제/재PUT하지 않는다. 원래 존재하던 object는 삭제하지 않는다. PUT 성공 뒤 GET/SSH가 실패해도 이번 object만 정리하고 실패 항목을 BLOCKED로 남긴다.

새 Cloud Shell에서는 기록한 REGION/NAMESPACE/BUCKET/VCN_ID/NSG_ADMIN_ID/VNIC_ID와 SECURITY_WORK의 실제 절대경로를 다시 대입한다. SECURITY_WORK가 본인 소유의 비공개 실제 디렉터리인지 확인하고 `cd "$SECURITY_WORK" || exit 1` 한다. 증거 디렉터리를 새로 만들어 이전 실행의 소유권을 추정하지 않는다. 재조회 출력 filename이 이미 있으면 덮어쓰지 말고 새 filename을 선택한다. 로컬 SSH 기록은 LOCAL_SECURITY_WORK이며 Cloud Shell에 자동으로 존재하지 않는다.

object `intro/security-demo.txt`만 Console → 실제 bucket → Objects → exact name → Delete 확인 후 삭제한다. Cloud Shell SECURITY_WORK의 새 filename으로 `oci os object list --namespace-name "$NAMESPACE" --bucket-name "$BUCKET" --prefix intro/ --all --region "$REGION" > security-final-objects.json || exit 1`을 실행하고 `data.objects`(또는 data list)의 name 집합에 `intro/security-demo.txt` 없음만 확인한다. 다른 intro/ object가 남아도 이 consumer는 지우지 않는다. 이 readback 이후에만 아래 local file cleanup을 실행한다. 

Cloud Shell의 owner08 원본/다운로드 cmp 및 hash 증거를 보존하고 remote object 삭제 readback 후 아래 두 파일만 제거한다. owner04 파일과 shared directory는 삭제하지 않는다. fresh terminal은 같은 HOME/owner08 registry의 정확한 경로를 다시 확인한다.
```bash
for f in "$HOME/oci-roadmap-lab/security-demo.txt" "$HOME/oci-roadmap-lab/security-demo.download.txt"; do
  if test ! -e "$f" && test ! -L "$f"; then continue; fi
  test -f "$f" && test ! -L "$f" && test -O "$f" || exit 1
  rm -- "$f" || exit 1
  test ! -e "$f" && test ! -L "$f" || exit 1
done
```


owner04/10/23 registry에 consumer08, actual bucket/VM/VNIC/subnet/NSG/SL OCID, object/cmp/security/guest 증거 UTC/path, session-end UTC, outstanding changes=none, lease RELEASED, 각 owner acknowledgment를 남긴다. owner09가 보안 evidence를 사용하면 SECURITY_WORK 및 LOCAL_SECURITY_WORK의 파일들은 그 반환과 승인 보존이 끝날 때까지 유지한다. 그 뒤 본인이 만든 두 evidence 디렉터리의 파일만 확인하여 제거하고 빈 디렉터리는 rmdir로 제거한다. 실패/보존기한 초과는 해당 owner에게 반환하고 lease RELEASED로 기록하지 않는다. nsg-admin은 shared vm-a 연결이 끝날 때까지 생성 owner가 보존한다. OS package를 변경하지 않았으므로 package rollback은 없다. shared bucket/NSG/VM을 이 topic 끝에 독자 삭제하지 않는다.


## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 아래 본문은 각 생성 필드·설정·검증·정리의 기술 근거이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [40] [Shared Security Model — Shared Security Model](https://docs.oracle.com/en-us/iaas/Content/cloud-adoption-framework/security.htm)
  - 확인 범위: Security in the cloud is.

### 추가 실행 절차 근거 (본문 확인)

- [security](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/securitylists.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [upload](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/managingobjects_topic-To_upload_objects_to_a_bucket.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [bucket](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/managingbuckets_topic-To_create_a_bucket.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [cloudshell](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/cloudshellgettingstarted.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.

- [추가 세부 task: objectpolicy](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm) — 실제 본문을 확인한 필드·API/권한·삭제 단계 근거.

- [NSG directions, union, memberships](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/networksecuritygroups.htm) — 규칙 방향·VNIC association·합집합 및 삭제 선행조건 본문.
