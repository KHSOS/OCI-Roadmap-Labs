# Policies

OCI IAM Policy는 **어떤 주체에게 어떤 리소스 접근을 허용하는지** 표현하는 문서다. Policy 문서는 하나 이상의 Statement를 담으며, 각 Statement는 주체·작업 수준·리소스 종류·대상 범위와 선택 조건을 결합한다. Policy라는 말은 개별 Statement, 이름과 OCID가 있는 문서, 조직의 전체 권한 규칙을 가리킬 수 있으므로 문맥을 구분해야 한다.

## 허용 모델과 기본 거부

OCI의 이 Policy 언어는 접근을 허용하는 모델이다. 허용되지 않은 작업은 기본적으로 거부되며, 별도의 Deny Statement로 기존 허용을 덮는 방식이 아니다. 여러 Group과 여러 Policy에서 얻는 권한은 함께 고려된다. 따라서 읽기만 허용하는 Statement 하나를 추가해도 이미 다른 경로에서 허용된 쓰기 권한이 취소되는 것은 아니다.

## Statement를 읽는 관점

|부분|뜻|예시의 의미|
|---|---|---|
|Subject|권한을 받을 주체|특정 Group 또는 Dynamic Group|
|Verb|허용하는 작업의 수준|inspect, read, use, manage|
|Resource-Type|대상 리소스 종류|instances 또는 instance-family|
|Location|허용이 적용되는 논리적 범위|한 Compartment 또는 전체 Tenancy|
|Condition|추가로 충족해야 할 조건|지원되는 요청·대상 변수 조건|

Group 이름 대신 OCID로 정확한 주체를 식별할 수 있다. 하나의 Statement는 한 Verb와 한 Resource-Type을 지정하며, 여러 리소스 종류를 허용하려면 별도의 Statement들로 표현한다. Family는 함께 관리하는 여러 리소스 종류를 묶으므로 개별 종류보다 범위가 넓을 수 있다.

## Verb 선택과 실제 작업

일반적인 접근 수준은 inspect → read → use → manage 순서로 넓어진다. inspect는 주로 조회·목록, read는 이에 더해 내용 읽기, use는 기존 리소스 사용, manage는 해당 종류의 전체 권한을 포함한다. 이 요약만으로 개별 API 권한을 결정할 수는 없다. 서비스별 Verb와 Resource-Type 조합에 예외와 작업 의존성이 있기 때문이다.

예를 들어 Object Storage에서 inspect objects는 Object 목록과 HEAD를 허용하지만 Object 내용 다운로드는 read objects와 관계된다. 반면 Networking의 inspect는 보안 규칙과 Route Table 내용도 보여준다. 따라서 inspect를 모든 서비스에서 단순한 이름 목록 조회라고 해석하면 안 된다. Policy 수정처럼 새 허용을 만들 수 있는 작업은 use가 아니라 manage에 속한다.

## Compartment와의 관계

Location은 어떤 리소스에 접근을 허용하는지 나타낸다. Policy Attachment는 그 문서를 어느 Compartment에 보관하며 누가 수정·삭제할 수 있는지와 관계된다. 두 범위는 같은 질문이 아니다. 상위 Compartment의 적용 가능한 허용은 하위로 상속된다.

예를 들어 프로젝트 VM 관리 권한과 공통 네트워크 사용 권한을 서로 다른 Compartment에 대해 조합할 수 있다. Instance 생성처럼 여러 리소스가 관계되는 작업은 한 Resource-Type의 허용만으로 충분하지 않을 수 있다.

## 설계 기준과 한계

개별 Resource-Type과 좁은 Compartment는 권한 경계를 세밀하게 만들고, Family는 관련 작업을 관리하기 쉽게 한다. 어떤 선택이 최소권한인지는 실제 API 작업과 의존성에 따라 달라진다. 전체 Tenancy의 all-resources 관리 권한을 일반적인 문제 해결 기본값으로 삼지 않는다.

조건은 지원되는 변수와 해당 요청에 적용 가능한 값이어야 한다. 또한 Policy 적용에는 지연이 있을 수 있다. 화면 표시, 인증 성공, API 목록 성공은 원하는 데이터 작업까지 허용되었다는 뜻이 아니다.

## 함께 읽기

[Groups](02_Groups.md), [Policy Conditions](07_Policy_Conditions.md), [Policy Scope / Inheritance](08_Policy_Scope_Inheritance.md)를 함께 보면 주체·조건·범위의 관계를 이해할 수 있다. [보존된 Policies 실습](../../Hands-on/Original-Provisional/04_Identity_and_Access_Management_IAM/04_Policies.md)은 기존 Verb 비교 절차를 담은 OCI 실행 미검증 초안이다.

## 공식 참고 문서

- [How Policies Work — Verbs, Resource-Types, Inheritance and Attachment](https://docs.oracle.com/en-us/iaas/Content/Identity/Concepts/policies.htm)
- [Policy Syntax — Subject, Verb, Location and Conditions](https://docs.oracle.com/en-us/iaas/Content/Identity/Concepts/policysyntax.htm)
