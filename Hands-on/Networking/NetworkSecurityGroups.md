# Network Security Groups: private 웹 응답을 허용·차단·복구하기

**Network Security Group(NSG)**은 subnet 전체가 아니라 선택한 VNIC의 애플리케이션 통신을 허용하는 규칙 묶음입니다. 이번에는 기존 public admin VM을 client로 사용하여 private VM의 작은 HTTP 페이지를 읽습니다. 서버 프로세스와 guest firewall은 그대로 두고 NSG 규칙 하나만 제거하여 새 연결이 막히는지 확인한 뒤 복구합니다.

[Networking 목록](README.md) · [NSG 생성](#2-client와-app-nsg-만들고-vnic-연결하기) · [HTTP 시험](#4-admin-vm에서-private-http-읽기) · [정리](#리소스정리)

## 학습목표

- 기존 두 VM의 primary VNIC에 client/app NSG를 연결하고 NSG를 source/destination으로 지정한 TCP8080 규칙을 만듭니다.
- private VM에서 실제 작은 HTTP 서버를 실행하여 admin VM에서 지정 파일의 응답을 읽습니다.
- 다른 허용 규칙과 guest firewall의 영향을 분리하고 NSG ingress 제거 → 새 연결 실패 → 같은 규칙 복구 → HTTP 성공을 확인합니다.

## 대응 Roadmap 항목

**Network Security Groups** — Virtual Cloud Network (VCN)의 VNIC 단위 security rules. Security Lists, Ingress/Egress와 stateful 응답의 의미를 실습 중 구분합니다.

## 사전준비

[NAT Gateway 1–4단계의 private SSH baseline](NATGateway.md#private-ssh-baseline-준비-완료)을 완료하여 public admin `lab-vm`과 `lab-private-vm`, 두 subnet, PC의 `NETWORKING_CONFIG`에 지정한 전용 config를 남깁니다. 새 PC 터미널 A/B/C마다 [NAT 4단계](NATGateway.md#4-pc에서-private-vm으로-접속하기)의 메모에 남긴 같은 절대경로로 변수를 설정하고 두 HostName/키를 확인합니다. private SSH 접속까지 완료해야 합니다. 새 VM/키를 추가하지 않습니다. 생성 단계가 부족하면 NAT 문서의 **private subnet/VM 전체 절차**로 돌아갑니다.

[Service Gateway](ServiceGateway.md)를 먼저 했다면 그 문서 [정리 1–4단계](ServiceGateway.md#리소스정리)를 모두 마칩니다. 계속 실습 분기를 선택하여 NAT default route/일반 HTTPS443을 복원하고, PAR·SGW에 이어 object와 bucket까지 삭제합니다. NSG에는 이 저장 자원이 필요하지 않습니다. Object Storage 실습은 NSG의 선행조건이 아닙니다. 설치가 필요하지 않으면 NAT outbound 사용 여부와 무관하게 내부 HTTP 시험은 가능합니다.

Console에서 두 VM의 **primary VNIC private IPv4**를 메모합니다. 아래 `ADMIN_PRIVATE_IP`, `PRIVATE_IP`는 예시 IP가 아니라 해당 상세에서 읽은 실제 값입니다. PC→private 관리 SSH는 ProxyJump, HTTP client는 **public admin VM의 guest 터미널**입니다. PC에서 private IP로 직접 curl하거나 SSH forwarding으로 우회하여 시험하지 않습니다.

## 권한

같은 Compartment/VCN의 NSG 생성·조회·규칙 추가/조회/제거·삭제와 VNIC NSG membership 변경/조회가 필요합니다. [NSG 공식 IAM 안내](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/networksecuritygroups.htm)는 VCN 관련 권한과 VNIC 사용 권한도 요구합니다. NSG를 source/destination으로 지정할 때는 해당 NSG inspect 권한이 필요합니다. 편의 범위 `manage virtual-network-family`는 이 작업을 포괄하지만 최소권한 policy는 아닙니다. 기존 개인 관리자 계정이면 새 policy를 만들 필요가 없습니다.

private guest에서 `opc`의 sudo로 필요한 Python 설치와 runtime firewalld 규칙 추가/제거를 합니다. 이것은 OCI IAM과 다른 OS 권한입니다. 권한 오류면 그 단계에서 중단하고 원인을 확인합니다. NSG나 guest firewall 전체를 끄지 않습니다.

## 비용및주의사항

VM 두 대/boot volume 두 개를 **재사용**합니다. NSG 두 개와 8080 규칙 두 개 외에 VM, LB, public listener, logging은 만들지 않습니다. VM·boot·공인 IPv4·NAT/전송의 현재 과금과 종료 시점을 확인합니다. 웹 페이지는 작은 한 줄이며 Python 서버는 이번 내부 시험용입니다. 인터넷 서비스 운영용으로 사용하지 않습니다.

**NSG와 Security List의 허용은 합집합**입니다. 좁은 NSG는 넓은 Security List를 취소하지 않습니다. private subnet에 all-ingress나 source VCN/16 TCP-all/8080 허용이 남으면 NSG 규칙을 지워도 성공할 수 있습니다. 그러한 상태에서 deny를 성공으로 기록하지 않습니다. 기존 다른 네트워크를 고치지 않고 이 새 개인 lab의 범위에서만 시험합니다.

## 실습구성

| 자원 | 이번 작업 | 남길 관리 경로 |
|---|---|---|
| public admin VM primary VNIC | 신규 `nsg-client`에 연결, app NSG 대상 egress8080 | sl-public의 PC/32 ingress22 |
| private VM primary VNIC | 신규 `nsg-app`에 연결, client NSG source ingress8080 | sl-private의 admin/32 ingress22 |
| private VM guest | 작은 파일 + foreground Python HTTP8080, 필요한 runtime firewall 허용 | SSH22/sshd 변경 없음 |

새 연결에는 **client egress와 server ingress 둘 다** 필요합니다. 두 규칙은 stateful이므로 server의 HTTP 응답만을 위한 egress8080 규칙은 추가하지 않습니다. NSG를 만들기만 해서는 VNIC에 적용되지 않으므로 membership도 확인합니다.

## 실습절차

### 1. 다른 8080 허용이 없는지 먼저 확인하기 — Console

**Networking → Virtual cloud networks → lab-vcn → Subnets**에서 `lab-public`은 `sl-public` 하나, `lab-private`는 `sl-private` 하나를 연결한 상태인지 확인합니다. 이어 각 security list의 양쪽 규칙을 읽습니다.

- `sl-public`: PC/32 ingress22, private VM/32 egress22, 선택 egress443만 허용. **8080/모든 TCP/모든 protocol egress 없음**.
- `sl-private`: admin/32 ingress22, egress443만 허용. **8080/모든 TCP/모든 protocol ingress 없음**.
- **Compute → 각 VM → Networking → primary VNIC → Network Security Groups**: 아직 NSG 없음(NAT 기본 구성). 있다면 모든 연결 NSG를 읽어 8080을 우회 허용하는 규칙이 없는지 확인합니다.

Default security list를 연결했거나 broad allow가 있다면 시험을 시작하지 않습니다. 새 개인 lab에서 문서와 다르게 추가한 허용만 확인하여 제거하고 위 baseline으로 되돌립니다. 모르는 규칙이 있는 기존 환경에서는 별도 새 lab으로 돌아갑니다. ZPR security attributes/policy는 추가하지 않습니다. NSG는 route나 OS firewall을 대신하지 않습니다.

### 2. Client와 app NSG 만들고 VNIC 연결하기

**같은 VCN → Security → Network Security Groups → Create Network Security Group**(또는 Resources 메뉴)에서 같은 Compartment에 `nsg-client`와 `nsg-app`을 각각 만듭니다. 처음에는 빈 규칙으로 만듭니다. 이름/VCN/Compartment와 규칙 0개를 확인합니다.

각 NSG의 **Add Rules**에서 다음 행을 추가합니다.

| NSG / 방향 | Source 또는 Destination Type | 선택 NSG | Protocol / ports | Description |
|---|---|---|---|---|
| nsg-client / Egress | Network Security Group | `nsg-app` | TCP / source All, destination 8080 | `client-to-app-8080` |
| nsg-app / Ingress | Network Security Group | `nsg-client` | TCP / source All, destination 8080 | `app-from-client-8080` |

**Stateless는 해제**합니다. 같은 VCN의 NSG를 선택하며 CIDR `0.0.0.0/0`으로 대신하지 않습니다. 저장 후 규칙을 읽고 제거/복구할 app ingress의 rule ID(표시되는 경우)와 필드를 메모합니다.

**Compute → admin VM → Networking → Attached VNICs → primary VNIC 상세 → Network Security Groups → Edit**에서 `nsg-client`를 연결합니다. private VM의 primary VNIC에는 같은 방법으로 `nsg-app`을 연결합니다. 화면에 따라 VNIC 상세의 Edit Network Security Groups로 표시됩니다. 기존 membership이 있었다면 임의로 해제하지 않습니다.

VNIC 상세와 NSG의 VNIC 목록에서 **public primary VNIC → client**, **private primary VNIC → app**를 양쪽으로 확인합니다. secondary VNIC나 반대 VM을 선택하지 않습니다. 실패하면 실제 membership을 읽고 같은 대상만 수정합니다. private SSH가 유지되는지도 확인합니다.

### 3. Private VM에서 작은 HTTP 서버 준비하기

| PC 터미널 | SSH 대상 / 역할 |
|---|---|
| A | private VM: 준비 후 foreground HTTP 서버 유지 |
| B | private VM: loopback 확인과 정리 |
| C | admin VM: private HTTP client |

**PC 터미널 A**에서 `ssh -F "${NETWORKING_CONFIG:?NAT 4단계의 config 절대경로를 먼저 설정하세요}" networking-private`로 접속합니다. 다음 명령은 모두 **private VM guest Bash**입니다.

```bash
id -un
command -v python3
```

`opc`와 Python 경로를 기대합니다. Python이 없을 때만 [NAT 5단계](NATGateway.md#5-private-vm에서-https-요청하기)에서 outbound 경로를 만들고 HTTPS를 확인합니다(NAT 6단계 차단 시험은 설치의 필수 조건이 아닙니다). NAT route/HTTPS443/Allow가 있는지 확인하고 **private VM**에서 `sudo dnf install -y python3`을 실행합니다. 설치 오류면 중단하고 NAT/저장소 HTTPS를 진단하며 큰 별도 VM을 만들지 않습니다. 설치한 Python은 OS 기본 의존성일 수 있으므로 정리할 때 무작정 uninstall하지 않습니다.

고유한 새 디렉터리와 지정 파일을 만듭니다. 기존 파일을 덮어쓰지 않습니다.

```bash
APP_DIR=$(mktemp -d "$HOME/nsg-web.XXXXXX") || exit 1
printf '%s\n' 'nsg-lab-ok' > "$APP_DIR/health.txt" || exit 1
printf '이번 app 경로: %s\n' "$APP_DIR"
ss -ltn '( sport = :8080 )'
```

8080 listener가 이미 있으면 다른 프로세스를 종료하지 말고 여기서 중단합니다. 이 새 VM에서는 비어 있어야 합니다. `APP_DIR` 값을 메모합니다. 아래 firewall 작업은 같은 터미널에서 이어갑니다.

**Guest firewall**은 cloud NSG와 별개입니다. `sudo firewall-cmd --state`와 `sudo firewall-cmd --get-active-zones`를 실행합니다. firewalld가 running이고 primary interface가 속한 zone을 확인했을 때만 아래 `ZONE`을 그 실제 값으로 바꿉니다. runtime 규칙만 추가하고 permanent 변경/전체 reload는 하지 않습니다. query 오류를 “규칙 없음”으로 취급하지 않습니다.

아래 접힌 실행 명령에서 admin private IP와 active zone을 지정하고 실행합니다. query가 성공한 뒤에만 이번 runtime 허용을 추가하며 기존 규칙은 보존합니다. 마지막 query가 `yes`/exit0가 아니면 서버를 시작하지 않습니다.

<details>
<summary>Guest firewall 실행 명령과 선택 사항: 중단 복구 메모</summary>

```bash
ADMIN_PRIVATE_IP='<admin VM 실제 private IPv4>'
ZONE='<primary interface의 실제 active zone>'
RULE="rule family=ipv4 source address=$ADMIN_PRIVATE_IP/32 port port=8080 protocol=tcp accept"
sudo firewall-cmd --state || exit 1
sudo firewall-cmd --zone="$ZONE" --list-all >/dev/null || exit 1
printf '%s\n' "$ZONE" > "$APP_DIR/firewall-zone.txt" || exit 1
printf '%s\n' "$RULE" > "$APP_DIR/firewall-rule.txt" || exit 1
if sudo firewall-cmd --zone="$ZONE" --query-rich-rule="$RULE"; then
  printf '%s\n' 'pre-existing' > "$APP_DIR/firewall-state.txt"
else
  RC=$?
  if test "$RC" -ne 1; then printf '%s\n' 'STOP: firewall query 오류'; exit 1; fi
  printf '%s\n' 'pending-add' > "$APP_DIR/firewall-state.txt" || exit 1
  sudo firewall-cmd --zone="$ZONE" --add-rich-rule="$RULE" || exit 1
  printf '%s\n' 'added' > "$APP_DIR/firewall-state.txt" || exit 1
fi
sudo firewall-cmd --zone="$ZONE" --query-rich-rule="$RULE"
```

마지막은 `yes`/exit0를 기대합니다. state/rule/zone 파일은 이 실습에서 추가했는지 정리 시 확인하기 위한 메모입니다. pre-existing이면 해당 기존 규칙은 삭제하지 않습니다. `pending-add`에서 끊기면 아래 정리의 interrupted branch를 사용합니다. Firewalld가 실행되지 않거나 zone을 확인할 수 없으면 자동으로 켜거나 다른 firewall을 끄지 말고 중단합니다. 기본 Oracle Linux 9의 firewall 경로와 다른 환경은 별도 확인이 필요합니다.


중단 복구는 정리 2번의 `pending-add` 분기입니다. 정상 첫 성공 경로에서 중단 실험이나 별도 복구 시험은 요구하지 않습니다.

</details>

이제 같은 터미널 A에서 다음 foreground 서버를 실행하여 그대로 둡니다.

```bash
python3 -m http.server 8080 --bind 0.0.0.0 --directory "$APP_DIR"
```

`Serving HTTP ... port 8080`을 기대합니다. Bind 오류면 위 listener와 프로세스를 확인하고 진행하지 않습니다. **PC의 새 터미널 B**에서 private VM으로 다시 SSH 접속하여 `curl --noproxy '*' --fail --max-time 5 http://127.0.0.1:8080/health.txt`를 실행합니다. `nsg-lab-ok`가 나오면 app 자체는 준비됐습니다. 이는 loopback 검사이며 아직 NSG 통신 성공이 아닙니다. `/` directory listing이 아니라 **정확한 `/health.txt`**를 사용합니다.

### 4. Admin VM에서 private HTTP 읽기

**PC의 새 터미널 C**에서 `ssh -F "${NETWORKING_CONFIG:?NAT 4단계의 config 절대경로를 먼저 설정하세요}" networking-admin`으로 public VM에 접속합니다. 아래 명령은 **admin VM guest**에서 실행합니다. NAT/인터넷을 통하지 않고 같은 VCN의 private IP로 요청합니다.

```bash
PRIVATE_IP='<private VM primary VNIC 실제 private IPv4>'
probe_app() {
  curl -4 --noproxy '*' --connect-timeout 5 --max-time 10 \
    -H 'Connection: close' -sS \
    -w '\nhttp=%{http_code} connect=%{time_connect}\n' \
    "http://$PRIVATE_IP:8080/health.txt"
  RC=$?
  printf 'curl_exit=%s\n' "$RC"
}
probe_app
```

예상은 본문 `nsg-lab-ok`, HTTP200, `curl_exit=0`입니다. 이것으로 client egress·app ingress·VNIC membership·guest firewall·서버까지 한 경로를 확인합니다. 404이면 서버에는 도착했지만 파일이 잘못된 것이므로 NSG 실패로 분류하지 않습니다. Connection refused이면 listener부터, timeout이면 rule/membership/guest firewall부터 확인합니다.

### 5. App ingress만 제거하고 복구하기

Console의 **lab-vcn → nsg-app → Security Rules**에서 **이번 `app-from-client-8080` ingress 한 행만 Remove**합니다. source=`nsg-client`, TCP8080인 행인지 ID/필드로 재확인합니다. `nsg-client` egress와 sl-private SSH22, 서버, guest firewall은 그대로 둡니다.

동일한 admin VM 터미널 C에서 `probe_app`를 다시 실행합니다. 새 curl은 연결을 재사용하지 않습니다. 예상은 HTTP000, connect0, 흔히 exit28입니다. 터미널 B에서는 같은 loopback `/health.txt`가 계속 `nsg-lab-ok`인지 확인합니다. 내부 app 정상인데 admin의 새 연결만 실패해야 NSG ingress 제거 효과를 분리한 것입니다.

**실패 결과와 무관하게 곧바로 nsg-app → Add Rules**에서 2단계와 정확히 같은 stateful ingress `source NSG=nsg-client`, TCP source All/destination8080을 복원합니다. 새 rule ID는 달라질 수 있습니다. 저장된 필드를 읽고 터미널 C의 같은 `probe_app`로 HTTP200/본문 복구를 확인합니다. NSG를 VNIC에서 떼거나 security list에 broad8080을 추가하는 방식으로 복구하지 않습니다.

제거 중에도 성공하면 1단계의 모든 Security List/NSG 허용을 다시 읽습니다. 같은 VM/IP에서 실행하는지, proxy/SSH tunnel을 사용하는지 확인합니다. 기존 tracked connection 대신 새 curl인지도 확인합니다. guest firewall을 같이 막아 가짜 NSG deny를 만들지 않습니다. 브라우저가 끊기면 SSH 관리 경로로 app 상태를 확인하고 Console에서 같은 ingress부터 복원합니다.

## 결과검증

완료 조건은 올바른 primary VNIC membership, 두 NSG 규칙, **admin VM의 private IP HTTP200 → app ingress 제거 시 신규 연결 실패(동시에 loopback 정상) → 같은 ingress 복원 후 HTTP200**입니다. 시험 중 destination IP/port/file, client host, guest firewall과 서버를 유지합니다. `AVAILABLE` 또는 rule 저장만으로 실제 응답을 대신하지 않습니다. 이 문서의 결과는 실행할 때 확인할 예상이며 OCI 실행 관측값이 아닙니다.

## 문제해결

| 증상 | 확인과 복구 |
|---|---|
| 첫 HTTP부터 timeout | client egress/app ingress 둘 다, 정확한 VNIC membership, 실제 두 private IP, guest active zone/rule 확인 |
| loopback도 실패 | foreground Python 프로세스·8080 listener·health.txt 경로 확인. NSG를 변경하기 전에 app부터 복구 |
| 404 / directory listing | 정확한 `/health.txt`와 `--directory`의 이번 APP_DIR 확인 |
| 제거 후 계속 HTTP200 | 모든 SL/NSG의 중복 allow와 다른 client/endpoint 경유를 확인. NSG에는 deny override가 없음 |
| SSH가 끊김 | 8080 행만 제거했는지 확인. 관리 ingress22/egress22는 보존해야 함 |
| NSG 삭제 불가 | 각 primary VNIC에서 이번 membership 먼저 해제. 다른 VNIC가 남으면 대상을 확인 |

## 리소스정리

1. 시험한 ingress를 복원한 상태에서 최종 HTTP200를 확인합니다. 터미널 A의 foreground Python을 **Ctrl+C**로 종료합니다. 터미널 B에서 `ss -ltn '( sport = :8080 )'`로 이번 listener가 사라졌는지 확인합니다. 다른 8080 프로세스가 생겼다면 임의로 kill하지 않습니다.
2. private VM의 Bash에서 아래를 실행합니다. `APP_DIR`에는 3단계에서 출력된 **이번 정확한 경로**를 다시 지정합니다. state=`added` 또는 `pending-add`는 이 개인 VM에서 본인이 단독 변경한 규칙만 정리합니다. 기존 rule/state 파일이 없거나 다른 변경을 했으면 자동 제거하지 말고 먼저 확인합니다.

```bash
APP_DIR='<3단계에서 출력된 이번 /home/opc/nsg-web.XXXXXX 경로>'
ZONE=$(cat "$APP_DIR/firewall-zone.txt") || exit 1
RULE=$(cat "$APP_DIR/firewall-rule.txt") || exit 1
STATE=$(cat "$APP_DIR/firewall-state.txt") || exit 1
sudo firewall-cmd --zone="$ZONE" --list-all >/dev/null || exit 1
case "$STATE" in
  added|pending-add)
    sudo firewall-cmd --state || exit 1
    if sudo firewall-cmd --zone="$ZONE" --query-rich-rule="$RULE"; then
      sudo firewall-cmd --zone="$ZONE" --remove-rich-rule="$RULE" || exit 1
    else
      RC=$?
      if test "$RC" -ne 1; then printf '%s\n' 'STOP: query 오류'; exit 1; fi
    fi
    ;;
  pre-existing) printf '%s\n' '기존 guest 규칙은 그대로 보존합니다' ;;
  *) printf '%s\n' 'STOP: state 확인 필요'; exit 1 ;;
esac
```

이번 추가 규칙은 runtime에서 제거되었고 permanent 설정은 바꾸지 않았습니다. 같은 query가 `no`/exit1인지 확인합니다(pre-existing은 `yes`로 보존). 확인을 마친 뒤 이번 `APP_DIR` 디렉터리만 삭제합니다. firewall 규칙을 영구 저장하거나 전체 reload하지 않습니다.

3. **Compute → 각 primary VNIC → Edit Network Security Groups**에서 이번 `nsg-client`/`nsg-app` membership만 해제하고 읽어 확인합니다. 기존 다른 membership은 유지합니다. NSG의 VNIC 목록이 비어 있으면 **lab-vcn → Network Security Groups → 각 NSG 행 Actions → Terminate**에서 이번 두 NSG를 삭제합니다. 목록을 정상 새로고침하여 삭제를 확인합니다. 관리 SSH는 security list 경로로 유지됩니다.
4. 더 네트워크 실습을 하지 않는다면 [NAT 리소스정리](NATGateway.md#리소스정리)로 private VM/boot, 이번 public egress22, NAT/private subnet, public VM/boot, VCN 순으로 삭제합니다. 위 사전준비대로 SGW 정리 1–4단계를 마쳤다면 PAR·SGW·object·bucket은 이미 없으므로 그 단계를 반복하지 않습니다. 예외적으로 PAR/SGW가 아직 있으면 먼저 [SGW 정리](ServiceGateway.md#리소스정리)의 남은 단계만 완료합니다. PAR/SGW는 삭제했고 object/bucket만 남았다면 [Object Storage 정리](../ObjectStorage/GettingStarted.md#리소스정리)로 바로 가서 남은 object→bucket만 삭제한 뒤 NAT 정리로 갑니다. Stop만으로 boot volume 비용이 끝난다고 생각하지 않습니다.

## 공식문서

- [Creating an NSG](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/create-nsg.htm)
- [Managing NSG Security Rules](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/manage-nsg-security-rules.htm)
- [Deleting an NSG](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/delete_nsg.htm)
- [Oracle Linux: active zones와 runtime 설정](https://docs.oracle.com/en/operating-systems/oracle-linux/9/firewall/firewall-ConfiguringfirewalldZones.html)
- [firewall-cmd rich-rule 명령](https://firewalld.org/documentation/man-pages/firewall-cmd.html)

- [Network Security Groups: membership, NSG source/destination, IAM와 삭제](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/networksecuritygroups.htm)
- [Security Rules: Security List/NSG 합집합과 stateful 응답](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/securityrules.htm)
- [VNIC 관리](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/managingVNICs.htm)
- [Core Services IAM reference](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm)
- [Oracle Linux 9: firewalld](https://docs.oracle.com/en/operating-systems/oracle-linux/9/firewall/)
- [Python http.server](https://docs.python.org/3/library/http.server.html) — 작은 내부 테스트 서버 명령의 근거
