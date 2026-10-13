# Metrics

> 문서 작성 검토와 실행 검증은 별개입니다. OCI 실행은 **USER_EXECUTION_PENDING**이며 사용자 캡처는 별도 자료입니다. 미실행 자체는 문서 작성 승인 차단 사유가 아닙니다. 실제 결과 필드는 실행자가 채웁니다.

> 실행용 한국어 교육 문서. OCI 리소스 생성·명령 실행 결과는 아직 검증하지 않았으며 모든 기대 패턴은 미실행이다.

## 학습목표

- 80의 같은 stream에 현재·10초 전 value4/2 두 점만 게시하고 실제 UTC minute bucket을 기록한다.
- 동일 START/END·resolution1m의 max/sum을 timestamp별로 비교하고 이전 point/경계가 합에 섞이는지를 실제 조회값으로 확인한다.
- wrong dimension 조회 후 정상 filter를 복구하여 데이터가 유지됨을 확인하고 publisher를 종료해82에 stream을 인계한다.

실제 OCI 요청·응답/SQL·정리 증거는 실행자가 기록한다. 아래 기대값은 실측값이 아니다.

## 대응 Roadmap 항목

- 항목명: **Metrics**
- Roadmap 경로: **Monitoring → Metrics**
- Node ID: `sequence-9-1`
- 상위 항목: **Monitoring** (`sequence-9-0`)
- 문서 범위: 이 항목의 개념, 실제 설정 절차와 학습자 실행 검증.
- [전체 항목 목록](../README.md)

## 사전준비

**시작 조건:** 80 failed-metrics-count0 및actual timestamp/value1 positive 조회 완료

