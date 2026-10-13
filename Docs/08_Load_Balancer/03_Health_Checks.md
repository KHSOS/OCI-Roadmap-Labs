# Health Checks

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../README.md)를 확인하세요.

> 절차 작성본 / OCI 미실행 — 아래 예상값은 실측 결과가 아닙니다. 비용·권한·사전준비 gate를 통과한 전용 환경에서만 실행하세요.

## 학습목표

- same LB의 HTTP health path만 /missing-health로 바꾸어 private direct404 또는 public health-only direct403/정상health200과 backend unhealthy를 대조한다.
- 120초 복구 담당자 gate 안에서 원래 health 설정을 복원하고 direct→backend health→LB 정상 응답을 기록한다.

완료 기준은 아래 실제 생성·설정·요청 결과로 판정한다. 이 목표는 수행할 작업이며 OCI에서 관찰한 결과가 아니다.

## 대응 Roadmap 항목

- 항목명: **Health Checks**
- Roadmap 경로: **Load Balancer → Health Checks**
- Node ID: `service-lb-3`
- 상위 항목: **Load Balancer** (`service-lb-0`)
- 문서 범위: 이 항목의 개념과 작성된 실습절차. 통합 품질검토는 BLOCKED이며 실제 실행은 NOT_EXECUTED입니다.
- [전체 항목 목록](../README.md)

## 사전준비


61 same LB_OCID/LB_IP/LB_MODE/BACKEND_IP/web8080·직접/LB health200과 원래 전체 HTTP checker 값을 받는다. 독립 관리자 snapshot/120초 복구 담당자가 없으면 negative branch를 선택하지 않는다. private404/public health-only403을 구분하고 기존 앱/VM/sole LB를 그대로 사용한다.

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

새 LB/NLB/VM/앱 없이 sole61 LB의 health path만 변경한다. 기존 LB/VM/boot와 제한 probe/client 요청·전송 비용을61 lease에 포함한다. 독립 담당자의 최대120초 복구 창과 전체 original health 설정을 변경 전에 확보한다. public health-only의403과 private404를 혼동하지 않는다. 오류/만료에도 original health 복원→direct200→backend healthy→LB200을 확인해야 반환할 수 있다.

## 실습구성

- 작업 목표: 동일 LB의 HTTP health path 장애와 복구를 검증한다.
- 생성 owner: 61. 동일 owner의 자원을 재사용하며 실제 결과 필드는 실행 전 비워 둔다.


### 이번 workshop 시나리오와 인수·반환

HTTP probe path만 잘못 바꾸어 앱은 정상인데 backend가 제외되는 원인을 본다.

- **신규/변경:** health path /missing-health→/health.
- **선행 재사용:** 61LB/web8080/direct정상.
- **완주 흐름:** 승인/입력→본문의 생성/편집→direct missing-health(private404/public health-only403)+directhealth200→unhealthy/LB오류→복구→120초이내원래healthpath복구→61 반환.
- **선택 이유/실패 경계:** health 지연/HTTP앱검사와TCP connect를 구분.

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

