# Service Gateway

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../README.md)를 확인하세요.

> **저술 완료 · OCI 실행 미검증** — Oracle 공식 작업 문서를 근거로 작성한 한국어 hands-on입니다. Oracle 공인 교육/실행 인증이 아니며 기대 결과와 실제 결과를 구분합니다. 비용 승인과 실제 선행 산출물 확인 후에만 실행하세요.

## 학습목표

- 같은 Region All Services label의 sgw-lab와 service route/egress443을 구성하고 NAT 없는 상태에서 기존 B instance principal로 private proof.txt를 읽는다.
- 인증·object·checksum을 고정해 SGW block 실패→unblock 새 다운로드 복구와 일반 인터넷 실패를 확인하고 route/gateway 소유권에 따라 반환한다.

## 대응 Roadmap 항목

- 항목명: **Service Gateway**
- Roadmap 경로: **Virtual Cloud Network (VCN) → Service Gateway**
- Node ID: `sequence-3-12`
- 상위 항목: **Virtual Cloud Network (VCN)** (`sequence-3-0`)
- 문서 범위: 같은 Region All Services label의 sgw-lab와 service route/egress443을 구성하고 NAT 없는 상태에서 기존 B instance principal로 private proof.txt를 읽는다. 인증·object·checksum을 고정해 SGW block 실패→unblock 새 다운로드 복구와 일반 인터넷 실패를 확인하고 route/gateway 소유권에 따라 반환한다. 실제 OCI 실행 결과가 아니라 이 절차의 완료 조건이다.
- [전체 항목 목록](../README.md)

## 사전준비

### 기준 계약과 실제 단계 선행조건

[공통 baseline-contract.json](../../baseline-contract.json)의 OS/CIDR/소유권을 따른다. Oracle Linux 9 / `opc` / `dnf` 외의 숨은 Ubuntu 또는 probe VM은 만들지 않는다. OCID/IP의 null은 실행 전 반드시 실제 출력으로 채우는 **실행 차단 입력**이다. approval/capacity/quota/identity 미확인은 완료가 아니다.

