> **기존 실습 초안 — 미검증**: 원래 93개 문서의 절차를 보존한 자료입니다. 새 튜토리얼의 품질 승인이나 OCI 실행 검증을 뜻하지 않습니다.

# Security Lists

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../../../../Docs/README.md)를 확인하세요.

> **저술 완료 · OCI 실행 미검증** — Oracle 공식 작업 문서를 근거로 작성한 한국어 hands-on입니다. Oracle 공인 교육/실행 인증이 아니며 기대 결과와 실제 결과를 구분합니다. 비용 승인과 실제 선행 산출물 확인 후에만 실행하세요.

## 학습목표

- private-app의 원래 SL 배열에 sl-private-probe를 추가하고 B TCP8081 workload와 /32 guest/client 조건을 고정한다.
- SL ingress8081만 허용→제거→복구하며 3회 새 요청을 비교한 뒤 원래 SL 배열·자기 rules를 복원하고 test SL을 삭제한다.

## 대응 Roadmap 항목

- 항목명: **Security Lists**
- Roadmap 경로: **Virtual Cloud Network (VCN) → Security Lists**
- Node ID: `sequence-3-7`
- 상위 항목: **Virtual Cloud Network (VCN)** (`sequence-3-0`)
- 문서 범위: private-app의 원래 SL 배열에 sl-private-probe를 추가하고 B TCP8081 workload와 /32 guest/client 조건을 고정한다. SL ingress8081만 허용→제거→복구하며 3회 새 요청을 비교한 뒤 원래 SL 배열·자기 rules를 복원하고 test SL을 삭제한다. 실제 OCI 실행 결과가 아니라 이 절차의 완료 조건이다.
- [전체 항목 목록](../../../../Docs/README.md)

## 사전준비

### 기준 계약과 실제 단계 선행조건

[공통 baseline-contract.json](../../../../baseline-contract.json)의 OS/CIDR/소유권을 따른다. Oracle Linux 9 / `opc` / `dnf` 외의 숨은 Ubuntu 또는 probe VM은 만들지 않는다. OCID/IP의 null은 실행 전 반드시 실제 출력으로 채우는 **실행 차단 입력**이다. approval/capacity/quota/identity 미확인은 완료가 아니다.

