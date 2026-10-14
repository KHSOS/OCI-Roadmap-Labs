# SSH Keys

SSH key pair는 Linux instance에 원격 사용자로 인증하는 공개키와 개인키의 쌍이다. public key는 instance에 제공하고 private key는 접속하는 사용자 컴퓨터에 보관한다. SSH client는 대응 private key를 사용해 인증하므로 private key 내용을 서버에 업로드할 필요가 없다. [Oracle: Managing Key Pairs on Linux Instances](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/managingkeypairs.htm)

## 두 key의 역할

public key는 공유 가능한 인증 등록 정보이며 OpenSSH 형식은 key type·public key 값·선택 comment로 구성된다. private key는 그 public key에 대응하는 인증 비밀이다. `.pub`라는 확장자나 filename만으로 실제 내용과 쌍의 일치를 확정할 수는 없다. public key fingerprint는 key를 식별하는 요약값이며 private key 자체가 아니다.

예를 들어 사용자의 노트북에 private key를 두고 Linux VM에는 대응 public key를 제공하는 것이 일반적인 관계다. 같은 key pair를 여러 instance에 사용할 수 있지만 그 private key가 노출되면 영향을 받는 범위도 커질 수 있다. 업무별 접근 범위와 key 보관 정책을 함께 고려한다.

## key-preparation

Console이 생성하는 OpenSSH key pair와 사용자가 로컬 OpenSSH로 준비하는 key pair는 서로 다른 준비 경로다. 어느 쪽이든 instance에는 public key를 제공하고 private key는 안전하게 보관한다. Oracle은 Console에서 생성한 private key의 사본을 저장하지 않으므로 나중에 다시 내려받을 수 있다고 기대하면 안 된다.

platform image는 RSA, DSA, DSS, ECDSA, Ed25519를 지원한다고 문서화되어 있지만 custom image의 지원 key type은 고객이 관리해야 한다. 지원 목록에 있다는 사실과 조직의 보안 정책상 권장된다는 사실은 다르다. 기존 로컬 key 준비 자료는 [보존된 key 준비 Hands-on](../../Hands-on/Original-Provisional/02_Compute/05_SSH_Keys.md#part-a--인스턴스-없이-key-생성-compute의-필수-선행)에 있다.

## IAM·사용자 key·host key의 구분

OCI IAM은 Console/API에서 instance를 관리하는 권한을 다룬다. SSH key는 guest 사용자 인증을 다루므로 OCI 관리 권한이 있다고 자동으로 OS 로그인이 가능한 것은 아니다. 반대로 SSH 접속이 된다고 OCI 자원의 생성·삭제 권한이 생기는 것도 아니다.

사용자 key와 서버의 **host key**도 다르다. 사용자 key는 누가 접속하는지, host key는 어떤 서버에 접속했는지를 확인하는 데 사용된다. 처음 접속할 때 보는 서버 fingerprint를 본인의 `.pub` fingerprint와 비교하는 것은 잘못된 비교다. [Compute](../../Hands-on/Original-Provisional/02_Compute/00_Compute.md#guest-trust-bootstrap)의 대상 식별 관계를 함께 읽을 수 있다.

## 보관과 접근의 한계

private key에 접근할 수 있는 사람은 그 key가 허용된 instance에 인증할 수 있다. private key의 접근 제한과 안전한 저장, 필요에 따른 passphrase는 서로 보완한다. passphrase는 private key 사용을 보호하는 값이며 server 로그인 password와 같은 것이 아니다.

key가 맞아도 network 경로·접속 사용자·guest SSH service가 맞지 않으면 접속할 수 없다. public key 등록만으로 internet 접근 경로가 생기지 않는다. key 분실이나 교체 문제는 접근 복구와 등록된 public key의 관리 문제이며 새 로컬 파일을 같은 이름으로 만든다고 기존 쌍이 복원되지는 않는다.

## 공식문서

- [Managing Key Pairs on Linux Instances](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/managingkeypairs.htm): key 역할, 생성 경로, 지원 type, private key 보관과 passphrase.
- [Connecting to a Linux Instance](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/connect-to-linux-instance.htm): 대상 주소·사용자·private key, 최초 서버 fingerprint와 접속 조건.
