> **기존 실습 초안 — 미검증**: 원래 93개 문서의 절차를 보존한 자료입니다. 새 튜토리얼의 품질 승인이나 OCI 실행 검증을 뜻하지 않습니다.

# Alarms

> 문서 작성 검토와 실행 검증은 별개입니다. OCI 실행은 **USER_EXECUTION_PENDING**이며 사용자 캡처는 별도 자료입니다. 미실행 자체는 문서 작성 승인 차단 사유가 아닙니다. 실제 결과 필드는 실행자가 채웁니다.

> 실행용 한국어 교육 문서. OCI 리소스 생성·명령 실행 결과는 아직 검증하지 않았으며 모든 기대 패턴은 미실행이다.

## 학습목표

- 86의 실제 수신 완료 topic과 초기 zero 조회를 인수하고 max>0, delay1minute, no repeat alarm1개를 만든다.
- bounded true3회/zero2회 발행으로 OK→FIRING→OK history 및 실제 alarm email을 대응시킨다. 데이터 부재나 CLI receipt를 상태전이로 대체하지 않는다.
- 발행·조회 실패 시 추가 발행을 멈추고 alarm disable/delete readback 후 topic을83까지 유지한다.

실제 OCI 요청·응답/SQL·정리 증거는 실행자가 기록한다. 아래 기대값은 실측값이 아니다.

## 대응 Roadmap 항목

- 항목명: **Alarms**
- Roadmap 경로: **Monitoring → Alarms**
- Node ID: `sequence-9-2`
- 상위 항목: **Monitoring** (`sequence-9-0`)
- 문서 범위: 이 항목의 개념, 실제 설정 절차와 학습자 실행 검증.
- [전체 항목 목록](../../../Docs/README.md)

## 사전준비

**시작 조건:** 80 metric readiness 및86 confirmed topic 실제 marker mail 수신 완료

