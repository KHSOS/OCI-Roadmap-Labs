> **기존 실습 초안 — 미검증**: 원래 93개 문서의 절차를 보존한 자료입니다. 새 튜토리얼의 품질 승인이나 OCI 실행 검증을 뜻하지 않습니다.

# Layer 3 / Layer 4

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../../../../Docs/README.md)를 확인하세요.

> 절차 작성본 / OCI 미실행 — 아래 예상값은 실측 결과가 아닙니다. 비용·권한·사전준비 gate를 통과한 전용 환경에서만 실행하세요.

## 학습목표

- same NLB TCP9090에 여러 line과 HTTP처럼 생긴 literal bytes를 보내 실제 echo/tuple를 기록한다.
- owner66 bounded stop/start 후 새 exact payload를 확인하고 sockets를 닫아 같은 서비스 running 상태를 반환한다.

완료 기준은 아래 실제 생성·설정·요청 결과로 판정한다. 이 목표는 수행할 작업이며 OCI에서 관찰한 결과가 아니다.

## 대응 Roadmap 항목

- 항목명: **Layer 3 / Layer 4**
- Roadmap 경로: **Network Load Balancer → Layer 3 / Layer 4**
- Node ID: `service-nlb-1`
- 상위 항목: **Network Load Balancer** (`service-nlb-0`)
- 문서 범위: 이 항목의 개념과 작성된 실습절차. 통합 품질검토는 BLOCKED이며 실제 실행은 NOT_EXECUTED입니다.
- [전체 항목 목록](../../../../Docs/README.md)

## 사전준비


66 nlb-ready의 same NLB_IP/OCID·TCP9090/Five Tuple/Full NAT·actual backend hostname와 새 echo 성공을 받는다. vm-a가 TCP_HOST=NLB_IP인 실제 private client이며 현재9090 규칙을 재사용한다. 새 UDP/HTTP app/VM/LB는 만들지 않는다.

