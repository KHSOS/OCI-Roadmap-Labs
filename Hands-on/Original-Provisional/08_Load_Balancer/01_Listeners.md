> **기존 실습 초안 — 미검증**: 원래 93개 문서의 절차를 보존한 자료입니다. 새 튜토리얼의 품질 승인이나 OCI 실행 검증을 뜻하지 않습니다.

# Listeners

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../../../Docs/README.md)를 확인하세요.

> 절차 작성본 / OCI 미실행 — 아래 예상값은 실측 결과가 아닙니다. 비용·권한·사전준비 gate를 통과한 전용 환경에서만 실행하세요.

## 학습목표

- 같은 LB에 승인된 HTTP8081 listener를 web8080으로 연결하고 8081 health/기존80 요청을 대조한다.
- 8081 listener 삭제/재생성의 실패→복구를 확인한 뒤 이번 listener와 새 8081 규칙만 제거한다.

완료 기준은 아래 실제 생성·설정·요청 결과로 판정한다. 이 목표는 수행할 작업이며 OCI에서 관찰한 결과가 아니다.

## 대응 Roadmap 항목

- 항목명: **Listeners**
- Roadmap 경로: **Load Balancer → Listeners**
- Node ID: `service-lb-1`
- 상위 항목: **Load Balancer** (`service-lb-0`)
- 문서 범위: 이 항목의 개념과 작성된 실습절차. 통합 품질검토는 BLOCKED이며 실제 실행은 NOT_EXECUTED입니다.
- [전체 항목 목록](../../../Docs/README.md)

## 사전준비


61 완료/복구된 same LB_OCID/LB_IP/LB_MODE/FRONTEND_CLIENT_CIDR와 web8080/backend8080/health200을 받는다.8081 listener 이름 부재 및 새8081 ingress/egress owner 승인을 확인한다. private가 기본이며 public8081/path extension은 별도 승인 branch다. 새 LB/backend/app/VM은 없다.

