# Fault Domains

Fault Domain (FD)은 하나의 Availability Domain 안에 있는 hardware와 infrastructure의 그룹이다. 각 AD에는 세 FD가 있으며, instance를 서로 다른 FD에 분산하면 동일한 물리 hardware에 함께 배치되지 않도록 하는 anti-affinity를 활용할 수 있다. 한 FD에 영향을 주는 hardware 장애나 Compute hardware maintenance는 다른 FD의 instance에 영향을 주지 않도록 분리된다. [Oracle: Fault Domains](https://docs.oracle.com/en-us/iaas/Content/General/Concepts/regions.htm)

## 같은 AD 안에서 분산하는 이유

예를 들어 같은 AD에 두 애플리케이션 서버를 두되 서로 다른 FD를 선택하면 한 hardware 그룹에 두 서버가 함께 의존하는 위험을 줄일 수 있다. 단일 AD Region에서도 가능한 배치 전략이다. FD는 사용자가 생성하는 서버나 네트워크가 아니라 provider의 배치 경계다.

## 배치 선택과 자동 선택

instance나 instance pool을 launch할 때 FD를 명시할 수 있다. 명시하지 않으면 OCI가 AD의 가용 capacity를 고려하면서 서로 다른 FD로의 배치를 best effort로 선택한다. 자동 선택이 모든 instance의 FD 분산을 확정하는 것은 아니므로 분산이 설계 요건이라면 실제 placement가 중요하다. [Compute placement](https://docs.oracle.com/en-us/iaas/Content/General/Concepts/regions.htm)

같은 AD의 FD 분산은 [Availability Domains](02_Availability_Domains.md) 사이의 기반 시설 분리나 [Regions](01_Regions.md) 사이의 지리적 독립성과 다르다. FD가 다르더라도 두 instance는 같은 AD에 남는다.

## 가용성과 비용의 균형

두 instance의 FD가 다르다는 것은 배치 분리를 뜻할 뿐 애플리케이션 데이터 복제나 자동 failover를 뜻하지 않는다. 트래픽을 살아 있는 서버로 보내는 경로와 상태 데이터의 처리 방식이 따로 필요하다. 분산을 위해 instance 수를 늘리면 Compute와 연결 storage 비용도 증가한다. 따라서 필요한 장애 범위와 실제 workload의 복구 방식을 먼저 정하는 것이 [Well-Architected Framework](05_Well_Architected_Framework.md)의 신뢰성·비용 관점에 맞는다.

## Hands-on 연결

[보존된 FD 배치 Hands-on](../../Hands-on/Original-Provisional/01_Introduction_to_OCI/03_Fault_Domains.md)은 같은 AD의 다른 FD에 VM을 배치하는 자료다. 그 자료의 배치 비교는 애플리케이션 장애 전환 시험과 구분된다.


## 공식문서

- [Regions and Availability Domains — Fault Domains](https://docs.oracle.com/en-us/iaas/Content/General/Concepts/regions.htm): 세 FD, anti-affinity, 선택과 best-effort 배치.
