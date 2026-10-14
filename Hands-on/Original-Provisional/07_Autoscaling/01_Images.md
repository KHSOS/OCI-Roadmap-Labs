> **기존 실습 초안 — 미검증**: 원래 93개 문서의 절차를 보존한 자료입니다. 새 튜토리얼의 품질 승인이나 OCI 실행 검증을 뜻하지 않습니다.

# Images

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../../../Docs/README.md)를 확인하세요.

> 절차 작성본 / OCI 미실행 — 아래 예상값은 실측 결과가 아닙니다. 비용·권한·사전준비 gate를 통과한 전용 환경에서만 실행하세요.

## 학습목표

- 선택 승인한 기존 private-vm 앱을 Custom Image로 capture하고 image AVAILABLE/source health 회복을 기록한다.
- capture-ready를 57/58에 인계하여 같은 pool member의 image-id/기존 baked-in health를 재설치 없이 검증하고 pool/config 반환 후 image를 삭제한다.

완료 기준은 아래 실제 생성·설정·요청 결과로 판정한다. 이 목표는 수행할 작업이며 OCI에서 관찰한 결과가 아니다.

## 대응 Roadmap 항목

- 항목명: **Images**
- Roadmap 경로: **Autoscaling → Images**
- Node ID: `sequence-7-1`
- 상위 항목: **Autoscaling** (`sequence-7-0`)
- 문서 범위: 이 항목의 개념과 작성된 실습절차. 통합 품질검토는 BLOCKED이며 실제 실행은 NOT_EXECUTED입니다.
- [전체 항목 목록](../../../Docs/README.md)

## 사전준비


custom branch만22의 실제 source OCID/Oracle Linux9/architecture/boot/SSH와61 backend-app의 같은 source conf/health/hash를 받는다. vm-a는 client/jump이고 capture target이 아니다. IMAGE_ID=AVAILABLE/source 재접속 health 및22 source STOPPED 반환을57/58에 전달한다; pool 복원 완료는 그 뒤다.

