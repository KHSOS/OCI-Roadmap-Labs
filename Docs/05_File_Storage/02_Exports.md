# Exports

Export는 File System을 Mount Target을 통해 NFS client에게 제공하는 연결 정보다. File System OCID, Mount Target과 Export Set, Export path, client Export Options의 관계를 담는다. File System이 존재해도 Export가 없으면 client는 그 File System을 mount할 수 없다.

## 세 종류의 경로

| 경로 | 의미 | 예 |
|---|---|---|
| Export path | Mount Target 안에서 File System을 식별하는 이름 | `/team-share` |
| Client mount point | client에서 원격 File System이 보이는 로컬 디렉터리 | `/mnt/team` |
| File System path | 저장된 파일과 디렉터리의 내부 경로 | `reports/result.csv` |

Export path가 `/team-share`라고 내부에 같은 이름의 디렉터리가 생기는 것은 아니다. 같은 File System의 두 Export는 동일 데이터를 가리키며 두 복제본이 아니다. Export를 없애는 것과 File System 데이터를 삭제하는 것도 다르다.

## 여러 Export가 필요한 경우

공통 File System을 운영 client와 읽기 전용 분석 client에게 다른 source·접근 수준으로 제공할 때 Export별 옵션을 나눌 수 있다. 한 File System은 같은 AD의 하나 이상의 Mount Target을 통해 제공될 수 있다. 각 Mount Target의 Export Set에는 여러 Export가 들어갈 수 있다.

한 Export에서 client의 source를 제외해도 다른 Export나 다른 Mount Target의 허용 경로가 남으면 같은 File System에 접근할 수 있다. File System 전체의 접근 경계는 모든 Export와 network 경로를 함께 보아야 한다.

## path 선택과 제한

Export path는 생성 후 수정할 수 없다. 새 path가 필요한 경우 새 Export로 표현한다. path는 `/`로 시작하고 끝의 slash를 허용하지 않으며 요소로 `.` 또는 `..`를 사용할 수 없다. 전체 길이는 1024 bytes 이하, 각 요소는 255 bytes 이하이다.

동일 Mount Target에서 `/example`과 `/example/path`처럼 완전한 요소 순서가 겹치는 Export path는 허용되지 않는다. `/example`과 `/example2`는 서로 다른 요소다. `/`를 한 File System의 Export path로 사용하면 다른 File System을 같은 Mount Target에 연결할 수 없다. 이름은 client가 사용할 접속 식별자이므로 File System display name이나 내부 디렉터리 구조와 분리해서 설계한다.

## 접근 옵션과의 구분

Export가 무엇을 어느 endpoint와 path로 제공할지 정한다면, Export Options는 그 경로로 들어오는 client에게 무엇을 허용할지 정한다. Read Only, source CIDR, privileged source port, 인증 방식과 identity squash는 데이터 복제 옵션이 아니라 접근 제어 옵션이다. IAM의 Export 관리 권한과 NFS 파일의 POSIX 권한 역시 별개다.

## 함께 읽기

- [Mount Targets](01_Mount_Targets.md)
- [Export Options](03_Export_Options.md)
- [보존된 Hands-on 초안](../../Hands-on/Original-Provisional/05_File_Storage/02_Exports.md) — 실습 절차는 별도 자료에서 다룬다.

## 공식 참고 문서

- [Overview of File Storage](https://docs.oracle.com/en-us/iaas/Content/File/Concepts/filestorageoverview.htm)
- [Working with NFS Exports and Export Options](https://docs.oracle.com/en-us/iaas/Content/File/Tasks/exportoptions.htm)
- [Paths in File Systems](https://docs.oracle.com/en-us/iaas/Content/File/Concepts/filesystempaths.htm)
