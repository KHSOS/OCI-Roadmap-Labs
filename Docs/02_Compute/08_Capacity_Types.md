# Capacity Types

Capacity Types는 Compute instance를 시작할 때 **어떤 host capacity를 사용할지** 정하는 선택이다. Shape가 CPU·메모리 등 자원의 크기를 정한다면 Capacity Type은 용량 확보·회수 가능성·물리 host의 전용성이라는 다른 요구를 다룬다.

## 네 가지 선택의 차이

|유형|핵심 역할|선택 사례와 대가|
|---|---|---|
|On-demand|현재 가용한 용량에서 필요한 Compute 사용|일반 서비스에 적합하지만 launch 시 capacity 가용성을 보장하지 않는다.|
|Preemptible|필요할 때 OCI가 회수할 수 있는 낮은 비용의 용량|중단 후 재시도 가능한 테스트·배치에 적합하며 언제든 종료될 수 있다.|
|Capacity Reservation|특정 조건의 미래 instance launch를 위한 용량 확보|정해진 AD·shape 등의 용량이 중요할 때 고려하며 미사용 예약 용량도 별도 metering 대상이다.|
|Dedicated Host|다른 고객과 공유하지 않는 host에서 VM 실행|격리·규제 또는 host 기반 라이선스 요구에 적합하며 host 전체 운영·비용을 고려해야 한다.|

예약은 할인 구매와 같은 뜻이 아니고, Dedicated Host는 고객이 데이터센터 전체를 운영하는 Private Cloud와 같은 뜻이 아니다. 생성 시 capacity 선택은 서로 배타적이다.

## 워크로드가 선택을 결정한다

다시 시작할 수 있는 테스트와 결과를 외부에 저장하는 batch는 Preemptible의 후보다. 반면 중단이 어려운 stateful 서비스는 회수 가능성을 그대로 받아들이기 어렵다. 특정 시점의 대규모 launch가 중요하면 reservation이 해결하는 문제가 있고, 물리 host 공유 금지가 중요하면 dedicated capacity가 해결하는 문제가 있다. 이는 설계 예시이며 각 타입이 항상 더 저렴하다는 비교는 아니다.

## 위치·수량과의 관계

예약을 사용하는 VM은 예약과 같은 AD에 있어야 하고 예약 구성에 맞는 자원을 소비한다. VM 종료 시 사용한 capacity는 reservation으로 돌아간다. Dedicated Host에 놓는 VM은 host가 지원하는 shape와 해당 VM에 필요한 여유 capacity 조건을 따른다. Service limit와 Compartment quota는 모든 capacity type에 적용되며, quota가 남았다는 사실만으로 host capacity가 있다는 뜻은 아니다.

## Preemptible의 중요한 한계

최소 실행 시간이 보장되지 않으며 회수되면 instance가 종료된다. 생성 후 start·stop·reboot를 지원하지 않고 On-demand로 변환할 수 없다. 지원 shape와 기능에도 제한이 있으며 instance configurations 및 instance pools에 사용할 수 없다. Boot volume을 회수 시 삭제할지의 선택은 데이터 보존과 관련되므로 CPU 비용만으로 전체 비용이나 데이터 수명을 판단하지 않는다.

## 관련 개념

- [Compute](00_Compute.md)
- [Shapes](01_Shapes.md)
- [Boot Volume](03_Boot_Volume.md)

## Hands-on으로 이어가기

개념 설명과 자원 생성 절차는 구분된다. 기존 작업·검증·정리 내용은 [보존된 Hands-on](../../Hands-on/Original-Provisional/02_Compute/08_Capacity_Types.md)에서 읽을 수 있다.

## 공식문서

- [Overview of the Compute Service](https://docs.oracle.com/en-us/iaas/Content/Compute/Concepts/computeoverview.htm)
- [Capacity Reservations](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/reserve-capacity.htm)
- [Dedicated Virtual Machine Hosts](https://docs.oracle.com/en-us/iaas/Content/Compute/Concepts/dedicatedvmhosts.htm)
- [Placing Instances on a Dedicated Virtual Machine Host](https://docs.oracle.com/en-us/iaas/Content/Compute/Concepts/dedicatedvmhosts_topic-Placing_an_Instance_on_a_Dedicated_Virtual_Machine_Host.htm)
- [Preemptible Instances](https://docs.oracle.com/en-us/iaas/Content/Compute/Concepts/preemptible.htm)
- [Creating an Instance](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/launchinginstance.htm)
