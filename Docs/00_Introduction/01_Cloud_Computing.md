# Cloud Computing

Cloud Computing은 infrastructure, software, storage, databases, development platforms 등을 네트워크를 통해 서비스로 제공하는 모델이다. 고객이 데이터센터의 장비를 직접 소유·유지하는 대신 필요한 기능을 이용하고, 제공자가 물리적 기반 시설을 운영한다. 비용은 일반적으로 구독 또는 사용량과 연결되며, 실제 과금 단위는 서비스별로 다르다. [Oracle: What Is Cloud Computing?](https://www.oracle.com/cloud/what-is-cloud-computing/)

## 핵심 특성

**자원에 대한 접근과 자원 소유는 다르다.** 고객은 필요한 기능과 용량을 선택하지만 제공자의 데이터센터·전력·냉각·물리 보안까지 직접 운영하지 않는다. 반면 애플리케이션, 데이터, 접근 권한과 같은 책임은 선택한 서비스 모델에 따라 고객에게 남는다.

**확장성은 수요 변화에 대응할 수 있는 능력이다.** 늘어난 사용자를 위해 자원을 확장하고 수요가 줄면 축소할 수 있다. 웹 애플리케이션의 자동 확장은 그 능력을 활용하는 설계 사례이지, 모든 클라우드 자원에 기본 적용되는 속성이 아니다. 파일 두 개를 저장했다가 하나를 삭제하는 행동은 저장 사용량의 변화일 뿐 Compute autoscaling과는 다르다. [Oracle: Cloud Computing Explained / Scalable Web Applications](https://www.oracle.com/cloud/what-is-cloud-computing/)

## 활용 사례와 선택 기준

| 요구 | 클라우드 활용 방향 | 함께 판단할 문제 |
|---|---|---|
| 짧은 개발·테스트 환경 | 필요한 기간에 개발 플랫폼이나 인프라 이용 | 팀이 운영할 계층과 사용 종료 시점 |
| 수요가 변하는 웹 서비스 | 수요에 맞춘 자원 확장·축소 설계 | 애플리케이션 구조와 실제 확장 기능 |
| 협업과 표준 업무 | 중앙화된 서비스와 SaaS 활용 | 사용자 접근과 데이터 관리 |
| 데이터 분석 | 클라우드의 처리·분석 기능 이용 | 데이터 위치와 여러 환경 사이의 관리 |
| 재해 복구 | 백업·복원 및 다른 위치의 자원 활용 | 실제 복구 설계와 필요한 서비스 선택 |

이 표는 Oracle의 Use Cases를 선택 질문으로 정리한 것이다. 특정 OCI 구성을 생성하거나 복구 성능을 보장하는 절차는 아니다. [Oracle: Cloud Computing Use Cases](https://www.oracle.com/cloud/what-is-cloud-computing/)

## 다른 개념과의 관계

[IaaS / PaaS / SaaS](02_IaaS_PaaS_SaaS.md)는 같은 클라우드 모델 안에서 서비스로 받는 계층을 구분한다. [배포 모델](03_Cloud_Deployment_Models.md)은 그 서비스를 어느 환경에 어떤 조합으로 두는지 설명한다. 클라우드 서비스에 인증해서 접근하는 것과 데이터를 익명으로 공개하는 것은 서로 다른 선택이다.

## 비용과 운영의 한계

클라우드는 큰 초기 장비 구매를 줄일 수 있지만 비용 관리를 없애지는 않는다. 사용하지 않는 용량을 유지하거나 제공자의 요금 구조를 잘못 이해하면 예상 밖 청구가 생길 수 있다. 소비량과 서비스별 가격 구조를 함께 봐야 한다.

보안·규제 준수는 공동 책임이며, 데이터가 놓인 국가와 누가 접근하는지도 중요하다. Hybrid / Multicloud는 선택 폭을 넓히지만 데이터 관리 복잡성과 장기 이식성 문제를 추가할 수 있다. [Oracle: Cloud Computing Challenges](https://www.oracle.com/cloud/what-is-cloud-computing/)

## Hands-on으로 이어가기

[보존된 Object Storage 사용량 워크숍](../../Hands-on/Original-Provisional/00_Introduction/01_Cloud_Computing.md)에서는 작은 합성 파일의 업로드·다운로드·삭제를 다룬다. 이 사례가 보여 주는 것은 object 저장량의 변화이며, 자동 확장이나 비용 절감률의 실험은 아니다.

## 공식문서

- [Oracle — What Is Cloud Computing?](https://www.oracle.com/cloud/what-is-cloud-computing/): Explained, Services, Use Cases, Challenges.
