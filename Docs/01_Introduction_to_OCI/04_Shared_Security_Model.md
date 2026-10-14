# Shared Security Model

OCI의 보안은 Oracle과 고객의 공동 책임이다. Oracle은 클라우드 infrastructure와 그 운영을 보호하고, 고객은 자신의 workload·데이터·사용자·논리적 설정을 보호한다. 서비스를 사용한다는 사실만으로 고객의 보안과 compliance 책임이 사라지지는 않는다. [Oracle: Shared Security Model](https://docs.oracle.com/en-us/iaas/Content/cloud-adoption-framework/security.htm)

## 누가 무엇을 보호하는가

| 영역 | Oracle의 책임 예 | 고객의 책임 예 |
|---|---|---|
| 물리 infrastructure | 데이터센터 물리 보안, provider hardware와 시설 | 데이터 위치·업무 요구에 맞는 사용 결정 |
| 자원 격리와 접근 | Compute·network·storage 격리, IAM framework | 사용자 credentials, 계정 접근, IAM policy |
| 네트워크 | 서비스를 운영하는 기반 network | VCN·route·firewall·security rule의 논리적 구성 |
| workload와 데이터 | Oracle 서비스의 기반 software·기능 운영 | 고객 platform·application·데이터 보호, 관련 patching |

이는 업무 분담의 예시다. 관리형 서비스와 직접 운영하는 OS에서는 고객이 제어하는 계층이 달라지므로 구체적인 서비스의 책임 범위를 함께 읽어야 한다. Oracle과 고객 모두 software와 관련 논리적 통제에 보안 책임을 가진다.

## 두 가지 일상적인 예

**VM의 OS와 접근 권한:** Oracle이 물리 서버를 보호해도 고객이 운영하는 application의 접근 권한이나 OS patching을 대신 설계해 주는 것은 아니다. IAM이 OCI API 작업을 허용하는 것과 SSH로 guest 사용자에 로그인하는 것도 서로 다른 인증·접근 경로다. [Compute](../02_Compute/00_Compute.md)와 [SSH Keys](../02_Compute/05_SSH_Keys.md)를 함께 보면 이 경계를 이해하기 쉽다.

**저장 데이터의 공개 여부:** 저장 service의 기반 격리가 제공되어도 고객은 데이터 접근 정책·credentials·공개 설정을 업무에 맞게 관리해야 한다. private 설정 하나가 조직 전체의 보안 준수를 증명하지는 않는다.

## 선택 기준과 한계

직접 관리하는 계층이 늘면 patching·구성·운영 책임도 구체화된다. 관리형 기능을 선택해도 계정 관리, 데이터 분류, 권한과 workload governance는 여전히 고객의 과제다. 보안 제품 하나를 켜는 방식보다 누가 어떤 위험과 설정을 책임지는지 분명히 하는 것이 중요하다. [Well-Architected Framework](05_Well_Architected_Framework.md)는 이를 신뢰성·성능·운영 요구와 함께 검토한다.

기존 private object와 제한접근 설정 사례는 [보존된 Hands-on](../../Hands-on/Original-Provisional/01_Introduction_to_OCI/04_Shared_Security_Model.md)에 있다. 설정을 관찰하는 활동과 patch 설치·침투시험·전체 compliance audit는 서로 다른 범위다.


## 공식문서

- [Shared Security Model](https://docs.oracle.com/en-us/iaas/Content/cloud-adoption-framework/security.htm): 책임표, 공동 논리적 통제, workload governance.
- [Overview of the Compute Service](https://docs.oracle.com/en-us/iaas/Content/Compute/Concepts/computeoverview.htm): IAM 통합과 instance 구성.
- [Managing Key Pairs on Linux Instances](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/managingkeypairs.htm): guest SSH 인증.
