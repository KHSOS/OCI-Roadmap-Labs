# Traffic Management

> 문서 작성 검토와 실행 검증은 별개입니다. OCI 실행은 **USER_EXECUTION_PENDING**이며 사용자 캡처는 별도 자료입니다. 미실행 자체는 문서 작성 승인 차단 사유가 아닙니다. 실제 결과 필드는 실행자가 채웁니다.

> 실행용 한국어 교육 문서. OCI 리소스 생성·명령 실행 결과는 아직 검증하지 않았으며 모든 기대 패턴은 미실행이다.

## 학습목표

- 추가 LB 없이 승인된 기존 LB IPv4 하나로 public Primary zone, Load Balancer steering policy 및 app attachment를 각1개 생성한다.
- 실제 authoritative NS 직접 질의의 A/TTL을 LB IP와 비교하고 VCN client의 HTTP --resolve /health 응답을 별도로 확인한다.
- 단일 answer Eligible 해제의 무응답과 재활성·TTL60 복구를 기록하고 attachment→policy→zone 순서로 삭제한다. weighted 분산·public recursive 위임은 완료로 주장하지 않는다.

실제 OCI 요청·응답/SQL·정리 증거는 실행자가 기록한다. 아래 기대값은 실측값이 아니다.

## 대응 Roadmap 항목

- 항목명: **Traffic Management**
- Roadmap 경로: **DNS → Traffic Management**
- Node ID: `sequence-8-2`
- 상위 항목: **DNS** (`sequence-8-0`)
- 문서 범위: 이 항목의 개념, 실제 설정 절차와 학습자 실행 검증.
- [전체 항목 목록](../README.md)

## 사전준비

**시작 조건:** 61 sole HTTP LB80와 /health200 완료;WAF branch 선택은 필수 아님

- [Load Balancer](../08_Load_Balancer/00_Load_Balancer.md) — 61의 실제 HTTP LB :80과 /health 200; 기본 private LB는 VCN 내부 client에서 검증 완료. LB 유지.

### 준비 완료 출력과 인수 확인

| 인수 항목 | 실제 출력·출처 | 다음 작업의 사용 위치 |
|---|---|---|
| 선행 완료 상태 | 61 sole HTTP LB80와 /health200 완료;WAF branch 선택은 필수 아님 | 첫 변경 전 재조회 |
| 비밀 없는 실행 입력 | actual LB OCID/IP/mode, 승인 public-zone IP노출 범위, vm-a HTTP host/외부DNS host, owner61 expiry | 아래 numbered Console/명령 단계 |
| 반환/후속 인계 | attachment/policy/public zone 정리 후61 LB 반환 | 종료 UTC 이전 exact owner 반환 |

실제 Compartment/Region은 승인 Console에서 읽고 동일 identity/Region을 고정한다. DNS host에서 `command -v dig`, HTTP VCN client에서 `command -v curl`을 확인한다. placeholder·명칭만 인수한 상태는 준비 완료가 아니다. 비밀번호·키·확인 token은 출력표에 적지 않는다.

## 권한

전용 Compartment의 관리자 승인 정책 예:
```text
Allow group <LabGroup> to manage dns in compartment <LabCompartment>
```
Create/Get/Update/DeleteSteeringPolicy=DNS_STEERING_POLICY_CREATE/READ/UPDATE/DELETE. attachment Create/Update/Delete=DNS_ZONE_UPDATE+DNS_STEERING_POLICY_READ이며 attachment resource manage 하나로 해결하지 않는다. GetSteeringPolicyAttachment=DNS_STEERING_ATTACHMENT_READ. Public zone create/delete=DNS_ZONE_CREATE/DELETE. Health Check가 None이므로 health-check-family 생성권한은 이 과제에서 사용하지 않는다.

