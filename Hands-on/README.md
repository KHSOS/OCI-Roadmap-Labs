# OCI Hands-on

OCI Console에서 첫 자원을 만들고 결과를 확인하는 실습 안내입니다. 서비스 개념은 [Docs](../Docs/README.md), 실행 절차는 이 디렉터리에서 찾습니다. 개념 설명과 실행 절차는 별도 문서로 구성되어 있습니다.

## 시작하기

| 하고 싶은 작업 | 안내 | 첫 성공 |
|---|---|---|
| 네트워크 만들기 | [Networking](Networking/README.md) | VCN과 subnet, route, security 연결 확인 |
| Linux VM 만들기 | [Compute 시작하기](Compute/GettingStarted.md) | VM 생성과 Console 확인; SSH는 선택 |
| 파일 저장하기 | [Object Storage 시작하기](ObjectStorage/GettingStarted.md) | 작은 파일 업로드와 다운로드 |

VM을 만들려면 Networking의 VCN 안내를 먼저 완료하고 네트워크를 남깁니다. Object Storage는 다른 실습 없이 시작할 수 있습니다. 리소스를 생성하기 전에 각 안내의 권한과 비용을 확인하세요.

## 기존 주제별 절차

원래 93개 문서의 절차는 `Original-Provisional` 디렉터리에 전체 본문으로 보존했습니다. 일부 Networking 자료는 Networking 아래에 있습니다. 이 자료는 **기존 초안**이며 새로운 튜토리얼의 품질 승인이나 OCI 실행 검증을 의미하지 않습니다. 93개 개념 항목은 Docs의 연결 매핑에서 찾을 수 있습니다.

## 실습 상태

시작하기 3편은 기존 검토용 pilot을 이 경로로 옮긴 문서입니다. [NAT Gateway](Networking/NATGateway.md), [Service Gateway](Networking/ServiceGateway.md), [Network Security Groups](Networking/NetworkSecurityGroups.md)의 생성·검증·복구·정리 튜토리얼을 제공합니다. 주제별 기본 실행 경로는 이 세 문서이며 Original-Provisional은 legacy 보존 자료입니다. OCI 리소스를 실제 생성하거나 검증한 결과는 포함하지 않습니다.
