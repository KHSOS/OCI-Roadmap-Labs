> **기존 실습 초안 — 미검증**: 원래 93개 문서의 절차를 보존한 자료입니다. 새 튜토리얼의 품질 승인이나 OCI 실행 검증을 뜻하지 않습니다.

# Instance Pools

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../../../Docs/README.md)를 확인하세요.

> 절차 작성본 / OCI 미실행 — 아래 예상값은 실측 결과가 아닙니다. 비용·권한·사전준비 gate를 통과한 전용 환경에서만 실행하세요.

## 학습목표

- owner57 configuration으로 private pool size1을 실제 생성하고 member IP→network rules→cloud-init→direct health/hostname을 확인한다.
- controller 없는 상태의 bounded nginx 장애/복구를 검증하고 선택 scaling 반환 후 pool/member/boot와 새 per-member rules를 정리한다.

완료 기준은 아래 실제 생성·설정·요청 결과로 판정한다. 이 목표는 수행할 작업이며 OCI에서 관찰한 결과가 아니다.

## 대응 Roadmap 항목

- 항목명: **Instance Pools**
- Roadmap 경로: **Autoscaling → Instance Pools**
- Node ID: `sequence-7-3`
- 상위 항목: **Autoscaling** (`sequence-7-0`)
- 문서 범위: 이 항목의 개념과 작성된 실습절차. 통합 품질검토는 BLOCKED이며 실제 실행은 NOT_EXECUTED입니다.
- [전체 항목 목록](../../../Docs/README.md)

## 사전준비


57 configuration-ready의 actual CONFIG_ID/image/shape/YAML/public key/nsg-pool-app와21 private-app,10 vm-a actual client/SSH를 받는다. pool Create 전 global running VM gate와 custom 선택의22 source STOPPED를 확인한다. POOL_ID/member IP는 생성 뒤 읽고 /32별 client 규칙→cloud-init→HTTP8080 health 순으로55/59/60에 인계한다.

