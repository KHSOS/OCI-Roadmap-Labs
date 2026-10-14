> **기존 실습 초안 — 미검증**: 원래 93개 문서의 절차를 보존한 자료입니다. 새 튜토리얼의 품질 승인이나 OCI 실행 검증을 뜻하지 않습니다.

# Cloud Computing

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../../../Docs/README.md)를 확인하세요.

> 실행 절차 작성본 / OCI 미실행. Oracle 공식 본문 근거와 교육용 선택을 구분한다. secrets·실제 고객값·diagram 없음.

## 학습목표

- owner04의 private bucket에 합성 normal/peak object 두 개를 순서대로 업로드하고 개별 byte size의 증가를 기록한다.
- normal object를 새 파일로 다운로드해 cmp를 통과시키고 peak만 삭제한 뒤 목록이 normal 한 개로 돌아오는지 확인한다.

위 항목은 사용자 실행 후 확인할 완료 조건이며 현재 관측 결과가 아니다. 개념은 아래 해당 생성 필드·configuration·test 단계에서 함께 적용한다.

## 대응 Roadmap 항목

- 항목명: **Cloud Computing**
- Roadmap 경로: **Introduction → Cloud Computing**
- Node ID: `sequence-0-1`
- 상위 항목: **Introduction** (`sequence-0-0`)
- 문서 범위: 이 항목의 생성·선택·기능검증·정리. 실습 미실행; 아래 기대 패턴은 실제 출력 아님.
- 번호는 navigation 식별자이며 실행 순서는 사전준비의 단계별 의존성에 따른다.

## 사전준비

### 기준 계약과 실제 단계 선행조건

[공통 baseline-contract.json](../../../baseline-contract.json)의 Oracle Linux 9 / opc / CIDR / 비용 상한을 따른다. 문서 번호는 실행 순서가 아니다. **준비 station과 생성 이후 ready를 분리**한다. 아래 단계 외의 기능완료를 소급하여 요구하지 않는다.

