# Health Check Policies

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../README.md)를 확인하세요.

> 절차 작성본 / OCI 미실행 — 아래 예상값은 실측 결과가 아닙니다. 비용·권한·사전준비 gate를 통과한 전용 환경에서만 실행하세요.

## 학습목표

- same NLB의 TCP9090 health를 기존 app-only nginx HTTP8080 /health로 바꾸고 실제 probe8080 규칙을 승인/readback한다.
- echo는 정상인 채 bounded health-file404/unhealthy를 대조하고 file/HTTP200/NLB echo 복구 후 original TCP health와 이번 probe규칙을 반환한다.

완료 기준은 아래 실제 생성·설정·요청 결과로 판정한다. 이 목표는 수행할 작업이며 OCI에서 관찰한 결과가 아니다.

## 대응 Roadmap 항목

- 항목명: **Health Check Policies**
- Roadmap 경로: **Network Load Balancer → Health Check Policies**
- Node ID: `service-nlb-3`
- 상위 항목: **Network Load Balancer** (`service-nlb-0`)
- 문서 범위: 이 항목의 개념과 작성된 실습절차. 통합 품질검토는 BLOCKED이며 실제 실행은 NOT_EXECUTED입니다.
- [전체 항목 목록](../README.md)

## 사전준비


66 same NLB_OCID/echo9090/probe actual source와61 same B의 app-only8080/health200을 받는다.69를 두 creator lease에 등록하고 frontend8080 egress·backend probe8080/client8080 규칙을 변경 전에 승인한다. LB 생성은 없고 original TCP9090 checker 전체값을 저장/복원한다.

