> **Legacy 보존 초안 — 기본 실행 경로 아님.** 새 개인 실습은 [주제별 기본 튜토리얼](../../NetworkSecurityGroups.md)을 사용하세요. 아래 원래 절차는 기존 참조 보존용이며 새 baseline과 혼용하지 않습니다.

> **기존 실습 초안 — 미검증**: 원래 93개 문서의 절차를 보존한 자료입니다. 새 튜토리얼의 품질 승인이나 OCI 실행 검증을 뜻하지 않습니다.

# Network Security Groups

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../../../../Docs/README.md)를 확인하세요.

> **저술 완료 · OCI 실행 미검증** — Oracle 공식 작업 문서를 근거로 작성한 한국어 hands-on입니다. Oracle 공인 교육/실행 인증이 아니며 기대 결과와 실제 결과를 구분합니다. 비용 승인과 실제 선행 산출물 확인 후에만 실행하세요.

## 학습목표

- nsg-client/nsg-web를 생성해 기존 VNIC에 연결하고 B ingress의 source NSG membership으로 TCP8080을 허용한다.
- 같은 listener와 sender egress를 고정하여 가입 성공→client membership 제거 실패→재가입 복구를 새 요청으로 확인하고 owner24 baseline을 복원한다.

## 대응 Roadmap 항목

- 항목명: **Network Security Groups**
- Roadmap 경로: **Virtual Cloud Network (VCN) → Network Security Groups**
- Node ID: `sequence-3-6`
- 상위 항목: **Virtual Cloud Network (VCN)** (`sequence-3-0`)
- 문서 범위: nsg-client/nsg-web를 생성해 기존 VNIC에 연결하고 B ingress의 source NSG membership으로 TCP8080을 허용한다. 같은 listener와 sender egress를 고정하여 가입 성공→client membership 제거 실패→재가입 복구를 새 요청으로 확인하고 owner24 baseline을 복원한다. 실제 OCI 실행 결과가 아니라 이 절차의 완료 조건이다.
- [전체 항목 목록](../../../../Docs/README.md)

## 사전준비

### 기준 계약과 실제 단계 선행조건

[공통 baseline-contract.json](../../../../baseline-contract.json)의 OS/CIDR/소유권을 따른다. Oracle Linux 9 / `opc` / `dnf` 외의 숨은 Ubuntu 또는 probe VM은 만들지 않는다. OCID/IP의 null은 실행 전 반드시 실제 출력으로 채우는 **실행 차단 입력**이다. approval/capacity/quota/identity 미확인은 완료가 아니다.

