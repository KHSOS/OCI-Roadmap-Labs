# IP Address Preservation

IP Address Preservation은 NLB를 통과한 packet에서 원래 client 주소가 backend에 얼마나 유지되는지를 정하는 기능이다. 애플리케이션의 client 식별, source 기반 보안, network appliance의 packet 검사 요구에 영향을 준다. 단순히 public/private IP의 수명을 보존한다는 뜻은 아니다.

## 역할과 구성

Full NAT는 backend로 보낼 때 source와 destination IP를 변환한다. Backend Set의 Source Preservation은 destination을 NLB VIP에서 backend IP로 변환하되 원래 source IP와 port를 유지한다. NLB 수준 Transparent mode는 source/destination 정보를 바꾸지 않는 별도 방식으로, VCN routing이 packet을 NLB를 거치도록 해야 한다.

## 사용 예시

client 주소를 기준으로 접속을 허용하는 backend라면 Source Preservation을 사용했을 때 실제 client CIDR가 보안 규칙의 대상이 된다. 반대로 NAT된 source만 허용한 규칙은 같은 client 요청을 막을 수 있다. Transparent firewall 구성은 원래 목적지까지 유지해야 하므로 일반 VIP 접속형 애플리케이션과 다른 반환 경로 설계가 필요하다.

## 선택 기준과 제약

Source Preservation을 켠 Backend Set은 Compute instance 선택 조건을 따른다. IP 주소만으로 등록한 backend를 동일하게 취급하지 않는다. L3 IP listener는 source 보존이 자동 활성화되어 비활성화할 수 없다. Source/destination 보존이 켜진 NLB에서도 Backend Set source 보존을 끌 수 없다. 보존하지 않는 경우 공식 문서는 backend listener port당 AD별 약 21,500 active connection의 제약을 별도로 설명한다. 이 값은 NLB 전체 동시 연결 한도와 다른 범위다.

## 관련 개념

- [Network Load Balancer](00_Network_Load_Balancer.md)
- [Layer 3 / Layer 4](01_Layer_3_Layer_4.md)
- [Health Check Policies](03_Health_Check_Policies.md)

## 더 읽기

- [Oracle — Introduction to Network Load Balancer](https://docs.oracle.com/en-us/iaas/Content/NetworkLoadBalancer/introduction.htm)
- [Oracle — Enabling Network Load Balancer Backend Set Source Preservation](https://docs.oracle.com/en-us/iaas/Content/NetworkLoadBalancer/BackendSets/preserve-backend-set-source-id.htm)

설정과 실습 절차는 [보존된 IP Address Preservation Hands-on 문서](../../Hands-on/Networking/Original-Provisional/09_Network_Load_Balancer/02_IP_Address_Preservation.md)에서 읽을 수 있다.