- [04 Introduction to OCI — 단계 링크](../01_Introduction_to_OCI/00_Introduction_to_OCI.md#baseline-04-ready): **bucket PUT/GET/cmp 완료; BUCKET/NAMESPACE/REGION**. 실제 출력/소비자/expiry를 전달하고 해당 owner가 이 lab 종료까지 보존한다.

생성 전 조직 승인(identity/권한/예산/expiry/서비스 지원/limit/capacity)을 확인한다. 실제 OCID/IP는 아직 생성하지 않았으면 비워 두고 해당 Create/readback 뒤 채운다. 동일 creation-event-id/OCID는 한 번의 제출만 뜻한다. `INSTANCE_A_ID`는 owner10 vm-a의 OCID, `INSTANCE_B_ID`는 owner22 private-vm의 OCID이며 owner07 vm-b와 다르다. 실습에 없는 자원을 추가하지 않는다. 실제 checkpoint는 사용자 실행 전 비워 둔다.

### 입력값과 실행 위치

Console에서 선택한 값만 사용한다. `<...>`는 예시 OCID나 비밀값이 아니라 **사용자가 대체할 입력 자리**다. 대체하지 않은 채 명령을 실행하지 않는다. Console → 우측 상단 Cloud Shell/Code Editor → Cloud Shell을 열면 OCI CLI가 사전 인증된다. Region 변경 뒤에는 Shell을 다시 열거나 명령의 `--region`을 바꾼다. SSH는 Cloud Shell이 아니라 본인 Linux/WSL/macOS 터미널에서 수행한다. 공인 IP/32도 그 터미널의 출구 주소여야 한다.

```bash
# 아래는 입력/출력의 대입 형식이다. 표의 단계 전에 생성 후 ID/IP export 행을 실행하지 않는다.
 # 생성 후 검증용 ID/IP는 해당 creator의 실제 응답에서 넣는다.
export REGION='<actual_REGION>'
export BUCKET='<actual_BUCKET>'
export NAMESPACE='<actual_NAMESPACE>'
oci --version
```

이 변수 중 해당 절차에서 사용하는 값만 채운다. CLI 조회의 `data`를 증거 파일에 저장하고 실제 시간·작업 OCID·선택값을 기록한다. API key/private key/passphrase/인증 토큰은 기록하지 않는다. Cloud Shell 설정파일이나 세션 토큰을 출력하지 않는다. `oci --version`을 실행하는 주제의 기대 패턴은 버전 문자열뿐이다. namespace는 Object Storage 주제에서 별도 조회하며 다른 주제에는 요구하지 않는다. [Cloud Shell 공식 본문](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/cloudshellgettingstarted.htm).

### Workshop 입력 계약 — 단계와 host

| 입력/출력 | 값 또는 실제 획득 경로 | 사용할 시점 | 실행 위치 | 선택 이유 |
|---|---|---|---|---|
| Console/설정 필드 | Object names=intro/intro-demand.txt, intro/intro-demand-peak.txt; Standard; 합성 내용; overwrite 없음 | 생성 전 선택 → 생성 뒤 readback | 본문 Console/로컬 지정 | 이 topic의 최소 생성 범위 |
| `BUCKET` | owner04 Console 생성한 고유 이름 | 선행 완료/생성 후 | Console/Cloud Shell | object scope |
| `NAMESPACE` | oci os ns get --region "$REGION" --query data --raw-output | 생성 전/선행 완료 | Cloud Shell | Object Storage namespace |
| `REGION` | 승인된 기존 subscribed Region identifier | 생성 전 | Console/Cloud Shell | API endpoint/데이터 위치 |

생성 전에는 행의 승인값/선행 완료값만 지정하고, 생성 후 ID/IP는 해당 owner 응답으로 채운다. Cloud Shell과 로컬 SSH의 환경변수는 공유되지 않는다. 로컬에는 KEY와 접속 주소만 다시 지정하고 guest disk 명령은 로그인한 guest에서만 실행한다. `<...>`가 남으면 해당 명령은 실행하지 않는다.

## 권한

### 실제 task/API별 공식 권한 본문 대조

아래는 이 문서 생성·조회·변경·정리 API의 공식 표 본문을 읽은 인용이다. 위 family 예제는 편의상 넓은 compartment-scope grant이며 **최소권한 정책이라고 부르지 않는다**. verb/dependency 행 인용은 단일 API의 완전한 최소권한 도출이 아니다. 조직 Identity Domain Console user/group 작업은 해당 Domain 관리자 역할도 필요하며 legacy IAM user API 권한만으로 대체하지 않는다.

- `GetBucket`: BUCKET_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm).
- `DeleteObject`: OBJECT_DELETE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm).
- `GetNamespace`: API requires no permissions and returns the caller's namespace. Use the API to validate your credentials. OBJECTSTORAGE_NAMESPACE_READ permission is required if you include the optional compartmentId parameter. Use the compartmentId parameter to find the namespace for a third-party tenancy. — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm).
- `GetObject`: OBJECT_READ For a customer-managed key encrypted bucket, the objectstorage-<location> subject must have KEY_DECRYPT. — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm).
- `ListObjects`: OBJECT_INSPECT — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm).
- `PutObject`: The permission required depends on whether the object already exists in the bucket: OBJECT_CREATE is required when an object with that name doesn't already exist in the bucket. OBJECT_OVERWRITE is required when an object with that name already exists in the bucket. For a customer-managed key encrypted bucket, the objectstorage-<location> subject must have KEY_ENCRYPT. — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm).

owner01는 owner04 bucket의 object-only 소비자다. bucket 생성·변경·삭제 권한은 요청하지 않고 아래 실제 bucket 조건의 read buckets/manage objects를 관리자에게 승인받는다.

