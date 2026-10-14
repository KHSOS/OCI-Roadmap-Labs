# Introduction to OCI

Oracle Cloud Infrastructure (OCI)는 애플리케이션을 실행하는 Compute, 데이터를 보관하는 Storage, 자원 사이의 연결을 구성하는 Networking 등을 함께 제공하는 클라우드 서비스 집합이다. 서버 하나를 빌리는 것뿐 아니라 업무에 필요한 실행 환경·데이터·접근 방식을 조합하는 것이 OCI 설계의 출발점이다. [Oracle OCI 소개](https://docs.oracle.com/en-us/iaas/Content/GSG/Concepts/baremetalintro.htm)

## 업무를 서비스에 연결하기

예를 들어 웹 애플리케이션은 Compute instance에서 실행되고, 운영체제는 Boot Volume에, 별도의 업무 데이터는 Block Volume에 놓일 수 있다. 파일을 object로 저장하는 Object Storage는 특정 instance의 디스크와 다른 서비스다. VCN은 instance의 가상 네트워크를 구성하며, IAM은 누가 어떤 자원을 사용할 수 있는지를 결정한다. 이 예시는 서비스 역할을 연결한 것이며 모든 애플리케이션에 VM이나 공개 인터넷 접속이 필요하다는 의미는 아니다. [Compute의 구성 요소와 Storage 관계](https://docs.oracle.com/en-us/iaas/Content/Compute/Concepts/computeoverview.htm)

## 위치와 관리 범위는 다르다

[Region](01_Regions.md)은 지리적 위치, [Availability Domain](02_Availability_Domains.md)은 Region 안의 격리된 데이터센터 범위, [Fault Domain](03_Fault_Domains.md)은 AD 안의 hardware·maintenance 격리 그룹이다. Compartment는 자원 접근과 관리를 나누는 논리적 범위이지 물리적 위치가 아니다. 하나의 Compartment에 속한 자원도 여러 Region에 있을 수 있다. VCN과 bucket은 regional 자원이고 Compute instance와 연결 volume은 AD 범위를 갖는다. [자원별 위치 범위](https://docs.oracle.com/en-us/iaas/Content/General/Concepts/regions.htm)

## 서비스를 고를 때의 질문

- 직접 운영할 OS와 소프트웨어가 필요한가, 아니면 관리형 기능을 이용할 것인가?
- 사용자와 데이터는 어디에 있으며 어떤 지연시간·데이터 위치 요구가 있는가?
- CPU·메모리·저장 용량 중 무엇이 병목이고, 필요한 가용성은 어느 수준인가?
- 누가 접근하며 어떤 설정·패치·데이터 보호를 고객이 담당하는가?

Compute를 선택해도 애플리케이션 설치·운영이 자동 완성되는 것은 아니다. Region이나 FD를 나눠도 데이터 복제와 장애 전환 설계가 저절로 생기지 않는다. [Shared Security Model](04_Shared_Security_Model.md)과 [Well-Architected Framework](05_Well_Architected_Framework.md)는 이런 선택을 보안·신뢰성·비용·운영 관점에서 검토하는 기준이다.

## Hands-on 연결

개념을 읽는 데 자원 생성은 필요하지 않다. 작은 private bucket과 파일을 다루는 기존 자료는 [보존된 Hands-on](../../Hands-on/Original-Provisional/01_Introduction_to_OCI/00_Introduction_to_OCI.md)에 있다. bucket 사용은 OCI 전체 서비스의 가용성이나 보안을 검증하는 실험이 아니다.


## 공식문서

- [Welcome to Oracle Cloud Infrastructure](https://docs.oracle.com/en-us/iaas/Content/GSG/Concepts/baremetalintro.htm): OCI의 목적과 서비스 집합.
- [Overview of the Compute Service](https://docs.oracle.com/en-us/iaas/Content/Compute/Concepts/computeoverview.htm): instance·image·shape·network·storage 관계.
- [Regions and Availability Domains](https://docs.oracle.com/en-us/iaas/Content/General/Concepts/regions.htm): 위치와 resource availability.
