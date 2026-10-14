# Regions

Region은 OCI 서비스를 제공하는 하나의 지리적 영역이다. 각 Region은 하나 이상의 Availability Domain으로 구성되며 다른 Region과 독립적이다. Region 선택은 화면의 표시 언어가 아니라 자원과 데이터가 위치하는 범위에 관한 선택이다. [Oracle: Regions and Availability Domains](https://docs.oracle.com/en-us/iaas/Content/General/Concepts/regions.htm)

## Region을 나누는 이유

사용자가 주로 접속하는 곳과 가까운 Region은 먼 Region보다 지연시간 측면에서 유리할 수 있다. 법적 관할·데이터 위치 요구, 업무 기준, Region 전체에 영향을 미치는 사건에 대한 복구 전략도 위치 선택에 영향을 준다. 예를 들어 한국 사용자를 위한 서비스는 한국 Region을 후보로 비교할 수 있지만, 가까운 곳이라는 이유만으로 모든 서비스·shape의 가용성이나 용량이 보장되지는 않는다.

## Region과 다른 경계의 관계

| 경계 | 의미 | Region과의 관계 |
|---|---|---|
| Availability Domain | 격리된 하나 이상의 데이터센터 | Region 내부에 있으며 개수는 Region별로 다름 |
| Fault Domain | hardware와 maintenance의 격리 그룹 | 각 AD 내부의 세 그룹 |
| Compartment | 자원의 논리적 관리·접근 범위 | 하나의 Compartment가 여러 Region의 자원을 포함할 수 있음 |
| VCN | 가상 네트워크 | 하나의 Region에 속하는 자원 |
| Object Storage bucket | object 보관 범위 | regional 자원이지만 올바른 Region endpoint를 통해 다른 위치에서도 접근 가능 |

다른 Region에서 같은 Compartment를 볼 수 있다는 사실은 VCN이나 instance가 자동 복제된다는 뜻이 아니다. [Availability Domains](02_Availability_Domains.md)는 Region 내부의 장애 격리를, [Fault Domains](03_Fault_Domains.md)는 AD 내부의 배치 분리를 설명한다.

## region-selection

위치를 고를 때는 사용자 지연시간, 데이터 위치 요구, 필요한 서비스·shape 지원, tenancy의 구독 범위를 함께 고려한다. Region의 표시 이름과 API에서 쓰는 identifier는 다르다. 예를 들어 South Korea Central (Seoul)의 identifier는 `ap-seoul-1`이다. tenancy는 하나의 realm에 속하며 다른 realm의 Region을 사용할 수 없다. Region이 존재한다는 사실과 현재 tenancy에서 구독·사용할 수 있다는 사실도 구분해야 한다.

기존 VCN 공동 생성 자료의 위치 선택 부분은 [보존된 Region 선택 Hands-on](../../Hands-on/Original-Provisional/01_Introduction_to_OCI/01_Regions.md#실습절차)에 있다.

## 독립성과 복구의 한계

Region 간 독립성은 재해 복구 설계의 기반이지 애플리케이션 복구 자체가 아니다. 다른 Region에 필요한 자원, 데이터 복제, 접속 전환과 복구 목표를 별도로 설계해야 한다. 모든 Region이 세 AD를 제공한다고 가정할 수도 없다. 서비스 지원과 구독 제한은 공식 목록과 실제 tenancy 조건에 따라 달라진다.

## 공식문서

- [Regions and Availability Domains](https://docs.oracle.com/en-us/iaas/Content/General/Concepts/regions.htm): Region 정의, realm, Region identifiers, 서비스 및 자원의 위치 범위.
