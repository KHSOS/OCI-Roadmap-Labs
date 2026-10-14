# Images

Image는 instance의 OS와 software 구성을 결정하는 virtual hard drive template다. 새 instance는 선택한 Image를 출발점으로 boot volume을 사용해 시작한다. Image와 실행 중인 VM, Instance Configuration은 서로 다른 자원이다.

## Platform Image와 Custom Image

| 선택 | 역할 | 주로 쓰는 경우 |
|---|---|---|
| Platform Image | OCI가 제공하는 OS별 기본 image | 일반 OS를 기반으로 초기 구성과 package 설치를 자동화할 때 |
| Custom Image | 기존 instance의 boot disk에서 보존한 software·설정 | 공통 application 구성을 여러 새 instance에 재사용할 때 |

Custom Image에는 capture 시점에 boot disk에 설치된 application과 설정이 들어간다. 연결된 Block Volume의 데이터는 포함하지 않는다. 데이터를 포함하는 boot disk를 capture하므로 credentials·private key·민감 정보가 이미지 이용자에게 전달될 위험도 고려해야 한다. Image 저장은 별도 비용 요소다.

## Pool과 bootstrap의 관계

Instance Configuration은 Image와 shape·metadata 등을 참조하고, Pool은 그 설정으로 실제 instance를 만든다. 기존 instance에서 Configuration을 만들었다고 설치된 파일까지 복제되는 것은 아니다. 설치 파일을 담는 Custom Image와 시작 시 초기화를 수행하는 user data는 서로 다른 역할이다.

Platform Image에는 cloud-init 또는 Windows의 cloudbase-init을 통해 launch 초기화를 수행하는 경로가 있다. 공통 software를 Image에 미리 담으면 시작 시 설치 의존성을 줄일 수 있고, launch 시 설치하면 최신 package 선택과 외부 repository 접근에 의존한다. Custom Image에서 이미 담긴 software를 확인하는 작업과 새로 재설치하는 작업을 혼동하지 않는 것이 중요하다.

## 호환성을 결정하는 선택

OS 이름뿐 아니라 architecture, image build, shape 및 image capabilities가 맞아야 한다. Arm용 `aarch64` image와 x86 image는 같은 선택이 아니다. GPU·Windows 등은 지원 shape와 driver 조건도 다르다. Custom Image는 base image의 기본 compatible shape를 상속하지만 모든 hardware에서 정상 boot하는 보장은 아니다.

Platform Image 목록과 release notes는 계속 바뀐다. 선택한 OS의 support lifecycle, 보안 업데이트 책임, 사용자 이름과 초기 firewall도 application 운용에 영향을 준다. 오래된 Image에서 새 instance를 만들 수 있는 것과 지원·업데이트가 계속되는 것은 다르다.

## capture와 수명의 한계

Running instance의 Custom Image 생성은 source를 shutdown하여 몇 분간 unavailable하게 하고 완료 후 재시작한다. 따라서 live workload에 영향이 없는 복사로 보면 안 된다. 같은 source의 image 생성 중에는 추가 capture가 제한되며 공식 문서는 시작 후 20분 timeout을 설명한다.

Image가 생성되었다는 상태는 새 instance에서 application이 정상 동작한다는 의미가 아니다. Image 삭제와 이미 launch된 instance의 수명도 구분된다. Custom Image를 다른 Region에서 활용하려는 설계는 image 이동 관련 기능과 지원 조건을 따로 고려해야 하며 자동 cross-region replication으로 가정하지 않는다.

## 함께 읽기

- [Instance Configurations](02_Instance_Configurations.md)
- [Instance Pools](03_Instance_Pools.md)
- [보존된 Hands-on 초안](../../Hands-on/Original-Provisional/07_Autoscaling/01_Images.md) — 실습 절차는 별도 자료에서 다룬다.

## 공식 참고 문서

- [Platform Images](https://docs.oracle.com/en-us/iaas/Content/Compute/References/images.htm)
- [Managing Custom Images](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/managingcustomimages.htm)
- [Creating an Instance Configuration](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/creatinginstanceconfig.htm)
