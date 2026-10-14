# Compute 시작하기: 내 Oracle Linux VM 생성·확인하기

Compute Instance는 OS와 프로그램을 실행하는 가상 서버입니다. 개인 실습 네트워크에 VM **한 대**를 만들고 Console에서 OS·CPU·메모리·디스크·네트워크를 확인합니다. SSH를 선택하면 Console이 만들어 주는 키로 로그인하여 `opc` 사용자와 OS를 확인합니다. 기본 경로는 2–3단계의 생성·Console 확인 후 정리입니다. SSH를 사용할 때만 1단계도 먼저 진행하고, 4–5단계를 이어서 선택합니다.

[실습 목록으로](../README.md)

**바로가기**: [1. 선택 SSH 준비](#1-ssh를-선택할-때만-내-pc-접속-허용하기) · [2. VM 생성](#2-vm-한-대-만들기--console) · [3. Console 확인](#3-console에서-vm-확인하기) · [4. 선택 SSH 로그인](#4-선택-ssh로-로그인하여-os-확인하기) · [5. 선택 외부 HTTPS](#5-선택-같은-vm의-https-확인하기) · [검증·문제해결](#결과검증과-문제해결) · [정리](#리소스정리)

## VM은 어떤 구성으로 실행되나요

Compute는 운영 체제와 프로그램을 실행할 서버를 제공합니다. 예를 들어 Linux 명령을 실행할 실습 서버가 필요하다면 VM을 만들고 SSH로 들어갈 수 있습니다. 파일 하나를 저장하고 내려받기만 한다면 VM 대신 [Object Storage](../ObjectStorage/GettingStarted.md)를 독립적으로 사용할 수 있습니다.

### Image와 Shape, Boot Volume의 관계

**Image**는 시작할 OS와 소프트웨어의 템플릿이고, **Shape**는 그 OS를 실행할 CPU·메모리 등의 자원 구성입니다. 둘을 선택하면 image를 바탕으로 이번 VM의 **Boot Volume**이 만들어져 OS 디스크로 연결됩니다. 같은 image를 고르더라도 실행 중인 VM의 디스크를 서로 공유한다는 뜻은 아닙니다.

이번에는 Arm 기반 A1 shape에 맞는 Oracle Linux image를 고릅니다. OS 이름만 보고 image를 선택하면 CPU 아키텍처가 맞지 않을 수 있으므로 생성 화면에서 호환성을 함께 확인합니다. CPU·메모리 크기 선택과 OS 디스크 크기는 서로 다른 필드입니다. VM을 종료할 때도 boot volume을 함께 삭제할지 별도로 선택하게 됩니다.

### VM의 위치와 접속 방법

VM은 선택한 Region의 Availability Domain(AD)에 생성됩니다. Fault Domain(FD)은 AD 안에서 장애 영향을 분리하는 배치 단위이며 이번에는 기본 선택을 사용합니다. Regional subnet은 여러 AD에서 사용할 수 있지만 VM 한 대가 여러 AD에 동시에 배치되는 것은 아닙니다.

VM의 **primary VNIC**를 준비한 `lab-public`에 연결하면 private IP로 VCN에 참여합니다. PC에서 직접 SSH를 하려면 public IP와 인터넷 경로, 내 PC를 허용한 TCP22 규칙도 필요합니다. 따라서 Console에서 VM을 보는 경로와 OS에 로그인하는 경로를 구분합니다.

| 이번에 할 일 | 필요한 준비 | 확인할 결과 |
|---|---|---|
| Console에서 생성과 구성 확인 | 기존 VCN/subnet, image/shape 선택 | RUNNING과 boot volume/VNIC 연결 |
| 선택: OS에 SSH 로그인 | 위 구성 + public IP + 내 PC `/32` ingress22 + 키 | VM 터미널의 `opc`와 Oracle Linux |
| 선택: VM에서 외부 HTTPS 요청 | SSH 완료 + egress443 | 외부 사이트 DNS 조회와 HTTP 응답 |

SSH의 키 쌍은 OCI 로그인 암호가 아닙니다. 공개키는 VM에 등록하고 private key는 PC가 접속할 때 사용합니다. 아래에서 SSH를 선택할 때만 내려받습니다. [공식 Compute 개요](https://docs.oracle.com/en-us/iaas/Content/Compute/Concepts/computeoverview.htm)에서 instance와 실행 구성의 관계를 더 볼 수 있습니다.

## 학습목표

Oracle Linux VM 한 대를 생성하고 instance·boot volume·primary VNIC의 연결을 확인합니다. SSH 선택 시 실제 로그인하여 OS를 확인하고, 마지막에 VM과 boot volume을 삭제합니다.

## 대응 Roadmap 항목

**Compute** · `sequence-2-0`. Image, Shape, Boot Volume, SSH Keys를 같은 VM 생성 화면에서 익힙니다. Cloud-init는 이 시작 실습의 필수 작업이 아닙니다.

## 사전준비

개인 OCI Console에 로그인합니다. [VCN 5단계: 네트워크 준비 완료](../Networking/VCN/GettingStarted.md#5-네트워크-준비-완료)까지 진행하고 **삭제하지 않은** `lab-vcn`과 `lab-public`을 사용합니다. 같은 Region과 Compartment를 선택합니다. 이미 VCN을 정리했다면 그 실습으로 새 네트워크를 먼저 준비합니다. VCN에는 VM이나 SSH 완료 조건이 없습니다.

SSH를 선택한다면 내 PC에 OpenSSH client와 private key를 저장할 공간이 필요합니다. PC 터미널에서 `ssh -V`로 client가 있는지 확인합니다. 키는 2단계에서 내려받습니다. 아래 PC 명령은 **Linux 또는 WSL의 Bash 터미널** 기준입니다(macOS에서도 같은 SSH 명령을 사용할 수 있습니다). Windows에서는 WSL 경로를 권장합니다. WSL을 사용하지 않을 때는 아래 4단계의 Oracle Windows 안내를 따릅니다. PowerShell에서 Bash 변수 설정이나 `chmod`를 실행하지 않습니다. SSH를 선택하지 않으면 접속 규칙·키 다운로드·터미널 명령은 생략하고 Console 확인으로 끝낼 수 있습니다.

## 권한

선택 Compartment에서 instance 생성·조회·종료, image 조회, 기존 subnet/VNIC 연결, boot volume 조회·삭제가 가능해야 합니다. SSH를 선택하면 이번 security list 규칙을 변경할 권한도 필요합니다. [Core Services IAM reference](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm)는 LaunchInstance의 image·VNIC·subnet 연결 권한도 설명합니다. 작업이 거부되면 멈추고 계정 IAM을 확인하며 무관한 자원이나 정책을 바꾸지 않습니다. OS의 `opc` 사용자 인증은 OCI Console 로그인과 별개입니다.

## 비용및주의사항

VM의 OCPU·메모리, boot volume, 공인 IPv4와 전송 요금을 생성 전에 확인합니다. 후보는 `VM.Standard.A1.Flex` **1 OCPU / 6 GB**이며 무료 적용과 잔여 한도·capacity는 계정별로 다릅니다. 선택할 수 없으면 큰 유료 shape로 자동 대체하지 말고 중단합니다. 추가 data volume, backup, LB, NAT, logging은 만들지 않습니다. 실습 후 Terminate하며 Stop만으로 디스크가 삭제되었다고 생각하지 않습니다.

private key는 내 PC에만 보관합니다. 채팅·공유 저장소·서버에 올리지 않습니다. SSH source는 내 PC 공인 IPv4 하나의 `/32`만 허용하며 모든 주소의 SSH는 열지 않습니다.

## 실습구성

기존 `lab-vcn`과 `lab-public`을 재사용하고 `lab-vm` 한 대와 그 boot volume·primary VNIC만 새로 만듭니다. Console만 사용할지 SSH도 사용할지 먼저 정하세요. 아래 생성 표에서 그 선택에 따라 public IP와 키 설정이 달라집니다. SSH 없이 만든 VM에 나중에 바로 접속할 수 있다고 가정하지 않습니다.

## 실습절차

### 1. SSH를 선택할 때만: 내 PC 접속 허용하기

현재 접속할 **PC의 공인 IPv4**를 브라우저에서 `https://api.ipify.org`를 열어 확인하거나 PC 터미널에서 다음을 실행합니다. 이 사이트는 공인 IP 확인용 외부 보조 서비스이며 OCI 기술 문서가 아닙니다. IPv4 대신 다른 결과가 나오면 진행하지 않습니다.

```bash
curl -4 --fail --max-time 10 https://api.ipify.org
```

예를 들어 결과가 `203.0.113.24`라면 Source CIDR 형식은 `203.0.113.24/32`입니다. **이 예시 주소를 그대로 쓰지 말고 현재 PC 결과를 입력합니다.** `/32`는 주소 하나만 허용한다는 뜻입니다.

Console → **Networking → Virtual cloud networks → lab-vcn → Security Lists → sl-public → Add Ingress Rules**에서 Stateless 해제(stateful), Source Type=`CIDR`, Source CIDR=`현재 PC 공인 IPv4/32`, IP Protocol=`TCP`, Source Port Range=`All`, Destination Port Range=`22`를 저장합니다. 저장된 규칙을 다시 읽습니다. 다른 허용을 추가하지 않습니다. IP가 나중에 바뀌면 이 규칙의 source만 새 `/32`로 수정합니다.

### 2. VM 한 대 만들기 — Console

**Compute → Instances → 선택 Compartment → Create instance**를 엽니다. 화면은 Basic Information, Security, Networking, Storage, Review 순서이며 필드 배치는 달라질 수 있습니다.

| 필드 | 값 | 이유 |
|---|---|---|
| Name / Compartment | `lab-vm` / 네트워크와 같은 Compartment | 새 개인 실습 VM |
| Placement | 현재 Region의 선택 가능한 AD, Fault domain 기본 | 실제 제공되는 위치 사용 |
| Capacity type | `On-demand` | 예약/전용 host 없이 한 대 생성 |
| Image | Oracle Linux 9의 Arm-compatible platform image | 아래 A1 shape와 호환 |
| Shape | Virtual machine → Ampere → `VM.Standard.A1.Flex`, 1 OCPU, 6 GB | 작은 시작 구성 |
| VCN / Subnet | 기존 `lab-vcn` / `lab-public` | 준비한 네트워크에 연결 |
| Automatically assign public IPv4 address | SSH 선택 시 자동 할당(ephemeral), Console만 사용하면 선택 해제 | 접속할 때만 공인 주소 사용 |
| Boot Volume | image 기본 크기를 읽고 요금 확인, 기본 성능/Oracle-managed 암호화 | OS 디스크만 생성 |

추가 초기화 script, data volume, backup policy, NSG, 별도 보안 서비스는 설정하지 않습니다. 보안 속성이 계정에서 필수라면 image/shape 호환을 확인한 뒤 진행합니다.

SSH를 선택했다면 **Add SSH keys → Generate a key pair for me → Save Private Key**로 PC에 키를 내려받습니다. 파일이 저장된 경로를 확인하세요. 공개키 등록은 Console이 처리하며 Save Public Key로 공개키도 보관할 수 있습니다. private key는 나중에 다시 내려받을 수 없습니다. Console만 사용할 때는 **No SSH keys**를 선택합니다. 이 경우 아래 SSH 단계는 건너뜁니다.

Review에서 image/shape/boot 크기/subnet/공인 IP와 비용을 다시 읽고 Create를 한 번 누릅니다. 생성 오류면 Work Requests와 현재 Instances 목록을 먼저 확인하고 중복 생성하지 않습니다. capacity 부족이면 여기서 중단합니다.

### 3. Console에서 VM 확인하기

같은 상세에서 `PROVISIONING → RUNNING`을 기다립니다. 이름뿐 아니라 방금 생성한 instance 상세를 기준으로 Image, Shape, OCPU/Memory를 확인합니다. **Networking → Attached VNICs**에서 primary VNIC의 subnet=`lab-public`인지 확인합니다. SSH를 선택했다면 실제 public IPv4를 메모합니다. Console만 선택했다면 public IPv4가 없는 것이 정상입니다. **Storage/Boot volume**에서 연결된 OS 디스크의 이름과 AD를 메모합니다. 정리할 때 이 디스크를 확인합니다.

VM 생성·Console 확인은 여기까지입니다. SSH를 선택하지 않았다면 [리소스정리](#리소스정리)로 이동합니다. SSH를 선택했다면 다음 단계에서 VM 안의 OS를 직접 확인합니다.

### 4. 선택: SSH로 로그인하여 OS 확인하기

**Linux/WSL PC 터미널**에서 `KEY`를 **다운로드한 private key의 실제 경로**, `PUBLIC_IP`를 **3단계 실제 public IPv4**로 바꿉니다. WSL에서는 Windows Downloads의 키를 WSL 홈의 개인 폴더(예: `~/oci-lab/ssh-key.key`)로 복사하고 그 Linux 경로를 `KEY`에 사용하세요. `/mnt/c`의 원본에 `chmod`가 Linux 권한처럼 적용된다고 가정하지 않습니다. 아래 Downloads 경로는 예시입니다.

**Windows OpenSSH를 직접 사용할 경우**: [Oracle: Windows OpenSSH 연결](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/connect-to-linux-instance.htm#linux-from-windows-openssh)의 **Set the Permissions for the Private Key File**을 파일 탐색기에서 완료한 뒤 **Connect to the Instance with PowerShell**을 따릅니다. private key에는 2단계의 실제 Windows 파일 경로, username에는 `opc`, public IP에는 3단계 값을 사용합니다. 아래 Bash 연결 블록은 건너뛰고, 연결 후 이 단계의 **첫 연결 신뢰 확인과 VM 터미널 OS 확인**으로 돌아오세요. 5단계 재접속도 같은 Windows 연결 절차를 사용합니다.

```bash
KEY="$HOME/Downloads/ssh-key.key"
PUBLIC_IP='<Console의 실제 public IPv4>'
chmod 400 "$KEY" &&
ssh -i "$KEY" -o IdentitiesOnly=yes -o StrictHostKeyChecking=ask \
  -o ConnectTimeout=10 -o ConnectionAttempts=1 opc@"$PUBLIC_IP"
```

첫 연결의 `Are you sure you want to continue connecting (yes/no/[fingerprint])?` 질문은 **서버 신원**을 처음 신뢰할지 묻는 것입니다. 접속 IP가 방금 만든 VM의 Console public IP와 같은지 확인합니다. 신뢰할 수 있는 별도 fingerprint가 있다면 함께 비교하고, 대상이 확실할 때 `yes`를 입력합니다. 이 최초 신뢰는 서버 키를 PC에 저장하는 것이며 독립적인 신원 검증은 아닙니다. 대상이 불확실하거나 다음 접속에서 host key 변경 경고가 나오면 취소하고 대상 VM을 확인합니다. host key 검사를 끄거나 경고를 무시하지 않습니다.

로그인된 **VM 터미널**에서 다음을 실행합니다.

```bash
id -un
uname -m
cat /etc/os-release
```

예상 결과는 `opc`, A1이면 `aarch64`, Oracle Linux 9입니다. 이 결과로 VM 안의 사용자와 OS를 확인합니다. `exit`로 PC에 돌아옵니다.

### 5. 선택: 같은 VM의 HTTPS 확인하기

이 단계는 VM에서 외부 웹사이트로 나가는 HTTPS 요청입니다. 웹서버를 설치하거나 PC에서 VM의 HTTPS 서비스를 여는 실습이 아닙니다. SSH 로그인을 마쳤고 이 요청도 확인하고 싶을 때만 진행합니다. 이번 `sl-public`의 **Add Egress Rules**에서 stateful, Destination Type=`CIDR`, Destination CIDR=`0.0.0.0/0`, IP Protocol=`TCP`, Source Port Range=`All`, Destination Port Range=`443`를 저장합니다. 이 규칙은 VM에서 시작하는 TCP443 연결을 허용합니다. stateful 규칙은 그 연결의 응답도 허용하므로 별도 inbound443 규칙은 필요하지 않습니다.

같은 PC에서 4단계 SSH 명령으로 재접속하여 **VM 터미널**에서 실행합니다.

```bash
getent ahostsv4 www.oracle.com
curl -4 --connect-timeout 5 --max-time 15 -I https://www.oracle.com/
exit
```

DNS 주소와 TLS 검증을 통과한 HTTP 2xx/3xx 응답을 확인합니다. 403/429이면 사이트의 접근 제한일 수 있습니다. timeout이면 `rt-public`의 IGW 경로와 방금 추가한 egress443을 확인하고, DNS 실패이면 subnet의 default DHCP 선택을 확인합니다. 도구가 없으면 이 선택 단계를 건너뛰고 정리합니다. TLS 오류를 `curl -k`로 우회하지 않습니다.

### 결과검증과 문제해결

기본 결과는 `RUNNING`인 VM 한 대와 올바른 image/shape/boot/VNIC/subnet 연결입니다. SSH를 선택하면 `opc`·`aarch64`·Oracle Linux 9를, HTTPS도 선택하면 DNS 조회와 HTTP 응답을 확인합니다. Console의 `RUNNING`은 OS 로그인 성공과는 다른 상태입니다.

- SSH timeout: 현재 PC 공인 IPv4/32, instance public IP, subnet의 `rt-public → igw-lab`, IGW Enabled, 연결된 `sl-public` ingress22를 확인합니다. 넓은 SSH 허용으로 고치지 않습니다.
- Permission denied (publickey): 계정 `opc`, 이 VM 생성 때 내려받은 private key 경로와 권한을 확인합니다. 키가 없으면 SSH를 완료할 수 없습니다. 복구 심화 대신 데이터가 없는 개인 새 실습 VM만 종료하고 키를 제대로 저장하여 다시 시작할 수 있습니다. 기존 데이터가 있으면 종료하지 않습니다.
- Host key 변경 경고: 접속 대상과 VM 변경 여부를 확인하기 전 계속하지 않습니다.
- 생성 실패: 선택 AD/shape 호환과 capacity/limit 및 Work Request 오류를 확인합니다. 이미 생성된 자원이 있는지 먼저 읽습니다.

### 리소스정리

기본은 지금 아래 순서로 삭제하는 것입니다. **선택: 같은 세션에서 [NAT Gateway](../Networking/NATGateway.md#사전준비)로 이어가려면**, 먼저 이 문서의 1–4단계 SSH 로그인을 완료하고 이번 `lab-vm`·boot volume·VCN과 PC의 같은 키를 잠시 남깁니다. Console-only VM은 그대로 NAT의 admin VM으로 사용할 수 없습니다. NAT에서 private VM 한 대를 추가하므로 두 VM의 비용을 확인하고, 네트워크 실습이 끝나면 NAT 정리를 따라 private 자원부터 삭제한 뒤 이 목록 1–3번과 VCN 정리로 돌아옵니다. 이어가지 않으면 바로 삭제합니다.

1. VM에 남길 파일이 있다면 먼저 PC 등 별도 저장소에 보관합니다. **Compute → Instances → 이번 lab-vm → Actions → Terminate**를 엽니다.
2. **Permanently delete the attached boot volume**을 체크합니다. 기본은 boot 보존이므로 체크 여부를 직접 확인하고 Terminate Instance를 누릅니다. 이번 VM/OS 디스크 삭제는 되돌릴 수 없습니다.
3. 같은 instance가 `TERMINATED`인지, **Storage → Block Storage → Boot Volumes**의 같은 AD/Compartment에서 이 boot volume이 삭제되었는지 확인합니다. 종료 record는 잠시 남을 수 있습니다.
4. [VCN 리소스정리](../Networking/VCN/GettingStarted.md#리소스정리)로 이번 네트워크를 삭제합니다. 다른 VM이 있다면 삭제를 진행하지 않습니다.
5. 이번 VM을 더 쓰지 않을 때 다운로드한 이번 키 파일만 PC에서 정리합니다. 다른 키나 전체 `.ssh` 폴더를 지우지 않습니다.

### 공식문서

- [Creating an Instance](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/launchinginstance.htm)
- [Managing Key Pairs](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/managingkeypairs.htm)
- [Connecting to a Linux Instance](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/connect-to-linux-instance.htm)
- [Security Rules](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/securityrules.htm)
- [Terminating an Instance](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/terminatinginstance.htm)
