# Audit

> 문서 작성 검토와 실행 검증은 별개입니다. OCI 실행은 **USER_EXECUTION_PENDING**이며 사용자 캡처는 별도 자료입니다. 미실행 자체는 문서 작성 승인 차단 사유가 아닙니다. 실제 결과 필드는 실행자가 채웁니다.

> 실행용 한국어 교육 문서. OCI 리소스 생성·명령 실행 결과는 아직 검증하지 않았으며 모든 기대 패턴은 미실행이다.

## 학습목표

- 83 exact bucket/Region/Compartment와 실제 생성·삭제 UTC로 Audit 범위를 설정한다. 새 bucket을 만들지 않는다.
- CreateBucket/DeleteBucket의 actual caller/requestId/resource/status를83 기록과 교차확인하고 unrelated bucket을 제외한다.
- 정상 raw 조회에서 wrong name 필터의 빈 결과와 exact name 복귀 결과를 비교한 뒤 본인 raw 파일만 정리한다. Audit event/retention은 변경하지 않는다.

실제 OCI 요청·응답/SQL·정리 증거는 실행자가 기록한다. 아래 기대값은 실측값이 아니다.

## 대응 Roadmap 항목

- 항목명: **Audit**
- Roadmap 경로: **Monitoring → Audit**
- Node ID: `sequence-9-5`
- 상위 항목: **Monitoring** (`sequence-9-0`)
- 문서 범위: 이 항목의 개념, 실제 설정 절차와 학습자 실행 검증.
- [전체 항목 목록](../README.md)

## 사전준비

**시작 조건:** 83 exact bucket create/delete 및 실제operation UTC 기록 완료

- [Events](03_Events.md) — bucket create API를 실제 실행하고 request/event 시각을 기록. bucket 삭제까지 완료해도 된다.

### 준비 완료 출력과 인수 확인

| 인수 항목 | 실제 출력·출처 | 다음 작업의 사용 위치 |
|---|---|---|
| 선행 완료 상태 | 83 exact bucket create/delete 및 실제operation UTC 기록 완료 | 첫 변경 전 재조회 |
| 비밀 없는 실행 입력 | actual EVENT_BUCKET/Compartment/Region/OP_START/END_UTC/request evidence,umask077 개인dir | 아래 numbered Console/명령 단계 |
| 반환/후속 인계 | read-only Audit 요약만 반환;raw 파일은 승인 보존 또는 본인 삭제 | 종료 UTC 이전 exact owner 반환 |

실제 Compartment/Region은 승인 Console에서 읽고 동일 identity/Region을 고정한다. Cloud Shell에서 `oci --version`과 `command -v python3`을 확인한다. 이 작업은 dig/HTTP 앱/VCN VM을 필요로 하지 않는다. placeholder·명칭만 인수한 상태는 준비 완료가 아니다. 비밀번호·키·확인 token은 출력표에 적지 않는다.

## 권한

실습 Compartment의 `read audit-events` 권한이 필요하다. Audit list/search는 리소스 관리 권한과 별개다. tenant 전체 감사 권한을 자동 요청하지 않는다.