- [Compute](../02_Compute/00_Compute.md#실습절차) — 링크의 `실습절차`와 `결과검증` 완료; 실제 OCID/IP를 registry에 인계한다.


생성 전 `REGION`, `LAB_COMPARTMENT_ID`, 승인 종료시각 `EXPIRY_UTC`, 예산 상한과 실제 resource OCID를 실행 registry에 기록한다. owner가 후속 실습 종료까지 보존하며 이번 세션은 60분 이내 종료한다. 비용 승인/리전 capacity 없으면 실행 중단한다. shell의 변수를 다른 터미널에 자동 전달된다고 가정하지 말고 비밀값 없이 다시 설정한다.
- [Compute](../02_Compute/00_Compute.md#shared-launch)의 `vm-a` SSH/Oracle Linux 9 확인 및 [Private Subnets](../../Networking/Original-Provisional/03_Virtual_Cloud_Network_VCN/03_Private_Subnets.md#실습절차)의 subnet/SSH preparation만 완료; custom-image capture 트랙은 기존 private-vm의 실제 생성/SSH 완료가 필수이며22owner와expiry를확인한다. platform pool 트랙만 이를 생략할 수 있다. owner58만승인pool VM1→2대를생성한다. `KEY`, `PUBLIC_IP`(vm-a), `BACKEND_IP`(private-vm), `CLIENT_PRIVATE_IP`(vm-a), `CLIENT_CIDR=$CLIENT_PRIVATE_IP/32`, subnet/NSG OCID를 실제 readback하여 인계한다. 테스트는 vm-a에서 수행한다; Cloud Shell은 자동 VCN private client가 아니다.

## 권한

Custom source stop/restart는22 owner의 실제 instance에 대한 `INSTANCE_POWER_ACTIONS` 별도 승인 작업이다. image creator의 `INSTANCE_CREATE_IMAGE` 권한을 source power 권한으로 취급하지 않는다. 현재 작업자는 image 생성/readback만 수행하고 stop/restart는 승인된22 owner에게 반환한다.

[corepolicyreference — Permissions Required for Each API Operation / Autoscaling API Operations](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm) 실제 본문 기준이다. 선행 owner의 생성/정리 권한을 현재 사용자에게 자동 부여하지 않는다. 관리자가 exact operation과 실제 resource compartment를 대조한다.

### 이문서 operation별 승인 allowlist
```text
Allow group <identity-domain>/<lab-group> to manage instance-images in compartment <lab-compartment> where any {request.permission='INSTANCE_IMAGE_INSPECT', request.permission='INSTANCE_IMAGE_READ', request.permission='INSTANCE_IMAGE_CREATE', request.permission='INSTANCE_IMAGE_DELETE'}
Allow group <identity-domain>/<lab-group> to manage instances in compartment <lab-compartment> where any {request.permission='INSTANCE_READ', request.permission='INSTANCE_CREATE_IMAGE'}
```
이목록은문서APItable기반의administrator검토용이며실제tenancy허용/거부시험미실행이다. IMAGE_READ는CreateInstancePoolAPI표의표기이고instance-imageverb표는INSTANCE_IMAGE_READ를쓰므로둘을임의동일하다고정규화하지않는다. bootvolume생성/termination및NSGattachment의서비스처리와실제authorization은corepolicy표/실제WorkRequest로확인한다. 후속owner생성권한을현재사용자에게자동부여하지않는다.

## 비용및주의사항

선택 capture branch에서 기존 private-vm의 Custom Image 하나만 만든다. image 저장·source VM/boot 유지와58의 동일 pool 복원 비용을 승인하며 별도 restore VM/LB는 만들지 않는다. capture/재접속 health 뒤 source를 owner22 승인으로 STOPPED/readback하고 pool을 launch하여 running 상한을 지킨다. STOPPED라도 boot 비용은 남는다.58pool/57configuration 반환 후 image 삭제를 확인하고 source restart는 owner22만 한다. 재설치로 baked-in image 복원 성공을 주장하지 않는다.

## 실습구성

- 작업 목표: 실제 custom image를 생성하고 복원 VM의 앱을 검증한다.
- 생성 owner: 56. 동일 owner의 자원을 재사용하며 실제 결과 필드는 실행 전 비워 둔다.


### 이번 workshop 시나리오와 인수·반환

기존 private-vm 앱을 담은 custom image를 만들고 같은 pool로 복원을 확인한다.

- **신규/변경:** custom image1개; source 앱 준비만,추가 restore VM 없음.
- **선행 재사용:** 22 private-vm/SSH,61 app-only.
- **완주 흐름:** 승인/입력→본문의 생성/편집→image Available와source재접속→58 새member image-id/health200/hostname→58 pool 종료→57config삭제→56image삭제→22 source 반환.
- **선택 이유/실패 경계:** capture-ready→config-ready→pool-ready→restore-verified 순서; 재설치로 이미지복원을 날조하지 않는다.

| 입력/산출물 | 실제 값 출처 | 준비 시점 |
|---|---|---|
| `SOURCE_INSTANCE_OCID` | creator의 실제 생성/조회 출력; 임의 OCID/IP 금지 | 사용 전 |
| `IMAGE_ID` | creator의 실제 생성/조회 출력; 임의 OCID/IP 금지 | 생성 후 |
| `BACKEND_IP` | creator의 실제 생성/조회 출력; 임의 OCID/IP 금지 | 사용 전 |

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

[아래생성/변경단계의정확한Oracle task본문](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/custom-images-create.htm) — Consolefield/CLI/API동작근거.

아래 단계는 Oracle task 본문을 근거로 작성한 lab recipe이며 공식 실행결과가 아니다. 각 단계의 근거는 끝의 정확한 task 링크를 따른다.

1. **custom-image 선택 트랙에서만** [Private Subnets](../../Networking/Original-Provisional/03_Virtual_Cloud_Network_VCN/03_Private_Subnets.md#baseline-22-ready)의 기존 `private-vm` 실제 생성/ProxyJump SSH 완료가 필수다. `SOURCE_INSTANCE_OCID=INSTANCE_B_ID`, `BACKEND_IP=B_PRIVATE_IP`를 같은 instance/VNIC readback에서 설정하고 source Compartment/Region/Oracle Linux 9/architecture/boot OCID를 기록한다. vm-a는 SSH jump/HTTP client 전용이며 capture 또는 앱 설치 대상으로 사용하지 않는다. source 전용 새 VM을 중복 생성하지 않는다. platform-image pool 트랙은 이 capture를 NOT_SELECTED로 남긴다; source 생성/SSH 입력이 없으면 custom-image는 BLOCKED이다. 라이선스 제한 image는 제외한다.
2. [Load Balancer app 전용](../08_Load_Balancer/00_Load_Balancer.md#backend-app)의 A 단계만 수행하고 LB는 생성하지 않는다. 아래 LB owner의 앱 설치 단계와 동일하게 Nginx에 비민감 index/health를 설치한다. 앱 설정 파일을 백업하고 credential/API key/사용자 데이터가 image에 포함되지 않음을 검사한다. 서비스 영향 승인을 받는다. Linux image capture는 서비스가 shutdown/restart하며 Windows의 사전 OS shutdown 요구를 Linux에 일괄 적용하지 않는다.
3. Console → Compute → Instances에서 **readback과 일치하는 SOURCE_INSTANCE_OCID/private-vm**을 열고 More Actions → Create Custom Image에서 `lab-clean-web-image`를 입력한다. step2의 설치/localhost health 증거도 같은 source guest/OCID여야 한다. source 종료/재시작 동안 vm-a jump의 TCP22 egress와 B의 A/32 ingress는 보존하고 host key 검증을 끄지 않는다. 이미지 생성 동안 source instance 상태와 image Available 완료를 확인하고 source ProxyJump 재접속 및 /health 200/ok 복구를 기록한다.
4. [Instance Configurations](02_Instance_Configurations.md#configuration-ready)의 owner57이 이 image OCID로 configuration-ready를 저장하고, [Instance Pools](03_Instance_Pools.md#pool-ready)의 owner58이 **같은 pool member**를 실제 생성한 뒤 `curl -i http://<new-private-ip>:8080/health`를 실행한다. 기존 hostname이 baked-in될 수 있으므로 `printf "backend=%s\n" "$(hostname)" | sudo tee /var/www/oci-lab/index.txt`로 새 hostname을 기록하고 :8080/도 검사한다. owner58의 동일 pool size1에서 custom image 복구를 검증하고 image 전용 추가 VM은 만들지 않는다.
5. 원본과 새 VM의 image OCID/shape/Region 호환성과 앱 상태를 기록한다. image Available만으로 boot 검증을 대체하지 않는다.


### image-capture-ready

### 이미지 handoff의 순환 제거
56의 `IMAGE_CAPTURE_READY`는 image AVAILABLE까지이다. 57에서그 OCID를선택하고58의동일pool을launch하여 `IMAGE_RESTORE_VERIFIED`가된다. 57이56의restore완료를사전요구하지않는다. 기본platform이미지트랙이면56을건너뛴다. source는 전용private-vm이며vm-a jump/admin을capture용으로중단하지않는다. guest 앱 설치후 `sudo nginx -t; curl -f http://127.0.0.1:8080/health`를먼저실행한다. ~/.oci/APIkeys/SSHprivatekey/secret을수집하지않고존재하면capture중단한다. boot volume내용이이미지로들어간다는점을승인한다.
```bash
export IMAGE_ID='<available-custom-image-ocid>'
oci compute image get --image-id "$IMAGE_ID"
```
restore guest의 :8080/health와 actual instance image-id를readback한다. snapshot/key/capture이미지는비밀관리대체물이아니다.

## 결과검증

새 VM Running과 /health 200을 실제 확인한다. 비밀번호/SSH private key 등 image 유출이 없는지 확인한다.

- 예상 pattern이며 실제 OCI 실행 결과가 아니다. timestamp, resource OCID, 설정 readback, 실제 HTTP/TCP/파일 결과를 실행자가 기록한다.

## 문제해결

image와shapearchitecture/imagecapability일치확인;nginx port8080/SELinuxlabel/firewall확인. customimagecontents를instanceconfiguration복제로착각하지않는다.

실패기록에는같은target/credentials/region/요청시간을고정한다. 복구는원래설정으로실행하고actualresponse/readback둘다확인한다;예상값을실측으로쓰지않는다.

## 리소스정리

58pool/VM종료→57configuration삭제인계후 `oci compute image delete --image-id "$IMAGE_ID"`→imageGet의NotFound확인. 원본private-vm과boot는22owner가종료.

정리실행자의실제UTC/OCID/readback을기록한다. 공유resource는owner확인전삭제금지; stop만으로bootvolume/LB/image비용종료가아니다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 목표는 학습 후 설명·구분·확인할 내용이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [13] [Platform Images — Images](https://docs.oracle.com/en-us/iaas/Content/Compute/References/images.htm)
  - 확인 범위: An image is a template / You also can.

### 절차별 공식 task body 추가 근거
- [images.htm](https://docs.oracle.com/en-us/iaas/Content/Compute/References/images.htm) — 본문 검토 근거; 아래 정확한 task와 작성한 입력을 구분한다. OCI 실행 증거가 아니다.
- [managingcustomimages.htm](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/managingcustomimages.htm) — 본문 검토 근거; 아래 정확한 task와 작성한 입력을 구분한다. OCI 실행 증거가 아니다.



### 이번 절차의 정확한 공식 task
- [custom-images-create.htm](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/custom-images-create.htm) — 이 문서 실습절차의 입력/동작을 대조한 본문.
