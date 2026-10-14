# Policy Conditions

Policy Condition은 IAM Statement의 허용 범위를 **요청이나 대상의 속성에 따라 더 세밀하게 제한하는 조건**이다. Group, Verb, Resource-Type, Compartment로 큰 접근 범위를 정한 뒤, 그 Statement가 어떤 경우에만 적용되는지를 표현한다.

## 조건이 필요한 상황

한 Group이 특정 서비스의 리소스를 읽어야 하지만 모든 대상이나 모든 작업까지 필요하지 않을 수 있다. 이때 대상 이름, 요청한 API 작업, 필요한 Permission, 승인된 네트워크 출처 또는 시간 같은 지원되는 변수를 검토한다. 조건은 새로운 인증 수단이 아니라 이미 정의된 허용 Statement의 범위를 좁히는 요소다.

## Request와 Target의 구분

|변수의 관점|의미|예시|
|---|---|---|
|request|요청 자체의 속성|request.operation은 요청한 API 작업을 나타낸다.|
|target|요청이 대상으로 삼는 리소스 속성|target.group.name은 대상 Group 이름과 관계된다.|
|request.permission|판단하는 원자적 권한|Verb가 포함하는 권한 중 일부만 허용할 때 관계된다.|

Condition을 여러 개 결합할 때 all은 모두 충족해야 하는 AND, any는 하나 이상 충족하면 되는 OR를 뜻한다. 예를 들어 특정 Permission과 특정 Operation이 모두 맞아야 한다는 설계는 all 관계이고, 허용한 여러 Operation 중 하나라는 설계는 any 관계다. 두 결합을 바꾸면 접근 범위가 달라진다.

## Permission과 Operation의 선택

Permission은 OCI가 정의한 원자적 권한이고 API Operation은 실제 작업이다. 같은 Permission이 여러 Operation을 지원할 수 있으며 한 Operation이 여러 Permission을 요구할 수도 있다. 따라서 특정 Permission을 허용하는 조건과 특정 Operation만 허용하는 조건은 항상 같은 범위가 아니다.

허용할 Permission들을 명시하면 이후 같은 Permission을 사용하는 새 API도 관계될 수 있다. 반대로 특정 Permission 하나만 제외하는 설계는 미래에 추가되는 다른 Permission을 자동으로 허용할 위험이 있다. 무엇을 계속 허용할지에 맞추어 서비스별 공식 권한 표와 조건을 함께 보아야 한다.

## 평가에서 중요한 한계

요청에 적용되지 않는 변수를 조건에 넣으면 그 조건은 false가 되어 해당 Statement가 요청을 허용하지 않는다. 예를 들어 Group 이름을 요구하는 조건이 있는 사용자 관리 Statement는 특정 Group을 대상으로 하지 않는 일반 사용자 목록 요청에 적용되지 않을 수 있다. 목록 조회 권한과 특정 대상 변경 권한은 같은 조건으로 표현할 수 있다고 가정하지 않는다.

조건의 문자열 매칭은 대소문자를 구분하지 않는다. Object Storage에서 BucketA와 bucketA처럼 대소문자만 다른 이름을 만들 수 있더라도, Bucket 이름 조건은 두 이름을 별개의 접근 대상으로 분리하지 못한다.

시간 조건은 UTC를 기준으로 한다. 지역 시각과 일광절약시간을 별도로 생각해야 하며 Oracle 문서는 요청 timestamp와 IAM 평가 사이의 차이에 대해 5분 여유를 고려하도록 설명한다. 시간 조건을 초 단위로 정확한 접근 종료 보장이라고 해석하지 않는다.

## 다른 Policy와의 관계

Condition은 그 Statement가 허용하는 범위를 정할 뿐이다. 다른 Statement나 Group, 상위 범위의 Policy가 같은 작업을 허용한다면 이 조건 하나가 전체 접근을 거부하지는 않는다. IAM 조건과 네트워크 보안 규칙도 다르다. 네트워크 출처를 조건으로 사용할 수 있지만 그것이 Route Table이나 통신 허용을 자동으로 구성하지는 않는다.

## 함께 읽기

[Policies](04_Policies.md)는 허용 모델을, [Policy Scope / Inheritance](08_Policy_Scope_Inheritance.md)는 여러 범위의 허용 관계를 설명한다. [보존된 Policy Conditions 실습](../../Hands-on/Original-Provisional/04_Identity_and_Access_Management_IAM/07_Policy_Conditions.md)은 기존 조건 비교 절차를 담은 OCI 실행 미검증 초안이다.

## 공식 참고 문서

- [Policy Syntax — Conditions, any/all and Case Insensitivity](https://docs.oracle.com/en-us/iaas/Content/Identity/Concepts/policysyntax.htm)
- [Advanced Policy Features — Variable Applicability, Permissions and Time Frames](https://docs.oracle.com/en-us/iaas/Content/Identity/Concepts/policyadvancedfeatures.htm)
- [How Policies Work — Allow Model](https://docs.oracle.com/en-us/iaas/Content/Identity/Concepts/policies.htm)
