> **기존 실습 초안 — 미검증**: 원래 93개 문서의 절차를 보존한 자료입니다. 새 튜토리얼의 품질 승인이나 OCI 실행 검증을 뜻하지 않습니다.

# Network Load Balancer

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../../../../Docs/README.md)를 확인하세요.

> 절차 작성본 / OCI 미실행 — 아래 예상값은 실측 결과가 아닙니다. 비용·권한·사전준비 gate를 통과한 전용 환경에서만 실행하세요.

## 학습목표

- 기존 private-vm에 line echo :9090을 설치하고 Compute-instance backend/Full NAT private NLB 하나를 생성한다.
- 실제 probe source 규칙 이후 direct/NLB exact echo와 bounded stop/start 장애·복구를 기록하고 67–70 반환 뒤 NLB/이번 앱을 정리한다.

완료 기준은 아래 실제 생성·설정·요청 결과로 판정한다. 이 목표는 수행할 작업이며 OCI에서 관찰한 결과가 아니다.

## 대응 Roadmap 항목

- 항목명: **Network Load Balancer**
- Roadmap 경로: **Network Load Balancer**
- Node ID: `service-nlb-0`
- 상위 항목: 없음 — 최상위 학습 항목
- 문서 범위: 이 항목의 개념과 작성된 실습절차. 통합 품질검토는 BLOCKED이며 실제 실행은 NOT_EXECUTED입니다.
- [전체 항목 목록](../../../../Docs/README.md)

## 사전준비


22 same backend OCID/VNIC/BACKEND_IP/SSH와10 vm-a CLIENT_PRIVATE_IP/CLIENT_CIDR/hostname client를 받는다. Python 부재 때만30 NAT 설치 준비를 사용한다. NLB_IP/OCID와 PROBE_CIDR는 생성 뒤 실제 조회값이고 probe를 VIP로 추정하지 않는다.69 선택 때만61 app-only를 보존하며 LB는 필요 없다.

