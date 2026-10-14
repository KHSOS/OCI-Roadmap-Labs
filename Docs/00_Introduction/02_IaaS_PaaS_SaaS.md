# IaaS / PaaS / SaaS

IaaS, PaaS, SaaS는 클라우드가 **어느 계층을 서비스로 제공하는지**를 구분하는 모델이다. 단순히 기능 수가 많고 적은 순서가 아니라, 고객이 제어할 범위와 직접 운영할 책임의 차이다. 실제 제품에서는 infrastructure와 platform의 경계가 겹칠 수 있으며, 조직은 세 모델을 함께 사용할 수 있다. [Oracle: Cloud Computing Services](https://www.oracle.com/cloud/what-is-cloud-computing/)

## 모델별 책임과 역할

| 모델 | 제공받는 것 | 고객에게 남는 주요 역할 | 적합한 요구의 예 |
|---|---|---|---|
| Infrastructure as a Service (IaaS) | Compute, Storage, Network 기반 자원 | 소프트웨어 설치·설정·보안·유지관리 | 기존 환경과 유사한 인프라에서 애플리케이션 이전 |
| Platform as a Service (PaaS) | 애플리케이션 개발·테스트·배포·실행을 위한 플랫폼과 도구 | 애플리케이션과 그 기능 관리 | 자체 애플리케이션을 개발하면서 기반 운영 부담 축소 |
| Software as a Service (SaaS) | 제공자가 개발하고 호스팅하는 완성된 소프트웨어 | 업무 기능 사용과 조직의 데이터·접근 관리 | 인사·물류·생산성 등 표준 업무 기능 이용 |

제공자는 IaaS에서도 전력·냉각·물리 보안·하드웨어 수리를 담당한다. PaaS에서는 플랫폼이 서버·네트워크 등 기반 운영을 맡지만 고객의 애플리케이션 기능까지 대신 작성하는 것은 아니다. SaaS는 사용자가 제품을 직접 설치하는 대신 네트워크를 통해 업무 기능을 이용하는 모델이다. [Oracle: IaaS / PaaS / SaaS subsections](https://www.oracle.com/cloud/what-is-cloud-computing/)

## 어떤 모델을 선택할까

기존 애플리케이션을 최대한 유지하면서 시스템 설정을 세밀하게 제어해야 한다면 IaaS가 검토 대상이다. 개발팀이 인프라 자체보다 코드와 기능 개발에 집중하려면 PaaS를 검토한다. 이미 표준화된 업무 기능을 빠르게 이용하는 것이 목적이라면 SaaS를 검토한다. 이는 Oracle의 모델 설명에서 도출한 선택 관점이며 모든 워크로드에 동일한 결론을 적용하는 규칙은 아니다.

한 회사에서 기존 시스템은 IaaS, 새 API 개발은 PaaS, 인사 업무는 SaaS로 운영할 수도 있다. 모델을 하나만 선택해야 하는 것은 아니다. [Oracle: Services / How to Migrate to the Cloud](https://www.oracle.com/cloud/what-is-cloud-computing/)

## 다른 개념과의 관계

서비스 모델은 [Cloud Deployment Models](03_Cloud_Deployment_Models.md)와 별개의 축이다. Public Cloud에서 IaaS를 이용할 수 있고, Public / Private 환경을 함께 쓰는 조직도 서로 다른 서비스 모델을 조합할 수 있다.

책임 경계는 보안과 연결된다. 제공자가 기반 계층을 더 많이 운영한다고 고객의 모든 보안 책임이 사라지는 것은 아니다. 누가 데이터와 서비스에 접근할 수 있는지, 조직이 어떤 의무를 지는지는 선택한 제품과 공동 책임 모델에 따라 확인한다. [Oracle: Cloud Computing Challenges](https://www.oracle.com/cloud/what-is-cloud-computing/)

## 모델을 해석할 때의 한계

서비스 이름이나 화면 하나만으로 실제 책임 경계를 확정하기 어렵다. Oracle도 infrastructure와 platform의 경계가 실무에서 명확하지 않을 수 있다고 설명한다. 제품별 운영 범위는 그 제품의 문서로 구체화해야 한다.

빈 application 리소스를 생성하는 것과 애플리케이션을 배포·실행하는 것은 다르다. 또한 PaaS의 application을 만들었다고 SaaS 제품을 도입한 것은 아니다. 개념 비교에서는 서비스의 제공 계층을, 실습에서는 실제 생성·실행한 대상을 구분한다.

## Hands-on으로 이어가기

[보존된 IaaS / PaaS 책임 비교 워크숍](../../Hands-on/Original-Provisional/00_Introduction/02_IaaS_PaaS_SaaS.md)은 기존 VM과 빈 Functions application의 비교를 다룬다. 함수 실행과 SaaS 도입은 그 워크숍의 대상이 아니다.

## 공식문서

- [Oracle — What Is Cloud Computing?](https://www.oracle.com/cloud/what-is-cloud-computing/): Cloud Computing Services, IaaS, PaaS, SaaS, Challenges, Migration.
