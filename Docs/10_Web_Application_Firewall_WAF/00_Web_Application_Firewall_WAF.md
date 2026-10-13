# Web Application Firewall (WAF)

> 문서 작성 검토와 실행 검증은 별개입니다. OCI 실행은 **USER_EXECUTION_PENDING**이며 사용자 캡처는 별도 자료입니다. 미실행 자체는 문서 작성 승인 차단 사유가 아닙니다. 실제 결과 필드는 실행자가 채웁니다.

> 실행용 한국어 교육 문서. OCI 리소스 생성·명령 실행 결과는 아직 검증하지 않았으며 모든 기대 패턴은 미실행이다.

## 학습목표

- 61의 동일 nginx :8080 앱을 인수하고 `/health` 200/ok 및 네 시험 경로의 200/roadmap-ok를 vm-a에서 직접·LB :80 경유로 확인한다.
- WAF policy/firewall 각 1개를 생성하여 실제 LB OCID에 연결하고 Check rule 요청의 URI·action·시각을 WAF service log에서 대조한다.
- 임시 Check rule을 제거하여 /health 200을 재확인하고 72~75에 인계한 후 config·firewall·policy·log를 소유자 순서로 정리한다.

실제 OCI 요청·응답/SQL·정리 증거는 실행자가 기록한다. 아래 기대값은 실측값이 아니다.

## 대응 Roadmap 항목

- 항목명: **Web Application Firewall (WAF)**
- Roadmap 경로: **Web Application Firewall (WAF)**
- Node ID: `service-waf-0`
- 상위 항목: 없음 — 최상위 학습 항목
- 문서 범위: 이 항목의 개념, 실제 설정 절차와 학습자 실행 검증.
- [전체 항목 목록](../README.md)

## 사전준비

**시작 조건:** 61 HTTP listener80/backend8080 및 직접/LB /health200/ok 완료; private mode 기본

- [Load Balancer](../08_Load_Balancer/00_Load_Balancer.md) — 완료 단계: HTTP listener :80, backend set health OK, 같은 LB IP의 GET /health가 200인 상태까지 완료하고 LB/backend를 유지한다. TCP listener 및 NLB는 사용할 수 없다. 본 실습은 HTTP 전달 경로 확인용이며 로그인·개인정보를 보내지 않는다.

### 준비 완료 출력과 인수 확인

| 인수 항목 | 실제 출력·출처 | 다음 작업의 사용 위치 |
|---|---|---|
| 선행 완료 상태 | 61 HTTP listener80/backend8080 및 직접/LB /health200/ok 완료; private mode 기본 | 첫 변경 전 재조회 |
| 비밀 없는 실행 입력 | actual LB OCID/IP/mode, 모든 backend IP, CLIENT_CIDR, config/health 원본, owner61/expiry | 아래 numbered Console/명령 단계 |
| 반환/후속 인계 | 71 생성 후 ACTIVE policy/firewall OCID 및 service log/group OCID를72~75에 인계 | 종료 UTC 이전 exact owner 반환 |

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

실제 operation별 권한: CreateWebAppFirewallPolicy=WAF_POLICY_CREATE; CreateWebAppFirewall=WEB_APP_FIREWALL_CREATE+WAF_POLICY_ATTACH+LOAD_BALANCER_UPDATE; DeleteWebAppFirewall=WEB_APP_FIREWALL_DELETE+WAF_POLICY_DETACH+LOAD_BALANCER_UPDATE. Start/StopLogging=WEB_APP_FIREWALL_UPDATE, GetLogging=WEB_APP_FIREWALL_READ. work request 조회는 공식 표의 WAF_POLICY_INSPECT+WEB_APP_FIREWALL_INSPECT+WAF_NETWORK_ADDRESS_LIST_INSPECT를 별도 검토한다. 위 use LB 정책의 LOAD_BALANCER_UPDATE 포함 여부를 network owner가 확인한다. log 생성은 LOG_GROUP_CREATE, search는 LOG_CONTENT_READ다.