- [Network Load Balancer](00_Network_Load_Balancer.md#실습절차) — 링크의 `실습절차`와 `결과검증` 완료; 실제 OCID/IP를 registry에 인계한다.


생성 전 `REGION`, `LAB_COMPARTMENT_ID`, 승인 종료시각 `EXPIRY_UTC`, 예산 상한과 실제 resource OCID를 실행 registry에 기록한다. owner가 후속 실습 종료까지 보존하며 이번 세션은 60분 이내 종료한다. 비용 승인/리전 capacity 없으면 실행 중단한다. shell의 변수를 다른 터미널에 자동 전달된다고 가정하지 말고 비밀값 없이 다시 설정한다.
- [Compute](../../../Original-Provisional/02_Compute/00_Compute.md#shared-launch)의 `vm-a` SSH/Oracle Linux 9 확인 및 [Private Subnets](../03_Virtual_Cloud_Network_VCN/03_Private_Subnets.md#실습절차)의 private-vm/jump 접속 완료. backend는 기존 private-vm을 재사용하고 pool 트랙만 owner58이 추가 승인 VM을 만든다. `KEY`, `PUBLIC_IP`(vm-a), `BACKEND_IP`(private-vm), `CLIENT_PRIVATE_IP`(vm-a), `CLIENT_CIDR=$CLIENT_PRIVATE_IP/32`, subnet/NSG OCID를 실제 readback하여 인계한다. 테스트는 vm-a에서 수행한다; Cloud Shell은 자동 VCN private client가 아니다.

## 권한

GetNetworkLoadBalancer/GetBackendSet/GetBackendHealth/GetHealthChecker → NETWORK_LOAD_BALANCER_READ. 이 read-only traffic 관찰 lab은 NLB 변경권한을 부여하지 않는다. listener timeout 변경이 승인된 선택시험이면 owner66이 별도 NETWORK_LOAD_BALANCER_UPDATE로 수행한다.

[API operation별 공식 permission table](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/nlbpolicyreference.htm) — 가족 단위 manage를 최소권한이라 부르지 않는다. 전용 lab compartment의 group만 적용하고 정책작성 권한은 관리자 소유다.
```text
Allow group <identity-domain>/<lab-group> to manage network-load-balancers in compartment <lab-compartment> where any {request.permission='NETWORK_LOAD_BALANCER_INSPECT', request.permission='NETWORK_LOAD_BALANCER_READ'}
```
이 allowlist는 위 작업만 허용한다. Console 탐색 List API가 거부되면 실제 API permission을 공식 표에서 별도로 승인하며 all-resources로 확대하지 않는다.
네트워크 생성/규칙 편집은 별도 network owner: use subnets/use vnics/use network-security-groups와 read vcns; 규칙 생성/삭제는 NETWORK_SECURITY_GROUP_UPDATE_SECURITY_RULES를 해당 NSG로 한정한다. Compute network permission table의 VNIC_CREATE/SUBNET_ATTACH/NSG membership 의존성을 확인한다. 공유 VCN 전체 manage는 최소가 아니다.

## 비용및주의사항

새 리소스/앱/listener 없이66 NLB/echo9090을 재사용하며 client socket과 소량 literal bytes만 추가한다. 기존 NLB/network transfer·VM/boot 유지 및66 lease를 확인한다. TCP가 main이고 UDP 앱이 없으면 UDP는 NOT_SELECTED다. bounded stop/start는 owner66 복구 station만 사용하며 모든 socket을 닫고 service running/새 echo 성공을 반환한다. HTTP처럼 생긴 bytes로 TLS/L7·HA·분산 성공을 주장하지 않는다.

## 실습구성

- 작업 목표: 66의 동일 NLB에서 L4 flow와 HTTP content-independent forwarding을 확인한다.
- 생성 owner: 66. 동일 owner의 자원을 재사용하며 실제 결과 필드는 실행 전 비워 둔다.


### 이번 workshop 시나리오와 인수·반환

같은 TCP9090에 HTTP처럼 생긴 bytes와 여러 line을 보내 L4 forwarding을 관찰한다.

- **신규/변경:** client sockets만.
- **선행 재사용:** 66nlb-ready.
- **완주 흐름:** 승인/입력→본문의 생성/편집→두line echo와literalHTTPbytes;66stop/start후새payload→clientclose→66running반환.
- **선택 이유/실패 경계:** UDP앱없으면UDP 제외;singlebackend affinity/HA 주장없음.

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

1. [Network Load Balancer](../../../../Docs/09_Network_Load_Balancer/00_Network_Load_Balancer.md)의 A/B/C/D 단계 완료 및 동일 OCID를 준비한다.
2. listener protocol TCP/9090, set Five Tuple를 readback한다. 66 C단계 TCP client payload를 `GET /not-http HTTP/1.0\r\n\r\n`으로 바꾸어 echo 응답을 확인한다. HTTP status를 해석/route하는 LB가 아니라 TCP bytes 전달임을 확인한다.
3. private vm-a client에서 `export TCP_HOST="<owner66-actual-NLB-private-IP>"` 및 `export EXPECTED_BACKEND_HOSTNAME="<owner66-actual-backend-hostname>"`를 먼저 설정하고 다음을 실행한다.
```bash
python3 - <<'PY'
import os,socket
for n in range(5):
    with socket.create_connection((os.environ["TCP_HOST"],9090),timeout=5) as s:
        with s.makefile("rb") as stream:
            for text in (b"line-one\n",b"line-two\n"):
                s.sendall(text)
                reply=stream.readline(4096)
                expected=os.environ["EXPECTED_BACKEND_HOSTNAME"].encode()+b":"+text
                if reply != expected: raise RuntimeError(repr(reply))
                print(n,reply.decode().strip())
PY
```
 1backend에서는 항상 같은 hostname이며 다중 backend flow affinity를 입증하지 않는다.
4. UDP 선택은 별도 UDP app/probe가 준비되지 않았으면 수행하지 않는다. TCP backend를 UDP listener에 등록하여 성공으로 처리하지 않는다.

### literal bytes와 same TCP endpoint
```bash
export TCP_HOST='<same-NLB-private-ip>'
export EXPECTED_BACKEND_HOSTNAME='<actual-approved-backend-hostname>'
python3 - <<'PY'
import os,socket
with socket.create_connection((os.environ['TCP_HOST'],9090),timeout=5) as s:
    s.sendall(b'GET /not-http HTTP/1.0\r\n\r\n')
    with s.makefile('rb') as stream:
        prefix=os.environ['EXPECTED_BACKEND_HOSTNAME'].encode()+b':'
        for expected in (prefix+b'GET /not-http HTTP/1.0\r\n',prefix+b'\r\n'):
            reply=stream.readline(4096)
            if reply != expected: raise RuntimeError('literal bytes changed/truncated/EOF: '+repr(reply))
            print(repr(reply))
PY
```
HTTP로해석되지않고lineecho두응답을기록한다. 제한실패복구는66D의echo stop/start후동일client로TCPbytes가복원되어야한다. UDP는다른protocol이므로제외한다.

## 결과검증

echo가 요청 bytes에 hostname을 붙여 반환한다. NLB listener/set 설정과 실제 TCP protocol을 함께 기록한다.

- 예상 pattern이며 실제 OCI 실행 결과가 아니다. timestamp, resource OCID, 설정 readback, 실제 HTTP/TCP/파일 결과를 실행자가 기록한다.

## 문제해결

TCP패킷경계는line경계가아님. readline(4096)/EOF로검증;echo서비스가HTTPstatus를생성한다고가정하지않는다.

실패기록에는같은target/credentials/region/요청시간을고정한다. 복구는원래설정으로실행하고actualresponse/readback둘다확인한다;예상값을실측으로쓰지않는다.

## 리소스정리

clientcontext종료;echo서비스가running인지확인하고66owner반환. 별도TCP/UDPresource삭제없음.

정리실행자의실제UTC/OCID/readback을기록한다. 공유resource는owner확인전삭제금지; stop만으로bootvolume/LB/image비용종료가아니다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 목표는 학습 후 설명·구분·확인할 내용이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [33] [Overview of Flexible Network Load Balancer — Layer 3 / Layer 4](https://docs.oracle.com/en-us/iaas/Content/NetworkLoadBalancer/overview.htm)
  - 확인 범위: Layer 3 / Layer 4.

### 절차별 공식 task body 추가 근거
- [overview.htm](https://docs.oracle.com/en-us/iaas/Content/NetworkLoadBalancer/overview.htm) — 본문 검토 근거; 아래 정확한 task와 작성한 입력을 구분한다. OCI 실행 증거가 아니다.



### 이번 절차의 정확한 공식 task
- [introduction.htm](https://docs.oracle.com/en-us/iaas/Content/NetworkLoadBalancer/introduction.htm) — 이 문서 실습절차의 입력/동작을 대조한 본문.
