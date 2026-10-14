> **기존 실습 초안 — 미검증**: 원래 93개 문서의 절차를 보존한 자료입니다. 새 튜토리얼의 품질 승인이나 OCI 실행 검증을 뜻하지 않습니다.

# Metric-based Autoscaling

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../../../Docs/README.md)를 확인하세요.

> 절차 작성본 / OCI 미실행 — 아래 예상값은 실측 결과가 아닙니다. 비용·권한·사전준비 gate를 통과한 전용 환경에서만 실행하세요.

## 학습목표

- same pool 최초 member에 승인 vCPU/360초 제한 부하를 실행하고 최근 평균 CPU와 scale-out event를 읽는다.
- 새 member의 규칙/cloud-init/health를 인수한 뒤 부하 종료·cooldown·scale-in을 기록하고 실패 시 controller disable/size1 복구한다.

완료 기준은 아래 실제 생성·설정·요청 결과로 판정한다. 이 목표는 수행할 작업이며 OCI에서 관찰한 결과가 아니다.

## 대응 Roadmap 항목

- 항목명: **Metric-based Autoscaling**
- Roadmap 경로: **Autoscaling → Metric-based Autoscaling**
- Node ID: `sequence-7-4`
- 상위 항목: **Autoscaling** (`sequence-7-0`)
- 문서 범위: 이 항목의 개념과 작성된 실습절차. 통합 품질검토는 BLOCKED이며 실제 실행은 NOT_EXECUTED입니다.
- [전체 항목 목록](../../../Docs/README.md)

## 사전준비


55 policy-saved의 actual AUTOSCALING_ID/same POOL_ID/min1/max2/cooldown300과58 모든 current member app/network-ready를 받는다. 최근 oci_computeagent CPU와 승인 APPROVED_VCPU/전체 running3대·lease를 확인한다. 신규 scaling configuration/부하 VM은 없고 실제 새 member IP/rule ID는58에서 인수한다.

