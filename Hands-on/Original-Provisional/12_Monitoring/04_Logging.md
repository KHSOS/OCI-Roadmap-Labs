> **기존 실습 초안 — 미검증**: 원래 93개 문서의 절차를 보존한 자료입니다. 새 튜토리얼의 품질 승인이나 OCI 실행 검증을 뜻하지 않습니다.

# Logging

> 문서 작성 검토와 실행 검증은 별개입니다. OCI 실행은 **USER_EXECUTION_PENDING**이며 사용자 캡처는 별도 자료입니다. 미실행 자체는 문서 작성 승인 차단 사유가 아닙니다. 실제 결과 필드는 실행자가 채웁니다.

> 실행용 한국어 교육 문서. OCI 리소스 생성·명령 실행 결과는 아직 검증하지 않았으며 모든 기대 패턴은 미실행이다.

## 학습목표

- 전용 log group/custom log 각1개를 만들고 type/Enabled/최소 retention과 실제 OCID를 읽는다.
- 현재 UTC marker1개를 PutLogs로 전송하고 same scope/time SearchLogs의 실제 id/source/data를 receipt와 구분하여 확인한다.
- wrong marker 빈 결과와 정상 marker 복귀를 비교하고 custom log→빈 group 삭제 readback 및 본인 payload 파일 정리를 수행한다.

실제 OCI 요청·응답/SQL·정리 증거는 실행자가 기록한다. 아래 기대값은 실측값이 아니다.

## 대응 Roadmap 항목

- 항목명: **Logging**
- Roadmap 경로: **Monitoring → Logging**
- Node ID: `sequence-9-4`
- 상위 항목: **Monitoring** (`sequence-9-0`)
- 문서 범위: 이 항목의 개념, 실제 설정 절차와 학습자 실행 검증.
- [전체 항목 목록](../../../Docs/README.md)

## 사전준비

**시작 조건:** Bash Cloud Shell OCI CLI/Python3 인증 및승인 scope/Region 준비



### 준비 완료 출력과 인수 확인

| 인수 항목 | 실제 출력·출처 | 다음 작업의 사용 위치 |
|---|---|---|
| 선행 완료 상태 | Bash Cloud Shell OCI CLI/Python3 인증 및승인 scope/Region 준비 | 첫 변경 전 재조회 |
| 비밀 없는 실행 입력 | actual COMPARTMENT_OCID/OCI_CLI_REGION,빈개인작업dir,marker84·UTC/expiry;group/log는본실습 생성후 획득 | 아래 numbered Console/명령 단계 |
| 반환/후속 인계 | PutLogs receipt와actual UUID 검색값·본인log/group 삭제 기록 | 종료 UTC 이전 exact owner 반환 |

실제 Compartment/Region은 승인 Console에서 읽고 동일 identity/Region을 고정한다. Cloud Shell에서 `oci --version`과 `command -v python3`을 확인한다. 이 작업은 dig/HTTP 앱/VCN VM을 필요로 하지 않는다. placeholder·명칭만 인수한 상태는 준비 완료가 아니다. 비밀번호·키·확인 token은 출력표에 적지 않는다.

## 권한

관리자는 `manage log-groups` 및 `use log-content`(PutLogs), `read log-content`(SearchLogs)를 실습 Compartment로 부여한다. `manage log-content`는 조회·발행까지 넓게 포함하므로 필요 범위로 제한할 수 있다. 전용 custom log creation 및 Log Search 권한은 공식 Logging policy table에서 확인한다.

CreateLogGroup/CreateLog=LOG_GROUP_CREATE; GetLogGroup/GetLog=LOG_GROUP_READ; DeleteLogGroup/DeleteLog=LOG_GROUP_DELETE; ingestion은 LOG_CONTENT_PUSH(use log-content), 조회는 LOG_CONTENT_READ(read에 포함). use는read를 포함하므로 중복 read grant는 필수 아님. API 표가 ListSearchLogs로 적힌 곳과 실제 SearchLogs 이름 차이를 기록하며 공식 SearchLogs SDK 본문과 scope를 함께 사용한다.

