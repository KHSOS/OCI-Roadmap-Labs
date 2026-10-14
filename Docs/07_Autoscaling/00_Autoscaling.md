# Autoscaling

Autoscaling은 Instance Pool의 instance 수 또는 lifecycle state를 policy에 따라 자동 조정하는 기능이다. 수요가 늘 때 scale-out하고 줄 때 scale-in하여 성능과 비용 사이의 균형을 맞춘다. 개별 VM의 CPU·RAM을 바꾸는 vertical scaling과는 다르다.

## 구성 요소의 연결

Image는 OS와 software의 출발점이고, Instance Configuration은 image·shape·metadata 등 새 instance의 생성 설정이다. Instance Pool은 실제 instance들을 한 그룹으로 관리한다. Autoscaling Configuration은 그 Pool에 연결되어 언제 무엇을 바꿀지 정하는 policy를 포함한다.

LB/NLB를 Pool에 연결할 수 있지만 Autoscaling의 필수 구성은 아니다. Pool의 instance 수가 늘어나는 것과 client 트래픽이 새 instance로 분산되는 것은 별도의 관계이며, 애플리케이션 준비·network 경로·backend health가 필요하다.

## 두 방식의 선택

| 방식 | 기준 | 적합한 상황 |
|---|---|---|
| Metric-based | Pool instance의 성능 metric과 threshold | 수요가 불규칙하고 CPU 등의 실제 부하에 따라 수량을 조정할 때 |
| Schedule-based | 특정 UTC 일정과 target size 또는 lifecycle action | 업무 시간·예정된 행사 등 수요 패턴을 미리 알 때 |

한 Pool에는 Autoscaling Configuration 하나만 연결할 수 있다. Configuration은 metric policy 하나 또는 schedule policy 여러 개를 가지며, schedule policy는 최대 50개다. 두 방식을 서로 다른 controller로 동시에 같은 Pool에 연결하는 모델은 아니다.

## 증가·감소·정지의 차이

Scale-out은 Pool에 instance를 추가하고 scale-in은 instance를 terminate한다. Scale-in은 AD 간 수량, FD 간 수량을 균형화한 뒤 같은 FD에서는 오래된 instance부터 제거한다. 특정 instance가 계속 남는다고 가정하는 상태 저장 방식은 적합하지 않다.

Schedule의 Stop/Start는 수량 축소·삭제와 다른 lifecycle action이다. 정지 후 다시 시작하는 경우 instance의 정보는 보존된다. 정지가 모든 저장 비용을 없앤다는 뜻도 아니다. 서비스의 정상 운용에는 자동 생성된 instance가 같은 애플리케이션 역할을 수행하고 제거돼도 필요한 데이터가 유지되는 설계가 중요하다.

## 지원 조건과 실제 제약

공식 문서는 standard, dense I/O, GPU shape를 사용하는 VM 및 bare metal Pool을 지원 범위로 설명한다. Metric 방식에는 instance Monitoring이 활성화되고 서비스가 실제 metric을 수신해야 한다. 지원 instance의 Pool 생성에서는 configuration 설정과 관계없이 Monitoring이 기본 활성화되지만 실제 전달과 수신은 별개의 조건이다.

Maximum 또는 schedule target 수량만큼 생성 가능한 Service Limit·quota·shape capacity·Subnet IP 여유가 필요하다. 목표 수량 설정은 capacity 예약이나 즉시 application-ready 보장이 아니다. Pool 외의 VM, boot volume, image와 관련 network 자원의 비용은 Pool size 상한만으로 제한되지 않는다.

## 함께 읽기

- [Images](01_Images.md)
- [Instance Configurations](02_Instance_Configurations.md)
- [Instance Pools](03_Instance_Pools.md)
- [Metric-based Autoscaling](04_Metric_based_Autoscaling.md)
- [Schedule-based Autoscaling](05_Schedule_based_Autoscaling.md)
- [보존된 Hands-on 초안](../../Hands-on/Original-Provisional/07_Autoscaling/00_Autoscaling.md) — 실습 절차는 별도 자료에서 다룬다.

## 공식 참고 문서

- [Autoscaling](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/autoscalinginstancepools.htm)
- [Creating Instance Pools](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/creatinginstancepool.htm)
- [Managing Instance Configurations](https://docs.oracle.com/en-us/iaas/Content/Compute/Concepts/instancemanagement.htm)
