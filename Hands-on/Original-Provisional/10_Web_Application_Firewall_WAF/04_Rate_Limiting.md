> **기존 실습 초안 — 미검증**: 원래 93개 문서의 절차를 보존한 자료입니다. 새 튜토리얼의 품질 승인이나 OCI 실행 검증을 뜻하지 않습니다.

# Rate Limiting

> 문서 작성 검토와 실행 검증은 별개입니다. OCI 실행은 **USER_EXECUTION_PENDING**이며 사용자 캡처는 별도 자료입니다. 미실행 자체는 문서 작성 승인 차단 사유가 아닙니다. 실제 결과 필드는 실행자가 채웁니다.

> 실행용 한국어 교육 문서. OCI 리소스 생성·명령 실행 결과는 아직 검증하지 않았으며 모든 기대 패턴은 미실행이다.

## 학습목표

- /rate의 사전 200을 확보하고 10 requests/60 seconds, action duration 60 seconds의 단일 path rule을 만든다.
- 같은 client에서 최대 15회 코드·curl exit를 저장하고 실제 429 로그와 /health 200을 대조하며 마지막 요청 65초 후 /rate 회복을 확인한다.
- rule/action과 WAF 자원을 정리하고 모든 등록 backend의 원본 config를 복구하여 direct/LB health 200을 확인한 뒤 app-only consumer까지 owner61에 반환한다.

실제 OCI 요청·응답/SQL·정리 증거는 실행자가 기록한다. 아래 기대값은 실측값이 아니다.

## 대응 Roadmap 항목

- 항목명: **Rate Limiting**
- Roadmap 경로: **Web Application Firewall (WAF) → Rate Limiting**
- Node ID: `service-waf-4`
- 상위 항목: **Web Application Firewall (WAF)** (`service-waf-0`)
- 문서 범위: 이 항목의 개념, 실제 설정 절차와 학습자 실행 검증.
- [전체 항목 목록](../../../Docs/README.md)

## 사전준비

**시작 조건:** 74 deny rule 삭제·/health200;72의429 action 유지

