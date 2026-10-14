# NAT Gateway: private VM에서 HTTPS 요청 보내기

공인 IP 없는 VM도 패키지 저장소나 외부 API에 요청을 보내야 할 때가 있습니다. **NAT Gateway**는 VM이 시작한 인터넷 연결과 그 응답을 전달합니다. 인터넷 사용자가 이 gateway를 통해 VM으로 새 연결을 시작할 수는 없습니다. 이번에는 private VM에서 실제 HTTPS를 요청하고, NAT의 traffic을 잠시 막았다가 복구합니다.

[Networking 목록](README.md) · [Private VM 준비](#3-private-vm-한-대-만들기) · [HTTPS 시험](#5-private-vm에서-https-요청하기) · [정리](#리소스정리)

## 학습목표

- private subnet과 공인 IP 없는 Oracle Linux VM을 만들고 기존 public VM을 거쳐 SSH로 접속합니다.
- NAT Gateway와 subnet route를 연결하여 private VM에서 TLS 검증을 통과한 HTTPS 응답을 받습니다.
- 같은 목적지로 NAT 차단 전·중·복구 후 요청을 비교하고 이번 자원을 정리합니다.

## 대응 Roadmap 항목

**NAT Gateway** — Virtual Cloud Network (VCN)의 인터넷 outbound 경로. Private Subnets, Route Tables, stateful egress를 실제 설정 과정에서 함께 사용합니다.

## 사전준비

개인 계정의 새로운 실습 네트워크를 사용합니다. 아래 두 안내를 순서대로 완료하되 **정리는 아직 하지 않습니다**.

1. [VCN 시작하기 1–5단계](VCN/GettingStarted.md#실습절차): `lab-vcn=10.77.0.0/16`, `lab-public=10.77.10.0/24`, `rt-public`, `sl-public`, `igw-lab`을 남깁니다.
2. [Compute 시작하기 1–4단계](../Compute/GettingStarted.md#실습절차): SSH를 선택하여 public `lab-vm` 한 대를 생성하고 PC에서 `opc` 로그인까지 확인합니다. 이 VM을 **admin VM**이라고 부릅니다. 5단계 외부 HTTPS는 필수가 아닙니다. Console-only로 만든 VM은 이 경로의 SSH client로 사용할 수 없습니다.

같은 Region·Compartment를 계속 사용합니다. admin VM의 **public IPv4와 primary VNIC private IPv4**, PC의 private key와 대응하는 공개키 파일을 준비합니다. 공개키를 저장하지 않았다면 Linux/WSL PC에서 `ssh-keygen -y -f "$KEY"`로 공개키를 확인하여 아래 VM 생성 화면에 붙여넣을 수 있습니다. private key 자체는 붙여넣지 않습니다. `KEY` 설정과 WSL 키 복사는 Compute 4단계를 따릅니다.

이 문서는 **Linux/WSL PC Bash + Oracle Linux 9 guest** 기준입니다. PC 명령, public admin VM 명령, private VM 명령을 구분하세요. Cloud Shell은 기본적으로 이 VCN의 private IP에 접근하는 client가 아니므로 private VM 시험을 Cloud Shell에서 대신하지 않습니다. 생성 순서는 VCN → public VM SSH → private subnet/VM → private SSH → NAT이며 SSH가 NAT에 의존하지 않습니다.

## 권한

선택 Compartment에서 NAT Gateway·route table·security list·subnet의 생성/조회/수정/삭제와 기존 VCN 연결, instance 생성/조회/종료, image 조회, VNIC/subnet 연결, boot volume 조회/삭제가 필요합니다. Gateway의 traffic 차단/허용은 UpdateNatGateway 작업입니다.

개인 tenancy의 기존 관리자 계정이라면 별도 policy를 만들지 않습니다. 제한된 계정에서 필요한 범위를 점검할 때 `manage virtual-network-family`, `manage instance-family`, `manage volume-family`와 `read instance-images`의 Compartment 범위가 편의 기준입니다. 이는 **최소권한 policy가 아니며**, 직접 권한을 확대하는 단계도 아닙니다. [Core Services IAM reference](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm)는 CreateNatGateway의 VCN 권한, route의 NAT attach, LaunchInstance의 image/VNIC/subnet 의존성까지 설명합니다. 권한 오류면 대상 Region·Compartment와 계정 권한을 확인한 뒤 재시도합니다.

## 비용및주의사항

동시에 **VM 두 대와 각 boot volume 하나**만 사용합니다. 신규 private VM은 `VM.Standard.A1.Flex` 1 OCPU / 6 GB 후보이며 admin VM과 합산한 무료 한도·capacity를 확인합니다. 작은 shape를 선택할 수 없으면 큰 유료 VM으로 자동 대체하지 않습니다. NAT/public IPv4/인터넷 전송의 현재 요금은 [OCI Networking pricing](https://www.oracle.com/cloud/networking/pricing/)에서 확인합니다. 무료라는 가정으로 진행하지 않습니다. 추가 data disk, logging, backup, LB는 만들지 않습니다.

NAT 공인 IP는 gateway의 주소이지 private VM의 주소가 아닙니다. private VM에는 inbound 인터넷 접속을 열지 않습니다. 키를 admin VM에 복사하지 않고 PC에서 ProxyJump를 사용합니다. 후속 실습을 하지 않으면 이번 세션에서 VM과 boot volume까지 삭제합니다.

## 실습구성

| 기존 자원 | 이번에 추가할 자원 | 역할 |
|---|---|---|
| `lab-vcn`, `lab-public`, `lab-vm`, IGW | `lab-private` (`10.77.20.0/24`) | VM에 공인 IPv4를 할당하지 않는 subnet |
| public VM의 `sl-public` | `sl-private`, `rt-private` | private SSH와 outbound443만 허용, NAT 목적지 선택 |
| PC의 같은 key pair | `lab-private-vm`, boot volume | 실제 HTTPS client |
| 기존 public 인터넷 경로 | `nat-lab` | private VM에서 시작한 인터넷 연결의 출구 |

VCN 내부 subnet 간 경로는 기본적으로 존재합니다. public→private SSH에는 두 VM의 security rules가 필요하지만 NAT/IGW를 경유하지 않습니다. NAT를 차단해도 관리 SSH를 계속 사용할 수 있어 복구가 가능합니다.

## 실습절차

### 1. Private subnet의 규칙과 경로표 만들기 — Console

**Networking → Virtual cloud networks → 이번 lab-vcn → Security → Security Lists → Create Security List**에서 `sl-private`를 만듭니다(Resources 메뉴로 표시될 수도 있습니다). 자동 입력된 모든 행을 제거한 뒤 다음 두 규칙만 넣습니다.

| 방향 | Type / CIDR | Protocol | Source port | Destination port | Stateful 선택 이유 |
|---|---|---|---|---|---|
| Ingress | CIDR / `admin VM 실제 private IPv4/32` | TCP | All | 22 | admin VM이 시작한 SSH와 그 응답 |
| Egress | CIDR / `0.0.0.0/0` | TCP | All | 443 | private VM이 시작한 HTTPS와 그 응답 |

각 행의 **Stateless는 해제**합니다. `/32`에는 Console → admin VM → Networking → primary VNIC의 **private** IPv4를 사용합니다. PC 주소나 admin 공인 IP가 아닙니다. Egress443은 internet HTTPS 허용이며 ingress443은 추가하지 않습니다. VCN 기본 resolver 사용에는 여기서 별도 외부 DNS 서버 규칙을 추가하지 않습니다.

**같은 VCN → Routing → Route Tables → Create Route Table**에서 `rt-private`, 같은 Compartment, route 없음으로 생성합니다. 양쪽 상세에서 규칙과 빈 경로표를 다시 확인합니다. 넓은 default security list를 사용하는 방식으로 대체하지 않습니다.

### 2. Private subnet 만들기 — Console

**같은 VCN → Subnets → Create subnet**을 엽니다.

| 필드 | 값 | 이유 |
|---|---|---|
| Name / Compartment | `lab-private` / 기존과 같은 Compartment | 이번 private 네트워크 식별 |
| Subnet Type | Regional | AD 하나에 제한하지 않음 |
| IPv4 CIDR | `10.77.20.0/24` | 기존 public 범위와 겹치지 않는 VCN 내부 범위 |
| Subnet Access | Private subnet | VNIC 공인 IPv4 할당 금지 |
| Route Table | `rt-private` | public IGW 경로와 분리 |
| Security Lists | `sl-private` 하나만 | 기본 all-egress와 중복 허용을 배제 |
| DHCP options | 이 VCN의 default | Internet and VCN Resolver 사용 |
| DNS hostnames / label | 사용 / `labprivate` | 기본 DNS 기능 사용 |

Default security list는 선택 해제하고 IPv6·logging은 추가하지 않습니다. Create 후 `AVAILABLE`, Private, Regional, CIDR과 route/security 연결을 확인합니다. 생성 오류면 목록을 확인하고 잘못된 필드만 수정하여 중복 생성을 피합니다.

### 3. Private VM 한 대 만들기

**Compute → Instances → Create instance**에서 아래를 선택합니다. [Compute 생성 화면 설명](../Compute/GettingStarted.md#2-vm-한-대-만들기--console)과 같은 OS/shape 방식이며 **subnet과 공인 IP 선택은 아래 값**을 사용합니다.

| 필드 | 값 | 이유 |
|---|---|---|
| Name / Compartment | `lab-private-vm` / 기존과 동일 | public admin VM과 구분 |
| Placement / capacity | 가용 AD, FD 기본 / On-demand | 별도 예약 자원 없음 |
| Image / Shape | Oracle Linux 9 Arm-compatible / A1.Flex, 1 OCPU, 6 GB | 작은 Linux client |
| VCN / Subnet | 기존 `lab-vcn` / `lab-private` | 이번 private subnet에 primary VNIC 생성 |
| Public IPv4 | 할당하지 않음 | private subnet에서 금지되어야 함 |
| SSH keys | Paste public keys 또는 Upload public key files / PC의 같은 공개키 | PC의 기존 private key로 인증 |
| Boot volume | image 기본 크기·기본 성능, Oracle-managed encryption | OS 디스크만 생성 |

No SSH keys나 새 key pair 생성을 선택하지 않습니다. 초기화 script·NSG·추가 volume·backup은 추가하지 않습니다. Review에서 값을 읽고 Create를 한 번 누릅니다. `RUNNING` 뒤 primary VNIC의 private IP와 **public IP 없음**, 연결 boot volume의 이름/AD를 메모합니다. 생성 실패면 Work Requests와 현재 목록을 먼저 읽습니다.

**Console → lab-vcn → sl-public → Add Egress Rules**에서 stateful, CIDR destination=`이번 private VM 실제 private IPv4/32`, TCP, source port All, destination port `22` 규칙 하나를 추가합니다. 추가 전 같은 규칙이 있는지 확인하여 중복하지 않습니다. 개인 메모에 `이번 추가` 또는 `이미 존재`와 해당 행의 rule ID(표시되는 경우)/목적지/port를 남깁니다. 이미 존재한 행은 이번 실습의 삭제 대상이 아닙니다. description=`networking-private-ssh`로 구분합니다. admin ingress22는 기존 PC/32 규칙을 유지합니다. private ingress22만으로 admin의 신규 outbound SSH를 허용하지는 못합니다.

### 4. PC에서 private VM으로 접속하기

**Linux/WSL PC Bash**에서 이번 세션의 전용 SSH config 경로를 정하고 빈 파일을 먼저 확보합니다. 아래 블록은 새 파일을 만들 때 **한 번만** 실행합니다. 기존 파일이나 symlink가 있으면 중단하며 덮어쓰거나 그 파일로 접속하지 않습니다. 충돌을 해결하기 전에는 다음으로 진행하지 않습니다.

```bash
export NETWORKING_CONFIG="$HOME/oci-networking.conf"
if test -e "$NETWORKING_CONFIG" || test -L "$NETWORKING_CONFIG"; then
  printf '%s\n' 'STOP: 전용 config 경로가 이미 있습니다. 기존 파일을 보존하고 대상을 확인하세요.'
  exit 1
fi
(umask 077; set -C; : > "$NETWORKING_CONFIG") || exit 1
```

`NETWORKING_CONFIG`의 실제 절대경로를 개인 메모에 남깁니다. NAT·SGW·NSG의 모든 PC SSH 명령은 이 변수만 사용합니다. 새 PC 터미널에서는 메모의 **같은 절대경로**로 `export NETWORKING_CONFIG='<이번 config의 실제 절대경로>'`를 다시 설정하고, 저장된 두 HostName·IdentityFile을 이번 VM/키와 대조한 뒤 접속합니다. 새 터미널에서 위 파일 생성 블록을 다시 실행하지 않습니다. 아래 `<...>` 값을 실제 값으로 바꿉니다. 두 VM은 같은 공개키를 등록했으며 private key는 PC에만 있습니다.

```sshconfig
Host networking-admin
  HostName <admin VM public IPv4>
  User opc
  IdentityFile <PC private key의 Linux 절대경로>
  IdentitiesOnly yes
  ConnectTimeout 10
  ConnectionAttempts 1
Host networking-private
  HostName <private VM private IPv4>
  User opc
  IdentityFile <PC의 같은 private key Linux 절대경로>
  IdentitiesOnly yes
  ProxyJump networking-admin
  ConnectTimeout 10
  ConnectionAttempts 1
```

PC의 텍스트 편집기로 `NETWORKING_CONFIG`에 지정한 이번 파일에 위 내용을 저장한 뒤 실행합니다. 경로에 공백이 있으면 IdentityFile 경로를 큰따옴표로 감쌉니다.

```bash
chmod 600 "${NETWORKING_CONFIG:?NAT 4단계의 config 절대경로를 먼저 설정하세요}"
ssh -F "${NETWORKING_CONFIG:?NAT 4단계의 config 절대경로를 먼저 설정하세요}" networking-private
```

첫 접속은 private VM의 host key도 신뢰할지 묻습니다. Console의 private IP와 접속 대상이 같은지 확인하고, 별도 신뢰할 fingerprint가 있다면 비교한 뒤 확실할 때만 `yes`를 입력합니다. 이는 최초 신뢰이며 독립 신원 검증은 아닙니다. host key 변경 경고는 우회하지 않습니다.

로그인된 **private VM**에서 `id -un`, `cat /etc/os-release`, `command -v curl`, `getent ahostsv4 www.oracle.com`을 실행합니다. `opc`, Oracle Linux 9, curl 경로와 IPv4 DNS 결과를 기대합니다. 도구가 없으면 NAT 연결 후 설치가 필요한 별도 분기로 취급하고 여기서는 진행을 멈춥니다. SSH timeout이면 admin egress private/32:22와 private ingress admin/32:22를 먼저 확인합니다. 인터넷 접속은 아직 기대하지 않습니다.

### Private SSH baseline 준비 완료

4단계의 private SSH와 OS/tool 확인까지 끝나면 NSG의 선행 준비가 완료됩니다. [NSG 실습](NetworkSecurityGroups.md)으로 바로 이동할 수 있습니다. NAT 생성·HTTPS·차단 시험은 NSG의 필수 조건이 아니며 Python 설치에 outbound가 필요할 때만 아래 5단계로 인터넷 경로를 준비합니다. 6단계 차단 시험은 설치의 필수 조건이 아닙니다. 자원은 아직 정리하지 않습니다.

### 5. Private VM에서 HTTPS 요청하기

**Console → lab-vcn → Gateways → NAT Gateways → Create NAT Gateway**에서 Name=`nat-lab`, 같은 Compartment, **Ephemeral IP Address**를 선택합니다. 별도 reserved IP를 만들지 않고 Route Table Association도 비웁니다(advanced gateway ingress routing이 아님). Create 후 `AVAILABLE`, traffic allowed와 NAT 공인 IP를 확인합니다.

**lab-vcn → Routing → Route Tables → rt-private → Add Route Rules**에서 Target Type=`NAT Gateway`, Destination Type=`CIDR Block`, Destination=`0.0.0.0/0`, Target=`nat-lab`을 저장합니다. private subnet 상세에서 실제 연결 route table이 `rt-private`인지 다시 읽습니다. Gateway를 만들기만 해서는 subnet 경로가 생기지 않습니다.

앞서 접속한 **private VM Bash 터미널**에서 아래를 실행합니다. DNS 결과 중 IPv4 하나를 고정하여 이후 비교에서 목적지가 바뀌지 않게 합니다. `--resolve`는 TLS hostname을 유지하고 `--noproxy '*'`는 proxy 환경변수 경유를 막습니다.

```bash
HOST=www.oracle.com
IP=$(getent ahostsv4 "$HOST" | cut -d ' ' -f 1 | sort -u | sed -n '1p')
if test -z "$IP"; then printf '%s\n' 'DNS 결과가 없습니다'; exit 1; fi
printf '고정 목적지: %s %s\n' "$HOST" "$IP"
probe_https() {
  curl -4 --noproxy '*' --resolve "$HOST:443:$IP" \
    --connect-timeout 5 --max-time 15 -H 'Connection: close' \
    -sS -o /dev/null \
    -w 'http=%{http_code} connect=%{time_connect} tls=%{time_appconnect}\n' \
    "https://$HOST/"
  RC=$?
  printf 'curl_exit=%s\n' "$RC"
}
probe_https
```

예상은 `curl_exit=0`, HTTP 2xx/3xx, 0보다 큰 TCP/TLS 연결 시간입니다. redirect는 여기서 따라가지 않습니다. HTTPS 응답이면 private VM에서 NAT를 통한 실제 연결을 확인한 것입니다. HTTP403/429도 TLS/HTTP 서버에 도착한 것이지만 웹사이트 제한이므로 원하는 페이지 응답과 구분합니다. `-k`로 TLS 검증을 끄지 않습니다. hostname/IP/출력을 메모하고 이 터미널을 유지합니다.

### 6. 차단하고 같은 요청 복구하기

먼저 **Console → nat-lab → Block Traffic**에서 차단을 확인합니다. 화면에 Actions 메뉴로 나타날 수도 있습니다. SSH는 VCN 내부 경로이므로 유지됩니다. Private VM에서 **같은 `probe_https`**를 다시 호출합니다. 새 curl은 같은 IP/hostname을 쓰며 기존 연결을 재사용하지 않습니다.

차단 중 예상은 HTTP000, TCP 연결 시간 0, 흔히 exit28(connection timeout)입니다. HTTP 응답 또는 TLS 연결이 있었다면 차단/다른 경로 여부를 확인하며 실패를 억지로 NAT 차단 성공으로 해석하지 않습니다. DNS 실패·인증 오류·연결 후 timeout은 같은 의미가 아닙니다.

**시험 결과와 관계없이 곧바로 Console → nat-lab → Allow Traffic**로 복구합니다. Allow 상태를 읽고 같은 터미널에서 `probe_https`를 다시 실행하여 최초와 같은 HTTPS 성공을 확인합니다. 브라우저/터미널이 끊겨도 Console에서 traffic부터 허용한 후 재접속하여 5단계의 동일 IP를 다시 지정합니다. 복구 전 추가 설치·다른 gateway 생성은 하지 않습니다.

## 결과검증

완료 조건은 private VNIC에 public IP가 없고, `rt-private`에 NAT default route, `sl-private`에 위 두 규칙만 있으며, **private VM에서** HTTPS 성공 → NAT 차단 시 신규 TCP 연결 실패 → traffic 허용 후 HTTPS 성공입니다. Console의 `AVAILABLE`만으로 실제 통신 성공을 대신하지 않습니다. 위 출력은 실행할 때 대조할 예상이며 이 문서 작성 과정에서 OCI 자원을 실행한 결과는 아닙니다.

## 문제해결

| 증상 | 확인과 복구 |
|---|---|
| SSH timeout | PC→admin ingress22와 admin→private egress22, private ingress22를 각각 확인. public key와 guest sshd도 확인 |
| Permission denied (publickey) | private VM에 등록한 공개키와 PC private key, `opc` 사용자 일치 확인. private key를 admin에 복사하지 않음 |
| DNS 실패 | default DHCP/resolver와 DNS hostnames 설정 확인. NAT 차단 성공으로 취급하지 않음 |
| HTTPS timeout | private subnet의 실제 route table, NAT target/Allow Traffic, 모든 연결 security list/NSG, guest firewall 확인 |
| 차단 중 여전히 성공 | block 상태, 요청 실행 host, 다른 proxy/route와 고정 IP 확인. 복구 후 다시 positive control |
| TLS 실패 / 403 | OS clock·CA와 사이트 제한 확인. `curl -k`나 넓은 ingress 추가로 해결하지 않음 |
| Delete의 Resource in use | route와 subnet/VNIC 의존성을 먼저 제거. 이름 같은 다른 자원을 삭제하지 않음 |

## 리소스정리

**다음 실습으로 이어갈 때**: [Service Gateway](ServiceGateway.md)는 NAT/443을 남긴 상태에서 시작하고, [NSG](NetworkSecurityGroups.md)는 이 private SSH 준비를 재사용합니다. VM을 추가로 만들지 않습니다. 모든 실습이 끝나면 다음 순서로 한 번만 정리합니다. SGW/PAR 또는 NSG 앱을 만들었다면 그 문서의 정리를 먼저 마칩니다.

NSG로 바로 이동하여 NAT 5–6단계를 건너뛰었다면 NAT gateway/default route는 생성하지 않았으므로 아래 NAT Allow/route/gateway 삭제만 생략합니다. 존재하는 private VM·boot·subnet·규칙은 그대로 정리합니다.

1. NAT를 생성했다면 traffic을 Allow로 되돌립니다. 필요한 파일을 보관하고 **Compute → 이번 lab-private-vm → Terminate**에서 **Permanently delete attached boot volume**을 선택합니다. `TERMINATED`와 같은 AD의 이번 boot volume 삭제를 확인합니다.
2. `sl-public`에서 3단계 메모가 `이번 추가`인 `networking-private-ssh` egress private/32:22 행만 ID/필드를 대조하여 제거합니다. `이미 존재`인 규칙은 보존합니다. 원래 PC ingress22와 선택 HTTPS443은 보존합니다.
3. NAT를 만들었다면 `rt-private`의 `0.0.0.0/0 → nat-lab` 규칙이 **있을 때만** 제거합니다. SGW 전체 삭제 경로에서는 이미 제거했으므로 다시 만들지 않습니다. VCN의 모든 route table에서 이번 NAT를 target으로 하는 규칙이 없음을 확인한 뒤 **NAT Gateways → nat-lab 행 Actions → Terminate**를 실행합니다. 같은 Region·Compartment의 NAT 목록을 정상 새로고침하여 삭제를 확인합니다. 이번 NAT는 ephemeral IP이므로 별도 reserved IP 정리가 없습니다.
4. Private subnet에 VNIC가 없음을 확인하여 `lab-private`를 삭제하고, 이번 `rt-private`, `sl-private`를 삭제합니다. 실패하면 해당 자원 의존성을 읽고 다시 확인합니다.
5. 네트워크 실습을 모두 끝냈으면 [Compute 정리 1–3단계](../Compute/GettingStarted.md#리소스정리)로 public admin VM/boot를 삭제한 뒤 [VCN 정리](VCN/GettingStarted.md#리소스정리)를 진행합니다. VM이 남아 있는데 VCN Delete All을 실행하지 않습니다.
6. 더 사용하지 않을 때 PC의 이번 config와 key 파일만 정리합니다. 다른 `.ssh` 파일은 지우지 않습니다.

## 공식문서

- [NAT Gateway: 동작과 subnet route 설정](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/NATgateway.htm)
- [Creating a NAT Gateway](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/nat-create.htm)
- [Blocking or Allowing NAT Traffic](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/nat-block.htm)
- [Deleting a NAT Gateway](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/nat-delete.htm)
- [Creating a Subnet](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/create_subnet.htm)
- [Creating an Instance](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/launchinginstance.htm)
- [Security Rules](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/securityrules.htm)
