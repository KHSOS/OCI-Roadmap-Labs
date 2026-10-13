# Introduction to OCI

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../README.md)를 확인하세요.

> 실행 절차 작성본 / OCI 미실행. Oracle 공식 본문 근거와 교육용 선택을 구분한다. secrets·실제 고객값·diagram 없음.

## 학습목표

- Standard·NoPublicAccess·Oracle-managed encryption의 고유 bucket 하나를 생성하고 설정을 readback한 뒤 intro/bootstrap.txt를 업로드한다.
- 새 다운로드 파일과 원본의 cmp/hash를 비교하고 소유 object만 삭제하며 공유 bucket의 보존 기한을 creator ledger에 기록한다.

위 항목은 사용자 실행 후 확인할 완료 조건이며 현재 관측 결과가 아니다. 개념은 아래 해당 생성 필드·configuration·test 단계에서 함께 적용한다.

## 대응 Roadmap 항목

- 항목명: **Introduction to OCI**
- Roadmap 경로: **Introduction to OCI**
- Node ID: `sequence-1-0`
- 상위 항목: 없음 — 최상위 학습 항목
- 문서 범위: 이 항목의 생성·선택·기능검증·정리. 실습 미실행; 아래 기대 패턴은 실제 출력 아님.
- 번호는 navigation 식별자이며 실행 순서는 사전준비의 단계별 의존성에 따른다.

## 사전준비

### 기준 계약과 실제 단계 선행조건

[공통 baseline-contract.json](../../baseline-contract.json)의 Oracle Linux 9 / opc / CIDR / 비용 상한을 따른다. 문서 번호는 실행 순서가 아니다. **준비 station과 생성 이후 ready를 분리**한다. 아래 단계 외의 기능완료를 소급하여 요구하지 않는다.