object-only 실습자는 아래 **조건부 read buckets**가 필요하다. 이 문서의 첫 PUT 전 `oci os bucket get`은 `GetBucket`/`BUCKET_READ`를 요구하므로 `manage objects`만으로는 gate를 통과할 수 없다. 관리자는 `<approved-bucket-name>`을 owner04의 실제 BUCKET으로 바꾸고 objects grant도 같은 조건으로 제한한다. bucket 생성·변경·삭제 권한은 부여하지 않는다.

```text
Allow group <domain>/<lab-group> to read buckets in compartment lab-root where target.bucket.name='<approved-bucket-name>'
Allow group <domain>/<lab-group> to manage objects in compartment lab-root where target.bucket.name='<approved-bucket-name>'
```

`target.bucket.name` 조건 비교는 case-insensitive이므로 대소문자만 다른 bucket에도 적용될 수 있다. 관리자는 승인 Compartment의 bucket 이름이 대소문자만으로 구분되지 않는지 확인한다. 조건부 GetBucket 조회가 거부되면 PUT 전 중단하고 관리자에게 권한을 확인한다. 정책은 교육용 scoped 예시이고 실제 identity-domain membership/조건부 정책 allow·deny 테스트는 미실행이다. Object Storage create/upload 공식문서의 Required IAM Policy 링크를 관리자와 확인한다. Cloud Shell을 쓰려면 별도 Cloud Shell 사용 허용이 필요하다.

### namespace 조회와 조건부 권한

`oci os ns get --region "$REGION" --query data --raw-output`는 caller namespace를 읽으며 optional compartmentId를 넣지 않는다. 공식 Object Storage permission 표는 이 경로가 별도 namespace permission을 요구하지 않는다고 명시한다. third-party tenancy namespace를 optional compartmentId로 조회하는 별도 확장만 OBJECTSTORAGE_NAMESPACE_READ를 확인한다. 이 lab에 불필요한 tenancy-wide namespace grant를 추가하지 않는다.

## 비용및주의사항

- 이 주제는 기존 private Standard bucket에 normal/peak 합성 텍스트 object **2개만** 추가한다. 승인 상한은 아래 두 파일의 실제 byte size 합계와 PUT/GET/LIST/DELETE 요청 및 해당 Region 전송 비용이다. 새 VM/volume/bucket은 만들지 않는다. peak를 기능검증 직후 지우고 normal도 consumer 반환 전에 정리한다. 저장량 감소를 autoscaling 또는 실제 청구 절감률로 쓰지 않는다.
- 생성/사용 당일 공식 단가와 Billing & Cost Management → Cost Analysis의 실제 Compartment/서비스 필터를 확인하되 청구 반영 지연을 기록한다. 예산 경보는 과금 차단이 아니다. 고객 데이터/secret/운영 자원을 사용하지 않는다.
- 이 문서는 Oracle 기술 본문 기반 교육 절차다. OCI/guest/IAM allow·deny 실측은 없으며 기대 결과를 실행 결과로 기록하지 않는다. 승인되지 않은 선택 확장은 실행하지 않는다.


## 실습구성

### 이번 Workshop에서 만드는 것과 진행 순서

이미 검증한 private bucket에 normal/peak 합성 object 두 개만 추가하여 사용량 변화를 관찰한다. autoscaling이나 비용 절감률은 검증하지 않는다.

1. **진입/선택:** bucket private/Standard 확인. 사전준비 링크의 지정 단계/실제 출력이 없으면 제출하지 않는다.
2. **실제 작업:** 충돌 없는 두 object PUT → 크기 목록 비교 → GET/cmp. 아래 Console 필드/명령을 해당 host에서 순서대로 실행한다. 같은 자원은 이름이 아니라 OCID/event로 재사용한다.
3. **완료/복구/반환:** peak 삭제/목록 회복. 결과검증 실패는 문제해결로 이동하고 동일 Create를 무작정 재제출하지 않는다.

