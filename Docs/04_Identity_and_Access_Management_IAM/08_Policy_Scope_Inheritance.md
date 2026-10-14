# Policy Scope / Inheritance

Policy Scope는 **허용이 적용되는 리소스 범위**이고, Inheritance는 상위 Compartment에서 적용되는 Policy가 하위 범위에도 이어지는 관계다. Policy 문서를 어디에 보관하는지, Statement가 어느 리소스에 접근을 허용하는지, 누가 그 문서를 변경할 수 있는지는 서로 구분해야 한다.

## Location과 Attachment의 차이

|관점|결정하는 내용|
|---|---|
|Statement의 Location|접근을 허용하는 대상 Compartment 또는 Tenancy|
|Policy Attachment|Policy 문서가 연결되어 있는 Compartment 또는 루트 Tenancy|
|Policy 관리 권한|그 Attachment 범위에서 문서를 수정하거나 삭제할 수 있는 주체|

Tenancy에 연결된 Policy를 자식 Compartment의 관리자라는 이유만으로 변경할 수 있는 것은 아니다. 반면 해당 자식 범위의 Policy 관리 권한을 가진 관리자는 그 범위에 연결된 문서를 관리할 수 있다. 문서를 어디에서 관리할지와 권한을 어느 대상에 부여할지는 별개의 설계 결정이다.

## 상속을 이해하는 예

Projects 아래에 Development와 Production이 있는 계층을 생각해 보자. 한 Group에 Projects의 관련 리소스 사용을 허용하면 해당 허용은 하위 Development와 Production에도 적용된다. Development만 대상으로 한 허용은 부모 Projects나 형제 Production에 대한 허용으로 올라가거나 옆으로 확장되지 않는다. Development 아래에 다시 자식이 있다면 그 하위 범위도 함께 고려해야 한다.

상속은 넓은 부모 권한을 더 좁은 자식 권한으로 덮는 방식이 아니다. OCI의 이 Policy 언어는 Allow 모델이므로 자식에 제한적인 Statement를 추가해도 부모에서 허용한 접근이 자동으로 취소되지 않는다. 실제 사용자의 접근은 다른 Group과 다른 Policy의 허용까지 함께 보아야 한다.

## 위임과 중앙 관리의 선택

정책 문서 변경을 중앙 관리자가 통제해야 하면 상위 Attachment를, 프로젝트 관리자가 자기 범위의 정책을 관리해야 하면 그 범위의 Attachment와 관리 권한을 검토한다. 어느 경우에도 문서가 존재하는 위치만 보고 전체 허용 대상을 추정하지 않고 Statement의 Location과 Condition을 함께 읽어야 한다.

범위를 이름으로 표현할 때 상위에서 멀리 떨어진 하위 Compartment는 계층 경로를 콜론으로 구분한다. OCID로 대상을 식별하는 방법도 있다. 표시 이름 하나만으로 계층에서 어느 대상을 가리키는지 모호해질 수 있으므로 정확한 객체와 부모 관계가 중요하다.

## 적용 범위와 한계

Compartment 계층은 논리적인 관리 범위이지 네트워크의 경로가 아니다. 개발 VM이 공통 네트워크의 Subnet을 사용하면 VM 관리 권한과 공통 네트워크 사용 권한이 서로 다른 Compartment에 관계될 수 있다. API 작업이 여러 리소스 권한을 요구하는 경우 이 허용들이 서로 다른 Statement에서 충족될 수도 있다.

정책 변경이나 삭제의 효과가 즉시 모든 요청에 나타난다고 가정할 수 없다. 한 요청의 접근 결과는 대상 존재 여부, 인증, 실제 적용되는 허용과 조건까지 관계되므로 형제 범위에서 실패했다는 결과 하나만으로 상속 모델 전체를 판단하지 않는다.

## 함께 읽기

[Compartments](05_Compartments.md)는 관리 계층을, [Policies](04_Policies.md)는 Statement와 허용 모델을, [Policy Conditions](07_Policy_Conditions.md)는 추가 범위 제한을 설명한다. [보존된 Scope / Inheritance 실습](../../Hands-on/Original-Provisional/04_Identity_and_Access_Management_IAM/08_Policy_Scope_Inheritance.md)은 부모·자식·형제 비교의 기존 절차를 담은 OCI 실행 미검증 초안이다.

## 공식 참고 문서

- [How Policies Work — Policy Inheritance, Attachment and Compartment Hierarchies](https://docs.oracle.com/en-us/iaas/Content/Identity/Concepts/policies.htm)
- [Policy Syntax — Location and Compartment Paths](https://docs.oracle.com/en-us/iaas/Content/Identity/Concepts/policysyntax.htm)
- [Overview of IAM — Compartment and Security Policy](https://docs.oracle.com/en-us/iaas/Content/Identity/getstarted/identity-domains.htm)
