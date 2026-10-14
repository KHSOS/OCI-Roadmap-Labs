> **기존 실습 초안 — 미검증**: 원래 93개 문서의 절차를 보존한 자료입니다. 새 튜토리얼의 품질 승인이나 OCI 실행 검증을 뜻하지 않습니다.

# Access Control

> 문서 작성 검토와 실행 검증은 별개입니다. OCI 실행은 **USER_EXECUTION_PENDING**이며 사용자 캡처는 별도 자료입니다. 미실행 자체는 문서 작성 승인 차단 사유가 아닙니다. 실제 결과 필드는 실행자가 채웁니다.

> 실행용 한국어 교육 문서. OCI 리소스 생성·명령 실행 결과는 아직 검증하지 않았으며 모든 기대 패턴은 미실행이다.

## 학습목표

- 동일 client의 /deny, /deny?test=1, /denied가 rule 생성 전 모두 200인지 확인한다.
- 정확일치 access rule 하나를 만들어 앞의 두 URI 403/전용 body와 /denied 및 /health 200을 로그와 비교한다.
- rule을 삭제하고 query 포함 동일 URI의 200을 재검증한 뒤 75에 공유 policy를 인계한다.

실제 OCI 요청·응답/SQL·정리 증거는 실행자가 기록한다. 아래 기대값은 실측값이 아니다.

## 대응 Roadmap 항목

- 항목명: **Access Control**
- Roadmap 경로: **Web Application Firewall (WAF) → Access Control**
- Node ID: `service-waf-3`
- 상위 항목: **Web Application Firewall (WAF)** (`service-waf-0`)
- 문서 범위: 이 항목의 개념, 실제 설정 절차와 학습자 실행 검증.
- [전체 항목 목록](../../../Docs/README.md)

## 사전준비

**시작 조건:** 73 rule 삭제와 동일SQLi200 복구;72의403 action 유지

