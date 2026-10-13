# Stateful / Stateless Rules

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../README.md)를 확인하세요.

> **저술 완료 · OCI 실행 미검증** — Oracle 공식 작업 문서를 근거로 작성한 한국어 hands-on입니다. Oracle 공인 교육/실행 인증이 아니며 기대 결과와 실제 결과를 구분합니다. 비용 승인과 실제 선행 산출물 확인 후에만 실행하세요.

## 학습목표

- 기존 A/B NSG에서 TCP8083 stateful forward 성공을 확인하고 실제 A ephemeral range를 조회한다.
- 두 forward를 stateless로 바꿔 실패를 기록한 뒤 B egress/A ingress response rule을 추가해 복구하고 자기 네 rule만 제거한다.

## 대응 Roadmap 항목

- 항목명: **Stateful / Stateless Rules**
- Roadmap 경로: **Virtual Cloud Network (VCN) → Stateful / Stateless Rules**
- Node ID: `sequence-3-9`
- 상위 항목: **Virtual Cloud Network (VCN)** (`sequence-3-0`)
- 문서 범위: 기존 A/B NSG에서 TCP8083 stateful forward 성공을 확인하고 실제 A ephemeral range를 조회한다. 두 forward를 stateless로 바꿔 실패를 기록한 뒤 B egress/A ingress response rule을 추가해 복구하고 자기 네 rule만 제거한다. 실제 OCI 실행 결과가 아니라 이 절차의 완료 조건이다.
- [전체 항목 목록](../README.md)

## 사전준비

### 기준 계약과 실제 단계 선행조건

[공통 baseline-contract.json](../../baseline-contract.json)의 OS/CIDR/소유권을 따른다. Oracle Linux 9 / `opc` / `dnf` 외의 숨은 Ubuntu 또는 probe VM은 만들지 않는다. OCID/IP의 null은 실행 전 반드시 실제 출력으로 채우는 **실행 차단 입력**이다. approval/capacity/quota/identity 미확인은 완료가 아니다.

