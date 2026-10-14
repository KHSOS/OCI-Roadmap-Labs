# Compute

OCI Compute는 instance라고 부르는 실행 host를 제공하고 관리하는 서비스다. instance는 OS와 application이 실행되는 환경이며, image·shape·network·storage를 조합해 workload 요구에 맞춘다. OCI API로 instance를 관리하는 것과 guest OS 안에서 application을 운영하는 것은 서로 다른 관리 계층이다. [Oracle: Overview of the Compute Service](https://docs.oracle.com/en-us/iaas/Content/Compute/Concepts/computeoverview.htm)

## VM과 Bare Metal의 역할

**Virtual Machine (VM)**은 물리 hardware 위에 격리된 실행 환경을 제공한다. 물리 서버 전체의 자원이 필요하지 않은 application에 적합하다. **Bare Metal**은 전용 물리 서버 접근이 필요한 성능·격리 요구를 위한 선택이다. 더 큰 형태를 고르는 것보다 workload가 요구하는 CPU·메모리·network·storage 특성을 맞추는 것이 중요하다.

## Instance 구성 요소의 관계

하나의 instance는 다음 요소의 관계로 이해할 수 있다. **Image**는 OS와 초기 software의 원본, **Shape**는 CPU·메모리 등의 자원 규격, **AD/FD**는 배치 범위, **VNIC/subnet**은 network 연결, **Boot Volume**은 부팅에 필요한 지속 storage다. Linux의 SSH public key는 원격 사용자 인증에 쓰이며 user data는 지원 image의 초기 구성에 쓰인다. 이 요소들을 따로 배운다고 instance를 각각 하나씩 만들 필요는 없다.

예를 들어 작은 Linux application VM은 호환 image와 shape, 하나의 subnet, 부팅 volume을 함께 사용한다. web server 설치나 public IP 부여는 application과 접속 요구에 따른 별도 선택이며 Compute의 정의 자체는 아니다. 기존의 단일 instance 구성 자료는 [보존된 공동 launch Hands-on](../../Hands-on/Original-Provisional/02_Compute/00_Compute.md#공유-launch-station--생성-전-선택부터-실제-제출까지)에 있다.

## IAM과 guest 인증의 경계

OCI IAM은 누가 instance 같은 OCI 자원에 어떤 API 작업을 할 수 있는지 제어한다. Linux SSH key pair는 guest 사용자 인증을 위한 것이다. SSH의 사용자 key와 접속 대상 서버를 식별하는 host key도 역할이 다르다. 사용자 private key를 가졌다는 사실만으로 접속한 서버가 기대한 instance라는 의미는 아니다. 기존 대상 식별 자료는 [보존된 SSH 신뢰 경로](../../Hands-on/Original-Provisional/02_Compute/00_Compute.md#최초-ssh-fingerprint의-신뢰-경로)에 있다. [SSH Keys](05_SSH_Keys.md)에서 공개키·개인키 관계를 읽을 수 있다.

## 자원 선택과 수명주기

[Shapes](01_Shapes.md)는 workload의 자원 크기를 결정한다. [Burstable Instances](02_Burstable_Instances.md)는 평소 CPU 사용량이 낮고 가끔 증가하는 workload를 위한 옵션이다. [Boot Volume](03_Boot_Volume.md)은 OS를, [Block Volume](04_Block_Volume.md)은 분리된 데이터 disk를 이해하는 출발점이다.

On-demand, preemptible, reserved, dedicated capacity는 CPU 크기와 별도의 capacity 선택이다. service limit·quota가 남아 있어도 원하는 AD에 실제 host capacity가 부족할 수 있다. instance를 정지하거나 종료하는 상태 변화와 storage 데이터의 보존 여부도 같은 개념이 아니다. 연결 storage와 network 비용은 Compute 비용과 별도로 발생할 수 있다.

## 실행 환경과 서비스 완성의 차이

instance의 RUNNING 상태는 application의 준비 완료나 SSH 접속 가능성을 보장하지 않는다. network 경로·허용 규칙·guest service·인증·초기화가 각각 맞아야 한다. 또한 instance 하나가 살아 있다는 사실과 application의 고가용성은 다르다. 초기 workload를 이해할 때는 instance·network·storage의 관계를 먼저 보고 필요한 운영·복구 기능을 추가하는 편이 명확하다.

## Hands-on 연결

기존 SSH·초기화·instance 상태 확인 자료는 [보존된 Compute Hands-on](../../Hands-on/Original-Provisional/02_Compute/00_Compute.md)에 있다. 이 문서는 명령을 실행하지 않고 Compute의 역할과 선택 범위를 설명한다.


## 공식문서

- [Overview of the Compute Service](https://docs.oracle.com/en-us/iaas/Content/Compute/Concepts/computeoverview.htm): instance 종류, 구성 요소, capacity, 비용과 제한.
- [Managing Key Pairs on Linux Instances](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/managingkeypairs.htm): Linux 사용자 인증.
- [Connecting to a Linux Instance](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/connect-to-linux-instance.htm): 접속 대상·사용자·private key와 최초 서버 fingerprint.