- [00 Introduction — 단계 링크](../00_Introduction/00_Introduction.md#baseline-00-ready): **ACTIVE lab-root/승인 REGION**. 실제 출력/소비자/expiry를 전달하고 해당 owner가 이 lab 종료까지 보존한다.

생성 전 조직 승인(identity/권한/예산/expiry/서비스 지원/limit/capacity)을 확인한다. 실제 OCID/IP는 아직 생성하지 않았으면 비워 두고 해당 Create/readback 뒤 채운다. 동일 creation-event-id/OCID는 한 번의 제출만 뜻한다. `INSTANCE_A_ID`는 owner10 vm-a의 OCID, `INSTANCE_B_ID`는 owner22 private-vm의 OCID이며 owner07 vm-b와 다르다. 실습에 없는 자원을 추가하지 않는다. 실제 checkpoint는 사용자 실행 전 비워 둔다.

### 입력값과 실행 위치

Console에서 선택한 값만 사용한다. `<...>`는 예시 OCID나 비밀값이 아니라 **사용자가 대체할 입력 자리**다. 대체하지 않은 채 명령을 실행하지 않는다. Console → 우측 상단 Cloud Shell/Code Editor → Cloud Shell을 열면 OCI CLI가 사전 인증된다. Region 변경 뒤에는 Shell을 다시 열거나 명령의 `--region`을 바꾼다. SSH는 Cloud Shell이 아니라 본인 Linux/WSL/macOS 터미널에서 수행한다. 공인 IP/32도 그 터미널의 출구 주소여야 한다.

```bash
# 아래는 입력/출력의 대입 형식이다. 표의 단계 전에 생성 후 ID/IP export 행을 실행하지 않는다.
 # 생성 후 검증용 ID/IP는 해당 creator의 실제 응답에서 넣는다.
export REGION='<actual_REGION>'
export LAB_COMPARTMENT_ID='<actual_LAB_COMPARTMENT_ID>'
export BUCKET='<actual_BUCKET>'
export NAMESPACE='<actual_NAMESPACE>'
oci --version
```

이 변수 중 해당 절차에서 사용하는 값만 채운다. CLI 조회의 `data`를 증거 파일에 저장하고 실제 시간·작업 OCID·선택값을 기록한다. API key/private key/passphrase/인증 토큰은 기록하지 않는다. Cloud Shell 설정파일이나 세션 토큰을 출력하지 않는다. `oci --version`을 실행하는 주제의 기대 패턴은 버전 문자열뿐이다. namespace는 Object Storage 주제에서 별도 조회하며 다른 주제에는 요구하지 않는다. [Cloud Shell 공식 본문](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/cloudshellgettingstarted.htm).

### Workshop 입력 계약 — 단계와 host

| 입력/출력 | 값 또는 실제 획득 경로 | 사용할 시점 | 실행 위치 | 선택 이유 |
|---|---|---|---|---|
| Console/설정 필드 | Name=lab-objects-승인고유suffix; Standard; Private; versioning/auto-tiering/events/logging Off; Oracle-managed; NAMESPACE scope | 생성 전 선택 → 생성 뒤 readback | 본문 Console/로컬 지정 | 이 topic의 최소 생성 범위 |
| `LAB_COMPARTMENT_ID` | owner00 Create 응답 OCID; topic00에서는 생성 뒤 | 선행 완료/생성 후 | Console/Cloud Shell | 정책/자원 scope |
| `BUCKET` | owner04 Console 생성한 고유 이름 | 선행 완료/생성 후 | Console/Cloud Shell | object scope |
| `NAMESPACE` | oci os ns get --region "$REGION" --query data --raw-output | 생성 전/선행 완료 | Cloud Shell | Object Storage namespace |
| `REGION` | 승인된 기존 subscribed Region identifier | 생성 전 | Console/Cloud Shell | API endpoint/데이터 위치 |

생성 전에는 행의 승인값/선행 완료값만 지정하고, 생성 후 ID/IP는 해당 owner 응답으로 채운다. Cloud Shell과 로컬 SSH의 환경변수는 공유되지 않는다. 로컬에는 KEY와 접속 주소만 다시 지정하고 guest disk 명령은 로그인한 guest에서만 실행한다. `<...>`가 남으면 해당 명령은 실행하지 않는다.

## 권한

### 실제 task/API별 공식 권한 본문 대조

아래는 이 문서 생성·조회·변경·정리 API의 공식 표 본문을 읽은 인용이다. 위 family 예제는 편의상 넓은 compartment-scope grant이며 **최소권한 정책이라고 부르지 않는다**. verb/dependency 행 인용은 단일 API의 완전한 최소권한 도출이 아니다. 조직 Identity Domain Console user/group 작업은 해당 Domain 관리자 역할도 필요하며 legacy IAM user API 권한만으로 대체하지 않는다.

- `CreateBucket`: BUCKET_CREATE If the KMS Key ID is provided to the operation, the following additional permissions are required: KEY_ASSOCIATE The objectstorage-<location> subject must also have: KEY_ENCRYPT, KEY_DECRYPT, KEY_READ. — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm).
- `DeleteBucket`: BUCKET_DELETE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm).
- `DeleteObject`: OBJECT_DELETE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm).
- `GetBucket`: BUCKET_READ For a customer-managed key encrypted bucket, the objectstorage-<location> subject must have KEY_DECRYPT. — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm).
- `GetNamespace`: API requires no permissions and returns the caller's namespace. Use the API to validate your credentials. OBJECTSTORAGE_NAMESPACE_READ permission is required if you include the optional compartmentId parameter. Use the compartmentId parameter to find the namespace for a third-party tenancy. — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm).
- `GetObject`: OBJECT_READ For a customer-managed key encrypted bucket, the objectstorage-<location> subject must have KEY_DECRYPT. — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm).
- `ListObjects`: OBJECT_INSPECT — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm).
- `PutObject`: The permission required depends on whether the object already exists in the bucket: OBJECT_CREATE is required when an object with that name doesn't already exist in the bucket. OBJECT_OVERWRITE is required when an object with that name already exists in the bucket. For a customer-managed key encrypted bucket, the objectstorage-<location> subject must have KEY_ENCRYPT. — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm).

관리자가 `lab-root`에 bucket 생성·조회·삭제와 object 생성·조회·삭제를 위임한다. 초기 bucket 작성자와 object-only 학습자를 분리할 수 있다.

```text
Allow group <domain>/<lab-group> to manage buckets in compartment lab-root
Allow group <domain>/<lab-group> to manage objects in compartment lab-root
```

