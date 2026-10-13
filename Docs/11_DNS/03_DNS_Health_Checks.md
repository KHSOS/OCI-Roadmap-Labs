# DNS Health Checks

> 문서 작성 검토와 실행 검증은 별개입니다. OCI 실행은 **USER_EXECUTION_PENDING**이며 사용자 캡처는 별도 자료입니다. 미실행 자체는 문서 작성 승인 차단 사유가 아닙니다. 실제 결과 필드는 실행자가 채웁니다.

> 실행용 한국어 교육 문서. OCI 리소스 생성·명령 실행 결과는 아직 검증하지 않았으며 모든 기대 패턴은 미실행이다.

## 학습목표

- 61의 sole LB 생성 전 public 선택·source 제한·비용 승인을 인수하고 승인 외부 client /health 200을 확인한다. private LB면 추가 LB 없이 중단한다.
- Targets actual public IP, HTTP80 GET /health, vantage point1개인 monitor1개를 만들고 서로 다른 실제 probe timestamp의 정상 두 건을 읽는다.
- 별도 downtime 승인 시 health 파일 충돌 guard와 자동복구로 실패→직접200→LB health OK→외부 정상2건을 확인하고 monitor disable/delete 및 새 probe rule 반환을 검증한다.

실제 OCI 요청·응답/SQL·정리 증거는 실행자가 기록한다. 아래 기대값은 실측값이 아니다.

## 대응 Roadmap 항목

- 항목명: **DNS Health Checks**
- Roadmap 경로: **DNS → DNS Health Checks**
- Node ID: `sequence-8-3`
- 상위 항목: **DNS** (`sequence-8-0`)
- 문서 범위: 이 항목의 개념, 실제 설정 절차와 학습자 실행 검증.
- [전체 항목 목록](../README.md)

## 사전준비

**시작 조건:** 기본 NOT_SELECTED;SELECTED일 때만61 public Create 전 공식 outbound source 계약 승인