- [Instance Pools](03_Instance_Pools.md#pool-ready) — 모든 current member의 cloud-init/direct health와 client path 완료.
- [Autoscaling](00_Autoscaling.md#policy-saved) — owner55 policy-saved/actual configuration OCID·same POOL_ID만 인수한다. 55의 scaling 기능완료는 본 문서 이후 반환하므로 완료를 미리 요구하지 않는다.
- Compute → 실제 pool member → Oracle Cloud Agent → **Compute Instance Monitoring** plugin Enabled/Running을 읽고 Instance → Metrics에서 namespace `oci_computeagent`, `CpuUtilization`, 같은 resourceId/최근1분 datapoints를 확인한다. [Compute Instance Monitoring 공식 본문](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/enablingmonitoring.htm)을 따른다. 기존 Monitoring80 LabHeartbeat custom metric/Alarm 생성은 이 CPU gate의 prerequisite가 아니다. datapoint가 없으면 agent/지원image/approvedegress를 고친 뒤 진행한다.


생성 전 `REGION`, `LAB_COMPARTMENT_ID`, 승인 종료시각 `EXPIRY_UTC`, 예산 상한과 실제 resource OCID를 실행 registry에 기록한다. owner가 후속 실습 종료까지 보존하며 이번 세션은 60분 이내 종료한다. 비용 승인/리전 capacity 없으면 실행 중단한다. shell의 변수를 다른 터미널에 자동 전달된다고 가정하지 말고 비밀값 없이 다시 설정한다.
- [Compute](../02_Compute/00_Compute.md#shared-launch)의 `vm-a` SSH/Oracle Linux 9 확인 및 [Private Subnets](../../Networking/Original-Provisional/03_Virtual_Cloud_Network_VCN/03_Private_Subnets.md#실습절차)의 subnet/SSH preparation만 완료; 기존private-vm 생성은pool트랙필수아니다. owner58만승인pool VM1→2대를생성한다. `KEY`, `PUBLIC_IP=A_PUBLIC_IP`, `CLIENT_PRIVATE_IP=A_PRIVATE_IP`, `CLIENT_CIDR=$CLIENT_PRIVATE_IP/32`, private subnet/NSG OCID만 생성 전에 인계한다. `POOL_ID`와 member IP는 owner58의 생성 후 실제 출력이다. 기본 platform 트랙은 `BACKEND_IP`/기존 private-vm을 요구하지 않는다. 검증은 vm-a guest에서 수행하며 Cloud Shell은 VCN private client가 아니다.

## 권한

[corepolicyreference — Permissions Required for Each API Operation / Autoscaling API Operations](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm) 실제 본문 기준이다. 선행 owner의 생성/정리 권한을 현재 사용자에게 자동 부여하지 않는다. 관리자가 exact operation과 실제 resource compartment를 대조한다.

### 관찰 API별 read-only 승인 allowlist
```text
Allow group <identity-domain>/<lab-group> to manage auto-scaling-configurations in compartment <lab-compartment> where any {request.permission='AUTO_SCALING_CONFIGURATION_READ'}
Allow group <identity-domain>/<lab-group> to manage instance-pools in compartment <lab-compartment> where any {request.permission='INSTANCE_POOL_READ'}
```
이목록은문서APItable기반의administrator검토용이며실제tenancy허용/거부시험미실행이다. IMAGE_READ는CreateInstancePoolAPI표의표기이고instance-imageverb표는INSTANCE_IMAGE_READ를쓰므로둘을임의동일하다고정규화하지않는다. bootvolume생성/termination및NSGattachment의서비스처리와실제authorization은corepolicy표/실제WorkRequest로확인한다. 후속owner생성권한을현재사용자에게자동부여하지않는다.

CPU 차트 조회는 [Monitoring API permission table](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/monitoringpolicyreference.htm)의 ListMetrics=METRIC_INSPECT, SummarizeMetricsData=METRIC_INSPECT + METRIC_READ이다. 기존 metric 수집을 읽기만 하며 PostMetricData/alarm 생성권한은 필요없다.
```text
Allow group <identity-domain>/<lab-group> to read metrics in compartment <lab-compartment>
```

## 비용및주의사항

새 autoscaling/pool/부하 VM 없이55 controller와58 pool을 재사용한다. 최초 member의 승인 vCPU 수만 최대360초 부하를 실행하며 size2의 추가 Compute/boot와 전체 running3대 비용 승인이 선행이다. CPU metric 전달 경로의 유지 비용과 동일 lease를 확인한다. 부하 종료/cooldown/scale-in 뒤 owner55 controller disable·복구 readback과 owner58 size1 반환을 기록한다. deadline 내 미복구는 성공이 아니라 BLOCKED_RECOVERY이며 무한 부하/재시도는 금지한다.

## 실습구성

- 작업 목표: 동일 autoscaling의 CPU 임계값을 실제 제한 부하로 검증한다.
- 생성 owner: autoscaling configuration/policy=55, pool/member=58; 이번 bounded workload와 결과 기록=59. 동일 owner의 자원을 재사용하며 실제 결과 필드는 실행 전 비워 둔다.


### 이번 workshop 시나리오와 인수·반환

같은 pool 최초 member에 제한 CPU부하를 걸고 scale-out/in과 새 앱을 읽는다.

- **신규/변경:** bounded workers만;새 autoscaling 없음.
- **선행 재사용:** 55policy-saved,58pool-ready,기존CPUmetric.
- **완주 흐름:** 승인/입력→본문의 생성/편집→최근평균CPU3연속/cooldown/1→2→1/새8080health→workers종료→55disable→58size1반환.
- **선택 이유/실패 경계:** 1 VM만 busy인 size2 평균은 약50%; 무제한 workers 금지.

| 입력/산출물 | 실제 값 출처 | 준비 시점 |
|---|---|---|
| `APPROVED_VCPU` | 본문 단계에서 승인/선택한 값 | 사용 전 |
| `POOL_ID` | creator의 실제 생성/조회 출력; 임의 OCID/IP 금지 | 사용 전 |
| `AUTOSCALING_ID` | creator의 실제 생성/조회 출력; 임의 OCID/IP 금지 | 사용 전 |

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

[아래생성/변경단계의정확한Oracle task본문](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/autoscalinginstancepools.htm) — Consolefield/CLI/API동작근거.

아래 단계는 Oracle task 본문을 근거로 작성한 lab recipe이며 공식 실행결과가 아니다. 각 단계의 근거는 끝의 정확한 task 링크를 따른다.

1. [Autoscaling](../../../Docs/07_Autoscaling/00_Autoscaling.md)의 동일 pool/policy 저장 단계까지 완료한다. size 1, min 1/max 2, cooldown 300초와 monitoring agent CPU metric 수집을 확인한다.
2. 아래 bounded workers절차를대신사용한다. 최초poolVM에서만최대2vCPU/360초를승인한다.
3. metric은 pool 평균 CPU이며 1분 평균 세 번 연속 조건을 충족해야 한다. scale-out 후 한 VM만 busy면 평균이 약50%가 되어 >70% 지속을 보장하지 않는다. **먼저 아래 신규 scale-out member client 경로 인수 station의 IP/rule/cloud-init handoff를 완료한 후** 신규 VM 자동 :8080/health를 검사하고 모든 승인 부하를 중단한 뒤 cooldown300초/3연속 낮은 평균을 관찰한다. CPU 차트에서 임계값 충족을 확인한다. 멀티코어 VM의 한 worker로 threshold가 안 올라가면 무제한 worker를 추가하지 말고 승인된 CPU 수/임계값을 다시 검토한다. scale event/size 2를 읽고 신규 VM /health를 검증한다.
4. timeout 후 부하가 사라졌는지 확인하고 CPU 낮음 조건/cooldown/scale in event로 size 1 복귀를 확인한다. 서비스 metric 수집/평가 대기시간을 결과에 기록한다.

### CPU bounded workers — 전체승인vCPU만
기존singleworker대신아래를pool의최초1VM에만실행한다. nproc값이승인vCPU상한보다크면중단한다. A1 1OCPU는1vCPU, E4 1OCPU는2vCPU일수있어같은worker수로70%임계값을가정하지않는다.
```bash
export APPROVED_VCPU='<approved-positive-integer>'
python3 - <<'PY'
import os,subprocess,time,signal
n=int(os.environ['APPROVED_VCPU'])
actual=int(subprocess.check_output(['nproc'],text=True))
if not 1 <= actual <= n <= 2: raise SystemExit('vCPU approval mismatch')
workers=[]
def interrupted(signum, frame):
    raise KeyboardInterrupt("approved workload interrupted")
for sig in (signal.SIGINT, signal.SIGTERM): signal.signal(sig, interrupted)
try:
    for _ in range(actual):
        workers.append(subprocess.Popen(['timeout','360s','sh','-c','while :; do :; done'], start_new_session=True))
    for p in workers: p.wait()
finally:
    for p in workers:
        if p.poll() is None:
            os.killpg(p.pid, signal.SIGTERM)
        p.wait(timeout=5)
        try: os.killpg(p.pid, 0)
        except ProcessLookupError: pass
        else: raise RuntimeError("worker process group still exists; stop before return")
PY
```
부하종료후평균3연속낮음/cooldown/size1을관찰한다. capacity실패는재시도로pool증설하지않고workrequest읽기→configurationdisable→owner58 size1복원이다.

### 신규 scale-out member client 경로 인수
scale event 후 owner58이 같은 pool list-instances/Attached VNIC에서 **실제 새 member /32**를 읽는다. network owner는58의 절차대로 vm-a nsg-admin에 Stateful egress TCP8080→새 member /32(SSH가 필요하면22도)를 승인 추가하고 exact rule ID를 owner58 registry에 기록한다. nsg-pool-app의 A_PRIVATE_IP/32 ingress 및 OS firewall을 조회한 후만 vm-a curl을 실행한다. member 종료/scale-in 뒤 owner58이 그 새 rule ID만 정리한다. 이전 member /32 규칙이 새 IP를 자동 허용한다고 가정하지 않는다.

## 결과검증

CPU metric, scale out/in event, VM 수와 앱 응답이 연결되어야 한다. 부하명령 종료만으로 scaling 성공을 주장하지 않는다.

- 예상 pattern이며 실제 OCI 실행 결과가 아니다. timestamp, resource OCID, 설정 readback, 실제 HTTP/TCP/파일 결과를 실행자가 기록한다.

## 문제해결

평균CPU3분/cooldown300/shapevCPU로인과확인. 무제한stress금지;6분내미충족은Pending이며임계값조작은별도승인설정변경으로기록.

실패기록에는같은target/credentials/region/요청시간을고정한다. 복구는원래설정으로실행하고actualresponse/readback둘다확인한다;예상값을실측으로쓰지않는다.

## 리소스정리

boundedworker모두종료→55autoscalingEnabled=false→owner58size1원복. `oci compute-management instance-pool get --instance-pool-id "$POOL_ID"`로size1. pool전체삭제는58만.

정리실행자의실제UTC/OCID/readback을기록한다. 공유resource는owner확인전삭제금지; stop만으로bootvolume/LB/image비용종료가아니다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 목표는 학습 후 설명·구분·확인할 내용이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [14] [Autoscaling — Metric-based Autoscaling](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/autoscalinginstancepools.htm)
  - 확인 범위: Metric-Based Autoscaling In metric-based / Metric-based autoscaling relies.

### 절차별 공식 task body 추가 근거
- [autoscalinginstancepools.htm](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/autoscalinginstancepools.htm) — 본문 검토 근거; 아래 정확한 task와 작성한 입력을 구분한다. OCI 실행 증거가 아니다.



### 이번 절차의 정확한 공식 task
- [autoscalinginstancepools.htm](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/autoscalinginstancepools.htm) — 이 문서 실습절차의 입력/동작을 대조한 본문.
- [autoscaling-create-metric-based_tabs.htm](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/autoscaling-create-metric-based_tabs.htm) — 이 문서 실습절차의 입력/동작을 대조한 본문.