- [24 Route Tables](05_Route_Tables.md#baseline-24-ready): **A/B 관리 SSH, owner24 packet-lab/index.html=oci-packet-lab·health.txt=packet-health-ok 및8080 허용/guest ledger**를 실제 출력으로 인수한다. 이 문서의 변경/반환까지 creator 승인 expiry 안에서 보존한다.
- [25 Network Security Groups](06_Network_Security_Groups.md#baseline-25-ready): **source-NSG 시험과 owner24 CIDR baseline 원복 readback; nsg-client/nsg-web는 재사용하지 않음**를 실제 출력으로 인수한다. 이 문서의 변경/반환까지 creator 승인 expiry 안에서 보존한다.

공유 A/B consumer는 INSTANCE_A_ID에 owner10 vm-a의 실제 OCID, INSTANCE_B_ID에 owner22 private-vm의 실제 OCID, `A_PUBLIC_IP`, `A_PRIVATE_IP`, `B_PRIVATE_IP`를 실제 생성 응답에서 전달받는다. 해당 주제에 필요하지 않은 VM/변수는 생성·요구하지 않는다.

- [Route Tables](../../../../Docs/03_Virtual_Cloud_Network_VCN/05_Route_Tables.md)의 **RT-ready** 완료. [Network Security Groups](../../../../Docs/03_Virtual_Cloud_Network_VCN/06_Network_Security_Groups.md)의 **NSG-tested** 완료 또는 모든 NSG에 8081 허용이 없음을 확인한다.
- 실험할 Subnet `private-app`가 업무 자원과 공유되지 않아야 한다. SL 변경 전 Subnet의 Security List OCID 배열과 모든 rule을 저장한다. 두 VM은 유지한다.

## 권한

sl-private-probe Create/Update/Delete와 private-app UpdateSubnet의 SECURITY_LIST_ATTACH/DETACH, A sender egress8081 NSG add/remove/read를 승인한다. SL 배열 전체 보존은 API 권한과 별개로 필수다. VM/subnet 생성이나 B NSG membership 변경은 수행하지 않는다. guest8081 process/runtime rule은 host owner가 승인한다.

### 실제 operation과 공식 권한

- `AddNetworkSecurityGroupSecurityRules`: NETWORK_SECURITY_GROUP _UPDATE_SECURITY_RULES and NETWORK_SECURITY_GROUP_INSPECT if writing a rule that specifies a network security group as the source (for ingress rules) or destination (for egress rules) — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `CreateSecurityList`: SECURITY_LIST_CREATE and VCN_ATTACH — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `DeleteSecurityList`: SECURITY_LIST_DELETE and VCN_DETACH — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `RemoveNetworkSecurityGroupSecurityRules`: NETWORK_SECURITY_GROUP _UPDATE_SECURITY_RULES — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `UpdateSecurityList`: SECURITY_LIST_UPDATE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `UpdateSubnet`: SUBNET_UPDATE If changing which route table is associated with the subnet, also need ROUTE_TABLE_ATTACH and ROUTE_TABLE_DETACH If changing which security lists are associated with the subnet, also need SECURITY_LIST_ATTACH and SECURITY_LIST_DETACH If changing which set of DHCP options are associated with the subnet, also need DHCP_ATTACH and DHCP_DETACH — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `ListNetworkSecurityGroupSecurityRules`: NETWORK_SECURITY_GROUP_LIST_SECURITY_RULES — [공식 API 권한 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).

위 permission 이름은 IAM policy resource-type이 아니다. 조직 관리자가 기존 group/compartment scope와 conditional dependency를 실제 API 표에 대조한다. 편의 family grant를 최소권한 정책으로 부르지 않으며 실습자는 policy 작성 권한을 자동 인수하지 않는다.

## 비용및주의사항

- 신규 클라우드 자원은 sl-private-probe 1개다. A/B와 owner24 packet-lab/index.html을 재사용하고 B에8081 전용 process/runtime rule만 추가한다. VM/boot의 보존 창을 늘려야 하면 owner에게 재승인받는다.
- SL은 private-app 전체 VNIC에 적용되므로 운영/다른 consumer와 공유된 subnet이면 시험 금지다. source A/32 ingress8081만 제거/복구하고 SSH22·기존 SL 배열을 보존한다.8081 process→자기 guest/OCI rule→원래 SL 배열→test SL 순으로 정리한다.

실제 OCI 작업은 아직 수행하지 않았다. 기대값은 실행 시 대조할 조건이며 무료 사용이나 과금 강제 차단을 보장하지 않는다. 승인 expiry가 끝나면 새 시험을 중단하고 위 소유권 순서로 원복·반환한다.

## 실습구성

### 이번 workshop 시나리오와 입력 계약

private-app에 전용 Security List를 연결하여 ingress만 제거·복구한다. 

|입력|값/출처|시점|실행 위치|선택 이유|
|---|---|---|---|---|
|`VCN_ID`|owner19 shared-vcn 생성 상세의 OCID; 모든 network target의 VCN 일치 확인|해당 선행 완료 또는 실제 guest 조회 후|본문에 지정한 Console/Cloud Shell 또는 guest| 모든 network target의 VCN 일치 확인|
|`SUBNET_PRIVATE_ID`|owner21 private-app 생성 OCID; route/SL association 조회 대상|해당 선행 완료 또는 실제 guest 조회 후|본문에 지정한 Console/Cloud Shell 또는 guest| route/SL association 조회 대상|
|`A_PRIVATE_IP`|owner10 primary GetVnic private-ip; B ingress/guest rule의 source /32|해당 선행 완료 또는 실제 guest 조회 후|본문에 지정한 Console/Cloud Shell 또는 guest| B ingress/guest rule의 source /32|
|`B_PRIVATE_IP`|owner22 primary GetVnic private-ip; A egress/HTTP URL의 대상|해당 선행 완료 또는 실제 guest 조회 후|본문에 지정한 Console/Cloud Shell 또는 guest| A egress/HTTP URL의 대상|

|Console field|실제 입력/선택|이유|
|---|---|---|
|SL name / ingress / subnet SL array|sl-private-probe; A/32 TCP8081; original SL array preserved|이번 기능/실패 원인을 한 변수로 분리|

**생성·변경 범위:** sl-private-probe:SecurityList. **완료 비교:** allow→ingress removal deny→restore success. 기대 패턴이며 OCI 실측값이 아니다. **정리:** 본 주제의 변경만 원복하고 전용 신규 resource만 dependency 역순으로 정리한다; 공유 자원은 creator와 consumer registry/expiry 기준으로 반환한다.

전용 SL `sl-private-probe`를 생성해 private Subnet에 연결한다. Default SL의 모든 egress 허용을 함께 남긴 채 egress deny를 주장하지 않는다. 이번 실험 포트는 TCP **8081**, source `<A_PRIVATE_IP>/32`이다. Subnet 전체 적용을 확인하기 위해 동일 private Subnet의 다른 VNIC가 있다면 구성 조회만 하고 새 VM은 만들지 않는다.

## 실습절차

### 명령 실행 gate — 실패하면 다음 생성으로 진행하지 않는다

1. 아래 Bash는 **전용 Bash 세션**에서만 사용한다. Console/Cloud Shell 조회, 본인 로컬 SSH, 로그인한 Oracle Linux guest를 섞지 않는다. `exit 1`은 이 전용 세션을 닫는 안전 중단이다. `<...>`·빈 입력·타 owner OCID가 남으면 실행하지 않는다.
2. 성공을 요구하는 파일 작성/조회/설치/attach 작업은 실패 시 중단한다. 실패한 API 뒤 기존 JSON이나 기존 다운로드 파일을 성공 근거로 읽지 않는다. 기대 deny는 `if command; then ...; else RC=$?; ...; fi`로 별도 기록하고, 실제 timeout/IAM 오류/로컬 파일 오류를 구별한다.
3. 첫 변경 전에 정확한 OCID·owner·현재 설정과 승인 expiry를 저장한다. 공유 자원 변경 중 다른 소비자가 시작하거나 원복값을 확인할 수 없으면 변경을 중단한다. 삭제 성공은 API 수락이 아니라 같은 identity/Region의 정확한 대상 재조회로 판정한다. `NotAuthorizedOrNotFound` 단독은 삭제 증거가 아니다.


### 개별 완주 보정 — 생성·검증·소유권

SL source port는 All, destination TCP8081, Stateless off로 입력한다. Subnet Edit에서 원래 SL OCID 배열에 test SL만 추가한다. 신규 서버는 B의 ~/packet-lab에서 실행하며 read body는 oci-packet-lab이다.

### Console 규칙 입력과 원복 ledger — TCP8081

1. **Networking → Virtual cloud networks → shared-vcn → Security → Network security groups**(또는 동일 이름 탭)에서 A egress용 기존 nsg-admin OCID를 연다. B ingress는 **VCN → Security Lists → sl-private-probe → Add ingress rule**에서 같은 필드로 입력한다. Add Rules의 Direction은 표의 방향, Stateless는 해당 단계값, IP Protocol은 TCP(6), Source/Destination Type은 본문 CIDR 또는 같은 VCN NSG를 선택한다. Forward source port=All, destination port=8081이다. /32는 실제 peer private IP이며 internet source는 없다.
2. **Compute → Instances → 대상 → Networking → Attached VNICs → primary → Edit**에서 기존 NSG OCID 배열을 보존하고 본문에서 생성한 NSG가 없다면 membership을 변경하지 않는다. 이 주제는 Subnet Edit에서 원래 SL 배열에 test SL만 추가한다. nsg-admin/nsg-private-admin의 SSH22 membership은 유지한다. A egress는 NSG Security Rules의 새 rule ID를 기록한다. B ingress는 sl-private-probe의 ingress-security-rules 배열에서 source A/32·TCP8081·stateful인 exact tuple과 원래 배열을 기록한다. SL rule에 NSG-style ID를 가정하지 않는다. Subnet 상세의 SL OCID 배열도 readback하고 각각 해당 ID/tuple/배열만 원복한다.
3. 첫 deny 전에 복구 Console 담당자에게 exact 원복 rule/멤버십과 승인된 **60초 최대 시험 창**을 전달한다. 한 번의 8초 유한 요청 뒤 곧바로 원복한다. 진단을 위해 deny 상태를 연장하지 않는다. 원복 후 새 연결 positive가 실패하면 후속 lab/cleanup을 시작하지 말고 listener→guest firewall→OCI union→route 순으로 복구한다.


### Oracle Linux guest positive-control 준비

이 주제의 B guest에서만 PEER_IP=A_PRIVATE_IP로 지정한다. A의 신규 listener/firewall은 이 topic에서 만들지 않는다. FIREWALL_ZONE은 `firewall-cmd --get-active-zones`에서 primary interface가 연결된 실제 zone으로 지정한다. 비활성 firewalld를 임의 활성화하거나 SELinux를 disable하지 않는다. shell별 RULE_ADDED/HTTP_RULE 값을 ledger에 보존한다.

```bash
export PEER_IP='<상대_VM_actual_private_IP>'
export FIREWALL_ZONE='<이_guest_primary_interface_active_zone>'
python3 - <<'PY'
import ipaddress,os
x=ipaddress.ip_address(os.environ['PEER_IP'])
assert x.version==4 and x.is_private, 'STOP: approved actual private IPv4 required'
PY
if [ "$?" -ne 0 ]; then exit 1; fi
export HTTP_RULE="rule family=\"ipv4\" source address=\"$PEER_IP/32\" port port=\"8081\" protocol=\"tcp\" accept"
STATE="$HOME/owner26-firewall.state"
# Before queries/add: owner26 records exclusive zone/rule lease through cleanup.
# No outside writer/reload/reboot; if this cannot be approved, STOP before mutation.
read -r -p 'Exclusive owner lease recorded? Type EXCLUSIVE-OWNER26: ' LEASE
[ "$LEASE" = EXCLUSIVE-OWNER26 ] || exit 1
if test -e "$STATE" || test -L "$STATE"; then printf 'STOP: ledger collision; use recovery below\n'; exit 1; fi
systemctl is-active --quiet firewalld || exit 1
case "$FIREWALL_ZONE" in ''|*'<'*|*'>'*) exit 1;; esac
sudo firewall-cmd --zone="$FIREWALL_ZONE" --list-all >/dev/null || exit 1
if sudo firewall-cmd --zone="$FIREWALL_ZONE" --query-rich-rule="$HTTP_RULE"; then RUNTIME_BEFORE=0; else RUNTIME_BEFORE=$?; fi
if sudo firewall-cmd --zone="$FIREWALL_ZONE" --permanent --query-rich-rule="$HTTP_RULE"; then PERMANENT_BEFORE=0; else PERMANENT_BEFORE=$?; fi
case "$RUNTIME_BEFORE:$PERMANENT_BEFORE" in 0:0|0:1|1:0|1:1) :;; *) printf 'STOP: query error\n'; exit 1;; esac
# Durable exact recovery inputs precede add; rc1=absent, rc0=present.
PENDING=0; [ "$RUNTIME_BEFORE" = 0 ] || PENDING=1
(umask 077; set -C; printf 'FIREWALL_ZONE=%q\nHTTP_RULE=%q\nRUNTIME_BEFORE=%q\nPERMANENT_BEFORE=%q\nPENDING=%q\n' "$FIREWALL_ZONE" "$HTTP_RULE" "$RUNTIME_BEFORE" "$PERMANENT_BEFORE" "$PENDING" > "$STATE") || exit 1
sync -f "$STATE" || exit 1
sync -f "$HOME" || exit 1
if [ "$PENDING" = 1 ]; then
  sudo firewall-cmd --zone="$FIREWALL_ZONE" --add-rich-rule="$HTTP_RULE" || exit 1
fi
sudo firewall-cmd --zone="$FIREWALL_ZONE" --query-rich-rule="$HTTP_RULE" || exit 1
# Keep pending after add. Interrupted/failed add uses the SAME cleanup below.
```

OCI deny 전 같은 listener/guest firewall에서 positive control이 성공해야 한다. listener는 owner24의 `$HOME/packet-lab/index.html`을 그대로 재사용하며 개인 HOME root에서 HTTP server를 열지 않는다. `8081`는 인터넷 source로 허용하지 않는다.

**터미널 실행 구분:** `tcpdump`는 별도 SSH 터미널에서 먼저 시작하고, `curl`/CLI workload는 다른 터미널에서 실행한다. 같은 shell에서 capture 종료를 기다렸다가 요청하면 packet이 수집되지 않는다. 본문의 유한 요청이 실패해도 담당자는 승인 deadline에 해당 HTTP/gateway 변경만 즉시 원복한다.

### A. Security List 생성
1. `private-vm`에서 `cd ~/packet-lab && python3 -m http.server 8081 --bind 0.0.0.0`를 별도 터미널에서 실행한다. `sudo ss -lntp`로 8081 listener를 확인한다.
2. A의 nsg-admin에 stateful Egress destination B/32 TCP8081을 추가하고 ID를 기록한다. B host firewall에는24와 같은 source A/32 runtime TCP8081 rule을 허용해 positive control을 먼저 만든다. NSG/SL deny 동안 guest firewall은 고정한다. **VCN → Security Lists → Create Security List**: `sl-private-probe`. stateful ingress TCP22 source `<A_PRIVATE_IP>/32`, stateful ingress TCP8081 source `<A_PRIVATE_IP>/32`, 새 test SL egress는 **빈 배열**로 생성한다. 기존 sl-private의 배열은 건드리지 않는다. stateful ingress의 응답은 추적으로 허용되며 별도 broad egress를 추가하지 않는다. 이 문서는 ingress를 테스트하므로 broad egress를 deny 증거로 사용하지 않는다.
3. **Subnets → private-app → Edit**에서 test SL을 선택한다. 다른 SL의 TCP8081 또는 all-protocol ingress가 허용하지 않는지 확인한다. 연결된 SL 전체 배열을 기록한다. 기존 default SL을 무조건 삭제하지 않는다.

### B. Subnet 단위 규칙 변경
4. 서버에서 `sudo timeout 20s tcpdump -ni any 'tcp port 8081'`, client에서 `curl --noproxy '*' --max-time 8 http://<B_PRIVATE_IP>:8081/index.html`를 실행해 성공을 기록한다.
5. test SL에서 **TCP8081 ingress만** 제거한다. SSH rule은 남긴다. 새 요청의 timeout과 server에 SYN이 오지 않는 상태를 기록한다. TCP8081을 다시 추가하고 성공을 확인한다.
6. client와 server VNIC의 Subnet OCID·NSG 배열을 함께 비교한다. SL은 Subnet 연결로 VNIC에 적용되며 NSG 가입을 요구하지 않는다. API로 SL을 업데이트할 경우 **전체 rule 배열을 교체**하는 점을 이해하고 현재 배열을 누락하지 않는다. 여기서는 안전한 Console 편집을 사용한다.

## 결과검증


### baseline-26-ready


완료 단계 **SL-tested**: Subnet→SL 연결, 허용/제거/복구 3회 요청 결과와 packet을 남긴다. VNIC NSG 소속을 바꾸지 않고 Subnet SL rule만 바꾼 사실이 핵심이다. 실패 시 TCP listener·호스트 firewall·경로가 변하지 않았다는 positive control도 남긴다.

**실행 기록(아직 미작성):** 실행자/Region/시각, exact resource OCID, 설정 전·후, 성공 출력, deny 오류·exit code/request-id, positive control, 정리 결과. 문서 검증은 실제 OCI 실행 성공을 뜻하지 않는다.

### 이 시나리오의 functional acceptance

| Positive | Negative / 미완료 판정 | Recovery / 반환 |
|---|---|---|
| test SL 연결 TCP8081 body | test SL ingress8081만 제거 후 deny | same ingress 복구 후 body; original SL 배열로 반환 |

모두 실행 시 확인할 기대 패턴이다. 생성 전 guard negative는 OCI 장애를 주입한 것과 다르며 다른 자원을 추가하지 않는다. 오류 메시지와 실제 exit code가 없는 항목은 NOT_EXECUTED 또는 BLOCKED로 남긴다.

## 문제해결

실습 SL을 편집했는데 영향이 없으면 Subnet에 실제 연결된 OCID인지 확인한다. 다른 SL/NSG all-ingress가 겹치면 제거 후 새 연결로 재시도한다. SSH 실패는 TCP22 source가 public IP 대신 `vm-a`의 private IP여야 하는지 점검한다. VCN 내 통신에서도 OCI security rule과 OS firewall 모두 필요하다.


## 리소스정리

### cleanup 시작 전 마지막 readback

1. 이 topic이 소유한 rule/file/resource만 목록화하고 실제 creator와 예약 consumer에게 반환을 받는다. 비용 expiry가 끝나면 새로운 consumer를 시작하지 않고 미완료 작업을 중단/원복한다. 다른 owner 리소스는 아래 본문 순서로 해당 owner가 정리한다.
2. 관리 SSH와 현재 app positive를 먼저 확인한 뒤 server를 자신이 연 terminal의 Ctrl-C로 종료한다. `ss -lntp`에서 본 topic 포트 listener가 사라지고 primary 새 SSH는 살아 있는지 확인한다. `pkill python`/전체 service stop은 쓰지 않는다.
3. runtime firewall 원복은 아래 durable-ledger block만 수행한다(이 단계에서 별도 remove 금지). OCI 자기 rule ID 제거 뒤 SSH22와 원래 배열을 readback한다. 공유 NSG/SL은 삭제하지 않는다.
4. topic26은 app 파일/디렉터리를 만들지 않는다. owner24의 packet-lab/index.html·health.txt·디렉터리는 삭제/rmdir하지 않는다. 증거 JSON/hash는 승인된 비민감 보존기한까지 별도 유지한다. glob/recursive HOME 삭제는 하지 않는다.


TCP8081 서버 중지 → Subnet Security List 배열을 원래대로 복원 → test SL 분리 후 삭제. 배열 복원 전에 삭제하면 연결 자원 때문에 실패하거나 관리 경로가 중단될 수 있다. 24 자원은 다음 네트워크 실습까지 보존한다.

**guest 원복은 이 block 한 번만:** B cleanup terminal에서 본인이 작성한 owner26 private ledger를 읽는다. PENDING=1은 add 직전 query=no였던 중단 복구 상태다. 동시 firewall writer가 있으면 중단하고 owner와 조정한다. pre-existing rule(RULE_ADDED=0/PENDING=0)은 제거하지 않는다. firewalld가 예기치 않게 inactive이면 소유 ledger를 지우지 말고 관리자 복구를 기다린다.

```bash
# Same backend guest, fresh terminal recovery after interrupted/failed add.
# Confirm original exclusive owner26 lease; outside mutation means STOP/reconcile.
STATE="$HOME/owner26-firewall.state"
test -f "$STATE" && test ! -L "$STATE" && test -O "$STATE" || exit 1
test "$(stat -c %a "$STATE")" = 600 || exit 1
source "$STATE" || exit 1
case "$RUNTIME_BEFORE:$PERMANENT_BEFORE:$PENDING" in 0:0:0|0:1:0|1:0:1|1:1:1) :;; *) exit 1;; esac
test -n "$FIREWALL_ZONE" && test -n "$HTTP_RULE" || exit 1
systemctl is-active --quiet firewalld || exit 1
sudo firewall-cmd --zone="$FIREWALL_ZONE" --list-all >/dev/null || exit 1
if sudo firewall-cmd --zone="$FIREWALL_ZONE" --query-rich-rule="$HTTP_RULE"; then runtime_now=0; else runtime_now=$?; fi
if sudo firewall-cmd --zone="$FIREWALL_ZONE" --permanent --query-rich-rule="$HTTP_RULE"; then permanent_now=0; else permanent_now=$?; fi
case "$runtime_now:$permanent_now" in 0:0|0:1|1:0|1:1) :;; *) exit 1;; esac
[ "$permanent_now" = "$PERMANENT_BEFORE" ] || { printf 'STOP: permanent changed; reconcile owner\n'; exit 1; }
if [ "$PENDING" = 1 ] && [ "$runtime_now" = 0 ]; then
  sudo firewall-cmd --zone="$FIREWALL_ZONE" --remove-rich-rule="$HTTP_RULE" || exit 1
fi
if sudo firewall-cmd --zone="$FIREWALL_ZONE" --query-rich-rule="$HTTP_RULE"; then rc=0; else rc=$?; fi
[ "$rc" = "$RUNTIME_BEFORE" ] || exit 1
if sudo firewall-cmd --zone="$FIREWALL_ZONE" --permanent --query-rich-rule="$HTTP_RULE"; then rc=0; else rc=$?; fi
[ "$rc" = "$PERMANENT_BEFORE" ] || exit 1
# Save both actual readbacks in registry, return exclusive lease, delete own ledger.
rm -- "$STATE" || exit 1
```

위 readback 성공 뒤에만 ledger를 제거한다. permanent/전체 ruleset을 수정하지 않으며 OCI rule은 자기 ID만 제거한다. VM/subnet/IGW/VCN은 원래 creator에게 반환한다.


## 공식문서

### HTTP workload의 공식 Oracle 근거와 적용 범위

[Oracle Linux 공식 HTTP 두 host 예제](https://docs.oracle.com/en/operating-systems/oracle-linux/mptcp/mptcp-testing_mptcp_subflows.html)는 B에서 `python3 -m http.server`를 실행하고 A에서 curl로 확인한다. [Oracle Linux9 공식 예제의 Python HTTP root](https://docs.oracle.com/en/learn/ol-podman-intro/)는 Oracle Linux9 image의 Python 표준 HTTP server와 working directory를 document root로 사용한다. 이 lab은 **MPTCP/Podman/container를 도입하지 않고**, 같은 HTTP 진단 원리만 기존 Oracle Linux9 guest에 적용한다. 본문의8080–8084, 명시적 index.html/health.txt, runtime /32 firewall 및 복구 trap은 이 lab의 제한접근·소유권 요구에 맞춘 저술값이며 공식 문서의 public80/root directory listing 값을 복사하지 않는다. 시작 전 `python3 -m http.server --help`에서 실제 guest의 `--bind` 지원을 확인하고 미지원이면 중단한다. Oracle Linux9/OCI에서 실행 결과는 아직 없다.


아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 아래 본문은 각 생성 필드·설정·검증·정리의 기술 근거이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [31] [Security Rules — Security Lists](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/securityrules.htm)
  - 확인 범위: Security lists let you define.

### 실습 작업을 지원하는 추가 공식 본문

- [실습근거 1] [Security Rules](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/securityrules.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
- [실습근거 2] [Network Security Groups](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/networksecuritygroups.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
