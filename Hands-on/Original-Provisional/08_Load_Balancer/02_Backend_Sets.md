> **기존 실습 초안 — 미검증**: 원래 93개 문서의 절차를 보존한 자료입니다. 새 튜토리얼의 품질 승인이나 OCI 실행 검증을 뜻하지 않습니다.

# Backend Sets

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../../../Docs/README.md)를 확인하세요.

> 절차 작성본 / OCI 미실행 — 아래 예상값은 실측 결과가 아닙니다. 비용·권한·사전준비 gate를 통과한 전용 환경에서만 실행하세요.

## 학습목표

- 같은 web8080의 원래 policy/backend flags/health를 저장하고 Least Connections로 변경하여 실제 health 응답을 확인한다.
- 원래 설정을 복원하고 readback한다; 이미 승인된 두 번째 backend가 있을 때만 drain/다중응답 분기를 실행한다.

완료 기준은 아래 실제 생성·설정·요청 결과로 판정한다. 이 목표는 수행할 작업이며 OCI에서 관찰한 결과가 아니다.

## 대응 Roadmap 항목

- 항목명: **Backend Sets**
- Roadmap 경로: **Load Balancer → Backend Sets**
- Node ID: `service-lb-2`
- 상위 항목: **Load Balancer** (`service-lb-0`)
- 문서 범위: 이 항목의 개념과 작성된 실습절차. 통합 품질검토는 BLOCKED이며 실제 실행은 NOT_EXECUTED입니다.
- [전체 항목 목록](../../../Docs/README.md)

## 사전준비


61 same LB_OCID/LB_IP/LB_MODE/web8080·기존 backend IP:8080/flags와 health200을 받는다. 변경 전 전체 original policy/weight/backup/drain/offline/health/SSL snapshot을 owner61과 보관한다. 두 번째 ready backend는 별도 승인된 선택 branch이며 필수 생성이 아니다.

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

sole61 LB/web8080/backend 하나를 재사용하고 이번 변경은 balancing policy 및 승인된 flags뿐이다. 기존 LB bandwidth/시간·backend VM/boot·요청/전송 비용과61 lease를 확인한다. 두 번째 backend 분산/drain branch는 이미 별도 capacity/예산 승인을 받은 backend가 있을 때만 선택하며 이를 위해 새 VM/LB를 만들지 않는다. 한 backend로 분산/HA를 증명하지 않는다. policy/weight/drain 원복과 health/응답 readback 뒤61에 반환한다.

## 실습구성

- 작업 목표: 61의 동일 LB backend set 정책과 weight를 저장/검증한다.
- 생성 owner: 61. 동일 owner의 자원을 재사용하며 실제 결과 필드는 실행 전 비워 둔다.


### 이번 workshop 시나리오와 인수·반환

같은 web8080의 policy를 편집하고 실제응답 후 원래 policy를 돌려준다.

- **신규/변경:** policy LeastConnections→원래값;선택2backend.
- **선행 재사용:** 61LB/web8080.
- **완주 흐름:** 승인/입력→본문의 생성/편집→policy readback+HTTP응답+64실패/복구→policy/weight/drain원복→선택backend등록해제→61 반환.
- **선택 이유/실패 경계:** 1backend로분산/HA증명금지;정책변경 downtime 승인.

| 입력/산출물 | 실제 값 출처 | 준비 시점 |
|---|---|---|
| `LB_OCID` | creator의 실제 생성/조회 출력; 임의 OCID/IP 금지 | 사용 전 |
| `LB_IP` | creator의 실제 생성/조회 출력; 임의 OCID/IP 금지 | 사용 전 |
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

