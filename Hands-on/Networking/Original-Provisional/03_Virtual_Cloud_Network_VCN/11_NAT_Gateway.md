> **Legacy 보존 초안 — 기본 실행 경로 아님.** 새 개인 실습은 [주제별 기본 튜토리얼](../../NATGateway.md)을 사용하세요. 아래 원래 절차는 기존 참조 보존용이며 새 baseline과 혼용하지 않습니다.

> **기존 실습 초안 — 미검증**: 원래 93개 문서의 절차를 보존한 자료입니다. 새 튜토리얼의 품질 승인이나 OCI 실행 검증을 뜻하지 않습니다.

# NAT Gateway

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../../../../Docs/README.md)를 확인하세요.

> **저술 완료 · OCI 실행 미검증** — Oracle 공식 작업 문서를 근거로 작성한 한국어 hands-on입니다. Oracle 공인 교육/실행 인증이 아니며 기대 결과와 실제 결과를 구분합니다. 비용 승인과 실제 선행 산출물 확인 후에만 실행하세요.

## 학습목표

- nat-lab 한 개를 생성·재사용해 private table에 승인된 default route와 B egress443을 추가하고 public IP 없는 B의 outbound TLS를 확인한다.
- 동일 endpoint에서 Block traffic→실패→unblock 복구를 시험하고 private SSH/HTTP를 보존한다. 설치용 임시 route는 RT-ready 전에 제거한다.

## 대응 Roadmap 항목

- 항목명: **NAT Gateway**
- Roadmap 경로: **Virtual Cloud Network (VCN) → NAT Gateway**
- Node ID: `sequence-3-11`
- 상위 항목: **Virtual Cloud Network (VCN)** (`sequence-3-0`)
- 문서 범위: nat-lab 한 개를 생성·재사용해 private table에 승인된 default route와 B egress443을 추가하고 public IP 없는 B의 outbound TLS를 확인한다. 동일 endpoint에서 Block traffic→실패→unblock 복구를 시험하고 private SSH/HTTP를 보존한다. 설치용 임시 route는 RT-ready 전에 제거한다. 실제 OCI 실행 결과가 아니라 이 절차의 완료 조건이다.
- [전체 항목 목록](../../../../Docs/README.md)

## 사전준비

### 기준 계약과 실제 단계 선행조건

[공통 baseline-contract.json](../../../../baseline-contract.json)의 OS/CIDR/소유권을 따른다. Oracle Linux 9 / `opc` / `dnf` 외의 숨은 Ubuntu 또는 probe VM은 만들지 않는다. OCID/IP의 null은 실행 전 반드시 실제 출력으로 채우는 **실행 차단 입력**이다. approval/capacity/quota/identity 미확인은 완료가 아니다.