- [Network Load Balancer](00_Network_Load_Balancer.md#실습절차) — 링크의 `실습절차`와 `결과검증` 완료; 실제 OCID/IP를 registry에 인계한다.


생성 전 `REGION`, `LAB_COMPARTMENT_ID`, 승인 종료시각 `EXPIRY_UTC`, 예산 상한과 실제 resource OCID를 실행 registry에 기록한다. owner가 후속 실습 종료까지 보존하며 이번 세션은 60분 이내 종료한다. 비용 승인/리전 capacity 없으면 실행 중단한다. shell의 변수를 다른 터미널에 자동 전달된다고 가정하지 말고 비밀값 없이 다시 설정한다.
- [Compute](../02_Compute/00_Compute.md#shared-launch)의 `vm-a` SSH/Oracle Linux 9 확인 및 [Private Subnets](../03_Virtual_Cloud_Network_VCN/03_Private_Subnets.md#실습절차)의 private-vm/jump 접속 완료. backend는 기존 private-vm을 재사용하고 pool 트랙만 owner58이 추가 승인 VM을 만든다. `KEY`, `PUBLIC_IP`(vm-a), `BACKEND_IP`(private-vm), `CLIENT_PRIVATE_IP`(vm-a), `CLIENT_CIDR=$CLIENT_PRIVATE_IP/32`, subnet/NSG OCID를 실제 readback하여 인계한다. 테스트는 vm-a에서 수행한다; Cloud Shell은 자동 VCN private client가 아니다.

## 권한

GetNetworkLoadBalancer/GetBackendSet/GetBackendHealth/GetHealthChecker → NETWORK_LOAD_BALANCER_READ. Create/Update/Delete backend set/listener, UpdateHealthChecker → NETWORK_LOAD_BALANCER_UPDATE.

[API operation별 공식 permission table](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/nlbpolicyreference.htm) — 가족 단위 manage를 최소권한이라 부르지 않는다. 전용 lab compartment의 group만 적용하고 정책작성 권한은 관리자 소유다.
```text
Allow group <identity-domain>/<lab-group> to manage network-load-balancers in compartment <lab-compartment> where any {request.permission='NETWORK_LOAD_BALANCER_INSPECT', request.permission='NETWORK_LOAD_BALANCER_READ', request.permission='NETWORK_LOAD_BALANCER_UPDATE'}
```
이 allowlist는 위 작업만 허용한다. Console 탐색 List API가 거부되면 실제 API permission을 공식 표에서 별도로 승인하며 all-resources로 확대하지 않는다.
네트워크 생성/규칙 편집은 별도 network owner: use subnets/use vnics/use network-security-groups와 read vcns; 규칙 생성/삭제는 NETWORK_SECURITY_GROUP_UPDATE_SECURITY_RULES를 해당 NSG로 한정한다. Compute network permission table의 VNIC_CREATE/SUBNET_ATTACH/NSG membership 의존성을 확인한다. 공유 VCN 전체 manage는 최소가 아니다.

## 비용및주의사항

66 NLB/echo9090과61 app-only nginx8080을 재사용하며 LB 생성은 필요하지 않다. 신규 범위는 임시 HTTP health/probe8080 규칙과 bounded health-file outage뿐이다. 기존 NLB/VM/boot·probe/client 전송 및 두 creator의 동일 승인 expiry를 확인한다. 실제 probe source는 NLB VIP로 추정하지 않는다. 파일/HTTP200/NLB echo 복구 후 original TCP9090 health→이번 probe rules 정리→61앱/66NLB 반환 순서다.

## 실습구성

- 작업 목표: 동일 NLB의 TCP/HTTP health를 앱 수준 응답으로 구분한다.
- 생성 owner: 66. 동일 owner의 자원을 재사용하며 실제 결과 필드는 실행 전 비워 둔다.


### 이번 workshop 시나리오와 인수·반환

echo는살아있는채 별도 nginx health404로 NLB backend 제외와 복원을 관찰한다.

- **신규/변경:** NLBhealth TCP9090→HTTP8080→TCP9090;probe8080rules.
- **선행 재사용:** 66NLB,61app-only200/ok;LB없음.
- **완주 흐름:** 승인/입력→본문의 생성/편집→HTTP404/unhealthy+directecho성공→file복구200/healthy/NLBecho→healthfile/TCPhealth원복→임시probe8080rules삭제→61앱/66NLB반환.
- **선택 이유/실패 경계:** health/traffic port독립;FailOpenfalse;앱보존consumer registry 필수.

| 입력/산출물 | 실제 값 출처 | 준비 시점 |
|---|---|---|
| `NLB_OCID` | creator의 실제 생성/조회 출력; 임의 OCID/IP 금지 | 사용 전 |
| `PROBE_CIDR` | creator의 실제 생성/조회 출력; 임의 OCID/IP 금지 | 사용 전 |
| `BACKEND_IP` | creator의 실제 생성/조회 출력; 임의 OCID/IP 금지 | 사용 전 |
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

[아래생성/변경단계의정확한Oracle task본문](https://docs.oracle.com/en-us/iaas/Content/NetworkLoadBalancer/HealthCheckPolicies/update-health-check-policy.htm) — Consolefield/CLI/API동작근거.

아래 단계는 Oracle task 본문을 근거로 작성한 lab recipe이며 공식 실행결과가 아니다. 각 단계의 근거는 끝의 정확한 task 링크를 따른다.

1. [Network Load Balancer](00_Network_Load_Balancer.md)의 A/B/C/D 완료 및 [61 앱 전용 A단계](../08_Load_Balancer/00_Load_Balancer.md#backend-app)의 같은 private-vm HTTP8080 /health 200/ok가 필요하다. 앱 owner61에 선택69/expiry를 등록하고 LB resource는 만들지 않는다. vm-a의 BACKEND_IP/32 TCP8080 egress 및 backend NSG/OS CLIENT_CIDR TCP8080 ingress도 직접404/200 대조 전에 승인/조회한다.
2. HTTP로 바꾸기 전에 exact NLB_OCID/echo9090의 원래 TCP health 전체값을 Console readback으로 저장하고 owner66/69 독립 복구 담당자·lease 내 deadline을 기록한다. 오류/세션 손실에는 health-file 복구→원래 TCP checker 전체값 복원/readback→new echo를 먼저 수행하고 이번 rules만 정리한다. **변경전** 아래 HTTP probe8080 허용과 guard station을먼저완료한다. frontendNLB NSG StatefulegressTCP8080→BACKEND_IP/32를추가/readback하고backendNSG/OSPROBE_CIDR8080ingress와client8080직접대조규칙도읽는다. 새ruleID는69소유다. 그뒤 echo9090 set health를 TCP9090에서 HTTP8080/path /health/status200으로 바꾼다. NLB의 health probe source가 NSG/OS firewall8080을 통과하는지 먼저 확인한다. listener TCP9090과 health8080이 독립임을 readback한다.
3. HTTP200/NLBhealthy/echo양성대조뒤 아래 bounded health-file outage block을backend별도터미널에서실행하여404를만들고 HTTP health unhealthy를 확인한다. 동시에 직접 TCP echo는 살아있음을 증명한다. 전용 단일 backend NLB 응답은 오류/timeout을 실제 기록한다.
4. boundedblock은이미파일을복구했다. `sudo test -f /var/www/oci-lab/health`와saved없음을확인하고,session중단으로saved가남았을때만위restore_health의ifguard로원복한다. 그뒤 HTTP200→healthy→NLB TCP payload 회복을 확인한다. 원래 TCP9090 health로 복원하고 저장값을 조회한다.

### bounded health-file outage 및 interruption 복구
아래는 **backend guest의 별도 터미널**에서 실행한다. 먼저HTTP200/ok를확인하고 파일move 후 network client/OCI관측터미널에서최대120초동안404/NLBhealth/directTCP양성대조를관찰한다. 관측완료시즉시Enter로복구하고, timeout/INT/TERM/EXIT도복구한다. 단독mv시험대신이block을사용한다.
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
curl --fail --max-time 10 http://127.0.0.1:8080/health
trap restore_health EXIT
trap 'exit 130' INT
trap 'exit 143' TERM
sudo mv -T -- "$HEALTH" "$SAVED"
read -r -t 120 -p 'observe from other terminals; Enter restores: ' answer || true
restore_health
trap - EXIT INT TERM
curl --fail --max-time 10 http://127.0.0.1:8080/health
```
터미널강제kill/powerloss는trap을보장하지않는다. owner69는120초/승인expiry 중 빠른 deadline 안에 재접속하여 위 HEALTH/SAVED 설정과 동일 restore_health 함수를 새 터미널에 다시 정의하고 실행한다. 전용 owner-only directory 및 원본 인수를 확인하며 saved는 regular/non-symlink source, health 목적지는 -e/-L 모두 부재여야 한다. collision은 덮어쓰지 않고 중단한다. 복원 뒤 direct200→NLBhealthy→newTCPpayload를수행하고120초deadlineowner가중단/복구한다.

### HTTP probe8080 허용과 guard
실제NLBHealthCheckResultsourceIP를PROBE_CIDR로읽고backendNSGingress8080source=그/32만추가한다. guestfirewalld도같은규칙:
```bash
set -euo pipefail
sudo systemctl is-active --quiet firewalld || { printf 'STOP: approved active firewalld required\n'; exit 1; }
# ip -br address로 primary VNIC IP와 인터페이스를 대조하여 입력한다.
BACKEND_IFACE='<actual-primary-VNIC-interface>'
ZONE=$(sudo firewall-cmd --get-zone-of-interface="$BACKEND_IFACE") || exit 1
case "$ZONE" in ''|'no zone') exit 1;; esac
sudo firewall-cmd --zone="$ZONE" --list-all || exit 1
LEDGER="$HOME/oci-lab69-firewall-0.tsv"
test ! -e "$HOME/oci-lab69-firewall-0.tsv" && test ! -L "$HOME/oci-lab69-firewall-0.tsv" || exit 1
(umask 077; set -o noclobber; : > "$HOME/oci-lab69-firewall-0.tsv") || exit 1
for cidr in "$PROBE_CIDR"; do
  rule="rule family=ipv4 source address=$cidr port port=8080 protocol=tcp accept"
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
firewalldactive승인전제이며inactive면승인OS규칙으로대체한다. healthcheckHTTP8080/path/health/status200/interval10000/timeout3000/retries3,Failopen=false를읽는다. backend 파일 이동/복원은 위 -e/-L guard와 regular/non-symlink source 검사가 있는 동일 bounded 함수만 사용한다. vm-a에서direct8080health404와directTCPecho성공을동시에기록한다. 원복mv→direct200→NLBhealthOK→NLBTCPpayload성공→originalTCPhealth9090복원이다.


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

TCP 앱 접속 가능하지만 HTTP health failed인 상태와 복구. health status만으로 모든 앱 요청 성공을 단정하지 않는다.

- 예상 pattern이며 실제 OCI 실행 결과가 아니다. timestamp, resource OCID, 설정 readback, 실제 HTTP/TCP/파일 결과를 실행자가 기록한다.

## 문제해결

TCP9090살아도HTTP8080health404면backend제외가의도. Failopen=true면실패예상이틀리므로false고정;probe8080rule필수.

실패기록에는같은target/credentials/region/요청시간을고정한다. 복구는원래설정으로실행하고actualresponse/readback둘다확인한다;예상값을실측으로쓰지않는다.

## 리소스정리

health파일복원/TCP9090originalhealth원복→NLBhealthOK/TCPpayload;위 owned guest firewall cleanup으로69 ledger의 runtime rule만 제거하고frontendegress8080와backendingress8080의69새ruleID삭제. 기존61규칙은유지.

정리실행자의실제UTC/OCID/readback을기록한다. 공유resource는owner확인전삭제금지; stop만으로bootvolume/LB/image비용종료가아니다.

## 공식문서

- [Oracle Linux 9 firewalld zone/runtime 설정](https://docs.oracle.com/en/operating-systems/oracle-linux/9/firewall/firewall-ConfiguringfirewalldZones.html) — 인터페이스별 zone과 temporary/runtime 설정 근거; query 오류/ownership ledger는 작성한 안전 gate다.

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 목표는 학습 후 설명·구분·확인할 내용이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [33] [Overview of Flexible Network Load Balancer — Health Check Policies](https://docs.oracle.com/en-us/iaas/Content/NetworkLoadBalancer/overview.htm)
  - 확인 범위: Health Check Policies.

### 절차별 공식 task body 추가 근거
- [overview.htm](https://docs.oracle.com/en-us/iaas/Content/NetworkLoadBalancer/overview.htm) — 본문 검토 근거; 아래 정확한 task와 작성한 입력을 구분한다. OCI 실행 증거가 아니다.
- [health-check-policy-management.htm](https://docs.oracle.com/en-us/iaas/Content/NetworkLoadBalancer/HealthCheckPolicies/health-check-policy-management.htm) — 본문 검토 근거; 아래 정확한 task와 작성한 입력을 구분한다. OCI 실행 증거가 아니다.



### 이번 절차의 정확한 공식 task
- [health-check-policy-management.htm](https://docs.oracle.com/en-us/iaas/Content/NetworkLoadBalancer/HealthCheckPolicies/health-check-policy-management.htm) — 이 문서 실습절차의 입력/동작을 대조한 본문.

- [정확한 NLB edit task](https://docs.oracle.com/en-us/iaas/Content/NetworkLoadBalancer/HealthCheckPolicies/update-health-check-policy.htm) — 보존/health변경입력과readback절차근거.
