# File Storage

File Storage는 여러 Compute instance나 container가 같은 파일을 함께 사용하는 managed network file system이다. NFSv3와 파일 잠금용 NLM을 지원하며, POSIX 파일 접근 의미를 필요로 하는 애플리케이션에 적합하다. VM의 로컬 디스크와 달리 데이터 저장 수명은 개별 client VM과 분리된다.

## 어떤 문제에 쓰는가

미디어 처리 worker가 공통 원본을 읽고 결과를 저장하거나, 분석 cluster가 같은 데이터셋을 사용하거나, 기존 NFS 기반 enterprise application을 옮길 때 공유 파일 경로가 필요하다. Object Storage의 Object API보다 디렉터리·파일·POSIX 접근이 필요한 경우 File Storage가 자연스럽다. 반대로 instance에 붙이는 block device가 필요한 경우에는 Block Volume이 다른 선택이다.

## 구성 요소와 관계

| 구성 요소 | 맡는 역할 |
|---|---|
| File System | 실제 파일과 디렉터리 데이터를 보관한다. |
| Mount Target | Subnet 안의 NFS endpoint로, client가 사용할 IP 또는 DNS 이름을 제공한다. |
| Export Set | 한 Mount Target이 제공하는 Export의 모음이다. |
| Export | File System을 Mount Target의 Export path에 연결한다. |
| Export Options | 해당 Export를 통해 들어오는 client의 source·접근 수준·인증 및 UID/GID 매핑을 정한다. |
| Snapshot | File System 전체의 특정 시점 상태를 보존한다. |

한 Mount Target은 여러 Export를 제공할 수 있다. 같은 File System도 여러 Export로 노출될 수 있으므로 endpoint 수와 데이터 복제본 수는 같지 않다. Export path는 File System 내부 디렉터리나 client의 mount point가 아니다.

## 접근과 보안의 층

IAM은 File System·Mount Target 등 OCI 자원을 관리할 주체를 정한다. NFS 파일 접근은 별도로 network 경로와 NSG/Security List, Export Options, client의 인증과 파일 UID/GID·POSIX 권한을 함께 따른다. IAM 관리 권한이 있다고 모든 파일을 읽는 것은 아니다.

NFS 데이터는 Mount Target을 향하는 private network 경로로 접근한다. Peering, FastConnect 또는 IPSec VPN으로 VCN 밖의 client를 연결할 수도 있다. Service Gateway를 통한 File Storage API 접근은 관리 요청 경로이며 NFS mount endpoint의 대체물이 아니다.

## 배치와 선택의 한계

File System 데이터는 선택한 Availability Domain 안에 저장되며 서비스가 동기 복제와 failover를 제공한다. File System을 다른 AD/Region으로 직접 이동하는 기능과 데이터 복사·Replication은 구분해야 한다. 지연을 줄이려면 client와 File Storage 자원을 같은 AD에 배치하는 것이 권장된다. 서비스 내부의 보호가 자동으로 다른 Region의 재해복구 구성을 만드는 것은 아니다.

사용량에 따라 저장 공간이 자동 확장되지만 실제 할당량은 파일의 논리적 크기와 같지 않을 수 있다. 최소 할당 block은 8192 bytes이며 Snapshot이 보존하는 변경 데이터도 사용량에 포함된다. 저장량·Snapshot 보존·관련 기능과 전송 조건을 함께 고려한다. Service Limit과 Compartment quota도 별도 제약이다.

## 함께 읽기

- [Mount Targets](01_Mount_Targets.md)
- [Exports](02_Exports.md)
- [Export Options](03_Export_Options.md)
- [Snapshots](04_Snapshots.md)
- [보존된 Hands-on 초안](../../Hands-on/Original-Provisional/05_File_Storage/00_File_Storage.md) — 실습 절차는 별도 자료에서 다룬다.

## 공식 참고 문서

- [Overview of File Storage](https://docs.oracle.com/en-us/iaas/Content/File/Concepts/filestorageoverview.htm)
