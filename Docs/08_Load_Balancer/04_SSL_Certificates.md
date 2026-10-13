# SSL Certificates

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../README.md)를 확인하세요.

> 절차 작성본 / OCI 미실행 — 아래 예상값은 실측 결과가 아닙니다. 비용·권한·사전준비 gate를 통과한 전용 환경에서만 실행하세요.

## 학습목표

- 승인된 기존 PEM chain/key를 same LB에 import하고 HTTPS443 listener를 web8080에 연결한다.
- CA/SAN 검증을 켠 health200과 hostname mismatch를 대조한 뒤 정상 trust를 재확인하고 listener→lab certificate→새443규칙을 정리한다.

완료 기준은 아래 실제 생성·설정·요청 결과로 판정한다. 이 목표는 수행할 작업이며 OCI에서 관찰한 결과가 아니다.

## 대응 Roadmap 항목

- 항목명: **SSL Certificates**
- Roadmap 경로: **Load Balancer → SSL Certificates**
- Node ID: `service-lb-4`
- 상위 항목: **Load Balancer** (`service-lb-0`)
- 문서 범위: 이 항목의 개념과 작성된 실습절차. 통합 품질검토는 BLOCKED이며 실제 실행은 NOT_EXECUTED입니다.
- [전체 항목 목록](../README.md)

## 사전준비


61 same LB_OCID/LB_IP/LB_MODE/FRONTEND_CLIENT_CIDR/web8080와 조직 승인 SAN DOMAIN/PEM chain/key/trust를 받는다. vm-a TRUST_MODE/공개 CA_FILE checksum/CA_ARGS 준비와 lab-tls/lab-https443 이름·association 부재가 import 이전 조건이다. 인증서/private key는 사용자 secure UI에만 있다.