**전달/정리 범위:** intro/intro-demand.txt와 peak만 삭제; bucket은 owner04 보존. 아래 정리 단계는 이 소유권을 넘어서 실행하지 않는다.


- **이번 생성/재사용/출력:** 신규: intro-demand 및 peak object 2개. 기존 bucket 재사용. 출력: before/peak/after 목록과 원본/download hash.
- **생성 목표:** Cloud Computing을 작은 Object Storage object의 생성·증가·삭제로 경험한다. VM autoscaling을 서론에서 강제하지 않는다.
- **완료 증거:** 생성 전 옵션 선택 + 실제 제출 대상 OCID + API readback + 해당 기능검증 + dependency 순서 정리. 실제 OCI 실행 checkpoint는 비워 둔다.
- **retention:** 예약 소비자와 expiry가 있는 resource만 보존. idle 유료 resource를 다음 주제 때문에 무기한 유지하지 않는다.


## 실습절차

### 명령 실행 gate — 실패하면 다음 생성으로 진행하지 않는다

1. 아래 Bash는 **전용 Bash 세션**에서만 사용한다. Console/Cloud Shell 조회, 본인 로컬 SSH, 로그인한 Oracle Linux guest를 섞지 않는다. `exit 1`은 이 전용 세션을 닫는 안전 중단이다. `<...>`·빈 입력·타 owner OCID가 남으면 실행하지 않는다.
2. 성공을 요구하는 파일 작성/조회/설치/attach 작업은 실패 시 중단한다. 실패한 API 뒤 기존 JSON이나 기존 다운로드 파일을 성공 근거로 읽지 않는다. 기대 deny는 `if command; then ...; else RC=$?; ...; fi`로 별도 기록하고, 실제 timeout/IAM 오류/로컬 파일 오류를 구별한다.
3. 첫 변경 전에 정확한 OCID·owner·현재 설정과 승인 expiry를 저장한다. 공유 자원 변경 중 다른 소비자가 시작하거나 원복값을 확인할 수 없으면 변경을 중단한다. 삭제 성공은 API 수락이 아니라 같은 identity/Region의 정확한 대상 재조회로 판정한다. `NotAuthorizedOrNotFound` 단독은 삭제 증거가 아니다.


### 개별 완주 보정 — 생성·검증·소유권

첫 PUT 전 같은 prefix에 기존 교육 object가 없는지 목록을 확인하고 다른 작업자의 object는 삭제하지 않는다. fresh output으로 GET/cmp한다. peak와 normal 두 이름만 정리하고 bucket은 owner04에 맡긴다.

1. 사전준비의 bucket은 private/Standard/versioning Disabled여야 한다. 같은 생성 session의 bucket 생성 증거와 namespace/Region을 확인한다. 클라우드 서비스가 인터넷을 통해 제공되는 것과 익명 public access는 다르다.
2. Cloud Shell에서 평상시와 피크용 파일을 만든다. 내용은 아래 합성 텍스트만 사용한다. 사용자 요구 증가를 object 두 개의 저장량 변화로 관찰하며 이것을 VM autoscaling 성공이라고 부르지 않는다. [upload objects](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/managingobjects_topic-To_upload_objects_to_a_bucket.htm).

