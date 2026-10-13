# Monitoring

> 문서 작성 검토와 실행 검증은 별개입니다. OCI 실행은 **USER_EXECUTION_PENDING**이며 사용자 캡처는 별도 자료입니다. 미실행 자체는 문서 작성 승인 차단 사유가 아닙니다. 실제 결과 필드는 실행자가 채웁니다.

> 실행용 한국어 교육 문서. OCI 리소스 생성·명령 실행 결과는 아직 검증하지 않았으며 모든 기대 패턴은 미실행이다.

## 학습목표

- 실제 Region/Compartment와 realm ingestion endpoint를 준비하고 roadmap_lab/LabHeartbeat/lab=roadmap stream에 UTC value1 한 점을 발행한다.
- failed-metrics-count=0과 같은 time window의 CLI 또는 Metrics Explorer의 timestamp/value를 읽어 accepted와 실제 조회를 구분한다.
- wrong dimension 빈 결과를 정상 dimension 재조회와 비교하고 publisher·본인 파일을 정리한다. datapoint 삭제를 시도하지 않는다.

실제 OCI 요청·응답/SQL·정리 증거는 실행자가 기록한다. 아래 기대값은 실측값이 아니다.

## 대응 Roadmap 항목

- 항목명: **Monitoring**
- Roadmap 경로: **Monitoring**
- Node ID: `sequence-9-0`
- 상위 항목: 없음 — 최상위 학습 항목
- 문서 범위: 이 항목의 개념, 실제 설정 절차와 학습자 실행 검증.
- [전체 항목 목록](../README.md)

## 사전준비

**시작 조건:** Bash Cloud Shell OCI CLI 인증과 Python3, 승인 scope/realm endpoint 준비



### 준비 완료 출력과 인수 확인

| 인수 항목 | 실제 출력·출처 | 다음 작업의 사용 위치 |
|---|---|---|
| 선행 완료 상태 | Bash Cloud Shell OCI CLI 인증과 Python3, 승인 scope/realm endpoint 준비 | 첫 변경 전 재조회 |
| 비밀 없는 실행 입력 | actual COMPARTMENT_OCID/OCI_CLI_REGION/realm/query·ingestion HTTPS endpoint, 빈 개인 작업dir,UTC/expiry | 아래 numbered Console/명령 단계 |
| 반환/후속 인계 | same-stream accepted/query 실제값·scope/endpoint를81에 인계 | 종료 UTC 이전 exact owner 반환 |

실제 Compartment/Region은 승인 Console에서 읽고 동일 identity/Region을 고정한다. Cloud Shell에서 `oci --version`과 `command -v python3`을 확인한다. 이 작업은 dig/HTTP 앱/VCN VM을 필요로 하지 않는다. placeholder·명칭만 인수한 상태는 준비 완료가 아니다. 비밀번호·키·확인 token은 출력표에 적지 않는다.

## 권한

전용 Compartment에 한정한 정책 예:
```text
Allow group <LabGroup> to use metrics in compartment <LabCompartment>
```

PostMetricData=METRIC_WRITE; SummarizeMetricsData=METRIC_INSPECT+METRIC_READ; ListMetrics=METRIC_INSPECT. scoped use metrics가 해당 요청을 포괄한다. 이 과제는 alarm/topic을 생성하지 않으므로 alarm/ons 정책은 후속82/86에서만 승인한다.

