# Load Balancer

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../README.md)를 확인하세요.

> 절차 작성본 / OCI 미실행 — 아래 예상값은 실측 결과가 아닙니다. 비용·권한·사전준비 gate를 통과한 전용 환경에서만 실행하세요.

## 학습목표

- 기존 private-vm에 nginx :8080의 index/health를 설치하고 vm-a direct 요청을 검증한 뒤 sole LB :80을 생성한다.
- 생성 전 private 또는 별도 승인 public79 mode를 결정하고 LB health-file 장애/복구와 direct/LB 200을 기록한 뒤 모든 service/app-only consumer 반환을 받아 정리한다.

완료 기준은 아래 실제 생성·설정·요청 결과로 판정한다. 이 목표는 수행할 작업이며 OCI에서 관찰한 결과가 아니다.

## 대응 Roadmap 항목

- 항목명: **Load Balancer**
- Roadmap 경로: **Load Balancer**
- Node ID: `service-lb-0`
- 상위 항목: 없음 — 최상위 학습 항목
- 문서 범위: 이 항목의 개념과 작성된 실습절차. 통합 품질검토는 BLOCKED이며 실제 실행은 NOT_EXECUTED입니다.
- [전체 항목 목록](../README.md)

## 사전준비


22의 same B instance/VNIC/BACKEND_IP·ProxyJump,10의 vm-a/A_PRIVATE_IP/A_PUBLIC_IP/SSH,21/23 actual subnet/NSG/route와30 설치 경로를 받는다. app-only는 LB_SELECTED=false이고 LB_OCID가 필요 없다. LB 선택이면 생성 전 LB_MODE/FRONTEND_CLIENT_CIDR·frontend subnet을 확정한다. backend direct8080/health200 뒤만 sole LB를 생성한다.

