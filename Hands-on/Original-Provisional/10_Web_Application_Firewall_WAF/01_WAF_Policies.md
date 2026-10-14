> **기존 실습 초안 — 미검증**: 원래 93개 문서의 절차를 보존한 자료입니다. 새 튜토리얼의 품질 승인이나 OCI 실행 검증을 뜻하지 않습니다.

# WAF Policies

> 문서 작성 검토와 실행 검증은 별개입니다. OCI 실행은 **USER_EXECUTION_PENDING**이며 사용자 캡처는 별도 자료입니다. 미실행 자체는 문서 작성 승인 차단 사유가 아닙니다. 실제 결과 필드는 실행자가 채웁니다.

> 실행용 한국어 교육 문서. OCI 리소스 생성·명령 실행 결과는 아직 검증하지 않았으며 모든 기대 패턴은 미실행이다.

## 학습목표

- 기존 roadmap-waf에 Response code 403/429, Content-Type text/plain, 전용 응답 body를 가진 action 두 개를 추가하고 저장값을 다시 읽는다.
- rule이 참조하지 않은 action 생성 전후 같은 `/probe` 요청의 200/roadmap-ok를 비교한다.
- 73~75의 실제 403/429 결과와 action 설정을 연결하고 모든 참조 rule 제거 뒤 본인 action만 삭제한다.

실제 OCI 요청·응답/SQL·정리 증거는 실행자가 기록한다. 아래 기대값은 실측값이 아니다.

## 대응 Roadmap 항목

- 항목명: **WAF Policies**
- Roadmap 경로: **Web Application Firewall (WAF) → WAF Policies**
- Node ID: `service-waf-1`
- 상위 항목: **Web Application Firewall (WAF)** (`service-waf-0`)
- 문서 범위: 이 항목의 개념, 실제 설정 절차와 학습자 실행 검증.
- [전체 항목 목록](../../../Docs/README.md)

## 사전준비

**시작 조건:** 71의 ACTIVE policy/firewall·/probe200·service log 완료