operation 근거: [공식 권한/작업 본문](https://docs.oracle.com/iaas/Content/Identity/Reference/wafpolicyreference_topic-Details_for_WAF_Policies.htm). family grant는 minimum-permission 정책으로 주장하지 않는다. 실행 전에 관리자 승인된 identity/domain·실제 Compartment scope를 확인한다.

## 비용및주의사항

이 단계는 WAF policy 1개/firewall 1개와 전용 service log/group을 새로 만들고 61의 sole LB/nginx backend를 재사용한다. 현재 계약의 WAF 정책·요청 처리, Logging 수집·저장·검색과 유지 중인 LB 대역폭/백엔드 비용을 승인한다. 각 기능 시험은 최대20회·로그 관찰10분이며 승인 종료 UTC를 넘겨 72~75를 기다리지 않는다. public mode는 비민감 exact 시험 경로와 source/만료의 추가 승인이 있어야 한다. 무료 요청·무료 로그 보존을 보장하지 않는다.

## 실습구성

### 워크숍 순서와 입력값

**진행 순서:** 61 실제 앱 인수 → 모든 backend 시험경로 확장 → WAF policy/firewall 연결 → Check 로그 입증 → 72 인계

**선택 이유:** nginx :8080과 LB :80을 유지하며 보안제어 전 앱200을 확보하여 앱404를 WAF차단으로 오인하지 않는다.

| 입력 | 값 또는 정확한 출처 | 실행 위치 | 생성 전/후 | 이유 |
|---|---|---|---|---|
| LB_IP/LAB_URL | 61 실제 LB주소 / http://$LB_IP | vm-a guest | 후 | 같은 soleLB 확인 |
| BACKEND_IP | 61 private-vm 실제IPv4 | vm-a→backend | 전 | direct8080 positivecontrol |
| WAF_POLICY_OCID/FIREWALL_OCID | 71생성후Console readback | Console | 후 | 이름 아닌실제 연결 확인 |


값이 다른 터미널에 자동 전파되지 않습니다. Console readback으로 비밀 없는 값만 인계하며, placeholder는 실제값으로 교체한 뒤 실행합니다. 새 리소스 생성 전 이름 충돌·소유자·최대 비용·종료 UTC를 확인하고 선택하지 않은 분기는 **NOT_SELECTED**로 기록합니다.


새 WAF policy 1개와 firewall 1개. 기존 LB/backend 재사용. firewall은 policy와 LB를 연결하는 논리 리소스이지 별도 서버가 아니다.

<a id="waf-setup"></a>


### 이번 실습의 생성·재사용·앱 경계

| 구분 | 이번 주제의 실제 범위 |
|---|---|
| 새로 생성 | WAF policy/firewall/service log/group |
| 재사용 | 61 sole HTTP LB·nginx 앱·VCN client |
| 앱/작업부하 및 변경 | 모든 backend 전용nginx config에 exact 시험경로 추가;75에서 원본복구 |
| 실행/반환 순서 | 61 HTTP listener80/backend8080 및 직접/LB /health200/ok 완료; private mode 기본 → 71 생성 후 ACTIVE policy/firewall OCID 및 service log/group OCID를72~75에 인계 |

새로 생성하지 않는 항목을선행완료로가장하지않는다.위표의실제OCID/주소와진행순서를첫mutation전에확인한다.

## 실습절차

### 인수·복구 gate (owner 61 → 71 → 75)
61 owner에게 Oracle Linux/private backend, `/etc/nginx/conf.d/oci-lab.conf`, `/var/www/oci-lab/{index.txt,health}`, HTTP :8080, listener :80, health `/health`=200/`ok`를 인수받는다. 61의 최종 파일·port·health body와 [실제 생성 전 mode 선택](../08_Load_Balancer/00_Load_Balancer.md#lb-mode-before-create)을 다시 읽는다. backend=private-vm, client=vm-a guest이고 CLI는 Cloud Shell이며 guest sudo/HTTP와 섞지 않는다. `LB_MODE=private`를71~75 기본으로 유지한다. 79를 위해 동일 sole LB를 public으로 처음부터 선택했다면61은 공개 HTTP `/health`만 승인하는 예외다. `/probe`, `/deny`, `/denied`, `/rate` WAF 시험을 public으로 수행할 별도 비민감 경로·source·종료시각 승인이 없으면71~75는 BLOCKED_PUBLIC_PATH_SCOPE로 남긴다. 추가 private/public LB를 만들지 않는다. backend는 기본 1대이며 승인된 2대인 경우 extension과 복구를 **모든 등록 backend**에 적용한다. 한쪽만 확장하면 간헐적 404를 WAF 판정으로 오해한다.
nginx config 적용 후 `nginx -t` 실패 시 saved config를 즉시 복구하고 다시 test/reload한다. 실제 LB 경로로 `/probe`, `/deny`, `/denied`, `/rate` 모두 200/roadmap-ok인지 확인한 후 policy를 연결한다. 정상 200은 policy/firewall ACTIVE와 service log readback을 함께 충족해야 완료다. WAF attach는 LB의 기존 firewall 유무를 먼저 확인하고 이미 연결된 운영 firewall을 교체하지 않는다.

### 실행 단계

1. **61의 실제 HTTP 앱을 먼저 확인한다.** 선행 61의 기본은 private HTTP LB와 nginx :8080이다. 따라서 요청 명령은 **같은 VCN의 승인된 private client**에서 실행한다. public IP를 가정하지 않는다. backend App이 `/health`에 `ok`를 반환하는지 61의 direct 및 LB 경로로 확인한다. WAF는 HTTP Load Balancer에 연결되며 NLB/TCP listener는 제외한다.
2. 61의 nginx는 `/probe`, `/deny`, `/denied`, `/rate`가 기본 404일 수 있다. WAF 응답과 앱 404를 혼동하지 않도록 **61의 전용 실습 backend**에서 아래 좁은 extension을 적용한다. 기존 운영 nginx 파일은 수정하지 않는다. 승인된 lab config만 최초 상태를 backup하고 이후 75에서 복구한다. :8080을 이미 쓰는 nginx 옆에 Python 서버를 같은 포트로 시작하지 않는다.
```bash
# 61의 전용 backend에서만. backup 파일이 이미 있으면 보존하고 내용 확인 후 진행한다.
set -euo pipefail
sudo test -f /etc/nginx/conf.d/oci-lab.conf
if sudo test -e /etc/nginx/conf.d/oci-lab.conf.roadmap-saved || sudo test -L /etc/nginx/conf.d/oci-lab.conf.roadmap-saved; then
  printf '%s\n' '기존 backup이 있으므로 중단: owner가 기존 상태를 확인해야 합니다.' >&2
  exit 1
fi
restore_original() {
  sudo cp -p /etc/nginx/conf.d/oci-lab.conf.roadmap-saved /etc/nginx/conf.d/oci-lab.conf &&
    sudo nginx -t && sudo systemctl reload nginx &&
    curl --fail --max-time 10 http://127.0.0.1:8080/health
}
rollback_extension() {
  trap - EXIT HUP INT TERM
  if ! restore_original; then
    printf 'RESTORE_FAILED: owner must recover exact saved config; preserve backup\n' >&2
  fi
}
sudo cp -p /etc/nginx/conf.d/oci-lab.conf /etc/nginx/conf.d/oci-lab.conf.roadmap-saved
trap rollback_extension EXIT
trap 'exit 129' HUP
trap 'exit 130' INT
trap 'exit 143' TERM
sudo tee /etc/nginx/conf.d/oci-lab.conf >/dev/null <<'NGINX'
server {
    listen 8080;
    server_name _;
    root /var/www/oci-lab;
    # WAF extension serves only the five approved exact paths in either mode.
    location / { return 403; }
    location = /health { default_type text/plain; try_files /health =404; }
    location = /probe { default_type text/plain; return 200 'roadmap-ok'; }
    location = /deny { default_type text/plain; return 200 'roadmap-ok'; }
    location = /denied { default_type text/plain; return 200 'roadmap-ok'; }
    location = /rate { default_type text/plain; return 200 'roadmap-ok'; }
}
NGINX
sudo nginx -t
sudo systemctl reload nginx
test "$(curl --fail --max-time 10 http://127.0.0.1:8080/health)" = ok
for path in probe deny denied rate; do
  test "$(curl --fail --max-time 10 "http://127.0.0.1:8080/$path")" = roadmap-ok
done
for path in / /index.txt /health.saved /unknown /health/; do
  test "$(curl --max-time 10 -o /dev/null -w '%{http_code}' "http://127.0.0.1:8080$path")" = 403
done
trap - EXIT HUP INT TERM
# Successful extension remains leased to 75; preserve saved original until confirmed return.
```
LB의 backend port **8080**, HTTP health port **8080**, path **/health**, expected status **200**, listener **HTTP :80**과 backend set health **OK**를 유지한다. client에서 실제 private LB IPv4를 입력한다. 외부 Health Checks를 목표로 **생성 전에 별도 public LB mode를 승인해 61에서 선택한 경우에만** 그 실제 public IPv4를 사용한다. 이 문서에서 LB를 두 번째로 생성하지 않는다.
```bash
export LB_IP='61에서 생성한 실제 LB IPv4 (기본 private)'
export LAB_URL="http://$LB_IP"
curl --max-time 10 -i "$LAB_URL/health"
curl --max-time 10 -i "$LAB_URL/probe?q=hello"
```
vm-a에서 다음 모든 client 경로가200/roadmap-ok인지 기록한 뒤만3단계 policy를 만든다.
```bash
for path in probe deny denied rate; do
  curl --fail --max-time 10 -i "$LAB_URL/$path"
done
```

### 공개 scope 재검증과 즉시 owner 반환
policy 연결 **전** 기본 private mode는같은VCN client에서다음을수행한다.public mode만승인외부client에서수행하며public 추가경로승인이없으면extension을실행하지않는다. 각 backend의 local 성공뿐 아니라 실제 LB body도 exact 비교한다. client/LB 실패가 하나라도 있으면 policy를 생성하지 말고 각 backend에서 위 `restore_original`과 동일 saved→test→reload→health 절차를 즉시 수행하여61 owner에 반환한다. 복구 실패는 RESTORE_FAILED이며 backup을 지우지 않는다.
```bash
set -euo pipefail
for path in probe deny denied rate; do
  test "$(curl --fail --max-time 10 "$LAB_URL/$path")" = roadmap-ok
done
for path in / /index.txt /health.saved /unknown /health/; do
  test "$(curl --max-time 10 -o /dev/null -w '%{http_code}' "$LAB_URL$path")" = 403
done
```
3. Identity & Security → Web Application Firewall → Policies → Create WAF policy. Name **roadmap-waf**, Compartment **실습 Compartment**. policy가 **ACTIVE**일 때만 수정한다. 정책 생성만으로 요청이 보호되지 않는다.[T1]
4. policy → Firewalls → Add firewalls. Firewall name **roadmap-fw**, Create in compartment **실습 Compartment**, Load balancer compartment **61 LB Compartment**, Load balancer **61의 실제 LB**를 선택한다. 생성 후 firewall OCID와 policy OCID, LB OCID를 별도 기록한다.[T3]
5. `/health` 요청3회는 vm-a의 `for n in 1 2 3; do curl --max-time 10 -i "$LAB_URL/health"; done`으로 실제 전송한다. firewall details → Monitoring → Log에서 제공되는 WAF service log를 Enable logging하고 전용 log group **roadmap-waf-logs**, 로그 이름 **roadmap-waf-log**, 보존 기간은 허용 최솟값으로 설정한다. 활성화 완료 후 policy → Access control → Request access rules → Add rule에서 임시 rule **roadmap-health-log-check**, condition `http.request.url.path == '/health'`, action **Preconfigured check action**을 저장하고 ACTIVE를 재조회한다. `/health`를 3회 요청하고 Logging Search에서 방금 시간·URI·rule/action을 조회한다. rule 미적용 정상 요청의 로그 생성을 보장하지 않기 때문에 Check rule로 관측 대상을 명시한다. 로그 확인 후 이 임시 rule만 삭제하고 `/health` 200을 재검증한다. 84의 custom log와 WAF service log는 서로 다른 리소스다.
<a id="waf-log-correlation"></a>
### WAF service log의 실제 scope와 correlation
Policies → actual policy → Firewalls → actual firewall → Monitoring → Log에서 **해당 service log 링크**와 **Log group 링크**를 열어 actual log OCID/group OCID/Compartment를 복사한다.[T-LOG] Logging → Search에서 이 exact Compartment/group/log만 선택하고 요청 UTC를 포함한 최근10분 범위를 지정한다. Advanced mode의 첫 query는 다음 **scope-only** query다. placeholder를 해당 실제 OCID로 바꾼다.
```text
search "<actual compartment OCID>/<actual WAF log group OCID>/<actual WAF log OCID>" | sort by datetime desc
```
Search 결과 한 행의 JSON detail을 펼쳐 actual payload schema를 먼저 읽는다. condition input인 `http.request.url.path`를 로그 field라고 가정하지 않는다. 실제 JSON key/path와 값을 비밀 없는 작업표에 기록하고 요청 UTC·GET·URI/query·실제 client/source·적용 rule 이름·action을 대조한다.73은 capability ID/version 및 override action,74는 exact access rule,75는 rate rule/429 action도 같은 행에서 확인한다. actual schema에 필드가 없으면 추정 key로 query를 꾸미지 말고 조건을 좁히지 않은 scope-only 결과와 부족한 증거를 기록한다. 일치한 행 없음/필수 capability 정보 없음은 NOT_VERIFIED이며 최대10분 후 중단한다. 서비스 로그 query는84 custom marker query와 다르다. [SearchLogs 공식 query 형식](https://docs.oracle.com/en-us/iaas/Content/Logging/Concepts/using_the_api_searchlogs.htm)을 따른다.



6. 72~75를 진행할 동안 policy, firewall, LB/backend를 유지한다.


### 진행·종료 판정
설정 readback·기능시험·원복·정리를 각각 실제 응답으로 기록한다. 기대값은 미실행이며 실패/만료 시 새 mutation을 중단하고 아래 topic별 종료 단계로 반환한다.

### 번호 순서로 완료하는 원복·정리

7. **Web Application Firewall (WAF) 종료 및 실패 반환.** 72~75 예약 여부와 종료 UTC를 확인해 인계한다. 중단이면 임시 health Check rule 삭제→모든 backend의 saved 원본복구→direct/LB health 확인→본인firewall/policy/log/group 순서로 정리한다. exact 삭제 readback과61 app-only 소비자 반환은 아래 리소스정리를 따른다.

실제 정리/원복 response, exact resource 또는rule ID,완료UTC와남은owner/expiry를비밀없는개인실행기록에남긴다.위의guarded복구가이미완료된경우중복mutation하지않는다.

## 결과검증

GET /health의 실제 HTTP 200과 61의 ok, firewall의 LB 연결 및 ACTIVE 상태, 로그의 실제 요청 시간·URI를 함께 기록한다. **기대 패턴(미실행):** HTTP/1.1 200, /health body ok, /probe body roadmap-ok. 스크린샷만으로 트래픽 검증을 대체하지 않는다.

실행 기록(미기입): 실행 UTC __ / 실제 resource OCID __ / 요청·SQL 실제 응답 __ / 공식 조건 차이 __ / 정리 완료 UTC __.

## 문제해결

502/504는 WAF 전에 LB backend health와 :8080 NSG를 확인한다. LB 목록이 비면 Region/Compartment, HTTP listener, 기존 firewall 연결을 점검한다. 로그가 없으면 enable 상태·최근 10분 범위·log-content 읽기 권한을 확인하고 10분 이상 무한 대기하지 않는다.

## 리소스정리

시험 중단/75 완료 후 임시 roadmap-health-log-check가 남아있으면 먼저 삭제하고 정상 /health를 확인한다. firewall 연결 리소스 삭제 → policy 삭제 → service log/log group 삭제. 공유 LB/backend는 61의 리소스정리 지침과 사용자가 정한 보존 여부에 따라 별도 정리한다. backend config는75의 guarded Exact config restoration 절차를 따른다.75 복구가 완료되고 backup도 제거했다면 반복cp하지 않는다. 중단/실패시 saved 원본 존재를 먼저 확인해 같은 절차로복구한다. backup 파일은 정상 복구 확인 후 본인 실습 파일만 정리한다.

### 소유자별 종료와 exact 대상 재조회
71 owner: Firewalls 목록의 actual firewall OCID → Actions/Delete/confirm → 같은 LB의 firewall 연결 목록 부재를 확인한다. 이어 exact policy OCID 삭제·Policies 목록 부재, 본인 WAF log 삭제·Log group의 Logs 부재, empty group 삭제·Log groups 부재를 읽는다. 공유 app/LB는75 복구 및 모든61 consumer 반환 전 삭제하지 않는다.

실제 정리 UTC __ / exact resource 또는 rule ID __ / 삭제 후 조회 응답 __ / 잔존 resource·만료·owner __. 승인 없는 잔존 비용은 다음 실습을 이유로 연장하지 않는다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 개념은 해당 생성·변경·조회 단계에서 입력 선택과 실제 결과 해석을 위해 사용하며, 공식 예시는 실제 실습 실행 결과가 아닙니다.

- [39] [Overview of Web Application Firewall — Web Application Firewall (WAF)](https://docs.oracle.com/en-us/iaas/Content/WAF/Concepts/overview.htm)
  - 확인 범위: Web Application Firewall (WAF).

### 실습 task 본문 근거

- [T1] [Policies / getting_started_with_waf-policies.htm](https://docs.oracle.com/en-us/iaas/Content/WAF/Policies/getting_started_with_waf-policies.htm) — 본문 절차·입력 필드·지원 조건을 읽어 실습에 적용.
- [T2] [Policies / create_waf-policy.htm](https://docs.oracle.com/en-us/iaas/Content/WAF/Policies/create_waf-policy.htm) — 본문 절차·입력 필드·지원 조건을 읽어 실습에 적용.
- [T3] [Firewall / create_firewall.htm](https://docs.oracle.com/en-us/iaas/Content/WAF/Firewall/create_firewall.htm) — 본문 절차·입력 필드·지원 조건을 읽어 실습에 적용.
- [T4] [firewall_management / firewall_management.htm](https://docs.oracle.com/en-us/iaas/Content/WAF/Firewall/firewall_management.htm) — 관리 task 탐색용 메뉴 본문. 기술 절차의 단독 근거로 사용하지 않으며 생성은 [T3], 로그는 [T-LOG]의 실제 task 본문으로 확인했다.


- [T-CHECK-RULE] [Create request access rule](https://docs.oracle.com/en-us/iaas/Content/WAF/AccessControl/create_access_rule_request_control.htm) — /health 한정 Check로 로그 생성 후 삭제한다. 운영 rule이 아니다.
- [T-CHECK-ACTION] [Check action semantics](https://docs.oracle.com/en-us/iaas/Content/WAF/Actions/create_action.htm) — Check는진행을멈추지않고log를생성.
- [T-LOG] [firewall details Monitoring / Log enable and view](https://docs.oracle.com/en-us/iaas/Content/WAF/Firewall/list_firewall_logs.htm) — firewall details Monitoring / Log enable and view 본문을 실제 읽어 적용.
