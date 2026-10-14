# Burstable Instances

Burstable instance는 평소에는 낮은 CPU baseline을 사용하고 가끔 더 높은 CPU 성능을 사용하는 VM이다. 대부분의 시간에 idle이거나 CPU 사용률이 낮은 microservice, 개발·테스트, CI/CD 도구, monitoring, 정적 website 같은 workload를 위한 선택이다. [Oracle: Burstable Instances](https://docs.oracle.com/en-us/iaas/Content/Compute/References/burstable-instances.htm)

## Baseline과 burst의 의미

설정은 **total OCPU**와 **baseline utilization per OCPU**를 구분한다. baseline은 각 CPU의 12.5% 또는 50%이며 지속적으로 사용할 수 있는 최소 CPU 수준이다. 예를 들어 total 1 OCPU와 baseline 12.5%는 평소 1/8 OCPU를 기준으로 하고 조건이 허용될 때 total 1 OCPU까지 사용하는 모델이다. 이것은 VM의 수를 늘리는 autoscaling이 아니다.

burst 허용은 이전 사용 패턴과 물리 host의 자원 사용량에 따라 달라진다. 연속 burst 상한은 최대 한 시간이지만 항상 한 시간 또는 필요한 순간의 full CPU가 보장되는 것은 아니다. burst가 끝나면 baseline으로 제한된다.

## 일반 Flexible instance와의 선택

대부분의 시간에 작은 CPU만 필요한 workload에는 baseline 기반 요금이 유용할 수 있다. 반면 CPU 전체를 지속적으로 사용하거나 필요한 순간의 성능을 확정해야 하는 critical workload에는 regular instance가 적합하다. regular Flexible instance는 할당 OCPU·memory를 크기 변경으로 조정하고, burstable은 설정된 baseline과 maximum 사이에서 가용 CPU가 동적으로 변한다. [Shapes](01_Shapes.md)의 크기 선택과 같은 뜻은 아니다.

## CPU 이외의 자원과 비용

메모리는 burst하지 않으며 설정된 메모리 크기로 과금된다. CPU 비용은 실제 사용률이 baseline보다 낮거나 burst하더라도 설정한 baseline OCPU를 기준으로 한다. network bandwidth도 oversubscription의 영향을 받아 최대값 사용이 보장되지 않는다. 따라서 CPU 비용 감소를 전체 VM 비용이 같은 비율로 감소한다고 해석하면 안 된다.

## 지원과 제약

Oracle의 지원 목록은 Standard3 Flex, E3 Flex, E4 Flex, E5 Flex를 포함한다. A1처럼 다른 Flexible shape에 기능을 자동 확대해서는 안 된다. Burstable은 paravirtualized networking을 사용하며 dedicated host, preemptible capacity, capacity reservation, extended memory VM과 함께 사용할 수 없다. 서비스 한도와 compartment quota는 실제 사용률이 아닌 설정 baseline OCPU로 계산된다.

짧은 CPU task의 성공이나 평균 CpuUtilization 하나만으로 burst 성능 보장을 판단할 수 없다. monitoring은 시간창·집계·agent 상태를 포함한 관측이며 workload의 지속 성능 요구와 구분해야 한다. 기존 생성·짧은 CPU 관찰 자료는 [보존된 Hands-on](../../Hands-on/Original-Provisional/02_Compute/02_Burstable_Instances.md)에 있다.

## 공식문서

- [Burstable Instances](https://docs.oracle.com/en-us/iaas/Content/Compute/References/burstable-instances.htm): baseline·burst 동작, supported shapes, 제한과 billing.
