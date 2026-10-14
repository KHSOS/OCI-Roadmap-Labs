> **기존 실습 초안 — 미검증**: 원래 93개 문서의 절차를 보존한 자료입니다. 새 튜토리얼의 품질 승인이나 OCI 실행 검증을 뜻하지 않습니다.

# Instance Configurations

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../../../Docs/README.md)를 확인하세요.

> 절차 작성본 / OCI 미실행 — 아래 예상값은 실측 결과가 아닙니다. 비용·권한·사전준비 gate를 통과한 전용 환경에서만 실행하세요.

## 학습목표

- platform install 또는 custom verify-only user-data 중 하나를 준비하여 lab-web-config와 exact image/shape/key/network를 저장한다.
- configuration-ready readback을 58 launch에 인계하고 pool 종료 뒤 configuration 및 이번 준비 규칙을 소유자 순서로 정리한다.

완료 기준은 아래 실제 생성·설정·요청 결과로 판정한다. 이 목표는 수행할 작업이며 OCI에서 관찰한 결과가 아니다.

## 대응 Roadmap 항목

- 항목명: **Instance Configurations**
- Roadmap 경로: **Autoscaling → Instance Configurations**
- Node ID: `sequence-7-2`
- 상위 항목: **Autoscaling** (`sequence-7-0`)
- 문서 범위: 이 항목의 개념과 작성된 실습절차. 통합 품질검토는 BLOCKED이며 실제 실행은 NOT_EXECUTED입니다.
- [전체 항목 목록](../../../Docs/README.md)

## 사전준비


10 vm-a의 실제 A_PRIVATE_IP/SSH public key,21 private-app/route/SL OCID와 승인 image/shape/OCPU/RAM/AD를 받는다. platform branch는22 기존 B 생성 없이30 승인 bootstrap NAT와 nsg-pool-app를 준비한다. custom branch만56 AVAILABLE IMAGE_ID/source STOPPED를 받는다. 생성한 CONFIG_ID/YAML hash/NSG/rules를58에 인계한다.