- [24 Route Tables](05_Route_Tables.md#baseline-24-ready): **A/B 관리 SSH, owner24 packet-lab/index.html=oci-packet-lab·health.txt=packet-health-ok 및8080 허용/guest ledger**를 실제 출력으로 인수한다. 이 문서의 변경/반환까지 creator 승인 expiry 안에서 보존한다.
- [29 Internet Gateway](10_Internet_Gateway.md#baseline-29-ready): **같은 owner23 IGW_ID의 is-enabled=true와 A 새 SSH/HTTPS 복구**를 실제 출력으로 인수한다. 이 문서의 변경/반환까지 creator 승인 expiry 안에서 보존한다.

공유 A/B consumer는 INSTANCE_A_ID에 owner10 vm-a의 실제 OCID, INSTANCE_B_ID에 owner22 private-vm의 실제 OCID, `A_PUBLIC_IP`, `A_PRIVATE_IP`, `B_PRIVATE_IP`를 실제 생성 응답에서 전달받는다. 해당 주제에 필요하지 않은 VM/변수는 생성·요구하지 않는다.

- [Route Tables](../../../../Docs/03_Virtual_Cloud_Network_VCN/05_Route_Tables.md) **RT-ready** 완료: A public jump host, B public IP 없는 private VM. [Internet Gateway](../../../../Docs/03_Virtual_Cloud_Network_VCN/10_Internet_Gateway.md) **IGW-tested** 완료 후 Enabled 복구 상태.
- B의 private Route Table에 IGW/다른 NAT default route가 없고 proxy/IPv6 egress도 없어야 한다. 두 VM과 NAT를 31 테스트 전 route 분리까지 보존한다.

## 권한

nat-lab Create/Get/Update/Delete와 rt-private의 자기 NAT default tuple UpdateRouteTable, B nsg-private-admin의 자기443 rule Add/Remove/read를 승인한다. NAT target 연결/해제에는 NAT_GATEWAY_ATTACH/DETACH 의존을 확인한다. VM/public IP 추가 생성 또는 전체 private table 덮어쓰기는 승인 범위가 아니다.

### 실제 operation과 공식 권한

- `CreateNatGateway`: NAT_GATEWAY_CREATE and VCN_READ and VCN_ATTACH — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `DeleteNatGateway`: NAT_GATEWAY_DELETE and VCN_READ and VCN_DETACH — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `UpdateNatGateway`: NAT_GATEWAY_UPDATE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `UpdateRouteTable`: ROUTE_TABLE_UPDATE and INTERNET_GATEWAY_ATTACH (if creating a route rule that uses an internet gateway as a target) and INTERNET_GATEWAY_DETACH (if deleting a route rule that uses an internet gateway as a target) and DRG_ATTACH (if creating a route rule that uses a DRG as a target) and DRG_DETACH (if deleting a route rule that uses a DRG as a target) and PRIVATE_IP _ROUTE_TABLE_ATTACH (if creating a route rule that uses a private IP as a target) and PRIVATE_IP _ROUTE_TABLE_DETACH (if deleting a route rule that uses a private IP as a target) and LOCAL_PEERING _GATEWAY_ATTACH (if creating a route rule that uses an LPG as a target) and LOCAL_PEERING _GATEWAY_DETACH (if deleting a route rule that uses an LPG as a target) and NAT_GATEWAY_ATTACH (if creating a route rule that uses a NAT gateway as a target) and NAT_GATEWAY_DETACH (if deleting a route rule that uses a NAT gateway as a target) and SERVICE_GATEWAY_ATTACH (if creating a route rule that uses a service gateway as a target) and SERVICE_GATEWAY_DETACH (if deleting a route rule that uses a service gateway as a target) — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `GetNatGateway`: NAT_GATEWAY_READ — [공식 API 권한 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `GetRouteTable`: ROUTE_TABLE_READ — [공식 API 권한 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `GetSubnet`: SUBNET_READ — [공식 API 권한 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `AddNetworkSecurityGroupSecurityRules`: NETWORK_SECURITY_GROUP_UPDATE_SECURITY_RULES — [공식 API 권한 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `RemoveNetworkSecurityGroupSecurityRules`: NETWORK_SECURITY_GROUP_UPDATE_SECURITY_RULES — [공식 API 권한 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `ListNetworkSecurityGroupSecurityRules`: NETWORK_SECURITY_GROUP_LIST_SECURITY_RULES — [공식 API 권한 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).

위 permission 이름은 IAM policy resource-type이 아니다. 조직 관리자가 기존 group/compartment scope와 conditional dependency를 실제 API 표에 대조한다. 편의 family grant를 최소권한 정책으로 부르지 않으며 실습자는 policy 작성 권한을 자동 인수하지 않는다.

## 비용및주의사항

- nat-lab 1개만 생성하고 설치 branch에서 이미 있으면 동일 OCID를 재사용한다. B에 public IP/reserved IP를 추가하지 않는다. NAT/공인주소/전송과 A/B Compute/boot 비용을 현재 Region 단가로 승인하고 설치용/기능시험용 종료시각을 구분한다.
- 설치 종료 후 자기 default route tuple/egress443 ID를 제거해 RT-ready로 반환한다. 기능시험 re-entry는 동일 NAT를 재연결하며 block 최대60초/요청8초 한 번이다. 소비자 반환 후 route 참조 제거→NAT 삭제→자기 egress 제거를 확인하고 공유 rt-private/VM은 삭제하지 않는다.

실제 OCI 작업은 아직 수행하지 않았다. 기대값은 실행 시 대조할 조건이며 무료 사용이나 과금 강제 차단을 보장하지 않는다. 승인 expiry가 끝나면 새 시험을 중단하고 위 소유권 순서로 원복·반환한다.

## 실습구성

### 이번 workshop 시나리오와 입력 계약

NAT 하나를 생성하고 private outbound TLS와 block/unblock을 수행한다. 

|입력|값/출처|시점|실행 위치|선택 이유|
|---|---|---|---|---|
|`VCN_ID`|owner19 shared-vcn 생성 상세의 OCID; 모든 network target의 VCN 일치 확인|해당 선행 완료 또는 실제 guest 조회 후|본문에 지정한 Console/Cloud Shell 또는 guest| 모든 network target의 VCN 일치 확인|
|`RT_PRIVATE_ID`|owner21 private-app GetSubnet route-table-id; gateway route 변경 대상|해당 선행 완료 또는 실제 guest 조회 후|본문에 지정한 Console/Cloud Shell 또는 guest| gateway route 변경 대상|
|`B_PRIVATE_IP`|owner22 primary GetVnic private-ip; A egress/HTTP URL의 대상|해당 선행 완료 또는 실제 guest 조회 후|본문에 지정한 Console/Cloud Shell 또는 guest| A egress/HTTP URL의 대상|

|Console field|실제 입력/선택|이유|
|---|---|---|
|NAT name / Block traffic / private route|nat-lab; Block traffic initially off; 0.0.0.0/0→NAT|이번 기능/실패 원인을 한 변수로 분리|

**생성·변경 범위:** nat-lab:NatGateway. **완료 비교:** B outbound TLS success→blocked deny→unblocked restored. 기대 패턴이며 OCI 실측값이 아니다. **정리:** 본 주제의 변경만 원복하고 전용 신규 resource만 dependency 역순으로 정리한다; 공유 자원은 creator와 consumer registry/expiry 기준으로 반환한다.

NAT Gateway `nat-lab` 한 개와 private default route를 만든다. `private-vm`의 공인 IP를 만들지 않고 외부로 시작하는 HTTPS만 검증한다. NAT Gateway 공인 IP는 workload로 연결하는 inbound endpoint가 아니다. 예약 공인 IP 옵션은 선택하지 않고 서비스가 지정한 주소를 기록한다.

## 실습절차

### 명령 실행 gate — 실패하면 다음 생성으로 진행하지 않는다

1. 아래 Bash는 **전용 Bash 세션**에서만 사용한다. Console/Cloud Shell 조회, 본인 로컬 SSH, 로그인한 Oracle Linux guest를 섞지 않는다. `exit 1`은 이 전용 세션을 닫는 안전 중단이다. `<...>`·빈 입력·타 owner OCID가 남으면 실행하지 않는다.
2. 성공을 요구하는 파일 작성/조회/설치/attach 작업은 실패 시 중단한다. 실패한 API 뒤 기존 JSON이나 기존 다운로드 파일을 성공 근거로 읽지 않는다. 기대 deny는 `if command; then ...; else RC=$?; ...; fi`로 별도 기록하고, 실제 timeout/IAM 오류/로컬 파일 오류를 구별한다.
3. 첫 변경 전에 정확한 OCID·owner·현재 설정과 승인 expiry를 저장한다. 공유 자원 변경 중 다른 소비자가 시작하거나 원복값을 확인할 수 없으면 변경을 중단한다. 삭제 성공은 API 수락이 아니라 같은 identity/Region의 정확한 대상 재조회로 판정한다. `NotAuthorizedOrNotFound` 단독은 삭제 증거가 아니다.


### 개별 완주 보정 — 생성·검증·소유권

NAT 준비 A는 설치만 위한 branch이며 RT-ready 이전에도 실행한다. 본 기능 시험 B는 RT-ready/IGW-tested 이후이다. NAT public-IP inbound timeout은 단독 증거가 아니며 일반 인터넷 TLS endpoint와 private same-server positive control을 함께 저장한다.

**터미널 실행 구분:** `tcpdump`는 별도 SSH 터미널에서 먼저 시작하고, `curl`/CLI workload는 다른 터미널에서 실행한다. 같은 shell에서 capture 종료를 기다렸다가 요청하면 packet이 수집되지 않는다. 본문의 유한 요청이 실패해도 담당자는 승인 deadline에 해당 HTTP/gateway 변경만 즉시 원복한다.


### nat-installation-preparation


### 본 단계의 입력/출력

|값|실제 creator/조회|실행 위치|
|---|---|---|
|VCN_ID|owner19 shared-vcn Create id|Console/Cloud Shell|
|RT_PRIVATE_ID|owner21 private-app의 route-table-id|Console/Cloud Shell|
|NAT_GATEWAY_ID / NAT_PUBLIC_IP|이 문서 Create NAT Gateway의 data.id/nat-ip; 설치 branch에서 이미 생성했다면 같은 값|Cloud Shell; 외부 보조 probe는 local|
|B_PRIVATE_IP / jump config|owner22 primary GetVnic/로컬 전용 config|로컬 접속; workload는 B guest|

생성 후 Cloud Shell에서 `export NAT_GATEWAY_ID='<본_Create의_id>'`, `export NAT_PUBLIC_IP='<동일_get의_nat-ip>'`를 지정한다. 실제 생성 전에는 그 두 값을 쓰는 명령을 실행하지 않는다. Route 추가 전 기존 RT_PRIVATE_ID 배열을 저장하고 해당 NAT rule만 제거하여 복원한다.

### A. NAT 생성과 route 연결 — 설치용 준비 단계

이 준비 단계는 [Public Subnets 준비](04_Public_Subnets.md#public-network-preparation), [Subnets](02_Subnets.md#baseline-21-ready), [Private Subnets](03_Private_Subnets.md#baseline-22-ready)의 실제 출력만 필요하다.24 RT-ready/29 IGW-tested/36 INSTANCE-PRINCIPAL-tested 완료는 **준비 단계 선행이 아니다**. 비용 승인 후 아래1~3으로 nat-lab 하나를 생성하거나 동일 OCID를 재사용한다. 기존 private route/egress 배열을 저장하고 B 설치용 HTTPS에만 사용한다. 설치 후 자기 추가 default route tuple/egress rule ID를 제거하여 원래 빈 rt-private로 복구하고24의 route negative를 수행한다. gateway는 예약 소비자/expiry가 있으면 보존하며 전체 NAT 기능 실험 B는 RT-ready/IGW-tested 후 수행한다.
1. **Networking → Virtual cloud networks → shared-vcn → Gateways → NAT Gateways → Create NAT Gateway**(legacy Resources → NAT Gateways). Name `nat-lab`, Compartment lab-root, Public IP **Ephemeral IP Address**(Oracle pool), **Route Table Association 비워 둠**, free-form purpose/owner/expiry. [Creating a NAT Gateway](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/nat-create.htm)의 Create NAT Gateway를 제출한다. 생성 기본은 Allow traffic이며 create에 Block checkbox가 있다고 가정하지 않는다. OCID/NAT public IP와 block-traffic=false를 get으로 읽는다. 이미 설치 branch에서 만든 동일 OCID가 있으면 중복 생성하지 않는다.
2. **Route Tables → rt-private → Add Route Rules**: target type **NAT Gateway**, destination CIDR `0.0.0.0/0`, target `nat-lab`. private Subnet의 실제 연결 table OCID를 재확인한다.
3. **shared-vcn → nsg-private-admin → Add Rules**: Direction Egress, Stateless off(stateful), Destination Type CIDR, Destination `0.0.0.0/0`, IP Protocol TCP, source All, destination443. 추가 rule ID를 저장한다. B ingress SSH22/SL 빈 배열은 유지한다. DHCP 기본 resolver에서 이름을 조회하고 실제 resolver 경로를 확인한다. 별도 조직 DNS를 쓰는 환경이면 실제 DNS IP/port를 승인받은 경우만 추가하고 해당 ID를 기록한다. 인터넷 ingress나 public IP는 만들지 않는다. 설치 중 repository가80/다른 endpoint를 요구하면 미리 승인받기 전 dnf를 중단한다.

### B 시작 전 full-test re-entry / network 복원 station

설치 branch에서 route/egress를 제거했으므로 B.4로 바로 건너뛰지 않는다. owner30의 동일 NAT_GATEWAY_ID/NAT_PUBLIC_IP, owner21 RT_PRIVATE_ID, B subnet/primary VNIC/IP의 실제 routing override 없음, owner24 RT-ready/owner29 IGW-tested, 승인 expiry 및 독립 Console 복구 담당자를 확인한다. 기존 nat-lab이 없어졌으면 자동 재생성하지 말고 비용/creator 재승인을 받는다.

1. Cloud Shell에서 아래 readback을 실행하고 VCN/OCID/nat-ip와 block-traffic=false를 대조한다. block 상태면 Console → exact nat-lab → Actions → Allow Traffic → 확인 뒤 같은 get을 다시 읽는다.
2. Console → shared-vcn → Route Tables → exact rt-private → Route Rules: 설치 후 baseline 배열을 저장하고 `Destination Type=CIDR`, `0.0.0.0/0`, `Target Type=NAT Gateway`, `Target=동일 NAT_GATEWAY_ID`를 **없을 때만** 추가한다. 같은 destination의 다른 target이 있으면 덮어쓰지 말고 중단한다. route에는 NSG-style rule ID를 발명하지 않고 `(destination, destination-type, network-entity-id)` tuple을 owner30 ledger에 기록한다.
3. Console → exact nsg-private-admin → Add Rules: Egress/stateful/Type CIDR/destination0.0.0.0/0/TCP/source All/destination443를 없을 때만 추가한다. exact 새 ID와 추가 여부를 기록하고 다른 owner의 기존 rule은 채택/삭제하지 않는다. owner30 전용 창 외 다른 consumer가 있으면 조정 전 중단한다. RT 재조회에서 같은 tuple 한 개, 나머지 배열 불변 및 Subnet association을 확인하고 NSG Security Rules에서 exact443 ID/flag/fields를 재확인한다. SSH22와 owner24 HTTP를 유지한다.

```bash
# Cloud Shell: actual creator outputs/Region 지정 후, 전용 증거 경로에서
oci network nat-gateway get --nat-gateway-id "$NAT_GATEWAY_ID" --region "$REGION" || exit 1
oci network route-table get --rt-id "$RT_PRIVATE_ID" --region "$REGION" || exit 1
```

B guest의 `getent ahostsv4 www.oracle.com` 성공 후 B.4의 동일 TLS endpoint positive가 성공해야 Block 시험으로 진행한다. 실패하면 차단하지 않고 owner30 추가 tuple/443 ID만 Console에서 제거 → 현재 배열/SSH/내부HTTP를 재조회하고 설치 후 baseline으로 돌아간다. gateway 자체는 승인 consumer/expiry 계약에 따라 보존/정리한다. 최종 NAT-tested 또는36 signed workload handoff까지 필요하면 lease를 갱신하며31 SGW-only 직전에는 owner30가 exact tuple/443 ID를 제거/readback한다. 새로운 NAT/VM/공인IP는 만들지 않는다.

### B. Outbound packet과 deny
4. B에서 아래 명령을 실행한다. DNS가 정상이어야 NAT 판단을 진행한다.

```bash
getent ahostsv4 www.oracle.com
# capture는 이 fence와 다른 터미널에서 먼저 실행: sudo timeout 20s tcpdump -ni any 'tcp port 443' -c 12
curl --noproxy '*' -4 -I --connect-timeout 3 --max-time 8 https://www.oracle.com/
```

5. 복구 담당자가 열린 Console에서 **shared-vcn → Gateways → NAT Gateways → nat-lab 행 Actions → Block Traffic → Block Traffic 확인**을 수행한다. [Blocking or Allowing Traffic](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/nat-block.htm)의 exact 메뉴다. 최대60초 창에서 B의8초 신규 HTTPS 한 번만 수행하고 담당자가 **Actions → Allow Traffic → Allow Traffic 확인**으로 즉시 복구한다. 진단이 실패해도60초 deadline을 연장하지 않는다. B의 신규 HTTPS 실패는 아래 RC station으로 기록한다. A→B private SSH/HTTP는 여전히 작동해야 한다. Block traffic을 해제하고 신규 HTTPS가 복구되는지 확인한다.
**단계6 전 A guest의 동일 B workload positive:** 실제 B_PRIVATE_IP를 owner22 값으로 지정하고 아래 code/body/RC 검사를 먼저 수행한다. 실패하면 외부 NAT probe를 성공 증거로 사용하지 않는다.

```bash
# A guest; owner24의 B8080은 유지
if RESPONSE=$(curl --noproxy '*' -sS --connect-timeout 3 --max-time 8 -w '\n%{http_code}' "http://$B_PRIVATE_IP:8080/index.html"); then RC=0; else RC=$?; printf 'private positive RC=%s\n' "$RC"; exit 1; fi
printf 'private positive RC=%s response=%s\n' "$RC" "$RESPONSE"
[ "$RESPONSE" = $'oci-packet-lab\n\n200' ] || exit 1
```


6. 외부 local client에서 NAT Gateway 공인 IP의8080으로 `curl --connect-timeout 3 --max-time 8 http://<NAT_PUBLIC_IP>:8080/index.html` 실행: B server로 연결되지 않아야 한다. 이 timeout만으로 NAT inbound 불가를 증명했다고 하지 않는다. 같은 B server의 A→B private HTTP 성공, NAT 공식 outbound-only 설명, NAT 주소/route 구성을 함께 기록한다.

### 상태 변경 직후의 exact target 조회

Console에서 생성/선행 owner가 전달한 실제 gateway OCID를 해당 변수에 지정하고 **허용→차단→복구 각각** 아래를 호출한다. 값이 없거나 다른 owner이면 중단한다.

```bash
oci network nat-gateway get --nat-gateway-id "$NAT_GATEWAY_ID" --region "$REGION" || exit 1
```

기대 비교: block-traffic=false/true/false; vcn-id=VCN_ID; nat-ip=생성 실제 주소. 최초 허용 요청이 실패하면 차단 실험을 시작하지 않는다. CLI 오류만으로 state를 추정하지 않고 Console/API를 읽는다. gateway와 별개로 subnet route association·NSG/SL 합집합·guest firewall·같은 endpoint/credential을 고정한다.

## 결과검증


### baseline-30-ready


완료 단계 **NAT-tested**: B public IP 없음, private default NAT rule, outbound TLS packet·응답, block 실패·unblock 복구와 private positive control. 외부에서 NAT IP로 실패한 결과는 보조 negative이며 NAT에 port forwarding rule을 만들 수 있다고 안내하지 않는다.

**실행 기록(아직 미작성):** 실행자/Region/시각, exact resource OCID, 설정 전·후, 성공 출력, deny 오류·exit code/request-id, positive control, 정리 결과. 문서 검증은 실제 OCI 실행 성공을 뜻하지 않는다.

### 이 시나리오의 functional acceptance

| Positive | Negative / 미완료 판정 | Recovery / 반환 |
|---|---|---|
| B NAT outbound TLS; private HTTP | Block Traffic 신규 TLS failure; NAT IP inbound는 보조 negative | Allow Traffic/readback 신규 TLS; SGW-only 전 NAT route 제거 |

모두 실행 시 확인할 기대 패턴이다. 생성 전 guard negative는 OCI 장애를 주입한 것과 다르며 다른 자원을 추가하지 않는다. 오류 메시지와 실제 exit code가 없는 항목은 NOT_EXECUTED 또는 BLOCKED로 남긴다.

## 문제해결

B만 실패하면 rt-private association/per-resource route, Block traffic, egress443, DNS를 확인한다. NAT IP8080 실패는 외부 network/OS firewall 때문에도 생길 수 있어 공식 설계와 함께 해석한다. SGW가 존재하면 Object Storage는 더 구체적인 service route를 선택할 수 있어 NAT HTTPS 테스트 endpoint를 일반 인터넷으로 유지한다.


## 리소스정리

### cleanup 시작 전 마지막 readback

1. 이 topic이 소유한 rule/file/resource만 목록화하고 실제 creator와 예약 consumer에게 반환을 받는다. 비용 expiry가 끝나면 새로운 consumer를 시작하지 않고 미완료 작업을 중단/원복한다. 다른 owner 리소스는 아래 본문 순서로 해당 owner가 정리한다.
2. owner30는 HTTP server/guest rich-rule/app 파일을 만들지 않았다. owner24의8080 process·packet-lab/index.html·health.txt·firewall ledger를 중지/삭제하지 않는다. private positive 성공과 app-only lease 반환만 owner24에게 전달한다.
3. NAT Allow Traffic readback을 먼저 확인한다. 설치 및 full-test re-entry에서 owner30가 추가한 route tuple/NSG443 ID만 제거/readback한다. 기존 타 owner rule/rt-private/공유 NSG를 삭제하지 않는다. consumer36의 signed calls 또는 다른 승인 consumer가 남으면 승인된 expiry까지 그 경로를 유지하고31 SGW-only와 동시 실행하지 않는다.
4. topic30는 workload 파일을 만들지 않았으므로 app 파일/rmdir 및 stale RULE_ADDED cleanup은 없다. 비민감 증거와 exact owner30 ledger만 승인된 보존기한에 따라 관리한다.


31에서 SGW-only 검증할 때 NAT default route를 먼저 제거한다. NAT가 다른 dependent workload에 필요하면 block 해제 후 보존하고 사용 목록에 consumer/owner/EXPIRY_UTC를 남긴다. expiry 도달 시 미완료 consumer를 중단/반환받고 idle 무기한 보존하지 않는다. 전체 종료는 owner30가 NAT target tuple과 자기 설치/full-test egress ID 제거 → exact NAT_GATEWAY_ID Delete → Cloud Shell get/list로 정확한 대상 제거 확인 순서다. rt-private는 owner21에게 반환하며 owner30가 삭제하지 않는다. VM 중지만으로 Volume/공인 IP 등 비용이 모두 사라졌다고 보고하지 않는다.


## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 아래 본문은 각 생성 필드·설정·검증·정리의 기술 근거이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [30] [Networking Overview — NAT Gateway](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/overview.htm)
  - 확인 범위: NETWORK ADDRESS TRANSLATION.

### 실습 작업을 지원하는 추가 공식 본문

- [실습근거 1] [NAT Gateway](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/NATgateway.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
- [실습근거 2] [VCN Route Tables](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/managingroutetables.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.

### B guest NAT 시험 RC station
positive / blocked / recovered 각각 한 번 실행한다. 같은 www.oracle.com/no-proxy/IPv4/credentials/DNS/TLS 설정을 고정한다. 각 상태에서 새 directory와 stderr를 보존한다. blocked 요청 직후 Console 담당자는 기존 단계의 exact NAT block=false를 60초 이내 복원하고 recovered 요청을 수행한다; 실패 분류 때문에 복원을 지연하지 않는다.
```bash
WORK30=$(mktemp -d "$HOME/nat30-probe.XXXXXXXX") || exit 1
getent ahostsv4 www.oracle.com > "$WORK30/dns.txt" || exit 1
if curl --noproxy '*' -4 -sS -I --connect-timeout 3 --max-time 8 https://www.oracle.com/ > "$WORK30/headers.txt" 2>"$WORK30/error.txt"; then RC30=0; else RC30=$?; fi
printf '%s\n' "$RC30" > "$WORK30/rc.txt" || exit 1
case "$RC30" in 0) printf 'HTTP headers must be 2xx/3xx\n';; 7|28) printf 'transport candidate only; require surrounding same-target positives\n';; *) printf 'INCONCLUSIVE DNS/TLS/local/service error; not NAT deny\n';; esac
```
blocked acceptance는 RC7/28 + unchanged DNS/settings + positive/recovered RC0/2xx-or3xx가 모두 필요하다. timeout 하나만 NAT 원인 증명이 아니다. 정확한 WORK30만 기록하고 관찰 종료 후 파일/rmdir 정리한다.