```bash
set -euo pipefail
# Cloud Shell: 본 lab 전용 작업 경로와 원격 이름 충돌을 먼저 확인한다.
umask 077
test -d "$HOME" && test ! -L "$HOME" && test -O "$HOME" || exit 1
INTRO_PARENT="$HOME/oci-roadmap-lab"
test ! -L "$INTRO_PARENT" || exit 1
if test -e "$INTRO_PARENT"; then
  test -d "$INTRO_PARENT" && test -O "$INTRO_PARENT" && test "$(stat -c %a "$INTRO_PARENT")" = 700 || exit 1
else
  mkdir -m 700 -- "$INTRO_PARENT" || exit 1
fi
INTRO_WORK=$(mktemp -d "$INTRO_PARENT/owner01.XXXXXXXX") || exit 1
export INTRO_WORK
printf '%s\n' "$INTRO_WORK" # owner01 registry에 저장: 새 터미널 복구 입력
# owner04/bootstrap files stay in the parent; owner01 never chmods/removes them.
cd -- "$INTRO_WORK" || exit 1
set -C # all fixtures and JSON evidence are exclusive writes in this new owned directory
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
owned=['intro/intro-demand.txt', 'intro/intro-demand-peak.txt']
assert not set(owned).intersection(x['name'] for x in rows), 'STOP: owned object name exists; do not overwrite'
PY

# reserve scope/intent before first PUT; owner04 grants exclusive mutation of these two names.
python3 - <<'PY'
import json,os
with open('owner01-scope.json','x') as f:
    json.dump({k:os.environ[k] for k in ('REGION','NAMESPACE','BUCKET','INTRO_WORK')},f)
with open('owner01-successful-uploads.txt','x'): pass
with open('owner01-deleted.txt','x'): pass
PY
printf '%s\n' 'normal synthetic demand'  > "$INTRO_WORK/intro-demand.txt"
printf '%s\n' 'peak synthetic demand' > "$INTRO_WORK/intro-demand-peak.txt"
printf '%s\n' 'intro/intro-demand.txt PENDING' > pending-put.txt || exit 1
oci os object put --namespace-name "$NAMESPACE" --bucket-name "$BUCKET" --name intro/intro-demand.txt --file "$INTRO_WORK/intro-demand.txt" --region "$REGION" --no-overwrite || exit 1
printf '%s\n' 'intro/intro-demand.txt SUCCESS' >> owner01-successful-uploads.txt || exit 1
rm -- pending-put.txt || exit 1
oci os object list --namespace-name "$NAMESPACE" --bucket-name "$BUCKET" --prefix intro/intro-demand --all --region "$REGION" > demand-before.json || exit 1
printf '%s\n' 'intro/intro-demand-peak.txt PENDING' > pending-put.txt || exit 1
oci os object put --namespace-name "$NAMESPACE" --bucket-name "$BUCKET" --name intro/intro-demand-peak.txt --file "$INTRO_WORK/intro-demand-peak.txt" --region "$REGION" --no-overwrite || exit 1
printf '%s\n' 'intro/intro-demand-peak.txt SUCCESS' >> owner01-successful-uploads.txt || exit 1
rm -- pending-put.txt || exit 1
oci os object list --namespace-name "$NAMESPACE" --bucket-name "$BUCKET" --prefix intro/intro-demand --all --region "$REGION" > demand-peak.json || exit 1
```
3. Console → bucket → Objects의 두 이름과 Size를 확인한다. 목록의 각 byte size를 합산해 before/peak 차이를 기록한다. approximate bucket size나 청구 반영 지연과 혼동하지 않는다.
4. 첫 object를 다운로드해서 원본을 비교하고 peak object만 삭제한다. Delete 확인창에서 이름을 읽고 제출한다. [Standard의 즉시 읽기](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/managingbuckets_topic-To_create_a_bucket.htm).

```bash
oci os object get --namespace-name "$NAMESPACE" --bucket-name "$BUCKET" --name intro/intro-demand.txt --file "$INTRO_WORK/demand.download.txt" --region "$REGION" || exit 1
cmp "$INTRO_WORK/intro-demand.txt" "$INTRO_WORK/demand.download.txt" || exit 1
oci os object delete --namespace-name "$NAMESPACE" --bucket-name "$BUCKET" --name intro/intro-demand-peak.txt --region "$REGION" || exit 1
printf '%s\n' 'intro/intro-demand-peak.txt' >> owner01-deleted.txt || exit 1
oci os object list --namespace-name "$NAMESPACE" --bucket-name "$BUCKET" --prefix intro/intro-demand --all --region "$REGION" > demand-after.json || exit 1
```

## 결과검증


### baseline-01-ready


