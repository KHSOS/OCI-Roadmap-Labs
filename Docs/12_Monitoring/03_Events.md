# Events

> 문서 작성 검토와 실행 검증은 별개입니다. OCI 실행은 **USER_EXECUTION_PENDING**이며 사용자 캡처는 별도 자료입니다. 미실행 자체는 문서 작성 승인 차단 사유가 아닙니다. 실제 결과 필드는 실행자가 채웁니다.

> 실행용 한국어 교육 문서. OCI 리소스 생성·명령 실행 결과는 아직 검증하지 않았으며 모든 기대 패턴은 미실행이다.

## 학습목표

- 86 topic을 기존 destination으로 선택한 Bucket-Create/Compartment filter rule1개를 저장하고 preview condition/action을 재조회한다.
- 예시 event positive/negative filter 검사를 실제 event와 구별하고 고유한 private Standard empty bucket1개를 생성하여 실제 createbucket email의 resourceId/time을 bucket 조회와 대조한다.
- rule disable/delete 및 exact bucket 삭제 readback을 수행하고 실제 작업 UTC window를85에 인계한다. replay·추가 bucket을 자동 생성하지 않는다.

실제 OCI 요청·응답/SQL·정리 증거는 실행자가 기록한다. 아래 기대값은 실측값이 아니다.

## 대응 Roadmap 항목

- 항목명: **Events**
- Roadmap 경로: **Monitoring → Events**
- Node ID: `sequence-9-3`
- 상위 항목: **Monitoring** (`sequence-9-0`)
- 문서 범위: 이 항목의 개념, 실제 설정 절차와 학습자 실행 검증.
- [전체 항목 목록](../README.md)

## 사전준비

**시작 조건:** 86 topic/subscription ACTIVE 및 실제test 수신 완료