- [Instance Configurations](02_Instance_Configurations.md#configuration-ready) — configuration-ready까지만 완료; member 생성/앱 검증은 이 문서의 pool-ready 후 57/59 결과검증에 인계한다.
- [Subnets](../../Networking/Original-Provisional/03_Virtual_Cloud_Network_VCN/02_Subnets.md#baseline-21-ready) — private-app subnet/route/SL 준비 완료. [Private Subnets](../../Networking/Original-Provisional/03_Virtual_Cloud_Network_VCN/03_Private_Subnets.md#실습절차)의 jump 규칙만 참조하고 새 member 실제 IP를 받은 뒤 승인 TCP22 egress를 추가한다; 기본 platform 트랙에서 기존 private-vm은 만들지 않는다.


생성 전 `REGION`, `LAB_COMPARTMENT_ID`, 승인 종료시각 `EXPIRY_UTC`, 예산 상한과 실제 resource OCID를 실행 registry에 기록한다. owner가 후속 실습 종료까지 보존하며 이번 세션은 60분 이내 종료한다. 비용 승인/리전 capacity 없으면 실행 중단한다. shell의 변수를 다른 터미널에 자동 전달된다고 가정하지 말고 비밀값 없이 다시 설정한다.
- [Compute](../02_Compute/00_Compute.md#shared-launch)의 `vm-a` SSH/Oracle Linux 9 확인 및 [Private Subnets](../../Networking/Original-Provisional/03_Virtual_Cloud_Network_VCN/03_Private_Subnets.md#실습절차)의 subnet/SSH preparation만 완료; 기존private-vm 생성은pool트랙필수아니다. owner58만승인pool VM1→2대를생성한다. `KEY`, `PUBLIC_IP=A_PUBLIC_IP`, `CLIENT_PRIVATE_IP=A_PRIVATE_IP`, `CLIENT_CIDR=$CLIENT_PRIVATE_IP/32`, private subnet/NSG OCID만 생성 전에 인계한다. `POOL_ID`와 member IP는 owner58의 생성 후 실제 출력이다. 기본 platform 트랙은 `BACKEND_IP`/기존 private-vm을 요구하지 않는다. 검증은 vm-a guest에서 수행하며 Cloud Shell은 VCN private client가 아니다.

## 권한

[corepolicyreference — Permissions Required for Each API Operation / Autoscaling API Operations](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm) 실제 본문 기준이다. 선행 owner의 생성/정리 권한을 현재 사용자에게 자동 부여하지 않는다. 관리자가 exact operation과 실제 resource compartment를 대조한다.

### 이문서 operation별 승인 allowlist
```text
Allow group <identity-domain>/<lab-group> to manage instance-pools in compartment <lab-compartment> where any {request.permission='INSTANCE_POOL_READ', request.permission='INSTANCE_POOL_INSPECT', request.permission='INSTANCE_POOL_CREATE', request.permission='INSTANCE_POOL_UPDATE', request.permission='INSTANCE_POOL_DELETE'}
Allow group <identity-domain>/<lab-group> to manage instances in compartment <lab-compartment> where any {request.permission='INSTANCE_READ', request.permission='INSTANCE_CREATE', request.permission='INSTANCE_DELETE'}
Allow group <identity-domain>/<lab-group> to read instance-images in compartment <actual-image-compartment>
Allow group <identity-domain>/<lab-group> to manage vnics in compartment <lab-compartment> where any {request.permission='VNIC_READ', request.permission='VNIC_CREATE', request.permission='VNIC_ATTACH', request.permission='VNIC_DELETE', request.permission='VNIC_ASSOCIATE_NETWORK_SECURITY_GROUP'}
Allow group <identity-domain>/<lab-group> to manage subnets in compartment <lab-compartment> where any {request.permission='SUBNET_READ', request.permission='SUBNET_ATTACH', request.permission='SUBNET_DETACH'}
Allow group <identity-domain>/<lab-group> to manage network-security-groups in compartment <lab-compartment> where any {request.permission='NETWORK_SECURITY_GROUP_READ', request.permission='NETWORK_SECURITY_GROUP_UPDATE_MEMBERS'}
```
이목록은문서APItable기반의administrator검토용이며실제tenancy허용/거부시험미실행이다. IMAGE_READ는CreateInstancePoolAPI표의표기이고instance-imageverb표는INSTANCE_IMAGE_READ를쓰므로둘을임의동일하다고정규화하지않는다. bootvolume생성/termination및NSGattachment의서비스처리와실제authorization은corepolicy표/실제WorkRequest로확인한다. 후속owner생성권한을현재사용자에게자동부여하지않는다.

TerminateInstancePool의 공식 API 표는 INSTANCE_POOL_DELETE + INSTANCE_DELETE + VNIC_DELETE + SUBNET_DETACH + VOLUME_ATTACHMENT_DELETE + VOLUME_WRITE이다. 추가선택 data volume을 만들지 않는다. owner58에게만 다음 cleanup 의존성을 승인한다. member VNIC readback에는 VNIC_ATTACHMENT_READ도 필요하다.
```text
Allow group <identity-domain>/<lab-group> to inspect vnic-attachments in compartment <lab-compartment>
Allow group <identity-domain>/<lab-group> to manage volume-attachments in compartment <lab-compartment> where request.permission='VOLUME_ATTACHMENT_DELETE'
Allow group <identity-domain>/<lab-group> to use volumes in compartment <lab-compartment> where request.permission='VOLUME_WRITE'
```

## 비용및주의사항

private pool 하나를 size1로 생성한다. vm-a+pool1의 전체 running2대와 각 Compute/boot, 승인 설치 NAT 및 선택 custom image 저장 비용을 확인한다. size2는 전체 running3대·capacity/예산을 별도 승인한 선택 scaling에서만 허용한다. 기존 private-vm을 platform branch의 필수로 새로 만들거나 자동 LB를 붙이지 않는다.55/60 controller 삭제/readback 후 pool/member/이번 boot와 per-member rule을 정리한다. STOPPED source나 남은 boot/image 저장 비용도 따로 확인한다.

## 실습구성

- 작업 목표: 단일 instance pool을 size 1로 생성하고 앱을 작동시킨다.
- 생성 owner: 58. 동일 owner의 자원을 재사용하며 실제 결과 필드는 실행 전 비워 둔다.


### 이번 workshop 시나리오와 인수·반환

private pool을 size1로 생성하여 자동 설치된 nginx를 직접 읽는다.

- **신규/변경:** pool1개/member1개;승인시 최대2.
- **선행 재사용:** 57configuration-ready,21 private-app,10vm-a.
- **완주 흐름:** 승인/입력→본문의 생성/편집→cloud-init/nginx/localhost200→vm-a8080 health/hostname→stop/start복구→55/60삭제→poolTerminate/boot삭제→57 반환.
- **선택 이유/실패 경계:** image-policy spelling은 관리자 승인 조건; 별도 B/자동LB연결 불필요.

| 입력/산출물 | 실제 값 출처 | 준비 시점 |
|---|---|---|
| `CONFIG_ID` | creator의 실제 생성/조회 출력; 임의 OCID/IP 금지 | 사용 전 |
| `POOL_ID` | creator의 실제 생성/조회 출력; 임의 OCID/IP 금지 | 생성 후 |
| `member-private-IP` | creator의 실제 생성/조회 출력; 임의 OCID/IP 금지 | 생성 후 |
| `CLIENT_CIDR` | creator의 실제 생성/조회 출력; 임의 OCID/IP 금지 | 사용 전 |

입력별 실행위치는 본문 터미널 구분을 따른다: OCI CLI=인증 Cloud Shell/승인 OCI 터미널, OS 앱/NFS/부하=지정 guest, private HTTP/TCP=vm-a guest, YAML 작성/SSH private key=본인 local. 조회/변경/정리의 관리자 승인은 **실행 조건**이며 이 문서는 IAM 성공을 입증하지 않는다. 실제 허용/거부와 capacity는 사용자 실행에서 확인한다. 미승인/NOT_SELECTED/ASYNC_PENDING을 성공으로 바꾸지 않는다.

### 전체 running VM 상한과 source 반환
pool size 상한은 전체 running VM 상한이 아니다. 기본 platform size1은 vm-a+pool1의2대다. size2 scale-out은 **vm-a+pool2=전체3대**를 budget/capacity 담당자가 별도 승인한 경우만 선택한다; 기존baseline의2대상한을자동승계/우회하지않는다. 다른running private-vm/부하VM이남아있으면owner가정리/stop하거나그전체수량을별도승인하기전중단한다. custom56은 source capture/재접속health확인뒤 owner22의 INSTANCE_POWER_ACTIONS 승인으로 source private-vm을Console Stop하고STOPPED/readback한다. bootvolume비용/expiry는계속기록하고source재시작은pool종료뒤owner22만한다. 이 handoff후58pool을launch하므로capture source와pool member를동시에추가running으로유지하지않는다. 선택scale-out은전체3대별도승인이고승인없으면NOT_SELECTED다.

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

[아래생성/변경단계의정확한Oracle task본문](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/creatinginstancepool.htm) — Consolefield/CLI/API동작근거.

아래 단계는 Oracle task 본문을 근거로 작성한 lab recipe이며 공식 실행결과가 아니다. 각 단계의 근거는 끝의 정확한 task 링크를 따른다.


### global-vm-gate

### Global running-VM 비용 gate — pool 첫 launch와 모든 size2 이전
owner58은 Console Compute → Instances에서 **모든 lab 사용 Region/Compartment**의 실제 Running 및 Starting/Provisioning VM을 OCID/owner/상태별 inventory한다. vm-a, 기존 private-vm/custom source, 다른 pool/member도 포함하고 중복 OCID를 제거한다. 현재 Running + 이미 승인된 pending launch + 이번 추가 member 수의 총합을 baseline ceiling **2대**와 대조한다. 다른 compartment의 lab VM 조회가 불가하면 중단한다. 초과하면 총 VM 수/shape/예산/expiry를 승인자에게 별도 승인받기 전 Create/resize 금지다. vm-a1 + pool2 = **총3대**이므로 size2 승인이라는 문장만으로 ceiling을 override하지 않는다.
custom 선택은56 AVAILABLE capture와 source 재접속/health 확인 반환 뒤 **첫 pool Create 이전** owner22가 INSTANCE_POWER_ACTIONS 승인을 받아 Console Compute → Instances → 실제 source OCID → Stop(Force stop 미선택) → Stop instance를 수행한다. 실제 instance 상태가 **STOPPED**가 될 때까지 상세 readback하고 source OCID/UTC를 기록한다. OS shutdown만으로 대체하지 않는다. STOPPED 확인 실패/다른 source consumer 미반환이면 pool 생성 금지다. source stop은 boot/image 저장비 삭제가 아니며 보존비용/shape별 stopped billing을 별도 승인한다. STOPPED 뒤 inventory를 다시 조회하여 source를 Running 합계에서 제외하고 첫 launch 총수를 승인한다. source 재시작은 pool 반환 후 owner22 승인으로만 한다. 이 gate는55 pool-ready를 선행조건으로 요구하지 않는다.

1. [Instance Configurations](../../../Docs/07_Autoscaling/02_Instance_Configurations.md)의 configuration 저장/readback 단계까지 완료한다. [Compute](../../../Docs/02_Compute/00_Compute.md) SSH 키와 접속 경로를 준비한다.
2. **CONDITIONAL_ADMIN_APPROVAL — IMAGE_POLICY_COMPATIBILITY:** 공식 CreateInstancePool operation 행의 `IMAGE_READ`와 instance-images read verb 행의 `INSTANCE_IMAGE_READ` 불일치를 alias라고 주장하지 않는다. 위 `read instance-images`는 실제 image Compartment에 관리자가 승인할 **non-minimum verb grant**이며 token 추측 필터를 쓰지 않는다. 관리자가 선택 image의 실제 compartment/tenancy scope, 정책 호환성 및 operation 의존 권한을 확인하고 registry에 승인 증거를 남기기 전 **pool Create를 제출하지 않는다**. 플랫폼 이미지에도 접근 범위를 자동 가정하지 않는다. 승인 후 Console → Instance Pools → Create에서 이름 `lab-pool`, configuration OCID, size 1, 승인된 AD/FD와 private subnet을 선택한다. 처음에는 LB 자동 연결을 추가하지 않는다.

### pool-provisioned

3. **pool-provisioned**(아직 app-ready 아님): pool Running 및 VM Running을 기다리고 각 VM IP를 읽는다. cloud-init 오류는 `/var/log/cloud-init-output.log`를 민감정보 없이 점검한다.
member IP를 읽은 **즉시** 아래 pool ID→instance/IP→app station의 network owner가 member /32별 vm-a22/8080 egress/NSG ingress를 승인/조회한다. 이 단계가 SSH/HTTP보다 앞선다.

4. [자동 app-bootstrap](02_Instance_Configurations.md#app-bootstrap)이 최초 boot에서 성공해야 한다. guest `sudo cloud-init status --wait && sudo nginx -t && curl -f http://127.0.0.1:8080/health`와 vm-a `curl --connect-timeout 3 --max-time 10 -f http://<pool-vm-private-ip>:8080/health` 및 :8080/ hostname을 각 VM에서 검사한다. 수동 설치로 자동 bootstrap 실패를 숨기지 않는다. 위 Global running-VM gate를 다시 수행하여 총3대 또는 실제 더 큰 총수의 별도 승인을 받은 경우만 size 2로 update하고 새 VM에 앱이 준비되었는지 확인한다. 확인 후 size 1로 돌린다.

### pool-ready

**pool-ready 완료 gate:** 위4의 모든 member cloud-init/nginx/localhost 및 vm-a direct health200/hostname이 실제 성공한 뒤만57/55/59/60에 반환한다. Running/IP만으로 완료하지 않는다.

5. autoscaling은 55가 소유한다. 수동 resize와 autoscaling policy를 동시에 시험하지 않는다.

### pool ID → instance/IP → app 검증
```bash
export POOL_ID='<created-instance-pool-ocid>'
oci compute-management instance-pool get --instance-pool-id "$POOL_ID"
oci compute-management instance-pool list-instances --instance-pool-id "$POOL_ID" --compartment-id "$LAB_COMPARTMENT_ID"
```
각instance Console Attached VNIC의privateIP를registry에기록한다. jump접속은22문서와동일opc/KEY이며nsg-adminTCP22egress대상에각새VM /32를승인추가한다. clientvm-a의TCP8080egress도새VM /32마다추가한다. private VM NSG ingress22/8080는vm-a /32로한정한다. 모든VM에서cloud-init완료와hostnamepayload를확인한다. 장애실험은 아래 **pool-app-recovery** station만 사용한다. scope는 autoscaling configuration이 연결되지 않은 수동 size1로 고정하고 원래 nginx active 상태를 먼저 확인한다.


### pool-app-recovery

### pool 앱 bounded 장애/복구

실제 member guest에서만 아래를 실행한다. vm-a/Console 관측은 다른 터미널이며 owner58의 별도 복구 세션이 필요하다. expiry 잔여가120초 미만이면 미선택 또는 그 짧은 승인시간으로 timeout을 줄인다. kill9/전원손실은 trap이 보장하지 않아 owner58이 즉시 member에 재접속/start한다.
```bash
set -euo pipefail
sudo systemctl is-active --quiet nginx || exit 1
curl --fail --max-time 10 http://127.0.0.1:8080/health
restore_pool_nginx() { sudo systemctl start nginx; }
trap restore_pool_nginx EXIT
trap 'exit 130' INT
trap 'exit 143' TERM
sudo systemctl stop nginx
read -r -t 120 -p 'observe direct failure from vm-a; Enter restores: ' answer || true
restore_pool_nginx
trap - EXIT INT TERM
curl --fail --max-time 10 http://127.0.0.1:8080/health
```
다른 터미널 vm-a에서 같은 actual member :8080/health 신규 요청 실패를 기록하고 복구 후 200/ok/hostname을 다시 대조한다. controller가 있으면 owner55/60 disable만으로 manual 경쟁을 강제하지 말고 이 생성 초기 station으로 돌아가지 않는다.

## 결과검증

size/instance 수/앱 응답 일치와 Running을 확인한다. Pending은 capacity 부족 가능성을 확인한다.

- 예상 pattern이며 실제 OCI 실행 결과가 아니다. timestamp, resource OCID, 설정 readback, 실제 HTTP/TCP/파일 결과를 실행자가 기록한다.

## 문제해결

capacityPending이면workrequest/ADFD/shape/subnetIP확인. SSH는opc+vm-a ProxyJump와양방향22새연결규칙확인;backendHTTP는vm-a8080egress필요.

실패기록에는같은target/credentials/region/요청시간을고정한다. 복구는원래설정으로실행하고actualresponse/readback둘다확인한다;예상값을실측으로쓰지않는다.

## 리소스정리

owner58은 각 member actual IP와 vm-a nsg-admin에 추가한 TCP22/8080 egress rule ID를 registry에 연결한다. 수동/자동 scale-in으로 terminated된 member도 해당 rule의 consumer가 없으면 exact 새 ID만 제거한다. 마지막 pool termination 뒤 모든 owner58 per-member rule이 없고 기존 admin443/다른consumer22가 유지됨을 readback한다. nsg-pool-app은57/network owner가 config/모든pool참조 해제 뒤 정리한다.

55/60autoscaling삭제후ConsolepoolTerminate,**bootvolume삭제**선택,instance도terminated인지확인. 보존bootvolume있으면OCID인계후owner58이Delete하고volume목록없음확인. CONFIG_ID는57owner반환. 보존 boot volume의 DeleteBootVolume은 BOOT_VOLUME_DELETE를추가승인한owner만수행한다. pool API의 VOLUME_WRITE를bootvolume삭제권한으로착각하지않는다.

정리실행자의실제UTC/OCID/readback을기록한다. 공유resource는owner확인전삭제금지; stop만으로bootvolume/LB/image비용종료가아니다.

## 공식문서

- [Stopping an Instance](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/restartinginstance-stop-instance.htm) — Console Stop 및 OS shutdown만으로 대체하지 않는 근거.

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 목표는 학습 후 설명·구분·확인할 내용이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [15] [Creating Instance Pools — Instance Pools](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/creatinginstancepool.htm)
  - 확인 범위: Use instance pools / When you create an instance pool / Optionally, you can associate.

### 절차별 공식 task body 추가 근거
- [creatinginstancepool.htm](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/creatinginstancepool.htm) — 본문 검토 근거; 아래 정확한 task와 작성한 입력을 구분한다. OCI 실행 증거가 아니다.



### 이번 절차의 정확한 공식 task
- [creatinginstancepool.htm](https://docs.oracle.com/iaas/Content/Compute/Tasks/creatinginstancepool.htm) — 이 문서 실습절차의 입력/동작을 대조한 본문.
