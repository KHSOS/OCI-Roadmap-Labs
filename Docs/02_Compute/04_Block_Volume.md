# Block Volume

OCI Block Volume은 instance에 연결해 일반 hard drive처럼 사용할 수 있는 지속 block storage를 제공한다. instance의 CPU·메모리와 별도로 저장 용량과 성능을 관리하고, volume을 분리해 다른 instance에 연결해도 데이터를 유지할 수 있다. 데이터는 volume을 다시 format하거나 삭제하기 전까지 남는다. [Oracle: Overview of Block Volume](https://docs.oracle.com/en-us/iaas/Content/Block/Concepts/overview.htm)

## Instance·Volume·Attachment의 관계

Instance는 실행 host, volume은 저장 자원, attachment는 둘의 연결이다. attachment를 제거하는 것은 volume 자체를 삭제하는 것과 다르다. [Boot Volume](03_Boot_Volume.md)은 부팅 image를 담는 특수한 용도의 volume이며 일반 data volume과 역할이 다르다.

예를 들어 application 데이터를 별도 volume에 두면 Compute instance의 교체와 데이터 보관을 나누어 생각할 수 있다. 다만 volume은 같은 AD의 instance에만 직접 연결할 수 있다. 다른 AD나 Region으로의 복구는 backup 복원·복사 또는 지원 replication 같은 별도 방식을 사용하며 기존 volume을 그대로 원격 연결하는 것이 아니다.

## 연결 방식의 선택

**Paravirtualized attachment**는 지원 VM에서 iSCSI 연결 명령 없이 disk를 연결하는 방식이다. **iSCSI attachment**는 guest에서 추가 연결 구성이 필요하며 Bare Metal 등에서 사용된다. attachment가 완료되어 disk가 보이는 것과 filesystem이 준비되어 application이 쓸 수 있는 것은 다르다. 기존 filesystem이 있는 volume을 새 disk처럼 format하면 데이터가 사라진다.

## 접근 모드와 공유의 한계

| 모드 | 의미 | 주의할 점 |
|---|---|---|
| Read/write | 연결 instance의 읽기·쓰기 | 기본적인 단일 instance 사용 |
| Read/write, shareable | 여러 instance의 동시 읽기·쓰기 연결 | cluster-aware solution이 없으면 데이터 손상 위험 |
| Read-only | 읽기만 허용 | guest의 다른 접근 경로까지 자동 통제하는 뜻은 아님 |

여러 instance에 연결할 수 있다는 기능은 일반 filesystem을 안전하게 동시 쓰기할 수 있다는 보장이 아니다. 여러 client가 파일을 공유하는 network filesystem과도 같은 모델이 아니다.

## 용량·성능·보호를 따로 고르기

Volume 크기와 performance level은 I/O 요구에 맞게 선택한다. Balanced, Higher Performance, Ultra High Performance, Lower Cost 같은 선택이 있으며 volume 크기도 성능에 영향을 준다. 모든 volume과 boot volume에는 at-rest 암호화가 제공된다. 서비스 내부의 중복 저장과 repair는 durability를 높이지만 AD 장애를 위한 정기 backup을 대신하지 않는다.

분리 후 재연결해 파일이 유지되는 예시는 persistence를 설명할 뿐 backup 복구나 AD 장애 복구의 검증은 아니다. 기존의 새 disk·filesystem·재연결 사례는 [보존된 Hands-on](../../Hands-on/Original-Provisional/02_Compute/04_Block_Volume.md)에 있다. Compute 종료, attachment 제거, volume 삭제는 서로 다른 수명주기 결정이다.

## 공식문서

- [Overview of Block Volume](https://docs.oracle.com/en-us/iaas/Content/Block/Concepts/overview.htm): 지속성, attachment·access type, AD 범위, 암호화·성능·durability.