- [WAF Policies](01_WAF_Policies.md#waf-actions) — 429 action 생성 완료.
- [Access Control](../../../Docs/10_Web_Application_Firewall_WAF/03_Access_Control.md) — deny rule 삭제 및 /health 200 확인. 같은 LB/firewall/log 유지.

### 준비 완료 출력과 인수 확인

| 인수 항목 | 실제 출력·출처 | 다음 작업의 사용 위치 |
|---|---|---|
| 선행 완료 상태 | 74 deny rule 삭제·/health200;72의429 action 유지 | 첫 변경 전 재조회 |
| 비밀 없는 실행 입력 | actual policy/firewall/log OCID, LAB_URL, 모든backend saved config, owner61 consumer registry/expiry | 아래 numbered Console/명령 단계 |
| 반환/후속 인계 | 전체backend 원본/health-only 복구와71 리소스정리·app-only/LB 소비자 반환 | 종료 UTC 이전 exact owner 반환 |

실제 Compartment/Region은 승인 Console에서 읽고 동일 identity/Region을 고정한다. VCN client에서 `command -v curl`과 승인된 backend sudo/nginx 권한을 확인한다. placeholder·명칭만 인수한 상태는 준비 완료가 아니다. 비밀번호·키·확인 token은 출력표에 적지 않는다.

## 권한

관리자는 전용 Compartment에 한정하여 아래 정책을 검토·부여한다. 기존 LB 연결 및 로그 활성화 권한도 필요하다. broad `manage all-resources`로 우회하지 않는다.
```text
Allow group <LabGroup> to manage waf-policy in compartment <LabCompartment>
Allow group <LabGroup> to manage web-app-firewall in compartment <LabCompartment>
Allow group <LabGroup> to use waf-network-address-list in compartment <LabCompartment>
Allow group <LabGroup> to use load-balancers in compartment <LabCompartment>
Allow group <LabGroup> to manage log-groups in compartment <LabCompartment>
Allow group <LabGroup> to manage log-content in compartment <LabCompartment>
```
공식 Before You Begin은 `waas-family`도 열거하지만 이 실습은 Edge WAAS를 생성하지 않는다. regional `waf-policy`와 `web-app-firewall` 및 실제 API에 필요한 LB 권한을 관리자와 확인한다. 그룹명에는 필요하면 Identity domain 접두사를 붙인다.

Rate rule/action 저장·제거=UpdateWebAppFirewallPolicy(WAF_POLICY_UPDATE), 확인=WAF_POLICY_READ, 로그=LOG_CONTENT_READ. 트랙 최종 firewall 삭제에는 WEB_APP_FIREWALL_DELETE+WAF_POLICY_DETACH+LOAD_BALANCER_UPDATE, policy 삭제=WAF_POLICY_DELETE가 필요하다. backend config 원복은 승인 OS 계정 권한이다.

operation 근거: [공식 권한/작업 본문](https://docs.oracle.com/iaas/Content/Identity/Reference/wafpolicyreference_topic-Details_for_WAF_Policies.htm). family grant는 minimum-permission 정책으로 주장하지 않는다. 실행 전에 관리자 승인된 identity/domain·실제 Compartment scope를 확인한다.

## 비용및주의사항

동일 /rate rule의 제한 시험은 순차15회이며 flood·병렬 부하를 만들지 않는다. 마지막 요청 후65초 대기, 정상 요청과 로그 관찰10분, 모든 backend 원본 복구 시간을71 lease 안에 확보한다. 현재 WAF 요청/Logging 및 유지 중인 sole LB 비용을 확인한다. 429가15회 내 없으면 무승인 반복 없이 NOT_VERIFIED로 종료한다. 선택 app-only76/69와 LB78/79 반환까지 비용 owner와 종료 UTC를 기록하며 그 이유로 새 LB를 만들지 않는다.

## 실습구성

### 워크숍 순서와 입력값

**진행 순서:** 적용 전 /rate200 → 제한15회 코드저장 → 429로그 →65초 후200 → 모든backend 원복 →71자원정리

**선택 이유:** bounded 요청으로 비용/부하를 제한하고 /health를 control로 유지한다.

| 입력 | 값 또는 정확한 출처 | 실행 위치 | 생성 전/후 | 이유 |
|---|---|---|---|---|
| LAB_URL | 71실제LB URL | vm-a guest | 전 | 동일client/source |
| requests/period/duration | 10 / 60 / 60 | Console | 전 | 짧은비파괴실험 |
| savedconfig | /etc/nginx/conf.d/oci-lab.conf.roadmap-saved | 모든 backend guest | 전 | 원본복구 |


값이 다른 터미널에 자동 전파되지 않습니다. Console readback으로 비밀 없는 값만 인계하며, placeholder는 실제값으로 교체한 뒤 실행합니다. 새 리소스 생성 전 이름 충돌·소유자·최대 비용·종료 UTC를 확인하고 선택하지 않은 분기는 **NOT_SELECTED**로 기록합니다.


단일 client IP, /rate 경로에 10 requests/60 seconds, 차단 duration 60 seconds. 20회 이하만 실행한다.


### 이번 실습의 생성·재사용·앱 경계

| 구분 | 이번 주제의 실제 범위 |
|---|---|
| 새로 생성 | /rate rate rule 하나 |
| 재사용 | 72 action 및동일LB/nginx 앱/log |
| 앱/작업부하 및 변경 | bounded15요청·65초 회복·모든backend 원본config 반환 |
| 실행/반환 순서 | 74 deny rule 삭제·/health200;72의429 action 유지 → 전체backend 원본/health-only 복구와71 리소스정리·app-only/LB 소비자 반환 |

새로 생성하지 않는 항목을선행완료로가장하지않는다.위표의실제OCID/주소와진행순서를첫mutation전에확인한다.

## 실습절차

### rate 제한·회복을 완주하기
같은 VCN client에서 `export LAB_URL='http://61의실제LBIPv4'`를 준비하고 적용 전 `/rate`의 200/roadmap-ok와 `/health` 200/ok를 기록한다. 동일 client/source IP를 유지한다. 15회 시험의 HTTP code 배열을 저장하고 실제 429가 하나도 없으면 실패로 명시한다. 최대 요청 시간으로 60초 period를 넘길 수 있으므로 실패/timeout도 배열에 남긴다.
65초 회복 대기는 마지막 요청 후 foreground `sleep 65` 후 `curl --max-time 10 -i "$LAB_URL/rate"`와 `/health`로 수행한다. rule 삭제 후도 정상 body를 읽는다. backend 전부의 config 원복→direct health→LB health OK→LB `/health` 순으로 복구 검증 후 firewall/policy를 삭제한다. config backup은 원복 검증 전 지우지 않는다.

### 변경 전 실제 client control
vm-a guest에서 actual LAB_URL을71에서 인수해 설정한다. 다음 요청의 실제 status/body를 기록한 뒤만 아래 Console 변경으로 진행한다.
```bash
curl --fail --max-time 10 -i "$LAB_URL/rate"
curl --fail --max-time 10 -i "$LAB_URL/health"
```

### 실행 단계

1. policy → Rate limiting → Add rate limiting rule. Name **roadmap-rate**, condition **`http.request.url.path == '/rate'`**, Requests limit **10**, Period in seconds **60**, Action duration in seconds **60**, Rule action **roadmap-rate-429**를 입력한다.[T1] 정상 /health를 제한하지 않도록 path condition을 둔다.
2. ACTIVE 후 같은 단말에서 아래 최대 15회를 순차 요청한다. client IP별 평가이므로 VPN·NAT 공유 사용자의 트래픽을 결과에서 구분한다.
```bash
test ! -e rate-codes.txt || { printf 'existing evidence: stop\n'; exit 1; }
for n in $(seq 1 15); do
  if code=$(curl --max-time 5 -s -o /dev/null -w '%{http_code}' "$LAB_URL/rate"); then rc=0; else rc=$?; fi
  printf '%s %s curl_exit=%s\n' "$n" "$code" "$rc" | tee -a rate-codes.txt
  sleep 0.2
done
curl --max-time 5 -s -o /dev/null -w '%{http_code}
' "$LAB_URL/health"
```
3. WAF 로그의 rate limiting rule, client IP, 429를 확인한다. enforcement 분산/전파 때문에 정확히 11번째에 차단된다고 단정하지 않는다. 15회 내 미차단이면 반복 flood하지 말고 rule 연결·condition·설정·동일 IP를 점검하고 실패로 기록한다.
4. 마지막 요청에서 아래 foreground 대기 후 실제 status/body를 다시 읽는다. /rate200/roadmap-ok와 /health200/ok 둘다 확인한다.
```bash
sleep 65
curl --max-time 10 -i "$LAB_URL/rate"
curl --max-time 10 -i "$LAB_URL/health"
```
5. rule 삭제 후 /rate 200을 확인하고 WAF 공유 트랙을 정리한다.


### 진행·종료 판정
설정 readback·기능시험·원복·정리를 각각 실제 응답으로 기록한다. 기대값은 미실행이며 실패/만료 시 새 mutation을 중단하고 아래 topic별 종료 단계로 반환한다.

### Exact config restoration and app-only retention
모든 등록 backend guest에서 저장된 원본을 확인한 뒤 복구한다.
```bash
set -euo pipefail
sudo test -f /etc/nginx/conf.d/oci-lab.conf.roadmap-saved
sudo cp -p /etc/nginx/conf.d/oci-lab.conf.roadmap-saved /etc/nginx/conf.d/oci-lab.conf
sudo nginx -t
sudo systemctl reload nginx
curl --fail --max-time 10 http://127.0.0.1:8080/health
```
client에서 LB /health200/ok와 backend health OK 및 **아래 public health-only acceptance(해당 mode)**까지 모두 확인한 뒤만 backup파일을 owner승인으로 제거한다. 복구 실패 시 backup을 남기고 owner61에 반환 실패를 알린다. **LB consumer78/79뿐 아니라 app-only76/69**의 반환까지61 앱을 유지하며 새 LB생성을 강제하지 않는다.

### 공유 공개 scope 및 실제 WAF 로그 인수
[71의 scope/실제 service log 절차](00_Web_Application_Firewall_WAF.md#waf-log-correlation)를 사용한다. public mode이면 별도 승인된 네 exact WAF 경로만 추가 공개하며 `/`, `/index.txt`, `/health.saved`, `/unknown`, `/health/`는403을 유지한다. root200을 이 branch의 정상 기준으로 사용하지 않는다. 기존 승인 source/만료는 바꾸지 않는다.

### public 원본 복구의 health-only acceptance
saved original을 모든 등록 backend에 복원한 뒤 public mode에서는 승인 외부 client에서 다음을 수행한다. `/probe` 등 WAF extension의200이 남으면 복구 완료가 아니다. 하나라도 실패하면 saved 파일을 제거하지 않고61 owner에게 RESTORE_FAILED로 반환한다. private mode는61의 원래 private acceptance를 따른다.
```bash
set -euo pipefail
test "$(curl --fail --max-time 10 "$LAB_URL/health")" = ok
for path in / /index.txt /health.saved /unknown /health/ /probe /deny /denied /rate; do
  test "$(curl --max-time 10 -o /dev/null -w '%{http_code}' "$LAB_URL$path")" = 403
done
```
79 실행 전 이 health-only 반환을 완료하고 추가 공개 승인 lease를 종료한다.


### 번호 순서로 완료하는 원복·정리

6. **Rate Limiting 종료 및 실패 반환.** 본인rate rule삭제 및same /rate200→모든등록backend saved config복구→direct/LBhealth→public일때health-only403 acceptance 순서로확인한다. 성공뒤만backup제거하고73/74 잔존rule→미참조72actions→71firewall/policy/log/group를각owner가정리한다.61 registry의app-only76/69 및선택LB78/79 반환전앱/LB삭제금지다.

실제 정리/원복 response, exact resource 또는rule ID,완료UTC와남은owner/expiry를비밀없는개인실행기록에남긴다.위의guarded복구가이미완료된경우중복mutation하지않는다.

## 결과검증

**기대 패턴(미실행):** 초반 /rate 200, 제한 이후 일부 429, /health 200, duration 종료 후 /rate 200. 실제 요청 횟수와 응답 배열, log rule/client IP를 기록한다.

실행 기록(미기입): 실행 UTC __ / 실제 resource OCID __ / 요청·SQL 실제 응답 __ / 공식 조건 차이 __ / 정리 완료 UTC __.

## 문제해결

429가 없는 경우 73 Protection보다 먼저 rate rule이 평가됨을 확인한다. 공유 NAT 환경이면 한 사용자만의 한도라고 해석하지 않는다. client header로 IP를 조작하는 테스트는 이 실습에 포함하지 않는다.

## 리소스정리

위 Exact config restoration 단계가 성공했다면 반복 cp하지 않고 config 원복/hash 및 `/health` 200/ok readback을 인수한다. 실패/중단 때만 저장 원본이 존재하는지 확인해 그 guarded 복구 단계를 수행한다. 본인 rate/sqli/deny 및 임시health-Check rule 잔존을 확인·삭제 → **미참조403/429 action 둘다** 삭제 → firewall → policy → WAF log/log group 순서로 삭제·각목록에서exact대상부재를재조회한다. 선택76(app-only DNS HTTP)/69(NLB HTTP health)는 app을, 선택78~79는 LB/backend를 예약 종료 UTC까지 유지하고 모두 owner61 registry에 반환한다. 불필요한 public LB를 방치하지 않는다.

### 소유자별 종료와 exact 대상 재조회
75 owner는 자체 rate rule 삭제 후 same /rate200을 기록하고73/74 rule 잔존 및72 action 미참조를 확인한다. 71 owner에게 exact firewall/policy/log 삭제 readback을 인계하며61 owner는 selected app/LB consumers 전부 반환 뒤만 최종 삭제한다.

실제 정리 UTC __ / exact resource 또는 rule ID __ / 삭제 후 조회 응답 __ / 잔존 resource·만료·owner __. 승인 없는 잔존 비용은 다음 실습을 이유로 연장하지 않는다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 개념은 해당 생성·변경·조회 단계에서 입력 선택과 실제 결과 해석을 위해 사용하며, 공식 예시는 실제 실습 실행 결과가 아닙니다.

- [39] [Overview of Web Application Firewall — Rate Limiting](https://docs.oracle.com/en-us/iaas/Content/WAF/Concepts/overview.htm)
  - 확인 범위: Rate Limiting.

### 실습 task 본문 근거

- [T1] [RateLimiting / create_rate_limiting_rule.htm](https://docs.oracle.com/en-us/iaas/Content/WAF/RateLimiting/create_rate_limiting_rule.htm) — 본문 절차·입력 필드·지원 조건을 읽어 실습에 적용.
- [T2] [RateLimiting / rate_limiting_rule_management.htm](https://docs.oracle.com/en-us/iaas/Content/WAF/RateLimiting/rate_limiting_rule_management.htm) — 본문 절차·입력 필드·지원 조건을 읽어 실습에 적용.