- [Notifications](06_Notifications.md#topic-ready) — topic/email ACTIVE 및 test message 실제 수신 완료. topic 유지.

### 준비 완료 출력과 인수 확인

| 인수 항목 | 실제 출력·출처 | 다음 작업의 사용 위치 |
|---|---|---|
| 선행 완료 상태 | 86 topic/subscription ACTIVE 및 실제test 수신 완료 | 첫 변경 전 재조회 |
| 비밀 없는 실행 입력 | actual TOPIC_OCID/scope/Region, namespace, 고유EVENT_BUCKET,실제operation UTC/expiry | 아래 numbered Console/명령 단계 |
| 반환/후속 인계 | rule/bucket 삭제 readback 및 OP_START/END_UTC를85에 인계 | 종료 UTC 이전 exact owner 반환 |

실제 Compartment/Region은 승인 Console에서 읽고 동일 identity/Region을 고정한다. Cloud Shell에서 `oci --version`과 `command -v python3`을 확인한다. 이 작업은 dig/HTTP 앱/VCN VM을 필요로 하지 않는다. placeholder·명칭만 인수한 상태는 준비 완료가 아니다. 비밀번호·키·확인 token은 출력표에 적지 않는다.

## 권한

관리자 승인 정책: `Allow group <LabGroup> to manage cloudevents-rules in compartment <LabCompartment>`, `Allow group <LabGroup> to use ons-topics in compartment <LabCompartment>`, `Allow group <LabGroup> to manage buckets in compartment <LabCompartment>`. 같은 tenancy에서 이 실습은 rule 관리자에게 use ons-topics가 필요하다. 다른 tenancy로의 발행은 이 실습에 포함하지 않으며 cross-tenancy endorse/admit 정책을 무조건 추가하지 않는다. 객체 event emission 옵션은 bucket create/delete event와 다르다.

rule 생성/수정/disable/delete 권한은 cloudevents-rules scoped manage를 사용한다. Notifications action에는 실제 topic scope의 ONS_TOPIC_PUBLISH/use ons-topics가 필요하며 topic CREATE는86 소유다. Create/Get/DeleteBucket에는 BUCKET_CREATE/READ/DELETE, namespace get에는 OBJECTSTORAGE_NAMESPACE_READ를 별도 검토한다. Events 공식 task의 단수 ons-topic 예시를 현재 Notifications 표의 ons-topics와 대조했고 cross-tenancy endorse/admit은 추가하지 않는다.

operation 근거: [공식 권한/작업 본문](https://docs.oracle.com/en-us/iaas/Content/Events/Concepts/eventspolicy.htm). family grant는 minimum-permission 정책으로 주장하지 않는다. 실행 전에 관리자 승인된 identity/domain·실제 Compartment scope를 확인한다.

## 비용및주의사항

Events rule1개와 private Standard empty bucket1개만 새로 만들고86 topic을 재사용한다. object 업로드가 없어도 bucket을 무료라고 보장하지 않는다. 현재 Events/Notifications·Object Storage 요청·잔존 조건을 확인하고5분 email 관찰 뒤 rule disable/delete와 exact bucket 정리를 수행한다. 실제 event가 늦으면 replay/추가 bucket을 자동 생성하지 않는다. bucket 이름/event metadata에 개인 정보를 넣지 않는다.

## 실습구성

### 워크숍 순서와 입력값

**진행 순서:** 86 actual수신 준비 →condition/action 저장·재조회 →emptybucket1회 →actualemail/resourceId →disable/delete →85 증거인계

**선택 이유:** 가짜 event를 만들지 않고 실제 control-plane state change를 사용한다.

| 입력 | 값 또는 정확한 출처 | 실행 위치 | 생성 전/후 | 이유 |
|---|---|---|---|---|
| TOPIC_OCID/COMPARTMENT_OCID | 86topic/실습scope | Console | 전 | filter/actionscope |
| NS/EVENT_BUCKET | namespace actual /유효고유ASCII이름 | Cloud Shell | 전 | 기존bucket 충돌금지 |
| OP_START_UTC/OP_END_UTC | 실제생성·삭제포함window UTC | Cloud Shell→85 | 후 | Audit 오래된작업도 추적 |


값이 다른 터미널에 자동 전파되지 않습니다. Console readback으로 비밀 없는 값만 인계하며, placeholder는 실제값으로 교체한 뒤 실행합니다. 새 리소스 생성 전 이름 충돌·소유자·최대 비용·종료 UTC를 확인하고 선택하지 않은 분기는 **NOT_SELECTED**로 기록합니다.


bucket Create event라는 실제 OCI service state change를 rule → topic → confirmed email로 전달한다. Console 예시 JSON의 filter 검사와 실제 bucket create event/email 결과를 별도 증거로 구분한다.


### 이번 실습의 생성·재사용·앱 경계

| 구분 | 이번 주제의 실제 범위 |
|---|---|
| 새로 생성 | Events rule1개/empty privatebucket1개 |
| 재사용 | 86 confirmedtopic |
| 앱/작업부하 및 변경 | 실제bucket control-plane event→email;object/app 서버 없음 |
| 실행/반환 순서 | 86 topic/subscription ACTIVE 및 실제test 수신 완료 → rule/bucket 삭제 readback 및 OP_START/END_UTC를85에 인계 |

새로 생성하지 않는 항목을선행완료로가장하지않는다.위표의실제OCID/주소와진행순서를첫mutation전에확인한다.

## 실습절차

### 실제 bucket 이벤트를 정확히 연결하기
86 actual topic OCID와 ACTIVE subscription을 확인한다. rule에는 이벤트 타입과 Compartment **둘 다** scope를 넣고 저장된 condition/action을 다시 읽는다. bucket 이름은 유효한 고유 ASCII suffix로 정하고 `EVENT_BUCKET`을 Audit85까지 보관한다. 기존 bucket이 있으면 재생성/삭제하지 않는다. namespace 읽기에는 `OBJECTSTORAGE_NAMESPACE_READ`도 필요하다.
실패·회복 과제는 rule Enabled 상태의 condition/action readback으로 잘못된 topic/filter를 발견하면 **본인 rule만** 수정하여 복구한다. 이미 bucket create가 끝났는데 email이 없다면 뒤늦은 수정으로 이전 event가 반드시 replay된다고 가정하지 않는다. 추가 bucket을 자동 생성하지 말고 실패 및 재시험 승인을 기록한다. email과 bucket get의 actual resourceId/time을 대조한다.

### 실행 단계

1. Observability & Management → Events service → Rules → Create rule. Name **roadmap-bucket-create**, Compartment **실습 Compartment**, Description **single bucket creation test**. Rule condition: **Event type**, Service **Object Storage**, Event **Bucket - Create**를 선택한다. 생성 완료 event identifier는 Console preview에서 **com.oraclecloud.objectstorage.createbucket**인지 확인한다. filter 추가: Attribute **compartmentId**, value **실습 Compartment OCID**. filter를 현재 Console이 노출하지 않으면 rule 생성 Compartment scope를 유지하고 공식 filter 본문에 맞춰 preview를 확인한다. Console이 제시하는 condition을 읽어 scope를 확인한다.[T1]
2. Actions: Action type **Notifications**, Topic compartment **86 topic Compartment**, Topic **roadmap-topic**. 아직 Create하지 않는다. 아래 helper 검사를 마친 후 rule **Enabled**로 Create하고 condition/action을 다시 읽는다. condition JSON은 다음 의미를 갖는다.
```json
{"eventType":["com.oraclecloud.objectstorage.createbucket"],"data":{"compartmentId":["실제 Compartment OCID"]}}
```
### 생성 전 filter positive/negative 확인 (example이지 실제 event 아님)
Create Rule의 **Validate rule**에서 Service Name Object Storage, Event Type Bucket-Create를 선택한다. 예시 JSON의 data.compartmentId만 승인 actual Compartment로 바꾸어 Check if Example Event Matches Rule을 누르고 match를 읽는다. 동일 예시의 compartmentId만 **승인된 다른 scope 또는 불일치 test 문자열**로 바꾸어 no-match를 읽고 actual scope로 되돌려 match를 재확인한다. 이는 Console helper의 예시 필터 검사이며 OCI bucket event·email 배달 증거가 아니다. Preview rule logic에서 actual condition/action/topic을 재조회한 뒤 실제 bucket을 딱 한 번 생성하는 아래 단계로 진행한다.


3. namespace를 읽고 실제 빈 bucket 1개를 생성한다. name은 본인 tenancy에서 고유한 소문자 suffix로 바꾼다.
```bash
set -euo pipefail
OP_START_UTC=$(date -u +%Y-%m-%dT%H:%M:%SZ)
export NS=$(oci os ns get --query data --raw-output)
export EVENT_BUCKET='roadmap-event-actual-approved-ascii-suffix'
# 위 suffix를 실제승인고유값으로교체하고Console목록에서부재를먼저확인한다.
[[ "$EVENT_BUCKET" != 'roadmap-event-actual-approved-ascii-suffix' ]] || { printf 'replace example bucket name: stop\n'; exit 1; }
oci os bucket create --namespace-name "$NS" --compartment-id "$COMPARTMENT_OCID"  --name "$EVENT_BUCKET" --storage-tier Standard --public-access-type NoPublicAccess --freeform-tags '{"lab":"roadmap-71-92"}'
oci os bucket get --namespace-name "$NS" --bucket-name "$EVENT_BUCKET"
```
4. 최대 5분 내 email에서 eventType, 실제 bucket name/resourceId, eventTime, compartmentId를 대조한다. Console Events rule metric과 Audit의 CreateBucket도 함께 확인한다. Object upload용 **Emit object events**는 이 bucket 생성 실험의 전제조건이 아니다.
5. 미수신이면 rule/actions/Region/filter/IAM을 점검하고 bucket을 여러 번 만들어 재시도하지 않는다. rule Disable 후 bucket 삭제한다.


### 진행·종료 판정
설정 readback·기능시험·원복·정리를 각각 실제 응답으로 기록한다. 기대값은 미실행이며 실패/만료 시 새 mutation을 중단하고 아래 topic별 종료 단계로 반환한다.

### Actual bucket operation evidence handoff to Audit
생성전에Console Bucket목록/CLI Get로같은name이존재하지않음을확인한다. create호출의actualUTC와응답 opc-request-id(현재CLI가표시하는header 또는Console/API작업기록)를비공개실행자료에기록한다. `oci --debug`전체출력에는인증자료가있을수있어수집/공유하지않는다. bucketget의actualresource와메일data.resourceId/name/time을대조한다. 삭제후종료시각을인계한다.
```bash
# 아래 출력은 리소스정리의 exact bucket 삭제 readback까지 끝난 뒤 실행한다.
OP_END_UTC=$(date -u +%Y-%m-%dT%H:%M:%SZ)
printf '%s %s %s\n' "$EVENT_BUCKET" "$OP_START_UTC" "$OP_END_UTC"
```
85에는bucket명/scope/Region/실제operationUTCwindow만인계한다. requestId가없으면85actualAudit에서읽고다른필드와교차검증하며invent하지않는다.


### 번호 순서로 완료하는 원복·정리

6. **Events 종료 및 실패 반환.** rule Disable/저장/disabled readback→rule Delete/목록부재→exact emptybucket Delete/확인prompt→same-scope Get NotFound 및목록부재를확인한다.권한오류는삭제완료아니다.OP_END_UTC를삭제readback뒤기록해85에actualwindow를인계하고86topic은owner반환한다.

실제 정리/원복 response, exact resource 또는rule ID,완료UTC와남은owner/expiry를비밀없는개인실행기록에남긴다.위의guarded복구가이미완료된경우중복mutation하지않는다.

## 결과검증

**기대 패턴(미실행):** actual bucket exists + createbucket event의 실제 timestamp/resourceName을 담은 email 1건 이상. at-least-once delivery는 중복될 수 있어 exact one을 보장하지 않는다.

실행 기록(미기입): 실행 UTC __ / 실제 resource OCID __ / 요청·SQL 실제 응답 __ / 공식 조건 차이 __ / 정리 완료 UTC __.

## 문제해결

rule condition preview에서 event 이름이 다르면 현재 official/Console event catalog와 비교하고 문서 identifier를 blindly 입력하지 않는다. subscription PENDING, action 비활성, cross-service permission을 우선 점검한다.

## 리소스정리

rule Disable → rule 삭제 → 실습 empty bucket 삭제 (`oci os bucket delete --namespace-name "$NS" --bucket-name "$EVENT_BUCKET"` 확인 prompt에서 실제 이름 확인). bucket delete 후 exact name Get이NotFound인것과목록부재를확인한다. authorization failure를삭제성공으로해석하지않는다. topic은 모든 연계 실습 완료 후 86에 따라 삭제한다.

### 소유자별 종료와 exact 대상 재조회
83 owner: actual Events rule Disable 저장·enabled=false → Delete/confirm·Rules 부재 → 본인 exact empty bucket 삭제·same scope Get NotFound 및 목록 부재 확인. OP_END_UTC는 **삭제 readback 뒤** 기록하며85로 create/delete를 포함한 window를 인계한다. Get forbidden은 NotFound와 구별한다.

실제 정리 UTC __ / exact resource 또는 rule ID __ / 삭제 후 조회 응답 __ / 잔존 resource·만료·owner __. 승인 없는 잔존 비용은 다음 실습을 이유로 연장하지 않는다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 개념은 해당 생성·변경·조회 단계에서 입력 선택과 실제 결과 해석을 위해 사용하며, 공식 예시는 실제 실습 실행 결과가 아닙니다.

- [18] [Overview of Events — Events](https://docs.oracle.com/en-us/iaas/Content/Events/Concepts/eventsoverview.htm)
  - 확인 범위: Oracle Cloud Infrastructure Events enables.

### 실습 task 본문 근거

- [T1] [Concepts / eventspolicy.htm](https://docs.oracle.com/en-us/iaas/Content/Events/Concepts/eventspolicy.htm) — 본문 절차·입력 필드·지원 조건을 읽어 실습에 적용.
- [T2] [Task / managingrules.htm](https://docs.oracle.com/iaas/Content/Events/Task/managingrules.htm) — 본문 절차·입력 필드·지원 조건을 읽어 실습에 적용.
- [T3] [Concepts / eventsoverview.htm](https://docs.oracle.com/en-us/iaas/Content/Events/Concepts/eventsoverview.htm) — 본문 절차·입력 필드·지원 조건을 읽어 실습에 적용.


- [T-CREATE] [Create Events Rule 필터·action Console 필드](https://docs.oracle.com/en-us/iaas/Content/Events/Task/create-events-rule.htm) — Create Events Rule 필터·action Console 필드 본문을 실제 읽어 적용.

- [T-EVENT] [Object Storage Bucket create event identifier](https://docs.oracle.com/en-us/iaas/Content/Events/Reference/eventsproducers.htm) — Object Storage Bucket create event identifier 본문을 실제 읽어 적용.
