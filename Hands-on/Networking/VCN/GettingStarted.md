# Virtual Cloud Network 시작하기: 내 실습 네트워크 만들기

VCN은 OCI 안에서 자원을 연결하는 가상 네트워크입니다. 이번에는 개인 계정에 **새로운 독립 실습 네트워크**를 만들고, Console에서 subnet·인터넷 경로·보안 설정의 연결을 확인한 뒤 정리합니다. VM, SSH, 키 준비는 필요하지 않습니다.

[실습 목록으로](../../README.md)

**바로가기**: [1. VCN 생성](#1-vcn-만들기--console) · [2. 보안 목록·경로표](#2-전용-security-list와-route-table-만들기) · [3. IGW·경로](#3-인터넷-출입구와-경로-연결하기) · [4. Subnet 생성](#4-public-subnet-만들기) · [5. 준비 완료](#5-네트워크-준비-완료) · [검증·문제해결](#결과검증과-문제해결) · [정리](#리소스정리)

## VCN은 무엇을 연결하나요

VCN은 VM을 만드는 서비스가 아니라, VM의 네트워크 인터페이스가 사용할 주소와 통신 경로를 구성하는 서비스입니다. 예를 들어 나중에 Linux VM을 만든다면 VM의 primary VNIC를 `lab-public`에 연결하고, 그 subnet 범위에서 private IP를 받습니다. 이번에는 이 기반만 먼저 만들어 봅니다.

### 주소 범위와 배치 이해하기

`10.77.0.0/16`과 `10.77.10.0/24`의 `/숫자`는 주소에서 네트워크 부분의 길이를 나타내는 CIDR 표기입니다. 이번 `/24` subnet은 `/16` VCN 범위의 일부입니다. VM의 private IP 예를 들면 `10.77.10.20`처럼 subnet 안의 주소가 되지만, 이 예시를 VM에 직접 입력하는 것은 아닙니다. 실제 주소는 나중에 생성된 VNIC 상세에서 확인합니다.

VCN은 Region 단위의 네트워크이고 이번 **Regional subnet**은 특정 Availability Domain 하나에 한정되지 않습니다. Compartment는 이 네트워크를 관리하는 분류 공간이지 subnet을 대신하는 주소 범위가 아닙니다.

### 경로와 허용은 어떻게 다른가요

후속 SSH를 예로 들면 인터넷의 PC는 VM의 public IP로 접속합니다. 이때 public IP를 허용하는 subnet, VM의 실제 public IP, Internet Gateway와 route, TCP22를 허용하는 security rule이 함께 필요합니다. 어느 하나의 설정만으로 접속이 완성되지 않습니다.

- **Route table**은 목적지에 따라 어디로 보낼지 정합니다. `0.0.0.0/0 → igw-lab`은 인터넷 목적지의 출구입니다.
- **Security list**는 subnet의 VNIC들에 적용할 통신 허용 규칙입니다. 같은 `0.0.0.0/0`이라도 SSH ingress의 source에 넣으면 모든 IPv4 주소를 허용하는 전혀 다른 의미가 됩니다.
- **Public subnet**은 VNIC에 public IP를 부여할 수 있다는 뜻입니다. subnet을 만들었다고 VM이나 public IP가 생성되지는 않습니다.

아래 절차는 먼저 경로를 만들고 security list는 비워 둡니다. 네트워크 구성 요소의 관계를 확인한 뒤, 실제 접속이 필요할 때만 Compute에서 내 PC의 SSH를 허용합니다. [공식 Networking 개요](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/overview.htm)에서 구성 요소의 역할을 더 볼 수 있습니다.

## 학습목표

VCN 하나와 public subnet 하나를 만들고, subnet에 연결된 route table과 security list를 확인합니다. Internet Gateway로 향하는 경로를 확인하고 이번에 만든 네트워크를 삭제합니다. VM 접속까지 해 보고 싶다면 네트워크 준비 뒤 Compute를 선택할 수 있습니다.

## 대응 Roadmap 항목

**Virtual Cloud Network (VCN)** · `sequence-3-0`. CIDR, Public Subnets, Internet Gateway, Route Tables, Security Rules를 한 번의 생성 과정에서 익힙니다.

## 사전준비

개인 OCI Console에 로그인하고 사용할 Region을 선택합니다. 자원을 담을 Compartment 하나를 Console 목록에서 선택하고 이 실습 전체에서 그대로 사용합니다. 기존 네트워크는 수정하지 않습니다. Compartment는 자원을 분류하는 공간이며 이 실습에서 별도로 만들 필요는 없습니다.

주소는 그대로 입력합니다. `10.77.0.0/16`은 **VCN 내부주소 범위**, `10.77.10.0/24`는 그 안에서 사용할 **서브넷 범위**입니다. 인터넷에서 접속할 공인 IP와는 다릅니다.

> 나중에 기존 네트워크와 연결할 때만 서로의 내부주소 범위가 겹치지 않는지 확인하세요.

## 권한

로그인한 계정으로 선택 Compartment의 VCN, subnet, gateway, route table, security list를 생성·조회·변경·삭제할 수 있어야 합니다. 개인 계정에서도 권한 오류가 나면 해당 작업을 멈추고 계정의 IAM 설정을 확인합니다. 권한을 확인하려고 기존 자원을 바꾸거나 광범위한 정책을 추가하지 않습니다. [Core Services IAM reference](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm)에서 네트워크 작업의 권한과 연결 의존성을 확인할 수 있습니다.

## 비용및주의사항

이 핵심 실습에는 VM, NAT Gateway, Load Balancer, DRG, logging을 추가하지 않습니다. 후속 Compute를 선택하면 VM·boot volume·공인 IPv4·전송의 요금을 별도로 확인합니다. 무료라는 가정으로 진행하지 않습니다. 네트워크 삭제는 되돌릴 수 없으므로 이번 새 VCN 상세에서만 작업합니다.

## 실습구성

이번에는 VCN 1개, subnet 1개, 전용 route table·security list·Internet Gateway를 각각 1개 만듭니다. VM은 만들지 않습니다. 아래 생성 순서는 VCN을 먼저 만든 뒤 subnet에 연결할 구성 요소를 준비하고 마지막에 subnet을 연결하는 순서입니다.

`lab-vcn` 안에 Regional public subnet `lab-public`을 둡니다. `rt-public`의 인터넷 목적지 경로는 `0.0.0.0/0 → igw-lab`입니다. 이 값은 **목적지 경로**이며 모든 사람의 SSH를 허용하는 규칙이 아닙니다. `sl-public`은 처음에는 ingress/egress 모두 비워 둡니다. 따라서 경로를 만들어도 실제 통신 허용이나 VM 공인 IP가 생긴 것은 아닙니다.

## 실습절차

### 1. VCN 만들기 — Console

**Networking → Virtual cloud networks → Create VCN**을 엽니다. 이번에는 Wizard 대신 VCN만 생성한 다음 subnet·route table·security list·Internet Gateway를 직접 연결합니다. 화면 이동은 더 많지만 어느 설정이 subnet에 연결되는지 하나씩 확인하기 위한 경로입니다.

| 필드 | 값 | 이유 |
|---|---|---|
| Name / Compartment | `lab-vcn` / 선택한 Compartment | 이번 개인 실습을 구분 |
| IPv4 CIDR Blocks | `10.77.0.0/16` 하나 | VCN 내부주소 범위 |
| Use DNS Hostnames in this VCN | 선택, DNS label=`labvcn` | 후속 VM의 내부 이름 사용 |
| IPv6 / 추가 보안 속성 | 추가하지 않음 | IPv4 시작 실습에 집중 |

Create를 한 번 누릅니다. 상세에서 `AVAILABLE`, CIDR, 선택 Compartment가 맞는지 확인합니다. 같은 이름이 여러 개일 수 있으므로 **방금 생성한 상세 페이지**를 계속 사용합니다. 자동 생성되는 default route table·security list·DHCP options는 그대로 둡니다. 실패하면 오류와 현재 목록을 먼저 읽고 생성 버튼을 반복하지 않습니다.

### 2. 전용 security list와 route table 만들기

같은 VCN의 **Security → Security Lists → Create Security List**(화면에 따라 Resources → Security Lists)에서 Name=`sl-public`을 입력합니다. 표시된 ingress/egress 행을 모두 제거하고 생성합니다. 생성 후 **Ingress 0개, Egress 0개**인지 확인합니다. 빈 목록을 저장할 수 없다면 [문제해결](#결과검증과-문제해결)을 확인합니다.

이어서 **Routing → Route Tables → Create Route Table**에서 Name=`rt-public`, 같은 Compartment, route rule 없음으로 생성합니다. 상세의 규칙 목록이 비어 있으면 다음으로 갑니다.

### 3. 인터넷 출입구와 경로 연결하기

같은 VCN의 **Gateways → Internet Gateways → Create Internet Gateway**에서 Name=`igw-lab`, 같은 Compartment를 선택하여 생성합니다. Enabled 상태를 확인합니다. gateway 자체의 Route Table Association은 비워 둡니다. 이번에는 subnet의 route table에 경로를 넣습니다.

**Routing → Route Tables → rt-public → Add Route Rules**에서 다음을 저장합니다.

- Target Type=`Internet Gateway`
- Destination Type=`CIDR Block`, Destination CIDR=`0.0.0.0/0`
- Target Compartment=이번 Compartment, Target=`igw-lab`

저장 후 해당 route의 target이 방금 만든 IGW인지 상세에서 확인합니다. IGW 생성만으로 이 경로가 자동 연결되는 것은 아닙니다.

### 4. Public subnet 만들기

같은 VCN의 **Subnets → Create subnet**을 엽니다.

| 필드 | 값 | 이유 |
|---|---|---|
| Name / Compartment | `lab-public` / 같은 Compartment | 이번 subnet 식별 |
| Subnet Type | `Regional` | Region 안의 여러 AD에서 사용 가능 |
| IPv4 CIDR | `10.77.10.0/24` | VCN 안의 서브넷 범위 |
| Subnet Access | `Public subnet` | 후속 VM에 공인 IP 할당 허용 |
| Route Table | `rt-public` | 3단계 인터넷 경로 사용 |
| Security Lists | `sl-public` 하나만 | 넓은 default 허용을 연결하지 않음 |
| DHCP options | 이 VCN의 default | 기본 resolver 설정 사용 |
| DNS hostnames / DNS label | 사용 / `labpublic` | 내부 hostname 사용 |

자동 선택된 default security list는 선택 해제합니다. IPv6와 Resource logging은 켜지 않습니다. Create 후 `AVAILABLE`을 확인합니다. 오류가 나면 잘못 선택한 VCN/CIDR/연결 필드를 고친 뒤, 이미 만들어졌는지 확인하고 다시 시도합니다.

### 5. 네트워크 준비 완료

`lab-public` 상세에서 **Public, Regional, `10.77.10.0/24`, route table=`rt-public`, security list=`sl-public` 하나, default DHCP**를 읽습니다. `rt-public`의 `0.0.0.0/0 → igw-lab`, IGW Enabled, `sl-public`의 양방향 빈 목록도 각각 확인합니다. 이 상태면 네트워크 생성과 연결 확인을 마쳤습니다. 통신할 VM은 아직 없고 security list도 비어 있으므로 인터넷 접속 시험은 하지 않습니다.

여기서 끝내면 바로 아래 정리로 갑니다. VM을 직접 사용해 보고 싶을 때만 이 네트워크를 잠시 남기고 [Compute 시작하기](../../Compute/GettingStarted.md#사전준비)로 이동합니다. Compute에서는 이 subnet을 선택하고 SSH를 선택할 때만 보안 규칙을 추가합니다. VM을 마치면 Compute의 VM/boot volume 정리 후 이 문서의 네트워크 정리로 돌아옵니다.

### 결과검증과 문제해결

5단계의 각 값을 Console에서 확인합니다. 연결이 다르면 해당 상세 화면에서 선택값을 고치고 다시 읽습니다. `Public subnet`은 공인 IP를 허용하는 속성이지 실제 VM/IP가 존재한다는 표시가 아닙니다.

- subnet 생성 오류: `10.77.10.0/24`를 VCN `10.77.0.0/16` 안의 범위로 입력했는지, 같은 subnet을 이미 만들었는지 확인합니다.
- security list 빈 목록을 저장할 수 없음: 입력 행을 모두 제거했는지 확인하고 공식 생성 절차와 화면을 비교합니다. 넓은 기본 허용으로 대체하지 않습니다.
- 연결이 다름: subnet 상세에서 route table=`rt-public`, security list=`sl-public` 하나로 수정한 뒤 5단계를 다시 확인합니다.
- 삭제의 `Resource in use`: 오류에 나온 subnet 사용 자원을 먼저 확인합니다. VM이 남아 있으면 아래 1번부터 진행합니다.

### 리소스정리

1. 이 subnet에 VM이 남아 있다면 [Compute 리소스정리](../../Compute/GettingStarted.md#리소스정리)의 1–3번으로 VM과 boot volume부터 정리한 뒤 이 목록의 2번을 진행합니다. Compute 정리를 마치고 온 경우나 VM을 만들지 않은 경우에는 곧바로 2번으로 갑니다.
2. **Networking → Virtual cloud networks → 방금 만든 lab-vcn → Actions → Delete**에서 **Search compartments for resources associated with this VCN → All compartments → Scan**을 선택합니다.
3. 검색 결과가 이번 VCN과 이번에 만든 네트워크 자원뿐인지 확인합니다. 모르는 자원이나 VM이 있으면 취소합니다. 맞으면 **Delete All**을 선택합니다. 자동 생성된 이 VCN의 default 자원도 이때 함께 정리합니다.
4. 진행 결과를 확인하고 같은 Region·Compartment 목록을 새로고침하여 해당 VCN이 사라졌는지 확인합니다. 일부 삭제가 실패하면 오류에 나온 이번 자원을 확인하고, 목록 조회가 성공한 뒤 삭제 여부를 다시 확인합니다. Compartment 자체는 삭제하지 않습니다.

### 공식문서

- [Creating a VCN](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/create_vcn.htm)
- [Creating a Subnet](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/create_subnet.htm)
- [Creating a Security List](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/creating-securitylist.htm)
- [Creating a Route Table](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/create-routetable.htm)
- [Creating an Internet Gateway](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/create-ig.htm)
- [Deleting a VCN](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/delete_vcn.htm)
