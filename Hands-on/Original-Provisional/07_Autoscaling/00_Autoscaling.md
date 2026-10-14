> **기존 실습 초안 — 미검증**: 원래 93개 문서의 절차를 보존한 자료입니다. 새 튜토리얼의 품질 승인이나 OCI 실행 검증을 뜻하지 않습니다.

# Autoscaling

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../../../Docs/README.md)를 확인하세요.

> 절차 작성본 / OCI 미실행 — 아래 예상값은 실측 결과가 아닙니다. 비용·권한·사전준비 gate를 통과한 전용 환경에서만 실행하세요.

## 학습목표

- owner58의 same pool에 CPU autoscaling configuration 하나를 min1/initial1/max2로 저장/readback한다.
- 전체 running VM 3대 승인과 최근 CPU metric gate 후 59에서 실제 1→2→1/events/새 member health를 확인하고 controller를 삭제/반환한다.

완료 기준은 아래 실제 생성·설정·요청 결과로 판정한다. 이 목표는 수행할 작업이며 OCI에서 관찰한 결과가 아니다.

## 대응 Roadmap 항목

- 항목명: **Autoscaling**
- Roadmap 경로: **Autoscaling**
- Node ID: `sequence-7-0`
- 상위 항목: 없음 — 최상위 학습 항목
- 문서 범위: 이 항목의 개념과 작성된 실습절차. 통합 품질검토는 BLOCKED이며 실제 실행은 NOT_EXECUTED입니다.
- [전체 항목 목록](../../../Docs/README.md)

## 사전준비


58 pool-ready의 실제 POOL_ID/size1/current member IP·direct8080 health/hostname과57 CONFIG_ID를 받는다. CPU agent/31 SGW 경로·최근 metric datapoints, 전체 running3대 size2 승인이 있어야 기능 분기를 실행한다. configuration 저장(policy-saved)만59에 인계하고59 완료를 사전 요구하지 않는다.

