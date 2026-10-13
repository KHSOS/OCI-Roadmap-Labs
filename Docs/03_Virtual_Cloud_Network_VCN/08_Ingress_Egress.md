# Ingress / Egress

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../README.md)를 확인하세요.

> **저술 완료 · OCI 실행 미검증** — Oracle 공식 작업 문서를 근거로 작성한 한국어 hands-on입니다. Oracle 공인 교육/실행 인증이 아니며 기대 결과와 실제 결과를 구분합니다. 비용 승인과 실제 선행 산출물 확인 후에만 실행하세요.

## 학습목표

- 전용 nsg-client-dir/nsg-server-dir와 TCP8082 listener를 양 guest에 구성하고 A→B 및 역방향 positive control을 각각 확인한다.
- B→A 신규 ingress 부재와 A egress 제거를 각각 단일 변경으로 시험하여 실패·복구를 확인하고 SSH22와 기존 설정을 유지한다.

## 대응 Roadmap 항목

- 항목명: **Ingress / Egress**
- Roadmap 경로: **Virtual Cloud Network (VCN) → Ingress / Egress**
- Node ID: `sequence-3-8`
- 상위 항목: **Virtual Cloud Network (VCN)** (`sequence-3-0`)
- 문서 범위: 전용 nsg-client-dir/nsg-server-dir와 TCP8082 listener를 양 guest에 구성하고 A→B 및 역방향 positive control을 각각 확인한다. B→A 신규 ingress 부재와 A egress 제거를 각각 단일 변경으로 시험하여 실패·복구를 확인하고 SSH22와 기존 설정을 유지한다. 실제 OCI 실행 결과가 아니라 이 절차의 완료 조건이다.
- [전체 항목 목록](../README.md)

## 사전준비

### 기준 계약과 실제 단계 선행조건

[공통 baseline-contract.json](../../baseline-contract.json)의 OS/CIDR/소유권을 따른다. Oracle Linux 9 / `opc` / `dnf` 외의 숨은 Ubuntu 또는 probe VM은 만들지 않는다. OCID/IP의 null은 실행 전 반드시 실제 출력으로 채우는 **실행 차단 입력**이다. approval/capacity/quota/identity 미확인은 완료가 아니다.

