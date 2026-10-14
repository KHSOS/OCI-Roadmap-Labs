# Dynamic Groups

Dynamic Group은 규칙에 맞는 OCI 리소스를 **API 요청의 주체인 Principal**로 묶는 단위다. 일반 Group이 User들의 집합이라면, Dynamic Group은 Compute Instance 같은 지원되는 리소스의 신원을 다룬다. 구성원은 사람이 수동으로 가입하는 방식이 아니라 Matching Rule에 부합하는지에 따라 결정된다.

## 워크로드 신원을 사용하는 이유

VM의 프로그램이 Object Storage를 읽어야 한다면 프로그램에도 인증과 권한이 필요하다. Instance Principal은 Compute Instance 자신의 신원으로 인증할 수 있게 해 사람의 API 키나 사용자 설정 파일을 VM에 배포할 필요를 줄인다. Instance의 인증서 생성과 교체는 서비스가 관리한다. 다만 인증된 Instance가 어느 Object를 읽을 수 있는지는 별도의 IAM Policy가 결정한다.

## Matching Rule과 Policy의 역할

|요소|담당하는 결정|
|---|---|
|Matching Rule|어떤 리소스가 Dynamic Group의 구성원인가?|
|Dynamic Group|공통 권한을 받을 리소스 주체들의 집합은 무엇인가?|
|Policy|그 집합에 어떤 리소스 작업과 범위를 허용하는가?|
|Instance Principal 인증|실제 API 호출이 Instance의 신원을 사용하는가?|

예를 들어 특정 Instance OCID를 기준으로 한 규칙은 한 VM만 포함하는 좁은 선택이다. Compartment 기준 규칙은 같은 관리 범위에 속한 여러 Instance를 포함할 수 있다. 후자는 새 Instance에도 권한을 적용하기 편하지만, 그 Compartment에서 누가 Instance를 만들 수 있는지까지 권한 경계의 일부가 된다.

규칙에 맞는 VM이라도 Policy가 원하는 작업을 허용하지 않으면 요청은 허용되지 않는다. 반대로 Dynamic Group과 Policy가 맞아도 애플리케이션이 다른 인증 방식을 사용한다면 기대한 Instance Principal 관계로 요청한 것이 아니다.

## 다른 개념과의 관계

Instance Principal은 Compute의 워크로드 신원 방식이다. 다른 서비스의 Resource Principal은 해당 서비스의 지원 방식과 리소스 규칙을 따르므로 Instance용 조건을 모든 서비스에 그대로 적용하지 않는다. 일반 Group에 VM을 User처럼 가입시키거나 사람이 쓰는 인증 정보를 복사하는 방식과 구분한다.

Principal 인증과 네트워크 통신도 별개다. IAM이 API 호출을 허용해도 목적 서비스에 도달할 경로와 통신 허용이 필요하다. Dynamic Group은 Route Table이나 보안 규칙을 대신하지 않는다.

## 선택 기준과 보안 경계

한정된 워크로드만 접근해야 하면 정확한 리소스 기준을, 동일한 책임 범위의 워크로드가 늘어나는 경우에는 그 관리 범위를 나타내는 규칙을 검토한다. 넓은 구성원 규칙과 넓은 Policy를 동시에 쓰면 의도보다 많은 워크로드에 권한이 적용될 수 있다.

특히 VM에 SSH로 접근할 수 있는 사용자는 그 Instance에 부여된 권한을 사용할 수 있다. Instance Principal이 사람의 키 배포를 줄인다고 해서 VM 내부 접근 통제가 불필요해지는 것은 아니다. Compute Instance Principal에는 철회할 수 없는 Compartment 조회 권한도 기본적으로 있으므로, 명시적으로 작성한 Policy만이 모든 권한의 전부라고 생각해서는 안 된다.

## 함께 읽기

[Groups](02_Groups.md)는 사람의 권한 집합을, [Policies](04_Policies.md)는 허용 작업과 범위를, [Resource Principals](09_Resource_Principals.md)는 다른 워크로드 신원 관계를 설명한다. [보존된 Dynamic Groups 실습](../../Hands-on/Original-Provisional/04_Identity_and_Access_Management_IAM/03_Dynamic_Groups.md)은 기존 Instance Principal 시험 절차를 담은 OCI 실행 미검증 초안이다.

## 공식 참고 문서

- [Calling Services from an Instance — Concepts and Security Considerations](https://docs.oracle.com/en-us/iaas/Content/Identity/Tasks/callingservicesfrominstances.htm)
- [Overview of IAM — Dynamic Groups](https://docs.oracle.com/en-us/iaas/Content/Identity/getstarted/identity-domains.htm)
