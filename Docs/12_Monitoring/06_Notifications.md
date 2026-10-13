# Notifications

> 문서 작성 검토와 실행 검증은 별개입니다. OCI 실행은 **USER_EXECUTION_PENDING**이며 사용자 캡처는 별도 자료입니다. 미실행 자체는 문서 작성 승인 차단 사유가 아닙니다. 실제 결과 필드는 실행자가 채웁니다.

> 실행용 한국어 교육 문서. OCI 리소스 생성·명령 실행 결과는 아직 검증하지 않았으며 모든 기대 패턴은 미실행이다.

## 학습목표

- 승인 수신자와 고유 topic1개/email subscription1개를 만들고 PENDING 및 수신자 직접 confirmation 후 ACTIVE를 재조회한다.
- test publish1회 title/body와 actual inbox 수신시각을 대조한다. messageId만으로 배달 완료를 주장하지 않는다.
- 미확인 구독에서는 배달 완료를 판정하지 않으며 실제 ACTIVE/수신으로 복구한 뒤82/83 반환 후 subscription→topic 삭제를 조회한다.

실제 OCI 요청·응답/SQL·정리 증거는 실행자가 기록한다. 아래 기대값은 실측값이 아니다.

## 대응 Roadmap 항목

- 항목명: **Notifications**
- Roadmap 경로: **Monitoring → Notifications**
- Node ID: `sequence-9-6`
- 상위 항목: **Monitoring** (`sequence-9-0`)
- 문서 범위: 이 항목의 개념, 실제 설정 절차와 학습자 실행 검증.
- [전체 항목 목록](../README.md)

## 사전준비

**시작 조건:** 승인 수신자·실제주소와topic 고유이름 및scope/Region 준비



### 준비 완료 출력과 인수 확인

| 인수 항목 | 실제 출력·출처 | 다음 작업의 사용 위치 |
|---|---|---|
| 선행 완료 상태 | 승인 수신자·실제주소와topic 고유이름 및scope/Region 준비 | 첫 변경 전 재조회 |
| 비밀 없는 실행 입력 | 수신자 동의/개인 mailbox,COMPARTMENT_OCID/OCI_CLI_REGION,absolute expiry/cleanup reserve | 아래 numbered Console/명령 단계 |
| 반환/후속 인계 | actual TOPIC_OCID/subscriptionOCID/ACTIVE·실제marker수신을82/83에 인계 | 종료 UTC 이전 exact owner 반환 |

실제 Compartment/Region은 승인 Console에서 읽고 동일 identity/Region을 고정한다. Console과승인수신자mailbox를사용한다.CLI publish 대안을선택할때만Cloud Shell에서 `oci --version`을 확인한다. placeholder·명칭만 인수한 상태는 준비 완료가 아니다. 비밀번호·키·확인 token은 출력표에 적지 않는다.

## 권한

전용 Compartment에 한정한 정책 예:
```text
Allow group <LabGroup> to manage ons-topics in compartment <LabCompartment>
Allow group <LabGroup> to manage ons-subscriptions in compartment <LabCompartment>
```

CreateTopic=ONS_TOPIC_CREATE; GetTopic=ONS_TOPIC_READ; Create/Delete/GetSubscription 및 ResendSubscriptionConfirmation=ONS_TOPIC_SUBSCRIBE; PublishMessage=ONS_TOPIC_PUBLISH; ListSubscriptions=ONS_SUBSCRIPTION_INSPECT; DeleteTopic=ONS_TOPIC_DELETE. GetConfirmSubscription에는 API 표상 IAM permission이 필요 없지만 URL을 수신자가 가진 것과 본인 승인 여부는 필수다. metrics/alarms 정책은86 자체 작업에는 불필요하다.