operation 근거: [공식 권한/작업 본문](https://docs.oracle.com/en-us/iaas/Content/Identity/policyreference/monitoringpolicyreference.htm). family grant는 minimum-permission 정책으로 주장하지 않는다. 실행 전에 관리자 승인된 identity/domain·실제 Compartment scope를 확인한다.

## 비용및주의사항

새 VM/agent/alarm/topic 없이 custom metric 한 stream에1 datapoint만 발행하고 positive/negative/복귀 조회 합계3회로 제한한다. 현재 Monitoring ingestion/query 단위·한도·계약을 확인하고120초 readiness는 작성한 대기 계획이지 SLA가 아니다. namespace/name/dimension을 늘려 stream 수를 키우지 않는다. 개별 datapoint 삭제를 시도하지 않고 publisher 종료와 본인 파일 정리로 추가 발행을 중단한다.

## 실습구성

### 워크숍 순서와 입력값

**진행 순서:** Region/realm/시간 준비 →1point 게시 →동일dimension query/Console →wrongdimension빈결과 →정상복구

**선택 이유:** publisher accepted와 실제 query 결과를 별도 증거로 판단한다.

| 입력 | 값 또는 정확한 출처 | 실행 위치 | 생성 전/후 | 이유 |
|---|---|---|---|---|
| COMPARTMENT_OCID/OCI_CLI_REGION | 승인실제scope/region | Cloud Shell | 전 | 발행조회동일scope |
| ingestionendpoint | 해당realm 실제telemetry-ingestion endpoint | Cloud Shell | 전 | commercial예자동전용금지 |
| namespace/name/dimension | roadmap_lab /LabHeartbeat /lab=roadmap | CLI/Console | 전 | 1stream 재사용 |


값이 다른 터미널에 자동 전파되지 않습니다. Console readback으로 비밀 없는 값만 인계하며, placeholder는 실제값으로 교체한 뒤 실행합니다. 새 리소스 생성 전 이름 충돌·소유자·최대 비용·종료 UTC를 확인하고 선택하지 않은 분기는 **NOT_SELECTED**로 기록합니다.


custom metric 발행부터 Console Metrics Explorer와 CLI query까지 실제 통신 경로를 만든다.

<a id="metric-publish"></a>


### 이번 실습의 생성·재사용·앱 경계

| 구분 | 이번 주제의 실제 범위 |
|---|---|
| 새로 생성 | custom stream에1 datapoint·본인payload |
| 재사용 | 승인Compartment/Region/Cloud Shell |
| 앱/작업부하 및 변경 | 앱 대신CLI publisher가workload;VM exporter/alarm/topic 없음 |
| 실행/반환 순서 | Bash Cloud Shell OCI CLI 인증과 Python3, 승인 scope/realm endpoint 준비 → same-stream accepted/query 실제값·scope/endpoint를81에 인계 |

새로 생성하지 않는 항목을선행완료로가장하지않는다.위표의실제OCID/주소와진행순서를첫mutation전에확인한다.

## 실습절차

- 발행 전 동일 Cloud Shell에서 `export METRIC_INGESTION_ENDPOINT='승인된 actual realm/Region telemetry-ingestion HTTPS endpoint'`를 설정한다. commercial realm 예는 `https://telemetry-ingestion.<actual-region>.oraclecloud.com`이며 다른 realm에 복사하지 않는다. 각 post 뒤 응답의 failed-metrics-count=0을 직접 읽고, 0이 아니거나 응답을 읽지 못했으면 다음 발행/조회 전에 중단한다.

### publisher·query의 입력/시간 gate
전용 빈 작업 디렉터리를 준비하고 metric.json이 이미 있으면 overwrite 전 내용을 확인한다. Python이 쓰는 `compartmentId/datapoints/dimensions/metadata/name/namespace`는 CLI 3.68.1 offline schema와 일치한다. ingestion endpoint는 commercial realm 예이며 생성 UI와 다른 Region에 발행하지 않는다.
query에는 `--start-time "$(date -u -d '15 minutes ago' +%Y-%m-%dT%H:%M:%SZ)" --end-time "$(date -u +%Y-%m-%dT%H:%M:%SZ)"`를 추가해 Console 최근15분과 동일하게 맞춘다. 같은 namespace의 dimension을 `lab="not-roadmap"`으로 바꾼 조회의 빈 결과와 `lab="roadmap"` 정상 query datapoint를 비교하고 정상 dimension으로 복구한다. namespace IAM 제한에 걸린 authorization denial을 빈 결과로 오해하지 않는다. 잘못된 metric을 추가 발행하지 않는다.

### 발행 전 actual endpoint discovery와 ingestion readiness
Oracle [Monitoring API reference](https://docs.oracle.com/en-us/iaas/api/#/en/monitoring/20180401/)의 Endpoints에서 **actual realm/Region Monitoring telemetry HTTPS endpoint**를 찾고 Region/realm을 관리자와 대조한다. 목록에서 실제 endpoint를 찾지 못하면 BLOCKED_ENDPOINT_UNKNOWN이며 suffix를 추측하지 않는다. [PostMetricData CLI 공식 본문](https://docs.oracle.com/en-us/iaas/tools/oci-cli/latest/oci_cli_docs/cmdref/monitoring/metric-data/post.html)은 해당 endpoint의 `telemetry`를 `telemetry-ingestion`으로 바꾸도록 명시한다. 정확한 authority의 service prefix만 바꾸고 Region/domain suffix는 그대로 보존한다.
```bash
export METRIC_QUERY_ENDPOINT='공식 Endpoints 목록의 actual telemetry HTTPS endpoint'
export METRIC_INGESTION_ENDPOINT=$(python3 - <<'PY'
import os,urllib.parse
u=urllib.parse.urlsplit(os.environ['METRIC_QUERY_ENDPOINT'])
assert u.scheme=='https' and u.hostname and u.hostname.startswith('telemetry.')
assert not u.username and not u.password and not u.port and not u.query and not u.fragment
print('https://telemetry-ingestion.'+u.hostname[len('telemetry.'):])
PY
) || exit 1
```
발행 전 payload의 실제 UTC timestamp와 승인 expiry를 기록한다. receipt 성공 뒤 **120초 foreground ingestion 대기**를 포함할 예산을 먼저 확보하고, 최초 positive 조회 전에만120초 기다린 후 START/END를 계산한다.120초는 작성한 bounded readiness 계획이지 Oracle ingestion SLA가 아니다. positive가 비면 추가 query로 예산을 소비하지 말고 ASYNC_PENDING으로 반환한다. positive 확인 후만 같은 고정 START/END로 negative/복귀를 수행한다.80 총3회,81 총4회는 Console+CLI 합계이며 Console chart 조회는 CLI 대응 조회의 대안이다. 무승인 재발행/추가조회는 없다.

### 실행 단계

1. Observability & Management → Monitoring → Metrics Explorer를 열고 Region·Compartment **실습 Compartment**를 선택한다. 기본 범위는 최근 **15분**. namespace roadmap_lab가 처음에는 없는 것은 아직 발행하지 않았기 때문이다.
2. 아래 명령으로 현재 UTC datapoint 1개를 발행한다. timestamp가 2시간 이전/10분 이후면 허용되지 않는다.[T1]
```bash
set -euo pipefail
# Bash/Cloud Shell. 실제 Compartment/Region export가 선행되어야 한다.
test ! -e metric.json || { printf 'existing lab payload: stop and review owner\n'; exit 1; }
python3 - <<'PY'
import os,json,datetime
now=datetime.datetime.now(datetime.timezone.utc).isoformat().replace('+00:00','Z')
a=[{'compartmentId':os.environ['COMPARTMENT_OCID'],'namespace':'roadmap_lab',
    'name':'LabHeartbeat','dimensions':{'lab':'roadmap'},
    'metadata':{'unit':'count'},'datapoints':[{'timestamp':now,'value':1,'count':1}]}]
open('metric.json','w').write(json.dumps(a))
PY
oci monitoring metric-data post --metric-data file://metric.json --batch-atomicity ATOMIC   --endpoint "${METRIC_INGESTION_ENDPOINT}" > metric-post-receipt.json
python3 - <<'PYRECEIPT'
import json
receipt=json.load(open('metric-post-receipt.json'))['data']
assert receipt['failed-metrics-count']==0 and not receipt['failed-metrics'], receipt
print('actual post receipt: failed metrics zero; query/history still required')
PYRECEIPT
sleep 120  # approved foreground ingestion readiness; not an OCI latency guarantee
START=$(date -u -d '15 minutes ago' +%Y-%m-%dT%H:%M:%SZ)
END=$(date -u +%Y-%m-%dT%H:%M:%SZ)
oci monitoring metric-data summarize-metrics-data --compartment-id "$COMPARTMENT_OCID"   --namespace roadmap_lab --query-text 'LabHeartbeat[1m]{lab="roadmap"}.max()' --start-time "$START" --end-time "$END" --resolution 1m
```
official actual realm endpoint와zero failed receipt를읽는다.최초positive1회→negative1회→복귀1회,총3회예산은Console/CLI합계다.빈positive는ASYNC_PENDING이며추가발행/조회로보완하지않는다.
3. 최초 positive 조회는위CLI 또는다음Console 중하나만선택한다.CLI에서이미positive를읽었으면다음Update Chart를추가실행하지않고negative/복귀두조회로진행한다. Metrics Explorer → Namespace **roadmap_lab**, Metric **LabHeartbeat**, Dimension **lab=roadmap**, Interval **1 minute**, Statistic **Max**를 선택한다. **Update Chart → Show Data Table**을 누르고 CLI와 같은 START/END 범위의 실제 datapoint timestamp/value를 읽는다. 최초positive1회+negative1회+복귀1회=기본3회이며 추가재조회는 별도관찰예산승인없이는하지않는다.
4. 발행 응답과 query timestamp/value, Console time zone을 기록한다. 다음 81에서 interval/statistic을 비교하기 위해 이 namespace/dimension을 유지한다.


### 진행·종료 판정
설정 readback·기능시험·원복·정리를 각각 실제 응답으로 기록한다. 기대값은 미실행이며 실패/만료 시 새 mutation을 중단하고 아래 topic별 종료 단계로 반환한다.

### Executable same-stream dimension control and restoration
같은 Cloud Shell에서 동일 window의 negative dimension과 복귀 query를 실행한다. 정상 query 응답을 먼저 확보하며 authorization error를 빈 배열로 해석하지 않는다.
```bash
# 동일 positive-query START/END를 재사용; 다른 shell이면 비밀 없는 실제 window를 인수한다.
oci monitoring metric-data summarize-metrics-data --compartment-id "$COMPARTMENT_OCID" --namespace roadmap_lab --query-text 'LabHeartbeat[1m]{lab="not-roadmap"}.max()'  --start-time "$START" --end-time "$END" --resolution 1m
oci monitoring metric-data summarize-metrics-data --compartment-id "$COMPARTMENT_OCID" --namespace roadmap_lab --query-text 'LabHeartbeat[1m]{lab="roadmap"}.max()' --start-time "$START" --end-time "$END" --resolution 1m
```
negative는 datapoint 없음, 정상은 실제 UTC/value를 포함한다. 기존 payload는 owner 확인 후만 덮어쓴다.


### 번호 순서로 완료하는 원복·정리

5. **Monitoring 종료 및 실패 반환.** same-window 정상dimension복귀query에서actualtimestamp/value를읽고publisher를종료한다.81/82가선택되면scope/stream/endpoints/expiry를인계하고본인payload는consumer반환뒤만삭제한다.개별datapoint/retention삭제는없다.

실제 정리/원복 response, exact resource 또는rule ID,완료UTC와남은owner/expiry를비밀없는개인실행기록에남긴다.위의guarded복구가이미완료된경우중복mutation하지않는다.

## 결과검증

**기대 패턴(미실행):** 발행 failed metrics=0, query aggregated-datapoints의 value=1. Console chart는 UTC와 로컬 시간 차이를 반영할 수 있다. 실제 결과는 실행 후 기록한다.

실행 기록(미기입): 실행 UTC __ / 실제 resource OCID __ / 요청·SQL 실제 응답 __ / 공식 조건 차이 __ / 정리 완료 UTC __.

## 문제해결

401/404는 인증·Region·ingestion endpoint·use metrics scope를 확인한다. post가 accepted인데 chart가 비면 namespace 대소문자, dimension filter, 시간 범위를 확인한다. UTC timestamp 대신 문자열 예시 날짜를 재사용하지 않는다.

## 리소스정리

metric publisher를 더 실행하지 않는다. custom metric datapoint는 개별 삭제 리소스가 아니므로 retention 경과로 소멸한다. local metric.json은 본인 파일만 삭제하고 82 alarm이 있으면 disable/delete한다.

### 소유자별 종료와 exact 대상 재조회
80 owner는 추가 publisher가 없음을 본인 shell에서 확인하고 own metric.json만 삭제·파일 부재를 읽는다. 삭제 가능한 stream resource를 찾거나 서비스 retention을 변경하지 않는다. 81/82로 실제 scope/stream/endpoint를 인계한 경우 their publisher 종료까지 비용 owner를 유지한다.

실제 정리 UTC __ / exact resource 또는 rule ID __ / 삭제 후 조회 응답 __ / 잔존 resource·만료·owner __. 승인 없는 잔존 비용은 다음 실습을 이유로 연장하지 않는다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 개념은 해당 생성·변경·조회 단계에서 입력 선택과 실제 결과 해석을 위해 사용하며, 공식 예시는 실제 실습 실행 결과가 아닙니다.

- [29] [Overview of Monitoring — Monitoring](https://docs.oracle.com/en-us/iaas/Content/Monitoring/Concepts/monitoringoverview.htm)
  - 확인 범위: Use the Oracle Cloud Infrastructure Monitoring.

### 실습 task 본문 근거

- [T1] [Tasks / publishingcustommetrics.htm](https://docs.oracle.com/en-us/iaas/Content/Monitoring/Tasks/publishingcustommetrics.htm) — 본문 절차·입력 필드·지원 조건을 읽어 실습에 적용.
- [T2] [Tasks / query-metric.htm](https://docs.oracle.com/en-us/iaas/Content/Monitoring/Tasks/query-metric.htm) — 본문 절차·입력 필드·지원 조건을 읽어 실습에 적용.
- [T3] [metric-data / post.html](https://docs.oracle.com/iaas/tools/oci-cli/latest/oci_cli_docs/cmdref/monitoring/metric-data/post.html) — 본문 절차·입력 필드·지원 조건을 읽어 실습에 적용.


- [T-ONS] [ons-topics / ons-subscriptions 권한](https://docs.oracle.com/en-us/iaas/Content/Identity/policyreference/notificationpolicyreference.htm) — ons-topics / ons-subscriptions 권한 본문을 실제 읽어 적용.