- [Monitoring](00_Monitoring.md#metric-publish) — metric 발행 및 CLI/Console value=1 조회 완료.

### 준비 완료 출력과 인수 확인

| 인수 항목 | 실제 출력·출처 | 다음 작업의 사용 위치 |
|---|---|---|
| 선행 완료 상태 | 80 failed-metrics-count0 및actual timestamp/value1 positive 조회 완료 | 첫 변경 전 재조회 |
| 비밀 없는 실행 입력 | same compartment/Region/endpoint,roadmap_lab/LabHeartbeat/lab=roadmap,80 본인payload 파일·발행UTC/expiry | 아래 numbered Console/명령 단계 |
| 반환/후속 인계 | publisher 종료·stream/실제bucket 기록을82에 인계;zero는82 담당 | 종료 UTC 이전 exact owner 반환 |

실제 Compartment/Region은 승인 Console에서 읽고 동일 identity/Region을 고정한다. Cloud Shell에서 `oci --version`과 `command -v python3`을 확인한다. 이 작업은 dig/HTTP 앱/VCN VM을 필요로 하지 않는다. placeholder·명칭만 인수한 상태는 준비 완료가 아니다. 비밀번호·키·확인 token은 출력표에 적지 않는다.

## 권한

전용 Compartment에 한정한 정책 예:
```text
Allow group <LabGroup> to use metrics in compartment <LabCompartment>
```

PostMetricData=METRIC_WRITE; max/sum/negative-control SummarizeMetricsData=METRIC_INSPECT+METRIC_READ. metrics 정책을 namespace roadmap_lab로 제한할 수 있다. 다른 namespace 조회 negative control이 권한 제한에 걸리면 빈 결과와 authorization denial을 구별하고 같은 namespace의 dimension control로 대체한다.

operation 근거: [공식 권한/작업 본문](https://docs.oracle.com/en-us/iaas/Content/Identity/policyreference/monitoringpolicyreference.htm). family grant는 minimum-permission 정책으로 주장하지 않는다. 실행 전에 관리자 승인된 identity/domain·실제 Compartment scope를 확인한다.

## 비용및주의사항

80의 같은 stream에2 datapoints만 추가한다. max/sum/negative/복귀 조회 총4회는 Console와 CLI 합산 예산이며 각 화면에서 같은 query를 중복 실행하지 않는다. 현재 Monitoring 발행·조회 조건을 확인한다. timestamp가 다른 bucket에 들어가거나80 point가 섞인 결과를 추가 발행으로 맞추지 않는다. VM exporter·새 namespace·추가 metric stream은 생성하지 않는다.

## 실습구성

### 워크숍 순서와 입력값

**진행 순서:** 80stream 인수 →2point 게시 →같은timewindow max/sum →dimensioncontrol/복구

**선택 이유:** minute bucket 경계를 반영하고 이전point가 합에 섞임을 실제timestamp로 판단한다.

| 입력 | 값 또는 정확한 출처 | 실행 위치 | 생성 전/후 | 이유 |
|---|---|---|---|---|
| scope/stream | 80동일Compartment·Region·name·dimension | Cloud Shell | 전 | 동일stream |
| timestamps/values | 실제UTC10초전·현재 /2·4 | payloadproducer | 후 | bucket비교 |


값이 다른 터미널에 자동 전파되지 않습니다. Console readback으로 비밀 없는 값만 인계하며, placeholder는 실제값으로 교체한 뒤 실행합니다. 새 리소스 생성 전 이름 충돌·소유자·최대 비용·종료 UTC를 확인하고 선택하지 않은 분기는 **NOT_SELECTED**로 기록합니다.


sum/max 차이를 합성한 실제 custom datapoint로 관찰한다. 사용자가 발행할 데이터이지 실제 OCI 실행 결과로 꾸민 데이터가 아니다.


### 이번 실습의 생성·재사용·앱 경계

| 구분 | 이번 주제의 실제 범위 |
|---|---|
| 새로 생성 | 동일stream에2 datapoints |
| 재사용 | 80stream/endpoints/Cloud Shell |
| 앱/작업부하 및 변경 | CLI publisher만;max/sum/query filter 비교/복구 |
| 실행/반환 순서 | 80 failed-metrics-count0 및actual timestamp/value1 positive 조회 완료 → publisher 종료·stream/실제bucket 기록을82에 인계;zero는82 담당 |

새로 생성하지 않는 항목을선행완료로가장하지않는다.위표의실제OCID/주소와진행순서를첫mutation전에확인한다.

## 실습절차

- 발행 전 동일 Cloud Shell에서 `export METRIC_INGESTION_ENDPOINT='승인된 actual realm/Region telemetry-ingestion HTTPS endpoint'`를 설정한다. commercial realm 예는 `https://telemetry-ingestion.<actual-region>.oraclecloud.com`이며 다른 realm에 복사하지 않는다. 각 post 뒤 응답의 failed-metrics-count=0을 직접 읽고, 0이 아니거나 응답을 읽지 못했으면 다음 발행/조회 전에 중단한다.

### minute bucket와 복구를 구분하기
80과 같은 전용 작업 디렉터리·Compartment·Region에서 publisher를 실행하고 `failed-metrics-count=0`을 확인한다. 같은 bucket 합산을 반드시 보려면 UTC minute boundary 직후 15초 이후에 게시해 10초 전·현재가 같은 minute인지를 timestamp로 확인한다. 그렇지 않으면 다른 bucket도 정상이며 sum6으로 고정 판정하지 않는다.
max/sum query는 양쪽 모두 동일 start/end 최근15분·resolution1m을 명시하고 timestamp별로 비교한다. dimension을 `lab="not-roadmap"`으로 바꾼 query가 비는 negative control→`lab="roadmap"` 복구 query를 수행한다. 입력 필터 오류를 데이터 유실로 주장하지 않는다. zero 발행은 82 owner에게 인계한다.

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

1. 현재 namespace **roadmap_lab**, metric **LabHeartbeat**, dimension **lab=roadmap**를 유지한다. 두 points 값 **2,4**를 현재 UTC와 10초 전 timestamp로 만들어 발행한다. 최근 1분에 이전 80 값=1이 섞이면 sum은 달라질 수 있어 결과를 고정 숫자로 단정하지 않는다.
```bash
set -euo pipefail
python3 - <<'PY'
import os,json,datetime
now=datetime.datetime.now(datetime.timezone.utc)
p=[{'timestamp':(now-datetime.timedelta(seconds=s)).isoformat().replace('+00:00','Z'),
    'value':v,'count':1} for s,v in [(10,2),(0,4)]]
a=[{'compartmentId':os.environ['COMPARTMENT_OCID'],'namespace':'roadmap_lab',
    'name':'LabHeartbeat','dimensions':{'lab':'roadmap'},'metadata':{'unit':'count'},'datapoints':p}]
open('metric.json','w').write(json.dumps(a))
PY
oci monitoring metric-data post --metric-data file://metric.json --batch-atomicity ATOMIC  --endpoint "${METRIC_INGESTION_ENDPOINT}" > metric-post-receipt.json
python3 - <<'PYRECEIPT'
import json
receipt=json.load(open('metric-post-receipt.json'))['data']
assert receipt['failed-metrics-count']==0 and not receipt['failed-metrics'], receipt
print('actual post receipt: failed metrics zero; query/history still required')
PYRECEIPT
sleep 120  # approved foreground ingestion readiness; not an OCI latency guarantee
START=$(date -u -d '15 minutes ago' +%Y-%m-%dT%H:%M:%SZ)
END=$(date -u +%Y-%m-%dT%H:%M:%SZ)
oci monitoring metric-data summarize-metrics-data --compartment-id "$COMPARTMENT_OCID"  --namespace roadmap_lab --query-text 'LabHeartbeat[1m]{lab="roadmap"}.max()' --start-time "$START" --end-time "$END" --resolution 1m
oci monitoring metric-data summarize-metrics-data --compartment-id "$COMPARTMENT_OCID"  --namespace roadmap_lab --query-text 'LabHeartbeat[1m]{lab="roadmap"}.sum()' --start-time "$START" --end-time "$END" --resolution 1m
```
2. Console은CLI max/sum 두조회에대한**대안**이다.둘다실행하여4회조회예산을초과하지않는다.Console을선택하면Metrics Explorer Advanced mode의같은query/START/END와**Update Chart→Show Data Table**로timestamp/value를읽고negative/복귀에남은2회를사용한다. max는 해당 bucket의 가장 큰 값, sum은 bucket에 포함된 points의 합이다.[T1]
3. 선택한CLI 또는Console의max/sum 결과에서같은bucket timestamp를짝지어기록한다. [1m]은 aggregation interval이며 query resolution/시간 범위와 같다는 뜻은 아니다.[T2]
4. 82 alarm에서는 max>0를 사용하므로 정상 상태 재설정을 위해 마지막에 value 0을 게시할 수 있다. 82의 절차가 이 발행을 처리한다.


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

5. **Metrics 종료 및 실패 반환.** 동일START/END의normalfilter복귀를확인하고publisher를중단한다.82에same-stream/endpoint를인계하며zero는82절차로만발행한다.본인payload파일보존계약을확인하고반환뒤삭제한다.

실제 정리/원복 response, exact resource 또는rule ID,완료UTC와남은owner/expiry를비밀없는개인실행기록에남긴다.위의guarded복구가이미완료된경우중복mutation하지않는다.

## 결과검증

**기대 패턴(미실행):** 두 points가 같은 aggregation bucket에 들어가면 max≥4, sum≥6. minute 경계를 넘었다면 서로 다른 bucket이므로 각 timestamp별 값을 검토한다. 이전 값이 섞일 수 있다는 이유를 설명한다.

실행 기록(미기입): 실행 UTC __ / 실제 resource OCID __ / 요청·SQL 실제 응답 __ / 공식 조건 차이 __ / 정리 완료 UTC __.

## 문제해결

두 query의 결과가 동일하면 points가 다른 minute bucket인지 확인한다. display timezone과 UTC를 혼동하지 않는다. 서비스 default namespace에 임의 custom metric을 발행하지 않는다.

## 리소스정리

publisher 중단 및 본인이 만든 local metric.json 삭제. custom metric stream은 별도 Delete API 없이 retention을 따른다. 82 진행 시 Compartment/namespace/dimension 그대로 유지.

### 소유자별 종료와 exact 대상 재조회
81 owner는 게시 loop가 없음을 확인하고 own payload 파일만 정리한다. 82가 같은 파일을 인수했으면82 반환 후만 제거한다. metric datapoint를 개별 삭제했다고 기록하지 않는다.

실제 정리 UTC __ / exact resource 또는 rule ID __ / 삭제 후 조회 응답 __ / 잔존 resource·만료·owner __. 승인 없는 잔존 비용은 다음 실습을 이유로 연장하지 않는다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 개념은 해당 생성·변경·조회 단계에서 입력 선택과 실제 결과 해석을 위해 사용하며, 공식 예시는 실제 실습 실행 결과가 아닙니다.

- [29] [Overview of Monitoring — Metrics](https://docs.oracle.com/en-us/iaas/Content/Monitoring/Concepts/monitoringoverview.htm)
  - 확인 범위: Metrics Feature Overview.

### 실습 task 본문 근거

- [T1] [Tasks / query-metric-statistic.htm](https://docs.oracle.com/en-us/iaas/Content/Monitoring/Tasks/query-metric-statistic.htm) — 본문 절차·입력 필드·지원 조건을 읽어 실습에 적용.
- [T2] [Tasks / query-metric-resolution.htm](https://docs.oracle.com/en-us/iaas/Content/Monitoring/Tasks/query-metric-resolution.htm) — 본문 절차·입력 필드·지원 조건을 읽어 실습에 적용.


- [T-ONS] [ons-topics / ons-subscriptions 권한](https://docs.oracle.com/en-us/iaas/Content/Identity/policyreference/notificationpolicyreference.htm) — ons-topics / ons-subscriptions 권한 본문을 실제 읽어 적용.