- [Private Subnets](../03_Virtual_Cloud_Network_VCN/03_Private_Subnets.md#실습절차) — 링크의 `실습절차`와 `결과검증` 완료; 실제 OCID/IP를 registry에 인계한다.
- [Compute](../02_Compute/00_Compute.md#실습절차) — 링크의 `실습절차`와 `결과검증` 완료; 실제 OCID/IP를 registry에 인계한다.


생성 전 `REGION`, `LAB_COMPARTMENT_ID`, 승인 종료시각 `EXPIRY_UTC`, 예산 상한과 실제 resource OCID를 실행 registry에 기록한다. owner가 후속 실습 종료까지 보존하며 이번 세션은 60분 이내 종료한다. 비용 승인/리전 capacity 없으면 실행 중단한다. shell의 변수를 다른 터미널에 자동 전달된다고 가정하지 말고 비밀값 없이 다시 설정한다.
- [Compute](../02_Compute/00_Compute.md#shared-launch)의 `vm-a` SSH/Oracle Linux 9 확인 및 [Private Subnets](../03_Virtual_Cloud_Network_VCN/03_Private_Subnets.md#실습절차)의 private-vm/jump 접속 완료. backend는 기존 private-vm을 재사용하고 pool 트랙만 owner58이 추가 승인 VM을 만든다. `KEY`, `PUBLIC_IP`(vm-a), `BACKEND_IP`(private-vm), `CLIENT_PRIVATE_IP`(vm-a), `CLIENT_CIDR=$CLIENT_PRIVATE_IP/32`, subnet/NSG OCID를 실제 readback하여 인계한다. 테스트는 vm-a에서 수행한다; Cloud Shell은 자동 VCN private client가 아니다.

## 권한

GetLoadBalancer/GetBackendSet/GetHealthChecker/ListWorkRequests → LOAD_BALANCER_READ. Create/Update/DeleteListener, Backend/BackendSet, UpdateHealthChecker 및 Create/DeleteCertificate → LOAD_BALANCER_UPDATE. owner61만 CreateLoadBalancer/DeleteLoadBalancer → LOAD_BALANCER_CREATE/LOAD_BALANCER_DELETE.

[API operation별 공식 permission table](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/lbpolicyreference.htm) — 가족 단위 manage를 최소권한이라 부르지 않는다. 전용 lab compartment의 group만 적용하고 정책작성 권한은 관리자 소유다.
```text
Allow group <identity-domain>/<lab-group> to manage load-balancers in compartment <lab-compartment> where any {request.permission='LOAD_BALANCER_INSPECT', request.permission='LOAD_BALANCER_READ', request.permission='LOAD_BALANCER_UPDATE', request.permission='LOAD_BALANCER_CREATE', request.permission='LOAD_BALANCER_DELETE'}
```
이 allowlist는 위 작업만 허용한다. Console 탐색 List API가 거부되면 실제 API permission을 공식 표에서 별도로 승인하며 all-resources로 확대하지 않는다.
네트워크 생성/규칙 편집은 별도 network owner: use subnets/use vnics/use network-security-groups와 read vcns; 규칙 생성/삭제는 NETWORK_SECURITY_GROUP_UPDATE_SECURITY_RULES를 해당 NSG로 한정한다. Compute network permission table의 VNIC_CREATE/SUBNET_ATTACH/NSG membership 의존성을 확인한다. 공유 VCN 전체 manage는 최소가 아니다.

## 비용및주의사항

기존 private-vm/nginx :8080과 vm-a client를 사용하고 sole LB 하나만 생성한다. LB bandwidth/사용시간·실제 전송, 기존 두 VM/boot 및 설치 NAT 유지 비용을 승인한다. Private가 기본이며 public79 mode는 Create 전 별도 source/exposure/비용 승인이 있어야 하고 두 번째 LB/NLB를 이 문서에서 만들지 않는다. wizard Access logs/Error logs는 off, approved NSG 사용 시 security-list 자동 관리는 사용하지 않는다. 서비스 consumer와 app-only56/69/76을 같은 expiry에 등록하고 모든 반환 후 LB/앱/이번 규칙을 순서대로 정리한다.

## 실습구성

- 작업 목표: 실제 Web application과 LB 1개를 생성하여 backend→listener 전체 경로를 검증한다.
- 생성 owner: 61. 동일 owner의 자원을 재사용하며 실제 결과 필드는 실행 전 비워 둔다.


### 이번 workshop 시나리오와 인수·반환

같은 private backend nginx와 LB 하나로 직접/경유 HTTP와 health 복구를 끝낸다.

- **신규/변경:** nginx 앱(conf/index/health),LB1개/frontendNSG 규칙.
- **선행 재사용:** 22private-vm,10vm-a,30NAT install prep.
- **완주 흐름:** 승인/입력→본문의 생성/편집→direct8080 /health200/ok→LB80 /health200/index→404/unhealthy→복구→선택62–65/71–79 및 app-only56/69/76 반환→LB삭제→앱/rules원복.
- **선택 이유/실패 경계:** private default;79 public은Create전 승인 soleLB/IGW이며2번째LB 금지.

| 입력/산출물 | 실제 값 출처 | 준비 시점 |
|---|---|---|
| `BACKEND_IP` | creator의 실제 생성/조회 출력; 임의 OCID/IP 금지 | 사용 전 |
| `CLIENT_CIDR` | creator의 실제 생성/조회 출력; 임의 OCID/IP 금지 | 사용 전 |
| `LB_MODE` | 본문 단계에서 승인/선택한 값 | 사용 전 |
| `LB_IP` | creator의 실제 생성/조회 출력; 임의 OCID/IP 금지 | 생성 후 |
| `LB_OCID` | creator의 실제 생성/조회 출력; 임의 OCID/IP 금지 | 생성 후 |
| `LB_SUBNET_CIDR` | creator의 실제 생성/조회 출력; 임의 OCID/IP 금지 | 사용 전 |

입력별 실행위치는 본문 터미널 구분을 따른다: OCI CLI=인증 Cloud Shell/승인 OCI 터미널, OS 앱/NFS/부하=지정 guest, private HTTP/TCP=vm-a guest, YAML 작성/SSH private key=본인 local. 조회/변경/정리의 관리자 승인은 **실행 조건**이며 이 문서는 IAM 성공을 입증하지 않는다. 실제 허용/거부와 capacity는 사용자 실행에서 확인한다. 미승인/NOT_SELECTED/ASYNC_PENDING을 성공으로 바꾸지 않는다.

## 실습절차

### 작업 순서 체크포인트
backend direct positive→노출 mode/path enforcement→frontend rule readback→sole wizard Submit→backend health/readback→client health 및 denied path→restoration/consumer return→owner cleanup 순서다. 단계 실패는 다음 생성으로 넘기지 않고 같은 target의 복구/반환으로 처리한다.

### 실행 터미널과 지역 고정
CLI는 Cloud Shell/본인 OCI 인증 터미널에서만, `sudo`/파일 변경은 명시한 guest 터미널에서만 실행한다. 네트워크 CIDR은 기존 probe-a/probe-b 또는 10.x 예시에서 복사하지 않고 실제 VNIC/subnet 값을 조회한다.
```bash
export REGION="<approved-region>"
export OCI_CLI_REGION="$REGION"
export LAB_COMPARTMENT_ID="<actual-lab-compartment-ocid>"
export EXPIRY_UTC="<approved-session-end-UTC>"
```
치환 전 실행 금지. 생성된 OCID/IP/원래 설정/새 rule ID/owner/종료시각을 비밀 없는 registry에 기록하고 다음 터미널에서는 필요한 변수만 다시 설정한다.

[아래생성/변경단계의정확한Oracle task본문](https://docs.oracle.com/en-us/iaas/Content/Balance/Tasks/managingloadbalancer_topic-Creating_Load_Balancers.htm) — Consolefield/CLI/API동작근거.

아래 단계는 Oracle task 본문을 근거로 작성한 lab recipe이며 공식 실행결과가 아니다. 각 단계의 근거는 끝의 정확한 task 링크를 따른다.

### A 이전 네트워크/노출 선택 — app-only 포함

1. 앱만 사용하는56/69/76이면 `LB_SELECTED=false`, `CLIENT_CIDR=A_PRIVATE_IP/32`, `LB_SUBNET_CIDR=<actual private-app CIDR>`로 둔다. 이것은 기존 subnet의 승인 source 경계이며 LB_OCID/새LB를 요구하지 않는다. 실제 앱만 source는 CLIENT_CIDR이며 미래 frontend traffic을 승인하지 않았다면 subnet-source rule은 생략한다.
2. LB도 선택하면 `LB_SELECTED=true`를 guest 작업 전용 입력에도 명시한다. A 실행 **전에** 아래 lb-mode-before-create의 private/public 비용·노출 결정을 읽고 `LB_MODE`/실제 frontend subnet을 확정한다. private는 FRONTEND_CLIENT_CIDR=CLIENT_CIDR, public은 FRONTEND_CLIENT_CIDR=승인 vm-a의 실제 public egress IPv4/32(A_PUBLIC_IP와 egress readback 비교)다. backend 직접접속 source CLIENT_CIDR와 public listener source를 혼동하지 않는다. 79 public HTTP withoutTLS는 /health만 승인하며 synthetic index/8081 extension은 NOT_SELECTED, 별도승인시에만사용한다.
3. A의 `dnf` **이전에** [30 NAT installation-preparation](../03_Virtual_Cloud_Network_VCN/11_NAT_Gateway.md#nat-installation-preparation)으로 기존 private-vm의 NAT route, repository TCP80/443 egress와 actual DNS를 승인/readback한다. nat-lab는기존OCID재사용이며 repository준비때문에 LB/nginx/새VM을추가하지않는다. app-only/capture 반환까지 필요한 route/egress만 유지하고 owner30에반환한다.
4. A backend remote direct curl **이전에** vm-a nsg-admin Stateful egress TCP8080→BACKEND_IP/32, backend NSG/OS TCP8080 ingress source=CLIENT_CIDR를 승인/기록한다. LB선택때만 frontendCIDR source8080 및 frontend NSG backend8080 egress를추가한다. 공유SL/NSG broadallow는좁은규칙으로취소되지않는다.

<a id="backend-app"></a>
### A. 실제 Web application 설치 및 backend 단독 검증

**8080 workload 인수 gate (56의 앱 전용 사용에도 동일):** [24 Route Tables](../03_Virtual_Cloud_Network_VCN/05_Route_Tables.md#리소스정리)/[25 NSG](../03_Virtual_Cloud_Network_VCN/06_Network_Security_Groups.md#리소스정리)의 승인 consumer(26–28 포함)가 끝났다는 owner24/25 반환을 먼저 받는다. B 서버 터미널의 전용 `python3 -m http.server 8080`만 Ctrl-C로 종료하고 `sudo ss -lntp`에서 8080 listener가 없음을 확인한다; `pkill python`/임의 PID kill은 금지한다. 24의 packet-lab 파일/runtime rich-rule과 25의 test NSG membership/rule은 각 owner가 자기 exact ID/source/zone만 정리한다. vm-a→B TCP22 egress와 B의 A/32 TCP22 ingress, jump config/key/host verification은 그대로 유지하고 새 ProxyJump SSH를 확인한다. 8080 OCI/client/host rule을 owner61 계약(frontend CIDR+client /32)으로 명시적으로 반환/승인하고 중복 허용과 deny 실험 rule 잔여를 조회한다. 아직 승인 consumer가 8080을 쓰거나 owner 반환이 없으면 BLOCKED_WORKLOAD_HANDOFF로 멈춘다. 이미56이 같은 B에 nginx를 설치했다면 source OCID/conf/hash/200 ok를 확인하고 기존 파일 guard를 우회하지 말고 승인된 **동일 앱 재사용**으로 A 설치를 생략한다; overwrite/두 번째 앱/VM은 금지한다.
Oracle Linux 전용 backend VM 1대를 기본으로 사용한다. private client에서 SSH 가능한 연결과 NAT/package repository 접근을 먼저 확인한다. 두 번째 VM은 응답 A/B 분산이 학습목표이고 비용을 승인한 경우만 추가한다.
```bash
set -euo pipefail
test ! -e /etc/nginx/conf.d/oci-lab.conf && test ! -L /etc/nginx/conf.d/oci-lab.conf && test ! -e /var/www/oci-lab && test ! -L /var/www/oci-lab || { printf 'existing lab app: stop and use owner-approved verified reuse, do not overwrite\n'; exit 1; }
if sudo ss -lnt | grep -qE ':8080[[:space:]]'; then printf '8080 occupied: stop\n'; exit 1; fi
# Record before-state in the non-secret owner registry BEFORE installation/mutation.
# Inspect installed package and service state; nonzero for not installed/inactive is expected here.
if rpm -q nginx; then printf 'nginx preinstalled=true\n'; else printf 'nginx preinstalled=false\n'; fi
if sudo systemctl is-enabled nginx; then :; else printf 'original enabled-state is the displayed value above\n'; fi
if sudo systemctl is-active nginx; then :; else printf 'original active-state is the displayed value above\n'; fi
sudo dnf install -y nginx policycoreutils-python-utils
# Approved root-owned real parents, no concurrent config/app writer.
for parent in /etc/nginx/conf.d /var/www; do
  sudo test -d "$parent" && ! sudo test -L "$parent" || exit 1
  test "$(sudo stat -c %u "$parent")" = 0 || exit 1
  test "$(realpath -e "$parent")" = "$parent" || exit 1
  test -z "$(sudo find "$parent" -maxdepth 0 -perm /022 -print)" || exit 1
done
# Recheck after install, before the first app/config write.
for target in /etc/nginx/conf.d/oci-lab.conf /var/www/oci-lab; do
  if sudo test -e "$target" || sudo test -L "$target"; then exit 1; fi
done
sudo install -d -m 0755 /var/www/oci-lab
printf 'backend=%s\n' "$(hostname)" | sudo tee /var/www/oci-lab/index.txt >/dev/null
printf 'ok\n' | sudo tee /var/www/oci-lab/health >/dev/null
sudo tee /etc/nginx/conf.d/oci-lab.conf >/dev/null <<'NGINX'
server {
    listen 8080;
    server_name _;
    root /var/www/oci-lab;
    location = / { default_type text/plain; try_files /index.txt =404; }
    location = /health { default_type text/plain; try_files /health =404; }
}
NGINX
sudo restorecon -RFv /var/www/oci-lab
sudo semanage port -l | grep '^http_port_t'
# 8080이 이미 http_port_t에 있으면 다음 add는 생략한다.
# 전용 lab의 기존8080 type을 registry에 기록; http_cache_port_t라면 이 port만 승인 후 http_port_t로 수정하고 cleanup시 원복한다.
if ! sudo semanage port -l | grep '^http_port_t[[:space:]]' | grep -qw 8080; then
  if sudo semanage port -l | grep -E '^http_cache_port_t[[:space:]]' | grep -qw 8080; then
    sudo semanage port -m -t http_port_t -p tcp 8080
  else
    sudo semanage port -a -t http_port_t -p tcp 8080
  fi
fi
sudo nginx -t
sudo systemctl enable --now nginx
sudo systemctl reload nginx
curl --fail --connect-timeout 3 --max-time 10 -sS http://127.0.0.1:8080/health | cmp - <(printf 'ok\n')
curl --fail --connect-timeout 3 --max-time 10 -sS http://127.0.0.1:8080/ | cmp - <(printf 'backend=%s\n' "$(hostname)")
```
SELinux를 disable/permissive로 만들지 않는다. 기존 8080 등록으로 semanage add가 실패하면 포트 목록을 확인하여 skip 여부를 판단한다. 운영 nginx config는 바꾸지 않는다. 기본 80 server와 lab 8080 server를 구분한다.

`LB_SUBNET_CIDR`는 실제 frontend subnet CIDR이다. private 검증 client CIDR도 별도 승인한다. 아래 한정 firewall rule을 적용한다. firewalld가 실제 active인 경우만 사용하며 다른 firewall이면 관리자 승인 규칙을 사용한다.
```bash
# backend guest; values are received separately from Console/local/Cloud Shell
export CLIENT_CIDR='<actual-vm-a-private-ip>/32'
export LB_SELECTED='<true-or-false-approved-before-A>'
allowed_sources=("$CLIENT_CIDR")
if [ "$LB_SELECTED" = true ]; then
  export LB_SUBNET_CIDR='<actual-approved-frontend-subnet-cidr>'
  allowed_sources+=("$LB_SUBNET_CIDR")
elif [ "$LB_SELECTED" != false ]; then
  printf 'STOP: choose LB_SELECTED before firewall mutation\n'; exit 1
fi
if sudo systemctl is-active --quiet firewalld; then
# Runtime-only owned rules; no reload or permanent mutation.
# BACKEND_IFACE: actual interface of B primary VNIC, verified with ip -br address.
export BACKEND_IFACE='<actual-B-primary-interface>'
RULE_LEDGER="$HOME/oci-lab61-firewall-owned.tsv"
if test -e "$RULE_LEDGER" || test -L "$RULE_LEDGER"; then printf 'STOP: firewall ledger collision\n'; exit 1; fi
ZONE=$(sudo firewall-cmd --get-zone-of-interface="$BACKEND_IFACE") || exit 1
case "$ZONE" in ''|'no zone') printf 'STOP: interface has no active zone\n'; exit 1;; esac
sudo firewall-cmd --zone="$ZONE" --list-all || exit 1
(umask 077; set -o noclobber; : > "$RULE_LEDGER") || exit 1
for cidr in "${allowed_sources[@]}"; do
  rule="rule family=ipv4 source address=$cidr port port=8080 protocol=tcp accept"
  runtime_absent=false
  permanent_before=unknown
  for variant in runtime permanent; do
    opts=(); [ "$variant" != permanent ] || opts+=(--permanent)
    if sudo firewall-cmd --zone="$ZONE" "${opts[@]}" --query-rich-rule="$rule"; then rc=0; else rc=$?; fi
    case "$rc" in 0|1) :;; *) printf 'STOP: firewall query error %s\n' "$rc"; exit 1;; esac
    printf 'before zone=%s variant=%s rc=%s rule=%s\n' "$ZONE" "$variant" "$rc" "$rule"
    if [ "$variant" = runtime ] && [ "$rc" -eq 1 ]; then runtime_absent=true; fi
    if [ "$variant" = permanent ]; then permanent_before=$rc; fi
  done
  # Both runtime and permanent inspection succeeded before the first mutation.
  if [ "$runtime_absent" = true ]; then
    # Reserve exact recovery input BEFORE mutation; keep pending even on add failure.
    # owner61 serializes this zone/rule: no concurrent firewall writer/reload.
    printf 'pending\t%s\t1\t%s\t%s\n' "$ZONE" "$permanent_before" "$rule" >> "$RULE_LEDGER" || exit 1
    sync -f "$RULE_LEDGER" || exit 1
    sudo firewall-cmd --zone="$ZONE" --add-rich-rule="$rule" || exit 1
    # No post-add append is required for ownership; pending is recoverable.
  fi
  sudo firewall-cmd --zone="$ZONE" --query-rich-rule="$rule" || exit 1
done
# Record both before query results with ledger in owner61 registry.
# Existing permanent/runtime rules remain owned by their original owner.
fi
```
Backend NSG ingress TCP8080은 frontend CIDR 및 승인된 private test-client CIDR만 허용한다. frontend NSG ingress TCP80은 test-client CIDR만, egress TCP8080은 backend CIDR만 허용한다. stateful rule을 기본으로 사용하고 health probe 출발지 요구사항을 공식 health body와 대조한다. backend direct 테스트의 client 소스도 OS firewall에 같은 방식으로 좁게 추가한다. `curl -i http://<backend-private-ip>:8080/health`에서 200/ok, `/`에서 backend 실제 hostname을 확인한다. 실패하면 LB 생성보다 backend를 먼저 고친다.

<a id="lb-mode-before-create"></a>
### 생성 전 LB mode 결정 — 79와 동일 owner
`LB_MODE=private`, `HEALTH79_SELECTION=NOT_SELECTED`가 기본이다. 79 선택은 명시적인 별도 승인 없이는 바꾸지 않는다. **probe-source 인수 책임:** owner79의 실제 vantage name/identity raw row와 network owner 제공의 현재 Oracle 공식 source CIDR 원문(문서 URL 또는 Support reference, hash, 인용, name 매핑, 확인 UTC/유효 종료 UTC)을 owner61이 [79 사전 승인 계약](../11_DNS/03_DNS_Health_Checks.md#61-public-노출-이전-vantage-준비--create보다-먼저)에서 read back한다. 공식 List API schema에 `addressPrefixes`/`probeCIDR`가 없고 routing은 inbound 경로 설명이므로 CLI 목록에서 source CIDR를 자동 도출하지 않는다. owner79/network owner/owner61의 TCP80·health-only·sole LB·비용/expiry 승인이 모두 있어야 SELECTED/public 분기를 진행한다. 누락/만료/불일치면 `BLOCKED_VANTAGE_SOURCE`; public NSG 변경 및 public Create 전에 멈추며 기본 private 실습은 그대로 보존한다. exact NSG source CIDR/TCP80/새 rule ID와 monitor의 동일 name readback은 79 계약에 따른다. broadallow를 추가하거나 backend health 보호를 없애는 우회는 금지한다.
61~65는 vm-a **guest**에서 private LB 주소로 요청한다. 인터넷/Cloud Shell에서 private 주소를 시험하지 않는다. 실제 frontend subnet OCID, CIDR, NSG와 backend IP를 기록한다.
**79 DNS Health Checks를 이번 세션에서 실행하려면 생성 전에** public branch를 승인하고 `LB_MODE=public`로 정한다. 기존 public-admin/rt-public/IGW와 제한된 frontend NSG를 사용하며 backend는 private-app 그대로이다. mode/IP class는 이후 자동전환하지 않는다. public LB는 승인 private client의 외부 주소 및 Oracle Health Checks의 선택 vantage-point 공식 public CIDR만 listener ingress에 허용한다. 임의 IP/0.0.0.0/0 ingress는 금지한다. 79를 선택하면 **public NSG 변경/Create 이전** [79 vantage 준비](../11_DNS/03_DNS_Health_Checks.md#61-public-노출-이전-vantage-준비--create보다-먼저)의 목록 조회와 공식 probe source 확인 계약을 인수한다. source 미확인은 BLOCKED_VANTAGE_SOURCE이며 public 노출하지 않는다. 이 링크는 preparation-only이고 monitor 생성 완료를 선행 요구하지 않는다. 79가 나중에 결정되었고 LB가 이미 private이면 79는 BLOCKED_PRIVATE_ENDPOINT로 남기고 두 번째 LB를 만들지 않는다. TLS branch 인증서가 준비되지 않으면 공개 HTTP는 비민감 /health 전용, 승인된 짧은세션만 허용한다. Public mode의 LB 테스트는 public IP를 명시하며 private-default 테스트에 섞지 않는다.
Console Visibility=Private/Public를 **이 결정대로**, VCN=shared-vcn, subnet=선택 기존 subnet, frontend NSG=승인 전용 NSG, flexible shape min=10Mbps/max=10Mbps(리전 UI 허용/가격 승인시), backend set web8080을 입력한다. public reserved IP는 기본 생성하지 않는다. 같은 LB_OCID를61–65/79에 인계한다.

### B 이전 frontend NSG 소유권 station
network owner61은 VCN→Network Security Groups에서 승인된 기존 frontend NSG의 OCID/consumer/rules를 조회하거나 `nsg-lb-front`를 lab-root에 Create한다. `FRONTEND_NSG_CREATED_BY61=true/false`를 registry에 기록한다. LB 생성 전에 이 NSG에 Stateful ingress TCP80 source=FRONTEND_CLIENT_CIDR(79public은선택vantageCIDR 추가), egressTCP8080 destination=BACKEND_IP/32를추가하고새ruleID를기록한다. broaddefaultSL규칙잔존도조회한다. B Create의 NSG selection은이actualOCID다. 최종LB삭제/attachment없음확인→owner61새rules제거→**61이새로만든NSG만** empty/미참조 readback후Delete한다;승인재사용NSG는삭제하지않는다.

### Public health-only enforcement — B Create 이전 필수
`LB_MODE=public`이면 backend guest에서 다음을 실행한다. initial A의 private/direct index 확인 이후, public listener 생성 **이전**에 적용한다. HTTP8080 전체를 /health 전용으로 제한하므로 private direct `/`도403이며 다른 app consumer와 동시 사용하지 않는다. index/WAF 확장 승인은 이 restriction 해제를 자동 허용하지 않는다; 해당 owner의 노출 승인 후만 원복한다.
```bash
set -euo pipefail
CONF=/etc/nginx/conf.d/oci-lab.conf
BACKUP=/etc/nginx/conf.d/oci-lab.conf.owner61-private
sudo test -f "$CONF" && ! sudo test -L "$CONF" || exit 1
sudo test -d /etc/nginx/conf.d && ! sudo test -L /etc/nginx/conf.d || exit 1
# Exclusive owner61 config mutation scope through restoration; no outside writer.
if sudo test -e "$BACKUP" || sudo test -L "$BACKUP"; then printf 'STOP: config backup collision\n'; exit 1; fi
sudo cp -p "$CONF" "$BACKUP"
sudo tee "$CONF" >/dev/null <<'NGINX'
server {
    listen 8080;
    server_name _;
    root /var/www/oci-lab;
    location = /health { default_type text/plain; try_files /health =404; }
    location / { return 403; }
}
NGINX
restore_owner61_conf() {
  sudo test -f "$BACKUP" && ! sudo test -L "$BACKUP" || return 1
  sudo test -f "$CONF" && ! sudo test -L "$CONF" || return 1
  sudo cp -p -- "$BACKUP" "$CONF" || return 1
  sudo nginx -t && sudo systemctl reload nginx || return 1
}
# Fresh terminal recovery: re-establish CONF/BACKUP and same function.
# Public listener must be disabled/deleted/read back before private restoration.
if ! sudo nginx -t; then restore_owner61_conf; exit 1; fi
sudo systemctl reload nginx
curl --fail --max-time 10 -sS http://127.0.0.1:8080/health | cmp - <(printf 'ok\n')
for path in / /index.txt /health.saved /unknown /health/; do
  code=$(curl --max-time 10 -sS -o /dev/null -w '%{http_code}' "http://127.0.0.1:8080$path") || exit 1
  test "$code" = 403 || { printf 'STOP: unexpected exposure %s %s\n' "$path" "$code"; exit 1; }
done
```
B 생성 뒤 **같은 승인 외부 client**에서 `/health`=200/ok와 위 non-health paths=403을 다시 확인한다. OCI ingress CIDR는 path enforcement가 아니다. 불일치면 public listener를 disable/delete하고 owner61이 구성/노출을 복구한다. 최종 public listener 삭제/readback 뒤에만 backend guest에서 CONF/BACKUP을 다시 설정하고 위 regular/non-symlink guard의 restore_owner61_conf 함수를 실행→nginx -t/reload→direct /health 200 및 `/` hostname 검증→owner61이 만든 backup만 제거한다. public listener가 남은 동안 private conf를 원복하지 않는다.

### B. LB 생성 및 실제 HTTP 경로

Console → Networking → Load Balancers → Create Load Balancer. 생성 전 mode에서 선택한 frontend subnet, 이름 lab-http-lb, flexible bandwidth의 승인된 최소/최대 값을 명시한다. 자동 무제한 확대를 가정하지 않는다. backend set `web8080`, policy Round Robin, 실제 backend private IP/port8080/weight1을 등록한다. HTTP health: port8080, path `/health`, expected status200, interval10000ms, timeout3000ms, retries3을 명시한다. timeout은 interval보다 작게 한다. listener `http80`, protocol HTTP, port80, default backend set web8080을 생성한다. advanced routing rule은 기본 실습에 추가하지 않는다. 위 listener/backend 입력 후 다음 마지막 wizard 필드와 review를 완료한다.

Create wizard의 추가 필드는 다음처럼 **명시적으로** 저장한다. **Security list**=Manually configure security list rules after the backend servers are added(기존 NSG 규칙 사용, 자동 broad SL 생성 금지); **Prevent deletion of the load balancer, listeners and backends when they are still active**=unchecked(이번 owner61의 승인된 bounded 정리 선택). Submit 직전 checkbox와 생성 뒤 exact LB details의 같은 protection 상태를 readback한다. checked/미확인이면 cleanup을 강행하지 않고 owner61이 Console Edit의 해당 protection 해제 승인/저장/readback을 완료한 뒤만 Delete한다; 해제 경로/권한이 확인되지 않으면 BLOCKED_CLEANUP으로 반환하며 drain/delete API를 추측하지 않는다. **Session persistence**=Disable; **Backend SSL/Use SSL**=off(승인 비민감 HTTP8080); **Listener Proxy protocol**=disabled(이 nginx가 proxy header parser가 아님); **Manage logging → Error logs Enable**=off, **Access logs Enable**=off(기본 error logging이 켜져 있을 수 있으므로 반드시 해제, Log/Log group 추가 없음). TLS는65의 기존 인증서 승인 분기에서만 추가한다. Review and create에서 sole LB/actual subnet/NSG/10–10Mbps/웹앱1/HTTP80/health8080/rules와 로그off를 대조한 뒤 Submit한다. 화면값이 지원되지 않으면 더 큰 bandwidth/다른 service를 임의 추가하지 않고 비용 owner에게 반환한다.


Submit 뒤 Work Request와 LB Active/backend health OK를 readback한다.

### C. 직접/LB 요청, 장애 및 복구

첫 curl 전에 B 생성의 actual LB_IP를 읽고 vm-a nsg-admin Stateful egress TCP80→LB_IP/32를 network owner가 승인 추가한다. frontend ingress80 source는 **FRONTEND_CLIENT_CIDR**로 private/public mode에 맞게 저장하고80→backend8080 경로를읽는다. 이 새 rule ID를61 registry에기록한다.
```bash
export LB_IP='<actual-IP-for-recorded-LB_MODE>'
curl --connect-timeout 3 --max-time 10 -i "http://$LB_IP/health"
if [ "$LB_MODE" = private ] || [ "${SYNTHETIC_INDEX_APPROVED:-false}" = true ]; then
  for n in $(seq 1 10); do curl --connect-timeout 3 --max-time 10 -sS "http://$LB_IP/"; done
else
  printf 'public79 HTTP health-only: index exercise NOT_SELECTED\n'
fi
```
1backend 기본에서는 동일 backend 문자열이 정상이며 HA/분산을 검증했다고 주장하지 않는다. 승인된 2backend에서만 각 VM index에 서로 다른 hostname이 들어갔는지 직접 curl 후 새 요청에서 두 문자열을 관찰한다. 쿠키/session persistence는 disable한다.
실패시험 전에 backendguest에서 아래boundedblock을실행하고client/OCI조회는별도터미널에서120초안에관찰한다. owner61은session손실복구담당이며 expiry가더짧으면미선택/그deadline안에복구한다.
```bash
set -euo pipefail
HEALTH=/var/www/oci-lab/health
SAVED=/var/www/oci-lab/health.saved
sudo test -f "$HEALTH" && ! sudo test -L "$HEALTH" || exit 1
if sudo test -e "$SAVED" || sudo test -L "$SAVED"; then printf 'saved collision: stop\n' >&2; exit 1; fi
# Dedicated owner-only directory: no concurrent writer/other outage experiment.
restore_health() {
  if sudo test -e "$SAVED" || sudo test -L "$SAVED"; then
    sudo test -f "$SAVED" && ! sudo test -L "$SAVED" || return 1
    if sudo test -e "$HEALTH" || sudo test -L "$HEALTH"; then printf 'restore destination collision: stop\n' >&2; return 1; fi
    sudo mv -T -- "$SAVED" "$HEALTH" || return 1
  else
    sudo test -f "$HEALTH" && ! sudo test -L "$HEALTH" || return 1
  fi
}
trap restore_health EXIT
trap 'exit 130' INT
trap 'exit 143' TERM
sudo mv -T -- "$HEALTH" "$SAVED"
read -r -t 120 -p 'observe from other terminals; Enter restores: ' answer || true
restore_health
trap - EXIT INT TERM
curl --fail --max-time 10 http://127.0.0.1:8080/health
```
kill9/powerloss는trap불가이므로owner61복구세션에서 전용 directory 소유권/원본 인수를 확인하고 C의 HEALTH/SAVED 설정과 restore_health 함수를 다시 정의하여 실행한다. saved의 -e 또는 -L을 검사하며 regular/non-symlink source 및 -e/-L 모두 부재인 health 목적지일 때만 원복한다; collision은 덮어쓰지 않는다. 여러 probe 후 backend unhealthy를 확인하고 client 응답 코드/본문을 실제 기록한다. 1backend는 서비스 중단을 기대하되 정확한 LB 5xx 코드는 실제 관찰값으로 기록한다. 두 backend라면 살아있는 backend 응답을 검사한다. block이이미복구한뒤 saved없음/health존재를확인한다(saved잔존때만guardedrestore). 이후 direct 200, health OK, LB 응답 회복을 확인한다. 예외가 생겨도 반드시 복구한다.

### D. TLS 선택 확장
HTTP는 격리된 비민감 데이터 검증만 사용한다. 실데이터는TLS선행필수이다. public79비민감healthHTTP는생성전분기의별도짧은승인예외만허용한다. [SSL Certificates](04_SSL_Certificates.md)의 승인된 certificate/domain/SAN 및 정상 trust 검증까지 완료한 후만 HTTPS를 사용한다. private key를 로그/JSON/저장소에 넣지 않는다.

### 터미널과 network rule 실제 범위
앱sudo명령은private-vm guest, HTTPclient는vm-a guest,OCICLI는CloudShell이다. vm-a의nsg-admin은원래443egress만있으므로 **새 연결**에필요한statefulegress TCP80→LB_IP/32, TCP8080→BACKEND_IP/32를networkowner가AddRules에서추가하고ruleID를기록한다. frontendNSGingress80source=FRONTEND_CLIENT_CIDR(기본private에서는CLIENT_CIDR,승인public에서는실제publicegress/32),egress8080destination=BACKEND_IP/32를저장한다. backendNSGingress8080source=frontendCIDR및CLIENT_CIDR,OSrule은아래둘다필요하다.
이 단계는 A의 active-zone/owned-runtime firewall station readback만 재사용하며 add/reload를 반복하지 않는다. guest reboot/reload로 runtime rule이 사라졌다면 원래 ownership ledger를 확인한 owner61이 새 query/readback 후에만 복구한다.
추가전기존conf/unit파일존재하면중단하고백업/owner승인을먼저한다. dnf는 [30 NAT 설치 준비](../03_Virtual_Cloud_Network_VCN/11_NAT_Gateway.md#nat-installation-preparation)의 실제 private-outbound route/NSG TCP80,443 egress 및 DNS 확인 후 실행한다. 31은 Service Gateway이며 일반 repository용 NAT owner가 아니다. SELinux8080기존type은semanage조회로기록하고이번port만승인후변경한다. health파일이동/복원은 C의 -e/-L guard와 regular/non-symlink source 검사를 포함한 함수만 사용한다.
```bash
export LB_OCID='<created-load-balancer-ocid>'
oci lb load-balancer get --load-balancer-id "$LB_OCID"
oci lb backend-set get --load-balancer-id "$LB_OCID" --backend-set-name web8080
oci lb backend-set-health get --load-balancer-id "$LB_OCID" --backend-set-name web8080
```
Public79branch는79HTTP80요구와같은단일LB를사용하며비민감health만짧은승인예외이다. 실데이터에는HTTPS필수;엄격TLS정책이면79도HTTPS443으로담당자와맞추기전BLOCKED이며자동public/TLS전환없다.

## 결과검증

direct /health와 LB /health의 200/ok, index hostname, backend health 전이와 복구를 구분해 기록한다. 실행 증거 없는 필드는 비워 둔다.

- 예상 pattern이며 실제 OCI 실행 결과가 아니다. timestamp, resource OCID, 설정 readback, 실제 HTTP/TCP/파일 결과를 실행자가 기록한다.

## 문제해결

direct8080먼저확인→LBhealthsource규칙→listener80→clientegress. NAT준비없으면dnf실패;publicprivatemode혼동시LB중복생성하지않는다.

실패기록에는같은target/credentials/region/요청시간을고정한다. 복구는원래설정으로실행하고actualresponse/readback둘다확인한다;예상값을실측으로쓰지않는다.

## 리소스정리

owner61은 LB/backend/app/network rule registry에 62–65, 선택 71–75 WAF, 선택78 steering, 선택79 external Health Checks 각각의 SELECTED/NOT_SELECTED·owner·반환 상태·EXPIRY_UTC·비용 상한을 기록한다. 앱만 소비하는 선택69 NLB HTTP health와 선택76 DNS HTTP 검증도 app registry에 기록하고 health 복구/반환 전 nginx/conf/health를 정리하지 않는다; 이 소비자에게 LB 생성을 강제하지 않는다. 56이 앱만 사용한 경우 그 capture 완료/반환도 기록한다. WAF75는71의 saved config 복원/nginx -t/reload 및 /health 200/ok와 firewall/policy/log 삭제를 반환하고,78은 attachment/policy/zone 삭제,79는 monitor disable/delete 및 health 복구를 반환한다. 모두 선택된 consumer의 실제 반환 전 LB/backend를 삭제하지 않는다. 예약 consumer가 없으면 즉시, expiry 도달 시 미완료 시험을 중단·복구·반환하고 정리한다; 추후 실습을 이유로 idle 유료 자원을 무기한 보존하지 않는다. private LB는79 대상이 아니며 public71–75의 추가 경로 승인은 별도다. 최종 삭제는 owner61만 수행한다. owner61은 정확한 LB의 active-resource deletion protection=off readback을 확인한 뒤 Console Delete→같은 Region/identity의 LB list에서 exact OCID 없음으로 검증한다. on/미확인은 B의 owner-approved 해제 gate를 먼저 완료한다. 새로 만든 /var/www/oci-lab/index.txt와health만확인후삭제하고빈labdirectory만제거한다. nginx가원래설치/실행중이었는지registry로확인하고공유서비스를임의stop/uninstall하지않는다. backend의실습nginxconf만삭제/백업원복→nginx -t/reload. firewall은 `$HOME/oci-lab61-firewall-owned.tsv`의 이 실행이 추가한 runtime rule만 아래 코드로 제거하며 permanent/preexisting rule과 다른 zone은 변경하지 않는다. NSG새ruleID만삭제. semanage8080원래type이http_cache_port_t면 `sudo semanage port -m -t http_cache_port_t -p tcp 8080`,이번에새로add했다면 `sudo semanage port -d -p tcp 8080`;기존http_port_t는유지. vm-a/private-vm은upstreamowner.

정리실행자의실제UTC/OCID/readback을기록한다. 공유resource는owner확인전삭제금지; stop만으로bootvolume/LB/image비용종료가아니다.


```bash
# backend guest; run after all consumers return; exact created-runtime ledger only
set -euo pipefail
RULE_LEDGER="$HOME/oci-lab61-firewall-owned.tsv"
test -f "$RULE_LEDGER" && test ! -L "$RULE_LEDGER" || exit 1
# New terminal recovery uses the same block, including after interrupted/failed add.
# Confirm owner61 exclusive mutation lease and original ledger before any removal.
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
done < "$RULE_LEDGER"
# Save actual cleanup evidence, then remove this owned ledger only.
rm -- "$RULE_LEDGER"
```

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 목표는 학습 후 설명·구분·확인할 내용이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [2] [Overview of Load Balancer — Load Balancer](https://docs.oracle.com/en-us/iaas/Content/Balance/Concepts/balanceoverview.htm)
  - 확인 범위: Load Balancer.

- [33] [Overview of Flexible Network Load Balancer — Load Balancer와 Network Load Balancer 비교](https://docs.oracle.com/en-us/iaas/Content/NetworkLoadBalancer/overview.htm)
  - 확인 범위: Comparing Load Balancer and Network Load Balancer / Load balancers / Network load balancers.

### 절차별 공식 task body 추가 근거
- [balanceoverview.htm](https://docs.oracle.com/en-us/iaas/Content/Balance/Concepts/balanceoverview.htm) — 본문 검토 근거; 아래 정확한 task와 작성한 입력을 구분한다. OCI 실행 증거가 아니다.
- [managingloadbalancer_topic-Creating_Load_Balancers.htm](https://docs.oracle.com/en-us/iaas/Content/Balance/Tasks/managingloadbalancer_topic-Creating_Load_Balancers.htm) — 본문 검토 근거; 아래 정확한 task와 작성한 입력을 구분한다. OCI 실행 증거가 아니다.
- [managingbackendsets.htm](https://docs.oracle.com/en-us/iaas/Content/Balance/Tasks/managingbackendsets.htm) — 본문 검토 근거; 아래 정확한 task와 작성한 입력을 구분한다. OCI 실행 증거가 아니다.
- [managinglisteners.htm](https://docs.oracle.com/en-us/iaas/Content/Balance/Tasks/managinglisteners.htm) — 본문 검토 근거; 아래 정확한 task와 작성한 입력을 구분한다. OCI 실행 증거가 아니다.
- [load_balancer_health_management.htm](https://docs.oracle.com/en-us/iaas/Content/Balance/Tasks/load_balancer_health_management.htm) — 본문 검토 근거; 아래 정확한 task와 작성한 입력을 구분한다. OCI 실행 증거가 아니다.



### 이번 절차의 정확한 공식 task
- [managingloadbalancer_topic-Creating_Load_Balancers.htm](https://docs.oracle.com/en-us/iaas/Content/Balance/Tasks/managingloadbalancer_topic-Creating_Load_Balancers.htm) — 이 문서 실습절차의 입력/동작을 대조한 본문.
- [managingbackendsets_topic-Creating_Backend_Sets.htm](https://docs.oracle.com/en-us/iaas/Content/Balance/Tasks/managingbackendsets_topic-Creating_Backend_Sets.htm) — 이 문서 실습절차의 입력/동작을 대조한 본문.

- [Oracle Linux NGINX tutorial — Install NGINX/Configure SELinux/Firewall](https://docs.oracle.com/en/learn/oracle-linux-nginx/index.html) — OS설치근거만사용; 이실습은공식예시의anywhereingress를따르지않고승인CIDR로제한한다.
