# Identity and Access Management (IAM)

OCI Identity and Access Management (IAM)는 **누가 어떤 리소스에 어떤 작업을 할 수 있는지** 결정하는 서비스다. 로그인한 사람의 신원을 확인하는 Authentication과 그 사람이 리소스를 읽거나 변경할 수 있는지 판단하는 Authorization은 서로 다른 문제다. 로그인 성공만으로 모든 Compartment나 Bucket에 접근할 수 있는 것은 아니다.

## IAM이 필요한 이유

개발자에게 개발 환경을 운영할 권한은 주면서 다른 팀의 운영 환경은 보호해야 할 때 IAM이 경계를 만든다. 권한을 개인마다 반복해서 지정하기보다 역할이 비슷한 사람을 Group으로 묶고, Policy로 작업 종류와 대상 범위를 표현한다. 자동화 프로그램도 주체가 필요하지만, 사람의 계정과 장기 키를 그대로 공유하는 대신 지원되는 Resource Principal 같은 방식으로 리소스의 신원을 사용할 수 있다.

## 구성 요소와 관계

|구성 요소|담당하는 질문|다른 요소와의 관계|
|---|---|---|
|User|요청한 사람 또는 시스템은 누구인가?|Identity Domain에서 관리하며 Group에 속할 수 있다.|
|Group|같은 권한을 필요로 하는 사람들은 누구인가?|Policy의 주체가 되며 구성원에게 해당 권한이 적용된다.|
|Dynamic Group|어떤 OCI 리소스를 요청 주체로 취급하는가?|규칙에 맞는 리소스를 묶고 별도 Policy로 접근을 허용한다.|
|Compartment|권한과 운영 관리 대상은 어느 논리적 범위인가?|관련 리소스를 조직하고 Policy의 대상 범위가 된다.|
|Policy|주체에게 어떤 접근을 허용하는가?|주체·작업·리소스 종류·범위·선택 조건을 결합한다.|
|Identity Domain|사용자와 인증 설정은 어디서 관리하는가?|User, Group, 관리자 역할, SSO와 MFA 같은 설정을 포함한다.|

Tenancy는 전체 리소스를 포함하는 루트 범위다. Compartment는 논리적 관리 경계이지 VCN이나 Subnet 같은 네트워크 위치가 아니다. 하나의 VCN을 사용하는 두 VM이 서로 다른 Compartment에 속할 수 있고, 네트워크 관리 권한과 VM 관리 권한은 다르게 부여할 수 있다.

## 접근 범위를 이해하는 예

한 조직의 NetworkAdmins는 공통 네트워크를 관리하고, 개발팀은 자기 프로젝트의 VM을 관리한다고 생각해 보자. 개발팀에 공통 네트워크의 사용 권한을 주는 것과 네트워크 자체를 변경할 권한을 주는 것은 다르다. 마찬가지로 Object Storage Bucket 목록을 보는 권한과 Object 내용을 읽는 권한도 다른 작업이다. 어느 화면이 보인다는 사실만으로 모든 데이터 접근이 허용된 것은 아니다.

## 선택 기준과 한계

- 사람의 공통 업무 역할은 Group과 좁은 범위의 Policy로 표현하고, 지원되는 OCI 워크로드의 신원은 Dynamic Group과 Resource Principal 관계를 검토한다.
- Compartment는 프로젝트·환경·부서처럼 관리 책임과 접근 범위가 갈라지는 기준으로 나눈다. 실제 네트워크 통신 경계는 별도의 네트워크 설정으로 결정된다.
- Identity Domain 관리자 역할과 OCI 리소스 접근 Policy는 같은 권한이 아니다. 특정 Domain의 관리자라는 이유만으로 다른 OCI 서비스에 대한 전역 관리자 권한이 생기지는 않는다.
- 상위 범위에서 허용된 권한도 함께 고려해야 한다. 더 좁은 Policy 하나를 추가하는 것이 기존의 넓은 권한을 자동으로 취소하지는 않는다.
- Policy 변경은 즉시 모든 요청에 반영된다고 가정할 수 없다. 인증 실패, 존재하지 않는 대상, 네트워크 오류도 권한 거부와 구분해야 한다.

## 함께 읽기

[Users](01_Users.md), [Groups](02_Groups.md), [Dynamic Groups](03_Dynamic_Groups.md), [Policies](04_Policies.md), [Compartments](05_Compartments.md)는 위 관계를 각각 자세히 설명한다. [보존된 IAM 실습](../../Hands-on/Original-Provisional/04_Identity_and_Access_Management_IAM/00_Identity_and_Access_Management_IAM.md)은 기존 절차를 담은 별도 초안이며 OCI 실행 검증은 완료되지 않았다.

## 공식 참고 문서

- [Overview of IAM — Identity Domains, Components, Administrator Roles](https://docs.oracle.com/en-us/iaas/Content/Identity/getstarted/identity-domains.htm)
- [Overview of Identity and Access Management — Example Scenario, Compartments and Policies](https://docs.oracle.com/en-us/iaas/Content/Identity/Concepts/overview.htm)
- [Calling Services from an Instance — Workload Identity](https://docs.oracle.com/en-us/iaas/Content/Identity/Tasks/callingservicesfrominstances.htm)
- [How Policies Work — Allow Model and Inheritance](https://docs.oracle.com/en-us/iaas/Content/Identity/Concepts/policies.htm)