owner04의 Create/visibility 복구/최종 Delete는 `manage buckets` 위임을 가진 bucket creator만 수행한다. 이미 생성한 bucket의 object-only 소비자는 `manage buckets`를 생략할 수 있지만 본문처럼 GetBucket 설정 readback을 수행하려면 **read buckets**가 필요하다. `manage objects`는 BUCKET_READ를 대신하지 않는다.

```text
Allow group <domain>/<object-consumer-group> to read buckets in compartment lab-root where target.bucket.name='<actual_bucket_name>'
Allow group <domain>/<object-consumer-group> to manage objects in compartment lab-root where target.bucket.name='<actual_bucket_name>'
```

`GetBucket`=BUCKET_READ, `ListObjects`=OBJECT_INSPECT를 [공식 operation 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm)와 대조한다. 같은 identity/Region/actual bucket의 GetBucket과 collision ListObjects가 성공하기 전 PUT하지 않으며 실패하면 BLOCKED_PERMISSION이다. bucket 조건 비교는 case-insensitive이므로 관리자와 실제 고유 이름을 확인한다. object-only 사용자는 bucket 생성/설정변경/삭제를 대신하지 않고 owner04에게 반환한다. 정책은 교육용 scoped 예시이며 최소권한 도출 또는 실제 allow·deny 검증을 뜻하지 않는다. Cloud Shell 사용 허용도 별도로 필요하다.

### namespace 조회와 조건부 권한

`oci os ns get --region "$REGION" --query data --raw-output`는 caller namespace를 읽으며 optional compartmentId를 넣지 않는다. 공식 Object Storage permission 표는 이 경로가 별도 namespace permission을 요구하지 않는다고 명시한다. third-party tenancy namespace를 optional compartmentId로 조회하는 별도 확장만 OBJECTSTORAGE_NAMESPACE_READ를 확인한다. 이 lab에 불필요한 tenancy-wide namespace grant를 추가하지 않는다.

## 비용및주의사항

- private Standard bucket **1개**와 합성 bootstrap object **1개**만 생성한다. 실제 byte size/요청/전송 요금을 승인 Region에서 확인하고 Archive·auto-tiering·versioning·events·logging·7일 multipart lifecycle·customer-managed KMS는 추가하지 않는다. 후속 소비자 때문에 bucket을 보존할 때는 creator lease에 실제 소비자/expiry를 기록한다. current object 삭제만으로 version/soft-delete/미완료 multipart 비용이 끝났다고 하지 않는다.
- 생성/사용 당일 공식 단가와 Billing & Cost Management → Cost Analysis의 실제 Compartment/서비스 필터를 확인하되 청구 반영 지연을 기록한다. 예산 경보는 과금 차단이 아니다. 고객 데이터/secret/운영 자원을 사용하지 않는다.
- 이 문서는 Oracle 기술 본문 기반 교육 절차다. OCI/guest/IAM allow·deny 실측은 없으며 기대 결과를 실행 결과로 기록하지 않는다. 승인되지 않은 선택 확장은 실행하지 않는다.


## 실습구성

### 이번 Workshop에서 만드는 것과 진행 순서

lab-root에 private Standard bucket 한 개와 bootstrap 합성 object를 생성한다. bucket은 VCN 밖의 regional 서비스다.

1. **진입/선택:** 고유 bucket 이름 승인. 사전준비 링크의 지정 단계/실제 출력이 없으면 제출하지 않는다.
2. **실제 작업:** Standard/Disabled/Oracle-managed 생성 → NoPublicAccess 확인 → namespace 획득. 아래 Console 필드/명령을 해당 host에서 순서대로 실행한다. 같은 자원은 이름이 아니라 OCID/event로 재사용한다.
3. **완료/복구/반환:** PUT/GET/cmp. 결과검증 실패는 문제해결로 이동하고 동일 Create를 무작정 재제출하지 않는다.

**전달/정리 범위:** BUCKET/NAMESPACE/Region/bucket OCID; object만 먼저 정리, bucket은 owner04가 최종 삭제. 아래 정리 단계는 이 소유권을 넘어서 실행하지 않는다.


