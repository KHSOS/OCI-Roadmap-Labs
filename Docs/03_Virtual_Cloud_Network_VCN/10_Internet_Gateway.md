# Internet Gateway

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../README.md)를 확인하세요.

> **저술 완료 · OCI 실행 미검증** — Oracle 공식 작업 문서를 근거로 작성한 한국어 hands-on입니다. Oracle 공인 교육/실행 인증이 아니며 기대 결과와 실제 결과를 구분합니다. 비용 승인과 실제 선행 산출물 확인 후에만 실행하세요.

## 학습목표

- owner23의 동일 igw-lab/public route와 owner10 public IP를 대조하고 새 SSH/HTTPS positive control을 확보한다.
- Console 복구 담당자와 유한 창에서 IGW enabled→disabled 신규 연결 실패→enabled 복구를 확인하고 같은 owner에게 반환한다.

## 대응 Roadmap 항목

- 항목명: **Internet Gateway**
- Roadmap 경로: **Virtual Cloud Network (VCN) → Internet Gateway**
- Node ID: `sequence-3-10`
- 상위 항목: **Virtual Cloud Network (VCN)** (`sequence-3-0`)
- 문서 범위: owner23의 동일 igw-lab/public route와 owner10 public IP를 대조하고 새 SSH/HTTPS positive control을 확보한다. Console 복구 담당자와 유한 창에서 IGW enabled→disabled 신규 연결 실패→enabled 복구를 확인하고 같은 owner에게 반환한다. 실제 OCI 실행 결과가 아니라 이 절차의 완료 조건이다.
- [전체 항목 목록](../README.md)

## 사전준비

### 기준 계약과 실제 단계 선행조건

[공통 baseline-contract.json](../../baseline-contract.json)의 OS/CIDR/소유권을 따른다. Oracle Linux 9 / `opc` / `dnf` 외의 숨은 Ubuntu 또는 probe VM은 만들지 않는다. OCID/IP의 null은 실행 전 반드시 실제 출력으로 채우는 **실행 차단 입력**이다. approval/capacity/quota/identity 미확인은 완료가 아니다.