- [24 Route Tables](05_Route_Tables.md#baseline-24-ready): **A/B 관리 SSH, owner24 packet-lab/index.html=oci-packet-lab·health.txt=packet-health-ok 및8080 허용/guest ledger**를 실제 출력으로 인수한다. 이 문서의 변경/반환까지 creator 승인 expiry 안에서 보존한다.

공유 A/B consumer는 INSTANCE_A_ID에 owner10 vm-a의 실제 OCID, INSTANCE_B_ID에 owner22 private-vm의 실제 OCID, `A_PUBLIC_IP`, `A_PRIVATE_IP`, `B_PRIVATE_IP`를 실제 생성 응답에서 전달받는다. 해당 주제에 필요하지 않은 VM/변수는 생성·요구하지 않는다.

- [Route Tables](../../../../Docs/03_Virtual_Cloud_Network_VCN/05_Route_Tables.md)의 **RT-ready** 완료: 두 Linux VM의 관리 SSH와 내부 HTTP가 정상이어야 한다. `private-vm` server 8080을 유지한다.
- `private-vm` Subnet의 모든 Security List와 VNIC의 모든 NSG를 내보낸다. 8080을 넓게 허용하는 rule은 테스트 기간에 제거하되 22 관리 rule은 유지한다. 공유 VM A/B와 owner24의8080 app·파일·기존 guest rule만 승인된26~32 소비자 반환까지 보존한다. topic25 nsg-client/nsg-web는25 종료 때 membership/규칙을 원복한 뒤 삭제하며 후속27/28의 선행 자원이 아니다.

## 권한

nsg-client/nsg-web Create/Delete, source NSG rule add/remove/read와 기존 A/B VNIC NSG 배열 UpdateVnic을 승인한다. source NSG 참조의 INSPECT 및 VNIC membership 의존 권한을 함께 확인한다. owner24 baseline CIDR 전환/복원도 exact rule 범위로 승인받으며 Compute launch와 guest 공유 app 변경 권한을 요구하지 않는다.

### 실제 operation과 공식 권한

- `AddNetworkSecurityGroupSecurityRules`: NETWORK_SECURITY_GROUP _UPDATE_SECURITY_RULES and NETWORK_SECURITY_GROUP_INSPECT if writing a rule that specifies a network security group as the source (for ingress rules) or destination (for egress rules) — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `CreateNetworkSecurityGroup`: NETWORK_SECURITY_GROUP_CREATE and VCN_ATTACH — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `DeleteNetworkSecurityGroup`: NETWORK_SECURITY_GROUP_DELETE and VCN_DETACH — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `RemoveNetworkSecurityGroupSecurityRules`: NETWORK_SECURITY_GROUP _UPDATE_SECURITY_RULES — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `UpdateVnic`: VNIC_UPDATE If adding or removing the VNIC from a network security group, also need NETWORK_SECURITY_GROUP_UPDATE_MEMBERS and VNIC_ASSOCIATE_NETWORK_SECURITY_GROUP — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `ListNetworkSecurityGroupSecurityRules`: NETWORK_SECURITY_GROUP_LIST_SECURITY_RULES — [공식 API 권한 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).

위 permission 이름은 IAM policy resource-type이 아니다. 조직 관리자가 기존 group/compartment scope와 conditional dependency를 실제 API 표에 대조한다. 편의 family grant를 최소권한 정책으로 부르지 않으며 실습자는 policy 작성 권한을 자동 인수하지 않는다.

## 비용및주의사항

- 새 자원은 같은 VCN의 nsg-client/nsg-web 두 개뿐이다. A/B와 owner24의8080 app·guest rule을 재사용하고 새 VM/listener/파일을 만들지 않는다. 기존 VM/boot 유지시간만 owner10/22 승인 창으로 관리한다.
- replacement-first로 owner24 CIDR baseline을 보존한 뒤 source-NSG 조건으로 전환한다. membership deny는 최대60초 창/8초 한 요청이며 cleanup은 baseline 복원·positive를 먼저 확인한다. 자기 membership/rule/NSG만 삭제한다.

실제 OCI 작업은 아직 수행하지 않았다. 기대값은 실행 시 대조할 조건이며 무료 사용이나 과금 강제 차단을 보장하지 않는다. 승인 expiry가 끝나면 새 시험을 중단하고 위 소유권 순서로 원복·반환한다.

## 실습구성

### 이번 workshop 시나리오와 입력 계약

source NSG membership만 바꾸어 HTTP authorization 경계를 검증한다. 

|입력|값/출처|시점|실행 위치|선택 이유|
|---|---|---|---|---|
|`VCN_ID`|owner19 shared-vcn 생성 상세의 OCID; 모든 network target의 VCN 일치 확인|해당 선행 완료 또는 실제 guest 조회 후|본문에 지정한 Console/Cloud Shell 또는 guest| 모든 network target의 VCN 일치 확인|
|`A_VNIC_ID`|owner10 vm-a Attached VNICs의 primary OCID; NSG 배열 변경 대상|해당 선행 완료 또는 실제 guest 조회 후|본문에 지정한 Console/Cloud Shell 또는 guest| NSG 배열 변경 대상|
|`B_VNIC_ID`|owner22 private-vm Attached VNICs의 primary OCID; SSH/HTTP 대상 보존|해당 선행 완료 또는 실제 guest 조회 후|본문에 지정한 Console/Cloud Shell 또는 guest| SSH/HTTP 대상 보존|
|`A_PRIVATE_IP`|owner10 primary GetVnic private-ip; B ingress/guest rule의 source /32|해당 선행 완료 또는 실제 guest 조회 후|본문에 지정한 Console/Cloud Shell 또는 guest| B ingress/guest rule의 source /32|
|`B_PRIVATE_IP`|owner22 primary GetVnic private-ip; A egress/HTTP URL의 대상|해당 선행 완료 또는 실제 guest 조회 후|본문에 지정한 Console/Cloud Shell 또는 guest| A egress/HTTP URL의 대상|

|Console field|실제 입력/선택|이유|
|---|---|---|
|NSG name / VCN / source type / port|nsg-client/nsg-web; shared-vcn; source NSG=nsg-client; TCP destination8080|이번 기능/실패 원인을 한 변수로 분리|

**생성·변경 범위:** nsg-client:NetworkSecurityGroup,nsg-web:NetworkSecurityGroup. **완료 비교:** member success→nonmember deny→member restored success. 기대 패턴이며 OCI 실측값이 아니다. **정리:** 본 주제의 변경만 원복하고 전용 신규 resource만 dependency 역순으로 정리한다; 공유 자원은 creator와 consumer registry/expiry 기준으로 반환한다.

NSG 두 개 `nsg-client`/`nsg-web`를 같은 `shared-vcn`에 만든다. client VNIC만 `nsg-client`, server VNIC만 `nsg-web`에 넣는다. TCP 8080 허용 source를 IP CIDR 대신 NSG로 지정해 **Subnet 소속과 NSG 소속이 다른 속성**임을 테스트한다. 새 VM/VCN은 만들지 않는다.

## 실습절차

### 명령 실행 gate — 실패하면 다음 생성으로 진행하지 않는다

1. 아래 Bash는 **전용 Bash 세션**에서만 사용한다. Console/Cloud Shell 조회, 본인 로컬 SSH, 로그인한 Oracle Linux guest를 섞지 않는다. `exit 1`은 이 전용 세션을 닫는 안전 중단이다. `<...>`·빈 입력·타 owner OCID가 남으면 실행하지 않는다.
2. 성공을 요구하는 파일 작성/조회/설치/attach 작업은 실패 시 중단한다. 실패한 API 뒤 기존 JSON이나 기존 다운로드 파일을 성공 근거로 읽지 않는다. 기대 deny는 `if command; then ...; else RC=$?; ...; fi`로 별도 기록하고, 실제 timeout/IAM 오류/로컬 파일 오류를 구별한다.
3. 첫 변경 전에 정확한 OCID·owner·현재 설정과 승인 expiry를 저장한다. 공유 자원 변경 중 다른 소비자가 시작하거나 원복값을 확인할 수 없으면 변경을 중단한다. 삭제 성공은 API 수락이 아니라 같은 identity/Region의 정확한 대상 재조회로 판정한다. `NotAuthorizedOrNotFound` 단독은 삭제 증거가 아니다.


### 개별 완주 보정 — 생성·검증·소유권

B workload는 owner24의 packet-lab 디렉터리/본문을 그대로 유지한다. cleanup은 owner24의 exact CIDR baseline rule을 먼저 복원/readback하고 신규 positive 뒤에 source-NSG 허용을 제거하여 RT-ready의 다음 소비자가 HTTP에 접근할 수 있게 한다.

### Console 규칙 입력과 원복 ledger — TCP8080

1. **Networking → Virtual cloud networks → shared-vcn → Security → Network security groups**(또는 동일 이름 탭)에서 본문 대상 NSG의 OCID를 연다. Add Rules의 Direction은 표의 방향, Stateless는 해당 단계값, IP Protocol은 TCP(6), Source/Destination Type은 본문 CIDR 또는 같은 VCN NSG를 선택한다. Forward source port=All, destination port=8080이다. /32는 실제 peer private IP이며 internet source는 없다.
2. **Compute → Instances → 대상 → Networking → Attached VNICs → primary → Edit**에서 기존 NSG OCID 배열을 보존하고 본문에서 생성한 NSG만 추가한다. nsg-admin/nsg-private-admin의 SSH22 membership은 유지한다. Rule 제출 뒤 NSG → Security Rules와 VNIC 상세를 다시 읽고 추가 rule ID·stateful flag·source/destination·port를 ledger에 적는다. 원복은 그 ID/배열만 대상으로 한다.
3. 첫 deny 전에 복구 Console 담당자에게 exact 원복 rule/멤버십과 승인된 **60초 최대 시험 창**을 전달한다. 한 번의 8초 유한 요청 뒤 곧바로 원복한다. 진단을 위해 deny 상태를 연장하지 않는다. 원복 후 새 연결 positive가 실패하면 후속 lab/cleanup을 시작하지 말고 listener→guest firewall→OCI union→route 순으로 복구한다.


### Oracle Linux guest positive-control 준비

B의 owner24 runtime8080 rule은 소비자가 추가/삭제하지 않는다. owner24 ledger와 실제 primary interface active zone을 대조하고 읽기만 한다. firewalld가 비활성이면 기존 상태를 유지한다. 아래 exact rule 조회가 실패/absent이면 owner24 RT-ready 복구를 요청하고25 mutation을 시작하지 않는다.

```bash
# B guest; actual owner10 A_PRIVATE_IP / primary zone 지정 후
export PEER_IP='<owner10_actual_A_PRIVATE_IP>'
export FIREWALL_ZONE='<B_primary_interface_actual_active_zone>'
export HTTP_RULE="rule family=\"ipv4\" source address=\"$PEER_IP/32\" port port=\"8080\" protocol=\"tcp\" accept"
if systemctl is-active --quiet firewalld; then
  sudo firewall-cmd --zone="$FIREWALL_ZONE" --list-all >/dev/null || exit 1
  sudo firewall-cmd --zone="$FIREWALL_ZONE" --query-rich-rule="$HTTP_RULE" || exit 1
fi
```

OCI deny 전 같은 listener/guest firewall에서 positive control이 성공해야 한다. listener는 owner24가 만든 `$HOME/packet-lab/index.html`/health.txt와 그8080 process를 그대로 사용한다. 새로운 파일/server를 만들지 않고 개인 HOME root에서 HTTP를 열지 않는다. `8080`는 인터넷 source로 허용하지 않는다.

**터미널 실행 구분:** `tcpdump`는 별도 SSH 터미널에서 먼저 시작하고, `curl`/CLI workload는 다른 터미널에서 실행한다. 같은 shell에서 capture 종료를 기다렸다가 요청하면 packet이 수집되지 않는다. 본문의 유한 요청이 실패해도 담당자는 승인 deadline에 해당 HTTP/gateway 변경만 즉시 원복한다.

### A. NSG와 멤버십 생성
1. **Networking → VCN → shared-vcn → Network Security Groups → Create**에서 `nsg-client`, `nsg-web`를 생성한다. Compartment와 VCN은 두 VM과 일치시킨다. 이름만 같은 다른 VCN의 NSG를 선택하지 않는다.
2. **Compute → vm-a → Attached VNICs → primary VNIC → Edit**에서 `nsg-client`를 추가하고, `private-vm` VNIC에는 `nsg-web`를 추가한다. NSG 화면 **VNICs**에서 두 OCID를 다시 확인한다.
3. **replacement-first 전환:** owner24 baseline ingress/egress와 guest8080을 유지한 채 A의 nsg-admin에는 별도 stateful Egress B/32 TCP8080을 추가하여 client membership 제거 뒤에도 sender egress가 유지되게 한다. 추가 ID를 기록한다. 따라서 deny 원인은 server Source NSG membership이며 sender egress 제거가 아니다. `nsg-web` **Add Rules**: Direction **Ingress**, Stateful(Stateless 체크 해제), Source type **Network Security Group**, Source `nsg-client`, Protocol **TCP**, source port All, destination port **8080**. `nsg-client`에는 이 테스트용 egress를 추가하지 않는다(A의 독립 nsg-admin egress를 사용). 관리 SSH용 rule은 별도로 보존한다. replacement 두 규칙과 VNIC membership readback 뒤 신규 HTTP positive를 확인한다. 그 후에만 owner24가 추가한 TCP8080 OCI CIDR ingress/egress exact ID를 제거하고 신규 positive를 다시 확인한다. 전환부터 최대60초 독립 복구 창/원래 CIDR 저장을 먼저 승인한다; 실패면 담당자가 baseline을 먼저 복원하고25 신규 rule만 제거한다. cleanup도 owner24 CIDR을 먼저 복원/positive한 뒤25 rule을 제거하여 별도 outage를 만들지 않는다.

### B. 실제 허용과 소속 제거 실험
4. 서버에서 `sudo timeout 20s tcpdump -ni any 'tcp port 8080' -c 12`, client에서 새 `curl --noproxy '*' --max-time 8 http://<B_PRIVATE_IP>:8080/index.html`를 실행한다. HTTP 본문과 캡처를 보관한다.
5. client VNIC에서 `nsg-client` 소속만 제거한다. Source NSG rule은 그대로 두고 새 TCP 연결로 다시 curl한다. 다른 NSG/SL에서 8080을 허용하지 않는 상태라면 실패해야 한다.
6. client VNIC를 다시 `nsg-client`에 넣고 새 요청이 복구되는지 확인한다. 각 요청은 새 프로세스이며 keepalive 연결을 재사용하지 않는다. 실패가 계속되면 먼저 원래 client membership과 exact OCI rule을 복원/readback하고 B의 `ss -lntp '( sport = :8080 )'` 및 localhost `/index.html`을 읽기만 한다. 공유 owner24 listener를25가 중지/재시작하지 않는다. owner24와 모든 활성 app consumer의 반환/중단 승인 및 60초 복구 창이 없으면 BLOCKED로 남긴다. 승인된 경우에만 owner24가 자기 server terminal에서 Ctrl-C →8080 부재 확인 → `cd "$HOME/packet-lab" && python3 -m http.server 8080 --bind 0.0.0.0`로 동일 파일을 재사용한다. 별도 B terminal에서 index=oci-packet-lab/health=packet-health-ok와 A 신규 요청 성공을 확인한 뒤 consumer를 재개한다. 실패 시 원래 rules/membership을 유지하고 후속 실습을 중단한다.

## 결과검증


### baseline-25-ready


완료 단계 **NSG-tested**: NSG와 VNIC OCID, source-NSG rule, 가입 상태의 성공/탈퇴 상태의 실패/재가입 성공 세 가지 결과를 기록한다. NSG에 rule만 만들고 VNIC를 넣지 않은 상태는 완료가 아니다. deny 시 server 캡처에 SYN이 없고 같은 관리 경로는 살아 있는지 함께 확인한다.

**실행 기록(아직 미작성):** 실행자/Region/시각, exact resource OCID, 설정 전·후, 성공 출력, deny 오류·exit code/request-id, positive control, 정리 결과. 문서 검증은 실제 OCI 실행 성공을 뜻하지 않는다.

### 이 시나리오의 functional acceptance

| Positive | Negative / 미완료 판정 | Recovery / 반환 |
|---|---|---|
| source NSG 가입 index body | client membership 제거 후 같은 sender/listener의 신규 connection deny | 원래 source membership 복구 후 body; 종료 시 owner24 CIDR baseline 복구 |

모두 실행 시 확인할 기대 패턴이다. 생성 전 guard negative는 OCI 장애를 주입한 것과 다르며 다른 자원을 추가하지 않는다. 오류 메시지와 실제 exit code가 없는 항목은 NOT_EXECUTED 또는 BLOCKED로 남긴다.

## 문제해결

탈퇴 후에도 성공하면 해당 VNIC의 다른 NSG, Subnet의 모든 SL, 기존 tracked flow를 확인한다. NSG와 SL은 단순한 순차 deny firewall이 아니며 어느 허용 rule이든 통신을 허용할 수 있다. `nsg-web` source로 이름만 동일한 다른 VCN NSG를 사용할 수 없다. NSG 삭제 실패는 VNIC/부모 리소스 멤버가 남았는지 확인한다. Network 권한과 VNIC 연결 권한은 공식 NSG Required IAM Policy에 별도로 설명된다.

## 리소스정리

1. owner24가 저장한 A egress B/32:8080 및 B ingress A/32:8080 baseline을 먼저 같은 설정으로 복원/readback하고 새 /index.html 요청의 oci-packet-lab를 확인한다.
2. 그 뒤 nsg-web의 source-NSG ingress8080과25 소유 독립 A egress8080 exact ID만 제거하고 신규 HTTP positive를 다시 확인한다.
3. 원래 VNIC NSG OCID 배열로 복원하고 nsg-client/nsg-web의 members가 빈 것을 읽은 뒤 두 topic25 NSG만 Delete한다. get/list로 정확한 OCID 제거를 확인한다.27/28은 자체 포트를 사용하므로25 NSG 보존은 필요하지 않다. VM·Subnet·IGW는 owner10/22/21/23의 승인 consumer 반환까지 보존한다. owner24는 생성 owner가 아니다.

**guest 반환:**25는 guest8080 rule을 만들지 않았으므로 remove하지 않는다. owner24의 process/파일/guest ledger와 runtime rule은 그대로 반환한다. OCI 자기 rule ID와 membership만 위 순서로 정리한다.


## 공식문서

### HTTP workload의 공식 Oracle 근거와 적용 범위

[Oracle Linux 공식 HTTP 두 host 예제](https://docs.oracle.com/en/operating-systems/oracle-linux/mptcp/mptcp-testing_mptcp_subflows.html)는 B에서 `python3 -m http.server`를 실행하고 A에서 curl로 확인한다. [Oracle Linux9 공식 예제의 Python HTTP root](https://docs.oracle.com/en/learn/ol-podman-intro/)는 Oracle Linux9 image의 Python 표준 HTTP server와 working directory를 document root로 사용한다. 이 lab은 **MPTCP/Podman/container를 도입하지 않고**, 같은 HTTP 진단 원리만 기존 Oracle Linux9 guest에 적용한다. 본문의8080–8084, 명시적 index.html/health.txt, runtime /32 firewall 및 복구 trap은 이 lab의 제한접근·소유권 요구에 맞춘 저술값이며 공식 문서의 public80/root directory listing 값을 복사하지 않는다. 시작 전 `python3 -m http.server --help`에서 실제 guest의 `--bind` 지원을 확인하고 미지원이면 중단한다. Oracle Linux9/OCI에서 실행 결과는 아직 없다.


아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 아래 본문은 각 생성 필드·설정·검증·정리의 기술 근거이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [31] [Security Rules — Network Security Groups](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/securityrules.htm)
  - 확인 범위: Network security groups (NSGs) / To use an NSG, you actively.

### 실습 작업을 지원하는 추가 공식 본문

- [실습근거 1] [Network Security Groups](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/networksecuritygroups.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
- [실습근거 2] [Security Rules](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/securityrules.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
