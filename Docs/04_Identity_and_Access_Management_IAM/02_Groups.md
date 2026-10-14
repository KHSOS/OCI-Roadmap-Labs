# Groups

Group은 비슷한 접근 권한을 필요로 하는 **User들의 집합**이다. 같은 업무 역할을 가진 사용자에게 공통 Policy를 적용하는 단위로 사용한다. 사람을 묶는 Group과 OCI 리소스를 규칙으로 묶는 Dynamic Group은 구성원의 종류가 다르다.

## Group을 사용하는 이유

프로젝트의 읽기 담당자가 여러 명이면 개인별로 접근 관계를 반복해서 관리하기보다 역할을 나타내는 Group에 사용자를 묶을 수 있다. 인원이 바뀌어도 Policy가 표현하는 업무 권한과 실제 멤버십을 나누어 관리할 수 있다는 점이 중요하다.

Group의 존재 자체는 리소스 접근 허용이 아니다. 해당 Group을 주체로 하는 Policy가 작업과 대상 범위를 허용해야 하며, 요청한 User가 그 Group에 속해야 한다. Group 이름은 역할을 설명할 수 있지만 이름만으로 Reader나 Administrator 권한이 생기지 않는다.

## User와 Policy 사이의 관계

|관계|의미|
|---|---|
|User → Group membership|요청한 사용자가 어느 권한 집합에 참여하는가?|
|Policy → Group subject|그룹에 어떤 작업과 리소스 범위를 허용하는가?|
|Policy → Compartment scope|허용된 접근이 어느 논리적 관리 범위에 적용되는가?|

예를 들어 프로젝트 자료를 읽는 Group과 자료를 변경하는 Group을 분리할 수 있다. 같은 User가 두 Group에 속하면 두 관계에서 허용되는 권한을 함께 고려한다. 읽기 Group에서 빠졌더라도 다른 Group이 같은 자료의 읽기를 허용한다면 접근은 계속 가능할 수 있다.

## Group과 Dynamic Group의 선택

사람이나 일반 IAM User의 업무 역할을 묶을 때는 Group을 사용한다. Compute Instance 같은 지원되는 OCI 리소스가 자체 신원으로 API를 호출할 때는 Dynamic Group과 해당 서비스의 principal 방식이 관계된다. Dynamic Group에 사용자 계정을 넣는 방식으로 두 모델을 혼합하지 않는다.

Identity Domain 안의 Group을 다룰 때는 어느 Domain의 어떤 Group인지 함께 식별해야 한다. 비슷한 이름의 Group이 있다고 같은 주체로 취급해서는 안 된다. OCID는 정확한 객체 식별에 쓰이며, Group을 삭제하고 같은 이름으로 새로 만든 객체가 이전 객체와 동일한 것은 아니다.

## 설계 기준과 한계

- 팀·업무 역할처럼 공통 권한이 지속되는 기준으로 묶고 실제 작업에 필요한 Policy 범위를 별도로 생각한다.
- 넓은 Administrators 멤버십은 편리한 기본 역할이 아니라 광범위한 관리 권한이다. 제한된 읽기 역할과 같은 것으로 취급하지 않는다.
- 멤버십 변경과 Policy 변경은 서로 다른 변화다. 적용 전파에 시간이 걸릴 수 있으므로 즉시 반영된다고 가정하지 않는다.
- Identity Domain의 Group 관리와 리소스 접근 허용은 다르다. 사용자 목록을 관리할 수 있다는 사실이 모든 서비스 데이터를 읽을 수 있다는 뜻은 아니다.

## 함께 읽기

[Users](01_Users.md), [Policies](04_Policies.md), [Dynamic Groups](03_Dynamic_Groups.md)를 함께 보면 구성원·권한·워크로드 신원의 차이가 드러난다. [보존된 Groups 실습](../../Hands-on/Original-Provisional/04_Identity_and_Access_Management_IAM/02_Groups.md)은 멤버십과 Policy 조합을 비교하는 기존 절차이며 OCI 실행 검증은 완료되지 않았다.

## 공식 참고 문서

- [Overview of IAM — Group, Dynamic Groups, Security Policy](https://docs.oracle.com/en-us/iaas/Content/Identity/getstarted/identity-domains.htm)
- [Overview of Identity and Access Management — Groups and Example Scenario](https://docs.oracle.com/en-us/iaas/Content/Identity/Concepts/overview.htm)
