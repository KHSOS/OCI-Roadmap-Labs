# Availability Domains

Availability Domain (AD)은 Region 안의 하나 이상의 데이터센터로 구성되는 격리 범위다. 같은 Region의 AD는 전력·냉각·내부 AD network 같은 기반 시설을 공유하지 않으므로 한 AD의 장애가 다른 AD에 동시에 영향을 줄 가능성을 낮춘다. AD 사이에는 낮은 지연시간과 높은 대역폭의 네트워크가 연결된다. [Oracle: Regions and Availability Domains](https://docs.oracle.com/en-us/iaas/Content/General/Concepts/regions.htm)

## AD가 필요한 이유

여러 AD에 애플리케이션 구성 요소를 분산하면 데이터센터 수준의 장애를 고려하는 설계를 할 수 있다. 예를 들어 복제된 애플리케이션 서버를 여러 AD에 배치하는 것은 장애 격리를 활용하는 방식이다. 다만 상태 데이터 복제, 트래픽 분산, health check와 장애 전환이 함께 설계되어야 실제 서비스 연속성을 얻을 수 있다. AD가 다르다는 사실만으로 복구가 완성되지는 않는다.

## ad-selection

Compute instance는 특정 AD에 배치되고 연결할 Block Volume과 Boot Volume도 그 instance와 같은 AD에 있어야 한다. 반면 Regional subnet은 특정 AD에 한정되지 않아 같은 Region의 여러 AD에 있는 자원을 수용할 수 있다. 따라서 subnet이 Regional이라는 이유로 instance의 AD 선택이 사라지는 것은 아니다. [Region별 자원 범위](https://docs.oracle.com/en-us/iaas/Content/General/Concepts/regions.htm)

AD 이름은 tenancy별 prefix를 포함한다. 같은 `AD-1`처럼 보이는 이름도 서로 다른 tenancy에서 같은 물리 데이터센터를 뜻한다고 가정하면 안 된다. 배치 후보는 현재 tenancy와 Region에 제공되는 실제 AD이며, 다른 계정의 예제 문자열은 배치 기준이 아니다. 기존 공동 launch 자료는 [보존된 AD 선택 Hands-on](../../Hands-on/Original-Provisional/01_Introduction_to_OCI/02_Availability_Domains.md#실습절차)에 있다.

## Region과 Fault Domain의 차이

- [Region](01_Regions.md)은 지리적 위치와 regional 자원의 범위다.
- AD는 Region 내부의 데이터센터 격리 범위다.
- [Fault Domain](03_Fault_Domains.md)은 AD 내부의 hardware·maintenance 격리 그룹이다.

AD를 여러 개 사용하는 설계와 여러 Region을 사용하는 설계는 장애 범위와 데이터 이동 조건이 다르다. 한 Region의 AD 분산을 다른 Region의 재해 복구와 동일하게 취급하면 안 된다.

## 배치 선택의 한계

Region에 따라 AD가 하나만 있을 수 있다. 고객이 AD 자체를 새로 생성하거나 개수를 늘리는 것은 아니다. 단일 AD에서는 FD 분산을 활용할 수 있지만 AD 전체 장애에 대한 분리와 같지는 않다. 또한 특정 AD에 원하는 shape의 실제 capacity가 있는지는 별개의 문제다. [Compute](../02_Compute/00_Compute.md)와 [Shapes](../02_Compute/01_Shapes.md)의 선택은 위치·지원·용량을 함께 고려한다.

## 공식문서

- [Regions and Availability Domains](https://docs.oracle.com/en-us/iaas/Content/General/Concepts/regions.htm): AD 격리, tenancy별 이름, 자원 scope, Regional subnet.