operation 근거: [공식 권한/작업 본문](https://docs.oracle.com/en-us/iaas/Content/Identity/policyreference/dnspolicyreference.htm). family grant는 minimum-permission 정책으로 주장하지 않는다. 실행 전에 관리자 승인된 identity/domain·실제 Compartment scope를 확인한다.

## 비용및주의사항

public Primary zone·steering policy·attachment 각1개만 새로 만들며 DNS/Traffic Management의 현재 가격과 query·정책 유지 조건을 별도 승인한다. public zone에 lab private IP를 게시하는 노출도 승인 대상이다. 기존 sole LB의 보존비는61 owner에게 귀속하며 추가 endpoint/LB는 만들지 않는다. domain 구매·등록기관 delegation 없이 authoritative NS 직접 질의만 하므로 internet recursive resolution/weighted 분산을 완료로 기록하지 않는다. attachment→policy→zone 삭제 후 잔존을 확인한다.

## 실습구성

### 워크숍 순서와 입력값

**진행 순서:** 비용/IP노출 승인 → public authoritative zone → 단일A steering/attachment →직접NS질의/HTTP →eligibility복구 →삭제

**선택 이유:** 단일 endpoint는 policy plumbing만 검증하며 weighted분산/recursive위임을 주장하지 않는다.

| 입력 | 값 또는 정확한 출처 | 실행 위치 | 생성 전/후 | 이유 |
|---|---|---|---|---|
| LB_IP/AUTH_NS | 61실제LB IP / 생성zone 실제NS | vm-a HTTP·외부shell DNS | 후 | privateHTTP와publicNS 경로구분 |
| zone/policy/answer | roadmap-lab.invalid /roadmap-steering /lb | Console | 전 | 단일answer·위임없음 |


값이 다른 터미널에 자동 전파되지 않습니다. Console readback으로 비밀 없는 값만 인계하며, placeholder는 실제값으로 교체한 뒤 실행합니다. 새 리소스 생성 전 이름 충돌·소유자·최대 비용·종료 UTC를 확인하고 선택하지 않은 분기는 **NOT_SELECTED**로 기록합니다.


분기 실습: roadmap-lab.invalid public zone의 app A에 Load Balancer steering policy를 attach한다. 실제 backend endpoint는 기존 실제 LB IPv4 하나만 사용하며, 두 번째 유료 LB를 만들지 않는다.


### 이번 실습의 생성·재사용·앱 경계

| 구분 | 이번 주제의 실제 범위 |
|---|---|
| 새로 생성 | public Primary zone/steering policy/app attachment |
| 재사용 | 61 soleLB와기존nginx /health |
| 앱/작업부하 및 변경 | 단일A answer/Eligible/TTL 변경·복구;별도앱 설치/Health Check 없음 |
| 실행/반환 순서 | 61 sole HTTP LB80와 /health200 완료;WAF branch 선택은 필수 아님 → attachment/policy/public zone 정리 후61 LB 반환 |

새로 생성하지 않는 항목을선행완료로가장하지않는다.위표의실제OCID/주소와진행순서를첫mutation전에확인한다.

## 실습절차

### 공개 zone과 private endpoint의 차이
`roadmap-lab.invalid`은 공개 authoritative zone에 **private LB IPv4를 게시할 수 있는 학습 범위**이며 private IP 노출도 승인한다. 회사 내부 IP를 게시하지 않는다. public zone 생성/steering 비용 및 authoritative NS UDP/TCP53 출구를 사전에 승인한다. internet DNS 경로가 없는 private client는 승인된 외부 shell에서 authoritative dig만 수행하고 HTTP --resolve 시험은 VCN client에서 별도로 수행한다.
1 answer 검증은 weighted 분산·failover 검증이 아니다. answer Eligible 해제→authoritative 빈 응답 확인→Eligible 복구→동일 A 응답 확인을 선택 비파괴 실패·회복 과제로 수행한다. 정책 편집이 완료된 후 1회씩 확인하고 최대 2분 내 반영 안 되면 실패로 기록한다. attachment의 zone OCID/domain/policy OCID를 삭제 전 읽어 잘못된 zone을 삭제하지 않는다.

### 실행 단계

1. DNS → Public zones → Create primary zone. Name **roadmap-lab.invalid**, Compartment **실습 Compartment**. `.invalid`은 학습용으로 실제 등록·delegation하지 않는다. Create review에서 DNS scope **Public**, Zone type **Primary**를 확인하고 생성 후 Zone information의 같은 값과 actual zone OCID/NS record names를 기록한다. private zone 76에는 public steering policy를 붙이지 않는다.
2. Traffic Management → Steering policies → Create steering policy. Policy type **Load Balancer**, Name **roadmap-steering**, TTL **60**. Answer pool **primary**, Answer name **lb**, Type **A**, Rdata **61의 실제 LB IPv4 (기본 private)**, Weight **1**, Eligible **checked**, Maximum answer count **1**, Attach health check **None**. 단일 실제 endpoint를 쓰는 이유는 중복 LB 비용 없이 policy/attachment/응답 경로를 검증하기 위해서다. 다중 endpoint weighted 분배를 검증한 것으로 기록하지 않는다.[T1]
3. Attach policy. policy → **Attached domains** → **Add Attached Domain(s)**. Zone **roadmap-lab.invalid**, Domain **app.roadmap-lab.invalid**. **Add → View changes → Publish → Publish changes**까지 완료하고 attachment zone/domain/policy OCID를 다시 읽는다. apex가 아닌 별도 이름을 선택하고 같은 이름의 고정 A/CNAME RRset과 충돌하지 않게 한다.[T2]
4. VCN 내부 승인 client에서 Console NS 중 하나를 실제 값으로 설정하여 authoritative 질의한다. NS로 나가는 DNS 접근도 선행 network 경로가 필요하다. 기본 private LB A 응답은 DNS record 검증이며 internet-accessible 앱을 뜻하지 않는다.
```bash
export LB_IP='61의 실제 LB IPv4'
export AUTH_NS='Console에 표시된 실제 NS hostname'
dig @"$AUTH_NS" app.roadmap-lab.invalid A +norecurse +comments +answer
curl --max-time 10 --resolve "app.roadmap-lab.invalid:80:$LB_IP"   http://app.roadmap-lab.invalid/health
```
5. 실제 반환 rdata가 LB_IP인지, curl body 61의 ok인지 기록한다. --resolve는 client DNS override이므로 recursive DNS 성공 증거가 아니며 두 검증을 구별한다. TTL을 120으로 Edit한 뒤 authoritative query TTL 변화를 1회 확인한다.


### 진행·종료 판정
설정 readback·기능시험·원복·정리를 각각 실제 응답으로 기록한다. 기대값은 미실행이며 실패/만료 시 새 mutation을 중단하고 아래 topic별 종료 단계로 반환한다.

### Steering eligibility failure recovery and TTL reset
선택 비파괴 실패시험은 Steering policy → Answers의 본인 **lb** answer만 Edit해서 **Eligible unchecked** 저장·재조회 후 수행한다.
```bash
dig @"$AUTH_NS" app.roadmap-lab.invalid A +norecurse +comments
```
A answer가 없음을 기록하되 exact DNS rcode는 실제값을 사용한다. **Eligible checked**, TTL **60**으로 원복·저장·재조회 후 같은질의로actualA/TTL을 확인한다. 최대2분 내 미반영이면실패로기록하고 삭제한다. HTTP backend자체를중단하거나두번째endpoint를만들지 않는다.

### authoritative 판정 host
공개 NS 질의는 승인된 DNS UDP/TCP53 출구가 있는 shell에서 수행한다. HEADER status=NOERROR 및 flags의 **aa**, ANSWER의 actual FQDN/A/LB_IP/TTL을 함께 읽는다. aa 없음·SERVFAIL·timeout을 성공으로 처리하지 않는다. HTTP --resolve는 VCN client에서 별도로 수행하며 public DNS resolution 성공을 뜻하지 않는다.


### 번호 순서로 완료하는 원복·정리

6. **Traffic Management 종료 및 실패 반환.** Eligible checked/TTL60 원복과authoritative aa/A응답을확인한뒤Attached domains의exactattachment Delete→policy Delete→publiczone Delete를수행하고각목록부재를읽는다.61 soleLB/nginx는삭제하지않고반환한다.79 NOT_SELECTED이면그실습때문에public으로바꾸지않는다.

실제 정리/원복 response, exact resource 또는rule ID,완료UTC와남은owner/expiry를비밀없는개인실행기록에남긴다.위의guarded복구가이미완료된경우중복mutation하지않는다.

## 결과검증

**기대 패턴(미실행):** authoritative A가 실제 LB IP, policy attachment에 정확한 FQDN, curl 61의 ok. public resolver `dig app.roadmap-lab.invalid`의 NXDOMAIN은 등록하지 않은 이 트랙의 정상 한계다.

실행 기록(미기입): 실행 UTC __ / 실제 resource OCID __ / 요청·SQL 실제 응답 __ / 공식 조건 차이 __ / 정리 완료 UTC __.

## 문제해결

SERVFAIL/빈 answer면 attachment FQDN, answer disabled, IPv4 type, NS hostname을 점검한다. CNAME과 steering A를 같은 owner에 두지 않는다. 실제 domain 위임을 실습에 추가하려면 소유권·과금 승인을 별도 받는다.

## 리소스정리

attachment 삭제 → steering policy 삭제 → 본인 public zone 삭제. 79 진행 시 LB/backend 유지. NS delegation을 하지 않았으므로 등록기관 변경은 없다.

### 소유자별 종료와 exact 대상 재조회
78 owner: Attached domains exact zone/domain/policy attachment Delete/confirm → Attached domains 부재 → Steering policies exact OCID Delete/confirm·목록 부재 → Public zones exact OCID Delete/confirm·목록 부재. 61 LB/app은 삭제하지 않고 반환 UTC만 인계한다.

실제 정리 UTC __ / exact resource 또는 rule ID __ / 삭제 후 조회 응답 __ / 잔존 resource·만료·owner __. 승인 없는 잔존 비용은 다음 실습을 이유로 연장하지 않는다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 개념은 해당 생성·변경·조회 단계에서 입력 선택과 실제 결과 해석을 위해 사용하며, 공식 예시는 실제 실습 실행 결과가 아닙니다.

- [38] [Overview of Traffic Management — Traffic Management](https://docs.oracle.com/en-us/iaas/Content/TrafficManagement/Concepts/overview.htm)
  - 확인 범위: Traffic Management helps.

### 실습 task 본문 근거

- [T1] [Tasks / create-tm-policy-lb.htm](https://docs.oracle.com/en-us/iaas/Content/TrafficManagement/Tasks/create-tm-policy-lb.htm) — 본문 절차·입력 필드·지원 조건을 읽어 실습에 적용.
- [T2] [Tasks / tm-attach-create.htm](https://docs.oracle.com/en-us/iaas/Content/TrafficManagement/Tasks/tm-attach-create.htm) — 본문 절차·입력 필드·지원 조건을 읽어 실습에 적용.

