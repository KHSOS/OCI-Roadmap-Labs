> **기존 실습 초안 — 미검증**: 원래 93개 문서의 절차를 보존한 자료입니다. 새 튜토리얼의 품질 승인이나 OCI 실행 검증을 뜻하지 않습니다.

# Schedule-based Autoscaling

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../../../Docs/README.md)를 확인하세요.

> 절차 작성본 / OCI 미실행 — 아래 예상값은 실측 결과가 아닙니다. 비용·권한·사전준비 gate를 통과한 전용 환경에서만 실행하세요.

## 학습목표

- 기존 metric controller 삭제/readback 뒤 same pool에 미래 UTC one-time out2/in1 schedule 두 policy를 저장한다.
- forecast와 실제 events/새 member health/size1 복귀를 대조하고 schedule configuration 삭제 후 예약 없는 pool을 반환한다.

완료 기준은 아래 실제 생성·설정·요청 결과로 판정한다. 이 목표는 수행할 작업이며 OCI에서 관찰한 결과가 아니다.

## 대응 Roadmap 항목

- 항목명: **Schedule-based Autoscaling**
- Roadmap 경로: **Autoscaling → Schedule-based Autoscaling**
- Node ID: `sequence-7-5`
- 상위 항목: **Autoscaling** (`sequence-7-0`)
- 문서 범위: 이 항목의 개념과 작성된 실습절차. 통합 품질검토는 BLOCKED이며 실제 실행은 NOT_EXECUTED입니다.
- [전체 항목 목록](../../../Docs/README.md)

## 사전준비


58 pool-ready actual POOL_ID/size1/member health와55/59 반환을 받는다. 선택 metric controller는 owner55 삭제·same pool association 부재 readback이 필수다. 승인 미래 UTC_OUT/UTC_IN/EXPIRY_UTC와 전체 running3대 capacity를 확보하고 새 SCHEDULE_ID를 만든다. metric과 schedule을 동시에 연결하지 않는다.

