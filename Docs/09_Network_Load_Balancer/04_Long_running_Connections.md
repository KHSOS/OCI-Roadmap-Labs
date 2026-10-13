# Long-running Connections

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../README.md)를 확인하세요.

> 절차 작성본 / OCI 미실행 — 아래 예상값은 실측 결과가 아닙니다. 비용·권한·사전준비 gate를 통과한 전용 환경에서만 실행하세요.

## 학습목표

- same NLB의 한 TCP socket에 최대30회/10초 active sequence를 전송하여 exact hostname/sequence 응답을 기록한다.
- 승인 bounded server stop의 EOF/예외를 관찰하고 start/health 회복 후 신규 connection만 성공 검증하며 idle 시험으로 주장하지 않는다.

완료 기준은 아래 실제 생성·설정·요청 결과로 판정한다. 이 목표는 수행할 작업이며 OCI에서 관찰한 결과가 아니다.

## 대응 Roadmap 항목

- 항목명: **Long-running Connections**
- Roadmap 경로: **Network Load Balancer → Long-running Connections**
- Node ID: `service-nlb-4`
- 상위 항목: **Network Load Balancer** (`service-nlb-0`)
- 문서 범위: 이 항목의 개념과 작성된 실습절차. 통합 품질검토는 BLOCKED이며 실제 실행은 NOT_EXECUTED입니다.
- [전체 항목 목록](../README.md)

## 사전준비


66 same NLB_IP/OCID/actual backend hostname·TCP9090/360초 listener/health와 fresh echo positive를 받는다. vm-a active socket과 별도 B 복구 세션만 사용한다. 기존 timeout 변경은 선택 owner66 작업이며 승인 없이 idle 관찰시간을 연장하지 않는다.

