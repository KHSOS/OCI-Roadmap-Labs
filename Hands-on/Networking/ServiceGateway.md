# Service Gateway: private VM에서 Object Storage 파일 읽기

**Service Gateway**는 같은 Region의 지원 Oracle 서비스로 가는 경로입니다. Object Storage endpoint가 public IP로 DNS 조회되더라도 트래픽은 Oracle 네트워크 안에서 전달될 수 있습니다. 이번에는 private VM에서 작은 private object를 읽고, **경로가 없을 때의 실패**와 **접근 자격이 없을 때의 HTTP 오류**를 따로 확인합니다.

[Networking 목록](README.md) · [경로 준비](#2-service-gateway와-service-route-만들기) · [실제 다운로드](#4-private-vm에서-파일-읽기) · [정리](#리소스정리)

## 학습목표

- 같은 Region의 Object Storage를 선택한 Service Gateway와 service route/egress443을 생성합니다.
- 인터넷 default route 없이 private VM에서 object 하나를 실제 내려받습니다.
- 같은 object·IP를 유지하며 unsigned 요청의 권한 실패와 SGW 차단의 연결 실패를 구분하고 복구합니다.

## 대응 Roadmap 항목

**Service Gateway** — Virtual Cloud Network (VCN)의 Oracle Services Network 접근. Object Storage 읽기는 gateway 기능을 확인하기 위한 작은 실제 작업입니다. SGW는 Object Storage IAM 권한을 부여하지 않습니다.

## 사전준비

1. [NAT Gateway 1–6단계](NATGateway.md#실습절차)를 완료하여 **같은 Region/Compartment의** public admin VM, `lab-private-vm`, `lab-private`, `rt-private`, `sl-private`, PC SSH config를 남깁니다. NAT는 Allow Traffic, private subnet에는 NAT default route와 egress443이 있는 상태입니다. 이번에는 새 VM을 만들지 않습니다. NAT 문서가 private subnet/VM과 ProxyJump를 처음부터 설명하므로 미검증 보존 초안을 따라갈 필요가 없습니다.
2. [Object Storage 시작하기 1–4단계](../ObjectStorage/GettingStarted.md#실습절차)를 **위 VCN과 같은 Region**에서 진행합니다. 새 private Standard bucket과 `lab/payload.txt`를 남기고 PC 원본/다운로드 내용이 `OCI Object Storage getting started`인지 확인합니다. Console 다운로드는 PC 인터넷 경로의 확인일 뿐 SGW 확인은 아닙니다.

두 안내의 정리는 아직 실행하지 않습니다. OS credential/API key를 VM에 설치하지 않습니다. 이 실습은 **특정 object의 read-only Pre-Authenticated Request(PAR)**로 접근 자격을 전달합니다. PAR 생성은 아래에서 새로 배웁니다. private bucket을 public으로 바꾸지 않습니다.

## 권한

선택 Compartment의 Service Gateway 생성/조회/traffic 변경/삭제, service 목록 조회, route table과 private security list 수정, 기존 VCN 연결이 필요합니다. [Core Services IAM reference](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm)의 CreateServiceGateway·UpdateRouteTable·UpdateSecurityList 의존성을 확인합니다.

Bucket/object 생성·업로드·읽기·삭제는 Object Storage 시작하기와 같습니다. 추가로 **PAR_MANAGE와 OBJECT_READ**가 필요합니다. [PAR 권한](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/usingpreauthenticatedrequests.htm)에서는 PAR creator의 접근 권한을 사용할 때마다 검사한다고 설명합니다. 제한 계정의 편의 Compartment 범위는 `manage virtual-network-family`와 `manage object-family`이며 최소권한 정책은 아닙니다. 개인 관리자 계정의 기존 권한이 충분하면 새 policy/dynamic group을 만들지 않습니다. 권한 오류를 PAR나 public 전환으로 우회하지 않습니다.

## 비용및주의사항

기존 VM 두 대/boot volume 두 개를 재사용하며 bucket의 작은 object 하나, PAR 하나, SGW 하나만 사용합니다. VM·boot·공인 IPv4·object 저장/요청/조건별 전송과 현재 [Networking 요금](https://www.oracle.com/cloud/networking/pricing/), [Object Storage 요금](https://www.oracle.com/cloud/storage/pricing/)을 확인합니다. SGW 경로가 있다고 저장/요청 요금이 모두 사라지는 것은 아닙니다.

PAR URL은 **가진 사람이 해당 object를 읽을 수 있는 비밀**입니다. read-only·특정 object·짧은 만료(현재 시각에서 약 1시간)로 제한하고 실습 후 즉시 삭제합니다. URL을 Git/채팅/스크린샷/공유 history에 남기지 않습니다. VM 두 대의 실행 시간도 늘어나므로 한 세션에서 마칩니다.

## 실습구성

`lab-private-vm → rt-private의 service destination → sgw-lab → 같은 Region Object Storage` 경로를 만듭니다. private subnet의 NAT default route를 잠시 **제거하여 fallback을 없앤 뒤** SGW로 파일을 읽습니다. admin→private SSH는 VCN 내부 경로라 영향받지 않습니다. public subnet의 IGW route는 변경하지 않습니다.

| 변경 | 최종 시험 상태 | 이유 |
|---|---|---|
| 신규 `sgw-lab` | `OCI <현재 Region> Object Storage`, traffic allowed | 이번 파일 서비스만 선택 |
| `rt-private` | Service destination → `sgw-lab`, NAT default route 없음 | 인터넷 경로로 성공하는 오판 방지 |
| `sl-private` | admin/32 ingress22 + service destination egress443 | 일반 인터넷443 허용 대신 서비스 범위 |
| 신규 PAR | `lab/payload.txt`만 read-only | VM에 API key 없이 private object 읽기 |

Service CIDR label은 실제 서비스 IP 범위를 대표하는 선택 항목입니다. 고정 숫자 CIDR을 직접 추정하지 않습니다. SGW는 Region 단위이며 다른 Region object endpoint나 일반 인터넷 사이트의 출구가 아닙니다.

## 실습절차

### 1. Object 하나의 read-only PAR 만들기 — Console

**Storage → Object Storage & Archive Storage → Buckets → 이번 bucket → Management → Pre-authenticated requests → Create pre-authenticated request**를 선택합니다. Target에서 Object를 선택하고 아래 이름을 입력합니다.

| 필드 | 값 | 이유 |
|---|---|---|
| Name | `sgw-read-one` | 정리할 PAR 식별 |
| Pre-Authenticated Request Target | Object / `lab/payload.txt` | bucket 전체·prefix에 접근시키지 않음 |
| Access Type | Permit object reads | 쓰기와 목록은 불필요 |
| Expiration | 현재 시각 약 1시간 뒤를 UTC로 입력(기본 1주는 변경) | 이번 세션으로 제한 |

Create 후 **전체 URL**을 개인적으로 임시 보관합니다. URL은 생성 시에만 표시되므로 창을 닫기 전에 복사하고, PAR 목록에서 target·read-only·만료를 확인합니다. URL을 잃으면 복구할 수 없으므로 이번 PAR를 삭제하고 한 번 새로 만듭니다. 아래에서 PC 원본과 같은 object를 읽을 예정입니다. 이 URL을 단순 일반 Object URL과 구분하세요.

### 2. Service Gateway와 service route 만들기

**Console → Networking → Virtual cloud networks → lab-vcn → Gateways → Service Gateways → Create Service Gateway**에서 Name=`sgw-lab`, 같은 Compartment, Services=`OCI <현재 Region> Object Storage`를 선택합니다. Region 표시 약어는 화면의 현재 Region 선택을 따릅니다. `All <region> Services in Oracle Services Network`까지 열 필요는 없습니다. Route Table Association은 비워 둡니다(이번에는 transit routing이 아님).

Create 후 gateway `AVAILABLE`, 선택 서비스와 **traffic allowed**를 확인합니다. service를 선택하지 않으면 traffic이 흐르지 않습니다. 같은 VCN에 SGW가 이미 있다면 중복 생성하지 말고 이번 개인 실습 자원인지 확인합니다.

**같은 VCN → Routing → Route Tables → rt-private → Add Route Rules**에서 다음을 저장합니다.

- Target Type=`Service Gateway`
- Destination Type=`Service`, Destination Service=`OCI <현재 Region> Object Storage`
- Target Compartment=이번 Compartment, Target=`sgw-lab`

Gateway의 서비스와 route의 서비스를 동일하게 선택합니다. Object Storage와 All Services 두 service route를 동시에 만들지 않습니다. 저장 후 `lab-private` 상세의 실제 연결 route table이 `rt-private`인지 다시 확인합니다.

### 3. 인터넷 fallback 제거하기

원복할 **`rt-private`의 `0.0.0.0/0 → nat-lab`**과 **`sl-private`의 CIDR `0.0.0.0/0` TCP443 egress**를 메모합니다. NAT gateway 자체는 삭제/차단하지 않습니다.

1. **sl-private → Add Egress Rules**에서 Stateless 해제, Destination Type=`Service`, Destination Service=`OCI <현재 Region> Object Storage`, Protocol TCP, source port All, destination port `443`을 추가합니다. 저장된 service label을 확인합니다.
2. 기존 `0.0.0.0/0` TCP443 egress 행만 제거합니다. admin/32 ingress22는 유지합니다. service443 한 행만 남는지 읽습니다.
3. **rt-private**에서 기존 NAT default route만 제거합니다. service destination → `sgw-lab` 한 규칙만 남는지 읽습니다.

이제 private VM은 Object Storage에는 갈 수 있지만 임의 인터넷 HTTPS를 사용할 경로가 없습니다. public admin VM의 IGW route와 private SSH 규칙을 바꾸지 않습니다. 오류가 나면 변경 상태를 확인하고 [정리 2번](#리소스정리)의 원복으로 돌아갑니다. SGW가 NAT 전체 기능을 대신한다고 생각하여 패키지 업데이트 성공까지 요구하지 않습니다.

### 4. Private VM에서 파일 읽기

**Linux/WSL PC**에서 NAT 4단계에 메모한 같은 config 절대경로로 `NETWORKING_CONFIG`를 설정하고 두 HostName/키를 확인한 뒤 `ssh -F "${NETWORKING_CONFIG:?NAT 4단계의 config 절대경로를 먼저 설정하세요}" networking-private`로 로그인합니다. 이후 모든 curl은 **private VM**에서 실행합니다. PAR를 명령에 직접 붙이지 않고 아래 숨김 입력으로 넣습니다. 쉘 tracing(`set -x`)은 끄고 URL을 출력하지 않습니다. 전용 개인 VM에서만 사용합니다. curl process 인자에 URL이 잠시 노출될 수 있으므로 공유 서버용 credential 전달 방식은 아닙니다.

```bash
set +x
read -r -s -p 'Console에서 복사한 전체 PAR URL: ' PAR_URL
printf '\n'
case "$PAR_URL" in
  https://*/p/*/n/*/b/*/o/*) ;;
  *) printf '%s\n' 'STOP: Object PAR 전체 URL이 필요합니다'; exit 1 ;;
esac
OS_HOST=${PAR_URL#https://}
OS_HOST=${OS_HOST%%/*}
OS_IP=$(getent ahostsv4 "$OS_HOST" | cut -d ' ' -f 1 | sort -u | sed -n '1p')
if test -z "$OS_IP"; then printf '%s\n' 'DNS 실패'; exit 1; fi
LAB_DIR=$(mktemp -d "$HOME/sgw-lab.XXXXXX") || exit 1
printf 'endpoint=%s fixed_ip=%s directory=%s\n' "$OS_HOST" "$OS_IP" "$LAB_DIR"
get_object() {
  curl -4 --noproxy '*' --resolve "$OS_HOST:443:$OS_IP" \
    --connect-timeout 5 --max-time 15 -H 'Connection: close' \
    -sS -o "$LAB_DIR/payload.txt" \
    -w 'http=%{http_code} connect=%{time_connect} tls=%{time_appconnect}\n' \
    "$PAR_URL"
  RC=$?
  printf 'curl_exit=%s\n' "$RC"
}
get_object
```

Printed endpoint가 **현재 Region의 regional Object Storage endpoint**인지 Console의 bucket Region/PAR URL과 비교합니다. 다른 Region URL이면 진행하지 않습니다. Standard commercial Region 기준 `objectstorage.<region-identifier>.oraclecloud.com` 형식이며 다른 realm의 domain은 해당 Console URL을 따릅니다. DNS가 public IPv4를 반환하는 것은 정상이며 SGW를 사용하지 않는다는 증거가 아닙니다.

기대는 `curl_exit=0`, HTTP200, TCP/TLS 연결 시간 0 초과입니다. 이 조건을 확인한 **뒤에만** `cat "$LAB_DIR/payload.txt"`로 내용을 읽습니다. PC 원본과 같은 `OCI Object Storage getting started` 한 줄이면 실제 object 읽기를 마쳤습니다. non-200 응답 파일은 object가 아니라 오류일 수 있습니다. 실패할 때도 이전 다운로드 파일을 새 성공으로 읽지 않습니다.

### 5. 접근 자격이 없을 때와 경로가 없을 때 비교하기

**먼저 접근 자격만 제거한 요청**을 같은 private VM에서 실행합니다. `/p/<token>` 부분을 빼고 같은 namespace/bucket/object와 endpoint/IP를 유지합니다. Bucket은 여전히 Private이고 PAR 자체는 삭제하지 않습니다.

```bash
OBJECT_URL="https://$OS_HOST/n/${PAR_URL#*/n/}"
curl -4 --noproxy '*' --resolve "$OS_HOST:443:$OS_IP" \
  --connect-timeout 5 --max-time 15 -H 'Connection: close' \
  -sS -o "$LAB_DIR/unsigned-error.json" \
  -w 'unsigned_http=%{http_code} connect=%{time_connect} tls=%{time_appconnect}\n' \
  "$OBJECT_URL"
RC=$?
printf 'curl_exit=%s\n' "$RC"
```

예상은 HTTP401/403/404와 TCP/TLS 연결 성공입니다. Object Storage는 권한 없는 자원에 not-found 계열 응답을 할 수 있습니다. 이 curl에는 `--fail`을 쓰지 않아 **HTTP 오류에도 exit0일 수 있습니다**. HTTP 코드와 `cat "$LAB_DIR/unsigned-error.json"`의 서비스 오류를 함께 읽습니다. 이는 서비스에 도착했으나 읽을 자격이 없다는 결과이지 SGW routing 실패가 아닙니다. HTTP200이면 bucket visibility/URL에 token이 남았는지 재확인합니다.

이어 **Console → sgw-lab → Block Traffic**을 실행하고 **같은 `get_object`**를 다시 호출합니다. NAT default route가 없고 service route만 남았으므로 이번에는 HTTP000, TCP 시간0과 신규 연결 실패(흔히 exit28)를 기대합니다. HTTP 오류가 돌아온 경우에는 이미 서비스에 도달했으므로 routing deny로 해석하지 않습니다.

**어떤 결과여도 곧바로 Console → sgw-lab → Allow Traffic**로 복구합니다. 같은 PAR·object·고정 IP로 `get_object`를 다시 실행하여 HTTP200와 원본 내용이 돌아오는지 확인합니다. PAR 만료/삭제, object 변경, DNS IP 변경을 섞으면 경로 비교가 성립하지 않으므로 그런 경우 먼저 현재 자격을 확인하고 positive부터 다시 시작합니다. 터미널이 끊겨도 Console traffic부터 허용하여 복구할 수 있습니다.

## 결과검증

- private VNIC public IP 없음, route는 regional Object Storage → SGW만 있으며 NAT/IGW 인터넷 default route 없음.
- PAR GET은 HTTP200와 원본 내용, unsigned GET은 연결된 HTTP 권한/not-found 오류.
- SGW block 중 같은 PAR GET은 신규 TCP 연결 실패, allow 후 같은 요청은 HTTP200.

이 세 조건을 같이 확인해야 경로와 접근 자격을 구분할 수 있습니다. endpoint의 public DNS IP, Console 다운로드, HTTP401 하나만으로 SGW 경로의 전체 성공을 주장하지 않습니다. 여기의 출력은 실행 시 기대값이며 OCI에서 관측한 결과가 아닙니다.

## 문제해결

| 증상 | 확인과 복구 |
|---|---|
| PAR GET이 401/403/404 | token 전체, exact object name, 만료 시간대, PAR creator의 현재 권한 확인. service에 도착한 응답이면 routing부터 바꾸지 않음 |
| TCP timeout | 같은 Region 서비스 선택, SGW Allow, subnet→rt-private 연결, service route와 service443 egress 확인 |
| DNS 실패 | default DHCP/resolver 확인. Object Storage IP를 임의 CIDR로 열지 않음 |
| 인터넷 사이트가 실패 | NAT route/일반443을 제거한 시험 상태에서는 정상. SGW는 임의 인터넷 출구가 아님 |
| block 중 HTTP 응답 | 요청 host·proxy·실제 route·gateway 상태 확인. 같은 endpoint/IP/자격의 positive를 다시 확보 |
| PAR 창을 닫아 URL 분실 | URL은 재조회 불가. 이번 PAR 삭제 후 read-only object PAR를 새로 생성 |
| bucket 삭제가 거부됨 | 먼저 이번 PAR 삭제. object/version/다른 의존성을 확인하고 모르는 자원은 삭제하지 않음 |

## 리소스정리

이미 정리한 항목은 정상 목록 조회로 없음을 확인하고 해당 동작을 건너뜁니다. 삭제한 SGW/PAR를 선택하거나 새로 만들지 않습니다. NSG로 이어가도 1–4번에서 PAR·SGW·object·bucket까지 정리하며 VM/NAT만 남깁니다.

1. SGW가 남아 있으면 traffic을 **Allow**로 복구합니다. **bucket → Pre-Authenticated Requests → sgw-read-one → Delete**로 접근 자격을 폐기하고 목록에서 사라졌는지 확인합니다. private VM에서 `unset PAR_URL OBJECT_URL`을 실행하고 `LAB_DIR`로 출력된 이번 다운로드 디렉터리만 필요 없으면 삭제합니다. URL 임시 사본도 정리합니다.
2. 다음 NAT/NSG 실습을 계속한다면 **rt-private → Add Route Rules**에서 CIDR `0.0.0.0/0 → NAT Gateway nat-lab`, **sl-private → Add Egress Rules**에서 stateful CIDR `0.0.0.0/0`, TCP source All/destination443을 복원합니다. NAT Allow 상태를 확인합니다. 이번 service route와 service443 행만 제거하고, subnet의 route/security 연결을 다시 읽습니다. NAT 문서의 5단계 요청으로 인터넷 경로를 다시 확인합니다. 바로 전체 삭제한다면 NAT/443 복원은 불필요하고 service 두 규칙만 제거합니다.
3. SGW로 향하는 route가 없음을 확인하여 **Service Gateways → sgw-lab 행 Actions → Terminate**를 실행하고 목록을 정상 새로고침합니다.
4. [Object Storage 정리](../ObjectStorage/GettingStarted.md#리소스정리)로 이번 object→bucket 순서로 삭제합니다. 다른 object가 생겼다면 먼저 대상과 필요성을 확인합니다.
5. [NSG 실습](NetworkSecurityGroups.md)을 이어갈 때 VM 두 대와 private SSH를 유지합니다. 끝낸다면 [NAT 정리](NATGateway.md#리소스정리)의 private VM/boot → NAT/private subnet → public VM/boot → VCN 순서를 따릅니다. Compartment는 삭제하지 않습니다.

## 공식문서

- [Controlling Service Gateway Traffic](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/sgw-traffic.htm)
- [Deleting a Service Gateway](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/delete-sgw.htm)

- [Service Gateway: service label, routing, security와 Region 범위](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/servicegateway.htm)
- [Creating a Service Gateway](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/create-sgw.htm)
- [Pre-Authenticated Requests: 권한, URL와 폐기](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/usingpreauthenticatedrequests.htm)
- [Creating a Pre-Authenticated Request: Console 필드와 object 접근](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/usingpreauthenticatedrequests_topic-To_create_a_preauthenticated_request_for_an_object.htm)
- [Core Services IAM reference](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm)
