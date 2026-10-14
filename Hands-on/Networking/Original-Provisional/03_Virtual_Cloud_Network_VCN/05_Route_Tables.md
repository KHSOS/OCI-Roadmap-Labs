> **기존 실습 초안 — 미검증**: 원래 93개 문서의 절차를 보존한 자료입니다. 새 튜토리얼의 품질 승인이나 OCI 실행 검증을 뜻하지 않습니다.

# Route Tables

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../../../../Docs/README.md)를 확인하세요.

> **저술 완료 · OCI 실행 미검증** — Oracle 공식 작업 문서를 근거로 작성한 한국어 hands-on입니다. Oracle 공인 교육/실행 인증이 아니며 기대 결과와 실제 결과를 구분합니다. 비용 승인과 실제 선행 산출물 확인 후에만 실행하세요.

## 학습목표

- 기존 A/B와 route associations를 조회하고 B에 전용 TCP8080 HTTP workload·health file·필요한 /32 OCI/guest 허용을 구성한다.
- 내부 HTTP/health 성공과 public 외부 HTTPS 성공·private 외부 HTTPS 실패를 구별하고 health 200→404→200 복구를 확인한다.

## 대응 Roadmap 항목

- 항목명: **Route Tables**
- Roadmap 경로: **Virtual Cloud Network (VCN) → Route Tables**
- Node ID: `sequence-3-5`
- 상위 항목: **Virtual Cloud Network (VCN)** (`sequence-3-0`)
- 문서 범위: 기존 A/B와 route associations를 조회하고 B에 전용 TCP8080 HTTP workload·health file·필요한 /32 OCI/guest 허용을 구성한다. 내부 HTTP/health 성공과 public 외부 HTTPS 성공·private 외부 HTTPS 실패를 구별하고 health 200→404→200 복구를 확인한다. 실제 OCI 실행 결과가 아니라 이 절차의 완료 조건이다.
- [전체 항목 목록](../../../../Docs/README.md)

## 사전준비

### 기준 계약과 실제 단계 선행조건

[공통 baseline-contract.json](../../../../baseline-contract.json)의 OS/CIDR/소유권을 따른다. Oracle Linux 9 / `opc` / `dnf` 외의 숨은 Ubuntu 또는 probe VM은 만들지 않는다. OCID/IP의 null은 실행 전 반드시 실제 출력으로 채우는 **실행 차단 입력**이다. approval/capacity/quota/identity 미확인은 완료가 아니다.