- **이번 생성/재사용/출력:** 신규: private Standard bucket 1개와 bootstrap synthetic object. 재사용: lab-root. 출력: BUCKET/NAMESPACE/bucket OCID/private readback/hash. 01/08와 48 이후 주제를 위해 보존.
- **생성 목표:** Object Storage의 단일 실제 생성 event에서 Introduction to OCI 옵션을 직접 선택하고 저장·기능 검증한다. 독립 중복 생성은 하지 않는다.
- **완료 증거:** 생성 전 옵션 선택 + 실제 제출 대상 OCID + API readback + 해당 기능검증 + dependency 순서 정리. 실제 OCI 실행 checkpoint는 비워 둔다.
- **retention:** 예약 소비자와 expiry가 있는 resource만 보존. idle 유료 resource를 다음 주제 때문에 무기한 유지하지 않는다.


## 실습절차

### 작업 순서 체크포인트
private bucket Create/readback→새 bootstrap PUT→새 GET/cmp→object cleanup→local fixture ownership cleanup→후속 consumer가 모두 반환하면 bucket cleanup 순서다. 단계 실패는 다음 생성으로 넘기지 않고 같은 target의 복구/반환으로 처리한다.

### 명령 실행 gate — 실패하면 다음 생성으로 진행하지 않는다

1. 아래 Bash는 **전용 Bash 세션**에서만 사용한다. Console/Cloud Shell 조회, 본인 로컬 SSH, 로그인한 Oracle Linux guest를 섞지 않는다. `exit 1`은 이 전용 세션을 닫는 안전 중단이다. `<...>`·빈 입력·타 owner OCID가 남으면 실행하지 않는다.
2. 성공을 요구하는 파일 작성/조회/설치/attach 작업은 실패 시 중단한다. 실패한 API 뒤 기존 JSON이나 기존 다운로드 파일을 성공 근거로 읽지 않는다. 기대 deny는 `if command; then ...; else RC=$?; ...; fi`로 별도 기록하고, 실제 timeout/IAM 오류/로컬 파일 오류를 구별한다.
3. 첫 변경 전에 정확한 OCID·owner·현재 설정과 승인 expiry를 저장한다. 공유 자원 변경 중 다른 소비자가 시작하거나 원복값을 확인할 수 없으면 변경을 중단한다. 삭제 성공은 API 수락이 아니라 같은 identity/Region의 정확한 대상 재조회로 판정한다. `NotAuthorizedOrNotFound` 단독은 삭제 증거가 아니다.


### 개별 완주 보정 — 생성·검증·소유권

bucket 이름은 namespace의 실제 미사용 고유 이름으로 승인받는다. object 업로드 전에 NoPublicAccess와 Standard/Disabled를 읽는다. 다운로드는 새 전용 filename을 사용한다. 후속 소비자 종료 후 이 bucket만 owner04가 삭제한다.

### 작은 private bucket을 생성한다

