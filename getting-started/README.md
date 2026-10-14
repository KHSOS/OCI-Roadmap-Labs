# OCI 시작 실습 — 개인 새 실습 3편

처음 OCI Console을 사용하는 개인 계정을 위한 실습입니다. 새 실습 자원만 만들며 기존 공유 자원은 수정하지 않습니다. 각 문서에서 생성 전 권한·요금을 확인하고, 실습 마지막에 만든 자원을 정리하세요.

**바로가기**: [읽는 방법](#읽는-방법) · [진행 순서](#진행-순서) · [시작 전 확인](#시작-전-확인)

| 배우고 싶은 작업 | 시작 문서 | 핵심 결과 |
|---|---|---|
| 내 네트워크 구성 | [VCN](VCN.md) | VCN·subnet·인터넷 경로·보안 목록의 연결 확인; VM·SSH 불필요 |
| 내 Linux VM 생성 | [Compute](Compute.md) | VM 생성·Console 확인; SSH 로그인과 outbound HTTPS는 선택 |
| 파일 저장·사용 | [Object Storage](ObjectStorage.md) | 작은 파일 업로드·다운로드·내용 비교; 다른 실습 불필요 |

## 읽는 방법

각 서비스 문서는 먼저 **어떤 문제에 쓰는지 → 구성 요소가 어떻게 연결되는지 → 이번에 고를 옵션의 의미**를 설명하고, 이어서 Console에서 첫 자원을 만드는 절차로 안내합니다. 개념을 모두 외우고 시작할 필요는 없습니다. 설정 중 헷갈리면 아래의 개념 부분으로 돌아오세요.

- [VCN: 주소 범위·경로·통신 허용](VCN.md#vcn은-무엇을-연결하나요) — subnet은 어디에 놓고, 인터넷 경로와 보안 규칙은 왜 따로 설정하나요?
- [Compute: Image·Shape·Boot Volume·VNIC](Compute.md#vm은-어떤-구성으로-실행되나요) — VM 생성과 SSH 로그인은 어떻게 다른가요?
- [Object Storage: Bucket·Object name·Private·Standard](ObjectStorage.md#파일이-object가-되면-무엇이-달라지나요) — 파일을 어디서 찾고, 공개하지 않고 어떻게 내려받나요?

## 진행 순서

- **네트워크만**: VCN 1–5단계 → [VCN 리소스정리](VCN.md#리소스정리).
- **VM까지**: [VCN 5단계: 네트워크 준비 완료](VCN.md#5-네트워크-준비-완료)에서 네트워크를 남김 → Compute 2–3단계 → [VM/boot volume 정리](Compute.md#리소스정리) → [VCN 정리](VCN.md#리소스정리). SSH도 하려면 Compute 1단계를 생성 전에 진행하고 4단계를 이어서 수행합니다. HTTPS는 SSH 이후 선택 5단계입니다.
- **파일만**: Object Storage 1–4단계 → [리소스정리](ObjectStorage.md#리소스정리). 삭제·재업로드는 정리 전에 선택할 수 있는 5단계입니다.

기본 규모는 새 VCN 1개와 subnet 1개, Compute 선택 시 VM 1대, Object Storage 선택 시 새 bucket 1개와 작은 파일 1개입니다. 자원 상세의 Region·Compartment·이름·연결을 확인해 이번 자원만 정리합니다.

## 시작 전 확인

- **Region**은 자원을 만들 지역이고 **Compartment**는 자원을 분류하는 공간입니다. VCN과 Compute는 같은 Region·Compartment를 사용합니다.
- Console 로그인만으로 모든 작업 권한이 생기지는 않습니다. 각 문서의 권한 항목을 확인하세요.
- 무료 적용 여부와 한도는 계정별로 다릅니다. 생성 화면과 요금을 먼저 확인하세요.
- SSH를 선택할 때만 PC의 OpenSSH client, 생성 중 내려받는 private key, PC 공인 IPv4 하나의 `/32` 허용이 필요합니다.

> 이 문서는 실행 안내이며 OCI에서 실제 실행한 결과를 제공하지 않습니다. 화면 이름과 제공 가능한 shape는 계정·Region에 따라 확인하세요.