공식 [Audit API 표](https://docs.oracle.com/iaas/Content/Identity/Reference/auditpolicyreference.htm)의 ListEvents=AUDIT_EVENT_READ이다. read audit-events는 감사 조회용이며 log-content, bucket 관리와 별개다. bucket scope의 actual audit query에만 승인하며 user/group 전체 tenancy 감사 접근은 이 과제에 자동 추가하지 않는다. raw Audit 저장·열람은 로컬 filesystem 권한으로 별도 보호한다.

operation 근거: [공식 권한/작업 본문](https://docs.oracle.com/en-us/iaas/Content/Audit/Tasks/viewinglogevents.htm). family grant는 minimum-permission 정책으로 주장하지 않는다. 실행 전에 관리자 승인된 identity/domain·실제 Compartment scope를 확인한다.

## 비용및주의사항

83의 실제 API 기록을 읽기만 하며 새 bucket/Logging group/VM/Trail은 생성하지 않는다. 최대1시간 안의 실제 operation window와 현재 Audit 조회·보존 조건을 확인한다. 관찰은 사전 승인된 최대20분/3회이며10분 예산만 있으면 ASYNC_PENDING으로 반환한다. raw Audit에는 principal/request metadata가 있어 개인700 디렉터리/600 파일로 보호하고 승인 없는 공개 저장은 금지한다. service Audit retention/event는 변경하지 않는다.

## 실습구성

### 워크숍 순서와 입력값

**진행 순서:** 83 exactbucket/operationUTC 인수 →Audit same scope/time →caller/requestId/2xx대조 →좁은시간복구 →raw삭제

**선택 이유:** 새 bucket을 만들지 않으며 이전 작업에도 현재시간 대신 실제 작업window를 사용한다.

| 입력 | 값 또는 정확한 출처 | 실행 위치 | 생성 전/후 | 이유 |
|---|---|---|---|---|
| EVENT_BUCKET/COMPARTMENT_OCID | 83actualbucket/scope | Cloud Shell | 전 | 다른Bucket증거제외 |
| START/END | 실제작업전후15분RFC3339 UTC | Cloud Shell | 전 | 현재시각자동덮어쓰기금지 |


값이 다른 터미널에 자동 전파되지 않습니다. Console readback으로 비밀 없는 값만 인계하며, placeholder는 실제값으로 교체한 뒤 실행합니다. 새 리소스 생성 전 이름 충돌·소유자·최대 비용·종료 UTC를 확인하고 선택하지 않은 분기는 **NOT_SELECTED**로 기록합니다.


83의 실제 CreateBucket/DeleteBucket API를 Audit에서 추적한다. Logging custom marker와 control-plane Audit 차이를 확인한다.


### 이번 실습의 생성·재사용·앱 경계

| 구분 | 이번 주제의 실제 범위 |
|---|---|
| 새로 생성 | 본인rawAudit 파일만 |
| 재사용 | 83실제bucket API 기록 |
| 앱/작업부하 및 변경 | read-only filter control;앱/OCI리소스 생성 없음 |
| 실행/반환 순서 | 83 exact bucket create/delete 및 실제operation UTC 기록 완료 → read-only Audit 요약만 반환;raw 파일은 승인 보존 또는 본인 삭제 |

새로 생성하지 않는 항목을선행완료로가장하지않는다.위표의실제OCID/주소와진행순서를첫mutation전에확인한다.

## 실습절차

### Audit 재현 가능한 증거 연계
83의 실제 `EVENT_BUCKET`, Compartment/Region, CreateBucket UTC를 인수받는다. 오래된 작업이면 현재 시각 기준30분 대신 recorded operation 전후15분 RFC3339 범위를 직접 입력한다. CLI JSON field명은 kebab-case 반환 구조를 읽고 exact bucket name+operation으로 좁힌다. 관련 없는 Bucket event는 과제 증거가 아니다.
잘못된 Compartment/time filter로 빈 결과를 관찰했다면 정상 scope/time 복구 조회가 재현되는지 확인한다. Audit event를 삭제·수정하는 장애 실험은 없다. CreateBucket 2xx와 caller/requestId를 83 resource/time과 연결하고 삭제 수행 여부에 따라 DeleteBucket 관측을 구분한다. raw JSON에는 개인 principal이 있으므로 `umask 077`로 저장하고 공개하지 않는다.

### 실행 단계

1. Identity & Security → Audit. Region **83 실행 Region**, Compartment **83 bucket Compartment**, 시간 범위 **실제 생성/삭제 전후 15분**으로 설정한다. API operation **CreateBucket** 및 **DeleteBucket**, resource bucket name을 찾아 펼친다.[T1]
2. caller principal, requestId, request method/path, response status, resourceId, eventTime를 확인한다. eventTime과 email 수신 시간이 같을 필요는 없다. payload 전체나 인증 header를 Git에 올리지 않는다.
3. CLI로 같은 범위를 조회한다.
```bash
set -euo pipefail
umask 077
if test -e audit-lab.json || test -L audit-lab.json; then
  printf 'audit evidence path exists: preserve and stop\n' >&2; exit 1
fi
set -o noclobber
export EVENT_BUCKET='83에서 실제 생성한 동일 bucket 이름'
export START='83 actual operation의15분전 UTC RFC3339'
export END='83 actual operation의15분후 UTC RFC3339'
# 예시날짜나현재30분을재사용하지말고실제operation기록에서값을입력한다.
if ! oci audit event list --compartment-id "$COMPARTMENT_OCID" --start-time "$START" --end-time "$END" --all > audit-lab.json; then
  printf 'LIST_FAILED: partial evidence retained; do not parse or reuse path\n' >&2
  exit 1
fi
python3 - <<'PY'
import json
for e in json.load(open('audit-lab.json')).get('data',[]):
    d=e.get('data',{}); n=d.get('event-name','')
    if n in ('CreateBucket','DeleteBucket') and d.get('resource-name') == __import__('os').environ['EVENT_BUCKET']:
        identity=d.get('identity') or {}
        request=d.get('request') or {}
        print(e.get('event-time'),n,d.get('resource-name'),d.get('response',{}).get('status'),
              identity.get('principal-id'),request.get('id'),d.get('resource-id'))
PY
```
4. 실제 bucket name과 operation을 대조한다. Audit 반영이 지연되면 조회 범위/Compartment/Region을 확인 후 공식 task는 통상15분(실제 지연은 변동)을 설명한다. 관찰 예산을 **최대20분/3회**로 생성 전에 승인하고 그 안에서만 조회한다. 예산이10분뿐이면10분에 ASYNC_PENDING으로 중단·반환하고 미관측을 API 실패로 확정하지 않는다. 로그 반영이 늦은 상태를 API 실행 실패로 단정하지 않는다.
5. 아래 검증값은 민감하지 않은 요약만 별도 기록하고 raw audit-lab.json을 공유하지 않는다.


### 진행·종료 판정
설정 readback·기능시험·원복·정리를 각각 실제 응답으로 기록한다. 기대값은 미실행이며 실패/만료 시 새 mutation을 중단하고 아래 topic별 종료 단계로 반환한다.

### 같은 실제 Audit 응답의 negative filter와 복귀
첫 정상 exact bucket event를 찾은 뒤 **이미 조회한 동일 raw 파일**에서 아래 비교를 수행한다. wrong name은 test filter 입력이며 OCI event를 생성하지 않는다.
```bash
python3 - <<'PY'
import json,os
rows=json.load(open('audit-lab.json')).get('data',[])
actual=os.environ['EVENT_BUCKET']
wrong=actual+'-not-created-in-this-lab'
for name in (actual,wrong,actual):
    matches=[e for e in rows if (e.get('data') or {}).get('resource-name')==name
             and (e.get('data') or {}).get('event-name') in ('CreateBucket','DeleteBucket')]
    print(name, 'actual_matching_event_count=',len(matches))
PY
```
wrong 이름에 실제 작업기록이 있으면 이 negative fixture가 유효하지 않으므로 다른 미사용 이름을 확인한다. actual positive rows가 없으면 두 empty 결과를 성공으로 판정하지 않는다. 원래 UTC window와 credentials는 바꾸지 않는다.


### 번호 순서로 완료하는 원복·정리

6. **Audit 종료 및 실패 반환.** 이미받은동일raw파일에서actual→wrongname→actual filter결과를비교한다.positive없는empty 비교는실패다.조회완료뒤승인보존이없는본인raw파일만삭제/부재확인하고Audit event/retention은건드리지않는다.

실제 정리/원복 response, exact resource 또는rule ID,완료UTC와남은owner/expiry를비밀없는개인실행기록에남긴다.위의guarded복구가이미완료된경우중복mutation하지않는다.

## 결과검증

**기대 패턴(미실행):** 동일 실제 bucket name/resource 및 CreateBucket(성공 2xx)와 승인된 caller. DeleteBucket 수행 시 해당 operation도 존재. 83 email event와 Audit request/event는 서로 다른 evidence다.

실행 기록(미기입): 실행 UTC __ / 실제 resource OCID __ / 요청·SQL 실제 응답 __ / 공식 조건 차이 __ / 정리 완료 UTC __.

## 문제해결

Audit UI filters가 CLI 결과를 숨기면 operation filter를 해제하고 requestId/resource-name으로 좁힌다. log-content 권한만으로 audit-events가 읽히는 것으로 가정하지 않는다.

## 리소스정리

Audit retention은 OCI 관리이므로 이 실습에서 Audit event를 삭제하거나 retention을 바꾸지 않는다. local raw audit-lab.json만 안전하게 삭제한다.

### 소유자별 종료와 exact 대상 재조회
85 owner는 비공개 evidence 보존승인이 없는 own audit-lab.json만 삭제·파일 부재를 읽는다. 원본 service Audit 삭제/retention 변경은 없다.83와86의 cleanup 반환에 영향 없는 read-only query 완료 UTC를 기록한다.

실제 정리 UTC __ / exact resource 또는 rule ID __ / 삭제 후 조회 응답 __ / 잔존 resource·만료·owner __. 승인 없는 잔존 비용은 다음 실습을 이유로 연장하지 않는다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 개념은 해당 생성·변경·조회 단계에서 입력 선택과 실제 결과 해석을 위해 사용하며, 공식 예시는 실제 실습 실행 결과가 아닙니다.

- [1] [Overview of Audit — Audit](https://docs.oracle.com/en-us/iaas/Content/Audit/Concepts/auditoverview.htm)
  - 확인 범위: Describes the Oracle Cloud Infrastructure Audit.

### 실습 task 본문 근거

- [T1] [Tasks / viewinglogevents.htm](https://docs.oracle.com/en-us/iaas/Content/Audit/Tasks/viewinglogevents.htm) — 본문 절차·입력 필드·지원 조건을 읽어 실습에 적용.
- [T2] [Concepts / auditoverview.htm](https://docs.oracle.com/en-us/iaas/Content/Audit/Concepts/auditoverview.htm) — 본문 절차·입력 필드·지원 조건을 읽어 실습에 적용.

