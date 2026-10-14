# Metric-based Autoscaling

Metric-based Autoscaling은 Instance Pool의 성능 metric이 threshold 조건을 만족할 때 수량을 늘리거나 줄인다. 시간표보다 실제 수요를 따라가려는 workload에 적합하며, 한 VM의 shape를 바꾸는 기능이 아니다.

## metric이 policy로 연결되는 방식

Monitoring이 수집한 성능 metric을 1분 단위로 집계한 뒤 Pool의 모든 instance에 대해 평균한다. 세 번 연속된 1분 평균값이 policy의 threshold 조건을 만족하면 scaling event가 발생한다. 잠깐의 CPU spike 하나와 즉시 scale-out은 같은 조건이 아니다.

CPU utilization이나 지원되는 성능 metric을 선택하고 scale-out/scale-in threshold와 증가·감소량, minimum·initial·maximum 수량을 함께 정한다. 실제 application의 병목이 CPU인지, threshold가 사용자 체감 성능과 연결되는지에 따라 선택이 달라진다. custom metric 또는 임의의 application 응답시간을 자동으로 지원한다고 가정하면 안 된다.

## Pool 평균을 읽는 예

두 member 중 하나만 바쁘고 다른 하나는 idle하면 Pool 평균은 바쁜 member 하나의 값보다 낮다. Scale-out 이후 평균이 낮아질 수 있으므로 단일 instance 차트와 Pool policy 평가를 혼동하면 안 된다. 여러 core 중 일부만 사용한 부하 역시 CPU utilization에 다르게 나타난다.

새 member의 수량이 늘어난 뒤에도 실제 traffic 분산과 application 초기화가 따라오지 않으면 기존 member의 부하는 줄지 않을 수 있다. Pool·Configuration·LB attachment·network·application health의 관계를 함께 고려해야 한다.

## Cooldown과 수량 제한

Cooldown은 Pool이 Running 상태에 도달한 때부터 시작하는 event 간 안정화 기간이다. 그 기간에도 metric 평가는 계속되고, 종료 후 필요하면 다시 수량을 조정한다. application의 모든 요청이 정상이라는 의미의 대기시간은 아니다. Schedule-based 방식에서는 같은 cooldown 속성이 보여도 schedule 동작에 영향을 주지 않는다.

Minimum은 줄어들 수 있는 하한, Maximum은 늘어날 수 있는 상한이며 Initial 값 변경도 주의가 필요하다. 이미 scale된 Pool에서 Initial을 현재 수량보다 낮추면 instance가 terminate될 수 있다. 값 변경을 무해한 표시 변경으로 보면 안 된다.

## 전제와 한계

Metric 방식에는 instance Monitoring 활성화와 실제 metric 수신이 필요하다. 지원 image·agent와 service에 도달하는 network 경로가 없으면 policy가 저장되어도 부하를 평가할 수 없다. 한 Pool에는 Autoscaling Configuration 하나와 metric policy 하나를 사용하는 모델이다.

Capacity·quota·Subnet IP 부족은 threshold 충족과 별개의 생성 제약이다. Scale-in은 instance를 terminate하므로 local state를 지속 데이터로 취급하면 안 된다. Threshold·cooldown·수량 선택은 성능과 비용의 타협이며 특정 숫자가 모든 workload의 권장값은 아니다.

## 함께 읽기

- [Autoscaling](00_Autoscaling.md)
- [Instance Pools](03_Instance_Pools.md)
- [Schedule-based Autoscaling](05_Schedule_based_Autoscaling.md)
- [보존된 Hands-on 초안](../../Hands-on/Original-Provisional/07_Autoscaling/04_Metric_based_Autoscaling.md) — 실습 절차는 별도 자료에서 다룬다.

## 공식 참고 문서

- [Autoscaling](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/autoscalinginstancepools.htm)