- [Compute](../02_Compute/00_Compute.md#실습절차) — 링크의 `실습절차`와 `결과검증` 완료; 실제 OCID/IP를 registry에 인계한다.


생성 전 `REGION`, `LAB_COMPARTMENT_ID`, 승인 종료시각 `EXPIRY_UTC`, 예산 상한과 실제 resource OCID를 실행 registry에 기록한다. owner가 후속 실습 종료까지 보존하며 이번 세션은 60분 이내 종료한다. 비용 승인/리전 capacity 없으면 실행 중단한다. shell의 변수를 다른 터미널에 자동 전달된다고 가정하지 말고 비밀값 없이 다시 설정한다.
- [Compute](../02_Compute/00_Compute.md#shared-launch)의 `vm-a` SSH/Oracle Linux 9 확인 및 [Private Subnets](../../Networking/Original-Provisional/03_Virtual_Cloud_Network_VCN/03_Private_Subnets.md#실습절차)의 subnet/SSH preparation만 완료; 기존private-vm 생성은pool트랙필수아니다. owner58만승인pool VM1→2대를생성한다. `KEY`, `PUBLIC_IP=A_PUBLIC_IP`, `CLIENT_PRIVATE_IP=A_PRIVATE_IP`, `CLIENT_CIDR=$CLIENT_PRIVATE_IP/32`, private subnet/NSG OCID만 생성 전에 인계한다. `POOL_ID`와 member IP는 owner58의 생성 후 실제 출력이다. 기본 platform 트랙은 `BACKEND_IP`/기존 private-vm을 요구하지 않는다. 검증은 vm-a guest에서 수행하며 Cloud Shell은 VCN private client가 아니다.

## 권한

[corepolicyreference — Permissions Required for Each API Operation / Autoscaling API Operations](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm) 실제 본문 기준이다. 선행 owner의 생성/정리 권한을 현재 사용자에게 자동 부여하지 않는다. 관리자가 exact operation과 실제 resource compartment를 대조한다.

### 이문서 operation별 승인 allowlist
```text
Allow group <identity-domain>/<lab-group> to manage instance-configurations in compartment <lab-compartment> where any {request.permission='INSTANCE_CONFIGURATION_READ', request.permission='INSTANCE_CONFIGURATION_CREATE', request.permission='INSTANCE_CONFIGURATION_UPDATE', request.permission='INSTANCE_CONFIGURATION_INSPECT', request.permission='INSTANCE_CONFIGURATION_DELETE'}
Allow group <identity-domain>/<lab-group> to manage instance-images in compartment <lab-compartment> where any {request.permission='INSTANCE_IMAGE_READ'}
Allow group <identity-domain>/<lab-group> to manage instances in compartment <lab-compartment> where any {request.permission='INSTANCE_READ'}
```
이목록은문서APItable기반의administrator검토용이며실제tenancy허용/거부시험미실행이다. IMAGE_READ는CreateInstancePoolAPI표의표기이고instance-imageverb표는INSTANCE_IMAGE_READ를쓰므로둘을임의동일하다고정규화하지않는다. bootvolume생성/termination및NSGattachment의서비스처리와실제authorization은corepolicy표/실제WorkRequest로확인한다. 후속owner생성권한을현재사용자에게자동부여하지않는다.

이실습은 Provide a list of settings / CreateInstanceConfigurationDetails이며 INSTANCE_CONFIGURATION_CREATE를사용한다. Existing instance 복사 방식으로 **변경하는 경우에만** CreateInstanceConfigurationFromInstanceDetails의 INSTANCE_READ/VNIC_READ/VNIC_ATTACHMENT_READ/VOLUME_INSPECT/VOLUME_ATTACHMENT_INSPECT를 추가승인한다. 관리자에게 아래 instance-copy 의존성만 승인한다. launch details는기존config에서임의수정하지않고새config로생성한다. 위UPDATE는config metadata/display 설정에한정되므로이번절차에필요없다면allowlist에서제외한다. 다음4정책은기본방식에적용하지않는다.
```text
Allow group <identity-domain>/<lab-group> to inspect vnic-attachments in compartment <lab-compartment>
Allow group <identity-domain>/<lab-group> to inspect vnics in compartment <lab-compartment>
Allow group <identity-domain>/<lab-group> to inspect volumes in compartment <lab-compartment>
Allow group <identity-domain>/<lab-group> to inspect volume-attachments in compartment <lab-compartment>
```

## 비용및주의사항

이번 생성은 instance configuration 하나와 user-data/승인 network rules이며 configuration 저장 자체를 VM 실행으로 계산하지 않는다. 실제 Compute/boot 과금은 owner58 pool launch 이후다. platform bootstrap이면 승인 NAT/package 경로, custom branch이면 owner56 image 저장/호환 shape 및 verify-only 입력을 확인한다. 별도 template VM/restore VM/LB는 만들지 않는다.58pool 및55/60 controller 반환 후 configuration/이번 준비 rule만 정리한다.

## 실습구성

- 작업 목표: 실제 image/shape/network/user-data를 담은 instance configuration을 생성한다.
- 생성 owner: 57. 동일 owner의 자원을 재사용하며 실제 결과 필드는 실행 전 비워 둔다.


### 이번 workshop 시나리오와 인수·반환

launch 설정과 자동 app bootstrap을 저장하는 configuration 하나를 만든다.

- **신규/변경:** instance configuration1개,YAML metadata.
- **선행 재사용:** 10 vm-a/key,21 private-app;선택56 capture-ready.
- **완주 흐름:** 승인/입력→본문의 생성/편집→configuration-ready readback; 앱은58 member 생성후검증→55/60삭제→58pool종료→57config삭제.
- **선택 이유/실패 경계:** platform install YAML과custom verify-only YAML을 구분; 설정은 VM 아니다.

| 입력/산출물 | 실제 값 출처 | 준비 시점 |
|---|---|---|
| `CONFIG_ID` | creator의 실제 생성/조회 출력; 임의 OCID/IP 금지 | 생성 후 |
| `IMAGE_ID` | creator의 실제 생성/조회 출력; 임의 OCID/IP 금지 | 사용 전 |
| `SHAPE` | 본문 단계에서 승인/선택한 값 | 사용 전 |
| `CLIENT_CIDR` | creator의 실제 생성/조회 출력; 임의 OCID/IP 금지 | 사용 전 |
| `LB_SUBNET_CIDR` | creator의 실제 생성/조회 출력; 임의 OCID/IP 금지 | 사용 전 |

입력별 실행위치는 본문 터미널 구분을 따른다: OCI CLI=인증 Cloud Shell/승인 OCI 터미널, OS 앱/NFS/부하=지정 guest, private HTTP/TCP=vm-a guest, YAML 작성/SSH private key=본인 local. 조회/변경/정리의 관리자 승인은 **실행 조건**이며 이 문서는 IAM 성공을 입증하지 않는다. 실제 허용/거부와 capacity는 사용자 실행에서 확인한다. 미승인/NOT_SELECTED/ASYNC_PENDING을 성공으로 바꾸지 않는다.

### Private pool CPU metric 전달 gate
[Enabling Monitoring 공식 본문](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/enablingmonitoring.htm)은 public IP 없는 instance에 Service Gateway 준비를 요구한다. metric55/59 선택이면 network owner31은 기존 `sgw-lab`를 조회/재사용하거나 같은 VCN에 하나만 Create하며 service label **All <actual Region> Services in Oracle Services Network**를 선택한다. private-app 실제 route table에 Destination Service=같은 All-services label, Target=SGW_OCID를 추가하고, nsg-pool-app Stateful egress TCP443 destination type=Service CIDR/동일 label을 승인한다. repository NAT default route와 더 구체적인 service route를 구분한다. SGW-only 실험 전체/기존 B 생성은 prerequisite가 아니다. configuration의 Oracle Cloud Agent에서 Compute Instance Monitoring=Enabled/Running, plugins enabled를 설정하고 guest/Console의 최근 `oci_computeagent` CPU datapoint를 읽는다. 관리자 승인/실제 datapoint 없는 metric 실험은 pending이다. 31 owner registry에 pool/metric consumer와expiry를등록하고 마지막 consumer 반환 뒤 새 route/rule ID를제거→미참조 gateway만삭제한다. publicIP를pool에추가하여우회하지않는다.

## 실습절차

### 작업 순서 체크포인트
network station의 실제 NSG/subnet/image output을 확보한 다음 configuration fields/user-data 선택→단일 Create/readback→58 actual member validation 순서다. 단계 실패는 다음 생성으로 넘기지 않고 같은 target의 복구/반환으로 처리한다.

### 실행 터미널과 지역 고정
CLI는 Cloud Shell/본인 OCI 인증 터미널에서만, `sudo`/파일 변경은 명시한 guest 터미널에서만 실행한다. 네트워크 CIDR은 기존 probe-a/probe-b 또는 10.x 예시에서 복사하지 않고 실제 VNIC/subnet 값을 조회한다.
```bash
export REGION="<approved-region>"
export OCI_CLI_REGION="$REGION"
export LAB_COMPARTMENT_ID="<actual-lab-compartment-ocid>"
export EXPIRY_UTC="<approved-session-end-UTC>"
```
치환 전 실행 금지. 생성된 OCID/IP/원래 설정/새 rule ID/owner/종료시각을 비밀 없는 registry에 기록하고 다음 터미널에서는 필요한 변수만 다시 설정한다.

[아래생성/변경단계의정확한Oracle task본문](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/creatinginstanceconfig.htm) — Consolefield/CLI/API동작근거.

아래 단계는 Oracle task 본문을 근거로 작성한 lab recipe이며 공식 실행결과가 아니다. 각 단계의 근거는 끝의 정확한 task 링크를 따른다.

1. [Compute](../02_Compute/00_Compute.md#baseline-10-ready)의 실제 vm-a client와 [SSH Keys](../02_Compute/05_SSH_Keys.md#key-preparation)의 공개키 준비, image/shape 선택과 [Subnets](../../Networking/Original-Provisional/03_Virtual_Cloud_Network_VCN/02_Subnets.md#baseline-21-ready)의 private subnet 준비와 [Private Subnets의 jump egress 절차](../../Networking/Original-Provisional/03_Virtual_Cloud_Network_VCN/03_Private_Subnets.md#실습절차)를 읽는다. 기본 platform 트랙에서 기존 private-vm 생성/접속은 요구하지 않는다. 이미지 선택은 platform Oracle Linux를 기본으로 하며 custom image는 [56 image-capture-ready](01_Images.md#image-capture-ready)의 actual AVAILABLE IMAGE_ID와 source stop 반환 때만 사용한다(restore완료선행금지).

### app-bootstrap

58의 [Global running-VM 비용 gate](03_Instance_Pools.md#global-vm-gate)를 첫 launch 전에 수행한다. custom source는 owner22 Console Stop/STOPPED readback이 필수이며55 pool-ready까지 미루지 않는다.

### Platform pool 생성 전 networking 준비 — 기존 B 생성 없음

이 단계는 [21 Subnets](../../Networking/Original-Provisional/03_Virtual_Cloud_Network_VCN/02_Subnets.md#baseline-21-ready)의 실제 private-app/rt-private/sl-private와 [10 vm-a](../02_Compute/00_Compute.md#baseline-10-ready)의 A_PRIVATE_IP만 요구한다. 22의 VM Create/baseline-22-ready는 platform 트랙 prerequisite가 아니다. network owner가 다음을 수행한다.

1. VCN → NSG → Create `nsg-pool-app`를 lab-root에 한 개 생성한다. Stateful ingress TCP22 및 TCP8080의 source는 A_PRIVATE_IP/32, source port All이다. LB 미선택이면 LB source 규칙은 없다. NSG OCID와 새 rule ID를 owner57에 전달하고 configuration의 nsgIds로 선택한다. sl-private의 broad ingress/egress 잔존을 읽는다. 새 private-vm은 생성하지 않는다.
2. [30 NAT Gateway](../../Networking/Original-Provisional/03_Virtual_Cloud_Network_VCN/11_NAT_Gateway.md#nat-installation-preparation)의 **network 생성 동작만** 사용한다: VCN → NAT Gateways → `nat-lab`의 기존 OCID를 조회/재사용하거나 owner30 승인으로 하나를 Create(Block traffic=false)한다. `rt-private` 원래 배열을 저장하고 destination0.0.0.0/0 target=NAT_OCID rule을 승인 추가한다. 이 pool-only 단계는 NAT 문서의 B/22-ready prerequisite와 B workload를 실행하지 않는다.
3. nsg-pool-app Stateful egress TCP80/443은 승인된 repository destination 범위(동적 주소 때문에0.0.0.0/0가 필요하면 별도 outbound 승인)에만 허용한다. 실제 guest resolver/DHCP DNS와 필요한 TCP/UDP53 경로는 network owner가 확인한다. 새 member 생성 후 DNS/dnf/cloud-init 상태를 실제 읽으며 service-agent CPU metric egress도 동일 승인 범위에서 확인한다.
4. NAT owner30/network owner57의 registry에 선택58/55/59/60의 launch 소비자, budget/expiry를 등록한다. **모든 platform scale-out boot가 끝날 때까지 route/NSG repository egress를 유지**한다. 이전30 installation-only의 즉시 route제거를 그대로 적용하지 않는다. expiry 도달 시 controller disable/시험중단 후 정리한다. 55/60 controller삭제→58 member/boot/rules정리→57 config삭제 후 network owner57가 자신의 nsg-pool-app 새 rules/NSG를 제거하고 owner30은 원래route를복구/미참조 NAT만삭제한다. 다른 승인 NAT consumer가 있으면 동일 gateway를 보존한다.


2. Console → Compute → Instance Configurations → Create를 연다. lab Compartment, 이름 `lab-web-config`, 승인된 smallest available shape, image OCID, backend NSG, 실제 subnet을 지정한다. Flex shape는 CPU/RAM 값을 명시해 비용을 제한한다.
3. SSH 공개키만 입력한다. Advanced options → Management → Initialization script에 아래 YAML을 원문으로 업로드한다. instance compartment=lab-root, placement=승인 AD/FD, On-demand, Oracle Linux9 image architecture와 shape 일치, Flex OCPU/RAM=승인 최소값, existing shared-vcn/private-app, public IPv4=None, backend NSG, boot default, additional volume=None를 선택한다.

### 자동 app bootstrap — Oracle Linux 9 전용
아래 YAML을 `lab-pool.yaml`로 저장하고 CIDR 두 개를 실제 값으로 바꾼다. 기존 `vm-a`는 public-admin의 Oracle Linux 9 / opc이며 Ubuntu probe-a 또는 10.0.x.x 예시와 무관하다. pool VM은 기존 `private-app`에 public IP 없이 생성한다. `LB_SUBNET_CIDR`는 선택 LB가 있으면 실제 frontend subnet CIDR이고, LB 미선택 pool-only에서는 기존 private-app CIDR을 승인된 app-client 경계로 기록한다(LB 생성/인계를 요구하지 않는다). `CLIENT_CIDR`는 vm-a **private IP/32**이다. private-app의 실제 CIDR는 Subnets의 readback으로 얻고 임의 10.0.x.x를 사용하지 않는다.
패키지 설치 전에 위 **Platform pool 생성 전 networking 준비**의 owner30 NAT route 및 nsg-pool-app repository TCP80/443 egress를 완료한다. 기존 B/22-ready 또는30 B workload 완료를 요구하지 않는다. OCI resolver/DNS 접근도 확인한다. 기존 최소 SSH-only 구성으로 dnf가 성공한다고 가정하지 않는다. NSG ingress8080은 frontend CIDR 및 client /32만 승인하고 frontend egress8080→backend CIDR도 추가한다. SSH22는 기존 vm-a /32 그대로 유지한다.
본인 로컬 승인 lab 디렉터리에서 아래 전체 파일을 저장한다. CIDR token을 치환한 뒤 내용/hash를 launch metadata에 연결한다.
```bash
test ! -e lab-pool.yaml && test ! -L lab-pool.yaml || { printf 'existing lab-pool.yaml: inspect instead of overwrite\n'; exit 1; }
cat > lab-pool.yaml <<'CLOUD'
#cloud-config
packages:
  - nginx
  - policycoreutils-python-utils
write_files:
  - path: /etc/nginx/conf.d/oci-lab.conf
    permissions: '0644'
    content: |
      server {
          listen 8080;
          server_name _;
          root /var/www/oci-lab;
          location = / { default_type text/plain; try_files /index.txt =404; }
          location = /health { default_type text/plain; try_files /health =404; }
      }
  - path: /usr/local/sbin/oci-lab-bootstrap
    permissions: '0755'
    content: |
      #!/bin/bash
      set -euo pipefail
      for cidr in REPLACE_FRONTEND_CIDR REPLACE_CLIENT_PRIVATE_IP/32; do
          [[ $cidr != REPLACE* ]] || { printf 'replace approved CIDRs first\n'; exit 1; }
      done
      install -d -m 0755 /var/www/oci-lab
      printf 'backend=%s\n' "$(hostname)" > /var/www/oci-lab/index.txt
      printf 'ok\n' > /var/www/oci-lab/health
      restorecon -RF /var/www/oci-lab
      # Oracle Linux can label 8080 as http_cache_port_t; only this lab port is relabeled.
      if ! semanage port -l | grep '^http_port_t[[:space:]]' | grep -qw 8080; then
          if semanage port -l | grep -E '^http_cache_port_t[[:space:]]' | grep -qw 8080; then
              semanage port -m -t http_port_t -p tcp 8080
          else
              semanage port -a -t http_port_t -p tcp 8080
          fi
      fi
      nginx -t
      systemctl enable --now nginx
      systemctl reload nginx
      set -euo pipefail
      systemctl is-active --quiet firewalld || { printf 'STOP: approved active firewalld required\n'; exit 1; }
      # ip -br address로 primary VNIC IP와 인터페이스를 대조하여 입력한다.
      BACKEND_IFACE=$(ip -4 route show default | awk 'NR==1 {print $5}') || exit 1
      test -n "$BACKEND_IFACE" || exit 1
      ip -4 address show dev "$BACKEND_IFACE" || exit 1
      ZONE=$(firewall-cmd --get-zone-of-interface="$BACKEND_IFACE") || exit 1
      case "$ZONE" in ''|'no zone') exit 1;; esac
      firewall-cmd --zone="$ZONE" --list-all || exit 1
      LEDGER='/var/lib/oci-lab57-firewall.tsv'
      test ! -e "/var/lib/oci-lab57-firewall.tsv" && test ! -L "/var/lib/oci-lab57-firewall.tsv" || exit 1
      (umask 077; set -o noclobber; : > "/var/lib/oci-lab57-firewall.tsv") || exit 1
      for cidr in REPLACE_FRONTEND_CIDR REPLACE_CLIENT_PRIVATE_IP/32; do
        rule="rule family=ipv4 source address=$cidr port port=8080 protocol=tcp accept"
        absent=false
      runtime_before=unknown
      permanent_before=unknown
        for variant in runtime permanent; do
          opts=(); [ "$variant" != permanent ] || opts+=(--permanent)
          if firewall-cmd --zone="$ZONE" "${opts[@]}" --query-rich-rule="$rule"; then rc=0; else rc=$?; fi
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
          firewall-cmd --zone="$ZONE" --add-rich-rule="$rule" || exit 1
        fi
        firewall-cmd --zone="$ZONE" --query-rich-rule="$rule" || exit 1
      done
      curl --fail --connect-timeout 3 --max-time 10 http://127.0.0.1:8080/health
runcmd:
  - [/usr/local/sbin/oci-lab-bootstrap]
CLOUD
```
이 platform bootstrap은 runtime-only owned firewall rules를 사용하며 permanent 규칙을 변경하거나 reload하지 않는다. primary default-route interface의 zone/query가 실패하면 bootstrap은 중단한다. 재부팅/reload 뒤에는58 검증을 중단하고 owner57이 같은 실제 zone/source 규칙을 별도 새 ledger로 승인 재적용한다; 기존 ledger 충돌을 무시하지 않는다. pool 종료 시 guest 자체를 종료하며 다른 guest/기존 source firewall은 삭제하지 않는다.

새 pool VM마다 cloud-init이 payload/health/hostname/firewall을 **자동** 구성한다. manual SSH 설치가 scale-out 완료 조건이 아니다. 사용자 데이터에 private key/token은 넣지 않는다. platform image 최초 boot에 적용하고 Console은 YAML 원문을 업로드한다. **custom image 복원 증거에서는 이 설치 YAML을 사용하지 않는다.** 56 image에 이미 nginx/conf/health 및 SELinux 설정이 들어있어야 한다. source의 runtime-only firewall은 image에 보존된다고 주장하지 않는다. 새 custom member의 실제 primary interface/active zone과 runtime/permanent 규칙은58 member network station에서61의 checked runtime ownership recipe로 승인/추가/readback한 뒤 vm-a HTTP를 시험한다. 앱 재설치는 금지하며 이 네트워크 재적용은 이미지 앱 복원과 구분한다. custom branch user-data는 아래 검증/hostname만이며기존앱을재설치하지않는다. firewalld inactive이면 방화벽을 임의 시작하지 말고 승인된 OS 방화벽 규칙을 준비한다.


### configuration-ready

4. **Create 전 review:** `lab-web-config`, configuration Compartment와 **Compartment to create instances in**=승인 lab-root, image/architecture/shape/OCPU/RAM, 선택한 YAML 원문(install 또는 verify-only 하나), SSH public key, nsg-pool-app OCID/public IP None/boot default/no data volume를 다시 대조한다. placeholder가 남으면 Submit 금지다. Review → Create를 한 번 제출하고 OCID를 기록한다. **configuration-ready**: 생성 후 configuration 상세의 image/shape/launch options/metadata를 readback한다. 이 단계만58의생성 prerequisite이며 member 검증은58 pool-ready 이후 수행한다. [Instance Pools](../../../Docs/07_Autoscaling/03_Instance_Pools.md)에서 이 OCID로 VM을 실행해야 완료이다.

### pool 배치 override와 선택 이유

공식 Creating an Instance Configuration 본문은 pool 생성 시 **pool의 AD/subnet 선택이 configuration의 AD/subnet보다 우선**한다고 명시한다. config의 image/shape/public key/user-data는 저장하고,58에서 승인된 같은 private-app와 AD를 다시 선택/readback하여 우연한 배치 변경을 차단한다. config OCID가 같아도 pool placement를 자동 같다고 주장하지 않는다. private IP None은 인터넷 inbound 제거, On-demand는 capacity reservation/dedicated 추가 자원 제거, additional volume None은 이 작은 웹앱에 불필요한 저장비 제거를 위한 선택이다.

### configuration은 VM이 아니다
새 설정 방식(Provide a list of settings)을사용한다. Existing instance template는installed files를포함하지않으므로nginx설치를자동복제하지않는다. custom image는56의capture-ready까지만필요하다.
```bash
export CONFIG_ID='<created-instance-configuration-ocid>'
oci compute-management instance-configuration get --instance-configuration-id "$CONFIG_ID"
```
metadata.user_data(업로드결과)/image/shapeConfig/assignPublicIp=false/nsgIds를읽고공개키만포함함을확인한다. yaml의REPLACE_*가남으면Create전에중단한다. runcmd및embeddedBash를문서밖에서검사한뒤upload한다. 실패하면새설정(immutablelaunchsettings)을만들기전oldconfiguration사용consumer를확인하고approvedowner만교체한다.

### custom image 분기: 복원 검증만, 재설치 금지
같은 approval CIDR를 image source/새 member에서 유지한다. private-vm source에서 nginx enabled, 8080 SELinux type, rich rule을 capture 전 확인한다. custom image 사용자 데이터는 아래 원문으로 바꾸며 token/secret은 없다. 기본 platform YAML과 둘 중 하나만 선택한다.
```yaml
#cloud-config
runcmd:
  - [bash, -c, 'set -euo pipefail; test -f /etc/nginx/conf.d/oci-lab.conf; test -f /var/www/oci-lab/health; nginx -t; systemctl is-active --quiet nginx; curl --fail --max-time 10 http://127.0.0.1:8080/health; printf "backend=%s\n" "$(hostname)" > /var/www/oci-lab/index.txt']
```
scale-out마다이검증이실행되고payloadhostname만갱신한다. baked-in앱이없으면생성실패이며platform설치로보정해이미지복원성공이라고쓰지않는다.

## 결과검증

configuration OCID와 실제 pool VM의 launch 설정이 일치한다. config 자체는 VM이 아니다.

- 예상 pattern이며 실제 OCI 실행 결과가 아니다. timestamp, resource OCID, 설정 readback, 실제 HTTP/TCP/파일 결과를 실행자가 기록한다.

## 문제해결

cloudinit실패는YAML/metadata/CIDRplaceholders/DNFoutbound로그확인. manualinstall은scaleout자동bootstrap실패를해결한증거아님.

실패기록에는같은target/credentials/region/요청시간을고정한다. 복구는원래설정으로실행하고actualresponse/readback둘다확인한다;예상값을실측으로쓰지않는다.


### Interrupted bootstrap firewall recovery — owner57 guest
전용 owner lease와 원본 ledger를 확인하고 다른 firewall writer/reload가 없을 때만 새 guest 터미널에서 실행한다. 정상 부팅 후 consumer가 사용 중이면 제거하지 않는다. failed/interrupted add도 같은 pending journal로 복구한다. permanent 변경/query error이면 제거 없이 중단한다. pool 종료가 최종 정리이며 기존 source guest에는 적용하지 않는다.
```bash
set -euo pipefail
LEDGER=/var/lib/oci-lab57-firewall.tsv
# root guest shell; ledger is root-owned mode0600, never source/eval it.
test -f "$LEDGER" && test ! -L "$LEDGER" || exit 1
while IFS=$'\t' read -r state zone runtime_before permanent_before rule; do
  test "$state" = pending && test "$runtime_before" = 1 || exit 1
  case "$permanent_before" in 0|1) :;; *) exit 1;; esac
  test -n "$zone" && test -n "$rule" || exit 1
  firewall-cmd --zone="$zone" --list-all >/dev/null || exit 1
  # Query BOTH actual runtime and permanent in the recorded exact zone.
  if firewall-cmd --zone="$zone" --query-rich-rule="$rule"; then runtime_now=0; else runtime_now=$?; fi
  if firewall-cmd --zone="$zone" --permanent --query-rich-rule="$rule"; then permanent_now=0; else permanent_now=$?; fi
  case "$runtime_now:$permanent_now" in 0:0|0:1|1:0|1:1) :;; *) exit 1;; esac
  # Changed permanent state means outside mutation: stop for owner reconciliation.
  test "$permanent_now" = "$permanent_before" || exit 1
  if [ "$runtime_now" = 0 ]; then
    firewall-cmd --zone="$zone" --remove-rich-rule="$rule" || exit 1
  fi
  if firewall-cmd --zone="$zone" --query-rich-rule="$rule"; then exit 1; else rc=$?; test "$rc" -eq 1 || exit 1; fi
  # Never remove permanent or pre-existing runtime rules.
  if firewall-cmd --zone="$zone" --permanent --query-rich-rule="$rule"; then rc=0; else rc=$?; fi
  test "$rc" = "$permanent_before" || exit 1
done < "$LEDGER"
rm -- "$LEDGER"
```

## 리소스정리

58pool종료와55autoscaling삭제완료인계후 `oci compute-management instance-configuration delete --instance-configuration-id "$CONFIG_ID"`→GetNotFound확인. shared subnet/image임의삭제없음.

정리실행자의실제UTC/OCID/readback을기록한다. 공유resource는owner확인전삭제금지; stop만으로bootvolume/LB/image비용종료가아니다.

## 공식문서

- [Oracle Linux 9 firewalld zone/runtime 설정](https://docs.oracle.com/en/operating-systems/oracle-linux/9/firewall/firewall-ConfiguringfirewalldZones.html) — 인터페이스별 zone과 temporary/runtime 설정 근거; query 오류/ownership ledger는 작성한 안전 gate다.

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 목표는 학습 후 설명·구분·확인할 내용이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [10] [Managing Instance Configurations — Instance Configurations](https://docs.oracle.com/en-us/iaas/Content/Compute/Concepts/instancemanagement.htm)
  - 확인 범위: An instance configuration is.

### 절차별 공식 task body 추가 근거
- [instancemanagement.htm](https://docs.oracle.com/en-us/iaas/Content/Compute/Concepts/instancemanagement.htm) — 본문 검토 근거; 아래 정확한 task와 작성한 입력을 구분한다. OCI 실행 증거가 아니다.
- [creatinginstanceconfig.htm](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/creatinginstanceconfig.htm) — 본문 검토 근거; 아래 정확한 task와 작성한 입력을 구분한다. OCI 실행 증거가 아니다.



### 이번 절차의 정확한 공식 task
- [creatinginstanceconfig.htm](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/creatinginstanceconfig.htm) — 이 문서 실습절차의 입력/동작을 대조한 본문.
- [launchinginstance.htm](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/launchinginstance.htm) — 이 문서 실습절차의 입력/동작을 대조한 본문.
