# Schedule-based Autoscaling

Schedule-based Autoscaling은 지정한 UTC 일정에 Instance Pool의 target size 또는 lifecycle state를 변경한다. 업무 시간, 예정된 행사, batch 처리처럼 수요를 미리 예측할 수 있을 때 적합하다. CPU metric threshold를 평가하는 방식과 다르다.

## 수량 변경과 lifecycle action

Scale action은 Pool을 지정한 target size로 만든다. 증가량을 더하는 상대 변경과 다르다. Scale-in으로 수량을 줄이면 instance가 terminate된다. 반면 Stop/Start 또는 Reboot는 instance의 lifecycle을 바꾸며 Stop 후 Start에서는 instance의 정보가 보존된다.

업무 시작 전 확대와 종료 후 축소에는 서로 다른 policy 두 개가 필요하다. Stop과 Start 역시 각각 별도의 policy다. 한 Configuration은 여러 schedule policy를 포함할 수 있으며 최대 50개를 지원한다. 같은 Pool에 별도 metric Configuration을 함께 연결하는 방식은 지원되지 않는다.

## 일정의 표현과 시간대

Schedule은 one-time 또는 recurring일 수 있고 Quartz cron 표현식을 사용한다. 형식은 second·minute·hour·day of month·month·day of week·선택적 year의 6 또는 7개 field다. 일반 Unix 5-field cron과 그대로 호환되는 것이 아니다. 날짜를 정하면 day of week에 `?`를, 요일을 정하면 day of month에 `?`를 사용하는 의미를 구분한다.

모든 시각은 UTC다. API/CLI/SDK의 second 값은 0이어야 하며 Console에서는 second 값을 입력하지 않는다. 지역 시간대를 그대로 넣으면 의도한 업무 시각과 달라진다. one-time 일정과 반복 일정의 구분은 특히 year·date 표현과 관련된다.

## event 이후의 상태

Policy가 실행된 뒤 Pool은 다른 동작이 변경하기 전까지 target size 또는 lifecycle state를 유지한다. 수동으로 수량이나 state를 바꾸면 schedule은 다음 실행 시점까지 이를 자동으로 원래 값에 맞추지 않는다. 계속 목표값을 유지하는 실시간 reconciliation controller로 생각하면 안 된다.

Metric 방식의 cooldown 속성이 Configuration에 보여도 schedule 실행에는 영향을 주지 않는다. 미래 Pool size forecast는 저장된 일정의 효과를 보여 주지만 실제 capacity 확보나 application 준비 성공을 보장하지 않는다.

## 일정 충돌과 선택의 한계

같은 시점의 충돌에서는 lifecycle policy 하나와 size policy 하나를 선택하고 lifecycle action이 먼저 실행된다. Lifecycle 우선순위는 Force reboot → Reboot → Start → Force stop → Stop이다. Size policy는 instance 수가 가장 큰 것을 선택한다. 여러 축소 일정이 있다고 가장 작은 수량이 우선하는 것은 아니다.

예정된 scale-out도 Service Limit·quota·shape capacity·Subnet IP 제약을 받는다. 정지는 저장 비용을 자동으로 없애지 않으며 반복 일정이 남으면 나중에 resource state가 다시 바뀔 수 있다. 필요한 시작·종료 시각과 최대 target 수량, 남아 있는 일정의 수명을 함께 설계해야 한다.

## 함께 읽기

- [Autoscaling](00_Autoscaling.md)
- [Instance Pools](03_Instance_Pools.md)
- [Metric-based Autoscaling](04_Metric_based_Autoscaling.md)
- [보존된 Hands-on 초안](../../Hands-on/Original-Provisional/07_Autoscaling/05_Schedule_based_Autoscaling.md) — 실습 절차는 별도 자료에서 다룬다.

## 공식 참고 문서

- [Autoscaling](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/autoscalinginstancepools.htm)
