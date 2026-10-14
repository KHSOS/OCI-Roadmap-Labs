# DNS Zones / Records

DNS Zone은 DNS namespace의 관리 영역이고 Resource Record는 그 영역 안의 이름과 type별 데이터를 표현한다. Zone과 domain은 관련되지만 같은 개념은 아니며 child zone은 별도의 SOA/NS를 가진 관리 영역으로 delegation될 수 있다.

## 역할과 구성

Record는 owner name, type, TTL, RDATA로 해석한다. A는 IPv4, AAAA는 IPv6, CNAME은 canonical name, MX는 mail exchanger, TXT는 text 정보를 담는다. SOA는 zone의 권한·serial·timer 정보를, NS는 authoritative nameserver를 표현한다. 같은 이름/type의 관련 record 묶음을 RRset이라고 한다.

## 사용 예시

app의 A와 alias의 CNAME은 주소 자체와 별칭의 관계를 보여준다. MX의 숫자 preference는 낮을수록 우선이며 target은 hostname이어야 하고 IP나 CNAME을 직접 가리키지 않는다. AAAA 또는 MX record가 존재하는 것만으로 IPv6 network 접속이나 메일 배달이 가능해지는 것은 아니다.

## 선택 기준과 제약

TTL은 cache를 고려한 변경 속도와 질의량에 영향을 주며 recursive 응답 TTL은 게시한 원래 값보다 작을 수 있다. 변경 직후 모든 cache가 즉시 바뀌지는 않는다. OCI는 RDATA를 정규화하여 hostname 끝의 dot 등을 보완할 수 있다. Public/private zone의 지원 record type을 따로 확인하고 record에 비밀을 넣지 않는다. 공식 DNS overview는 계정당 zone 1,000개, zone당 record 25,000개와 zone file upload 1MB 한도를 설명한다.

## 관련 개념

- [DNS](00_DNS.md)
- [Traffic Management](02_Traffic_Management.md)

## 더 읽기

- [Oracle — Overview of DNS](https://docs.oracle.com/en-us/iaas/Content/DNS/Concepts/dnszonemanagement.htm)
- [Oracle — Managing Resource Records](https://docs.oracle.com/en-us/iaas/Content/DNS/Reference/supporteddnsresource.htm)
- [Oracle — Private DNS](https://docs.oracle.com/en-us/iaas/Content/DNS/Tasks/privatedns.htm)

설정과 실습 절차는 [보존된 DNS Zones / Records Hands-on 문서](../../Hands-on/Networking/Original-Provisional/11_DNS/01_DNS_Zones_Records.md)에서 읽을 수 있다.
