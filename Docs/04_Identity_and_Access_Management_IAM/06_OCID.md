# OCID

OCID(Oracle Cloud Identifier)는 OCI가 대부분의 리소스에 부여하는 고유 식별자다. 사람이 붙이는 이름과 달리 Console과 API에서 같은 리소스를 지정하는 기준이며, IAM 정책의 대상이나 API 요청의 입력으로 사용된다. OCID를 안다는 사실은 해당 리소스를 읽거나 변경할 권한이 있다는 뜻이 아니다. [Oracle Resource Identifiers](https://docs.oracle.com/en-us/iaas/Content/General/Concepts/identifiers.htm)

## 이름이 아닌 식별자를 사용하는 이유

여러 Compute 인스턴스가 같은 display name을 가질 수 있고 display name은 변경할 수 있다. 따라서 자동화, 장애 조사, 리소스 간 연결에는 화면에 보이는 이름만으로 대상을 정하기보다 OCID를 기준으로 삼는다. IAM 리소스에는 별도의 이름 제약도 있으므로 모든 서비스의 이름이 중복 가능하거나 변경 가능하다고 일반화하면 안 된다. 설명(description)은 식별자를 대신하는 값이 아니라 사람이 용도를 이해하도록 돕는 정보다. [이름과 Display Name의 구분](https://docs.oracle.com/en-us/iaas/Content/General/Concepts/identifiers.htm)

## 문자열에 담긴 정보와 해석의 한계

공식 구문은 `ocid1.<RESOURCE TYPE>.<REALM>.[REGION][.FUTURE USE].<UNIQUE ID>`다. `ocid1`은 형식 버전, resource type은 instance·volume·vcn·user 같은 종류, realm은 리전들이 속한 격리된 집합을 나타낸다. Region 부분은 리전 또는 특정 Availability Domain에 속하는 리소스에 나타나며, 적용되지 않는 리소스에서는 비어 있을 수 있다. Unique ID의 형식도 서비스마다 다를 수 있다.

이 구문은 복사한 값의 종류를 이해하는 단서이지 새로운 OCID를 만드는 공식이 아니다. tenancy 식별자에 Region이 비었다고 잘못된 값으로 판정하거나, 짧은 과거 Region 코드만 허용하는 검증기를 만들면 정상 식별자를 거부할 수 있다. 실제 요청에는 받은 전체 문자열을 보존해야 한다. [공식 구문과 예시](https://docs.oracle.com/en-us/iaas/Content/General/Concepts/identifiers.htm)

## IAM과의 관계

Tenancy OCID는 OCI API 요청 서명과 일부 IAM 작업에 사용되는 계정 범위의 식별자다. Compartment OCID는 리소스의 관리 범위를, 리소스 OCID는 작업 대상을 나타낸다. 같은 형태의 문자열처럼 보여도 이 역할은 서로 대체할 수 없다. 정책에서 이름을 사용할 수 있는 경우에도 주체와 대상의 의미를 먼저 구분해야 한다.

OCID는 인증 비밀키를 대체하지 않는다. 또한 요청 실패만으로 리소스가 없다고 결론 내리면 안 된다. 정확한 대상 식별, 요청 주체의 권한, 해당 서비스의 endpoint 범위는 별도의 판단 대상이다. 이 페이지는 식별자 개념을 설명하며 권한 정책이나 API 호출 절차는 아래 보존 실습으로 분리한다.

## 사용 사례와 주의점

- 운영 기록에서 이름이 비슷한 리소스를 구별하고 API 응답과 Console 대상을 대응시킨다.
- 자동화에 리소스 참조를 전달할 때 type이 다른 OCID를 혼용하지 않는다.
- 화면 캡처나 문서에 식별자를 담을 때 실제 고객·tenancy 정보를 불필요하게 공개하지 않는다. 이름·설명·tag에도 비밀정보를 넣지 말라는 공식 주의사항이 적용된다.

관련 개념은 [Compartments](05_Compartments.md), [Policies](04_Policies.md), [Resource Principals](09_Resource_Principals.md)다. OCID는 대상 식별, Policy는 허용 범위, Resource Principal은 요청 주체를 다룬다.

## Oracle 공식 참고자료

- [Resource Identifiers — OCIDs, Name and Description, Display Name](https://docs.oracle.com/en-us/iaas/Content/General/Concepts/identifiers.htm)

## 보존 실습으로 이동

[기존 전체 실습 초안](../../Hands-on/Original-Provisional/04_Identity_and_Access_Management_IAM/06_OCID.md)에서 식별자를 활용하는 기존 실습 절차를 읽을 수 있다. 보존된 초안으로, OCI 실행 검증은 완료되지 않았다.

