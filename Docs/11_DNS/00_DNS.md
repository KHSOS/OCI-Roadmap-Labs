# DNS

DNS는 사람이 읽는 이름을 IP 주소 또는 필요한 resource record 정보로 해석하는 체계다. OCI DNS는 zone과 record를 관리하고 이름에 대한 응답을 제공한다. DNS가 주소를 반환하는 것과 그 주소의 애플리케이션에 network 연결이 가능한 것은 서로 다른 문제다.

## 역할과 구성

Authoritative nameserver는 zone의 데이터를 제공하고 recursive resolver는 client 대신 질의를 해결하며 응답을 cache한다. Public DNS는 인터넷에서 이름을 찾도록 registrar를 통한 delegation이 필요하다. Private DNS는 VCN resolver와 view를 통해 private zone 데이터를 제공한다. View는 zone의 묶음이고 resolver가 어떤 view를 평가하는지에 따라 같은 이름의 응답도 달라질 수 있다.

## 사용 예시

내부 앱 이름 app.lab.internal을 backend private IPv4에 연결하면 VCN client는 주소를 몰라도 이름으로 대상을 찾는다. 인터넷에서 같은 이름을 질의한다고 그 private view의 응답을 얻지는 않는다. 이름이 올바르게 해석돼도 route·NSG·guest firewall·앱 port가 맞지 않으면 접속은 실패할 수 있다.

## 선택 기준과 제약

VCN resolver의 기본 진입 주소는 169.254.169.254이며 외부 network 연계에는 별도의 connectivity와 resolver endpoint/forwarding 설계가 필요할 수 있다. Split-horizon은 같은 zone 이름에 public/private 응답을 다르게 제공하는 방식이다. Static private zone은 포함된 이름이 없으면 NXDOMAIN, 이름은 있으나 type 데이터가 없으면 NODATA를 반환한다. Traffic Management는 private DNS에 지원되지 않는다. Public zone은 공식 문서상 OC1 commercial realm에서만 지원된다.

## 관련 개념

- [DNS Zones / Records](01_DNS_Zones_Records.md)
- [Traffic Management](02_Traffic_Management.md)
- [DNS Health Checks](03_DNS_Health_Checks.md)

## 더 읽기

- [Oracle — Overview of DNS](https://docs.oracle.com/en-us/iaas/Content/DNS/Concepts/dnszonemanagement.htm)
- [Oracle — Private DNS](https://docs.oracle.com/en-us/iaas/Content/DNS/Tasks/privatedns.htm)

설정과 실습 절차는 [보존된 DNS Hands-on 문서](../../Hands-on/Networking/Original-Provisional/11_DNS/00_DNS.md)에서 읽을 수 있다.