- [24 Route Tables](05_Route_Tables.md#baseline-24-ready): **A/B 관리 SSH, owner24 packet-lab/index.html=oci-packet-lab·health.txt=packet-health-ok 및8080 허용/guest ledger**를 실제 출력으로 인수한다. 이 문서의 변경/반환까지 creator 승인 expiry 안에서 보존한다.
- [30 NAT Gateway](11_NAT_Gateway.md#baseline-30-ready): **동일 NAT_GATEWAY_ID의 NAT-tested 결과와 owner30의 default route/임시 broad443 제거 readback**를 실제 출력으로 인수한다. 이 문서의 변경/반환까지 creator 승인 expiry 안에서 보존한다.
- [33 Identity and Access Management (IAM)](../04_Identity_and_Access_Management_IAM/00_Identity_and_Access_Management_IAM.md#baseline-33-ready): **같은 Region의 private iam-proof-a/proof.txt, NS와 관리자 검증 EXPECTED_SHA256**를 실제 출력으로 인수한다. 이 문서의 변경/반환까지 creator 승인 expiry 안에서 보존한다.
- [36 Dynamic Groups](../04_Identity_and_Access_Management_IAM/03_Dynamic_Groups.md#baseline-36-ready): **exact INSTANCE_B_ID Dynamic Group membership와 --auth instance_principal signed GET 성공**를 실제 출력으로 인수한다. 이 문서의 변경/반환까지 creator 승인 expiry 안에서 보존한다.

공유 A/B consumer는 INSTANCE_A_ID에 owner10 vm-a의 실제 OCID, INSTANCE_B_ID에 owner22 private-vm의 실제 OCID, `A_PUBLIC_IP`, `A_PRIVATE_IP`, `B_PRIVATE_IP`를 실제 생성 응답에서 전달받는다. 해당 주제에 필요하지 않은 VM/변수는 생성·요구하지 않는다.

- [Route Tables](05_Route_Tables.md) **RT-ready**, [Identity and Access Management (IAM)](../04_Identity_and_Access_Management_IAM/00_Identity_and_Access_Management_IAM.md) **IAM-fixture-ready**, [Dynamic Groups](../04_Identity_and_Access_Management_IAM/03_Dynamic_Groups.md) **INSTANCE-PRINCIPAL-tested** 완료. B VM에 `oci` CLI가 설치됐고 instance principal로 `proof.txt` 읽기가 이미 성공해야 한다.
- [NAT Gateway](11_NAT_Gateway.md) **NAT-tested** 후 private default NAT route를 제거한다. 외부 프록시·IPv6·per-resource route가 없어야 한다. 같은 Region의 private `iam-proof-a` Bucket/namespace/object를 보존한다.

## 권한

sgw-lab Create/Get/Update/Delete, rt-private의 자기 service tuple UpdateRouteTable 및 B service443 NSG rule Add/Remove/read를 승인한다. signed Object Get는 owner36 exact B Dynamic Group의 기존 OBJECT_READ이며 관리자 user credential으로 대신하지 않는다. NS/expected hash는 owner33에서 인수하고 새 Bucket/정책을 이 네트워크 consumer가 만들지 않는다.

### 실제 operation과 공식 권한

- `CreateServiceGateway`: SERVICE_GATEWAY_CREATE and VCN_READ and VCN_ATTACH ROUTE_TABLE_ATTACH is necessary to associate a route table with the service gateway during creation. — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `DeleteServiceGateway`: SERVICE_GATEWAY_DELETE and VCN_READ and VCN_DETACH — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `GetObject`: OBJECT_READ For a customer-managed key encrypted bucket, the objectstorage-<location> subject must have KEY_DECRYPT. — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm).
- `UpdateRouteTable`: ROUTE_TABLE_UPDATE and INTERNET_GATEWAY_ATTACH (if creating a route rule that uses an internet gateway as a target) and INTERNET_GATEWAY_DETACH (if deleting a route rule that uses an internet gateway as a target) and DRG_ATTACH (if creating a route rule that uses a DRG as a target) and DRG_DETACH (if deleting a route rule that uses a DRG as a target) and PRIVATE_IP _ROUTE_TABLE_ATTACH (if creating a route rule that uses a private IP as a target) and PRIVATE_IP _ROUTE_TABLE_DETACH (if deleting a route rule that uses a private IP as a target) and LOCAL_PEERING _GATEWAY_ATTACH (if creating a route rule that uses an LPG as a target) and LOCAL_PEERING _GATEWAY_DETACH (if deleting a route rule that uses an LPG as a target) and NAT_GATEWAY_ATTACH (if creating a route rule that uses a NAT gateway as a target) and NAT_GATEWAY_DETACH (if deleting a route rule that uses a NAT gateway as a target) and SERVICE_GATEWAY_ATTACH (if creating a route rule that uses a service gateway as a target) and SERVICE_GATEWAY_DETACH (if deleting a route rule that uses a service gateway as a target) — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `UpdateServiceGateway`: SERVICE_GATEWAY_UPDATE ROUTE_TABLE_ATTACH is necessary to associate a route table with the service gateway during the update. — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `GetServiceGateway`: SERVICE_GATEWAY_READ — [공식 API 권한 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `GetRouteTable`: ROUTE_TABLE_READ — [공식 API 권한 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `GetSubnet`: SUBNET_READ — [공식 API 권한 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `AddNetworkSecurityGroupSecurityRules`: NETWORK_SECURITY_GROUP_UPDATE_SECURITY_RULES — [공식 API 권한 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `RemoveNetworkSecurityGroupSecurityRules`: NETWORK_SECURITY_GROUP_UPDATE_SECURITY_RULES — [공식 API 권한 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `ListNetworkSecurityGroupSecurityRules`: NETWORK_SECURITY_GROUP_LIST_SECURITY_RULES — [공식 API 권한 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).

위 permission 이름은 IAM policy resource-type이 아니다. 조직 관리자가 기존 group/compartment scope와 conditional dependency를 실제 API 표에 대조한다. 편의 family grant를 최소권한 정책으로 부르지 않으며 실습자는 policy 작성 권한을 자동 인수하지 않는다.

## 비용및주의사항

- sgw-lab 1개와 service-destination route/egress443만 추가한다. B·instance-principal DG·iam-proof-a/proof.txt는 owner22/33/36의 보존 출력이다. 신규 Bucket/VM/NAT를 생성하지 않으며 Object Storage 저장·GET/전송 및 기존 VM 비용 창을 별도로 승인한다.
- 같은 Region service label을 route/NSG와 맞추고 NAT/default/broad443 경로를 제거한다. signed GET는 상태마다 새 파일로 최대45초, 차단은 독립 담당자가60초 전에 해제한다. auth/object/hash를 고정하고 선택 agent/DB 소비자 반환 전 SGW를 삭제하지 않는다.

실제 OCI 작업은 아직 수행하지 않았다. 기대값은 실행 시 대조할 조건이며 무료 사용이나 과금 강제 차단을 보장하지 않는다. 승인 expiry가 끝나면 새 시험을 중단하고 위 소유권 순서로 원복·반환한다.

## 실습구성

### 이번 workshop 시나리오와 입력 계약

SGW를 생성하고 NAT 없는 상태에서 signed Object Storage get을 검증한다. 

|입력|값/출처|시점|실행 위치|선택 이유|
|---|---|---|---|---|
|`VCN_ID`|owner19 shared-vcn 생성 상세의 OCID; 모든 network target의 VCN 일치 확인|해당 선행 완료 또는 실제 guest 조회 후|본문에 지정한 Console/Cloud Shell 또는 guest| 모든 network target의 VCN 일치 확인|
|`RT_PRIVATE_ID`|owner21 private-app GetSubnet route-table-id; gateway route 변경 대상|해당 선행 완료 또는 실제 guest 조회 후|본문에 지정한 Console/Cloud Shell 또는 guest| gateway route 변경 대상|
|`B_PRIVATE_IP`|owner22 primary GetVnic private-ip; A egress/HTTP URL의 대상|해당 선행 완료 또는 실제 guest 조회 후|본문에 지정한 Console/Cloud Shell 또는 guest| A egress/HTTP URL의 대상|
|`NS`|owner33의 실제 Object Storage namespace; B signed GET의 namespace|해당 선행 완료 또는 실제 guest 조회 후|본문에 지정한 Console/Cloud Shell 또는 guest| B signed GET의 namespace|
|`DG_PRIVATE_VM_OCID`|owner36 exact INSTANCE_B_ID membership Dynamic Group OCID; 인증 범위 확인|해당 선행 완료 또는 실제 guest 조회 후|본문에 지정한 Console/Cloud Shell 또는 guest| 인증 범위 확인|

|Console field|실제 입력/선택|이유|
|---|---|---|
|SGW service / route destination / egress type|All actual Region Services in OSN; matching Service route/egress TCP443|이번 기능/실패 원인을 한 변수로 분리|

**생성·변경 범위:** sgw-lab:ServiceGateway. **완료 비교:** same signed get success→blocked deny→restored; internet denied. 기대 패턴이며 OCI 실측값이 아니다. **정리:** 본 주제의 변경만 원복하고 전용 신규 resource만 dependency 역순으로 정리한다; 공유 자원은 creator와 consumer registry/expiry 기준으로 반환한다.

SGW `sgw-lab`를 생성하고 **All <Region> Services in Oracle Services Network** label을 route destination으로 사용한다. `0.0.0.0/0` 또는 임의 Object Storage IP CIDR을 넣지 않는다. Public IP 없는 B에서 동일 Region private object를 signed CLI로 읽고 일반 인터넷은 차단하는 상태를 만든다.

## 실습절차

### 작업 순서 체크포인트
같은 endpoint/object/principal의 positive→SGW block negative→unblock new GET를 수행한 뒤 authoritative consumer registry를 조회한다. gateway deletion은 마지막 반환 이후다. 단계 실패는 다음 생성으로 넘기지 않고 같은 target의 복구/반환으로 처리한다.

### 명령 실행 gate — 실패하면 다음 생성으로 진행하지 않는다

1. 아래 Bash는 **전용 Bash 세션**에서만 사용한다. Console/Cloud Shell 조회, 본인 로컬 SSH, 로그인한 Oracle Linux guest를 섞지 않는다. `exit 1`은 이 전용 세션을 닫는 안전 중단이다. `<...>`·빈 입력·타 owner OCID가 남으면 실행하지 않는다.
2. 성공을 요구하는 파일 작성/조회/설치/attach 작업은 실패 시 중단한다. 실패한 API 뒤 기존 JSON이나 기존 다운로드 파일을 성공 근거로 읽지 않는다. 기대 deny는 `if command; then ...; else RC=$?; ...; fi`로 별도 기록하고, 실제 timeout/IAM 오류/로컬 파일 오류를 구별한다.
3. 첫 변경 전에 정확한 OCID·owner·현재 설정과 승인 expiry를 저장한다. 공유 자원 변경 중 다른 소비자가 시작하거나 원복값을 확인할 수 없으면 변경을 중단한다. 삭제 성공은 API 수락이 아니라 같은 identity/Region의 정확한 대상 재조회로 판정한다. `NotAuthorizedOrNotFound` 단독은 삭제 증거가 아니다.


### 개별 완주 보정 — 생성·검증·소유권

signed get은 반드시 --auth instance_principal를 명령에도 명시한다. 상태마다 새로운 전용 다운로드 디렉터리/파일을 사용하고 CLI exit code가 0일 때만 checksum한다. private Bucket/namespace/object/Region과 policy는 block 실험 내내 변경하지 않는다.

**터미널 실행 구분:** `tcpdump`는 별도 SSH 터미널에서 먼저 시작하고, `curl`/CLI workload는 다른 터미널에서 실행한다. 같은 shell에서 capture 종료를 기다렸다가 요청하면 packet이 수집되지 않는다. 본문의 유한 요청이 실패해도 담당자는 승인 deadline에 해당 HTTP/gateway 변경만 즉시 원복한다.

### 실제 출력과 signed workload 입력

**Cloud Shell route 입력 gate (첫 snapshot 이전):** owner21가 전달한 `PRIVATE_ROUTE_TABLE_ID`를 `RT_PRIVATE_ID`로 명시 인계한다. owner22의 B primary subnet OCID도 다시 지정한다. 오래된 ROUTE_TABLE_ID를 상속하지 않는다.

```bash
export RT_PRIVATE_ID='<owner21_actual_PRIVATE_ROUTE_TABLE_ID>'
export PRIVATE_SUBNET_ID='<owner22_B_primary_subnet_actual_OCID>'
case "$RT_PRIVATE_ID" in ocid1.routetable.*) ;; *) exit 1;; esac
case "$PRIVATE_SUBNET_ID" in ocid1.subnet.*) ;; *) exit 1;; esac
export ROUTE_TABLE_ID="$RT_PRIVATE_ID"
ASSOCIATED_RT=$(oci network subnet get --subnet-id "$PRIVATE_SUBNET_ID" --region "$REGION" --query 'data."route-table-id"' --raw-output) || exit 1
test "$ASSOCIATED_RT" = "$ROUTE_TABLE_ID" || { printf 'STOP: B subnet route association mismatch\n'; exit 1; }
```

cleanup의 새 Cloud Shell에도 위 alias/association gate를 다시 실행한다. subnet 조회 권한은 owner21/22의 관리자에게 승인받고 실패하면 route 변경 금지다.

SGW_ID는 본 Create 응답 id, NS는 owner33의 caller namespace, EXPECTED_SHA256는 owner33 관리자 GET으로 확인한 **같은 Region/iam-proof-a/proof.txt**의 SHA256다. DG_PRIVATE_VM_OCID는 owner36의 exact B instance membership Group OCID이며 credential file이 아니다. B guest의 OCI_CLI_REGION은 승인 identifier다. 생성 후 Cloud Shell에서 `export SGW_ID='<본_Create의_id>'`, 조회용 `export OCI_CLI_REGION='<같은_REGION>'`를 지정하고 gateway get을 수행한다. B guest에는 NS/Region/expected checksum만 별도로 넣고 API private key를 복사하지 않는다. hash를 모르거나 fixture 보존 승인이 없으면 Get/차단을 시작하지 않는다.

### 일반 인터넷 negative의 동일 host positive 준비

A의 인터넷 성공만으로 B negative를 승인하지 않는다. A.3의 NAT tuple/443 제거 **전에**, owner30/36의 승인된 NAT 보존 창에서 B guest로 `getent ahostsv4 www.oracle.com`와 아래 probe를 INTERNET_PHASE=positive로 실행하여 같은 endpoint TLS/header/RC0를 기록한다. 이후 NAT/default/broad443 제거 readback과 SGW signed positive를 완료한 뒤 INTERNET_PHASE=negative를 실행한다. credential/endpoint/IPv4/no-proxy/guest firewall/DNS를 고정한다. endpoint 장애나 TLS/DNS/local 오류면 BLOCKED이며 추가 NAT 생성으로 우회하지 않는다. 회복 관측은 owner30가 다음 승인 NAT 창에 exact 경로를 복원한 때 INTERNET_PHASE=recovered로 같은 B probe RC0를 확인한다. SGW-only 중에는 NAT를 재추가하지 않는다; 끝까지 SGW를 보존할 branch이면 internet restoration은 NOT_SELECTED이고 SGW unblock signed checksum이 recovery다.

```bash
# B guest; positive/recovered는 승인 NAT 창, negative는 SGW-only에서
INTERNET_PHASE='<positive_or_negative_or_recovered>'
case "$INTERNET_PHASE" in positive|negative|recovered) ;; *) exit 1;; esac
INET_DIR="$HOME/owner31-internet-$INTERNET_PHASE"
if test -e "$INET_DIR" || test -L "$INET_DIR"; then exit 1; fi
mkdir -m 700 "$INET_DIR" || exit 1
getent ahostsv4 www.oracle.com > "$INET_DIR/dns.txt" || exit 1
if curl --noproxy '*' -4 -sS -I --connect-timeout 3 --max-time 8 https://www.oracle.com/ > "$INET_DIR/headers.txt" 2> "$INET_DIR/curl.stderr"; then RC=0; else RC=$?; fi
printf '%s\n' "$RC" > "$INET_DIR/rc.txt" || exit 1
case "$INTERNET_PHASE:$RC" in
  positive:0|recovered:0) test -s "$INET_DIR/headers.txt" || exit 1;;
  negative:7|negative:28) printf '%s\n' 'transport candidate only: require B positive, DNS, same target and SGW-only routing readback';;
  *) printf 'STOP: unexpected/unclassified phase=%s rc=%s\n' "$INTERNET_PHASE" "$RC"; exit 1;;
esac
# 5/6 DNS/proxy, 35/60 TLS, 23 local write, 22 HTTP errors는 network deny 아님.
# rc7 connection refusal/endpoint outage도 가능하므로 stderr/positive/readback 없이는 승인 금지.
```

### A. Service Gateway 생성
1. **Networking → Virtual cloud networks → shared-vcn → Gateways → Service Gateways → Create Service Gateway**(legacy Resources 탭). Name `sgw-lab`, Compartment lab-root, **Services: All <현재 Region> Services in Oracle Services Network**, **Route Table Association 없음**(gateway ingress/transit routing 불필요), free-form purpose/owner/expiry를 선택한다. [Creating a Service Gateway](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/create-sgw.htm)의 Create Service Gateway로 제출하고 SGW_ID/block-traffic=false/services label을 조회한다. 정확한 region label을 드롭다운에서 복사한다. Object Storage-only와 All Services label을 동시에 잘못 조합하지 않는다.
**route 추가 전 Cloud Shell snapshot:** rt-private의 실제 OCID를 owner21 출력으로 받고 `oci network route-table get --rt-id "$ROUTE_TABLE_ID" --region "$REGION" > owner31-route-before.json || exit 1`를 실행한다. JSON 전체 route-rules를 보존한다. 제출 뒤 동일 get을 owner31-route-after.json으로 저장하고 새 route의 exact `(destination, destination-type=SERVICE_CIDR_BLOCK, network-entity-id=SGW_ID)` 한 개와 나머지 배열 불변을 대조한다. 이 tuple을 owner31 route ledger에 기록한다.

2. **rt-private → Add Route Rules**: Destination type **Service**, destination 위와 같은 label, target type **Service Gateway**, target `sgw-lab`. SGW의 services 선택과 route destination이 일치하는지 확인한다.
3. Networking → shared-vcn → nsg-private-admin → Add Rules: Direction Egress, Stateless off, Destination Type Service, Destination 같은 All Region Services label, IP Protocol TCP, source port All, destination443. 추가 rule ID를 기록한다. 시험 전에 owner30 승인하에 NAT default route와 설치용0.0.0.0/0 egress443의 exact 추가 ID를 제거하고 readback한다. 기존 SSH22는 유지한다. IGW/default/override 경로도 없음을 확인한다. DNS resolver는 DHCP/VCN resolver를 계속 사용한다.

### B. Signed workload와 일반 인터넷 차단
4. B에서 실제 Region/namespace/bucket을 환경변수로 입력하고 파일을 읽는다. API key를 VM에 업로드하지 않는다.

```bash
export OCI_CLI_AUTH=instance_principal
export OCI_CLI_REGION='<REGION_IDENTIFIER>'
export NS='<ACTUAL_NAMESPACE>'
# B guest: 매 상태(positive/blocked/recovered)마다 새 전용 경로 생성
export EXPECTED_SHA256='<owner33_admin의_동일_proof.txt_실제_SHA256>'
case "$EXPECTED_SHA256" in *'<'*|*'>'*|'') printf '%s\n' 'STOP: fixture hash required'; exit 1;; esac
python3 - <<'PY'
import os,re
assert re.fullmatch(r'[0-9a-f]{64}',os.environ['EXPECTED_SHA256']), 'STOP: exact lowercase SHA256 from creator required'
assert '<' not in os.environ['NS'] and os.environ['NS'], 'STOP: actual namespace required'
assert '<' not in os.environ['OCI_CLI_REGION'] and os.environ['OCI_CLI_REGION'], 'STOP: actual Region required'
PY
if [ "$?" -ne 0 ]; then exit 1; fi
export SGW_PHASE='<positive_or_blocked_or_recovered>'
case "$SGW_PHASE" in positive|blocked|recovered) ;; *) exit 1;; esac
SGW_DIR="$HOME/oci-roadmap-sgw-$SGW_PHASE"
test ! -e "$SGW_DIR" || { printf '%s\n' 'STOP: fresh state directory required'; exit 1; }
mkdir -m 700 "$SGW_DIR" || exit 1
if timeout 45s oci os object get --auth instance_principal --region "$OCI_CLI_REGION" --namespace-name "$NS" --bucket-name iam-proof-a --name proof.txt --file "$SGW_DIR/proof.txt" --max-retries 0 2>"$SGW_DIR/get.stderr"; then
  RC=0
  test "$SGW_PHASE" != blocked || { printf '%s\n' 'FAIL: blocked gateway still succeeded'; exit 1; }
  test -f "$SGW_DIR/proof.txt" || exit 1
  ACTUAL_SHA256="$(sha256sum "$SGW_DIR/proof.txt" | cut -d ' ' -f 1)" || exit 1
  test "$ACTUAL_SHA256" = "$EXPECTED_SHA256" || exit 1
else
  RC=$?
  test "$SGW_PHASE" = blocked || { printf 'STOP: expected success, rc=%s\n' "$RC"; exit 1; }
  # No automatic deny credit for arbitrary nonzero. Unblock before diagnosis below.
  printf '%s\n' 'MANDATORY: Console operator must unblock now; then classify captured stderr'
  export RC
  python3 - "$SGW_DIR/get.stderr" <<'PY'
import os,sys,re
s=open(sys.argv[1]).read()
assert not re.search(r'NotAuthorized|NotAuthenticated|ServiceError|Permission denied|No such file|Traceback.*FileNotFound|Invalid|Unknown option',s,re.I), 'STOP: auth/API/local error is not SGW deny'
assert int(os.environ['RC'])==124 or re.search(r'ConnectTimeout|ReadTimeout|timed out|ConnectionError|Network is unreachable',s,re.I), 'STOP: unclassified failure; no deny acceptance'
print('transport candidate only; verify same-target recovered checksum and unchanged credentials')
PY
  if [ "$?" -ne 0 ]; then exit 1; fi
fi
printf 'get phase=%s exit status=%s\n' "$SGW_PHASE" "$RC"
# 인터넷 negative는 위 별도 probe를 실행한다. SGW_PHASE별 signed fence 안에서 반복 생성하지 않는다.
```

5. 서버/클라이언트 `tcpdump -ni any 'tcp port 443'`로 OSN endpoint TCP/TLS packet을 기록한다. OSN 주소가 public endpoint로 resolve돼도 SGW route를 사용하면 공용 인터넷 경로라는 뜻은 아니다. Guest capture만으로 Gateway 내부 hop이 보인다고 주장하지 않는다.
6. 복구 담당자가 Console → shared-vcn → Service Gateways → sgw-lab → Block traffic에서 차단한다. 최대60초 창에서 SGW_PHASE=blocked의 같은 fenced 명령을 한 번 실행하고 즉시 unblock한다. 최대45초 명령이 끝나지 않아도 담당자가60초 시점에 먼저 원복한다. SGW_PHASE=recovered로 새 파일/hash 성공을 확인하고 기존 negative 증거를 보존한다. 재실행은 새 승인 경로를 준비한 경우만 하며 기존 디렉터리를 덮어쓰지 않는다. Object permission은 이 상태 전환 동안 바꾸지 않는다.

**다운로드 negative 안전:** 매 상태에서 새 전용 output filename을 사용하거나 이전에 본 실습이 만든 파일만 제거한 뒤 호출한다. local file이 이미 있다는 CLI overwrite prompt/기존 hash를 서비스 deny/success로 해석하지 않는다. 명령 종료코드와 새 파일 생성 여부를 함께 기록하고 실패한 요청 뒤 오래된 파일의 checksum을 성공 증거로 쓰지 않는다.

### 상태 변경 직후의 exact target 조회

Console에서 생성/선행 owner가 전달한 실제 gateway OCID를 해당 변수에 지정하고 **허용→차단→복구 각각** 아래를 호출한다. 값이 없거나 다른 owner이면 중단한다.

```bash
oci network service-gateway get --service-gateway-id "$SGW_ID" --region "$OCI_CLI_REGION" || exit 1
```

기대 비교: block-traffic=false/true/false; services의 service-id/cidr-block=선택한 같은 Region label. 최초 허용 요청이 실패하면 차단 실험을 시작하지 않는다. CLI 오류만으로 state를 추정하지 않고 Console/API를 읽는다. gateway와 별개로 subnet route association·NSG/SL 합집합·guest firewall·같은 endpoint/credential을 고정한다.

## 결과검증

<a id="baseline-31-ready"></a>

완료 단계 **SGW-tested**: NAT/IGW default 없음, service label/target OCID, private object hash 일치, SGW block 실패·복구 성공, 일반 인터넷 실패. HTTP401/404 같은 unsigned 응답만으로 private bucket workload 완료로 처리하지 않는다. Region이 다른 Bucket이면 지원 경로 검증과 섞이지 않도록 동일 Region으로 바꾼다.

**실행 기록(아직 미작성):** 실행자/Region/시각, exact resource OCID, 설정 전·후, 성공 출력, deny 오류·exit code/request-id, positive control, 정리 결과. 문서 검증은 실제 OCI 실행 성공을 뜻하지 않는다.

### 이 시나리오의 functional acceptance

| Positive | Negative / 미완료 판정 | Recovery / 반환 |
|---|---|---|
| fresh signed proof hash; no NAT default | blocked same get transport failure; generic internet deny | unblock/recovered fresh file hash; credential/object fixed |

모두 실행 시 확인할 기대 패턴이다. 생성 전 guard negative는 OCI 장애를 주입한 것과 다르며 다른 자원을 추가하지 않는다. 오류 메시지와 실제 exit code가 없는 항목은 NOT_EXECUTED 또는 BLOCKED로 남긴다.

## 문제해결

object401/404는 IAM/namespace/이름/Region 문제일 수 있다. SGW 활성일 때36의 positive control이 성공하는지 먼저 확인한다. 통신 timeout이면 route와 service label·SGW block·egress443·DNS를 점검한다. internet 성공이면 NAT default/per-resource route/proxy/IPv6를 찾아 제거한다. Instance principal token 발급도 네트워크가 필요하므로 auth 오류와 Object Storage packet 오류를 구별한다.

## 리소스정리

### owner31 route 반환 — shared table 전체 덮어쓰기 금지

모든 선택 consumer 반환 후 Cloud Shell에서 현재 rt-private를 새 JSON으로 조회하고 Console → rt-private → Route Rules → 기록한 **service destination/SGW_ID tuple 한 개만** Edit/Delete한다. 타 owner route가 추가됐으면 보존한다. 삭제 전/후 아래 get으로 동일 tuple 부재와 타 route 불변을 비교한 후 SGW Delete로 이동한다.

```bash
oci network route-table get --rt-id "$ROUTE_TABLE_ID" --region "$REGION" > owner31-route-cleanup-before.json || exit 1
# 위 Console exact tuple 한 개 제거 후 실행 (전체 배열 교체하지 않음)
oci network route-table get --rt-id "$ROUTE_TABLE_ID" --region "$REGION" > owner31-route-cleanup-after.json || exit 1
python3 - <<'PY'
import json,os
before=json.load(open('owner31-route-cleanup-before.json'))['data']['route-rules']
after=json.load(open('owner31-route-cleanup-after.json'))['data']['route-rules']
owned=[x for x in before if x['network-entity-id']==os.environ['SGW_ID']]
assert len(owned)==1 and owned[0]['destination-type']=='SERVICE_CIDR_BLOCK'
assert after==[x for x in before if x not in owned], 'STOP: other owner route changed'
PY
if [ "$?" -ne 0 ]; then exit 1; fi
```

다른 table에서도 SGW_ID target 참조가 없는지 creator registry와 전체 table 목록으로 확인한다. 부재 미확인이면 gateway 삭제를 중단한다.


SGW block 해제. owner31 authoritative registry에서 선택42 Functions, 55/57/58/59 pool/metric agent, 60 중 Monitoring을 사용하는 consumer, 선택87–92 DB network consumer를 SELECTED/NOT_SELECTED·owner·SGW/route/rule ID·expiry·반환 상태로 조회한다. 실제 선택 consumer 전원의 복구/return 또는 승인 abort 전 SGW/route/egress를 보존한다. expiry에는 신규 시험 중단→각 owner 복구/반환이며 무승인 비용 연장은 금지한다. registry 미확인/참조 잔존이면 삭제 중단한다. 모든 소비자 반환과 미참조 readback 뒤에만 owner31가 SGW target route 삭제 → exact SGW_ID 삭제 → 자기 service egress rule ID 제거 뒤 같은 Region get/list에서 gateway 부재와 원래 route/rule 배열을 읽는다. rt-private/sl-private는 owner21에게 반환하며 삭제하지 않는다. Instance principal 정책/그룹·Bucket은 IAM 실습 종료까지 보존한다. B에서 이 lab이 만든 `$HOME/oci-roadmap-sgw-positive`, `-blocked`, `-recovered`의 자기 proof.txt(부분 다운로드 포함)·get.stderr를 아래 guarded block으로 정리하고 빈 디렉터리를 제거한다. 다른 /tmp/파일은 삭제하지 않는다.


```bash
# B guest: 이번 실행에서 직접 생성한 경로만; 증거 보존/반환 승인 후
for phase in positive blocked recovered; do
  d="$HOME/oci-roadmap-sgw-$phase"
  if test -L "$d"; then exit 1; fi
  if test -e "$d"; then
    test -d "$d" && test -O "$d" || exit 1
    for f in proof.txt get.stderr; do
      if test -L "$d/$f"; then exit 1; fi
      if test -e "$d/$f"; then test -f "$d/$f" && test -O "$d/$f" || exit 1; rm -- "$d/$f" || exit 1; fi
    done
    rmdir -- "$d" || exit 1
  fi
done
for phase in positive negative recovered; do
  d="$HOME/owner31-internet-$phase"
  if test -L "$d"; then exit 1; fi
  if test -e "$d"; then
    test -d "$d" && test -O "$d" || exit 1
    for f in dns.txt headers.txt curl.stderr rc.txt; do
      if test -L "$d/$f"; then exit 1; fi
      if test -e "$d/$f"; then test -f "$d/$f" && test -O "$d/$f" || exit 1; rm -- "$d/$f" || exit 1; fi
    done
    rmdir -- "$d" || exit 1
  fi
done
```

미생성 phase는 건너뛰고 다른 파일로 rmdir가 실패하면 중단한다. glob/recursive 삭제는 없다.


## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 아래 본문은 각 생성 필드·설정·검증·정리의 기술 근거이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [30] [Networking Overview — Service Gateway](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/overview.htm)
  - 확인 범위: SERVICE GATEWAY.

### 실습 작업을 지원하는 추가 공식 본문

- [실습근거 1] [Access to Oracle Services: Service Gateway](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/servicegateway.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
- [실습근거 2] [VCN Route Tables](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/managingroutetables.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
- [실습근거 3] [Details for Object Storage and Archive Storage](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
