# Introduction

클라우드를 배우는 출발점은 서버를 만드는 화면이 아니라 **어떤 기능을 서비스로 이용하고, 무엇을 직접 관리할지**를 정하는 것이다. Cloud Computing은 서버·저장소·데이터베이스·개발 플랫폼·업무 소프트웨어 등을 네트워크를 통해 이용하는 모델이다. 제공자가 데이터센터와 기반 시설을 운영하고, 고객은 업무에 필요한 서비스를 선택한다. 책임 분담은 서비스 모델에 따라 달라진다. [Oracle: What Is Cloud Computing?](https://www.oracle.com/cloud/what-is-cloud-computing/)

## 이 Roadmap을 읽는 관점

- **서비스 모델:** IaaS / PaaS / SaaS는 고객이 직접 관리하는 계층과 제공자가 운영하는 계층을 구분한다. 같은 조직이 기존 애플리케이션에는 IaaS, 신규 개발에는 PaaS, 인사 업무에는 SaaS를 함께 사용할 수 있다.
- **배포 모델:** Public / Private / Hybrid / Multicloud는 서비스를 배치하고 조합하는 방식이다. 서비스 모델과 별개의 축이므로 IaaS를 선택했다고 Private Cloud를 선택한 것은 아니다.
- **업무 요구:** 개발·테스트, 웹 서비스 확장, 데이터 분석, 협업, 재해 복구는 서로 다른 요구를 가진다. 필요한 제어 수준·데이터 위치·접근 권한·가용성과 비용을 함께 고려해야 한다. [Oracle: Services / Types / Use Cases](https://www.oracle.com/cloud/what-is-cloud-computing/)

## 언제 클라우드가 유용한가

새 서비스의 수요를 아직 모를 때는 장비를 먼저 구매하는 대신 필요한 자원을 이용하는 방식이 초기 투자를 줄이고 변경을 빠르게 한다. 자체 소프트웨어를 개발하는 팀은 개발·테스트 환경과 배포 플랫폼을 활용할 수 있고, 표준 업무 기능을 원하는 팀은 완성된 SaaS를 선택할 수 있다. 이는 교육용 선택 예시이며, 특정 서비스가 항상 가장 저렴하거나 가장 적합하다는 의미는 아니다. [Oracle: Cloud Computing Explained / Benefits](https://www.oracle.com/cloud/what-is-cloud-computing/)

## 선택의 경계와 한계

클라우드의 확장성은 애플리케이션을 아무 설정 없이 자동 확장해 준다는 뜻이 아니다. 필요한 자원을 늘리고 줄이는 서비스 구성과 운영이 따로 필요하다. 또한 사용량 기반 비용은 낮은 초기 투자와 관련되지만 전체 비용 절감을 보장하지 않는다. 사용하지 않는 자원을 유지하면 비용이 계속 발생할 수 있다.

데이터와 애플리케이션 보안은 고객과 제공자의 공동 책임이다. 데이터 위치와 규제, 접근 통제, 다른 환경으로의 이식성도 설계 기준이다. 따라서 이 Roadmap은 자원 종류를 외우는 데서 끝나지 않고 위치·연결·책임·수명주기의 관계를 이해하는 데 사용한다. [Oracle: Cloud Computing Challenges](https://www.oracle.com/cloud/what-is-cloud-computing/)

## 관련 개념

- [Cloud Computing](01_Cloud_Computing.md): 서비스 제공 방식과 탄력성·사용량의 의미.
- [IaaS / PaaS / SaaS](02_IaaS_PaaS_SaaS.md): 관리 책임과 제어 수준의 차이.
- [Cloud Deployment Models](03_Cloud_Deployment_Models.md): 전용성·배치·환경 조합의 차이.

## Hands-on으로 이어가기

개념을 읽는 데 자원 생성은 필요하지 않다. 기존 Compartment 생성 워크숍은 [보존된 Hands-on](../../Hands-on/Original-Provisional/00_Introduction/00_Introduction.md)에 있다.

## 공식문서

- [Oracle — What Is Cloud Computing?](https://www.oracle.com/cloud/what-is-cloud-computing/): 정의, Services, Types, Use Cases, Challenges.
