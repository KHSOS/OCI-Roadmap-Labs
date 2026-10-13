# DNS

> 문서 작성 검토와 실행 검증은 별개입니다. OCI 실행은 **USER_EXECUTION_PENDING**이며 사용자 캡처는 별도 자료입니다. 미실행 자체는 문서 작성 승인 차단 사유가 아닙니다. 실제 결과 필드는 실행자가 채웁니다.

> 실행용 한국어 교육 문서. OCI 리소스 생성·명령 실행 결과는 아직 검증하지 않았으며 모든 기대 패턴은 미실행이다.

## 학습목표

- 승인 VCN default view에 private Primary zone lab.internal을 생성하고 Static mode·view/zone OCID를 재조회한다.
- A app.lab.internal에 실제 기존 backend IPv4, TTL60을 게시하고 vm-a의 metadata resolver dig/getent 및 :8080 /health 200/ok를 비교한다.
- missing 이름의 NXDOMAIN과 정상 A를 같은 client에서 비교하고 TTL60→120→60 게시·캐시 만료를 관찰한 뒤 77 반환 후 본인 A/zone만 삭제한다.

실제 OCI 요청·응답/SQL·정리 증거는 실행자가 기록한다. 아래 기대값은 실측값이 아니다.

## 대응 Roadmap 항목

- 항목명: **DNS**
- Roadmap 경로: **DNS**
- Node ID: `sequence-8-0`
- 상위 항목: 없음 — 최상위 학습 항목
- 문서 범위: 이 항목의 개념, 실제 설정 절차와 학습자 실행 검증.
- [전체 항목 목록](../README.md)

## 사전준비

**시작 조건:** VCN DNS/default resolver 준비 및61 app-only nginx8080 완료;LB_SELECTED=false