- [Private Subnets](../03_Virtual_Cloud_Network_VCN/03_Private_Subnets.md#실습절차) — 링크의 `실습절차`와 `결과검증` 완료; 실제 OCID/IP를 registry에 인계한다.
- [Compute](../../../Original-Provisional/02_Compute/00_Compute.md#실습절차) — 링크의 `실습절차`와 `결과검증` 완료; 실제 OCID/IP를 registry에 인계한다.


생성 전 `REGION`, `LAB_COMPARTMENT_ID`, 승인 종료시각 `EXPIRY_UTC`, 예산 상한과 실제 resource OCID를 실행 registry에 기록한다. owner가 후속 실습 종료까지 보존하며 이번 세션은 60분 이내 종료한다. 비용 승인/리전 capacity 없으면 실행 중단한다. shell의 변수를 다른 터미널에 자동 전달된다고 가정하지 말고 비밀값 없이 다시 설정한다.
- [Compute](../../../Original-Provisional/02_Compute/00_Compute.md#shared-launch)의 `vm-a` SSH/Oracle Linux 9 확인 및 [Private Subnets](../03_Virtual_Cloud_Network_VCN/03_Private_Subnets.md#실습절차)의 private-vm/jump 접속 완료. backend는 기존 private-vm을 재사용하고 pool 트랙만 owner58이 추가 승인 VM을 만든다. `KEY`, `PUBLIC_IP`(vm-a), `BACKEND_IP`(private-vm), `CLIENT_PRIVATE_IP`(vm-a), `CLIENT_CIDR=$CLIENT_PRIVATE_IP/32`, subnet/NSG OCID를 실제 readback하여 인계한다. 테스트는 vm-a에서 수행한다; Cloud Shell은 자동 VCN private client가 아니다.

## 권한

GetNetworkLoadBalancer/GetBackendSet/GetBackendHealth/GetHealthChecker → NETWORK_LOAD_BALANCER_READ. Create/Update/Delete backend set/listener, UpdateHealthChecker → NETWORK_LOAD_BALANCER_UPDATE. owner66만 CreateNetworkLoadBalancer/DeleteNetworkLoadBalancer → NETWORK_LOAD_BALANCER_CREATE/NETWORK_LOAD_BALANCER_DELETE.

[API operation별 공식 permission table](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/nlbpolicyreference.htm) — 가족 단위 manage를 최소권한이라 부르지 않는다. 전용 lab compartment의 group만 적용하고 정책작성 권한은 관리자 소유다.
```text
Allow group <identity-domain>/<lab-group> to manage network-load-balancers in compartment <lab-compartment> where any {request.permission='NETWORK_LOAD_BALANCER_INSPECT', request.permission='NETWORK_LOAD_BALANCER_READ', request.permission='NETWORK_LOAD_BALANCER_UPDATE', request.permission='NETWORK_LOAD_BALANCER_CREATE', request.permission='NETWORK_LOAD_BALANCER_DELETE'}
```
이 allowlist는 위 작업만 허용한다. Console 탐색 List API가 거부되면 실제 API permission을 공식 표에서 별도로 승인하며 all-resources로 확대하지 않는다.
네트워크 생성/규칙 편집은 별도 network owner: use subnets/use vnics/use network-security-groups와 read vcns; 규칙 생성/삭제는 NETWORK_SECURITY_GROUP_UPDATE_SECURITY_RULES를 해당 NSG로 한정한다. Compute network permission table의 VNIC_CREATE/SUBNET_ATTACH/NSG membership 의존성을 확인한다. 공유 VCN 전체 manage는 최소가 아니다.

## 비용및주의사항

기존 private-vm의 line echo9090과 vm-a client를 쓰고 private NLB 하나만 만든다. NLB/전송의 현행 계정 가격조건, 두 기존 VM/boot 및 정확한 probe/데이터 경로를 승인하며 LB bandwidth/인증서 비용을 이 TCP 실습에 끼워 넣지 않는다. 추가 VM/LB/public endpoint는 없다.69 HTTP probe를 선택할 때만61 app-only nginx 보존을 별도 등록한다.67–70 반환 뒤 NLB→이번 echo/unit/rules를 정리하고 공유 VM은 각 creator에게 반환한다.

## 실습구성

- 작업 목표: 실제 TCP app과 NLB 1개를 생성하고 return path까지 검증한다.
- 생성 owner: 66. 동일 owner의 자원을 재사용하며 실제 결과 필드는 실행 전 비워 둔다.


### 이번 workshop 시나리오와 인수·반환

기존 private-vm의 line echo와 private NLB 하나로 TCPpayload/반환을 읽는다.

- **신규/변경:** echo.py/unit,TCP9090,NLB1개.
- **선행 재사용:** 22private-vm,10vm-a;선택69앱만61A.
- **완주 흐름:** 승인/입력→본문의 생성/편집→direct9090 echo→NLB echo/peer→stop/unhealthy→start/새echo→67–70반환→NLB삭제→echo/unit/rules삭제→22 반환.
- **선택 이유/실패 경계:** Compute-instance backend 선택;FullNAT/FailOpenfalse/transparentfalse;nginx 선택69만.

| 입력/산출물 | 실제 값 출처 | 준비 시점 |
|---|---|---|
| `BACKEND_IP` | creator의 실제 생성/조회 출력; 임의 OCID/IP 금지 | 사용 전 |
| `NLB_OCID` | creator의 실제 생성/조회 출력; 임의 OCID/IP 금지 | 생성 후 |
| `NLB_IP` | creator의 실제 생성/조회 출력; 임의 OCID/IP 금지 | 생성 후 |
| `NLB_SUBNET_CIDR` | creator의 실제 생성/조회 출력; 임의 OCID/IP 금지 | 사용 전 |
| `PROBE_CIDR` | creator의 실제 생성/조회 출력; 임의 OCID/IP 금지 | 생성 후 |
| `CLIENT_CIDR` | creator의 실제 생성/조회 출력; 임의 OCID/IP 금지 | 사용 전 |

입력별 실행위치는 본문 터미널 구분을 따른다: OCI CLI=인증 Cloud Shell/승인 OCI 터미널, OS 앱/NFS/부하=지정 guest, private HTTP/TCP=vm-a guest, YAML 작성/SSH private key=본인 local. 조회/변경/정리의 관리자 승인은 **실행 조건**이며 이 문서는 IAM 성공을 입증하지 않는다. 실제 허용/거부와 capacity는 사용자 실행에서 확인한다. 미승인/NOT_SELECTED/ASYNC_PENDING을 성공으로 바꾸지 않는다.

## 실습절차

### 실행 터미널과 지역 고정
CLI는 Cloud Shell/본인 OCI 인증 터미널에서만, `sudo`/파일 변경은 명시한 guest 터미널에서만 실행한다. 네트워크 CIDR은 기존 probe-a/probe-b 또는 10.x 예시에서 복사하지 않고 실제 VNIC/subnet 값을 조회한다.
```bash
export REGION="<approved-region>"
export OCI_CLI_REGION="$REGION"
export LAB_COMPARTMENT_ID="<actual-lab-compartment-ocid>"
export EXPIRY_UTC="<approved-session-end-UTC>"
```
치환 전 실행 금지. 생성된 OCID/IP/원래 설정/새 rule ID/owner/종료시각을 비밀 없는 registry에 기록하고 다음 터미널에서는 필요한 변수만 다시 설정한다.

[아래생성/변경단계의정확한Oracle task본문](https://docs.oracle.com/en-us/iaas/Content/NetworkLoadBalancer/NetworkLoadBalancers/create-network-load-balancer.htm) — Consolefield/CLI/API동작근거.

아래 단계는 Oracle task 본문을 근거로 작성한 lab recipe이며 공식 실행결과가 아니다. 각 단계의 근거는 끝의 정확한 task 링크를 따른다.

### A. 실제 TCP backend 서비스
선택69 HTTP health를 수행할 때만 [Load Balancer 앱 전용 A단계](../../../Original-Provisional/08_Load_Balancer/00_Load_Balancer.md#backend-app)로 Nginx HTTP8080을 준비한다. 기본66 TCP9090에는 nginx/LB resource 생성이 모두 필수조건이 아니다. 추가 서비스는 아래 전용 파일로 TCP9090 line echo를 구현한다. Oracle Linux python3 설치 상태를 `command -v python3`로 확인한다. 없으면 **dnf 이전에** [30 NAT installation-preparation](../03_Virtual_Cloud_Network_VCN/11_NAT_Gateway.md#nat-installation-preparation)의 기존 B용 NAT/repository TCP80/443 egress 및 DNS를 승인/readback하고 `sudo dnf install -y python3`를 실행한다. 설치후임시route/rule을owner30에반환한다. nginx/LB생성은이fallback의조건아니다.
```bash
set -euo pipefail
test ! -e /opt/oci-lab/echo.py && test ! -L /opt/oci-lab/echo.py && test ! -e /etc/systemd/system/oci-lab-echo.service && test ! -L /etc/systemd/system/oci-lab-echo.service || { printf 'existing app/unit: stop and inspect\n'; exit 1; }
if sudo ss -lnt | grep -qE ':9090[[:space:]]'; then printf '9090 already in use\n'; exit 1; fi
for parent in /opt /etc/systemd/system; do
  sudo test -d "$parent" && ! sudo test -L "$parent" || exit 1
  test "$(realpath -e "$parent")" = "$parent" || exit 1
done
ECHO_DIR_CREATED=0
if sudo test -e /opt/oci-lab || sudo test -L /opt/oci-lab; then
  sudo test -d /opt/oci-lab && ! sudo test -L /opt/oci-lab || exit 1
else
  sudo install -d -m 0755 /opt/oci-lab || exit 1
  ECHO_DIR_CREATED=1
fi
# persist exact ECHO_DIR_CREATED and own file/unit paths for later cleanup
sudo tee /opt/oci-lab/echo.py >/dev/null <<'PY'
import socketserver, socket
class Echo(socketserver.StreamRequestHandler):
    def handle(self):
        print('peer=', self.client_address, flush=True)
        for line in self.rfile:
            print('request=', repr(line), 'peer=', self.client_address, flush=True)
            self.wfile.write((socket.gethostname()+':').encode()+line)
            self.wfile.flush()
class Server(socketserver.ThreadingTCPServer):
    allow_reuse_address=True
    daemon_threads=True
with Server(('0.0.0.0',9090), Echo) as server:
    server.serve_forever()
PY
sudo tee /etc/systemd/system/oci-lab-echo.service >/dev/null <<'UNIT'
[Unit]
Description=OCI lab TCP echo
After=network.target
[Service]
User=nobody
ExecStart=/usr/bin/python3 /opt/oci-lab/echo.py
Restart=on-failure
NoNewPrivileges=true
ProtectSystem=strict
ProtectHome=true
[Install]
WantedBy=multi-user.target
UNIT
sudo systemctl daemon-reload
sudo systemctl enable --now oci-lab-echo
sudo ss -lntp | grep ':9090'
```

### tcp-firewall

Health source는 Backend server details의 Health check source IP 또는 HealthCheckResult.sourceIpAddress에서 실제조회하여 PROBE_CIDR=<source-ip>/32로 설정한다. NLB의 VIP가 probe source라고 단정하지 않는다. 이 source gate는 B 생성 후 C 검증 전에 수행한다. B 생성전에는 승인된 NLB subnet/client 9090 규칙만 준비한다. NLB 생성 직후 실제 probe source를 받아 규칙을 추가하고 healthy를 기다린다. frontend NSG ingress9090 source=$CLIENT_CIDR, egress9090 destination=$BACKEND_IP/32; backend NSG ingress9090 source=실제 NLB subnet CIDR, PROBE_CIDR 및 CLIENT_CIDR를 승인한다. CLIENT_CIDR는 C의 직접 TCP 양성대조에도 필수다. 68 source-preserve 실험용 CLIENT_CIDR도 사전승인한다.
```bash
export NLB_SUBNET_CIDR='<actual-selected-subnet-cidr>'
export CLIENT_CIDR='<vm-a-private-ip>/32'
set -euo pipefail
sudo systemctl is-active --quiet firewalld || { printf 'STOP: approved active firewalld required\n'; exit 1; }
# ip -br address로 primary VNIC IP와 인터페이스를 대조하여 입력한다.
BACKEND_IFACE='<actual-primary-VNIC-interface>'
ZONE=$(sudo firewall-cmd --get-zone-of-interface="$BACKEND_IFACE") || exit 1
case "$ZONE" in ''|'no zone') exit 1;; esac
sudo firewall-cmd --zone="$ZONE" --list-all || exit 1
LEDGER="$HOME/oci-lab66-firewall-0.tsv"
test ! -e "$HOME/oci-lab66-firewall-0.tsv" && test ! -L "$HOME/oci-lab66-firewall-0.tsv" || exit 1
(umask 077; set -o noclobber; : > "$HOME/oci-lab66-firewall-0.tsv") || exit 1
for cidr in "$NLB_SUBNET_CIDR" "$CLIENT_CIDR"; do
  rule="rule family=ipv4 source address=$cidr port port=9090 protocol=tcp accept"
  absent=false
  runtime_before=unknown
  permanent_before=unknown
  for variant in runtime permanent; do
    opts=(); [ "$variant" != permanent ] || opts+=(--permanent)
    if sudo firewall-cmd --zone="$ZONE" "${opts[@]}" --query-rich-rule="$rule"; then rc=0; else rc=$?; fi
    case "$rc" in 0|1) :;; *) printf 'STOP: query error %s\n' "$rc"; exit 1;; esac
    printf 'before zone=%s variant=%s rc=%s rule=%s\n' "$ZONE" "$variant" "$rc" "$rule"
    if [ "$variant" = runtime ]; then runtime_before=$rc; [ "$rc" != 1 ] || absent=true; fi
    if [ "$variant" = permanent ]; then permanent_before=$rc; fi
  done
  if [ "$absent" = true ]; then
    # Exclusive owner lease: no concurrent firewall writer/reload.
    test "$runtime_before" = 1 || exit 1
    case "$permanent_before" in 0|1) :;; *) exit 1;; esac
    printf 'pending\t%s\t%s\t%s\t%s\n' "$ZONE" "$runtime_before" "$permanent_before" "$rule" >> "$LEDGER" || exit 1
    sync -f "$LEDGER" || exit 1
    sudo firewall-cmd --zone="$ZONE" --add-rich-rule="$rule" || exit 1
  fi
  sudo firewall-cmd --zone="$ZONE" --query-rich-rule="$rule" || exit 1
done
```
추가 전 sync한 pending journal의 exact zone/rule/runtime/permanent before-state를 보존하고 아래 owned cleanup을 사용한다. permanent/preexisting 규칙은 보존한다.
OS firewall/NSG ingress9090은 실제 승인 client CIDR과 NLB/probe 송신원만 허용한다. source preservation을 켜면 backend에서 client source가 보이므로 frontend CIDR 규칙만으로는 부족하다. 61 A단계 rich rule의 port를9090으로 바꿔 한정 적용하고 SELinux는 enforcing을 유지한다. AVC가 생기면 로그/승인 정책을 조사하며 disable하지 않는다.


### nlb-create

### B 이전 NLB frontend NSG 생성/재사용
network owner66은 기존승인frontendNSG가있으면actualOCID/rule/다른consumer를조회하여재사용한다. 없으면동일VCN/lab-root에 `nsg-nlb-front` 한개Create하고 `NLB_FRONTEND_NSG_OCID`, `NLB_FRONTEND_NSG_CREATED_BY66=true/false`를registry에기록한다. Stateful ingressTCP9090 source=CLIENT_CIDR,egressTCP9090 destination=BACKEND_IP/32를추가/readback한다. B Create의NSGselection은이actualOCID이며attach상태를NLBreadback에서확인한다. 최종NLB삭제/attachment없음→66새rules삭제→66신규NSG만empty/미참조확인뒤Delete한다. 재사용NSG와타owner규칙은보존하며69새8080규칙은69가먼저반환한다.

### B. NLB 생성
Console → Networking → Network Load Balancers → Create에서 private NLB 1개, 승인 frontend subnet, 이름 lab-tcp-nlb를 선택한다. backend set echo9090, policy Five Tuple, **Backend type=Compute instances**를 선택하고 owner22 private-vm의 실제 instance OCID와 해당 VNIC private IP/port9090/weight1을 입력한다. IP address-only backend는68 source-preservation 분기와 호환되는 선택으로 취급하지 않는다. source IP preservation의 실제 UI 선택값을 명시해 저장한다. 기본 Full NAT: NLB 수준 Source/destination header preservation=false, backend set Preserve source IP=false를 명시한다. 68에서 backend-set source만 true로 바꾼다. source-preserve 선택값을 빈칸으로 두지 않는다. TCP health port9090, interval10000ms/timeout3000ms/retries3을 명시한다. listener TCP9090/default echo9090을 생성한다. NLB에 HTTP routing/TLS termination을 설정하지 않는다.


### nlb-ready

### C 이전 post-create network/readback station
B생성 직후 actual NLB_IP/OCID/HealthCheckResult.sourceIpAddress를조회한다. backendguest에 PROBE_CIDR=actual-source/32를설정하고 NSG/OS Stateful9090 ingress를그source로추가한다. frontend NSG egress9090→BACKEND_IP/32, ingress9090→CLIENT_CIDR와 vm-a NSG egress9090→BACKEND_IP/32 및 NLB_IP/32를승인/readback한다. NLB/GetBackendSet/GetListener에서 FullNAT,FailOpen=false,instantfailover=false,transparent=false,listener9090/360s/proxyNone을확인한후healthy를기다린다. probe-source가조회불가면추측VIP규칙을추가하지말고중단한다. 이후에만C/D접속한다.
```bash
export PROBE_CIDR='<actual-health-check-source-ip>/32'
set -euo pipefail
sudo systemctl is-active --quiet firewalld || { printf 'STOP: approved active firewalld required\n'; exit 1; }
# ip -br address로 primary VNIC IP와 인터페이스를 대조하여 입력한다.
BACKEND_IFACE='<actual-primary-VNIC-interface>'
ZONE=$(sudo firewall-cmd --get-zone-of-interface="$BACKEND_IFACE") || exit 1
case "$ZONE" in ''|'no zone') exit 1;; esac
sudo firewall-cmd --zone="$ZONE" --list-all || exit 1
LEDGER="$HOME/oci-lab66-firewall-1.tsv"
test ! -e "$HOME/oci-lab66-firewall-1.tsv" && test ! -L "$HOME/oci-lab66-firewall-1.tsv" || exit 1
(umask 077; set -o noclobber; : > "$HOME/oci-lab66-firewall-1.tsv") || exit 1
for cidr in "$PROBE_CIDR"; do
  rule="rule family=ipv4 source address=$cidr port port=9090 protocol=tcp accept"
  absent=false
  runtime_before=unknown
  permanent_before=unknown
  for variant in runtime permanent; do
    opts=(); [ "$variant" != permanent ] || opts+=(--permanent)
    if sudo firewall-cmd --zone="$ZONE" "${opts[@]}" --query-rich-rule="$rule"; then rc=0; else rc=$?; fi
    case "$rc" in 0|1) :;; *) printf 'STOP: query error %s\n' "$rc"; exit 1;; esac
    printf 'before zone=%s variant=%s rc=%s rule=%s\n' "$ZONE" "$variant" "$rc" "$rule"
    if [ "$variant" = runtime ]; then runtime_before=$rc; [ "$rc" != 1 ] || absent=true; fi
    if [ "$variant" = permanent ]; then permanent_before=$rc; fi
  done
  if [ "$absent" = true ]; then
    # Exclusive owner lease: no concurrent firewall writer/reload.
    test "$runtime_before" = 1 || exit 1
    case "$permanent_before" in 0|1) :;; *) exit 1;; esac
    printf 'pending\t%s\t%s\t%s\t%s\n' "$ZONE" "$runtime_before" "$permanent_before" "$rule" >> "$LEDGER" || exit 1
    sync -f "$LEDGER" || exit 1
    sudo firewall-cmd --zone="$ZONE" --add-rich-rule="$rule" || exit 1
  fi
  sudo firewall-cmd --zone="$ZONE" --query-rich-rule="$rule" || exit 1
done
```

### C. TCP 직접/경유 검증 및 return path
private test client에서 아래 프로그램의 host에 backend IP를 넣어 먼저 성공시킨 다음 NLB IP로 재실행한다.
```bash
export TCP_HOST='<backend-or-nlb-private-ip>'
export EXPECTED_BACKEND_HOSTNAME='<actual-approved-backend-hostname>'
export REQUEST_ID="lab-$(date -u +%Y%m%dT%H%M%SZ)-$$"
python3 - <<'PY'
import os,socket
with socket.create_connection((os.environ['TCP_HOST'],9090),timeout=5) as s:
    payload=(os.environ['REQUEST_ID']+':hello-nlb\n').encode()
    print('client_tuple=',s.getsockname(),'target=',s.getpeername(),flush=True)
    s.sendall(payload)
    with s.makefile('rb') as stream:
        line=stream.readline(4096)
        expected=os.environ['EXPECTED_BACKEND_HOSTNAME'].encode()+b':'+payload
        if line != expected: raise RuntimeError('wrong/empty/truncated echo: '+repr(line))
        print(line.decode())
PY
```
기대 pattern은 `<실제-hostname>:hello-nlb`이다. Active/healthy와 실제 payload response 모두 필요하다. `sudo journalctl -u oci-lab-echo --since '-5 min'`의 peer로 source를 확인한다. preservation 설정과 backend `ip route get <client-ip>` 및 VCN route table을 대조한다. source/destination preservation/transparent 방식을 사용하면 공식 문서의 symmetry/return routing 조건을 모두 충족해야 하며 기본 인터넷 gateway로 돌려보내지 않는다. 첫 실습은 같은 VCN의 승인된 private client를 사용한다.

### D. 제한 장애/복구
**먼저 아래 stop 이전 bounded 서비스 복구 station을실행한다**(barestop금지). 다른client/OCI터미널에서 direct TCP 실패, 반복 health 후 unhealthy, NLB 접속 실패를 기록한다. 1backend에서는 중단이 정상 expected negative test이다. `sudo systemctl start oci-lab-echo` 후 direct echo→healthy→NLB echo를 재확인한다. 여러 backend/flow HA 검증은 별도 승인된 선택 과제로만 한다.

### Full NAT와 fail-closed 명시
Createbackendset에서**Failopen=unchecked/false**,Enableinstantfailover=false,PreservesourceIP=false를명시하고NLB수준source/destinationpreservation=false를읽는다. Failopen이true면전체unhealthy여도TCPtraffic이전달될수있어69오류예상이성립하지않는다. listenerTCP9090/timeout360초/proxyprotocolNone,NSG=승인frontendNSG,subnet=private-app으로선택한다. port9090이이미사용중이면설치중단한다. `sudo ss -lntp`를읽고/opt/oci-lab/echo.py/unit기존존재시덮어쓰지않는다. vm-a NSGstatefulegress9090→backend /32및NLB /32를승인한다.
```bash
export NLB_OCID='<created-nlb-ocid>'
oci nlb network-load-balancer get --network-load-balancer-id "$NLB_OCID"
oci nlb backend-set get --network-load-balancer-id "$NLB_OCID" --backend-set-name echo9090
oci nlb backend-set-health get --network-load-balancer-id "$NLB_OCID" --backend-set-name echo9090
```
privateNLBclient는vm-a;shared LBresource는필수아니다. A단계의nginx는69HTTPhealth를선택할때만준비한다.

### stop 이전 bounded 서비스 복구 station
backend guest 별도터미널에서다음을사용하고client/OCI관측은다른터미널에서수행한다. expiry가120초보다짧으면남은승인시간만사용/미선택한다. kill9/powerloss는별도owner66복구세션이즉시start한다.
```bash
set -euo pipefail
sudo systemctl is-active --quiet oci-lab-echo || exit 1
restore_echo() { sudo systemctl start oci-lab-echo; }
trap restore_echo EXIT
trap 'exit 130' INT
trap 'exit 143' TERM
sudo systemctl stop oci-lab-echo
read -r -t 120 -p 'observe bounded failure in other terminals; Enter restores: ' answer || true
restore_echo
trap - EXIT INT TERM
sudo systemctl is-active --quiet oci-lab-echo
```
복구뒤direct새echo→NLBhealthOK→NLB새echo를66C정확payload로확인한다. barestop/start대신이station만사용한다.


### Owned guest firewall cleanup — 각 ledger만 반환
변경 전 runtime/permanent 조회 출력과 zone/rule을 registry에 보존한다. 기존 규칙은 ledger에 추가하지 않는다. reload/runtime-to-permanent는 금지한다. 재부팅/firewalld reload 뒤에는 runtime 허용이 사라질 수 있으므로 시험중단/owner 재승인 후 새 ledger로 다시 준비한다. 다른 owner의 runtime 규칙을 flush하지 않는다.
```bash
set -euo pipefail
# 이번 owner의 실제 ledger 경로를 하나씩 선택한다. 66은 0과 1, 69는 0.
LEDGER='<actual-this-owner-ledger>'
test -f "$LEDGER" && test ! -L "$LEDGER" || exit 1
while IFS=$'\t' read -r state zone runtime_before permanent_before rule; do
  test "$state" = pending && test "$runtime_before" = 1 || exit 1
  case "$permanent_before" in 0|1) :;; *) exit 1;; esac
  test -n "$zone" && test -n "$rule" || exit 1
  sudo firewall-cmd --zone="$zone" --list-all >/dev/null || exit 1
  # Query BOTH actual runtime and permanent in the recorded exact zone.
  if sudo firewall-cmd --zone="$zone" --query-rich-rule="$rule"; then runtime_now=0; else runtime_now=$?; fi
  if sudo firewall-cmd --zone="$zone" --permanent --query-rich-rule="$rule"; then permanent_now=0; else permanent_now=$?; fi
  case "$runtime_now:$permanent_now" in 0:0|0:1|1:0|1:1) :;; *) exit 1;; esac
  # Changed permanent state means outside mutation: stop for owner reconciliation.
  test "$permanent_now" = "$permanent_before" || exit 1
  if [ "$runtime_now" = 0 ]; then
    sudo firewall-cmd --zone="$zone" --remove-rich-rule="$rule" || exit 1
  fi
  if sudo firewall-cmd --zone="$zone" --query-rich-rule="$rule"; then exit 1; else rc=$?; test "$rc" -eq 1 || exit 1; fi
  # Never remove permanent or pre-existing runtime rules.
  if sudo firewall-cmd --zone="$zone" --permanent --query-rich-rule="$rule"; then rc=0; else rc=$?; fi
  test "$rc" = "$permanent_before" || exit 1
done < "$LEDGER"
rm -- "$LEDGER"
```
동일 rule을 다른 consumer가 인수했으면 cleanup 전에 소유권/반환을 조율한다. permanent/preexisting/61 direct-client rules는 삭제하지 않는다.

## 결과검증

backend direct/NLB TCP payload와 peer source, health 장애/복구를 실제 기록한다. TCP connect만으로 payload 검증을 대체하지 않는다.

- 예상 pattern이며 실제 OCI 실행 결과가 아니다. timestamp, resource OCID, 설정 readback, 실제 HTTP/TCP/파일 결과를 실행자가 기록한다.

## 문제해결

healthsourceIP는NLBVIP아님;FullNAT/Failopenfalse/9090serverlisten및firewall확인. healthconnect는echoappbytes검증아님.

실패기록에는같은target/credentials/region/요청시간을고정한다. 복구는원래설정으로실행하고actualresponse/readback둘다확인한다;예상값을실측으로쓰지않는다.

## 리소스정리

owner66은67–70의 SELECTED/NOT_SELECTED, 실제 반환, expiry/비용 상한을 먼저 읽는다. 승인 expiry 또는 모든 선택 consumer 반환 후 stop/복구하고 최종 정리한다. 69 nginx는 owner61 앱 소비자 registry에 반환하며 LB resource를 요구하지 않는다. NLBConsoleDelete→NLBlist없음. guest `sudo systemctl disable --now oci-lab-echo; sudo rm -- /etc/systemd/system/oci-lab-echo.service /opt/oci-lab/echo.py; sudo systemctl daemon-reload`. 추가9090richrule은 위 owned guest firewall cleanup으로 ledger에 기록한 runtime 규칙만 제거;NSG새ruleID만삭제. 기존private-vm/LB미삭제. echo.py/unit 삭제·daemon-reload 후 ECHO_DIR_CREATED=1 소유 기록일 때만 빈 `/opt/oci-lab`을 `sudo rmdir /opt/oci-lab`로 제거한다; 기존 directory/다른 파일은 보존한다.

정리실행자의실제UTC/OCID/readback을기록한다. 공유resource는owner확인전삭제금지; stop만으로bootvolume/LB/image비용종료가아니다.

## 공식문서

- [Oracle Linux 9 firewalld zone/runtime 설정](https://docs.oracle.com/en/operating-systems/oracle-linux/9/firewall/firewall-ConfiguringfirewalldZones.html) — 인터페이스별 zone과 temporary/runtime 설정 근거; query 오류/ownership ledger는 작성한 안전 gate다.

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 목표는 학습 후 설명·구분·확인할 내용이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [33] [Overview of Flexible Network Load Balancer — Network Load Balancer](https://docs.oracle.com/en-us/iaas/Content/NetworkLoadBalancer/overview.htm)
  - 확인 범위: Network Load Balancer.

### 절차별 공식 task body 추가 근거
- [overview.htm](https://docs.oracle.com/en-us/iaas/Content/NetworkLoadBalancer/overview.htm) — 본문 검토 근거; 아래 정확한 task와 작성한 입력을 구분한다. OCI 실행 증거가 아니다.
- [create-network-load-balancer.htm](https://docs.oracle.com/en-us/iaas/Content/NetworkLoadBalancer/NetworkLoadBalancers/create-network-load-balancer.htm) — 본문 검토 근거; 아래 정확한 task와 작성한 입력을 구분한다. OCI 실행 증거가 아니다.
- [backend-set-management.htm](https://docs.oracle.com/en-us/iaas/Content/NetworkLoadBalancer/BackendSets/backend-set-management.htm) — 본문 검토 근거; 아래 정확한 task와 작성한 입력을 구분한다. OCI 실행 증거가 아니다.
- [health-check-policy-management.htm](https://docs.oracle.com/en-us/iaas/Content/NetworkLoadBalancer/HealthCheckPolicies/health-check-policy-management.htm) — 본문 검토 근거; 아래 정확한 task와 작성한 입력을 구분한다. OCI 실행 증거가 아니다.



### 이번 절차의 정확한 공식 task
- [create-network-load-balancer.htm](https://docs.oracle.com/en-us/iaas/Content/NetworkLoadBalancer/NetworkLoadBalancers/create-network-load-balancer.htm) — 이 문서 실습절차의 입력/동작을 대조한 본문.
- [health-check-policy-management.htm](https://docs.oracle.com/en-us/iaas/Content/NetworkLoadBalancer/HealthCheckPolicies/health-check-policy-management.htm) — 이 문서 실습절차의 입력/동작을 대조한 본문.