```bash
python3 - <<'PY'
import json
def rows(path):
    data=json.load(open(path))['data']
    if isinstance(data,dict): data=data['objects']
    assert isinstance(data,list), 'STOP: unexpected ListObjects envelope'
    assert all(isinstance(x,dict) and isinstance(x.get('name'),str)
               and type(x.get('size')) is int and x['size']>=0 for x in data)
    assert len({x['name'] for x in data})==len(data), 'STOP: duplicate object rows'
    return data
a,b,c=map(rows,['demand-before.json','demand-peak.json','demand-after.json'])
assert {x['name'] for x in a}=={'intro/intro-demand.txt'}
assert {x['name'] for x in b}=={'intro/intro-demand.txt','intro/intro-demand-peak.txt'}
assert {x['name'] for x in c}=={'intro/intro-demand.txt'}
assert sum(x['size'] for x in b)>sum(x['size'] for x in a)
normal='intro/intro-demand.txt'
assert a[0]['size']==c[0]['size']>0
assert next(x['size'] for x in b if x['name']==normal)==a[0]['size']
assert next(x['size'] for x in b if x['name']!=normal)>0
print('demand assertions passed')
PY
sha256sum "$INTRO_WORK/intro-demand.txt" "$INTRO_WORK/demand.download.txt"
```
기대 패턴: assertion 종료코드 0, 두 checksum 같음. 저장량 증가/감소만 입증한다. 짧은 lab로 비용 절감 비율이나 무한 탄력성을 입증하지 않는다.

**증거 기록:** 실제 명령·UTC 시간·exit code·가려진 OCID·옵션 readback·검증 조건을 남긴다. 현재 이 문서의 실행상태는 `NOT_EXECUTED`다. output/creation-event-id/result는 실행 전 비워 둔다. 오류/NOT_SELECTED를 PASS로 바꾸지 않는다.

### 이 시나리오의 functional acceptance

| Positive | Negative / 미완료 판정 | Recovery / 반환 |
|---|---|---|
| normal GET/cmp; peak 추가 목록 2개 | peak 삭제 후 해당 exact name 미존재를 목록으로 확인 | normal은 그대로 cmp; peak가 필요하면 owner 승인된 새 event로만 다시 업로드 |

모두 실행 시 확인할 기대 패턴이다. 생성 전 guard negative는 OCI 장애를 주입한 것과 다르며 다른 자원을 추가하지 않는다. 오류 메시지와 실제 exit code가 없는 항목은 NOT_EXECUTED 또는 BLOCKED로 남긴다.

## 문제해결

### 공통 오류의 진단 순서

1. `NotAuthorizedOrNotFound` 또는 목록에 리소스가 없음: 먼저 Region/Compartment/OCID와 identity domain/group을 확인한다. 존재하지 않는 OCID와 권한 부족 모두 같은 오류로 보일 수 있다. 관리자에게 request-id·작업 시각·서비스·가려진 대상 OCID를 제공한다. 비밀키/토큰은 보내지 않는다.
2. `LimitExceeded`/quota/capacity 부족: Billing & Cost Management → Limits, Quotas and Usage와 생성 화면의 지원 shape/AD를 확인한다. 운영 quota 증가나 다른 Region 구독을 임의 요청하지 않는다. 승인 범위 안의 더 작은 compatible shape 또는 다른 AD를 선택하거나 `BLOCKED_CAPACITY`를 기록한다.
3. 장시간 provisioning/예상 밖 state: Work Requests/오류 상세를 읽고 같은 요청을 반복 제출하지 않는다. 만들어진 부분 리소스(boot/IP/attachment 포함)를 먼저 조사한다.
4. 메뉴 이름이 다름: 공식 task body의 동등한 탭(예: Storage 또는 Attached Block Volumes)을 찾는다. 필드가 문서와 다르면 값·스크린샷에서 secrets를 가린 뒤 관리자 또는 Oracle Support에 확인하고 제출을 중단한다. 이 문서는 현재 tenancy의 live Console 화면을 검사한 결과가 아니다.

