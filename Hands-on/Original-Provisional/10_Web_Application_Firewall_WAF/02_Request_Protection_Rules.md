> **기존 실습 초안 — 미검증**: 원래 93개 문서의 절차를 보존한 자료입니다. 새 튜토리얼의 품질 승인이나 OCI 실행 검증을 뜻하지 않습니다.

# Request Protection Rules

> 문서 작성 검토와 실행 검증은 별개입니다. OCI 실행은 **USER_EXECUTION_PENDING**이며 사용자 캡처는 별도 자료입니다. 미실행 자체는 문서 작성 승인 차단 사유가 아닙니다. 실제 결과 필드는 실행자가 채웁니다.

> 실행용 한국어 교육 문서. OCI 리소스 생성·명령 실행 결과는 아직 검증하지 않았으며 모든 기대 패턴은 미실행이다.

## 학습목표

- 같은 SQLi query를 direct backend에서 200으로 확인하고 /probe 한정 942100 capability를 현재 version으로 Check 적용한다.
- Check의 200+탐지로그와 Block의 403+capability/action 로그를 정상 query 200과 비교한다.
- 본인 rule을 삭제하고 동일 query의 200/roadmap-ok 회복을 확인한다. 탐지 미관측은 성공으로 기록하지 않는다.

실제 OCI 요청·응답/SQL·정리 증거는 실행자가 기록한다. 아래 기대값은 실측값이 아니다.

## 대응 Roadmap 항목

- 항목명: **Request Protection Rules**
- Roadmap 경로: **Web Application Firewall (WAF) → Request Protection Rules**
- Node ID: `service-waf-2`
- 상위 항목: **Web Application Firewall (WAF)** (`service-waf-0`)
- 문서 범위: 이 항목의 개념, 실제 설정 절차와 학습자 실행 검증.
- [전체 항목 목록](../../../Docs/README.md)

## 사전준비

**시작 조건:** 72 action 완료 및71 모든 backend /probe200 준비

