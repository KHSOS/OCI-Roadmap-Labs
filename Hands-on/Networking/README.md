# Networking

VCN은 OCI 리소스의 IP 주소 공간, subnet, 경로와 통신 허용 규칙을 연결합니다. 처음에는 네트워크를 만들고 Console에서 연결 관계를 확인하세요. VM이나 SSH는 첫 네트워크 실습의 필수 조건이 아닙니다.

[Hands-on 홈](../README.md) · [VCN 개념](../../Docs/03_Virtual_Cloud_Network_VCN/00_Virtual_Cloud_Network_VCN.md)

## 시작하기

[VCN 만들기](VCN/GettingStarted.md)에서 개인 새 실습 네트워크를 구성합니다. 이어서 VM을 만들려면 [네트워크 준비 완료](VCN/GettingStarted.md#5-네트워크-준비-완료)에서 자원을 남기고 [Compute 시작하기](../Compute/GettingStarted.md)를 진행하세요. 네트워크만 배운다면 같은 안내의 [리소스정리](VCN/GettingStarted.md#리소스정리)로 마칩니다.

## 이어서 직접 해 보기

[Compute 시작하기](../Compute/GettingStarted.md)의 SSH 경로를 완료하여 public admin VM을 남긴 뒤 NAT부터 진행합니다. NAT 안내에 private subnet·VM 생성과 ProxyJump 접속 전체 단계가 있습니다. 세 실습은 같은 작은 baseline(VM 두 대)을 재사용하며 새 VM을 주제마다 추가하지 않습니다.

| 순서 / 튜토리얼 | 실제로 할 작업 | 시작 조건 |
|---|---|---|
| 1. [NAT Gateway](NATGateway.md) | private VM의 HTTPS 성공 → NAT 차단 → 복구 | VCN 완료 + public VM SSH |
| 2. [Service Gateway](ServiceGateway.md) | 인터넷 default route 없이 private Object Storage 읽기; 권한 오류와 경로 실패 비교 | NAT 완료 + 같은 Region의 Object Storage 첫 파일 |
| 3. [Network Security Groups](NetworkSecurityGroups.md) | admin→private HTTP200 → NSG ingress 제거 → 복구 | [NAT 1–4단계 private SSH baseline](NATGateway.md#private-ssh-baseline-준비-완료); SGW/Object Storage는 선택 |

생성 필드·실제 client 명령·예상 결과·복구·정리를 포함한 튜토리얼입니다. OCI 실행 결과를 대신하는 것은 아닙니다. 다음 주제는 아직 실행 안내가 없습니다.

| 계획된 주제 | 용도 | 상태 |
|---|---|---|
| Local Peering Gateway | 같은 Region의 VCN peering | 계획됨; 실행 안내 없음 |
| Dynamic Routing Gateway | 외부 네트워크 및 VCN 연결의 라우팅 허브 | 계획됨; 실행 안내 없음 |

NAT/SGW/NSG의 기본 실행 경로는 위 새 튜토리얼 하나씩입니다. 기존 Networking 절차는 이 디렉터리의 `Original-Provisional`에 보존된 **legacy 미검증 초안**이며 기본 실행 경로가 아닙니다. 위 새 튜토리얼의 필수 선행 단계로 사용하지 않습니다. 전체 실습이 끝나면 NSG 앱/규칙과 SGW/PAR부터 정리하고 NAT 안내의 VM/boot→gateway/subnet→VCN 정리로 마칩니다.

## 네트워크와 통신의 확인은 다릅니다

Console에서 VCN과 subnet을 확인한 것만으로 VM의 실제 접속 성공을 의미하지 않습니다. 연결을 시험하는 후속 실습에서는 route, NSG/Security List, guest firewall과 클라이언트 경로를 함께 확인해야 합니다. 생성한 자원은 후속 실습이 끝난 뒤 의존성 역순으로 정리하세요.
