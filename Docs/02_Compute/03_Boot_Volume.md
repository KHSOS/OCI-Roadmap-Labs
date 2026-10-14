# Boot Volume

Boot Volume은 Compute instance를 부팅하는 image를 담는 분리 가능한 block storage다. platform image 또는 custom image로 instance를 만들면 새 boot volume이 같은 Compartment에 생성된다. instance의 실행 자원과 OS·데이터를 보관하는 volume의 수명은 구분된다. [Oracle: Working with Boot Volumes](https://docs.oracle.com/en-us/iaas/Content/Block/Concepts/bootvolumes.htm)

## OS와 데이터 disk의 관계

Boot Volume은 OS 부팅을 위한 disk이고, [Block Volume](04_Block_Volume.md)은 application 데이터 저장 용량을 별도로 확장할 수 있는 disk다. 둘 다 Block Volume 서비스의 자원이며 at-rest 암호화가 제공된다. instance에 연결되는 volume은 같은 AD에 있어야 한다. [Block Volume 개요](https://docs.oracle.com/en-us/iaas/Content/Block/Concepts/overview.htm)

application 데이터를 별도 disk에 둔다고 backup이 자동 완성되는 것은 아니다. 마찬가지로 boot volume에 OS와 application 설정이 남아 있다는 것과 재해 복구가 준비되었다는 것도 다른 문제다.

## boot-selection

선택 image의 기본 boot 크기와 필요한 OS 공간이 출발점이다. 일반적인 Linux·Windows image의 custom boot 크기는 image의 기본 크기와 50 GB 중 더 큰 값보다 커야 한다. Windows Server 2012 R2 Datacenter와 2021년 10월 이전에 게시된 Windows platform image에는 50 GB 대신 256 GB 기준이 적용된다. 모든 image의 default가 50 GB라는 뜻은 아니다. volume 크기를 늘려도 guest partition·filesystem에서 새 공간을 이용하려면 확장 작업이 추가로 필요할 수 있다.

성능은 volume 크기와 performance level에도 영향을 받는다. 저장 용량과 성능을 같은 선택으로 단순화하지 않고 OS·application의 I/O 요구를 함께 고려한다. 기존 단일 launch의 boot 선택 자료는 [보존된 Boot Volume Hands-on](../../Hands-on/Original-Provisional/02_Compute/03_Boot_Volume.md#실습절차)에 있다.

## 보존·재사용·복구의 차이

instance를 종료할 때 boot volume을 보존할 수 있다. 보존한 boot volume으로 호환되는 새 instance를 만드는 것은 기존 OS·설정을 재사용하는 방법이다. 문제 진단을 위해 정지한 instance의 boot volume을 분리하고 다른 instance의 data volume으로 연결하는 방식도 있다. 이는 부팅 disk의 역할과 데이터 disk의 연결 방식을 구분해야 이해할 수 있다.

보존 volume, backup, 비동기 replication은 서로 같은 기능이 아니다. volume group은 boot와 data volume을 함께 관리·backup·clone하는 범위가 될 수 있다. 각각의 복구 목표와 데이터 일관성을 별도로 고려해야 한다.

## 데이터 수명과 비용의 한계

instance 종료와 boot 삭제는 별도 선택이며 보존한 volume은 storage 자원으로 남는다. OS가 부팅되지 않는다고 바로 disk를 새로 format하는 것은 복구가 아니라 데이터 소실을 일으킬 수 있다. 암호화는 접근 통제나 backup을 대체하지 않는다. [Compute](00_Compute.md)의 실행 수명과 storage 수명을 나누어 이해하는 것이 중요하다.


## 공식문서

- [Working with Boot Volumes](https://docs.oracle.com/en-us/iaas/Content/Block/Concepts/bootvolumes.htm): 생성·보존·repair·custom size·성능·replication.
- [Overview of Block Volume](https://docs.oracle.com/en-us/iaas/Content/Block/Concepts/overview.htm): boot/data volume, AD 제약과 암호화.
