# IP Address Preservation

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../README.md)를 확인하세요.

> 절차 작성본 / OCI 미실행 — 아래 예상값은 실측 결과가 아닙니다. 비용·권한·사전준비 gate를 통과한 전용 환경에서만 실행하세요.

## 학습목표

- same Compute-instance backend set의 Preserve source IP만 켜고 NLB 대상 새 REQUEST_ID의 client tuple/backend peer/echo를 연결한다.
- NLB-level source/destination=false를 유지하여 same-VCN return을 대조하고 Full NAT로 원복한 새 연결 성공을 기록한다.

완료 기준은 아래 실제 생성·설정·요청 결과로 판정한다. 이 목표는 수행할 작업이며 OCI에서 관찰한 결과가 아니다.

## 대응 Roadmap 항목

- 항목명: **IP Address Preservation**
- Roadmap 경로: **Network Load Balancer → IP Address Preservation**
- Node ID: `service-nlb-2`
- 상위 항목: **Network Load Balancer** (`service-nlb-0`)
- 문서 범위: 이 항목의 개념과 작성된 실습절차. 통합 품질검토는 BLOCKED이며 실제 실행은 NOT_EXECUTED입니다.
- [전체 항목 목록](../README.md)

## 사전준비


66 same Compute-instance backend OCID/VNIC/private IP:9090와 Full NAT positive/actual REQUEST_ID peer를 받는다. CLIENT_PRIVATE_IP/32 ingress·probe와 same-VCN return path가 준비되어야 한다. NLB-level source/destination=false를 고정하며 backend set original 설정과 독립 복구 담당자를 확보한다.

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

새 NLB/LB/VM/backend를 만들지 않고66의 동일 Compute-instance backend set source preservation만 false→true→false로 변경한다. 기존 NLB/실제 전송·VM/boot 유지와66 lease를 따른다. 새 연결로만 tuple/peer/payload를 비교하며 NLB-level source/destination=false와 return path를 고정한다. IP-only backend를 그대로 toggle하는 branch는 금지한다. original Full NAT와 새 echo 성공을 readback한 뒤 이번 변경/소켓만 반환한다.

## 실습구성

- 작업 목표: 66의 동일 NLB source preservation 설정과 return path를 검증한다.
- 생성 owner: 66. 동일 owner의 자원을 재사용하며 실제 결과 필드는 실행 전 비워 둔다.


### 이번 workshop 시나리오와 인수·반환

Compute-instance backend set의 source 보존만 바꾸고 새연결 peer/응답을 비교한다.

- **신규/변경:** isPreserveSource false→true→원래false.
- **선행 재사용:** 66FullNAT/Compute-instance backend/client9090허용.
- **완주 흐름:** 승인/입력→본문의 생성/편집→새연결peer=clientIP+payload/return→FullNAT원복새연결→preserve 원복→consumer rule보존필요검토→66 반환.
- **선택 이유/실패 경계:** NLB-level source/destination false;IP-only backend로는그냥toggle하지않는다.

| 입력/산출물 | 실제 값 출처 | 준비 시점 |
|---|---|---|
| `NLB_OCID` | creator의 실제 생성/조회 출력; 임의 OCID/IP 금지 | 사용 전 |
| `CLIENT_PRIVATE_IP` | creator의 실제 생성/조회 출력; 임의 OCID/IP 금지 | 사용 전 |
| `BACKEND_IP` | creator의 실제 생성/조회 출력; 임의 OCID/IP 금지 | 사용 전 |

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