1. Storage → Object Storage & Archive Storage → Buckets. Region은 승인 REGION, Compartment `lab-root`로 선택한다. Create bucket을 누른다. Name은 `lab-objects-<비민감_고유_suffix>`(namespace 내 충돌 방지), Default storage tier **Standard**. Archive는 즉시 읽기 불가와 최소 보관 부담 때문에 쓰지 않는다. [Creating a Bucket — Standard and name fields](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/managingbuckets_topic-To_create_a_bucket.htm).
2. Auto-tiering Off, Object versioning Off, Emit object events Off, Resource logging Off. Encryption **Oracle-managed keys**, Bucket scope **NAMESPACE**(bucket 이름은 **이 tenancy와 이 Region 안에서** 고유; path-style만 사용). **REGION**은 같은 Region의 모든 tenancy 사이 고유 이름이 필요하며 virtual-hosted URL도 지원한다. 이 lab은 REGION scope/virtual-hosted access가 필요하지 않아 NAMESPACE를 선택한다. scope를 REGION으로 변경하면 NAMESPACE로 되돌릴 수 없으므로 변경하지 않는다. **Uncommitted multipart uploads cleanup = Off**를 명시적으로 선택한다. On이면 7일 뒤 uncommitted upload를 삭제하는 lifecycle rule이 생기므로 이 lab의 자동변경 없음 범위를 어긴다. Retention/lifecycle/replication/customer-managed KMS를 추가하지 않는다.  Create bucket을 제출한다.
3. Create 한 번 이후 생성 직후 bucket → **Policies → Lifecycle policy rules**에서 모든 행/페이지가 없는지 확인한다. 예상 밖 rule이면 PUT 전 BLOCKED_LIFECYCLE로 owner04에게 반환하며 다른 owner rule을 삭제하지 않는다. [Lifecycle list Console task](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/usinglifecyclepolicies.htm). soft-delete는 cached Create task가 입력 필드를 확정하지 않는다: 새 policy를 임의 설정하지 않고 생성 뒤 GetBucket의 `soft-delete-policy.current-policy.soft-delete-mode`가 제공되면 Disabled인지 읽는다. enabled/변경예정 next-policy/unknown live 필드는 PUT 전 BLOCKED_SOFT_DELETE로 관리자에게 정확한 현재 공식 task/승인 cleanup을 확보한다; cached example를 live default로 가정하지 않는다. 상세의 Visibility/Manage visibility에서 **Private / NoPublicAccess**를 확인한다. 생성 화면에 visibility 필드가 없으면 기본을 믿고 끝내지 말고 생성 직후 읽어 private인지 검증한다. 잘못 public이면 즉시 private로 고친 뒤 object 업로드 전 상태를 읽는다.
4. Cloud Shell에서 caller namespace를 **checked substitution**으로 얻는다. 실패한 response를 빈 namespace로 넘기지 않는다.

```bash
# Cloud Shell 전용; Console에서 승인한 REGION을 먼저 지정한다.
NAMESPACE=$(oci os ns get --region "$REGION" --query data --raw-output) || { printf '%s\n' 'BLOCKED_NAMESPACE: retain actual error/request-id'; exit 1; }
test -n "$NAMESPACE" || exit 1
case "$NAMESPACE" in *'<'*|*'>'*|null|None) printf '%s\n' 'BLOCKED_NAMESPACE_INPUT'; exit 1;; esac
export NAMESPACE
```

namespace를 확보한 뒤 `BUCKET`은 Console에서 실제 생성한 이름으로 지정한다. bucket 이름, namespace, Region, Compartment와 OCID를 기록한다. 이 생성은 Introduction to OCI와 후속 Object Storage 주제가 공유하는 한 번의 creation event다. 서론 때문에 두 번째 bucket을 만들지 않는다.
5. Cloud Shell에서 아래 1줄 텍스트를 만들고 아래 CLI로만 업로드한다. Console Objects → Upload는 실행하지 않고 PUT 후 이름/size 관찰만 한다. Object name `intro/bootstrap.txt`, tier Standard, overwrite 대상 없는 새 이름을 사용한다. Download 후 같은 내용/체크섬을 확인한다. [Object upload](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/managingobjects_topic-To_upload_objects_to_a_bucket.htm).

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
# Exclusive owner fixture scope: no concurrent writer in this real private directory.
test ! -e "$HOME/oci-roadmap-lab/bootstrap.txt" && test ! -L "$HOME/oci-roadmap-lab/bootstrap.txt" || { printf "%s\n" "STOP: dedicated file exists"; exit 1; }
test ! -e "$HOME/oci-roadmap-lab/bootstrap.download.txt" && test ! -L "$HOME/oci-roadmap-lab/bootstrap.download.txt" || { printf "%s\n" "STOP: dedicated file exists"; exit 1; }
oci os bucket get --namespace-name "$NAMESPACE" --name "$BUCKET" --region "$REGION" > intro-bucket-before-write.json || exit 1
python3 - <<'PY'
import json
x=json.load(open('intro-bucket-before-write.json'))['data']
assert x['public-access-type']=='NoPublicAccess'
assert x['storage-tier']=='Standard'
assert x['versioning']=='Disabled'
assert x['kms-key-id'] is None
if 'bucket-scope' in x: assert x['bucket-scope']=='NAMESPACE'
assert x.get('object-lifecycle-policy-etag') is None, 'BLOCKED_LIFECYCLE'
if x.get('soft-delete-policy'):
    p=x['soft-delete-policy']
    assert p['current-policy']['soft-delete-mode']=='Disabled', 'BLOCKED_SOFT_DELETE'
    assert p.get('next-policy') is None, 'BLOCKED_PENDING_SOFT_DELETE' 