- [24 Route Tables](05_Route_Tables.md#baseline-24-ready): **A/B 관리 SSH, owner24 packet-lab/index.html=oci-packet-lab·health.txt=packet-health-ok 및8080 허용/guest ledger**를 실제 출력으로 인수한다. 이 문서의 변경/반환까지 creator 승인 expiry 안에서 보존한다.

공유 A/B consumer는 INSTANCE_A_ID에 owner10 vm-a의 실제 OCID, INSTANCE_B_ID에 owner22 private-vm의 실제 OCID, `A_PUBLIC_IP`, `A_PRIVATE_IP`, `B_PRIVATE_IP`를 실제 생성 응답에서 전달받는다. 해당 주제에 필요하지 않은 VM/변수는 생성·요구하지 않는다.

- [Route Tables](05_Route_Tables.md) **RT-ready** 완료. public VM 공인 IPv4, `rt-public`의 IGW rule 및 SSH 정상 상태가 필요하다.
- [Public Subnets](04_Public_Subnets.md) **Public IP 허용 Subnet** 및 [Compute](../02_Compute/00_Compute.md#baseline-10-ready)의 동일 vm-a primary VNIC ephemeral public-ip/private-ip readback을 인수한다. [Public IP Addresses](../02_Compute/06_Public_IP_Addresses.md)는 선택 설명 참고만이며 reserved IP/추가VM 생성 완료를 요구하지 않는다. Console 복구자는 owner23의 동일 IGW Enabled를 복구할 수 있어야 한다. 기존 두 VM·IGW는 선택30~32의 예약 consumer/expiry가 있을 때만 보존한다.

## 권한

owner23의 동일 IGW Get/Update와 owner10 primary VNIC/rt-public 조회만 승인한다. 독립 Console 복구 담당자에게 Enable 권한이 있어야 disable을 시작한다. IGW Create/Delete 및 route target 변경은 owner23 creator 작업이며 이 Enabled 시험의 필수 권한이 아니다.

### 실제 operation과 공식 권한

- `GetInternetGateway`: INTERNET_GATEWAY_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `GetVnic`: VNIC_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `UpdateInternetGateway`: INTERNET_GATEWAY_UPDATE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `GetRouteTable`: ROUTE_TABLE_READ — [공식 API 권한 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).

위 permission 이름은 IAM policy resource-type이 아니다. 조직 관리자가 기존 group/compartment scope와 conditional dependency를 실제 API 표에 대조한다. 편의 family grant를 최소권한 정책으로 부르지 않으며 실습자는 policy 작성 권한을 자동 인수하지 않는다.

## 비용및주의사항

- 신규 IGW/VM/reserved IP는 없다. owner23 igw-lab와 owner10 vm-a ephemeral 주소를 재사용한다. B를 이 시험용으로 새로 생성하거나 reserved address 실습을 필수 선행으로 요구하지 않는다.
- IGW disable은 public/jump 소비자 전체에 영향을 준다. owner23와 소비자 중단 승인·독립 Console 담당자 없으면 실행 금지다.10초 신규 SSH 한 번 후 최대60초 전에 Enable하고 새 SSH/HTTPS 복구를 확인한다. gateway 삭제는 owner23의 책임이다.

실제 OCI 작업은 아직 수행하지 않았다. 기대값은 실행 시 대조할 조건이며 무료 사용이나 과금 강제 차단을 보장하지 않는다. 승인 expiry가 끝나면 새 시험을 중단하고 위 소유권 순서로 원복·반환한다.

## 실습구성

### 이번 workshop 시나리오와 입력 계약

기존 IGW를 Console에서 disable/enable해 public 신규 연결과 복구를 확인한다. 

|입력|값/출처|시점|실행 위치|선택 이유|
|---|---|---|---|---|
|`IGW_ID`|owner23 igw-lab OCID; rt-public target 및 Enabled 상태 대조|해당 선행 완료 또는 실제 guest 조회 후|본문에 지정한 Console/Cloud Shell 또는 guest| rt-public target 및 Enabled 상태 대조|
|`A_PUBLIC_IP`|owner10의 actual output; `<...>` 대체 필수|실행 전 선행 완료값|Console/사용자 전용 shell|정확한 대상/소유관계 고정|

|Console field|실제 입력/선택|이유|
|---|---|---|
|IGW / Enabled / route target|igw-lab owner23; disabled briefly only with Console recovery operator|이번 기능/실패 원인을 한 변수로 분리|

**생성·변경 범위:** 새 infrastructure 없음; existing resource의 configuration만 변경. **완료 비교:** enabled success→disabled new-connection deny→enabled restored. 기대 패턴이며 OCI 실측값이 아니다. **정리:** 본 주제의 변경만 원복하고 전용 신규 resource만 dependency 역순으로 정리한다; 공유 자원은 creator와 consumer registry/expiry 기준으로 반환한다.

23의 public-network-preparation에서 만든 `igw-lab`를 재사용한다. Public Subnet이라는 이름만으로 인터넷 접속이 되지 않는다는 것을 **공인 IP + enabled IGW + route + security**의 조합으로 검증한다. 새 IGW를 중복 생성하지 않는다. Gateway에 ingress route table을 연결하는 고급 routing은 사용하지 않는다.

## 실습절차

### 명령 실행 gate — 실패하면 다음 생성으로 진행하지 않는다

1. 아래 Bash는 **전용 Bash 세션**에서만 사용한다. Console/Cloud Shell 조회, 본인 로컬 SSH, 로그인한 Oracle Linux guest를 섞지 않는다. `exit 1`은 이 전용 세션을 닫는 안전 중단이다. `<...>`·빈 입력·타 owner OCID가 남으면 실행하지 않는다.
2. 성공을 요구하는 파일 작성/조회/설치/attach 작업은 실패 시 중단한다. 실패한 API 뒤 기존 JSON이나 기존 다운로드 파일을 성공 근거로 읽지 않는다. 기대 deny는 `if command; then ...; else RC=$?; ...; fi`로 별도 기록하고, 실제 timeout/IAM 오류/로컬 파일 오류를 구별한다.
3. 첫 변경 전에 정확한 OCID·owner·현재 설정과 승인 expiry를 저장한다. 공유 자원 변경 중 다른 소비자가 시작하거나 원복값을 확인할 수 없으면 변경을 중단한다. 삭제 성공은 API 수락이 아니라 같은 identity/Region의 정확한 대상 재조회로 판정한다. `NotAuthorizedOrNotFound` 단독은 삭제 증거가 아니다.


### 개별 완주 보정 — 생성·검증·소유권

Disable은 복구 담당자의 열린 Console와 승인된 짧은 시험 창에서만 진행한다. 다른 승인 consumer의 연결이 실행 중이면 먼저 중단/반환받는다. owner23의 Enabled와 원래 route association을 복구한 뒤 반환한다.

**터미널 실행 구분:** `tcpdump`는 별도 SSH 터미널에서 먼저 시작하고, `curl`/CLI workload는 다른 터미널에서 실행한다. 같은 shell에서 capture 종료를 기다렸다가 요청하면 packet이 수집되지 않는다. 본문의 유한 요청이 실패해도 담당자는 승인 deadline에 해당 HTTP/gateway 변경만 즉시 원복한다.

### A. IGW와 public IP 확인
1. **VCN → Internet Gateways → igw-lab**에서 Compartment/VCN/Enabled를 확인한다. 없으면23의 public-network-preparation으로 돌아가 생성/readback을 완료한다. 이 문서에서 두 번째 IGW/route table을 만들지 않는다.
2. **Compute → vm-a → VNIC → IPv4 Addresses**에서 primary private IP와 public IP를 기록한다. **Subnet**이 public IP 허용인지 확인한다. private VM에는 public IP를 할당하지 않는다.
3. public egress TCP443이 허용되는지 확인하고 public VM에서 DNS resolve와 HTTPS를 실행한다.

```bash
getent ahostsv4 www.oracle.com
curl --noproxy '*' -4 -I --connect-timeout 3 --max-time 8 https://www.oracle.com/
# capture는 이 fence와 다른 터미널에서 먼저 실행: sudo timeout 20s tcpdump -ni any 'tcp port 443' -c 12
```

### B. Gateway deny와 복구
4. 먼저 복구 담당자가 Console → Networking → shared-vcn → Internet Gateways → 정확한 igw-lab OCID 상세를 열어 Enable 권한/작업 경로를 확인한다. owner23와 모든 public 소비자의60초 중단 창을 승인받고 승인 종료시각을 기록한다. disable 후 **한 번의10초 신규 SSH probe**만 수행하고 최대60초 전에 담당자가 반드시 Enable한다. SSH가 끊기거나 진단이 실패해도 원복 deadline을 연장하지 않는다. 이 조건 없이는 Disable하지 않는다. 관리자 Console에서 `igw-lab`를 **Disable**한다. SSH도 끊길 수 있으므로 local SSH만으로 복구하려고 하지 않는다. **Disable readback → 아래 local10초 신규 SSH probe 한 번 → 실패 RC/transport 기록 → 즉시 Enable → enabled readback → 같은 local 신규 SSH positive → guest HTTPS positive** 순서다. probe가 멈추거나 분류가 안 돼도 담당자가60초 전에 Enable한다. host/network capture를 할 수 없는 시간은 미수집으로 남긴다.
5. Enabled 복구 후 신규 SSH와 HTTPS가 회복되는지 확인한다. 기존 연결 유지 여부는 신규 연결 허용의 증거가 아니다.
6. private VM의 public-ip=null과 owner24의 empty-private-route negative 증거만 연결한다. 이 topic의 primary 비교는 public IGW enabled/disabled이며 private table에 IGW route를 추가하는 별도 변경은 필요하지 않아 수행하지 않는다.

### 상태 변경 직후의 exact target 조회

Console에서 생성/선행 owner가 전달한 실제 gateway OCID를 해당 변수에 지정하고 **허용→차단→복구 각각** 아래를 호출한다. 값이 없거나 다른 owner이면 중단한다.

```bash
oci network internet-gateway get --ig-id "$IGW_ID" --region "$REGION" || exit 1
```

기대 비교: is-enabled=true/false/true; vcn-id=VCN_ID. 최초 허용 요청이 실패하면 차단 실험을 시작하지 않는다. CLI 오류만으로 state를 추정하지 않고 Console/API를 읽는다. gateway와 별개로 subnet route association·NSG/SL 합집합·guest firewall·같은 endpoint/credential을 고정한다.

### 신규 연결 probe는 본인 로컬에서 한 번씩 실행한다

```bash
# local terminal; 이 gate는 다음 상태 변경 전에 완료한다
case "$A_PUBLIC_IP" in ''|*'<'*|*'>'*) exit 1;; esac
test -r "$KEY" || exit 1
if timeout 10s ssh -o ConnectTimeout=5 -o ConnectionAttempts=1 -o ControlMaster=no -o ControlPath=none -i "$KEY" opc@"$A_PUBLIC_IP" 'test "$(id -un)" = opc'; then
  printf '%s\n' 'new SSH connected; compare with enabled/disabled phase'
else
  RC=$?
  printf 'new SSH failed rc=%s; record timeout versus key/host/auth error\n' "$RC"
fi
```

enabled와 recovered는 exit0을 요구하고 disabled는 새 transport 실패를 요구한다. `Permission denied`, host-key mismatch 또는 passphrase 대기 timeout은 IGW deny가 아니다. 최초 positive에서 passphrase/host-key 신뢰를 준비하되 StrictHostKeyChecking을 끄거나 private key를 원격으로 보내지 않는다.

## 결과검증

<a id="baseline-29-ready"></a>

완료 단계 **IGW-tested**: enabled 성공·disabled 신규 연결 실패·enabled 복구 성공을 기록한다. private VM의 no-public-IP도 증거로 남긴다. `ping` 실패만으로 IGW 실패를 판정하지 않는다. HTTP3/IPv6/proxy를 배제한 TCP/IPv4 HTTPS 사용이 핵심이다.

**실행 기록(아직 미작성):** 실행자/Region/시각, exact resource OCID, 설정 전·후, 성공 출력, deny 오류·exit code/request-id, positive control, 정리 결과. 문서 검증은 실제 OCI 실행 성공을 뜻하지 않는다.

### 이 시나리오의 functional acceptance

| Positive | Negative / 미완료 판정 | Recovery / 반환 |
|---|---|---|
| enabled 신규 SSH/HTTPS | disabled same-target 신규 transport failure | bounded Console Enable/readback→신규 SSH/HTTPS |

모두 실행 시 확인할 기대 패턴이다. 생성 전 guard negative는 OCI 장애를 주입한 것과 다르며 다른 자원을 추가하지 않는다. 오류 메시지와 실제 exit code가 없는 항목은 NOT_EXECUTED 또는 BLOCKED로 남긴다.

## 문제해결

Enabled인데 실패하면 public IP/route target/table association → egress443/OS firewall → DNS/TLS 순으로 확인한다. Gateway disable로 SSH가 끊기면 Console Enable로 복구한다. Oracle endpoint가 301/403을 반환해도 TCP/TLS 연결 도달과 애플리케이션 응답이 확인되면 network success이며 상태코드를 미리200으로 가정하지 않는다.

## 리소스정리

Enable 상태와 원래 routes를 복원한다. 후속 NAT/SGW/VNIC/Functions가 public 경로를 사용하므로 IGW를 보존한다. 전체 종료 시 owner23이 IGW를 target으로 하는 모든 route를 먼저 제거한 후 exact IGW를 삭제한다. owner29는 공유 IGW 삭제 owner가 아니다. 실습용 reserved public IP를 별도 만든 경우 VM terminate 후에도 남는지 조회하고 삭제한다.


## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 아래 본문은 각 생성 필드·설정·검증·정리의 기술 근거이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [30] [Networking Overview — Internet Gateway](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/overview.htm)
  - 확인 범위: INTERNET GATEWAY.

### 실습 작업을 지원하는 추가 공식 본문

- [실습근거 1] [Internet Gateway](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/managingIGs.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
- [실습근거 2] [VCN Route Tables](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/managingroutetables.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
