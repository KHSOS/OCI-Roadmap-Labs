# Mount Targets

Mount Target은 File Storage의 NFS 접속 endpoint다. 선택한 Subnet에 위치하며 IP 주소 또는 DNS 이름으로 client와 File System을 연결한다. File System의 데이터를 저장하는 사용자 관리 VM이나 SSH 접속 대상이 아니다.

## File System과의 관계

Mount Target마다 Export Set 하나가 있으며 그 안의 Export가 File System과 Export path를 연결한다. 한 Mount Target으로 여러 File System을 제공하거나 같은 File System을 여러 Export로 제공할 수 있다. 따라서 Mount Target IP만으로 어떤 데이터를 사용하는지 알 수 없고 Export path도 함께 필요하다.

예를 들어 `mt-lab`의 주소와 `/shared`라는 Export path가 함께 NFS 접속 대상을 나타낸다. client 쪽 `/mnt/team`은 별개의 로컬 mount point다. DNS 이름은 IP를 찾는 수단이며 Export Options를 대신하지 않는다.

## Subnet과 AD 배치

Mount Target은 AD 자원이고 File System과 같은 AD의 Mount Target에서 Export를 제공한다. Subnet은 regional 또는 AD-specific일 수 있다. Regional Subnet에 있다고 File System 데이터가 자동으로 여러 AD에 복제되는 것은 아니다. 다른 AD의 client도 연결할 수 있지만 낮은 지연을 위해 client와 같은 AD 배치가 권장된다.

NSG는 Mount Target의 network interface에 연결되어 NFS 통신을 제한하고, Security List는 Subnet의 interface에 적용된다. 두 규칙의 허용 범위를 함께 고려해야 한다. Export Options는 endpoint에 도달한 뒤 어떤 File System 접근을 허용할지 결정하는 다음 층이다.

## endpoint 선택과 한계

IP 주소를 이용하면 DNS에 의존하지 않는 명시적 endpoint를 얻는다. DNS 이름을 이용하면 읽기 쉬운 주소를 사용할 수 있지만 client의 resolver와 VCN DNS 설정에 의존한다. 어느 쪽도 network 경로나 NFS 권한을 자동으로 제공하지 않는다. Mount Target은 IPv4 또는 IPv6 주소를 지원하며 client 경로와 주소 계열도 맞아야 한다.

공식 overview의 연결 한도는 Mount Target당 일반 NFS client connection 최대 100,000개, in-transit TLS 사용 시 NFS/SSL connection 최대 4096개다. 이는 client VM 수나 처리량 보장치가 아니다. 기본 Mount Target 생성 한도는 account/AD당 두 개로 설명되며 실제 tenancy Service Limit과 quota, 증설 가능성을 별도로 고려한다. endpoint를 공유하는 설계는 자원 수를 줄이지만 network 변경의 영향도 공유하므로 File System별 접근은 Export로 분리한다.

## 함께 읽기

- [File Storage](00_File_Storage.md)
- [Exports](02_Exports.md)
- [Export Options](03_Export_Options.md)
- [보존된 Hands-on 초안](../../Hands-on/Original-Provisional/05_File_Storage/01_Mount_Targets.md) — 실습 절차는 별도 자료에서 다룬다.

## 공식 참고 문서

- [Overview of File Storage](https://docs.oracle.com/en-us/iaas/Content/File/Concepts/filestorageoverview.htm)
- [Working with NFS Exports and Export Options](https://docs.oracle.com/en-us/iaas/Content/File/Tasks/exportoptions.htm)
