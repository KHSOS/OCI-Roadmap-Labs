# Cloud Deployment Models

Cloud Deployment Models는 클라우드 환경의 **전용성, 배치 방식, 여러 환경의 조합**을 설명한다. IaaS / PaaS / SaaS가 제공 계층의 차이라면 Public / Private / Hybrid / Multicloud는 배포 환경의 차이다. 두 축은 별도로 판단해야 한다. [Oracle: Types of Cloud Computing](https://www.oracle.com/cloud/what-is-cloud-computing/)

## 주요 모델

| 모델 | 의미 | 활용을 검토하는 상황 | 선택의 부담 |
|---|---|---|---|
| Public Cloud | 제공자의 데이터센터에 있는 기반 시설을 서비스로 이용 | 데이터센터를 직접 운영하지 않고 수요 변화에 대응 | 고객의 데이터·접근 관리와 서비스별 책임 판단 |
| Private Cloud | 한 조직만 사용하는 전용 서버·기반 시설의 클라우드 | 전용 환경과 높은 제어 수준이 중요한 요구 | 일반적으로 더 높은 비용과 Public Cloud보다 작은 유연성 |
| Hybrid Cloud | Public과 Private Cloud를 함께 사용 | 일부 중요 업무를 전용 환경에 유지하거나 단계적 이전 | 두 환경을 함께 운영하고 데이터를 관리하는 복잡성 |
| Multicloud | 둘 이상의 클라우드 제공자를 이용 | 제공자별 기술 활용 또는 단일 제공자 의존 위험 분산 | 여러 환경의 데이터 관리와 이식성 |

Private Cloud는 고객 시설에만 존재하는 것이 아니다. 제공자의 데이터센터에 호스팅될 수도 있다. Public Cloud는 보통 여러 고객이 기반 시설을 공유하지만, 제공자가 고객 간 데이터를 분리하고 보호할 책임을 가진다. [Oracle: Public / Private / Hybrid / Multicloud subsections](https://www.oracle.com/cloud/what-is-cloud-computing/)

## Public Cloud와 Public 접근은 다르다

Public Cloud라는 분류는 서비스 제공 환경을 설명하며 모든 애플리케이션이나 데이터가 익명으로 공개된다는 뜻이 아니다. Public Cloud 안에서도 고객의 데이터는 다른 고객으로부터 보호되어야 한다. 네트워크의 Public / Private Subnet이라는 표현은 이 배포 모델과 별도로 해석해야 한다. OCI의 Subnet Access 선택은 subnet 안의 Compute instance나 Load Balancer에 public IP를 사용할 수 있는지와 관련된다. 조직 전용 클라우드 기반 시설의 여부를 선택하는 필드는 아니다. [Oracle: Creating a Subnet — Subnet Access](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/create_subnet.htm)

따라서 Private Subnet에 배치된 VM을 Private Cloud라고 부르거나, 개인 PC에서 원격 접속했다는 이유만으로 Hybrid Cloud를 구현했다고 보는 것은 적절하지 않다. 배포 모델은 전체 환경의 성격과 조합으로, 접속 방식은 네트워크 구성으로 구분한다. [Oracle: Types of Cloud Computing](https://www.oracle.com/cloud/what-is-cloud-computing/)

## 요구와 모델을 연결하기

빠른 서비스 시작과 데이터센터 운영 부담 감소가 우선이면 Public Cloud를 검토한다. 단일 조직용 전용 기반 시설이 요구되면 Private Cloud의 비용·운영 범위를 함께 검토한다. 기존 전용 환경을 유지하면서 일부 서비스를 이전하려면 Hybrid Cloud를 검토할 수 있다. 서로 다른 제공자의 기술을 조합하려면 Multicloud를 검토한다.

이 선택은 한 가지 모델의 우열을 정하는 문제가 아니다. 데이터 위치와 규제, 제어 수준, 비용, 기술 요구, 기존 자산과 장기 이식성을 함께 보는 설계 문제다. Hybrid와 Multicloud는 함께 나타날 수 있지만 같은 뜻은 아니다. [Oracle: Types / Challenges / Migration](https://www.oracle.com/cloud/what-is-cloud-computing/)

## 한계와 오해

전용 환경을 선택한다고 모든 보안 문제가 자동 해결되지는 않는다. 보안은 제공자와 고객의 공동 책임이며, 여러 환경에 데이터가 분산되면 관리 복잡성이 높아질 수 있다. Public Cloud의 낮은 초기 투자도 항상 낮은 총비용을 보장하지 않는다. 제품과 환경의 실제 조건을 따로 검토해야 한다. [Oracle: Cloud Computing Challenges](https://www.oracle.com/cloud/what-is-cloud-computing/)

## 관련 개념과 Hands-on

- [IaaS / PaaS / SaaS](02_IaaS_PaaS_SaaS.md): 배포 위치가 아니라 제공 계층의 선택.
- [Cloud Computing](01_Cloud_Computing.md): 서비스 이용과 탄력성·비용 모델.
- [보존된 배포 모델 해석 워크숍](../../Hands-on/Original-Provisional/00_Introduction/03_Cloud_Deployment_Models.md): Public Cloud 안의 Private Subnet 배치를 읽는 사례. 별도 Private Cloud나 Hybrid VPN 구축 절차는 아니다.

## 공식문서

- [Oracle — What Is Cloud Computing?](https://www.oracle.com/cloud/what-is-cloud-computing/): Types of Cloud Computing, Challenges, Migration.