- [Web Application Firewall (WAF)](00_Web_Application_Firewall_WAF.md#waf-setup) — 1~5단계 완료: 정상 HTTP 200, ACTIVE policy/firewall, WAF log 활성화. 리소스 유지.

### 준비 완료 출력과 인수 확인

| 인수 항목 | 실제 출력·출처 | 다음 작업의 사용 위치 |
|---|---|---|
| 선행 완료 상태 | 71의 ACTIVE policy/firewall·/probe200·service log 완료 | 첫 변경 전 재조회 |
| 비밀 없는 실행 입력 | actual policy/firewall/log OCID, LAB_URL, /health body ok, owner71 expiry | 아래 numbered Console/명령 단계 |
| 반환/후속 인계 | 403/429 action 이름·code/header/body를73~75에 인계 | 종료 UTC 이전 exact owner 반환 |

실제 Compartment/Region은 승인 Console에서 읽고 동일 identity/Region을 고정한다. VCN client에서 `command -v curl`과 승인된 backend sudo/nginx 권한을 확인한다. placeholder·명칭만 인수한 상태는 준비 완료가 아니다. 비밀번호·키·확인 token은 출력표에 적지 않는다.

## 권한

본 checkpoint의 policy 변경·읽기와기존service log검색만관리자가승인한다.예시scoped grant:
```text
Allow group <LabGroup> to use waf-policy in compartment <LabCompartment>
Allow group <LabGroup> to read log-content in compartment <LabCompartment>
```
공유firewall/LB 생성·Logging enable은71 creator 작업이며본checkpoint에서추가하지않는다.Console resource selector에필요한조회scope는관리자와확인한다.

Actions는 별도 action 서버 생성 API가 아니라 policy update 범위: UpdateWebAppFirewallPolicy=WAF_POLICY_UPDATE, 재조회 GetWebAppFirewallPolicy=WAF_POLICY_READ. 삭제도 policy update이며 참조 rule을 먼저 제거한다. 본 checkpoint에서 firewall/LB/log를 새로 만드는 권한을 사용하지 않는다.

operation 근거: [공식 권한/작업 본문](https://docs.oracle.com/iaas/Content/Identity/Reference/wafpolicyreference_topic-Details_for_WAF_Policies.htm). family grant는 minimum-permission 정책으로 주장하지 않는다. 실행 전에 관리자 승인된 identity/domain·실제 Compartment scope를 확인한다.

## 비용및주의사항

새 LB/firewall/policy 없이 71 policy 안에 403/429 action 두 개만 추가한다. action 정의 자체를 별도 VM 또는 무료 리소스라고 취급하지 않는다. 같은 LB의 control 요청 및 기존 WAF service log에 대한 현재 과금·보존 조건을 확인한다. 73~75 참조가 끝날 때까지 action을 유지하되 71의 예산/종료 UTC는 연장하지 않는다. 사용하지 않는 action 생성만으로 차단 또는 추가 서버 배포가 완료된 것은 아니다.

## 실습구성

### 워크숍 순서와 입력값

**진행 순서:** 71 ACTIVE/로그 인수 → 403/429 action 생성·재조회 → 미연결 정상200 → 73 인계

**선택 이유:** 정의와 적용을 분리하고 Check/Allow 차이를 실제 정상 요청으로 확인한다.

| 입력 | 값 또는 정확한 출처 | 실행 위치 | 생성 전/후 | 이유 |
|---|---|---|---|---|
| LAB_URL | 71인수주소 | vm-a guest | 전 | 동일경로200 |
| policy/actions | roadmap-waf / roadmap-block-403 / roadmap-rate-429 | Console | 전/후 | 공유policy와정확한 참조 |


값이 다른 터미널에 자동 전파되지 않습니다. Console readback으로 비밀 없는 값만 인계하며, placeholder는 실제값으로 교체한 뒤 실행합니다. 새 리소스 생성 전 이름 충돌·소유자·최대 비용·종료 UTC를 확인하고 선택하지 않은 분기는 **NOT_SELECTED**로 기록합니다.


71의 policy 1개에 Check와 HTTP 응답 action을 설정한다. 새로운 LB나 policy를 생성하지 않는다.


### waf-actions



### 이번 실습의 생성·재사용·앱 경계

| 구분 | 이번 주제의 실제 범위 |
|---|---|
| 새로 생성 | policy 내부403/429 action 두 개 |
| 재사용 | 71 policy/firewall/log와61 LB/nginx 앱 |
| 앱/작업부하 및 변경 | action 정의만;rule 참조 전에는응답변화 없음 |
| 실행/반환 순서 | 71의 ACTIVE policy/firewall·/probe200·service log 완료 → 403/429 action 이름·code/header/body를73~75에 인계 |

새로 생성하지 않는 항목을선행완료로가장하지않는다.위표의실제OCID/주소와진행순서를첫mutation전에확인한다.

## 실습절차

### policy 설정을 과제로 완주하기
생성 owner는 71이다. 새 shell의 VCN client에서 `export LAB_URL='http://61의실제LBIPv4'`를 준비하고 71의 `/probe` 정상 200을 재검증한다. Actions → Add action에서 403/429의 code, `Content-Type` header, body를 각각 저장·재조회하고 아직 rule이 참조하지 않는다는 것을 확인한다. body가 빈 default response와 혼동되지 않도록 명시한 marker를 쓴다.
선행 잘못된 rule로 정상 probe가 차단되면 **본인이 만든** rule만 Check로 복구 후 정상 요청을 다시 보낸다. shared policy 전체 default action을 바꾸지 않는다. 정리는 action 참조 rule 제거→Actions에서 미사용 확인→action 삭제이며 후속 73~75가 끝나기 전 삭제하지 않는다.

### 변경 전 실제 client control
vm-a guest에서 actual LAB_URL을71에서 인수해 설정한다. 다음 요청의 실제 status/body를 기록한 뒤만 아래 Console 변경으로 진행한다.
```bash
curl --fail --max-time 10 -i "$LAB_URL/probe"
```

### 실행 단계

1. WAF → Policies → **roadmap-waf** → Actions에서 existing preconfigured Check action을 확인한다. Check는 현재 module의 다음 rule을 계속 실행하며 로그를 남긴다. Allow는 현재 module의 남은 rule을 건너뛰므로 blanket Allow rule을 만들지 않는다.[T1]
2. Actions → Add action. Name **roadmap-block-403**, Type **Return HTTP response**, Response code **403**, Response page body **roadmap-waf-blocked**. 저장한다. Rate Limiting용 action도 Name **roadmap-rate-429**, Response code **429**, Response page body **roadmap-rate-limited**로 생성한다. Headers → Header name **Content-Type**, Header value **text/plain**, Dynamic text support **off**를 각각 명시하고 Add action한다. 전용 body는 앱 응답과 차단 응답을 구별한다.
3. Actions 목록에서 이름·유형·status를 다시 읽어 확인한다. 73의 Protection에서는 먼저 Check를 선택하고 74의 Access Control에는 403 action, 75에는 429 action을 참조한다.
4. 정책 단계 순서는 requestAccessControl → requestRateLimiting → requestProtection → backend → responseAccessControl → responseProtection이다. 이전 module에서 응답을 반환하면 backend에 도달하지 않는다.[T2]
5. 저장 후 ACTIVE를 확인하고 변경 전과 같은 `curl --fail --max-time 10 -i "$LAB_URL/probe"` 및 `/health` 1회를 요청한다. action을 생성만 한 시점에는 정상 200이어야 한다. 지금 즉시 403이면 이전 실습 rule/default action을 조사한다.


### 진행·종료 판정
설정 readback·기능시험·원복·정리를 각각 실제 응답으로 기록한다. 기대값은 미실행이며 실패/만료 시 새 mutation을 중단하고 아래 topic별 종료 단계로 반환한다.

### 공유 공개 scope 및 실제 WAF 로그 인수
[71의 scope/실제 service log 절차](00_Web_Application_Firewall_WAF.md#waf-log-correlation)를 사용한다. public mode이면 별도 승인된 네 exact WAF 경로만 추가 공개하며 `/`, `/index.txt`, `/health.saved`, `/unknown`, `/health/`는403을 유지한다. root200을 이 branch의 정상 기준으로 사용하지 않는다. 기존 승인 source/만료는 바꾸지 않는다.

### 기존 rule 변경 경로
잘못된 본인 access rule은 policy → Access control → Request access rules → exact rule 행의 Actions → Edit에서 Action을 Preconfigured check action으로 저장하고 ACTIVE 후 해당 행 condition/action을 다시 읽는다. Protection rule은73의 rule action과 capability override **두 곳**을 재조회한다. Actions 목록에서 response action의 code/body를 편집하는 일과 rule이 참조하는 action 변경은 다른 작업이다. 운영 default/rule은 변경하지 않는다.


### 번호 순서로 완료하는 원복·정리

6. **WAF Policies 종료 및 실패 반환.** 73~75 반환 후 Actions 목록에서 본인403/429 action을 참조하는 rule이 없는지 읽고 두 action만 Delete/Confirm→목록부재 확인한다. 아직 consumer가있으면expiry 안에서만 유지하며 공유policy/firewall/LB는삭제하지않는다.

실제 정리/원복 response, exact resource 또는rule ID,완료UTC와남은owner/expiry를비밀없는개인실행기록에남긴다.위의guarded복구가이미완료된경우중복mutation하지않는다.

## 결과검증

action 생성은 차단 활성화가 아니다. 실제 `/health` 200과 Actions 목록의 403/429 설정을 기록한다. **기대 패턴(미실행):** 사용하지 않은 action 때문에 HTTP 응답이 바뀌지 않음.

실행 기록(미기입): 실행 UTC __ / 실제 resource OCID __ / 요청·SQL 실제 응답 __ / 공식 조건 차이 __ / 정리 완료 UTC __.

## 문제해결

policy가 UPDATING이면 ACTIVE까지 기다린다. 기존 운영 policy 이름을 혼동하지 않도록 OCID로 다시 확인한다. Check와 Allow를 혼동했으면 운영 전환 전에 수정한다.

## 리소스정리

action을 참조한 73~75 rule을 먼저 삭제한 뒤 403/429 action을 삭제한다. 71의 policy/firewall은 75 종료까지 유지한다.

### 소유자별 종료와 exact 대상 재조회
72 owner: policy Actions 목록에서 actual403/429 각각의 참조를73~75 rule readback과 대조한다. 참조 rule 없는 뒤만 해당 action의 Actions/Delete를 수행하고 이름/type 부재를 refresh 확인한다. 공유 policy/firewall 삭제는71 owner다.

실제 정리 UTC __ / exact resource 또는 rule ID __ / 삭제 후 조회 응답 __ / 잔존 resource·만료·owner __. 승인 없는 잔존 비용은 다음 실습을 이유로 연장하지 않는다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 개념은 해당 생성·변경·조회 단계에서 입력 선택과 실제 결과 해석을 위해 사용하며, 공식 예시는 실제 실습 실행 결과가 아닙니다.

- [39] [Overview of Web Application Firewall — WAF Policies](https://docs.oracle.com/en-us/iaas/Content/WAF/Concepts/overview.htm)
  - 확인 범위: WAF Policies.

### 실습 task 본문 근거

- [T1] [Adding a WAF action / create_action.htm](https://docs.oracle.com/en-us/iaas/Content/WAF/Actions/create_action.htm) — 본문 절차·입력 필드·지원 조건을 읽어 실습에 적용.
- [T2] [Policies / getting_started_with_waf-policies.htm](https://docs.oracle.com/en-us/iaas/Content/WAF/Policies/getting_started_with_waf-policies.htm) — 본문 절차·입력 필드·지원 조건을 읽어 실습에 적용.

