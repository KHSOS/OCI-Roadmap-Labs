# Users

OCI IAM의 User는 OCI 리소스를 관리하거나 사용하는 **개별 사람 또는 시스템의 신원**이다. 요청의 주체를 식별하고 그 주체에 맞는 인증 정보를 관리한다. 애플리케이션을 이용하는 모든 고객이 OCI 인프라를 관리하는 IAM User여야 하는 것은 아니다.

## 인증과 권한의 구분

Authentication은 요청자가 누구인지 확인하는 과정이다. Authorization은 확인된 주체에게 특정 작업이 허용되는지 판단하는 과정이다. 사용자 이름과 비밀번호로 Console에 로그인할 수 있어도 Bucket의 Object를 읽을 권한은 없을 수 있다. 반대로 잘못된 인증 정보로 로그인하지 못한 경우는 Object 읽기 Policy의 문제와 다르다.

User는 하나 이상의 인증 정보를 사용할 수 있다. Console 로그인과 API 요청의 인증 방식은 같지 않으므로, 비밀번호·API 서명 키 등은 용도에 맞는 별개의 인증 정보로 이해해야 한다. MFA는 사용자의 신원을 확인하는 보호 수단이며 리소스 접근 Policy를 대신하지 않는다.

## Identity Domain과 Group의 관계

Identity Domain은 사용자 집합과 인증·보안 설정을 담는 컨테이너다. User는 그 Domain의 Group에 속할 수 있으며, Group에 허용된 OCI 리소스 권한을 멤버십을 통해 사용한다. 사용자 관리에 쓰는 Domain 관리자 역할과 Compute 또는 Object Storage를 사용하는 권한은 구분된다.

예를 들어 같은 Domain의 두 사용자가 모두 로그인에 성공해도, 한 명만 읽기 권한이 있는 Group에 속하면 특정 Bucket의 Object 읽기 결과가 다를 수 있다. 이 차이는 계정 활성화 여부만으로 설명할 수 없고 Group membership과 적용되는 Policy를 함께 보아야 한다.

## 어떤 주체를 선택할까

|상황|주체를 바라보는 기준|
|---|---|
|사람이 Console로 클라우드를 운영|개인별 User와 필요한 Group 권한이 책임을 구분한다.|
|외부 Identity Provider를 사용하는 조직|Federation으로 인증하고 OCI에서 리소스 권한을 관리하는 관계를 구분한다.|
|지원되는 OCI 워크로드가 API를 호출|사람의 키를 공유하기보다 Resource Principal과 Dynamic Group 같은 워크로드 신원 방식을 검토한다.|
|일반 고객이 업무 애플리케이션을 사용|애플리케이션 로그인과 OCI 인프라 관리 User의 범위를 분리한다.|

## 경계와 주의점

사용자 이름, 이메일, User OCID, 인증 정보는 같은 값이 아니다. 이름이 비슷하다고 같은 주체이거나 같은 권한인 것은 아니다. 또한 Group 하나의 멤버십을 제거해도 다른 Group이나 상위 범위의 Policy가 접근을 허용하면 전체 접근이 사라지는 것은 아니다. 계정 관리, 인증 정보 보호, MFA, 최소권한은 서로 보완하는 요소다.

Identity Domains를 사용하는 환경과 이전 IAM 환경은 사용자 관리 API와 문서 경로가 다를 수 있다. 이전 IAM 개요의 관리 방식이나 이름 규칙을 현재 Domain 사용자 관리에 그대로 적용하지 않는다.

## 함께 읽기

[Identity and Access Management (IAM)](00_Identity_and_Access_Management_IAM.md)은 전체 관계를, [Groups](02_Groups.md)는 사용자 권한을 공유하는 단위를, [OCID](06_OCID.md)는 리소스 식별자를 설명한다. [보존된 Users 실습](../../Hands-on/Original-Provisional/04_Identity_and_Access_Management_IAM/01_Users.md)은 계정·멤버십 시험의 기존 절차를 담은 OCI 실행 미검증 초안이다.

## 공식 참고 문서

- [Overview of IAM — User, Identity Domain, MFA, Federation, Administrator Roles](https://docs.oracle.com/en-us/iaas/Content/Identity/getstarted/identity-domains.htm)
- [Overview of Identity and Access Management — User and Group](https://docs.oracle.com/en-us/iaas/Content/Identity/Concepts/overview.htm)
- [User Credentials — Console Password and API Signing Keys](https://docs.oracle.com/en-us/iaas/Content/Identity/Concepts/usercredentials.htm)
- [Calling Services from an Instance — Instance Principal](https://docs.oracle.com/en-us/iaas/Content/Identity/Tasks/callingservicesfrominstances.htm)
