# Instance Pools

Instance Pool은 같은 Region의 여러 Compute instance를 그룹으로 생성·관리하는 자원이다. Instance Configuration을 template로 사용하며 target size와 placement를 갖는다. Pool은 template를 저장하는 Configuration과 실제 실행 member 사이의 관리 계층이다.

## 무엇을 관리하는가

Pool size는 목표 instance 수다. 새 member는 Configuration의 image·shape·metadata 등에 따라 생성되며, Pool은 member 목록과 배치를 관리한다. 기존 instance를 Pool에 연결하는 선택도 있지만 그 instance의 기존 software가 자동으로 template와 같아진다고 보면 안 된다.

독립적인 web worker처럼 같은 역할의 instance를 반복 배치할 때 유용하다. member마다 launch 초기화가 성공하고 필요한 데이터가 instance 교체에도 유지되도록 application을 설계해야 한다. Running 상태와 application-ready 상태는 다르다.

## AD·FD·Subnet의 선택

Pool placement는 AD와 Subnet을 정의하고 Configuration의 AD/Subnet보다 우선한다. 여러 AD에 배치하는 선택도 있다. regional Subnet은 AD를 넘어 배치하기에 편리하지만 Pool이 자동으로 모든 AD를 사용한다는 뜻은 아니다.

FD를 비워 두면 서비스는 capacity에 따라 best-effort 분산한다. FD를 지정하면 선택한 FD들에 균등 배치를 요구하므로 capacity가 부족할 때 launch나 scale이 실패할 수 있다. 더 강한 배치 제약과 가용 capacity 사이에 선택이 필요하다. Private Subnet은 member에 public IP를 허용하지 않지만 내부 client의 route·보안·OS firewall은 여전히 필요하다.

## Load Balancer와의 관계

Pool에 LB/NLB의 Backend Set을 선택적으로 연결하면 새 member가 backend로 추가된다. 연결할 port와 VNIC의 private IP가 서비스 경로를 결정한다. backend가 정상 상태에 도달한 뒤 client 트래픽이 전달된다.

이 attachment는 선택 기능이다. Pool-only workload에는 LB가 필요하지 않다. Pool 수량 증가가 backend application 설치·health와 client 접근을 대신하는 것도 아니다. 자동 attachment를 사용하지 않는 경우 application 트래픽 분산 방식은 별도로 설계한다.

## 수량·capacity·수명의 한계

Autoscaling은 같은 Pool의 수량 또는 lifecycle state를 policy로 조정하는 별도 controller다. 수동 size 변경과 자동 제어를 경쟁시키지 않는 관리 모델이 중요하다. Scale-in은 member를 terminate하므로 특정 member의 로컬 상태에 의존하면 데이터 손실 가능성이 있다.

Target size를 지정해도 shape capacity, Service Limit·quota, FD 제약과 Subnet IP가 부족하면 목표를 채울 수 없다. image source가 boot volume인 Configuration은 Pool 생성에 사용할 수 없다. Pool의 size 상한은 Pool 밖의 instance나 보존 boot volume·Custom Image 비용의 상한이 아니다. 정지·scale-in·Pool 종료와 저장 자원의 보존 정책을 구분해야 한다.

## 함께 읽기

- [Instance Configurations](02_Instance_Configurations.md)
- [Autoscaling](00_Autoscaling.md)
- [Load Balancer](../08_Load_Balancer/00_Load_Balancer.md)
- [보존된 Hands-on 초안](../../Hands-on/Original-Provisional/07_Autoscaling/03_Instance_Pools.md) — 실습 절차는 별도 자료에서 다룬다.

## 공식 참고 문서

- [Creating Instance Pools](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/creatinginstancepool.htm)
- [Creating an Instance Configuration](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/creatinginstanceconfig.htm)
- [Autoscaling](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/autoscalinginstancepools.htm)
