# Instance Configurations

Instance Configuration은 Compute instance를 생성할 때 사용할 설정 template다. Image·shape·metadata뿐 아니라 network 설정, Block Volume attachment, 선택적 capacity reservation 등의 관계를 담을 수 있다. 설정 자원 자체는 실행 중인 VM이 아니다.

## Image·Pool과의 역할 분담

Image가 OS와 저장된 software 내용의 출발점이라면 Configuration은 어떤 Image·shape·초기화 입력으로 만들지 정한다. Pool은 그 설정을 사용해 실제 instance 그룹을 관리한다. Configuration은 Pool 외의 개별 instance 생성 template로도 사용할 수 있다.

웹 application의 반복 배포를 생각하면 Custom Image의 공통 software와 Configuration의 public SSH key·user data·network 설정이 서로 다른 부분이다. 사용자 데이터에 credential을 넣는 방식은 reusable template가 비밀을 퍼뜨릴 위험을 만든다.

## 두 생성 모델의 차이

- **기존 instance를 template로 사용**: 기존 instance의 설정을 기준으로 한다. boot volume의 application·binary·파일과 연결된 Block Volume의 내용은 포함하지 않는다. 생성될 instance의 Compartment는 Configuration을 둔 Compartment가 아니라 기준 instance의 Compartment를 따른다.
- **설정 목록을 직접 제공**: Image·shape·metadata와 생성될 instance의 Compartment 등을 명시적으로 선택한다. secondary VNIC나 Block Volume 관계를 지정하려는 경우 이 방식이 필요하다.

기존 instance의 software를 재사용하려면 Custom Image 역할을 함께 고려해야 한다. 설정 복사와 disk 내용 복사를 같은 기능으로 이해하면 새 member가 application 없이 시작될 수 있다.

## 배치와 변경의 경계

Configuration에 AD/Subnet이 있어도 Pool이 instance를 만들 때는 Pool의 AD/Subnet 설정이 우선한다. Configuration이 같은 OCID라고 모든 Pool 배치가 같아지는 것은 아니다. 개별 instance를 Configuration에서 만드는 경우와 Pool이 placement를 결정하는 경우를 구분해야 한다.

기존 Configuration의 launch 설정을 바꾸려면 원하는 설정의 새 Configuration을 사용한다. display name·tag 같은 관리 metadata 변경과 launch template 변경은 다르다. Pool이 새 template를 참조하는 것과 기존 member의 OS·software가 자동으로 교체되는 것도 별개다.

## 공유 설정의 선택과 한계

Linux Configuration에 넣은 SSH key는 그것으로 생성되는 instance들에 공통 적용되며 Configuration 생성 후 key를 변경할 수 없다. Linux instance의 SSH 접속에 사용할 Pool용 Configuration에는 생성 시 public SSH key를 포함해야 한다. 저장되는 것은 public key이며 client의 private key는 별도로 보호한다.

Pool은 image source가 boot volume인 Configuration으로 생성할 수 없다. Capacity reservation을 연결한 Configuration은 해당 reservation을 instance/Pool에 적용하지만 실제 launch에는 충분한 capacity가 필요하다. Template가 존재하는 것만으로 IAM·network 도달성·application 초기화 성공을 보장하지 않는다.

## 함께 읽기

- [Images](01_Images.md)
- [Instance Pools](03_Instance_Pools.md)
- [보존된 Hands-on 초안](../../Hands-on/Original-Provisional/07_Autoscaling/02_Instance_Configurations.md) — 실습 절차는 별도 자료에서 다룬다.

## 공식 참고 문서

- [Managing Instance Configurations](https://docs.oracle.com/en-us/iaas/Content/Compute/Concepts/instancemanagement.htm)
- [Creating an Instance Configuration](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/creatinginstanceconfig.htm)
- [Creating Instance Pools](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/creatinginstancepool.htm)
