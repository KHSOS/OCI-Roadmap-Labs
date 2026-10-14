# Shapes

Shape는 instance에 할당되는 OCPU·메모리와 그 밖의 자원을 정하는 규격이다. Image가 실행할 OS와 software의 원본이라면 shape는 그것을 실행할 hardware 자원의 특성을 정한다. 동일한 application도 CPU architecture·메모리·network 요구에 따라 적합한 shape가 달라진다. [Oracle: Compute Shapes](https://docs.oracle.com/en-us/iaas/Content/Compute/References/computeshapes.htm)

## 자원 규격을 읽는 방법

OCPU와 vCPU는 항상 같은 수가 아니다. x86 AMD/Intel에서는 1 OCPU가 두 hardware execution thread, 즉 2 vCPU에 대응한다. Ampere A1에서는 1 OCPU가 1 core/1 vCPU다. Arm 계열 전체를 A1과 같은 비율로 일반화하면 안 된다. Oracle 문서는 A2와 A4에 다른 OCPU 대응을 설명한다.

메모리, 최대 network bandwidth, VNIC 수와 local disk도 shape별 특성이다. CPU 수만 같다고 서로 같은 성능이나 같은 application 호환성을 뜻하지 않는다. network bandwidth 표는 VCN 내부에서 기대되는 대역폭을 기준으로 읽어야 하며 인터넷 application의 end-to-end 속도를 보장하는 값은 아니다.

## shape-selection

Flexible shape는 OCPU와 메모리를 workload에 맞게 조정할 수 있다. 메모리를 자유롭게 무제한 선택하는 기능은 아니며 shape별 OCPU당 memory 비율과 전체 상한이 있다. CPU 연산 위주의 작업은 core 수를, in-memory 작업은 memory 비율을 함께 고려한다. 예를 들어 A1과 E4는 서로 다른 processor architecture이므로 같은 OS 이름만 보고 image와 application binary를 호환된다고 볼 수 없다.

기존 단일 VM의 image·shape 선택 자료는 [보존된 Shapes Hands-on](../../Hands-on/Original-Provisional/02_Compute/01_Shapes.md#실습절차)에 있다. [Compute](00_Compute.md)의 image·배치·network·boot 관계와 함께 읽으면 shape만으로 실행 환경을 결정할 수 없는 이유를 이해할 수 있다.

## workload에 따른 계열 선택

| 계열 | 주로 고려하는 요구 | 함께 볼 경계 |
|---|---|---|
| Standard | 일반적인 application의 균형 | CPU·memory·network 크기 |
| DenseIO | 높은 local storage 성능 | local NVMe의 데이터 보호와 수명 |
| GPU | hardware 가속 workload | GPU 종류·memory와 software 호환성 |
| HPC / Optimized | 높은 CPU frequency와 고성능 연산 | network와 workload 지원 조건 |

이는 workload 분류이지 특정 계열이 모든 작업에서 빠르다는 순위가 아니다. [Burstable Instances](02_Burstable_Instances.md)는 지원 shape에서 CPU 사용 패턴에 따라 선택하는 별도 특성이다.

## 선택의 한계

공식 shape 목록에 있다는 것과 현재 tenancy·Region·AD에서 사용할 수 있다는 것은 다르다. service limit, quota, 실제 host capacity와 image 지원을 함께 고려해야 한다. 이전 세대 shape도 지원될 수 있지만 높은 수요의 Region에서는 capacity가 제한될 수 있다. 가격·지원 범위·workload 성능을 고려하지 않은 단순 증설은 비용 최적화를 보장하지 않는다.

## 공식문서

- [Compute Shapes](https://docs.oracle.com/en-us/iaas/Content/Compute/References/computeshapes.htm): OCPU/vCPU, Flexible shapes, 계열, image·Region 지원, 이전 세대.