operation 근거: [공식 권한/작업 본문](https://docs.oracle.com/en-us/iaas/Content/Identity/policyreference/loggingpolicyreference.htm). family grant는 minimum-permission 정책으로 주장하지 않는다. 실행 전에 관리자 승인된 identity/domain·실제 Compartment scope를 확인한다.

## 비용및주의사항

전용 log group1개/custom log1개에 marker1개만 PutLogs로 보낸다. 현재 Logging ingestion·저장·retention·검색 가격/한도를 승인한다. 최소 제공 retention을 선택하되30일을 무료 보존으로 해석하지 않는다. 최초positive/negative/복귀 총3회, 관찰10분의 absolute 종료를 정한다. Console 조회와CLI는 같은 예산의 대안이다. VM agent/dynamic group/새 server는 만들지 않으며 WAF service log/Audit를 삭제하지 않는다.

## 실습구성

### 워크숍 순서와 입력값

**진행 순서:** group/customlog →1marker PutLogs →same scope/time SearchLogs →wrongmarker빈결과 →실제marker →삭제재조회

**선택 이유:** custom log는71 WAF service log 및85 Audit와 별도 리소스다.

| 입력 | 값 또는 정확한 출처 | 실행 위치 | 생성 전/후 | 이유 |
|---|---|---|---|---|
| LOG_OCID/LOG_GROUP_OCID | actualcustomlog/group | Cloud Shell | 후 | WAFservice와혼동금지 |
| marker/scope | roadmap-log-marker-84 /실제comp/group/log | CLI/Console | 후 | time/scope같게검색 |


값이 다른 터미널에 자동 전파되지 않습니다. Console readback으로 비밀 없는 값만 인계하며, placeholder는 실제값으로 교체한 뒤 실행합니다. 새 리소스 생성 전 이름 충돌·소유자·최대 비용·종료 UTC를 확인하고 선택하지 않은 분기는 **NOT_SELECTED**로 기록합니다.


custom log resource 생성 후 실제 PutLogs 호출로 marker를 보내고 SearchLogs로 조회한다.


### 이번 실습의 생성·재사용·앱 경계

| 구분 | 이번 주제의 실제 범위 |
|---|---|
| 새로 생성 | loggroup/customlog 각1개·marker1개 |
| 재사용 | 기존Cloud Shell/인증scope |
| 앱/작업부하 및 변경 | CLI PutLogs publisher가workload;agent/VM/service log 없음 |
| 실행/반환 순서 | Bash Cloud Shell OCI CLI/Python3 인증 및승인 scope/Region 준비 → PutLogs receipt와actual UUID 검색값·본인log/group 삭제 기록 |

새로 생성하지 않는 항목을선행완료로가장하지않는다.위표의실제OCID/주소와진행순서를첫mutation전에확인한다.

## 실습절차

### custom log end-to-end 오류·복구 과제
custom log의 type=Custom, enabled=true, group/log OCID를 Console에서 읽는다. local log-batch.json 기존 파일 확인 후 1 message만 게시한다. marker를 명시적으로 바꾼 search(`roadmap-log-marker-not-sent`)가 빈 결과인 negative control과 실제 marker84 조회 결과를 동일 time/scope에서 비교한다. marker가 없는 결과를 합성하지 않는다.
Logging query field는 실제 반환 구조를 읽어 확인한다. data 문자열은 scope 안의 `logContent.data`이고 source/time/id는 실제 반환 location에서 기록한다. scope는 큰따옴표, data literal은 작은따옴표로 보존한다. 최대10분/3회 안에 검색되지 않으면 receipt만으로 완료 처리하지 않는다. 삭제 후 exact log/group가 목록에서 사라졌는지 readback한다.

### 실행 단계

1. Observability & Management → Logging → Log groups → Create log group. Name **roadmap-custom-logs**, Compartment **실습 Compartment**. Create를 누르고 생성 group OCID를 읽는다. Logging → Logs → Create custom log에서 Name **roadmap-custom**, Compartment **실습 Compartment**, Log group **roadmap-custom-logs**를 선택한다. Show additional options → Retention **30 days**(현재 제공 최소값을 확인) → Next → Create agent configuration 페이지의 **Add configuration later** → Create. 이는 CLI PutLogs 트랙이므로 Host groups/dynamic group/agent를 추가하지 않는다. 실제 log type Custom/Enabled on 및 log/group OCID를 재조회한다. [정확한 Log 생성 task](https://docs.oracle.com/en-us/iaas/Content/Logging/Task/create-logging-log.htm)와 [Log group 생성 task](https://docs.oracle.com/en-us/iaas/Content/Logging/Task/create-logging-log-group.htm)를 따른다.
2. Bash/Cloud Shell에서 실제 log OCID를 export하고 현재 UTC message batch를 만든다.[T1]
```bash
set -euo pipefail
export LOG_OCID='실제 custom log OCID'
export LOG_GROUP_OCID='실제 log group OCID'
test ! -e log-batch.json || { printf 'existing lab payload: stop and review owner\n'; exit 1; }
python3 - <<'PY'
import json,datetime,uuid
now=datetime.datetime.now(datetime.timezone.utc).isoformat().replace('+00:00','Z')
b=[{'source':'roadmap-cli','type':'roadmap.marker','subject':'logging-lab',
'defaultlogentrytime':now,'entries':[{'id':str(uuid.uuid4()),'time':now,'data':'roadmap-log-marker-84'}]}]
open('log-batch.json','w').write(json.dumps(b))
print('actual entry UUID:',b[0]['entries'][0]['id'],'UTC:',now)
PY
if ! oci logging-ingestion put-logs --log-id "$LOG_OCID" --specversion 1.0 --log-entry-batches file://log-batch.json; then
  printf 'PUT_FAILED: stop; do not search or republish automatically\n' >&2
  exit 1
fi
```
2a. payload의 `defaultlogentrytime`(모두 소문자)는 현재 OCI CLI 3.68.1 offline 생성 schema와 일치한다. camelCase로 임의 수정하지 않는다. PutLogs 성공 receipt와 SearchLogs 결과는 별도 증거다.

3. 첫positive검색은아래CLI4단계또는Console3단계중하나만선택한다.두방식을중복실행하면negative/복귀를포함한총3회예산을초과한다. Logging → Search에서 Compartment **실습 Compartment**, Log group **roadmap-custom-logs**, Log **roadmap-custom**, Time range **최근 15분**. Advanced search는 아래 scope에서 marker를 검색한다.[T2]
```text
search "<compartment OCID>/<log group OCID>/<log OCID>" | where logContent.data = 'roadmap-log-marker-84' | sort by datetime desc
```
3a. scope의 큰따옴표와 marker의 작은따옴표는 Oracle SearchLogs SDK 본문의 `search \"scope\" | where level = 'INFO'` 형식을 따른다. shell은 scope 큰따옴표를 `\"`로 보존한다. 아래 CLI 문자열은 실제 offline Bash argument 검사에서 scope 큰따옴표와 marker 작은따옴표가 보존됨을 확인했다. 이는 OCI 서버 query acceptance 실측은 아니다.

4. CLI query도 실제 scope와 UTC 범위를 만들어 실행한다. 아래는 최근 15분의 값이므로 오래된 예시 날짜를 쓰지 않는다.
```bash
START=$(date -u -d '15 minutes ago' +%Y-%m-%dT%H:%M:%SZ)
END=$(date -u +%Y-%m-%dT%H:%M:%SZ)
oci logging-search search-logs --time-start "$START" --time-end "$END" --search-query "search \"$COMPARTMENT_OCID/$LOG_GROUP_OCID/$LOG_OCID\" | where logContent.data = 'roadmap-log-marker-84' | sort by datetime desc"
```
5. 실제 entry id/time/source/data를 읽는다. API response만 accepted인 경우는 emit 검증이고 query 결과까지 읽어야 end-to-end 완료다. WAF 로그는 71에서 활성화한 별도 service log scope로 검색한다.


### 진행·종료 판정
설정 readback·기능시험·원복·정리를 각각 실제 응답으로 기록한다. 기대값은 미실행이며 실패/만료 시 새 mutation을 중단하고 아래 topic별 종료 단계로 반환한다.

### Executable actual log search negative marker control
3~4단계와**같은 START/END 및exactscope**로1회negative marker를검색한뒤정상marker를재조회한다. 전체query는정상최초1회+negative1회+정상복귀1회=최대3회다.
```bash
oci logging-search search-logs --time-start "$START" --time-end "$END" --search-query "search \"$COMPARTMENT_OCID/$LOG_GROUP_OCID/$LOG_OCID\" | where logContent.data = 'roadmap-log-marker-not-sent' | sort by datetime desc"
oci logging-search search-logs --time-start "$START" --time-end "$END" --search-query "search \"$COMPARTMENT_OCID/$LOG_GROUP_OCID/$LOG_OCID\" | where logContent.data = 'roadmap-log-marker-84' | sort by datetime desc"
```
negative는 entries 없음, normal은 전송한 실제id/time/source/data다. timewindow 밖의 조회를 negative 성공으로 처리하지 않는다. CLI/Console을 모두 사용하면 같은 총 조회budget 안에서 선택한다.

### Exact entry evidence
발행된 payload의 출력 UUID/UTC와 실제 PutLogs 반환의 비밀 없는 receipt만 개인 작업표에 기록한다. Search 결과 detail의 actual id/time/source/data가 **그 UUID**와 일치해야 한다. 다른 실행 marker의 행은 성공 증거가 아니다. receipt 성공은 ingestion/query 성공과 별개다.


### 번호 순서로 완료하는 원복·정리

6. **Logging 종료 및 실패 반환.** same START/END의actualmarker복귀결과를확인한뒤exact customlog Actions/Delete/Confirm→Logs부재→group가빈것확인→group Delete/Confirm/목록부재를읽는다.본인log-batch.json만정리하며71service log/Audit는보존한다.

실제 정리/원복 response, exact resource 또는rule ID,완료UTC와남은owner/expiry를비밀없는개인실행기록에남긴다.위의guarded복구가이미완료된경우중복mutation하지않는다.

## 결과검증

**기대 패턴(미실행):** Search result logContent.data가 정확히 roadmap-log-marker-84이며 source=roadmap-cli, time은 방금 UTC. PutLogs의 반환 receipt를 조회 결과로 꾸미지 않는다.

실행 기록(미기입): 실행 UTC __ / 실제 resource OCID __ / 요청·SQL 실제 응답 __ / 공식 조건 차이 __ / 정리 완료 UTC __.

## 문제해결

PutLogs schema error면 `oci logging-ingestion put-logs --generate-param-json-input log-entry-batches`로 현재 CLI schema를 확인한다. 대소문자 field에 유의한다. Search가 비면 isEnabled, Region, 범위·log OCID·ingestion 지연을 확인하고 최대 10분 후 실패로 기록한다.

## 리소스정리

custom log 삭제 → empty log group 삭제. 본인 local log-batch.json 삭제. service/production log, tenancy Audit log는 삭제하지 않는다.

### 소유자별 종료와 exact 대상 재조회
84 owner: Logging→Log groups→actual group→Logs→exact custom log Actions/Delete/confirm → Logs 목록 부재. group에 다른 logs가 없음을 읽고 exact group Delete/confirm → Log groups 부재. payload 파일은 본인 file만 제거·부재 확인한다. WAF service log/Audit는 대상이 아니다.

실제 정리 UTC __ / exact resource 또는 rule ID __ / 삭제 후 조회 응답 __ / 잔존 resource·만료·owner __. 승인 없는 잔존 비용은 다음 실습을 이유로 연장하지 않는다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 개념은 해당 생성·변경·조회 단계에서 입력 선택과 실제 결과 해석을 위해 사용하며, 공식 예시는 실제 실습 실행 결과가 아닙니다.

- [28] [Logging Overview — Logging](https://docs.oracle.com/en-us/iaas/Content/Logging/Concepts/loggingoverview.htm)
  - 확인 범위: The Oracle Cloud Infrastructure Logging service.

### 실습 task 본문 근거

- [T1] [logging-ingestion / put-logs.html](https://docs.oracle.com/iaas/tools/oci-cli/latest/oci_cli_docs/cmdref/logging-ingestion/put-logs.html) — 본문 절차·입력 필드·지원 조건을 읽어 실습에 적용.
- [T2] [Concepts / using_the_api_searchlogs.htm](https://docs.oracle.com/en-us/iaas/Content/Logging/Concepts/using_the_api_searchlogs.htm) — 본문 절차·입력 필드·지원 조건을 읽어 실습에 적용.
- [T3] [Task / managinglogs.htm](https://docs.oracle.com/en-us/iaas/Content/Logging/Task/managinglogs.htm) — 본문 절차·입력 필드·지원 조건을 읽어 실습에 적용.


- [T-IAM] [Logging LOG_GROUP_CREATE / LOG_CONTENT_PUSH / LOG_CONTENT_READ](https://docs.oracle.com/en-us/iaas/Content/Identity/policyreference/loggingpolicyreference.htm) — Logging LOG_GROUP_CREATE / LOG_CONTENT_PUSH / LOG_CONTENT_READ 본문을 실제 읽어 적용.
