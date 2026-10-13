# DNS Zones / Records

> 문서 작성 검토와 실행 검증은 별개입니다. OCI 실행은 **USER_EXECUTION_PENDING**이며 사용자 캡처는 별도 자료입니다. 미실행 자체는 문서 작성 승인 차단 사유가 아닙니다. 실제 결과 필드는 실행자가 채웁니다.

> 실행용 한국어 교육 문서. OCI 리소스 생성·명령 실행 결과는 아직 검증하지 않았으며 모든 기대 패턴은 미실행이다.

## 학습목표

- 76의 승인 zone에서 app A를 실제 IP/TTL60으로 읽고 alias CNAME·proof TXT 및 DNS-only mail A/MX를 정확한 입력으로 게시한다. 선택 AAAA는 DNS-only 게시·질의만 수행한다.
- vm-a에서 A/CNAME/TXT/MX와 선택 AAAA의 owner/type/TTL/rdata를 게시값과 비교하고 SOA serial을 기록한다. IPv6 접속·메일 배달은 별도 승인 없으면 미실행이다.
- 존재하지 않는 이름 및 없는 type의 결과를 positive A와 구별하고 TXT 변경/원복 후 본인이 추가한 RRset만 삭제·게시·재조회하여 zone을76에 반환한다.

실제 OCI 요청·응답/SQL·정리 증거는 실행자가 기록한다. 아래 기대값은 실측값이 아니다.

## 대응 Roadmap 항목

- 항목명: **DNS Zones / Records**
- Roadmap 경로: **DNS → DNS Zones / Records**
- Node ID: `sequence-8-1`
- 상위 항목: **DNS** (`sequence-8-0`)
- 문서 범위: 이 항목의 개념, 실제 설정 절차와 학습자 실행 검증.
- [전체 항목 목록](../README.md)

## 사전준비

**시작 조건:** 76 Static private zone/A 게시와vm-a DNS-name HTTP8080/health200 완료

