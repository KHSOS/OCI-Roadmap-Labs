# Compartments

Compartment는 관련 OCI 리소스를 묶는 **논리적 관리 범위**다. 프로젝트·부서·환경별로 리소스를 정리하고, 접근 Policy와 사용량·비용을 구분하는 기준으로 쓴다. Region, Availability Domain, VCN, Subnet처럼 실제 배치나 네트워크 위치를 나타내는 단위는 아니다.

## 관리 경계가 필요한 이유

개발팀과 운영팀이 같은 Tenancy를 사용하더라도 각자 관리해야 하는 리소스와 책임은 다를 수 있다. Compartment는 이런 소유·운영 관계를 표현하는 기반이다. 예를 들어 개발 프로젝트 리소스와 공통 네트워크를 별도로 분류하면 프로젝트 담당자에게 VM 관리 권한을, 네트워크 담당자에게 네트워크 관리 권한을 다르게 줄 수 있다.

Compartment 생성 자체가 접근을 완전히 차단하는 것은 아니다. 실제 접근은 적용되는 IAM Policy가 결정하며 상위 범위의 허용도 함께 고려된다. 관리 분류와 권한 설정을 함께 설계해야 격리 의도가 구현된다.

## Tenancy와 계층 관계

Tenancy는 모든 리소스를 포함하는 루트 Compartment다. 그 아래에 Compartment를 두고 다시 하위 Compartment로 책임 범위를 나눌 수 있다. 부모와 자식은 계층 관계이며 같은 부모 아래의 두 자식은 서로 동등한 위치의 형제 관계다.

예를 들어 Projects 아래의 Development와 Production은 형제 Compartment다. Development에만 적용되는 허용과 Projects 전체에 적용되는 허용은 범위가 다르다. Projects에 적용되는 Policy는 그 하위의 Development와 Production에도 관계되므로 부모 수준의 광범위한 권한을 하위 Policy 하나로 자동 축소할 수 없다.

## Region과 네트워크의 구분

|관점|답하는 질문|
|---|---|
|Compartment|이 리소스는 어느 관리·권한 범위에 속하는가?|
|Region|이 리소스의 서비스는 어느 지리적 Region에서 제공되는가?|
|VCN와 Subnet|어느 네트워크와 주소 범위에 연결되는가?|
|IAM Policy|누가 어떤 리소스 작업을 할 수 있는가?|

같은 VCN을 사용하는 VM이 서로 다른 Compartment에 속할 수 있다. VM의 Compartment가 네트워크의 Compartment와 다르더라도 필요한 네트워크 사용 권한이 있으면 그 관계를 구성할 수 있다. 반대로 같은 Compartment에 있다는 이유만으로 두 VM 사이의 통신이 허용되지는 않는다.

Compartment는 Region별로 새로 만드는 네트워크가 아니다. Console에서 지역 리소스를 볼 때에는 Compartment 선택과 현재 Region 선택을 함께 생각해야 한다. 어떤 목록이 비어 있다는 사실만으로 다른 Region을 포함한 전체 리소스가 없다고 판단할 수 없다.

## 분류 기준과 한계

프로젝트별로 권한과 비용을 구분할지, 개발·운영 환경별로 책임을 나눌지, 공통 네트워크처럼 여러 팀이 사용하는 영역을 따로 관리할지가 분류 기준이다. 이름보다 실제 부모 관계와 Policy 범위가 중요하다. 같은 이름을 다시 사용해도 리소스 식별자는 별도의 객체를 나타낸다.

Compartment에 보관하는 Policy 문서의 위치와 Statement가 허용하는 대상 범위는 구분된다. 문서의 Attachment 위치는 그 Policy를 누가 변경할 수 있는지와 관계된다. 관리 경계를 넓게 허용하면 하위 범위에도 영향이 있으므로 처음부터 책임과 허용 범위를 같이 검토한다.

## 함께 읽기

[Policies](04_Policies.md)와 [Policy Scope / Inheritance](08_Policy_Scope_Inheritance.md)는 관리 계층에 허용이 적용되는 방식을 설명한다. [OCID](06_OCID.md)는 이름과 별개인 리소스 식별을 다룬다. [보존된 Compartments 실습](../../Hands-on/Original-Provisional/04_Identity_and_Access_Management_IAM/05_Compartments.md)은 기존 자식 Compartment와 자료 생성 절차를 담은 OCI 실행 미검증 초안이다.

## 공식 참고 문서

- [Overview of IAM — Compartment and Tenancy](https://docs.oracle.com/en-us/iaas/Content/Identity/getstarted/identity-domains.htm)
- [Overview of Identity and Access Management — Network Topology and Compartment Access](https://docs.oracle.com/en-us/iaas/Content/Identity/Concepts/overview.htm)
- [How Policies Work — Policy Inheritance and Policy Attachment](https://docs.oracle.com/en-us/iaas/Content/Identity/Concepts/policies.htm)