- [Load Balancer](00_Load_Balancer.md#실습절차) — 링크의 `실습절차`와 `결과검증` 완료; 실제 OCID/IP를 registry에 인계한다.


생성 전 `REGION`, `LAB_COMPARTMENT_ID`, 승인 종료시각 `EXPIRY_UTC`, 예산 상한과 실제 resource OCID를 실행 registry에 기록한다. owner가 후속 실습 종료까지 보존하며 이번 세션은 60분 이내 종료한다. 비용 승인/리전 capacity 없으면 실행 중단한다. shell의 변수를 다른 터미널에 자동 전달된다고 가정하지 말고 비밀값 없이 다시 설정한다.
- [Compute](../02_Compute/00_Compute.md#shared-launch)의 `vm-a` SSH/Oracle Linux 9 확인 및 [Private Subnets](../../Networking/Original-Provisional/03_Virtual_Cloud_Network_VCN/03_Private_Subnets.md#실습절차)의 private-vm/jump 접속 완료. backend는 기존 private-vm을 재사용하고 pool 트랙만 owner58이 추가 승인 VM을 만든다. `KEY`, `PUBLIC_IP`(vm-a), `BACKEND_IP`(private-vm), `CLIENT_PRIVATE_IP`(vm-a), `CLIENT_CIDR=$CLIENT_PRIVATE_IP/32`, subnet/NSG OCID를 실제 readback하여 인계한다. 테스트는 vm-a에서 수행한다; Cloud Shell은 자동 VCN private client가 아니다.

## 권한

GetLoadBalancer/GetBackendSet/GetHealthChecker/ListWorkRequests → LOAD_BALANCER_READ. Create/Update/DeleteListener, Backend/BackendSet, UpdateHealthChecker 및 Create/DeleteCertificate → LOAD_BALANCER_UPDATE.

[API operation별 공식 permission table](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/lbpolicyreference.htm) — 가족 단위 manage를 최소권한이라 부르지 않는다. 전용 lab compartment의 group만 적용하고 정책작성 권한은 관리자 소유다.
```text
Allow group <identity-domain>/<lab-group> to manage load-balancers in compartment <lab-compartment> where any {request.permission='LOAD_BALANCER_INSPECT', request.permission='LOAD_BALANCER_READ', request.permission='LOAD_BALANCER_UPDATE'}
```
이 allowlist는 위 작업만 허용한다. Console 탐색 List API가 거부되면 실제 API permission을 공식 표에서 별도로 승인하며 all-resources로 확대하지 않는다.
네트워크 생성/규칙 편집은 별도 network owner: use subnets/use vnics/use network-security-groups와 read vcns; 규칙 생성/삭제는 NETWORK_SECURITY_GROUP_UPDATE_SECURITY_RULES를 해당 NSG로 한정한다. Compute network permission table의 VNIC_CREATE/SUBNET_ATTACH/NSG membership 의존성을 확인한다. 공유 VCN 전체 manage는 최소가 아니다.

## 비용및주의사항

새 LB/NLB/VM 없이 owner61 sole LB에 HTTP8081 listener와 해당 narrow ingress/egress rule만 추가한다. 기존 LB bandwidth/시간·VM/boot·실제 요청/전송 비용과61 lease를 따른다. frontend8081은 backend8080과 별개이며80 기존 listener를 유지한다. public health-only 승인은8081/index 노출을 포함하지 않으므로 별도 승인 없으면 이 public branch는 NOT_SELECTED다. 시험 종료 후 이번 listener/rule만 삭제하고80 정상 응답을 반환한다.

## 실습구성

- 작업 목표: 61의 동일 LB에 listener 설정을 실제 변경하고 routing을 검증한다.
- 생성 owner: 61. 동일 owner의 자원을 재사용하며 실제 결과 필드는 실행 전 비워 둔다.


### 이번 workshop 시나리오와 인수·반환

같은 LB에 HTTP8081 listener를 만들고 기존80과 함께 비교한다.

- **신규/변경:** lab-http8081 listener/임시8081규칙.
- **선행 재사용:** 61LB/backend/health복구.
- **완주 흐름:** 승인/입력→본문의 생성/편집→8081health200→listener삭제8081실패+80성공→재생성200→8081listener/새규칙삭제→80정상→61 반환.
- **선택 이유/실패 경계:** frontend8081과backend8080 구분;추가 routing resource 없음.

| 입력/산출물 | 실제 값 출처 | 준비 시점 |
|---|---|---|
| `LB_OCID` | creator의 실제 생성/조회 출력; 임의 OCID/IP 금지 | 사용 전 |
| `LB_IP` | creator의 실제 생성/조회 출력; 임의 OCID/IP 금지 | 사용 전 |
| `CLIENT_CIDR` | creator의 실제 생성/조회 출력; 임의 OCID/IP 금지 | 사용 전 |

입력별 실행위치는 본문 터미널 구분을 따른다: OCI CLI=인증 Cloud Shell/승인 OCI 터미널, OS 앱/NFS/부하=지정 guest, private HTTP/TCP=vm-a guest, YAML 작성/SSH private key=본인 local. 조회/변경/정리의 관리자 승인은 **실행 조건**이며 이 문서는 IAM 성공을 입증하지 않는다. 실제 허용/거부와 capacity는 사용자 실행에서 확인한다. 미승인/NOT_SELECTED/ASYNC_PENDING을 성공으로 바꾸지 않는다.

### frontend source와 public 선택 gate
backend 직접 CLIENT_CIDR는 A_PRIVATE_IP/32다. LB_MODE=private이면 FRONTEND_CLIENT_CIDR=CLIENT_CIDR, LB_MODE=public이면 승인된 vm-a 실제 public egress IPv4/32를 FRONTEND_CLIENT_CIDR로61에서인계한다. frontend listener ingress는 이 mode별 source만 사용한다. public79 health-only HTTP 승인은8081/index확장을허용하지않는다;62 public extension은별도승인없으면NOT_SELECTED다.65의443은별도TLS/source/비용승인후진행한다. 기존mode를전환하거나두번째LB는만들지않는다.

## 실습절차

### 작업 순서 체크포인트
선택 mode/port/path 승인→기존 sole LB에8081 listener→health/80 positive→8081만 삭제 negative→같은 listener 복원→새 rule/listener 반환을 수행한다. 단계 실패는 다음 생성으로 넘기지 않고 같은 target의 복구/반환으로 처리한다.

### 실행 터미널과 지역 고정
CLI는 Cloud Shell/본인 OCI 인증 터미널에서만, `sudo`/파일 변경은 명시한 guest 터미널에서만 실행한다. 네트워크 CIDR은 기존 probe-a/probe-b 또는 10.x 예시에서 복사하지 않고 실제 VNIC/subnet 값을 조회한다.
```bash
export REGION="<approved-region>"
export OCI_CLI_REGION="$REGION"
export LAB_COMPARTMENT_ID="<actual-lab-compartment-ocid>"
export EXPIRY_UTC="<approved-session-end-UTC>"
```
치환 전 실행 금지. 생성된 OCID/IP/원래 설정/새 rule ID/owner/종료시각을 비밀 없는 registry에 기록하고 다음 터미널에서는 필요한 변수만 다시 설정한다.

[아래생성/변경단계의정확한Oracle task본문](https://docs.oracle.com/en-us/iaas/Content/Balance/Tasks/managinglisteners_topic-Creating_Listeners.htm) — Consolefield/CLI/API동작근거.

아래 단계는 Oracle task 본문을 근거로 작성한 lab recipe이며 공식 실행결과가 아니다. 각 단계의 근거는 끝의 정확한 task 링크를 따른다.

1. [Load Balancer](../../../Docs/08_Load_Balancer/00_Load_Balancer.md)의 A/B/C 단계(앱/기존 http80/장애복구) 완료와 동일 LB OCID를 확보한다. LB/backend 재생성은 하지 않는다.
2. 기본은 LB_MODE=private만 진행한다. public이면 **port8081와 요청 paths(`/health`, `/`)의 별도 노출 승인 및 owner61 제한 구성과의 양립 확인**이 없으면 여기서 NOT_SELECTED로 반환한다. 79 health-only 승인은 해당하지 않으며 restriction을 우회/원복하지 않는다. 승인 후에만 Listeners → Create에서 이름 lab-http8081, protocol HTTP, port8081, default backend set web8080을 지정한다. frontend NSG는 승인 client CIDR의 TCP8081만 임시 허용한다.
**첫8081curl 전에** vm-a egressTCP8081→LB_IP/32와 frontend ingressTCP8081 source=FRONTEND_CLIENT_CIDR를승인추가/readback하고ruleID를기록한다.

3. `curl -i http://<LB-IP>:8081/health`와 `/`를 호출한다. backend port8080과 frontend port8081이 달라도 연결되는 이유를 저장된 listener/default set으로 확인한다.
4. Rule sets는 path-route set과 다른 기능이다. 이 실습은 routing policy/path route set/rule sets=None을 명시하고 전용 listener만 추가한다. 별도 routing resource를 암시적으로 생성하지 않는다.
5. 기존 http80이 여전히 정상인지 확인한다.

### 8081 요청, 실패, 복원
같은LB_OCID/LB_IP/mode를61에서인계한다. frontendNSGingress8081와vm-aegress8081→LB_IP/32를승인한다.
```bash
curl --connect-timeout 3 --max-time 10 -i "http://$LB_IP:8081/health"
curl --connect-timeout 3 --max-time 10 -i "http://$LB_IP:8081/"
oci lb load-balancer get --load-balancer-id "$LB_OCID" --query 'data.listeners."lab-http8081"'
```
**Delete 이전 recovery gate:** owner62는 별도 승인 Console 복구세션을 실제 열고 같은 LB_OCID의 lab-http8081 전체 readback(name/protocol/port/default set/idle timeout/proxy/hostname/SSL/routing/rule sets)을 비밀 없는 registry에 저장한다. 복구담당/UTC deadline=현재+120초(lease 이내)를 등록하고 Console Listeners → Create에 저장값을 재입력할 권한을 확인한다. 별도세션/담당자/남은120초가 없으면 negative만 NOT_SELECTED로 반환한다. Delete 뒤 관측 실패/세션중단/deadline이면 별도 담당자가 즉시 동일 listener를 Create하고 list/readback 및8081 /health200을 확인한다. 이름이 이미 존재하면 덮어쓰지 않고 실제값을 대조한다. 복구 실패는 ABORTED_NOT_RECOVERED로 기록하고 owner61에 escalation하며 성공/완료로 쓰지 않는다. resource 작업 지연은120초 이내 복구 보장을 뜻하지 않는다.

장애실험은listener8081삭제후8081새요청실패/기존80health200양성대조한다. 동일name/protocolHTTP/port8081/defaultweb8080으로재생성하고200을다시확인한다. 최종정리때listener와추가규칙만삭제한다.

## 결과검증

8081 /health 200과 동일 backend 문자열, 기존 80 정상. listener 존재만으로 routing 성공을 대신하지 않는다.

- 예상 pattern이며 실제 OCI 실행 결과가 아니다. timestamp, resource OCID, 설정 readback, 실제 HTTP/TCP/파일 결과를 실행자가 기록한다.

## 문제해결

8081frontendingress/clientegress와defaultbackendset확인.8081실패/80성공은listener차이양성대조이며backend문제로단정하지않는다.

실패기록에는같은target/credentials/region/요청시간을고정한다. 복구는원래설정으로실행하고actualresponse/readback둘다확인한다;예상값을실측으로쓰지않는다.

## 리소스정리

Consolelistenerlab-http8081Delete→Listenerslist없음;새8081frontendingress/vm-aegressruleID만삭제;80health200유지확인. LBowner61보존.

정리실행자의실제UTC/OCID/readback을기록한다. 공유resource는owner확인전삭제금지; stop만으로bootvolume/LB/image비용종료가아니다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 목표는 학습 후 설명·구분·확인할 내용이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [6] [Listeners for Load Balancers — Listeners](https://docs.oracle.com/en-us/iaas/Content/Balance/Tasks/managinglisteners.htm)
  - 확인 범위: Listeners.

### 절차별 공식 task body 추가 근거
- [managinglisteners.htm](https://docs.oracle.com/en-us/iaas/Content/Balance/Tasks/managinglisteners.htm) — 본문 검토 근거; 아래 정확한 task와 작성한 입력을 구분한다. OCI 실행 증거가 아니다.



### 이번 절차의 정확한 공식 task
- [managinglisteners_topic-Creating_Listeners.htm](https://docs.oracle.com/en-us/iaas/Content/Balance/Tasks/managinglisteners_topic-Creating_Listeners.htm) — 이 문서 실습절차의 입력/동작을 대조한 본문.