- [DNS](00_DNS.md#private-dns-ready) — 1~5단계 완료: Static lab.internal private zone/A 게시, vm-a dig와 app-only :8080 /health200/ok 성공. zone/app 유지.

### 준비 완료 출력과 인수 확인

| 인수 항목 | 실제 출력·출처 | 다음 작업의 사용 위치 |
|---|---|---|
| 선행 완료 상태 | 76 Static private zone/A 게시와vm-a DNS-name HTTP8080/health200 완료 | 첫 변경 전 재조회 |
| 비밀 없는 실행 입력 | actual zone/view OCID·suffix, app A/BACKEND_IP/TTL, SOA serial, app owner/expiry | 아래 numbered Console/명령 단계 |
| 반환/후속 인계 | 본인RRset 삭제·게시·캐시 readback 후76 zone 반환 | 종료 UTC 이전 exact owner 반환 |

실제 Compartment/Region은 승인 Console에서 읽고 동일 identity/Region을 고정한다. VCN client에서 `command -v dig curl`을 확인한다. 일반 Cloud Shell을 private DNS client로 사용하지 않는다. placeholder·명칭만 인수한 상태는 준비 완료가 아니다. 비밀번호·키·확인 token은 출력표에 적지 않는다.

## 권한

전용 Compartment의 관리자 승인 정책 예:
```text
Allow group <LabGroup> to manage dns in compartment <LabCompartment>
```
UpdateRRSet/PatchRRSet/DeleteRRSet=DNS_RECORD_UPDATE; GetRRSet=DNS_RECORD_READ; zone-level 게시=DNS_ZONE_UPDATE+DNS_RECORD_UPDATE. SOA 확인 GetZoneRecords=DNS_ZONE_READ+DNS_RECORD_READ. 기존 app A·SOA/NS를 삭제할 권한이 있다고 삭제해도 되는 것은 아니다.

operation 근거: [공식 권한/작업 본문](https://docs.oracle.com/en-us/iaas/Content/Identity/policyreference/dnspolicyreference.htm). family grant는 minimum-permission 정책으로 주장하지 않는다. 실행 전에 관리자 승인된 identity/domain·실제 Compartment scope를 확인한다.

## 비용및주의사항

76의 private zone/A/app을 유지하고 CNAME/TXT/mail A/mailtest MX 네 RRset 및 승인된 경우 DNS-only AAAA 하나만 추가한다. DNS 쿼리·기존 zone 보존의 현재 계약 조건과 관찰 종료 UTC를76 owner에게 확인한다. mail A/MX는 SMTP 서비스가 아니므로 메일서버/port25/IPv6 VNIC/LB를 만들지 않는다. TXT 복구와 RRset 삭제 후 캐시 만료 관찰을 예산에 넣고 도메인 등록을 추가하지 않는다.

## 실습구성

### 워크숍 순서와 입력값

**진행 순서:** 76 zone 인수 → CNAME/TXT/mail A/MX/선택AAAA publish → SOA/TTL/rdata질의 → TXT변경/복구 →본인RRset삭제 →76 반환

**선택 이유:** 게시 전후와 recursive cache를 구분하고 시스템 SOA/NS는 건드리지 않는다.

| 입력 | 값 또는 정확한 출처 | 실행 위치 | 생성 전/후 | 이유 |
|---|---|---|---|---|
| zone/FQDN | 76실제suffix /alias·proof | Console/vm-a | 전 | 충돌없는RRset |
| CNAME/TXT/TTL | app.lab.internal. / roadmap-dns /60 | Console | 전 | absoluteFQDN/text 확인 |


값이 다른 터미널에 자동 전파되지 않습니다. Console readback으로 비밀 없는 값만 인계하며, placeholder는 실제값으로 교체한 뒤 실행합니다. 새 리소스 생성 전 이름 충돌·소유자·최대 비용·종료 UTC를 확인하고 선택하지 않은 분기는 **NOT_SELECTED**로 기록합니다.


게시 전 staging과 게시 후 authoritative 데이터·recursive TTL 차이를 실제 RRset으로 확인한다.


### 이번 실습의 생성·재사용·앱 경계

| 구분 | 이번 주제의 실제 범위 |
|---|---|
| 새로 생성 | alias CNAME/proof TXT/mail A/mailtest MX·선택v6record AAAA |
| 재사용 | 76 zone/app A와동일app-only nginx/client |
| 앱/작업부하 및 변경 | TXT 변경/복구;MX/AAAA는DNS-only이며SMTP/IPv6앱 없음 |
| 실행/반환 순서 | 76 Static private zone/A 게시와vm-a DNS-name HTTP8080/health200 완료 → 본인RRset 삭제·게시·캐시 readback 후76 zone 반환 |

새로 생성하지 않는 항목을선행완료로가장하지않는다.위표의실제OCID/주소와진행순서를첫mutation전에확인한다.

## 실습절차

### RRset 게시·복구 checkpoint
기존 app A owner/IP와 SOA serial을 먼저 읽고 alias/proof의 충돌이 없는지 확인한다. TXT RDATA는 Console이 요구하는 quoting으로 `"roadmap-dns"`를 저장하되 반환된 실제 rdata를 읽어 중복 quoting 여부를 검사한다. alias CNAME target의 trailing dot은 absolute FQDN을 표현한다.
변경 시험 후 proof TXT를 roadmap-dns로 되돌려 Stage→Publish→dig 복구를 검증한 뒤 cleanup에서 본인 alias/proof/mail/mailtest 및 선택v6record RRset을 삭제·게시한다. missing 이름 NXDOMAIN은 잘못된 view/suffix가 아닌 동일 zone의 positive A 응답과 함께 판정한다. 76 owner가 zone 삭제를 수행하므로 본 문서가 shared default view를 삭제하지 않는다.

### A·AAAA·MX 입력을 실제 RRset 작업으로 연결하기
이 단계는 76의 **승인된 동일 private zone**에서만 진행한다. vm-a가 같은 VCN resolver를 쓰고 Static mode인지를 다시 읽는다. 운영 RRset 또는 기존 동일 이름/type이 있으면 덮어쓰지 않는다. [Private DNS supported records](https://docs.oracle.com/en-us/iaas/Content/DNS/Tasks/privatedns.htm#privatedns_topic_supported_resource_records)에서 A/AAAA/MX 지원과 MX target 제약을 확인한다.

| 작업 | Name / Type / TTL | RDATA/Answers Basic 입력 | 선택 이유와 검증 범위 |
|---|---|---|---|
| 기존 A 재조회(새로 만들지 않음) | app / A / 60 | 76의 actual BACKEND_IP | IPv4 실제 앱 :8080의 주소; 변경 금지 |
| DNS-only mail target A 생성 | mail / A / 60 | 동일 actual BACKEND_IP | MX가 CNAME/IP가 아닌 주소 record를 가진 hostname을 가리키게 한다. nginx VM을 SMTP 서버라고 주장하지 않는다 |
| DNS-only MX 생성 | mailtest / MX / 60 | Preference **10**, Mail server **mail.lab.internal.**; advanced 단일 RDATA는 **10 mail.lab.internal.** | 정확한 preference·absolute name을 응답과 비교하며 SMTP25 개방·메일 발송·메일서버 생성은 하지 않는다 |
| 선택 DNS-only AAAA 생성 | v6record / AAAA / 60 | **2001:db8::61** | IPv6 documentation 주소를 DNS 데이터 입력으로만 사용한다. 실제 VNIC 주소/응답이 아니다. DNS-only 데이터 승인 없으면 NOT_SELECTED |

각 RRset은 Records → Manage records → Add record에서 Name/Type/TTL/RDATA를 입력하여 Add record → Publish changes → Confirm diff → Confirm publish changes한다. AAAA의 실제 IPv6 앱 확장은 승인된 기존 VNIC의 actual IPv6, route/NSG/OS/앱 bind/외부 path까지 별도 준비됐을 때만 다른 범위로 수행한다. 이 문서는 `curl -6` 성공 또는 MX mail delivery를 주장하지 않는다. MX priority 숫자는 낮을수록 우선순위가 높지만 단일 MX에서는 failover를 검증하지 않는다. [record task](https://docs.oracle.com/en-us/iaas/Content/DNS/Tasks/record-add.htm)의 RDATA normalization 때문에 hostname 끝의 dot이 붙을 수 있다.

vm-a guest에서 같은 resolver로 실제 응답을 읽는다. HEADER status와 ANSWER owner/type/rdata를 Console 게시값과 대조한다. resolver TTL은 남은 캐시시간이므로 항상60은 아니다.
```bash
dig @169.254.169.254 app.lab.internal A +comments +answer
dig @169.254.169.254 mail.lab.internal A +comments +answer
dig @169.254.169.254 mailtest.lab.internal MX +comments +answer
# AAAA DNS-only branch를 실제 게시한 경우에만:
dig @169.254.169.254 v6record.lab.internal AAAA +comments +answer
# positive A와 같은 이름의 미게시 type: Static zone에서는 NOERROR/no AAAA (NODATA)
dig @169.254.169.254 app.lab.internal AAAA +comments +answer
dig @169.254.169.254 missing.lab.internal A +comments +answer
curl --fail --max-time 10 http://app.lab.internal:8080/health
```
**판정:** app/mail A=actualBACKEND_IP, MX=`10 mail.lab.internal.`, 선택 AAAA=`2001:db8::61`가 실제 query에서 반환돼야 DNS 데이터 작업 완료다. missing=NXDOMAIN과 app/AAAA=NODATA를 timeout/SERVFAIL과 구별한다. TXT `roadmap-dns`→`roadmap-dns-v2`→`roadmap-dns` 복구를 아래 절차로 수행한다. MX/AAAA 삭제 시험은 이 단계 cleanup에서 본인 RRset만 Delete → Publish/Confirm 후 TTL/negative cache 만료를 고려한 동일 query 무응답을 읽고, 유지되는 app A/HTTP200을 positive control로 확인한다. 새 VM/LB/mailserver/IPv6 VNIC를 만들지 않는다.

### 실행 단계

1. lab.internal → Records → Manage records → Add record에서 기존 **app A**를 유지하고 **alias CNAME app.lab.internal.**, TTL **60**, **proof TXT roadmap-dns**, TTL **60**을 Add한다. CNAME 이름 alias에는 A/TXT 등 다른 record type을 함께 두지 않는다.[T1]
2. 변경 preview를 검토한다. 아직 Publish 전에는 recursive resolver가 새 record를 반환하지 않을 수 있다. Publish changes 후 SOA serial이 바뀌는지 기록한다. SOA/NS 시스템 record를 삭제하지 않는다.
3. VCN 내부 Compute에서 3종 record와 존재하지 않는 이름을 질의한다.
```bash
dig alias.lab.internal CNAME +noall +answer
dig alias.lab.internal A +noall +answer
dig proof.lab.internal TXT +noall +answer
dig missing.lab.internal A +comments
```
4. **proof TXT**를 **roadmap-dns-v2**로 Edit → Stage → Publish한다. 이전 캐시 TTL/negative cache를 고려하여 최대 2분 후 다시 질의한다. 운영 RRset에 섞지 않는다.
5. 반환 record owner/type/TTL/rdata를 Console의 게시 값과 비교한다. CNAME A query는 alias와 target A가 같이 나올 수 있지만 +short를 쓰면 owner/type 정보가 사라지므로 이 실습은 +answer를 사용한다.


### 진행·종료 판정
설정 readback·기능시험·원복·정리를 각각 실제 응답으로 기록한다. 기대값은 미실행이며 실패/만료 시 새 mutation을 중단하고 아래 topic별 종료 단계로 반환한다.

### Actual SOA query and TXT restoration
vm-a에서 게시 전후 및 복구 후 SOA를 실제 조회한다.
```bash
dig lab.internal SOA +noall +answer
```
proof TXT를 **roadmap-dns**로 Edit→Stage→Publish한 뒤 최대2분/캐시TTL 후 다음으로 복구를 확인한다.
```bash
dig proof.lab.internal TXT +noall +answer
dig alias.lab.internal A +noall +answer
dig lab.internal SOA +noall +answer
```
serial은 Console 게시값과 비교하고 캐시응답 serial이 즉시 바뀐다고 보장하지 않는다. 이 RRset시험후76 owner에게 zone반환을 알린다.

### 76-ready 인수 기준
76의 actual app A, client DNS resolution, DNS-name HTTP :8080 `/health`=200/ok를 모두 인수해야 한다. DNS-only 대안은 별도 DNS_ONLY 결과이며 이 문서의 app-ready prerequisite를 충족하지 않는다.


### 번호 순서로 완료하는 원복·정리

6. **DNS Zones / Records 종료 및 실패 반환.** proof TXT를roadmap-dns로원복/게시/질의한뒤본인alias/proof/mailtest/선택v6record RRset삭제·게시→mail A삭제·게시→캐시만료동일질의와app A/HTTPpositive를읽는다.76 zone/app A/SOA/NS는남겨owner76에반환한다.

실제 정리/원복 response, exact resource 또는rule ID,완료UTC와남은owner/expiry를비밀없는개인실행기록에남긴다.위의guarded복구가이미완료된경우중복mutation하지않는다.

## 결과검증

**기대 패턴(미실행):** alias CNAME app.lab.internal., proof TXT "roadmap-dns"→"roadmap-dns-v2", missing은 NXDOMAIN. 응답에 SOA와 negative cache가 있을 수 있다. 실제 serial·TTL·query 결과를 기록한다.

실행 기록(미기입): 실행 UTC __ / 실제 resource OCID __ / 요청·SQL 실제 응답 __ / 공식 조건 차이 __ / 정리 완료 UTC __.

## 문제해결

CNAME conflict면 같은 owner 이름의 A/TXT 존재를 확인한다. Publish가 실패하면 etag/concurrent change를 검토하고 refresh 후 재시도한다. cache가 오래 남으면 앱 오류라고 해석하지 않는다.

## 리소스정리

alias/proof/mailtest MX/선택 v6record AAAA 삭제 → Publish changes → Confirm publish changes → mail A 삭제·게시. 각 exact name/type을 Records에서 재조회하고 vm-a dig에서 캐시 만료 뒤 무응답을 확인한다. app A와 /health200은 유지한다. 본 문서는 자신이 추가한 RRset만 정리하고76 owner에 반환한다. **zone/app A삭제는76 owner만**, 모든 selected consumer 반환 확인 후 수행한다. shared default private view는 유지한다.

### 소유자별 종료와 exact 대상 재조회
77 owner는 생성한 네 RRset 및 선택AAAA만 삭제·게시·cache 만료 query를 확인한다. 76 zone/app A/SOA/NS는 보존하고 actual zone OCID/본인 삭제 RRset 목록/반환 UTC를76에 전달한다.

실제 정리 UTC __ / exact resource 또는 rule ID __ / 삭제 후 조회 응답 __ / 잔존 resource·만료·owner __. 승인 없는 잔존 비용은 다음 실습을 이유로 연장하지 않는다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 개념은 해당 생성·변경·조회 단계에서 입력 선택과 실제 결과 해석을 위해 사용하며, 공식 예시는 실제 실습 실행 결과가 아닙니다.

- [17] [Overview of DNS — DNS Zones / Records](https://docs.oracle.com/en-us/iaas/Content/DNS/Concepts/dnszonemanagement.htm)
  - 확인 범위: zone A zone is / resource records.

### 실습 task 본문 근거

- [T1] [Tasks / record-add.htm](https://docs.oracle.com/en-us/iaas/Content/DNS/Tasks/record-add.htm) — 본문 절차·입력 필드·지원 조건을 읽어 실습에 적용.
- [T2] [Reference / supporteddnsresource.htm](https://docs.oracle.com/en-us/iaas/Content/DNS/Reference/supporteddnsresource.htm) — 본문 절차·입력 필드·지원 조건을 읽어 실습에 적용.