- [Instance Pools](03_Instance_Pools.md#pool-ready) — 56은 capture 준비만, 58은 pool-ready 단계가 선행; 55 생성후 scaling 검증하며 image/pool 정리는 최후 owner가 수행한다.


생성 전 `REGION`, `LAB_COMPARTMENT_ID`, 승인 종료시각 `EXPIRY_UTC`, 예산 상한과 실제 resource OCID를 실행 registry에 기록한다. owner가 후속 실습 종료까지 보존하며 이번 세션은 60분 이내 종료한다. 비용 승인/리전 capacity 없으면 실행 중단한다. shell의 변수를 다른 터미널에 자동 전달된다고 가정하지 말고 비밀값 없이 다시 설정한다.
- [Compute](../02_Compute/00_Compute.md#shared-launch)의 `vm-a` SSH/Oracle Linux 9 확인 및 [Private Subnets](../../Networking/Original-Provisional/03_Virtual_Cloud_Network_VCN/03_Private_Subnets.md#실습절차)의 subnet/SSH preparation만 완료; 기존private-vm 생성은pool트랙필수아니다. owner58만승인pool VM1→2대를생성한다. `KEY`, `PUBLIC_IP=A_PUBLIC_IP`, `CLIENT_PRIVATE_IP=A_PRIVATE_IP`, `CLIENT_CIDR=$CLIENT_PRIVATE_IP/32`, private subnet/NSG OCID만 생성 전에 인계한다. `POOL_ID`와 member IP는 owner58의 생성 후 실제 출력이다. 기본 platform 트랙은 `BACKEND_IP`/기존 private-vm을 요구하지 않는다. 검증은 vm-a guest에서 수행하며 Cloud Shell은 VCN private client가 아니다.

## 권한

[corepolicyreference — Permissions Required for Each API Operation / Autoscaling API Operations](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm) 실제 본문 기준이다. 선행 owner의 생성/정리 권한을 현재 사용자에게 자동 부여하지 않는다. 관리자가 exact operation과 실제 resource compartment를 대조한다.

### 이문서 operation별 승인 allowlist
```text
Allow group <identity-domain>/<lab-group> to manage auto-scaling-configurations in compartment <lab-compartment> where any {request.permission='AUTO_SCALING_CONFIGURATION_INSPECT', request.permission='AUTO_SCALING_CONFIGURATION_CREATE', request.permission='AUTO_SCALING_CONFIGURATION_READ', request.permission='AUTO_SCALING_CONFIGURATION_UPDATE', request.permission='AUTO_SCALING_CONFIGURATION_DELETE'}
Allow group <identity-domain>/<lab-group> to manage instance-pools in compartment <lab-compartment> where any {request.permission='INSTANCE_POOL_READ', request.permission='INSTANCE_POOL_UPDATE'}
```
이목록은문서APItable기반의administrator검토용이며실제tenancy허용/거부시험미실행이다. IMAGE_READ는CreateInstancePoolAPI표의표기이고instance-imageverb표는INSTANCE_IMAGE_READ를쓰므로둘을임의동일하다고정규화하지않는다. bootvolume생성/termination및NSGattachment의서비스처리와실제authorization은corepolicy표/실제WorkRequest로확인한다. 후속owner생성권한을현재사용자에게자동부여하지않는다.

CPU 차트 조회는 [Monitoring API permission table](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/monitoringpolicyreference.htm)의 ListMetrics=METRIC_INSPECT, SummarizeMetricsData=METRIC_INSPECT + METRIC_READ이다. 기존 metric 수집을 읽기만 하며 PostMetricData/alarm 생성권한은 필요없다.
```text
Allow group <identity-domain>/<lab-group> to read metrics in compartment <lab-compartment>
```

## 비용및주의사항

owner58의 pool 하나에 autoscaling configuration 하나만 생성한다. min1/initial1/max2라도 size2는 vm-a+pool2=전체 running3대 및 추가 Compute/boot 비용을 별도 승인한 경우만 실행한다. 새 pool/부하 VM/LB는 만들지 않는다. metric 전달용31 SGW/route/rule 및 기존 VM/network의 유지 비용을 실제 선택 범위에서 승인한다. lease 만료 전 controller disable/delete와 association 부재를 확인한 뒤 pool을 반환한다; pool stop만으로 controller 정리를 대신하지 않는다.

## 실습구성

- 작업 목표: 하나의 pool과 bounded autoscaling 구성으로 실제 확장/축소를 수행한다.
- 생성 owner: autoscaling configuration=55, pool/VM=58. 동일 owner의 자원을 재사용하며 실제 결과 필드는 실행 전 비워 둔다.


### 이번 workshop 시나리오와 인수·반환

pool 한 개에 CPU autoscaling configuration을 만들고 실제 1→2→1을 검사한다.

- **신규/변경:** autoscaling configuration1개/metric policy.
- **선행 재사용:** 57 configuration-ready,58 pool-ready.
- **완주 흐름:** 승인/입력→본문의 생성/편집→최근CPU datapoints+scale events+member HTTP8080→59 반환 후 disable/delete→58 pool 반환.
- **선택 이유/실패 경계:** CPU agent/egress 및 min1/max2/cooldown300; config저장은 scale 성공 아니다.

| 입력/산출물 | 실제 값 출처 | 준비 시점 |
|---|---|---|
| `POOL_ID` | creator의 실제 생성/조회 출력; 임의 OCID/IP 금지 | 사용 전 |
| `AUTOSCALING_ID` | creator의 실제 생성/조회 출력; 임의 OCID/IP 금지 | 생성 후 |
| `CLIENT_CIDR` | creator의 실제 생성/조회 출력; 임의 OCID/IP 금지 | 사용 전 |

입력별 실행위치는 본문 터미널 구분을 따른다: OCI CLI=인증 Cloud Shell/승인 OCI 터미널, OS 앱/NFS/부하=지정 guest, private HTTP/TCP=vm-a guest, YAML 작성/SSH private key=본인 local. 조회/변경/정리의 관리자 승인은 **실행 조건**이며 이 문서는 IAM 성공을 입증하지 않는다. 실제 허용/거부와 capacity는 사용자 실행에서 확인한다. 미승인/NOT_SELECTED/ASYNC_PENDING을 성공으로 바꾸지 않는다.

### Private pool CPU metric 전달 gate
[Enabling Monitoring 공식 본문](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/enablingmonitoring.htm)은 public IP 없는 instance에 Service Gateway 준비를 요구한다. metric55/59 선택이면 network owner31은 기존 `sgw-lab`를 조회/재사용하거나 같은 VCN에 하나만 Create하며 service label **All <actual Region> Services in Oracle Services Network**를 선택한다. private-app 실제 route table에 Destination Service=같은 All-services label, Target=SGW_OCID를 추가하고, nsg-pool-app Stateful egress TCP443 destination type=Service CIDR/동일 label을 승인한다. repository NAT default route와 더 구체적인 service route를 구분한다. SGW-only 실험 전체/기존 B 생성은 prerequisite가 아니다. configuration의 Oracle Cloud Agent에서 Compute Instance Monitoring=Enabled/Running, plugins enabled를 설정하고 guest/Console의 최근 `oci_computeagent` CPU datapoint를 읽는다. 관리자 승인/실제 datapoint 없는 metric 실험은 pending이다. 31 owner registry에 pool/metric consumer와expiry를등록하고 마지막 consumer 반환 뒤 새 route/rule ID를제거→미참조 gateway만삭제한다. publicIP를pool에추가하여우회하지않는다.

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

[아래생성/변경단계의정확한Oracle task본문](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/autoscaling-create-metric-based_tabs.htm) — Consolefield/CLI/API동작근거.

아래 단계는 Oracle task 본문을 근거로 작성한 lab recipe이며 공식 실행결과가 아니다. 각 단계의 근거는 끝의 정확한 task 링크를 따른다.

1. [Instance Configurations](../../../Docs/07_Autoscaling/02_Instance_Configurations.md)의 configuration 생성/readback과 [Instance Pools](../../../Docs/07_Autoscaling/03_Instance_Pools.md)의 size 1 및 앱 응답 검증까지 완료한다. 별도 pool을 생성하지 않는다.
2. **Create 이전 sole-controller gate:** same POOL_ID의 현재 autoscaling association을 관리자 Console에서 확인한다. 기존55/60 configuration이 남으면 exact owner의 삭제/association 부재 반환까지 중단한다. disable만으로 연결 해제를 주장하지 않는다. 그 다음 Console → Compute → Autoscaling Configurations → Create autoscaling configuration → Add basic details: Name=`lab-cpu-autoscaling`, configuration Compartment=승인 LAB_COMPARTMENT_ID, pool Compartment=owner58 실제 scope, Instance pool=actual POOL_ID, tags=미선택. 이름/풀/소유권을 구분하고 size2의 전체 running3대/추가 Compute·boot 비용 승인을 재확인한다.
3. Next → Configure autoscaling policy → **Metric-based autoscaling**, policy Name=`lab-cpu-policy`, Cooldown=300s, Performance metric=CPU utilization을 선택한다. Scale-out operator=`>`/threshold70/Number to add1, Scale-in operator=`<`/threshold20/Number to remove1; Minimum1/Initial1/Maximum2를 입력한다. Next → Review에서 same POOL_ID/quantity/sole controller/expiry를 재대조하고 Create를 한 번만 제출한다. actual configuration/policy readback 실패면 재Create하지 않고 owner55 복구/정리한다. CPU 임계값은 실습 입력값이며 공식 권장값을 뜻하지 않는다.

### policy-saved

**policy-saved:** 이 시점은 configuration/policy actual 저장과 same POOL_ID readback만 완료한다. 59 기능완료를 선행 요구하지 않는다.

4. [Metric-based Autoscaling](../../../Docs/07_Autoscaling/04_Metric_based_Autoscaling.md)의 제한 부하 절차로 실행한 뒤 pool size 변화와 새 VM 앱 응답을 확인한다. 충분한 metric 시간창이 없으면 pending으로 기록한다.
5. 부하를 중지하고 축소를 확인한다. autoscaling enabled/config/policy 값을 다시 읽고 최대 2를 유지한다.

### 설정·pool 실제 readback
```bash
export AUTOSCALING_ID='<created-autoscaling-ocid>'
export POOL_ID='<owner58-pool-ocid>'
oci autoscaling configuration get --auto-scaling-configuration-id "$AUTOSCALING_ID"
oci compute-management instance-pool get --instance-pool-id "$POOL_ID"
oci compute-management instance-pool list-instances --instance-pool-id "$POOL_ID" --compartment-id "$LAB_COMPARTMENT_ID"
```
configuration resource.id가같은 POOL_ID인지확인한다. Compute instance → Oracle Cloud Agent → Compute Instance Monitoring plugin=Enabled/Running, Monitoring CPU utilization 최근 datapoints가 있어야 한다. metric을임의주입하여 autoscaling성공대체하지않는다. 59완료후 owner55가 configuration disable/delete하고 owner58이 pool을종료한다.

## 결과검증

policy 저장만으로 합격하지 않는다. size 1→2→1과 두 번째 VM 생성/삭제 이벤트 및 앱 응답을 실제 확인한다.

- 예상 pattern이며 실제 OCI 실행 결과가 아니다. timestamp, resource OCID, 설정 readback, 실제 HTTP/TCP/파일 결과를 실행자가 기록한다.

## 문제해결

최근CPUmetric없으면agentplugin지원/Enabled/Running/egress확인. config추가전다른autoscaling같은pool연결없음확인;duplicatepaidpool금지.

실패기록에는같은target/credentials/region/요청시간을고정한다. 복구는원래설정으로실행하고actualresponse/readback둘다확인한다;예상값을실측으로쓰지않는다.

## 리소스정리

ConsoleautoscalingconfigurationEnabled=false후readback; configurationDelete후list없음확인. pool은owner58에게반환; instance configuration은owner57, image는56만삭제.

정리실행자의실제UTC/OCID/readback을기록한다. 공유resource는owner확인전삭제금지; stop만으로bootvolume/LB/image비용종료가아니다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 목표는 학습 후 설명·구분·확인할 내용이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [14] [Autoscaling — Autoscaling](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/autoscalinginstancepools.htm)
  - 확인 범위: Autoscaling lets you.

### 절차별 공식 task body 추가 근거
- [autoscalinginstancepools.htm](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/autoscalinginstancepools.htm) — 본문 검토 근거; 아래 정확한 task와 작성한 입력을 구분한다. OCI 실행 증거가 아니다.



### 이번 절차의 정확한 공식 task
- [autoscaling-create-metric-based_tabs.htm](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/autoscaling-create-metric-based_tabs.htm) — 이 문서 실습절차의 입력/동작을 대조한 본문.