### 이 주제의 실패 지점

object put overwrite prompt가 뜨면 동일 prefix에 이전 실습 object가 있는지 확인한다. 다른 사용자 object를 overwrite하지 않는다. before/peak 집계가 예상과 다르면 개별 object 이름/size를 확인하고 approximate bucket count 대신 object list를 쓴다.

## 리소스정리

### 정상 종료·중간 실패의 공통 owner01 abort/cleanup station

첫 PUT 전에 기록한 INTRO_WORK 절대경로를 registry에서 받아 **같은 Cloud Shell HOME**의 새 전용 Bash에 지정한다. owner04가 두 exact name의 배타적 mutation 권한을 반환하지 않았거나 PENDING PUT의 API 응답이 없으면 자동 삭제하지 않는다. PENDING은 서버 수락 후 journal 손실일 수 있으므로 owner04/관리자가 request-id/UTC/원본 hash/실제 object GET을 대조해 이번 생성 여부를 확인하고 성공 journal에만 수동 승인 기록한다. 업로드 실패/권한 오류는 object 없음 증거가 아니다. 아래는 **성공 journal에 기록된 두 이름만** 정리한다; prefix의 다른 object는 보존한다.

```bash
set -euo pipefail
export INTRO_WORK='<registry_actual_owner01_run_absolute_path>'
test -d "$INTRO_WORK" && test ! -L "$INTRO_WORK" && test -O "$INTRO_WORK" || exit 1
cd "$INTRO_WORK" || exit 1
# scope.json의 REGION/NAMESPACE/BUCKET을 읽고 owner04 인계값과 대조한 뒤 export한다.
export REGION='<scope.json_actual_REGION>' NAMESPACE='<scope.json_actual_NAMESPACE>' BUCKET='<scope.json_actual_BUCKET>'
python3 - <<'PY'
import json,os
s=json.load(open('owner01-scope.json'))
assert all(s[k]==os.environ[k] for k in ('REGION','NAMESPACE','BUCKET','INTRO_WORK'))
PY
# A damaged/missing journal is not authorization to issue Delete.
for journal in owner01-successful-uploads.txt owner01-deleted.txt owner01-scope.json; do
  test -f "$journal" && test ! -L "$journal" && test -O "$journal" || exit 1
done
test ! -e pending-put.txt && test ! -L pending-put.txt || { printf '%s\n' 'BLOCKED_PENDING_PUT: owner reconciliation first'; exit 1; }
while read -r name status; do
  case "$name/$status" in intro/intro-demand.txt/SUCCESS|intro/intro-demand-peak.txt/SUCCESS) ;; *) exit 1;; esac
  ALREADY=$(python3 - "$name" <<'PY'
import sys,pathlib
print(int(sys.argv[1] in pathlib.Path('owner01-deleted.txt').read_text().splitlines()))
PY
) || exit 1
  if [ "$ALREADY" = 0 ]; then
    oci os object delete --namespace-name "$NAMESPACE" --bucket-name "$BUCKET" --name "$name" --region "$REGION" || exit 1
    printf '%s\n' "$name" >> owner01-deleted.txt || exit 1
  fi
done < owner01-successful-uploads.txt
FINAL=$(mktemp "$INTRO_WORK/owner01-final.XXXXXXXX.json") || exit 1
oci os object list --namespace-name "$NAMESPACE" --bucket-name "$BUCKET" --prefix intro/ --all --region "$REGION" > "$FINAL" || exit 1
python3 - "$FINAL" <<'PY'
import json,sys
r=json.load(open(sys.argv[1]))['data']
if isinstance(r,dict):r=r['objects']
assert not {'intro/intro-demand.txt','intro/intro-demand-peak.txt'}.intersection(x['name'] for x in r)
print('owner01 exact names absent; other consumer objects retained')
PY
```

