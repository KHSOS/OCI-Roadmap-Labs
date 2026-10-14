> **기존 실습 초안 — 미검증**: 원래 93개 문서의 절차를 보존한 자료입니다. 새 튜토리얼의 품질 승인이나 OCI 실행 검증을 뜻하지 않습니다.

# VNICs

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../../../../Docs/README.md)를 확인하세요.

> **저술 완료 · OCI 실행 미검증** — Oracle 공식 작업 문서를 근거로 작성한 한국어 hands-on입니다. Oracle 공인 교육/실행 인증이 아니며 기대 결과와 실제 결과를 구분합니다. 비용 승인과 실제 선행 산출물 확인 후에만 실행하세요.

## 학습목표

- 기존 B에 private secondary VNIC 한 개와 test NSG를 attach하고 OCI MAC/IP를 guest interface 및 공식 OCI Utilities 구성과 대조한다.
- secondary IP bind TCP8084 성공→test ingress 제거 실패→복구를 확인하며 primary SSH를 보존하고 secondary OS 구성→detach→test NSG 순서로 정리한다.

## 대응 Roadmap 항목

- 항목명: **VNICs**
- Roadmap 경로: **Virtual Cloud Network (VCN) → VNICs**
- Node ID: `sequence-3-13`
- 상위 항목: **Virtual Cloud Network (VCN)** (`sequence-3-0`)
- 문서 범위: 기존 B에 private secondary VNIC 한 개와 test NSG를 attach하고 OCI MAC/IP를 guest interface 및 공식 OCI Utilities 구성과 대조한다. secondary IP bind TCP8084 성공→test ingress 제거 실패→복구를 확인하며 primary SSH를 보존하고 secondary OS 구성→detach→test NSG 순서로 정리한다. 실제 OCI 실행 결과가 아니라 이 절차의 완료 조건이다.
- [전체 항목 목록](../../../../Docs/README.md)

## 사전준비

### 기준 계약과 실제 단계 선행조건

[공통 baseline-contract.json](../../../../baseline-contract.json)의 OS/CIDR/소유권을 따른다. Oracle Linux 9 / `opc` / `dnf` 외의 숨은 Ubuntu 또는 probe VM은 만들지 않는다. OCID/IP의 null은 실행 전 반드시 실제 출력으로 채우는 **실행 차단 입력**이다. approval/capacity/quota/identity 미확인은 완료가 아니다.

