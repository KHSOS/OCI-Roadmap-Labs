# Traffic Management

Traffic Management는 DNS 응답에 정책을 적용하여 어느 endpoint 주소를 돌려줄지 결정하는 기능이다. Client의 실제 HTTP/TCP 연결을 proxy하는 Load Balancer와 달리 이름 해석 단계에서 traffic 방향을 유도한다.

## 역할과 구성

Steering Policy는 answer와 rule을 담고 Attachment는 policy를 zone의 domain에 연결한다. Attachment가 다루는 record type에서는 해당 domain의 일반 record 대신 policy가 응답을 구성한다. Failover, Load Balancer, Geolocation, ASN, IP Prefix가 서로 다른 선택 근거를 제공하며 Health Checks를 이용해 endpoint health를 판단할 수 있다.

## 사용 예시

두 지역 endpoint에 가중치를 달리 주어 신규 지역으로 일부 DNS 응답을 보내는 migration이나 primary 장애 시 secondary 주소를 제공하는 failover를 생각할 수 있다. Endpoint가 하나뿐이면 policy 연결과 응답 선택은 설명할 수 있지만 weighted 분산이나 대체 endpoint failover는 성립하지 않는다.

## 선택 기준과 제약

Public DNS에서만 지원하며 private zone에는 적용할 수 없다. DNS cache와 TTL 때문에 policy 변화가 모든 client 연결에 즉시 반영되지는 않고 이미 열린 연결도 옮기지 않는다. Authoritative 응답을 직접 읽는 것과 registrar delegation을 통한 일반 recursive resolution은 구분한다. 한 domain의 특정 record type을 덮는 attachment는 최대 하나다. 공식 기본 한도는 tenancy당 policy 100개, attachment 1,000개다.

## 관련 개념

- [DNS](00_DNS.md)
- [DNS Zones / Records](01_DNS_Zones_Records.md)
- [DNS Health Checks](03_DNS_Health_Checks.md)

## 더 읽기

- [Oracle — Overview of Traffic Management](https://docs.oracle.com/en-us/iaas/Content/TrafficManagement/Concepts/overview.htm)
- [Oracle — Private DNS](https://docs.oracle.com/en-us/iaas/Content/DNS/Tasks/privatedns.htm)

설정과 실습 절차는 [보존된 Traffic Management Hands-on 문서](../../Hands-on/Networking/Original-Provisional/11_DNS/02_Traffic_Management.md)에서 읽을 수 있다.
