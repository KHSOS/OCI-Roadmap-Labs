# Resource Principals

Resource Principal은 지원되는 OCI 리소스가 **자신의 신원으로 다른 OCI 서비스에 요청하는 방식**이다. 사람이 로그인한 User를 대신 공유하는 모델이 아니라 실행 중인 워크로드의 신원을 인증하고 IAM Policy로 필요한 접근을 허용하는 관계다. 각 서비스의 지원 방식과 리소스 규칙을 따르므로 모든 리소스에 같은 설정이 적용되는 것은 아니다.

## Function이 데이터를 읽는 예

OCI Function이 Object Storage의 자료를 읽고 처리해야 한다고 생각해 보자. Function을 호출하는 사용자의 권한과 실행 중인 Function이 Bucket을 읽는 권한은 서로 다른 요청 관계다. 사용자가 Function을 호출할 수 있어도 Function의 데이터 접근이 자동으로 허용되지는 않는다.

Functions의 Resource Principal 모델은 Function을 Dynamic Group에 포함하고 그 Group에 대상 리소스 접근을 허용하는 Policy를 연결한다. 함수 코드는 Resource Principal Provider를 통해 해당 신원으로 인증한다. 이 관계를 사용하면 Function에 사람의 사용자 API 키를 별도로 배포하는 방식과 구분할 수 있다.

## 인증과 허용의 연결

|요소|역할|
|---|---|
|Function 리소스의 신원|어떤 워크로드가 요청했는지 식별한다.|
|Dynamic Group Matching Rule|권한을 받을 Function 집합을 정한다.|
|IAM Policy|대상 서비스의 어떤 작업과 범위를 허용하는지 정한다.|
|Resource Principal Provider|실행 환경의 신원 자료를 이용해 서비스 요청을 인증한다.|
|Resource Principal Session Token (RPST)|인증에 사용되는 세션 토큰으로 허용된 접근과 관계된다.|

Dynamic Group에 포함되는 것과 SDK가 실제 Resource Principal 신원으로 요청하는 것은 다른 조건이다. 인증할 수 있어도 필요한 Policy가 없으면 원하는 데이터 작업은 허용되지 않는다. 또한 Function 실행 요청의 성공과 내부에서 수행하는 데이터 API 요청의 성공은 별도로 해석해야 한다.

## Matching Rule 선택

정확한 Function OCID를 기준으로 하면 특정 Function 하나를 포함할 수 있다. Function 리소스 종류와 Compartment를 함께 기준으로 하면 해당 범위의 Function 집합을 포함할 수 있다. Defined Tag를 이용한 Function 집합도 공식 예시에 있으며 Free-form Tag는 그 조건에 지원되지 않는다.

좁은 리소스 기준은 접근 대상을 명확하게 하고, 범위·태그 기준은 워크로드가 늘어날 때 관리하기 편하다. 대신 해당 Compartment에 새 Function을 만들거나 접근 관련 태그를 관리할 수 있는 주체가 누구인지도 함께 고려해야 한다. Application OCID와 Function OCID는 서로 다른 객체를 가리키므로 같은 대상으로 취급하지 않는다.

## Instance Principal과의 관계

Compute의 Instance Principal도 워크로드 신원을 사용하지만 Functions의 리소스 종류와 Matching Rule을 그대로 공유하는 것은 아니다. VM용 instance 조건과 Function용 resource 조건을 구분한다. 서비스별 인증 Provider, 실행 환경과 대상 API 지원을 확인하는 것이 선택 기준이다.

Oracle은 SDK에 포함된 Resource Principal Provider 사용을 권장한다. 자체 Provider는 실행 환경의 토큰·키와 요청 서명을 직접 다루므로 인증 자료를 안전하게 처리해야 한다. 토큰과 private key는 일반 설정값이나 학습 결과로 공개할 자료가 아니다.

## 시간과 통신의 한계

Functions 공식 문서는 토큰이 15분간 캐시되므로 Policy나 Dynamic Group을 변경했을 때 효과를 보려면 이 캐시를 고려해야 한다고 설명한다. 이 시간이 전체 장애 복구나 정책 전파의 보장 시간이라는 뜻은 아니다.

Resource Principal은 네트워크 경로나 목적 서비스의 통신 허용을 만들지 않는다. Function 배포·호출, 실행 환경의 네트워크 접근, 대상 데이터 권한은 각각 관계되는 조건이다. IAM을 넓게 허용하는 것으로 연결 실패나 잘못된 대상 이름을 해결할 수는 없다.

## 함께 읽기

[Dynamic Groups](03_Dynamic_Groups.md)는 워크로드 집합을, [Policies](04_Policies.md)는 데이터 접근 허용을, [Policy Conditions](07_Policy_Conditions.md)는 그 범위의 세부 제한을 설명한다. [보존된 Resource Principals 실습](../../Hands-on/Original-Provisional/04_Identity_and_Access_Management_IAM/09_Resource_Principals.md)은 Function 배포·호출의 기존 절차를 담은 OCI 실행 미검증 초안이다.

## 공식 참고 문서

- [Accessing Other OCI Resources from Running Functions — Dynamic Groups, RPST and Provider](https://docs.oracle.com/en-us/iaas/Content/Functions/Tasks/functionsaccessingociresources.htm)
- [Calling Services from an Instance — Instance Principals](https://docs.oracle.com/en-us/iaas/Content/Identity/Tasks/callingservicesfrominstances.htm)