- [24 Route Tables](05_Route_Tables.md#baseline-24-ready): **A/B 관리 SSH, owner24 packet-lab/index.html=oci-packet-lab·health.txt=packet-health-ok 및8080 허용/guest ledger**를 실제 출력으로 인수한다. 이 문서의 변경/반환까지 creator 승인 expiry 안에서 보존한다.
- [25 Network Security Groups](06_Network_Security_Groups.md#baseline-25-ready): **source-NSG 시험과 owner24 CIDR baseline 원복 readback; nsg-client/nsg-web는 재사용하지 않음**를 실제 출력으로 인수한다. 이 문서의 변경/반환까지 creator 승인 expiry 안에서 보존한다.

공유 A/B consumer는 INSTANCE_A_ID에 owner10 vm-a의 실제 OCID, INSTANCE_B_ID에 owner22 private-vm의 실제 OCID, `A_PUBLIC_IP`, `A_PRIVATE_IP`, `B_PRIVATE_IP`를 실제 생성 응답에서 전달받는다. 해당 주제에 필요하지 않은 VM/변수는 생성·요구하지 않는다.

- [Route Tables](05_Route_Tables.md) **RT-ready**와 [Network Security Groups](06_Network_Security_Groups.md) **NSG-tested** 완료. client A/server B 실제 private IP를 기록한다.
- 8082를 허용하는 기존 NSG/SL ingress·egress가 없어야 한다. egress 전체 허용을 제거할 때 SSH 관리용 stateful rule과 DNS가 필요한 경우 resolver 경로를 보존한다. 기존 배열을 백업한다.

## 권한

두 direction NSG Create/rule add/remove/read/Delete와 기존 A/B VNIC membership UpdateVnic을 승인한다. OCI rule 및 양 guest exclusive firewall lease는 별도다. 이 주제는 A/B 신규 VM이나 VCN/route 생성 권한을 요청하지 않는다. owner27-return의 original arrays/자기 rule만 대상으로 정리한다.

### 실제 operation과 공식 권한

- `AddNetworkSecurityGroupSecurityRules`: NETWORK_SECURITY_GROUP _UPDATE_SECURITY_RULES and NETWORK_SECURITY_GROUP_INSPECT if writing a rule that specifies a network security group as the source (for ingress rules) or destination (for egress rules) — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `CreateNetworkSecurityGroup`: NETWORK_SECURITY_GROUP_CREATE and VCN_ATTACH — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `DeleteNetworkSecurityGroup`: NETWORK_SECURITY_GROUP_DELETE and VCN_DETACH — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `RemoveNetworkSecurityGroupSecurityRules`: NETWORK_SECURITY_GROUP _UPDATE_SECURITY_RULES — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `UpdateVnic`: VNIC_UPDATE If adding or removing the VNIC from a network security group, also need NETWORK_SECURITY_GROUP_UPDATE_MEMBERS and VNIC_ASSOCIATE_NETWORK_SECURITY_GROUP — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `ListNetworkSecurityGroupSecurityRules`: NETWORK_SECURITY_GROUP_LIST_SECURITY_RULES — [공식 API 권한 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).

위 permission 이름은 IAM policy resource-type이 아니다. 조직 관리자가 기존 group/compartment scope와 conditional dependency를 실제 API 표에 대조한다. 편의 family grant를 최소권한 정책으로 부르지 않으며 실습자는 policy 작성 권한을 자동 인수하지 않는다.

## 비용및주의사항

- 추가 범위는 nsg-client-dir/nsg-server-dir 두 개와 A/B의 전용 packet-dir-27 HTTP8082 process 및 runtime /32 rules다. 기존 두 VM·route를 재사용하고 새 probe VM/NAT를 만들지 않는다.
- 역방향 positive를 먼저 확인한 후 A ingress 부재와 A egress 제거를 별도로 시험한다. 각8초 요청·60초 복구 창이며 양 guest durable firewall ledger/독점 lease가 필수다. owner27-return의 단일 station으로 양쪽 원복·자기 NSG 삭제 후 반환한다.

실제 OCI 작업은 아직 수행하지 않았다. 기대값은 실행 시 대조할 조건이며 무료 사용이나 과금 강제 차단을 보장하지 않는다. 승인 expiry가 끝나면 새 시험을 중단하고 위 소유권 순서로 원복·반환한다.

## 실습구성

### 이번 workshop 시나리오와 입력 계약

신규 연결의 방향과 stateful response를 별도 listener로 비교한다. 

|입력|값/출처|시점|실행 위치|선택 이유|
|---|---|---|---|---|
|`VCN_ID`|owner19 shared-vcn 생성 상세의 OCID; 모든 network target의 VCN 일치 확인|해당 선행 완료 또는 실제 guest 조회 후|본문에 지정한 Console/Cloud Shell 또는 guest| 모든 network target의 VCN 일치 확인|
|`A_VNIC_ID`|owner10 vm-a Attached VNICs의 primary OCID; NSG 배열 변경 대상|해당 선행 완료 또는 실제 guest 조회 후|본문에 지정한 Console/Cloud Shell 또는 guest| NSG 배열 변경 대상|
|`B_VNIC_ID`|owner22 private-vm Attached VNICs의 primary OCID; SSH/HTTP 대상 보존|해당 선행 완료 또는 실제 guest 조회 후|본문에 지정한 Console/Cloud Shell 또는 guest| SSH/HTTP 대상 보존|
|`A_PRIVATE_IP`|owner10 primary GetVnic private-ip; B ingress/guest rule의 source /32|해당 선행 완료 또는 실제 guest 조회 후|본문에 지정한 Console/Cloud Shell 또는 guest| B ingress/guest rule의 source /32|
|`B_PRIVATE_IP`|owner22 primary GetVnic private-ip; A egress/HTTP URL의 대상|해당 선행 완료 또는 실제 guest 조회 후|본문에 지정한 Console/Cloud Shell 또는 guest| A egress/HTTP URL의 대상|

|Console field|실제 입력/선택|이유|
|---|---|---|
|NSG name / direction / destination port|nsg-client-dir/nsg-server-dir; A egress/B ingress TCP8082; B egress A/32 fixed|이번 기능/실패 원인을 한 변수로 분리|

**생성·변경 범위:** nsg-client-dir:NetworkSecurityGroup,nsg-server-dir:NetworkSecurityGroup. **완료 비교:** A→B success; B→A new-flow deny; remove A egress deny→restore. 기대 패턴이며 OCI 실측값이 아니다. **정리:** 본 주제의 변경만 원복하고 전용 신규 resource만 dependency 역순으로 정리한다; 공유 자원은 creator와 consumer registry/expiry 기준으로 반환한다.

`nsg-client-dir`와 `nsg-server-dir`에 TCP8082 규칙을 작성한다. A에서 시작하는 연결만 허용하며 B→A 신규 연결은 허용하지 않는다. Client egress destination port와 server ingress destination port를 맞추고, 응답은 stateful tracking으로 처리한다. 두 방향에 모두 8082를 무조건 여는 방식이 아니다.

## 실습절차

### 명령 실행 gate — 실패하면 다음 생성으로 진행하지 않는다

1. 아래 Bash는 **전용 Bash 세션**에서만 사용한다. Console/Cloud Shell 조회, 본인 로컬 SSH, 로그인한 Oracle Linux guest를 섞지 않는다. `exit 1`은 이 전용 세션을 닫는 안전 중단이다. `<...>`·빈 입력·타 owner OCID가 남으면 실행하지 않는다.
2. 성공을 요구하는 파일 작성/조회/설치/attach 작업은 실패 시 중단한다. 실패한 API 뒤 기존 JSON이나 기존 다운로드 파일을 성공 근거로 읽지 않는다. 기대 deny는 `if command; then ...; else RC=$?; ...; fi`로 별도 기록하고, 실제 timeout/IAM 오류/로컬 파일 오류를 구별한다.
3. 첫 변경 전에 정확한 OCID·owner·현재 설정과 승인 expiry를 저장한다. 공유 자원 변경 중 다른 소비자가 시작하거나 원복값을 확인할 수 없으면 변경을 중단한다. 삭제 성공은 API 수락이 아니라 같은 identity/Region의 정확한 대상 재조회로 판정한다. `NotAuthorizedOrNotFound` 단독은 삭제 증거가 아니다.


### 개별 완주 보정 — 생성·검증·소유권

1번의 양쪽 listener는 HOME root에서 실행하지 않는다. A는 새 전용 ~/packet-dir-27를 만들고 index.html에 oci-direction-lab 한 줄을 넣는다; B도 같은 새 디렉터리에서 실행한다. 역방향 deny 전에 A ingress8082를 임시 source B/32로 허용하여 B→A body 성공을 먼저 기록하고 그 exact rule만 제거한다. 그래야 A host/listener 실패와 OCI ingress 부재를 분리할 수 있다.

### Console 규칙 입력과 원복 ledger — TCP8082

1. **Networking → Virtual cloud networks → shared-vcn → Security → Network security groups**(또는 동일 이름 탭)에서 본문 대상 NSG의 OCID를 연다. Add Rules의 Direction은 표의 방향, Stateless는 해당 단계값, IP Protocol은 TCP(6), Source/Destination Type은 본문 CIDR 또는 같은 VCN NSG를 선택한다. Forward source port=All, destination port=8082이다. /32는 실제 peer private IP이며 internet source는 없다.
2. **Compute → Instances → 대상 → Networking → Attached VNICs → primary → Edit**에서 기존 NSG OCID 배열을 보존하고 본문에서 생성한 NSG만 추가한다. nsg-admin/nsg-private-admin의 SSH22 membership은 유지한다. Rule 제출 뒤 NSG → Security Rules와 VNIC 상세를 다시 읽고 추가 rule ID·stateful flag·source/destination·port를 ledger에 적는다. 원복은 그 ID/배열만 대상으로 한다.
3. 첫 deny 전에 복구 Console 담당자에게 exact 원복 rule/멤버십과 승인된 **60초 최대 시험 창**을 전달한다. 한 번의 8초 유한 요청 뒤 곧바로 원복한다. 진단을 위해 deny 상태를 연장하지 않는다. 원복 후 새 연결 positive가 실패하면 후속 lab/cleanup을 시작하지 말고 listener→guest firewall→OCI union→route 순으로 복구한다.


### Oracle Linux guest positive-control 준비

B guest에서 PEER_IP는 A_PRIVATE_IP다. 첫 mutation 전에 [단일 owner27 cleanup/중단복구 station](#owner27-return)을 담당자에게 전달한다.27의 A guest에서도 역방향 listener를 시험하므로 PEER_IP=B_PRIVATE_IP로 같은 준비를 수행한다. FIREWALL_ZONE은 `firewall-cmd --get-active-zones`에서 primary interface가 연결된 실제 zone으로 지정한다. 비활성 firewalld를 임의 활성화하거나 SELinux를 disable하지 않는다. shell별 RUNTIME_BEFORE/PERMANENT_BEFORE/PENDING/HTTP_RULE 값을 ledger에 보존한다. inactive이면 journal 보존/BLOCKED_FIREWALL로 중단한다.

```bash
export PEER_IP='<상대_VM_actual_private_IP>'
export FIREWALL_ZONE='<이_guest_primary_interface_active_zone>'
python3 - <<'PY'
import ipaddress,os
x=ipaddress.ip_address(os.environ['PEER_IP'])
assert x.version==4 and x.is_private, 'STOP: approved actual private IPv4 required'
PY
if [ "$?" -ne 0 ]; then exit 1; fi
export HTTP_RULE="rule family=\"ipv4\" source address=\"$PEER_IP/32\" port port=\"8082\" protocol=\"tcp\" accept"
STATE27="$HOME/owner27-firewall.state"
STATE="$STATE27"
# 이 exact guest/zone/rule에 대한 독점 owner lease를 승인받는다.
# lease 동안 다른 owner add/remove/reload/restart 금지; 미승인이면 BLOCKED_OWNER.
if test -e "$STATE" || test -L "$STATE"; then exit 1; fi
(umask 077; set -C; printf 'FIREWALL_ZONE=%q\nHTTP_RULE=%q\nLEASE=exclusive\nPENDING=0\n' "$FIREWALL_ZONE" "$HTTP_RULE" > "$STATE") || exit 1
systemctl is-active --quiet firewalld || { printf 'BLOCKED_FIREWALL: preserve ledger\n'; exit 1; }
case "$FIREWALL_ZONE" in ''|*'<'*|*'>'*) exit 1;; esac
sudo firewall-cmd --zone="$FIREWALL_ZONE" --list-all >/dev/null || exit 1
sudo firewall-cmd --permanent --zone="$FIREWALL_ZONE" --list-all >/dev/null || exit 1
query_state() {
  if sudo firewall-cmd "$@" --zone="$FIREWALL_ZONE" --query-rich-rule="$HTTP_RULE"; then QUERY=1;
  else RC=$?; [ "$RC" = 1 ] || return 1; QUERY=0; fi
}
query_state || exit 1; RUNTIME_BEFORE=$QUERY
query_state --permanent || exit 1; PERMANENT_BEFORE=$QUERY
printf 'RUNTIME_BEFORE=%s\nPERMANENT_BEFORE=%s\n' "$RUNTIME_BEFORE" "$PERMANENT_BEFORE" >> "$STATE" || exit 1
sync -f "$STATE" || exit 1
sync -f "$HOME" || exit 1
if [ "$RUNTIME_BEFORE" = 0 ] && [ "$PERMANENT_BEFORE" = 1 ]; then
  printf 'BLOCKED: permanent/runtime divergence; preserve journal\n'; exit 1
fi
if [ "$RUNTIME_BEFORE" = 0 ]; then
  printf 'PENDING=1\n' >> "$STATE" || exit 1
  sync -f "$STATE" || exit 1
  sudo firewall-cmd --zone="$FIREWALL_ZONE" --add-rich-rule="$HTTP_RULE" || exit 1
fi
query_state || exit 1; [ "$QUERY" = 1 ] || exit 1
# PENDING remains durable ownership intent until cleanup; never clear it after add.

```

OCI deny 전 같은 listener/guest firewall에서 positive control이 성공해야 한다. listener는 새 전용 workload 디렉터리의 synthetic index.html로 실행하며 개인 HOME root에서 HTTP server를 열지 않는다. `8082`는 인터넷 source로 허용하지 않는다.

**터미널 실행 구분:** `tcpdump`는 별도 SSH 터미널에서 먼저 시작하고, `curl`/CLI workload는 다른 터미널에서 실행한다. 같은 shell에서 capture 종료를 기다렸다가 요청하면 packet이 수집되지 않는다. 본문의 유한 요청이 실패해도 담당자는 승인 deadline에 해당 HTTP/gateway 변경만 즉시 원복한다.

### 전용 NSG를 먼저 생성하고 실제 OCID를 전달한다

1. Networking → shared-vcn → Network Security Groups → Create Network Security Group. Name `nsg-client-dir`, Compartment lab-root, 규칙 없음으로 Create. 같은 화면에서 `nsg-server-dir`를 한 개 더 만든다. 생성 OCID 두 개를 저장한다. 동일 이름의 타 owner NSG를 재사용하지 않는다.
2. A primary VNIC의 기존 배열에 nsg-client-dir, B primary에 nsg-server-dir만 추가한다. 아래 direction rule은 이들 두 NSG에 넣는다. Create/Add/association을 각각 readback한다.
3. 아래 A의 listener/guest firewall/B egress 구성까지 끝난 뒤, B.5 negative **직전에만** 역방향 positive control을 수행한다. B→A 역방향 positive control에는 nsg-client-dir **Ingress/stateful/source B_PRIVATE_IP/32/TCP/source All/destination8082** 한 개만 임시 추가한다. B egress 및 두 listener/guest firewall은 고정한 채 B의 `/index.html` 요청에서 `oci-direction-lab`을 확인한다. 그 ingress ID만 제거한 뒤 본문 B→A negative로 진행한다. 복구 확인 때 다시 같은 ingress를 넣어 성공을 확인하고 제거한다.

### A. 방향을 기준으로 규칙 구성

```bash
# 양 guest에서 각각 실행: 기존 directory를 덮어쓰지 않는다
if test -e "$HOME/packet-dir-27" || test -L "$HOME/packet-dir-27"; then printf '%s\n' 'STOP: workload directory exists'; exit 1; fi
mkdir -m 700 "$HOME/packet-dir-27" || exit 1
(umask 077; set -C; printf '%s\n' 'oci-direction-lab' > "$HOME/packet-dir-27/index.html") || exit 1
# HTTP server는 아래1번에서 별도 terminal로 시작
```

1. B에서 `cd "$HOME/packet-dir-27" && python3 -m http.server 8082 --bind 0.0.0.0` 실행. A에서도 다른 터미널로 동일 포트 listener를 실행한다. 양쪽 `ss -lntp` 결과를 남긴다.
2. 두 guest의 firewalld에 source 상대 private /32 TCP8082 runtime 허용을 준비하고 listener/localhost 성공을 확인한다. A→B 및 B→A의 guest 조건을 같게 유지해야 역방향 실패를 OCI ingress 부재로 해석할 수 있다. 관리 SSH TCP22 egress/ingress는 제거하지 않는다. **VCN → NSG → Add Rules**에서 A용 NSG에는 stateful **Egress**, destination CIDR `<B_PRIVATE_IP>/32`, protocol TCP, source All, destination8082를 넣는다. B용 NSG에는 stateful **Ingress**, source CIDR `<A_PRIVATE_IP>/32`, TCP source All/destination8082를 넣는다. 역방향 negative를 A ingress 부재로 분리하려면 B NSG에 stateful Egress destination A/32 TCP8082만 추가해 고정한다. A ingress8082는 추가하지 않는다. 해당 VNIC에 NSG를 연결한다.
3. 아래 reverse positive-control 창을 제외한 negative 상태에서만 A ingress8082를 비운다. A의 SL/다른 NSG의 all-egress는 실험 동안 제거 또는 목적지/port 범위를 제한한다. SSH 응답용 tracked flow가 유지되는지 확인하고 관리자 Console 복구 경로를 열어둔다. B→A8082 ingress는 어디에도 추가하지 않는다.

### B. 요청/응답과 신규 역방향 구별
4. A에서 `curl --noproxy '*' --connect-timeout 3 --max-time 8 http://<B_PRIVATE_IP>:8082/index.html` 실행. B에서 `sudo timeout 20s tcpdump -ni any 'tcp port 8082'`로 요청과 response tuple을 확인한다. Response source port는8082, destination은 client ephemeral port다.
5. nsg-client-dir에 임시 stateful Ingress source B/32 TCP destination8082를 추가하고 rule ID를 기록한다. B→A 새 요청의 body=oci-direction-lab 성공을 먼저 확인 → 그 ID만 제거/readback → 아래 신규 deny 요청 → 60초 이내 동일 ingress 복원/새 요청 body 성공 → 복원용 새 ID만 제거/readback 순서로 수행한다. listener/B egress/guest firewall은 고정한다. 각 curl RC와 HTTP body를 기록한다. B에서 `curl --noproxy '*' --connect-timeout 3 --max-time 8 http://<A_PRIVATE_IP>:8082/index.html` 실행. A ingress8082가 없으므로 **신규 역방향**은 실패해야 한다. 정상 response가 이 요청의 허용을 의미하지 않는다.
6. A의 test egress8082를 제거하고 A→B 새 요청도 실패하는지 확인한다. 다시 rule을 넣고 성공을 회복한다. B ingress만으로 A egress가 자동 허용되는 것은 아니다.

## 결과검증

<a id="baseline-27-ready"></a>

완료 단계 **DIRECTION-tested**: A→B 성공, B→A 신규 연결 실패, A egress 제거 후 실패·복구 성공을 남긴다. SYN의 source/destination IP/port와 SYN-ACK 반대 tuple을 packet으로 기록한다. Source port8082를 client 요청 규칙에 잘못 지정하면 실패하므로 destination field를 확인한다.

**실행 기록(아직 미작성):** 실행자/Region/시각, exact resource OCID, 설정 전·후, 성공 출력, deny 오류·exit code/request-id, positive control, 정리 결과. 문서 검증은 실제 OCI 실행 성공을 뜻하지 않는다.

### 이 시나리오의 functional acceptance

| Positive | Negative / 미완료 판정 | Recovery / 반환 |
|---|---|---|
| A→B 및 임시 ingress 역방향 positive | A ingress 없을 때 B→A 신규 deny; A egress 제거 후 A→B deny | 각 exact rule 복구/새 connection body; SSH22 고정 |

모두 실행 시 확인할 기대 패턴이다. 생성 전 guard negative는 OCI 장애를 주입한 것과 다르며 다른 자원을 추가하지 않는다. 오류 메시지와 실제 exit code가 없는 항목은 NOT_EXECUTED 또는 BLOCKED로 남긴다.

## 문제해결

egress 제거 후에도 성공하면 default SL all-egress, 다른 NSG, proxy, 기존 연결 추적을 확인한다. DNS 실패를 egress8082 차단으로 해석하지 않도록 HTTP는 IP를 직접 사용한다. 역방향 실패는 A listener 정상과 A→B 성공을 확인하고 호스트 firewall 변경이 없음을 남겨 OCI 규칙과 구별한다.

## 리소스정리

<a id="owner27-return"></a>

**단일 owner27 cleanup/중단복구 station:** 첫 firewall 변경 전에 양 guest의 identity/zone/exact rule, OCI rule ID·VNIC/SL/NSG 원본 배열, listener terminal, 독점 lease ID/담당자/expiry를 저장하고 이 station을 복구 담당자에게 전달한다. add 후 terminal loss/SIGKILL도 새 guest terminal에서 아래 같은 block을 사용한다. lease 만료·동시 writer·reload가 있으면 `BLOCKED_OWNER`로 journal을 보존하고 owner와 조정한다. `DIRECTION-tested`는 기능 checkpoint일 뿐 반환완료가 아니다. 양 guest/OCI cleanup과 SSH22·owner24 HTTP 재검증의 실제 기록이 모두 있어야28 진입을 승인한다.

1. A/B 각 server terminal에서 본인8082 process만 Ctrl-C 후 `ss -lntp '( sport = :8082 )'`에 listener 부재를 확인한다. owner24의8080은 중지하지 않는다.
2. 각 guest에서 자기 owner27-firewall.state의 exact zone/source/8082/RUNTIME_BEFORE/PERMANENT_BEFORE/PENDING를 읽고 아래 fenced 원복을 수행한다.
3. Console에서 원래 SL/NSG 배열 복원 → 두 임시 NSG의 VNIC 소속 제거 → members 빈 readback → exact nsg-client-dir/nsg-server-dir 삭제 순서다. A/B primary SSH와 owner24 HTTP를 재확인한다.
4. 파일 삭제도 아래 fenced block에서 한 번만 수행한다. 다른 파일로 rmdir가 실패하면 중단하고 타 파일을 삭제하지 않는다.

```bash
# A 및 B guest 각각; 직접 작성한 private ledger만 읽는다
STATE="$HOME/owner27-firewall.state"
test -f "$STATE" && test ! -L "$STATE" && test -O "$STATE" || exit 1
test "$(stat -c %a "$STATE")" = 600 || exit 1
# Inspect exact own journal before source; do not source an untrusted file.
source "$STATE" || exit 1
[ "$LEASE" = exclusive ] || exit 1
case "$RUNTIME_BEFORE:$PERMANENT_BEFORE:$PENDING" in 0:0:0|0:0:1|0:1:0|1:0:0|1:1:0) ;; *) exit 1;; esac
# Confirm lease remained exclusive; reload/concurrent mutation means BLOCKED_OWNER.
systemctl is-active --quiet firewalld || exit 1
sudo firewall-cmd --zone="$FIREWALL_ZONE" --list-all >/dev/null || exit 1
sudo firewall-cmd --permanent --zone="$FIREWALL_ZONE" --list-all >/dev/null || exit 1
query_state() {
  if sudo firewall-cmd "$@" --zone="$FIREWALL_ZONE" --query-rich-rule="$HTTP_RULE"; then QUERY=1;
  else RC=$?; [ "$RC" = 1 ] || return 1; QUERY=0; fi
}
query_state --permanent || exit 1
[ "$QUERY" = "$PERMANENT_BEFORE" ] || { printf 'BLOCKED: permanent changed; preserve journal\n'; exit 1; }
query_state || exit 1
if [ "$RUNTIME_BEFORE" = 0 ] && [ "$PENDING" = 1 ] && [ "$QUERY" = 1 ]; then
  sudo firewall-cmd --zone="$FIREWALL_ZONE" --remove-rich-rule="$HTTP_RULE" || exit 1
fi
query_state || exit 1
[ "$QUERY" = "$RUNTIME_BEFORE" ] || exit 1
rm -- "$STATE" || exit 1
# Setup가 app 생성 전에 중단됐으면 없는 directory는 정상이다.
# 생성된 경우 본인 synthetic file만 제거; symlink/타 파일이면 중단한다.
if test -e "$HOME/packet-dir-27" || test -L "$HOME/packet-dir-27"; then
  test -d "$HOME/packet-dir-27" && test ! -L "$HOME/packet-dir-27" && test -O "$HOME/packet-dir-27" || exit 1
  if test -e "$HOME/packet-dir-27/index.html" || test -L "$HOME/packet-dir-27/index.html"; then
    test -f "$HOME/packet-dir-27/index.html" && test ! -L "$HOME/packet-dir-27/index.html" && test -O "$HOME/packet-dir-27/index.html" || exit 1
    test "$(cat "$HOME/packet-dir-27/index.html")" = oci-direction-lab || exit 1
    rm -- "$HOME/packet-dir-27/index.html" || exit 1
  fi
  rmdir -- "$HOME/packet-dir-27" || exit 1
fi
```
 후속 28에서 동일 rule을 stateless로 바꾸지 말고 새로운 전용 포트와 명시적 reverse rule을 사용한다. VM은 보존한다.

**guest 원복 완료 판정:** 위 fenced block의 runtime=RUNTIME_BEFORE 및 permanent 불변과 ledger 제거가 유일한 guest cleanup이다. 다시 remove/source하지 않는다. permanent/전체 ruleset은 수정하지 않는다. OCI는 자기 rule ID만 제거하고 공유 VM/네트워크는 원래 creator에게 반환한다.


## 공식문서

### HTTP workload의 공식 Oracle 근거와 적용 범위

[Oracle Linux 공식 HTTP 두 host 예제](https://docs.oracle.com/en/operating-systems/oracle-linux/mptcp/mptcp-testing_mptcp_subflows.html)는 B에서 `python3 -m http.server`를 실행하고 A에서 curl로 확인한다. [Oracle Linux9 공식 예제의 Python HTTP root](https://docs.oracle.com/en/learn/ol-podman-intro/)는 Oracle Linux9 image의 Python 표준 HTTP server와 working directory를 document root로 사용한다. 이 lab은 **MPTCP/Podman/container를 도입하지 않고**, 같은 HTTP 진단 원리만 기존 Oracle Linux9 guest에 적용한다. 본문의8080–8084, 명시적 index.html/health.txt, runtime /32 firewall 및 복구 trap은 이 lab의 제한접근·소유권 요구에 맞춘 저술값이며 공식 문서의 public80/root directory listing 값을 복사하지 않는다. 시작 전 `python3 -m http.server --help`에서 실제 guest의 `--bind` 지원을 확인하고 미지원이면 중단한다. Oracle Linux9/OCI에서 실행 결과는 아직 없다.


아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 아래 본문은 각 생성 필드·설정·검증·정리의 기술 근거이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [30] [Networking Overview — Ingress / Egress](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/overview.htm)
  - 확인 범위: SECURITY RULES.

### 실습 작업을 지원하는 추가 공식 본문

- [실습근거 1] [Security Rules](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/securityrules.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