- [WAF Policies](01_WAF_Policies.md#waf-actions) — 403 action 생성과 HTTP 200 검증 완료. 71의 LB/firewall/log 유지.

### 준비 완료 출력과 인수 확인

| 인수 항목 | 실제 출력·출처 | 다음 작업의 사용 위치 |
|---|---|---|
| 선행 완료 상태 | 72 action 완료 및71 모든 backend /probe200 준비 | 첫 변경 전 재조회 |
| 비밀 없는 실행 입력 | LAB_URL/BACKEND_IP, actual policy/log OCID, 현재942100 key/version,403 action | 아래 numbered Console/명령 단계 |
| 반환/후속 인계 | rule 삭제·동일SQLi200 복구를74에 반환 | 종료 UTC 이전 exact owner 반환 |

실제 Compartment/Region은 승인 Console에서 읽고 동일 identity/Region을 고정한다. VCN client에서 `command -v curl`과 승인된 backend sudo/nginx 권한을 확인한다. placeholder·명칭만 인수한 상태는 준비 완료가 아니다. 비밀번호·키·확인 token은 출력표에 적지 않는다.

## 권한

본 checkpoint의 policy 변경·읽기와기존service log검색만관리자가승인한다.예시scoped grant:
```text
Allow group <LabGroup> to use waf-policy in compartment <LabCompartment>
Allow group <LabGroup> to read log-content in compartment <LabCompartment>
```
공유firewall/LB 생성·Logging enable은71 creator 작업이며본checkpoint에서추가하지않는다.Console resource selector에필요한조회scope는관리자와확인한다.

ListProtectionCapabilities=WAF_POLICY_INSPECT; rule/capability action 저장=UpdateWebAppFirewallPolicy(WAF_POLICY_UPDATE); 재조회=WAF_POLICY_READ; 서비스 로그 조회=LOG_CONTENT_READ. 신규 LB/Compute 권한은 필요하지 않다. nginx access log 읽기는 OS sudo 권한으로 OCI IAM과 별개다.

operation 근거: [공식 권한/작업 본문](https://docs.oracle.com/iaas/Content/Identity/Reference/wafpolicyreference_topic-Details_for_WAF_Policies.htm). family grant는 minimum-permission 정책으로 주장하지 않는다. 실행 전에 관리자 승인된 identity/domain·실제 Compartment scope를 확인한다.

## 비용및주의사항

동일 policy의 /probe 한정 protection rule 하나와 현재 capability/version만 사용한다. 정상·SQLi query를 각 단계 한 번씩 보내며 최대20회/10분 로그 관찰을 넘겨 문자열을 바꿔 공격하지 않는다. WAF 요청과 service log 사용량 및 61의 기존 LB 유지 비용을 승인한다. SQL 문자열은 lab HTTP query이며 실제 DB에 실행하지 않는다. 탐지 미관측을 무료 재시험 권리로 해석하지 않는다.

## 실습구성

### 워크숍 순서와 입력값

**진행 순서:** 동일 query direct200 → SQLi Check+로그 → Block403+로그 → Check/삭제200 복구

**선택 이유:** signature 탐지와 실제 enforcement를 구분하며 실습 SQL query는 DB에 실행하지 않는다.

| 입력 | 값 또는 정확한 출처 | 실행 위치 | 생성 전/후 | 이유 |
|---|---|---|---|---|
| BACKEND_IP/LAB_URL | 61/71실제값 | vm-a guest | 전 | direct와WAF경로 분리 |
| capability/key/version | 현재942100 실제version | Console | 전 | 지원없는version 추측금지 |


값이 다른 터미널에 자동 전파되지 않습니다. Console readback으로 비밀 없는 값만 인계하며, placeholder는 실제값으로 교체한 뒤 실행합니다. 새 리소스 생성 전 이름 충돌·소유자·최대 비용·종료 UTC를 확인하고 선택하지 않은 분기는 **NOT_SELECTED**로 기록합니다.


Request protection rule 1개. 단일 SQLi capability로 Check→Block 전환을 실제 요청과 비교한다.


### 이번 실습의 생성·재사용·앱 경계

| 구분 | 이번 주제의 실제 범위 |
|---|---|
| 새로 생성 | /probe protection rule 하나 |
| 재사용 | 72 action 및같은LB/nginx 앱/log |
| 앱/작업부하 및 변경 | Check→Block→삭제;애플리케이션DB SQL 실행 없음 |
| 실행/반환 순서 | 72 action 완료 및71 모든 backend /probe200 준비 → rule 삭제·동일SQLi200 복구를74에 반환 |

새로 생성하지 않는 항목을선행완료로가장하지않는다.위표의실제OCID/주소와진행순서를첫mutation전에확인한다.

## 실습절차

### 동일 요청·동일 backend positive control
같은 VCN client shell에서 `export LAB_URL='http://61의실제LBIPv4'`를 입력한다. 71 확장과 direct backend :8080 경로가 정상임을 확인한다. rule 생성 전에도 동일 `--get --data-urlencode` 공격 query를 **승인된 lab backend의 /probe**에 한 번 보내 200/roadmap-ok를 기록한다. 실제 애플리케이션 SQL 실행은 없다.
Check 단계의 capability별 action도 확인한다. rule action만 변경하고 capability override가 Check로 남으면 Block 시험은 불완전하다. 942100 실제 key/version, condition, override action과 동일 query 로그를 묶는다. 탐지가 없으면 미완료이지 문자열을 계속 바꿔 flood하지 않는다. Block→Check 또는 삭제 후 동일 공격 query가 다시 backend 200인 복구까지 기록한다.


### Direct positive control and exact-query restoration commands
**rule 생성 전**, vm-a에서 actual BACKEND_IP를 설정하고 동일 query를 direct로 보낸다.
```bash
export BACKEND_IP='61에서 인수한 실제 private backend IPv4'
curl --max-time 10 -i --get --data-urlencode "q=' OR 1=1 --" "http://$BACKEND_IP:8080/probe"
```
200/roadmap-ok를 확인한 뒤 아래 Check→Block 시험으로 진행한다.

### 실행 단계

1. policy → Protections → Add request protection rule. Name **roadmap-sqli**, condition은 path가 **/probe**인 요청만 해당하도록 `http.request.url.path == '/probe'`로 설정한다. Rule action **Preconfigured check action**. Body inspection은 이 GET query 실습에서 필수가 아니므로 끈다.
2. Choose protection capabilities에서 **942100 (SQLi Libinjection)**을 찾아 현재 Console에 표시되는 version을 기록하고 선택한다. 숨겨진 version이나 삭제된 key를 임의로 입력하지 않는다. capability가 없으면 공식 목록과 Region 지원을 확인하고 이 단계를 중단한다.[T1]
3. 정상 요청과 실습 공격 문자열을 각각 한 번 보낸다. 실제 DB에 보내는 SQL이 아니라 테스트 HTTP query다.
```bash
curl --max-time 10 -i "$LAB_URL/probe?q=hello"
curl --max-time 10 -i --get --data-urlencode "q=' OR 1=1 --" "$LAB_URL/probe"
```
4. firewall Logging Search에서 같은 시간 `/probe`, capability key, rule/action을 확인한다. Check 단계에서 응답은 backend 200이며 악성 패턴은 탐지 로그가 있어야 한다. signature는 Oracle 내부 구현이므로 모든 문자열의 탐지를 보장하지 않는다. 로그가 없으면 정책이 연결된 LB인지, query 인코딩·condition·key/version을 확인하고 차단 전환하지 않는다.
5. 정상 요청 오탐이 없고 탐지 로그를 확보했으면 rule/capability의 action을 **roadmap-block-403**으로 변경한다. ACTIVE 후 위 2개 요청을 다시 보낸다. 응답 403과 WAF 로그를 함께 저장한다. backend access log에 동일 차단 요청이 도달하지 않았는지도 비교한다.
6. 다른 항목과 충돌하지 않도록 **본인 roadmap-sqli rule을 삭제**하고 같은 공격 query가200/roadmap-ok임을 확인한뒤74로이동한다. Check로만변경하면capability403override가남을수있으므로이워크숍정리는rule삭제를기본으로한다.


### 진행·종료 판정
설정 readback·기능시험·원복·정리를 각각 실제 응답으로 기록한다. 기대값은 미실행이며 실패/만료 시 새 mutation을 중단하고 아래 topic별 종료 단계로 반환한다.

### 동일 query 최종 복구
Check/삭제 후같은query를LB에다시보내200/roadmap-ok를확인한다.
```bash
curl --max-time 10 -i --get --data-urlencode "q=' OR 1=1 --" "$LAB_URL/probe"
```

### 공유 공개 scope 및 실제 WAF 로그 인수
[71의 scope/실제 service log 절차](00_Web_Application_Firewall_WAF.md#waf-log-correlation)를 사용한다. public mode이면 별도 승인된 네 exact WAF 경로만 추가 공개하며 `/`, `/index.txt`, `/health.saved`, `/unknown`, `/health/`는403을 유지한다. root200을 이 branch의 정상 기준으로 사용하지 않는다. 기존 승인 source/만료는 바꾸지 않는다.

### Check → Block → Check의 두 편집 위치
policy → Protections → Request protection rules → **본인 roadmap rule 행** Actions → Edit에서 rule Action을 Preconfigured check action으로 선택한다. 같은 편집의 Protection capabilities 목록에서 **942100/선택한 actual version 행**의 action override를 Check로 맞추거나 override 없음으로 rule action 상속을 명시한다. 현재 UI가 capability 행의 Change action으로 노출하면 그 exact 행에서 같은 값을 저장한다. 저장→ACTIVE→rule action과 capability override readback 후 Check 요청을 수행한다. Block 단계는 동일 두 위치를 roadmap-block-403으로 맞춘 뒤 재조회하고, 복구는 다시 Check로 두 위치를 돌려 readback한다. 현재 UI에서 둘 중 하나를 확인할 수 없으면 시험을 중단하며 default action을 넓히지 않는다.


### 번호 순서로 완료하는 원복·정리

7. **Request Protection Rules 종료 및 실패 반환.** 본인roadmap-sqli rule 삭제→ACTIVE 목록부재→동일 SQLi query200/roadmap-ok를 확인한다. 실패하면 rule/capability 두 편집위치를Check로 맞춰재조회하고 삭제를owner와완료한다.72 action은74/75에유지한다.

실제 정리/원복 response, exact resource 또는rule ID,완료UTC와남은owner/expiry를비밀없는개인실행기록에남긴다.위의guarded복구가이미완료된경우중복mutation하지않는다.

## 결과검증

**기대 패턴(미실행):** 정상 200; 탐지되는 SQLi query는 Check에서 200+탐지 로그, Block에서 403+차단 로그. 단지 HTTP 403만 받은 것으로 SQLi capability가 동작했다고 단정하지 않고 key/action을 로그로 결합한다.

실행 기록(미기입): 실행 UTC __ / 실제 resource OCID __ / 요청·SQL 실제 응답 __ / 공식 조건 차이 __ / 정리 완료 UTC __.

## 문제해결

403이 정상 요청에도 발생하면 default action과 74 access rule을 확인한다. 탐지되지 않은 요청을 성공으로 기록하지 않는다. false positive는 이 실습 경로에 한정한 exclusion을 검토하되 전체 보호를 끄지 않는다.

## 리소스정리

본인 roadmap-sqli rule을 삭제하고 동일 공격 query의200복구를 확인한다. **72의403/429 action은74/75가사용하므로75까지삭제하지않는다.** firewall/policy/LB도75까지유지한다.

### 소유자별 종료와 exact 대상 재조회
73 owner: Protections의 exact roadmap-sqli rule Delete/confirm → ACTIVE에서 목록 부재 → same SQLi query200을 읽는다. capability/rule 부재와 정상query200을 각각 기록하고72 action은 유지한다.

실제 정리 UTC __ / exact resource 또는 rule ID __ / 삭제 후 조회 응답 __ / 잔존 resource·만료·owner __. 승인 없는 잔존 비용은 다음 실습을 이유로 연장하지 않는다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 개념은 해당 생성·변경·조회 단계에서 입력 선택과 실제 결과 해석을 위해 사용하며, 공식 예시는 실제 실습 실행 결과가 아닙니다.

- [39] [Overview of Web Application Firewall — Request Protection Rules](https://docs.oracle.com/en-us/iaas/Content/WAF/Concepts/overview.htm)
  - 확인 범위: Request Protection Rules.

### 실습 task 본문 근거

- [T1] [Protections / create_request_control.htm](https://docs.oracle.com/en-us/iaas/Content/WAF/Protections/create_request_control.htm) — 본문 절차·입력 필드·지원 조건을 읽어 실습에 적용.
- [T2] [Protections / protections_management.htm](https://docs.oracle.com/en-us/iaas/Content/WAF/Protections/protections_management.htm) — 본문 절차·입력 필드·지원 조건을 읽어 실습에 적용.
- [T3] [Concepts / understanding_conditions.htm](https://docs.oracle.com/en-us/iaas/Content/WAF/Concepts/understanding_conditions.htm) — 본문 절차·입력 필드·지원 조건을 읽어 실습에 적용.