operation 근거: [공식 권한/작업 본문](https://docs.oracle.com/en-us/iaas/Content/Identity/policyreference/notificationpolicyreference.htm). family grant는 minimum-permission 정책으로 주장하지 않는다. 실행 전에 관리자 승인된 identity/domain·실제 Compartment scope를 확인한다.

## 비용및주의사항

topic1개/email subscription1개/marker publish1회만 생성한다. 현재 Notifications 발행·email 전달 가격·한도를 확인하고 SMS/HTTPS/Functions 구독을 추가하지 않는다. 수신자 동의와 세션 expiry/cleanup reserve2분을 먼저 확보한다. confirmation URL3일 유효성은 비용 lease 연장을 뜻하지 않는다. PENDING 또는 ACTIVE 미배달이면82/83을 만들지 않고 본인 구독/topic을 정리한다. 두 producer가 인수한 경우 실제 반환까지 비용 owner를 유지한다.

## 실습구성

### 워크숍 순서와 입력값

**진행 순서:** topic1개 →승인email subscription1개 →수신자 직접confirmation →ACTIVE →actualmarker수신 →82/83 후삭제

**선택 이유:** 86은 번호와 달리82/83 전에 실행하며 confirmationtoken은 공유하지 않는다.

| 입력 | 값 또는 정확한 출처 | 실행 위치 | 생성 전/후 | 이유 |
|---|---|---|---|---|
| topic/recipient | roadmap-topic또는승인고유suffix /수신자승인주소 | Console/수신자 | 전 | 주소·token비공개 |
| TOPIC_OCID/apiEndpoint | actualtopic /같은Region APIendpoint | Console/Cloud Shell | 후 | 82/83동일destination |


값이 다른 터미널에 자동 전파되지 않습니다. Console readback으로 비밀 없는 값만 인계하며, placeholder는 실제값으로 교체한 뒤 실행합니다. 새 리소스 생성 전 이름 충돌·소유자·최대 비용·종료 UTC를 확인하고 선택하지 않은 분기는 **NOT_SELECTED**로 기록합니다.


82/83 공유 Notifications foundation. 번호 순서와 실행 순서는 다르며 86을 먼저 완료한다.

<a id="topic-ready"></a>


### 이번 실습의 생성·재사용·앱 경계

| 구분 | 이번 주제의 실제 범위 |
|---|---|
| 새로 생성 | topic/email subscription 각1개 |
| 재사용 | 승인수신자mailbox·Cloud Shell |
| 앱/작업부하 및 변경 | test marker email;웹앱/Compute/metrics 생성 없음 |
| 실행/반환 순서 | 승인 수신자·실제주소와topic 고유이름 및scope/Region 준비 → actual TOPIC_OCID/subscriptionOCID/ACTIVE·실제marker수신을82/83에 인계 |

새로 생성하지 않는 항목을선행완료로가장하지않는다.위표의실제OCID/주소와진행순서를첫mutation전에확인한다.

## 실습절차

### foundation 생성·수신·정리 완주
topic name은 tenancy 안에서 unique이므로 roadmap-topic 기존 이름이 있으면 본인 고유 suffix를 선택하고 82/83 destination을 같은 실제 topic으로 업데이트한다. subscription은 topic과 같은 Compartment에 생성한다. confirmation URL은 공식 email task에서 3일 유효하며 수신자가 직접 확인한다.
PENDING일 때 발행으로 수신 성공을 주장하지 않는다. ACTIVE 후1회 실제 marker 메일→topic OCID/title/body/time 기록이 복구·수신 완료다. 주소 오류는 본인 PENDING subscription 삭제 후 수신자 승인된 주소로1개만 재생성한다. topic의 apiEndpoint와 현재 Region을 확인하며 CLI publish가 실패하면 Console1회 publish 대안도 실제 수신으로 검증한다.

### foundation 세션 deadline (Create 전)
생성 전 승인 absolute expiry를 기록한다. confirmation 대기는 **Create subscription 시각+5분 또는 expiry−2분 중 빠른 때**, ACTIVE 후 marker delivery 대기는 **Publish 시각+5분 또는 expiry−2분 중 빠른 때**까지만이다. 두 단계가 세션 expiry를 연장하지 않는다. token3일 유효성은 비용 lease/대기 허용이 아니다.2분 cleanup reserve가 없으면 새 topic/subscription을 만들지 않는다. 거부·PENDING timeout·ACTIVE-but-undelivered에는 NOT_READY를82/83에 반환하고 producer를 만들지 않는다. 본인 exact subscription(상태 PENDING/ACTIVE 모두) Delete→Subscriptions 부재 확인 후, 실제 producer 없음/다른 subscriber 없음의 **본인 새 topic**만 Delete→Topics exact OCID 부재를 확인한다. 공유 topic은 삭제하지 않고 owner에게 반환한다. 생성응답 불명이면 재Create하지 말고 exact 이름/OCID를 조회한다.

### 실행 단계

1. Observability & Management → Notifications → Topics → Create topic. Name **roadmap-topic**, Description **short-session lab**, Compartment **실습 Compartment**. topic OCID를 기록한다.[T1]
2. topic → Create subscription. Protocol **Email**, Email address **본인 또는 승인된 실습 수신자 실제 주소**. Create 후 상태 **PENDING**을 확인한다.
3. 수신자가 confirmation email의 확인 링크를 직접 열어 승인한다. link/token은 문서·채팅·Git에 남기지 않는다. Console subscription을 refresh하여 **ACTIVE**인지 다시 읽는다. 수신 확인 버튼을 누르지 않은 채 완료 처리하지 않는다.[T2]
4. topic → Publish message. Title **roadmap-notification-test**, Body **roadmap-topic-ready**. Publish 1회 후 실제 inbox에서 title/body를 확인한다. CLI 대안:
```bash
export TOPIC_OCID='실제 topic OCID'
oci ons message publish --topic-id "$TOPIC_OCID"  --title roadmap-notification-test --body roadmap-topic-ready
```
5. 실제 메일 수신 시간과 subscription ACTIVE를 기록한다. 82 alarm/83 event action에서 이 topic을 선택한다. topic 발행 응답의 messageId만으로 email 수신을 주장하지 않는다.


### 진행·종료 판정
설정 readback·기능시험·원복·정리를 각각 실제 응답으로 기록한다. 기대값은 미실행이며 실패/만료 시 새 mutation을 중단하고 아래 topic별 종료 단계로 반환한다.

### 번호 순서로 완료하는 원복·정리

6. **Notifications 종료 및 실패 반환.** 82alarm/83rule 소비자가선택된경우actual삭제반환을먼저받는다.exact subscription Delete/Confirm→목록부재→본인emptytopic Delete/Confirm→Topics exactOCID부재를읽는다.PENDING거부/timeout은producer없이즉시본인자원정리하며sharedtopic을삭제하지않는다.

실제 정리/원복 response, exact resource 또는rule ID,완료UTC와남은owner/expiry를비밀없는개인실행기록에남긴다.위의guarded복구가이미완료된경우중복mutation하지않는다.

## 결과검증

**기대 패턴(미실행):** PENDING→ACTIVE, test publish가 actual inbox에 도착. 후속 82/83이 완료될 때까지 topic/subscription 유지한다.

실행 기록(미기입): 실행 UTC __ / 실제 resource OCID __ / 요청·SQL 실제 응답 __ / 공식 조건 차이 __ / 정리 완료 UTC __.

## 문제해결

confirmation mail 미수신이면 spam·주소·재전송 기능·subscription 만료를 확인한다. 수신자가 confirmation을 거부하면 즉시 중단하고 subscription 삭제한다. ACTIVE인데 test 미수신이면 publish 권한·message delivery metrics를 확인한다.

## 리소스정리

82 alarm disable/delete 및 83 rule disable/delete가 선행. email subscription 삭제 → 목록에서exactsubscription없음확인 → topic 삭제 → Topics에서exactOCID없음확인. 확인 token이나 수신자 주소가 들어간 개인 evidence는 public 저장 금지.

### 소유자별 종료와 exact 대상 재조회
86 owner는82 alarm 및83 rule 실제 삭제 반환을 받은 뒤 Topics→exact topic→Subscriptions→exact subscription Delete/confirm → subscription 목록 부재 → topic Delete/confirm → Topics 목록 부재를 읽는다. 수신자 거부/PENDING 만료이면 시험을 중단하고 본인 구독만 정리한다.

실제 정리 UTC __ / exact resource 또는 rule ID __ / 삭제 후 조회 응답 __ / 잔존 resource·만료·owner __. 승인 없는 잔존 비용은 다음 실습을 이유로 연장하지 않는다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 개념은 해당 생성·변경·조회 단계에서 입력 선택과 실제 결과 해석을 위해 사용하며, 공식 예시는 실제 실습 실행 결과가 아닙니다.

- [34] [Overview of Notifications — Notifications](https://docs.oracle.com/en-us/iaas/Content/Notification/Concepts/notificationoverview.htm)
  - 확인 범위: The Notifications service lets / How Notifications Works.

### 실습 task 본문 근거

- [T-CREATE] [Creating a topic](https://docs.oracle.com/en-us/iaas/Content/Notification/Tasks/create-topic.htm) — actual creation fields and uniqueness.
- [T-EMAIL] [Creating an email subscription](https://docs.oracle.com/en-us/iaas/Content/Notification/Tasks/create-subscription-email.htm) — same-compartment creation and three-day confirmation validity.
- [T1] [Concepts / notificationoverview.htm](https://docs.oracle.com/en-us/iaas/Content/Notification/Concepts/notificationoverview.htm) — 본문 절차·입력 필드·지원 조건을 읽어 실습에 적용.
- [T2] [Tasks / confirm-subscription.htm](https://docs.oracle.com/en-us/iaas/Content/Notification/Tasks/confirm-subscription.htm) — 본문 절차·입력 필드·지원 조건을 읽어 실습에 적용.


- [T-ONS] [ons-topics / ons-subscriptions 권한](https://docs.oracle.com/en-us/iaas/Content/Identity/policyreference/notificationpolicyreference.htm) — ons-topics / ons-subscriptions 권한 본문을 실제 읽어 적용.