- [Network Load Balancer](00_Network_Load_Balancer.md#실습절차) — 링크의 `실습절차`와 `결과검증` 완료; 실제 OCID/IP를 registry에 인계한다.


생성 전 `REGION`, `LAB_COMPARTMENT_ID`, 승인 종료시각 `EXPIRY_UTC`, 예산 상한과 실제 resource OCID를 실행 registry에 기록한다. owner가 후속 실습 종료까지 보존하며 이번 세션은 60분 이내 종료한다. 비용 승인/리전 capacity 없으면 실행 중단한다. shell의 변수를 다른 터미널에 자동 전달된다고 가정하지 말고 비밀값 없이 다시 설정한다.
- [Compute](../02_Compute/00_Compute.md#shared-launch)의 `vm-a` SSH/Oracle Linux 9 확인 및 [Private Subnets](../03_Virtual_Cloud_Network_VCN/03_Private_Subnets.md#실습절차)의 private-vm/jump 접속 완료. backend는 기존 private-vm을 재사용하고 pool 트랙만 owner58이 추가 승인 VM을 만든다. `KEY`, `PUBLIC_IP`(vm-a), `BACKEND_IP`(private-vm), `CLIENT_PRIVATE_IP`(vm-a), `CLIENT_CIDR=$CLIENT_PRIVATE_IP/32`, subnet/NSG OCID를 실제 readback하여 인계한다. 테스트는 vm-a에서 수행한다; Cloud Shell은 자동 VCN private client가 아니다.

## 권한

GetNetworkLoadBalancer/GetBackendSet/GetBackendHealth/GetHealthChecker → NETWORK_LOAD_BALANCER_READ. 이 read-only traffic 관찰 lab은 NLB 변경권한을 부여하지 않는다. listener timeout 변경이 승인된 선택시험이면 owner66이 별도 NETWORK_LOAD_BALANCER_UPDATE로 수행한다.

[API operation별 공식 permission table](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/nlbpolicyreference.htm) — 가족 단위 manage를 최소권한이라 부르지 않는다. 전용 lab compartment의 group만 적용하고 정책작성 권한은 관리자 소유다.
```text
Allow group <identity-domain>/<lab-group> to manage network-load-balancers in compartment <lab-compartment> where any {request.permission='NETWORK_LOAD_BALANCER_INSPECT', request.permission='NETWORK_LOAD_BALANCER_READ'}
```
이 allowlist는 위 작업만 허용한다. Console 탐색 List API가 거부되면 실제 API permission을 공식 표에서 별도로 승인하며 all-resources로 확대하지 않는다.
네트워크 생성/규칙 편집은 별도 network owner: use subnets/use vnics/use network-security-groups와 read vcns; 규칙 생성/삭제는 NETWORK_SECURITY_GROUP_UPDATE_SECURITY_RULES를 해당 NSG로 한정한다. Compute network permission table의 VNIC_CREATE/SUBNET_ATTACH/NSG membership 의존성을 확인한다. 공유 VCN 전체 manage는 최소가 아니다.

## 비용및주의사항

새 서비스/VM/listener 없이66 NLB와 echo9090을 재사용한다. 한 active socket 최대30회/10초(5분 이내)의 소량 데이터와 제한 stop/start만 수행하며 기존 VM/boot·NLB/전송 비용은66 lease를 따른다. heartbeat traffic은 idle 시험이 아니고 socket5초는 listener360초와 다르다. outage 후 기존 socket 자동복원을 기대하지 않고 EOF/예외→service/health 복구→새 direct/NLB 연결 성공을 확인하고 모든 stream/socket을 닫아 반환한다.

## 실습구성

- 작업 목표: 동일 NLB에서 한 TCP connection의 장기 유지와 재접속을 관찰한다.
- 생성 owner: 66. 동일 owner의 자원을 재사용하며 실제 결과 필드는 실행 전 비워 둔다.


### 이번 workshop 시나리오와 인수·반환

같은socket에서 bounded activeTCP를 유지하고 server중단후새connection만복구한다.

- **신규/변경:** 5분이내 client only;stop/start 승인.
- **선행 재사용:** 66NLB/echo/health정상.
- **완주 흐름:** 승인/입력→본문의 생성/편집→sequence hostname→EOF/예외→start/health→새direct/NLBecho→client/streamclose→serviceRunning/timeout원복→66 반환.
- **선택 이유/실패 경계:** 10초app traffic은idle시험아님;socket5초와listener360초구분.

| 입력/산출물 | 실제 값 출처 | 준비 시점 |
|---|---|---|
| `TCP_HOST=NLB_IP` | creator의 실제 생성/조회 출력; 임의 OCID/IP 금지 | 사용 전 |

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

[아래생성/변경단계의정확한Oracle task본문](https://docs.oracle.com/en-us/iaas/Content/NetworkLoadBalancer/introduction.htm) — Consolefield/CLI/API동작근거.

아래 단계는 Oracle task 본문을 근거로 작성한 lab recipe이며 공식 실행결과가 아니다. 각 단계의 근거는 끝의 정확한 task 링크를 따른다.

1. [Network Load Balancer](00_Network_Load_Balancer.md)의 A/B/C/D 완료를 사용한다. main client는 최대30회/10초·5분 이내 active traffic과 단일 backend로 제한한다. 별도 idle branch는 추가 시간/비용 승인 전 실행하지 않는다.
2. private client에서 아래를 실행한다. keepalive는 OS idle 정책과 별개이며 10초 간격 app traffic으로 idle timeout 시험을 대체하지 않는다.
```bash
export TCP_HOST='<actual-nlb-private-ip>'
export EXPECTED_BACKEND_HOSTNAME='<actual-approved-backend-hostname>'
python3 - <<'PY'
import os,socket,time
with socket.create_connection((os.environ['TCP_HOST'],9090),timeout=5) as s:
    s.setsockopt(socket.SOL_SOCKET,socket.SO_KEEPALIVE,1)
    with s.makefile("rb") as stream:
        for n in range(30):
            s.sendall(('seq=%s\n'%n).encode())
            reply=stream.readline(4096)
            expected=os.environ["EXPECTED_BACKEND_HOSTNAME"].encode()+b":"+("seq=%s\n"%n).encode()
            if reply != expected: raise RuntimeError("EOF/wrong sequence/backend; reconnect only after restore: "+repr(reply))
            print(reply.decode().strip(),flush=True)
            time.sleep(10)
PY
```
3. 동일 연결의 sequence/hostname을 기록한다. 한 backend 결과로 months-long persistence/flow HA를 입증하지 않는다.
4. 승인된 실패시험은 **먼저 아래 stop 이전 bounded 서비스 복구 station을실행하여** client진행중echo를boundedstop한다(barestop금지). client 예외/EOF를 관찰하고 start 후 새로운 connection으로 payload 성공을 확인한다. 기존 connection이 자동 복구된다고 가정하지 않는다.
5. idle timeout은 공식 NLB timeout 조건을 읽고 별도 시간/비용 승인시에만 정확한 무송신 기간으로 시험한다.

### idle 범위와 서비스 복구
30회/10초간격은5분이내의activeconnection시험이다. listenerreadbackTCPidleTimeout=360초와매request의5초sockettimeout은서로다르다. 6분idledefault를변경했다면실제설정값을기록한다. client실행중backend다른터미널에서아래bounded복구station만사용하고EOF/timeout관찰후그station의Enter/timeout자동start를확인한다;66C새연결direct→NLBpayload성공을확인한다. 기존socket자동복원은없다. stream.close()는socketcontext종료전수행한다. idle실험은별도승인없이늘리지않는다.

### stop 이전 bounded 서비스 복구 station
backend guest 별도터미널에서다음을사용하고client/OCI관측은다른터미널에서수행한다. expiry가120초보다짧으면남은승인시간만사용/미선택한다. kill9/powerloss는별도owner66복구세션이즉시start한다.
```bash
set -euo pipefail
sudo systemctl is-active --quiet oci-lab-echo || exit 1
restore_echo() { sudo systemctl start oci-lab-echo; }
trap restore_echo EXIT
trap 'exit 130' INT
trap 'exit 143' TERM
sudo systemctl stop oci-lab-echo
read -r -t 120 -p 'observe bounded failure in other terminals; Enter restores: ' answer || true
restore_echo
trap - EXIT INT TERM
sudo systemctl is-active --quiet oci-lab-echo
```
복구뒤direct새echo→NLBhealthOK→NLB새echo를66C정확payload로확인한다. barestop/start대신이station만사용한다.

## 결과검증

동일 socket sequence 응답과 강제중단 후 신규 connection recovery. 실제 유지시간을 기록하며 수개월 유지 claims를 만들지 않는다.

- 예상 pattern이며 실제 OCI 실행 결과가 아니다. timestamp, resource OCID, 설정 readback, 실제 HTTP/TCP/파일 결과를 실행자가 기록한다.

## 문제해결

app10초traffic은idle시험아님. EOF후oldsocket재사용금지;backendstart/health회복후newconnection만복구검증.

실패기록에는같은target/credentials/region/요청시간을고정한다. 복구는원래설정으로실행하고actualresponse/readback둘다확인한다;예상값을실측으로쓰지않는다.

## 리소스정리

client종료/streamclose→echo서비스start→새TCPpayload성공;listenertimeout원래값과healthOK확인. NLB66반환.

정리실행자의실제UTC/OCID/readback을기록한다. 공유resource는owner확인전삭제금지; stop만으로bootvolume/LB/image비용종료가아니다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 목표는 학습 후 설명·구분·확인할 내용이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [33] [Overview of Flexible Network Load Balancer — Long-running Connections](https://docs.oracle.com/en-us/iaas/Content/NetworkLoadBalancer/overview.htm)
  - 확인 범위: Long-running Connections.

### 절차별 공식 task body 추가 근거
- [overview.htm](https://docs.oracle.com/en-us/iaas/Content/NetworkLoadBalancer/overview.htm) — 본문 검토 근거; 아래 정확한 task와 작성한 입력을 구분한다. OCI 실행 증거가 아니다.



### 이번 절차의 정확한 공식 task
- [introduction.htm](https://docs.oracle.com/en-us/iaas/Content/NetworkLoadBalancer/introduction.htm) — 이 문서 실습절차의 입력/동작을 대조한 본문.