- [19 Virtual Cloud Network (VCN)](00_Virtual_Cloud_Network_VCN.md#baseline-19-ready): **AVAILABLE shared-vcn VCN_ID와 default DHCP OCID**를 실제 출력으로 인수한다. 이 문서의 변경/반환까지 creator 승인 expiry 안에서 보존한다.
- [21 Subnets](02_Subnets.md#baseline-21-ready): **AVAILABLE private-app SUBNET_PRIVATE_ID, rt-private/sl-private OCID와 빈 rule 배열**를 실제 출력으로 인수한다. 이 문서의 변경/반환까지 creator 승인 expiry 안에서 보존한다.
- [22 Private Subnets](03_Private_Subnets.md#baseline-22-ready): **INSTANCE_B_ID/B_PRIVATE_IP/public-ip=null, 로컬 jump config와 새 ProxyJump SSH 성공**를 실제 출력으로 인수한다. 이 문서의 변경/반환까지 creator 승인 expiry 안에서 보존한다.
- [10 Compute](../../../Original-Provisional/02_Compute/00_Compute.md#baseline-10-ready): **INSTANCE_A_ID, primary VNIC의 A_PUBLIC_IP/A_PRIVATE_IP와 새 opc SSH 성공**를 실제 출력으로 인수한다. 이 문서의 변경/반환까지 creator 승인 expiry 안에서 보존한다.

공유 A/B consumer는 INSTANCE_A_ID에 owner10 vm-a의 실제 OCID, INSTANCE_B_ID에 owner22 private-vm의 실제 OCID, `A_PUBLIC_IP`, `A_PRIVATE_IP`, `B_PRIVATE_IP`를 실제 생성 응답에서 전달받는다. 해당 주제에 필요하지 않은 VM/변수는 생성·요구하지 않는다.

- [VCN](00_Virtual_Cloud_Network_VCN.md#baseline-19-ready) 완료: `shared-vcn`, `10.77.0.0/16`, DNS label roadmap, AVAILABLE.
- [Public Subnets 준비](04_Public_Subnets.md#public-network-preparation) 완료: `public-admin` / `10.77.10.0/24`, `rt-public`, `igw-lab`, empty `sl-public`, `nsg-admin`.
- [Compute](../../../Original-Provisional/02_Compute/00_Compute.md#baseline-10-ready) 완료: Oracle Linux 9 `vm-a`, opc SSH, INSTANCE_A_ID/A_PUBLIC_IP/A_PRIVATE_IP. [Private Subnets](03_Private_Subnets.md#baseline-22-ready) 완료: Oracle Linux 9 `private-vm`, public-ip=null, INSTANCE_B_ID/B_PRIVATE_IP와 ProxyJump 성공. private-app/rt-private/sl-private는 [Subnets](02_Subnets.md#baseline-21-ready) owner21의 실제 생성값을 사용한다.
- [SSH Keys 준비](../../../Original-Provisional/02_Compute/05_SSH_Keys.md#key-preparation)의 KEY와 owner22의 `oci-roadmap-jump.conf`를 로컬에만 보관한다. 없으면 해당 owner 절차로 돌아간다. 이 문서에서는 VM/VCN/Subnet/IGW/Route Table을 추가 생성하지 않는다.
- VM 두 개 비용·패키지 다운로드용 잠깐의 NAT 비용과 종료시각을 승인한다. NAT 생성은 owner30의 준비 단계만 허용하며 NAT-tested를 이 lab의 선행으로 요구하지 않는다. 공유 자원은 25~32/36/42/43~46의 예약 소비자 종료까지 보존한다.

## 권한

기존 A/B subnet·RT·IGW 조회, 자기 HTTP8080 NSG rule add/remove/read와 실제 association이 틀린 경우의 승인된 UpdateSubnet 복구만 필요하다. 이 문서에서는 CreateRouteTable/CreateVcn/VM launch를 실행하지 않는다. guest package 준비·runtime firewall·합성 파일/server는 opc/sudo 및 host owner 승인 범위다.

### 실제 operation과 공식 권한

- `AddNetworkSecurityGroupSecurityRules`: NETWORK_SECURITY_GROUP _UPDATE_SECURITY_RULES and NETWORK_SECURITY_GROUP_INSPECT if writing a rule that specifies a network security group as the source (for ingress rules) or destination (for egress rules) — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `GetInternetGateway`: INTERNET_GATEWAY_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `GetRouteTable`: ROUTE_TABLE_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `GetSubnet`: SUBNET_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `RemoveNetworkSecurityGroupSecurityRules`: NETWORK_SECURITY_GROUP _UPDATE_SECURITY_RULES — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `UpdateSubnet`: SUBNET_UPDATE If changing which route table is associated with the subnet, also need ROUTE_TABLE_ATTACH and ROUTE_TABLE_DETACH If changing which security lists are associated with the subnet, also need SECURITY_LIST_ATTACH and SECURITY_LIST_DETACH If changing which set of DHCP options are associated with the subnet, also need DHCP_ATTACH and DHCP_DETACH — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `ListNetworkSecurityGroupSecurityRules`: NETWORK_SECURITY_GROUP_LIST_SECURITY_RULES — [공식 API 권한 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).

위 permission 이름은 IAM policy resource-type이 아니다. 조직 관리자가 기존 group/compartment scope와 conditional dependency를 실제 API 표에 대조한다. 편의 family grant를 최소권한 정책으로 부르지 않으며 실습자는 policy 작성 권한을 자동 인수하지 않는다.

## 비용및주의사항

- 신규 VM/VCN/IGW/Route Table은 없다. A/B 기존 두 VM을 승인 창에서 재사용하고 B의 packet-lab HTTP8080/health.txt와 /32 허용만 추가한다. 도구가 없을 때만 owner30의 승인 설치용 NAT를 잠깐 사용하고 RT-ready 전 자기 default route/egress를 제거한다.
- 내부 HTTP와 외부 TLS는 각 상태1~3회, 유한 timeout으로 시험한다. 빈 private route와 sender egress 부재는 combined isolation이지 route-only 증명이 아니다. health rename은 consumer 중단/guarded recovery 뒤 수행하며 예약 app 소비자 반환 전 파일/server를 정리하지 않는다.

실제 OCI 작업은 아직 수행하지 않았다. 기대값은 실행 시 대조할 조건이며 무료 사용이나 과금 강제 차단을 보장하지 않는다. 승인 expiry가 끝나면 새 시험을 중단하고 위 소유권 순서로 원복·반환한다.

## 실습구성

### 이번 workshop 시나리오와 입력 계약

public/private route를 분리하고 B의 전용 HTTP workload를 실제 실행한다. 

|입력|값/출처|시점|실행 위치|선택 이유|
|---|---|---|---|---|
|`VCN_ID`|owner19 shared-vcn 생성 상세의 OCID; 모든 network target의 VCN 일치 확인|해당 선행 완료 또는 실제 guest 조회 후|본문에 지정한 Console/Cloud Shell 또는 guest| 모든 network target의 VCN 일치 확인|
|`SUBNET_PRIVATE_ID`|owner21 private-app 생성 OCID; route/SL association 조회 대상|해당 선행 완료 또는 실제 guest 조회 후|본문에 지정한 Console/Cloud Shell 또는 guest| route/SL association 조회 대상|
|`INSTANCE_A_ID`|owner10의 actual output; `<...>` 대체 필수|실행 전 선행 완료값|Console/사용자 전용 shell|정확한 대상/소유관계 고정|
|`INSTANCE_B_ID`|owner22의 actual output; `<...>` 대체 필수|실행 전 선행 완료값|Console/사용자 전용 shell|정확한 대상/소유관계 고정|
|`A_PRIVATE_IP`|owner10 primary GetVnic private-ip; B ingress/guest rule의 source /32|해당 선행 완료 또는 실제 guest 조회 후|본문에 지정한 Console/Cloud Shell 또는 guest| B ingress/guest rule의 source /32|
|`B_PRIVATE_IP`|owner22 primary GetVnic private-ip; A egress/HTTP URL의 대상|해당 선행 완료 또는 실제 guest 조회 후|본문에 지정한 Console/Cloud Shell 또는 guest| A egress/HTTP URL의 대상|

|Console field|실제 입력/선택|이유|
|---|---|---|
|Route association / destination / target|public-admin→rt-public; private-app→rt-private; private explicit routes=[]|이번 기능/실패 원인을 한 변수로 분리|

**생성·변경 범위:** packet-workload:GuestWorkload,HTTP-runtime-rule:GuestFirewallRule. **완료 비교:** A→B HTTP success; A external TLS success; B external TLS deny. 기대 패턴이며 OCI 실측값이 아니다. **정리:** 본 주제의 변경만 원복하고 전용 신규 resource만 dependency 역순으로 정리한다; 공유 자원은 creator와 consumer registry/expiry 기준으로 반환한다.

|역할|선택값|선택 이유|
|---|---|---|
|Public route table|`rt-public`, `0.0.0.0/0 → igw-lab`|public VM만 직접 인터넷 사용|
|Private route table|`rt-private`, 시작 시 빈 명시적 rule|인터넷 차단과 VCN local routing 분리|
|Public VM|Oracle Linux 9, opc, Compute의 승인된 최소 Shape, public IPv4|관리 접속 및 packet client|
|Private VM|동일 이미지, public IP 없음, private IP 예: `10.77.20.10`|HTTP server 및 NAT/SGW 테스트|

IGW/두 Route Table은 owner23/21에서 이미 생성한 OCID를 조회한다. 없으면 그 owner의 실제 준비 절차로 돌아간다. NAT/SGW/DRG를 자동 추가하는 VCN Wizard는 사용하지 않는다. VM `vm-a` 사설 IP 예시는 `10.77.10.10`; 실제 자동 할당값을 기록한다.

## 실습절차

### 작업 순서 체크포인트
동일 A/B HTTP positive를 유지하면서 private external request를 combined isolation으로 확인한다. DNS/transport/local errors를 기록하고 route-only 성공으로 바꾸지 않는다. 단계 실패는 다음 생성으로 넘기지 않고 같은 target의 복구/반환으로 처리한다.

### 명령 실행 gate — 실패하면 다음 생성으로 진행하지 않는다

1. 아래 Bash는 **전용 Bash 세션**에서만 사용한다. Console/Cloud Shell 조회, 본인 로컬 SSH, 로그인한 Oracle Linux guest를 섞지 않는다. `exit 1`은 이 전용 세션을 닫는 안전 중단이다. `<...>`·빈 입력·타 owner OCID가 남으면 실행하지 않는다.
2. 성공을 요구하는 파일 작성/조회/설치/attach 작업은 실패 시 중단한다. 실패한 API 뒤 기존 JSON이나 기존 다운로드 파일을 성공 근거로 읽지 않는다. 기대 deny는 `if command; then ...; else RC=$?; ...; fi`로 별도 기록하고, 실제 timeout/IAM 오류/로컬 파일 오류를 구별한다.
3. 첫 변경 전에 정확한 OCID·owner·현재 설정과 승인 expiry를 저장한다. 공유 자원 변경 중 다른 소비자가 시작하거나 원복값을 확인할 수 없으면 변경을 중단한다. 삭제 성공은 API 수락이 아니라 같은 identity/Region의 정확한 대상 재조회로 판정한다. `NotAuthorizedOrNotFound` 단독은 삭제 증거가 아니다.


### 개별 완주 보정 — 생성·검증·소유권

B HTTP rich-rule 추가는 firewalld가 active인 경우에만 수행한다. inactive이면 journal을 보존하고 BLOCKED_FIREWALL로 중단한다; 활성화하지 않는다. 기존 rule query가 yes이면 이 lab 소유로 삭제하지 않는다.

**실행 위치:** 로컬 SSH 접속 명령, vm-a client shell, private-vm server shell은 별도 터미널이다. capture를 먼저 시작하고 요청을 다른 터미널에서 실행한다.

### A. 기존 Oracle Linux VM 준비 (VM 생성 없음)
1. creator readback을 대조한다. A=`vm-a`, B=`private-vm`, 둘 다 Oracle Linux 9/opc. primary VNIC/Subnet/AD/architecture와 실제 IP를 기록한다. `ssh -i "$KEY" opc@"$A_PUBLIC_IP"`, `ssh -F "$HOME/.ssh/oci-roadmap-jump.conf" roadmap-private`로 각 새 세션 성공을 확인한다.
2. A의 nsg-admin에는 B/32 TCP22 egress, B의 nsg-private-admin에는 A/32 TCP22 ingress가 이미 있어야 한다. 이를 유지한다. HTTP용 규칙은 기존 A `nsg-admin`에 stateful Egress destination B_PRIVATE_IP/32 TCP/source All/destination8080, 기존 B `nsg-private-admin`에 stateful Ingress source A_PRIVATE_IP/32 TCP/source All/destination8080을 추가한다. 신규 NSG는 만들지 않는다. Networking → shared-vcn → Network Security Groups → 각각 exact NSG → Add Rules에서 입력하고 추가 rule ID와 기존 배열을 기록한다. sl-public/sl-private는 빈 상태로 유지한다. all-egress/all-ingress로 덮어쓰지 않는다.
3. A/B guest에서 필요한 도구를 확인한다. 사전 설치됐으면 dnf를 실행하지 않는다. B 설치가 필요하면 [NAT 준비](11_NAT_Gateway.md#nat-installation-preparation)의 gateway/route/TCP443만 잠깐 승인해 먼저 설치하고 RT-ready 검사 전에 route와 임시 egress를 제거한다. repository가 HTTP80/다른 endpoint를 요구하면 제한적 추가 허용을 별도 승인하고 rule ID를 기록한다. 설치 실패를 route deny로 해석하지 않는다.

```bash
# 각 Oracle Linux guest에서 실행; 승인된 다운로드 경로가 있을 때만 설치
command -v python3
command -v curl
command -v tcpdump
# 없는 package만 설치; 승인된 download path가 있는 경우에만
MISSING=()
for package in python3 curl tcpdump; do command -v "$package" >/dev/null 2>&1 || MISSING+=("$package"); done
if [ "${#MISSING[@]}" -gt 0 ]; then sudo dnf install -y "${MISSING[@]}" || exit 1; fi
cat /etc/os-release
```

4. B의 실제 A_PRIVATE_IP와 guest firewall active zone을 확인한다. firewalld가 active면 전용 runtime rich-rule `source A/32, TCP8080 accept`를 추가한다. 기존 동일 rule이 있으면 추가·삭제하지 않고 기존 owner를 기록한다. 규칙 추가 전/후 `--query-rich-rule`과 `--list-rich-rules`를 저장한다. disabled firewall을 켜거나 전체 disable하지 않는다. 이 host firewall 조건은 25의 NSG deny 동안 고정한다.

```bash
# B guest; 값을 실제 vm-a private IP로 설정
export A_PRIVATE_IP='<owner10_VNIC_private_ip>'
python3 - <<'PY'
import ipaddress,os
x=ipaddress.ip_address(os.environ['A_PRIVATE_IP'])
assert x.version==4 and x.is_private, 'STOP: approved actual private IPv4 required'
PY
if [ "$?" -ne 0 ]; then exit 1; fi
export HTTP_RULE="rule family=\"ipv4\" source address=\"$A_PRIVATE_IP/32\" port port=\"8080\" protocol=\"tcp\" accept"
# active zone 이름은 firewall-cmd --get-active-zones와 primary interface로 확인
export FIREWALL_ZONE='<B_primary_interface_active_zone>'
LEDGER24="$HOME/owner24-firewall.state"
STATE="$LEDGER24"
# 이 exact guest/zone/rule에 대한 독점 owner lease를 승인받는다.
# lease 동안 다른 owner add/remove/reload/restart 금지; 미승인이면 BLOCKED_OWNER.
if test -e "$STATE" || test -L "$STATE"; then exit 1; fi
(umask 077; set -C; printf 'FIREWALL_ZONE=%q\nHTTP_RULE=%q\nLEASE=exclusive\nPENDING=0\nRUNTIME_BEFORE=0\nPERMANENT_BEFORE=0\n' "$FIREWALL_ZONE" "$HTTP_RULE" > "$STATE") || exit 1
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
if [ "$RUNTIME_BEFORE" = 0 ]; then
  printf 'PENDING=1\n' >> "$STATE" || exit 1
  sync -f "$STATE" || exit 1
  sudo firewall-cmd --zone="$FIREWALL_ZONE" --add-rich-rule="$HTTP_RULE" || exit 1
fi
query_state || exit 1; [ "$QUERY" = 1 ] || exit 1
# PENDING remains durable ownership intent until cleanup; never clear it after add.
# B guest 원래 shell 종료 후에도 이 ledger만 cleanup terminal로 전달한다. 실패 시 덮어쓰지 않는다.
# B server shell: 새 전용 디렉터리만 사용
if test -e "$HOME/packet-lab"; then printf '%s\n' 'STOP: select a new dedicated workload directory'; exit 1; fi
mkdir "$HOME/packet-lab" || exit 1
printf '%s\n' 'oci-packet-lab' > "$HOME/packet-lab/index.html" || exit 1
printf '%s\n' 'packet-health-ok' > "$HOME/packet-lab/health.txt" || exit 1
cd "$HOME/packet-lab" || exit 1
python3 -m http.server 8080 --bind 0.0.0.0
```

### A-2. 실제 listener/health를 먼저 확인하고 유한 복구를 준비한다

1. B의 두 번째 guest 터미널에서 `ss -lntp '( sport = :8080 )'`로 Python listener를 확인한다. `curl --noproxy '*' --fail --connect-timeout 3 --max-time 8 http://127.0.0.1:8080/index.html`은 `oci-packet-lab`, `/health.txt`는 `packet-health-ok`여야 한다. 설치 성공이나 포트 열림만으로 app 준비가 아니다.
2. 다른 consumer가 이 health를 사용하는 동안 failure를 시작하지 않는다. 아래는 B guest에서 **그대로 한 번** 실행하는 bounded exercise다. trap을 첫 rename 이전에 등록한다. 실패하더라도 본인이 이동한 파일만 복구한다. 서버 프로세스를 죽이거나 SSH22를 변경하지 않는다.

```bash
# B guest, server는 다른 터미널에서 계속 실행 중
(
set -euo pipefail
cd "$HOME/packet-lab"
test -f health.txt && test ! -L health.txt && test ! -e health.txt.owner24-disabled && test ! -L health.txt.owner24-disabled
MOVED=0
restore_health() {
  if [ "$MOVED" = 1 ]; then
    test -f health.txt.owner24-disabled && test ! -L health.txt.owner24-disabled || return 1
    test ! -e health.txt && test ! -L health.txt || return 1
    mv -T -- health.txt.owner24-disabled health.txt || return 1
    MOVED=0
  fi
}
trap restore_health EXIT
trap 'exit 130' INT TERM
CODE=$(curl --noproxy '*' -sS --connect-timeout 3 --max-time 8 -o /dev/null -w '%{http_code}' http://127.0.0.1:8080/health.txt)
test "$CODE" = 200
mv -T -- health.txt health.txt.owner24-disabled
MOVED=1
CODE=$(curl --noproxy '*' -sS --connect-timeout 3 --max-time 8 -o /dev/null -w '%{http_code}' http://127.0.0.1:8080/health.txt)
test "$CODE" = 404
restore_health
CODE=$(curl --noproxy '*' -sS --connect-timeout 3 --max-time 8 -o /dev/null -w '%{http_code}' http://127.0.0.1:8080/health.txt)
test "$CODE" = 200
test "$(curl --noproxy '*' -fsS --connect-timeout 3 --max-time 8 http://127.0.0.1:8080/health.txt)" = packet-health-ok
printf '%s\n' 'Expected health transition validated by this invocation: 200 -> 404 -> 200'
)
```

**SIGKILL/세션 손실 후 owner 승인 새 B 터미널 복구:** 같은 packet-lab와 saved 파일 소유권을 확인한다. destination 충돌 시 삭제/덮어쓰기하지 않고 BLOCKED_RECOVERY. 다음은 위와 동일 helper이며 정상 복구 뒤 반복하지 않는다.
```bash
cd "$HOME/packet-lab" || exit 1
MOVED=1
restore_health() {
  if [ "$MOVED" = 1 ]; then
    test -f health.txt.owner24-disabled && test ! -L health.txt.owner24-disabled || return 1
    test ! -e health.txt && test ! -L health.txt || return 1
    mv -T -- health.txt.owner24-disabled health.txt || return 1
    MOVED=0
  fi
}
restore_health || exit 1
```

3. A guest에서 실제 B 주소의 `/index.html`과 `/health.txt`를 각각 요청한다. `HEALTH=$(curl --noproxy '*' -fsS --connect-timeout 3 --max-time 8 "http://$B_PRIVATE_IP:8080/health.txt") || exit 1` 뒤 `test "$HEALTH" = packet-health-ok || exit 1`로 health bytes도 검사한다. 내부 positive가 성공하기 전 private internet deny를 해석하지 않는다. 코드는 조건 충족 시 출력하는 authored pattern이며 현재 OCI/guest에서 관측된 출력이 아니다.

### B. 기존 route association readback (중복 생성 없음)
5. Console → shared-vcn → Internet Gateways에서 owner23 IGW OCID/Enabled를 읽는다. Route Tables에서 owner23 `rt-public`의 `0.0.0.0/0 → IGW_ID`, owner21 `rt-private`의 `route-rules=[]`를 읽는다. VCN local route를 IGW로 보내는 rule은 만들지 않는다.
6. public-admin/private-app에 각각 해당 table이 연결돼 있는지 확인한다. 기존 VNIC/IP별 route override가 있다면 운영값을 삭제하지 말고 전용 lab VNIC임을 확인한 후 원복 계획을 승인받는다. owner 생성값과 달라졌으면 기존 rule/association을 보존한 뒤 정확한 OCID로 복구한다.

### C. 내부 HTTP / 외부 HTTPS 분리
7. B 별도 capture 터미널에서 먼저 `sudo timeout 20s tcpdump -ni any 'tcp port 8080' -c 12`를 실행한다. A guest에서는 다음을 실행한다.

```bash
export B_PRIVATE_IP='<owner22_VNIC_private_ip>'
if CODE=$(curl --noproxy '*' -sS --connect-timeout 3 --max-time 8 -w '\n%{http_code}' "http://$B_PRIVATE_IP:8080/index.html"); then RC=0; else RC=$?; printf 'HTTP RC=%s\n' "$RC"; exit 1; fi
printf 'HTTP RC=%s response=%s\n' "$RC" "$CODE"
[ "$CODE" = $'oci-packet-lab\n\n200' ] || exit 1
if CODE=$(curl --noproxy '*' -4 -sS -I --connect-timeout 3 --max-time 8 -w '%{http_code}' https://www.oracle.com/); then RC=0; else RC=$?; printf 'TLS RC=%s\n' "$RC"; exit 1; fi
printf 'TLS verified (no -k); RC=%s headers/code=%s\n' "$RC" "$CODE"
case "${CODE: -3}" in 2??|3??) ;; *) exit 1;; esac
```

8. B guest에서 같은 HTTPS 요청을 새 프로세스로 실행한다. DNS 성공을 먼저 `getent ahostsv4 www.oracle.com`으로 확인한다. private rt에 default route가 없고 proxy/IPv6/override가 없는 상태에서 실패해야 한다. 외부443 sender egress도 없는 fixture이므로 결과는 **route+egress combined private isolation**이며 route-only 실패로 귀속하지 않는다. 같은 B/endpoint의 NAT+고정443 egress positive→route만 제거 negative→route 복구 positive를 별도 승인/실행하지 않았다면 route-only acceptance는 NOT_SELECTED다. 기본 실습을 위해 새 NAT/VM을 추가하지 않는다. OCI resolver DNS가 거부되면 DNS 설치/정책 문제로 남기고 network-route 실패로 단정하지 않는다. 내부 A→B HTTP는 계속 성공해야 한다.
9. public table을 빈 table로 바꿔 SSH를 끊는 테스트는 기본 완주에서 제외한다. 별도 승인과 Console 복구 담당자가 있을 때만 변경 전 association 저장 → 신규 연결 실패 → 즉시 exact OCID 원복 → 신규 SSH/HTTPS 성공을 검증한다.

## 결과검증


### baseline-24-ready


완료 단계 **RT-ready**: 두 Subnet의 route table OCID/명시적 rule을 저장하고 `vm-a → private-vm:8080`의 본문 `oci-packet-lab`, SYN/SYN-ACK/ACK와 HTTP packet을 실제 캡처한다. 외부 HTTPS는 public 성공, private 실패로 기록한다. private에서 실패해도 VCN 내부 HTTP가 성공해야 경로 분리 증거가 된다. `ip route`만으로 OCI Route Table 선택을 입증할 수 없다. 실패는 timeout/exit code를 기록하되 기대값을 실행 결과로 미리 채우지 않는다.

**실행 기록(아직 미작성):** 실행자/Region/시각, exact resource OCID, 설정 전·후, 성공 출력, deny 오류·exit code/request-id, positive control, 정리 결과. 문서 검증은 실제 OCI 실행 성공을 뜻하지 않는다.

### 이 시나리오의 functional acceptance

| Positive | Negative / 미완료 판정 | Recovery / 반환 |
|---|---|---|
| B index/health, internal HTTP/publicHTTPS | health-file missing404; private emptyRT internet transport failure | bounded trap health200; 기존 routing unchanged |

모두 실행 시 확인할 기대 패턴이다. 생성 전 guard negative는 OCI 장애를 주입한 것과 다르며 다른 자원을 추가하지 않는다. 오류 메시지와 실제 exit code가 없는 항목은 NOT_EXECUTED 또는 BLOCKED로 남긴다.

## 문제해결

|증상|확인·복구|
|---|---|
|HTTP timeout|서버 `ss -lntp`, OS firewall, client egress/server ingress, 실제 IP 순서로 점검|
|Private 외부 HTTPS 성공|NAT/SGW, proxy 환경변수, IP/VNIC per-resource route, IPv6 경로를 확인; `--noproxy '*' -4` 유지|
|DNS 실패|`getent ahostsv4 www.oracle.com` 확인; DNS 오류와 route drop을 구별|
|SSH 단절|Console 관리자가 Subnet route table을 원래 OCID로 복원; 새 key 생성으로 우회하지 않음|


## 리소스정리

### cleanup 시작 전 마지막 readback

1. 이 topic이 소유한 rule/file/resource만 목록화하고 실제 creator와 예약 consumer에게 반환을 받는다. 비용 expiry가 끝나면 새로운 consumer를 시작하지 않고 미완료 작업을 중단/원복한다. 다른 owner 리소스는 아래 본문 순서로 해당 owner가 정리한다.
2. **보존/반환 gate:** app-only 소비자25/26/28/30의 반환 또는 승인 중단과 expiry를 확인한다. 반환 전에는8080 process·guest rule·packet-lab을 그대로 보존하고 아래 cleanup으로 진행하지 않는다. 소비자26/28의 별도 listener도 해당 owner가 먼저 종료/반환한다. expiry 갱신이 없으면 새 소비자를 시작하지 않고 활성 소비자를 owner와 중단/반환 조정한다.
3. 반환 후 관리 새 SSH와 현재 app positive를 확인한다. owner24가 최초 server terminal에서 Ctrl-C를 **한 번만** 수행하고 B의 `ss -lntp '( sport = :8080 )'`에서 listener 부재와 primary 새 SSH 성공을 확인한다. terminal/PID 소유권이 없으면 BLOCKED_OWNER로 남기고 `pkill python`/전체 service stop으로 우회하지 않는다.
4. 아래 **유일한 guest cleanup block**을 한 번 수행한 뒤에만 owner24의 OCI HTTP rule ID 제거/readback → 소유 파일 정리 순서로 진행한다. 여기서 guest rule을 먼저 제거하거나 뒤에서 다시 제거하지 않는다.


**B cleanup terminal ledger readback:** owner24의 private file을 직접 읽어 source/zone/8080·RUNTIME_BEFORE/PERMANENT_BEFORE/PENDING과 creator 기록을 대조한 뒤 아래를 실행한다. 타인이 쓴 state file을 source하지 않는다. PENDING=1은 checked-absent 뒤 add가 중단된 상태이므로 같은 exact rule만 owner24가 회수한다. rule이 이미 사라졌으면 새 rule을 만들지 않는다. 실패하면 ledger를 보존하고 이 block에서 복구하며 파일/OCI cleanup으로 넘어가지 않는다.

```bash
# B guest; owner24 작성 파일만
LEDGER24="$HOME/owner24-firewall.state"
STATE="$HOME/owner24-firewall.state"
test -f "$STATE" && test ! -L "$STATE" && test -O "$STATE" || exit 1
test "$(stat -c %a "$STATE")" = 600 || exit 1
# Inspect exact own journal before source; do not source an untrusted file.
source "$STATE" || exit 1
[ "$LEASE" = exclusive ] || exit 1
case "$RUNTIME_BEFORE:$PERMANENT_BEFORE:$PENDING" in 0:0:1|0:1:1|1:0:0|1:1:0) ;; *) exit 1;; esac
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

```


24가 추가한 HTTP NSG rule ID와 guest runtime rich-rule/packet-lab의 index.html·health.txt 파일만 소유한다. 25 소비자가 있으면 승인 expiry까지8080 workload/host rule을 유지하되25에서 baseline OCI8080 허용을 제거하고 새 NSG rule로 대체해야 한다. **위1~4와 fenced block이 유일한 stop/guest 원복 절차**이며 여기서 Ctrl-C/remove/source를 반복하지 않는다. 다음으로 owner24의 HTTP rule ID 중 아직 존재하는 자기 추가 ID만 Console에서 제거한다. NSG/SL 전체 readback으로 SSH22/원래 배열을 대조하며25가 이미 제거한 ID를 다시 제거하거나 consumer의 rule ID를 대신 삭제하지 않는다. guest 소유 파일/health rename의 정상 복구와 모든 listener 반환을 확인한 뒤 exact `$HOME/packet-lab/index.html`, `$HOME/packet-lab/health.txt`만 삭제하고 빈 전용 directory를 rmdir한다. disabled health가 남으면 먼저 A-2의 owner 복구로 반환하며 glob/recursive HOME 삭제는 하지 않는다. 비민감 증거 JSON/hash는 승인 보존기한까지 유지한다. VM은 owner10/22, private subnet/table/SL은 owner21, public subnet/table/IGW/SL/NSG는 owner23, VCN은 owner19, Compartment는 owner00가 소비자 종료 후 정리한다. Runtime rule은 permanent로 저장하지 않는다. 공유 VM/네트워크를24에서 일괄 삭제하지 않는다.


## 공식문서

### HTTP workload의 공식 Oracle 근거와 적용 범위

[Oracle Linux 공식 HTTP 두 host 예제](https://docs.oracle.com/en/operating-systems/oracle-linux/mptcp/mptcp-testing_mptcp_subflows.html)는 B에서 `python3 -m http.server`를 실행하고 A에서 curl로 확인한다. [Oracle Linux9 공식 예제의 Python HTTP root](https://docs.oracle.com/en/learn/ol-podman-intro/)는 Oracle Linux9 image의 Python 표준 HTTP server와 working directory를 document root로 사용한다. 이 lab은 **MPTCP/Podman/container를 도입하지 않고**, 같은 HTTP 진단 원리만 기존 Oracle Linux9 guest에 적용한다. 본문의8080–8084, 명시적 index.html/health.txt, runtime /32 firewall 및 복구 trap은 이 lab의 제한접근·소유권 요구에 맞춘 저술값이며 공식 문서의 public80/root directory listing 값을 복사하지 않는다. 시작 전 `python3 -m http.server --help`에서 실제 guest의 `--bind` 지원을 확인하고 미지원이면 중단한다. Oracle Linux9/OCI에서 실행 결과는 아직 없다.


아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 아래 본문은 각 생성 필드·설정·검증·정리의 기술 근거이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [30] [Networking Overview — Route Tables](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/overview.htm)
  - 확인 범위: ROUTE TABLES.

### 실습 작업을 지원하는 추가 공식 본문

- [실습근거 1] [VCN Route Tables](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/managingroutetables.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
- [실습근거 2] [Internet Gateway](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/managingIGs.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
- [실습근거 3] [Security Rules](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/securityrules.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
- [실습근거 4] [Details for the Core Services](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