- [Virtual Cloud Network (VCN)](../03_Virtual_Cloud_Network_VCN/00_Virtual_Cloud_Network_VCN.md) — VCN 생성 및 DNS resolution/hostnames 활성화 완료. default private resolver 유지.
- [61 app-only backend 준비](../08_Load_Balancer/00_Load_Balancer.md#backend-app) — A 이전 app-only network/outbound/8080 인수 gate 및 A nginx 설치까지 완료하여 vm-a→actual backend:8080 /health200/ok와 CLIENT_CIDR/BACKEND_IP를 인수한다. LB_SELECTED=false, LB 생성 불필요. app/config/health와 client8080 rules를76/77 반환까지61 owner가 유지한다.

### 준비 완료 출력과 인수 확인

| 인수 항목 | 실제 출력·출처 | 다음 작업의 사용 위치 |
|---|---|---|
| 선행 완료 상태 | VCN DNS/default resolver 준비 및61 app-only nginx8080 완료;LB_SELECTED=false | 첫 변경 전 재조회 |
| 비밀 없는 실행 입력 | actual VCN/resolver/default view OCID, BACKEND_IP, CLIENT_CIDR, vm-a→backend8080/health200/ok, owner61 expiry | 아래 numbered Console/명령 단계 |
| 반환/후속 인계 | zone OCID/suffix/Static/A/TTL/HTTP 검증을77에 인계 | 종료 UTC 이전 exact owner 반환 |

실제 Compartment/Region은 승인 Console에서 읽고 동일 identity/Region을 고정한다. VCN client에서 `command -v dig curl`을 확인한다. 일반 Cloud Shell을 private DNS client로 사용하지 않는다. placeholder·명칭만 인수한 상태는 준비 완료가 아니다. 비밀번호·키·확인 token은 출력표에 적지 않는다.

## 권한

전용 Compartment의 관리자 승인 정책 예:
```text
Allow group <LabGroup> to manage dns in compartment <LabCompartment>
Allow group <LabGroup> to read virtual-network-family in compartment <LabCompartment>
```
VCN resolver/default private view가 다른 Compartment에 있으면 해당 DNS/VCN scope도 필요하다. DNS zone만 관리하는 권한으로 resolver view attachment가 항상 가능하다고 가정하지 않는다. Traffic Management/Health Checks는 각각 dns steering policy/attachment 및 health-check-family 생성·조회·삭제 권한을 별도 확인한다.

CreateZone/DeleteZone=DNS_ZONE_CREATE/DELETE; 게시 UpdateZoneRecords 또는 PatchZoneRecords=DNS_ZONE_UPDATE+DNS_RECORD_UPDATE; GetZoneRecords=DNS_ZONE_READ+DNS_RECORD_READ. GetView=DNS_VIEW_READ, GetResolver=DNS_RESOLVER_READ. 새 view를 연결하는 선택 분기는 UpdateResolver=DNS_RESOLVER_UPDATE가 필요하며 default view 재사용 기본에는 resolver update가 없다. 위 manage dns는 최소정책이 아니라 scoped 교육용 family grant다.

operation 근거: [공식 권한/작업 본문](https://docs.oracle.com/en-us/iaas/Content/Identity/policyreference/dnspolicyreference.htm). family grant는 minimum-permission 정책으로 주장하지 않는다. 실행 전에 관리자 승인된 identity/domain·실제 Compartment scope를 확인한다.

## 비용및주의사항

새 private zone1개/A RRset1개만 만든다. 현재 OCI DNS 가격·쿼리 집계 조건과 기존 app/client 유지비를 확인하되 private DNS를 무조건 유료 쿼리 또는 무료라고 단정하지 않는다. domain 등록/public NS delegation/new view/resolver endpoint/LB는 범위 밖이다. TTL60→120→60과 캐시 관찰을 승인 세션에 포함하고77 반환 후 owner76이 zone을 삭제한다.

## 실습구성

### 워크숍 순서와 입력값

**진행 순서:** VCN default view 확인 → private zone/A 생성·게시 → vm-a DNS/8080 → TTL변경/복구 →77 인계

**선택 이유:** DNS 데이터와 실제 app path를 분리 검증하며 새 LB/view/endpoint를 만들지 않는다.

| 입력 | 값 또는 정확한 출처 | 실행 위치 | 생성 전/후 | 이유 |
|---|---|---|---|---|
| zone/view | lab.internal / actualdefaultviewOCID | Console | 전 | defaultview 재사용 |
| BACKEND_IP | 61app 실제privateIPv4 | Console/vm-a | 전 | A가 실제8080app을 가리킴 |
| FQDN/TTL | app.lab.internal /60 | vm-a guest | 후 | DHCP DNS/HTTP분리 |


값이 다른 터미널에 자동 전파되지 않습니다. Console readback으로 비밀 없는 값만 인계하며, placeholder는 실제값으로 교체한 뒤 실행합니다. 새 리소스 생성 전 이름 충돌·소유자·최대 비용·종료 UTC를 확인하고 선택하지 않은 분기는 **NOT_SELECTED**로 기록합니다.


기존 VCN default resolver/default private view에 lab.internal private zone을 생성한다. 실제 backend private IP를 A record로 반환하고 VCN 안에서 dig한다.

<a id="private-dns-ready"></a>


### 이번 실습의 생성·재사용·앱 경계

| 구분 | 이번 주제의 실제 범위 |
|---|---|
| 새로 생성 | private Primary zone 및app A |
| 재사용 | VCN default view/resolver와61 app-only nginx8080/client |
| 앱/작업부하 및 변경 | LB/WAF 없이DNS→actual backend /health;TTL 변경/복구 |
| 실행/반환 순서 | VCN DNS/default resolver 준비 및61 app-only nginx8080 완료;LB_SELECTED=false → zone OCID/suffix/Static/A/TTL/HTTP 검증을77에 인계 |

새로 생성하지 않는 항목을선행완료로가장하지않는다.위표의실제OCID/주소와진행순서를첫mutation전에확인한다.

## 실습절차

### 생성부터 private DNS/앱 검증까지
`lab.internal`이 이미 존재하면 덮어쓰지 않고 본인 전용 zone인지 확인한다. 공동 zone이면 관리자와 다른 고유 zone suffix를 승인하고 이하 모든 owner/FQDN을 일관되게 변경한다. scope Private와 실제 VCN default view OCID를 생성 review에서 기록한다. default view 재사용에는 새 view/endpoint 생성이 없다.
존재하지 않는 이름 `dig missing.lab.internal A +comments`의 NXDOMAIN과 게시된 app A 결과를 같은 VCN client에서 비교한다. wrong IP를 의도적으로 게시하는 대신 TTL 60→120 변경·게시 후 Console 게시 TTL과 recursive 남은 캐시 TTL의 차이를 관찰하고 TTL60으로 복구한다. 앱 HTTP 시험은 client→backend :8080의 NSG 및 OS firewall source 허용이 선행이다. DNS 성공을 port 허용 성공으로 오인하지 않는다.

### 실행 단계

1. Networking → Virtual cloud networks → 실습 VCN → DNS resolver에서 default private view와 resolver OCID를 확인한다. 이 view와 VCN의 연결을 기록한다. 기존 default view를 detach하지 않는다.[T1]
2. Networking → DNS management → Private zones → Create zone. Name **lab.internal**, Type **Primary**, Compartment **실습 Compartment**, DNS scope **Private**, Private view **이 VCN resolver의 default private view**를 선택한다. 이 트랙은 기존 default view만 선택한다. Show Advanced Options → DNS resolution mode **Static**을 명시하고 Create한다. 생성 후 Zone information의 view·scope·resolution mode를 다시 읽는다. 별도 view/endpoint는 생성하지 않는다.[T2]
3. zone → Records → Manage records → Add record. Type **A**, Name **app**(UI가 FQDN을 요구하면 **app.lab.internal**), TTL **60**, Address **61 backend 실제 private IPv4**. RDATA/Answers **Basic**에서 IPv4를 입력하고 Add record → Publish changes → Confirm 페이지에서 diff 확인 → Confirm publish changes로 게시한다. 변경 preview와 게시 serial을 기록한다.[T3]
4. VCN 안의 **vm-a client guest** SSH 세션에서 확인한다. backend 자체의 loopback/self-request를 client 경로 검증으로 대체하지 않는다. OS에 dig가 없으면 Oracle Linux의 승인된 설치 절차로 `sudo dnf install -y bind-utils`를 실행한다. resolver가 VCN DNS DHCP 설정으로 전달되어 있는지 `/etc/resolv.conf`를 읽어 확인한다.
```bash
dig app.lab.internal A +noall +answer
dig @169.254.169.254 app.lab.internal A +noall +answer
getent ahostsv4 app.lab.internal
```
5. DNS 응답은 IP 확인이지 HTTP port 연결 확인이 아니다. 이 선택 app-only 시나리오에서는61 nginx App :8080을 반드시 유지하고 `curl --max-time 10 http://app.lab.internal:8080/health`로 기능까지 확인한다. private resolver 주소는 VCN 내부 전용이므로 laptop/일반 Cloud Shell에서 같은 결과를 기대하지 않는다.


### 진행·종료 판정
설정 readback·기능시험·원복·정리를 각각 실제 응답으로 기록한다. 기대값은 미실행이며 실패/만료 시 새 mutation을 중단하고 아래 topic별 종료 단계로 반환한다.

### Private DNS negative control TTL update and restoration
vm-a guest에서 정상A와 미존재 이름을 비교한다.
```bash
dig app.lab.internal A +comments
dig missing.lab.internal A +comments
curl --fail --max-time 10 http://app.lab.internal:8080/health
```
Console app A의 TTL을 **120**으로 Edit → Stage changes → Publish changes한다. 60초 이전캐시 만료 후 최대2분 내 같은A를 질의해 IP불변과 TTL을 기록한다. TTL을 **60**으로 되돌려 다시 Stage→Publish하고120초 캐시가 만료된 후 같은질의를 수행한다. 직접 metadata resolver 질의도 recursive이므로 authoritativeTTL이라고 부르지 않는다.


### 번호 순서로 완료하는 원복·정리

6. **DNS 종료 및 실패 반환.** app A TTL60 게시·캐시회복을확인하고77에zone/app lease를인계한다.77 본인RRset반환후app A Delete→Publish/Confirm→zone Delete/Confirm→목록부재를읽는다.동일DNS/HTTP검증결과를61app owner에게반환하고default view/resolver는보존한다.

실제 정리/원복 response, exact resource 또는rule ID,완료UTC와남은owner/expiry를비밀없는개인실행기록에남긴다.위의guarded복구가이미완료된경우중복mutation하지않는다.

## 결과검증

**기대 패턴(미실행):** app.lab.internal. <남은 TTL, 게시 TTL60 이하> IN A <실제 backend private IP>, getent도 같은 IPv4. DNS query time과 zone/view/resolver OCID를 기록한다. curl의 실제200/61의 ok까지 필수 완료 조건이다. DNS-only 성공은77-ready가 아니며 앱 미유지/HTTP 실패이면 BLOCKED_APP_NOT_READY로 남긴다.

실행 기록(미기입): 실행 UTC __ / 실제 resource OCID __ / 요청·SQL 실제 응답 __ / 공식 조건 차이 __ / 정리 완료 UTC __.

## 문제해결

NXDOMAIN이면 private scope, zone suffix, record 게시 여부, view attachment를 확인한다. 다른 VCN에서 질의하면 default view가 공유된 것으로 가정하지 않는다. timeout은 VCN DNS/DHCP·OS resolver 설정부터 확인한다.

## 리소스정리

77의 alias/proof/mail/mailtest/선택v6record 정리·반환을 인수할 때까지 zone/view/app을 유지한다. 종료 시 Records → Manage records에서 본인 app A만 Delete → Publish changes → Confirm publish changes하고 목록·dig의 캐시 만료 후 무응답을 확인한다. 이후 Private zones → exact zone OCID Actions → Delete → Confirm하고 같은 view/Compartment의 zone 목록에서 부재를 재조회한다. 앱 자체는61 owner에 반환한다. default private view/resolver 및 기존 VCN을 삭제하지 않는다.

### 소유자별 종료와 exact 대상 재조회
76 owner는77의 RRset 반환을 받고 위 record/zone 삭제 readback을 실행한다. default resolver/view는 보존한다. 61 app owner에게 DNS zone 종료와 HTTP/health 변경 없음 및 완료 UTC를 반환한다.

실제 정리 UTC __ / exact resource 또는 rule ID __ / 삭제 후 조회 응답 __ / 잔존 resource·만료·owner __. 승인 없는 잔존 비용은 다음 실습을 이유로 연장하지 않는다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 개념은 해당 생성·변경·조회 단계에서 입력 선택과 실제 결과 해석을 위해 사용하며, 공식 예시는 실제 실습 실행 결과가 아닙니다.

- [17] [Overview of DNS — DNS](https://docs.oracle.com/en-us/iaas/Content/DNS/Concepts/dnszonemanagement.htm)
  - 확인 범위: The DNS service helps.

### 실습 task 본문 근거

- [T1] [Tasks / privatedns.htm](https://docs.oracle.com/en-us/iaas/Content/DNS/Tasks/privatedns.htm) — 본문 절차·입력 필드·지원 조건을 읽어 실습에 적용.
- [T2] [Tasks / create-private-zone.htm](https://docs.oracle.com/en-us/iaas/Content/DNS/Tasks/create-private-zone.htm) — 본문 절차·입력 필드·지원 조건을 읽어 실습에 적용.
- [T3] [Tasks / record-add.htm](https://docs.oracle.com/en-us/iaas/Content/DNS/Tasks/record-add.htm) — 본문 절차·입력 필드·지원 조건을 읽어 실습에 적용.