- [WAF Policies](01_WAF_Policies.md#waf-actions) — 403 action과 ACTIVE policy 준비.
- [Request Protection Rules](../../../Docs/10_Web_Application_Firewall_WAF/02_Request_Protection_Rules.md) — 실습 rule을 삭제하고 동일 SQLi query 및 /health 200 복구 확인. capability403 override가 남을 수 있어 Check 전환만으로 인수하지 않는다.

### 준비 완료 출력과 인수 확인

| 인수 항목 | 실제 출력·출처 | 다음 작업의 사용 위치 |
|---|---|---|
| 선행 완료 상태 | 73 rule 삭제와 동일SQLi200 복구;72의403 action 유지 | 첫 변경 전 재조회 |
| 비밀 없는 실행 입력 | LAB_URL, actual policy/log OCID, /deny·/denied 사전body,403 action | 아래 numbered Console/명령 단계 |
| 반환/후속 인계 | deny rule 삭제·query-bearing URI200 복구를75에 인계 | 종료 UTC 이전 exact owner 반환 |

실제 Compartment/Region은 승인 Console에서 읽고 동일 identity/Region을 고정한다. VCN client에서 `command -v curl`과 승인된 backend sudo/nginx 권한을 확인한다. placeholder·명칭만 인수한 상태는 준비 완료가 아니다. 비밀번호·키·확인 token은 출력표에 적지 않는다.

## 권한

본 checkpoint의 policy 변경·읽기와기존service log검색만관리자가승인한다.예시scoped grant:
```text
Allow group <LabGroup> to use waf-policy in compartment <LabCompartment>
Allow group <LabGroup> to read log-content in compartment <LabCompartment>
```
공유firewall/LB 생성·Logging enable은71 creator 작업이며본checkpoint에서추가하지않는다.Console resource selector에필요한조회scope는관리자와확인한다.

Request access rule 생성/수정/삭제는 UpdateWebAppFirewallPolicy(WAF_POLICY_UPDATE), readback은 WAF_POLICY_READ다. WAF log 조회는 LOG_CONTENT_READ, nginx log 읽기는 OS 권한이다. IP/address list를 만들지 않으므로 waf-network-address-list 관리 권한을 새로 요구하지 않는다.

operation 근거: [공식 권한/작업 본문](https://docs.oracle.com/iaas/Content/Identity/Reference/wafpolicyreference_topic-Details_for_WAF_Policies.htm). family grant는 minimum-permission 정책으로 주장하지 않는다. 실행 전에 관리자 승인된 identity/domain·실제 Compartment scope를 확인한다.

## 비용및주의사항

기존 policy에 exact /deny rule 하나만 추가하고 /deny·query·/denied·/health 비교를 최대20회/10분 이내 수행한다. WAF 요청·Logging 비용은71 예산에 포함하며 새 address list, LB, backend는 만들지 않는다. 공용 NAT 사용자를 IP차단하거나 운영 default action을 바꾸지 않는다. rule을 삭제해200을 복구해도 공유71 리소스 비용이 즉시 끝나는 것은 아니다.

## 실습구성

### 워크숍 순서와 입력값

**진행 순서:** 73 복구 → 동일 세 URI200 → exact /deny403 → /denied200 → rule삭제 후 동일URI200

**선택 이유:** url.path의 정확일치로 query가 있는 /deny와 prefix만 같은 /denied를 구분한다.

| 입력 | 값 또는 정확한 출처 | 실행 위치 | 생성 전/후 | 이유 |
|---|---|---|---|---|
| LAB_URL | 71실제LB URL | vm-a guest | 전 | URI/source동일성 |
| condition/action | http.request.url.path == '/deny' / roadmap-block-403 | Console | 전 | exactpath 제어 |


값이 다른 터미널에 자동 전파되지 않습니다. Console readback으로 비밀 없는 값만 인계하며, placeholder는 실제값으로 교체한 뒤 실행합니다. 새 리소스 생성 전 이름 충돌·소유자·최대 비용·종료 UTC를 확인하고 선택하지 않은 분기는 **NOT_SELECTED**로 기록합니다.


path 기반 access rule 1개로 재현 가능한 허용/차단 비교. 지역/IP 차단은 공용 NAT 때문에 결과가 흔들려 기본 실습에서 제외한다.


### 이번 실습의 생성·재사용·앱 경계

| 구분 | 이번 주제의 실제 범위 |
|---|---|
| 새로 생성 | /deny exact access rule 하나 |
| 재사용 | 72 action 및동일LB/nginx 앱/log |
| 앱/작업부하 및 변경 | 본인rule만 추가/삭제;IP list/운영default 변경 없음 |
| 실행/반환 순서 | 73 rule 삭제와 동일SQLi200 복구;72의403 action 유지 → deny rule 삭제·query-bearing URI200 복구를75에 인계 |

새로 생성하지 않는 항목을선행완료로가장하지않는다.위표의실제OCID/주소와진행순서를첫mutation전에확인한다.

## 실습절차

### 규칙 생성 전후 같은 URI 비교
같은 client에서 `export LAB_URL='http://61의실제LBIPv4'`를 준비한다. rule 생성 **전** `/deny`, `/deny?test=1`, `/denied` 모두 backend 200/roadmap-ok임을 요청으로 확인한다. 73 rule은 삭제 완료, 75 rule은 아직 미적용 상태로 고정한다. 차단 뒤 body=roadmap-waf-blocked, access rule name 및 같은 URI/time을 서비스 로그에서 읽는다.
삭제 뒤 `/deny?test=1`도 다시 200임을 확인한다. `/denied` positive control을 계속 유지해 전체 앱 장애와 exact path 차단을 구분한다. expected 403만으로 통과시키지 않는다.

### 변경 전 실제 client control
vm-a guest에서 actual LAB_URL을71에서 인수해 설정한다. 다음 요청의 실제 status/body를 기록한 뒤만 아래 Console 변경으로 진행한다.
```bash
curl --fail --max-time 10 -i "$LAB_URL/deny"
curl --fail --max-time 10 -i "$LAB_URL/deny?test=1"
curl --fail --max-time 10 -i "$LAB_URL/denied"
```

### 실행 단계

1. policy → Access control → Request access rules → Add rule. Name **roadmap-deny-path**, advanced condition **`http.request.url.path == '/deny'`**, Rule action **roadmap-block-403**. Request control의 default action은 정상 요청을 진행시키는 기존 기본값으로 유지한다. all-traffic block 기본값을 설정하지 않는다.[T1]
2. rule 저장 후 ACTIVE 확인. condition에 query까지 넣지 않는 이유는 url.path와 queryParameters가 별도 필드이기 때문이다.[T2]
```bash
curl --max-time 10 -i "$LAB_URL/health"
curl --max-time 10 -i "$LAB_URL/deny"
curl --max-time 10 -i "$LAB_URL/deny?test=1"
curl --max-time 10 -i "$LAB_URL/denied"
```
3. path 정확히 일치하는 두 요청은 403, `/health`, `/denied`는 backend 200을 기대한다. 71에서 지정한 실습 경로는 nginx가 200을 반환하므로 차단이 앱의 404 때문이 아님을 구별할 수 있다.
4. WAF 로그에서 request path, 해당 access rule, action을 조회하고 backend의 실제 nginx access log(`/var/log/nginx/access.log`, `sudo nginx -T`의 access_log 값으로 확인)에서 같은 client·URI·시각을 대조한다. terminal 출력 유무로 요청 도달 여부를 판단하지 않는다. rule 순서를 읽고 앞선 Allow rule이 같은 module의 뒤 rule을 건너뛰지 않는지 점검한다.
5. rule 삭제 후 `/deny`가 다시 200인지 1회 확인한다.


### 진행·종료 판정
설정 readback·기능시험·원복·정리를 각각 실제 응답으로 기록한다. 기대값은 미실행이며 실패/만료 시 새 mutation을 중단하고 아래 topic별 종료 단계로 반환한다.

### Same query-bearing URI recovery
rule 삭제 후 vm-a에서 동일 URI를 다시 확인한다.
```bash
curl --max-time 10 -i "$LAB_URL/deny?test=1"
curl --max-time 10 -i "$LAB_URL/denied"
```
두 요청200/roadmap-ok와 /health200/ok를 기록한다. query를 제거한 다른 URI만 성공한 것으로 복구를 주장하지 않는다.

### 공유 공개 scope 및 실제 WAF 로그 인수
[71의 scope/실제 service log 절차](00_Web_Application_Firewall_WAF.md#waf-log-correlation)를 사용한다. public mode이면 별도 승인된 네 exact WAF 경로만 추가 공개하며 `/`, `/index.txt`, `/health.saved`, `/unknown`, `/health/`는403을 유지한다. root200을 이 branch의 정상 기준으로 사용하지 않는다. 기존 승인 source/만료는 바꾸지 않는다.


### 번호 순서로 완료하는 원복·정리

6. **Access Control 종료 및 실패 반환.** 본인roadmap-deny-path 삭제→ACTIVE 목록부재→동일 /deny?test=1 및 /denied200/roadmap-ok와/health200/ok를 읽어75에반환한다. 공유default/policy는원복대상으로넓히지않는다.

실제 정리/원복 response, exact resource 또는rule ID,완료UTC와남은owner/expiry를비밀없는개인실행기록에남긴다.위의guarded복구가이미완료된경우중복mutation하지않는다.

## 결과검증

**기대 패턴(미실행):** /deny와 /deny?test=1은 403+roadmap-waf-blocked, /denied는 200. 삭제 후 /deny 200. 실제 URI·status·로그 시간으로 검증한다.

실행 기록(미기입): 실행 UTC __ / 실제 resource OCID __ / 요청·SQL 실제 응답 __ / 공식 조건 차이 __ / 정리 완료 UTC __.

## 문제해결

condition validation error는 JMESPath string 작은따옴표 및 `==`를 점검한다. 모든 요청 403이면 default action 및 더 높은 우선순위 rule을 점검한다. 운영 IP deny/allow-list 변경으로 우회하지 않는다.

## 리소스정리

roadmap-deny-path rule 삭제 후 정상 요청 확인. 75를 위해 429 action과 firewall을 유지한다.

### 소유자별 종료와 exact 대상 재조회
74 owner: Request access rules에서 exact roadmap-deny-path Delete/confirm → ACTIVE 목록 부재 → 동일 /deny?test=1 200, /denied200 및 /health200을 읽는다.

실제 정리 UTC __ / exact resource 또는 rule ID __ / 삭제 후 조회 응답 __ / 잔존 resource·만료·owner __. 승인 없는 잔존 비용은 다음 실습을 이유로 연장하지 않는다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 개념은 해당 생성·변경·조회 단계에서 입력 선택과 실제 결과 해석을 위해 사용하며, 공식 예시는 실제 실습 실행 결과가 아닙니다.

- [39] [Overview of Web Application Firewall — Access Control](https://docs.oracle.com/en-us/iaas/Content/WAF/Concepts/overview.htm)
  - 확인 범위: Access Control.

### 실습 task 본문 근거

- [T1] [Adding request access rule / create_access_rule_request_control.htm](https://docs.oracle.com/en-us/iaas/Content/WAF/AccessControl/create_access_rule_request_control.htm) — 본문 절차·입력 필드·지원 조건을 읽어 실습에 적용.
- [T2] [Concepts / understanding_conditions.htm](https://docs.oracle.com/en-us/iaas/Content/WAF/Concepts/understanding_conditions.htm) — 본문 절차·입력 필드·지원 조건을 읽어 실습에 적용.