Delete 성공 뒤 journal 기록이 끊긴 경우도 자동 재삭제하지 말고 exact-name list/owner 판단으로 복구한다. GET/cmp/list 실패는 오류/request-id/UTC와 위 journal을 보존하여 owner04에게 전달한다. 정상 peak 삭제 후에도 두 이름 최종 없음 assertion을 통과해야 반환한다.

owner04 registry에 `consumer=01`, REGION/NAMESPACE/BUCKET, INTRO_WORK, 성공·삭제 journal/FINAL/cmp/hash 경로, 종료 UTC, `outstanding_changes=none`, lease RELEASED(또는 PENDING 사유/승인 deadline), owner04 acknowledgment를 기록한다. bucket은 [owner04 cleanup](../01_Introduction_to_OCI/00_Introduction_to_OCI.md#리소스정리)에서 모든 소비자 종료 후 정리한다. 증거를 승인 보존 경로에 복사하고 hash 확인한 뒤 INTRO_WORK의 이번 실행 생성 파일을 record의 **exact paths**로만 rm한다. 미생성 파일은 건너뛰고 symlink/다른 owner 파일이면 중단한다. 마지막 `rmdir -- "$INTRO_WORK"`가 성공하면 run directory 정리 완료; 남은 파일을 glob/recursive delete로 숨기지 않는다. INTRO_PARENT/owner04 파일은 그대로 둔다. 승인 보존 사본의 hash를 확인한 후 같은 INTRO_WORK에서 아래를 실행한다. FINAL은 위 cleanup에서 새로 만든 정확한 경로이며 새 터미널에는 registry 값으로 다시 지정한다. PENDING이 있으면 이 단계도 중단한다.

```bash
# Cloud Shell; evidence archive/hash 확인 후, owner01 actual INTRO_WORK/FINAL만.
test ! -e "$INTRO_WORK/pending-put.txt" && test ! -L "$INTRO_WORK/pending-put.txt" || exit 1
case "$FINAL" in "$INTRO_WORK"/owner01-final.*.json) ;; *) exit 1;; esac
for f in "$INTRO_WORK/intro-demand.txt" "$INTRO_WORK/intro-demand-peak.txt" "$INTRO_WORK/demand.download.txt" "$INTRO_WORK/intro-bucket-before-write.json" "$INTRO_WORK/intro-collision-check.json" "$INTRO_WORK/owner01-scope.json" "$INTRO_WORK/owner01-successful-uploads.txt" "$INTRO_WORK/owner01-deleted.txt" "$INTRO_WORK/demand-before.json" "$INTRO_WORK/demand-peak.json" "$INTRO_WORK/demand-after.json" "$FINAL"; do
  test ! -L "$f" || exit 1
  if test -e "$f"; then
    test -f "$f" && test -O "$f" || exit 1
    rm -- "$f" || exit 1
  fi
done
cd "$HOME" || exit 1
rmdir -- "$INTRO_WORK" || { printf '%s\n' 'BLOCKED_LOCAL_REMAINDER: inspect remaining exact paths, no recursive delete'; exit 1; }
```


## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 아래 본문은 각 생성 필드·설정·검증·정리의 기술 근거이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [45] [What Is Cloud Computing? — Cloud Computing](https://www.oracle.com/cloud/what-is-cloud-computing/)
  - 확인 범위: Cloud computing is a model.

### 추가 실행 절차 근거 (본문 확인)

- [upload](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/managingobjects_topic-To_upload_objects_to_a_bucket.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [bucket](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/managingbuckets_topic-To_create_a_bucket.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.
- [cloudshell](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/cloudshellgettingstarted.htm) — 생성 필드·제약·readback 관계를 위 해당 단계에서 인용. 자료 본문을 실제 읽었으며 live Console/OCI 실행 확인과는 구분한다.

- [추가 세부 task: objectpolicy](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm) — 실제 본문을 확인한 필드·API/권한·삭제 단계 근거.