- [Load Balancer](00_Load_Balancer.md#실습절차) — 링크의 `실습절차`와 `결과검증` 완료; 실제 OCID/IP를 registry에 인계한다.


생성 전 `REGION`, `LAB_COMPARTMENT_ID`, 승인 종료시각 `EXPIRY_UTC`, 예산 상한과 실제 resource OCID를 실행 registry에 기록한다. owner가 후속 실습 종료까지 보존하며 이번 세션은 60분 이내 종료한다. 비용 승인/리전 capacity 없으면 실행 중단한다. shell의 변수를 다른 터미널에 자동 전달된다고 가정하지 말고 비밀값 없이 다시 설정한다.
- [Compute](../02_Compute/00_Compute.md#shared-launch)의 `vm-a` SSH/Oracle Linux 9 확인 및 [Private Subnets](../03_Virtual_Cloud_Network_VCN/03_Private_Subnets.md#실습절차)의 private-vm/jump 접속 완료. backend는 기존 private-vm을 재사용하고 pool 트랙만 owner58이 추가 승인 VM을 만든다. `KEY`, `PUBLIC_IP`(vm-a), `BACKEND_IP`(private-vm), `CLIENT_PRIVATE_IP`(vm-a), `CLIENT_CIDR=$CLIENT_PRIVATE_IP/32`, subnet/NSG OCID를 실제 readback하여 인계한다. 테스트는 vm-a에서 수행한다; Cloud Shell은 자동 VCN private client가 아니다.

## 권한

GetLoadBalancer/GetBackendSet/GetHealthChecker/ListWorkRequests → LOAD_BALANCER_READ. Create/Update/DeleteListener, Backend/BackendSet, UpdateHealthChecker 및 Create/DeleteCertificate → LOAD_BALANCER_UPDATE.

[API operation별 공식 permission table](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/lbpolicyreference.htm) — 가족 단위 manage를 최소권한이라 부르지 않는다. 전용 lab compartment의 group만 적용하고 정책작성 권한은 관리자 소유다.
```text
Allow group <identity-domain>/<lab-group> to manage load-balancers in compartment <lab-compartment> where any {request.permission='LOAD_BALANCER_INSPECT', request.permission='LOAD_BALANCER_READ', request.permission='LOAD_BALANCER_UPDATE'}
```
이 allowlist는 위 작업만 허용한다. Console 탐색 List API가 거부되면 실제 API permission을 공식 표에서 별도로 승인하며 all-resources로 확대하지 않는다.
네트워크 생성/규칙 편집은 별도 network owner: use subnets/use vnics/use network-security-groups와 read vcns; 규칙 생성/삭제는 NETWORK_SECURITY_GROUP_UPDATE_SECURITY_RULES를 해당 NSG로 한정한다. Compute network permission table의 VNIC_CREATE/SUBNET_ATTACH/NSG membership 의존성을 확인한다. 공유 VCN 전체 manage는 최소가 아니다.

## 비용및주의사항

sole61 LB에 기존 승인 PEM chain/key의 import와 HTTPS443 listener/narrow rules만 추가한다. 새 Certificates service/CA/Vault/LB/NLB/VM은 만들지 않는다. 기존 LB bandwidth/시간·TLS 요청/전송과 인증서 발급/보존 조건이 있으면 실제 조직 계약 기준으로 승인한다.443 public exposure는 기존80 health-only와 별도 승인이다. 개인키는 secure UI에서만 다루며 -k로 trust 오류를 우회하지 않는다. 종료는443 listener→미참조 lab certificate→이번 rules 순서이며61 HTTP80은 보존한다.

## 실습구성

- 작업 목표: 승인된 인증서로 동일 LB의 HTTPS를 구성하고 신뢰 검증한다.
- 생성 owner: 65. 동일 owner의 자원을 재사용하며 실제 결과 필드는 실행 전 비워 둔다.


### 이번 workshop 시나리오와 인수·반환

승인된 기존 인증서로 같은 LB의 TLS 종료와 SAN/trust 검증을 수행한다.

- **신규/변경:** lab-tls import,lab-https443 listener/443rules.
- **선행 재사용:** 61LB,조직승인chain/key/SAN/trustroot.
- **완주 흐름:** 승인/입력→본문의 생성/편집→정상CA/SAN200→mismatch TLS오류→정상200→443listener→lab-tls association없는certificate삭제→새rules삭제.
- **선택 이유/실패 경계:** -k금지;키는secureUI만;backendHTTP8080이고다른Certificates service 생성없음.

| 입력/산출물 | 실제 값 출처 | 준비 시점 |
|---|---|---|
| `DOMAIN` | 본문 단계에서 승인/선택한 값 | 사용 전 |
| `CA_FILE` | 본문 단계에서 승인/선택한 값 | 사용 전 |
| `LB_IP` | creator의 실제 생성/조회 출력; 임의 OCID/IP 금지 | 사용 전 |
| `LB_OCID` | creator의 실제 생성/조회 출력; 임의 OCID/IP 금지 | 사용 전 |

입력별 실행위치는 본문 터미널 구분을 따른다: OCI CLI=인증 Cloud Shell/승인 OCI 터미널, OS 앱/NFS/부하=지정 guest, private HTTP/TCP=vm-a guest, YAML 작성/SSH private key=본인 local. 조회/변경/정리의 관리자 승인은 **실행 조건**이며 이 문서는 IAM 성공을 입증하지 않는다. 실제 허용/거부와 capacity는 사용자 실행에서 확인한다. 미승인/NOT_SELECTED/ASYNC_PENDING을 성공으로 바꾸지 않는다.

### frontend source와 public 선택 gate
backend 직접 CLIENT_CIDR는 A_PRIVATE_IP/32다. LB_MODE=private이면 FRONTEND_CLIENT_CIDR=CLIENT_CIDR, LB_MODE=public이면 승인된 vm-a 실제 public egress IPv4/32를 FRONTEND_CLIENT_CIDR로61에서인계한다. frontend listener ingress는 이 mode별 source만 사용한다. public79 health-only HTTP 승인은8081/index확장을허용하지않는다;62 public extension은별도승인없으면NOT_SELECTED다.65의443은별도TLS/source/비용승인후진행한다. 기존mode를전환하거나두번째LB는만들지않는다.

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

[아래생성/변경단계의정확한Oracle task본문](https://docs.oracle.com/en-us/iaas/Content/Balance/Tasks/create_certificate.htm) — Consolefield/CLI/API동작근거.

아래 단계는 Oracle task 본문을 근거로 작성한 lab recipe이며 공식 실행결과가 아니다. 각 단계의 근거는 끝의 정확한 task 링크를 따른다.

1. [Load Balancer](00_Load_Balancer.md)의 A/B/C 완료와 동일 LB를 사용한다.
**단계1 필수 preflight — 이하 import/listener/rule 변경 전 vm-a 입력 gate:** 승인 SAN domain과 creator61 actual LB_IP를 vm-a에 별도로 지정한다. `TRUST_MODE=public`은 조직 승인 공인 CA 및 vm-a 기본 trust store로 검증하는 branch이고 CA_FILE을 전송하지 않는다. `TRUST_MODE=file`은 관리자가 승인한 **public root/chain PEM만** 안전한 조직 파일 전송 경로로 vm-a의 새 owned regular/non-symlink/readable absolute CA_FILE에 전달하고 공개 checksum을 out-of-band 대조한다. private key/passphrase/leaf secret는 vm-a로 전송하지 않고 secure Console import만 한다. 아래 preflight 성공을 registry에 기록하기 전2를 시작하지 않는다.

```bash
# vm-a only, before any Console certificate/listener/rule mutation
export DOMAIN='<approved_exact_SAN_domain>'
export LB_IP='<owner61_actual_LB_IPv4>'
export TRUST_MODE='<public_or_file>'
export CA_FILE='<file_branch_actual_absolute_public_CA_path>'
python3 - <<'PY'
import ipaddress,os,re,pathlib,stat
d=os.environ['DOMAIN']; ip=ipaddress.ip_address(os.environ['LB_IP'])
assert ip.version==4
assert len(d)<=253 and '.' in d and d!='mismatch.invalid'
assert all(re.fullmatch(r'[A-Za-z0-9](?:[A-Za-z0-9-]{0,61}[A-Za-z0-9])?',x) for x in d.split('.')), 'STOP: literal approved DNS SAN only'
mode=os.environ['TRUST_MODE']; assert mode in ('public','file')
if mode=='file':
    p=pathlib.Path(os.environ['CA_FILE']); assert p.is_absolute() and '<' not in str(p)
    assert all(not x.is_symlink() for x in [p,*p.parents]), 'STOP: CA path symlink'
    s=p.stat(); assert stat.S_ISREG(s.st_mode) and s.st_uid==os.getuid() and os.access(p,os.R_OK)
    assert b'-----BEGIN CERTIFICATE-----' in p.read_bytes()
    assert b'PRIVATE KEY' not in p.read_bytes(), 'STOP: public CA only'
PY
if [ "$?" -ne 0 ]; then exit 1; fi
CA_ARGS=()
case "$TRUST_MODE" in public) ;; file) CA_ARGS=(--cacert "$CA_FILE");; *) exit 1;; esac
```

**동일 LB ownership gate (Console 관리자,2 이전):** creator61 LB_OCID/Region을 다시 읽고 Certificates and ciphers의 전체 목록과 Listeners에서 `lab-tls`, `lab-https443` 이름 모두 absent를 확인한다. 기존 항목/같은443 listener/동시 writer가 있으면 수정·삭제·재import하지 않고 중단한다. 독점 owner65 lease/expiry와 before 목록을 registry에 저장한다. 성공한 생성마다 `(LB_OCID, exact certificate-name, CREATED_BY65=1, event UTC)` 및 `(LB_OCID, exact listener-name, CREATED_BY65=1, certificate association, backendset, port)`를 readback해 기록한다(이 API의 certificate/listener는 LB scoped name으로 식별; 신규 OCID를 발명하지 않는다). 부분 성공도 기록하여 실패 후 해당 항목만 정리한다. network owner는 기존 동일443 rule을 재사용만 하고 owns=0, 새로 추가한 exact rule ID만 owns=1로 기록한다. 생성 수락/이름 alone은 소유권 증거가 아니다.

조직 승인 certificate chain/private key 및 SAN에 맞는 DNS 이름, trust root를 안전한 UI/비밀관리 경로로 준비한다. 없으면 실행 금지이며 임의 발급/secret 생성은 하지 않는다.
2. Load balancer managed certificate / Import PEM을 선택한다(Certificates service 새 리소스와 혼동하지 않는다). Name=lab-tls, SSL certificate=leaf PEM, CA certificate=승인 chain, private key=matching PEM, passphrase=필요시 secure UI만 사용. Certificates → Add Certificate에서 승인된 leaf/chain/private key를 보안 UI로 import한다. 키 내용은 캡처/터미널 기록/문서에 쓰지 않는다. 인증서 이름/expiry/SAN만 기록한다.
3. 해당 LB → Listeners → Create listener: Name=lab-https443, Protocol=HTTPS, Port=443, Default backend set=web8080, Certificate resource=Load balancer managed certificate/lab-tls, Hostname=미선택, Idle timeout=60s, SSL protocols=조직 승인 TLS1.2 이상, Cipher suite=승인 suite를 선택하고 Create/Work Request SUCCEEDED를 읽는다. TLS termination 이후 backend HTTP8080임을 확인한다. backend TLS가 필수인 조직은 승인된 backend 인증서 검증을 별도 구성하고 HTTP를 허용하지 않는다.
4. frontend NSG443을 client CIDR만 허용한다. `curl --resolve '<approved-domain>:443:<LB-IP>' --cacert '<approved-root-ca-path>' -i https://<approved-domain>/health`로 신뢰/hostname/200을 확인한다. 공인 trust이면 --cacert는 생략한다. `-k`는 합격검증으로 금지한다.
5. 만료/hostname mismatch 오류는 expected negative test로 별도 기록하고 정상 DNS/SAN으로 다시 성공해야 한다.

### hostname 실패와 정상trust복구
frontendNSG443및vm-aegress443→LB_IP/32를추가한다. Listenername=lab-https443/protocolHTTPS/port443/certificateresource=Loadbalancermanagedcertificate lab-tls/backendsetweb8080/hostnameoptionalnone/idle60s/approvedTLS1.2+를선택한다. trustroot경로/DOMAIN은인증서SAN과일치해야한다.
```bash
# vm-a의 전용 Bash 세션: 기존 경로/깨진 symlink면 첫 curl 전에 중단한다.
set -euo pipefail
umask 077
TLS_WORK_DIR="$HOME/oci-roadmap-tls65"
if test -e "$TLS_WORK_DIR" || test -L "$TLS_WORK_DIR"; then
  printf '%s\n' 'STOP: dedicated TLS output path occupied; preserve it' >&2
  exit 1
fi
mkdir -m 700 -- "$TLS_WORK_DIR" || exit 1
cleanup_tls_outputs() {
  rm -f -- "$TLS_WORK_DIR/tls-health-before.txt" "$TLS_WORK_DIR/mismatch-tls-error.txt" "$TLS_WORK_DIR/tls-health-after.txt"
  rmdir -- "$TLS_WORK_DIR"
}
trap cleanup_tls_outputs EXIT
trap 'exit 129' HUP
trap 'exit 130' INT
trap 'exit 143' TERM
# freshly created private directory owns only these three outputs; no CA/key copy.
# Same vm-a shell: keep preflight DOMAIN/LB_IP/TRUST_MODE/CA_ARGS; never reset placeholders here.
declare -p CA_ARGS >/dev/null || exit 1
status=$(curl --noproxy '*' --connect-timeout 3 --max-time 10 --resolve "$DOMAIN:443:$LB_IP" "${CA_ARGS[@]}" -sS -o "$TLS_WORK_DIR/tls-health-before.txt" -w '%{http_code}' "https://$DOMAIN/health") || exit 1
test "$status" = 200 || exit 1
printf 'ok\n' | cmp - "$TLS_WORK_DIR/tls-health-before.txt" || exit 1
set +e
curl --noproxy '*' --connect-timeout 3 --max-time 10 --resolve "mismatch.invalid:443:$LB_IP" "${CA_ARGS[@]}" -i https://mismatch.invalid/health 2> "$TLS_WORK_DIR/mismatch-tls-error.txt"
rc=$?
set -e
test "$rc" -eq 60 || { printf 'not a certificate validation failure: rc=%s\n' "$rc"; exit 1; }
# curl60 also covers other certificate errors: require an actual hostname/SAN mismatch diagnostic.
grep -Ei 'subject.*(match|name)|alternative.*(match|name)|hostname.*(match|mismatch)' "$TLS_WORK_DIR/mismatch-tls-error.txt" || { printf 'not proven hostname mismatch; inspect actual diagnostic\n'; exit 1; }
status=$(curl --noproxy '*' --connect-timeout 3 --max-time 10 --resolve "$DOMAIN:443:$LB_IP" "${CA_ARGS[@]}" -sS -o "$TLS_WORK_DIR/tls-health-after.txt" -w '%{http_code}' "https://$DOMAIN/health") || exit 1
test "$status" = 200 || exit 1
printf 'ok\n' | cmp - "$TLS_WORK_DIR/tls-health-after.txt" || exit 1
# 필요한 비밀 없는 status/rc/diagnostic을 실행 registry에 기록한 뒤 exact local files만 정리.
cleanup_tls_outputs || exit 1
trap - EXIT HUP INT TERM
```
승인된cert없이실행불가이며실제blocker다. -k로우회하지않는다.

## 결과검증

정상 CA/SAN 검증과 /health 200. TLS handshake 실패를 앱 장애로 오해하지 않는다.

- 예상 pattern이며 실제 OCI 실행 결과가 아니다. timestamp, resource OCID, 설정 readback, 실제 HTTP/TCP/파일 결과를 실행자가 기록한다.

## 문제해결

curlcertificate/hostname오류는TLSchain/SAN/trustroot점검. -k합격금지;443접속timeout은frontend규칙문제로chain오류와다르다.

실패기록에는같은target/credentials/region/요청시간을고정한다. 복구는원래설정으로실행하고actualresponse/readback둘다확인한다;예상값을실측으로쓰지않는다.

## 리소스정리

**owner65 exact cleanup:** registry의 CREATED_BY65=1와 같은 LB_OCID/name/event/current association을 확인한 listener만 Delete하고 새 Listeners 목록에서 부재 readback한다. 해당 listener 제거와 **다른 listener association 없음**을 확인한 CREATED_BY65=1 certificate lab-tls만 Delete/readback한다. 전용 frontend443/vm-a443는 owns=1의 exact 새 rule ID만 제거/readback한다. 기존 같은 이름/owns=0/pre-existing rule/불명확한 association은 삭제하지 않는다. 일부 생성 실패도 기록된 owned 항목만 이 순서로 정리하며 공유61 LB는 보존한다. 조직CA/key원본보존. vm-a의 위 전용 세션은 성공·오류·signal exit에서 자신이 새로 만든 TLS_WORK_DIR의 세 파일만 제거하고 빈 directory를 rmdir한다. 기존 경로는 guard에서 중단하여 정리 대상으로 삼지 않는다. 세션 강제종료(SIGKILL 등)로 남았다면 owner가 exact directory/세 파일 소유권을 확인한 후 같은 cleanup 함수만 실행한다; glob/recursive 삭제와 CA/key 삭제는 금지한다.

정리실행자의실제UTC/OCID/readback을기록한다. 공유resource는owner확인전삭제금지; stop만으로bootvolume/LB/image비용종료가아니다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 목표는 학습 후 설명·구분·확인할 내용이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [5] [Load Balancer- Managed SSL Certificates — SSL Certificates](https://docs.oracle.com/en-us/iaas/Content/Balance/Tasks/managingcertificates.htm)
  - 확인 범위: SSL Certificates.

### 절차별 공식 task body 추가 근거
- [managingcertificates.htm](https://docs.oracle.com/en-us/iaas/Content/Balance/Tasks/managingcertificates.htm) — 본문 검토 근거; 아래 정확한 task와 작성한 입력을 구분한다. OCI 실행 증거가 아니다.



### 이번 절차의 정확한 공식 task
- [create_certificate.htm](https://docs.oracle.com/en-us/iaas/Content/Balance/Tasks/create_certificate.htm) — 이 문서 실습절차의 입력/동작을 대조한 본문.
- [managinglisteners_topic-Creating_Listeners.htm](https://docs.oracle.com/en-us/iaas/Content/Balance/Tasks/managinglisteners_topic-Creating_Listeners.htm) — 이 문서 실습절차의 입력/동작을 대조한 본문.