- [24 Route Tables](05_Route_Tables.md#baseline-24-ready): **A/B 관리 SSH, owner24 packet-lab/index.html=oci-packet-lab·health.txt=packet-health-ok 및8080 허용/guest ledger**를 실제 출력으로 인수한다. 이 문서의 변경/반환까지 creator 승인 expiry 안에서 보존한다.
- [27.ready Ingress / Egress — FUNCTIONAL_COMPLETION](08_Ingress_Egress.md#baseline-27-ready): DIRECTION-tested와 **owner27 cleanup의 실제 반환 증거**를 요구한다. owner27의 [단일 cleanup/중단복구 station](08_Ingress_Egress.md#owner27-return)은 양 guest의 exact zone/rule, runtime/permanent before-state, add 전 durable pending, query-error/inactive fail-closed, owned rule 제거 후 readback을 정의한다. 실행자는 양 guest의 listener8082 부재·firewall before-state 복원·ledger 제거, OCI 임시 rule/NSG 제거·원래 SL/NSG 배열 복원, SSH22와 owner24 HTTP 성공을 실제 증거로 전달해야 한다. 현재 문서 수정이나 로컬 stub PASS는 반환 증거가 아니며, 누락·실패·lease expiry이면28 mutation은 `BLOCKED_PREREQUISITE_27`이다. 새 station이나 별도27 remove를 만들지 않는다.

공유 A/B consumer는 INSTANCE_A_ID에 owner10 vm-a의 실제 OCID, INSTANCE_B_ID에 owner22 private-vm의 실제 OCID, `A_PUBLIC_IP`, `A_PRIVATE_IP`, `B_PRIVATE_IP`를 실제 생성 응답에서 전달받는다. 해당 주제에 필요하지 않은 VM/변수는 생성·요구하지 않는다.

- [Route Tables](05_Route_Tables.md) **RT-ready**, [Ingress / Egress](08_Ingress_Egress.md) **DIRECTION-tested** 완료. packet 캡처 도구가 준비돼야 한다.
- 전용 TCP8083에 해당하는 다른 stateful/stateless allow가 없어야 한다. A/B의 기존 설정을 백업하고 SSH22는 별도 stateful로 보존한다.

## 권한

기존 A nsg-admin/B nsg-private-admin의 자기8083 네 rule Add/Update/Remove/read만 승인한다. overlap Security List를 변경해야 하면 기존 owner의 UpdateSecurityList 승인을 별도로 확인한다. 새 NSG/VM/VNIC membership은 생성·변경하지 않는다. host firewall ledger와 readback은 opc/sudo 및 exclusive lease로 수행한다.

### 실제 operation과 공식 권한

- `AddNetworkSecurityGroupSecurityRules`: NETWORK_SECURITY_GROUP _UPDATE_SECURITY_RULES and NETWORK_SECURITY_GROUP_INSPECT if writing a rule that specifies a network security group as the source (for ingress rules) or destination (for egress rules) — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `RemoveNetworkSecurityGroupSecurityRules`: NETWORK_SECURITY_GROUP _UPDATE_SECURITY_RULES — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `UpdateNetworkSecurityGroupSecurityRules`: NETWORK_SECURITY_GROUP _UPDATE_SECURITY_RULES and NETWORK_SECURITY_GROUP_INSPECT if writing a rule that specifies a network security group as the source (for ingress rules) or destination (for egress rules) — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `UpdateSecurityList`: SECURITY_LIST_UPDATE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `ListNetworkSecurityGroupSecurityRules`: NETWORK_SECURITY_GROUP_LIST_SECURITY_RULES — [공식 API 권한 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).

위 permission 이름은 IAM policy resource-type이 아니다. 조직 관리자가 기존 group/compartment scope와 conditional dependency를 실제 API 표에 대조한다. 편의 family grant를 최소권한 정책으로 부르지 않으며 실습자는 policy 작성 권한을 자동 인수하지 않는다.

## 비용및주의사항

- 새 VM/NSG는 없다. 기존 nsg-admin/nsg-private-admin에 자기 TCP8083 forward2/response2만 추가하고 B의 owner24 파일을8083 전용 process로 읽는다. 기존 A/B의 보존 비용 창만 사용한다.
- A 실제 ephemeral range를 읽어 response destination을 정하며 broad allow/겹친 stateful rule을 남기지 않는다. 작은 body로 제한하고 SSH22는 stateful 유지한다.60초 안에 네 rule 구성이 안 되면 forward를 stateful로 원복하고 중단한다. cleanup은 자기 process/guest ledger/네 OCI rule만 대상으로 한다.

실제 OCI 작업은 아직 수행하지 않았다. 기대값은 실행 시 대조할 조건이며 무료 사용이나 과금 강제 차단을 보장하지 않는다. 승인 expiry가 끝나면 새 시험을 중단하고 위 소유권 순서로 원복·반환한다.

## 실습구성

### 이번 workshop 시나리오와 입력 계약

같은 작은 HTTP 요청을 stateful에서 네 개 stateless rule로 전환한다. 

|입력|값/출처|시점|실행 위치|선택 이유|
|---|---|---|---|---|
|`A_PRIVATE_IP`|owner10 primary GetVnic private-ip; B ingress/guest rule의 source /32|해당 선행 완료 또는 실제 guest 조회 후|본문에 지정한 Console/Cloud Shell 또는 guest| B ingress/guest rule의 source /32|
|`B_PRIVATE_IP`|owner22 primary GetVnic private-ip; A egress/HTTP URL의 대상|해당 선행 완료 또는 실제 guest 조회 후|본문에 지정한 Console/Cloud Shell 또는 guest| A egress/HTTP URL의 대상|
|`A_EPHEMERAL_MIN`|A guest sysctl net.ipv4.ip_local_port_range 첫 값; response destination 하한|해당 선행 완료 또는 실제 guest 조회 후|본문에 지정한 Console/Cloud Shell 또는 guest| response destination 하한|
|`A_EPHEMERAL_MAX`|같은 A sysctl 출력 둘째 값; response destination 상한|해당 선행 완료 또는 실제 guest 조회 후|본문에 지정한 Console/Cloud Shell 또는 guest| response destination 상한|

|Console field|실제 입력/선택|이유|
|---|---|---|
|Stateless / TCP source range / destination range|forward destination8083; reverse source8083/destination actual A ephemeral range|이번 기능/실패 원인을 한 변수로 분리|

**생성·변경 범위:** stateless-forward:SecurityRule,stateless-reverse:SecurityRule. **완료 비교:** stateful success→forward-only deny→four-rule success. 기대 패턴이며 OCI 실측값이 아니다. **정리:** 본 주제의 변경만 원복하고 전용 신규 resource만 dependency 역순으로 정리한다; 공유 자원은 creator와 consumer registry/expiry 기준으로 반환한다.

동일 HTTP 흐름을 stateful → stateless forward only → stateless reverse 추가 순으로 바꾼다. 실제 A ephemeral 범위는 `sysctl net.ipv4.ip_local_port_range`로 확인한다. 예시 `32768-60999`를 자신의 값으로 교체한다. Stateless reverse는 destination8083이 아니라 **source8083 → client ephemeral destination**이다.

## 실습절차

### 명령 실행 gate — 실패하면 다음 생성으로 진행하지 않는다

1. 아래 Bash는 **전용 Bash 세션**에서만 사용한다. Console/Cloud Shell 조회, 본인 로컬 SSH, 로그인한 Oracle Linux guest를 섞지 않는다. `exit 1`은 이 전용 세션을 닫는 안전 중단이다. `<...>`·빈 입력·타 owner OCID가 남으면 실행하지 않는다.
2. 성공을 요구하는 파일 작성/조회/설치/attach 작업은 실패 시 중단한다. 실패한 API 뒤 기존 JSON이나 기존 다운로드 파일을 성공 근거로 읽지 않는다. 기대 deny는 `if command; then ...; else RC=$?; ...; fi`로 별도 기록하고, 실제 timeout/IAM 오류/로컬 파일 오류를 구별한다.
3. 첫 변경 전에 정확한 OCID·owner·현재 설정과 승인 expiry를 저장한다. 공유 자원 변경 중 다른 소비자가 시작하거나 원복값을 확인할 수 없으면 변경을 중단한다. 삭제 성공은 API 수락이 아니라 같은 identity/Region의 정확한 대상 재조회로 판정한다. `NotAuthorizedOrNotFound` 단독은 삭제 증거가 아니다.


### 개별 완주 보정 — 생성·검증·소유권

B listener는 ~/packet-lab에서 실행한다. Rule editor에서 TCP forward source=All/destination8083; reverse source8083/destination 실제 A ephemeral 범위이다. SSH22 stateful rule은 그대로 둔다. 각 새 curl의 source port를 별도 capture에서 확인한다.

### Console 규칙 입력과 원복 ledger — TCP8083

1. **Networking → Virtual cloud networks → shared-vcn → Security → Network security groups**(또는 동일 이름 탭)에서 본문 대상 NSG의 OCID를 연다. Add Rules의 Direction은 표의 방향, Stateless는 해당 단계값, IP Protocol은 TCP(6), Source/Destination Type은 본문 CIDR 또는 같은 VCN NSG를 선택한다. Forward source port=All, destination port=8083이다. /32는 실제 peer private IP이며 internet source는 없다.
2. **Compute → Instances → 대상 → Networking → Attached VNICs → primary → Edit**에서 기존 NSG OCID 배열을 보존하고 새 NSG를 생성하지 않고 nsg-admin(A)/nsg-private-admin(B)에 본인 rule만 추가한다. nsg-admin/nsg-private-admin의 SSH22 membership은 유지한다. Rule 제출 뒤 NSG → Security Rules와 VNIC 상세를 다시 읽고 추가 rule ID·stateful flag·source/destination·port를 ledger에 적는다. 원복은 그 ID/배열만 대상으로 한다.
3. 첫 deny 전에 복구 Console 담당자에게 exact 원복 rule/멤버십과 승인된 **60초 최대 시험 창**을 전달한다. 한 번의 8초 유한 요청 뒤 곧바로 원복한다. 진단을 위해 deny 상태를 연장하지 않는다. 원복 후 새 연결 positive가 실패하면 후속 lab/cleanup을 시작하지 말고 listener→guest firewall→OCI union→route 순으로 복구한다.


### Oracle Linux guest positive-control 준비

이 주제의 B guest에서만 PEER_IP=A_PRIVATE_IP로 지정한다. A의 신규 listener/firewall은 이 topic에서 만들지 않는다. FIREWALL_ZONE은 `firewall-cmd --get-active-zones`에서 primary interface가 연결된 실제 zone으로 지정한다. 비활성 firewalld를 임의 활성화하거나 SELinux를 disable하지 않는다. shell별 RULE_ADDED/HTTP_RULE 값을 ledger에 보존한다.
첫 firewall mutation 전에 owner28은 **같은 zone/rule의 독점 변경 lease**를 host security owner에게 승인받고 다른 writer/reload/permanent edit를 금지한다. lease ID/담당자/expiry를 보존한다. 중단 복구도 lease 확인 전 remove하지 않는다. query exit1만 absent이며 inactive/query-error는 ledger 보존 후 중단한다.

```bash
export PEER_IP='<상대_VM_actual_private_IP>'
export FIREWALL_ZONE='<이_guest_primary_interface_active_zone>'
python3 - <<'PY'
import ipaddress,os
x=ipaddress.ip_address(os.environ['PEER_IP'])
assert x.version==4 and x.is_private, 'STOP: approved actual private IPv4 required'
PY
if [ "$?" -ne 0 ]; then exit 1; fi
export HTTP_RULE="rule family=\"ipv4\" source address=\"$PEER_IP/32\" port port=\"8083\" protocol=\"tcp\" accept"
STATE="$HOME/owner28-firewall.state"
# first mutation 이전 collision/symlink 차단; noclobber로 private ledger 선점
if test -e "$STATE" || test -L "$STATE"; then printf '%s\n' 'STOP: existing ledger; owner recovery required'; exit 1; fi
(umask 077; set -C; printf 'FIREWALL_ZONE=%q\nHTTP_RULE=%q\nRULE_ADDED=0\nPENDING=0\n' "$FIREWALL_ZONE" "$HTTP_RULE" > "$STATE") || exit 1
RULE_ADDED=0
systemctl is-active --quiet firewalld || exit 1
case "$FIREWALL_ZONE" in ''|*'<'*|*'>'*) exit 1;; esac
sudo firewall-cmd --zone="$FIREWALL_ZONE" --list-all >/dev/null || exit 1
sudo firewall-cmd --permanent --zone="$FIREWALL_ZONE" --list-all >/dev/null || exit 1
query_rule28() {
  if sudo firewall-cmd "$@" --zone="$FIREWALL_ZONE" --query-rich-rule="$HTTP_RULE"; then QUERY_STATE=1;
  else QUERY_RC=$?; [ "$QUERY_RC" = 1 ] || return 2; QUERY_STATE=0; fi
}
query_rule28 || exit 1; RUNTIME_BEFORE=$QUERY_STATE
query_rule28 --permanent || exit 1; PERMANENT_BEFORE=$QUERY_STATE
printf 'RUNTIME_BEFORE=%s\nPERMANENT_BEFORE=%s\n' "$RUNTIME_BEFORE" "$PERMANENT_BEFORE" >> "$STATE" || exit 1
sync -f "$STATE" || exit 1
sync -f "$HOME" || exit 1
if [ "$RUNTIME_BEFORE" = 0 ] && [ "$PERMANENT_BEFORE" = 1 ]; then printf 'STOP: permanent/runtime divergence; preserve ledger\n'; exit 1; fi
if [ "$RUNTIME_BEFORE" = 0 ]; then
  # fsync-equivalent durable intent BEFORE add; interrupted add retains exact inputs
  printf 'PENDING=1\n' >> "$STATE" || exit 1
  sync -f "$STATE" || exit 1
  sudo firewall-cmd --zone="$FIREWALL_ZONE" --add-rich-rule="$HTTP_RULE" || exit 1
  RULE_ADDED=1
  printf 'RULE_ADDED=1\nPENDING=0\n' >> "$STATE" || exit 1
  sync -f "$STATE" || exit 1
fi
sudo firewall-cmd --zone="$FIREWALL_ZONE" --query-rich-rule="$HTTP_RULE" || exit 1
# 다른 terminal도 같은 guest의 이 private file을 읽는다. 실패 후 재실행/add 금지.
```

OCI deny 전 같은 listener/guest firewall에서 positive control이 성공해야 한다. listener는 owner24의 `$HOME/packet-lab/index.html`을 읽기 전용으로 재사용하며 개인 HOME root에서 HTTP server를 열지 않는다. `8083`는 인터넷 source로 허용하지 않는다.

**터미널 실행 구분:** `tcpdump`는 별도 SSH 터미널에서 먼저 시작하고, `curl`/CLI workload는 다른 터미널에서 실행한다. 같은 shell에서 capture 종료를 기다렸다가 요청하면 packet이 수집되지 않는다. 본문의 유한 요청이 실패해도 담당자는 승인 deadline에 해당 HTTP/gateway 변경만 즉시 원복한다.

### 사용하는 실제 rule container와 필드

A의 기존 `nsg-admin`과 B의 기존 `nsg-private-admin`에 이 topic 소유 TCP8083 rule만 추가한다. 신규 NSG는 만들지 않는다. Create/association 대신 기존 member readback을 확인한다. forward 2개(A Egress destination B/32; B Ingress source A/32), response 2개(B Egress destination A/32; A Ingress source B/32)의 정확한 rule ID를 구분한다. response source port8083/destination actual ephemeral range만 지정한다. 정리 때도 해당 네 rule만 제거한다.

### A. Stateful 기준 결과
1. 위 durable block으로 B source A/32 TCP8083 runtime rule을 한 번만 준비한다. B의 별도 server terminal에서 `cd "$HOME/packet-lab" && python3 -m http.server 8083 --bind 0.0.0.0` 실행 후 다른 B terminal의 `curl --noproxy '*' --fail --max-time 8 http://127.0.0.1:8083/index.html` 본문=oci-packet-lab를 확인하고 listener/guest rule을 고정한다. A egress `<B>/32` TCP dest8083와 B ingress `<A>/32` TCP dest8083를 **stateful**로 추가한다. A/B의 broad 허용은 관리용을 제외하고 제거한다.
2. B에서 `sudo timeout 20s tcpdump -ni any 'tcp port 8083'`, A에서 새 curl을 실행하여 handshake와 HTTP body를 기록한다.

### B. Stateless 응답 누락
3. 앞의 두 rule의 **Stateless** 체크를 켠다. 겹치는 stateful rule을 남기지 않는다. B egress response/A ingress response rule은 추가하지 않은 상태로 새 curl을 실행한다. 요청 SYN은 B에서 보이지만 response가 정상 전달되지 않는 상태를 기록한다. 기존 tracked TCP와 혼동하면 B의 **이 topic8083 server terminal에서만 Ctrl-C** → ss로8083 listener 부재 확인 → 같은 terminal에서 `cd "$HOME/packet-lab" && python3 -m http.server 8083 --bind 0.0.0.0` 재실행 → B 별도 terminal localhost/index.html body=oci-packet-lab 확인 → A에서 새 curl process를 시작한다. owner24의8080은 중지/재시작하지 않는다. 60초 시험 창 안에 준비가 안 되면 두 forward rule을 stateful로 복구하고 재승인 전 중단한다.

```bash
# A/B capture terminal 각각 먼저 시작 (다음 요청 terminal과 다름)
sudo timeout 20s tcpdump -ni any 'tcp port 8083'
```

capture 시작을 확인한 후 **A의 다른 request terminal**에서 실행한다.

```bash
sysctl net.ipv4.ip_local_port_range || exit 1
if curl --noproxy '*' --connect-timeout 3 --max-time 8 "http://$B_PRIVATE_IP:8083/index.html"; then
  printf '%s\n' 'Unexpected success without reverse rules: investigate overlaps'
else RC=$?; printf 'stateless incomplete RC=%s; classify transport versus local failure\n' "$RC"; fi
```

### C. Stateless reverse 규칙 추가
4. B NSG **Egress/Stateless** destination `<A>/32`, TCP **source8083**, destination `<A_EPHEMERAL_MIN>-<A_EPHEMERAL_MAX>`를 추가한다. A NSG **Ingress/Stateless** source `<B>/32`, TCP source8083, destination 같은 ephemeral 범위를 추가한다. Forward 두 rule은 유지한다.
5. 새 curl을 실행하여 성공을 기록한다. 상태 추적에 의존하지 않는 네 방향 rule(A egress/B ingress/B egress/A ingress)과 packet tuple이 일치하는지 확인한다. 큰 packet/path MTU 확장 실험은 별도 ICMP type3 code4 규칙이 필요하므로 이 실습은 작은 body로 제한한다.

## 결과검증

<a id="baseline-28-ready"></a>

완료 단계 **STATE-tested**: stateful 성공 → stateless forward-only 실패 → stateless 양방향 성공. 각 상태의 rule ID/Stateless flag와 새 연결의 source port를 남긴다. 같은 트래픽을 stateful/stateless가 동시에 허용하면 stateless가 우선하는 공식 동작을 읽되 모호한 overlap 상태를 성공 근거로 사용하지 않는다.

**실행 기록(아직 미작성):** 실행자/Region/시각, exact resource OCID, 설정 전·후, 성공 출력, deny 오류·exit code/request-id, positive control, 정리 결과. 문서 검증은 실제 OCI 실행 성공을 뜻하지 않는다.

### 이 시나리오의 functional acceptance

| Positive | Negative / 미완료 판정 | Recovery / 반환 |
|---|---|---|
| stateful TCP8083 body | 두 stateless forward-only는 response 부족으로 실패 | B egress/A ingress source8083 destinationactual ephemeral 추가 후 body |

모두 실행 시 확인할 기대 패턴이다. 생성 전 guard negative는 OCI 장애를 주입한 것과 다르며 다른 자원을 추가하지 않는다. 오류 메시지와 실제 exit code가 없는 항목은 NOT_EXECUTED 또는 BLOCKED로 남긴다.

## 문제해결

forward-only인데 성공하면 broad egress/ingress와 기존 tracked connection을 조사한다. reverse 추가 후 실패하면 source와 destination port 뒤집힘, A ephemeral 실제 범위, SL에서도 response가 차단됐는지 확인한다. TCP handshake가 성공했는데 큰 응답만 멈추면 MTU/ICMP 문제이며 stateful 차이로 단정하지 않는다. OS firewall rule은 실험 내내 동일하게 유지한다.


## 리소스정리

### cleanup 시작 전 마지막 readback

1. 이 topic이 소유한 rule/file/resource만 목록화하고 실제 creator와 예약 consumer에게 반환을 받는다. 비용 expiry가 끝나면 새로운 consumer를 시작하지 않고 미완료 작업을 중단/원복한다. 다른 owner 리소스는 아래 본문 순서로 해당 owner가 정리한다.
2. 관리 SSH와 현재 app positive를 먼저 확인한 뒤 server를 자신이 연 terminal의 Ctrl-C로 종료한다. `ss -lntp`에서 본 topic 포트 listener가 사라지고 primary 새 SSH는 살아 있는지 확인한다. `pkill python`/전체 service stop은 쓰지 않는다.
3. runtime firewall 원복은 아래 durable-ledger block만 수행한다(이 단계에서 별도 remove 금지). OCI 자기 rule ID 제거 뒤 SSH22와 원래 배열을 readback한다. 공유 NSG/SL은 삭제하지 않는다.
4. 이 topic은 app 파일/디렉터리를 만들지 않았다. owner24 packet-lab/index.html·health.txt·8080 process를 삭제/rmdir/중지하지 않는다. 자기8083 process와 ledger만 정리하고 app lease를 owner24에게 반환한다. 증거는 승인 보존기한까지 유지한다.


8083 서버 중지 → 새 reverse/forward rule 제거 → original SL/NSG 복구 → 기존 nsg-admin/nsg-private-admin의 SSH22·membership을 재조회. 이 lab은 신규 NSG를 만들지 않았으므로 공유 NSG를 삭제하지 않는다. ephemeral 포트 전체를 인터넷에 열어둔 상태로 종료하지 않는다. 관리 stateful22와 VM은 29~32까지 보존한다.

**guest 원복은 이 block 한 번만:** B cleanup terminal에서 본인이 작성한 owner28 private ledger를 읽는다. PENDING=1은 add 직전 query=no였던 중단 복구 상태다. 동시 firewall writer가 있으면 중단하고 owner와 조정한다. pre-existing rule(RULE_ADDED=0/PENDING=0)은 제거하지 않는다. firewalld가 예기치 않게 inactive이면 소유 ledger를 지우지 말고 관리자 복구를 기다린다.

```bash
STATE="$HOME/owner28-firewall.state"
test -f "$STATE" && test ! -L "$STATE" && test -O "$STATE" || exit 1
source "$STATE" || exit 1
case "${RUNTIME_BEFORE:-}:${PERMANENT_BEFORE:-}" in 0:0|0:1|1:0|1:1) ;; *) exit 1;; esac
systemctl is-active --quiet firewalld || exit 1
sudo firewall-cmd --zone="$FIREWALL_ZONE" --list-all >/dev/null || exit 1
sudo firewall-cmd --permanent --zone="$FIREWALL_ZONE" --list-all >/dev/null || exit 1
if sudo firewall-cmd --permanent --zone="$FIREWALL_ZONE" --query-rich-rule="$HTTP_RULE"; then CURRENT_PERMANENT=1;
else RC=$?; [ "$RC" = 1 ] || exit 1; CURRENT_PERMANENT=0; fi
test "$CURRENT_PERMANENT" = "$PERMANENT_BEFORE" || { printf 'STOP: permanent state changed; preserve ledger\n'; exit 1; }
if sudo firewall-cmd --zone="$FIREWALL_ZONE" --query-rich-rule="$HTTP_RULE"; then CURRENT_RUNTIME=1;
else RC=$?; [ "$RC" = 1 ] || exit 1; CURRENT_RUNTIME=0; fi
# Permanent/pre-existing rules are not owned; preserve them and journal on ambiguity.
if [ "$PERMANENT_BEFORE" = 1 ] || [ "$RUNTIME_BEFORE" = 1 ]; then
  test "$CURRENT_RUNTIME" = "$RUNTIME_BEFORE" || exit 1
elif [ "$RULE_ADDED" = 1 ] || [ "$PENDING" = 1 ]; then
  systemctl is-active --quiet firewalld || exit 1
  sudo firewall-cmd --zone="$FIREWALL_ZONE" --list-all >/dev/null || exit 1
  if sudo firewall-cmd --zone="$FIREWALL_ZONE" --query-rich-rule="$HTTP_RULE"; then
    sudo firewall-cmd --zone="$FIREWALL_ZONE" --remove-rich-rule="$HTTP_RULE" || exit 1
  else RC=$?; [ "$RC" = 1 ] || exit 1; fi
  if sudo firewall-cmd --zone="$FIREWALL_ZONE" --query-rich-rule="$HTTP_RULE"; then exit 1; else RC=$?; [ "$RC" = 1 ] || exit 1; fi
fi
rm -- "$STATE" || exit 1
```

위 readback 성공 뒤에만 ledger를 제거한다. permanent/전체 ruleset을 수정하지 않으며 OCI rule은 자기 ID만 제거한다. VM/subnet/IGW/VCN은 원래 creator에게 반환한다.


## 공식문서

### HTTP workload의 공식 Oracle 근거와 적용 범위

[Oracle Linux 공식 HTTP 두 host 예제](https://docs.oracle.com/en/operating-systems/oracle-linux/mptcp/mptcp-testing_mptcp_subflows.html)는 B에서 `python3 -m http.server`를 실행하고 A에서 curl로 확인한다. [Oracle Linux9 공식 예제의 Python HTTP root](https://docs.oracle.com/en/learn/ol-podman-intro/)는 Oracle Linux9 image의 Python 표준 HTTP server와 working directory를 document root로 사용한다. 이 lab은 **MPTCP/Podman/container를 도입하지 않고**, 같은 HTTP 진단 원리만 기존 Oracle Linux9 guest에 적용한다. 본문의8080–8084, 명시적 index.html/health.txt, runtime /32 firewall 및 복구 trap은 이 lab의 제한접근·소유권 요구에 맞춘 저술값이며 공식 문서의 public80/root directory listing 값을 복사하지 않는다. 시작 전 `python3 -m http.server --help`에서 실제 guest의 `--bind` 지원을 확인하고 미지원이면 중단한다. Oracle Linux9/OCI에서 실행 결과는 아직 없다.


아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 아래 본문은 각 생성 필드·설정·검증·정리의 기술 근거이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [31] [Security Rules — Stateful / Stateless Rules](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/securityrules.htm)
  - 확인 범위: Stateful or stateless:.

### 실습 작업을 지원하는 추가 공식 본문

- [실습근거 1] [Security Rules](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/securityrules.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