[아래생성/변경단계의정확한Oracle task본문](https://docs.oracle.com/en-us/iaas/Content/Balance/Tasks/managingbackendsets_topic-Editing_Backend_Sets.htm) — Consolefield/CLI/API동작근거.

아래 단계는 Oracle task 본문을 근거로 작성한 lab recipe이며 공식 실행결과가 아니다. 각 단계의 근거는 끝의 정확한 task 링크를 따른다.

1. [Load Balancer](../../../Docs/08_Load_Balancer/00_Load_Balancer.md)의 A/B/C 단계 완료 후 같은 web8080 set을 연다.
2. backend IP:8080/weight1, backup=false, drain=false를 확인한다. 기본 한 backend에서는 정책별 분산 차이를 측정할 수 없음을 명시한다.
3. **변경전에** GetBackendSet/Backend전체조회로현재policy/각backendIP:port/weight/backup/drain/offline및health/SSL설정을저장하고성공readback/복구owner/expiry를등록한다. 원래값저장실패면변경하지않는다. 변경 이전 owner61의 독립 Console 복구 담당자와 lease 이내 최대120초 UTC deadline을 지정한다. 조회/관측/세션 손실 또는 deadline에는 담당자가 저장된 전체 original policy/flags/health/SSL을 복원/readback하고 direct→backend health→LB health를 확인한다. 복구 담당자/원래 snapshot 없음은 mutation 금지다. 그다음 Console → 해당 Load Balancer → Backend Sets → web8080 → Edit에서 Load balancing policy=Least Connections를 선택한다. 기존 HTTP health/port8080/path/health/status200/SSL disabled를 보존한다. backend 세부 Edit의 Weight=1, Backup=false, Drain=false, Offline=false를 읽는다. 변경은 잠시 연결을 끊을 수 있으므로 승인된 downtime만 사용한다. set policy를 Least Connections로 변경하고 Work Request 완료 후 저장값을 조회한다. client의 새요청은항상/health로200을확인한다. /index는LB_MODE=private또는SYNTHETIC_INDEX_APPROVED=true에만10회허용하고public79health-only면NOT_SELECTED다.
4. 두 backend 비교는 **이미 별도 승인·생성·앱/네트워크 준비가 완료된 기존 두 번째 backend**의 실제owner/OCID/IP/전체runningVM예산/expiry를인계받은경우만등록한다. 이문서에서새VM을임의생성하지않고61A의기존B를다른VM이라고복제하지않는다. 해당readybackend없으면NOT_SELECTED다. 한 backend를 drain=true로 변경하고 새 연결에서 다른 backend 응답을 확인한다. 기존 연결을 자동 종료한다고 가정하지 않는다.
5. mutation 이전 저장한 **원래 policy/weight/backup/drain/offline/health 설정**으로 복원하고 health OK를 확인한다. 원래 RoundRobin/weight1/drainfalse였을때만그값을사용한다.

### 같은 backend 새요청과 제한실패복구
```bash
oci lb backend-set get --load-balancer-id "$LB_OCID" --backend-set-name web8080
if [ "$LB_MODE" = private ] || [ "${SYNTHETIC_INDEX_APPROVED:-false}" = true ]; then
  for n in $(seq 1 10); do curl --connect-timeout 3 --max-time 10 -sS "http://$LB_IP/"; done
else
  printf 'public79 index NOT_SELECTED\n'
fi
curl --connect-timeout 3 --max-time 10 -i "http://$LB_IP/health"
```
단일backend실습에서는분산/HA를주장하지않는다. 별도실패복구는64의같은sethealthpath404→unhealthy→/health복원→OK를수행한다. 두번째backend를추가하지않아도backendset의policy+health구성과실제service완주를검증한다. 원래policyreadback이RoundRobin일때만그값으로원복한다.

## 결과검증

policy readback과 실제 응답. 2backend 선택시에만 drain/분산 결과를 기록한다.

- 예상 pattern이며 실제 OCI 실행 결과가 아니다. timestamp, resource OCID, 설정 readback, 실제 HTTP/TCP/파일 결과를 실행자가 기록한다.

## 문제해결

LeastConnections변경은기존연결drop가능.1backend로분산증명못함;healthOK라도listener/clientpath를따로검증.

실패기록에는같은target/credentials/region/요청시간을고정한다. 복구는원래설정으로실행하고actualresponse/readback둘다확인한다;예상값을실측으로쓰지않는다.

## 리소스정리

정책/weight/drain원래값복원→GetBackendSet/healthOK;optional추가backend만등록해제;추가VMowner승인정리. LB삭제없음.

정리실행자의실제UTC/OCID/readback을기록한다. 공유resource는owner확인전삭제금지; stop만으로bootvolume/LB/image비용종료가아니다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 목표는 학습 후 설명·구분·확인할 내용이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [4] [Backend Sets for Load Balancers — Backend Sets](https://docs.oracle.com/en-us/iaas/Content/Balance/Tasks/managingbackendsets.htm)
  - 확인 범위: Backend Sets.

### 절차별 공식 task body 추가 근거
- [managingbackendsets.htm](https://docs.oracle.com/en-us/iaas/Content/Balance/Tasks/managingbackendsets.htm) — 본문 검토 근거; 아래 정확한 task와 작성한 입력을 구분한다. OCI 실행 증거가 아니다.



### 이번 절차의 정확한 공식 task
- [managingbackendsets_topic-Editing_Backend_Sets.htm](https://docs.oracle.com/en-us/iaas/Content/Balance/Tasks/managingbackendsets_topic-Editing_Backend_Sets.htm) — 이 문서 실습절차의 입력/동작을 대조한 본문.