- [Instance Pools](03_Instance_Pools.md#pool-ready) — 링크의 `실습절차`와 `결과검증` 완료; 실제 OCID/IP를 registry에 인계한다.


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

## 비용및주의사항

같은58 pool에 schedule configuration 하나/one-time policy 두 개만 생성한다. out2 기간의 추가 Compute/boot와 vm-a 포함 전체 running3대를 승인하고 out/in UTC 모두 lease 안에 둔다.55 metric configuration은 disable만 하지 말고 owner 삭제/association 부재를 받은 뒤 생성한다. 반복 schedule/두 controller/새 pool/VM/LB는 만들지 않는다. 마지막 event/size1/새 member health 확인 후 schedule configuration 삭제·예약 부재 readback을 하고 pool을 반환한다.

## 실습구성

- 작업 목표: 동일 pool에 bounded UTC schedule을 저장하고 실제 시각에 동작시킨다.
- 생성 owner: 60. 동일 owner의 자원을 재사용하며 실제 결과 필드는 실행 전 비워 둔다.


### 이번 workshop 시나리오와 인수·반환

같은 pool에 미래 UTC out/in 일회성 스케줄을 저장하고 2→1을 관찰한다.

- **신규/변경:** schedule configuration **1개**, 그 안의 one-time policy **2개**(out2/in1).
- **선행 재사용:** 58pool-ready;55metric config 삭제/반환.
- **완주 흐름:** 승인/입력→본문의 생성/편집→Quartz forecast와actual event/size2/새health/size1→schedule disable/delete→58size1→55선택재개.
- **선택 이유/실패 경계:** 지난날짜/시간대/반복잔존 금지; controller 두개 중복 금지.

| 입력/산출물 | 실제 값 출처 | 준비 시점 |
|---|---|---|
| `POOL_ID` | creator의 실제 생성/조회 출력; 임의 OCID/IP 금지 | 사용 전 |
| `UTC_OUT` | 본문 단계에서 승인/선택한 값 | 사용 전 |
| `UTC_IN` | 본문 단계에서 승인/선택한 값 | 사용 전 |
| `SCHEDULE_ID` | creator의 실제 생성/조회 출력; 임의 OCID/IP 금지 | 생성 후 |

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

[아래생성/변경단계의정확한Oracle task본문](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/autoscaling-create-schedule-based_tabs.htm) — Consolefield/CLI/API동작근거.

아래 단계는 Oracle task 본문을 근거로 작성한 lab recipe이며 공식 실행결과가 아니다. 각 단계의 근거는 끝의 정확한 task 링크를 따른다.

1. [Instance Pools](../../../Docs/07_Autoscaling/03_Instance_Pools.md)의 size 1/앱 검증까지 완료한다. owner55의 metric configuration을 disable/delete하고 실제 반환/list없음을 확인한 뒤 동일 pool에 schedule configuration 하나를 생성한다. 저장한 metric 설정은 나중에55 승인 후 재생성하며 두 configuration을 동시에 연결하지 않는다. 중복 제어는 금지한다.
2. Console → Compute → Autoscaling configurations → Create autoscaling configuration → Add basic details: Name=`lab-once-schedule`, configuration Compartment=승인 LAB_COMPARTMENT_ID, pool Compartment=owner58 실제 scope, Instance pool=actual POOL_ID, tags=미선택. Next → Configure autoscaling policy → Schedule-based autoscaling을 선택한다. schedule UI에는 metric scaling limits와 같은 minimum/maximum을 입력하지 않는다. policy1 이름 lab-once-out/Action=Scale pool size/Target pool size=2, Another Policy 이름 lab-once-in/Target=1을 입력한다. 두 target은 비용 상한2를 유지한다. 현재 UTC와 실습 종료시각을 실제 확인하고 미래 scale-out 시각/scale-in 시각을 승인한다.
3. UI의 cron 필드 의미를 공식 schedule 본문과 대조하여 UTC 일시를 입력한다. 예시는 Quartz 형식 `0 15 9 ? * MON *`이며 매주 월요일 09:15 UTC이다. 이를 그대로 실행하지 말고 승인한 미래 날짜에 맞춘 one-time 연도/날짜 표현식을 입력한다. out target 2, in target 1로 구성하고 preview next run을 검증한다.
4. 해당 UTC까지 승인된 세션만 유지하고 pool size2 뒤 **먼저 아래 신규 scale-out member client 경로 인수 station의 IP/rule/cloud-init handoff를 완료한 후** 새 VM /health를 확인한다. deadline내준비가끝나지않으면schedule disable/owner58size1복구로중단한다. in schedule 이후 size 1을 확인한다. 예약 저장만으로 실제 실행 완료가 아니다.
5. 반복 schedule은 즉시 disable/delete한다. 예약을 남겨 종료 후 재생성되게 하지 않는다.

### 실제미래UTC Quartz 생성
로컬clock는정확한UTC로동기되어야한다. 생성+10분/30분예시는expiry60분안에서승인후사용한다. poolbootstrap이30분이상걸리면schedule실습중단한다.
```bash
python3 - <<'PY'
from datetime import datetime,timedelta,timezone
now=datetime.now(timezone.utc)
for label,minutes in [('out',10),('in',30)]:
    t=(now+timedelta(minutes=minutes)).replace(second=0,microsecond=0)
    expiry=datetime.fromisoformat(__import__("os").environ["EXPIRY_UTC"].replace("Z","+00:00"))
    if expiry.tzinfo is None: raise SystemExit("expiry must include UTC timezone")
    if not now < t < expiry-timedelta(minutes=5): raise SystemExit("schedule outside approved deadline/recovery margin")
    print(label,t.isoformat(),f'0 {t.minute} {t.hour} {t.day} {t.month} ? {t.year}')
PY
```
out/in expression을 각각 Execution schedule에 붙이고 Next → Review에서 same POOL_ID/one-time 두 policy/UTC future forecast/out2/in1/lease를 확인한다. Create를 한 번만 제출하고 actual SCHEDULE_ID 및 두 저장 expression을 조회/readback한다. 오류면 재Create하지 않고 owner60 원복/정리한다. forecast UTC를 실제 승인 UTC와 비교한다. 지난스케줄은성공하지않는다. deadline도달하면scheduleconfigurationdisable후owner58이poolsize1로manual복원하며두schedule을삭제한다.

### 신규 scale-out member client 경로 인수
scale event 후 owner58이 같은 pool list-instances/Attached VNIC에서 **실제 새 member /32**를 읽는다. network owner는58의 절차대로 vm-a nsg-admin에 Stateful egress TCP8080→새 member /32(SSH가 필요하면22도)를 승인 추가하고 exact rule ID를 owner58 registry에 기록한다. nsg-pool-app의 A_PRIVATE_IP/32 ingress 및 OS firewall을 조회한 후만 vm-a curl을 실행한다. member 종료/scale-in 뒤 owner58이 그 새 rule ID만 정리한다. 이전 member /32 규칙이 새 IP를 자동 허용한다고 가정하지 않는다.

## 결과검증

next execution UTC, 실제 scale 이벤트/size 2→1. 시간대 오류와 지난 일정은 실행 성공이 아니다.

- 예상 pattern이며 실제 OCI 실행 결과가 아니다. timestamp, resource OCID, 설정 readback, 실제 HTTP/TCP/파일 결과를 실행자가 기록한다.

## 문제해결

Quartz7field와UTCyear/day/month를확인. metric/schedule중복제어금지;delay시deadline에disable/manualsize1복구.

실패기록에는같은target/credentials/region/요청시간을고정한다. 복구는원래설정으로실행하고actualresponse/readback둘다확인한다;예상값을실측으로쓰지않는다.

## 리소스정리

scheduleconfigurationdisable/delete와forecast예약없음확인→owner58 `oci compute-management instance-pool update --instance-pool-id "$POOL_ID" --size 1`→Running/size1. metric configuration은 이전 삭제 상태를 유지하며, 재개가 필요하면55의 별도 승인과 같은 pool sole-controller 조건으로 재생성한다. 삭제된 configuration을 enable하려 하지 않는다.

정리실행자의실제UTC/OCID/readback을기록한다. 공유resource는owner확인전삭제금지; stop만으로bootvolume/LB/image비용종료가아니다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 목표는 학습 후 설명·구분·확인할 내용이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [14] [Autoscaling — Schedule-based Autoscaling](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/autoscalinginstancepools.htm)
  - 확인 범위: Schedule-Based Autoscaling You can use / Provide all times in UTC..

### 절차별 공식 task body 추가 근거
- [autoscalinginstancepools.htm](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/autoscalinginstancepools.htm) — 본문 검토 근거; 아래 정확한 task와 작성한 입력을 구분한다. OCI 실행 증거가 아니다.



### 이번 절차의 정확한 공식 task
- [autoscaling-create-schedule-based_tabs.htm](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/autoscaling-create-schedule-based_tabs.htm) — 이 문서 실습절차의 입력/동작을 대조한 본문.