[아래생성/변경단계의정확한Oracle task본문](https://docs.oracle.com/en-us/iaas/Content/Balance/Tasks/editinghealthcheck_topic-Editing_Health_Check_Policies.htm) — Consolefield/CLI/API동작근거.

아래 단계는 Oracle task 본문을 근거로 작성한 lab recipe이며 공식 실행결과가 아니다. 각 단계의 근거는 끝의 정확한 task 링크를 따른다.

1. [Load Balancer](00_Load_Balancer.md)의 A/B/C 단계 완료 및 실제 /health 200을 확인한다.
2. Backend Sets → web8080 → Health Check에서 HTTP/8080/path /health/status200, interval10000ms/timeout3000ms/retries3을 readback한다.
3. 변경 **전에** 아래 snapshot station을 먼저 실행하여 GetHealthChecker 원래 모든 health값을저장하고 owner61과복구담당/별도Console세션/UTCdeadline=현재+120초를등록한다. 관측실패/세션중단/deadline이면담당자가즉시Console에서그원래값으로복구→GetHealthChecker→direct/LB200을확인한다. 복구세션/담당자없으면negative분기는NOT_SELECTED다. 그다음 path를 `/missing-health`로 잠시 바꾸고 backend 직접 요청의 mode-specific negative를 확인한다: LB_MODE=private이면404, owner61 public health-only이면403이다. 어떤 mode도 /health는200이어야 한다. public restriction을404를 만들려고 완화하지 않는다. 반복 probe 후 unhealthy 상태와 LB client 오류를 실제 기록한다. 운영 설정에서 하지 않는다.
`curl --connect-timeout 3 --max-time 10 -i "http://$BACKEND_IP:8080/missing-health"`로 private404/public403을 확인하고, 같은 backend :8080/health200을 양성대조한다.
4. path를 `/health`로 복원하고 direct 200→health OK→client 정상 순으로 확인한다. 단일 TCP 접속 성공과 HTTP 앱 health는 다른 검사임을 기록한다.

### 변경 전 Cloud Shell snapshot station (step3 이전)
```bash
mkdir -m 700 health64-snapshot || exit 1
(timeout -k 5s 45s oci lb health-checker get --load-balancer-id "$LB_OCID" --backend-set-name web8080 --region "$REGION" --max-retries 0 > health64-snapshot/before.json) || exit 1
python3 -c "import json; d=json.load(open('health64-snapshot/before.json'))['data']; assert d['url-path']=='/health'; print(d)" || exit 1
```
조회 실패/빈 값이면 편집 금지. 독립 복구담당자가 이 파일의 모든 원래값과 exact LB/Backend Set/120초 deadline을 보유한다. Step3의 단일 path 변경→step4의 단일 원복 후 별도 Cloud Shell에서:
```bash
timeout -k 5s 45s oci lb health-checker get --load-balancer-id "$LB_OCID" --backend-set-name web8080 --region "$REGION" --max-retries 0 > health64-snapshot/restored.json || exit 1
python3 -c "import json; assert json.load(open('health64-snapshot/before.json'))['data']==json.load(open('health64-snapshot/restored.json'))['data']" || exit 1
timeout -k 5s 45s oci lb backend-set-health get --load-balancer-id "$LB_OCID" --backend-set-name web8080 --region "$REGION" --max-retries 0 || exit 1
```
**vm-a guest 전용** (Cloud Shell fence와 분리):
```bash
curl --noproxy '*' --fail --connect-timeout 3 --max-time 10 "http://$BACKEND_IP:8080/health" || exit 1
curl --noproxy '*' --fail --connect-timeout 3 --max-time 10 "http://$LB_IP/health" || exit 1
```
health 상태 지연/timeout은 성공이 아니다. 원복 완료 후 cleanup에서 다시 편집하지 않고 saved snapshot/readback으로 owner61에게 반환한다.

## 결과검증

probe 설정 readback과 health 전이/복구. health 계산 대기 중 값은 최종 결과로 쓰지 않는다.

- 예상 pattern이며 실제 OCI 실행 결과가 아니다. timestamp, resource OCID, 설정 readback, 실제 HTTP/TCP/파일 결과를 실행자가 기록한다.

## 문제해결

healthy상태지연은interval/retries동안관찰;badpath는private direct404 또는public health-only direct403/정상path200동시대조.항상120초내originalpath복원.

실패기록에는같은target/credentials/region/요청시간을고정한다. 복구는원래설정으로실행하고actualresponse/readback둘다확인한다;예상값을실측으로쓰지않는다.

## 리소스정리

원복/readback evidence 보존이 끝나면 이번 생성한 health64-snapshot/before.json·restored.json만 regular/non-symlink/owned 확인 후 exact path로 삭제하고 빈 health64-snapshot을 rmdir한다. 실패 중에는 복구 입력을 보존하고 다른 snapshot은 건드리지 않는다.

step4 원복/readback 완료면 다시 Console 편집하지 않는다. 중단 재개만 snapshot의 원래값으로 복구→GetHealthChecker/healthOK/LB200 후 owner61 반환한다.

정리실행자의실제UTC/OCID/readback을기록한다. 공유resource는owner확인전삭제금지; stop만으로bootvolume/LB/image비용종료가아니다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 목표는 학습 후 설명·구분·확인할 내용이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [3] [Health Check Policies for Load Balancers — Health Checks](https://docs.oracle.com/en-us/iaas/Content/Balance/Tasks/load_balancer_health_management.htm)
  - 확인 범위: Health Checks.

### 절차별 공식 task body 추가 근거
- [load_balancer_health_management.htm](https://docs.oracle.com/en-us/iaas/Content/Balance/Tasks/load_balancer_health_management.htm) — 본문 검토 근거; 아래 정확한 task와 작성한 입력을 구분한다. OCI 실행 증거가 아니다.



### 이번 절차의 정확한 공식 task
- [editinghealthcheck_topic-Editing_Health_Check_Policies.htm](https://docs.oracle.com/en-us/iaas/Content/Balance/Tasks/editinghealthcheck_topic-Editing_Health_Check_Policies.htm) — 이 문서 실습절차의 입력/동작을 대조한 본문.
