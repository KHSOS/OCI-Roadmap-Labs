# Well-Architected Framework

OCI Well-Architected Framework는 클라우드 topology를 설계·운영하면서 업무 가치를 높이기 위한 권장사항을 다섯 business goal, 즉 pillar로 정리한 체계다. 특정 서비스를 생성하는 기능이나 모든 workload에 동일하게 적용하는 정답 architecture가 아니다. [Oracle: Well-Architected Framework](https://docs.oracle.com/en/solutions/oci-best-practices/index.html)

## 다섯 pillar와 설계 질문

| Pillar | 목적 | 생각해 볼 질문 |
|---|---|---|
| Security and compliance | 시스템과 정보 자산 보호 | 누가 접근하며 데이터 보호·감사·격리는 어떻게 이루어지는가? |
| Reliability and resilience | 장애에 견디는 infrastructure로 신뢰성 확보 | health check·backup·복제·재해 복구는 어떤 장애 범위를 다루는가? |
| Performance and cost optimization | 효율적인 자원 사용과 성능·비용 균형 | workload에 맞는 Compute 크기와 storage 전략인가? |
| Operational efficiency | application과 infrastructure의 운영·모니터링 | 배포·OS 관리·장애 대응을 지속할 수 있는가? |
| Distributed cloud | 분산 배치의 설계·통합·최적화 | public·dedicated·hybrid·edge·multicloud 중 어디에 workload를 두는가? |

이 분류는 Oracle 문서의 현재 다섯 pillar다. 다른 cloud의 유사 framework가 같은 이름과 구성을 사용한다고 가정하지 않는다.

## 하나의 선택을 여러 관점으로 보기

애플리케이션 서버를 두 FD에 분산하는 예를 생각해 보자. [Fault Domains](03_Fault_Domains.md)는 hardware·maintenance 격리를 제공하지만, application failover에는 트래픽 분산과 상태 데이터 처리가 더 필요하다. 두 번째 instance는 비용을 추가하며 운영해야 할 OS도 늘어난다. 보안은 [Shared Security Model](04_Shared_Security_Model.md)에 따라 계정·규칙·patching 책임까지 포함한다. 따라서 FD만 분리했다고 다섯 pillar를 모두 만족한 것은 아니다.

## 요구에 따른 선택과 trade-off

CPU가 필요한 작업과 메모리가 필요한 작업은 [Shapes](../02_Compute/01_Shapes.md)의 다른 구성을 요구한다. 여유 자원을 크게 잡는 것은 capacity를 늘리지만 비용 효율을 떨어뜨릴 수 있다. 반대로 비용만 줄이면서 복구·관측 기능을 없애면 업무의 가용성 목표를 충족하지 못할 수 있다. Framework는 이런 선택을 업무 요구와 연결해 토론하는 기준이다.

Distributed cloud 역시 서버 수를 늘렸다는 뜻이 아니다. 서로 다른 배포 환경의 통합·데이터 주권·운영·복구를 다루며, 같은 Region 안의 두 VM만으로 그 범위를 설명할 수 없다.

## 적용의 경계

권장사항 목록을 읽는 것과 실제 시스템의 성능·보안·복구 능력을 확인하는 것은 다르다. workload별 목표와 증거가 있어야 개선의 효과를 판단할 수 있다. 모든 pillar를 한 번의 작은 lab으로 입증하려 하기보다 현재 요구에 가장 중요한 위험과 선택을 구분하는 것이 유용하다. 기존 배치·보안·운영 사례는 [보존된 Hands-on](../../Hands-on/Original-Provisional/01_Introduction_to_OCI/05_Well_Architected_Framework.md)에 있다.

## 공식문서

- [Learn About the Well-Architected Framework for Oracle Cloud Infrastructure](https://docs.oracle.com/en/solutions/oci-best-practices/index.html): 다섯 pillar, focus areas, 관련 역할.