- [Monitoring](00_Monitoring.md#metric-publish) — use metrics 및 custom metric query 성공.
- [Notifications](06_Notifications.md#topic-ready) — topic 생성·email subscription ACTIVE·실제 test message 수신까지 완료. topic 유지.

### 준비 완료 출력과 인수 확인

| 인수 항목 | 실제 출력·출처 | 다음 작업의 사용 위치 |
|---|---|---|
| 선행 완료 상태 | 80 metric readiness 및86 confirmed topic 실제 marker mail 수신 완료 | 첫 변경 전 재조회 |
| 비밀 없는 실행 입력 | actual TOPIC_OCID/subscription ACTIVE 수신기록, metric stream/endpoint, absolute session expiry | 아래 numbered Console/명령 단계 |
| 반환/후속 인계 | alarm disable/delete 반환;86 topic은83 consumer까지 유지 | 종료 UTC 이전 exact owner 반환 |

실제 Compartment/Region은 승인 Console에서 읽고 동일 identity/Region을 고정한다. Cloud Shell에서 `oci --version`과 `command -v python3`을 확인한다. 이 작업은 dig/HTTP 앱/VCN VM을 필요로 하지 않는다. placeholder·명칭만 인수한 상태는 준비 완료가 아니다. 비밀번호·키·확인 token은 출력표에 적지 않는다.

## 권한

전용 Compartment에 한정한 정책 예:
```text
Allow group <LabGroup> to use metrics in compartment <LabCompartment>
Allow group <LabGroup> to manage alarms in compartment <LabCompartment>
Allow group <LabGroup> to use ons-topics in compartment <ActualTopicCompartment>
```
`use metrics`는 custom metric 발행·조회 용도이며 alarm create에 metric 조회가 필요하다. Notifications publish/subscribe 및 email confirmation은 topic 정책 외에 실제 수신자의 승인 절차도 필요하다.

CreateAlarm=ALARM_CREATE+METRIC_READ; UpdateAlarm(Disable)=ALARM_UPDATE+METRIC_READ; GetAlarm=ALARM_READ+METRIC_READ; GetAlarmHistory=ALARM_READ; DeleteAlarm=ALARM_DELETE. metric Compartment가 alarm Compartment와 다르면 METRIC_READ를 실제 metric scope에 부여한다. destination topic 사용은 ONS_TOPIC_PUBLISH/목록·읽기를 관리자와 확인하고 topic 생성 권한을 본 과제에 무조건 부여하지 않는다.

operation 근거: [공식 권한/작업 본문](https://docs.oracle.com/en-us/iaas/Content/Identity/policyreference/monitoringpolicyreference.htm). family grant는 minimum-permission 정책으로 주장하지 않는다. 실행 전에 관리자 승인된 identity/domain·실제 Compartment scope를 확인한다.

## 비용및주의사항

새 alarm1개만 만들고86의 기존 confirmed topic을 destination으로 사용한다. 초기0 한 점+true3점+복구0 두 점, 총6점 및 반복 알림 off를 유지한다. 현재 Monitoring alarm/ingestion/query와 Notifications 발행·메일 전달 계약을 승인한다. 단일10분 absolute lease에서 복구/disable/delete 시간을 먼저 예약하고 미관측을 추가 publisher로 보완하지 않는다. 새 topic/VM를 만들지 않으며 미배달을 배달 성공으로 계산하지 않는다.

## 실습구성

### 워크숍 순서와 입력값

**진행 순서:** 86 actual수신 준비 →initialzero →alarm생성/OK →3true →FIRING/email →2zero →OK →disable/delete

**선택 이유:** 데이터 부재를0으로 취급하지 않고 발행·평가·배달을 따로 확인한다.

| 입력 | 값 또는 정확한 출처 | 실행 위치 | 생성 전/후 | 이유 |
|---|---|---|---|---|
| TOPIC_OCID | 86 ACTIVE/actualmarker수신topic | Console | 전 | 배달경로준비 |
| scope/query | 80동일scope /LabHeartbeat[1m]{lab="roadmap"}.max() > 0 | Console/Cloud Shell | 전 | 동일stream 평가 |
| true/zero/budget | 3회1 / 초기1회0+복구2회0 /10분 | Cloud Shell | 전 | 최대6points |


값이 다른 터미널에 자동 전파되지 않습니다. Console readback으로 비밀 없는 값만 인계하며, placeholder는 실제값으로 교체한 뒤 실행합니다. 새 리소스 생성 전 이름 충돌·소유자·최대 비용·종료 UTC를 확인하고 선택하지 않은 분기는 **NOT_SELECTED**로 기록합니다.


실제 custom metric과 확인 완료 Notifications topic을 연결해 FIRING/OK 전환을 검증한다.


### 이번 실습의 생성·재사용·앱 경계

| 구분 | 이번 주제의 실제 범위 |
|---|---|
| 새로 생성 | alarm1개와동일stream6points |
| 재사용 | 80/81 metric stream 및86 confirmedtopic |
| 앱/작업부하 및 변경 | CLI publisher→alarm evaluation→email;새앱/VM/topic 없음 |
| 실행/반환 순서 | 80 metric readiness 및86 confirmed topic 실제 marker mail 수신 완료 → alarm disable/delete 반환;86 topic은83 consumer까지 유지 |

새로 생성하지 않는 항목을선행완료로가장하지않는다.위표의실제OCID/주소와진행순서를첫mutation전에확인한다.

## 실습절차

- 발행 전 동일 Cloud Shell에서 `export METRIC_INGESTION_ENDPOINT='승인된 actual realm/Region telemetry-ingestion HTTPS endpoint'`를 설정한다. commercial realm 예는 `https://telemetry-ingestion.<actual-region>.oraclecloud.com`이며 다른 realm에 복사하지 않는다. 각 post 뒤 응답의 failed-metrics-count=0을 직접 읽고, 0이 아니거나 응답을 읽지 못했으면 다음 발행/조회 전에 중단한다.

### 단일 absolute lease (최초 alarm mutation 전)
승인 expiry와 현재시각+600초 중 빠른 것을 **단 하나의 SESSION_END_EPOCH**로 고정한다. 새 터미널에서도 이 exact 값과 expiry를 인수하며 재계산해10분을 연장하지 않는다. 종료300초 전에는 true 발행/새 FIRING wait를 중단한다. 마지막300초는 zero2회(120초), OK/history 확인(최대120초), disable/delete readback(60초)에 예약한다. 시간이 부족하면 true 시험을 시작하지 않으며, 중간 실패에는 publisher 중단→본인 alarm 즉시 Disable/readback→Delete/readback을 수행하고 OK 미관측을 성공으로 기록하지 않는다.
```bash
export EXPIRY_UTC='<approved-absolute-expiry-UTC>'
export SESSION_START_EPOCH=$(date -u +%s)
export SESSION_END_EPOCH=$(python3 - <<'PY'
import os,time,datetime
expiry=datetime.datetime.fromisoformat(os.environ['EXPIRY_UTC'].replace('Z','+00:00'))
assert expiry.tzinfo is not None
print(int(min(expiry.timestamp(),time.time()+600)))
PY
) || exit 1
# Preserve/export this exact nonsecret deadline to all following publisher shells.
```

### alarm 상태와 메시지 복구 gate
86 topic/subscription ACTIVE와 실제 test 수신이 먼저다. 최근 81의 nonzero point가 섞이면 과제 시작 전에 그 minute bucket이 지난 뒤 zero1점을 발행하고 alarm OK를 먼저 확인한다. Console에서 Enabled=true, no repeat와 실제 query/destination topic OCID를 재조회한다.
3회 true 공급은 1분 delay 이상의 데이터지만 서버 평가 지연까지 보장하지 않는다. 전체 alarm 세션의 단일10분 deadline 안에서 FIRING/history·email과 zero2회→OK/history·이메일·disable/delete를 모두 처리한다. FIRING만10분 기다리지 않는다. 복구 통지가 UI에 비활성인 경우 history OK와 통지 설정 차이를 남긴다. FIRING에서 계속 nonzero 발행하지 않는다. disable/delete를 Console에서 재조회하며 topic은 83 뒤 정리한다.

### 실행 단계

1. 먼저 86에서 확인한 topic OCID를 준비한다. PENDING email subscription은 수신 완료가 아니다.
1a. Cloud Shell에서 기존nonzero minute bucket이지난뒤초기0을1회게시한다. alarm-metric.json은본인전용작업파일만사용한다.
```bash
set -euo pipefail
test ! -e alarm-metric.json || { printf 'existing alarm payload: stop and review owner\n'; exit 1; }
python3 - <<'PY'
import os,json,datetime
p=[{'compartmentId':os.environ['COMPARTMENT_OCID'],'namespace':'roadmap_lab','name':'LabHeartbeat','dimensions':{'lab':'roadmap'},'metadata':{'unit':'count'},'datapoints':[{'timestamp':datetime.datetime.now(datetime.timezone.utc).isoformat(),'value':0,'count':1}]}]
open('alarm-metric.json','w').write(json.dumps(p))
PY
oci monitoring metric-data post --metric-data file://alarm-metric.json --batch-atomicity ATOMIC --endpoint "${METRIC_INGESTION_ENDPOINT}" > metric-post-receipt.json
python3 - <<'PYRECEIPT'
import json
receipt=json.load(open('metric-post-receipt.json'))['data']
assert receipt['failed-metrics-count']==0 and not receipt['failed-metrics'], receipt
print('actual post receipt: failed metrics zero; query/history still required')
PYRECEIPT
```
failed-metrics-count=0 확인 뒤80의 같은 stream max query를 새 실제 최근15분 window로1회실행해 최신완료minute value0를확인한다. 이전nonzero bucket만보이면새발행없이minute완료/수집을bounded관찰하고초기OK없이true시험을시작하지않는다. 다른realm은실제ingestionendpoint를사용한다.
2. Monitoring → Alarm definitions → Create alarm. Name **roadmap-heartbeat-alarm**, Compartment/Metric compartment **실습 Compartment**, Namespace **roadmap_lab**, Metric **LabHeartbeat**, Dimension **lab=roadmap**, Interval **1 minute**, Statistic **Max**. Trigger rule **greater than 0**, Trigger delay **1 minute**, Severity **Warning**, Destination service **Notifications**, Topic **86의 확인된 topic**, Repeat notification **off**, Enabled **on**. Advanced query는 `LabHeartbeat[1m]{lab="roadmap"}.max() > 0`이다.[T1]
2a. Alarm detail/history에서 초기 **OK**와 실제query/topic/Enabled/no-repeat를 읽고나서진행한다. zero가수집되지않거나OK가확인되지않으면생성을기능성공으로표시하지않는다.
3. 아래 publisher를 3회만 실행해 1분 이상 true metric을 공급한다. CLI exit status만으로 alarm transition을 주장하지 않는다.
```bash
set -euo pipefail
test "$((SESSION_END_EPOCH-$(date -u +%s)))" -ge 480 || {
  printf 'ABORT: insufficient budget for all true points plus recovery; disable/delete now\n' >&2; exit 1;
}
for n in 1 2 3; do
  test "$((SESSION_END_EPOCH-$(date -u +%s)))" -ge 360 || {
    printf 'ABORT: true phase would consume reserved recovery/cleanup budget\n' >&2; exit 1;
  }
  python3 - <<'PY'
import os,json,datetime
p=[{'compartmentId':os.environ['COMPARTMENT_OCID'],'namespace':'roadmap_lab','name':'LabHeartbeat',
'dimensions':{'lab':'roadmap'},'metadata':{'unit':'count'},'datapoints':[{'timestamp':datetime.datetime.now(datetime.timezone.utc).isoformat(), 'value':1,'count':1}]}]
open('alarm-metric.json','w').write(json.dumps(p))
PY
  oci monitoring metric-data post --metric-data file://alarm-metric.json --batch-atomicity ATOMIC    --endpoint "${METRIC_INGESTION_ENDPOINT}" > metric-post-receipt.json
  python3 - <<'PYRECEIPT'
import json
receipt=json.load(open('metric-post-receipt.json'))['data']
assert receipt['failed-metrics-count']==0 and not receipt['failed-metrics'], receipt
print('actual post receipt: failed metrics zero; query/history still required')
PYRECEIPT
  sleep 60
done
```
4. SESSION_START_EPOCH+300 또는 SESSION_END_EPOCH−300 중 빠른 시각까지만 Alarm detail/history에서 실제 FIRING timestamp를 확인하고 이메일의 alarm OCID·query·metric stream을 대조한다. 지연이 지나도 없으면 실패로 기록하고 반복 발행을 중단한다.
5. 다음과 같이 현재 timestamp의 **0**을 1분 간격 2회 발행한다. metric 최대값 bucket에서 이전 1이 빠진 뒤 OK transition을 기록한다. absence가 0과 같지 않으므로 발행 중단만으로 OK라고 단정하지 않는다.
```bash
set -euo pipefail
for n in 1 2; do
  test "$((SESSION_END_EPOCH-$(date -u +%s)))" -ge 180 || {
    printf 'ABORT: disable/delete alarm now; do not overrun lease\n' >&2; exit 1;
  }
  python3 - <<'PY'
import json,datetime
p=json.load(open('alarm-metric.json'))
p[0]['datapoints']=[{'timestamp':datetime.datetime.now(datetime.timezone.utc).isoformat(),
                     'value':0,'count':1}]
open('alarm-metric.json','w').write(json.dumps(p))
PY
  oci monitoring metric-data post --metric-data file://alarm-metric.json --batch-atomicity ATOMIC --endpoint "${METRIC_INGESTION_ENDPOINT}" > metric-post-receipt.json
  python3 - <<'PYRECEIPT'
import json
receipt=json.load(open('metric-post-receipt.json'))['data']
assert receipt['failed-metrics-count']==0 and not receipt['failed-metrics'], receipt
print('actual post receipt: failed metrics zero; query/history still required')
PYRECEIPT
  sleep 60
done
```
6. alarm을 Disable하고 history/e-mail 결과를 기록한다.


### 진행·종료 판정
설정 readback·기능시험·원복·정리를 각각 실제 응답으로 기록한다. 기대값은 미실행이며 실패/만료 시 새 mutation을 중단하고 아래 topic별 종료 단계로 반환한다.

### 번호 순서로 완료하는 원복·정리

7. **Alarms 종료 및 실패 반환.** deadline 안에OK history/실제통지기록을확인하되미관측이면성공으로쓰지않는다.추가publisher중단→actualalarm Edit/Enable unchecked/Save→enabled=false readback→Delete/목록부재→본인payload삭제를수행한다.86topic은83반환까지남긴다.

실제 정리/원복 response, exact resource 또는rule ID,완료UTC와남은owner/expiry를비밀없는개인실행기록에남긴다.위의guarded복구가이미완료된경우중복mutation하지않는다.

## 결과검증

**기대 패턴(미실행):** 실제 alarm history FIRING→OK, 확인된 topic을 통해 실제 email 수신. metric post accepted/chart 변경만으로 notification 완료를 표시하지 않는다.

실행 기록(미기입): 실행 UTC __ / 실제 resource OCID __ / 요청·SQL 실제 응답 __ / 공식 조건 차이 __ / 정리 완료 UTC __.

## 문제해결

alarm query가 비면 Region/metric Compartment 확인. FIRING인데 email 없으면 subscription ACTIVE, topic destination, spam folder·Notifications metrics를 확인한다. trigger delay는 aggregation interval과 다르다.

## 리소스정리

alarm disable → alarm 삭제 → local alarm-metric.json 삭제. topic/subscription은 83 완료 후 86에 따라 정리한다. metric publisher는 종료한다.

### 소유자별 종료와 exact 대상 재조회
82 owner: Alarm Definitions actual alarm OCID Edit/Enable this alarm unchecked/Save → enabled=false readback → Delete/confirm → same Region/Compartment 목록 부재. 본인 alarm-metric.json 부재를 확인한다. topic 삭제는83 반환 후86 owner만 수행한다.

실제 정리 UTC __ / exact resource 또는 rule ID __ / 삭제 후 조회 응답 __ / 잔존 resource·만료·owner __. 승인 없는 잔존 비용은 다음 실습을 이유로 연장하지 않는다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 개념은 해당 생성·변경·조회 단계에서 입력 선택과 실제 결과 해석을 위해 사용하며, 공식 예시는 실제 실습 실행 결과가 아닙니다.

- [29] [Overview of Monitoring — Alarms](https://docs.oracle.com/en-us/iaas/Content/Monitoring/Concepts/monitoringoverview.htm)
  - 확인 범위: Alarms Feature Overview.

### 실습 task 본문 근거

- [T1] [Tasks / create-alarm-basic.htm](https://docs.oracle.com/en-us/iaas/Content/Monitoring/Tasks/create-alarm-basic.htm) — 본문 절차·입력 필드·지원 조건을 읽어 실습에 적용.
- [T2] [Tasks / create-edit-alarm-query-trigger-delay.htm](https://docs.oracle.com/en-us/iaas/Content/Monitoring/Tasks/create-edit-alarm-query-trigger-delay.htm) — 본문 절차·입력 필드·지원 조건을 읽어 실습에 적용.
- [T3] [Tasks / confirm-subscription.htm](https://docs.oracle.com/en-us/iaas/Content/Notification/Tasks/confirm-subscription.htm) — 본문 절차·입력 필드·지원 조건을 읽어 실습에 적용.


- [T-ONS] [ons-topics / ons-subscriptions 권한](https://docs.oracle.com/en-us/iaas/Content/Identity/policyreference/notificationpolicyreference.htm) — ons-topics / ons-subscriptions 권한 본문을 실제 읽어 적용.