- [24 Route Tables](05_Route_Tables.md#baseline-24-ready): **A/B 관리 SSH, owner24 packet-lab/index.html=oci-packet-lab·health.txt=packet-health-ok 및8080 허용/guest ledger**를 실제 출력으로 인수한다. 이 문서의 변경/반환까지 creator 승인 expiry 안에서 보존한다.
- [25 Network Security Groups](06_Network_Security_Groups.md#baseline-25-ready): **source-NSG 시험과 owner24 CIDR baseline 원복 readback; nsg-client/nsg-web는 재사용하지 않음**를 실제 출력으로 인수한다. 이 문서의 변경/반환까지 creator 승인 expiry 안에서 보존한다.

공유 A/B consumer는 INSTANCE_A_ID에 owner10 vm-a의 실제 OCID, INSTANCE_B_ID에 owner22 private-vm의 실제 OCID, `A_PUBLIC_IP`, `A_PRIVATE_IP`, `B_PRIVATE_IP`를 실제 생성 응답에서 전달받는다. 해당 주제에 필요하지 않은 VM/변수는 생성·요구하지 않는다.

- [Route Tables](../../../../Docs/03_Virtual_Cloud_Network_VCN/05_Route_Tables.md) **RT-ready** 완료. A/B VM·Subnets·private IP 목록과 SSH 관리 경로를 보존한다.
- [Network Security Groups](../../../../Docs/03_Virtual_Cloud_Network_VCN/06_Network_Security_Groups.md)의 NSG 생성/가입을 완료했으면 멤버십을 기록한다. B의 Shape가 추가 VNIC를 지원하는지 Console Shape 문서/제한을 확인한다. 지원하지 않으면 승인된 Shape로 변경하거나 미실행으로 남기며 존재하지 않는 VNIC 성공을 적지 않는다.

## 권한

B의 secondary AttachVnic/DetachVnic·GetVnic, nsg-vnic-test Create/rule add/remove/read/Delete 및 A 자기8084 egress를 승인한다. VNIC/subnet attach와 NSG membership 의존은 Compute와 network 양쪽에 걸친다. primary VNIC 변경/VM resize/새 VM launch는 이 범위가 아니다. B OS configure/unconfigure는 opc/sudo와 다른 secondary가 없는 exclusive 조건이 필요하다.

### 실제 operation과 공식 권한

- `CreateNetworkSecurityGroup`: NETWORK_SECURITY_GROUP_CREATE and VCN_ATTACH — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `AddNetworkSecurityGroupSecurityRules`: NETWORK_SECURITY_GROUP _UPDATE_SECURITY_RULES and NETWORK_SECURITY_GROUP_INSPECT if writing a rule that specifies a network security group as the source (for ingress rules) or destination (for egress rules) — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `AttachVnic`: INSTANCE_ATTACH _SECONDARY_VNIC and VNIC_ATTACH and VNIC_CREATE and SUBNET_ATTACH If putting the secondary VNIC in a network security group during VNIC creation, also need NETWORK_SECURITY_GROUP _UPDATE_MEMBERS and VNIC_ASSOCIATE _NETWORK_SECURITY_GROUP — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `DetachVnic`: INSTANCE_DETACH _SECONDARY_VNIC and VNIC_DETACH and VNIC_DELETE and SUBNET_DETACH — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `GetVnic`: VNIC_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `RemoveNetworkSecurityGroupSecurityRules`: NETWORK_SECURITY_GROUP _UPDATE_SECURITY_RULES — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `DeleteNetworkSecurityGroup`: NETWORK_SECURITY_GROUP_DELETE and VCN_DETACH — [공식 API 권한 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `ListNetworkSecurityGroupSecurityRules`: NETWORK_SECURITY_GROUP_LIST_SECURITY_RULES — [공식 API 권한 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).

위 permission 이름은 IAM policy resource-type이 아니다. 조직 관리자가 기존 group/compartment scope와 conditional dependency를 실제 API 표에 대조한다. 편의 family grant를 최소권한 정책으로 부르지 않으며 실습자는 policy 작성 권한을 자동 인수하지 않는다.

oci-network-config show가 SDK/API session 오류를 내면 성공으로 숨기지 않는다. 실제 B/MAC/IP의 OS/IMDS 대응을 확인하거나 exact B Dynamic Group에 필요한 VNIC/VCN/subnet 조회만 별도로 승인한다. API private key를 B에 복사하거나 manage all-resources를 위임하지 않는다.

## 비용및주의사항

- 신규 범위는 B의 secondary VNIC 1개와 nsg-vnic-test 1개, packet-vnic-32/index.html·HTTP8084 process/runtime rule이다. 기존 A/B를 재사용하고 VNIC 부족을 이유로 shape resize/reboot/추가VM을 자동 실행하지 않는다.
- primary SSH/default route를 보존한다. OCI MAC↔실제 secondary interface/IP 대응과 공식 utility ADD 한 개를 확인한 뒤 configure한다. test ingress 제거 시험은60초 창이며 guest rule 정리→secondary OS unconfigure→Delete VNIC→test NSG 순으로 자기 자원만 정리한다.

실제 OCI 작업은 아직 수행하지 않았다. 기대값은 실행 시 대조할 조건이며 무료 사용이나 과금 강제 차단을 보장하지 않는다. 승인 expiry가 끝나면 새 시험을 중단하고 위 소유권 순서로 원복·반환한다.

## 실습구성

### 이번 workshop 시나리오와 입력 계약

B에 secondary VNIC를 실제 attach하고 OS 구성 후 그 IP의 HTTP를 검증한다. 

|입력|값/출처|시점|실행 위치|선택 이유|
|---|---|---|---|---|
|`VCN_ID`|owner19 shared-vcn 생성 상세의 OCID; 모든 network target의 VCN 일치 확인|해당 선행 완료 또는 실제 guest 조회 후|본문에 지정한 Console/Cloud Shell 또는 guest| 모든 network target의 VCN 일치 확인|
|`INSTANCE_B_ID`|owner22의 actual output; `<...>` 대체 필수|실행 전 선행 완료값|Console/사용자 전용 shell|정확한 대상/소유관계 고정|
|`A_PRIVATE_IP`|owner10 primary GetVnic private-ip; B ingress/guest rule의 source /32|해당 선행 완료 또는 실제 guest 조회 후|본문에 지정한 Console/Cloud Shell 또는 guest| B ingress/guest rule의 source /32|
|`SECONDARY_IP`|본 topic32 신규 secondary GetVnic private-ip; B listener bind와 A 요청 대상|생성 후 실제 조회|B guest/Console| B listener bind와 A 요청 대상|
|`SECONDARY_INTERFACE`|본 topic32 OCI MAC과 B ip -br link MAC을 대조한 실제 interface; zone/routing 검사|생성 후 실제 조회|B guest/Console| zone/routing 검사|

|Console field|실제 입력/선택|이유|
|---|---|---|
|VNIC subnet / public IP / NSG / source-destination check|private-app; no public IP; nsg-vnic-test; skip check off|이번 기능/실패 원인을 한 변수로 분리|

**생성·변경 범위:** vnic-b-test:Vnic,nsg-vnic-test:NetworkSecurityGroup. **완료 비교:** MAC/OS/IP match; secondary HTTP allow→deny→restore; primary SSH stable. 기대 패턴이며 OCI 실측값이 아니다. **정리:** 본 주제의 변경만 원복하고 전용 신규 resource만 dependency 역순으로 정리한다; 공유 자원은 creator와 consumer registry/expiry 기준으로 반환한다.

기존 B instance에 같은 VCN의 private Subnet에 secondary VNIC `vnic-b-test` 한 개를 연결한다. **source/destination check 유지**, public IP 없음. OS 자동 구성이 확인되지 않으면 NIC/주소 조회까지만 진행한 뒤 공식 image별 설정 지침을 따른다. primary VNIC를 제거하거나 default route를 바꾸지 않는다.

## 실습절차

### 명령 실행 gate — 실패하면 다음 생성으로 진행하지 않는다

1. 아래 Bash는 **전용 Bash 세션**에서만 사용한다. Console/Cloud Shell 조회, 본인 로컬 SSH, 로그인한 Oracle Linux guest를 섞지 않는다. `exit 1`은 이 전용 세션을 닫는 안전 중단이다. `<...>`·빈 입력·타 owner OCID가 남으면 실행하지 않는다.
2. 성공을 요구하는 파일 작성/조회/설치/attach 작업은 실패 시 중단한다. 실패한 API 뒤 기존 JSON이나 기존 다운로드 파일을 성공 근거로 읽지 않는다. 기대 deny는 `if command; then ...; else RC=$?; ...; fi`로 별도 기록하고, 실제 timeout/IAM 오류/로컬 파일 오류를 구별한다.
3. 첫 변경 전에 정확한 OCID·owner·현재 설정과 승인 expiry를 저장한다. 공유 자원 변경 중 다른 소비자가 시작하거나 원복값을 확인할 수 없으면 변경을 중단한다. 삭제 성공은 API 수락이 아니라 같은 identity/Region의 정확한 대상 재조회로 판정한다. `NotAuthorizedOrNotFound` 단독은 삭제 증거가 아니다.


### 개별 완주 보정 — 생성·검증·소유권

SECONDARY_INTERFACE는 OCI MAC과 ip -br link의 MAC이 일치한 실제 interface 이름을 기록한다. HTTP workload는 새 전용 ~/packet-vnic-32 디렉터리에 index.html을 만든 뒤 SECONDARY_IP에 bind한다. Console detach 전 OS unconfigure 범위에 다른 secondary가 없는지 다시 확인한다.

### Console 규칙 입력과 원복 ledger — TCP8084

1. **Networking → Virtual cloud networks → shared-vcn → Security → Network security groups**(또는 동일 이름 탭)에서 본문 대상 NSG의 OCID를 연다. Add Rules의 Direction은 표의 방향, Stateless는 해당 단계값, IP Protocol은 TCP(6), Source/Destination Type은 본문 CIDR 또는 같은 VCN NSG를 선택한다. Forward source port=All, destination port=8084이다. /32는 실제 peer private IP이며 internet source는 없다.
2. **Compute → Instances → 대상 → Networking → Attached VNICs → vnic-b-test secondary → Edit**에서 기존 NSG OCID 배열을 보존하고 본문에서 생성한 NSG만 추가한다. primary nsg-admin/nsg-private-admin의 SSH22 membership은 그대로 유지한다. Rule 제출 뒤 NSG → Security Rules와 VNIC 상세를 다시 읽고 추가 rule ID·stateful flag·source/destination·port를 ledger에 적는다. 원복은 그 ID/배열만 대상으로 한다.
3. 첫 deny 전에 복구 Console 담당자에게 exact 원복 rule/멤버십과 승인된 **60초 최대 시험 창**을 전달한다. 한 번의 8초 유한 요청 뒤 곧바로 원복한다. 진단을 위해 deny 상태를 연장하지 않는다. 원복 후 새 연결 positive가 실패하면 후속 lab/cleanup을 시작하지 말고 listener→guest firewall→OCI union→route 순으로 복구한다.


### A. VNIC 생성과 OCI 속성


0. **Networking → Virtual Cloud Networks → shared-vcn → Network Security Groups → Create NSG**에서 `nsg-vnic-test`를 lab-root에 생성하고 초기 규칙은 비운다. 이번 topic32 소유 NSG OCID를 기록한다.
1. **Compute → Instances → private-vm → Networking/Resources → Attached VNICs → Create VNIC**. Name `vnic-b-test`, VCN shared-vcn, **Network: Normal Setup: Subnet**, Subnet private-app(Regional), Private IPv4 자동, public IPv4 없음(Private이면 필드 불가), **Use network security groups to control traffic 체크 → nsg-vnic-test**, **Skip source/destination check 해제**, DNS hostname 비워 둠, IPv6 없음, purpose/owner/expiry tags. [Creating and attaching a secondary VNIC](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/managingvnics_tasks-attach.htm)의 **Save Changes**를 제출하고 Attached VNICs에서 새 OCID를 읽는다. physical NIC 필드는 bare metal 전용이므로 이 VM에 선택하지 않는다.
2. VNIC details에서 OCID, primary=false, subnet OCID, private IP, MAC, NSG 배열을 기록한다. **Skip source/destination check**는 끈 상태로 유지한다. Router appliance가 아니므로 forwarding을 활성화할 이유가 없다.

### B. OS NIC와 실제 도달성
3. B에서 `ip -br link`, `ip -br addr`, `ip route show`, `ip rule show`를 저장하고 OCI MAC과 매칭한다. 이미지가 자동 구성하지 않으면 [Managing VNICs](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/managingVNICs.htm)의 **Configuring the OS for Secondary VNICs** 연결 지침에 따라 해당 이미지용 도구를 확인한다. DHCP를 임의 NIC에 실행해 primary default route를 덮어쓰지 않는다. 그 지침/지원 도구를 확인하지 못하면 OS 설정을 미검증으로 표시하고 B의 primary로 복구한다.
### Oracle Linux 9 공식 OCI Utilities 구성

[Managing VNICs — Oracle Linux](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/managingVNICs.htm)은 `oci-network-config`를 권고한다. [OCI Utilities reference — Configuring VNICs](https://docs.oracle.com/en-us/iaas/oracle-linux/oci/oci-utilities-reference.htm)은 `show`, `configure`, `unconfigure`의 실제 subcommand와 policy-based routing을 설명한다. Ubuntu 임시 ip-table/rp_filter recipe는 이 트랙에서 사용하지 않는다.

B에서 OCI MAC ↔ OS interface와 primary route/rule을 먼저 저장한다. Oracle Linux platform image의 oci-utils가 없으면 NAT 설치 준비 승인 아래 `sudo dnf install -y oci-utils` 후 설치 경로를 복원한다. `oci-network-config show`가 API session 오류를 내면 **그 오류를 성공으로 숨기지 않는다**. OS/IMDS 자료가 secondary VNIC/MAC/IP를 충분히 대응하는지 확인하거나 exact B instance DG에 해당 VNIC/VCN/subnet 조회만 한정 위임한다. broad `manage all-resources`를 VM에 주지 않는다. SDK/user API private key를 VM에 복사하지 않는다.

```bash
# B guest: 다른 미구성 secondary VNIC가 없는 전용 VM에서만
rpm -q oci-utils || exit 1
ip -br link
ip -br addr
ip route show
ip rule show
sudo oci-network-config show || exit 1
# show의 ADD가 이번 신규 VNIC 하나임을 Console MAC/OCID와 대조한 후만
sudo oci-network-config configure || exit 1
sudo oci-network-config show || exit 1
ip -br addr
ip rule show
export SECONDARY_IP='<이번_secondary_VNIC_actual_private_ip>'
export A_PRIVATE_IP='<vm-a_actual_private_ip>'
ip route get "$A_PRIVATE_IP" from "$SECONDARY_IP" || exit 1
```

configure는 모든 미구성 VNIC를 동기화하는 명령이므로 다른 ADD/DELETE가 있으면 실행하지 말고 owner와 조정한다. primary SSH를 별도 터미널에서 재검증한다. secondary source의 응답 경로가 올바른 interface인지 확인하기 전 HTTP 성공을 기대하지 않는다.32 때문에 shape resize/재부팅을 자동 실행하지 않는다. 지원 VNIC 수가 부족하면 cost/capacity/image 재승인을 받고 미완료 상태를 유지한다.

### Oracle Linux guest positive-control 준비

이 주제의 B guest에서만 PEER_IP=A_PRIVATE_IP로 지정한다. A의 신규 listener/firewall은 이 topic에서 만들지 않는다. 아래 guest rule 준비는 B의 secondary VNIC 구성 이후 수행한다. FIREWALL_ZONE은 `firewall-cmd --get-active-zones` 및 `firewall-cmd --get-zone-of-interface "$SECONDARY_INTERFACE"`로 이번 secondary interface의 실제 inbound zone을 확인해 지정한다. primary zone과 같다고 추정하지 않는다. 비활성 firewalld를 임의 활성화하거나 SELinux를 disable하지 않는다. shell별 RULE_ADDED/HTTP_RULE 값을 ledger에 보존한다.

```bash
export PEER_IP='<상대_VM_actual_private_IP>'
export FIREWALL_ZONE='<secondary_interface_actual_active_zone>'
python3 - <<'PY'
import ipaddress,os
x=ipaddress.ip_address(os.environ['PEER_IP'])
assert x.version==4 and x.is_private, 'STOP: approved actual private IPv4 required'
PY
if [ "$?" -ne 0 ]; then exit 1; fi
export HTTP_RULE="rule family=\"ipv4\" source address=\"$PEER_IP/32\" port port=\"8084\" protocol=\"tcp\" accept"
STATE="$HOME/owner32-firewall.state"
# Before queries/add: owner32 records exclusive zone/rule lease through cleanup.
# No outside writer/reload/reboot; if this cannot be approved, STOP before mutation.
read -r -p 'Exclusive owner lease recorded? Type EXCLUSIVE-OWNER32: ' LEASE
[ "$LEASE" = EXCLUSIVE-OWNER32 ] || exit 1
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

OCI deny 전 같은 listener/guest firewall에서 positive control이 성공해야 한다. listener는 새 전용 workload 디렉터리의 synthetic index.html로 실행하며 개인 HOME root에서 HTTP server를 열지 않는다. `8084`는 인터넷 source로 허용하지 않는다.

**터미널 실행 구분:** `tcpdump`는 별도 SSH 터미널에서 먼저 시작하고, `curl`/CLI workload는 다른 터미널에서 실행한다. 같은 shell에서 capture 종료를 기다렸다가 요청하면 packet이 수집되지 않는다. 본문의 유한 요청이 실패해도 담당자는 승인 deadline에 해당 HTTP/gateway 변경만 즉시 원복한다.



```bash
# B guest, actual secondary IP 구성 후 step4 직전에 준비
if test -e "$HOME/packet-vnic-32"; then printf '%s\n' 'STOP: workload directory exists'; exit 1; fi
mkdir "$HOME/packet-vnic-32" || exit 1
printf '%s\n' 'oci-vnic-lab' > "$HOME/packet-vnic-32/index.html" || exit 1
# step4: cd "$HOME/packet-vnic-32" && python3 -m http.server 8084 --bind "$SECONDARY_IP"
```

4. 위 durable 준비 block으로 B secondary-zone source A/32 TCP8084 runtime rule을 한 번만 준비하고 ledger를 보존한다. 아직 listener success를 판정하지 않는다. Secondary IP가 OS에 구성되고 egress reply가 해당 VNIC에서 돌아가는 것을 확인한 경우 B의 별도 server terminal에서 `cd "$HOME/packet-vnic-32" && python3 -m http.server 8084 --bind "$SECONDARY_IP"`를 실행한다. 요청 전에 secondary VNIC의 nsg-vnic-test에 stateful Ingress/source A_PRIVATE_IP/32/TCP/source All/destination8084를, A nsg-admin에 stateful Egress/destination SECONDARY_IP/32/TCP/source All/destination8084를 추가하고 각각 새 rule ID를 기록한다. 중복 SL 허용이 없음을 확인한다. **A guest의 별도 요청 터미널**에서 owner32의 실제 SECONDARY_IP를 다시 지정하고 `curl --noproxy '*' --fail --connect-timeout 3 --max-time 8 "http://$SECONDARY_IP:8084/index.html"`를 실행하여 본문 oci-vnic-lab를 확인한다. B에서 자기 secondary 주소로 보내는 요청을 A/32 허용의 positive control로 대신하지 않는다.
5. A에서 `curl --noproxy '*' --max-time 8 http://<SECONDARY_PRIVATE_IP>:8084/index.html`; B에서 `sudo timeout 20s tcpdump -ni <SECONDARY_INTERFACE> 'tcp port 8084'`로 listener와 packet을 검증한다.
6. secondary VNIC의 test NSG ingress만 제거하고 신규 curl 실패 → rule 복원 후 성공을 확인한다. Primary SSH는 계속 정상이어야 한다. API 관점의 VNIC와 OS interface가 항상 자동 1:1 ready인 것은 아니다.

## 결과검증


### baseline-32-ready


완료 단계 **VNIC-tested**: OCI VNIC MAC/OCID↔OS NIC/주소 대응표, primary 유지, secondary HTTP 성공·security rule 제거 실패·복구 성공. OS secondary 구성이 검증되지 않으면 creation-only 상태로 남기고 기능 검증 완료로 표시하지 않는다. 실제 image별 추가 NIC 명령은 공식 지침과 대조해야 하는 실행 게이트다.

**실행 기록(아직 미작성):** 실행자/Region/시각, exact resource OCID, 설정 전·후, 성공 출력, deny 오류·exit code/request-id, positive control, 정리 결과. 문서 검증은 실제 OCI 실행 성공을 뜻하지 않는다.

### 이 시나리오의 functional acceptance

| Positive | Negative / 미완료 판정 | Recovery / 반환 |
|---|---|---|
| secondary OCI/OS MAC/IP match/index body; primarySSH | test secondary ingress8084 제거 후 신규 deny | exact rule restore/body; utility unconfigure→Delete VNIC/readback→primarySSH |

모두 실행 시 확인할 기대 패턴이다. 생성 전 guard negative는 OCI 장애를 주입한 것과 다르며 다른 자원을 추가하지 않는다. 오류 메시지와 실제 exit code가 없는 항목은 NOT_EXECUTED 또는 BLOCKED로 남긴다.

## 문제해결

NIC가 없으면 attachment state/Shape limit/VM reboot 요구 여부 확인. SYN은 오는데 reply가 primary로 나가면 policy routing/OS image 설정 문제다. public IP 없는 private VNIC의 인터넷 접근은 NAT/SGW route에 달려 있다. primary를 떼거나 source/destination check를 끄는 임의 조치로 해결하지 않는다.


## 리소스정리

### cleanup 시작 전 마지막 readback

1. 이 topic이 소유한 rule/file/resource만 목록화하고 실제 creator와 예약 consumer에게 반환을 받는다. 비용 expiry가 끝나면 새로운 consumer를 시작하지 않고 미완료 작업을 중단/원복한다. 다른 owner 리소스는 아래 본문 순서로 해당 owner가 정리한다.
2. 관리 SSH와 현재 app positive를 먼저 확인한 뒤 server를 자신이 연 terminal의 Ctrl-C로 종료한다. `ss -lntp`에서 본 topic 포트 listener가 사라지고 primary 새 SSH는 살아 있는지 확인한다. `pkill python`/전체 service stop은 쓰지 않는다.
3. runtime firewall 원복은 아래 durable-ledger block만 수행한다(이 단계에서 별도 remove 금지). OCI 자기 rule ID 제거 뒤 SSH22와 원래 배열을 readback한다. 공유 NSG/SL은 삭제하지 않는다.
4. 전용 파일은 이 절차가 만든 이름만 지우고 빈 전용 디렉터리에 rmdir한다. 증거 JSON/hash는 승인된 비민감 보존기한까지 별도 유지한다. glob/recursive HOME 삭제는 하지 않는다.


8084 서버 중지 → **A nsg-admin에 이번 topic이 추가한 destination SECONDARY_IP/32 TCP8084 Egress의 기록된 rule ID만 제거하고 rules list에서 부재/SSH22 불변 확인** → B secondary8084 NSG의 자기 rule ID만 제거하고 guest firewall은 아래 durable block을 **OS unconfigure 이전에 한 번만** 실행한다. B에 본 실습 secondary VNIC **한 개만 존재하고 다른 secondary가 없을 때** 공식 `sudo oci-network-config unconfigure`로 secondary IP 구성을 제거한다(모든 secondary를 대상으로 하므로 다른 secondary가 있으면 이 명령 금지). primary IP/route는 그대로이며 별도 SSH에서 확인한다. Console → Compute → private-vm → Attached VNICs → exact vnic-b-test 행 Actions → **Delete VNIC** → Confirm([공식 detach/delete task](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/managingvnics_tasks-detatch.htm)) → 임시 NSG 멤버십 분리/삭제 → `ip -br addr; ip route show; ip rule show`와 primary 새 SSH를 다시 비교한다. configure/unconfigure가 남긴 DHCP/route 항목은 official utility readback으로 확인하며 임의 table200/global rp_filter 복원 명령을 실행하지 않는다. VM/primary VNIC/VCN은 creator10/22/19가 예약 소비자 종료 후 정리한다.

**guest 원복은 이 block 한 번만:** B cleanup terminal에서 본인이 작성한 owner32 private ledger를 읽는다. PENDING=1은 add 직전 query=no였던 중단 복구 상태다. 동시 firewall writer가 있으면 중단하고 owner와 조정한다. pre-existing rule(RULE_ADDED=0/PENDING=0)은 제거하지 않는다. firewalld가 예기치 않게 inactive이면 소유 ledger를 지우지 말고 관리자 복구를 기다린다.

```bash
# Same backend guest, fresh terminal recovery after interrupted/failed add.
# Confirm original exclusive owner32 lease; outside mutation means STOP/reconcile.
STATE="$HOME/owner32-firewall.state"
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

- [30] [Networking Overview — VNICs](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/overview.htm)
  - 확인 범위: VNIC.

### 실습 작업을 지원하는 추가 공식 본문

- [실습근거 1] [Virtual Network Interface Cards (VNICs)](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/managingVNICs.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
- [실습근거 2] [Network Security Groups](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/networksecuritygroups.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