[아래생성/변경단계의정확한Oracle task본문](https://docs.oracle.com/en-us/iaas/Content/NetworkLoadBalancer/BackendSets/preserve-backend-set-source-id.htm) — Consolefield/CLI/API동작근거.

아래 단계는 Oracle task 본문을 근거로 작성한 lab recipe이며 공식 실행결과가 아니다. 각 단계의 근거는 끝의 정확한 task 링크를 따른다.

1. [Network Load Balancer](00_Network_Load_Balancer.md)의 A/B/C/D 완료와 **Compute instances backend type**, instance OCID/VNIC/private IP 일치를 확인한다. 기존 IP-only backend면 true를 제출하지 말고 owner66이 같은 private-vm의 Compute-instance 등록으로 교체/기존등록제거/FullNAT positive control을 승인한 뒤 진행한다. 두 번째 VM/NLB를 만들지 않는다. backend NSG/OS firewall에 승인 client CIDR9090 및 health probe 출발지를 먼저 허용한다.
2. Edit 이전 same NLB_OCID/echo9090의 전체 original source/health/failOpen/backend 등록값을 저장하고 owner66 독립 Console 복구세션·lease 이내 최대120초 deadline을 확보한다. 오류/세션 손실/기한에는 원래 Full NAT를 복원/readback하고 새66C echo를 검증한다. 담당자/원래값 없으면 mutation 금지다. Backend set → Edit의 **Preserve source IP** 설정(isPreserveSource=true, backend set 수준)을 공식 body와 대조하고 현재값을 기록한다. 지원되는 source preservation을 enable한 후 Work Request 완료/readback한다. NLB 수준 isPreserveSourceDestination=false를 유지한다. destination preservation/transparent를 별도 승인 없이 같이 켜지 않는다.
3. 66 C단계의 **NLB_IP 대상만** 새 REQUEST_ID/시각/source-port로재실행한다. directbackend양성대조는별도ID로그이며preservation증거에서제외한다. backendjournal의동일REQUEST_ID peer와client출력getsockname을매칭하여source-preserve의clientIP를확인한다. broad5분peer목록만으로성공판정하지않는다. `ip route get <client-private-ip>`와 VCN subnet route의 반환 경로를 검사한다. 예상과 다르면 frontend 설정만 바꾸지 말고 client→NLB→backend와 backend→client route를 모두 조사한다.
4. 원래 preserve 설정으로 복원하고 동일 payload 테스트를 반복한다. backend에서 보이는 source가 변한 증거를 기록한다.

### packet tuple/반환 의미
C=client privateIP/cport, V=NLB privateVIP:9090, B=backend privateIP:9090라한다. FullNAT 요청client-visible C→V는 backend-visible S:translated-port→B로변환된다(S/port는actualjournal에서읽으며VIP라고단정안함). 응답B→S가service번역을통해 client-visible V→C로돌아온다. backend-set source-preserve=true/transparent=false 요청은client C→V, backend C→B이며guestreturn은B→C의sameVCNroute를선택한다. clientsocket은 V→C의응답을관찰하므로guest route get만으로OCI reverse translation/payloadreturn을증명하지않는다. 실제correlatedecho/tuple/readback이필요하며sourceport보존여부도actualtuple로기록한다. TCPtrackedreturn은새reverseconnection허가가아니다.

### source preservation test control
66FullNATclientpayload성공및backendpeer를먼저기록한다. BackendSetecho9090EditPreservesourceIP=true만변경하고기존isFailOpen=false/healthpolicy/weight를보존한다. NLB-levelSource/Destination=false이다.
```bash
oci nlb backend-set get --network-load-balancer-id "$NLB_OCID" --backend-set-name echo9090
```
변경후**새TCP연결**을66C로만들고peer=실제vm-a privateIP/응답payload확인한다. backend`ip route get "$CLIENT_PRIVATE_IP"`와sameVCNlocal경로확인;source/destinationtransparent용route를임의추가하지않는다. 거부시즉시Preservesource=false원복→GetBackendSet→새연결payload성공이다. client /32허용규칙은복원후66정리전까지필요하면유지한다.

## 결과검증

preserve readback과 peer source/return TCP 응답을 함께 확인한다. TCP timeout은 asymmetry 또는 source 규칙 누락부터 점검한다.

- 예상 pattern이며 실제 OCI 실행 결과가 아니다. timestamp, resource OCID, 설정 readback, 실제 HTTP/TCP/파일 결과를 실행자가 기록한다.

## 문제해결

새연결peer비교만유효;sourceIPpreserve시backendclient /32규칙과returnroute확인. transparentSourceDestination은이실습범위아님.

실패기록에는같은target/credentials/region/요청시간을고정한다. 복구는원래설정으로실행하고actualresponse/readback둘다확인한다;예상값을실측으로쓰지않는다.

## 리소스정리

Preservesource=false원래값과NLBSourceDestination=falsereadback→66C새연결응답;임시추가clientrule은다른consumer없을때만제거. NLB66보존.

정리실행자의실제UTC/OCID/readback을기록한다. 공유resource는owner확인전삭제금지; stop만으로bootvolume/LB/image비용종료가아니다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 목표는 학습 후 설명·구분·확인할 내용이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [33] [Overview of Flexible Network Load Balancer — IP Address Preservation](https://docs.oracle.com/en-us/iaas/Content/NetworkLoadBalancer/overview.htm)
  - 확인 범위: IP Address Preservation.

### 절차별 공식 task body 추가 근거
- [overview.htm](https://docs.oracle.com/en-us/iaas/Content/NetworkLoadBalancer/overview.htm) — 본문 검토 근거; 아래 정확한 task와 작성한 입력을 구분한다. OCI 실행 증거가 아니다.
- [backend-set-management.htm](https://docs.oracle.com/en-us/iaas/Content/NetworkLoadBalancer/BackendSets/backend-set-management.htm) — 본문 검토 근거; 아래 정확한 task와 작성한 입력을 구분한다. OCI 실행 증거가 아니다.



### 이번 절차의 정확한 공식 task
- [introduction.htm](https://docs.oracle.com/en-us/iaas/Content/NetworkLoadBalancer/introduction.htm) — 이 문서 실습절차의 입력/동작을 대조한 본문.

- [정확한 NLB edit task](https://docs.oracle.com/en-us/iaas/Content/NetworkLoadBalancer/BackendSets/preserve-backend-set-source-id.htm) — 보존/health변경입력과readback절차근거.