- [Load Balancer](../08_Load_Balancer/00_Load_Balancer.md) — **외부 probe 분기 gate:** 61의 기본 private LB는 이 실습 대상이 아니다. [61의 생성 전 mode 선택](../08_Load_Balancer/00_Load_Balancer.md#lb-mode-before-create)에서 생성 전 public HTTP LB :80 옵션을 별도 비용·공개 접근 승인한 경우 또는 이미 승인된 기존 public endpoint가 있는 경우만 진행한다. 기존 private LB를 복제하지 않는다. 승인 public endpoint가 없으면 이 분기는 BLOCKED/미선택으로 표시한다. 실제 외부 client /health 200. 73~75 차단 rule은 제거하고 외부 probe가 통과 가능한 실습 endpoint로 유지.

### 준비 완료 출력과 인수 확인

| 인수 항목 | 실제 출력·출처 | 다음 작업의 사용 위치 |
|---|---|---|
| 선행 완료 상태 | 기본 NOT_SELECTED;SELECTED일 때만61 public Create 전 공식 outbound source 계약 승인 | 첫 변경 전 재조회 |
| 비밀 없는 실행 입력 | HEALTH79_SELECTION, actual vantage name/raw hash, owner-provided IPv4 CIDR 근거/hash/유효기간, owner61/network 승인, soleLB IP | 아래 numbered Console/명령 단계 |
| 반환/후속 인계 | monitor disable/delete 및 exact 새 probe rule 반환;CIDR 입력 부재면 BLOCKED | 종료 UTC 이전 exact owner 반환 |

실제 Compartment/Region은 승인 Console에서 읽고 동일 identity/Region을 고정한다. Cloud Shell에서 `oci --version`과 `command -v python3`, 승인된 외부/client 및backend guest에서 `command -v curl`을 확인한다. REGION은승인OCI_CLI_REGION과같은실제값으로설정한다. placeholder·명칭만 인수한 상태는 준비 완료가 아니다. 비밀번호·키·확인 token은 출력표에 적지 않는다.

## 권한

관리자 승인 정책: `Allow group <LabGroup> to manage health-check-family in compartment <LabCompartment>`. 대상 LB 조회는 read load-balancers, endpoint NSG 변경이 필요하면 해당 network-security-groups 관리 권한을 별도 요청한다. public HTTP probe는 VCN 내부 health check와 별개다.

scoped manage health-check-family는 교육용 family grant이며 minimum policy로 부르지 않는다. 공식 [Health Checks API 표](https://docs.oracle.com/iaas/Content/Identity/Reference/healthcheckpolicyreference.htm)에서 CreateHttpMonitor/UpdateHttpMonitor(Disable)/DeleteHttpMonitor=HEALTH_CHECK_MONITOR_MANAGE, GetHttpMonitor=HEALTH_CHECK_MONITOR_READ, ListHttpProbeResults=HEALTH_CHECK_RESULTS_INSPECT, ListHealthChecksVantagePoints=VANTAGE_POINTS_INSPECT를 확인했다. on-demand probe를 생성하지 않으므로 ON_DEMAND_PROBE_MANAGE는 사용하지 않는다. public endpoint NSG 변경은 별도 network owner 작업이다. private LB 조회 권한만으로 public probe 접근이 생기지 않는다.

operation 근거: [공식 권한/작업 본문](https://docs.oracle.com/iaas/Content/Security/Reference/healthchecks_security.htm). family grant는 minimum-permission 정책으로 주장하지 않는다. 실행 전에 관리자 승인된 identity/domain·실제 Compartment scope를 확인한다.


목록부재 readback의 ListHttpMonitors=HEALTH_CHECK_MONITOR_INSPECT도동일monitorCompartment에승인한다. NSG새rule 추가·제거는networkowner만수행한다.

## 비용및주의사항

기본 HEALTH79_SELECTION=NOT_SELECTED이며 public 변경·monitor 생성·probe rule 추가가 없다. SELECTED 계약이 모두 승인된 경우에만 monitor1개/target1개/vantage point1개, 실제 제공 interval(목표60초), 최대10분 관찰과 별도 downtime를 승인한다. 현재 Health Checks 사용량/LB 요청·트래픽 가격을 확인하며 무료 probe를 보장하지 않는다. owner-provided 공식 outbound source CIDR 근거/expiry가 없으면61 public Create 전 BLOCKED_VANTAGE_SOURCE다. provider routing prefix나0.0.0.0/0로 대체하지 않는다.

## 실습구성

### 워크숍 순서와 입력값

**진행 순서:** 61 생성 전 public선택 승인 →실제외부200 →monitor →두 번 정상probe →선택health장애/복구 →disable/delete

**선택 이유:** 외부 probe는 private LB로 시험할 수 없다. 새로운 LB를 추가하지 않는다.

| 입력 | 값 또는 정확한 출처 | 실행 위치 | 생성 전/후 | 이유 |
|---|---|---|---|---|
| LB_MODE/LB_IP | public /61처음부터승인된soleLB실제publicIP | Console/승인외부client | 전 | private대상제외 |
| monitor/vantagepoint | roadmap-http-health /현재1개actualpoint | Console | 전 | sourceCIDR승인 |
| HEALTH/SAVED | /var/www/oci-lab/health /health.saved | backend guest | 전 | 원본충돌방지/회복 |


값이 다른 터미널에 자동 전파되지 않습니다. Console readback으로 비밀 없는 값만 인계하며, placeholder는 실제값으로 교체한 뒤 실행합니다. 새 리소스 생성 전 이름 충돌·소유자·최대 비용·종료 UTC를 확인하고 선택하지 않은 분기는 **NOT_SELECTED**로 기록합니다.


외부 vantage point에서 기존 public LB의 실제 HTTP 응답을 점검한다. 별도 server/LB 및 DNS 도메인 구매 없음.


### 이번 실습의 생성·재사용·앱 경계

| 구분 | 이번 주제의 실제 범위 |
|---|---|
| 새로 생성 | 기본없음;SELECTED일때만HTTP monitor |
| 재사용 | 사전승인된61 solepublicLB/nginx health-only |
| 앱/작업부하 및 변경 | 별도승인outage 외앱변경없음;ownerCIDR계약전public mutation 없음 |
| 실행/반환 순서 | 기본 NOT_SELECTED;SELECTED일 때만61 public Create 전 공식 outbound source 계약 승인 → monitor disable/delete 및 exact 새 probe rule 반환;CIDR 입력 부재면 BLOCKED |

새로 생성하지 않는 항목을선행완료로가장하지않는다.위표의실제OCID/주소와진행순서를첫mutation전에확인한다.

## 실습절차

### 작업 순서 체크포인트
public path acceptance→monitor fields/actual vantage CIDR→서로 다른 timestamps의 정상2건→lease guard 후 선택 outage→direct/LB/external 정상2건→disable/delete/readback 순서다. 단계 실패는 다음 생성으로 넘기지 않고 같은 target의 복구/반환으로 처리한다.

### 61 public 노출 이전 vantage 준비 — Create보다 먼저
Cloud Shell에서 승인 REGION/identity를 고정하고 다음 조회를 **61 frontend NSG / public LB Create 이전** 수행한다. 새 owner79 private directory가 충돌하면 새 승인 경로를 사용한다.
```bash
umask 077
mkdir -m 700 health79-vantage || exit 1
oci health-checks vantage-point list --all --region "$REGION" > health79-vantage/list.json || exit 1
python3 - <<'PY'
import json
rows=json.load(open('health79-vantage/list.json'))['data']
assert isinstance(rows,list) and rows
for row in rows:
 print(json.dumps(row,ensure_ascii=False))
PY
```
[공식 ListHealthChecksVantagePoints task](https://docs.oracle.com/en-us/iaas/Content/HealthChecks/Tasks/list-vantage-point.htm) · [CLI list/--all](https://docs.oracle.com/en-us/iaas/tools/oci-cli/latest/oci_cli_docs/cmdref/health-checks/vantage-point/list.html). 이 조회는 **vantage identity 목록**이며 probe-source CIDR publication API가 아니다. 공식 API 응답형은 `HealthChecksVantagePointSummary[]`이고 JSON 필드는 `displayName`, `providerName`, `name`, `geo`, `routing`이다. `addressPrefixes`/`probeCIDR`는 이 공식 schema에 없다. `routing`는 **traffic to this vantage point**의 prefix/ASN 설명으로 outbound probe source allowlist가 아니다. [API operation](https://docs.oracle.com/iaas/api/#/en/healthchecks/20180501/HealthChecksVantagePointSummary/ListHealthChecksVantagePoints) · [공식 response model](https://docs.oracle.com/en-us/iaas/tools/python/latest/api/healthchecks/models/oci.healthchecks.models.HealthChecksVantagePointSummary.html). 공식 [Network Security 요구사항](https://docs.oracle.com/en-us/iaas/Content/Security/Reference/healthchecks_security.htm#network-security)은 public internet 접근 및 SL/NSG를 요구하지만 source CIDR 목록/게시 endpoint를 지정하지 않는다. 따라서 목록 조회만으로 CIDR를 획득했다거나 공개 publication mechanism을 확인했다고 주장하지 않는다.

**기본 선택:** `HEALTH79_SELECTION=NOT_SELECTED`, 61은 `LB_MODE=private`를 유지한다. 79를 명시적으로 SELECTED로 승인한 경우만 아래 owner-provided input contract를 충족해야 한다. 입력 부재/만료/불일치는 `BLOCKED_VANTAGE_SOURCE`로 **61 public NSG 변경/public LB Create/monitor Create 이전** 중단한다. 예시 주소, provider 전체 IP 목록, routing prefix, DNS 추정값, packet 한 건 또는 응답의 미문서 필드는 대체 근거가 아니다. `0.0.0.0/0` 또는 `::/0` 확대는 금지한다.

**owner79 → network owner → owner61 사전 승인 계약 (실제 입력, 아직 미제공):**
1. owner79가 현재 identity 목록의 actual `name` 하나와 provider/display-name, Region/identity, 조회 UTC, 전체 raw row 및 파일 SHA256을 저장한다. 이름만 source 근거로 사용하지 않는다.
2. network owner가 **선택한 동일 name의 outbound HTTP probe source**임을 명시하는 현재 Oracle 공식 문서 또는 Oracle Support의 공식 확인 자료를 제공한다. source URL 또는 SR/reference ID, 원문 파일/hash, 정확한 인용, 발행/확인 UTC와 유효 종료 UTC, IPv4 CIDR 목록 및 해당 name과의 매핑을 registry에 저장한다. 공개 목록을 찾지 못한 경우 owner가 공식 확인을 받아 전달할 때까지 BLOCKED이며 SR 답변을 만들어 쓰지 않는다. 이 HTTP80/IPv4 recipe에는 IPv6 입력을 자동 적용하지 않는다.
3. owner79/network owner/owner61이 source provenance, CIDR별 범위, TCP80, 비민감 `/health`만, sole LB/비용/expiry, 선택 name 하나, 변경·복구·정리 담당자에 명시적으로 승인 서명/UTC를 기록한다. source validity는 세션 expiry 전체를 포함해야 한다. `HEALTH79_SELECTION=SELECTED`는 이 계약 승인 후에만 기록한다.
4. 첫 public mutation 전 owner61이 계약 원문/hash와 source CIDR 목록을 read back하여 승인값과 정확히 대조한다. network owner는 기존 SL/NSG overlap을 확인하고 새 stateful TCP80 ingress에 계약의 exact IPv4 CIDR만 입력한다. exact NSG OCID/새 rule IDs/source type CIDR/source CIDR/TCP destination80/readback UTC를 저장하고 owner61이 Rules에서 다시 읽는다. 미확인/더 넓은 source는 public Create 중단 및 자기 새 rule만 rollback/readback한다.
5. monitor Create의 vantage name은 계약과 동일 하나다. 생성 뒤 exact monitor OCID의 vantagePoints/target/protocol/port/path와 exact NSG rule CIDR를 다시 읽고 원문 계약과 대조한다. 불일치/expiry는 monitor disable/delete/readback → public listener 제거/readback → 자기 probe rules 제거/readback → 61 반환 순서로 중단한다. backend `/health` 보호와 public non-health403 제한은 그대로 유지한다. source provenance 변경은 새 owner 승인 전 자동 갱신하지 않는다.

### public branch 인수 gate — 61 최종 mode 선택 단계 확인
현재 인수 계약은 61의 **기본 private LB**와 생성 전 `LB_MODE=public` 선택 분기다. 실제61의 [mode 선택 단계](../08_Load_Balancer/00_Load_Balancer.md#lb-mode-before-create)를 다시 읽었으며 shared public-admin/rt-public/IGW, frontend NSG, sole LB_OCID 인계가 명시돼 있다. 이 분기의 public frontend subnet/IGW/route, 승인 probe source→TCP80 ingress, LB→private backend TCP8080, 비민감 HTTP 허용·예산·종료시각을 readback한 뒤만 이 과제를 실행한다. private LB를 public으로 즉석 전환하거나 추가 LB를 생성하지 않는다. 이미 승인된 public endpoint 대안은 해당 owner가 `/health` success body를 정의해야 하며 `ok`를 무조건 요구하지 않는다.
Health Checks는 public internet에서 접근 가능한 target만 지원한다. vantage identity는 현재 목록으로, outbound probe CIDR는 위 owner-provided 공식 근거 계약으로 각각 확인하며 둘을 혼동하지 않는다. network owner의 source 승인 없이는 진행하지 않는다. 장애 파일 rename에는 기존 `.saved` 충돌을 먼저 확인한다. 기본1backend에만 전체 endpoint failure를 기대한다. 승인2backend인 경우1대 health 중단은 degradation 시험이므로 외부200이 정상일 수 있으며 전체 failure라고 기록하지 않는다. 모든 backend 중단은 별도 downtime/범위 승인 없이는 하지 않는다. 예외·시험 중단 때도 파일 복구→direct health→LB health→외부 2회 성공을 확인한다. 정상 검사만 수행한 경우 장애 분기는 NOT_SELECTED로 기록한다.

### TLS-only 조건부 확장 — primary HTTP와 분리
엄격 TLS 환경은 `BLOCKED_TLS_HANDOFF`를 유지한다. owner65의 local --resolve 검증만으로 monitor 성공을 주장하지 않는다. approved certificate domain/SAN/유효한 trust chain, actual HTTPS443 listener, 선택 vantage source→443 rules 및 owner61 비용/expiry 인계가 모두 있어야 별도 HTTPS 절차를 승인받는다. IP target HTTPS는 공식 Create task의 Headers에 certificate domain의 Host header가 필요하다; 또는 certificate-matching FQDN target을 별도 승인한다. 입력/실제 monitor TLS 결과가 없으면 NOT_EXECUTED/BLOCKED이며 현재 HTTP80 recipe를 자동 HTTPS로 전환하지 않는다. [HTTPS IP-target Host header 조건](https://docs.oracle.com/en-us/iaas/Content/HealthChecks/Tasks/create-http-monitor.htm).

### 실행 단계

public gate를 완료한 뒤 실제 값으로 `export LB_IP='승인된 기존 public LB IPv4'`를 입력한다. 다른 endpoint 대안은 실제 승인 public IPv4/FQDN을 Target에 직접 입력한다. private IPv4를 재사용하지 않는다.

1. monitor Create 전 같은 승인 외부 client에서61의 /health 200/ok와 `/`, `/index.txt`, `/health.saved`, `/unknown`, `/health/` 모두403을 확인한다. 하나라도 non-health2xx면 exposure gate 실패로 monitor 생성 중단·61 반환한다. 이어 Observability & Management → Health checks → Create health check. Health check name **roadmap-http-health**, Create in compartment **실습 Compartment**, Targets **actual LB_IP**, Request type **HTTP**, Protocol **HTTP**, Port **80**, Method **GET**, Target path **/health**, Interval **60 seconds**, Timeout **10 seconds**, Vantage point **위 사전조회에서 source prefixes까지 승인/전달한 동일 actual name 1개**를 선택한다. 해당 Region/계정에 60초 옵션이 없으면 현재 허용되는 가장 긴 interval을 선택하고 기록한다.[T1]
2. Create 후 monitor OCID와 선택 vantage point를 기록한다. max 10분 내 results에서 HTTP status, latency, healthy 여부를 읽는다. 내부 LB backend health OK와 외부 probe 성공은 별도 관측값이다.
3. 다른 기존 public endpoint를 쓰는 경우 아래 nginx 장애 실험은 적용하지 않으며 그 endpoint owner의 승인된 장애·복구 절차가 없으면 정상 probe만 검증한다. 정상 probe 두 번을 확인한 뒤 **선택 장애 실험**은 61 dedicated nginx health file만 잠시 다른 이름으로 옮기는 방식으로 수행한다. 아래 **Collision-safe health failure and automatic restoration**의 guard/trap 포함 코드만 사용해 다음 probe failure를 기록한다. bare mv로 기존 saved 파일을 덮어쓰지 않는다. 복구 후 direct 200/ok, LB backend health OK, external probe success를 확인한다. 운영 서비스는 변경하지 않는다. 장애 실험은 별도 downtime 승인된 경우만 수행한다.
4. 실험 직후 monitor를 Disable하고 삭제한다. 78과 연계하는 optional failover는 실제 두 번째 endpoint 승인 없이는 만들지 않는다.


### 진행·종료 판정
설정 readback·기능시험·원복·정리를 각각 실제 응답으로 기록한다. 기대값은 미실행이며 실패/만료 시 새 mutation을 중단하고 아래 topic별 종료 단계로 반환한다.

### Collision-safe health failure and automatic restoration
61 sole nginx backend1대의 **선택 장애 승인 후** backend guest에서 수행한다. 기존health.saved가있으면중단한다. 2backend는별도승인없이전체중단하지않는다.
```bash
set -euo pipefail
# Backend guest: copy approved deadlines/actual interval from owner registry.
export EXPIRY_UTC='<approved-resource-expiry-UTC>'
export DOWNTIME_END_UTC='<approved-downtime-end-UTC>'
export PROBE_INTERVAL_SECONDS='<actual-monitor-interval-seconds>'
# 120s failure + 60s direct/LB recovery + two samples and one interval margin.
WAIT_SECONDS=$(python3 - <<'PY'
import os,time,datetime
parse=lambda key: datetime.datetime.fromisoformat(os.environ[key].replace('Z','+00:00')).timestamp()
interval=int(os.environ['PROBE_INTERVAL_SECONDS']); assert interval>0
assert 2*interval<=120, 'NOT_SELECTED: actual cadence cannot cover two probe opportunities in approved 120s outage'
remaining=int(min(parse('EXPIRY_UTC'),parse('DOWNTIME_END_UTC'))-time.time())
reserve=60+3*interval
assert remaining>=120+reserve, 'NOT_SELECTED: insufficient approved failure/recovery/probe lease'
print(min(120,remaining-reserve))
PY
) || exit 1
HEALTH=/var/www/oci-lab/health
SAVED=/var/www/oci-lab/health.saved
sudo test -f "$HEALTH" && ! sudo test -L "$HEALTH" || exit 1
if sudo test -e "$SAVED" || sudo test -L "$SAVED"; then printf 'saved collision: stop\n' >&2; exit 1; fi
# Dedicated owner-only directory: no concurrent writer/other outage experiment.
restore_health() {
  if sudo test -e "$SAVED" || sudo test -L "$SAVED"; then
    sudo test -f "$SAVED" && ! sudo test -L "$SAVED" || return 1
    if sudo test -e "$HEALTH" || sudo test -L "$HEALTH"; then printf 'restore destination collision: stop\n' >&2; return 1; fi
    sudo mv -T -- "$SAVED" "$HEALTH" || return 1
  else
    sudo test -f "$HEALTH" && ! sudo test -L "$HEALTH" || return 1
  fi
}
trap restore_health EXIT
trap 'exit 129' HUP
trap 'exit 130' INT
trap 'exit 143' TERM
sudo mv -T -- "$HEALTH" "$SAVED"
curl --max-time 10 -i http://127.0.0.1:8080/health
# 위 direct 실제404를 기록; --fail은 의도한404에서 script를 조기 종료하므로 사용하지 않는다.
# 최대120초 내 별도 Console에서 external probe failure 1건을 관찰한다.
# direct는404, LB는health판정후non-2xx/timeout일수있다. 정확한code는실측한다.
sleep "$WAIT_SECONDS"
restore_health
trap - EXIT HUP INT TERM
curl --fail --max-time 10 http://127.0.0.1:8080/health
```
LB health OK → 실제 외부200/ok → monitor results 정상2건을 다시 확인한다. network/session 손실로 trap이 실행되지 않았으면 owner가 전용 directory의 소유권/원래 파일 인수를 확인한 뒤 위 HEALTH/SAVED 설정과 restore_health 함수를 새 터미널에서 다시 정의하여 실행한다. -e 또는 -L인 saved는 regular/non-symlink일 때만, health는 -e와 -L 모두 부재일 때만 복원한다. collision은 덮어쓰지 않고 중단한다. remaining lease guard가 통과한 최대120초 관찰대기는 foreground이며 OCI결과를 합성하지 않는다. guard와 trap이 있는 위 절차를 bare mv 대신 사용한다.

### 장애 선택 cadence gate
최대120초 장애 안에 actual cadence의 두 probe 기회를 포함하도록 **2×actual interval≤120초**를 mutation 전에 확인한다. interval이 더 길면 outage는 NOT_SELECTED이고 정상 probe만 검증한다. probe phase/지연 때문에 실패 sampling을 보장하지 않으며 실제 failure 행이 없으면 NOT_OBSERVED다.120초를 몰래 늘리지 않는다.


### 번호 순서로 완료하는 원복·정리

6. **DNS Health Checks 종료 및 실패 반환.** NOT_SELECTED이면monitor/public규칙을만들지않았음을기록하고반환한다.SELECTED는선택장애복구후monitor Disable/isEnabled=false→Delete/목록부재→networkowner의이번probe rule IDs만제거/readback→61반환순서다.health-only,정상external두timestamp,원본health파일확인은생략하지않는다.

실제 정리/원복 response, exact resource 또는rule ID,완료UTC와남은owner/expiry를비밀없는개인실행기록에남긴다.위의guarded복구가이미완료된경우중복mutation하지않는다.

## 결과검증

**기대 패턴(미실행):** healthy HTTP 200+latency; 선택 장애 시 non-2xx/timeout→복구 후 200. actual probe timestamp와 vantage point를 기록한다. 설정 화면만으로 health check 동작을 주장하지 않는다.

실행 기록(미기입): 실행 UTC __ / 실제 resource OCID __ / 요청·SQL 실제 응답 __ / 공식 조건 차이 __ / 정리 완료 UTC __.

## 문제해결

timeout은 public LB NSG/security list의 :80 허용 범위 및 probe vantage point의 접근 가능성을 확인한다. private IP는 external Health Checks 대상이 아니다. WAF rule이 probe를 차단하면 해당 실습 경로의 rule만 정리한다.

## 리소스정리

monitor disable → monitor 삭제. 공유 LB/backend는 마지막 종속 실습 확인 후 61에 따라 삭제한다. 승인된 장애 변경은 원래 App 상태로 복구한다.

### 소유자별 종료와 exact 대상 재조회
79 owner: Health check actual monitor OCID Edit/Disable 저장 → isEnabled=false readback → Actions/Delete/confirm → Health checks 목록 부재. health.saved 부재/health 존재, direct/LB/외부200 복귀를61에 반환한다. network owner는79가 추가한 exact vantage-point rule ID만 삭제·Rules 부재를 읽고 기존 frontend rules는 유지한다.

실제 정리 UTC __ / exact resource 또는 rule ID __ / 삭제 후 조회 응답 __ / 잔존 resource·만료·owner __. 승인 없는 잔존 비용은 다음 실습을 이유로 연장하지 않는다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 개념은 해당 생성·변경·조회 단계에서 입력 선택과 실제 결과 해석을 위해 사용하며, 공식 예시는 실제 실습 실행 결과가 아닙니다.

- [24] [Overview of the Health Checks Service — DNS Health Checks](https://docs.oracle.com/en-us/iaas/Content/HealthChecks/Concepts/healthchecks.htm)
  - 확인 범위: Health Checks service provides.

### 실습 task 본문 근거

- [T1] [Tasks / create-http-monitor.htm](https://docs.oracle.com/en-us/iaas/Content/HealthChecks/Tasks/create-http-monitor.htm) — 본문 절차·입력 필드·지원 조건을 읽어 실습에 적용.
- [T2] [Concepts / healthchecks.htm](https://docs.oracle.com/en-us/iaas/Content/HealthChecks/Concepts/healthchecks.htm) — 본문 절차·입력 필드·지원 조건을 읽어 실습에 적용.