PY
oci os object list --namespace-name "$NAMESPACE" --bucket-name "$BUCKET" --prefix intro/ --all --region "$REGION" > intro-collision-check.json || exit 1
python3 - <<'PY'
import json
rows=json.load(open('intro-collision-check.json'))['data']
if isinstance(rows,dict): rows=rows['objects']
owned=['intro/bootstrap.txt']
assert not set(owned).intersection(x['name'] for x in rows), 'STOP: owned object name exists; do not overwrite'
PY

set -C
printf '%s\n' 'oci roadmap synthetic bootstrap' > "$HOME/oci-roadmap-lab/bootstrap.txt"
oci os object put --namespace-name "$NAMESPACE" --bucket-name "$BUCKET" --name intro/bootstrap.txt --file "$HOME/oci-roadmap-lab/bootstrap.txt" --region "$REGION" --no-overwrite || exit 1
oci os object get --namespace-name "$NAMESPACE" --bucket-name "$BUCKET" --name intro/bootstrap.txt --file "$HOME/oci-roadmap-lab/bootstrap.download.txt" --region "$REGION" || exit 1
cmp "$HOME/oci-roadmap-lab/bootstrap.txt" "$HOME/oci-roadmap-lab/bootstrap.download.txt" || exit 1
```

## 결과검증

<a id="baseline-04-ready"></a>

```bash
oci os bucket get --namespace-name "$NAMESPACE" --name "$BUCKET" --region "$REGION" > bucket-readback.json || exit 1
oci os object list --namespace-name "$NAMESPACE" --bucket-name "$BUCKET" --prefix intro/ --all --region "$REGION" > object-readback.json || exit 1
sha256sum "$HOME/oci-roadmap-lab/bootstrap.txt" "$HOME/oci-roadmap-lab/bootstrap.download.txt"
```
기대 패턴: bucket `public-access-type=NoPublicAccess`, `storage-tier=Standard`, `versioning=Disabled`, `object-events-enabled=false`, `compartment-id=LAB_COMPARTMENT_ID`, `kms-key-id=null`(Oracle-managed). 두 파일 hash 일치와 cmp 종료코드 0, object 목록의 `intro/bootstrap.txt` 및 실제 byte size를 확인한다. 대략 count/size 집계는 갱신 지연이 있으므로 개별 object 조회를 우선한다.

**증거 기록:** 실제 명령·UTC 시간·exit code·가려진 OCID·옵션 readback·검증 조건을 남긴다. 현재 이 문서의 실행상태는 `NOT_EXECUTED`다. output/creation-event-id/result는 실행 전 비워 둔다. 오류/NOT_SELECTED를 PASS로 바꾸지 않는다.

### 이 시나리오의 functional acceptance

| Positive | Negative / 미완료 판정 | Recovery / 반환 |
|---|---|---|
| private Standard bucket; PUT/GET/cmp | 같은 object name 충돌 시 Python guard 또는 --no-overwrite 거부 | 기존 object 변경 없이 owner 확인; 승인된 새 prefix로 guard부터 재시작 |

모두 실행 시 확인할 기대 패턴이다. 생성 전 guard negative는 OCI 장애를 주입한 것과 다르며 다른 자원을 추가하지 않는다. 오류 메시지와 실제 exit code가 없는 항목은 NOT_EXECUTED 또는 BLOCKED로 남긴다.

## 문제해결

### 공통 오류의 진단 순서

1. `NotAuthorizedOrNotFound` 또는 목록에 리소스가 없음: 먼저 Region/Compartment/OCID와 identity domain/group을 확인한다. 존재하지 않는 OCID와 권한 부족 모두 같은 오류로 보일 수 있다. 관리자에게 request-id·작업 시각·서비스·가려진 대상 OCID를 제공한다. 비밀키/토큰은 보내지 않는다.
2. `LimitExceeded`/quota/capacity 부족: Billing & Cost Management → Limits, Quotas and Usage와 생성 화면의 지원 shape/AD를 확인한다. 운영 quota 증가나 다른 Region 구독을 임의 요청하지 않는다. 승인 범위 안의 더 작은 compatible shape 또는 다른 AD를 선택하거나 `BLOCKED_CAPACITY`를 기록한다.
3. 장시간 provisioning/예상 밖 state: Work Requests/오류 상세를 읽고 같은 요청을 반복 제출하지 않는다. 만들어진 부분 리소스(boot/IP/attachment 포함)를 먼저 조사한다.
4. 메뉴 이름이 다름: 공식 task body의 동등한 탭(예: Storage 또는 Attached Block Volumes)을 찾는다. 필드가 문서와 다르면 값·스크린샷에서 secrets를 가린 뒤 관리자 또는 Oracle Support에 확인하고 제출을 중단한다. 이 문서는 현재 tenancy의 live Console 화면을 검사한 결과가 아니다.

### 이 주제의 실패 지점

BucketAlreadyExists면 namespace/Region/name 충돌 여부를 확인하고 승인 suffix를 바꾼다. Archive에 잘못 올렸다면 Standard 즉시 다운로드를 기대하지 말고 restore/보관 과금 조건을 확인한다. 버전이 남으면 versioning 상태와 object versions를 조회한다.

## 리소스정리

최종 cmp/hash 증거 저장 뒤 owner04가 생성 guard를 통과한 `bootstrap.txt`와 성공 GET으로 생성한 `bootstrap.download.txt`의 소유권을 registry로 확인한다. 생성 시각/path/hash가 이번 실행과 일치할 때만 두 exact `$HOME/oci-roadmap-lab/` 경로를 제거한다. 실패 run의 미생성 파일/다른 소유 파일은 삭제하지 않는다. directory는 이번에 생성했고 다른 lab 파일이 없는 경우에만 rmdir한다. 보존하면 파일 owner/expiry를 별도 기록하고 재진입 전에 owner04가 반환/정리한다.

### owner04의 최종 bucket closeout

Cloud Computing/Shared Security 및 후속 Object Storage 소비자는 모두 이 bucket의 **같은 creator lease**를 사용한다. registry의 각 consumer(01/08/48 이후 실제 승인 소비자) 반환 UTC/owned object exact names/미완료/retention deadline/creator acknowledgment를 확인한다. Archive restore/다음날 lifecycle 관찰이면 승인 연장 또는 owner의 abort/owned cleanup/반환이 필요하며 expiry만으로 반환을 대신하지 않는다. 미반환/unknown owner가 있으면 BLOCKED_CONSUMER로 bucket 보존/예산 재승인을 요청한다.

1. owner04 자신의 `intro/bootstrap.txt`만 Console → Storage → Object Storage & Archive Storage → Buckets → 실제 bucket → Objects에서 exact name을 선택 → Delete → 대상 이름 확인 → Delete. 같은 Objects 목록을 필터 없이 모든 페이지 새로 읽고 bootstrap 이름 없음 기록; 타 owner object는 그 creator가 정리한다.
2. 승인 consumer 반환 뒤 동일 Region/lab-root/namespace/bucket **OCID와 이름**을 creation record에 대조한다. Objects 목록의 prefix/search를 해제하고 모든 object/해당 object version 표시를 검사한다. **버전/soft-delete가 활성 또는 이전에 활성된 record가 있으면** 기본 current-object 목록의 빈 상태만으로 진행하지 않는다; 아래 Delete preview의 `objects and object versions` count와 version owner의 공식 cleanup evidence를 함께 받는다.
3. bucket → Policies → Lifecycle policy rules: **0행**, Retention rules의 모든 페이지: **0행** 확인. Retention/locked rule 또는 lifecycle이 발견되면 owner registry와 관리자에게 반환하며 lock을 해제하거나 기간을 축소하는 실습을 여기서 추가하지 않는다. Replication policy 및 Pre-authenticated requests도 **없음**을 확인한다. 새 rule/PAR를 만들지 않는다.
4. Buckets 목록 → 같은 bucket의 Actions → **Delete**를 선택하여 **최종 confirmation 전에 preview count**를 읽는다. 현재 공식 Delete task는 Console이 objects/object versions, pre-authenticated requests, replication policy, uncommitted multipart uploads의 삭제를 막는 리소스 count를 수집하며 confirmation이 그 리소스까지 함께 삭제할 수 있다고 설명한다. **이 lab은 모든 count가 0일 때만** confirmation을 허용한다. Uncommitted multipart uploads의 count가 0인지 반드시 record에 쓴다. 하나라도 nonzero/unknown이면 **Cancel**, exact 유형/count/owner/UTC를 creator registry에 기록하고 해당 creator가 공식 task로 자기 자원만 정리하도록 반환한다. owner04가 일괄 Cascade 삭제를 누르지 않는다. count 확보 불가/부분 보기 권한이면 BLOCKED_INVENTORY다. [현재 Delete Bucket body](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/managingbuckets_topic-To_delete_a_bucket.htm).
5. zero inventory/consumer 반환/owner04 삭제승인 뒤 위 confirmation의 **Delete를 한 번** 누른다. 10초 간격으로 최대 600초 동안 같은 Region/Compartment의 Buckets 목록을 필터 없이 새로 고쳐 **creation record의 exact bucket OCID 없음**을 확인하고 UTC를 기록한다. GetBucket을 아래처럼 새 전용 evidence에서 함께 읽되 오류만으로 삭제 성공을 선언하지 않는다.

```bash
# Cloud Shell; Console Delete 1회 제출 후, owner04 same namespace/name/Region.
CLOSE_DIR=$(mktemp -d "$HOME/owner04-close.XXXXXXXX") || exit 1
if oci os bucket get --namespace-name "$NAMESPACE" --name "$BUCKET" --region "$REGION" --connection-timeout 10 --read-timeout 20 --max-retries 0 > "$CLOSE_DIR/bucket-after.json" 2> "$CLOSE_DIR/bucket-after.error"; then
  printf '%s\n' 'Still readable: compare actual id; continue bounded Console observation, no second Delete'
else
  RC=$?
  printf 'get exit=%s; admin must distinguish permission and absence\n' "$RC"
fi
```

같은 이름/다른 OCID로 재생성된 bucket은 이번 대상이 아니다. 관리자가 same-identity visibility를 확인하고 600초 안의 목록/Console Delete 상태 및 exact target 부재 증거를 반환하기 전 closeout은 미완료다. IAM 거부/timeout/Delete 실패/nonzero inventory는 request-id/UTC/count/OCID/JSON/error를 owner04/관리자에게 반환한다. 진행 중 재Delete 금지; 미해결이면 예산/기한 연장 승인 또는 Support 대응을 기록한다. bucket 삭제는 복구 불가이며 cancel해도 이미 삭제한 자원은 돌아오지 않는다. VM/VCN 삭제는 bucket 정리의 대체가 아니다.

6. owner04 registry에 actual bucket OCID/namespace/name/Region, zero count/consumer 반환, Delete 제출 UTC, 검증 UTC/evidence, 관리자 visibility 승인, owner00에 대한 lease RELEASED를 기록한다. CLOSE_DIR evidence는 승인 보존경로에서 hash 확인 후 exact 두 파일만 제거/rmdir한다. 앞 bootstrap local cleanup은 자기 소유 record를 따르며 다른 owner/공유 parent는 보존한다.


## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 아래 본문은 각 생성 필드·설정·검증·정리의 기술 근거이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [21] [Welcome to Oracle Cloud Infrastructure — Introduction to OCI](https://docs.oracle.com/en-us/iaas/Content/GSG/Concepts/baremetalintro.htm)
  - 확인 범위: Oracle Cloud Infrastructure (OCI) is.

### 추가 실행 절차 근거 (본문 확인)

- [bucket](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/managingbuckets_topic-To_create_a_bucket.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [upload](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/managingobjects_topic-To_upload_objects_to_a_bucket.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [cloudshell](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/cloudshellgettingstarted.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.

- [추가 세부 task: objectpolicy](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm) — 실제 본문을 확인한 필드·API/권한·삭제 단계 근거.
